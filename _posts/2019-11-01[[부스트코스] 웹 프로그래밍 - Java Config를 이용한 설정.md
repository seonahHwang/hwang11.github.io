# Java config를 이용한 설정을 위한 어노테이션

### @Configuration

스프링 설정 클래스를 선언하는 어노테이션

### @Bean

bean을 정의하는 어노테이션

### @ComponentScan

@Controller, @Service, @Repository, @Component 어노테이션이 붙은 클래스를 찾아 컨테이너에 등록

### @Component

컴포넌트 스캔의 대상이 되는 애노테이션 중 하나로써 주로 유틸, 기타 지원 클래스에 붙이는 어노테이션

### @Autowired

주입 대상이되는 bean을 컨테이너에 찾아 주입하는 어노테이션

​

​

@Configuration : 나는 config에요

@ComponentScan : 알아서 니가 여기에 있는거 읽어들여서 지정해라. 어노테이션 붙어있는 것들을 알아서 찾아서 지정해라

이런 @ComponentScan을 수행하게 할 때는 반드시 패키지명을 알려줘야함

@ComponentScan어노테이션은 파라미터로 들어온 패키지 이하에서 @Controller, @Service, @Repository, @Component 어노테이션이 붙어 있는 클래스를 찾아 메모리에 몽땅 올려줍니다.

​

### @Autowired

@Autowired가 알아서 해주기 때문에 굳이 setter메서드는 없어도 괜춘

@Autowired를 붙이면 여기서 사용되는 애를 알아서 생성해라 이런말인듯

​

Spring에서 사용하기에 알맞게 @Controller, @Service, @Repository, @Component 어노테이션이 붙어 있는 객체들은 ComponentScan을 이용해서 읽어들여 메모리에 올리고 DI를 주입하도록 하고, 이러한 어노테이션이 붙어 있지 않은 객체는 @Bean어노테이션을 이용하여 직접 생성해주는 방식으로 클래스들을 관리하면 편리합니다.

​

### @Bean

설정 파일 중에 bean이라는 이런 어노테이션이 붙어 있는 메서드들을

AnnotationConfigApplicationContext는 자동으로 실행해서 그 결과로 리턴하는 객체들을 기본적으로 싱글턴으로 관리를 하게 해줘요.

​

@Bean 사용
```java


public class ApplicationConfig {

@Bean

public Car car(Engine e) {

Car c = new Car();

c.setEngine(e);

return c;

}

@Bean

public Engine engine() {

return new Engine();

}

}
​
```

@Component와 @Autowired  사용​
```Java
@Component

public class Car {

@Autowired //Engine 객체를 알아서 생성해서 v8에 넣어줘.

//@Autowired가 있기 때문에 굳이 setter메소드는 필요가 없다.

private Engine v8;

public Car() {

System.out.println("Car 생성자");

}

public void run() {

System.out.println("엔진을 이용하여 달립니다.");

v8.exec();

}

}

```
