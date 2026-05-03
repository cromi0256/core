아주 간단한 프로세스를 시작으로 자동화되고 고도화된 MLops 설계를 목표로 합니다.

## Tools
![image](https://mlops-for-all.github.io/assets/images/mlops-component-540cce1f22f97807b54c5e0dd1fec01e.png)
- Experimentation : [Git, GitHub, VSCode], [MLflow, DVC], [matplotlib, seaborn, plotly, TensorBoard], [pandas, polars], [KubeFlow]
- Data processing : [Apache Spark], [Apache kafka], [Scikit-learn, Hugging Face], [Apache Flink]
- Model training : [PyTorch, Scikit-learn], [Optuna, Ray Tune]
- Model evaluation : [MLflow], [SHAP, LIME, Captum]
- Model serving : [Docker], [KServe, FastAPI]
- Online experimentation : <카나리, 섀도 배포>, <A/B 테스트>, <MAB 테스트>
- Model monitioring : [Prometheus, Grafana], [MLflow]
- ML pipelines : [Apache Airflow, Kubeflow]
- Model registry : [MLflow]

## Overview
![image](https://towardsdatascience.com/wp-content/uploads/2024/08/1M2Cmy6S6P4ozs9Wylz1eFg.jpeg)

## Local Test
소규모 프로젝트 프로세스

로컬환경에서 애자일 방식으로 빠른 배포를 목표로 합니다

**구성방식**
1. Data : 데이터셋 버전 컨트롤<DVC>
2. Code : 파이프라인 관리(.venv(환경변수), Dependency(Requirements.txt), config, .gitignore, CI/CD)<Git, GitHub, GitHub Actions>
3. Model : 실험기록 및 레지스트리 관리(Artifact, 하이퍼파라미터 탐색, Build/Train/Evaluate)<Scikit-learn(ML), PyTorch(DL), MLflow, Optuna>
4. Artifact : 이해관계자 해석(EDA, Inspection, Monitoring)<Pandas, Matplotlib, SHAP>
5. Deploy : 컨테이너화(로컬 호스팅)<Docker, FastAPI>

**기능**
원본 데이터로드 Load
데이터 준비 Clean
CI/CD가 가능한 Train
(최초 1회 최적 모델 탐색) HPO
모델 검증 Eval
모델 로그, 아티팩트 저장 Save
컨테이너화 Serve
(해석 필요시) Visualize
드리프트 모니터링 Monitor

CI/CD툴을 사용하여 수동으로 코드 오류 감지

## External Link
[Blog](https://mlops-for-all.github.io/docs/introduction/component/)
[Article](https://towardsdatascience.com/machine-learning-operations-mlops-for-beginners-a5686bfe02b2/?utm_source=roadmap&utm_medium=Referral&utm_campaign=TDS+roadmap+integration)
[Google](https://services.google.com/fh/files/misc/practitioners_guide_to_mlops_whitepaper.pdf)
