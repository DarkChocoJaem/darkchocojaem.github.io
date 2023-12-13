---
title: "Inflern 스프링 입문(김영한) - Project 환경설정 & 스프링 웹 개발 기초(1)"
categories:
  - Java/Spring
tags:
  - Inflern 스프링 입문(김영한)

toc: true
toc_sticky : true

date : 2023-08-21
last_modified_at : 2023-08-21
---

> 해당 글은 기존 Velog로부터 새롭게 Github Blog로 이전되었습니다.

Flutter를 사용하다 보니 API로 통신하여 데이터를 가져오는 일이 많았습니다. 

그러다가 문득 데이터가 구체적으로 어떤 방식으로 전송되고 생성되는지 궁금해져서 학교의 백엔드 스터디에 들어가 인프런(Inflearn)에서 한국의 백엔드로 가장 많이 자리잡은 Java/Spring 생태계를 맛 보기로 결정했습니다.

# Project 환경설정

## Spring Project 생성

- [https://start.spring.io](https://start.spring.io/)를 통해 간편하게 가능

## Build Tools

- Maven(3세대)
    - 사용할 라이브러리와 해당 라이브러리가 작동하는데 필요한 다른 라이브러리도 함께 자동으로 다운로드 하여 프로젝트를 생성
    - xml을 이용한 Build 시스템
- Gradle(4세대)
    - Build 자동화 및 여러 언어 개발 지원에 중점을 둔 Build 도구
    - Maven 보다 속도가 빠르고 가독성이 좋음
    - Groovy 혹은 Kotlin 문법 사용

## Package 구조

- main
    - 실질적으로 프로젝트의 실행되는 코드 내용이 들어있는 부분
- test
    - main에 있는 내용을 테스트 할 수 있는 환경 → TDD(Test Driven Development)
    - 기능을 개발한 후 잘 구현 되었는지 테스트 코드를 작성하고 코드 구조 재조정을 통해 성능을  향상
    - 원래는 개발한 기능을 테스트 할 때 자바의 main을 통해서 실행하거나, Web Application의 Controller를 통해서 해당 기능을 실행함 → 준비 및 실행까지 오래 걸리고, 여러 테스트를 한번에 실행하기 어려움 → JAVA는 JUnit이라는 Framework로 테스트를 지원
    - `@Test` Annotation을 명시

## Spring Boot 주요 라이브러리

- spring-boot-starter
    - Spring Boot +  Spring Core + Logging으로 구성 되며 라이브러리 명이 spring-boot-starter로 시작하면 공통적으로 필요로 함
- spring-boot-starter-test
    - Junit, Hamcrest, Mockito를 포함하여 Spring Application의 테스트를 가능하게 만들어 줌
- spring-boot-starter-web
    - Spring MVC를 사용한 RESTful서비스를 개발하는데 사용
- spring-boot-starter-thymeleaf
    - 타임리프 Template Engine (Web 상의 동적인 View 생성)

## View 생성

- Static
    - resources의 static 폴더 내부에 index.html 파일을 직접 로드하여 정적인 View 생성
- Dynamic
    - Template Engine을 이용하여 프로그래밍을 통해 동적인 View 생성
    - spring-boot-devtools → 서버 재시작 없이 View의 변경 사항을 reload 할 수 있음

## Build(Gradle) 하는 법

- Command를 통해 Spring Project 폴더에 접근 후 `./gradlew build` 실행→ build/libs 폴더에 jar 형식으로 저장 되어 있음 → `java -jar filename.jar` 형식으로  실행
- 오류 나면 `./gradlew clean build` 를 해볼 것

# 스프링 웹 개발 기초

## 정적(Static) 컨텐츠

- 서버를 통한 프로그래밍적 요소를 거치지 않고 파일을 그대로 브라우저로 내려주는 컨텐츠
- 만약 클라이언트가 요청하는 URL속 HTML 파일이 Controller와 Mapping 되어 있지 않을 경우 resources의 static 폴더 내부에서 파일을 찾고, 존재한다면 viewResolver가 브라우저로 내려줌

## MVC(Model-View-Controller)와 템플릿 엔진

- 서버를 통해 프로그래밍이 된 요소를 동적으로 브라우저에 내려주는 컨텐츠
- 이전에는 View에 Controller의 기능을 함께 코딩 했지만(Model1 방식), 프로그래밍에서 역할과 책임의 분리가 중요 해짐에 따라 현재는 View를 가시적인 부분에만 집중하기 위해 Controller과 분리
- Controller을 통해 데이터 및 로직을 처리하고 Model에 담아 View(viewResolver가 HTML로 변환)로 넘기는 방식 → 동적인 방식

## API(Application Programming Interface)

- JSON과 같은 특정 데이터 구조 포맷으로 클라이언트에게 데이터를 전달하는 방식
- Controller 작성시 `@ResponseBody` Annotation을 명시 → Http의 body부로 데이터를 직접 넣어줌(viewResolver 대신 HttpMessageConverter가 동작) → 반환할 데이터의 포맷이 단순 문자일 경우 String Converter가 동작하고 객체일 경우 기본적으로 Json Converter를 통해 JSON 형태로 변환하여 반환됨
- Jackson → JAVA에서 객체를 Json으로 바꿔주는 외부 라이브러리
- 클라이언트의 HTTP Accept Header에 따라 Controller에서 XML, JSON 등 어떤 Converter가 작동될지 정해짐