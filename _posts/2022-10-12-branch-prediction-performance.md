---
layout: post
title:  "분기예측확률에 따른 성능향상"
date:   2022-10-12 19:22:00 +0900
categories: CS
comments: true
---
갑자기 분기예측이 데이터 비율(분기 확률(?))에 따라 얼마만큼 성능향상이 나오는지 궁금해서 테스트 해봤다.  

[**[테스트 코드(github)]**](https://github.com/Ria9993/PlayGround/blob/main/Branch-prediction%20performance%20test/Branch-prediction%20performance%20test/main.cpp)
![image](https://user-images.githubusercontent.com/44316628/195322282-598083d7-2a44-47a2-ac61-39d21ad05325.png)  

현대 프로세서의 평균 분기예측률이 90% 정도 되므로 실행속도가 40% 정도까지 줄어듬  