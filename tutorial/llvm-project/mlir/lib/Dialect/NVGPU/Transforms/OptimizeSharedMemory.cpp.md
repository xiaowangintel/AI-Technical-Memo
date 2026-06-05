# OptimizeSharedMemory.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/NVGPU/Transforms/OptimizeSharedMemory.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file implements transforms to optimize accesses to shared memory.
  - **CN**: 该文件位于 `mlir/lib/Dialect/NVGPU/Transforms`，围绕 NVGPU 方言实现相关能力。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- OptimizeSharedMemory.cpp - MLIR NVGPU pass implementation ----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 8-14
```cpp
//
// This file implements transforms to optimize accesses to shared memory.
//
//===----------------------------------------------------------------------===//

#include "mlir/Dialect/NVGPU/Transforms/Passes.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/NVGPU/Transforms/Passes.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/NVGPU/Transforms/Passes.h`。

### Lines 15-24
```cpp
#include "mlir/Dialect/Arith/IR/Arith.h"
#include "mlir/Dialect/MemRef/IR/MemRef.h"
#include "mlir/Dialect/NVGPU/IR/NVGPUDialect.h"
#include "mlir/Dialect/NVGPU/Transforms/Transforms.h"
#include "mlir/Dialect/NVGPU/Transforms/Utils.h"
#include "mlir/Dialect/Vector/IR/VectorOps.h"
#include "mlir/Interfaces/SideEffectInterfaces.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/Support/MathExtras.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/MemRef/IR/MemRef.h`, `mlir/Dialect/NVGPU/IR/NVGPUDialect.h`, `mlir/Dialect/NVGPU/Transforms/Transforms.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/MemRef/IR/MemRef.h`, `mlir/Dialect/NVGPU/IR/NVGPUDialect.h`, `mlir/Dialect/NVGPU/Transforms/Transforms.h`。

### Lines 25-31
```cpp
namespace mlir {
namespace nvgpu {
#define GEN_PASS_DEF_OPTIMIZESHAREDMEMORY
#include "mlir/Dialect/NVGPU/Transforms/Passes.h.inc"
} // namespace nvgpu
} // namespace mlir

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/NVGPU/Transforms/Passes.h.inc`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/NVGPU/Transforms/Passes.h.inc`。

### Lines 32-38
```cpp
using namespace mlir;
using namespace mlir::nvgpu;

/// The size of a shared memory line according to NV documentation.
constexpr int64_t kSharedMemoryLineSizeBytes = 128;
/// We optimize for 128bit accesses, but this can be made an argument in the
/// future.
```
- **EN**: Introduces declarations for `mlir`, `mlir::nvgpu`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `mlir`, `mlir::nvgpu` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 39-45
```cpp
constexpr int64_t kDefaultVectorSizeBits = 128;

/// Uses `srcIndexValue` to permute `tgtIndexValue` via
/// `result = xor(floordiv(srcIdxVal,permuteEveryN),
///               floordiv(tgtIdxVal,vectorSize)))
///            + tgtIdxVal % vectorSize`
/// This is done using an optimized sequence of `arith` operations.
```
- **EN**: Contains supporting implementation details for the surrounding MLIR dialect component.
- **CN**: 包含周边 MLIR 方言组件所需的辅助实现细节。

### Lines 46-52
```cpp
static Value permuteVectorOffset(OpBuilder &b, Location loc,
                                 ArrayRef<Value> indices, MemRefType memrefTy,
                                 int64_t srcDim, int64_t tgtDim) {
  // Adjust the src index to change how often the permutation changes
  // if necessary.
  Value src = indices[srcDim];

```
- **EN**: Implements logic around `permuteVectorOffset`.
- **CN**: 围绕 `permuteVectorOffset` 实现具体逻辑。

### Lines 53-59
```cpp
  // We only want to permute every N iterations of the target dim where N is
  // ceil(sharedMemoryLineSizeBytes / dimSizeBytes(tgtDim)).
  const int64_t permuteEveryN = std::max<int64_t>(
      1, kSharedMemoryLineSizeBytes / ((memrefTy.getDimSize(tgtDim) *
                                        memrefTy.getElementTypeBitWidth()) /
                                       8));

```
- **EN**: Implements logic around `max`, `getDimSize`, `getElementTypeBitWidth`.
- **CN**: 围绕 `max`, `getDimSize`, `getElementTypeBitWidth` 实现具体逻辑。

### Lines 60-72
```cpp
  // clang-format off
  // Index bit representation (b0 = least significant bit) for dim(1)
  // of a `memref<?x?xDT>` is as follows:
  // N := log2(128/elementSizeBits)
  // M := log2(dimSize(1))
  // then
  // bits[0:N] = sub-vector element offset
  // bits[N:M] = vector index
  // clang-format on
  int64_t n =
      llvm::Log2_64(kDefaultVectorSizeBits / memrefTy.getElementTypeBitWidth());
  int64_t m = llvm::Log2_64(memrefTy.getDimSize(tgtDim));

```
- **EN**: Implements logic around `Log2_64`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `Log2_64` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 73-79
```cpp
  // Capture bits[0:(M-N)] of src by first creating a (M-N) mask.
  int64_t mask = (1LL << (m - n)) - 1;
  if (permuteEveryN > 1)
    mask = mask << llvm::Log2_64(permuteEveryN);
  Value srcBits = arith::ConstantIndexOp::create(b, loc, mask);
  srcBits = arith::AndIOp::create(b, loc, src, srcBits);

```
- **EN**: Implements logic around `Log2_64`, `create`.
- **CN**: 围绕 `Log2_64`, `create` 实现具体逻辑。

### Lines 80-93
```cpp
  // Use the src bits to permute the target bits b[N:M] containing the
  // vector offset.
  if (permuteEveryN > 1) {
    int64_t shlBits = n - llvm::Log2_64(permuteEveryN);
    if (shlBits > 0) {
      Value finalShiftVal = arith::ConstantIndexOp::create(b, loc, shlBits);
      srcBits = b.createOrFold<arith::ShLIOp>(loc, srcBits, finalShiftVal);
    } else if (shlBits < 0) {
      Value finalShiftVal =
          arith::ConstantIndexOp::create(b, loc, -1 * shlBits);
      srcBits = b.createOrFold<arith::ShRUIOp>(loc, srcBits, finalShiftVal);
    }
  } else {
    Value finalShiftVal = arith::ConstantIndexOp::create(b, loc, n);
```
- **EN**: Implements logic around `Log2_64`, `create`, `ShLIOp>`, `ShRUIOp>`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `Log2_64`, `create`, `ShLIOp>`, `ShRUIOp>` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 94-101
```cpp
    srcBits = b.createOrFold<arith::ShLIOp>(loc, srcBits, finalShiftVal);
  }

  Value permutedVectorIdx =
      arith::XOrIOp::create(b, loc, indices[tgtDim], srcBits);
  return permutedVectorIdx;
}

```
- **EN**: Implements logic around `ShLIOp>`, `create`.
- **CN**: 围绕 `ShLIOp>`, `create` 实现具体逻辑。

### Lines 102-109
```cpp
static void transformIndices(OpBuilder &builder, Location loc,
                             SmallVector<Value, 4> &indices,
                             MemRefType memrefTy, int64_t srcDim,
                             int64_t tgtDim) {
  indices[tgtDim] =
      permuteVectorOffset(builder, loc, indices, memrefTy, srcDim, tgtDim);
}

```
- **EN**: Implements logic around `transformIndices`, `permuteVectorOffset`.
- **CN**: 围绕 `transformIndices`, `permuteVectorOffset` 实现具体逻辑。

### Lines 110-123
```cpp
/// Return all operations within `parentOp` that read from or write to
/// `shmMemRef`.
static LogicalResult
getShmReadAndWriteOps(Operation *parentOp, Value shmMemRef,
                      SmallVector<Operation *, 16> &readOps,
                      SmallVector<Operation *, 16> &writeOps) {
  parentOp->walk([&](Operation *op) {
    MemoryEffectOpInterface iface = dyn_cast<MemoryEffectOpInterface>(op);
    if (!iface)
      return;
    std::optional<MemoryEffects::EffectInstance> effect =
        iface.getEffectOnValue<MemoryEffects::Read>(shmMemRef);
    if (effect) {
      readOps.push_back(op);
```
- **EN**: Implements logic around `getShmReadAndWriteOps`, `walk`, `Read>`, `push_back`.
- **CN**: 围绕 `getShmReadAndWriteOps`, `walk`, `Read>`, `push_back` 实现具体逻辑。

### Lines 124-130
```cpp
      return;
    }
    effect = iface.getEffectOnValue<MemoryEffects::Write>(shmMemRef);
    if (effect)
      writeOps.push_back(op);
  });

```
- **EN**: Implements logic around `Write>`, `push_back`.
- **CN**: 围绕 `Write>`, `push_back` 实现具体逻辑。

### Lines 131-144
```cpp
  // Restrict to a supported set of ops. We also require at least 2D access,
  // although this could be relaxed.
  if (llvm::any_of(readOps, [](Operation *op) {
        return !isa<memref::LoadOp, vector::LoadOp, nvgpu::LdMatrixOp>(op) ||
               getIndices(op).size() < 2;
      }))
    return failure();
  if (llvm::any_of(writeOps, [](Operation *op) {
        return !isa<memref::StoreOp, vector::StoreOp, nvgpu::DeviceAsyncCopyOp>(
                   op) ||
               getIndices(op).size() < 2;
      }))
    return failure();

```
- **EN**: Implements logic around `any_of`, `LdMatrixOp>`, `getIndices`, `failure`, and 1 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `any_of`, `LdMatrixOp>`, `getIndices`, `failure`, and 1 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 145-154
```cpp
  return success();
}

llvm::LogicalResult
mlir::nvgpu::optimizeSharedMemoryReadsAndWrites(Operation *parentOp,
                                                Value memrefValue) {
  auto memRefType = dyn_cast<MemRefType>(memrefValue.getType());
  if (!memRefType || !NVGPUDialect::hasSharedMemoryAddressSpace(memRefType))
    return failure();

```
- **EN**: Implements logic around `success`, `optimizeSharedMemoryReadsAndWrites`, `getType`, `hasSharedMemoryAddressSpace`, and 1 more symbols.
- **CN**: 围绕 `success`, `optimizeSharedMemoryReadsAndWrites`, `getType`, `hasSharedMemoryAddressSpace`, and 1 more symbols 实现具体逻辑。

### Lines 155-163
```cpp
  // Not support 0D MemRefs.
  if (memRefType.getRank() == 0)
    return failure();

  // Only support memrefs with scalar element types (i.e., int or float).
  // Memrefs with vector element types are not supported.
  if (!memRefType.getElementType().isIntOrFloat())
    return failure();

```
- **EN**: Implements logic around `getRank`, `failure`, `getElementType`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getRank`, `failure`, `getElementType` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 164-170
```cpp
  // Abort if the given value has any sub-views; we do not do any alias
  // analysis.
  bool hasSubView = false;
  parentOp->walk([&](memref::SubViewOp subView) { hasSubView = true; });
  if (hasSubView)
    return failure();

```
- **EN**: Implements logic around `walk`, `failure`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `walk`, `failure` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 171-184
```cpp
  // Check if this is necessary given the assumption of 128b accesses:
  // If dim[rank-1] is small enough to fit 8 rows in a 128B line.
  const int64_t rowSize = memRefType.getDimSize(memRefType.getRank() - 1);
  if (ShapedType::isDynamic(rowSize) || rowSize == 0)
    return failure();

  const int64_t rowsPerLine =
      (8 * kSharedMemoryLineSizeBytes / memRefType.getElementTypeBitWidth()) /
      rowSize;
  const int64_t threadGroupSize =
      1LL << (7 - llvm::Log2_64(kDefaultVectorSizeBits / 8));
  if (rowsPerLine >= threadGroupSize)
    return failure();

```
- **EN**: Implements logic around `getDimSize`, `isDynamic`, `failure`, `getElementTypeBitWidth`, and 1 more symbols.
- **CN**: 围绕 `getDimSize`, `isDynamic`, `failure`, `getElementTypeBitWidth`, and 1 more symbols 实现具体逻辑。

### Lines 185-192
```cpp
  // Get sets of operations within the function that read/write to shared
  // memory.
  SmallVector<Operation *, 16> shmReadOps;
  SmallVector<Operation *, 16> shmWriteOps;
  if (failed(getShmReadAndWriteOps(parentOp, memrefValue, shmReadOps,
                                   shmWriteOps)))
    return failure();

```
- **EN**: Implements logic around `failed`, `failure`.
- **CN**: 围绕 `failed`, `failure` 实现具体逻辑。

### Lines 193-200
```cpp
  if (shmReadOps.empty() || shmWriteOps.empty())
    return failure();

  OpBuilder builder(parentOp->getContext());

  int64_t tgtDim = memRefType.getRank() - 1;
  int64_t srcDim = memRefType.getRank() - 2;

```
- **EN**: Implements logic around `empty`, `failure`, `builder`, `getRank`.
- **CN**: 围绕 `empty`, `failure`, `builder`, `getRank` 实现具体逻辑。

### Lines 201-213
```cpp
  // Transform indices for the ops writing to shared memory.
  while (!shmWriteOps.empty()) {
    Operation *shmWriteOp = shmWriteOps.back();
    shmWriteOps.pop_back();
    builder.setInsertionPoint(shmWriteOp);

    auto indices = getIndices(shmWriteOp);
    SmallVector<Value, 4> transformedIndices(indices.begin(), indices.end());
    transformIndices(builder, shmWriteOp->getLoc(), transformedIndices,
                     memRefType, srcDim, tgtDim);
    setIndices(shmWriteOp, transformedIndices);
  }

```
- **EN**: Implements logic around `empty`, `back`, `pop_back`, `setInsertionPoint`, and 4 more symbols.
- **CN**: 围绕 `empty`, `back`, `pop_back`, `setInsertionPoint`, and 4 more symbols 实现具体逻辑。

### Lines 214-226
```cpp
  // Transform indices for the ops reading from shared memory.
  while (!shmReadOps.empty()) {
    Operation *shmReadOp = shmReadOps.back();
    shmReadOps.pop_back();
    builder.setInsertionPoint(shmReadOp);

    auto indices = getIndices(shmReadOp);
    SmallVector<Value, 4> transformedIndices(indices.begin(), indices.end());
    transformIndices(builder, shmReadOp->getLoc(), transformedIndices,
                     memRefType, srcDim, tgtDim);
    setIndices(shmReadOp, transformedIndices);
  }

```
- **EN**: Implements logic around `empty`, `back`, `pop_back`, `setInsertionPoint`, and 4 more symbols.
- **CN**: 围绕 `empty`, `back`, `pop_back`, `setInsertionPoint`, and 4 more symbols 实现具体逻辑。

### Lines 227-233
```cpp
  return success();
}

namespace {
class OptimizeSharedMemoryPass
    : public nvgpu::impl::OptimizeSharedMemoryBase<OptimizeSharedMemoryPass> {
public:
```
- **EN**: Introduces declarations for `OptimizeSharedMemoryPass`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `OptimizeSharedMemoryPass` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 234-247
```cpp
  OptimizeSharedMemoryPass() = default;

  void runOnOperation() override {
    Operation *op = getOperation();
    SmallVector<memref::AllocOp> shmAllocOps;
    op->walk([&](memref::AllocOp allocOp) {
      if (!NVGPUDialect::hasSharedMemoryAddressSpace(allocOp.getType()))
        return;
      shmAllocOps.push_back(allocOp);
    });
    for (auto allocOp : shmAllocOps) {
      if (failed(optimizeSharedMemoryReadsAndWrites(getOperation(),
                                                    allocOp.getMemref())))
        return;
```
- **EN**: Implements logic around `OptimizeSharedMemoryPass`, `runOnOperation`, `getOperation`, `walk`, and 4 more symbols; this block packages logic as an MLIR pass or pass helper; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `OptimizeSharedMemoryPass`, `runOnOperation`, `getOperation`, `walk`, and 4 more symbols 实现具体逻辑；该代码块将逻辑组织为 MLIR Pass 或 Pass 辅助组件，并协调核心结构化 MLIR 方言之间的行为。

### Lines 248-255
```cpp
    }
  }
};
} // namespace

std::unique_ptr<Pass> mlir::nvgpu::createOptimizeSharedMemoryPass() {
  return std::make_unique<OptimizeSharedMemoryPass>();
}
```
- **EN**: Implements logic around `createOptimizeSharedMemoryPass`, `make_unique`; this block packages logic as an MLIR pass or pass helper.
- **CN**: 围绕 `createOptimizeSharedMemoryPass`, `make_unique` 实现具体逻辑；该代码块将逻辑组织为 MLIR Pass 或 Pass 辅助组件。

## Key Concepts / 关键概念

- **Rewrite and lowering / 重写与 lowering**:
  - **EN**: Uses MLIR pattern infrastructure or passes to canonicalize, legalize, or lower operations.
  - **CN**: 使用 MLIR 模式基础设施或 Pass 对操作进行规范化、合法化或 lowering。
- **Pass infrastructure / Pass 基础设施**:
  - **EN**: Wraps transformations as reusable passes that can be scheduled in MLIR pipelines.
  - **CN**: 将变换封装为可在 MLIR 流水线中调度的可复用 Pass。
- **Interface-based extensibility / 基于接口的可扩展性**:
  - **EN**: Models reusable capabilities that can be queried across dialect boundaries.
  - **CN**: 建模可跨方言查询的可复用能力。
- **Structured IR coordination / 结构化 IR 协同**:
  - **EN**: Interacts with structured MLIR dialects that model loops, tensors, memory, and vector semantics.
  - **CN**: 与建模循环、张量、内存和向量语义的结构化 MLIR 方言交互。
- **Target-specific semantics / 目标相关语义**:
  - **EN**: Encodes rules tied to accelerator, GPU, or binary target environments.
  - **CN**: 编码与加速器、GPU 或二进制目标环境绑定的规则。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/NVGPU/Transforms/Passes.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/MemRef/IR/MemRef.h`, `mlir/Dialect/NVGPU/IR/NVGPUDialect.h`, `mlir/Dialect/NVGPU/Transforms/Transforms.h`, `mlir/Dialect/NVGPU/Transforms/Utils.h`, `mlir/Dialect/Vector/IR/VectorOps.h`, `mlir/Interfaces/SideEffectInterfaces.h`, `llvm/ADT/STLExtras.h`, `llvm/Support/MathExtras.h` ... (+1 more)
- **Subsystem categories / 子系统类别**: dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (8), cross-dialect interfaces and capability contracts / 跨方言接口与能力约定 (1), LLVM ADT containers and low-level helpers / LLVM ADT 容器与底层辅助工具 (1), LLVM support-library facilities / LLVM Support 库设施 (1)
