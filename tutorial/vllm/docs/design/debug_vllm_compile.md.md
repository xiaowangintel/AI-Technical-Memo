# debug_vllm_compile.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/design/debug_vllm_compile.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
- **EN:** This document is a troubleshooting guide for the layered vLLM–`torch.compile` stack, not just for compilation itself but also for CUDAGraphs, custom IR passes, and compile caching. **CN:** 本文是一份面向 vLLM–`torch.compile` 分层栈的排障指南，覆盖的不只是编译本身，还包括 CUDAGraphs、自定义 IR pass 与编译缓存。
- **EN:** It starts with a practical isolation matrix, mapping online/offline flags to subsystems so users can disable only the failing layer instead of falling back to full eager mode immediately. **CN:** 文档一开始就给出实用的隔离矩阵，把在线/离线 flag 映射到各子系统，让用户能只关闭出问题的层，而不是立刻完全退回 eager 模式。
- **EN:** `tlparse` is presented as the primary observability tool, because it exposes TorchDynamo capture, TorchInductor output, and compilation artifacts per rank. **CN:** `tlparse` 被作为主要观测工具，因为它能展示按 rank 划分的 TorchDynamo 捕获过程、TorchInductor 输出与编译产物。
- **EN:** The guide then walks through failure classes in order: graph breaks in TorchDynamo, dynamic-shape capture issues, guard violations, TorchInductor miscompiles, compile-cache bugs, and CUDAGraph misuse. **CN:** 随后文档按层次梳理常见故障：TorchDynamo 的 graph break、动态形状捕获问题、guard 违规、TorchInductor 误编译、编译缓存错误以及 CUDAGraph 误用。
- **EN:** Overall, the emphasis is on narrowing the blast radius of failures while preserving as much performance as possible. **CN:** 总体来看，文档强调的是在尽量保留性能的前提下，把故障影响范围收窄到最小。

## Key Concepts / 关键概念
- **EN:** **Isolation flags** — `--enforce-eager`, `-cc.mode`, `-cc.cudagraph_mode`, `-cc.backend=eager`, and `ir_enable_torch_wrap` each disable a different layer. **CN:** **隔离开关**——`--enforce-eager`、`-cc.mode`、`-cc.cudagraph_mode`、`-cc.backend=eager` 与 `ir_enable_torch_wrap` 分别对应关闭不同层级。
- **EN:** **`tlparse` workflow** — `TORCH_TRACE=<dir>` plus `tlparse` is the recommended way to capture evidence for debugging and bug reports. **CN:** **`tlparse` 工作流**——通过 `TORCH_TRACE=<dir>` 配合 `tlparse`，可以系统化收集调试证据并附在 bug report 中。
- **EN:** **TorchDynamo graph breaks** — full-graph capture can fail on unsupported Python patterns, so model code may need rewriting or custom operators. **CN:** **TorchDynamo graph break**——全图捕获会因不支持的 Python 模式而失败，因此模型代码可能需要重写或封装成自定义算子。
- **EN:** **Dynamic-shape guards** — vLLM assumes guards are droppable, so constraint violations and stricter dynamic-shape modes are central debugging signals. **CN:** **动态形状 guard**——vLLM 假设这些 guard 可以被去除，因此约束冲突与更严格的动态形状模式是关键调试信号。
- **EN:** **TorchInductor inspection** — runtime asserts, output-code logging, and editable cache artifacts help diagnose bad generated kernels. **CN:** **TorchInductor 检查**——运行时断言、输出代码日志以及可编辑的缓存产物可帮助定位错误生成的 kernel。
- **EN:** **Compile cache and CUDAGraph limits** — cache invalidation bugs and CUDAGraph memory/replay rules are treated as distinct failure domains. **CN:** **编译缓存与 CUDAGraph 限制**——缓存失效错误与 CUDAGraph 的内存/回放约束被视为两个独立的故障域。
