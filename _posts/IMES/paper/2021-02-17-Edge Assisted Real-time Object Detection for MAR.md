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
