# gemm.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/cpu/gemm.h`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Provides GEMM / matrix multiplication kernels, tiling strategies, or low-level launch wrappers. This header primarily defines interfaces, templates, constants, and inline helpers shared by compilation units. / 提供 GEMM / 矩阵乘内核、分块策略或底层启动封装。 该头文件主要定义编译单元共享的接口、模板、常量和内联辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-25: Runtime integration and dispatch
```cpp
#pragma once
#include <ATen/native/CPUBlas.h>

#include "common.h"

// amx-bf16
#define TILE_M 16
#define TILE_N 16
#define TILE_K 32

// block size for AMX gemm
constexpr int block_size_m() {
  return 2 * TILE_M;
}
constexpr int block_size_n() {
  return 2 * TILE_N;
}

// define threshold using brgemm (intel AMX)
template <typename T>
inline bool can_use_brgemm(int M);
template <>
inline bool can_use_brgemm<at::BFloat16>(int M) {
  return M > 4;
}
```
**EN:** This section uses `block_size_m`, `block_size_n`, `can_use_brgemm` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`block_size_m`、`block_size_n`、`can_use_brgemm`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 26-48: Runtime integration and dispatch
```cpp
template <>
inline bool can_use_brgemm<at::Half>(int M) {
  return true;
}
// this requires PyTorch 2.7 or above
template <>
inline bool can_use_brgemm<int8_t>(int M) {
  return M > 4;
}

template <>
inline bool can_use_brgemm<uint8_t>(int M) {
  return M > 4;
}

template <>
inline bool can_use_brgemm<at::Float8_e4m3fn>(int M) {
  return M > 4;
}

// work around compiler internal error
#define BLOCK_K 128  // 4 * TILE_K
```
**EN:** This section uses the surrounding logic to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过相关逻辑等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 49-71: Types and data layout
```cpp
// adjust leading dimension size for K
template <typename T>
inline int64_t get_row_size(int64_t K) {
  return K;
}

template <>
inline int64_t get_row_size<int8_t>(int64_t K) {
  return K + sizeof(int32_t);
}

// uint8: mxfp4 or int4
template <>
inline int64_t get_row_size<uint8_t>(int64_t K) {
  return K >> 1;
}

inline int64_t get_row_size(int64_t K, bool use_int8_w8a8) {
  return use_int8_w8a8 ? K + sizeof(int32_t) : K;
}

enum class CPUQuantMethod : int64_t { BF16 = 0, INT8_W8A8 = 1, FP8_W8A16 = 2, INT4_W4A8 = 3 };
```
**EN:** This section defines `CPUQuantMethod`, `get_row_size`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`CPUQuantMethod`、`get_row_size`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 72-96: Types and data layout
```cpp
constexpr bool operator==(CPUQuantMethod a, int64_t b) {
  return static_cast<int64_t>(a) == b;
}

constexpr bool operator==(int64_t a, CPUQuantMethod b) {
  return a == static_cast<int64_t>(b);
}

enum class CPUQuantAlgo : int64_t { AWQ = 0, GPTQ = 1 };

constexpr bool operator==(CPUQuantAlgo a, int64_t b) {
  return static_cast<int64_t>(a) == b;
}

constexpr bool operator==(int64_t a, CPUQuantAlgo b) {
  return a == static_cast<int64_t>(b);
}

inline int64_t get_4bit_block_k_size(int64_t group_size) {
  return group_size > 128 ? 128 : group_size;
}

// pack weight to vnni format
at::Tensor convert_weight_packed(at::Tensor& weight);
```
**EN:** This section defines `CPUQuantAlgo`, `get_4bit_block_k_size`, `convert_weight_packed`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`CPUQuantAlgo`、`get_4bit_block_k_size`、`convert_weight_packed`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 97-126: Runtime integration and dispatch
```cpp
// pack weight to vnni format for int4
std::tuple<at::Tensor, at::Tensor, at::Tensor>
convert_weight_packed_scale_zp(at::Tensor qweight, at::Tensor qzeros, at::Tensor scales);

// moe implementations for int8 w8a8
template <typename scalar_t>
void fused_experts_int8_kernel_impl(
    scalar_t* __restrict__ output,
    scalar_t* __restrict__ ic1,
    scalar_t* __restrict__ ic2,
    uint8_t* __restrict__ A_tmp,
    float* __restrict__ C_tmp,
    uint8_t* __restrict__ Aq_tmp,
    float* __restrict__ As_tmp,
    const scalar_t* __restrict__ input,
    const int8_t* __restrict__ packed_w1,
    const int8_t* __restrict__ packed_w2,
    const float* __restrict__ w1s,
    const float* __restrict__ w2s,
    const float* __restrict__ topk_weights,
    const int32_t* __restrict__ sorted_ids,
    const int32_t* __restrict__ expert_ids,
    const int32_t* __restrict__ offsets,
    int64_t M,
    int64_t N,
    int64_t K,
    int64_t E,
    int64_t topk,
    int64_t num_tokens_post_pad);
```
**EN:** This section uses `convert_weight_packed_scale_zp`, `fused_experts_int8_kernel_impl` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`convert_weight_packed_scale_zp`、`fused_experts_int8_kernel_impl`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 127-154: Runtime integration and dispatch
```cpp
// moe implementations for fp8 w8a16
template <typename scalar_t>
void fused_experts_fp8_kernel_impl(
    scalar_t* __restrict__ output,
    scalar_t* __restrict__ ic0,
    scalar_t* __restrict__ ic1,
    scalar_t* __restrict__ ic2,
    scalar_t* __restrict__ A_tmp,
    scalar_t* __restrict__ B_tmp,
    float* __restrict__ C_tmp,
    const scalar_t* __restrict__ input,
    const at::Float8_e4m3fn* __restrict__ packed_w1,
    const at::Float8_e4m3fn* __restrict__ packed_w2,
    const float* __restrict__ w1s,
    const float* __restrict__ w2s,
    int64_t block_size_N,
    int64_t block_size_K,
    const float* __restrict__ topk_weights,
    const int32_t* __restrict__ sorted_ids,
    const int32_t* __restrict__ expert_ids,
    const int32_t* __restrict__ offsets,
    int64_t M,
    int64_t N,
    int64_t K,
    int64_t E,
    int64_t topk,
    int64_t num_tokens_post_pad);
```
**EN:** This section uses `fused_experts_fp8_kernel_impl` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`fused_experts_fp8_kernel_impl`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 155-173: Templates, aliases, and constants
```cpp
// shared expert implementation for int8 w8a8
template <typename scalar_t>
void shared_expert_int8_kernel_impl(
    scalar_t* __restrict__ output,
    scalar_t* __restrict__ ic1,
    float* __restrict__ C_tmp,
    uint8_t* __restrict__ Aq_tmp,
    float* __restrict__ As_tmp,
    const scalar_t* __restrict__ input,
    const int8_t* __restrict__ packed_w1,
    const int8_t* __restrict__ packed_w2,
    const float* __restrict__ w1s,
    const float* __restrict__ w2s,
    const scalar_t* __restrict__ fused_experts_out,
    float routed_scaling_factor,
    int64_t M,
    int64_t N,
    int64_t K);
```
**EN:** This section defines `shared_expert_int8_kernel_impl`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`shared_expert_int8_kernel_impl`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 174-197: Templates, aliases, and constants
```cpp
template <typename scalar_t>
void fused_experts_int4_w4a8_kernel_impl(
    scalar_t* __restrict__ output,
    scalar_t* __restrict__ ic0,
    scalar_t* __restrict__ ic1,
    scalar_t* __restrict__ ic2,
    uint8_t* __restrict__ A_tmp,
    uint8_t* __restrict__ Aq_tmp,
    float* __restrict__ As_tmp,
    int32_t* __restrict__ Azp_tmp,
    float* __restrict__ C_tmp,
    int8_t* __restrict__ dqB_tmp,
    const scalar_t* __restrict__ input,
    const uint8_t* __restrict__ packed_w1,
    const uint8_t* __restrict__ packed_w2,
    const int8_t* __restrict__ w1z,
    const int8_t* __restrict__ w2z,
    const float* __restrict__ w1s,
    const float* __restrict__ w2s,
    int group_size,
    const float* __restrict__ topk_weights,
    const int32_t* __restrict__ sorted_ids,
    const int32_t* __restrict__ expert_ids,
    const int32_t* __restrict__ offsets,
```
**EN:** This section defines the surrounding logic, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了相关逻辑等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 198-224: Runtime integration and dispatch
```cpp
    int64_t M,
    int64_t N,
    int64_t K,
    int64_t E,
    int64_t topk,
    int64_t num_tokens_post_pad);

template <typename scalar_t>
void shared_expert_fp8_kernel_impl(
    scalar_t* __restrict__ output,
    scalar_t* __restrict__ ic0,
    scalar_t* __restrict__ ic1,
    scalar_t* __restrict__ B_tmp,
    float* __restrict__ C_tmp,
    const scalar_t* __restrict__ input,
    const at::Float8_e4m3fn* __restrict__ packed_w1,
    const at::Float8_e4m3fn* __restrict__ packed_w2,
    const float* __restrict__ w1s,
    const float* __restrict__ w2s,
    int64_t block_size_N,
    int64_t block_size_K,
    const scalar_t* __restrict__ fused_experts_out,
    float routed_scaling_factor,
    int64_t M,
    int64_t N,
    int64_t K);
```
**EN:** This section uses `shared_expert_fp8_kernel_impl` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`shared_expert_fp8_kernel_impl`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 225-248: Templates, aliases, and constants
```cpp
// tinygemm interface
template <typename scalar_t>
void tinygemm_kernel(
    const scalar_t* __restrict__ A,
    const scalar_t* __restrict__ B,
    scalar_t* __restrict__ C,
    float* __restrict__ Ctmp,
    int64_t M,
    int64_t N,
    int64_t K,
    int64_t lda,
    int64_t ldb,
    int64_t ldc,
    bool brg);

template <typename scalar_t>
void tinygemm_kernel(
    const uint8_t* __restrict__ A,
    const int8_t* __restrict__ B,
    scalar_t* __restrict__ C,
    int32_t* __restrict__ Ctmp,
    const float* __restrict__ As,
    const float* __restrict__ Bs,
    int64_t M,
```
**EN:** This section defines `tinygemm_kernel`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`tinygemm_kernel`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 249-274: Runtime integration and dispatch
```cpp
    int64_t N,
    int64_t K,
    int64_t lda,
    int64_t ldb,
    int64_t ldc,
    bool brg);

// block quantization
template <typename scalar_t>
void tinygemm_kernel(
    const scalar_t* __restrict__ A,
    const at::Float8_e4m3fn* __restrict__ B,
    scalar_t* __restrict__ C,
    scalar_t* __restrict__ Btmp,
    float* __restrict__ Ctmp,
    const float* __restrict__ scale,
    int64_t M,
    int64_t N,
    int64_t K,
    int64_t lda,
    int64_t ldb,
    int64_t ldc,
    bool brg,
    int64_t block_size_K,
    bool do_unpack = true);
```
**EN:** This section uses `tinygemm_kernel` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`tinygemm_kernel`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 275-298: Runtime integration and dispatch
```cpp
// per tensor quantization
template <typename scalar_t>
void tinygemm_kernel(
    const scalar_t* __restrict__ A,
    const at::Float8_e4m3fn* __restrict__ B,
    scalar_t* __restrict__ C,
    scalar_t* __restrict__ Btmp,
    float* __restrict__ Ctmp,
    float scale,
    int64_t M,
    int64_t N,
    int64_t K,
    int64_t lda,
    int64_t ldb,
    int64_t ldc,
    bool brg);

template <typename scalar_t>
void tinygemm_kernel(
    scalar_t* C,
    float* C_temp,
    const uint8_t* A,
    const float* scales_a,
    const int32_t* qzeros_a,
```
**EN:** This section uses `tinygemm_kernel` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`tinygemm_kernel`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 299-322: Templates, aliases, and constants
```cpp
    const uint8_t* B,
    const float* scales_b,
    const int8_t* qzeros_b,
    const int32_t* compensation,
    int8_t* dqB_tmp,
    int64_t M,
    int64_t K,
    int64_t lda,
    int64_t ldc_f,
    int64_t ldc_s,
    bool store_out,
    bool use_brgemm);

// mxfp4
template <typename scalar_t>
void tinygemm_kernel(
    const scalar_t* __restrict__ A,
    const uint8_t* __restrict__ B,
    scalar_t* __restrict__ C,
    scalar_t* __restrict__ Btmp,
    float* __restrict__ Ctmp,
    const uint8_t* __restrict__ scale,
    int64_t M,
    int64_t N,
```
**EN:** This section defines the surrounding logic, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了相关逻辑等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 323-329: Local implementation details
```cpp
    int64_t K,
    int64_t lda,
    int64_t ldb,
    int64_t ldc,
    bool brg,
    int64_t block_size_K,
    bool do_unpack = true);
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

## Key Concepts / 关键概念
- **Template specialization / 模板特化**: Relies on C++ templates to specialize code paths by type or shape. / 依赖 C++ 模板按类型或形状特化代码路径。
- **Quantization / 量化**: Handles low-precision representations and conversion logic. / 处理低精度表示及其转换逻辑。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `common.h`
- **External headers / 外部头文件**: `ATen/native/CPUBlas.h`
- **Path context / 路径上下文**: cpu / gemm.h
