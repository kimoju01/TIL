# [Spring Security] Spring Security란?


## ✔ Spring Security 용어
- **Principal(접근 주체)**: 보호된 리소스에 접근하는 사용자
- **Authentication(인증)**: 해당 사용자(user, principal)가 본인이 맞는지
- **Authorization(인가)**: 인증된 사용자가 요청한 리소스에 접근할 수 있는 **권한**을 부여
  - 인증 과정을 먼저 거치고 주체가 증명 된 이후 권한을 부여할 수 있음 (인증 -> 인가)
- **Credential(패스워드, 증명서)**: 주체가 본인임을 인증하기 위해 서버에 제공하는 것

## ✔ Spring Security 흐름도
<img src="https://chathurangat.files.wordpress.com/2017/08/blogpost-spring-security-architecture.png" width="100%" height="60%" alt="Spring-Security-Flow-iagram"></img><br/>

## ✔ Spring Security 구조
> ###### 흐름도 숫자랑 완전히 같진 않음
- #### 1. 유저가 로그인화면에서 아이디, 비밀번호를 입력하여 로그인을 시도함
- #### 2. 서버에 로그인 인증 요청이 오면 ```SpringSecurityFilterChain```이 동작한다.
  - 이 필터 객체는 여러 필터 리스트를 가지고 있으며 ```doFilter``` 메서드들이 리스트를 순회하며 필터링을 실시하며, 이 필터 리스트가 ```AuthenticationFilter``` 들이다.
- #### 3. form 기반 로그인 이라면 ```UsernamePasswordAuthenticationFilter``` 필터가 실행된다.
  - 사용자 인증 요청을 현재 접근하는 주체의 정보와 권한을 담는 ```Authentication``` 인터페이스로 추상화하고 ```AuthenticationManager```를 통한 인증을 실행한다.
  - 인증에 성공하면 ```Authentication``` 객체를 ```SecurityContext```에 저장하고 ```AuthenticationSuccessHandler```를 호출한다.
    - 이 때, ```SecurityContext```는 ```SecurityContextHolder```를 통해 접근하고, ```SecurityContext```를 통해 ```Authentication```에 접근할 수 있다.
  - 인증에 실패하면 ```AuthenticationFailureHandler```를 호출한다.
  - 인증이 성공했다면 리턴값인 ```UsernamePasswordAuthenticationToken```을 세션에 저장한다.
  - ```UsernamePasswordAuthenticationToken```은 ```Authentication``` 인터페이스의 구현체이므로 ```AuthenticationManager```에서 인증과정을 수행할 수 있다.
  - form 로그인이 아닌 OAuth 2.0을 이용한 인증 이라면 ```UsernamePasswordAuthenticationFilter```에서 인증되지 않으니 인증되지 않은 채로 다음 필터로 넘어간다. 
그 후, ```OAuth2ClientAuthenticationProcessingFilter``` 에서 OAuth2.0을 이용한 인증을 해준다.
  - 즉, 여러 필터를 거치면서 어떤 필터에서 인증이 완료되면 해당 요청은 인증되는 것이고, 모든 필터를 거치는 동안 인증에 실패하면 인증되지 않은 요청이 된다.
- #### 4. ```AuthenticationManager```가 인증을 위해 적절한 ```AuthenticationProvider```를 찾아 처리를 위임한다.
  - ```AuthenticationManager```는 인터페이스이며 구현체는 ```ProviderManager```이다. ```ProviderMagager```는 직접 인증 과정을 진행하지 않고
실제 인증을 처리하는 로직이 포함된 ```AuthenticationProvider```를 List로 가지고 있으며, for문을 통해 순회하며 ```AuthenticationProvider```들에게 인증(authenticate)을 위임한다.
  - ```AuthenticationProvider```는 실제 인증에 대한 부분을 처리하는데, 인증 전의 ```Authentication``` 객체를 받아서 인증이 완료된 ```Authentication``` 객체를 반환한다.
- #### 5. ```Authentication```객체로부터 인증에 필요한 정보를 받아오고, ```UserDetailsService```의 구현체로 조회할 아이디를 전달한다.
  - ```UserDetailsService```의 구현체는 아이디를 기반으로 회원 정보를 조회한다.
- #### 6. 전달받은 아이디를 기반으로 DB를 조회하여 비밀번호가 일치하는지 확인하여 인증을 처리한다.
  - ```UserDetailsService```의 반환값은 ```UserDetails``` 인터페이스이기 때문에 이를 구현한 구현체를 반환한다.
- #### 7. 인증 처리 후 인증된 토큰을 ```AuthenticationManager```에게 반환한다.
  - ```userDetailsService```를 통해 조회한 정보와 입력받은 비밀번호가 일치해 인증이 완료되면 ```UsernamePasswordAuthenticationToken```에 회원정보를 담아 리턴한다.
- #### 8. ```AuthenticationManager```에서 인증이 완료된 ```UsernamePasswordAuthenticationToken```을 ```AuthenticationFilter```로 반환한다.
  - ```AuthenticationFilter```는 ```UsernamePasswordAuthenticationToken```을 ```LoginSuccessHandler```로 전달한다.
- #### 9. ```LoginSuccessHandler```로 넘어온 ```Autentication``` 객체(```UsernamePasswordAuthenticationToken```)를 ```SecurityContextHolder```에 저장하면 인증 과정이 끝난다.
  - 그 이후 유저가 로그인에 성공하면 ```SecurityContextHolder```라는 세션을 활용하여 저장하기 때문에 한 번만 로그인 해도 로그인이 필요한 페이지들에 바로 접근이 가능해진다.
---
##### 참고한 글
###### [📃[Spring Security] 스프링시큐리티 기본개념과 동작구조의 이해(1)](https://kimchanjung.github.io/programming/2020/07/01/spring-security-01/)   
###### [📃로그인 과정으로 살펴보는 스프링 시큐리티 아키텍처(Spring Security Architecture)](https://jeong-pro.tistory.com/205)   
###### [📃[SpringBoot] Spring Security 처리 과정 및 구현 예제](https://mangkyu.tistory.com/77)   

