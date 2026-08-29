# VPN · IPSec 정리

## VPN (Virtual Private Network)

기존의 **인터넷 인프라(public)** 위에서 **사설 네트워크(Private Network)**를 구성하기 위해 **두 네트워크를 연결하는 기술(망연계)**이다. 핵심은 **터널링(Tunneling)** 기술이며, private하게 만들기 위해 **보안 · 데이터 인증 · 암호화** 기술이 적용된다.

## IPSec (Internet Protocol Security)

공용 인터넷망을 통해 **3계층(Network Layer)에서 데이터를 암호화·인증**하여 안전한 VPN 연결을 제공하는 보안 프로토콜 스위트다.

### 활용

- **IPv4, IPv6** 모두 지원
- VPN 연결 형태: `GtoG(Gateway to Gateway)`, `GtoE(Gateway to Endpoint)`, `to(연결됨 == 터널링 + 암호화)`
- IPSec을 지원하는 Router를 **SG(Security Gateway)**라고 부름

### IPSec SA (Security Association)

IPSec 통신을 하기 위해 정의하는 **파라미터들(프로토콜 · 알고리즘 · 키 값 · 속성 등)**을 SA라고 한다. 아래의 보안 프로토콜(AH, ESP)은 모두 이 SA를 이용해 보안 서비스를 제공한다.

### 인증·암호화를 위한 헤더 (AH vs ESP)

| 헤더 | 정식 명칭 | 제공 기능 |
|------|-----------|-----------|
| **AH** | Authentication Header | 데이터의 원본 인증 + 무결성 + 재연공격 방지 (암호화 X) |
| **ESP** | Encapsulation Security Payload | 데이터의 기밀성(암호화) + 원본·인증 + 재연공격 방지 |

핵심 차이: **AH는 암호화를 안 하고 인증만**, **ESP는 암호화까지** 제공한다.

### 전송 방법 (Transport vs Tunnel)

| 모드 | 동작 | 원본 IP 헤더 |
|------|------|-------------|
| **전송 모드 (Transport)** | IP payload만 암호화, 원래 IP 헤더로 캡슐화 | 원래 헤더 사용 → IP 노출 가능성 있음 |
| **터널 모드 (Tunnel)** | IP 패킷 전체를 암호화 | 새 IP 헤더를 만들어 기존 헤더를 숨김 |

### 키 관리 (Key Management)

- **ISAKMP** (Internet Security Association and Key Management): 보안 협상 및 암호화 키들을 **관리하는 메커니즘** 제공
- **IKE** (Internet Key Exchange): 실제 키 교환을 담당

## 재택근무 예시 = GtoE (Gateway to Endpoint)

![재택근무 GtoE 터널링 구조](/img/vpn_gtoe_tunnel.svg)

1. 자택 PC에서 **VPN Client 프로그램**에 접속하면 **새로운 IP를 부여**받는다.
2. VPN Client가 부여받은 IP로 **해당 지사 SG(Security Gateway)와 터널링**을 맺는다.
3. 해당 지사는 받은 패킷을 다시 **GtoG를 통해 암호화하여 서버로** 보낸다.
4. 즉, 터널링이 **한 번 더 이뤄지는 것**(GtoE)이고, 나머지는 GtoG의 연속으로 이해하면 된다.

---

## 참고 — 실무 관점 보충

- **AH는 실무에서 거의 안 쓰인다.** 암호화를 안 해 내용이 그대로 보이고, NAT 환경과 충돌한다(AH가 IP 헤더까지 인증하는데 NAT가 그 헤더를 바꿔버리기 때문). 그래서 실무 VPN은 대부분 **ESP(+터널 모드)**를 쓴다.
- **IKE는 보통 IKEv2**를 쓴다. ISAKMP는 협상 틀(프레임워크)이고, IKE가 그 위에서 실제 키 합의를 수행하는 관계다.
- IPSec 외에 **WireGuard**(더 가볍고 빠름)나 **SSL/TLS VPN**(OpenVPN 등)도 많이 쓰인다. IPSec은 L3에서, SSL VPN은 L4 이상에서 동작한다는 계층 차이가 있다.
