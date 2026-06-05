# HexagonVectorLoopCarriedReuse.h — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonVectorLoopCarriedReuse.h`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Declares Hexagon vector loop-carried reuse optimization.
- Purpose / 作用 (CN): 该文件声明 Hexagon 后端组件的接口、类型或辅助入口。 重点涉及循环优化。 重点涉及 HVX/向量处理。

## Line-by-Line Analysis / 逐行分析

### Lines 1-25 / 第 1-25 行

```cpp
     1: //===- HexagonVectorLoopCarriedReuse.h ------------------------------------===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: //
     9: // This pass removes the computation of provably redundant expressions that have
    10: // been computed earlier in a previous iteration. It relies on the use of PHIs
    11: // to identify loop carried dependences. This is scalar replacement for vector
    12: // types.
    13: //
    14: //-----------------------------------------------------------------------------
    15: // Motivation: Consider the case where we have the following loop structure.
    16: //
    17: // Loop:
    18: //  t0 = a[i];
    19: //  t1 = f(t0);
    20: //  t2 = g(t1);
    21: //  ...
    22: //  t3 = a[i+1];
    23: //  t4 = f(t3);
    24: //  t5 = g(t4);
    25: //  t6 = op(t2, t5)
```
- EN: This range is dominated by comments or banner text that documents the surrounding section. It declares or implements routines such as f, g, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonVectorLoopCarriedReuse, showing how the code connects to sibling backend components.
- CN: 这一段主要由注释或分隔说明组成，用来解释后续代码区域。 这里声明或实现了 f, g 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonVectorLoopCarriedReuse，说明了它与同级后端组件的连接关系。

### Lines 26-50 / 第 26-50 行

```cpp
    26: //  cond_branch <Loop>
    27: //
    28: // This can be converted to
    29: //  t00 = a[0];
    30: //  t10 = f(t00);
    31: //  t20 = g(t10);
    32: // Loop:
    33: //  t2 = t20;
    34: //  t3 = a[i+1];
    35: //  t4 = f(t3);
    36: //  t5 = g(t4);
    37: //  t6 = op(t2, t5)
    38: //  t20 = t5
    39: //  cond_branch <Loop>
    40: //
    41: // SROA does a good job of reusing a[i+1] as a[i] in the next iteration.
    42: // Such a loop comes to this pass in the following form.
    43: //
    44: // LoopPreheader:
    45: //  X0 = a[0];
    46: // Loop:
    47: //  X2 = PHI<(X0, LoopPreheader), (X1, Loop)>
    48: //  t1 = f(X2)   <-- I1
    49: //  t2 = g(t1)
    50: //  ...
```
- EN: This range is dominated by comments or banner text that documents the surrounding section. It declares or implements routines such as f, g, translating Hexagon-specific policy into reusable code paths.
- CN: 这一段主要由注释或分隔说明组成，用来解释后续代码区域。 这里声明或实现了 f, g 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

### Lines 51-75 / 第 51-75 行

```cpp
    51: //  X1 = a[i+1]
    52: //  t4 = f(X1)   <-- I2
    53: //  t5 = g(t4)
    54: //  t6 = op(t2, t5)
    55: //  cond_branch <Loop>
    56: //
    57: // In this pass, we look for PHIs such as X2 whose incoming values come only
    58: // from the Loop Preheader and over the backedge and additionally, both these
    59: // values are the results of the same operation in terms of opcode. We call such
    60: // a PHI node a dependence chain or DepChain. In this case, the dependence of X2
    61: // over X1 is carried over only one iteration and so the DepChain is only one
    62: // PHI node long.
    63: //
    64: // Then, we traverse the uses of the PHI (X2) and the uses of the value of the
    65: // PHI coming  over the backedge (X1). We stop at the first pair of such users
    66: // I1 (of X2) and I2 (of X1) that meet the following conditions.
    67: // 1. I1 and I2 are the same operation, but with different operands.
    68: // 2. X2 and X1 are used at the same operand number in the two instructions.
    69: // 3. All other operands Op1 of I1 and Op2 of I2 are also such that there is a
    70: //    a DepChain from Op1 to Op2 of the same length as that between X2 and X1.
    71: //
    72: // We then make the following transformation
    73: // LoopPreheader:
    74: //  X0 = a[0];
    75: //  Y0 = f(X0);
```
- EN: This range is dominated by comments or banner text that documents the surrounding section. It declares or implements routines such as f, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这一段主要由注释或分隔说明组成，用来解释后续代码区域。 这里声明或实现了 f 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 76-100 / 第 76-100 行

```cpp
    76: // Loop:
    77: //  X2 = PHI<(X0, LoopPreheader), (X1, Loop)>
    78: //  Y2 = PHI<(Y0, LoopPreheader), (t4, Loop)>
    79: //  t1 = f(X2)   <-- Will be removed by DCE.
    80: //  t2 = g(Y2)
    81: //  ...
    82: //  X1 = a[i+1]
    83: //  t4 = f(X1)
    84: //  t5 = g(t4)
    85: //  t6 = op(t2, t5)
    86: //  cond_branch <Loop>
    87: //
    88: // We proceed until we cannot find any more such instructions I1 and I2.
    89: //
    90: // --- DepChains & Loop carried dependences ---
    91: // Consider a single basic block loop such as
    92: //
    93: // LoopPreheader:
    94: //  X0 = ...
    95: //  Y0 = ...
    96: // Loop:
    97: //  X2 = PHI<(X0, LoopPreheader), (X1, Loop)>
    98: //  Y2 = PHI<(Y0, LoopPreheader), (X2, Loop)>
    99: //  ...
   100: //  X1 = ...
```
- EN: This range is dominated by comments or banner text that documents the surrounding section.
- CN: 这一段主要由注释或分隔说明组成，用来解释后续代码区域。

### Lines 101-125 / 第 101-125 行

```cpp
   101: //  ...
   102: //  cond_branch <Loop>
   103: //
   104: // Then there is a dependence between X2 and X1 that goes back one iteration,
   105: // i.e. X1 is used as X2 in the very next iteration. We represent this as a
   106: // DepChain from X2 to X1 (X2->X1).
   107: // Similarly, there is a dependence between Y2 and X1 that goes back two
   108: // iterations. X1 is used as Y2 two iterations after it is computed. This is
   109: // represented by a DepChain as (Y2->X2->X1).
   110: //
   111: // A DepChain has the following properties.
   112: // 1. Num of edges in DepChain = Number of Instructions in DepChain = Number of
   113: //    iterations of carried dependence + 1.
   114: // 2. All instructions in the DepChain except the last are PHIs.
   115: //
   116: //===----------------------------------------------------------------------===//
   117: 
   118: #ifndef LLVM_LIB_TARGET_HEXAGON_HEXAGONVLCR_H
   119: #define LLVM_LIB_TARGET_HEXAGON_HEXAGONVLCR_H
   120: 
   121: #include "llvm/Transforms/Scalar/LoopPassManager.h"
   122: 
   123: namespace llvm {
   124: 
   125: class Loop;
```
- EN: This range is dominated by comments or banner text that documents the surrounding section. It imports headers such as llvm/Transforms/Scalar/LoopPassManager.h, establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components.
- CN: 这一段主要由注释或分隔说明组成，用来解释后续代码区域。 这里引入了 llvm/Transforms/Scalar/LoopPassManager.h 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。

### Lines 126-139 / 第 126-139 行

```cpp
   126: 
   127: /// Hexagon Vector Loop Carried Reuse Pass
   128: struct HexagonVectorLoopCarriedReusePass
   129:     : public OptionalPassInfoMixin<HexagonVectorLoopCarriedReusePass> {
   130:   HexagonVectorLoopCarriedReusePass() = default;
   131: 
   132:   /// Run pass over the Loop.
   133:   PreservedAnalyses run(Loop &L, LoopAnalysisManager &LAM,
   134:                         LoopStandardAnalysisResults &AR, LPMUpdater &U);
   135: };
   136: 
   137: } // end namespace llvm
   138: 
   139: #endif // LLVM_LIB_TARGET_HEXAGON_HEXAGONVLCR_H
```
- EN: It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as HexagonVectorLoopCarriedReusePass, which carry the state or API of this component. It declares or implements routines such as HexagonVectorLoopCarriedReusePass, run, translating Hexagon-specific policy into reusable code paths. Notable Hexagon symbols referenced here include HexagonVectorLoopCarriedReusePass, showing how the code connects to sibling backend components.
- CN: 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 HexagonVectorLoopCarriedReusePass 等类型，用来承载该组件的状态或接口。 这里声明或实现了 HexagonVectorLoopCarriedReusePass, run 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这里引用的重要 Hexagon 符号包括 HexagonVectorLoopCarriedReusePass，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- loop-aware code generation / 循环相关代码生成
- instruction semantics / 指令语义
- generated metadata tables / 生成的元数据表

## Dependencies / 依赖关系

- Direct includes / 直接包含: `llvm/Transforms/Scalar/LoopPassManager.h`
- Hexagon symbols / Hexagon 符号: `HexagonVectorLoopCarriedReuse, HexagonVectorLoopCarriedReusePass`
- Interface role / 接口角色: included by sibling Hexagon implementation files to share declarations safely. / 由同级 Hexagon 实现文件包含，以共享声明。
