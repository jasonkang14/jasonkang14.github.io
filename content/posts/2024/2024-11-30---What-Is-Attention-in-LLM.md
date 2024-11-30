---
title: "What Is Attention?"
date: "2024-11-30T20:35:37.121Z"
template: "post"
draft: false
slug: "/llm/what-is-attention"
category: "LLM"
tags:
  - "LLM"

description: "An exploration of the concept of Attention in LLMs, discussing its significance and impact on model performance and understanding."
---

In Korea, there is a tradition of "study" groups where engineers from various companies or backgrounds gather to explore specific engineering concepts. Participants typically select a book or an online curriculum to follow, and each week, one member is responsible for studying a chapter in depth and presenting the ideas to the group. I recently joined a study group focused on understanding how Large Language Models (LLMs) function. Last week, I was tasked with presenting the concepts of Attention and Transformers within LLMs. While I am familiar with these concepts, explaining them to others proved to be a distinct challenge compared to understanding them on my own. Here is my perspective on explaining what Attention is.

# Attention이란 무엇인가?

Attention이라는 개념은 [Attention Is All You Need](https://arxiv.org/abs/1706.03762)에서 처음 소개되었다. 언어 모델에서 "Attention"라는 용어는 데이터를 처리할 때 입력 시퀀스의 특정 부분에 집중할 수 있게 하는 메커니즘을 의미한다. 이는 특히 단어가 여러 의미를 가질 때 중요하다. 모델은 적절한 해석을 결정하기 위해 주변 문맥을 고려해야 한다. Self-Attention 모델은 문장에서 각 단어의 중요도를 다르게 부여하여, 각 단어의 다른 단어와의 관련성을 평가할 수 있게 한다. 이러한 접근 방식은 모델이 문맥적 관계를 포착하는 능력을 향상시키고, 언어에 대한 전반적인 이해를 개선한다.

## Attention은 어떻게 작동하는가

언어 모델의 Attention 메커니즘은 시퀀스 내 다른 단어와의 관련성에 따라 각 단어에 다양한 중요도, 즉 `Attention Score`를 부여함으로써 작동한다. 자세한 작동 방식은 다음과 같다:

### 1. **Input Representation**
   - 입력 시퀀스의 각 단어는 임베딩을 사용하여 숫자 벡터로 변환된다.
   - Attention의 경우, 모델 훈련 중 학습된 임베딩을 사용한다. 이 학습된 임베딩은 정적 임베딩에 비해 여러 가지 장점이 있다:
      - 학습 데이터에 따라 적응한다.
      - 토큰이 나타나는 문맥에 따라 달라진다.
      - 모델의 목표에 특정한 토큰 간의 미묘한 관계를 포착한다.

### 2. **Self-Attention 메커니즘**
![Self-Attention Architecture](https://i.imgur.com/mKTYPDc.png)
   - Attention의 핵심은 Self-Attention 메커니즘으로, 시퀀스 내 각 단어를 다른 모든 단어와 비교한다.
      - [Self-Attention: A Better Building Block for Sentiment Analysis Neural Network Classifiers](https://arxiv.org/abs/1812.07860v1)
   - 각 단어에 대해 세 가지 벡터가 계산된다:
     - **쿼리(Q):** 단어가 "묻고" 있거나 찾고 있는 것을 나타낸다.
     - **키(K):** 각 단어가 제공하는 특징이나 문맥을 나타낸다.
     - **값(V):** 단어의 실제 내용을 나타낸다.

![Scaled Dot Product Attention](https://i.imgur.com/FDnAsqa.png)

   - 왜 "Self"인가?
     - "Self"라는 용어는 메커니즘이 동일한 시퀀스 내에서 작동함을 나타낸다.
     - 각 토큰은 동일한 시퀀스 내의 모든 다른 토큰과 자신을 비교하여 주의 점수를 계산한다.
        - 예를 들어, "어제 밤에 배가 고파서 피자를 먹었다"라는 문장에서 "배"라는 토큰을 처리할 때, 모델은 "어제", "밤에", "배가", "고파서", "피자를", "먹었다"와의 관계를 `Attention Score`를 통해서 계산한다.
        - 이 과정은 외부 시퀀스나 문맥이 아닌 입력 자체의 토큰만을 포함하므로 "Self-Attention"이라는 용어가 사용된다.

### 3. **Attention Score**
   - 한 단어의 쿼리(Q)는 모든 단어의 키(K)와 비교하여 점수(dot product)를 계산한다. 이 점수는 현재 단어에 대해 각 단어가 얼마나 관련이 있는지를 나타낸다.
      - 예를들면, "어제 밤에 배가 고파서 피자를 먹었다"라는 문장에서 `배`라는 단어가, 과일, 이동수단이 아닌 사람의 신체 부위를 나타낸다는 것을 이해하기 위해 `배`라는 단어를 `피자`라는 단어와 연관지어야 하는데, 이를 Attention Score를 활용해 계산한다.
   - `Attention Score`는 소프트맥스 함수를 통해 정규화되어 확률로 변환되며, 이 확률은 1로 합산된다. 이는 현재 단어에 대해 각 단어의 중요도나 가중치를 결정한다.


## Multi-Head Attention으로 확장

![Multi-Head Attention](https://i.imgur.com/blJ7Vuk.png)

현대의 주의 메커니즘은 **Multi-Head Attention**을 사용하여, 서로 다른 학습된 가중치로 병렬로 과정을 반복하여 시퀀스 내 관계의 다양한 측면을 포착한다. 이러한 헤드의 출력은 연결되어 추가로 처리된다.

### Multi-Head Attention의 장점

Multi-Head Attention은 Self-Attention 메커니즘을 확장하여, 입력 데이터를 동시에 다양한 관점에서 학습하고 표현할 수 있도록 한다. 

- **관계 및 특징 학습**:
  - **다양한 관계 캡처**: 각 Attention Head는 독립적으로 작동하며, 입력 시퀀스를 참조하는 고유한 방식을 학습한다. 예를 들어, 한 헤드는 구문적 관계(예: 주어-동사 관계)에 집중할 수 있고, 다른 헤드는 의미적 관계(예: 동의어 또는 문맥적 의미)에 집중할 수 있다.
  - **문맥 이해력 향상**: 개별 헤드가 시퀀스의 다른 부분에 주목함으로써, Self-Attention 메커니즘이 놓칠 수 있는 미묘한 관계를 더 잘 캡처할 수 있다.
  - **다양한 특징 학습**: Multi-Head Attention은 여러 종류의 정보를 추출할 수 있다(예: 한 Head는 지역적 종속성을, 다른 Head는 전체적 종속성을 캡처).

- **효율성 및 성능 향상**:
  - **차원 축소 및 표현력 강화**: 각 헤드는 임베딩 차원을 나누어 계산을 수행한다(\(d_h = d / h\), 여기서 \(h\)는 헤드의 수). 이를 통해 계산 효율성을 높이면서도 풍부한 관계를 캡처할 수 있다.
  - **일반화 성능 향상**: 여러 헤드 간의 다양한 어텐션 메커니즘은 모델이 보지 못한 데이터에 대해 더 나은 일반화 성능을 발휘하도록 돕는다.


## Self-Attention과 Multi-Head Attention의 차이

#### 1. **Self-Attention**
- 셀프 어텐션은 하나의 어텐션 메커니즘으로 입력 시퀀스 내 모든 토큰 간의 관계를 계산한다.
- 각 토큰은 단일 **Query(Q)**, **Key(K)**, **Value(V)** 벡터를 생성하고, 이를 사용해 어텐션 스코어와 최종 문맥 표현을 계산한다.
- **한계**: 단일 어텐션 메커니즘은 너무 좁은 관점에 집중하거나, 다양한 관계를 효과적으로 캡처하지 못할 수 있다.

#### 2. **Multi-Head Attention**
- 멀티-헤드 어텐션은 셀프 어텐션의 확장판으로, 여러 독립적인 어텐션 메커니즘(헤드)이 병렬로 작동한다.
- 각 헤드는 고유한 학습 가중치 행렬(\(W_Q, W_K, W_V\))을 사용하여 \(Q\), \(K\), \(V\) 벡터를 계산하고, 임베딩 차원을 나눠 사용한다.
- 모든 헤드의 출력을 연결(concatenate)한 후 선형 레이어를 통과시켜 최종 표현을 생성한다.


### **주요 차이점**

| 특징                   | Self-Attention                           | Multi-Head Attention             |
|-----------------------|------------------------------------|------------------------------------|
| **헤드 수**           | 단일 어텐션 메커니즘.                 | 여러 독립적인 어텐션 메커니즘.         |
| **집중 영역**          | 토큰 관계에 대한 단일 관점.            | 다양한 관점에서 관계를 포착.            |
| **차원성**            | 하나의 어텐션 메커니즘에 전체 임베딩 차원을 사용. | 각 헤드가 임베딩 차원을 나눠 병렬적으로 어텐션 수행. |
| **표현력**             | 다양한 관계를 모델링하는 데 한계가 있다. | 복잡하고 다양한 관계를 더 잘 캡처.       |
| **출력**              | 각 토큰당 하나의 어텐션 출력.           | 여러 헤드의 출력을 결합해 풍부한 표현 생성. |


## Attention이 주목받게된 이유

Attention은 NLP에서 혁신적인 개념으로, 시퀀스를 처리할 때 **Recurrent Neural Network(RNN)**과 **Convolutional Neural Network(CNN)**의 주요 한계를 해결한 것이다. 주의가 주목받는 이유는 다음과 같다:

- **효율적인 관계 포착 및 병렬화**
  - **장거리 의존성 포착**: Attention 메커니즘은 시퀀스 내 모든 토큰 간의 직접적인 관계를 계산하여, 모델이 장거리 의존성을 효율적으로 포착하고 전체 시퀀스 문맥을 이해할 수 있게 한다.
  - **병렬화**: Transformer와 같은 주의 기반 모델은 모든 토큰을 동시에 처리하여 학습과 추론을 빠르고 확장 가능하게 만든다.

- **동적 문맥화 및 해석 가능성**
  - **동적 문맥화**: 주의는 현재 문맥에 따라 토큰의 중요도에 가중치를 동적으로 부여하여, 모델이 입력의 가장 중요한 부분에 집중할 수 있게 한다.
  - **해석 가능성**: 주의 점수는 모델이 집중하고 있는 토큰을 명확하게 나타내어 모델의 동작을 해석하고 디버그하기 쉽게 만든다.

- **확장성 및 다양한 적용 가능성**
  - **트랜스포머로의 확장성**: 주의는 트랜스포머 아키텍처의 기초가 되어, RNN의 필요성을 제거하고 BERT, GPT 등 현대 NLP 시스템의 중추가 된다.
  - **다양한 작업에서의 성능 향상**: 주의 메커니즘은 기계 번역, 텍스트 요약, 질문 응답, 언어 모델링 등 다양한 NLP 작업에서 성능을 크게 향상시킨다.
  - **보편적 적용 가능성**: 주의는 NLP에 국한되지 않고, 비전, 음성, 멀티모달 모델 등 다양한 분야에 성공적으로 적용된다.

요약하자면, Attention Mechanism은 효율적인 관계 포착과 병렬화를 가능하게 하고, 동적이고 문맥에 민감한 토큰 표현을 제공하며, 다양한 작업과 분야에 걸쳐 확장성과 적용 가능성을 보여준다. 이러한 이점은 주의를 게임 체인저로 만들어 RNN 기반 모델에서 트랜스포머 기반 아키텍처로의 전환을 주도하고, 현대 NLP 시대의 무대를 마련한다.

Attention은 다양한 장점이 있다. 이제 다음 포스트에서 Attention이 어마어마한 힘을 발휘한 Transformer에 대해 작성해보도록 하겠다.