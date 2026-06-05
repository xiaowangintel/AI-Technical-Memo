# intra_node_comm.cu — Code Analysis / 代码分析

## Source / 来源

- File: `torch/csrc/distributed/c10d/symm_mem/intra_node_comm.cu`
- Repository: `pytorch` (`/root/xw/pytorch`)
- Purpose (EN): This file provides implementation logic for intra node comm in the c10d symmetric-memory support. Representative routines include `checkInput`, `isIntraNodeCommSupported`, `C10_THROW_ERROR`, `getIntraNodeCommUsageCounter`. GPU-oriented code paths are present in this file.
- 用途 (CN): 该文件在c10d 对称内存支持中提供intra node comm 的实现逻辑。 代表性例程包括 `checkInput`、`isIntraNodeCommSupported`、`C10_THROW_ERROR`、`getIntraNodeCommUsageCounter`。 该文件还包含面向 GPU 的代码路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12 / 第 1-12 行

```cpp
1: #include <torch/csrc/distributed/c10d/symm_mem/intra_node_comm.hpp>
2: 
3: #include <torch/csrc/distributed/c10d/symm_mem/CUDASymmetricMemory-inl.cuh>
4: 
5: namespace c10d {
6: namespace intra_node_comm {
7: 
8: static constexpr size_t kOneShotThreshBytes = 256 * 1024;
9: static constexpr size_t kTwoShotThreshBytes = 10 * 1024 * 1024;
10: 
11: static void checkInput(const at::Tensor& input, int deviceIdx) {
12:   TORCH_CHECK(
```

- EN: Lines 1-12 pulls in the headers required by this translation unit or interface; opens or closes namespaces to place the code in the correct distributed component; introduces executable logic in routines such as `checkInput`.
- CN: 第 1-12 行引入该实现单元或接口所需的头文件；打开或关闭命名空间，以便将代码放入正确的分布式组件中；在 `checkInput` 等例程中引入具体执行逻辑。

### Lines 13-24 / 第 13-24 行

```cpp
13:       input.dtype() == at::kBFloat16 || input.dtype() == at::kFloat,
14:       "oneShotAllReduce only supports float and bf16 for now");
15:   TORCH_CHECK(input.is_non_overlapping_and_dense());
16:   TORCH_CHECK(input.device().is_cuda());
17:   TORCH_CHECK(
18:       input.get_device() == deviceIdx,
19:       "IntraNodeComm: expect input to be on device ",
20:       deviceIdx,
21:       ", got device ",
22:       input.get_device());
23: }
24: 
```

- EN: Lines 13-24 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing; performs validation and error handling to keep distributed state consistent.
- CN: 第 13-24 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程；执行校验与错误处理，以保持分布式状态一致。

### Lines 25-36 / 第 25-36 行

```cpp
25: bool isIntraNodeCommSupported() {
26: #if defined(__CUDA_ARCH__) && (__CUDA_ARCH__ < 800)
27:   return false;
28: #else
29:   return true;
30: #endif
31: }
32: 
33: at::Tensor IntraNodeComm::oneShotAllReduce(
34:     const at::Tensor& input,
35:     at::cuda::CUDAStream& stream) {
36:   checkInput(input, deviceIdx_);
```

- EN: Lines 25-36 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `isIntraNodeCommSupported`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 25-36 行使用条件编译来适配特性开关、平台或可选后端；在 `isIntraNodeCommSupported` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 37-48 / 第 37-48 行

```cpp
37: 
38:   auto op = c10::Dispatcher::singleton()
39:                 .findSchemaOrThrow("symm_mem::one_shot_all_reduce_out", "")
40:                 .typed<at::Tensor(
41:                     const at::Tensor&, std::string, std::string, at::Tensor)>();
42: 
43:   auto symmMemTensor = at::from_blob(
44:       symmetricMemoryPtr_,
45:       input.sizes(),
46:       at::TensorOptions().dtype(input.dtype()).device(input.device()));
47: 
48:   symmMemTensor.copy_(input);
```

- EN: Lines 37-48 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 37-48 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

### Lines 49-60 / 第 49-60 行

```cpp
49:   op.call(symmMemTensor, "sum", groupName_, input);
50:   return input;
51: }
52: 
53: at::Tensor IntraNodeComm::twoShotAllReduce(
54:     const at::Tensor& input,
55:     at::cuda::CUDAStream& stream) {
56:   checkInput(input, deviceIdx_);
57: 
58:   auto op = c10::Dispatcher::singleton()
59:                 .findSchemaOrThrow("symm_mem::two_shot_all_reduce_", "")
60:                 .typed<at::Tensor(at::Tensor, std::string, std::string)>();
```

- EN: Lines 49-60 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 49-60 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

### Lines 61-72 / 第 61-72 行

```cpp
61: 
62:   auto symmMemTensor = at::from_blob(
63:       symmetricMemoryPtr_,
64:       input.sizes(),
65:       at::TensorOptions().dtype(input.dtype()).device(input.device()));
66: 
67:   symmMemTensor.copy_(input);
68:   op.call(symmMemTensor, "sum", groupName_);
69:   input.copy_(symmMemTensor);
70:   return input;
71: }
72: 
```

- EN: Lines 61-72 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 61-72 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

### Lines 73-84 / 第 73-84 行

```cpp
73: AllReduceAlgo IntraNodeComm::selectAllReduceAlgo(const at::Tensor& input) {
74:   // Only support float and bf16 for now
75:   if (input.dtype() != at::kBFloat16 && input.dtype() != at::kFloat) {
76:     return AllReduceAlgo::NONE;
77:   }
78:   const auto inputSize =
79:       static_cast<size_t>(input.numel() * input.element_size());
80:   const size_t ptrAlignment = get_alignment(
81:       static_cast<size_t>(input.storage_offset() * input.element_size()));
82:   const size_t sizeAlignment = get_alignment(inputSize);
83:   const size_t alignment = std::min(ptrAlignment, sizeAlignment);
84: 
```

- EN: Lines 73-84 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 73-84 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

### Lines 85-96 / 第 85-96 行

```cpp
85:   if (topology_ == Topology::FULLY_CONNECTED) {
86:     // Both symm_mem::one_shot_all_reduce and symm_mem::two_shot_all_reduce_
87:     // currently requires the input to be at least 4-bytes aligned.
88:     if (alignment >= 4 && inputSize <= kOneShotThreshBytes &&
89:         inputSize <= bufferSize_) {
90:       return AllReduceAlgo::ONE_SHOT;
91:     }
92:     if (alignment >= 4 && inputSize <= kTwoShotThreshBytes &&
93:         inputSize <= bufferSize_) {
94:       return AllReduceAlgo::TWO_SHOT;
95:     }
96:   }
```

- EN: Lines 85-96 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 85-96 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

### Lines 97-108 / 第 97-108 行

```cpp
97:   return AllReduceAlgo::NONE;
98: }
99: 
100: static int64_t usageCounter = 0;
101: 
102: at::Tensor IntraNodeComm::allReduce(
103:     const at::Tensor& input,
104:     AllReduceAlgo algo) {
105:   // Report usage for testing purposes.
106:   // We don't care about overflowing.
107:   ++usageCounter;
108:   auto stream = at::cuda::getCurrentCUDAStream();
```

- EN: Lines 97-108 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 97-108 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

### Lines 109-120 / 第 109-120 行

```cpp
109:   switch (algo) {
110:     case AllReduceAlgo::ONE_SHOT:
111:       return oneShotAllReduce(input, stream);
112:     case AllReduceAlgo::TWO_SHOT:
113:       return twoShotAllReduce(input, stream);
114:     default:
115:       C10_THROW_ERROR(ValueError, "IntraNodeComm: invalid algo");
116:   }
117: }
118: 
119: int64_t getIntraNodeCommUsageCounter() {
120:   return usageCounter;
```

- EN: Lines 109-120 introduces executable logic in routines such as `C10_THROW_ERROR`, `getIntraNodeCommUsageCounter`; contains CUDA-oriented declarations, kernels, or launch plumbing; performs validation and error handling to keep distributed state consistent.
- CN: 第 109-120 行在 `C10_THROW_ERROR`、`getIntraNodeCommUsageCounter` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程；执行校验与错误处理，以保持分布式状态一致。

### Lines 121-124 / 第 121-124 行

```cpp
121: }
122: 
123: } // namespace intra_node_comm
124: } // namespace c10d
```

- EN: Lines 121-124 opens or closes namespaces to place the code in the correct distributed component; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 121-124 行打开或关闭命名空间，以便将代码放入正确的分布式组件中；包含面向 CUDA 的声明、内核或启动流程。

## Key Concepts / 关键概念

- EN: Subsystem: c10d symmetric-memory support.
- CN: 子系统：c10d 对称内存支持。
- EN: File role: implementation/registration.
- CN: 文件角色：实现/注册。
- EN: Primary symbols: `checkInput`, `isIntraNodeCommSupported`, `C10_THROW_ERROR`, `getIntraNodeCommUsageCounter`
- CN: 核心符号：`checkInput`、`isIntraNodeCommSupported`、`C10_THROW_ERROR`、`getIntraNodeCommUsageCounter`
- EN: Notable themes: CUDA paths, collective communication logic.
- CN: 值得关注的主题：CUDA 路径、集合通信逻辑。

## Dependencies / 依赖关系

- Internal distributed headers / 内部分布式头文件: `torch/csrc/distributed/c10d/symm_mem/intra_node_comm.hpp`, `torch/csrc/distributed/c10d/symm_mem/CUDASymmetricMemory-inl.cuh`
- PyTorch/ATen/C10 headers / PyTorch/ATen/C10 头文件: None / 无
- External or system headers / 外部或系统头文件: None / 无
- Local symbols / 本地符号: `checkInput`, `isIntraNodeCommSupported`, `C10_THROW_ERROR`, `getIntraNodeCommUsageCounter`