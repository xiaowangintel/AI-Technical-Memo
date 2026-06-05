# minimax_reduce_rms_kernel.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/minimax_reduce_rms_kernel.cu`
- **Repository**: `vllm-project/vllm`
- **Purpose**: [EN] Implements MiniMax-specific cross-rank RMSNorm kernels that combine local variance reduction, Lamport-style workspace communication, and TensorRT-LLM-oriented launch policies. / [CN] 实现面向 MiniMax 的跨 rank RMSNorm 内核，将本地方差归约、基于 Lamport 风格的 workspace 通信以及面向 TensorRT-LLM 的启动策略结合在一起。

## Line-by-Line Analysis / 逐行分析

### Lamport-style communicator / Lamport 风格通信器
```cpp
template <int NRanks>
struct LamportComm {
  __device__ __forceinline__ LamportComm(void** workspace, int rank) {
    counter_ptr = &reinterpret_cast<int*>(workspace[NRanks * 3])[0];
    flag_ptr = &reinterpret_cast<int*>(workspace[NRanks * 3])[2];
    clear_ptr = &reinterpret_cast<int64_t*>(workspace[NRanks * 3 + 1])[0];
    flag_value = *flag_ptr;
    auto comm_size = reinterpret_cast<int64_t*>(workspace[NRanks * 3 + 1])[1];
    clear_size = *clear_ptr;
    int data_offset = flag_value % 3;
    int clear_offset = (flag_value + 2) % 3;
    for (int r = 0; r < NRanks; ++r) {
      data_bufs[r] = reinterpret_cast<uint8_t*>(workspace[2 * NRanks + r]) +
                     data_offset * comm_size;
    }
    clear_buf = reinterpret_cast<uint8_t*>(workspace[2 * NRanks + rank]) +
                clear_offset * comm_size;
    __syncthreads();
    if (threadIdx.x == 0) {
      atomicAdd(counter_ptr, 1);
    }
  }
```
**EN:** `LamportComm` maps a raw workspace array into synchronization metadata and per-rank communication buffers. The rotating `flag_value % 3` logic creates a triple-buffered protocol: one region is being written, one is being read, and one is being cleared for the next iteration.
**CN:** `LamportComm` 会把原始 workspace 指针数组解释成同步元数据和各 rank 的通信缓冲区。`flag_value % 3` 形成一个三缓冲协议：一个区域用于当前写入、一个区域用于当前读取、还有一个区域提前清零供下一轮使用。

### Sentinel and reduction helpers / 哨兵值与归约辅助
```cpp
__device__ __forceinline__ bool is_neg_zero(float v) {
  return *reinterpret_cast<uint32_t*>(&v) == 0x80000000;
}
...
template <typename T, int NUM>
__inline__ __device__ T blockReduceSumV2(T* val) {
  static __shared__ T shared[NUM][33];
  int lane = threadIdx.x & 0x1f;
  int wid = threadIdx.x >> 5;
  ...
  warpReduceSumV2<T, NUM>(val);
  ...
}
```
**EN:** Negative zero is used as an in-band sentinel meaning “data not ready yet.” That is why the code needs exact bitwise tests instead of ordinary `== 0`. The reduction helpers then combine warp-level shuffles with shared-memory staging to produce block-wide sums.
**CN:** 负零在这里被当作“数据尚未就绪”的带内哨兵，因此必须使用按位判断，而不能简单写成 `== 0`。归约辅助函数则结合 warp shuffle 和共享内存中转，完成 block 级求和。

### Index helper for the scalar kernel / 标量内核的索引助手
```cpp
template <typename DType>
class IndexHelper {
 public:
  __device__ __forceinline__ IndexHelper(MiniMaxReduceRMSParams const& params) {
#if (defined(__CUDA_ARCH__) && (__CUDA_ARCH__ >= 900))
    namespace cg = cooperative_groups;
    cg::cluster_group cluster = cg::this_cluster();
    cg::grid_group grid = cg::this_grid();
    token_id = grid.cluster_rank();
    access_id_in_token = cluster.thread_rank();
    token_stride = grid.num_clusters();
#else
    token_id = blockIdx.x;
    access_id_in_token = threadIdx.x;
    token_stride = gridDim.x;
#endif
    access_id = token_id * params.hidden_dim / kElemsPerAccess<DType> +
                access_id_in_token;
```
**EN:** `IndexHelper` abstracts the difference between Hopper-style cluster launches and ordinary block indexing. The scalar kernel can therefore compute token/access coordinates the same way on both architectures.
**CN:** `IndexHelper` 把 Hopper 风格 cluster launch 与普通 block 索引方式统一起来，因此标量内核可以在不同架构上用相同的逻辑计算 token/access 坐标。

### Scalar all-reduce RMSNorm kernel / 标量 all-reduce RMSNorm 内核
```cpp
template <typename DType, int NRanks>
__global__ void __launch_bounds__(1024)
    minimax_reduce_rms_kernel_lamport(MiniMaxReduceRMSParams params) {
  IndexHelper<DType> index_helper(params);
  ...
  for (int idx = access_id; idx < tot_access;
       idx += access_stride, token_id += token_stride) {
    alignas(16) DType vals[kElemsPerAccess<DType>];
    float sum_variance = 0.F;
    *reinterpret_cast<float4*>(vals) =
        reinterpret_cast<float4*>(params.allreduce_in)[idx];
```
**EN:** This kernel reads one packed vector of local activations, accumulates local squared norms, and then uses the Lamport workspace to exchange those partial sums across ranks. After the full sum is available, it applies RMS normalization with `rms_gamma` and writes the normalized vector to `rms_norm_out`.
**CN:** 该内核每次读取一个打包向量的本地激活值，先累加本地平方范数，再通过 Lamport workspace 在各 rank 之间交换这些部分和。当全局平方和到齐后，它用 `rms_gamma` 执行 RMS 归一化，并把结果写入 `rms_norm_out`。

### Cross-rank polling and normalization / 跨 rank 轮询与归一化
```cpp
if (threadIdx.x == 0) {
  for (int r = 0; r < NRanks; ++r) {
    reinterpret_cast<float*>(
        comm.data_bufs[r])[(params.rank * tot_tokens) + token_id] =
        (sum_variance);
  }
}
...
while (!done) {
  done = true;
  for (int r = 0; r < NRanks; ++r) {
    vars_all_ranks[r] = ld_global_volatile(&reinterpret_cast<float*>(
        comm.data_bufs[params.rank])[(r * tot_tokens) + token_id]);
    done &= !is_neg_zero(vars_all_ranks[r]);
  }
}
...
vals[i] = static_cast<DType>(
    static_cast<float>(vals[i]) *
    rsqrtf(
        (sum_variance / static_cast<float>(params.hidden_dim) / NRanks) +
        params.rms_eps) *
    static_cast<float>(norm_weight[i]));
```
**EN:** Each rank publishes its local variance sum into every peer buffer, then spins until all rank contributions become non-sentinel. The global RMS factor divides by both per-rank hidden width and `NRanks`, reconstructing the full hidden dimension before applying gamma.
**CN:** 每个 rank 会把本地方差和写入所有对端缓冲区，然后自旋等待所有 rank 的贡献都从哨兵值变成有效值。最终的 RMS 系数同时除以单 rank hidden 宽度和 `NRanks`，从而恢复完整 hidden 维度下的均方值，再乘以 gamma。

### Optimized Q/K float4 kernel / 优化版 Q/K float4 内核
```cpp
template <typename DType, int NRanks, int OriginQDim, int OriginKDim>
__global__ void __launch_bounds__(1024)
    minimax_reduce_qk_rms_kernel_lamport_float4(MiniMaxReduceRMSParams params) {
  constexpr int RankQDim = OriginQDim / NRanks;
  constexpr int RankKDim = OriginKDim / NRanks;
  constexpr int ThreadsPerRowQ = RankQDim / kElemsPerAccess<DType>;
  constexpr int ThreadsPerRowK = RankKDim / kElemsPerAccess<DType>;
  constexpr int NumWarpQ = (ThreadsPerRowQ + MINIMAX_REDUCE_RMS_WARP_SIZE - 1) /
                           MINIMAX_REDUCE_RMS_WARP_SIZE;
  constexpr int NumWarpK = (ThreadsPerRowK + MINIMAX_REDUCE_RMS_WARP_SIZE - 1) /
                           MINIMAX_REDUCE_RMS_WARP_SIZE;
```
**EN:** The specialized kernel assumes MiniMax M2 dimensions (`6144` for Q, `1024` for K globally) and hardwires a float4-oriented mapping. It dedicates separate warp groups to Q and K so both can be normalized in one kernel launch over a fused QKV layout.
**CN:** 这个特化内核假设 MiniMax M2 的全局维度（Q 为 `6144`，K 为 `1024`），并采用面向 float4 的固定映射。它给 Q 和 K 分配独立的 warp 组，从而能在一次内核启动中对融合的 QKV 布局同时完成 Q/K 归一化。

### Group-of-4 processing and scale broadcast / 以 4 行为组处理并广播缩放
```cpp
for (int g = group_id; g < tot_groups; g += group_stride) {
  alignas(16) DType vals[4][kElemsPerAccess<DType>]{};
  float warp_sum_variance[4]{0.F, 0.F, 0.F, 0.F};
  ...
  local_warp_reduce_sum_array<MINIMAX_REDUCE_RMS_WARP_SIZE, float, 4>(
      warp_sum_variance);
  ...
  if (tid == 0) {
    *reinterpret_cast<float4*>(global_scale_q) =
        rms_rsqrt<OriginQDim>(var_all_ranks, params.rms_eps);
  }
```
**EN:** Instead of handling one token row at a time, the float4 kernel processes four rows as a bundle. That lets it all-reduce four variance sums together in one `float4`, improving communication packing and reducing per-token overhead.
**CN:** float4 内核不是一次只处理一行，而是把 4 行打成一组。这样就可以把 4 个方差和封装在一个 `float4` 里一起做 all-reduce，从而提升通信打包效率并降低单 token 开销。

### Launcher policy and dtype/rank dispatch / 启动策略与类型/rank 分发
```cpp
template <typename DType, int NRanks>
void minimax_reduce_rms_kernel_launcher(MiniMaxReduceRMSParams const& params) {
  static int SM = getSMVersion();
  int token_num = params.size_q / params.hidden_dim;
  int sm_count = get_sm_count();
  ...
  int max_blocks_per_sm = get_max_active_blocks(
      minimax_reduce_rms_kernel_lamport<DType, NRanks>, block_size);
```
**EN:** Launcher code computes occupancy-aware grid sizes, enables programmatic stream serialization, and optionally attaches cluster attributes for SM90+. This is not generic boilerplate: it is tuned to keep enough blocks resident while matching the communication pattern of the kernels.
**CN:** launcher 代码会根据占用率估计 grid 大小，启用 programmatic stream serialization，并在 SM90+ 上附加 cluster 属性。这不是通用样板，而是针对这些通信型内核的执行模式做过调整。

```cpp
bool use_float4 = (params.allreduce_in_k != nullptr) &&
                  (params.hidden_dim * params.nranks == 6144) &&
                  (params.hidden_dim_k * params.nranks == 1024);
```
**EN:** The float4 kernel is deliberately narrow in scope: it is only enabled when K exists and the full dimensions exactly match the known MiniMax M2 shape. Otherwise the implementation falls back to the more general scalar kernel.
**CN:** float4 内核的适用范围被刻意限制得很窄：只有在存在 K 输入，且完整维度精确匹配已知的 MiniMax M2 形状时才启用；否则退回更通用的标量内核。

### Torch extension wrappers / Torch 扩展包装函数
```cpp
torch::Tensor minimax_allreduce_rms(torch::Tensor const& input,
                                    torch::Tensor const& norm_weight,
                                    torch::Tensor workspace, int64_t const rank,
                                    int64_t const nranks, double const eps) {
  auto allreduce_params = vllm::tensorrt_llm::MiniMaxReduceRMSParams();
  ...
  torch::Tensor rms_norm_out = torch::empty_like(input);
  allreduce_params.rms_norm_out = rms_norm_out.mutable_data_ptr();
  vllm::tensorrt_llm::minimax_reduce_rms_op(allreduce_params);
  return rms_norm_out;
}
```
**EN:** The bottom-level wrappers translate ordinary torch tensors into `MiniMaxReduceRMSParams`, allocate output tensors, fill workspace/gamma/input pointers, and then call the internal dispatch entry point.
**CN:** 底部的包装函数会把普通 torch 张量转换成 `MiniMaxReduceRMSParams`，分配输出张量，填好 workspace/gamma/input 指针，然后调用内部统一分发入口。

## Key Concepts / 关键概念
- **EN:** This file implements RMSNorm across tensor-parallel ranks by explicitly exchanging local variance sums through a shared GPU workspace.  
  **CN:** 该文件通过共享 GPU workspace 显式交换各 rank 的局部方差和，从而实现跨张量并行 rank 的 RMSNorm。
- **EN:** Negative zero is used as a readiness marker, enabling simple polling without extra side buffers.  
  **CN:** 负零被用作“是否就绪”的标记，因此无需额外 side buffer 就能进行轮询同步。
- **EN:** There are two main execution modes: a general scalar kernel and a shape-specific float4 Q/K kernel specialized for MiniMax M2.  
  **CN:** 主要有两种执行模式：通用标量内核，以及针对 MiniMax M2 形状特化的 float4 Q/K 内核。
- **EN:** Host launch code is performance-sensitive because occupancy and cluster settings strongly affect this communication-heavy workload.  
  **CN:** 主机侧启动代码同样是性能关键点，因为占用率和 cluster 设置会显著影响这种通信密集型工作负载。

## Dependencies / 依赖关系
- **EN:** Depends on `minimax_reduce_rms_kernel.h` for parameter definitions and access-width traits.  
  **CN:** 依赖 `minimax_reduce_rms_kernel.h` 提供参数定义和访问宽度特征。
- **EN:** Uses CUDA cooperative groups, occupancy APIs, and low-level volatile loads/inline assembly.  
  **CN:** 使用 CUDA cooperative groups、occupancy API，以及低层的 volatile load/内联汇编。
- **EN:** Exposes `minimax_allreduce_rms` and `minimax_allreduce_rms_qk`, which are declared in `csrc/ops.h` and used by the torch extension layer.  
  **CN:** 对外暴露 `minimax_allreduce_rms` 与 `minimax_allreduce_rms_qk`，它们在 `csrc/ops.h` 中声明，并由 torch 扩展层使用。
