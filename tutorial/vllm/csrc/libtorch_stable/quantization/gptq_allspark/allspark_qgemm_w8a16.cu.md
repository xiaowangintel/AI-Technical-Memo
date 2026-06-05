# allspark_qgemm_w8a16.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `./csrc/libtorch_stable/quantization/gptq_allspark/allspark_qgemm_w8a16.cu`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements the Ampere W8A16 per-channel GPTQ GEMM path, including the custom tensor-core kernel, split-K reduction, cuBLAS fallback, and Torch binding. / 实现面向 Ampere 的 W8A16 按通道 GPTQ GEMM 路径，包括自定义 Tensor Core 内核、Split-K 归约、cuBLAS 回退和 Torch 绑定。

## Line-by-Line Analysis / 逐行分析
### Architecture gate and file role / 架构门控与文件角色
```cpp
#if defined(__CUDA_ARCH__) && __CUDA_ARCH__ < 800
torch::stable::Tensor allspark_w8a16_gemm(
    torch::stable::Tensor const& a, torch::stable::Tensor const& b_qweight,
    torch::stable::Tensor const& b_scales,
    std::optional<torch::stable::Tensor> const& b_qzeros, int64_t n,
    int64_t group_size, int64_t sm_count, int64_t sm_version,
    int64_t CUBLAS_M_THRESHOLD, bool has_zp, bool n32k16_reorder) {
  STD_TORCH_CHECK_NOT_IMPLEMENTED(
      false, "allspark_w8a16_gemm(..) requires CUDA_ARCH >= 8.0");
  return torch::stable::empty({1, 1});
}
#else
namespace allspark {
```
**EN:** The file is explicitly Ampere-or-newer code. Older architectures receive a stub that fails fast, which keeps the rest of the implementation free to use `cp.async`, tensor-core MMA, and Ampere-specific memory layouts.
**CN:** 该文件明确只服务于 Ampere 及更新架构。较老架构会走一个快速失败的桩实现，因此后续代码可以放心使用 `cp.async`、Tensor Core MMA 和 Ampere 专用内存布局。

### Global-memory tile loader / 全局内存分块加载器
```cpp
template <typename FType, typename QType, int Mtile, int Ntile, int NStage,
          int BLOCK>
struct GmemTile_W8A16_PerC_MtilexNtilex32_multistage_SM8x_SplitK {
  __device__ GmemTile_W8A16_PerC_MtilexNtilex32_multistage_SM8x_SplitK(
      const SM8x_GEMM_W8A16_Splitk_Params<FType, QType>& k_params,
      const uint32_t& A_smem_addr, const uint32_t& BQ_smem_addr,
      const uint32_t& A_stage_stride, const uint32_t& BQ_stage_stride)
      : params(k_params) {
    this_block_A_base_ptr = params.A_ptr + blockIdx.x * Mtile * params.K +
                            blockIdx.z * params.SplitK;
    this_block_B_base_ptr = params.B_ptr + blockIdx.y * Ntile * params.K +
                            blockIdx.z * params.SplitK * 4;
  }
```
**EN:** `GmemTile` computes the per-block starting addresses for activation tile `A` and packed quantized weight tile `B`. The comment above it explains the crucial `N32K16`/`n8k4n4k4` packing used to make later warp-level tensor-core loads contiguous.
**CN:** `GmemTile` 负责计算当前线程块对应的激活分块 `A` 与打包量化权重分块 `B` 的起始地址。其上方注释说明了关键的 `N32K16`/`n8k4n4k4` 打包方式，用于让后续 warp 级 Tensor Core 访存保持连续。

### Async prefetch pipeline / 异步预取流水线
```cpp
__device__ void ldgsts_first_ktiles(const int& first_k_tile,
                                    const int& k_tiles) {
  const int A_src_size = Aldg_col_idx < first_k_tile ? 16 : 0;
  for (int i = 0; i < (Mtile + M_SIZE_ONE_LOAD - 1) / M_SIZE_ONE_LOAD; ++i) {
    cp_async<16>(
        A_smem_base_addr + (i * M_SIZE_ONE_LOAD * 32) * sizeof(FType),
        this_block_A_base_ptr + i * M_SIZE_ONE_LOAD * params.K, A_src_size,
        (A_ldg_guard & (1u << i)) != 0);
  }
  const int B_src_size = (Bldg_col_idx / 4) < first_k_tile ? 16 : 0;
  for (int i = 0; i < (Ntile + N_SIZE_ONE_LOAD - 1) / N_SIZE_ONE_LOAD; ++i) {
    cp_async<16>(
        BQ_smem_base_addr + (i * N_SIZE_ONE_LOAD * 32) * sizeof(uint8_t),
        this_block_B_base_ptr + i * N_SIZE_ONE_LOAD * params.K, B_src_size,
        (B_ldg_guard & (1u << i)) != 0);
  }
  cp_async_commit_group();
}
```
**EN:** The loader uses guarded `cp_async` copies to prefill the first K-slices of `A` and quantized `B` into shared memory. The first slice may be partial, so the source width is masked by `first_k_tile`; later stages then advance in fixed 32-column chunks.
**CN:** 这里用带保护条件的 `cp_async` 将 `A` 和量化 `B` 的前几个 K 分块预取到共享内存。首个分块可能不是完整 32 列，因此通过 `first_k_tile` 控制源宽度；之后各 stage 再按固定的 32 列步进推进。

### Shared-memory decode and MMA / 共享内存解码与 MMA 计算
```cpp
__device__ void lds(const int& smem_stage_idx, const int& reg_buf_idx,
                    const int& k_phase_idx) {
  ldsm_4(A_frag[reg_buf_idx][i][0], A_frag[reg_buf_idx][i][1],
         A_frag[reg_buf_idx][i][2], A_frag[reg_buf_idx][i][3], ...);
  lds128(BQ_frag[reg_buf_idx][4 * i + 0], BQ_frag[reg_buf_idx][4 * i + 1],
         BQ_frag[reg_buf_idx][4 * i + 2], BQ_frag[reg_buf_idx][4 * i + 3], ...);
  cvt_8bx4_to_16bx4_bias128(BQ_frag[reg_buf_idx][2 * i],
                            BF_frag[reg_buf_idx][2 * i]);
  if (has_zp) {
    BF_frag[reg_buf_idx][2 * i][0] =
        __hsub2(BF_frag[reg_buf_idx][2 * i][0], num2num2(B_zero[i].x));
  }
  BF_frag[reg_buf_idx][2 * i][0] =
      __hmul2(BF_frag[reg_buf_idx][2 * i][0], num2num2(B_scale[i].x));
}
```
**EN:** `ComputeTile::lds` is the core dequantization stage. It loads A fragments with `ldmatrix`, loads packed uint8 B values from shared memory, converts 4 unsigned bytes into two half/bfloat16 pairs with a bias-128 convention, optionally subtracts zero points, and finally multiplies by per-channel scales.
**CN:** `ComputeTile::lds` 是核心解量化阶段。它通过 `ldmatrix` 读取 A 片段，从共享内存读取打包的 uint8 权重，将 4 个无符号字节按 bias-128 规则转换为两个 half/bfloat16 对，按需减去 zero-point，最后再乘上按通道 scale。

### Tensor-core accumulation and split-K fusion / Tensor Core 累加与 Split-K 融合
```cpp
__device__ void mma(const int& reg_buf_idx) {
  for (int m_idx = 0; m_idx < Mtile / 16; ++m_idx) {
    for (int n_idx = 0; n_idx < WARP_NITER; ++n_idx) {
      hmma16816_f32<FType>(
          C_frag[m_idx][n_idx], A_frag[reg_buf_idx][m_idx],
          reinterpret_cast<uint32_t (&)[2]>(BF_frag[reg_buf_idx][n_idx]));
    }
  }
}

__device__ void fused_splitk_reduce() {
  if (gridDim.z > 1) {
    ...
    if (threadIdx.x == 0) {
      atomicInc(red_count_ptr, gridDim.z);
    }
  }
}
```
**EN:** `mma` emits the actual HMMA instructions and accumulates in FP32. When Split-K is enabled, `fused_splitk_reduce` serializes partial tiles along `blockIdx.z`, accumulates them through a temporary float buffer, and lets only the final K-slice write the finished output tile.
**CN:** `mma` 发出真正的 HMMA 指令，并以 FP32 进行累加。启用 Split-K 时，`fused_splitk_reduce` 会沿 `blockIdx.z` 串行汇聚各个部分分块，通过临时 float 缓冲区累加，最终仅由最后一个 K-slice 写回完整输出。

### Kernel main loop / 内核主循环
```cpp
gmem_tile.ldgsts_first_ktiles(first_k_tile, k_tiles);
compute_tile.ldg_params();
cp_asyc_wait_group<NStage - 2>();
__syncthreads();

compute_tile.lds(lds_stage_idx, 0, 0);
int reg_buf_idx = 1;
for (; k_tiles > NStage - 1; --k_tiles) {
  sts_stage_idx = sts_stage_idx < NStage - 1 ? sts_stage_idx + 1 : 0;
  gmem_tile.ldgsts(sts_stage_idx);
  for (int k_phase_idx = 0; k_phase_idx < 2; k_phase_idx++) {
    if (k_phase_idx == 1) {
      cp_asyc_wait_group<NStage - 2>();
      __syncthreads();
    }
    compute_tile.lds(lds_stage_idx, reg_buf_idx, (k_phase_idx + 1) % 2);
    compute_tile.mma(reg_buf_idx ^ 1);
    reg_buf_idx ^= 1;
  }
}
```
**EN:** The kernel overlaps global-memory fetch, shared-memory staging, register decode, and tensor-core MMA through a multistage circular pipeline. The two register buffers (`reg_buf_idx`) implement ping-pong execution so one fragment can be consumed while the next is being prepared.
**CN:** 该内核通过多级循环流水线重叠了全局内存加载、共享内存暂存、寄存器解码和 Tensor Core MMA。两个寄存器缓冲区（`reg_buf_idx`）形成乒乓机制，使一个片段在被消费时，下一片段已经在准备中。

### Launch heuristics and fallback path / 启发式启动与回退路径
```cpp
size_t allspark_qgemm_w8a16_perc_n32k16_ampere_workspace_size(
    int m, int n, int k, int sm_count,
    BlockTileSplitkParams& fused_gemm_params) {
  int Mtile = m16_times <= 4 ? m16_times * 16 : 64;
  int Ntile =
      (float(grid_x * ((n + 127) / 128)) / sm_count > 10) || (Mtile < 64) ? 256
                                                                          : 128;
  ...
  bool enable_fuse = float(grid_x * grid_y) / sm_count >= 0.5 ? 1 : 0;
}

if (M > CUBLAS_M_THRESHOLD) {
  w8a16_gemm_dq_cublas<FType, QType>(...);
} else {
  ampere_hgemm_W8A16_perc_f16_f16_MtilexNtilex32_mma16816_multistage_AN_BTN32K16_CN_splitk<
      FType, QType>(...);
}
```
**EN:** Small-`M` cases use the custom fused kernel; large-`M` cases dequantize weights and delegate GEMM to cuBLAS. The workspace helper also chooses tile sizes, Split-K slice size, and whether fused reduction is worthwhile based on occupancy and SM count.
**CN:** 小 `M` 场景走自定义融合内核；大 `M` 场景则先解量化权重，再把 GEMM 交给 cuBLAS。工作区计算函数还会根据占用率和 SM 数量选择 tile 大小、Split-K 切分尺寸，以及是否值得启用融合归约。

### Weight restore and Torch entrypoint / 权重复原与 Torch 入口
```cpp
template <typename FT, typename QT>
__global__ void restore_N32_K16_dequantize_rhs_w8a16_perc_kernel(
    const QT* qdata, const FT* scales, const FT* zeros, FT* fdata,
    const int N_32align, const int N, const int K) {
  ...
  cvt_8bx4_to_16bx4_bias128(...);
  if (zeros != nullptr) {
    fval_reg[ni * 4 + ki] = __hsub(fval_reg[ni * 4 + ki], zero_reg[ni]);
  }
  fval_reg[ni * 4 + ki] = __hmul(fval_reg[ni * 4 + ki], scale_reg[ni]);
}
```
**EN:** The fallback path contains a dedicated kernel that converts packed `N32K16` weights back into plain N-major floating-point weights. The public Torch function validates tensors, allocates/reuses a global workspace tensor, dispatches on fp16/bf16, and registers `allspark_w8a16_gemm` into the stable extension library.
**CN:** 回退路径中包含一个专用内核，用于把打包的 `N32K16` 权重还原成普通 N-major 浮点权重。公开的 Torch 函数负责校验输入张量、分配或复用全局工作区、按 fp16/bf16 分发，并将 `allspark_w8a16_gemm` 注册到 stable 扩展库中。

## Key Concepts / 关键概念
- `N32K16` packing: weights are rearranged for warp-friendly tensor-core access. / `N32K16` 打包：将权重重排为更适合 warp 级 Tensor Core 访问的格式。
- Per-channel GPTQ dequantization: uint8 values are bias-corrected, optionally zero-point adjusted, then scaled. / 按通道 GPTQ 解量化：先做 bias 修正，再按需减 zero-point，最后乘 scale。
- Multistage `cp.async` pipeline: global memory and compute are overlapped through shared-memory staging. / 多级 `cp.async` 流水线：通过共享内存分阶段缓存，重叠全局内存访问与计算。
- Split-K strategy: either fuse reduction inside the kernel or write partial outputs for a later reduction kernel. / Split-K 策略：要么在内核内融合归约，要么先写出部分结果再做后续归约。
- cuBLAS fallback: large matrices prefer dequantize-then-GEMM for throughput. / cuBLAS 回退：大矩阵更适合“先解量化再 GEMM”的吞吐路径。

## Dependencies / 依赖关系
- `allspark_utils.cuh`: parameter structs, PTX helpers, HMMA wrappers, and dequant conversion helpers. / `allspark_utils.cuh`：提供参数结构、PTX 辅助函数、HMMA 封装和解量化转换工具。
- cuBLAS (`cublasGemmEx`): used by the large-`M` fallback path. / cuBLAS（`cublasGemmEx`）：用于大 `M` 场景的回退路径。
- Stable Torch runtime: tensor checks, device guards, workspace tensors, and operator registration. / Stable Torch 运行时：负责张量校验、设备守卫、工作区张量和算子注册。
- Repacked weights/scales/zeros from `allspark_repack.cu`: the custom kernel assumes the special packed layout already exists. / 来自 `allspark_repack.cu` 的重排权重/scale/zero：自定义内核默认输入已经是特殊打包布局。
