---
title: 4장 머스테치로 화면 구성하기
description: 4장 머스테치로 화면 구성하기
categories:
 - Spring-boot
tags:
---  


#### 머스테치  
  * JSP와 같이 HTML을 만들어주는 템플릿 엔진  
  * 수많은 언어를 지원하는 가장 심플한 템플릿 엔진  
  * 서버 템플릿 엔진, 클라이언트 템플릿 엔진으로 모두 사용 가능  
  * 스프링 부트에서 공식 지원하는 템플릿 엔진  
  의존성 추가로 추가 설정 없이 설치 가능  
  * src/main/resources/templates 에 머스테치 파일을 두면 스프링 부트에서 자동으로 로딩  

```java
@GetMapping("/")
public String index(){
  return "index"
}
```  
머스테치 스타터 덕분에 컨트롤러에서 "index"문자열을 반환할 때 앞의 경로와 뒤의 파일 확장자는 자동으로 지정됨  
즉 src/main/resources/templates/index.mustache로 전환되어 View Resolver가 처리하게 됨  

#### View Resolver  
<U>URL 요청의 결과를 전달할 타입과 값을 지정하는 관리자</U> 격으로 볼 수 있음   

## 머스테치의 기본 사용 방법  
* ```{ {>layout/header } }```  

```{ {>} }```는 현재 머스테치 파일을 기준으로 다른 파일을 가져옴  

* **리스트 순회 / 객체 필드 사용**  

```HTML
{ {#posts} }
  <tr>
    <td>{ {id} }</td>
    <td><a href="/posts/update/{ {id} }">{ {title} }</a></td>
    <td>{ {author} }</td>
    <td>{ {modifiedDate} }</td>
  </tr>
{ {/posts} }
```  

  * { {#posts} }  
    posts 라는 List를 순회함  
    Java의 for문과 동일함  
  * { {id} } 등의 { {변수명} }  
  List에서 뽑아낸 객체의 필드 사용  

## Model  
```java  
@GetMapping("/posts/update/{id}")
    public String postsUpdate(@PathVariable Long id, Model model){
        PostsResponseDto dto = postsService.findById(id);
        model.addAttribute("post",dto);
        return "posts-update";
    }

```
* 서버 템플릿 엔진에서 사용할 수 있는 객체를 저장할 수 있음  
* 여기서는 postsService.findById로 가져온 결과를 posts로 index.mustache에 전달  

## js/css 선언 위치를 다르게 하여 웹사이트의 로딩 속도를 향상하는 방법  
부트스트랩, 제이쿼리 등 프론트엔드 라이브러리를 사용할 수 있는 방법 2가지  
1. 외부 CDN 사용  
  본인의 프로젝트에 직접 내려 받아 사용할 필요 없고, HTML/JSP/Mustache에 코드만 한줄 추가하므로 간단  
2. 직접 라이브러리를 받아 사용  

실제 서비스에서는 외부 서비스에 우리 서비스가 의존을 하게 돼버려서, CDN을 서비스 하는 곳에 문제가 생기면 덩달아 문제가 생기므로 CDN을 잘 사용하지 않음  

* 레이아웃 방식  
공통 영역을 별도의 파일로 분리하여 필요한 곳에서 가져다 쓰는 방식  






## js 객체를 이용하여 브라우저의 전역 변수 충돌 문제를 회피하는 방법  
=> 객체를 이용하여 스코프 관리하기  

```javascript
var init = function(){

};
var save = function(){

};

init();
```  
index.js에 위와 같이 정의 되어있고, a.js라는 별도의 js파일에도 init과 save가 정의 되어있는 상태에서 index.mustache가 두 js파일을 모두 추가한다면 덮어쓰이게 된다  

브라우저의 스코프는 공용 공간으로 쓰이기 때문에 나중에 로딩된 js의 init,save가 먼저 로딩된 js의 function을 <u>덮어쓰게 됨</u>  

여러 사람이 참여하는 프로젝트에서 중복된 함수이름은 자주 발생하므로 모든 function의 이름을 확인하긴 어려움  
이러한 문제를 피하려고 index.js만의 유효범위를 만들어 사용하는 것  

방법  
: var index라는 객체를 만들어 해당 객체에서 필요한 모든 function을 선언 하는것  
index 객체 안에서만 function이 유효하므로 다른 js와 겹칠 위험 사라짐  

* 이 식의 프론트엔드의 의존성 관리, 스코프 관리는 최신 자바스크립트 버전이나 앵귤러, 리액트, 뷰 등은 지원하고 있음   


## SpringDataJpa 쿼리 작성  
```java
public interface PostsRepository extends JpaRepository<Posts, Long> {
    @Query("SELECT p FROM Posts p ORDER BY p.id DESC")
    List<Posts> findAllDesc();
}
```  
@Query 어노테이션을 이용해서 쿼리 작성 가능  
* 참고  
규모가 있는 프로젝트에서 데이터 조회는 FK의 조인, 복잡한 조건 등으로 인해 Entity클래스만으로 처리하기 어려워 조회용 프레임워크를 추가로 사용  
대표적으로 querydsl, jooq, MyBatis 등  
조회는 위 3가지 프레임워크 중 1개로 진행하고, 수정/삭제/등록 은 SpringDataJpa 로 진행  


* Querydsl을 추천하는 이유  
  1. 타입 안정성 보장  
    문자열 쿼리가 아니라 메소드 기반의 쿼리 오타, 잘못된 컬럼명 명시할 경우 IDE에서 자동으로 검출  
  2. 국내 많은 회사에서 사용 중   
  3. 레퍼런스 많음  
