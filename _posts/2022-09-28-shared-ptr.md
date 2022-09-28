---
layout: post
title:  "std::shared_ptr의 문제점"
date:   2022-09-28 20:44:00 +0900
categories: CS
comments: true
---
(MSVC를 기준으로 작성된 글 입니다.)  

std::shared_ptr은 내부적으로 어떻게 동작할까?  
strong_ref가 증가하다가, 0이 되면 곧바로 해제된다고 보통 생각할 것이다.  
그런데 MSVC같은 경우 사실 weak_ref까지 없어져야 메모리가 해제된다.  
무슨 소린지 싶겠지만 다 성능때문이다.  

자 천천히 생각해보자.  
std::shared_ptr 은 2가지가 필요한데  
참조 정보를 저장하는 [control_block] 과 들고있는 [object] 이다.  
아래 다이어그램을 보자.  
![image](https://user-images.githubusercontent.com/44316628/192790085-082a7b94-c006-45be-bdd5-aaa1cf06f13e.png)  
[1]  
다들 위 다이어그램처럼 구현이 되어있다 생각할 것이다.  

그런데 이러면 문제점이 있는데, new/free를 두 번이나 해야한다.  
[저번 포스트](https://ria9993.github.io/cs/2022/09/28/performance-of-allocation.html)에서 얘기했듯이 동적할당은 느리다.  
그래서 성능상의 문제로 사실 저 구조가 아니다. 

visual studio에서도 shared_ptr을 까보면 표면적으로도 저렇게 보이지만  
조금 더 까보면 그냥 [control_block] 과 [object]를 따로 관리하지 않고  
하나의 클래스로 한번에 new 해둔다.  
그래서 free를 한번만 해도 되도록 만들었다.  

하지만 이 구조도 문제가 존재한다.  
아래처럼 하나의 클래스로 존재한다 생각해보자.
```cpp
template <typename T>
class shared_block
{
    control_t   control_block;
    T           data;
};
```
자, 여기서 [control_block] 의 strong_ref가 0이 되었을 때  
메모리를 한번에 해제하면 되겠다!  

안된다.  
왜냐면 weak_ptr이 expired 되었는지 체크하려면 control_block을 참조해야 하는데,  
둘 다 해제해버리면 참조할 수 없기 때문이다.  
[object]는 해제하더라도 [control_block]은 남겨야 한다.

그래서 이걸 MSVC는 어떻게 해결했는가?  
strong_ref가 0이 되면 data의 소멸자만 호출한다.  
그리고 weak_ref가 0이 되면 그제서야 메모리를 해제한다.  

그래서 std::shared_ptr은 프로그래머가 메모리 해제 시점을 정확히 알기 힘들게 만든다.  
사용할 일이 있다면 염두하면서 체크할 것.

[1] : <https://learn.microsoft.com/ko-kr/cpp/cpp/how-to-create-and-use-shared-ptr-instances?view=msvc-170>