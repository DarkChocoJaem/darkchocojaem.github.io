---
title: "상태관리 패키지 Riverpod 공식문서 정리(1)"
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
