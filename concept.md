* istio란?

클라우드 플랫폼은 이를 사용하는 조직에 다양한 이점을 제공한다. 
그러나 클라우드를 채택하여 DevOps 팀에 부담을 줄 수 있다는 것을 부인할 수는 없다. 
개발자는 이식성을 위해 마이크로 서비스를 사용하여 아키텍처를 설계해야 하지만, 
운영자는 매우 큰 하이브리드 및 멀티 클라우드 배포를 관리하고 있다. 
Istio를 사용하면 서비스를 연결, 보안, 제어 및 관찰 할 수 있다.

높은 수준에서 Istio는 이러한 배포의 복잡성을 줄이고 개발 팀의 부담을 덜어준다. 
기존의 분산 응용 프로그램에 투명하게 계층화되는 완전히 오픈 소스 서비스 메시이다. 
또한 모든 로깅 플랫폼 또는 원격 측정 또는 정책 시스템에 통합 할 수있는 API를 포함한 플랫폼이다. 
Istio의 다양한 기능을 사용하면 분산 마이크로 서비스 아키텍처를 성공적이고 효율적으로 실행할 수 있으며 
마이크로 서비스를 안전하게 연결하고 모니터링 할 수있는 균일 한 방법을 제공한다.

* 서비스 매시란?

Istio는 "단일 애플리케이션이 분산 마이크로 서비스 아키텍처로 전환함에 따라" 
개발자와 운영자가 겪는 문제를 해결한다. 
방법을 알아 보려면 Istio의 서비스 메시를 자세히 살펴본다.

서비스 메시라는 용어는 
이러한 응용 프로그램과 이들 간의 상호 작용을 구성하는 마이크로 서비스 네트워크를 설명하는 데 사용된다.
서비스 메시의 크기와 복잡성이 증가함에 따라 "이해하고 관리하기가" 더 어려워 진다.
서비스 메시를 위한 요구사항에는 감지,로드 밸런싱, 장애복구, 메트릭 및 모니터링이 포함될 수 있다.
서비스 메시에는 종종 A/B 테스트, 카나리아 롤아웃, 속도 제한, 액세스 제어 및 엔드 투 엔드 인증과 같은
보다 복잡한 운영 요구사항이 있다.

Istio는 서비스 메시 전체에 대한 행동 통찰력과 운영 제어 기능을 제공하여 
마이크로 서비스 애플리케이션의 다양한 요구 사항을 충족시키는 완벽한 솔루션을 제공한다.

* 왜 istio를 사용하는가?

Istio를 사용하면, 서비스 코드의 코드를 "거의 또는 전혀" 변경하지 않고도 
로드 밸런싱, 서비스 간 인증, 모니터링 등을 통해
배포 된 서비스 네트워크를 쉽게 만들 수 있다.
마이크로 서비스 간의 모든 네트워크 통신을 차단하는 환경에 
특수 사이드카 프록시를 배치하여 서비스에 Istio 지원을 추가 한 후 
다음과 같은 제어 기능을 사용하여 Istio를 구성 및 관리한다.

   -> HTTP, gRPC, WebSocket 및 TCP 트래픽에 대한 자동 로드 밸런싱.
   -> 풍부한 라우팅 규칙, 재시도, 장애 조치 및 결함 주입을 통해 트래픽 동작을 세밀하게 제어.
   -> 액세스 제어, 속도 제한 및 할당량을 지원하는 플러그 가능 정책 계층 및 구성 API
   -> 클러스터 수신 및 송신을 포함하여 클러스터 내의 모든 트래픽에 대한 자동 메트릭, 로그 및 추적.
   -> 강력한 ID 기반 인증 및 권한 부여를 통해 클러스터에서 안전한 서비스 간 통신.

Istio는 확장성을 위해 설계되었으며 다양한 구축 요구를 충족한다. 
다음 다이어그램과 같이 메시 트래픽을 인터셉트하고 구성하여이를 수행한다.

![istion architecture](https://istio.io/latest/docs/ops/deployment/architecture/arch.svg)