# moe.cuh — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/quantization/gguf/moe.cuh`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Implements routed GGUF quantized matrix-matrix kernels for fused MoE experts, reusing the tiled MMQ design but adding token/expert routing. / [CN] 实现面向融合 MoE 专家的路由式 GGUF 量化矩阵乘内核，在复用 MMQ 分块设计的同时加入 token/expert 路由逻辑。

## Line-by-Line Analysis / 逐行分析
### Generic routed matrix kernel (L5-L45)
```cpp
template <typename scalar_t, int qk, int qr, int qi, bool need_sum, ...>
static __device__ __forceinline__ void moe_q(...) {
  ...
  int token_offs[mmq_x / nwarps];
  for (int i = 0; i < mmq_x; i += nwarps) {
    token_offs[i / nwarps] = sorted_token_ids[col_dst_0 + threadIdx.y + i];
  }
  const int exp_idx = expert_ids[blockIdx.y];
  if (exp_idx > 255 || exp_idx < 0) return;
  if (blockIdx.y * mmq_x > num_tokens_post_padded[0]) return;
  const block_q_t* x = (const block_q_t*)((char*)vx + exp_idx * exp_stride);
}
```
**EN:** Compared with `mmq.cuh`, this kernel adds routing metadata. `sorted_token_ids` tells the block which token slots it owns, while `expert_ids` and `exp_stride` redirect the weight pointer to the selected expert matrix.
**CN:** 与 `mmq.cuh` 相比，这个内核新增了路由元数据。`sorted_token_ids` 告诉当前 block 负责哪些 token 槽位，而 `expert_ids` 和 `exp_stride` 会把权重指针重定向到对应专家矩阵。

### Loading routed activations and accumulating tiles (L51-L110)
```cpp
for (int ib0 = 0; ib0 < blocks_per_row_x; ib0 += blocks_per_warp) {
  load_tiles(...);
  ...
  const int col_y_eff = token_offs[i / nwarps] / top_k;
  ...
  tile_y_qs[index_y] = get_int_from_int8_aligned(by0->qs, ...);
  ...
  sum[...] += vec_dot(tile_x_ql, tile_x_dm, tile_x_qh, tile_x_sc,
                      tile_y_qs, tile_y_ds, ...);
}
```
**EN:** The right-hand-side tile is not a contiguous matrix slice anymore: each column comes from a routed token, and `token_offs / top_k` maps repeated top-k entries back to the original token index. The accumulation path then matches the regular MMQ kernel.
**CN:** 这里右侧 tile 已不再是连续矩阵切片：每一列都来自一个路由后的 token，而 `token_offs / top_k` 会把重复的 top-k 条目映射回原始 token 索引。之后的累加路径则与普通 MMQ 内核保持一致。

### Output writeback in routed layout (L113-L128)
```cpp
for (int j = 0; j < mmq_x; j += nwarps) {
  const int col_dst = token_offs[j / nwarps];
  ...
  dst[col_dst * nrows_dst + row_dst] = sum[i / WARP_SIZE_GGUF][j / nwarps];
}
```
**EN:** Results are written in routed order, not plain token order. Each column index already includes the top-k expansion, so downstream MoE code can consume the expert outputs without another scatter.
**CN:** 结果是按路由顺序写回的，而不是普通 token 顺序。每个列索引已经包含了 top-k 展开后的偏移，因此下游 MoE 代码无需再做一次 scatter。

### Per-format wrappers for classic formats (L131-L434)
```cpp
template <typename scalar_t, bool need_check>
static __global__ void moe_q4_0(...) {
  moe_q<scalar_t, QK4_0, QR4_0, QI4_0, true, block_q4_0, ...,
        allocate_tiles_q4_0<mmq_y>, load_tiles_q4_0<mmq_y, nwarps, need_check>,
        VDR_Q4_0_Q8_1_MMQ, vec_dot_q4_0_q8_1_mul_mat>(...);
}
```
**EN:** `q4_0`, `q4_1`, `q5_0`, `q5_1`, and `q8_0` all reuse the same routed kernel body. The wrappers only bind the block layout, tile helpers, dot-product implementation, and whether the format needs sum correction.
**CN:** `q4_0`、`q4_1`、`q5_0`、`q5_1` 与 `q8_0` 都复用同一个路由内核主体。包装函数只负责绑定块布局、tile 辅助函数、点积实现以及是否需要求和修正。

### K-quant wrappers and launch geometry (L436-L739)
```cpp
static void ggml_moe_q4_K_q8_1_cuda(...) {
  const int block_num_x = (nrows_x + mmq_y - 1) / mmq_y;
  const int block_num_y = (tokens_post_padded) / mmq_x;
  if (nrows_x % mmq_y == 0) {
    constexpr bool need_check = false;
    moe_q4_K<scalar_t, need_check><<<block_nums, block_dims, 0, stream>>>(...);
  }
}
```
**EN:** The second half repeats the pattern for `q2_K` through `q6_K`. The MoE launch grid differs from plain GEMM because the Y dimension iterates over padded routed tokens instead of output columns.
**CN:** 后半部分对 `q2_K` 到 `q6_K` 重复同样模式。与普通 GEMM 不同，MoE 的 Y 维 launch 网格遍历的是经过 padding 的路由 token，而不是输出列。

## Key Concepts / 关键概念
- MoE routing fused into quantized GEMM / 在量化 GEMM 中融合 MoE 路由
- Expert selection by `expert_ids` and `exp_stride` / 通过 `expert_ids` 与 `exp_stride` 选择专家
- Routed writeback indexed by sorted token slots / 按排序 token 槽位写回结果
- Shared MMQ-style tiling with extra routing metadata / 在 MMQ 式分块上附加路由元数据

## Dependencies / 依赖关系
- `ggml-common.h`: quant block structs and warp/tile macros
- `vecdotq.cuh`: tile alloc/load helpers and `vec_dot_*_mul_mat`
- `gguf_kernel.cu`: entry point `ggml_moe_a8`
