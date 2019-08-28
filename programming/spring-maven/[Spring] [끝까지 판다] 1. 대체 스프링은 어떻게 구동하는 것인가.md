## 대체 스프링은 어떻게 구동하는 것인가

입사한지 3년이 지나니 이제서야 궁금해졌다.
처음에 교육과정에서 Spring 교육을 받을 때, AOP/DI/IOC 뭐 어쩌구저쩌구, 스프링은 스프링 컨테이너에서 알아서 관리하고 불라불라
이것저것 듣긴했으나 결국 구동의 원리나 시작을 알지 못한채 몇 년이 흘렀다.

너무 deep 한건 어렵겠지만 한번 끝까지 정리해봄.

---------------------------------------------------
 
스프링 Web 구동은 지금의 boot 도 있긴 하지만, 고전적인 방식으로 tomcat 으로 한다고 가정해보고 썰을 풀어보려한다. (부트는 추후 업데이트) 
 
### 1. tomcat webapp 폴더엔 패키지 결과물 `WAR`를 넣어야한다.
- WAR란? WAR(WebApplication Archive)
- `intellij` 에서 로컬 tomcat 설정하다보면 war exploded 를 해주게 되는데 자세한 war 종류는 아래와 같다.
 [아키텍트를 꿈꾸며 - 에코지오](https://ecogeo.tistory.com/tag/war%3Aexploded)
```aidl
1. package(archive) : 아카이브(war,ear) 파일로 배포
- 아카이브는 결국 WAS에 의해 압축이 풀림
- 파일이 많을 경우 압축해제 시간 오래걸릴 수 있음
- 리모트 서버에 배포시 한개의 파일만 전송하면 됨
- WAS에서 제공하는 업로드를 통한 배포기능 활용가능
2. exploded(expanded) : 아카이브를 압축해제한 디렉토리 형태 구조
- 압축 및 해제 과정이 불필요
- 별도의 디렉토리에 원본 소스를 복사하여 만듬
- 파일이 많은 경우 복사 시간 오래걸릴 수 있음
- 원본 소스를 건드리지 않고 배포를 원하는 경우 적합
- 리모트 서버에 배포시 파일이 많은 경우 전송 시간이 오래걸릴 수 있음.(rsync는 빠르다?)
3. in-place : 소스 디렉토리(전체 또는 일부)를 그대로 배포
- 추가적인 복사 과정 불필요
- 로컬 서버에 배포하는 경우에 적합
- WAS가 런타임시 생성하는 파일이 소스와 섞일 수 있음
``` 
하나의 zip file 이라고 생각하면 될 것 같고, 메이븐 기준 pom.xml에 기술 된 대로 war packaging 하여 압축 결과물을 만든다.

### 2. war를 만들어서 tomcat webapp 에 갖다 놨다. tomcat 구동할 때 필요한 web.xml 은 대체 뭔가?

- web application의 설정을 위한 deployment descriptor [참고](https://gmlwjd9405.github.io/2018/10/29/web-application-structure.html)
- SUN에서 정해놓은 규칙에 맞게 작성해야 하며 모든 WAS에 대하여 작성 방법이 동일하다. 일종의 Servlet Container Config 규약이라고 보면됨.
- A라는 Url 패턴이 왔을 때 1번 Servlet, B라는 Url 패턴이 왔을 떄 2번 Servlet 하듯이 Servlet 들을 정의해놓고 맵핑되는 녀석에게 던져 준다.
```xml
<web-app>
    <!-- 1. aliases 설정 -->
    <servlet>
        <servlet-name>welcome</servlet-name>
        <servlet-class>servlets.WelcomeServlet</servlet-class>
    </servlet>
    <!-- 2. 매핑 -->
    <servlet-mapping>
        <servlet-name>welcome</servlet-name>
        <url-pattern>/welcome</url-pattern>
    </servlet-mapping>
</web-app>
```  
- web.xml 에는 servlet 위주지만 Filter 역시 적용 할 수 있다. 요청이 Servlet 에 보내전 처리해주고 싶은 것들을 Servlet 에 넘기기 전에 정의하는 것. [파라미터의 빈공백], [encodingFilter] 등등
```xml
<filter>
	<filter-name>springSecurityFilterChain</filter-name>
	<filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
</filter>
<filter-mapping>
	<filter-name>springSecurityFilterChain</filter-name>
	<url-pattern>/welcome</url-pattern>
</filter-mapping>
``` 
- spring 에서는 DelegatingFilterProxy 를 등록하여 그 위 filter-name 에 입력된 bean 을 찾아 등록해준다.

### 3. tomcat 구동시 web.xml 에서 먼저 동작하는 부분 
- Spring project 는 web.xml 에 반드시 아래와 같은 설정을 추가해야한다.
```xml
<listener>
	<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
```
ContextLoaderListener 클래스는 스프링 설정 파일(디폴트에서 파일명 applicationContext.xml)을 로드하면 ServletContextListener 인터페이스를 구현하고 있기 때문에 ServletContext 인스턴스 생성 시(톰켓으로 어플리케이션이 로드된 때)에 호출된다. 
즉, ContextLoaderListener 클래스는 DispatcherServlet 클래스의 로드보다 먼저 동작하여 비즈니스 로직층을 정의한 스프링 설정 파일을 로드한다.

ContextLoaderListener가 엄마가 되고 DispatcherServlet 이 자식 컨텍스트가 되는 것.
결국 아래와 같은 그림을 가지게 된다.

### 4. tomcat `${TOMCAT_HOME}/bin/catalina.sh start` 로 스타트 하면 생기는 일들.
webapp 하위에 war + web.xml 을 읽고 구동을 하게 된다.
아래 이미지와 같은 형태로 결국 컨테이너가 생성이 되는데... 

다음은 DispatcherServlet 이 떠있으며 사용자의 요청을 받기 까지 직접적인 스프링 구동 과정에 대한 정리할 예정임.
아래 스프링 컨테이너가 어떤클래스를 시작으로 bean 이 등록 되고, 하는지 뒤져본다.
<img width="875" alt="스크린샷 2019-08-28 오후 6 06 05" src="https://user-images.githubusercontent.com/22016317/63841879-93135e00-c9be-11e9-849f-4e3ab8c28ef9.png"> 



