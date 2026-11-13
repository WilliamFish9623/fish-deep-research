# 可移植化改造清单（PORTABILITY-PLAN）

本清单用于把 fish-deep-research 从「Claude Code / OpenCode / Codex 专用」改造为「任意支持 Agent Skills 的平台可用」。基线为上游 master 提交 6ce38f6，仓库共 44 个文件。

判断标准只有一条：这句话描述的是目的，还是手段。描述手段的（用哪个工具、放在哪个目录、敲哪个命令、用哪个模型）一律改；描述目的的（要查政策原文、要交叉验证、要标注不确定）一律留。

验收标准有三条：在 WorkBuddy 中可用自然语言触发并跑通；在没有子代理能力的平台可退化为串行执行；全仓库检索不到任何平台专属关键词与绝对路径。

## 一、现状盘点

| 部分 | 文件数 | 作用 | 平台绑定程度 |
|---|---|---|---|
| skills/research-zh | 6 | 中文主版本：五张工序卡与字段校验脚本，共 400 行 | 中，方法与内容可留，触发与工具申报需改 |
| skills/research-en | 6 | 英文版，同一套工序卡 | 高，整目录可删 |
| skills/research-codex-en | 6 | Codex 英文版 | 极高，整目录可删 |
| skills/research-codex-zh | 6 | Codex 中文版 | 极高，整目录可删 |
| agents | 7 | Claude 检索助手说明书与五个检索策略模块 | 高，说明书需改写，策略模块需保留 |
| agents-codex | 6 | Codex 检索助手配置，含 TOML 与模块副本 | 极高，整目录可删 |
| scripts/install-codex.sh | 1 | 自动改写 ~/.codex/config.toml | 极高，可删 |
| tests | 2 | 字段校验回归测试、Codex 安装测试 | 一半可留 |
| README.md / README.zh.md | 2 | 三平台安装说明 | 中，需重写安装章节 |
| workflow.png | 1 | 流程图 | 无，可留 |

## 二、阶段 A：删除（不触碰方法，最安全）

- [x] 删除 `skills/research-en/`，英文版工序卡，与中文版内容等价，保留会造成触发冲突
- [x] 删除 `skills/research-codex-en/`
- [x] 删除 `skills/research-codex-zh/`
- [x] 删除 `agents-codex/`，含 `web-researcher.toml` 与五份模块副本，仅 Codex 读取
- [x] 删除 `agents/web-search-opencode.md`，仅 OpenCode 读取
- [x] 删除 `scripts/install-codex.sh`，作用是往 Codex 配置写入 `multi_agent`、`suppress_unstable_features_warning` 等开关
- [x] 删除 `tests/test_install_codex.sh`，验证上述安装逻辑
- [x] 重写 README：合并为中英文单份中文说明，改为通用安装步骤与能力要求，删除平台专属安装段；`README.zh.md` 已合并删除

验收：删除后 `skills/` 下只剩 `research-zh` 一个目录，五张工序卡仍完整。

## 三、阶段 B：去暗号（逐文件逐行）

| 文件 | 行 | 现状 | 改法 |
|---|---|---|---|
| skills/research-zh/research/SKILL.md | 3 | `user-invocable: true` | 删除整行 |
| 同上 | 4 | `allowed-tools: Read, Write, Glob, WebSearch, Task, AskUserQuestion` | 删除，正文改为四类能力要求 |
| 同上 | 11 | 触发方式 `/research <topic>` | 改为「何时使用」自然语言描述 |
| 同上 | 20、25、111、120、121 | 使用 `AskUserQuestion` | 改为「向用户提一个确认问题」，保留纯对话回退 |
| 同上 | 143-145 | 后续命令三个斜杠命令 | 改为「下一阶段」与产物文件名 |
| research-deep/SKILL.md | 3、4、5 | `user-invocable`、`allowed-tools` | 删除 |
| 同上 | 11 | `/research-deep` | 改为阶段说明 |
| 同上 | 56、82 | `python ~/.claude/skills/research/validate_json.py` | 改为 `python {skill_dir}/scripts/validate_json.py` |
| 同上 | 99 | `Task Output: 禁用` | 改为「子任务完成以产物文件为准」 |
| research-report/SKILL.md | 3、4、5、11 | 同上四项 | 同上 |
| 同上 | 27 | `使用AskUserQuestion询问用户` | 改为自然语言询问 |
| research-add-items/SKILL.md | 3、4、5、11 | 同上四项 | 同上 |
| research-add-fields/SKILL.md | 3、4、5、11 | 同上四项 | 同上 |
| agents/web-search-agent.md | 4 | `model: opus` | 删除 |
| 同上 | 29 | 强制读取 `~/.claude/agents/web-search-modules/` | 改为相对路径 `references/source-strategy/`，并允许宿主无文件读取能力时内联 |
| tests/test_validate_json_schemas.py | 注释与路径 | 提到 `/research` 与 `skills/*/research/` | 改为通用描述与新路径 |

另需全局替换的三类措辞：

- [x] 「启动 web-search-agent（后台并行）」改为「若平台支持并行子任务，按下述提示词并行；否则串行执行，结果等价」
- [x] 「agent 配置：后台执行、Task Output、断点续传」改为「执行约定：产物落盘、按文件判完成、可断点续传」
- [x] 所有「禁用 task output」改为「不依赖运行日志，以输出文件是否存在判定完成」

验收：`grep -rIn -E "allowed-tools|user-invocable|AskUserQuestion|~/\\.claude|model: opus" skills agents tests` 无输出。

## 四、阶段 C：合并为单入口

- [x] 新建根级 `SKILL.md`，只写五件事：何时使用、五阶段流程、每阶段产物、质量门禁、能力降级规则
- [x] 新建 `references/01-scope.md`，收拢原 research 的立项与问题确认部分
- [x] 新建 `references/02-outline.md`，收拢 outline.yaml 与 fields.yaml 的设计规则
- [x] 新建 `references/03-deep.md`，收拢分批并行、断点续传、字段填充与不确定标注
- [x] 新建 `references/04-report.md`，收拢汇总规则、目录格式与不确定值跳过
- [x] 新建 `references/05-agent-prompt.md`，由 `agents/web-search-agent.md` 改写：删除模型指定与目录依赖，保留查询生成、来源优先级、质量自检
- [x] 将 `agents/web-search-modules/` 迁为 `references/source-strategy/`：保留 `academic-papers.md`、`general-web.md`；`chinese-tech.md` 改为「政策与统计来源」；`github-debug.md` 与 `stackoverflow.md` 合并为「问答与经验来源」
- [x] 将 `skills/research-zh/research/validate_json.py` 迁至 `scripts/validate_json.py`，路径改为相对调用
- [x] 新建 `templates/outline.example.yaml` 与 `templates/fields.example.yaml`，从原 SKILL.md 示例提取
- [x] 原五个技能目录归档删除，历史留在 Git 提交中

验收：新入口的 description 能覆盖「深度调研、综述、多对象对照、政策梳理、资料汇总」五类说法，且不与既有技能描述重复。

## 五、阶段 D：本地化

- [x] `scripts/validate_json.py` 的 `CATEGORY_MAPPING` 已由八个 AI 产品分类替换为五类：基本信息、制度与依据、做法与机制、成效与证据、争议与局限（随脚本迁移一并完成）
- [x] `references/04-report.md` 的分类双向映射已同步替换，与脚本一致
- [x] `templates/outline.example.yaml` 与 `fields.example.yaml` 已改为医院统战工作品牌场景，字段数 11，校验通过
- [x] 配额已写入 `references/source-strategy/policy-statistics.md`，其余三份模块补权重说明
- [x] `references/04-report.md` 新增「文风与格式」节，对齐直角引号、无加粗、段落论证、文末参考资料
- [x] 新建 `CHANGELOG.md`，按删除、去绑定、合并、本地化、保留五类记录

## 六、阶段 E：验收

验收时间 2026-11-13。测试题目为「公立医院高质量发展政策要求对照」，两个对象、六个字段，产物位于 `D://agent-skills//_e2e//fish-deep-research//2026-11-13-policy`。

- [x] 平台关键词清零：README、SKILL、references、scripts、tests 内无 claude、codex、opencode；仅 CHANGELOG 与本清单作为历史记录保留
- [x] 绝对路径清零：README、SKILL、references、scripts、tests 内无 `~/.claude`、`~/.codex`、`~/.config/opencode`；CHANGELOG 与本清单作为历史记录保留原文引用
- [x] 校验脚本可用：在演练目录执行 `python D:/agent-skills/fish-deep-research/scripts/validate_json.py -f fields.yaml -j results/*.json`，两个对象覆盖率均为 100%，额外字段「来源链接」被正确识别
- [x] 回归测试通过：六项全部 PASS
- [x] 流程演练通过：按五阶段手工执行，产出 outline.yaml、fields.yaml、两份 JSON、report.md；检索使用 DeepSeek，抓取两份政策原文，未获取项单列
- [x] 串行路径验证：本次为逐项串行执行，产物结构与并行设计一致
- [ ] 技能自动触发：本会话的技能清单在启动时加载，映射后需新会话验证；当前只验证了文件可读与流程可执行
- [ ] 触发不冲突：同上，需在新会话中与 thesis-writing-companion、deep-fact-check 同时验证

遗留两项。一是自动触发与冲突检测依赖新会话，不能在本会话内完成。二是 04 要求把 JSON 中的额外字段收入「其他信息」，本次测试脚本未实现该分支，属执行偏差，真实任务中需补上，技能文本本身无需改动。

## 七、明确保留（容易被误删）

- 五阶段先后逻辑：立项、补对象、补字段、逐项深挖、汇总
- outline 与 fields 的「对象乘字段」矩阵设计
- 分批执行、断点续传、`[不确定]` 标注与 `uncertain` 数组
- 检索策略模块中的来源分层与查询构造经验
- 字段完整性校验脚本与回归测试
- LICENSE 与原作者版权声明

## 八、目标结构

```
fish-deep-research/
├── SKILL.md                    唯一入口：五阶段、产物、门禁、降级规则
├── references/
│   ├── 01-scope.md
│   ├── 02-outline.md
│   ├── 03-deep.md
│   ├── 04-report.md
│   ├── 05-agent-prompt.md
│   └── source-strategy/
│       ├── policy-statistics.md
│       ├── academic-papers.md
│       ├── general-web.md
│       └── qa-experience.md
├── scripts/validate_json.py
├── templates/
│   ├── outline.example.yaml
│   └── fields.example.yaml
├── tests/test_validate_json_schemas.py
├── CHANGELOG.md
├── README.md
├── LICENSE
└── workflow.png
```

## 九、执行与回滚约定

分支为 `customize/portable`，`master` 保持与上游一致，便于一键同步。

提交分五次，每次可单独回滚：A 删除冗余平台版本；B 去除平台暗号；C 合并单入口；D 内容本地化；E 验收修正。

每次提交前报告文件清单、改法与提交信息，确认后执行。任何一步验收不通过，回到上一步修正，不带病推进。
