# BACKLOG —— shell/oh-my-zsh

> 这个文件是这个项目"接下来做什么、做到哪了"的**唯一权威**。
> 引擎/跨项目的事在 `~/self/wtool/harness/BACKLOG.md`，别混。
>
> 状态：⬜ 待做 · 🔄 在做 · ✅ 做完（写清怎么做的、验证到什么程度）· ⏸ 待决定（要人来拍）

---

## ⏸ 待拍板

### 1. CI 根本不会触发：workflow 写的是 `master`，本 fork 的真源分支是 `main`

**现状**（2026-10-04 P2 核对时发现）：

* `.github/workflows/main.yml` 的触发条件是 `on.push.branches: [master]` 和
  `on.pull_request.branches: [master]`；
* 它唯一的 job 还带一句 `if: github.repository == 'ohmyzsh/ohmyzsh'` ——
  **fork 上直接跳过**；
* 本仓库的真源分支是 `main`（`.repo/manifests/default.xml` 的 `<default revision="main">`）。

所以那条 `zsh -n` 语法检查**在我们这个 fork 上永远不会跑**。

**要人拍**：① 改 workflow（`master` → `main`，并去掉/放宽 repository 判断）—— 那是
**改上游文件**，将来跟上游 rebase 会冲突，要慎重；② 加一个本地脚本（比如
`scripts/syntax-check.sh`，但本仓库目前没有 `scripts/`）；③ 保持原样，只在 README 里
写明"CI 不触发，语法检查得自己跑"（**当前就是这一条**）。

### 2. 上游两处"写死 `master`"的地方，在这个 fork 上跑不通

* `tools/check_for_upgrade.sh` + `tools/upgrade.sh`：默认
  `branch=${"$(git config --local oh-my-zsh.branch)":-master}`，然后
  `git checkout -q "$branch" -- || exit 1` —— 本仓库**没有 `master`**，
  所以 `omz update` 会直接退出（README「排错」里写了）。
* `.github/workflows/*.yml`（见上）。

**要人拍**：要不要给本仓库设一个本地 git config
（`git config --local oh-my-zsh.branch main`）让 `omz update` 能走 —— 但"update"到底
该 rebase 上游还是别的，是单独的一件事（`WTOOL.md` 里也记着"跟上游 rebase/merge 属于
单独处理的事项"）。助手没动。

### 3. `lib/key-bindings.zsh` 里那条注释与 zsh 手册相反

`bindkey ' ' magic-space  # [Space] - don't do history expansion` ——
`man zshzle` 写的是 `magic-space  Perform history expansion and insert a space into the buffer.`
**注释说反了**（README 里已按手册订正）。这是**上游文件**，不建议为了这句注释改它
（将来 rebase 冲突）；要改就整份跟上游同步时一起提。助手没动。

---

## ✅ 做完的

- **2026-10-04 P2 文档核对**（本次提交，只改「本项目说明」那节，上游英文 README 一个字没动）：
  1. `lib/functions.zsh` 的函数个数 **13 → 15 个名字**（列全：`zsh_stats`、
     `uninstall_oh_my_zsh`、`upgrade_oh_my_zsh`、`open_command`、`mkcd` / `takedir`、
     `takeurl`、`takegit`、`take`、`alias_value`、`try_alias_value`、`default`、
     `env_default`、`omz_urlencode`、`omz_urldecode`）。
     数错的原因：`function mkcd takedir()` 一行两个名字、`function omz_urldecode {` 没写括号。
  2. **`空格` = `magic-space` 那条写反了**（原写"不做历史展开"）→ 按 `man zshzle` 订正。
  3. 更新频率 **"每 2 周" → 13 天**（`epoch_target=${UPDATE_ZSH_DAYS:-13}`）。
  4. 安装一节：`项目 id` → **路径就是身份**（ADR-0037）+ 容器装/测规矩。
  5. 测试一节补两条已核事实：CI 的 glob 逐字来自 `main.yml`、且该 CI 只认 `master`
     又在 fork 上被 `if:` 跳过（所以本地必须自己跑）。
  6. `AGENTS.md`：加容器规矩、"别信那条 magic-space 注释"、"数字宁可不写"三条。
  7. 新建本文件。

  **验证**：CI 等价语法检查 **553 个文件、0 失败**（`zsh -n env.zsh` + 五个 glob 逐个
  `zsh -n`）；冒烟（`ZDOTDIR`/`ZSH_CACHE_DIR` 都指临时目录，避免写真 `$HOME` 和仓库
  `cache/`）→ `ZSH=<本仓库>`、`THEME=ys`、`PLUGINS=git`；
  数字逐个 `ls | wc -l` 核过：插件 337、主题 143、lib 21、git 插件别名 195、
  `_omz::` 名字 10 个（8 个给用户 + `confirm`/`log` 两个内部）、`diagnostics.zsh` 5 个函数；
  加载顺序 8 步逐条对着 `oh-my-zsh.sh` 核过；报错原文 4 条 `grep -F` 核过；
  `custom/`、`cache/`、`log/` 的 `.gitignore` 规则与仓库体积 11M 核过。
