# Canonicalization.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `polly/lib/Transform/Canonicalization.cpp`
- **Repository**: `llvm-project` / `llvm-project`
- **Purpose (EN)**: Canonicalizes LLVM IR into a form expected by Polly analyses and transformations.
- **用途（CN）**: 将 LLVM IR 规范化为 Polly 分析与变换期望的形式。

## Line-by-Line Analysis / 逐行分析

### Lines 1-13
```cpp
1: //===---- Canonicalization.cpp - Run canonicalization passes --------------===//
2: //
3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
4: // See https://llvm.org/LICENSE.txt for license information.
5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
6: //
7: //===----------------------------------------------------------------------===//
8: //
9: // Run the set of default canonicalization passes.
10: //
11: // This pass is mainly used for debugging.
12: //
13: //===----------------------------------------------------------------------===//
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 15-31
```cpp
15: #include "polly/Canonicalization.h"
16: #include "polly/Options.h"
17: #include "llvm/Analysis/GlobalsModRef.h"
18: #include "llvm/Analysis/ProfileSummaryInfo.h"
19: #include "llvm/IR/LegacyPassManager.h"
20: #include "llvm/Transforms/IPO.h"
21: #include "llvm/Transforms/IPO/FunctionAttrs.h"
22: #include "llvm/Transforms/InstCombine/InstCombine.h"
23: #include "llvm/Transforms/Scalar.h"
24: #include "llvm/Transforms/Scalar/EarlyCSE.h"
25: #include "llvm/Transforms/Scalar/IndVarSimplify.h"
26: #include "llvm/Transforms/Scalar/LoopRotation.h"
27: #include "llvm/Transforms/Scalar/Reassociate.h"
28: #include "llvm/Transforms/Scalar/SimplifyCFG.h"
29: #include "llvm/Transforms/Scalar/TailRecursionElimination.h"
30: #include "llvm/Transforms/Utils.h"
31: #include "llvm/Transforms/Utils/Mem2Reg.h"
```
- **EN**: Brings in the headers required by this compilation unit, covering Polly, LLVM, ISL, or standard-library facilities.
- **CN**: 这里引入当前编译单元所需的头文件，覆盖 Polly、LLVM、ISL 或标准库能力。

### Lines 33-33
```cpp
33: using namespace llvm;
```
- **EN**: Introduces namespace aliases/shorthands so later code can refer to LLVM or Polly entities more compactly.
- **CN**: 这里引入命名空间简写，使后续代码能更紧凑地引用 LLVM 或 Polly 实体。

### Lines 34-34
```cpp
34: using namespace polly;
```
- **EN**: Introduces namespace aliases/shorthands so later code can refer to LLVM or Polly entities more compactly.
- **CN**: 这里引入命名空间简写，使后续代码能更紧凑地引用 LLVM 或 Polly 实体。

### Lines 36-39
```cpp
36: static cl::opt<bool>
37:     PollyInliner("polly-run-inliner",
38:                  cl::desc("Run an early inliner pass before Polly"), cl::Hidden,
39:                  cl::cat(PollyCategory));
```
- **EN**: Declares the command-line option `PollyInliner`, allowing Polly to tune this optimization according to target or experiment settings.
- **CN**: 这里声明命令行选项 `PollyInliner`，使 Polly 能根据目标平台或实验设置调节该优化。

### Lines 41-41
```cpp
41: /// Adapted from llvm::PassBuilder::buildInlinerPipeline
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 42-45
```cpp
42: static ModuleInlinerWrapperPass
43: buildInlinePasses(llvm::OptimizationLevel Level) {
44:   InlineParams IP = getInlineParams(200);
45:   ModuleInlinerWrapperPass MIWP(IP);
```
- **EN**: Introduces or continues `buildInlinePasses`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `buildInlinePasses`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 47-48
```cpp
47:   // Require the GlobalsAA analysis for the module so we can query it within
48:   // the CGSCC pipeline.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 49-49
```cpp
49:   MIWP.addModulePass(RequireAnalysisPass<GlobalsAA, Module>());
```
- **EN**: Introduces or continues `addModulePass`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `addModulePass`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 50-51
```cpp
50:   // Invalidate AAManager so it can be recreated and pick up the newly available
51:   // GlobalsAA.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 52-53
```cpp
52:   MIWP.addModulePass(
53:       createModuleToFunctionPassAdaptor(InvalidateAnalysisPass<AAManager>()));
```
- **EN**: Introduces or continues `addModulePass`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `addModulePass`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 55-56
```cpp
55:   // Require the ProfileSummaryAnalysis for the module so we can query it within
56:   // the inliner pass.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 57-57
```cpp
57:   MIWP.addModulePass(RequireAnalysisPass<ProfileSummaryAnalysis, Module>());
```
- **EN**: Introduces or continues `addModulePass`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `addModulePass`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 59-62
```cpp
59:   // Now begin the main postorder CGSCC pipeline.
60:   // FIXME: The current CGSCC pipeline has its origins in the legacy pass
61:   // manager and trying to emulate its precise behavior. Much of this doesn't
62:   // make a lot of sense and we should revisit the core CGSCC structure.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 63-63
```cpp
63:   CGSCCPassManager &MainCGPipeline = MIWP.getPM();
```
- **EN**: Introduces or continues `getPM`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `getPM`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 65-65
```cpp
65:   // Now deduce any function attributes based in the current code.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 66-66
```cpp
66:   MainCGPipeline.addPass(PostOrderFunctionAttrsPass());
```
- **EN**: Introduces or continues `addPass`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `addPass`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 68-68
```cpp
68:   return MIWP;
```
- **EN**: Returns the computed value or forwards the current result to the caller.
- **CN**: 这里返回计算结果，或将当前结果继续传递给调用方。

### Lines 69-69
```cpp
69: }
```
- **EN**: Closes the current block or scope and hands control back to the surrounding context.
- **CN**: 这里关闭当前代码块或作用域，并将控制流交还给外层上下文。

### Lines 71-74
```cpp
71: FunctionPassManager
72: polly::buildCanonicalicationPassesForNPM(llvm::ModulePassManager &MPM,
73:                                          llvm::OptimizationLevel Level) {
74:   FunctionPassManager FPM;
```
- **EN**: Introduces or continues `polly::buildCanonicalicationPassesForNPM`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `polly::buildCanonicalicationPassesForNPM`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 76-76
```cpp
76:   bool UseMemSSA = true;
```
- **EN**: Carries supporting computation, data movement, or bookkeeping for the surrounding algorithm.
- **CN**: 这里承担周边算法所需的辅助计算、数据传递或簿记工作。

### Lines 77-77
```cpp
77:   FPM.addPass(PromotePass());
```
- **EN**: Introduces or continues `addPass`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `addPass`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 78-78
```cpp
78:   FPM.addPass(EarlyCSEPass(UseMemSSA));
```
- **EN**: Introduces or continues `addPass`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `addPass`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 79-79
```cpp
79:   FPM.addPass(InstCombinePass());
```
- **EN**: Introduces or continues `addPass`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `addPass`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 80-80
```cpp
80:   FPM.addPass(SimplifyCFGPass());
```
- **EN**: Introduces or continues `addPass`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `addPass`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 81-81
```cpp
81:   FPM.addPass(TailCallElimPass());
```
- **EN**: Introduces or continues `addPass`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `addPass`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 82-82
```cpp
82:   FPM.addPass(SimplifyCFGPass());
```
- **EN**: Introduces or continues `addPass`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `addPass`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 83-83
```cpp
83:   FPM.addPass(ReassociatePass());
```
- **EN**: Introduces or continues `addPass`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `addPass`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 84-89
```cpp
84:   {
85:     LoopPassManager LPM;
86:     LPM.addPass(LoopRotatePass());
87:     FPM.addPass(createFunctionToLoopPassAdaptor<LoopPassManager>(
88:         std::move(LPM), /*UseMemorySSA=*/false));
89:   }
```
- **EN**: Introduces or continues `addPass`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `addPass`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 90-93
```cpp
90:   if (PollyInliner) {
91:     MPM.addPass(createModuleToFunctionPassAdaptor(std::move(FPM)));
92:     MPM.addPass(buildInlinePasses(Level));
93:     FPM = FunctionPassManager();
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 95-95
```cpp
95:     FPM.addPass(PromotePass());
```
- **EN**: Introduces or continues `addPass`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `addPass`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 96-96
```cpp
96:     FPM.addPass(SimplifyCFGPass());
```
- **EN**: Introduces or continues `addPass`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `addPass`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 97-97
```cpp
97:     FPM.addPass(InstCombinePass());
```
- **EN**: Introduces or continues `addPass`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `addPass`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 98-98
```cpp
98:   }
```
- **EN**: Closes the current block or scope and hands control back to the surrounding context.
- **CN**: 这里关闭当前代码块或作用域，并将控制流交还给外层上下文。

### Lines 99-99
```cpp
99:   FPM.addPass(InstCombinePass());
```
- **EN**: Introduces or continues `addPass`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `addPass`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 100-105
```cpp
100:   {
101:     LoopPassManager LPM;
102:     LPM.addPass(IndVarSimplifyPass());
103:     FPM.addPass(createFunctionToLoopPassAdaptor<LoopPassManager>(
104:         std::move(LPM), /*UseMemorySSA=*/false));
105:   }
```
- **EN**: Introduces or continues `addPass`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `addPass`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 107-107
```cpp
107:   return FPM;
```
- **EN**: Returns the computed value or forwards the current result to the caller.
- **CN**: 这里返回计算结果，或将当前结果继续传递给调用方。

### Lines 108-108
```cpp
108: }
```
- **EN**: Closes the current block or scope and hands control back to the surrounding context.
- **CN**: 这里关闭当前代码块或作用域，并将控制流交还给外层上下文。

## Key Concepts / 关键概念

- **IR canonicalization** / **IR 规范化**
- **Loop normalization** / **循环规范化**
- **Polly preconditions** / **Polly 前置条件**

## Dependencies / 依赖关系

- **EN**: Polly headers such as polly/Canonicalization.h, polly/Options.h
- **CN**: Polly 头文件，例如 polly/Canonicalization.h, polly/Options.h
- **EN**: LLVM infrastructure headers such as llvm/Analysis/GlobalsModRef.h, llvm/Analysis/ProfileSummaryInfo.h, llvm/IR/LegacyPassManager.h, llvm/Transforms/IPO.h
- **CN**: LLVM 基础设施头文件，例如 llvm/Analysis/GlobalsModRef.h, llvm/Analysis/ProfileSummaryInfo.h, llvm/IR/LegacyPassManager.h, llvm/Transforms/IPO.h
