2026-11-25 002 TFM(Tabular Foundation Model)
# TFM - KumoTabular
[아카이브](https://arxiv.org/abs/2611.23001)

[공식문서](https://docs.nvidia.com/sdgm/structured-data-models/models/kumo-tabular)

[API](https://nvidia.github.io/structured-data-models/api/generated/sdm.models.KumoTabular.html)

# 캐글 실습

실습 데이터셋 : https://www.kaggle.com/competitions/playground-series-s6e11

[실습코드](https://www.kaggle.com/code/kimyounghwan24/notebookb4a1f04a98)

5만개의 샘플 train으로 auc=0.65115달성 (이전 AutoML보다 점수가 낮다)

# Kumo Taublar이란
[출처](https://docs.nvidia.com/sdgm/about-nvidia-kumo/in-context-learning)

Kumo Tabular는 “컨텍스트 기반 학습(in-context learning)” 모델이라고 불리는 특별한 종류의 기초 모델에 속한다.

훈련 데이터셋을 사용하여 트리나 MLP를 학습시키는 기존 모델과 달리, 컨텍스트 기반 학습은 교대로 배치된 어텐션 레이어를 활용하여 일련의 컨텍스트 예시로부터 직접 예측을 수행한다.

이 모델은 단 한 번만 훈련되면 여러 데이터셋에 재사용할 수 있다.

컨텍스트 기반 학습 모델은 일반적으로 수백만 개의 시뮬레이션된 데이터셋을 통해 훈련되므로, 불완전하거나 누락된 데이터에 대해 매우 견고할 뿐만 아니라, 소수의 컨텍스트 예시만으로도 정확한 예측을 수행하는 데 매우 뛰어나다.

# Kumo를 고른 이유
[TabPFN](https://docs.nvidia.com/sdgm/about-nvidia-kumo/in-context-learning)을 시초로 TFM 모델이 폭발적으로 늘어났지만 대부분 라이선스 비용을 지불해야 하며, 가중치가 공개되어 있지 않다.

TabICL과 Kumo는 TFM모델 중에서 가중치가 공개되어 있으며 영리적 사용이 가능했기에 선택했다.

Tabular 뿐만이 아닌 Anomaly, Relation, Forecast, Transaction등 다양한 솔루션을 위한 모델도 존재한다.

# Kumo Tabular 사용법
```bash
pip install structured-data-models
```

```python
import torch
from sklearn.datasets import load_breast_cancer

import sdm

df = load_breast_cancer(as_frame=True).frame
device = torch.device("cuda" if torch.cuda.is_available() else "cpu")

table = sdm.TableTensor.from_pandas(
    df=df,
    stypes=sdm.infer_stypes(df, overrides={"target": "categorical"}),
    device=device,
)
model = sdm.models.KumoTabular(task="classification", device=device)

with torch.amp.autocast(
    device.type,
    dtype=torch.float16,
    enabled=device.type == "cuda",
):
    probs = model(
        x_context=table[:300].drop_columns("target"),
        y_context=table[:300, "target"],
        x_query=table[300:].drop_columns("target"),
        num_estimators=8,
    )

print(probs)
```

# 주의점
GPU와 CUDA를 사용하기에 몇가지 작업이 필요하다:

* 파이썬 3.11 이후 버전
* CPU로도 작동하나 큰 데이터엔 GPU가 필수
* 가중치 다운로드를 위한 네트워킹 필요

# 작동방식
![image](https://nvidia.github.io/structured-data-models/_images/kumo_tabular.svg)

[모범 사례](https://docs.nvidia.com/sdgm/structured-data-models/best-practices/kumo-tabular)

# 한계점
프로덕션 환경에 안정적으로 적용하려면 결국 GPU를 필요로 하므로 비용 부담이 커진다.

[https://huggingface.co/spaces/TabArena/leaderboard?tab=beyondarena]IID가정이 깨지더라도 대체로 트리 모델 계열보다 나은 성능을 보인다.

그렇지만 무작정 첫모델로 적용하기보다 가벼운 모델로 먼저 실험을 해본 후 이 모델을 적용하는 편이 좋다.

하이퍼 파라미터를 조절하지 않아도 되지만, 원활한 GPU 활용을 위해 `모범 사례` 문서를 살펴보며 조정해야 한다.

모델 해석 기능은 아직 추가되지 않았으며(experimental), 더 나은 성능을 위한 특성 엔지니어링은 여전히 사람이 해야한다.