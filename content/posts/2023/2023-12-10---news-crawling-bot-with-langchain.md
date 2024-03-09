---
title: "Langchain + DeepL을 활용한 뉴스 크롤링 봇 개발"
date: "2023-12-10T20:35:37.121Z"
template: "post"
draft: false
slug: "/llm/news-crawling-bot-with-langchain-and-deepl"
category: "LLM"
tags:
  - "LLM"

description: "LLM을 활용한 업무 자동화"
---

# TL;DR
Langchain을 활용하면 토큰 수를 초과했을 때 비교적 쉽게 개발할 수 있다 

--------------

8월에 이직하고 다양한 계열사들의 개발자들이 모이는 사내 컨퍼런스에 참여했다. 그중 한 세션에서, 

1. 매일 아침 회사에 관련된 기사를 검색하고 
2. ChatGPT를 활용해서 긍정적인 기사인지, 부정적인 기사인지를 판단해서
3. 리더십에 공유하는

프로젝트에 관한 설명이 있었다. 

감정분석은 OpenAI 공식문서에 [sentiment analysis](https://platform.openai.com/docs/guides/legacy-fine-tuning/case-study-sentiment-analysis)섹션이 따로 있을 정도로 LLM을 활용해서 흔히 구현하는 기능인데, 회사에 대한 고객들의 인식을 파악하기 위해 이런 시스템을 자동화 했다는 점이 인상적이었다. 

행사가 끝나고 사무실로 복귀하는 중에, 내가 속한 조직에서도 환경에 관한 기사를 정기적으로 리더십과 동료들에게 공유한다는 말을 들었다. 나는 입사한지 2개월정도 밖에 되지않아 수신처에서 누락됐었는데, 들어보니 

1. 담당자가 정기적으로 기사를 검색하고, 
2. 직접 요약해서 노션 테이블에 저장하고
3. 그것들 중에서도 중요한 기사들만 추려서 표로 만들어서
4. 이메일로 공유한다는 

것이었다. 

사실 감정분석만 빼면 컨퍼런스에서 소개된 것과 같은 내용이었다. 팀 리더 뒷자리에 타고있던 나에게 자연스럽게 "Jason이 한번 자동화 해봐라" 라고해서 다음날 담당자를 만났다. 만나서 이야기를 들어보니, 기사를 검색하는 것은 사람이 검토를 해야하기 때문에 크게 어려움이 없는데, 노션에서 csv를 추출하고, 이메일에 첨부하는 것이 더 시간이 오래걸린다는 피드백을 받았다. 

그래서 자동화는 

1. 사용자가 기간을 선택하면 노션에서 해당 날짜에 해당하는 기사들을 가져와서 표로 만들고 
2. [AWS SES](https://aws.amazon.com/ses/)를 활용해서 담당자에게 이메일을 보내주면 
3. 담당자가 자동으로 작성된 이메일 내용을 검토해서 이메일을 전달하는 방식

으로 진행해서 먼저 공유하고, 그 후에 뉴스를 자동으로 요약해서 노션에 저장하는 것을 시도해보기로 했다. 

[AWS Batch](https://docs.aws.amazon.com/batch/latest/userguide/what-is-batch.html)등을 활용해서 정해진 시각에 이메일을 보낼 수도 있겠지만, 담당자가 원하는 시점에 이메일을 확인하는 편이 더 효율적이라고 생각했다. 그렇다면 이메일 전송 이벤트를 발생시켜야 하는데, 버튼 하나 때문에 클라이언트를 개발하기는 너무 공수가 커서, 담당자에게 날짜를 선택할 수 있는 chrome extension을 만들어서 제공했다. 사용성을 검증하기 위해서 디자인은 크게 신경쓰지 않고 일단 필수 `<input>`과 `<button>` 만 만들어서 전달했다. 

![syncus-news-bot-chrome-extension](https://i.imgur.com/f4zkrxZ.png)


노션에서 데이터를 가져오는 것은 [Notion API](https://developers.notion.com/)를 활용했다. 다만 notion api는 클라이언트에서 호출하면 CORS에러가 발생하는 경우가 많아서, 스크린샷의 chrome extension에서 Notion API에 request를 보내는 [AWS Lambda](https://aws.amazon.com/pm/lambda/?gclid=CjwKCAiAvdCrBhBREiwAX6-6Unb3-bTwJELuwJzB0Um3b3d_zN_JsKeXMRs3liJGcD2ucYGGHP7gKBoCYjwQAvD_BwE&trk=b28d8305-f5fb-4858-9ae6-04a78cfcc154&sc_channel=ps&ef_id=CjwKCAiAvdCrBhBREiwAX6-6Unb3-bTwJELuwJzB0Um3b3d_zN_JsKeXMRs3liJGcD2ucYGGHP7gKBoCYjwQAvD_BwE:G:s&s_kwcid=AL!4422!3!651510601848!e!!g!!aws%20lambda!19836398350!150095232634)를 호출하고, lambda에서 SES를 활용해서 이메일을 보내도록 했다. 

간단한 기능인데 다른 업무로 인해 생각보다 오래걸렸고(약 2주), 이제 뉴스 크롤링 봇을 만들기 시작했다. 업무 흐름이 뉴스를 특정 사이트에서만 가져오기 때문에, 처음 생각했던 흐름은 

1. [BeautifulSoup](https://beautiful-soup-4.readthedocs.io/en/latest/)으로 해당 사이트를 크롤링 해서 뉴스 리스트를 가져오고
2. 리스트에 있는 뉴스 링크 url를 활용해서 ChatGPT로 요약해서 
3. LLM이 중요하다고 생각하는 기사를 노션에 저장

하려고 했다.

BeautifulSoup을 활용한 크롤링은 익숙했기 때문에 별 문제 없었고, 요약을 하기위한 prompt를 먼저 작성했다. 

```python
def get_completion(user_prompt, system_prompt, model=model_name):
    messages = [{"role": "system", "content": system_prompt}, {"role": "user", "content": user_prompt}]
    chat_completion = client.chat.completions.create(
        model=model_name,
        response_format={ "type": "json_object" },
        messages=messages,
    )

    return chat_completion.choices[0].message.content
```

`system prompt`를 상수로 두지 않고 함수의 매개변수로 사용한 이유는, LLM을 활용해서

1. 뉴스 리스트에 있는 각 뉴스기사 요약
2. 요약된 뉴스 리스트에서 가장 적합한 기사를 선정

하는 2가지 기능을 담당해야 하기 때문이다. 우선 뉴스 리스트를 요약하기 위한 prompt를 먼저 작성했다. 

```python
    # Define prompt
    system_prompt = f"""
    you are a SENIOR NEWS REPORTER who can analyze a news article about environment. 
    SUMMARIZE a news article from a user input into 3-4 sentences.
    """
    
    user_prompt = f"""
    text delimited by triple backtics is a news article about environment
    ```{article_list}```
    """
```

이번에 OpenAI DevDay에서 공개된 [JSON mode](https://platform.openai.com/docs/guides/text-generation/json-mode)를 사용하지 않으면 아래 스크린샷은 가장 적합한 기사를 선정한 결과인데, 가끔 output앞에 "내가 판단한 결과는 이렇다"와 같은 텍스트가 추가되서 output parsing이 어려울 때가 있으니 `response_format`을 지정해두는 편이 좋다 

![llm-output-with-intro-text](https://i.imgur.com/t87XoVB.png)

추가로 [DeepLearning.ai의 ChatGPT Prompt Engineering for Developers](https://www.deeplearning.ai/short-courses/chatgpt-prompt-engineering-for-developers/)에서 설명하는대로 `user_prompt`에 사용되는 기사 본문을 `으로 감싸줬고 어디 컨퍼런스에 가니 영문 대문자로 프롬프트를 작성하면 더 잘 작동한다고 해서 주요단어인 system role과 summarize만 대문자로 작성했다. 

첫번째 사이트를 잘 처리하고 다음 사이트로 넘어가니, 뉴스 본문이 너무 길어져서 아래와 같은 에러 메세지가 나타났다. 토큰 수를 초과해버린것이다 ㅠㅠ 

![llm-max-token-error](https://i.imgur.com/elP36tY.png)

직접 tokenizer를 사용해서 처리할 수도 있지만 예전에 LangChain의 [MapReduce](https://python.langchain.com/docs/modules/chains/document/map_reduce)를 활용해서 긴 본문을 요약했던 경험이 있어서 LangChain을 활용하기로 했다. LangChain에서 제공하는 [Document와 관련된 기능](https://python.langchain.com/docs/modules/chains/document/)은 Stuff, Refine, MapReduce, Map re-rank가 있는데, MapReduce가 속도는 빠르지만, 한번에 모든걸 때려박고 요약하는 [Stuff](https://python.langchain.com/docs/modules/chains/document/stuff)하고 비교했을 때는 성능이 살짝 떨어진다. 지금 개발하는 서비스 특성 상 요약하는 속도가 중요한 것은 아니기 때문에 Stuff를 사용하기로 결정했다. 

[LangChain Summarization Stuff 공식문서](https://python.langchain.com/docs/use_cases/summarization#option-1.-stuff)에는 예제가 잘 나와있는데, python string이 아니라 [WebBaseLoader](https://python.langchain.com/docs/integrations/document_loaders/web_base)를 활용해서 웹사이트의 내용을 요약해주는 기능을 기본으로 제공하고 있었다. 따라서 위에서 작성했던 프롬프트를 손절하고 공식문서에 나와있는 코드를 따라서 기사의 url만 넘겨주니 요약을 매우매우 잘해준다. 

```python
from langchain.chains.summarize import load_summarize_chain
from langchain.chat_models import ChatOpenAI
from langchain.document_loaders import WebBaseLoader

def summarize_news_article(article_url, model_name):
    loader = WebBaseLoader(article_url)
    docs = loader.load()

    llm = ChatOpenAI(temperature=0, model_name=model_name)
    chain = load_summarize_chain(llm, chain_type="stuff")

    chain.run(docs)
```

이제 얘를 활용해서 기사를 요약하니, 아래와 같은 포맷의 뉴스기사 리스트가 생성되었다. 

```python
news_article_list = [
    {
        'title': 뉴스 기사 제목,
        'link': 뉴스 기사 링크,
        'summary': 뉴스 기사 요약
    }
]
```

여러개의 기사들 중 가장 적합한 기사를 찾아내기 위해서 prompt를 작성했다. 

```python
def get_main_article(article_list):
    system_prompt = f"""
    you are a SENIOR NEWS REPORTER who can analyze a list of news articles and 
    determine which one is the most relevant when reporting to a CEO who would like to 
    gain insights about ESG (environmental, social, and governance) 
    the OUTPUT MUST BE FORMATTED as a JSON dictionary from the input of which 
    summary is the most relevant according to your analysis. like below

    'title' : <NEWS ARTICLE TITLE IN ENGLISH>,
    'link' : <NEWS ARTICLE LINK>,
    'summary' : <NEWS ARTICLE SUMMARY>
    """
    
    user_prompt = f"""
    text delimited by triple backtics is a list of JSON dictionaries 
    which represent news articles about environment
    ```{article_list}```
    """
```

아웃풋을 JSON으로 고정한다면 [Pydantic(JSON) parser](https://python.langchain.com/docs/modules/model_io/output_parsers/pydantic)를 사용하는 것이 효과적인데, prompt에 output format을 지정해주는 것 만으로도 충분히 처리할 수 있어서 굳이 output parser를 사용하지는 않았다. 위 prompt를 사용해서 LangChain에 전달하면, 아래와 같은 결과물을 받을 수 있다. 

![llm-json-output](https://i.imgur.com/7iYZuRx.png)

한국에 있는 팀과 기사를 공유하니 `summary`는 한국어로 변경해서 전달하는 것이 좋다고 판단했다. 하지만 ChatGPT 3.5를 사용해서인지 한국어 번역이 약간 어색하다는 느낌을 받았다. 그래서 번역에 특화된 [DeepL](https://www.deepl.com/pro-api?cta=header-pro-api)을 사용해서 ChatGPT가 판단한 공유하기 가장 적합한 기사의 `summary`를 번역하도록 했다. DeepL도 월에 500,000자까지는 무료로 사용할 수 있어서, 격주로 기사의 영문요약만 번역하면 되기 때문에 개발하고자하는 니즈에 적합한 툴이라고 생각했다. [deepl-python](https://github.com/DeepLcom/deepl-python)을 활용하면 매우 쉽게 구현할 수 있다. 

```python
def translate_into_korean(text, target_language='KO'):
    result = translator.translate_text(text, target_lang=target_language) 
    translated_text = result.text
    return translated_text
```

그럼 이제 한국어로 번역된 뉴스 요약본을 확인할 수 있다. 

![news-summary-translated-into-korean](https://i.imgur.com/6bzk08B.png)

개발을 여러날에 나눠서 해서, 영문 요약된 기사와 한국말로 번역된 기사가 다르다. 앞으로 회사에서 LLM을 사용한 다양한 업무를 할 예정인데, 공개 가능한 선에서 공유해보도록 하겠다. 
