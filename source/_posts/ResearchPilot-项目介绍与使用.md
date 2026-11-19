---
title: ResearchPilot：给自己写了个技术研究工作台
date: 2026-11-19 10:00:00
tags:
 - Golang
 - AI
 - 项目实践
---

最近写了个新项目，叫 **ResearchPilot**。

名字起得有点大，做的事情倒比较具体：给它一个技术问题，让它去读资料、整理要点，最后交一份带来源的报告。写技术方案、了解一个开源项目，或者想弄清楚某份官方文档里几个容易混淆的概念时，都可以试试。

现在用 AI 查东西已经很方便了。不过，一份回答写得流畅，不代表里面的结论都有出处。尤其是准备把它用到代码或方案里时，我还是想知道：这句话是文档说的，还是模型推出来的？资料没读到，它会不会也给我写上一大段？

ResearchPilot 就是围绕这些问题做的一次尝试。现在也放了一个线上版本，不想折腾本地环境的话，可以先打开 [ResearchPilot 线上试用](https://researchpilot.towerman1990.cn) 看看。

代码在这里：[towerman1990/researchpilot](https://github.com/towerman1990/researchpilot)。下面聊聊它怎么用，以及写这个项目时处理的几个问题。

<!-- more -->

### 先说它能干什么

打开工作台，注册、登录，填一个研究目标，剩下的过程就交给后台。它会规划步骤，用只读工具读取公网页面或公开 GitHub 信息，保存用到的材料，再整理报告。页面上可以看进度，也能回头查看历史任务、报告和来源。

问题最好具体一点。比如“看一下这个仓库怎么启动”“从官方文档里整理 HTTP 客户端的超时设置”，都比“帮我研究一下 Go”更容易得到有用的结果。

研究不一定每次都能完成。网页打不开、材料不够、模型没按要求返回，都可能让任务失败或者停在「部分完成」。这也是我希望保留在页面上的信息：没查到，就把没查到的部分交代清楚。

### 先在线上试试

直接访问 [researchpilot.towerman1990.cn](https://researchpilot.towerman1990.cn)，注册并登录后就可以进入工作台。试用额度和模型配置以页面提示为准；后面介绍的研究目标写法、进度查看和报告核对，在线上也一样适用。

如果想在自己的电脑上运行，或者看看代码怎么实现，再按下面的步骤部署。

### 想自己部署的话，在本地跑起来

我把本地环境放进了 Docker Compose。Windows、macOS 和 Linux 都有对应的启动说明，主要区别在 Docker 的安装方式和初始化脚本。先装好 Git，再准备各自的环境：

- Windows：安装 PowerShell 7 和 Docker Desktop，启动 Docker，并切到 Linux 容器。
- macOS：按 Intel 或 Apple Silicon 芯片选择 Docker Desktop，安装并启动；还需要 OpenSSL，用来生成本地密钥。
- Linux：安装并启动 Docker Engine，再装 Compose v2 插件，同时准备 OpenSSL。

可以先在终端执行 `docker version`、`docker compose version` 和 `docker info`，确认 Docker 客户端、服务和 Compose 都正常。

**Windows 下这样启动：**

```powershell
git clone https://github.com/towerman1990/researchpilot.git
cd researchpilot
pwsh -File ./scripts/init-local.ps1
docker compose up --build -d
docker compose ps -a
```

**macOS 和 Linux 下则用这组命令，不用另外安装 PowerShell：**

```sh
git clone https://github.com/towerman1990/researchpilot.git
cd researchpilot
sh scripts/init-local.sh
docker compose up --build -d
docker compose ps -a
curl -fsS http://localhost:8080/healthz
```

健康检查正常时会返回 `{"ok":true}`。如果 Linux 上只有 `sudo docker` 能用，就给 Docker 命令加 `sudo`；初始化脚本仍然用项目文件所有者的身份执行，别把两件事混在一起。

Apple Silicon 和 Linux ARM64 可以先尝试本机架构构建，不过目前还没有完成这两类设备的实机验收，生产镜像发布也暂时只有 `linux/amd64`。如果卡在构建阶段，记得把芯片架构和报错一起留下，方便定位。

第一次要下载镜像和编译依赖，可能需要等一会儿。Go 和前端都在镜像里构建，宿主机不用另外装对应的工具链。

这里有个容易让人疑惑的地方：`migrate` 是负责数据库迁移的一次性服务，正常退出、退出码为 0 就可以，不用等它一直显示 Running。迁移完成后，API 和 Worker 才会启动。

然后访问 [http://localhost:8080](http://localhost:8080)，注册账号并登录。先按这个默认地址访问；如果要换地址或端口，也要一起调整 `PUBLIC_ORIGIN`。

#### 配好模型，再做真实研究

进入「模型设置」，填写 Base URL、模型名称和 API Key。

Base URL 填供应商提供的兼容接口基础地址，例如 `https://api.deepseek.com`；有些供应商需要带 `/v1`，按它的说明来。程序会自己拼接 `/chat/completions`，不用把整条请求地址填进去。模型名称则填写自己账号实际可用的标识。

保存后，点一下「检查连接与能力」，确认配置能用，再提交任务。不同供应商对兼容接口的实现有差别，连接成功以后，仍然需要用实际任务试一下。

如果没有配置个人模型，环境也没启用平台试用，任务会走演示流程。它能帮你确认页面、进度和报告这条链路跑通了，但返回的不是一次真实研究。代码里也加了可选的试用额度，只有运行环境配置并启用后才会显示；本地尝试不依赖这部分。

#### 第一个问题别问太大

可以先把下面这段填进「研究目标」：

```text
请阅读下面的 Go 官方资料，整理 net/http 客户端的超时与连接复用注意事项。
区分整体请求超时、连接池空闲连接管理，以及调用方需要关闭响应体的原因。
每项结论附上对应来源；材料没有说明的部分请明确标注，不要补写。
https://pkg.go.dev/net/http
```

这段输入把问题、资料和输出要求都写出来了。这样报告出来以后，也容易判断它有没有答到点上。当然，实际效果还取决于页面能不能读到，以及使用的模型。

目前支持公网页面、公开 GitHub 信息和可选的 Brave 搜索。没有配置搜索服务时，直接给公开 URL 会更合适。

想启用 Brave 搜索，可以在仓库根目录的本地 `.env` 里填写 `BRAVE_SEARCH_API_KEY`，再重新创建 API 和 Worker：

```powershell
docker compose up -d --force-recreate api worker
```

读 GitHub 时也可能碰到未认证 API 配额用完的问题。需要提高配额，可以在同一个本地配置里填写最小权限的 `GITHUB_READ_TOKEN`，然后执行上面的命令。真实密钥别提交到仓库。

#### 报告出来了，还是要看一眼来源

任务提交后，页面会显示等待派发、执行中、等待重试等状态，也可以取消任务。报告生成后，建议挑重要结论打开引用看一眼：原文是否真的支持它的说法？有没有漏掉适用条件？哪些是资料里的事实，哪些是推断？

系统会检查引用是否来自当前任务已保存的材料。不过，有来源和读对了来源，仍然是两回事。这一步目前还得靠人复查。

### 后台为什么拆成这样

后端用了 Go，HTTP 层先用标准库 `net/http`；前端是 React 19 和 Vite 7。任务、事件、检查点、报告和来源放在 MySQL 8.4，Redis 7.4 加 Asynq 负责异步派发，浏览器通过 SSE 看进度。

Agent 部分目前是单 Agent、自研运行时。研究过程可以简单理解成：

```text
提交目标 → 规划步骤 → 读取资料 → 保存证据
                              ↓
                   检查有没有回答目标问题
                              ↓
                       整理报告和引用
```

真正需要多处理一些的，是这条流程中间出问题怎么办。

比如，数据库已经创建了任务，往队列发消息却失败了，任务就可能一直等着。所以这里用了 Outbox：在同一个 MySQL 事务里写入任务和待派发记录，再由后台把它转交给 Redis。任务状态以 MySQL 为准，队列主要负责把工作送到 Worker 手里。

再比如，同一条消息可能被重复投递，一个执行器超时以后也未必立刻停止。Worker 因此需要租约和执行代次来约束写入，并保存检查点供恢复使用。做过异步任务的同学，对这些问题应该都不陌生。

前端用 SSE 接收持久化后的任务事件，断线后可以按游标回放。这里展示的是任务进度；研究主链路主要使用模型的完整 JSON 响应，所以页面有进度更新，不代表模型正在逐字输出答案。

### 测下来，有些失败挺值得留下来

项目里做过一轮固定十题的真实模型评估。八题生成了完整报告，两题因为没拿到足够的 MySQL 官方材料，没有通过。

这个数字只能说明那一轮的结果，不能当成通用成功率。相比报一个分数，更值得记录的是：早期有报告遗漏了问题要求里的关键内容，任务状态却已经显示完成。后来加了基于归档证据的目标覆盖检查，缺口补不齐时，就返回「部分完成」。相关过程写在[评估记录](https://github.com/towerman1990/researchpilot/blob/main/docs/11-M2-EVALUATION.md)里。

这类问题比“接口调通了没有”更难判断。模型返回了东西，程序也没报错，还得继续问一句：它交出来的结果，真的够用吗？

### 跑不起来时，可以先看这里

先检查服务和日志，通常比反复点提交更有效：

```powershell
docker compose ps -a
docker compose logs --tail=100 migrate api worker
```

页面打不开，先看看 Docker、8080 端口和 API。任务一直等待，就看 Worker、Redis 和迁移状态。模型认证失败，回头核对 Base URL、模型名称和密钥；来源读取失败，再确认页面是否公开可访问，或 GitHub 配额是否耗尽。

如果提示域名解析被拒绝，也可能和代理的 Fake-IP 有关。可以检查服务端 DNS，但不要为了跑通就关掉私网、保留地址的校验。

仓库还有一个演示烟测脚本：

```powershell
pwsh -File ./scripts/smoke.ps1
```

这个脚本需要 PowerShell 7。macOS 和 Linux 用户想跑它，可以另外安装 PowerShell；只启动和使用工作台则不需要。它会创建测试账号和任务，检查幂等、租户隔离、SSE 回放和演示报告。适合验证基本流程，真实模型的效果还要另外看。

macOS 或 Linux 上如果报密钥读取失败，先检查 Docker 的目录共享、文件权限和 SELinux 等系统策略。应用容器使用 UID 65532，需要能读取挂载进去的密钥；`init-local.sh` 会保留已有密钥，迁移旧环境时也不会替你修改旧文件的权限，别为了排错直接重新生成覆盖。

另外，初始化时生成的 `.cache/model-master.key` 要保管好。模型配置加密后存在数据库里，备份 MySQL 时也需要保留对应的主密钥，否则恢复了数据也解不开配置。

日常停止服务用下面这条就行：

```powershell
docker compose down
```

别顺手加 `-v`，那会删除 Compose 管理的数据卷。升级之前也记得先备份。

Web 版本现在已经提供线上试用，桌面端仍在推进。欢迎直接在线上体验，也可以把项目拉下来自己运行。如果碰到“看起来答了，其实没答到”的例子，也欢迎提 Issue。

项目和文档放在这里：

- [ResearchPilot 线上试用](https://researchpilot.towerman1990.cn)

- [ResearchPilot 源码与启动说明](https://github.com/towerman1990/researchpilot)
- [实现记录](https://github.com/towerman1990/researchpilot/blob/main/docs/09-M2-IMPLEMENTATION.md)
- [真实模型评估记录](https://github.com/towerman1990/researchpilot/blob/main/docs/11-M2-EVALUATION.md)
- [macOS / Linux 使用指南](https://github.com/towerman1990/researchpilot/blob/main/docs/14-MACOS-LINUX.md)
- [Web 发布与部署说明](https://github.com/towerman1990/researchpilot/blob/main/docs/12-M3-RELEASE.md)
