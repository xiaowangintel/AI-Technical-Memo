# marlin_template.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/quantization/marlin/marlin_template.h`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Implement the main Marlin GEMM CUDA kernel, including async staging, dequantization, Tensor Core matmul, reduction, and writeback. / [CN] 实现 Marlin 主 GEMM CUDA 内核，包含异步 staging、反量化、Tensor Core 矩阵乘、归约与结果写回。

## Line-by-Line Analysis / 逐行分析
### Architecture guard and helper primitives
```cpp
#if defined(__CUDA_ARCH__) && __CUDA_ARCH__ < 750
__global__ void Marlin(...) {}
#else

template <int count, vllm::ScalarTypeId type_id>
__device__ inline void ldsm(...)
...
__device__ inline void barrier_acquire(int* lock, int count) {
```
**EN:** The file starts with a stub for unsupported architectures, then defines the building blocks used by the real kernel: `ldmatrix` loaders, scale helpers, zero-point subtraction helpers, and two global barrier routines for cross-block reduction.
**CN:** 这个文件先为不支持的架构提供空壳实现，然后定义真实内核所需的基础构件：`ldmatrix` 加载器、scale 辅助函数、zero-point 减法辅助函数，以及跨线程块归约用的全局 barrier 例程。

### Kernel signature and compile-time policy selection
```cpp
template <const vllm::ScalarTypeId a_type_id,
          const vllm::ScalarTypeId b_type_id,
          const vllm::ScalarTypeId c_type_id,
          const vllm::ScalarTypeId s_type_id,
          const int threads, const int thread_m_blocks,
          const int thread_n_blocks, const int thread_k_blocks,
          const bool m_block_size_8, const int stages,
          const int group_blocks, const bool is_zp_float>
__global__ void Marlin(...)
```
**EN:** Marlin bakes almost every policy into template arguments: activation/weight/scale types, tile geometry, pipeline depth, grouped quantization shape, and zero-point representation. The early `constexpr` checks prune unsupported combinations before codegen goes deeper.
**CN:** Marlin 把几乎所有策略都编码进模板参数：激活/权重/scale 类型、tile 几何、流水深度、分组量化形状和 zero-point 表示方式。前面的 `constexpr` 检查会在代码进一步展开前就裁掉不支持的组合。

### Stripe partitioning and slice scheduling
```cpp
int global_mn_tiles = parallel * n_tiles;
int part2_mn_tiles = global_mn_tiles;
int part1_mn_iters = 0;
...
auto init_part2_slice = [&]() {
  slice_iters = iters * (blockIdx.x + 1) - (k_tiles * slice_col_par + slice_row);
```
**EN:** The kernel does not assign thread blocks to a simple 2D grid. Instead, it partitions work into “stripes” across the N and K dimensions so that all SMs stay busy while minimizing the number of expensive inter-block reductions.
**CN:** 这个内核不会把线程块简单地映射到一个 2D 网格上。相反，它会在 N 和 K 维上按“条带（stripe）”方式切分工作，以便尽量让所有 SM 都保持忙碌，同时尽量减少昂贵的跨块归约次数。

### Shared-memory layouts and register buffers
```cpp
constexpr int sh_red_size = (2 * thread_n_blocks + 1) * 16 * thread_m_blocks;
constexpr int sh_b_size = stages * b_sh_stage;
int4* sh_b = sh_new;
int4* sh_red = sh_new;
...
FragA frag_a[2][thread_m_blocks];
I4 frag_b_quant[2][b_thread_vecs];
FragC frag_c[thread_m_blocks][is_a_8bit ? 2 : 4][2];
FragS frag_s[2][4];
```
**EN:** A large part of the kernel is dedicated to carefully packing shared memory and registers. `A`, `B`, scales, zero-points, reduction scratch, bias, and act-order metadata all share one dynamic shared-memory region, while register arrays implement double buffering.
**CN:** 这个内核有很大一部分代码都在精细安排共享内存和寄存器。`A`、`B`、scale、zero-point、归约暂存、bias 和 act-order 元数据都复用同一块动态共享内存，而寄存器数组则负责双缓冲。

### Async fetch pipeline and act-order scale caching
```cpp
auto fetch_to_shared = [&](int pipe, int a_off, bool pred = true) {
  cp_async4_pred(&sh_a_stage[a_sh_wr_trans[i]],
                 &A[a_gl_rd_delta_i * i + a_gl_rd + a_gl_rd_delta_o * a_off],
                 a_sh_wr_pred[i]);
  cp_async4(&sh_b_stage[threads * i + threadIdx.x], &B[b_gl_idx]);
```
**EN:** `fetch_to_shared` is the producer side of the pipeline. It asynchronously stages activation tiles, quantized weight tiles, act-order indices, scales, and zero-points. For act-order, the kernel may also cache a chunk of group scales in shared memory to avoid repeated global reads.
**CN:** `fetch_to_shared` 是流水线的生产端。它会异步搬运激活 tile、量化权重 tile、act-order 索引、scale 和 zero-point。对于 act-order，内核还可能把一段 group scale 预先缓存到共享内存里，以避免重复全局读取。

### Register fetch, scale loading, and zero-point loading
```cpp
auto fetch_to_registers = [&](int k, int pipe) {
  ldsm<m_block_size_8 ? 2 : 4, a_type_id>(frag_a[k % 2][i], ...);
  frag_b_quant[k % 2][i] = *reinterpret_cast<I4*>(...);
};
...
auto fetch_scales_to_registers = [&](int k, int full_pipe) {
...
auto fetch_zp_to_registers = [&](int k, int full_pipe) {
```
**EN:** Once data lands in shared memory, the kernel pulls it into tensor-core fragment registers. Scale and zero-point loading are much more conditional than A/B loading because they depend on grouped quantization, act-order, columnwise scaling, and whether the same group can be reused across multiple K steps.
**CN:** 一旦数据进入共享内存，内核就会把它们提取到 Tensor Core 片段寄存器中。相比 A/B 的加载，scale 和 zero-point 的读取条件复杂得多，因为它们依赖分组量化、act-order、按列缩放，以及同一 group 是否能跨多个 K 步复用。

### Dequantization and Tensor Core compute
```cpp
auto dequant_data = [&](int q, scalar_32bit_t* frag_b_ptr, int zp = 0) {
  if constexpr (is_a_8bit && has_zp) {
    sub_zp_and_dequant<scalar_32bit_t, b_type_id, dequant_skip_flop>(q, frag_b_ptr, zp);
  } else {
    dequant<scalar_32bit_t, b_type_id, dequant_skip_flop>(q, frag_b_ptr);
  }
};
...
if constexpr (!is_a_8bit) {
  mma<a_type_id, use_fp16_accum>(frag_a[k2][i], frag_b0, frag_c[i][j][0]);
}
```
**EN:** The inner loop reconstructs quantized weights into register fragments, applies scale/zero-point logic, and immediately feeds them to Tensor Core MMA instructions. There are two matmul paths: the normal A16/A?? path and the A8 path, which accumulates differently and applies activation scales later.
**CN:** 内层循环会把量化权重重建成寄存器片段，应用 scale/zero-point 逻辑，然后立刻送入 Tensor Core MMA 指令。这里有两条矩阵乘路径：常规的 A16/A?? 路径，以及 A8 路径；后者的累加方式不同，并且会在更后面应用激活 scale。

### In-block reduction, cross-block reduction, and final writeback
```cpp
auto thread_block_reduce = [&]() { ... };
auto global_reduce_fp16 = [&](bool first = false, bool last = false) { ... };
auto global_reduce_fp32 = [&](bool first = false, bool last = false) { ... };
auto write_result = [&](bool last) {
  if (use_atomic_add && slice_count > 1) {
    atomicAdd(&C_half2[a], sh_red_half2[a]);
  } else {
    C[c_gl_wr] = sh_red[c_sh_rd];
  }
};
```
**EN:** Partial sums are first reduced within a thread block, then optionally across thread blocks that share the same output slice. Final writeback also applies late scale/bias logic and chooses between direct stores and `atomicAdd` when multiple producers target the same output tile.
**CN:** 部分和先在线程块内部归约，再在必要时跨处理同一输出切片的线程块做全局归约。最终写回阶段还会应用延迟的 scale/bias 逻辑，并在多个生产者写同一输出 tile 时在直接写入和 `atomicAdd` 之间做选择。

### Pipeline bootstrap and main loop
```cpp
auto start_pipes = [&]() {
  for (int i = 0; i < stages - 1; i++) {
    fetch_to_shared(i, i, i < slice_iters);
  }
  zero_accums();
  wait_for_stage();
  fetch_to_registers(0, 0);
};
...
while (slice_iters) {
  for (int pipe = 0; pipe < stages;) {
    for (int k = 0; k < b_sh_wr_iters; k++) {
      fetch_to_registers(k + 1, pipe % stages);
      ...
      matmul(k, ...);
    }
  }
}
```
**EN:** The main loop is a carefully unrolled producer/consumer pipeline: stage future tiles, load current fragments, execute MMA, drain the slice, reduce, write out, then initialize the next slice. The structure is intentionally dense because small control-flow changes can hurt generated code quality.
**CN:** 主循环是一个精心展开的生产者/消费者流水线：预取未来 tile、加载当前片段、执行 MMA、清空当前切片、归约、写回，然后初始化下一切片。代码结构看起来很密集，是因为细微的控制流变化都会影响最终生成代码的质量。

## Key Concepts / 关键概念
- The kernel overlaps global memory, shared memory, and Tensor Core work. / 该内核把全局内存、共享内存和 Tensor Core 计算重叠起来执行。
- Grouped quantization, act-order, zero-points, and FP8/FP4 formats are all fused into one implementation. / 分组量化、act-order、zero-point、FP8/FP4 格式都被融合进同一个实现里。
- Reduction is hierarchical: warp/block first, then optional inter-block reduction. / 归约采用分层结构：先块内，再按需做跨块归约。

## Dependencies / 依赖关系
- Depends heavily on `marlin.cuh`, `marlin_dtypes.cuh`, `dequant.h`, and `marlin_mma.h`. / 强依赖 `marlin.cuh`、`marlin_dtypes.cuh`、`dequant.h` 和 `marlin_mma.h`。
- Launched via `marlin.cu` after runtime specialization and validation. / 由 `marlin.cu` 在运行时完成特化选择和参数校验后启动。
