AI Review sandbox
实验方案：GitHub Actions 手动触发 PR-Agent，调用指定的国产模型兼容 API，在 PR 中发布审查结果。 尚未开启自动审查、代码修改或自动合并。当前 review 命令以审查摘要为主，逐行建议可在后续实验中增加 improve 命令。

1. 配置模型
在本仓库 Settings → Secrets and variables → Actions 中配置：

类型	名称	内容
Secret	AI_API_KEY	国产模型 API Key
Variable	AI_API_BASE	服务商文档给出的 HTTPS 兼容 API 基础地址，包含所需版本路径，不填 chat/completions 完整地址
Variable	AI_MODEL	服务商实际支持的模型 ID，不加 openai/ 前缀
直接填写国产服务商地址。配置中 openai/ 和 OPENAI__* 是 PR-Agent/LiteLLM 的兼容接口命名，并不代表调用 OpenAI。 没有默认服务地址、默认模型或备用模型；必须显式配置。不要填写国外模型或转发到国外模型的网关。 审查输入预算暂设 16000 tokens，输出上限 2000；选择支持该参数的模型，需要时再按服务商文档调整。

2. 发布工作流
将 .github/workflows/ai-review.yml 和本说明提交到 GitHub 默认分支。 工作流出现在 Actions 页面后才可以手动运行。首次接入不需要安装 GitHub App，不需要另配 GitHub PAT。 github.token 由 GitHub 自动提供，只授予读取代码和写 PR 的权限。

3. 创建并审查测试 PR
从默认分支创建 experiment/review-case-1 分支，加入少量示例代码并提交 PR。
在 Actions → AI Review (sandbox) → Run workflow 中选择默认分支、填写 PR 编号。
查看任务日志和 PR 中 github-actions[bot] 的审查结果。
修复问题并推送，再手动运行，观察是否正确更新审查意见。
建议分别测试：明确边界错误、遗漏调用点、正确修复、无错误的格式修改。记录有效发现、误报、耗时和模型账单。 不要为了验证效果故意向模型提供标准答案。手动触发也会消耗模型费用。

当前边界
固定 PR-Agent 源码提交和 setup-python Action 提交；Python 依赖尚未锁全，运行环境不保证完全可复现。
通过 GitHub API 读取 PR，不检出或执行 PR 的代码。
禁止加载 PR 所带的 .pr_agent.toml，模型路由与规则集中放在工作流中。
只配置指定模型，关闭备用模型；实际服务路由仍由所配置的 API 服务商决定。
没有进行真实模型调用或端到端验证。成功接入后，再添加自动触发、逐行建议和领域规则。
参考：PR-Agent GitHub 集成、兼容模型配置。
