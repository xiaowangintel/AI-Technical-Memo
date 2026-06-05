# ParallelLoopMapper.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/GPU/Transforms/ParallelLoopMapper.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file implements utilities to generate mappings for parallel loops to GPU devices.
  - **CN**: 实现 GPU 方言与异构加速器支持 的重写模式与变换 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- ParallelLoopMapper.cpp - Utilities for mapping parallel loops to GPU =//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件的高层描述。

### Lines 8-15
```cpp
//
// This file implements utilities to generate mappings for parallel loops to
// GPU devices.
//
//===----------------------------------------------------------------------===//

#include "mlir/Dialect/GPU/Transforms/Passes.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/GPU/Transforms/Passes.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/GPU/Transforms/Passes.h`。

### Lines 16-22
```cpp
#include "mlir/Dialect/Func/IR/FuncOps.h"
#include "mlir/Dialect/GPU/IR/GPUDialect.h"
#include "mlir/Dialect/GPU/Transforms/ParallelLoopMapper.h"
#include "mlir/Dialect/SCF/IR/SCF.h"
#include "mlir/IR/AffineMap.h"

namespace mlir {
```
- **EN**: Introduces declarations for `mlir`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `mlir` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 23-30
```cpp
#define GEN_PASS_DEF_GPUMAPPARALLELLOOPSPASS
#include "mlir/Dialect/GPU/Transforms/Passes.h.inc"
} // namespace mlir

namespace mlir {

using scf::ParallelOp;

```
- **EN**: Introduces declarations for `mlir`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `mlir` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 31-44
```cpp
StringRef gpu::getMappingAttrName() { return "mapping"; }

LogicalResult
gpu::setMappingAttr(ParallelOp ploopOp,
                    ArrayRef<ParallelLoopDimMappingAttr> mapping) {
  // Verify that each processor is mapped to only once.
  llvm::DenseSet<gpu::Processor> specifiedMappings;
  for (auto dimAttr : mapping) {
    gpu::Processor processor = dimAttr.getProcessor();
    if (processor != gpu::Processor::Sequential &&
        specifiedMappings.count(processor))
      return ploopOp.emitError(
          "invalid mapping multiple loops to same processor");
    specifiedMappings.insert(processor);
```
- **EN**: Implements logic around `getMappingAttrName`, `setMappingAttr`, `getProcessor`, `count`, and 2 more symbols; this block implements verifier, folding, parsing, or printing hooks; encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `getMappingAttrName`, `setMappingAttr`, `getProcessor`, `count`, and 2 more symbols 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子，并编码加速器专用执行或 lowering 规则。

### Lines 45-51
```cpp
  }
  ArrayRef<Attribute> mappingAsAttrs(mapping.data(), mapping.size());
  ploopOp->setAttr(getMappingAttrName(),
                   ArrayAttr::get(ploopOp.getContext(), mappingAsAttrs));
  return success();
}

```
- **EN**: Implements logic around `mappingAsAttrs`, `setAttr`, `get`, `success`.
- **CN**: 围绕 `mappingAsAttrs`, `setAttr`, `get`, `success` 实现具体逻辑。

### Lines 52-59
```cpp
namespace gpu {
namespace {
enum MappingLevel { MapGrid = 0, MapBlock = 1, Sequential = 2 };
enum class MappingPolicy { OutermostFirst, InnermostFirst };
} // namespace

static constexpr int kNumHardwareIds = 3;

```
- **EN**: Introduces declarations for `gpu`, `MappingLevel`, `MappingPolicy`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `gpu`, `MappingLevel`, `MappingPolicy` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 60-68
```cpp
/// Bounded increment on MappingLevel. Increments to the next
/// level unless Sequential was already reached.
static MappingLevel &operator++(MappingLevel &mappingLevel) {
  if (mappingLevel < Sequential) {
    mappingLevel = static_cast<MappingLevel>(mappingLevel + 1);
  }
  return mappingLevel;
}

```
- **EN**: Implements logic around `static_cast`.
- **CN**: 围绕 `static_cast` 实现具体逻辑。

### Lines 69-79
```cpp
// Map the policy string to a typed mapping policy.
// TODO: Revisit this and possibly use a loop interchange pass instead.
static FailureOr<MappingPolicy> getMappingPolicyFromStr(StringRef policy) {
  std::string policyCanonical = policy.trim().lower();

  std::optional<MappingPolicy> option =
      llvm::StringSwitch<std::optional<MappingPolicy>>(policyCanonical)
          .Case("innermost-first", MappingPolicy::InnermostFirst)
          .Case("outermost-first", MappingPolicy::OutermostFirst)
          .Default(std::nullopt);

```
- **EN**: Implements logic around `getMappingPolicyFromStr`, `trim`, `optional`, `Case`, and 1 more symbols.
- **CN**: 围绕 `getMappingPolicyFromStr`, `trim`, `optional`, `Case`, and 1 more symbols 实现具体逻辑。

### Lines 80-86
```cpp
  if (!option)
    return failure();
  return *option;
}

/// Computed the hardware id to use for a given mapping level. Will
/// assign x,y and z hardware ids for the first 3 dimensions and use
```
- **EN**: Implements logic around `failure`.
- **CN**: 围绕 `failure` 实现具体逻辑。

### Lines 87-100
```cpp
/// sequential after.
static Processor getHardwareIdForMapping(MappingLevel level, int dimension) {

  if (dimension >= kNumHardwareIds || level == Sequential)
    return Processor::Sequential;

  switch (level) {
  case MapGrid:
    switch (dimension) {
    case 0:
      return Processor::BlockX;
    case 1:
      return Processor::BlockY;
    case 2:
```
- **EN**: Implements logic around `getHardwareIdForMapping`.
- **CN**: 围绕 `getHardwareIdForMapping` 实现具体逻辑。

### Lines 101-114
```cpp
      return Processor::BlockZ;
    default:
      return Processor::Sequential;
    }
    break;
  case MapBlock:
    switch (dimension) {
    case 0:
      return Processor::ThreadX;
    case 1:
      return Processor::ThreadY;
    case 2:
      return Processor::ThreadZ;
    default:
```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 115-121
```cpp
      return Processor::Sequential;
    }
  default:;
  }
  return Processor::Sequential;
}

```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 122-132
```cpp
/// Add mapping information to the given parallel loop. Do not add
/// mapping information if the loop already has it. Also, don't
/// start a mapping at a nested loop.
static void
mapParallelOp(ParallelOp parallelOp, MappingLevel mappingLevel = MapGrid,
              MappingPolicy mappingPolicy = MappingPolicy::OutermostFirst) {
  // Do not try to add a mapping to already mapped loops or nested loops.
  if (parallelOp->getAttr(getMappingAttrName()) ||
      ((mappingLevel == MapGrid) && parallelOp->getParentOfType<ParallelOp>()))
    return;

```
- **EN**: Implements logic around `mapParallelOp`, `getAttr`, `getParentOfType`.
- **CN**: 围绕 `mapParallelOp`, `getAttr`, `getParentOfType` 实现具体逻辑。

### Lines 133-140
```cpp
  const int numLoops = static_cast<int>(parallelOp.getNumLoops());
  const int loopsToMap = std::min(numLoops, kNumHardwareIds);

  MLIRContext *ctx = parallelOp.getContext();
  Builder b(ctx);
  SmallVector<ParallelLoopDimMappingAttr, 4> attrs;
  attrs.reserve(numLoops);

```
- **EN**: Implements logic around `static_cast`, `min`, `getContext`, `b`, and 1 more symbols.
- **CN**: 围绕 `static_cast`, `min`, `getContext`, `b`, and 1 more symbols 实现具体逻辑。

### Lines 141-151
```cpp
  for (int i = 0; i < numLoops; ++i) {

    // Determine the mapping to use for this loop.
    // If the are more loops to map than HW IDs map to sequential.
    int hwMapping = kNumHardwareIds;
    if (i < loopsToMap) {
      hwMapping = (mappingPolicy == MappingPolicy::OutermostFirst)
                      ? i
                      : (loopsToMap - 1 - i);
    }

```
- **EN**: Contains supporting implementation details for the surrounding MLIR dialect component.
- **CN**: 包含周边 MLIR 方言组件所需的辅助实现细节。

### Lines 152-165
```cpp
    attrs.push_back(b.getAttr<ParallelLoopDimMappingAttr>(
        getHardwareIdForMapping(mappingLevel, hwMapping), b.getDimIdentityMap(),
        b.getDimIdentityMap()));
  }
  (void)setMappingAttr(parallelOp, attrs);
  ++mappingLevel;
  // Parallel loop operations are immediately nested, so do not use
  // walk but just iterate over the operations.
  for (Operation &op : *parallelOp.getBody()) {
    if (ParallelOp nested = dyn_cast<ParallelOp>(op))
      mapParallelOp(nested, mappingLevel, mappingPolicy);
  }
}

```
- **EN**: Implements logic around `push_back`, `getHardwareIdForMapping`, `getDimIdentityMap`, `setMappingAttr`, and 3 more symbols.
- **CN**: 围绕 `push_back`, `getHardwareIdForMapping`, `getDimIdentityMap`, `setMappingAttr`, and 3 more symbols 实现具体逻辑。

### Lines 166-179
```cpp
namespace {
struct GpuMapParallelLoopsPass
    : public impl::GpuMapParallelLoopsPassBase<GpuMapParallelLoopsPass> {
  using Base::Base;

  void runOnOperation() override {
    // Parse the mapping policy.
    FailureOr<MappingPolicy> policyOrFailure =
        getMappingPolicyFromStr(mappingPolicyStr);
    if (failed(policyOrFailure)) {
      getOperation()->emitError() << "Invalid mapping policy specified.";
      return signalPassFailure();
    }

```
- **EN**: Introduces declarations for `GpuMapParallelLoopsPass`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `GpuMapParallelLoopsPass` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 180-190
```cpp
    MappingPolicy policy = *policyOrFailure;
    MappingLevel topLevel = MappingLevel::MapGrid;

    for (Region &region : getOperation()->getRegions()) {
      region.walk([&](ParallelOp parallelOp) {
        mapParallelOp(parallelOp, topLevel, policy);
      });
    }
  }
};

```
- **EN**: Implements logic around `getOperation`, `walk`, `mapParallelOp`; this block manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `getOperation`, `walk`, `mapParallelOp` 实现具体逻辑；该代码块处理 MLIR region、block 或控制流边。

### Lines 191-193
```cpp
} // namespace
} // namespace gpu
} // namespace mlir
```
- **EN**: Contains supporting implementation details for the surrounding MLIR dialect component.
- **CN**: 包含周边 MLIR 方言组件所需的辅助实现细节。

## Key Concepts / 关键概念

- **Pattern rewriting and passes / 模式重写与 Pass**:
  - **EN**: Applies conversion patterns, canonicalizations, or pass pipelines over MLIR operations.
  - **CN**: 对 MLIR 操作应用转换模式、规范化或 pass 流水线。
- **GPU execution model / GPU 执行模型**:
  - **EN**: Models kernels, memory spaces, launch geometry, and accelerator-specific rewrites.
  - **CN**: 建模 kernel、内存空间、启动几何以及加速器专用重写。
- **Type/attribute storage / 类型/属性存储**:
  - **EN**: Uses uniqued storage and parser/printer hooks for extensible compile-time metadata.
  - **CN**: 使用唯一化存储以及解析/打印钩子来表示可扩展的编译期元数据。
- **Region-based control flow / 基于 Region 的控制流**:
  - **EN**: Represents nested blocks and successors as first-class IR structure.
  - **CN**: 把嵌套 block 与后继边表示为一等 IR 结构。
- **LLVM interop / LLVM 互操作**:
  - **EN**: Bridges MLIR concepts to LLVM-compatible data structures, intrinsics, or codegen expectations.
  - **CN**: 把 MLIR 概念桥接到 LLVM 兼容的数据结构、intrinsic 或代码生成预期。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/GPU/Transforms/Passes.h`, `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/Dialect/GPU/IR/GPUDialect.h`, `mlir/Dialect/GPU/Transforms/ParallelLoopMapper.h`, `mlir/Dialect/SCF/IR/SCF.h`, `mlir/IR/AffineMap.h`, `mlir/Dialect/GPU/Transforms/Passes.h.inc`
- **Subsystem categories / 子系统类别**: other MLIR dialect declarations / 其他 MLIR 方言声明 (6), MLIR IR core abstractions / MLIR IR 核心抽象 (1)
