---
layout: post
title:  "if-else와 switch의 차이"
date:   2022-09-29 05:04:00 +0900
categories: CS
comments: true
---
if-else와 switch는 뭐가 다를까?

컴파일러가 발전한 지금에는 굳이 몰라도 된다고 말하긴 하지만  
언제나 컴파일러가 최적화 해준다는 보장이 없으니 알아두면 좋다.  

먼저, if-else는 모두가 아는 그 방식이다.  
비교하고, 점프하고, 비교하고, 점프하고.  

switch가 특이한데, 점프 테이블 혹은 LUT라고 부르는 방식으로 최적화 한다.  
아래와 같이 switch문을 작성 해보자.  
```c
int test(int n)
{
    switch (n)
    {
        case 0:
            return 5;
        case 1: // fallthrough
        case 2:
        case 3:
            return 10;
        case 5: // fallthrough
        case 7:
            return 15;
        default:
            return 0;
    }
}
```
이 코드를 if로 하나하나 비교한다고 생각해보면 너무 비효율적이다.  
굳이 비교할 필요가 없이 테이블을 만들어 한번에 해결할 수 있는데,  
C언어로 예시를 보자.  
```c
int test(int n)
{
    const int LUT[8] = { 5, 10, 10, 10, 0, 15, 0, 15 };

    if ((unsigned int)n > 7u)
    {
        return 0;
    }
    return LUT[n];
}
```
배열 테이블을 만들어 미리 return값을 다 넣어두고  
n이 0-7 범위 밖인 경우를 처리한 뒤,  
추가적인 분기 없이 한번에 return 해버린다.  

그런데 실제로 컴파일러도 이렇게 최적화를 해준다  
```nasm
;Clang x86-64
test(int):                               # @test(int)
        xor     eax, eax
        cmp     edi, 7
        ja      .LBB0_2
        movsxd  rax, edi
        mov     eax, dword ptr [4*rax + .Lswitch.table.test(int)]
.LBB0_2:
        ret
.Lswitch.table.test(int):
        .long   5                               # 0x5
        .long   10                              # 0xa
        .long   10                              # 0xa
        .long   10                              # 0xa
        .long   0                               # 0x0
        .long   15                              # 0xf
        .long   0                               # 0x0
        .long   15                              # 0xf
```
테이블을 만들어 둔 뒤, 
테이블 범위 이외는 default로 0 반환.  
테이블 범위 이내는 바로 테이블 값으로 바로 반환.  

n이 0~7만 무조건 들어온다고 가정한다면  
맨 앞의 cmp를 없애 더욱 최적화 시킬수도 있다.  
C언어 코드를 바꿔보자.  
```c
#include <assert.h>
#define NDEBUG

int test(int n)
{
    switch (n)
    {
        case 0:
            return 5;
        case 1: // fallthrough
        case 2:
        case 3:
            return 10;
        case 4: // fallthrough
        case 6:
            return 0;
        case 5: // fallthrough
        case 7:
            return 15;
        default:
            __builtin_unreachable(); //GCC hint
            // or __assume(0); //MSVC hint
            // or assert(0);
    }
}
```
이러면 컴파일러는 0-7 이외 다른 값이 들어오지 않는다 가정하고  
release 빌드에서 범위를 체크하는 cmp를 없앤다.
```nasm
test(int):                               # @test(int)
        movsxd  rax, edi
        lea     rcx, [rip + .Lswitch.table.test(int)]
        mov     eax, dword ptr [rcx + 4*rax]
        ret
.Lswitch.table.test(int):
        .long   5                               # 0x5
        .long   10                              # 0xa
        .long   10                              # 0xa
        .long   10                              # 0xa
        .long   0                               # 0x0
        .long   15                              # 0xf
        .long   0                               # 0x0
        .long   15                              # 0xf
```
이렇게 분기를 아예 없애버려서 성능을 많이 올릴 수 있다.  

사실 요즘엔 if-else 도 컴파일러가 알아서 결정해서 이 방식으로 최적화 해버린다.  
그래도 내부 최적화가 어떻게 진행되는지 안다면 위처럼 default 케이스를 제거하는 식으로  
컴파일러에게 적극적인 힌트를 제공할 수 있다.  