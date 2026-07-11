# RAG
RAG는 두 가지 구성 요소로 이루어져 있는데, 하나는 데이터베이스에서 관련 정보를 검색하는 검색기(Retrieval)이고, 다른 하나는 검색된 데이터를 기반으로 응답을 생성하는 생성기(Generater)다.

![Image](https://docs.aws.amazon.com/images/sagemaker/latest/dg/images/jumpstart/jumpstart-fm-rag.jpg)

RAG 과정 :
1. Retrieval : 사용자의 쿼리를 임베딩 하여 벡터 DB에서 유사한 자료 검색
2. Generation : 검색한 문서를 프롬프트에 삽입하여, LLM모델에 전달하여 답변 생성

RAG를 통해 정확하고 문맥에 맞는 응답을 생성하므로 질문 답변, 문서 생성, 의미 검색과 같은 작업에 이상적이다.

llama index나 langchain 프레임워크를 사용하면 간단히 RAG를 구현할 수 있지만, 필수는 아니다

# 코드 예제
1. Indexing
```python
from sentence_transformers import SentenceTransformer
import faiss
import numpy as np

model = SentenceTransformer('all-MiniLM-L6-v2')

docs = ["문서1", "문서2", "문서3"]

# embedding
embeddings = model.encode(docs)

# FAISS index
index = faiss.IndexFlatL2(embeddings.shape[1])
index.add(np.array(embeddings))
```

2. Retrieval
```python
query = "질문"
q_vec = model.encode([query])

D, I = index.search(q_vec, k=3)

retrieved_docs = [docs[i] for i in I[0]]
```
3. Generation
```python
context = "\n".join(retrieved_docs)

prompt = f"""
다음 정보를 기반으로 답변하라:

{context}

질문: {query}
"""

response = llm(prompt)
```
