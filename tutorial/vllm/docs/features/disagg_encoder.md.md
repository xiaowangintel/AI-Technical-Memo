# disagg_encoder.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/features/disagg_encoder.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
- **EN:** This doc explains disaggregated encoder serving for multimodal LLMs: the vision encoder runs separately from the prefill/decoder stage. It highlights three concrete benefits from the source text: independent scaling, lower TTFT, and cross-process reuse/caching of encoder outputs.
- **CN:** 本文档介绍了多模态 LLM 的解耦式编码器服务：视觉编码器与 prefill / decoder 阶段分开运行。原文明确强调三点收益：独立扩缩容、更低的首 token 延迟（TTFT），以及编码器输出的跨进程复用 / 缓存。
- **EN:** The structure is practical rather than theoretical: it points to ready-to-run scripts for 1E1PD and 1E1P1D setups, references tests under `tests/v1/ec_connector`, and maps implementation to `vllm/distributed/ec_transfer`.
- **CN:** 文档结构偏实践而非纯理论：给出了 1E1PD 与 1E1P1D 的可运行脚本，引用了 `tests/v1/ec_connector` 测试目录，并指出实现代码位于 `vllm/distributed/ec_transfer`。
- **EN:** In development details, the key abstraction is `ECConnector`, split into scheduler and worker roles. The doc also notes that in the E->P->D path, Prefill executes one step and then transfers KV cache onward to Decode, with the P-to-D KV transfer facilitated in the example via `NixlConnector` and a toy proxy server.
- **CN:** 在开发细节中，核心抽象是 `ECConnector`，分为 scheduler 与 worker 两类角色。文档还说明在 E->P->D 路径中，Prefill 先执行一步，再把 KV cache 传给 Decode；示例里该 P 到 D 的 KV 传输由 `NixlConnector` 和 toy proxy server 协助完成。

## Key Concepts / 关键概念
- **EN:** Disaggregation separates vision encoding from language prefill/decode.  
  **CN:** 解耦部署将视觉编码与语言 prefill / decode 分离。
- **EN:** The doc’s stated goals are fine-grained scaling, lower TTFT, and shared encoder-output reuse.  
  **CN:** 文档明确给出的目标是细粒度扩缩容、更低 TTFT，以及共享式编码器输出复用。
- **EN:** `ECConnector` handles encoder-cache retrieval through scheduler and worker roles.  
  **CN:** `ECConnector` 通过 scheduler / worker 两类角色处理 encoder-cache 获取。
- **EN:** Example scripts cover both encoder→PD and encoder→prefill→decode workflows.  
  **CN:** 示例脚本同时覆盖 encoder→PD 与 encoder→prefill→decode 两种流程。
