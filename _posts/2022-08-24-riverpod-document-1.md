---
title: "상태관리 패키지 Riverpod 공식문서 정리"
categories:
  - Riverpod
tags:
  - [Dart, Flutter]

toc: true
toc_sticky : true

date : 2023-08-24
last_modified_at : 2023-08-24
---

Flutter의 상태관리 패키지인 Riverpod의 공식 문서 내용을 공부하면서 정리해보고자 합니다.

flutter hooks와 code generation은 사용하지 않는다고 가정하고, 이 기능들에 대해선 추후에 따로 글을 올리겠습니다.

# Why is Riverpod?

## What is Riverpod?

Riverpod(Provider의 anagram)은 Dart/Flutter의 반응형 캐싱 프레임워크입니다. 

선언적(Declararive) 및 반응형(Reactive) 프로그래밍을 사용하여 애플리케이션의 상당 부분의 로직을 자동으로 처리할 수 있으며, 빌트인 되어 있는 오류 처리 및 캐싱 기능을 통해 네트워크 요청을 실행하면서 필요할 때 데이터를 자동으로 다시 가져올 수 있습니다.

Riverpod은 기존 Provider 패키지를 개선하여 애플리케이션 로직 및 상태 관리에 있어 뛰어난 효율성과 관리성을 제공합니다.

## Motivation

현대의 애플리케이션들은 데이터를 종종 서버에서 비동기적으로 가져옵니다.

문제는 비동기 코드를 사용하는 것이 어렵다는 점입니다. Flutter에는 상태 변수를 만들고 UI 변경시 업데이트하는 방법이 있지만, 여전히 제한적이고 다음과 같은 문제들이 존재합니다.

- 비동기 요청은 로컬에 캐시(Cache) 되어야 합니다(UI가 업데이트될 때마다 이를 다시 실행하는 것은 비합리적입니다).
- 캐시를 사용할때 주의하지 않으면 오래된 데이터로 남을 수 있습니다.
- 오류와 로딩 상태를 처리해야 합니다.

이 문제들은 규모에 따라 다루기 어려울 수 있으며, 아래와 같은 문제점을 파생시킵니다.

- 새로 고침 기능
- 무한 스크롤링 및 데이터 불러오기
- 검색창에서 입력과 동시에 검색(검색 결과 미리보기)
- 비동기 요청 디바운싱(연속적으로 발생한 이벤트를 하나로 묶어서 처리하는 방식)
- 사용되지 않는 비동기 요청 취소
- 최적화된 UI
- 오프라인 모드
- 기타 등등

이러한 기능들은 구현하기 까다롭지만, 사용자 경험을 위해 필수적입니다.

그러나 이러한 문제들을 해결해주기 위한 패키지는 많지 않아 보통은 작업을 수동으로 작성해야 합니다.

이런 점들이 Riverpod 탄생한 이유이며, Riverpod은 Flutter 위젯에서 영감을 받은 새로운 방식으로 비즈니스 로직을 작성함으로써 이러한 문제를 해결하는데 초점을 맞춥니다. 

다양한 면에서 Riverpod은 위젯과 비교할 수 있지만, 상태와 관련된 복잡한 기능들을 대부분 손쉽게 수행할 수 있도록 도와주므로 개발자는 UI 중심 작업에 집중할 수 있습니다.

Riverpod을 사용하여 구현된 아래의 Pub.dev 클라이언트 애플리케이션의 간소화된 예제 코드를 확인해보겠습니다.

```dart
// pub.dev의 패키지 목록을 가져올 것
final fetchPackagesProvider = FutureProvider.autoDispose
    .family<List<Package>, ({int page, String? search})>((ref, params) async {
    
  final page = params.page;
  final search = params.search ?? '';
  final dio = Dio();
  
  // Dio 패키지를 이용하여 패키지 목록을 가져오는 API를 호출
  final response = await dio.get<List<Object?>>(
    'https://pub.dartlang.org/api/search?page=$page&q=${Uri.encodeQueryComponent(search)}',
  );

  // JSON으로 디코딩하고 모델 객체를 생성하여 리스트로 반환
  return response.data?.map(Package.fromJson).toList() ?? const [];
});
```
이 스니펫(Snippet)은 앞서 설명한 기능들을 처리할 수 있는 로직을 전부 포함하고 있습니다.

- 함수 fetchPackages는 search 매개 변수를 받아 URL을 생성하며, search 값이 변경될 때 변경된 값을 URL에도 반영하여 새로운 결과를 얻을 수 있습니다. 따라서 입력 시 자동으로 검색이 가능합니다.
- 무한 스크롤링 기능을 구현하려면 페이지 번호를 기반으로 결과를 받아올 수 있어야 합니다. 위 코드에선 page 매개 변수를 이용해 현재 페이지를 받아오고 있으며, 페이지 번호가 변경되면 자동으로 새로운 페이지 결과를 받아오게 할 수 있습니다.
- 새로 고침 기능은 사용자가 다시 데이터를 불러올 수 있어야 하며, 이 또한 fetchPackages 함수를 재 호출함으로써 가능합니다.
- FutureProvider을 통해 데이터의 상태에 따른 로딩 및 오류처리에 대해 유연하게 대처할 수 있습니다.

# Getting started

## Installing the package

수동으로 패키지 의존성을 추가할 경우 다음과 같이 진행하며, 보다 자세한 패키지 종속성에 대한 이해관계는 [pub.dev](https://pub.dev) 를 참조하길 바랍니다.

```yaml
# pubspec.yaml

name: my_app_name
environment:
  sdk: ">=3.0.0 <4.0.0" # Dart SDK 버전 지정
  flutter: ">=3.0.0" # Flutter SDK 버전 지정

dependencies:
  flutter:
    sdk: flutter # Flutter 프레임워크를 사용하기 위한 기본 패키지
  # Riverpod 기본 패키지
  flutter_riverpod: ^2.3.6

dev_dependencies:
  # 프로젝트에 사용자 정의 린트 규칙을 적용하는 데 도움을 주는 패키지
  custom_lint:
  # Riverpod 프레임워크에 최적화된 린트 규칙을 제공하는 패키지
  riverpod_lint: ^1.4.0
```

패키지 종속성 추가 후 'flutter pub get' 명령어를 통해 패키지를 설치하고,
'dart run build_runner watch' 명령어를 통해 code-generator를 실행할 수 있습니다.

## Enabling riverpod_lint/custom_lint

Riverpod은 코드 품질 향상을 돕고 사용자 정의 리팩터링 옵션을 제공하기 위해 riverpod_lint 패키지를 포함하고 있으며, pubspec.yaml 파일이 위치한 곳에 analysis_options.yaml 파일을 새로 추가하고 다음 내용을 작성하여 활성화 할 수 있습니다.

```yaml
# analysis_options.yaml

analyzer:
  plugins:
    - custom_lint
```

이제 Riverpod을 코드에서 사용할 때 실수를 했다면 IDE에서 경고를 볼 수 있습니다.

전체 경고 목록과 리팩터링 내용 등, 보다 자세한 내용은 [pub.dev](https://pub.dev)에서 riverpod_lint 페이지를 참조하시면 됩니다.

참고로 이 경고들은 'dart analyze' 명령어로 보이지 않기 때문에 CI(Continuous Integration) / 터미널을 이용하는 경우, 따로 'dart run custom_lint' 명령어를 실행하여 경고들을 볼 수 있습니다.

## Usage example: Hello world

다음 코드는 Riverpod을 사용하여 만든 "Hello world" 예제입니다.

```dart
//main.dart

import 'package:flutter/material.dart';
import 'package:flutter_hooks/flutter_hooks.dart';
import 'package:hooks_riverpod/hooks_riverpod.dart';
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'main.g.dart';

// "Hello world" 문자열을 상태 값으로 가지는 프로바이더 생성
// 프로바이더를 이용하면 상태값을 재정의(override)하거나 모조값(mock)으로 대체 가능합니다
final helloWorldProvider = Provider((_) => 'Hello world');

void main() {
  runApp(
	// 위젯이 provider를 읽을 수 있도록 전체 애플리케이션을 "ProviderScope" 위젯으로 감쌉니다
	// 이곳에서 provider의 상태가 저장됩니다
    ProviderScope(
      child: MyApp(),
    ),
  );
}


// Riverpod의 상태값을 이용하기 위해 StatelessWidget 대신 ConsumerWidget을 상속합니다
class MyApp extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final String value = ref.watch(helloWorldProvider);

    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(title: const Text('Example')),
        body: Center(
          child: Text(value),
        ),
      ),
    );
  }
}
```

# Concepts

## Providers

Riverpod에서 가장 중요한 부분은 프로바이더입니다. 프로바이더는 상태의 일부를 캡슐화한 객체로, 해당 상태에 대한 수신을 허용합니다.

### Why use providers?

프로바이더를 통해 상태를 관리하면 다음과 같은 이점들이 존재합니다

- 프로바이더는 싱글톤(Singletons), 서비스 로케이터(Service Locators), 의존성 주입(Dependency Injection), 상속 위젯(InheritedWidgets) 등의 패턴을 완전히 대체하고, 여러 위치에서 상태를 쉽게 접근할 수 있습니다.
- 상태를 다른 상태와 간편하게 결합할 수 있습니다.
- 위젯 재구성 필터링이나 높은 비용이 드는 상태 계산 캐싱 등, 상태 변경에 영향을 받는 부분만 재계산되는 것을 보장합니다.
- 프로바이더를 사용하면 복잡한 setUp/tearDown 단계가 필요하지 않고 테스트 중 다른 행동을 적용할 수 있도록 재정의할 수 있어, 매우 구체적인 동작을 쉽게 테스트할 수 있습니다.
- 로깅(Logging)이나 pull-to-refresh와 같은 고급 기능과의 쉬운 통합을 가능하게 합니다.

이러한 이점들로 인해 프로바이더를 사용하면 애플리케이션의 상태 관리가 쉽고 효율적으로 이루어질 수 있습니다.

### Creating a provider

프로바이더는 다양한 종류가 있지만, 모두 동일한 방식으로 작동합니다. 

가장 일반적인 사용법은 전역 상수로 선언하는 것이며, 다음과 같이 작성할 수 있습니다.

```dart
final myProvider = Provider((ref) {
  return MyValue();
});
```

프로바이더를 전역적으로 선언하더라도 변경이 불가능하고, 함수를 선언하는 것과 다르지 않으며, 테스트가 가능하고 유지 보수가 쉽기 때문에 걱정하실 필요는 없습니다.

위의 코드 스니펫(Snippet)은 다음과 같이 분석할 수 있습니다.

- final myProvider : myProvider 변수는 추후에 프로바이더 상태를 읽기 위해 사용할 것이며, 프로바이더는 항상 변경 불가능한 final로 선언하는 것이 좋습니다.
- Provider : Provider는 모든 프로바이더 중 가장 기본적인 것이고, 변경이 불가능한 객체를 반환합니다. 또한 Provider를 StreamProvider나 NotifierProvider 등의 다른 프로바이더로 교체하여 값을 다루는 방식을 변경할 수 있습니다.
- 상태 생성 함수 : 이 함수는 'ref'라는 매개변수에 Ref 객체가 들어갑니다. 이 객체를 통해 다른 프로바이더를 읽거나, 프로바이더 상태가 소멸될 때 일부 작업을 수행할 수 있습니다.

프로바이더에게 전달한 상태 생성 함수에 의해 반환되는 객체의 종류는 사용하는 프로바이더에 따라 달라집니다. 예를 들어, 프로바이더는 어떠한 종류의 객체든 생성하여 반환할 수 있으며, 스트림 프로바이더(StreamProvider)는 스트림을 반환할 것으로 예상할 수 있습니다.

또한 기존 Provider 패키지와는 달리, Riverpod의 경우 동일한 타입의 상태를 가진 Provider를 제한 없이 여러 개 선언할 수 있습니다.

```dart
final cityProvider = Provider((ref) => 'London');
final countryProvider = Provider((ref) => 'England');
```

단, 위젯이 provider를 읽을 수 있도록 전체 애플리케이션을 "ProviderScope" 위젯으로 감싸야 한다는 것만 주의하시면 됩니다.

```dart
void main() {
  runApp(ProviderScope(child: MyApp()));
}
```

### Different Types of Providers

프로바이더의 종류는 다양하며, 상황에 맞게 적절히 골라 사용하시면 됩니다.

각 프로바이더에 대한 간략한 설명은 아래와 같습니다.

- Provider : 어떠한 타입이든 반환할 수 있으며, 서비스 클래스 혹은 필터링된 목록을 상태값으로 유지하기에 적합합니다
- StateProvider : 어떠한 타입이든 반환할 수 있으며, 필터 조건 혹은 간단한 상태 객체를 상태값으로 유지하기에 적합합니다
- FutureProvider : Future 형태의 타입을 반환할 수 있으며, API 호출 결과를 상태값으로 유지하기에 적합합니다
- StreamProvider : Stream 형태의 타입을 반환할 수 있으며, API 호출 결과를 스트림하여 상태값으로 유지하기에 적합합니다
- NotifierProvider : (Async)Notifier의 서브클래스 타입을 반환할 수 있으며, 복잡한 상태 객체를 유지하기에 적합합니다
- StateNotifierProvider : StateNotifier의 서브클래스 타입을 반환할 수 있으며, 복잡한 상태 객체를 유지하기에 적합합하지만 NotifierProvider 사용을 권장합니다
- ChangeNotifierProvider : ChangeNotifier의 서브클래스 타입을 반환할 수 있으며, 변경 가능성이 필요한 복잡한 상태 객체를 유지하기에 적합하지만 가변 상태와 관련된 문제 때문에 사용을 권장하지 않습니다

### Provider Modifiers

모든 프로바이더에는 추가 기능을 제공할 수 있도록 내장되어 있는 방법(built-in way)이 존재합니다. 즉, ref 객체에 새로운 기능을 추가하거나 프로바이더가 사용되는 방식을 약간 변경할 수 있습니다. 

Modifiers는 모든 프로바이더에서 사용할 수 있으며, 아래와 같이 이름이 지정된 생성자와 유사한 구문으로 사용할 수 있습니다.

```dart
final myAutoDisposeProvider = StateProvider.autoDispose<int>((ref) => 0);
final myFamilyProvider = Provider.family<String, int>((ref, id) => '$id');

//또는
final userProvider = FutureProvider.autoDispose.family<User, int>((ref, userId) async {
  return fetchUser(userId);
});
```

위의 Modifiers 기능은 다음과 같습니다:

- autoDispose : 프로바이더가 더이상 사용되지 않을 때 자동으로 상태를 제거하는 기능을 추가합니다.
- family : 외부 매개변수를 통해 프로바이더를 생성하는 기능을 추가합니다.

# All Providers

## Provider

Provider는 모든 종류의 프로바이더들 중에서 가장 기본이 되는 존재이며, 새로운 상태값을 생성하고 유지할 수 있습니다.

Provider는 주로 다음과 같은 경우에 사용됩니다.

- 계산 결과 캐싱(Caching)
- 다른 프로바이더에게 상태값 제공 (Ex: 레포지토리/HttpClient)
- 테스트나 위젯에서 값을 재정의(Override)할 수 있는 방법 제공
- 'select'를 사용하지 않고 프로바이더/위젯의 리빌드(rebuild) 수 줄이기

### Using Provider to cache computations

Provider는 ref.watch와 결합하여 동기 작업을 캐싱하는 데 강력한 도구입니다. 

예를 들어 Todo 목록 필터링 작업의 경우, 목록을 필터링 것은 비용이 발생할 수 있으므로 애플리케이션이 다시 렌더링 될 때마다 Todo 목록을 필터링하는 것은 효율적이지 않습니다.

이런 상황에서는 Provider를 사용하여 필터링 작업을 처리할 수 있습니다. 

아래의 Todo List 구현 예시를 살펴보겠습니다.

```dart
class Todo {
  Todo(this.description, this.isCompleted);
  final bool isCompleted;
  final String description;
}

class TodosNotifier extends Notifier<List<Todo>> {
  @override
  List<Todo> build() {
    return [];
  }

  void addTodo(Todo todo) {
    state = [...state, todo];
  }
}

final todosProvider = NotifierProvider<TodosNotifier, List<Todo>>(() {
  return TodosNotifier();
});
```
여기서 아래와 같이 Provider를 사용하여 완료된 Todo만 표시하도록 필터링한 목록을 보여줄 수 있습니다.

```dart
final completedTodosProvider = Provider<List<Todo>>((ref) {
  // todosProvider로 부터 Todo List를 불러옵니다
  final todos = ref.watch(todosProvider);

  // 완료된 Todo List를 불러옵니다
  return todos.where((todo) => todo.isCompleted).toList();
});
```

이제 UI는 completedTodosProvider를 듣고 완료된 Todo List를 표시할 수 있습니다.

```dart
Consumer(builder: (context, ref, child) {
  final completedTodos = ref.watch(completedTodosProvider);
});
```

여기서 흥미로운 점은 Todo List 필터링이 캐시된다는 것입니다. 

즉, Todo가 추가/제거/업데이트되지 않는 한 완료된 Todo List는 여러 번 읽어도 다시 계산되지 않습니다. 

따라서 Todo List가 변경될 때 캐시를 수동으로 갱신할 필요가 없으며, ref.watch 덕분에 Provider는 결과가 다시 계산되어야 할 때를 자동으로 알 수 있습니다.

### Reducing provider/widget rebuilds by using Provider

Provider의 특징은 값이 변경되지 않으면 Provider가 다시 계산되더라도(일반적으로 ref.watch를 사용하는 경우) 위젯/프로바이더들이 업데이트되지 않는다는 점입니다.

예시로 페이지를 변경하는 UI에서 "이전" 버튼 및 "다음" 버튼을 활성화/비활성화하는 경우를 들 수 있습니다.

단순한 구현 예시로, 아래와 같이 현재 페이지 인덱스를 가져와 인덱스가 0과 같으면 "이전" 버튼을 비활성화하는 위젯을 가정해 보겠습니다.

```dart
final pageIndexProvider = StateProvider<int>((ref) => 0);

class PreviousButton extends ConsumerWidget {
  const PreviousButton({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // 첫 번째 페이지가 아니라면, "이전" 버튼이 활성화됩니다
    final canGoToPreviousPage = ref.watch(pageIndexProvider) != 0;

    void goToPreviousPage() {
      ref.read(pageIndexProvider.notifier).update((state) => state - 1);
    }

    return ElevatedButton(
      onPressed: canGoToPreviousPage ? goToPreviousPage : null,
      child: const Text('previous'),
    );
  }
}
```

위 코드의 문제점은 현재 페이지를 변경할 때마다 "이전" 버튼이 다시 빌드된다는 것입니다.

버튼이 활성화와 비활성화 사이를 변경할 때만 다시 빌드되면 좋겠지만, 문제의 근본적인 원인은 "이전" 버튼 내부에서 사용자가 이전 페이지로 이동할 수 있는지를 계산한다는 점입니다. 

이런 종류의 문제는 아래와 같이 로직을 위젯 외부로 빼내고 Provider로 옮겨서 해결할 수 있습니다.

```dart
final pageIndexProvider = StateProvider<int>((ref) => 0);

// 사용자가 이전 페이지로 갈 수 있는지를 계산하는 프로바이더
final canGoToPreviousPageProvider = Provider<bool>((ref) {
  return ref.watch(pageIndexProvider) != 0;
});

class PreviousButton extends ConsumerWidget {
  const PreviousButton({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // 여기서 기존 프로바이더가 아닌 새로운 canGoToPreviousPageProvider를 관찰합니다
    // 이 버튼 위젯은 이제 더이상 이전 페이지로 갈 수 있는지의 여부를 계산하지 않습니다
    final canGoToPreviousPage = ref.watch(canGoToPreviousPageProvider);

    void goToPreviousPage() {
      ref.read(pageIndexProvider.notifier).update((state) => state - 1);
    }

    return ElevatedButton(
      onPressed: canGoToPreviousPage ? goToPreviousPage : null,
      child: const Text('previous'),
    );
  }
}
```

해당 리팩토링(Refactoring)을 통해 페이지 인덱스가 변경되면 canGoToPreviousPageProvider가 다시 계산되며, 값이 변경되지 않을 경우 PreviousButton은 다시 빌드되지 않습니다. 

이러한 변경으로 위젯의 성능을 개선시키고, 로직을 외부로 추출하는 이점을 얻을 수 있습니다.

## (Async)NotifierProvider

NotifierProvider는 Notifier를 듣고 전달하는 데 사용되는 프로바이더며, AsyncNotifierProvider는 비동기적으로 초기화될 수 있는 AsyncNotifier를 듣고 전달하는 데 사용되는 프로바이더입니다. 

(Async)NotifierProvider와 (Async)Notifier는 사용자의 상호작용에 대한 반응으로 변할 수 있는 상태 관리에 탁월하며 주로 다음과 같은 경우에 사용됩니다.

- 커스텀 이벤트에 대한 반응으로 시간이 지남에 따라 변경될 수 있는 상태를 전달합니다
- 비즈니스 로직을 한 곳에서 중앙화하여 시간이 지남에 따른 유지 보수성을 향상시킵니다

아래의 Todo List 예시를 한 번 살펴보겠습니다.

```dart
// 불변인 상태를 유지하는 것이 좋습니다
// 이를 위해 Freezed와 같은 패키지를 사용하는 것을 추천드립니다
@immutable
class Todo {
  const Todo({
    required this.id,
    required this.description,
    required this.completed,
  });

  // 클래스의 모든 속성은 final이어야 합니다
  final String id;
  final String description;
  final bool completed;

  // Todo가 불변이므로 수정할 수 없기 때문에 Todo를 복제하는 메서드를 구현합니다.
  Todo copyWith({String? id, String? description, bool? completed}) {
    return Todo(
      id: id ?? this.id,
      description: description ?? this.description,
      completed: completed ?? this.completed,
    );
  }
}

// NotifierProvider에 전달할 Notifier 클래스입니다
// 이 클래스는 "state" 속성을 외부에 상태를 노출해서는 안 되므로,
// 공개 getter/properties는 없어야 합니다
// 이 클래스의 공개 메서드를 통해 UI가 상태를 수정할 수 있게 합니다
class TodosNotifier extends Notifier<List<Todo>> {
  // Todo List를 빈 목록으로 초기화합니다
  @override
  List<Todo> build() {
    return [];
  }

  void addTodo(Todo todo) {
    // 상태가 불변이기 때문에 state.add(todo)를 할 수 없습니다
    // 대신 이전 항목과 새로운 항목을 포함하는 새로운 할 일 목록을 만듭니다
    // 여기서 Dart의 스프레드 연산자를 사용하는 것이 좋습니다
    state = [...state, todo];
    // 기존 Provider 패키지와 다르게 상태 변경을 위해 "notifyListeners"를 호출할 필요가 없고, 
	// "state =" 를 호출하면 필요할 때 자동으로 UI를 다시 빌드합니다
  }

  void removeTodo(String todoId) {
    // 상태가 불변이므로 기존 목록을 변경하는 대신 새 목록을 만듭니다
    state = [
      for (final todo in state)
        if (todo.id != todoId) todo,
    ];
  }

  void toggle(String todoId) {
    state = [
      for (final todo in state)
        // ID가 일치하는 Todo를 완료로 표시합니다
        if (todo.id == todoId)
          // 상태가 불변이므로 할 일의 사본을 만들어야 합니다
		  // 그러기 위해 전에 구현한 copyWith 메서드를 사용합니다
          todo.copyWith(completed: !todo.completed)
        else
          // 다른 경우엔 Todo가 수정되지 않습니다
          todo,
    ];
  }
}

//마지막으로, NotifierProvider를 사용하여 UI가 TodosNotifier와 상호 작용하게 합니다
final todosProvider = NotifierProvider<TodosNotifier, List<Todo>>(() {
  return TodosNotifier();
});
```

이제 NotifierProvider를 정의했으므로 UI에서 Todo List를 이용해 보겠습니다.

```dart
class TodoListView extends ConsumerWidget {
  const TodoListView({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
  
    // Todo List가 변경될 때 위젯을 다시 빌드합니다
    List<Todo> todos = ref.watch(todosProvider);

    return ListView(
      children: [
        for (final todo in todos)
          CheckboxListTile(
            value: todo.completed,
            // Todo를 누르면 상태가 "완료"로 변경됩니다
            onChanged: (value) =>
                ref.read(todosProvider.notifier).toggle(todo.id),
            title: Text(todo.description),
          ),
      ],
    );
  }
}
```

이번에는 예제를 조금 변형하여 AsyncNotifierProvider를 통해 원격 리포지토리를 이용한 Todo List를 구현 해보겠습니다.

```dart
// 불변인 상태를 유지하는 것이 좋습니다
// 이를 위해 Freezed와 같은 패키지를 사용하는 것을 추천드립니다
@immutable
class Todo {
  const Todo({
    required this.id,
    required this.description,
    required this.completed,
  });

  // JSON 데이터를 Todo 객체로 변환하는 팩토리 메서드
  factory Todo.fromJson(Map<String, dynamic> map) {
    return Todo(
      id: map['id'] as String,
      description: map['description'] as String,
      completed: map['completed'] as bool,
    );
  }

  // 클래스의 모든 속성은 final이어야 합니다
  final String id;
  final String description;
  final bool completed;

  // Todo 객체를 JSON 형식으로 변환하는 메서드
  Map<String, dynamic> toJson() => <String, dynamic>{
        'id': id,
        'description': description,
        'completed': completed,
      };
}

class AsyncTodosNotifier extends AsyncNotifier<List<Todo>> {

  // 원격 리포지토리에서 할 일 목록을 가져옵니다
  Future<List<Todo>> _fetchTodo() async {
    final json = await http.get('api/todos');
    final todos = jsonDecode(json) as List<Map<String, dynamic>>;
    return todos.map(Todo.fromJson).toList();
  }

  @override
  Future<List<Todo>> build() async {
    // 리모트 저장소에서 초기 Todo List를 가져옵니다
    return _fetchTodo();
  }

  Future<void> addTodo(Todo todo) async {
    // 로딩 상태로 설정합니다
    state = const AsyncValue.loading();
    // 새로운 Todo를 추가하고, 원격 리포지토리에서 할 일 목록을 다시 로드합니다
    state = await AsyncValue.guard(() async {
      await http.post('api/todos', todo.toJson());
      return _fetchTodo();
    });
  }

  Future<void> removeTodo(String todoId) async {
    state = const AsyncValue.loading();
    state = await AsyncValue.guard(() async {
      await http.delete('api/todos/$todoId');
      return _fetchTodo();
    });
  }

  Future<void> toggle(String todoId) async {
    state = const AsyncValue.loading();
    state = await AsyncValue.guard(() async {
      await http.patch(
        'api/todos/$todoId',
        <String, dynamic>{'completed': true},
      );
      return _fetchTodo();
    });
  }
}

final asyncTodosProvider =
    AsyncNotifierProvider<AsyncTodosNotifier, List<Todo>>(() {
  return AsyncTodosNotifier();
});
```

이제 AsyncNotifierProvider를 정의했으므로 UI에서 Todo List를 이용해 보겠습니다.

```dart
class TodoListView extends ConsumerWidget {
  const TodoListView({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final asyncTodos = ref.watch(asyncTodosProvider);
	//asyncTodos.when을 통해 상태에 따라 다른 위젯을 반환할 수 있습니다
    return asyncTodos.when(
      data: (todos) => ListView(
        children: [
          for (final todo in todos)
            CheckboxListTile(
              value: todo.completed,
              onChanged: (value) =>
                  ref.read(asyncTodosProvider.notifier).toggle(todo.id),
              title: Text(todo.description),
            ),
        ],
      ),
      loading: () => const Center(
        child: CircularProgressIndicator(),
      ),
      error: (err, stack) => Text('Error: $err'),
    );
  }
}
```

## StateNotifierProvider

StateNotifierProvider는 StateNotifier를 듣고 전달하는 데 사용되는 프로바이더입니다.

다만, Riverpod에서는 NotifierProvider + Notifier을 사용할 것을 권장하고 있으니 참고하시길 바랍니다.

StateNotifierProvider는 주로 다음과 같은 경우에 사용됩니다.

- 사용자 정의 이벤트에 대한 반응으로 시간의 흐름에 따라 변경될 수 있는 불변한 상태를 전달합니다
- 비즈니스 로직을 한 곳에서 중앙 집중화하여 시간이 지남에 따른 유지 보수성을 향상시킵니다

예시로 StateNotifierProvider를 통해 Todo List를 구현해 보겠습니다.

```dart
// 불변인 상태를 유지하는 것이 좋습니다
// 이를 위해 Freezed와 같은 패키지를 사용하는 것을 추천드립니다
@immutable
class Todo {
  const Todo({required this.id, required this.description, required this.completed});

  // 클래스의 모든 속성은 final이어야 합니다
  final String id;
  final String description;
  final bool completed;

  // Todo가 불변이므로 수정할 수 없기 때문에 Todo를 복제하는 메서드를 구현합니다.
  Todo copyWith({String? id, String? description, bool? completed}) {
    return Todo(
      id: id ?? this.id,
      description: description ?? this.description,
      completed: completed ?? this.completed,
    );
  }
}

// 이 클래스는 "state" 속성을 외부에 상태를 노출해서는 안 되므로,
// 공개 getter/properties는 없어야 합니다
// 이 클래스의 공개 메서드를 통해 UI가 상태를 수정할 수 있게 합니다
class TodosNotifier extends StateNotifier<List<Todo>> {
  // Todo List를 빈 리스트로 초기화합니다
  TodosNotifier(): super([]);

  void addTodo(Todo todo) {
   	// 상태가 불변이기 때문에 state.add(todo)를 할 수 없습니다
    // 대신 이전 항목과 새로운 항목을 포함하는 새로운 할 일 목록을 만듭니다
    // 여기서 Dart의 스프레드 연산자를 사용하는 것이 좋습니다
    state = [...state, todo];
    // 기존 Provider 패키지와 다르게 상태 변경을 위해 "notifyListeners"를 호출할 필요가 없고, 
	// "state =" 를 호출하면 필요할 때 자동으로 UI를 다시 빌드합니다
  }

  void removeTodo(String todoId) {
    // 상태가 불변이므로 기존 목록을 변경하는 대신 새 목록을 만듭니다
    state = [
      for (final todo in state)
        if (todo.id != todoId) todo,
    ];
  }

  void toggle(String todoId) {
    state = [
      for (final todo in state)
        // ID가 일치하는 Todo를 완료로 표시합니다
        if (todo.id == todoId)
          // 상태가 불변이므로 할 일의 사본을 만들어야 합니다
		  // 그러기 위해 전에 구현한 copyWith 메서드를 사용합니다
          todo.copyWith(completed: !todo.completed)
        else
          // 다른 경우엔 Todo가 수정되지 않습니다
          todo,
    ];
  }
}

//마지막으로, StateNotifierProvider를 사용하여 UI가 TodosNotifier와 상호 작용하게 합니다
final todosProvider = StateNotifierProvider<TodosNotifier, List<Todo>>((ref) {
  return TodosNotifier();
});
```

이제 StateNotifierProvider를 정의했으므로 UI에서 Todo List를 이용해 보겠습니다.

```dart
class TodoListView extends ConsumerWidget {
  const TodoListView({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
  
    // Todo List가 변경될 때 위젯을 다시 빌드합니다
    List<Todo> todos = ref.watch(todosProvider);

    return ListView(
      children: [
        for (final todo in todos)
          CheckboxListTile(
            value: todo.completed,
            // Todo를 누르면 상태가 "완료"로 변경됩니다
            onChanged: (value) =>
                ref.read(todosProvider.notifier).toggle(todo.id),
            title: Text(todo.description),
          ),
      ],
    );
  }
}
```

## Future Provider

FutureProvider는 비동기(async) 코드를 위한 프로바이더로, Provider와 같은 기능을 수행합니다.

다만, 간단한 사용 사례를 해결하기 위해 설계되었기 때문 사용자의 입력에 따른 변경 사항을 즉시 반영하지 않으므로 Riverpod에서는 AsyncNotifierProvider를 사용할 것을 권장하고 있으니 참고하시길 바랍니다.

FutureProvider는 주로 다음과 같은 경우에 사용됩니다.

- 비동기 작업(네트워크 요청 같은)을 수행하고 캐싱(caching)합니다
- 비동기 작업의 오류 및 로딩 상태를 처리합니다
- 여러 비동기 값을 다른 값으로 결합합니다

FutureProvider는 ref.watch와 함께 사용하면 일부 변수가 변경될 때 데이터를 자동으로 다시 가져오게 하여 최신 값을 유지할 수 있습니다.

### Usage example: reading a configuration file

FutureProvider는 JSON 파일을 읽어 Configuration 객체를 생성하는 데 있어서 편리한 솔루션을 제공합니다.

아래와 같이 Configuration을 생성하기 위해 Flutter의 에셋 시스템(Asset System)과 async/await 구문을 사용하여 provider 내에서 수행할 수 있습니다.

```dart
final configProvider = FutureProvider<Configuration>((ref) async {
  final content = json.decode(
    await rootBundle.loadString('assets/configurations.json'),
  ) as Map<String, Object?>;

  return Configuration.fromJson(content);
});
```

그리고 UI에서 다음과 같이 Configuration을 수신할 수 있습니다.

```dart
Widget build(BuildContext context, WidgetRef ref) {
  AsyncValue<Configuration> config = ref.watch(configProvider);

  return config.when(
    loading: () => const CircularProgressIndicator(),
    error: (err, stack) => Text('Error: $err'),
    data: (config) {
      return Text(config.host);
    },
  );
}
```

이렇게 하면 Future가 완료될 때 UI가 자동으로 다시 빌드되며, 여러 위젯이 Configuration을 사용하더라도 에셋(asset)은 한 번만 디코드(decode)됩니다. 

또한 위젯 내에서 FutureProvider를 수신하면 AsyncValue를 반환하므로 오류 및 로딩 상태를 처리할 수 있습니다.

## StreamProvider

StreamProvider는 Future 대신 Stream을 다루는 프로바이더이며, 기본적으로 FutureProvider와 유사합니다.

StreamProvider는 주로 다음과 같은 상황에 사용됩니다.

- Firebase 또는 웹소켓(Web Socket)과 같은 실시간 업데이트 데이터를 수신할 때
- 몇 초마다 다른 프로바이더를 다시 빌드할 필요가 있는 경우

스트림은 기본적으로 업데이트를 수신하는 방법을 제공하며, StreamBuilder 위젯이 스트림을 수신하는 데 유용하므로 StreamProvider를 사용하는 것이 큰 가치가 없다고 생각하실 수 있습니다.

하지만 StreamProvider를 StreamBuilder 대신 사용하면 다음과 같은 이점이 존재합니다.

- ref.watch를 사용하여 다른 프로바이더가 스트림을 수신할 수 있습니다
- AsyncValue를 통해 로딩 및 오류 사례를 적절하게 처리합니다
- 브로드캐스트 스트림(broadcast stream)과 일반 스트림을 구별할 필요가 없습니다
- 스트림에서 출력된 최신 값을 캐시하여 이벤트가 전달된 후 리스너가 추가되면 최신 이벤트에 바로 접근 가능합니다
- StreamProvider를 오버라이드하여 테스트 시 스트림을 쉽게 모조 값으로 대체할 수 있습니다

### Usage example: live chat using sockets

StreamProvider는 비디오 스트리밍, 날씨 방송 API, 실시간 채팅 등과 같이 비동기 데이터 스트림을 처리할 때 사용됩니다.

아래의 코드 예시를 확인해보겠습니다.

```dart
final chatProvider = StreamProvider<List<String>>((ref) async* {
  // 소켓을 이용하여 API에 연결합니다
  final socket = await Socket.connect('my-api', 4242);
  ref.onDispose(socket.close);

  var allMessages = const <String>[];
  await for (final message in socket.map(utf8.decode)) {
    // 새 메시지를 받을 때 마다, 리스트에 추가합니다
    allMessages = [...allMessages, message];
    yield allMessages;
  }
});
```

그리고 UI에서 다음과 같이 실시간 채팅 스트림을 수신할 수 있습니다.

```dart
Widget build(BuildContext context, WidgetRef ref) {
  final liveChats = ref.watch(chatProvider);
  // FutureProvider과 마찬가지로 AsyncValue.when을 통해 에러 및 로딩 상태를 처리할 수 있습니다
  return liveChats.when(
    loading: () => const CircularProgressIndicator(),
    error: (error, stackTrace) => Text(error.toString()),
    data: (messages) {
      return ListView.builder(
        // 메시지를 최신순(bttom to top)으로 나열합니다
        reverse: true,
        itemCount: messages.length,
        itemBuilder: (context, index) {
          final message = messages[index];
          return Text(message);
        },
      );
    },
  );
}
```

## StateProvider

StateProvider는 상태 수정 방법을 제공하는 프로바이더이며, 간단한 용도의 경우 Notifier 클래스 작성을 피하기 위해 고안된 NotifierProvider의 단순화 버전입니다.

StateProvider는 주로 사용자 인터페이스에 의해 간단한 변수를 수정하기 위해 존재합니다.

StateProvider는 다음과 같은 상황에 사용됩니다.

- 필터 타입과 같은 열거형 상수(enum)
- 텍스트 필드에 들어가는 일반적인 문자열(String)
- 체크박스에 대한 On/Off(boolean)
- 페이지 인덱스 또는 폼 필드에 들어가는 숫자(number)

StateProvider는 다음과 같은 상황에선 사용하지 않는것이 좋습니다.

- 상태에 유효성(validation) 검사 로직이 필요한 경우
- 상태가 복잡한 객체인 경우 (예: 사용자 정의 클래스, List/Map 등)
- 상태를 수정하는 로직이 단순한 count++보다 더 복잡한 경우

따라서 복잡한 로직이 필요한 경우 StateProvider 대신 NotifierProvider를 사용하고 Notifier 클래스를 만드는 것이 좋습니다.

처음에는 템플릿 코드가 갈어보일 수 있지만, Notifier 클래스를 사용하는 것은 프로젝트의 장기적인 관리 가능성에 있어 핵심적이며, 이를 통해 상태의 업무 로직을 한 곳에 집중시킬 수 있습니다.

### Usage example: Changing the filter type using a dropdown

StateProvider은 드롭다운(DropDown), 텍스트 필드(Text Field), 체크박스(Check Box)와 같은 간단한 구성 요소의 상태를 관리하기에 용이합니다.

예시로 StateProvider를 사용하여 제품 목록 정렬 방식을 변경할 수 있는 드롭다운 예제를 한 번 살펴보겠습니다.

```dart
class Product {
  Product({required this.name, required this.price});

  final String name;
  final double price;
}

final _products = [
  Product(name: 'iPhone', price: 999),
  Product(name: 'cookie', price: 2),
  Product(name: 'ps5', price: 500),
];

final productsProvider = Provider<List<Product>>((ref) {
  return _products;
});
```

보통 실제 애플리케이션에서 이러한 상품 목록은 일반적으로 네트워크 요청을 통해 FutureProvider로 얻으며, UI에서 다음과 같이 사용할 수 있습니다.

```dart
Widget build(BuildContext context, WidgetRef ref) {
  final products = ref.watch(productsProvider);
  return Scaffold(
    body: ListView.builder(
      itemCount: products.length,
      itemBuilder: (context, index) {
        final product = products[index];
        return ListTile(
          title: Text(product.name),
          subtitle: Text('${product.price} \$'),
        );
      },
    ),
  );
}
```

이제 가격이나 이름으로 제품을 필터링할 수 있는 드롭다운을 추가하여 구현해 보겠습니다.

```dart
// 필터 타입을 나타내는 열거형 상수는 다음과 같이 작성할 수 있습니다
enum ProductSortType {
  name,
  price,
}

Widget build(BuildContext context, WidgetRef ref) {
  final products = ref.watch(productsProvider);
  return Scaffold(
    appBar: AppBar(
      title: const Text('Products'),
      actions: [
        DropdownButton<ProductSortType>(
          value: ProductSortType.price,
          onChanged: (value) {},
          items: const [
            DropdownMenuItem(
              value: ProductSortType.name,
              child: Icon(Icons.sort_by_alpha),
            ),
            DropdownMenuItem(
              value: ProductSortType.price,
              child: Icon(Icons.sort),
            ),
          ],
        ),
      ],
    ),
    body: ListView.builder(
      // ... 
    ),
  );
}
```

드롭다운 위젯이 완성되었으므로, StateProvider를 만들고 드롭다운의 상태를 프로바이더와 연결할 차례입니다.

```dart
final productSortTypeProvider = StateProvider<ProductSortType>(
  // 기본적인 필터 타입은 이름순으로 정렬하는 것 입니다
  (ref) => ProductSortType.name,
);
```

다음으로 아래와 같이 드롭다운 위젯을 productSortTypeProvider와 연결하여 사용할 수 있습니다.

```dart
DropdownButton<ProductSortType>(
  // 정렬 타입이 변경되면 드롭다운을 다시 빌드하여 표시되는 아이콘을 업데이트합니다
  value: ref.watch(productSortTypeProvider),
  // 사용자가 드롭다운과 상호작용하면 프로바이더의 상태를 업데이트합니다
  onChanged: (value) =>
      ref.read(productSortTypeProvider.notifier).state = value!,
  items: [
    // ...
  ],
),
```

이후 필터 타입의 변경에 따라 제품을 정렬하기 위해 productsProvider를 작성해줍니다.

productsProvider에서 ref.watch를 사용하여 정렬 타입을 얻으면, 정렬 타입이 변경될 때마다 제품 목록을 다시 계산하게 됩니다.

```dart
final productsProvider = Provider<List<Product>>((ref) {
  final sortType = ref.watch(productSortTypeProvider);
  switch (sortType) {
    case ProductSortType.name:
      return _products.sorted((a, b) => a.name.compareTo(b.name));
    case ProductSortType.price:
      return _products.sorted((a, b) => a.price.compareTo(b.price));
  }
});
```

이제 UI는 정렬 유형이 변경될 때마다 자동으로 제품 목록을 다시 렌더링합니다.

### How to update the state based on the previous value without reading the provider twice

StateProvider의 상태를 이전 값에 기반하여 업데이트하고 싶은 경우 다음과 같이 작성할 수 있습니다.

```dart
final counterProvider = StateProvider<int>((ref) => 0);

class HomeView extends ConsumerWidget {
  const HomeView({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    return Scaffold(
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          // 이전 상태에서 상태를 업데이트하려면 프로바이더를 두 번 읽어야 합니다
          ref.read(counterProvider.notifier).state = ref.read(counterProvider.notifier).state + 1;
        },
      ),
    );
  }
}
```

위 코드에 특별히 잘못된 점은 없지만, 아래와 같이 update 함수를 사용해 현재 상태와 함께 콜백(callback)으로 새로운 상태를 기입하여 문법을 약간 개선할 수 있습니다.

```dart
final counterProvider = StateProvider<int>((ref) => 0);

class HomeView extends ConsumerWidget {
  const HomeView({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    return Scaffold(
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          ref.read(counterProvider.notifier).update((state) => state + 1);
        },
      ),
    );
  }
}
```

이러한 변경으로 문법을 조금 개선하면서도 동일한 효과를 얻을 수 있습니다.

## ChangeNotifierProvider

ChangeNotifierProvider (flutter_riverpod/hooks_riverpod 전용)는 Flutter 자체의 ChangeNotifier를 수신하고 전달하는 데 사용되는 프로바이더입니다. 

Riverpod에서는 가변 상태의 문제점으로 인해 ChangeNotifierProvider 사용을 권장하지 않지만, 주로 다음과 같은 상황에 사용됩니다.

- ChangeNotifierProvider 사용시 package:provider로 부터의 쉬운 전환을 위해
- 불변 상태가 선호되지만 가변 상태를 지원하기 위해

불변 상태 대신 가변 상태를 사용하는 것이 때때로 효율적일 수 있습니다. 그러나 유지하기 어렵고 여러 기능들이 깨질 수 있다는 단점이 있습니다. 

예를 들어, 가변 상태라면 provider.select를 사용하여 위젯의 다시 빌드를 최적화하려고 할 때 select는 값이 변경되지 않았다고 판단하고 작동하지 않을 수 있습니다.

그러므로 보통은 불변 데이터 구조를 사용하는 것이 효율적이며, 만약 ChangeNotifierProvider를 이용하는 경우 실제로 성능을 향상시키고 있는지 확인하기 위해 벤치마크를 수행하는 것이 좋습니다.

그럼 이제 ChangeNotifierProvider를 사용하여 구현한 Todo List를 예제를 한 번 확인해 보겠습니다.

```dart
class Todo {
  Todo({
    required this.id,
    required this.description,
    required this.completed,
  });

  String id;
  String description;
  bool completed;
}

class TodosNotifier extends ChangeNotifier {
  final todos = <Todo>[];

  void addTodo(Todo todo) {
    todos.add(todo);
    notifyListeners();
  }

  void removeTodo(String todoId) {
    todos.remove(todos.firstWhere((element) => element.id == todoId));
    notifyListeners();
  }

  void toggle(String todoId) {
    final todo = todos.firstWhere((todo) => todo.id == todoId);
    todo.completed = !todo.completed;
    notifyListeners();
  }
}

// ChangeNotifierProvider를 사용하여 UI가 ChangeNotifier와 상호 작용하게 합니다
final todosProvider = ChangeNotifierProvider<TodosNotifier>((ref) {
  return TodosNotifier();
});
```

이제 ChangeNotifierProvider를 정의했으므로 UI에서 Todo List를 이용해 보겠습니다.

```dart
class TodoListView extends ConsumerWidget {
  const TodoListView({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // Todo List가 변경되면 위젯이 다시 빌드됩니다
    List<Todo> todos = ref.watch(todosProvider).todos;

    return ListView(
      children: [
        for (final todo in todos)
          CheckboxListTile(
            value: todo.completed,
            // Todo를 누르면 상태가 "완료"로 변경됩니다
            onChanged: (value) =>
                ref.read(todosProvider.notifier).toggle(todo.id),
            title: Text(todo.description),
          ),
      ],
    );
  }
}
```

