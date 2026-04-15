이 프로젝트는 kaggle의 데이터셋을 받아 실행하였습니다.

**데이터셋 정보**

[Kaggle]https://www.kaggle.com/competitions/playground-series-s6e4

자세한 정보는 위 링크에서 확인

## 프로젝트 셋업
playground-series-s6e4 에서 시작

시작 전에 python3.13.13 버전을 설치

VSCode에서 실행

1. 파이썬 가상환경 설정

```py -3.13 -m venv venv```

가상환경 설치

```venv\Scripts\activate```

가상환경 활성화

```python --version```

(파이썬 버전 확인)

2. 패키지 설치

```pip install -U scikit-learn```

사이킷런 설치

3. git 연동
```git init```

git과 연결

(.gitignore은 이미 설정되어 있지만 필요에 따라 생성)

```pip freeze > requirements.txt```

패키지 목록 저장

```
git add .
git commit -m "첫 커밋"
```

스테이징 후 첫 커밋

(첫 실행시 git 이름이나 이메일을 설정)

## 모델 훈련
