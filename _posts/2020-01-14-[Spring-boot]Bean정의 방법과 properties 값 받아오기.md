---
title: 스프링부트 스터디 STEP1 해결과정
description: STEP1 해결과정 - Bean정의 방법과 properties 값 받아오기
categories:
 - Spring-boot
tags:
---   
스프링에서 프로퍼티 값을 가져오는 방법은 여러가지가 있다.  
이전에 사용해본 것은 @value 어노테이션을 사용해서 값을 가져오는 것이었고  
이번에 시도해본 것은 @EnableConfigurationProperties 와 @ConfigurationProperties 를 이용한 프로퍼티 값 가져오기 였다.


#### 1. @EnableConfigurationProperties 와 @ConfigurationProperties 를 이용한 프로퍼티 값 가져오기

**@EnableConfigurationProperties**  
=> 해당 properties class로 object를 생성하겠다는 선언

**@ConfigurationProperties**  
=> properties 파일에 설정한 property 값을 java object에 매핑하여 java 코드에서 해당 값을 사용하기 편하게 해줌

​

application.properties

```Java
naver.openapi.search-url=https://openapi.naver.com/v1/search/blog.json
naver.openapi.clientId=-
naver.openapi.clientSecret=-
```
properties 파일에서는 카멜표기법이 아니라 중간에 하이픈으로  표기한다.


BlogProperties.java
```Java
@Getter
@Setter
@ConfigurationProperties(prefix = "naver.openapi")
public class BlogProperties {
    private String searchUrl;
    private String clientId;
    private String clientSecret;
}
```
<br>  

ApiConfiguration.java
```Java
@Configuration
@EnableConfigurationProperties({BlogProperties.class})
public class ApiConfiguration {
    @Bean
    public RestTemplate restTemplate(){
        return new RestTemplate();
    }
}
```
<br>

SearchService.java
```Java
@Service
public class SearchService {
    RestTemplate restTemplate;
    BlogProperties blogProperties;
    SearchService(BlogProperties blogProperties, RestTemplate restTemplate){
        this.blogProperties = blogProperties;
        this.restTemplate = restTemplate;
    }

    public ResponseEntity<SearchResponse> Search(String text){
        HttpHeaders httpHeaders = new HttpHeaders();
        httpHeaders.add("X-Naver-Client-Id", blogProperties.getClientId());
        httpHeaders.add("X-Naver-Client-Secret", blogProperties.getClientSecret());
        String url = blogProperties.getSearchUrl() + "?query=" + text + "?display=" + 10;

        return restTemplate.exchange(url, HttpMethod.GET, new HttpEntity(httpHeaders), SearchResponse.class);
    }
}
```


application.properties에 정의한 값을 쉽게 사용하기위해 BlogProperties 클래스를 만들어서 그 BlogProperties에 접근하여 값을 가져오도록 만들었다.

DI를 주입할 때 @Autowired를 지양하자고 하셨기 때문에 생성자를 이용해서 주입했다. 같은 방법으로  restTemplate도 Bean객체로 만들고 주입했다.


#### 2. Bean 정의 방법

1. @Bean 과 @Configuration을 이용한 방법

2. @ComponentScan과 각 컴포넌트 어노테이션을 이용한 방법(ex) @Service, @Controller)

#### 3. "local" profile 로 애플리케이션 실행

기존 application.properties 이외에 application-local.properties 파일을 별도로 정의하여 local환경일때 적용되는 설정을 정의해봤다.

현재는 혼자 공부중이라 다양한 properties값이 필요없지만 회사에서 근무하게 된다면 배포서버, 실서버 등 서버마다 적용되는 설정값이 다를 수 있기때문에 application.properties를 환경별로 정의하는 것 같다.

local profile로 실행되게 하기위해서 application.properties에서
spring.profiles.active=local
를 추가해주었다. application-local.properties에서 정의한 설정대로 돌아갈 뿐만아니라 application.properties에서 설정한 부분도 함께 적용된다.

​
#### 4. @Data 어노테이션

@Getter, @Setter, @RequiredArgsConstructor, @ToString, @EqualsAndHashCode을 한꺼번에 설정해주는 어노테이션이다.
이 스터디에서는 @Data 어노테이션 사용을 지양하기로했다.

#### 5. DI주입

DI 주입 방법은 여러가지가 있다.

- @Autowired
- 생성자 주입
- Setter 주입
- Lombok 라이브러리

스터디에서 @Autowired 사용을 지양하기로 해서 생성자를 통해서 DI를 주입했다.

예시코드
```Java
@Service
public class SearchService {
    RestTemplate restTemplate;
    BlogProperties blogProperties;
    SearchService(BlogProperties blogProperties, RestTemplate restTemplate){
        this.blogProperties = blogProperties;
        this.restTemplate = restTemplate;
    }
}
```


#### 리팩토링

* @BlogProperties를 bean으로 등록하지 않아도 돌아감   
=> @EnableConfigurationProperties 어노테이션이 @BlogProperties를 bean으로 등록해서 그런거였다.  

* blogername, blogerlink null 값    
=> 검색 API를 호출 했을 때 변수명이 일치하지 않아서 null값으로 처리된것이었다.  

* 컴포넌트를 빈으로 등록하기 위한 별도의 config class는 필요가 없다. @SpringBootApplication이  
스프링 부트의 자동 설정, 스프링 Bean 읽기와 생성을 모두 자동으로 설정하기 때문이다.  
