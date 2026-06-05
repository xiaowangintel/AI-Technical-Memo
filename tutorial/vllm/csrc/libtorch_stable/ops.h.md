# ops.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/libtorch_stable/ops.h`
- **Repository**: vllm-project/vllm
- **Purpose**: Central stable-ABI declaration header for vLLM custom operators, covering quantization, CUTLASS GEMM, MoE helpers, AWQ, and other CUDA extensions. / vLLM 自定义算子的稳定 ABI 总声明头文件，涵盖量化、CUTLASS GEMM、MoE 辅助函数、AWQ 以及其他 CUDA 扩展。

## Line-by-Line Analysis / 逐行分析
### Quantization and scaled GEMM API surface
```cpp
torch::stable::Tensor permute_cols(torch::stable::Tensor const& A,
                                   torch::stable::Tensor const& perm);
...
void per_token_group_quant_fp8(const torch::stable::Tensor& input,
                               torch::stable::Tensor& output_q,
                               torch::stable::Tensor& output_s,
                               int64_t group_size, double eps, double fp8_min,
                               double fp8_max, bool scale_ue8m0,
                               bool dummy_is_scale_transposed,
                               bool dummy_is_tma_aligned);
...
void cutlass_scaled_mm(torch::stable::Tensor& out,
                       torch::stable::Tensor const& a,
                       torch::stable::Tensor const& b,
```
**EN:** The header begins with the core low-level ops that many higher-level execution paths depend on: column permutation, per-token quantization, and CUTLASS scaled matrix multiplication.
**CN:** 头文件开头声明了许多高层执行路径都会依赖的基础算子：列置换、逐 token 量化，以及 CUTLASS 的 scaled matrix multiplication。

### Mixture-of-Experts helper declarations
```cpp
void cutlass_moe_mm(torch::stable::Tensor& out_tensors,
                    torch::stable::Tensor const& a_tensors,
                    torch::stable::Tensor const& b_tensors,
                    ...);
...
void get_cutlass_moe_mm_data(
    const torch::stable::Tensor& topk_ids,
    torch::stable::Tensor& expert_offsets,
    torch::stable::Tensor& problem_sizes1,
    torch::stable::Tensor& problem_sizes2,
```
**EN:** These declarations separate the actual MoE GEMM kernels from the preprocessing utilities that compute expert offsets, permutation maps, and grouped problem sizes.
**CN:** 这些声明把真正的 MoE GEMM 内核与前处理辅助函数分离开来；后者负责计算 expert offset、置换映射以及分组后的问题尺寸。

### FP4/NVFP4 section
```cpp
bool cutlass_scaled_mm_supports_fp4(int64_t cuda_device_capability);
...
void cutlass_scaled_fp4_mm(torch::stable::Tensor& D,
                           torch::stable::Tensor const& A,
                           torch::stable::Tensor const& B,
                           torch::stable::Tensor const& A_sf,
                           torch::stable::Tensor const& B_sf,
                           torch::stable::Tensor const& alpha);
...
void silu_and_mul_nvfp4_quant(torch::stable::Tensor& out,
                              torch::stable::Tensor& output_block_scale,
                              torch::stable::Tensor& input,
                              torch::stable::Tensor& input_global_scale);
```
**EN:** A large portion of the file is dedicated to FP4/NVFP4 support: plain scaled matmul, grouped expert matmul, quantization helpers, and fused activation+quantization entrypoints.
**CN:** 文件中很大一部分都在为 FP4/NVFP4 提供声明：包括普通 scaled matmul、按 expert 分组的 matmul、量化辅助函数，以及融合激活与量化的入口。

### AWQ and other extension points
```cpp
torch::stable::Tensor awq_gemm(torch::stable::Tensor _in_feats,
                               torch::stable::Tensor _kernel,
                               torch::stable::Tensor _scaling_factors,
                               torch::stable::Tensor _zeros,
                               int64_t split_k_iters);
...
// DSV3 fused A GEMM: conditionally compiled so declaration and impl
// registration are in the source file (dsv3_fused_a_gemm.cu)
...
torch::stable::Tensor hadacore_transform(torch::stable::Tensor& x,
                                         bool inplace);
```
**EN:** The tail of the file documents which ops are declared here versus directly in their implementation files. That note is important because some kernels, such as DSV3 fused A GEMM, are conditionally compiled and must keep declaration and registration together.
**CN:** 文件尾部说明了哪些算子在这里声明、哪些算子直接放在实现文件中声明。这一点很重要，因为某些内核（例如 DSV3 fused A GEMM）是条件编译的，必须把声明与注册放在一起。

## Key Concepts / 关键概念
- **Stable ABI interface layer / 稳定 ABI 接口层**: This file is an operator catalog, not an implementation file.
- **Feature grouping / 功能分组**: Quantization, MoE, FP4, AWQ, and misc ops are grouped by domain.
- **Conditional exposure / 条件暴露**: Some declarations stay in source files when build-time guards matter.

## Dependencies / 依赖关系
- Includes `torch/csrc/stable/library.h` and `torch/csrc/stable/tensor.h` for stable-ABI operator signatures.
- Implementations live across many `.cu` and `.cc` files under `csrc/libtorch_stable`.
