# 05-moe-bmm1-fused-gather.py — Code Analysis / 代码分析

## Source / 来源
- **Path:** `python/examples/gluon/05-moe-bmm1-fused-gather.py`
- **EN:** Blackwell-focused MoE BMM1 example that fuses routed activation gather, block-scaled matrix multiply, bias addition, SwiGLU activation, and packed FP8 output storage for expert-parallel workloads.
- **CN:** 该文件是面向 Blackwell 的 MoE BMM1 示例，把路由激活的 gather、分块缩放矩阵乘、bias 加法、SwiGLU 激活以及打包后的 FP8 输出写回融合到一个适用于 expert-parallel 场景的内核中。

## Line-by-Line Analysis / 逐行分析
### Lines 1-41
- **EN:** The imports show that this is both a kernel example and a full experiment harness. Besides Triton/Gluon, it depends heavily on `triton_kernels` utilities for precision metadata, ragged routing metadata, tensor wrappers/layouts, top-k routing, reference matmul, and testing helpers.
- **CN:** 导入部分表明这不仅是一个内核示例，也是一个完整实验脚手架。除了 Triton/Gluon 外，它还大量依赖 `triton_kernels` 中的精度元数据、ragged 路由元数据、张量包装/布局、top-k 路由、参考 matmul 和测试辅助工具。

### Lines 48-128
- **EN:** `advance`, `unpack_block_schedule`, `banded_row_major`, and `apply_block_schedule` implement a block scheduler for ragged expert batches. `unswizzle_mx_scale` restores logical scale matrices from the packed MX format, while the barrier allocators build reusable ring buffers for producer/consumer synchronization.
- **CN:** `advance`、`unpack_block_schedule`、`banded_row_major` 和 `apply_block_schedule` 为 ragged expert batch 实现了块级调度器。`unswizzle_mx_scale` 负责从打包后的 MX 格式恢复逻辑 scale 矩阵，而 barrier 分配辅助函数则构造可复用的生产者/消费者环形同步结构。

### Lines 131-191
- **EN:** `pack_e4m3x2`, `pack_u16x2`, `pack_fp8x4`, `_split_m`, `_split_m_float2`, and `split_m_subtiles` are output-epilogue helpers. They use inline PTX and custom `float2` packing so the epilogue can convert accumulator fragments into packed FP8 efficiently, including optional M-dimension subtiling.
- **CN:** `pack_e4m3x2`、`pack_u16x2`、`pack_fp8x4`、`_split_m`、`_split_m_float2` 和 `split_m_subtiles` 是输出 epilogue 的辅助函数。它们利用内联 PTX 和自定义 `float2` 打包方式，把累加器片段高效转换为打包 FP8，同时支持可选的 M 维子分块。

### Lines 194-269
- **EN:** `PartitionArgs` is the central kernel state object. It holds tensor descriptors, raw pointers for bias/gather/scales/output, all staging buffers and barriers, launch/grid metadata, shape constants, fusion parameters, and tuning switches such as multicast usage, gather-index reuse, and inline input release.
- **CN:** `PartitionArgs` 是核心内核状态对象。它统一保存张量描述符、bias/gather/scale/output 的原始指针、所有流水线缓冲区与 barrier、launch/grid 元数据、形状常量，以及 multicast、gather 索引复用、内联输入释放等调优开关。

### Lines 272-352
- **EN:** `issue_activation_tile` and `load_activations` implement the routed-input side of the pipeline. For each scheduled block they look up the corresponding expert slice, gather token rows from `gather_indx_ptr`, optionally reuse cached gather indices when the same `(pid_m, slice)` repeats, and launch asynchronous TMA gather operations for every K tile.
- **CN:** `issue_activation_tile` 和 `load_activations` 实现了路由输入一侧的流水线。对于每个调度到的 block，它们查找对应的 expert slice，从 `gather_indx_ptr` 中收集 token 行；当相同 `(pid_m, slice)` 重复出现时还可以复用缓存索引；随后对每个 K tile 发起异步 TMA gather。

### Lines 354-389
- **EN:** `load_weights` is the weight-side producer. It walks the same block schedule, loads expert-local weight tiles plus their MX scales, and can multicast scale tiles across CTAs when the chosen 2CTA layout allows broadcast reuse.
- **CN:** `load_weights` 是权重侧的生产者。它沿用同一 block schedule，加载本地 expert 的权重 tile 及其 MX scale；在 2CTA 布局支持广播复用时，还可以把 scale tile 进行 multicast。

### Lines 391-444
- **EN:** `mma_partition` is the compute engine. It waits on activation and weight readiness barriers, unswizzles weight scales into tensor memory, feeds the activation tile, weight tile, and both scale tensors into `blackwell.tcgen05_mma_scaled`, and optionally releases input buffers inline instead of committing them afterward.
- **CN:** `mma_partition` 是计算引擎。它等待激活和权重就绪 barrier，把权重 scale 反 swizzle 后写入 tensor memory，再把激活 tile、权重 tile 以及两个 scale 张量一起送入 `blackwell.tcgen05_mma_scaled`；根据调优选项，它还可以在 MMA 内联释放输入缓冲，而不是事后再 commit。

### Lines 447-529
- **EN:** `store_packed_out`, `_swiglu_step1`, `_swiglu_step2`, `pack_fp8_out_fragment`, `get_store_layout`, and `epilogue_direct_store` implement the fused epilogue. Accumulator fragments are bias/scale adjusted, transformed through SwiGLU, converted to packed FP8, and written directly to the output buffer using a layout tailored for the packed storage format.
- **CN:** `store_packed_out`、`_swiglu_step1`、`_swiglu_step2`、`pack_fp8_out_fragment`、`get_store_layout` 和 `epilogue_direct_store` 共同实现了融合 epilogue。累加器片段先经过 bias/scale 调整和 SwiGLU 变换，再转换为打包 FP8，并以适配该打包存储格式的布局直接写回输出缓冲区。

### Lines 533-612
- **EN:** `apply_bias_and_scale` and `epilogue_partition` form the consumer side for accumulator buffers. They load per-expert bias, apply flexpoint-style global scaling (`x_scale`, `w_scale`, `out_scale`), reshape accumulator tiles into the packing-friendly `float2` layout, and store the reduced output width `BLOCK_N // REDUCTION_N`.
- **CN:** `apply_bias_and_scale` 与 `epilogue_partition` 构成累加器缓冲区的消费端。它们加载每个 expert 的 bias，应用 flexpoint 风格的全局缩放（`x_scale`、`w_scale`、`out_scale`），把累加器 tile 重排成便于打包的 `float2` 布局，并按 `BLOCK_N // REDUCTION_N` 的输出宽度进行存储。

### Lines 616-804
- **EN:** `ws_matmul_kernel` is the fused warp-specialized kernel. It validates the CTA mode, derives runtime grid sizes from ragged metadata, allocates activation/weight/shared-memory rings plus tensor-memory scale and accumulator buffers, seeds activation scales with a constant encoded scale tile, packages all state into `PartitionArgs`, and then launches four partitions: epilogue, activation loader, weight loader, and MMA.
- **CN:** `ws_matmul_kernel` 是融合后的 warp-specialized 主内核。它会校验 CTA 模式，根据 ragged 元数据推导运行时网格大小，分配激活/权重共享内存环形缓冲区以及 tensor memory 中的 scale 与累加器缓冲区，用一个常量编码 scale tile 初始化激活 scale，然后把所有状态打包进 `PartitionArgs`，最后启动四个 partition：epilogue、激活加载、权重加载和 MMA。

### Lines 812-867
- **EN:** `make_tensor_descriptor` bridges high-level tensor objects to Gluon/NVMMA descriptors. It knows how to describe ordinary PyTorch tensors, custom FP4 tensor wrappers, raw UINT8 scale tensors, FP32 bias/output tensors, and FP8 tensors with backend-specific swizzle rules.
- **CN:** `make_tensor_descriptor` 负责把高层张量对象转换为 Gluon/NVMMA 描述符。它同时处理普通 PyTorch 张量、自定义 FP4 张量包装、原始 UINT8 scale 张量、FP32 bias/output 张量，以及带有后端特定 swizzle 规则的 FP8 张量。

### Lines 870-914
- **EN:** `KernelConfig` defines the tunable shape of the kernel: tile sizes, CTA count, buffering depths, warp split, register budgets, occupancy hints, epilogue subtiling, multicast toggles, and helper methods to estimate shared-memory demand.
- **CN:** `KernelConfig` 定义了该内核的可调形状：tile 尺寸、CTA 数量、缓冲深度、warp 划分、寄存器预算、occupancy 提示、epilogue 子分块、多播开关，以及若干用于估算共享内存需求的辅助方法。

### Lines 917-1097
- **EN:** The selector family (`_select_tiny16_config` through `_select_high128_config`) maps expected slice size to hand-tuned launch parameters. The code favors 2CTA kernels across many slice regimes, then tweaks band width, buffer counts, register caps, gather reuse, and multicast behavior for specific hot sizes.
- **CN:** 选择器族（`_select_tiny16_config` 到 `_select_high128_config`）会根据期望的 slice size 映射到手工调优过的启动参数。代码在许多 slice 区间中优先采用 2CTA 内核，并针对热点尺寸进一步调整 band 宽度、缓冲数量、寄存器上限、gather 复用和 multicast 行为。

### Lines 1100-1240
- **EN:** `matmul` is the host-side public API. It validates the fused activation contract (`swiglu` only), derives launch configuration from ragged routing metadata and GPU SM count, constructs tensor descriptors for activations/weights/scales/output, and launches `ws_matmul_kernel` with all structural and precision metadata.
- **CN:** `matmul` 是宿主侧公开 API。它先校验融合激活契约（目前仅支持 `swiglu`），再根据 ragged 路由元数据和 GPU SM 数量推导启动配置，为激活/权重/scale/输出构造张量描述符，并把所有结构与精度元数据传入 `ws_matmul_kernel`。

### Lines 1248-1470
- **EN:** The helper layer defines model/problem configuration (`MLPConfig`), allocates randomized FP8/FP4 inputs, synthesizes production-like MoE routing distributions, prepares `PreparedCase` objects, builds a `PrecisionConfig`, estimates FLOPs and bytes for benchmarking, and provides wrappers to run either the example kernel or the reference implementation.
- **CN:** 辅助层定义了模型/问题配置（`MLPConfig`），分配随机 FP8/FP4 输入，合成接近生产环境的 MoE 路由分布，构造 `PreparedCase`，生成 `PrecisionConfig`，估算基准测试所需的 FLOPs 与字节数，并提供运行示例内核或参考实现的统一封装。

### Lines 1477-1519
- **EN:** The unit test targets a GPT-OSS-120B-like MoE setup. It compares both output values and output scaling metadata against the reference provider, reflecting that correctness here includes flexpoint scale propagation, not just tensor elements.
- **CN:** 单元测试面向一个类似 GPT-OSS-120B 的 MoE 配置。它不仅比较输出张量值，还比较输出 scale 元数据，因为这里的正确性不仅包含元素结果，也包含 flexpoint scale 的传播是否一致。

### Lines 1526-1583
- **EN:** The benchmark driver prints model metadata, iterates through batch sizes, prepares one case for the example kernel and one for the reference kernel, and reports both compute throughput (TFLOPS) and effective bandwidth (TBPS).
- **CN:** 基准驱动会打印模型元数据，遍历不同 batch size，分别为示例内核和参考内核准备测试用例，并同时报告计算吞吐（TFLOPS）与有效带宽（TBPS）。

## Key Concepts / 关键概念
- **EN:** **Fused-gather MoE BMM1:** the kernel directly gathers routed token rows for each expert slice instead of materializing a separate dense regrouped activation tensor.
- **CN:** **融合 gather 的 MoE BMM1：** 内核直接为每个 expert slice 收集被路由的 token 行，而不是先物化一个单独的致密重排激活张量。
- **EN:** **Quantized block-scaled compute:** weights are stored in FP4/MXFP form with block scales, activations are FP8, and the MMA path reconstructs scale-aware products in hardware.
- **CN:** **量化分块缩放计算：** 权重以带块级 scale 的 FP4/MXFP 形式存储，激活为 FP8，而 MMA 路径在硬件中完成感知 scale 的乘加重建。
- **EN:** **Fused epilogue:** bias, SwiGLU, output scaling, packing, and direct store are merged into the same kernel to avoid extra memory traffic.
- **CN:** **融合 epilogue：** bias、SwiGLU、输出缩放、打包与直接写回都被合并进同一个内核，以减少额外内存流量。
- **EN:** **Ragged scheduling:** the block schedule follows expert slice sizes instead of a uniform dense M dimension, which is crucial for MoE imbalance.
- **CN:** **Ragged 调度：** block schedule 依据各 expert slice 的实际大小而不是统一致密的 M 维进行调度，这对处理 MoE 的负载不均衡至关重要。

## Dependencies / 依赖关系
- **EN:** `torch`, `pytest`, `triton`, and `triton.experimental.gluon` provide tensor runtime support, testing, kernel compilation, and Gluon DSL primitives.
- **CN:** `torch`、`pytest`、`triton` 与 `triton.experimental.gluon` 提供张量运行时、测试、内核编译以及 Gluon DSL 原语。
- **EN:** Blackwell-specific modules (`blackwell`, `tma`, `mbarrier`, `float2`) are fundamental to the implementation; the kernel is architecture-specialized rather than backend-generic.
- **CN:** Blackwell 专用模块（`blackwell`、`tma`、`mbarrier`、`float2`）是实现基础；该内核是架构特化版本，而非后端无关实现。
- **EN:** `triton_kernels` modules supply routing metadata, tensor layout wrappers, precision/flexpoint metadata, fused activation definitions, reference matmul, and validation helpers.
- **CN:** `triton_kernels` 系列模块提供路由元数据、张量布局包装、精度/flexpoint 元数据、融合激活定义、参考 matmul 以及校验辅助工具。
- **EN:** The host wrapper depends on ragged metadata and expert-distribution utilities to transform a model-level MoE problem into per-expert block schedules for the device kernel.
- **CN:** 宿主侧包装依赖 ragged 元数据和 expert 分布工具，把模型层面的 MoE 问题转换成设备内核可执行的按 expert 分块调度。
