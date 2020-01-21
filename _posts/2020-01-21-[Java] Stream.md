---
title: Java stream api
description: Java stream api
categories:
 - JAVA
tags:
---
# Stream  
* 컬렉션, 배열들의 저장 요소를 하나씩 참조하며 함수형 인터페이스(람다식)를 반복적으로 처리할 수 있도록 해주는 기능  
* 다양한 데이터 소스를 표준화된 방법으로 다루기 위한 것  

### 특징  
* 불필요한 코딩(for, if 문법)을 사용하지 않으면서 직관적임  
* 주로 Collection, Arrays에서 쓰임  
* Iterator 처럼 재사용 불가  
* 데이터 소스로부터 데이터를 읽기만할 뿐 변경하지 않는다.  
* 스트림은 작업을 내부 반복으로 처리한다 (for문을 메서드 안으로 넣어 실행)  


### 스트림의 구조  
1. 스트림 생성  
2. 중개 연산 : 연산결과가 스트림인 연산. 반복적으로 적용가능  
3. 최종 연산 : 연산결과가 스트림이 아닌 연산. 스트림의 요소를 소모하므로 한번만 적용가능  

![예시](https://user-images.githubusercontent.com/37287788/72809316-bbb63000-3c9e-11ea-9cff-a0551cbdc608.png)  
### 병렬 스트림  
parallelStream
한가지 작업을 서브 작업으로 나누고, 서브 작업들을 분리된 스레드에서 병렬적으로 처리하는것  



### 기본형 스트림  
IntStream, LongStream, DoubleStream

- 오토박싱 & 언박싱 비효율이 제거 됨 (stream<Integer> 대신 IntStream 사용)  
- 숫자와 관련된 유용한 메서드를  보다 더 많이 제공  




### 스트림의 중간연산    

#### peak()  

스트림을 소비하지 않고 엿보는 방법  

```java
fileStream.map(File::getName) // Stream<File> → Stream<String>  
 .filter(s -> s.indexOf('.')!=-1) // 확장자가 없는 것은 제외   
 .peek(s->System.out.printf("filename=%s%n", s)) // 파일명을 출력한다.
 .map(s -> s.substring(s.indexOf('.')+1)) // 확장자만 추출   
 .peek(s->System.out.printf("extension=%s%n", s)) // 확장자를 출력한다.  
.forEach(System.out::println); // 최종연산 스트림을 소비.
```

#### 스트림 자르기  
```java
Stream<T> skip(long n) // 앞에서부터 n개  
Stream<T> limit(long maxSize) // maxSize 이후의 요소는 잘라냄 n개 건너뛰기  
IntStream intStream = IntStream.rangeClosed(1, 10); // 12345678910  
intStream.skip(3).limit(5).forEach(System.out::print); // 45678
```

참고 : 자바의 정석, https://jeong-pro.tistory.com/165
