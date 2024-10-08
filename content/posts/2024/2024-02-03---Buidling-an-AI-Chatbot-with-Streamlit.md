---
title: "Streamlit + AzureOpenAI를 활용한 챗봇 개발"
date: "2024-02-03T20:35:37.121Z"
template: "post"
draft: false
slug: "/llm/building-an-ai-chatbot-with-streamlit"
category: "LLM"
tags:
  - "LLM"
  - "Azure"
  - "AI"


description: "파이썬으로도 화면을 만들 수 있다"
---

새해를 맞이해서(?) 회사에서 프롬프톤을 열었다. GenAI를 활용해서 문제를 해결할만한 무언가를 만들어보는 주제였는데, 커리어를 정유공장의 프로세스 엔지니어로 시작했기 때문에 무언가 제조공장에서 할만한 것을 해보고 싶었다. 작년에 같이 프로젝트를 했던 실력있는 PM과 같은팀을 하는데 성공했고, 우연히 제조산업과 관련된 분들과 같이 팀을 이룰 수 있었다. (나는 팀장이 아니다). 아이디어를 내던 중에 안전에 관련된 질문이 많을텐데 이것을 해결해보면 어떠냐로 뭉쳤고, 사용자 인터뷰를 해보니 니즈가 있어서 이것을 개발해보기로 했다. **채팅을 통한 대화** 를 하면 좋겠다는 의견에 따라 챗봇을 만들어보기로 했고, 전에 봤던 [Streamlit](https://streamlit.io/)을 활용해서 UI를 개발해보기로 했다. 

파이썬 코드로 UI를 만들 수 있어서 더 빠르게 작업할 수 있을것 같았고, 파이썬으로 작업하니 [Langchain](https://python.langchain.com/)을 사용하기에도 수월할 것 같다는 판단에 선택했는데, 옳은 판단이었다. 개발하면서 체득한 스트림릿 사용법을 코드와 함께 소개해보겠다. 

**Disclaimer**
- 개발기간이 1주일이라 공식문서를 전혀 읽어보지 않고 [llm-examples](https://github.com/streamlit/llm-examples) repository를 fork해서 사용했습니다.
- Streamlit이 추구하는 방향에 맞지 않을수도 있습니다.

1. 스트림릿은 엔트리 파일(`chatbot.py`)을 매번 처음부터 끝까지 작동시키면서 UI를 업데이트 한다. 

처음 실행할 때 `chatbot.py`을 읽는건 이해가 되는데, 채팅을 하면서 사용자 답변, AI 답변이 추가될때도 전체적인 코드를 첫줄부터 다시 실행한다. UI를 그릴 때 업데이트 된 부분만 확인해서 추가하는게 아니라 일단 처음부터 끝까지 다 훑는 느낌이다. 코드를 사용해서 같이 보자 

```python
import streamlit as st

print('beginning')
st.set_page_config(page_title="안Gen봇", page_icon='⛑️')
st.title("👷 안Gen봇")
st.caption("🚀 A safety chatbot powered by 52g")
```

위에서 작성한 것 처럼 파일의 최상단에 `print('beginning')`을 작성하고 챗봇을 사용해보니 beginning이 계속 찍힌다 

![log-indicates-beginning](https://i.imgur.com/gfAkhZj.png)

그래서 UI를 업데이트 할때도 `st.success("성공했습니다")` 와 같은 method를 실행한다고 할 때, 별도의 함수에서 선언하는 것보다 채팅의 흐름에 맞게 파일 내에서 해당 로직의 위치를 조절하는 것이 매우 중요하다. 예를들면 이번 서비스의 경우에는 아래 스크린샷처럼, 추천질문을 버튼 형태로 제공하고, 피드백 버튼을 보여줘야했는데 그렇게 작업하고싶다면 `chatbot.py` 내에서 위치를 잘 조절해야한다 

![showing-different-buttons](https://i.imgur.com/rYQD4Vv.png)

2. `session_state`를 잘 활용해야한다. 

`session_state`는 간단히 말하자면 전역변수이다. 파이썬의 `global` 또는 프론트엔드 개발자라면 리덕스나 리코일 등을 활용해서 관리하는 변수라고 보면 되겠다. 무언가 업데이트가 일어날 때마다 `chatbot.py`를 처음부터 끝까지 다 훑어보기 때문에 `session_state`를 활용한 조건문 처리와, 해당 조건문의 위치가 매우 중요하다. 

위에서 스크린샷으로 첨부한 것처럼 추천질문을 보여주고 피드백 버튼을 보여주려면 `chatbot.py`내에서 추천질문을 처리하는 코드가 피드백 버튼의 위에 위치해야한다. 별도의 함수로 만들어서 호출하는 순서를 활용해서 처리하려고 했는데 실패했다. 

```python
# 파일의 상단에서 session_state에 내가 원하는 변수를 설정한다.
if "show_feedback" not in st.session_state:
    st.session_state["show_feedback"] = False
    
if "additional_questions" not in st.session_state:
    st.session_state["additional_questions"] = None

...

# session_state에 있는 값을 확인해서 UI를 수정한다
if st.session_state["additional_questions"] is not None:
    for question in st.session_state["additional_questions"]:
        if st.button(question.replace("추가질문:", "")):
            st.session_state["show_feedback"] = False
            st.session_state["additional_questions"] = None
            process_question(question, system_prompt, chat_history, model_name=model_name)
            

if st.session_state["show_feedback"]:
    col1, col2, col3 = st.columns(3)
    with col1:
        st.write('방금 답변은 도움이 되셨나요?')    

    with col2:
        if st.button('👍'):
            st.write('피드백 감사합니다!')
            update_feedback(session_id=session_id, vote_count=1)
            
    with col3:
        if st.button('👎'):
            st.write('피드백 감사합니다')
            update_feedback(session_id=session_id, vote_count=-1)
```

추가질문을 선택하면, LLM에게 질의를 하고, 피드백 버튼은 사라져야한다. 추가질문 버튼을 클릭하면, `show_feedback`을 `False`로 변환하기 때문에, 피드백 버튼이 사라지게 되는것이다. 

3. `st.rerun()`을 적당히 활용하면 매우 유용하다

[공식문서](https://docs.streamlit.io/library/api-reference/control-flow/st.rerun)에 보면, `rerun()`을 호출하는 경우, 스크립트 진행을 멈추고 다시 처음부터 시작하게된다. 위의 예제에서는 추가질문을 선택했을 때 피드백 버튼이 사라져야해서 `show_feedback`을 확인하는 조건문이 다시 돌아가야 했기 때문에 그대로 진행시켰지만, 만약 피드백 버튼이 보여도 상관 없는 구조라면 `process_question()`에서 LLM에게 질의한 이후에 `st.rerun()`을 호출해서 불필요하게 조건문을 다시 타는 것을 막을 수 있다. 

4. pyaudio를 비롯한 음성관련 패키지가 배포환경에서 작동하지 않는다

Streamlit discuss에 상당히 많은 내용이 올라와있는 것으로 봐서는 해결하고 싶은 의지는 없어보인다. 현장에서 폰으로 타이핑 하기가 어려우니 음성인식 기능을 활용해보려고 했는데, 로컬에서는 잘 작동하던 기능들이 배포하니 에러가났다. 물론 빠르게 확인하기 위해 [Streamlit Cloud](https://streamlit.io/cloud)를 사용해서 그런것이고, 스트림릿에서 소개하는 직접 배포 기능을 사용하면 문제없이 잘 작동하는 것 같다. 

스트림릿을 활용하면서 불편함이 있긴했지만 그래도 빠르게 프로토타입을 만드는데는 매우 유용하다고 생각한다. 기회가 된다면 공식문서를 전체적으로 읽어보고 maintainer의 의도대로 코드를 작성해보고 싶다. 어쩌면 간단한 강의를 만들 수 있을지도 모르겠다는 생각이 든다. LLM 활용 솔루션을 만들면서 고생하는 분들이 많을텐데 한번 만나서 서로의 고충을 듣고 노하우를 공유하는 시간이 있었으면 좋겠다. 