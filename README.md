# 🧬 dot-skill（同事.skill）

## 🚀 使用

在你装了 dot-skill 的宿主里启动它 —— 输入 `/dot-skill`，或直接和你的 Agent 说「启动 dot-skill」。

启动后会先让你选择蒸馏类型：`colleague` · `relationship` · `celebrity`。

然后按提示输入花名、基础信息、性格标签，再选择数据来源。所有字段均可跳过，仅凭描述也能生成。

完成后用 `/{character}-{slug}` 调用生成好的 Skill。

### 🎛️ 管理命令

| 命令 | 说明 |
|------|------|
| `/dot-skill` | 统一主入口 |
| `/{character}-{slug}` | 调用完整 Skill（Persona + Work） |
| `/{character}-{slug}-work` | 仅工作能力 |
| `/{character}-{slug}-persona` | 仅人物性格 |
| `python3 tools/skill_writer.py --action list ...` | 列出三类 Skill |
| `python3 tools/version_manager.py --action rollback ...` | 回滚历史版本 |

### 🔬 名人研究工具链

`celebrity` 类型内置了一套研究工具链，从字幕到成品一条龙：

```bash
# 下载视频字幕
bash tools/research/download_subtitles.sh "<video-url>" "./tmp/subtitles"

# 字幕转文稿
python3 tools/research/srt_to_transcript.py "./tmp/subtitles/example.srt"

# 合并研究笔记
python3 tools/research/merge_research.py "./skills/celebrity/<slug>"

# 质量检查
python3 tools/research/quality_check.py "./skills/celebrity/<slug>/SKILL.md"
```

---

## 🔧 功能特性

### 🧱 生成的 Skill 结构

dot-skill 以 **Persona** 为通用底座，不同家族按场景挂载各自的模块：

| 家族 | Persona 内容 | 附加模块 |
|------|-------------|---------|
| 🧑‍💼 **colleague** | 6 层性格结构：硬规则 → 身份 → 表达风格 → 决策模式 → 人际行为 → Correction | ➕ **Work Skill**：负责范围、工作流程、输出偏好、经验知识库 |
| 💞 **relationship** | 表达 DNA · 情绪触发点 · 冲突模式 · 修复模式 | — |
| 🌟 **celebrity** | 心智模型 · 决策启发式 · 表达 DNA · 外部评价对照 | ➕ 六维度 research 档案（著作 / 访谈 / 决策 / 时间线...） |

> **运行逻辑**：接到任务 → Persona 判断态度与语气 → 附加模块补齐执行细节 → 用他的方式输出

### 🧬 进化机制

- 📥 **追加文件** → 自动分析增量 → merge 进对应部分，不覆盖已有结论
- 💬 **对话纠正** → 说「他不会这样，他应该是 xxx」→ 写入 Correction 层，立即生效
- 🕰️ **版本管理** → 每次更新自动存档，支持回滚到任意历史版本
- 🔬 **名人研究管线** → 字幕下载 → 文稿清洗 → 六维度研究 → 质量检查

---

## 📂 项目结构

本项目遵循 [AgentSkills](https://agentskills.io) 开放标准，整个 repo 就是一个 skill 目录：

```
dot-skill/
├── SKILL.md                        # skill 入口（官方 frontmatter）
├── prompts/                        # 三大家族的 Prompt 体系
│   ├── intake.md                   #   [colleague] 信息录入
│   ├── work_analyzer.md            #   [colleague] 工作能力提取
│   ├── persona_analyzer.md         #   [colleague] 性格行为提取
│   ├── work_builder.md             #   [colleague] work.md 生成
│   ├── persona_builder.md          #   [colleague] persona.md 六层结构
│   ├── merger.md                   #   [共享] 增量 merge 逻辑
│   ├── correction_handler.md       #   [共享] 对话纠正处理
│   ├── relationship/               #   [relationship] 情感/冲突/修复模式专属 prompt
│   └── celebrity/                  #   [celebrity] 六维度研究 + 心智模型专属 prompt
├── tools/                          # Python 工具
│   ├── feishu_auto_collector.py    #   [colleague] 飞书全自动采集
│   ├── dingtalk_auto_collector.py  #   [colleague] 钉钉全自动采集
│   ├── slack_auto_collector.py     #   [colleague] Slack 全自动采集
│   ├── email_parser.py             #   [共享] 邮件解析
│   ├── research/                   #   [celebrity] 名人研究工具链
│   │   ├── download_subtitles.sh   #     字幕下载
│   │   ├── transcribe_audio.py     #     音频转文字
│   │   ├── srt_to_transcript.py    #     字幕转文稿
│   │   ├── merge_research.py       #     六维度 research 合并
│   │   └── quality_check.py        #     质量检查
│   ├── install_*_skill.py          #   [共享] 多宿主一键安装器
│   ├── skill_writer.py             #   [共享] Skill 文件管理
│   └── version_manager.py          #   [共享] 版本存档与回滚
├── skills/                         # 生成的 Skill（gitignored）
│   ├── colleague/                  #   同事
│   ├── relationship/               #   亲近关系
│   └── celebrity/                  #   名人 / 公众人物
├── docs/PRD.md
├── requirements.txt
└── LICENSE
```

---

## ⚠️ 注意事项

**原材料质量决定 Skill 质量**，不同家族的优质信源不一样：

| 家族 | 信源优先级（高 → 低） |
|------|----------------------|
| 🧑‍💼 **colleague** | 他**主动写的**长文（设计文档 / 评审意见） **›** **决策类回复** **›** 日常群聊消息 |
| 💞 **relationship** | 完整的聊天记录 **›** 往来信件 / 朋友圈 / 日记 **›** 旁人描述 |
| 🌟 **celebrity** | 第一人称著作 / 博客 / 长访谈 **›** 决策记录（发布会、commit、采访）**›** 他人评价 |

- **colleague** 飞书自动采集：需将 App bot 加入相关群聊
- **relationship**：时间跨度越长越好，能覆盖冲突与和解更佳
- **celebrity**：避免只喂二手解读
- 目前还是 demo 版本，如果有 bug 请多多提 issue！

---
