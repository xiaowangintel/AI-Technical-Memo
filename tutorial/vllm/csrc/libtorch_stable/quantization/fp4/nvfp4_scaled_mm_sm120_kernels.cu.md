# nvfp4_scaled_mm_sm120_kernels.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/libtorch_stable/quantization/fp4/nvfp4_scaled_mm_sm120_kernels.cu`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements the SM120-specific scaled NVFP4 GEMM backend with CUTLASS dispatch for BF16 and FP16 outputs. / 实现面向 SM120 的缩放 NVFP4 GEMM 后端，并通过 CUTLASS 分发 BF16 与 FP16 输出。

## Line-by-Line Analysis / 逐行分析
### Input validation macro bundle
```cpp
#define CHECK_INPUT(x, st, m) \
  CHECK_TH_CUDA(x, m);        \
  CHECK_CONTIGUOUS(x, m);     \
  CHECK_TYPE(x, st, m)
```
**EN:** The file starts by bundling CUDA, contiguity, and dtype checks into one macro so every public entry can enforce the same tensor contract concisely.
**CN:** 文件一开始把 CUDA、连续性和数据类型检查打包成一个宏，便于所有公开入口以简洁方式统一执行张量契约检查。

### SM120 GEMM type specialization
```cpp

  using ElementAccumulator = float;
  using ArchTag = cutlass::arch::Sm120;
  using OperatorClass = cutlass::arch::OpClassBlockScaledTensorOp;

  using MmaTileShape = typename Config::MmaTileShape;
  using ClusterShape = typename Config::ClusterShape;
  using PerSmTileShape_MNK = typename Config::PerSmTileShape_MNK;

  using CollectiveEpilogue =
      typename cutlass::epilogue::collective::CollectiveBuilder<
```
**EN:** This template block specializes the GEMM path for SM120, fixing the architecture tag and the block-scaled tensor-core operator class used by CUTLASS.
**CN:** 该模板块为 SM120 专门化了 GEMM 路径，固定了 CUTLASS 所使用的架构标签和分块缩放 Tensor Core 算子类别。

### Scale-layout construction for SM120 arguments
```cpp
  auto layout_SFA = Sm1xxBlkScaledConfig::tile_atom_to_shape_SFA(
      cute::make_shape(M, N, K, 1));
  auto layout_SFB = Sm1xxBlkScaledConfig::tile_atom_to_shape_SFB(
      cute::make_shape(M, N, K, 1));

  typename Gemm::Arguments arguments{
      cutlass::gemm::GemmUniversalMode::kGemm,
```
**EN:** The argument builder reconstructs the scale-factor layouts expected by the SM1xx block-scaled kernels so CUTLASS sees the right logical tensor shape.
**CN:** 参数构建器会重建 SM1xx 分块缩放内核所期望的尺度布局，从而让 CUTLASS 看到正确的逻辑张量形状。

### Dispatch helper based on the M dimension
```cpp
  uint32_t const mp2 = std::max(static_cast<uint32_t>(16), next_pow_2(m));
  if (mp2 <= 256) {
    runGemm<Fp4GemmSm120<sm120_fp4_config_M256, cutlass::bfloat16_t>::Gemm>(
        D, A, B, A_sf, B_sf, alpha, m, n, k, stream);
  } else {
    runGemm<Fp4GemmSm120<sm120_fp4_config_default, cutlass::bfloat16_t>::Gemm>(
        D, A, B, A_sf, B_sf, alpha, m, n, k, stream);
  }
}

void cutlass_fp4_f16_gemm_dispatch(torch::stable::Tensor& D,
```
**EN:** Like the SM100 backend, the SM120 dispatcher chooses between configuration presets by rounding M upward and selecting the best matching tile regime.
**CN:** 与 SM100 后端类似，SM120 分发器也会通过向上取整 M 并选择最匹配的 tile 配置来决定具体实现。

### Public entry with device guard and stream lookup
```cpp
  auto out_dtype = D.scalar_type();
  const torch::stable::accelerator::DeviceGuard device_guard(
      A.get_device_index());
  const cudaStream_t stream = get_current_cuda_stream(A.get_device_index());

  if (out_dtype == torch::headeronly::ScalarType::BFloat16) {
    return cutlass_fp4_bf16_gemm_dispatch(D, A, B, A_sf, B_sf, alpha, m, n, k,
                                          stream);
  } else if (out_dtype == torch::headeronly::ScalarType::Half) {
```
**EN:** The public entry acquires the correct device context, retrieves the active CUDA stream, and then dispatches to the BF16 or FP16 GEMM specialization.
**CN:** 公开入口会先切换到正确的设备上下文、获取当前 CUDA stream，然后再分发到 BF16 或 FP16 的 GEMM 专门化实现。

## Key Concepts / 关键概念
- SM120-specific FP4 GEMM specialization / 面向 SM120 的 FP4 GEMM 专门化
- Unified validation plus dtype dispatch / 统一校验与数据类型分发
- Scale-layout reconstruction for CUTLASS / 为 CUTLASS 重建尺度布局

## Dependencies / 依赖关系
- CUTLASS SM120 block-scaled tensor-core components / CUTLASS 的 SM120 分块缩放 Tensor Core 组件
- `core/math.hpp` and stream helpers for dispatch logic / 通过 `core/math.hpp` 与 stream 辅助函数实现分发逻辑
- `torch::stable` tensor/device APIs for device-guarded execution / 使用 `torch::stable` 张量/设备 API 做受设备保护的执行
