# fused_kernel.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/csrc/jit/codegen/fuser/cuda/fused_kernel.cpp`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Contains code-generation logic or generated declarations that synthesize JIT-related bindings, schemas, or helper code. This specific file centers on `fused_kernel.cpp`. Generation of declarations, registrations, or helper code is part of the responsibility.
- **Purpose (CN)**: 包含代码生成逻辑或生成后的声明，用于产出 JIT 相关绑定、模式或辅助代码。 该文件具体围绕 `fused_kernel.cpp` 展开。 其职责包含声明、注册或辅助代码的生成。

## Line-by-Line Analysis / 逐行分析

### Lines 1-28 / 第 1-28 行

```cpp
#include <torch/csrc/jit/codegen/fuser/cuda/fused_kernel.h>

#include <torch/csrc/jit/codegen/fuser/compiler.h>

#include <ATen/ATen.h>
#include <ATen/cuda/CUDAContext.h>
#include <ATen/cuda/CUDAGeneratorImpl.h>
#include <ATen/cuda/nvrtc_stub/ATenNVRTC.h>
#include <ATen/native/cuda/jit_utils.h>
#include <c10/cuda/CUDAGuard.h>
#include <c10/util/Exception.h>
#include <torch/csrc/jit/resource_guard.h>

#include <cuda_runtime.h>

#include <algorithm>
#include <cmath>
#include <sstream>
#include <stdexcept>
#include <vector>

namespace torch::jit::fuser::cuda {

// See NOTE [ USE OF NVRTC AND DRIVER API ]
const at::cuda::NVRTC& nvrtc() {
  return at::globalContext().getNVRTC();
}

```

- **EN:** This range collects the headers or imports needed by the following declarations and implementation blocks.
- **CN:** 这一段集中引入后续声明与实现所需的头文件或模块。
- **EN:** It enters or references namespace scopes such as torch::jit::fuser::cuda, which positions the code inside the JIT subsystem hierarchy.
- **CN:** 这里进入或引用了 torch::jit::fuser::cuda 等命名空间作用域，用于把代码放入 JIT 子系统层级中。
- **EN:** Important callable entry points in this range include nvrtc, globalContext.
- **CN:** 这一段的重要可调用入口包括 nvrtc, globalContext。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。

### Lines 29-56 / 第 29-56 行

```cpp
// query codegen output arch and target
void codegenOutputQuery(
    const cudaDeviceProp* const prop,
    int& major,
    int& minor,
    bool& compile_to_sass) {
#ifdef USE_ROCM
  AT_CUDA_NVRTC_CHECK(nvrtc().nvrtcVersion(&major, &minor));
  compile_to_sass = false;
#else
  using CudaVersion = std::pair<int, int>;
  CudaVersion nvrtc_version;
  AT_CUDA_NVRTC_CHECK(
      nvrtc().nvrtcVersion(&nvrtc_version.first, &nvrtc_version.second));

  TORCH_CHECK(
      nvrtc_version.first >= 6,
      "NVRTC versions less than 6 are not supported. Is: ",
      nvrtc_version.first);

  // Version supported by device
  // Usually any lower version works too but is less efficient
  const CudaVersion dev_version = CudaVersion(prop->major, prop->minor);
  // Maximum version supported by the driver, cap dev_version to this
  CudaVersion max_dev_version;
  if (nvrtc_version.first <= 7) { // 7 supports 2-5.x
    max_dev_version = CudaVersion(5, 0);
  } else if (nvrtc_version.first <= 8) { // 8 supports 2-6.x
```

- **EN:** Alias declarations such as CudaVersion simplify later API usage.
- **CN:** CudaVersion 等别名声明简化了后续 API 的使用。
- **EN:** Important callable entry points in this range include codegenOutputQuery, AT_CUDA_NVRTC_CHECK, TORCH_CHECK.
- **CN:** 这一段的重要可调用入口包括 codegenOutputQuery, AT_CUDA_NVRTC_CHECK, TORCH_CHECK。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Concepts touched here: Nodes and values / 节点与值, Backend integration / 后端集成, Code generation / 代码生成, Declared symbols / 声明的符号, Macro control flow / 宏控制流, Branching logic / 分支逻辑.
- **CN:** 这里涉及的概念包括：Nodes and values / 节点与值, Backend integration / 后端集成, Code generation / 代码生成, Declared symbols / 声明的符号, Macro control flow / 宏控制流, Branching logic / 分支逻辑。

### Lines 57-84 / 第 57-84 行

```cpp
    max_dev_version = CudaVersion(6, 0);
  } else if (nvrtc_version.first <= 9) { // 9 supports 3-7.2
    max_dev_version = CudaVersion(7, 2);
  } else if (nvrtc_version.first <= 10) { // 10 supports 3-7.5
    max_dev_version = CudaVersion(7, 5);
  } else if (nvrtc_version == CudaVersion(11, 0)) { // 11.0 supports 3-8.0
    max_dev_version = CudaVersion(8, 0);
  } else if (nvrtc_version.first == 11 && nvrtc_version.second < 8) {
    max_dev_version = CudaVersion(8, 6);
  } else {
    // If the driver version is unknown (i.e. newer than this code)
    // assume the driver supports this device
    max_dev_version = dev_version;
  }
  if (dev_version > max_dev_version) {
    major = max_dev_version.first;
    minor = max_dev_version.second;
    // if we are clamping major/minor, sass is not compatible
    compile_to_sass = false;
  } else {
    major = dev_version.first;
    minor = dev_version.second;
    compile_to_sass = true;
  }
#endif
}

// Compiles the specified kernel and stores the metadata required to run it
```

- **EN:** Comments in this range document the intent of the API or the invariants expected by later code.
- **CN:** 这一段中的注释说明了 API 意图，或描述了后续代码依赖的不变式。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Concepts touched here: Backend integration / 后端集成, Macro control flow / 宏控制流, Branching logic / 分支逻辑.
- **CN:** 这里涉及的概念包括：Backend integration / 后端集成, Macro control flow / 宏控制流, Branching logic / 分支逻辑。

### Lines 85-112 / 第 85-112 行

```cpp
FusedKernelCUDA::FusedKernelCUDA(
    at::DeviceIndex device,
    std::string name,
    std::string code,
    std::vector<TensorDesc> input_desc,
    std::vector<TensorDesc> output_desc,
    std::vector<PartitionDesc> chunk_desc,
    std::vector<PartitionDesc> concat_desc,
    bool has_random)
    : FusedKernel(
          std::move(name),
          std::move(code),
          std::move(input_desc),
          std::move(output_desc),
          std::move(chunk_desc),
          std::move(concat_desc),
          has_random),
      device_(device) {
  // Initializes driver's API context (if necessary)
  at::cuda::jit::initializeCudaContext();

  // Note: hacked at::DeviceGuard since at::DeviceGuard was failing to work
  // properly in some scenarios
  const auto prior_device = at::cuda::current_device();
  at::cuda::set_device(device_);

  // Acquires device and NVRTC properties (for compile arch and occupancy
  // calculations)
```

- **EN:** Important callable entry points in this range include FusedKernelCUDA, initializeCudaContext, set_device.
- **CN:** 这一段的重要可调用入口包括 FusedKernelCUDA, initializeCudaContext, set_device。
- **EN:** Comments in this range document the intent of the API or the invariants expected by later code.
- **CN:** 这一段中的注释说明了 API 意图，或描述了后续代码依赖的不变式。
- **EN:** Concepts touched here: Nodes and values / 节点与值, Backend integration / 后端集成, Shape/resource guard / 形状或资源保护, Declared symbols / 声明的符号.
- **CN:** 这里涉及的概念包括：Nodes and values / 节点与值, Backend integration / 后端集成, Shape/resource guard / 形状或资源保护, Declared symbols / 声明的符号。

### Lines 113-140 / 第 113-140 行

```cpp
  // NOLINTNEXTLINE(cppcoreguidelines-prefer-member-initializer)
  prop_ = at::cuda::getCurrentDeviceProperties();
  int major = 0, minor = 0;
  bool compile_to_sass = false;
  codegenOutputQuery(prop_, major, minor, compile_to_sass);

  // Creates the NVRTC program
  nvrtcProgram program{};
  AT_CUDA_NVRTC_CHECK(nvrtc().nvrtcCreateProgram(
      &program, code_.c_str(), nullptr, 0, nullptr, nullptr));

#if defined(USE_ROCM)
  std::vector<const char*> args = {"--std=c++20"};
  args.push_back("-hip-pch");
#else
  const std::string compute = std::string("--gpu-architecture=") +
#if !defined(USE_ROCM)
      // CUDA 11.1 allows going directly to SASS (sm_) instead of PTX (compute_)
      // which gives better backwards compatibility to work on older driver,
      // (since older driver doesn't necessrily recognize PTX emitted by new
      // toolkit);
      // Meanwhile, for forward compatibility (future device with
      // `compile_to_sass==false`), since SASS are not necessarily compatible,
      // we fallback to PTX instead.
      (compile_to_sass ? "sm_" : "compute_") +
#else
      "compute_" +
#endif
```

- **EN:** Important callable entry points in this range include codegenOutputQuery, AT_CUDA_NVRTC_CHECK.
- **CN:** 这一段的重要可调用入口包括 codegenOutputQuery, AT_CUDA_NVRTC_CHECK。
- **EN:** Comments in this range document the intent of the API or the invariants expected by later code.
- **CN:** 这一段中的注释说明了 API 意图，或描述了后续代码依赖的不变式。
- **EN:** Concepts touched here: Nodes and values / 节点与值, Backend integration / 后端集成, Code generation / 代码生成, Declared symbols / 声明的符号, Macro control flow / 宏控制流.
- **CN:** 这里涉及的概念包括：Nodes and values / 节点与值, Backend integration / 后端集成, Code generation / 代码生成, Declared symbols / 声明的符号, Macro control flow / 宏控制流。

### Lines 141-168 / 第 141-168 行

```cpp
      std::to_string(major) + std::to_string(minor);
  const std::vector<const char*> args = {
      "--std=c++20", compute.c_str(), "-default-device"};
#endif
  const auto result =
      nvrtc().nvrtcCompileProgram(program, args.size(), args.data());
  if (result != NVRTC_SUCCESS) {
    size_t logsize = 0;
    AT_CUDA_NVRTC_CHECK(nvrtc().nvrtcGetProgramLogSize(program, &logsize));
    std::vector<char> log(logsize);
    AT_CUDA_NVRTC_CHECK(nvrtc().nvrtcGetProgramLog(program, log.data()));
    TORCH_CHECK(false, std::string(log.data(), log.size()));
  }
  ResourceGuard holdProgram(
      [&] { AT_CUDA_NVRTC_CHECK(nvrtc().nvrtcDestroyProgram(&program)); });
  AT_CUDA_NVRTC_CHECK(result);
  size_t ptx_size = 0;
#if !defined(USE_ROCM)
  // compile_to_sass determines whether we are generating SASS or PTX, hence
  // the different API.
  const auto getSize = compile_to_sass
      ? at::globalContext().getNVRTC().nvrtcGetCUBINSize
      : at::globalContext().getNVRTC().nvrtcGetPTXSize;
  const auto getFunc = compile_to_sass
      ? at::globalContext().getNVRTC().nvrtcGetCUBIN
      : at::globalContext().getNVRTC().nvrtcGetPTX;
#else
  const auto getSize = at::globalContext().getNVRTC().nvrtcGetPTXSize;
```

- **EN:** Important callable entry points in this range include to_string, nvrtc, AT_CUDA_NVRTC_CHECK, log, TORCH_CHECK.
- **CN:** 这一段的重要可调用入口包括 to_string, nvrtc, AT_CUDA_NVRTC_CHECK, log, TORCH_CHECK。
- **EN:** Comments in this range document the intent of the API or the invariants expected by later code.
- **CN:** 这一段中的注释说明了 API 意图，或描述了后续代码依赖的不变式。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Concepts touched here: Nodes and values / 节点与值, Backend integration / 后端集成, Shape/resource guard / 形状或资源保护, Declared symbols / 声明的符号, Macro control flow / 宏控制流, Branching logic / 分支逻辑.
- **CN:** 这里涉及的概念包括：Nodes and values / 节点与值, Backend integration / 后端集成, Shape/resource guard / 形状或资源保护, Declared symbols / 声明的符号, Macro control flow / 宏控制流, Branching logic / 分支逻辑。

### Lines 169-196 / 第 169-196 行

```cpp
  const auto getFunc = at::globalContext().getNVRTC().nvrtcGetPTX;
#endif
  AT_CUDA_NVRTC_CHECK(getSize(program, &ptx_size));
  ptx_.resize(ptx_size);
  AT_CUDA_NVRTC_CHECK(getFunc(program, ptx_.data()));

  AT_CUDA_DRIVER_CHECK(nvrtc().cuModuleLoadData(&module_, ptx_.data()));
  AT_CUDA_DRIVER_CHECK(
      nvrtc().cuModuleGetFunction(&function_, module_, name_.c_str()));

  // Computes max blocks
  AT_CUDA_DRIVER_CHECK(nvrtc().cuOccupancyMaxActiveBlocksPerMultiprocessor(
      &maxBlocks_, function_, 128, 0));
  maxBlocks_ *= prop_->multiProcessorCount;

  // Resets device (end of hacked at::DeviceGuard)
  at::cuda::set_device(prior_device);
}

static int ceilDiv(const int a, const int b) {
  return (a + b - 1) / b;
}

void FusedKernelCUDA::launch_raw(
    const uint32_t numel,
    std::vector<void*>& arguments) const {
  at::cuda::CUDAGuard guard{device_};
  // Hacked at::DeviceGuard (see note above)
```

- **EN:** Important callable entry points in this range include AT_CUDA_NVRTC_CHECK, AT_CUDA_DRIVER_CHECK, set_device, ceilDiv, launch_raw.
- **CN:** 这一段的重要可调用入口包括 AT_CUDA_NVRTC_CHECK, AT_CUDA_DRIVER_CHECK, set_device, ceilDiv, launch_raw。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。
- **EN:** Concepts touched here: Nodes and values / 节点与值, Control-flow blocks / 控制流块, Operator schema / 算子模式, Module API / 模块 API, Shape/resource guard / 形状或资源保护, Declared symbols / 声明的符号.
- **CN:** 这里涉及的概念包括：Nodes and values / 节点与值, Control-flow blocks / 控制流块, Operator schema / 算子模式, Module API / 模块 API, Shape/resource guard / 形状或资源保护, Declared symbols / 声明的符号。

### Lines 197-224 / 第 197-224 行

```cpp
  const auto prior_device = at::cuda::current_device();
  at::cuda::set_device(device_);

  const auto nBlocks = std::min(maxBlocks_, ceilDiv(numel, kBlockSize));

  // Adds random state to arguments if necessary
  // Note: philox_engine_inputs defined here so its lifetime extends to the
  // launch
  std::pair<uint64_t, uint64_t> philox_engine_inputs;
  if (has_random_) {
    const auto rand_offset =
        4 * (std::ceil(numel / (4.0 * kBlockSize * nBlocks)) + 1);
    auto gen = at::cuda::detail::getDefaultCUDAGenerator();
    {
      // See Note [Acquire lock when using random generators]
      std::lock_guard<std::mutex> lock(gen.mutex());
      philox_engine_inputs =
          at::check_generator<at::CUDAGeneratorImpl>(gen)->philox_engine_inputs(
              rand_offset);
    }
    arguments.push_back(&philox_engine_inputs.first);
    arguments.push_back(&philox_engine_inputs.second);
  }

  // Launches kernel on current stream (device was set by executor)
  auto stream = at::cuda::getCurrentCUDAStream();
  AT_CUDA_DRIVER_CHECK(nvrtc().cuLaunchKernel(
      function_,
```

- **EN:** Important callable entry points in this range include set_device, lock.
- **CN:** 这一段的重要可调用入口包括 set_device, lock。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Concepts touched here: Control-flow blocks / 控制流块, Operator schema / 算子模式, Shape/resource guard / 形状或资源保护, Branching logic / 分支逻辑.
- **CN:** 这里涉及的概念包括：Control-flow blocks / 控制流块, Operator schema / 算子模式, Shape/resource guard / 形状或资源保护, Branching logic / 分支逻辑。

### Lines 225-252 / 第 225-252 行

```cpp
      nBlocks,
      1,
      1,
      kBlockSize,
      1,
      1,
      0,
      stream,
      arguments.data(),
      nullptr));

  // Resets device (see at::DeviceGuard notes above)
  at::cuda::set_device(prior_device);
}

FusedKernelCUDA::~FusedKernelCUDA() {
  AT_CUDA_DRIVER_CHECK(nvrtc().cuModuleUnload(module_));
}

static std::shared_ptr<FusedKernel> createFusionKernel(
    int16_t device,
    std::string name,
    std::string code,
    std::vector<TensorDesc> input_desc,
    std::vector<TensorDesc> output_desc,
    std::vector<PartitionDesc> chunk_desc,
    std::vector<PartitionDesc> concat_desc,
    bool has_random) {
```

- **EN:** Important callable entry points in this range include set_device, AT_CUDA_DRIVER_CHECK, createFusionKernel.
- **CN:** 这一段的重要可调用入口包括 set_device, AT_CUDA_DRIVER_CHECK, createFusionKernel。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Concepts touched here: Nodes and values / 节点与值, Control-flow blocks / 控制流块, Operator schema / 算子模式, Module API / 模块 API, Optimization pass / 优化 pass, Shape/resource guard / 形状或资源保护.
- **CN:** 这里涉及的概念包括：Nodes and values / 节点与值, Control-flow blocks / 控制流块, Operator schema / 算子模式, Module API / 模块 API, Optimization pass / 优化 pass, Shape/resource guard / 形状或资源保护。

### Lines 253-266 / 第 253-266 行

```cpp
  return std::make_shared<FusedKernelCUDA>(
      static_cast<at::DeviceIndex>(device),
      std::move(name),
      std::move(code),
      std::move(input_desc),
      std::move(output_desc),
      std::move(chunk_desc),
      std::move(concat_desc),
      has_random);
}

RegisterFusionBackend reg(DeviceType::CUDA, createFusionKernel);

} // namespace torch::jit::fuser::cuda
```

- **EN:** Important callable entry points in this range include move, reg.
- **CN:** 这一段的重要可调用入口包括 move, reg。
- **EN:** Registration-related statements connect the implementation to operators, schemas, or runtime dispatch tables.
- **CN:** 与注册相关的语句把实现连接到算子、模式或运行时分发表。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。
- **EN:** Concepts touched here: Nodes and values / 节点与值, Type system / 类型系统, Operator schema / 算子模式, Backend integration / 后端集成, Optimization pass / 优化 pass, Registration / 注册机制.
- **CN:** 这里涉及的概念包括：Nodes and values / 节点与值, Type system / 类型系统, Operator schema / 算子模式, Backend integration / 后端集成, Optimization pass / 优化 pass, Registration / 注册机制。

## Key Concepts / 关键概念

- **Generated code pipeline** — 代码生成流水线
- **Nodes and values** — 节点与值
- **Control-flow blocks** — 控制流块
- **Type system** — 类型系统
- **Operator schema** — 算子模式
- **Module API** — 模块 API
- **Backend integration** — 后端集成
- **Code generation** — 代码生成
- **Optimization pass** — 优化 pass
- **Registration** — 注册机制

## Dependencies / 依赖关系

- `torch/csrc/jit/codegen/fuser/cuda/fused_kernel.h`
- `torch/csrc/jit/codegen/fuser/compiler.h`
- `ATen/ATen.h`
- `ATen/cuda/CUDAContext.h`
- `ATen/cuda/CUDAGeneratorImpl.h`
- `ATen/cuda/nvrtc_stub/ATenNVRTC.h`
- `ATen/native/cuda/jit_utils.h`
- `c10/cuda/CUDAGuard.h`
- `c10/util/Exception.h`
- `torch/csrc/jit/resource_guard.h`
