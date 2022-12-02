---
layout: post
title:  "동적할당(new/delete)은 느리다"
date:   2022-09-28 16:50:00 +0900
categories: CS
comments: true
---
성능 병목이 new/delete 에서, 즉 동적할당에서 많이 발생한다는 소리를 많이 듣는다.  

별 생각없이 동적할당을 남용하는 경우가 많은데  
동적할당은 내부적으로 상당히 복잡하다.  
```cpp
// 컴파일러 마다 구현이 다름
new -> malloc() -> heapAlloc() -> virtualAlloc()
1. new가 malloc 및 생성자 호출
2. malloc은 heapAlloc(windowsAPI)을 호출
3. heapAlloc은 가지고 있는 메모리가 없다면 virtualAlloc에 페이지 단위의 메모리 할당 요청
4. 페이지 단위의 메모리를 new에서 요청한 사이즈로 잘라서 제공

delete -> free() -> heapFree() -> virtualFree()
1. free가 소멸자 및 free 호출
2. free는 heapFree(windowsAPI)를 호출
3. heapFree는 해제된 메모리를 바로 반환하지 않고 빠른 재할당을 위해 small, large 등으로 bin을 만들어 관리.
4. 병합할 수 있으면 병합. (병합시점은 모르겠음)
5. 적당히 쌓인 free chunk가 많으면 virtualFree 호출하여 반환.
```
이것 말고도 멀티스레딩을 위해 lock도 걸어야 한다.(없앨 순 있다)  
그리고 메모리를 스택메모리에 잡지 않기 때문에 메모리 지역성도 하락한다.  
여러모로 느리다.  

그럼 어떻게 하라는 건가?  
할 수 있다면 스택메모리(지역변수)를 최우선으로 활용한다.  
할당하는데 시간도 걸리지 않으며, (esp 내리면 끝)  
메모리 지역성도 향상.  
(예로 스택 자료구조를 만들 때 인라인 어셈블리에서 push/pop으로 곧바로 구현할 수도 있다.)  

물론 linked-list 같은 문제들은 지역변수로 불가능하다.  
이런 경우는 고정된 크기만 할당이 가능한 메모리 풀을 만들어 해결한다.  
메모리 풀을 사용하면 동적할당은 풀을 만들때 한번만 하면 끝나고,  
메모리 지역성이 높아지며,  
동적인 사이즈를 취급하지 않기 때문에 할당/해제를 100사이클 이하로 끝낼 수 있다.  
[**[메모리 풀 코드(Github)]**](https://github.com/Ria9993/PlayGround/tree/main/Static%20Memory%20Pool)  
위처럼 메모리 풀로 작성하면 기존 CRT Heap 보다 100배 까지도 빠르다.   
![image](https://user-images.githubusercontent.com/44316628/193073869-d7c8a50b-9ae9-404a-8da5-cf02977bdc21.png)  
해당 내용은 프로그래머 유영천 님이 잘 정리해주신 비디오가 있다.  
성능 측정도 해당 영상에서 가져왔다.  
<iframe width="560" height="315" src="https://www.youtube.com/embed/wB74q02x_P0" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

(런타임에 이루어지는 것들과  
범용적이고 유연한 기능은 대체로 느리다 (ex. Heap 메모리))
