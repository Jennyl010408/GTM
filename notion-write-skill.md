---
name: notion-write
version: 1.0.0
description: |
  Notion 文档写作规范。创建或更新 Notion 页面时自动加载，确保页面结构、
  callout 颜色体系、表格、columns 布局、强调方式和语言风格保持一致。
  Use when creating Notion pages, writing Notion content, or updating
  Notion documents.
triggers:
  - notion
  - 写notion
  - notion文档
  - 创建notion页面
  - 写文档
---

# Notion 写作规范

写 Notion 内容时，严格遵守以下规范，保持所有文档的视觉和结构一致性。

---

## 页面结构

- 页面顶部：**蓝底 callout** 作为摘要（含 `**定位/目标/关键信息**` 等加粗标签）
- `---` 分隔符隔开大板块
- 大板块用 `#` 标题（如 `# 核心机制`、`# Part 1：竞品调研`）
- 子板块用 `## 标题 {toggle="true"}`，中文编号（一、二、三）
- Toggle 内子内容用 tab 缩进
- 每个大板块开头可用 `gray_bg` callout 做一句话说明

## Callout 颜色体系

| 颜色 | 用途 | 示例 |
|------|------|------|
| `blue_bg` | 定位、身份、信息总结 | 页面摘要、身份认同 |
| `green_bg` | 权益、正面结论、共性规律 | 核心权益、官方曝光 |
| `yellow_bg` | 规则、洞察、提示 | 阶梯式结算规则、建议 |
| `red_bg` | 警告、门槛、安全条款 | 达标线、品牌安全 |
| `orange_bg` | 里程碑、升级 | 升级条件 |
| `purple_bg` | 特殊权益、创意主题 | 特权体验、活动主题 |
| `gray_bg` | 中性说明、板块引言、流程 | 板块开头一句话说明 |

## 强调方式

- 关键强调：`<span color="red">**文字**</span>`
- 可借鉴要点：`<span color="pink_bg">**Helio 可借鉴：**</span>`
- 普通强调：`**加粗**`
- 颜色标注文字：`<span color="blue/green/orange">**文字**</span>`

## 表格

- 始终用 `header-row="true"`
- 表头加粗：`**列名**`
- 用 `<colgroup>` 控制列宽（需要时）

## 并列内容（Columns）

- 2-3 列用 `<columns>` 布局
- 每列内通常是一个 callout（同色系或 `gray_bg`）
- 列内 callout 用 `<br>` 换行，不用多段落

## 可展开内容（Toggle）

- 大结构用 `{toggle="true"}` 在 heading 上
- 小粒度用 `<details>/<summary>` 嵌套在 toggle 内部
- details 内用 **加粗** 做子标题，列表用 `-`

## 语言风格

- 中英混用：中文写说明，英文写术语/标题/专有名词
- 如 `## Plan 1：Helio Pioneers`、`<span color="blue">**"Helio Star Creator"**</span>`

## 流程图

- 简单流程用 code block（```html）+ 文字箭头
- 复杂关系用 mermaid flowchart

## 方案类文档额外规则

- 聚焦策略层（"做什么"和"为什么"），不写过于细节的实现步骤
- 实现细节留到用户要求执行时再展开

---

## 快速参考模板

### 标准页面骨架

```
<callout icon="🎯" color="blue_bg">
	**目标：** xxx
	**定位：** xxx
</callout>
---
# Part 1：xxx
<callout icon="🔍" color="gray_bg">
	一句话说明这个板块的目的
</callout>
## 一、xxx {toggle="true"}
	内容...
## 二、xxx {toggle="true"}
	内容...
---
# Part 2：xxx
...
```

### 对比表格

```
<table header-row="true">
<tr>
<td>**维度**</td>
<td>**选项 A**</td>
<td>**选项 B**</td>
</tr>
<tr>
<td>xxx</td>
<td>xxx</td>
<td>xxx</td>
</tr>
</table>
```

### 并列 Callout

```
<columns>
	<column>
		<callout icon="📸" color="gray_bg">
			**标题**<br>说明文字
		</callout>
	</column>
	<column>
		<callout icon="🎥" color="gray_bg">
			**标题**<br>说明文字
		</callout>
	</column>
</columns>
```
