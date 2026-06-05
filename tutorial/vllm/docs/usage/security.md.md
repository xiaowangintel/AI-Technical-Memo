# security.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/usage/security.md`
- **Repository**: `vllm-project/vllm`

## Content Analysis / 内容分析
- **EN:** This security guide warns that several communication paths and control surfaces in vLLM are insecure by default. It combines network-isolation guidance, SSRF mitigation, firewall recommendations, and a detailed warning that API-key protection does not cover many sensitive endpoints.
- **CN:** 这份安全指南强调：vLLM 中若干通信链路和控制接口默认并不安全。文档同时给出了网络隔离建议、SSRF 缓解方法、防火墙策略，并详细提醒 API key 并不能覆盖许多敏感端点。

## Key Concepts / 关键概念
- **EN:** Inter-node traffic for PyTorch distributed, KV transfer, and parallel execution should be treated as unencrypted and unauthenticated.
  **CN:** 与 PyTorch 分布式、KV 传输及并行执行相关的节点间通信都应视为未加密、未认证。
- **EN:** Operators should explicitly configure communication IPs/ports and isolate them behind trusted networks and firewalls.
  **CN:** 运维方应显式配置通信所用 IP/端口，并通过可信网络与防火墙进行隔离。
- **EN:** `--allowed-media-domains` and redirect controls help reduce SSRF and unbounded download risk for media URLs.
  **CN:** `--allowed-media-domains` 及重定向控制有助于降低媒体 URL 带来的 SSRF 与超大下载风险。
- **EN:** The built-in API key only protects selected `/v1`/`/v2`/`/inference` routes, leaving many inference, operational, and development endpoints exposed unless a reverse proxy blocks them.
  **CN:** 内置 API key 只保护部分 `/v1`/`/v2`/`/inference` 路由；若没有反向代理做额外拦截，许多推理、运维和开发端点仍然暴露。
