# spec_decode_utils.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/cpu/spec_decode_utils.cpp`
- **Repository**: vllm-project/vllm
- **Purpose**: Provides CPU utilities for speculative decoding workflows. / 提供推测解码流程使用的 CPU 工具函数。

## Line-by-Line Analysis / 逐行分析
### Preamble and includes (lines 1-3)
```cpp
#include "cpu_types.hpp"

#include <algorithm>
```
**EN:** This opening block pulls in the headers, feature macros, and compile-time aliases that the rest of the file depends on.
**CN:** 这一开头代码块引入了后续实现依赖的头文件、特性宏以及编译期别名。

### Function / Kernel: eagle_prepare_next_token_padded_kernel_impl (lines 39-77)
```cpp
void eagle_prepare_next_token_padded_kernel_impl(
    const torch::Tensor& sampled_token_ids,
    const torch::Tensor& discard_request_mask,
    const torch::Tensor& backup_next_token_ids, torch::Tensor& next_token_ids,
    torch::Tensor& valid_sampled_tokens_count, const int64_t vocab_size,
    const int64_t num_sampled_tokens_per_req, const int64_t num_reqs) {
  const int64_t* sampled_ids_ptr = sampled_token_ids.data_ptr<int64_t>();
  const bool* discard_mask_ptr = discard_request_mask.data_ptr<bool>();
  const int64_t* backup_ids_ptr = backup_next_token_ids.data_ptr<int64_t>();
  int64_t* next_ids_out_ptr = next_token_ids.data_ptr<int64_t>();
  int64_t* valid_count_out_ptr = valid_sampled_tokens_count.data_ptr<int64_t>();

  const int64_t stride = sampled_token_ids.stride(0);

// ...
    }
  }
}
```
**EN:** This host-side function validates inputs and dispatches to the best specialization or launch configuration for the current runtime shape.
**CN:** 这个宿主端函数负责校验输入，并根据当前运行时形状选择最佳专用实现或启动配置。

### Function / Kernel: eagle_step_slot_mapping_metadata_kernel_impl (lines 79-121)
```cpp
void eagle_step_slot_mapping_metadata_kernel_impl(
    const torch::Tensor& positions, const torch::Tensor& block_table,
    torch::Tensor& seq_lens, torch::Tensor& out_clamped_positions,
    torch::Tensor& out_slot_mapping, const int64_t block_size,
    const int64_t max_model_len, const int64_t PAD_ID) {
  const int64_t batch_size = positions.size(0);
  const int64_t input_batch_size = out_slot_mapping.size(0);

  const int64_t* pos_ptr = positions.data_ptr<int64_t>();
  const int32_t* bt_ptr = block_table.data_ptr<int32_t>();
  int32_t* seq_lens_ptr = seq_lens.data_ptr<int32_t>();
  int64_t* out_clamped_ptr = out_clamped_positions.data_ptr<int64_t>();
  int64_t* out_slot_ptr = out_slot_mapping.data_ptr<int64_t>();

// ...
    seq_lens_ptr[req_idx] = new_seq_len;
  }
}
```
**EN:** This host-side function validates inputs and dispatches to the best specialization or launch configuration for the current runtime shape.
**CN:** 这个宿主端函数负责校验输入，并根据当前运行时形状选择最佳专用实现或启动配置。

### Function / Kernel: copy_and_expand_eagle_inputs_kernel_impl (lines 123-209)
```cpp
void copy_and_expand_eagle_inputs_kernel_impl(
    const torch::Tensor& target_token_ids,
    const torch::Tensor& target_positions, const torch::Tensor& next_token_ids,
    torch::Tensor& out_input_ids, torch::Tensor& out_positions,
    torch::Tensor& out_is_rejected_token_mask,
    torch::Tensor& out_is_masked_token_mask,
    torch::Tensor& out_new_token_indices,
    torch::Tensor& out_hidden_state_mapping,
    const torch::Tensor& query_start_loc, const torch::Tensor& query_end_loc,
    const int64_t padding_token_id, const int64_t parallel_drafting_token_id,
    const int64_t total_input_tokens,
    const int64_t num_padding_slots_per_request, const bool shift_input_ids) {
  const int64_t num_reqs = query_end_loc.size(0);

// ...
    }
  }
}
```
**EN:** This host-side function validates inputs and dispatches to the best specialization or launch configuration for the current runtime shape.
**CN:** 这个宿主端函数负责校验输入，并根据当前运行时形状选择最佳专用实现或启动配置。

### Function / Kernel: rejection_greedy_sample_kernel_impl (lines 211-253)
```cpp
void rejection_greedy_sample_kernel_impl(
    torch::Tensor& output_token_ids, const torch::Tensor& cu_num_draft_tokens,
    const torch::Tensor& draft_token_ids, const torch::Tensor& target_argmax,
    const torch::Tensor& bonus_token_ids,
    const std::optional<torch::Tensor>& is_greedy, const int64_t max_spec_len) {
  const int64_t batch_size = cu_num_draft_tokens.size(0);

  int64_t* out_ptr = output_token_ids.data_ptr<int64_t>();
  const int64_t* cu_draft_ptr = cu_num_draft_tokens.data_ptr<int64_t>();
  const int64_t* draft_ids_ptr = draft_token_ids.data_ptr<int64_t>();
  const int64_t* target_argmax_ptr = target_argmax.data_ptr<int64_t>();
  const int64_t* bonus_ids_ptr = bonus_token_ids.data_ptr<int64_t>();
  const bool* greedy_ptr =
      is_greedy.has_value() ? is_greedy.value().data_ptr<bool>() : nullptr;
// ...
    }
  }
}
```
**EN:** This host-side function validates inputs and dispatches to the best specialization or launch configuration for the current runtime shape.
**CN:** 这个宿主端函数负责校验输入，并根据当前运行时形状选择最佳专用实现或启动配置。

### Function / Kernel: rejection_random_sample_kernel_impl (lines 255-320)
```cpp
void rejection_random_sample_kernel_impl(
    torch::Tensor& output_token_ids, const torch::Tensor& cu_num_draft_tokens,
    const torch::Tensor& draft_token_ids,
    const std::optional<torch::Tensor>& draft_probs,
    const torch::Tensor& target_probs, const torch::Tensor& bonus_token_ids,
    const torch::Tensor& recovered_token_ids,
    const torch::Tensor& uniform_probs,
    const std::optional<torch::Tensor>& is_greedy, const int64_t max_spec_len,
    const int64_t vocab_size, const bool no_draft_probs) {
  const int64_t batch_size = cu_num_draft_tokens.size(0);

  int64_t* out_ptr = output_token_ids.data_ptr<int64_t>();
  const int64_t* cu_draft_ptr = cu_num_draft_tokens.data_ptr<int64_t>();
  const int64_t* draft_ids_ptr = draft_token_ids.data_ptr<int64_t>();
// ...
    }
  }
}
```
**EN:** This host-side function validates inputs and dispatches to the best specialization or launch configuration for the current runtime shape.
**CN:** 这个宿主端函数负责校验输入，并根据当前运行时形状选择最佳专用实现或启动配置。

### Function / Kernel: sample_recovered_tokens_kernel_impl (lines 347-407)
```cpp
void sample_recovered_tokens_kernel_impl(
    torch::Tensor& output_token_ids, const torch::Tensor& cu_num_draft_tokens,
    const torch::Tensor& draft_token_ids,
    const std::optional<torch::Tensor>& draft_probs,
    const torch::Tensor& target_probs, const torch::Tensor& inv_q,
    const int64_t vocab_size, const bool no_draft_probs) {
  const int64_t batch_size = cu_num_draft_tokens.size(0);

  int64_t* out_ptr = output_token_ids.data_ptr<int64_t>();
  const int64_t* cu_draft_ptr = cu_num_draft_tokens.data_ptr<int64_t>();
  const int64_t* draft_ids_ptr = draft_token_ids.data_ptr<int64_t>();
  const float* draft_probs_ptr =
      no_draft_probs ? nullptr : draft_probs.value().data_ptr<float>();
  const float* target_probs_ptr = target_probs.data_ptr<float>();
// ...
    }
  }
}
```
**EN:** This host-side function validates inputs and dispatches to the best specialization or launch configuration for the current runtime shape.
**CN:** 这个宿主端函数负责校验输入，并根据当前运行时形状选择最佳专用实现或启动配置。

## Key Concepts / 关键概念
- Tensor layout and low-level dispatch / 张量布局与底层分派

## Dependencies / 依赖关系
- **External libraries / 外部库**: C++ standard library or platform support, C++ standard library
