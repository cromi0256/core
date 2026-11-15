# 임베딩
임베딩은 이미지, 오디오 같은 비정형 데이터를 일정 길이의 벡터로 변환하는 작업이다.

의미적으로 유사한 데이터끼리는 벡터공간에서 가까이 위치해 있다.

이를 통해 모델은 데이터를 더욱 효과적으로 학습할 수 있다.

임베딩 활용 :
1. 시맨틱 검색 : 단순한 키워드 일치가 아닌, 자연어의 의미와 문맥, 검색 의도를 파악하여 가장 관련성 높은 결과를 제공하는 검색 기술
2. 데이터 분류 : 복잡하거나 고차원의 데이터일지라도 데이터 포인트 간의 유사성을 포착
3. 추천 시스템 : 제품이나 콘텐츠와 같은 항목과 사용자 선호도를 임베딩으로 변환하여 사용자에게 유사한 제품이나 콘텐츠를 추천
4. 이상탐지(+클러스터링) : 유사한 데이터 포인트는 서로 가까이 위치하는 반면, 이상치는 일반적인 분포에서 크게 벗어남
5. RAG

[임베딩 모델](https://huggingface.co/models?pipeline_tag=feature-extraction)

# 벡터 DB
벡터 데이터베이스는 AI 모델이 생성한 고차원 벡터(임베딩)를 저장, 관리 및 검색하도록 설계되었다.

벡터 데이터베이스는 근사 최인접 이웃(ANN) 알고리즘과 같은 색인 기법을 사용하여 대규모 데이터셋을 신속하게 검색하고 관련 결과를 반환한다.

데이터 수집 > 텍스트 분할(Chunking) > 임베딩 생성 > 벡터 DB 저장

벡터 DB :
- pgvector, Pinecone, Chroma, Qdrant, Milvus
- 이외에도 더 있지만 이미 존재하는 인프라 중심으로 설계한다
- 임베딩 생성 > 임베딩 인덱싱 > 유사성 검색

코드 :
```python
import numpy as np
from sklearn.metrics.pairwise import cosine_similarity

# 1. 문서 데이터 정의 (가상의 임베딩 벡터 데이터 - 3차원 가정)
# 실제 구현 시에는 OpenAI나 HuggingFace 모델을 통해 추출한 벡터 값이 들어갑니다.
document_database = {
    "문서1 (인공지능 가이드)": [0.15, 0.88, 0.43],
    "문서2 (가을 패션 트렌드)": [0.72, 0.11, 0.65],
    "문서3 (파이썬 코딩 입문)": [0.21, 0.34, 0.91]
}

# 2. 사용자 검색어의 쿼리 벡터 변환 (예시 데이터)
query_vector = np.array([[0.18, 0.30, 0.85]]) 

# 3. 유사도 계산 및 비교
results = []
for doc_name, doc_vector in document_database.items():
    # 2차원 배열 형태로 변환하여 코사인 유사도 측정
    similarity = cosine_similarity(query_vector, [doc_vector])[0][0]
    results.append((doc_name, similarity))

# 4. 유사도가 높은 순서대로 정렬하여 반환
results.sort(key=lambda x: x[1], reverse=True)

print("🔍 [검색 결과] 가장 유사한 순서:")
for doc, score in results:
    print(f"- {doc}: 유사도 {score:.4f}")

```
