# hadamard_transform_cuda.cu — Code Analysis / 代码分析

        ## Source / 来源
        - **File**: `./csrc/libtorch_stable/quantization/hadamard/hadacore/hadamard_transform_cuda.cu`
        - **Repository**: vllm-project/vllm
        - **Purpose**: Implements a CUDA Hadamard transform for fp16/bf16 tensors using tensor cores, asynchronous shared-memory staging, and a Torch-facing wrapper. / 为 fp16/bf16 张量实现 CUDA Hadamard 变换，使用 Tensor Core、异步共享内存分阶段加载以及面向 Torch 的包装接口。

        ## Line-by-Line Analysis / 逐行分析
        ### Launch tables and MMA helpers / 启动参数表与 MMA 辅助函数
        ```cpp
        constexpr int launch_configs_big[7][3] = {
            {2, 1, 24},
            {2, 2, 16},
            {2, 4, 8},
            {2, 8, 4},
            {2, 16, 3},
            {4, 16, 2},
            {8, 16, 1}
        };

        template <torch::headeronly::ScalarType dtype>
        __device__ __forceinline__ void mma_m16_n8_k16_b16_b16_b16_noacc(...)
        ```
        **EN:** Before the kernel body starts, the file defines launch heuristics for large Hadamard sizes and provides tiny wrappers around tensor-core MMA instructions. Those wrappers normalize fp16 and bf16 execution into a shared fragment-oriented API.
        **CN:** 在进入内核主体之前，文件先定义了大尺寸 Hadamard 的启动参数表，并用一些小型封装包装 Tensor Core MMA 指令。这些封装把 fp16 与 bf16 的执行路径统一成共享的 fragment 级接口。

        ### Kernel entry and async staging / 内核入口与异步暂存
        ```cpp
        template<int64_t num_chunks, int64_t warps_per_block, int64_t log_had_size,
                 int64_t blocks_per_sm, bool enable_mask,
                 torch::headeronly::ScalarType dtype>
        __global__ void __launch_bounds__(32 * warps_per_block, blocks_per_sm)
        hadamard_transform_kernel(b16* a, b16* out, int total_num_chunks) {
            b32 b_frag_all[num_chunks][4];
            int64_t blockid = blockIdx.x * warps_per_block + threadIdx.x / 32;
            int64_t threadid = threadIdx.x % 32;
            extern __shared__ b32 bfrag_arr[];
            ...
            asm volatile(
                "cp.async.cg.shared.global [%0], [%1], 16;
"
                "cp.async.commit_group;
"
                :: "l"(shared_ptr), "l"(a_ptr));
        }
        ```
        **EN:** The transform kernel is fully template-specialized by chunk count, block composition, Hadamard size, masking mode, and dtype. Its first phase streams packed b16 fragments from global memory into shared memory with `cp.async`, so the warp can later reinterpret them in tensor-core-friendly layouts.
        **CN:** 该变换内核会对 chunk 数、block 组成、Hadamard 大小、mask 模式和 dtype 进行完全模板特化。它的第一阶段通过 `cp.async` 把打包的 b16 片段从全局内存流式搬运到共享内存，为后续按 Tensor Core 友好布局重解释做准备。

        ### Building the Hadamard fragments / 构造 Hadamard 片段
        ```cpp
        constexpr b16 fp16_1p[4] = {...};
        constexpr b16 fp16_1n[4] = {...};
        constexpr b16 bf16_1p[4] = {...};
        constexpr b16 bf16_1n[4] = {...};

        const b32 had_16_p1[4][4] = { ... };
        const b32 had_16_p2[4][4] = { ... };
        const b32 had_16_mask[3][4] = { ... };
        ```
        **EN:** Instead of materializing dense Hadamard matrices in memory, the kernel synthesizes 16x16 sign patterns directly in registers. The `p_p`, `p_n`, `n_p`, and `n_n` combinations encode the ±1 structure required by the transform.
        **CN:** 内核并不会在内存中显式构造稠密 Hadamard 矩阵，而是直接在寄存器中合成 16x16 的符号模式。`p_p`、`p_n`、`n_p`、`n_n` 等组合编码了变换所需的 ±1 结构。

        ### Warp-level data reshaping and MMA loop / Warp 级数据重排与 MMA 循环
        ```cpp
        for (int64_t k = 0; k < num_chunks; k++) {
            ...
            for(int64_t i = 0, remaining_log_had_size = log_had_size - l * 8;
                i < 2 && remaining_log_had_size > 0; i++) {
                int64_t had_off = ((remaining_log_had_size < 4) &&
                                   !(log_had_size <= 4 || log_had_size % 4 == 0)) ? 4 : 0;
                mma_m16_n16_k16_b16_b16_b16_noacc<dtype>(...);
                remaining_log_had_size -= 4;
                if (remaining_log_had_size <= 0 && i == 0) {
                    matrix_transpose_m8_n8_b16_inplace(b_frag_all[k][0]);
                } else {
                    b32 temp = b_frag_all[k][1];
                    b_frag_all[k][1] = b_frag_all[k][2];
                    b_frag_all[k][2] = temp;
                }
            }
        }
        ```
        **EN:** The heart of the kernel is not just MMA; it is the repeated reshaping of fragments so they match tensor-core register layout at each transform stage. Shuffle instructions, register swaps, and in-place transposes turn a Hadamard butterfly into a sequence of MMA-friendly subproblems.
        **CN:** 内核的核心不仅是 MMA 本身，更在于反复重排片段，使其在每个变换阶段都匹配 Tensor Core 的寄存器布局。shuffle、寄存器交换和原地转置共同把 Hadamard 蝶形结构改写成一系列适合 MMA 的子问题。

        ### Multi-stage store strategy / 多阶段写回策略
        ```cpp
        if (log_had_size <= 8)
            break;

        if constexpr(log_had_size >= 12) {
            ...
            b32* store = bfrag_arr + (128 >> part8_log_had_size) *
                                   (num_chunks * (blockid % warps_per_block));
            ...
            __syncthreads();
            store = ((b32*) out) + (blockid / warps_per_block) *
                                 (num_chunks * warps_per_block) * 128;
            int4* store4 = (int4*) store;
            int4* bfrag_arr4 = (int4*) bfrag_arr;
            for (int64_t warp_off = 0; warp_off < ...; warp_off += 32 * warps_per_block) {
                store4[total_off] = bfrag_arr4[total_off];
            }
        }
        ```
        **EN:** Small transforms can write directly to global memory after the first pass. Larger transforms reuse shared memory as an intermediate transpose/cache space and only flush to global memory after all chunks have been regrouped into a coalesced layout.
        **CN:** 小尺寸变换在第一轮之后就可以直接写回全局内存；更大的变换则把共享内存继续当作中间转置/缓存空间使用，等所有 chunk 被重组为连续布局后再统一刷回全局内存。

        ### Runtime dispatch / 运行时分发
        ```cpp
        template <torch::headeronly::ScalarType dtype>
        void run_fht(void* a_mat_ptr, void* out_ptr, int64_t numel,
                     int64_t had_size, cudaStream_t stream) {
            if (numel <= 256) {
                switch (had_size) {
                    case (1<<1): run_kernel<dtype, ..., 1, ...>(a_mat, out, num_chunks, stream); break;
                    ...
                    case (1<<8): run_kernel<dtype, ..., 8, ...>(a_mat, out, num_chunks, stream); break;
                }
            } else {
                switch (had_size) {
                    ...
                    case (1<<15): run_kernel<dtype, launch_configs_big[6][0], launch_configs_big[6][1], 15, launch_configs_big[6][2]>(a_mat, out, num_chunks, stream); break;
                }
            }
        }
        ```
        **EN:** `run_fht` converts the runtime Hadamard size into a compile-time template instantiation. That gives the compiler fixed loop bounds and fixed fragment shapes, which is essential for this style of heavily unrolled tensor-core code.
        **CN:** `run_fht` 把运行时的 Hadamard 大小转换为编译期模板实例。这样编译器就能获得固定的循环边界和固定的片段形状，这对大量展开的 Tensor Core 代码至关重要。

        ### Torch wrapper / Torch 包装层
        ```cpp
        torch::stable::Tensor hadacore_transform(torch::stable::Tensor& x, bool inplace) {
            const int had_size = x.size(-1);
            STD_TORCH_CHECK(is_power_of_two(had_size) && (had_size <= (1U << 15)),
                "Only power of two Hadamard sizes up to 2^15 are supported, got ", had_size);
            x = torch::stable::reshape(x, {-1, had_size});
            if (numel % 256 != 0) {
                x = torch::stable::pad(x, {0, 0, 0, (256 - numel % 256) / had_size});
            }
            torch::stable::Tensor out = inplace ? x : torch::stable::empty_like(x);
            ...
            hadacore::run_fht<SCALAR_TYPE>(x.data_ptr(), x.data_ptr(), x.numel(), had_size, stream);
        }
        ```
        **EN:** The public wrapper enforces CUDA + fp16/bf16 + power-of-two constraints, reshapes the input into rows of `had_size`, pads to a 256-element multiple, and dispatches into the compiled kernels. As written, the call passes `x.data_ptr()` as both input and output, so the transform execution path is effectively in-place.
        **CN:** 公开包装层会检查 CUDA、fp16/bf16 以及 2 的幂长度约束，把输入重塑为按 `had_size` 分行的矩阵，并补齐到 256 元素倍数后再分发到编译好的内核。按当前代码写法，调用时把 `x.data_ptr()` 同时作为输入与输出，因此执行路径实际上是原地变换。

        ## Key Concepts / 关键概念
        - Tensor-core Hadamard: the transform is expressed as MMA-friendly 16x16 building blocks. / Tensor Core Hadamard：该变换被表示成适合 MMA 的 16x16 基础块。
        - Register choreography: shuffles, swaps, and transposes are as important as arithmetic. / 寄存器编排：shuffle、交换和转置与算术计算同样关键。
        - Size-specialized kernels: each Hadamard size becomes a separate template instantiation. / 尺寸特化内核：每个 Hadamard 大小都会生成独立模板实例。
        - Shared-memory staging: large transforms use shared memory as both async input cache and intermediate transpose buffer. / 共享内存暂存：大尺寸变换把共享内存同时用作异步输入缓存和中间转置缓冲区。

        ## Dependencies / 依赖关系
        - CUDA runtime + tensor-core PTX (`mma.sync`, `cp.async`, `movmatrix`): underpin the whole implementation. / CUDA 运行时与 Tensor Core PTX（`mma.sync`、`cp.async`、`movmatrix`）：支撑整个实现。
        - Stable Torch helpers (`torch_utils.h`, `dispatch_utils.h`): provide stream access, dispatch macros, and registration support. / Stable Torch 辅助头（`torch_utils.h`、`dispatch_utils.h`）：提供 stream 获取、dispatch 宏和注册支持。
        - Input tensors must be fp16/bf16 CUDA tensors with power-of-two last dimension. / 输入张量必须是 fp16/bf16 的 CUDA 张量，且最后一维大小为 2 的幂。
