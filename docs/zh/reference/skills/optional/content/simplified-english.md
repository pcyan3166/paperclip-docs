---
seo_title: 简明英语技能
seo_description: 要求智能体遵循 ASD-STE100 简明技术英语撰写评论、计划和交接文档，让使用任何语言的读者都能一次读懂。
---

# 简明英语

> 使用 ASD-STE100 简明技术英语撰写面向用户的评论、计划和文档：句子简短、含义明确，使用认可词汇且每句话只表达一个意思，让读者首次阅读就能理解。

当智能体写出的内容会由真人阅读时（例如任务评论、计划或交接文档），你希望读者第一次看就能理解。此技能要求智能体遵循 ASD-STE100 简明技术英语：使用短句、每次只给出一项指令、使用主动语态，并遵循受控词汇表，使每个词只表达一种含义。

此技能是**可选**目录技能，可按需安装。如何安装、审计、更新、分配和重置目录技能，请参阅[技能参考](../../../skills.md#3-app-shipped-catalog)。

## 适用场景

- 读者中有人使用第二语言工作，含糊的表达会浪费他们的时间。
- 智能体撰写人类会直接据以行动的计划、评论或文档。
- 你希望整个智能体团队在面向用户的写作中遵循统一的风格。

### 不适用场景

- 输出是代码、配置或机器可读数据，而不是供读者阅读的文字。
- 内容需要营销语气或叙事修饰；受控词汇表不适用于这类表达。

## 目录元数据

| 字段 | 值 |
|---|---|
| 目录 ID | `paperclipai:optional:content:simplified-english` |
| 规范键 | `paperclipai/optional/content/simplified-english` |
| 目录路径 | `catalog/optional/content/simplified-english` |
| 类型 | `optional` |
| 类别 | `content` |
| Slug | `simplified-english` |
| 入口文件 | `SKILL.md` |
| 信任级别 | `markdown_only` |
| 兼容性 | `compatible` |
| 默认安装 | `false` |
| 建议角色 | `engineer`、`product`、`writer`、`devrel` |
| 依赖项 | — |
| 标签 | `writing`、`communication`、`clarity`、`style` |
| 文件数 | 1 |
| 内容哈希 | `sha256:642321f40d51e0125df05c6fbace49aeb07ec30392058756909c259c9c5eac6a` |
| 软件包 | `@paperclipai/skills-catalog@0.3.1` |

## 文件清单

| 路径 | 类型 | 字节数 |
|---|---|---:|
| `SKILL.md` | `skill` | 1,578 |

## 完整技能定义

下方代码块是此技能完整且权威的 `SKILL.md`，也是智能体运行时实际加载的文件。可以使用代码块右上角的控件进行复制，或将其下载为 `SKILL.md`。

````markdown skill-source
---
name: simplified-english
description: Write user-facing comments, plans, and documents in ASD-STE100 Simplified Technical English — short, unambiguous sentences with approved words and one meaning each — so readers understand them the first time.
key: paperclipai/optional/content/simplified-english
recommendedForRoles:
  - engineer
  - product
  - writer
  - devrel
tags:
  - writing
  - communication
  - clarity
  - style
---

# Simplified English

For user-facing comments, plans, and documents, write using only ASD-STE100 Simplified Technical English.

## Core rules

- Use short sentences (procedures ≤ 20 words, descriptions ≤ 25 words).
- Give one instruction per sentence.
- Use approved words with one meaning each; avoid synonyms and jargon.
- Use the active voice and the present tense.
- Use articles ("the", "a") and do not drop words to save space.
- Write positive instructions; avoid negative or vague qualifiers.
- Keep paragraphs to one topic.

## Approved words

The approved words are the ASD-STE100 controlled vocabulary — the Dictionary in the current ASD-STE100 specification, plus the technical names and technical verbs that your subject needs. When a word is not approved, use the simplest common word that has one meaning. Prefer these house choices:

- "start" / "stop" (not "initiate", "commence", "terminate", "kill")
- "make" (not "implement", "leverage", "utilize")
- "before" / "after" (not "prior to", "subsequent to")
- "about" (not "regarding", "in relation to")
- "help" (not "facilitate")
- "use" (not "utilize", "employ")
````

## 中文译文

# 简明英语

面向用户撰写评论、计划和文档时，只使用 ASD-STE100 简明技术英语。

## 核心规则

- 使用短句（步骤句不超过 20 个单词，描述句不超过 25 个单词）。
- 每句话只给出一条指令。
- 使用每个词义明确的认可词汇；避免同义替换和行话。
- 使用主动语态和现在时。
- 使用冠词（“the”“a”），不要为了节省篇幅省略单词。
- 使用肯定的指令；避免否定或含糊的限定语。
- 每段只讨论一个主题。

## 认可词汇

认可词汇是 ASD-STE100 的受控词汇，包括当前 ASD-STE100 规范中的词典，以及主题所需的技术名称和技术动词。若某个词不在认可词汇中，应使用含义明确且最简单常见的词。优先采用以下写作习惯：

- 使用 “start” / “stop”，不要用 “initiate” / “commence” / “terminate” / “kill”。
- 使用 “make”，不要用 “implement” / “leverage” / “utilize”。
- 使用 “before” / “after”，不要用 “prior to” / “subsequent to”。
- 使用 “about”，不要用 “regarding” / “in relation to”。
- 使用 “help”，不要用 “facilitate”。
- 使用 “use”，不要用 “utilize” / “employ”。

## 另请参阅

- [可选技能](../../optional.md)——按类别整理的全部可选目录技能。
- [技能参考](../../../skills.md)——文件结构、安装流程、目录浏览/安装/审计/更新/重置、分配和故障排查。
- [应用内置目录](../../../skills.md#3-app-shipped-catalog)——内置和可选目录技能的版本管理与更新方式。
- [技能指南](../../../../guides/org/skills.md)——界面操作指南和内置目录概览。
