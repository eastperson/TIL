# Overview

2013년 [Jacques Dubray는](https://web.archive.org/web/20180202134605/https://www.ebpml.org/blog2/index.php/2013/11/25/understanding-the-costs-of-versioning) 웹 API 버전 관리 전략을 세 가지 범주(Knot, Point-to-Point 및 Compatible)로 분류하고 각 범주에서 시간 경과에 따른 비용을 측정하는 방법에 대한 공식을 작성했다.

> API 변경시 API 사용자에게 추가로 소요되는 비용은 개발자에게는 크게 와닿지 않을 수도 있다. API 사용자에게 아무런 즉각적인 비즈니스 가치도 만들어주지 못하는 변경은 기존 API에 변경이 없을것이라고 예상한 API 사용자에게 단순히 비용이 아니라 프로젝트 계획 붕괴, 예산 초과등으로 이어질 수 있는 큰 위험이라는 점을 이해하는 것이 핵심이다. - 장자끄 뒤브레(Jacques Dubray)
> 

# API 변경 유형 3가지 by 뒤브레

- 매듭(knot) : 모든 API 사용자가 단 하나의 버전에 묶여 있다. API가 변경되면 모든 사용자도 함께 변경을 반영해야 하므로 여파를 몰고 온다.

![image](https://user-images.githubusercontent.com/66561524/197806827-9b30cf00-5fb2-40ba-82cb-8b34b3f4a93f.png)

- 점대점(point-to-point) : 사용자마다 별도의 API 서버를 통해 API를 제공한다. 사용자별로 적절한 시점에 API를 변경할 수 있다.

![image](https://user-images.githubusercontent.com/66561524/197806871-d4a5f59c-6c7d-4ad2-b3b1-f4b4c0909934.png)

- 호환성 버저닝(compatible versioning) : 모든 사용자가 호환 가능한 하나의  API 서비스 버전을 사용한다.

![image](https://user-images.githubusercontent.com/66561524/197806927-5c0028b4-1ae1-4a20-8d26-89eb943cb4b1.png)

# 성능 비교

![image](https://user-images.githubusercontent.com/66561524/197806976-063ee77b-66a1-495a-a283-f563a95789c5.png)

- x축은 진화하는 API의 버전을 나타낸다.
- Y축은 API 변경 대응 비용을 나타낸다.

매듭 방식의 비용은 급속도로 증가한다. 변경하는 양이 적더라도 영향은 엄청나다. API 사용자는 변경된 API에 포함된 기능을 사용하지 않아도 강제적으로 업그레이드를 해야한다.

점대점방식은 여전히 적지 않은 비용이 소요되고 증가속도가 가파르다. API 사용자에게 미치는 영향을 줄지만 개발팀은 그만큼 많은 API를 제공해야 한다.

호환성 방식은 가장 효율적으로 보인다. 동일한 API에 기존 사용자도 그대로 사용하고 추가된 기능도 사용할 수 있다. 하위 호환성을 유지하는 방식의 서비스는 하이퍼 미디어를 적용해서 만들 수 있다.

# Reference

[API Change Strategy | Nordic APIs |](https://nordicapis.com/api-change-strategy/)

[스프링 부트 실전 활용 마스터](https://search.shopping.naver.com/book/catalog/32491903647)
