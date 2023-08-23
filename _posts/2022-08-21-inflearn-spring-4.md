---
title: "Inflern 스프링 입문(김영한) - 회원 관리 예제 : 웹 MVC 개발(4)"
categories:
  - Inflern 스프링 입문(김영한)
tags:
  - [Java, Spring]

toc: true
toc_sticky : true

date : 2023-08-21
last_modified_at : 2023-08-21
---

> 해당 글은 기존 Velog로부터 새롭게 Github Blog로 이전되었습니다.

# 회원 관리 예제 : 웹 MVC 개발

## 홈 화면 추가

- HomeController 추가
    
    ```java
    package com.example.demo.controller;
    
    @Controller
    public class HomeController {
    
        @GetMapping("/") // "/" -> Default 루트
        public String Home() {
            return "home";
        }
    }
    ```
    
- resources/templates 폴더에 home.html 추가
    
    ```html
    <!DOCTYPE HTML>
    <html xmlns:th="http://www.thymeleaf.org">
    <body>
    <div class="container">
        <div>
            <h1>Spring Demo</h1> <p>회원 기능</p>
            <p>
                <a href="/members/new">회원 가입</a> 
    						<a href="/members">회원 목록</a>
            </p> </div>
    </div> <!-- /container -->
    </body>
    </html>
    ```
    

## 회원 등록

- resources/templates/members에 회원 등록 폼 createMemberForm.html 추가
    
    ```html
    <!DOCTYPE HTML>
    <html xmlns:th="http://www.thymeleaf.org">
    
    <body>
    <div class="container">
      <form action="/members/new" method="post">
        <div class="form-group">
          <label for="name">이름</label>
          <input type="text" id="name" name="name" placeholder="이름을
    입력하세요"> </div>
        <button type="submit">등록</button> </form>
    </div> <!-- /container -->
    </body>
    </html>
    ```
    
- MemberController에 회원 등록 기능 추가
    
    ```java
    @GetMapping("/members/new")
    public String createForm() {
        return "members/createMemberForm";
    }
    
    //회원 조회와 같은 기능은 Get 방식, 회원 등록과 같은 기능은 POST 방식
    @PostMapping("/members/new")
    public String create(MemberForm form){
        Member member = new Member();
        member.setName(form.getName());
    
        memberService.join(member);
    
        return "redirect:/";
    }
    ```
    
- 회원 등록 Form 객체 추가
    
    ```java
    package com.example.demo.controller;
    
    @Controller
    public class MemberForm {
        private String name;
    
        public String getName() {
            return name;
        }
    
        public void setName(String name) {
            this.name = name;
        }
    }
    ```
    

## 회원 조회

- resources/templates/members에 회원 리스트 memberList.html 추가
    
    ```html
    <!DOCTYPE HTML>
    <html xmlns:th="http://www.thymeleaf.org">
    <body>
    <div class="container">
        <div>
            <table>
                <thead>
    
                <tr>
                    <th>#</th>
                    <th>이름</th> </tr>
                </thead>
                <tbody>
    						<!-- 리스트를 돌면서 각 모델 안의 데이터를 가져옴 -->
                <tr th:each="member : ${members}">
                    <td th:text="${member.id}"></td>
                    <td th:text="${member.name}"></td>
                </tr>
                </tbody>
            </table>
        </div>
    </div> <!-- /container -->
    </body>
    </html>
    ```
    
- MemberController에 회원 리스트 조회 기능 추가
    
    ```java
    @GetMapping("/members")
        public String list(Model model){
            List<Member> members = memberService.findMembers();
            model.addAttribute("members",members);
            return "members/memberList";
        }
    ```