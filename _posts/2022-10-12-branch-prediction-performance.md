---
layout: post
title:  "분기 예측률에 따른 성능향상 테스트"
date:   2022-10-12 19:22:00 +0900
categories: CS
comments: true
---
갑자기 분기예측이 데이터 비율(분기 확률(?))에 따라 얼마만큼 성능향상이 나오는지 궁금해서 테스트 해봤다.  

arr에는 0 혹은 랜덤 수가 들어있다.  
이 0과 랜덤 값의 비율을 5:5로 한다면 분기 예측률은 50%에 수렴할 것이고  
10:0 으로 한다면 분기 예측률은 100%에 수렴할 것이다.  

분기 없애는 걸 방지하기 위해 Debug로 컴파일.  
```c
    for (int i = 0; i < ITER_NUM; i++)
    {
        if (arr[i] != 0)
            sum += arr[i];
        else
            sum += 1;
    }
```
[**[테스트 코드(github)]**](https://github.com/Ria9993/PlayGround/blob/main/Branch-prediction%20performance%20test/Branch-prediction%20performance%20test/main.cpp)  
![image](https://user-images.githubusercontent.com/44316628/195467772-84d61930-e333-4ed3-ba57-0ec93d0d1b5c.png)

현대 프로세서의 평균 분기예측률이 90% 정도 되므로 실행속도가 기존의 40% 정도까지 줄어듬  
실행할 때마다 확률이 튀긴 하지만 거의 선형적으로 줄어드는 모습  

연산이 간단하니 좀 더 연산이 복잡한 경우엔?  
연산을 좀 더 복잡하게 변경  
```c
    for (int i = 0; i < ITER_NUM - 2; i++)
    {
        if (arr[i] != 0)
            sum += (arr[i] * 5) + (arr[i] % 5) 
                + (arr[i + 1] * 5) + (arr[i + 1] % 5)
                + (arr[i + 2] * 5) + (arr[i + 2] % 5);
        else
            sum -= (arr[i] * 7) + (arr[i] % 7) 
                + (arr[i + 1] * 7) + (arr[i + 1] % 7)
                + (arr[i + 2] * 7) + (arr[i + 2] % 7);;
    }
```

![image](https://user-images.githubusercontent.com/44316628/195467465-0e7e6248-715f-4259-af4d-d5bdea01c269.png)  
이번엔 실행속도가 기존의 70% 정도까지만 줄어듬  

## 여담  
여담인데, Branch prediction 할 때 과거 히스토리를 어디다 저장할까 싶었는데  
따로 테이블 만들어놓고 명령어 주소 하위비트로 직접사상(Direct Mapping) 해서 쓴다 카더라  