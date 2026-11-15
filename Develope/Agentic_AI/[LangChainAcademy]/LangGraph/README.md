[랭체인아카데미](https://academy.langchain.com/)에서 배운 것을 정리합니다.

최신 버전과 차이가 있으며 아래 작성된 코드는 변경 되었을 수 있습니다.

![Image](https://mintcdn.com/langchain-5e9cc07a/-_xGPoyjhyiDWTPJ/oss/images/agent_workflow.png?fit=max&auto=format&n=-_xGPoyjhyiDWTPJ&q=85&s=c217c9ef517ee556cae3fc928a21dc55)

# 랭그래프

1. 그래프

그래프 : 상태(데이터), 노드(작업), 엣지(연결)
```python
from langgraph.graph import StateGraph, MessagesState, START, END

def mock_llm(state: MessagesState):
    return {"messages": [{"role": "ai", "content": "hello world"}]}

graph = StateGraph(MessagesState)
graph.add_node(mock_llm)
graph.add_edge(START, "mock_llm")
graph.add_edge("mock_llm", END)
graph = graph.compile()

graph.invoke({"messages": [{"role": "user", "content": "hi!"}]})
```

2. 에이전트

모델&도구 정의 > 상태 정의 > 노드 정의 > 엣지 정의 > 에이전트 결합
```python
# Step 1: Define tools and model

from langchain.tools import tool
from langchain.chat_models import init_chat_model


model = init_chat_model(
    "claude-sonnet-4-6",
    temperature=0
)


# Define tools
@tool
def multiply(a: int, b: int) -> int:
    """Multiply `a` and `b`.

    Args:
        a: First int
        b: Second int
    """
    return a * b


@tool
def add(a: int, b: int) -> int:
    """Adds `a` and `b`.

    Args:
        a: First int
        b: Second int
    """
    return a + b


@tool
def divide(a: int, b: int) -> float:
    """Divide `a` and `b`.

    Args:
        a: First int
        b: Second int
    """
    return a / b


# Augment the LLM with tools
tools = [add, multiply, divide]
tools_by_name = {tool.name: tool for tool in tools}
model_with_tools = model.bind_tools(tools)

# Step 2: Define state

from langchain.messages import AnyMessage
from typing_extensions import TypedDict, Annotated
import operator


class MessagesState(TypedDict):
    messages: Annotated[list[AnyMessage], operator.add]
    llm_calls: int

# Step 3: Define model node
from langchain.messages import SystemMessage


def llm_call(state: MessagesState):
    """LLM decides whether to call a tool or not"""

    return {
        "messages": [
            model_with_tools.invoke(
                [
                    SystemMessage(
                        content="You are a helpful assistant tasked with performing arithmetic on a set of inputs."
                    )
                ]
                + state["messages"]
            )
        ],
        "llm_calls": state.get('llm_calls', 0) + 1
    }


# Step 4: Define tool node

from langchain.messages import ToolMessage


def tool_node(state: MessagesState):
    """Performs the tool call"""

    result = []
    for tool_call in state["messages"][-1].tool_calls:
        tool = tools_by_name[tool_call["name"]]
        observation = tool.invoke(tool_call["args"])
        result.append(ToolMessage(content=observation, tool_call_id=tool_call["id"]))
    return {"messages": result}

# Step 5: Define logic to determine whether to end

from typing import Literal
from langgraph.graph import StateGraph, START, END


# Conditional edge function to route to the tool node or end based upon whether the LLM made a tool call
def should_continue(state: MessagesState) -> Literal["tool_node", END]:
    """Decide if we should continue the loop or stop based upon whether the LLM made a tool call"""

    messages = state["messages"]
    last_message = messages[-1]

    # If the LLM makes a tool call, then perform an action
    if last_message.tool_calls:
        return "tool_node"

    # Otherwise, we stop (reply to the user)
    return END

# Step 6: Build agent

# Build workflow
agent_builder = StateGraph(MessagesState)

# Add nodes
agent_builder.add_node("llm_call", llm_call)
agent_builder.add_node("tool_node", tool_node)

# Add edges to connect nodes
agent_builder.add_edge(START, "llm_call")
agent_builder.add_conditional_edges(
    "llm_call",
    should_continue,
    ["tool_node", END]
)
agent_builder.add_edge("tool_node", "llm_call")

# Compile the agent
agent = agent_builder.compile()


from IPython.display import Image, display
# Show the agent
display(Image(agent.get_graph(xray=True).draw_mermaid_png()))

# Invoke
from langchain.messages import HumanMessage
messages = [HumanMessage(content="Add 3 and 4.")]
messages = agent.invoke({"messages": messages})
for m in messages["messages"]:
    m.pretty_print()
```

3. 상태

리듀서 : 상태를 기록하는 로직
```python
from typing import Annotated
from typing_extensions import TypedDict
from operator import add

# 1. 기본 제공 리듀서 사용 (예: operator.add를 사용하여 리스트 누적)
class State(TypedDict):
    foo: Annotated[list[int], add]

# 2. 커스텀 리듀서 정의 (예: None 처리 로직 포함)
def reduce_list(left: list | None, right: list | None) -> list:
    if not left: left = []
    if not right: right = []
    return left + right

class CustomState(TypedDict):
    foo: Annotated[list[int], reduce_list]
```

메시지 : 에이전트 대화 기록물 > 이전 대화를 수정하거나 변경할때
```python
from langchain_core.messages import RemoveMessage

# Message list
messages = [AIMessage("Hi.", name="Bot", id="1")]
messages.append(HumanMessage("Hi.", name="Lance", id="2"))
messages.append(AIMessage("So you said you were researching ocean mammals?", name="Bot", id="3"))
messages.append(HumanMessage("Yes, I know about whales. But what others should I learn about?", name="Lance", id="4"))

# Isolate messages to delete
delete_messages = [RemoveMessage(id=m.id) for m in messages[:-2]]
print(delete_messages)

add_messages(messages , delete_messages)
```

멀티 스키마 : 노드 간 내부 통신용 스키마와 외부 입출력용 스키마를 분리하여 제어할때
```python
# 내부 통신용
class OverallState(TypedDict):
    foo: int

class PrivateState(TypedDict):
    baz: int  # 내부 노드끼리만 공유하는 중간 키

def node_1(state: OverallState) -> PrivateState:
    return {"baz": state['foo'] + 1}

def node_2(state: PrivateState) -> OverallState:
    return {"foo": state['baz'] + 1}

# 외부 입출력용
class InputState(TypedDict):
    question: str

class OutputState(TypedDict):
    answer: str

class OverallState(TypedDict):
    question: str
    answer: str
    notes: str

graph = StateGraph(OverallState, input_schema=InputState, output_schema=OutputState)
```

4. 메모리

메모리 압축 : 긴 컨텍스트로 인한 추론 저하로 메시지를 줄여 메모리 확보
```python
# 메시지 삭제
from langchain_core.messages import RemoveMessage
from langgraph.graph import MessagesState, StateGraph, START, END

# 최근 2개의 메시지만 남기고 나머지 삭제
def filter_messages(state: MessagesState):
    delete_messages = [RemoveMessage(id=m.id) for m in state["messages"][:-2]]
    return {"messages": delete_messages}

def chat_model_node(state: MessagesState):
    return {"messages": [llm.invoke(state["messages"])]}


# 메시지 필터링
def chat_model_node(state: MessagesState):
    # 그래프 상태는 유지하고, 모델 호출 시에만 최근 1개의 메시지만 전달
    return {"messages": [llm.invoke(state["messages"][-1:])]}


# 메시지 트림
from langchain_core.messages import trim_messages
from langchain_openai import ChatOpenAI

def chat_model_node(state: MessagesState):
    # 토큰 수 100개를 초과하지 않도록 최근 메시지 위주로 자름
    messages = trim_messages(
        state["messages"],
        max_tokens=100,
        strategy="last",
        token_counter=ChatOpenAI(model="gpt-4o"),
        allow_partial=False,
    )
    return {"messages": [llm.invoke(messages)]}
```

외부 메모리 : db연결을 통한 메모리 영구저장
```python
import sqlite3
from langgraph.checkpoint.sqlite import SqliteSaver

# 로컬 DB 파일 연결
db_path = "state_db/example.db"
conn = sqlite3.connect(db_path, check_same_thread=False)

# Checkpointer 인스턴스 생성
memory = SqliteSaver(conn)

from typing_extensions import Literal
from langchain_openai import ChatOpenAI
from langchain_core.messages import SystemMessage, HumanMessage, RemoveMessage
from langgraph.graph import MessagesState, END

model = ChatOpenAI(model="gpt-4o", temperature=0)

# State 확장 (요약 정보 필드 추가)
class State(MessagesState):
    summary: str

# LLM 호출 노드 (기존 요약이 존재하면 시스템 메시지로 첨부)
def call_model(state: State):
    summary = state.get("summary", "")
    if summary:
        system_message = f"Summary of conversation earlier: {summary}"
        messages = [SystemMessage(content=system_message)] + state["messages"]
    else:
        messages = state["messages"]
    
    response = model.invoke(messages)
    return {"messages": response}

# 요약 생성 및 메시지 정리 노드
def summarize_conversation(state: State):
    summary = state.get("summary", "")
    if summary:
        summary_message = (
            f"This is summary of the conversation to date: {summary}\n\n"
            "Extend the summary by taking into account the new messages above:"
        )
    else:
        summary_message = "Create a summary of the conversation above:"

    messages = state["messages"] + [HumanMessage(content=summary_message)]
    response = model.invoke(messages)

    # 최근 2개 메시지만 남기고 이전 메시지 삭제
    delete_messages = [RemoveMessage(id=m.id) for m in state["messages"][:-2]]
    return {"summary": response.content, "messages": delete_messages}

# 조건부 분기 (메시지가 6개 초과 시 요약 실행)
def should_continue(state: State) -> Literal["summarize_conversation", END]:
    if len(state["messages"]) > 6:
        return "summarize_conversation"
    return END

from langgraph.graph import StateGraph, START

workflow = StateGraph(State)
workflow.add_node("conversation", call_model)
workflow.add_node(summarize_conversation)

workflow.add_edge(START, "conversation")
workflow.add_conditional_edges("conversation", should_continue)
workflow.add_edge("summarize_conversation", END)

# SQLite Checkpointer를 이용해 컴파일
graph = workflow.compile(checkpointer=memory)   # 여기에 메모리 삽입

# thread_id를 지정하여 대화 실행 (DB에 실시간 상태 기록)
config = {"configurable": {"thread_id": "1"}}
input_message = HumanMessage(content="hi! I'm Lance")
output = graph.invoke({"messages": [input_message]}, config)
```

5. 

# 참고 링크

[랭그래프 공식문서](https://docs.langchain.com/oss/python/langgraph)