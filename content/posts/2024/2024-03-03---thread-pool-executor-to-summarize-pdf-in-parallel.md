---
title: "Python ThreadPoolExecutor를 활용한 PDF 요약"
date: "2024-03-03T20:35:37.121Z"
template: "post"
draft: false
slug: "/llm/thread-pool-executor-to-summarize-pdf-in-parallel"
category: "LLM"
tags:
  - "LLM"
  - "AI"

description: "pdf 요약 시간을 단축하기 위한 병렬처리"
---

사내 프롬프톤에서 1위를 한 덕분(?)인지 각종 LLM관련 프로젝트들이 나에게 할당되고 있다. 1월에는 분명 하나였는데 갑자기 4개로 늘어났... 이건 어쩔 수 없는 일이니 그냥 넘어가고, 프로젝트 시작 전, 어떻게 문제를 해결할 것인가에 대한 검증을 하기로 했다. 지금 하는 프로젝트는 석유화학 산업에서 특정 제품의 시장 트렌드와 제품의 가격 변동을 예측하는 프로젝트이다. 회사는 다양한 곳으로부터 유료 자료들을 받고있는데, pdf로 이루어진 이 자료들을 활용해서 짧게는 1주일, 1개월, 분기, 반기, 나아가 연단위의 트렌드를 예측하는 것이 목표이다. 

예시로 받아온 pdf자료가 13개여서, 생각보다 많은 분량이다보니 [LangChain의 MapReduce](https://python.langchain.com/docs/use_cases/summarization#option-2.-map-reduce)를 활용해서 요약을 시도해봤다. 예제코드에서 주어진 것은 거대한 pdf를 요약하는 것이기 때문에, 일단 13개의 pdf를 하나로 합치고 (Thanks to ChatGPT), 하나로 합쳐진 pdf를 `MapReduceDocumentsChain`을 활용해서 요약해보기로 했다. 

```python
import os
from PyPDF2 import PdfReader, PdfWriter

def merge_pdfs(directory_path, output_filename):
    # Create a PDF writer object
    pdf_writer = PdfWriter()

    # List all files in the directory and sort them alphabetically
    files = sorted([f for f in os.listdir(directory_path) if f.endswith('.pdf')])

    # Loop through all PDF files
    for filename in files:
        filepath = os.path.join(directory_path, filename)
        
        # Open the PDF file
        with open(filepath, 'rb') as fileobj:
            pdf_reader = PdfReader(fileobj)
            
            # Add all its pages to the writer
            for page_num in range(len(pdf_reader.pages)):
                page = pdf_reader.pages[page_num]
                pdf_writer.add_page(page)
    
    # Save the merged PDF to a file
    with open(os.path.join(directory_path, output_filename), 'wb') as out:
        pdf_writer.write(out)
```

pdf를 하나로 합치고 요약을 시도하니, 한시간이 지나도 요약이 끝나지 않았다(아쉽게도 스크린샷은 없다). chunk를 너무 쪼개서 오래걸렸을 수 있어서 chunk size와 overlap size를 다양하게 바꿔가면서 시도해봤는데, 계속해서 한시간을 초과했다. 서비스를 출시하면 실무진들도 사용하겟지만 보고자료에도 활용되어야해서 시간이 꽤나 중요한 작업이다. 따라서 `MapReduce`는 포기하고 다른 작업을 찾아보기로 했다. 

LangChain에서는 요약을 목적으로 [Refine](https://python.langchain.com/docs/use_cases/summarization#option-3.-refine)도 제공한다. Refine은 병렬 처리는 아니고 chunk된 문서들을 순서대로 읽어들이는 방법이기 때문에 MapReduce보다 느릴 수밖에 없어서 선택할 수 있는 옵션이 아니었다. [MapRerank](https://api.python.langchain.com/en/latest/chains/langchain.chains.combine_documents.map_rerank.MapRerankDocumentsChain.html)는 병렬 처리를 하긴 하지만, 점수를 부여하는 과정이 추가되기 때문에 `MapReduce`보다 더 오래 걸릴 것 같았다. 

그러던 중에 <u>**혹시 각각의 pdf파일이 너무 커서 문제가 되는건 아닌가?**</u> 하는 생각이 들었다. 그래서 pdf문서 하나만 일단 요약해보기로 하고, [StuffDocumentsChain](https://python.langchain.com/docs/use_cases/summarization#option-1.-stuff)을 활용하니 <u>**1분정도 소요되는 것을 확인**</u>했다. 문서 하나를 요약하는데 1분이 걸리는데, 13개를 합치면 왜 한시간이 넘게 걸리는지 알 수는 없지만, 일단은 pdf를 하나로 merge한 후에 요약을 하는 것은 답이 아니라는 것을 깨달았으니 빠르게 방법을 변경할 필요가 있었다. 

그래서 LangChain이 제공하는 기능들을 더 찾아보지 않고, 프로그래밍을 통해 기능을 구현하기로 했다. `MapReduce`나 `Refine`을 사용하면 좋겠지만, 요약이 잘 되더라도 누락되는 부분이 많고, 토큰수를 초과하지 않는다면 `Stuff`를 활용하는 것이 요약의 퀄리티는 제일 좋기 때문이다. 그래서 MultiThread를 활용해서 여러개의 문서를 병렬적으로 요약하고, 텍스트로 요약된 결과들을 활용해서 최종 결론을 내는 것이다. 그림으로 보자면 아래와 같은 느낌이다. 

![system-flow](https://i.imgur.com/642gSIG.png)

StuffChain을 활용하는 코드는 공식문서에 공개되어있으니 생략하고, 다만 여기서 어떤 `DocumentLoader`를 활용할지가 중요했다. 공식문서에는 `PyPDFLoader`를 활용한 예제를 보여주지만, 실제 깃헙 소스코드를 찾아보면 LangChain에서 제공하는 pdf loader는 13개가 있다 

![langchain-pdf-loaders](https://i.imgur.com/zuDUAMT.png)

한국어로 된 문서의 경우 다를 수도 있지만, 내가 참고해야하는 문서들은 모두 영어여서인지, `PyPDFLoader`로도 충분했다. 하지만 metadata에서 더 많은 정보를 가져와서 RAG를 생성해야 한다면 `PDFPlumberLoader`를 추천한다. 

TextSplitter도 매우 중요하다. PDF를 잘 쪼개야(?) 원하는 결과를 얻을 수 있기 때문이다. 공식문서에서는 [CharacterTextSplitter](https://python.langchain.com/docs/modules/data_connection/document_transformers/character_text_splitter)를 활용한다. `chunk_size`와 `chunk_overlap`을 지정하면 무지성으로 정해진 토큰수대로만 문서를 잘라낸다. 이런식으로 chunk를 나누게 되면, 다음 페이지에서 내용이 이어지는 경우 chunk가 같이 묶이지 않아 RAG를 활용할 때 어려움이 있기 때문에 문서 요약에도 효율이 떨어진다. 이를 방지하기위해 [RecursiveCharacterTextSplitter](https://python.langchain.com/docs/modules/data_connection/document_transformers/recursive_text_splitter)를 활용했다. `CharacterTextSplitter`와 다르게 문단 -> 문장 -> 단어 순서를 최대한 유지하면서 문서를 분할하기 때문에 요약에도 유리하지만 RAG활용에도 훨씬 유리하다. 

그리고 이제 이러한 기능들을 병렬적으로 활용할 수 있도록 ChatGPT의 도움을 받아 코드를 작성했다. 

```python
from concurrent.futures import ThreadPoolExecutor, as_completed

def create_analysis_parallel(product_name_list, pdf_directory_path):
    analysis = {}
    futures = []

    with ThreadPoolExecutor() as executor:
        # Submit each product analysis as a separate task
        for product_name in product_name_list:
            """
            pdf 파일들의 경로를 제공하면 해당 경로에 있는 모든 pdf를 병렬로 분석한다
            혹시나 각종 정책 변경으로 인해 참고할 문서가 늘어나는 것들에 대해 더 쉽게 대비할 수 있다. 
            """
            future = executor.submit(analyze_product, product_name, pdf_directory_path)
            futures.append(future)

        # Wait for all tasks to complete and collect results
        for future in as_completed(futures):
            result = future.result()
            analysis[result[0]] = result[1]

    return analysis
```

이렇게 작성하면 기존에 `MapReduceChain`을 활용했을 때 한시간이 넘게 소요되었던 작업이 1분여만에 끝나게 된다. 그리고 ChatGPT는 영어문서를 영어로 요약해서 제공하기 때문에, 보고자료를 만들기 위해서는 번역이 필요했다. ChatGPT의 prompt를 수정해서 작업하기 보다는, ChatGPT는 영문으로 답변을 받아 <u>**사용되는 Output 토큰 수를 줄이고**</u> 번역은 [DeepL API](https://www.deepl.com/pro-api?cta=header-pro-api)를 활용했다. `DeepL`이 월에 500,000글자까지 번역이 무료라서, 왠만하면 추가 과금 없이 무료로 사용이 가능하다. 

이 프로젝트는 아니지만 RAG를 구성하면서 다양한 DocumentLoader, TextSplitter, Vector Store의 조합을 시험해봤는데, 다음 포스트에서 한번 공유해보도록 하겠다. 