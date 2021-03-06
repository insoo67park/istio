Traffic Management
---
 
Istio의 트래픽 라우팅 규칙을 사용하면 
서비스 간의 트래픽 흐름과 API 호출을 쉽게 제어 할 수 있다.   
Istio는 Circuit Breaker, Timeout 및 Retry와 같은 서비스 수준 속성 구성을 단순화하고   
A/B 테스트, 카나리아 Roll out 및 백분율 기반 트래픽 분할을 통한   
단계적 출시와 같은 중요한 작업을 쉽게 설정할 수 있다.   
또한 종속 서비스 또는 네트워크의 장애에 대비하여   
애플리케이션을 보다 강력하게 만드는 데 도움이 되는 기본 장애복구 기능을 제공한다.   
    
stio의 트래픽 관리 모델은   
서비스와 함께 배포 된 Envoy 프록시에 의존한다.   
메시 서비스에 주고 받는 모든 트래픽 (데이터 평면 트래픽)은   
Envoy를 통해 프록시 되므로   
서비스를 변경하지 않고도 메시 주변의 트래픽을 쉽게 지시하고 제어 할 수 있다.   
이 가이드에 설명 된 기능의 작동 방식에 대한 자세한 내용은   
아키텍처 개요에서 Istio의 트래픽 관리 구현에 대해 자세히 알아볼 수 있다.   
이 안내서의 나머지 부분에서는 Istio의 트래픽 관리 기능을 소개한다.   


Istio 트래픽 관리 소개
---

메시 내에서 트래픽을 전달하려면   
Istio가 모든 엔드 포인트의 위치와 서비스가 속한 서비스를 알아야 한다.    
자체 서비스 레지스트리를 채우려면 Istio가 서비스 검색 시스템에 연결한다.  
   
예를 들어 Kubernetes 클러스터에 Istio를 설치 한 경우,   
Istio는 해당 클러스터의 서비스 및 엔드 포인트를 자동으로 감지한다.   

Envoy 프록시는   
이 서비스 레지스트리를 사용하여 트래픽을 관련 서비스로 보낼 수 있다.    

대부분의 마이크로 서비스 기반 애플리케이션에는 서비스 트래픽을 처리하기 위해   
각 서비스 워크로드의 여러 인스턴스가 있다 ( 때로는 로드 밸런싱 풀 이라고도 한다)    
Envoy 프록시는 기본적으로 Round Robin 모델을 사용하여,   
각 서비스의 로드 밸런싱 풀에 트래픽을 분배한다.   
이 모델에서는 요청이 각 풀 멤버에게 차례로 전송되어   
각 서비스 인스턴스가 요청을 받으면 풀의 맨 위로 돌아간다.   
   
Istio의 기본 서비스 검색 및 로드 밸런싱은 동작하는 서비스 메시를 제공하지만,   
Istio가 할 수있는 모든 것과는 다소 거리가 있다.    
많은 경우 메쉬 트래픽에 발생하는 상황을 보다 세밀하게 제어 할 수 있다.   
특정 비율의 트래픽을 A/B 테스트의 일부로 (새로운 버전의 서비스로) 보내거나   
특정 서비스 인스턴스 하위 집합의 트래픽에 다른로드 밸런싱 정책을 적용 할 수 있다.   
메시로 들어 오거나 나가는 트래픽에 특수 규칙을 적용하거나   
메시의 외부 종속성을 서비스 레지스트리에 추가 할 수도 있다.   
Istio의 트래픽 관리 API를 사용하여 Istio에 자체 트래픽 구성을 추가하여이 모든 작업을 수행 할 수 있다.   
   
다른 Istio 구성과 마찬가지로 API는 Kubernetes 사용자 지정 리소스 정의 (CRD)를 사용하여 지정되며,    
예에서 볼 수 있듯이 YAML을 사용하여 구성 할 수 있다.   
이 가이드의 나머지 부분에서는 각 트래픽 관리 API 리소스와 리소스로 수행 할 수있는 작업에 대해 설명한다.   
이러한 리소스는 다음과 같습니다   
   
* 가상 서비스
* 대상 규칙
* 게이트웨이
* 서비스 항목
* 사이드카
   
이 가이드는 또한 API 리소스에 내장된 일부 네트워크 탄력성 및 테스트 기능에 대한 개요를 제공한다.   
   
가상 서비스
---

가상 서비스는 대상 규칙과 함께 Istio의 트래픽 라우팅 기능의 핵심 구성요소이다.    
가상 서비스를 사용하면   
Istio 및 사용자 플랫폼에서 제공하는 기본 연결 및 검색을 기반으로   
요청이 Istio 서비스 메시 내의 서비스로 라우팅되는 방식을 구성 할 수 있다.    
각 가상 서비스는 순서대로 평가되는 일련의 라우팅 규칙으로 구성되어 있어    
Istio가 각각의 지정된 요청을   
가상 서비스에 대한 메시 내의 특정 실제 대상과 일치시킬 수 있다.   
메시는 여러 가상 서비스를 요구하거나 사용 사례에 따라 필요하지 않을 수 있다.
   
가상 서비스를 사용하는 이유
---
가상 서비스는 Istio의 트래픽 관리를 "유연하고 강력하게 만드는 데" 핵심적인 역할을 한다.   
대상 워크로드에서는 클라이언트로부터의 요청 위치를 강력하게 분리하여 이를 수행한다.   
가상 서비스는   
트래픽을 해당 워크로드로 보내기 위해   
다른 트래픽 라우팅 규칙을 지정하기 위한 다양한 방법을 제공한다.   
   
이것이 왜 그렇게 유용한가요?   
가상 서비스가 없으면   
Envoy는 모든 서비스 인스턴스간에 Round Robin 부하 분산을 사용하여 트래픽을 분산시킨다.   
워크로드에 대해 알고있는 내용으로, 이 동작을 개선 할 수 있다.   
예를 들어, 일부는 다른 버전을 나타낼 수 있다.   
이 기능은   
A/B 테스트에서 유용 할 수 있다.   
A/B 테스트에서는 여러 서비스 버전에서 백분율을 기반으로 트래픽 경로를 구성하거나   
내부 사용자의 트래픽을 특정 인스턴스 집합으로 보낼 수 있다.

가상 서비스를 사용하면   
하나 이상의 호스트 이름에 대한 트래픽 동작을 지정할 수 있다.   
가상 서비스의 트래픽을 적절한 대상으로 보내는 방법을 Envoy에게 알려주는 라우팅 서비스를 가상 서비스에서 사용한다.   
경로 목적지는 동일한 서비스 또는 완전히 다른 서비스의 버전 일 수 있다.   
   
일반적인 사용 사례는   
서비스 하위 집합으로 지정된 다른 버전의 서비스로 트래픽을 보내는 것이다.   
클라이언트는 단일 엔터티인 것처럼 가상 서비스 호스트에 요청을 보내고   
Envoy는 가상 서비스 규칙에 따라 트래픽을 다른 버전으로 라우팅한다.   
예를 들어 "20%가 새로운 버전으로 이동" 또는 "이 사용자들로부터의 요청은 버전 2”로 간다.   
예를 들어 새로운 서비스 버전으로 전송되는 트래픽의 비율을 점차적으로 증가시키는   
카나리아 롤아웃을 만들 수 있다.   
트래픽 라우팅은 인스턴스 배포와 완전히 별개이므로   
새 서비스 버전을 구현하는 인스턴스 수는 트래픽 라우팅을 전혀 참조하지 않고도   
트래픽로드에 따라 확장 및 축소 할 수 있다.    
반대로 Kubernetes와 같은 컨테이너 오케스트레이션 플랫폼은   
인스턴스 스케일링을 기반으로 한   
트래픽 분배만 지원하므로 빠르게 복잡해진다.   
Istio를 사용하여 카나리아 배포에서   
가상 서비스가 카나리아 배포를 지원하는 방법에 대해 자세히 읽을 수 있다.  
   
가상 서비스를 통해 다음을 수행 할 수 있습니다.   

* 단일 가상 서비스를 통해 여러 애플리케이션 서비스를 처리할 수 있다. 
예를 들어 메시가 Kubernetes를 사용하는 경우  
특정 네임 스페이스의 모든 서비스를 처리하도록 가상 서비스를 구성 할 수 있다.   
단일 가상 서비스를 여러 "실제"서비스에 매핑하면 서비스 소비자가 전환에 적응할 필요 없이   
단일 응용 프로그램을 개별 마이크로 서비스로 구성된 복합 서비스로 쉽게 전환 할 수 있다.   
라우팅 규칙은 "monoli.com의 이러한 URI에 대한 호출은 마이크로 서비스 A로 이동"등을 지정할 수 있다.   
아래 예제 중 하나에서 이것이 어떻게 작동하는지 볼 수 있습니다.
   
* 게이트웨이와 결합하여 트래픽 규칙을 구성하여 수신 및 송신 트래픽을 제어하십시오.   
   
경우에 따라 서비스 하위 집합을 지정하는 위치에서 이러한 기능을 사용하도록 대상 규칙을 구성해야 한다.    
별도의 개체에 서비스 하위 집합 및 기타 대상별 정책을 지정하면   
가상 서비스간에 이를 명확하게 재사용 할 수 있다.   
다음 섹션에서 대상 규칙에 대한 자세한 내용을 확인할 수 있다.

가상 서비스 예제
---

다음 가상 서비스는 특정 사용자의 요청에 따라 요청을 다른 버전의 서비스로 라우팅한다.   

<pre>
<code>
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
  - reviews
  http:
  - match:
    - headers:
        end-user:
          exact: jason
    route:
    - destination:
        host: reviews
        subset: v2
  - route:
    - destination:
        host: reviews
        subset: v3
</code>
</pre>

host field
---
호스트 필드에는 가상 서비스의 호스트, 즉 이러한 라우팅 규칙이 적용되는 사용자가 주소를 지정할 수있는 대상이 나열된다.   
이것은 클라이언트가 서비스에 요청을 보낼 때 사용하는 주소이다.
<pre>
<code>
hosts:
- reviews
</code>
</pre>

가상 서비스 호스트 이름은 IP 주소, DNS 이름 또는 플랫폼에 따라 암시 적 또는 명시 적으로 FQDN (정규화 된 도메인 이름)으로 확인되는 짧은 이름 (예 : Kubernetes 서비스 짧은 이름) 일 수 있다.   
와일드 카드 ( "*") 접두사를 사용하여 일치하는 모든 서비스에 대해 단일 라우팅 규칙 집합을 만들 수 있다.   
가상 서비스 호스트는 실제로 Istio 서비스 레지스트리의 일부일 필요는 없으며   
단순히 가상 대상이다.   
이를 통해 메시 내부에 라우팅 가능한 항목이없는 가상 호스트의 트래픽을 모델링 할 수 있다.

## Routing Rule

http 섹션에는 호스트 필드에 지정된 대상으로 전송 된 
HTTP/1.1, HTTP2 및 gRPC 트래픽을 라우팅하기 위한 일치 조건, 조치를 설명하는 가상 서비스의 라우팅 규칙이 포함되어 있다    
(tcp 및 tls 섹션을 사용하여 라우팅을 구성 할 수도 있음)    
TCP 및 종료되지 않은 TLS 트래픽에 대한 규칙). 라우팅 규칙은   
사용 사례에 따라 트래픽을 이동하려는 대상과 0 개 이상의 일치 조건으로 구성된다.

### Match condition

예제의 첫 번째 라우팅 규칙에는 조건이 있으므로 일치 필드로 시작한다.   
이 경우이 라우팅이 사용자 "jason"의 모든 요청에 적용되기를 원하므로 헤더, 최종 사용자 및 정확한 필드를 사용하여 적절한 요청을 선택한다.

<pre>
<code>
- match:
   - headers:
       end-user:
         exact: jason
</code>
</pre>
### Destination
경로 섹션의 목적지 필드는   
이 조건과 일치하는 트래픽의 실제 목적지를 지정한다.   
가상 서비스의 호스트와 달리   
데스티네이션 호스트는 Istio의 서비스 레지스트리에 존재하는 실제 대상이어야 한다.   
그렇지 않으면 Envoy는   
트래픽을 어디로 보낼지 알 수 없다.   
프록시가 있는 메시 서비스이거나 서비스 항목을 사용하여 추가 된 비 메시 서비스 일 수 있다.   
이 경우 Kubernetes에서 실행 중이며 호스트 이름은 Kubernetes 서비스 이름이다.

<pre>
<code>
route:
- destination:
    host: reviews
    subset: v2
</code>
</pre>

이 페이지와이 페이지의 다른 예제에서는 단순성을 위해 데스티네이션 호스트에 Kubernetes 축약 형 이름을 사용한다.   
이 규칙이 평가되면 Istio는 라우팅 규칙이 포함 된 가상 서비스의 네임 스페이스를 기반으로   
도메인 접미사를 추가하여 호스트의 정규화 된 이름을 얻는다.   
예제에서 짧은 이름을 사용한다는 것은 원하는 네임 스페이스에서 이름을 복사하여 사용해 볼 수 있다는 의미이다.

이와 같은 짧은 이름을 사용하면   
데스티네이션 호스트와 가상 서비스가 실제로 동일한 Kubernetes 네임 스페이스에있는 경우에만 작동한다.   
Kubernetes 짧은 이름을 사용하면 구성이 잘못 될 수 있으므로   
프로덕션 환경에서 정규화 된 호스트 이름을 지정하는 것이 좋다.

데스티네이션 섹션은   
이 규칙의 조건과 일치하는 요청을 원하는   
Kubernetes 서비스의 하위 집합 (이 경우 v2라는 하위 집합)도 지정한다.   
아래의 대상 규칙 섹션에서 서비스 하위 세트를 정의하는 방법을 볼 수 있다.

### 라우팅 규칙 우선 순위
라우팅 규칙은 위에서 아래로 순차적으로 평가되며   
가상 서비스 정의의 첫 번째 규칙에 가장 높은 우선 순위가 부여된다.   
이 경우 첫 번째 라우팅 규칙과 일치하지 않는 항목을   
두 번째 규칙에 지정된 기본 대상으로 이동하려고 한다.   
이 때문에 두 번째 규칙에는 일치 조건이 없으며 트래픽을 v3 하위 집합으로 보낸다.
<pre>
<code>
route:
  - destination:
      host: reviews
      subset: v3
</code>
</pre>
가상 서비스에 대한 트래픽이   
항상 하나 이상의 일치하는 경로를 갖도록   
각 가상 서비스의 마지막 규칙으로   
이와 같은 기본 "무조건"또는 가중치 기반 규칙 (아래 설명)을 제공하는 것이 좋다.

### 라우팅 규칙에 대한 추가 정보
위에서 본 것처럼 라우팅 규칙은   
특정 트래픽 하위 집합을 특정 대상으로 라우팅하는 강력한 도구이다.   
트래픽 포트, 헤더 필드, URI 등에서 일치 조건을 설정할 수 있다.   
예를 들어, 이 가상 서비스를 통해 사용자는 마치 http://bookinfo.com/에서   
더 큰 가상 서비스의 일부인 것처럼   
등급과 리뷰의 두 가지 개별 서비스로 트래픽을 보낼 수 있다.   
가상 서비스 규칙은 요청 URI를 기반으로 트래픽을 일치시키고 요청을 해당 서비스로 보낸다.
<pre>
<code>
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: bookinfo
spec:
  hosts:
    - bookinfo.com
  http:
  - match:
    - uri:
        prefix: /reviews
    route:
    - destination:
        host: reviews
  - match:
    - uri:
        prefix: /ratings
    route:
    - destination:
        host: ratings
</code>
</pre>
일부 Match 조건의 경우 정확한 값, prefix 또는 regex을 사용하여 선택하도록 선택할 수도 있다.

AND 조건은 동일한 일치 블록에 여러 개의 March 조건을 추가하거나   
조건에 따라 동일한 규칙에 여러 일치 블록을 추가 할 수 있다.    
특정 가상 서비스에 대해 여러 라우팅 규칙을 가질 수도 있다.    
이를 통해 단일 가상 서비스 내에서 원하는대로 라우팅 조건을 복잡하거나 간단하게 만들 수 있다.    
일치 조건 필드 및 가능한 값의 전체 목록은 HTTPMatchRequest 참조에서 찾을 수 있다.
   
일치 조건을 사용하는 것 외에도 "중량"백분율로 트래픽을 분배 할 수 있다. 
이는 A/B 테스트 및 카나리아 Rollout에 유용하다.
<pre>
<code>
spec:
  hosts:
  - reviews
  http:
  - route:
    - destination:
        host: reviews
        subset: v1
      weight: 75
    - destination:
        host: reviews
        subset: v2
      weight: 25
</code>
</pre>
라우팅 규칙을 사용하여 트래픽에 대한 일부 작업을 수행 할 수도 있다 
예를 들면
* 헤더를 추가하거나 제거한다.
* URL을 다시 작성한다.
* 이 대상으로의 통화에 대한 재시도 정책을 설정한다.   

사용 가능한 조치에 대한 자세한 정보는 HTTPRoute 참조를 참조한다.

Destination Rule
---

가상 서비스와 함께 데스티네이션 규칙은 Istio의 트래픽 라우팅 기능의 핵심 부분이다.    
(트래픽을 지정된 대상으로 라우팅하는 방법으로 가상 서비스를 생각한 다음) 데스티네이션 규칙을 사용하여    
해당 대상에 대한 트래픽 발생을 구성 할 수 있다.    
데스티네이션 규칙은 가상 서비스 라우팅 규칙이 평가 된 후에 적용되므로 트래픽의 "실제"대상에 적용된다.

특히 데스티네이션 규칙을 사용하여 지정된 서비스 인스턴스를 버전별로 그룹화하는 등   
명명된 서비스 하위 집합을 지정한다.    
그런 다음 가상 서비스의 라우팅 규칙에서, 이러한 서비스 하위 세트를 사용하여   
서비스의 다른 인스턴스에 대한 트래픽을 제어 할 수 있다.

또한 대상 규칙을 사용하면   
전체 대상 서비스 또는 선호하는 로드 밸런스 조정 모델, TLS 보안 모드 또는 회로 차단기 설정과 같은    
특정 서비스 하위 집합을 호출 할 때 Envoy의 트래픽 정책을 사용자 지정할 수 있다.    
대상 규칙 참조에서 대상 규칙 옵션의 전체 목록을 볼 수 있다.

### 로드 밸런싱 옵션
기본적으로 Istio는 라운드 로빈 로드 밸런싱 정책을 사용하며,   
여기서 인스턴스 풀의 각 서비스 인스턴스는 차례로 요청을 받는다.   
Istio는 또한 다음 모델을 지원한다.   
이 모델은 특정 서비스 또는 서비스 하위 집합에 대한 요청의 대상 규칙에서 지정할 수 있다.

- 무작위 : 요청이 무작위로 풀의 인스턴스로 전달됩니다.
- 가중 : 요청이 특정 백분율에 따라 풀의 인스턴스로 전달됩니다.
- 최소 요청 : 요청 수가 가장 적은 인스턴스로 요청이 전달됩니다.   
각 옵션에 대한 자세한 내용은 Envoy로드 균형 조정 설명서를 참조한다.

### 대상 규칙 예
다음 예제 대상 규칙은로드 균형 조정 정책이 다른 my-svc 대상 서비스에 대해 서로 다른 세 가지 하위 집합을 구성한다.   
<pre>
<code>
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: my-destination-rule
spec:
  host: my-svc
  trafficPolicy:
    loadBalancer:
      simple: RANDOM
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
    trafficPolicy:
      loadBalancer:
        simple: ROUND_ROBIN
  - name: v3
    labels:
      version: v3
</code>
</pre>
각 하위 집합은 하나 이상의 레이블을 기반으로 정의되며   
Kubernetes에서는 포드와 같은 개체에 연결된 키/값 쌍이다.   
이 레이블은 Kubernetes 서비스 배포시 메타 데이터로 다른 버전을 식별하기 위해 적용된다.  
   
하위 집합을 정의 할 뿐만 아니라,   
이 데스티네이션 규칙에는 이 데스티네이션의 모든 하위 집합에 대한   
기본 트래픽 정책과 해당 하위 집합에 대해서만 재정의하는 하위 집합 특정 정책이 있다.   
서브 세트 필드 위에 정의 된 기본 정책은 v1 및 v3 서브 세트에 간단한 랜덤로드 밸런서를 설정한다.   
v2 정책에서 라운드 로빈로드 밸런서는 해당 하위 집합의 필드에 지정된다.   
   
## 게이트웨이
게이트웨이를 사용하여 메시의 인바운드 및 아웃 바운드 트래픽을 관리함으로써   
메시에 들어가거나 나가는 트래픽을 지정할 수 있다.   
게이트웨이 구성은 서비스 워크로드와 함께 실행되는 사이드카 Envoy 프록시가 아니라   
메시 에지에서 실행되는 독립형 Envoy 프록시에 적용된다.   
   
Kubernetes Ingress API와 같이 시스템으로 들어오는 트래픽을 제어하는 다른 메커니즘과 달리   
Istio 게이트웨이를 사용하면 Istio 트래픽 라우팅의 모든 기능과 유연성을 사용할 수 있다.   
Istio의 게이트웨이 리소스를 사용하면 노출 할 포트, TLS 설정 등과 같은 레이어 4-6 로드 밸런싱 속성 만 구성 할 수 있기 때문에   
이 작업을 수행 할 수 있다.    
그런 다음 L7 (Application Layer Traffic Routing)을 동일한 API 리소스에 추가하는 대신    
일반 Istio 가상 서비스를 게이트웨이에 바인딩한다.    
이를 통해 기본적으로 Istio 메시의 다른 데이터 플레인 트래픽과 같은 게이트웨이 트래픽을 관리 할 수 있다.   
   
게이트웨이는 주로 수신 트래픽을 관리하는 데 사용되지만    
송신 게이트웨이를 구성 할 수도 있다.    
송신 게이트웨이를 사용하면 메시를 떠나는 트래픽에 대한 전용 이탈 노드를 구성하여    
외부 네트워크에 액세스 할 수 있거나 액세스 해야 하는 서비스를 제한하거나   
송신 트래픽을 안전하게 제어하여   
메시에 보안을 추가 할 수 있다.   
게이트웨이를 사용하여 순전히 내부 프록시를 구성 할 수도 있다.

Istio는 사용할 수있는 
사전 구성된 게이트웨이 프록시 배포 (istio-ingressgateway 및 istio-egressgateway)를 제공한다.   
데모 설치를 사용하는 경우 둘 다 배포되고,    
수신 게이트웨이 만 기본 프로필로 배포된다.   
이러한 배포에 고유한 게이트웨이 구성을 적용하거나   
고유 한 게이트웨이 프록시를 배포 및 구성 할 수 있다.   
   
### 게이트웨이 예시
다음 예는 외부 HTTPS 수신 트래픽에 가능한 게이트웨이 구성을 보여준다.
<pre>
<code>
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: ext-host-gwy
spec:
  selector:
    app: my-gateway-controller
  servers:
  - port:
      number: 443
      name: https
      protocol: HTTPS
    hosts:
    - ext-host.example.com
    tls:
      mode: SIMPLE
      serverCertificate: /tmp/tls.crt
      privateKey: /tmp/tls.key
</code>
</pre>
