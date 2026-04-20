이 프로젝트는 kaggle의 데이터셋을 받아 실행하였습니다.

**데이터셋 정보**

[Kaggle]https://www.kaggle.com/competitions/playground-series-s6e4

자세한 정보는 위 링크에서 확인

## 프로젝트 셋업
playground-series-s6e4 에서 시작

시작 전에 python3.13.13 버전을 설치

VSCode에서 실행

1. 파이썬 가상환경 설정

```PowerShell
py -3.13 -m venv venv
```

가상환경 설치

```PowerShell
venv\Scripts\activate
```

가상환경 활성화

```PowerShell
python --version
```

(파이썬 버전 확인)

2. 패키지 설치

```PowerShellPowerShell
pip install -U scikit-learn
```

사이킷런 설치

```PowerShell
pip install -U mlflow
```

3. git 연동
```PowerShell
git init
```

git과 연결

(.gitignore은 이미 설정되어 있지만 필요에 따라 생성)

```PowerShell
pip freeze > requirements.txt
```

패키지 목록 저장

```PowerShell
git add .
git commit -m "첫 커밋"
```

스테이징 후 첫 커밋

(첫 실행시 git 이름이나 이메일을 설정)

## 모델 훈련 & MLflow 연결
```Main.py
import pandas as pd
import numpy as np
import logging
import mlflow
from sklearn.compose import ColumnTransformer
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import OrdinalEncoder, LabelEncoder
from sklearn.model_selection import StratifiedKFold, cross_val_score
from sklearn.ensemble import HistGradientBoostingClassifier
from sklearn.metrics import classification_report, f1_score
from pickle import dump

# Set up logging
logging.basicConfig(level=logging.INFO,format='%(asctime)s:%(levelname)s:%(message)s')

def main():

    mlflow.set_experiment("Model Training Experiment")

    with mlflow.start_run() as run:

        # Load Data
        train_df = pd.read_csv('./train.csv')
        test_df = pd.read_csv('./test.csv')
        logging.info("Data ingestion completed successfully")

        # Clean Data
        cat_cols = train_df.select_dtypes(include=['object']).columns.drop(['Irrigation_Need']).tolist()
        num_cols = train_df.select_dtypes(include=['int64', 'float64']).columns.drop(['id']).tolist()
        target_col = 'Irrigation_Need'
        X = train_df.drop(['Irrigation_Need'], axis=1)
        y = train_df[target_col]
        le = LabelEncoder()
        y_ = le.fit_transform(y)

        # PipeLine
        preprocessor = ColumnTransformer(
            transformers=[
                ('id', 'drop', ['id']),
                ('num', 'passthrough', num_cols),
                ('cat', OrdinalEncoder(
                    handle_unknown='use_encoded_value',
                    unknown_value=-1
                ), cat_cols)
            ]
        )
        logging.info("Data cleaning completed successfully")

        # Model Train
        params = {"random_state" : 42}
        pipeline = Pipeline([
            ("preprocess", preprocessor),
            ("model", HistGradientBoostingClassifier(**params))
        ])
        pipeline.fit(X, y_) # 전체 데이터 학습

        # Save Model
        with open("model.pkl", "wb") as f:
            dump(pipeline, f, protocol=5)
        logging.info("Model training completed successfully")

        # Model Evaluate
        cv = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)
        oof_preds = np.zeros(len(X))

        for fold, (train_idx, valid_idx) in enumerate(cv.split(X, y_)):
            X_train, X_valid = X.iloc[train_idx], X.iloc[valid_idx]
            y_train, y_valid = y_[train_idx], y_[valid_idx]
            
            model = pipeline
            model.fit(X_train, y_train)
            
            preds = model.predict(X_valid)
            oof_preds[valid_idx] = preds
        
        # classification_report(y_, oof_preds)
        logging.info("Model evaluation completed successfully")

        # Tags 
        mlflow.set_tag('Model developer', '김영환')
        mlflow.set_tag('preprocessing', 'OrdinalEncoder')

        # Log Metrics
        mlflow.log_metric('f1-score', f1_score(y_, oof_preds, average='weighted'))
        mlflow.sklearn.log_model(pipeline, 'model')

        # Register the model
        model_name = "HGB_model" 
        model_uri = f"runs:/{run.info.run_id}/model"
        mlflow.register_model(model_uri, model_name)

        logging.info("MLflow tracking completed successfully")

        # Print evaluation results
        print("\n============= Model Evaluation Results ==============")
        print(f"Model: {model_name}")
        print(f"\n{classification_report(y_, oof_preds)}")
        print("=====================================================\n")

if __name__ == "__main__":
    main()
```
[link](https://towardsdatascience.com/machine-learning-operations-mlops-for-beginners-a5686bfe02b2/?utm_source=roadmap&utm_medium=Referral&utm_campaign=TDS+roadmap+integration) 를 참조하여 작성하였으며, 확정성과 재사용성을 위해선 함수화가 더 필요합니다.

```PowerShell
mlflow ui
```
이후 http://127.0.0.1:5000/ 에서 실험 결과를 볼 수 있습니다.

## 모델 배포
```app.py
from fastapi import FastAPI
from pydantic import BaseModel
import pandas as pd
import joblib

app = FastAPI()

bundle = joblib.load("model.pkl")
model = bundle["model"]
le = bundle["label_encoder"]

class InputData(BaseModel):
    id: int
    Soil_Type: str
    Soil_pH: float
    Soil_Moisture: float
    Organic_Carbon: float
    Electrical_Conductivity: float
    Temperature_C: float
    Humidity: float
    Rainfall_mm: float
    Sunlight_Hours: float
    Wind_Speed_kmh: float
    Crop_Type: str
    Crop_Growth_Stage: str
    Season: str
    Irrigation_Type: str
    Water_Source: str
    Field_Area_hectare: float
    Mulching_Used: str
    Previous_Irrigation_mm: float
    Region: str

@app.get("/")
async def read_root():
    return {"health_check": "OK", "model_version": "0.1dev"}

@app.post("/predict")
def predict(data: InputData):
    df = pd.DataFrame([data.dict()])

    pred_num = model.predict(df)
    pred_label = le.inverse_transform(pred_num)[0]

    return {
        "prediction": pred_label
    }
```
app.py라는 새로운 파일을 만듭니다.
```PowerShell
uvicorn app:app --reload
```
위의 명령어를 터미널에 입력후

```https
http://127.0.0.1:8000/docs.
```
위 링크로 들어가 모델이 제대로 배포되었는지 확인합니다.

![image](https://github.com/cromi0256/core/blob/main/Operations/Project03dev/%ED%99%94%EB%A9%B4%20%EC%BA%A1%EC%B2%98%202026-04-18%20223519.png)
label까지 제대로 출력이 되는군요
## 도커 
```
pip freeze > requirements.txt
```
다시 한번 가상환경에서 requirements.txt를 업데이트 합니다.

[Docker](https://www.docker.com/)에서 다운받습니다.
```Dockerfile
FROM python:3.13-slim

#set the working directory 
WORKDIR /app

#add app.py and models directory
COPY app.py .
COPY model.pkl .

# add requirements file
COPY requirements.txt .

# install python libraries
RUN pip install --no-cache-dir -r requirements.txt

# specify default commands
CMD ["uvicorn", "app:app", "--host", "0.0.0.0", "--port", "80"]
```
파이썬 버전, 실행 위치, 모델과 배포파일, pip패키지를 설정합니다.

```bash
docker build -t <도커이미지 파일이름> <도커파일위치>
```
위의 명령어를 터미널에 실행하여 도커이미지를 만듭니다.

![image](https://github.com/cromi0256/core/blob/main/Operations/Project03dev/%ED%99%94%EB%A9%B4%20%EC%BA%A1%EC%B2%98%202026-04-19%20032839.png)
도커 프로그램에서도 확인할 수 있습니다.

```PowerShell
docker tag <도커이미지> <dockerhub_username>/<docker-repo-name>
```
만든 도커이미지를 도커허브와 연결합니다.

```PowerShell
docker push <dockerhub_username>/<docker-repo-name>:latest
```
깃허브로 push하듯 도커허브로 push합니다.

## 클라우드 배포
[아마존ECS](https://ap-southeast-2.console.aws.amazon.com/ecs/v2/clusters?region=ap-southeast-2)
위 링크로 접속하여 계정 로그인 합니다.

Amazon Elastic Container Service 에 접속하여 Express Mode로 배포합니다.(10분 정도 소요)
![image](https://github.com/cromi0256/core/blob/main/Operations/Project03dev/%ED%99%94%EB%A9%B4%20%EC%BA%A1%EC%B2%98%202026-04-20%20004314.png)
배포가 잘 되었다면 머신러닝 글로벌 배포가 완료된 겁니다.

## 모니터링
```PowerShell
pip install evidently
```
evidently AI 패키지를 설치합니다.

```py
# 패키지 로드
import pandas as pd
from evidently import Dataset
from evidently import DataDefinition
from evidently import Report
from evidently.presets import DataDriftPreset, DataSummaryPreset

# 데이터로드 & 스키마 지정
reference = pd.read_csv('train.csv')
production = pd.read_csv('test.csv')

cat_cols = reference.select_dtypes(include=['object']).columns.drop(['Irrigation_Need']).tolist()
num_cols = reference.select_dtypes(include=['int64', 'float64']).columns.drop(['id']).tolist()

schema = DataDefinition(
    numerical_columns=num_cols,
    categorical_columns=cat_cols,
)

# 데이터 평가
eval_data_1 = Dataset.from_pandas(
    pd.DataFrame(production),
    data_definition=schema
)

eval_data_2 = Dataset.from_pandas(
    pd.DataFrame(reference),
    data_definition=schema
)
report = Report([
    DataDriftPreset() 
])

my_eval = report.run(eval_data_1, eval_data_2)

print(my_eval)    # 데이터 드리프트 확인
```
monitor라는 파일을 .py나 .ipynb로 작성하여 결과를 확인합니다.

![image](https://github.com/cromi0256/core/blob/main/Operations/Project03dev/%ED%99%94%EB%A9%B4%20%EC%BA%A1%EC%B2%98%202026-04-20%20221512.png)
train.csv와 test.csv는 데이터가 크게 다르지 않아 보입니다.
