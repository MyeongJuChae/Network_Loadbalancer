# Network_Loadbalancer
HAproxy와 Keepalived를 사용한 Loadbalancing, fault tolerant(폴트 톨러런트)

gRPC 기반의 서비스에 필요한 이중화(Fault Tolerant), 로드 밸런싱(Load Balacing)을 동시에 제공

HTTP/2 기반의 gRPC 트래픽 load-balancing 지원

다양한 로드 밸런싱 방법 지원: Round robin, Least connection 등

오픈 소스 keepalived의 VRRP(Virtual Router Redudancy Protocol) stack을 이용한 HA 지원

Virtual IP를 사용해서 추가의 장비 필요 없이 기존 서버(TTS 서버) 설치 가능 – 서버 자원 영향 최소화

Health check 커스터마이징: gPRC 프로토 파일 기반의 health check 스크립트가 AI 엔진의 서비스 up/down 여부를 체크하여 잘못된 부하 분산으로 서비스 오류 발생을 방지

HA traffic 분석툴과 서버 상태 및 통계 페이지 제공
