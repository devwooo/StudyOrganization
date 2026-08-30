# HttpSecurity와 SecurityConfigurer의 관계

Spring Security의 설정 구조에서 `HttpSecurity`(빌더)와 `SecurityConfigurer`(설정자)가 어떻게 협력하는지 정리한 문서입니다.

---

## 한 줄 요약

> 빌더가 여러 개의 `SecurityConfigurer`를 **컬렉션으로 소유**하고, 빌드 시점에 각 Configurer에게 **자기 자신(`this`)을 넘겨** 설정을 위임한다.

![HttpSecurity와 SecurityConfigurer의 관계](/img/httpsecurity-configurer.svg)

---

## 1. 상속 관계

`HttpSecurity`는 `SecurityBuilder`를 직접 구현한 것이 아니라, 중간 추상 클래스를 거쳐 상속받습니다.

```
SecurityBuilder<O>                                   (interface)
        ▲ implements
AbstractSecurityBuilder<O>                            (abstract)
        ▲ extends
AbstractConfiguredSecurityBuilder<O, B>               (abstract)  ← 핵심
        ▲ extends
HttpSecurity extends AbstractConfiguredSecurityBuilder<DefaultSecurityFilterChain, HttpSecurity>
```

핵심은 중간의 `AbstractConfiguredSecurityBuilder`입니다. Configurer들을 담아두는 컬렉션이 바로 여기에 선언되어 있습니다.

---

## 2. "참조"가 아니라 "소유(composition)"

`HttpSecurity`가 직접 리스트를 선언한 게 아니라, **부모인 `AbstractConfiguredSecurityBuilder`가 컬렉션으로 보유**하고 있고 `HttpSecurity`는 이를 상속받아 사용합니다.

```java
public abstract class AbstractConfiguredSecurityBuilder<O, B extends SecurityBuilder<O>>
        extends AbstractSecurityBuilder<O> {

    // SecurityConfigurer들을 담아두는 컬렉션 (조합 / composition 관계)
    private final LinkedHashMap<Class<? extends SecurityConfigurer<O, B>>,
                                List<SecurityConfigurer<O, B>>> configurers = new LinkedHashMap<>();
    // ...
}
```

그래서 "참조(reference)"보다 **조합(composition)** 이 더 정확한 표현입니다. 빌더가 여러 Configurer를 **필드로 소유**하는 구조이기 때문입니다.

---

## 3. 관계는 사실 양방향

`SecurityConfigurer` 쪽에서도 빌더를 참조합니다. 메서드가 빌더를 **인자로 받기** 때문입니다.

```java
public interface SecurityConfigurer<O, B extends SecurityBuilder<O>> {
    void init(B builder) throws Exception;      // 빌더를 인자로 받음
    void configure(B builder) throws Exception; // 빌더를 인자로 받음
}
```

- 제네릭 타입 `B extends SecurityBuilder<O>` 자체가 `SecurityBuilder`에 의존
- `init()` / `configure()`는 빌더(`HttpSecurity`)를 넘겨받아 그 빌더를 설정

---

## 4. 전체 동작 흐름

### 등록 단계

`http.formLogin()`, `http.csrf()` 등을 호출할 때마다, 대응하는 Configurer가 부모 빌더의 `configurers` 컬렉션에 쌓입니다.

### 빌드 단계

`build()`가 호출되면 컬렉션을 순회하며 **2단계(two-phase)** 로 실행됩니다.

| 순서 | 단계 | 하는 일 |
|------|------|---------|
| 1 | `init(this)` 전체 호출 | 양방향 참조 초기화, 공유 객체 등록 등 준비 작업 |
| 2 | `configure(this)` 전체 호출 | 실제 필터 생성 후 빌더에 추가 |

이때 넘기는 `this`가 바로 `HttpSecurity` 인스턴스이고, Configurer는 이걸 받아 `builder.addFilter(...)` 같은 식으로 빌더 자신을 조립합니다.

```
build() → doBuild()
        → 모든 configurer.init(this)      // 1단계: 초기화
        → 모든 configurer.configure(this) // 2단계: 구성
        → DefaultSecurityFilterChain 완성
```

---

## 5. 정리

- `HttpSecurity`(→ `AbstractConfiguredSecurityBuilder`)는 **여러 `SecurityConfigurer`를 컬렉션으로 소유**한다 (조합 관계).
- 각 `SecurityConfigurer`는 `init()` / `configure()` 시점에 **그 빌더를 넘겨받아** 설정 로직을 실행한다.
- 즉, **빌더가 Configurer를 모아두고, 빌드 시점에 각 Configurer에게 자기 자신을 넘겨 설정을 위임하는** 협력 구조다.

---

> **다음 학습 포인트**: `init()`과 `configure()`를 왜 굳이 두 단계로 나눴는지가 이 설계의 핵심입니다. Configurer 간에 서로의 공유 객체에 의존해야 할 때, 모든 초기화(`init`)를 먼저 끝내두어야 구성(`configure`) 시점에 안전하게 참조할 수 있기 때문입니다.
