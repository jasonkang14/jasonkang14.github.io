---
title: "LECL 활용한 LLM Chain 분기"
date: "2024-02-18T20:35:37.121Z"
template: "post"
draft: false
slug: "/llm/routing-llm-chain-with-lecl"
category: "LLM"
tags:
  - "LLM"
  - "Langchain"
  - "AI"

description: "사용자의 의도를 파악한 LLM Chain 커스텀"
---

[지난 포스트](/llm/building-an-ai-chatbot-with-streamlit)에서 언급한 프롬프톤에서 1등을 했다. 벌써 한달이 지났는데, 주말을 맞이해서 Future Plan으로 놔두었던 작업을 해보기로 했다. 기존 시스템은 아래와 같은 구조이다 

![safety-chatbot-original-service-flow](https://i.imgur.com/zv2mahR.png)

서비스 흐름은 아래와 같다 

1. 사용자의 질문을 받아서 Vector Database 에서 유사도 검색을 하고 
2. System Prompt + 유사도 검색 결과 + 채팅 기록 + 사용자 질문을 활용해 프롬프트를 생성하고
3. 생성한 프롬프트를 OpenAI API를 활용해서 던지면
4. LLM의 답변을 받아서 사용자에게 전달한다. 

GPT-4-turbo를 사용해서 토큰을 많이 사용해가면서, 현 시점에서 가장 좋은 모델을 사용하다보니 결과는 나쁘지 않았다. 하지만 서비스 특성상 사용자의 질문에 최대한 빠르게 정확한 답변을 내줘야 했는데, 오래 걸릴 때는 질문 하나당 30초~40초가 넘어가는 문제가 있었다. 답변을 생성하는 방식을 스트리밍으로 개선해서 사용자가 "답변을 빨리 받는 것처럼" 느끼게 할 수도 있었지만, 조금 더 근본적인 문제를 해결하고 싶었다. 

설 연휴간 운전을 엄청 많이 했는데, 도로에 있는 시간이 많다보니 이런저런 컨퍼런스의 발표를 들으면서 집에 돌아왔다. 그중 작년 AI Conference에서 Pinecone VP of Engineering Ram Sriharsha의 발표를 들었는데, RAG를 활용해서 context를 너무 많이 넣어주면 오히려 hallucination이 증가한다는 충격적인 정보를 접했다 

<iframe width="560" height="315" src="https://www.youtube.com/embed/u2okMsJC8Cg?si=vd4yq5dH-CdsEG2F" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

현재 포지션에 오기 전에 인공지능 기반 음성인식기를 만드는 회사에 있었는데, GPT와 마찬가지로 Transformer를 사용해서 만들었다. Transformer의 동작 원리를 생각해보면 context를 최대한 자세히 주어야 더 정확한 답변이 생성될 것이라고 생각했었기 때문에 상당히 충격적이었다. 유사도 검색의 결과가 길어서 context의 양이 증가하면, 토큰 수가 늘어나기 때문에 속도가 더 오래 걸릴 수는 있을거라고 생각했는데, hallucination의 가능성이 높아질거라고 생각하지는 않았다. 

서비스가 산업안전과 관련이 있다 보니, 정보의 정확성이 무엇보다 중요했고. 그래서 context를 줄이기 위해 고민하기 시작했다. 슈퍼앱처럼 슈퍼 챗봇을 만들고 싶어서 산업안전보건법, 사내 안전작업 절차, 산업안전 질의사항 등등 다양한 정보를 RAG로 활용해서 context에 넣어주고 있었다. 한국어 특성 상 영어보다 토큰도 더 많이 소비하니 상당히 많은 정보를 LLM에게 넘기고 있는 것이었다. 

그러던 중 사내 LLM스터디에서 들었던 `routing`에 관한 내용이 떠올랐다. [LECL How to 공식문서](https://python.langchain.com/docs/expression_language/how_to/routing)에 보면, 사용자의 질문의 의도를 파악해서 어떤 Chain을 가동시킬 것인지 결정하는 예제가 있다. routing을 적용해서 사용자의 질문이 들어오면 질문의 의도를 파악하고,  

1. 산업안전 보건법을 찾아볼지
2. 사내 안전작업 절차 관련 문서를 찾아볼지 
3. 산업안전 질의사항을 찾아볼지 

결정해서 사용자의 의도에 맞는 Vector DB를 검색한 후에, 해당 RAG에 적합한 system prompt를 활용해서 LLM을 호출하기로 했다. 따라서 시스템의 흐름도 아래와 같이 변경되었다.

![updated-system-flow](https://i.imgur.com/ToPRAvh.png)

- 일단 RAG로 활용되는 context가 1/3으로 줄어들기 때문에 사용되는 input 토큰 수가 확연히 줄어서 비용을 저감할 수 있었고, 
- 토큰 수가 줄어드니 자연스럽게 LLM의 응답속도도 빨라졌다. 
- 사용자 질문의 의도를 파악하는데 LLM을 한번더 호출하니, 호출의 수가 늘어나서 시간이 더 오래걸릴거라고 생각할 수 있다.
    - 의도를 파악하는 작업은 어렵지 않으니 GPT 3.5를 사용했고
    - 확연히 줄어든 input context로 GPT 4를 호출하니 **전체적인 질의 시간은 확연히 감소한 것을 확인**할 수 있었다. 

속도가 확연히 줄었는데, routing을 활용했기 때문이라고만 보긴 어렵고, 프롬프톤이 끝나고 OpenAI에서 [새로운 GPT 4 turbo 모델](https://platform.openai.com/docs/models/gpt-4-and-gpt-4-turbo)을 출시했기 때문일수도 있다. 

속도와 결과는 상당히 좋아졌지만, Vector Search 또한 개선하고 싶었다. 기존 코드에서는 사용자의 질문을 바로 vector search의 query로 활용하고 있었다. 

```python
def get_context(index_name, query):
  # RAG를 위해 Azure AI Search를 활용하고 있다.
  context = perform_azure_search(index_name=index_name, query=query) 
```

답변의 퀄리티에는 크게 영항을 끼치지 않을 수도 있지만, 사용자 질문에서 키워드를 추출한 후 Vector Search에 활용하면 조금 더 좋은 검색이 가능할 것이라고 생각해서, 키워드 추출 후 LLM을 호출하는 방식으로 변경했다. 

```python
from functions import get_model

from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate


def extract_keyword(user_input, model_name="gpt-3.5"):
    prompt = ChatPromptTemplate.from_template("""
Given a statement about safety operations at a power plant, 
Extract one keyword from the statement delimited by triple backticks
```{statement}```
""")
    current_llm = get_model(model_name)
    output_parser = StrOutputParser()
    chain = prompt | current_llm | output_parser
    keyword = chain.invoke({"statement": user_input})

    return keyword
```

로그를 확인하니 검색 결과는 크게 차이나지 않았는데, **검색 결과의 score가 확연히 개선**되었다. 업데이트를 통한 최종 서비스 흐름은 아래와 같다. 

![final-service-flow](https://i.imgur.com/oXKZkhM.png)

전반적으로 답변의 퀄리티도 좋아지고, 속도가 빨라져서 만족하고 있다. 사용자 피드백을 받아서 추가로 개선사항을 도출하게 되면 다시 글을 써보도록 하겠다.