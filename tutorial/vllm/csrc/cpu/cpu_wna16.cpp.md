# cpu_wna16.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/cpu/cpu_wna16.cpp`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements CPU weight-only low-bit (WNA16) kernels and dispatch logic. / 实现 CPU 仅权重量化低比特（WNA16）内核及其分派逻辑。

## Line-by-Line Analysis / 逐行分析
### Preamble and includes (lines 1-9)
```cpp
#include "cpu/cpu_types.hpp"
#include "cpu/utils.hpp"

#ifdef CPU_CAPABILITY_AMXBF16
  #include "cpu/micro_gemm/cpu_micro_gemm_amx.hpp"
#endif
#include "cpu/micro_gemm/cpu_micro_gemm_vec.hpp"

#define VLLM_DISPATCH_CASE_16B_TYPES(...)                 \
```
**EN:** This opening block pulls in the headers, feature macros, and compile-time aliases that the rest of the file depends on.
**CN:** 这一开头代码块引入了后续实现依赖的头文件、特性宏以及编译期别名。

### Compile-time setup: VLLM_DISPATCH_CASE_16B_TYPES (lines 13-14)
```cpp
#define VLLM_DISPATCH_16B_TYPES(TYPE, NAME, ...) \
  AT_DISPATCH_SWITCH(TYPE, NAME, VLLM_DISPATCH_CASE_16B_TYPES(__VA_ARGS__))
```
**EN:** These macros encode compile-time dispatch rules, selecting the right specialization based on ISA, dtype, or backend capabilities.
**CN:** 这些宏封装了编译期分派规则，用于根据 ISA、数据类型或后端能力选择合适的专用实现。

### Function / Kernel: print_logits (lines 16-31)
```cpp
template <typename T>
void print_logits(const char* name, T* ptr, int32_t row, int32_t col,
                  int32_t stride) {
  std::stringstream ss;
  ss << std::fixed << std::setprecision(5) << name << ": [\n";
  auto* curr_logits_buffer = ptr;
  for (int32_t m = 0; m < row; ++m) {
    for (int32_t n = 0; n < col; ++n) {
      ss << curr_logits_buffer[n] << ", ";
    }
    ss << "\n";
    curr_logits_buffer += stride;
  }
  ss << "]\n";
  std::printf("%s", ss.str().c_str());
}
```
**EN:** This function provides a reusable piece of the file’s main compute pipeline, usually handling validation, indexing, or a fused math step.
**CN:** 该函数是本文件主计算流水线中的一个可复用环节，通常负责校验、索引计算或融合数学步骤。

### Class: Dequantizer4b (lines 38-138)
```cpp
class Dequantizer4b {
 public:
  constexpr static int32_t pack_num = 32 / 4;
  using scalar_vec_t = typename VecTypeTrait<scalar_t>::vec_t;

 public:
  static void dequant(int32_t* __restrict__ q_weight,
                      scalar_t* __restrict__ weight,
                      scalar_t* __restrict__ scales,
                      int32_t* __restrict__ zeros, int32_t* __restrict__ g_idx,
                      const int64_t scales_stride, const int64_t zeros_stride,
                      const int32_t k_size, const int32_t group_size) {
    vec_op::FP32Vec16 lut;
    if constexpr (has_zp) {
// ...
    }
  }
};
```
**EN:** This type centralizes parameters and compile-time traits, making the later kernel code simpler and safer to specialize.
**CN:** 该类型集中保存参数和编译期 trait，使后续内核更容易进行专用化并保持实现安全。

### Function / Kernel: cpu_gemm_wna16_impl (lines 141-286)
```cpp
template <typename scalar_t, typename dequantizer_t, typename gemm_t>
void cpu_gemm_wna16_impl(
    scalar_t* __restrict__ input, int32_t* __restrict__ q_weight,
    scalar_t* __restrict__ output, scalar_t* __restrict__ scales,
    int32_t* __restrict__ zeros, int32_t* __restrict__ g_idx,
    scalar_t* __restrict__ bias, const int32_t m_size, const int32_t n_size,
    const int32_t k_size, const int64_t input_stride,
    const int64_t output_stride, const int64_t scales_group_stride,
    const int64_t zeros_group_stride, const int32_t group_num,
    const int32_t group_size, const int64_t pack_factor) {
  constexpr int32_t gemm_n_tile_size = gemm_t::NSize;
  constexpr int32_t gemm_m_tile_size = gemm_t::MaxMSize;
  constexpr int32_t n_block_size = 16;
  static_assert(gemm_n_tile_size % n_block_size == 0);
// ...
    }
  }
}
```
**EN:** This host-side function validates inputs and dispatches to the best specialization or launch configuration for the current runtime shape.
**CN:** 这个宿主端函数负责校验输入，并根据当前运行时形状选择最佳专用实现或启动配置。

### Function / Kernel: cpu_gemm_wna16 (lines 288-402)
```cpp
void cpu_gemm_wna16(
    const torch::Tensor& input,  // [M, K]
    const torch::Tensor&
        q_weight,           // [N / 16, K * 16 / pack_factor], packed as int32
    torch::Tensor& output,  // [M, N]
    const torch::Tensor& scales,  // [group_num, N]
    const std::optional<torch::Tensor>&
        zeros,  // [group_num, N / pack_factor], packed as int32
    const std::optional<torch::Tensor>& g_idx,  // [K]
    const std::optional<torch::Tensor>& bias,   // [N]
    const int64_t pack_factor, const std::string& isa_hint) {
  using cpu_utils::ISA;
  TORCH_CHECK_EQ(pack_factor, 8);  // only supports 4bits
  const int32_t a_m_size = input.size(0);
// ...
    }
  });
}
```
**EN:** This function provides a reusable piece of the file’s main compute pipeline, usually handling validation, indexing, or a fused math step.
**CN:** 该函数是本文件主计算流水线中的一个可复用环节，通常负责校验、索引计算或融合数学步骤。

## Key Concepts / 关键概念
- Tiled matrix multiplication / 分块矩阵乘法
- Integer kernel specialization / 整数内核专用化
- Weight-only quantization / 仅权重量化
- Intel AMX tiling / Intel AMX 分块
- SIMD vectorization / SIMD 向量化
- Template-based specialization / 基于模板的专用化

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `cpu/cpu_types.hpp`, `cpu/utils.hpp`, `cpu/micro_gemm/cpu_micro_gemm_amx.hpp`, `cpu/micro_gemm/cpu_micro_gemm_vec.hpp`
- **Runtime coupling / 运行时耦合**: Uses templates to specialize code paths at compile time / 通过模板在编译期专用化代码路径
