# 📉 American Companies Bankruptcy Prediction (기업 신용평가모형)

## 📌 프로젝트 목적 (Project Objective)
본 프로젝트는 **미국 상장기업의 재무 데이터(1999~2018)**를 활용하여, 대상 기업의 **부도(Bankruptcy) 가능성을 평가/예측하는 신용평가 파이프라인(CSS, Credit Scoring System)**을 구축하는 것을 목적으로 합니다. 
실제 신용평가사(NICE디앤비 등)의 평가모형 개발 절차와 논문(「김종윤, 2019」)의 핵심 방법론을 직접 구현하여, 단순 재무 수치가 아닌 시계열적 구조재무비율(성장성, 수익성, 안정성, 활동성 등)의 변화 방향과 속도를 측정합니다.

---

## 📊 데이터셋 개요 (Data Overview)
- **출처 (Kaggle)**: [American Companies Bankruptcy Prediction Dataset](https://www.kaggle.com/datasets/utkarshx27/american-companies-bankruptcy-prediction-dataset)
- **데이터 기간**: 1999년 ~ 2018년
- **데이터 규모**: 8,971개 기업 (총 78,682 관측치/행)
- **원본 변수**: 기업 재무상태표 및 손익계산서 기반 18개 핵심 재무 변수 (유동자산, 이익잉여금, 감가상각, EBIT 등)
- **타깃 변수 (Target Label)**:  
  - `failed (1)`: 부도/불량 상태  
  - `alive (0)`: 정상/우량 상태
  - *데이터베이스 내 기업 단위 기준 부도율*: **6.79%**

---

## ⚙️ 분석 파이프라인 및 개발 구조 (Methodology)

### 1. 시점별/기업별 데이터 분할 (Data Splitting)
데이터의 겹침 현상(`Data Leakage`)과 타깃 불균형을 해결하기 위해 일반적인 횡단면 분할이 아닌 **기업 단위 랜덤 분할**을 적용했습니다.
- **Train (70%)**: 주요 평가 변수 선택 및 평가모형 초기 적합 (6,279 기업)
- **Hold-Out (15%)**: 모형의 중간 검증 및 튜닝 (1,346 기업)
- **OOT, Out-of-Time (15%)**: 시점 차이에 따른 모델 변별력 평가(PSI 산출 및 ROC AUC 테스트 등 최종 단계)

### 2. 고도화된 파생변수 생성 (Advanced Feature Engineering)
신용평가에서 재무 제표의 단순 절댓값 수치는 해석하기 어렵습니다. 이에 따라 원본 18개 변수로부터 **총 896개의 평가항목 후보군(파생변수)**을 자동 생성하는 구조를 택했습니다.
- **시계열 파생변수**: 1~3년 변화율(성장성), 이동 평활화(추세), 단기 변동성 표준편차, 가속도 악화 속도 등
- **2개 변수 조합**: 나누기 비율 지표(수익성, 안정성) 및 빼기 차이 지표(순운전자본)
- **듀퐁 & Altman Z-Score 분석**: 재무비율을 다중 조합한 복합 평가 지표 산출

### 3. 검증 및 모델링 (Modeling & Validation)
- 생성된 파생변수의 결측치(NaN)나 초기연도 편차를 최적 클래스(Fine Classing)화 처리할 단계입니다.
- 로지스틱 회귀분석(Logistic Regression) 모델 베이스라인 적용 및 다중공선성(VIF) 통제.
- `PSI` 및 변별력 측정 지표인 `ROC AUC`를 바탕으로 단일 평점/모형 성능 평가를 수행합니다.

---

## 📁 디렉토리 구조 (Directory Structure)
```text
📦 tech_seminar_02
 ┣ 📂 data                 # Kaggle CSV 원본 데이터
 ┃ ┗ 📜 american_bankruptcy.csv
 ┣ 📂 references           # 모형 설계 핵심 참고 문헌 및 공시 논문
 ┃ ┣ 📜 기업신용등급체계 공시.pdf
 ┃ ┗ 📜 통신 빅데이터 활용 개인신용평가모형(통신스코어) 개발.pdf
 ┣ 📜 CSS.ipynb            # 전체 CSS 파이프라인 개발 노트북 파일
 ┣ 📜 requirements.txt     # 프로젝트 실행을 위한 Python 환경 의존성 목록
 ┗ 📜 README.md            # 현재 프로젝트 소개 마크다운
```