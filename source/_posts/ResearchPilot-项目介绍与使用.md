---
title: ResearchPilot：技术研究工作台的使用与实现
date: 2026-11-19 10:00:00
updated: 2026-11-28
tags:
 - Golang
 - AI
 - 项目实践
 - RAG
 - MCP
---

**ResearchPilot** 是我开发的技术研究工作台。

输入一个技术问题后，工作台会查阅公开资料、整理要点，并生成带来源的报告。它适合用来了解开源项目、准备技术方案，或梳理官方文档中的具体问题。

在把 AI 生成的结论用到代码或技术方案之前，我希望能查到支撑结论的原始资料，并区分文档中明确写出的事实与模型的推断。资料不足时，报告也应说明哪些问题仍待确认。

这是我开发 ResearchPilot 的出发点。线上版本已开放试用。[ResearchPilot 线上试用](https://researchpilot.towerman1990.cn)

### 项目技术要点

- **Go + Hertz，React + Vite + Ant Design**。Go 后端提供任务、认证和 SSE 接口，前端用于配置研究、追踪进度和核查报告证据。
- **Eino 工作流与多 Agent 分工**。Eino 编排规划、取证、证据覆盖检查和报告生成。开启分工后，证据分析员把带来源 ID 的笔记交给撰稿员，引用审核员逐条核对结论。
- **RAG 与检索评估**。公开资料和私有文档都可以进入证据链。线上使用关键词检索，本地可对照向量检索与 RRF 混合排序，也提供可选 Milvus 后端。
- **MCP 与联网取证**。通过 Microsoft Learn MCP、公开网页和 GitHub 获取资料，可选接入 Brave Search。引用指向归档的原始资料或固定提交的源码。
- **MySQL、Redis + Asynq、断点续跑**。事务 Outbox 保障任务记录和派发意图一起入库，Worker 使用租约和检查点恢复执行，SSE 在浏览器重连后补发进度事件。
- **会话与多轮研究**。会话承载连续问题，每轮保留独立任务、进度、报告和引用。上下文携带首轮及近期报告摘录，服务端通过事务锁分配轮次，支持确认后删除会话。
- **长期研究记忆**。把用户主动保存的技术环境和关注点按任务快照传给报告模型，供不同会话中的新研究参考。后续修改只影响新任务。
- **研究使用统计**。展示每日和累计使用人数、次数，登录按账号、访客按 IP 去重；成功创建研究时计次，幂等重放不会重复计数。

下面用一个 .NET 问题演示研究流程，再介绍各部分实现。文末附有 Windows 客户端和自托管方式。[源代码与启动说明](https://github.com/towerman1990/researchpilot)

<!-- more -->

### M1 到 M5 开发阶段

| 阶段 | 主要交付 |
| --- | --- |
| **M1 · 最小闭环** | 用 Docker Compose 启动环境，完成注册登录、提交研究任务和 Web 实时进度。模型与工具使用测试桩，先跑通从提交到演示报告的链路。 |
| **M2 · 研究与恢复** | 接入实际模型及网页读取、GitHub 仓库和文档查找工具，加入证据覆盖检查、带引用的报告和部分成果。通过事务 Outbox、任务租约和检查点支持重试、取消、断点续跑与 SSE 回放。 |
| **M3 · Web 上线** | 在腾讯云部署 HTTPS 工作台，验收账号、模型配置和线上研究流程。完成固定镜像发布、MySQL 备份恢复和回滚演练。 |
| **M4 · Windows 桌面端** | 比较 MyGo 与 Tauri 原型，采用 MyGo + WebView2 打包 Windows x64 客户端。桌面端与网页共享账号、任务历史和报告，并完成安装与更新验证。 |
| **M5 · 研究工作台增强** | 新增 GitHub 深度分析，跟踪固定提交的源码与 npm 依赖链。登录后可保存长期研究记忆，访客可看到登录提示。多 Agent 分工支持证据交接、报告撰写和引用复核。 |

[工程路线图](https://github.com/towerman1990/researchpilot/blob/main/docs/07-ROADMAP.md) · [M5 发布记录](https://github.com/towerman1990/researchpilot/blob/main/docs/16-M5-RELEASE.md)

此后继续完善了会话与多轮研究、删除会话、研究使用统计和任务创建请求 Log ID。下面按使用场景介绍这些功能，不另立新的开发阶段。

### 项目架构图

下面是当前项目的研究主链路。API 负责认证、研究额度、会话和任务；MySQL 保存业务状态与证据，Redis 和 Asynq 负责异步派发。Worker 通过 Eino 编排研究，按需调用模型、知识库和联网工具，执行进度写入事件表后经 SSE 提供给页面。

{% asset_img researchpilot-architecture.svg ResearchPilot 项目架构图 %}

{% asset_link researchpilot-architecture.svg 查看架构图原图 %}

API 创建任务时会把待派发记录一起写入事务，由 Outbox 派发器转交队列。Worker 把步骤结果与检查点持久化，重启后可以复用已完成的部分。图中的向量检索和 Milvus 是可选能力，当前生产环境使用关键词检索。独立 stdio MCP 服务还可向外部客户端提供公开资料的只读检索。

### 工作台如何开展研究

打开工作台就能输入研究目标。比如勾选「先查公开知识库」，问「HttpClient 如何在复用连接池的同时应对 DNS 记录变化？」任务会先找相关文档片段，再检查证据是否足以覆盖问题。完成后可在运行记录里查看检索方式和命中条数，并在中文报告中点开引用核对来源。分析开源实现时，还可以引用固定提交的源码。

如果要限定研究范围，可以先审阅和编辑计划再开始取证。也可以上传 TXT 或 Markdown，补充背景或约束。报告生成后，可逐条查看归档摘录，下载 Markdown 或另存为 PDF，也可在同一会话底部继续提问。侧边栏按会话组织历史，换一个主题时再点击「新建会话」。

多任务工作区集中展示进行中、待审批和已结束的研究。配置个人模型后，可以安排按日或按周运行的研究，并在站内订阅结果。研究评估和用量面板可查看近期任务结果、报告反馈和模型用量。

我保留了「部分完成」状态。当证据只覆盖问题的一部分时，报告会保留有依据的内容和待确认事项。也可以在提交任务时启用发布前的引用复核。

### 发起研究的流程图

从用户输入到报告发布，流程包括额度校验、事务创建任务、后台派发、上下文准备、证据采集和引用复核。审批计划、知识库检索、多 Agent 分工与引用复核按研究选项启用；证据不足时，在重规划次数和预算允许的范围内补充取证。

{% asset_img researchpilot-research-flow.svg 发起研究到生成报告的流程图 %}

{% asset_link researchpilot-research-flow.svg 查看研究流程图原图 %}

证据仍有缺口时，已有材料可以支撑的部分结论会被单独归纳并复核，报告同时说明限制；无法形成结论时保留归档摘录。没有可用资料或遇到不可恢复错误时，任务会结束为失败。运行中可以取消，任务恢复时会复用已提交的检查点。同一会话的下一轮问题沿用这条执行链，每轮保持独立的任务、证据和预算。

### 一个会话承载多轮研究

以前每次提交问题都会形成一条独立研究记录。现在增加了会话层，把围绕同一主题的多轮任务放在一起。例如先问「比较两种 Go 服务部署方案」，拿到报告后，再问「如果服务器只有 2 GiB 内存，该怎样调整？」页面会依次展示两轮问题与报告，可以加载更早的轮次。刷新页面也能恢复当前研究，旧的独立任务和追问链已迁移到对应会话。

会话负责串联主题，每一轮仍有自己的任务 ID、进度、检查点、证据和报告。当前轮结束后才能发送下一个问题；某轮失败或取消后，也能留在原会话换个角度继续。每轮按一次新研究使用试用额度和 Token 预算，任务重试和恢复不会再扣一次额度。

上下文只选取首个问题及近期问题、报告摘录，控制随轮次增长的输入长度。旧报告作为理解追问的背景，新结论仍须引用当前轮实际归档的证据。这和「长期研究记忆」分工不同：会话延续当前主题，长期记忆保存用户主动填写的技术环境和约束，供不同会话参考。

创建新轮次时，服务端在 MySQL 事务内锁定会话、判断上一轮状态，再分配轮次。同一个请求的重放依靠幂等键返回原任务，避免网络重试生成重复轮次。浏览器仍通过原有 SSE 任务事件查看进度，每轮的报告可以单独导出。

会话旁有删除按钮，确认后会从历史列表移除；删除当前会话会回到新建页面，并取消尚未完成的研究。删除与追加问题共用事务锁，避免同时操作后留下新的运行中轮次。当前采用软删除，数据库保留报告，同身份的原有任务、报告直链接口仍可访问；目前没有恢复或彻底清除入口。长期记忆、已消耗额度和累计使用统计不会随删除回退。

### 用 RAG 把检索证据写进报告

我把 .NET、ASP.NET Core 和 Go 的公开文档整理成可复用的资料摘要。研究任务命中的片段会归档到该任务，引用保留资料库版本和官方链接。Worker 将相关片段放进模型上下文，要求每条结论引用已有的来源 ID。遇到证据缺口时，流程会继续取证，或发布保留可用摘录的部分成果。

线上公开知识库目前走关键词检索。英文按词匹配，中文使用相邻双字，API 名和技术术语能直接命中。自托管时配置嵌入服务后，Worker 可以用余弦相似度做向量检索，再通过 RRF 合并两路排名。使用真实多语言模型跑过本地评估；当前线上未启用向量检索与 Milvus。 [知识库实现与评估](https://github.com/towerman1990/researchpilot/blob/main/docs/15-KNOWLEDGE-MCP.md)

登录后还可以建私有知识库并上传 Markdown 或 TXT，由 Worker 在后台切分和索引。报告保留命中的文档版本及摘录，更新文档后仍能回查旧报告的依据。「研究评估」支持对照关键词、纯向量和混合检索的召回率、延迟及无答案误命中，并可导入自己的评估 JSON。

### 用 MCP 和联网搜索补齐资料缺口

知识库覆盖不足时，Worker 可以抓取公开网页、读取 GitHub 仓库与固定提交的源码，或调用 Microsoft Learn MCP 查官方文档。MCP 客户端用 `tools/list` 发现工具，只调用白名单里的只读搜索，并把预设主题映射成公开查询词。线上已验证 Microsoft Learn 返回的资料能进入报告引用。项目也提供可在本地运行的 MCP 服务，外部客户端可通过 `stdio` 调用 `search_knowledge` 和 `get_evidence`，访问同一套公开资料。

Brave Search API 是可选的通用网页搜索入口。搜索结果先用来定位候选网页，系统再读取并归档实际页面作为引用依据。没有配置 Brave 时，工作台仍可根据模型规划、已知官方入口和用户提供的链接联网取证。真实 Brave API 的对照验收仍待完成。 [工具与研究流程](https://github.com/towerman1990/researchpilot/blob/main/docs/03-AGENT-RUNTIME.md)

### 断点续跑与进度回放

提交研究时，API 在同一个 MySQL 事务中写入任务和 Outbox 记录。任务经 Redis 和 Asynq 派发，Worker 领取后用租约管理执行权，完成阶段会把结果保存为检查点。演练时，我在采集结果落库后终止第一个 Worker，接替进程复用了该结果，最终只发布一份报告。浏览器断开连接后可经 SSE 补读仍在保留期内的事件，运行记录也会标明复用的检查点。[恢复演练步骤](https://github.com/towerman1990/researchpilot/blob/main/docs/15-KNOWLEDGE-MCP.md#断点续跑演示)

### M5 研究工作台增强

**GitHub 仓库深度分析**。打开源码结论的引用时，可以回到报告所依据的提交。目录树、源码和清单固定到同一个 SHA。Go 和 JavaScript 清单可用于解释模块和依赖约束，npm 锁文件可进一步核对锁定版本、部分传递依赖和工作区链接。例如，报告可以说明哪个直接依赖引入了某个间接依赖，以及它对应锁文件中的哪个路径。

**长期研究记忆**。如果经常研究同一套技术环境，可以在「研究偏好」保存版本、环境或约束。新研究、追问和定时任务创建时会保留快照，生成报告时参考。清空记忆后，新任务就不会携带这段背景，已创建任务仍使用提交时的快照。

**多 Agent 分工**。在新建研究时勾选该选项。规划和采集之后，证据分析员对归档资料整理交接笔记，撰稿员结合原始摘录形成报告，引用审核员核对每条结论。依据不足的结论会被过滤，各角色共享该任务的模型配置和 Token 预算。线上验证任务走完了分析、撰写和引用复核，最终报告包含 3 条归档引用。[M5 发布记录](https://github.com/towerman1990/researchpilot/blob/main/docs/16-M5-RELEASE.md)

### 线上试用与模型配置

未登录访客每天可提交 5 次研究，按北京时间零点重置。第 6 次提交会弹出登录或注册窗口，当前草稿会保留。登录后，平台模型为每个账号提供累计 10 次试用额度，额度每天不会重置。任务创建成功时扣减一次，重试和恢复不会重复扣减，失败或取消也不返还。用完后，可在「模型设置」填写个人模型的 Base URL、模型名称和 API Key，继续提交研究。[打开线上工作台](https://researchpilot.towerman1990.cn)

明确写出技术名称、版本和要比较的问题，通常比只写几个关键词更容易获得切题的报告。如果手边已有官方资料，也可以附上链接限定取证范围。

### 研究使用人数与次数

工作台底部用小字显示「今日人数、今日次数、累计人数、累计次数」。计数发生在成功提交研究任务时，追问也算一次；刷新页面、重复发送同一个幂等请求，以及任务重试或恢复都不会额外计次。

人数按登录账号或访客 IP 去重，每日数据按北京时间划分，累计数据从统计功能启用时开始记录，可见页面约每分钟刷新。这里统计的是研究服务使用情况，共用出口 IP 的访客会合并计为一人。

### Windows 桌面客户端

{% asset_img researchpilot-icon.png ResearchPilot 罗盘图标 %}

Windows x64 客户端使用 MyGo 和 WebView2，默认连接线上工作台。网页版与桌面端使用同一个账号，任务历史和报告可在两端查看。

`v0.1.2` 已作为未签名的预发行版公开提供。发布页附有安装包和 SHA-256 校验文件。[下载 Windows 安装包](https://github.com/towerman1990/researchpilot/releases/tag/v0.1.2)

### 使用 Docker Compose 本地部署

项目通过 Docker Compose 启动本地环境。请先安装 Git，再按操作系统准备 Docker 和初始化工具。

- Windows 使用 PowerShell 7 和 Docker Desktop。启动 Docker 后切换到 Linux 容器。
- macOS 按 Intel 或 Apple Silicon 芯片选择 Docker Desktop，并准备 OpenSSL，用于生成本地密钥。
- Linux 安装并启动 Docker Engine，安装 Compose v2 插件，并准备 OpenSSL。

可以先在终端执行 `docker version`、`docker compose version` 和 `docker info`，确认 Docker 客户端、服务和 Compose 都正常。

**Windows**

```powershell
git clone https://github.com/towerman1990/researchpilot.git
cd researchpilot
pwsh -File ./scripts/init-local.ps1
docker compose up --build -d
docker compose ps -a
```

**macOS 和 Linux**

```sh
git clone https://github.com/towerman1990/researchpilot.git
cd researchpilot
sh scripts/init-local.sh
docker compose up --build -d
docker compose ps -a
curl -fsS http://localhost:8080/healthz
```

健康检查正常时返回 `{"ok":true}`。如果 Linux 需要通过 `sudo docker` 访问 Docker，请仅对 Docker 命令使用 `sudo`，初始化脚本仍由项目文件所有者执行。

Apple Silicon 和 Linux ARM64 可以尝试本机架构构建，目前这两类设备的实机验收仍待完成，生产镜像仅提供 `linux/amd64`。反馈构建问题时，请附上芯片架构和错误信息。

首次启动需要下载镜像和构建依赖。Go 后端与前端均在镜像中构建，主机只需准备上面列出的工具。

`migrate` 是一次性数据库迁移服务，退出码为 0 表示迁移成功。API 和 Worker 在迁移完成后启动。

迁移完成后，访问 [本地工作台](http://localhost:8080)，注册并登录。修改访问地址或端口时，请同步调整 `PUBLIC_ORIGIN`。

#### 配置研究模型

进入「模型设置」，填写 Base URL、模型名称和 API Key。

Base URL 使用供应商提供的兼容接口基础地址，例如 `https://api.deepseek.com`。是否需要带 `/v1` 取决于供应商，程序会在基础地址后追加 `/chat/completions`。模型名称使用当前账号可用的标识。

保存后，点击「检查连接与能力」。各供应商实现的兼容能力有所差异，连接检查通过后，请再提交一个实际任务验证。

本地环境未配置个人模型且未启用平台试用时，任务会进入演示流程。演示用于验证页面、进度和报告的基本链路。平台试用为可选功能，配置并启用后才会显示额度。

#### 提交第一个研究任务

可将以下示例填入「研究目标」。

```text
请阅读下面的 Go 官方资料，整理 net/http 客户端的超时与连接复用注意事项。
区分整体请求超时、连接池空闲连接管理，以及调用方需要关闭响应体的原因。
每项结论附上对应来源；材料没有说明的部分请明确标注，不要补写。
https://pkg.go.dev/net/http
```

示例明确了研究问题、资料范围和输出要求，便于评估报告是否涵盖目标。报告质量仍取决于来源的可访问性和所使用的模型。

工作台可以读取公网页面与 GitHub 资料。Brave Search API 是可选的网页搜索服务，用于发现候选网页；进入报告的引用来自实际读取的资料。即使没有配置搜索服务，也可以直接描述问题或附上公开 URL。

要启用 Brave 搜索，在仓库根目录的本地 `.env` 中填写 `BRAVE_SEARCH_API_KEY`，再重新创建 API 和 Worker。

```powershell
docker compose up -d --force-recreate api worker
```

未认证的 GitHub API 请求可能达到配额上限。需要提高配额时，可在同一个本地配置中填写最小权限的 `GITHUB_READ_TOKEN`，然后重新创建 API 和 Worker。密钥应保存在本地配置中。

#### 核对报告引用

页面会显示等待派发、执行中、等待重试等状态，并提供取消操作。报告生成后，可以打开重要结论的引用，核对原文是否支持结论、适用条件是否完整，以及哪些内容属于推断。

系统会检查引用是否来自当前任务已归档的材料。引用是否足以支撑具体结论，仍需结合原文判断。

### 后台架构与任务执行

后端使用 Go 和 Hertz，前端使用 React、Vite 和 Ant Design。会话、各轮任务、事件、检查点、报告和来源保存在 MySQL 中，仓储层使用 GORM Gen。会话和轮次由租户标识与外键关联，读取及删除都按当前账号或访客身份隔离。Redis 和 Asynq 负责异步派发，页面通过 SSE 接收任务进度。

Worker 使用 Eino 编排研究流程。独立的取证步骤可以并行执行；覆盖检查发现缺口后，Worker 会重新规划取证，再根据证据决定发布完整报告或部分成果。

```text
提交目标 → 上传资料或可选知识库检索 → 检查证据覆盖
证据不足 → 规划并联网取证 → 归档证据 → 再检查
证据充足 → 整理报告和引用
仍有缺口 → 保留可用摘录，标注待核实事项
```

任务执行设计主要处理派发失败、重复投递和中途恢复。

如果任务已经写入数据库，但队列消息发送失败，任务可能一直等待派发。为此，系统使用 Outbox，在同一个 MySQL 事务中写入任务和待派发记录，再由后台转交给 Redis。任务状态以 MySQL 为准，队列负责将工作交给 Worker。

#### Worker 断点续跑与浏览器断线回放

Worker 中途退出后，任务可以在有效期内重新派发。已提交的研究计划、取证结果等按步骤保存为 MySQL 检查点，新 Worker 会复用这些结果，从尚未完成的步骤继续。证据与对应检查点在同一事务中写入。遇到重复投递或旧 Worker 超时后仍在运行时，租约和执行代次用来约束任务写入。

浏览器断网是另一条恢复链路。后端将任务事件持久化，浏览器通过 SSE 接收进度，重连时可用 `Last-Event-ID` 回放遗漏事件。页面刷新也能重新读取任务快照和已完成的报告。模型阶段处理结构化 JSON，事件流里显示的是已记录的任务进度。

排查某一轮研究时，可在任务详情复制「创建请求 Log ID」，关联该轮的 API 创建请求、Worker 和执行步骤；会话中的多轮研究仍保留各自的排查入口。

### 真实模型评估

早期固定十题评估中，有两道 MySQL 题因原网页无法读取而取证失败。后来，抓取工具在原站返回 403 时会尝试同版本的 Oracle 官方手册，并将实际读取的页面写入引用。开源实现题也可从目录和清单继续查到固定提交里的具体文件。

我在隔离环境中复跑了十道固定技术题。题面没有提供 URL，也没有启用 Brave，工作台通过模型规划定位资料并读取原文。十题都生成了完整报告，通过归档引用完整性检查。这个数字只反映该轮固定题集的自动结果；引用是否足以支持关键结论，仍值得查阅报告摘录。[评估方法与记录](https://github.com/towerman1990/researchpilot/blob/main/docs/11-M2-EVALUATION.md)

检索评估单独使用 11 道固定题和 5 个资料片段。在真实多语言嵌入模型的本地实验里，关键词、纯向量、混合检索的 Recall@5 分别为 100%、94.4%、100%；两道无答案题都没有返回候选。这组结果适合做回归检查，当前样本里混合检索与关键词持平。 [检索题集与复现方式](https://github.com/towerman1990/researchpilot/blob/main/docs/15-KNOWLEDGE-MCP.md)

### 常见问题与排查

遇到启动或执行问题时，可以先查看服务状态和日志。

```powershell
docker compose ps -a
docker compose logs --tail=100 migrate api worker
```

页面无法访问时，检查 Docker、8080 端口和 API 状态。任务持续等待时，检查 Worker、Redis 和迁移状态。模型认证失败时，核对 Base URL、模型名称和密钥。读取来源失败时，检查页面是否公开可访问，以及 GitHub API 配额是否耗尽。

域名解析被拒绝时，可检查服务端 DNS 和代理的 Fake-IP 设置，并保留对私网及保留地址的校验。

仓库提供演示流程的冒烟测试脚本。

```powershell
pwsh -File ./scripts/smoke.ps1
```

脚本使用 PowerShell 7，会创建测试账号和任务，检查幂等、租户隔离、SSE 回放和演示报告。macOS 和 Linux 用户仅在运行该脚本时需要 PowerShell。真实模型的效果需要单独评估。

macOS 或 Linux 上如果出现密钥读取错误，请检查 Docker 目录共享、文件权限和 SELinux 等系统策略。应用容器以 UID 65532 运行，需要读取挂载的密钥。`init-local.sh` 会保留已有密钥及其权限，排查时应避免重新生成密钥并覆盖原文件。

备份 MySQL 时，应一并妥善保管初始化生成的 `.cache/model-master.key`。恢复加密的模型配置需要原来的主密钥。

停止本地服务可运行以下命令。

```powershell
docker compose down
```

保留现有数据时，请省略会删除 Compose 数据卷的 `-v` 参数。升级前请先备份。

如果报告遗漏了研究要求，欢迎提交 Issue，并附上研究目标、公开来源和报告缺失的内容，便于复现。

### 项目链接

- [Windows x64 预发行版](https://github.com/towerman1990/researchpilot/releases/tag/v0.1.2)
- [ResearchPilot 线上试用](https://researchpilot.towerman1990.cn)
- [ResearchPilot 源码与启动说明](https://github.com/towerman1990/researchpilot)
- [实现记录](https://github.com/towerman1990/researchpilot/blob/main/docs/09-M2-IMPLEMENTATION.md)
- [RAG、MCP 与检索实验](https://github.com/towerman1990/researchpilot/blob/main/docs/15-KNOWLEDGE-MCP.md)
- [真实模型评估记录](https://github.com/towerman1990/researchpilot/blob/main/docs/11-M2-EVALUATION.md)
- [macOS / Linux 使用指南](https://github.com/towerman1990/researchpilot/blob/main/docs/14-MACOS-LINUX.md)
- [Web 发布与部署说明](https://github.com/towerman1990/researchpilot/blob/main/docs/12-M3-RELEASE.md)
