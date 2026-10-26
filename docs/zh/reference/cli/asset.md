---
paperclip_version: v2026.609.0
seo_title: CLI 资源命令
seo_description: 从终端向公司上传图像资源，例如徽标、文档图片或之后需要引用的二进制文件。
---

# 资源命令

需要从终端向公司上传图像资源时，可使用这些命令，例如上传公司徽标、文档图片或之后需要取回的任意二进制文件。CLI 会以 `multipart/form-data` 格式将文件上传到 Paperclip API，并输出保存的资源记录；`asset content` 则执行相反操作，可按 ID 将资源字节流传回。编写公司品牌相关脚本或在不打开界面的情况下初始化内容时，可以使用这些命令。

三个子命令都接受[通用客户端选项](./common-options.md)（`--data-dir`、`--api-base`、`--api-key`、`--context`、`--profile`、`--json`）。两个上传命令都按公司限定范围，接受 `-C, --company-id <id>`；`asset content` 使用全局资源 ID，不限定公司。

---

## 上传图片

```sh
paperclipai asset image:upload \
  --company-id <company-id> \
  --file ./diagram.png \
  --namespace docs \
  --alt "Architecture diagram" \
  --title "System overview"
```

`image:upload` 会将文件发送到 `/api/companies/<company-id>/assets/images`，并返回保存的资源记录（其中的资源 ID 之后可传给 `asset content`）。

| 标志 | 用途 |
|---|---|
| `--file <path>` | **必填。** 磁盘上的图片文件路径。CLI 会读取文件字节、根据文件扩展名推断内容类型，并发送原始文件名。 |
| `-C, --company-id <id>` | 资源所属公司。除非可从所选上下文/配置文件中解析，否则必填。 |
| `--namespace <value>` | 可选命名空间后缀，用于将资源归入逻辑分组（例如 `docs`）。 |
| `--alt <text>` | 随资源保存的可选替代文本元数据。 |
| `--title <text>` | 随资源保存的可选标题元数据。 |

> **提示：** 内容类型根据文件路径推断，因此上传文件时请确保扩展名正确（例如 `.png`、`.svg`、`.jpg`）。文件名扩展名错误会导致资源以错误的类型保存。

---

## 上传徽标

```sh
paperclipai asset logo:upload \
  --company-id <company-id> \
  --file ./logo.svg
```

`logo:upload` 会将文件发送到 `/api/companies/<company-id>/logo`，并将其设置为公司的徽标。它与 `image:upload` 一样使用 multipart 上传，但**不接受** `--namespace`、`--alt` 或 `--title`——公司只有一个固定徽标位置，因此不附带这类元数据。

| 标志 | 用途 |
|---|---|
| `--file <path>` | **必填。** 徽标文件路径。SVG 和常见栅格图像格式均可；内容类型根据扩展名推断。 |
| `-C, --company-id <id>` | 要设置徽标的公司。除非可从上下文/配置文件中解析，否则必填。 |

> **注意：** 此操作会替换公司当前的徽标。此处没有单独的“移除徽标”子命令；上传新文件即可更换。

---

## 下载资源内容

```sh
# Write the bytes to a file
paperclipai asset content <asset-id> --out ./asset.bin

# Or stream the bytes to stdout (pipe or redirect them)
paperclipai asset content <asset-id> > ./asset.bin
```

`asset content <assetId>` 会从 `/api/assets/<asset-id>/content` 获取原始字节。默认情况下，字节会直接写入标准输出，因此可以通过管道传递或重定向。传入 `--out <path>` 后，CLI 会改为替你写入文件；此模式下会输出简短确认对象（`{ ok, out, bytes }`），而不是输出二进制内容。

| 标志 | 用途 |
|---|---|
| `--out <path>` | 将下载内容写入指定文件，而不是标准输出。设置后，命令会输出包含字节数的确认摘要。 |

> **警告：** 未指定 `--out` 时，命令会将原始二进制写入标准输出。请始终重定向或通过管道传递（`> file` 或 `| tool`）；直接将二进制打印到终端会破坏终端显示。`--json` 在标准输出流模式下无效；只有使用 `--out` 时，它才会影响输出的确认对象格式。

---

## 编写脚本

两个上传命令都支持 `--json`，因此可以捕获返回的资源 ID，并将其传给下载命令或其他命令：

```sh
ASSET_ID=$(paperclipai asset image:upload \
  --company-id <company-id> \
  --file ./diagram.png \
  --json | jq -r '.id')

paperclipai asset content "$ASSET_ID" --out ./roundtrip.png
```

API 地址按以下标准顺序解析：`--api-base`、`PAPERCLIP_API_URL`、所选上下文配置文件、本地配置端口，最后是 `http://localhost:3100`。连接失败时，错误信息会列出尝试连接的 URL，以便你确认目标实例。完整约定请参阅[通用选项](./common-options.md)和[输出和脚本](./output-and-scripting.md)。

---

## 另请参阅

- [通用选项](./common-options.md)——所有客户端命令共享的标志，包括 `--company-id` 和 API 地址解析。
- [输出和脚本](./output-and-scripting.md)——`--json` 如何影响结果格式，以及如何使用输出内容。
- [技能命令](./skills.md)——管理公司的技能库；另一种按公司限定范围处理文件的工作流。
- [适配器命令](./adapter.md)——配置执行智能体工作的运行时适配器。
- [公司命令](./company.md)——检查和导出资源所属公司。
