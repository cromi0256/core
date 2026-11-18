2026-11-18 001 AutoML
# AutoML - mljar
[출처](https://supervised.mljar.com/)

[깃허브](https://github.com/mljar/mljar-supervised)

# 캐글 실습

실습 데이터셋 : https://www.kaggle.com/competitions/playground-series-s6e11

[실습코드](https://www.kaggle.com/code/kimyounghwan24/basic-automl)

아주 기본적인 `explain`모드로 auc=0.69301 달성

# AutoML이란
AutoML은 데이터 전처리부터, 특성 엔지니어링, 모델 학습까지 일련의 과정을 자동으로 실행한다.

mljar은 추가로 특성 탐색, 모델 해석, 모델 배포 기능도 더해져 있어 프로토타입에 유리하다.

다른 AutoML패키지인 AutoGluon과 작동 방식은 비슷하며 현재에도 꾸준히 개발이 이루어지고 있다.

# mljar을 고른 이유
데이콘 알고리즘 경진대회에서 자원과 시간이 많이 소요된 부분은 모델의 해석이었다.

단순 자동화는 개발의 편리함을 주지만 그것이 해석의 간단함을 의미하진 않았다.

mljar은 AutoGluon이 제공하지 않은 특성 탐색과 모델 해석이 있기에 실무에 적용하기도 유용하다.

# mljar 사용법
```bash
python -m pip install "mljar-supervised==1.4.0rc6"
```
1.4.0 버전에는 TFM(Tabular Foundation Model)을 포함하며 더 강력한 모델을 만들 수 있게 되었다.

```python
from sklearn.datasets import load_iris
from sklearn.metrics import accuracy_score
from sklearn.model_selection import train_test_split
from supervised import AutoML

# Predict the flower species from four measurements.
X, y = load_iris(return_X_y=True, as_frame=True)
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, stratify=y, random_state=42
)

automl = AutoML(
    results_path="AutoML_Iris",
    mode="Explain",
    algorithms=["Baseline", "Decision Tree", "Linear", "LightGBM"],
    total_time_limit=120,
    n_jobs=2,
)
automl.fit(X_train, y_train)

predictions = automl.predict(X_test)
print(f"Test accuracy: {accuracy_score(y_test, predictions):.1%}")
automl.report()  # Creates AutoML_Iris/README.html.
# Save example inputs so the reload example also works in a new Python session.
X_test.head(3).to_csv("iris_new_rows.csv", index=False)
```
사이킷런을 쓰듯 .fit과 .predict로 모델을 학습한다.

특히, 전처리를 자동으로 처리하여 결측치 처리나 인코딩을 할 필요도 없다.

게다가 모델 학습의 과정과 결과도 `report`로 출력하여 문서화 한다.

# 주의점
그럴지라도 몇가지 작업은 수동으로 설정해야 한다:

* 타깃 컬럼 설정
* 분명한 데이터 유출 컬럼 삭제
* 예측에 불필요한 식별자 삭제
* 훈련 데이터가 실제 예측 시나리오를 반영
* 훈련과 테스트 데이터의 컬럼을 정렬

# 작동방식
모델을 자동으로 만들어 주더라도 어떤 방법이 쓰였는지 알 필요가 있다.

최고의 예측 성능을 계산하는 `compete`에 쓰인 기법:

1. 알고리즘
LGBM, CatBoost 등의 앙상블 트리 계열과 Kumo, TabICLv2 등의 TFM 모델을 학습

여러 학습 모델을 앙상블, 스태킹하여 조합

![image](https://supervised.mljar.com/images/stack_process_flow.png)

2. 스텝
* 기본 하이퍼 파라미터, 약간 랜덤한 하이퍼 파라미터 적용
* 파생변수 생성, 특성 골드마인, Kmeans 특성 적용(+불필요 특성 제거)
* 최상위 모델의 하이퍼 파라미터 영역 미세 조정
* OOF 검증에 쓰인 모델들을 탐욕적 선택으로 앙상블, 스태킹

![image](https://raw.githubusercontent.com/mljar/features_goldmine/main/assets/features_engineering_pipeline.jpg)

3. 편의 기능
```python
from supervised import AutoML

automl = AutoML(results_path="AutoML_results", explain_level=2)
automl.fit(X, y)
automl.report()
```
HTML 리포트 출력

`explain_level=2`는 다음과 같은 추가 기능 제공:
* 학습 곡선(level=0)
* Permutation Importance, 결정 트리 플롯, 선형 모델 가중치(level=1)
* SHAP 해석, 모델 청사진(level=2)

```python
from supervised import AutoML

automl = AutoML(results_path="AutoML")
automl.fit(X, y)
automl.app()
```
로컬&원격 배포

```python
# Binary classification; sensitive_train has one row per training example.
# Its columns identify the groups you want to evaluate.
fairness_search = AutoML(
    results_path="AutoML_Fairness",
    algorithms=["Linear", "LightGBM"],
    fairness_metric="demographic_parity_ratio",
    fairness_threshold=0.8,
)
fairness_search.fit(X_train, y_train, sensitive_features=sensitive_train)
```
집단 간 차이를 측정하고 편향 완화 방안을 평가

# 한계점
AutoML은 사용목적에 따라 적절히 모델 성능을 타협해야한다.

최고 성능 모델은 학습 시간과 예측 시간이 길어지며 주로 경진대회에 쓰인다.

프로덕션에 적용하기엔 부적절하며 모델의 해석과 빠른 추론이 유리할 수 있다.

도메인 데이터의 특징과 문제를 진단하는 것은 여전히 사람의 몫이며, 상황에 맞는 모델 설계에 여전히 코드 작성은 유효하다.