# Rewrite.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/include/mlir-c/Rewrite.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This header declares the registration and creation method for rewrite patterns.
  - **CN**: 该文件位于 `mlir/include/mlir-c`，主要提供 `Rewrite` 相关的 MLIR 稳定 C API 声明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-13
```cpp
//===-- mlir-c/Rewrite.h - Helpers for C API to Rewrites ----------*- C -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM
// Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This header declares the registration and creation method for
// rewrite patterns.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 14-25
```cpp

#ifndef MLIR_C_REWRITE_H
#define MLIR_C_REWRITE_H

#include "mlir-c/IR.h"
#include "mlir-c/Support.h"
#include "mlir/Config/mlir-config.h"

#ifdef __cplusplus
extern "C" {
#endif

```
- **EN**: Pulls in the headers needed by this declaration unit, including `mlir-c/IR.h`, `mlir-c/Support.h`, `mlir/Config/mlir-config.h`.
- **CN**: 引入该声明单元所需的头文件，其中包括 `mlir-c/IR.h`, `mlir-c/Support.h`, `mlir/Config/mlir-config.h`。

### Lines 26-35
```cpp
//===----------------------------------------------------------------------===//
/// Opaque type declarations (see mlir-c/IR.h for more details).
//===----------------------------------------------------------------------===//

#define DEFINE_C_API_STRUCT(name, storage)                                     \
  struct name {                                                                \
    storage *ptr;                                                              \
  };                                                                           \
  typedef struct name name

```
- **EN**: Defines preprocessor-controlled structure, include guards, generated hook points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、头文件保护、生成式钩子或编译期常量。

### Lines 36-46
```cpp
DEFINE_C_API_STRUCT(MlirRewriterBase, void);
DEFINE_C_API_STRUCT(MlirFrozenRewritePatternSet, void);
DEFINE_C_API_STRUCT(MlirGreedyRewriteDriverConfig, void);

/// Greedy rewrite strictness levels.
typedef enum {
  /// No restrictions wrt. which ops are processed.
  MLIR_GREEDY_REWRITE_STRICTNESS_ANY_OP,
  /// Only pre-existing and newly created ops are processed.
  MLIR_GREEDY_REWRITE_STRICTNESS_EXISTING_AND_NEW_OPS,
  /// Only pre-existing ops are processed.
```
- **EN**: Implements logic around `DEFINE_C_API_STRUCT`.
- **CN**: 围绕 `DEFINE_C_API_STRUCT` 实现具体逻辑。

### Lines 47-56
```cpp
  MLIR_GREEDY_REWRITE_STRICTNESS_EXISTING_OPS
} MlirGreedyRewriteStrictness;

/// Greedy simplify region levels.
typedef enum {
  /// Disable region control-flow simplification.
  MLIR_GREEDY_SIMPLIFY_REGION_LEVEL_DISABLED,
  /// Run the normal simplification (e.g. dead args elimination).
  MLIR_GREEDY_SIMPLIFY_REGION_LEVEL_NORMAL,
  /// Run extra simplifications (e.g. block merging).
```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 57-67
```cpp
  MLIR_GREEDY_SIMPLIFY_REGION_LEVEL_AGGRESSIVE
} MlirGreedySimplifyRegionLevel;
DEFINE_C_API_STRUCT(MlirRewritePatternSet, void);
DEFINE_C_API_STRUCT(MlirPatternRewriter, void);
DEFINE_C_API_STRUCT(MlirRewritePattern, const void);
DEFINE_C_API_STRUCT(MlirConversionTarget, void);
DEFINE_C_API_STRUCT(MlirConversionPattern, const void);
DEFINE_C_API_STRUCT(MlirTypeConverter, void);
DEFINE_C_API_STRUCT(MlirConversionPatternRewriter, void);
DEFINE_C_API_STRUCT(MlirConversionConfig, void);

```
- **EN**: Declares APIs or declarative rules around `DEFINE_C_API_STRUCT`.
- **CN**: 声明与 `DEFINE_C_API_STRUCT` 相关的 API 或声明式规则。

### Lines 68-77
```cpp
//===----------------------------------------------------------------------===//
/// RewriterBase API inherited from OpBuilder
//===----------------------------------------------------------------------===//

/// Get the MLIR context referenced by the rewriter.
MLIR_CAPI_EXPORTED MlirContext
mlirRewriterBaseGetContext(MlirRewriterBase rewriter);

//===----------------------------------------------------------------------===//
/// Insertion points methods
```
- **EN**: Declares APIs or declarative rules around `mlirRewriterBaseGetContext`; this block uses rewrite-pattern infrastructure to transform operations; defines ABI-stable C bindings.
- **CN**: 声明与 `mlirRewriterBaseGetContext` 相关的 API 或声明式规则；该代码块使用重写模式基础设施变换操作，并定义 ABI 稳定的 C 绑定。

### Lines 78-89
```cpp
//===----------------------------------------------------------------------===//

// These do not include functions using Block::iterator or Region::iterator, as
// they are not exposed by the C API yet. Similarly for methods using
// `InsertPoint` directly.

/// Reset the insertion point to no location.  Creating an operation without a
/// set insertion point is an error, but this can still be useful when the
/// current insertion point a builder refers to is being removed.
MLIR_CAPI_EXPORTED void
mlirRewriterBaseClearInsertionPoint(MlirRewriterBase rewriter);

```
- **EN**: Declares APIs or declarative rules around `mlirRewriterBaseClearInsertionPoint`; this block uses rewrite-pattern infrastructure to transform operations; defines ABI-stable C bindings.
- **CN**: 声明与 `mlirRewriterBaseClearInsertionPoint` 相关的 API 或声明式规则；该代码块使用重写模式基础设施变换操作，并定义 ABI 稳定的 C 绑定。

### Lines 90-101
```cpp
/// Sets the insertion point to the specified operation, which will cause
/// subsequent insertions to go right before it.
MLIR_CAPI_EXPORTED void
mlirRewriterBaseSetInsertionPointBefore(MlirRewriterBase rewriter,
                                        MlirOperation op);

/// Sets the insertion point to the node after the specified operation, which
/// will cause subsequent insertions to go right after it.
MLIR_CAPI_EXPORTED void
mlirRewriterBaseSetInsertionPointAfter(MlirRewriterBase rewriter,
                                       MlirOperation op);

```
- **EN**: Declares APIs or declarative rules around `mlirRewriterBaseSetInsertionPointBefore`, `mlirRewriterBaseSetInsertionPointAfter`; this block uses rewrite-pattern infrastructure to transform operations; defines ABI-stable C bindings.
- **CN**: 声明与 `mlirRewriterBaseSetInsertionPointBefore`, `mlirRewriterBaseSetInsertionPointAfter` 相关的 API 或声明式规则；该代码块使用重写模式基础设施变换操作，并定义 ABI 稳定的 C 绑定。

### Lines 102-111
```cpp
/// Sets the insertion point to the node after the specified value. If value
/// has a defining operation, sets the insertion point to the node after such
/// defining operation. This will cause subsequent insertions to go right
/// after it. Otherwise, value is a BlockArgument. Sets the insertion point to
/// the start of its block.
MLIR_CAPI_EXPORTED void
mlirRewriterBaseSetInsertionPointAfterValue(MlirRewriterBase rewriter,
                                            MlirValue value);

/// Sets the insertion point to the start of the specified block.
```
- **EN**: Declares APIs or declarative rules around `mlirRewriterBaseSetInsertionPointAfterValue`; this block uses rewrite-pattern infrastructure to transform operations; defines ABI-stable C bindings.
- **CN**: 声明与 `mlirRewriterBaseSetInsertionPointAfterValue` 相关的 API 或声明式规则；该代码块使用重写模式基础设施变换操作，并定义 ABI 稳定的 C 绑定。

### Lines 112-121
```cpp
MLIR_CAPI_EXPORTED void
mlirRewriterBaseSetInsertionPointToStart(MlirRewriterBase rewriter,
                                         MlirBlock block);

/// Sets the insertion point to the end of the specified block.
MLIR_CAPI_EXPORTED void
mlirRewriterBaseSetInsertionPointToEnd(MlirRewriterBase rewriter,
                                       MlirBlock block);

/// Return the block the current insertion point belongs to.  Note that the
```
- **EN**: Declares APIs or declarative rules around `mlirRewriterBaseSetInsertionPointToStart`, `mlirRewriterBaseSetInsertionPointToEnd`; this block uses rewrite-pattern infrastructure to transform operations; defines ABI-stable C bindings.
- **CN**: 声明与 `mlirRewriterBaseSetInsertionPointToStart`, `mlirRewriterBaseSetInsertionPointToEnd` 相关的 API 或声明式规则；该代码块使用重写模式基础设施变换操作，并定义 ABI 稳定的 C 绑定。

### Lines 122-131
```cpp
/// insertion point is not necessarily the end of the block.
MLIR_CAPI_EXPORTED MlirBlock
mlirRewriterBaseGetInsertionBlock(MlirRewriterBase rewriter);

/// Returns the current block of the rewriter.
MLIR_CAPI_EXPORTED MlirBlock
mlirRewriterBaseGetBlock(MlirRewriterBase rewriter);

/// Returns the operation right after the current insertion point
/// of the rewriter. A null MlirOperation will be returned
```
- **EN**: Declares APIs or declarative rules around `mlirRewriterBaseGetInsertionBlock`, `mlirRewriterBaseGetBlock`; this block uses rewrite-pattern infrastructure to transform operations; defines ABI-stable C bindings.
- **CN**: 声明与 `mlirRewriterBaseGetInsertionBlock`, `mlirRewriterBaseGetBlock` 相关的 API 或声明式规则；该代码块使用重写模式基础设施变换操作，并定义 ABI 稳定的 C 绑定。

### Lines 132-142
```cpp
// if the current insertion point is at the end of the block.
MLIR_CAPI_EXPORTED MlirOperation
mlirRewriterBaseGetOperationAfterInsertion(MlirRewriterBase rewriter);

//===----------------------------------------------------------------------===//
/// Block and operation creation/insertion/cloning
//===----------------------------------------------------------------------===//

// These functions do not include the IRMapper, as it is not yet exposed by the
// C API.

```
- **EN**: Declares APIs or declarative rules around `mlirRewriterBaseGetOperationAfterInsertion`; this block uses rewrite-pattern infrastructure to transform operations; defines ABI-stable C bindings.
- **CN**: 声明与 `mlirRewriterBaseGetOperationAfterInsertion` 相关的 API 或声明式规则；该代码块使用重写模式基础设施变换操作，并定义 ABI 稳定的 C 绑定。

### Lines 143-154
```cpp
/// Add new block with 'argTypes' arguments and set the insertion point to the
/// end of it. The block is placed before 'insertBefore'. `locs` contains the
/// locations of the inserted arguments, and should match the size of
/// `argTypes`.
MLIR_CAPI_EXPORTED MlirBlock mlirRewriterBaseCreateBlockBefore(
    MlirRewriterBase rewriter, MlirBlock insertBefore, intptr_t nArgTypes,
    MlirType const *argTypes, MlirLocation const *locations);

/// Insert the given operation at the current insertion point and return it.
MLIR_CAPI_EXPORTED MlirOperation
mlirRewriterBaseInsert(MlirRewriterBase rewriter, MlirOperation op);

```
- **EN**: Declares APIs or declarative rules around `mlirRewriterBaseCreateBlockBefore`, `mlirRewriterBaseInsert`; this block uses rewrite-pattern infrastructure to transform operations; defines ABI-stable C bindings.
- **CN**: 声明与 `mlirRewriterBaseCreateBlockBefore`, `mlirRewriterBaseInsert` 相关的 API 或声明式规则；该代码块使用重写模式基础设施变换操作，并定义 ABI 稳定的 C 绑定。

### Lines 155-164
```cpp
/// Creates a deep copy of the specified operation.
MLIR_CAPI_EXPORTED MlirOperation
mlirRewriterBaseClone(MlirRewriterBase rewriter, MlirOperation op);

/// Creates a deep copy of this operation but keep the operation regions
/// empty.
MLIR_CAPI_EXPORTED MlirOperation mlirRewriterBaseCloneWithoutRegions(
    MlirRewriterBase rewriter, MlirOperation op);

/// Clone the blocks that belong to "region" before the given position in
```
- **EN**: Declares APIs or declarative rules around `mlirRewriterBaseClone`, `mlirRewriterBaseCloneWithoutRegions`; this block uses rewrite-pattern infrastructure to transform operations; defines ABI-stable C bindings.
- **CN**: 声明与 `mlirRewriterBaseClone`, `mlirRewriterBaseCloneWithoutRegions` 相关的 API 或声明式规则；该代码块使用重写模式基础设施变换操作，并定义 ABI 稳定的 C 绑定。

### Lines 165-174
```cpp
/// another region "parent".
MLIR_CAPI_EXPORTED void
mlirRewriterBaseCloneRegionBefore(MlirRewriterBase rewriter, MlirRegion region,
                                  MlirBlock before);

//===----------------------------------------------------------------------===//
/// RewriterBase API
//===----------------------------------------------------------------------===//

/// Move the blocks that belong to "region" before the given position in
```
- **EN**: Declares APIs or declarative rules around `mlirRewriterBaseCloneRegionBefore`; this block uses rewrite-pattern infrastructure to transform operations; defines ABI-stable C bindings.
- **CN**: 声明与 `mlirRewriterBaseCloneRegionBefore` 相关的 API 或声明式规则；该代码块使用重写模式基础设施变换操作，并定义 ABI 稳定的 C 绑定。

### Lines 175-184
```cpp
/// another region "parent". The two regions must be different. The caller
/// is responsible for creating or updating the operation transferring flow
/// of control to the region and passing it the correct block arguments.
MLIR_CAPI_EXPORTED void
mlirRewriterBaseInlineRegionBefore(MlirRewriterBase rewriter, MlirRegion region,
                                   MlirBlock before);

/// Replace the results of the given (original) operation with the specified
/// list of values (replacements). The result types of the given op and the
/// replacements must match. The original op is erased.
```
- **EN**: Declares APIs or declarative rules around `mlirRewriterBaseInlineRegionBefore`; this block uses rewrite-pattern infrastructure to transform operations; defines ABI-stable C bindings.
- **CN**: 声明与 `mlirRewriterBaseInlineRegionBefore` 相关的 API 或声明式规则；该代码块使用重写模式基础设施变换操作，并定义 ABI 稳定的 C 绑定。

### Lines 185-195
```cpp
MLIR_CAPI_EXPORTED void
mlirRewriterBaseReplaceOpWithValues(MlirRewriterBase rewriter, MlirOperation op,
                                    intptr_t nValues, MlirValue const *values);

/// Replace the results of the given (original) operation with the specified
/// new op (replacement). The result types of the two ops must match. The
/// original op is erased.
MLIR_CAPI_EXPORTED void
mlirRewriterBaseReplaceOpWithOperation(MlirRewriterBase rewriter,
                                       MlirOperation op, MlirOperation newOp);

```
- **EN**: Declares APIs or declarative rules around `mlirRewriterBaseReplaceOpWithValues`, `mlirRewriterBaseReplaceOpWithOperation`; this block uses rewrite-pattern infrastructure to transform operations; defines ABI-stable C bindings.
- **CN**: 声明与 `mlirRewriterBaseReplaceOpWithValues`, `mlirRewriterBaseReplaceOpWithOperation` 相关的 API 或声明式规则；该代码块使用重写模式基础设施变换操作，并定义 ABI 稳定的 C 绑定。

### Lines 196-205
```cpp
/// Erases an operation that is known to have no uses.
MLIR_CAPI_EXPORTED void mlirRewriterBaseEraseOp(MlirRewriterBase rewriter,
                                                MlirOperation op);

/// Erases a block along with all operations inside it.
MLIR_CAPI_EXPORTED void mlirRewriterBaseEraseBlock(MlirRewriterBase rewriter,
                                                   MlirBlock block);

/// Inline the operations of block 'source' before the operation 'op'. The
/// source block will be deleted and must have no uses. 'argValues' is used to
```
- **EN**: Declares APIs or declarative rules around `mlirRewriterBaseEraseOp`, `mlirRewriterBaseEraseBlock`; this block uses rewrite-pattern infrastructure to transform operations; defines ABI-stable C bindings.
- **CN**: 声明与 `mlirRewriterBaseEraseOp`, `mlirRewriterBaseEraseBlock` 相关的 API 或声明式规则；该代码块使用重写模式基础设施变换操作，并定义 ABI 稳定的 C 绑定。

### Lines 206-215
```cpp
/// replace the block arguments of 'source'
///
/// The source block must have no successors. Otherwise, the resulting IR
/// would have unreachable operations.
MLIR_CAPI_EXPORTED void
mlirRewriterBaseInlineBlockBefore(MlirRewriterBase rewriter, MlirBlock source,
                                  MlirOperation op, intptr_t nArgValues,
                                  MlirValue const *argValues);

/// Inline the operations of block 'source' into the end of block 'dest'. The
```
- **EN**: Declares APIs or declarative rules around `mlirRewriterBaseInlineBlockBefore`; this block uses rewrite-pattern infrastructure to transform operations; defines ABI-stable C bindings.
- **CN**: 声明与 `mlirRewriterBaseInlineBlockBefore` 相关的 API 或声明式规则；该代码块使用重写模式基础设施变换操作，并定义 ABI 稳定的 C 绑定。

### Lines 216-226
```cpp
/// source block will be deleted and must have no uses. 'argValues' is used to
/// replace the block arguments of 'source'
///
/// The dest block must have no successors. Otherwise, the resulting IR would
/// have unreachable operation.
MLIR_CAPI_EXPORTED void mlirRewriterBaseMergeBlocks(MlirRewriterBase rewriter,
                                                    MlirBlock source,
                                                    MlirBlock dest,
                                                    intptr_t nArgValues,
                                                    MlirValue const *argValues);

```
- **EN**: Declares APIs or declarative rules around `mlirRewriterBaseMergeBlocks`; this block uses rewrite-pattern infrastructure to transform operations; defines ABI-stable C bindings.
- **CN**: 声明与 `mlirRewriterBaseMergeBlocks` 相关的 API 或声明式规则；该代码块使用重写模式基础设施变换操作，并定义 ABI 稳定的 C 绑定。

### Lines 227-236
```cpp
/// Unlink this operation from its current block and insert it right before
/// `existingOp` which may be in the same or another block in the same
/// function.
MLIR_CAPI_EXPORTED void mlirRewriterBaseMoveOpBefore(MlirRewriterBase rewriter,
                                                     MlirOperation op,
                                                     MlirOperation existingOp);

/// Unlink this operation from its current block and insert it right after
/// `existingOp` which may be in the same or another block in the same
/// function.
```
- **EN**: Declares APIs or declarative rules around `mlirRewriterBaseMoveOpBefore`; this block uses rewrite-pattern infrastructure to transform operations; defines ABI-stable C bindings.
- **CN**: 声明与 `mlirRewriterBaseMoveOpBefore` 相关的 API 或声明式规则；该代码块使用重写模式基础设施变换操作，并定义 ABI 稳定的 C 绑定。

### Lines 237-246
```cpp
MLIR_CAPI_EXPORTED void mlirRewriterBaseMoveOpAfter(MlirRewriterBase rewriter,
                                                    MlirOperation op,
                                                    MlirOperation existingOp);

/// Unlink this block and insert it right before `existingBlock`.
MLIR_CAPI_EXPORTED void
mlirRewriterBaseMoveBlockBefore(MlirRewriterBase rewriter, MlirBlock block,
                                MlirBlock existingBlock);

/// This method is used to notify the rewriter that an in-place operation
```
- **EN**: Declares APIs or declarative rules around `mlirRewriterBaseMoveOpAfter`, `mlirRewriterBaseMoveBlockBefore`; this block uses rewrite-pattern infrastructure to transform operations; defines ABI-stable C bindings.
- **CN**: 声明与 `mlirRewriterBaseMoveOpAfter`, `mlirRewriterBaseMoveBlockBefore` 相关的 API 或声明式规则；该代码块使用重写模式基础设施变换操作，并定义 ABI 稳定的 C 绑定。

### Lines 247-256
```cpp
/// modification is about to happen. A call to this function *must* be
/// followed by a call to either `finalizeOpModification` or
/// `cancelOpModification`. This is a minor efficiency win (it avoids creating
/// a new operation and removing the old one) but also often allows simpler
/// code in the client.
MLIR_CAPI_EXPORTED void
mlirRewriterBaseStartOpModification(MlirRewriterBase rewriter,
                                    MlirOperation op);

/// This method is used to signal the end of an in-place modification of the
```
- **EN**: Declares APIs or declarative rules around `mlirRewriterBaseStartOpModification`; this block uses rewrite-pattern infrastructure to transform operations; defines ABI-stable C bindings.
- **CN**: 声明与 `mlirRewriterBaseStartOpModification` 相关的 API 或声明式规则；该代码块使用重写模式基础设施变换操作，并定义 ABI 稳定的 C 绑定。

### Lines 257-269
```cpp
/// given operation. This can only be called on operations that were provided
/// to a call to `startOpModification`.
MLIR_CAPI_EXPORTED void
mlirRewriterBaseFinalizeOpModification(MlirRewriterBase rewriter,
                                       MlirOperation op);

/// This method cancels a pending in-place modification. This can only be
/// called on operations that were provided to a call to
/// `startOpModification`.
MLIR_CAPI_EXPORTED void
mlirRewriterBaseCancelOpModification(MlirRewriterBase rewriter,
                                     MlirOperation op);

```
- **EN**: Declares APIs or declarative rules around `mlirRewriterBaseFinalizeOpModification`, `mlirRewriterBaseCancelOpModification`; this block uses rewrite-pattern infrastructure to transform operations; defines ABI-stable C bindings.
- **CN**: 声明与 `mlirRewriterBaseFinalizeOpModification`, `mlirRewriterBaseCancelOpModification` 相关的 API 或声明式规则；该代码块使用重写模式基础设施变换操作，并定义 ABI 稳定的 C 绑定。

### Lines 270-281
```cpp
/// Find uses of `from` and replace them with `to`. Also notify the listener
/// about every in-place op modification (for every use that was replaced).
MLIR_CAPI_EXPORTED void
mlirRewriterBaseReplaceAllUsesWith(MlirRewriterBase rewriter, MlirValue from,
                                   MlirValue to);

/// Find uses of `from` and replace them with `to`. Also notify the listener
/// about every in-place op modification (for every use that was replaced).
MLIR_CAPI_EXPORTED void mlirRewriterBaseReplaceAllValueRangeUsesWith(
    MlirRewriterBase rewriter, intptr_t nValues, MlirValue const *from,
    MlirValue const *to);

```
- **EN**: Declares APIs or declarative rules around `mlirRewriterBaseReplaceAllUsesWith`, `mlirRewriterBaseReplaceAllValueRangeUsesWith`; this block uses rewrite-pattern infrastructure to transform operations; defines ABI-stable C bindings.
- **CN**: 声明与 `mlirRewriterBaseReplaceAllUsesWith`, `mlirRewriterBaseReplaceAllValueRangeUsesWith` 相关的 API 或声明式规则；该代码块使用重写模式基础设施变换操作，并定义 ABI 稳定的 C 绑定。

### Lines 282-291
```cpp
/// Find uses of `from` and replace them with `to`. Also notify the listener
/// about every in-place op modification (for every use that was replaced)
/// and that the `from` operation is about to be replaced.
MLIR_CAPI_EXPORTED void
mlirRewriterBaseReplaceAllOpUsesWithValueRange(MlirRewriterBase rewriter,
                                               MlirOperation from, intptr_t nTo,
                                               MlirValue const *to);

/// Find uses of `from` and replace them with `to`. Also notify the listener
/// about every in-place op modification (for every use that was replaced)
```
- **EN**: Declares APIs or declarative rules around `mlirRewriterBaseReplaceAllOpUsesWithValueRange`; this block uses rewrite-pattern infrastructure to transform operations; defines ABI-stable C bindings.
- **CN**: 声明与 `mlirRewriterBaseReplaceAllOpUsesWithValueRange` 相关的 API 或声明式规则；该代码块使用重写模式基础设施变换操作，并定义 ABI 稳定的 C 绑定。

### Lines 292-303
```cpp
/// and that the `from` operation is about to be replaced.
MLIR_CAPI_EXPORTED void mlirRewriterBaseReplaceAllOpUsesWithOperation(
    MlirRewriterBase rewriter, MlirOperation from, MlirOperation to);

/// Find uses of `from` within `block` and replace them with `to`. Also notify
/// the listener about every in-place op modification (for every use that was
/// replaced). The optional `allUsesReplaced` flag is set to "true" if all
/// uses were replaced.
MLIR_CAPI_EXPORTED void mlirRewriterBaseReplaceOpUsesWithinBlock(
    MlirRewriterBase rewriter, MlirOperation op, intptr_t nNewValues,
    MlirValue const *newValues, MlirBlock block);

```
- **EN**: Declares APIs or declarative rules around `mlirRewriterBaseReplaceAllOpUsesWithOperation`, `mlirRewriterBaseReplaceOpUsesWithinBlock`; this block uses rewrite-pattern infrastructure to transform operations; defines ABI-stable C bindings.
- **CN**: 声明与 `mlirRewriterBaseReplaceAllOpUsesWithOperation`, `mlirRewriterBaseReplaceOpUsesWithinBlock` 相关的 API 或声明式规则；该代码块使用重写模式基础设施变换操作，并定义 ABI 稳定的 C 绑定。

### Lines 304-313
```cpp
/// Find uses of `from` and replace them with `to` except if the user is
/// `exceptedUser`. Also notify the listener about every in-place op
/// modification (for every use that was replaced).
MLIR_CAPI_EXPORTED void
mlirRewriterBaseReplaceAllUsesExcept(MlirRewriterBase rewriter, MlirValue from,
                                     MlirValue to, MlirOperation exceptedUser);

//===----------------------------------------------------------------------===//
/// IRRewriter API
//===----------------------------------------------------------------------===//
```
- **EN**: Declares APIs or declarative rules around `mlirRewriterBaseReplaceAllUsesExcept`; this block uses rewrite-pattern infrastructure to transform operations; defines ABI-stable C bindings.
- **CN**: 声明与 `mlirRewriterBaseReplaceAllUsesExcept` 相关的 API 或声明式规则；该代码块使用重写模式基础设施变换操作，并定义 ABI 稳定的 C 绑定。

### Lines 314-323
```cpp

/// Create an IRRewriter and transfer ownership to the caller.
MLIR_CAPI_EXPORTED MlirRewriterBase mlirIRRewriterCreate(MlirContext context);

/// Create an IRRewriter and transfer ownership to the caller. Additionally
/// set the insertion point before the operation.
MLIR_CAPI_EXPORTED MlirRewriterBase
mlirIRRewriterCreateFromOp(MlirOperation op);

/// Takes an IRRewriter owned by the caller and destroys it. It is the
```
- **EN**: Declares APIs or declarative rules around `mlirIRRewriterCreate`, `mlirIRRewriterCreateFromOp`; this block defines ABI-stable C bindings.
- **CN**: 声明与 `mlirIRRewriterCreate`, `mlirIRRewriterCreateFromOp` 相关的 API 或声明式规则；该代码块定义 ABI 稳定的 C 绑定。

### Lines 324-333
```cpp
/// responsibility of the user to only pass an IRRewriter class.
MLIR_CAPI_EXPORTED void mlirIRRewriterDestroy(MlirRewriterBase rewriter);

//===----------------------------------------------------------------------===//
/// FrozenRewritePatternSet API
//===----------------------------------------------------------------------===//

/// Freeze the given MlirRewritePatternSet to a MlirFrozenRewritePatternSet.
/// Note that the ownership of the input set is transferred into the frozen set
/// after this call.
```
- **EN**: Declares APIs or declarative rules around `mlirIRRewriterDestroy`; this block packages logic as an MLIR pass or pass-related API; uses rewrite-pattern infrastructure to transform operations; defines ABI-stable C bindings.
- **CN**: 声明与 `mlirIRRewriterDestroy` 相关的 API 或声明式规则；该代码块将逻辑组织为 MLIR Pass 或相关 API，并使用重写模式基础设施变换操作，并定义 ABI 稳定的 C 绑定。

### Lines 334-344
```cpp
MLIR_CAPI_EXPORTED MlirFrozenRewritePatternSet
mlirFreezeRewritePattern(MlirRewritePatternSet set);

/// Destroy the given MlirFrozenRewritePatternSet.
MLIR_CAPI_EXPORTED void
mlirFrozenRewritePatternSetDestroy(MlirFrozenRewritePatternSet set);

MLIR_CAPI_EXPORTED MlirLogicalResult mlirApplyPatternsAndFoldGreedilyWithOp(
    MlirOperation op, MlirFrozenRewritePatternSet patterns,
    MlirGreedyRewriteDriverConfig);

```
- **EN**: Declares APIs or declarative rules around `mlirFreezeRewritePattern`, `mlirFrozenRewritePatternSetDestroy`, `mlirApplyPatternsAndFoldGreedilyWithOp`; this block defines ABI-stable C bindings.
- **CN**: 声明与 `mlirFreezeRewritePattern`, `mlirFrozenRewritePatternSetDestroy`, `mlirApplyPatternsAndFoldGreedilyWithOp` 相关的 API 或声明式规则；该代码块定义 ABI 稳定的 C 绑定。

### Lines 345-356
```cpp
MLIR_CAPI_EXPORTED MlirLogicalResult mlirApplyPatternsAndFoldGreedily(
    MlirModule op, MlirFrozenRewritePatternSet patterns,
    MlirGreedyRewriteDriverConfig config);

//===----------------------------------------------------------------------===//
/// GreedyRewriteDriverConfig API
//===----------------------------------------------------------------------===//

/// Creates a greedy rewrite driver configuration with default settings.
MLIR_CAPI_EXPORTED MlirGreedyRewriteDriverConfig
mlirGreedyRewriteDriverConfigCreate(void);

```
- **EN**: Declares APIs or declarative rules around `mlirApplyPatternsAndFoldGreedily`, `mlirGreedyRewriteDriverConfigCreate`; this block defines ABI-stable C bindings.
- **CN**: 声明与 `mlirApplyPatternsAndFoldGreedily`, `mlirGreedyRewriteDriverConfigCreate` 相关的 API 或声明式规则；该代码块定义 ABI 稳定的 C 绑定。

### Lines 357-366
```cpp
/// Destroys a greedy rewrite driver configuration.
MLIR_CAPI_EXPORTED void
mlirGreedyRewriteDriverConfigDestroy(MlirGreedyRewriteDriverConfig config);

/// Sets the maximum number of iterations for the greedy rewrite driver.
/// Use -1 for no limit.
MLIR_CAPI_EXPORTED void mlirGreedyRewriteDriverConfigSetMaxIterations(
    MlirGreedyRewriteDriverConfig config, int64_t maxIterations);

/// Sets the maximum number of rewrites within an iteration.
```
- **EN**: Declares APIs or declarative rules around `mlirGreedyRewriteDriverConfigDestroy`, `mlirGreedyRewriteDriverConfigSetMaxIterations`; this block defines ABI-stable C bindings.
- **CN**: 声明与 `mlirGreedyRewriteDriverConfigDestroy`, `mlirGreedyRewriteDriverConfigSetMaxIterations` 相关的 API 或声明式规则；该代码块定义 ABI 稳定的 C 绑定。

### Lines 367-376
```cpp
/// Use -1 for no limit.
MLIR_CAPI_EXPORTED void mlirGreedyRewriteDriverConfigSetMaxNumRewrites(
    MlirGreedyRewriteDriverConfig config, int64_t maxNumRewrites);

/// Sets whether to use top-down traversal for the initial population of the
/// worklist.
MLIR_CAPI_EXPORTED void mlirGreedyRewriteDriverConfigSetUseTopDownTraversal(
    MlirGreedyRewriteDriverConfig config, bool useTopDownTraversal);

/// Enables or disables folding during greedy rewriting.
```
- **EN**: Declares APIs or declarative rules around `mlirGreedyRewriteDriverConfigSetMaxNumRewrites`, `mlirGreedyRewriteDriverConfigSetUseTopDownTraversal`; this block defines ABI-stable C bindings.
- **CN**: 声明与 `mlirGreedyRewriteDriverConfigSetMaxNumRewrites`, `mlirGreedyRewriteDriverConfigSetUseTopDownTraversal` 相关的 API 或声明式规则；该代码块定义 ABI 稳定的 C 绑定。

### Lines 377-386
```cpp
MLIR_CAPI_EXPORTED void
mlirGreedyRewriteDriverConfigEnableFolding(MlirGreedyRewriteDriverConfig config,
                                           bool enable);

/// Sets the strictness level for the greedy rewrite driver.
MLIR_CAPI_EXPORTED void mlirGreedyRewriteDriverConfigSetStrictness(
    MlirGreedyRewriteDriverConfig config,
    MlirGreedyRewriteStrictness strictness);

/// Sets the region simplification level.
```
- **EN**: Declares APIs or declarative rules around `mlirGreedyRewriteDriverConfigEnableFolding`, `mlirGreedyRewriteDriverConfigSetStrictness`; this block defines ABI-stable C bindings.
- **CN**: 声明与 `mlirGreedyRewriteDriverConfigEnableFolding`, `mlirGreedyRewriteDriverConfigSetStrictness` 相关的 API 或声明式规则；该代码块定义 ABI 稳定的 C 绑定。

### Lines 387-398
```cpp
MLIR_CAPI_EXPORTED void
mlirGreedyRewriteDriverConfigSetRegionSimplificationLevel(
    MlirGreedyRewriteDriverConfig config, MlirGreedySimplifyRegionLevel level);

/// Enables or disables constant CSE.
MLIR_CAPI_EXPORTED void mlirGreedyRewriteDriverConfigEnableConstantCSE(
    MlirGreedyRewriteDriverConfig config, bool enable);

/// Gets the maximum number of iterations for the greedy rewrite driver.
MLIR_CAPI_EXPORTED int64_t mlirGreedyRewriteDriverConfigGetMaxIterations(
    MlirGreedyRewriteDriverConfig config);

```
- **EN**: Declares APIs or declarative rules around `mlirGreedyRewriteDriverConfigSetRegionSimplificationLevel`, `mlirGreedyRewriteDriverConfigEnableConstantCSE`, `mlirGreedyRewriteDriverConfigGetMaxIterations`; this block defines ABI-stable C bindings.
- **CN**: 声明与 `mlirGreedyRewriteDriverConfigSetRegionSimplificationLevel`, `mlirGreedyRewriteDriverConfigEnableConstantCSE`, `mlirGreedyRewriteDriverConfigGetMaxIterations` 相关的 API 或声明式规则；该代码块定义 ABI 稳定的 C 绑定。

### Lines 399-410
```cpp
/// Gets the maximum number of rewrites within an iteration.
MLIR_CAPI_EXPORTED int64_t mlirGreedyRewriteDriverConfigGetMaxNumRewrites(
    MlirGreedyRewriteDriverConfig config);

/// Gets whether top-down traversal is used for initial worklist population.
MLIR_CAPI_EXPORTED bool mlirGreedyRewriteDriverConfigGetUseTopDownTraversal(
    MlirGreedyRewriteDriverConfig config);

/// Gets whether folding is enabled during greedy rewriting.
MLIR_CAPI_EXPORTED bool mlirGreedyRewriteDriverConfigIsFoldingEnabled(
    MlirGreedyRewriteDriverConfig config);

```
- **EN**: Declares APIs or declarative rules around `mlirGreedyRewriteDriverConfigGetMaxNumRewrites`, `mlirGreedyRewriteDriverConfigGetUseTopDownTraversal`, `mlirGreedyRewriteDriverConfigIsFoldingEnabled`; this block defines ABI-stable C bindings.
- **CN**: 声明与 `mlirGreedyRewriteDriverConfigGetMaxNumRewrites`, `mlirGreedyRewriteDriverConfigGetUseTopDownTraversal`, `mlirGreedyRewriteDriverConfigIsFoldingEnabled` 相关的 API 或声明式规则；该代码块定义 ABI 稳定的 C 绑定。

### Lines 411-420
```cpp
/// Gets the strictness level for the greedy rewrite driver.
MLIR_CAPI_EXPORTED MlirGreedyRewriteStrictness
mlirGreedyRewriteDriverConfigGetStrictness(
    MlirGreedyRewriteDriverConfig config);

/// Gets the region simplification level.
MLIR_CAPI_EXPORTED MlirGreedySimplifyRegionLevel
mlirGreedyRewriteDriverConfigGetRegionSimplificationLevel(
    MlirGreedyRewriteDriverConfig config);

```
- **EN**: Declares APIs or declarative rules around `mlirGreedyRewriteDriverConfigGetStrictness`, `mlirGreedyRewriteDriverConfigGetRegionSimplificationLevel`; this block defines ABI-stable C bindings.
- **CN**: 声明与 `mlirGreedyRewriteDriverConfigGetStrictness`, `mlirGreedyRewriteDriverConfigGetRegionSimplificationLevel` 相关的 API 或声明式规则；该代码块定义 ABI 稳定的 C 绑定。

### Lines 421-430
```cpp
/// Gets whether constant CSE is enabled.
MLIR_CAPI_EXPORTED bool mlirGreedyRewriteDriverConfigIsConstantCSEEnabled(
    MlirGreedyRewriteDriverConfig config);

/// Applies the given patterns to the given op by a fast walk-based pattern
/// rewrite driver.
MLIR_CAPI_EXPORTED void
mlirWalkAndApplyPatterns(MlirOperation op,
                         MlirFrozenRewritePatternSet patterns);

```
- **EN**: Declares APIs or declarative rules around `mlirGreedyRewriteDriverConfigIsConstantCSEEnabled`, `mlirWalkAndApplyPatterns`; this block defines ABI-stable C bindings.
- **CN**: 声明与 `mlirGreedyRewriteDriverConfigIsConstantCSEEnabled`, `mlirWalkAndApplyPatterns` 相关的 API 或声明式规则；该代码块定义 ABI 稳定的 C 绑定。

### Lines 431-440
```cpp
/// Apply a partial conversion on the given operation.
MLIR_CAPI_EXPORTED MlirLogicalResult mlirApplyPartialConversion(
    MlirOperation op, MlirConversionTarget target,
    MlirFrozenRewritePatternSet patterns, MlirConversionConfig config);

/// Apply a full conversion on the given operation.
MLIR_CAPI_EXPORTED MlirLogicalResult mlirApplyFullConversion(
    MlirOperation op, MlirConversionTarget target,
    MlirFrozenRewritePatternSet patterns, MlirConversionConfig config);

```
- **EN**: Declares APIs or declarative rules around `mlirApplyPartialConversion`, `mlirApplyFullConversion`; this block defines ABI-stable C bindings.
- **CN**: 声明与 `mlirApplyPartialConversion`, `mlirApplyFullConversion` 相关的 API 或声明式规则；该代码块定义 ABI 稳定的 C 绑定。

### Lines 441-451
```cpp
//===----------------------------------------------------------------------===//
/// ConversionConfig API
//===----------------------------------------------------------------------===//

/// Create a default ConversionConfig.
MLIR_CAPI_EXPORTED MlirConversionConfig mlirConversionConfigCreate(void);

/// Destroy the given ConversionConfig.
MLIR_CAPI_EXPORTED void
mlirConversionConfigDestroy(MlirConversionConfig config);

```
- **EN**: Declares APIs or declarative rules around `mlirConversionConfigCreate`, `mlirConversionConfigDestroy`; this block defines ABI-stable C bindings.
- **CN**: 声明与 `mlirConversionConfigCreate`, `mlirConversionConfigDestroy` 相关的 API 或声明式规则；该代码块定义 ABI 稳定的 C 绑定。

### Lines 452-462
```cpp
typedef enum {
  MLIR_DIALECT_CONVERSION_FOLDING_MODE_NEVER,
  MLIR_DIALECT_CONVERSION_FOLDING_MODE_BEFORE_PATTERNS,
  MLIR_DIALECT_CONVERSION_FOLDING_MODE_AFTER_PATTERNS,
} MlirDialectConversionFoldingMode;

/// Set the folding mode for the given ConversionConfig.
MLIR_CAPI_EXPORTED void
mlirConversionConfigSetFoldingMode(MlirConversionConfig config,
                                   MlirDialectConversionFoldingMode mode);

```
- **EN**: Implements logic around `mlirConversionConfigSetFoldingMode`; this block defines ABI-stable C bindings.
- **CN**: 围绕 `mlirConversionConfigSetFoldingMode` 实现具体逻辑；该代码块定义 ABI 稳定的 C 绑定。

### Lines 463-472
```cpp
/// Get the folding mode for the given ConversionConfig.
MLIR_CAPI_EXPORTED MlirDialectConversionFoldingMode
mlirConversionConfigGetFoldingMode(MlirConversionConfig config);

/// Enable or disable building materializations during conversion.
MLIR_CAPI_EXPORTED void
mlirConversionConfigEnableBuildMaterializations(MlirConversionConfig config,
                                                bool enable);

/// Check if building materializations during conversion is enabled.
```
- **EN**: Declares APIs or declarative rules around `mlirConversionConfigGetFoldingMode`, `mlirConversionConfigEnableBuildMaterializations`; this block defines ABI-stable C bindings.
- **CN**: 声明与 `mlirConversionConfigGetFoldingMode`, `mlirConversionConfigEnableBuildMaterializations` 相关的 API 或声明式规则；该代码块定义 ABI 稳定的 C 绑定。

### Lines 473-483
```cpp
MLIR_CAPI_EXPORTED bool
mlirConversionConfigIsBuildMaterializationsEnabled(MlirConversionConfig config);

//===----------------------------------------------------------------------===//
/// PatternRewriter API
//===----------------------------------------------------------------------===//

/// Cast the PatternRewriter to a RewriterBase
MLIR_CAPI_EXPORTED MlirRewriterBase
mlirPatternRewriterAsBase(MlirPatternRewriter rewriter);

```
- **EN**: Declares APIs or declarative rules around `mlirConversionConfigIsBuildMaterializationsEnabled`, `mlirPatternRewriterAsBase`; this block uses rewrite-pattern infrastructure to transform operations; defines ABI-stable C bindings.
- **CN**: 声明与 `mlirConversionConfigIsBuildMaterializationsEnabled`, `mlirPatternRewriterAsBase` 相关的 API 或声明式规则；该代码块使用重写模式基础设施变换操作，并定义 ABI 稳定的 C 绑定。

### Lines 484-493
```cpp
//===----------------------------------------------------------------------===//
/// ConversionPatternRewriter API
//===----------------------------------------------------------------------===//

/// Cast the ConversionPatternRewriter to a PatternRewriter
MLIR_CAPI_EXPORTED MlirPatternRewriter
mlirConversionPatternRewriterAsPatternRewriter(
    MlirConversionPatternRewriter rewriter);

/// Apply a signature conversion to each block in the given region.
```
- **EN**: Declares APIs or declarative rules around `mlirConversionPatternRewriterAsPatternRewriter`; this block uses rewrite-pattern infrastructure to transform operations; defines ABI-stable C bindings.
- **CN**: 声明与 `mlirConversionPatternRewriterAsPatternRewriter` 相关的 API 或声明式规则；该代码块使用重写模式基础设施变换操作，并定义 ABI 稳定的 C 绑定。

### Lines 494-503
```cpp
MLIR_CAPI_EXPORTED MlirLogicalResult
mlirConversionPatternRewriterConvertRegionTypes(
    MlirConversionPatternRewriter rewriter, MlirRegion region,
    MlirTypeConverter typeConverter);

//===----------------------------------------------------------------------===//
/// ConversionTarget API
//===----------------------------------------------------------------------===//

/// Create an empty ConversionTarget.
```
- **EN**: Declares APIs or declarative rules around `mlirConversionPatternRewriterConvertRegionTypes`; this block uses rewrite-pattern infrastructure to transform operations; coordinates dialect conversion or lowering decisions; defines ABI-stable C bindings.
- **CN**: 声明与 `mlirConversionPatternRewriterConvertRegionTypes` 相关的 API 或声明式规则；该代码块使用重写模式基础设施变换操作，并协调方言转换或 lowering 决策，并定义 ABI 稳定的 C 绑定。

### Lines 504-515
```cpp
MLIR_CAPI_EXPORTED MlirConversionTarget
mlirConversionTargetCreate(MlirContext context);

/// Destroy the given ConversionTarget.
MLIR_CAPI_EXPORTED void
mlirConversionTargetDestroy(MlirConversionTarget target);

/// Register the given operations as legal.
MLIR_CAPI_EXPORTED void
mlirConversionTargetAddLegalOp(MlirConversionTarget target,
                               MlirStringRef opName);

```
- **EN**: Declares APIs or declarative rules around `mlirConversionTargetCreate`, `mlirConversionTargetDestroy`, `mlirConversionTargetAddLegalOp`; this block coordinates dialect conversion or lowering decisions; defines ABI-stable C bindings.
- **CN**: 声明与 `mlirConversionTargetCreate`, `mlirConversionTargetDestroy`, `mlirConversionTargetAddLegalOp` 相关的 API 或声明式规则；该代码块协调方言转换或 lowering 决策，并定义 ABI 稳定的 C 绑定。

### Lines 516-525
```cpp
/// Register the given operations as illegal.
MLIR_CAPI_EXPORTED void
mlirConversionTargetAddIllegalOp(MlirConversionTarget target,
                                 MlirStringRef opName);

/// Register the operations of the given dialect as legal.
MLIR_CAPI_EXPORTED void
mlirConversionTargetAddLegalDialect(MlirConversionTarget target,
                                    MlirStringRef dialectName);

```
- **EN**: Declares APIs or declarative rules around `mlirConversionTargetAddIllegalOp`, `mlirConversionTargetAddLegalDialect`; this block defines ABI-stable C bindings.
- **CN**: 声明与 `mlirConversionTargetAddIllegalOp`, `mlirConversionTargetAddLegalDialect` 相关的 API 或声明式规则；该代码块定义 ABI 稳定的 C 绑定。

### Lines 526-535
```cpp
/// Register the operations of the given dialect as illegal.
MLIR_CAPI_EXPORTED void
mlirConversionTargetAddIllegalDialect(MlirConversionTarget target,
                                      MlirStringRef dialectName);

//===----------------------------------------------------------------------===//
/// TypeConverter API
//===----------------------------------------------------------------------===//

/// Create a TypeConverter.
```
- **EN**: Declares APIs or declarative rules around `mlirConversionTargetAddIllegalDialect`; this block coordinates dialect conversion or lowering decisions; defines ABI-stable C bindings.
- **CN**: 声明与 `mlirConversionTargetAddIllegalDialect` 相关的 API 或声明式规则；该代码块协调方言转换或 lowering 决策，并定义 ABI 稳定的 C 绑定。

### Lines 536-545
```cpp
MLIR_CAPI_EXPORTED MlirTypeConverter mlirTypeConverterCreate(void);

/// Destroy the given TypeConverter.
MLIR_CAPI_EXPORTED void
mlirTypeConverterDestroy(MlirTypeConverter typeConverter);

/// Callback type for type conversion functions.
/// Returns failure or sets convertedType to MlirType{NULL} to indicate failure.
/// If failure is returned, the converter is allowed to try another
/// conversion function to perform the conversion.
```
- **EN**: Implements logic around `mlirTypeConverterCreate`, `mlirTypeConverterDestroy`; this block coordinates dialect conversion or lowering decisions; defines ABI-stable C bindings.
- **CN**: 围绕 `mlirTypeConverterCreate`, `mlirTypeConverterDestroy` 实现具体逻辑；该代码块协调方言转换或 lowering 决策，并定义 ABI 稳定的 C 绑定。

### Lines 546-555
```cpp
typedef MlirLogicalResult (*MlirTypeConverterConversionCallback)(
    MlirType type, MlirType *convertedType, void *userData);

/// Add a type conversion function to the given TypeConverter.
MLIR_CAPI_EXPORTED void
mlirTypeConverterAddConversion(MlirTypeConverter typeConverter,
                               MlirTypeConverterConversionCallback convertType,
                               void *userData);

/// Convert the given type using the given TypeConverter.
```
- **EN**: Declares APIs or declarative rules around `MlirLogicalResult`, `mlirTypeConverterAddConversion`; this block coordinates dialect conversion or lowering decisions; defines ABI-stable C bindings.
- **CN**: 声明与 `MlirLogicalResult`, `mlirTypeConverterAddConversion` 相关的 API 或声明式规则；该代码块协调方言转换或 lowering 决策，并定义 ABI 稳定的 C 绑定。

### Lines 556-565
```cpp
MLIR_CAPI_EXPORTED MlirType
mlirTypeConverterConvertType(MlirTypeConverter typeConverter, MlirType type);

//===----------------------------------------------------------------------===//
/// ConversionPattern API
//===----------------------------------------------------------------------===//

typedef struct {
  /// Optional constructor for the user data.
  /// Set to nullptr to disable it.
```
- **EN**: Implements logic around `mlirTypeConverterConvertType`; this block uses rewrite-pattern infrastructure to transform operations; coordinates dialect conversion or lowering decisions; defines ABI-stable C bindings.
- **CN**: 围绕 `mlirTypeConverterConvertType` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调方言转换或 lowering 决策，并定义 ABI 稳定的 C 绑定。

### Lines 566-579
```cpp
  void (*construct)(void *userData);
  /// Optional destructor for the user data.
  /// Set to nullptr to disable it.
  void (*destruct)(void *userData);
  /// The callback function to match against code rooted at the specified
  /// operation, and perform the conversion rewrite if the match is successful,
  /// corresponding to ConversionPattern::matchAndRewrite.
  MlirLogicalResult (*matchAndRewrite)(MlirConversionPattern pattern,
                                       MlirOperation op, intptr_t nOperands,
                                       MlirValue *operands,
                                       MlirConversionPatternRewriter rewriter,
                                       void *userData);
} MlirConversionPatternCallbacks;

```
- **EN**: Declares APIs or declarative rules around `void`, `MlirLogicalResult`; this block uses rewrite-pattern infrastructure to transform operations; defines ABI-stable C bindings.
- **CN**: 声明与 `void`, `MlirLogicalResult` 相关的 API 或声明式规则；该代码块使用重写模式基础设施变换操作，并定义 ABI 稳定的 C 绑定。

### Lines 580-590
```cpp
/// Create a conversion pattern that matches the operation with the given
/// rootName, corresponding to mlir::OpConversionPattern.
MLIR_CAPI_EXPORTED MlirConversionPattern mlirOpConversionPatternCreate(
    MlirStringRef rootName, unsigned benefit, MlirContext context,
    MlirTypeConverter typeConverter, MlirConversionPatternCallbacks callbacks,
    void *userData, size_t nGeneratedNames, MlirStringRef *generatedNames);

/// Get the type converter used by this conversion pattern.
MLIR_CAPI_EXPORTED MlirTypeConverter
mlirConversionPatternGetTypeConverter(MlirConversionPattern pattern);

```
- **EN**: Declares APIs or declarative rules around `mlirOpConversionPatternCreate`, `mlirConversionPatternGetTypeConverter`; this block coordinates dialect conversion or lowering decisions; defines ABI-stable C bindings.
- **CN**: 声明与 `mlirOpConversionPatternCreate`, `mlirConversionPatternGetTypeConverter` 相关的 API 或声明式规则；该代码块协调方言转换或 lowering 决策，并定义 ABI 稳定的 C 绑定。

### Lines 591-601
```cpp
/// Cast the ConversionPattern to a RewritePattern.
MLIR_CAPI_EXPORTED MlirRewritePattern
mlirConversionPatternAsRewritePattern(MlirConversionPattern pattern);

//===----------------------------------------------------------------------===//
/// RewritePattern API
//===----------------------------------------------------------------------===//

/// Callbacks to construct a rewrite pattern.
typedef struct {
  /// Optional constructor for the user data.
```
- **EN**: Implements logic around `mlirConversionPatternAsRewritePattern`; this block uses rewrite-pattern infrastructure to transform operations; defines ABI-stable C bindings.
- **CN**: 围绕 `mlirConversionPatternAsRewritePattern` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并定义 ABI 稳定的 C 绑定。

### Lines 602-615
```cpp
  /// Set to nullptr to disable it.
  void (*construct)(void *userData);
  /// Optional destructor for the user data.
  /// Set to nullptr to disable it.
  void (*destruct)(void *userData);
  /// The callback function to match against code rooted at the specified
  /// operation, and perform the rewrite if the match is successful,
  /// corresponding to RewritePattern::matchAndRewrite.
  MlirLogicalResult (*matchAndRewrite)(MlirRewritePattern pattern,
                                       MlirOperation op,
                                       MlirPatternRewriter rewriter,
                                       void *userData);
} MlirRewritePatternCallbacks;

```
- **EN**: Declares APIs or declarative rules around `void`, `MlirLogicalResult`; this block uses rewrite-pattern infrastructure to transform operations; defines ABI-stable C bindings.
- **CN**: 声明与 `void`, `MlirLogicalResult` 相关的 API 或声明式规则；该代码块使用重写模式基础设施变换操作，并定义 ABI 稳定的 C 绑定。

### Lines 616-625
```cpp
/// Create a rewrite pattern that matches the operation
/// with the given rootName, corresponding to mlir::OpRewritePattern.
MLIR_CAPI_EXPORTED MlirRewritePattern mlirOpRewritePatternCreate(
    MlirStringRef rootName, unsigned benefit, MlirContext context,
    MlirRewritePatternCallbacks callbacks, void *userData,
    size_t nGeneratedNames, MlirStringRef *generatedNames);

//===----------------------------------------------------------------------===//
/// RewritePatternSet API
//===----------------------------------------------------------------------===//
```
- **EN**: Declares APIs or declarative rules around `mlirOpRewritePatternCreate`; this block uses rewrite-pattern infrastructure to transform operations; defines ABI-stable C bindings.
- **CN**: 声明与 `mlirOpRewritePatternCreate` 相关的 API 或声明式规则；该代码块使用重写模式基础设施变换操作，并定义 ABI 稳定的 C 绑定。

### Lines 626-635
```cpp

/// Create an empty MlirRewritePatternSet.
MLIR_CAPI_EXPORTED MlirRewritePatternSet
mlirRewritePatternSetCreate(MlirContext context);

/// Get the context associated with a MlirRewritePatternSet.
MLIR_CAPI_EXPORTED MlirContext
mlirRewritePatternSetGetContext(MlirRewritePatternSet set);

/// Destruct the given MlirRewritePatternSet.
```
- **EN**: Declares APIs or declarative rules around `mlirRewritePatternSetCreate`, `mlirRewritePatternSetGetContext`; this block defines ABI-stable C bindings.
- **CN**: 声明与 `mlirRewritePatternSetCreate`, `mlirRewritePatternSetGetContext` 相关的 API 或声明式规则；该代码块定义 ABI 稳定的 C 绑定。

### Lines 636-645
```cpp
MLIR_CAPI_EXPORTED void mlirRewritePatternSetDestroy(MlirRewritePatternSet set);

/// Add the given MlirRewritePattern into a MlirRewritePatternSet.
/// Note that the ownership of the pattern is transferred to the set after this
/// call.
MLIR_CAPI_EXPORTED void mlirRewritePatternSetAdd(MlirRewritePatternSet set,
                                                 MlirRewritePattern pattern);

//===----------------------------------------------------------------------===//
/// PDLPatternModule API
```
- **EN**: Declares APIs or declarative rules around `mlirRewritePatternSetDestroy`, `mlirRewritePatternSetAdd`; this block defines ABI-stable C bindings.
- **CN**: 声明与 `mlirRewritePatternSetDestroy`, `mlirRewritePatternSetAdd` 相关的 API 或声明式规则；该代码块定义 ABI 稳定的 C 绑定。

### Lines 646-655
```cpp
//===----------------------------------------------------------------------===//

#if MLIR_ENABLE_PDL_IN_PATTERNMATCH
DEFINE_C_API_STRUCT(MlirPDLPatternModule, void);
DEFINE_C_API_STRUCT(MlirPDLValue, const void);
DEFINE_C_API_STRUCT(MlirPDLResultList, void);

MLIR_CAPI_EXPORTED MlirPDLPatternModule
mlirPDLPatternModuleFromModule(MlirModule op);

```
- **EN**: Defines preprocessor-controlled structure, include guards, generated hook points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、头文件保护、生成式钩子或编译期常量。

### Lines 656-665
```cpp
MLIR_CAPI_EXPORTED void mlirPDLPatternModuleDestroy(MlirPDLPatternModule op);

MLIR_CAPI_EXPORTED MlirRewritePatternSet
mlirRewritePatternSetFromPDLPatternModule(MlirPDLPatternModule op);

/// Cast the MlirPDLValue to an MlirValue.
/// Return a null value if the cast fails, just like llvm::dyn_cast.
MLIR_CAPI_EXPORTED MlirValue mlirPDLValueAsValue(MlirPDLValue value);

/// Cast the MlirPDLValue to an MlirType.
```
- **EN**: Declares APIs or declarative rules around `mlirPDLPatternModuleDestroy`, `mlirRewritePatternSetFromPDLPatternModule`, `mlirPDLValueAsValue`; this block defines ABI-stable C bindings.
- **CN**: 声明与 `mlirPDLPatternModuleDestroy`, `mlirRewritePatternSetFromPDLPatternModule`, `mlirPDLValueAsValue` 相关的 API 或声明式规则；该代码块定义 ABI 稳定的 C 绑定。

### Lines 666-676
```cpp
/// Return a null value if the cast fails, just like llvm::dyn_cast.
MLIR_CAPI_EXPORTED MlirType mlirPDLValueAsType(MlirPDLValue value);

/// Cast the MlirPDLValue to an MlirOperation.
/// Return a null value if the cast fails, just like llvm::dyn_cast.
MLIR_CAPI_EXPORTED MlirOperation mlirPDLValueAsOperation(MlirPDLValue value);

/// Cast the MlirPDLValue to an MlirAttribute.
/// Return a null value if the cast fails, just like llvm::dyn_cast.
MLIR_CAPI_EXPORTED MlirAttribute mlirPDLValueAsAttribute(MlirPDLValue value);

```
- **EN**: Declares APIs or declarative rules around `mlirPDLValueAsType`, `mlirPDLValueAsOperation`, `mlirPDLValueAsAttribute`; this block defines ABI-stable C bindings.
- **CN**: 声明与 `mlirPDLValueAsType`, `mlirPDLValueAsOperation`, `mlirPDLValueAsAttribute` 相关的 API 或声明式规则；该代码块定义 ABI 稳定的 C 绑定。

### Lines 677-689
```cpp
/// Push the MlirValue into the given MlirPDLResultList.
MLIR_CAPI_EXPORTED void
mlirPDLResultListPushBackValue(MlirPDLResultList results, MlirValue value);

/// Push the MlirType into the given MlirPDLResultList.
MLIR_CAPI_EXPORTED void mlirPDLResultListPushBackType(MlirPDLResultList results,
                                                      MlirType value);

/// Push the MlirOperation into the given MlirPDLResultList.
MLIR_CAPI_EXPORTED void
mlirPDLResultListPushBackOperation(MlirPDLResultList results,
                                   MlirOperation value);

```
- **EN**: Declares APIs or declarative rules around `mlirPDLResultListPushBackValue`, `mlirPDLResultListPushBackType`, `mlirPDLResultListPushBackOperation`; this block defines ABI-stable C bindings.
- **CN**: 声明与 `mlirPDLResultListPushBackValue`, `mlirPDLResultListPushBackType`, `mlirPDLResultListPushBackOperation` 相关的 API 或声明式规则；该代码块定义 ABI 稳定的 C 绑定。

### Lines 690-702
```cpp
/// Push the MlirAttribute into the given MlirPDLResultList.
MLIR_CAPI_EXPORTED void
mlirPDLResultListPushBackAttribute(MlirPDLResultList results,
                                   MlirAttribute value);

/// This function type is used as callbacks for PDL native rewrite functions.
/// Input values can be accessed by `values` with its size `nValues`;
/// output values can be added into `results` by `mlirPDLResultListPushBack*`
/// APIs. And the return value indicates whether the rewrite succeeds.
typedef MlirLogicalResult (*MlirPDLRewriteFunction)(
    MlirPatternRewriter rewriter, MlirPDLResultList results, size_t nValues,
    MlirPDLValue *values, void *userData);

```
- **EN**: Declares APIs or declarative rules around `mlirPDLResultListPushBackAttribute`, `MlirLogicalResult`; this block uses rewrite-pattern infrastructure to transform operations; defines ABI-stable C bindings.
- **CN**: 声明与 `mlirPDLResultListPushBackAttribute`, `MlirLogicalResult` 相关的 API 或声明式规则；该代码块使用重写模式基础设施变换操作，并定义 ABI 稳定的 C 绑定。

### Lines 703-712
```cpp
/// Register a rewrite function into the given PDL pattern module.
/// `userData` will be provided as an argument to the rewrite function.
MLIR_CAPI_EXPORTED void mlirPDLPatternModuleRegisterRewriteFunction(
    MlirPDLPatternModule pdlModule, MlirStringRef name,
    MlirPDLRewriteFunction rewriteFn, void *userData);

/// This function type is used as callbacks for PDL native constraint functions.
/// Input values can be accessed by `values` with its size `nValues`;
/// output values can be added into `results` by `mlirPDLResultListPushBack*`
/// APIs. And the return value indicates whether the constraint holds.
```
- **EN**: Declares APIs or declarative rules around `mlirPDLPatternModuleRegisterRewriteFunction`; this block defines ABI-stable C bindings.
- **CN**: 声明与 `mlirPDLPatternModuleRegisterRewriteFunction` 相关的 API 或声明式规则；该代码块定义 ABI 稳定的 C 绑定。

### Lines 713-722
```cpp
typedef MlirLogicalResult (*MlirPDLConstraintFunction)(
    MlirPatternRewriter rewriter, MlirPDLResultList results, size_t nValues,
    MlirPDLValue *values, void *userData);

/// Register a constraint function into the given PDL pattern module.
/// `userData` will be provided as an argument to the constraint function.
MLIR_CAPI_EXPORTED void mlirPDLPatternModuleRegisterConstraintFunction(
    MlirPDLPatternModule pdlModule, MlirStringRef name,
    MlirPDLConstraintFunction constraintFn, void *userData);

```
- **EN**: Declares APIs or declarative rules around `MlirLogicalResult`, `mlirPDLPatternModuleRegisterConstraintFunction`; this block uses rewrite-pattern infrastructure to transform operations; defines ABI-stable C bindings.
- **CN**: 声明与 `MlirLogicalResult`, `mlirPDLPatternModuleRegisterConstraintFunction` 相关的 API 或声明式规则；该代码块使用重写模式基础设施变换操作，并定义 ABI 稳定的 C 绑定。

### Lines 723-731
```cpp
#endif // MLIR_ENABLE_PDL_IN_PATTERNMATCH

#undef DEFINE_C_API_STRUCT

#ifdef __cplusplus
}
#endif

#endif // MLIR_C_REWRITE_H
```
- **EN**: Defines preprocessor-controlled structure, include guards, generated hook points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、头文件保护、生成式钩子或编译期常量。

## Key Concepts / 关键概念

- **Stable C API surface / 稳定 C API 接口**:
  - **EN**: Exposes MLIR functionality through ABI-stable C wrappers suitable for non-C++ clients.
  - **CN**: 通过 ABI 稳定的 C 包装层向非 C++ 客户端暴露 MLIR 功能。
- **Pattern rewriting / 模式重写**:
  - **EN**: Uses MLIR rewrite patterns to match operations and replace them with improved forms.
  - **CN**: 使用 MLIR 重写模式匹配操作，并将其替换为更合适的形式。
- **Dialect conversion pipeline / 方言转换流水线**:
  - **EN**: Coordinates legality checks, type adaptation, and target-specific lowering.
  - **CN**: 协调合法性检查、类型适配以及目标相关 lowering。
- **C API bridging / C API 桥接**:
  - **EN**: Wraps C++ MLIR facilities in opaque C handles and exported entry points.
  - **CN**: 通过不透明 C 句柄和导出入口包装 C++ MLIR 能力。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir-c/IR.h`, `mlir-c/Support.h`, `mlir/Config/mlir-config.h`
- **Subsystem categories / 子系统类别**: stable C API declarations / 稳定的 C API 声明 (2)
