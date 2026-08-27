# Git 원격 브랜치 fetch & clone 설정 정리

원격 브랜치 fetch 설정(refspec) 점검·수정, 얕은 clone, 커밋 이력 정리까지 정리한 문서입니다.

---

## 1. 원격 브랜치 fetch 설정

원격에 새 브랜치가 있는데 `git branch -r`에 보이지 않을 때, fetch 설정(refspec)을 점검하고 수정하는 절차입니다.

### 1) 원격 브랜치 실제 목록 확인

```bash
git ls-remote --heads origin
```

### 2) 현재 fetch 설정 확인

```bash
git config --get-all remote.origin.fetch
```

### 3) 결과 해석

**개별 브랜치만 등록되어 있으면** → 나머지 브랜치는 안 보임:

```
+refs/heads/master:refs/remotes/origin/master
+refs/heads/2025_car_renewal_sub:refs/remotes/origin/2025_car_renewal_sub
+refs/heads/2025_car_renewal_master:refs/remotes/origin/2025_car_renewal_master
```

**와일드카드(`*`)로 되어 있으면** → 모든 브랜치 fetch 가능 (정상):

```
+refs/heads/*:refs/remotes/origin/*
```

### 4) 해결 — 둘 중 택1

#### 방법 A. 특정 브랜치만 추가

```bash
git config --add remote.origin.fetch "+refs/heads/car_26_01_20:refs/remotes/origin/car_26_01_20"
git fetch origin --prune
```

#### 방법 B. 모든 브랜치를 바라보도록 전체 교체

```bash
git config --unset-all remote.origin.fetch
git config --add remote.origin.fetch "+refs/heads/*:refs/remotes/origin/*"
git fetch origin --prune
```

### 5) 반영 확인

```bash
git branch -r
```

### 핵심

- fetch refspec에 원하는 브랜치(또는 `*`)가 포함되어야 원격 브랜치가 보인다.
- 대부분은 **방법 B(와일드카드)** 로 두는 것이 편하다.
- 특정 브랜치만 필요한 경우에만 **방법 A** 를 사용한다.
- `--prune` 옵션은 원격에서 삭제된 브랜치의 로컬 추적 참조도 함께 정리해 준다.

---
