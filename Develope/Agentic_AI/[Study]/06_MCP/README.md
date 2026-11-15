[공식문서](https://modelcontextprotocol.io/docs/getting-started/intro)

[참고자료](https://discuss.pytorch.kr/t/deep-research-model-context-protocol-mcp/6594)

![Image](https://mintcdn.com/mcp/bEUxYpZqie0DsluH/images/mcp-simple-diagram.png?w=2500&fit=max&auto=format&n=bEUxYpZqie0DsluH&q=85&s=dc4ab238184b6c70e06e871681c921c5)

모델 컨텍스트 프로토콜(MCP)은 AI 에이전트가 컨텍스트 정보를 관리하고 공유할 수 있는 표준화된 방식을 제공한다.

기존의 방식은 도구마다 api를 불러와 개별의 프롬프트를 작성했다면 MCP는 간단히 tool로써 정의하면 된다.

핵심 구성요소:

MCP Host : AI 모델을 운용하는 주체 애플리케이션
  - Claude Desktop, IDE의 AI 어시스턴트(ex. Cursor) 등
  - 에이전트가 컨텍스트 정보, 도구 및 리소스에 접근하고 활용할 수 있도록 표준화된 인터페이스를 제공하는 브릿지 역할

MCP Client : 하나의 MCP 서버와 1:1 연결을 담당하는 구성요소

[MCP Server](https://mcpservers.org/ko/) : MCP Client에 맥락을 제공하는 프로그램
  - 사용자가 서버를 추가하여 필요한 도구를 확장가능하다

# MCP 구현
로컬 서버 연결 : 인터넷을 거치지 않고 컴퓨터 내부와 통신
```python
# 1. 라이브러리 설치: pip install fastmcp
from mcp.server.fastmcp import FastMCP

# MCP 서버 생성 (이름 지정)
mcp = FastMCP("MyLocalServer")

# AI가 사용할 로컬 도구(Tool) 정의
@mcp.tool()
def calculate_bmi(weight_kg: float, height_cm: float) -> str:
    """사용자의 몸무게와 키를 받아 BMI 지수를 계산합니다."""
    height_m = height_cm / 100
    bmi = weight_kg / (height_m ** 2)
    return f"계산된 BMI는 {bmi:.2f}입니다."

if __name__ == "__main__":
    mcp.run()
```

[원격 서버 연결](https://mcpservers.org/ko/remote-mcp-servers) : 깃헙 같은 외부 서버와 연결하여 실행한다

예시 : Slack(클로드 코드)
```bash
claude mcp add slack --transport http https://mcp.slack.com/mcp
```

클라이언트 개발 : Claude나 Cursor같은 Host에 의존하지 않고 커스터마이징 할 수 있다
[공식 가이드](https://modelcontextprotocol.io/docs/develop/build-client)
