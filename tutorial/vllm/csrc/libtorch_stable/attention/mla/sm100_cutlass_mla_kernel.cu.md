# sm100_cutlass_mla_kernel.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/libtorch_stable/attention/mla/sm100_cutlass_mla_kernel.cu`
- **Repository**: vllm-project/vllm
- **Purpose**: Stable-ABI CUDA entrypoints that build MLA kernel arguments, dispatch supported dtypes/layouts, and register SM100 decode operators with PyTorch. / 提供稳定 ABI 的 CUDA 入口：构建 MLA 内核参数、分发受支持的数据类型/布局，并向 PyTorch 注册 SM100 decode 算子。

## Line-by-Line Analysis / 逐行分析
### CUDA version guard
```cpp
#if !defined(CUDA_VERSION) || CUDA_VERSION < 12040
void sm100_cutlass_mla_decode(... ) {
  STD_TORCH_CHECK(false, "CUDA version must be >= 12.4 for cutlass_mla_decode");
}
int64_t sm100_cutlass_mla_get_workspace_size(...) {
  STD_TORCH_CHECK(false, "CUDA version must be >= 12.4 for cutlass_mla_get_workspace_size");
}
#else
```
**EN:** The file is only valid for CUDA 12.4+ because the SM100 CUTLASS path depends on new compiler/runtime support. Older toolchains expose the same symbols but fail fast with a clear error.
**CN:** 该文件仅在 CUDA 12.4+ 下有效，因为 SM100 的 CUTLASS 路径依赖较新的编译器与运行时支持。旧工具链虽然暴露相同符号，但会立即报出清晰错误。

### Kernel type assembly
```cpp
template <typename T, typename TOut, bool IsPaged128, typename PersistenceOption = IsPersistent<true>>
struct MlaSm100 {
  using Element = T;
  using ElementAcc = float;
  using ElementOut = TOut;
  ...
  using TileScheduler =
      std::conditional_t<PersistenceOption::value, Sm100MlaPersistentTileScheduler, Sm100MlaIndividualTileScheduler>;
  using FmhaKernel = cutlass::fmha::kernel::Sm100FmhaMlaKernelTmaWarpspecialized<
      TileShape,
      Element,
      ElementAcc,
      ElementOut,
      ElementAcc,
      TileScheduler,
      /*kIsCpAsync=*/!IsPaged128>;
```
**EN:** `MlaSm100` is a type-level factory. It selects the tile scheduler, output type, and cp.async/TMA path based on template parameters, then builds the concrete CUTLASS kernel and device wrapper types.
**CN:** `MlaSm100` 是一个类型层工厂。它根据模板参数选择 tile 调度器、输出类型以及 cp.async/TMA 路径，然后组合出具体的 CUTLASS 内核与设备包装器类型。

### Building runtime arguments from tensors
```cpp
cutlass::KernelHardwareInfo hw_info;
hw_info.device_id = q_nope.get_device_index();
hw_info.sm_count = cutlass::KernelHardwareInfo::query_device_multiprocessor_count(hw_info.device_id);
...
auto problem_shape = cute::make_tuple(TileShapeH{}, max_seq_len, TileShapeD{}, batches);
...
typename T::Fmha::Arguments arguments{
    problem_shape,
    {scale,
     Q_nope_ptr,
     stride_Q_nope,
     Q_pe_ptr,
     stride_Q_pe,
     C_ptr,
     stride_C,
     C_ptr + D_latent,
     stride_C,
     static_cast<int*>(seq_lens.data_ptr()),
     static_cast<int*>(page_table.data_ptr()),
```
**EN:** `args_from_options()` translates stable-ABI tensors into the exact CUTLASS argument layout: tensor pointers, strides, page-table metadata, problem shape, and SM information. It also computes the maximum sequence length from page geometry.
**CN:** `args_from_options()` 把稳定 ABI 的张量转换成 CUTLASS 所需的精确参数布局：包括张量指针、步长、页表元数据、问题形状以及 SM 信息。同时它还会根据分页几何信息计算最大序列长度。

### Launch and dtype dispatch
```cpp
typename MlaSm100Type::Fmha fmha;
auto arguments = args_from_options<MlaSm100Type>(...);
CUTLASS_CHECK(fmha.can_implement(arguments));
CUTLASS_CHECK(fmha.initialize(arguments, workspace.data_ptr(), stream));
CUTLASS_CHECK(fmha.run(arguments, workspace.data_ptr(), stream));
...
if (in_dtype == torch::headeronly::ScalarType::Half) {
  runMla<cutlass::half_t, cutlass::half_t, IsPaged128, IsPersistent<NotManualSplitKV>>(...);
} else if (in_dtype == torch::headeronly::ScalarType::BFloat16) {
  runMla<cutlass::bfloat16_t, cutlass::bfloat16_t, IsPaged128, IsPersistent<NotManualSplitKV>>(...);
} else if (in_dtype == torch::headeronly::ScalarType::Float8_e4m3fn) {
  runMla<cutlass::float_e4m3_t, cutlass::bfloat16_t, IsPaged128, IsPersistent<NotManualSplitKV>>(...);
}
```
**EN:** The runtime path validates implementability, initializes workspace-backed parameters, and launches the kernel. Dispatch depends on page size (128 vs other), whether split-KV is manual, and the input dtype.
**CN:** 运行时路径会先检查该问题是否可实现，再初始化依赖 workspace 的参数并启动内核。分派逻辑同时取决于 page size（128 与否）、split-KV 是否手动指定以及输入数据类型。

### Workspace sizing and op registration
```cpp
arguments.problem_shape =
    cute::make_tuple(TileShapeH{}, static_cast<int>(max_seq_len), TileShapeD{}, static_cast<int>(num_batches));
...
MlaSm100Type::Fmha::set_split_kv(arguments);
return MlaSm100Type::Fmha::get_workspace_size(arguments);
...
STABLE_TORCH_LIBRARY_IMPL(_C, CUDA, m) {
  m.impl("sm100_cutlass_mla_decode", TORCH_BOX(&sm100_cutlass_mla_decode));
}
```
**EN:** The helper for workspace size reuses the same split-KV heuristic as the real kernel launch, ensuring the caller allocates enough temporary memory. The final section exports the decode op and workspace-size query into PyTorch's stable operator registry.
**CN:** workspace 大小查询函数复用了真实内核启动时相同的 split-KV 启发式，因此调用方能分配到足够的临时内存。文件末尾则把 decode 算子和 workspace 大小查询注册到 PyTorch 的稳定算子表中。

## Key Concepts / 关键概念
- **Type-driven kernel assembly / 类型驱动内核组装**: Template aliases encode scheduler and dtype choices.
- **Stable ABI tensor lowering / 稳定 ABI 张量下沉**: Converts `torch::stable::Tensor` into CUTLASS-compatible arguments.
- **Split-KV reuse / 复用 split-KV 逻辑**: Workspace sizing and execution share the same heuristic.

## Dependencies / 依赖关系
- `device/sm100_mla.hpp` and `kernel/sm100_mla_tile_scheduler.hpp` provide the actual kernel wrapper and scheduler.
- `libtorch_stable/torch_utils.h` supplies stream/device helpers and validation macros.
- Registered into `STABLE_TORCH_LIBRARY_IMPL` for Python-facing invocation.
