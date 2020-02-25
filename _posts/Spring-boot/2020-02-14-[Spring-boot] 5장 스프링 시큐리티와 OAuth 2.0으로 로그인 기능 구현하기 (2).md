---
title: 5장 스프링 시큐리티와 OAuth 2.0으로 로그인 기능 구현하기 (2)
description: 5장 스프링 시큐리티와 OAuth 2.0으로 로그인 기능 구현하기 (2)
categories:
 - Spring-boot
tags:
---  
## 5.4 어노테이션 기반으로 개선하기  
https://hwang11.github.io/spring-boot/2020/02/14/spring-boot-HandlerMethodArgumentResolver/  

## 5.5 세션 저장소로 데이터베이스 사용하기  
현재 서비스는 애플리케이션을 재실행하면 로그인이 풀림  
이는 세션이 내장 톰캣의 메모리에 저장되기 때문  
기본적으로 세션은 실행되는 WAS의 메모리에서 저장되고 호출됨  
메모리에 저장되니 내장 톰캣처럼 애플리케이션 실행 시 실행되는 구조에선 항상 초기화됨  
즉, 배포할 때마다 톰캣이 재시작 되는 것  
만약 2대 이상의 서버에서 서비스하고 있다면 톰캣마다 세션 동괴화 설정을 해야함  

#### 현업의 세션 저장소 방법  
1) 톰캣 세션 사용  
톰캣들 간의 세션 공유를 위한 추가 설정 필요  

2) MySQL과 같은 데이터베이스를 세션 저장소로 사용  
로그인 요청마다 DB IO가 발생하여 성능상 이유 발생가능성  

3) Redis, Memecached와 같은 메모리 DB를 세션 저장소로 사용  
B2C 서비스에서 가장 많이 사용하는 방식  
실제 서비스로 사용하기 위해선 Embedded Redis와 같은 방식이 아닌 외부 메모리 서버 필요  


## 5.7 기존 테스트에 시큐리티 적용하기  
기존 테스트에 시큐리티 적용으로 문제가 되는 부분들 해결하기    
시큐리티를 사용하면 인증된 사용자만 API 호출가능  

src/main 환경과 src/test 환경의 차이가 있음  
둘은 각자의 환경 구성을 가짐    
다만 테스트 코드 수행할 때도 src/main의 ```application.properties```가 적용되는 이유는 test에 ```application.properties```가 없으면 main의 설정을 그대로 가져오기 때문  
그러나 자동으로 가져오는 범위는 ```application.properties```일 뿐 ```application-oauth.properties``` 는 가져오지 못함  

API test에서 사용자에게 권한이 있는 것 처럼 하기위해서  
인증된 모의 사용자를 만들어서 사용  
=> ```@WithMockUser(roles="USER")```를 사용  

그러나 ```@WithMockUser```는 ```MockMvc```에서만 작동하므로 API 테스트가 ```@SpringBootTest```로만 되어있으면 사용이 불가함  
따라서 ```@SpringBootTest```에서 ```MockMvc```를 사용할 수 있도록
```@Before```을 이용해 매번 테스트 시작전 ```MockMvc``` 인스턴스를 생성  

```mvc.perform```  
생성된 ```MockMvc```를 통해 API 테스트  
#### 문제 : @WebMvcTest에서 CustomOauth2UserService를 찾을 수 없음   
이유는 ```@WebMvcTest```가 ```CustomOauth2UserService```를 스캔하지 않기 때문에  
```@WebMvcTest```는 ```@ControllerAdvice```, ```@Controller```는 읽고, ```@Repository```, ```@Service```, ```@Component```는 읽지 않음  
 따라서, ```SecurityConfig```는 읽었지만, ```SecurityConfig```를 생성하기 위한 ```CustomOauth2UserService```을 읽을 수 없어 에러가 발생  
해결 : 스캔 대상에서 ```SecurityConfig``` 제거  

```java
@WebMvcTest(controllers = HelloController.class,
excludeFilters = {
       @ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE,classes = SecurityConfig.class)
})
```
#### 문제 : @EnableJpaAuditing 에러  
 ```@EnableJpaAuditing```를 사용하기 위해서는 Entity클래스가 필요한데,
 ```@WebMvcTest```는 JPA 사용불가이므로 Entity클래스 없음  
 ```@EnableJpaAuditing```이 ```@SpringBootApplication```와 함께 있어서 ```@WebMvcTest```에서도 스캔하게 되었음.  
 **해결** : ```@EnableJpaAuditing```과 ```@SpringBootApplication``` 분리  
 그리고 config 패키지에 ```JpaConfig``` 별도 생성하여 ```@EnableJpaAuditing```설정  

#### @SpringBootTest vs @WebMvcTest  
@WebMvcTest  
@Controller @ControllerAdvice만 읽음  
web에 집중할 수 있는 어노테이션  
JPA 사용불가  
