# 스프링 부트 개발자 도구

기존 스프링은 WAR 파일을 만들고 애플리케이션에 배포를 하고 있었다. 스프링 부트가 나오기 한참 전부터 스프링 프레임워크는 무거운 애플리케이션 서버 대신 서블릿 컨테이너를 선택하여 재시작 문제 해결을 시도했었다. 하지만 더더욱 개선을 했어야 했다.

2014년 출시된 스프링부트는 내장형 서블릿 컨테이너(embedded servlet container)라는 혁신으로 WAR 파일을 사용해서 아파치 톰캣 등의 서블릿 컨테이너에 애플리케이션을 배포하는 방식이 아닌 애플리케이션에 서블릿 컨테이너를 포함하는 방식으로 변화했다. 따라서 애플리케이션 시작 속도와 애플리케이션 배포 개념을 뒤집는데 성공했다. 애플리케이션은 서블릿 컨테이너에 종속되지도 않았다.

결국 JVM만 설치돼있으면 어떤 장비에도 JAR 파일을 배포해서 애플리케이션을 실행시킬 수 있었다.

> 2021.03 릴리즈된 Spring Native는 JVM마저도 애플리케이션에서 떼내었다. 권장 스프링 버전은 2.4.x 이상이다.

이 외에도 스프링 부트팀은 개발을 할 때 편리하게 개발하기 위한 방법을 제공하려고 노력을 하는데 devtools라는 모듈을 만들었다.



```
dependencies {
    developmentOnly("org.springframework.boot:spring-boot-devtools")
}
```

[스프링 공식문서](https://docs.spring.io/spring-boot/docs/current/reference/html/using.html#using.devtools) 에 따르면 Developer Tools에는 다음과 같은 기능이 있다고 ㅎ나다.

**DevTools 기능**

- 애플리케이션 재시작(restart)과 리로드(reload) 자동화
- 환경설정 정보 기본값 제공
- 자동설정(autoconfiguration) 변경사항 로깅
- 정적 자원 제외
- 라이브 리로드(LiveReload) 지원

## 자동 재시작과 리로딩

- 스프링 부트는 재시작 호출을 감지하면 자동으로 재시작 된다.
- 재시작 호출은 IDE마다 다르다. 이클립스는 ‘save’, 인텔리제이는 ‘build’
- 정적 자원이나 지정된 자원의 변경은 무시할 수 있다.
- 재시작 하는 방법은 ‘콜드 스타트’ ‘웜 스타트’ 이라는 개념들이 있다.
  - 콜드 스타트 모든 정보를 리로드 하는 방식이다.
  - 클래스로더가 서드파티 라이브러리도 모두 로드한다.
- 웝 스타트는 일부 필요한 부분만 로딩하는 것이다.
  - 서드파티 라이브러리는 그대로 두고, 개발자가 작성한 클래스만 로드한다.


DevTools는 개발자가 작성한 코드를 하나의 클래스로더로 로딩하고 서드파티 라이브러리는 별도의 클래스로더로 로딩한다. 따라서 애플리케이션이 재시작되면 개발자가 작성했던 코드를 로딩했던 클래스로더도 종료되고 새로운 클래스로더가 사용된다. 

하지만 서드파티 라이브러리를 로딩했던 클래스로더는 그대로 남는다. 이렇게 개발자 코드만 새로 리로드하여 모든 것을 새로 시작하는 콜드(cold) 시작 방식보다 빨리 애플리케이션을 재시작할 수 있다. 사실 리로드 개선 효과를 최선으로 끌어올리려면 제이레블(JRebel) 같은 자바 에이전트 솔루션을 사용해야 한다.

DevTools의 재시작 방식은 IDE의 재시작 유발 신호을 받으면 실행된다.
다만 정적 자원의 변경이 발생해도 재시작을 하지 않는다.

- /META-INF/maven
- /META-INF/resource
- /resource
- /static
- /public
- /templates

application.properties

```
# 경로 지정
spring.devtools.restart.exclude=static/**,public/**

# 재시작 비활성화
spring.devtools.restart.enabled=false
```

위와 같은 방법으로 재시작을 유발하지 않는 파일의 경로를 설정할 수 있다.

> **Restart vs Reload** </br></br>
> The restart technology provided by Spring Boot works by using two classloaders. Classes that do not change (for example, those from third-party jars) are loaded into a base classloader. Classes that you are actively developing are loaded into a restart classloader. When the application is restarted, the restart classloader is thrown away and a new one is created. This approach means that application restarts are typically much faster than “cold starts”, since the base classloader is already available and populated.</br></br>
> If you find that restarts are not quick enough for your applications or you encounter classloading issues, you could consider reloading technologies such as JRebel from ZeroTurnaround. These work by rewriting classes as they are loaded to make them more amenable to reloading.

## 개발 모드에서 캐시 비활성화

스프링 부트와 통합되는 많은 컴포넌트는 다양한 캐시(cache) 수단을 가지고 있다. 어떤 템플릿 엔진은 컴파일된 템플릿을 캐시하기도 한다. 이는 상용 운영환경에서는 편리한 기능이지만 변경사항을 계속 확인해야 하는 개발과정에서는 불편함을 가중시킨다.

빌드 파일에 spring-boot-devtools를 추가해서 IDE나 메이븐으로 애플리케이션을 실행하면 개발 모드로 실행이 되고 환경설정이 기본값으로 지정된다.

## 부가적 웹 활동 로깅


[application.properties](http://application.properties) 파일에 다음 내용을 추가하면 web 로깅 그룹에 대한 로깅을 활성화 할 수 있다.

```
logging.level.web=DEBUG
```

스프링 로직 내의 웹 관련 로직이 로깅이 된다.

## 자동설정에서의 로깅 변경

스프링 부트에서 가장 강력한 기능은 자동설정(Autoconfiguration)이다. 스프링 부트2.x 부터 자동으로 구성한 모든 내용은 로깅되지 않는다. 다만 기본 설정에서 변경된 내용만 로깅이 된다.

## 라이브 리로드 지원

라이브 리로드 서버가 내장되어 있다. 서버가 재시작 됐을 때 웹 페이지를 새로 로딩하는 단순한 작업을 수행한다. 서버가 재시작되면 브라우저의 새로고침을 자동으로 눌러준다.
