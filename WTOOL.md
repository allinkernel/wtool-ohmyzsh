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

## 关于 compaudit 的 "Insecure completion-dependent directories" 警告

如果你看到这样的横幅：

```
[oh-my-zsh] Insecure completion-dependent directories detected:
drwxrwxr-x ... /path/to/oh-my-zsh/custom
```

这是 oh-my-zsh 自带的安全检查，意思是 `$ZSH/custom`（或其中某些目录）
**属主不是当前用户，或者 group/other 有写权限**。

| 场景 | 说明 |
|---|---|
| 正常使用（你自己 clone 的仓库） | 一般不会出现，`custom/` 属主就是你 |
| **在 docker 容器里以 root 访问宿主目录** | 必然出现（属主是宿主用户，不是 root）——这是容器现象，不是配置问题 |
| 真出现了 | 按提示执行 `compaudit \| xargs chmod g-w,o-w`，或设 `ZSH_DISABLE_COMPFIX=true` |

本项目**没有**替你设 `ZSH_DISABLE_COMPFIX`：那会把一个真实的安全提醒永久静音。
