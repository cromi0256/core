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
import logging
import mlflow
from sklearn.preprocessing import OrdinalEncoder, LabelEncoder
from sklearn.model_selection import train_test_split
from sklearn.ensemble import HistGradientBoostingClassifier
from sklearn.metrics import classification_report
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
        cat_cols = ['Soil_Type', 'Crop_Type', 'Crop_Growth_Stage', 'Season', 'Irrigation_Type', 'Water_Source', 'Mulching_Used', 'Region']
        X = train_df.drop(['id','Irrigation_Need'], axis=1)
        Oe = OrdinalEncoder()
        Le = LabelEncoder()
        X[cat_cols] = Oe.fit_transform(X[cat_cols])
        y = train_df['Irrigation_Need']
        y = Le.fit_transform(y)
        X_test = test_df.drop(['id'], axis=1)
        X_test[cat_cols] = Oe.transform(X_test[cat_cols])
        logging.info("Data cleaning completed successfully")

        # Model Train
        X_train, X_val, y_train, y_val = train_test_split(X, y, test_size=0.2, random_state=42, stratify=y)
        params = {
        "random_state" : 42,
        }
        hgbc = HistGradientBoostingClassifier(**params)
        hgbc.fit(X_train, y_train)

        # Save Model
        with open("filename.pkl", "wb") as f:
            dump(hgbc, f, protocol=5)
        logging.info("Model training completed successfully")

        # Model Evaluate
        y_pred = hgbc.predict(X_val)
        report = classification_report(y_val, y_pred, output_dict=True)
        print_report = classification_report(y_val, y_pred)
        logging.info("Model evaluation completed successfully")

        # Tags 
        mlflow.set_tag('Model developer', '김영환')
        mlflow.set_tag('preprocessing', 'OrdinalEncoder')

        # Log Metrics
        mlflow.log_metric('f1-score', report['weighted avg']['f1-score'])
        mlflow.sklearn.log_model(hgbc, 'model')

        # Register the model
        model_name = "HGB_model" 
        model_uri = f"runs:/{run.info.run_id}/model"
        mlflow.register_model(model_uri, model_name)

        logging.info("MLflow tracking completed successfully")

        # Print evaluation results
        print("n============= Model Evaluation Results ==============")
        print(f"Model: {model_name}")
        print(f"n{print_report}")
        print("=====================================================n")

if __name__ == "__main__":
    main()
```
[link](https://towardsdatascience.com/machine-learning-operations-mlops-for-beginners-a5686bfe02b2/?utm_source=roadmap&utm_medium=Referral&utm_campaign=TDS+roadmap+integration) 를 참조하여 작성하였으며, 확정성과 재사용성을 위해선 함수화가 더 필요합니다.

```PowerShell
mlflow ui
```
이후 http://127.0.0.1:5000/ 에서 실험 결과를 볼 수 있습니다.

## 모델 배포
