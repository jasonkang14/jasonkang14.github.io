---
title: "플러터에 MVVM 패턴 적용"
date: "2023-05-10T20:35:37.121Z"
template: "post"
draft: false
slug: "/flutter/mvvm-in-flutter-with-riverpod"
category: "Flutter"
tags:
  - "Flutter"

description: "Riverpod를 사용해서 MVVM에서 ViewModel을 구현한 방법"
---

입사하고 [Flutter](https://flutter.dev)를 사용해서 모바일 어플리케이션 개발을 시작했다. 어떤 디자인 패턴을 적용할지 고민하던 중에, 안드로이드는 공식적으로 MVVM 패턴을 권장하는 것을 확인했다. 정확히는 [ViewModel에 대한 설명](https://developer.android.com/topic/libraries/architecture/viewmodel)을 공식문서에서 다루고 있다. 플러터가 구글에서 만든거기도 하고, Material UI를 지원하니, MVVM 패턴을 좋겠다고 생각하던 와중에 네트워킹 이벤트에서 만난 iOS 개발자분이 iOS도 MVVM이 기본이라고 하셔서, 플러터로 진행하는 프로젝트에도 MVVM을 적용해보기로 했다. 

# MVVM (Model-View-ViewModel) 이란?

[Microsoft에서 처음으로 등장한 개념](https://learn.microsoft.com/en-us/xamarin/xamarin-forms/enterprise-application-patterns/mvvm)인데, 

1. Model - 데이터
2. View - UI
3. ViewModel - 데이터와 UI를 연결하는 역할

으로 구분하는 것이다. 

![mvvm-pattern](https://learn.microsoft.com/en-us/xamarin/xamarin-forms/enterprise-application-patterns/mvvm-images/mvvm.png)

글로 표현하자면 Model에 있는 데이터를 ViewModel이 처리해서 View에서 보여주는 방식이다.

# MVVM in Flutter

MVVM을 플러터 용어로 바꾸면, Model은 앱에서 사용되는 데이터를 관리하고, View는 화면을 담당하는 Widget, ViewModel은 [riverpod](https://riverpod.dev/)패키지를 사용하기로 했다. 

### Model

Model은 타입 관리가 중요하기 때문에 [freezed](https://pub.dev/packages/freezed) 패키지를 사용하기로 했다. 물론 Dart에서 기본으로 제공하는 Class를 사용해서 모델을 선언해도 되지만, immutable한 데이터를 사용하고 JSON serialize를 편하게 하기위해서 패키지를 사용하기로 결정했다. freezed는 아래처럼 사용할 수 있다. 

```dart
import 'package:freezed_annotation/freezed_annotation.dart';

part 'user.freezed.dart';
part 'user.g.dart';

@freezed
class User with _$User {
  factory User({
    required String name,
    required String email,
    required String phone,
  }) = _User;

  factory User.fromJson(Map<String, Object?> json) => _$UserFromJson(json);
}
```

위와 같이 입력하고 터미널에서 `flutter pub run build_runner build --delete-conflicting-outputs`를 입력하면 `user.freezed.dart`, `user.g.dart`라는 보일러플레이트를 생성해준다. 

### ViewModel

View에 앞서 freezed를 사용해서 model에서 선언한 데이터를 ViewModel에서 어떻게 처리하는지 알아보자. 

Riverpod는 Provider2.0의 느낌이다. 초반 세팅만 잘하면 매우 편리하게 사용할 수 있다. 앱 자체에서 관리되는 로컬상태 관리하기 위해서는 [StateNotifierProvider](https://riverpod.dev/docs/providers/state_notifier_provider)를 사용하지만, API call은 [FutureProvider](https://riverpod.dev/docs/providers/future_provider)를 사용한다.

Riverpod를 사용해서 구현한 ViewModel은 아래와 같다. 

```dart
final userProvider = FutureProvider.autoDispose<Account>((ref) async {
  final response = await getUserInfo();
  final user = User.fromJSON(jsonDecode(response.data));
  return user;
});
```

실제로는 api call을 처리하는 부분에서 json으로 변환한 후 `User`로 변경하지만, 이해를 돕기위해 provider에 바로 작성했다. 이제 riverpod가 return하는 user를 View를 담당하는 Widget에서 보여준다. 

### View

기본적으로 StatelessWidget을 사용하겠지만, riverpod를 사용하는 경우 ConsumerWidget을 사용한다. 아래와 같은 느낌이다. 

```dart
class UserInfo extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final AsyncValue<Account> user = ref.watch(userProvider);

    return user.when(
      data: (userInfo) {
        return Wrap(
            runSpacing: 8,
            children: [
              Text(
                '이름',
                style: TextStyle(
                  fontSize: Theme.of(context).textTheme.titleMedium?.fontSize,
                  fontWeight: FontWeight.w600,
                ),
              ),
              Text(
                userInfo.name,
                style: TextStyle(
                  fontSize: Theme.of(context).textTheme.bodyLarge?.fontSize,
                ),
              ),
              Text(
                '전화번호',
                style: TextStyle(
                  fontSize: Theme.of(context).textTheme.titleMedium?.fontSize,
                ),
              ),
              Text(
                userInfo.phone,
                style: TextStyle(
                  fontSize: Theme.of(context).textTheme.bodyLarge?.fontSize,
                ),
              ),
              Text(
                '이메일',
                style: TextStyle(
                  fontSize: Theme.of(context).textTheme.titleMedium?.fontSize,
                  fontWeight: FontWeight.w600,
                ),
              ),
              Text(
                userInfo.email,
                style: TextStyle(
                  fontSize: Theme.of(context).textTheme.bodyLarge?.fontSize,
                  fontWeight: FontWeight.w600,
                ),
              ),
            ],
          );
      },
      error: (err, s) => Text(err.toString()),
      loading: () => Center(
        child: CircularProgressIndicator(
          color: Theme.of(context).colorScheme.primary,
        ),
      ),
    );
  }
}
```

`AsyncValue`를 사용하면 자체적으로 상태를 api call이 진행중인 경우 `AsyncLoading`이 되어 Loading indicator를 보여줄 수 있다. 실패한 경우 `AsyncError`는 에러메세지를 보여주고, 성공시 `AsyncData`는 View에서 데이터를 보여준다.

MVVM 패턴을 적용하면 ViewModel에서 데이터를 모두 처리하기 때문에, 같은 데이터를 다양한 Widget에서 보여줘야 하는 경우 해당 데이터를 불러오기만 하면 된다. Widget에서 api call로 볼러온 값을 별도로 처리하지 않아도 되고, 여러 Widget에서 같은 값을 사용하는 경우에도 한번의 api call로 해결이 가능하다. 

플러터로 개발하실 분들은 MVVM 패턴을 적극 추천한다.