# cuda_distributions_test.cu — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `aten/src/ATen/test/cuda_distributions_test.cu`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Contains ATen unit tests or installation checks that validate runtime behavior. This file specifically implements the logic associated with `cuda_distributions_test.cu`. It is structured around assertions and parameter sweeps rather than a production runtime path. Backend-specific integration details shape the API or implementation choices. Threading and execution-policy decisions are important to understanding the code.
- **Purpose (CN)**: 包含验证运行时行为的 ATen 单元测试或安装检查。 该文件具体实现与 `cuda_distributions_test.cu` 相关的逻辑。 它主要围绕断言与参数遍历组织，而不是直接提供生产运行时路径。 后端专用的集成细节会影响这里的 API 或实现选择。 线程与执行策略的决策是理解该文件的重要切入点。

## Line-by-Line Analysis / 逐行分析

### Lines 1-10 / 第 1-10 行

```cpp
#include <gtest/gtest.h>

#include <ATen/ATen.h>
#include <ATen/cuda/CUDAContext.h>
#include <ATen/native/cuda/Randperm.cuh>

#include <cuda.h>
#include <cuda_runtime.h>
#include <vector>

```

- **EN:** This range is mainly assembling the headers or imports needed by the declarations and runtime logic that follow.
- **CN:** 这一段主要在组织后续声明与运行时逻辑所需的头文件或导入项。
- **EN:** Backend-specific identifiers appear in this range, so the code is adapting ATen abstractions to a concrete device/runtime interface.
- **CN:** 这一段出现了后端专用标识符，说明代码在把 ATen 抽象适配到具体设备或运行时接口。
- **EN:** Concepts touched here: Backend interop / 后端互操作, Testing harness / 测试框架, Header composition / 头文件组织.
- **CN:** 这里涉及的概念包括：Backend interop / 后端互操作, Testing harness / 测试框架, Header composition / 头文件组织。

### Lines 11-26 / 第 11-26 行

```cpp
#include <curand.h>
#include <curand_kernel.h>
#include <curand_philox4x32_x.h>

__global__ void expected_uniforms(float* x, uint64_t counter_offset) {
  for(int i=0; i < 4; i++) {
    curandStatePhilox4_32_10_t state;
    curand_init(
            123,
            i,
            counter_offset,
            &state);
    auto ret = curand_uniform4(&state);
    x[i] = ret.x;
  }
}
```

- **EN:** Important callable entry points in this range include expected_uniforms, curand_init.
- **CN:** 这一段的重要可调用入口包括 expected_uniforms, curand_init。
- **EN:** Loop constructs iterate over dimensions, parameters, test sweeps, or kernel inputs in this portion.
- **CN:** 这一部分中的循环用于遍历维度、参数组合、测试扫面或内核输入。
- **EN:** Concepts touched here: Random generator state / 随机数生成器状态, Declared symbols / 声明符号, Header composition / 头文件组织, Iteration / 迭代处理.
- **CN:** 这里涉及的概念包括：Random generator state / 随机数生成器状态, Declared symbols / 声明符号, Header composition / 头文件组织, Iteration / 迭代处理。

### Lines 27-42 / 第 27-42 行

```cpp

/**
 * Helper function that asserts call to uniform_ starts from the correct
 * philox offset.
 *   - Get 4 randoms with counter_offset for thread {0,1,2,3} from expected_uniforms
 *     kernel above.
 *   - Now get 4 more randoms from uniform_ (note thread {0,1,2,3} for this call should
 *     start from a counter_offset value)
 *   - the 4 randoms from expected_uniforms kernel and the 4 randoms from the previous call
 *     of uniform_ should match, signifying that the philox offset was
 *     incremented properly and no randoms are being reused from previous calls
 */
void assert_with_expected_uniforms(uint64_t counter_offset) {
  // allocate 4 float on host memory
  float *x;
  cudaMallocManaged(&x, 4*sizeof(float));
```

- **EN:** Important callable entry points in this range include assert_with_expected_uniforms, cudaMallocManaged.
- **CN:** 这一段的重要可调用入口包括 assert_with_expected_uniforms, cudaMallocManaged。
- **EN:** Concurrency-related constructs appear here, showing how work is partitioned, synchronized, or scheduled.
- **CN:** 这里出现了并发相关结构，说明代码在处理任务划分、同步或调度。
- **EN:** Concepts touched here: Parallel runtime / 并行运行时, Random generator state / 随机数生成器状态, Backend interop / 后端互操作, Testing harness / 测试框架, Declared symbols / 声明符号.
- **CN:** 这里涉及的概念包括：Parallel runtime / 并行运行时, Random generator state / 随机数生成器状态, Backend interop / 后端互操作, Testing harness / 测试框架, Declared symbols / 声明符号。

### Lines 43-50 / 第 43-50 行

```cpp

  // launch kernel to get expected randoms
  expected_uniforms<<<1, 1>>>(x, counter_offset);
  C10_CUDA_KERNEL_LAUNCH_CHECK();

  // Wait for GPU to finish before accessing on host
  ASSERT_EQ(cudaSuccess, cudaDeviceSynchronize());

```

- **EN:** Backend-specific identifiers appear in this range, so the code is adapting ATen abstractions to a concrete device/runtime interface.
- **CN:** 这一段出现了后端专用标识符，说明代码在把 ATen 抽象适配到具体设备或运行时接口。
- **EN:** Concepts touched here: Random generator state / 随机数生成器状态, Backend interop / 后端互操作, Testing harness / 测试框架.
- **CN:** 这里涉及的概念包括：Random generator state / 随机数生成器状态, Backend interop / 后端互操作, Testing harness / 测试框架。

### Lines 51-60 / 第 51-60 行

```cpp
  // get 4 new float from uniform_()
  auto self = at::empty({4}, at::TensorOptions(at::kCUDA));
  self.uniform_();

  // check randoms from expected_uniforms kernel are equal to the randoms from the second
  // call of uniform_()
  for (int i = 0; i < 4; i++) {
    ASSERT_EQ(self[i].item().to<float>(), x[i]);
  }

```

- **EN:** Tensor shape, metadata, or storage concerns are visible here, indicating data-model rather than pure algorithmic plumbing.
- **CN:** 这里可以看到张量形状、元数据或存储相关处理，说明重点偏向数据模型而非纯算法封装。
- **EN:** Loop constructs iterate over dimensions, parameters, test sweeps, or kernel inputs in this portion.
- **CN:** 这一部分中的循环用于遍历维度、参数组合、测试扫面或内核输入。
- **EN:** Concepts touched here: Tensor metadata / 张量元数据, Random generator state / 随机数生成器状态, Backend interop / 后端互操作, Testing harness / 测试框架, Iteration / 迭代处理.
- **CN:** 这里涉及的概念包括：Tensor metadata / 张量元数据, Random generator state / 随机数生成器状态, Backend interop / 后端互操作, Testing harness / 测试框架, Iteration / 迭代处理。

### Lines 61-76 / 第 61-76 行

```cpp
  // Free memory
  cudaFree(x);
}

TEST(DistributionsTest, TestPhiloxIncrementSmallUniformTensor) {
  // Test Description:
  //   In Distributions.cu we mentioned that philox increment
  //   should be at least the number of curand() random numbers used in
  //   each thread. In this test, we make sure that uniform_ correctly
  //   increments philox and doesn't reuse randoms from previous calls
  //   for a small tensor size of 4.
  //    - We check that by first getting 4 randoms from uniform_.
  //      Once we get these 4 randoms, that would mean that philox counter for
  //      thread 0, 1, 2 and 3, was incremented by 4 (check calc_execution_policy
  //      function for details).
  //    - assert the call to uniform_ will start from counter_offset of 4
```

- **EN:** Important callable entry points in this range include cudaFree.
- **CN:** 这一段的重要可调用入口包括 cudaFree。
- **EN:** Test cases such as DistributionsTest exercise behavior variations or corner cases in this span.
- **CN:** DistributionsTest 等测试用例在这一段覆盖了行为变化与边界情况。
- **EN:** Tensor shape, metadata, or storage concerns are visible here, indicating data-model rather than pure algorithmic plumbing.
- **CN:** 这里可以看到张量形状、元数据或存储相关处理，说明重点偏向数据模型而非纯算法封装。
- **EN:** Concepts touched here: Tensor metadata / 张量元数据, Parallel runtime / 并行运行时, Random generator state / 随机数生成器状态, Backend interop / 后端互操作, Testing harness / 测试框架, Declared symbols / 声明符号.
- **CN:** 这里涉及的概念包括：Tensor metadata / 张量元数据, Parallel runtime / 并行运行时, Random generator state / 随机数生成器状态, Backend interop / 后端互操作, Testing harness / 测试框架, Declared symbols / 声明符号。

### Lines 77-86 / 第 77-86 行

```cpp

  // if cuda not available, return
  if (!at::cuda::is_available()) return;

  // manual seed to 123
  at::manual_seed(123);

  // get 4 randoms from uniform_(), philox offset is now incremented to 4 by this call
  at::empty({4}, at::TensorOptions(at::kCUDA)).uniform_();

```

- **EN:** Important callable entry points in this range include manual_seed.
- **CN:** 这一段的重要可调用入口包括 manual_seed。
- **EN:** Tensor shape, metadata, or storage concerns are visible here, indicating data-model rather than pure algorithmic plumbing.
- **CN:** 这里可以看到张量形状、元数据或存储相关处理，说明重点偏向数据模型而非纯算法封装。
- **EN:** Conditional branches separate feature checks, special cases, or fast-path decisions.
- **CN:** 条件分支用于区分特性检查、特殊情况或快路径决策。
- **EN:** Concepts touched here: Tensor metadata / 张量元数据, Random generator state / 随机数生成器状态, Backend interop / 后端互操作, Declared symbols / 声明符号, Branching logic / 分支逻辑.
- **CN:** 这里涉及的概念包括：Tensor metadata / 张量元数据, Random generator state / 随机数生成器状态, Backend interop / 后端互操作, Declared symbols / 声明符号, Branching logic / 分支逻辑。

### Lines 87-102 / 第 87-102 行

```cpp
  // expected uniforms will start from counter offset of 4
  assert_with_expected_uniforms(4);
}

TEST(DistributionsTest, TestPhiloxIncrementBigUniformTensor) {
  // Test Description:
  //   In Distributions.cu we mentioned that philox increment
  //   should be at least the number of curand() random numbers used in
  //   each thread. In this test, we make sure that uniform_ correctly
  //   increments philox and doesn't reuse randoms from previous calls
  //   for a big size tensor.
  //    - First of all, we come up with what the size of the big tensor
  //      should be for this test. Our goal is to show that when the uniform_
  //      kernel runs at full occupancy (i.e. when the number of elements is
  //      greater the number of threads launched), it hits the unroll loop in
  //      the uniform_ kernel.
```

- **EN:** Important callable entry points in this range include assert_with_expected_uniforms.
- **CN:** 这一段的重要可调用入口包括 assert_with_expected_uniforms。
- **EN:** Test cases such as DistributionsTest exercise behavior variations or corner cases in this span.
- **CN:** DistributionsTest 等测试用例在这一段覆盖了行为变化与边界情况。
- **EN:** Tensor shape, metadata, or storage concerns are visible here, indicating data-model rather than pure algorithmic plumbing.
- **CN:** 这里可以看到张量形状、元数据或存储相关处理，说明重点偏向数据模型而非纯算法封装。
- **EN:** Concepts touched here: Tensor metadata / 张量元数据, Tensor iteration / 张量迭代, Parallel runtime / 并行运行时, Random generator state / 随机数生成器状态, Testing harness / 测试框架, Declared symbols / 声明符号.
- **CN:** 这里涉及的概念包括：Tensor metadata / 张量元数据, Tensor iteration / 张量迭代, Parallel runtime / 并行运行时, Random generator state / 随机数生成器状态, Testing harness / 测试框架, Declared symbols / 声明符号。

### Lines 103-112 / 第 103-112 行

```cpp
  //    - Hence, we set the size of the tensor in this test to be 8 times the
  //      maximum number of threads we can launch. This means that, each thread
  //      will be yielding 8 elements, and as a result, curand_uniform4 will be
  //      called twice and all the 8 elements in a thread will consume all the
  //      float4 from the two calls of curand_uniform4 as a result of the unroll
  //      loop. Therefore, after this call to the uniform_, counter_offset for
  //      the next call to uniform_ will start from 8. This is what we test
  //      next.
  //    - assert that call to uniform_ will start from counter_offset of 8

```

- **EN:** Tensor shape, metadata, or storage concerns are visible here, indicating data-model rather than pure algorithmic plumbing.
- **CN:** 这里可以看到张量形状、元数据或存储相关处理，说明重点偏向数据模型而非纯算法封装。
- **EN:** Concepts touched here: Tensor metadata / 张量元数据, Tensor iteration / 张量迭代, Parallel runtime / 并行运行时.
- **CN:** 这里涉及的概念包括：Tensor metadata / 张量元数据, Tensor iteration / 张量迭代, Parallel runtime / 并行运行时。

### Lines 113-125 / 第 113-125 行

```cpp
  // if cuda not available, return
  if (!at::cuda::is_available()) return;

  // manual seed to 123
  at::manual_seed(123);

  // calculate maximum number of threads that can be launched
  // and set the numel to be 8 times that
  const int block_size = 256;
  uint32_t blocks_per_sm = at::cuda::getCurrentDeviceProperties()->maxThreadsPerMultiProcessor / block_size;
  dim3 grid(static_cast<uint32_t>(at::cuda::getCurrentDeviceProperties()->multiProcessorCount) * blocks_per_sm);
  auto numel = block_size * grid.x * 8;

```

- **EN:** Important callable entry points in this range include manual_seed, grid.
- **CN:** 这一段的重要可调用入口包括 manual_seed, grid。
- **EN:** Concurrency-related constructs appear here, showing how work is partitioned, synchronized, or scheduled.
- **CN:** 这里出现了并发相关结构，说明代码在处理任务划分、同步或调度。
- **EN:** Conditional branches separate feature checks, special cases, or fast-path decisions.
- **CN:** 条件分支用于区分特性检查、特殊情况或快路径决策。
- **EN:** Concepts touched here: Tensor metadata / 张量元数据, Parallel runtime / 并行运行时, Random generator state / 随机数生成器状态, Backend interop / 后端互操作, Declared symbols / 声明符号, Branching logic / 分支逻辑.
- **CN:** 这里涉及的概念包括：Tensor metadata / 张量元数据, Parallel runtime / 并行运行时, Random generator state / 随机数生成器状态, Backend interop / 后端互操作, Declared symbols / 声明符号, Branching logic / 分支逻辑。

### Lines 126-133 / 第 126-133 行

```cpp
  // get numel randoms from uniform_(), philox offset is now incremented to 8 by this call
  at::empty({numel}, at::TensorOptions(at::kCUDA)).uniform_();

  // expected uniforms will start from counter offset of 8
  assert_with_expected_uniforms(8);
}

TEST(DistributionsTest, TestPhiloxIncrementSmallMultinomialTensor) {
```

- **EN:** Important callable entry points in this range include assert_with_expected_uniforms.
- **CN:** 这一段的重要可调用入口包括 assert_with_expected_uniforms。
- **EN:** Test cases such as DistributionsTest exercise behavior variations or corner cases in this span.
- **CN:** DistributionsTest 等测试用例在这一段覆盖了行为变化与边界情况。
- **EN:** Tensor shape, metadata, or storage concerns are visible here, indicating data-model rather than pure algorithmic plumbing.
- **CN:** 这里可以看到张量形状、元数据或存储相关处理，说明重点偏向数据模型而非纯算法封装。
- **EN:** Concepts touched here: Tensor metadata / 张量元数据, Random generator state / 随机数生成器状态, Backend interop / 后端互操作, Testing harness / 测试框架, Declared symbols / 声明符号, Test coverage / 测试覆盖.
- **CN:** 这里涉及的概念包括：Tensor metadata / 张量元数据, Random generator state / 随机数生成器状态, Backend interop / 后端互操作, Testing harness / 测试框架, Declared symbols / 声明符号, Test coverage / 测试覆盖。

### Lines 134-143 / 第 134-143 行

```cpp
  // Test Description:
  //   Same concept as TestPhiloxIncrementSmallUniformTensor.
  //   Multinomial increments offset by 4. Tests if uniform starts from the correct offset.

  // if cuda not available, return
  if (!at::cuda::is_available()) return;

  // manual seed to 123
  at::manual_seed(123);

```

- **EN:** Important callable entry points in this range include manual_seed.
- **CN:** 这一段的重要可调用入口包括 manual_seed。
- **EN:** Tensor shape, metadata, or storage concerns are visible here, indicating data-model rather than pure algorithmic plumbing.
- **CN:** 这里可以看到张量形状、元数据或存储相关处理，说明重点偏向数据模型而非纯算法封装。
- **EN:** Conditional branches separate feature checks, special cases, or fast-path decisions.
- **CN:** 条件分支用于区分特性检查、特殊情况或快路径决策。
- **EN:** Concepts touched here: Tensor metadata / 张量元数据, Random generator state / 随机数生成器状态, Backend interop / 后端互操作, Declared symbols / 声明符号, Branching logic / 分支逻辑.
- **CN:** 这里涉及的概念包括：Tensor metadata / 张量元数据, Random generator state / 随机数生成器状态, Backend interop / 后端互操作, Declared symbols / 声明符号, Branching logic / 分支逻辑。

### Lines 144-153 / 第 144-153 行

```cpp
  // get some multinomial samples
  // this will trigger torch.multinomial without replacement
  // which utilizes uniform which increments counter by 4.
  // num_samples in the following call is 4.
  at::ones({4}, at::TensorOptions(at::kCUDA)).multinomial(4);

  // expected uniforms will start from counter offset of 4
  assert_with_expected_uniforms(4);
}

```

- **EN:** Important callable entry points in this range include assert_with_expected_uniforms.
- **CN:** 这一段的重要可调用入口包括 assert_with_expected_uniforms。
- **EN:** Tensor shape, metadata, or storage concerns are visible here, indicating data-model rather than pure algorithmic plumbing.
- **CN:** 这里可以看到张量形状、元数据或存储相关处理，说明重点偏向数据模型而非纯算法封装。
- **EN:** Concepts touched here: Tensor metadata / 张量元数据, Backend interop / 后端互操作, Testing harness / 测试框架, Declared symbols / 声明符号.
- **CN:** 这里涉及的概念包括：Tensor metadata / 张量元数据, Backend interop / 后端互操作, Testing harness / 测试框架, Declared symbols / 声明符号。

### Lines 154-163 / 第 154-163 行

```cpp
__managed__ int keys[] = {
  1, (1 << 15) + 1,  (1 << 16) + 1,
  2, (1 << 14) + 2, 2
};

__managed__ int values[] = { 1, 2, 3, 4, 5, 9999 };

std::vector<std::vector<int>> valid_perms1 = {
  {1, 2, 3}, {1, 3, 2}, {2, 1, 3}, {2, 3, 1}, {3, 1, 2}, {3, 2, 1}
};
```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or data movement.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或数据传递。

### Lines 164-171 / 第 164-171 行

```cpp
std::vector<std::vector<int>> valid_perms2 = {
  {4, 5}, {5, 4}
};

TEST(RandomPermutationTest, TestIslandShuffle) {
  if (!at::cuda::is_available()) return;
  at::manual_seed(123);

```

- **EN:** Important callable entry points in this range include manual_seed.
- **CN:** 这一段的重要可调用入口包括 manual_seed。
- **EN:** Test cases such as RandomPermutationTest exercise behavior variations or corner cases in this span.
- **CN:** RandomPermutationTest 等测试用例在这一段覆盖了行为变化与边界情况。
- **EN:** Backend-specific identifiers appear in this range, so the code is adapting ATen abstractions to a concrete device/runtime interface.
- **CN:** 这一段出现了后端专用标识符，说明代码在把 ATen 抽象适配到具体设备或运行时接口。
- **EN:** Conditional branches separate feature checks, special cases, or fast-path decisions.
- **CN:** 条件分支用于区分特性检查、特殊情况或快路径决策。

### Lines 172-181 / 第 172-181 行

```cpp
  bool shuffled1 = false;
  bool shuffled2 = false;
  for (int i = 0; i < 100; i++) {
    ASSERT_EQ(cudaSuccess, cudaDeviceSynchronize());
    std::optional<at::Generator> gen = std::nullopt;
    randperm_handle_duplicate_keys(keys, values, 8, 5, gen);
    ASSERT_EQ(cudaSuccess, cudaDeviceSynchronize());
    std::vector<int> slice1 = {values[0], values[1], values[2]};
    std::vector<int> slice2 = {values[3], values[4]};
    if (slice1 != valid_perms1[0]) {
```

- **EN:** Important callable entry points in this range include randperm_handle_duplicate_keys.
- **CN:** 这一段的重要可调用入口包括 randperm_handle_duplicate_keys。
- **EN:** Backend-specific identifiers appear in this range, so the code is adapting ATen abstractions to a concrete device/runtime interface.
- **CN:** 这一段出现了后端专用标识符，说明代码在把 ATen 抽象适配到具体设备或运行时接口。
- **EN:** Conditional branches separate feature checks, special cases, or fast-path decisions.
- **CN:** 条件分支用于区分特性检查、特殊情况或快路径决策。
- **EN:** Loop constructs iterate over dimensions, parameters, test sweeps, or kernel inputs in this portion.
- **CN:** 这一部分中的循环用于遍历维度、参数组合、测试扫面或内核输入。

### Lines 182-189 / 第 182-189 行

```cpp
      shuffled1 = true;
    }
    if (slice2 != valid_perms2[0]) {
      shuffled2 = true;
    }
    bool passed1 = false;
    bool passed2 = false;
    for (auto &i : valid_perms1) {
```

- **EN:** Conditional branches separate feature checks, special cases, or fast-path decisions.
- **CN:** 条件分支用于区分特性检查、特殊情况或快路径决策。
- **EN:** Loop constructs iterate over dimensions, parameters, test sweeps, or kernel inputs in this portion.
- **CN:** 这一部分中的循环用于遍历维度、参数组合、测试扫面或内核输入。
- **EN:** Concepts touched here: Branching logic / 分支逻辑, Iteration / 迭代处理.
- **CN:** 这里涉及的概念包括：Branching logic / 分支逻辑, Iteration / 迭代处理。

### Lines 190-205 / 第 190-205 行

```cpp
      if (i == slice1) {
        passed1 = true;
        break;
      }
    }
    for (auto &i : valid_perms2) {
      if (i == slice2) {
        passed2 = true;
        break;
      }
    }
    ASSERT_TRUE(passed1);
    ASSERT_TRUE(passed2);
  }
  ASSERT_TRUE(shuffled1);
  ASSERT_TRUE(shuffled2);
```

- **EN:** Conditional branches separate feature checks, special cases, or fast-path decisions.
- **CN:** 条件分支用于区分特性检查、特殊情况或快路径决策。
- **EN:** Loop constructs iterate over dimensions, parameters, test sweeps, or kernel inputs in this portion.
- **CN:** 这一部分中的循环用于遍历维度、参数组合、测试扫面或内核输入。
- **EN:** Concepts touched here: Testing harness / 测试框架, Branching logic / 分支逻辑, Iteration / 迭代处理.
- **CN:** 这里涉及的概念包括：Testing harness / 测试框架, Branching logic / 分支逻辑, Iteration / 迭代处理。

### Lines 206-206 / 第 206-206 行

```cpp
}
```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or data movement.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或数据传递。

## Key Concepts / 关键概念

- **ATen test harness** — ATen 测试框架
- **Tensor metadata** — 张量元数据
- **Tensor iteration** — 张量迭代
- **Parallel runtime** — 并行运行时
- **Random generator state** — 随机数生成器状态
- **Backend interop** — 后端互操作
- **Testing harness** — 测试框架
- **Core symbols: expected_uniforms, curand_init, assert_with_expected_uniforms, cudaMallocManaged, cudaFree, manual_seed, grid, randperm_handle_duplicate_keys** — 核心符号：expected_uniforms、curand_init、assert_with_expected_uniforms、cudaMallocManaged、cudaFree、manual_seed、grid、randperm_handle_duplicate_keys

## Dependencies / 依赖关系

- `gtest/gtest.h`
- `ATen/ATen.h`
- `ATen/cuda/CUDAContext.h`
- `ATen/native/cuda/Randperm.cuh`
- `cuda.h`
- `cuda_runtime.h`
- `vector`
- `curand.h`
- `curand_kernel.h`
- `curand_philox4x32_x.h`
