---
title: "스프링 입문(김영한) - Spring Bean과 Dependency(3)"
categories:
  - Java/Spring
tags:
  - Java
  - Spring
---

> 해당 글은 기존 Velog로 부터 새롭게 Github Blog로 이전하면서 복사 되었습니다.

# Spring Bean과 Dependency

## Spring Bean

- Spring IoC Container가 관리하는 JAVA 객체
- 단순히 `new` 연산자로 생성 한 객체가 아닌 `ApplicationContext.getBean()`으로 얻어질 수 있는 객체

## Component Scan과 자동 의존관계 설정을 통해 Spring Bean에 등록

- `@ComponentScan` Annotation과 `@Component` Annotation을 사용해서 Bean을 등록하는 방법
- SpringBoot 기반의 Application 제작시 기본적으로 `@SpringBootAplication` Annotation이 선언되는데, 해당 Annotation 내부에 `@ComponentScan`이 등록되어 있는 것을 확인할수 있음
- `@Component` 는 사용자가 생성한 객체를 Spring Bean에 자동으로 등록해주는 Annotation → `@Controller`, `@Service`, `@Repository`은 모두 내부에 `@Component` 가 등록되어 있음
- Spring Container에 Spring Bean을 등록할 때, 기본적으로 Singleton(싱글톤)패턴으로 등록함(유일하게 하나만 등록해서 공유) → 같은 Spring Bean이면 모두 같은 Instance
- 객체 의존관계를 외부에서 넣어주는 것을 DI(Dependency Injection)라 함 → DI에는 필드 주입, setter 주입, 생성자 주입 이렇게 3가지 방법이 존재 → 의존관계가 실행중에 동적으로 변하는 경우는 거의 없으므로 생성자 주입을 권장
- 생성자에 `@Autowired` 를 사용하면 객체 생성 시점에 Spring Container에서 해당 Spring Bean을 찾아서 주입함(생성자가 1개만 있으면 `@Autowired` 는 생략 가능)
    - MemberController 추가 및 Spring bean에 등록
        
        ```java
        package com.example.demo.controller;
        
        @Controller
        public class MemberController {
        
            private final MemberService memberService;
        
            @Autowired
            //@Autowired -> Spring 이 연관된 객체를 Spring Container 에서 찾아서 넣어줌
        
        		// 아래는 DI 생성자 주입 방법
            public MemberController(MemberService memberService) {
                this.memberService = memberService;
            }
        }
        ```
        
    - MemberService를 Spring bean에 등록
        
        ```java
        @Service
        public class MemberService {
            private final MemberRepository memberRepository;
            @Autowired
            public MemberService(MemberRepository memberRepository) {
                this.memberRepository = memberRepository;
            }
        }
        ```
        
    - MemoryMemberRepository 구현체를 Spring bean에 등록
        
        ```java
        @Repository
        public class MemoryMemberRepository implements MemberRepository {}
        ```
        

## JAVA 코드로 직접 Spring Bean에 등록

- 실무에서는 주로 정형화된 Controller, Service, Repository 같은 코드는 Component Scan을 사용 → 정형화 되지 않거나, 상황에 따라 구현 클래스를 변경해야 하면 Configuration을 통해 Spring Bean 으로 등록한다 → 해당 예제에선 DB가 선정되지 않았으므로 나중에 Repository를 변경할 때 용이
- Controller를 제외한 회원 Service와 회원 Repository의 `@Service`, `@Repository`, `@Autowired` Annotation을 제거하고 진행
    
    ```java
    package com.example.demo;
    
    @Configuration
    public class SpringConfig {
    
        @Bean
    		//DB 선정시 해당 부분만 바꿔주면 됨
        public MemberRepository memberRepository() {
            return new MemoryMemberRepository();
        }
    
        @Bean
        public MemberService memberService() {
            return new MemberService(memberRepository());
        }
    
    }
    ```
    
- XML로 설정하는 방식도 있지만 최근에는 잘 사용하지 않으므로 생략