# fused_silu_mul_block_quant.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/quantization/fused_kernels/fused_silu_mul_block_quant.cu`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Fuses SiLU gating and per-block quantization so each `(token, group)` block computes its local activation result, scale, and quantized output in one pass. / [CN] 将 SiLU 门控与分块量化融合起来，使每个 `(token, group)` block 在一次遍历中同时得到局部激活结果、scale 和量化输出。

## Line-by-Line Analysis / 逐行分析

### Kernel mapping and addressing
```cpp
template <typename scalar_t, typename scalar_out_t, bool is_scale_transposed,
          int32_t group_size>
__global__ void silu_and_mul_per_block_quant_kernel(
    scalar_out_t* __restrict__ out,  // Output: [num_tokens, hidden_size] in
                                     // FP8/INT8
    float* __restrict__ scales,      // Output: [num_tokens, hidden_size /
                                 // group_size] or [hidden_size / group_size,
                                 // num_tokens]
    scalar_t const* __restrict__ input,  // Input: [num_tokens, hidden_size * 2]
    float const* scale_ub,               // Optional scale upper bound
    int32_t const hidden_size  // Output hidden size (input is 2x this)
) {
  static_assert((group_size & (group_size - 1)) == 0,
                "group_size must be a power of 2 for correct reduction");

  // Grid: (num_tokens, num_groups)
  int const token_idx = blockIdx.x;
  int const group_idx = blockIdx.y;
  int const tid = threadIdx.x;  // tid in [0, group_size)
  int const num_tokens = gridDim.x;

  // Input layout: [gate || up] concatenated along last dimension
  int const input_stride = hidden_size * 2;
  int const group_start = group_idx * group_size;

  // Pointers to this token's data
  scalar_t const* token_input_gate =
      input + token_idx * input_stride + group_start;
  scalar_t const* token_input_up = token_input_gate + hidden_size;
  scalar_out_t* token_output = out + token_idx * hidden_size + group_start;

  // Scale pointer for this group
  int const num_groups = gridDim.y;
  float* group_scale_ptr = is_scale_transposed
                               ? scales + group_idx * num_tokens + token_idx
                               : scales + token_idx * num_groups + group_idx;
```
**EN:** The kernel template binds input type, output quant type, scale layout, and group size at compile time. Each block is mapped to one token and one hidden-state group, then computes raw pointers for the gate slice, up slice, output slice, and scale slot.
**CN:** 该模板内核在编译期绑定输入类型、输出量化类型、scale 布局以及 group size。每个 block 对应一个 token 和一个隐藏维 group，然后计算 gate 片段、up 片段、输出片段与 scale 位置的原始指针。

### In-block max reduction and scale generation
```cpp
  // Shared memory for reduction (compile-time sized)
  __shared__ float shared_max[group_size];

  // Step 1: Each thread loads one element, computes SiLU, stores in register
  float gate = static_cast<float>(token_input_gate[tid]);
  float up = static_cast<float>(token_input_up[tid]);

  // Compute SiLU(gate) * up
  float sigmoid_gate = 1.0f / (1.0f + expf(-gate));
  float silu_gate = gate * sigmoid_gate;
  float result = silu_gate * up;  // Keep in register

  // Step 2: Reduce to find group max
  shared_max[tid] = fabsf(result);
  __syncthreads();

// Power-of-2 reduction (group_size guaranteed to be power of 2)
#pragma unroll
  for (int stride = group_size / 2; stride > 0; stride >>= 1) {
    if (tid < stride) {
      shared_max[tid] = fmaxf(shared_max[tid], shared_max[tid + stride]);
    }
    __syncthreads();
  }

  // Step 3: Compute scale (thread 0), broadcast via shared memory
  if (tid == 0) {
    float group_max = shared_max[0];

    float const quant_range = quant_type_max_v<scalar_out_t>;
    float group_scale = group_max / quant_range;

    // Apply scale upper bound if provided
    if (scale_ub != nullptr) {
      group_scale = fminf(group_scale, *scale_ub);
    }

    // Use minimum safe scaling factor
    group_scale = fmaxf(group_scale, min_scaling_factor<scalar_out_t>::val());

    // Store scale to global memory
    *group_scale_ptr = group_scale;

    // Reuse shared_max[0] to broadcast scale
    shared_max[0] = group_scale;
  }
  __syncthreads();

  float group_scale = shared_max[0];
```
**EN:** Each thread computes one `SiLU(gate) * up` value, stores its absolute value into shared memory, and participates in a power-of-two reduction. Thread 0 converts the final maximum into a quantization scale, applies an optional upper bound, clamps it with `min_scaling_factor`, stores it globally, and broadcasts it through shared memory.
**CN:** 每个线程计算一个 `SiLU(gate) * up` 结果，把其绝对值写入共享内存并参与 2 的幂形式的规约。线程 0 再把最终最大值转换成量化 scale，应用可选上界，用 `min_scaling_factor` 做下界保护，写回全局内存，并通过共享内存广播给整个 block。

### Quantized writeback
```cpp
  // Step 4: Quantize and write output
  token_output[tid] =
      vllm::ScaledQuant<scalar_out_t, false>::quant_fn(result, group_scale);
}
```
**EN:** After the scale is known, each thread quantizes its local result with `ScaledQuant` and writes directly to the output tensor. There is no second pass over the input.
**CN:** 在得到 scale 后，每个线程通过 `ScaledQuant` 对自己的局部结果量化并直接写入输出张量，整个流程不需要再次遍历输入。

### PyTorch-facing dispatch
```cpp
void silu_and_mul_per_block_quant(torch::Tensor& out,
                                  torch::Tensor const& input,
                                  torch::Tensor& scales, int64_t group_size,
                                  std::optional<torch::Tensor> scale_ub,
                                  bool is_scale_transposed) {
  static c10::ScalarType kFp8Type = is_fp8_ocp()
                                        ? c10::ScalarType::Float8_e4m3fn
                                        : c10::ScalarType::Float8_e4m3fnuz;

  TORCH_CHECK(out.dtype() == kFp8Type || out.dtype() == torch::kInt8);
  TORCH_CHECK(out.is_contiguous() && input.is_contiguous());
  TORCH_CHECK(
      input.dtype() == torch::kFloat16 || input.dtype() == torch::kBFloat16,
      "Input must be FP16 or BF16");
  TORCH_CHECK(scales.dtype() == torch::kFloat32, "Scales must be FP32");
  TORCH_CHECK(group_size == 128 || group_size == 64,
              "Unsupported group size: ", group_size);

  if (scale_ub.has_value()) {
    TORCH_CHECK(out.dtype() == kFp8Type);
  }

  int32_t hidden_size = out.size(-1);
  auto num_tokens = input.size(0);
  int32_t num_groups = hidden_size / group_size;

  TORCH_CHECK(input.size(-1) == hidden_size * 2,
              "input last dim must be 2x output hidden_size");
  TORCH_CHECK(hidden_size % group_size == 0,
              "hidden_size must be divisible by group_size");

  const at::cuda::OptionalCUDAGuard device_guard(device_of(input));
  const cudaStream_t stream = at::cuda::getCurrentCUDAStream();

  dim3 grid(num_tokens, num_groups);
  dim3 block(group_size);

  VLLM_DISPATCH_FLOATING_TYPES(
      input.scalar_type(), "silu_and_mul_per_block_quant", [&] {
        using scalar_in_t = scalar_t;

        VLLM_DISPATCH_QUANT_TYPES(
            out.scalar_type(), "silu_and_mul_per_block_quant", [&] {
              using scalar_out_t = scalar_t;

              VLLM_DISPATCH_GROUP_SIZE(group_size, gs, [&] {
                VLLM_DISPATCH_BOOL(is_scale_transposed, transpose_scale, [&] {
                  vllm::silu_and_mul_per_block_quant_kernel<
                      scalar_in_t, scalar_out_t, transpose_scale, gs>
                      <<<grid, block, 0, stream>>>(
                          out.data_ptr<scalar_out_t>(),
                          scales.data_ptr<float>(),
                          input.data_ptr<scalar_in_t>(),
                          scale_ub.has_value() ? scale_ub->data_ptr<float>()
                                               : nullptr,
                          hidden_size);
                });
              });
            });
      });
}
```
**EN:** The host wrapper validates tensor dtypes and shapes, chooses the grid as `(num_tokens, num_groups)`, uses the group size as block size, and dispatches across input types, quant output types, supported group sizes, and transposed/non-transposed scale layouts.
**CN:** 主机侧包装函数会校验张量 dtype 与形状，将 grid 设为 `(num_tokens, num_groups)`、block 大小设为 group size，并在输入类型、量化输出类型、支持的 group size，以及 scale 的转置/非转置布局之间做分发。

## Key Concepts / 关键概念
- EN: This kernel is a simple one-block-per-group design, unlike the persistent activation kernel in `activation_kernels.cu`.
  CN: 该内核采用简单的“一组一个 block”设计，不同于 `activation_kernels.cu` 中的持久化激活内核。
- EN: Scale generation happens on the same pass as activation computation, minimizing memory traffic.
  CN: scale 生成与激活计算在同一次遍历中完成，从而减少内存流量。
- EN: `is_scale_transposed` only changes where scales are written, not how quantization is computed.
  CN: `is_scale_transposed` 只影响 scale 的写入位置，不改变量化计算本身。

## Dependencies / 依赖关系
- EN: Uses `quant_conversions.cuh` and `quantization/utils.cuh` for datatype-specific scale and quantization rules.
  CN: 使用 `quant_conversions.cuh` 与 `quantization/utils.cuh` 提供与类型相关的 scale/量化规则。
- EN: Launches through ATen CUDA helpers and vLLM dispatch macros.
  CN: 通过 ATen CUDA 辅助函数与 vLLM dispatch 宏启动。
- EN: Supports FP8 and int8 outputs via `VLLM_DISPATCH_QUANT_TYPES`.
  CN: 通过 `VLLM_DISPATCH_QUANT_TYPES` 同时支持 FP8 与 int8 输出。
