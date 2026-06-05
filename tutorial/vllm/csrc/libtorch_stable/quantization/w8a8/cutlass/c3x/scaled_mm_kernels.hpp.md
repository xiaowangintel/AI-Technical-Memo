# scaled_mm_kernels.hpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `./csrc/libtorch_stable/quantization/w8a8/cutlass/c3x/scaled_mm_kernels.hpp`
- **Repository**: vllm-project/vllm
- **Purpose**: Declares the public scaled-matmul entry points exported by the CUTLASS C3x backend. / 声明由 CUTLASS C3x 后端导出的公开 scaled matmul 入口函数。

## Line-by-Line Analysis / 逐行分析
### SM90 kernel declarations / SM90 内核声明
```cpp
void cutlass_scaled_mm_sm90_fp8(
    torch::stable::Tensor& out, torch::stable::Tensor const& a,
    torch::stable::Tensor const& b, torch::stable::Tensor const& a_scales,
    torch::stable::Tensor const& b_scales,
    std::optional<torch::stable::Tensor> const& bias);

void cutlass_scaled_mm_sm90_int8(
    torch::stable::Tensor& out, torch::stable::Tensor const& a,
    torch::stable::Tensor const& b, torch::stable::Tensor const& a_scales,
    torch::stable::Tensor const& b_scales,
    std::optional<torch::stable::Tensor> const& bias);

void cutlass_scaled_mm_azp_sm90_int8(
    torch::stable::Tensor& out, torch::stable::Tensor const& a,
    torch::stable::Tensor const& b, torch::stable::Tensor const& a_scales,
    torch::stable::Tensor const& b_scales, torch::stable::Tensor const& azp_adj,
    std::optional<torch::stable::Tensor> const& azp,
    std::optional<torch::stable::Tensor> const& bias);

void cutlass_scaled_mm_blockwise_sm90_fp8(
    torch::stable::Tensor& out, torch::stable::Tensor const& a,
    torch::stable::Tensor const& b, torch::stable::Tensor const& a_scales,
    torch::stable::Tensor const& b_scales);
```
**EN:** These declarations expose SM90 FP8, SM90 int8, AZP-aware int8, and SM90 blockwise FP8 entry points to the rest of the backend.
**CN:** 这些声明向后端其他部分暴露了 SM90 FP8、SM90 int8、带 AZP 的 int8，以及 SM90 blockwise FP8 入口。

### SM100 and SM120 declarations / SM100 与 SM120 声明
```cpp
void cutlass_scaled_mm_sm100_fp8(
    torch::stable::Tensor& out, torch::stable::Tensor const& a,
    torch::stable::Tensor const& b, torch::stable::Tensor const& a_scales,
    torch::stable::Tensor const& b_scales,
    std::optional<torch::stable::Tensor> const& bias);

void cutlass_scaled_mm_sm120_fp8(
    torch::stable::Tensor& out, torch::stable::Tensor const& a,
    torch::stable::Tensor const& b, torch::stable::Tensor const& a_scales,
    torch::stable::Tensor const& b_scales,
    std::optional<torch::stable::Tensor> const& bias);

void cutlass_scaled_mm_blockwise_sm100_fp8(
    torch::stable::Tensor& out, torch::stable::Tensor const& a,
    torch::stable::Tensor const& b, torch::stable::Tensor const& a_scales,
    torch::stable::Tensor const& b_scales);

void cutlass_scaled_mm_blockwise_sm120_fp8(
    torch::stable::Tensor& out, torch::stable::Tensor const& a,
    torch::stable::Tensor const& b, torch::stable::Tensor const& a_scales,
    torch::stable::Tensor const& b_scales);
```
**EN:** The remaining declarations add FP8 and blockwise FP8 entry points for newer SM100 and SM120 architectures.
**CN:** 其余声明为较新的 SM100 与 SM120 架构增加了 FP8 和 blockwise FP8 入口。

## Key Concepts / 关键概念
- **Stable ABI surface / 稳定 ABI 接口面**: This header is the shared declaration layer that connects callers to architecture-specific implementations. / 该头文件是共享的声明层，用于把调用方连接到架构特定实现。

## Dependencies / 依赖关系
- **Implementation units / 实现单元**: Each declaration is defined in a matching `.cu` or dispatch header file under the same directory. / 每个声明都在同目录下匹配的 `.cu` 或 dispatch 头文件中实现。
