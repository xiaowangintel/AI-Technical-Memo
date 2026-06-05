# permute_cols.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/libtorch_stable/permute_cols.cu`
- **Repository**: vllm-project/vllm
- **Purpose**: CUDA helper that permutes the last-dimension columns of a half/bfloat16 tensor using a provided permutation index vector. / 一个 CUDA 辅助算子，按给定置换索引向量对 half/bfloat16 张量最后一维的列进行重排。

## Line-by-Line Analysis / 逐行分析
### Device kernel
```cpp
__global__ void permute_cols_kernel(int4 const* __restrict__ a_int4_ptr,
                                    int const* __restrict__ perm_int_ptr,
                                    int4* __restrict__ out_int4_ptr, int size_m,
                                    int size_k, int block_rows) {
  int start_row = block_rows * blockIdx.x;
  int finish_row = start_row + block_rows;
  ...
  auto permute_row = [&](int row) {
    int iters = size_k / default_threads;
    int rest = size_k % default_threads;
```
**EN:** The kernel processes a contiguous 2D view of the input and assigns each block a chunk of rows. Within a row, each thread maps destination column `cur_k` to source column `perm[cur_k]`.
**CN:** 该内核把输入视作连续的二维矩阵，并给每个 block 分配一段行区间。在单行内部，每个线程都会把目标列 `cur_k` 映射到源列 `perm[cur_k]`。

### Half-based column remap
```cpp
half const* a_row_half = reinterpret_cast<half const*>(a_int4_ptr + offset);
half* out_half = reinterpret_cast<half*>(out_int4_ptr + offset);
...
int src_pos = perm_int_ptr[cur_k];
out_half[cur_k] = a_row_half[src_pos];
```
**EN:** Even though the pointers are passed as `int4*` for 128-bit alignment, the actual permutation is performed elementwise on 16-bit values. This is why the public API only allows half and bfloat16 tensors.
**CN:** 虽然指针以 `int4*` 形式传入以保证 128-bit 对齐，但真正的列重排是按 16-bit 元素逐个执行的。这也是公共 API 只允许 half 和 bfloat16 张量的原因。

### Host wrapper and launch policy
```cpp
STD_TORCH_CHECK(
    A.scalar_type() == torch::headeronly::ScalarType::Half ||
        A.scalar_type() == torch::headeronly::ScalarType::BFloat16,
    "Currently only 16bit types are supported");
STD_TORCH_CHECK(A.is_contiguous(), "A must be contiguous");
STD_TORCH_CHECK(A.size(-1) % 8 == 0,
                "A columns must be a multiple of 8 (128bits)");
...
int block_rows = div_ceil(A_2d.size(0), sms);
permute_cols_kernel<<<sms, default_threads, 0, stream>>>(...);
```
**EN:** The wrapper validates dtype, contiguity, and 128-bit column alignment, flattens all leading dimensions into a 2D matrix, and launches one block per SM. The row chunk size is chosen so the full matrix is covered evenly.
**CN:** 包装器会检查数据类型、连续性以及 128-bit 列对齐要求，把所有前导维展平成二维矩阵，然后按“每个 SM 一个 block”的方式启动。`block_rows` 用来保证所有行都能被较均匀地覆盖。

## Key Concepts / 关键概念
- **Column gather / 列 gather**: Output columns are gathered from arbitrary source positions.
- **16-bit tensor restriction / 16 位张量限制**: Kernel is written around `half`-sized elements.
- **SM-count-based launch / 基于 SM 数量的启动策略**: The host uses hardware SM count to partition rows.

## Dependencies / 依赖关系
- Uses `torch_utils.h` for stream retrieval and runtime checks.
- Declared in `ops.h` and exposed through the stable operator layer.
