---
name: research
description: 对目标话题做初步调研，生成调研提纲与字段定义两个文件，供后续逐项深挖使用。当用户要求「系统调研一个话题」「先列个调研提纲」「梳理该看哪些对象和维度」「把调研范围定下来」时使用。不适用于只查一个事实、只读一篇文献、或已有现成字段表只需填表的情形。
---

# Research Skill - 初步调研

## 能力要求

需要四类能力：文件读写、目录检索、网页检索、向用户确认。可选第五类：并行子任务。没有并行能力时改为串行执行，产物结构不变，并在交付时说明降级原因。

## 何时使用

- 用户给出一个话题，要求先搭调研框架，再逐项查证
- 用户要把模糊的研究意图收敛成可清点的对象清单和字段清单
- 用户已有初稿提纲，需要补齐遗漏对象或维度

不适用：只查一个事实、只读一篇文献、已有完整字段表只差填表。后两种情形直接进入深挖或汇总阶段。

## 执行流程

### Step 1: 模型内部知识生成初步框架
基于topic，利用模型已有知识生成：
- 该领域的主要研究对象/items列表
- 建议的调研字段框架

输出{step1_output}，向用户确认（提一个带选项的问题，若无选项控件则用一句话提问并给出默认值）：
- items列表是否需要增减？
- 字段框架是否满足需求？

### Step 2: 联网补充
询问时间范围（如：最近6个月、2024年至今、不限）。默认不限，用户未回答时按不限执行。

**参数获取**：
- `{topic}`: 用户输入的调研话题
- `{YYYY-MM-DD}`: 当前日期
- `{step1_output}`: Step 1生成的完整输出内容
- `{time_range}`: 用户指定的时间范围
- `{step2_output}`: 检索子任务返回的补充结果

**硬约束**：以下prompt必须严格复述，仅替换{xxx}中的变量，禁止改写结构或措辞。

派发 1 个检索子任务（有并行能力则后台并行，否则串行），把下述提示词原样交给它：

```python
prompt = f"""## 任务
调研话题: {topic}
当前日期: {YYYY-MM-DD}

基于以下初步框架，补充最新items和推荐调研字段。

## 已有框架
{step1_output}

## 目标
1. 验证已有items是否遗漏重要对象
2. 根据遗漏对象进行补充items
3. 继续搜索{topic}相关且{time_range}内的items并补充
4. 补充新fields

## 输出要求
直接返回结构化结果（不写文件）：

### 补充Items
- item_name: 简要说明（为什么应该加入）
...

### 推荐补充字段
- field_name: 字段描述（为什么需要这个维度）
...

### 信息来源
- [来源1](url1)
- [来源2](url2)
"""
```

**One-shot示例**（假设调研AI Coding发展史）：
```
## 任务
调研话题: AI Coding 发展史
当前日期: 2025-12-30

基于以下初步框架，补充最新items和推荐调研字段。

## 已有框架
### Items列表
1. GitHub Copilot: Microsoft/GitHub开发，首个主流AI编程助手
2. Cursor: AI-first IDE，基于VSCode
...

### 字段框架
- 基本信息: name, release_date, company
- 技术特性: underlying_model, context_window
...

## 目标
1. 验证已有items是否遗漏重要对象
2. 根据遗漏对象进行补充items
3. 继续搜索AI Coding 发展史相关且2024年至今内的items并补充
4. 补充新fields

## 输出要求
直接返回结构化结果（不写文件）：

### 补充Items
- item_name: 简要说明（为什么应该加入）
...

### 推荐补充字段
- field_name: 字段描述（为什么需要这个维度）
...

### 信息来源
- [来源1](url1)
- [来源2](url2)
```

### Step 3: 询问用户已有字段
询问用户是否有已定义的字段文件，如有则读取并合并。

### Step 4: 生成Outline（分离文件）
合并{step1_output}、{step2_output}和用户已有字段，生成两个文件：

**outline.yaml**（items + 配置）：
- topic: 调研主题
- items: 调研对象列表
- execution:
  - batch_size: 并行子任务数量（需向用户确认）
  - items_per_agent: 每个子任务负责的调研项目数（需向用户确认）
  - output_dir: 结果输出目录（默认./results）

**fields.yaml**（字段定义）：
- 字段分类和定义
- 每个字段的name、description、detail_level
- detail_level分层：极简 → 简要 → 详细
- uncertain: 不确定字段列表（保留字段，deep阶段自动填充）

### Step 5: 输出并确认
- 创建目录: `./{topic_slug}/`
- 保存: `outline.yaml` 和 `fields.yaml`
- 展示给用户确认

## 输出路径
```
{当前工作目录}/{topic_slug}/
  ├── outline.yaml    # items列表 + execution配置
  └── fields.yaml     # 字段定义
```

## 后续阶段
- 补充调研对象：使用 research-add-items 技能
- 补充调研字段：使用 research-add-fields 技能
- 开始逐项深挖：使用 research-deep 技能
- 汇总成报告：使用 research-report 技能
