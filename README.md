# 深度调研技能（fish-deep-research）

把一次调研拆成五步的可移植技能：立项、提纲与字段、逐项深挖、汇总、自检。目标是「不漏项、可追溯、能判断」，产物是带来源的 Markdown 报告。

本仓库源自 [Weizhena/Deep-Research-skills](https://github.com/Weizhena/Deep-Research-skills)，在其基础上做了四件事：去除平台绑定、合并为单入口、按中文政策研究场景本地化、补充语料配额与质量门禁。

## 与原项目的差异

| 维度 | 原项目 | 本仓库 |
|---|---|---|
| 入口 | 五个独立技能，靠斜杠命令串联 | 一个 `SKILL.md`，五阶段顺序执行 |
| 平台 | Claude Code、OpenCode、Codex 各一套 | 任意支持 Agent Skills 的平台 |
| 触发 | 斜杠命令 | 自然语言描述匹配 |
| 检索助手 | 平台代理配置文件 | 提示词模板，可内联或外挂 |
| 领域 | 技术选型、产品调研 | 医院管理、党建研究、政策梳理（可替换） |
| 证据规则 | 字段覆盖校验 | 字段校验 + 来源分层 + 五道质量门 |

## 目录结构

```
fish-deep-research/
├── SKILL.md                     唯一入口：五阶段、门禁、降级规则
├── references/
│   ├── 01-scope.md              立项与问题冻结
│   ├── 02-outline.md            对象与字段设计
│   ├── 03-deep.md               分批深挖与字段校验
│   ├── 04-report.md             汇总规则与报告结构
│   ├── 05-agent-prompt.md       检索子任务提示词
│   └── source-strategy/         四类来源的检索策略
├── scripts/validate_json.py     字段完整性校验
├── templates/                   提纲与字段的示例文件
├── tests/                       校验脚本回归测试
├── PORTABILITY-PLAN.md          改造清单与验收记录
└── CHANGELOG.md
```

## 安装

通用步骤有三步。

第一，把本目录放到平台的技能目录。技能入口是根目录的 `SKILL.md`，平台会读取其 `name` 与 `description` 做触发匹配。

第二，安装唯一依赖：

```bash
pip install pyyaml
```

第三，在对话中用自然语言触发，例如「帮我深度调研……，先列提纲，再逐项查」。

WorkBuddy 的做法是用目录联结，不复制副本，保证真身唯一。在命令提示符中执行：

```bat
mklink /J "%USERPROFILE%\.workbuddy\skills\fish-deep-research" "D:\agent-skills\fish-deep-research"
```

其他平台把目标路径换成各自的技能目录即可。Windows 上用目录联结而非符号链接，无需开发者模式或管理员权限。

## 能力要求与降级

| 能力 | 是否必需 | 缺失时的处理 |
|---|---|---|
| 文件读写 | 必需 | 无法运行，改用对话式调研 |
| 网页检索 | 必需 | 无法运行，改用已有材料分析 |
| 并行子任务 | 可选 | 串行执行，产物结构不变，交付时注明 |
| 命令行执行 | 可选 | 汇总阶段手工生成报告，注明「手工汇总」 |
| 文件读取策略模块 | 可选 | 把模块内容直接附在子任务提示词后 |

## 使用示例

「帮我深度调研上海市公立医院统战工作品牌的运行机制，先列提纲，再逐项查。」

「把这个方向近三年的政策、研究和案例系统梳理一遍，出一份带来源的报告。」

「按这份提纲继续查没完成的对象，然后汇总成一版材料。」

## 五阶段与产物

| 阶段 | 产物 | 完成条件 |
|---|---|---|
| 一 立项 | 一页立项简报 | 用户确认研究问题 |
| 二 提纲与字段 | `outline.yaml`、`fields.yaml` | 用户确认对象与字段 |
| 三 逐项深挖 | `results/*.json` | 填充率达标，字段校验通过 |
| 四 汇总 | `report.md` | 目录完整，无占位符 |
| 五 自检 | 自检记录 | 五道门全部通过 |

五道门是对象覆盖、字段填充、来源分层、引用可核验、判断可反驳，判定标准见 `SKILL.md`。

## 换领域怎么改

| 要改的地方 | 作用 |
|---|---|
| `references/01-scope.md` | 语料四级分层与配额，换成你的来源体系 |
| `references/02-outline.md` | 对象粒度与字段设计原则 |
| `templates/fields.example.yaml` | 字段清单与口径示例，直接照着改 |
| `references/source-strategy/` | 来源清单与检索式写法 |
| `scripts/validate_json.py` 的 `CATEGORY_MAPPING` | 分类中英对照，需与字段模板一致 |
| `references/04-report.md` | 报告结构与文风要求 |

## 校验与测试

```bash
pip install pyyaml
python tests/test_validate_json_schemas.py
python scripts/validate_json.py -f templates/fields.example.yaml -j results/示例.json
```

## 许可与致谢

MIT 许可，保留原作者版权声明。方法论源自 Weizhena 的 Deep-Research-skills，其设计受论文 [RhinoInsight: Improving Deep Research through Control Mechanisms for Model Behavior and Context](https://arxiv.org/abs/2511.18743) 启发。

改造记录见 `CHANGELOG.md`，逐步验收记录见 `PORTABILITY-PLAN.md`。
