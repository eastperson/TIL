# Overview

자바스크립트는 싱글 스레드 기반의 언어입니다. 그래서 비동기적인 필요한 상황을 많이 겪게 됩니다. 그런데 잘 만들어진 사이트들을 뒤적거리다보면 하나의 페이지에서도 다양한 작업이 동시에 수행되고 있음을 확인 할 수 있습니다. 그만큼 사용자의 UX가 최적화가 되어있다고 볼 수 있습니다. 자바스크립트는 이러한 동시성(Concurrency)을 지원하기 위해 비동기 프로그래밍이 발전되어 있습니다.

비동기 프로그래밍을 위한 개념은 대표적으로 `이벤트 루프`, `콜백 패턴`, `프로미스`가 있습니다.

1. 이벤트 루프
2. 콜백 패턴
3. 프로미스

이벤트 루프는 연결된 이벤트 타깃을 추적하기 때문에 문제가 많습니다. 콜백 패턴도 마찬가지로 콜백 헬(콜백이 중첩되었을 때 발생하는 문제)이 발생할 수 있습니다. 이를 해결하는 프로미스는 비동기 연산의 순서를 마이크로 태스크(microtask)를 이용해서 기본 태스크 큐가 아닌 마이크로 태스크 큐에 추가를 합니다.  그렇게 연산의 순서를 임의로 지정할 수 있습니다.

마치 프로미스가 비동기 프로그래밍의 문제를 모두 해결할 것 같지만, 근본적인 문제가 있습니다. **'싱글 쓰레드의 단점'**입니다. 프로미스는 결국의 코드의 순서를 지정해서 순차적으로 수행하는 것이지 여러개의 작업을 동시에 수행하는 것이 아닙니다.

![image](https://user-images.githubusercontent.com/66561524/199025058-b9b37ec3-4f0e-45c1-a79a-c83c09962126.png)
병렬 처리

따라서 연산처리가 무겁고 복잡한 프로그래밍을 하기 위해선 '병렬 프로그래밍'이 필요합니다. 자바에서는 병렬 프로그래밍을 '멀티 쓰레드'를 사용해서 구현합니다. 싱글 스레드 언어인 자바스크립트는 병렬 프로그래밍을 위해 Web Worker를 사용합니다.

[자바스크립트와 이벤트 루프 : TOAST Meetup](https://meetup.toast.com/posts/89)

[비동기 프로그래밍과 병렬 프로그래밍의 차이점을 분명히 표현하는 방법은 무엇입니까?](https://qastack.kr/programming/6133574/how-to-articulate-the-difference-between-asynchronous-and-parallel-programming)

# Web Worker란?

## Web worker API - [TCP School](http://www.tcpschool.com/html/html5_api_webWorker)

웹 페이지에서 스크립트가 실행되면, 해당 웹 페이지는 실행 중인 스크립트가 종료될 때까지 응답 불가 상태가 됩니다.

web worker는 스크립트가 웹 페이지의 성능에 영향을 미치지 않도록 백그라운드에서 동작하게 해주는 자바스크립트입니다. 즉, web worker는 스크립트의 다중 스레드(multi-thread)를 지원합니다.

따라서 사용자가 웹 페이지를 이용하면서도, 동시에 시간이 오래 걸리는 자바스크립트 작업도 병행할 수 있도록 해줍니다.

# Web Worker란?

## Web worker API - [TCP School](http://www.tcpschool.com/html/html5_api_webWorker)

웹 페이지에서 스크립트가 실행되면, 해당 웹 페이지는 실행 중인 스크립트가 종료될 때까지 응답 불가 상태가 됩니다.

web worker는 스크립트가 웹 페이지의 성능에 영향을 미치지 않도록 백그라운드에서 동작하게 해주는 자바스크립트입니다. 즉, web worker는 스크립트의 다중 스레드(multi-thread)를 지원합니다.

따라서 사용자가 웹 페이지를 이용하면서도, 동시에 시간이 오래 걸리는 자바스크립트 작업도 병행할 수 있도록 해줍니다.

![image](https://user-images.githubusercontent.com/66561524/199025546-f7eecf70-43a6-438b-832a-0efa1bd599ae.png)
Web Worker

연산처리는 Worker에게 맡겨버리고 브라우저에서 다른 작업을 할 수 있습니다. 5초가 지난후 Worker가 일처리를 완료하면 계산된 로직의 결과를 업데이트합니다. 그러면 브라우저는 정상적으로 작동을 하면서도 반영된 DOM의 업데이트 결과를 확인할 수 있습니다. 자바의 쓰레드 개념과 동일하게 이러한 연산처리가 필요한 경우 Worker를 하나씩 늘려 동시에 여러 연산처리도 진행할 수 있습니다.

Worker는 작업을 수행하지만, DOM에 직접 접근할 수 없습니다. DOM 업데이트를 해주는 쓰레드는 오직 메인 쓰레드입니다. 따라서 Web Worker를 사용하는 경우에는 대부분의 연산처리를 워커에게만 맡기고 메인쓰레드와 Web Worker는 postMessage()를 통해 이벤트 처리와 결과값을 주고 받습니다. 메인쓰레드는 DOM 업데이트와 이벤트 리스너 기능을 담당합니다.

위에 나왔던 이벤트 루프와 연관지어 설명하면 싱글 스레드인 자바스크립트는 각각의 이벤트 루프가 있고, Worker를 생성하면 각각의 독립적인 이벤트 루프(Worker Event Loop)를 사용하게 됩니다. 따라서 다양한 작업을 병렬 구조로 처리할 수 있게 됩니다.

- [웹 워커 제약](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Functions_and_classes_available_to_workers)

[(HTML&DOM) 웹 워커 - 멀티 쓰레드 활용하기](https://www.zerocho.com/category/HTML&DOM/post/5a85672158a199001b42ed9c)

[Using Web Workers](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Using_web_workers)

[[Javascript] 쓰레드(웹 워커-Web worker)를 사용하는 방법](https://nowonbun.tistory.com/727)

# 서비스 워커(Service Worker)란?

일단 서비스 워커도 웹 워커와 같은 워커입니다.  따라서 브라우저가 백그라운드에서 실행하는 스크립트로, 웹 페이지와는 별개로 작동합니다. 또한 역시 DOM에 직접 접근하지 못합니다. 대신에 사용자가 웹 페이지와 상호작용하지 않는 기능들을 제공합니다. 예시로 푸시 알림과 백그라운드 동기화와 같은 기능을 제공하고 있습니다.

![image](https://user-images.githubusercontent.com/66561524/199025962-e5464a2e-7993-4bef-a272-83b29de125a4.png)
![image](https://user-images.githubusercontent.com/66561524/199025984-817865e8-03a6-47ce-82ce-1d72fbe7762f.png)

서비스 워커는 웹의 한계를 뛰어넘는 획기적인 기술입니다. 기본적으로 웹 브라우저는 HTTP 통신을 통해 서버에게 정보를 요청하고 받습니다. HTTP는 대단한 기술이지만 당연하게도 인터넷이라는 연결 배관이 필요합니다. 따라서 배관이 연결되지 않으면 요청을 보내지도 응답을 받을 수도 없습니다.

![image](https://user-images.githubusercontent.com/66561524/199026021-c07d18ac-3424-494b-beeb-a5a010ddc939.png)

하지만 서비스 워커를 사용하면 서버에 직접 정보를 요청하는 대신 서비스 워커에 필요한 내용을 전달합니다. 서비스 워커는 요청을 받아 직접 수행을 하게 됩니다. 수행이 마칠 때까지 사라지지도 않습니다. 따라서 웹페이지가 닫혀지더라도 주어진 임무를 수행하게 됩니다. 예를 들면 이미지 파일을 다운 받다가 50%쯤에서 브라우저를 닫으면 보통은 중단되기 마련입니다. 하지만 서비스 워커는 100%가 될 때까지 작업을 수행합니다.

또한 서비스 워커는 캐시(cache)와 상호작용을 합니다. fetch 이벤트의 중간자 역할로도 사용이 가능합니다. 사용자는 서비스 워커에게 정보를 캐시로 저장하라고 요청을 할 수 있습니다. 그 이후에 서비스 워커에게 정보를 요청하면 서비스 워커는 HTTP를 통해 정보 요청을 하지 않고 가지고 있는 캐시에서 자료를 꺼내 보여줍니다.  캐시가 삭제되지 않는 한 브라우저는 인터넷 연결 없이도 정보를 화면에 뿌려 보여줄 수 있습니다.

이와 같은 방식으로 서비스워커는 브라우저 창이 닫힌 상태(오프라인)에서도 푸시 알림, 백그라운드 동기화 등을 구현할 수 있습니다. 

다음은 크롬 개발자가 명시한 서비스 워커의 유의 사항입니다.

- 서비스 워커는 [자바스크립트 Worker](https://www.html5rocks.com/en/tutorials/workers/basics/)이므로, DOM에 직접 액세스할 수 없습니다. 대신에 서비스 워커는 [postMessage](https://html.spec.whatwg.org/multipage/workers.html#dom-worker-postmessage) 인터페이스를 통해 전달된 메시지에 응답하는 방식으로 제어 대상 페이지와 통신할 수 있으며, 해당 페이지는 필요한 경우 DOM을 조작할 수 있습니다.
- 서비스 워커는 프로그래밍 가능한 네트워크 프록시이며, 페이지의 네트워크 요청 처리 방법을 제어할 수 있습니다.
- 서비스 워커는 사용하지 않을 때는 종료되고 다음에 필요할 때 다시 시작되므로 서비스 워커의 **`onfetch`** 및 **`onmessage`** 핸들러의 전역 상태에 의존할 수 없습니다. 보관했다가 다시 시작할 때 재사용해야 하는 정보가 있는 경우 서비스 워커가 [IndexedDB API](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API)에 대한 액세스 권한을 가집니다.
- 서비스 워커는 프라미스를 광범위하게 사용하므로 프라미스에 대해 잘 모르는 경우 이 가이드를 읽는 것을 멈추고 [프라미스 소개](https://developers.google.com/web/fundamentals/getting-started/primers/promises?hl=ko)를 확인해 보세요.

[서비스 워커: 소개 | Web Fundamentals | Google Developers](https://developers.google.com/web/fundamentals/primers/service-workers?hl=ko)

[서비스 워커(Service Worker) 정체가 뭐니?](https://b.limminho.com/archives/1384)

[선비같은 개발자 : 네이버 블로그](https://blog.naver.com/z1004man/221761665845)
