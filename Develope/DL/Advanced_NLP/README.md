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

## 실제 작동방식
위의 pipeline은 사실 아래와 같은 과정을 거친다.

1. Tokenizer
```JupyterNotebook
from transformers import AutoTokenizer

checkpoint = "distilbert-base-uncased-finetuned-sst-2-english"
tokenizer = AutoTokenizer.from_pretrained(checkpoint)

raw_inputs = [
    "I've been waiting for a HuggingFace course my whole life.",
    "I hate this so much!",
]
inputs = tokenizer(raw_inputs, padding=True, truncation=True, return_tensors="pt")
print(inputs)


# 결과
{
    'input_ids': tensor([
        [  101,  1045,  1005,  2310,  2042,  3403,  2005,  1037, 17662, 12172, 2607,  2026,  2878,  2166,  1012,   102],
        [  101,  1045,  5223,  2023,  2061,  2172,   999,   102,     0,     0,     0,     0,     0,     0,     0,     0]
    ]), 
    'attention_mask': tensor([
        [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
        [1, 1, 1, 1, 1, 1, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0]
    ])
}
```
입력한 문장을 토큰화하여 컴퓨터가 이해할 수 있는 텐서로 변환한다

2. 모델 선택
```JupyterNotebook
from transformers import AutoModel

checkpoint = "distilbert-base-uncased-finetuned-sst-2-english"
model = AutoModel.from_pretrained(checkpoint)
```
모델은 토큰화를 거친 텐서를 고차원 벡터로 변환한다

torch.Size([2, 16, 768])의 텐서로 변환되며, (배치, 시퀀스길이, 은닉상태)로 출력된다

![image](https://huggingface.co/datasets/huggingface-course/documentation-images/resolve/main/en/chapter2/transformer_and_head-dark.svg)

그렇지만 특정 작업을 원한다면 알맞은 아키텍쳐를 선택한다

```JupyterNotebook
from transformers import AutoModelForSequenceClassification

checkpoint = "distilbert-base-uncased-finetuned-sst-2-english"
model = AutoModelForSequenceClassification.from_pretrained(checkpoint)
outputs = model(**inputs)
```

이는 로짓 값을 출력하는 텐서(배치, 로짓)로 변환한다

3. 후처리
```JupyterNotebook
import torch

predictions = torch.nn.functional.softmax(outputs.logits, dim=-1)
print(predictions)
```
로짓 값을 확률로 변환한다

```
# 로짓
tensor([[-1.5607,  1.6123],
        [ 4.1692, -3.3464]], grad_fn=<AddmmBackward>)

# 확률
tensor([[4.0195e-02, 9.5980e-01],
        [9.9946e-01, 5.4418e-04]], grad_fn=<SoftmaxBackward>)
```
이후 label을 붙여 최종값을 출력한다(label, score)

## 파인튜닝
이 과정엔 파이토치 지식을 상당히 요구한다

[허깅페이스](https://huggingface.co/learn/llm-course/chapter3/4) 강좌 참조

전이학습과 비슷한 방법을 따르면서도 효율적인 학습과 GPU 최적화도 필요하다

위 링크의 방법은 전체 아키텍쳐를 학습시키기에 상당히 오래걸린다

대안적인 방법(PEFT)으로 LoRA, Prompt tuning 등이 있다

## 참고링크
[허깅페이스](https://huggingface.co/learn/llm-course/chapter0/1)
