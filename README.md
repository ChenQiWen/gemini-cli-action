# Gemini CLI Action

一个用于 GitHub 的自动化辅助仓库，围绕 Google 提供的 Gemini CLI（或相关集成）构建的可复用 GitHub Actions 工作流集合，提供 Issue 自动分级、定时批量分级、PR 审查以及触发器来在 Issue/PR 上运行 Gemini 分析。

> 说明：本 README 基于仓库内的工作流定义文件（`.github/workflows/*.yml`）生成。如需补充项目源码说明（比如 npm 包或二进制发布），请把对应的 `package.json` / 源码路径提交到仓库，我会把 README 同步更新。

## 主要内容（文件）
- `.github/workflows/gemini-cli.yml` — 可复用工作流：在单个 Issue/PR 上运行 Gemini CLI 并发表评论。
- `.github/workflows/gemini-issue-automated-triage.yml` — 可复用工作流：接收 Issue 信息并运行 Gemini，返回并应用标签。
- `.github/workflows/gemini-issue-scheduled-triage.yml` — 定时/批量查找未分级 Issue 并调用 Gemini 批量分级。
- `.github/workflows/gemini-pr-review.yml` — 可复用工作流：在 PR 上运行 Gemini 进行代码审查建议并可发表评论。
- `.github/workflows/publish.yml` — 基于 release 事件的发布流程（示例含 npm 与 GitHub Packages 发布步骤）。

## 功能亮点
- Reusable workflows（`workflow_call`）: 方便其它仓库或同仓库 workflow 调用。
- 支持 GitHub App token（可选）以获得更高权限的操作。
- 通过 `google-github-actions/run-gemini-cli@v0` 执行 Gemini CLI 分析并把结果用于评论、标签或其它自动化动作。
- 支持单 Issue/PR 触发，也支持批量定时触发。

## 快速开始
1. 在仓库设置中添加需要的 Secrets：
   - `GEMINI_API_KEY`（可选/视 workflow 需求）
   - `APP_PRIVATE_KEY`（可选，如果使用 GitHub App 来生成 token）
   - `NPM_TOKEN`（如果使用 `publish.yml` 发布到 npm）

2. 可选仓库级变量（在仓库 Settings → Variables）
   - `APP_ID`（使用 GitHub App 时）
   - `GEMINI_CLI_VERSION`、`GCP_WIF_PROVIDER`、`GOOGLE_CLOUD_PROJECT`、`GOOGLE_CLOUD_LOCATION`、`SERVICE_ACCOUNT_EMAIL`、`GOOGLE_GENAI_USE_VERTEXAI`、`GOOGLE_GENAI_USE_GCA` 等，工作流中引用了这些变量来配置 Gemini/Google Cloud 运行时。

3. 以可复用工作流方式调用（示例：在另一个 workflow 中调用 `gemini-cli.yml`）：

```yaml
# 示例：在另一个 workflow 中调用 gemini-cli 可复用工作流
jobs:
  call-gemini:
    uses: ./.github/workflows/gemini-cli.yml@main
    with:
      user_request: "请检查这个 issue 是否需要进一步信息"
      issue_number: 123
      is_pr: false
    secrets:
      APP_PRIVATE_KEY: ${{ secrets.APP_PRIVATE_KEY }}
      GEMINI_API_KEY: ${{ secrets.GEMINI_API_KEY }}
```

注意：`uses` 引用本地工作流时可以使用相对路径 `./.github/workflows/<file>.yml@<ref>`，跨仓库调用则使用 `owner/repo/.github/workflows/<file>.yml@<ref>`。

## 各工作流要点
- `gemini-cli.yml`
  - 输入（inputs）: `user_request`（string）、`issue_number`（number）、`is_pr`（boolean）。
  - 作用：在单个 Issue/PR 上运行 Gemini，支持在 PR 分支或 main 分支 checkout，并使用 `gh` 工具发表评论以告知用户已开始处理。

- `gemini-issue-automated-triage.yml`
  - 输入（inputs）: `issue_number`、`issue_title`、`issue_body`
  - 作用：返回可用 labels 列表并调用 Gemini，解析 Gemini 输出 JSON 并将 label 应用到对应 Issue 上。

- `gemini-issue-scheduled-triage.yml`
  - 作用：定期查找未分级的 Issue，批量调用 Gemini 进行标签推荐并应用。

- `gemini-pr-review.yml`
  - 输入（inputs）: `pr_number`
  - 作用：获取 PR 的变更文件与元数据，调用 Gemini 进行 PR 审查并可将结果作为评论或状态更新。

- `publish.yml`
  - 触发条件：`release` created
  - 作用：示例性地展示如何发布到 npm 与 GitHub Packages（依赖仓库存在 npm 包配置）。

## 调试与本地测试建议
- 使用 GitHub CLI（gh）在本地模拟部分动作：例如 `gh workflow run` 触发 workflow（需要设置 secrets/variables）。
- 在 workflow 中临时打开更多调试信息（将 `settings.debug` 设为 `true`）以便 Gemini CLI 输出更详细的日志。
- 如果使用 GitHub App，确保 `APP_ID` 与 `APP_PRIVATE_KEY` 配置正确，并在需要写权限的步骤中使用生成的 token。

## 常见问题
- Q: 为什么工作流中有些步骤把 `GITHUB_TOKEN` 设为空？
  - A: 某些步骤有意禁用默认 token（例如当需要对外部服务交互或避免权限冲突时）以便使用更合适的身份验证方式（如 GH App token 或外部 API key）。

- Q: 仓库没有 `package.json`，`publish.yml` 怎么工作？
  - A: `publish.yml` 是一个示例/模板：如果仓库包含 npm 包并在 release 时需要发布，请确保 `package.json`、构建与版本等都已就绪并填充相应的 secret（如 `NPM_TOKEN`）。否则可以移除或修改该流程以匹配实际发布方式。

## 贡献
欢迎通过 Issues/PR 提交改进建议。若要本地测试工作流：
1. 在 `.github/workflows/` 中编辑或新增工作流。 
2. 使用 `gh` CLI 结合仓库 secrets 在测试分支上运行。 

## 许可证
仓库当前没有在工作流中声明许可证信息。如需我帮你添加 `LICENSE` 文件，请告知你希望使用的许可协议（例如 MIT、Apache-2.0）。

---

如果你希望我把 README 内容调整为英文版、为 README 添加更详细的示例（比如完整的 `gh workflow run` 命令示例）或将 README 同步到仓库的特定分支，请告诉我我要做的下一步。
