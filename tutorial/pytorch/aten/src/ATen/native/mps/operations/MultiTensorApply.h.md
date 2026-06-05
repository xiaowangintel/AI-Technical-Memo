# MultiTensorApply.h — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/mps/operations/MultiTensorApply.h`
- Repository: `pytorch/pytorch` (`27a4844d7fb`)
- Purpose (EN): This file belongs to the MPS backend in PyTorch ATen native code and focuses on multi tensor apply; it mainly declares interfaces, helper types, and inline utilities.
- 目的（CN）: 该文件属于 PyTorch ATen 原生代码中的MPS 后端，主题聚焦于 multi tensor apply；其主要作用是声明接口、辅助类型以及内联工具。

## Line-by-Line Analysis / 逐行分析

### Lines 1-60
```cpp
#pragma once
#include <ATen/core/Tensor.h>
#include <ATen/mps/MPSProfiler.h>
#include <ATen/native/mps/OperationUtils.h>

static_assert(sizeof(bool) == 1);

namespace at::native::mps {

static constexpr int64_t kChunkSize = 65536;
static constexpr int64_t kmaxThreadGroups = 32;
static constexpr int64_t kmaxTensors = 32;

struct MetadataArguments { // the size of this struct must be less than 4 kilobytes
  uint64_t numels[kmaxTensors];
  uint64_t threadgroup_to_tensor[kmaxThreadGroups];
  uint64_t threadgroup_to_chunk[kmaxThreadGroups];
};

struct FusedAdamEncodingFunctor {
  void operator()(id<MTLComputeCommandEncoder>& computeEncoder,
                  id<MTLBuffer>& tensorArgumentBuffer,
                  const MetadataArguments& metadata_arguments,
                  const double lr,
                  const double beta1,
                  const double beta2,
                  const double weight_decay,
                  const double eps,
                  const bool maximize) const {
    mtl_setArgs(
        computeEncoder, tensorArgumentBuffer, metadata_arguments, lr, beta1, beta2, weight_decay, eps, maximize);
  }

  void operator()(id<MTLComputeCommandEncoder>& computeEncoder,
                  id<MTLBuffer>& tensorArgumentBuffer,
                  const MetadataArguments& metadata_arguments,
                  const at::Tensor& lr,
                  const double beta1,
                  const double beta2,
                  const double weight_decay,
                  const double eps,
                  const bool maximize) const {
    mtl_setArgs(
        computeEncoder, tensorArgumentBuffer, metadata_arguments, lr, beta1, beta2, weight_decay, eps, maximize);
  }
};

template <bool momentum>
struct FusedSgdEncodingFunctor {};

template <>
struct FusedSgdEncodingFunctor<true> {
  void operator()(id<MTLComputeCommandEncoder>& computeEncoder,
                  id<MTLBuffer>& tensorArgumentBuffer,
                  const MetadataArguments& metadata_arguments,
                  const double weight_decay,
                  const double momentum,
                  const double lr,
                  const double dampening,
                  const bool nesterov,
```
- EN: Lines 1-60 pull in 3 header dependencies, defining the compilation surface for this segment.
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 第 1-60 行引入了 3 个头文件依赖，为该代码段确定编译边界和可见接口。
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 61-120
```cpp
                  const bool maximize,
                  const bool is_first_step) const {
    mtl_setArgs(computeEncoder,
                tensorArgumentBuffer,
                metadata_arguments,
                weight_decay,
                momentum,
                lr,
                dampening,
                nesterov,
                maximize,
                is_first_step);
  }

  void operator()(id<MTLComputeCommandEncoder>& computeEncoder,
                  id<MTLBuffer>& tensorArgumentBuffer,
                  const MetadataArguments& metadata_arguments,
                  const double weight_decay,
                  const double momentum,
                  const at::Tensor& lr,
                  const double dampening,
                  const bool nesterov,
                  const bool maximize,
                  const bool is_first_step) const {
    mtl_setArgs(computeEncoder,
                tensorArgumentBuffer,
                metadata_arguments,
                weight_decay,
                momentum,
                lr,
                dampening,
                nesterov,
                maximize,
                is_first_step);
  }
};

template <>
struct FusedSgdEncodingFunctor<false> {
  void operator()(id<MTLComputeCommandEncoder>& computeEncoder,
                  id<MTLBuffer>& tensorArgumentBuffer,
                  const MetadataArguments& metadata_arguments,
                  const double weight_decay,
                  const double lr,
                  const bool maximize) const {
    mtl_setArgs(computeEncoder, tensorArgumentBuffer, metadata_arguments, weight_decay, lr, maximize);
  }

  void operator()(id<MTLComputeCommandEncoder>& computeEncoder,
                  id<MTLBuffer>& tensorArgumentBuffer,
                  const MetadataArguments& metadata_arguments,
                  const double weight_decay,
                  const at::Tensor& lr,
                  const bool maximize) const {
    mtl_setArgs(computeEncoder, tensorArgumentBuffer, metadata_arguments, weight_decay, lr, maximize);
  }
};

std::pair<id<MTLComputePipelineState>, id<MTLFunction>> getFusedAdamCPLState(const std::string& fname);
template <int depth, uint32_t kThreadGroupSize, typename encoder_func_t, typename... ArgTypes>
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 121-180
```cpp
static void multi_tensor_apply_for_fused_optimizer(const std::string& kernel_name,
                                                   std::vector<std::vector<at::Tensor>>& tensor_lists,
                                                   at::TensorList state_steps,
                                                   encoder_func_t encode,
                                                   ArgTypes... args) {
  const auto num_tensors = tensor_lists[0].size();

  if (num_tensors == 0) {
    return;
  }

  TORCH_CHECK(tensor_lists.size() == depth, "Number of tensor lists has to match the depth");
  for (const auto& d : c10::irange(depth)) {
    const auto scalar_type = tensor_lists[d][0].scalar_type();
    TORCH_CHECK(scalar_type == kFloat || scalar_type == kHalf || scalar_type == kBFloat16,
                "Only float, bfloat and half are supported");
  }

  id<MTLDevice> device = MPSDevice::getInstance()->device();
  MPSStream* mpsStream = getCurrentMPSStream();

  // Remove comment for debugging
  /*
  mpsStream->addCompletedHandler(^(id<MTLCommandBuffer> cb) {
    [cb.logs enumerateObjectsUsingBlock:^(NSString* log, NSUInteger idx, BOOL* stop) {
      NSLog(@"MPSStream: %@", log);
      }
    ];
  });
  */

  dispatch_sync_with_rethrow(mpsStream->queue(), ^() {
    @autoreleasepool {
      id<MTLComputeCommandEncoder> computeEncoder = mpsStream->commandEncoder();
      auto [fusedOptimizerPSO, fusedOptimizerFunc] = getFusedAdamCPLState(kernel_name);

      // this function call is a no-op if MPS Profiler is not enabled
      getMPSProfiler().beginProfileKernel(fusedOptimizerPSO, kernel_name, {tensor_lists[0]});

      [computeEncoder setComputePipelineState:fusedOptimizerPSO];

      // BufferIndex is the index in the kernel function
      auto tensorArgumentEncoder = [[fusedOptimizerFunc newArgumentEncoderWithBufferIndex:0] autorelease];
      id<MTLBuffer> tensorArgumentBuffer = [[device newBufferWithLength:tensorArgumentEncoder.encodedLength
                                                                options:0] autorelease];
      [tensorArgumentEncoder setArgumentBuffer:tensorArgumentBuffer offset:0];

      int64_t tensor_loc = 0;
      int64_t threadgroup_loc = 0;
      MetadataArguments metadata_arguments;

      for (const auto tensor_index : c10::irange(num_tensors)) {
        // short-circuit to avoid adding empty tensors to tensorListMeta
        if (tensor_lists[0][tensor_index].numel() == 0) {
          continue;
        }

        for (const auto& d : c10::irange(depth)) {
          mtl_setBuffer(tensorArgumentEncoder, tensor_lists[d][tensor_index], d * kmaxTensors + tensor_loc);
          [computeEncoder useResource:getMTLBufferStorage(tensor_lists[d][tensor_index])
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are addCompletedHandler, dispatch_sync_with_rethrow, useResource:getMTLBufferStorage, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 addCompletedHandler, dispatch_sync_with_rethrow, useResource:getMTLBufferStorage，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 181-240
```cpp
                                usage:MTLResourceUsageRead | MTLResourceUsageWrite];
        }
        if (!state_steps.empty()) {
          mtl_setBuffer(tensorArgumentEncoder, state_steps[tensor_index], depth * kmaxTensors + tensor_loc);
          [computeEncoder useResource:getMTLBufferStorage(state_steps[tensor_index]) usage:MTLResourceUsageRead];
        }
        metadata_arguments.numels[tensor_loc] = tensor_lists[0][tensor_index].numel();

        tensor_loc++;

        const auto numel = tensor_lists[0][tensor_index].numel();
        const auto chunks = numel / kChunkSize + (numel % kChunkSize != 0);
        TORCH_CHECK(chunks > -1);

        for (const auto& chunk : c10::irange(chunks)) {
          metadata_arguments.threadgroup_to_tensor[threadgroup_loc] = tensor_loc - 1;
          metadata_arguments.threadgroup_to_chunk[threadgroup_loc] = chunk;

          threadgroup_loc++;

          const auto tensor_full = tensor_loc == kmaxTensors && chunk == chunks - 1;
          // Reach the maximum threadgroups per dispatch
          const auto blocks_full = threadgroup_loc == kmaxThreadGroups;

          if (tensor_full || blocks_full) {
            encode(computeEncoder, tensorArgumentBuffer, metadata_arguments, args...);
            MTLSize gridSize = MTLSizeMake(threadgroup_loc, 1, 1);
            uint32_t maxThreadsPerGroup = [fusedOptimizerPSO maxTotalThreadsPerThreadgroup];
            MTLSize threadGroupSize = MTLSizeMake(std::min(maxThreadsPerGroup, kThreadGroupSize), 1, 1);
            [computeEncoder dispatchThreadgroups:gridSize threadsPerThreadgroup:threadGroupSize];

            // Reset
            threadgroup_loc = 0;
            if (chunk == chunks - 1) {
              // last chunk
              tensor_loc = 0;
              tensorArgumentBuffer = [[device newBufferWithLength:tensorArgumentEncoder.encodedLength
                                                          options:0] autorelease];
              [tensorArgumentEncoder setArgumentBuffer:tensorArgumentBuffer offset:0];
            } else {
              // reuse the current tensor since the current one isn't done.
              metadata_arguments.numels[0] = metadata_arguments.numels[tensor_loc - 1];

              tensorArgumentBuffer = [[device newBufferWithLength:tensorArgumentEncoder.encodedLength
                                                          options:0] autorelease];
              [tensorArgumentEncoder setArgumentBuffer:tensorArgumentBuffer offset:0];

              for (const auto& d : c10::irange(depth)) {
                mtl_setBuffer(tensorArgumentEncoder, tensor_lists[d][tensor_index], d * kmaxTensors);
                [computeEncoder useResource:getMTLBufferStorage(tensor_lists[d][tensor_index])
                                      usage:MTLResourceUsageWrite | MTLResourceUsageRead];
              }
              if (!state_steps.empty()) {
                mtl_setBuffer(tensorArgumentEncoder, state_steps[tensor_index], depth * kmaxTensors);
                [computeEncoder useResource:getMTLBufferStorage(state_steps[tensor_index]) usage:MTLResourceUsageRead];
              }
              tensor_loc = 1;
            }
          }
        }
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are useResource:getMTLBufferStorage, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 useResource:getMTLBufferStorage，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 241-300
```cpp
      }

      if (threadgroup_loc != 0) {
        encode(computeEncoder, tensorArgumentBuffer, metadata_arguments, args...);
        MTLSize gridSize = MTLSizeMake(threadgroup_loc, 1, 1);
        uint32_t maxThreadsPerGroup = [fusedOptimizerPSO maxTotalThreadsPerThreadgroup];
        MTLSize threadGroupSize = MTLSizeMake(std::min(maxThreadsPerGroup, kThreadGroupSize), 1, 1);
        [computeEncoder dispatchThreadgroups:gridSize threadsPerThreadgroup:threadGroupSize];
      }

      getMPSProfiler().endProfileKernel(fusedOptimizerPSO);
    }
  });
}

std::pair<id<MTLComputePipelineState>, id<MTLFunction>> getAmpCPLState(const std::string& fname);
template <int depth, typename... ArgTypes>
void multi_tensor_apply(const std::string& kernel_name,
                        std::vector<std::vector<at::Tensor>>& tensor_lists,
                        ArgTypes... args) {
  const auto num_tensors = tensor_lists[0].size();
  if (num_tensors == 0) {
    return;
  }

  TORCH_CHECK(tensor_lists.size() == depth, "Number of tensor lists must match depth.");

  id<MTLDevice> device = MPSDevice::getInstance()->device();
  MPSStream* mpsStream = getCurrentMPSStream();

  dispatch_sync_with_rethrow(mpsStream->queue(), ^() {
    @autoreleasepool {
      id<MTLComputeCommandEncoder> computeEncoder = mpsStream->commandEncoder();
      auto [pipeline, function] = getAmpCPLState(kernel_name);
      [computeEncoder setComputePipelineState:pipeline];

      id<MTLArgumentEncoder> argumentEncoder = [function newArgumentEncoderWithBufferIndex:0];
      auto tensorArgumentBuffer = [[device newBufferWithLength:argumentEncoder.encodedLength options:0] autorelease];
      [argumentEncoder setArgumentBuffer:tensorArgumentBuffer offset:0];

      int tensor_loc = 0;
      int threadgroup_loc = 0;
      MetadataArguments metadata_arguments;
      std::memset(&metadata_arguments, 0, sizeof(metadata_arguments));

      for (size_t t = 0; t < num_tensors; t++) {
        if (tensor_lists[0][t].numel() == 0)
          continue;

        // bind each tensor in this list to the correct slots across depths
        for (int d = 0; d < depth; d++) {
          mtl_setBuffer(argumentEncoder, tensor_lists[d][t], d * kmaxTensors + tensor_loc);
          [computeEncoder useResource:getMTLBufferStorage(tensor_lists[d][t])
                                usage:(MTLResourceUsageRead | MTLResourceUsageWrite)];
        }

        // save number of elements for this tensor
        metadata_arguments.numels[tensor_loc] = tensor_lists[0][t].numel();
        int currentTensorIndex = tensor_loc;
        tensor_loc++;
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are dispatch_sync_with_rethrow, useResource:getMTLBufferStorage, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 dispatch_sync_with_rethrow, useResource:getMTLBufferStorage，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 301-360
```cpp

        const auto numel = tensor_lists[0][t].numel();
        const auto chunks = numel / kChunkSize + ((numel % kChunkSize) ? 1 : 0);

        // process tensor in chunks based on max chunk size
        for (uint chunk = 0; chunk < chunks; chunk++) {
          metadata_arguments.threadgroup_to_tensor[threadgroup_loc] = currentTensorIndex;
          metadata_arguments.threadgroup_to_chunk[threadgroup_loc] = chunk;
          threadgroup_loc++;

          // dispatch when we've filled the threadgroup array or finished the chunks
          const bool dispatch_now = (threadgroup_loc == kmaxThreadGroups) || (chunk == chunks - 1);
          if (dispatch_now) {
            // check for a partial dispatch (i.e. more chunks remain for the current tensor)
            bool partial = (chunk != chunks - 1);
            uint carried_numels = 0;
            if (partial) {
              carried_numels = metadata_arguments.numels[currentTensorIndex];
            }

            mtl_setArgs(computeEncoder, tensorArgumentBuffer, metadata_arguments, args...);
            MTLSize gridSize = MTLSizeMake(threadgroup_loc, 1, 1);
            uint32_t maxThreads = [pipeline maxTotalThreadsPerThreadgroup];
            MTLSize threadGroupSize = MTLSizeMake(std::min(maxThreads, (uint32_t)64), 1, 1);
            [computeEncoder dispatchThreadgroups:gridSize threadsPerThreadgroup:threadGroupSize];

            // prepare for the next batch: reset threadgroup count and create a new buffer
            threadgroup_loc = 0;
            tensorArgumentBuffer = [[device newBufferWithLength:argumentEncoder.encodedLength options:0] autorelease];
            [argumentEncoder setArgumentBuffer:tensorArgumentBuffer offset:0];

            if (partial) {
              // for a partial dispatch, rebind the partially processed tensor to slot 0
              // so that its metadata is in the correct location
              for (int d = 0; d < depth; d++) {
                mtl_setBuffer(argumentEncoder, tensor_lists[d][t], d * kmaxTensors + 0);
                [computeEncoder useResource:getMTLBufferStorage(tensor_lists[d][t])
                                      usage:(MTLResourceUsageRead | MTLResourceUsageWrite)];
              }
              metadata_arguments.numels[0] = carried_numels;
              // the currently processed tensor now lives at index 0
              currentTensorIndex = 0;
              tensor_loc = 1;
            } else {
              tensor_loc = 0;
            }
          }
        }
      }

      if (threadgroup_loc != 0) {
        mtl_setArgs(computeEncoder, tensorArgumentBuffer, metadata_arguments, args...);
        MTLSize gridSize = MTLSizeMake(threadgroup_loc, 1, 1);
        uint32_t maxThreads = [pipeline maxTotalThreadsPerThreadgroup];
        MTLSize threadGroupSize = MTLSizeMake(std::min(maxThreads, static_cast<uint32_t>(64)), 1, 1);
        [computeEncoder dispatchThreadgroups:gridSize threadsPerThreadgroup:threadGroupSize];
      }
    }
  });
}
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are dispatch, useResource:getMTLBufferStorage, concentrating a specific part of the operator behavior.
- EN: Loop constructs iterate over tensor dimensions, descriptors, lookup tables, or grouped work items to build results or metadata.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 dispatch, useResource:getMTLBufferStorage，它们承载了某一部分算子行为的核心逻辑。
- CN: 循环结构会遍历张量维度、描述符、查找表或分组工作项，以构造结果或元数据。

### Lines 361-362
```cpp

} // namespace at::native::mps
```
- EN: This range provides supporting glue code: declarations, simple forwarding logic, or structural scaffolding reused by nearby operator code.
- CN: 这一段提供了辅助性胶水代码：声明、简单转发逻辑，或被邻近算子代码复用的结构骨架。

## Key Concepts / 关键概念

- EN: Backend focus: MPS backend.
- CN: 后端重点：MPS 后端。
- EN: Shape, dtype, layout, or backend validation is a first-class concern.
- CN: 形状、数据类型、布局或后端校验是该文件的重要关注点。
- EN: Template-based reuse helps share logic across scalar types, layouts, or backends.
- CN: 基于模板的复用帮助在不同标量类型、布局或后端之间共享逻辑。
- EN: Notable symbols: operator, multi_tensor_apply_for_fused_optimizer, dispatch_sync_with_rethrow, multi_tensor_apply.
- CN: 重要符号：operator, multi_tensor_apply_for_fused_optimizer, dispatch_sync_with_rethrow, multi_tensor_apply。

## Dependencies / 依赖关系

- EN: Primary internal headers: `ATen/core/Tensor.h, ATen/mps/MPSProfiler.h, ATen/native/mps/OperationUtils.h`.
- CN: 主要内部头文件：`ATen/core/Tensor.h, ATen/mps/MPSProfiler.h, ATen/native/mps/OperationUtils.h`。
- EN: External/system headers are minimal in this file.
- CN: 该文件中的外部/系统头文件依赖较少。
- EN: The implementation revolves around symbols such as `operator, multi_tensor_apply_for_fused_optimizer, dispatch_sync_with_rethrow, multi_tensor_apply`.
- CN: 实现围绕 `operator, multi_tensor_apply_for_fused_optimizer, dispatch_sync_with_rethrow, multi_tensor_apply` 等符号展开。
