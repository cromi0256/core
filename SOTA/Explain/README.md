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

explainer = shap.Explainer(model)
explaination = explainer(X1000)  # SHAP 값 생성
```

## SHAP Plots
```Jupyter Notebook
# 전역적 중요도
shap.plots.beeswarm(explaination) 
```
<img width="744" height="436" alt="image" src="https://github.com/user-attachments/assets/5f3122e4-1590-4bb2-8084-7a735fb4ee21" />

위의 표를 보면 가장 중요한 특성은 MedInc이며 값이 높을수록 타깃 또한 높아지는 경향이 있다

SHAP값을 히스토그램과 비슷하게 출력하여 다각적인 분석을 할 수 있다

```Jupyter Notebook
# 전역적 중요도
shap.plots.bar(explaination)
```
<img width="756" height="491" alt="image" src="https://github.com/user-attachments/assets/559de991-fb77-4159-a352-663e15d04adc" />

막대 그래프는 평균적인 SHAP값으로 이 값이 낮다면 타깃과 연관이 낮다는 뜻이다

```Jupyter Notebook
# 국소적 설명
shap.plots.bar(explaination[50])
```
<img width="846" height="491" alt="image" src="https://github.com/user-attachments/assets/5b45648d-cf14-43d4-bdfb-3715dcd81a6d" />

특정 샘플에서 어떤 특성이 영향을 주었는지 알 수 있다

```Jupyter Notebook
# 코호트 분석
shap.plots.bar(explaination.cohorts(2))
```
<img width="751" height="619" alt="image" src="https://github.com/user-attachments/assets/e41d480d-0d20-479a-9986-0ac3b1e47467" />

N개의 집단으로 구분하여 각 집단의 특성 영향을 비교할수도 있
