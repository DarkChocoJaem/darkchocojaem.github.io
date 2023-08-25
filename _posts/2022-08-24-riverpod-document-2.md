---
title: "상태관리 패키지 Riverpod 공식문서 정리(2)"
categories:
  - Dart/Flutter
tags:
  - Riverpod

toc: true
toc_sticky : true

date : 2023-08-24
last_modified_at : 2023-08-24
---

# Concepts

## Providers

Riverpod에서 가장 중요한 부분은 프로바이더입니다.

프로바이더는 상태의 일부를 캡슐화한 객체로, 해당 상태에 대한 수신을 허용합니다.

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

## Reading a Provider

이번에는 프로바이더를 사용하는 방법을 알아보겠습니다.

### Obtaining a "ref" object

먼저, 프로바이더를 읽기 전에 ref 객체를 얻어야 합니다.

이 객체는 위젯이나 다른 프로바이더에서 프로바이더와 상호작용할 수 있게 해주며, 모든 프로바이더는 다음과 같이 ref를 매개변수로 받습니다.

```dart
final valueProvider = Provider((ref) {
  final repository = ref.watch(repositoryProvider);
  return repository.get();
});
```

이처럼 ref 매개변수는 프로바이더의 상태값을 안전하게 전달할 수 있습니다.

아래의 코드 예시를 통해 ref를 StateNotifier에 전달하는 경우를 한 번 살펴보겠습니다.

```dart
final counterProvider = StateNotifierProvider<Counter, int>((ref) {
  return Counter(ref);
});

class Counter extends StateNotifier<int> {
  Counter(this.ref) : super(0);

  final Ref ref;

  void increment() {
    // Counter는 프로바이더를 읽기 위해 ref를 사용합니다.
    final repository = ref.read(repositoryProvider);
    repository.post('...');
  }
}
```

이렇게 함으로써 Counter는 프로바이더를 읽기 위해 ref를 사용할 수 있습니다.

만약  위젯에서 프로바이더의 값을 읽어야 하는 경우, 위젯 자체에는 ref 매개변수가 없지만 Riverpod은 위젯에서 ref를 얻을 수 있는 다양한 솔루션을 제공합니다.

위젯 트리에서 ref를 얻는 가장 일반적인 방법은 StatelessWidget을 ConsumerWidget으로 대체하는 것입니다.

ConsumerWidget은 StatelessWidget과 사용법이 동일하지만, build 메서드에 추가 매개변수로 ref 객체가 존재합니다.

아래의 ConsumerWidget 예시 코드를 한 번 확인해 보겠습니다.

```dart
class HomeView extends ConsumerWidget {
  const HomeView({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // use ref to listen to a provider
    final counter = ref.watch(counterProvider);
    return Text('$counter');
  }
}
```

마찬가지로 StatefulWidget + State를 이용하는 경우엔, ConsumerStatefulWidget + ConsumerState를 통해 대체할 수 있습니다. 

단지 차이점은 상태(State)에 ref 객체의 유무입니다.

이번에는 ref를 build 메서드의 매개변수로 전달하는 대신, ConsumerState 객체의 속성을 통해 사용하는 예시를 살펴보겠습니다.

```dart
class HomeView extends ConsumerStatefulWidget {
  const HomeView({super.key});

  @override
  HomeViewState createState() => HomeViewState();
}

class HomeViewState extends ConsumerState<HomeView> {
  @override
  void initState() {
    super.initState();
    // "ref" can be used in all life-cycles of a StatefulWidget.
    ref.read(counterProvider);
  }

  @override
  Widget build(BuildContext context) {
    // We can also use "ref" to listen to a provider inside the build method
    final counter = ref.watch(counterProvider);
    return Text('$counter');
  }
}
```

### Using ref to interact with providers

이제 ref의 주요 사용법에 대해 알아보겠습니다.

ref는 주로 다음과 같은 경우에 사용됩니다.

- provider의 값을 얻고 변경 사항을 감지하여 해당 값이 변경될 때 위젯이나 provider를 다시 빌드하는 경우(ref.watch를 사용).
- provider에 대한 리스너를 추가하여, 해당 provider가 변경될 때 새로운 페이지로 이동하거나 모달을 표시하는 등의 작업을 실행하는 경우(ref.listen을 사용).
- 클릭과 같은 이벤트에서 provider의 값을 필요로 할 때, 값의 변경 사항을 감지하지 않는 일회성의 값으로 읽어야 하는 경우(ref.read를 사용).

참고로 공식 문서에서는 기능을 구현할 때 ref.read나 ref.listen 대신 ref.watch 사용을 권장합니다.

ref.watch를 사용하면 응용 프로그램이 반응형(reactive)이면서 선언적(declarative)이 되므로, 유지 보수하기 좋아집니다.

ref.watch는 위젯의 build 메서드 내부나 프로바이더의 본문(body) 내에서 사용되어 해당 위젯이나 프로바이더가 다른 프로바이더를 감지하도록 하는 데 사용됩니다.

예를 들어, 프로바이더는 ref.watch를 사용하여 여러 프로바이더를 결합하여 새로운 상태값을 만들 수 있습니다.

아래의 Todo List 필터링 예시를 살펴보겠습니다.

```dart
final filterTypeProvider = StateProvider<FilterType>((ref) => FilterType.none);
final todosProvider =
    StateNotifierProvider<TodoList, List<Todo>>((ref) => TodoList());

final filteredTodoListProvider = Provider((ref) {
  // 필터 타입과 Todo 목록을 얻습니다
  final FilterType filter = ref.watch(filterTypeProvider);
  final List<Todo> todos = ref.watch(todosProvider);

  switch (filter) {
    case FilterType.completed:
      // 완료된 Todo 목록을 반환합니다
      return todos.where((todo) => todo.isCompleted).toList();
    case FilterType.none:
      // 필터 타입이 없는 나머지 Todo를 반환합니다
      return todos;
  }
});
```

이렇게 하면 filteredTodoListProvider는 이제 필터링된 Todo List를 반환합니다.

필터나 Todo List 중 하나라도 변경 사항이 발생하면 필터링된 목록은 자동으로 업데이트 되며, 변경 사항이 발생하지 않았다면 필터링된 목록은 다시 계산되지 않습니다.

이제 위젯은 ref.watch를 사용하여 프로바이더의 내용을 표시하고 해당 내용이 변경될 때마다 UI를 업데이트할 수 있습니다.

```dart
final counterProvider = StateProvider((ref) => 0);

class HomeView extends ConsumerWidget {
  const HomeView({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // 프로바이더 값을 감지하기 위해 ref 사용
    final counter = ref.watch(counterProvider);

    return Text('$counter');
  }
}
```

해당 예시는 카운트 값을 저장하는 프로바이더를 감지하는 위젯입니다. 만약 카운트 값이 변경되면 위젯이 다시 빌드되고 UI가 새로운 값을 보여줄 것입니다.

단, 주의할 점은 watch가 비동기적으로 호출되어서는 안 됩니다.

예를 들어 Button의 onPressed 내부나 initState 및 기타 State 생명주기 내에서 사용되어서는 안 되며, 이러한 경우엔 ref.read를 사용하는 것이 적절합니다.

다음으로 ref.listen에 대해 알아보겠습니다.

ref.watch와 마찬가지로, ref.listen을 사용하여 provider를 관찰할 수 있습니다.

둘 사이의 주요 차이점은, watch를 사용할 땐 provider가 변경되면 위젯 혹은 provider를 다시 빌드하지만, ref.listen을 사용하면 커스텀 함수를 호출하게 됩니다.

이는 오류가 발생했을 때 스낵바를 표시하는 것과 같이 특정 변화가 일어났을 때 동작을 수행하는 데 있어서 유용할 수 있습니다.

ref.listen 메서드는 2개의 위치 매개변수가 필요하며, 첫 번째는 Provider이고 두 번째는 상태 변화 시 실행하려는 콜백 함수입니다. 콜백 함수가 호출될 때 이전 State 값과 새 State 값이 전달됩니다.

ref.listen 메서드는 다음과 같이 provider의 본문 내에서 사용할 수 있습니다.

```dart
final anotherProvider = Provider((ref) {
  ref.listen<int>(counterProvider, (int? previousCount, int newCount) {
    print('The counter changed $newCount');
  });
  // ...
});
```

혹은 위젯의 build 함수 내부에서 호출할 수 있습니다.

```dart
final counterProvider =
    StateNotifierProvider<Counter, int>(Counter.new);

class HomeView extends ConsumerWidget {
  const HomeView({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    ref.listen<int>(counterProvider, (int? previousCount, int newCount) {
      print('The counter changed $newCount');
    });

    return Container();
  }
}
```

단, 주의할 점은 watch와 마찬가지로 listen 메서드 역시 Button의 onPressed 내부나 생명주기와 같은 곳에서 비동기적으로 호출되어서는 안 됩니다.

이번에는 ref.read 메서드를 통해 프로바이더의 상태값을 얻는 방법에 대해 알아보겠습니다.

ref.read 메서드는 provider의 상태를 관찰하지 않고 얻는 방법이며, 일반적으로 사용자 상호작용에 의해 트리거된 함수 내에서 사용됩니다. 

예를 들어, 다음과 같이 사용자가 버튼을 클릭할 때 카운터를 증가시키기 위해 ref.read를 사용할 수 있습니다.

```dart
final counterProvider =
    StateNotifierProvider<Counter, int>(Counter.new);

class HomeView extends ConsumerWidget {
  const HomeView({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    return Scaffold(
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          ref.read(counterProvider.notifier).increment();
        },
      ),
    );
  }
}
```

ref.read는 반응성(reactive)이 없기 때문에 가능한 사용을 자제하고, watch 혹은 listen을 사용하는 것이 더 좋습니다.

때로는 다음과 같이 위젯의 성능을 최적화하기 위해 ref.read를 사용하는 것이 좋아보일 때가 있습니다.

```dart
final counterProvider = StateProvider((ref) => 0);

Widget build(BuildContext context, WidgetRef ref) {
  // 프로바이더의 watch와 state++로 인한 이중 rebuild를 방지하기 위해 read를 사용
  final counter = ref.read(counterProvider.notifier);
  return ElevatedButton(
    onPressed: () => counter.state++, //상태 업데이트에 따라 rebuild 호출
    child: const Text('button'),
  );
}
```

하지만 이것은 좋지 않은 아이디어이며, 추적하기 어려운 버그를 발생시킬 수 있습니다.

이런 방식으로 ref.read를 사용하는 것은 일반적으로 "ref.read를 사용할 경우 provider가 전달하는 값이 변경되지 않으므로 안전하다"는 생각과 연관되어 있습니다. 

이 생각의 문제점은, 지금 당장은 해당 provider가 실제로 값을 업데이트하지 않을 수는 있지만, 내일도 같을 것이라는 보장이 없다는 것입니다.

소프트웨어는 대체로 자주 많이 변하며, 과거에는 변경되지 않았던 값이 미래에 변경될 필요가 있을 가능성이 큽니다.

ref.read를 사용하면 값이 변경될 필요가 있을 때 코드베이스 전체를 통해 ref.read를 ref.watch로 바꾸어야 하는데, 이것은 오류가 발생하기 쉽고 일부 경우를 잊어버릴 가능성이 높습니다.

따라서 처음부터 ref.watch를 사용하면 리팩토링할 때 문제가 더 적게 발생합니다.

위젯의 rebuild 횟수를 줄이기 위해 ref.read를 사용하려고 했다면, ref.watch를 대신 사용하더라도 같은 효과(빌드 수 감소)를 누릴 수 있다는 점을 주의해 주세요.

프로바이더는 rebuild 횟수를 줄이면서 값을 얻는 다양한 방법을 제공하며, 이러한 방법들을 대신 사용할 수 있습니다.

```dart
final counterProvider = StateProvider((ref) => 0);

Widget build(BuildContext context, WidgetRef ref) {
  StateController<int> counter = ref.read(counterProvider.notifier);
  return ElevatedButton(
    onPressed: () => counter.state++,
    child: const Text('button'),
  );
}
```

위의 코드 대신 다음과 같이 사용할 수 있습니다

```dart
final counterProvider = StateProvider((ref) => 0);

Widget build(BuildContext context, WidgetRef ref) {
  StateController<int> counter = ref.watch(counterProvider.notifier);
  return ElevatedButton(
    onPressed: () => counter.state++,
    child: const Text('button'),
  );
}
```

위의 두 코드 모두 카운터가 증가해도 버튼이 rebuild 되지 않습니다

반면에, 두 번째 접근법은 카운터가 리셋(reset)되는 경우를 지원합니다.

예를 들어, 애플리케이션의 다른 부분에서 아래와 같은 코드를 호출한다고 가정해 보겠습니다.

```dart
ref.refresh(counterProvider);
```

해당 코드를 호출하면 StateController 객체를 다시 생성하게 됩니다.

그런데 만약 여기서 ref.read를 사용하면, 버튼은 해제되었고 더 이상 사용되어서는 안되는 이전의 StateController 인스턴스를 사용하게 됩니다.

하지만 ref.watch를 사용할 경우, 버튼을 올바르게 재구성하여 새로운 StateController를 사용합니다.

### Deciding what to read

프로바이더의 종류에 따라 여러 값을 받아서 읽어야 하는 경우가 있습니다.

아래의 StreamProvider을 예시로 들어보겠습니다.

```dart
final userProvider = StreamProvider<User>(...);
```

해당 userProvider을 사용할 때, 다음과 같은 사항들을 고려해야 합니다.

- userProvider 자체에 대한 리스닝을 하여 현재 상태값을 동기적으로 읽습니다.
  ```dart
  Widget build(BuildContext context, WidgetRef ref) {
    AsyncValue<User> user = ref.watch(userProvider);

    return user.when(
      loading: () => const CircularProgressIndicator(),
      error: (error, stack) => const Text('Oops'),
      data: (user) => Text(user.name),
    );
  }
  ```
- userProvider.stream에 대한 리스닝으로 Stream 객체를 얻습니다.
  ```dart
  Widget build(BuildContext context, WidgetRef ref) {
    Stream<User> user = ref.watch(userProvider.stream);
  }
  ```
- userProvider.future를 리스닝하여 가장 최근에 반환된 Future 값을 얻습니다.
  ```dart
  Widget build(BuildContext context, WidgetRef ref) {
    Future<User> user = ref.watch(userProvider.future);
  }
  ```

더 자세한 정보는 리버팟 공식 사이트를 통해 각 프로바이더에 대한 문서를 확인하시기 바랍니다.

### Using "select" to filter rebuilds

프로바이더를 읽는 것과 관련된 최종 기능은 ref.watch에서 위젯과 프로바이더가 rebuild 되는 횟수를 줄이거나, ref.listen이 함수를 실행하는 빈도를 조절할 수 있는 능력입니다.

기본적으로 프로바이더는 객체의 전체 상태를 관찰합니다. 하지만 경우에 따라 객체 전체가 아닌 일부 속성의 변경을 관찰할 수도 있습니다.

아래의 예시 코드를 한 번 확인해 보겠습니다.

```dart
abstract class User {
  String get name;
  int get age;
}
```

그리고 위젯에선 User 클래스의 name 속성만을 사용한다고 가정해 보겠습니다.

```dart
Widget build(BuildContext context, WidgetRef ref) {
  User user = ref.watch(userProvider);
  return Text(user.name);
}
```

만약 우리가 단순히 ref.watch를 사용한다면, 사용자의 나이가 변경될 때도 위젯을 다시 빌드하게 됩니다.

이때 select를 사용하면 Riverpod에게 User의 name 속성만 관찰하길 원한다고 명시적으로 알려줄 수 있습니다.

따라서 업데이트된 코드는 다음과 같습니다.

```dart
Widget build(BuildContext context, WidgetRef ref) {
  String name = ref.watch(userProvider.select((user) => user.name));
  return Text(name);
}
```

이처럼 select를 사용하여 원하는 속성을 반환하는 함수를 지정할 수 있습니다.

User가 변경될 때마다 Riverpod은 이 함수를 호출하여 name 속성의 이전 값과 새 값을 비교하고, 다른 경우에는 위젯을 다시 빌드하며 동일한 경우에는 위젯을 다시 빌드하지 않습니다.

추가로 select는 ref.listen과 함께 사용될 수 있습니다.

```dart
ref.listen<String>(
  userProvider.select((user) => user.name),
  (String? previousName, String newName) {
    print('The user name changed $newName');
  }
);
```

이렇게 하면 User의 name 속성이 변할 때에만 listener을 불러올 수 있습니다.

또한 Riverpod에서 select를 사용할 때, 꼭 객체의 속성을 반환해야 할 필요는 없습니다.

아래의 예시처럼 두 객체의 동등성을 비교할 수 있도록 '==' 연산자를 오버라이드한 값이라면 모두 가능합니다.

```dart
final label = ref.watch(userProvider.select((user) => 'Mr ${user.name}'));
```
