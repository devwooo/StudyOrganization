---
## 조건 어노테이션 (범용 규칙)

| 어노테이션 | 범용 의미 |
|---|---|
| `@ConditionalOnClass(A.class)` | 클래스패스에 `A`가 **존재하면** 활성화 (해당 라이브러리가 있을 때만) |
| `@ConditionalOnMissingBean(name = "b")` | 컨테이너에 이름이 `b`인 빈이 **없으면** 활성화 (있으면 물러남) |
| `@ConditionalOnMissingBean(A.class)` | 컨테이너에 `A` 타입 빈이 **없으면** 활성화 |
| `@ConditionalOnDefaultWebSecurity` | 위 둘의 조합 = `A`가 클래스패스에 존재 **AND** 사용자 `SecurityFilterChain` 빈이 없을 때 활성화 |

---

## 설정 / 기능 어노테이션 (범용 규칙)

| 어노테이션 | 범용 의미 |
|---|---|
| `@Configuration(proxyBeanMethods = false)` | 설정 클래스로 등록, CGLIB 프록시 없음(lite mode) → 빈 간 의존은 **파라미터 주입**으로 |
| `@EnableWebSecurity` | 해당 기능(웹 보안) 인프라를 켜서 관련 빈들을 `@Import`로 등록 |

---

## 핵심 패턴 두 가지

### 1. `@ConditionalOnClass(A.class)` → "A라는 도구가 있을 때만"

그 라이브러리가 의존성에 포함돼 있는지로 켤지 말지 결정.
없으면 관련 코드를 아예 시도하지 않음 (안전장치).

### 2. `@ConditionalOnMissingBean(...)` → "아무도 안 만들었을 때만"

개발자가 직접 등록한 빈이 있으면 물러나고, 없으면 Boot가 기본값 제공.
**"사용자 우선, 기본값은 fallback"** 원칙.

---

## 한 줄 요약

Spring Boot auto-configuration의 뼈대:

- **`OnClass`** = 있으면 켜고
- **`OnMissingBean`** = 없으면 채운다
