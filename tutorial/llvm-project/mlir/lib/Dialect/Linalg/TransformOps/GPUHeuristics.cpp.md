# GPUHeuristics.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/Linalg/TransformOps/GPUHeuristics.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements transform-dialect operations, matchers, or extensions for the Linalg dialect and structured tensor computation.
  - **CN**: 实现 Linalg 方言与结构化张量计算 的 Transform Dialect 操作、匹配器或扩展。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- GPUHeuristics.cpp - Heuristics Implementation for Transforms -------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件的高层描述。

### Lines 8-21
```cpp

#include "mlir/Dialect/Linalg/TransformOps/GPUHeuristics.h"

#include "mlir/Dialect/GPU/IR/GPUDialect.h"
#include "llvm/ADT/ArrayRef.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/ADT/SmallVectorExtras.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/DebugLog.h"
#include "llvm/Support/InterleavedRange.h"
#include "llvm/Support/MathExtras.h"
#include "llvm/Support/raw_ostream.h"
#include <cmath>
#include <numeric>
```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Linalg/TransformOps/GPUHeuristics.h`, `mlir/Dialect/GPU/IR/GPUDialect.h`, `llvm/ADT/ArrayRef.h`, `llvm/ADT/STLExtras.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Linalg/TransformOps/GPUHeuristics.h`, `mlir/Dialect/GPU/IR/GPUDialect.h`, `llvm/ADT/ArrayRef.h`, `llvm/ADT/STLExtras.h`。

### Lines 22-35
```cpp

using namespace mlir;

#define DEBUG_TYPE "linalg-transforms"

static Attribute linearId0(MLIRContext *ctx) {
  return gpu::GPUThreadMappingAttr::get(ctx, gpu::MappingId::LinearDim0);
}
static Attribute linearId1(MLIRContext *ctx) {
  return gpu::GPUThreadMappingAttr::get(ctx, gpu::MappingId::LinearDim1);
}
static Attribute linearId2(MLIRContext *ctx) {
  return gpu::GPUThreadMappingAttr::get(ctx, gpu::MappingId::LinearDim2);
}
```
- **EN**: Implements logic around `linearId0`, `get`, `linearId1`, `linearId2`; this block encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `linearId0`, `get`, `linearId1`, `linearId2` 实现具体逻辑；该代码块编码加速器专用执行或 lowering 规则。

### Lines 36-45
```cpp

transform::gpu::CopyMappingInfo::CopyMappingInfo(MLIRContext *ctx,
                                                 int totalNumThreads,
                                                 int64_t desiredBitAlignment,
                                                 ArrayRef<int64_t> copySizes,
                                                 bool favorPredication,
                                                 int64_t elementalBitwidth) {
  assert(!copySizes.empty() && copySizes.size() <= 3 &&
         "only 1,2,3-D copies are supported for now");

```
- **EN**: Implements logic around `CopyMappingInfo`, `assert`; this block encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `CopyMappingInfo`, `assert` 实现具体逻辑；该代码块编码加速器专用执行或 lowering 规则。

### Lines 46-54
```cpp
  LDBG() << "START CopyMappingInfo, favorPredication: " << favorPredication;
  LDBG() << "--copy shape: " << llvm::interleaved(copySizes);

  // Greedily find the largest vector size that can be used to copy the most
  // minor dimension: we are in the business of filling kMaxVectorLoadBitWidth
  // contiguous memory transactions with as few threads as possible.
  int64_t desiredVectorSize = CopyMappingInfo::maxContiguousElementsToTransfer(
      desiredBitAlignment, copySizes.back(), elementalBitwidth);

```
- **EN**: Implements logic around `LDBG`, `maxContiguousElementsToTransfer`, `back`.
- **CN**: 围绕 `LDBG`, `maxContiguousElementsToTransfer`, `back` 实现具体逻辑。

### Lines 55-64
```cpp
  LDBG() << "--greedily determined vectorSize: " << desiredVectorSize
         << " elements of " << elementalBitwidth << "b each -> "
         << (desiredVectorSize * elementalBitwidth)
         << "b total out of a max of " << kMaxVectorLoadBitWidth << "b";

  status = inferNumThreads(totalNumThreads, copySizes, desiredVectorSize,
                           favorPredication);
  if (status == Status::Invalid)
    return;

```
- **EN**: Implements logic around `LDBG`, `inferNumThreads`; this block implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `LDBG`, `inferNumThreads` 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子。

### Lines 65-78
```cpp
  LDBG() << "--copy: " << llvm::interleaved(copySizes) << "\n"
         << "--numThreads: " << llvm::interleaved(this->numThreads) << "\n"
         << "--vectorSize: " << this->vectorSize;
  assert(this->numThreads.size() == copySizes.size() &&
         "compute copy mapping expected same number of threads and copy sizes");

  // Compute the smallest bounding box.
  this->smallestBoundingTileSizes = llvm::map_to_vector(
      llvm::zip(copySizes, this->numThreads), [](auto &&pair) {
        int64_t size, numThreads;
        std::tie(size, numThreads) = pair;
        return llvm::divideCeilSigned(size, numThreads);
      });
  SmallVector<Attribute> allThreadMappings{linearId2(ctx), linearId1(ctx),
```
- **EN**: Implements logic around `LDBG`, `interleaved`, `assert`, `map_to_vector`, and 4 more symbols.
- **CN**: 围绕 `LDBG`, `interleaved`, `assert`, `map_to_vector`, and 4 more symbols 实现具体逻辑。

### Lines 79-87
```cpp
                                           linearId0(ctx)};

  // Set the thread mapping.
  this->threadMapping =
      llvm::to_vector(ArrayRef(allThreadMappings)
                          .take_back(this->smallestBoundingTileSizes.size()));
  LDBG() << *this;
}

```
- **EN**: Implements logic around `linearId0`, `to_vector`, `take_back`, `LDBG`; this block encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `linearId0`, `to_vector`, `take_back`, `LDBG` 实现具体逻辑；该代码块编码加速器专用执行或 lowering 规则。

### Lines 88-99
```cpp
int64_t transform::gpu::CopyMappingInfo::maxContiguousElementsToTransfer(
    int64_t desiredBitAlignment, int64_t numContiguousElements,
    int64_t elementalBitwidth) {
  assert(kMaxVectorLoadBitWidth % elementalBitwidth == 0 &&
         "elemental bitwidth does not divide kMaxVectorLoadBitWidth");
  assert(desiredBitAlignment % elementalBitwidth == 0 &&
         "elemental bitwidth does not divide desired bit alignment");
  return std::gcd(
      std::gcd(desiredBitAlignment / elementalBitwidth, numContiguousElements),
      kMaxVectorLoadBitWidth / elementalBitwidth);
}

```
- **EN**: Implements logic around `maxContiguousElementsToTransfer`, `assert`, `gcd`; this block encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `maxContiguousElementsToTransfer`, `assert`, `gcd` 实现具体逻辑；该代码块编码加速器专用执行或 lowering 规则。

### Lines 100-112
```cpp
/// Get the list of all factors that divide `val`, not just the prime factors.
static SmallVector<int64_t> getFactors(int64_t val) {
  SmallVector<int64_t> factors;
  factors.reserve(val);
  for (int64_t factor = 1; factor <= val; ++factor) {
    if (val % factor != 0)
      continue;
    factors.push_back(factor);
  }
  factors.push_back(val);
  return factors;
}

```
- **EN**: Implements logic around `getFactors`, `reserve`, `push_back`.
- **CN**: 围绕 `getFactors`, `reserve`, `push_back` 实现具体逻辑。

### Lines 113-119
```cpp
static int64_t product(ArrayRef<int64_t> vals) {
  int64_t res = 1;
  for (auto val : vals)
    res *= val;
  return res;
}

```
- **EN**: Implements logic around `product`.
- **CN**: 围绕 `product` 实现具体逻辑。

### Lines 120-126
```cpp
/// Extract `result` from `sizes` with the following constraints:
///   1. sizes[i] % result[i] for all i
///   2. product_of_threadsPerDim <= maxNumThreads
///   3. if `currentIndex` is sizes.size() - 1, then threadsPerDim[currentIndex]
///      must be sizes[currentIndex].
/// This is used to greedily extract the maximum number of threads usable for
/// mapping a copy of size `sizes`, while being bounded by `totalNumThreads` and
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 127-140
```cpp
/// ensuring coalesced access along the most minor dimension.
/// Return the number of threads used in the range:
///   threadsPerDim[currentIndex .. sizes.end()]
// The implementation uses a dynamic programming approach to greedily extract
// the best combination under the constraints.
// TODO: Implementation details can be improved but putting effort there is a
// tradeoffs: `sizes` is expected to be of small rank and contain small values.
static SmallVector<int64_t> maximizeNumThreads(ArrayRef<int64_t> sizes,
                                               int64_t currentIndex,
                                               int64_t maxNumThreads) {
  assert(static_cast<size_t>(currentIndex) < sizes.size() &&
         "currentIndex out of bounds");
  std::string indent(2 * currentIndex, '-');
  if (static_cast<size_t>(currentIndex) == sizes.size() - 1) {
```
- **EN**: Implements logic around `maximizeNumThreads`, `assert`, `indent`, `static_cast`.
- **CN**: 围绕 `maximizeNumThreads`, `assert`, `indent`, `static_cast` 实现具体逻辑。

### Lines 141-154
```cpp
    LDBG() << indent << "mandated globalBest: " << sizes[currentIndex];
    return SmallVector<int64_t>{sizes[currentIndex]};
  }

  int64_t best = 0;
  int64_t s = sizes[currentIndex];
  SmallVector<int64_t> factors = getFactors(s);
  SmallVector<int64_t> localThreadsPerDim;
  localThreadsPerDim.reserve(sizes.size());
  LDBG() << indent << "maximizeNumThreads in " << s
         << " with limit: " << maxNumThreads;
  for (auto factor : factors) {
    auto nestedThreadsPerDim =
        maximizeNumThreads(sizes, currentIndex + 1, maxNumThreads / factor);
```
- **EN**: Implements logic around `LDBG`, `getFactors`, `reserve`, `maximizeNumThreads`.
- **CN**: 围绕 `LDBG`, `getFactors`, `reserve`, `maximizeNumThreads` 实现具体逻辑。

### Lines 155-166
```cpp
    int64_t localBest = factor * product(nestedThreadsPerDim);
    if (localBest > best && localBest <= maxNumThreads) {
      LDBG() << indent << "new localBest: " << localBest;
      LDBG() << indent << "nestedThreadsPerDim: "
             << llvm::interleaved(nestedThreadsPerDim);
      localThreadsPerDim.clear();
      localThreadsPerDim.push_back(factor);
      llvm::append_range(localThreadsPerDim, nestedThreadsPerDim);
      best = localBest;
    }
  }

```
- **EN**: Implements logic around `product`, `LDBG`, `interleaved`, `clear`, and 2 more symbols.
- **CN**: 围绕 `product`, `LDBG`, `interleaved`, `clear`, and 2 more symbols 实现具体逻辑。

### Lines 167-177
```cpp
  LDBG() << indent << "found globalBest: " << best;
  LDBG() << indent << "numThreads: " << llvm::interleaved(localThreadsPerDim);
  return localThreadsPerDim;
}

transform::gpu::CopyMappingInfo::Status
transform::gpu::CopyMappingInfo::inferNumThreads(int64_t totalNumThreads,
                                                 ArrayRef<int64_t> sizes,
                                                 int64_t desiredVectorSize,
                                                 bool favorPredication) {

```
- **EN**: Implements logic around `LDBG`, `inferNumThreads`; this block implements verifier, folding, parsing, or printing hooks; encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `LDBG`, `inferNumThreads` 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子，并编码加速器专用执行或 lowering 规则。

### Lines 178-191
```cpp
  if (!favorPredication) {
    int64_t localVectorSize = desiredVectorSize;
    for (; localVectorSize >= 1; localVectorSize /= 2) {
      // Attempt to map the copy with predication and current fixed vector size:
      //   1. if the status is Success, we are done.
      //   2. if the status is Invalid, we fail immediately, no amount of
      //   vector size reduction can offset the bad tile size selection from the
      //   higher-level.
      //   3. if the status is RequiresPredication, we try again with a smaller
      //   vector size.
      Status status =
          inferNumThreadsImpl(totalNumThreads, sizes, localVectorSize);
      if (status == Status::Success || status == Status::Invalid)
        return status;
```
- **EN**: Implements logic around `inferNumThreadsImpl`; this block implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `inferNumThreadsImpl` 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子。

### Lines 192-203
```cpp

      LDBG() << "requires predication, try reducing vector size to "
             << (localVectorSize / 2);
    }
  }

  // If we have not yet returned, it means that we have tried all vector sizes
  // and we still require predication. Restart from the original vector size and
  // do not attempt to
  return inferNumThreadsImpl(totalNumThreads, sizes, desiredVectorSize);
}

```
- **EN**: Implements logic around `LDBG`, `inferNumThreadsImpl`; this block implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `LDBG`, `inferNumThreadsImpl` 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子。

### Lines 204-210
```cpp
transform::gpu::CopyMappingInfo::Status
transform::gpu::CopyMappingInfo::inferNumThreadsImpl(
    int64_t totalNumThreads, ArrayRef<int64_t> sizes,
    int64_t desiredVectorSize) {
  assert(sizes.back() % desiredVectorSize == 0 &&
         "most-minor size not divisible by actualVectorSize");

```
- **EN**: Implements logic around `inferNumThreadsImpl`, `assert`; this block implements verifier, folding, parsing, or printing hooks; encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `inferNumThreadsImpl`, `assert` 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子，并编码加速器专用执行或 lowering 规则。

### Lines 211-224
```cpp
  LDBG() << "inferNumThreadsImpl with totalNumThreads: " << totalNumThreads
         << " and vectorSize: " << desiredVectorSize;

  // Scale the most minor size to account for the chosen vector size and
  // maximize the number of threads without exceeding the total number of
  // threads.
  SmallVector<int64_t> scaledSizes(sizes);
  scaledSizes.back() /= desiredVectorSize;
  if (scaledSizes.back() > totalNumThreads) {
    LDBG() << "--Too few threads given the required vector size -> FAIL";
    return Status::Invalid;
  }
  SmallVector<int64_t> inferredNumThreads =
      maximizeNumThreads(scaledSizes, 0, totalNumThreads);
```
- **EN**: Implements logic around `LDBG`, `scaledSizes`, `back`, `maximizeNumThreads`; this block implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `LDBG`, `scaledSizes`, `back`, `maximizeNumThreads` 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子。

### Lines 225-238
```cpp

  LDBG() << "inferred numThreads: " << llvm::interleaved(inferredNumThreads);
  LDBG() << "computed actualVectorSize: " << desiredVectorSize;

  // Corner case: we cannot use more threads than available. If the dimension of
  // the copy is so bad it is because higher-level tiling did not do its job, we
  // do not try to recover from it here.
  int64_t totalNumThreadsUsed = product(inferredNumThreads);
  LDBG() << "--totalNumThreadsUsed: " << totalNumThreadsUsed;
  if (totalNumThreadsUsed == 0 || totalNumThreadsUsed > totalNumThreads) {
    LDBG() << "--Too few threads given the required vector size -> FAIL";
    return Status::Invalid;
  }

```
- **EN**: Implements logic around `LDBG`, `product`; this block implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `LDBG`, `product` 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子。

### Lines 239-246
```cpp
  this->vectorSize = desiredVectorSize;
  this->numThreads = inferredNumThreads;
  if (totalNumThreadsUsed == totalNumThreads)
    return Status::Success;

  return Status::RequiresPredication;
}

```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 247-254
```cpp
void transform::gpu::CopyMappingInfo::print(llvm::raw_ostream &os) const {
  os << "MappingInfo{"
     << "CopyMappingInfo: " << "valid: " << (status != Status::Invalid) << ", "
     << "vectorSize: " << vectorSize << ", numThreads: {"
     << llvm::interleaved(numThreads) << "}, smallestBoundingTileSizes: {"
     << llvm::interleaved(smallestBoundingTileSizes) << "}, threadMapping: {"
     << llvm::interleaved(threadMapping) << "}}";
}
```
- **EN**: Implements logic around `print`, `interleaved`; this block implements verifier, folding, parsing, or printing hooks; encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `print`, `interleaved` 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子，并编码加速器专用执行或 lowering 规则。

## Key Concepts / 关键概念

- **Transform dialect integration / Transform Dialect 集成**:
  - **EN**: Adds transform-dialect operations or extensions that steer other rewrites.
  - **CN**: 添加驱动其他重写的 Transform Dialect 操作或扩展。
- **Structured tensor ops / 结构化张量操作**:
  - **EN**: Represents loop-nest-like structured computations over tensors or buffers.
  - **CN**: 表示在张量或缓冲区上执行的类循环嵌套结构化计算。
- **GPU execution model / GPU 执行模型**:
  - **EN**: Models kernels, memory spaces, launch geometry, and accelerator-specific rewrites.
  - **CN**: 建模 kernel、内存空间、启动几何以及加速器专用重写。
- **Type/attribute storage / 类型/属性存储**:
  - **EN**: Uses uniqued storage and parser/printer hooks for extensible compile-time metadata.
  - **CN**: 使用唯一化存储以及解析/打印钩子来表示可扩展的编译期元数据。
- **LLVM interop / LLVM 互操作**:
  - **EN**: Bridges MLIR concepts to LLVM-compatible data structures, intrinsics, or codegen expectations.
  - **CN**: 把 MLIR 概念桥接到 LLVM 兼容的数据结构、intrinsic 或代码生成预期。
- **Streaming output / 流式输出**:
  - **EN**: Writes diagnostics or assembly-like textual forms through LLVM/MLIR stream APIs.
  - **CN**: 通过 LLVM/MLIR 流式 API 输出诊断或类汇编文本。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Linalg/TransformOps/GPUHeuristics.h`, `mlir/Dialect/GPU/IR/GPUDialect.h`, `llvm/ADT/ArrayRef.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/SmallVectorExtras.h`, `llvm/Support/Debug.h`, `llvm/Support/DebugLog.h`, `llvm/Support/InterleavedRange.h`, `llvm/Support/MathExtras.h`, `llvm/Support/raw_ostream.h`
- **Standard-library headers / 标准库头文件**: `<cmath>`, `<numeric>`
- **Subsystem categories / 子系统类别**: LLVM support-library helpers / LLVM Support 库辅助功能 (5), LLVM ADT containers and utility types / LLVM ADT 容器与工具类型 (3), other MLIR dialect declarations / 其他 MLIR 方言声明 (2)
