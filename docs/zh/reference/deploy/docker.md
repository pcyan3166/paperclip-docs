---
paperclip_version: v2026.1023.0
seo_title: 使用 Docker 部署 Paperclip
seo_description: 无需在主机上安装 Node 或 pnpm，即可运行自包含的 Paperclip 实例。涵盖快速入门镜像、手动构建和配置。
---

# Docker

如果你希望运行自包含的 Paperclip 实例，而不想在主机上安装 Node 或 pnpm，请使用 Docker。

本页介绍快速入门镜像、手动构建镜像、云镜像变体，以及容器重启后哪些数据会保留。

---

## Compose 快速入门

推荐使用基于 Compose 的快速入门方式：

```sh
docker compose -f docker/docker-compose.quickstart.yml up --build
```

在以下地址打开应用：

```txt
http://localhost:3100
```

默认设置：

- 主机端口 `3100`；
- 数据目录 `./data/docker-paperclip`。

使用环境变量覆盖默认值：

```sh
PAPERCLIP_PORT=3200 PAPERCLIP_DATA_DIR=../data/pc \
  docker compose -f docker/docker-compose.quickstart.yml up --build
```

> **注意：** `PAPERCLIP_DATA_DIR` 相对于 `docker/` 中的 Compose 文件解析，因此 `../data/pc` 会映射到仓库根目录中的 `data/pc`。

两个 Compose 文件都为容器设置了 `90s` 的 `stop_grace_period`。停止整个服务时，这段时间可供运行中的智能体完成当前工作并执行最终清理。

---

## 手动构建镜像

如果要直接运行普通容器，而不是使用 Compose，可以手动构建并启动镜像：

```sh
docker build --target production -t paperclip-local .
docker run --name paperclip \
  -p 3100:3100 \
  -e HOST=0.0.0.0 \
  -e PAPERCLIP_HOME=/paperclip \
  -v "$(pwd)/data/docker-paperclip:/paperclip" \
  paperclip-local
```

如果你希望严格控制容器生命周期，或要将 Paperclip 集成到更大的 Docker 工作流中，请使用此方式。

> **注意：** 请明确指定 `production` 阶段。Dockerfile 在 `production` 之后还声明了 `cloud` 阶段；不带 `--target` 构建时会选择文件中的最后一个阶段，因此省略该参数会在不提示的情况下得到云版本。Paperclip 发布的常规镜像也采用相同方式固定构建阶段。

---

## 云镜像变体

除常规镜像外，Paperclip 还会发布第二种镜像：**云变体**。自托管时，你几乎肯定应该使用普通标签。云变体面向托管部署，包含额外内容，自托管通常用不到。

两种镜像唯一的区别是预先构建的内容。云变体以生产镜像为基础，额外预编译了一小组沙盒服务提供商插件。托管实例通过 `PAPERCLIP_MANAGED_CONFIG` 接收 `plugins.autoInstall` 列表，并在启动时从随附目录安装这些插件；只有每个插件的 `dist/` 构建输出已存在于镜像中时，这一流程才能正常工作。默认镜像包含插件源码，但不包含构建输出，因此自动安装会跳过这些插件，并记录资源包不存在的日志。

云镜像与常规镜像使用相同的标签系列，但各标签都带有 `-cloud` 后缀：

- `sha-<short>-cloud`
- `latest-cloud`
- `<version>-cloud`

你也可以通过指定 `cloud` 阶段自行构建此变体：

```sh
docker build --target cloud -t paperclip-cloud .
```

通过 `CLOUD_BUNDLED_PLUGINS` 构建参数控制哪些插件会被编入镜像。其值是 `packages/plugins/sandbox-providers` 下以空格分隔的目录名列表，默认值为 `daytona`：

```sh
docker build --target cloud \
  --build-arg CLOUD_BUNDLED_PLUGINS="daytona" \
  -t paperclip-cloud .
```

每添加一个名称，镜像都会包含相应插件的依赖，因此列表只应包含部署实际会自动安装的插件。目录名不存在时构建会失败，不会悄悄发布缺少插件的变体。

---

## 持久化数据

所有持久化数据都位于绑定挂载目录下：

- 内嵌 PostgreSQL 数据；
- 上传的资源；
- 本地密钥；
- 智能体工作区数据。

如果移除绑定挂载，下次运行时实例会从空白状态启动。

---

## LLM 适配器支持

Docker 镜像会预先安装内置本地适配器使用的本地 CLI 工具，因此这些适配器开箱即可在容器中运行：

- `claude`，供 `claude_local` 使用；
- `codex`，供 `codex_local` 使用；
- `opencode`，供 `opencode_local` 使用；
- `gemini`，供 `gemini_local` 使用。

镜像将 `GEMINI_SANDBOX=false`，让 Gemini CLI 可以在容器中安全运行，而无需再启用自己的沙盒层。

如果希望这些适配器在容器内运行，请传入相应的 API 密钥：

```sh
docker run --name paperclip \
  -p 3100:3100 \
  -e HOST=0.0.0.0 \
  -e PAPERCLIP_HOME=/paperclip \
  -e OPENAI_API_KEY=sk-... \
  -e ANTHROPIC_API_KEY=sk-... \
  -e GEMINI_API_KEY=... \
  -v "$(pwd)/data/docker-paperclip:/paperclip" \
  paperclip-local
```

不提供这些密钥时，应用仍会运行；相应适配器的环境测试只会报告缺少前置条件。

### 远程 Runner 服务提供商资源包

标准镜像还会在 `/opt/paperclip-runner/provider-pack` 中打包远程 Runner 服务提供商资源包，并将 `PAPERCLIP_RUNNER_REMOTE_PROVIDER_PACK_PATH` 指向该路径。远程 OpenCode 和 ACPX 运行需要此资源包；提供资源包后，就不会再因 `runner_remote_provider_artifact_incompatible` 错误而失败。

如果在标准镜像上构建自己的镜像，会自动继承该资源包。如果从源码构建，则通过构建参数传入完整的 40 字符 Git SHA；只有在已知提交时才会构建资源包，本地构建如果没有此参数就会跳过资源包并记录一条日志：

```sh
docker build --target production \
  --build-arg PAPERCLIP_BUILD_COMMIT="$(git rev-parse HEAD)" \
  -t paperclip-local .
```

> **提示：** 如果你在 Docker 中测试适配器行为，请先检查绑定挂载。许多意外故障来自状态丢失，而不是容器镜像本身。
