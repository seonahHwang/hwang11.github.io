---
title: HandlerMethodArgumentResolver
description:  HandlerMethodArgumentResolver
categories:
 - Spring-boot
tags:
---
# HandlerMethodArgumentResolver
컨트롤러 메서드에서 특정 조건에 맞는 파라미터가 있을 때 원하는 값을 바인딩해주는 인터페이스  
Ex) 어떤 값을 메서드인자로 사용할 때, 그 인자에 값을 넣어주는 것을 직접하는 게 아니라 resolver를 통해서 값이 들어와있는 상태로 사용하고 싶을 때라고 이해했다  


## HandlerMethodArgumentResolver 구현시 필요한 메서드 2개
### supportsParameter()
```boolean supportsParameter(MethodParameter var1)```
현재 파라미터를 resolver가 지원하는지에 대한 결과 리턴

### resolveArgument()
```object resolveArgument(MethodParameter var1, @Nullable ModelAndViewContainer var2, @Nullable NativeWebRequest var3, @Nullable WebDataBinderFactory var4)```  

실제로 바인딩 할 객체 리턴  
바인딩할 객체 조작, 리턴  

## 예시
```java
@RequiredArgsConstructor
@Component
public class LoginUserArgumentResolver implements HandlerMethodArgumentResolver {
    private final HttpSession httpSession;

    @Override
    public boolean supportsParameter(MethodParameter parameter){
        boolean isLoginUserAnnotation = parameter.getParameterAnnotation(LoginUser.class)!=null;
        boolean isUserClass = SessionUser.class.equals(parameter.getParameterType());
        return isLoginUserAnnotation && isUserClass;
    }

    @Override
    public Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer, NativeWebRequest webRequest,
                                  WebDataBinderFactory binderFactory) throws Exception{
        return httpSession.getAttribute("user");
    }
}

```
