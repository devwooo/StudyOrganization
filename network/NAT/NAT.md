# 공유기와 NAT 정리

## 공유기 (Home Router)

NAT 기술이 적용된 장치로, **IP 주소와 포트 번호를 제어**해서 여러 대의 컴퓨터(여러 Private IP)가 **하나의 공인 IP(Public IP)**를 통해 외부와 통신할 수 있게 해준다.

## NAT (Network Address Translation)

사설 IP ↔ 공인 IP 변환을 담당하는 기술이며, 외부 포트를 어떻게 매핑하느냐에 따라 크게 두 종류로 나뉜다.

### Symmetric NAT
- **TCP 세션마다** 외부 포트를 새로 지정
- 목적지가 바뀌면 매핑도 달라지므로, 상대적으로 **보안성이 높음** (대신 P2P 연결은 어려움)

### Cone NAT
Host 단위로 외부 포트를 지정 (같은 내부 IP:Port는 항상 같은 외부 포트로 매핑)

- **Full Cone**: 외부 아무나 매핑된 포트로 접근 가능 (가장 개방적)
- **Restricted Cone**: 내가 먼저 보낸 적 있는 상대만 응답 가능
    - **IP Restricted**: 목적지 **IP** 기준으로 제한
    - **Port Restricted**: 목적지 **IP + Port** 기준으로 제한

## 방향에 따른 IP 변환 (핵심 개념)

LAN 내부(**Inbound**)에서는 Private IP를 사용하지만, WAN으로 나갈 때(**Outbound**)는 Private IP를 Public IP로 바꿔줘야 한다. 이 역할을 하는 것이 **공유기**이고, 이 변환 관계를 관리하기 위해 **NAT Table**을 사용한다.

### 동작 흐름 예시

- 서버 입장에서는 공유기가 이미 Private IP를 Global IP로 바꿔서 요청을 보냈기 때문에, `3.3.3.3:23000`에서 요청이 온 것으로 인식한다.
- 따라서 응답할 때 서버는 `3.3.3.3:23000`을 **목적지(DST)**로 설정해 패킷을 보낸다.
- 공유기는 NAT Table을 참조해 이 `3.3.3.3:23000`이 실제로 내부의 어느 Private IP:Port에 해당하는지 찾아 되돌려 준다.

