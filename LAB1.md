# 实验 1 记录：第 14 组

现场检查时将打开本文件，按其中的链接核对。证据一律以链接或文本形式给出，不要只贴截图。将 `<填写>` 替换为本组的内容。main 受保护之后，修改本文件也须通过 PR。

## 1. 基本信息

| 项                | 内容                                                                                    |
| ----------------- | --------------------------------------------------------------------------------------- |
| 小组              | 第 14 组                                                                                |
| 仓库地址          | [https://github.com/jeff-060111/ops-lab1-14](https://github.com/jeff-060111/ops-lab1-14) |
| 代码载体          | 统一示例（Go，程序 `enroll-conflict`）                                                |
| runner 与语言版本 | `ubuntu-24.04`，Go `1.26.9`                                                         |
| 合并方式          | squash                                                                                  |

## 2. 任务 0：本地基线

构建主机（本组全组共用同一台，T0/T6 均在此机上执行）：华为云 ECS，主机名 `ops-lab1-main`，区域华东-上海一；架构 `x86_64`，系统 Ubuntu 24.04.4 LTS（内核 `6.8.0-136-generic`），4 vCPU / 8 GiB。本机 Go 版本为 `go1.26.9 linux/amd64`（`/usr/local/go/bin/go`），与 `ci.yml` 中固定的 `go-version` 完全一致。

| 问题                                                                                                 | 观察结果                                                                                                                                                                                                                                                                                                                                                                                         |
| ---------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 第一次 `make package` 得到的 BUILD_INFO（粘贴）                                                    | 见下方「第一次 BUILD_INFO」。                                                                                                                                                                                                                                                                                                                                                                    |
| 修改一行、不提交，再次打包：BUILD_INFO 是否变化？二进制文件的 sha256 是否变化？（贴出两次的 sha256） | **BUILD_INFO 逐字节完全不变**（`diff -u` 无任何输出，六个字段全部相同）；**二进制 sha256 与 tar.gz sha256 都发生了变化**。完整哈希见下方「两次 sha256」。                                                                                                                                                                                                                          |
| 两次 `go version -m` 输出中的 vcs 行有何不同？                                                     | 只有 `vcs.modified` 由 `false` 变为 `true`；`vcs.revision`（`7b63b41c13f061439997b0e6e4eaa2af922a7aa1`）与 `vcs.time`（`2026-10-12T16:26:07Z`）都没有变化。见下方「两次 vcs 行」。                                                                                                                                                                                                 |
| 仅凭 BUILD_INFO，能否回答该包是什么、在哪台机器上、何时、用哪份代码构建？哪一项信息会出错？          | **不能完整回答。**「是什么」可以回答（`name`/`version`/`build_no`）；「在哪台机器上」无法回答，因为没有主机标识；「何时」无法回答，因为没有构建时间戳；「用哪份代码」**会出错**：`commit` 由 Makefile 通过 `git rev-parse` 注入，未提交的改动不移动 HEAD，因此它仍指向旧提交，与实际编译的源码不一致。本地打包时 `run_id` 也是字面量 `local`，无法定位具体哪一次构建。 |

第一次 BUILD_INFO（原始源码，未做任何改动）：

```text
name=enroll-conflict
version=1.0.0
build_no=1
run_id=local
commit=7b63b41c13f061439997b0e6e4eaa2af922a7aa1
go=go1.26.9
```

第二次 BUILD_INFO（在 `conflict.go` 首行插入一行注释 `// T0 baseline probe`，**不提交**，用完全相同的参数 `VERSION=1.0.0 BUILD_NO=1` 重新打包）：

```text
name=enroll-conflict
version=1.0.0
build_no=1
run_id=local
commit=7b63b41c13f061439997b0e6e4eaa2af922a7aa1
go=go1.26.9
```

两次 BUILD_INFO 的 `diff -u` 结果：**无任何差异**（逐字节相同）。

两次 sha256：

```text
二进制 enroll-conflict
第一次：f23447d406c42af7dd35e23c2d1b2f15484cfb709cc9b4bb1262643694b8528e
第二次：0700b590351a5ac4a78a49e351247a33709d4efe6565adf54ef1743b8867304e

归档 enroll-conflict_1.0.0_linux_amd64.tar.gz
第一次：e22a256a9549ffdede6e9e6d0361582005369eb36d2619b564e412e190c255a2
第二次：c0108c388e181c8210adaa33d820d902c8c598bca8e3aaf7e509366ae3f83421
```

> 补充观察：本组在两个不同时间点各完整执行过一次上述两次打包，原始源码状态下得到的二进制 sha256 均为 `f23447d4…b8528e`，两次一致。这说明在 Go 版本、源码、构建参数都固定的前提下，`go build -trimpath` 产出的二进制是可重复的；而本次未提交改动后，哈希发生了变化。

两次 `go version -m` 的 vcs 行（其余行一致；以下按附件转录，制表符用 `\t` 表示）：

```text
第一次：
build\tvcs=git
build\tvcs.revision=7b63b41c13f061439997b0e6e4eaa2af922a7aa1
build\tvcs.time=2026-10-12T16:26:07Z
build\tvcs.modified=false

第二次：
build\tvcs=git
build\tvcs.revision=7b63b41c13f061439997b0e6e4eaa2af922a7aa1
build\tvcs.time=2026-10-12T16:26:07Z
build\tvcs.modified=true
```

**结论**：源码改动之后，制品（二进制与 tar.gz 的 sha256）都已经不同，但 `BUILD_INFO` 的六个字段一字未变，`vcs.revision` 也仍是改动前的旧提交号。可见「版本号 / build_no 相同」**不能**推出「源码相同」；`vcs.modified` 虽然变成了 `true`，但它只是一个布尔标志，只能说明「工作区不干净」，无法说明改了什么、改在哪一处。这正是需要由流水线自动注入 `github.sha` 并写入制品元数据的原因：手工传参加上未提交的改动，会使制品与其来源代码之间的关联断裂，无法回答「线上运行的二进制来自哪次提交」。

> 附注（留作第 6 节问题 3 的证据线索）：Makefile 第 48 行的打包命令为 `tar --sort=name --owner=0 --group=0 --numeric-owner --mtime=@0 -C $(DIST) -cf - $(PKG) | gzip -n`。其中 `--sort=name` 固定归档内文件的顺序，`--owner/--group/--numeric-owner` 固定属主信息，`--mtime=@0` 把归档内所有文件的修改时间统一为 Unix 时间 0，`gzip -n` 不写入文件名与时间戳（管道输入本就不写，保留是为了换成「压缩文件」写法时仍可重复）。这四项共同使 tar.gz 归档本身可重复。`BUILD_INFO` 中刻意不写构建时间，原因相同。问题 3 的 ④⑤ 要求临时去掉 `--mtime=@0` 后复测，正是为了验证这一项对哈希的影响。

## 3. 证据

| 项                                               | 链接或文本                                                                                                                                                                                                                                                                                                                                |
| ------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 规则集里启用的规则                               | 规则集 `protect-main`，作用目标为默认分支（Include: Default），Enforcement = Active，Bypass list 为空。启用 Restrict deletions、Block force pushes、Require a pull request before merging（Required approvals = 1）和 Require status checks to pass（必需检查为 job `ci`，并勾选 Require branches to be up to date before merging）。 |
| 组长直接 `git push origin main` 被拒的终端输出 | 见下方代码块。验证由仓库 Owner 账号 `jeff-060111` 执行。                                                                                                                                                                                                                                                                                |
| 第一轮：失败的 PR 运行                           | [https://github.com/jeff-060111/ops-lab1-14/actions/runs/39567086007](https://github.com/jeff-060111/ops-lab1-14/actions/runs/39567086007)                                                                                                                                                                                                 |
| 第一轮：修复后通过的 PR 运行                     | [https://github.com/jeff-060111/ops-lab1-14/actions/runs/39568404719](https://github.com/jeff-060111/ops-lab1-14/actions/runs/39568404719)                                                                                                                                                                                                 |
| 第一轮：合入 main 后的运行（含制品）             | [https://github.com/jeff-060111/ops-lab1-14/actions/runs/39568947726](https://github.com/jeff-060111/ops-lab1-14/actions/runs/39568947726)                                                                                                                                                                                                 |
| 第二轮：找到的缺陷，以及现有测试为何无法发现它   | 将 `Section.Valid()` 中的星期上界判断由 `s.Weekday > 7` 改为 `s.Weekday >= 7`，导致合法的周日（`Weekday=7`）课程被误判为非法。现有测试没有覆盖 `Weekday=7` 这一合法边界，因此引入缺陷后 `make test` 和 CI 仍然通过。                                                                                                          |
| 第二轮：仅含缺陷时仍然通过的运行                 | [https://github.com/jeff-060111/ops-lab1-14/actions/runs/39571101808](https://github.com/jeff-060111/ops-lab1-14/actions/runs/39571101808)                                                                                                                                                                                                 |
| 第二轮：补测试后失败的运行                       | [https://github.com/jeff-060111/ops-lab1-14/actions/runs/39571842625](https://github.com/jeff-060111/ops-lab1-14/actions/runs/39571842625)                                                                                                                                                                                                 |
| 第二轮：撤销缺陷后通过的运行                     | [https://github.com/jeff-060111/ops-lab1-14/actions/runs/39572259840](https://github.com/jeff-060111/ops-lab1-14/actions/runs/39572259840)                                                                                                                                                                                                 |
| 未合入分支上的 tag 被 release gate 拒绝的运行    | [https://github.com/jeff-060111/ops-lab1-14/actions/runs/39576523847](https://github.com/jeff-060111/ops-lab1-14/actions/runs/39576523847)                                                                                                                                                                                |
| Release v1.0.0                                   | [https://github.com/jeff-060111/ops-lab1-14/releases#release-v1.0.0](https://github.com/jeff-060111/ops-lab1-14/releases#release-v1.0.0)                                                                                                                                                                                                           |
| Release v1.0.1                                   | <链接>                                                                                                                                                                                                                                                                                                                                    |

直接推送 main 被规则集拒绝的终端记录。执行环境为华为云 ECS `ops-lab1-main`（Ubuntu 24.04 LTS），小组仓库位于 `~/lab1`：

```text
cd ~/lab1
git switch main
git pull
git status --porcelain
git commit --allow-empty -m "test: direct push"
git push origin main
remote: error: GH013: Repository rule violations found for refs/heads/main.
remote: - Changes must be made through a pull request.
remote: - Required status check "ci" is expected.
! [remote rejected] main -> main (push declined due to repository rule violations)
```

两条拒绝理由分别对应必须通过 PR 合入和必须通过 `ci` 检查。`git status --porcelain` 无输出；Owner 账号的直接推送仍被拒绝，与规则集的空 Bypass list 一致。验证结束后执行 `git reset --hard origin/main`，使本地 main 回到远端状态，不留下该空提交。

## 4. 追溯与运行

对 v1.0.0 和 v1.0.1 各运行一次 `scripts/trace.sh`，粘贴输出，并填写下表。

| 项                                                 | v1.0.0       | v1.0.1       |
| -------------------------------------------------- | ------------ | ------------ |
| 版本号                                             | <填写>       | <填写>       |
| 构建号                                             | <填写>       | <填写>       |
| 提交号（SHA）                                      | <填写>       | <填写>       |
| 引入该提交的 PR                                    | <编号与链接> | <编号与链接> |
| 构建它的 Actions 运行（`trace.sh` 给出的链接）   | <链接>       | <链接>       |
| 制品 sha256（发布当时，取自 Release 里的 .sha256） | <填写>       | <填写>       |

仅凭构建号能否唯一确定是哪次运行？v1.0.0 的构建号在 `ci` 和 `release` 的运行列表中各对应哪次运行？<填写>

发布 v1.0.1 之后重新下载 v1.0.0 的附件，计算得到的 sha256：<填写>。与上表中发布当时的值是否一致？<填写>

`trace.sh` 输出（v1.0.0）：

```text
<粘贴>
```

`trace.sh` 输出（v1.0.1）：

```text
<粘贴>
```

任务 6 第 4 步，两次 `/version` 的输出：

```text
<运行 v1.0.1 时>
<换回 v1.0.0 后>
```

## 5. 数据

以下时间按北京时间（Asia/Shanghai，UTC+08:00）记录；未取得精确时间的事件单独说明。

第一轮 gate 演练的时间线：

| 事件                           | 时间                                                                                             | 链接                                                                                                                                                                                    |
| ------------------------------ | ------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 发起 PR                        | 2026-10-21 16:08:49                                                                              | [https://github.com/jeff-060111/ops-lab1-14/pull/4](https://github.com/jeff-060111/ops-lab1-14/pull/4)                                                                                   |
| 第一次 `ci` 失败（运行结束） | 2026-10-21 16:09:19                                                                              | [https://github.com/jeff-060111/ops-lab1-14/actions/runs/39567086007](https://github.com/jeff-060111/ops-lab1-14/actions/runs/39567086007)                                               |
| 推送修复提交                   | 约 2026-10-21 16:19；修复提交时间为 16:19:14，更新 PR 的 CI 于 16:19:32 触发，未取得精确推送时间 | [https://github.com/jeff-060111/ops-lab1-14/commit/faa5e12c690a38216724d5cbbd0e66066b2fbb07](https://github.com/jeff-060111/ops-lab1-14/commit/faa5e12c690a38216724d5cbbd0e66066b2fbb07) |
| `ci` 通过（运行结束）        | 2026-10-21 16:20:04                                                                              | [https://github.com/jeff-060111/ops-lab1-14/actions/runs/39568404719](https://github.com/jeff-060111/ops-lab1-14/actions/runs/39568404719)                                               |
| 合入 main                      | 2026-10-21 16:23:48                                                                              | [https://github.com/jeff-060111/ops-lab1-14/pull/4](https://github.com/jeff-060111/ops-lab1-14/pull/4)                                                                                   |

提交到发布的间隔，至少记录三个合入的 PR：

| PR      | 第一个提交的时间 | 合入时间 | 对应 Release 的发布时间 | 提交到发布的间隔 |
| ------- | ---------------- | -------- | ----------------------- | ---------------- |
| #<编号> | <填写>           | <填写>   | <填写>                  | <填写>           |
| #<编号> | <填写>           | <填写>   | <填写>                  | <填写>           |
| #<编号> | <填写>           | <填写>   | <填写>                  | <填写>           |

`ci` 工作流的运行时长（从 Actions 页面读取），至少 5 次，全部列出，不要只给出平均值：

| 运行    | 事件（PR / push） | 时长   | 备注   |
| ------- | ----------------- | ------ | ------ |
| #<编号> | <填写>            | <填写> | <填写> |

局限说明：`ci` 时长受共享 runner 负载影响，单次噪声大，因此上表应列出全部运行而非只给平均值，且样本量很小；「提交到发布的间隔」借用 DORA 的变更前置时间概念，但本组只有 3 个 PR 的样本，不能代表真实团队水平。

## 6. 分析问题

每题先列出所用数据（可引用上面的表），再用一段话（4～8 句）写出结论，并说明结论的成立条件。

**问题 1：gate 的收益与成本。** 两轮演练中，gate 拦住了什么，没有拦住什么？用第 5 节的时间线计算：第一轮因失败多花了多少时间，其中等待 `ci` 的时间和人工修改的时间各占多少？一次 `ci` 大约需要多久，在“提交到发布的间隔”中占多大比例？第二轮补充测试之后，还有哪类问题是测试本身无法发现的？

<填写>

**问题 2：PR 中的构建与 main 上的构建。** 在 PR 触发的运行日志中，`github.sha` 与 `pull_request.head.sha` 是否相同？它们能否在 main 的历史中找到？答案与合并方式有关，请写明本组使用的方式。合并之后，main 上这次改动的提交号是哪个？据此说明，为什么待发布的制品只能从 main 上的提交或 tag 构建。

<填写>

**问题 3：可重复构建。** 在本地同一个提交上按下表构建，记录哈希值。④、⑤ 需临时去掉 Makefile 中 tar 的 `--mtime=@0`，完成后用 `git checkout -- Makefile` 恢复，不要提交。

| 构建    | 条件                                                                                | 二进制 sha256 | tar.gz sha256 |
| ------- | ----------------------------------------------------------------------------------- | ------------- | ------------- |
| ①      | `make package`，`VERSION`、`BUILD_NO`、`RUN_ID`、`COMMIT` 均指定固定值    | <填写>        | <填写>        |
| ②      | 与 ① 完全相同，再构建一次                                                          | <填写>        | <填写>        |
| ③      | 仅将 `BUILD_NO` 换成另一个值                                                      | <填写>        | <填写>        |
| ④      | 去掉 `--mtime=@0`，其余同 ①                                                      | <填写>        | <填写>        |
| ⑤      | 同 ④，间隔几秒后再构建一次                                                         | <填写>        | <填写>        |
| Actions | 对 main 上的一次 push 运行执行 Re-run all jobs，比较两次尝试中 package 步骤打印的值 | <两次的值>    | <两次的值>    |

哪些相同、哪些不同，原因分别是什么？④ 的二进制文件与 ① 是否相同？如果不同，请结合任务 0 中看到的 vcs 信息分析。据此说明制品中应当包含什么、不应包含什么。另外，`release` 在同一个提交上再次编译，而没有直接发布 `ci` 产出的主干制品：为什么这样做？这与 L-05 第 11 页的“一次构建、多处部署”是否冲突，代价是什么？

<填写>

**问题 4：发布与回退。** 假设 v1.0.1 发布后发现缺陷，应当如何处理？为什么不应删除 v1.0.1 的 tag、再用同一版本号重新发布？任务 6 中将服务从 v1.0.1 切换回 v1.0.0 时，使用的是哪个文件，是否需要重新构建？请用两次 `/version` 的输出说明。如果改为从源码重新构建 v1.0.0，得到的文件与原附件是否相同（参考问题 3）？最后，如果有人删除 v1.0.0 的附件，换上一个新文件和对应的新 `.sha256`，`trace.sh` 能否发现？本组在第 4 节记录的值能否发现？可以查阅 GitHub 的 Immutable releases 设置的用途。

<填写>

## 7. 选做记录

| 选做                    | 完成情况与证据 |
| ----------------------- | -------------- |
| O1 容器镜像             | 未做           |
| O2 缓存对反馈时间的影响 | 未做           |
| O3 迁移到小组项目       | 未做           |

## 8. 成员贡献

每位成员一行，只填写 GitHub 用户名，无需填写姓名；行数应与小组人数一致。

| GitHub 用户名 | 发起并合入的 PR 编号 | 审批过的 PR 编号 |
| ------------- | -------------------- | ---------------- |
| jeff-060111   | <编号>               | <编号>           |
