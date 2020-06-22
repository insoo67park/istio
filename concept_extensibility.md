# 확장성

WebAssembly는 Istio 프록시 (Envoy)를 확장하는 데 사용할 수있는 샌드 박싱 기술이다.   
Proxy-Wasm 샌드 박스 API는 믹서를 Istio의 기본 확장 메커니즘으로 대체한다.   
Istio 1.6은 Proxy-Wasm 플러그인을위한 균일 한 구성 API를 제공한다.   

### 웹 어셈블리 샌드 박스 목표 :

- 효율성-확장기능은 낮은 대기 시간, CPU 및 메모리 오버헤드를 추가한다.
- 기능-확장은 정책을 시행하고 원격 분석을 수집하며 페이로드 돌연변이를 수행 할 수 있다.
- 격리-하나의 플러그인에서 프로그래밍 오류 또는 충돌이 다른 플러그인에 영향을 미친다.
- 구성-플러그인은 다른 Istio API와 일치하는 API를 사용하여 구성됩니다. 확장은 동적으로 구성 될 수 있다.
- 운영자-확장은 로그 전용, 페일 오픈 또는 페일 클로즈로 카나리아 및 배포 할 수 있다.
- 확장 개발자-플러그인은 여러 프로그래밍 언어로 작성 될 수 있다.   
   
이 [비디오 토크](https://www.youtube.com/watch?v=XdWmm_mtVXI&feature=youtu.be, "비디오토크")는 WebAssembly 통합 아키텍처에 대한 소개이다.

## 고급 아키텍처
Istio 확장 (Proxy-Wasm 플러그인)에는 몇 가지 구성 요소가 있습니다.

- 필터 용 프록시 -Wasm 플러그인을 빌드하기위한 SPI (Filter Service Provider Interface).
- Envoy에 내장 된 샌드 박스 V8 Wasm 런타임.
- 헤더, 트레일러 및 메타 데이터를위한 호스트 API.
- gRPC 및 HTTP 호출을위한 API를 호출.
- 메트릭 및 모니터링을위한 통계 및 로깅 API.   
   
![Extending Istio/Envoy](https://istio.io/latest/docs/concepts/wasm/extending.svg)
