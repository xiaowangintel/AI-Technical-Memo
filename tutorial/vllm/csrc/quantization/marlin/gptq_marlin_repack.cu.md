# gptq_marlin_repack.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/quantization/marlin/gptq_marlin_repack.cu`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Repack GPTQ quantized weights, optionally applying a permutation for act-order, into Marlin layout. / [CN] 将 GPTQ 量化权重（可选 act-order 排列）重排成 Marlin 所需布局。

## Line-by-Line Analysis / 逐行分析
### Kernel setup with optional permutation cache
```cpp
template <int const num_threads, int const num_bits, bool const has_perm,
          bool is_a_8bit>
__global__ void gptq_marlin_repack_kernel(...)
{
  constexpr int perm_size = target_tile_k_size / 4;
  int4* sh_perm_ptr = sh;
  int4* sh_pipe_ptr = sh_perm_ptr;
  if constexpr (has_perm) {
    sh_pipe_ptr += perm_size;
  }
}
```
**EN:** Unlike AWQ, GPTQ may need a K-dimension permutation (`act_order`). When `has_perm` is true, the kernel reserves shared memory for the permutation so every tile can cheaply remap source rows.
**CN:** 与 AWQ 不同，GPTQ 可能需要 K 维排列（`act_order`）。当 `has_perm` 为真时，内核会额外在共享内存中缓存排列信息，从而让每个 tile 都能低成本地重映射源行。

### Loading permutation and source tiles
```cpp
auto load_perm_to_shared = [&](int k_tile_id) {
  int first_k_int4 = (k_tile_id * target_tile_k_size) / 4;
  if (threadIdx.x < perm_size) {
    sh_perm_ptr[threadIdx.x] = perm_int4_ptr[first_k_int4 + threadIdx.x];
  }
};
```
**EN:** The permutation array is staged once per K tile. This avoids repeatedly reading `perm` from global memory during the hot repack loop.
**CN:** 排列数组在每个 K tile 上只会搬运一次。这样就避免了在热点重排循环中反复从全局内存读取 `perm`。

### Two fetch modes: direct or permuted
```cpp
if constexpr (has_perm) {
  uint32_t const* sh_perm_int_ptr = reinterpret_cast<uint32_t const*>(sh_perm_ptr);
  int src_k = sh_perm_int_ptr[k_id];
  int src_k_packed = src_k / pack_factor;
  cp_async4(&sh_ptr[k_id * stage_n_threads + n_id],
            reinterpret_cast<int4 const*>(&(b_q_weight_ptr[src_k_packed * size_n +
                                                           first_n + (n_id * 4)])));
} else {
  cp_async4(&sh_ptr[k_id * stage_n_threads + n_id],
            reinterpret_cast<int4 const*>(&(b_q_weight_ptr[(first_k_packed + k_id) * size_n +
                                                           first_n + (n_id * 4)])));
}
```
**EN:** The fetch path changes meaningfully depending on `has_perm`: without permutation it streams the tensor as-is; with permutation it first maps the logical K row to a physically different packed row.
**CN:** 取数路径会随着 `has_perm` 发生实质变化：无排列时直接顺序读取；有排列时则先把逻辑 K 行映射到另一个物理打包行，再进行加载。

### Repacking logic and permutation-aware unpacking
```cpp
if constexpr (has_perm) {
  uint32_t src_k = sh_perm_int_ptr[k_idx];
  uint32_t src_k_pos = src_k % pack_factor;
  uint32_t b1_val = sh_stage_int_ptr[k_idx * sh_stride + cur_n];
  uint32_t b1_cur_val = (b1_val >> (src_k_pos * num_bits)) & mask;
}
```
**EN:** In the permuted path, the kernel does not just reorder rows globally; it also has to pick the correct subword position inside each packed integer. That is why it tracks both `src_k` and `src_k_pos`.
**CN:** 在带排列的路径里，内核不只是全局重排行，还必须从每个打包整数里选择正确的子字位置。因此这里既要跟踪 `src_k`，也要跟踪 `src_k_pos`。

### Host wrapper and supported combinations
```cpp
bool has_perm = perm.size(0) != 0;
...
CALL_IF(4, false, false)
CALL_IF(4, true, false)
CALL_IF(8, false, false)
CALL_IF(8, true, false)
CALL_IF(4, false, true)
CALL_IF(8, false, true)
```
**EN:** The host function accepts an empty `perm` tensor to represent the non-act-order case. Note that `has_perm && is_a_8bit` is intentionally unsupported here, which matches the kernel’s `static_assert(!is_a_8bit)` in the permutation path.
**CN:** 主机函数允许用空 `perm` 张量表示非 act-order 情况。注意这里刻意不支持 `has_perm && is_a_8bit`，这也与内核排列路径中的 `static_assert(!is_a_8bit)` 保持一致。

## Key Concepts / 关键概念
- GPTQ repacking optionally includes a K permutation step. / GPTQ 重排可选地包含 K 维排列步骤。
- Packed-position tracking is necessary when permutation breaks natural order. / 当排列打乱自然顺序时，必须额外跟踪打包位位置。
- The output layout still targets the same Marlin tile format as AWQ. / 输出布局仍然面向与 AWQ 相同的 Marlin tile 格式。

## Dependencies / 依赖关系
- Depends on `marlin.cuh` for shared constants and async-copy helpers. / 依赖 `marlin.cuh` 提供共享常量和异步拷贝辅助函数。
- Registered as `gptq_marlin_repack` through Torch extension macros. / 通过 Torch 扩展宏注册为 `gptq_marlin_repack`。
