---
layout: post
title:  "GPU로의 데이터 전송 (작성 중)"
date:   2023-05-13 23:13:00 +0900
categories: CS
comments: true
---
드라이버가 GPU의 VRAM에 매핑되는 non-pageable memory 를 제공함
여기다가 보낼 데이터 복사.
read 할 일 없으니 write_combined 속성으로 최적화.
이제 전송을 할 건데 CPU가 이걸 하나하나 쏘고 있을 수 없으니 GPU쪽에서 DMA로 pcie 통해서 쭈아아압 가져감.
원래 이게 정석인데 이번에 또 판도가 뒤집힘
https://quasarzone.com/bbs/qn_hardware/views/1448743
Memory mapped i/o
옛날 cpu랑 주변장치 클럭이 비슷할 땐 유용했음
cpu가 빨라지면서 cpu가 대기해야 할 clock이 늘어나 aync i/o가 대세
하지만 이젠 또 PCIE 버젼이 오르면서 cpu의 대기가 줄어들었으므로
Mem-mapped io로 다시 감
