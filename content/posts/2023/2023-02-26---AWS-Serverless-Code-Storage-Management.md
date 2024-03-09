---
title: "AWS Lambda Code Storage 최적화"
date: "2023-02-26T20:35:37.121Z"
template: "post"
draft: false
slug: "/aws/how-to-manage-lambda-code-storage"
category: "AWS"
tags:
  - "AWS"

description: "불필요한 버전 제거로 code storage 최적화를 통한 비용 절감"
---

### TL;DR

`serverless.yml`에 `versionFunctions: false`를 설정하면 lambda function이 불필요한 버전들을 stacking하는 것을 방지하고 클라우드 비용을 아낄 수 있다. 

------------

모바일 어플리케이션 담당자로 이직하고 [flutter](https://flutter.dev/)로 신나게(?) 앱을 만들고 있었다. 회사에서 만드는 서비스는 

1. 웹에서 어드민이 제작한 자료를
2. Lambda를 활용해서 Dynamo DB에 저장하고
3. 앱에서 보여주는 방식이다. 

기존에 앱을 만들기 전에는 웹에서 만든 자료를 어드민이 로컬 컴퓨터에서 편집해서 사용자에게 문자로 보내는 방식이었는데, 수작업으로 서비스를 운영하다 보니 시스템에 존재하는 다양한 에러들을 인지하지 못한 상황이었다. 수작업으로 서비스를 운영하는 중에도 Lambda를 활용해서 
Dynamo DB에 데이터를 저장하긴 했지만

1. 어떤 식으로 데이터가 저장되는지
2. 저장하는 방식에 어떤 문제가 있는지

아무도 관심을 갖지 않고 있던 상황이었다. 

그대로 두고 운영하기에는 앱에서 데이터를 불러오는데 너무 비효율적이라서 Lambda로 배포된 서버 코드를 수정해야 했다. 하지만 배포를 하려고 보니 Code Storage가 다 찼다는 에러와함께 배포를 진행할 수 없었다. 이전 회사에서는 Azure를 주로 사용해서 AWS도 처음써보고, serverless는 더더욱 처음이라 일단 먼저 입사한 엔지니어에게 공유했고, 먼저 입사한 엔지니어는 일단 code storage limit을 올려줬다. 

|![slack](https://i.imgur.com/BuL4aY5.png)|
|:----:|
|이직한 회사의 엔지니어링 팀은 인원의 절반이 미국에 있어 영어로 대화한다|

일단 배포는 했고, 백그라운드가 화학공학인 나는 이제 5년차 개발자이지만 아직도 비용에 민감하다. 불필요한 비용이 낭비되는 것이 싫어서 Lambda에 들어가보니, 함수 갯수에 비해 너무 많은 용량을 차지하고 있었다

![lambda-taking-up-too-much-space](https://i.imgur.com/RkmwSWf.png)

50개의 함수가 86GB를 잡고있으니, 대충 계산해도 하나의 함수가 1.6GB 정도이다. 그냥 봐도 말이 안된다. 뭐가 문제인가 싶어서 각 함수를 보니, 그동안 배포된 모든 버전을 갖고 있었다. 

![keep-all-the-versions](https://i.imgur.com/yqfsy5K.png)

50개의 함수가 128개의 버전을 가지고 있으니, 가장 최신 버전을 제외하고 6350개의 함수를 AWS Console에서 지워야했다. 함수 버전관리는 좋은게 아닌가? 라고 생각할 수 있지만. 

1. 나는 CI/CD를 구축할 예정이고(블로그를 쓰는 지금은 구축완료상태)
2. 버전관리는 Git으로 하면 되기 때문에

불필요하게 용량을 잡아먹고 비용만 증가시키는 함수들을 정리하기로 했다. 개인적으로 GUI를 별로 좋아하지 않고, 왠지 코드로 지울 수 있을 것 같아서 요즘 가장 친한 친구 ChatGPT에게 물어봤다. 

![chatgpt-how-to-version-control-serverless](https://i.imgur.com/DumofgH.png)

그리고 친절하게 node.js package를 사용한 예제코드도 제공해줬다. 

```javascript
async function deleteOldVersions(functionName, functionArn) {
  const versions = await lambda.listVersionsByFunction({ FunctionName: functionArn }).promise();
  const sortedVersions = versions.Versions.filter(version => version.Version !== '$LATEST').sort((a, b) => b.Version - a.Version);

  for (const version of sortedVersions.slice(NUM_VERSIONS_TO_KEEP)) {
    console.log(`Deleting version ${version.Version} of function ${functionName}`);
    await lambda.deleteFunction({ FunctionName: functionArn, Qualifier: version.Version }).promise();
  }
}

async function deleteOldLogStreams(logGroupName) {
  const logStreams = await logs.describeLogStreams({
    logGroupName,
    orderBy: 'LastEventTime',
    descending: true,
  }).promise();

  for (const logStream of logStreams.logStreams.slice(NUM_LOG_STREAMS_TO_KEEP)) {
    console.log(`Deleting log stream ${logStream.logStreamName} in log group ${logGroupName}`);
    await logs.deleteLogStream({ logGroupName, logStreamName: logStream.logStreamName }).promise();
  }
}
```

각각의 변수들은 AWS config를 사용해서 처리했다. 하지만 하나의 문제가 더 있었는데, Lambda에서 초당 보낼 수 있는 request의 숫자를 제한하고 있었다. 따라서 에러를 막기위해 interval을 추가해서 lambda function들을 지웠다. 불필요한 구버전들을 제거해서 용량을 아낄 수 있었다. 

![lambda-optimized](https://i.imgur.com/y91pTGs.png)

serverless는 처음 써보는거라서 혹시 모르니 최근 2개의 버전을 남겨두는 식으로 작성했고, 85.7GB를 6.7GB로 줄이면서 약 **92%**의 비용을 아낄 수 있게됐다. 배포가 끝나면 구 버전을 삭제하는 코드를 돌리도록 설정하고, 이를 팀에 공유하니, 같이 입사한 AWS경험 많은 다른 개발자분이 config에 `versionFunctions: false`를 추가하면 알아서 구버전들을 제거한다고 알려줬다. 역시 아는 것이 힘이다. 

글또에서 보니 AWS 자격증 12개를 모두 취득한 분이 있더라. 이직하고 AWS를 사용하게 된 김에 틈틈이 공부해서 자격증에 도전해볼 계획이다. 공부하면서 알게된 내용, 일하면서 겪은 문제점들을 해결한 내용들을 꾸준히 블로그에 작성해보도록 하겠다. 