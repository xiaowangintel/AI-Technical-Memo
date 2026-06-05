# SplitFunctions.h — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `bolt/include/bolt/Passes/SplitFunctions.h`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: Split function code. It also sits in code that declares BOLT optimization and transformation pass interfaces. / 该文件声明 BOLT 优化与变换 Pass 接口。 源码头部说明其职责是：Split function code。

## Line-by-Line Analysis / 逐行分析

### Lines 1-8

```cpp
//===- bolt/Passes/SplitFunctions.h - Split function code -------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 9-17

```cpp
#ifndef BOLT_PASSES_SPLIT_FUNCTIONS_H
#define BOLT_PASSES_SPLIT_FUNCTIONS_H

#include "bolt/Core/FunctionLayout.h"
#include "bolt/Passes/BinaryPasses.h"
#include "llvm/ADT/Hashing.h"
#include "llvm/Support/CommandLine.h"
#include <atomic>
```

- EN: Pulls in 5 header(s) from local project, LLVM, system dependencies needed by this range. Establishes header-guard or prologue macros so the file can be compiled safely. Defines macros such as `BOLT_PASSES_SPLIT_FUNCTIONS_H` for constants or compile-time switches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里引入 5 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里建立头文件保护或前导宏，保证源码可被安全编译与重复包含控制。这里定义宏 `BOLT_PASSES_SPLIT_FUNCTIONS_H`，用于常量或编译期开关。这里的注释记录了设计假设、不变量或使用说明。

### Lines 18-30

```cpp
namespace llvm {
namespace bolt {

class SplitStrategy {
public:
  using BlockIt = BinaryFunction::BasicBlockOrderType::iterator;

  virtual ~SplitStrategy() = default;
  virtual bool canSplit(const BinaryFunction &BF) = 0;
  virtual bool compactFragments() = 0;
  virtual void fragment(const BlockIt Start, const BlockIt End) = 0;
};
```

- EN: Works inside namespace scope `llvm`, `bolt` to organize symbols. Introduces type definitions such as `SplitStrategy`. Declares or implements routines including `SplitStrategy`, `canSplit`, `compactFragments`, `fragment`. Notable symbols here include `SplitStrategy`, `canSplit`, `compactFragments`, `fragment`, `llvm`, `bolt`.
- CN: 这里位于命名空间 `llvm`, `bolt` 中，用于组织符号作用域。这里引入类型定义，例如 `SplitStrategy`。这里声明或实现函数，例如 `SplitStrategy`, `canSplit`, `compactFragments`, `fragment`。这里较值得关注的符号包括 `SplitStrategy`, `canSplit`, `compactFragments`, `fragment`, `llvm`, `bolt`。

### Lines 31-40

```cpp
/// Split function code in multiple parts.
class SplitFunctions : public BinaryFunctionPass {
private:
  /// Split function body into fragments.
  void splitFunction(BinaryFunction &Function, SplitStrategy &Strategy);

  struct TrampolineKey {
    FragmentNum SourceFN = FragmentNum::main();
    const MCSymbol *Target = nullptr;
```

- EN: Introduces type definitions such as `SplitFunctions`, `TrampolineKey`. Declares or implements routines including `splitFunction`, `main`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `SplitFunctions`, `TrampolineKey`, `splitFunction`, `main`.
- CN: 这里引入类型定义，例如 `SplitFunctions`, `TrampolineKey`。这里声明或实现函数，例如 `splitFunction`, `main`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `SplitFunctions`, `TrampolineKey`, `splitFunction`, `main`。

### Lines 41-56

```cpp
    TrampolineKey() = default;
    TrampolineKey(const FragmentNum SourceFN, const MCSymbol *const Target)
        : SourceFN(SourceFN), Target(Target) {}

    static inline TrampolineKey getEmptyKey() { return TrampolineKey(); };
    static inline TrampolineKey getTombstoneKey() {
      return TrampolineKey(FragmentNum(UINT_MAX), nullptr);
    };
    static unsigned getHashValue(const TrampolineKey &Val) {
      return llvm::hash_combine(Val.SourceFN.get(), Val.Target);
    }
    static bool isEqual(const TrampolineKey &LHS, const TrampolineKey &RHS) {
      return LHS.SourceFN == RHS.SourceFN && LHS.Target == RHS.Target;
    }
  };
```

- EN: Declares or implements routines including `TrampolineKey`, `SourceFN`, `getEmptyKey`, `getTombstoneKey`, `getHashValue`, and 1 more. Notable symbols here include `TrampolineKey`, `SourceFN`, `getEmptyKey`, `getTombstoneKey`, `getHashValue`, `isEqual`.
- CN: 这里声明或实现函数，例如 `TrampolineKey`, `SourceFN`, `getEmptyKey`, `getTombstoneKey`, `getHashValue`, and 1 more。这里较值得关注的符号包括 `TrampolineKey`, `SourceFN`, `getEmptyKey`, `getTombstoneKey`, `getHashValue`, `isEqual`。

### Lines 57-69

```cpp
  /// Map basic block labels to their trampoline block labels.
  using TrampolineSetType =
      DenseMap<TrampolineKey, const MCSymbol *, TrampolineKey>;

  using BasicBlockOrderType = BinaryFunction::BasicBlockOrderType;

  /// Create trampoline landing pads for exception handling code to guarantee
  /// that every landing pad is placed in the same function fragment as the
  /// corresponding thrower block. The trampoline landing pad, when created,
  /// will redirect the execution to the real landing pad in a different
  /// fragment.
  TrampolineSetType createEHTrampolines(BinaryFunction &Function) const;
```

- EN: Declares or implements routines including `createEHTrampolines`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `createEHTrampolines`.
- CN: 这里声明或实现函数，例如 `createEHTrampolines`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `createEHTrampolines`。

### Lines 70-79

```cpp
  /// Merge trampolines into \p Layout without trampolines. The merge will place
  /// a trampoline immediately before its destination. Used to revert the effect
  /// of trampolines after createEHTrampolines().
  BasicBlockOrderType
  mergeEHTrampolines(BinaryFunction &BF, BasicBlockOrderType &Layout,
                     const TrampolineSetType &Trampolines) const;

  std::atomic<uint64_t> SplitBytesHot{0ull};
  std::atomic<uint64_t> SplitBytesCold{0ull};
```

- EN: Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里的注释记录了设计假设、不变量或使用说明。

### Lines 80-87

```cpp
public:
  explicit SplitFunctions(const cl::opt<bool> &PrintPass)
      : BinaryFunctionPass(PrintPass) {}

  bool shouldOptimize(const BinaryFunction &BF) const override;

  const char *getName() const override { return "split-functions"; }
```

- EN: Declares or implements routines including `SplitFunctions`, `BinaryFunctionPass`, `shouldOptimize`, `getName`. Notable symbols here include `SplitFunctions`, `BinaryFunctionPass`, `shouldOptimize`, `getName`.
- CN: 这里声明或实现函数，例如 `SplitFunctions`, `BinaryFunctionPass`, `shouldOptimize`, `getName`。这里较值得关注的符号包括 `SplitFunctions`, `BinaryFunctionPass`, `shouldOptimize`, `getName`。

### Lines 88-94

```cpp
  Error runOnFunctions(BinaryContext &BC) override;
};

} // namespace bolt
} // namespace llvm

#endif
```

- EN: Uses conditional compilation to choose platform- or feature-specific code paths. Works inside namespace scope `bolt`, `llvm` to organize symbols. Declares or implements routines including `runOnFunctions`. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里使用条件编译，根据平台或特性切换不同代码路径。这里位于命名空间 `bolt`, `llvm` 中，用于组织符号作用域。这里声明或实现函数，例如 `runOnFunctions`。这里的注释记录了设计假设、不变量或使用说明。

## Key Concepts / 关键概念

- `SplitStrategy`: class or struct interface / 类或结构体接口
- `SplitFunctions`: class or struct interface / 类或结构体接口
- `TrampolineKey`: class or struct interface / 类或结构体接口
- `SplitStrategy`: function or method entry point / 函数或方法入口
- `canSplit`: function or method entry point / 函数或方法入口
- `compactFragments`: function or method entry point / 函数或方法入口
- `fragment`: function or method entry point / 函数或方法入口
- `splitFunction`: function or method entry point / 函数或方法入口

## Dependencies / 依赖关系

- Local headers / 本地头文件: `bolt/Core/FunctionLayout.h`, `bolt/Passes/BinaryPasses.h`
- LLVM headers / LLVM 头文件: `llvm/ADT/Hashing.h`, `llvm/Support/CommandLine.h`
- System headers / 系统头文件: `atomic`
- Directory context / 目录上下文: `bolt/include/bolt/Passes` neighbors usually cooperate with this file to provide the surrounding subsystem / `bolt/include/bolt/Passes` 下的相邻文件通常与本文件协作组成对应子系统
