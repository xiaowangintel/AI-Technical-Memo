# nvfp4_quant_entry.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/libtorch_stable/quantization/fp4/nvfp4_quant_entry.cu`
- **Repository**: vllm-project/vllm
- **Purpose**: Acts as the architecture-gated front door for NVFP4 quantization and expert-quantization entry points. / 作为 NVFP4 普通量化与 expert 量化入口的架构门面与统一前端。

## Line-by-Line Analysis / 逐行分析
### Feature-gated declarations of SM1xx kernels
```cpp
void scaled_fp4_quant_sm1xxa(torch::stable::Tensor const& output,
                             torch::stable::Tensor const& input,
                             torch::stable::Tensor const& output_sf,
                             torch::stable::Tensor const& input_sf,
                             bool is_sf_swizzled_layout);
#endif

#if (defined(ENABLE_NVFP4_SM100) && ENABLE_NVFP4_SM100) || \
    (defined(ENABLE_NVFP4_SM120) && ENABLE_NVFP4_SM120)
void scaled_fp4_experts_quant_sm1xxa(
    torch::stable::Tensor& output, torch::stable::Tensor& output_scale,
```
**EN:** Kernel declarations are wrapped in compile-time feature guards so the file only exposes symbols for architectures that were actually built.
**CN:** 这些内核声明被包裹在编译期特性开关中，确保文件只暴露那些真实参与编译的架构符号。

### Runtime support probe for NVFP4 quantization
```cpp
static bool nvfp4_quant_sm_supported() {
  const int32_t sm = get_sm_version_num();
  #if defined(ENABLE_NVFP4_SM100) && ENABLE_NVFP4_SM100
  if (sm >= 100 && sm < 120) return true;
  #endif
  #if defined(ENABLE_NVFP4_SM120) && ENABLE_NVFP4_SM120
  if (sm >= 120 && sm < 130) return true;
  #endif
  return false;
}
```
**EN:** A small helper checks the current SM version against the compiled feature set, preventing the higher-level API from selecting an unavailable backend.
**CN:** 这个小辅助函数会把当前 SM 版本与已编译的特性集合做比对，避免上层 API 选择到不可用的后端。

### Tensor allocation for the generic quant API
```cpp
  auto output = torch::stable::empty(
      {m, n / 2}, torch::headeronly::ScalarType::Byte, std::nullopt, device);

  torch::stable::Tensor output_sf;
  if (is_sf_swizzled_layout) {
    auto [sf_m, sf_n] = vllm::computeSwizzledSFShape(m, n);
    output_sf = torch::stable::empty(
```
**EN:** The convenience wrapper allocates the packed output tensor and its scale tensor using the source device, then forwards to the architecture-specific implementation.
**CN:** 这个便捷包装器会在源设备上分配打包输出张量及其尺度张量，然后转发给对应架构的实现。

### Thin expert-quant forwarding wrappers
```cpp
                  ". Recompile with the appropriate CUDA arch.");
  return scaled_fp4_experts_quant_sm1xxa(
      output, output_scale, input, input_global_scale, input_offset_by_experts,
      output_scale_offset_by_experts);
#endif
  STD_TORCH_CHECK_NOT_IMPLEMENTED(
      false, "No compiled nvfp4 experts quantization kernel");
}

void silu_and_mul_nvfp4_quant(torch::stable::Tensor& output,
                              torch::stable::Tensor& output_sf,
                              torch::stable::Tensor& input,
                              torch::stable::Tensor& input_sf) {
#if (defined(ENABLE_NVFP4_SM100) && ENABLE_NVFP4_SM100) || \
```
**EN:** The remaining wrappers simply forward arguments to expert quantization or fused expert quantization after verifying that a suitable backend exists.
**CN:** 其余包装器只是在确认存在合适后端之后，将参数直接转发给 expert 量化或融合版 expert 量化实现。

## Key Concepts / 关键概念
- Compile-time feature guards combined with runtime SM checks / 编译期特性开关与运行时 SM 检查结合
- Unified entry layer over multiple quant backends / 对多个量化后端进行统一封装的入口层
- Convenience allocation for packed output tensors / 为打包输出张量提供便捷分配逻辑

## Dependencies / 依赖关系
- `nvfp4_quant_kernels.cu` and `nvfp4_experts_quant.cu` as concrete backends / 具体后端来自 `nvfp4_quant_kernels.cu` 与 `nvfp4_experts_quant.cu`
- `nvfp4_utils.cuh` for scale-shape calculation / 使用 `nvfp4_utils.cuh` 计算尺度张量形状
- `cutlass_extensions/common.hpp` for SM version helpers / 通过 `cutlass_extensions/common.hpp` 获取 SM 版本辅助函数
