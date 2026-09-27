[랭체인아카데미](https://academy.langchain.com/)에서 배운 것을 정리합니다.

최신 버전과 차이가 있으며 아래 작성된 코드는 변경 되었을 수 있습니다.

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

```python
from langchain.agents import create_agent
from langgraph.checkpoint.memory import InMemorySaver
from langchain.agents.middleware import SummarizationMiddleware

# 요약
agent = create_agent(
    model="gpt-5-nano",
    checkpointer=InMemorySaver(),
    middleware=[
        SummarizationMiddleware(
            model="gpt-4o-mini",
            trigger=("tokens", 100),
            keep=("messages", 1)
        )
    ],
)

# 절단/삭제
from typing import Any
from langchain.agents import AgentState
from langchain.messages import RemoveMessage
from langgraph.runtime import Runtime
from langchain.agents.middleware import before_agent
from langchain.messages import ToolMessage

@before_agent
def trim_messages(state: AgentState, runtime: Runtime) -> dict[str, Any] | None:
    """Remove all the tool messages from the state"""
    messages = state["messages"]

    tool_messages = [m for m in messages if isinstance(m, ToolMessage)]
    
    return {"messages": [RemoveMessage(id=m.id) for m in tool_messages]}
```

11. HITL

```python
from langchain.agents import create_agent, AgentState
from langgraph.checkpoint.memory import InMemorySaver
from langchain.agents.middleware import HumanInTheLoopMiddleware

class EmailState(AgentState):
    email: str

agent = create_agent(
    model="gpt-5-nano",
    tools=[read_email, send_email],
    state_schema=EmailState,
    checkpointer=InMemorySaver(),
    middleware=[
        HumanInTheLoopMiddleware(
            interrupt_on={
                "read_email": False,
                "send_email": True,
            },
            description_prefix="Tool execution requires approval",
        ),
    ],
)

# 승인
response = agent.invoke(
    Command( 
        resume={"decisions": [{"type": "approve"}]}
    ), 
    config=config # Same thread ID to resume the paused conversation
)

# 거절
response = agent.invoke(
    Command(        
        resume={
            "decisions": [
                {
                    "type": "reject",
                    # An explanation of why the request was rejected
                    "message": "No please sign off - Your merciful leader, Seán."
                }
            ]
        }
    ), 
    config=config # Same thread ID to resume the paused conversation
    )   

# 수정
response = agent.invoke(
    Command(        
        resume={
            "decisions": [
                {
                    "type": "edit",
                    # Edited action with tool name and args
                    "edited_action": {
                        # Tool name to call.
                        # Will usually be the same as the original action.
                        "name": "send_email",
                        # Arguments to pass to the tool.
                        "args": {"body": "This is the last straw, you're fired!"},
                    }
                }
            ]
        }
    ), 
    config=config # Same thread ID to resume the paused conversation
    )   
```

12. 동적 에이전트

```python
# 동적 모델
from langchain.agents.middleware import wrap_model_call, ModelRequest, ModelResponse
from langchain.chat_models import init_chat_model
from typing import Callable

large_model = init_chat_model("claude-sonnet-4-6")
standard_model = init_chat_model("gpt-5-nano")


@wrap_model_call
def state_based_model(request: ModelRequest, 
handler: Callable[[ModelRequest], ModelResponse]) -> ModelResponse:
    """Select model based on State conversation length."""
    # request.messages is a shortcut for request.state["messages"]
    message_count = len(request.messages)  

    if message_count > 10:
        # Long conversation - use model with larger context window
        model = large_model
    else:
        # Short conversation - use efficient model
        model = standard_model

    request = request.override(model=model)  

    return handler(request)

# 동적 프롬프트
from dataclasses import dataclass
from langchain.agents.middleware import dynamic_prompt, ModelRequest

@dataclass
class LanguageContext:
    user_language: str = "English"

@dynamic_prompt
def user_language_prompt(request: ModelRequest) -> str:
    """Generate system prompt based on user role."""
    user_language = request.runtime.context.user_language
    base_prompt = "You are a helpful assistant."

    if user_language != "English":
        return f"{base_prompt} only respond in {user_language}."
    elif user_language == "English":
        return base_prompt

# 동적 도구
from langchain.agents.middleware import wrap_model_call, ModelRequest, ModelResponse
from typing import Callable

@wrap_model_call
def dynamic_tool_call(request: ModelRequest, 
handler: Callable[[ModelRequest], ModelResponse]) -> ModelResponse:

    """Dynamically call tools based on the runtime context"""

    user_role = request.runtime.context.user_role
    
    if user_role == "internal":
        pass # internal users get access to all tools
    else:
        tools = [web_search] # external users only get access to web search
        request = request.override(tools=tools) 

    return handler(request)
```

# 추가자료
[로드맵](https://roadmap.sh/ai-agents)

[랭체인 교육자료 github](https://github.com/langchain-ai/lca-lc-foundations) 
