# Jupyter-Notebook_-GPS-Spoofing-and-Jamming-Detection-in-Drone-Flight-Using-Machine-Learning

# 드론 공격 탐지 모델 개발 

## 개요
이 프로젝트는 드론의 GPS 재밍(Jamming)과 스푸핑(Spoofing) 공격을 탐지하는 머신러닝 모델을 개발하는 방법을 보여줍니다. 드론 비행 중 수집된 다양한 GPS 관련 데이터를 사용하여 드론 공격을 탐지하고, 이를 "공격"과 "비공격"으로 분류한다.

## 데이터셋
데이터셋은 세 가지 다른 조건에서 수집된 드론 비행 데이터를 포함하고 있다:
1. **정상 비행 (Benign Flight)**: 공격이 없는 정상 비행
2. **GPS 재밍 (GPS Jamming)**: GPS 재밍 공격이 발생한 데이터
3. **GPS 스푸핑 (GPS Spoofing)**: GPS 스푸핑 공격이 발생한 데이터

### 주요 특징:
- **hpos_drift_rate**: 수평 위치 드리프트 비율
- **vpos_drift_rate**: 수직 위치 드리프트 비율
- **hspd**: 수평 속도
- **eph**: 수평 정확도
- **epv**: 수직 정확도
- **hdop**: 수평 정확도 감소 비율
- **vdop**: 수직 정확도 감소 비율
- **jamming_indicator**: 재밍 여부 표시
- **blocked**: GPS 신호 차단 여부

## 방법론

### 1. 데이터 전처리
먼저, 데이터셋을 불러와서 필요한 특징들(`hpos_drift_rate`, `vpos_drift_rate`, `hspd`)을 선택하고, 공격 라벨(0: 정상 비행, 1: 재밍, 2: 스푸핑)을 추가한다.

### 2. 데이터 정규화
모델 학습을 위한 데이터 정규화가 진행됩니다. **MinMax Scaling** 기법을 사용하여 모든 데이터의 범위를 0과 1 사이로 변환한다.

##결과
모델 정확도: 99.85%
Confusion Matrix:
True Positive (TP): 정상 데이터가 공격으로 잘못 분류된 경우 (1345)
False Positive (FP): 정상 데이터를 공격으로 예측한 경우 (6)
True Negative (TN): 정상 데이터를 정상으로 예측한 경우 (0)
False Negative (FN): 공격 데이터를 정상으로 예측한 경우 (2)

![image](https://github.com/user-attachments/assets/2f1ec06a-c44a-4a27-8939-38ecbf2b2994)



![image](https://github.com/user-attachments/assets/ca5cc534-430c-42d0-8d99-0d80fde63837)

