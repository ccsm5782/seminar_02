# 🛡️ Two-Tier Master Credit Scoring System (70:30)

본 프로젝트는 상장 기업의 부도 예측력을 극대화하기 위해 **내부 재무지표(70%)**와 **외부 비재무 지표(30%)**중 재무모형 신용평가 모델 시스템을 재현한 프로젝트입니다. 국내 신용평가사의 방법론(NICE 등)과 학술적 가이드라인(김종윤, 2019)을 엄격히 준수하여 설계되었습니다.

---

## 🏗️ 1. 프로젝트 아키텍처 및 철학

*   **Financial Model (70%)**: 기업의 장부상 실적(수수성, 활동성, 안정성 등)을 896개의 비율로 분석.

---

## 📈 2. 단계별 상세 과정 (Sequential Workflow)

### [Phase 1] Financial Model (`CSS_Financial.ipynb`)
재무 비율 기반의 내부 건전성 측정 모델 개발 과정입니다.

1.  **Candidate Generation (Cell 1-4)**: 20여 개의 기초 재무제표 항목으로부터 896개의 재무 비율 및 시계열 변화량 생성.
2.  **Sample Sync (Cell 5)**: `Fixed Seed=42`를 사용하여 Train(70%), Hold-out(15%), OOT(15%) 데이터 분할. 분석 단위는 기업당 '최근 1개년' 데이터로 동기화.
3.  **Fine Classing (Cell 6)**: 모든 변수를 20개 구간으로 균등 분할. KS 통계량 10% 미만의 변별력 없는 변수 1차 탈락.
4.  **Coarse Classing (Cell 7)**: 인접 구간 간 부도율 역전 현상 발생 시 반복 병합. **최소 비중 5%** 및 **단조성(Monotonicity)** 확보 (김종윤, 2019, p.64).
5.  **Recoding (Cell 8)**: 구간별 부도율 순서에 따라 0(고위험) ~ N(저위험) 정화 수치 부여.
6.  **Univariate Filter (Cell 9)**: WOE/IV 산출 및 단변량 로지스틱 회귀. 유의수준 0.05 미만 및 모든 계수가 양수(+)인 변수 선별.
7.  **Multi-Collinearity Audit (Cell 10-11)**: 상관계수 0.6 초과 쌍 중 IV 낮은 쪽 제거 및 **VIF 5 이상** 변수 축출.
8.  **Stepwise Selection (Cell 12)**: AIC(Akaike Information Criterion) 최소화 기준 전진/후진 선택법으로 최종 14개 변수 확정.
9.  **Scorecard & Validation (Cell 13-15)**: **Zero-Anchoring** 방식의 스코어카드 산출 및 KS, AR, PSI를 통한 모델 성능/안정성 검증.


---

## 📂 3. 결과물 저장 및 아카이빙 (Synchronized Results)

두 모델의 결과물은 각각 독립된 폴더에 동일한 파일 이름으로 저장됩니다.

*   `CSS_Financial_results/` & `CSS_NonFinancial_results/`
    *   `feature_dictionary.csv`: 전체 후보 변수 (896) 명단
    *   `final_vars.json`: 최종 선발 변수 리스트
    *   `final_logit_model.pkl`: 학습 완료된 모델 직렬화 파일
    *   `model_params.json`: 상수항 및 스케일링(Factor, Offset) 파라미터
    *   `scorecard_final.csv`: 구간별 배점이 명시된 최종 평점표
    *   `validation_charts.png`: KS, AR, PSI, ROC/CAP 커브 시각화 리포트

---

## 🔬 4. 주요 방법론 특이사항

*   **Zero-Anchoring**: 각 변수의 가장 위험한 구간(Worst Bin)에 0점을 부여하고, 계수 차이만큼 가산점을 주는 방식. 이는 점수가 높을수록 우량함을 직관적으로 보여주며, 총점 관리가 용이함.
*   **Sample Synchronization**: 두 모델이 같은 시점에 같은 기업을 평가하도록 `Last Observation` 로직을 동일하게 적용하여 통합 시의 데이터 오차 제거.

---
**Reference**: 김종윤(2019). "중소협력업체 신용평가모델 개발 및 검증에 관한 연구"  
**Created By**: Antigravity (Advanced AI Coding Assistant)



# 앞으로 할 것
- 비재무 모형의 변수를 증강 등의 방법으로 생성해 7:3 가중치 적용해서 비교할 것. 
- 7을 어떻게 나눌지, 3을 어떻게 나눌지 그 최적 비율을 찾아야 함.
- **Dart API 활용으로 Airflow로 모델의 적합성을 자동으로 검증 가능하게 신규 데이터로 학습시킬 것. (실제 사용 가능성과 연결 및 모니터링 가능성 확인)**
