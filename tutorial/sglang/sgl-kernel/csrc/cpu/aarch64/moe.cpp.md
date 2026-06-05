# moe.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/cpu/aarch64/moe.cpp`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Provides CPU-side reference implementations, preprocessing, or fallback execution paths. This source file primarily implements kernels, host-side wrappers, bindings, or launch logic. / 提供 CPU 侧参考实现、预处理或回退执行路径。 该源文件主要实现内核、主机侧封装、绑定或启动逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-24: Namespace and shared declarations
```cpp
#include "../common.h"
#include "op.h"

namespace {

// key: expert id, value: input rows and weights for this expert
using expert_to_rows_t = std::map<int, std::vector<std::tuple<int, float>>>;

// for expert_id, row_weight_list in x_per_expert.items():
//   rows, weights = zip(*row_weight_list)
//   x_rows = x[rows]
//   w1, w3 = torch.chunk(w13[expert_id], chunks=2)
//   gate = x_rows @ w1
//   up = x_rows @ w3
//   up *= silu(gate)
//   down = up @ w2[expert_id]
//   down *= weights
//   y.index_add_(0, rows, down)
template <typename scalar_t>
void fused_experts_int8_kernel_impl(
    scalar_t* __restrict__ y,              // [M, K], row major
    const int8_t* __restrict__ x,          // [M, K], row major
    const int8_t* __restrict__ w13,        // [E, K, 2N], per expert [K, N], column major, w1 before w3
    const int8_t* __restrict__ w2,         // [E, N, K], per expert [N, K], column major
```
**EN:** This section organizes shared declarations under namespace scope, keeping related symbols together and reducing naming conflicts.
**CN:** 本段把共享声明组织在命名空间作用域下，使相关符号保持聚合并减少命名冲突。

### Lines 25-48: Runtime integration and dispatch
```cpp
    const float* __restrict__ x_scale,     // [M, 1]
    const float* __restrict__ w13_scale,   // [E, 1, 2N], per expert [1, N], w1 before w3
    const float* __restrict__ w2_scale,    // [E, 1, K], per expert [1, K]
    const expert_to_rows_t& x_per_expert,  // expert id -> related x rows and weights
    int64_t M,
    int64_t N,
    int64_t K,
    int64_t E,
    int64_t topk) {
  TORCH_CHECK(false, "not implemented yet");
}

template <>
void fused_experts_int8_kernel_impl<at::BFloat16>(
    at::BFloat16* __restrict__ y,
    const int8_t* __restrict__ x,
    const int8_t* __restrict__ w13,
    const int8_t* __restrict__ w2,
    const float* __restrict__ x_scale,
    const float* __restrict__ w13_scale,
    const float* __restrict__ w2_scale,
    const expert_to_rows_t& x_per_expert,
    int64_t M,
    int64_t N,
```
**EN:** This section uses `TORCH_CHECK` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`TORCH_CHECK`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 49-73: Runtime integration and dispatch
```cpp
    int64_t K,
    int64_t E,
    int64_t topk) {
  // x dispatch buffer to aggregate all rows per expert
  int64_t max_agg_rows = 0;
  for (const auto& [eid, rows] : x_per_expert) {
    max_agg_rows = std::max<int64_t>(max_agg_rows, rows.size());
  }

  // x_scale_agg[max_agg_rows] + up_scale[max_agg_rows] +
  // gate[max_agg_rows,N] + up[max_agg_rows,N] + down[max_agg_rows,K]
  auto f32_buffer = at::empty({max_agg_rows, 1 + 1 + N + N + K}, at::kFloat);
  float* x_scale_agg = f32_buffer.data_ptr<float>();
  float* up_scale = x_scale_agg + max_agg_rows;
  float* gate = up_scale + max_agg_rows;
  float* up = gate + max_agg_rows * N;
  float* down = up + max_agg_rows * N;
  // x_agg[max_agg_rows,K] + up_q8[max_agg_rows,N]
  auto int8_buffer = at::empty({max_agg_rows, K + N}, at::kChar);
  int8_t* x_agg = int8_buffer.data_ptr<int8_t>();
  int8_t* up_q8 = x_agg + max_agg_rows * K;
  // out[M,K]: accumulated output
  auto out_buffer = at::zeros({M, K}, at::kFloat);
  float* out = out_buffer.data_ptr<float>();
```
**EN:** This section uses the surrounding logic to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过相关逻辑等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 74-100: Control flow and branching
```cpp
  // iterate used experts
  for (const auto& [eid, rows] : x_per_expert) {
    const int64_t n_agg = rows.size();

    // copy input rows using this expert to contiguous buffer
    {
      int8_t* x_agg_ptr = x_agg;
      float* x_scale_agg_ptr = x_scale_agg;
      for (const auto [row, weight] : rows) {
        // int row; float weight;
        std::memcpy(x_agg_ptr, x + row * K, K * sizeof(int8_t));
        *x_scale_agg_ptr = x_scale[row];
        x_agg_ptr += K;
        ++x_scale_agg_ptr;
      }
    }

    // gate = x_agg @ w1
    // up = x_agg @ w3
    // up *= silu(gate)
    {
      // expert specific tensors
      const int8_t* w1e = w13 + eid * 2 * N * K;
      const int8_t* w3e = w1e + N * K;
      const float* w1e_scale = w13_scale + eid * 2 * N;
      const float* w3e_scale = w1e_scale + N;
```
**EN:** This section drives `memcpy`, `n_agg`, `w1e` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`memcpy`、`n_agg`、`w1e`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 101-126: Control flow and branching
```cpp
      // tensor shapes
      // - x_agg:         [n_agg, K], int8, row major
      // - x_scale_agg:   [n_agg, 1], float
      // - w{1,3}e:       [K, N], int8, col major
      // - w{1,3}e_scale: [1, N], float
      // - gate:          [n_agg, N], float, row major
      // - up:            [n_agg, N], float, row major
      // - up_q8:         [n_agg, N], int8, row major
      // - up_scale:      [n_agg, 1], float

      const int slice_size = (n_agg * K * sizeof(int8_t)) > kL2Size ? 64 : 8;
      const int num_slices = (N + slice_size - 1) / slice_size;

      auto mm = [&](int64_t begin, int64_t end) {
        for (int64_t slice_idx = begin; slice_idx < end; ++slice_idx) {
          const int64_t n_start = slice_idx * slice_size;
          const int64_t n_end = std::min(n_start + slice_size, N);
          const int slice_width = static_cast<int>(n_end - n_start);

          const int8_t* w1e_ptr = w1e + n_start * K;
          const int8_t* w3e_ptr = w3e + n_start * K;
          const float* w1e_scale_ptr = w1e_scale + n_start;
          const float* w3e_scale_ptr = w3e_scale + n_start;
          float* gate_ptr = gate + n_start;
          float* up_ptr = up + n_start;
```
**EN:** This section drives `min`, `slice_size`, `num_slices` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`min`、`slice_size`、`num_slices`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 127-150: Runtime integration and dispatch
```cpp
          op::i8mm_matmul(x_agg, w1e_ptr, gate_ptr, n_agg, K, N, slice_width, x_scale_agg, w1e_scale_ptr);
          op::i8mm_matmul(x_agg, w3e_ptr, up_ptr, n_agg, K, N, slice_width, x_scale_agg, w3e_scale_ptr);

          for (int i = 0; i < n_agg; ++i) {
            const float* __restrict__ gate_ptr = gate + n_start + i * N;
            float* __restrict__ up_ptr = up + n_start + i * N;
            // TODO: vectorize
            for (int j = 0; j < slice_width; ++j) {
              up_ptr[j] *= gate_ptr[j] / (1 + std::exp(-gate_ptr[j]));
            }
          }
        }
      };

      at::parallel_for(0, num_slices, 0, mm);
    }

    // quantize
    {
      const int64_t grain = kL1Size / (K * sizeof(float));
      at::parallel_for(0, n_agg, grain, [&](int64_t begin, int64_t end) {
        for (int64_t i = begin; i < end; ++i) {
          op::quantize_row_int8(up_q8 + i * N, up_scale + i, up + i * N, N);
        }
```
**EN:** This section uses `at::parallel_for`, `i8mm_matmul`, `exp` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`at::parallel_for`、`i8mm_matmul`、`exp`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 151-176: Control flow and branching
```cpp
      });
    }

    // down = up @ w2
    {
      // expert specific tensors
      const int8_t* w2e = w2 + eid * K * N;
      const float* w2e_scale = w2_scale + eid * K;

      // tensor shapes
      // - up_q8:       [n_agg, N], int8, row major
      // - up_scale:    [n_agg, 1], float
      // - w2e:         [N, K], int8, col major
      // - w2e_scale:   [1, K], float
      // - down:        [n_agg, K], float, row major
      // - out:         [M, K], float, row major

      const int slice_size = (n_agg * N * sizeof(int8_t)) > kL2Size ? 64 : 8;
      const int num_slices = (K + slice_size - 1) / slice_size;

      auto mm = [&](int64_t begin, int64_t end) {
        for (int64_t slice_idx = begin; slice_idx < end; ++slice_idx) {
          const int64_t n_start = slice_idx * slice_size;
          const int64_t n_end = std::min(n_start + slice_size, K);
          const int slice_width = static_cast<int>(n_end - n_start);
```
**EN:** This section drives `min`, `w2e`, `w2e_scale` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`min`、`w2e`、`w2e_scale`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 177-200: Control flow and branching
```cpp
          {
            const int8_t* w2e_ptr = w2e + n_start * N;
            const float* w2e_scale_ptr = w2e_scale + n_start;
            float* down_ptr = down + n_start;

            op::i8mm_matmul(up_q8, w2e_ptr, down_ptr, n_agg, N, K, slice_width, up_scale, w2e_scale_ptr);
          }

          // accumulate to out buffer
          {
            const float* __restrict__ down_ptr = down + n_start;
            for (const auto [row, weight] : rows) {
              // int row; float weight;
              float* __restrict__ out_ptr = out + n_start + row * K;
              // auto vectorizable
              for (int i = 0; i < slice_width; ++i) {
                out_ptr[i] += down_ptr[i] * weight;
              }
              down_ptr += K;
            }
          }
        }
      };
```
**EN:** This section drives `i8mm_matmul`, `w2e_ptr`, `w2e_scale_ptr` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`i8mm_matmul`、`w2e_ptr`、`w2e_scale_ptr`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 201-220: Runtime integration and dispatch
```cpp
      at::parallel_for(0, num_slices, 0, mm);
    }
  }

  // copy output: float -> bf16
  {
    // tensor shapes
    // - out:    [M, K], float, row major
    // - y:      [M, K], bf16, row major
    const int64_t grain = kL1Size / (K * sizeof(float));
    at::parallel_for(0, M, grain, [&](int64_t begin, int64_t end) {
      const float* out_ptr = out + begin * K;
      bfloat16_t* y_ptr = reinterpret_cast<bfloat16_t*>(y) + begin * K;
      op::f32_to_bf16(out_ptr, y_ptr, (end - begin) * K);
    });
  }
}

}  // anonymous namespace
```
**EN:** This section uses `at::parallel_for`, `parallel_for`, `f32_to_bf16` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`at::parallel_for`、`parallel_for`、`f32_to_bf16`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 221-244: Runtime integration and dispatch
```cpp
// hidden_states: [M, K]
// w13: [E, 2N, K]
// w2: [E, K, N]
// topk_weights: [M, topk]
// topk_ids: [M, topk] (int32_t)
// w13_scale: [E, 2N]
// w2_scale: [E, K]
at::Tensor fused_experts_cpu(
    at::Tensor& hidden_states,
    at::Tensor& w13,
    at::Tensor& w2,
    at::Tensor& topk_weights,
    at::Tensor& topk_ids,
    bool inplace,
    int64_t moe_comp_method,
    const std::optional<at::Tensor>& w13_scale,
    const std::optional<at::Tensor>& w2_scale,
    const std::optional<at::Tensor>& /*w13_zero*/,
    const std::optional<at::Tensor>& /*w2_zero*/,
    const std::optional<std::vector<int64_t>> block_size,
    bool /*is_vnni*/) {
  const auto st = hidden_states.scalar_type();
  CHECK_INPUT(hidden_states);
  CHECK_INPUT(w13);
```
**EN:** This section uses `fused_experts_cpu`, `scalar_type`, `CHECK_INPUT` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`fused_experts_cpu`、`scalar_type`、`CHECK_INPUT`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 245-269: Runtime integration and dispatch
```cpp
  CHECK_INPUT(w2);
  CHECK_EQ(topk_weights.sizes(), topk_ids.sizes());
  CHECK_DIM(2, hidden_states);
  CHECK_DIM(3, w13);
  CHECK_DIM(3, w2);
  CHECK_DIM(2, topk_weights);
  CHECK_DIM(2, topk_ids);

  CHECK_EQ(topk_ids.scalar_type(), at::kInt);

  // TODO: support topk_weights to be bf16 or fp16 in the kernel
  auto topk_weights_ = topk_weights.to(at::kFloat);

  int64_t M = hidden_states.size(0);
  int64_t K = hidden_states.size(1);
  int64_t N = w13.size(1) / 2;
  int64_t E = w13.size(0);
  int64_t topk = topk_weights_.size(1);

  // check weight shapes
  CHECK_EQ(w2.size(0), E);
  CHECK_EQ(w2.size(1), K);
  CHECK_EQ(w13.size(2), K);
  CHECK_EQ(w2.size(2), N);
```
**EN:** This section uses `CHECK_INPUT`, `CHECK_EQ`, `CHECK_DIM` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`CHECK_INPUT`、`CHECK_EQ`、`CHECK_DIM`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 270-292: Runtime integration and dispatch
```cpp
  CHECK_EQ(inplace, false);
  at::Tensor out = at::empty_like(hidden_states);

  // expert id -> related input rows and weights
  expert_to_rows_t x_per_expert;  // std::map<int, std::vector<std::tuple<int, float>>>
  {
    const int* ids = topk_ids.data_ptr<int>();
    const float* weights = topk_weights_.data_ptr<float>();
    for (int i = 0; i < M; ++i) {
      for (int j = 0; j < topk; ++j) {
        x_per_expert[*ids].emplace_back(i, *weights);
        ++ids;
        ++weights;
      }
    }
  }

  AT_DISPATCH_REDUCED_FLOATING_TYPES(st, "fused_experts_kernel_impl", [&] {
    auto& w13s = w13_scale.value();
    auto& w2s = w2_scale.value();
    TORCH_CHECK(w13s.numel() == E * 2 * N);
    TORCH_CHECK(w2s.numel() == E * K);
```
**EN:** This section uses `CHECK_EQ`, `empty_like`, `emplace_back` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`CHECK_EQ`、`empty_like`、`emplace_back`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 293-321: Runtime integration and dispatch
```cpp
    // quantize hidden_states
    auto x_buffer = at::empty({M * K}, hidden_states.options().dtype(at::kChar));
    auto x_scale_buffer = at::empty({M}, at::kFloat);
    int8_t* x = x_buffer.data_ptr<int8_t>();
    float* x_scale = x_scale_buffer.data_ptr<float>();
    scalar_t* in = hidden_states.data_ptr<scalar_t>();
    const int64_t grain = kL1Size / (K * sizeof(scalar_t));
    at::parallel_for(0, M, grain, [&](int64_t begin, int64_t end) {
      for (int64_t m = begin; m < end; ++m) {
        op::quantize_row_int8(x + m * K, x_scale + m, in + m * K, K);
      }
    });

    fused_experts_int8_kernel_impl<scalar_t>(
        out.data_ptr<scalar_t>(),
        x,
        w13.data_ptr<int8_t>(),
        w2.data_ptr<int8_t>(),
        x_scale,
        w13s.data_ptr<float>(),
        w2s.data_ptr<float>(),
        x_per_expert,
        M,
        N,
        K,
        E,
        topk);
  });
```
**EN:** This section uses `at::parallel_for`, `options`, `quantize_row_int8` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`at::parallel_for`、`options`、`quantize_row_int8`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 322-323: Local implementation details
```cpp
  return out;
}
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

## Key Concepts / 关键概念
- **Template specialization / 模板特化**: Relies on C++ templates to specialize code paths by type or shape. / 依赖 C++ 模板按类型或形状特化代码路径。
- **Quantization / 量化**: Handles low-precision representations and conversion logic. / 处理低精度表示及其转换逻辑。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `../common.h`, `op.h`
- **External headers / 外部头文件**: Standard library or none / 标准库或无
- **Path context / 路径上下文**: cpu / aarch64 / moe.cpp
