[랭체인아카데미](https://academy.langchain.com/)에서 배운 것을 정리합니다.

![Image](https://mintcdn.com/langchain-5e9cc07a/jtty0O--UJOKG0nK/oss/images/agent_model_harness.svg?w=1650&fit=max&auto=format&n=jtty0O--UJOKG0nK&q=85&s=206e8f37e9b0ddf8dde21270c1e1d333)

1. 랭체인 : 간단한 에이전트 프레임워크
2. 랭그래프 : 낮은 단계의 오케스트레이션 제작
3. 딥에이전트 : 복잡한 에이전트 하네스 구축
4. 랭스미스 : 에이전 테스트, 배포, 모니터링

# 랭체인

0. 준비사항

시스템 프롬프트, LLM 모델 API키(호출 필요), 도구, 메모리

```python
# 간단요약
agent = create_agent(
    model=model,
    tools=[tool],
    system_prompt=SYSTEM_PROMPT,
    checkpointer=checkpointer,
)
```

1. 파운데이션 모델

```python
from langchain.agents import create_agent

model = init_chat_model(model="gpt-5-nano")

response = model.invoke("What's the capital of the Moon?")

print(response.content)

# 스트리밍
for token, metadata in agent.stream(
    {"messages": [HumanMessage(content="Tell me all about Luna City, the capital of the Moon")]},
    stream_mode="messages"
):

    # token is a message chunk with token content
    # metadata contains which node produced the token
    
    if token.content:  # Check if there's actual content
        print(token.content, end="", flush=True)  # Print token
```

2. 도구

```python
from langchain.tools import tool

@tool
def square_root(x: float) -> float:
    """Calculate the square root of a number"""
    return x ** 0.5

# 데코레이터, 설명, 어노테이션 작성
```

3. 단기 메모리

```python
from langgraph.checkpoint.memory import InMemorySaver  


agent = create_agent(
    "gpt-5-nano",
    checkpointer=InMemorySaver(),  
)

# thread_id로 구분
config = {"configurable": {"thread_id": "1"}}
```

4. +멀티모달

```python
# 이미지 입력
import base64

uploader = FileUpload(accept='.png', multiple=False)
# Get the first (and only) uploaded file dict
uploaded_file = uploader.value[0]

# This is a memoryview
content_mv = uploaded_file["content"]

# Convert memoryview -> bytes
img_bytes = bytes(content_mv)  # or content_mv.tobytes()

# Now base64 encode
img_b64 = base64.b64encode(img_bytes).decode("utf-8")

multimodal_question = HumanMessage(content=[
    {"type": "text", "text": "Tell me about this capital"},
    {"type": "image", "base64": img_b64, "mime_type": "image/png"}
])

# 오디오 입력
audio = sd.rec(int(duration * sample_rate), samplerate=sample_rate, channels=1)
# Progress bar for the duration
for _ in tqdm(range(duration * 10)):   # update 10× per second
    time.sleep(0.1)
sd.wait()
print("Done.")

# Write WAV to an in-memory buffer
buf = io.BytesIO()
write(buf, sample_rate, audio)
wav_bytes = buf.getvalue()

aud_b64 = base64.b64encode(wav_bytes).decode("utf-8")

multimodal_question = HumanMessage(content=[
    {"type": "text", "text": "Tell me about this audio file"},
    {"type": "audio", "base64": aud_b64, "mime_type": "audio/wav"}
])
```

5. MCP

[로컬 서버](https://github.com/langchain-ai/lca-lc-foundations/blob/main/notebooks/module-2/resources/2.1_mcp_server.py)

[원격 서버](https://mcp.so/servers)
```python
# 로컬 서버
from langchain_mcp_adapters.client import MultiServerMCPClient

client = MultiServerMCPClient(
    {
        "local_server": {
                "transport": "stdio",
                "command": "python",
                "args": ["resources/2.1_mcp_server.py"],
            }
    }
)

# 원격 서버
client = MultiServerMCPClient(
    {
        "time": {
            "transport": "stdio",
            "command": "uv",
            "args": [
                "run",
                "python",
                "-m",
                "mcp_server_time",
                "--local-timezone=America/New_York"
            ]
        }
    }
)

tools = await client.get_tools()
```

6. 컨텍스트

```python
from dataclasses import dataclass

@dataclass
class ColourContext:
    favourite_colour: str = "blue"
    least_favourite_colour: str = "yellow"

response = agent.invoke(
    {"messages": [HumanMessage(content="What is my favourite colour?")]},
    context=ColourContext(favourite_colour="green")
)
```

7. 상태

```python
from langchain.agents import AgentState

class CustomState(AgentState):
    favourite_colour: str

# 쓰기
from langchain.tools import tool, ToolRuntime
from langgraph.types import Command
from langchain.messages import ToolMessage

@tool
def update_favourite_colour(favourite_colour: str, runtime: ToolRuntime) -> Command:
    """Update the favourite colour of the user in the state once they've revealed it."""
    return Command(update={
        "favourite_colour": favourite_colour, 
        "messages": [ToolMessage("Successfully updated favourite colour", tool_call_id=runtime.tool_call_id)]}
        )

# 읽기
@tool
def read_favourite_colour(runtime: ToolRuntime) -> str:
    """Read the favourite colour of the user from the state."""
    try:
        return runtime.state["favourite_colour"]
    except KeyError:
        return "No favourite colour found in state"

agent = create_agent(
    "gpt-5-nano",
    tools=[update_favourite_colour, read_favourite_colour],
    checkpointer=InMemorySaver(),
    state_schema=CustomState
)
```

8. 멀티 에이전트

서브 에이전트 생성 > 도구로 래핑 > 메인 에이전트에 도구로 호출
```python
from langchain.tools import tool
from langchain.agents import create_agent

# create subagents

subagent_1 = create_agent(
    model='gpt-5-nano',
    tools=[square_root]
)

@tool
def call_subagent_1(x: float) -> float:
    """Call subagent 1 in order to calculate the square root of a number"""
    response = subagent_1.invoke({"messages": [HumanMessage(content=f"Calculate the square root of {x}")]})
    return response["messages"][-1].content

## Creating the main agent

main_agent = create_agent(
    model='gpt-5-nano',
    tools=[call_subagent_1, call_subagent_2],
    system_prompt="You are a helpful assistant who can call subagents to calculate the square root or square of a number.")
```

9. +RAG

파싱 > 청킹 > 임베딩 > 벡터 저장 > 검색 설정 > 프롬프트 생성
```python
from langchain_community.document_loaders import PyPDFLoader

loader = PyPDFLoader("resources/acmecorp-employee-handbook.pdf")

data = loader.load()

from langchain_text_splitters import RecursiveCharacterTextSplitter

text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000, chunk_overlap=200, add_start_index=True
)
all_splits = text_splitter.split_documents(data)

print(len(all_splits))

from langchain_openai import OpenAIEmbeddings

embeddings = OpenAIEmbeddings(model="text-embedding-3-large")

from langchain_core.vectorstores import InMemoryVectorStore

vector_store = InMemoryVectorStore(embeddings)

from langchain.tools import tool

@tool
def search_handbook(query: str) -> str:
    """Search the employee handbook for information"""
    results = vector_store.similarity_search(query)
    return results[0].page_content
```

10. 긴 대화 관리

# 추가자료
[로드맵](https://roadmap.sh/ai-agents)

[랭체인 교육자료 github](https://github.com/langchain-ai/lca-lc-foundations) 
