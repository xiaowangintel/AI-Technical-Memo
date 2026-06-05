# NVPTXSetByValParamAlign.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/NVPTX/NVPTXSetByValParamAlign.cpp`
- Repository: `llvm-project`
- Purpose (EN): Set explicit alignment on byval parameter attributes in the NVPTX backend.
- 目的（中文）: 该文件用于支撑 LLVM 目标后端的相关功能。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-40
```cpp
 1: //===-- NVPTXSetByValParamAlign.cpp - Set byval param alignment -----------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // Set explicit alignment on byval parameter attributes in the NVPTX backend.
10: // Without this, the alignment is left unspecified and IR-level analyses (e.g.,
11: // computeKnownBits via Value::getPointerAlignment) conservatively assume
12: // Align(1), since the actual alignment is a target-specific codegen detail not
13: // visible at the IR level.
14: //
15: // The alignment is chosen as follows:
16: //   - Externally-visible functions: ABI type alignment (capped at 128).
17: //   - Internal/private functions: max(16, ABI align) to enable 128-bit
18: //     vectorized param loads. The compiler can _increase_ alignment beyond ABI
19: //     in this case because it has control over all of the call sites and byval
20: //     parameters are copies allocated by the caller in .param space.
21: //
22: // After updating the attribute, the pass propagates the improved alignment to
23: // all loads from the byval pointer that use a known constant offset.
24: //
25: // TODO: Consider removing the load propagation in favor of infer-alignment,
26: // which should be able to pick up the improved alignment from the attribute.
27: //
28: //===----------------------------------------------------------------------===//
29:
30: #include "NVPTX.h"
31: #include "NVPTXUtilities.h"
32: #include "llvm/IR/Function.h"
33: #include "llvm/IR/Instructions.h"
34: #include "llvm/InitializePasses.h"
35: #include "llvm/Pass.h"
36: #include "llvm/Support/Debug.h"
37: #include <queue>
38:
39: #define DEBUG_TYPE "nvptx-set-byval-param-align"
40:
```
- EN: This range contains connective backend logic, including helper statements, control flow, and data movement needed by the surrounding implementation.
- CN: 这一段包含衔接性的后端逻辑，包括辅助语句、控制流和数据处理，用于支撑周围实现。

### Lines 41-80
```cpp
41: using namespace llvm;
42:
43: namespace {
44: class NVPTXSetByValParamAlignLegacyPass : public FunctionPass {
45:   bool runOnFunction(Function &F) override;
46:
47: public:
48:   static char ID;
49:   NVPTXSetByValParamAlignLegacyPass() : FunctionPass(ID) {}
50:   StringRef getPassName() const override {
51:     return "Set alignment of byval parameters (NVPTX)";
52:   }
53: };
54: } // namespace
55:
56: char NVPTXSetByValParamAlignLegacyPass::ID = 0;
57:
58: INITIALIZE_PASS(NVPTXSetByValParamAlignLegacyPass,
59:                 "nvptx-set-byval-param-align",
60:                 "Set alignment of byval parameters (NVPTX)", false, false)
61:
62: static Align setByValParamAlign(Argument *Arg) {
63:   Function *F = Arg->getParent();
64:   Type *ByValType = Arg->getParamByValType();
65:   const DataLayout &DL = F->getDataLayout();
66:
67:   const Align OptimizedAlign = getFunctionParamOptimizedAlign(F, ByValType, DL);
68:   const Align CurrentAlign = Arg->getParamAlign().valueOrOne();
69:
70:   if (CurrentAlign >= OptimizedAlign)
71:     return CurrentAlign;
72:
73:   LLVM_DEBUG(dbgs() << "Try to use alignment " << OptimizedAlign.value()
74:                     << " instead of " << CurrentAlign.value() << " for " << *Arg
75:                     << '\n');
76:
77:   Arg->removeAttr(Attribute::Alignment);
78:   Arg->addAttr(Attribute::getWithAlignment(F->getContext(), OptimizedAlign));
79:
80:   return OptimizedAlign;
```
- EN: This range defines or declares important types such as NVPTXSetByValParamAlignLegacyPass, runOnFunction, getPassName, parameters, shaping the data model used by NVPTXSetByValParamAlign.cpp.
- CN: 这一段定义或声明了 NVPTXSetByValParamAlignLegacyPass、runOnFunction、getPassName、parameters 等关键类型，构成 NVPTXSetByValParamAlign.cpp 使用的数据模型。

### Lines 81-120
```cpp
 81: }
 82:
 83: // Adjust alignment of arguments passed byval in .param address space. We can
 84: // increase alignment of such arguments in a way that ensures that we can
 85: // effectively vectorize their loads. We should also traverse all loads from
 86: // byval pointer and adjust their alignment, if those were using known offset.
 87: // Such alignment changes must be conformed with parameter store and load in
 88: // NVPTXTargetLowering::LowerCall.
 89: static void propagateAlignmentToLoads(Value *Val, Align NewAlign,
 90:                                       const DataLayout &DL) {
 91:   struct Load {
 92:     LoadInst *Inst;
 93:     uint64_t Offset;
 94:   };
 95:
 96:   struct LoadContext {
 97:     Value *InitialVal;
 98:     uint64_t Offset;
 99:   };
100:
101:   SmallVector<Load> Loads;
102:   std::queue<LoadContext> Worklist;
103:   Worklist.push({Val, 0});
104:
105:   while (!Worklist.empty()) {
106:     LoadContext Ctx = Worklist.front();
107:     Worklist.pop();
108:
109:     for (User *CurUser : Ctx.InitialVal->users()) {
110:       if (auto *I = dyn_cast<LoadInst>(CurUser))
111:         Loads.push_back({I, Ctx.Offset});
112:       else if (isa<BitCastInst>(CurUser) || isa<AddrSpaceCastInst>(CurUser))
113:         Worklist.push({cast<Instruction>(CurUser), Ctx.Offset});
114:       else if (auto *I = dyn_cast<GetElementPtrInst>(CurUser)) {
115:         APInt OffsetAccumulated =
116:             APInt::getZero(DL.getIndexTypeSizeInBits(I->getType()));
117:
118:         if (!I->accumulateConstantOffset(DL, OffsetAccumulated))
119:           continue;
120:
```
- EN: This range defines or declares important types such as Load, LoadContext, push, front, shaping the data model used by NVPTXSetByValParamAlign.cpp.
- CN: 这一段定义或声明了 Load、LoadContext、push、front 等关键类型，构成 NVPTXSetByValParamAlign.cpp 使用的数据模型。

### Lines 121-160
```cpp
121:         uint64_t OffsetLimit = -1;
122:         uint64_t Offset = OffsetAccumulated.getLimitedValue(OffsetLimit);
123:         assert(Offset != OffsetLimit && "Expect Offset less than UINT64_MAX");
124:
125:         Worklist.push({I, Ctx.Offset + Offset});
126:       }
127:     }
128:   }
129:
130:   for (Load &CurLoad : Loads) {
131:     Align NewLoadAlign = commonAlignment(NewAlign, CurLoad.Offset);
132:     Align CurLoadAlign = CurLoad.Inst->getAlign();
133:     CurLoad.Inst->setAlignment(std::max(NewLoadAlign, CurLoadAlign));
134:   }
135: }
136:
137: static bool setByValParamAlignment(Function &F) {
138:   const DataLayout &DL = F.getDataLayout();
139:   bool Changed = false;
140:   for (Argument &Arg : F.args()) {
141:     if (!Arg.hasByValAttr())
142:       continue;
143:     const Align NewArgAlign = setByValParamAlign(&Arg);
144:     propagateAlignmentToLoads(&Arg, NewArgAlign, DL);
145:     Changed = true;
146:   }
147:   return Changed;
148: }
149:
150: bool NVPTXSetByValParamAlignLegacyPass::runOnFunction(Function &F) {
151:   return setByValParamAlignment(F);
152: }
153:
154: FunctionPass *llvm::createNVPTXSetByValParamAlignPass() {
155:   return new NVPTXSetByValParamAlignLegacyPass();
156: }
157:
158: PreservedAnalyses
159: NVPTXSetByValParamAlignPass::run(Function &F, FunctionAnalysisManager &AM) {
160:   return setByValParamAlignment(F) ? PreservedAnalyses::none()
```
- EN: This range implements operational logic in helpers such as getLimitedValue, assert, push, commonAlignment, translating backend policy into executable code.
- CN: 这一段实现了 getLimitedValue、assert、push、commonAlignment 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 161-162
```cpp
161:                                    : PreservedAnalyses::all();
162: }
```
- EN: This range implements operational logic in helpers such as PreservedAnalyses::all, translating backend policy into executable code.
- CN: 这一段实现了 PreservedAnalyses::all 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: NVPTX backend code reflects GPU execution concerns such as address spaces, intrinsics, and PTX-specific lowering.
  - CN: NVPTX 后端代码体现了 GPU 执行特性，例如地址空间、内建函数以及 PTX 特定降级。
- EN: Key symbols in this file include NVPTXSetByValParamAlignLegacyPass, runOnFunction, getPassName, parameters, setByValParamAlign, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 NVPTXSetByValParamAlignLegacyPass, runOnFunction, getPassName, parameters, setByValParamAlign，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `NVPTX.h`
  - `NVPTXUtilities.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/IR/Function.h`
  - `llvm/IR/Instructions.h`
  - `llvm/InitializePasses.h`
  - `llvm/Pass.h`
  - `llvm/Support/Debug.h`
- System/standard headers / 系统或标准头文件:
  - `queue`
