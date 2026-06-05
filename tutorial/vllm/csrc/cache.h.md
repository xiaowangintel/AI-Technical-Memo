# cache.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/cache.h`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Declares the public C++/CUDA extension entry points for moving, reshaping, quantizing, gathering, and concatenating KV-cache data. / [CN] 声明用于 KV cache 搬运、重排、量化、收集与拼接的公共 C++/CUDA 扩展入口。

## Line-by-Line Analysis / 逐行分析
### Block migration APIs / 块迁移接口
```cpp
void swap_blocks(torch::Tensor& src, torch::Tensor& dst,
                 int64_t block_size_in_bytes,
                 const torch::Tensor& block_mapping);

void swap_blocks_batch(const torch::Tensor& src_ptrs,
                       const torch::Tensor& dst_ptrs,
                       const torch::Tensor& sizes,
                       bool is_src_access_order_any);
```
**EN:** These declarations expose both per-block copying (`swap_blocks`) and batched raw-pointer copy submission (`swap_blocks_batch`). They are used when cache pages move between devices or memory pools.
**CN:** 这些声明同时暴露了按 block 复制（`swap_blocks`）和按原始指针批量提交拷贝（`swap_blocks_batch`）两类接口，用于 cache 页面在设备或内存池之间迁移。

### Reshape and MLA cache writers / 重排与 MLA cache 写入接口
```cpp
void reshape_and_cache(torch::Tensor& key, torch::Tensor& value,
                       torch::Tensor& key_cache, torch::Tensor& value_cache,
                       torch::Tensor& slot_mapping,
                       const std::string& kv_cache_dtype,
                       torch::Tensor& k_scale, torch::Tensor& v_scale);

void reshape_and_cache_flash(torch::Tensor& key, torch::Tensor& value,
                             torch::Tensor& key_cache,
                             torch::Tensor& value_cache,
                             torch::Tensor& slot_mapping,
                             const std::string& kv_cache_dtype,
                             torch::Tensor& k_scale, torch::Tensor& v_scale);

void concat_and_cache_mla(torch::Tensor& kv_c, torch::Tensor& k_pe,
                          torch::Tensor& kv_cache, torch::Tensor& slot_mapping,
                          const std::string& kv_cache_dtype,
                          torch::Tensor& scale);

// NOTE: k_pe and kv_c order is flipped compared to concat_and_cache_mla
void concat_and_cache_mla_rope_fused(
    torch::Tensor& positions, torch::Tensor& q_pe, torch::Tensor& k_pe,
    torch::Tensor& kv_c, torch::Tensor& rope_cos_sin_cache, bool rope_is_neox,
    torch::Tensor& kv_cache_slot_mapping, torch::Tensor& kv_cache,
    const std::string& kv_cache_dtype, torch::Tensor& kv_cache_quant_scale);

// Just for unittest
void convert_fp8(torch::Tensor& dst_cache, torch::Tensor& src_cache,
                 const double scale, const std::string& kv_cache_dtype);
```
**EN:** This group covers the main write-side paths: classic key/value reshape, flash-attention-friendly cache layout, MLA concatenation, and a fused variant that applies RoPE before writing cache entries.
**CN:** 这一组覆盖主要的写入路径：传统 key/value 重排、适配 flash-attention 的 cache 布局、MLA 拼接，以及先做 RoPE 再写入 cache 的融合版本。

### Gather, dequantize, and indexer-quant APIs / 收集、反量化与索引量化接口
```cpp
void gather_and_maybe_dequant_cache(
    torch::Tensor const& src_cache,     // [NUM_BLOCKS, BLOCK_SIZE, ENTRIES...]
    torch::Tensor const& dst,           // [TOT_TOKENS, ENTRIES...]
    torch::Tensor const& block_table,   // [BATCH, BLOCK_INDICES]
    torch::Tensor const& cu_seq_lens,   // [BATCH+1]
    torch::Tensor const& token_to_seq,  // [MAX_TOKEN_ACROSS_CHUNKS]
    int64_t num_tokens, const std::string& kv_cache_dtype,
    torch::Tensor const& scale,
    std::optional<torch::Tensor> seq_starts = std::nullopt);

// TODO(hc): cp_gather_cache need support scaled kvcahe in the future.
void cp_gather_cache(
    torch::Tensor const& src_cache,    // [NUM_BLOCKS, BLOCK_SIZE, ENTRIES...]
    torch::Tensor const& dst,          // [TOT_TOKENS, ENTRIES...]
    torch::Tensor const& block_table,  // [BATCH, BLOCK_INDICES]
    torch::Tensor const& cu_seq_lens,  // [BATCH+1]
    int64_t batch_size, std::optional<torch::Tensor> seq_starts = std::nullopt);

// Gather and upconvert FP8 KV cache to BF16 workspace
void cp_gather_and_upconvert_fp8_kv_cache(
    torch::Tensor const& src_cache,         // [NUM_BLOCKS, BLOCK_SIZE, 656]
    torch::Tensor const& dst,               // [TOT_TOKENS, 576]
    torch::Tensor const& block_table,       // [BATCH, BLOCK_INDICES]
    torch::Tensor const& seq_lens,          // [BATCH]
    torch::Tensor const& workspace_starts,  // [BATCH]
    int64_t batch_size);

// Indexer K quantization and cache function
void indexer_k_quant_and_cache(
    torch::Tensor& k,             // [num_tokens, head_dim]
    torch::Tensor& kv_cache,      // [num_blocks, block_size, cache_stride]
    torch::Tensor& slot_mapping,  // [num_tokens]
    int64_t quant_block_size,     // quantization block size
    const std::string& scale_fmt);

// Concatenate query nope and rope for MLA/DSA attention
void concat_mla_q(
    torch::Tensor& ql_nope,  // [num_tokens, num_heads, nope_dim]
    torch::Tensor& q_pe,     // [num_tokens, num_heads, rope_dim]
    torch::Tensor& q_out);   // [num_tokens, num_heads, nope_dim + rope_dim]

// Extract function to gather quantized K cache
void cp_gather_indexer_k_quant_cache(
    const torch::Tensor& kv_cache,  // [num_blocks, block_size, cache_stride]
    torch::Tensor& dst_k,           // [num_tokens, head_dim]
    torch::Tensor& dst_scale,  // [num_tokens, head_dim / quant_block_size * 4]
    const torch::Tensor& block_table,   // [batch_size, num_blocks]
    const torch::Tensor& cu_seq_lens);  // [batch_size + 1]
```
**EN:** The remaining declarations are read-side and quantization helpers: gather cached tokens back into dense workspaces, upconvert FP8 cache, quantize indexer K blocks, reconstruct quantized K, and concatenate MLA query components.
**CN:** 剩余声明主要是读侧和量化辅助接口：把 cache 中的 token 收集回稠密工作区、把 FP8 cache 升精度、对 indexer K block 量化、恢复量化后的 K，以及拼接 MLA query 组件。

## Key Concepts / 关键概念
- **EN:** The header is an interface contract; all heavy logic lives in `.cu` implementations.
  **CN:** 这个头文件只是接口契约，真正的重逻辑都位于 `.cu` 实现文件中。
- **EN:** Function signatures encode expected tensor layouts directly in comments, which is essential for PyTorch extension correctness.
  **CN:** 函数签名旁的注释直接编码了期望的 tensor 布局，这对 PyTorch 扩展的正确性非常关键。
- **EN:** MLA- and FP8-specific entry points show that vLLM maintains several cache formats side by side.
  **CN:** MLA 与 FP8 专用入口说明 vLLM 同时维护多种 KV cache 格式。

## Dependencies / 依赖关系
- **EN:** Depends on `torch::Tensor` and `std::optional` because every entry point is called from the PyTorch extension boundary.
  **CN:** 依赖 `torch::Tensor` 与 `std::optional`，因为所有入口都服务于 PyTorch 扩展边界。
- **EN:** Implementations are provided mainly by `cache_kernels.cu` and `cache_kernels_fused.cu`.
  **CN:** 具体实现主要位于 `cache_kernels.cu` 与 `cache_kernels_fused.cu`。
