# CORS

> **교차 출처 리소스 공유**
(Cross-Origin Resource Sharing, [CORS](https://developer.mozilla.org/ko/docs/Glossary/CORS))는 추가 [HTTP](https://developer.mozilla.org/ko/docs/Glossary/HTTP)  헤더를 사용하여, 한 [출처](https://developer.mozilla.org/ko/docs/Glossary/Origin)에서 실행 중인 웹 애플리케이션이 다른 출처의 선택한 자원에 접근할 수 있는 권한을 부여하도록 브라우저에 알려주는 체제 - mdn
> 

출처(origin)이란 

protocol은 `https://`  Host는 `www.google.com` Port는 `:443`로 구성되며 **동일 출처(Same Origin)**란 *scheme, host, port* 가 모두 같을 때를 말함.

## 사용이유

클라이언트 어플리케이션은 사용자의 공격에 매우 취약함

다른 출처의 어플리케이션이 서로 통신하는 것에 제약이 없다면, 악의적인 사용자가 소스코드를 보고 CSRF ( cross-site Request Forgery) 나 XXS 같은 방법으로 어플리케이션 코드 인 척 속여서 사용자 정보 탈취 가능 

### CSRF

- 특정 웹 사이트가 사용자의 웹 브라우저를 신용하는 상태를 노린 공격
- 
1. 이용자는 웹사이트에 로그인하여 정상적인 [쿠키](https://ko.wikipedia.org/wiki/HTTP_%EC%BF%A0%ED%82%A4)를 발급받는다
2. 공격자는 다음과 같은 [링크](https://ko.wikipedia.org/wiki/%ED%95%98%EC%9D%B4%ED%8D%BC%EB%A7%81%ED%81%AC)를 이메일이나 게시판 등의 경로를 통해 이용자에게 전달한다.
    
    http://www.geocities.com/attacker
    
3. 공격용 [HTML](https://ko.wikipedia.org/wiki/HTML) 페이지는 다음과 같은 이미지태그를 가진다.
    
    `<**img** src= "https://travel.service.com/travel_update?.src=Korea&.dst=Hell">`
    해당 링크는 클릭시 정상적인 경우 출발지와 도착지를 등록하기위한 링크이다. 위의 경우 도착지를 변조하였다.
    
4. 이용자가 공격용 페이지를 열면, 브라우저는 이미지 파일을 받아오기 위해 공격용 URL을 연다.
5. 이용자의 승인이나 인지 없이 출발지와 도착지가 등록됨으로써 공격이 완료된다. 해당 서비스 페이지는 등록 과정에 대해 단순히 쿠키를 통한 본인확인 밖에 하지 않으므로 공격자가 정상적인 이용자의 수정이 가능하게 된다.

## 동작 방식

- 클라이언트와 서버의 출처의 비교를 브라우저에서 실행
    - 만약 CORS 정책을 위반하는 출처가 다른 요청을 서버에서 응답하더라고 브라우저가 검사해서 버림
    - 이런 경우 서버쪽 로그에는 정상 응답만 기록됨!!

- 웹 클라이언트 어플리케이션은 다른 출처의 리소스를 요청 할 때 HTTP프로토콜을 사용하고, 요청 헤더에 Origin 이라는 필드에 출처를 담아서 보냄
- 서버가 응답할 때 Origin과 응답의 Access-Control-Allow-Origin과 비교해보고 유효한 응답인지 확인

### 3가지 시나리오

1. Preflight Request

![Preflight Request](https://raw.githubusercontent.com/dyparkkk/TIL/main/Network/img/cors1.png)

Preflight Request

브라우저는 자신이 보낸 예비 요청과 서버가 응답에 담아준 허용 정책을 비교 → 요청을 보내는 것이 안전하다고 판단되면 같은 엔드포인트로 다시 본 요청을 보냄

 이후 서버가 이 본 요청에 대한 응답을 하면 브라우저는 최종적으로 이 응답 데이터를 자바스크립트에게 넘겨줌

- Origin 에 대한 정보 뿐만아니라 다른 정보들도 포함되어있음
- Origin 과  Access-Control-Allow-Origin이 다르면 응답코드가 200이더라도 에러가 뜸

### Simple Request

![Simple Request](https://raw.githubusercontent.com/dyparkkk/TIL/main/Network/img/cors2.png)

Simple Request

프리플라이트와 단순 요청 시나리오는 전반적인 로직 자체는 같되, 예비 요청의 존재 유무만 다르다.

- 특정 조건을 만족하는 경우에만 예비 요청을 생략할 수 있음

### Credentialed Request

조금 더 보안을 강화하고 싶을 때 사용하는 방법

- 출처 검사 뿐만 아니라 요청에 인증과 관련한 정보도 포함되어야 함
- include 옵션으로 인증 정보를 담아서 보내야 함
- Access-Control-Allow-Origin 에는 *을 사용할 수 없으며 명시적인 url이여야 함

## CORS 해결 방법

1. **Access-Control-Allow-Origin 세팅하기**
2. **Webpack Dev Server로 리버스 프록싱하기**
    - 프론트 개발할 때  백엔드에서 Access-Control-Allow-Origin에 [localhost](http://localhost) 넣지않음
    - 따라서 리버스 프록시로 우회해서 접근

---
참고자료 :  
[https://developer.mozilla.org/ko/docs/Web/HTTP/CORS](https://developer.mozilla.org/ko/docs/Web/HTTP/CORS)  
[https://evan-moon.github.io/2020/05/21/about-cors/](https://evan-moon.github.io/2020/05/21/about-cors/)  
[https://ko.wikipedia.org/wiki/사이트_간_요청_위조](https://ko.wikipedia.org/wiki/%EC%82%AC%EC%9D%B4%ED%8A%B8_%EA%B0%84_%EC%9A%94%EC%B2%AD_%EC%9C%84%EC%A1%B0)  
