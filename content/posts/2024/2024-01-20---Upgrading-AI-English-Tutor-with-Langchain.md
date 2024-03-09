---
title: "Langchain을 활용한 인공지능 영어과외 업그레이드"
date: "2024-01-20T20:35:37.121Z"
template: "post"
draft: false
slug: "/llm/upgrading-ai-english-tutor-with-langchain"
category: "LLM"
tags:
  - "LLM"

description: "이제 직접 대화 기록을 메모리에서 관리하지 않아도 된다"
---

AI English Tutor의 MVP는 순수하게 OpenAI의 API만 활용해서 개발했다. 다음주에 회사에서 해커톤의 LLM버전인 프롬프톤이라는 행사를 하는데, 이로인해 개발팀이 모여서 `Langchain을 활용한 어플리케이션 개발` 이라는 주제로 스터디를 했다. 스터디를 하면서 보니 Langchain에서 제공하는 [ConversaionChain](https://python.langchain.com/docs/modules/memory/conversational_customization)을 사용하면, 내가 직접 대화기록을 관리하면서 리스트로 넘겨주지 않아도 된다는 것을 깨달았다. 그래서 코드 관리 효율을 개선하고, 귀찮음(?)을 줄이기 위해 Langchain을 활용해서 한 번 업데이트 해보기로 했다. 

기존 코드는 파이썬 리스트를 활용해서 대화 기록을 관리한다
```python
import json

history = [] # 대화 내용을 저장하는 메모리

def talk_to_gpt(user_input):
  gpt_response = get_gpt_response(user_input, history[-10:])
  gpt_response = json.loads(gpt_response)
  gpt_response = gpt_response['response']
  history.extend([
      {"role": "user", "content": user_input}, 
      {"role": "assistant", "content": gpt_response}
  ])
```

gpt에서 json 형태로 응답을 주면, 해당 json을 parsing해서 리스트에 넣는 방식이다. 이제 Langchain을 활용하면 어떻게 이 코드가 간단해지는지 확인해보자. 

```python
# Now we can override it and set it to "AI Assistant"
from langchain.prompts.prompt import PromptTemplate

template = """The following is a friendly conversation between a human and an AI. The AI is talkative and provides lots of specific details from its context. If the AI does not know the answer to a question, it truthfully says it does not know.

Current conversation:
{history}
Human: {input}
AI Assistant:"""
PROMPT = PromptTemplate(input_variables=["history", "input"], template=template)
conversation = ConversationChain(
    prompt=PROMPT,
    llm=llm,
    verbose=True,
    memory=ConversationBufferMemory(ai_prefix="AI Assistant"),
)

conversation.predict(input="Hi there!")
```

공식문서에서 제시하는 예제코드를 보면, 사용자가 전달하는 내용은 프롬프트에 `input`으로 들어가고, 기존의 대화 내용은 프롬프트에 `history`로 들어가는 것을 볼 수 있다. 파이썬 스트링에서 f-string 문법에서 f만 제외하고 중괄호를 넣으면, 랭체인은 해당 값이 `input_variables`에 포함되었는지 확인하고, 함수 파라미터로 받아와서 넘겨주는 방식이다. 공식문서에서 제공하는 프롬프트는, 대화 내용을 기억하고 대화를 잘나누라고 되어있는데. 나는 목적이 영어과외이니 기본 system prompt를 수정해보았다. 

```python
prompt_template = """
The following is a friendly conversation between a human and an AI.
The AI a top-tier English tutor with years of experience.
The AI is talking to a student who wants to practice speaking English. 
The AI is to help the student practice speaking English by having a conversation. 

The AI should feel free to correct the student's grammar and pronunciation and/or suggest different words or phrases to use whenever the AI feels needed.
And when the AI corrects the student, the AI must start the sentence with "it is better to put it this way"
But even when you correct the student, try to make a conversation first, and then correct the student

Current conversation:
{history}
Human: {input}
AI Tutor:"""
```

트랜스포머의 작동 원리를 생각하면 프롬프트를 최대한 자세히 적어주어야 context를 잘 파악해서 가장 좋은 결과를 낼 것 같은데, 이런저런 실험들을 해보니 오히려 간결하게 instruction을 전달하는 것이 LLM이 답변을 생성하는데 더 유리한 것 같다. 일단 위와 같이 프롬프트를 작성하고, `ConversationChain`을 활용해서 답변을 제작하도록 기존 코드를 수정했다. 

```python
# Import necessary classes from the langchain and langchain_openai libraries
from langchain.chains import ConversationChain
from langchain.memory import ConversationBufferMemory
from langchain.prompts.prompt import PromptTemplate
from langchain_openai import OpenAI

# Initialize the OpenAI model with a specified temperature.
# Temperature set to 0 for deterministic, consistent responses.
llm = OpenAI(
    temperature=0
)

# Create a conversation buffer memory to keep track of the conversation.
# This includes prefixes to distinguish between the AI tutor and the human user.
memory = ConversationBufferMemory(
    ai_prefix="AI Tutor:",
    human_prefix="Human:",
)

# Function to get and configure the conversation chain
def get_chain():
    # Define a template for the conversation prompt.
    # This template sets the context for the conversation and instructions for the AI.
    ...
    # Create a PromptTemplate object with the defined prompt template.
    # This template includes variables for the conversation history and the latest human input.
    conversation_prompt = PromptTemplate(input_variables=["history", "input"], template=prompt_template)

    # Initialize the conversation chain.
    # This chain uses the defined prompt, the language model (llm), and the conversation memory.
    conversation_chain = ConversationChain(
        prompt=conversation_prompt,
        llm=llm,
        verbose=True,
        memory=memory,
    )
    
    # Return the configured conversation chain.
    return conversation_chain
```

이제 `talk_to_gpt()`라는 함수에서 위에서 리턴하는 `conversation_chain`을 활용해 ChatGPT와 대화를 나눈다 

```python
def get_gpt_response(transcript):
    # Talk to the AI Tutor via langchain 
    conversation = get_chain()
    answer = conversation.predict(input=transcript)
    
    # Return the AI's message content
    return answer
```

이제 리스트를 활용해서 대화 히스토리를 관리할 필요 없이 `langchain`에서 대화이력을 관리해준다. `verbose=True`로 세팅해두면 langchain의 프롬프트를 확인할 수 있다. 스크린샷으로 첨부해본다. 

![Langchain Prompt](https://i.imgur.com/2qqRJsZ.png)

그리고 `langchain`은 다양한 [memory](https://python.langchain.com/docs/modules/memory/types/)들을 제공한다. 위에서 사용한 `ConversationBufferMemory`는 가장 간단한 메모리인데, 기존에 있던 모든 대화를 `{history}`에 넣어서 프롬프트로 활용하는 식이다. 그냥 로컬 메모리에 저장하는 방식이기 때문에, 이 대화를 만약 15분 길게는 30분씩 이어간다고 한다면, ChatGPT의 토큰수가 초과되는 순간 대화를 더이상 이어갈 수 없게된다. 

이를 해결하기 위해 다른 메모리들이 존재한다. 첫번째로 [ConversationBufferWindowMemory](https://python.langchain.com/docs/modules/memory/types/buffer_window)는 변수명에서 나타나는 것처럼 `window`를 활용해서, k를 파라미터로하는 가장 최근의 몇개 대화만 `{history}`에 넣어서 프롬프트로 활용한다. 한시간 정도 대화를 한다면 처음 10분정도의 대화는 뒤에 10분의 대화의 맥락에 큰 영향을 미치지 않는다고 판단할 때 사용하면 좋다. 

다음은 [ConversationEntityMemory](https://python.langchain.com/docs/modules/memory/types/entity_summary_memory). 변수명에서 유추할 수 있듯 기존의 대화 기록에서 특정 주제에 대해 정보를 기억하는 것이다. `load_memory_variables()`이라는 method를 활용해서 대화를 나누는 중 대화 이력에서 해당 정보를 가져와야 할 때 memory에서 LLM을 활용하여 해당 정보를 가져오는 방식이다. `ConversationEntityMemory`는 대화에서 주요 내용을 추출하기 때문에 OpenAI API를 두번 호출하게 된다. 따라서 좀 느리다

다음은 [ConversationSummary](https://python.langchain.com/docs/modules/memory/types/summary). 변수명에서 보이는 것처럼 대화를 요약해서 `{history}`에 넣어주는 방식이다. 요약할 때 OpenAI API를 추가로 호출해야 하기 때문에 이 역시 느리다. 하지만 대화 내용이 너무 길어서 토큰을 초과할 것 같다면 `ConversaionSummary`를 사용하는 것이 속도는 좀 느리겠지만 정확한 대화를 이어나가는데 훨씬 유리하다. 

다음은 [ConversationSummaryBuffer](https://python.langchain.com/docs/modules/memory/types/summary_buffer). `ConversationSummary`와 `ConversationBufferWindowMemory`를 합쳐둔거라고 보면 된다. 토큰 수를 사용해서 가장 최근의 몇개 대화 내용을 요약해서 `{history}`에 넣어주는 방식이다. `ConversationSummary`가 전체를 요약하는 것과 비교하면 속도는 훨씬 빠르다

다음은 [ConversationTokenBuffer](https://python.langchain.com/docs/modules/memory/types/token_buffer). `ConversationSummaryBuffer`와 유사하지만 요악을 하지 않고 토큰수를 기반으로 전체 내용을 다 넘긴다. 여기서는 왠지 `tiktoken`으로 연산이 한번 들어갈 것 같으니, 각각의 대화가 너무 길어지는게 아니라면 `ConversationBufferWindowMemory`가 더 유리할 것 같다. 

지금은 로컬 메모리에 저장하기 때문에 주피터 노트북을 닫으면 기존의 대화가 날아간다. Langchain에서 DynamoDB나 Firebase 등을 메모리로 쉽게 활용할 수 있는 기능을 지원하기 때문에, 다음 포스트에서는 무료인 Firebase를 활용해서 대화 내용을 저장하고, 노트북을 닫더라도 마지막 대화 시점부터 이야기를 이어나갈 수 있도록 작업해보겠다. 