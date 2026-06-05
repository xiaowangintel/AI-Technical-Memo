# IRBuilder.h — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `polly/include/polly/CodeGen/IRBuilder.h` | `polly/include/polly/CodeGen/IRBuilder.h` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | Declares public Polly code-generation interfaces, builders, and helper types. The leading comment describes it as: The IR builder used by Polly. | 声明 Polly 代码生成相关的公共接口、构建器与辅助类型。 文件开头注释将其概括为：The IR builder used by Polly。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-13

````cpp
//===- Codegen/IRBuilder.h - The IR builder used by Polly -*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// The Polly IRBuilder file contains Polly specific extensions for the IRBuilder
// that are used e.g. to emit the llvm.loop.parallel metadata.
//
//===----------------------------------------------------------------------===//

````
- **EN**: This block records the standard LLVM file banner and license metadata; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 给出 LLVM 标准文件头与许可证元数据; 保留用于解释意图、用法或算法背景的注释.

### Lines 14-25

````cpp
#ifndef POLLY_CODEGEN_IRBUILDER_H
#define POLLY_CODEGEN_IRBUILDER_H

#include "llvm/ADT/MapVector.h"
#include "llvm/IR/IRBuilder.h"

namespace llvm {
class Loop;
class SCEV;
class ScalarEvolution;
} // namespace llvm

````
- **EN**: This block imports LLVM-family headers needed by the surrounding code; uses preprocessor directives to control compilation, macros, or include guards; opens or organizes declarations inside a C++ namespace; declares or references types such as `Loop`, `SCEV`, `ScalarEvolution`; and continues the surrounding implementation details.
- **CN**: 该代码块 引入周边逻辑所需的 LLVM-family 头文件; 使用预处理指令控制编译、宏或头文件保护; 在 C++ 命名空间中组织声明或实现; 声明或引用类型，例如 `Loop`, `SCEV`, `ScalarEvolution`；并延续周边实现细节。

### Lines 26-41

````cpp
namespace polly {
class Scop;
struct BandAttr;

/// Helper class to annotate newly generated SCoPs with metadata.
///
/// The annotations are twofold:
///   1) Loops are stored in a stack-like structure in the order they are
///      constructed and the LoopID metadata node is added to the backedge.
///      Contained memory instructions and loop headers are annotated according
///      to all parallel surrounding loops.
///   2) The new SCoP is assumed alias free (either due to the result of
///      AliasAnalysis queries or runtime alias checks). We annotate therefore
///      all memory instruction with alias scopes to indicate that fact to
///      later optimizations.
///      These alias scopes live in a new alias domain only used in this SCoP.
````
- **EN**: This block opens or organizes declarations inside a C++ namespace; declares or references types such as `Scop`, `BandAttr`; touches Polly SCoP abstractions that model analyzable regions; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 在 C++ 命名空间中组织声明或实现; 声明或引用类型，例如 `Scop`, `BandAttr`; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域; 保留用于解释意图、用法或算法背景的注释.

### Lines 42-54

````cpp
///      Each base pointer has its own alias scope and is annotated to not
///      alias with any access to different base pointers.
class ScopAnnotator {
public:
  ScopAnnotator();
  ~ScopAnnotator();

  /// Build all alias scopes for the given SCoP.
  void buildAliasScopes(Scop &S);

  /// Add a new loop @p L which is parallel if @p IsParallel is true.
  void pushLoop(llvm::Loop *L, bool IsParallel);

````
- **EN**: This block declares or references types such as `ScopAnnotator`; declares or defines routines around `ScopAnnotator`, `~ScopAnnotator`, `buildAliasScopes`, `pushLoop`; touches Polly SCoP abstractions that model analyzable regions; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或引用类型，例如 `ScopAnnotator`; 声明或定义与 `ScopAnnotator`, `~ScopAnnotator`, `buildAliasScopes`, `pushLoop` 相关的例程; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域; 保留用于解释意图、用法或算法背景的注释.

### Lines 55-67

````cpp
  /// Remove the last added loop.
  void popLoop(bool isParallel);

  /// Annotate the new instruction @p I for all parallel loops.
  void annotate(llvm::Instruction *I);

  /// Annotate the loop latch @p B.
  /// Last argument is optional, if no value is passed, we don't annotate
  /// any vectorize metadata.
  void annotateLoopLatch(
      llvm::CondBrInst *B, bool IsParallel,
      std::optional<bool> EnableVectorizeMetadata = std::nullopt) const;

````
- **EN**: This block declares or defines routines around `popLoop`, `annotate`, `annotateLoopLatch`; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `popLoop`, `annotate`, `annotateLoopLatch` 相关的例程; 保留用于解释意图、用法或算法背景的注释.

### Lines 68-84

````cpp
  /// Add alternative alias based pointers
  ///
  /// When annotating instructions with alias scope metadata, the right metadata
  /// is identified through the base pointer of the memory access. In some cases
  /// (e.g. OpenMP code generation), the base pointer of the memory accesses is
  /// not the original base pointer, but was changed when passing the original
  /// base pointer over a function boundary. This function allows to provide a
  /// map that maps from these new base pointers to the original base pointers
  /// to allow the ScopAnnotator to still find the right alias scop annotations.
  ///
  /// @param NewMap A map from new base pointers to original base pointers.
  void addAlternativeAliasBases(
      llvm::DenseMap<llvm::AssertingVH<llvm::Value>,
                     llvm::AssertingVH<llvm::Value>> &NewMap) {
    AlternativeAliasBases.insert_range(NewMap);
  }

````
- **EN**: This block declares or defines routines around `addAlternativeAliasBases`, `insert_range`; touches Polly SCoP abstractions that model analyzable regions; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `addAlternativeAliasBases`, `insert_range` 相关的例程; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域; 保留用于解释意图、用法或算法背景的注释.

### Lines 85-98

````cpp
  /// Delete the set of alternative alias bases
  void resetAlternativeAliasBases() { AlternativeAliasBases.clear(); }

  /// Stack for surrounding BandAttr annotations.
  llvm::SmallVector<BandAttr *, 8> LoopAttrEnv;
  BandAttr *&getStagingAttrEnv() { return LoopAttrEnv.back(); }
  BandAttr *getActiveAttrEnv() const {
    return LoopAttrEnv[LoopAttrEnv.size() - 2];
  }

private:
  /// The ScalarEvolution analysis we use to find base pointers.
  llvm::ScalarEvolution *SE;

````
- **EN**: This block declares or defines routines around `resetAlternativeAliasBases`, `getStagingAttrEnv`, `getActiveAttrEnv`; emits return paths that hand results or status codes back to callers; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `resetAlternativeAliasBases`, `getStagingAttrEnv`, `getActiveAttrEnv` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 保留用于解释意图、用法或算法背景的注释.

### Lines 99-110

````cpp
  /// All loops currently under construction.
  llvm::SmallVector<llvm::Loop *, 8> ActiveLoops;

  /// Access groups for the parallel loops currently under construction.
  llvm::SmallVector<llvm::MDNode *, 8> ParallelLoops;

  /// The alias scope domain for the current SCoP.
  llvm::MDNode *AliasScopeDomain;

  /// A map from base pointers to its alias scope.
  llvm::MapVector<llvm::AssertingVH<llvm::Value>, llvm::MDNode *> AliasScopeMap;

````
- **EN**: This block touches Polly SCoP abstractions that model analyzable regions; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 涉及 Polly 的 SCoP 抽象，用于表示可分析区域; 保留用于解释意图、用法或算法背景的注释.

### Lines 111-127

````cpp
  /// A map from base pointers to an alias scope list of other pointers.
  llvm::DenseMap<llvm::AssertingVH<llvm::Value>, llvm::MDNode *>
      OtherAliasScopeListMap;

  llvm::DenseMap<llvm::AssertingVH<llvm::Value>, llvm::AssertingVH<llvm::Value>>
      AlternativeAliasBases;
};

/// Add Polly specifics when running IRBuilder.
///
/// This is used to add additional items such as e.g. the llvm.loop.parallel
/// metadata.
class IRInserter final : public llvm::IRBuilderDefaultInserter {
public:
  IRInserter() = default;
  IRInserter(ScopAnnotator &A) : Annotator(&A) {}

````
- **EN**: This block declares or references types such as `IRInserter`; declares or defines routines around `IRInserter`; touches Polly SCoP abstractions that model analyzable regions; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或引用类型，例如 `IRInserter`; 声明或定义与 `IRInserter` 相关的例程; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域; 保留用于解释意图、用法或算法背景的注释.

### Lines 128-144

````cpp
  void InsertHelper(llvm::Instruction *I, const llvm::Twine &Name,
                    llvm::BasicBlock::iterator InsertPt) const override {
    llvm::IRBuilderDefaultInserter::InsertHelper(I, Name, InsertPt);
    if (Annotator)
      Annotator->annotate(I);
  }

private:
  ScopAnnotator *Annotator = nullptr;
};

// TODO: We should not name instructions in NDEBUG builds.
//
// We currently always name instructions, as the polly test suite currently
// matches for certain names.
typedef llvm::IRBuilder<llvm::ConstantFolder, IRInserter> PollyIRBuilder;

````
- **EN**: This block declares or defines routines around `InsertHelper`, `annotate`; contains control flow with 1 conditional check(s); touches Polly SCoP abstractions that model analyzable regions; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `InsertHelper`, `annotate` 相关的例程; 包含控制流结构：1 处条件判断; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域; 保留用于解释意图、用法或算法背景的注释.

### Lines 145-146

````cpp
} // namespace polly
#endif
````
- **EN**: This block uses preprocessor directives to control compilation, macros, or include guards.
- **CN**: 该代码块 使用预处理指令控制编译、宏或头文件保护.

## Key Concepts / 关键概念

- **SCoP modeling**
  - **CN**: SCoP 建模
- **Loop transformation**
  - **CN**: 循环变换
- **Code generation**
  - **CN**: 代码生成
- **Template/type wrappers**
  - **CN**: 模板与类型包装
- **OpenMP/runtime support**
  - **CN**: OpenMP/运行时支持
- **Public interface declarations**
  - **CN**: 公共接口声明

## Dependencies / 依赖关系

- **LLVM-family headers**: `llvm/ADT/MapVector.h`, `llvm/IR/IRBuilder.h` — LLVM/Clang/MLIR infrastructure for IR, passes, utilities, or diagnostics.
  **LLVM-family headers（CN）**：`llvm/ADT/MapVector.h`, `llvm/IR/IRBuilder.h` —— 用于 IR、Pass、工具或诊断的 LLVM/Clang/MLIR 基础设施。
