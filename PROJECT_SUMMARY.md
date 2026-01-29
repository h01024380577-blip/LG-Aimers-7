# LG Aimers 7기 프로젝트 정리 요약

## 📋 프로젝트 개요

**주제**: 식음업장 메뉴별 판매 수량 예측  
**과제 유형**: 시계열 예측 (Time Series Forecasting)  
**대회**: LG Aimers 7기 온라인 해커톤

---

## 🗂️ 정리된 파일 구조

```
LG Aimers/
├── README.md                      # 프로젝트 설명서
├── requirements.txt               # Python 패키지 의존성
├── .gitignore                     # Git 제외 파일 목록
├── GITHUB_SETUP.md               # GitHub 업로드 가이드
├── PROJECT_SUMMARY.md            # 이 파일
│
├── Gonjiam_lgbm.ipynb            # ⭐ 최종 LightGBM 모델
├── Submission_Ensemble.ipynb     # ⭐ 최종 제출 노트북
│
└── data/
    ├── train/
    │   └── train.csv              # 훈련 데이터
    ├── test/
    │   └── TEST_00.csv ~ TEST_09.csv  # 원본 테스트 데이터
    ├── test_pre/
    │   └── TEST_00.csv ~ TEST_09.csv  # 전처리된 테스트 데이터
    └── sample_submission.csv      # 제출 양식
```

---

## ✅ Git 저장소 상태

- ✓ Git 레포지토리 초기화 완료
- ✓ 핵심 파일들 커밋 완료 (27개 파일, 212,591 라인)
- ✓ `.gitignore` 설정으로 불필요한 파일 제외
  - Jupyter 체크포인트 파일 (`.ipynb_checkpoints/`)
  - 모델 출력 파일 (`oof_output/`, `catboost_info/`)
  - 제출 결과 파일 (`*submission*.csv`)
  - Python 캐시 파일 (`__pycache__/`, `*.pkl`)

---

## 🚀 다음 단계: GitHub 업로드

### 1. GitHub에서 레포지토리 생성
- 레포지토리 이름: `LG-Aimers-7`
- 설명: `LG Aimers 7기 온라인 해커톤 - 식음업장 메뉴별 판매 수량 예측 프로젝트`
- **중요**: README 초기화 체크 해제

### 2. 로컬 레포지토리 연결 및 푸시
```bash
cd "c:\Users\user\Desktop\LG Aimers 7기\LG Aimers"
git remote add origin https://github.com/YOUR_USERNAME/LG-Aimers-7.git
git branch -M main
git push -u origin main
```

자세한 내용은 `GITHUB_SETUP.md` 파일을 참고하세요.

---

## 🎯 제외된 파일들 (불필요한 파일)

다음 파일들은 `.gitignore`로 제외되었습니다:

### 개발 중 실험 파일
- `Ensemble.ipynb`
- `Gonjiam.ipynb`
- `Gonjiam_catb.ipynb`
- `Gonjiam_xgb.ipynb` 등

### 중간 결과 파일
- `data/xgb.csv`, `data/xgb_2.csv ~ xgb_13.csv`
- `data/LGBM2.csv`, `data/catb.csv`
- `data/train_.csv`

### 모델 출력 파일
- `oof_output/` 폴더 전체
- `catboost_info/` 폴더 전체
- 제출 결과 CSV 파일들

### 시스템 파일
- `.ipynb_checkpoints/` 폴더
- `__pycache__/` 폴더
- `*.pkl`, `*.pickle` 파일

---

## 📊 핵심 모델 설명

### Gonjiam_lgbm.ipynb
- **알고리즘**: LightGBM (Light Gradient Boosting Machine)
- **주요 기법**:
  - Feature Engineering (시계열 특성 추출)
  - Optuna 하이퍼파라미터 튜닝
  - MultiOutputRegressor (다중 출력 회귀)
  - STL 분해를 통한 트렌드/계절성 분석
  - 한국 공휴일 특성 추가

### Submission_Ensemble.ipynb
- 최종 제출 파일 생성
- 앙상블 기법 적용 가능
- 다양한 모델 결과 통합

---

## 🔧 개발 환경

### 필수 패키지
```
numpy>=1.21.0
pandas>=1.3.0
scikit-learn>=1.0.0
xgboost>=1.5.0
lightgbm>=3.3.0
optuna>=3.0.0
tqdm>=4.62.0
statsmodels>=0.13.0
holidays>=0.14.0
jupyter>=1.0.0
notebook>=6.4.0
```

### 설치 방법
```bash
pip install -r requirements.txt
```

---

## 📝 커밋 히스토리

```
17b48ec Initial commit: LG Aimers 7기 프로젝트
  - LightGBM 기반 최종 모델 (Gonjiam_lgbm.ipynb)
  - 앙상블 제출 노트북 (Submission_Ensemble.ipynb)
  - 훈련 데이터 및 테스트 데이터
  - 프로젝트 문서 및 의존성 파일
```

---

## 📌 참고 사항

1. **데이터 크기**: 훈련 및 테스트 데이터는 Git에 포함되어 있습니다.
2. **모델 파일**: 학습된 모델 파일(`.pkl`)은 제외되었습니다. 노트북 실행 시 재생성됩니다.
3. **제출 파일**: 제출 결과 CSV 파일은 `.gitignore`로 제외되어 있습니다.
4. **라이선스**: MIT License

---

## 🎓 프로젝트 성과

LG Aimers 7기 온라인 해커톤 참여 완료

---

**정리 완료일**: 2026-01-29  
**Git 레포지토리**: 초기화 완료, GitHub 업로드 준비 완료
