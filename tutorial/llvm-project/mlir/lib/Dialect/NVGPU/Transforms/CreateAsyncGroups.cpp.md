# CreateAsyncGroups.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/NVGPU/Transforms/CreateAsyncGroups.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements NVGPU dialect support for rewrite patterns, passes, and IR-to-IR transformation logic, centered on `CreateAsyncGroups`.
  - **CN**: 实现 NVGPU 方言中围绕 `CreateAsyncGroups` 的重写模式、Pass 以及 IR 到 IR 的变换逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- CreateAsyncGroups.cpp - Create async device copies -----------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 8-17
```cpp

#include "mlir/Dialect/NVGPU/Transforms/Transforms.h"

#include "mlir/Dialect/Arith/IR/Arith.h"
#include "mlir/Dialect/NVGPU/IR/NVGPUDialect.h"
#include "mlir/Dialect/NVGPU/Transforms/Utils.h"
#include "mlir/Dialect/Vector/IR/VectorOps.h"
#include "mlir/IR/BuiltinAttributes.h"
#include "mlir/IR/BuiltinTypes.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/NVGPU/Transforms/Transforms.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/NVGPU/IR/NVGPUDialect.h`, `mlir/Dialect/NVGPU/Transforms/Utils.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/NVGPU/Transforms/Transforms.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/NVGPU/IR/NVGPUDialect.h`, `mlir/Dialect/NVGPU/Transforms/Utils.h`。

### Lines 18-29
```cpp
using namespace mlir;

/// Return "true" if the given vector transfer op is contiguous and suitable
/// for replacement with an async copy.
template <typename OpTy>
static bool isContiguousXferOp(OpTy op) {
  return op.getPermutationMap().isMinorIdentity() && op.isDimInBounds(0) &&
         op.hasPureBufferSemantics() &&
         cast<MemRefType>(nvgpu::getMemrefOperand(op).getType())
             .isLastDimUnitStride();
}

```
- **EN**: Introduces declarations for `mlir`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `mlir` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 30-38
```cpp
/// Return "true" if the given op is a contiguous and suitable
/// vector.transfer_write or vector.store op.
static bool isContiguousStore(Operation *write) {
  if (auto transferWrite = dyn_cast<vector::TransferWriteOp>(write))
    return isContiguousXferOp(transferWrite) && !transferWrite.getMask();
  // vector.store are always contiguous.
  return isa<vector::StoreOp>(write);
}

```
- **EN**: Implements logic around `isContiguousStore`, `TransferWriteOp>`, `isContiguousXferOp`, `StoreOp>`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `isContiguousStore`, `TransferWriteOp>`, `isContiguousXferOp`, `StoreOp>` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 39-47
```cpp
/// Return "true" if the given op is a contiguous and suitable
/// vector.transfer_read or vector.load op.
static bool isContiguousRead(Operation *read) {
  if (auto transferRead = dyn_cast<vector::TransferReadOp>(read))
    return isContiguousXferOp(transferRead);
  // vector.load are always contiguous.
  return isa<vector::LoadOp>(read);
}

```
- **EN**: Implements logic around `isContiguousRead`, `TransferReadOp>`, `isContiguousXferOp`, `LoadOp>`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `isContiguousRead`, `TransferReadOp>`, `isContiguousXferOp`, `LoadOp>` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 48-55
```cpp
namespace {
/// A vector.create_mask op and extract position.
struct TransferMask {
  vector::CreateMaskOp createMaskOp;
  SmallVector<int64_t> extractPosition;
};
} // namespace

```
- **EN**: Introduces declarations for `TransferMask`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `TransferMask` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 56-64
```cpp
/// If the given vector load op has a mask that is defined by
/// vector.create_mask, return that op.
static FailureOr<TransferMask> getMaskOp(Operation *loadOp) {
  auto transferRead = dyn_cast<vector::TransferReadOp>(loadOp);
  if (!transferRead || !transferRead.getMask())
    return TransferMask{{}, {}};
  assert(transferRead.getMask().getType().getRank() == 1 &&
         "expected 1-D mask");

```
- **EN**: Implements logic around `getMaskOp`, `TransferReadOp>`, `getMask`, `assert`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getMaskOp`, `TransferReadOp>`, `getMask`, `assert` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 65-77
```cpp
  // Case 1: Mask is the result of a vector.create_mask.
  if (auto maskOp =
          transferRead.getMask().getDefiningOp<vector::CreateMaskOp>())
    return TransferMask{maskOp, {}};

  // Case 2: Mask is the result of a vector.extract(vector.create_mask).
  if (auto extractOp =
          transferRead.getMask().getDefiningOp<vector::ExtractOp>())
    if (auto maskOp =
            extractOp.getSource().getDefiningOp<vector::CreateMaskOp>())
      return TransferMask{maskOp,
                          SmallVector<int64_t>(extractOp.getStaticPosition())};

```
- **EN**: Implements logic around `getMask`, `getSource`, `SmallVector`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getMask`, `getSource`, `SmallVector` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 78-87
```cpp
  // All other cases: not supported.
  return failure();
}

/// Build an SSA value that represents the number of read elements.
static Value buildNumReadElements(OpBuilder &b, Location loc,
                                  Operation *readOp) {
  FailureOr<TransferMask> transferMask = getMaskOp(readOp);
  assert(succeeded(transferMask) && "invalid transfer mask");

```
- **EN**: Implements logic around `failure`, `buildNumReadElements`, `getMaskOp`, `assert`.
- **CN**: 围绕 `failure`, `buildNumReadElements`, `getMaskOp`, `assert` 实现具体逻辑。

### Lines 88-98
```cpp
  // No mask => no num_read_elements.
  if (!transferMask->createMaskOp)
    return Value();

  // No extract: return size of "ones" segment in the mask.
  if (transferMask->extractPosition.empty()) {
    assert(transferMask->createMaskOp.getNumOperands() == 1 &&
           "expected single operand");
    return transferMask->createMaskOp.getOperand(0);
  }

```
- **EN**: Implements logic around `Value`, `empty`, `assert`, `getOperand`.
- **CN**: 围绕 `Value`, `empty`, `assert`, `getOperand` 实现具体逻辑。

### Lines 99-112
```cpp
  // vector.extract(vector.create_mask).
  // If extract_pos < num_ones, take number of elements from the least
  // significant dimension. (Do this for all dimensions and bit-AND the
  // conditions.)
  assert(transferMask->createMaskOp.getVectorType().getRank() -
                 transferMask->extractPosition.size() ==
             1 &&
         "expected N-D -> (N-1)-D extract");
  Value cond;
  // Note: There is one more `sz` than `pos`. The loop end with the last `pos`.
  for (auto [pos, sz] : llvm::zip(transferMask->extractPosition,
                                  transferMask->createMaskOp->getOperands())) {
    Value cmp =
        arith::CmpIOp::create(b, loc, arith::CmpIPredicate::slt,
```
- **EN**: Implements logic around `assert`, `size`, `zip`, `getOperands`, and 1 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `assert`, `size`, `zip`, `getOperands`, and 1 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 113-124
```cpp
                              arith::ConstantIndexOp::create(b, loc, pos), sz);
    if (!cond) {
      cond = cmp;
      continue;
    }
    cond = arith::AndIOp::create(b, loc, cmp, cond);
  }
  return arith::SelectOp::create(
      b, loc, cond, transferMask->createMaskOp->getOperands().back(),
      arith::ConstantIndexOp::create(b, loc, 0));
}

```
- **EN**: Implements logic around `create`, `getOperands`.
- **CN**: 围绕 `create`, `getOperands` 实现具体逻辑。

### Lines 125-138
```cpp
/// Return "true" if the conversion to async copy is supported by "async copy".
static bool resultsInSupportedAsyncCopy(MemRefType memrefType,
                                        VectorType vecType) {
  assert(vecType.getRank() == 1 && "expected 1-D vector");
  constexpr int64_t kSupportedCpAsyncAlignmentsInBytes[3] = {4, 8, 16};

  // Condition 1: the copy size must be supported.
  bool supportedCopySize = false;
  int64_t numElements = vecType.getNumElements();
  Type elementType = vecType.getElementType();
  for (int64_t alignmentInBytes : kSupportedCpAsyncAlignmentsInBytes) {
    if (alignmentInBytes * 8 ==
        numElements * elementType.getIntOrFloatBitWidth()) {
      supportedCopySize = true;
```
- **EN**: Implements logic around `resultsInSupportedAsyncCopy`, `assert`, `getNumElements`, `getElementType`, and 1 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `resultsInSupportedAsyncCopy`, `assert`, `getNumElements`, `getElementType`, and 1 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 139-151
```cpp
      break;
    }
  }
  if (!supportedCopySize)
    return false;

  // TODO: Condition 2: the alignments must be supported. For cp.async the
  // NVIDIA doc (section 6.4.1) says: "The address must be naturally aligned to
  // a multiple of the access size. If an address is not properly aligned, the
  // resulting behavior is undefined.".
  return true;
}

```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 152-165
```cpp
void nvgpu::createAsyncGroups(RewriterBase &rewriter, Operation *op,
                              bool bypassL1) {
  llvm::SmallSetVector<Operation *, 16> copyToSharedMem;

  // Look for all the copy that can be converted to async copy ops.
  op->walk([&](Operation *writeOp) {
    // Look for contiguous 1D vector store into shared memory.
    if (!isContiguousStore(writeOp))
      return;
    Value vectorVal = nvgpu::getValueStored(writeOp);
    if (cast<VectorType>(vectorVal.getType()).getRank() != 1)
      return;
    Value storeBase = nvgpu::getMemrefOperand(writeOp);
    if (!nvgpu::NVGPUDialect::hasSharedMemoryAddressSpace(
```
- **EN**: Implements logic around `createAsyncGroups`, `walk`, `isContiguousStore`, `getValueStored`, and 3 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `createAsyncGroups`, `walk`, `isContiguousStore`, `getValueStored`, and 3 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 166-178
```cpp
            cast<MemRefType>(storeBase.getType())))
      return;

    // The stored vector must originate from a contiguous 1D vector load.
    Operation *readOp = vectorVal.getDefiningOp();
    if (readOp == nullptr || !isContiguousRead(readOp))
      return;
    Value loadBase = nvgpu::getMemrefOperand(readOp);
    // Should be reading from global memory (not shared memory).
    if (nvgpu::NVGPUDialect::hasSharedMemoryAddressSpace(
            cast<MemRefType>(loadBase.getType())))
      return;

```
- **EN**: Implements logic around `getType`, `getDefiningOp`, `isContiguousRead`, `getMemrefOperand`, and 1 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getType`, `getDefiningOp`, `isContiguousRead`, `getMemrefOperand`, and 1 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 179-189
```cpp
    // Look for compatible mask and padding.
    if (auto transferRead = dyn_cast<vector::TransferReadOp>(readOp)) {
      if (Value mask = transferRead.getMask()) {
        if (getConstantIntValue(transferRead.getPadding()) ==
            static_cast<int64_t>(0))
          return;
        if (failed(getMaskOp(readOp)))
          return;
      }
    }

```
- **EN**: Implements logic around `TransferReadOp>`, `getMask`, `getConstantIntValue`, `static_cast`, and 1 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `TransferReadOp>`, `getMask`, `getConstantIntValue`, `static_cast`, and 1 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 190-199
```cpp
    // Check whether both accesses are supported before we emit: this is
    // necessary to ensure the correctness of DeviceAsyncCopyOp.
    VectorType vecType = cast<VectorType>(vectorVal.getType());

    if (!resultsInSupportedAsyncCopy(cast<MemRefType>(loadBase.getType()),
                                     vecType) ||
        !resultsInSupportedAsyncCopy(cast<MemRefType>(storeBase.getType()),
                                     vecType))
      return;

```
- **EN**: Implements logic around `getType`, `resultsInSupportedAsyncCopy`.
- **CN**: 围绕 `getType`, `resultsInSupportedAsyncCopy` 实现具体逻辑。

### Lines 200-211
```cpp
    copyToSharedMem.insert(writeOp);
    return;
  });

  while (!copyToSharedMem.empty()) {
    // Start a group with the first write.
    SmallVector<Operation *> group;
    Operation *writeOp = *copyToSharedMem.begin();
    copyToSharedMem.remove(writeOp);
    group.push_back(writeOp);
    Operation *nextNode = writeOp;

```
- **EN**: Implements logic around `insert`, `empty`, `begin`, `remove`, and 1 more symbols.
- **CN**: 围绕 `insert`, `empty`, `begin`, `remove`, and 1 more symbols 实现具体逻辑。

### Lines 212-225
```cpp
    // Look in the next nodes for more copies to add to the same group.
    while ((nextNode = nextNode->getNextNode())) {
      // Ignore ops without side effects.
      auto memInterface = dyn_cast<MemoryEffectOpInterface>(nextNode);
      if (memInterface && memInterface.hasNoEffect() &&
          !nextNode->hasTrait<OpTrait::HasRecursiveMemoryEffects>())
        continue;
      // Ignore read from a different address space.
      if (isa<vector::TransferReadOp, vector::LoadOp>(nextNode)) {
        Operation *readOp = nextNode;
        Value memrefOperand = nvgpu::getMemrefOperand(readOp);
        if (!nvgpu::NVGPUDialect::hasSharedMemoryAddressSpace(
                cast<MemRefType>(memrefOperand.getType()))) {
          continue;
```
- **EN**: Implements logic around `getNextNode`, `hasNoEffect`, `HasRecursiveMemoryEffects>`, `LoadOp>`, and 3 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getNextNode`, `hasNoEffect`, `HasRecursiveMemoryEffects>`, `LoadOp>`, and 3 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 226-237
```cpp
        }
      }
      if (copyToSharedMem.count(nextNode)) {
        // Found another copy, add it to the group.
        copyToSharedMem.remove(nextNode);
        group.push_back(nextNode);
        continue;
      }
      // If the op is something else stop the accumulating op in the group.
      break;
    }

```
- **EN**: Implements logic around `count`, `remove`, `push_back`.
- **CN**: 围绕 `count`, `remove`, `push_back` 实现具体逻辑。

### Lines 238-251
```cpp
    // Emit the group.
    SmallVector<Value> tokens;
    for (Operation *writeOp : group) {
      rewriter.setInsertionPoint(writeOp);
      Value vectorVal = nvgpu::getValueStored(writeOp);
      auto vectorType = cast<VectorType>(vectorVal.getType());
      int64_t numElements = vectorType.getNumElements();
      Operation *readOp = vectorVal.getDefiningOp();
      Value storeBase = nvgpu::getMemrefOperand(writeOp);
      Value loadBase = nvgpu::getMemrefOperand(readOp);
      Value numReadElements =
          buildNumReadElements(rewriter, writeOp->getLoc(), readOp);
      auto dstMemref = cast<MemRefType>(storeBase.getType());
      int64_t sizeInBytes =
```
- **EN**: Implements logic around `setInsertionPoint`, `getValueStored`, `getType`, `getNumElements`, and 3 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `setInsertionPoint`, `getValueStored`, `getType`, `getNumElements`, and 3 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 252-265
```cpp
          (dstMemref.getElementTypeBitWidth() * numElements) / 8;
      // bypass_l1 only possible with 16 byte transfer.
      Value token = nvgpu::DeviceAsyncCopyOp::create(
          rewriter, writeOp->getLoc(),
          nvgpu::DeviceAsyncTokenType::get(op->getContext()),
          /*dst=*/storeBase, /*dstIndices=*/nvgpu::getIndices(writeOp),
          /*src=*/loadBase,
          /*srcIndices=*/nvgpu::getIndices(readOp),
          /*dstElements=*/rewriter.getIndexAttr(numElements),
          /*srcElements=*/numReadElements,
          /*bypassL1=*/bypassL1 && sizeInBytes == 16 ? rewriter.getUnitAttr()
                                                     : UnitAttr());
      tokens.push_back(token);
    }
```
- **EN**: Implements logic around `getElementTypeBitWidth`, `create`, `getLoc`, `get`, and 5 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `getElementTypeBitWidth`, `create`, `getLoc`, `get`, and 5 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 266-277
```cpp

    // Create the group and wait for it right after.
    Value groupToken = nvgpu::DeviceAsyncCreateGroupOp::create(
        rewriter, op->getLoc(),
        nvgpu::DeviceAsyncTokenType::get(op->getContext()), tokens);
    nvgpu::DeviceAsyncWaitOp::create(rewriter, op->getLoc(), groupToken,
                                     nullptr);
    // Clean up old stores.
    for (Operation *writeOp : group)
      rewriter.eraseOp(writeOp);
  }
}
```
- **EN**: Implements logic around `create`, `getLoc`, `get`, `eraseOp`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `create`, `getLoc`, `get`, `eraseOp` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

## Key Concepts / 关键概念

- **Rewrite and lowering / 重写与 lowering**:
  - **EN**: Uses MLIR pattern infrastructure or passes to canonicalize, legalize, or lower operations.
  - **CN**: 使用 MLIR 模式基础设施或 Pass 对操作进行规范化、合法化或 lowering。
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

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/NVGPU/Transforms/Transforms.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/NVGPU/IR/NVGPUDialect.h`, `mlir/Dialect/NVGPU/Transforms/Utils.h`, `mlir/Dialect/Vector/IR/VectorOps.h`, `mlir/IR/BuiltinAttributes.h`, `mlir/IR/BuiltinTypes.h`
- **Subsystem categories / 子系统类别**: dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (5), MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (2)
