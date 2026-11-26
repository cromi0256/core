### Preface : uv, polars, IsolationForest, SHAP, Plotly를 사용한 머신러닝
1. uv : pip를 대체하는 패키지 관리자
2. polars : polars를 대체하는 데이터 전처리 도구
3. IsolationForest : 트리 기반 이상치 탐지 알고리즘
4. SHAP : 게임이론에 기반을 둔 모델 해석 알고리즘
5. Plotly : 동적 시각화 도구

# 0. 개발 코드
[실제 작성 코드](https://github.com/cromi0256/core/blob/main/SOTA/Project05/src/project05/iso_dev00.ipynb)

\src\project05\iso_dev00.ipynb참조

# 1. uv&패키지 설치
[설치가이드](https://docs.astral.sh/uv/getting-started/installation/)
```PowerShell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```
uv설치

```PowerShell
uv init
```
현재 디렉토리에 프로젝트 연결

```PowerShell
uv venv --python 3.12
```
파이썬 3.12 가상환경 설치

```PowerShell
.venv\Scripts\activate
```
가상환경 활성화

```PowerShell
python -V
```
파이썬 버전 확인

```PowerShell
uv add scikit-learn polars plotly shap --active
```
현재 실행환경(--active > 가상환경 명시)에 패키지를 설치

이 과정은 pip로 설치했을때보다 확실히 빠르다


+ VSCode환경에서 시작
```PowerShell
uv add --dev ipykernel
```
# 2. 데이터셋 불러오기
데이터 : load_breast_cancer(from 사이킷런)

크기 : (569, 30) 타깃 열은 제외됨

```python
data = load_breast_cancer()
X = data.data
schema = list(data.feature_names)

df = pl.DataFrame(X, schema=schema)
```

# 3. 이상치 모델 적용
트리모델이므로 특성의 스케일링은 불필요

모든 특성이 수치형이므로 원핫인코딩 또한 불필요
```python
model = IsolationForest(n_jobs=-1, random_state=42)
model.fit(df)
```

# 4. 모델 해석
[SHAP](https://shap.readthedocs.io/en/latest/index.html)을 사용하여 모델을 해석
```python
explainer = shap.Explainer(model)
shap_values = explainer(X)  # polars를 지원하지 않아 넘파이 배열을 입력

sv = shap_values.values
shap_values.feature_names = schema
```

플롯은 개발 코드를 참조

# 5. 동적 시각화 설정
matplotlib과 seaborn은 정적 시각화 툴로 인터랙션 할 수 없다

plotly.graph_object는 한 그래프에 사용자가 원하는 특성을 골라 시각화 할 수 있도록 한다

```python
fig = go.Figure()

fig.add_trace(...)

buttons = []

for col in sv_df.columns:
    buttons.append(...)

fig.update_layout(...)

fig.show()
```

# 끝으로
코드 작성중의 충돌이나 제안의 경우 AI를 쓰면 귀찮은 검색 시간이 줄어든다.

그렇지만 이 작업을 모두 AI에 맡긴다면 지식은 사라지고 의도와 설계는 무너지게 된다.

개발환경은 차츰 바뀌고 있지만 중요한 것을 잊으면 안된다.
