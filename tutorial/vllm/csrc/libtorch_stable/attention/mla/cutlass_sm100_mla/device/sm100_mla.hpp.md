# sm100_mla.hpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/libtorch_stable/attention/mla/cutlass_sm100_mla/device/sm100_mla.hpp`
- **Repository**: vllm-project/vllm
- **Purpose**: Universal CUTLASS device wrapper that combines the SM100 MLA main kernel with an optional split-KV reduction kernel and launch-time resource management. / 通用 CUTLASS 设备层包装器，将 SM100 MLA 主内核、可选的 split-KV 归约内核以及启动时资源管理整合在一起。

## Line-by-Line Analysis / 逐行分析
### Template wrapper and reduction binding
```cpp
template<
    class Kernel_
>
class MLA {
public:

  using Kernel = Kernel_;

  using ReductionKernel = cutlass::fmha::kernel::Sm100FmhaMlaReductionKernel<
      typename Kernel::ElementOut,
      typename Kernel::ElementAcc,
      typename Kernel::ElementAcc,
      Kernel::TileShapeH::value,
      Kernel::TileShapeL::value,
      256 /*Max split*/
  >;
```
**EN:** `MLA` is a high-level device API around a concrete CUTLASS kernel type. It derives the reduction kernel directly from the main kernel's output, accumulator, head count, and latent dimension, so the two stages stay type-compatible.
**CN:** `MLA` 是围绕具体 CUTLASS 内核类型构建的高层设备 API。它直接从主内核的输出类型、累加类型、头数和 latent 维度推导出归约内核，确保两阶段在类型和形状上保持一致。

### Automatic split-KV heuristic
```cpp
static void set_split_kv (KernelArguments& args) {
  if (args.split_kv >= 1) return;
  auto [H, K, D, B] = args.problem_shape;
  int sm_count = args.hw_info.sm_count;
  float seq_length_k = static_cast<float>(K) / 1024.0f;
  int max_splits = 1;

  if (B <= 4 && seq_length_k >= 16) {
    max_splits = 16;
  }
  else if (B <= 8 && seq_length_k >= 4) {
    max_splits = 8;
  }
  ...
  int sms_per_batch = max(1, sm_count / B);
  int split_heur = min(max_splits, sms_per_batch);
  int waves = ceil_div(B * split_heur, sm_count);
  int k_waves = ceil_div(max_splits, split_heur);
  int split_wave_aware = ceil_div(max_splits, k_waves);
  args.split_kv = split_wave_aware;
}
```
**EN:** This helper chooses `split_kv` automatically when the caller leaves it unset. The heuristic uses batch size, sequence length, and SM count to trade off more parallelism on long contexts against wave efficiency on the GPU.
**CN:** 当调用方未显式设置 `split_kv` 时，这个辅助函数会自动选择分裂数。启发式同时考虑 batch 大小、序列长度和 SM 数量，在长上下文下提高并行度，同时尽量保持 GPU wave 调度效率。

### Initialization and parameter lowering
```cpp
Status
initialize(Arguments const& args, void* workspace = nullptr, cudaStream_t stream = nullptr) {
  Status status = Kernel::initialize_workspace(args, workspace, stream);
  if (status != Status::kSuccess) {
    return status;
  }
  status = ReductionKernel::initialize_workspace(to_reduction_args(args), workspace, stream);
  if (status != Status::kSuccess) {
    return status;
  }
  KernelParams kernel_params = Kernel::to_underlying_arguments(args, workspace);

  ReductionArguments reduction_args = to_reduction_args(args);
  if (reduction_args.split_kv > 1) {
    reduction_args.ptr_oaccum   = kernel_params.epilogue.ptr_o_acc;
    reduction_args.ptr_lseaccum = kernel_params.epilogue.ptr_lse_acc;
  }
  ReductionParams reduction_params = ReductionKernel::to_underlying_arguments(reduction_args, workspace);
  params_ = Params {kernel_params, reduction_params};
```
**EN:** `initialize()` lowers user-facing arguments into launch-ready parameter structs for both stages. When split-KV is enabled, it wires the reduction kernel to the main kernel's temporary output and LSE accumulation buffers in workspace.
**CN:** `initialize()` 会把用户层参数下沉为两个阶段都可直接启动的底层参数结构。当启用 split-KV 时，它会把归约内核连接到主内核在 workspace 中写入的临时输出与 LSE 累加缓冲区。

### Launch path
```cpp
static Status
run(Params& params, cudaStream_t stream = nullptr) {
  dim3 const block = Kernel::get_block_shape();
  dim3 const grid = Kernel::get_grid_shape(params.fmha_params);
  ...
  if constexpr(Kernel::ArchTag::kMinComputeCapability >= 90) {
    dim3 cluster(cute::size<0>(typename Kernel::ClusterShape{}),
                 cute::size<1>(typename Kernel::ClusterShape{}),
                 cute::size<2>(typename Kernel::ClusterShape{}));
    ...
    launch_result = ClusterLauncher::launch(grid, cluster, block, smem_size, stream, kernel, kernel_params);
  }
  else {
    device_kernel<Kernel><<<grid, block, smem_size, stream>>>(params.fmha_params);
  }
  ...
  if (params.reduction_params.split_kv > 1) {
    device_kernel<ReductionKernel><<<grid, block, 0, stream>>>(params.reduction_params);
```
**EN:** The wrapper first launches the main MLA kernel, using cluster launch on modern architectures, and then conditionally launches the reduction kernel only when multiple KV splits produced partial results. This keeps the fast path lean for `split_kv == 1`.
**CN:** 包装器先启动主 MLA 内核；在新架构上使用 cluster launch；随后仅在存在多个 KV split 产生部分结果时才启动归约内核。这样 `split_kv == 1` 的快速路径不会承担额外开销。

## Key Concepts / 关键概念
- **Argument lowering / 参数下沉**: Converts ergonomic host-side arguments into CUTLASS kernel parameter layouts.
- **Split-KV scheduling / Split-KV 调度**: Chooses or honors a KV partition count to improve long-context parallelism.
- **Two-stage execution / 两阶段执行**: Main attention kernel writes partial accumulations, then a reduction kernel merges them.
- **Occupancy-aware launch / 占用率感知启动**: Configures dynamic shared memory and queries active blocks before launch.

## Dependencies / 依赖关系
- `kernel/sm100_fmha_mla_tma_warpspecialized.hpp`: defines the main SM100 MLA kernel type.
- `kernel/sm100_fmha_mla_reduction.hpp`: defines the split-KV merge kernel.
- CUTLASS runtime helpers such as `device_kernel`, `ClusterLauncher`, and `KernelHardwareInfo`.
