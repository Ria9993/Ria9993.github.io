---
layout: post
title:  "동적할당(new/delete)은 느리다"
date:   2022-09-28 16:50:00 +0900
categories: CS
comments: true
---
성능 병목은 주로 new/delete 에서, 즉 동적할당에서 많이 발생한다.
내부적으로 동적할당을 수시로 사용하는 STL container도 해당한다.

별 생각없이 동적할당을 남용하는 경우가 많은데  
동적할당은 복잡해서 오버헤드가 크다. (할게 많음)  

thread-safe를 위해 lock도 걸고... (없앨 순 있다)  
메모리를 스택메모리에 잡지 않고 메모리 지역성도 하락한다.  
LFH나 bin 같은 효율성을 위해 여러 기술을 붙여도 느리다.  
그리고 파편화 문제도 발생한다.  

그럼 어떻게 하라는 건가?  
할 수 있다면 스택메모리(지역변수)를 최우선으로 활용한다.  
할당하는데 시간도 걸리지 않으며, (esp 내리면 끝)  
메모리 지역성도 향상.  
(스택 자료구조를 만들 때 인라인 어셈블리에서 push/pop으로 곧바로 구현할 수도 있다.)  

물론 linked-list 같은 문제들은 스택 메모리로 해결하기 힘들다.  
이런 경우는 고정된 크기만 할당이 가능한 메모리 풀을 만들어 해결할 수 있다.  
메모리 풀을 사용하면 동적할당은 풀을 만들때 한번만 하면 끝나고,  
메모리 지역성이 높아지며,  
동적인 사이즈를 취급하지 않기 때문에 할당/해제를 100사이클 이하로 끝낼 수 있다.  
[**[메모리 풀 코드(Github)]**](https://github.com/Ria9993/PlayGround/tree/main/Static%20Memory%20Pool)  
위처럼 메모리 풀로 작성하면 기존 CRT Heap 보다 100배 까지도 빠르다.   
![image](https://user-images.githubusercontent.com/44316628/193073869-d7c8a50b-9ae9-404a-8da5-cf02977bdc21.png)  
해당 내용은 프로그래머 유영천 님이 잘 정리해주신 비디오가 있다.  
성능 측정도 해당 영상에서 가져왔다.  
<iframe width="560" height="315" src="https://www.youtube.com/embed/wB74q02x_P0" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

(c/c++ 에서 런타임에 이루어지는 것들과  
범용적이고 유연한 기능은 대체로 느리다 (ex. Heap 메모리))
