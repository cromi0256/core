### 들어가며
기술과 코드는 빠르게 변화하며 기껏 공부한 코드는 최신 버전과 종속성 문제로 쓰이지 못 할 수 있다.

그러므로 기술을 익히되 공식문서를 참고하고, 유지보수가 용이한 패키지 위주로 설계한다.

언제든 최신의 정보를 AI로 쉽고 빠르게 배울 수 있는 세상이다.

# MLOps 기술 스택
![Image](https://docs.cloud.google.com/static/architecture/images/mlops-continuous-delivery-and-automation-pipelines-in-machine-learning-1-elements-of-ml.png?hl=ko)
## 1. 프로그래밍 기초
전체적인 기술을 다루면서도 다른 기술을 배우기 위한 기초가 된다.
1. 파이썬 - 코드 설계
* [공식문서](https://docs.python.org/ko/3/tutorial/index.html)
* [추천로드맵](https://roadmap.sh/python)
2. SQL - 데이터 파이프라인
* [PostgreSQL 공식문서](https://www.postgresql.org/docs/current/)
* [추천로드맵](https://roadmap.sh/sql)
* [추천실습(유료)](https://roadmap.sh/courses/sql)
3. Shell/Bash - 운영체제와 터미널
* [Bash 공식문서](https://www.gnu.org/software/bash/manual/bash.html)
* [추천로드맵](https://roadmap.sh/shell-bash)

## 2. 버전 컨트롤
파일의 변화를 기록하고 추적한다.
1. Git & GitHub
* [Git 공식문서](https://git-scm.com/book/ko/v2)
* [GitHub 공식문서](https://docs.github.com/ko)
* [추천로드맵](https://roadmap.sh/git-github)

## 3. CI/CD
지속적 통합 및 지속적 배포로 빌드, 테스트, 릴리즈를 간소화하고 자동화한다.
1. GitHub Actions
* [공식문서](https://docs.github.com/ko/actions)
* 

## 4. 머신러닝 기초
데이터 전처리, 특성 선택, 모델 선택, 모델 학습, 평가 지표, 과적합 방지가 필요하다.
1. 수학과 통계
* 미적분학, 통계학, 선형대수학 학부수준의 지식
* [추천도서-기초통계학 e-book](https://assets.openstax.org/oscms-prodcms/media/documents/IntroductoryStatistics-OP_i6tAI7e.pdf)
2. 머신러닝
* [사이킷런 공식문서](https://scikit-learn.org/stable/user_guide.html)
* [추천로드맵](https://roadmap.sh/machine-learning)
* [추천실습](https://www.kaggle.com/learn)
3. 딥러닝
* [파이토치 공식문서](https://docs.pytorch.org/docs/stable/index.html)
* [추천로드맵(위와 동일)](https://roadmap.sh/machine-learning)
4. 모델 평가
* [MLFlow 공식문서](https://mlflow.org/docs/latest/ml/)

## 5. 클라우드 컴퓨팅
AWS, Azure, GCP 등 인프라(서버, 스토리지, 네트워킹, SDK, 소프트웨어)를 빌려 운영을 담당한다.
또한, 독자적인 ML서비스(AWS-SageMaker, GCP-Gemini Enterprise Agent Platform(구)Vertex AI, Azure ML)도 제공한다.
* 이 아래부터 백엔드 지식도 필요해진다.
* [GCP 공식문서](https://docs.cloud.google.com/docs?hl=ko)
* [Vertex AI 공식문서](https://cloud.google.com/products/gemini-enterprise-agent-platform?hl=ko)
* [추천실습](https://www.skills.google/paths/8)

## 6. 컨테이너화
앱의 코드와 실행에 필요한 모든 파일, 라이브러리, 환경 설정을 하나로 묶어 '컨테이너'라는 독립된 단위로 만든다. 개발환경과 같은 설정의 배포로 안정성을 향상한다.
1. Docker
* [공식문서](https://docs.docker.com/)
* [추천로드맵](https://roadmap.sh/docker)
2. Kubernetes(k9s)
* [공식문서](https://kubernetes.io/docs/home/)
* [추천로드맵](https://roadmap.sh/kubernetes)

## 7. 데이터 엔지니어링 기초
