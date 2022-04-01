# Oauth

**OAuth를 바탕으로 제 3자 서비스는 외부 서비스(Facebook, Google 등)의 특정 자원을 접근 및 사용할 수 있는 권한을 인가받게 됨**

client 는 등록절차를 통해 clientID, client Secret, Authorized redirect URL 을 받음

![Untitled](https://raw.githubusercontent.com/dyparkkk/TIL/main/Network/img/OAuth00.png)

- Authorization Code 는 client secret와 같이 보내야 access token을 발급해준다.
    - Authorization Code 탈취 위험 때문에 ....
- 리소스 서비스는 리다이렉트 url로 Authorization Code 를 보내준다
    - 그걸로 클라이언트는 access token을 발급 받을 수 있다.
- 

---
참고자료 :  
[https://showerbugs.github.io/2017-11-16/OAuth-란-무엇일까](https://showerbugs.github.io/2017-11-16/OAuth-%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9D%BC%EA%B9%8C)  