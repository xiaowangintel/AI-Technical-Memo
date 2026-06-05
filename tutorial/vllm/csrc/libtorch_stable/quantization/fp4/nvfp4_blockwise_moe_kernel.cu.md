# nvfp4_blockwise_moe_kernel.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/libtorch_stable/quantization/fp4/nvfp4_blockwise_moe_kernel.cu`
- **Repository**: vllm-project/vllm
- **Purpose**: Provides NVFP4 grouped blockwise MoE GEMM kernels with SM100 and SM120 CUTLASS backends. / 提供基于 CUTLASS 的 NVFP4 分块式 MoE 分组 GEMM 内核，并支持 SM100 与 SM120 后端。

## Line-by-Line Analysis / 逐行分析
### Per-expert pointer and scale preparation
```cpp
  int64_t sf_offset = static_cast<int64_t>(sf_offsets[expert_id]);
  // size for block in block scale.
  int64_t group_size = 16;
  int64_t m = static_cast<int64_t>(problem_sizes_as_shapes[expert_id * 3]);
  int64_t n = static_cast<int64_t>(problem_sizes_as_shapes[expert_id * 3 + 1]);
  int64_t k = static_cast<int64_t>(problem_sizes_as_shapes[expert_id * 3 + 2]);
  assert((m >= 0 && n == N && k == K && k % 2 == 0) &&
         "unexpected problem sizes");

  int64_t half_k = static_cast<int64_t>(k / 2);
  int64_t group_k = static_cast<int64_t>(k / group_size);
  // Shape of A as uint8/byte = [M, K // 2]
```
**EN:** The setup kernel computes expert-local problem sizes and pointer starts using NVFP4's 16-element group size, preparing the metadata arrays consumed by CUTLASS grouped GEMM.
**CN:** 准备内核使用 NVFP4 的 16 元素分组大小来计算 expert 局部问题规模和起始指针，为 CUTLASS 分组 GEMM 准备元数据数组。

### Template launch macro for block-scale metadata
```cpp
#define __CALL_GET_STARTS_KERNEL_BLOCKSCALE(ELEMENT_AB_TYPE, SF_TYPE,         \
                                            TENSOR_C_TYPE, C_TYPE, LayoutSFA, \
                                            LayoutSFB, ScaleConfig)           \
  else if (out_tensors.scalar_type() == TENSOR_C_TYPE) {                      \
    __get_group_gemm_starts<ELEMENT_AB_TYPE, C_TYPE, SF_TYPE, float,          \
                            LayoutSFA, LayoutSFB, ScaleConfig>                \
        <<<1, num_experts, 0, stream>>>(                                      \
            static_cast<ELEMENT_AB_TYPE**>(a_starts.data_ptr()),              \
            static_cast<ELEMENT_AB_TYPE**>(b_starts.data_ptr()),              \
            static_cast<C_TYPE**>(out_starts.data_ptr()),                     \
```
**EN:** This macro expands the correct pointer-setup kernel instantiation, including the scale-layout configuration required by block-scaled FP4 execution.
**CN:** 该宏会展开正确的指针准备内核实例，并包含分块缩放 FP4 执行所需的尺度布局配置。

### SM100 CUTLASS grouped GEMM path
```cpp

  // Architecture definitions
  using ArchTag = cutlass::arch::Sm100;
  using EpilogueOperatorClass =
      cutlass::arch::OpClassTensorOp;  // Epilogue Operator class tag
  using MainloopOperatorClass =
      cutlass::arch::OpClassBlockScaledTensorOp;  // Mainloop Operator class tag
  using StageCountType =
      cutlass::gemm::collective::StageCountAuto;  // Stage count maximized based
                                                  // on the tile size

  using ClusterShape = Shape<_1, _1, _1>;
  struct MMA1SMConfig {
```
**EN:** The SM100 path binds NVFP4 element types, tensor-core operator classes, and linear-combination epilogues into a grouped GEMM implementation for pre-SM120 devices.
**CN:** SM100 路径把 NVFP4 元素类型、Tensor Core 算子类别与线性组合 epilogue 绑定到分组 GEMM 实现中，面向 SM120 之前的设备。

### SM120 specialization with fixed BF16 output
```cpp
  // NOTE: For SM120 it seems templating the output type is not supported and
  // we need to hardcode the output type to bfloat16
  using ElementC = cutlass::bfloat16_t;
  using ElementD = ElementC;
  using ElementAccumulator = float;
  // Layout definitions
  using LayoutA = cutlass::layout::RowMajor;
```
**EN:** The SM120 branch documents an architectural constraint: output type templating is avoided, so the kernel is hard-wired to bfloat16 accumulation/output behavior.
**CN:** SM120 分支记录了一个架构限制：这里不再对输出类型做模板化，因此内核被固定为 bfloat16 的累加/输出行为。

### Runtime architecture dispatch
```cpp
#if defined ENABLE_NVFP4_SM120 && ENABLE_NVFP4_SM120
  if (version_num >= 120 && version_num < 130) {
    run_fp4_blockwise_scaled_group_mm_sm120(
        output, a, b, a_blockscale, b_blockscales, alphas, problem_sizes,
        expert_offsets, sf_offsets, M, N, K);
    return;
  }
#endif
#if defined ENABLE_NVFP4_SM100 && ENABLE_NVFP4_SM100
  if (version_num >= 100 && version_num < 120) {
    run_fp4_blockwise_scaled_group_mm_sm100<OutType>(
```
**EN:** The wrapper selects the SM120 or SM100 implementation based on runtime hardware and fails loudly if the binary was not compiled for the active architecture.
**CN:** 该包装器会根据运行时硬件选择 SM120 或 SM100 实现；若当前架构未被编译进二进制，则显式报错。

### Public API checks for NVFP4 MoE tensors
```cpp
  CHECK_INPUT(a, FLOAT4_E2M1X2, "a");
  CHECK_INPUT(b, FLOAT4_E2M1X2, "b");
  CHECK_INPUT(a_blockscale, SF_DTYPE, "a_blockscale");
  CHECK_INPUT(b_blockscales, SF_DTYPE, "b_blockscales");
  CHECK_INPUT(alphas, torch::headeronly::ScalarType::Float, "alphas");

  STD_TORCH_CHECK(
      a_blockscale.dim() == 2,
      "expected a_blockscale to be of shape [num_experts, rounded_m,"
      " k // group_size], observed rank: ",
      a_blockscale.dim())
  STD_TORCH_CHECK(b_blockscales.dim() == 3,
                  "expected b_blockscale to be of shape: "
```
**EN:** The exported operator validates FP4 payloads, block-scale tensors, and optional alpha factors before constructing the grouped GEMM launch state.
**CN:** 公开算子会先校验 FP4 载荷、分块尺度张量以及可选 alpha 因子，然后再构建分组 GEMM 的启动状态。

## Key Concepts / 关键概念
- Grouped MoE GEMM over multiple experts / 跨多个 expert 的分组 MoE GEMM
- Architecture-specific CUTLASS backends for SM100 and SM120 / 面向 SM100 与 SM120 的架构专用 CUTLASS 后端
- Separate payload, block-scale, and alpha tensors / 分离的载荷、块尺度与 alpha 张量

## Dependencies / 依赖关系
- CUTLASS grouped GEMM components and packed-stride helpers / CUTLASS 分组 GEMM 组件与 packed-stride 辅助工具
- `cutlass_extensions/common.hpp` for SM detection and common utilities / 通过 `cutlass_extensions/common.hpp` 进行 SM 探测与公共工具复用
- Stable torch tensor APIs for validation and op registration / 使用 stable torch 张量 API 做校验与算子注册
