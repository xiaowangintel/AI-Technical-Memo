# mxfp4_blockwise_moe_kernel.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/libtorch_stable/quantization/fp4/mxfp4_blockwise_moe_kernel.cu`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements CUTLASS grouped GEMM for MXFP4 blockwise MoE execution on SM100-class GPUs. / 为 SM100 类 GPU 实现基于 CUTLASS 的 MXFP4 分块式 MoE 分组 GEMM。

## Line-by-Line Analysis / 逐行分析
### Per-expert grouped GEMM pointer setup
```cpp
  int64_t expert_offset = static_cast<int64_t>(expert_offsets[expert_id]);
  int64_t sf_offset = static_cast<int64_t>(sf_offsets[expert_id]);
  int64_t group_size = 32;
  int64_t m = static_cast<int64_t>(problem_sizes_as_shapes[expert_id * 3]);
  int64_t n = static_cast<int64_t>(problem_sizes_as_shapes[expert_id * 3 + 1]);
  int64_t k = static_cast<int64_t>(problem_sizes_as_shapes[expert_id * 3 + 2]);
  assert((m >= 0 && n == N && k == K && k % 2 == 0) &&
         "unexpected problem sizes");

  int64_t half_k = static_cast<int64_t>(k / 2);
  int64_t group_k = static_cast<int64_t>(k / group_size);
  // Shape of A as uint8/byte = [M, K // 2]
```
**EN:** The setup kernel derives each expert's M/N/K problem size and computes grouped pointers and strides using MXFP4's 32-element scaling granularity.
**CN:** 该准备内核根据每个 expert 的 M/N/K 问题规模计算分组指针与步长，使用 MXFP4 的 32 元素尺度粒度。

### Macro-based launcher for start buffers
```cpp
#define __CALL_MXFP4_GET_STARTS_KERNEL(ELEMENT_AB_TYPE, SF_TYPE,               \
                                       TENSOR_C_TYPE, C_TYPE, LayoutSFA,       \
                                       LayoutSFB, ScaleConfig)                 \
  else if (out_tensors.scalar_type() == TENSOR_C_TYPE) {                       \
    __mxfp4_get_group_gemm_starts<ELEMENT_AB_TYPE, C_TYPE, SF_TYPE, LayoutSFA, \
                                  LayoutSFB, ScaleConfig>                      \
        <<<1, num_experts, 0, stream>>>(                                       \
            static_cast<ELEMENT_AB_TYPE**>(a_starts.data_ptr()),               \
            static_cast<ELEMENT_AB_TYPE**>(b_starts.data_ptr()),               \
            static_cast<C_TYPE**>(out_starts.data_ptr()),                      \
```
**EN:** A helper macro expands the correct template instantiation for the pointer-setup kernel so host code can dispatch output types without repeating a long launch expression.
**CN:** 该辅助宏展开指针准备内核的正确模板实例，使宿主代码在分发输出类型时不必重复冗长的启动表达式。

### CUTLASS SM100 grouped GEMM specialization
```cpp

  // Architecture definitions
  using ArchTag = cutlass::arch::Sm100;
  using EpilogueOperatorClass = cutlass::arch::OpClassTensorOp;
  using MainloopOperatorClass = cutlass::arch::OpClassBlockScaledTensorOp;
  using StageCountType = cutlass::gemm::collective::StageCountAuto;

  using ClusterShape = Shape<_1, _1, _1>;
  struct MMA1SMConfig {
    using MmaTileShape = Shape<_128, _128, _128>;
    using KernelSchedule =
        cutlass::gemm::KernelPtrArrayTmaWarpSpecialized1SmMxf4Sm100;
    using EpilogueSchedule = cutlass::epilogue::PtrArrayTmaWarpSpecialized1Sm;
```
**EN:** This block fixes the architecture, operator class, and epilogue policy for the MXFP4 grouped GEMM path, wiring the kernel into CUTLASS block-scaled tensor-core machinery.
**CN:** 该代码块固定了 MXFP4 分组 GEMM 路径的架构、算子类别与 epilogue 策略，把内核接入 CUTLASS 的分块缩放 Tensor Core 实现。

### Architecture gate for runtime dispatch
```cpp
  int32_t version_num = get_sm_version_num();
#if defined ENABLE_NVFP4_SM100 && ENABLE_NVFP4_SM100
  if (version_num >= 100 && version_num < 120) {
    run_mxfp4_blockwise_scaled_group_mm_sm100<OutType>(
        output, a, b, a_blockscale, b_blockscales, problem_sizes,
        expert_offsets, sf_offsets, M, N, K);
    return;
  }
```
**EN:** The runtime wrapper only enables this path when the process is running on a compiled SM100-range device; otherwise it raises a clear not-supported error.
**CN:** 运行时包装器仅在进程运行于已编译支持的 SM100 范围设备时启用该路径，否则抛出明确的不支持错误。

### Public API validation for MXFP4 tensors
```cpp
  CHECK_INPUT(a, MXFP4_FLOAT4_E2M1X2, "a");
  CHECK_INPUT(b, MXFP4_FLOAT4_E2M1X2, "b");
  // MXFP4 uses E8M0 scale factors (stored as uint8)
  CHECK_INPUT(a_blockscale, MXFP4_SF_DTYPE, "a_blockscale");
  CHECK_INPUT(b_blockscales, MXFP4_SF_DTYPE, "b_blockscales");

  STD_TORCH_CHECK(
      a_blockscale.dim() == 2,
      "expected a_blockscale to be of shape [num_experts, rounded_m,"
      " k // group_size], observed rank: ",
      a_blockscale.dim())
  STD_TORCH_CHECK(b_blockscales.dim() == 3,
```
**EN:** The exported operator verifies MXFP4 payload tensors, scale tensors, and problem-size metadata before invoking the grouped GEMM pipeline.
**CN:** 导出的算子会先校验 MXFP4 载荷张量、尺度张量与问题规模元数据，然后再调用分组 GEMM 流程。

## Key Concepts / 关键概念
- Grouped GEMM for multiple experts / 面向多个 expert 的分组 GEMM
- MXFP4 block scales stored with E8M0-like metadata / 使用类似 E8M0 元数据的 MXFP4 分块尺度
- CUTLASS SM100 block-scaled tensor-core integration / 与 CUTLASS SM100 分块缩放 Tensor Core 集成

## Dependencies / 依赖关系
- CUTLASS/CUTE grouped GEMM builders and layouts / CUTLASS 与 CUTE 的分组 GEMM 构建器和布局
- `cutlass_extensions/common.hpp` for SM/version helpers / 通过 `cutlass_extensions/common.hpp` 获取 SM/版本辅助函数
- Stable torch tensor wrappers for tensor validation and registration / 使用 stable torch 张量封装进行校验与算子注册
