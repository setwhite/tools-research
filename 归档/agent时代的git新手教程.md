# Agent 时代的 Git 新手教程

> 零基础入门。**不用记命令**——理解概念，把意图告诉 AI Agent，它会帮你执行。

## 目录

- [一、Git 是什么](#一git-是什么)
  - [1.1 手工版本管理的痛点](#11-手工版本管理的痛点)
  - [1.2 Git 帮你自动做这件事](#12-git-帮你自动做这件事)
  - [1.3 和其他工具的区别](#13-和其他工具的区别)
  - [1.4 Git 不擅长二进制文件](#14-git-不擅长二进制文件)
- [二、为什么要用 Git](#二为什么要用-git)
  - [AI 时代更需要 Git](#ai-时代更需要-git)
- [三、怎么用 —— 理解概念，告诉 Agent](#三怎么用--理解概念告诉-agent)
  - [3.1 四层结构（add / commit / push）](#31-四层结构add--commit--push)
  - [3.2 分支（branch / checkout）](#32-分支branch--checkout)
  - [3.3 合并（merge / rebase）](#33-合并merge--rebase)
  - [3.4 冲突](#34-冲突)
  - [3.5 撤销（reset / revert）](#35-撤销reset--revert)
  - [3.6 暂存现场（stash）](#36-暂存现场stash)
  - [3.7 后悔药（reflog）](#37-后悔药reflog)
- [四、每天对 Agent 说这几句就够了](#四每天对-agent-说这几句就够了)
- [五、常见问题速查](#五常见问题速查)
- [六、进阶能力（知道存在即可）](#六进阶能力知道存在即可)
- [附录：一个进阶坑](#附录一个进阶坑)

---

## 一、Git 是什么

### 1.1 手工版本管理的痛点

你写过毕业论文吗？

```
论文_v1.doc
论文_v2_导师修改.doc
论文_v3_最终版.doc
论文_v3_最终版_真的最终.doc
论文_v3_最终版_打死不改.doc
```

靠文件名区分版本——混乱、易丢、无法协作。

### 1.2 Git 帮你自动做这件事

Git 是分布式版本控制系统，记录每次变化，随时回到任意历史版本。每人电脑上都有完整仓库。

**核心优势**

| 优势 | 一句话说明 |
|------|-----------|
| 不用联网就能存 | 没网照样保存进度、看历史、开实验线，有网了再同步 |
| 代码不会丢 | 服务器坏了，人手一份完整备份 |
| 开实验线零成本 | 一秒创建，随便改，好了合并，不好删掉，主线不受影响 |
| 谁改了什么都记着 | 每行代码可追溯到人、时间、原因 |
| 存得省空间 | 不改的内容不重复存，改过的只记变化 |
| 回溯历史飞快 | 切换到任意历史版本几乎瞬间完成 |

### 1.3 和其他工具的区别

| 对比维度 | 集中式（SVN） | 分布式（Git） |
|---------|-------------|-------------|
| 历史存放 | 中央服务器 | 每人电脑都有 |
| 离线提交 | 不行 | 可以 |
| 服务器崩了 | 全完 | 人手一份 |
| 回溯历史 | 逐层反推，慢 | 瞬间切换 |
| 开分支 | 复制整个目录 | 秒建秒切 |

### 1.4 Git 不擅长二进制文件

二进制文件（Word、PPT、Excel）无法对比差异、无法自动合并、仓库迅速膨胀。

**但 AI 正在改变这个局面**

| 二进制 | → 文本替代 | 好处 |
|--------|-----------|------|
| Word | Markdown | 纯文本，可对比，可合并 |
| PPT | HTML 幻灯片 | 代码即演示 |
| Excel | Python + CSV | 逻辑可版本控制、可复现 |

> Markdown > Word，Python > Excel。AI 降低了文本格式的成本，二进制不可版本控制的缺陷却不变。

---

## 二、为什么要用 Git

**经典理由**

| 理由 | 一句话 |
|------|-------|
| 防丢失 | 只要 commit 过，永远能找回 |
| 多人协作不打架 | 改不同位置自动合并，改同一行提示冲突 |
| 大胆试错 | 建分支随便改，好了合并，不好直接删 |
| 追溯一切 | 每行代码都能查到谁改的、什么时候、为什么 |

### AI 时代更需要 Git

AI 让代码的产生速度翻了几倍——你一句话，Agent 就写了一堆文件。但 AI 也会犯错、会误解你的意图、会在你以为没问题的地方埋坑。

**没有 Git：** AI 改了一个函数，你发现线上崩了，却不知道是哪个改动引起的，也回不去修改前的状态。

**有了 Git：**
- AI 的每次修改都有 snapshot，随时回退到让 AI 动手之前
- AI 同时改多个文件时，逐个 commit 审查，不满意就 revert
- 多个 AI Agent 协作时，各自开分支，冲突可控

> Git 是 AI 时代的「撤销键」。没有它，你只能祈祷 AI 每次都对。

---

## 三、怎么用 —— 理解概念，告诉 Agent

> **先看 3.1 和 3.2 就够了。** 这是你每天都会用到的。后面几节（合并、冲突、撤销、stash、reflog）是出问题时再查的，不用一次读完。

### 3.1 四层结构（add / commit / push）

```
工作目录              暂存区               本地仓库              远端仓库
(你改文件)  ──add──▶  (准备提交)  ──commit──▶  (存快照)  ──push──▶  (GitHub)
```

> 暂存区就像购物车——`add` 把改动放进去，`commit` 才结账保存。

| 你想做的事 | 对 Agent 说 | 对应命令 |
|-----------|------------|---------|
| 看看改了什么 | 「帮我看看仓库状态」 | `git status` |
| 提交修改 | 「帮我暂存并提交所有修改」 | `git add . && git commit -m "描述信息"` |
| 同步到 GitHub | 「帮我推送到远端仓库」 | `git push origin <分支名>` |
| 拉同事的更新 | 「帮我拉取最新代码」 | `git pull origin <分支名>` |

---

### 3.2 分支（branch / checkout）

分支是一条独立的开发线。新功能写到一半要修线上 bug，切回主线修完再回来，互不干扰。

```
master ──●──●──●──── 线上稳定版
               \
feature ──      ●──●──●  随便折腾
```

| 你想做的事 | 对 Agent 说 | 对应命令 |
|-----------|------------|---------|
| 开始新功能 | 「从 master 创建 feature-xxx 分支并切换过去」 | `git checkout -b feature-xxx` |
| 功能做完 | 「把 feature-xxx 合并到 master」 | `git merge feature-xxx` |
| 分支不要了 | 「删掉 feature-xxx 分支」 | `git branch -d feature-xxx` |

---

### 3.3 合并（merge / rebase）

**merge** —— 保留分叉，生成一个合并节点

```
合并前                           合并后
A──B──C  (master)              A──B──C──E  (master)
    \                              \  /
     D──E  (feature)                D─┘
```

**rebase** —— 把 feature 的提交「接」到 master 最新处，历史成一条直线

```
rebase 前                        rebase 后
A──B──C  (master)              A──B──C  (master)
    \                                 \
     D──E  (feature)                   D'──E'  (feature)
```

| | merge | rebase |
|---|---|---|
| 历史 | 保留真实分叉 | 拉成一条直线 |
| 安全性 | 不改写，安全 | 生成新 commit，原 commit 废弃 |
| 适用 | 公共分支、多人协作 | 个人分支 |

> **铁律：已推送且多人共用的分支绝不做 rebase。** 不确定就用 merge。

| 你想做的事 | 对 Agent 说 | 对应命令 |
|-----------|------------|---------|
| 合并分支 | 「帮我把 feature-xxx merge 到当前分支」 | `git merge feature-xxx` |
| 整理历史 | 「帮我把当前分支 rebase 到 master」 | `git rebase master` |

---

### 3.4 冲突

两个人改了同一文件的同一行，Git 无法自动取舍——这就是冲突。

| 解决方式 | 对 Agent 说 | 对应命令 |
|---------|------------|---------|
| 用我的版本 | 「帮我解决冲突，保留我的版本」 | 手动编辑冲突文件，然后 `git add . && git commit -m "解决冲突"` |
| 用同事的版本 | 「保留同事的版本」 | 手动编辑冲突文件，然后 `git add . && git commit -m "解决冲突"` |
| 两个都要 | 「两个都保留」 | 手动编辑冲突文件，然后 `git add . && git commit -m "解决冲突"` |

---

### 3.5 撤销（reset / revert）

| 场景 | 对 Agent 说 | 对应命令 |
|------|------------|---------|
| commit 完发现漏了文件 | 「帮我修改最近一次提交，补上 xxx」 | `git add 漏掉的文件 && git commit --amend` |
| 改乱了想全丢掉 | 「帮我丢弃所有未提交的修改」 | `git checkout .` |
| commit 了但没 push，想撤销 | 「帮我撤销最近一次 commit，但保留修改」 | `git reset --soft HEAD~1` |
| 已经 push 了，有问题 | 「帮我 revert 掉那个有问题的提交」 | `git revert <commit-hash>` |


---

### 3.6 暂存现场（stash）

写了一半要切去修 bug，但还不想 commit。

| 对 Agent 说 | 对应命令 |
|------------|---------|
| 「帮我把当前修改 stash 起来」 | `git stash` |
| 「帮我把 stash 恢复出来」 | `git stash pop` |

---

### 3.7 后悔药（reflog）

误删分支、`reset --hard` 了？只要 commit 没被垃圾回收（不可达条目默认保留 30 天，可达条目 90 天），就能找回。

| 对 Agent 说 | 对应命令 |
|------------|---------|
| 「帮我看 reflog，恢复误删的分支」 | `git reflog` → 找到 hash → `git reset --hard <hash>` |
| 「帮我恢复到 reset 之前的状态」 | `git reflog` → 找到 hash → `git reset --hard <hash>` |

---

## 四、每天对 Agent 说这几句就够了

| 场景 | 对 Agent 说 |
|------|------------|
| 早上开工 | 「帮我切到 master 分支，拉取最新代码」 |
| 开始新功能 | 「帮我从 master 建一个 feature-xxx 分支并切换过去」 |
| 写完一部分代码 | 「帮我暂存所有修改并提交」 |
| 开发期间同步主线 | 「帮我把当前分支 rebase 到 master」 |
| 功能完成，交给同事审查 | 「帮我推送到远端仓库，然后创建 Pull Request」 |

---

## 五、常见问题速查

| 问题 | 对 Agent 说 | 对应命令 |
|------|------------|---------|
| push 被拒，远端有更新 | 「先拉取合并，再推送」 | `git pull && git push`（会生成一个合并节点，安全） |
| add 了不该加的文件 | 「帮我取消暂存 xxx」 | `git reset HEAD <文件名>` |
| 想看某行谁改的 | 「帮我 blame xxx 文件第 N 行」 | `git blame <文件名>` |
| 切分支提示有未保存修改 | 「帮我 stash 当前修改，切分支，做完 stash pop」 | `git stash && git checkout <分支> && git stash pop` |
| detached HEAD 状态 | 「帮我基于当前 commit 创建新分支」 | `git checkout -b <新分支名>` |

---

## 六、进阶能力（知道存在即可）

| 能力 | 对 Agent 说 | 对应命令 |
|------|-----------|---------|
| 把某个 commit 单独拿来用 | 「帮我 cherry-pick xxx」 | `git cherry-pick <commit-hash>` |
| 把几个 commit 合并成一个 | 「帮我 rebase -i 整理最近 3 个 commit」 | `git rebase -i HEAD~3` |
| 二分定位 bug commit | 「帮我用 bisect 定位引入 bug 的提交」 | `git bisect start` |
| 从历史中删除敏感文件 | 「帮我删除历史中的 xxx 文件」（⚠️ 高危） | `git filter-repo --path <文件> --invert-paths` |

---

## 记住这五点就够了

1. **四层结构：** 工作目录 → 暂存区 → 本地仓库 → 远端仓库
2. **分支：** 独立开发线，互不干扰
3. **合并：** merge 安全，rebase 干净（多人分支禁用）
4. **撤销分场景：** 没 commit / commit 没 push / 已 push / revert 了合并——处理方式不同
5. **后悔药：** stash 暂存现场，reflog 找回丢失的一切

其余的事，告诉 Agent 就好。


---

## 附录：一个进阶坑（出问题时再看）

> 以下场景新手暂时遇不到，但如果你合并过分支又被 revert 了，回来看这里。

```
① dev 分支新增了 http.js

   master   ●────●  (没有 http.js)
                 ╲
   dev            ● (+http.js)

② 合并 dev → master

   master   ●────●────M  (合并后，master 也有了 http.js)
                 ╲  ╱
   dev            ●

③ 发现 bug，revert 合并

   master   ●────●────M────R  (R 把 http.js 删了)
                 ╲  ╱
   dev            ●  (http.js 还在)

④ dev 继续开发，再次合并 dev → master

   合并 base（分叉点 ●）:  http.js ❌ 没有
   master (R 节点):         http.js ❌ 已删
   dev (当前节点):           http.js ✅ 还在
                   │
                   ▼
   三向合并逻辑：dev 新增了 http.js，但 master 主动删了它
   → Git 判定「master 的决定优先」
   → http.js 消失！
```

> **根因：** revert 不是「撤销合并」，而是新增一个提交来删除文件。再次合并时，Git 看到 master 删了这个文件，就会照做。
>
> **正确做法：** revert 后删掉 dev，从 R 重新拉分支。或先对 R 再做一次 revert（撤销上一次撤销，让文件回来），再合并 dev。

**参考**
- [LZANE - 这才是真正的 Git：分支合并](https://www.lzane.com/tech/git-merge/)
- [LZANE - 这才是真正的 Git：实用技巧](https://www.lzane.com/tech/git-tips/)
- [Pro Git（中文版）](https://git-scm.com/book/zh/v2)
