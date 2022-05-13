# Servlet

## 서블릿

- 웹 서버의 요청과 응답을 처리해줌
    - 서버에서 처리해야 할 업무 → tcp/ip 처리, 소켓 연결, http 헤더 파싱 등등
- HTTP 스펙을 쉽게 사용할 수 있음
- WAS는 HTTP요청시 request와 response 객체를 새로 만들어서 서블릿 객체를 호출
    - 개발자는 req에서 HTTP정보를 꺼내서 사용

### 서블릿 컨테이너

- 톰캣처럼 서블릿을 지원하는 WAS를 서블릿 컨테이너라고 함
    - 서블릿 컨테이너는 서블릿 객체를 생성, 초기화, 호출 등 생명주기 관리
- 서블릿 객체는 싱글톤으로 관리
    - → 모든 요청은 동일한 서블릿 객체에 접근하게 됨
- **동시 요청을 위한 멀티 스레드 지원( 개발자가 신경쓰지 않아도 됨)**

### 스레드 풀

- 필요한 스레드를 스레드 풀에 보관하고 관리
    - 스레드는 생성비용이 비쌈, 생성을 계속하면 관리가 안됨
- 생성 가능한 스레드의 최대치가 잇음 →
- WAS 주요 튜닝 포인트임 !! ( 클라우드 사용중일시 서버를 먼저 늘림)

## HttpServletRequest

- 서블릿은 HTTP 요청 메시지를 개발자가 편리하게 사용하도록 파싱해서 제공
    - HttpServletRequest 객체에 담아서 제공
- 임시 저장소 기능
    - 해당 HTTP요청이 끝날때까지 유지됨
    - req.setAttribute(name, val)
- 세션 관리 기능
    - req.getSession()

### HTTP 요청 데이터 전달 방식

- GET - 쿼리 파라미터
    - /url?username=hello
    - 메시지 바디 없이, url만으로 데이터 전달
    - ex) 검색필터, 페이징 등에서 많이 사용
- POST - HTML Form
    - content-type: application/x-www-form-urlencoded
    - 메지지 바디에 쿼리 파라미터 형식으로 전달 - username=hello?age=20
    - 쿼리 파라미터 조회 방식으로 동일하게 조회 가능
    - ex) 회원가입, html form
- HTTP message Body 사용
    - HTTP API에서 주로 사용, json, xml, text (주로 json)
    - post, put, patch
    

```java
// GET - 쿼리 파라미터
@WebServlet(name = "helloServlet", urlPatterns = "/hello")
public class HelloServlet extends HttpServlet {

    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.println("HelloServlet.service");

        String username = req.getParameter("username");

        resp.setContentType("text/plain");
        resp.setCharacterEncoding("utf-8");
        resp.getWriter().write("hello " + username);
    }
}
```

```java
// HTTP message Body 사용
@WebServlet(name = "RequestBodyJsonServlet", urlPatterns = "/request-body-json")
public class RequestBodyJsonServlet extends HttpServlet {

    private ObjectMapper objectMapper = new ObjectMapper();

    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        ServletInputStream inputStream = req.getInputStream();
        String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);

        HelloData helloData = objectMapper.readValue(messageBody, HelloData.class);
        System.out.println("username = " + helloData.getUsername());
        System.out.println("age = " + helloData.getAge());

        resp.getWriter().write("ok");
    }
}
```

> 참고 :
json결과를 파싱해서 자바 객체로 바꾸려면 jackson, Gson 같은 json변환 라이브러리를 사용해야함 
스프링 부트로 Spring MVC를 사용하면 기본은로 **Jackson 라이브러리(ObjectMapper)** 를 함께 제공해줌
> 

## HttpServletResponse

- 편의기능 제공
    - 헤더세팅, 쿠키, 리다이렉트

```java
@WebServlet(name = "responseHeaderServlet", urlPatterns = "/response-header")
public class ResponseHeaderServlet extends HttpServlet {

    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //[status-line]
        resp.setStatus(HttpServletResponse.SC_OK);

        //[response-header]
        resp.setHeader("Content-Type", "text/plain;charset=utf-8");
        resp.setHeader("my-header", "hello");

        // 헤더 편의 메서드
//        resp.setContentType("text/plain");
//        resp.setCharacterEncoding("utf-8");

        // 쿠키 설정
        cookie(resp);

        // 리다이렉트
        resp.sendRedirect("/basic/hello-form.html");

        resp.getWriter().write("ok");
    }

    private void cookie(HttpServletResponse resp) {
        Cookie cookie = new Cookie("myCookie", "good");
        cookie.setMaxAge(600);
        resp.addCookie(cookie);
    }
}
```

- json 반환 ( 오브젝트 매퍼 사용)

```java
@WebServlet(name = "responseJsonServlet", urlPatterns = "/response-json")
public class ResponseJsonServlet extends HttpServlet {

    private ObjectMapper objectMapper = new ObjectMapper();

    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.setContentType("application/json");
        resp.setCharacterEncoding("utf-8");

        HelloData hellodata = new HelloData("park", 21);
        String result = objectMapper.writeValueAsString(hellodata);
        resp.getWriter().write(result);

    }
}
```

---  
참고자료 :   
https://www.inflearn.com/course/스프링-mvc-1   