---
title: "Do it 깡샘의 플러터 & 다트 프로그래밍 - 3, 4, 5, 6장 내용 정리(1)"
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

"Do it 깡샘의 플러터 & 다트 프로그래밍" 책을 공부하면서 유용했던 내용들을 위주로 정리하고자 합니다.

# 3장 - 처음 만나는 다트

## 03-3 라이브러리 만들기

플러터를 이용하다 보면 외부 패키지를 이용할 일들이 많은데 이러한 외부 패키지의 코드를 들여다보면 library, export, part of 등의 키워드들이 많이 보입니다.

플러터에서 다른 파일을 불러오기 위해선 import 구문을 사용합니다.

```dart
//main.dart
import a.dart;
import b.dart;
```

그러나 파일 수가 늘어나면 일일이 import 구문을 사용하기 어렵습니다. 따라서 관련된 파일들을 하나의 라이브러리로 만들어 사용하는 것이 좋습니다.

만약 a.dart, b.dart라는 파일들을 myLib.dart 파일의 라이브러리로 만들 경우 다음과 같이 작성할 수 있습니다.

```dart
//a.dart
part of my_lib;
int _a = 10;
```

```dart
//b.dart
part of my_lib;
int _b = _a + 10; //a.dart 파일의 비공개 멤버 이용 가능
```

여기서 part of는 my_lib라는 이름의 라이브러리에 포함한다는 의미입니다.
part of 로 선언한 파일은 part 예약어를 통해 불러오며, 라이브러리에 속한 다른 파일의 비공개 변수를 사용할 수 있습니다.

또한 여러 파일들을 import할 필요 없이 필요한 곳에서 my_lib 파일만을 import하여 사용해 코드 수를 절약할 수 있습니다.

```dart
//myLib.dart
library my_lib;
part 'a.dart';
part 'b.dart'
```

```dart
//main.dart
import 'myLib.dart' //a.dart, b.dart 포함
```

추가로 책에는 언급되어 있지 않지만 export 키워드의 경우 다른 파일에 정의된 여러 모듈을 한 파일에 모아서 사용하고자 할 때 유용합니다.

```dart
//main.dart
import 'myLib1.dart';
import 'myLib2.dart';
```

위의 파일들을 다음과 같이 myExport.dart 파일에서 export 키워드로 선언할 수 있습니다.

```dart
//myExport.dart
export 'myLib1.dart';
export 'myLib2.dart'
```

이후 library 방식과 마찬가지 방식으로 import하여 사용합니다.

```dart
//main.dart
import 'myExport.dart';
```

library와 export의 방식은 유사해 보이지만 library, part of, part는 캡슐화 및 비공개 멤버를 구현하고 지원할 때 이점이 있는 반면, export의 경우 공개 멤버만 모아두어 사용하고자 하는 경우에 적합합니다.

# 4장 - 데이터 타입과 널 안정성

## 04-2 상수 변수 - const, final

Dart에서 const와 final은 제가 자주 사용하는 키워드이지만, 정확한 정의를 모르고 그저 불변인 상수로 선언하기 위해 사용한다고 생각하고 있었습니다.

const와 final 키워드는 다음과 같은 특징을 가지고 있습니다.

const는 컴파일 타임 상수 변수입니다. 따라서 변수를 선언할 때 초깃값을 대입해야 합니다. const 키워드는 Top-level이나 함수 내의 지역 변수로 선언할 때 사용할 수 있지만 클래스 내부에선 static 변수로만 선언할 수 있습니다. 그 이유는 클래스의 인스턴스는 각 객체마다 별도의 값을 가지며 인스턴스가 생성될 때마다 값이 초기화 되지만 const 변수는 한 번 할당된 후 변경되지 않아야 하기 때문입니다.

즉, const 변수는 컴파일 타임에 메모리 상에 올리기 위해 static 키워드를 사용하여 인스턴스에 종속되지 않는 클래스 변수로 지정해야 합니다.

```dart
//const int data1; Error
const int data1 = 1;

class User {
  //static const int data2; Error
  static const int data2 = 2;
    
  void some(){
    //const int data3; Error
    const int data3 = 3;
    
    //data3 = 4; 값을 바꿀 수 없으므로 Error
  }
} 
```

final은 런타임 상수 변수입니다. 따라서 Top-level 뿐만 아니라 클래스나 함수 내부에서 자유롭게 사용할 수 있습니다.

final도 const와 마찬가지로 값을 변경할 수 없지만, 런타임 변수이므로 초기 값 대입이 꼭 선언과 동시에 이루어지지 않아도 상관없고 객체를 생성하거나 함수에서 값을 참조하기 이전에만 초기화를 진행해주면 됩니다.

```dart
//final int data1; 값을 대입하지 않았으므로 변수 사용시 Error

class User {
  final int data2 = 2;
    
  void some(){
    final int data3;
        
    data3 = 3;
    //data3 = 4; 값을 바꿀 수 없으므로 Error
  }
} 
```

## 04-4 컬렉션 타입 - List, Set, Map

List는 filled() 생성자를 통해 크기를 지정할 수 있는데 이때 List의 크기를 자유롭게 늘리고 싶을 경우 growable 매개변수에 true를 넘겨주면 됩니다.

```dart
main(){
  //크기가 2이고, 초기 값은 0으로 채워지는 List 생성
  var list1 = List<int>.filled(2, 0);
  list1[1] = 10;
  //list1.add(20); 크기를 늘릴 수 없으므로 원소 추가시 Error
    
  //크기가 3이고, 초기 값은 0으로 채워지며, 크기 변경이 가능한 List 생성
  var list2 = List<int>.filled(2, 0, growable: true);
  list2[1] = 10;
  list2.add(20);
}
```

추가로 책에 언급된 generate() 생성자는 리스트에 일정한 패턴을 가진 원소들을 채울 수 있게 해줍니다.

# 5장 - 함수와 제어문

## 05-2 명명된 매개변수

원래 함수를 호출할 때는 기본적으로 함수에 선언된 매개변수의 타입, 순서에 맞춰서 인자를 전달해야 하지만 명명된 매개변수(Named Parameter)는 Optional이므로 값을 넘겨주지 않아도 되고 매개변수의 순서로부터 자유롭습니다.

```dart
//기본적인 함수
void a(int? value){
  print("a");
}

//Named Parameter을 이용한 함수
void b({int? value}){
  print("b");
}
```

위와 같이 Named Parameter는 매개변수 타입과 이름을 중괄호로 묶어 표현하며, 아래와 같이 사용할 수 있습니다.

```dart
main(){
  //a(); 매개변수의 갯수를 맞춰서 호출해야하므로 Error
  a(null); //value에 null 전달
    
  b(); //자동으로 value에 null이 전달
  b(value: null); //"name:value"형식으로 값 전달
}
```

개인적으로 API 연결에 사용할 메서드가 어떤 매개변수를 필요로 하는지 쉽게 알아보고 지정할 수 있어서 유용했습니다.

## 05-3 위치 매개변수

위치 매개변수(Positional Parameter) 역시 Optional이므로 값을 넘겨주지 않아도 되지만, 매개변수가 선언된 순서에 맞게 호출해야 합니다.

```dart
//기본적인 함수
void a(int? value1, int? value2){
  print("a");
}

//Positional Parameter을 이용한 함수
void b([int? value1, int? value2]){
  print("b");
}
```

위와 같이 Positional Parameter는 매개변수 타입과 이름을 대괄호로 묶어 표현하며, 아래와 같이 사용할 수 있습니다.

```dart
main(){
  //a(); 매개변수의 갯수를 맞춰서 호출해야 하므로 Error
  a(null, null); // value1, value2에 null 전달

  b(); //자동으로 value1, value2에 null이 전달
  b(1,2); //value 1 = 1, value 2 = 2 전달
}
```

추가적으로 Named Parameter와 Positional Parameter는 다음과 같은 특징이 있습니다.

- 함수 파라미터의 가장 마지막 부분에 작성해야 합니다.
- 한 함수에서 2번 선언할 수 없습니다.
- 한 함수에서 두 파라미터를 동시에 사용할 수 없습니다.

```dart
main(){
  //Named Parameter
  void some1(int? a, {int? b, int? c}){}
  //void some2({int? a, int? b}, int? c){} Error
  //void some3({int? a, int? b}, {int? c}){} Error
    
  //Positional Parameter
  void some4(int? a, [int? b, int? c]){}
  //void some5([int? a, int? b], int? c){} Error
  //void some6([int? a, int? b], [int? c]){} Error
    
  //Both
  //void some7({int? a}, [int? b]){} Error
}
```

# 6장 - 클래스와 생성자

## 06-1 클래스와 객체

클래스 내부에서 선언한 변수와 함수는 각각 멤버 변수(Instance Variable), 멤버 함수(Method)라고 부릅니다. 그리고 이 멤버들은 다시 객체(Instance of Class) 맴버와 클래스(혹은 정적: Static) 멤버로 구분됩니다.

이해를 돕기 위해 아래의 코드 예시를 살펴보겠습니다.

```dart
class User{
  static int id = 1; //멤버 변수(인스턴스 변수) : 정적 멤버
  int money = 0; //멤버 변수(인스턴스 변수) : 객체 맴버
    
  User(){} //기본 생성자
    
  void some(){} //멤버 함수(메서드) : 객체 맴버
}
```

User 클래스의 객체 멤버와 클래스(정적) 멤버는 다음과 같이 접근할 수 있습니다.

```dart
class User{
  static int id = 1; 
  int money = 0;
    
  User(){} //기본 생성자
    
  void some(){}
}

main(){
  User.id = 2; //정적 멤버는 객체 생성 없이 접근 가능
    
  User user = User();
  user.money = 100; //객체 멤버는 객체 생성 후 접근 가능
}
```

이 내용은 이전에 설명한 const, final과 비슷한 경우입니다. static으로 선언된 변수는 컴파일 시점에 메모리의 Method 영역에 할당되므로 객체 생성 없이 접근 가능하며, 객체 멤버는 런타임 시점에 메모리의 Heap 영역에 할당되므로 객체 생성 후 접근이 가능합니다.

## 06-2 생성자와 멤버 초기화

생성자는 클래스의 객체를 생성하는 역할을 하며, 클래스에 생성자가 없을 경우 컴파일러가 자동으로 기본 생성자를 추가합니다. Dart에서 생성자를 선언할 땐 초기화 목록을 사용할 수 있습니다. 초기화 목록은 생성자 오른쪽에 ":"으로 구분하여 작성합니다.

```dart
class Example{
  //클래스에 생성자가 없을 경우 컴파일러가 자동으로 기본 생성자 추가
  //Example(){} 따라서 기본 생성자 생략 가능
}

class User{
  late String name;
  late int age;
  //User(String name, int age) : this.name = name, this.age = age {}
  User(this.name, this.age); //위 생성자는 이렇게 간소화 가능
}
```

초기화 목록은 매개변수를 멤버에 대입하는데 사용할 수 있지만, 그보다 함수에 인자로 넣어 멤버를 초기화 하거나 List로부터 특정 항목을 선택해 멤버를 초기화 할 때 주로 쓰입니다. 다만 생성자 초기화 목록의 실행 시점이 객체 생성 이전이므로 정적(Static)인 멤버를 호출해야 합니다.

```dart
class MyClass1{
  late int data1;
  late int data2;
    
  MyClass1(List<int> args) : 
    this.data1 = args[0], this.data2 = args[1] {}
}

class MyClass2{
  late int data1;
  late int data2;
  
  MyClass2(int arg1, int arg2) : 
    this.data1 = calFun(arg1), this.data2 = calFun(arg2) {}
  
  static int calFun(int arg){
    return arg * 10;
  }
}
```

## 06-3 명명된 생성자

명명된 생성자(Named Constructor)는 한 클래스에서 이름이 다른 생성자를 여러 개 선언하는 기법입니다.

```dart
class User{
  User(){} //기본 생성자 없을 경우 Error
  User.init(){}
  User.auto(){}
}
```

다른 프로그래밍 언어에선 비슷하게 생산자 오버로딩 기능을 제공하지만, 이름을 다르게 해서 여러 생성자를 만들 수 있는 건 각각의 생성자를 좀 더 명료하게 구분하려는 Dart언어의 특징입니다.

한 클래스에서 생성자를 여러 개 선언해서 다른 생성자를 호출해야 하는 경우가 존재한다면 생성자를 중복 호출하여 객체를 생성할 수 있는데, 이때 this 키워드를 이용합니다.

this 키워드는 현재 클래스의 인스턴스(객체)를 의미합니다. 여기서 주의할 점은 생성자를 중복해서 초기화 하는 상황을 방지하기 위해 생성자 본문에서 리다이렉팅 생성자를 호출할 수 없으며, 초기화 목록에서 작성해야 합니다. 또한 리다이렉팅 생성자와 다른 초기화 구문을 함께 사용할 수 없습니다.

아래의 코드를 살펴보겠습니다.

```dart
class User{
  late int id;
  late String name;
  
  User(int id, String name){
    //생성자 본문
    print("User id : $id, User name : $name");
  } //기본 생성자 
  
  /*
  User.init(int id){
    //this.id = 0; 해당 코드와 함께 아래를 호출할 경우 중복 초기화 Error
    this(id, "Jaem"); //본문에서 리다이렉팅 생성자 호출 불가 Error
  }
  */
      
  User.init(int id) : this.(id, "Jaem");
  
  User.auto() : this.init(0);
}
```

명명된 생성자는 클래스의 인스턴스를 생성할 때 다양한 방법으로 초기화가 필요한 경우 유용하다고 하니 복잡한 클래스를 만들 때 이용하면 좋을것 같습니다.

## 06-4 팩토리 생성자

팩토리 생성자(Factory Constructor)는 클래스 외부에서 생성자처럼 이용하지만 생성자 호출만으로 객체가 생성되진 않고, 상황에 맞는 객체를 준비해서 반환합니다. 따라서 팩토리 생성자가 선언된 클래스에는 객체를 생성하는 다른 생성자가 함께 선언됩니다.

아래의 예시 코드를 확인해보겠습니다.

```dart
class User {
  final String name;
  final int id;

  // private 기본 생성자
  User._(this.name, this.id);

  // 인스턴스 저장소
  static final Map<String, User> _instances = {};

  // 팩토리 생성자
  factory User(String name, int id) {
    if (_instances.containsKey(name)) {
      //같은 이름이 존재할 경우 해당 객체 반환
      return _instances[name]!;
    } else {
      //아닐 경우 새로운 객체 생성 및 저장소에 추가
      final newUser = User._(name, id);
      _instances[name] = newUser;
      return newUser;
    }
  }
}

void main() {
  User user1 = User('Jaem', 0);
  User user2 = User('Jaem', 1);

  print(user1 == user2); // true : name = Jaem, id = 0
}

```

팩토리 생성자 이용하면 캐시(Cache)와 유사하게 구현할 수 있습니다. 같은 정보를 가진 객체를 새로 만들지 않고 인스턴스 저장소로부터 재사용하며, 새로운 정보를 가친 객체의 경우 새로 만들어 반환합니다.

따라서 객체 생성 로직을 캡슐화 하여 클라이언트 코드에서 직접 복잡한 객체 생성 과정을 거치지 않아도 됩니다. 또한 객체를 동적으로 생성해 프로그램을 유연하게 동작시킬 수 있습니다.

## 06-5 상수 생성자

상수 생성자는 const 키워드로 선언하며 본문을 가질 수 없습니다. 또한 클래스의 모든 멤버 변수가 final로 선언되어야 하며, 객체를 const 형태로 생성할 수 있습니다. 상수 생성자는 클래스의 모든 변수를 초기값으로 사용하도록 강제하기 위해 사용합니다.

```dart
class User{
  //int id; final로 선언해야 하므로 Error
  final int id;
  const User(this.id);
}

main(){
  final User user = const User(0);
}
```

한가지 특징은 상수 생성자로 만든 일반 객체의 경우 서로 다른 존재이만, const를 붙여 생성한 객체는 같은 존재입니다(이전 객체 재사용).

```dart
class User{
  //int id; final로 선언해야 하므로 Error
  final int id;
  const User(this.id);
}

main(){
  var user1 = User(0);
  var user2 = User(0);
  print(user3 == user4); //false

  var user3 = const User(0);
  var user4 = const User(0);
  print(user4 == user5); //true
    
  print(user1 == user3); //false
}
```

같은 값으로 생성한 객체를 재활용할 수 있기 때문에 동일한 Widget을 여러 개 찍어낼 때 메모리 효율성을 높여줍니다.
