# gemm_kernels.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `./csrc/libtorch_stable/quantization/awq/gemm_kernels.cu`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements AWQ int4 dequantization and GEMM CUDA kernels, plus Torch-facing wrapper functions. / 实现 AWQ int4 反量化与 GEMM CUDA 内核，并提供面向 Torch 的封装函数。

## Line-by-Line Analysis / 逐行分析
### Kernel template and shared-memory tiling / 内核模板与共享内存分块
```cpp
template <int N>
__global__ void __launch_bounds__(64)
    gemm_forward_4bit_cuda_m16nXk32(int G, int split_k_iters,
                                    half* __restrict__ A, int* __restrict__ B,
                                    half* __restrict__ scaling_factors,
                                    int* __restrict__ zeros, int M, int IC,
                                    int OC, half* __restrict__ C) {
  // Only support matrix n = 64 or 128
  assert(N == 64 || N == 128);
#if defined(__CUDA_ARCH__) && __CUDA_ARCH__ < 750
  assert(false);
#else
  static constexpr uint32_t ZERO = 0x0;
  float C_warp[32];
  __shared__ half A_shared[16 * (32 + 8)];
  __shared__ half B_shared[32 * (N + 8)];
```
**EN:** The main kernel is templated on output tile width `N` and allocates shared memory for A and B tiles. Accumulation happens in FP32 registers while the kernel assumes only `N=64` or `128` are valid.
**CN:** 主内核以输出分块宽度 `N` 为模板参数，并为 A、B 分块分配共享内存。累加在 FP32 寄存器中完成，同时该实现只接受 `N=64` 或 `128`。

### On-the-fly dequantization of B / 在线反量化 B 权重
```cpp
  // preload s.f. and zeros
  int k_bound = (IC / 32 + split_k_iters - 1) / split_k_iters;
  if ((k_bound - 1) * split_k_iters * 32 + blockIdx_z * 32 >= IC) k_bound -= 1;
  for (int _k_0_0 = 0; _k_0_0 < k_bound; ++_k_0_0) {
    int k_0_0 = _k_0_0 * split_k_iters + blockIdx_z;
    __syncthreads();
    // TODO: Haotian: blockIdx_y / j_factors1 in A loading to support bsz > 16
    if (ld_A_flag) {
      *(uint4*)(A_shared_ptr) = *(uint4*)(A_ptr + (k_0_0 * 32));
    } else {
      *(uint4*)(A_shared_ptr) = make_uint4(0, 0, 0, 0);
    }

    // for (int ax0_ax1_fused_0 = 0; ax0_ax1_fused_0 < 2; ++ax0_ax1_fused_0) {
    uint32_t zeros_loaded = *(uint32_t*)(zeros_ptr + k_0_0 * 32 / G * (OC / 8));
    uint4 B_loaded_zero = dequantize_s4_to_fp16x2(zeros_loaded);
    uint4 B_loaded_scale =
        *(uint4*)(scaling_factors_ptr + k_0_0 * 32 / G * (OC));
    /*
    if (blockIdx_z == 0 && blockIdx_y == 0 && k_0_0 == 0 && threadIdx.x == 0 &&
    threadIdx.y == 0){ printf("%x %x %x %x %x %x %x %x\n", B_loaded_scale.x,
    B_loaded_scale.y, B_loaded_scale.z, B_loaded_scale.w, B_loaded_zero.x,
    B_loaded_zero.y, B_loaded_zero.z, B_loaded_zero.w);
    }
    */
    // uint4 B_loaded_scale = make_uint4(0, 0, 0, 0);
    int* B_ptr_local = B_ptr + k_0_0 * 32 * (OC / 8);

    for (int ax0_ax1_fused_0 = 0; ax0_ax1_fused_0 < N / 16; ++ax0_ax1_fused_0) {
      // B: 32 x 136 (128+8) float16
      // each warp: 32 x 4
      // each thr: read 32 bit -> convert to 8xFP16 (a UINT4) -> scale and minus
      // zero -> WB UINT4
      // *(uint4*)(B_shared + ((((ax0_ax1_fused_0 * 544) + (((int)threadIdx.y) *
      // 272)) + ((((int)threadIdx.x) >> 4) * 136)) + ((((int)threadIdx.x) & 15)
      // * 8))) = *(uint4*)(B + ((((((k_0_0 * 163840) + (ax0_ax1_fused_0 *
      // 20480)) + (((int)threadIdx.y) * 10240)) + ((((int)threadIdx.x) >> 4) *
      // 5120)) + (((int)blockIdx_y) * 128)) + ((((int)threadIdx.x) & 15) *
      // 8))); row stride in shared memory: (NWARPS * 32 * 8 / cta_N)
      uint32_t B_loaded =
          *(uint32_t*)(B_ptr_local + ax0_ax1_fused_0 * row_stride * (OC / 8));
      uint4 B_loaded_fp16 = dequantize_s4_to_fp16x2(B_loaded);

      // - zero and * scale
      // TODO (Haotian): can save 4 assembly instructions if sormulate as deq =
      // q * scale - zero * scale.
      asm volatile("sub.f16x2 %0, %1, %2;\n"
                   : "=r"(B_loaded_fp16.x)
                   : "r"(B_loaded_fp16.x), "r"(B_loaded_zero.x));
      asm volatile("fma.rn.f16x2 %0, %1, %2, %3;\n"
                   : "=r"(B_loaded_fp16.x)
                   : "r"(B_loaded_fp16.x), "r"(B_loaded_scale.x), "r"(ZERO));
      asm volatile("sub.f16x2 %0, %1, %2;\n"
                   : "=r"(B_loaded_fp16.y)
                   : "r"(B_loaded_fp16.y), "r"(B_loaded_zero.y));
      asm volatile("fma.rn.f16x2 %0, %1, %2, %3;\n"
                   : "=r"(B_loaded_fp16.y)
                   : "r"(B_loaded_fp16.y), "r"(B_loaded_scale.y), "r"(ZERO));
      asm volatile("sub.f16x2 %0, %1, %2;\n"
                   : "=r"(B_loaded_fp16.z)
                   : "r"(B_loaded_fp16.z), "r"(B_loaded_zero.z));
      asm volatile("fma.rn.f16x2 %0, %1, %2, %3;\n"
                   : "=r"(B_loaded_fp16.z)
                   : "r"(B_loaded_fp16.z), "r"(B_loaded_scale.z), "r"(ZERO));
      asm volatile("sub.f16x2 %0, %1, %2;\n"
                   : "=r"(B_loaded_fp16.w)
                   : "r"(B_loaded_fp16.w), "r"(B_loaded_zero.w));
      asm volatile("fma.rn.f16x2 %0, %1, %2, %3;\n"
                   : "=r"(B_loaded_fp16.w)
                   : "r"(B_loaded_fp16.w), "r"(B_loaded_scale.w), "r"(ZERO));
      /*
      if (ax0_ax1_fused_0 == 0 && blockIdx_z == 0 && blockIdx_y == 0 && k_0_0 ==
      0 && threadIdx.x == 17 && threadIdx.y == 0){ printf("[x] %X %X %X %X\n",
      B_loaded_fp16.x, B_loaded_fp16.y, B_loaded_fp16.z, B_loaded_fp16.w);
      }
      */

      // write back
      *(uint4*)(B_shared_ptr + ax0_ax1_fused_0 * row_stride * (N + 8)) =
          B_loaded_fp16;
    }
```
**EN:** Inside the K loop, the kernel loads packed zeros, scales, and int4 weights, then calls `dequantize_s4_to_fp16x2`. Inline half2 arithmetic subtracts zero-points and applies scales before storing the dequantized tile into shared memory.
**CN:** 在 K 循环内部，内核会加载打包后的 zero、scale 和 int4 权重，然后调用 `dequantize_s4_to_fp16x2`。随后通过内联 half2 运算完成减 zero-point 与乘 scale，并把反量化后的分块写入共享内存。

### Fragment loads with ldmatrix / 使用 ldmatrix 载入片段
```cpp
    for (int k_0_1 = 0; k_0_1 < 2; ++k_0_1) {
      {
        unsigned int addr;
        __asm__ __volatile__(
            "{ .reg .u64 addr; cvta.to.shared.u64 addr, %1; cvt.u32.u64 %0, "
            "addr; }\n"
            : "=r"(addr)
            : "l"((void*)((&(A_shared[(k_0_1 * 16)])) +
                          (((((int)threadIdx.x) & 15) * 40) +
                           ((((int)threadIdx.x) >> 4) * 8)))));

        __asm__ __volatile__(
            "ldmatrix.sync.aligned.m8n8.x4.shared.b16"
            "{%0, %1, %2, %3}, [%4];\n"
            : "=r"(((unsigned*)(A_shared_warp + 0))[0]),
              "=r"(((unsigned*)(A_shared_warp + 0))[1]),
              "=r"(((unsigned*)(A_shared_warp + 0))[2]),
              "=r"(((unsigned*)(A_shared_warp + 0))[3])
            : "r"(addr));
      }

      for (int ax1_0 = 0; ax1_0 < N / 32; ++ax1_0) {
        {
          unsigned int addr;
          __asm__ __volatile__(
              "{ .reg .u64 addr; cvta.to.shared.u64 addr, %1; cvt.u32.u64 %0, "
              "addr; }\n"
              : "=r"(addr)
              : "l"((void*)((&(B_shared[(((k_0_1 * (N * 16 + 128)) +
                                          (((int)threadIdx.y) * (N / 2))) +
                                         (ax1_0 * 16))])) +
                            (((((int)threadIdx.x) & 15) * (N + 8)) +
                             ((((int)threadIdx.x) >> 4) * 8)))));
          __asm__ __volatile__(
              "ldmatrix.sync.aligned.m8n8.x4.trans.shared.b16"
              "{%0, %1, %2, %3}, [%4];\n"
              : "=r"(((unsigned*)(B_shared_warp + (ax1_0 * 8)))[0]),
                "=r"(((unsigned*)(B_shared_warp + (ax1_0 * 8)))[1]),
                "=r"(((unsigned*)(B_shared_warp + (ax1_0 * 8)))[2]),
                "=r"(((unsigned*)(B_shared_warp + (ax1_0 * 8)))[3])
              : "r"(addr));
        }
```
**EN:** The kernel converts shared-memory addresses to the format expected by Tensor Core instructions and loads matrix fragments with `ldmatrix.sync`. A uses the regular form while B uses the transposed variant.
**CN:** 内核先把共享内存地址转换为 Tensor Core 指令所需的形式，再用 `ldmatrix.sync` 载入矩阵片段。A 使用普通形式，B 使用转置形式。

### Tensor Core MMA accumulation / Tensor Core MMA 累加
```cpp
  #else
        {
          __asm__ __volatile__(
              "mma.sync.aligned.m16n8k16.row.col.f32.f16.f16.f32"
              "{%0, %1, %2, %3}, {%4, %5, %6, %7}, {%8, %9}, {%10, %11, %12, "
              "%13};\n"
              : "=f"(((float*)(C_warp + (j_0_4 * 8)))[0]),
                "=f"(((float*)(C_warp + (j_0_4 * 8)))[1]),
                "=f"(((float*)(C_warp + (j_0_4 * 8)))[2]),
                "=f"(((float*)(C_warp + (j_0_4 * 8)))[3])
              : "r"(((unsigned*)(A_shared_warp + 0))[0]),
                "r"(((unsigned*)(A_shared_warp + 0))[1]),
                "r"(((unsigned*)(A_shared_warp + 0))[2]),
                "r"(((unsigned*)(A_shared_warp + 0))[3]),
                "r"(((unsigned*)(B_shared_warp + (j_0_4 * 8)))[0]),
                "r"(((unsigned*)(B_shared_warp + (j_0_4 * 8)))[1]),
                "f"(((float*)(C_warp + (j_0_4 * 8)))[0]),
                "f"(((float*)(C_warp + (j_0_4 * 8)))[1]),
                "f"(((float*)(C_warp + (j_0_4 * 8)))[2]),
                "f"(((float*)(C_warp + (j_0_4 * 8)))[3]));
        }

        {
          __asm__ __volatile__(
              "mma.sync.aligned.m16n8k16.row.col.f32.f16.f16.f32"
              "{%0, %1, %2, %3}, {%4, %5, %6, %7}, {%8, %9}, {%10, %11, %12, "
              "%13};\n"
              : "=f"(((float*)(C_warp + ((j_0_4 * 8) + 4)))[0]),
                "=f"(((float*)(C_warp + ((j_0_4 * 8) + 4)))[1]),
                "=f"(((float*)(C_warp + ((j_0_4 * 8) + 4)))[2]),
                "=f"(((float*)(C_warp + ((j_0_4 * 8) + 4)))[3])
              : "r"(((unsigned*)(A_shared_warp + 0))[0]),
                "r"(((unsigned*)(A_shared_warp + 0))[1]),
                "r"(((unsigned*)(A_shared_warp + 0))[2]),
                "r"(((unsigned*)(A_shared_warp + 0))[3]),
                "r"(((unsigned*)(B_shared_warp + ((j_0_4 * 8) + 4)))[0]),
                "r"(((unsigned*)(B_shared_warp + ((j_0_4 * 8) + 4)))[1]),
                "f"(((float*)(C_warp + ((j_0_4 * 8) + 4)))[0]),
                "f"(((float*)(C_warp + ((j_0_4 * 8) + 4)))[1]),
                "f"(((float*)(C_warp + ((j_0_4 * 8) + 4)))[2]),
                "f"(((float*)(C_warp + ((j_0_4 * 8) + 4)))[3]));
        }
```
**EN:** For newer architectures the code issues `mma.sync.aligned.m16n8k16.row.col.f32.f16.f16.f32` to accumulate FP16 fragments into FP32 outputs. The exact PTX variant is selected by architecture guards.
**CN:** 对于较新的架构，这里通过 `mma.sync.aligned.m16n8k16.row.col.f32.f16.f16.f32` 将 FP16 片段累加到 FP32 输出中。具体使用哪种 PTX 变体由架构条件编译决定。

### Write-back and standalone weight dequantizer / 回写与独立权重反量化
```cpp
  // TODO: Shang: Hoist loop invariance.
  for (int ax1_0_1 = 0; ax1_0_1 < (N / 32); ++ax1_0_1) {
    for (int local_id = 0; local_id < 8; ++local_id) {
      int row_offset = (((int)blockIdx_y) / j_factors1) * 16 +
                       ((int)threadIdx.x) / 4 + (local_id % 4) / 2 * 8;
      if (row_offset < M) {
        *(C_ptr + ax1_0_1 * 16 + row_offset * OC + (local_id / 4) * 8 +
          local_id % 2) = __float2half(C_warp[(ax1_0_1 * 8) + local_id]);
      }
    }
  }
#endif
}

__global__ void __launch_bounds__(64)
    dequantize_weights(int* __restrict__ B, half* __restrict__ scaling_factors,
                       int* __restrict__ zeros, half* __restrict__ C, int G) {
  static constexpr uint32_t ZERO = 0x0;
  half B_shared[32 * (128 + 8)];

  half* B_shared_ptr2 = B_shared;

  int N = blockDim.x * gridDim.x;  // 2
  int col = (blockIdx.x * blockDim.x + threadIdx.x);
  int row = blockIdx.y * blockDim.y + threadIdx.y;
  int index1 = 8 * col + 8 * row * N;
  half* C_ptr2 = C + index1;

  int index2 = col + row * N;
  int* B_ptr2 = B + index2;

  int index3 = col + (int)(row / G) * N;
  int* zeros_ptr2 = zeros + index3;
  int index4 = 8 * col + (int)(row / G) * N * 8;
  half* scaling_factors_ptr2 = scaling_factors + index4;

  uint32_t zeros_loaded = *(uint32_t*)(zeros_ptr2);
  uint4 B_loaded_zero = dequantize_s4_to_fp16x2(zeros_loaded);
  uint4 B_loaded_scale = *(uint4*)(scaling_factors_ptr2);

  uint32_t B_loaded = *(uint32_t*)B_ptr2;
  uint4 B_loaded_fp16 = dequantize_s4_to_fp16x2(B_loaded);
  asm volatile("sub.f16x2 %0, %1, %2;\n"
               : "=r"(B_loaded_fp16.x)
               : "r"(B_loaded_fp16.x), "r"(B_loaded_zero.x));
  asm volatile("fma.rn.f16x2 %0, %1, %2, %3;\n"
               : "=r"(B_loaded_fp16.x)
               : "r"(B_loaded_fp16.x), "r"(B_loaded_scale.x), "r"(ZERO));
  asm volatile("sub.f16x2 %0, %1, %2;\n"
               : "=r"(B_loaded_fp16.y)
               : "r"(B_loaded_fp16.y), "r"(B_loaded_zero.y));
  asm volatile("fma.rn.f16x2 %0, %1, %2, %3;\n"
               : "=r"(B_loaded_fp16.y)
               : "r"(B_loaded_fp16.y), "r"(B_loaded_scale.y), "r"(ZERO));
  asm volatile("sub.f16x2 %0, %1, %2;\n"
               : "=r"(B_loaded_fp16.z)
               : "r"(B_loaded_fp16.z), "r"(B_loaded_zero.z));
  asm volatile("fma.rn.f16x2 %0, %1, %2, %3;\n"
               : "=r"(B_loaded_fp16.z)
               : "r"(B_loaded_fp16.z), "r"(B_loaded_scale.z), "r"(ZERO));
  asm volatile("sub.f16x2 %0, %1, %2;\n"
               : "=r"(B_loaded_fp16.w)
               : "r"(B_loaded_fp16.w), "r"(B_loaded_zero.w));
  asm volatile("fma.rn.f16x2 %0, %1, %2, %3;\n"
               : "=r"(B_loaded_fp16.w)
               : "r"(B_loaded_fp16.w), "r"(B_loaded_scale.w), "r"(ZERO));

  *(uint4*)B_shared_ptr2 = B_loaded_fp16;

  for (int i = 0; i < 8; ++i) {
    *(C_ptr2 + i) = B_shared[i];
  }
}
```
**EN:** After accumulation, the kernel converts FP32 accumulators back to half and writes them into the split-K output tensor. The same file also defines `dequantize_weights`, a standalone kernel that fully expands packed AWQ weights into FP16.
**CN:** 累加完成后，内核把 FP32 累加结果转回 half，并写入 split-K 输出张量。该文件还定义了 `dequantize_weights`，用于把打包的 AWQ 权重完整展开为 FP16。

### Torch wrapper: awq_dequantize / Torch 封装：awq_dequantize
```cpp
torch::stable::Tensor awq_dequantize(torch::stable::Tensor _kernel,
                                     torch::stable::Tensor _scaling_factors,
                                     torch::stable::Tensor _zeros,
                                     int64_t split_k_iters, int64_t thx,
                                     int64_t thy) {
  int in_c = _kernel.size(0);
  int qout_c = _kernel.size(1);
  int out_c = qout_c * 8;
  int G = in_c / _scaling_factors.size(0);

  int x_thread = thx;
  int y_thread = thy;

  int x_blocks = 1;
  int y_blocks = 1;
  if (thx == 0) {
    x_thread = qout_c;
  }
  if (thy == 0) {
    y_thread = in_c;
  }
  if (thx == 0 && thy == 0) {
    x_thread = 8;
    y_thread = 8;
    x_blocks = (int)(qout_c / 8);
    y_blocks = (int)(in_c / 8);
  }

  const torch::stable::accelerator::DeviceGuard device_guard(
      _scaling_factors.get_device_index());

  auto _de_kernel =
      torch::stable::empty({in_c, out_c}, _scaling_factors.scalar_type(),
                           std::nullopt, _scaling_factors.device());

  auto kernel = reinterpret_cast<int*>(_kernel.mutable_data_ptr<int>());
  auto de_kernel = reinterpret_cast<half*>(
      _de_kernel.mutable_data_ptr<torch::headeronly::Half>());
  auto scaling_factors = reinterpret_cast<half*>(
      _scaling_factors.mutable_data_ptr<torch::headeronly::Half>());
  auto zeros = reinterpret_cast<int*>(_zeros.mutable_data_ptr<int>());

  dim3 num_blocks(x_blocks, y_blocks);
  dim3 threads_per_block(x_thread, y_thread);

  const cudaStream_t stream = get_current_cuda_stream();
  vllm::awq::dequantize_weights<<<num_blocks, threads_per_block, 0, stream>>>(
      kernel, scaling_factors, zeros, de_kernel, G);

  return _de_kernel;
```
**EN:** `awq_dequantize` computes launch dimensions, allocates the dense FP16 output tensor, casts Torch buffers to raw pointers, and launches the CUDA dequantization kernel on the current stream.
**CN:** `awq_dequantize` 会计算 launch 维度、分配稠密 FP16 输出张量、把 Torch 缓冲区转换为原始指针，并在当前 stream 上启动 CUDA 反量化内核。

### Torch wrapper: awq_gemm dispatch / Torch 封装：awq_gemm 调度
```cpp
torch::stable::Tensor awq_gemm(torch::stable::Tensor _in_feats,
                               torch::stable::Tensor _kernel,
                               torch::stable::Tensor _scaling_factors,
                               torch::stable::Tensor _zeros,
                               int64_t split_k_iters) {
  int num_in_feats = _in_feats.size(0);
  int num_in_channels = _in_feats.size(1);
  const torch::stable::accelerator::DeviceGuard device_guard(
      _in_feats.get_device_index());

  auto _out_feats = torch::stable::empty(
      {split_k_iters, num_in_feats, _kernel.size(1) * 8},
      _in_feats.scalar_type(), std::nullopt, _in_feats.device());
  int num_out_feats = _out_feats.size(-2);
  int num_out_channels = _out_feats.size(-1);

  auto in_feats = reinterpret_cast<half*>(
      _in_feats.mutable_data_ptr<torch::headeronly::Half>());
  auto kernel = reinterpret_cast<int*>(_kernel.mutable_data_ptr<int>());
  auto out_feats = reinterpret_cast<half*>(
      _out_feats.mutable_data_ptr<torch::headeronly::Half>());
  auto scaling_factors = reinterpret_cast<half*>(
      _scaling_factors.mutable_data_ptr<torch::headeronly::Half>());
  auto zeros = reinterpret_cast<int*>(_zeros.mutable_data_ptr<int>());
  int group_size = num_in_channels / _scaling_factors.size(0);

  if (num_out_channels % 64 != 0)
    throw std::invalid_argument("OC is not multiple of cta_N = 64");
  if (num_out_channels % 8 != 0)
    throw std::invalid_argument("OC is not multiple of pack_num = 8");
  if (group_size % 32 != 0)
    throw std::invalid_argument("Group size should be a multiple of 32");
  if (num_out_channels % group_size != 0)
    throw std::invalid_argument("OC is not multiple of Group size");

  const cudaStream_t stream = get_current_cuda_stream();
  if (num_out_channels % 128 == 0) {
    int j_factors1 = num_out_channels / 128 / 1;
    dim3 num_blocks((num_out_feats + 16 - 1) / 16 * j_factors1 * split_k_iters);
    // threadIdx.x: 32
    // threadIdx.y: i_factors[2] * j_factors[2]
    dim3 threads_per_block(32, 2);
    vllm::awq::gemm_forward_4bit_cuda_m16nXk32<128>
        <<<num_blocks, threads_per_block, 0, stream>>>(
            group_size, split_k_iters, in_feats, kernel, scaling_factors, zeros,
            num_in_feats, num_in_channels, num_out_channels, out_feats);
  } else if (num_out_channels % 64 == 0) {
    int j_factors1 = num_out_channels / 64 / 1;
    dim3 num_blocks(1 * (num_out_feats + 16 - 1) / 16 * j_factors1 *
                    split_k_iters);

    // threadIdx.x: 32
    // threadIdx.y: i_factors[2] * j_factors[2]
    dim3 threads_per_block(32, 2);
    vllm::awq::gemm_forward_4bit_cuda_m16nXk32<64>
        <<<num_blocks, threads_per_block, 0, stream>>>(
            group_size, split_k_iters, in_feats, kernel, scaling_factors, zeros,
            num_in_feats, num_in_channels, num_out_channels, out_feats);
  }
  return torch::stable::sum(_out_feats, 0);
```
**EN:** `awq_gemm` validates output-channel and group-size constraints, allocates a split-K workspace tensor, dispatches either the 128-wide or 64-wide kernel variant, and finally reduces the split-K dimension with `torch::stable::sum`.
**CN:** `awq_gemm` 会校验输出通道与 group size 约束，分配 split-K 工作张量，按条件调度 128 宽或 64 宽内核，最后通过 `torch::stable::sum` 对 split-K 维做归约。

## Key Concepts / 关键概念
- AWQ uses online dequantization so quantized weights stay compact in memory / AWQ 采用在线反量化，使量化权重在内存中保持紧凑
- `ldmatrix` plus `mma.sync` expose Tensor Core throughput directly / `ldmatrix` 与 `mma.sync` 直接利用 Tensor Core 吞吐能力
- Split-K is used to parallelize the K dimension and reduced afterward / 通过 Split-K 并行处理 K 维，并在事后完成归约

## Dependencies / 依赖关系
- Includes `dequantize.cuh` for the packed-int4 to FP16 helper / 依赖 `dequantize.cuh` 提供 packed-int4 到 FP16 的辅助函数
- Uses CUDA half/Tensor Core instructions and `torch::stable` tensor APIs / 使用 CUDA half/Tensor Core 指令以及 `torch::stable` 张量 API
