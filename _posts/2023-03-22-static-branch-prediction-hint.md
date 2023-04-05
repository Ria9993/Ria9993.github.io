---
layout: post
title:  "정적 분기예측 컴파일러 힌트"
date:   2023-03-22 16:46:00 +0900
categories: CS
comments: true
---
현대 CPU는 Branch Prediction이라는 기능을 가지고 있다.  
파이프라인에서 분기문을 만나면 분기 결과가 나올 때까지 놀기 때문에,  
분기 결과를 런타임에서 히스토리를 기반으로 때려맞추는 것이다.  

하지만 런타임 Predictor는 분기 확률을 컴파일 타임때 알 수 없다.  
컴파일러도 분기 확률을 알 수 없다.  
왜냐하면 어떤 데이터들이 들어올 지 알 수 없기 때문이다.  
그래서 컴파일러는 둘 다 적당히 처리할 수 있는 보편적인 어셈블리어를 작성해주게 된다.  

하지만 프로그래머는 둘 중 어느쪽 분기를 많이 탈 지 예측할 수 있다.  
어떤 상황과 어떠한 데이터가 들어올지 알기 때문이다.  
그러면 직접 어셈블리를 작성하거나 컴파일러에게 힌트를 주는 식으로  
특화된 빠른 어셈블리 코드를 작성할 수 있다.  

특화된 방식은 비트연산이나 인라인어셈 등으로 직접 작성하면 되나  
컴파일러에게 힌트를 줘서 자동으로 최적화를 해준다면 편리함과 가독성까지 챙길 수 있다.  

그 컴파일러 힌트가 gcc쪽 에서는 **__builtin_expect(exp, c)**  
C++20부터는 **[[likely]] [[unlikely]]** 라는 녀석이다.  
```c
if (x == tmp)
    A_Task();
else
    B_Task();
```
다음과 같은 코드에서,  
우리가 들어올 데이터를 예상하기에 (x == tmp)의 분기를 탈 확률이 높다고 생각해보자.  

그러면 다음과 같이 힌트를 작성할 수 있다.  
```c
// gcc
if (__builtin_expect(x == tmp, true))

// c++20
if (x == tmp) [[likely]]
```
이러면 위에서 말했듯이 컴파일러가 특화된 로직으로 바꾸는 작업을 해줄 수 있다.  

물론 컴파일러가 만능은 아니기에 잘 사용되지 않거나 간단한 로직이 아니라면 로직 최적화를 기대하기 힘들다.  
그러면 로직이 복잡하여 로직을 바꿀 수 없는 상황에서는 어떻게 처리해줄까?  

자. 아까 힌트를 주지 않은 if문 코드가 어셈블리 코드로 어떻게 바뀔지 생각해보자.  
```c
if (x == tmp)
    A_Task();
else
    B_Task();
```
```nasm
    cmp   [x], [tmp]   ; compare
    je    _A_Task      ; jmp equal. x == tmp 라면 _A_Task로 뛴다
_B_Task:
    call  B_Task
    jmp   end_if
_A_Task:
    call  A_Task
end_if:
    ...
```
컴파일러는 둘 중 어느쪽으로 갈 확률이 높은지 모르기 때문에 그냥 이런식으로 배치할 것이다.  

이 코드에서 아까 힌트를 작성할 때 처럼 A_Task()가 실행될 확률이 높다고 생각하자.  
그런데 어셈블리 코드에서는 if문 분기를 계산하는 쪽 코드가 B_Task 쪽에 붙어있다.  

이러면 어떤 성능적 문제가 발생할까.  

## 1.  pipeline flush  
분기 히스토리가 쌓이지 않은 상태.  
즉 해당 if문 코드를 처음 계산하는 시점에서  
cpu의 branch predictor는 히스토리가 없기 때문에  
바로 아래에 있는 B_Task 쪽의 코드를 파이프라인에 집어넣는다.  

그러다 사실은 A_Task를 실행해야 한다는게 판명나면,  
계산하던 B_Task 실행을 rollback하고 파이프라인을 flush해야 한다.  

## 2. instruction cache  
런타임에서 branch history가 충분히 쌓여서  
이제 B_Task 코드를 실행하지 않고 예측을 잘 해서 A_Task를 우선 실행한다고 해도, 
분기 부분과 A_Task 부분 사이에 B_Task 코드가 껴있기 때문에 캐시 지역성이 떨어진다.  

(메모리 지역성이 낮으면, 즉 데이터가 모여있지 않다면 Cache memory를 더 많이 사용하게 되고  
원래 Cache를 점유하던 다른 데이터는 축출되기에 Cache-miss 가 많아진다.)  

## 해결 방식
여기서 컴파일러 힌트를 사용하면 이렇게 바꿔줄 수 있겠다.  
확률이 높은 A_Task 코드를 분기 코드 바로 아래에 작성한다.  
```nasm
    cmp   [x], [tmp]   ; compare
    jne   _B_Task      ; jmp not equal. x != tmp 라면 _B_Task로 뛴다
_A_Task:
    call  A_Task
    jmp   end_if
_B_Task:
    call  B_Task
end_if:
    ...
```
이러면 위에서 말한 2가지 문제점이 해결된다.  