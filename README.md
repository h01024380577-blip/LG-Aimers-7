# LG Aimers 7기 - 식음업장 수요 예측

LG Aimers 7기 온라인 해커톤 프로젝트입니다. 식음업장의 메뉴별 판매 수량을 예측하는 시계열 예측 과제입니다.

## 프로젝트 구조

```
LG Aimers/
├── data/
│   ├── train/
│   │   └── train.csv              # 훈련 데이터
│   ├── test/
│   │   └── TEST_0X.csv            # 테스트 데이터 (X=0~9)
│   ├── test_pre/
│   │   └── TEST_0X.csv            # 전처리 완료된 테스트 데이터
│   └── sample_submission.csv      # 제출 양식
├── Gonjiam_lgbm.ipynb             # LightGBM 모델 (최종)
└── Submission_Ensemble.ipynb      # 최종 제출 노트북
```

## 모델 설명

### Gonjiam_lgbm.ipynb
- **최종 모델**: LightGBM 기반 회귀 모델
- **주요 특징**:
  - Feature Engineering을 통한 시계열 특성 추출
  - Optuna를 활용한 하이퍼파라미터 튜닝
  - 다중 출력 회귀(MultiOutputRegressor) 적용
  
### Submission_Ensemble.ipynb
- 최종 제출 파일 생성
- 앙상블 기법 적용 가능

## 데이터셋

- **train.csv**: 훈련 데이터 (시계열 판매 데이터)
- **TEST_00.csv ~ TEST_09.csv**: 10개의 테스트 데이터셋
- **test_pre/**: 전처리가 완료된 테스트 데이터

## 실행 방법

1. 데이터 준비
```bash
# data 폴더에 train.csv와 TEST_XX.csv 파일 배치
```

2. 모델 학습 및 예측
```python
# Gonjiam_lgbm.ipynb 실행
jupyter notebook Gonjiam_lgbm.ipynb
```

3. 최종 제출 파일 생성
```python
# Submission_Ensemble.ipynb 실행
jupyter notebook Submission_Ensemble.ipynb
```

## 요구사항

```
numpy
pandas
scikit-learn
xgboost
lightgbm
optuna
tqdm
statsmodels
holidays
```

## 성과

LG Aimers 7기 온라인 해커톤 참여 프로젝트

## 라이선스

MIT License
