# custom_all_reduce_test.cu — Code Analysis / 代码分析

## Source / 来源

- **File**: `csrc/custom_all_reduce_test.cu`
- **Repository**: `vllm-project/vllm`
- **Purpose**: **EN:** Implements a standalone MPI/NCCL test and benchmark for the custom all-reduce kernels, including data generation, IPC setup, correctness checks, and timing. **CN:** 实现一个独立的 MPI/NCCL 测试与基准程序，用于验证自定义 all-reduce 内核，包括数据生成、IPC 初始化、正确性检查和性能计时。

## Line-by-Line Analysis / 逐行分析

### Test harness macros and warmup kernel / 测试框架宏与预热内核
```cpp
#define MPICHECK(cmd)                                                  \
  do {                                                                 \
    int e = cmd;                                                       \
    if (e != MPI_SUCCESS) {                                            \
      printf("Failed: MPI error %s:%d '%d'\n", __FILE__, __LINE__, e); \
      exit(EXIT_FAILURE);                                              \
    }                                                                  \
  } while (0)

#define NCCLCHECK(cmd)                                              \
  do {                                                              \
    ncclResult_t r = cmd;                                           \
    if (r != ncclSuccess) {                                         \
      printf("Failed, NCCL error %s:%d '%s'\n", __FILE__, __LINE__, \
             ncclGetErrorString(r));                                \
      exit(EXIT_FAILURE);                                           \
    }                                                               \
  } while (0)

#ifdef USE_ROCM
__global__ void dummy_kernel() {
  for (int i = 0; i < 100; i++) {
    uint64_t start = wall_clock64();
    uint64_t cycles_elapsed;
    do {
      cycles_elapsed = wall_clock64() - start;
    } while (cycles_elapsed < 100);
  }
  for (int i = 0; i < 100; i++) __nanosleep(1000000);  // 100ms
}
#else
__global__ void dummy_kernel() {
  #if defined(__CUDA_ARCH__) && __CUDA_ARCH__ >= 700
  for (int i = 0; i < 100; i++) __nanosleep(1000000);  // 100ms
  #else
  for (int i = 0; i < 100; i++) {
    long long int start = clock64();
    while (clock64() - start < 150000000);  // approximately 98.4ms on P40
  }
  #endif
}
#endif
```
**EN:** The MPICHECK/NCCLCHECK macros turn library failures into immediate process exits, and dummy_kernel creates background work so benchmark timing resembles real overlap scenarios instead of measuring an otherwise idle device.
**CN:** MPICHECK/NCCLCHECK 宏会把库调用失败直接升级为进程退出，而 dummy_kernel 会制造背景负载，使性能测试更接近真实重叠场景，而不是在完全空闲设备上测量。

### Randomized data generation kernels / 随机数据生成内核
```cpp
template <typename T>
__global__ void set_data(T* data, int size, int myRank) {
  for (int idx = blockIdx.x * blockDim.x + threadIdx.x; idx < size;
       idx += gridDim.x * blockDim.x) {
    data[idx] = myRank * 0.11f;
  }
}

template <typename T>
__global__ void convert_data(const T* data1, const T* data2, double* fdata1,
                             double* fdata2, int size) {
  for (int idx = blockIdx.x * blockDim.x + threadIdx.x; idx < size;
       idx += gridDim.x * blockDim.x) {
    fdata1[idx] = data1[idx];
    fdata2[idx] = data2[idx];
  }
}

__global__ void init_rand(curandState_t* state, int size, int nRanks) {
  for (int idx = blockIdx.x * blockDim.x + threadIdx.x; idx < size;
       idx += gridDim.x * blockDim.x) {
    for (int i = 0; i < nRanks; i++) {
      curand_init(i + 1, idx, 0, &state[idx * nRanks + i]);
    }
  }
}

template <typename T>
__global__ void gen_data(curandState_t* state, T* data, double* ground_truth,
                         int myRank, int nRanks, int size) {
  for (int idx = blockIdx.x * blockDim.x + threadIdx.x; idx < size;
       idx += gridDim.x * blockDim.x) {
    double sum = 0.0;
    for (int i = 0; i < nRanks; i++) {
      double val = curand_uniform_double(&state[idx * nRanks + i]) * 4;
      T hval = val;  // downcast first
      sum += static_cast<double>(hval);
      if (i == myRank) data[idx] = hval;
    }
    ground_truth[idx] = sum;
  }
}
```
**EN:** set_data, convert_data, init_rand, and gen_data generate per-rank inputs and a host-side fp64 ground truth. The test intentionally downcasts random values into the target dtype before summing so the reference reflects the same storage precision as the kernel input.
**CN:** set_data、convert_data、init_rand 和 gen_data 用来生成各 rank 的输入以及主机侧 fp64 真值。测试会先把随机值降精度到目标 dtype 再求和，这样参考结果与内核输入的存储精度保持一致。

### IPC setup and CustomAllreduce construction / IPC 初始化与 CustomAllreduce 构造
```cpp
template <typename T>
void run(int myRank, int nRanks, ncclComm_t& comm, int threads, int block_limit,
         int data_size, bool performance_test) {
  T* result;
  cudaStream_t stream;
  CUDACHECK(cudaStreamCreateWithFlags(&stream, cudaStreamNonBlocking));
  CUDACHECK(cudaMalloc(&result, data_size * sizeof(T)));
  CUDACHECK(cudaMemset(result, 0, data_size * sizeof(T)));

  cudaIpcMemHandle_t self_data_handle;
  cudaIpcMemHandle_t data_handles[8];
  vllm::Signal* buffer;
  T* self_data_copy;
  /**
   * Allocate IPC buffer
   *
   * The first section is a temporary buffer for storing intermediate allreduce
   * results, if a particular algorithm requires it. The second section is for
   * the input to the allreduce. The actual API takes the input pointer as an
   * argument (that is, they can and usually should be allocated separately).
   * But since the input pointers and the temporary buffer all require IPC
   * registration, they are allocated and registered together in the test for
   * convenience.
   */
#ifdef USE_ROCM
  CUDACHECK(hipExtMallocWithFlags(
      (void**)&buffer, 2 * data_size * sizeof(T) + sizeof(vllm::Signal),
      hipDeviceMallocUncached));
#else
  CUDACHECK(
      cudaMalloc(&buffer, 2 * data_size * sizeof(T) + sizeof(vllm::Signal)));
#endif
  CUDACHECK(
      cudaMemset(buffer, 0, 2 * data_size * sizeof(T) + sizeof(vllm::Signal)));
  CUDACHECK(cudaMalloc(&self_data_copy, data_size * sizeof(T)));
  CUDACHECK(cudaIpcGetMemHandle(&self_data_handle, buffer));

  MPICHECK(MPI_Allgather(&self_data_handle, sizeof(cudaIpcMemHandle_t),
                         MPI_BYTE, data_handles, sizeof(cudaIpcMemHandle_t),
                         MPI_BYTE, MPI_COMM_WORLD));

  void* rank_data;
  size_t rank_data_sz = 16 * 1024 * 1024;
  CUDACHECK(cudaMalloc(&rank_data, rank_data_sz));
  vllm::Signal* ipc_ptrs[8];
  for (int i = 0; i < nRanks; i++) {
    if (i == myRank)
      ipc_ptrs[i] = buffer;
    else
      CUDACHECK(cudaIpcOpenMemHandle((void**)&ipc_ptrs[i], data_handles[i],
                                     cudaIpcMemLazyEnablePeerAccess));
  }
  vllm::CustomAllreduce fa(ipc_ptrs, rank_data, rank_data_sz, myRank, nRanks);
  auto* self_data =
      reinterpret_cast<T*>(reinterpret_cast<char*>(buffer) +
                           sizeof(vllm::Signal) + data_size * sizeof(T));
  // hack buffer registration
  {
    void* data[8];
    for (int i = 0; i < nRanks; i++) {
      data[i] =
          ((char*)ipc_ptrs[i]) + sizeof(vllm::Signal) + data_size * sizeof(T);
    }
    fa.register_buffer(data);
  }
```
**EN:** run<T> allocates an IPC-visible buffer whose prefix stores Signal metadata and whose payload stores temporary/input data. MPI_Allgather exchanges cudaIpcMemHandle_t values so every rank can open every other rank's buffer before constructing CustomAllreduce and registering the input region.
**CN:** run<T> 会分配一个可通过 IPC 共享的缓冲区，其前缀存放 Signal 元数据，后续区域存放临时/输入数据。随后通过 MPI_Allgather 交换 cudaIpcMemHandle_t，使每个 rank 都能打开其它 rank 的缓冲区，再构造 CustomAllreduce 并注册输入区间。

### Performance and correctness comparison / 性能与正确性对比
```cpp
  ncclDataType_t ncclDtype;
  if (std::is_same<T, half>::value) {
    ncclDtype = ncclFloat16;
  } else if (std::is_same<T, nv_bfloat16>::value) {
    ncclDtype = ncclBfloat16;
  } else {
    ncclDtype = ncclFloat;
  }
  double *nccl_result, *my_result;
  CUDACHECK(cudaMallocHost(&nccl_result, data_size * sizeof(double)));
  CUDACHECK(cudaMallocHost(&my_result, data_size * sizeof(double)));
  if (performance_test) {
    dummy_kernel<<<1, 1, 0, stream>>>();
    constexpr int warmup_iters = 5;
    constexpr int num_iters = 100;
    // warmup
    for (int i = 0; i < warmup_iters; i++) {
      NCCLCHECK(ncclAllReduce(result, result, data_size, ncclDtype, ncclSum,
                              comm, stream));
    }
    CUDACHECK(cudaEventRecord(start, stream));
    for (int i = 0; i < num_iters; i++) {
      NCCLCHECK(ncclAllReduce(result, result, data_size, ncclDtype, ncclSum,
                              comm, stream));
    }
    CUDACHECK(cudaEventRecord(stop, stream));
    CUDACHECK(cudaStreamSynchronize(stream));
    float allreduce_ms = 0;
    cudaEventElapsedTime(&allreduce_ms, start, stop);

    dummy_kernel<<<1, 1, 0, stream>>>();
    // warm up
    for (int i = 0; i < warmup_iters; i++) {
      fa.allreduce<T>(stream, self_data, result, data_size, threads,
                      block_limit);
    }
    CUDACHECK(cudaEventRecord(start, stream));
    for (int i = 0; i < num_iters; i++) {
      fa.allreduce<T>(stream, self_data, result, data_size, threads,
                      block_limit);
    }
    CUDACHECK(cudaEventRecord(stop, stream));
    CUDACHECK(cudaStreamSynchronize(stream));

    float duration_ms = 0;
    cudaEventElapsedTime(&duration_ms, start, stop);
    if (myRank == 0)
      printf(
          "Rank %d done, nGPUs:%d, sz (kb): %d, %d, %d, my time:%.2fus, nccl "
          "time:%.2fus\n",
          myRank, nRanks, data_size * sizeof(T) / 1024, threads, block_limit,
          duration_ms * 1e3 / num_iters, allreduce_ms * 1e3 / num_iters);

    // And wait for all the queued up work to complete
    CUDACHECK(cudaStreamSynchronize(stream));

    NCCLCHECK(ncclAllReduce(self_data_copy, self_data, data_size, ncclDtype,
                            ncclSum, comm, stream));

    convert_data<T><<<108, 1024, 0, stream>>>(self_data, result, nccl_result,
                                              my_result, data_size);
    CUDACHECK(cudaStreamSynchronize(stream));

    for (unsigned long j = 0; j < data_size; j++) {
      auto diff = abs(nccl_result[j] - my_result[j]);
      if (diff >= 4e-2) {
        printf("Rank %d: Verification mismatch at %lld: %f != (my) %f, gt=%f\n",
               myRank, j, nccl_result[j], my_result[j], ground_truth[j]);
        break;
      }
    }
    long double nccl_diffs = 0.0;
    long double my_diffs = 0.0;
    for (int j = 0; j < data_size; j++) {
      nccl_diffs += abs(nccl_result[j] - ground_truth[j]);
      my_diffs += abs(my_result[j] - ground_truth[j]);
    }
    if (myRank == 0)
      std::cout << "average abs diffs: nccl: " << nccl_diffs / data_size
                << " me: " << my_diffs / data_size << std::endl;
  } else {
    for (int i = 0; i < 100; i++) {
      fa.allreduce<T>(stream, self_data, result, data_size, threads,
                      block_limit);
      CUDACHECK(cudaStreamSynchronize(stream));
      NCCLCHECK(ncclAllReduce(self_data, self_data_copy, data_size, ncclDtype,
                              ncclSum, comm, stream));
      convert_data<T><<<108, 1024, 0, stream>>>(
          self_data_copy, result, nccl_result, my_result, data_size);
      CUDACHECK(cudaStreamSynchronize(stream));

      for (unsigned long j = 0; j < data_size; j++) {
        auto diff = abs(nccl_result[j] - my_result[j]);
        if (diff >= 4e-2) {
          printf(
              "Rank %d: Verification mismatch at %lld: %f != (my) %f, gt=%f\n",
              myRank, j, nccl_result[j], my_result[j], ground_truth[j]);
          break;
        }
      }
    }
    if (myRank == 0)
      printf("Test passed: nGPUs:%d, sz (kb): %d, %d, %d\n", nRanks,
             data_size * sizeof(T) / 1024, threads, block_limit);
    // long double nccl_diffs = 0.0;
    // long double my_diffs = 0.0;
    // for (int j = 0; j < data_size; j++) {
    //   nccl_diffs += abs(nccl_result[j] - ground_truth[j]);
    //   my_diffs += abs(my_result[j] - ground_truth[j]);
    // }
    // if (myRank == 0)
    //   std::cout << "average abs diffs: nccl: " << nccl_diffs / data_size
    //             << " me: " << my_diffs / data_size << std::endl;
  }

  CUDACHECK(cudaFree(result));
  CUDACHECK(cudaFree(self_data_copy));
  CUDACHECK(cudaFree(rank_data));
  CUDACHECK(cudaFree(buffer));
  CUDACHECK(cudaFree(states));
  CUDACHECK(cudaFreeHost(ground_truth));
  CUDACHECK(cudaFreeHost(nccl_result));
  CUDACHECK(cudaFreeHost(my_result));
  CUDACHECK(cudaStreamDestroy(stream));
```
**EN:** The benchmark path measures NCCL and the custom kernel back-to-back on the same stream, then compares both outputs against one another and against the generated ground truth. The non-performance path repeatedly validates numerical agreement across many runs.
**CN:** 基准路径会在同一 stream 上先后测量 NCCL 与自定义内核，然后把两者输出互相比较，并与生成的真值做对照。非性能路径则在多轮运行中反复验证数值一致性。

### Program entry and size sweep / 程序入口与问题规模扫描
```cpp
int main(int argc, char** argv) {
  int nRanks, myRank;
  MPICHECK(MPI_Init(&argc, &argv));
  MPICHECK(MPI_Comm_rank(MPI_COMM_WORLD, &myRank));
  MPICHECK(MPI_Comm_size(MPI_COMM_WORLD, &nRanks));
  CUDACHECK(cudaSetDevice(myRank));
  ncclUniqueId id;
  ncclComm_t comm;
  if (myRank == 0) ncclGetUniqueId(&id);
  MPICHECK(MPI_Bcast(static_cast<void*>(&id), sizeof(id), MPI_BYTE, 0,
                     MPI_COMM_WORLD));
  NCCLCHECK(ncclCommInitRank(&comm, nRanks, id, myRank));

  bool performance_test = true;
  cudaProfilerStart();
// Uncomment to scan through different block size configs.
// for (int threads : {256, 512, 1024}) {
//   for (int block_limit = 16; block_limit < 112; block_limit += 4) {
//     run<half>(myRank, nRanks, comm, threads, block_limit, 1024 * 1024,
//     performance_test);
//   }
// }
#ifdef USE_ROCM
  const int block_limit = 16;
#else
  const int block_limit = 36;
#endif
  // Scan through different sizes to test performance.
  for (int sz = 512; sz <= (8 << 20); sz *= 2) {
    run<half>(myRank, nRanks, comm, 512, 36, sz + 8 * 47, performance_test);
  }

  cudaProfilerStop();
  MPICHECK(MPI_Finalize());
  return EXIT_SUCCESS;
```
**EN:** main initializes MPI and NCCL, binds each rank to one GPU, chooses a block limit based on platform, and sweeps message sizes from 512 elements up to 8<<20 to characterize scaling behavior.
**CN:** main 负责初始化 MPI 与 NCCL，把每个 rank 绑定到一张 GPU，根据平台选择 block limit，并把消息规模从 512 扫描到 8<<20，以观察扩展行为。

## Key Concepts / 关键概念

- **EN:** The test compares the custom implementation against NCCL, which acts as the de facto correctness/performance reference.
  **CN:** 测试把自定义实现与 NCCL 对照，NCCL 在这里充当事实上的正确性/性能参考基线。
- **EN:** Ground truth is accumulated in double precision even though inputs are half or bf16.
  **CN:** 尽管输入可能是 half 或 bf16，真值仍在 double 精度中累加。
- **EN:** MPI is only used for process coordination and IPC handle exchange; data movement happens on GPU buffers.
  **CN:** MPI 只负责进程协同与 IPC 句柄交换，真正的数据搬运发生在 GPU 缓冲区上。

## Dependencies / 依赖关系

- **EN:** Depends on MPI for rank discovery and handle exchange.
  **CN:** 依赖 MPI 完成 rank 发现与句柄交换。
- **EN:** Depends on NCCL or RCCL for the reference all-reduce implementation.
  **CN:** 依赖 NCCL 或 RCCL 作为参考 all-reduce 实现。
- **EN:** Includes csrc/custom_all_reduce.cuh to instantiate the custom runtime directly in C++.
  **CN:** 包含 csrc/custom_all_reduce.cuh，以便在 C++ 中直接实例化自定义运行时。
