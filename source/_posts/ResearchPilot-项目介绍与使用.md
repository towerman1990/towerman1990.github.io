---
title: ResearchPilot：技术研究工作台的使用与实现
date: 2026-11-19 10:00:00
updated: 2026-11-21 10:00:00
tags:
 - Golang
 - AI
 - 项目实践
---

**ResearchPilot** 是我开发的技术研究工作台。

输入一个技术问题后，工作台会查阅公开资料、整理要点，并生成带来源的报告。它适合用来了解开源项目、准备技术方案，或梳理官方文档中的具体问题。

在把 AI 生成的结论用到代码或技术方案之前，我希望能查到支撑结论的原始资料，并区分文档中明确写出的事实与模型的推断。资料不足时，报告也应说明哪些问题仍待确认。

这是我开发 ResearchPilot 的出发点。线上版本已开放试用。[ResearchPilot 线上试用](https://researchpilot.towerman1990.cn)

下文介绍线上试用、Windows 客户端和自托管方式，也记录了证据采集与最新一轮评估的结果。[源代码与启动说明](https://github.com/towerman1990/researchpilot)

<!-- more -->

### 工作台如何开展研究

注册并登录后，输入一个具体的研究目标。工作台会规划步骤，读取公开网页或 GitHub 资料，归档证据，再生成带引用的报告。分析开源实现时，可以引用固定提交的源码，方便日后核对依据。

如果要限定研究范围，可以先审阅和编辑计划再开始取证。也可以上传 TXT 或 Markdown，补充背景或约束。报告生成后，可逐条查看归档摘录，下载 Markdown 或另存为 PDF，也可基于这份报告继续追问。

多任务工作区集中展示进行中、待审批和已结束的研究。配置个人模型后，可以安排按日或按周运行的研究，并在站内订阅结果。研究评估和用量面板可查看近期任务结果、报告反馈和模型用量。

我保留了「部分完成」状态。当证据只覆盖问题的一部分时，报告会保留有依据的内容和待确认事项。也可以在提交任务时启用发布前的引用复核。

### 线上试用与模型配置

使用平台模型时，每个账号有累计 10 次试用额度，每天不会重置。任务创建成功时扣减一次，重试和恢复不会重复扣减，失败或取消也不返还。用完后，可在「模型设置」填写个人模型的 Base URL、模型名称和 API Key，继续提交研究。[打开线上工作台](https://researchpilot.towerman1990.cn)

明确写出技术名称、版本和要比较的问题，通常比只写几个关键词更容易获得切题的报告。如果手边已有官方资料，也可以附上链接限定取证范围。

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

后端使用 Go 和 Hertz，前端使用 React、Vite 和 Ant Design。任务、事件、检查点、报告和来源保存在 MySQL 中，仓储层使用 GORM Gen。Redis 和 Asynq 负责异步派发，页面通过 SSE 接收任务进度。

Worker 使用 Eino 编排研究流程。独立的取证步骤可以并行执行。覆盖检查发现缺口后，工作台会尝试补充资料，再决定是否生成完整报告。

```text
提交目标 → 规划步骤 → 读取资料 → 保存证据
                              ↓
                         检查研究目标覆盖情况
                              ↓
                       整理报告和引用
```

任务执行设计主要处理派发失败、重复投递和中途恢复。

如果任务已经写入数据库，但队列消息发送失败，任务可能一直等待派发。为此，系统使用 Outbox，在同一个 MySQL 事务中写入任务和待派发记录，再由后台转交给 Redis。任务状态以 MySQL 为准，队列负责将工作交给 Worker。

消息可能重复投递，旧执行器也可能在超时后继续运行。Worker 通过租约和执行代次约束写入，并保存检查点供恢复使用。

前端通过 SSE 接收已持久化的任务事件，断线后可按游标回放。事件流展示任务进度，研究主链路主要处理模型返回的完整 JSON 响应。

### 真实模型评估

早期固定十题评估中，有两道 MySQL 题因原网页无法读取而取证失败。后来，抓取工具在原站返回 403 时会尝试同版本的 Oracle 官方手册，并将实际读取的页面写入引用。开源实现题也可从目录和清单继续查到固定提交里的具体文件。

最近一轮在隔离环境复跑了十道固定技术题。题面没有提供 URL，也没有启用 Brave，工作台通过模型规划定位资料并读取原文。十题都生成了完整报告，通过归档引用完整性检查。这个数字只反映该轮固定题集的自动结果；引用是否足以支持关键结论，仍值得查阅报告摘录。真实 Brave 调用仍待验收。[评估方法与记录](https://github.com/towerman1990/researchpilot/blob/main/docs/11-M2-EVALUATION.md)

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
- [真实模型评估记录](https://github.com/towerman1990/researchpilot/blob/main/docs/11-M2-EVALUATION.md)
- [macOS / Linux 使用指南](https://github.com/towerman1990/researchpilot/blob/main/docs/14-MACOS-LINUX.md)
- [Web 发布与部署说明](https://github.com/towerman1990/researchpilot/blob/main/docs/12-M3-RELEASE.md)
