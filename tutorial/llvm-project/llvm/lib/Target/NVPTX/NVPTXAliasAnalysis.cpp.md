# NVPTXAliasAnalysis.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/NVPTX/NVPTXAliasAnalysis.cpp`
- Repository: `llvm-project`
- Purpose (EN): This is the NVPTX address space based alias analysis pass.
- 目的（中文）: 该文件用于支撑 LLVM 目标后端的相关功能。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-40
```cpp
 1: //===--------------------- NVPTXAliasAnalysis.cpp--------------------------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: /// \file
 9: /// This is the NVPTX address space based alias analysis pass.
10: //===----------------------------------------------------------------------===//
11:
12: #include "NVPTXAliasAnalysis.h"
13: #include "MCTargetDesc/NVPTXBaseInfo.h"
14: #include "NVPTX.h"
15: #include "llvm/Analysis/ValueTracking.h"
16: #include "llvm/IR/InlineAsm.h"
17: #include "llvm/IR/Instructions.h"
18: #include "llvm/Support/CommandLine.h"
19:
20: using namespace llvm;
21:
22: #define DEBUG_TYPE "NVPTX-aa"
23:
24: static cl::opt<unsigned> TraverseAddressSpacesLimit(
25:     "nvptx-traverse-address-aliasing-limit", cl::Hidden,
26:     cl::desc("Depth limit for finding address space through traversal"),
27:     cl::init(6));
28:
29: AnalysisKey NVPTXAA::Key;
30:
31: char NVPTXAAWrapperPass::ID = 0;
32: char NVPTXExternalAAWrapper::ID = 0;
33:
34: INITIALIZE_PASS(NVPTXAAWrapperPass, "nvptx-aa",
35:                 "NVPTX Address space based Alias Analysis", false, true)
36:
37: INITIALIZE_PASS(NVPTXExternalAAWrapper, "nvptx-aa-wrapper",
38:                 "NVPTX Address space based Alias Analysis Wrapper", false, true)
39:
40: ImmutablePass *llvm::createNVPTXAAWrapperPass() {
```
- EN: This range defines command-line tuning knobs that influence backend lowering, code generation, or diagnostics.
- CN: 这一段定义命令行调优选项，用来影响后端的降级、代码生成或诊断行为。

### Lines 41-80
```cpp
41:   return new NVPTXAAWrapperPass();
42: }
43:
44: ImmutablePass *llvm::createNVPTXExternalAAWrapperPass() {
45:   return new NVPTXExternalAAWrapper();
46: }
47:
48: NVPTXAAWrapperPass::NVPTXAAWrapperPass() : ImmutablePass(ID) {}
49:
50: void NVPTXAAWrapperPass::getAnalysisUsage(AnalysisUsage &AU) const {
51:   AU.setPreservesAll();
52: }
53:
54: static unsigned getAddressSpace(const Value *V, unsigned MaxLookup) {
55:   // Find the first non-generic address space traversing the UD chain.
56:   // It is undefined behaviour if a pointer belongs to more than one
57:   // non-overlapping address spaces along a valid execution path.
58:   auto GetAS = [](const Value *V) -> unsigned {
59:     if (const auto *PTy = dyn_cast<PointerType>(V->getType()))
60:       return PTy->getAddressSpace();
61:     return ADDRESS_SPACE_GENERIC;
62:   };
63:   while (MaxLookup-- && GetAS(V) == ADDRESS_SPACE_GENERIC) {
64:     const Value *NewV = getUnderlyingObject(V, 1);
65:     if (NewV == V)
66:       break;
67:     V = NewV;
68:   }
69:   return GetAS(V);
70: }
71:
72: static AliasResult::Kind getAliasResult(unsigned AS1, unsigned AS2) {
73:   if ((AS1 == ADDRESS_SPACE_GENERIC) || (AS2 == ADDRESS_SPACE_GENERIC))
74:     return AliasResult::MayAlias;
75:
76:   // PTX s6.4.1.1. Generic Addressing:
77:   // A generic address maps to global memory unless it falls within
78:   // the window for const, local, or shared memory. The Kernel
79:   // Function Parameters (.param) window is contained within the
80:   // .global window.
```
- EN: This range implements operational logic in helpers such as NVPTXAAWrapperPass, llvm::createNVPTXExternalAAWrapperPass, NVPTXExternalAAWrapper, NVPTXAAWrapperPass::NVPTXAAWrapperPass, translating backend policy into executable code.
- CN: 这一段实现了 NVPTXAAWrapperPass、llvm::createNVPTXExternalAAWrapperPass、NVPTXExternalAAWrapper、NVPTXAAWrapperPass::NVPTXAAWrapperPass 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 81-120
```cpp
 81:   //
 82:   // Therefore a global pointer may alias with a param pointer on some
 83:   // GPUs via addrspacecast(param->generic->global) when cvta.param
 84:   // instruction is used (PTX 7.7+ and SM_70+).
 85:   //
 86:   // TODO: cvta.param is not yet supported. We need to change aliasing
 87:   // rules once it is added.
 88:
 89:   // Distributed shared memory aliases with shared memory.
 90:   if (((AS1 == ADDRESS_SPACE_SHARED) &&
 91:        (AS2 == ADDRESS_SPACE_SHARED_CLUSTER)) ||
 92:       ((AS1 == ADDRESS_SPACE_SHARED_CLUSTER) && (AS2 == ADDRESS_SPACE_SHARED)))
 93:     return AliasResult::MayAlias;
 94:
 95:   return (AS1 == AS2 ? AliasResult::MayAlias : AliasResult::NoAlias);
 96: }
 97:
 98: AliasResult NVPTXAAResult::alias(const MemoryLocation &Loc1,
 99:                                  const MemoryLocation &Loc2, AAQueryInfo &AAQI,
100:                                  const Instruction *) {
101:   unsigned AS1 = getAddressSpace(Loc1.Ptr, TraverseAddressSpacesLimit);
102:   unsigned AS2 = getAddressSpace(Loc2.Ptr, TraverseAddressSpacesLimit);
103:
104:   return getAliasResult(AS1, AS2);
105: }
106:
107: // TODO: .param address space may be writable in presence of cvta.param, but
108: // this instruction is currently not supported. NVPTXLowerArgs also does not
109: // allow any writes to .param pointers.
110: static bool isConstOrParam(unsigned AS) {
111:   return AS == AddressSpace::ADDRESS_SPACE_CONST ||
112:          AS == AddressSpace::ADDRESS_SPACE_ENTRY_PARAM;
113: }
114:
115: ModRefInfo NVPTXAAResult::getModRefInfoMask(const MemoryLocation &Loc,
116:                                             AAQueryInfo &AAQI,
117:                                             bool IgnoreLocals) {
118:   if (isConstOrParam(getAddressSpace(Loc.Ptr, TraverseAddressSpacesLimit)))
119:     return ModRefInfo::NoModRef;
120:
```
- EN: This range implements operational logic in helpers such as getAddressSpace, getAliasResult, isConstOrParam, translating backend policy into executable code.
- CN: 这一段实现了 getAddressSpace、getAliasResult、isConstOrParam 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 121-148
```cpp
121:   return ModRefInfo::ModRef;
122: }
123:
124: MemoryEffects NVPTXAAResult::getMemoryEffects(const CallBase *Call,
125:                                               AAQueryInfo &AAQI) {
126:   // Inline assembly with no side-effect or memory clobbers should not
127:   // indirectly access memory in the PTX specification.
128:   if (const auto *IA = dyn_cast<InlineAsm>(Call->getCalledOperand())) {
129:     // Volatile is translated as side-effects.
130:     if (IA->hasSideEffects())
131:       return MemoryEffects::unknown();
132:
133:     for (const InlineAsm::ConstraintInfo &Constraint : IA->ParseConstraints()) {
134:       // Indirect constraints (e.g. =*m) are unsupported in inline PTX.
135:       if (Constraint.isIndirect)
136:         return MemoryEffects::unknown();
137:
138:       // Memory clobbers prevent optimization.
139:       if ((Constraint.Type & InlineAsm::ConstraintPrefix::isClobber) &&
140:           any_of(Constraint.Codes,
141:                  [](const auto &Code) { return Code == "{memory}"; }))
142:         return MemoryEffects::unknown();
143:     }
144:     return MemoryEffects::none();
145:   }
146:
147:   return MemoryEffects::unknown();
148: }
```
- EN: This range implements operational logic in helpers such as MemoryEffects::unknown, MemoryEffects::none, translating backend policy into executable code.
- CN: 这一段实现了 MemoryEffects::unknown、MemoryEffects::none 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: NVPTX backend code reflects GPU execution concerns such as address spaces, intrinsics, and PTX-specific lowering.
  - CN: NVPTX 后端代码体现了 GPU 执行特性，例如地址空间、内建函数以及 PTX 特定降级。
- EN: Key symbols in this file include cl::init, llvm::createNVPTXAAWrapperPass, NVPTXAAWrapperPass, llvm::createNVPTXExternalAAWrapperPass, NVPTXExternalAAWrapper, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 cl::init, llvm::createNVPTXAAWrapperPass, NVPTXAAWrapperPass, llvm::createNVPTXExternalAAWrapperPass, NVPTXExternalAAWrapper，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `NVPTXAliasAnalysis.h`
  - `MCTargetDesc/NVPTXBaseInfo.h`
  - `NVPTX.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/Analysis/ValueTracking.h`
  - `llvm/IR/InlineAsm.h`
  - `llvm/IR/Instructions.h`
  - `llvm/Support/CommandLine.h`
