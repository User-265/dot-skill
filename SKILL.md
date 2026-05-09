---
name: dot-skill
description: 统一的 meta-skill 引擎，把 colleague、relationship、celebrity 三类对象蒸馏成可复用 Skill。"
argument-hint: "[character] [name-or-slug]"
version: "1.0.0"
user-invocable: true
allowed-tools: Read, Write, Edit, Bash
---

# dot-skill 创建器

## 触发条件

当用户说类似以下内容时启动：
- "帮我创建一个 skill"
- "我想蒸馏一个人"
- "新建一个 skill"
- "给我做一个 XX 的 skill"

当用户对已有 Skill 说类似以下内容时，进入进化模式：
- "我有新文件" / "追加"
- "这不对" / "他不会这样" / "他应该是"

---

## 主流程：创建新 Skill

### Step 1：基础信息录入

阅读`intake.md`,根据intake规则与用户对话。如果用户直接提供原材料，跳转至Step 2

### Step 2：原材料导入

如果用户说"没有文件"或"跳过"，仅凭 Step 1 的手动信息生成 Skill。

### Step 3：分析原材料

首先读取`persona_analyzer.md``persona_builder.md``merger.md``work_analyzer.md``work_builder.md``correction_handler.md`

再按两条线分析：

**线路 A（Work Skill）**：
- 参考 `work_analyzer.md`
- 提取：负责系统、技术规范、工作流程、输出偏好、经验知识
- celebrity 场景下，`work` 更偏方法论、判断框架、决策习惯，不要机械套成“工作职责”

**线路 B（Persona）**：
- 参考 `persona_analyzer.md`
- 将用户填写的标签翻译为具体行为规则
- 从原材料中提取：表达风格、决策模式、人际行为
- celebrity 场景下，必须保留：
  - mental models
  - decision heuristics
  - expression DNA
  - contradictions
  - honest boundaries

### Step 4：生成并预览

使用 `work_builder.md` 生成 Work 内容。
使用 `persona_builder.md` 生成 Persona 内容。

向用户展示摘要（各 5-8 行），询问：
```
Work Skill 摘要：
  - 负责：{xxx}
  - 技术栈：{xxx}
  - CR 重点：{xxx}
  ...

Persona 摘要：
  - 核心性格：{xxx}
  - 表达风格：{xxx}
  - 决策模式：{xxx}
  ...

确认生成？还是需要调整？
```

### Step 5：写入文件

用户确认后，不要手工拼接 `skills/colleague/{slug}` 这类文件树。统一走 writer：

1. 先解析当前 storage root：
   - `colleague` → `./skills/colleague`
   - `relationship` → `./skills/relationship`
   - `celebrity` → `./skills/celebrity`
2. 用 `Write` 工具写三个临时文件：
   - `/tmp/dot_skill_{slug}_meta.json`
   - `/tmp/dot_skill_{slug}_work.md`
   - `/tmp/dot_skill_{slug}_persona.md`
3. `meta.json` 至少包含：
   - `name`
   - `display_name`
   - `character`
   - `research_profile`（当 character=`celebrity` 时必填）
   - `classification.language`（必须设置为用户当前语言，例如 `zh-CN` 或 `en`）
   - `profile`
   - `tags`
   - `knowledge_sources`
4. 然后调用：
   ```bash
   python3 tools/skill_writer.py \
     --action create \
     --character {character} \
     --research-profile {research_profile} \
     --slug {slug} \
     --name "{name}" \
     --meta /tmp/dot_skill_{slug}_meta.json \
     --work /tmp/dot_skill_{slug}_work.md \
     --persona /tmp/dot_skill_{slug}_persona.md \
     --base-dir {resolved_base_dir}
   ```
5. 该命令会统一生成：
   - `SKILL.md`
   - `work.md`
   - `persona.md`
   - `work_skill.md`
   - `persona_skill.md`
   - `manifest.json`
   - `meta.json`
   
---

## 进化模式：追加文件

用户提供新文件或文本时：

1. 按 Step 2 的方式读取新内容
2. 读取现有 `work.md` 和 `persona.md`
4. 使用 `merger.md` 分析增量内容

---

## 进化模式：对话纠正

用户表达"不对"/"应该是"时：

1. 参考 `correction_handler.md` 识别纠正内容
2. 判断属于 Work（技术/流程）还是 Persona（性格/沟通）
3. 如果属于 Work：
   - 生成 `/tmp/dot_skill_{slug}_work_patch.md`
   - patch 必须是可替换的 `##` section，不要直接手改最终文件
   - 调用：
     ```bash
     python3 tools/skill_writer.py \
       --action update \
       --character {character} \
       --slug {slug} \
       --work-patch /tmp/dot_skill_{slug}_work_patch.md \
       --base-dir {resolved_base_dir}
     ```
4. 如果属于 Persona：
   - 将 correction 写入 `/tmp/dot_skill_{slug}_correction.json`
   - 单条纠正可直接写成 `{scene, wrong, correct}`
   - 多条 persona 纠正可写成 `{"persona_corrections": [{...}, {...}]}`
   - 调用：
     ```bash
     python3 tools/skill_writer.py \
       --action update \
       --character {character} \
       --slug {slug} \
       --correction-json /tmp/dot_skill_{slug}_correction.json \
       --base-dir {resolved_base_dir}
     ```

6. 直接手改 `work.md`、`persona.md`、`SKILL.md`、`meta.json`
