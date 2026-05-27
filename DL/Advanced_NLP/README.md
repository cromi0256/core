## LLM 이전
과거 자연어처리엔 규칙/통계 기반 모델을 시작으로 딥러닝 RNN/LSTM에 의존했다.

이는 여러 한계점이 있었으며, 낮은 성능으로 세상의 주목을 받진 못 했다.

2017년 트랜스포머의 등장으로 수십억개의 매개변수를 가진 거대언어모델이 만들어지기 시작했다.

## 트랜스포머 기반 LLM
패키지 설치
```PowerShell
pip install "transformers[torch]"
```

감성분석
```JupyterNotebook
from transformers import pipeline

classifier = pipeline("sentiment-analysis")
classifier(
    ["I've been waiting for a HuggingFace course my whole life.", "I hate this so much!"]
)

# 결과
# [{'label': 'POSITIVE', 'score': 0.9598049521446228},
#  {'label': 'NEGATIVE', 'score': 0.9994558691978455}]
```
기본적으론 distilbert를 사용하며, task와 model 매개변수를 설정하여 허깅페이스 모델을 골라 사용할 수 있다.

```JupyterNotebook
from transformers import pipeline

generator = pipeline("text-generation", model="HuggingFaceTB/SmolLM2-360M")
generator(
    "In this course, we will teach you how to",
    max_new_tokens=25,
    num_return_sequences=3,
)
# 결과
# [{'generated_text': "In this course, we will teach you how to write your own program to solve problems. You'll learn how to use the following tools:\n\n• Python for beginners\n"},
#  {'generated_text': 'In this course, we will teach you how to build your own robot that can move around and find its own way back to the place where it was started. We will start'},
#  {'generated_text': 'In this course, we will teach you how to solve multiple linear regression problems. We will start with the basics and gradually progress to more complex techniques. We will also explore the'}]
```
트랜스포머와 허깅페이스 모델에 더 알고 싶다면 [공식 튜토리얼](https://huggingface.co/learn/llm-course/chapter1/6)을 참고한다.



## 참고링크
[허깅페이스](https://huggingface.co/learn/llm-course/chapter0/1)
