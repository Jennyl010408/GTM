---
name: notion-task-sync
version: 1.0.0
description: |
  从团队成员的 Notion 追踪数据库中提取任务数据，生成/更新【每日最新进度追踪】
  页面的每周任务清单。自动扫描所有追踪文档，按规则筛选任务并分配到每日。
triggers:
  - 任务清单
  - 进度追踪
  - 每日任务
  - task sync
  - 更新追踪
  - 生成清单
---

# Notion 每日任务清单同步

从团队追踪数据库自动生成每周任务清单，写入【每日最新进度追踪】页面。

---

## 目标页面

- **页面 ID：** `364bb9e3b84d809bbc5dc65b0574bfe6`
- **页面名：** 【每日最新进度追踪】

---

## 团队成员 & User ID

| 成员 | User ID | mention-user URL |
|------|---------|------------------|
| Jiangran | 35dd872b | `user://35dd872b-594c-81b3-873e-0002c11a3104` |
| Aurora | 34bd872b | `user://34bd872b-594c-8116-a376-00022c244535` |
| Yori | 34ad872b | `user://34ad872b-594c-817c-b7db-000244a6a43c` |
| Jenny | 304d872b | `user://304d872b-594c-81a0-ac82-00024866a7db` |

---

## 数据源（全部扫描）

每次执行必须查询以下所有数据库 view，不能遗漏：

### Jiangran
| 数据库 | View URL |
|--------|----------|
| 中推-任务追踪 | `view://367bb9e3-b84d-80c6-aa72-000c380893f0` |
| Reddit-内容追踪 | `view://368bb9e3-b84d-80b9-b4a6-000c387656a8` |

### Aurora
| 数据库 | View URL |
|--------|----------|
| X运营追踪 | `view://367bb9e3-b84d-80e0-a053-000c3a309b84` |
| x kol合作 | `view://367bb9e3-b84d-80fb-b38c-000c137cb755` |
| Corgi线下 | `view://367bb9e3-b84d-8054-8ade-000c0a5dd7b1` |

### Yori
| 数据库 | View URL |
|--------|----------|
| LinkedIn-任务追踪 | `view://367bb9e3-b84d-80b4-9aae-000ce044c1ec` |
| 领英素材计划 | `view://36bbb9e3-b84d-80c8-a647-000c8b6bf811` |
| YouTube-任务追踪 | `view://367bb9e3-b84d-805a-aedb-000c6e9d328f` |
| PR类-任务追踪 | `view://368bb9e3-b84d-80e3-ae92-000c9cb0556a` |

### Jenny
| 数据库 | View URL |
|--------|----------|
| Reddit 追踪-任务追踪 | `view://e15b217d-9047-4862-be76-85e524a0f28a` |
| Reddit 追踪-内容&账号 | `view://36bbb9e3-b84d-8030-95b4-000c9f2e0045` |
| Community-Led 追踪 | `view://367bb9e3-b84d-8051-ac5d-000cab9c0164` |

### 共享
| 数据库 | View URL |
|--------|----------|
| 主页顶部 DB（KOL英推 view） | `view://367bb9e3-b84d-8025-ae36-000c9f5fb449` |

---

## 任务筛选规则（严格执行）

### 1. 状态映射

| DB 状态值 | 处理方式 |
|-----------|----------|
| Done / 完成 / 已完成 | 写入，标记 `[x]` |
| In progress / 进行中 | 写入，标记 `[ ]` |
| Not started / 未开始 | **排除，不写入** |
| 无状态字段 | 按下面的特殊规则判断 |

### 2. "每日"任务规则

任务名称含**"每日"**且状态为 **进行中/完成** → 在目标周的**每一天**都记录。
判断方法：日期字段通常设在未来（如 5.30、5.31、6.30），表示持续性任务。

### 3. 素材/内容任务规则

素材类任务（如 X运营追踪、领英素材计划）如果有**日期字段可匹配到目标周的某一天** → 写入对应那天，无论状态是否为空。

### 4. 未开始绝对排除

状态明确为"未开始 / Not started"的任务 → 无论其他条件如何，**一律不写入**。

---

## 输出格式

### 页面结构

```
# 每日任务清单（M.DD-M.DD） {toggle="true"}
	<callout icon="💡" color="gray_bg">
		list是严格按照大家写的plan自动分配生成的🙅不要手动增减，只可以备注📝<br>如果未完成**不要删除内容：**简易描述一下why，遇到什么问题等
	</callout>
	### M.DD 周X {toggle="true"}
		**<mention-user url="user://..."/>**
		- [ ] 任务名称
		- [x] 已完成任务
		<empty-block/>
		**<mention-user url="user://..."/>**
		- [ ] 任务名称
```

### 格式要点

- 每日标题用 `### M.DD 周X {toggle="true"}`
- 每人用 `**<mention-user url="..."/>**` 作标题
- 任务用 `- [ ]` 或 `- [x]`
- 人与人之间用 `<empty-block/>` 分隔
- 所有内容在 toggle 内部用**双 tab 缩进**（`\t\t`）
- 每人出现顺序固定：Jiangran → Aurora → Yori → Jenny
- 如果某人当天没有任务，**不写该人的 section**

---

## 执行流程

### Step 1：获取当前页面

用 `notion-fetch` 获取目标页面，确认：
- 当前已有的清单内容（保留手动勾选和备注）
- 页面整体结构

### Step 2：并行查询所有数据库

用 `notion-query-database-view` 并行查询上面列出的所有 12 个 view。

### Step 3：数据处理

对每个 DB 返回的数据：
1. 识别状态字段（不同 DB 可能叫"状态"/"Status"，类型可能是 status/select）
2. 识别日期字段（可能叫"日期"/"Date"/"ddl"）
3. 识别负责人字段（可能叫"人员"/"负责人"）
4. 按规则筛选：排除未开始，包含完成和进行中
5. 判断"每日"任务和"素材"任务的特殊处理

### Step 4：生成每日清单

按日期分组，每天按固定人员顺序排列：
- Jiangran → Aurora → Yori → Jenny
- 每人的任务合并自所有 DB

### Step 5：写入页面

- 如果是**新建清单**：用 `replace_content` 或 `update_content` 写入
- 如果是**更新清单**：用 `update_content` 精确替换变更部分
- **必须保留**用户手动勾选的 `[x]` 和文字备注

### Step 6：验证

写入后 re-fetch 页面，确认：
- 所有 toggle `{toggle="true"}` 正常
- 缩进统一为双 tab
- 内容无遗漏

---

## 注意事项

1. **保留手动编辑**：用户可能手动勾选 checkbox 或添加备注，更新时必须保留这些改动
2. **缩进严格**：Notion enhanced markdown 对缩进敏感，toggle 内容必须用 tab 缩进，内容在 `###` 下用双 tab
3. **update_content 匹配**：使用 update_content 时，old_str 必须与页面内容完全匹配（包括缩进），否则会报错。如果匹配失败，改用 replace_content 重写整段
4. **空任务名跳过**：DB 中任务名为空的行跳过不处理
5. **日期范围**：只处理目标周内的任务（周一到周五），超出范围的忽略
6. **同一任务出现在多个 DB**：去重，保留信息最完整的版本

---

## ⛔ Toggle 丢失问题（严重，反复出现）

这是一个已经出现多次的严重 bug，必须在每次写入时严格防范。

### 问题描述

使用 `update_content` 时，如果 old_str 或 new_str 的边界落在 `### heading` 行上，但没有包含完整的 `{toggle="true"}` 属性，API 会做前缀匹配并**吞掉 `{toggle="true"}`**，导致：
- toggle 折叠功能消失
- 内容缩进从双 tab 降级为单 tab
- 页面结构被破坏

### 触发条件

```
old_str 结尾: "...\n\t### 5.28 周四"
实际页面内容: "...\n\t### 5.28 周四 {toggle="true"}"
→ API 匹配成功，但 {toggle="true"} 被吞掉
→ 5.28 的 toggle 丢失，内容缩进变成单 tab
```

### 强制规则（每次 update_content 必须检查）

1. **old_str 和 new_str 的边界绝不能落在 `###` heading 行的中间**
2. 如果必须包含 heading 行，**必须写完整**：`### 5.28 周四 {toggle="true"}`
3. **推荐做法**：old_str 结尾在 heading 行的**上一行**（如最后一个任务行），不要跨到下一天的 heading
4. **最安全做法**：如果要更新多天内容，直接用 `replace_content` 重写整个清单部分，而不是用 `update_content` 逐段修补
5. **写入后必须验证**：re-fetch 页面，逐一确认 5.25-5.29 每天的 heading 都有 `{toggle="true"}`

### 验证 Checklist

写入完成后，在 re-fetch 的结果中搜索以下内容，确保全部存在：
- `### 5.25 周一 {toggle="true"}`
- `### 5.26 周二 {toggle="true"}`
- `### 5.27 周三 {toggle="true"}`
- `### 5.28 周四 {toggle="true"}`
- `### 5.29 周五 {toggle="true"}`

如果任何一天缺少 `{toggle="true"}`，立即修复后再报告完成。
