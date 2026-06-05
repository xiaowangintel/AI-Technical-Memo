# vertical_slash_index.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/attention/vertical_slash_index.cu`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Convert MInference-style vertical/slash sparse attention indexes into per-row block ranges and explicit column lists that downstream kernels can consume. / [CN] 将 MInference 风格的 vertical/slash 稀疏注意力索引转换为下游内核可消费的“每行 block 范围 + 显式列索引”表示。

## Line-by-Line Analysis / 逐行分析
### Helper for materializing block ranges / 物化 block 范围的辅助函数
```cpp
__device__ int64_t save_blocks(int* block_offset, int64_t range_start,
                               int64_t range_end, int64_t block_size,
                               int64_t input_block_count, int64_t kv_seqlen) {
  if (range_start >= kv_seqlen) {
    return input_block_count;
  }
  if (range_end > kv_seqlen) {
    range_end = kv_seqlen;
  }
  int64_t current_block_count = input_block_count;
  for (int idx = range_start; idx < range_end; idx += block_size) {
    block_offset[current_block_count++] = idx;
  }
  return current_block_count;
}
```
**EN:** `save_blocks` turns a contiguous token range into a list of block starting offsets, clipped to the actual KV sequence length. It is the bridge between slash-derived windows (ranges) and the blockwise sparse representation emitted by this file.
**CN:** `save_blocks` 会把一个连续 token 区间转换成 block 起始偏移列表，并裁剪到真实的 KV 序列长度范围内。它是 slash 派生“区间”和本文件输出的 block 稀疏表示之间的桥梁。

### Per-row kernel indexing / 每行 kernel 索引布局
```cpp
__global__ void convert_vertical_slash_indexes_kernel(
    const int* q_seqlens,
    const int* kv_seqlens,
    const int* vertical_indexes,
    const int* slash_indexes,
    int* block_count,
    int* block_offset,
    int* column_count,
    int* column_index,
    int64_t N_HEADS, int64_t N_ROWS, int64_t BLOCK_SIZE_M, int64_t BLOCK_SIZE_N,
    int64_t NNZ_V, int64_t NNZ_S,
    bool causal) {
  const int batch_idx = blockIdx.y;
  const int head_idx = blockIdx.x;
  const int group_idx = blockIdx.z;

  int64_t q_seqlen = q_seqlens[batch_idx];
  int64_t kv_seqlen = kv_seqlens[batch_idx];
  int64_t block_idx_m = group_idx * blockDim.x + threadIdx.x;
  int64_t start_m = block_idx_m * BLOCK_SIZE_M;
  if (start_m >= q_seqlen) {
    return;
  }
  int64_t end_m = start_m + BLOCK_SIZE_M;
  ...
  int64_t row_offset = (batch_idx * N_HEADS + head_idx) * N_ROWS + block_idx_m;
  block_count += row_offset;
  block_offset += row_offset * NNZ_S;
  column_count += row_offset;
  column_index += row_offset * NNZ_V;
}
```
**EN:** Each thread corresponds to one query block row for a specific `(batch, head)` pair. The kernel first checks whether that row is inside the actual query length, then re-bases all output pointers so it can write the row’s block count, block offsets, column count, and column indices in-place.
**CN:** 每个线程对应某个 `(batch, head)` 下的一行 query block。内核会先检查该行是否落在真实 query 长度之内，然后把所有输出指针重定位到这一行对应的位置，以便原地写入 block 数量、block 偏移、列数量和列索引。

### Initial slash window construction / 初始 slash 窗口构造
```cpp
bool has_slash = true;
int64_t tmp_col_cnt = 0, tmp_blk_cnt = 0;
int64_t s = 0, v = 0;
int64_t v_idx = vertical_indexes[v++];
int64_t s_idx = slash_indexes[s++];
if (causal) {
  while (s_idx >= end_m + (kv_seqlen - q_seqlen) && s < NNZ_S) {
    s_idx = slash_indexes[s++];
  }
  if (s_idx > end_m + (kv_seqlen - q_seqlen)) has_slash = false;
  s_idx = max((kv_seqlen - q_seqlen) + end_m - s_idx, BLOCK_SIZE_M);
} else {
  while (s_idx >= end_m + kv_seqlen && s < NNZ_S) {
    s_idx = slash_indexes[s++];
  }
  if (s_idx > end_m + kv_seqlen) has_slash = false;
  s_idx = max(kv_seqlen + end_m - s_idx, BLOCK_SIZE_M);
}

int64_t range_start = s_idx - BLOCK_SIZE_M, range_end = s_idx;
```
**EN:** The kernel interprets slash indices as diagonally defined regions and converts the first valid slash entry into a concrete `[range_start, range_end)` interval. The formula differs between causal and successor (`succ`) modes because causal attention aligns query and KV coordinates with a `kv_seqlen - q_seqlen` shift.
**CN:** 内核把 slash 索引看作沿对角线定义的区域，并把首个有效 slash 条目转换成具体的 `[range_start, range_end)` 区间。由于因果模式需要用 `kv_seqlen - q_seqlen` 进行坐标对齐，因此 causal 与 successor（`succ`）模式下的公式不同。

### Merging vertical points and slash ranges / 合并 vertical 点与 slash 区间
```cpp
bool slash_finished = false;
while (1) {
  if (v_idx < range_end) {
    if (v_idx < range_start) {
      column_index[tmp_col_cnt++] = v_idx;
    }
    if (v < NNZ_V) {
      v_idx = vertical_indexes[v++];
    } else {
      if (causal)
        v_idx = end_m + BLOCK_SIZE_N + (kv_seqlen - q_seqlen);
      else
        v_idx = end_m + BLOCK_SIZE_N + kv_seqlen;
    }
  } else {
    if ((s < NNZ_S && causal) ||
        (s < NNZ_S && !causal && slash_indexes[s] >= start_m)) {
      ...
    } else {
      if (v == NNZ_V || (v_idx > range_start && causal)) {
        if (v == NNZ_V && !causal && v_idx < kv_seqlen) {
          column_index[tmp_col_cnt++] = v_idx;
        }
        tmp_blk_cnt = save_blocks(block_offset, range_start, range_end,
                                  BLOCK_SIZE_N, tmp_blk_cnt, kv_seqlen);
        break;
      } else {
        ...
        slash_finished = true;
      }
    }
    if (!slash_finished) {
      if (s_idx > range_end + BLOCK_SIZE_M) {
        tmp_blk_cnt = save_blocks(block_offset, range_start, range_end,
                                  BLOCK_SIZE_N, tmp_blk_cnt, kv_seqlen);
        range_start = s_idx - BLOCK_SIZE_M;
        range_end = s_idx;
      } else if (s_idx > range_end) {
        range_end += BLOCK_SIZE_M;
      }
    }
  }
}
```
**EN:** This loop is the core of Algorithm 4: vertical indices that fall before the active slash range become explicit column entries, while slash-derived intervals are merged into block ranges. When a later slash is adjacent, the range expands; when it is separated, the current range is flushed via `save_blocks` and a new range begins.
**CN:** 这段循环就是论文 Algorithm 4 的核心：落在当前 slash 区间之前的 vertical 索引会被保留下来，作为显式列；由 slash 推导出的连续区间则会被合并成 block 范围。若后续 slash 与当前区间相邻，就扩展区间；若两者分离，则先通过 `save_blocks` 刷出当前区间，再开始新的区间。

### Standard wrapper exported to PyTorch / 导出给 PyTorch 的标准封装
```cpp
void convert_vertical_slash_indexes(
    torch::Tensor& block_count,
    torch::Tensor& block_offset,
    torch::Tensor& column_count,
    torch::Tensor& column_index,
    torch::Tensor q_seqlens,
    torch::Tensor kv_seqlens,
    torch::Tensor vertical_indexes,
    torch::Tensor slash_indexes,
    int64_t context_size, int64_t block_size_M, int64_t block_size_N,
    bool causal) {
  cudaSetDevice(q_seqlens.get_device());

  int batch_size = slash_indexes.size(0);
  int num_heads = slash_indexes.size(1);
  int nnz_slash = slash_indexes.size(2);
  int nnz_vertical = vertical_indexes.size(2);
  int num_rows = (context_size + block_size_M - 1) / block_size_M;

  convert_vertical_slash_indexes_64x64(
      q_seqlens.data_ptr<int>(), kv_seqlens.data_ptr<int>(),
      vertical_indexes.data_ptr<int>(), slash_indexes.data_ptr<int>(),
      block_count.data_ptr<int>(), block_offset.data_ptr<int>(),
      column_count.data_ptr<int>(), column_index.data_ptr<int>(), batch_size,
      num_heads, num_rows, block_size_M, block_size_N, nnz_vertical, nnz_slash,
      causal);
}
```
**EN:** The exported wrapper is mostly shape plumbing. It selects the current CUDA device from `q_seqlens`, derives sizes from tensor shapes, computes the number of row blocks from `context_size`, and launches the fixed `64x64` configuration helper.
**CN:** 这个导出封装主要负责形状与设备层面的整理。它根据 `q_seqlens` 选择当前 CUDA 设备，从张量形状中提取尺寸，再依据 `context_size` 计算 query 方向的 block 行数，最后调用固定的 `64x64` 启动辅助函数。

### Merge-head variant with per-head top-k counts / 带每个 head 独立 top-k 计数的 merge-head 变体
```cpp
// MergeHead: each head has it's unique max topk NNZ_V，NNZ_S. (NNZ_V，NNZ_S
// above is buffer size, use to compute offset)
NNZ_S = per_head_slash_topkv[head_idx];
NNZ_V = per_head_vertical_topkv[head_idx];
```
```cpp
void convert_vertical_slash_indexes_mergehead(
    torch::Tensor& block_count,
    torch::Tensor& block_offset,
    torch::Tensor& column_count,
    torch::Tensor& column_index,
    torch::Tensor q_seqlens,
    torch::Tensor kv_seqlens,
    torch::Tensor vertical_indexes,
    torch::Tensor slash_indexes,
    torch::Tensor vertical_indices_count,
    torch::Tensor slash_indices_count,
    int64_t context_size, int64_t block_size_M, int64_t block_size_N,
    bool causal) {
  ...
  convert_vertical_slash_indexes_64x64_mergehead(
      q_seqlens.data_ptr<int>(), kv_seqlens.data_ptr<int>(),
      vertical_indexes.data_ptr<int>(), slash_indexes.data_ptr<int>(),
      vertical_indices_count.data_ptr<int>(),
      slash_indices_count.data_ptr<int>(), block_count.data_ptr<int>(),
      block_offset.data_ptr<int>(), column_count.data_ptr<int>(),
      column_index.data_ptr<int>(), batch_size, num_heads, num_rows,
      block_size_M, block_size_N, nnz_vertical, nnz_slash, causal);
}
```
**EN:** The merge-head path reuses almost the same algorithm but overrides `NNZ_V` and `NNZ_S` per head using precomputed top-k counts. That lets callers store a padded buffer for all heads while still iterating only over the valid entries for each individual head.
**CN:** merge-head 路径基本复用了同一套算法，但会利用预先计算好的 top-k 计数，为每个 head 单独覆盖 `NNZ_V` 和 `NNZ_S`。这样调用方既可以为所有 head 分配统一的填充缓冲区，又能在计算时只遍历各自真实有效的条目数。

## Key Concepts / 关键概念
- **EN:** Vertical indices are treated as individual columns; slash indices are transformed into contiguous token ranges and then into block offsets.  
  **CN:** vertical 索引被视为单独列；slash 索引则先转成连续 token 区间，再转成 block 偏移。
- **EN:** The output is a hybrid sparse format: explicit columns plus coalesced block ranges.  
  **CN:** 输出是一种混合稀疏格式：既有显式列，也有合并后的 block 区间。
- **EN:** Causal and non-causal modes differ mainly in how query rows are aligned with KV coordinates.  
  **CN:** causal 与非 causal 模式的主要区别在于 query 行如何与 KV 坐标对齐。
- **EN:** The merge-head variant keeps a common buffer layout but applies per-head valid counts.  
  **CN:** merge-head 变体保持统一的缓冲区布局，同时对每个 head 使用各自的有效计数。
- **EN:** This file is an indexing/preprocessing kernel; it does not perform attention score computation itself.  
  **CN:** 这个文件属于索引/预处理内核，本身并不直接计算 attention 分数。

## Dependencies / 依赖关系
- **EN:** `torch/all.h` is used for PyTorch tensor interop on the C++ side.  
  **CN:** `torch/all.h` 用于 C++ 侧与 PyTorch 张量交互。
- **EN:** `cuda.h` and raw CUDA kernels are used directly; the file does not rely on higher-level ATen launch helpers.  
  **CN:** 文件直接使用 `cuda.h` 和原生 CUDA kernel，没有依赖更高层的 ATen 启动辅助。
- **EN:** The algorithm references MInference paper Algorithm 4 and notes the original Microsoft implementation as a source.  
  **CN:** 该算法引用了 MInference 论文中的 Algorithm 4，并注明 Microsoft 原始实现为参考来源。
- **EN:** Downstream sparse-attention kernels depend on the emitted `block_offset` / `column_index` representation.  
  **CN:** 下游稀疏 attention 内核会依赖这里输出的 `block_offset` / `column_index` 表示。
