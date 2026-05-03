## 모델 성능
[PriorLabs](https://docs.priorlabs.ai/improving-performance)

정형 데이터 최고 성능의 Foundation Model인 TabPFN 2.6 문서에서 발췌

**성능 향상 순서**

0. 기본 성능(기본 파라미터) 평가
1. 특성 엔지니어링 : 도메인 특성(비율, 상호작용, 그룹) 추가, Datetime분할
2. 특성 선택 : 특성이 많은 경우(100+) 관련없는 특성 삭제
3. 평가 튜닝 : 분류기 보정, 평가 지표 변경
4. 전처리 변형 : [비선형 변형](https://scikit-learn.org/stable/auto_examples/preprocessing/plot_map_data_to_normal.html), 타깃인코딩 적용, (회귀)타깃 변형
5. 하이퍼파라미터 조정 : 모델의 파라미터 영역 탐색
6. 모델 앙상블 : 다양한 하이퍼파라미터 모델 결합
7. ~~파인 튜닝 : 파운데이션 모델에 한함~~

## HistGradientBoosting 뜯어보기
scikit-learn 1.8.0 버전 기준

**공통 파라미터**
1. *loss*(손실함수)['log_loss' / 'squared_error'] : 추정기의 최적화 방식 - 분류기의 경우 설정 불필요
2. *learning_rate*(학습률)[0.1] : 오차 조정 정도 - 더 정밀한 계산을 원한다면 낮출것
3. *max_iter*(반복횟수)[100] : 전체 트리의 수 - 높을수록 과적합 위험이 있어, 조기종료와 사용
4. *max_leaf_node*(최대 리프 노드수)[31] : 높을수록 과적합 위험
5. max_depth(최대 깊이)[None] : 제한없음, HPO 설정엔 불필요
6. *min_samples_leaf*(리프 최소 샘플 수)[20] : 분기 기준 샘플 수 - 낮을수록 과적합 위험
7. *l2_regularization*(L2정규화)[0.] : 리프 노드값 정규화 - 높을수록 강력한 규제
8. max_features(최대 특성)[1.] : 노드 분할시 특성수 제한 - 특성이 많다면 고려 대상
9. max_bins(최대 빈)[255] : 연속형 변수 이산화 기능 - HPO 대상 아님(최대치255)
10. categorical_features(범주형 변수)['from_dtype'] : 카테고리형 특성 인식 - 전처리 과정에서 object를 category로 지정할것
11. monotonic_cst(단조 제한)[None] : 특정 특성과 타깃의 관계 제한, 이해관계자용 고급설정
12. interaction_cst(상호작용 제한)[None] : 특성간의 상호작용 분리
13. warm_start[False] : 이전 학습 이어하기 - HPO 대상 아님
14. early_stopping(조기종료)['auto'] : < 10000 True - HPO 대상 아님
15. scoring['loss'] : 조기종료 기준 지표 - 비즈니스 목적에 따라 설정
16. validation_fraction(검증 비율)[0.1] : 더 나은 검증 방식을 위해 .fit()에 X_val, y_val을 준비할것(검증데이터가 있다면 무시됨)
17. n_iter_no_change(조기종료 허용치)[10] : 해당 수치만큼 조기종료 트리거 연기
18. tol(허용치)[1e-7] : 조기종료 한계점 - HPO 대상 아님
19. verbose[0] : 프로세스 정보 출력도 - 모델의 로그 추적용
20. random_state[None] : 재현성을 위해 특정 int값으로 설정(config파일로 설정)
분류기A. class_weight[None] : 클래스 가중치 조정 - 불균형이 있다면 'balanced'로 조절
회귀B. quantile[None] : loss가 'quantile'일 경우 활성화, 구간단위 예측에 사용

