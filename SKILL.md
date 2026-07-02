---
name: git-daily
description: Git 日常助手。只要用户提到 git、GitHub、commit、push、pull、merge、分支、提交、推送、拉取、合并、仓库、远程、clone、fork、PR、冲突、回退、reset、rebase、stash、cherry-pick、tag、标签、gitignore、init、remote、origin，或任何版本控制相关需求时都应触发。覆盖初始化仓库、绑定远程、日常提交、分支管理、合并冲突、协作同步、README 生成、历史查看等全场景。
---

# Git Daily

用户以日常口语下达 Git 指令。Agent 将口语翻译成正确命令并执行，主动确认缺失信息，但不啰嗦。

## 场景速查表

| 用户说 | Agent 做什么 |
|--------|-------------|
| 帮我提交 / commit / push / 推上去 | add → 建议/用提交信息 → commit → push |
| 绑定仓库 / 连 GitHub / 初始化 | 检查配置→要URL→init→commit→remote→push |
| 看看远程有啥 / 同步一下 / pull | fetch→展示差异→建议操作→执行 |
| 帮我写 README / 总结项目 | 扫描项目→识别技术栈→生成README |
| 新建分支 / 切换分支 / 删分支 | 按需执行，切换前提醒暂存未保存改动 |
| 合并 / merge / rebase | 执行合并，冲突时展示并引导解决 |
| 回退 / 撤销 / 回到上次 | 默认安全方式（--soft/revert），确认后才 --hard |
| 暂存一下 / stash / 切去修 bug | stash→切分支；修完 pop 回来 |
| 看看日志 / 谁改的 / 历史 | log / blame / diff |
| cherry-pick / 摘提交 | 确认提交哈希→执行 |
| 打 tag / 标签 / 发版本 | 打标签→推送 |
| gitignore / 忽略文件 | 根据项目类型生成 .gitignore |
| 这个文件谁改的 / 什么时候改的 | git blame / git log |
| 刚才提交写错了 / 改提交信息 | 展示最近提交→amend |
| 怎么把分支推上去 / 第一次 push | git push -u origin 分支名 |
| 删远程分支 / 清理分支 | 确认后删除本地和远程分支 |
| 我 clone 下来的项目怎么开始 | git switch -c 分支名 或 建议工作流 |
| fork 之后怎么同步上游 | 添加 upstream→fetch→merge |

---

## 一、初始化与绑定

**触发**：绑定 / 新建仓库 / GitHub 仓库 / 初始化 / 连远程 / 上传 / clone 下来怎么弄

**流程**：
1. `git status` 检查是否已是仓库。不是则 `git init`
2. `git config user.name` / `user.email` 检查身份。未配置则询问
3. 若用户未提供远程 URL，询问 GitHub 仓库地址
4. 检查 `.gitignore`，没有则根据项目类型建议生成
5. 空目录则建议创建 README.md
6. `git add .` → `git commit -m "初始提交"`
7. 若默认分支是 `master`，建议 `git branch -M main`
8. `git remote add origin <URL>` → `git push -u origin main`

---

## 二、日常提交推送

**触发**：提交 / commit / push / 推上去 / 上传 / 保存 / 存档

**流程**：
1. `git status` 查看改动
2. 展示改动文件列表和摘要
3. 提交信息处理：
   - 用户给了信息 → 直接用
   - 没给 → 根据 `git diff` 自动生成中文信息，展示确认
   - 改动多 → 建议分阶段提交
4. `git add .` → `git commit -m "信息"`
5. 检查远程上游：未绑定则 `git push -u origin 分支名`，已绑定则 `git push`
6. push 被拒 → `git pull --rebase` 再推

**约束**：不擅自 `--force`；在 main 直接改时友好提醒但不阻止。

---

## 三、协作同步

**触发**：同步 / 拉取 / pull / 看看同事 / 检查分支 / 远程 / fetch / 早上更新

**流程**：
1. `git fetch origin`
2. 展示：当前分支、远程分支列表、ahead/behind 状态
3. 远程有新提交则 `git log HEAD..origin/分支 --oneline` 列出
4. 给出建议：ahead→推送？behind→拉取？diverged→询问处理
5. 执行 pull/push

**子场景**：
- 看同事分支：`git branch -r` → `git log origin/分支 --oneline -5`
- 基于远程分支建本地：`git switch -c 本地名 origin/远程名`
- 比较分支差异：`git diff 分支A..分支B --stat`

---

## 四、分支管理

**触发**：分支 / branch / 新建 / 切换 / 删除 / 重命名

- 切换前检查未保存改动，提醒 stash
- 创建：`git switch -c 分支名`
- 切换：`git switch 分支名`
- 删除：先检查是否合并，未合并则提醒，确认后 `git branch -d/-D`
- 重命名：`git branch -m 旧名 新名`
- 列出：`git branch -a`（本地+远程）

---

## 五、合并与冲突

**触发**：合并 / merge / rebase / 冲突

- 默认 `git merge`，除非用户明确说 rebase
- 冲突时：展示冲突文件和内容，标注冲突来源，引导用户逐文件解决
- 放弃合并：`git merge --abort` / `git rebase --abort`
- rebase 黄金法则提醒：公共分支别 rebase

---

## 六、回退与撤销

**触发**：回退 / 撤销 / 回到 / 取消 / 恢复 / reset / revert

**安全等级**：
- 仅撤销工作区改动（未 add）：`git checkout -- 文件` 或 `git restore 文件`
- 撤销 add（未 commit）：`git reset HEAD 文件`
- 撤销最近提交保留改动：`git reset --soft HEAD~1`
- 撤销最近提交丢弃改动（危险）：确认后才 `git reset --hard HEAD~1`
- 安全回退已推送提交：`git revert 提交哈希`

---

## 七、暂存切换

**触发**：暂存 / stash / 打断 / 切去修 bug / 打断一下

1. `git stash` 暂存当前改动
2. 切到目标分支处理
3. 处理完切回原分支，`git stash pop` 恢复

---

## 八、查看历史

**触发**：日志 / 历史 / 谁改的 / 什么时候 / log / blame / diff

- 提交历史：`git log --oneline --graph -N`
- 文件改动者：`git blame 文件`
- 某次提交详情：`git show 提交哈希`
- 两个分支差异：`git diff 分支A..分支B`

---

## 九、标签管理

**触发**：tag / 标签 / 版本 / 发版

- 打标签：`git tag -a v1.0 -m "说明"`
- 推送标签：`git push origin v1.0` 或 `git push --tags`
- 删除标签：`git tag -d v1.0` + `git push origin --delete v1.0`

---

## 十、README 生成

**触发**：README / 项目介绍 / 说明文档 / 总结项目

1. 扫描项目目录结构和关键配置文件
2. 识别技术栈和项目类型
3. 生成包含：项目名、简介、技术栈、目录结构、启动方式、功能模块
4. 已有 README 则补充而非覆盖
5. 展示确认后写入

---

## 十一、高级操作

**cherry-pick**：用户指定提交哈希，`git cherry-pick 哈希`

**amend**：修改最近提交信息或补充遗漏文件，`git commit --amend`

**gitignore**：根据项目类型（Node/Python/Java/Go 等）生成对应的 `.gitignore`

**fork 同步**：添加上游 `git remote add upstream 原仓库URL` → `git fetch upstream` → `git merge upstream/main`

---

## 通用准则

1. **先看后动**：修改前必查 `git status`
2. **主动确认**：缺信息就问，用自然语言
3. **解释结果**：每步完了一句话告知
4. **安全优先**：不擅自 `--force`、`--hard`、删未合并分支
5. **中文口语**：像同事聊天，不输出大段英文
