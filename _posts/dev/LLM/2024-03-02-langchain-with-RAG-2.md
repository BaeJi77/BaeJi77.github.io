---
title:  "아주 간단한 langchain - langchain with RAG (2)"
excerpt: ""

categories:
  - LLM
  - Langchin
tags:
  - dev
  - LLM
  - Langchain
---

해당 아티클에 존재하는 코드를 보고 싶으시다면 [해당 주소](https://github.com/BaeJi77/langchain-with-RAG)에서 보실 수 있습니다.

# 이전 글

- [1편: langchain과 RAG에 대한 소개](https://baeji77.github.io/llm/langchin/langchain-with-RAG-1/)

# Langchain과 RAG를 활용한 고급 자연어 처리 기법

이번 글에서는 Langchain과 Retrieval-Augmented Generation(RAG)을 활용하여 자연어 처리를 고급화하는 방법에 대해 알아보겠습니다. 이전 글에서는 이 두 기술의 기본적인 소개를 했으니, 아직 읽지 않으셨다면 [1편: langchain과 RAG에 대한 소개](https://baeji77.github.io/llm/langchin/langchain-with-RAG-1/)를 먼저 확인해 주세요.

# Embedding

이전 글에서 Embedding에 대해서 아주 간단하게 핵심만 말하고 넘어갔었습니다. 이번에는 조금 더 다양한 이야기와 실제 코드와 함께 설명을 해보겠습니다.

## Embedding이란 무엇인가

`Embedding`은 자연어 텍스트를 기계가 이해할 수 있는 수치적 표현, 즉 `벡터로 변환하는 과정`입니다. 이러한 벡터는 텍스트의 의미를 고차원 공간에서 저차원으로 표현하여, 컴퓨터가 텍스트의 의미를 비교하고 분석할 수 있게 합니다.

이전에는 float로 된 array(vector)라고 설명했는데요. Embedding도 여러 모델을 가지고 있습니다. 그 모델마다 학습된 데이터뿐만 아니라 파인튜닝 및 Embedding의 결과인 vector의 길이도 다를 수 있습니다. 

그런 이유로 반드시 Embedding을 하기 위해서는 똑같은 Embedding한 결과끼리 비교를 해야됩니다. 당연한 이야기이지만 중요한 사항이라고 생각하기 때문에 한번 이야기드립니다.

## 실제로 Embedding한 결과

```python
from langchain_openai import OpenAIEmbeddings

embedding_model = OpenAIEmbeddings(
    openai_api_key="OPENAI_API_KEY"
)

text = "hello world!"
em = embedding_model.embed_documents([text])

print(len(em[0])) # vector size
print(em[0]) # 실제 vector에 저장된 값 보기
---
# 출력
1536
[-0.007764335207892291, -0.005596709868602093, ..., ]
```

이 코드는 `hello world!` 문장을 embedding한 결과를 출력합니다. 여기서 보게되면 OPENAI에서 만든 Embedding 모델은 1536길이를 가진 vector를 만드는 것을 알 수 있습니다.

## Embedding 결과 비교하기

RAG에서 임베딩을 하는 이유는 유저가 입력하는 데이터에 대해서 비슷한 데이터를 찾기 위해서 입니다. 그러기 위해서는 현재 들어온 데이터와 현재 저장된 데이터 중에 비슷한 데이터를 찾아야됩니다. 이때 벡터 끼리 비교를 하게 됩니다.

비교라고 했지만 실제로는 여러 알고리즘이 존재합니다. 벡터로 된 값들에 대해서 유클리드 거리를 계산하기도 하고 코사인을 이용해서 유사도를 계산합니다.

예제에서는 코사인을 이용해서 유사도를 검사하는 코드를 만들어보겠습니다.

```python
def cos_sim(A, B):
    return dot(A, B) / (norm(A) * norm(B))

conversation = [
    "안녕하세요!",
    "넵! 무엇을 도와드릴까요?",
    "저의 직업은 개발자입니다.",
    "저는 개발을 잘하고 싶어요!",
]

embeddings = embedding_model.embed_documents(
    conversation
)

q = "대화를 나누고 있는 사람의 직업은 무엇인가요?"
a = "개발자입니다."
embedded_query_q = embedding_model.embed_query(q)
embedded_query_a = embedding_model.embed_query(a)

print(q + " / " + a)
print(cos_sim(embedded_query_q, embedded_query_a))

print(q + " / " + conversation[0])
print(cos_sim(embedded_query_q, embeddings[0]))
print(q + " / " + conversation[1])
print(cos_sim(embedded_query_q, embeddings[1]))
print(q + " / " + conversation[2])
print(cos_sim(embedded_query_q, embeddings[2]))
print(q + " / " + conversation[3])
print(cos_sim(embedded_query_q, embeddings[3]))
---
# 출력
대화를 나누고 있는 사람의 직업은 무엇인가요? / 개발자입니다.
0.8226396184358739

대화를 나누고 있는 사람의 직업은 무엇인가요? / 안녕하세요!
0.8057939904422297
대화를 나누고 있는 사람의 직업은 무엇인가요? / 넵! 무엇을 도와드릴까요?
0.8147186405030087
대화를 나누고 있는 사람의 직업은 무엇인가요? / 저의 직업은 개발자입니다.
0.8379169010457393
대화를 나누고 있는 사람의 직업은 무엇인가요? / 저는 개발을 잘하고 싶어요!
0.7868311429134318
```

제가 만든 임의의 대화에서 어떤 실제로 가장 유사한 결과가 무엇인지에 대해서 임베딩한 결과끼리 코사인 계산을 통해서 구했습니다. log에서 보이는것과 같이 `저의 직업은 개발자입니다.` 라는 내용이 가장 높은 결과가 나왔습니다.

## 전체 문서를 자르는 이유

저는 구체적으로 설명하지는 않았지만 임베딩을 할때 기본적으로 tokenize라는 단계를 진행합니다. 한마디로 전체문서를 모두 임베딩에서 데이터를 vector로 만드는 것이 아니라 데이터를 잘게 잘라서 임베딩을 하는 것입니다. `왜 그럴까요?`

임베딩은 아까 말씀드린 것처럼 데이터를 float로 구성된 array로 만듭니다. openai에서 제공하는 api를 사용하는 경우 1536개의 float number를 가진 array가 됩니다. 이 과정은 순수 데이터를 결국 손실을 해서 변환하는(압축하는) 과정이라고 할 수 있습니다. 

몇기가짜리 데이터도 1536 크기 배열로 바뀌고, 1byte짜리 데이터도 1536 크기 배열로 바뀝니다. 그러기 때문에 너무 큰 데이터를 한번에 임베딩하게 된다면 너무 많은 데이터들이 압축되기 때문에 검색에서 우리가 원하는 데이터를 찾기 어려울 수 있습니다. 

두번째 이유는 gpt-4와 같은 경우 매우 큰 데이터도 같이 첨부해서 프롬프트를 구성해서 LLM으로 요청할 수 있지만 gpt-3와 3.5는 구성할 수 있는 프롬프트 길이에 대한 제한이 있습니다. (명확하게는 LLM이 인식하는 토큰이라는 것에 대한 수 제한이 있습니다.) 그러다보니 너무 큰 문서를 프롬프트에 넣어서 프롬프트를 만들어서 요청할 수 없습니다. 

그리고 추가적으로 큰 문서를 임베딩하는데는 계산 효율성도 떨어지는 문제들이 있습니다.

그러기 때문에 큰 문서를 잘라서 벡터화시키고 그것을 저장하게 됩니다.

## 자르는 문서의 크기에 따라서

위에서 설명했듯이 임베딩에서 적절한 크기로 쪼개는 것은 중요합니다. 

- 너무 크게 쪼개는 경우

문맥을 잃고 모델이 중요한 정보를 놓칠 수 있으며, 토큰 제한을 초과하고 계산 효율성이 떨어질 수 있습니다. 

- 너무 작게 쪼개는 경우 

의미의 연속성이 손상되고, vectordb에서 검색이 비효율적이 될 수 있습니다.

# Vectordb

## 사용 이유

Vectordb는 벡터 데이터를 효율적으로 저장하고 검색할 수 있는 데이터베이스입니다. 

매번 데이터를 vector화하는 것은 계산 자원과 시간을 낭비할 수 있으므로, 한 번 vector화한 데이터는 vectordb에 저장하여 재사용하는 것이 효율적입니다. 

또한, 실시간으로 db를 업데이트하면 추가적인 데이터에 대해서도 빠르게 처리할 수 있습니다.

## 현재 Langchain에서 지원하는 Vectordb 예시

Langchain은 현재 Redis, MongoDB, Elasticsearch와 같은 NoSQL 데이터베이스에 벡터 데이터를 저장하도록 지원합니다. 

당연히 vectordb를 사용했을 때에 장점이 있겠지만 vectordb를 따로 운영하기 어려운 경우 간단하게 현재 사용하고 있는 NoSQL를 기반으로 벡터 데이터를 관리할 수 있습니다.

# Langchain with RAG code example

RAG 기술을 활용하기 위해서 기본적으로 (1) documents를 db을 적재하는 단계와 (2) 유저 요청을 처리하는 단계인 두 단계로 분리해서 설명할 수 있습니다.

제가 작성한 글에서는 대부분 메소드에 구체적인 내용보다는 전체적인 흐름에 대해서 이야기를 드릴려고 합니다. 구체적인 내용은 추가적으로 학습하기를 권장합니다.

## documents를 DB에 적재하는 단계

제가 보여주는 코드는 간단하게 하기 위해서 db에 적재하지는 않았으면 local에 index 파일을 만들어서 처리했습니다. 

```python
def get_new_vector_db_and_store_local_index(
        base_filename=pdf_filename,
        index_name=local_index_filename,
):
    # load the document and split it into chunks
    loader = PyPDFLoader(base_filename)
    pages = loader.load_and_split()

    # split it into chunks
    text_splitter = RecursiveCharacterTextSplitter(
        chunk_size=1000,
        chunk_overlap=50,
        length_function=tiktoken_len # tictoken_len은 text를 나눌때 어떤 기준으로 할것인지에 대한 결과값을 제공합니다. 저는 openai에서 사용하는 기준으로 했습니다.
    )
    docs = text_splitter.split_documents(pages) # docs는 나눠진 문서들에 배열입니다.

    new_vectordb = FAISS.from_documents(docs, embeddings)
    new_vectordb.save_local(index_name)

    return new_vectordb

vectordb = get_new_vector_db_and_store_local_index(base_filename=pdf_filename, index_name=local_index_filename)
```

이 코드는 제가 embedding하기를 원하는 문서를 쪼개고, 쪼갠 문서를 embedding하여 vectordb에 저장하는 코드입니다.

```python
# search arguments를 넣을 수 있는데 k는 실제 문서가 반환되는 수이며, fetch_k는 MMR 알고리즘에 전달할 문서의 양을 의마합니다. 
# 밑에와 같이 설정하면 3개의 문서가 LLM에 요청을 보낼때 사용될 수 있습니다.
retriever = vectordb.as_retriever(search_type="mmr", search_kwargs={'k': 3, 'fetch_k': 10}) 

question = "클라우드 네이티브 앱 모범 사례에 대해서 말해줘."
docs_from_db = retriever.invoke(question)
print(docs_from_db[0].metadata)
print(docs_from_db[1].metadata)
print(docs_from_db[2].metadata)
---
{'source': './data/VMware_K8sForDevelopers_eBook.pdf', 'page': 13}
{'source': './data/VMware_K8sForDevelopers_eBook.pdf', 'page': 0}
{'source': './data/VMware_K8sForDevelopers_eBook.pdf', 'page': 9}
```

실제로 해당 vectordb에 특정 요청을 보내게 되면 자동으로 embedding을하고 그것에 대해서 가장 유사성이 높은 문서를 반환합니다. 저 같은 경우 일부러 내용이 길기 때문에 metadata만 로그로 남겼습니다.

```python
docs_and_scores = vectordb.similarity_search_with_score(question)
print(docs_and_scores)
---
[(Document(page_content='클라우드  네이티브  앱 모범 사례\n구형 애플리케이션의  마이그레이션...', metadata={'source': './data/VMware_K8sForDevelopers_eBook.pdf', 'page': 13}), 0.19348186), (Document(page_content='개발자를  위한 쿠버네티스\n 10적절한  가시...', metadata={'source': './data/VMware_K8sForDevelopers_eBook.pdf', 'page': 9}), 0.30422744), (Document(page_content='모르겠습니다 . 기존  애플리케이션을  이식하거...', metadata={'source': './data/VMware_K8sForDevelopers_eBook.pdf', 'page': 2}), 0.3196209), (Document(page_content='수도 있고, 원하는  대로 변경할  수도 있습니다 .\nGoogle 이 최근  게시한  블로그  게시물 에 오퍼레이터  만들기  모범 사례가  자세히  나와 \n있습니다 .\n커뮤니티를  통해 소통,...', metadata={'source': './data/VMware_K8sForDevelopers_eBook.pdf', 'page': 13}), 0.3218215)]
```

위에 있는 코드는 실제로 스코어에 대한 결과까지 같이 보여주도록 요청을 해봤습니다. 

## 유저 요청 처리 단계

```python
# 유저가 입력한 프롬프트에 대해서 임베딩
llm = ChatOpenAI(openai_api_key="OPENAI_API_KEY")

# context에는 임베딩한 결과가 들어감. input에는 유저가 입력했던 input이 들어감.
prompt = ChatPromptTemplate.from_template("""
Answer the following question based only on the provided context:

<context>
{context}
</context>

Question: {input}
""")

document_chain = create_stuff_documents_chain(llm, prompt)

retrieval_chain = create_retrieval_chain(retriever, document_chain)

response = retrieval_chain.invoke({"input": question})
# print(response["input"])
# print(response["context"])
print(response["answer"])
---
# 실제 출력 결과
클라우드 네이티브 앱 모범 사례에는 구형 애플리케이션의 마이그레이션 작업을 첫 번째 쿠버네티스 프로젝트로 삼지 말고 새로운 클라우드 네이티브 애플리케이션을 처음부터 개발하는 것을 명확히 알아야 한다는 내용이 포함되어 있습니다. 또한 알맞은 툴을 선택하고 패턴을 사용하며 오퍼레이터를 최대한 활용하는 등의 지침이 소개되어 있습니다. 이러한 모범 사례를 따르면 최소의 노력으로 최대의 효과를 거두는데 도움이 될 것입니다.
```

위와 같이 prompt를 작성할때 임베딩한 결과를 같이 넣을 수 있도록 만들고 그것을 input같이 자동으로 넣어서 LLM 결과가 나오도록 할 수 있는 메소드들을 이용해서 바로 결과물을 만들 수 있습니다.

주석 처리한 로그를 찍어보게되면 실제 사용자가 입력한 input과 vectordb에서 반환된 데이터를 볼수가 있습니다.

## 전체 코드

전체 코드는 [해당 주소](https://github.com/BaeJi77/langchain-with-RAG)에 남겨두겠습니다. 잘 모르는 영역이기도하며 급하게 만들어서 코드가 좋다고 할 수 없지만 좋은 참고자료가 되기를 바라겠습니다.

# 마무리

이번 글에서는 실제 코드를 보면서 langchain에 RAG을 같이 사용하는 것에 대해서 이야기해봤습니다. 해당 기술에도 충분히 단점이 존재하지만 많은 시간과 비용을 투자하지 않고도 LLM의 안정성을 높힘으로서 LLM의 단점을 극복할 수 있다고 말할 수 있을것 같습니다.

제가보여준 예시는 단순히 pdf를 읽은 것이지만 엑셀, docs 뿐만 아니라 md이나 웹에 있는 데이터를 학습해서 임베딩할 수 있기에 더 높은 안정성을 높일 수 있다고 생각합니다.

# ref

- https://python.langchain.com/docs/modules/data_connection/document_loaders/pdf
- https://python.langchain.com/docs/integrations/vectorstores/faiss
- [모두의ai 유튜브 및 코랩](https://www.youtube.com/watch?v=WWRCLzXxUgs&list=PLQIgLu3Wf-q_Ne8vv-ZXuJ4mztHJaQb_v)


<!-- # 개요

# embedding
- embedding은 무엇인가.
- 실제로 embedding한 결과는 무엇인가? 예제 코드 필요.
- 실제 embedding한 결과를 어떻게 비교하는가. (거리 계산에 대한 이야기.) 코드 필요.
- embedding 옵션으로 거리가 비슷한 여러개 문서를 같이 뽑아낼 수 있도록 하는 코드 예시.
- 전체 문서를 자르는 이유.
- 자르는 문서의 size에 대한 이야기. 너무 크거나 작으면 발생할 수 있는 이슈.

# vectordb
- 사용 이유. 
  - 매번 데이터를 vector화 하는건 낭비. 
  - 실시간으로 db만 업데이트하면 추가적인 데이터에 대해서 처리할 수 있음.
- vectordb로 사용되는 예시들. ex) faiss, ...
- 현재 langchain에서는 redis, mongo, es와 같은 nosql에 vector을 저장하도록 지원해줌. 예시 코드 필요.

# Langchain with RAG code example

## 전처리 단계
- 문서 쪼개기
- 쪼갠 문서 임베딩 (vector화)
- vector 저장하기 (local file로)

## 실제 유저 응답 단계
- 유저가 입력한 프롬프트에 대해서 임베딩.
- vectordb로 임베딩한 결과를 요청하여서 가장 비슷한 데이터 가져오기.
- 유저 프롬프트와 vectordb에 검색결과로 나온 것을 합쳐서 하나의 프롬프트로 만들어서 LLM에 요청하기.
- 결과 유저에게 반환하기.
 -->
