# NVPTXLowerUnreachable.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/NVPTX/NVPTXLowerUnreachable.cpp`
- Repository: `llvm-project`
- Purpose (EN): PTX does not have a notion of `unreachable`, which results in emitted basic blocks having an edge to the next block:.
- 目的（中文）: 该文件用于支撑 LLVM 目标后端的相关功能。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-40
```cpp
 1: //===-- NVPTXLowerUnreachable.cpp - Lower unreachables to exit =====--===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // PTX does not have a notion of `unreachable`, which results in emitted basic
10: // blocks having an edge to the next block:
11: //
12: //   block1:
13: //     call @does_not_return();
14: //     // unreachable
15: //   block2:
16: //     // ptxas will create a CFG edge from block1 to block2
17: //
18: // This may result in significant changes to the control flow graph, e.g., when
19: // LLVM moves unreachable blocks to the end of the function. That's a problem
20: // in the context of divergent control flow, as `ptxas` uses the CFG to
21: // determine divergent regions, and some intructions may not be executed
22: // divergently.
23: //
24: // For example, `bar.sync` is not allowed to be executed divergently on Pascal
25: // or earlier. If we start with the following:
26: //
27: //   entry:
28: //     // start of divergent region
29: //     @%p0 bra cont;
30: //     @%p1 bra unlikely;
31: //     ...
32: //     bra.uni cont;
33: //   unlikely:
34: //     ...
35: //     // unreachable
36: //   cont:
37: //     // end of divergent region
38: //     bar.sync 0;
39: //     bra.uni exit;
40: //   exit:
```
- EN: This range contains banner comments or file-level notes that explain the role of the file and establish context for the backend component.
- CN: 这一段包含文件头注释或说明文字，用来交代该后端组件的职责与上下文。

### Lines 41-80
```cpp
41: //     ret;
42: //
43: // it is transformed by the branch-folder and block-placement passes to:
44: //
45: //   entry:
46: //     // start of divergent region
47: //     @%p0 bra cont;
48: //     @%p1 bra unlikely;
49: //     ...
50: //     bra.uni cont;
51: //   cont:
52: //     bar.sync 0;
53: //     bra.uni exit;
54: //   unlikely:
55: //     ...
56: //     // unreachable
57: //   exit:
58: //     // end of divergent region
59: //     ret;
60: //
61: // After moving the `unlikely` block to the end of the function, it has an edge
62: // to the `exit` block, which widens the divergent region and makes the
63: // `bar.sync` instruction happen divergently.
64: //
65: // To work around this, we add an `exit` instruction before every `unreachable`,
66: // as `ptxas` understands that exit terminates the CFG. We do only do this if
67: // `unreachable` is not lowered to `trap`, which has the same effect (although
68: // with current versions of `ptxas` only because it is emited as `trap; exit;`).
69: //
70: //===----------------------------------------------------------------------===//
71:
72: #include "NVPTX.h"
73: #include "llvm/IR/Function.h"
74: #include "llvm/IR/InlineAsm.h"
75: #include "llvm/IR/Instructions.h"
76: #include "llvm/IR/Type.h"
77: #include "llvm/Pass.h"
78:
79: using namespace llvm;
80:
```
- EN: This range contains connective backend logic, including helper statements, control flow, and data movement needed by the surrounding implementation.
- CN: 这一段包含衔接性的后端逻辑，包括辅助语句、控制流和数据处理，用于支撑周围实现。

### Lines 81-120
```cpp
 81: namespace {
 82: class NVPTXLowerUnreachable : public FunctionPass {
 83:   StringRef getPassName() const override;
 84:   bool runOnFunction(Function &F) override;
 85:   bool isLoweredToTrap(const UnreachableInst &I) const;
 86:
 87: public:
 88:   static char ID; // Pass identification, replacement for typeid
 89:   NVPTXLowerUnreachable(bool TrapUnreachable, bool NoTrapAfterNoreturn)
 90:       : FunctionPass(ID), TrapUnreachable(TrapUnreachable),
 91:         NoTrapAfterNoreturn(NoTrapAfterNoreturn) {}
 92:
 93: private:
 94:   bool TrapUnreachable;
 95:   bool NoTrapAfterNoreturn;
 96: };
 97: } // namespace
 98:
 99: char NVPTXLowerUnreachable::ID = 1;
100:
101: INITIALIZE_PASS(NVPTXLowerUnreachable, "nvptx-lower-unreachable",
102:                 "Lower Unreachable", false, false)
103:
104: StringRef NVPTXLowerUnreachable::getPassName() const {
105:   return "add an exit instruction before every unreachable";
106: }
107:
108: // =============================================================================
109: // Returns whether a `trap` intrinsic would be emitted before I.
110: //
111: // This is a copy of the logic in SelectionDAGBuilder::visitUnreachable().
112: // =============================================================================
113: bool NVPTXLowerUnreachable::isLoweredToTrap(const UnreachableInst &I) const {
114:   if (const auto *Call = dyn_cast_or_null<CallInst>(I.getPrevNode())) {
115:     // We've already emitted a non-continuable trap.
116:     if (Call->isNonContinuableTrap())
117:       return true;
118:
119:     // No traps are emitted for calls that do not return
120:     // when this option is enabled.
```
- EN: This range defines or declares important types such as NVPTXLowerUnreachable, getPassName, runOnFunction, isLoweredToTrap, shaping the data model used by NVPTXLowerUnreachable.cpp.
- CN: 这一段定义或声明了 NVPTXLowerUnreachable、getPassName、runOnFunction、isLoweredToTrap 等关键类型，构成 NVPTXLowerUnreachable.cpp 使用的数据模型。

### Lines 121-159
```cpp
121:     if (NoTrapAfterNoreturn && Call->doesNotReturn())
122:       return false;
123:   }
124:
125:   // In all other cases, we will generate a trap if TrapUnreachable is set.
126:   return TrapUnreachable;
127: }
128:
129: // =============================================================================
130: // Main function for this pass.
131: // =============================================================================
132: bool NVPTXLowerUnreachable::runOnFunction(Function &F) {
133:   if (skipFunction(F))
134:     return false;
135:   // Early out iff isLoweredToTrap() always returns true.
136:   if (TrapUnreachable && !NoTrapAfterNoreturn)
137:     return false;
138:
139:   LLVMContext &C = F.getContext();
140:   FunctionType *ExitFTy = FunctionType::get(Type::getVoidTy(C), false);
141:   InlineAsm *Exit = InlineAsm::get(ExitFTy, "exit;", "", true);
142:
143:   bool Changed = false;
144:   for (auto &BB : F)
145:     for (auto &I : BB) {
146:       if (auto unreachableInst = dyn_cast<UnreachableInst>(&I)) {
147:         if (isLoweredToTrap(*unreachableInst))
148:           continue; // trap is emitted as `trap; exit;`.
149:         CallInst::Create(ExitFTy, Exit, "", unreachableInst->getIterator());
150:         Changed = true;
151:       }
152:     }
153:   return Changed;
154: }
155:
156: FunctionPass *llvm::createNVPTXLowerUnreachablePass(bool TrapUnreachable,
157:                                                     bool NoTrapAfterNoreturn) {
158:   return new NVPTXLowerUnreachable(TrapUnreachable, NoTrapAfterNoreturn);
159: }
```
- EN: This range implements operational logic in helpers such as NVPTXLowerUnreachable::runOnFunction, getContext, FunctionType::get, CallInst::Create, translating backend policy into executable code.
- CN: 这一段实现了 NVPTXLowerUnreachable::runOnFunction、getContext、FunctionType::get、CallInst::Create 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: Lowering turns higher-level IR constructs into forms accepted by the target pipeline.
  - CN: 降级过程会把更高层的 IR 构造转换成目标流水线可接受的形式。
- EN: NVPTX backend code reflects GPU execution concerns such as address spaces, intrinsics, and PTX-specific lowering.
  - CN: NVPTX 后端代码体现了 GPU 执行特性，例如地址空间、内建函数以及 PTX 特定降级。
- EN: Key symbols in this file include NVPTXLowerUnreachable, getPassName, runOnFunction, isLoweredToTrap, NoTrapAfterNoreturn, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 NVPTXLowerUnreachable, getPassName, runOnFunction, isLoweredToTrap, NoTrapAfterNoreturn，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `NVPTX.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/IR/Function.h`
  - `llvm/IR/InlineAsm.h`
  - `llvm/IR/Instructions.h`
  - `llvm/IR/Type.h`
  - `llvm/Pass.h`
