---
paperclip_version: v2026.1023.0
seo_title: Cloudinary 连接器
seo_description: 允许智能体搜索和管理 Cloudinary 资源。涵盖资源管理连接的功能范围、基于角色的限制、与计费相关的操作以及故障排查。
---

# Cloudinary

智能体可以搜索和管理您 Cloudinary 产品环境中的媒体资源——查找图像和视频、读取元数据，并使用您的资源库。

此连接是 Cloudinary 的资源管理界面。它专注于查找和组织资源，而非 Cloudinary 所有功能的通用接口。

## 连接前准备

- 一个 Cloudinary 账户，且有权访问您希望智能体使用的产品环境。
- 登录用户的角色至关重要：Cloudinary 将连接可执行的操作限制在该用户角色允许的范围内。在连接之前，请决定使用哪个用户进行授权。

## 连接 Cloudinary

1. 打开 **连接器** 并选择 **Cloudinary**。
2. 阅读主按钮上方的访问说明行。它说明了连接以谁的身份登录以及哪些智能体可以使用它。若要选择其他身份、缩小智能体范围或使用其他登录方法，请选择 **更改**。
3. 选择 **继续前往 Cloudinary** 并完成浏览器登录。

Paperclip 会自动注册其客户端，因此无需在开发者控制台中进行任何配置。

## 选择访问权限

访问范围是登录用户的产品环境，而授权受限于该用户的 Cloudinary 角色。使用只读 Cloudinary 角色是赋予智能体搜索权限而不允许更改的最清晰方式。

在设置权限时，请区分以下三种操作类型：

| 操作 | 后果 |
| --- | --- |
| 搜索和读取资源元数据 | 不更改资源库中的任何内容 |
| 上传或修改资源 | 更改您的资源库 |
| 删除资源 | 移除实时站点可能引用的媒体 |

> **警告：** 删除资源可能会破坏嵌入该资源的每个页面，且从 Paperclip 无法恢复删除操作。请将删除功能保持为 **关闭** 状态。上传和转换会消耗存储和转换配额，这会对您的 Cloudinary 套餐产生计费影响。

> **注意：** 读取操作不会更改任何内容，但请勿假设它们是免费的。API 调用会计入您 Cloudinary 套餐自身的速率和使用限制，而这些限制的计量方式是 Cloudinary 的业务范畴，而非 Paperclip 的。如果智能体将进行大量读取操作，请检查您的套餐。

参见 [设置操作权限](action-permissions.md)。

## 试用

```txt
Search Cloudinary for assets tagged "product-hero" and tell me how many there are and the public ID of the newest one. Do not upload, transform, or delete anything.
```

与 Cloudinary 媒体库进行对比。将返回的 ID 与已知资源进行核对，并检查任务的连接器活动。搜索不会更改资源，但仍会消耗提供商请求并可能计入使用限制。

> **注意：** 此为示例任务，而非记录的测试结果。请替换为您自己库中的标签。

## 故障排查与限制

| 问题 | 可能原因 | 解决方法 |
| --- | --- | --- |
| 授权成功但可用内容很少 | 登录用户的角色受限 | 在 Cloudinary 中检查该用户的角色 |
| 出现错误的资源 | 账户有多个产品环境，且选择了另一个环境 | 使用预期的环境重新连接 |
| 操作被拒绝 | 用户角色不允许该操作 | 在 Cloudinary 中调整角色，或保持该功能关闭 |
| 实时站点上的资源消失 | 允许并执行了删除操作 | 如果已启用，请从 Cloudinary 备份中恢复，然后将删除功能设置为 **关闭** |
| 配额或账单意外增加 | 上传或转换消耗了套餐资源 | 在 Cloudinary 中审查使用情况并限制这些操作 |
| **需要关注** | 授权已被撤销 | 选择 **重新连接** |

限制：每个连接仅支持一个产品环境。基于角色的限制由 Cloudinary 设定，而非 Paperclip。从 Paperclip 无法恢复删除操作。

## 相关指南

- [连接器概述](https://paperclip.ing/product/connectors/cloudinary/)

- [Google Drive](google-drive.md)、[Box](box.md) — 通用文件存储而非媒体资源。
- [设置操作权限](action-permissions.md)
- [连接器访问机制](access-model.md)
- [Cloudinary MCP 文档](https://cloudinary.com/documentation/cloudinary_llm_mcp)
