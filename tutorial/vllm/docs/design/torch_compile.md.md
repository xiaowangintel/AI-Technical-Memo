# torch_compile.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/design/torch_compile.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
- **EN:** The document explains that `torch.compile` is a default and central part of vLLM V1, with compilation artifacts stored in a cache directory derived from vLLM configs, PyTorch configs, and traced source files. **CN:** 文档说明 `torch.compile` 是 vLLM V1 的默认且核心能力，编译产物会存放在一个由 vLLM 配置、PyTorch 配置和被追踪源码共同决定的缓存目录中。
- **EN:** A key design promise is that vLLM finishes compilation before serving requests, preventing latency spikes from first-request compilation. **CN:** 一个关键设计承诺是：vLLM 会在开始服务请求前完成编译，从而避免首次请求触发编译带来的延迟尖峰。
- **EN:** The dynamic-shape section compares `BACKED`, `UNBACKED`, and `BACKED_SIZE_OBLIVIOUS`, showing how users can trade safety against extra guards versus performance opportunities. **CN:** 动态形状部分比较了 `BACKED`、`UNBACKED` 和 `BACKED_SIZE_OBLIVIOUS`，展示了用户如何在防止额外 guard 的安全性与性能潜力之间做权衡。
- **EN:** The compilation walkthrough covers Dynamo tracing, saved transformed/computation graph files, graph splitting around attention, and later Inductor compilation plus cache reuse. **CN:** 编译流程讲解覆盖了 Dynamo tracing、保存的 transformed/computation graph 文件、围绕 attention 的图切分，以及后续的 Inductor 编译与缓存复用。
- **EN:** It also describes shape-specialized compilation with optional autotuning, which can improve performance at the cost of longer first-run compile time. **CN:** 文档还说明了针对特定 shape 的专门编译及可选 autotuning，这能提升性能，但会增加首次运行时的编译时间。
- **EN:** For runtime execution, vLLM uses piecewise CUDA graph capture that excludes attention by default but can be configured or expanded if the attention backend supports full capture. **CN:** 在运行时，vLLM 使用分段式 CUDA graph capture，默认把 attention 排除在外；如果 attention backend 支持，也可以进行配置甚至扩展为完整捕获。

## Key Concepts / 关键概念
- **EN:** **Compilation cache:** cached artifacts can be copied across deployments, and cache safety relies on hashing both configuration and traced-code inputs. **CN:** **编译缓存：** 编译产物可在部署间复制复用，而其安全性依赖于对配置和被追踪代码输入的联合哈希。
- **EN:** **Dynamic-shape modes:** the three modes mainly differ in how strongly they avoid guards and how much optimization freedom they leave to PyTorch. **CN:** **动态形状模式：** 三种模式的主要区别在于它们对 guard 的规避强度，以及给 PyTorch 保留多少优化空间。
- **EN:** **Attention as a custom op:** attention is wrapped as `torch.ops.vllm.unified_attention_with_output` so the rest of the graph can still be compiled as a full graph. **CN:** **Attention 作为自定义算子：** attention 被封装成 `torch.ops.vllm.unified_attention_with_output`，从而让图的其余部分仍可按完整图进行编译。
- **EN:** **Subgraph compilation:** splitting around attention creates reusable graph pieces such as the first layer, repeated middle layers, and final layer. **CN:** **子图编译：** 围绕 attention 的切分会产生可复用的图片段，例如首层、重复的中间层和末层。
- **EN:** **Piecewise cudagraphs:** CUDA graphs are captured for attention-adjacent compute regions, balancing replay benefits with attention flexibility. **CN:** **分段 CUDA 图：** CUDA graph 只覆盖 attention 之间的计算区域，在重放收益与 attention 灵活性之间做平衡。
