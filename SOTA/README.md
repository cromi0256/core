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
