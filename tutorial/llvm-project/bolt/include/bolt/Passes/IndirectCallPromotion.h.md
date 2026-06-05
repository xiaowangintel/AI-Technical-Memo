# IndirectCallPromotion.h — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `bolt/include/bolt/Passes/IndirectCallPromotion.h`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: *- C++ -*. It also sits in code that declares BOLT optimization and transformation pass interfaces. / 该文件声明 BOLT 优化与变换 Pass 接口。 源码头部说明其职责是：*- C++ -*。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12

```cpp
//===- bolt/Passes/IndirectCallPromotion.h ----------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// The indirect call promotion (ICP) optimization pass.
//
//===----------------------------------------------------------------------===//
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 13-20

```cpp
#ifndef BOLT_PASSES_INDIRECT_CALL_PROMOTION_H
#define BOLT_PASSES_INDIRECT_CALL_PROMOTION_H

#include "bolt/Passes/BinaryPasses.h"

namespace llvm {
namespace bolt {
```

- EN: Pulls in 1 header(s) from local project dependencies needed by this range. Establishes header-guard or prologue macros so the file can be compiled safely. Works inside namespace scope `llvm`, `bolt` to organize symbols. Defines macros such as `BOLT_PASSES_INDIRECT_CALL_PROMOTION_H` for constants or compile-time switches.
- CN: 这里引入 1 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里建立头文件保护或前导宏，保证源码可被安全编译与重复包含控制。这里位于命名空间 `llvm`, `bolt` 中，用于组织符号作用域。这里定义宏 `BOLT_PASSES_INDIRECT_CALL_PROMOTION_H`，用于常量或编译期开关。

### Lines 21-38

```cpp
/// Optimize indirect calls.
/// The indirect call promotion pass visits each indirect call and
/// examines a branch profile for each. If the most frequent targets
/// from that callsite exceed the specified threshold (default 90%),
/// the call is promoted. Otherwise, it is ignored. By default,
/// only one target is considered at each callsite.
///
/// When an candidate callsite is processed, we modify the callsite
/// to test for the most common call targets before calling through
/// the original generic call mechanism.
///
/// The CFG and layout are modified by ICP.
///
/// A few new command line options have been added:
///   -indirect-call-promotion=[none,call,jump-tables,all]
///   -indirect-call-promotion-threshold=<percentage>
///   -indirect-call-promotion-mispredict-threshold=<percentage>
///   -indirect-call-promotion-topn=<int>
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 39-56

```cpp
///
/// The threshold is the minimum frequency of a call target needed
/// before ICP is triggered.
///
/// The mispredict threshold is used to disable the optimization at
/// any callsite where the branch predictor does a good enough job
/// that ICP wouldn't help regardless of the frequency of the most
/// common target.
///
/// The topn option controls the number of targets to consider for
/// each callsite, e.g. ICP is triggered if topn=2 and the total
/// frequency of the top two call targets exceeds the threshold.
///
/// The minimize code size option controls whether or not the hot
/// calls are to registers (callq %r10) or to function addresses
/// (callq $foo).
///
/// Example of ICP:
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 57-74

```cpp
///
/// C++ code:
///
///   int B_count = 0;
///   int C_count = 0;
///
///   struct A { virtual void foo() = 0; }
///   struct B : public A { virtual void foo() { ++B_count; }; };
///   struct C : public A { virtual void foo() { ++C_count; }; };
///
///   A* a = ...
///   a->foo();
///   ...
///
/// original assembly:
///
///   B0: 49 8b 07             mov    (%r15),%rax
///       4c 89 ff             mov    %r15,%rdi
```

- EN: Introduces type definitions such as `A`, `B`, `C`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `A`, `B`, `C`.
- CN: 这里引入类型定义，例如 `A`, `B`, `C`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `A`, `B`, `C`。

### Lines 75-92

```cpp
///       ff 10                callq  *(%rax)
///       41 83 e6 01          and    $0x1,%r14d
///       4d 89 e6             mov    %r12,%r14
///       4c 0f 44 f5          cmove  %rbp,%r14
///       4c 89 f7             mov    %r14,%rdi
///       ...
///
/// after ICP:
///
///   B0: 49 8b 07             mov    (%r15),%rax
///       4c 89 ff             mov    %r15,%rdi
///       48 81 38 e0 0b 40 00 cmpq   $B::foo,(%rax)
///       75 29                jne    B3
///   B1: e8 45 03 00 00       callq  $B::foo
///   B2: 41 83 e6 01          and    $0x1,%r14d
///       4d 89 e6             mov    %r12,%r14
///       4c 0f 44 f5          cmove  %rbp,%r14
///       4c 89 f7             mov    %r14,%rdi
```

- EN: Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里的注释记录了设计假设、不变量或使用说明。

### Lines 93-104

```cpp
///       ...
///
///   B3: ff 10                callq  *(%rax)
///       eb d6                jmp    B2
///
class IndirectCallPromotion : public BinaryFunctionPass {
  using BasicBlocksVector = std::vector<std::unique_ptr<BinaryBasicBlock>>;
  using MethodInfoType = std::pair<std::vector<std::pair<MCSymbol *, uint64_t>>,
                                   std::vector<MCInst *>>;
  using JumpTableInfoType = std::vector<std::pair<uint64_t, uint64_t>>;
  using SymTargetsType = std::vector<std::pair<MCSymbol *, uint64_t>>;
  struct Location {
```

- EN: Introduces type definitions such as `IndirectCallPromotion`, `Location`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `IndirectCallPromotion`, `Location`.
- CN: 这里引入类型定义，例如 `IndirectCallPromotion`, `Location`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `IndirectCallPromotion`, `Location`。

### Lines 105-112

```cpp
    MCSymbol *Sym{nullptr};
    uint64_t Addr{0};
    bool isValid() const { return Sym || Addr != 0; }
    Location() {}
    explicit Location(MCSymbol *Sym) : Sym(Sym) {}
    explicit Location(uint64_t Addr) : Addr(Addr) {}
  };
```

- EN: Declares or implements routines including `isValid`, `Location`. Notable symbols here include `isValid`, `Location`.
- CN: 这里声明或实现函数，例如 `isValid`, `Location`。这里较值得关注的符号包括 `isValid`, `Location`。

### Lines 113-127

```cpp
  struct Callsite {
    Location From;
    Location To;
    uint64_t Mispreds{0};
    uint64_t Branches{0};
    // Indices in the jmp table (jt only)
    std::vector<uint64_t> JTIndices;
    bool isValid() const { return From.isValid() && To.isValid(); }
    Callsite(BinaryFunction &BF, const IndirectCallProfile &ICP);
    Callsite(const Location &From, const Location &To, uint64_t Mispreds,
             uint64_t Branches, uint64_t JTIndex)
        : From(From), To(To), Mispreds(Mispreds), Branches(Branches),
          JTIndices(1, JTIndex) {}
  };
```

- EN: Introduces type definitions such as `Callsite`. Declares or implements routines including `isValid`, `Callsite`, `From`, `JTIndices`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `Callsite`, `isValid`, `From`, `JTIndices`.
- CN: 这里引入类型定义，例如 `Callsite`。这里声明或实现函数，例如 `isValid`, `Callsite`, `From`, `JTIndices`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `Callsite`, `isValid`, `From`, `JTIndices`。

### Lines 128-135

```cpp
  std::unordered_set<const BinaryFunction *> Modified;
  // Total number of calls from all callsites.
  uint64_t TotalCalls{0};

  // Total number of indirect calls from all callsites.
  // (a fraction of TotalCalls)
  uint64_t TotalIndirectCalls{0};
```

- EN: Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里的注释记录了设计假设、不变量或使用说明。

### Lines 136-143

```cpp
  // Total number of jmp table calls from all callsites.
  // (a fraction of TotalCalls)
  uint64_t TotalIndirectJmps{0};

  // Total number of callsites that use indirect calls.
  // (the total number of callsites is not recorded)
  uint64_t TotalIndirectCallsites{0};
```

- EN: Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里的注释记录了设计假设、不变量或使用说明。

### Lines 144-153

```cpp
  // Total number of callsites that are jump tables.
  uint64_t TotalJumpTableCallsites{0};

  // Total number of indirect callsites that are optimized by ICP.
  // (a fraction of TotalIndirectCallsites)
  uint64_t TotalOptimizedIndirectCallsites{0};

  // Total number of method callsites that can have loads eliminated.
  mutable uint64_t TotalMethodLoadEliminationCandidates{0};
```

- EN: Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里的注释记录了设计假设、不变量或使用说明。

### Lines 154-163

```cpp
  // Total number of method callsites that had loads eliminated.
  uint64_t TotalMethodLoadsEliminated{0};

  // Total number of jump table callsites that are optimized by ICP.
  uint64_t TotalOptimizedJumpTableCallsites{0};

  // Total number of indirect calls that are optimized by ICP.
  // (a fraction of TotalCalls)
  uint64_t TotalNumFrequentCalls{0};
```

- EN: Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里的注释记录了设计假设、不变量或使用说明。

### Lines 164-173

```cpp
  // Total number of jump table calls that are optimized by ICP.
  // (a fraction of TotalCalls)
  uint64_t TotalNumFrequentJmps{0};

  // Total number of jump table sites that can use hot indices.
  mutable uint64_t TotalIndexBasedCandidates{0};

  // Total number of jump table sites that use hot indices.
  uint64_t TotalIndexBasedJumps{0};
```

- EN: Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里的注释记录了设计假设、不变量或使用说明。

### Lines 174-184

```cpp
  void printDecision(llvm::raw_ostream &OS,
                     std::vector<IndirectCallPromotion::Callsite> &Targets,
                     unsigned N) const;

  std::vector<Callsite> getCallTargets(BinaryBasicBlock &BB,
                                       const MCInst &Inst) const;

  size_t canPromoteCallsite(const BinaryBasicBlock &BB, const MCInst &Inst,
                            const std::vector<Callsite> &Targets,
                            uint64_t NumCalls);
```

- EN: Continues the surrounding implementation, wiring declarations, state, and helper logic together.
- CN: 这里继续展开周边实现，衔接声明、状态与辅助逻辑。

### Lines 185-193

```cpp
  void printCallsiteInfo(const BinaryBasicBlock &BB, const MCInst &Inst,
                         const std::vector<Callsite> &Targets, const size_t N,
                         uint64_t NumCalls) const;

  JumpTableInfoType maybeGetHotJumpTableTargets(BinaryBasicBlock &BB,
                                                MCInst &Inst,
                                                MCInst *&TargetFetchInst,
                                                const JumpTable *JT) const;
```

- EN: Continues the surrounding implementation, wiring declarations, state, and helper logic together.
- CN: 这里继续展开周边实现，衔接声明、状态与辅助逻辑。

### Lines 194-201

```cpp
  SymTargetsType findCallTargetSymbols(std::vector<Callsite> &Targets,
                                       size_t &N, BinaryBasicBlock &BB,
                                       MCInst &Inst,
                                       MCInst *&TargetFetchInst) const;

  MethodInfoType maybeGetVtableSyms(BinaryBasicBlock &BB, MCInst &Inst,
                                    const SymTargetsType &SymTargets) const;
```

- EN: Continues the surrounding implementation, wiring declarations, state, and helper logic together.
- CN: 这里继续展开周边实现，衔接声明、状态与辅助逻辑。

### Lines 202-211

```cpp
  std::vector<std::unique_ptr<BinaryBasicBlock>>
  rewriteCall(BinaryBasicBlock &IndCallBlock, const MCInst &CallInst,
              MCPlusBuilder::BlocksVectorTy &&ICPcode,
              const std::vector<MCInst *> &MethodFetchInsns) const;

  BinaryBasicBlock *fixCFG(BinaryBasicBlock &IndCallBlock,
                           const bool IsTailCall, const bool IsJumpTable,
                           BasicBlocksVector &&NewBBs,
                           const std::vector<Callsite> &Targets) const;
```

- EN: Continues the surrounding implementation, wiring declarations, state, and helper logic together.
- CN: 这里继续展开周边实现，衔接声明、状态与辅助逻辑。

### Lines 212-226

```cpp
public:
  explicit IndirectCallPromotion(const cl::opt<bool> &PrintPass)
      : BinaryFunctionPass(PrintPass) {}

  const char *getName() const override { return "indirect-call-promotion"; }
  bool shouldPrint(const BinaryFunction &BF) const override {
    return BinaryFunctionPass::shouldPrint(BF) && Modified.count(&BF) > 0;
  }
  bool shouldOptimize(const BinaryFunction &BF) const override {
    return BF.isSimple() && !BF.isIgnored() && BF.hasProfile() &&
           !BF.hasUnknownControlFlow();
  }
  Error runOnFunctions(BinaryContext &BC) override;
};
```

- EN: Declares or implements routines including `IndirectCallPromotion`, `BinaryFunctionPass`, `getName`, `shouldPrint`, `shouldOptimize`, and 1 more. Notable symbols here include `IndirectCallPromotion`, `BinaryFunctionPass`, `getName`, `shouldPrint`, `shouldOptimize`, `runOnFunctions`.
- CN: 这里声明或实现函数，例如 `IndirectCallPromotion`, `BinaryFunctionPass`, `getName`, `shouldPrint`, `shouldOptimize`, and 1 more。这里较值得关注的符号包括 `IndirectCallPromotion`, `BinaryFunctionPass`, `getName`, `shouldPrint`, `shouldOptimize`, `runOnFunctions`。

### Lines 227-230

```cpp
} // namespace bolt
} // namespace llvm

#endif
```

- EN: Uses conditional compilation to choose platform- or feature-specific code paths. Works inside namespace scope `bolt`, `llvm` to organize symbols. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `bolt`, `llvm`.
- CN: 这里使用条件编译，根据平台或特性切换不同代码路径。这里位于命名空间 `bolt`, `llvm` 中，用于组织符号作用域。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `bolt`, `llvm`。

## Key Concepts / 关键概念

- `A`: class or struct interface / 类或结构体接口
- `B`: class or struct interface / 类或结构体接口
- `C`: class or struct interface / 类或结构体接口
- `IndirectCallPromotion`: class or struct interface / 类或结构体接口
- `isValid`: function or method entry point / 函数或方法入口
- `Location`: function or method entry point / 函数或方法入口
- `Callsite`: function or method entry point / 函数或方法入口
- `From`: function or method entry point / 函数或方法入口

## Dependencies / 依赖关系

- Local headers / 本地头文件: `bolt/Passes/BinaryPasses.h`
- Directory context / 目录上下文: `bolt/include/bolt/Passes` neighbors usually cooperate with this file to provide the surrounding subsystem / `bolt/include/bolt/Passes` 下的相邻文件通常与本文件协作组成对应子系统
