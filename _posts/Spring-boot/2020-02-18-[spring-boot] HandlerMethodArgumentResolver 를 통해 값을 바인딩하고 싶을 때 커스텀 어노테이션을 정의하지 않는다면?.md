---
title: HandlerMethodArgumentResolver
description:  HandlerMethodArgumentResolver 를 통해 값을 바인딩하고 싶을 때 커스텀 어노테이션을 정의하지 않는다면?
categories:
 - Spring-boot
tags:
---  

## HandlerMethodArgumentResolver 사용 단계  
HandlerMethodArgumentResolver 를 통해 값을 바인딩하고 싶을 때(객체를 Controller 파라미터에 바인딩하기) 해야하는 단계는

1. 커스텀 어노테이션 정의

2. HandlerMethodArgumentResolver의 구현체 만들기

3. 구현체에서 supportsParameter() & resolveArgument() 오버라이드

4. 작성한 구현체를 스프링에 등록

   WebMvcConfigurer의 구현체에 addArgumentResolvers를 오버라이드해서 등록한다

​

## 궁금했던 점  

커스컴 어노테이션을 구현하지 않아도 리졸버가 대상 클래스를 인식하는데 문제가 없고, 따라서 파라미터로 쓰이는 객체에 값을 바인딩하는데 문제가 없다고 생각했다. 그래서 커스텀 어노테이션이 왜 필요하지? 라는 의문이 들었다.

​

대답은 커스텀 어노테이션을 구현하지 않아도 값이 바인딩 되는데에는 문제가 없지만,

파라미터로 쓰이는 값을 리졸버에서 바인딩하는 게 아니라 다른 메서드를 통해서 변경하고 싶을 때 변경할 수 있는 선택지가 줄어드는 것이다.

​
## 예제
예제로 이해를 해보면,
```java
Controller {
   public void a (LoginUser loginUser) {
       loginUser.updateLoginTime();
       service.save(loginUser);
   }
}

Service {
       save (LoginUser loginUser) {
          ~~~
      }
}
```

loginUser에 리졸버를 통해 값을 받게되면, a()에서 user의 login 시간을 업데이트 하고 싶을때 업데이트 한 값이 저장되는 것이 아니라,

save()를 호출할 때 다시 리졸버를 통해서 값이 바인딩 되므로, a()메소드에서 의도한 값이 담기지 않는다.

​

그러나, @loginUser라는 커스텀어노테이션을 붙이게되면, 해당 어노테이션이 붙을때만 값을 리졸버를 통해서 받게된다.

​
## 커스텀 어노테이션 유무의 차이점  
따라서

1. 리졸버를 통한 값 할당

2. 호출하는 메서드에서 넣어준 값 할당

2가지 방법으로 파라미터의 값을 할당할 수 있는데, 커스텀어노테이션 없이는 1번의 선택지만 가지게 되는 것이다.


<br>
<br>

<br>



​(직접 jojoldu님의 springboot-webservice 리파지토리에 질문을 남겼다.)

참고 : https://github.com/jojoldu/freelec-springboot2-webservice/issues/252
