# AI-Model-for-Pitch-Level-Control-Success-Probability-Prediction
Pitch-level control success probability prediction using pre-pitch game context and historical baseball data.

# Baseline

01. EDA 및 데이터 무결성 검증
- train.csv 데이터 구조, 결측치 및 데이터 타입(float32/category) 최적화
- 시즌 및 경기 유형(game_type)별 제구 성공률(control_success) 타깃 드리프트 분석
- 데이터 논리적 정합성 검점 및 Trackman 데이터 결측·시즌 요약 분석
- 과거 시즌 학습 후 2024년 시즌을 검증하는 시간축 검증(Temporal Backtest) 수행

02. 베이스라인 구축 및 모델 실험
- LightGBM 및 CatBoost 분류기 기반 5-Fold Stratified Cross-Validation 구축
- 범주형 변수(top_bottom, game_type, base_state)에 대한 Fold별 TargetEncoder 적용
- Brier Score 최적화를 위한 예측 확률값(Probability Calibration) 보존 기법 적용
- 단일 모델(LGBM, CatBoost) 대비 앙상블을 통한 예측 안정성 확인

03. 앙상블 및 블렌딩 실험
- LightGBM과 CatBoost 간 최적 가중치 비율 탐색 (5:5, 6:4, 7:3 등)
- 예측값 순위화(Rank Averaging) 적용 시 BSS 산식 왜곡 문제 발견 및 원본 확률 블렌딩으로 보정
- Brier Score 기반 예측 편향 최소화 및 0~1 범위 클리핑(Clipping) 적용

04. 제출 파이프라인 무결화 및 서버 검증
- 데이콘 서버 규격(script.py, ./data, ./model, ./output) 100% 반영
- 파이썬/C++ 라이브러리 간 타입 불일치 에러 방지를 위한 float32 강제 변환 레이어 구축
- 로컬 가상 추론 테스트(Mock Run)를 통한 0점(채점 실패) 2중 예외 방어망 구축
- submit.zip 일괄 자동 압축 및 검증 파이프라인 완성

05. 1차 제출 및 기준 점수 확보
- LightGBM 5-Fold + CatBoost 5-Fold (5:5 Simple Averaging) 제출
- 리더보드 754.64점 기록 (정상 채점 및 베이스라인 성능 확인)

06. 결론
- LightGBM(50%)과 CatBoost(50%)의 5-Fold 앙상블을 적용한 모델을 최종으로 선정함
- 리더보드 754.64점을 기록하며 가장 안정적인 성능을 보임
- 향후 파생변수 추가 및 모델별 가중치 미세 조정을 통해 추가 성능 향상을 도모할 예정임

# StrikeBall

07. 볼카운트 지표 추가
- 가설 설정:
  볼카운트 상황(ball, strike)에 따른 투수의 심리적 압박감이 제구 성공률(control_success)에 직접적인 영향을 미칠 것으로 판단함
- 신규 생성 파생변수 (4종):
  ball_strike_diff: 볼과 스트라이크의 차이 (ball - strike)
  is_hitter_count: 타자가 절대적으로 유리한 카운트 여부 (2-0, 3-0, 3-1)
  is_2strike: 투수가 유리한 2스트라이크 상황 여부
  is_full_count: 3볼 2스트라이크 풀카운트 상황 여부
- 검증 결과: 리더보드 779.07점 기록, Baseline 대비 소폭 점수 상승

# HandClose

08. 손잡이, 접전 지표 추가
- 가설 설정:
  투수-타자의 손잡이 상성(hand_match) 따라 좌/우타자 상대 제구 패턴 변화
  점수 차이가 크지 않은 접전 상황(is_close_game)에서 투수의 심리적 압박 제구에 영향
- 신규 추가 파생변수 (2종):
  hand_match: 투수와 타자의 투타 손잡이 조합 (p_throws + b_bats, 예: R_L, L_L 등)
  is_close_game & abs_score_diff: 점수 차 절대값 및 2점 차 이내 접전 상황(Close Game) 여부 플래그
- 전처리 변경: hand_match 범주형 조합에 대한 Target Encoding 신규 추가 적용
- 검증 결과 : 리더보드 789.51점 기록, 소폭 점수 상승

# Scoring

09. 득점권 지표 추가
- 가설 설정:
  득점권 위기(is_scoring_position): 득점권 상황에서 작은 제구 미스가 실점으로 직결되므로 투수의 투구 신중함과 제구 분포가 크게 변화
  득점권 × 카운트 결합 위험도(scoring_hitter_count_risk): 득점권 위기와 볼카운트가 복합적으로 작용할 때 제구 중압감이 극대화, 제구 성공률에 결정적인 영향
- 신규 생성 파생변수:
  is_scoring_position: base_state 파싱을 통한 2루/3루 득점권 주자 존재 여부 플래그 (1/0)
  scoring_hitter_count_risk: is_scoring_position × is_hitter_count 상호작용 피처 (득점권 위기 속 카운트 몰림 상황)
- 인코딩 및 구조 개선:
  base_state 및 득점권 관련 변수가 외부 TargetEncoder와 충돌하여 발생할 수 있는 과적합 방지 위해 외부 인코더를 완전히 제거
  범주형 변수를 모델이 주자 상황별 제구 패턴을 안전하게 학습하도록 개선
- 검중 결과 : 리더보드 773.59점 기록, 소폭 점수 하락



