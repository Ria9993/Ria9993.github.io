---
layout: post
title:  "동적할당(new/delete)은 느리다"
date:   2022-09-28 16:50:00 +0900
categories: CS
comments: true
---
제품을 만드는 분들의 얘기를 많이 듣는데 (포프님이나 유영천님...)  
성능 병목이 new/delete 에서, 즉 동적할당에서 많이 발생한다는 소리를 많이 듣는다.  

별 생각없이 동적할당을 남용하는 경우가 많은데  
new/delete는 내부적으로 상당히 복잡하다.  
```
OS에 시스템 콜로 메모리를 페이지 단위로 할당받아온 후에  
(커널까지 갔다 와야하므로 당연히 느릴 수 밖에...)  
Best-fit을 위해 몇개의 버킷을 만들어 메모리를 small,large 등으로 나누어 구분하고..  
반환 요청 오면 병합 처리하고...  
(작은 사이즈는 LFH(Low Fragmentation Heap)으로  
어느정도 속도를 방어하긴 하지만 그래도 여전히 느리다)  
```
그리고 메모리를 스택메모리에 잡지 않기 때문에 메모리 지역성도 하락한다.  

그럼 어떻게 하라는 건가?  
할 수 있다면 스택메모리(지역변수)를 최우선으로 활용한다.  
할당하는데 시간도 걸리지 않으며, (esp 내리면 끝)  
메모리 지역성도 향상.

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

(런타임에 이루어지는 것들은 대체로 느린게 당연하다.  
C/C++은 컴파일타임과 런타임을 잘 구분해야 하는 언어라 생각한다.)