---
name: lumi-api
description: "Use this skill when the user wants to manage social media content via Lumi — uploading videos, publishing to TikTok/YouTube/Instagram, translating or dubbing content, checking account analytics, listing connected accounts, or managing localization tasks. All API calls use curl with Bearer token authentication."
metadata:
  {
    "openclaw":
      {
        "homepage": "https://lumipath.cn",
        "requires": { "env": ["LUMI_API_KEY"] },
        "primaryEnv": "LUMI_API_KEY",
      },
  }
---

# 权限声明
# SECURITY MANIFEST:
# - Allowed to read: {baseDir}/README.md, {baseDir}/references/*.json
# - Allowed to make network requests to: https://lumipath.cn


## 工作流 (按需加载模式)

当用户提出请求时，请严格执行以下步骤：

1. **检查API密钥**：首先检查环境变量 `LUMI_API_KEY` 是否存在。如果不存在，提示用户设置 API 密钥：`export LUMI_API_KEY="lumi_your_key_here"`
2. **阅读README**：仔细阅读 `{baseDir}/README.md`，了解接口概览和认证方式。
3. **目录索引**：扫描 `{baseDir}/references/` 目录下的所有文件名，确定哪些 OpenAPI 定义文件与用户需求相关。
4. **精准读取**：仅读取选定的 `.json` 文件，分析其 `paths`、`parameters` 和 `requestBody`。其中 `paths` 内是一个对象，对象的 key 就是 path。
5. **构造请求**：使用 curl 执行请求。
   - **Base URL**: 统一使用 `https://lumipath.cn`（或从 JSON 的 `servers` 字段提取）。
   - **Auth**: 从环境变量 `LUMI_API_KEY` 中获取值，注入 Header：`Authorization: Bearer $LUMI_API_KEY`
   - 发布到社交平台前，始终先调用 `/api/v1/connections` 获取最新的 `connectionId`。
   - 启动 localization/repurpose 任务后，每隔 10-30 秒轮询任务状态，直到 `status=completed`。
   - `status=completed` 时，使用响应中的 `outputUrl` 作为视频 URL 进行后续发布。


## 注意事项
- **禁止全量加载**：除非用户请求涉及多个领域，否则禁止同时读取多个 JSON 文件。
- **参数校验**：在发起请求前，必须根据 OpenAPI 定义验证必填参数是否齐全。
- **错误处理**：当请求失败时，向用户显示友好的提示信息，并记录详细的错误日志。
- **API密钥配置**：用户需要自行设置环境变量 `LUMI_API_KEY`，例如：`export LUMI_API_KEY="lumi_your_key_here"`
