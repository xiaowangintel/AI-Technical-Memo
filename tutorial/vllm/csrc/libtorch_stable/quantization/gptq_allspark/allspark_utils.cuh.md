# allspark_utils.cuh — Code Analysis / 代码分析

        ## Source / 来源
        - **File**: `./csrc/libtorch_stable/quantization/gptq_allspark/allspark_utils.cuh`
        - **Repository**: vllm-project/vllm
        - **Purpose**: Supplies low-level data structures, reduction helpers, PTX memory primitives, MMA wrappers, and quantized-value conversion utilities for the AllSpark GPTQ CUDA kernels. / 为 AllSpark GPTQ CUDA 内核提供底层数据结构、归约辅助函数、PTX 访存原语、MMA 封装以及量化值转换工具。

        ## Line-by-Line Analysis / 逐行分析
        ### Error macros and kernel parameters / 错误宏与内核参数
        ```cpp
        #define CHECK_CUDA(cmd) ...
        #define CHECK_CUBLAS(cmd) ...

        template <typename FType, typename QType>
        struct SM8x_GEMM_W8A16_Splitk_Params {
          const FType* A_ptr;
          const QType* B_ptr;
          const FType* B_scale_ptr;
          const FType* B_zero_ptr;
          FType* C_ptr;
          int M;
          int N;
          int K;
          int SplitK;
          FType* C_split_ptr;
          float* C_tmp_ptr;
          uint32_t* red_count_ptr;
        };
        ```
        **EN:** This section defines the common contract shared by the AllSpark kernels. `SM8x_GEMM_W8A16_Splitk_Params` packages all matrix pointers plus the buffers needed for either non-fused or fused Split-K reduction.
        **CN:** 这一段定义了 AllSpark 内核共享的统一契约。`SM8x_GEMM_W8A16_Splitk_Params` 打包了所有矩阵指针，以及非融合或融合 Split-K 归约所需的额外缓冲区。

        ### Split-K reduction helper / Split-K 归约辅助函数
        ```cpp
        template <typename FType, int BLOCK, int N_MATRIX>
        __global__ void f16_gemm_splitk_reduce_kernel(const FType* C_split, FType* C,
                                                      uint32_t n, uint32_t n_matrix,
                                                      uint32_t matrix_size) {
          auto idx = blockIdx.x * BLOCK + threadIdx.x;
          if (idx >= matrix_size) {
            return;
          }
          float sum = 0.f;
          int n_mat = N_MATRIX > 0 ? N_MATRIX : (int)n_matrix;
          for (int i = 0; i < n_mat; ++i) {
            sum += MarlinScalarType2<FType>::num2float(C_split[idx + i * matrix_size]);
          }
          C[idx] = MarlinScalarType2<FType>::float2num(sum);
        }
        ```
        **EN:** When Split-K is not fused into the main GEMM, this kernel sums multiple partial result matrices elementwise. The template specializes common `n_matrix` counts to avoid an extra runtime loop bound.
        **CN:** 当 Split-K 没有融合进主 GEMM 时，这个内核会对多个部分结果矩阵做逐元素求和。模板对常见 `n_matrix` 数量做了特化，从而减少运行时循环边界的开销。

        ### Type traits and shared-memory addresses / 类型特征与共享内存地址
        ```cpp
        template <typename T>
        struct HalfType;
        template <>
        struct HalfType<half> {
          using T1 = __half;
          using T2 = __half2;
        };
        template <>
        struct HalfType<__nv_bfloat16> {
          using T1 = __nv_bfloat16;
          using T2 = __nv_bfloat162;
        };

        __device__ __forceinline__ uint32_t smem_u32addr(const void* smem_ptr) {
          asm("{.reg .u64 u64addr; ... cvt.u32.u64 %0, u64addr;}" : "=r"(addr) : "l"(smem_ptr));
          return addr;
        }
        ```
        **EN:** `HalfType` abstracts over fp16 and bf16 packed pair types, while `smem_u32addr` converts a generic shared-memory pointer into the 32-bit address form required by several inline PTX instructions.
        **CN:** `HalfType` 抽象了 fp16 与 bf16 的成对打包类型，而 `smem_u32addr` 则把普通共享内存指针转换为多种 PTX 内联指令所要求的 32 位地址形式。

        ### PTX load/store wrappers / PTX 加载与存储封装
        ```cpp
        template <typename T>
        __device__ __forceinline__ void ldg64_ca(T& r0, T& r1, const void* ptr,
                                                 bool guard) {
          asm volatile(
              "{.reg .pred p;
"
              " setp.ne.b32 p, %3, 0;
"
              " @p ld.global.ca.v2.b32 {%0, %1}, [%2];}
"
              : "=r"(reinterpret_cast<uint32_t&>(r0)),
                "=r"(reinterpret_cast<uint32_t&>(r1))
              : "l"(ptr), "r"((int)guard));
        }
        ```
        **EN:** The header wraps common global/shared memory transactions in strongly-typed helpers (`ldg16`, `ldg64`, `ldg128`, `lds128`, `stg128`, `ldsm_4`). These wrappers centralize alignment assumptions and guard predicates used throughout the kernels.
        **CN:** 该头文件把常见的全局/共享内存事务封装成强类型辅助函数（`ldg16`、`ldg64`、`ldg128`、`lds128`、`stg128`、`ldsm_4`）。这样可以把对齐假设和 guard 谓词集中管理，供各个内核复用。

        ### MMA and async copy primitives / MMA 与异步拷贝原语
        ```cpp
        template <typename FType>
        __device__ __forceinline__ void hmma16816_f32(float (&d)[4],
                                                      const uint32_t (&a)[4],
                                                      const uint32_t (&b)[2]);

        template <int SIZE_IN_BYTES>
        __device__ __forceinline__ void cp_async(const uint32_t smem_addr,
                                                 const void* gmem_ptr,
                                                 const int src_in_bytes, bool guard) {
          asm volatile(
              "{.reg.pred p;
"
              " setp.ne.b32 p, %4, 0;
"
              " @p cp.async.cg.shared.global [%0], [%1], %2, %3;}
"
              ::"r"(smem_addr), "l"(gmem_ptr), "n"(SIZE_IN_BYTES),
                "r"(src_in_bytes), "r"((int)guard));
        }
        ```
        **EN:** These wrappers expose the two hardware features the GEMM path relies on most: tensor-core MMA (`mma.sync`) and asynchronous global-to-shared copies (`cp.async`). Specializations are provided separately for fp16 and bf16 accumulator behavior.
        **CN:** 这些封装暴露了 GEMM 路径最依赖的两类硬件能力：Tensor Core MMA（`mma.sync`）和异步全局到共享拷贝（`cp.async`）。同时还分别为 fp16 与 bf16 的累加行为提供了特化实现。

        ### Quantized byte to half/bfloat conversion / 量化字节到 half/bfloat 的转换
        ```cpp
        template <>
        __device__ __forceinline__ void cvt_8bx4_to_16bx4_bias128<__half2>(
            const uint32_t& idata, __half2* fdata) {
          uint32_t i10, i32;
          asm volatile(
              "prmt.b32 %0, %2, 0x64, 0x4140;"
              "prmt.b32 %1, %2, 0x64, 0x4342;"
              : "=r"(i10), "=r"(i32)
              : "r"(idata));
          static constexpr uint32_t MAGIC_NUM = 0x64806480;
          fdata[0] = __hsub2(reinterpret_cast<const __half2&>(i10),
                             reinterpret_cast<const __half2&>(MAGIC_NUM));
        }
        ```
        **EN:** `cvt_8bx4_to_16bx4_bias128` is the fast dequantization primitive. It uses bit permutation instructions to unpack four uint8 values into fp16/bf16 lanes and subtract the implicit `128` bias expected by this quantization scheme.
        **CN:** `cvt_8bx4_to_16bx4_bias128` 是快速解量化原语。它利用位重排指令把 4 个 uint8 展开到 fp16/bf16 通道中，并减去该量化方案约定的隐式偏置 `128`。

        ## Key Concepts / 关键概念
        - Shared parameter packing: one struct carries everything a kernel needs. / 共享参数打包：单个结构体携带了内核所需的全部关键信息。
        - Inline PTX abstraction: low-level instructions are wrapped once and reused everywhere. / 内联 PTX 抽象：底层指令统一封装后可在各处复用。
        - Mixed-precision handling: utilities explicitly support both fp16 and bf16 paths. / 混合精度处理：工具明确同时支持 fp16 与 bf16 路径。
        - Fast dequantization: uint8-to-b16 conversion is optimized as a register-level primitive. / 快速解量化：uint8 到 b16 的转换被优化成寄存器级原语。

        ## Dependencies / 依赖关系
        - CUDA headers (`cuda_fp16.h`, `cuda_bf16.h`, `cuda_runtime.h`): provide scalar/vector types and runtime APIs. / CUDA 头文件（`cuda_fp16.h`、`cuda_bf16.h`、`cuda_runtime.h`）：提供标量/向量类型和运行时 API。
        - Marlin dtypes (`quantization/marlin/marlin_dtypes.cuh`): used for numeric conversions between storage and float. / Marlin 类型头（`quantization/marlin/marlin_dtypes.cuh`）：用于存储格式与浮点之间的数值转换。
        - AllSpark CUDA source files: both `allspark_qgemm_w8a16.cu` and `allspark_repack.cu` depend heavily on these helpers. / AllSpark CUDA 源文件：`allspark_qgemm_w8a16.cu` 与 `allspark_repack.cu` 都大量依赖这里的辅助函数。
