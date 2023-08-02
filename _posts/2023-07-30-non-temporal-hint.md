---
layout: post
title:  "SIMD Non-temporal Hint (w/ LFB,BUS,DMA)"
date:   2023-07-30 22:01:00 +0900
categories: CS
comments: true
---
SIMD instructions 에 재밌는 명령어가 꽤 있다.  

오늘은 movntq, vmovntpd 등이 그 주인공이다.  
(nt 가 non-temporal 을 의미한다.  
intrinsic에선 mm_steram_pd 등의 이름.)  

# Non-Temporal memory hint  
한 줄로 요약하자면, non-temporal. 즉 캐시메모리에 쓰지 말라는 얘기다.  
non-temporal move 명령어들은 캐시에 쓰지 않고 곧바로 메모리에 읽거나 써버린다.  

## 대체 왜 필요한가?
굉장히 많이 쓰고, 쓸 일이 많다.  

cpu의 모든 읽기/쓰기 연산은 기본적으로 L1 Cache에 행해진다.  
무슨 말이냐면,   
읽기/쓰기 연산을 하려면 해당 연산을 수행할 데이터가  
DRAM→L3→L2를 거쳐 L1까지 위치해 있어야 한다.  
(쓰기도 L1까지 load해야 하는 이유는 아래에서 서술)  

그러면 내가 사용할 데이터가 L1에 미리 없다면 어떻게 되는가?  
상위레벨의 L3, L2, L1 에 로드되어있는 다른 데이터들을 축출해내서 공간을 만든 뒤 L1까지 로드해야한다.   

하지만 다른 데이터들을 다 쫓아내고 L1까지 해당 데이터를 로드했는데,  
해당 데이터가 한 번만 쓰고 마는 데이터라면?  
해당 데이터가 캐시보다 큰 매우 큰 데이터라서, 다시 접근할때는 이미 L1에서 내려간 상태라면?  

상당한 성능 하락, Cache miss를 겪게 되는것이다.  

그래서 한 번만 쓰거나 읽고 마는 데이터,  
매우 큰 데이터를 사용할 때 non-temporal hint를 사용해 캐시를 점유하지 않도록 하는것이다.  

# Write Combined Buffer (Line Fill Buffer)

non-temporal hint 는 사실 무엇보다  
Write Combined Buffer의 동작에 날개를 달아주는 기능이다. (intel에선 Line Fill Buffer, LFB라고 부름)  

현대 CPU들은 보통 load buffer, store buffer, write combined buffer 등의 버퍼를 가지고 있다.  
load/store buffer 는 단순히 다음 메모리에서 읽기/쓰기 할 명령의 대기열이지만,  
Write Combined Buffer는 약간 특이하다.

이게 무얼 하는 유닛들인지를 이해하려면,   
한 가지 재밌는 사실을 알아야 한다.  

**CPU는 write를 수행해도 곧바로 캐시/메모리에 write를 하지 않는다.**  

성능을 위해 지원하는 위대한 기능이다.  
실제로 메모리에 write가 필요한 시점까지 최대한 write를 버퍼에 쌓아 지연하다가,  
”진짜로” 메모리에 기록해야하는 상황이 오면 그 때 메모리에 write를 수행한다.  

여기엔 세 가지 이유가 있다.  

**1.**  
CPU 내부 캐시 버스는 256/512 bit을 지원한다. (SIMD register width)  
즉 한 번의 read/write에 256/512 bit을 한꺼번에 수행한다.  
그래서 조금씩 write하는 것 보다, 256/512 bit을 한꺼번에 write하는게 이득이다.  

**2.**  
read가 아닌 write 연산도 write하려는 해당 캐시라인을 L1까지 끌고와야한다.  
왜냐하면 캐시라인의 일부에만 write 하는 경우, 다른 부분의 값도 알고있어야 하기 때문이다.  
(그래야 나중에 하위레벨 메모리에 다시 쓸 수 있음)  

하지만 캐시라인 전체에 write를 수행하는 경우,  
해당 캐시라인에서 알아야 하는 다른 값이 존재하지 않기 때문에 load가 필요없어진다.  

**3.**  
버스 트랜잭션을 절약할 수 있다.  

만약 32비트씩 8번의 버스 전송을 해야한다면 버스 트랜잭션을 8번 수행해야 한다.  

하지만 8개의 32비트가 한 캐시라인 전체를 이루고 있다면  
Burst 모드로 한 번의 트랜잭션동안 캐시라인 단위의 전송을 할 수 있다.  
트랜잭션이 길어지면 DMA를 사용할 여지도 커진다.  
(정확히는 전송이 끝날 때 까지 트랜잭션을 끝내지 않는다)  

특히 멀티코어에서 DRAM에 대한 접근은 모두 직렬화(순서화/동기화) 되므로  
트랜잭션 개수 감소는 멀티스레딩에서 이득이 크다  

# LFB Flush
장점이 많은 만큼 주의해야 할 점도 많다.  

우선, MFENCE를 만나면 어떻게 되냐는거다.  
그러면 당장 store buffer, load buffer, write-combined buffer에 쌓인 명령을 모두 처리해야한다.  
(mutex, atomic 도 마찬가지다. x86 lock prefix는 mfence를 동반한다)  

store/load buffer 에 쌓인 작업은 대부분 곧바로 L1에 쓰면 되지만,  
write-combined buffer에 쌓인 데이터는 캐시에 없을것이기 때문에  
WC buffer가 전부 채워지지 않았을 경우,   
DRAM에서부터 L1까지 데이터를 다시 가져온 다음 써야한다.  
(DRAM 액세스 레이턴시는 100사이클 이상)  

그리고, WC memory를 읽는 행위도 위와 같이 느린 flush 작업을 수행해야 한다.  
(유영천님 블로그)  
[https://megayuchi.com/2021/06/06/ddraw-surface-d3d-dynamic-buffer-에서의-write-combine-memory/](https://megayuchi.com/2021/06/06/ddraw-surface-d3d-dynamic-buffer-%EC%97%90%EC%84%9C%EC%9D%98-write-combine-memory/)  

# WC property page 
굳이 SIMD의 nt 명령어가 아니더라도,  
일반적인 read/write가 non-temporal로 작동하도록 하는 방법이 있다.  
바로 WC 속성의 page이다.  

page property에 WC 속성 비트가 존재하고,  
페이지 접근 시 MMU가 속성 비트를 검사하고 non-temporal 하게 read/write를 수행할 수 있다.  

주로 사용되는 곳은 그래픽스 API에서 Driver가 제공하는 버퍼를 예로 들 수 있다.  
GPU로 메모리 전송을 사용되는 버퍼이기 때문에,  
read 할 일이 없으면 WC 속성의 버퍼를 사용하는 것이다.  
