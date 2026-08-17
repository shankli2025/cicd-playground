# PR vs Worktree PR 对照表

| 步骤 | 普通 PR（单书桌） | Worktree PR（第二张桌） |
|------|-------------------|-------------------------|
| 所在目录 | 主目录 `cicd-playground` | 主目录旁的新目录，如 `cicd-playground-new02` |
| 开支线 | `git checkout -b practice/xxx` | `git worktree add -b practice/xxx ../cicd-playground-xxx` |
| 改代码 | 在主目录改 | 进 worktree 目录改 |
| 暂存 | `git add .` | 同左 |
| 提交 | `git commit -m "..."` | 同左 |
| 首次推送 | `git push -u origin HEAD` | 同左 |
| 开 PR | `gh pr create` | 同左 |
| 看编号 | `gh pr view` 或 `gh pr list` | 同左（须在对应支线目录，或 `gh pr view <ID>`） |
| 合完后列表为空 | `gh pr list` 只显示 OPEN | 同左；worktree 合完仍在第二桌时更容易误以为「没了」 |
| 查已合并的 PR | `gh pr list --state merged` | **同左**（合完要用这条，不要只用 `gh pr list`） |
| 合并 | `gh pr merge <ID> --merge --delete-branch` | `gh pr merge <ID> --merge --delete-branch` |
| 合完切回 main | `gh` 常自动切回并 fast-forward | 常失败（`main` 已在主桌占用），可忽略 |
| 同步主线 | 多数情况不必再 `pull` | **必须回主目录** `git pull origin main` |
| 删远程支线 | `--delete-branch` 通常已删 | 同左（merge 成功即可） |
| 删本地支线 | `--delete-branch` 通常已删 | `git worktree remove ../目录` 后再 `git branch -d practice/xxx` |
| 额外收尾 | 无 | 主桌：`git pull` → `git worktree remove` → `git branch -d` |
| 同时改两条线 | 要 `checkout` 来回切 | 两张桌并行，不用切 |
| 同一支线 | 只能在当前目录 | 不能同时占两张桌 |

---

## 复制即用

### 普通 PR

```bash
git checkout main && git pull
git checkout -b practice/xxx
git add . && git commit -m "..."
git push -u origin HEAD
gh pr create
gh pr merge <ID> --merge --delete-branch
```

### Worktree PR

```bash
# 主桌开桌
git worktree add -b practice/xxx ../cicd-playground-xxx

# 第二桌
cd ../cicd-playground-xxx
git add . && git commit -m "..."
git push -u origin HEAD
gh pr create
gh pr merge <ID> --merge --delete-branch

# 主桌收尾
cd /Users/fwang/project/cicd-playground
git pull origin main
git worktree remove ../cicd-playground-xxx
git branch -d practice/xxx
```

---

## 合完看不到 PR？

`gh pr list` **默认只列 OPEN**。合并后申请单变成 Merged，列表会空，**不是删掉了**。

worktree 里合完人还在第二桌时尤其容易碰到：`gh pr list` 空白，以为单号丢了。

```bash
gh pr list --state merged    # 看已合并的（含编号）
gh pr view <ID>              # 指定编号仍可查看
```
