---
name: git-team-workflow
description: 按团队的 fork 与 upstream 分支流程同步 main 和 feature/vip，维护 feature/change 与 feature/vip_change，并创建对应 Pull Request。
---

# 团队 Git Fork 工作流

当用户要求同步个人 fork 与主仓库、维护 `feature/change` 或 `feature/vip_change`，或按团队流程提交 Pull Request（PR）时，使用此 skill。

## 团队流程

- 个人仓库由团队共享的主仓库 fork 而来。不要假定 remote 的名称：根据 URL 识别各个 remote，并核实 fork 的父仓库。如果 URL 或 GitHub 元数据无法明确仓库关系，先让用户确认对应关系。
- 下文使用 `upstream` 表示团队主仓库，使用 `origin` 表示个人 fork。这些是角色称呼，不要求实际 remote 必须使用这些名称。
- 不要直接在本地 `main` 分支进行开发修改。
- 普通开发流程：将个人仓库的 `main` 合入本地 `feature/change`，推送到个人仓库，然后先发起合入个人仓库 `main` 的 PR，再从个人仓库 `main` 发起合入主仓库 `main` 的 PR。
- 两个 PR 都合并后，先将主仓库最新 `main` 同步回个人仓库，再次 fetch，并将个人仓库最新的 `main` 合入本地 `feature/change`，补齐主仓库 PR 产生的合并历史。取得推送确认后，将补齐后的 `feature/change` 普通推送到个人仓库同名分支。
- VIP 开发流程：先将主仓库 `feature/vip` 同步到个人仓库的同名分支。本地 `feature/vip_change` 必须包含主仓库 `main`、主仓库 `feature/vip` 和本地最新 `feature/change` 的内容。明确合并这些分支，处理冲突时不能直接丢弃任意一侧。推送到个人仓库后，从个人仓库 `feature/vip_change` 发起合入主仓库 `feature/vip` 的 PR。
- VIP PR 合并后，将主仓库 `feature/vip` 再次同步到个人仓库的同名分支，再次 fetch，并将个人仓库最新的 `feature/vip` 合入本地 `feature/vip_change`，补齐合并历史。取得推送确认后，将补齐后的 `feature/vip_change` 普通推送到个人仓库同名分支，为下一轮开发做准备。
- 并非每个项目都配置了 `upstream`。缺少时，不要编造 URL 或静默跳过必需的主仓库操作；应询问用户是否使用其他主仓库，或是否省略该步骤。

## 安全与执行规则

1. 首先检查 `pwd`、`git status --short --branch`、`git branch --show-current` 和 `git remote -v`。检查 remote URL；对于 GitHub 仓库，使用 `gh api` 核实仓库归属和 fork 的父仓库。不要输出 URL 中嵌入的凭据。
2. 如果存在未提交改动、当前分支不符合预期，或正在进行 merge/rebase，应在切换分支、合并或其他工作区修改前暂停。除非用户明确要求，否则不要 stash、reset、clean、rebase 或覆盖现有工作。
3. 检查所需的本地和远程分支是否存在。创建缺失分支或使用其他分支名之前，先询问用户。
4. 操作 GitHub 前执行 `gh auth status`。如果无法使用 GitHub CLI，说明哪些 GitHub 操作需要用户手动完成；只有另行确认过的本地步骤才能继续执行。
5. 每次写入远端前，包括同步 fork、push 或创建 PR，先展示准确的仓库、分支和操作，并取得用户确认。除非用户明确确认了该项具体操作，否则不得强推、使用 `gh repo sync --force` 或合并 PR。
6. 按阶段执行。每次 fetch、merge、push、同步 fork 或操作 PR 后，都要核实结果并汇报当前阶段。进入依赖 PR 合并结果的下一阶段前，等待用户完成或确认 PR 已合并。
7. 如果无法快进同步、合并出现冲突、分支意外落后或命令失败，暂停并说明当前状态。未经用户指示，不得自行选择冲突处理方案、强制更新或继续后续阶段。

## 仓库识别

- 使用 `git remote -v` 查看所有已配置的 remote；将 GitHub SSH 和 HTTPS URL 解析为 `OWNER/REPO`。
- 查询 GitHub 仓库元数据，例如 `gh api repos/OWNER/REPO`，识别 fork 及其 `.parent.full_name`。根据父子关系匹配个人仓库与主仓库，而不是依赖 remote 名称。
- 如果没有配置主仓库 remote，但能核实 fork 的父仓库，先展示父仓库信息，取得确认后再添加 remote。如果无法获取 fork 元数据或 remote 含义不明确，让用户确认。
- GitHub 操作必须明确指定仓库和分支；同步时不要依赖当前仓库的默认分支。

## 普通 `main` 流程

1. 确认本地工作区干净，预期工作分支为 `feature/change`。确认个人仓库、主仓库及其各自的 `main` 分支存在。
2. 取得确认后，执行 `gh repo sync PERSONAL --source UPSTREAM --branch main`，将主仓库 `main` 同步到个人仓库 `main`，不得带 `--force`。其中 `PERSONAL` 和 `UPSTREAM` 分别替换为已核实的个人仓库和主仓库 `OWNER/REPO`。随后执行 `git fetch --all`，获取所有已配置 remote 的最新内容。
3. 在本地 `feature/change` 上，合入 fetch 得到的个人仓库 `main` 分支引用。展示合并后的状态和差异摘要。取得确认后，将本地 `feature/change` 推送到个人仓库同名分支。
4. 取得确认后，创建个人仓库 `feature/change` → 个人仓库 `main` 的 PR。核实目标分支（base）和来源分支（head），等待用户确认已合并。
5. 取得确认后，创建个人仓库 `main` → 主仓库 `main` 的 PR。核实目标分支和来源分支，等待用户确认已合并。
6. 主仓库 PR 合并后，取得确认，再将主仓库最新 `main` 同步回个人仓库 `main`，确保个人仓库也包含主仓库 PR 产生的合并提交；随后再次 fetch 所有 remote。
7. 将 fetch 得到的个人仓库最新 `main` 合入本地 `feature/change`，按下文的“合并历史收尾检查”核实基线已完整包含。
8. 展示补齐历史后的待推送提交，取得确认后，普通推送 `feature/change` 到个人仓库同名分支，不使用强推。随后 fetch 并核实本地与远端同名分支完全对齐。

## VIP `feature/vip` 流程

1. 确认普通 `main` 流程已完成，本地 `feature/change` 已包含个人仓库最新 `main` 的内容，本地 `feature/vip_change` 存在且工作区干净。未经确认，不要创建或切换分支。
2. 取得确认后，将主仓库 `feature/vip` 同步到个人仓库同名分支，再 fetch 所有 remote。
3. 在本地 `feature/vip_change` 上，依次合入 fetch 得到的主仓库 `main`、主仓库 `feature/vip` 和本地 `feature/change` 分支引用。每次合并后检查结果；出现冲突时暂停，请用户协助处理。
4. 展示合并后的状态和差异摘要。取得确认后，将本地 `feature/vip_change` 推送到个人仓库同名分支。
5. 取得确认后，创建个人仓库 `feature/vip_change` → 主仓库 `feature/vip` 的 PR。核实目标分支和来源分支，等待用户确认已合并。
6. 取得确认后，将主仓库 `feature/vip` 再次同步到个人仓库同名分支，并核实更新结果；随后再次 fetch 所有 remote。
7. 将 fetch 得到的个人仓库最新 `feature/vip` 合入本地 `feature/vip_change`，按下文的“合并历史收尾检查”核实基线已完整包含。
8. 展示补齐历史后的待推送提交，取得确认后，普通推送 `feature/vip_change` 到个人仓库同名分支，不使用强推。随后 fetch 并核实本地与远端同名分支完全对齐。

## 合并历史收尾检查

- 普通流程检查本地 `feature/change` 与个人仓库远程跟踪引用 `main`；VIP 流程检查本地 `feature/vip_change` 与个人仓库远程跟踪引用 `feature/vip`。例如 remote 名为 `origin` 时，分别使用 `origin/main` 和 `origin/feature/vip`，不要误用可能过期或不存在的本地基线分支。
- 代码没有差异不代表提交历史已经同步。即使只有 GitHub PR 新增的合并提交，也必须补齐到对应本地开发分支，不能仅凭文件差异为空跳过。
- 操作前仍须遵守工作区和分支安全检查。可以快进时，使用 `git merge --ff-only <远程跟踪引用>` 补齐历史，不额外创建合并提交。如果不能快进，先说明分叉情况并等待用户确认，不得强制移动分支。
- 使用 `git merge-base --is-ancestor <远程跟踪引用> <本地开发分支>` 确认基线已完整包含，再用 `git rev-list --left-right --count <远程跟踪引用>...<本地开发分支>` 检查：左侧应为 `0`，表示本地开发分支没有遗漏基线提交；右侧允许保留尚未合入基线的本地提交。
- 补齐历史后，将对应开发分支普通推送回个人仓库作为收尾步骤；即使只有合并历史、没有文件差异，也不要省略。仍须遵守远端写入确认规则；用户已明确确认本次具体推送时无需重复询问，否则先展示仓库、分支和待推送提交并取得确认。用户暂不授权时，明确说明本地历史已补齐、远端开发分支尚未对齐。
- 推送后再次 fetch，使用 `git rev-list --left-right --count <个人仓库远程跟踪引用/开发分支>...<本地开发分支>` 确认输出为 `0 0`。例如 remote 名为 `origin` 时，分别检查 `origin/feature/change...feature/change` 和 `origin/feature/vip_change...feature/vip_change`。已对齐的分支无需重复推送。
- 只有本次涉及的开发分支都完成基线包含检查、收尾推送及远端对齐检查，才能汇报相应流程全部完成；若受未提交改动、缺少授权或推送失败等原因阻塞，应明确说明未完成的步骤，不能仅因个人 fork 同步成功就宣告全部完成。

## GitHub CLI 注意事项

- 创建 PR 时明确指定仓库和分支。普通流程为个人仓库 `feature/change` → 个人仓库 `main`，再由个人仓库 `main` → 主仓库 `main`；VIP 流程为个人仓库 `feature/vip_change` → 主仓库 `feature/vip`。
- 检查每条 `gh` 命令的执行结果，并核实新建 PR 的目标分支和来源分支后，才能认定当前阶段完成。
- 同步、push 或 PR 命令失败不代表成功。应报告错误输出；未经用户指示，不得以强制方式重试或改用其他目标。
