---
paperclip_version: v2026.1012.0
seo_title: 技能：可复用的智能体流程
seo_description: 将检查清单、操作手册或模板打包一次并供多个智能体复用。了解技能导入、文件夹、技能工作室及其成本和性能收益。
---

# 技能

如果你希望智能体遵循某个特定流程——例如代码审查检查清单、部署手册或客户回复模板——可以把说明写在智能体的主配置中。但如果多个智能体要使用同一流程，或你希望智能体配置保持精简、聚焦，就可以使用**技能（skills）**。

技能是可复用的指令文档，智能体可按需加载。不必将每种流程都写入每个智能体的配置；你只需在公司技能库中编写一次，然后提供给需要它的智能体。智能体开始任务时会先读取技能描述，判断技能是否与当前任务相关，只有需要时才加载完整指令。

---

## 技能文件的结构

技能是一个包含 `SKILL.md` 文件的文件夹。该文件由简短的前置元数据（用于路由选择的描述）和正文（实际指令）组成。

```
skills/
└── code-review/
    ├── SKILL.md
    └── references/
        └── examples.md
```

前置元数据中的描述是智能体首先读取的内容——应将其写成判断逻辑，让智能体知道何时使用该技能：

```markdown
---
name: code-review
description: >
  Use when asked to review a pull request or code diff.
  Don't use when writing new code from scratch.
---

# Code Review

When reviewing code, check the following...
```

每次心跳开始时，智能体都会看到所有可用技能的名称和描述。如果某个描述与当前工作相符，智能体就会加载该技能的完整内容并遵循其中指令。

---

## 为什么技能能改善性能并降低成本

技能可保持基础上下文精简。拥有 10 项技能的智能体不会在每次心跳时都把 10 项全部加载到上下文中，而只会加载与当前任务有关的技能。因此：

- 每次运行消耗更少 token，成本更低。
- 智能体的上下文更清晰、更聚焦，结果也更好。
- 更易维护：在一个位置更新技能，所有智能体都能受益。

---

## 添加技能

在公司级 **Skills** 页面管理技能。侧边菜单可以切换 **Installed**、**Discover**、**Sources**，以及 **Author** 下的 **My Skills**。**Installed skills** 视图会以卡片显示技能库中的每项技能、使用它的智能体数量和来源。点击卡片可打开技能详情页。

卡片上方有 **Search installed skills…** 搜索框、**Sort** 菜单、扫描按钮（圆形箭头），以及两个按钮：用于整理技能库的 **New folder** 和 **+ New ▾**。所有添加方式都从 **New** 菜单开始：

- **Create new skill**——在 Skill Studio 中从头编写技能。
- **Discover skills**——浏览所有可用来源中的技能。
- **Import from GitHub**——在 **Sources** 下关联代码仓库。
- **Manage sources**——打开 **Sources** 列表。
- **Import from path or URL**——粘贴本地路径、GitHub URL 或 `skills.sh` 命令。
- **Import skills from project**——扫描项目工作区中的技能。

以下小节会根据技能来源逐一介绍这些方式。

### 1. 从 GitHub 仓库导入

如果技能存放在 GitHub 仓库中，建议在 **Skills → Sources** 中添加来源。选择 **New → Import from GitHub**，选定仓库并勾选需要的技能。Paperclip 会保持这些技能与仓库关联，以便之后刷新。下文[从 GitHub 仓库同步技能](#sync-skills-from-a-github-repository)一节会介绍具体操作。

也可以选择 **New → Import from path or URL**。这会打开 **Import a skill** 对话框，其中只有一个输入框——粘贴 GitHub URL（或 `skills.sh` 安装命令）并点击 **Import**。如果还在寻找技能，对话框也提供 **Browse skills.sh** 和 **Search GitHub** 链接。普通的 GitHub 仓库、文件夹或 `SKILL.md` URL 也会归入 **Sources**，因此同样支持刷新。GitHub 来源的技能会固定到具体提交；你可以在技能详情面板检查更新并升级固定版本。

![已安装技能视图：技能卡片上方显示搜索框、Sort 菜单、扫描按钮、New 和 New folder 按钮](../../../user-guides/screenshots/light/skills/add-skill-dialog.png)

### 2. 从 gist 导入

Gist 也可通过同一个 **Import from path or URL** 对话框导入。粘贴 gist URL 后，Paperclip 会将其视为远程文件来源。当你想分享一个一次性的流程、又不想为此创建完整仓库时，这种方式很方便。

### 3. 从本地文件或文件夹导入

也可以在 **Import from path or URL** 对话框中粘贴本地路径（Linux、WSL 或 Windows）。Paperclip 会遍历文件夹、查找 `SKILL.md` 并导入。当你已在本机的项目工作区中保存技能时，这种方式很合适。扫描按钮（**New** 旁边的圆形箭头，标签为 *Scan project workspaces for skills*）会重新扫描公司中所有项目工作区；新添加的 `SKILL.md` 文件夹会自动显示。

出于安全考虑，本地路径必须位于 Paperclip 已知的目录中：公司的受管理技能文件夹、某个项目工作区，或 Paperclip 为你克隆项目的目录（受管理检出目录）。即使项目尚未注册工作区，Paperclip 代为检出的项目也可以就地导入自身技能。其他位置的路径会被拒绝，并显示 *“Local skill source is outside approved company workspace roots”*。请先将该目录添加为项目工作区，或将技能复制到 Paperclip 管理的位置。

### 4. 从 skills.sh 市场导入

[skills.sh](https://skills.sh) 是一个社区现成技能目录。每个条目都会提供可直接复制的安装命令；将命令粘贴到 **New → Import from path or URL** 后，Paperclip 会连同来源元数据一起导入技能（技能行上会显示小型 skills.sh 徽标）。

<a id="5-import-skills-straight-from-a-project"></a>
### 5. 直接从项目导入技能

如果技能已经存放在某个项目所指向的仓库中，就无需自己查找路径。选择 **New → Import skills from project**，对话框会自动查找：*“Pick a project, scan its workspaces for skills, and import them as references.”*

整个流程分为三步。

**选择项目。**界面显示公司的项目列表，顶部有 **Filter projects** 搜索框，每个项目名称下方会显示工作区数量。没有可扫描内容的项目会置灰。

**选择导入内容。**Paperclip 会扫描该项目工作区中的 `SKILL.md` 文件，并按工作区和所在文件夹分组显示找到的内容。勾选需要的技能。**Search discovered skills…** 可缩小较长列表。如果某个技能的 slug 与技能库中的现有条目冲突，该行会显示冲突标记，并出现 **Import as** 输入框；你可以给导入副本指定不同的小写 URL 安全 slug，而不会覆盖现有技能。

扫描结果为空时会显示 **No skills found**，并说明扫描过哪些常见智能体 harness 文件夹。保存在非标准位置的技能仍可通过上文介绍的 **Import from path or URL** 导入。

**检查结果。**最后一步会汇总已导入、已更新和已跳过的技能数量；每项已导入技能都会提供一个 **Open** 链接，直接在 Skill Studio 中打开。

使用此功能前要了解一点：*“No files were copied. These skills reference the files in the project workspace — editing them in Skill Studio saves directly back to those files.”* 项目导入的技能会实时指向检出目录，而不是文件快照。如果仓库是权威来源，这正是所需行为；否则请谨慎使用。若想改为编辑副本，请参阅[编辑不归你所有的技能](#editing-a-skill-that-isnt-yours-to-edit)。

### 从头创建技能

如果以上来源都不适用，可选择 **New → Create new skill**。Skill Studio 会打开 **Create a new skill** 页面，按步骤引导你填写 **Basics**、**Design**、**Content** 和 **Review**；也可随时在步骤之间切换：

- **Basics**——技能名称（卡片上显示的易读名称）、可选短名称（智能体用于引用技能的 `kebab-case` 键；留空时 Paperclip 会根据名称生成），以及单行描述。描述应写成路由规则（“Use when… Don't use when…”），因为智能体会先读取它，再决定是否加载技能正文。
- **Design**——技能磁贴使用的 **Color** 以及可选的 **Categories**。
- **Content**——技能指令正文。
- **Review**——检查全部内容，并选择 **Sharing** 范围（**Organization** 或 **Private**）。

点击 **Create skill** 后，技能会以 Paperclip 管理、可编辑的条目形式加入技能库。

技能现在已可在公司技能库中使用。创建或编辑智能体时，可以将技能库中的可选技能附加给它。

![Skills 页面显示可用技能名称和描述](../../../user-guides/screenshots/light/org/skills-list.png)

### 安装内置目录中的现成技能

你不必从头编写每项技能。Paperclip 自带现成技能**目录（catalog）**，可供浏览并一键安装，帮助你快速采用常见流程。

目录中的技能分为两类：

- **Bundled** 技能是日常工具集，例如维护文档、分流问题、规划任务、执行 QA 验收、管理 GitHub PR 工作流，以及绘制低保真 UI **wireframe（线框图）**。
- **Optional** 技能是按需添加的扩展，例如让智能体操作浏览器、起草发布公告、进行设计评审，或研究过去 30 天里人们对某个主题的讨论。

目录并非只面向工程师。设计、产品和研究角色也能从中获益：例如，`wireframe` 技能可以让智能体先生成 SVG 格式的黑白低保真屏幕布局，再决定是否真正构建界面；这对想先展示结构的设计或产品智能体很有用。`last30days` 技能会从整个网络收集近期帖子和互动情况，让研究人员或营销人员了解当下人们对某个主题的真实讨论。“Bundled”不代表“强制所有人使用”：你仍然可以自行浏览、选择和安装。

有些目录技能并不随应用本身发布，而是从固定到特定版本的外部仓库获取。安装和更新方式相同，只是由 Paperclip 代为获取文件。无论哪种来源，应用都会记录你安装的确切版本，因此下文介绍的保持更新、审计和重置功能适用于所有目录技能。

浏览目录、选中技能并将其安装到公司技能库。安装只会将技能放入技能库，不会自动附加给任何智能体，因此谁来使用仍由你控制。之后它与其他已安装技能相同：可以打开、阅读 `SKILL.md`，并分配给需要它的智能体。

这些技能来自 Paperclip，应用知道你安装的确切版本。因此，可以让目录技能**保持最新**：检查是否有新版本、在信任前审计技能内容是否安全，以及本地编辑后将技能重置为最初发布版本。关于安装、更新、审计和重置目录技能的具体操作，请参阅 [Skills 参考文档](../../reference/skills.md#3-app-shipped-catalog)。

---

<a id="sync-skills-from-a-github-repository"></a>
## 从 GitHub 仓库同步技能

许多团队已将技能与其他代码一起放在 GitHub 仓库中，并通过拉取请求审查。使用 **Sources** 可以将 Paperclip 指向该仓库、选择要导入的技能，并在仓库更新后刷新；无需反复输入 URL，也不会丢失技能来源信息。

可在 Skills 侧边栏的 **Sources** 中打开此功能，也可通过 Skills 页面添加菜单中的 **Import from GitHub** 或 **Manage sources** 进入。

### 添加仓库

1. 在 **Skill sources** 页面点击 **Import from GitHub**。
2. 从列表中选择仓库。列表会汇总你在 **Apps** 中可用的所有 GitHub 连接所关联的仓库，因此无需记住仓库归属于哪个账户。私有仓库会显示锁形图标。
   - 尚未连接 GitHub？点击 **Connect GitHub to see your repos**。如果缺少某个仓库，**Add repos** 也可以执行相同操作。这两个入口都会带你完成 Apps 中的常规设置，然后返回导入器，并保留尚未提交的草稿。
   - 想添加尚未连接的公共仓库？点击 **... or add public repo by URL**，然后将地址粘贴到 **Repository URL**。普通仓库 URL 会跟踪默认分支；如要跟踪其他分支，请粘贴分支 URL，例如 `https://github.com/owner/repository/tree/my-branch`。
3. 点击 **Find skills**。Paperclip 会扫描整个仓库，包括隐藏和嵌套文件夹，并显示扫描进度。任何时候都可以点击 **Cancel scan**；你确认前不会导入任何内容。
4. 检查扫描结果。每个技能包默认都处于选中状态。你选择的是整个技能包，而不是单个文件：一个包会带入其文件夹中的所有文件；嵌套在另一个技能文件夹中的技能则可单独选择。使用 **Search skills or files…** 缩小较长列表。
5. 点击 **Import** 按钮（按钮会显示已勾选的技能数量，例如 **Import 3 skills**）进行导入。带有验证错误的技能会被跳过，并告知跳过数量。

导入前，建议先打开技能检查内容。点击技能或其文件，即可只读预览扫描提交中的确切文件。预览会列出作者声明的 **Runtime requirements**——Paperclip 只展示这些要求，导入时不会安装依赖或运行脚本。**Check references** 会标记指向技能包之外的链接和相对文件路径，帮助你发现该技能依赖于不会随包导入的文件。

导入的技能会直接加入公司技能库，之后可像其他技能一样分配给智能体。

### 保持来源更新

返回 **Skill sources** 后，每个仓库都会列出从该仓库安装的技能。仓库菜单提供三项操作：

- **Refresh** 获取分支的最新提交，并更新你正在同步的技能。Paperclip 不会自行刷新，因此何时将新指令提供给智能体由你决定。
- **Select skills** 重新打开选择器。仓库中新出现的技能会在此等待，行中会显示 *“1 new skill available”*（或相应数量）；勾选后点击 **Save selection** 才会添加。取消勾选已安装技能会停止同步，但 *“Unchecked skills stay installed.”*：技能内容和智能体分配关系都会保留。
- **Disconnect source** 停止同步整个仓库。已安装技能仍会保留在技能库中；如需彻底移除，请另行删除。

刷新会保留每项技能的身份、文件夹、分配关系和历史记录；只有文件实际变化时才会创建新版本。已在运行中的智能体以及固定到特定版本的智能体会继续使用原版本。刷新失败时，智能体继续使用最后一个正常版本；来源会显示错误及 **Review source** 链接。上游已删除的技能仍会保留安装副本，并标记为 **Removed from source · installed copy retained**。

Paperclip 会保存每项同步技能的完整副本，因此智能体运行时无需访问 GitHub。

### 同步的技能为只读

从 GitHub 同步的技能归其仓库所有，因此应在仓库中修改，再执行刷新。在 Skill Studio 中，这类技能会显示 **Read-only** 徽标和说明 *“This skill is synced from GitHub and is read-only.”* 技能详情面板会显示来源信息——**Synced from GitHub**、仓库中的路径和提交——并提供返回 Sources 的 **Manage source** 链接。

如果希望在不修改仓库的情况下为公司定制技能，请点击 **Make a copy** 创建可编辑副本（参见[编辑不归你所有的技能](#editing-a-skill-that-isnt-yours-to-edit)）。

在 Sources 推出之前从 GitHub 导入的技能会自动被识别并列在 Sources 下；刷新一次即可完成本地副本。

---

## 将技能整理到文件夹

技能只有五项时，平铺列表还算方便；有五十项时就会很难管理。Skills 页面左侧有 **Folders** 栏，可将技能放进嵌套文件夹树中，按公司实际使用的方式分组。

文件夹栏按固定顺序排列：

- **All skills**——显示全部技能，不考虑文件夹。这是默认视图。
- **My Skills**——个人空间。每位董事会用户都有自己的空间，只有本人能看到其中内容。适合存放尚在起草、不希望公司其他人误用的技能。
- **Company**——共享文件夹。你可以创建、嵌套、重命名和重新排列这些文件夹。尚未创建文件夹时会显示 *“No company folders yet.”*。
- **Projects**——每个项目自动对应一个文件夹，是项目结构的镜像，因此从项目工作区导入的技能会有明确归属。
- **Bundled**——Paperclip 随附的技能，按类别分组，只读。
- **System → Unfiled**——尚未放入任何文件夹的技能。

**My Skills**、**Projects** 和 **Bundled** 由 Paperclip 管理，不能重命名或删除；**Company** 文件夹则完全由你管理。点击 **Folders** 标题旁的 **+** 可创建顶层文件夹。打开文件夹的 **⋯** 菜单，可选择 **Rename**、**Edit color**，以及 **Move to Company** / **Move to My Skills**——将个人文件夹提升到共享目录，或将共享文件夹移回个人空间。若要将文件夹移到更深的层级，可在父文件夹中使用 **New folder inside…**，或在移动选择器中使用 **Search folders** 搜索框和 **Move here** 按钮。

文件夹最多支持四层嵌套，通常已足够。文件树下方还有 **Tags** 区域：文件夹描述技能**位于何处**，标签则描述技能**是什么**；选择标签后，会在当前浏览的目录范围内筛选。

如需通过脚本创建文件夹、移动技能或重新排序，可以使用 REST API 操作整棵目录树。字段结构、嵌套层数限制和系统管理文件夹规则见[文件夹 API](../../reference/api/folders.md)。

---

## Skill Studio

在一个窗口写技能、另一个窗口测试技能，很快就会变得麻烦。**Skill Studio** 是专用工作区，你可以在其中编写技能、提供输入、使用真实智能体运行测试并查看结果，无需离开页面。

从技能库中的某项技能打开 Studio，或直接访问 `/skills/studio`。面包屑会显示 **Skills → Studio → <skill name>**，让你始终知道所在位置。

### 起始页

访问 `/skills/studio` 但尚未选择技能时，页面会提供两个列表，方便你快速继续工作：

- **Recently visited**——在当前浏览器中最近于 Studio 打开的技能。
- **Recently updated**——公司最近修改的技能；每行显示更新时间和最后编辑者的小头像。

标题栏提供技能选择器（可按名称、slug 或 key 搜索）和 **New skill** 按钮。如果公司尚无任何技能，则会显示带 **Create a new skill** 按钮的空状态。

### 三个面板

打开技能后，Studio 会分成三个面板。拖动分隔线可调整面板大小，布局会被记住以便下次使用。屏幕较窄时，三个面板会改为标签页。

**左侧——技能本身。**显示技能文件夹中的文件树，下方是编辑器。Markdown 文件会在富文本编辑器中打开；如果文件含有前置元数据，正文上方会出现专用面板，避免路由描述埋在原始 YAML 中。其他文件会以纯文本打开。

**中间——输入。**这里是你准备交给智能体的文本。可以粘贴一次性输入，也可以选择已保存的输入以供复用。已保存输入通过名称中的 `/` 组织，方式与文件夹相同——例如命名为 `onboarding/happy-path` 就会形成嵌套结构。点击 **+** 可创建新的 **New input (not saved)**；如果发现值得保留，再保存即可。需要更多空间时可点击箭头折叠整个面板，折叠后会缩为 **Input folded** 单行。

**右侧——测试运行。**选择智能体并按 **Run**，运行会出现在下方历史记录中。运行期间会自动刷新，最终状态为 **succeeded**、**failed** 或 **cancelled**。点击任一运行可打开详情，其中包括输出、生成的文档或附件；运行失败时还会显示解释原因的错误卡片。如果智能体在运行中提问，可直接在运行详情中回答，无需另行查找任务。

### 编辑与保存

Studio 会明确显示未保存的修改，因为不应使用半成品技能进行测试。更改文件后，界面会立即显示 **Unsaved** 徽标、标题栏中的 **Unsaved edits** 标记，并提示 *“Unsaved edits live only in this Studio session. Save to create the next version before running tests or switching files.”* 如果在有未保存修改时尝试切换文件，Studio 会先询问是否放弃修改。

点击 **Save** 会保存文件并立即创建技能的新版本——标题栏会显示当前修订号 `v1`、`v2` 等；打开 **Version history** 可查看完整版本列表并恢复旧版本。

文件树上方的工具栏提供 **Add file**、**Add folder**、**Delete file** 和 **Delete folder**。有两点需要注意：`SKILL.md` 不能删除，因此按钮会禁用并显示提示；新建文件夹时会自动放入 `README.md`，因为空文件夹在文件树中没有可显示内容。

标题栏中的 **⋯** 菜单提供 **Share link**，可复制当前 Studio URL——包括当前查看的输入和运行——让队友打开完全相同的视图。

### 运行测试

**Run** 按钮不可用时会说明原因，而不是让你猜测。将鼠标悬停在按钮上，会显示以下某一条原因：

- *“This skill has no files to test”*
- *“Pick an agent to run”*
- *“Add or paste input text to run”*
- *“Save skill edits before running”*

最后一条尤其重要——测试始终针对已保存的内容，而不是只存在于浏览器草稿中的修改。

在 **Advanced** 下可以选择 **run template**，即 Paperclip 包装输入内容时使用的模板。**Default test template** 是合适的起点；**No template**（*“Run only the input text.”*）只会将原始输入交给智能体，不添加其他内容。你也可以在内置模板旁保存自己的模板，并编辑、复制或删除它们。模板选择按公司分别记忆。

测试运行使用隐藏的 harness 任务，因此不会出现在真实问题列表中。运行记录会保留一段时间后过期；过期运行仍会显示已保存快照，但指向底层任务的链接会被禁用，并显示 **Test task expired**。

### 在 Studio 中创建技能

**New skill** 会打开创建页面，引导你填写基本信息：**Name** 和适合路由的 **Slug**（除非手动指定，否则从名称生成）、**Tagline**、技能在商店和选择器中显示的 **Color** 与 **Categories**，以及 **Sharing** 范围。可选 **Company**（*“Visible inside this company.”*）或 **Private**（*“Only visible in your library.”*）。界面还会显示公共选项，但标为 *Coming later*。如需在创建时编写 `SKILL.md` 开头内容，可展开 **Starter content**。点击 **Create skill** 后，Studio 会打开新技能供你编辑。

如果从某个文件夹内开始创建（例如在 **My Skills** 中），新技能会放在该文件夹里，而不会落入 **Unfiled**。

---

<a id="skills-your-agents-create-during-a-task"></a>
## 智能体在任务中创建的技能

你不必总是亲自编写技能。例如，你可以让智能体将刚总结出的流程保存为可复用技能：“把你审阅这些发布说明的做法保存成一个技能。”在 Paperclip 原生运行器上运行的智能体有专用 **Create skill** 工具：它会编写完整的单文件 `SKILL.md`，并直接保存到公司的技能库。

以下规则让行为保持可预测：

- **只在 Auto 模式下可用。**该工具只在任务以 Auto 模式运行时（以及技能测试运行中）提供；Ask 和 Plan 模式不可用。参见[工作模式](../day-to-day/work-modes.md)。
- **遵守技能策略。**以此方式创建技能时，会执行与 Skill Studio 相同的规则。未设置策略时允许智能体创建；如果 `skills.create` 被限制，智能体会像其他调用者一样被拒绝。参见[谁可以更改技能](#who-is-allowed-to-change-skills)。
- **不会自动分配给任何人。**新技能只会加入技能库；只有你或有权操作的智能体进行分配后，其他智能体才会开始使用。参见[将技能分配给智能体](#assigning-a-skill-to-an-agent)。
- **重试不会创建重复项。**如果智能体请求中断后重试，它会取回已创建的技能，而不是生成第二份。

技能创建后，任务讨论串中会出现 **Skill created** 卡片，显示技能名称和描述。点击卡片，技能会在任务侧边栏的新标签中打开，显示 slug、当前修订和指令。该标签始终读取技能库中的实时副本，不需要另行同步第二份版本。使用 **Open in Skill Studio** 编辑；返回任务后，标签会显示已保存的内容。如果技能之后被删除或你失去访问权限，标签会分别提示 *“Skill no longer available.”* 或 *“You do not have access to this skill.”*；讨论串中的卡片仍会保留，作为技能曾被创建的记录。

创建操作还会记入 `company.skill_created`，并同时出现在任务历史和公司活动日志中。

<a id="letting-an-agent-improve-an-existing-skill"></a>
### 让智能体改进现有技能

技能会随着使用不断改进，而刚刚执行过技能的智能体通常最清楚需要如何修正——例如“检查清单漏了变更日志步骤，把它补上”。原生运行器也提供 **Update skill** 工具，供智能体一次性重写技能的整个 `SKILL.md`；其行为与在 Skill Studio 中保存相同。

该工具的名称为 `update_skill`。智能体读取技能及其 `currentVersionId`，然后提交完整的主文件以及 `expectedVersionId` 和 `idempotencyKey`，而不只是想修改的几行。

此设计可避免智能体覆盖其他人的工作：

- **模式相同，规则相同。**与 **Create skill** 一样，该工具只在 Auto 模式和技能测试运行中提供，并遵守技能策略——此处使用 `skills.edit` 规则。智能体无权编辑技能时会被拒绝。它只能修改你可以在 Studio 中编辑的技能，因此只读技能（内置目录、GitHub 来源、skills.sh）不可编辑。
- **不会覆盖更新的修改。**智能体必须指出它读取的技能版本。如果其他人在此期间保存了更改，更新会被拒绝，并提示 *“Skill version changed. Read the current version before retrying.”* 智能体随后会重新读取技能，并基于最新版本重试，而不会静默撤销他人修改。
- **重试不会重复创建版本。**如果请求中断后重试，智能体会获得原有结果，而不是创建第二个版本。
- **所有操作都有记录。**每次有效更新都会在技能历史中创建一个新版本，可在 Studio 中比较或恢复；同时会以 `company.skill_file_updated` 事件记入执行该操作的任务。

版本冲突后，智能体会重新读取技能，并为修订后的请求使用新的重试键。输入字段结构见 [Skills 参考文档](../../reference/skills.md#native-runner-skill-tools)。实现参考：[tool contract](https://github.com/paperclipai/paperclip/blob/bba6a45f0d364cfd962ac59b3bdf2426a8222adb/packages/paperclip-runner/src/protocol-actions/update-skill.ts) 和 [file API binding](https://github.com/paperclipai/paperclip/blob/bba6a45f0d364cfd962ac59b3bdf2426a8222adb/server/src/services/skill-tools.ts)。

---

## 重命名技能

名称会随着使用而过时。你第一周创建的 `code-review` 技能后来变成了 PR 分流流程，但所有智能体仍使用这个容易误导其他人的名称。你可以重命名技能，无需拆除并重新建立所有关系。

重命名会同时修改三项内容，因为它们在 Paperclip 中彼此关联：

- **名称（name）**——你和团队在技能库中看到的名称。技能 `SKILL.md` 前置元数据中的 `name:` 行也会同步修改，因此文件和技能库不会不一致。
- **slug**——URL 中使用的 `kebab-case` 短名称。可以明确指定；留空时 Paperclip 会从新名称生成。
- **key**——智能体引用技能时使用的稳定标识符。它会随 slug 一起变化，因此更换 slug 也会生成新 key。

最后一点通常最容易在重命名时造成问题，但 Paperclip 会自动处理：**原先附加了旧技能的每个智能体都会自动转到新 key**，并保留其固定版本。技能不会被静默解绑，也无需逐个打开智能体列表重新勾选。

重命名前请了解以下事项：

- **只有 Paperclip 管理的技能可以重命名。**内置目录、GitHub、skills.sh 或项目扫描导入的技能均为只读，原因与其内容只读相同。若需创建可重命名的副本，请参阅[编辑不归你所有的技能](#editing-a-skill-that-isnt-yours-to-edit)。
- **新 slug 必须尚未被使用。**如果公司中的另一项技能已占用该 slug，重命名会被拒绝而不会覆盖任何内容，并会告知冲突对象。
- **重命名会写入审计记录。**公司活动日志会新增 `company.skill_renamed`，记录旧名称、新名称、slug 和 key，以及所有已重新分配的智能体。
- **重命名遵守技能策略。**重命名属于编辑操作，因此公司限制技能修改权限时，同一策略也适用。参见[谁可以更改技能](#who-is-allowed-to-change-skills)。

如需通过脚本重命名，端点为 `POST /api/companies/{companyId}/skills/{skillId}/rename`。必填的 `name` 必须为单行；`slug` 可选。省略 slug 时会从名称生成。响应会返回重命名后的技能、旧名称/slug/key，以及已重新分配的智能体列表。

---

<a id="editing-a-skill-that-isnt-yours-to-edit"></a>
## 编辑不归你所有的技能

许多实用技能来自你无法控制的来源，例如 GitHub 仓库、skills.sh 或内置目录。它们被设为只读，是为了避免有人在智能体使用期间悄悄改写已固定的技能。在 Studio 中打开此类技能时，会看到 **Read-only** 徽标、只读原因说明和 **Make a copy** 按钮。

点击 **Make a copy** 会打开标题为 *Edit a copy of …* 的对话框，将技能复制为技能库中可编辑且由 Paperclip 管理的副本。执行前会先进行检查，并明确显示当前有多少智能体正在使用原技能；同时会提供一个默认开启的开关，用于将这些智能体转移到副本。这个选择至关重要：复制但不转移智能体，得到的副本不会被运行；打开开关后，你的修改会在下次运行时生效。无论如何，原技能都不会被修改。

副本创建后，Studio 标题栏会显示溯源标记 **Forked from `owner/repo` @ `<short sha>`**，并链接回来源技能；数月后仍能看出副本的起源。如果你已经复制过该技能且尚未产生差异，Studio 会提供打开现有副本的选项，而不会再创建一份。

**从项目导入的技能属于特殊情况。**这类技能可以编辑，但它们并非副本——而是直接指向工作树中的文件。Studio 会持续显示来源位置，并警告 *“Saves write to the project working tree and are not committed.”* 如果不想修改检出目录，可直接点击 **Edit a copy instead**。

---

<a id="who-is-allowed-to-change-skills"></a>
## 谁可以更改技能

默认情况下没有权限限制：所有智能体都可以创建、导入、安装、编辑、测试和移除技能，UI 也不会显示权限相关控件。这是有意设计的，因为大多数公司无需额外限制。

需要收紧权限时，可通过**公司技能策略（company skill policy）**指定哪些智能体可以对哪些技能执行哪些操作，以及可使用哪些来源。策略启用且确实拒绝某项操作时，Paperclip 会明确显示横幅，指出被拒绝的操作及后续处理方式，而不只返回通用错误。

策略文档结构、八种技能操作以及策略判定过程，见[公司技能策略 API](../../reference/api/company-skill-policy.md)。

---

## 文件清单

每项技能都是一个文件夹，而非单个文件。打开技能详情页并切换到 **Files** 标签（与 **Overview**、**Versions** 和 **Agents** 并列），即可查看技能根目录下的完整**文件清单（file inventory）**。每个文件都会显示若干实用元数据：

- **Entry file**——顶层 `SKILL.md`，智能体会首先读取它。它始终排在文件树顶部，包含技能前置元数据（名称、描述）和指令正文。
- **Other files**——`references/` 或 `scripts/` 等子文件夹中可放置支持材料（示例、风格指南、辅助脚本）。文件类型图标会标明每个文件是 Markdown、参考文档还是脚本。
- **Editable**——只有 Paperclip 控制其来源时技能才可编辑。从 GitHub、gist 或 skills.sh 目录导入的技能是**只读**的：可以查看文件，但无法保存更改；详情面板会显示原因徽标，而不是 Edit 按钮。Paperclip 管理的技能和本地文件夹技能可点击 **Edit** 在 Markdown 编辑器中打开。
- **Deprecated**——技能退役后会在技能库中标记为 deprecated，并从智能体技能菜单隐藏。现有智能体仍可继续使用，直到你将其解除分配。
- **Virtual**——部分清单条目是虚拟的，代表适配器在运行时合成的内容（例如不原生支持技能的适配器所需的运行时技能）。行中会标记为虚拟，且无法直接编辑。

![技能详情页的 Files 标签：文件列表中含 SKILL.md，渲染文件上方有 View、Code 和 Edit 控件，右侧显示 Location、Tags、Agents 和 Source 栏](../../../user-guides/screenshots/light/skills/file-inventory.png)

文件查看器上方会显示文件名、**View** / **Code** 切换按钮；技能可编辑时还会显示 **Edit**。无论当前打开哪个标签，页面右侧栏都会显示技能元数据：

- **Location**——技能所在文件夹，提供 **Copy path** 和 **Move** 按钮。
- **Tags**——技能标签，以及 **Add a tag…** 输入框。
- **Agents**——已附加该技能的智能体，以及 **Add to agent** 按钮。
- **Source**——技能来源（例如 Paperclip 工作区、本地文件夹或 GitHub），并在下方显示路径或定位信息。
- **Updates**——仅适用于 GitHub 来源技能。显示 Paperclip 固定的提交、**Check for updates** 按钮；有更新时还会显示 **Install update** 操作。
- **Settings**——位于侧栏底部的技能配置。

技能的 **Key**（适配器使用的稳定标识符）和 **Mode**（Editable 或 Read only）位于 **Overview** 标签的 **About** 详情中。

---

## 适配器同步偏好

不同适配器对技能的处理方式不同。智能体的 Agent → Skills 标签会在技能列表上方、技能启用数量旁显示 **skill application mode** 徽标；悬停即可查看详情：

- **Kept in workspace**（`persistent`）——适配器会将技能文件写入智能体工作区目录，并在运行之间保留。大多数长期运行的本地适配器使用此模式。
- **Applied on next run**（`ephemeral`）——每次运行时临时生成技能文件，结束后清理。将工作区视为临时目录的沙盒适配器通常默认使用此模式。
- **Tracked only**（`unsupported`）——Paperclip 无法将技能推送到适配器，因此只记录已分配的技能。`openclaw_gateway` 等适配器属于此类；界面会显示横幅，指导你在远程运行时（例如 OpenClaw）中管理技能，而不是通过 Paperclip 管理。

无论同步模式如何，分配关系都会保存；模式只控制 Paperclip 是否实际同步技能文件。如果公司技能库中缺少指定技能，或适配器拒绝同步，界面会通过琥珀色横幅显示警告。

---

<a id="assigning-a-skill-to-an-agent"></a>
## 将技能分配给智能体

技能属于公司级资源，但每个智能体各自决定要使用哪些技能。可以从技能或智能体两侧进行分配。

**从技能侧分配。**在 **Installed** 视图中打开技能详情页。在右侧 **Agents** 区域点击 **Add to agent**。选择器会显示 **Filter agents** 搜索框、**Version** 菜单和每个智能体对应的复选框；勾选目标智能体并点击 **Save**。

![技能详情页打开 Add to agent 选择器，显示智能体复选列表和 Version 菜单](../../../user-guides/screenshots/light/skills/assign-to-agent.png)

**从智能体侧分配。**如需查看和更改某个智能体使用的所有技能：

1. 从 Agents 列表或组织架构图打开智能体详情页。
2. 切换到 **Skills** 标签。顶部会显示 “*N* of *M* enabled”、技能应用模式徽标和 **Search skills** 搜索框。
3. 页面会显示以下分组：
   - **Enabled on this agent**——智能体当前使用的技能。关闭某行的开关即可解除分配。
   - **Available from the library**——公司技能库中的其他技能，智能体可以使用。打开某行开关即可分配。
   - **Automatic and detected skills (read-only)**——可折叠分组，显示适配器提供的技能，或从 Paperclip 无法控制的位置检测到的技能（例如主机上的全局技能包）。此处仅供查看。
4. 停止点击后，修改会在短暂延迟后自动保存；状态标记会依次显示 “Saving soon…”，然后 “Saving…”，最后 “Saved”。智能体会在下次运行时使用新的技能列表。

标签顶部的 **Browse skills store** 按钮会跳转到公司级 Skills 页面，因此你可以打开对应技能、调整指令，然后返回原位置。

> **提示：**预期的技能没有出现在 **Available from the library** 中，表示它尚未加入公司技能库。请先在 Skills 页面添加，再返回此处。

你可能会在技能库和入职时创建的首个智能体上看到 `first-task` 技能。这是 Paperclip 为首次任务提供的指南——说明智能体如何读取你对开场问题的回答、提出计划或单项任务，并在招聘或开始工作前等待你的同意。它仅适用于入职任务，不会应用于该智能体的其他工作，也不会自动分配给其他智能体。

---

<a id="pinning-a-beta-release-of-the-paperclip-skill"></a>
## 固定 Paperclip 技能的测试版

捆绑的 `paperclip` 技能是每个智能体每次运行都会遵循的基础流程，它会持续更新。通常这正是你希望的状态：改进后的指导发布后，所有智能体都会采用。但有时你可能希望固定某个智能体，例如比较两个流程版本的表现，或在评估新版时暂时不改变某项运行所依赖的快照。

**测试版发布（Beta releases）**允许你固定版本。发布版是 Paperclip 核心技能的冻结快照，会与实时版本一同放入公司技能库。将智能体固定到某个发布版后，它会运行该快照；其他智能体继续使用实时默认版本。

### 先启用功能

有人启用前，版本选择器会隐藏。打开 **Settings → Experimental** 并启用 **Beta skills** 卡片，说明文字为：*“Allow agents to pin beta releases of the Paperclip core skill. Disabling this returns every agent to the default live skill without removing saved pins.”*

云端和自托管实例都默认关闭该功能。它不属于云端管理的标志，因此任何实例上的开关都由你控制。关于该页面行为，参见[实验功能](../../experimental/overview.md)。

### 选择发布版

打开智能体并切换到 **Skills** 标签，找到 Paperclip 核心技能行——该技能必须已对该智能体启用。旁边会出现 **Skill release** 下拉菜单：

- **Default — current (recommended)**——不固定版本。智能体运行实时技能，并获取未来所有更新。所有智能体的初始设置都是此项。
- 各个冻结发布版会显示其名称和发布日期，例如 **V7 — Roster champion · released 2026-07-21**，并带有小型 **Beta** 徽标。

目前提供两个发布版：原始核心技能快照 **V0 — Original**，以及 **V7 — Roster champion**。这两个版本都刻意保留为历史快照。**V7 — Roster champion** 早于 `SKILL.md`、`references/api-reference.md` 和 `references/company-skills.md` 后续三次编辑；实时默认版本包含更新的指导。选择固定版本时应考虑这项取舍：固定可获得不会变化的流程，但也无法获得快照发布后的任何改进。这就是默认选项标为 *recommended* 的原因。

选择发布版后会立即保存，无需等待常规自动保存；技能行会显示类似 **Beta · V7** 的徽标，让你快速查看哪些智能体使用固定版本。切回 **Default — current (recommended)** 即可取消固定。

### 关闭功能标志后的行为

再次关闭 **Beta skills** 不会删除任何内容。智能体上保存的固定版本仍保留，只是暂时忽略；所有智能体会在下次运行时恢复使用实时默认技能。重新启用后，固定版本会再次生效。

API 遵循相同规则：功能关闭时保存版本固定会被拒绝，并返回 *“Beta skill version pins require the Beta skills experimental setting to be enabled.”* 请求结构、发布版字段和快照来源见 [Skills 参考文档 → 捆绑 `paperclip` 技能的 Beta releases](../../reference/skills.md#beta-releases-of-the-bundled-paperclip-skill)。

---

## 信任级别

技能会作为附加指令加载到智能体运行中，因此来源很重要。Paperclip 会在每个技能行上显示 **Source** 徽标，明确标出来源：

- **Paperclip-managed**——在 Paperclip 中编写。内容完全由你控制，是交给智能体使用时最安全的一类。
- **Local folder**——从同一机器上的项目工作区导入。信任级别取决于你对该文件夹本身的信任程度。
- **GitHub / skills.sh**——固定到特定提交或发布版本。由于固定版本不会在未经明确 **Install update** 的情况下自行变化，你可以先检查差异，再决定是否将更新提供给智能体。
- **Gist / URL**——外部内容在某个时间点导入的快照。应像审查第三方代码一样先检查正文，再将技能附加给智能体。

将技能附加到高信任角色（例如 CEO、财务智能体或可访问机密的操作人员）之前，请打开技能并从头到尾阅读其 `SKILL.md`。详情面板中的 **Used by** 列表可帮助你审计哪些智能体已经接触到该技能。

---

## 如何编写好技能

**将描述写成路由逻辑。**描述不是宣传文案，而是智能体决定是否加载技能的信号。请写明 “Use when” 和 “Don't use when” 条件。

**内容具体且可执行。**智能体应能照做而无需猜测。含糊的指令（例如“仔细审查代码”）会带来含糊的结果；明确的检查清单和步骤能带来一致结果。

**在适用时加入示例。**代码片段、示例输出、前后对照等具体示例通常比抽象说明更可靠。

**每项技能只处理一个主题。**同时涵盖代码审查、部署和客户沟通的技能会让三个目标互相争夺注意力。一项技能应有一个用途。

**将补充细节放入 `references/`。**技能需要风格指南、示例文件或参考资料时，应放入 `references/` 子文件夹，而不是把所有内容塞进 `SKILL.md`。

---

## 多个智能体共享技能

技能属于公司级资源，而不属于某一个智能体。因此：

- CTO 和 Backend Engineer 可以共用 `code-review` 技能，无需重复创建。
- CEO 可以使用 `delegation-checklist` 技能；CMO 也可以将它用于自己的任务拆分。
- 更新技能后，所有使用它的智能体都会同时受益。

如果磁盘上已经有技能文件，也可以扫描并从项目工作区导入，适合将现有 `SKILL.md` 文件夹迁移到 Paperclip 技能库。参见[直接从项目导入技能](#5-import-skills-straight-from-a-project)。

---

## 通过程序管理技能

大多数时候你会通过 UI 管理技能，但也可以使用 REST 端点编写自动化入职流程，或从另一家公司复现已知的智能体设置。

- **列出智能体已附加的技能**：`GET /api/agents/{agentId}/skills`
- **同步已附加技能集合**：使用 `POST /api/agents/{agentId}/skills/sync`，请求结构为 `{ "desiredSkills": [...], "mode": "add" | "remove" | "replace" }`。每个 `desiredSkills` 条目可以是技能 UUID、规范 key 或唯一 slug。必填的 `mode` 决定列表如何应用：`add` 将指定技能并入当前集合；`remove` 只移除指定技能；`replace` 会协调技能集合，使其与指定内容完全一致——添加缺少的技能并移除列表之外的技能。
- **招聘时预配技能**：在 `POST /api/companies/{companyId}/agents` 中传入 `desiredSkills`，新智能体上线时就会附带相应技能。

分配技能前，必须先将它们安装到公司级技能库。完整参考资料涵盖文件结构、安装流程、规范 key、版本管理和故障排查：

- [Skills 参考文档](../../reference/skills.md)——技能在磁盘和传输过程中的完整工作方式。
- [Agents API → Skills](../../reference/api/agents.md#skills)——智能体级路由的请求/响应结构。
- [Skills 参考文档 → GitHub 技能来源](../../reference/skills.md#github-skill-sources)——**Skills → Sources** 背后的路由。
- [文件夹 API](../../reference/api/folders.md)——创建、嵌套和重新排列技能文件夹树。
- [公司技能策略 API](../../reference/api/company-skill-policy.md)——限制哪些人可以创建、导入、编辑、测试或移除技能。

---

## 准备就绪

技能为智能体提供可靠、可复用的流程，提升一致性并降低成本。下一篇指南介绍导出与导入——如何备份、分享和复用公司配置。

[导出与导入 →](../power/export-import.md)
