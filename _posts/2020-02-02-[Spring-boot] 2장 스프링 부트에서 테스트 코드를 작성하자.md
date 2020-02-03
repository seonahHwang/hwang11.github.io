---
title: Spring-boot 2장 스프링부트에서 테스트 코드를 작성하자
description: Spring-boot 2장 스프링부트에서 테스트 코드를 작성하자
categories:
 - Spring-boot
tags:  
---  
```JAVA
@SpringBootApplication
public class Application { //메인클래스, 내장 WAS 실행
    public static void main(String[] args) {
        SpringApplication.run(Application.class,args);
    }
}

```
### **Application 클래스**  
* 메인 클래스  
* @SpringBootApplication 어노테이션으로 인해 스프링 부트의 자동 설정, 스프링 Bean 읽기와 생성을 모두 자동으로 설정  
* @SpringBootApplication이 있는 클래스 부터 읽어나가기 때문에 Application클래스는 프로젝트 상단에 위치해야한다    

### **내장 WAS**
* SpringApplication.run 으로 인해서 내장 톰캣이 실행됨  
* 언제 어디서나 같은 환경에서 배포할 수 있기 때문에 내장 WAS 권장
외장 WAS를 쓴다면 모든 서버는 WAS의 종류, 버전, 설정을 일치시켜야함     
* 톰캣도 자바 어플리케이션이므로 내장 WAS에 대한 성능상 이슈는 크게 고려할 필요 없음  

### **HelloControllerTest code**
```JAVA
@RunWith(SpringRunner.class)
@WebMvcTest
public class HelloControllerTest{
  @Autowired
  private MockMvc mvc;

  @Test
  public void Hello가_리턴된다() throws Exception{
    String hello = "hello";
    mvc.perform(get("/hello"))
        .andExpect(status().isOK))
        .andExpect(content().string(hello));

  }
}
```
1.  @RunWith(SpringRunner.class)  
* 테스트를 진행할 때 Junit에 내장된 실행자(runner)외에 다른 실행자를 실행시킴  
* 스프링 부트 테스트와 JUnit사이에 연결자 역할  

2. @WebMvcTest  
* 여러 스프링 어노테이션중에 Web(SpringMvc)에 집중할 수 있는 어노테이션  
* 선언할 경우 @Controller, @ControllerAdvice를 사용 가능. But @Service, @Repository, @Component 등은 사용할 수 없음  

3. @Autowired
* 스프링이 관리하는 Bean 주입 받는 방법 중 1개  

4. private MockMvc mvc  
* 웹 애플리케이션을 애플리케이션 서버에 배포하지 않고도 스프링 MVC의 동작을 재현할 수 있는 클래스  
* 웹 API 테스트할 때 사용가능  
* 스프링 MVC 테스트의 시작점   

5. mvc.perform(get/"hello")
* 체이닝이 지원되어 여러 검증 기능을 이어서 선언가능  

6. .andExpect(status().isOk())  
* mvc.perform의 결과검증  
* HTTP header의 Status가 200인지 검증  

7. .andExpect(content().string(hello))  
* mvc.perform의 결과검증  
* 응답 본문의 내용검증  

### **Lombok**  
1. @Getter : 선언된 모든 필드의 get 메소드를 생성  
2. @RequiredArgsConstructor : final 붙은 필드를 모두 포함한 생성자 생성  


### **HelloResponseDtoTest Code**
```JAVA
@Test
public void 롬복_기능_테스트(){
    //given
    String name = "test";
    int amount = 1000;

    //when
    HelloResponseDto dto = new HelloResponseDto(name,amount); //생성자를 통해서 값 넣기

    //then
    assertThat(dto.getName()).isEqualTo(name);
    assertThat(dto.getAmount()).isEqualTo(amount);
}

```  
1. assertThat
* assertj라는 테스트 검증 라이브러리의 검증 메소드  
* 검증하고싶은 대상을 메소드 인자로 받음  
* 메소드 체이닝이 지원되어 isEqualTo와 같이 메소드를 이어서 사용가능  

2. isEqualTo  
* assertj의 동등 비교 메소드  


### Junit과 비교한 assertj의 장점  
1. coreMatchers와 달리 추가적으로 라이브러리가 필요하지 X  
- Junit의 assertThat을 사용하면 is()와 같이 coreMatchers 라이브러리가 필요함  
2. 자동완성 확실하게 지원  
- IDE에서는 coreMatchers와 matchers의 자동완성 기능이 약함  

### HelloControllerTest with Params Code  
```JAVA    
@Test
    public void hellDto가_리턴된다() throws Exception{
        String name = "hello";
        int amount = 1000;
        mvc.perform(get ("/hello/dto")
                .param("name",name)
                .param("amount", String.valueOf(amount)))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.name", is(name)))
                .andExpect(jsonPath("$.amount", is(amount)));
    }

```  
1. param  
* API test할때 사용할 요청 파라미터를 사용하는 방법  
* String으로만 사용가능, 날짜/시간 등도 String으로 변경 필요  

2. jsonPath  
* Json응답 값을 필드별로 검증할 수 있는 메소드  
