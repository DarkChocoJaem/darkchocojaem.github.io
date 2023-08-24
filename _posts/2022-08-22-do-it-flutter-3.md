---
title: "Do it 깡샘의 플러터 & 다트 프로그래밍 - 10, 12, 13, 14, 15장 내용 정리(3)"
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

# 10장 - 위젯 배치하기

## 10-3 크기 설정하기

IntrinsicWidth나 IntrinsicHeight 위젯을 이용하면 자식 위젯인 Row나 Column에 추가한 여러 위젯들 중 크기가 가장 큰 것을 기준으로 모든 위젯들의 크기를 똑같이 설정할 수 있습니다.이름처럼 IntrinsicWidth는 가로 크기를 통일하고, IntrinsicHeight는 세로 크기를 통일합니다.

단, 두 위젯을 사용할 땐 Row나 Column의 crossAxisAlignment 값을 CrossAxisAlignment.strech 로 설정해주어야 합니다. CrossAxisAlignment.strech 값은 각 위젯에 설정된 교차축 크기를 무시하고 전체 공간을 차지하도록 위젯들을 늘립니다.

가끔가다 IntrinsicWidth나 IntrinsicHeight를 사용할 때, Row나 Column의 crossAxisAlignment 값을 CrossAxisAlignment.strech로 설정해야 한다는 사실을 잊는 경우가 종종 있으니 주의하시길 바랍니다.

## 10-4 기타 배치와 관련된 위젯

빈 공간을 채우기 위해 사용하는 Spacer라는 위젯이 있습니다. 화면 구성시 특정 위젯들은 우측에 딱 붙여서 출력하고, 특정 위젯들은 좌측에 딱 붙여 출력해야 하는 경우에 사용할 수 있습니다.

개발할 땐 Row를 하나 만들고 거기에 위젯들을 담은 뒤 해당 Row와 또 다른 위젯들을 담은 Row를 생성하여, mainAxisAlignment 속성을 MainAxisAlignment.SpaceBetween으로 설정했었는데, 이것보다 훨씬 편한 방법인 것 같습니다.

# 12장 - 목록 구성과 다이얼로그 위젯

## 12-1 리스트 뷰

리스트 뷰의 ListView.builder() 생성자는 itemCount와 itemBuilder이라는 속성이 있습니다. itemCount는 위젯 항목의 수이며, itemBuilder은 항목을 구성하는 위젯을 만들어주는 함수로, 이 함수에서 반환된 위젯이 각 항목에 출력됩니다. 

여기서 중요한 점은 itemCount에 100을 설정하더라도 itemBuilder에 지정한 항목 위젯을 만드는 함수가 처음부터 100번 호출된는 것이 아니라, 처음 화면에 나올 개수만큼만 호출되며 이후 스크롤 발생으로 인해 항목이 더 필요해질 때 추가적으로 호출됩니다.

리스트 뷰를 만들 때 메모리를 효율적으로 사용하기 위해 필요할 때 itemBuilder를 추가로 호출하는 것 같습니다. 

앱을 만들다보면 게시판과 같은 무한 스크롤링을 구현해야 할 일이 있는데 개인적으로 까다롭다고 생각하는 부분입니다. 

그런데 이 내용을 보니까 사용자가 스크롤하다가 항목이 필요해지면 자동으로 itemBuilder가 호출된다고 하니, 그냥 itemCount를 명시하지 않고 ListView를 생성한 후 로직을 짜서 itemBuilder에서 API를 추가 호출 시키면 되지 않을까 싶어 나중에 한번 해봐야겠습니다.

# 13장 - 머티리얼과 쿠퍼티노 디자인

## 13-5 커스텀 스크롤 뷰와 슬리버 앱바

Scaffold의 appbar 속성을 이용하면 화면 상단을 다양하게 꾸밀 수 있지만 Appbar 영역이 커질수록 본문 크기가 줄어들게 됩니다. 따라서 화면 상단이 커지면 본문이 스크롤될 때 접혔다가 다시 나오는 것이 요즘 앱들의 트렌드입니다.

이처럼 화면의 한 영역에서 스크롤이 발생할 때 다른 영역도 함께 스크롤 되도록 할 땐 CustomScrollView를 사용합니다. 

그러나 CustomScrollView 하위에 추가한 위젯이 전부 스크롤 가능한 것은 아니므로, 스크롤 정보를 공유할 수 있는 위젯을 사용해야 합니다. 이를 위해 SliverList, SliverFixedExtentList, SliverGrid, SliverAppbar 등을 사용할 수 있습니다.

토스(Toss)나 인스타그램(Instagram) 등 IT 대기업들이 만든 앱들을 보면 대부분 스크롤할 때 Appbar 부분이 함께 접히는 것을 볼 수 있는데 이런 앱을 만들때 유용할 것 같습니다.

# 14장 - 내비게이션을 이용한 화면 전환

## 14-2 내비게이션 2.0 사용하기

플러터는 웹, 앱 등의 다양한 플랫폼 지원이 시작되면서 화면이 복잡해짐에 따라 화면 전환 기법이 필요해졌고, 기존 내비게이션으로는 여러 페이지를 추가하거나 제거하는 것이 어려우며 현재 화면 스택 아래에 있는 화면을 제거하려면 복잡했기 때문에 이런 점들을 개선하고자 API를 추가하여 내비게이션 2.0을 출시했습니다.

내비게이션 2.0의 기본 구조 다음과 같습니다.
- Page(출력할 화면 정보를 가진 클래스)
- Router(페이지 정보를 스택 구조로 가지는 위젯 클래스)
- RouteInformationParser(라우팅 요청 분석 클래스)
- RouteDelegate(상황에 맞게 라우팅을 처리 하는 클래스) 

페이지(Page)는 위젯을 포함한 화면을 구성하는 정보이며, 서브 클래스로 만들어진 MaterialPage나 CupertinoPage 클래스를 통해 구현됩니다.

그리고 이 페이지를 스택(Stack) 구조로 출력해주는 위젯이 내비게이터(Navigator)입니다. 내비게이터는 onPopPage라는 속성을 꼭 설정해야 하는데, 이 속성 값은 앱바에서 제공하는 뒤로가기 버튼을 누를 때 호출됩니다.

만약 화면 구성이 복잡해질 경우 RouteInformationParser와 RouteDelegate를 이용해야 하며, 구현 절차는 아래와 같습니다.

먼저 앱의 라우팅 설계에 맞게 라우트 경로(RoutePath) 클래스를 작성하고, 플랫폼이 앱을 처음 실행하거나 라우팅될 때 정보를 분석하고, 라우팅이 결정된 후 현재 라우팅 상태를 저장하는 작업을 진행하는 RouteInformationParser를 상속받는 클래스를 구현해야 합니다. 

RouteInformationParser의 라우팅 정보 분석은 parseRouteInformation() 함수에서 구현하며, 반드시 재정의해야 합니다. 이 함수는 앱이 실행될 때 URL의 정보를 받아 어떤 정보로 실행된 것인지를 분석해 그 결과를 라우트 경로에 담아 반환합니다.

RouteInformationParser의 라우팅 정보 저장 기능은restoreRouteInformation() 함수에서 구현하며, 선택적으로 재정의할 수 있습니다. 이 함수는 현재 라우팅의 상태를 저장하는 역할을 하고 RouteDelegate에서 화면 전환이 결정되면 자동으로 호출됩니다.

다음으로 RouteDelegate를 상속받는 클래스를 구현해야 합니다. RouteDelegate은 라우팅 대리자로서 RouteInformationParser가 분석한 경로를 받아 내비게이터를 만들어 내고 라우팅을 담당합니다.

RouteDelegate를 구현할 땐 RouteDeletage 초기화시 처음 호출되는setNewRoutePath() 함수를 반드시 재정의해야하며, RouteInformationParser에서 처음에 앱의 라우팅 정보를 분석하고 만들어낸 결과를 이 함수에 전달하여 초기 라우팅 정보를 생성합니다. 

그리고 필수는 아니지만 build() 함수 호출 이전에 자동으로 동작하는 currentConfiguration() 함수를 재정의할 수 있습니다. 이 함수는 라우팅 때 마다 호출되고, 함수에서 만든 정보가 RouteInformationParser에 라우팅 정보로 저장됩니다. 

위의 두 클래스를 구현한 후, 화면 전환이 필요할 때 적절한 이벤트를 처리하고 마지막에 RouteDelegate의 notifyListeners() 함수를 호출하면 됩니다.

# 15장 - 네트워크 프로그래밍

## 15-1 JSON 파싱하기

플러터는 자바스크립트와 같이 JSON과 모델 객체의 자동 매핑을 지원하지 않습니다.

이는 빌드 시점에 사용하지 않는 코드를 제거하여 앱의 크기를 최적화하는 Tree Shaking과 관련있는데, 자동 매핑을 지원하려면 런타임 시점에 타입 분석을 하는 Reflection을 지원해야 하지만 Reflection이 Tree Shaking을 방해하기 때문입니다.

그러나 json_serializable 패키지를 이용하면 좀 더 편리하게 매핑할 수 있습니다. json_serializable 패키지는 JSON 매핑 코드를 자동으로 만들어주는 역할을 합니다. 

이 패키지를 사용하는 방법은 pub.dev의 공식 문서를 참조하면 그렇게 어렵지 않게 적용할 수 있습니다.