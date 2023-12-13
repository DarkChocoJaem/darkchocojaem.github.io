---
title: "Do it 깡샘의 플러터 & 다트 프로그래밍 - 16, 17장 내용 정리(4)"
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

# 16장 - 퓨처와 스트림으로 비동기 프로그래밍

## 16-3 스트림과 스트림 빌더

비동기 관점에서 퓨처(Future)는 미래에 한 번 발생할 데이터를 의미하는 반면, 스트림(Stream)은 미래에 연속될 데이터 발생을 의미하며 반복해서 발생하는 데이터를 다룰 때 유용합니다.

Future와 Stream의 차이점은 데이터 반환시 Future은 데이터를 한 번만 반환하므로 return 문을 이용하지만 Stream은 여러번 반환하므로 yield 문을 사용합니다.

또한 Stream 타입을 반환하는 함수를 비동기로 만들 땐 aynsc* 와 await 키워드를 써야하고 listen() 함수를 통해 여러번 반환하는 데이터를 받아 낼 수 있다는 점입니다.

```dart
Future<int> futureFun() async{
  await Future.delayed(Duration(seconds: 1));
  return 5;
}

Stream<int> streamFun() async*{
  for(int i = 1; i < 6; i++){
    await Future.delayed(Duration(seconds: 1));
    yield i;
  }
}

main(){
  //5 출력
  futureFun().then((value) => print(value));
    
  //1부터 5까지 1초마다 한번씩 출력 
  streamFun().listen((value) => print(value)); 
}
```

스트림은 다음과 같은 다양한 생성자와 함수를 통해 비동기 연속 데이터를 유연하게 처리할 수 있도록 지원합니다.

- fromIterable() 생성자를 통해 Iterable 타입의 데이터를 스트림 객체로 생성 할 수 있습니다
- fromFuture() 생성자를 통해 Future 타입의 데이터를 스트림 객체로 생성할 수 있습니다
- periodic() 생성자를 통해 주기적으로 어떤 작업을 실행할 수 있습니다
- take() 함수를 통해 데이터 발생 횟수를 지정할 수 있습니다
- takeWhile() 함수를 통해 데이터 발생 횟수 조건을 지정할 수 있습니다
- skip() 함수를 통해 지정한 횟수 만큼 데이터 발생을 생략할 수 있습니다
- skipWhile() 함수를 통해 특정 데이터 발생 생략 조건을 지정할 수 있습니다
- toList() 함수를 통해 리스트로 변환 가능합니다

``` dart
import 'dart:async';

void main() async {
  
  // Iterable 데이터를 스트림 객체로 생성
  Stream<int> numStream = Stream<int>.fromIterable([1, 2, 3, 4, 5]);

  // 데이터 발생 횟수를 3회로 지정
  numStream = numStream.take(3);

  // 첫 두 데이터 발생을 생략
  numStream = numStream.skip(2);

  // 스트림 데이터를 비동기로 처리
  numStream.listen((value) => print(value)); //3 출력

  int count = 0;
  
  // 1초 마다 count에 1을 더하고 반환하는 스트림 생성
  Stream countStream = Stream.periodic(
    Duration(seconds: 1), (_) => count+=1);

  //value(count)가 < 3인 동안 반복
  countStream = countStream.takeWhile((value) => value < 3);

  // 스트림 데이터를 리스트로 변환
  List countList = await countStream.toList();
  
  print(count); // 3초 후 3 출력
  
  print(countList); //3초 후 [1,2] 출력
  
}
```

참고로 takeWhile() 조건문 비교시 false가 반환되는 시점은 count += 1 이후이므로 count 값은 3이고, 리스트에 들어있는 값은 [1, 2] 입니다. 

1. 1초 후 count++ 실행, value(count) < 3 비교(true), count = 1
2. 1초 후 count++ 실행, value(count) < 3 비교(true), count = 2
3. 1초 후 count++ 실행, value(count) < 3 비교(false), count = 3
4. 1,2 반환

StreamBuilder은 FutureBuilder와 마찬가지로 미래에 받을 데이터를 출력할 수 있도록 해주는 위젯입니다.

StreamBuilder 생성자의 stream 매개변수에는 반복해서 발생할 데이터 스트림을 지정해주고, builder 매개변수에는 UI를 구성할 함수를 지정합니다.

이때 builder에는 BuildContext 객체와 함께 AsyncSnapshot 객체를 인자로 넘겨주어 data 상태를 판단하고 적절하게 사용할 수 있도록 합니다.

AsyncSnapshot은 스트림의 결과를 받아 자동으로 제공되며, AsyncSnapshot의 connectionState라는 속성은 데이터 발생을 기다리고 있는지, 발생 중인지, 발생이 끝났는지 등의 여부를 알 수 있습니다.

Future는 미래에 한번 발생할 데이터를 API로부터 받아올 때 유용하고, 스트림은 실시간 채팅 같은 언제 어떤식으로 발생할지 모르는 연속된 데이터를 받아올 때 유용합니다.

## 16-4 스트림 구독, 제어기, 변환기

StreamSubscription은 스트림의 데이터를 소비하는 구독자입니다. 구독자라는 말은 생소할 수 있지만, 여기선 반복해서 발생하는 데이터를 처리하는 존재(타입)라고 이해하면 될 것 같습니다. 대표적으로는 이전에 예시로 들었던 listen() 함수가 있고 이 함수의 반환 타입이 StreamSubscription입니다. 

listen() 함수의 매개변수엔 데이터를 받는 기능 외에 오류나 데이터 발생이 끝났을 때 실행할 함수를 지정할 수 있고, listen() 함수에 구현하는 것이 복잡하다면 listen() 함수의 매개변수로 null을 전달하여 별도로 데이터 처리를 진행하지 않는 대신 반환값으로 StreamSubscription을 받아 따로 구현할 수 있습니다.

```dart

void main() {
  // Iterable 데이터를 스트림 객체로 생성
  Stream<int> numStream = Stream<int>.fromIterable([1, 2, 3, 4, 5]);
    
  //listen() 함수를 통한 데이터 처리
  numStream.listen((value) => print(value),
    onError : (error) {
      print(error);
    },
    onDone : (){
      print("done");
    }
  );
  	
  //Subscription 객체를 통한 데이터 처리
  StreamSubscription subscription = numStream.listen(null);

  subscription.onData((value) => print(value));

  subscription.onError((error) => print(error));

  subscription.onDone(() => print("done"));
}
```

StreamController은 스트림의 제어기로, 필수적으로 이용해야 하는 것은 아니지만 내부적으로 하나의 스트림을 가지고 있으며 스트림과 관련된 작업을 더 쉽게 관리하고 제어할 수 있게 해줍니다.

```dart

void main() {
  
  StreamController controller = StreamController();
  
  Stream<int> stream1 = Stream<int>.fromIterable([1, 2, 3]);

  Stream<String> stream2 = Stream<String>.fromIterable(["A","B","C"]);

  stream1.listen((value) => controller.add(value));

  stream2.listen((value) => controller.add(value));

  controller.stream.listen((value) => print(value));
  // 1 A 2 B 3 C 순으로 출력
}
```

StreamTransformer은 발생한 데이터를 변환하여 listen() 함수에서 사용할 때 유용합니다. 스트림 변환기를 사용할 때는 StreamTransformer 객체를 선언하고 stream.transform() 함수에 매개변수로 전달하여 Stream으로 발생한 데이터가 스트림 변환기를 거치도록 하면 됩니다. 

단, StreamTransformer의 fromHandlers() 함수에 전달하는 EventSink 객체에 add() 함수로 값을 추가해주지 않을 경우 listen() 함수에 값이 전달되지 않습니다. 이는 EventSink의 add() 메소드 호출이 누락될 경우, 변환 작업이 정상적으로 수행되지 않았거나 변환된 결과가 없다고 간주하여, 다음 단계로 전달되지 않기 때문입니다.

```dart

void main() {
  
  Stream<int> stream = Stream<int>.fromIterable([1, 2, 3]);
  
  //Transformer 생성
  StreamTransformer<int, dynamic> transformer = StreamTransformer
  .fromHandlers(handleData : (value, sink) {
    value = value +1;
    sink.add(value); //해당 부분 주석 처리시 출력결과 없음
  });

  //Transformer 적용
  stream.transform(transformer).listen((value) => print(value));
  
}
```

스트림 변환기를 이용하면 Stream 데이터의 로그(log) 출력 및 데이터 변환에 용이합니다.

# 17장 - 아이솔레이트로 비동기 프로그래밍

## 17-1 아이솔레이트 소개

다트 언어는 스레드(Thread)를 지원하지 않고 아이솔레이트(Isolate)를 통해 작업을 수행합니다. 아이솔레이트는 스레드와 유사하지만, 서로 다른 아이솔레이트의 메모리를 독립적으로 격리시켜 작업하므로 스레드보다 안전하게 비동기를 처리할 수 있습니다. 

다트의 메인 함수는 메인 아이솔레이트라고 부릅니다. 메인 아이솔레이트의 하위에 새로운 실행 흐름을 만들고 싶을 경우, 별도의 아이솔레이트를 생성해야 하는데 이때 spawn() 함수를 사용합니다.

```dart
import 'dart:isolate';

myIsolate1(var arg){
  Future.delayed(Duration(seconds: 2), () {
    print("myIsolate 1");
  });
}

void main(){

  myIsolate2(var arg){
    Future.delayed(Duration(seconds: 2), () {
      print("myIsolate 2");
    });
  }

  print("before Isolate");

  Isolate.spawn(myIsolate1, 1);
  Isolate.spawn(myIsolate2, 2);

  print("after Isolate");

  //실행 결과는 다음과 같은 순으로 출력.
  //before Isolate
  //after Isolate
  //myIsolate 1
  //myIsolate 2
}
```

## 17-2 포트로 데이터 주고받기

새로 생성한 아이솔레이트는 메인 함수(메인 아이솔레이트)와 메모리로부터 독립적이므로 메인 함수의 변수 값이 변경되어도 영향을 받지 않습니다. 따라서 원칙적으론 서로 다른 아이솔레이트는 직접적으로 데이터를 주고 받을 수 없습니다. 그러나 데이터를 주고 받아야 하는 상황이 발생할 경우 ReceivePort와 SendPort를 이용해 구현할 수 있는데, ReveivePort와 SendPort는 서로 쌍을 이루며 한 아이솔레이트 내에 여러 개 공존할 수 있습니다. 

먼저 데이터를 받을 곳에 ReceivePort를 만들고, 이 ReceivePort를 통해 SendPort를 만들어 spawn() 함수의 매개변수로 전달해야 합니다. 그리고 새로 생성된 아이솔레이트에선 매개변수로 전달받은 SendPort의 send()함수를 이용해 데이터를 메인 함수로 전달하고, 메인 함수에서 ReceivePort의 first 속성을 이용해 단일 값을 받아 올 수 있습니다.

```dart
import 'dart:async';
import 'dart:isolate';

myIsolate1(SendPort sendPort){
  Future.delayed(Duration(seconds: 2), () {
    sendPort.send("Isolate 1");
  });
}

void main() async{

  ReceivePort receivePort = ReceivePort();
  
  await Isolate.spawn(myIsolate1, receivePort.sendPort);
  
  String data = await receivePort.first;
  
  print(data); //Isolate 1 출력
}
```

그러나 ReceivePort의 first 속성은 한번 데이터를 받은 후에 포트가 자동으로 닫히므로 값을 지속적으로 공유해야 하는 경우 listen() 함수를 이용해야 합니다. 그리고 데이터 공유 작업 후 포트를 닫을 땐 close() 함수를 사용합니다.

```dart
import 'dart:async';
import 'dart:isolate';

//1초마다 값을 1씩 증가시켜 보내는 작업
myIsolate1(SendPort sendPort){
  int counter = 0;
  Timer.periodic(Duration(seconds: 1), (Timer timer){
    sendPort.send(++counter);
  });
}

void main() async{

  ReceivePort receivePort = ReceivePort();
    
  await Isolate.spawn(myIsolate1, receivePort.sendPort);
  
  receivePort.listen((data){
    if((data as int) > 5){
      receivePort.close();
    } else{
      print(data);
    }
  });
}
```

만약 단일 포트가 아닌 다중 포트를 이용하여 데이터를 주고 받고 싶다면 아래와 같이 구현할 수 있습니다.

```dart
import 'dart:async';
import 'dart:isolate';

//count를 받아서 제곱하여 반환하는 작업을 하는 아이솔레이트
myIsolate1(SendPort sendPort){
  ReceivePort isoPort = ReceivePort();
    
  sendPort.send({'port' : isoPort.sendPort});
    
  isoPort.listen((map){
    if(map['value'] != "finish"){
      int count = map['value'];
      //아에솔레이트에서 count를 제곱해 다시 메인 함수로 전송
      sendPort.send({'value' : count * count});
    }else{
      isoPort.close();
    }
  });
}

void main() async{

  ReceivePort receivePort = ReceivePort();
    
  var isolate = await Isolate.spawn(myIsolate1, receivePort.sendPort);
  
  SendPort? isoPort;
  
  receivePort.listen((map){
    if(map['port'] != null){
      isoPort = map['port'];
    }else if(map['value'] != null){
      print(map['value'])
    }
  });
  
  int count = 0;
  Timer.periodic(Duration(seconds : 1), (Timer timer){
    count++;
    if(count < 5){
      isoPort?.send({'value' : count});
    }else{
      isoPort?.send({'value' : "finish"});
      receivePort.close();
    }
  });
  
  //실행 결과는 다음과 같다
  // 1
  // 4
  // 9
  // 16
}
```

조금 어려워 보이는 예제이지만 자세히 흐름을 따라가보면 쉽게 이해할 수 있습니다.

사실 간단한 작업의 경우 compute() 함수를 이용하면 조금 더 편리하게 아이솔레이트를 이용할 수 있는데, compute() 함수는 특정 작업을 아이솔레이트로 실행하고 최종 결과를 반환하는 함수입니다. 즉, 데이터를 반복해서 주고받는 구조가 아니라 모든 작업을 완료한 후 최종 결과값을 반환합니다.

```dart
import 'dart:isolate';

//1초마다 값을 1씩 증가시켜 보내는 작업
myIsolate1(int num){
  int sum = 0;
  for (int i = 0; i <= no; i++){
    sleep(Duration(seconds : 1)); //1초 쉬고 다음 연산
    sum += i;
  }
  return sum;
}

void main() {

  compute(myIsolate1, 3).then((value) => print(value));
  
  //출력 결과는 아래와 같다 (1초 마다 출력)
  // 1
  // 2
  // 3
}
```