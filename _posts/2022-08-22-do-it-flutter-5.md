---
title: "Do it 깡샘의 플러터 & 다트 프로그래밍 - 18, 19, 20, 21장 내용 정리(5)"
categories:
  - Dart/Flutter
tags:
  - Do it 깡샘의 플러터 & 다트 프로그래밍

toc: true
toc_sticky : true

date : 2023-08-22
last_modified_at : 2023-08-22
---

> 해당 글은 기존 Velog로부터 새롭게 Github Blog로 이전되었습니다.

# 18장 - 상태 관리하기

## 18-1 위젯의 상태 관리하기

위젯의 상태를 관리하는 경우는 크게 3가지로 나눌 수 있습니다.

- 위젯 자체의 상태를 이용
- 상위 위젯의 상태를 이용
- 위젯 자체의 상태와 상위 위젯의 상태를 함께 이용

먼저 위젯 자체의 상태를 이용하는 경우엔 상태 데이터를 해당 위젯에 선언하고 관리하면 되는데, 보통은 그냥 StatefulWidget으로 만듭니다.

여러 위젯의 상태가 서로 의존성을 가진 경우, 상위 위젯을 생성하여 상태를 묶고 하위 위젯의 매개변수를 통해 상태 데이터를 전달하여 상태를 참조하면 됩니다. 

만약 이런 상위 위젯의 구조가 복잡해져서 바로 위의 위젯이 아니라 그 위 어디엔가에 있는 조상 위젯의 상태를 참조해야하는 경우, 생성자 매개변수를 이용할 경우 조상 위젯에서 하위 위젯에게 전달을 반복해야 하므로 비 효율적입니다.

따라서 이럴땐 하위 위젯에서 context의 findAncestorStateOfType() 함수를 이용해 조상 위젯의 상태를 참조하거나, 하위 위젯에 키를 지정하고 상위 위젯에서 GlobalKey()를 생성하여 currentState를 통해 하위 위젯의 상태를 참조하면 됩니다.

위젯의 상태관리는 매우 중요하므로 꼭 인지하고 있어야 합니다.

## 18-2 공용 상태 관리 위젯 만들기

여러 위젯이 이용하는 상태를 가지는 상위 위젯을 만들 땐 InheritedWidget을 이용할 수 있습니다. InheritedWidget은 화면을 만드는데 관여하지 않으며 단지 상태 데이터와 이를 관리하는 함수를 만드는 것이 목적이므로 build() 함수가 존재하지 않습니다.

InheritedWidget을 이용하려면 InheritedWidget을 상속받는 클래스를 만들고, 해당 클래스에 하위 위젯에서 이용할 상태 데이터와 관리 함수를 선언합니다.

위젯 계층 구조에서 InheritedWidget 상위에 부모 위젯이 있다면 부모 위젯의 상태 변경에 따라 InheritedWidget를 다시 빌드해야 하는 경우가 존재합니다. 하지만 InheritedWidget은 상태만 가지는 위젯이므로 다시 생성되더라도 상태 데이터가 이전과 같다면 하위 위젯을 다시 빌드할 필요가 없습니다.

따라서 InheritedWidget이 다시 생성될 때 자동으로 호출되는 updateShouldNotify() 함수의 반환값을 ture 또는 false로 설정하여 하위 위젯을 다시 빌드할지 결정할 수 있습니다.

또한 하위 위젯에서는 데이터나 함수를 이용하기 위해 InheritedWidget의 객체를 참조하여 사용하므로 InheritedWidget 위젯에 static으로 선언된 of() 함수를 만들어야 하는데, 이때 context의 dependOnInheritedWidgetOfExactType() 함수를 이용하여 InheritedWidget 인스턴스를 반환할 수 있습니다.

참고로 GetX, Provider, Bloc 상태관리 패키지도 내부적으로 InheritedWidget을 사용한다고 하니 알아두면 좋을 것 같습니다.

# 19장 - 프로바이더 패키지 이용하기

## 19-1 프로바이더의 기본 개념

프로바이더(Provider)는 InheritedWidget을 기반으로 조금 더 쉽게 상태를 관리 할 수 있도록 도와주는 패키지이며, 플러터에서 공식적으로 권장하고 있는 유용한 상태관리 패키지입니다.

프로바이더에서 제공하는 상태 관리 기법은 크게 2가지로 나뉩니다.

- Provider : 하위 위젯이 이용할 상태 데이터를 제공
- Consume : 하위 위젯에서 상태 데이터를 이용

먼저 하위 위젯이 이용할 상태를 제공하는 여러 종류의 Provider들이 있습니다.

- Provider : 기본 프로바이더(단순하게 하위 위젯에게 상태 데이터 제공)
- ChangeNotifierProvider : 상태 변경을 감지하는 프로바이더
- Multiprovider : 여러 가지 상태를 등록할 수 있는 프로바이더
- ProxyProvider : 다른 상태를 참조하여 새로운 상태를 생성하는 프로바이더
- StreamProvider : Stream 결과를 상태로 제공하는 프로바이더
- FutureProvider : Future 결과를 상태로 제공하는 프로바이더

또한 하위 위젯에서 프로바이더의 상태를 이용할 수 있도록 여러 방법을 제공합니다.

- Provider.of() : 데이터 타입(Type)을 통해 프로바이더의 상태 데이터 획득
- Consumer : 프로바이더의 상태를 사용하는 위젯을 명확하게 표시하며, 상태가 변경될 때마다 위젯을 다시 빌드하여 상태 변화에 즉각적으로 반응
- Selector : 사용할 상태와 해당 상태를 사용하는 위젯을 지정하며, 전체 상태 변경에 영향을 받지 않고 필요한 특정 부분의 상태가 변경될 때 위젯을 다시 빌드

프로바이더는 위젯이므로 데이터를 이용할 위젯의 상위에 Provider< T >.value() 생성자를 작성하고 value 속성으로 데이터를 명시하거나, Provider< T >() 생성자를 작성하고 create 속성으로 데이터를 명시하면, 이 데이터는 child에 작성한 위젯부터 그 하위 위젯들이 이용가능합니다.

```dart
//Ex1
Provider<int>.value(
  vlaue : 10,
  chuld : Widget(),
)

//Ex2
Provider<int>(
  create : (context) => 10,
  child : Widget(),
)
```

그리고 프로바이더의 상태 데이터를 하위 위젯에서 이용할 땐 Provider< T >.of() 함수에 타입을 명시하여 얻을 수 있습니다.

```dart
Widget build(BuildContext context){
  final data = Provider.of<int>(context);
}
```

## 19-2 다양한 프로바이더 알아보기

프로바이더를 통해 데이터를 하위 위젯에서 이용할 순 있지만 프로바이더에 등록된 상태 데이터는 값이 변경되더라도 하위 위젯이 자동으로 다시 빌드하지 않습니다. 따라서 ChangeNotifierProvider 위젯을 이용해야 하는데, 이때 ChangeNotifier을 구현한 상태 데이터를 작성하고 notifyListeners() 함수를 호출해 등록된 위젯을 다시 빌드해야 합니다.

```dart
class Counter with ChangeNotifier{
  int _count = 0;
  int get count => _count; //getter
}

ChangeNotifierProvider<Counter>.value(
  value : Counter(),
  child : Widget(),
)
```

만약 앱에서 이용할 상태 데이터가 고객 정보, 상품 정보, 주문정보 등과 같이 여러가지라면, 상태 데이터 관리를 위해 프로바이더를 계층구조로 작성하는 것 보다 여러 프로바이더를 한꺼번에 등록하여 사용하는 것이 가독성이 좋고 효율적입니다.

```dart
//bad
Provider<int>.value(
  value : 10,
  child : Provider<String>.value(
    value : "hello",
    child : Provider<int>.value(
      value : 20,
      child : Widget(),
    )
  )
)

//good
MultiProvider(
  provider : [
    Providr<int>.value(value : 10),
    Provider<String>.value(value : "hello"),
    Providr<int>.value(value : 20),
  ]
)

Widget build(BuildContext context){
  final data = Provider.of<int>(context);
  print(data); //20 출력
}
```

하지만 하위 위젯은 타입을 기준으로 프로바이더를 식별하고 이용하기 때문에 만약 같은 타입의 데이터가 여러 개 존재할 경우, 가장 마지막에 등록한 프로바이더를 이용하게 되므로 주의해야 합니다.

ProxyProvider은 어떤 상탯값을 참조해서 다른 상태값을 결정해야 하는 경우 참조 상태를 다른 상태에 전달해야 하는데, 이를 쉽게 구현할 수 있도록 도와줍니다.

ProxyProvider을 사용할 땐 ProxyProvider<A,B> 형태로 제네릭(Generic) 타입을 기본적으로 2개 선언합니다. 여기서 A는 전달받을 상태 타입이며, B는 A를 참조하여 만들 상태 타입입니다(즉, 마지막엔 새로 만들 상태의 제네릭 타입이 들어갑니다).

ProxyProvider을 이용하려면 update 속성에 함수를 등록해야 하는데, 이때 등록한 함수는 맨 뒤의 제네릭 타입에 맞는 객체를 반환해야 하며, 반환된 객체는 상태로 등록됩니다.

```dart
ProxyProvider<Counter, Sum>{
  update : (context, counter, sum){
    return Sum(counter);	
  }
}
```

또한 만약 여러 상태 데이터를 전달받아 새로운 상태를 만들 땐 ProxyProvider2~6 위젯을 사용할 수 있습니다(위젯명 뒤의 숫자는 전달받을 상태의 갯수를 의미).

```dart
ProxyProvider2<Counter, Sum, String>(
  update : (context, counter, sum, data) {
    return "${counter.count}, ${sum.sum}";
  }
)
```

본래 프로바이더에 등록한 객체는 싱글톤(Singleton)으로 운영되기 때문에 객체가 처음 생성되면 그 객체의 데이터를 변경할 뿐 객체를 다시 생성하진 않습니다. 그러나 ProxyProvider는 상태 객체와 다른 데이터에 의존하므로 데이터 변경이 발생할 때마다 새로운 상태 객체가 생성될 수 있습니다.

update() 콜백 함수는 종속성이 변경될 때마다 호출되는데, 이때 상태 객체도 새로 생성됩니다. 그러나 객체 생성과 관련된 비용과 상태 변경에 따른 성능 문제가 생길 수 있으므로 이전 상태 객체를 이용하면서 상태값만 바꾸는 것이 효율적일 수 있습니다.

이때는 update 속성에 등록한 함수의 마지막 매개변수인 이전 상태 객체에 데이터 값만 바꾸어 전달하면 됩니다.

```dart
ProxyProvider<Counter, Sum>(
  update : (context, counter, sum){
    if(sum != null){
      sum.sum = counter.count;
      return sum; //기존 객체 이용
    }else{
      return Sum(counter); //새로운 객체 생성
    }
  }
)
```

결론적으로 객체를 다시 생성할 것인지, 기존 객체를 이용하여 값을 변경할 것인지 상황에 따라 적절하게 판단하고 사용하면 됩니다.

FutureProvider은 Future로 발생하는 데이터를 상태로 등록하는 프로바이더입니다. create 속성에 지정한 함수에서 Future 타입을 반환하면 미래에 발생하는 데이터를 상태로 등록할 수 있으며, initialData 속성에 지정한 데이터가 초기 상태값으로 등록됩니다.

```dart
//상태 데이터는 처음에 hello이고, 1초 뒤에 world로 변경
FutureProvider<String>(
  create : (context) => 
    Future.delayed(Duration(seconds : 1), () => "world");
  initialData : "hello"
)
```

StreamProvider는 Stream으로 발생하는 데이터를 상태로 등록하는 프로바이더입니다. FutureProvider와 마찬가지로 create 속성에 Stream 타입을 반환하는 함수를 지정하고 initialData 값을 작성하면 스트림에 의해 지속적으로 생성되는 새로운 값이 자동으로 출력됩니다.

```dart
//1초 마다 1~5까지 증가하는 값을 반환하는 stream 함수 
Stream<int> streamFun() async* {
  for (int i = 1; i <=5; i++){
    await Future.delayed(Duration seconds : 1));
    yield i;
  }
}

// 0에서 시작해서 1초마다 5까지 증가
StreamProvider<int>(
  create : (context) => streamFun();
  initialData : 0
)
```

Provider은 객체 생성에 대한 싱글톤 패턴과 함께 위젯의 상태에 대해 의존성 주입(Dependency Injection) 패턴을 띄는 것 같습니다. 이 두가지 패턴은 단순하면서 유용하다고 하니, Provider을 좀 더 공부해서 디자인 패턴을 명확히 익히면 좋을 것 같습니다.

## 19-3 컨슈머와 셀렉터

Provider로 등록한 상태를 하위 위젯에서 사용할 땐 Provider.of() 함수를 이용합니다. 그런데 Provider.of() 함수를 이용하면 프로바이더의 하위 위젯들 중 상태를 이용하지 않는 위젯들도 부모 위젯이 상태를 이용하기 때문에 불필요하게 다시 빌드됩니다.

따라서 복잡하게 작성된 위젯에서 일부만 상태를 이용한다면 Consumer를 통해 특정한 타입만 새로 빌드하도록 한정하여 빌드 비용을 최소화 시킬 수 있습니다. 

먼저 Provider 하위에 Consumer을 추가한 후, builder 속성엔 상태를 이용할 위젯을 반환하는 함수를 지정하고, child 속성엔 상태를 이용하지 않을 위젯을 지정하면 됩니다.

이렇게 하면 상태값이 변경될 때 위젯 전체를 다시 빌드하지 않고 상태를 이용하는 부분만 빌드할 수 있습니다.

```dart

class MyDataModel with ChangeNotifier{
  int data1 = 0;
}

Consumer<MyDataModel>(
  builder : (context, model, child){
    return SubWidget1(model, child); //data1이 변하면 다시 빌드
  },
  child : SubWidget1_2(), // 다시 빌드되지 않음
)
```

추가로 builder 함수의 두 번째 매개변수를 통해 이용할 상태를 전달하므로 Provider.of()로 상태 데이터를 가져오지 않아도 됩니다.

 또한 Provider과 마찬가지로 여러 상태를 이용하는 경우 Consumer2~6 위젯들을 사용할 수 있습니다.

Selector은 Consumer과 유사하지만 좀 더 세부적으로 상태의 내부 데이터 중 특정한 타입의 변경에 따라 다시 빌드할 수 있습니다. Selector은 제네릭 타입을 두 개 지정하는데, 앞의 타입은 이용할 상태 객체의 타입이며 뒤의 타입은 해당 객체에서 이용할 데이터의 타입입니다.

Selector을 이용할 땐 builder 속성과 selector 속성을 사용합니다. selector 속성엔 상태 객체의 특정한 데이터를 지정하고, builder 속성엔 해당 데이터를 전달합니다. 이렇게 하면 selector에 지정된 데이터가 변경될 때만 builder의 위젯을 다시 빌드합니다.

``` dart
class MyDataModel with ChangeNotifier{
  int data1 = 0;
}

Selector<MyDataModel, int>(
  selector : (context, model) => model.data1, //상태 객체 내부의 데이터 지정
  builder : (context, data, child){
    return Text("$data"); //model의 data1이 변하면 자동으로 다시 빌드 
  }
)
```

플러터의 상태 관리 패키지들을 공부하다 보니 UI 빌드 비용을 줄이는데 정말 많은 노력들을 하고 있음을 깨달을 수 있었습니다.

# 20장 - Bloc로 상태 관리하기

## 20-1 Bloc 패턴

BLoC(Business logic component)은 관심 분리(Separation of Concern, SoC)를 통해 UI와 비지니스 로직을 분리하는 디자인 패턴을 적용한 상태 관리 패키지입니다.

위젯 코드를 작성할 땐 최대한 화면에 관련된 정보만 담고, 위젯의 초기화 혹은 이벤트(Event) 발생에 따른 업무 로직의 처리 부분은 BLoC을 이용해 따로 클래스에 작성하여 개발과 유지보수, 테스트 등을 쉽게 하는 것이 목적입니다.

## 20-2 Bloc 구성 요소

이벤트는 블록(BLoC)의 입력 요소입니다. 예를 들어 버튼의 경우 터치가 발생했을 때 이를 감지하고 적절한 기능을 수행해야 하는데, 여기서 터치의 발생은 이벤트에 해당하고 적절한 기능은 업무 로직에 해당합니다.

만약 위젯 코드를 짤 때 화면에 대한 정보 뿐만 아니라 업무 로직까지 함께 작성한다면, 코드가 길어지고 복잡해지므로 위젯은 위젯의 초기화나 사용자 이벤트가 발생할 때 블록에 이벤트만 주입하는 역할만 하고 업무 로직에 대한 부분은 관여하지 않는 것이 좋습니다.

블록이 감지할 이벤트를 작성할 때 특별한 규칙이 존재하진 않습니다. 따라서 enum 타입으로 작성하거나 개발자 클래스를 작성하여 주입할 수 있습니다.

```dart
// enum Counter Event{increment, decrement} 혹은 아래와 같이도 가능

abstract class CounterEvent{}
class IncrementEvent extends CounterEvent{}
class DecrementEvent extends CounterEvent{}
```

상태(state)는 블록의 출력 요소입니다. 이벤트에 의해 실행된 로직의 처리 결과로, 앱 내의 여러 위젯에서 사용하는 상태 데이터를 의미합니다.

트랜지션(transitions)은 Bloc에 이벤트가 발생하고 업무 로직이 실행되어 상태 데이터가 발생하거나 변경될 때의 흐름에 관한 정보를 의미합니다. 트랜지션 정보는 이벤트에 따라 Bloc에서 자동으로 발생되며, 로그(Log)와 유사하게 이벤트에 따른 상태값 변경을 추적, 관리하는 용도로 사용할 수 있습니다.

핵심인 BLoC을 상속받는 클래스는 다음과 같은 역할을 합니다.

- 위젯에서 발생하는 이벤트를 수신
- 이벤트에 따른 적절한 업무 로직 실행
- 로직 실행 결과를 앱의 상태로 유지 및 위젯에 방출(emit)

블록 클래스를 작성할 땐 제네릭으로 BloC에 발생할 이벤트의 타입과 함께 유지할 상태 타입을 명시해야 하며, 상위 클래스의 생성자 매개변수로 초기값을 지정해야 합니다.

또한 on() 함수를 작성하여 이벤트를 등록하면 해당 타입의 이벤트가 발생할 때 on() 함수의 매개변수로 지정된 함수가 자동으로 호출됩니다.

이 함수의 첫 번째 매개변수는 이벤트의 정보이고, 두 번째 매개변수는 상태를 내보낼 때 이용할 함수입니다.

```dart

abstract class CounterEvent{}
class IncrementEvent extends CounterEvent{}
class DecrementEvent extends CounterEvent{}

class blocCounter extends Bloc<CounterEvent, int>{
  BlocCounter() : super(0){
    //초기화
    on<IncrementEvent>((event, emit){
      emit(state + 1);
    });
    
    on<DecrementEvent>((event, emit){
      emit(state - 1);
    });
  }
}
```

BLoC 클래스의 핵심은 on() 함수를 통해 이벤트를 등록하고 이벤트가 발생할 때 로직을 처리하여 상태 데이터를 발생시키는 것입니다. 그리고 이를 돕기 위해 onError(), onTransition(), onEvent() 함수를 지원하며, 이 함수들을 재정의하여 사용할 수 있습니다.

onEvent() 함수는 이벤트가 발생할 때마다 자동으로 호출되며, on() 함수보다 먼저 호출됩니다. 따라서 해당 블록에 내에 존재하는 모든 이벤트의 발생 전에 공통으로 처리할 로직 혹은 1차적으로 처리할 로직이 존재할 경우 이용할 수 있습니다.

onTransition() 함수는 이벤트의 발생에 따른 상태값의 변화로 생성된 트랜지션 정보를 활용할 수 있는 함수입니다. 이 함수는 on() 함수가 호출된 후 자동으로 호출되며, 이벤트를 로깅(Logging) 할 때 트랜지션에 대한 공통적인 처리 혹은 변화를 주기 위해 이용할 수 있습니다.

onError() 함수는 이벤트 발생으로 특정 로직을 처리하다가 오류가 발생했을 때 자동으로 호출되는 함수입니다. try catch문과 유사하게 오류를 기록하거나 특별한 처리를 위해 이용할 수 있습니다.

BLoC 클래스를 작성했으면 이젠 위젯에서 이용할 수 있게 등록해야 하는데, 이 때는 BlocProvider 위젯을 사용합니다. BlocProvider 위젯을 작성할 땐 사용할 BLoC을 제네릭으로 타입으로 선언해야 하고, create 속성의 함수에서 BLoC 객체를 반환하여 child에 명시한 위젯부터 그 하위 위젯들이 사용할 수 있게 해줍니다.

BLoC을 등록했으면 하위 위젯에선 BloC 객체를 얻어 이벤트를 주입하거나 상탯값을 이용해야 합니다. 이 때 of() 함수를 통해 객체를 불러오거나 다른 방법을 이용할 수도 있습니다.

하위 위젯에서 BLoC 객체를 얻었다면 이벤트를 주입할 땐 add() 메소드를 통해 진행하고, 객체의 상태를 이용할 땐 state 멤버를 이용할 수 있습니다.

```dart

//메인 build 함수 생략
BlocProvider<BlocCounter>(
  create : (context) => BlocCounter(),
  child : MyWidget(),
)

class MyWidget extends StatelessWidget{

  @override
  Widget build(BuildContext context){
  
    final counterBloc = BlocProvider.of<BlocCounter>(context);
      
    return ElevatedButton(
      child : Text("${counterBloc.state}"),
      onPressed : (){
        counterBloc.add(IncrementEvent());
      }
    );
  }
}
```

## 20-3 다양한 Bloc 이용 기법

Bloc 옵저버는 BlocObserver를 상속받아 작성하는 클래스입니다. Bloc 클래스처럼 onEvent(), onTransisiton, onError() 함수를 멤버로 가지며, 추가로 Bloc에서 상태값이 변경될 때 호출되는 onChange() 함수도 가집니다.

옵저버는 BloC 내부 함수보다 먼저 작동하기 때문에  인터셉터(Interceptor)처럼 모든 이벤트, 에러, 트렌지션을 가로채고 데이터를 가공 혹은 추적하기 위해 사용할 수 있습니다.

복잡한 앱을 만들 때 앱의 전역적인 범위에서 발생하는 이벤트와 상태 데이터를 하나의 Bloc에 담아 작성하는 것은 비효율적일 수 있으므로, 여러 개의 Bloc을 작성하고 공통된 코드를 묶어 옵저버에 담아서 유지보수 및 개발의 효율성을 증가시킬 수 있습니다.

```dart
class MyObserver extends BlocObserver{
  @override
  void onEvent(Bloc bloc, Object? event){
    super.onEvent(bloc, event);
  }
  
  @override
  void onTransition(Bloc bloc, Transition transition){
    super.onTransition(bloc, transition);
  }
  
  @override
  void onError(Bloc bloc, Object error, StackTrace stackTrace){
    super.onError(bloc, error, stackTrace);
  }
  
  @override
  void onChange(Bloc bloc, Change change){
    super.onChange(bloc, change);
  }
}
```

옵저버를 작성했으면 Bloc에 등록해야 하는데, 등록은 한번만 하면 되므로 보통 앱의 진입점인 main() 함수에서 진행합니다.

```dart
void main(){
  BlocOverrides.runZoned((){
      runApp(MyApp());
    },
    blocObserver : MyObserver()
  );
}
```

앞에서 설명한 것 처럼 BLoC은 BlocProvider을 통해 위젯으로 등록하여 사용할 수 있습니다. 만약 위젯에서 이용하는 BLoC이 여러 개일 경우 이전의 내용의 Provider 패키지와 마찬가지 방법으로 MultiBlocProvider을 사용하여 편리하게 등록할 수 있습니다.

```dart
MultiBlocProvider(
  providers : [
    BlocProvider<BlocCounter>(create : (context) => BlocCounter()),
    BlocProvider<UserBloc>(create : (context) => UserBloc()),
  ],
  child : MyWidget()
)
```

Bloc 객체를 얻는 기법은 기본적으로 of() 함수를 이용하는 것입니다. 그러나 BlocBuilder 위젯을 통해 Bloc의 상태 데이터를 좀 더 쉽게 얻을 수 있는 방법을 제공합니다. 

BlocBuilder은 이용할 블록과 상태 타입을 제네릭으로 지정하고, builder 속성에 화면에 담길 위젯 함수를 작성하면 됩니다. 이때 builder의 함수의 두 번째 매개변수인 상태값을 이용할 수 있으며, 해당 상태값이 변경되면 builder은 다시 호출됩니다.

그런데 만약 블록의 상태값이 변하더라도 builder에 명시한 위젯이 다시 빌드되지 않는다면 buildWhen 속성을 이용하여 빌드를 강제할 수 있습니다.

buildWhen 함수는 첫 번째 매개변수로 이전 상태값을 가지고, 두번째 매개변수로 현재 변경된 상태값을 가집니다. 이 함수가 true를 반환하면 위젯을 다시 빌드하고, false를 반환할 경우 builder에 작성된 함수가 호출되지 않습니다.

즉, 상태값은 변경되지만 화면은 그대로가 되므로 화면 갱신이 필요 없을 때 사용할 수 있습니다.

BlocBuilder은 일반적으로 상위 위젯인 BlocProvider에서 제공하는 블록을 사용하지만, 필요할 경우 bloc 속성을 통해 해당 위젯에서 내부적으로 이용할 블록을 지정할 수 있습니다.

```dart
BlocBuilder<BlocCounter, int>(
  bloc : BlocCounter(), //내부적으로 이용할 블록(생략, 변경 가능)
  buildWhen : (previous, current) => true, //상태값 변경시 무조건 다시 빌드
  builder : (context, count){
    return Text($count);
  }
)
```

BlocListener은 BlocBuilder와 유사하지만 builder 속성이 없습니다. 따라서 상태값의 변경에 따라 특정한 화면을 구성할 수 없고, 단지 변경된 상태값을 이용해 스낵바(SnackBar) 띄우기 혹은 다이얼로그(Dialog) 띄우기 등의 로직을 처리하기 위해 사용됩니다.

이 위젯의 listenWhen 속성은 BlocBuilder의 buildWhen 속성과 마찬가지로 상태값이 변경될 때 자동으로 호출되며, listener에 지정한 함수를 호출해야 하는지를 제어할 수 있습니다. 

또한 Provider과 마찬가지로 MultiBlocListener도 존재합니다.

```dart
MultiBlocListener(
  listeners : [
    BlocListener<BlocCounter, int>(
      listenWhen : (previous, current) => true,
      listener : (context, state){
        //Logic 처리
      }
    ),
    BlocListener<UserBloc, User?>(
      listener : (context, state){
        //Logic 처리
      }
    ),
  ]
)
```

BlocConsumer은 BlocBuilder과 BlocListener의 혼용체입니다.

BlocCounsumer을 이용하면 상태값의 변경에 따라 listener 속성을 통한 로직 구현과 builder을 통한 화면 구성요소 작성을 함께 작성할 수 있고, listenWhen과 buildWhen 함수를 통해 이를 제어할 수 있어 복잡하게 계층 구조로 작성하는 것을 방지해줍니다.

```dart
BlocConsumer<BlocCounter, int>(
  listener : (context, state){
    //생략
  },
  builder : (context, state){
    //생략
  }
  listenWhen : (previous, current) => true,
  buildWhen : (previous, current) => true,
)
```

Repository Provider은 저장소(Repository)를 등록하고 하위 위젯이 이를 이용할 수 있도록 해줍니다.

여기서 말하는 저장소는 화면에서 발생하거나 필요로 하는 데이터가 저장된 곳을 의미하며, 서버 혹은 로컬 데이터베이스를 지칭합니다. 

저장소 프로바이더 작성시 특별한 규칙은 없으니 자유롭게 작성해도 됩니다.

```dart
class UserRepository{
  void login(){};
}

//메인 build 함수
RepositoryProvider(
  create : (context) => UserRepository(),
    child : BlocProvider<BlocCounter>(
      create : (context) => BlocCounter(),
      child : MyWidget(),
  )
)
```

이렇게 등록한 저장소는 Provider과 마찬가지로 of() 함수를 통해 객체에 접근하여 이용 할 수 있습니다.

## 20-4 Bloc 큐빗

BLoC 패키지를 이용할 때 블록을 상속받는 클래스를 작성하고 이벤트를 주입해 업무로직을 수행할 수 있지만, 복잡한 상태관리가 필요하지 않는 로직이고 비교적 간단한 동기 작업을 처리해야 한다면 큐빗(Cubit)을 이용하는 것이 효율적일 수 있다고 합니다.

큐빗은 블록과 구조가 거의 유사하지만 이벤트를 주입하는 것이 아니라 함수를 호출하여 정의된 로직을 실행하고 상태 데이터를 유지하는 방법입니다.

큐빗은 Cubit 클래스를 상속받아 작성하며, 제네릭으로 상태 데이터 타입을 명시하고 블록과 마찬가지로 상위 클래스의 생성자 매개변수에 초기값을 지정해야 합니다.

하지만 블록과 다르게 이벤트를 주입하지 않으므로 이벤트 클래스 및 onEvent() 함수와 on() 함수를 통한 이벤트 핸들러를 등록할 필요가 없고 단지 큐빗 클래스에 사용할 함수만 선언하면 됩니다. 

또한 큐빗 클래스를 작성하고 위젯에서 선언된 함수를 직접 호출하여 적절한 로직을 처리한 후 그 결과 데이터를 emit() 함수로 내보내면 onChange() 함수가 자동으로 호출되며, onChange() 함수의 매개변수인 Change 객체로부터 이전 상태값과 현재 상태값을 불러올 수 있습니다.

```dart
class CounterCubit extends Cubit<int>{
  Counter() : super(0);
    
  void increment() => emit(state + 1);
  void decrement() => emit(state -1);
  
  @override
  void onChange(Change<int> change){
    super.onChange(change);
    print(change.toString()); 
    //increment Ex : Change{ currentState : 0, nextState : 1} 
  }
}
```

큐빗을 이용할때는 블록과 마찬가지로 BlocProvider을 통해 create 속성에 큐빗 클래스를 지정해주면 됩니다.

블록을 통해 관심사를 분리할 수 있다는 것은 정말로 매력적인 것 같습니다.

# 21장 - GetX로 상태 관리하기

## 21-1 단순 상태 관리하기

GetX 패키지를 이용할 땐 앱의 루트 위젯으로 MaterialApp 혹은 CupertinoApp이 대신 GetmaterialApp을 이용하여 라우팅(Routing) 및 스낵바(SnackBar)와 국제화(Localization) 기능을 쉽게 구현할 수 있습니다.

GetX의 상태관리는 상태값이 변경될 때 함수 호출로 변경 사항을 직접 적용해야 하는 단순 상태 관리와 자동으로 적용 해주는 반응형 상태 관리로 나뉩니다.

우선 단순 상태 관리를 구현할 땐 GetxController을 상속 받는 클래스를 작성하고 이용할 상태 변수들을과 함께 상태값을 내보내거나 변경하는 함수를 작성합니다. 그리고 이 상탯값의 변경사항을 적용할 때 update() 함수를 호출하면 됩니다. 

또한 GetxController에는 onInit()와 onClose() 같은 생명주기 함수를 선언하여 초기화를 하거나 Controller의 메모리 소멸 시점에 호출할 로직을 작성할 수 있습니다. 

```dart
class CounterController extends GetxController{
  int count = 0;
    
  @override
  onInit() {
    count = 1;
    super.onInit();
  }
  
  void increment(){
    count ++;
    update(); //view 새로고침
  }
  
  @override
  onClose() {
    //주로 Event Listener 제거 코드 작성
    super.onClose();
  }
}
```

GetXController을 이용할 땐 먼저 GetXController를 위젯에 등록해주어야 하는데, 이땐 주로 Get.put() 함수를 이용하거나 GetBuilder 위젯을 통해 등록하는 방법을 사용합니다.

그리고 하위 위젯에서 이용할 땐 Get.find() 함수 혹은 GetBuilder 위젯의 builder 속성 매개변수인 controller을 통해 접근합니다.

```dart
class MyWidget extends StatelessWidget{

  @override 
  Widget build(BuildContext context){
  
    Get.put(CounterController());
      
    return ElevatedButton(
      child : Text("Test"),
      onPressed : (){
        final CounterController controller = Get.find();
        controller.increment();
      }
    );
  }
}

//OR

class MyWidget extends StatelessWidget{

  @override 
  Widget build(BuildContext context){

    return GetBuilder(
      init : CounterController(),
      builder : (controller){
        return ElevatedButton(
          child : Text("Test");
          onPressed : (){
            controller.increment();
          }
        )
      }
    );
  }
}
```

## 21-2 반응형 상태 관리하기

반응형 상태 관리는 GetXController가 상태값 변경을 자동으로 감지하여 상태 변화가 생길 경우 위젯을 다시 빌드해줍니다. 즉, update() 함수를 이용하지 않아도 됩니다.

반응형 상태 관리를 사용하려면 GetXController의 변수를 Rx 객체에 제네릭 타입으로 선언하여 Rx< T >() 형식으로 바꾸어주거나 변수 선언 값 뒤에 .obs를 추가하여 선언한 후 value 속성으로 접근하여 사용할 수 있습니다.

```dart
class CounterController extends GetxController {
  final Rx<int> count = 0.obs;
  //var Rx<int> count = RxInt(0); 도 가능
}
```

참고로 위젯에서의 사용법은 단순 상태 관리와 동일하게 Get.find() 함수 혹은 GetxBuilder을 이용하면 됩니다.

만약 GetxController의 상태가 변경될 때 이를 감지해 특정 함수를 호출하고 싶다면 Worker 기능을 이용할 수 있습니다. Worker 기능은 GetxController의 생성자나 onInit() 함수에 등록하여 사용할 수 있으며, 다음과 같은 종류들이 있습니다.

- ever() : Rx 값 변경시 반복해서 호출
- once() : Rx 값이 최초에 변경될 때 호출
- debounce() : Rx값이 마지막으로 변한 후 설정한 특정 시간이 지났을 때 호출
- interval() Rx값 변경 횟수와 무관하게 설정한 특정 시간 간격으로 호출

```dart
//class 선언부 생략
@override
onInit(){
  super.onInit();
  ever(count, (value) => print(value)); //count 값이 변할 때 마다 출력
  once(count, (value) => print(value)); //count 값이 최초 한번 변할 때 출력
  
  //지속적으로 count 값이 변할 경우 마지막으로 변한 후 1초 뒤에 출력
  debounce(count, (value) => print(value), time: Duration(seconds : 1));
  
  // count 값 변경 횟수와 무관하게 1초마다 지속적으로 출력
  interval(count, (value) => print(value), time: Duration(seconds : 1));
}
```

GetX는 이미 제가 충분히 다루어 보았기 때문에 내용을 생략하려 했지만, 혹시나 나중에 잊을 일이 있을까봐 중요한 내용만 간략하게 적어보았습니다. 

# P.S.
이렇게 약 1달간 "Do it 깡샘의 플러터 & 다트 프로그래밍" 책의 내용을 읽고 정리해 보았습니다.

어떤 부분은 내용이 빠져있고 어떤 부분은 책에 없는 내용도 들어있는데, 그건 제가 더 조사하여 추가하거나 이미 알고 있거나 혹은 다룰 가능성이 희박한 내용들은 생략하고 작성했기 때문입니다.

책 내용은 전반적으로 마음에 들었지만 한가지 아쉬웠던 점은 나는 코드를 체계적으로 작성하고 디렉토리 구조를 어떤식으로 만들어야 하는지도 배우고 싶었는데 예제가 대부분 main.dart 단일 파일로 되어 있었다는 점입니다.

해당 포스팅은 제가 주관적으로 쓴 글이오니, 만약 오타 혹은 내용에 문제가 있을 경우 언제든지 지적해주시길 바랍니다. 