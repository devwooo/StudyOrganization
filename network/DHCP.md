# DHCP 동작 과정 정리

DHCP의 IP 할당 4단계(DORA), 중복 검사(ARP), 갱신 과정을 정리한 문서입니다.

---

## DHCP (Dynamic Host Configuration Protocol) — 자동 설정

인터넷 공유기에도 DHCP 서버가 탑재되어 있으며, 각각의 PC에 **사설 IP를 할당**한다.

---

## 1. DHCP 4단계 흐름 (DORA)

| 단계 | 방향 | 메시지 |
|------|------|--------|
| 1 | Client → Server | **D**HCP Discover |
| 2 | Client ← Server | **O**ffer |
| 3 | Client → Server | **R**equest |
| 4 | Client ← Server | **A**CK |

---

## 2. 각 단계 상세

### 1. DHCP Discover (브로드캐스트) — "IP 주소 필요해요"

아직 IP가 없으므로 출발지/목적지를 브로드캐스트로 뿌린다.

- `src(출발지) = 0.0.0.0`
- `Dst(목적지) = 255.255.255.255`

### 2. DHCP Offer (서버 응답) — "이 IP 사용해"

서버가 사용 가능한 IP를 제안하며, 다음 정보를 함께 내려준다.

- IP 주소 / 서브넷 마스크 / 게이트웨이 / DNS / 임대시간(Lease Time)

### 3. DHCP Request (선택) — "이 IP 사용할게요"

- 여러 DHCP 서버가 있다면 하나를 선택한 후 브로드캐스트로 요청한다.

### 4. DHCP ACK (확정) — 확정

- 서버가 승인하여, **이 시점부터 IP를 사용해 네트워크 통신 가능**하다.

---

## 3. 중복 검사 (IP 충돌 확인)

### 4.1 ARP Probe

`Sender IP(SPA) = 0.0.0.0`, `Target IP(TPA) = 할당받은 IP` → 브로드캐스트

- 중복 여부 판단
- `Sender IP(SPA) = 0.0.0.0`은 **아직 해당 IP를 쓰고 있지 않다**는 의미
- 응답이 오면(누군가 이미 사용 중) → DHCP 서버에 **DECLINE**(거부) 통보

### 4.1 Gratuitous ARP (= ARP Announcement)

할당받은 IP를 브로드캐스트로 통신(선언)한다.

- 중복이 없다면 → 주변 기기들의 **ARP Cache 업데이트**

---

## 4. 그 외

- **갱신(Renew)** 시에는 **1, 4번(Request → ACK)** 만 진행된다.
- **추가 개념**: DHCP Relay Agent, DHCP Snooping, DHCP Proxy Agent
- 여기서 얻은 **게이트웨이 IP**를 통해 게이트웨이 **MAC 주소**를 얻는데,
  이는 **네트워크 연결 상태(서비스 시작 여부)를 판단**하기 위한 행동이다.
