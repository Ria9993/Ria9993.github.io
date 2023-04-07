---
layout: post
title:  "Race Condition은 왜 발생하는가 + atomic은 느리다"
date:   2023-03-22 16:12:00 +0900
categories: CS
comments: true
---
Cache Coherence가 보장되는 cpu에서 atomic operation(std::atomic)이 왜 필요할까.  

하나의 operation은 여러 단계로 쪼개어질 수 있다.  
예를 들어 i++ 이라는 코드는 다음과 같은 uop으로 쪼개어질 수 있다.  
```nasm
// i++
load  eax, [i]
add.  eax, 1
store [i], eax
```  
캐시 일관성 프로토콜은 저기서 “store를 수행할 때” 만 다른 코어들의 캐시라인 버스를 잠근다.(MESI Protocol)  
그래서 다른 코어의 load 실행을 위 세 단계 중간에 막지 못하기 때문에 Race condition 이 존재한다.  

여기까지가 Race Condition의 발생 이유이고,  
사실 이 글에서 다룰 건 이 문제를 해결하는 atomic operation 원리와 문제점이다.  

자. 위에서 발생한 Race Condition을 방지하려면 어떻게 하면 될까?  
세 단계를 수행할 동안 다른 코어의 작업을 막으면 된다.  
Cache Coherence에서 "store 할 때 다른 코어의 데이터에 대한 접근을 잠그고 데이터를 무효화 시킨다"  
라는 개념을 세 단계 모두에 확장시키면 된다.  
그래서 atomic operation은  
"load-modify-store 단계를 진행하는 동안 다른 코어의 해당 데이터 캐시라인에 대한 접근을 잠그고 데이터를 무효화 시킨다"  
라는 원리으로 동작한다.  

보통 atomic이 엄청 빠르고, lock-free라고 얘기를 하지만  
위의 원리를 보면 생각보다 느리고 lock이 없진 않다.  
atomic으로 발생할 문제들이 무엇이 있을까.  

## 1. Cache Lock  
atomic opertion을 구현되려면,  
서술했듯이 해당 캐시라인을 들고있는 모든 코어들의 캐시버스를 잠궈야 한다.  
그렇게 되면 다른 코어들이 전부 방해를 받고 속도가 느려진다.  
(캐시 일관성 프로토콜도 마찬가지. 거기선 false sharing이란 이름으로 알려져 있음)  

+atomic operation이 MESI protocol의 확장이라고 설명했는데,  
실제로 현재 x86의 atomic은 캐시 일관성의 MESI protocol을 사용해 주로 구현된다.  
MESI protocol이 불가능한 프로세서에서는 메모리 버스 전체를 잠궈버리는 식으로 구현한다.  
(이건 is_lock_free() 로 확인할 수 있다)

## 2. Write-combined buffer flush (mfence)  
std::atomic 은 연산이 끝난 후,  
즉시 다른 코어에서 연산결과를 관측할 수 있어야 하기 때문에 mfence 명령을 동반한다. (or lock 접두)  
쓰기가 reorder되거나 버퍼링 되면 결과를 곧바로 기록하지 않을 수 있기 때문이다.  

이 mfence는 reodering 을 제한하는데서 오는 성능하락도 있고,  
cpu에는 write-combined buffer라고 하는 버퍼가 있는데 이를 flush해야 한다.  

(cpu에서 bus transaction을 열려면 꽤 많은 시간이 들고,  
한 번 열리면 burst mode로 캐시라인 단위로 빠르게 전송할 수 있다.  
그래서 write할 메모리를 캐시라인 단위로 모아놨다가 버스 개통시 한 번에 flush 한다.  
이게 WC buffer 이다.)  

여기서 flush를 해야 한다면 버스를 마스터링 하는 시간 걸리고,  
만약 write-combined buffer가 캐시라인 단위로 꽉 채워지지 않았다면  
캐시라인 단위로 한번에 전송하지 못하고 워드단위로 하나하나 메모리에 써야한다.  
이러면 엄청난 병목이 발생하게 된다.  
(CACHE - DRAM 간의 전송은 워드단위만 가능하지만 CACHE - CACHE 간의 전송은 캐시라인 사이즈로 한 방에 전송 가능하다)

## 성능저하 예시 - spinlock  
mutex 같은 다른 lock 들도 결국에 atomic operation을 내부적으로 사용해야 한다.  
(혹은 같은 원리)  
그 중에 위에서 얘기한 성능저하 문제가 제일 잘 나타나는 경우가 spinlock 이다.  
```c
while (try_lock() == false)
{
}
```
spinlock은 lock을 시도하면서 같은 락 개체를 가진 다른 코어들의 캐시라인을 계속 잠근다.  
그럼 다른 코어들이 모두 성능저하를 겪는 것이다.  
또 WC buffer 도 계속 flush 한다.  

이걸 어떻게 해결할 수 있을까?  
wnidows에는 SRWlock이라는 해결책을 제시한다.  
x86에는 pause라는 명령어가 있는데 20~500클럭 정도 그냥 대기하는 명령어다.  
이걸 이용해서 한 번 lock을 획득할 수 있는지 검사하고,  
획득할 수 없다면 다시 검사해서 계속 버스를 잠그는 것이 아닌 pause를 써서 잠시 대기하는 것이다.  
이런 식으로 가끔씩 lock 획득을 시도하거나, 안된다 싶으면 wait 큐로 간다.  
