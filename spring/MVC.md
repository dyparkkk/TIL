# MVC 패턴

## 기존 jsp, HttpServlet 문제점

- 너무 많은 역할
    - 비즈니스 로직과 뷰 렌더링이 섞여서 유지보수가 어려워짐
- 변경의 라이프 사이클
    - UI의 변경과 비즈니스 로직의 수정은 다르게 발생, 서로에게 영향을 주지 않음
- 기능의 특화
    - JSP는 뷰 렌더링에 특화

## MVC 패턴

- 컨트롤러 : HTTP요청을 받고, 비즈니스 로직 수행, 뷰에 전달할 데이터를 모델에 담음
- 모델 : 뷰에 출력할 데이터를 담아둠, 뷰와 컨트롤러의 의존관계를 끊음
- 뷰

 

![Untitled](https://raw.githubusercontent.com/dyparkkk/TIL/main/spring/img/mvc00.png)

### MVC 적용 (servlet + jsp)

```java
@WebServlet(name = "mvcMemberFormServlet", urlPatterns = "/servlet-mvc/members/new-form")
public class MvcMemberFormServlet extends HttpServlet {

    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String viewPath = "/WEB-INF/views/new-form.jsp";
        RequestDispatcher dispatcher = req.getRequestDispatcher(viewPath);
        dispatcher.forward(req, resp);
    }
}
```

```html
<!-- new-form.jsp -->
<%@ page contentType="text/html;charset=UTF-8" language="java" %>

<html>
<head>
<meta charset="UTF-8">
    <title>Title</title>
</head>
<body>

<!-- 상대경로 사용, [현재 URL이 속한 계층 경로 + /save] -->
<form action="save" method="post">
    username: <input type="text" name="username" />
    age: <input type="text" name="age" />
<button type="submit">전송</button>
</form>
</body>
</html>
```

- `/WEB-INF` 이 경로 안에 jsp가 있으면 외부에서 직접 호출이 불가능
- redirect vs forward
    - 리다이렉트는 클라이언트에 응답을 하고 다시 요청이 들어옴
    - 포워드는 서버 내부에서 일어나는 호출
- <form action="save" : 상대경로이다
    - 현재 계층의 경로 + /save 를 호출해줌
    

### mvc 한계

- 컨트롤러의 중복
    - forward를 계속 호출해햐암
    - viewPath 중복
- 사용하지 않는 코드
    - HttpServletRequest, HttpServletResponse 등 중복
- 공통처리가 어렵다
    - 컨트롤러 호출 전에 공통 처리를 해주면 좋음
    - → **프론트 컨트롤러 패턴**( 수문장 역할을 하는 기능) - 스프링 MVC의 핵심

### FrontController 패턴

- 서블릿의 하나로 클라이언트의 요청을 받음
- 프론트 컨트롤러가 요청에 맞는 컨트롤러를 찾아서 호출
    - 나머지 컨트롤러는 서블릿을 사용하지 않아도 됨
    - 입구를 하나로 ( 공통 처리 가능)
- 스프링 웹 MVC의 `DispatcherServlet`이 프론트 컨트롤러 패턴임


---
참고자료 :  
https://www.inflearn.com/course/스프링-mvc-1  
