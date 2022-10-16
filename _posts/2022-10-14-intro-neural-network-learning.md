---
layout: post
title:  "인공신경망과 학습의 기본원리"
date:   2022-10-14 14:46:00 +0900
categories: CS
comments: true
---
최근 핫한 AI.(30년 째 핫하다)  
인공신경망과 학습(경사하강법) 에 대해 간단히 설명 해보겠다.  
<br>

## 인공신경망(Artificial Nerual Network)
먼저 인간의 신경망을 본딴 인공신경망을 구축해보자.  

X와 Y라는 신경세포가 있다고 가정했을 때,  
X 세포가 Y 세포에게 신호를 보내는 과정을 생각해보자.  
![image](https://user-images.githubusercontent.com/44316628/195788279-b271fb9f-438b-4ff8-ad45-b78e3fc52047.png)  

X에서 Y로 신호를 어떻게 전달해야 할까.  
X가 가진 값을 그대로 Y에게 전달하면 그건 그냥 대입이 되어버린다.  
실제 신경망처럼 신호를 전달하려면 어떻게 해야할까?  
우리는 그래서 Weight(연결강도)와 Bias(편향)이란 것을 추가할 것이다.  

Weight는 X가 가진 값을 얼마나 Y에게 전달할 것인지 곱연산 할 값이다.  
Bias는 말 그대로 편향인데, X에서 weight를 거쳐 받은 값에 편향값을 더한다.(합연산)  
X1, X2가 Y에게 연결되어 있다면 이런느낌.  
![image](https://user-images.githubusercontent.com/44316628/195786104-5e3a1bf9-3b76-422d-90b6-48276f444844.png)  

이걸 많이 붙이면 신경망이 된다.  
![image](https://user-images.githubusercontent.com/44316628/195774770-770fdf78-1530-415b-ba27-b77cb1c8352d.png)  

근대 여기서 문제가 있다.  
우리가 만든건 **X * W + B** 인데, 이걸 여러개 붙여봤자 곱셈과 덧셈이 많이 붙은 선형적인 1차함수 아닌가?  
그래서 여기다 비선형 함수인 Activation Function(활성화 함수)라는 걸 추가해서 비선형적인 신경망을 만들 수 있다.  
대표적인건 Sigmoid나 ReLU 함수.  
![image](https://user-images.githubusercontent.com/44316628/195775588-0b6b8d82-adfa-48cc-852b-69be2308aada.png)  

이걸 아까 Bias처럼 붙인다.  
![image](https://user-images.githubusercontent.com/44316628/195775882-cd8dbf6d-d0c1-4bc7-8f68-1a6228dbf547.png)  

이러면 인공신경망 완성~  
간단하다.  
weight와 bias는 랜덤값으로 초기화 해두고, 
데이터를 넣으면 연산된 값이 나온다.  

그런데 당연히 이 연산된 값은 우리가 원한 어떤 예측값이 되질 못한다.  
우리는 이 신경망을 학습시켜야 한다.  
<br>

## 학습 (Gradient Descent)
이 신경망을 그래서 어떻게 학습시킬까?  
신경망 모델에 어떤 데이터를 넣었을 때, 랜덤한 값을 뱉는다면  
우리가 원하는 "정답 값"에 근접하도록 바꾸어주면 된다.  

학습방법은 여러가지가 있지만 경사하강법(Gradient Descent)를 사용하겠다.  
이게 뭐냐면 신경망의 "예측 값(O)" 과 "정답 값(A)" 의 오차를 **(O - A) ^ 2** 로 정의했을 때,  
이 오차함수의 기울기를 찾아내, 오차를 줄이는 방향으로 Weight와 Bias를 업데이트 하는것이다.  

![image](https://user-images.githubusercontent.com/44316628/196015360-bcd303be-6a53-4be6-9844-0d0cfe88ad83.png)  
맨처음 오차가 A 지점 이라고 생각해보자.  
여기서 각 Weight와 Bias들이 이 오차에 끼친 미분을 계산하여 오차가 낮은쪽으로 업데이트를 각각 한다.  
그러면 오차가 B 지점으로 이동하고, 거기서 한번 더 위 과정으로 학습을 시킨다면,  
C 지점으로 이동한다.  
(A 지점에선 기울기가 큼으로 많이 이동하고, B 지점에선 상대적으로 기울기가 작으므로 적게 이동한다.)  

이렇게 오차가 낮은쪽으로 계속 업데이트하는게 바로 "학습"이라 불리는 것이다.  
이 과정을 하다보면 모델의 예측 정확도가 올라가게 되는 것.  