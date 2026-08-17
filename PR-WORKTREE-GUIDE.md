# PR 与 Worktree 速查手册

> 比喻：出版社 = GitHub，主线 = `main`，支线 = feature 分支，改稿申请单 = PR，手指 = `HEAD`，主书桌 = 主目录，第二张桌 = worktree 目录。

本仓库推荐 clone 方式（shankli2025 账号）：

```bash
git clone git@work:shankli2025/cicd-playground.git
# 练 PR / worktree 不要用 --depth 1（浅克隆会导致 origin/支线 标签缺失）
```

---

## 一、四个词（先记牢）

| 词 | 比喻 | 在哪 |
|----|------|------|
| `HEAD` | 手指：正在改哪一页 | 本地 |
| `main` | 正式书架 / 主线 | 本地 + 远程 |
| `origin` | 出版社电话（远程 URL 外号） | 本地配置 |
| `origin/main` | 抽屉里「远程主线」的复印件标签 | 本地（fetch/pull 后更新） |

---

## 二、PR 标准流程（单书桌 / checkout）

适合：一次只做一件事，改完再换任务。

### 流程图

```
main → 开支线 → 改代码 → add → commit → push → gh pr create → merge → 回 main pull
```

### 逐步命令

```bash
# 0. 在主目录，确保主线最新
cd /Users/fwang/project/cicd-playground
git checkout main
git pull origin main

# 1. 开支线（手指挪过去）
git checkout -b practice/my-feature

# 2. 改文件后提交
git add .
git commit -m "docs: describe your change"

# 3. 第一次 push 这条支线（必须 -u）
git push -u origin HEAD

# 4. 开 PR（默认：当前支线 → main）
gh pr create
# 或指定：gh pr create --title "..." --body "..."

# 5. 合并（编号看 gh pr list 或 gh pr view 里的 #数字）
gh pr merge <ID> --merge --delete-branch

# 6. 收尾
git checkout main
git pull origin main
```

### 验收清单

```bash
git status                    # 在 main，working tree clean
git branch -vv                # 只剩 main，且 [origin/main] 对齐
gh pr list --state merged     # 能看到刚合并的 PR
```

### 常用查看命令

```bash
git branch                    # 本地支线，* 为当前
git branch -vv                # 带远程跟踪信息
git status                    # 工作区 / 暂存区状态
git log --oneline -5          # 最近提交（注意是 --oneline 不是 --online）
gh pr list                    # 进行中的 PR
gh pr view                    # 当前支线的 PR（在 main 上通常为空）
gh pr view <ID>               # 指定编号
gh pr list --state merged       # 已合并的 PR
```

### PR 编号在哪看

- `gh pr view` 标题行：`cicd-playground#8` → 编号是 **8**
- 网页链接：`.../pull/8`
- `gh pr list` 的 **ID** 列

---

## 三、Worktree 流程（多书桌 / 并行）

适合：主线还要修 bug，功能支线继续写；或同时开多个 PR，不想来回 `checkout`。

### 和 checkout 的区别

| | `git checkout` | `git worktree` |
|--|----------------|----------------|
| 书桌数量 | 1 个文件夹 | 多个文件夹 |
| 换支线 | 同一目录里换，文件全变 | 进另一个目录，互不影响 |
| 并行改两条支线 | 麻烦（要 stash） | 自然支持 |

### 流程图

```
主桌 [main]                    第二桌 [practice/wt01]
     │                              │
     │  git worktree add ...        │  改代码 → commit → push → PR
     │                              │
     └──────── 共用同一个 .git ─────┘
                    │
              gh pr merge → 回主桌 pull → worktree remove
```

### 逐步命令

```bash
# 1. 在主桌开第二张桌 + 新建支线
cd /Users/fwang/project/cicd-playground
git worktree list
git worktree add -b practice/wt01 ../cicd-playground-wt01

# 2. 去第二张桌干活（不要在主桌改这条支线）
cd ../cicd-playground-wt01
git status                    # 应在 practice/wt01
# 改文件…
git add .
git commit -m "docs: worktree change"
git push -u origin HEAD
gh pr create

# 3. 在第二张桌合并（推荐加 --delete-branch 删远程支线）
gh pr merge <ID> --merge --delete-branch
# 可能报错：main is already used by worktree at '...' → 合并已成功，忽略即可

# 4. 回主桌收尾（这 3 步基本省不了）
cd /Users/fwang/project/cicd-playground
git pull origin main
git worktree remove ../cicd-playground-wt01
git branch -d practice/wt01     # 本地删支线名（远程已在 merge 时删则不必 push --delete）
```

### 验收清单

```bash
git worktree list               # 只剩主目录一行
git branch -vv                  # 无 practice/wt01
git log --oneline -3            # 能看到 Merge pull request #N
```

### 忘了支线名怎么查

```bash
git worktree list               # [方括号里] 就是支线名
git branch -vv                  # worktree 占用时名字前有 +
gh pr view <ID> --json headRefName --jq .headRefName
```

---

## 四、PR vs Worktree 对照

| 步骤 | 单书桌（PR only） | 多书桌（Worktree + PR） |
|------|-------------------|-------------------------|
| 开支线 | `git checkout -b xxx` | `git worktree add -b xxx ../目录` |
| 改代码 | 当前目录 | 进 worktree 目录 |
| push / PR | 相同 | 相同 |
| merge | `gh pr merge ID --merge --delete-branch` | 相同（建议在第二桌执行） |
| 同步 main | `git checkout main && git pull` | **只在主桌** `git pull` |
| 额外收尾 | 无 | `git worktree remove` + `git branch -d` |

**PR 流程本身不变**；worktree 只改变「你在哪个文件夹里改」，不改变 push / PR / merge 的逻辑。

---

## 五、注意事项

### 1. push 与 gh pr create

- 新支线 **第一次** push：`git push -u origin HEAD`
- 同一支线后续：`git push`
- 已 push 且 `-u` 后，`gh pr create` 通常不再问「推到哪里」

### 2. 浅克隆 `--depth 1` 的坑

浅克隆只跟踪 `origin/main`，push 后本地可能没有 `origin/支线名`，`gh pr create` 会再问一次 push 目标。

**修复（在现有目录）：**

```bash
git fetch --unshallow origin
git config remote.origin.fetch '+refs/heads/*:refs/remotes/origin/*'
git fetch origin
```

**或删了重 clone（不要加 --depth 1）。**

### 3. worktree 规则

- 同一支线不能同时在两张桌上检出
- `main` 已在主桌，第二桌不能再 checkout `main`
- worktree 目录放 **主目录旁边**（`../xxx`），不要放在仓库里面

### 4. 删支线：远程 vs 本地

| 命令 | 删什么 |
|------|--------|
| `gh pr merge --delete-branch` | 远程支线（GitHub） |
| `git push origin --delete 支线名` | 远程支线（手动） |
| `git branch -d 支线名` | 本地支线名 |
| `git worktree remove 目录` | 本地 worktree 文件夹 + 登记 |

### 5. 追加文件内容

```bash
echo 'text' >> file    # 追加（推荐）
echo 'text' > file     # 覆盖整个文件（慎用）
```

---

## 六、最小命令模板（复制即用）

### 模板 A：单桌 PR

```bash
git checkout main && git pull
git checkout -b practice/TODO
# 改文件
git add . && git commit -m "docs: TODO"
git push -u origin HEAD
gh pr create
gh pr merge <ID> --merge --delete-branch
git checkout main && git pull
```

### 模板 B：worktree + PR

```bash
# 主桌
git worktree add -b practice/TODO ../cicd-playground-TODO

# 第二桌
cd ../cicd-playground-TODO
# 改文件
git add . && git commit -m "docs: TODO"
git push -u origin HEAD
gh pr create
gh pr merge <ID> --merge --delete-branch

# 主桌收尾
cd /Users/fwang/project/cicd-playground
git pull origin main
git worktree remove ../cicd-playground-TODO
git branch -d practice/TODO
```

---

## 七、gh 常用命令

```bash
gh auth status                # 是否已登录
gh repo clone owner/repo      # clone（本机建议用 git@work:...）
gh pr create                  # 开 PR
gh pr list                    # 进行中的 PR
gh pr view [ID]               # 查看 PR（ID 可选）
gh pr merge <ID> --merge --delete-branch
```

登录：`gh auth login`（本机 SSH 用 `git@work` 时，PR 仍可用 gh；clone 建议直接用 work 地址）。

---

*最后更新：2026-08-17*
