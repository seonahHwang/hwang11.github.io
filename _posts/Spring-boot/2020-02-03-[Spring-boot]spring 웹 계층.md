---
title: Spring 웹 계층
description: Spring 웹 계층
categories:
 - Spring-boot
tags:  
---  
## Spring 웹 계층  
![spring 웹 계층](https://user-images.githubusercontent.com/37287788/73628177-d7540a00-4692-11ea-8987-dd41b1d74460.png)  

#### Web Layer  
  *  컨트롤러와 JSP/FreeMarker 등의 뷰 템플릿 영역  
  * 필터, 인터셉터, 컨트롤러 어드바이스 등 외부 요청과 응답에 대한 전반적인 영역

#### Service Layer
  * @Service에 사용되는 영역  
  * Controller와 Dao의 중간 영역  
  * @Transactional이 사용되어야 하는 영역  

#### Repository Layer
  * Database와 같이 데이터 저장소에 접근 하는 영역

#### Dtos  
  * Dto는 계층 간에 데이터 교환을 위한 객체, Dtos는 이들의 영역   
  * 뷰 템플릿 엔진에서 사용될 객체거나 Repository Layer에서 결과로 넘겨준 객체  

#### Domain Model  
  * 도메인이라고 불리는 것을 모든 사람이 공유하고 이해할 수 있도록 단순화 시킨 것이 도메인 모델  
  * 택시 앱이라면 배차, 탑승, 요금 등이 모두 도메인이 될 수 있음  
  * @Entity가 사용된 영역 역시 도메인 모델  
  * 무조건 데이터베이스의 테이블과 관계가 있어야만 하는 것은 아님  

#### 비지니스 처리를 담당해야 할 곳
=> Domain      

* 기존에 서비스로 처리하던 방식은 <U>*트랜잭션 스크립트*</U>  
* 서비스는 트랜잭션과 도메인 간의 순서만 보장  
