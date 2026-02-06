# 🍴 LG Aimers 7기 - 식음업장 메뉴별 수요 예측 해커톤

> 각 메뉴에 대해 향후 7일간의 일별 매출 수량을 예측하여 재고 관리 최적화 및 폐기 손실 최소화

## 📋 프로젝트 개요

| 항목 | 내용 |
|------|------|
| **주제** | LG Aimers 7기 - 식음업장 메뉴별 매출 수량 예측 |
| **기간** | 2025.07 ~ 2025.08 |
| **팀 구성** | 5인 |
| **담당 역할** | 데이터 전처리, Feature Engineering, '0 판매량 예측' 고도화 |
| **성과** | Public 173rd / Private 168th of 817 teams |

## 📊 데이터 개요

- **102,676건** 레코드 (2023.01.01 ~ 2024.06.15)
- **9개 업장**, **193개 메뉴**
- **52.6%** Zero-Inflated 분포 (54,034건이 0 판매)

---

## 🔄 프로젝트 파이프라인

```
데이터 전처리 → EDA (6개 분석) → Feature Engineering → 모델링 → 평가/검증
```

### 1️⃣ 데이터 전처리

#### 1.1 기본 전처리
- 영업장명/메뉴명 분리 (정규표현식 기반)
- 날짜 형식 변환 및 시간 관련 변수 추출
- 결측치 처리 (median → ffill → 0 전략)
- 음수 매출 처리: 환불/취소 추정 데이터(최소 -80) → `clip(0)` 적용

#### 1.2 시간적 특성 추출
- **주기성 피처**: 월별, 요일별, 연중 일자의 sin/cos 변환
- **휴일 피처**: 한국 공휴일, 공휴일 전날, 주말 여부
- **월 관련 피처**: 월 시작/끝 여부, 월말까지 남은 일수, 월의 몇 주차

#### 주요 도전과제: 순환 인코딩

**문제**: 단순 숫자형 시간 피처(월=1~12)는 순환 특성을 반영하지 못함 (12월↔1월이 숫자상 가장 먼 거리)

**해결**: Sin/Cos 변환을 통한 순환 인코딩
```python
df['month_sin'] = np.sin(2 * np.pi * df['month'] / 12)
df['month_cos'] = np.cos(2 * np.pi * df['month'] / 12)
df['weekday_sin'] = np.sin(2 * np.pi * df['weekday'] / 7)
df['weekday_cos'] = np.cos(2 * np.pi * df['weekday'] / 7)
```

---

### 2️⃣ EDA (탐색적 데이터 분석)

데이터 전처리 이후, 본격적인 피처 엔지니어링에 앞서 데이터의 구조와 패턴을 심층 탐색하여 분석 전략의 방향성을 수립하였습니다.

#### EDA 1: 업장별 매출 분포 분석

| 업장 | 일평균 매출 | 표준편차 | 메뉴 수 | 0 판매 비율 |
|------|------------|---------|---------|------------|
| 포레스트릿 | 574 | 1,022 | 12 | 42% |
| 카페테리아 | 453 | 498 | 24 | 44% |
| 화담숲주막 | 275 | 419 | 8 | 47% |
| 담하 | 235 | 129 | 42 | 47% |
| 미라시아 | 184 | 106 | 31 | 52% |
| 느티나무 셀프BBQ | 131 | 164 | 23 | 60% |
| 화담숲카페 | 118 | 163 | 5 | 47% |
| 연회장 | 53 | 75 | 23 | 66% |
| 라그로타 | 33 | 24 | 25 | 60% |

**핵심 발견**: 업장 간 일평균 매출 차이가 약 **17배**(574 vs 33) → 업장별 차등 가중치 전략 필요

#### EDA 2: 시간적 패턴 분석

- **요일 효과**: 토요일(2,948) > 월요일(1,498) → 주말 매출 **+56%**
- **월별 계절성**: 1월(4,177) 피크, 3월(510) 최저 → 겨울 성수기 & 10월 가을 성수기
- **공휴일 효과**: 평일 대비 **+87%** (3,242 vs 1,735), 공휴일 전날도 +17%

#### EDA 3: Zero-Inflated 분포 분석

- 전체 데이터의 **52.6%**가 0 판매 → 전형적인 Zero-Inflated 분포
- 업장별 편차: 연회장(66%) ~ 포레스트릿(42%)
- 연속 0 판매: 평균 4.5일, 7일 이상 연속 0인 경우 전체의 8.6%

| 0 판매 유형 | 특징 | 대응 피처 |
|------------|------|----------|
| 간헐적 판매 | 주 1-2회만 판매 | `zero_ratio_7d`, `days_since_nonzero` |
| 장기 미판매 | 7일+ 연속 0 | `zero_streak`, `is_chronic_zero` |
| 계절적 중단 | 특정 시기 미판매 | `month_sin/cos` + `zero_streak` |

#### EDA 4: 시계열 자기상관 분석

- **ACF 분석**: Lag 1(0.40), Lag 7(0.42) → 전일 효과 + **7일 주기 패턴** 확인
- **업장 단위**: 포레스트릿 Lag1=0.85, 카페테리아 Lag1=0.84, 담하 Lag1=0.57
- **STL 분해**: 추세(Trend) + 계절성(Seasonal) + 잔차(Residual) 성분 확인

```python
# ACF 분석 결과 → Lag/Rolling 피처 설계
# Lag 1 (0.40) → lag_1 (업장단위 0.57~0.85)
# Lag 7 (0.42) → lag_7 (주간 주기 반영)
# Lag 28 (0.26) → lag_28 (월간 패턴 반영)
```

#### EDA 5: 피처 상관관계 및 설계 전략

- `lag_1` ↔ `rolling_7d`: **0.84** → 다중공선성 주의
- `zero_streak` ↔ 매출수량: **-0.10** → 0 판매 패턴의 지표
- `weekday_sin`/`month_sin`: -0.05~0.01 → 단독보다 조합으로 활용

| EDA 인사이트 | 설계된 피처 | 근거 |
|-------------|-----------|------|
| 7일 주기 패턴 | `lag_7`, `rolling_7d`, `weekday_sin/cos` | ACF Lag 7 = 0.42 |
| Zero-Inflated | `zero_streak`, `days_since_nonzero` | 52.6%가 0 판매 |
| 업장별 편차 | 업장별 가중치, 카테고리 통계 | 17배 차이 |
| 공휴일 효과 | `is_weekend`, `is_holiday`, `holiday_eve` | +87% 매출 증가 |
| 월별 계절성 | `month_sin/cos`, STL seasonal | 1월 피크, 3월 최저 |
| 단기 추세 | EWM, slope, `rolling_3d` | 최근 트렌드 반영 |

#### EDA 6: sMAPE 평가지표 분석

- **actual = 0** 인 경우 평가에서 **완전 제외** → 전략적 0 예측 가능
- 전체 102,676건 → 0 제외 시 48,642건(-53%) → 추가 0 예측 시 ~38,423건(-21%)

---

### 3️⃣ Feature Engineering

#### 3.1 시계열 이동 통계

| 윈도우 크기 | 통계량 | 설명 |
|-----------|-------|------|
| 3일 | 평균, 표준편차, 합계 | 단기 트렌드 |
| 7일 | 평균, 표준편차, 합계 | 주간 패턴 |
| 28일 | 평균, 표준편차, 합계 | 월간 패턴 |

```python
# shift(1) 사용으로 데이터 누수(Data Leakage) 방지
df['rolling_avg_7d'] = df.groupby('영업장명_메뉴명')['매출수량'].shift(1).rolling(7, min_periods=1).mean()
```

#### 3.2 지연 피처 (Lag Features)
과거 1일, 3일, 7일, 28일 전의 매출 수량을 직접 피처로 사용

#### 3.3 고급 시계열 피처
- **EWM**: 지수가중이동평균 (최근 데이터에 더 높은 가중치)
- **기울기(Slope)**: log1p 변환 후 7일 롤링 윈도우 기울기
- **분위수(Quantile)**: 10%, 25%, 75%, 90% 분위수 및 IQR
- **Z-Score**: 7일 롤링 윈도우 기준 이상치 탐지
- **Zero Streak**: 연속 0 판매 일수
- **Days Since Non-Zero**: 마지막 판매 이후 경과 일수

#### 3.4 누적 및 집계 통계
- 누적 매출, ISO 주차별/월별/카테고리별 평균·표준편차

#### 3.5 STL 분해

```python
stl = STL(y, period=7, robust=True).fit()
group['trend_7'] = stl.trend        # 장기 추세
group['seasonal_7'] = stl.seasonal  # 주간 계절성
group['resid_7'] = stl.resid        # 설명되지 않는 변동
```

---

### 4️⃣ 모델링: LightGBM 단일 모델 정교화

**선택 근거**: XGBoost 대비 빠른 학습 속도, Leaf-wise 트리 성장, Categorical Features 직접 처리, MAE(L1) 손실 함수가 sMAPE와 높은 정렬성

#### 핵심 전략 1: sMAPE 평가 규칙 활용

```python
# actual=0 → 평가 제외 (리스크 없음)
# 불확실한 예측(< threshold) → 0으로 변환
threshold = 0.5  # Optuna로 최적화
prediction[prediction < threshold] = 0
```

#### 핵심 전략 2: Optuna 하이퍼파라미터 튜닝 (100 trials)

```python
{
    'objective': 'l1',              # MAE - sMAPE와 정렬
    'n_estimators': 400~1200,
    'learning_rate': 0.02~0.2,      # log scale
    'num_leaves': 31~255,
    'max_depth': -1~12,
    'min_child_samples': 10~100,
    'feature_fraction': 0.6~1.0,
    'bagging_fraction': 0.6~1.0,
    'lambda_l1': 0.001~10.0,
    'lambda_l2': 0.001~10.0,
}
```

#### 핵심 전략 3: 업장별 차등 가중치

```python
store_weights = {
    '포레스트릿': 2.9,    # 최고매출 (574)
    '카페테리아': 2.3,    # 고매출 (453)
    '화담숲주막': 1.5,    # 중상위 (275)
    '담하': 1.7,          # 중상위 (235, 메뉴 42개 최다)
    '미라시아': 1.4,      # 중위 (184)
    # 기타: 0.6~1.2
}
```

#### 핵심 전략 4: 다중 출력 회귀 (MultiOutputRegressor)

```python
from sklearn.multioutput import MultiOutputRegressor

base_model = LGBMRegressor(**optimized_params)
model = MultiOutputRegressor(base_model)
model.fit(X_train, Y_train)  # Y_train.shape = (n_samples, 7)
```

#### 최종 모델 구조

```
입력 피처 (66개)
   ↓
[카테고리 인코딩] - Label Encoding (영업장명, 메뉴명)
   ↓
[LightGBM MultiOutputRegressor] - MAE 손실, 업장별 가중치, 7개 출력
   ↓
[후처리] - Threshold 기반 0 변환 (< 0.5), Clip to [0, inf]
   ↓
최종 예측값
```

---

### 5️⃣ 평가 및 검증

**평가 지표**: Weighted sMAPE

$$sMAPE = \frac{2 \cdot |actual - predicted|}{|actual| + |predicted|}$$

**검증 전략**: Time-Series Split
- 학습: 2023.01.01 ~ 2024.06.15
- 검증: TEST_08, TEST_09 (마지막 2개 테스트 세트)
- 시간 순서 유지 (과거로 미래 예측)

---

## 🛠️ 기술 스택

| 카테고리 | 기술 |
|---------|------|
| 언어 | Python |
| 머신러닝 | LightGBM, scikit-learn |
| 데이터 처리 | Pandas, NumPy |
| 시계열 분석 | statsmodels (STL) |
| 하이퍼파라미터 튜닝 | Optuna |
| 기타 | holidays (공휴일 데이터) |

---

## 📊 프로젝트 구조

```
LG-Aimers-7/
├── data/
│   ├── train/
│   │   └── train.csv
│   ├── test/
│   │   └── TEST_0X.csv (X=0~9)
│   ├── test_pre/
│   │   └── TEST_0X.csv (전처리 완료)
│   └── sample_submission.csv
├── Gonjiam_lgbm.ipynb              # LightGBM 모델
├── Submission.ipynb                # 최종 제출
└── README.md
```

---

## 🎯 프로젝트 성과

### 해커톤 순위
- **Public**: 173rd of 817 teams
- **Private**: 168th of 817 teams

### 주요 성과
- 50개 이상의 파생 피처 엔지니어링 (STL 분해, 롤링 통계, 래그 피처 등)
- Optuna 기반 100 trials 하이퍼파라미터 자동 최적화
- 업장별 차등 가중치를 고려한 맞춤형 손실 함수 설계
- Zero-Inflated 데이터 특화 피처로 0 판매 패턴 학습
- sMAPE 0값 제외 규칙을 전략적으로 활용한 후처리 파이프라인

---

## 💡 핵심 학습 내용

1. **시계열 데이터 특수성**: `shift(1)` 데이터 누수 방지, Time-based split, Sin/Cos 순환 인코딩
2. **평가 지표 역공학**: sMAPE 0값 제외 규칙 → 전략적 0 예측으로 평가 대상 53% 축소
3. **도메인 지식 활용**: 공휴일(+87%), 주말(+56%), 겨울 성수기(1월 피크) 패턴 반영
4. **단일 모델 정교화**: 복잡한 앙상블 대신 LightGBM 단일 모델 깊이 최적화가 효과적
5. **Zero-Inflated 대응**: 52.6% 0 판매 데이터를 `zero_streak`, `days_since_nonzero` 등으로 명시적 학습

---

## 🔍 향후 개선 가능성

- **외부 데이터**: 날씨(기온, 강수량), 이벤트/프로모션, 관광객 데이터 활용
- **모델 고도화**: Transformer(Self-Attention), Prophet, 딥러닝 앙상블
- **실시간 시스템**: 온라인 학습, A/B 테스트, Flask/FastAPI API 서빙
- **비즈니스 연계**: 예측 기반 발주량 최적화, 폐기 손실 경고 시스템
