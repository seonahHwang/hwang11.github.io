---
title: Spring-boot DI 주입 방법
description: Spring-boot DI 주입 방법
categories:
 - Spring-boot
tags:  
---  

# DI주입

### 💡주입이란 ?

=> 내부에서 객체를 생성하는 것이 아니라 외부에서 객체를 생성하여 넣어주는 것

### 💡DI 주입 방법

- @Autowired
- 생성자 주입
- Setter 주입
- Lombok 라이브러리

### 💡생성자 주입 예시
```java
@RequiredArgsConstructor
public class SearchController {
    private final SearchService searchService;
    private final CombineSearchService combineSearchService;
    private final MovieSortService movieSortService;
    private final SearchProperties searchProperties;
    //..
}
```


```@RequiredArgsConstructor```를 이용하면 ```final```이나 ```@NonNull```인 필드 값만 파라미터로 받는 생성자를 만들어준다.

이를 사용하면 멤버변수가 수정되는 경우에 생성자까지 수정할 필요도 없어진다
