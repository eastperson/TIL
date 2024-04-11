# Strangler fig 패턴

Strangler fig 패턴은 마틴 파울러(Martin Fowler)가 도입한 인기 있는 설계 패턴입니다. 마틴 파울러는 나무 위쪽 가지에 씨를 뿌리는 특정한 종류의 무화과에서 영감을 얻었죠. 기존 나무는 처음에는 새 무화과를 지탱하는 구조물이 됩니다. 그런 다음 무화과는 뿌리를 땅으로 내보내고, 점차 원래 나무를 감싸면서 자라 결국 스스로 선 새 무화과 나무만 남게 됩니다.

이 패턴은 일반적으로 특정 기능을 새 서비스로 대체하는 방식으로 모놀리식 애플리케이션을 마이크로서비스로 점진적으로 전환하는 데 사용됩니다. 목표는 기존 버전과 현대화된 새 버전이 공존하는 것입니다. 처음에는 기존 시스템이 새 시스템을 지원하며, 점차 새 시스템이 기존 시스템을 대체하게 됩니다. 이러한 지원을 통해 새 시스템을 확장하고 잠재적으로 기존 시스템을 완전히 교체할 수 있는 시간을 확보할 수 있습니다.

Strangler fig 패턴을 구현하여 모놀리식 애플리케이션에서 마이크로서비스로 전환하는 프로세스는 변환, 공존, 제거라는 세 단계로 구성됩니다.

- 변환: 기존 애플리케이션과 병렬로 포팅하거나 재작성하여 현대화된 구성 요소를 식별하고 생성합니다.
- 공존: 모놀리스 애플리케이션을 롤백에 대비해 보관합니다. 모놀리스 경계에 HTTP 프록시(예: Amazon API Gateway)를 통합하여 외부 시스템 직접 호출을 가로채고 트래픽을 현대화된 버전으로 리디렉션합니다. 이를 통해 기능을 점진적으로 구현할 수 있습니다.
- 제거: 트래픽이 기존 모놀리스에서 현대화된 서비스로 리디렉션되었으므로 모놀리스에서 기존 기능을 사용 중지합니다.

그림은 Strangler fig 패턴을 애플리케이션 아키텍처에 적용하여 모놀리스를 마이크로서비스로 분할하는 방법을 보여줍니다. 두 시스템 모두 병렬로 작동하지만 기능을 모놀리스 코드 베이스 외부로 이동시키고 새로운 기능으로 개선해야 합니다. 이러한 새로운 기능을 통해 필요에 가장 적합한 방식으로 마이크로서비스를 설계할 수 있습니다. 마이크로서비스로 모두 대체될 때까지 모놀리스에서 계속 기능을 제거해야 합니다. 이 시점에서 모놀리스 애플리케이션을 제거할 수 있습니다. 여기서 주목해야 할 요점은 모놀리스와 마이크로서비스가 일정 기간 동안 함께 작동할 것이라는 점입니다.
![image](https://github.com/eastperson/TIL/assets/66561524/3d8b8235-f9d4-4037-8c78-167dbd74edd4)

| 장점 | 단점 |
| :--: | :--: |
| - 서비스에서 하나 이상의 대체 서비스로 원활하게 마이그레이션할 수 있습니다. - 업데이트된 버전으로 리팩터링하는 동안 이전 서비스를 계속 사용할 수 있습니다. - 이전 서비스를 리팩터링하면서 새 서비스와 기능을 추가할 수 있습니다. - 패턴을 API 버전 관리에 사용할 수 있습니다. - 이 패턴을 업그레이드되지 않았거나 업그레이드되지 않을 솔루션의 기존 상호 작용에 사용할 수 있습니다. | - 복잡성이 낮고 크기가 작은 소규모 시스템에는 적합하지 않습니다. - 백엔드 시스템에 대한 요청을 가로채거나 라우팅할 수 없는 시스템에서는 사용할 수 없습니다. - 프록시 또는 파사드 레이어는 제대로 설계하지 않은 경우 단일 장애 지점이 되거나 성능 병목 현상이 발생할 수 있습니다. - 문제가 발생할 경우 작업을 수행하는 이전 방식으로 빠르고 안전하게 되돌리려면 리팩터링된 각 서비스에 대한 롤백 계획이 필요합니다. |



![image](https://github.com/eastperson/TIL/assets/66561524/6d4f412b-3200-43ca-b194-45451797e7ec)
https://microservices.io/patterns/refactoring/strangler-application.html


# Reference
- [StranglerFigApplication](https://martinfowler.com/bliki/StranglerFigApplication.html)
- [Strangler fig 패턴 - AWS](https://docs.aws.amazon.com/ko_kr/prescriptive-guidance/latest/modernization-decomposing-monoliths/strangler-fig.html)