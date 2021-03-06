
<div class="pull-right">  업데이트 :: 2018.08.09 </div><br>

---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->
<!-- code_chunk_output -->

* [프런트 컨트롤러](#프런트-컨트롤러)
* [DispaterServlet](#dispaterservlet)
* [Handler](#handler)
* [HandlerMapping](#handlermapping)
* [HandlerAdapter](#handleradapter)
* [ViewResolver](#viewresolver)
* [View](#view)
* [DI 컨테이너와 연계](#di-컨테이너와-연계)
	* [ApplicationContext 구성](#applicationcontext-구성)
	* [ApplicationContext LifeCycle](#applicationcontext-lifecycle)

<!-- /code_chunk_output -->

### 프런트 컨트롤러

- 스프링 MVC는 프론트 컨트롤러 패턴(Front Controller)의 아키택처를 채택하고 있음
  - 클라이언트 요청을 프런트 컨트롤러라는 컴포넌트가 받아 요청 내용에 따라 수행하는 핸들러를 선택하는 아키택처

> 기능

- 클라이언트의 요청 접수
- 요청 데이터를 자바객체로 변환
- 입력값 검사 (Bean Validation)
- 핸들러 호출
- 뷰 선택
- 클라이언트에 요청 결과 응답
- 예외 처리

> 흐름

- DispatcherServlet 클래스는 클라이언트의 요청을 받음
  - Controller 처리방법
    - 'HandlerMapping 인터페이스'의 getHandler()를 호출해서 요청 처리하는 Handler객체(컨트롤러)를 가져옴
    - 'HandlerAdapter 인터페이스'의 handle()를 호출해서 Handler객체(컨트롤러)의 메서드를 호출을 의뢰
    - 'HandlerAdapter 인터페이스'의 구현클래스는 Handler객체(컨트롤러)에 구현된 메서드를 호출해서 요청을 수행
  - View의 처리방법
    - DispatcherServlet 클래스는 ViewResolver 인터페이스의 resolveViewName()을 호출해서
    - Handler객체(컨트롤러)에게 반환된 뷰 이름에 대응하는 View 인터페이스 객체를 가져옴
    - DispatcherServlet 클래스는 View 인터페이스의 render()를 호출해서 응답 데이터에 대한 랜더링 요청
    - View 인터페이스의 구현클래스는 JSP와 같은 템플릿 엔진을 사용해서 랜더링할 데이터를 생성
- DispatcherServlet 클래스는 클라이언트에 응답을 반환


### DispaterServlet

- 프론트 컨트롤러와 연동되는 진입점 역할
- 기본적인 처리 흐름을 제어하는 사령탑 역할

> 인터페이스

- HandleExceptionResolver
  - 예외를 처리하기 위한 인터페이스
  - 스프링 MVC가 제공하는 기본 구현클래스 적용
- LocaleResolver, LocaleContextResolver
  - 클라이언트 로캘 정보를 확인하기 위한 인터페이스
  - 스프링 MVC가 제공하는 기본 구현클래스 적용
- ThemeResolver
  - 클라이언트의 테마(UI)를 결정하기 위한 인터페이스
  - 스프링 MVC가 제공하는 기본 구현클래스 적용
- FlashMapManager
  - FlashMap이라는 객체를 관리하기 위한 인터페이스
  - FlashMap은 PRG(Post Redirect Get)패턴의 Redirect와 Get사이에서 모델을 공유하기 위한 Map
  - 스프링 MVC가 제공하는 기본 구현클래스 적용
- RequestToViewNameTranslator
  - 핸들러가 뷰 이름과 뷰를 반환하지 않은 경우에 적용되는 뷰 이름을 해결하기 위한 인터페이스
  - 스프링 MVC가 제공하는 기본 구현클래스 적용
- HandleInterceptor
  - 핸들러 실행 전후에 하는 공통 처리를 구현하기 위한 인터페이스
  - 애플리케이션 개발자가 구현하고 스프링 MVC에서 사용가능
- MultipartResolver
  - 멀티파트 요청을 처리하기 위한 인터페이스
  - 스프링 MVC에서 몇가지 구현 클래스가 제공되지만 기본적용은 아님

### Handler

- 핸들러란, 스프링 MVC에서 프론트 컨트롤러가 받은 요청에 따라 필요한 처리를 수행
- HandlerMapping & HandlerAdapter 인터페이스

> 컨트롤러 구현
- @Controller를 클래스에 지정
  - 요청처리를 수행하는 메서드에 @RequestMapping을 지정
- Controller 인터페이스를 구현
  - 요청을 처리할 메서드(handleRequest)를 구현

> @Controller & @RequestMapping

```java
@Controller
 public class WelcomeController {
   @RequestMapping("/")
   public String home(Model model) {
     model.addAttribute("now", new Date());
     return "home";
   }
 }
```

> Controller 인터페이스

```java
@Component("/")
public class WelcomeController extends AbstractController {

  @Override
  protected ModelAndView handleRequestInternal(HttpServletRequest req, HttpServletResponse res) throws Exception {
    ModelAndView mav = new ModelAndView("home");
    mav.addObject("now", new Date());
    return mav;
  }
}
```

### HandlerMapping

- 요청에 대응할 핸들러를 선택
- RequestMappingHandlerMapping, @RequestMapping에 정의된 설정정보를 바탕으로 실행할 핸들러를 선택
  - 요청과 요청매핑정보를 매칭해서 실행할 핸들러를 선택

```java
@Controller
public class GreethingController {

  @RequestMapping("/hello")
  public String hello() {
    return "hello";
  }

  @RequestMapping("/goodbye")
  public String goodbye() {
    return "goodbye";
  }
}
```

### HandlerAdapter

- 핸들러 메서드를 호출하는 역할
- RequestMappingHandlerAdapter, RequestMappingHandlerMapping에 의핸 선택된 핸들러 매서드를 호출할때 사용

> 핸들러 메서드 시그니처를 정의하는데 사용되는 인터페이스

- HandlerMethodArgumentResolver
  - 핸들러 메서드 매개변수에 전달하는 값을 다루기 위한 인터페이스
- HandlerMethodReturnValueHandler
  - 핸들러 메서드에서 반환된 값을 처리하기 위한 인터페이스

### ViewResolver

- 반환한 뷰의 이름을 보고 이후에 사용할 View 인터페이스를 구현하는 클래스를 선택하는 역할

> 스프링 MVC가 제공하는 ViewResolver

- InternalResourceViewResolver
  - 뷰가 JSP일때 사용, 가장 기본
- BeanNameViewResolver
  - DI 컨테이너에 등록된 빈의 형태로 뷰 객체를 가져올때

### View

- 클라이언트에 반환하는 응답 데이터를 생성하는 역할
- InternalResourceView
  - 템플릿 엔진으로 JSP를 이용할때 사용
- JstView
  - 템플릿 엔진으로 JSP + JSTL을 사용하는 클래스

### DI 컨테이너와 연계

#### ApplicationContext 구성

- 웹 애플리케이션용 애플리케이션 컨텍스트
  - 여러 DispaterServlet용 컨텍스트를 공유
- DispaterServlet용 애플리케이션 컨텍스트
  - DispaterServlet은 각각 독립되어 있어 서로 참조 불가능

#### ApplicationContext LifeCycle

- 초기화 단계
  - WebApplication용 ApplicationContext와 DispaterServlet용 ApplicationContext를 생성하는 단계
  - 서블릿 컨테이너를 기동할때 진행
- 사용 단계
  - ApplicationContext에서 Bean을 이용하는 단계
- 파기 단계
  - 두 ApplicationContext를 파기하는 단계
  - 서블릿 컨테이너가 중지할때 진행

> 흐름

1. 서블릿 컨테이너
2. ContextLoader
3. ApplicationContext(WebApplication용)
4. DispaterServlet
5. WebApplicationContext(DispaterServlet용)



---

**Created by MoonsCoding**

e-mail :: jm921106@gmail.com
