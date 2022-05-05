# HTTP

## 특징

- Stateless : 상태를 관리 하지 않음 → 무한한 서버 증식이 가능
    - 필요한 정보는 모든 요청에 다 보내야 함
    - 한계 : 로그인
- connectionless (비연결성) : 연결을 유지하지 않음 → 최소한의 자원으로 통신
    - 한계 : 모든 요청에 3way handshake....
    - 최적화 : HTTP 지속연결(Persistent Connection) - 연결을 어느정도 유지함

** 최대한 Stateless하게 설계하자 !! **

## HTTP 메시지

### 시작라인

요청

- HTTP 메서드
- 요청대상(경로)
- HTTP 버젼

응답

- HTTP 버전
- HTTP 상태 코드
- 이유 문구

### 헤더

- HTTP 전송에 필요한 부가정보

** 단순하지만 확장 가능한 기능 !! **

## HTTP 메서드 종류

- GET : 리소스 조회
    - 데이터는 쿼리 파라미터, 쿼리 스트링을 통해서 전달
    - 바디 사용은 권장하지 않음 (되긴함)
- POST : 요청 데이터 처리, 주로 등록
    - 메시지 바디를 통해 데이터를 전달
- PUT : 리소스를 완전히 대체 → 해당 리소스가 없으면 생성
    - 클라이언트가 리소스 위치를 알고 URL을 지정함
- PATCH : 리소스 부분 변경
- DELETE : 리소스 삭제

안전 : 여러번 호출해도 안전한가

멱등 : 여러 번 호출해도 결과가 동일한가

캐시 가능 

## 상태코드

- 1xx : 처리중
- 2xx :  요청 정상 처리
    - 200 OK 요청 성공
    - 201 Created : 요청 성공해서 새로운 리소스가 생성됨
    - 202 Accepted : 요청이 접수는 되었으나 처리되는 중
    - 204 No Content : 요청은 성공, 응답 본문에 보낼 내용이 없음
- 3xx (Redirection ) : 요청은 완료, 추가 정보 필요
    - 영구 리다이렉션
    - 301 Moved Permanently : 리다이렉트 시 요청 메서드가 GET으로 변하고, 본문이 제거됨
    - 308 Pemanent Redirect : 리다이렉트 시 요청 메서드와 본문 유지
    - 일시적 리다이렉션
    - 302 Found : 리다이렉트 시 요청 메서드가 GET으로 변하고, 본문이 제거
    - 307 Temporary Redirect : 리다이렉트 시 요청 메서드와 본문 유지
- 4xx (Client Error)
    - 401 Anauthorized : 인증이 안됨
    - 403 Forbidden : 서버가 요청은 이해했지만 인가가 안됨
    - 404 Not Found
- 5xx (Server Error)
    - 500 : 서버 오류
    - 503 Service Unavailalbe
    - 서버 문제 → 재시도하면 성공할 수도 있는 요청

PRG : Post/Redirect/Get

- 주문 후에 새로고침으로 중복 주문되는 문제
- 주문 후에 Get으로 리다이렉트 해줌

## Header

- 표현 헤더와 표현으로 나뉨
- 헤더
    - content-type : 표현 데이터의 형식
    - content-encoding : 표현 데이터 인코딩
    - content-language : 표현 언어의 자연 언어
- 협상 (contents nego)
    - Accept-... : 요청 시 클라이언트가 원하는 정보로 요청해 볼 수 있음
    - 우선순위를 줄 수 있음 ex) Accept-language: ko-KR,ko;q=0.9,en-US;q=0.8 ...
    - 구체적인 것에 우선함 ex) text/* ...

---
참고 자료 :  
https://www.inflearn.com/course/http-웹-네트워크  