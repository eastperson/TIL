# Overview

[Spring 6의 새로운 HTTP Interface와 3 가지 REST Clients 라이브 코딩](https://www.youtube.com/watch?v=Kb37Q5GCyZs)

- Http Interface 를 구현한 3가지 라이브러리를 활용
- 환율 정보를 가져오는 api로 테스트
    - https://open.er-api.com/v6/latest
    - 기준환율 USD

# RestTemplate

- RestTemplate은 하나만들어서 사용하는 것을 권장
    - spring bean 등으로 등록
- getForObject() 로 받으면 body를 가져올 수 있고 entity로 받으면 헤더 등의 정보를 가져올 수 있다.

# WebClient

- 메서드 체이닝을 사용해서 다음에 오는 순서를 외우지 않아도 쉽게 사용할 수 있다. 이러한 방법을 모던하다고 부르는 것 같다.
- webclient의 exchange는 retrieve이다.
- 응답이 Mono로 받으면 reactive java로 비동기 방식으로 가져온다. block() 메서드로 붙여주면 동기 방식으로 받아온다.
- webflux 기반이다.

webclient가 모던하고 비동기 방식으로 사용하는 로직인데 굳이 webflux를 안쓰면 resttemplate를 사용하는 게 좋다.

# Http Interface

[HTTP Interface in Spring 6 | Baeldung](https://www.baeldung.com/spring-6-http-interface)

[Integration](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#rest-http-interface)

- Spring6에서 새로 추가되었다.
- open feign을 대체할 수 있을 듯 하다.
- 사용이 직관적이다.
- webclient 기반이다.
