# 그런 REST API로 괜찮은가?

### API
Application Programming Interface
- 코딩할 때 만드는 자바의 인터페이스도 API의 일종이다.
- 우리가 다룰 API는 웹으로 접근하는 API 중의 REST API이다.

### REST
- REpresentational State Transfer
- 인터넷 상의 시스템 간의 상호 운용성(interoperability)을 제공하는 방법중 하나
- 시스템 제각각의 독립적인 진화를 보장하기 위한 방법

**REST API** : REST 아키텍처 스타일을 따르는 API


박사 논문에서 따온 이름이다. 그 논문에서 다루는 주제는 어떻게 하면 인터넷 상의 서로다른 시스템의 독립적인 진화를 보전할 것인가. 웹을 깨트리지 않으면서 HTTP를 진화시키는 논문에서 비롯되었다.
그곳에서 기반한 REST Style을 우리가 개발하고 있지만 정확하게 따르고 있지 않다.

[네이버에서 주관한 DEVIEW 컨퍼런스에서 REST API에 대한 발표](https://deview.kr/2017/schedule/212?lang=ko)
 
오늘날 REST API는 REST API가 아니다. REST API를 구현하려면 아래의 제시된 아키텍처 스타일을 따라해야 하는데 특히 이중에서 **Uniform Interface**, 그 중에서도 **self-descrptive messages**, **hypermisa as the engine of application state(HATEOAS)** 를 만족하지 않는다. 나머지 스타일은 사실 HTTP로 구현하면 자연스럽게 만족할 수 있다.

### REST 아키텍처 스타일 ([발표 영상](https://www.youtube.com/watch?v=RP_f5dMoHFc) 11분)
- Client-Server
- Stateless
- Cache
- Uniform Interface
- Layered System
- Code-On-Demand (optional)
 
### Uniform Interface (발표 영상 11분 40초)
- Identification of resources
- manipulation of resources through represenations
- **self-descrive messages**
- **hypermedia as the engine of appliaction state (HATEOAS)**

두 문제를 좀 더 자세히 살펴보자. (발표 영상 37분 50초)
### Self-descriptive message
- 메시지 스스로 메시지에 대한 설명이 가능해야 한다.
- 서버가 변해서 메시지가 변해도 클라이언트는 그 메시지를 보고 해석이 가능하다.
- 확장 가능한 커뮤니케이션

오늘날 REST API는 과연 Self-descriptive message를 만족하고 있을까? 바뀐 메시지는 바로 대응을 할 수 있어야 한다.

### HATEOAS
- 하이퍼미디어(링크)를 통해 애플리케이션 상태 변화가 가능해야 한다.
- 링크 정보를 동적으로 바꿀 수 있다. (Versioning 할 필요 없이!)

클라이언트에게 응답을 받은 다음 상태로 전이하기 위해 Application의 서버의 응답을 위한 하이퍼미디어 정보를 전해준다. 그 하이퍼미디어 정보를 통해 다음 상태로 전이를 해야한다.

![image](https://user-images.githubusercontent.com/66561524/189549972-bf4e7525-15d9-4208-baf9-a047a03fb658.png)

**Self-descriptive message** : content type이 rss에 맞춰 있다고 했을 때, rss 태그의 내용은 해석이 가능하다. rss안의 있는 태그들은 태그 자체로서 내용의 설명을 담고 있기 때문이다.

**HATEOS** : 내용을 받고, 해당 링크 정보에 관심이 있을 때, 주어진 링크(<link></link>)를 통해 정보를 확인할 수 있기 때문이다.

응답만 가지고 응답을 해석할 수 있어야 한다.

### Github Issues
https://docs.github.com/en/rest/reference/issues
깃허브의 REST API가 베스트이다. Github는 헤더의 미디어 타입을 미리 정해놓았다. 미디어 타입을 정의해놓으면 해석이 가능하다. 또한 다음 상태로 전이할 수 있는 URL이 상당히 많이 첨부되어있는 것을 확인할 수 있다.

### Self-descriptive message 해결 방법 
- 방법 1: 미디어 타입을 정의하고 IANA에 등록하고 그 미디어 타입을 리소스 리턴할 때 Content-Type으로 사용한다.
- 방법 2: profile 링크 헤더를 추가한다. (발표 영상 41분 50초)
  - 브라우저들이 아직 스팩 지원을 잘 안해
대안으로 HAL의 링크 데이터에 profile 링크 추가 

 
### HATEOAS 해결 방법
- 방법1: 데이터에 링크 제공
  - 링크를 어떻게 정의할 것인가? HAL
- 방법2: 링크 헤더나 Location을 제공

ref. [백기선 - 스프링 기반 REST API 개발](https://www.inflearn.com/course/spring_rest-api)
