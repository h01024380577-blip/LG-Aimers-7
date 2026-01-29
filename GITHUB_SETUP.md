# GitHub 레포지토리 설정 가이드

이 프로젝트를 GitHub에 업로드하는 방법입니다.

## 1. GitHub에서 새 레포지토리 생성

1. GitHub 웹사이트 (https://github.com)에 접속
2. 우측 상단의 `+` 버튼 클릭 → `New repository` 선택
3. 레포지토리 정보 입력:
   - **Repository name**: `LG-Aimers-7`
   - **Description**: `LG Aimers 7기 온라인 해커톤 - 식음업장 메뉴별 판매 수량 예측 프로젝트`
   - **Public** 또는 **Private** 선택
   - **중요**: ✗ "Initialize this repository with a README" 체크 해제
4. `Create repository` 클릭

## 2. 로컬 레포지토리를 GitHub에 연결

GitHub에서 레포지토리를 생성하면 나타나는 페이지에서 제공하는 명령어를 사용하거나, 아래 명령어를 터미널에서 실행하세요:

```bash
# 현재 디렉토리로 이동
cd "c:\Users\user\Desktop\LG Aimers 7기\LG Aimers"

# GitHub 레포지토리를 원격 저장소로 추가 (YOUR_USERNAME을 본인의 GitHub 사용자명으로 변경)
git remote add origin https://github.com/YOUR_USERNAME/LG-Aimers-7.git

# 또는 SSH를 사용하는 경우
# git remote add origin git@github.com:YOUR_USERNAME/LG-Aimers-7.git

# 브랜치 이름을 main으로 설정 (이미 main이면 생략 가능)
git branch -M main

# GitHub에 푸시
git push -u origin main
```

## 3. 인증 설정

처음 푸시할 때 인증이 필요할 수 있습니다:

### Personal Access Token 사용 (권장)

1. GitHub → Settings → Developer settings → Personal access tokens → Tokens (classic)
2. `Generate new token (classic)` 클릭
3. 필요한 권한 선택:
   - ✓ `repo` (전체 레포지토리 제어)
4. 생성된 토큰을 복사
5. Git 푸시 시 비밀번호 대신 토큰 입력

### SSH 키 사용

```bash
# SSH 키 생성 (아직 없는 경우)
ssh-keygen -t ed25519 -C "your_email@example.com"

# SSH 키를 GitHub에 추가
# 생성된 공개 키를 복사하여 GitHub → Settings → SSH and GPG keys에 추가
```

## 4. 푸시 완료 확인

푸시가 완료되면 GitHub 레포지토리 페이지에서 모든 파일을 확인할 수 있습니다.

## 프로젝트 구조 (업로드된 파일)

```
LG-Aimers-7/
├── .gitignore                     # Git 제외 파일 목록
├── README.md                      # 프로젝트 설명
├── requirements.txt               # Python 의존성
├── Gonjiam_lgbm.ipynb            # 최종 LightGBM 모델
├── Submission_Ensemble.ipynb     # 최종 제출 노트북
└── data/
    ├── train/
    │   └── train.csv
    ├── test/
    │   └── TEST_00.csv ~ TEST_09.csv
    ├── test_pre/
    │   └── TEST_00.csv ~ TEST_09.csv (전처리 완료)
    └── sample_submission.csv
```

## 추가 파일 업로드

나중에 추가 파일을 업로드하려면:

```bash
# 파일 추가
git add 파일명

# 커밋
git commit -m "커밋 메시지"

# 푸시
git push
```

## 문제 해결

### 인증 오류
- Personal Access Token이 올바른지 확인
- Token에 `repo` 권한이 있는지 확인

### 푸시 충돌
```bash
# 원격 저장소의 변경사항 먼저 가져오기
git pull origin main --rebase

# 다시 푸시
git push
```

### 큰 파일 경고
- 100MB 이상의 파일은 Git LFS 사용 권장
- 또는 `.gitignore`에 추가하여 제외
