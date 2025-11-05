# Jupyter-Notebook_— GPS Spoofing & Jamming Detection in Drone Flight (ML)

# 드론 공격 탐지 모델 개발 

## 개요
GPS 공격을 실기체 환경에서 재현(HackRF)하고, 수집된 PX4 비행 로그(.ulg) 기반으로 탐지 모델(1D-CNN/XGBoost)을 학습·검증·경량화(TFLM)하여 온보드 실시간 탐지까지 이어지는 전 과정을 정리한다.
스푸핑 신호는 해당 날짜의 RINEX 방송 천체력(에페머리스) 를 사용해 GPS-SDR-SIM으로 생성 후 SDR로 송출

## 주요 기여(Contributions)
현장 재현 데이터: HackRF One + RINEX nav 데이터로 GPS 스푸핑 신호를 합성·송출, PortaPack 기반 재밍 신호를 가변 파라미터로 송출 → 실제 비행 로그 수집 
통합 피처 설계: hpos_drift_rate, vpos_drift_rate, hspd, eph/epv, hdop/vdop, satellites_used, jamming_indicator, blocked 등 GNSS 품질·동역학 융합
모델 이중화: 시계열 패턴용 1D-CNN + 저연산 고속 XGBoost(규모/환경에 맞춰 선택), TFLM 양자화로 MCU 온보드 추론
재현·분석 파이프라인 공개: .ulg → csv 변환, 특징 엔지니어링, 학습/검증/평가, 혼동행렬·ROC, 온보드 배포 스텁까지 재현성 제공 
GitHub
MAVLink 이상 행위 규칙 세트 포함: HEARTBEAT·PARAM_SET 등 메시지 빈도 상한/변조 감시(플러딩·설정값 위변조) — 운영 중 복합 위협 조기 경보

## 데이터셋
직접 PX4 쿼드로터 기체에서 수집한 데이터셋은 세 가지 다른 조건에서 수집된 드론 비행 데이터를 포함하고 있다:
1. **Benign Flight**: 공격이 없는 정상 비행
2. **GPS Jamming**: GPS 재밍 공격이 발생한 데이터
3. **GPS Spoofing**: GPS 스푸핑 공격이 발생한 데이터

### 특징:
- **hpos_drift_rate**: 수평 위치 드리프트 비율
- **vpos_drift_rate**: 수직 위치 드리프트 비율
- **hspd**: 수평 속도
- **eph**: 수평 정확도
- **epv**: 수직 정확도
- **hdop**: 수평 정확도 감소 비율
- **vdop**: 수직 정확도 감소 비율
- **jamming_indicator**: 재밍 여부 표시
- **blocked**: GPS 신호 차단 여부

## 수집·라벨링 요약
장비: HackRF One(+PortaPack), PX4 쿼드콥터, GCS(QGC)
환경: 실외 평지, Manual/Waypoint 혼합 임무
라벨링 기준 예시
Spoofing: 스푸핑 송출 구간 + GNSS 품질·관성 불일치 임계 초과
Jamming: 수신 위성 급감, fix_type 하락, eph/epv 급증, jamming_indicator 상승
Benign: 위 조건 미해당·품질 지표 안정 구간

## 공격 재현 상세
1) GPS 스푸핑
데이터 준비: 해당 날짜의 RINEX 방송 천체력(brdc*) 다운로드 → GPS-SDR-SIM 입력으로 신호 합성 
송출: HackRF One로 L1 RF 송출(출력/샘플링/오프셋 튜닝) → 드론 GNSS가 조작 좌표로 수렴(위성 수·Fix는 정상처럼 보일 수 있음)

2) GPS 재밍
대역: L1 1575.42 MHz 중심(±Δf 설정)
모드: Random CW/Noise, 홉 레이트·dwell 가변
관찰 지표: fix_type(→0/2), satellites_used(↓), eph/epv(↑), jamming_indicator(↑)
재현 방법은 RF 법령을 철저히 준수하는 차폐 환경/시험용 시설에서만 수행

방법론
1) 전처리
누락·비정상 값 정리, 동기화(센서 시계열 align), 이동 윈도우(예: 2–5 s)로 클립/세그먼트화
MinMaxScaling 또는 RobustScaling 적용(학습/검증 누수 방지: split 후 fit)

2) 모델
1D-CNN: 시계열 특성 학습, 정수 양자화(INT8)로 TFLM에 탑재
XGBoost: 경량·고속 분류기(실시간 임계 판단)

온보드 추론: TensorFlow Lite for Microcontrollers/LiteRT for μC 사용(수 kB–수 백 kB RAM에도 구동)

3) 학습·검증
train/val/test = 7/1/2 (비행 세션 단위 Group Split 권장)
클래스 불균형: 시간 가중/샘플 가중 또는 focal loss
평가 지표: Acc/Macro-F1/ROC-AUC, per-class 재현율(스푸핑·재밍 민감도)

##결과
모델 정확도: 99.85%
Confusion Matrix:
True Positive (TP): 정상 데이터가 공격으로 잘못 분류된 경우 (1345)
False Positive (FP): 정상 데이터를 공격으로 예측한 경우 (6)
True Negative (TN): 정상 데이터를 정상으로 예측한 경우 (0)
False Negative (FN): 공격 데이터를 정상으로 예측한 경우 (2)

![image](https://github.com/user-attachments/assets/2f1ec06a-c44a-4a27-8939-38ecbf2b2994)



![image](https://github.com/user-attachments/assets/ca5cc534-430c-42d0-8d99-0d80fde63837)

## 실험/장비
SDR: HackRF One (Great Scott Gadgets) 
신호 합성: GPS-SDR-SIM (RINEX nav 사용) 
에페머리스: NASA/EGI/CDDIS RINEX(일일 brdc* 파일) 
비행 로그: PX4 ULog (pyulog/ulog2csv) 
온보드 추론: TensorFlow Lite for Microcontrollers / LiteRT for μC 
GCS: QGroundControl (MAVLink Heartbeat 기본 1 Hz 운용) 
