# get_group_starts.cuh — Code Analysis / 代码分析

## Source / 来源
- **File**: `./csrc/libtorch_stable/quantization/cutlass_w4a8/get_group_starts.cuh`
- **Repository**: vllm-project/vllm
- **Purpose**: Builds per-expert pointer arrays for grouped W4A8 GEMM launches. / 为 grouped W4A8 GEMM 启动构建按 expert 划分的指针数组。

## Line-by-Line Analysis / 逐行分析
### Kernel computes per-expert pointers / 内核计算每个 expert 的指针
```cpp
__global__ void get_group_gemm_starts(
    int64_t* expert_offsets, ElementA** a_offsets, ElementB** b_offsets,
    ElementC** out_offsets, ElementAccumulator** a_scales_offsets,
    ElementAccumulator** b_scales_offsets,
    ElementGroupScale** b_group_scales_offsets, ElementA* a_base_as_int,
    ElementB* b_base_as_int, ElementC* out_base_as_int,
    ElementAccumulator* a_scales_base_as_int,
    ElementAccumulator* b_scales_base_as_int,
    ElementGroupScale* b_group_scales_base_as_int, int64_t n, int64_t k,
    int64_t scale_k) {
  int expert_id = threadIdx.x;

  int64_t expert_offset = expert_offsets[expert_id];

  // same as w8a8
  a_offsets[expert_id] = a_base_as_int + expert_offset * k;
  out_offsets[expert_id] = out_base_as_int + expert_offset * n;
  a_scales_offsets[expert_id] = a_scales_base_as_int + expert_offset;
  b_scales_offsets[expert_id] = b_scales_base_as_int + (n * expert_id);

  // w4a8 specific
  constexpr int pack_factor = 8;  // pack 8 int4 into int32
  b_offsets[expert_id] = b_base_as_int + (expert_id * k * n / pack_factor);
  b_group_scales_offsets[expert_id] =
      b_group_scales_base_as_int + (expert_id * scale_k * n);
}
```
**EN:** Each thread handles one expert and derives offsets for activations, quantized weights, outputs, token scales, channel scales, and group scales from the shared base tensors.
**CN:** 每个线程负责一个 expert，并基于公共底层张量计算激活、量化权重、输出、token scale、channel scale 与 group scale 的偏移指针。

### Type-dispatched launcher macro / 按类型分发的启动宏
```cpp
#define __CALL_GET_STARTS_KERNEL(TENSOR_C_TYPE, C_TYPE)                  \
  else if (out_tensors.scalar_type() == TENSOR_C_TYPE) {                 \
    get_group_gemm_starts<cutlass::float_e4m3_t, int32_t, C_TYPE, float, \
                          cutlass::Array<cutlass::float_e4m3_t, 8>>      \
        <<<1, num_experts, 0, stream>>>(                                 \
            static_cast<int64_t*>(expert_offsets.data_ptr()),            \
            static_cast<cutlass::float_e4m3_t**>(a_ptrs.data_ptr()),     \
            static_cast<int32_t**>(b_ptrs.data_ptr()),                   \
            static_cast<C_TYPE**>(out_ptrs.data_ptr()),                  \
            static_cast<float**>(a_scales_ptrs.data_ptr()),              \
            static_cast<float**>(b_scales_ptrs.data_ptr()),              \
            static_cast<cutlass::Array<cutlass::float_e4m3_t, 8>**>(     \
                b_group_scales_ptrs.data_ptr()),                         \
            static_cast<cutlass::float_e4m3_t*>(a_tensors.data_ptr()),   \
            static_cast<int32_t*>(b_tensors.data_ptr()),                 \
            static_cast<C_TYPE*>(out_tensors.data_ptr()),                \
            static_cast<float*>(a_scales.data_ptr()),                    \
            static_cast<float*>(b_scales.data_ptr()),                    \
            static_cast<cutlass::Array<cutlass::float_e4m3_t, 8>*>(      \
                b_group_scales.data_ptr()),                              \
            n, k, scale_k);                                              \
  }
```
**EN:** The macro expands into a typed kernel launch so the output pointer array matches the runtime Torch scalar type. It currently supports BF16 and FP16 outputs.
**CN:** 该宏会展开为带类型参数的内核启动，从而让输出指针数组与运行时 Torch 标量类型一致。目前支持 BF16 与 FP16 输出。

### Validation and launch wrapper / 参数校验与启动封装
```cpp
void run_get_group_gemm_starts(
    torch::stable::Tensor const& expert_offsets, torch::stable::Tensor& a_ptrs,
    torch::stable::Tensor& b_ptrs, torch::stable::Tensor& out_ptrs,
    torch::stable::Tensor& a_scales_ptrs, torch::stable::Tensor& b_scales_ptrs,
    torch::stable::Tensor& b_group_scales_ptrs,
    torch::stable::Tensor const& a_tensors,
    torch::stable::Tensor const& b_tensors, torch::stable::Tensor& out_tensors,
    torch::stable::Tensor const& a_scales,
    torch::stable::Tensor const& b_scales,
    torch::stable::Tensor const& b_group_scales, const int64_t b_group_size) {
  STD_TORCH_CHECK(a_tensors.scalar_type() ==
                  torch::headeronly::ScalarType::Float8_e4m3fn);
  STD_TORCH_CHECK(
      b_tensors.scalar_type() ==
      torch::headeronly::ScalarType::Int);  // int4 8x packed into int32
  STD_TORCH_CHECK(a_scales.scalar_type() ==
                  torch::headeronly::ScalarType::Float);
  STD_TORCH_CHECK(b_scales.scalar_type() ==
                  torch::headeronly::ScalarType::Float);
  STD_TORCH_CHECK(
      b_group_scales.scalar_type() ==
      torch::headeronly::ScalarType::Float8_e4m3fn);  // the underlying torch
                                                      // type is e4m3
  STD_TORCH_CHECK(
      out_tensors.scalar_type() ==
      torch::headeronly::ScalarType::BFloat16);  // only support bf16 for now
  // expect int64_t to avoid overflow during offset calculations
  STD_TORCH_CHECK(expert_offsets.scalar_type() ==
                  torch::headeronly::ScalarType::Long);

  int num_experts = static_cast<int>(expert_offsets.size(0));
  // logical k, n
  int64_t n = out_tensors.size(1);
  int64_t k = a_tensors.size(1);
  int64_t scale_k = cutlass::ceil_div(k, b_group_size);

  auto stream = get_current_cuda_stream(a_tensors.get_device_index());

  if (false) {
  }
  __CALL_GET_STARTS_KERNEL(torch::headeronly::ScalarType::BFloat16,
                           cutlass::bfloat16_t)
  __CALL_GET_STARTS_KERNEL(torch::headeronly::ScalarType::Half, half)
  else {
    STD_TORCH_CHECK(false, "Invalid output type (must be float16 or bfloat16)");
  }
}
```
**EN:** The wrapper checks tensor dtypes, computes logical `n`, `k`, and `scale_k`, acquires the current CUDA stream, and dispatches the proper instantiation of `get_group_gemm_starts`.
**CN:** 该封装函数会校验张量数据类型，计算逻辑上的 `n`、`k` 和 `scale_k`，获取当前 CUDA stream，并调度正确实例化的 `get_group_gemm_starts`。

## Key Concepts / 关键概念
- Grouped GEMM needs arrays of pointers instead of one contiguous tensor pointer / Grouped GEMM 需要指针数组，而不是单一连续张量指针
- Expert offsets map token ranges to the correct expert-local slices / expert offset 用于把 token 范围映射到正确的 expert 局部切片
- Packed int4 weights require custom offset arithmetic using a pack factor of 8 / 打包 int4 权重需要使用 pack factor=8 的专用偏移计算

## Dependencies / 依赖关系
- Relies on CUDA, CUTLASS scalar types, and `torch::stable::Tensor` / 依赖 CUDA、CUTLASS 标量类型与 `torch::stable::Tensor`
- Used by `w4a8_grouped_mm_entry.cu` before launching grouped GEMM / 由 `w4a8_grouped_mm_entry.cu` 在启动 grouped GEMM 前调用
