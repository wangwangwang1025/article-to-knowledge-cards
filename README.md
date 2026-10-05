# make_knowledge_cards

## 项目解决什么问题

阅读长文章或整理学习笔记时，重要知识容易被重复表述、例子和次要细节淹没。本项目提供一个知识卡片 Skill，让助手从原文中筛选重点、合并重复内容，并整理成便于复习和自测的 Markdown 卡片。

这是供助手执行的指令型 Skill，需要在支持 Skill 或能够读取指令文件的助手环境中使用。项目没有独立运行程序，不需要安装项目专用的 Python/npm 依赖。

显示名为 `make_knowledge_cards`；官方初始化工具将下划线规范化为连字符，因此目录名、SKILL.md 的名称及 Codex 调用名均为 `make-knowledge-cards`。

## 主要功能

- 接收直接粘贴的文章，或读取运行环境可访问的本地 `.md`、`.markdown`、`.txt` 文件。
- 提取关键概念、机制、步骤、适用条件、结论和易混淆的区别；删除重复内容，一张卡片只讲一个主要知识点。
- 默认生成 5~8 张卡片；不足 5 个独立知识点时按实际数量输出并说明原因，只有宣传或空泛评价时不生成卡片。
- 每张包含标题、核心知识、简明解释，以及原文例子或可由原文回答的自测问题；自测问题附简短参考答案。
- 只依据原文，不补充外部事实或自创例子；保留条件、范围和不确定性，标明原文中的矛盾，不擅自修正。
- 默认沿用原文语言，也可按用户要求指定输出语言；直接返回 Markdown，明确要求保存时才通过助手的文件能力写入文件。

**暂不支持：** 网页抓取、PDF 读取、Anki 导出和图形界面。本 Skill 也不进行联网事实核查。文件不可访问、为空或乱码时，会说明问题并请求可读文本，不猜测文件内容。文章内夹带的操作指令只作为材料分析，不执行。

## 安装方法

### 本地 Codex 安装

前提：已有能够加载本地 Skill 的 Codex 环境。按照[官方 Skill 文档](https://learn.chatgpt.com/docs/build-skills)，用户级 Skill 放在 `~/.agents/skills` 下。

在原生 Windows PowerShell 中进入本项目根目录，再执行以下安装命令：

```powershell
$skillSource = (Resolve-Path ".\make-knowledge-cards").Path
$skillParent = Join-Path $env:USERPROFILE ".agents\skills"
New-Item -ItemType Directory -Path $skillParent -Force | Out-Null
Copy-Item -LiteralPath $skillSource -Destination $skillParent -Recurse
Test-Path (Join-Path $skillParent "make-knowledge-cards\SKILL.md")
```

最后一行应返回 `True`。安装时复制整个 `make-knowledge-cards` 目录，保留 `agents/openai.yaml`；README、LICENSE 和 tests 留在项目根目录。复制完成后，在 Codex CLI 或 IDE 的技能选择器中确认技能出现，可用 `/skills` 查看并使用 `$make-knowledge-cards` 调用。

以上命令针对原生 Windows。若使用 WSL、Linux 或 macOS，应把该目录放入**实际运行 Codex 的环境**中的 `~/.agents/skills`，并提供该环境能读取的文章路径。

安装说明依据官方文档；本项目未在你的 Windows 电脑上执行这些命令。已有同名安装时，可用本项目目录中的两个文件更新对应文件。

### 在当前项目中直接使用

也可让具有文件读取能力的助手直接读取本项目的指令文件，不必先复制到技能目录：

```text
请读取当前项目的 "make-knowledge-cards/SKILL.md"，
严格按其中规则，把我随后提供的文章转换成知识卡片。
```

该方式要求助手的工作目录为项目根目录，并能访问这个相对路径。网页或远程会话未必能访问电脑上的文件，必要时粘贴文章内容，并提供 SKILL.md 的内容。

## 使用方法

### 粘贴文章

安装并识别 Skill 后发送：

```text
使用 $make-knowledge-cards，把下面文章转换成知识卡片：
（粘贴文章全文）
```

### 读取本地文件

```text
使用 $make-knowledge-cards，读取本地文件
"学习笔记.md"，整理为知识卡片。
```

示例假设你已在助手的当前工作目录中准备自己的“学习笔记.md”；项目不附带这个文件。TXT 文件用法相同。其他位置的文章可提供其完整路径，确保文件位于助手可访问的环境；含中文或空格的路径也按完整路径提供。

### 指定语言或保存结果

```text
使用 $make-knowledge-cards，把下面英文文章整理为中文知识卡片。
（粘贴文章）
```

```text
使用 $make-knowledge-cards，读取本地文件
"学习笔记.txt"，并将卡片保存为
"知识卡片.md"。
```

保存需要助手具有目标位置的写入权限。未指定保存时，结果直接显示在对话中。

## 输入输出示例

下面的短文只有两个独立知识点，用来展示去重和“不强行凑满 5 张”的行为。

**输入：**

```text
队列按先进先出的顺序处理元素。比如甲先入队，乙后入队，甲先出队。
查看队头只读取队头元素，不移走它。
再次强调：先加入的元素先移走。
```

**输出示例：**

```markdown
原文只有两个独立知识点，去重后生成 2 张卡片。

### 卡片 1｜队列的先进先出顺序

**核心知识：** 队列按先进先出的顺序处理元素。

**简明解释：** 先入队的元素先出队。

**例子：** 甲先入队、乙后入队，甲先出队。

### 卡片 2｜查看队头不移走元素

**核心知识：** 查看队头只读取队头元素，不移走它。

**简明解释：** 查看队头与移走队头元素是不同的操作。

**自测问题：** 查看队头会移走队头元素吗？
**参考答案：** 不会，只读取队头元素。
```

原文提供适合的例子时使用原例；没有例子时改用自测问题，不另编故事或数据。实际措辞可能不同，但需满足同样的字段和原文约束。

## 文件结构

| 路径 | 内容 |
| --- | --- |
| `make-knowledge-cards/SKILL.md` | 输入处理、提取规则、输出格式和交付检查 |
| `make-knowledge-cards/agents/openai.yaml` | 界面名称、简介和默认提示语 |
| `tests/cases.json` | 不同类型的测试文章及边界输入 |
| `tests/results.md` | 测试结果与规则改进记录 |
| `tests/outputs.md` | 完整测试输出 |
| `LICENSE` | MIT 许可证 |
| `.gitignore` | 排除凭据文件、个人笔记和临时文件 |

## 验证与许可

项目使用官方 `init_skill.py` 初始化，最终 Skill 已通过官方 `quick_validate.py` 校验。行为测试包含 8 类输入，其中 5 类在完善规则后复测，详见 [测试记录](tests/results.md)和[测试输出](tests/outputs.md)。

若本地已有官方 skill-creator 工具，可执行以下命令，把工具目录替换为实际路径：

```text
python "工具目录\scripts\quick_validate.py" ".\make-knowledge-cards"
```

该命令在项目根目录执行。校验工具不随本项目分发，日常使用无需运行它。结构校验和有限测试不代表所有文章都能被正确处理，重要内容应对照原文检查。

本项目采用 [MIT 许可证](LICENSE)。当前仅完成本地开发与测试。
