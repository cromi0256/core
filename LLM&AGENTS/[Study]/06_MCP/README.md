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

...
