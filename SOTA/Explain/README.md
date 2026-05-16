[공식페이지](https://shap.readthedocs.io/en/latest/)

SHAP은 게임 이론 접근방식으로 머신러닝 모델의 출력물을 설명한다.

현재(2026.05) 정형데이터, 이미지, 텍스트, 유전체학 모델 설명을 지원한다.

```PowerShell
pip install shap
```

본론에 앞서 Feature Importance, Partial Dependence Plots는 SHAP와 쓰임이 다르다.

Feature Importance는 모델에 쓰인 특성의 중요도로 이 수치가 타깃과 선형적으로 일치하리라는 보장이 없다.

즉, 각각의 샘플을 설명하는데 FI가 일관된 해석(전역적)을 하기에 한계가 분명하다.

PDP는 특성과 타깃간의 관계를 다른 변수를 고정시킴으로써 설명한다.

다중공선성에 매우 취약하며, 이 역시 특정 샘플에 대한 설명을 제공하지 못한다.

두 방법 모두 모델의 설명보다는 점검에 가까우며, 사용자가 아닌 개발자를 위한 정보를 제공한다.

```Python
import sklearn
import shap
from sklearn.ensemble import HistGradientBoostingRegressor

X, y = shap.datasets.california(n_points=10000)
X1000 = shap.utils.sample(X, 1000)  # 전체 샘플중 1000개 샘플링(10%)

model = HistGradientBoostingRegressor(random_state=42)
model.fit(X, y)

explainer = shap.TreeExplainer(model)  # Tree알고리즘에 최적화된 explainer
shap_values = explainer(X1000)  # SHAP 값 생
```

## SHAP Plots
