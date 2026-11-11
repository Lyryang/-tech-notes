# Git 学习笔记

## 一、核心概念

### 1. 快照机制

Git 本质是一个**时间机器**。每次 `git commit`，Git 给当前所有文件拍一张"快照"，并生成唯一的 hash 编号（如 `927e44b9`）。

```
时间轴 →
[快照A] → [快照B] → [快照C] → [快照D] → [快照E(当前)]
 a1b2c3    c3d4e5    927e44b   e5f6g7    HEAD
```

**关键结论**：
- 每个快照完整保存那一刻所有文件的状态
- 历史快照默认**不会被删除**，即使代码被覆盖也能找回
- 要真正删除历史需要 `git rebase` 或 `git reset --hard` + `git push --force`（多人协作中危险，慎用）

---

### 2. 三个区域

```
[工作区]  ──git add──▶  [暂存区]  ──git commit──▶  [本地仓库]  ──git push──▶  [远程仓库]
实际编辑的文件           准备提交的文件              本地历史快照              服务器上的仓库
```

| 区域 | 说明 |
|------|------|
| 工作区 | 编辑器里看到和修改的文件 |
| 暂存区 | `git add` 后进入，表示"这些改动我打算提交" |
| 本地仓库 | `git commit` 后永久记录为快照 |
| 远程仓库 | `git push` 后上传到服务器，其他人可以拉取 |

---

### 3. push 与 pull

**push**：把本地新快照上传到远程仓库，让别人也能看到。

**pull = fetch + merge 两步**：

```
第一步 fetch（下载）：
远程: [A] → [B] → [C(同事)]
                       ↓ 下载但还没合并
本地: [A] → [B] → [D(自己)]   +   [C(同事)] 暂存

第二步 merge（合并）：
本地: [A] → [B] → [D(自己)] → [E(合并结果)]
                 ↘ [C(同事)] ↗
```

---

### 4. 冲突的触发条件

| 情况 | 是否冲突 |
|------|---------|
| 两人修改不同文件 | ❌ 不冲突，自动合并 |
| 同一文件，修改不同位置（行） | ❌ 不冲突，自动合并 |
| 同一文件，修改**同一行** | ✅ 冲突，需手动解决 |

冲突标记长这样：

```
<<<<<<< HEAD（你的版本）
const speed = 100;
=======
const speed = 200;
>>>>>>> origin/main（同事的版本）
```

手动保留正确内容，删掉标记符，再 `git add` + `git commit`。

---

## 二、代码恢复流程

### 场景：自己的代码被别人合并冲突时误删

**第一步：下载远程最新记录**
```bash
git fetch --all
```

**第二步：找到自己的提交 hash**
```bash
git log --oneline --author="你的用户名" --all
```

**第三步：确认那次提交改了哪些文件**
```bash
git show <hash> --stat
```

**第四步：对比当前和那次提交的差异**
```bash
git diff <hash> HEAD --name-only   # 只列文件名
git diff <hash> HEAD -- <文件路径>  # 查看具体改动内容
```

diff 输出解读：
- `-` 红色开头 → 那次提交之后**被删掉**的内容（可能是你的代码）
- `+` 绿色开头 → 那次提交之后**新增**的内容（可能是别人加的）

**第五步：精准恢复（只恢复指定文件）**
```bash
git checkout <hash> -- <文件路径>
```
> ⚠️ 注意：如果该文件在你的提交之后别人也做了合法修改，直接 checkout 会把别人的改动也一起抹掉，需要手动合并。

**第六步：提交并推送**
```bash
git add .
git commit -m "fix: 恢复被覆盖的xxx功能"
git pull   # 先拉取，处理可能的冲突
git push
```

---

## 三、本地排除文件追踪

### 方案一：.gitignore（适合从未提交过的文件）

在 `.gitignore` 中添加文件路径即可。

> ⚠️ 对已经被 git 追踪过的文件无效！需要先执行：
> ```bash
> git rm --cached <文件路径>
> ```

### 方案二：--skip-worktree（适合已追踪文件，且不能改 .gitignore）

```bash
# 让 git 忽略该文件的本地改动
git update-index --skip-worktree public/mapConfig.json

# 恢复追踪
git update-index --no-skip-worktree public/mapConfig.json
```

效果：`git status` 不显示、`git add .` 不会加入，**不产生任何提交记录，对他人无感知**。

| | `--skip-worktree` | `--assume-unchanged` |
|---|---|---|
| 语义 | 故意在本地修改，不想提交 | 承诺文件不会变（性能优化） |
| pull 时远程有更新 | 不会被覆盖 | 可能被覆盖 |
| 推荐场景 | 本地配置文件覆盖 ✅ | 大型仓库性能优化 |

---

## 四、常用命令速查

```bash
# 查看提交历史
git log --oneline -20
git log --oneline --author="用户名" --all

# 查看某次提交内容
git show <hash>              # 完整改动
git show <hash> --stat       # 只看文件列表
git show <hash>:<文件路径>   # 查看那次提交时某文件的内容

# 对比差异
git diff <hash> HEAD --name-only     # 列出差异文件名
git diff <hash> HEAD -- <文件路径>   # 某文件的具体差异

# 恢复文件
git checkout <hash> -- <文件路径>    # 从历史快照取出文件

# 同步远程
git fetch --all   # 下载远程记录，不合并
git pull          # 下载 + 合并
git push          # 上传本地提交

# 本地忽略文件
git update-index --skip-worktree <文件路径>
git update-index --no-skip-worktree <文件路径>
git rm --cached <文件路径>   # 取消追踪（配合 .gitignore 使用）
```

---

## 五、本次实战复盘

**问题**：同事 `b63b51e` 在添加调参面板功能时，同时删除了多飞行器协同相关代码。

**时间线**：
```
c0de390  缩短数据传递链路（自己）← 删除了冗余 props
927e44b  多飞协同场景初步完成（自己）← 要恢复的功能
b63b51e  添加调参面板功能（同事）← 误删了多飞协同，且把已删的冗余 props 又加回来
c333146  Merge（当前 HEAD）
```

**解决方案**：不能直接 `git checkout 927e44b -- DynaConfigScenario.vue`，因为会抹掉同事合法的调参面板新增内容。需要**手动合并**：只恢复被删除的多飞协同代码，同时保留同事新增的内容，并移除被重新引入的冗余 props。

**验证方法**：
```bash
git diff 927e44b -- src/.../DynaConfigScenario.vue
# 无输出 = 文件内容与那次提交完全一致
```
