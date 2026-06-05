# sgl_kernel_ops.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/include/sgl_kernel_ops.h`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Contains low-level kernel code and support utilities for the sgl-kernel backend. This header primarily defines interfaces, templates, constants, and inline helpers shared by compilation units. / 包含 sgl-kernel 后端的底层内核代码和配套工具。 该头文件主要定义编译单元共享的接口、模板、常量和内联辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-29: Headers and compile-time setup
```cpp
/* Copyright 2025 SGLang Team. All Rights Reserved.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
==============================================================================*/

#pragma once

#include <ATen/ATen.h>
#include <ATen/Tensor.h>
#include <Python.h>
#include <torch/all.h>
#include <torch/library.h>
#include <torch/torch.h>

#include <tuple>
#include <vector>

#include "scalar_type.hpp"
```
**EN:** This section prepares the file's headers, macros, and compile-time switches so later code can rely on the right platform and API definitions.
**CN:** 本段准备文件依赖的头文件、宏和编译期开关，使后续代码能够依赖正确的平台与 API 定义。

### Lines 30-57: Headers and compile-time setup
```cpp
#ifdef USE_MUSA
#include "sgl_kernel_musa_ops.h"
#endif

#define _CONCAT(A, B) A##B
#define CONCAT(A, B) _CONCAT(A, B)

#define _STRINGIFY(A) #A
#define STRINGIFY(A) _STRINGIFY(A)

#define TORCH_LIBRARY_EXPAND(NAME, MODULE) TORCH_LIBRARY(NAME, MODULE)

#define REGISTER_EXTENSION(NAME)                                                                      \
  PyMODINIT_FUNC CONCAT(PyInit_, NAME)() {                                                            \
    static struct PyModuleDef module = {PyModuleDef_HEAD_INIT, STRINGIFY(NAME), nullptr, 0, nullptr}; \
    return PyModule_Create(&module);                                                                  \
  }

using fptr_t = int64_t;

/*
 * From csrc/allreduce
 */
#ifdef USE_ROCM
// ROCM custom allreduce
fptr_t init_custom_ar(
    torch::Tensor& meta,
    torch::Tensor& rank_data,
```
**EN:** This section prepares the file's headers, macros, and compile-time switches so later code can rely on the right platform and API definitions.
**CN:** 本段准备文件依赖的头文件、宏和编译期开关，使后续代码能够依赖正确的平台与 API 定义。

### Lines 58-91: Runtime integration and dispatch
```cpp
    const std::vector<std::string>& handles,
    const std::vector<int64_t>& offsets,
    int64_t rank,
    bool full_nvlink);
void all_reduce_reg(fptr_t _fa, torch::Tensor& inp, torch::Tensor& out);
void all_reduce_unreg(fptr_t _fa, torch::Tensor& inp, torch::Tensor& reg_buffer, torch::Tensor& out);
void dispose(fptr_t _fa);
int64_t meta_size();
void register_buffer(
    fptr_t _fa, torch::Tensor& t, const std::vector<std::string>& handles, const std::vector<int64_t>& offsets);
std::tuple<torch::Tensor, std::vector<int64_t>> get_graph_buffer_ipc_meta(fptr_t _fa);
void register_graph_buffers(
    fptr_t _fa, const std::vector<std::string>& handles, const std::vector<std::vector<int64_t>>& offsets);
torch::Tensor allocate_meta_buffer(int64_t size);
torch::Tensor get_meta_buffer_ipc_handle(torch::Tensor& inp);
// quick allreduce
fptr_t init_custom_qr(int64_t rank, int64_t world_size, std::optional<int64_t> qr_max_size = std::nullopt);
void qr_destroy(fptr_t _fa);
torch::Tensor qr_get_handle(fptr_t _fa);
void qr_open_handles(fptr_t _fa, const std::vector<torch::Tensor>& handles);
void qr_all_reduce(fptr_t _fa, torch::Tensor& inp, torch::Tensor& out, int64_t quant_level, bool cast_bf2half = false);
int64_t qr_max_size();
#else
// custom allreduce
fptr_t
init_custom_ar(const std::vector<fptr_t>& fake_ipc_ptrs, torch::Tensor& rank_data, int64_t rank, bool full_nvlink);
void dispose(fptr_t _fa);
int64_t meta_size();
void all_reduce(fptr_t _fa, torch::Tensor& inp, torch::Tensor& out, fptr_t _reg_buffer, int64_t reg_buffer_sz_bytes);
std::tuple<std::vector<int64_t>, std::vector<int64_t>> get_graph_buffer_ipc_meta(fptr_t _fa);
void register_buffer(fptr_t _fa, const std::vector<fptr_t>& fake_ipc_ptrs);
void register_graph_buffers(
    fptr_t _fa, const std::vector<std::vector<int64_t>>& handles, const std::vector<std::vector<int64_t>>& offsets);
```
**EN:** This section uses `all_reduce_reg`, `all_reduce_unreg`, `dispose` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`all_reduce_reg`、`all_reduce_unreg`、`dispose`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 92-119: Runtime integration and dispatch
```cpp
// mscclpp
torch::Tensor mscclpp_generate_unique_id();
fptr_t mscclpp_init_context(
    const torch::Tensor& unique_id,
    const int64_t rank,
    const int64_t world_size,
    torch::Tensor& scratch,
    torch::Tensor& put_buffer,
    const int64_t nranks_per_node,
    const std::vector<int64_t>& rank_to_node,
    const std::vector<int64_t>& rank_to_ib,
    const int64_t context_selection);
void mscclpp_allreduce(fptr_t _context, torch::Tensor& inp, torch::Tensor& out, int64_t nthreads, int64_t nblocks);
#endif

/*
 * From csrc/attention
 */
void merge_state_v2(
    at::Tensor v_a, at::Tensor s_a, at::Tensor v_b, at::Tensor s_b, at::Tensor v_merged, at::Tensor s_merged);
void cutlass_mla_decode(
    torch::Tensor const& out,
    torch::Tensor const& q_nope,
    torch::Tensor const& q_pe,
    torch::Tensor const& kv_c_and_k_pe_cache,
    torch::Tensor const& seq_lens,
    torch::Tensor const& page_table,
    torch::Tensor const& workspace,
```
**EN:** This section uses `mscclpp_generate_unique_id`, `mscclpp_init_context`, `mscclpp_allreduce` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`mscclpp_generate_unique_id`、`mscclpp_init_context`、`mscclpp_allreduce`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 120-149: Runtime integration and dispatch
```cpp
    double sm_scale,
    int64_t num_kv_splits = 1 /* Set to 1 to avoid cuda_graph issue by default. */);
int64_t cutlass_mla_get_workspace_size(
    int64_t max_seq_len,
    int64_t num_batches,
    int64_t sm_count = 0,
    int64_t num_kv_splits = 1 /* Set to 1 to avoid cuda_graph issue by default. */);

/*
 * From csrc/elementwise
 */
void rmsnorm(at::Tensor& output, at::Tensor& input, at::Tensor& weight, double eps, bool enable_pdl);
void sgl_fused_add_rmsnorm(
    torch::Tensor input, torch::Tensor residual, torch::Tensor weight, double eps, bool enable_pdl);
void musa_fused_add_rms_norm(
    torch::Tensor& input, torch::Tensor& residual, torch::Tensor& weight, double epsilon, bool enable_pdl);
void gemma_rmsnorm(at::Tensor& output, at::Tensor& input, at::Tensor& weight, double eps, bool enable_pdl);
void gemma_fused_add_rmsnorm(at::Tensor& input, at::Tensor& residual, at::Tensor& weight, double eps, bool enable_pdl);
void silu_and_mul(at::Tensor& out, at::Tensor& input);
void gelu_tanh_and_mul(at::Tensor& out, at::Tensor& input);
void gelu_and_mul(at::Tensor& out, at::Tensor& input);

void rotary_embedding(
    torch::Tensor& positions,
    torch::Tensor& query,
    std::optional<torch::Tensor> key,
    int64_t head_size,
    torch::Tensor& cos_sin_cache,
    bool is_neox);
```
**EN:** This section uses `cutlass_mla_get_workspace_size`, `rmsnorm`, `sgl_fused_add_rmsnorm` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`cutlass_mla_get_workspace_size`、`rmsnorm`、`sgl_fused_add_rmsnorm`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 150-176: Runtime integration and dispatch
```cpp
void copy_to_gpu_no_ce(const at::Tensor& input, at::Tensor& output);
void concat_mla_k(torch::Tensor k, torch::Tensor k_nope, torch::Tensor k_rope);
void concat_mla_absorb_q(at::Tensor a, at::Tensor b, at::Tensor out);

void fast_topk_interface(
    const at::Tensor& score,
    at::Tensor& indices,
    const at::Tensor& lengths,
    std::optional<at::Tensor> row_starts_opt = std::nullopt);
void fast_topk_transform_interface(
    const at::Tensor& score,
    const at::Tensor& lengths,
    at::Tensor& dst_page_table,
    const at::Tensor& src_page_table,
    const at::Tensor& cu_seqlens_q,
    std::optional<at::Tensor> row_starts_opt = std::nullopt);
void fast_topk_transform_ragged_interface(
    const at::Tensor& score,
    const at::Tensor& lengths,
    at::Tensor& topk_indices_ragged,
    const at::Tensor& topk_indices_offset,
    std::optional<at::Tensor> row_starts_opt = std::nullopt);

#ifdef USE_ROCM
void gelu_quick(at::Tensor& out, const at::Tensor& input);
#endif
```
**EN:** This section uses `copy_to_gpu_no_ce`, `concat_mla_k`, `concat_mla_absorb_q` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`copy_to_gpu_no_ce`、`concat_mla_k`、`concat_mla_absorb_q`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 177-204: Runtime integration and dispatch
```cpp
/*
 * From csrc/gemm
 */
torch::Tensor awq_dequantize(torch::Tensor qweight, torch::Tensor scales, torch::Tensor qzeros);
torch::Tensor int8_scaled_mm(
    const torch::Tensor& mat_a,
    const torch::Tensor& mat_b,
    const torch::Tensor& scales_a,
    const torch::Tensor& scales_b,
    const torch::Dtype& out_dtype,
    const c10::optional<torch::Tensor>& bias);
torch::Tensor fp8_scaled_mm(
    const torch::Tensor& mat_a,
    const torch::Tensor& mat_b,
    const torch::Tensor& scales_a,
    const torch::Tensor& scales_b,
    const torch::Dtype& out_dtype,
    const c10::optional<torch::Tensor>& bias);
torch::Tensor fp8_blockwise_scaled_mm(
    const torch::Tensor& mat_a,
    const torch::Tensor& mat_b,
    const torch::Tensor& scales_a,
    const torch::Tensor& scales_b,
    const torch::Dtype& out_dtype);
void sgl_per_token_group_quant_8bit(
    at::Tensor input,
    at::Tensor output_q,
    at::Tensor output_s,
```
**EN:** This section uses `awq_dequantize`, `int8_scaled_mm`, `fp8_scaled_mm` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`awq_dequantize`、`int8_scaled_mm`、`fp8_scaled_mm`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 205-232: Runtime integration and dispatch
```cpp
    int64_t group_size,
    double eps,
    double fp8_min,
    double fp8_max,
    bool scale_ue8m0);
void sgl_per_token_group_quant_8bit_v2(
    at::Tensor input,
    at::Tensor output_q,
    at::Tensor output_s,
    int64_t group_size,
    double eps,
    double min_8bit,
    double max_8bit,
    bool scale_ue8m0,
    bool fuse_silu_and_mul,
    const std::optional<torch::Tensor>& masked_m);
void sgl_per_token_quant_fp8(at::Tensor input, at::Tensor output_q, at::Tensor output_s);
void bmm_fp8(
    at::Tensor A,
    at::Tensor B,
    at::Tensor D,
    at::Tensor A_scale,
    at::Tensor B_scale,
    at::Tensor workspace_buffer,
    int64_t cublas_handle);
void dsv3_router_gemm(torch::Tensor& output, const torch::Tensor& mat_a, const torch::Tensor& mat_b);
void dsv3_fused_a_gemm(torch::Tensor& output, torch::Tensor const& mat_a, torch::Tensor const& mat_b);
```
**EN:** This section uses `sgl_per_token_group_quant_8bit_v2`, `sgl_per_token_quant_fp8`, `bmm_fp8` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`sgl_per_token_group_quant_8bit_v2`、`sgl_per_token_quant_fp8`、`bmm_fp8`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 233-264: Runtime integration and dispatch
```cpp
torch::Tensor gptq_gemm(
    torch::Tensor a,
    torch::Tensor b_q_weight,
    torch::Tensor b_gptq_qzeros,
    torch::Tensor b_gptq_scales,
    torch::Tensor b_g_idx,
    bool use_shuffle,
    int64_t bit);

void gptq_shuffle(torch::Tensor q_weight, torch::Tensor q_perm, int64_t bit);

/*
 * From csrc/moe
 */
void moe_align_block_size(
    torch::Tensor topk_ids,
    int64_t num_experts,
    int64_t block_size,
    torch::Tensor sorted_token_ids,
    torch::Tensor experts_ids,
    torch::Tensor num_tokens_post_pad,
    torch::Tensor cumsum_buffer,
    bool pad_sorted_token_ids);

void topk_softmax(
    torch::Tensor& topk_weights,
    torch::Tensor& topk_indices,
    torch::Tensor& gating_output,
    bool renormalize,
    double moe_softcapping,
    const c10::optional<torch::Tensor>& correction_bias);
```
**EN:** This section uses `gptq_gemm`, `gptq_shuffle`, `moe_align_block_size` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`gptq_gemm`、`gptq_shuffle`、`moe_align_block_size`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 265-293: Runtime integration and dispatch
```cpp
void topk_sigmoid(
    torch::Tensor& topk_weights,
    torch::Tensor& topk_indices,
    torch::Tensor& gating_output,
    bool renormalize,
    const c10::optional<torch::Tensor>& correction_bias);

void moe_sum_reduce(at::Tensor& input, at::Tensor& output, double routed_scaling_factor);

void moe_sum(torch::Tensor& input, torch::Tensor& output);

std::vector<at::Tensor> moe_fused_gate(
    at::Tensor& input,
    at::Tensor& bias,
    int64_t num_expert_group,
    int64_t topk_group,
    int64_t topk,
    int64_t num_fused_shared_experts,
    double routed_scaling_factor,
    bool apply_routed_scaling_factor_on_output);

std::vector<at::Tensor> kimi_k2_moe_fused_gate(
    at::Tensor& input,
    at::Tensor& bias,
    int64_t topk,
    bool renormalize,
    double routed_scaling_factor,
    bool apply_routed_scaling_factor_on_output);
```
**EN:** This section uses `topk_sigmoid`, `moe_sum_reduce`, `moe_sum` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`topk_sigmoid`、`moe_sum_reduce`、`moe_sum`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 294-325: Runtime integration and dispatch
```cpp
void fp8_blockwise_scaled_grouped_mm(
    torch::Tensor& output,
    torch::Tensor& a_ptrs,
    torch::Tensor& b_ptrs,
    torch::Tensor& out_ptrs,
    torch::Tensor& a_scales_ptrs,
    torch::Tensor& b_scales_ptrs,
    const torch::Tensor& a,
    const torch::Tensor& b,
    const torch::Tensor& scales_a,
    const torch::Tensor& scales_b,
    const torch::Tensor& stride_a,
    const torch::Tensor& stride_b,
    const torch::Tensor& stride_c,
    const torch::Tensor& layout_sfa,
    const torch::Tensor& layout_sfb,
    const torch::Tensor& problem_sizes,
    const torch::Tensor& expert_offsets,
    const torch::Tensor& workspace);

void prepare_moe_input(
    const torch::Tensor& topk_ids,
    torch::Tensor& expert_offsets,
    const std::optional<torch::Tensor>& blockscale_offsets,
    torch::Tensor& problem_sizes1,
    torch::Tensor& problem_sizes2,
    torch::Tensor& input_permutation,
    torch::Tensor& output_permutation,
    const int64_t num_experts,
    const int64_t n,
    const int64_t k);
```
**EN:** This section uses `fp8_blockwise_scaled_grouped_mm`, `prepare_moe_input` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`fp8_blockwise_scaled_grouped_mm`、`prepare_moe_input`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 326-351: Runtime integration and dispatch
```cpp
void shuffle_rows(const torch::Tensor& input_tensor, const torch::Tensor& dst2src_map, torch::Tensor& output_tensor);

void apply_shuffle_mul_sum(
    const torch::Tensor& input,
    torch::Tensor& output,
    const torch::Tensor& permutation,
    const std::optional<torch::Tensor>& factors);

void fused_qk_norm_rope(
    torch::Tensor& qkv,
    int64_t num_heads_q,
    int64_t num_heads_k,
    int64_t num_heads_v,
    int64_t head_dim,
    double eps,
    torch::Tensor& q_weight,
    torch::Tensor& k_weight,
    double base,
    bool is_neox,
    torch::Tensor& position_ids,
    double factor,
    double low,
    double high,
    double attention_factor,
    int64_t rotary_dim);
```
**EN:** This section uses `shuffle_rows`, `apply_shuffle_mul_sum`, `fused_qk_norm_rope` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`shuffle_rows`、`apply_shuffle_mul_sum`、`fused_qk_norm_rope`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 352-379: Runtime integration and dispatch
```cpp
/*
 * From csrc/moe/cutlass_moe/w4a8
 */
void get_cutlass_w4a8_moe_mm_data(
    const torch::Tensor& topk_ids,
    torch::Tensor& expert_offsets,
    torch::Tensor& problem_sizes1,
    torch::Tensor& problem_sizes2,
    torch::Tensor& input_permutation,
    torch::Tensor& output_permutation,
    const int64_t num_experts,
    const int64_t n,
    const int64_t k);

void cutlass_w4a8_moe_mm(
    torch::Tensor& d_tensors,
    torch::Tensor const& a_tensors,
    torch::Tensor const& b_tensors,
    torch::Tensor const& a_scales,
    torch::Tensor const& b_scales,
    torch::Tensor const& expert_offsets,
    torch::Tensor const& problem_sizes,
    torch::Tensor const& a_strides,
    torch::Tensor const& b_strides,
    torch::Tensor const& d_strides,
    torch::Tensor const& s_strides,
    int64_t chunk_size,
    int64_t topk);
```
**EN:** This section uses `get_cutlass_w4a8_moe_mm_data`, `cutlass_w4a8_moe_mm` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`get_cutlass_w4a8_moe_mm_data`、`cutlass_w4a8_moe_mm`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 380-408: Runtime integration and dispatch
```cpp
/*
 * From csrc/speculative
 */
void tree_speculative_sampling_target_only(
    at::Tensor predicts,          // mutable
    at::Tensor accept_index,      // mutable
    at::Tensor accept_token_num,  // mutable
    at::Tensor candidates,
    at::Tensor retrive_index,
    at::Tensor retrive_next_token,
    at::Tensor retrive_next_sibling,
    at::Tensor uniform_samples,
    at::Tensor uniform_samples_for_final_sampling,
    at::Tensor target_probs,
    at::Tensor draft_probs,
    double threshold_single = 1,
    double threshold_acc = 1,
    bool deterministic = true);

void verify_tree_greedy(
    at::Tensor predicts,          // mutable
    at::Tensor accept_index,      // mutable
    at::Tensor accept_token_num,  // mutable
    at::Tensor candidates,
    at::Tensor retrive_index,
    at::Tensor retrive_next_token,
    at::Tensor retrive_next_sibling,
    at::Tensor target_predict);
```
**EN:** This section uses `tree_speculative_sampling_target_only`, `verify_tree_greedy` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`tree_speculative_sampling_target_only`、`verify_tree_greedy`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 409-440: Runtime integration and dispatch
```cpp
void reconstruct_indices_from_tree_mask(
    at::Tensor tree_mask,
    at::Tensor verified_seq_len,
    at::Tensor positions,             // mutable
    at::Tensor retrive_index,         // mutable
    at::Tensor retrive_next_token,    // mutable
    at::Tensor retrive_next_sibling,  // mutable
    int64_t batch_size,
    int64_t draft_token_num);

void build_tree_kernel_efficient(
    at::Tensor parent_list,
    at::Tensor selected_index,
    at::Tensor verified_seq_len,
    at::Tensor tree_mask,
    at::Tensor positions,
    at::Tensor retrive_index,
    at::Tensor retrive_next_token,
    at::Tensor retrive_next_sibling,
    int64_t topk,
    int64_t depth,
    int64_t draft_token_num,
    int64_t tree_mask_mode);

void segment_packbits(
    at::Tensor x,
    at::Tensor input_indptr,
    at::Tensor output_indptr,
    at::Tensor y,
    int64_t batch_size,
    int64_t cuda_stream = 0);
```
**EN:** This section uses `reconstruct_indices_from_tree_mask`, `build_tree_kernel_efficient`, `segment_packbits` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`reconstruct_indices_from_tree_mask`、`build_tree_kernel_efficient`、`segment_packbits`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 441-467: Device helpers and synchronization
```cpp
/*
 * From csrc/kvcacheio
 */
void transfer_kv_per_layer(
    const at::Tensor src_k,
    at::Tensor dst_k,
    const at::Tensor src_v,
    at::Tensor dst_v,
    const at::Tensor src_indices,
    const at::Tensor dst_indices,
    int64_t item_size,
    int64_t block_quota,
    int64_t num_warps_per_block);

void transfer_kv_per_layer_pf_lf(
    const at::Tensor src_k,
    at::Tensor dst_k,
    const at::Tensor src_v,
    at::Tensor dst_v,
    const at::Tensor src_indices,
    const at::Tensor dst_indices,
    int64_t layer_id,
    int64_t item_size,
    int64_t src_layout_dim,
    int64_t block_quota,
    int64_t num_warps_per_block);
```
**EN:** This section implements `transfer_kv_per_layer`, `transfer_kv_per_layer_pf_lf`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`transfer_kv_per_layer`、`transfer_kv_per_layer_pf_lf`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 468-494: Device helpers and synchronization
```cpp
void transfer_kv_per_layer_ph_lf(
    const at::Tensor src_k,
    at::Tensor dst_k,
    const at::Tensor src_v,
    at::Tensor dst_v,
    const at::Tensor src_indices,
    const at::Tensor dst_indices,
    int64_t layer_id,
    int64_t item_size,
    int64_t src_layout_dim,
    int64_t page_size,
    int64_t head_num,
    int64_t block_quota,
    int64_t num_warps_per_block);

void transfer_kv_all_layer(
    const at::Tensor src_k_layers,
    const at::Tensor dst_k_layers,
    const at::Tensor src_v_layers,
    const at::Tensor dst_v_layers,
    const at::Tensor src_indices,
    const at::Tensor dst_indices,
    int64_t item_size,
    int64_t num_layers,
    int64_t block_quota,
    int64_t num_warps_per_block);
```
**EN:** This section implements `transfer_kv_per_layer_ph_lf`, `transfer_kv_all_layer`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`transfer_kv_per_layer_ph_lf`、`transfer_kv_all_layer`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 495-522: Device helpers and synchronization
```cpp
void transfer_kv_all_layer_lf_pf(
    const at::Tensor src_k_layers,
    at::Tensor dst_k,
    const at::Tensor src_v_layers,
    at::Tensor dst_v,
    const at::Tensor src_indices,
    const at::Tensor dst_indices,
    int64_t item_size,
    int64_t dst_layout_dim,
    int64_t num_layers,
    int64_t block_quota,
    int64_t num_warps_per_block);

void transfer_kv_all_layer_lf_ph(
    const at::Tensor src_k_layers,
    at::Tensor dst_k,
    const at::Tensor src_v_layers,
    at::Tensor dst_v,
    const at::Tensor src_indices,
    const at::Tensor dst_indices,
    int64_t item_size,
    int64_t dst_layout_dim,
    int64_t num_layers,
    int64_t page_size,
    int64_t head_num,
    int64_t block_quota,
    int64_t num_warps_per_block);
```
**EN:** This section implements `transfer_kv_all_layer_lf_pf`, `transfer_kv_all_layer_lf_ph`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`transfer_kv_all_layer_lf_pf`、`transfer_kv_all_layer_lf_ph`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 523-552: Device helpers and synchronization
```cpp
void transfer_kv_per_layer_mla(
    const at::Tensor src,
    at::Tensor dst,
    const at::Tensor src_indices,
    const at::Tensor dst_indices,
    int64_t item_size,
    int64_t block_quota,
    int64_t num_warps_per_block);

void transfer_kv_per_layer_mla_pf_lf(
    const at::Tensor src,
    at::Tensor dst,
    const at::Tensor src_indices,
    const at::Tensor dst_indices,
    int64_t layer_id,
    int64_t item_size,
    int64_t src_layout_dim,
    int64_t block_quota,
    int64_t num_warps_per_block);

void transfer_kv_all_layer_mla(
    const at::Tensor src_layers,
    const at::Tensor dst_layers,
    const at::Tensor src_indices,
    const at::Tensor dst_indices,
    int64_t item_size,
    int64_t num_layers,
    int64_t block_quota,
    int64_t num_warps_per_block);
```
**EN:** This section implements `transfer_kv_per_layer_mla`, `transfer_kv_per_layer_mla_pf_lf`, `transfer_kv_all_layer_mla`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`transfer_kv_per_layer_mla`、`transfer_kv_per_layer_mla_pf_lf`、`transfer_kv_all_layer_mla`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 553-578: Device helpers and synchronization
```cpp
void transfer_kv_all_layer_mla_lf_pf(
    const at::Tensor src_layers,
    at::Tensor dst,
    const at::Tensor src_indices,
    const at::Tensor dst_indices,
    int64_t item_size,
    int64_t dst_layout_dim,
    int64_t num_layers,
    int64_t block_quota,
    int64_t num_warps_per_block);

void transfer_kv_direct(
    const std::vector<at::Tensor>& src_layers,
    std::vector<at::Tensor> dst_layers,
    const at::Tensor src_indices,
    const at::Tensor dst_indices,
    int64_t page_size);

void transfer_kv_per_layer_direct_pf_lf(
    const std::vector<at::Tensor>& src_ptrs,
    std::vector<at::Tensor> dst_ptrs,
    const at::Tensor& src_indices,
    const at::Tensor& dst_indices,
    int64_t layer_id,
    int64_t page_size);
```
**EN:** This section implements `transfer_kv_all_layer_mla_lf_pf`, `transfer_kv_direct`, `transfer_kv_per_layer_direct_pf_lf`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`transfer_kv_all_layer_mla_lf_pf`、`transfer_kv_direct`、`transfer_kv_per_layer_direct_pf_lf`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 579-606: Runtime integration and dispatch
```cpp
void transfer_kv_all_layer_direct_lf_pf(
    const std::vector<at::Tensor>& src_ptrs,
    std::vector<at::Tensor> dst_ptrs,
    const at::Tensor& src_indices,
    const at::Tensor& dst_indices,
    int64_t page_size);

/*
 * From csrc/memory
 */
at::Tensor weak_ref_tensor(const at::Tensor& tensor);

/*
 * From FlashInfer
 */
void top_k_renorm_probs(
    at::Tensor probs, at::Tensor renorm_probs, std::optional<at::Tensor> maybe_top_k_arr, int64_t top_k_val);

void top_p_renorm_probs(
    at::Tensor probs, at::Tensor renorm_probs, std::optional<at::Tensor> maybe_top_p_arr, double top_p_val);

namespace flash {
/*
 * From fa2 sparse
 */
std::vector<at::Tensor> mha_fwd_sparse(
    at::Tensor& q,        // batch_size x seqlen_q x num_heads x head_size
    const at::Tensor& k,  // batch_size x seqlen_k x num_heads_k x head_size
```
**EN:** This section uses `transfer_kv_all_layer_direct_lf_pf`, `weak_ref_tensor`, `top_k_renorm_probs` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`transfer_kv_all_layer_direct_lf_pf`、`weak_ref_tensor`、`top_k_renorm_probs`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 607-634: Runtime integration and dispatch
```cpp
    const at::Tensor& v,  // batch_size x seqlen_k x num_heads_k x head_size
    const at::Tensor& block_count,
    const at::Tensor& block_offset,
    const at::Tensor& column_count,
    const at::Tensor& column_index,
    const std::optional<at::Tensor>& out_,           // batch_size x seqlen_q x num_heads x head_size
    const std::optional<at::Tensor>& alibi_slopes_,  // num_heads or batch_size x num_heads
    const double p_dropout,
    const double softmax_scale,
    bool is_causal,
    const double softcap,
    const bool return_softmax,
    std::optional<at::Generator> gen_);

std::vector<at::Tensor> mha_varlen_fwd_sparse(
    at::Tensor& q,        // total_q x num_heads x head_size, total_q := \sum_{i=0}^{b} s_i
    const at::Tensor& k,  // total_k x num_heads_k x head_size, total_k := \sum_{i=0}^{b} s_i.
    const at::Tensor& v,  // total_k x num_heads_k x head_size, total_k := \sum_{i=0}^{b} s_i.
    const at::Tensor& block_count,
    const at::Tensor& block_offset,
    const at::Tensor& column_count,
    const at::Tensor& column_index,
    const c10::optional<at::Tensor>& out_,  // total_q x num_heads x head_size, total_k := \sum_{i=0}^{b} s_i
    const at::Tensor& cu_seqlens_q,         // b+1
    const at::Tensor& cu_seqlens_k,         // b+1
    const c10::optional<at::Tensor>&
        seqused_k,  // b. If given, only this many elements of each batch element's keys are used.
    const c10::optional<at::Tensor>& alibi_slopes_,  // num_heads or b x num_heads
```
**EN:** This section uses the surrounding logic to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过相关逻辑等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 635-659: Runtime integration and dispatch
```cpp
    int64_t max_seqlen_q,
    const int64_t max_seqlen_k,
    const double p_dropout,
    const double softmax_scale,
    const bool zero_tensors,
    bool is_causal,
    const double softcap,
    const bool return_softmax,
    c10::optional<at::Generator> gen_);
}  // namespace flash

void convert_vertical_slash_indexes(
    torch::Tensor& block_count,      // [BATCH, N_HEADS, NUM_ROWS]
    torch::Tensor& block_offset,     // [BATCH, N_HEADS, NUM_ROWS, NNZ_S]
    torch::Tensor& column_count,     // [BATCH, N_HEADS, NUM_ROWS]
    torch::Tensor& column_index,     // [BATCH, N_HEADS, NUM_ROWS, NNZ_V]
    torch::Tensor q_seqlens,         // [BATCH, ]
    torch::Tensor kv_seqlens,        // [BATCH, ]
    torch::Tensor vertical_indexes,  // [BATCH, N_HEADS, NNZ_V]
    torch::Tensor slash_indexes,     // [BATCH, N_HEADS, NNZ_S]
    int64_t context_size,
    int64_t block_size_M,
    int64_t block_size_N,
    bool causal);
```
**EN:** This section uses `convert_vertical_slash_indexes` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`convert_vertical_slash_indexes`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 660-692: Runtime integration and dispatch
```cpp
void convert_vertical_slash_indexes_mergehead(
    torch::Tensor& block_count,            // [BATCH, N_HEADS, NUM_ROWS]
    torch::Tensor& block_offset,           // [BATCH, N_HEADS, NUM_ROWS, NNZ_S]
    torch::Tensor& column_count,           // [BATCH, N_HEADS, NUM_ROWS]
    torch::Tensor& column_index,           // [BATCH, N_HEADS, NUM_ROWS, NNZ_V]
    torch::Tensor q_seqlens,               // [BATCH, ]
    torch::Tensor kv_seqlens,              // [BATCH, ]
    torch::Tensor vertical_indexes,        // [BATCH, N_HEADS, NNZ_V]
    torch::Tensor slash_indexes,           // [BATCH, N_HEADS, NNZ_S]
    torch::Tensor vertical_indices_count,  // [N_HEADS, ]
    torch::Tensor slash_indices_count,
    int64_t context_size,
    int64_t block_size_M,
    int64_t block_size_N,
    bool causal);

/*
 * From csrc/grammar
 */
void ApplyTokenBitmaskInplace(at::Tensor logits, at::Tensor bitmask, at::optional<at::Tensor> indices = at::nullopt);

/*
 * From csrc/gemm (QServe)
 */
void qserve_w4a8_per_chn_gemm(
    const torch::Tensor& _in_feats,
    const torch::Tensor& _kernel,
    const torch::Tensor& _wscales,
    const torch::Tensor& _ascales,
    const torch::Tensor& _w_szs,
    const torch::Tensor& _a_ssums,
    torch::Tensor& _out_feats);
```
**EN:** This section uses `convert_vertical_slash_indexes_mergehead`, `ApplyTokenBitmaskInplace`, `qserve_w4a8_per_chn_gemm` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`convert_vertical_slash_indexes_mergehead`、`ApplyTokenBitmaskInplace`、`qserve_w4a8_per_chn_gemm`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 693-722: Runtime integration and dispatch
```cpp
void qserve_w4a8_per_group_gemm(
    const torch::Tensor& _in_feats,
    const torch::Tensor& _kernel,
    const torch::Tensor& _zeros,
    const torch::Tensor& _scales_i8,
    const torch::Tensor& _wscales,
    const torch::Tensor& _ascales,
    torch::Tensor& _out_feats);

/*
 * From csrc/quantization/gguf
 */
torch::Tensor
ggml_dequantize(torch::Tensor W, int64_t type, int64_t m, int64_t n, std::optional<at::ScalarType> const& dtype);

torch::Tensor ggml_mul_mat_vec_a8(torch::Tensor W, torch::Tensor X, int64_t type, int64_t row);

torch::Tensor ggml_mul_mat_a8(torch::Tensor W, torch::Tensor X, int64_t type, int64_t row);

torch::Tensor ggml_moe_a8(
    torch::Tensor X,
    torch::Tensor W,
    torch::Tensor sorted_token_ids,
    torch::Tensor expert_ids,
    torch::Tensor num_tokens_post_padded,
    int64_t type,
    int64_t row,
    int64_t top_k,
    int64_t tokens);
```
**EN:** This section uses `qserve_w4a8_per_group_gemm`, `ggml_dequantize`, `ggml_mul_mat_vec_a8` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`qserve_w4a8_per_group_gemm`、`ggml_dequantize`、`ggml_mul_mat_vec_a8`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 723-745: Runtime integration and dispatch
```cpp
torch::Tensor ggml_moe_a8_vec(
    torch::Tensor X, torch::Tensor W, torch::Tensor topk_ids, int64_t top_k, int64_t type, int64_t row, int64_t tokens);

int64_t ggml_moe_get_block_size(int64_t type);

/*
 * From csrc/spatial
 */
std::vector<int64_t> create_greenctx_stream_by_value(int64_t smA, int64_t smB, int64_t device);

/*
 * From csrc/mamba
 */
void causal_conv1d_update(
    const at::Tensor& x,
    const at::Tensor& conv_state,
    const at::Tensor& weight,
    const std::optional<at::Tensor>& bias_,
    bool silu_activation,
    const std::optional<at::Tensor>& cache_seqlens_,
    const std::optional<at::Tensor>& conv_state_indices_,
    int64_t pad_slot_id);
```
**EN:** This section uses `ggml_moe_a8_vec`, `ggml_moe_get_block_size`, `create_greenctx_stream_by_value` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`ggml_moe_a8_vec`、`ggml_moe_get_block_size`、`create_greenctx_stream_by_value`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 746-772: Runtime integration and dispatch
```cpp
void causal_conv1d_fwd(
    const at::Tensor& x,
    const at::Tensor& weight,
    const std::optional<at::Tensor>& bias_,
    const std::optional<at::Tensor>& conv_states,
    const std::optional<at::Tensor>& query_start_loc,
    const std::optional<at::Tensor>& cache_indices,
    const std::optional<at::Tensor>& has_initial_state,
    bool silu_activation,
    int64_t pad_slot_id);

/*
 * From csrc/expert_specialization
 */
void es_fp8_blockwise_scaled_grouped_mm(
    torch::Tensor& output,
    const torch::Tensor& a,
    const torch::Tensor& b,
    const torch::Tensor& scales_a,
    const torch::Tensor& scales_b,
    const torch::Tensor& stride_a,
    const torch::Tensor& stride_b,
    const torch::Tensor& stride_d,
    const torch::Tensor& problem_sizes,
    const torch::Tensor& expert_offsets,
    const torch::Tensor& workspace);
```
**EN:** This section uses `causal_conv1d_fwd`, `es_fp8_blockwise_scaled_grouped_mm` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`causal_conv1d_fwd`、`es_fp8_blockwise_scaled_grouped_mm`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 773-801: Runtime integration and dispatch
```cpp
void es_sm100_mxfp8_blockscaled_grouped_mm(
    const torch::Tensor& a,
    const torch::Tensor& b,
    const torch::Tensor& sfa,
    const torch::Tensor& sfb,
    torch::Tensor& d,
    const torch::Tensor& problem_sizes,
    const torch::Tensor& expert_offsets,
    const torch::Tensor& blockscale_offsets);

void es_sm100_mxfp8_blockscaled_grouped_quant(
    const torch::Tensor& input,
    const torch::Tensor& problem_sizes,
    const torch::Tensor& expert_offsets,
    const torch::Tensor& blockscale_offsets,
    torch::Tensor& quant_output,
    torch::Tensor& scale_factor);

/*
 * From flashmla
 */
std::vector<at::Tensor> get_mla_decoding_metadata(
    at::Tensor& seqlens_k,
    const int64_t num_q_tokens_per_head_k,
    const int64_t h_k,
    const std::optional<int64_t> h_q,
    const bool is_fp8_kvcache,
    const std::optional<int64_t> topk);
```
**EN:** This section uses `es_sm100_mxfp8_blockscaled_grouped_mm`, `es_sm100_mxfp8_blockscaled_grouped_quant`, `get_mla_decoding_metadata` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`es_sm100_mxfp8_blockscaled_grouped_mm`、`es_sm100_mxfp8_blockscaled_grouped_quant`、`get_mla_decoding_metadata`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 802-831: Runtime integration and dispatch
```cpp
std::vector<at::Tensor> fwd_kvcache_mla(
    at::Tensor& q,             // batch_size x seqlen_q x num_heads x head_size
    const at::Tensor& kcache,  // num_blocks x page_block_size x num_heads_k x head_size (when is_fp8 is False) or
                               // num_blocks x num_heads_k x (page_block_size*656) (when is_fp8 is True)
    const int64_t head_size_v,
    const at::Tensor& seqlens_k,    // batch_size
    const at::Tensor& block_table,  // batch_size x max_num_blocks_per_seq
    const double softmax_scale,
    bool is_causal,
    const at::Tensor& tile_scheduler_metadata,  // num_sm_parts x TileSchedulerMetaDataSize
    const at::Tensor& num_splits,               // batch_size + 1
    const bool& is_fp8,
    const std::optional<at::Tensor>& indices  // None, or batch_size x seqlen_q x topk
);

void FMHACutlassSM100FwdRun(
    at::Tensor workspace_buffer,
    at::Tensor q,
    at::Tensor k,
    at::Tensor v,
    at::Tensor cumulative_seqlen_q,
    at::Tensor cumulative_seqlen_kv,
    at::Tensor o,
    at::Tensor lse,
    int64_t mask_mode_code,
    double softmax_scale,
    int64_t max_seqlen_q,
    int64_t max_seqlen_kv,
    bool is_varlen);
```
**EN:** This section uses `fwd_kvcache_mla`, `FMHACutlassSM100FwdRun` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`fwd_kvcache_mla`、`FMHACutlassSM100FwdRun`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 832-849: Runtime integration and dispatch
```cpp
std::vector<at::Tensor>
sparse_prefill_fwd(const at::Tensor& q, const at::Tensor& kv, const at::Tensor& indices, double sm_scale, int64_t d_v);

std::vector<at::Tensor> fwd_kvcache_mla_fp8(
    at::Tensor& q,             // batch_size x seqlen_q x num_heads x head_size
    const at::Tensor& kcache,  // num_blocks x page_block_size x num_heads_k x head_size (when is_fp8 is False) or
                               // num_blocks x num_heads_k x (page_block_size*656) (when is_fp8 is True)
    const int64_t head_size_v,
    const at::Tensor& seqlens_k,    // batch_size
    const at::Tensor& block_table,  // batch_size x max_num_blocks_per_seq
    const double softmax_scale,
    bool is_causal,
    const at::Tensor& tile_scheduler_metadata,   // num_sm_parts x TileSchedulerMetaDataSize
    const at::Tensor& num_splits,                // batch_size + 1
    const std::optional<at::Tensor>& descale_q,  // None or batch_size
    const std::optional<at::Tensor>& descale_k   // None or batch_size
);
```
**EN:** This section uses `sparse_prefill_fwd`, `fwd_kvcache_mla_fp8` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`sparse_prefill_fwd`、`fwd_kvcache_mla_fp8`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 850-851: Runtime integration and dispatch
```cpp
std::vector<at::Tensor> get_mla_decoding_metadata_dense_fp8(
    at::Tensor& seqlens_k, const int64_t num_heads_per_head_k, const int64_t num_heads_k);
```
**EN:** This section uses `get_mla_decoding_metadata_dense_fp8` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`get_mla_decoding_metadata_dense_fp8`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

## Key Concepts / 关键概念
- **PyTorch extension / PyTorch 扩展**: Bridges low-level kernels with PyTorch tensor/runtime abstractions. / 把底层内核与 PyTorch 张量/运行时抽象连接起来。
- **CUDA runtime / CUDA 运行时**: Uses CUDA APIs, streams, or intrinsics for NVIDIA GPUs. / 使用 CUDA API、stream 或 intrinsic 支持 NVIDIA GPU。
- **MUSA portability / MUSA 可移植性**: Contains conditional logic for the MUSA toolchain or runtime. / 包含面向 MUSA 工具链或运行时的条件分支。
- **CUTLASS integration / CUTLASS 集成**: Builds on CUTLASS/CUTE abstractions for optimized kernel generation. / 基于 CUTLASS/CUTE 抽象构建优化内核。
- **Quantization / 量化**: Handles low-precision representations and conversion logic. / 处理低精度表示及其转换逻辑。
- **Collective communication / 集合通信**: Coordinates reductions across devices or ranks. / 在设备或 rank 之间协调归约。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `scalar_type.hpp`, `sgl_kernel_musa_ops.h`
- **External headers / 外部头文件**: `ATen/ATen.h`, `ATen/Tensor.h`, `Python.h`, `torch/all.h`, `torch/library.h`, `torch/torch.h`, `tuple`, `vector`
- **Path context / 路径上下文**: include / sgl_kernel_ops.h
