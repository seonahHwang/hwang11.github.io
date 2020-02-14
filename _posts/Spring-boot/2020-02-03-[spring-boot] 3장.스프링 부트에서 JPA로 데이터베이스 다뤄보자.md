---
title: Spring-boot 3장 스프링 부트에서 JPA로 데이터베이스 다뤄보자
description: Spring-boot 3장 스프링 부트에서 JPA로 데이터베이스 다뤄보자
categories:
 - Spring-boot
tags:  
---  
MyBatis, iBatis는 ORM이 아님. SQL Mapper.  
ORM : 객체 매핑  
SQL Mapper : 쿼리 매핑  

### Spring Data JPA  
JPA : <U>인터페이스</U>  
인터페이스 이므로 구현체(ex) Hibernate,Eclispe Link 등) 필요하지만 Spring에서 JPA를 다룰 때 구현체를 직접 다루지 않음  

구현체들을 더 쉽게 사용하고자 추상화시킨 *Spring Data JPA* 라는 모듈을 이용하여 JPA 기술을 다룸  

* JPA <- Hibernate <- Spring Data JPA  

Hibernate 쓰는 것과 Spring Data JPA를 쓰는 것에 큰 차이는 없지만, 이렇게 한단계 감싸놓은 Spring Data JPA가 등장한 이유 2가지  
1. 구현체 교체의 용이성  
  * Hibernate 외에 다른 구현체로 쉽게 교체 하기 위함   
2. 저장소 교체의 용이성  
  * 관계형 데이터베이스 외에 다른 저장소로 쉽게 교체하기 위함  
  * 관계형 DB로 트래픽이 감당이 안될때 MongoDB로 교체하고 싶다면 Spring Data JPA 대신 Spring Data MongoDB로 의존성 교체만 하면됨. Spring Data의 하위 프로젝트 들은 기본적인 CRUD 인터페이스가 동일하기 때문  


### h2
* 인메모리형 관계형 데이터베이스  
* 별도의 설치 필요 없이 프로젝트의 의존성 관리만으로 사용 가능  
* 메모리에서 실행되기 때문에 어플리케이션을 시작할때마다 초기화되는 이점이 있어 테스트 용도로 사용  

**도메인**  
게시글, 댓글, 회원 등 소프트웨어에 대한 요구사항 또는 문제영역  

### Post class Code
```java
@Getter
@NoArgsConstructor
@Entity
public class Posts extends BaseTimeEntity {
    @Id //해당 테이블의 PK필드를 나타냄
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(length = 500, nullable = false)
    private String title;

    @Column(columnDefinition = "TEXT", nullable = false)
    private String content;

    private String author;

    @Builder
    public Posts(String title, String content, String author){
        this.title = title;
        this.content = content;
        this.author = author;
    }

    public void update(String title,String content){
        this.title = title;
        this.content = content;
    }

}
```  
1. 어노테이션 순서  
  * 주요어노테이션을 클래스에 가까이 두어서 나중에 코틀린 등의 새언어로 변경하게 되더라도 클래스에서 먼 부분만 수정할 수 있도록

2. @Entity  
  * 테이블과 링크 될 클래스  
  * 클래스의 카멜케이스 이름으로로 테이블 명 칭함  

3. @Id  
  * 해당 테이블의 PK를 나타냄  

4. @GeneratedValue  
  * PK의 생성 규칙을 나타냄
  * 스프링 부트 2.0에서는 GenerationType.IDENTITY 옵션을 추가해야하만 auto_increment가 됨  


5. auto_increment  
  * 데이터 추가할 때 마다 id값 1씩 증가  

6. @NoArgsConstructor
  * 기본 생성자 자동 추가  

7. @Builder
  * 해당 클래스의 빌더 패턴 클래스 생성
  * 생성사 상단에 선언시 생성자에 포함된 필드만 빌더에 포함  
  * 어느 필드에 어떤 값을 채워야하는지 명확하게 인지 가능

8. Entity 클래스에서는 절대 Setter 메소드를 만들지 않음. 그렇지 않으면 해당 클래스의 인스턴스 값들이 언제 어디서 변해야하는지 코드상으로 명확하게 구분하기 힘들어짐  

9. Setter가 없는 상황에서 어떻게 값을 채워 DB 삽입?
  * 생성자를 통해서 최종값을 채운 후 DB 삽입   

### PostsRepository code  
```java
public interface PostsRepository extends JpaRepository<Posts, Long> {
}
```  
* DB Layer 접근자. 인터페이스. Repository라고 부름  
* JpaRepository<Entity클래스, PK타입>를 상속하면 기본적인 CRUD메소드 자동 생성
* @Repository를 추가할 필요도 없이, Entity class와 같은 곳에 위치하면 됨. 따라서 Entity class와 기본 Repository는 함께 움직여야하므로 도메인 패키지에서 관리  

### PostsRepository Test code  
```java
@After
   public void cleanup(){
       postsRepository.deleteAll();
   }

   @Test
   public void 게시글저장_불러오기(){
       //given
       String title = "테스트 게시글";
       String content = "테스트 본문";

       postsRepository.save(Posts.builder()
               .title(title)
               .content(content)
               .author("tjsdk2769@gmail.com")
               .build());

       //when
       List<Posts> postsList = postsRepository.findAll();

       //then
       Posts posts = postsList.get(0);
       assertThat(posts.getTitle()).isEqualTo(title);
       assertThat(posts.getContent()).isEqualTo(content);
   }
```

1. @After  
    * Junit에서 테스트가 끝난 후 실행되는 메소드 지정  
    * 보통 배포 전 전체테스트 때 데이터간 침범을 막기 위해서 사용  
    * 여러 테스트가 동시에 수행되면 h2에 데이터가 남아 있을 수 있어서 @After로 데이터 삭제해주는게 좋음  

2. postsRepository.save  
  * 테이블 posts에 insert/update 쿼리 실행  
  * id 값 없으면 insert, 있으면 update  

3. 별다른 설정없이 @SpringBootTest를 사용할 경우 H2 데이터베이스를 자동으로 실행  


### SQL 로그 관련 설정  
application.properties 에 SQL 로그 설정하기

```
spring.jpa.show_sql=true // 콘솔에서 쿼리로그 확인 가능
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect // 쿼리로그를 MySQL 버전으로 변경

```
<br>
<br>

### Spring 웹 계층  
![spring 웹 계층](https://user-images.githubusercontent.com/37287788/73628177-d7540a00-4692-11ea-8987-dd41b1d74460.png)  

Web Layer  
  *  컨트롤러와 JSP/FreeMarker 등의 뷰 템플릿 영역  
  * 필터, 인터셉터, 컨트롤러 어드바이스 등 외부 요청과 응답에 대한 전반적인 영역

Service Layer
  * @Service에 사용되는 영역  
  * Controller와 Dao의 중간 영역  
  * @Transactional이 사용되어야 하는 영역  

Repository Layer
  * Database와 같이 데이터 저장소에 접근 하는 영역

Dtos  
  * Dto는 계층 간에 데이터 교환을 위한 객체, Dtos는 이들의 영역   
  * 뷰 템플릿 엔진에서 사용될 객체거나 Repository Layer에서 결과로 넘겨준 객체  

Domain Model  
  * 도메인이라고 불리는 것을 모든 사람이 공유하고 이해할 수 있도록 단순화 시킨 것이 도메인 모델  
  * 택시 앱이라면 배차, 탑승, 요금 등이 모두 도메인이 될 수 있음  
  * @Entity가 사용된 영역 역시 도메인 모델  
  * 무조건 데이터베이스의 테이블과 관계가 있어야만 하는 것은 아님  

**비지니스 처리를 담당해야 할 곳**  
=> Domain      

* 기존에 서비스로 처리하던 방식은 <U>*트랜잭션 스크립트*</U>  
* 서비스는 트랜잭션과 도메인 간의 순서만 보장  

### 생성자로 Bean 주입  
bean 주입 방법은 setter / @Autowired / 생성자 가 있음  
가장 권장하는 방식은 생성자  
@RequiredArgsConstructor에서 final이 선언된 모든필드를 생성자로 만들어줌  


### Entity 클래스 Request / Response 클래스로 사용 금지  
* Entity 클래스는 데이터베이스와 맞닿은 핵심 클래스이므로, Entity 클래스 기준으로 테이블 생성되고 스키마가 변경됨  
* Entity 클래스가 변경되면 여러 클래스에 영향을 끼치지만, Request와 Response용 Dto는 View를 위한 클래스라 정말 자주 변경이 필요함  
* *View Layer*와 *DB Layer*의 역할 분리를 철저하게 하는 게 좋음  


@WebMvcTest의 경우 JPA기능이 작동하지 않아 @SprintBootTest와 TestRestTemlate를 사용하면됨  


### 더티체킹  

영속성 컨텍스트란 엔티티를 영구저장하는 환경  
JPA의 핵심 내용은 엔티티가 영속성 컨텍스트에 포함되느냐 아니냐  
트랜잭션안에서 데이터베이스에서 데이터를 가져오면 이 데이터는 영속성 컨텍스트가 유지된 상태  
이 상태에서 해당 데이터의 값을 변경하면 트랜잭션이 끝나는 시점에 해당 테이블에 변경분을 반영  
Entity의 객체의 값만 변경하면 별도로 Update 쿼리를 날릴 필요가 없음  


### JPA Auditing  

#### BaseTimeEntity class code  
```java
@Getter
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
public abstract class BaseTimeEntity {

    @CreatedDate //Entity가 생성되어 저장될 때 시간이 자동 저장됨
    private LocalDateTime createdDate;

    @LastModifiedDate // 조회한 Entity의 값을 변경할 때 시간이 자동 저장됨
    private LocalDateTime modifiedDate;
}
```
이 class가 모든 Entity의 상위클래스가 되면 모든 테이블에 생성, 수정 시간이 컬럼으로 등록된다  

1. @MappedSuperclass  
  * JPA Entity 클래스들이 BaseTimeEntity를 상속할 경우 필드들도 칼럼으로 인식하도록 함  

2. @EntityListeners(AuditingEntityListener.class)  
  * BaseTimeEntity에 Auditing기능을 포함시킴  

  
