# 变更记录

## 2026-11-13 v0.1.0

基于上游 Weizhena/Deep-Research-skills 提交 6ce38f6 改造，目标是去除平台绑定、合并单入口、按中文政策研究场景本地化。

### 删除

- `skills/research-en/`、`skills/research-codex-en/`、`skills/research-codex-zh/`：与中文版重复的平台版本
- `agents-codex/`、`agents/web-search-opencode.md`：Codex 与 OpenCode 专用检索助手
- `scripts/install-codex.sh`、`tests/test_install_codex.sh`：改写 Codex 配置的安装与测试脚本

### 去除平台绑定

- 删除 `user-invocable`、`allowed-tools`、`model: opus` 等平台专属字段
- 斜杠命令触发改为自然语言触发，写入各技能的 `description`
- `AskUserQuestion` 改为普通询问，保留无选项控件的回退
- 写死的 `~/.claude/...` 路径改为 `{validator_path}` 参数与相对路径
- 「启动 agent、禁用 task output」改为「派发子任务、以输出文件判定完成」

### 合并与迁移

- 五个技能合并为根级 `SKILL.md` 单入口，细节拆入 `references/01` 至 `05`
- `agents/web-search-agent.md` 改为 `references/05-agent-prompt.md`
- 五个检索模块合并为四个：`policy-statistics.md`、`academic-papers.md`、`general-web.md`、`qa-experience.md`
- `validate_json.py` 迁至 `scripts/`，测试同步改为新路径

### 本地化

- 分类映射由八个 AI 产品分类改为五类：基本信息、制度与依据、做法与机制、成效与证据、争议与局限
- 新增医院统战工作品牌场景的提纲与字段模板，共 11 个字段
- 新增政策与统计来源模块，写入四级来源分层与检索式写法
- 报告结构补充来源分层、跨对象比较、未获取项与文风要求

### 保留

- 五阶段流程、对象乘字段的矩阵设计
- 分批执行、断点续传、`[不确定]` 标注与 `uncertain` 数组
- 字段完整性校验脚本与回归测试
- MIT 许可与原作者版权声明
