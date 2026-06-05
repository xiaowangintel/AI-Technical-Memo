# SPIRVPreLegalizerCombiner.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/SPIRV/SPIRVPreLegalizerCombiner.cpp`
- Repository: `llvm-project`
- Purpose (EN): This pass does combining of machine instructions at the generic MI level, before the legalizer.
- 目的（中文）: 该文件用于支撑 LLVM 目标后端的相关功能。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-40
```cpp
 1: //===-- SPIRVPreLegalizerCombiner.cpp - combine legalization ----*- C++ -*-===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // This pass does combining of machine instructions at the generic MI level,
10: // before the legalizer.
11: //
12: //===----------------------------------------------------------------------===//
13:
14: #include "SPIRV.h"
15: #include "SPIRVCombinerHelper.h"
16: #include "llvm/CodeGen/GlobalISel/CSEInfo.h"
17: #include "llvm/CodeGen/GlobalISel/Combiner.h"
18: #include "llvm/CodeGen/GlobalISel/CombinerInfo.h"
19: #include "llvm/CodeGen/GlobalISel/GIMatchTableExecutorImpl.h"
20: #include "llvm/CodeGen/GlobalISel/GISelChangeObserver.h"
21: #include "llvm/CodeGen/GlobalISel/GISelValueTracking.h"
22: #include "llvm/CodeGen/GlobalISel/MIPatternMatch.h"
23: #include "llvm/CodeGen/MachineDominators.h"
24: #include "llvm/CodeGen/MachineFunctionPass.h"
25: #include "llvm/CodeGen/TargetPassConfig.h"
26:
27: #define GET_GICOMBINER_DEPS
28: #include "SPIRVGenPreLegalizeGICombiner.inc"
29: #undef GET_GICOMBINER_DEPS
30:
31: #define DEBUG_TYPE "spirv-prelegalizer-combiner"
32:
33: using namespace llvm;
34: using namespace MIPatternMatch;
35:
36: namespace {
37:
38: #define GET_GICOMBINER_TYPES
39: #include "SPIRVGenPreLegalizeGICombiner.inc"
40: #undef GET_GICOMBINER_TYPES
```
- EN: This range contains connective backend logic, including helper statements, control flow, and data movement needed by the surrounding implementation.
- CN: 这一段包含衔接性的后端逻辑，包括辅助语句、控制流和数据处理，用于支撑周围实现。

### Lines 41-80
```cpp
41:
42: class SPIRVPreLegalizerCombinerImpl : public Combiner {
43: protected:
44:   const SPIRVCombinerHelper Helper;
45:   const SPIRVPreLegalizerCombinerImplRuleConfig &RuleConfig;
46:   const SPIRVSubtarget &STI;
47:
48: public:
49:   SPIRVPreLegalizerCombinerImpl(
50:       MachineFunction &MF, CombinerInfo &CInfo, GISelValueTracking &VT,
51:       GISelCSEInfo *CSEInfo,
52:       const SPIRVPreLegalizerCombinerImplRuleConfig &RuleConfig,
53:       const SPIRVSubtarget &STI, MachineDominatorTree *MDT,
54:       const LegalizerInfo *LI);
55:
56:   static const char *getName() { return "SPIRVPreLegalizerCombiner"; }
57:
58:   bool tryCombineAll(MachineInstr &I) const override;
59:
60:   bool tryCombineAllImpl(MachineInstr &I) const;
61:
62: private:
63: #define GET_GICOMBINER_CLASS_MEMBERS
64: #include "SPIRVGenPreLegalizeGICombiner.inc"
65: #undef GET_GICOMBINER_CLASS_MEMBERS
66: };
67:
68: #define GET_GICOMBINER_IMPL
69: #include "SPIRVGenPreLegalizeGICombiner.inc"
70: #undef GET_GICOMBINER_IMPL
71:
72: SPIRVPreLegalizerCombinerImpl::SPIRVPreLegalizerCombinerImpl(
73:     MachineFunction &MF, CombinerInfo &CInfo, GISelValueTracking &VT,
74:     GISelCSEInfo *CSEInfo,
75:     const SPIRVPreLegalizerCombinerImplRuleConfig &RuleConfig,
76:     const SPIRVSubtarget &STI, MachineDominatorTree *MDT,
77:     const LegalizerInfo *LI)
78:     : Combiner(MF, CInfo, &VT, CSEInfo),
79:       Helper(Observer, B, /*IsPreLegalize*/ true, &VT, MDT, LI, STI),
80:       RuleConfig(RuleConfig), STI(STI),
```
- EN: This range defines or declares important types such as SPIRVPreLegalizerCombinerImpl, getName, tryCombineAll, tryCombineAllImpl, shaping the data model used by SPIRVPreLegalizerCombiner.cpp.
- CN: 这一段定义或声明了 SPIRVPreLegalizerCombinerImpl、getName、tryCombineAll、tryCombineAllImpl 等关键类型，构成 SPIRVPreLegalizerCombiner.cpp 使用的数据模型。

### Lines 81-120
```cpp
 81: #define GET_GICOMBINER_CONSTRUCTOR_INITS
 82: #include "SPIRVGenPreLegalizeGICombiner.inc"
 83: #undef GET_GICOMBINER_CONSTRUCTOR_INITS
 84: {
 85: }
 86:
 87: bool SPIRVPreLegalizerCombinerImpl::tryCombineAll(MachineInstr &MI) const {
 88:   return tryCombineAllImpl(MI);
 89: }
 90:
 91: // Pass boilerplate
 92: // ================
 93:
 94: class SPIRVPreLegalizerCombiner : public MachineFunctionPass {
 95: public:
 96:   static char ID;
 97:
 98:   SPIRVPreLegalizerCombiner();
 99:
100:   StringRef getPassName() const override { return "SPIRVPreLegalizerCombiner"; }
101:
102:   bool runOnMachineFunction(MachineFunction &MF) override;
103:
104:   void getAnalysisUsage(AnalysisUsage &AU) const override;
105:
106: private:
107:   SPIRVPreLegalizerCombinerImplRuleConfig RuleConfig;
108: };
109:
110: } // end anonymous namespace
111:
112: void SPIRVPreLegalizerCombiner::getAnalysisUsage(AnalysisUsage &AU) const {
113:   AU.setPreservesCFG();
114:   getSelectionDAGFallbackAnalysisUsage(AU);
115:   AU.addRequired<GISelValueTrackingAnalysisLegacy>();
116:   AU.addPreserved<GISelValueTrackingAnalysisLegacy>();
117:   AU.addRequired<MachineDominatorTreeWrapperPass>();
118:   AU.addPreserved<MachineDominatorTreeWrapperPass>();
119:   MachineFunctionPass::getAnalysisUsage(AU);
120: }
```
- EN: This range defines or declares important types such as SPIRVPreLegalizerCombinerImpl::tryCombineAll, tryCombineAllImpl, SPIRVPreLegalizerCombiner, getPassName, shaping the data model used by SPIRVPreLegalizerCombiner.cpp.
- CN: 这一段定义或声明了 SPIRVPreLegalizerCombinerImpl::tryCombineAll、tryCombineAllImpl、SPIRVPreLegalizerCombiner、getPassName 等关键类型，构成 SPIRVPreLegalizerCombiner.cpp 使用的数据模型。

### Lines 121-160
```cpp
121:
122: SPIRVPreLegalizerCombiner::SPIRVPreLegalizerCombiner()
123:     : MachineFunctionPass(ID) {
124:   if (!RuleConfig.parseCommandLineOption())
125:     report_fatal_error("Invalid rule identifier");
126: }
127:
128: bool SPIRVPreLegalizerCombiner::runOnMachineFunction(MachineFunction &MF) {
129:   if (MF.getProperties().hasFailedISel())
130:     return false;
131:
132:   const SPIRVSubtarget &ST = MF.getSubtarget<SPIRVSubtarget>();
133:   const auto *LI = ST.getLegalizerInfo();
134:
135:   const Function &F = MF.getFunction();
136:   bool EnableOpt =
137:       MF.getTarget().getOptLevel() != CodeGenOptLevel::None && !skipFunction(F);
138:   GISelValueTracking *VT =
139:       &getAnalysis<GISelValueTrackingAnalysisLegacy>().get(MF);
140:   MachineDominatorTree *MDT =
141:       &getAnalysis<MachineDominatorTreeWrapperPass>().getDomTree();
142:   CombinerInfo CInfo(/*AllowIllegalOps*/ true, /*ShouldLegalizeIllegal*/ false,
143:                      /*LegalizerInfo*/ nullptr, EnableOpt, F.hasOptSize(),
144:                      F.hasMinSize());
145:   // Disable fixed-point iteration to reduce compile-time
146:   CInfo.MaxIterations = 1;
147:   CInfo.ObserverLvl = CombinerInfo::ObserverLevel::SinglePass;
148:   // This is the first Combiner, so the input IR might contain dead
149:   // instructions.
150:   CInfo.EnableFullDCE = false;
151:   SPIRVPreLegalizerCombinerImpl Impl(MF, CInfo, *VT, /*CSEInfo*/ nullptr,
152:                                      RuleConfig, ST, MDT, LI);
153:   return Impl.combineMachineInstrs();
154: }
155:
156: char SPIRVPreLegalizerCombiner::ID = 0;
157: INITIALIZE_PASS_BEGIN(SPIRVPreLegalizerCombiner, DEBUG_TYPE,
158:                       "Combine SPIRV machine instrs before legalization", false,
159:                       false)
160: INITIALIZE_PASS_DEPENDENCY(GISelValueTrackingAnalysisLegacy)
```
- EN: This range implements operational logic in helpers such as SPIRVPreLegalizerCombiner::SPIRVPreLegalizerCombiner, MachineFunctionPass, report_fatal_error, SPIRVPreLegalizerCombiner::runOnMachineFunction, translating backend policy into executable code.
- CN: 这一段实现了 SPIRVPreLegalizerCombiner::SPIRVPreLegalizerCombiner、MachineFunctionPass、report_fatal_error、SPIRVPreLegalizerCombiner::runOnMachineFunction 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 161-169
```cpp
161: INITIALIZE_PASS_END(SPIRVPreLegalizerCombiner, DEBUG_TYPE,
162:                     "Combine SPIRV machine instrs before legalization", false,
163:                     false)
164:
165: namespace llvm {
166: FunctionPass *createSPIRVPreLegalizerCombiner() {
167:   return new SPIRVPreLegalizerCombiner();
168: }
169: } // end namespace llvm
```
- EN: This range implements operational logic in helpers such as createSPIRVPreLegalizerCombiner, SPIRVPreLegalizerCombiner, translating backend policy into executable code.
- CN: 这一段实现了 createSPIRVPreLegalizerCombiner、SPIRVPreLegalizerCombiner 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: Legalization logic rewrites unsupported operations into forms that the target can handle.
  - CN: 合法化逻辑会把目标不支持的操作重写为可处理的形式。
- EN: SPIR-V backend code must preserve rich type information and encode capabilities/extensions explicitly.
  - CN: SPIR-V 后端需要保留丰富的类型信息，并显式编码 capability 与 extension。
- EN: Key symbols in this file include SPIRVPreLegalizerCombinerImpl, getName, tryCombineAll, tryCombineAllImpl, SPIRVPreLegalizerCombinerImpl::tryCombineAll, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 SPIRVPreLegalizerCombinerImpl, getName, tryCombineAll, tryCombineAllImpl, SPIRVPreLegalizerCombinerImpl::tryCombineAll，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `SPIRV.h`
  - `SPIRVCombinerHelper.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/CodeGen/GlobalISel/CSEInfo.h`
  - `llvm/CodeGen/GlobalISel/Combiner.h`
  - `llvm/CodeGen/GlobalISel/CombinerInfo.h`
  - `llvm/CodeGen/GlobalISel/GIMatchTableExecutorImpl.h`
  - `llvm/CodeGen/GlobalISel/GISelChangeObserver.h`
  - `llvm/CodeGen/GlobalISel/GISelValueTracking.h`
  - `llvm/CodeGen/GlobalISel/MIPatternMatch.h`
  - `llvm/CodeGen/MachineDominators.h`
  - `llvm/CodeGen/MachineFunctionPass.h`
  - `llvm/CodeGen/TargetPassConfig.h`
- System/standard headers / 系统或标准头文件:
  - `SPIRVGenPreLegalizeGICombiner.inc`
  - `SPIRVGenPreLegalizeGICombiner.inc`
  - `SPIRVGenPreLegalizeGICombiner.inc`
  - `SPIRVGenPreLegalizeGICombiner.inc`
  - `SPIRVGenPreLegalizeGICombiner.inc`
