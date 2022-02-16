**restful api란 rest 아키텍처의 제약조건을 준수하는 api(application programming interface)** 

- api는 정보 제공자와 정보 사용자의 계약으로 지칭
    - 호출과 응답으로 구성됨
    

Restful api를 통해 요청이 수행될 때 **리소스 상태에 대한 표현**을 전송 → http:json, html, xlt, text 등의 형식으로 전송 

Restful api 조건 

- 클라이언트, 서버, 리소스로 구성
- Stateless 클라이언트-서버 커뮤니케이션
- 응답을 캐시하여 재사용할 수 있어야 함
- 어플리케이션 요구 사항별로 다른 형식이 아닌 , 표준화된 형식으로 정보를 전송하도록 통합된 인터페이스
- 서버는 클라이언트가 기능을 확장할 수 있게 url을 같이 보내줌 (선택사항)

api는 소스코드들 간에 데이터를 주고 받는 반면 restful api는 웹의 프로토콜에서 동작함. 

(물론 api는 restful api를 포함함)

참고자료 : 

[RESTful API 개념 정리](https://rural-mouse.tistory.com/34)

[REST API(RESTful API, 레스트풀 API)란 - 서버, 구현, 사용법](https://www.redhat.com/ko/topics/api/what-is-a-rest-api)
