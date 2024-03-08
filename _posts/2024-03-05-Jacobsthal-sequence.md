---
layout: post
title:  "Jacobsthal sequence의 closed form 유도"
date:   2024-03-05 00:22:30 +0900
categories: CS
comments: true
use_math: true
---
<img src="/assets/images/20240305.png" width="400" height="400">   

# 개요
Merge-insertion sort 라는 정렬 알고리즘을 찾아보려고 원본 논문을 봤는데,  
알고리즘에서 사용되는 수열에 대한 closed form 식을 설명없이 적어만 놨다...(빨간색으로 표시한 부분)  
일단 수열의 명칭이 원문에서 언급되진 않았으나 동일한 형태의 수열이 Jacobsthal sequence라는 이름으로 존재한다.  

그래서 이 수열을 직접 점화식에서부터 유도해보려고 한다.   

수열의 점화식은 아래와 같다.  

$$ a_0 = 0 \\  
a_1 = 1 \\  
a_n = a_{n-1} + 2a_{n-2} \ \ \  (n \geq 2)
$$  
$$ \{ 0, 1, 1, 3, 5, 11, 21, 43, ... \} $$  

피보나치 수열 $ \{ a_n = a_{n-1} + a_{n-2} \} $ 과 비슷한 형태.  

이 점화식을 가지고 closed form을 유도해볼 건데,  
방법은 이것저것 있으나 생성함수로 일단 유도해보자.  
(사실 Characteristic equation을 사용하면 쉽게 풀 수 있지만 )

***
# Generating function  
일단 생성함수가 무엇인지 설명하자면,  
일단 수열이나 경우의 수 등의 각 항을 계수로 하는 다항식을 생각하는 것이다.  

예를 들어,  
1~6까지 적힌 주사위를 던져서 나올 수 있는 경우의 수를 식으로 나타내려면 이렇게 나타낼 수 있다.  

$$ (1x + 1x^2 + 1x^3 + 1x^4 + 1x^5 + 1x^6) $$  
각 지수는 주사위의 눈을 나타내고, 각 계수는 그 눈이 나올 경우의 수를 나타낸다.  
생성함수는 이렇게 각 항을 계수로 하는 다항식을 생각하는 것이다.  

수열도 마찬가지로 각 항을 계수로 생성함수를 정의할 수 있다.  
$$ G(x) = 0 + 1x + 1x^2 + 3x^3 + 5x^4 + 11x^5 + ... $$  

여기서 특정 지수의 계수를 골라낼 수 있다면, 곧바로 수열의 n번째 항을 구할 수 있을 것이다.  


일단 Jacobsthal sequence의 생성함수 $ G(x) $를 다음과 같이 정의하자.  
$$ G(x) = a_0 + a_1x + a_2x^2 + a_3x^3 + ... = \sum\limits_{n=0}^{\infty} a_nx^n $$  

이걸 기존 점화식의 양 변에 $ x^n $을 곱하고  
생성함수에 맞게 $ \sum\limits_{n=0}^{\infty} a_nx^n $으로 정리하면,  

$$
\begin{aligned}
a_n & = a_{n-1} + 2a_{n-2} \\  
a_nx^n & = a_{n-1}x^n + 2a_{n-2}x^n \\  
\sum\limits_{n=2}^{\infty} a_nx^n & = \sum\limits_{n=2}^{\infty} a_{n-1}x^n + 2\sum\limits_{n=2}^{\infty} a_{n-2}x^n \\  
\sum\limits_{n=0}^{\infty} [a_nx^n] - a_0 - a_1x & = x \sum\limits_{n=2}^{\infty} [a_{n-1}x^{n-1}] + 2x^2 \sum\limits_{n=2}^{\infty} [a_{n-2}x^{n-2}] \\  
\sum\limits_{n=0}^{\infty} [a_nx^n] - x & = x \sum\limits_{n=0}^{\infty} [a_{n}x^{n}] + 2x^2 \sum\limits_{n=0}^{\infty} [a_{n}x^{n}] \\  
G(x) - x & = xG(x) + 2x^2G(x) \\  
G(x) - xG(x) - 2x^2G(x) & = x \\  
G(x)(1 - x - 2x^2) & = x \\  
G(x) & = \frac{x}{1 - x - 2x^2}  
\end{aligned}
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
\begin{aligned}  
x & = \frac{-1 \pm \sqrt{1 + 8}}{-4} = \frac{-1 \pm 3}{-4} \\  
& = \frac{1}{2} \ or -1
\end{aligned}  
$$  

$x = \frac{1}{2} \ or -1$ 이므로 부분분수분해 형태로 나타내도록 하자.  

$$
\begin {aligned}
G(x) & = \frac{x}{1 - x - 2x^2} \\  
& = \frac{x}{(x - \frac{1}{2})(x + 1)} \\  
& = \frac{x}{(1 - 2x)(1 - (-x))} \\  
\end{aligned}
\\
\begin{aligned}
\frac{x}{(1 - 2x)(1 - (-x))} & = \frac{A}{1 - 2x} + \frac{B}{1 - (-x)} \\  
x & = A(1 - (-x)) + B(1 - 2x) \\  
\\  
A  = \frac{1}{3} & \quad when \ x = -1 \\  
B = -\frac{1}{3} & \quad when \ x = \frac{1}{2} \\   
\end{aligned}
$$  
후에 무한등비급수를 사용할 것이므로 $(1 - x)$ 형태로 정리했다.  

이제 $G(x)$의 부분분수분해 식에 위에서 구한 $A, B$를 대입해보자.  
$$
\begin{aligned}
G(x) & = \frac{1}{3} \cdot \frac{1}{1 - 2x} - \frac{1}{3} \cdot \frac{1}{1 - (-x)}
\end{aligned}
$$  

여기서 $\frac{1}{1 - x}$의 형태는 무한등비급수를 사용할 수 있다.  
원래 ${\|x\|} < 1$일 때만 성립하는데,  
우리가 사용하는 생성함수는 formal power series이므로 coverage를 무시해도 된다.  
$$
\begin{aligned}
G(x) & = \frac{1}{3} \cdot \frac{1}{1 - 2x} - \frac{1}{3} \cdot \frac{1}{1 - (-x)} \\  
& = \frac{1}{3} \sum\limits_{n=0}^{\infty} (2x)^n - \frac{1}{3} \sum\limits_{n=0}^{\infty} (-x)^n \\  
& = \sum\limits_{n=0}^{\infty} \frac{1}{3} \cdot 2^n \cdot x^n - \sum\limits_{n=0}^{\infty} \frac{1}{3} \cdot (-1)^n \cdot x^n \\  
& = \sum\limits_{n=0}^{\infty} [\frac{2^n - (-1)^n}{3}] \cdot x^n \\  
\end{aligned}
$$  

우리는 $n$번 항의 계수를 구하는 것이 목적이므로  
$x^n$는 필요없음.  

$$
a_n = \frac{2^n - (-1)^n}{3}  
$$  
최종적으로 구한 Jacobsthal sequence의 closed form 식은 위와 같다.  

***
# Characteristic equation  
Charaacteristic equation을 사용하여 유도하는 방법은 다음과 같은데,  

$a_n = a_{n-1} + 2a_{n-2}$  

야콥스탈 수열의 경우 상수계수를 가지는 점화식으로 만들어지므로  
수열이 지수적으로 증가한다.  

$r^n = r^{n-1} + 2r^{n-2}$  

그러므로 $a_n$을 $r^n$으로 가정해볼 수 있다. (정수여서 가능)    

$r^{n-2}$로 나눠서 정리.  
$r^2 = r + 2$  

$r^2 - r - 2 = 0$  

$r = \frac{1 \pm \sqrt{1 + 8}}{2} = 2  \ or \ -1$  

그러므로 $a_n$은 2와 -1의 거듭제곱에 대한 선형결합으로 이루어진다고 볼 수 있다.  

$$
\begin{aligned}
a_n & = c_0\cdot2^n + c_1\cdot(-1)^n \\  
\\  
a_0 & = 0 = c_0 + c_1 \\  
c_0 & = -c_1 \\  
\\  
a_1 & = 1 = 2c_0 - c_1 \\  
\\  
c_0 & = \frac{1}{3} \\  
c_1 & = -\frac{1}{3} \\
\end{aligned}
$$  

최종적으로 $a_n$은 다음과 같다.  

$$
a_n = \frac{2^n - (-1)^n}{3} \\
$$  

결국 $c_0, c_1$이 생성함수 유도에서의 $A, B$에 해당하는 녀석들이라는걸 알 수 있다.  
근본적인 풀이 원리는 같다.  

***
# Eigenbasis
수열의 점화식은 선형변환의 형태를 가지므로 이를 행렬로 나타낼 수 있다.  
$$
a_n = a_{n-1} + 2a_{n-2} \\  
$$  

$$
\begin{aligned}
\begin{bmatrix}
a_n \\
a_{n-1}
\end{bmatrix}
& =
\begin{bmatrix}
1 & 2 \\
1 & 0
\end{bmatrix}
\begin{bmatrix}
a_{n-1} \\
a_{n-2}
\end{bmatrix}
\end{aligned}
$$  

$a_n$은 
$$
\begin{bmatrix} a_{n-1} \\  
a_{n-2} \end{bmatrix}
$$
를 
$$\begin{bmatrix}
    1 & 2 \\  
    1 & 0
\end{bmatrix}$$
로 선형변환한 결과이다.  
그러니 $a_n$은 시작점 $$\begin{bmatrix} 1 \\ 0 \end{bmatrix} = \begin{bmatrix} a_1 \\ a_0 \end{bmatrix}$$ 을 n-1번 선형변환한 결과라고 볼 수 있으므로,  
다음과 같이 제곱을 이용하여 표현할 수 있다.  

$$
\begin{aligned}
\begin{bmatrix}
a_n \\
a_{n-1}
\end{bmatrix}
& =
\begin{bmatrix}
1 & 2 \\
1 & 0
\end{bmatrix}^{n-1}
\begin{bmatrix}
1 \\
0
\end{bmatrix}
\end{aligned}
$$

여기서 문제가 있는데, $$\begin{bmatrix} 1 & 2 \\ 1 & 0 \end{bmatrix}$$ 의 제곱은 계산하기 쉽지 않다. (다른 원소에 영향을 미치기 때문)  
하지만 $$\begin{bmatrix} x & 0 \\ 0 & y \end{bmatrix}$$같은 꼴의 행렬의 제곱은 간단하게 $$\begin{bmatrix} x^n & 0 \\ 0 & y^n \end{bmatrix}$$로 나타낼 수 있는데,  

이 특성을 적용시키기 위해 선형변환 $$\begin{bmatrix} 1 & 2 \\ 1 & 0 \end{bmatrix}$$ 에 대해 회전 영향을 받지 않는 두 벡터를 찾은 뒤  
이를 축(basis)으로 하는 좌표계로 변환하여 선형변환을 적용시키면 간단하게 제곱을 구할 수 있을 것이다.  

이러한 벡터를 eigenvector라고 하며,  
회전엔 영향을 받진 않으나 스케일에 영향을 받을 수 있으므로 이 스케일 값을 eigenvalue라고 한다.  

그러면 다음 행렬변환 $$\begin{bmatrix} 1 & 2 \\ 1 & 0 \end{bmatrix}$$ 의 영향을 받지 않는 벡터를 찾아보자.  
$$\begin{bmatrix} 1 & 2 \\ 1 & 0 \end{bmatrix}$$ 를 회전변환 $T$라고 하고,  
이를 식으로 표현하면 다음과 같다.  
$$
\begin{aligned}
T\vec{v} & = \lambda\vec{v} \quad (\text{ where } \vec{v} \text{ is eigenvector and } \lambda \text{ is eigenvalue})
\end{aligned}
$$  

우리는 $\vec{v}$을 원하므로 $\vec{v}$에 대해 풀어보자.  
$$
\begin{aligned}
T\vec{v} & = \lambda\vec{v} \\
T\vec{v} - \lambda\vec{v} & = \vec{0} \\
(T - \lambda I)\vec{v} & = \vec{0} \\
\vec{v} & = (T - \lambda I)^{-1}\vec{0} \\
\end{aligned}
$$  

그런데 우변은 $\vec{0}$이므로 어떠한 행렬변환의 결과도 $\vec{0}$이고 $\vec{v}$는 영벡터가 될 것이다.  
우리는 축으로 사용할 벡터가 필요하므로 영벡터는 사용할 수 없으니 $\vec{v}$는 영벡터가 아니어야 한다.  

그러니 $(T - \lambda I)$의 역행렬이 존재하지 않는 경우에만 영벡터가 아닌 $\vec{v}$를 찾을 수 있다.  

역행렬이 존재하기 위해선 $\det(T - \lambda I) = 0$ 이 아니어야 한다.  
왜냐하면 $\det(\vec{a})$가 $0$이라는 것은 한 축이 소실되어 차원이 축소되었다는 것을 의미하기 때문에,  
선형변환 이전의 벡터로 되돌릴 수 없다.  

그러므로 $\det(T - \lambda I) = 0$을 만족하는 $\lambda$를 찾는다.  

$$
\begin{aligned}
\det(T - \lambda I) & = 0 \\
\det(\begin{bmatrix} 1 & 2 \\ 1 & 0 \end{bmatrix} - \begin{bmatrix} \lambda & 0 \\ 0 & \lambda \end{bmatrix}) & = 0 \\
\det\begin{bmatrix} 1 - \lambda & 2 \\ 1 & -\lambda \end{bmatrix} & = 0 \\
(1 - \lambda)(-\lambda) - 2 & = 0 \\
\lambda^2 - \lambda - 2 & = 0 \\
\lambda & = \frac{1 \pm \sqrt{1 + 8}}{2} \\
\lambda & = 2 \ or \ -1 \\
\end{aligned}
$$  

이렇게 $\lambda$를 찾았으므로 이제 $\lambda$를 대입해서 만족하는 $\vec{v}$들을 찾아보자.  

$$
\begin{aligned}
(T - \lambda I)\vec{v} & = \vec{0} \\ 
\begin{bmatrix} 1 - \lambda & 2 \\ 1 & -\lambda \end{bmatrix}\vec{v} & = \vec{0} \\
if \ \lambda = 2&, \ \begin{bmatrix} -1 & 2 \\ 1 & -2 \end{bmatrix}\vec{v} = \vec{0} \\
v_1 - 2v_2 & = 0 \\
v_1 & = 2v_2 \\
\vec{v} & = \begin{bmatrix} 2 \\ 1 \end{bmatrix} \\
\\
if \ \lambda = -1&, \ \begin{bmatrix} 2 & 2 \\ 1 & 1 \end{bmatrix}\vec{v} = \vec{0} \\
2v_1 + 2v_2 & = 0 \\
v_1 + v_2 & = 0 \\
v_1 & = -v_2 \\
\vec{v} & = \begin{bmatrix} 1 \\ -1 \end{bmatrix} \\
\end{aligned}
$$  

이 벡터 2개를 열벡터로 하는 행렬을 새로운 좌표계 $B$로 사용하여  
선형연산을 이 좌표계에서 수행할 것이다.  
기존 선형변환 $T$를 좌표계변환 -> $\hat{T}$변환 -> 좌표계복구 로 치환.  
$$
\begin{aligned}
B & = 
\begin{bmatrix}
2 & 1 \\
1 & -1
\end{bmatrix} \\
\\
T & = B\hat{T}B^{-1} \\
B^{-1}TB & = \hat{T} \\
\end{aligned}
$$  

이제 $\hat{T}$를 구하면 이를 제곱하여 $a_n$을 구할 수 있을 것이다.  

$$
\begin{aligned}
\hat{T} & = B^{-1}TB \\
\hat{T} & = B^{-1}TB = \begin{bmatrix} 2 & 1 \\ 1 & -1 \end{bmatrix}^{-1}\begin{bmatrix} 1 & 2 \\ 1 & 0 \end{bmatrix}\begin{bmatrix} 2 & 1 \\ 1 & -1 \end{bmatrix}
\\
\hat{T} & = B^{-1}TB = \begin{bmatrix} \frac{1}{3} & \frac{1}{3} \\ \frac{1}{3} & -\frac{2}{3} \end{bmatrix}\begin{bmatrix} 1 & 2 \\ 1 & 0 \end{bmatrix}\begin{bmatrix} 2 & 1 \\ 1 & -1 \end{bmatrix} \\
\hat{T} & = B^{-1}TB = \begin{bmatrix} 2 & 0 \\ 0 & -1 \end{bmatrix}
\end{aligned}
$$  

$$
\begin{aligned}
T = B\hat{T}B^{-1} & = \begin{bmatrix} 2 & 1 \\ 1 & -1 \end{bmatrix}\begin{bmatrix} 2 & 0 \\ 0 & -1 \end{bmatrix}\begin{bmatrix} \frac{1}{3} & \frac{1}{3} \\ \frac{1}{3} & -\frac{2}{3} \end{bmatrix} \\
T^{n-1} & = \begin{bmatrix} 2 & 1 \\ 1 & -1 \end{bmatrix}\begin{bmatrix} 2^{n-1} & 0 \\ 0 & (-1)^{n-1} \end{bmatrix}\begin{bmatrix} \frac{1}{3} & \frac{1}{3} \\ \frac{1}{3} & -\frac{2}{3} \end{bmatrix}
\end{aligned}
$$  

그런데 사실 $\hat{T}$를 직접 구할 필요는 없다.  
왜냐하면 선형변환 $T$와 그 고유값 $\lambda^d$로 이루어진 대각행렬의 선형변환은,  
Aigenvector로 이루어진 Aigenbasis $B$ 좌표계에서 동일한 선형변환이기 때문이다.  

그래서 구해진 $\hat{T}$가 우리가 구한 $\lambda_0, \lambda_1 = 2, -1$ 을 원소로 하는 대각행렬인 것이다.  
$$
\hat{T} = \begin{bmatrix} 2 & 0 \\ 0 & -1 \end{bmatrix} = \begin{bmatrix} \lambda_0 & 0 \\ 0 & \lambda_1 \end{bmatrix}
$$

최종적으로 $a_n$은 다음과 같다.  
$$
\begin{aligned}
\begin{bmatrix}
a_n \\
a_{n-1}
\end{bmatrix}
& =
\begin{bmatrix}
1 & 2 \\
1 & 0
\end{bmatrix}^{n-1}
\begin{bmatrix}
1 \\
0
\end{bmatrix} \\
& = \begin{bmatrix} 2 & 1 \\ 1 & -1 \end{bmatrix}\begin{bmatrix} 2^{n-1} & 0 \\ 0 & (-1)^{n-1} \end{bmatrix}\begin{bmatrix} \frac{1}{3} & \frac{1}{3} \\ \frac{1}{3} & -\frac{2}{3} \end{bmatrix}\begin{bmatrix} 1 \\ 0 \end{bmatrix} \\
& = \begin{bmatrix} 2 & 1 \\ 1 & -1 \end{bmatrix}\begin{bmatrix} 2^{n-1} & 0 \\ 0 & (-1)^{n-1} \end{bmatrix}\begin{bmatrix} \frac{1}{3} \\ \frac{1}{3} \end{bmatrix} \\
& = \begin{bmatrix} 2 & 1 \\ 1 & -1 \end{bmatrix}\begin{bmatrix} \frac{2^{n-1}}{3} \\ \frac{(-1)^{n-1}}{3} \end{bmatrix} \\
& = \begin{bmatrix} \frac{2^{n} - (-1)^{n}}{3} \\ \frac{2^{n-1} - (-1)^{n-1}}{3} \end{bmatrix} \\
\\
a_n = \frac{2^{n} - (-1)^{n}}{3} \\
\end{aligned}
$$
