---
title: Database Concurrency Problems
date: "2022-07-24T12:44:37.121Z"
template: "post"
draft: false
slug: "/database/what-are-database-concurrency-problems"
category: "Database"
tags:
  - "Database"

description: "Database concurrency는 어떤 문제들을 야기할 수 있는가"
---

![database-concurrency](https://i.imgur.com/bwsa7kv.png)

최근 회사에서 신규 프로젝트를 시작했다. 데이터베이스 구조에 대해서 이야기하고, 서버 <-> 클라이언트 간 통신을 어떻게 할지 고민중에 내 생각을 전달했더니 CTO님께서 database concurrency 이슈가 있을 것 같다고 하셨다. 단어 뜻만 들어보고 `write가 끝나지 않은 상태에서 read를 해서 그런가?` 라고 생각만 하고 넘어갔다. 생각해보면 그 때 바로 뭔지 질문할 걸 그랬다.

용어 자체에 답이 숨겨져 있을 것 같아서 단어 뜻을 찾아봤다. 구글에 검색하니 `여러가지 일이 동시에 일어나는 것`이라고 하고, computing에 `한정짓자면 여러가지 task를 동시에 실행시키는 것`이라고 한다. 구글의 친절한 검색엔진 덕분에 [MIT자료](https://web.mit.edu/6.005/www/fa14/classes/17-concurrency/#:~:text=Concurrency%20means%20multiple%20computations%20are,cores%20on%20a%20single%20chip)를 찾았는데, concurrency는 여러가지 계산이 동시에 일어나는 것이라고 한다. 

단어 뜻만 보면 내가 생각한게 맞는 것 같다. 이제 구체적으로 database concurrency에 대해 알아본다. 

구체적으로 database concurrency에 대해 찾아보니, Transaction A가 일어나는 와중에 Transaction B가 일어나는 경우이다. 이게 문제가 되는 이유는 Transaction도 상태 있기 때문이다. 

```
1. Active state
2. Partially committed state
3. Committed state
4. Failed state
5. Aborted state
6. Terminated state
```

![database-transaction](https://i.imgur.com/UlGqwvb.png)

단어에서 알 수 있듯, terminated state에서 하나의 transaction이 완료된다. commited state는 변경될 수 없다는 걸 기억한다면 concurrency 문제들을 이해하는데 더 도움이 된다. 

```
1. Temporary Update Problem
2. Incorrect Summary Problem
3. Lost Update Problem
4. Unrepeatable Read Problem
5. Phantom Read Problem
```

하나씩 알아본다. 

#### 1. Temporary Update Problem(a.k.a. Dirty Read Problem)
Update를 시도한 transaction이 commit 되기 전 read가 발생한 경우이다. 
두가지 경우가 있겠는데
  - `Partially Commited State`에서 commit 되기 전에 read가 발생한 경우
    ![temporary-update-problem](https://i.imgur.com/zyCzOQo.png)
  - `Failed Stated`에서 `Aborted State`로 rollback되기 전 read가 발생한 경우
    ![temporary-update-problem](https://i.imgur.com/787imMr.png)

#### 2. Incorrect Summary Problem
요약을 잘못하는 경우. aggregation을 시도하는 중 update가 발생하는 경우이다.
![incorrect-summary-problem](https://i.imgur.com/t7CWM2k.png)
위 표를 보면 X는 A,B를 aggregate하는데, Y가 그 중간에 값을 업데이트 한다. 따라서 X는 정확한 값을 얻을 수 없다. 

#### 3. Lost Update Problem
하나의 transaction이 시도한 update를 다른 transaction이 덮어쓰는 것을 뜻한다.
![lost-update-problem](https://i.imgur.com/aI7ox1U.png)

#### 4. Unrepeatable Read Problem
여러개의 READ operation이 다른 결과를 낳는 것을 뜻한다.
![Unrepeatable-read-problem](https://i.imgur.com/dHRNXAy.png)
이게 왜 문제가 되는지 이해하기 어려워서 조금 더 찾아봤다. 다른 transaction이 update했으니, update된 값을 불러오는게 맞지 않나? 
Unrepeatable Read Problem은 같은 사용자가 두 번 READ할 때 보다는, 다른 사용자가 동시(?)에 같은 값을 read하려고 할 때 발생하는 문제이다. 
따라서 첨부한 표도 아래와 같이 보면 조금 더 이해하기 쉽다. 

![Unrepeatable](https://i.imgur.com/ZHAME7I.png)

#### 5. Phantom Read Problem
한 번 READ를 시도하고 같은 값을 다시 READ를 시도하는 사이에 해당 값이 지워지는 경우이다. 
![phantom-read-problem](https://i.imgur.com/eUW9xkw.png)


글 처음에 CTO님께서 언급한 database concurrency 문제는 Temporary Update Problem인 것 같다. 업데이트가 마무리 되기 전 사용자가 값을 read할 수 있는 문제가 발생할 수 있다. 지금 생각으로는 pub/sub을 통해서 sub이 일어나기 전에는 read할 수 없게 하려고 하는데 이게 맞는 방향인지는 조금 더 고민이 필요하다.