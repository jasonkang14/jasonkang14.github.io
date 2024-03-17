---
title: "Flutter Gemini SDK 활용기"
date: "2024-03-17T20:35:37.121Z"
template: "post"
draft: false
slug: "/llm/flutter-gemini-sdk"
category: "LLM"
tags:
  - "LLM"

description: "GPT랑 비교했을 때 과연?"
---

빠르게 LLM 서비스를 테스트 할 니즈가 생겼다. 노코드 툴을 검토해보려다가 개발기간이 1주일밖에 되지 않아서 그냥 빠르게 만들기로 하고, 팀에서 플러터를 사용하니. 일단 플러터로 개발을 시작했다. "실 사용자 대상으로 테스트를 해보고 실패하면 과감하게 접자" 는 의견으로 시작했기 때문에 [Flutter Gemini SDK](https://pub.dev/packages/flutter_gemini)를 한번 사용해보기로 했다. 구글에서 공식으로 제공하는 패키지이기도 하고, 처음 패키지 출시를 발표했을 때 [Flutter 공식 YouTube](https://www.youtube.com/live/sojm449IB-4?feature=shared)채널에서 PST 기준으로 아침일찍 라이브를 할만큼 무언가 대대적인 행사인것 마냥 공개해서 기대를 갖고 연동해보았다. 

```bash
flutter pub add flutter_gemini
```

설치 후에는 FlutterFire를 활용해서 firebase를 연동할 때와 비슷하게 `main()` 함수에서 `.init()`하고 시작한다.

```dart
void main() {

  /// Add this line
  Gemini.init(apiKey: '--- Your Gemini Api Key ---');

  runApp(const MyApp());
}
```

공식문서가 꼼꼼하게 작성되어있어서 연동 자체는 크게 어렵지 않았다. 그리고 [Flutter Gemini GitHub Repository](https://github.com/babakcode/flutter_gemini/tree/master/example/lib/sections)를 확인하면 다양한 케이스에 다른 예제코드가 매우 잘 나와있어서 쉽게 연동해볼 수 있었다. 특히 Gemini의 모델 자체가 공식적으로 런칭이 되지 않아서인지 아직까지는 공짜로 사용할 수 있다는 매력이 있다. 

![gemini-pricing](https://i.imgur.com/DBCm8Kc.png)

OpenAI에서 제공하는 GPT 모델들과의 차이점을 두가지 꼽자면 

1. Gemini에는 **system prompt**가 없다. 

다른 기회로 네이버 클라우드에서 제공하는 [Clova Studio](https://www.ncloud.com/product/aiService/clovaStudio)를 활용해볼 기회가 있었는데. Clova Studio도 OpenAI의 GPT 모델들과 유사하게 system prompt가 존재한다. 하지만 Gemini는 아직은 지원하지 않고, 한동안은 지원하지 않을 계획인 것 같다. [Google Cloud Community](https://www.googlecloudcommunity.com/gc/AI-ML/Implementing-System-Prompts-in-Gemini-Pro-for-Chatbot-Creation/m-p/715501/highlight/true#M5332)에 구글 스태프가 작성한 글을 보면, 메타데이터를 제공하거나 하는 식으로 처리하라고 되어있다. 공식 문서에서 제공하는 예제에서 볼때도 `role`에는 `user`와 OpenAI API를 사용할 때 `assistant`에 해당하는 `model`만 존재한다. 

```dart
final gemini = Gemini.instance;

gemini.chat([
  Content(parts: [
    Parts(text: 'Write the first line of a story about a magic backpack.')],
      role: 'user'),
  Content(parts: [ 
    Parts(text: 'In the bustling city of Meadow brook, lived a young girl named Sophie. She was a bright and curious soul with an imaginative mind.')],
      role: 'model'),
  Content(parts: [ 
    Parts(text: 'Can you set it in a quiet village in 1600s France?')], 
      role: 'user'),
  ])
      .then((value) => log(value?.output ?? 'without output'))
      .catchError((e) => log('chat', error: e));
```

2. Gemini에서는 moderation을 할 수 있는 **Safety Setting**을 제공한다. 

OpenAI API 중에 [moderation](https://platform.openai.com/docs/guides/moderation/quickstart)을 활용하면 감정 분석이 가능하다. 사용자가 LLM 모델이나 우리의 서비스를 악용하는 것을 막기 위해 폭력적이거나 성적인 문구를 포함하는지 등등을 파악하는 절차가 필요하다면, `Moderation API`를 먼저 사용하고 직접 설정한 Threshold를 기준으로 질의를 가능하게 하거나, system prompt를 사용해서 막아야한다. 하지만 Gemini에서는 [Safety Setting](https://pub.dev/packages/flutter_gemini#safety-settings)을 사용해서 이런 설정을 간단하게 할 수 있다. 

```dart
gemini.streamGenerateContent('Utilizing Google Ads in Flutter',
  safetySettings: [
    SafetySetting(
      category: SafetyCategory.harassment,
      threshold: SafetyThreshold.blockLowAndAbove,
    ),
    SafetySetting(
      category: SafetyCategory.hateSpeech,
      threshold: SafetyThreshold.blockOnlyHigh,
    )
  ])
.listen((value) {})
.onError((e) {});
```

공식문서의 예제에서는 `harassment`와 `hateSpeech`만 보여주고 있지만 `sexuallyExplicit`과, `dangerous`와 같은 옵션을 제공해서, 추가로 필터링을 활용할 수 있다.

플러터를 사용하는 팀이라면 Gemini가 유료서비스를 시작하고 조금 더 안정화된다면 충분히 production에서 사용할만 하다고 생각한다. 하지만 지금은 무슨 이유에서인지 종종 500에러가 날 때도 있고, 500에러는 서버에서 예외처리를 잘못했을 때 발생하는 에러라는 점을 감안할 때. 감히 구글의 서비스를 평가하자면 아직은 대규모 서비스에서 사용할만한 수준까지는 올라오지 못한 것 같다. 지금 테스트중인 서비스에서 2가지 기능을 LLM을 연동해서 테스트중인데, 먼저 개발한 기능은 (먼저 프롬프트를 작성한 기능은) gemini를 사용했지만, 다른 기능은 OpenAI의 GPT-3.5를 사용해서 개발했다. 회사에서 Claude3도 활용할 계획이 있는데, 기회가 된다면 Gemini, OpenAI, Claude 모두를 비교해보고 싶다. 