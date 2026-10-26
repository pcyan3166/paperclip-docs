---
paperclip_version: v2026.1012.0
seo_title: 自定义沙盒镜像
seo_description: 不必在每次运行时重新安装相同的 CLI 和依赖。构建并捕获沙盒镜像，然后管理后续运行使用的镜像。
---

# 自定义沙盒镜像

智能体在沙盒环境中运行时，通常会从干净的基础镜像启动，并在每次运行时安装所需内容——CLI、语言运行环境、项目依赖、登录会话等。偶尔运行一次时这样没有问题；但如果每次都要重复相同的设置，过程就会缓慢而繁琐。

自定义沙盒镜像可以解决这个问题。你可以启动一个临时设置沙盒，在浏览器中直接打开终端，安装并登录所需工具，然后将运行中的机器捕获为可复用镜像。之后，该环境中的运行会从已准备好的镜像启动，而不是从纯净基础镜像启动；所需工具已经就位。

更方便的是，整个流程都不必离开浏览器。内嵌终端会直接连接设置沙盒，因此无需将 SSH 命令复制到外部终端，也无需在本机处理密钥。你可以完全通过环境配置页面准备并捕获镜像。

本指南介绍如何启用此功能、在哪里找到相关设置、如何构建并捕获镜像，以及如何长期管理捕获的镜像。

---

## 开始之前：启用 Environments

自定义镜像属于沙盒环境，而环境是一项**实验性**功能——启用前，整个环境 UI 都会隐藏。

1. 前往 **Settings → Experimental**。
2. 打开 **Enable Environments**。

界面中对该开关的说明是 *“Show environment management in company settings and allow project and agent environment assignment controls.”* 启用后，**Settings → Instance settings** 下会出现 **Environments** 页面，你可以在这里创建和编辑智能体运行所使用的环境。

> **注意：**自定义镜像仅适用于 **sandbox** 环境，即由沙盒提供程序插件支持的环境（参见[沙盒提供程序](../../reference/adapters/sandbox-providers.md)）。它们不适用于 `local` 或 `ssh` 环境。

---

## 自定义镜像区域的位置

自定义镜像控件只会在你**编辑已有的沙盒环境**时出现，首次创建环境时不会显示。操作流程如下：

1. 打开 **Settings → Instance settings → Environments**。
2. 创建一个沙盒环境（或打开已有环境）并保存。
3. 再次打开该环境进行编辑。
4. 滚动到 **Custom image** 区域。

该区域的说明文字是：*“Start a setup sandbox, SSH in to customize the instance, then capture the running machine as a reusable image for future runs.”* 以下所有操作都在这个面板中完成。

### 提供程序不支持时

并非所有沙盒提供程序都支持交互式设置和镜像捕获，因此面板会根据环境提供程序声明的能力显示不同内容：

- **Template setup**——提供程序同时支持交互式设置和镜像捕获，因此可以使用下文介绍的完整流程。
- **Unsupported provider**——显示 *“This provider does not advertise interactive template setup.”* 该环境没有可配置的相关选项。
- **Setup capture unavailable**——显示 *“This provider advertises setup, but image capture is unavailable.”* 你可以打开终端，但提供程序无法将运行中的机器转换为可复用镜像。

如果看到后两种状态，本指南其余内容不适用于该环境。请切换到支持捕获的提供程序，或在 Paperclip 之外准备镜像。

---

## 构建并捕获镜像

在受支持的环境中首次打开 **Custom image** 区域时，状态会显示 **Not configured**，并出现类似 *“Capture a custom Daytona image with your tools already logged in.”* 的提示（其中提供程序名称会与环境匹配）。完整流程如下。

### 1. 启动设置会话

点击 **Configure image**。Paperclip 会为你启动一个全新的设置沙盒，并打开一个**设置会话（setup session）**——这是一个有时限的会话，仅用于准备机器。你会看到状态行从 **Setup starting** 开始，沙盒准备好连接后变为 **Setup running**。

设置会话有时限（默认最长约一小时），面板会显示当前会话的过期时间。如果你离开后会话过期，状态会变为 **Setup expired**，此时只需重新启动会话。

### 2. 在浏览器终端中准备机器

会话运行后，面板中会打开内嵌的 **Browser terminal** 并连接到设置沙盒。这是机器上的真实 Shell——你可以安装软件包、克隆仓库、登录 CLI、设置缓存，或执行平时手动完成的任何操作：

- 终端标题栏会显示连接状态，以及 **Open terminal** / **Reconnect** 按钮；连接后还会显示 **Disconnect**。
- 在此安装或配置的所有内容都会包含在下一步捕获的镜像中，因此你可以在这里将机器准备到所需状态。

如果浏览器终端无法连接某个提供程序连接，面板会改为显示原始连接详情（见下方[浏览器终端不可用时](#if-the-browser-terminal-isnt-available)）。

### 3. 捕获镜像

机器准备好后，点击 **Finished**。Paperclip 会将运行中的沙盒捕获为镜像——状态先变为 **Capturing template**，完成后变为 **Template captured**。在后台，捕获的镜像会提升为环境的**活动模板（active template）**；之前处于活动状态的镜像会被取代，但仍会保留，供你回滚使用。

如果在捕获前改变主意，请点击 **Cancel**。状态会变为 **Setup cancelled**；已有的活动镜像（如果存在）不会受到影响。

> **注意：**只有在会话处于 **Setup running** 状态（等待你操作）时，才能点击 **Finished**。捕获过程中按钮会被禁用，避免你中断捕获。

---

## “已捕获”对后续运行的含义

捕获的镜像只属于构建它的特定环境——每个环境最多有一个活动自定义镜像。环境启用活动镜像后，Paperclip 会自动应用它：所有通过该环境解析的运行都会从你捕获的镜像启动，而不是从提供程序基础镜像启动；Paperclip 也会记录此次运行使用了该镜像。

根据提供程序不同，捕获的镜像可能以**快照（snapshot）**、**镜像（image）**或提供程序的**模板（template）**形式存储。面板会标明活动镜像的类型，供你识别。此类型由提供程序决定；Paperclip 会捕获提供程序生成的内容。

正在运行的智能体没有任何变化——它仍然使用原来的沙盒。区别在于，启动时机器已经具备你的工具和登录状态，比每次运行都重新安装更快，也更可预测。

---

## 管理活动镜像

捕获镜像后，**Custom image** 区域会切换为 **Active template** 视图。它会显示提供程序、镜像类型、镜像捕获时间和最近使用时间，并提供三种操作：

**Refresh（刷新）**
基于当前活动镜像启动一个新的设置会话。如果你想在现有镜像基础上添加内容（例如安装其他工具或更新登录状态），可以使用此操作。完成后再次点击 **Finished**，捕获的新镜像会取代旧镜像。流程与首次构建和捕获相同，只是从现有镜像而不是基础镜像启动。

**Rollback（回滚）**
恢复到当前镜像之前的活动镜像。后续运行会重新使用之前的镜像。如果新捕获的镜像不如被替换的版本，此操作就相当于撤销。

**Disable（停用）**
停用当前活动自定义镜像。后续运行会回退到环境的提供程序基础配置，效果与从未捕获过自定义镜像相同。

每项操作完成后都会显示简短提示，例如 *“Future runs can use the promoted template,”*、*“Future runs will use the previous template,”* 或 *“Future runs will use the base provider configuration”*，让你始终清楚下一次运行将使用什么启动。

---

<a id="if-the-browser-terminal-isnt-available"></a>
## 浏览器终端不可用时

内嵌浏览器终端是推荐方式，但某些提供程序连接无法使用它。此时面板会提供两种备用方案，避免流程中断：

- **SSH command fallback** 折叠项。展开后可以看到设置沙盒的原始 SSH 命令，你可以在外部终端中运行该命令，以传统方式准备机器。
- 一条提示：*“Browser terminal is not available for this provider connection. Use the provider setup instructions, then finish or cancel here.”* 它会提醒你使用提供程序的设置说明，并在机器准备好后返回面板点击 **Finished**（或 **Cancel**）。

无论采用哪种方式，捕获步骤仍然要在面板中完成：通过你能使用的方式准备机器，然后回到这里结束会话。

---

## 会话与镜像状态一览

面板中会反复出现两组状态术语，了解它们会更容易操作。

构建镜像时，**设置会话（setup session）**会经过以下状态：

- **Setup starting**——正在配置沙盒。
- **Setup running**——沙盒已就绪，等待你准备环境；此时可以使用终端和 **Finished**。
- **Capturing template**——你已点击 **Finished**，Paperclip 正在将机器转换为镜像。
- **Template captured**——镜像已捕获并提升为活动镜像。
- **Setup cancelled**——你在捕获前取消了操作。
- **Setup expired**——会话的时限在你完成前已结束。
- **Setup failed**——操作发生错误；面板会显示失败原因。

捕获的**镜像**始终处于以下某种状态，不过你主要会与活动镜像交互：

- **active**——后续运行将从该镜像启动。
- **superseded**——之前的镜像，保留下来供 **Rollback** 恢复。
- **revoked**——已停用的镜像。
- **failed**——未能完成捕获的镜像。

---

## 相关内容

- [沙盒提供程序](../../reference/adapters/sandbox-providers.md)——为沙盒环境提供支持的提供程序，以及各自提供的能力。
- [工作区](./workspaces.md)——了解智能体开始运行后使用的执行工作区。
