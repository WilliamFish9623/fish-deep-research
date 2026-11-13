---
name: research-deep
description: 读取调研提纲，为每个调研对象派发独立检索子任务，逐项填充字段并输出结构化JSON。当用户要求「开始深挖」「按提纲逐项查」「接着上次没查完的继续」时使用。前置条件是当前目录已存在 outline.yaml 与 fields.yaml。
---

# Research Deep - 深度调研

## 能力要求

需要四类能力：文件读写、网页检索、命令行执行（运行字段校验脚本）、批量或并行子任务。没有并行能力时逐个串行，产物结构不变。子任务完成与否以输出文件是否存在判定，不依赖运行日志或任务面板。

## 何时使用

- 用户已确认 outline.yaml 与 fields.yaml，要求开始逐项调研
- 上一轮中断，需要续跑未完成的对象
- 只想补齐某几个对象的字段，而非重跑全部

不适用：提纲或字段尚未确定（先回初步调研阶段）；只需要一篇文献的精读（直接检索并阅读即可）。

## 执行流程

### Step 1: 自动定位Outline
在当前工作目录查找 `*/outline.yaml` 文件，读取items列表、execution配置（含items_per_agent）。

### Step 2: 断点续传检查
- 检查output_dir下已完成的JSON文件
- 跳过已完成的items

### Step 3: 分批执行
- 按batch_size分批（完成一批需要得到用户同意才可进行下一批）
- 每个子任务负责items_per_agent个项目
- 按下方提示词派发检索子任务（有并行能力则并行，否则串行）

**参数获取**：
- `{topic}`: outline.yaml中的topic字段
- `{item_name}`: item的name字段
- `{item_related_info}`: item的完整yaml内容（name + category + description等）
- `{output_dir}`: outline.yaml中execution.output_dir（默认./results）
- `{fields_path}`: {topic}/fields.yaml的绝对路径
- `{validator_path}`: 字段校验脚本路径，即本仓库 `skills/research-zh/research/validate_json.py` 的绝对路径；不便引用绝对路径时，复制到当前目录后用相对路径
- `{output_path}`: {output_dir}/{item_name_slug}.json的绝对路径（slugify处理item_name：空格替换为_，移除特殊字符）

**硬约束**：以下prompt必须严格复述，仅替换{xxx}中的变量，禁止改写结构或措辞。

**Prompt模板**：
```python
prompt = f"""## 任务
调研 {item_related_info}，输出结构化JSON到 {output_path}

## 字段定义
读取 {fields_path} 获取所有字段定义

## 输出要求
1. 按fields.yaml定义的字段输出JSON
2. 不确定的字段值标注[不确定]
3. JSON末尾添加uncertain数组，列出所有不确定的字段名
4. 所有字段值必须使用中文输出（调研过程可用英文，但最终JSON值为中文）

## 输出路径
{output_path}

## 验证
完成JSON输出后，运行验证脚本确保字段完整覆盖：
python {validator_path} -f {fields_path} -j {output_path}
验证通过后才算完成任务。
"""
```

**One-shot示例**（假设调研GitHub Copilot）：
```
## 任务
调研 name: GitHub Copilot
category: 国际产品
description: Microsoft/GitHub开发，首个主流AI编程助手，市场份额约40%，输出结构化JSON到 {project_dir}/results/GitHub_Copilot.json

## 字段定义
读取 {project_dir}/fields.yaml 获取所有字段定义

## 输出要求
1. 按fields.yaml定义的字段输出JSON
2. 不确定的字段值标注[不确定]
3. JSON末尾添加uncertain数组，列出所有不确定的字段名
4. 所有字段值必须使用中文输出（调研过程可用英文，但最终JSON值为中文）

## 输出路径
{project_dir}/results/GitHub_Copilot.json

## 验证
完成JSON输出后，运行验证脚本确保字段完整覆盖：
python {validator_path} -f {project_dir}/fields.yaml -j {project_dir}/results/GitHub_Copilot.json
验证通过后才算完成任务。
```

### Step 4: 等待与监控
- 等待当前批次完成
- 启动下一批
- 显示进度

### Step 5: 汇总报告
全部完成后输出：
- 完成数量
- 失败/不确定标记的items
- 输出目录

## 执行约定
- 执行方式: 并行或串行均可，结果等价
- 完成判定: 输出文件存在且通过字段校验
- 断点续传: 支持，按已存在的结果文件跳过
- 降级说明: 串行执行时在交付末尾注明「本次为串行执行，耗时较长，结果结构一致」
