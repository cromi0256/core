거대언어모델은 날마다 새로워지고 향상되고 있다.

1년이 지난 모델은 금방 세대교체가 되기도 하고 오픈소스 모델이 클로즈드 모델로 바뀌기도 한다.

모델의 가중치에 접근할 수 있는지의 유무에 따라 모델 사용이 구분된다.

오픈소스 모델 :
- 대표적 모델 : Meta Llama, DeepSeek, Qwen, Gemma
- 배포 : 로컬 혹은 자체서버
- 장점 : 온프레미스 환경에서 실행가능, 커스터마이징 수준이 높음, 토큰 비용 통제 가능
- 단점 : 개인 GPU에 의존, 클로즈드(프론티어) 모델에 비해 성능이 낮음, 모델 교체가 번거로움

클로즈드 모델 : 
- 대표적 모델 : Antropic Claude, Google Gemini, OpenAI, Cohere, Mistral
- 배포 : API
- 장점 : 최고 수준의 모델, 인프라 관리가 필요없음, 빠른 업데이트
- 단점 : 토큰 정책 비용에 예민함, 인터넷 연결이 필요하며 데이터가 전송됨으로써 유출 위험도 존재

또한 Task에 따라 모델의 성능 또한 제각각 다르므로 리더보드를 확인해 보는것도 좋다

[허깅페이스](https://huggingface.co/models)

[리더보드 링크](https://arena.ai/leaderboard)

# 코드 예시
[코드출처](https://bentoml.com/llm/model-interaction/openai-compatible-api)

대부분의 모델은 코드 작성 방식이 비슷하다

OpenAI와 호환되는 API로 다른 모델에서도 작동한다

```python
from openai import OpenAI

# Use your custom endpoint URL and API key
client = OpenAI(
    base_url="https://your-custom-endpoint.com/v1",
    api_key="your-api-key"
)

response = client.chat.completions.create(
    model="your-model-name",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "How can I integrate OpenAI-compatible APIs?"}
    ]
)

print(response.choices[0].message)
```
