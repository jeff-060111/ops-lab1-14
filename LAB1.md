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
| Release v1.0.0                                   | <https://github.com/jeff-060111/ops-lab1-14/releases/tag/v1.0.0>                                                                                                                                                                                                           |
| Release v1.0.1                                   | <https://github.com/jeff-060111/ops-lab1-14/releases/tag/v1.0.1>                                                                                                                                                                                                                                                                                                                                    |

两轮 gate 演练分别见 [第一轮 PR #4](https://github.com/jeff-060111/ops-lab1-14/pull/4) 和 [第二轮 PR #5](https://github.com/jeff-060111/ops-lab1-14/pull/5)；两个 PR 页面均保留了 `ci` 失败的记录。release gate 测试结束后，远端 tag `v0.0.1` 和分支 `scratch/gate-test` 已删除，且未生成 `v0.0.1` Release。

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

对 v1.0.0 和 v1.0.1 各运行一次 `scripts/trace.sh`，输出与追溯结果如下。

| 项 | v1.0.0 | v1.0.1 |
|---|---|---|
| 版本号 | `1.0.0` | `1.0.1` |
| 构建号 | `1` | `3` |
| 提交号（SHA） | `e2acab7b9d372db8f9e87dd5764d3ad928618f02` | `15c1efa7d0a5296bababf36f433ac00ed2492c32` |
| 引入该提交的 PR | [#6](https://github.com/jeff-060111/ops-lab1-14/pull/6) | [#7](https://github.com/jeff-060111/ops-lab1-14/pull/7) |
| 构建它的 Actions 运行（`trace.sh` 给出的链接） | <https://github.com/jeff-060111/ops-lab1-14/actions/runs/39574857048> | <https://github.com/jeff-060111/ops-lab1-14/actions/runs/39584254408> |
| 制品 sha256（发布当时，取自 Release 里的 `.sha256`） | `2fb80a7fa53d4a6dbfc1b29b9f88ac3484a9bb9c04315b5033201070d23edbc0` | `65d1d9946dcc664de70f41681a27f0265e440ff4694a7db4e8a459fef4b5be1f` |

仅凭构建号不能唯一确定是哪次运行。`build_no` 取自 `github.run_number`，每个工作流各自从 1 开始计数。`v1.0.0` 的构建号是 1，但仓库中同时存在 `ci` 的[第 1 次运行](https://github.com/jeff-060111/ops-lab1-14/actions/runs/39558591004)（run 39558591004，提交 `ef89a31`，事件为 push 到 main）和 `release` 的[第 1 次运行](https://github.com/jeff-060111/ops-lab1-14/actions/runs/39574857048)（run 39574857048，提交 `e2acab7`，事件为 tag `v1.0.0`），所以 `build_no=1` 不能唯一定位运行。`BUILD_INFO` 中的 `run_id` 取自全局唯一的 `github.run_id`，`trace.sh` 据此给出运行链接。`v1.0.1` 的构建号是 3，因为中间的 [release 运行](https://github.com/jeff-060111/ops-lab1-14/actions/runs/39576523847)（tag `v0.0.1`）虽在 release gate 处失败，仍占用了构建号 2。

发布 `v1.0.1` 后重新下载 `v1.0.0` 附件，计算得到的 sha256 为 `2fb80a7fa53d4a6dbfc1b29b9f88ac3484a9bb9c04315b5033201070d23edbc0`，与发布时记录的值完全一致，说明所下载的 Release 附件内容未变化。

`trace.sh` 输出（v1.0.0）：

```text
== 1. 校验和
enroll-conflict_1.0.0_linux_amd64.tar.gz: OK
== 2. 制品里的 BUILD_INFO
name=enroll-conflict
version=1.0.0
build_no=1
run_id=39574857048
commit=e2acab7b9d372db8f9e87dd5764d3ad928618f02
go=go1.26.9
== 3. 提交 e2acab7b9d372db8f9e87dd5764d3ad928618f02
作者: WANG YIBO <wyibo8981@gmail.com>
时间: 2026-10-21T17:07:55+08:00
标题: ci: add tag release workflow (#6)
主干: 该提交在 origin/main 的历史上
指向它的 tag: v1.0.0
版本号: 1.0.0    构建号: 1
提交页面（含关联的 PR）: https://github.com/jeff-060111/ops-lab1-14/commit/e2acab7b9d372db8f9e87dd5764d3ad928618f02
构建它的运行: https://github.com/jeff-060111/ops-lab1-14/actions/runs/39574857048
```

`trace.sh` 输出（v1.0.1）：

```text
== 1. 校验和
enroll-conflict_1.0.1_linux_amd64.tar.gz: OK
== 2. 制品里的 BUILD_INFO
name=enroll-conflict
version=1.0.1
build_no=3
run_id=39584254408
commit=15c1efa7d0a5296bababf36f433ac00ed2492c32
go=go1.26.9
== 3. 提交 15c1efa7d0a5296bababf36f433ac00ed2492c32
作者: jeff-060111 <wwj051212@sjtu.edu.cn>
时间: 2026-10-21T18:05:54+08:00
标题: Add version section to README (#7)
主干: 该提交在 origin/main 的历史上
指向它的 tag: v1.0.1
版本号: 1.0.1    构建号: 3
提交页面（含关联的 PR）: https://github.com/jeff-060111/ops-lab1-14/commit/15c1efa7d0a5296bababf36f433ac00ed2492c32
构建它的运行: https://github.com/jeff-060111/ops-lab1-14/actions/runs/39584254408
```

任务 6 第 4 步，两次 `/version` 的输出（先运行 v1.0.1，再换回 v1.0.0）：

```text
{"version":"1.0.1","commit":"15c1efa7d0a5296bababf36f433ac00ed2492c32","build_no":"3","go":"go1.26.9"}
{"version":"1.0.0","commit":"e2acab7b9d372db8f9e87dd5764d3ad928618f02","build_no":"1","go":"go1.26.9"}
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

提交到发布的间隔，记录四个已合入的 PR。起点取各 PR 中第一个提交的时间，终点取首次包含该 PR 的 Release 的 `published_at`，间隔为两者之差；时间均已换算为北京时间。

| PR | 第一个提交的时间 | 合入时间 | 对应 Release 的发布时间 | 提交到发布的间隔 |
| --- | --- | --- | --- | --- |
| [#4](https://github.com/jeff-060111/ops-lab1-14/pull/4) | [2026-10-21 16:04:24](https://github.com/jeff-060111/ops-lab1-14/commit/f9732523ae1c353e60bc44b580599540f7b5a6fe) | 2026-10-21 16:23:48 | [v1.0.0：2026-10-21 17:10:54](https://github.com/jeff-060111/ops-lab1-14/releases/tag/v1.0.0) | 1 小时 6 分 30 秒 |
| [#5](https://github.com/jeff-060111/ops-lab1-14/pull/5) | [2026-10-21 16:39:35](https://github.com/jeff-060111/ops-lab1-14/commit/914608e0139e9fc9220bcf4d74af14767179399b) | 2026-10-21 16:56:40 | [v1.0.0：2026-10-21 17:10:54](https://github.com/jeff-060111/ops-lab1-14/releases/tag/v1.0.0) | 31 分 19 秒 |
| [#6](https://github.com/jeff-060111/ops-lab1-14/pull/6) | [2026-10-21 17:05:15](https://github.com/jeff-060111/ops-lab1-14/commit/0042ddf095ac3424d7967f6f2efb82c6a0bd17b0) | 2026-10-21 17:07:55 | [v1.0.0：2026-10-21 17:10:54](https://github.com/jeff-060111/ops-lab1-14/releases/tag/v1.0.0) | 5 分 39 秒 |
| [#7](https://github.com/jeff-060111/ops-lab1-14/pull/7) | [2026-10-21 18:03:36](https://github.com/jeff-060111/ops-lab1-14/commit/d2cdbb2fb0048458c6fb378d3513305c3f33cc30) | 2026-10-21 18:05:55 | [v1.0.1：2026-10-21 18:24:17](https://github.com/jeff-060111/ops-lab1-14/releases/tag/v1.0.1) | 20 分 41 秒 |

`ci` 工作流的运行时长（采用 Actions 页面显示的 **Total duration**，而非单个 job 的时长）：

| 运行 | 事件（PR / push） | 时长 | 备注 |
| --- | --- | --- | --- |
| [#7](https://github.com/jeff-060111/ops-lab1-14/actions/runs/39567086007) | PR #4 | 28 秒 | 第一轮，失败；测试拦住了引入的缺陷 |
| [#8](https://github.com/jeff-060111/ops-lab1-14/actions/runs/39568404719) | PR #4 | 33 秒 | 第一轮，修复后通过 |
| [#9](https://github.com/jeff-060111/ops-lab1-14/actions/runs/39568947726) | push 到 main | 52 秒 | 第一轮合入后通过，生成制品 |
| [#10](https://github.com/jeff-060111/ops-lab1-14/actions/runs/39571101808) | PR #5 | 23 秒 | 第二轮，引入未覆盖的缺陷后仍通过 |
| [#11](https://github.com/jeff-060111/ops-lab1-14/actions/runs/39571842625) | PR #5 | 23 秒 | 第二轮，补充边界测试后失败 |
| [#12](https://github.com/jeff-060111/ops-lab1-14/actions/runs/39572259840) | PR #5 | 27 秒 | 第二轮，撤销缺陷后通过 |

六次运行的时长为 23～52 秒，算术平均为 31 秒。局限说明：`ci` 时长受共享 runner 负载影响，单次噪声大，且本组样本量很小；「提交到发布的间隔」借用 DORA 的变更前置时间概念，这四个 PR 的样本不能代表真实团队水平。

## 6. 分析问题

每题先列出所用数据（可引用上面的表），再用一段话（4～8 句）写出结论，并说明结论的成立条件。

**问题 1：gate 的收益与成本。** 两轮演练中，gate 拦住了什么，没有拦住什么？用第 5 节的时间线计算：第一轮因失败多花了多少时间，其中等待 `ci` 的时间和人工修改的时间各占多少？一次 `ci` 大约需要多久，在“提交到发布的间隔”中占多大比例？第二轮补充测试之后，还有哪类问题是测试本身无法发现的？

所用数据：第 3 节两轮 gate 的 [PR #4](https://github.com/jeff-060111/ops-lab1-14/pull/4)、[PR #5](https://github.com/jeff-060111/ops-lab1-14/pull/5) 及第 5 节的时间线、四个提交到发布间隔和六次 `ci` 时长。

第一轮的 [失败运行](https://github.com/jeff-060111/ops-lab1-14/actions/runs/39567086007)在 `make test` 拦住了缺陷；第二轮仅引入周日边界缺陷时 [CI 仍通过](https://github.com/jeff-060111/ops-lab1-14/actions/runs/39571101808)，补充 `Weekday=7` 测试后才 [失败](https://github.com/jeff-060111/ops-lab1-14/actions/runs/39571842625)，说明 gate 的效果取决于测试覆盖范围。以首次失败结果 16:09:19 到修复后通过 16:20:04 计算，失败带来的额外时间为 **10 分 45 秒**：到第二次 CI 触发的人工修改与推送窗口为 **10 分 13 秒**，第二次 CI 从触发到通过为 **32 秒**（页面显示总时长 33 秒）；首次 CI 从 PR 创建到失败另耗约 30 秒。六次 CI 为 **23～52 秒，平均 31 秒**，约占第 5 节四个提交到发布间隔的 **0.8%～9.1%**，例如 PR #6 的 5 分 39 秒中约占 9.1%。即使补上周日测试，错误的需求假设、未覆盖的输入组合以及发布来源不在 main 等问题仍需评审和 release gate；这些比例只反映本组少量运行，且人工窗口并非纯编码耗时。

**问题 2：PR 中的构建与 main 上的构建。** 在 PR 触发的运行日志中，`github.sha` 与 `pull_request.head.sha` 是否相同？它们能否在 main 的历史中找到？答案与合并方式有关，请写明本组使用的方式。合并之后，main 上这次改动的提交号是哪个？据此说明，为什么待发布的制品只能从 main 上的提交或 tag 构建。

所用数据：[PR #4 修复后的 CI 日志](https://github.com/jeff-060111/ops-lab1-14/actions/runs/39568404719/job/119116238666)记录 `github.ref=refs/pull/4/merge`、`github.sha=83da91d26028e3fe5e130e05f5604bc8729b09b6`、`pull_request.head.sha=faa5e12c690a38216724d5cbbd0e66066b2fbb07`；[合入后的 main 运行](https://github.com/jeff-060111/ops-lab1-14/actions/runs/39568947726/job/119118013180)记录 `github.sha=b17b81fa799cedcbe281546e962cc6e99df42745`。

这三个 SHA 各不相同：PR 的 `github.sha` 指向 GitHub 为测试生成的临时合并提交，`head.sha` 指向 PR 分支上的修复提交。按照本组的 squash 合并方式，main 新建的提交是 `b17b81f…`；前两个 SHA 均不在 main 的提交历史中，但 PR 页面仍保留分支提交与运行记录。PR 检查验证的是合并前的候选结果，只有 main 上的 squash 提交或指向 main 历史的 tag 才对应正式版本来源，因此 `ci.yml` 仅在 push 到 main 时打包，`release.yml` 还用祖先关系检查 tag。以上判断针对本组使用的 `pull_request` 触发器和 squash 方式；[GitHub 对 PR 事件的说明](https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows#pull_request)也区分临时合并 SHA 与分支 head SHA。

**问题 3：可重复构建。** 在本地同一个提交上按下表构建，记录哈希值。④、⑤ 需临时去掉 Makefile 中 tar 的 `--mtime=@0`，完成后用 `git checkout -- Makefile` 恢复，不要提交。

本地复测环境为 WSL Ubuntu 24.04（x86_64）、Go `1.26.9 linux/amd64`、GNU tar 1.35，在干净的 `v1.0.1` 提交 `15c1efa7…` 上构建。①②④⑤固定 `VERSION=1.0.1 BUILD_NO=101 RUN_ID=local-q3 COMMIT=15c1efa7d0a5296bababf36f433ac00ed2492c32`；③仅把 `BUILD_NO` 改为 `102`。④⑤临时删去 Makefile 的 `--mtime=@0`，两次间隔 3 秒；复测后已恢复临时实验仓库的 Makefile，未提交任何实验改动。

| 构建 | 条件 | 二进制 sha256 | tar.gz sha256 |
| --- | --- | --- | --- |
| ① | 固定四项参数 | `470b8d8ef87f22f8e8debe21eb1150fadeb5116c6d887d06981ede16e45b4135` | `321ce276837443789c1cdb15d24bd1aed0c20be9c4cd437a39347efb5cf79202` |
| ② | 与①完全相同 | `470b8d8ef87f22f8e8debe21eb1150fadeb5116c6d887d06981ede16e45b4135` | `321ce276837443789c1cdb15d24bd1aed0c20be9c4cd437a39347efb5cf79202` |
| ③ | 仅 `BUILD_NO=102` | `2da24d057b7351daf44f2e88b967b40fd2952f0571faab6e590bbed290c701fe` | `0c04faa2a2f12e558603df3008c5df2a246863e20ef1a8e0afc15bda137a774c` |
| ④ | 去掉 `--mtime=@0`，其余同① | `b00e780bbc75a325b5d7aa24a19aa8ca716265933c419c756f084acaaee6a023` | `253a0c5c32e35ea44d32764c91c1f3c09e260461ce53d313f1c6810ab1d608c3` |
| ⑤ | 同④，间隔 3 秒 | `b00e780bbc75a325b5d7aa24a19aa8ca716265933c419c756f084acaaee6a023` | `bbdfad7175b41a115bc7f1f805ba3d25a61c2f50a10e71d8ebcee46c028c27a1` |
| Actions | [main push #17：首次](https://github.com/jeff-060111/ops-lab1-14/actions/runs/39582054130/job/119161766937)与 [Re-run 第 2 次](https://github.com/jeff-060111/ops-lab1-14/actions/runs/39582054130/job/119192615995) | 首次、重跑均为 `4bf4046162f9cc8a1bc481a030fbcb6d221bb6055d282a55f9965a5cbc321d21` | 首次、重跑均为 `7c5adaf7321fb3b32d479912c3f6468e473a0fe09878e4959c87b06cbff6acd8` |

哪些相同、哪些不同，原因分别是什么？④ 的二进制文件与 ① 是否相同？如果不同，请结合任务 0 中看到的 vcs 信息分析。据此说明制品中应当包含什么、不应包含什么。另外，`release` 在同一个提交上再次编译，而没有直接发布 `ci` 产出的主干制品：为什么这样做？这与 L-05 第 11 页的“一次构建、多处部署”是否冲突，代价是什么？

①②及 Actions 的两次尝试分别得到相同哈希，说明在这些固定输入和工具链下可以复现；Actions 重跑仍使用原来的 `run_id`、`run_number` 和提交，只增加 `run_attempt`。③的 `BUILD_NO` 写入二进制和 `BUILD_INFO`，所以两个哈希都变。④的二进制与①不同：修改 Makefile 使工作区变脏，`go version -m` 的 `vcs.modified` 从 `false` 变为 `true`，而 `vcs.revision` 仍为 `15c1efa7…`；④⑤同为脏工作区，二进制相同。④⑤的归档哈希不同，因为去掉 `--mtime=@0` 后 tar 记录了新生成文件的时间；因此制品应保留版本、提交、构建号、运行 ID、Go 版本等追溯信息，并固定归档元数据，避免把每次变化的构建时间或随机值写入制品。`release` 重新编译，是因为 main 上 `ci` 制品的版本为 `0.0.0-main`，发布时还要把 `1.0.0` 或 `1.0.1` 写进二进制；这与“一次构建、多处部署”的同一制品复用原则有冲突，代价是多一次构建及环境漂移风险。精确复现仍以源码干净、Go 版本、构建参数及归档工具行为一致为条件；若要复用同一制品，应在发布前确定版本并只构建一次，再将经检验的制品提升到 Release。

**问题 4：发布与回退。** 假设 v1.0.1 发布后发现缺陷，应当如何处理？为什么不应删除 v1.0.1 的 tag、再用同一版本号重新发布？任务 6 中将服务从 v1.0.1 切换回 v1.0.0 时，使用的是哪个文件，是否需要重新构建？请用两次 `/version` 的输出说明。如果改为从源码重新构建 v1.0.0，得到的文件与原附件是否相同（参考问题 3）？最后，如果有人删除 v1.0.0 的附件，换上一个新文件和对应的新 `.sha256`，`trace.sh` 能否发现？本组在第 4 节记录的值能否发现？可以查阅 GitHub 的 Immutable releases 设置的用途。

所用数据：第 4 节的两次 `/version`、v1.0.0 发布及重下后的 tar.gz sha256 `2fb80a7fa53d4a6dbfc1b29b9f88ac3484a9bb9c04315b5033201070d23edbc0`，以及 [v1.0.0](https://github.com/jeff-060111/ops-lab1-14/releases/tag/v1.0.0)、[v1.0.1](https://github.com/jeff-060111/ops-lab1-14/releases/tag/v1.0.1) 的附件。

若 v1.0.1 有缺陷，先把服务切回已验证的 v1.0.0 附件，再经 PR 修复、测试并发布新版本（如 v1.0.2）；保留 v1.0.1 的 tag 与 Release，避免同一个版本号先后指向不同代码和文件。任务 6 回退时重新使用 `enroll-conflict_1.0.0_linux_amd64.tar.gz` 中的二进制，无需重新构建；`/version` 从 `1.0.1`、提交 `15c1efa7…`、构建号 `3`，变为 `1.0.0`、提交 `e2acab7b…`、构建号 `1`。本组另在干净的 `v1.0.0` 提交上用 Go 1.26.9 和发布时相同的 `VERSION=1.0.0 BUILD_NO=1 RUN_ID=39574857048 COMMIT=e2acab7b9d372db8f9e87dd5764d3ad928618f02` 重新打包，tar.gz 的 sha256 **也得到 `2fb80a7f…edbc0`**；这一复现依赖源码、工具链、参数和归档行为一致，按本地默认 `RUN_ID=local` 构建则不会得到同一文件。如果有人同时替换 tar.gz 和 `.sha256`，且伪造的 `BUILD_INFO` 仍指向 main 上的提交，`trace.sh` 的成对校验无法识别替换；第 4 节事先独立记录的完整哈希可以识别新附件的差异。[v1.0.0](https://api.github.com/repos/jeff-060111/ops-lab1-14/releases/tags/v1.0.0) 与 [v1.0.1](https://api.github.com/repos/jeff-060111/ops-lab1-14/releases/tags/v1.0.1) 的 Release API 字段当前均为 `immutable=false`；[GitHub 的 Immutable releases](https://docs.github.com/en/code-security/concepts/supply-chain-security/immutable-releases) 启用后可锁定已发布附件及关联 tag，但仅对启用后的发布生效。

## 7. 选做记录

| 选做                    | 完成情况与证据 |
| ----------------------- | -------------- |
| O1 容器镜像             | 已做；见下方 [v1.0.2 发布运行](https://github.com/jeff-060111/ops-lab1-14/actions/runs/39591801985) 与 [GHCR 镜像](https://github.com/jeff-060111/ops-lab1-14/pkgs/container/ops-lab1-14) |
| O2 缓存对反馈时间的影响 | 已做；见下方 [PR #12](https://github.com/jeff-060111/ops-lab1-14/pull/12) 的 10 次对照运行 |
| O3 迁移到小组项目       | 已迁移至 [PaperMate](https://github.com/li-yuelang/SE26Project-05)。[CI/门禁 PR #93](https://github.com/li-yuelang/SE26Project-05/pull/93)、[打包发布 PR #94](https://github.com/li-yuelang/SE26Project-05/pull/94) 均已合入。[main CI 与制品](https://github.com/li-yuelang/SE26Project-05/actions/runs/39602884628)；[Release v1.0.0](https://github.com/li-yuelang/SE26Project-05/releases/tag/v1.0.0)；[release gate 拒绝](https://github.com/li-yuelang/SE26Project-05/actions/runs/39603204895)。实际下载附件后已校验哈希、追溯到 PR/提交/构建，并从 wheel 独立安装验证 `/version`。[完整报告、原始 SHA-256、trace 与接口输出及迁移差异](https://github.com/li-yuelang/SE26Project-05/blob/b0a20fd130615f84d96f2354315b9af482affa49/docs/O3.md)。迁移使用根目录 Python/FastAPI 后端与 React/Vite 前端，依赖锁定；非 integration 测试不覆盖真实数据库和外部服务。 |

### O1：容器镜像

[PR #11](https://github.com/jeff-060111/ops-lab1-14/pull/11) 在 `release.yml` 中增加 `container` job，并设置 `needs: release`。发布成功后，该 job 下载 v1.0.2 的 tar.gz 与 `.sha256` 附件，执行 `sha256sum -c`，解包后以仓库的 `Dockerfile` 构建、推送 `ghcr.io/jeff-060111/ops-lab1-14:v1.0.2`。`Dockerfile` 直接复制附件中的 `enroll-conflict` 和 `BUILD_INFO`，镜像内没有重新编译。构建指定 `--platform linux/amd64`；[发布运行 39591801985](https://github.com/jeff-060111/ops-lab1-14/actions/runs/39591801985) 的两个 job 均成功，容器 job 的日志打印了以下完整镜像引用：

```text
ghcr.io/jeff-060111/ops-lab1-14@sha256:593ecc29474ca4f3a34c055d235118f647dfda7417ba7ea34bd5282cc75aec50
```

组员确认在 `ops-lab1-main`（主机架构 `x86_64`）按上述 **digest** 运行容器，`/version` 返回版本 `1.0.2`、构建号 `4`。镜像配置中的平台为 `linux/amd64`。在 arm64 主机上需指定 `--platform linux/amd64`；能否运行取决于 Docker 的跨架构模拟支持，运行速度通常较慢。本实验没有构建多架构镜像。

**为什么不另用 Release 事件触发镜像工作流？** `release` job 使用自动提供的 `GITHUB_TOKEN` 创建 Release。GitHub 会抑制该令牌产生的大多数后续工作流触发事件，`release` 发布事件也不会启动另一个工作流，以避免递归运行。因此本组将 `container` 放在同一 `release.yml` 中，通过 `needs: release` 保证发布门禁、测试及附件创建成功后再构建镜像；发布失败时镜像 job 不执行。参见 [GitHub 的 GITHUB_TOKEN 说明](https://docs.github.com/en/actions/concepts/security/github_token)。

**GHCR 能否匿名拉取？** [包页面](https://github.com/jeff-060111/ops-lab1-14/pkgs/container/ops-lab1-14) 当前显示 **Public**。本次另用空 `DOCKER_CONFIG` 执行 `docker manifest inspect`，并用无账号凭证取得的 GHCR 匿名读取令牌下载该 digest 的镜像配置及全部两层内容；三者的 SHA256 均与清单吻合，说明镜像内容可匿名获取。由于本机 Docker daemon 未启动，此处未取得无凭证 `docker pull` 的原始终端输出。GHCR 新包默认私有；仓库关联只会默认继承访问权限，不保证继承公开可见性，须以包的可见性和实际匿名访问为准。参见 [GitHub 的容器注册表说明](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry)及[包权限说明](https://docs.github.com/en/packages/learn-github-packages/configuring-a-packages-access-control-and-visibility)。使用 digest 可固定镜像内容，避免 tag 更新后指向另一份镜像。

### O2：缓存对反馈时间的影响

**预期与设置。** 示例的 `go.mod` 没有第三方依赖，因此预计缓存几乎不节省下载时间；Go 编译缓存可能使 `vet`、`test` 快几秒到十余秒。原 `ci.yml` 虽未显式关闭 `setup-go` 缓存，但仓库没有 `go.sum`，[原主干运行日志](https://github.com/jeff-060111/ops-lab1-14/actions/runs/39582054130/job/119161766937) 报告找不到依赖文件、未生成缓存键。为得到有效对照，在不合入的草稿 [PR #12](https://github.com/jeff-060111/ops-lab1-14/pull/12) 中先设置 `cache: true`、`cache-dependency-path: go.mod`，并打印 `cache-hit`；首次运行用于写入缓存，此后用 **Re-run all jobs** 重跑 5 次。第二个提交仅把 `cache` 改为 `false`，保持其他设置相同，再运行 5 次（首次运行加 4 次重跑）。两组均使用 `ubuntu-24.04`、Go `1.26.9`，只执行 PR 的 `ci` job。下表的耗时是 GitHub API 中该 job 的 `completed_at - started_at`，单位为秒，不包含排队时间。

| 组别 | Actions 运行及 attempt | `cache-hit` | `ci` job 耗时 |
| --- | --- | --- | ---: |
| 开启缓存 | [39597803962 / 2](https://github.com/jeff-060111/ops-lab1-14/actions/runs/39597803962/attempts/2) | `true` | 17 s |
| 开启缓存 | [39597803962 / 3](https://github.com/jeff-060111/ops-lab1-14/actions/runs/39597803962/attempts/3) | `true` | 11 s |
| 开启缓存 | [39597803962 / 4](https://github.com/jeff-060111/ops-lab1-14/actions/runs/39597803962/attempts/4) | `true` | 17 s |
| 开启缓存 | [39597803962 / 5](https://github.com/jeff-060111/ops-lab1-14/actions/runs/39597803962/attempts/5) | `true` | 17 s |
| 开启缓存 | [39597803962 / 6](https://github.com/jeff-060111/ops-lab1-14/actions/runs/39597803962/attempts/6) | `true` | 13 s |
| 关闭缓存 | [39598265198 / 1](https://github.com/jeff-060111/ops-lab1-14/actions/runs/39598265198/attempts/1) | 空（未启用） | 25 s |
| 关闭缓存 | [39598265198 / 2](https://github.com/jeff-060111/ops-lab1-14/actions/runs/39598265198/attempts/2) | 空（未启用） | 32 s |
| 关闭缓存 | [39598265198 / 3](https://github.com/jeff-060111/ops-lab1-14/actions/runs/39598265198/attempts/3) | 空（未启用） | 28 s |
| 关闭缓存 | [39598265198 / 4](https://github.com/jeff-060111/ops-lab1-14/actions/runs/39598265198/attempts/4) | 空（未启用） | 23 s |
| 关闭缓存 | [39598265198 / 5](https://github.com/jeff-060111/ops-lab1-14/actions/runs/39598265198/attempts/5) | 空（未启用） | 29 s |

开启缓存的预热运行 [39597803962 / 1](https://github.com/jeff-060111/ops-lab1-14/actions/runs/39597803962/attempts/1) 显示 `cache-hit=false`、耗时 28 s，结束时成功保存缓存，因此未计入上述 5 次命中样本。开启缓存组的 job 中位数 **17 s**（范围 11–17 s），关闭缓存组中位数 **28 s**（范围 23–32 s），本组样本的中位数相差 **11 s**。`Setup Go` 步骤两组的中位数分别为 2 s、1 s；`make vet` 分别为 6 s、18 s，`make test` 分别为 1 s、3 s。日志显示，主要收益来自复用 Go 编译缓存，而非下载第三方依赖。

**波动与局限。** 两组内部 job 耗时分别波动 6 s、9 s；共享 runner 的负载、机器性能、缓存下载与解压、进程调度、网络和 Actions 步骤时间的秒级取整都会影响单次结果。样本仅各 5 次，按先开启后关闭的顺序采集，没有随机交错，也未比较包含排队的端到端反馈时间；这个 11 s 差值仅说明本仓库、本次 PR 测得的情况，不能直接推广到其他项目或 runner。

## 8. 成员贡献

每位成员一行，只填写 GitHub 用户名，无需填写姓名；行数应与小组人数一致。

| GitHub 用户名 | 主要分工 | 发起并合入的 PR 编号 | 审批过的 PR 编号 |
| --- | --- | --- | --- |
| jeff-060111 | 版本说明与仓库文档 | [#7](https://github.com/jeff-060111/ops-lab1-14/pull/7) | [#2](https://github.com/jeff-060111/ops-lab1-14/pull/2#pullrequestreview-5596279306)、[#3](https://github.com/jeff-060111/ops-lab1-14/pull/3#pullrequestreview-5596363421) |
| stardustwang | CI、Release 与 O1 容器镜像 | [#2](https://github.com/jeff-060111/ops-lab1-14/pull/2)、[#3](https://github.com/jeff-060111/ops-lab1-14/pull/3)、[#6](https://github.com/jeff-060111/ops-lab1-14/pull/6)、[#11](https://github.com/jeff-060111/ops-lab1-14/pull/11) | [#4](https://github.com/jeff-060111/ops-lab1-14/pull/4#pullrequestreview-5596845205)、[#5](https://github.com/jeff-060111/ops-lab1-14/pull/5#pullrequestreview-5597250578)、[#7](https://github.com/jeff-060111/ops-lab1-14/pull/7#pullrequestreview-5598052861) |
| li-yuelang | 第一轮测试门禁与 O3 项目迁移 | [#4](https://github.com/jeff-060111/ops-lab1-14/pull/4)、[#13](https://github.com/jeff-060111/ops-lab1-14/pull/13) | [#7](https://github.com/jeff-060111/ops-lab1-14/pull/7#pullrequestreview-5598059969)、[#8](https://github.com/jeff-060111/ops-lab1-14/pull/8#pullrequestreview-5598572089)、[#11](https://github.com/jeff-060111/ops-lab1-14/pull/11#pullrequestreview-5598899215) |
| S1mon-Chen | 第二轮周日边界测试 | [#5](https://github.com/jeff-060111/ops-lab1-14/pull/5) | [#6](https://github.com/jeff-060111/ops-lab1-14/pull/6#pullrequestreview-5597385656)、[#11](https://github.com/jeff-060111/ops-lab1-14/pull/11#pullrequestreview-5598897772) |
| Fancyyyf | 实验报告、追溯分析与 O2 缓存实验 | [#8](https://github.com/jeff-060111/ops-lab1-14/pull/8) | [#4](https://github.com/jeff-060111/ops-lab1-14/pull/4#pullrequestreview-5596847788)、[#13](https://github.com/jeff-060111/ops-lab1-14/pull/13#pullrequestreview-5600173611) |

本表只将已合入 main 的 PR 计入“发起并合入”；文档 [#9](https://github.com/jeff-060111/ops-lab1-14/pull/9) 待审，O2 实验 [#12](https://github.com/jeff-060111/ops-lab1-14/pull/12) 按题目要求不合入。
