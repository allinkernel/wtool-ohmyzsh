# AGENTS.md（shell/oh-my-zsh）

> 给后续的 AI 助手看。用户级规则在 `~/.dsh/AGENTS.md`，工作区规则在根目录 `AGENTS.md`；
> 本文件只讲**动这个仓库**必须知道的事。

## 1. README.md 是这个项目给用户的完整功能说明书

这个仓库的 `README.md` 结构特殊，**动之前先认清两层**：

1. **最前面那节「本项目说明（wtool 的 `shell/oh-my-zsh` 分支）」是我们要维护的** ——
   它是中文的、讲这个 fork 到底提供什么；
2. 后面接的是 **oh-my-zsh 上游的原始 README**（英文，按 wtool 口径删掉了
   "怎么装 / 怎么卸 / 高级安装"几节，其余保持原样）。**上游那部分不要手改**
   （除非是明确的错别字/链接坏了）—— 它将来要被上游版本替换。

所以：

- **代码/配置一有变化，必须同步更新 README 前面那节**。改了 `env.zsh`
  （`ZSH_THEME` / `plugins` / `ZSH` / 注释开关）、加载顺序、`omz` 子命令、
  lib 文件的作用、快捷键、更新机制，都要回到对应表格/条目改。
- **README 里不写"怎么装"** —— 安装统一由 wtool 管，只有一句话 + 指向 GitHub 上
  wtool README（`allinkernel/wtool` 仓库的 `README.md`）的链接。
  **上游那几节安装/卸载说明（`tools/install.sh`、`--unattended`、
  `git clone ~/.oh-my-zsh`、`uninstall_oh_my_zsh`）不许加回来** —— 它们是给
  "直接安装 oh-my-zsh"的用户的，本项目由 wtool 装、由 wtool 撤。
- **README 的章节结构**（"本项目说明"那节保持这个骨架）：

  | 章节 | 写什么 |
  |---|---|
  | 功能说明 | 提供什么（内容清单 + 加载顺序 + `lib/` 每个文件 + 插件/主题 + `omz` 命令 + `custom/` 覆盖规则 + 更新机制） |
  | 安装（由 wtool 统一管） | 一句话 + wtool README 链接；本仓库只是源码/配置 |
  | 配置项 | `env.zsh` 真正设了哪几个 + 上游注释开关表 + `zstyle ':omz:update'` |
  | 快捷键 | `lib/key-bindings.zsh` 里实际绑了的键（逐条对着文件核） |
  | 排错 | 报错原文（`[oh-my-zsh] theme '...' not found`、compaudit、`omz update` 走 master…） |
  | 测试 | 没有本地测试脚本；CI 的 `zsh -n` 语法检查，给等价命令 |
  | （上游原文） | 分隔线之后，保持上游内容不变 |

- **只写从代码里读出来的东西。** 数字（插件数、主题数、别名数）要对目录数过再写；
  函数个数这类容易被多处定义搞乱的数字，**宁可写"有哪些"而不写"有几个"**。
  核不实的宁可不写（README 是给用户的功能说明书，写错比不写更坏）。

## 2. 这个仓库的硬规矩

- **`ZSH` 只能来自 `$WTOOL_PROJECT_DIR`**（`env.zsh` 里 `export ZSH="$WTOOL_PROJECT_DIR"`），
  别改回 `~/.oh-my-zsh` 或自己算路径 —— 中转链接由 wtool 建，路径在任何机器上都对。
- **这个项目只有 `env.zsh`，没有 `env.bash`** —— 这是工作区里唯一的例外（oh-my-zsh 本身就是
  zsh 框架，bash 里没有对应物）。`wtool.xml` 里只有 `<zshrc>`。
- `priority=10`：**所有项目里最先加载**（主题/补全先就位），别随手改。
- **不要替用户设 `ZSH_DISABLE_COMPFIX=true`** —— 那会把 compaudit 的真实安全提醒永久静音；
  要解释就写进 README 的排错表，让人自己决定。
- 上游文件（`lib/`、`plugins/`、`themes/`、`oh-my-zsh.sh`、`tools/`、
  `templates/`）能不动就不动：跟上游同步靠 rebase/merge，属于单独处理的事项。
- `custom/`、`cache/`、`log/` 被上游 `.gitignore` 忽略（只留 `.gitkeep` / example），
  别把本地定制提交进去。
- `WTOOL.md` 是迁移记录，不是用户手册；改安装口径时它也要跟着改（它以前写着
  `./install.sh` / `./uninstall.sh`，那两个脚本已经不在仓库里了）。

## 3. 验证（改完必须做）

**本仓库没有本地测试脚本**；CI（`.github/workflows/main.yml`）跑的是**语法检查**：
对下面这些逐个 `zsh -n`，任何一个语法错就失败：

```sh
zsh -n env.zsh
for f in ./oh-my-zsh.sh ./lib/*.zsh ./plugins/*/*.plugin.zsh ./plugins/*/_* ./themes/*.zsh-theme; do
  zsh -n "$f" || echo "语法错: $f"
done
```

再冒烟看一眼真正加载的值（**要钉住 `WTOOL_PROJECT_DIR`**，否则机器上残留的旧值会让
`$ZSH` 指向别处）：

```sh
zsh -ic 'WTOOL_PROJECT_DIR=$PWD; source env.zsh; echo "ZSH=$ZSH"; echo "THEME=$ZSH_THEME"; echo "PLUGINS=$plugins"'
```

- 改 README 里的"提供什么"时，**顺手用 `ls` / `grep` 把数字和名字重新数一遍**，
  别让文档里的数字比目录旧的。
- 提交只提交到 `ds_dev`，`git add` 之前先 `git diff` 看一遍（这个仓库改动面大，
  尤其要确认没把上游那段英文 README 误删）；不 push、不动 `main`。
