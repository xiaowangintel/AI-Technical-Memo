# nvfp4_scaled_mm_entry.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/libtorch_stable/quantization/fp4/nvfp4_scaled_mm_entry.cu`
- **Repository**: vllm-project/vllm
- **Purpose**: Dispatches scaled FP4 matrix multiplication calls to the SM100 or SM120 CUTLASS backend compiled into the binary. / 将缩放 FP4 矩阵乘调用分发到二进制中已编译的 SM100 或 SM120 CUTLASS 后端。

## Line-by-Line Analysis / 逐行分析
### Feature-gated backend declarations
```cpp
#if defined ENABLE_NVFP4_SM100 && ENABLE_NVFP4_SM100
void cutlass_scaled_fp4_mm_sm100a(torch::stable::Tensor& D,
                                  torch::stable::Tensor const& A,
                                  torch::stable::Tensor const& B,
                                  torch::stable::Tensor const& A_sf,
                                  torch::stable::Tensor const& B_sf,
                                  torch::stable::Tensor const& alpha);
#endif

#if defined ENABLE_NVFP4_SM120 && ENABLE_NVFP4_SM120
void cutlass_scaled_fp4_mm_sm120a(torch::stable::Tensor& D,
```
**EN:** The file first declares the SM-specific implementations behind compile-time flags, keeping unsupported backends completely out of the exported interface.
**CN:** 文件首先在编译期开关后声明各个 SM 专用实现，从而把不受支持的后端完全排除在导出接口之外。

### Runtime SM-based backend selection
```cpp
  const int32_t sm = get_sm_version_num();

#if defined(ENABLE_NVFP4_SM100) && ENABLE_NVFP4_SM100
  if (sm >= 100 && sm < 120) {
    cutlass_scaled_fp4_mm_sm100a(D, A, B, A_sf, B_sf, alpha);
    return;
  }
#endif

#if defined(ENABLE_NVFP4_SM120) && ENABLE_NVFP4_SM120
  if (sm >= 120 && sm < 130) {
    cutlass_scaled_fp4_mm_sm120a(D, A, B, A_sf, B_sf, alpha);
    return;
  }
#endif
```
**EN:** At call time, the entry function checks the active GPU architecture and routes the request to the matching SM100 or SM120 implementation.
**CN:** 在调用时，入口函数会检查当前 GPU 架构，并把请求路由到匹配的 SM100 或 SM120 实现。

### Support probe for higher-level backend selection
```cpp
  int runtimeVersion;
  cudaRuntimeGetVersion(&runtimeVersion);
  if (runtimeVersion < 12080) return false;
  // Only report support when the SM-specific kernel was actually compiled in,
  // so the Python-side backend selector does not choose CUTLASS and then hit
  // TORCH_CHECK_NOT_IMPLEMENTED (or worse, fall through to Marlin).
#if defined(ENABLE_NVFP4_SM100) && ENABLE_NVFP4_SM100
  if (cuda_device_capability >= 100 && cuda_device_capability < 120)
    return true;
#endif
#if defined(ENABLE_NVFP4_SM120) && ENABLE_NVFP4_SM120
  if (cuda_device_capability >= 120 && cuda_device_capability < 130)
```
**EN:** A separate helper reports whether FP4 scaled GEMM is truly available by checking both the CUDA runtime version and whether a compatible backend was compiled.
**CN:** 另一个辅助函数会同时检查 CUDA 运行时版本与兼容后端是否被编译，从而报告 FP4 缩放 GEMM 是否真正可用。

## Key Concepts / 关键概念
- Runtime architecture dispatch / 运行时架构分发
- Compile-time backend availability / 编译期后端可用性控制
- Capability probing for higher-level scheduling / 供上层调度使用的能力探测

## Dependencies / 依赖关系
- `nvfp4_scaled_mm_kernels.cu` and `nvfp4_scaled_mm_sm120_kernels.cu` as implementation backends / 具体实现后端来自 `nvfp4_scaled_mm_kernels.cu` 与 `nvfp4_scaled_mm_sm120_kernels.cu`
- `cutlass_extensions/common.hpp` for SM detection utilities / 通过 `cutlass_extensions/common.hpp` 获取 SM 探测工具
- CUDA runtime APIs for version checks / 使用 CUDA runtime API 做版本检查
