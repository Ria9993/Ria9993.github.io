---
layout: post
title:  "배열 순회 방법에 따른 성능 차이 with Cache"
date:   2022-10-04 04:25:00 +0900
categories: CS
comments: true
---
2차원 배열 arr[len][len] 가 있다 가정해보자.  
```c
int arr[len][len]
```
이 배열을 순회하는 방법이라고 한다면 일단 2개가 떠오를 것이다.  
```c
1. row-first
for (int row = 0; row < len; row++)
    for (int col = 0; col < len; col++)
        arr[row][col];

2. col-first
for (int col = 0; col < len; col++)
    for (int row = 0; row < len; row++)
        arr[row][col];
```
row부터 도는 방식과 col부터 도는 방식이다.  
편하게 row-first, col-first로 각각 지칭하겠다.  

둘 중에 뭐가 순회 속도가 더 빠를까?  
row-first 방식이 물론 더 빠를 것이다.  

모르는 이를 위해 설명하자면  
Cache에 메모리를 fetch 할 때 캐시라인 단위로 인접한 원소들이 한꺼번에 fetch 되기 때문이다.  
하지만 col-first 방식으로 순회한다면 다음 접근할 원소가 연속적이지 않으니 Cache Miss 가 증가한다.  
성능 테스트를 해보자.  
```
row-first loop : 1893800ns
col-first loop : 2583700ns
```
그런데 어라. 생각보다 성능차이가 안 난다.  

많이 난다고 생각한다면 random하게 순회하는 속도를 보자.  
생각대로라면 col-first 방식과 random하게 순회하는 방식이 속도가 비슷해야 할 것이다.  
**[[테스트 코드(Github)]](https://github.com/Ria9993/PlayGround/blob/main/Cache%20Prefetch%20Test/Cache%20Prefetch%20Test/main.cpp)**
```
row-first loop : 1893800ns
col-first loop : 2583700ns
random loop    : 5404900ns
```
random이 훨-씬 느리다.  
왜 이럴까?  

Cache miss는 큰 병목이다.  
그렇다면 명령어 수행 이전에 미리 cache에 fetch 해온다면 어떨까.  
이걸 cache prefetch 라고 한다.  
Cache prefetch는 소프트웨어 에서 해주거나, 하드웨어에서 해주는 방식 2가지가 있는데  
하드웨어가 꽤나 좋아져서 소프트웨어에서는 잘 사용하지 않는다.(소프트웨어가 하드웨어보다 당연히 느리다)  

어쨋든 하드웨어에서 이 역할을 하는 녀석을 **Cache Prefetcher** 라고 부른다.  
이 녀석이 어떻게 동작하느냐?  
바로 일정한 스텝이나 패턴으로 cache에 fetch 하는걸 인식하고,  
다음 fetch를 예측하여 미리 prefetch 한다.  
Branch prediction과 비슷하다.  

그래서 row-first 방식의 순회가 일정한 스텝으로 접근하기에 Prefetch 되어 빨랐던 것이다.  
random 하게 순회하는 방식은 다음 fetch를 예측할 수 없으니 느렸던 것.
