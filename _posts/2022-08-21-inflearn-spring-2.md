---
title: "Inflern 스프링 입문(김영한) - 회원 관리 예제 : Backend 개발(2)"
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

# 회원 관리 예제 : Backend 개발

## 비지니스 요구사항 정리

- 요구 사항
    - 데이터 → 회원 ID, 이름
    - 기능 → 회원 등록, 조회
    - 아직 데이터 저장소가 선정되지 않음 → 추후 변경이 가능하도록 Interface로 구현 클래스를 설계
    - 개발 진행을 위해 초기 개발 단계에서는 구현체로 가벼운 메모리 기반의 데이터 저장소 사용
- 일반적인 Web Application 계층 구조(참고 사항)
    - Controller → MVC의 Controller 역할
    - Service → 핵심 비지니스 로직
    - Repository → Domain 객체를 DB에 저장 및 관리
    - Domain → 객체로서 주문, 쿠폰, 회원 등의 정보를 담고 있음

## 회원 도메인과 Repository 만들기

- Member 객체 추가
    
    ```java
    package com.example.demo.domain;
    
    public class Member {
        private Long id;
        private String name;
    
        public Long getId() {
            return id;
        }
    
        public void setId(Long id) {
            this.id = id;
        }
    
        public String getName() {
            return name;
        }
    
        public void setName(String name) {
            this.name = name;
        }
    }
    ```
    
- MemberRepository 인터페이스 추가
    
    ```java
    package com.example.demo.repository;
    
    public interface MemberRepository {
    
        Member save(Member member);
    
        Optional<Member> findById(Long id);
    
        Optional<Member> findByName(String name);
    
        List<Member> findAll();
    
    }
    ```
    
- MemoryMemberRepository 구현체 추가
    
    ```java
    package com.example.demo.repository;
    
    public class MemoryMemberRepository implements MemberRepository{
        private static Map<Long, Member> store = new HashMap<>();
        private static long sequence = 0L; // Key 값 변수
        //위의 변수들은 실무에는 동시성 문제를 고려하여 Atomic으로 사용해야 함
    
        @Override
        public Member save(Member member) {
            member.setId(++sequence); //store에 넣기 전에 id 값을 1씩 올려줌
            store.put(member.getId(), member);
            return member;
        }
    
        @Override
        public Optional<Member> findById(Long id) {
            return Optional.ofNullable(store.get(id));
            //Optional을 통해 ID 조회 결과가 NUll 이어도 정상적으로 반환(NPE 방지)
        }
    
        @Override
        public Optional<Member> findByName(String name) {
            return store.values().stream()
                    .filter(member -> member.getName().equals(name))
                    .findAny();
            //store의 value 값을 돌면서 name과 같은 value를 가진 객체를 return
        }
    
        @Override
        public List<Member> findAll() {
            return new ArrayList<>(store.values());
        }
    
        public void clearStore(){
            store.clear();
        }
    }
    ```
    

## 회원 Repository Test Case 작성

- MemoryMemberRepository 구현체 테스트
    
    ```java
    package com.example.demo.repository;
    
    class MemoryMemberRepositoryTest {
        MemoryMemberRepository repository = new MemoryMemberRepository();
    
        @AfterEach
        public void afterEach(){
            repository.clearStore();
            /*
                Test를 진행할 때 각 함수는 무순서로 실행되므로 store에 저장되어 있는
                서로 다른 객체가 반환되어 오류가 남
                EX: findAll() -> save() findAll 함수에서 만든 member 객체가
                save 함수에서 findByName으로 반환됨
                따라서 @AfterEach를 통해 각 함수를 테스트할 때 마다 store을 비워 줘야 함
            */
        }
    
        @Test
        public void save() {
            Member member = new Member();
            member.setName("user");
            repository.save(member);
    
            Member result = repository.findByName("user").get();
            Assertions.assertEquals(result, member);
        }
    
        @Test
        public void findAll() {
            Member member = new Member();
            member.setName("user");
            repository.save(member);
    
            List<Member> result = repository.findAll();
    
            Assertions.assertEquals(result.size(), 1);
        }
    
    }
    ```
    

## 회원 Service 개발

- MemberService 추가
    
    ```java
    package com.example.demo.service;
    
    public class MemberService {
        private final MemberRepository memberRepository = new MemoryMemberRepository();
    
        /**
         * 회원 가입
         */
        public Long join(Member member){
            validateDuplicateMember(member);  //중복이름 회원 검증
    
            memberRepository.save(member);
            return member.getId();
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
            return memberRepository.findAll();
        }
    
        public  Optional<Member> findOne(Long memberId){
            return memberRepository.findById(memberId);
        }
    
    }
    ```
    

## 회원 Service 테스트

- MemberService 테스트
    
    ```java
    package com.example.demo.service;
    
    //static import
    import static org.junit.jupiter.api.Assertions.assertEquals;
    import static org.junit.jupiter.api.Assertions.assertThrows;
    
    class MemberServiceTest {
    
        //Test는 given, when, then 으로 나눠 작성하는 것이 효율적
        //MAC OS의 Test Class 생성 단축키 -> command + shift + t
    
        MemberService memberService;
        MemoryMemberRepository memberRepository;
    
        @BeforeEach
        public void beforeEach() {
    
            memberRepository = new MemoryMemberRepository();
            memberService = new MemberService(memberRepository);
            // MemberService 와  MemberServiceTest 에서 MemoryMemberRepository
            // 객체를 따로 생성해 사용하면 서로 다른 객체를 참조하게 되므로
            // MemoryMemberRepository 객체를 테스트 전 매번 만들어 넘겨줌
            // -> DI(Dependency Injection)
    
        }
    
        @AfterEach
        public void afterEach() {
            memberRepository.clearStore();
        }
    
        @Test
        void join() {
    
            //given
            Member member = new Member();
            member.setName("user");
    
            //when
            Long savedId = memberService.join(member);
            Member findMember = memberService.findOne(savedId).get();
    
            //then
            assertEquals(member.getName(), findMember.getName());
    
        }
    
        @Test
        void duplicate_member_join() {
    
            //given
            Member member1 = new Member();
            member1.setName("user");
    
            Member member2 = new Member();
            member2.setName("user");
    
            //when
            memberService.join(member1);
    
           //then
            assertThrows(IllegalStateException.class, 
                    () -> memberService.join(member2));
    
        }
    
    }
    ```