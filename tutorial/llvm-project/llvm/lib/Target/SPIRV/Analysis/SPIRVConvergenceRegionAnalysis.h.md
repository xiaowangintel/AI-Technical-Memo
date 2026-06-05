# SPIRVConvergenceRegionAnalysis.h — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/SPIRV/Analysis/SPIRVConvergenceRegionAnalysis.h`
- Repository: `llvm-project`
- Purpose (EN): The analysis determines the convergence region for each basic block of the module, and provides a tree-like structure describing the region hierarchy.
- 目的（中文）: 该文件声明 LLVM 目标后端使用的数据结构、接口或辅助类型。
- Language: C++ header

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-40
```cpp
 1: //===- SPIRVConvergenceRegionAnalysis.h ------------------------*- C++ -*--===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // The analysis determines the convergence region for each basic block of
10: // the module, and provides a tree-like structure describing the region
11: // hierarchy.
12: //
13: //===----------------------------------------------------------------------===//
14:
15: #ifndef LLVM_LIB_TARGET_SPIRV_ANALYSIS_SPIRVCONVERGENCEREGIONANALYSIS_H
16: #define LLVM_LIB_TARGET_SPIRV_ANALYSIS_SPIRVCONVERGENCEREGIONANALYSIS_H
17:
18: #include "llvm/ADT/SmallPtrSet.h"
19: #include "llvm/Analysis/CFG.h"
20: #include "llvm/Analysis/LoopInfo.h"
21: #include "llvm/IR/Dominators.h"
22: #include <optional>
23:
24: namespace llvm {
25: class IntrinsicInst;
26: class SPIRVSubtarget;
27: class MachineFunction;
28: class MachineModuleInfo;
29:
30: namespace SPIRV {
31:
32: // Returns the first convergence intrinsic found in |BB|, |nullopt| otherwise.
33: std::optional<IntrinsicInst *> getConvergenceToken(BasicBlock *BB);
34: std::optional<const IntrinsicInst *> getConvergenceToken(const BasicBlock *BB);
35:
36: // Describes a hierarchy of convergence regions.
37: // A convergence region defines a CFG for which the execution flow can diverge
38: // starting from the entry block, but should reconverge back before the end of
39: // the exit blocks.
40: class ConvergenceRegion {
```
- EN: This range defines or declares important types such as IntrinsicInst, SPIRVSubtarget, MachineFunction, MachineModuleInfo, shaping the data model used by SPIRVConvergenceRegionAnalysis.h.
- CN: 这一段定义或声明了 IntrinsicInst、SPIRVSubtarget、MachineFunction、MachineModuleInfo 等关键类型，构成 SPIRVConvergenceRegionAnalysis.h 使用的数据模型。

### Lines 41-80
```cpp
41:   DominatorTree &DT;
42:   LoopInfo &LI;
43:
44: public:
45:   // The parent region of this region, if any.
46:   ConvergenceRegion *Parent = nullptr;
47:   // The sub-regions contained in this region, if any.
48:   SmallVector<ConvergenceRegion *> Children = {};
49:   // The convergence instruction linked to this region, if any.
50:   std::optional<IntrinsicInst *> ConvergenceToken = std::nullopt;
51:   // The only block with a predecessor outside of this region.
52:   BasicBlock *Entry = nullptr;
53:   // All the blocks with an edge leaving this convergence region.
54:   SmallPtrSet<BasicBlock *, 2> Exits = {};
55:   // All the blocks that belongs to this region, including its subregions'.
56:   SmallPtrSet<BasicBlock *, 8> Blocks = {};
57:
58:   // Creates a single convergence region encapsulating the whole function |F|.
59:   ConvergenceRegion(DominatorTree &DT, LoopInfo &LI, Function &F);
60:
61:   // Creates a single convergence region defined by entry and exits nodes, a
62:   // list of blocks, and possibly a convergence token.
63:   ConvergenceRegion(DominatorTree &DT, LoopInfo &LI,
64:                     std::optional<IntrinsicInst *> ConvergenceToken,
65:                     BasicBlock *Entry, SmallPtrSet<BasicBlock *, 8> &&Blocks,
66:                     SmallPtrSet<BasicBlock *, 2> &&Exits);
67:
68:   ConvergenceRegion(ConvergenceRegion &&CR)
69:       : DT(CR.DT), LI(CR.LI), Parent(std::move(CR.Parent)),
70:         Children(std::move(CR.Children)),
71:         ConvergenceToken(std::move(CR.ConvergenceToken)),
72:         Entry(std::move(CR.Entry)), Exits(std::move(CR.Exits)),
73:         Blocks(std::move(CR.Blocks)) {}
74:
75:   ~ConvergenceRegion() { releaseMemory(); }
76:
77:   ConvergenceRegion &operator=(ConvergenceRegion &&CR) = delete;
78:   ConvergenceRegion(const ConvergenceRegion &other) = delete;
79:   ConvergenceRegion &operator=(const ConvergenceRegion &other) = delete;
80:
```
- EN: This range declares interfaces or inline helpers such as ConvergenceRegion, Blocks, ~ConvergenceRegion, defining how other backend pieces interact with this header.
- CN: 这一段声明了 ConvergenceRegion、Blocks、~ConvergenceRegion 等接口或内联辅助函数，定义其他后端组件如何与该头文件协作。

### Lines 81-120
```cpp
 81:   // Returns true if the given basic block belongs to this region, or to one of
 82:   // its subregion.
 83:   bool contains(const BasicBlock *BB) const { return Blocks.count(BB) != 0; }
 84:
 85:   void releaseMemory();
 86:
 87:   // Write to the debug output this region's hierarchy.
 88:   // |IndentSize| defines the number of tabs to print before any new line.
 89:   void dump(const unsigned IndentSize = 0) const;
 90: };
 91:
 92: // Holds a ConvergenceRegion hierarchy.
 93: class ConvergenceRegionInfo {
 94:   // The convergence region this structure holds.
 95:   ConvergenceRegion *TopLevelRegion;
 96:
 97: public:
 98:   ConvergenceRegionInfo() : TopLevelRegion(nullptr) {}
 99:
100:   // Creates a new ConvergenceRegionInfo. Ownership of the TopLevelRegion is
101:   // passed to this object.
102:   ConvergenceRegionInfo(ConvergenceRegion *TopLevelRegion)
103:       : TopLevelRegion(TopLevelRegion) {}
104:
105:   ~ConvergenceRegionInfo() { releaseMemory(); }
106:
107:   ConvergenceRegionInfo(const ConvergenceRegionInfo &LHS) = delete;
108:   ConvergenceRegionInfo &operator=(const ConvergenceRegionInfo &LHS) = delete;
109:
110:   ConvergenceRegionInfo(ConvergenceRegionInfo &&LHS)
111:       : TopLevelRegion(LHS.TopLevelRegion) {
112:     if (TopLevelRegion != LHS.TopLevelRegion) {
113:       releaseMemory();
114:       TopLevelRegion = LHS.TopLevelRegion;
115:     }
116:     LHS.TopLevelRegion = nullptr;
117:   }
118:
119:   ConvergenceRegionInfo &operator=(ConvergenceRegionInfo &&LHS) {
120:     if (TopLevelRegion != LHS.TopLevelRegion) {
```
- EN: This range defines or declares important types such as contains, releaseMemory, dump, ConvergenceRegionInfo, shaping the data model used by SPIRVConvergenceRegionAnalysis.h.
- CN: 这一段定义或声明了 contains、releaseMemory、dump、ConvergenceRegionInfo 等关键类型，构成 SPIRVConvergenceRegionAnalysis.h 使用的数据模型。

### Lines 121-160
```cpp
121:       releaseMemory();
122:       TopLevelRegion = LHS.TopLevelRegion;
123:     }
124:     LHS.TopLevelRegion = nullptr;
125:     return *this;
126:   }
127:
128:   void releaseMemory() {
129:     if (TopLevelRegion == nullptr)
130:       return;
131:
132:     TopLevelRegion->releaseMemory();
133:     delete TopLevelRegion;
134:     TopLevelRegion = nullptr;
135:   }
136:
137:   const ConvergenceRegion *getTopLevelRegion() const { return TopLevelRegion; }
138:   ConvergenceRegion *getWritableTopLevelRegion() const {
139:     return TopLevelRegion;
140:   }
141: };
142:
143: } // namespace SPIRV
144:
145: // Wrapper around the function above to use it with the legacy pass manager.
146: class SPIRVConvergenceRegionAnalysisWrapperPass : public FunctionPass {
147:   SPIRV::ConvergenceRegionInfo CRI;
148:
149: public:
150:   static char ID;
151:
152:   SPIRVConvergenceRegionAnalysisWrapperPass();
153:
154:   void getAnalysisUsage(AnalysisUsage &AU) const override {
155:     AU.setPreservesAll();
156:     AU.addRequired<LoopInfoWrapperPass>();
157:     AU.addRequired<DominatorTreeWrapperPass>();
158:   };
159:
160:   bool runOnFunction(Function &F) override;
```
- EN: This range defines or declares important types such as releaseMemory, getTopLevelRegion, getWritableTopLevelRegion, SPIRVConvergenceRegionAnalysisWrapperPass, shaping the data model used by SPIRVConvergenceRegionAnalysis.h.
- CN: 这一段定义或声明了 releaseMemory、getTopLevelRegion、getWritableTopLevelRegion、SPIRVConvergenceRegionAnalysisWrapperPass 等关键类型，构成 SPIRVConvergenceRegionAnalysis.h 使用的数据模型。

### Lines 161-184
```cpp
161:
162:   SPIRV::ConvergenceRegionInfo &getRegionInfo() { return CRI; }
163:   const SPIRV::ConvergenceRegionInfo &getRegionInfo() const { return CRI; }
164: };
165:
166: // Wrapper around the function above to use it with the new pass manager.
167: class SPIRVConvergenceRegionAnalysis
168:     : public AnalysisInfoMixin<SPIRVConvergenceRegionAnalysis> {
169:   friend AnalysisInfoMixin<SPIRVConvergenceRegionAnalysis>;
170:   static AnalysisKey Key;
171:
172: public:
173:   using Result = SPIRV::ConvergenceRegionInfo;
174:
175:   Result run(Function &F, FunctionAnalysisManager &AM);
176: };
177:
178: namespace SPIRV {
179: ConvergenceRegionInfo getConvergenceRegions(Function &F, DominatorTree &DT,
180:                                             LoopInfo &LI);
181: } // namespace SPIRV
182:
183: } // namespace llvm
184: #endif // LLVM_LIB_TARGET_SPIRV_ANALYSIS_SPIRVCONVERGENCEREGIONANALYSIS_H
```
- EN: This range defines or declares important types such as getRegionInfo, SPIRVConvergenceRegionAnalysis, run, shaping the data model used by SPIRVConvergenceRegionAnalysis.h.
- CN: 这一段定义或声明了 getRegionInfo、SPIRVConvergenceRegionAnalysis、run 等关键类型，构成 SPIRVConvergenceRegionAnalysis.h 使用的数据模型。

## Key Concepts / 关键概念

- EN: SPIR-V backend code must preserve rich type information and encode capabilities/extensions explicitly.
  - CN: SPIR-V 后端需要保留丰富的类型信息，并显式编码 capability 与 extension。
- EN: Key symbols in this file include IntrinsicInst, SPIRVSubtarget, MachineFunction, MachineModuleInfo, getConvergenceToken, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 IntrinsicInst, SPIRVSubtarget, MachineFunction, MachineModuleInfo, getConvergenceToken，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- LLVM infrastructure / LLVM 基础设施:
  - `llvm/ADT/SmallPtrSet.h`
  - `llvm/Analysis/CFG.h`
  - `llvm/Analysis/LoopInfo.h`
  - `llvm/IR/Dominators.h`
- System/standard headers / 系统或标准头文件:
  - `optional`
