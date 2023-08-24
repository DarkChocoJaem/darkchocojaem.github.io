---
title: "Do it 깡샘의 플러터 & 다트 프로그래밍 - 7, 8장 내용 정리(2)"
categories:
  - Do it 깡샘의 플러터 & 다트 프로그래밍
tags:
  - [Dart, Flutter]

toc: true
toc_sticky : true

date : 2023-08-22
last_modified_at : 2023-08-22
---

> 해당 글은 기존 Velog로부터 새롭게 Github Blog로 이전되었습니다.

# 7장 - 상속과 추상클래스

## 07-1 상속 알아보기

Dart에서 상속은 extend 키워드를 통해 진행합니다. 상속받은 자식 클래스는 부모 클래스에 선언된 멤버를 그대로 사용하거나, 원할 경우 재정의(Override)할 수 있습니다. 또한 부모 클래스에서 선언된 멤버를 이용할고 싶을 경우 super 키워드로 접근 가능합니다. 

단, super 키워드는 부모 생성자가 호출된 이후 사용 가능하므로 생성자 호출 시점엔 사용할 수 있지만 필드 초기화 시점엔 이용할 수 없습니다.
```dart 
class SuperClass {
  int data1 = 10;
  int data2 = 20;
  
  // SuperClass() {} 기본 생성자 생략
  
  void some() {
    print("super");
  }
}

class SubClass extends SuperClass{
  @override //@override 어노테이션은 생략 가능
  int data1 = 0;
  
  //int data3 = super.data2; 필드 초기화 시점에 이용 불가 Error
  
  //SubClass() : super() {} 기본 생성자 생략
  
  @override
  void some() {
  	super.some();
    print("sub");
  }
}

void main() {
  SubClass sub = SubClass();
  sub.some(); // super 출력 후 sub 출력
  
  print(sub.data1); //0
  print(sub.data2); //20
}
```

만약 부모 클래스가 매개 변수를 가지는 경우 아래와 같이 작성할 수 있습니다.
```dart
class SuperClass {
  final int data1;
  
  SuperClass(this.data1); //부모 클래스 생성자
  
  void some() {
    print("super");
  }
}

class SubClass extends SuperClass{
  
  final int data2;
  
  SubClass.first(int data1, this.data2) : super(data1); //첫째 생성자
  
  SubClass.second(super.data1, this.data2); //둘째 생성자
  
  //셋째 생성자
  SubClass.third(int data1, int data2) : this.data2 = data2, super(data1);
  
  SubClass.fourth(int data1, this.data2) : super(data1) {
    print(data1);
    print(data2);
  } //넷째 생성자

  
  @override
  void some() {
  	super.some();
    print("sub");
  }
}

void main() {
	
  //var error = SubClass(); 클래스에 기본 생성자가 없으므로 Error

  SubClass sub = SubClass.first(10,20);
  print(sub.data1); //10
  print(sub.data2); //20
  
  SubClass sub2 = SubClass.second(10,20);
  print(sub2.data1); //10
  print(sub2.data2); //20
  
  SubClass sub3 = SubClass.third(10,20);
  print(sub3.data1); //10
  print(sub3.data2); //20
  
  SubClass.fourth(10,20); //위와 동일 결과 출력

}
```

코드를 작성하다 보면 상속을 이용하는 경우는 많지만 생성자에 대한 이해 부족으로 헤매는 경우도 많으므로, 생성자를 정확하게 이해해고 유연하게 작성할 수 있어야 코드도 잘 작성할 수 있습니다.

## 07-2 추상 클래스와 인터페이스

추상 클래스(Abstract Class)는 추상 함수를 제공하여 상속받는 클래스에서 재정의해 사용하는 방법입니다(쉽게 표현하자면 함수의 껍데기만 정의하고 내용물은 자식 클래스에서 결정하는 방법입니다). 

추상 클래스는 객체를 생성할 수 없으며, abstract 키워드를 통해 선언합니다.
```dart
abstract class User{
  void some(); //추상 메서드
  void hello(){
    print("hello"); //구현된 메서드
  }
}

class Admin extends User{
  @override
  void some() {
    print("I am admin");
  }
}

main(){
  // final user = User(); 객체 생성 불가 Error
    
  final admin = Admin();
  admin.some(); // "I am admin" 출력
  admin.hello(); // "hello" 출력
}
```
추상 클래스는 구현 클래스들 간의 공통 코드를 추출할 때 유용합니다. 또한 객체를 생성하지 않고서도 상속 구조를 만들 수 있으므로, 객체 생성에 관계없이 클래스 간의 협력이 필요한 경우에 효율적입니다.

인터페이스(Interface)는 부모의 특징을 이용해 새로운 특징을 만들어 사용하는 방법입니다. 대부분의 객체지향 언어들은 interface라는 예약어를 지원하지만, Dart의 경우 클래스 자체가 암시적 인터페이스를 의미하므로 따로 예약어를 붙이지 않고 자식 클래스에서 implements 키워드를 통해 인터페이스를 구현합니다.

인터페이스를 구현할 때는 모든 부모의 멤버를 자식 클래스에서 재정의해야 합니다. 또한 Java와 마찬가지로 상속을 구현할 경우 하나의 부모만을 가져야하지만, 인터페이스를 구현할 경우 여러 개의 부모를 가질 수 있습니다. 

```dart
//부모 클래스
class User{
  int id;
  String name;
    
  User(this.id, this.name);
  
  void hello(){
    print("hello");
  }
}

//상속 구현
class Admin extends User{

  Admin(super.id, super.name);
    
  void permission() {
    print("admin");
  }
}

//인터페이스 구현
class Manager implements User, Admin{
  @override
  int id = 0;
   	
  @override
  String name = "Jaem";
  
  //Manager(){}
  
  void hello(){
    print("hello Manager");
  }

  void permission(){
    print("manager");
  }

}
```

추상 클래스와 인터페이스는 Java를 공부할 때 많이 다루는 내용입니다. 상속, 추상 클래스, 인터페이스, 믹스인 등을 익히고나면 구현시 서로 헷갈리는 일이 많으니 잘 기억하고 잊지 않도록 주의해야 합니다.

## 07-3 멤버를 공유하는 믹스인

다른 객체지향 언어와 마찬가지로 Dart에선 다중 상속을 지원하지 않습니다. 그러나 여러 클래스에 선언된 멤버를 상속한 것처럼 이용하고 싶은 경우 믹스인(mixin)을 사용할 수 있습니다. 

지금까지 배운  상속, 추상 클래스, 인터페이스는 모두 공통적으로 class라는 키워드로 선언되지만, 믹스인은 선언시 따로 mixin이라는 키워드를 사용합니다. 따라서 믹스인은 클래스가 아니므로 생성자를 선언할 수 없고, 생성자가 존재하지 않으므로 객체를 생성할 수 없습니다. 

믹스인은 with 키워드를 통해 자식 클래스에 멤버를 공유할 수 있는데, 한가지 재미있는 점은 만약 서로 다른 mixin에 같은 이름의 멤버가 존재하는 경우 with 키워드 제일 뒤에 선언된 믹스인의 멤버값으로 재정의됩니다.
```dart
mixin User{
  int id = 0;
  bool isAdmin = false;
}

mixin Admin{
  bool isAdmin = true;
}

class Manager with Admin, User{} //User의 isAdmin 적용

class Developer with User, Admin{} //Admin의 isAdmin 적용

main(){
  final manager = Manager();
  print(manager.isAdmin); //false
  
  final developer = Developer();
  print(developer.isAdmin); //true
}
```

믹스인은 모든 클래스에서 사용 가능하지만, 만약 특정 클래스에서만 사용하도록 하고 싶을 경우 on 키워드를 통해 타입을 지정할 수 있습니다.

```dart
abstract class User{}

mixin Tester on User{
  int id = 0;
}

//class Manager with Tester{} User을 상속받은 클래스에서만 사용 가능 Error
 
class Developer extends User with Tester{}
```
중복되는 코드를 믹스인으로 묶어 선언하고 이용하면 코드의 가독성이 좋아지고 효율적으로 작성할 수 있습니다.

# 8장 - 사용자 인터페이스 아키텍쳐

## 08-1 화면을 구성하는 위젯

플러터(Flutter)는 리엑트(React)와 마찬가지로 선언형 프로그래밍입니다. 따라서 화면 구성 정보인 위젯(Widget)의 정보만 작성하면 프레임워크가 API를 통해 처리하여 화면을 출력합니다.

선언형과 반대되는 개념은 명령형 프로그래밍으로, 개발자가 화면 구성과 관련된 모든 코드를 작성해야 합니다. 

예를 들어 화면에 문자를 출력해야 한다면 플러터는 문자열 데이터만 입력하면 되지만, 명령형 프로그래밍의 경우 문자열 데이터뿐만 아니라 위치, 색상, 크기 등 화면에 대한 부가 정보들을 함께 코드로 작성해야 합니다. 

따라서 코드가 늘어나고 화면과 관련된 많은 API를 숙지하고 있어야합니다.

## 08-2 위젯 트리 알아보기

플러터 프레임워크에서 화면을 생성할 때 우리가 작성한 위젯 코드들은 위젯 트리(Widget Tree)가 됩니다. 그런데 플러터 프레임워크는 위젯 트리를 제외하고도 엘리먼트 트리(Element Tree)와 렌더 트리(Render Tree) 이렇게 2개의 트리를 더 만듭니다. 

위젯 트리는 개발자가 작성하는 화면 주문서에 해당하고, 어떤 위젯이 어떻게 사용되고 어떻게 배치되어야 하는지를 담고 있습니다. 이 주문서를 토대로 실제 화면을 구성하는 정보는 프레임워크 내부에서 엘리먼트 트리로 만듭니다. 

이러한 엘리먼트 트리의 구성요소는 ComponentElement와 RenderObjectElement 객체로 나뉘는데, ComponentElement 객체는 다른 객체를 포함하는 역할만 하며 따로 화면에 출력할 정보를 가지지는 않는 반면, RenderObjectElement 객체는 실제 화면에 출력할 정보를 담고 있습니다. 

엘리먼트 트리는 위젯 트리와 일대일로 매칭되며, 위젯의 위치정보와 상태정보를 가지고 있습니다. 따라서 나중에 위젯 트리의 위젯 정보가 변경될 경우 렌더 트리에게 이를 알리고 뷰를 다시 그릴 것을 요청하는 등의 중간자 역할을 수행합니다. 

렌더 트리는 엘리먼트 트리가 가진 화면 정보를 토대로 만들어지고 실제 화면에 출력되는 RenderObject 타입을 구현한 객체들로 이루어집니다. 렌더 트리는 엘리먼트 트리와 연결되지만, 위젯 트리와는 직접적으로 연결되지 않기 때문에 위젯이 여러번 다시 생성되거나 없어지더라도 렌더 트리가 위젯 트리에 의존적이지 않으므로 성능에 큰 영향을 미치지 않습니다.

플러터가 이와 같이 복잡한 트리 구조를 가지는 이유는 렌더링 속도 때문입니다. 화면 변경 사항을 최대한 빠르게 반영하는 것은 프레임워크의 성능을 결정하는 중요한 요소이며, 플러터는 네이티브 앱 수준의 성능을 목표로 하기 때문에 화면에 변화가 생길 경우 최적의 알고리즘으로 변경할 부분만을 다시 렌더링하기 위해 이러한 구조를 가지고 설계되었습니다.

## 08-4 동적인 화면 만들기

플러터에서 상태(State)관리에 대한 개념은 매우 중요합니다. 상태는 화면에서 갱신해야 하는 데이터를 의미합니다. 플러터의 위젯은 상태관리를 하지 않는 정적인 위젯 StatelessWidget과 상태관리를 하는 동적인 위젯 StatefulWidget으로 나뉩니다. 

StatelessWidget과 다르게 StatefulWidget을 상속 받은 클래스에선 직접적으로 build() 함수를 사용할 수 없습니다. 대신 State를 상속받는 상태 클래스를 만들어 build() 함수를 작성하고, 해당 클래스 객체를 createState() 함수에 반환하여 화면을 구성합니다. 

State 객체는 위젯의 속성을 가지고 있으므로 값이 변경되어 화면 갱신이 필요한 경우 자동으로 갱신되진 않고 따로 setState() 라는 함수를 호출하여 화면을 갱신해야 하는데, 그 이유는 화면 갱신과 관련 없는 변수가 변경 되었을 때도 build() 함수를 호출하면 매우 비효율적이기 때문입니다. 

또한 build() 함수가 호출 될 때 마다 무거운 위젯을 처음부터 다시 생성하는 것은 많은 비용이 들기 때문에 StatefulWidget은 위젯 트리 구조에 포함시켜 빌드시 다시 생성되게 만들고, 실제 데이터와 로직은 State 객체에 두고 엘리먼트 트리 구조에 포함시켜 메모리에 유지하면서 화면이 다시 빌드 될 때마다 재생성되지 않도록 합니다. 

추가로 키(key)에 대한 개념은 나중에 다시 다루겠지만, State 객체는 우선 타입을 기준으로 위젯을 찾고 같은 타입의 객체가 여러개 존재하면 키 값을 통해 찾습니다.

## 08-5 상태의 생명 주기

StatelessWidget과 StatefulWidget을 포함한 모든 위젯은 불변이며 화면 갱신시 매번 생성되므로 생성주기가 없지만, State는 한 번 생성된 후 메모리에 유지되므로 생명 주기를 가집니다. 

생명주기는 기본적으로 다음과 같은 순서로 동작합니다.

initState() -> didChangeDependencies() -> build() -> didUpdateWidget() -> dispose()

이제 각 생명주기 함수에 대한 호출 시점과 역할을 알아보겠습니다.

initState() 함수는 State 객체가 생성되자마자 가장 먼저 한 번만 실행됩니다. 따라서 최초에 서버와 연동하여 초기 데이터를 가져오거나, 이벤트 리스너(Event Listener)를 등록하는 등의 주로 상태값을 초기화하는 로직을 담당합니다. 

didChangeDependencies() 함수는 State 객체가 생성될 때 initState() 함수에 이어서 자동으로 호출됩니다. 그리고 위젯 트리에서 상위 위젯의 상태 데이터를 하위 위젯에 전달하여 반영해야 하는 경우 다시 호출됩니다.

따라서 상태관리 도구인 Provider이나 InheritedWidget에서 상위 위젯의 상태 데이터가 변경될 때 하위 위젯의 didChangeDependencies() 함수가 자동 호출되어 상위 위젯의 변경된 데이터를 이용할 수 있습니다.

build() 함수는 화면을 구성할 때 호출되는 함수이며 State 클래스를 작성할 때 꼭 재정의해야 합니다. build() 함수가 호출되는 경우는 3가지로 State 객체 생성에 따른 최초 호출, setState() 함수에 의한 호출, didUpdateWidget() 함수에 의한 호출로 구분됩니다. 

일단 State 객체가 생성되면 build() 함수까지 호출되어야 화면이 출력됩니다. 이 때의 State 상태를 Clean이라고 부르며, 화면 갱신이 필요하여 setState() 함수를 호출하면 Dirty 상태가 되어 자동으로 build() 함수를 다시 호출하게 됩니다.

didUpdateWidget() 함수는 위젯의 State 객체와 연결된 StatefulWidget이 다시 생성되는 순간 자동으로 호출됩니다. 화면을 구성하는 위젯은 트리 구조로 표현되며 상위 위젯에서 상태 변화에 따라 하위 위젯을 다시 생성해야 하는 경우가 발생합니다. 

이때 StatefulWidget은 매번 다시 생성되지만 State 객체는 메모리에 유지되므로 위젯이 다시 생성되었음을 감지할 수 있어야 합니다.

dispose() 함수는 State 객체가 소멸될 때 자동으로 호출됩니다. 따라서 주로 이벤트 리스너와 연결을 끊는 등의 작업을 구현합니다. dispose() 함수는 최후에 한 번 호출되는 함수이며, 호출된 후 객체에 접근하면 예외가 발생합니다.

## 08-6 BuildContext 객체와 위젯 키

build() 함수의 매개변수로 사용하는 BuildContext 객체에는 위젯을 이용할 때 필요한 다양한 정보가 들어 있습니다. 그 중 가장 대표적인 것은 위젯 트리에서의 위치와 관련된 정보입니다. 이 위치 정보를 이용해 위젯 트리의 상위 노드(위젯)를 참조할 수 있습니다.

사실 엘리먼트 트리는 BuildContext 객체로 이루어진 트리입니다. 그 이유는 엘리먼트 트리를 구성하는 ComponentElement와 RenderObjectElement의 상위 타입은 Element인데, Element는 BuildContext를 구현하여 만들어지기 때문입니다. 

그러므로 엘리먼트 트리에 유지되는 객체가 BuildContext 객체라고 보면 됩니다.

플러터에서 모든 위젯은 키(Key)를 가질 수 있으며, 키값은 보통 StatefulWidget을 식별하는 용도로 사용합니다. 만약 같은 타입의 위젯이 두 개 이상 존재할 때 위젯의 트리구조가 변경되는 경우 State 객체를 각 위젯 객체에 연결해야 하는데, 타입이 같으므로 구분이 불가능한 문제가 생깁니다.

따라서 둘 사이를 연결해서 사용하므로 키가 없으면 통신에 문제가 발생합니다.

위젯의 키값을 설정할 땐 Key의 하위 타입인 GlobalKey, LocalKey, ValueKey, UniqueKey, ObjectKey 등을 사용할 수 있습니다. GlobalKey와 LocalKey의 가장 큰 차이점은 키값 유일성의 적용 범위로 GlobalKey는 앱 전체에서, LocalKey는 위젯의 부모와 자식에서 유일한 값입니다. 

또한 GlobalKey는 다른 키들과 다르게 currentState, currentWidget 속성을 제공하여 키값으로 식별되는 위젯과 State 객체를 얻을 수 있습니다. 그러나 GlobalKey를 사용하면 위젯의 트리구조가 변동되거나 모든 위젯을 다시 빌드하는 경우가 발생할 수 있으므로 조심스럽게 사용해야 합니다.

LocalKey는 하위 클래스로 ValueKey(문자열 혹은 숫자 값), UniqueKey(난수 값), ObjectKey(객체형 키값)를 가지며 식별할 위젯의 부모 및 자식에서 유일한 값을 지정할 때 사용합니다.