---
title: "플러터 실수 모음"
date: "2023-04-22T20:35:37.121Z"
template: "post"
draft: false
slug: "/flutter/what-i-wish-i-had-kwnown-as-a-react-dev"
category: "Flutter"
tags:
  - "Flutter"
  - "React"

description: "프론트엔드 개발자가 말하는 플러터 개발 전 알았으면 좋았을 것들"
---

이직한지 이제 3개월이 되어간다. 처음으로 주어진 태스크는 2달안에 모바일 앱 프로토타입을 만드는 것. 익숙한 [리액트 네이티브](https://reactnative.dev/)로 하려다가, 생산성과 성능측면에서 더 뛰어나다는 [플러터](https://docs.flutter.dev/get-started/install?gclid=Cj0KCQjwi46iBhDyARIsAE3nVrZ-ksZNHCw_ZVggI5uBngEO3YTD9pIYNGaahA1M7sb13Mo8rra63QcaAkhcEALw_wcB&gclsrc=aw.ds)로 해보기로 했다. 

최근 플러터가 힙해지면서 관심을 갖고 있던차에, 같이 입사한 다른 개발자는 백엔드 + AWS 인프라를 전담하기 원해서 어차피 혼자 개발해야하는 상황이었다. 덕분에 원하던 스택을 경험해볼 수 있었고. 초반에 살짝 힘들었지만 일정에 맞춰 앱을 출시할 수 있었다. 기사라도 났으면 여기 공유했을텐데 살짝 아쉽...지만 어쨌든 1.0.0을 성공적으로 출시했다. 하지만 공식문서를 꼼꼼하게 읽어보지 않은 탓인지, 더 쉽게 개발할 수 있었는데 놓쳤던 부분들을 발견했고, 그것들을 이 포스트에서 공유해보고자 한다. 

1. [freezed](https://pub.dev/packages/freezed)를 활용한 `response.body` 변환

플러터 개발자들은 MVVM 패턴을 적용하는 경우 모델을 관리하기 위해 freezed를 많이 사용한다. freezed를 사용해서 immutable class를 선언하고 타입 체킹을 하는데, 타입스크립트에서 interface, type, 또는 class를 선언하는 것과 유사하다고 보면 된다. 물론 freezed 패키지를 사용하지 않고 Class를 선언해서 타입을 관리할 수도 있지만, freezed를 사용하면 

1. boilerplate 코드를 많이 작성하지 않아도 되고, 
2. serialization을 제공한다는 이점이 있다. 

나는 이 serialization을 제대로 사용하지 못했다. 

사용자 정보를 불러오는 API의 response body를 처리한다고 가정하고 기존에 작성했던 코드를 먼저 보자. 

```dart
final response = await http.get('profile'); 
if (response.statusCode == HttpStatus.ok) {

  final jsonBody = jsonDecode(response.body);
  final profile = Profile(
    id: jsonBody['id'],
    name: jsonBody['name'],
  );
}
```

위와 같은 느낌이다. 

`jsonBody['id']`와 같이 접근하는 이유는, json decode된 값의 타입이 `Map<String, String>`이기 때문이다. 다트에서 [Map](https://api.dart.dev/be/180791/dart-core/Map-class.html)은 자바스크립트의 객체와 비슷하다. 하지만 다트의 Map은 `jsonBody.id` 와 같은 방식으로 접근할 수 없고 `jsonBody['id']`로 접근해야 한다. 위 예제에서 `Profile`은 freezed를 사용해서 선언된 클래스이다. 이런 느낌으로 생겼다. 

```dart
@freezed
class Profile with _$Profile {
  factory Profile({
    required int id,
    required String name,
  }) = _Profile;

  factory Profile.fromJson(Map<String, Object?> json) => _$ProfileFromJson(json);
}
```

jsonBody를 `Profile`로 변경해주는 이유는, 다트에서 꽤나 좋은 타입체크 기능을 지원하는데, Map을 사용해서는 그 기능을 활용할 수 없기 때문이다. 그래서 처음 예제처럼 Profile 인스턴스를 새로 선언해서 값을 변경시켜줬다. 하지만 위 예제에서 보이는 것처럼 `.fromJson()`이라는 기능을 제공한다. Map타입을 넣어서 호출하면 바로 `Profile` 인스턴스로 해당 값을 변경시켜주는 것이다. 만약 서버에서 오는 response body를 클라이언트에서 바로 사용할 수 있도록 합의했다면, 처음 작성했던 코드는 아래처럼 간단해질 수 있다. 

```dart
final response = await http.get('profile'); 
if (response.statusCode == HttpStatus.ok) {
  final jsonBody = jsonDecode(response.body);
  final profile = Profile.fromJson(jsonBody);
}
```

원한다면 one liner도 가능하겠지만, 개인적으로 가독성이 떨어진다고 생각해서 지양하는 편이다. 코드는 간단해졌고, 타입체킹을 바로 해주기때문에, 서버에서 response가 바뀌거나 하는 경우 바로 확인이 가능하다. 공식문서를 꼼꼼하게 읽어보지 못한 불찰이다. 

2. `ListView` 활용

리액트에서는 리스트를 활용해서 UI를 그릴 때 `Array.prototype.map()`을 사용한다. 리액트 공식문서에 [rendering lists](https://react.dev/learn/rendering-lists)라는 섹션이 따로 있을 정도로 흔히 사용되는 개념이다. 다트로 리스트에 `.map()`이라는 method를 지원해서 평소에 쓰던대로 사용해봤다. MVVM패턴의 ViewModel을 구현하기 위해 riverpod라는 패키지를 사용중인데, 해당 패키지를 사용하면 코드를 아래와 같이 쓸 수 있다. 

```dart
...
list.when(
  data: (list) => list.map((item) => Column(
    children: [
      Text(item.name), 
      Text(item.email),
    ],
  )).tolist(),
)
...
```

map을 사용해서 만든 값들은 `Iterable<Widget>`이기 때문에 뒤에 `.tolist()`를 호출해서 `List<Widget>`으로 변경해야한다. 그리고 다트에서의 `.map()`과 자바스크립트의 `.map()`의 가장 큰 차이점은, 다트는 리스트의 인덱스에 접근하는 기능을 제공하지 않는다는 것이다. 그래서 인덱스가 필요한 경우 별도의 값을 해당 클래스에 추가해서 사용했다. 

```dart
final tempList = [
  Temp(
    id: 0,
    name: 'test1',
  ),
  Temp(
    id: 1,
    name: 'test2',
  ),
]
```

이런 느낌이다. 만약 인덱스에 해당하는 값을 사용하려면 필요 없을수도 있는 값을 하나 추가해서 관리해야 한다. 

하지만 플러터는 map을 사용해서 인덱스에 접근하는 것이 아니라, `ListView`의 `itemBuilder`를 사용해서 인덱스에 접근하게 한다. 나중에 공식문서에서 [Work with long lists](https://docs.flutter.dev/cookbook/lists/long-lists)라는 섹션이 따로 있는 것을 보고 알았다. 

처음 작성했던 코드는 아래와 같이 변경될 수 있다. 

```dart
list.when(
  data => (list) => ListView.builder(
    itemBuilder: (context, index) => Column(
      children: [
        Text(list[index].name),
        Text(list[index].email),
      ],
    ),
  ),
)
```

`itemBuilder`를 통해 index를 사용할 수 있고, 위의 예제처럼 별도의 값을 추가해주지 않아도 된다는 이점이 있다. 공식문서를 꼼꼼하게 읽어보지 못한 불찰이 크지만, 그래도 같은 실수를 할 개발자분들이 있을 수 있으니 한 번 공유해본다. 다음 포스트에서는 freezed + riverpod를 사용해서 MVVM 패턴을 어떻게 구축했는지에 대해 작성해보도록 하겠다. 