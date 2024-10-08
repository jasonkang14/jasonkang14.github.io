---
title: "Semantic, Vector, and Hybrid"
date: "2024-03-31T20:35:37.121Z"
template: "post"
draft: false
slug: "/llm/comparing-rag-search-methods"
category: "LLM"
tags:
  - "LLM"
  - "AI"

description: "RAG에서 데이터를 가져오는 다양한 방법 비교"
---

회사에서 [Azure AI Search](https://learn.microsoft.com/en-us/azure/search/search-what-is-azure-search)를 활용해서 RAG를 구성하고 있다. [Azure OpenAI](https://learn.microsoft.com/en-us/azure/ai-services/openai/overview)를 사용하고 있기도 하고, [파이썬 예제 코드](https://github.com/Azure/azure-search-vector-samples/tree/main/demo-python)를 나름 친절하게 제공하고 있어서 연동하기도 어렵지 않았다. 

RAG를 구성하기 위해서 

1. 자료로 사용될 데이터를 chunking하고, 
2. vector화 한 후에 
3. Azure AI Search에 적재하게 된다. 

그리고 적재된 데이터를

1. keyword search
2. semantic search 
3. hybrid search

를 통해서 가져오게 되는데, 여기서 사용되는 3가지 검색방법에 대해 이야기 해보려고 한다. 

## Keyword Search

Keyword Search는 단어 자체를 활용해서 검색하는 것이다. 적재된 데이터에 내가 사용한 단어가 포함되어있는지를 확인하고, 해당 값을 가져온다. SQL에서 `LIKE` 문과 유사한 느낌이다. 따라서 비교적 쉽게 구현이 가능하다는 장점이 있지만, 특정 단어를 포함해야만 결과를 가져올 수 있기 때문에 검색 능력이 떨어진다. 예를 들면 산소농도가 낮다는 것은 호흡을 하기 어렵다는 뜻인데, 단순히 Keyword Search만 활용해서 데이터를 가져오려고 한다면, 호흡이라는 단어를 사용해서는 산소 농도가 낮은 경우 어떻게 해야할지 설명하는 문서를 가져올 수 없다. 

Keyword Search는 데이터 내에서 정확한 단어 일치를 기반으로 하기 때문에, 사용자의 질의 의도나 문맥을 깊이 있게 이해하는 데는 한계가 있다. 이러한 방식은 특히 언어의 다양성과 복잡성을 다루어야 하는 경우, 즉 사용자가 비슷한 의미를 지닌 다양한 표현을 사용할 수 있는 상황에서 그 한계가 분명해진다. 만약 사용자가 한국어로 질의하는데, 영어로 된 문서를 검색한다면 Keyword Search는 유용한 자료가 있다고 하더라도 불러올 수 없다 

## Semantic Search

Keyword Search가 단어를 활용해서 검색한다면 Semantic Search는 의미의 유사도를 활용해서 검색한다. 의미의 유사도를 비교하기 위해 NLP에서는 [embedding](https://platform.openai.com/docs/guides/embeddings)을 활용한다. embedding은 간단하게 설명하자면 미리 학습된 데이터를 기반으로 자연어를 숫자로 변환하는 것이다. `여자` 라는 단어와 `남자`라는 단어가 얼마나 유사한지, 아니면 `여자`라는 단어와 `여왕`이라는 단어가 얼마나 유사한지를 숫자를 통해서 비교할 수 있도록 도와주는 기술이다. 일반적으로 vector의 형태로 저장하게되고, RAG로 활용하기 위해 저장된 데이터의 vector와 사용자의 질의의 vector를 cosine similarity를 활용해서 비교한다. 

Azure AI Search의 예제코드에서 사용되는 vector dimension은 1536인데, 두가지 vector의 위치를 cosine similarity를 활용해서 비교하면 숫자가 1에 가까울 수록 두 값은 유사하다는 뜻이니, 이를 활용해 사용자의 질의에 가장 적합한 자료라고 생각되는 문서를 가져온다고 보면 된다. Azure AI Search는 semantic search를 통해 값을 가져올 때 `score` 점수를 같이 보여준다. score가 cosine similiarity를 활용해서 해당 문서와 사용자 질의의 유사도를 나타내는 숫자인데, silver bullet처럼 `특정 값을 넘을 때 유사하다`라고 판단할 수 있는 기준은 없고, 다양한 실험과 테스트를 통해서 threshold를 정해야한다. 예를 들면 keyword search는 언급한 산소농도가 낮을 때 조치방법을 호흡이라는 단어를 활용해서 검색하면 가져올 수 없지만, semantic search를 활용하면 해당 정보를 가져올 수 있다.

## Hybrid Search

Hybrid Search는 Keyword Search와 Semantic Search를 조합해서 사용하는 방법이다. 흔히 Hybrid Search가 가장 효과가 좋다고 알려져 있다. 이러한 접근 방식은 검색의 정확도와 효율성을 동시에 향상시키기 위해 설계되었다. 예를 들어, 사용자가 "산소농도가 낮을 때 조치방법"에 대해 질의할 경우, 키워드 검색만을 사용하면 "호흡"이라는 중요 단어가 누락되어 관련 문서를 찾지 못할 수 있다. 하지만 하이브리드 검색을 사용하면, 키워드 검색으로는 "산소농도", "조치방법"과 같은 명확한 용어를 포착하고, 의미론적 검색을 통해 "호흡"과 같이 연관된 의미를 가진 단어들을 이해하여 더 넓은 범위의 관련 문서를 찾을 수 있다.

Hybrid Search가 효과가 가장 좋다고 설명한 논문들도 있고 hybrid search를 활용하는 것이 약간 국룰인것 처럼 형성되어있다. 하지만 개인적으로는 semantic search만 쓰는 것이 더 결과가 좋았다. 5월에 프로젝트 마무리 예정이라 다양한 RAG 검색 방법을 시도하는 중인데, 프로젝트 마무리 할 때 의견이 달라진다면 이 글은 수정하도록 하겠다. 