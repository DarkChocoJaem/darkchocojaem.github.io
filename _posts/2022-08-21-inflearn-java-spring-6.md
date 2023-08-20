---
title: "스프링 입문(김영한) - AOP : Aspect Oriented Programming(6)"
categories:
  - Java/Spring
tags:
  - Java
  - Spring
---

> 해당 글은 기존 Velog로 부터 새롭게 Github Blog로 이전하면서 복사 되었습니다.

# AOP : Aspect Oriented Programming

## AOP가 필요한 상황

- 모든 메소드의 호출 시간을 측정하고 싶을 때 → Ex: 회원 가입 시간, 회원 조회 시간을 측정하고 싶을 때
- 공통 관심 사항과 핵심 관심 사항을 분리
- MemberService 회원 조회 시간 측정 추가(일반적인 방식)
    
    ```java
    package com.example.demo.service;
    
    @Transactional
    public class MemberService {
        private final MemberRepository memberRepository;
    
        public MemberService(MemberRepository memberRepository) {
            this.memberRepository = memberRepository;
        }
    
        /**
         * 회원 가입
         */
        public Long join(Member member){
    
            long start = System.currentTimeMillis();
    
            try {
    
                validateDuplicateMember(member);  //중복이름 회원 검증
                //Mac OS Method 추출 단축키 -> option + command + v
    
                memberRepository.save(member);
                return member.getId();
    
            } finally {
    
                long finish = System.currentTimeMillis();
                long timeMs = finish - start;
                System.out.println("timeMs = " + timeMs + "ms");
    
            }
        }
    
        private void validateDuplicateMember(Member member) {
            /*
             isPresent() 메소드
             - Boolean 타입
             - Optional 객체가 값을 가지고 있다면 true, 값이 없다면 false 리턴
             */
            memberRepository.findByName(member.getName()).ifPresent(m -> {
                throw new IllegalStateException("이미 존재하는 이름입니다.");
            });
        }
    
        /**
         * 전체 회원 조회
         */
        public List<Member> findMembers(){
    
            long start = System.currentTimeMillis();
    
            try {
    
                return memberRepository.findAll();
    
            } finally {
    
                long finish = System.currentTimeMillis();
                long timeMs = finish - start;
                System.out.println("findMembers = " + timeMs + "ms");
    
            }
    
        }
    
        public  Optional<Member> findOne(Long memberId){
            return memberRepository.findById(memberId);
        }
    
    }
    ```
    
    - 문제점
        - 회원가입, 회원 조회에 시간을 측정하는 기능은 핵심 관심 사항이 아님
        - 시간을 측정하는 로직은 공통 관심 사항
        - 시간을 측정하는 로직과 핵심 비즈니스의 로직이 섞여서 유지보수가 어려움
        - 시간을 측정하는 로직을 별도의 공통 로직으로 만들기 매우 어려움
        - 시간을 측정하는 로직을 변경할 때 모든 로직을 찾아가면서 변경해야 함

## AOP 적용

- 시간 측정 AOP 등록
    
    ```java
    package com.example.demo.aop;
    
    @Component //Spring bean으로 등록
    @Aspect //AOP Annotation
    public class TimeTraceAop {
    
        @Around("execution(* com.example.demo..*(..))")
        //공통 관심사항 -> com.example.demo 해당 패키지 내부 모든 클래스에 적용
    
    		//joinPoint -> 다음 메서드 실행 전 발생하는 Intercept Point
        public Object execute(ProceedingJoinPoint joinPoint) throws Throwable {
    
            long start = System.currentTimeMillis();
    
            System.out.println("START: " + joinPoint.toString());
    
            try {
    
                return joinPoint.proceed(); //다음 메소드 진행
    
            } finally {
    
                long finish = System.currentTimeMillis();
                long timeMs = finish - start;
                System.out.println("END: " + joinPoint + " " + timeMs + "ms");
    
            }
    
        }
    }
    ```
    
    - 해결
        - 회원가입, 회원 조회등 핵심 관심사항과 시간을 측정하는 공통 관심 사항을 분리
        - 시간을 측정하는 로직을 별도의 공통 로직으로 제작
        - 핵심 관심 사항을 깔끔하게 유지
        - 변경이 필요하면 해당 로직만 변경하면 됨
        - 원하는 적용 대상 선택 가능
- AOP 동작 원리
    - Controller에서 Service를 호출하면 Spring에서 프록시(가상)Service를 만들고 호출하며, `joinPoint.proceed()` 가 발생하면 실제 Service를 호출

# 다음으로

- 지금까지 스프링으로 웹 애플리케이션을 개발하는 방법에 대해서 얇고 넓게 학습 했으며 이제부터는 각각의 기술들을 깊이있게 이해해야 함
- 거대한 스프링의 모든 것을 세세하게 알 필요는 없음
- 우리는 스프링을 만드는 개발자가 아니므로 스프링을 활용해서 실무에서 발생하는 문제들을 잘 해결하는 것이 훨씬 중요
- 따라서 핵심 원리를 이해하고, 문제가 발생했을 때 대략 어디쯤 부터 찾아들어가면 될지, 필요한 부분을 찾아서 사용할 수 있는 능력이 더 중요