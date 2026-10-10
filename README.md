# IDEA Skills

通过 IntelliJ IDEA MCP 为 AI 编程助手提供源码索引、数据库操作与 JVM 调试能力。三个技能集中在同一个仓库维护，可直接使用 [Skills CLI](https://github.com/vercel-labs/skills) 安装；无需发布 npm 包。

## 包含的技能

| 技能 | 能力 |
| --- | --- |
| [idea](skills/idea/SKILL.md) | 搜索项目、依赖和 JDK 符号，读取源码或反编译内容，分析语义调用关系，执行重命名重构。 |
| [idea-database](skills/idea-database/SKILL.md) | 使用 IDEA 已配置的数据源检查连接、数据库结构、表数据和 SQL 查询。默认优先只读操作。 |
| [idea-debug](skills/idea-debug/SKILL.md) | 使用 JVM 调试器收集运行时证据，优先使用不暂停线程的日志断点，定位根因。 |

## 安装

需要 Node.js / npm 提供 `npx`。通用安装命令如下，按 CLI 提示选择你实际使用的助手：

```bash
npx skills add zz-zhi54/idea-skills -g --skill '*'
```

### `-a codex` 到底安装到哪里？

**本仓库命令中的 `-g -a codex`，会把三个技能安装到当前用户主目录下的 `~/.agents/skills/`，不是 `~/.codex/skills/`，也不是本仓库目录。**

执行：

```bash
npx skills add zz-zhi54/idea-skills -g --skill '*' -a codex -y
```

安装后的目录是：

```text
~/.agents/skills/
├── idea/
│   ├── SKILL.md
│   └── scripts/idea.py
├── idea-database/
│   ├── SKILL.md
│   └── scripts/idea-database.py
└── idea-debug/
    ├── SKILL.md
    └── scripts/idea-debug.py
```

`~` 表示当前用户的主目录。在本次安装的机器上，完整路径分别是：

```text
/Users/zz_zhi/.agents/skills/idea/
/Users/zz_zhi/.agents/skills/idea-database/
/Users/zz_zhi/.agents/skills/idea-debug/
```

**安装位置由 `-g` 和 `-a codex` 共同决定：**

| 参数组合 | 安装位置 |
| --- | --- |
| `-g -a codex` | 用户主目录下的 `~/.agents/skills/<技能名>/`，可跨项目使用 |
| `-a codex`，不加 `-g` | 执行命令时当前目录下的 `.agents/skills/<技能名>/`，用于该项目 |

例如，在 `/path/to/my-project` 中执行不带 `-g` 的安装命令，会安装到 `/path/to/my-project/.agents/skills/`。

`-a`（`--agent`）只选择助手对应的安装目录，`-g` 再选择用户级而非项目级范围。**`-a codex` 不会启动 Codex，不会切换模型，也不会限制技能只能由 Codex 使用。** 同样，省略 `-a` 不表示自动安装到所有助手；CLI 会检测助手并让你选择。

| 实际使用环境 | 安装参数 | 用户级技能目录 |
| --- | --- | --- |
| 本项目当前使用的 DeepSeek Harness（DSH） | `-a codex`，借用 CLI 的共享目录安装目标 | `~/.agents/skills/` |
| Codex | `-a codex` | `~/.agents/skills/` |
| Claude Code | `-a claude-code` | `~/.claude/skills/` |
| Cursor | `-a cursor` | `~/.cursor/skills/` |

本次迁移中，**实际读取和使用三个技能的是 DSH，不是 Codex**。DSH 当前会话从 `~/.agents/skills/` 加载技能，因此使用 `-a codex` 将文件安装到这个共享目录。这里的 `codex` 只是 Skills CLI 的安装目标名称，不要求安装或运行 Codex；也不是 `-a dsh` 的别名。其他助手能否使用，取决于它们是否读取相应目录并支持技能执行。

当前 DSH 环境的确定性安装命令：

```bash
npx skills add zz-zhi54/idea-skills -g --skill '*' -a codex -y
```

参数含义：`-g -a codex` 将技能安装到 `~/.agents/skills/`；`--skill '*'` 选择仓库内全部技能；`-y` 跳过安装确认。

技能被触发后的实际调用链是：**DSH 中的 AI 助手 → 技能内 Python 包装脚本 → mcpc → IntelliJ IDEA MCP**。Skills CLI 只负责安装和更新，不参与这条运行链。

查看仓库中的技能（不安装）：

```bash
npx skills add zz-zhi54/idea-skills --list
```

当前 DSH 环境中只安装一个技能：

```bash
npx skills add zz-zhi54/idea-skills -g --skill idea-debug -a codex -y
```

去掉 `-g` 可安装到当前项目。安装会替换同名技能；迁移前请先保存未发布的本地改动。

## 运行前提

- Python 3，命令示例中的 `python` 应指向 Python 3；若本机只有 `python3`，使用 `python3`。
- 已安装并可从 `PATH` 调用的 [apify/mcpc](https://github.com/apify/mcpc)。包装脚本不会自动安装它。
- 在 IntelliJ IDEA 中启用并配置 MCP 服务；服务必须提供对应技能所需的工具。工具是否可用以及精确参数，以当前服务的工具定义为准。
- 当前脚本使用固定地址 `http://127.0.0.1:64342/stream`。请确认本机 IDEA MCP 使用此地址；若端口或路径不同，需修改三个包装脚本中的 `ENDPOINT` 后再使用。本仓库没有新增地址配置机制。
- 数据库技能使用 IDEA 中已配置的数据源；调试技能需要可运行的 JVM 项目以及服务提供的 debugger / run-configuration 工具。

IDEA MCP 配置可参考 [JetBrains 官方文档](https://www.jetbrains.com/help/idea/mcp-server.html)。安装技能本身不代表本机已经具备全部 MCP 工具。

## 使用

让助手加载对应技能，再通过技能内的 Python 包装脚本调用工具。**工作目录应位于当前处理的项目，不要切换到已安装技能目录。**

下面是安装到 DSH / Codex 共享目录后的示例，请在目标项目中执行；若安装给其他助手，改用其实际技能目录：

```bash
python3 "$HOME/.agents/skills/idea/scripts/idea.py" search_symbol '{"q":"OrderService","limit":20}'
python3 "$HOME/.agents/skills/idea-database/scripts/idea-database.py" list_database_connections
python3 "$HOME/.agents/skills/idea-debug/scripts/idea-debug.py" xdebug_get_debugger_status
```

查看实时工具定义：

```bash
python3 "$HOME/.agents/skills/idea/scripts/idea.py" search_symbol --help
```

三个包装脚本会向上查找最近的 `.git` 目录或文件作为项目根目录，未找到时使用当前工作目录；通过 `IJ_MCP_SERVER_PROJECT_PATH` 绑定该项目。mcpc 状态保存在 `<项目根目录>/.dsh/mcpc`，三个技能共用会话 `@idea`。不要提交该运行时目录，其中可能包含会话信息。

重命名重构、修改 SQL、数据库连接修改以及调试器变量修改会影响项目或运行状态，只应在明确要求时使用；具体安全规则见各技能说明。

## 后续维护与更新

仓库中的 `skills/` 是唯一维护源。不要直接修改 `~/.agents/skills/` 中的安装副本，否则下一次安装或更新可能覆盖修改。

1. 修改仓库中对应技能的 `SKILL.md` 或包装脚本。
2. 检查修改，提交并推送到 GitHub：

   ```bash
   git add skills
   git commit -m "docs: update IDEA skills"
   git push origin main
   ```

3. 在使用技能的机器上更新这三个全局技能：

   ```bash
   npx skills update idea idea-database idea-debug -g -y
   ```

当前 DSH 环境也可以重新运行以下命令，从仓库覆盖安装全部三个技能；其他助手请换成对应的 `-a` 参数：

```bash
npx skills add zz-zhi54/idea-skills -g --skill '*' -a codex -y
```

`git push` 不会自动更新其他机器上的安装副本；每台使用机器都需要执行更新或重新安装。若助手缓存了技能内容，更新后重新加载技能或新建会话。

## 目录结构

```text
skills/
├── idea/
│   ├── SKILL.md
│   └── scripts/idea.py
├── idea-database/
│   ├── SKILL.md
│   └── scripts/idea-database.py
└── idea-debug/
    ├── SKILL.md
    └── scripts/idea-debug.py
```

三个技能各自包含完整包装脚本，因此可独立安装，不依赖仓库中未安装的共享文件。

## 许可证

[MIT](LICENSE)。本项目不是 JetBrains 官方项目。
