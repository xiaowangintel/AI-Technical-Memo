# model_runner_v2.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/design/model_runner_v2.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
- **EN:** This design document presents Model Runner V2 (MRV2) as a ground-up redesign of V1, motivated by technical debt, lessons from async scheduling, Triton, Gumbel-max sampling, and CUDA features such as UVA.  
  **CN:** 本设计文档将 Model Runner V2（MRV2）描述为对 V1 的自底向上重构，其动机来自技术债、异步调度经验、Triton、Gumbel-max 采样以及 UVA 等 CUDA 特性的积累。
- **EN:** MRV2 reworks persistent batching by decoupling persistent state tensors from per-step model inputs, assigning stable rows to active requests, and gathering step inputs from GPU-resident state instead of constantly rebuilding or reordering tensors.  
  **CN:** MRV2 重新设计了持久批处理：将持久状态张量与每一步的模型输入解耦，为活动请求分配稳定行位，并从驻留 GPU 的状态中收集 step 输入，而不是反复重建或重排张量。
- **EN:** The document is explicitly async-first: it avoids CPU/GPU synchronization barriers, replaces shared pinned-buffer races with temporary pinned copies, and introduces `StagedWriteTensor` to apply sparse/ragged GPU updates efficiently.  
  **CN:** 文档明确采取异步优先思路：避免 CPU/GPU 同步屏障，用临时 pinned 副本取代共享 pinned 缓冲区的竞争，并引入 `StagedWriteTensor` 高效应用稀疏/不规则的 GPU 更新。
- **EN:** Beyond execution flow, MRV2 pushes more work onto the GPU via Triton kernels, adds a Triton-native sampler, simplifies `dummy_run`, uses explicit CUDA graph management, and emphasizes modularity over the monolithic V1 runner.  
  **CN:** 除了执行流程外，MRV2 还通过 Triton kernel 把更多工作下沉到 GPU，引入 Triton 原生采样器，简化 `dummy_run`，显式管理 CUDA Graph，并用模块化结构替代 V1 的单体式 runner。

## Key Concepts / 关键概念
- **EN:** **Persistent batch redesign** — request state persists in fixed slots, while actual step inputs are gathered separately for flexibility.  
  **CN:** **持久批重构** —— 请求状态保存在固定槽位中，而真正的 step 输入单独 gather，以提升灵活性。
- **EN:** **Async-first execution** — CPU prepares future work while GPU runs the current step, so synchronization points must be aggressively removed.  
  **CN:** **异步优先执行** —— CPU 在 GPU 执行当前 step 时准备后续工作，因此必须尽量移除同步点。
- **EN:** **`StagedWriteTensor`** — a GPU-base, CPU-staged diff mechanism for applying many small writes with low copy and launch overhead.  
  **CN:** **`StagedWriteTensor`** —— 以 GPU 为基底、CPU 暂存差异的机制，可用较低拷贝与启动开销应用大量小写入。
- **EN:** **GPU-native metadata preparation** — Triton/UVA paths compute or access input metadata closer to execution, reducing Python overhead and data duplication.  
  **CN:** **GPU 原生元数据准备** —— 借助 Triton/UVA 在更接近执行的位置计算或访问输入元数据，减少 Python 开销与数据复制。
- **EN:** **Triton-native sampling and explicit CUDA graphs** — MRV2 improves memory behavior for sampling while making graph capture/launch lifecycle more explicit.  
  **CN:** **Triton 原生采样与显式 CUDA Graph** —— MRV2 改善了采样阶段的内存行为，并让图捕获/启动生命周期更加显式。
