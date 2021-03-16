---
title: "Edge Assisted Real-time Object Detection for Mobile Augmented Reality"
categories: [IMES, paper]
tags: [IMES, paper]
use_math: true
---

> 2021 / 3 / 18 IMES 세미나

## Abstract

- 대부분의 AR/MR : 주변 환경의 3D 형상을 이해할 수 있음
  - 단점 : 현실의 <u>복잡한 물체 탐지, 분류하는 능력 부족</u>
  - 해결 방법 : CNN에서 기능을 활성화할 수 있음
    - 단점 : mobile device에서 <u>대규모 network 실행은 어려움</u>
  - edge / cloud에 object detection을 offload하는 것도 어려움
    - 원인 : 높은 accuracy/낮은 E2E latency에 대한 엄격한 요구사항
  - 기존의 offloading 기술의 긴 latency : **detection accuracy를 상당히 낮출 수 있음**
    - 원인 : user view의 변화
- 높은 accuracy의 object detection를 할 수 있는 system 설계
  - 환경 : 60fps / 상용 AR/MR system
  - 특징
    - 낮은 latency의 offloading 기술 사용
    - rendering pipeline을 offload pipeline으로부터 분리
    - 빠른 object tracking method 사용
      - detection accuracy 유지
  - 결과
    - object detection / human keypoint detection task에서의 accuracy : 20.2% -> 34.8%
    - AR device에서의 object tracking의 latency : 2.24ms
  - 결론
    - 다음 frame에 대한 가상 element를 rendering하기 위한 <u>시간과 resource를 더 많이 남겨놓을 수 있음</u>
    - 더 높은 quality의 AR/MR 체험을 가능하게 함

## Introduction

- AR, Mixed Reality : 전례없는 몰입 경험 제공할 것 ( 교육 / 오락 / 의료 분야 등 )
  - 카메라를 통한 주변환경 이해
  - 사용자의 시야에 가상 오버레이 렌더링
- MR : system이 실제 다양한 object와 instance를 이해하고 렌더링해야함
  - **더 많은 computing resource가 필요**
  - 미래시장이 좋음 ( 2021년에 9900만 기기 / 1080억 달러 )
- 기존 모바일 AR solution ( ARKit / ARCore ) : 스마트폰에서의 표면 감지 / 객체 고정 기능 지원
  - 보다 정교한 AR 헤드셋 ( Microsoft HoloLens / announced Magic Leap One ) : 주변환경의 3D 형상 이해, 60fps에서의 가상 오버레이 렌더링 가능
  - 기존 AR 시스템 : 표면 감지는 가능 / **실제 환경에서의 복잡한 물체 감지, 분류 능력 부족**
    - 많고 새로운 AR / MR app에서의 필수요소
      - 자동차 운전자에게 위험요소 알려주는 것
      - 사람 위에 피카츄 올려놓는 것
- 위의 기능 : CNN과 함께 enable 가능 ( 객체탐지 task에서의 우수한 성능 )
  - 문제점 : 모바일 장치에서 latency가 적도록 대규모 네트워크 실행하는 것은 어려움
    - 예시 : tensorflow lite는 한 프레임에서 CNN 모델을 실행하는데 1초 이상 소요
- edge / cloud로의 객체탐지 offload는 어렵다 ( 높은 정확도, 적은 delay 등의 엄격한 요구사항이 원인 )
  - 좋은 품질의 AR 장치 : 객체를 성공적으로 분류해야함 + <u>높은 정확도로 객체를 현지화해야함</u>
  - delay가 100ms보다 작더라도 user view의 변화에 의해 정확도가 떨어짐 ( 탐지한 객체의 프레임 위치와 현재 프레임 위치랑 다를 수 있음 )
  - MR 그래픽이 VR의 복잡성에 근접 -> VR app에서 멀미를 일으키지 않을 정도의 delay ( 20ms ) 가 요구될 것
    - 기존 AR ( 간단한 annotation만을 렌더링 ) 과의 비교 : 더 많은 가상요소를 더 좋은 품질로 렌더링해야함 -> **객체 탐지 task에서의 latency budget이 적어짐**
- 기존 연구들 : 모바일 장치에서의 높은 프레임률 객체 탐지 기능 지원에 초점
  - **고품질 AR / MR에서의 latency 요구사항은 고려하지 않음**
    - Glimpse : 트리거 프레임을 클라우드 서버로 offload -> 30fps 객체탐지 달성 / 모바일 장치의 나머지 프레임에서의 bounding box를 로컬로 추적
    - DeepDecision : 네트워크 상황에 따라 객체탐지 task를 edge cloud로 보낼지 / local 추론을 실행할 것인지 결정하는 프레임워크 설계
    - 위의 방법 : 400ms 이상의 latency 필요 / 대량의 local 계산 필요 -> **고품질 가상 오버레이를 렌더링할 resource가 거의 남지 않음**
      - <u>움직이는 시나리오에서의 높은 감지 정확도 달성 / 전체 감지 및 렌더링 pipeline을 20ms 미만으로 완료할 수 없음</u>
- 위의 문제점을 달성하기 위해, 새로운 시스템 제안 ( **offload 감지 latency 크게 줄이고 기기 내에서 빠른 객체 추적 방법으로 나머지 latency를 숨김** )
  - offloading latency 줄이는 방법 : *Dynamic RoI Encoding* 기술 / *Parallel Streaming and Inference* 기술
    - Dynamic RoI Encoding : 마지막 offload된 프레임에서 검출된 *관심 영역*에 기초하여 전송 latency를 줄이기 위해 각 프레임의 encoding 품질을 조정
      - key innovation : 이전 프레임에서 후보 영역의 잠재적 관심 대상이 존재하는 영역을 식별하는 것
      - 객체가 탐지될 가능성이 높은 영역에서 더 높은 encoding 품질 제공 / 다른 영역에서는 더 강한 compression 실행 -> **bandwidth 절약 / latency 줄임**
    - Parallel Streaming and Inference : 스트리밍 / 추론 프로세스를 pipeline -> offloading latency 줄이는 목적
      - 검출 결과에 영향을 주지 않고 CNN 객체탐지 모델의 slice 기반 추론을 가능하게 하는 *Dependency Aware Inference* 방식 제안
      - AR 장치에서 렌더링 pipeline과 offloading pipeline을 분리 ( 모든 프레임에 대한 edge cloud로부터의 검출 결과를 기다리지 않음 )
        - 빠르고 가벼운 객체 탐지 방법 사용 ( encode된 비디오 프레임에서 추출되거나 edge cloud에서, 이전 프레임에서 처리된 프레임에서 캐시된 객체탐지 결과를 기반으로 한 모션벡터 기반 )
        - 모션이 있는 경우 현재 프레임에서의 bounding box나 key point를 조정
  - 낮은 offload 지연시간 -> **정확한 객체탐지 결과 제공 / AR이 고품질 가상 overlay를 렌더링할 시간과 resource 남겨둘 수 있음**
    - *Adaptive Offloading* 기술 도입 : 이전의 offload 프레임과 비교하여 현재 프레임의 변경사항을 기반으로, 각 프레임을 edge cloud로 offload할지 결정 -> bandwidth / energy 효율적
- 이 시스템은 현재 AR/MR 시스템, 60fps 환경 ( 객체 감지 / 인간 keypoint 감지 task ) 에서 높은 정확도의 객체탐지를 이루어냄
  - 상용 AR기기에서 E2E AR 플랫폼 구축 ( 평가목적 )
    - 정확도 향상 ( 20.2% ~ 34.8% )
    - 탐지 오류 감소 ( 27.0% ~ 38.2% )
    - latency : 2.24ms ( 매우 짧음 ) / AR 장치의 15% 미만의 resource 사용 => 프레임 사이의 여유시간이 남음
      - **고품질 AR/MR 환경을 위한 고품질 가상 element를 렌더링 할 시간이 있음**
- 기여
  - 객체탐지 task offload의 상태에서, E2E AR 시스템에서 정확도와 latency 필요를 수량화
  - 개별 렌더링 / offload pipeline이 포함된 프레임워크 제안
  - *Dynamic RoI Encoding* 기술 설계 : 관심영역을 동적으로 결정하여 offload pipeline 전송 latency / bandwidth 사용을 줄임
  - *Parallel Streaming and Inference* 메소드 개발 : 스트리밍 / 추론 프로세스를 pipeline하여 offload latency를 더욱 줄임
  - *Motion Vector Based Object Tracking* 기술 개발 : 인코딩된 비디오 스트림에서의 내장 motion 벡터에 기반하여 AR장치에서 빠르고 가벼운 객체 추적 달성
  - 상용 하드웨어에 E2E system 구현 / 평가 : 제안된 시스템이 정확한 객체탐지를 통해 60fps AR experience 실현할 수 있음을 보임

## Challenges and Analysis

- 모바일에서의 정교한 객체탐지는 어려움 ( 모바일에서 계산을 모두 돌리기 힘듬 / bandwidth 사용이 너무 많어 edge로 offload할 수 없음 )
  - 간단한 객체탐지 : GPU 사용하는 모바일에서도 500ms의 processing time 소요
  - 더 좋은 GPU를 사용하더라도 HD frame에서 50ms의 delay 소요
  - 초래하는 문제점
    - 60Hz의 모든 frame을 process할 수 없음
    - 에너지 소비 문제
    - 방열 문제

##### latency analysis

- 객체탐지를 더 강한 edge나 cloud로 offload하는 것은 상당한 latency를 추가함

  - 탐지 정확도 감소 / AR experience 저하 초래

- 전체 latency 모델 생성
  $$
  t_{e2e} = t_{offload}+t_{render}\\
  t_{offload}=t_{stream}+t_{infer}+t_{trans\_back}\\
  t_{stream}=t_{encode}+t_{trans}+t_{decode}
  $$

  - 과정
    - AR device -> Edge : frame $n$개를 capture
    - Edge : $n$개의 frame을 받고, 그것을 inference
    - Edge -> AR device : 결과 전송
    - AR device : screen에 결과를 렌더링
  - $t_{stream}$을 줄이는 방법 : 몇개의 raw frame을 압축 ( H.264 video )

- 기존 AR system으로는 1280x720 resolution / 60fps에서의 높은 객체탐지 정확도를 실현하는 것은 어려움

##### detection accuracy metrics

- 물체 분류 / localization에서의 객체 정확도 평가 목적 : 각 경계 상자의 IoU, ground truth를 정확도 지표로써 계산
  - IoU : 교집합 영역 넓이 / 합집합 영역 넓이 ( bounding box의 감지 비율이 0.75 미만이라면 실패했다고 판단 )
- **적은 latency의 객체탐지는 높은 정확도와 직결됨**
  - $t_{offload}의 높은 latency가 정확도가 낮아지는 원인이 됨 ( user view의 변화 / user motion의 변화 / scene motion의 변화 )
- 상용 인프라에서 객체탐지 latency를 낮추는 것이 어려움
  - 모든 backbone CNN 네트워크에서 최소 10ms의 객체탐지 latency가 요구됨
- 영상의 bitrate가 높아질수록 offload할 latency가 높아짐
  - bitrate를 낮추면 frame의 손실이 발생
- $t_{infer} + t_{trans}$ latency가 한 frame의 display time을 넘어섬
  - 인코딩 bitrate의 resolution을 낮추면 latency는 낮출 수 있지만 정확도가 감소
  - 적어도 90%의 정확도를 위해서는 50Mbps의 인코딩 bitrate가 필요
  - 해상도를 낮추는 것 또한 정확도 감소
- **60fps 기존 AR 시스템에서 높은 정확도를 나타내는 것은 어렵다**
  - 품질이 좋지 않은 객체를 렌더링하는 결과를 초래할 것

## System Architecture

- 한계 돌파하기 위해, AR 플랫폼에서 <u>높은 정확도의 객체탐지 + 적은 랜더링 오버헤드 시스템 고안</u>
  - low latency offloading 기술로 detection latency를 낮춤
  - on-device fast object tracking method로 잔여 latency를 숨김
- 시스템 구조 : 2개의 시스템의 무선 연결구조
  - local tracking + rendering system ( 모바일 )
  - pipelined object detection system ( edge )
- 객체 탐지 task에서의 latency를 줄이기 위해 rendering process / CNN offloading process를 2개의 pipeline으로 분리
  - local rendering pipeline은 scene을 추적하고 가상 overlay 렌더링 시작
  - 결과를 받으면 탐지 결과를 포함시킴
- 2개의 pipeline에서 *Dynamic RoI Encoding technique* 기술 사용
  - 2가지 기능
    - CNN offloading pipeline : raw frame을 압축
    - tracking and rendering pipeline : on-device tracking 모듈에 메타데이터 제공
  - **탐지 정확도는 유지하면서 edge cloud의 bandwidth 소모 / 전송 latency 줄일 수 있는 기술**
  - 이전의 탐지 결과에 따라 <u>가능성이 없는 frame 부분의 인코딩 품질을 낮게, 가능성이 높은 부분의 인코딩 품질은 유지하는 기법</u>
  - 이후의 video frame에 대한 시공간 상관관계 존재 -> 마지막에 offload된 frame의 intermediate inference 결과를 후보 영역으로 사용
    - 높은 인코딩 품질을 유지 / RoI로써 참고되기도 함
- CNN offloading pipeline에서 *Adaptive Offloading* + *Parallel Streaming and Inference* 기술 사용
  - *Adaptive Offloading* 기술 : 이전 frame과의 변화를 기준으로 <u>현재 frame을 edge로 offload할 것인지를 결정하는 기술</u>
    - system의 bandwidth와 power 소모를 줄이는 기술
    - macroblock 타입을 재사용하여 변화를 감지
  - *Parallel Streaming and Inference* 실행 : frame이 offload되어야한다고 mark되면 <u>전송, decode, inference를 병렬로 실행하는 method</u>
    - 모든 frame을 전송받기를 기다리지 않고, 1개의 slice ( frame을 분할한 것 ) 가 들어올 때마다 CNN 객체탐지 task 수행
    - **수신 / decode / 객체탐지 기능이 병렬로 실행될 수 있음**
    - slice 간의 종속성 문제 존재 -> *Dependency Aware Inference* 매커니즘 고안
      - 각 slice의 수신 이후 계산할 수 있는 충분한 input feature를 가진 각 feature map 상에서 region 결정
      - 결정된 region에 존재하는 feature만 계산
  - 계산된 결과는 AR 기기에 return 
    - cache되어 나중에 사용할 수도 있음
- tracking and rendering pipeline에서 *Motion vector based Object Tracking* 기술 사용
  - fast / light-weight 기술
  - viewer / scene의 motion과 이전에 캐시해둔 detection 결과를 조정하기 위함
  - 인코딩된 video frame에 내장된 motion vector 활용하여 추가 processing overhead 없이 객체탐지를 가능하게 함
    - 기존 객체탐지 기술 : 2개의 frame에서 match되는 image feature point를 찾는 방법 사용
  - 더 적은 frame 시간으로도 tracking 가능 / 상당한 정확도 제공 가능
  - device에서 렌더링할 시간과 연산 resource를 남겨놓을 수 있음 ( 16.7ms 미만 )

## Dynamic RoI Encoding

- 객체탐지 정확도는 유지하면서 offloading pipeline의 transmission latency를 줄이는 기법
  - 높은 품질의 frame을 전송하는 것은 큰 bandwidth 소모 발생 -> latency로 이어짐
- 관심 영역에 따라 높은 degree의 frame 압축을 할 것인지 결정
  - 가능성이 없다면 압축하여 frame의 크기를 줄임
  - 가능성이 높다면 높은 품질로 유지
- 객체탐지 정확도 향상에 도움이 됨
- 핵심 : 관심영역 ( RoI ) region을 식별하는 것
  - 이전 frame을 CNN으로 돌려서 생성된 후보영역을 사용
- 기존 RoI 인코딩 기법을 사용 + 새로운 기술을 추가하여 **각 frame의 RoI를 동적으로 결정**

### 1. Preliminaries

##### RoI encoding

- RoI encoding을 만드는 것은 다른 app들에서도 사용되고 있음
  - region을 선택하는 현재 method는 AR 객체탐지 task에 적합하지 않음
- 이미 대부분의 video 인코딩 플랫폼에서 지원됨
  - user가 frame의 macroblock마다 인코딩 품질을 조정할 수 있음
- 선택된 region은 손실이 거의 없는 압축 ( 품질 유지 )
  - 선택되지 못한 region ( 배경 등 ) 은 손실이 있는 압축
- RoI는 사용자의 초점에 맞는 현재 object에 기초할 수 없음

##### object detection CNNs

- 존재하는 다른 네트워크처럼 비슷한 구조를 공유함
  - CNN 네트워크를 이용하여 input image로부터 feature를 추출
  - region 제안 네트워크를 통해 RoI와 가능성을 내부적으로 제안
  - 객체 분류를 수행 / 정제
- CNN 네트워크를 backbone 네트워크라고도 부름
- 대체로 frame에서 가능성이 있는 영역으로 수백개의 region을 생성함

### 2. Design

- bandwidth 소모와 전송 latency를 줄이는 것이 목적
- interal RoI를 image incoder와 link하는 기술
- 이전 frame에서 생성된 candidate RoI를 사용하여 다음 camera frame에서의 인코딩 품질 결정
- 하나의 macroblock으로 RoI를 확장 -> motion의 degree를 약간 수용 / 2개의 frame에 대한 유사성에 대한 이점을 얻을 수 있음
- **object가 발견된 region만을 RoI로 지정하면 안됨**
  - 새로운 object가 나타날 수 있는 region이 심하게 압축되버릴 수 있음
  - 그래도 RoI의 일부로 종종 확인되며, region 제안 네트워크의 출력으로 나오기도 함
    - <u>최소 예측 신뢰 임계값 0.02로 필터링하여 RoI를 사용</u>
- 현재 frame의 인코딩 품질을 조정하기 위해 선택된 RoI 사용 // QP map 계산
  - QP : ( Quantization Parameter ) : 인코딩 품질
  - QP map : frame의 각 macroblock에 대해 인코딩 품질 정의
    - macroblock이 다른 RoI와 겹치는가를 표시
    - 결과를 AR device에 반환 // **이것을 다음 frame의 RoI로 사용**
      - RoI는 품질 유지, 이외의 범위는 손실압축

## Parallel Streaming and Inference

- 무거운 DNN 연산은 edge cloud로 옮겨 계산 -> 카메라 frame을 전송해야하는 필요성
  - conventional 구조 특성 : <u>모든 frame이 도착해야 객체탐지 process 실행 가능</u>
    - deep 신경망이 이웃간의 종속성에 의해 설계되기 때문
  - streaming과 inference 모두에게 상당한 latency가 발생함
- *Parallel Streaming and Inference* 기술 도입 : 각 frame마다 inference를 수행할 수 있게 함
  - **streaming과 inference가 효율적으로 pipelined되고 병렬적으로 수행됨**
  - 가능한 이유 : <u>각 단계마다 사용하는 resource가 다름</u>
    - 전송 : 무선 link의 bandwidth
    - decode : edge cloud의 하드웨어 decoders
    - 네트워크 inference : GPU / FPGA resource
  - frame마다 실행에서의 challenge : input 간의 종속성
    - 이웃 value를 input으로써 받는 신경만 operation에 의함
    - **Dependency Aware Infefence** 기술 제안 : 각 layer의 종속성을 분석하고, 충분한 이웃 value가 있는 region만을 infer
  - 전체 image를 slice로 분할하여 각기 전송하고, 특정 단계가 실행 가능하다면 실행하면서 offloading latency를 낮춤

### 1. Dependency Aware Inference

- 단순히 slice 기반으로 inference한 이후 merge를 수행하면 경계지점에서 잘못된 결과를 초래할 수 있음

- *Dependency Aware Inference* 기술 도입 : input feature point의 양이 충분한 region의 feature point만을 계산하는 기법

  - 종속성 : convolutional layer에 의해 발생 // 각 frame slice의 경계를 둘러싼 feature 계산에도 인접한 slice가 필요
  - 경계 feature 연산 : last convolution layer에서 추가적인 pixel이 필요함

- 병렬로 inference하는 방법 : 다음 slice를 받았을 때 특정 region을 재연산

  - 추가 연산 소요 // inference latency 팽창

- 종속성 문제 해결법 : 각 layer의 output feature map에 대한 *valid region*의 크기를 먼저 계산하고, 그것에 기반하여 infer

  - valid region : 가능한 input feature이 출분한 각 convolutional feature map의 영역으로 정의

    - 크기
      $$
      H_{i}^{out}=(H_{i}^{in}-1)/S+1\\
      W_{i}^{out}=
      \begin{cases}
      \frac{W_{i}^{in}-(F-1)/2-1}{S}+1, &\text{i=1,2,...,n-1}\\
      \frac{W_{i}^{in}-1}{S}+1, &{i=n}
      \end{cases}
      $$

      - $H_{i}^{out}$, $W_{i}^{out}$ : edge cloud에 $i$ slice가 도착했을 때, convolution layer에서의 outpur feather map의 <u>valid region의 높이 / 넓이</u>
      - $F$, $S$ : convolution 공간 범위 / 보폭

- 컨셉

  - 1개의 frame을 4개로 분할함 ( $n=4$ )
    - width는 전체 frame의 1/4, height는 고정
    - $H_{i}^{out}$ : 상수 취급 // $H_{i}^{in}$, $S$의 값에만 의해 영향을 받음
    - $W_{i}^{out}$ : slice가 도착할 때마다 값이 커짐
  - slice의 가장 오른쪽 열은 계산하지 않음 ( 이후 slice에 의해 input feature의 종속성이 생길 수 있음 )
    - 더 많은 slice가 들어올수록 vaild region이 계속해서 증가하기 때문
    - slice가 들어올수록 **기존에 들어온 slice에서의 input feature가 감소** -> 더 적은 연산만을 필요로함
  - 모든 slice가 들어온 이후에는 남아있는 모든 input feature를 연산

## Motion Vectors based Object Tracking

- 인코딩된 video frame에서의 motion vector \+ 이전에 offload된 frame에서 캐시된 객체탐지 결과 => **현재 frame에서의 객체탐지 결과 추정치**
- 모션 벡터 : video의 높은 압축률을 달성하기 위해 **frame 간의 pixel offset을 표현하는 방법**
- 과정
  - 새로운 frame이 캡처되면 *Dynamic RoI Encoding* 단계로 전송
    - encoder : frame 간의 압축을 위해 마지막에 cache된 탐지 결과에 대응하는 frame 사용
  - system : 인코딩된 frame에서 모든 motion vector 추출
  - 현재 frame에서의 객체를 추적하기 위해, bounding box를 motion vector의 평균치만큼 이동시킴
- human keypoint 추적에도 이 방식을 비슷하게 적용할 수 있었음
- 레퍼런스 frame과 현재 frame의 시간 간격이 길어질수록 정확도가 떨어짐
  - latency를 매우 낮추었기 때문에, 정확도 있는 객체탐지 결과를 보여줄 수 있었음
- latency를 줄였다 -> **AR기기가 가상 오버레이를 높은 품질로 렌더링할 수 있다** ( 충분한 시간과 자원을 남길 수 있기 때문 )

## Adaptive Offloading

- 효율적으로 offloading pipeline을 scheduling하기 위함
  - 인코딩된 frame을 edge로 보내야하는가?
- 2가지 원리에 의해 작동함
  - 이전 frame이 edge cloud에 의해 완전히 수신되었을 경우에만 frame을 offload할 수 있음
    - 네트워크 혼잡을 피하기 위해 frame queueing을 없앰
    - 실현하기 위해서는 이전 frame의 transmission latency를 계산해야함
      - edge -> AR device로 slice를 받았다는 신호를 발송
      - reception 시간과 transmission 시간의 차이를 이용하여 latency 계산
      - 이것을 이용하여 다음 frame을 offload할 것인지 결정
  - 현재 frame이 이전 frame과 상당히 다른 점이 있을 때에만 frame을 offload함
    - 소통 / 연산 cost를 최소화하기 위해 변화가 상당히 있는 필요한 view만을 offload함
    - 실현하기 위해서는 두 frame 간의 차이를 계산해야함
      - 2가지 원칙을 충족시키는 2가지 관점으로 평가 ( 둘 중 하나만 만족해도 된다? )
        - frame 사이에서 큰 motion이 발생했는지 ( user / object의 motion 모두 포함 )
        - frame에 나타나는 상당한 양의 변화된 pixel이 있는지
      - frame의 motion : 모든 motion vector의 합으로 정량
      - new pixel의 수 : 인코딩된 frame의 intra-predicted macroblock 수로 정량
    - 인코딩된 frame에서의 2개의 type의 macroblock 사이에서, **intra-predicted block은 새로 나타난 region을 레퍼런스로 잡는 경향이 있음**
      - macroblock이 인코딩되는 동안 레퍼런스 frame에서 레퍼런스 pixel block을 찾지 못함
  - 2가지 원리를 만족해야만 edge cloud로 frame을 offload

## Implementation

- 



