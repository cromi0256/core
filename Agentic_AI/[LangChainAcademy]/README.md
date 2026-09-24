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

1. 

# 추가자료
[로드맵](https://roadmap.sh/ai-agents)

[랭체인 교육자료 github](https://github.com/langchain-ai/lca-langchainV1-essentials/tree/main/python) 
