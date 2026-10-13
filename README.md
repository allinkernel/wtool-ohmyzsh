<p align="center"><img src="https://ohmyzsh.s3.amazonaws.com/omz-ansi-github.png" alt="Oh My Zsh"></p>

Oh My Zsh is an open source, community-driven framework for managing your [zsh](https://www.zsh.org/) configuration.

Sounds boring. Let's try again.

**Oh My Zsh will not make you a 10x developer...but you may feel like one.**

Once installed, your terminal shell will become the talk of the town _or your money back!_ With each keystroke in your command prompt, you'll take advantage of the hundreds of powerful plugins and beautiful themes. Strangers will come up to you in cafés and ask you, _"that is amazing! are you some sort of genius?"_

Finally, you'll begin to get the sort of attention that you have always felt you deserved. ...or maybe you'll use the time that you're saving to start flossing more often. 😬

To learn more, visit [ohmyz.sh](https://ohmyz.sh), follow [@ohmyzsh](https://x.com/ohmyzsh) on X (formerly Twitter), and join us on [Discord](https://discord.gg/ohmyzsh).

[![CI](https://github.com/ohmyzsh/ohmyzsh/workflows/CI/badge.svg)](https://github.com/ohmyzsh/ohmyzsh/actions?query=workflow%3ACI)
[![X (formerly Twitter) Follow](https://img.shields.io/twitter/follow/ohmyzsh?label=%40ohmyzsh&logo=x&style=flat)](https://twitter.com/intent/follow?screen_name=ohmyzsh)
[![Mastodon Follow](https://img.shields.io/mastodon/follow/111169632522566717?label=%40ohmyzsh&domain=https%3A%2F%2Fmstdn.social&logo=mastodon&style=flat)](https://mstdn.social/@ohmyzsh)
[![Discord server](https://img.shields.io/discord/642496866407284746)](https://discord.gg/ohmyzsh)
[![Gitpod ready](https://img.shields.io/badge/Gitpod-ready-blue?logo=gitpod)](https://gitpod.io/#https://github.com/ohmyzsh/ohmyzsh)

---

# 本项目说明（wtool 的 `shell/oh-my-zsh` 分支）

> 这是 **oh-my-zsh 的个人 fork**（原 `mytool/zsh/oh-my-zsh`），现在作为 wtool 的独立项目
> `shell/oh-my-zsh`（`priority=10`，所有项目里最先加载）。
> 再往下是 **oh-my-zsh 上游的原始 README**（按 wtool 的口径删掉了"怎么装 / 怎么卸"的几节，
> 其余未改动）；迁移细节另见 [`WTOOL.md`](WTOOL.md)。

## 功能说明

### 1. 这个 fork 到底提供什么

上游 oh-my-zsh 的**全部内容**都在这个仓库里，一个文件没少（都对着目录数过）：

| 内容 | 数量 | 位置 |
|---|---|---|
| 插件目录 | 337 | `plugins/` |
| 主题 | 143（`*.zsh-theme`） | `themes/` |
| 库文件 | 21（`*.zsh`） | `lib/` |
| 框架本体 | 1 | `oh-my-zsh.sh` |
| 新建 `~/.zshrc` 用的模板 | 1 | `templates/zshrc.zsh-template` |

这个 fork 与上游的差别只有"**怎么被加载**"：`env.zsh`（原 `wsw.zsh`）里不再自己猜路径，
而是用 wtool 导出的 `ZSH="$WTOOL_PROJECT_DIR"`，然后 `source $ZSH/oh-my-zsh.sh`；
同时钉住 **主题 `ys`**、**插件 `plugins=(git)`**。`WTOOL.md` 里有逐条对照。

### 2. 启动时按什么顺序加载（读 `oh-my-zsh.sh`）

1. 检查是不是真的 zsh（`$ZSH_VERSION`、`emulate` 必须是 `zsh`，否则报错返回 1）；
2. `ZSH_CUSTOM`（默认 `$ZSH/custom`）、`ZSH_CACHE_DIR`（默认 `$ZSH/cache`，
   不可写就退到 `${XDG_CACHE_HOME:-$HOME/.cache}/oh-my-zsh`）就位；
3. `source $ZSH/tools/check_for_upgrade.sh` —— 更新检查（见下面第 8 条）；
4. `fpath` 加上 `$ZSH/{functions,completions}` 和 `$ZSH_CUSTOM/{functions,completions}`，
   然后 `compinit`（补全；zcompdump 里记着 `$ZSH` 的 git revision，revision 变了会自动重建）；
5. **逐个 source `$ZSH/lib/*.zsh`**（`_omz_source`：同名文件在 `$ZSH_CUSTOM` 里存在就用 custom 的）；
6. **source `plugins=(...)` 里每个插件**（同样是 custom 优先）；当前只有 `git`；
7. source `$ZSH_CUSTOM/*.zsh`（你自己往 `custom/` 丢的散装配置）；
8. 加载主题：先在 `$ZSH_CUSTOM`、再 `$ZSH_CUSTOM/themes`、最后 `$ZSH/themes` 里找
   `<ZSH_THEME>.zsh-theme`；找不到打印
   `[oh-my-zsh] theme '<名字>' not found`。

### 3. `lib/` 这 21 个文件各给什么

| 文件 | 给什么 |
|---|---|
| `async_prompt.zsh` | 异步取 prompt 信息（代码取自 zsh-autosuggestions / git-prompt.zsh） |
| `bzr.zsh` | `bzr_prompt_info`（Bazaar） |
| `cli.zsh` | **`omz` 命令本体**（下一节单独讲） |
| `clipboard.zsh` | 跨平台系统剪贴板 copy/paste 助手 |
| `compfix.zsh` | `compaudit`：补全目录属主/权限不安全时给出人话警告 |
| `completion.zsh` | 补全系统默认设置（关闭 `menu_complete`、`WORDCHARS` 等） |
| `correction.zsh` | `ENABLE_CORRECTION=true` 时给 `cp` / `man` / `mkdir` 等加 `nocorrect` |
| `diagnostics.zsh` | 5 个诊断函数（`omz_diagnostic_dump` 等） |
| `directories.zsh` | 目录相关的 `setopt`（`auto_cd` / `auto_pushd` / `pushd_ignore_dups`）+ 别名（见下面） |
| `functions.zsh` | 通用函数（**15 个名字**，都对着文件列过）：`zsh_stats`、`uninstall_oh_my_zsh`、`upgrade_oh_my_zsh`、`open_command`、`mkcd` / `takedir`（同一行定义的两个名字）、`takeurl`、`takegit`、`take`、`alias_value`、`try_alias_value`、`default`、`env_default`、`omz_urlencode`、`omz_urldecode` |
| `git.zsh` | prompt 用的 git 信息函数：`git_prompt_info` / `git_prompt_status`、`parse_git_dirty`、`git_current_branch`、`git_current_user_name`、`git_commits_ahead` / `git_commits_behind` … |
| `grep.zsh` | 给 `grep` 上色（结果缓存到 `$ZSH_CACHE_DIR/grep-alias`） |
| `history.zsh` | 历史相关设置 + `omz_history` |
| `key-bindings.zsh` | 按键绑定（见「快捷键」） |
| `misc.zsh` | 杂项 `setopt`；`DISABLE_MAGIC_FUNCTIONS=true` 可关掉 `*-magic` |
| `nvm.zsh` | `nvm_prompt_info` |
| `prompt_info_functions.zsh` | 给主题用的 `*_prompt_info` 占位实现 |
| `spectrum.zsh` | 256 色助手（`FX` / `FG` / `BG` 关联数组） |
| `termsupport.zsh` | 终端窗口/标签标题 |
| `theme-and-appearance.zsh` | 颜色变量、`ls` 颜色、`setopt prompt_subst` |
| `vcs_info.zsh` | `vcs_info` 集成 |

`lib/directories.zsh` 还带来这些**直接可用的命令**（都对着文件核过）：
`d`（列出最近去过的目录），**global alias** `...` / `....` / `.....` / `......`
（分别展开成 `../..` … `../../../../..`），别名 `-`（`cd -`）、`1`…`9`（`cd -N`）、
`md`（`mkdir -p`）、`rd`（`rmdir`）、`lsa` / `l`（`ls -lah`）、`ll`（`ls -lh`）、`la`（`ls -lAh`）。
（注意**没有** `..` 这个别名，只有从 `...` 起的。）

### 4. 插件：当前只启用 `git`

`env.zsh` 里写的是 `plugins=(git)`，所以启动时只加载 `plugins/git/git.plugin.zsh`。
这个插件自己就提供 **195 个别名**（`g`=`git`、`ga`=`git add`、`gaa`=`git add --all`、
`gst`/`gd`/`gl`/`gp`/`gr`… 全表见 `plugins/git/git.plugin.zsh`）。

仓库里另外 336 个插件都**没启用**；`omz` 命令能查和改：

```sh
omz plugin list              # 列出所有可用插件
omz plugin info <plugin>     # 看某个插件的信息
omz plugin enable <plugin>   # 启用（它写的是 ~/.zshrc，见「排错」）
omz plugin disable <plugin>  # 停用
```

### 5. 主题：当前用 `ys`

`env.zsh` 里 `ZSH_THEME="ys"`，对应 `themes/ys.zsh-theme`；
`ZSH_THEME="strug"` 那行是注释掉的（历史上用过的主题）。

```sh
omz theme list               # 列出全部 143 个主题
omz theme use <theme>        # 只在当前 shell 里换成这个主题
omz theme set <theme>        # 改 ~/.zshrc 里的 ZSH_THEME（见「排错」）
```

### 6. `omz` 命令（`lib/cli.zsh`）

| 子命令 | 作用 |
|---|---|
| `omz version` | 版本（`$ZSH` 的 git 描述） |
| `omz update` | 更新 oh-my-zsh（跑 `tools/upgrade.sh`，见「排错」） |
| `omz reload` | 重新加载 zsh 配置 |
| `omz plugin {list,info,enable,disable}` | 插件管理 |
| `omz theme {list,set,use}` | 主题管理 |
| `omz changelog` | 看更新日志 |
| `omz pr <编号>` | 把某个 PR 拉到本地试用 |
| `omz help` | 帮助 |

（顶层 `omz` 的分派规则是"看 `_omz::<子命令>` 这个函数在不在"；`lib/cli.zsh` 里一共
定义了 10 个这样的函数 —— 上面 8 个之外，`_omz::confirm` / `_omz::log` 是内部辅助，
不是给用户敲的。认不出的子命令会打帮助并返回 1。）

### 7. 覆盖上游：`custom/`

`$ZSH_CUSTOM` 默认 `$ZSH/custom`，仓库里带了示例文件
（`custom/example.zsh`、`custom/plugins/example/example.plugin.zsh`、
`custom/themes/example.zsh-theme`）。规则（`_omz_source` / 加载顺序里读出来的）：

- `custom/*.zsh` 在 lib 和插件之后 source —— 用来覆盖别名/函数；
- `custom/lib/<文件>`、`custom/plugins/<插件>/<文件>` 同名时**优先于上游那份**；
- 主题在 `custom/` 和 `custom/themes/` 里也找。

> `custom/`、`cache/`、`log/` 被上游 `.gitignore` 忽略（只留 `.gitkeep` / example），
> 所以你自己丢进去的东西不会被提交。

### 8. 更新机制（上游的）

`tools/check_for_upgrade.sh` 在每次启动时按 `zstyle ':omz:update' …` 决定要不要检查更新，
默认**每 13 天**提示一次（`zstyle -s ':omz:update' frequency epoch_target || epoch_target=${UPDATE_ZSH_DAYS:-13}`，
也就是常说的"约两周"）；`mode` 可选 `auto` / `reminder` / `disabled`，
`frequency` 改间隔天数，`verbose` 改提示详细程度，`omz update` 手动更新。

> ⚠️ 这套更新脚本是**上游**的：它默认 checkout `master` 分支再 `git pull --rebase`
> （`tools/upgrade.sh` 里 `branch=${"$(git config --local oh-my-zsh.branch)":-master}`，
> 然后 `git checkout -q "$branch" -- || exit 1`）。本仓库是 fork、真源分支是 `main`，
> **没有 `master`**，所以这条命令在这个仓库上跑不起来（见「排错」）。

---

## 安装（由 wtool 统一管）

安装由 wtool 统一管：见 [wtool 的 README（GitHub：allinkernel/wtool-doc）](https://github.com/allinkernel/wtool-doc/blob/main/README.md) ——
本仓库只是源码/配置，装的时候是 `wtool install shell/oh-my-zsh`（**项目路径就是它的身份** ——
没有单独的 id，见 ADR-0037）。

> ⚠️ **wtool 的项目只在容器里装 / 测**（用户级规矩，2026-10-04）：本机（WSL）是临时
> 的手工环境，wtool 彻底调通之前**不在本地落地**。要在容器里验证就 `--network=host`
> 挂工作区；**真机上装本项目必须由用户明确同意**，助手不得自行 `wtool install`。

## 配置项

`env.zsh` 里**真正设了的**只有三个：`ZSH="$WTOOL_PROJECT_DIR"`、`ZSH_THEME="ys"`、
`plugins=(git)`，外加一行 `WTOOL_PROJECT_DIR` 的兜底默认值。
下面这些里，**除了 `ZSH_CACHE_DIR`**（它是 `oh-my-zsh.sh` 自己设的，没写在模板里）
都在 `env.zsh` 里以注释形式列着，取消注释即可生效：

| 变量 / zstyle | 作用 |
|---|---|
| `ZSH_CUSTOM` | 换一个 custom 目录（默认 `$ZSH/custom`） |
| `ZSH_CACHE_DIR` | 缓存目录（`oh-my-zsh.sh` 设的：默认 `$ZSH/cache`，不可写时退到 `${XDG_CACHE_HOME:-$HOME/.cache}/oh-my-zsh`） |
| `ZSH_THEME_RANDOM_CANDIDATES` | `ZSH_THEME="random"` 时从这些主题里抽 |
| `CASE_SENSITIVE` / `HYPHEN_INSENSITIVE` | 补全大小写 / 连字符敏感 |
| `ENABLE_CORRECTION` | 打开命令拼写纠正（见 `lib/correction.zsh`） |
| `DISABLE_MAGIC_FUNCTIONS` | 关掉粘贴 URL 用的 `*-magic` |
| `DISABLE_LS_COLORS` / `DISABLE_AUTO_TITLE` | 关 `ls` 颜色 / 关自动改终端标题 |
| `COMPLETION_WAITING_DOTS` | 补全等待时显示的点（可自定义字符串） |
| `DISABLE_UNTRACKED_FILES_DIRTY` | git prompt 不算 untracked（大仓库提速） |
| `HIST_STAMPS` | `history` 输出里的时间戳格式 |
| `zstyle ':omz:update' mode/frequency/verbose` | 更新检查的行为（见功能说明第 8 条） |

## 快捷键

来自 `lib/key-bindings.zsh`（`bindkey -e`：默认 emacs 模式；下面每条在 `emacs` / `viins` /
`vicmd` 里基本都绑了）：

| 键 | 作用 |
|---|---|
| `↑` / `↓` | `up-line-or-beginning-search` / `down-line-or-beginning-search`：**打了前缀再按上下键**，只在匹配该前缀的历史里翻 |
| `PageUp` / `PageDown` | 上/下翻一行历史 |
| `Home` / `End` | 行首 / 行尾 |
| `Shift-Tab` | 补全菜单里**反向**选择 |
| `Backspace` / `Delete` | 删前一个字符 / 删后一个字符 |
| `Ctrl-Delete` | 删掉整个词（`kill-word`） |
| `Ctrl-←` / `Ctrl-→` | 按词前后移动 |
| `Esc-w` | `kill-region` |
| `Esc-l` | 直接执行 `ls`（宏绑定） |
| `Ctrl-R` | `history-incremental-search-backward`（增量搜历史） |
| `空格` | `magic-space`：**做**历史展开 —— 比如敲了 `!!` 再按空格，当场就展开成上一条命令，然后插入一个空格。⚠️ `lib/key-bindings.zsh` 里那行注释 `# [Space] - don't do history expansion` 是**上游写反了**（2026-10-04 订正，判据：`man zshzle` → `magic-space  Perform history expansion and insert a space into the buffer.`）。**别照着注释把这条改回去** |
| `Ctrl-X Ctrl-E` | 用 `$EDITOR` 编辑当前命令行 |
| `Esc-m` | `copy-prev-shell-word` |

> 这些绑定依赖 `$terminfo`；terminfo 里没有对应键时会退到硬编码的转义序列
> （`^[[3~`、`^[[1;5C` 之类）。

## 排错

| 现象 | 原因 / 怎么办 |
|---|---|
| `[oh-my-zsh] Insecure completion-dependent directories detected:`（附 `compaudit` 输出） | 这是 `lib/compfix.zsh` 的安全检查：`$ZSH/custom`（或其中某些目录）**属主不是当前用户，或 group/other 有写权限**。正常自用一般不会出现；**在 docker 里以 root 访问宿主目录必然出现**（属主是宿主用户）——容器现象，不是配置问题。真出现了按提示 `compaudit \| xargs chmod g-w,o-w`，或设 `ZSH_DISABLE_COMPFIX=true`（本仓库**故意没有**替你设它：那会把真实的安全提醒永久静音） |
| `[oh-my-zsh] theme '<名字>' not found` | `ZSH_THEME` 写的主题在 `$ZSH_CUSTOM` / `$ZSH_CUSTOM/themes` / `$ZSH/themes` 里都没有；用 `omz theme list` 看有哪些 |
| `Error: Oh My Zsh can't be loaded from: bash` | `source $ZSH/oh-my-zsh.sh` 落在了 non-zsh shell 里（`oh-my-zsh.sh` 开头就检查 `$ZSH_VERSION`）。本项目的 env 只往 `~/.zshrc` 注入，bash 里不该 source 它 |
| `Error: Oh My Zsh can't be loaded in \`<emulate>\` emulation mode` | zsh 处于 `sh` / `ksh` 兼容模式时不能加载（`oh-my-zsh.sh` 里 `[[ "$(emulate)" = zsh ]]` 那条） |
| `omz update` 失败 / 提示切不到分支 | 上游 `tools/upgrade.sh` 默认 checkout `master`；本仓库真源分支是 `main`（没有 `master`），`git checkout -q "$branch" -- || exit 1` 直接退出。要跟上游同步得人来定（rebase/merge 是单独的事） |
| `omz theme set` / `omz plugin enable` 改完没生效 | 它们改的是 **`~/.zshrc`**（还会留一份 `.zshrc.bck`），而本项目的 `ZSH_THEME` / `plugins` 在**仓库里的 `env.zsh`**。要长期生效请改 `env.zsh` |
| 手滑跑了 `uninstall_oh_my_zsh` | 它是给"直接安装 oh-my-zsh"的用户的：确认之后会把 `~/.zshrc` 改名成 `~/.zshrc.omz-uninstalled-<日期>`（wtool 的受管块就在里面），`~/.oh-my-zsh` 存在的话还会被删。本项目卸载用 `wtool uninstall shell/oh-my-zsh`；真跑错了就把改名的那个文件改回 `.zshrc` |
| 以前 `ZSH=~/.oh-my-zsh`，现在这个目录不见了 | 现在 `ZSH` 指向 `$HOME/.wtool/wtool-work-dir/links/shell/oh-my-zsh`（wtool 建的中转链接），**不再需要 `~/.oh-my-zsh`**；`custom/`、`cache/`、`log/` 都在本仓库里 |
| 想升级 oh-my-zsh 本体 | 需要跟上游 rebase/merge，属于单独处理的事项（`WTOOL.md` 里也这么记着） |

## 测试

**本仓库没有本地测试脚本**；`.github/workflows/main.yml` 里跑的是**语法检查**：
对下面这些文件逐个 `zsh -n`，任何一个语法错就失败：

```
./oh-my-zsh.sh
./lib/*.zsh
./plugins/*/*.plugin.zsh
./plugins/*/_*
./themes/*.zsh-theme
```

> ⚠️ 两个事实（2026-10-04 核对）：
> ① 上面这份 glob **逐字来自** `.github/workflows/main.yml` 的 “Check syntax” 那一步
> （另外它只在 `github.repository == 'ohmyzsh/ohmyzsh'` 时才跑 —— fork 上不跑）；
> ② 这个 workflow 的触发分支写的是 **`master`**（`on.push.branches: [master]`），
> 而本 fork 的真源分支是 `main` —— 所以在我们的 fork 上**这条 CI 根本不会触发**，
> 语法检查只能自己在本地跑（就是下面这段）。

本地等价命令（改完 `env.zsh` / `oh-my-zsh.sh` / 任何 lib、plugin、theme 之后至少跑一遍）：

```sh
zsh -n env.zsh
for f in ./oh-my-zsh.sh ./lib/*.zsh ./plugins/*/*.plugin.zsh ./plugins/*/_* ./themes/*.zsh-theme; do
  zsh -n "$f" || echo "语法错: $f"
done
# 冒烟：ZSH / 主题 / 插件是不是从本仓库来的
zsh -ic 'WTOOL_PROJECT_DIR=$PWD; source env.zsh;
         echo "ZSH=$ZSH"; echo "THEME=$ZSH_THEME"; echo "PLUGINS=$plugins"'
```

> 冒烟命令里 `WTOOL_PROJECT_DIR=$PWD` 是**故意**加的：这台机器上 shell 启动时可能已经带着
> 别的项目的值（旧安装留下的），不钉住的话 `echo $ZSH` 显示的不是本仓库。
> 钉住之后应看到 `ZSH=<本仓库路径>`、`THEME=ys`、`PLUGINS=git` —— 这三条**这次跑过**。
> 没有 tty 的环境里 `zsh -ic` 会多打印几行 `can't change option: zle`，那是 zle 在无 tty
> 时的现象，不是配置错。

---

<details>
<summary>Table of Contents</summary>

- [Getting Started](#getting-started)
  - [Operating System Compatibility](#operating-system-compatibility)
  - [Prerequisites](#prerequisites)
- [Using Oh My Zsh](#using-oh-my-zsh)
  - [Plugins](#plugins)
    - [Enabling Plugins](#enabling-plugins)
    - [Using Plugins](#using-plugins)
  - [Themes](#themes)
    - [Selecting A Theme](#selecting-a-theme)
  - [FAQ](#faq)
- [Advanced Topics](#advanced-topics)
  - [Custom Plugins And Themes](#custom-plugins-and-themes)
  - [Enable GNU ls In macOS And freeBSD Systems](#enable-gnu-ls-in-macos-and-freebsd-systems)
  - [Skip Aliases](#skip-aliases)
  - [Disable async git prompt](#disable-async-git-prompt)
- [Getting Updates](#getting-updates)
  - [Updates Verbosity](#updates-verbosity)
  - [Manual Updates](#manual-updates)
- [How Do I Contribute To Oh My Zsh?](#how-do-i-contribute-to-oh-my-zsh)
  - [Do Not Send Us Themes](#do-not-send-us-themes)
- [Contributors](#contributors)
- [Follow Us](#follow-us)
- [Merchandise](#merchandise)
- [License](#license)
- [About Planet Argon](#about-planet-argon)

</details>

## Getting Started

### Operating System Compatibility

| O/S            | Status  |
| :------------- | :-----: |
| Android        |   ✅    |
| freeBSD        |   ✅    |
| LCARS          |   🛸    |
| Linux          |   ✅    |
| macOS          |   ✅    |
| OS/2 Warp      |   ❌    |
| Windows (WSL2) |   ✅    |


### Prerequisites

- [Zsh](https://www.zsh.org) should be installed (v4.3.9 or more recent is fine but we prefer 5.0.8 and newer). If not pre-installed (run `zsh --version` to confirm), check the following wiki instructions here: [Installing ZSH](https://github.com/ohmyzsh/ohmyzsh/wiki/Installing-ZSH)
- `curl` or `wget` should be installed
- `git` should be installed (recommended v2.4.11 or higher)

### Basic Installation

> **这一节已按 wtool 的口径改写。** 上游那种"下载并运行 `tools/install.sh`"的安装方式
> 在本 fork 里**不再使用**：安装统一由 wtool 管 ——
> 见 [wtool 的 README（GitHub：allinkernel/wtool-doc）](https://github.com/allinkernel/wtool-doc/blob/main/README.md)。
> 本仓库只是源码/配置；也就不存在"把 install.sh 先下下来看一眼再跑"这一步了
> （想审查 wtool 做什么，看它的仓库；项目的 `install.sh` / `uninstall.sh` 已经退休）。

## Using Oh My Zsh

### Plugins

Oh My Zsh comes with a shitload of plugins for you to take advantage of. You can take a look in the [plugins](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins) directory and/or the [wiki](https://github.com/ohmyzsh/ohmyzsh/wiki/Plugins) to see what's currently available.

#### Enabling Plugins

Once you spot a plugin (or several) that you'd like to use with Oh My Zsh, you'll need to enable them in the `.zshrc` file. You'll find the zshrc file in your `$HOME` directory. Open it with your favorite text editor and you'll see a spot to list all the plugins you want to load.

```sh
vi ~/.zshrc
```

For example, this might begin to look like this:

```sh
plugins=(
  git
  bundler
  dotenv
  macos
  rake
  rbenv
  ruby
)
```

_Note that the plugins are separated by whitespace (spaces, tabs, new lines...). **Do not** use commas between them or it will break._

#### Using Plugins

Each built-in plugin includes a **README**, documenting it. This README should show the aliases (if the plugin adds any) and extra goodies that are included in that particular plugin.

### Themes

We'll admit it. Early in the Oh My Zsh world, we may have gotten a bit too theme happy. We have over one hundred and fifty themes now bundled. Most of them have [screenshots](https://github.com/ohmyzsh/ohmyzsh/wiki/Themes) on the wiki (We are working on updating this!). Check them out!

#### Selecting A Theme

_Robby's theme is the default one. It's not the fanciest one. It's not the simplest one. It's just the right one (for him)._

Once you find a theme that you'd like to use, you will need to edit the `~/.zshrc` file. You'll see an environment variable (all caps) in there that looks like:

```sh
ZSH_THEME="robbyrussell"
```

To use a different theme, simply change the value to match the name of your desired theme. For example:

```sh
ZSH_THEME="agnoster" # (this is one of the fancy ones)
# see https://github.com/ohmyzsh/ohmyzsh/wiki/Themes#agnoster
```

> [!NOTE]
> You will many times see screenshots for a zsh theme, and try it out, and find that it doesn't look the same for you.
> 
> This is because many themes require installing a [Powerline Font](https://github.com/powerline/fonts) or a [Nerd Font](https://github.com/ryanoasis/nerd-fonts) in order to render properly.
> Without them, these themes will render weird prompt symbols. Check out [the FAQ](https://github.com/ohmyzsh/ohmyzsh/wiki/FAQ#i-have-a-weird-character-in-my-prompt) for more information.
>
> Also, beware that themes only control what your prompt looks like. This is, the text you see before or after your cursor, where you'll type your commands.
> Themes don't control things such as the colors of your terminal window (known as _color scheme_) or the font of your terminal. These are settings that you can change in your terminal emulator.
> For more information, see [what is a zsh theme](https://github.com/ohmyzsh/ohmyzsh/wiki/FAQ#what-is-a-zsh-theme).

Open up a new terminal window and your prompt should look something like this:

![Agnoster theme](https://cloud.githubusercontent.com/assets/2618447/6316862/70f58fb6-ba03-11e4-82c9-c083bf9a6574.png)

In case you did not find a suitable theme for your needs, please have a look at the wiki for [more of them](https://github.com/ohmyzsh/ohmyzsh/wiki/External-themes).

If you're feeling feisty, you can let the computer select one randomly for you each time you open a new terminal window.

```sh
ZSH_THEME="random" # (...please let it be pie... please be some pie..)
```

And if you want to pick random theme from a list of your favorite themes:

```sh
ZSH_THEME_RANDOM_CANDIDATES=(
  "robbyrussell"
  "agnoster"
)
```

If you only know which themes you don't like, you can add them similarly to an ignored list:

```sh
ZSH_THEME_RANDOM_IGNORED=(pygmalion tjkirch_mod)
```

### FAQ

If you have some more questions or issues, you might find a solution in our [FAQ](https://github.com/ohmyzsh/ohmyzsh/wiki/FAQ).

## Advanced Topics

If you're the type that likes to get their hands dirty, these sections might resonate.

### Custom Plugins And Themes

If you want to override any of the default behaviors, just add a new file (ending in `.zsh`) in the `custom/` directory.

If you have many functions that go well together, you can put them as a `XYZ.plugin.zsh` file in the `custom/plugins/` directory and then enable this plugin.

If you would like to override the functionality of a plugin distributed with Oh My Zsh, create a plugin of the same name in the `custom/plugins/` directory and it will be loaded instead of the one in `plugins/`.

### Enable GNU ls In macOS And freeBSD Systems

<a name="enable-gnu-ls"></a>

The default behaviour in Oh My Zsh is to use BSD `ls` in macOS and FreeBSD systems. If GNU `ls` is installed
(as `gls` command), you can choose to use it instead. To do it, you can use zstyle-based config before
sourcing `oh-my-zsh.sh`:

```zsh
zstyle ':omz:lib:theme-and-appearance' gnu-ls yes
```

_Note: this is not compatible with `DISABLE_LS_COLORS=true`_

### Skip Aliases

<a name="remove-directories-aliases"></a>

If you want to skip default Oh My Zsh aliases (those defined in `lib/*` files) or plugin aliases,
you can use the settings below in your `~/.zshrc` file, **before Oh My Zsh is loaded**. Note that
there are many different ways to skip aliases, depending on your needs.

```sh
# Skip all aliases, in lib files and enabled plugins
zstyle ':omz:*' aliases no

# Skip all aliases in lib files
zstyle ':omz:lib:*' aliases no
# Skip only aliases defined in the directories.zsh lib file
zstyle ':omz:lib:directories' aliases no

# Skip all plugin aliases
zstyle ':omz:plugins:*' aliases no
# Skip only the aliases from the git plugin
zstyle ':omz:plugins:git' aliases no
```

You can combine these in other ways taking into account that more specific scopes take precedence:

```sh
# Skip all plugin aliases, except for the git plugin
zstyle ':omz:plugins:*' aliases no
zstyle ':omz:plugins:git' aliases yes
```

A previous version of this feature was using the setting below, which has been removed:

```sh
zstyle ':omz:directories' aliases no
```

Instead, you can now use the following:

```sh
zstyle ':omz:lib:directories' aliases no
```

#### Notice <!-- omit in toc -->

> This feature is currently in a testing phase and it may be subject to change in the future.
> It is also not currently compatible with plugin managers such as zpm or zinit, which don't
> source the init script (`oh-my-zsh.sh`) where this feature is implemented in.

> It is also not currently aware of "aliases" that are defined as functions. Example of such
> are `gccd`, `ggf`, or `ggl` functions from the git plugin.

### Disable async git prompt

Async prompt functions are an experimental feature (included on April 3, 2024) that allows Oh My Zsh to render prompt information
asynchronously. This can improve prompt rendering performance, but it might not work well with some setups. We hope that's not an
issue, but if you're seeing problems with this new feature, you can turn it off by setting the following in your .zshrc file,
before Oh My Zsh is sourced:

```sh
zstyle ':omz:alpha:lib:git' async-prompt no
```

## Getting Updates

> 这一段讲的是**上游**的更新机制。本仓库是 fork（真源分支 `main`），
> `tools/upgrade.sh` 却默认 checkout `master`，所以 `omz update` 在这个仓库上跑不起来 ——
> 细节见前面「本项目说明」的功能说明第 8 条和排错表。

By default, you will be prompted to check for updates every 2 weeks. You can choose other update modes by adding a line to your `~/.zshrc` file, **before Oh My Zsh is loaded**:

1. Automatic update without confirmation prompt:

   ```sh
   zstyle ':omz:update' mode auto
   ```

2. Just offer a reminder every few days, if there are updates available:

   ```sh
   zstyle ':omz:update' mode reminder
   ```

3. To disable automatic updates entirely:

   ```sh
   zstyle ':omz:update' mode disabled
   ```

NOTE: you can control how often Oh My Zsh checks for updates with the following setting:

```sh
# This will check for updates every 7 days
zstyle ':omz:update' frequency 7
# This will check for updates every time you open the terminal (not recommended)
zstyle ':omz:update' frequency 0
```

### Updates Verbosity

You can also limit the update verbosity with the following settings:

```sh
zstyle ':omz:update' verbose default # default update prompt

zstyle ':omz:update' verbose minimal # only few lines

zstyle ':omz:update' verbose silent # only errors
```

### Manual Updates

If you'd like to update at any point in time (maybe someone just released a new plugin and you don't want to wait a week?) you just need to run:

```sh
omz update
```

Magic! 🎉

## Uninstalling Oh My Zsh

> **这一节已按 wtool 的口径改写。** 卸载同样由 wtool 管
> （见 [wtool 的 README](https://github.com/allinkernel/wtool-doc/blob/main/README.md)；
> 对应的命令是 `wtool uninstall shell/oh-my-zsh`）。
>
> 上游自带的 `uninstall_oh_my_zsh`（`lib/functions.zsh` 里那个函数，会去跑
> `tools/uninstall.sh`）是给"直接安装 oh-my-zsh"的用户准备的，本项目**不要**用它：
> 它只认 `~/.oh-my-zsh` 那套布局，看不到 wtool 的记账；而且 `tools/uninstall.sh` 在
> 确认之后会把 **`~/.zshrc` 改名成 `.zshrc.omz-uninstalled-<日期>`**（wtool 写的受管块
> 就在这个文件里），只有存在 `~/.zshrc.pre-oh-my-zsh` 时才谈得上"还原"。
> 本项目由 wtool 装、由 wtool 撤。

## How Do I Contribute To Oh My Zsh?

Before you participate in our delightful community, please read the [code of conduct](CODE_OF_CONDUCT.md).

I'm far from being a [Zsh](https://www.zsh.org/) expert and suspect there are many ways to improve – if you have ideas on how to make the configuration easier to maintain (and faster), don't hesitate to fork and send pull requests!

We also need people to test out pull requests. So take a look through [the open issues](https://github.com/ohmyzsh/ohmyzsh/issues) and help where you can.

See [Contributing](CONTRIBUTING.md) for more details.

### Do Not Send Us Themes

We have (more than) enough themes for the time being. Please add your theme to the [external themes](https://github.com/ohmyzsh/ohmyzsh/wiki/External-themes) wiki page.

## Contributors

Oh My Zsh has a vibrant community of happy users and delightful contributors. Without all the time and help from our contributors, it wouldn't be so awesome.

Thank you so much!

<a href="https://github.com/ohmyzsh/ohmyzsh/graphs/contributors">
  <img src="https://contrib.rocks/image?repo=ohmyzsh/ohmyzsh" width="100%"/>
</a>

## Follow Us

We're on social media:

- [@ohmyzsh](https://x.com/ohmyzsh) on X (formerly Twitter). You should follow it.
- [Facebook](https://www.facebook.com/Oh-My-Zsh-296616263819290/) poke us.
- [Instagram](https://www.instagram.com/_ohmyzsh/) tag us in your post showing Oh My Zsh!
- [Discord](https://discord.gg/ohmyzsh) to chat with us!

## Merchandise

We have [stickers, shirts, and coffee mugs available](https://shop.planetargon.com/collections/oh-my-zsh?utm_source=github) for you to show off your love of Oh My Zsh. Again, you will become the talk of the town!

## License

Oh My Zsh is released under the [MIT license](LICENSE.txt).

## About Planet Argon

![Planet Argon](https://pa-github-assets.s3.amazonaws.com/PARGON_logo_digital_COL-small.jpg)

Oh My Zsh was started by the team at [Planet Argon](https://www.planetargon.com/?utm_source=github), a [Ruby on Rails development agency](https://www.planetargon.com/services/ruby-on-rails-development?utm_source=github). Check out our [other open source projects](https://www.planetargon.com/open-source?utm_source=github).
