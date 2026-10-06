# one-more-trick

保存和持续维护我在实践中使用的 AI Agent Skills。每个技能独立存放，可按需安装，也可批量安装。

## 技能目录

| 技能 | 用途 | 调用方式 |
| --- | --- | --- |
| [gpt6-model-routing](skills/gpt6-model-routing/SKILL.md) | 按任务选择 GPT-6 Luna、GPT-6.1 Sol 或 GPT-6 Astra，以及合适的推理档位 | `$gpt6-model-routing` |

模型分工是个人工作流建议。模型是否可用取决于当前宿主；技能本身不能切换主对话模型。价格来自用户提供的截图，未经官方定价核验。

## 安装前准备

- 使用支持 Skills 的 Codex。
- 私有仓库需要当前 GitHub 账号拥有访问权限。先配置 Git 凭据，或使用 GitHub CLI 执行 `gh auth login`、`gh auth setup-git`。不要把令牌写进仓库 URL 或文件。
- 手动安装需要 Git；使用安装器脚本还需要 Python。
- 安装的是完整技能文件夹，包括 `SKILL.md`、`agents/` 以及未来可能添加的 `scripts/`、`references/` 等资源。

### 选择安装位置

[Codex 官方文档](https://developers.openai.com/codex/skills/) 当前列出的个人技能目录为 `~/.agents/skills/`，项目专用目录为 `<项目根目录>/.agents/skills/`。

本机内置 `skill-installer` 使用 `$CODEX_HOME/skills/`（未设置时为 `~/.codex/skills/`）。现有技能已安装在该位置的用户可继续使用该目录。下面的手动安装示例采用官方个人目录；请按实际 Codex 版本选择一个位置，同名技能只保留一份，避免重复发现。

## 方式一：让 Codex 安装一个技能

在 Codex 中发送：

```text
使用 $skill-installer，从 https://github.com/fadeawaylove/one-more-trick
安装 skills/gpt6-model-routing，使用 main 分支。
```

安装器默认不会覆盖已经存在的同名目录。更新请使用下文的备份再安装流程。

也可在 Windows PowerShell 中直接运行本机安装器：

```powershell
$codexRoot = if ($env:CODEX_HOME) { $env:CODEX_HOME } else { Join-Path $HOME '.codex' }
$installer = Join-Path $codexRoot 'skills/.system/skill-installer/scripts/install-skill-from-github.py'
python $installer --repo fadeawaylove/one-more-trick --path skills/gpt6-model-routing --ref main --method git
```

该命令要求安装器文件确实存在；找不到时使用方式二。私有仓库的 Git 认证由当前环境处理。

## 方式二：手动安装一个或全部技能

先克隆仓库（在希望保存源码的目录执行）：

```sh
git clone https://github.com/fadeawaylove/one-more-trick.git
cd one-more-trick
```

以下命令都在仓库根目录运行。它们先把旧版移到技能发现目录之外的备份目录，再复制完整新版，避免遗留已删除的资源文件。已有本地修改可从备份恢复。

### Windows PowerShell

```powershell
$ErrorActionPreference = 'Stop'
$skillsRoot = Join-Path $HOME '.agents/skills'
$backupRoot = Join-Path $HOME '.skill-backups/one-more-trick'
# 如果本机已使用旧目录，改为：
# $skillsRoot = Join-Path $HOME '.codex/skills'
# 自定义 CODEX_HOME 的用户使用 Join-Path $env:CODEX_HOME 'skills'

# 安装一个技能：
$names = @('gpt6-model-routing')
# 安装全部技能时，用下面这一行替换上一行：
# $names = @(Get-ChildItem -LiteralPath './skills' -Directory | Where-Object { Test-Path (Join-Path $_.FullName 'SKILL.md') } | Select-Object -ExpandProperty Name)

New-Item -ItemType Directory -Force -Path $skillsRoot, $backupRoot | Out-Null
foreach ($name in $names) {
    $source = Join-Path (Join-Path $PWD 'skills') $name
    if (-not (Test-Path -LiteralPath (Join-Path $source 'SKILL.md'))) {
        throw "找不到技能：$name"
    }
    $destination = Join-Path $skillsRoot $name
    if (Test-Path -LiteralPath $destination) {
        $backup = Join-Path $backupRoot ("{0}-{1}" -f $name, [guid]::NewGuid().ToString('N'))
        Move-Item -LiteralPath $destination -Destination $backup
        Write-Host "已备份到 $backup"
    }
    Copy-Item -LiteralPath $source -Destination $destination -Recurse
    Write-Host "已安装 $name -> $destination"
}
```

### macOS / Linux（Bash）

```bash
(
set -e
skills_root="$HOME/.agents/skills"
backup_root="$HOME/.skill-backups/one-more-trick"
# 使用旧目录时改为：
# skills_root="${CODEX_HOME:-$HOME/.codex}/skills"

# 安装一个技能：
sources=(skills/gpt6-model-routing)
# 安装全部技能时，用下面这一行替换上一行：
# sources=(skills/*)

mkdir -p "$skills_root" "$backup_root"
for source in "${sources[@]}"; do
  [ -f "$source/SKILL.md" ] || { echo "不是有效技能目录：$source" >&2; exit 1; }
  name="$(basename "$source")"
  destination="$skills_root/$name"
  if [ -e "$destination" ] || [ -L "$destination" ]; then
    backup="$(mktemp -d "$backup_root/$name.XXXXXX")"
    mv "$destination" "$backup/$name"
    echo "已备份到 $backup/$name"
  fi
  cp -R "$source" "$destination"
  echo "已安装 $name -> $destination"
done
)
```

## 更新技能

1. 在本地仓库根目录执行 `git pull --ff-only origin main`。若本地源码有修改，先提交或备份并处理冲突。
2. 重新运行方式二中对应系统的安装命令，选择需要更新的技能。它会保留旧版备份。
3. 安装目录是源码的副本，单独执行 `git pull` 不会更新已安装技能。未来增加的新技能可再次按名称安装，或使用“安装全部”选项。
4. 若之前使用安装器默认目录，更新时务必使用同一 `skillsRoot` / `skills_root`，不要又安装到另一个发现目录。

## 使用与确认

安装后在下一轮对话输入：

```text
$gpt6-model-routing 帮我完成这个任务：……
```

确认安装目录下存在 `gpt6-model-routing/SKILL.md`，并能在技能选择器中找到“GPT-6 模型路由”。Codex 通常会自动发现变化；未显示时重启 Codex，并核对安装位置及同名技能是否重复。

## 后续添加技能

```text
skills/
  gpt6-model-routing/
    SKILL.md
    agents/
      openai.yaml
  another-skill/
    SKILL.md
```

- 每个技能放在 `skills/<技能名>/`，目录名与 `SKILL.md` 的 frontmatter `name` 一致。
- `SKILL.md` 必须包含 `name` 和 `description`；按需添加 UI 元数据和辅助资源。
- 新增技能时同步更新上方技能目录，并确认单个/全部安装后可以发现。
- 仅提交可分享的技能内容，不提交本机凭据、会话记录或缓存。

本仓库的 `skills/` 是分发源码目录，需要安装后才能作为个人技能使用。
