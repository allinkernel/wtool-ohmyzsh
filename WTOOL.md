# 迁移说明（shell/oh-my-zsh）

本目录是 **oh-my-zsh 的个人 fork**（原 `mytool/zsh/oh-my-zsh`，远端 `wsw-ohmyzsh`），
现在作为 wtool 的独立项目：由 `~/.zshrc` 里的 wtool 块 source。

> 目录里的 `README.md` 是 oh-my-zsh 自带的，未改动；迁移说明就是本文件。

## 安装

```sh
./install.sh      # 建中转链接 + 往 ~/.zshrc 写块（priority=10，最先加载）
./uninstall.sh    # 完全回退
```

> 首次（尚未 `git init`）需 `--force`；提交后不需要。

## 与 mytool 版本的差异

| 原 mytool | 现在 | 原因 |
|---|---|---|
| `wsw.zsh` 里 `export ZSH=$(get_this_dir)` | `env.zsh` 里 `export ZSH="$WTOOL_PROJECT_DIR"` | 路径由加载器提供，脚本不再自己猜 |
| 文件名叫 `wsw.zsh` | 改名为 `env.zsh`（框架约定，便于识别） | `<env src="env.zsh">` 指向它 |
| 由 `wsw-zshrc/wsw_env.sh` 里的 `source ${cur_dir}/oh-my-zsh/wsw.zsh` 间接触发 | 自己就是一个 wtool 项目，有独立块 | 拆开 `shell/zsh` 与 `shell/oh-my-zsh`，两者只靠 priority 排序，无代码耦合 |
| 无 `wtool.xml` | 有 `wtool.xml`（1 个 env） | 成为框架里的正式项目 |

**迁移时携带的未提交修改**：`wsw.zsh` 在迁移前有一处工作区改动
（`ZSH_THEME="strug"` 被注释掉，实际用 `ys`）。该改动已随迁移带入 `env.zsh`，
并包含在迁移提交里，未丢失。

## 关于 `~/.oh-my-zsh`

原来的用法是在 `~/.zshrc` 里设 `ZSH=~/.oh-my-zsh`。现在 `ZSH` 指向
`$HOME/.wtool/links/shell/oh-my-zsh`（引擎自动创建的中转链接），
**不再需要 `~/.oh-my-zsh` 这个目录**。`custom/`、`cache/`、`log/` 都在本项目内。

## 注意

- 本仓库体积约 11MB（oh-my-zsh 上游内容），其中 `cache/`、`log/`、`custom/` 被
  上游 `.gitignore` 忽略（只保留 `.gitkeep` / example 文件）。
- 升级 oh-my-zsh 需要与上游 rebase/merge，属于后续单独处理的事项。
