# DNS

## Socket 라이브러리

- 도메인 명을 ip주소로 변환하기 위해 DNS 서버에 접속해야함
    
    → 이 때 클라이언트로 동작하는 것을 DNS 리졸버 라고 함
    
- 리졸버는 Socket 라이브러리에 들어있는 프로그램 !

## DNS

Domain Name System의 약자로 인터넷 주소창에 Host Domain Name을 입력했을 때(ex, [naver.com](http://naver.com/), [google.com](http://google.com/) 등..) 해당 문자를 IP주소로 변환해 주는 시스템

![dns](https://raw.githubusercontent.com/dyparkkk/TIL/main/Network/img/DNS00.png)

dns

## Local DNS server

url에 domain name을 입력했을때 가장 먼저 찾는 dns 서버

여기에 있으면 리턴, 없으면 root DNS server로 물어봄 

- Resolver : dns의 클라이언트이며 호스트 정보를 구하는 프로그램의 요청을 dns 서버에 대한 쿼리로 번역하고 그 응답을 프로그램에 적절한 형태로 변경하는 일을 함.

## Root DNS server

![](https://raw.githubusercontent.com/dyparkkk/TIL/main/Network/img/DNS01.png)

있으면 리턴, 없으면 최상위 도메인 DNS 서버 주소 리턴.

ex) com, kr 

이후 반복 

Recursive Query라고 함

이렇게 찾은 ip주소는 DNS cache에 저장