---
layout: post
title:  "Jacobsthal sequence의 일반항 유도"
date:   2024-03-05 00:22:30 +0900
categories: CS
comments: true
use_math: true
---

# 개요
Merge-insertion sort 라는 정렬 알고리즘을 찾아보려고 원본 논문을 봤는데,  
알고리즘에서 사용되는 수열에 대한 일반항을 유도 설명없이 식만 적어놨다...  

$$ a_0 = 0 \\  
a_1 = 1 \\  
a_n = a_{n-1} + 2a_{n-2} \ \ \  (n \geq 2)
$$  
$$ \{ 0, 1, 1, 3, 5, 11, 21, 43, ... \} $$  

일단 수열의 점화식은 위와 같다.  
피보나치 수열 $ \{ a_n = a_{n-1} + a_{n-2} \} $ 과 비슷한 형태.  

일단 점화식을 가지고 일반항을 유도해볼 건데,  
방법은 이것저것 있긴 한데 생성함수로 일단 유도해보자.  

***
# Generating function  
일단 생성함수가 무엇인지 설명하자면,  
일단 수열이나 경우의 수 등의 각 항을 계수로 하는 다항식을 생각하는 것이다.  

예를 들어,  
1~6까지 적힌 주사위를 던져서 나올 수 있는 경우의 수를 식으로 나타내려면 이렇게 나타낼 수 있다.  

$$ (1x + 1x^2 + 1x^3 + 1x^4 + 1x^5 + 1x^6) $$  
각 지수는 주사위의 눈을 나타내고, 각 계수는 그 눈이 나올 확률을 나타낸다.  
생성함수는 이렇게 각 항을 계수로 하는 다항식을 생각하는 것이다.  

수열도 마찬가지로 각 항을 계수로 생성함수를 정의할 수 있다.  
$$ G(x) = 0 + 1x + 1x^2 + 3x^3 + 5x^4 + 11x^5 + ... $$  

여기서 특정 지수의 계수를 골라낼 수 있다면, 곧바로 수열의 n번째 항을 구할 수 있을 것이다.  


일단 Jacobsthal sequence의 생성함수 $ G(x) $를 다음과 같이 정의하자.  
$$ G(x) = a_0 + a_1x + a_2x^2 + a_3x^3 + ... = \sum\limits_{n=0}^{\infty} a_nx^n $$  

이걸 기존 점화식의 양 변에 $ x^n $을 곱하고  
생성함수에 맞게 $ \sum\limits_{n=0}^{\infty} a_nx^n $으로 정리하면,  

$$
\begin{align*}
a_n & = a_{n-1} + 2a_{n-2} \\  
a_nx^n & = a_{n-1}x^n + 2a_{n-2}x^n \\  
\sum\limits_{n=2}^{\infty} a_nx^n & = \sum\limits_{n=2}^{\infty} a_{n-1}x^n + 2\sum\limits_{n=2}^{\infty} a_{n-2}x^n \\  
\sum\limits_{n=0}^{\infty} [a_nx^n] - a_0 - a_1x & = x \sum\limits_{n=2}^{\infty} [a_{n-1}x^{n-1}] + 2x^2 \sum\limits_{n=2}^{\infty} [a_{n-2}x^{n-2}] \\  
\sum\limits_{n=0}^{\infty} [a_nx^n] - x & = x \sum\limits_{n=0}^{\infty} [a_{n}x^{n}] + 2x^2 \sum\limits_{n=0}^{\infty} [a_{n}x^{n}] \\  
G(x) - x & = xG(x) + 2x^2G(x) \\  
G(x) - xG(x) - 2x^2G(x) & = x \\  
G(x)(1 - x - 2x^2) & = x \\  
G(x) & = \frac{x}{1 - x - 2x^2}  
\end{align*}
$$  

이렇게 수열의 생성함수를 $ x $에 대하여 구할 수 있는데,  
우리가 원하는 $n$번째 수열의 항을 구하기 위해선 여기서 $x^n$의 계수 걸러내야 한다.  

이를 위해 **부분분수분해** 를 사용할 것이다.  
부분분수분해는 통분된 분수를 다른 분수로 분해하는 방법인데,  

$$ \frac{P(x)}{Q(x)} = \frac{c_0}{x - r_0} + \frac{c_1}{x - r_1} + ... + \frac{c_n}{x - r_n} $$  
$P(x)$와 $Q(x)$가 x에 대한 다항식이고,  
$Q(x)$가 $(x - a_0)(x - a_1) \cdot\cdot\cdot (x - a_n)$의 형태라면  
위와 같이 분해할 수 있다.  
$c_0, c_1, ... c_n$은 임의의 상수.  

$ \frac{x}{1 - x - 2x^2} $ 는 분모와 분자가 x에 대한 다항식이므로  
위의 형태로 분해할 수 있다.  

일단 $x$의 해를 구하면  

$$ 1 - x - 2x^2 = 0 $$  
$$ 
\begin{align*}  
x & = \frac{-1 \pm \sqrt{1 + 8}}{-4} = \frac{-1 \pm 3}{-4} \\  
& = \frac{1}{2} \ or -1
\end{align*}  
$$  
$x = \frac{1}{2} \ or -1$ 이므로 부분분수분해 형태로 나타내도록 하자.  

$$
\begin {align*}
G(x) & = \frac{x}{1 - x - 2x^2} \\  
& = \frac{x}{(x - \frac{1}{2})(x + 1)} \\  
& = \frac{x}{(1 - 2x)(1 - (-x))} \\  
\end{align*}
$$  

$$
\begin{align*}
\frac{x}{(1 - 2x)(1 - (-x))} & = \frac{A}{1 - 2x} + \frac{B}{1 - (-x)} \\  
x & = A(1 - (-x)) + B(1 - 2x) \\  
\\  
A  = \frac{1}{3} & \quad when \ x = -1 \\  
B = -\frac{1}{3} & \quad when \ x = \frac{1}{2} \\   
\end{align*}
$$  
후에 무한등비급수를 사용할 것이므로 $(1 - x)$ 형태로 정리했다.  

이제 $G(x)$의 부분분수분해 식에 위에서 구한 $A, B$를 대입해보자.  
$$
\begin{align*}
G(x) & = \frac{1}{3} \cdot \frac{1}{1 - 2x} - \frac{1}{3} \cdot \frac{1}{1 - (-x)}
\end{align*}
$$  

여기서 $\frac{1}{1 - x}$의 형태는 무한등비급수를 사용할 수 있다.  
원래 ${\|x\|} < 1$일 때만 성립하는데,  
우리가 사용하는 생성함수는 formal power series이므로 coverage를 무시해도 된다.  
$$
\begin{align*}
G(x) & = \frac{1}{3} \cdot \frac{1}{1 - 2x} - \frac{1}{3} \cdot \frac{1}{1 - (-x)} \\  
& = \frac{1}{3} \sum\limits_{n=0}^{\infty} (2x)^n - \frac{1}{3} \sum\limits_{n=0}^{\infty} (-x)^n \\  
& = \sum\limits_{n=0}^{\infty} \frac{1}{3} \cdot 2^n \cdot x^n - \sum\limits_{n=0}^{\infty} \frac{1}{3} \cdot (-1)^n \cdot x^n \\  
& = \sum\limits_{n=0}^{\infty} [\frac{2^n - (-1)^n}{3}] \cdot x^n \\  
\end{align*}
$$  

우리는 $n$번 항의 계수를 구하는 것이 목적이므로  
$x^n$는 필요없음.  

$$
a_n = \frac{2^n - (-1)^n}{3}  
$$  
최종적으로 구한 Jacobsthal sequence의 일반항은 위와 같다.  
