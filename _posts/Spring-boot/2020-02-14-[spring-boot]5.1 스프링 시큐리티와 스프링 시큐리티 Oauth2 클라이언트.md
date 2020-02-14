---
title: 5장 스프링 시큐리티와 OAuth 2.0으로 로그인 기능 구현하기 (1)
description: 5장 스프링 시큐리티와 OAuth 2.0으로 로그인 기능 구현하기 (1)
categories:
 - Spring-boot
tags:
---  
## 스프링 시큐리티  
**인증**과 **인가** 기능을 가진 프레임워크  
스프링 기반의 애플리케이션에서는 보안을 위한 표준  

### 스프링 부트 1.5 vs 스프링 부트 2.0  
spring-security-oauth2-autoconfigure 라이브러리를 사용할 경우 스프링 부트2에서도 1.5에서 쓰던 설정을 그대로 사용 가능  

### 구글로그인  
* 구글로그인 과정에서 승인된 리디렉션 URI는  
서비스에서 파라미터로 인증 정보를 주엇을 때 인증이 성공하면 구글에서 리다이렉트할 URK  
시큐리티에서 이미 구현해 놨으므로 사용자가 별도로 리다이렉트 URL을 지원하는 Controller를 만들 필요 없음  

* ```application-oauth.properties```에서    
```
spring.security.oauth2.client.registration.google.scope=profile,email
```   
설정한 이유는 별도로 scope를 등록하지 않으면 기본값인 openid,profile,email이 등록된다  
openid라는 scope가 있으면 openidProvider로 인식하고, 그러면 openidProvider인 서비스(구글)과 그렇지 않은 서비스로 나눠서 각각 OAuth2Service를 만들어야한다  
따라서 하나의 OAuth2Service로 사용하기 위해 일부러 openid scope를 빼고 등록  

* applcation.properties에서 application-oauth.properties를 호출하기 위해 ```applcation.properties``` 에  
```
spring.profiles.include=oauth
```  
를 추가한다  

### 5.3 구글 로그인 연동하기  
* EnumType 설정  

  ```java
  @Enumerated(EnumType.STRING)  
  @Column(nullable = false)  
  private Role role;
  ```
  ```@Enumerated(EnumType.STRING)```  
  JPA로 데이터베이스 저장할 때 Enum 값을 어떤 형태로 저장할지 결정  
  기본적으로 int로 저자오디지만, 숫자로 저장될시 DB로 확인할 때 그 값이 어떤 코드인지 알수 없어서 문자열로 저장될 수 있도록 함   


* 권한 대상 지정 등 관리 설정  
 ```java
@RequiredArgsConstructor
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    private final CustomOAuth2UserService customOAuth2UserService;

    @Override
    protected void configure(HttpSecurity http) throws Exception{
        http
                .csrf().disable()
                .headers().frameOptions().disable()
                .and()
                    .authorizeRequests()
                .antMatchers("/","/css/**","/images/**","/js/**","/h2-console/**").permitAll() //permitAll은 전체 열람 권한을 주는 것  
                .antMatchers("/api/v1/**").hasRole(Role.USER.name())
                .anyRequest().authenticated()
                .and()
                    .logout()
                        .logoutSuccessUrl("/")
                .and()
                    .oauth2Login()
                        .userInfoEndpoint()
                            .userService(customOAuth2UserService);
    }
}

 ```

1. ```@EnableWebSecurity```  
spring security 설정들을 활성화  

2. ```authorizeRequests```  
URL 별 권한 관리 설정하는 옵션의 시작점  
이게 선언 되어야먄 antMatchers 옵션 사용 가능  

3. ```anyMatchers```  
권한 관리 대상 지정 옵션  
URL, HTTP 메소드별 관리 가능  

4. ```anyRequest```  
설정된 값들 이외 나머지 URL  
authenticated()를 추가하여 나머지 URL들은 모두 인증된 사용자들에게만 허용  

5. ```oauth2Login```  
OAuth 2 로그인 기능에 대한 여러 설정의 진입점  

6. ```userService```    
소셜 로그인 성공 시 후속 조치를 할 진행할 UserService 인터페이스의 구현체를 등록  
리소스 서버에서 사용자 정보를 가져온 상태에서 추가로 진행하고자 하는 기능 명시할 수 있음  


<br>
<br>

* 구글 로그인 이후 가져온 사용자 정보를 기반으로 가입 및 정보수정, 세션 저장 등의 기능 지원  

  ```java
  @RequiredArgsConstructor
  @Service
  public class CustomOAuth2UserService implements OAuth2UserService<OAuth2UserRequest, OAuth2User> {
      private final UserRepository userRepository;
      private final HttpSession httpSession;

      @Override
      public OAuth2User loadUser(OAuth2UserRequest userRequest)
          throws OAuth2AuthenticationException {
          OAuth2UserService delegate = new DefaultOAuth2UserService();
          OAuth2User oAuth2User = delegate.loadUser(userRequest);

          String registrationId = userRequest.getClientRegistration().getRegistrationId();
          // 현재 로그인 진행 중인 서비스를 구분하는 코드. 이후 네이버 로그인 연동 시 네이버 로그인인지, 구글로그인인지 구분하기 위해 사용

          String userNameAttributeName = userRequest.
                  getClientRegistration().getProviderDetails()
                  .getUserInfoEndpoint().getUserNameAttributeName();
          /*OAuth2 로그인 진행 시 키가 되는 필드값. Primary Key와 같은 의미
          구글의 경우 기본적으로 코드를 지원하지만, 네이버 카카오 등은 기본 지원하지 않음. 구글의 기본 코드는 "sub"
          네이버 로그인과 구글 로그인을 동시 지원할 때 사용됨
          */
          OAuthAttributes attributes = OAuthAttributes.of(registrationId, userNameAttributeName,
                  oAuth2User.getAttributes());
          //OAuthAtrributes : OAuth2UserService를 통해 가져온 OAuth2User의 attribute를 담을 클래스
          //이후 네이버 등 다른 소셜 로그인도 이 클래스 사용

          User user = saveOrUpdate(attributes);

          httpSession.setAttribute("user", new SessionUser(user)); //세션에 사용자 정보 저장
          return new DefaultOAuth2User(
                  Collections.singleton(new SimpleGrantedAuthority(user.getRoleKey())),
                  attributes.getAttributes(),
                  attributes.getNameAttributeKey());

      }

      //구글 사용자 정보가 업데이트 되었을 때를 대비한 update
      private User saveOrUpdate(OAuthAttributes attributes){
          User user = userRepository.findByEmail(attributes.getEmail())
                  .map(entity -> entity.update(attributes.getName(),
                          attributes.getPicture()))
                  .orElse(attributes.toEntity());
          return userRepository.save(user);
      }
  }

  ```

<br>

* user 객체를 이용할 때 User 클래스말고 SessionUserDto 를 사용하는 이유  
```httpSession.setAttribute("user", new SessionUser(user));```   

  User 클래스를 session에 저장하려하면 직렬화가 되지 않았다는 오류가 발생한다  
  그렇다고 User 클래스에 직렬화를 구현하는 것이 답은 아니다  
  User 클래스는 DB와 맞닿아있는 Entity클래스이므로 자식 엔티티를 갖고 있다면 직렬화 대상에 자식들도 포함되어 성능 이슈, 부수 효과가 발생할 확률이 높다.  
  때문에 <u>직렬화 기능을 가진 세션 Dto를 이용하는 것</u>이 운영 및 유지보수에 도움이 된다  

<br>

* 로그인 테스트  
```index.mustache```  

  ```java  
  {{#userName}}
  Logged in as : <span id="user">{{userName}}</span>
  <a href="/logout" class="btn btn-info active"
  role="button">Logout</a>
  {{/userName}}

  {{^userName}}
  <a href="/oauth2/authorization/google"
  class="btn btn-success active" role="button">Google login</a>
  <a href="/oauth2/authorization/naver"
  class="btn btn-secondary active" role="button">Naver login</a>
  {{/userName}}

  ```  
  1. ```{{#userName}}```  
  머스테치는 if문을 제공하지 않고 true/false 여부만 판단  
  따라서 항상 머스테치에는 최종값을 넘겨줘야한다  
  ```{{^userName}}```  
  해당값이 존재하지 않을 때는 ^ 사용    

  2. ```a href="/logout"```  
  스프링 시큐리티에서 기본적으로 제공하는 로그아웃 URL. 따로 logout 컨트롤러를 만들필요가 없음  
