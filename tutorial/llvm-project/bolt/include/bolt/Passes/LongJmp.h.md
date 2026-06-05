# LongJmp.h — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `bolt/include/bolt/Passes/LongJmp.h`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: *- C++ -*. It also sits in code that declares BOLT optimization and transformation pass interfaces. / 该文件声明 BOLT 优化与变换 Pass 接口。 源码头部说明其职责是：*- C++ -*。

## Line-by-Line Analysis / 逐行分析

### Lines 1-8

```cpp
//===- bolt/Passes/LongJmp.h ------------------------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 9-16

```cpp
#ifndef BOLT_PASSES_LONGJMP_H
#define BOLT_PASSES_LONGJMP_H

#include "bolt/Passes/BinaryPasses.h"

namespace llvm {
namespace bolt {
```

- EN: Pulls in 1 header(s) from local project dependencies needed by this range. Establishes header-guard or prologue macros so the file can be compiled safely. Works inside namespace scope `llvm`, `bolt` to organize symbols. Defines macros such as `BOLT_PASSES_LONGJMP_H` for constants or compile-time switches.
- CN: 这里引入 1 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里建立头文件保护或前导宏，保证源码可被安全编译与重复包含控制。这里位于命名空间 `llvm`, `bolt` 中，用于组织符号作用域。这里定义宏 `BOLT_PASSES_LONGJMP_H`，用于常量或编译期开关。

### Lines 17-32

```cpp
/// LongJmp is veneer-insertion pass originally written for AArch64 that
/// compensates for its short-range branches, typically done during linking. We
/// pull this pass inside BOLT because here we can do a better job at stub
/// inserting by manipulating the CFG, something linkers can't do.
///
/// We iteratively repeat the following until no modification is done: we do a
/// tentative layout with the current function sizes; then we add stubs for
/// branches that we know are out of range or we expand smaller stubs (28-bit)
/// to a large one if necessary (32 or 64).
///
/// This expansion inserts the equivalent of "linker stubs", small
/// blocks of code that load a 64-bit address into a pre-allocated register and
//  then executes an unconditional indirect branch on this register. By using a
/// 64-bit range, we guarantee it can reach any code location.
///
class LongJmpPass : public BinaryFunctionPass {
```

- EN: Introduces type definitions such as `LongJmpPass`. Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `LongJmpPass`.
- CN: 这里引入类型定义，例如 `LongJmpPass`。这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `LongJmpPass`。

### Lines 33-41

```cpp
  /// Used to implement stub grouping (reusing a stub from one function into
  /// another)
  using StubTy = std::pair<uint64_t, BinaryBasicBlock *>;
  using StubGroupTy = SmallVector<StubTy, 4>;
  using StubGroupsTy = DenseMap<const MCSymbol *, StubGroupTy>;
  StubGroupsTy HotStubGroups;
  StubGroupsTy ColdStubGroups;
  DenseMap<const MCSymbol *, BinaryBasicBlock *> SharedStubs;
```

- EN: Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里的注释记录了设计假设、不变量或使用说明。

### Lines 42-51

```cpp
  /// Stubs that are local to a function. This will be the primary lookup
  /// before resorting to stubs located in foreign functions.
  using StubMapTy = DenseMap<const BinaryFunction *, StubGroupsTy>;
  /// Used to quickly fetch stubs based on the target they jump to
  StubMapTy HotLocalStubs;
  StubMapTy ColdLocalStubs;

  /// Used to quickly identify whether a BB is a stub, sharded by function
  DenseMap<const BinaryFunction *, std::set<const BinaryBasicBlock *>> Stubs;
```

- EN: Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里的注释记录了设计假设、不变量或使用说明。

### Lines 52-60

```cpp
  using FuncAddressesMapTy = DenseMap<const BinaryFunction *, uint64_t>;
  /// Hold tentative addresses
  FuncAddressesMapTy HotAddresses;
  FuncAddressesMapTy ColdAddresses;
  DenseMap<const BinaryBasicBlock *, uint64_t> BBAddresses;

  /// Used to identify the stub size
  DenseMap<const BinaryBasicBlock *, int> StubBits;
```

- EN: Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里的注释记录了设计假设、不变量或使用说明。

### Lines 61-69

```cpp
  /// Stats about number of stubs inserted
  uint32_t NumHotStubs{0};
  uint32_t NumColdStubs{0};
  uint32_t NumSharedStubs{0};

  /// The shortest distance for any branch instruction on AArch64.
  static constexpr size_t ShortestJumpBits = 11;
  static constexpr size_t ShortestJumpSpan = 1ULL << (ShortestJumpBits - 1);
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 70-78

```cpp
  /// The longest single-instruction branch.
  static constexpr size_t LongestJumpBits = 28;
  static constexpr size_t LongestJumpSpan = 1ULL << (LongestJumpBits - 1);

  /// Relax all internal function branches including those between fragments.
  /// Assume that fragments are placed in different sections but are within
  /// 128MB of each other.
  void relaxLocalBranches(BinaryFunction &BF);
```

- EN: Declares or implements routines including `relaxLocalBranches`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `relaxLocalBranches`.
- CN: 这里声明或实现函数，例如 `relaxLocalBranches`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `relaxLocalBranches`。

### Lines 79-93

```cpp
  ///                 -- Layout estimation methods --
  /// Try to do layout before running the emitter, by looking at BinaryFunctions
  /// and MCInsts -- this is an estimation. To be correct for longjmp inserter
  /// purposes, we need to do a size worst-case estimation. Real layout is done
  /// by RewriteInstance::mapFileSections()
  void tentativeLayout(const BinaryContext &BC,
                       BinaryFunctionListType &SortedFunctions);
  uint64_t tentativeLayoutRelocMode(const BinaryContext &BC,
                                    BinaryFunctionListType &SortedFunctions,
                                    uint64_t DotAddress);
  uint64_t tentativeLayoutRelocColdPart(const BinaryContext &BC,
                                        BinaryFunctionListType &SortedFunctions,
                                        uint64_t DotAddress);
  void tentativeBBLayout(const BinaryFunction &Func);
```

- EN: Declares or implements routines including `tentativeBBLayout`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `tentativeBBLayout`.
- CN: 这里声明或实现函数，例如 `tentativeBBLayout`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `tentativeBBLayout`。

### Lines 94-106

```cpp
  /// Update stubs addresses with their exact address after a round of stub
  /// insertion and layout estimation is done.
  void updateStubGroups();

  ///              -- Relaxation/stub insertion methods --
  /// Creates a  new stub jumping to \p TgtSym and updates bookkeeping about
  /// this stub using \p AtAddress as its initial location. This location is
  /// an approximation and will be later resolved to the exact location in
  /// a next iteration, in updateStubGroups.
  std::pair<std::unique_ptr<BinaryBasicBlock>, MCSymbol *>
  createNewStub(BinaryBasicBlock &SourceBB, const MCSymbol *TgtSym,
                bool TgtIsFunc, uint64_t AtAddress);
```

- EN: Declares or implements routines including `updateStubGroups`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `updateStubGroups`.
- CN: 这里声明或实现函数，例如 `updateStubGroups`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `updateStubGroups`。

### Lines 107-124

```cpp
  /// Replace the target of call or conditional branch in \p Inst with a
  /// a stub that in turn will branch to the target (perform stub insertion).
  /// If a new stub was created, return it.
  std::unique_ptr<BinaryBasicBlock>
  replaceTargetWithStub(BinaryBasicBlock &BB, MCInst &Inst, uint64_t DotAddress,
                        uint64_t StubCreationAddress);

  /// Helper used to fetch the closest stub to \p Inst at \p DotAddress that
  /// is jumping to \p TgtSym. Returns nullptr if the closest stub is out of
  /// range or if it doesn't exist. The source of truth for stubs will be the
  /// map \p StubGroups, which can be either local stubs for a particular
  /// function that is very large and needs to group stubs, or can be global
  /// stubs if we are sharing stubs across functions.
  BinaryBasicBlock *lookupStubFromGroup(const StubGroupsTy &StubGroups,
                                        const BinaryFunction &Func,
                                        const MCInst &Inst,
                                        const MCSymbol *TgtSym,
                                        uint64_t DotAddress) const;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 125-136

```cpp

  /// Lookup closest stub from the global pool, meaning this can return a basic
  /// block from another function.
  BinaryBasicBlock *lookupGlobalStub(const BinaryBasicBlock &SourceBB,
                                     const MCInst &Inst, const MCSymbol *TgtSym,
                                     uint64_t DotAddress) const;

  /// Lookup closest stub local to \p Func.
  BinaryBasicBlock *lookupLocalStub(const BinaryBasicBlock &SourceBB,
                                    const MCInst &Inst, const MCSymbol *TgtSym,
                                    uint64_t DotAddress) const;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 137-146

```cpp
  /// Helper to identify whether \p Inst is branching to a stub
  bool usesStub(const BinaryFunction &Func, const MCInst &Inst) const;

  /// True if Inst is a branch that is out of range
  bool needsStub(const BinaryBasicBlock &BB, const MCInst &Inst,
                 uint64_t DotAddress) const;

  /// Expand the range of the stub in StubBB if necessary
  Error relaxStub(BinaryBasicBlock &StubBB, bool &Modified);
```

- EN: Declares or implements routines including `usesStub`, `relaxStub`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `usesStub`, `relaxStub`.
- CN: 这里声明或实现函数，例如 `usesStub`, `relaxStub`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `usesStub`, `relaxStub`。

### Lines 147-156

```cpp
  /// Helper to resolve a symbol address according to our tentative layout
  uint64_t getSymbolAddress(const BinaryContext &BC, const MCSymbol *Target,
                            const BinaryBasicBlock *TgtBB) const;

  /// Relax function by adding necessary stubs or relaxing existing stubs
  Error relax(BinaryFunction &BF, bool &Modified);

public:
  /// BinaryPass public interface
```

- EN: Declares or implements routines including `relax`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `relax`.
- CN: 这里声明或实现函数，例如 `relax`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `relax`。

### Lines 157-166

```cpp
  explicit LongJmpPass(const cl::opt<bool> &PrintPass)
      : BinaryFunctionPass(PrintPass) {}

  const char *getName() const override { return "long-jmp"; }

  Error runOnFunctions(BinaryContext &BC) override;
};
} // namespace bolt
} // namespace llvm
```

- EN: Works inside namespace scope `bolt`, `llvm` to organize symbols. Declares or implements routines including `LongJmpPass`, `BinaryFunctionPass`, `getName`, `runOnFunctions`. Notable symbols here include `LongJmpPass`, `BinaryFunctionPass`, `getName`, `runOnFunctions`, `bolt`, `llvm`.
- CN: 这里位于命名空间 `bolt`, `llvm` 中，用于组织符号作用域。这里声明或实现函数，例如 `LongJmpPass`, `BinaryFunctionPass`, `getName`, `runOnFunctions`。这里较值得关注的符号包括 `LongJmpPass`, `BinaryFunctionPass`, `getName`, `runOnFunctions`, `bolt`, `llvm`。

### Lines 167-167

```cpp
#endif
```

- EN: Uses conditional compilation to choose platform- or feature-specific code paths. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里使用条件编译，根据平台或特性切换不同代码路径。这里的注释记录了设计假设、不变量或使用说明。

## Key Concepts / 关键概念

- `LongJmpPass`: class or struct interface / 类或结构体接口
- `relaxLocalBranches`: function or method entry point / 函数或方法入口
- `tentativeBBLayout`: function or method entry point / 函数或方法入口
- `updateStubGroups`: function or method entry point / 函数或方法入口
- `usesStub`: function or method entry point / 函数或方法入口
- `relaxStub`: function or method entry point / 函数或方法入口
- `BOLT_PASSES_LONGJMP_H`: macro or compile-time switch / 宏或编译期开关
- `llvm`: namespace scope / 命名空间作用域

## Dependencies / 依赖关系

- Local headers / 本地头文件: `bolt/Passes/BinaryPasses.h`
- Directory context / 目录上下文: `bolt/include/bolt/Passes` neighbors usually cooperate with this file to provide the surrounding subsystem / `bolt/include/bolt/Passes` 下的相邻文件通常与本文件协作组成对应子系统
