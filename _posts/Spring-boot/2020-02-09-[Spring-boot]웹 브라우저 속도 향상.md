---
title: 웹 사이트의 로딩 속도 향상 방법
description: 웹 사이트의 로딩 속도 향상 방법
categories:
 - Spring-boot
tags:
---  
### 웹 사이트의 로딩 속도 향상 방법  
페이지 로딩속도를 높이기 위해  
header에 css를
footer에 js를 두었음  
HTML은 위에서부터 코드가 실행되기 때문에 head가 모두 실행된 후에 footer가 실행됨  
즉, head가 다 불러지지 않으면 사용자 쪽은 백지만 노출됨  

특히 js의 용량이 클수록 body 부분의 실행이 늦어지므로 js는 body 하단에 두어 화면이 다 그려진 뒤 호출하는 것이 좋음  

css는 화면을 그리는 역할이므로 head에서 불러오는 것이 좋음  
그렇지 않으면 css가 적용되지 않은 깨진 화면을 사용자가 볼 수 있기 때문  

bootstrap.js의 경우 제이쿼리가 꼭 있어야만 하기 때문에 부트스크랩보다 먼저 호출 되도록 작성  
이런 상황을 bootstrap.js가 제이쿼리에 의존한다고 함  
