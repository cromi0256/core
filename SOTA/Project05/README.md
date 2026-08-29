### Preface : uv, polars, IsolationForest, SHAP, Plotly를 사용한 머신러닝
1. uv : pip를 대체하는 패키지 관리자
2. polars : polars를 대체하는 데이터 전처리 도구
3. IsolationForest : 트리 기반 이상치 탐지 알고리즘
4. SHAP : 게임이론에 기반을 둔 모델 해석 알고리즘
5. Plotly : 동적 시각화 도구

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
uv add scikit-learn polars 
```
최신버전의 scikit-learn 설치

이 과정은 pip로 설치했을때보다 확실히 빠르다

# 2. 