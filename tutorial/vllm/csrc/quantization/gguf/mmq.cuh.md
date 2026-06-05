# mmq.cuh — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/quantization/gguf/mmq.cuh`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Implements tiled matrix-matrix CUDA/ROCm kernels that multiply GGUF-quantized weights by Q8_1 activations, then exposes one launcher per quant format. / [CN] 实现将 GGUF 量化权重与 Q8_1 激活相乘的分块矩阵乘 CUDA/ROCm 内核，并为每种量化格式提供启动包装函数。

## Line-by-Line Analysis / 逐行分析
### Generic tiled kernel (L2-L33)
```cpp
template <typename scalar_t, int qk, int qr, int qi, bool need_sum, ...>
static __device__ __forceinline__ void mul_mat_q(...) {
    const block_q_t  * x = (const block_q_t  *) vx;
    const block_q8_1 * y = (const block_q8_1 *) vy;
    ...
    allocate_tiles(&tile_x_ql, &tile_x_dm, &tile_x_qh, &tile_x_sc);
    __shared__ int   tile_y_qs[mmq_x * WARP_SIZE_GGUF];
    __shared__ half2 tile_y_ds[mmq_x * WARP_SIZE_GGUF/QI8_1];
    float sum[mmq_y/WARP_SIZE_GGUF][mmq_x/nwarps] = {{0.0f}};
}
```
**EN:** This template is the real compute core. It abstracts the quant block type, tile loaders, tile layout, and dot-product backend so one implementation can serve Q4/Q5/Q8 and K-quants.
**CN:** 这个模板是真正的计算核心。它把量化块类型、tile 装载函数、tile 布局和点积后端都参数化，因此一份实现即可覆盖 Q4/Q5/Q8 和 K 系列量化。

### Loading tiles and accumulating partial sums (L35-L84)
```cpp
for (int ib0 = 0; ib0 < blocks_per_row_x; ib0 += blocks_per_warp) {
    load_tiles(...);
    for (int ir = 0; ir < qr && ...; ++ir) {
        ...
        tile_y_qs[index_y] = get_int_from_int8_aligned(by0->qs, ...);
        ...
        if (need_sum) *dsi_dst = *dsi_src;
        else *((float *) dsi_dst) = __low2float(*dsi_src);
        __syncthreads();
        for (int k = ...; k < ...; k += vdr) {
            sum[...] += vec_dot(tile_x_ql, tile_x_dm, tile_x_qh, tile_x_sc,
                                tile_y_qs, tile_y_ds, ...);
        }
    }
}
```
**EN:** Each warp-sized chunk of the quantized weight matrix is staged into shared memory, the matching Q8_1 activation tiles are loaded, and `vec_dot` performs the format-specific inner product. `need_sum` controls whether the Q8_1 `ds.y` sum term must be preserved for asymmetric quant formats.
**CN:** 每个 warp 大小的权重量化块都会先放入共享内存，再装入对应的 Q8_1 激活 tile，然后由 `vec_dot` 执行格式相关的内积。`need_sum` 控制是否要保留 Q8_1 的 `ds.y` 求和项，这对非对称量化格式很重要。

### Writing the output tile (L88-L102)
```cpp
for (int j = 0; j < mmq_x; j += nwarps) {
    const auto col_dst = col_dst_0 + j + threadIdx.y;
    ...
    for (int i = 0; i < mmq_y; i += WARP_SIZE_GGUF) {
        const auto row_dst = row_dst_0 + threadIdx.x + i;
        dst[col_dst*nrows_dst + row_dst] = sum[i/WARP_SIZE_GGUF][j/nwarps];
    }
}
```
**EN:** After all K-dimension tiles are reduced, each thread writes one element of the output tile. Bounds checks protect the last partial tile on both row and column edges.
**CN:** 在 K 维度的所有 tile 都规约完成后，每个线程把自己的结果写回输出 tile。边界判断用于保护最后一块不完整的行列边界。

### Format-specific launch wrappers (L106-L357)
```cpp
#define MMQ_X_Q4_0 4
#define MMQ_Y_Q4_0 32
#define NWARPS_Q4_0 4

template<typename scalar_t, bool need_check> static __global__ void mul_mat_q4_0(...) {
    mul_mat_q<scalar_t, QK4_0, QR4_0, QI4_0, true, block_q4_0, ...,
              allocate_tiles_q4_0<mmq_y>, load_tiles_q4_0<mmq_y, nwarps, need_check>,
              VDR_Q4_0_Q8_1_MMQ, vec_dot_q4_0_q8_1_mul_mat>(...);
}
```
**EN:** Wrappers such as `mul_mat_q4_0`, `mul_mat_q4_1`, `mul_mat_q5_0`, `mul_mat_q5_1`, and `mul_mat_q8_0` only choose compile-time parameters: block type, VDR, tile helpers, and whether the format needs the Q8 sum correction.
**CN:** `mul_mat_q4_0`、`mul_mat_q4_1`、`mul_mat_q5_0`、`mul_mat_q5_1`、`mul_mat_q8_0` 这类包装函数主要负责选择编译期参数：块类型、VDR、tile 辅助函数，以及该格式是否需要 Q8 求和修正项。

### K-quant family and boundary-aware launch (L359-L610)
```cpp
template<typename scalar_t>
static void ggml_mul_mat_q4_K_q8_1_cuda(...) {
    const int block_num_x = (nrows_x + mmq_y - 1) / mmq_y;
    const int block_num_y = (ncols_y + mmq_x - 1) / mmq_x;
    if (nrows_x % mmq_y == 0) {
        const bool need_check = false;
        mul_mat_q4_K<scalar_t, need_check><<<block_nums, block_dims, 0, stream>>>(...);
    } else {
        const bool need_check = true;
        mul_mat_q4_K<scalar_t, need_check><<<block_nums, block_dims, 0, stream>>>(...);
    }
}
```
**EN:** The rest of the file repeats the same launch pattern for `q2_K` through `q6_K`. The main runtime choice is `need_check`, which avoids extra bounds handling on perfectly aligned matrices but enables safe loads for tail tiles.
**CN:** 文件剩余部分对 `q2_K` 到 `q6_K` 重复相同的启动模式。主要的运行时选择是 `need_check`：矩阵整齐对齐时可省去额外边界处理，尾部 tile 存在时则启用安全加载。

## Key Concepts / 关键概念
- Shared-memory tiling for quantized GEMM / 面向量化 GEMM 的共享内存分块
- Per-format template specialization without duplicating the main loop / 在不复制主循环的前提下做按格式特化
- `need_sum` for asymmetric quant formats / 为非对称量化保留 `need_sum`
- `need_check` for edge-tile safety / 用 `need_check` 处理边界 tile
- ROCm vs CUDA launch geometry / ROCm 与 CUDA 的不同启动参数

## Dependencies / 依赖关系
- `ggml-common.h`: `block_q*` structs, `QK* / QR* / QI*`, `WARP_SIZE_GGUF`
- `vecdotq.cuh`: `allocate_tiles_*`, `load_tiles_*`, `vec_dot_*`, `VDR_*`
- CUDA/ROCm intrinsics: `__syncthreads`, `half2`, launch bounds
