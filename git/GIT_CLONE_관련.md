
## 2. 얕은 clone (depth / single-branch)

크기가 큰 repo를 clone 받을 때 `--depth`로 히스토리 깊이를 제한하고, `--no-single-branch` / `--single-branch`로 가져올 브랜치 범위를 조정한다.

- `--depth <N>` : 최근 N개 커밋의 히스토리만 가져옴 (히스토리 축소)
- `--no-single-branch` : 모든 브랜치를 가져옴
- `--single-branch` : 하나의 브랜치만 가져옴 (기본 동작)

### 특정 브랜치 기준으로 clone

```bash
git clone --depth 1 --single-branch -b master <repo-url>
```

- `master` 브랜치를 기준으로 clone 한다는 의미로, `-b` 옵션이 어느 브랜치를 기준으로 둘지 정하는 옵션.
- clone 후 fetch refspec 확인:

```bash
git config --get-all remote.origin.fetch
# +refs/heads/master:refs/remotes/origin/master
```

### 브랜치를 지정하지 않고 clone

```bash
git clone --depth 1 --single-branch <repo-url>
```

- `-b` 옵션이 없는 경우, 내부적으로 default로 지정되어 있는 브랜치를 기준으로 clone 된다.
- fetch refspec 확인:

```bash
git config --get-all remote.origin.fetch
# +refs/heads/<default>:refs/remotes/origin/<default>
```

---
