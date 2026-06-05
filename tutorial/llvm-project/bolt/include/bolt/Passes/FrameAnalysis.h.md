# FrameAnalysis.h — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `bolt/include/bolt/Passes/FrameAnalysis.h`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: *- C++ -*. It also sits in code that declares BOLT optimization and transformation pass interfaces. / 该文件声明 BOLT 优化与变换 Pass 接口。 源码头部说明其职责是：*- C++ -*。

## Line-by-Line Analysis / 逐行分析

### Lines 1-8

```cpp
//===- bolt/Passes/FrameAnalysis.h ------------------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 9-18

```cpp
#ifndef BOLT_PASSES_FRAMEANALYSIS_H
#define BOLT_PASSES_FRAMEANALYSIS_H

#include "bolt/Passes/StackPointerTracking.h"
#include <tuple>

namespace llvm {
namespace bolt {
class BinaryFunctionCallGraph;
```

- EN: Pulls in 2 header(s) from local project, system dependencies needed by this range. Establishes header-guard or prologue macros so the file can be compiled safely. Works inside namespace scope `llvm`, `bolt` to organize symbols. Introduces type definitions such as `BinaryFunctionCallGraph`.
- CN: 这里引入 2 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里建立头文件保护或前导宏，保证源码可被安全编译与重复包含控制。这里位于命名空间 `llvm`, `bolt` 中，用于组织符号作用域。这里引入类型定义，例如 `BinaryFunctionCallGraph`。

### Lines 19-34

```cpp
/// Alias analysis information attached to each instruction that accesses a
/// frame position. This is called a "frame index" by LLVM Target libs when
/// it is building a MachineFunction frame, and we use the same name here
/// because we are essentially doing the job of frame reconstruction.
struct FrameIndexEntry {
  /// If both IsLoad and IsStore are set, it means this is an instruction that
  /// reads and updates this frame location.
  bool IsLoad;
  bool IsStore;
  /// If a store, this controls whether the store uses a register os an imm
  /// as the source value.
  bool IsStoreFromReg;
  /// If load, this holds the destination register. If store, this holds
  /// either the source register or source immediate.
  int32_t RegOrImm;
```

- EN: Introduces type definitions such as `FrameIndexEntry`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `FrameIndexEntry`.
- CN: 这里引入类型定义，例如 `FrameIndexEntry`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `FrameIndexEntry`。

### Lines 35-44

```cpp
  /// StackOffset and Size are the two aspects that identify this frame access
  /// for the purposes of alias analysis.
  int64_t StackOffset;
  uint8_t Size;

  /// If this is false, we will never attempt to remove or optimize this
  /// instruction. We just use it to keep track of stores we don't fully
  /// understand but we know it may write to a frame position.
  bool IsSimple;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 45-55

```cpp
  uint16_t StackPtrReg;
};

/// Record an access to an argument in stack. This should be attached to
/// call instructions, so StackOffset and Size are determined in the context
/// of the caller. This information helps the caller understand how the callee
/// may access its private stack.
struct ArgInStackAccess {
  int64_t StackOffset;
  uint8_t Size;
```

- EN: Introduces type definitions such as `ArgInStackAccess`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `ArgInStackAccess`.
- CN: 这里引入类型定义，例如 `ArgInStackAccess`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `ArgInStackAccess`。

### Lines 56-68

```cpp
  bool operator<(const ArgInStackAccess &RHS) const {
    return std::tie(StackOffset, Size) < std::tie(RHS.StackOffset, RHS.Size);
  }
};

/// The set of all args-in-stack accesses for a given instruction. If
/// AssumeEverything is true, then the set should be ignored and the
/// corresponding instruction should be treated as accessing the entire
/// stack for the purposes of analysis and optimization.
struct ArgAccesses {
  bool AssumeEverything;
  std::set<ArgInStackAccess> Set;
```

- EN: Introduces type definitions such as `ArgAccesses`. Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `ArgAccesses`.
- CN: 这里引入类型定义，例如 `ArgAccesses`。这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `ArgAccesses`。

### Lines 69-86

```cpp
  explicit ArgAccesses(bool AssumeEverything)
      : AssumeEverything(AssumeEverything) {}
};

raw_ostream &operator<<(raw_ostream &OS, const FrameIndexEntry &FIE);

/// This pass attaches stack access information to instructions. If a load/store
/// instruction accesses a stack position, it will identify the CFA offset and
/// size information of this access, where CFA is the Canonical Frame Address
/// (using DWARF terminology).
///
/// This pass also computes frame usage information obtained by a bottom-up call
/// graph traversal: which registers are clobbered by functions (including their
/// callees as determined by the call graph), whether a function accesses its
/// caller's stack frame and whether a function demands its stack to be aligned
/// due to the use of SSE aligned load/store operations present in itself or any
/// of its direct or indirect callees.
///
```

- EN: Declares or implements routines including `ArgAccesses`, `AssumeEverything`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `ArgAccesses`, `AssumeEverything`.
- CN: 这里声明或实现函数，例如 `ArgAccesses`, `AssumeEverything`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `ArgAccesses`, `AssumeEverything`。

### Lines 87-104

```cpp
/// Initialization:
///
///   FrameAnalysis FA(PrintPass);
///   FA.runOnFunctions(BC);
///
/// Usage (fetching frame access information about a given instruction):
///
///   auto FIE = FA.getFIEFor(BC, Instruction);
///   if (FIE && FIE->IsSimple) {
///     ... = FIE->StackOffset
///     ... = FIE->Size
///   }
///
/// Usage (determining the set of stack positions accessed by the target of a
/// call:
///
///    auto Args = FA.getArgAccessesFor(BC, CallInst);
///    if (Args && Args->AssumeEverything) {
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 105-115

```cpp
///      ... callee may access any position of our current stack frame
///    }
///
class FrameAnalysis {
  BinaryContext &BC;

  /// Map functions to the set of <stack offsets, size> tuples representing
  /// accesses to stack positions that belongs to caller
  std::map<const BinaryFunction *, std::set<std::pair<int64_t, uint8_t>>>
      ArgsTouchedMap;
```

- EN: Introduces type definitions such as `FrameAnalysis`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `FrameAnalysis`.
- CN: 这里引入类型定义，例如 `FrameAnalysis`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `FrameAnalysis`。

### Lines 116-126

```cpp
  /// The set of functions we were able to perform the full analysis up to
  /// restoring frame indexes for all load/store instructions.
  DenseSet<const BinaryFunction *> AnalyzedFunctions;

  /// Set of functions that require the stack to be 16B aligned
  DenseSet<const BinaryFunction *> FunctionsRequireAlignment;

  /// Set of functions that performs computations with stack addresses and
  /// complicates our understanding of aliasing of stack spaces.
  DenseSet<const BinaryFunction *> FunctionsWithStackArithmetic;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 127-138

```cpp
  /// Owns ArgAccesses for all instructions. References to elements are
  /// attached to instructions as indexes to this vector, in MCAnnotations.
  std::vector<ArgAccesses> ArgAccessesVector;
  /// Same for FrameIndexEntries.
  std::vector<FrameIndexEntry> FIEVector;

  /// Analysis stats counters
  uint64_t NumFunctionsNotOptimized{0};
  uint64_t NumFunctionsFailedRestoreFI{0};
  uint64_t CountFunctionsFailedRestoreFI{0};
  uint64_t CountDenominator{0};
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 139-148

```cpp
  /// Convenience functions for appending MCAnnotations to instructions with
  /// our specific data
  void addArgAccessesFor(MCInst &Inst, ArgAccesses &&AA);
  void addArgInStackAccessFor(MCInst &Inst, const ArgInStackAccess &Arg);
  void addFIEFor(MCInst &Inst, const FrameIndexEntry &FIE);

  /// Perform the step of building the set of registers clobbered by each
  /// function execution, populating RegsKilledMap and RegsGenMap.
  void traverseCG(BinaryFunctionCallGraph &CG);
```

- EN: Declares or implements routines including `addArgAccessesFor`, `addArgInStackAccessFor`, `addFIEFor`, `traverseCG`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `addArgAccessesFor`, `addArgInStackAccessFor`, `addFIEFor`, `traverseCG`.
- CN: 这里声明或实现函数，例如 `addArgAccessesFor`, `addArgInStackAccessFor`, `addFIEFor`, `traverseCG`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `addArgAccessesFor`, `addArgInStackAccessFor`, `addFIEFor`, `traverseCG`。

### Lines 149-160

```cpp
  /// Analyzes an instruction and if it is a call, checks the called function
  /// to record which args in stack are accessed, if any. Returns true if
  /// the args data associated with this instruction were updated.
  bool updateArgsTouchedFor(const BinaryFunction &BF, MCInst &Inst,
                            int CurOffset);

  /// Performs a pass over \p BF to check for accesses to arguments in stack,
  /// flagging those as accessing the caller stack frame. All functions called
  /// by \p BF must have been previously analyzed. Returns true if updated
  /// args data about this function.
  bool computeArgsAccessed(BinaryFunction &BF);
```

- EN: Declares or implements routines including `computeArgsAccessed`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `computeArgsAccessed`.
- CN: 这里声明或实现函数，例如 `computeArgsAccessed`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `computeArgsAccessed`。

### Lines 161-171

```cpp
  /// Alias analysis to disambiguate which frame position is accessed by each
  /// instruction in function \p BF. Add MCAnnotation<FrameIndexEntry> to
  /// instructions that access a frame position. Return false if it failed
  /// to analyze and this information can't be safely determined for \p BF.
  bool restoreFrameIndex(BinaryFunction &BF);

  /// A store for SPT info per function
  std::unordered_map<const BinaryFunction *,
                     std::unique_ptr<StackPointerTracking>>
      SPTMap;
```

- EN: Declares or implements routines including `restoreFrameIndex`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `restoreFrameIndex`.
- CN: 这里声明或实现函数，例如 `restoreFrameIndex`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `restoreFrameIndex`。

### Lines 172-179

```cpp
public:
  explicit FrameAnalysis(BinaryContext &BC, BinaryFunctionCallGraph &CG);

  /// Return true if we could fully analyze \p Func
  bool hasFrameInfo(const BinaryFunction &Func) const {
    return AnalyzedFunctions.count(&Func);
  }
```

- EN: Declares or implements routines including `FrameAnalysis`, `hasFrameInfo`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `FrameAnalysis`, `hasFrameInfo`.
- CN: 这里声明或实现函数，例如 `FrameAnalysis`, `hasFrameInfo`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `FrameAnalysis`, `hasFrameInfo`。

### Lines 180-190

```cpp
  /// Return true if \p Func cannot operate with a misaligned CFA
  bool requiresAlignment(const BinaryFunction &Func) const {
    return FunctionsRequireAlignment.count(&Func);
  }

  /// Return true if \p Func does computation with the address of any stack
  /// position, meaning we have limited alias analysis on this function.
  bool hasStackArithmetic(const BinaryFunction &Func) const {
    return FunctionsWithStackArithmetic.count(&Func);
  }
```

- EN: Declares or implements routines including `requiresAlignment`, `hasStackArithmetic`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `requiresAlignment`, `hasStackArithmetic`.
- CN: 这里声明或实现函数，例如 `requiresAlignment`, `hasStackArithmetic`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `requiresAlignment`, `hasStackArithmetic`。

### Lines 191-200

```cpp
  /// Functions for retrieving our specific MCAnnotation data from instructions
  ErrorOr<ArgAccesses &> getArgAccessesFor(const MCInst &Inst);

  ErrorOr<const ArgAccesses &> getArgAccessesFor(const MCInst &Inst) const;

  ErrorOr<const FrameIndexEntry &> getFIEFor(const MCInst &Inst) const;

  /// Remove all MCAnnotations attached by this pass
  void cleanAnnotations();
```

- EN: Declares or implements routines including `getArgAccessesFor`, `getFIEFor`, `cleanAnnotations`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getArgAccessesFor`, `getFIEFor`, `cleanAnnotations`.
- CN: 这里声明或实现函数，例如 `getArgAccessesFor`, `getFIEFor`, `cleanAnnotations`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getArgAccessesFor`, `getFIEFor`, `cleanAnnotations`。

### Lines 201-216

```cpp
  ~FrameAnalysis() { cleanAnnotations(); }

  /// Print to standard output statistics about the analysis performed by this
  /// pass
  void printStats();

  /// Get or create an SPT object and run the analysis
  StackPointerTracking &getSPT(BinaryFunction &BF) {
    if (!SPTMap.count(&BF)) {
      SPTMap.emplace(&BF, std::make_unique<StackPointerTracking>(BF));
      auto Iter = SPTMap.find(&BF);
      assert(Iter != SPTMap.end() && "item should exist");
      Iter->second->run();
      return *Iter->second;
    }
```

- EN: Declares or implements routines including `FrameAnalysis`, `printStats`, `getSPT`, `assert`, `run`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `FrameAnalysis`, `printStats`, `getSPT`, `assert`, `run`.
- CN: 这里声明或实现函数，例如 `FrameAnalysis`, `printStats`, `getSPT`, `assert`, `run`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `FrameAnalysis`, `printStats`, `getSPT`, `assert`, `run`。

### Lines 217-224

```cpp
    auto Iter = SPTMap.find(&BF);
    assert(Iter != SPTMap.end() && "item should exist");
    return *Iter->second;
  }

  /// Clean and de-allocate all SPT objects
  void clearSPTMap();
```

- EN: Declares or implements routines including `assert`, `clearSPTMap`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `assert`, `clearSPTMap`.
- CN: 这里声明或实现函数，例如 `assert`, `clearSPTMap`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `assert`, `clearSPTMap`。

### Lines 225-232

```cpp
  /// Perform SPT analysis for all functions in parallel
  void preComputeSPT();
};

} // namespace bolt
} // namespace llvm

#endif
```

- EN: Uses conditional compilation to choose platform- or feature-specific code paths. Works inside namespace scope `bolt`, `llvm` to organize symbols. Declares or implements routines including `preComputeSPT`. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里使用条件编译，根据平台或特性切换不同代码路径。这里位于命名空间 `bolt`, `llvm` 中，用于组织符号作用域。这里声明或实现函数，例如 `preComputeSPT`。这里的注释记录了设计假设、不变量或使用说明。

## Key Concepts / 关键概念

- `BinaryFunctionCallGraph`: class or struct interface / 类或结构体接口
- `FrameIndexEntry`: class or struct interface / 类或结构体接口
- `ArgInStackAccess`: class or struct interface / 类或结构体接口
- `ArgAccesses`: class or struct interface / 类或结构体接口
- `ArgAccesses`: function or method entry point / 函数或方法入口
- `AssumeEverything`: function or method entry point / 函数或方法入口
- `addArgAccessesFor`: function or method entry point / 函数或方法入口
- `addArgInStackAccessFor`: function or method entry point / 函数或方法入口

## Dependencies / 依赖关系

- Local headers / 本地头文件: `bolt/Passes/StackPointerTracking.h`
- System headers / 系统头文件: `tuple`
- Directory context / 目录上下文: `bolt/include/bolt/Passes` neighbors usually cooperate with this file to provide the surrounding subsystem / `bolt/include/bolt/Passes` 下的相邻文件通常与本文件协作组成对应子系统
