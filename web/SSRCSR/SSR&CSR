# 렌더링 방식과 라우팅 정리 (SSR · CSR · SPA · Express)

> 핵심 질문 하나로 전체가 연결됩니다: **"HTML을 어디서 완성하느냐"**
> 이 선택이 SSR/CSR을 가르고, 그 결정이 SPA · 라우팅 · 빌드 환경 · 백엔드 역할까지 연쇄적으로 이어집니다.

---

## 1. 출발점 — HTML을 어디서 완성하나

사용자가 도메인으로 접속하면 결국 브라우저는 HTML을 받아야 합니다. 이 HTML을 **서버가 미리 완성해서 주느냐**, 아니면 **뼈대만 주고 브라우저가 채우느냐**가 SSR과 CSR의 갈림길입니다.

### SSR (Server Side Rendering)

- 요청이 들어오면 주소에 매핑된 **컨트롤러**가 실행됨
- 컨트롤러가 필요한 데이터를 채운 뒤 **완성된 HTML**을 내려줌
- "채우는" 작업은 **JSP · Thymeleaf** 같은 템플릿 엔진이 담당
- `th:text`, `<th:each>` 같은 문법이 **서버에서** 실제 값으로 치환됨
- 브라우저는 이미 완성된 화면을 받음

### CSR (Client Side Rendering) — 진행한 프로젝트 방식

- 서버는 정적 HTML을 **그대로** 내려줌
- HTML 내부의 **JS가 동작**하며 서버에 추가 요청을 보내 내용을 채움
- 즉, 렌더링 책임이 **브라우저(클라이언트)** 로 넘어옴

---

## 2. CSR의 실제 동작 흐름

```
index.html 호출
 └─ main.js 실행
     ├─ 프레임워크 / 내부 파일 import
     └─ 최초 페이지: fetch로 HTML 조각을 가져오고, 짝이 되는 JS import
         (HTML · JS가 같은 이름으로 쌍을 이룸)

페이지 이동 시
 └─ 다음 페이지의 HTML을 fetch로 가져오고
     └─ 짝 JS를 import
```

> **연결 고리:** CSR은 화면 조각과 데이터를 계속 **서버에 요청**해야 합니다.
> 그 요청을 받아주는 곳이 뒤에 나오는 **Express 라우터**입니다.

---

## 3. SPA와 라우팅 — 화면은 그대로인데 URL만 바뀌는 이유

CSR로 만든 이런 구조가 곧 **SPA(Single Page Application)** 입니다. 페이지 전체를 새로 받지 않고 필요한 조각만 교체합니다.

- SPA에서 URL이 바뀌어도 브라우저가 새 페이지를 **다시 로딩하지 않는** 이유는 **History API의 `pushState`** 때문
- `pushState`는 서버에 요청을 보내지 않고 **주소창의 URL만 조작**함
- 그래서 URL은 바뀌지만, 실제 화면 전환은 JS가 fetch로 처리하는 구조가 성립

> SPA에서 라우팅이 성립하는 기술적 핵심이 바로 이 부분입니다.

---

## 4. 개발 · 빌드 환경 — Node · Vite · Webpack

이 프론트엔드를 **개발하고 빌드**하는 환경은 **Node.js**로 구성됩니다.

- **로컬 개발 서버**란 내 컴퓨터 안에서만 접속되는 임시 웹 서버를 띄워, 브라우저에서 `http://localhost:5173` 같은 주소로 작성 중인 화면을 확인하는 것
- **Vite**는 이 개발 서버가 돌 때 일부 런타임을 주입해 동작시킴 → 실제 **배포용 서버와는 구분**됨
- **Webpack**도 같은(비슷한) 역할 → 레거시에서 많이 쓰였고 지금도 사용 중

> **한 줄기:** SPA(구조) → CSR(렌더링 방식) → 로컬 개발 환경은 **Node.js + Vite**

---

## 5. 백엔드 라우팅 — CSR이 보내는 요청을 받는 쪽

CSR/SPA가 fetch로 보낸 데이터 요청은 서버가 받아 처리해야 합니다.
Express에서는 이를 **라우터 모듈 + 미들웨어 등록**으로 나눕니다.

### `routes/products.js` — 라우터 정의 (`/products` **뒤쪽 경로만** 담당)

```js
const express = require('express');
const router = express.Router();

router.get('/',         (req, res) => res.send('상품 정보 조회')); // GET    /products/
router.post('/insert',  (req, res) => res.send('신규 상품 추가')); // POST   /products/insert
router.put('/update',   (req, res) => res.send('상품 정보 수정')); // PUT    /products/update
router.delete('/delete',(req, res) => res.send('상품 정보 삭제')); // DELETE /products/delete

module.exports = router;
```

### `app.js` — 라우터 등록 (URL 앞부분을 라우터에 위임)

```js
const express = require('express');
const productRouter = require('./routes/products');

const app = express();
app.use('/products', productRouter);
```

### 핵심 요약

- **Router**로 모듈을 만들고, 그 모듈을 **미들웨어로 등록**해 `/products` 경로로 매핑·처리되게 함
- `app.use('/products', productRouter)` → `/products`로 들어온 요청이 `productRouter`로 넘어감
- `productRouter` 안에서는 `/`, `/insert` 등 **뒤쪽 경로만** 처리
  (예: `/products/insert` → 라우터의 `/insert`)

> **한 줄로:** `app.use`로 URL 앞부분(`/products`)을 라우터에 위임하고,
> 라우터는 그 뒤 경로를 **CRUD 메서드별로 나눠** 처리한다.

---

## 전체 연결 고리 한눈에

```
접속 시 HTML을 어디서 완성하나 (SSR vs CSR)
        │
        └─▶ 진행한 프로젝트는 CSR
                │
                └─▶ 그게 곧 SPA 구조
                        │
                        └─▶ URL 전환은 History API pushState 로 성립
                                │
                                └─▶ 프론트엔드는 Node.js + Vite 로 개발 / 빌드
                                        │
                                        └─▶ CSR이 fetch로 보내는 요청은
                                            Express 라우터가 경로별·메서드별로 나눠 처리
```
