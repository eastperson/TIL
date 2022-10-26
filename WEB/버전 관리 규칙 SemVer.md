# Overview

---

소프트웨어 관리를 하다보면 ‘의존성 지옥’ 문제에 빠지게 된다. 의존성이 높은 시스템에서 명세를 엄격하게하면 모든 패키지의 버전을 업그레이드 해야 배포할 수 있는 경우가 생긴다. 반대로 느슨하게 관리를 하면 버전이 엉켜서 호환이 안맞는 경우가 생길 수있다.

# 버전 관리의 기준 SemVer
![image](https://user-images.githubusercontent.com/66561524/198070705-549f96d2-d238-4c5f-94f1-33b9070751c0.png)

**Semantic Versioning Specification**

`x.y.z`의 체계를 따르는 버전 전략이다.

- MAJOR: 이전 버전과 호환되지 않는 API 변경
    - 주 버전으로 0으로 시작되면 개발 중임을 의미한다. 즉 불안정한(unstable) 버전이므로 사용 중간에 사용법이 지속적으로 바뀔 수 있다.
    - 1보다 클 경우 각 버전 내에서 API 명세가 많이 바뀌지 않으며 바뀌더라도 이전 버전에 대한 호환성이 보장될 것임을 예상할 수 있다.
- MINOR: 이전버전과 호환되면서 기능의 변경이나 추가된 경우
    - 직전 버전의 API 명세를 그대로 사용하더라도 z 버전의 업데이트보다는 큰 변경이 있을 때 올린다.
- PATCH: 버그 수정
    - MAJOR 혹은 MINOR 버전이 올라갈경우 0으로 초기화된다.

가령 회사에서 사용하는 Semver를 준수하는 외부 라이브러리가 2.7.3에서 2.7.4로 변경이 되었을 때, 버그를 수정했다고 볼 수 있고 2.8.0이면 기능의 변경, 3.0.0이면 호환이 안되는 새로운 버전의 개발이 되었다고 보면 된다. 

Patch 버전 뒤에는 하이픈(-)이나 마침표(.)로 식별자를 붙여 배포 전의 버전을 네이밍할 수 있다.

- `1.0.0-alpha`
- `1.0.0-alpha.1`

하지만 결국의 Semver의 버전업 기준은 주관적이다. 개발 조직에 따라 표준이 제각각인 것이다. 즉 실제 코딩에서의 변경에 대한 의미를 개발자가 주관적으로 판단하고 수동적으로 관리를 해야한다. 문서화나 버저닝 작업을 수행하는 소프트웨어 개발자는 이에 대한 인식이 중요하다.

## 라인의 HeadVer
[https://github.com/line/headver](https://github.com/line/headver)

![image](https://user-images.githubusercontent.com/66561524/198070813-36585a90-ed3a-4795-aa88-e536f87239b7.png)

기존 Semver는 수동으로 버전을 관리하는 지점이 major, minor, patch 이렇게 3가지이다. 라인은 이 문제를 해결하기 위 headver 개념을 도입했다. header는 버전의 단계를 3가지로 가져가지만 아래와 같이 구분한다.

```java
{head} - manual. Zero-based number.
{yearweek} - automatic. 2-digit for year and 2-digit for week number.
{build} - automatic. Incremental number from a build server.
```

따라서 수동으로 개발자가 관리하는 포인트는 head만 사용하고 나머지는 자동화로 만들어 놓았다. headver의 뜻은 아래와 같다.

The name `HeadVer` stands for **"only head number is enough to maintain!"**
 because it only allows to set the first number manually, and rest numbers are automatic

# Reference

---

[버저닝 전략 어떻게 할 것인가](https://jypthemiracle.medium.com/%EB%B2%84%EC%A0%80%EB%8B%9D-%EC%A0%84%EB%9E%B5-%EC%96%B4%EB%96%BB%EA%B2%8C-%ED%95%A0-%EA%B2%83%EC%9D%B8%EA%B0%80-63a93a9cb960)

[https://github.com/line/headver](https://github.com/line/headver)

[쳬계적인 버전관리, Semver - 지어소프트 개발 가이드](http://developer.gaeasoft.co.kr/development-guide/knowledge/management/systematic-versioning-with-semver/)

[Semantic Versioning 2.0.0](https://semver.org/)
