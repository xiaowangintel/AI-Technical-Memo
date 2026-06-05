# PhaseManager.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `polly/lib/Pass/PhaseManager.cpp`
- **Repository**: `llvm-project` / `llvm-project`
- **Purpose (EN)**: Implements the Polly phase manager that sequences major optimization and analysis stages.
- **用途（CN）**: 实现 Polly 的阶段管理器，用于串联主要优化与分析阶段。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
1: //===------ PhaseManager.cpp ------------------------------------*- C++ -*-===//
2: //
3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
4: // See https://llvm.org/LICENSE.txt for license information.
5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
6: //
7: //===----------------------------------------------------------------------===//
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 9-32
```cpp
9: #include "polly/Pass/PhaseManager.h"
10: #include "polly/CodeGen/CodeGeneration.h"
11: #include "polly/CodeGen/IslAst.h"
12: #include "polly/CodePreparation.h"
13: #include "polly/DeLICM.h"
14: #include "polly/DeadCodeElimination.h"
15: #include "polly/DependenceInfo.h"
16: #include "polly/FlattenSchedule.h"
17: #include "polly/ForwardOpTree.h"
18: #include "polly/JSONExporter.h"
19: #include "polly/MaximalStaticExpansion.h"
20: #include "polly/PruneUnprofitable.h"
21: #include "polly/ScheduleOptimizer.h"
22: #include "polly/ScopDetection.h"
23: #include "polly/ScopDetectionDiagnostic.h"
24: #include "polly/ScopGraphPrinter.h"
25: #include "polly/ScopInfo.h"
26: #include "polly/Simplify.h"
27: #include "polly/Support/PollyDebug.h"
28: #include "llvm/ADT/PriorityWorklist.h"
29: #include "llvm/Analysis/AssumptionCache.h"
30: #include "llvm/Analysis/OptimizationRemarkEmitter.h"
31: #include "llvm/Analysis/TargetTransformInfo.h"
32: #include "llvm/IR/Module.h"
```
- **EN**: Brings in the headers required by this compilation unit, covering Polly, LLVM, ISL, or standard-library facilities.
- **CN**: 这里引入当前编译单元所需的头文件，覆盖 Polly、LLVM、ISL 或标准库能力。

### Lines 34-34
```cpp
34: #define DEBUG_TYPE "polly-pass"
```
- **EN**: Uses preprocessor directives to control compilation, feature selection, debugging tags, or header guards.
- **CN**: 这里使用预处理指令来控制编译、特性选择、调试标签或头文件保护。

### Lines 36-36
```cpp
36: using namespace polly;
```
- **EN**: Introduces namespace aliases/shorthands so later code can refer to LLVM or Polly entities more compactly.
- **CN**: 这里引入命名空间简写，使后续代码能更紧凑地引用 LLVM 或 Polly 实体。

### Lines 37-37
```cpp
37: using namespace llvm;
```
- **EN**: Introduces namespace aliases/shorthands so later code can refer to LLVM or Polly entities more compactly.
- **CN**: 这里引入命名空间简写，使后续代码能更紧凑地引用 LLVM 或 Polly 实体。

### Lines 39-39
```cpp
39: namespace {
```
- **EN**: Carries supporting computation, data movement, or bookkeeping for the surrounding algorithm.
- **CN**: 这里承担周边算法所需的辅助计算、数据传递或簿记工作。

### Lines 41-41
```cpp
41: /// Recurse through all subregions and all regions and add them to RQ.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 42-46
```cpp
42: static void addRegionIntoQueue(Region &R, SmallVector<Region *> &RQ) {
43:   RQ.push_back(&R);
44:   for (const auto &E : R)
45:     addRegionIntoQueue(*E, RQ);
46: }
```
- **EN**: Executes an iterative step of the algorithm, usually scanning statements, dimensions, accesses, operands, or schedule nodes.
- **CN**: 这里执行算法中的迭代步骤，通常遍历语句、维度、访问、操作数或调度节点。

### Lines 48-54
```cpp
48: /// The phase pipeline of Polly to be embedded into another pass manager than
49: /// runs passes on functions.
50: ///
51: /// Polly holds state besides LLVM-IR (RegionInfo and ScopInfo) between phases
52: /// that LLVM pass managers do not consider when scheduling analyses and passes.
53: /// That is, the ScopInfo must persist between phases that a pass manager must
54: /// not invalidate to recompute later.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 55-59
```cpp
55: class PhaseManager {
56: private:
57:   Function &F;
58:   FunctionAnalysisManager &FAM;
59:   PollyPassOptions Opts;
```
- **EN**: Defines `PhaseManager`, a named type that packages structured state or classification rules for the surrounding algorithm.
- **CN**: 这里定义了 `PhaseManager`，它是一个具名类型，用于为周边算法封装结构化状态或分类规则。

### Lines 61-63
```cpp
61: public:
62:   PhaseManager(Function &F, FunctionAnalysisManager &FAM, PollyPassOptions Opts)
63:       : F(F), FAM(FAM), Opts(std::move(Opts)) {}
```
- **EN**: Introduces or continues `PhaseManager`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `PhaseManager`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 65-65
```cpp
65:   /// Execute Polly's phases as indicated by the options.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 66-76
```cpp
66:   bool run() {
67:     // Get analyses from the function pass manager.
68:     // These must be preserved during all phases so that if processing one SCoP
69:     // has finished, the next SCoP can still use them. Recomputing is not an
70:     // option because ScopDetection stores references to the old results.
71:     // TODO: CodePreparation doesn't actually need these analysis, it just keeps
72:     // them up-to-date. If they are not computed yet, can also compute after the
73:     // prepare phase.
74:     LoopInfo &LI = FAM.getResult<LoopAnalysis>(F);
75:     DominatorTree &DT = FAM.getResult<DominatorTreeAnalysis>(F);
76:     bool ModifiedIR = false;
```
- **EN**: Introduces or continues `run`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `run`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 78-80
```cpp
78:     // Phase: prepare
79:     // TODO: Setting ModifiedIR will invalidate any analysis, even if DT, LI are
80:     // preserved.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 81-89
```cpp
81:     if (Opts.isPhaseEnabled(PassPhase::Prepare)) {
82:       if (runCodePreparation(F, &DT, &LI, nullptr)) {
83:         PreservedAnalyses PA;
84:         PA.preserve<DominatorTreeAnalysis>();
85:         PA.preserve<LoopAnalysis>();
86:         FAM.invalidate(F, PA);
87:         ModifiedIR = true;
88:       }
89:     }
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 91-91
```cpp
91:     // Can't do anything without detection
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 92-93
```cpp
92:     if (!Opts.isPhaseEnabled(PassPhase::Detection))
93:       return false;
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 95-95
```cpp
95:     AAResults &AA = FAM.getResult<AAManager>(F);
```
- **EN**: Introduces or continues `getResult<AAManager>`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `getResult<AAManager>`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 96-96
```cpp
96:     ScalarEvolution &SE = FAM.getResult<ScalarEvolutionAnalysis>(F);
```
- **EN**: Introduces or continues `getResult<ScalarEvolutionAnalysis>`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `getResult<ScalarEvolutionAnalysis>`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 97-98
```cpp
97:     OptimizationRemarkEmitter &ORE =
98:         FAM.getResult<OptimizationRemarkEmitterAnalysis>(F);
```
- **EN**: Introduces or continues `getResult<OptimizationRemarkEmitterAnalysis>`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `getResult<OptimizationRemarkEmitterAnalysis>`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 100-101
```cpp
100:     // ScopDetection is modifying RegionInfo, do not cache it, nor use a cached
101:     // version.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 102-102
```cpp
102:     RegionInfo RI = RegionInfoAnalysis().run(F, FAM);
```
- **EN**: Introduces or continues `RegionInfoAnalysis`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `RegionInfoAnalysis`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 104-104
```cpp
104:     // Phase: detection
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 105-105
```cpp
105:     ScopDetection SD(DT, SE, LI, RI, AA, ORE);
```
- **EN**: Introduces or continues `SD`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `SD`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 106-106
```cpp
106:     SD.detect(F);
```
- **EN**: Introduces or continues `detect`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `detect`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 107-112
```cpp
107:     if (Opts.isPhaseEnabled(PassPhase::PrintDetect)) {
108:       outs() << "Detected Scops in Function " << F.getName() << "\n";
109:       for (const Region *R : SD.ValidRegions)
110:         outs() << "Valid Region for Scop: " << R->getNameStr() << '\n';
111:       outs() << "\n";
112:     }
```
- **EN**: Executes an iterative step of the algorithm, usually scanning statements, dimensions, accesses, operands, or schedule nodes.
- **CN**: 这里执行算法中的迭代步骤，通常遍历语句、维度、访问、操作数或调度节点。

### Lines 114-115
```cpp
114:     if (Opts.isPhaseEnabled(PassPhase::DotScops))
115:       printGraphForFunction(F, &SD, "scops", false);
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 116-117
```cpp
116:     if (Opts.isPhaseEnabled(PassPhase::DotScopsOnly))
117:       printGraphForFunction(F, &SD, "scopsonly", true);
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 119-121
```cpp
119:     auto ViewScops = [&](const char *Name, bool IsSimply) {
120:       if (Opts.ViewFilter.empty() && !F.getName().count(Opts.ViewFilter))
121:         return;
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 123-124
```cpp
123:       if (Opts.ViewAll || std::distance(SD.begin(), SD.end()) > 0)
124:         viewGraphForFunction(F, &SD, Name, IsSimply);
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 125-125
```cpp
125:     };
```
- **EN**: Closes the current block or scope and hands control back to the surrounding context.
- **CN**: 这里关闭当前代码块或作用域，并将控制流交还给外层上下文。

### Lines 126-127
```cpp
126:     if (Opts.isPhaseEnabled(PassPhase::ViewScops))
127:       ViewScops("scops", false);
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 128-129
```cpp
128:     if (Opts.isPhaseEnabled(PassPhase::ViewScopsOnly))
129:       ViewScops("scopsonly", true);
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 131-131
```cpp
131:     // Can't do anything after this without ScopInfo.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 132-133
```cpp
132:     if (!Opts.isPhaseEnabled(PassPhase::ScopInfo))
133:       return false;
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 135-135
```cpp
135:     // Phase: scops
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 136-136
```cpp
136:     AssumptionCache &AC = FAM.getResult<AssumptionAnalysis>(F);
```
- **EN**: Introduces or continues `getResult<AssumptionAnalysis>`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `getResult<AssumptionAnalysis>`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 137-137
```cpp
137:     const DataLayout &DL = F.getParent()->getDataLayout();
```
- **EN**: Introduces or continues `getParent`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `getParent`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 138-138
```cpp
138:     ScopInfo Info(DL, SD, SE, LI, AA, DT, AC, ORE);
```
- **EN**: Introduces or continues `Info`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `Info`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 139-142
```cpp
139:     if (Opts.isPhaseEnabled(PassPhase::PrintScopInfo)) {
140:       if (Region *TLR = RI.getTopLevelRegion()) {
141:         SmallVector<Region *> Regions;
142:         addRegionIntoQueue(*TLR, Regions);
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 144-144
```cpp
144:         // reverse iteration because the regression tests expect it.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 145-155
```cpp
145:         for (Region *R : reverse(Regions)) {
146:           Scop *S = Info.getScop(R);
147:           outs() << "Printing analysis 'Polly - Create polyhedral "
148:                     "description of Scops' for region: '"
149:                  << R->getNameStr() << "' in function '" << F.getName()
150:                  << "':\n";
151:           if (S)
152:             outs() << *S;
153:           else
154:             outs() << "Invalid Scop!\n";
155:         }
```
- **EN**: Executes an iterative step of the algorithm, usually scanning statements, dimensions, accesses, operands, or schedule nodes.
- **CN**: 这里执行算法中的迭代步骤，通常遍历语句、维度、访问、操作数或调度节点。

### Lines 156-156
```cpp
156:       }
```
- **EN**: Closes the current block or scope and hands control back to the surrounding context.
- **CN**: 这里关闭当前代码块或作用域，并将控制流交还给外层上下文。

### Lines 157-157
```cpp
157:     }
```
- **EN**: Closes the current block or scope and hands control back to the surrounding context.
- **CN**: 这里关闭当前代码块或作用域，并将控制流交还给外层上下文。

### Lines 159-159
```cpp
159:     SmallPriorityWorklist<const Region *, 4> Worklist;
```
- **EN**: Carries supporting computation, data movement, or bookkeeping for the surrounding algorithm.
- **CN**: 这里承担周边算法所需的辅助计算、数据传递或簿记工作。

### Lines 160-161
```cpp
160:     for (const Region *R : SD)
161:       Worklist.insert(R);
```
- **EN**: Executes an iterative step of the algorithm, usually scanning statements, dimensions, accesses, operands, or schedule nodes.
- **CN**: 这里执行算法中的迭代步骤，通常遍历语句、维度、访问、操作数或调度节点。

### Lines 163-163
```cpp
163:     TargetTransformInfo &TTI = FAM.getResult<TargetIRAnalysis>(F);
```
- **EN**: Introduces or continues `getResult<TargetIRAnalysis>`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `getResult<TargetIRAnalysis>`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 164-173
```cpp
164:     while (!Worklist.empty()) {
165:       const Region *R = Worklist.pop_back_val();
166:       Scop *S = Info.getScop(R);
167:       if (!S) {
168:         // This can happen if the region is not maximal, is not determined a
169:         // valid SCoP by ScopBuilder, or codegenning of a previous SCoP made
170:         // this region not-a-SCoP anymore.
171:         POLLY_DEBUG(dbgs() << "SCoP in Region '" << *R << "' disappeared");
172:         continue;
173:       }
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 175-176
```cpp
175:       if (!SD.isMaxRegionInScop(*R, /*Verify=*/false))
176:         continue;
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 178-178
```cpp
178:       // Phase: flatten
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 179-180
```cpp
179:       if (Opts.isPhaseEnabled(PassPhase::Flatten))
180:         runFlattenSchedulePass(*S);
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 182-184
```cpp
182:       // Phase: deps
183:       // Actual analysis runs on-demand, so it does not matter whether the phase
184:       // is actually enabled, but use this location to print dependencies.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 185-185
```cpp
185:       DependenceAnalysis::Result DA = runDependenceAnalysis(*S);
```
- **EN**: Introduces or continues `runDependenceAnalysis`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `runDependenceAnalysis`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 186-190
```cpp
186:       if (Opts.isPhaseEnabled(PassPhase::PrintDependences)) {
187:         assert(Opts.isPhaseEnabled(PassPhase::Dependences));
188:         const Dependences &D = DA.getDependences(Opts.PrintDepsAnalysisLevel);
189:         D.print(outs());
190:       }
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 192-192
```cpp
192:       // Phase: import-jscop
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 193-194
```cpp
193:       if (Opts.isPhaseEnabled(PassPhase::ImportJScop))
194:         runImportJSON(*S, DA);
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 196-196
```cpp
196:       // Phase: simplify-0
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 197-197
```cpp
197:       bool ModifiedSinceSimplify = true;
```
- **EN**: Carries supporting computation, data movement, or bookkeeping for the surrounding algorithm.
- **CN**: 这里承担周边算法所需的辅助计算、数据传递或簿记工作。

### Lines 198-201
```cpp
198:       if (Opts.isPhaseEnabled(PassPhase::Simplify0)) {
199:         runSimplify(*S, 0);
200:         ModifiedSinceSimplify = false;
201:       }
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 203-203
```cpp
203:       // Phase: optree
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 204-207
```cpp
204:       if (Opts.isPhaseEnabled(PassPhase::Optree)) {
205:         bool ModifiedByOptree = runForwardOpTree(*S);
206:         ModifiedSinceSimplify |= ModifiedByOptree;
207:       }
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 209-209
```cpp
209:       // Phase: delicm
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 210-213
```cpp
210:       if (Opts.isPhaseEnabled(PassPhase::DeLICM)) {
211:         bool ModifiedByDelicm = runDeLICM(*S);
212:         ModifiedSinceSimplify |= ModifiedByDelicm;
213:       }
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 215-217
```cpp
215:       // Phase: simplify-1
216:       // If we have already run simplify-0, do not re-run it if the SCoP has not
217:       // changed since then.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 218-221
```cpp
218:       if (ModifiedSinceSimplify && Opts.isPhaseEnabled(PassPhase::Simplify1)) {
219:         runSimplify(*S, 1);
220:         ModifiedSinceSimplify = false;
221:       }
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 223-223
```cpp
223:       // Phase: dce
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 224-225
```cpp
224:       if (Opts.isPhaseEnabled(PassPhase::DeadCodeElimination))
225:         runDeadCodeElim(*S, DA);
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 227-227
```cpp
227:       // Phase: mse
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 228-229
```cpp
228:       if (Opts.isPhaseEnabled(PassPhase::MaximumStaticExtension))
229:         runMaximalStaticExpansion(*S, DA);
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 231-231
```cpp
231:       // Phase: prune
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 232-233
```cpp
232:       if (Opts.isPhaseEnabled(PassPhase::PruneUnprofitable))
233:         runPruneUnprofitable(*S);
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 235-235
```cpp
235:       // Phase: opt-isl
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 236-237
```cpp
236:       if (Opts.isPhaseEnabled(PassPhase::Optimization))
237:         runIslScheduleOptimizer(*S, &TTI, DA);
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 239-239
```cpp
239:       // Phase: import-jscop
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 240-241
```cpp
240:       if (Opts.isPhaseEnabled(PassPhase::ExportJScop))
241:         runExportJSON(*S);
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 243-244
```cpp
243:       // Phase: ast
244:       // Cannot run codegen unless ast is enabled
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 245-246
```cpp
245:       if (!Opts.isPhaseEnabled(PassPhase::AstGen))
246:         continue;
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 247-247
```cpp
247:       std::unique_ptr<IslAstInfo> IslAst = runIslAstGen(*S, DA);
```
- **EN**: Introduces or continues `runIslAstGen`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `runIslAstGen`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 249-249
```cpp
249:       // Phase: codegen
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 250-251
```cpp
250:       if (!Opts.isPhaseEnabled(PassPhase::CodeGen))
251:         continue;
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 252-252
```cpp
252:       bool ModifiedByCodeGen = runCodeGeneration(*S, RI, *IslAst);
```
- **EN**: Introduces or continues `runCodeGeneration`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `runCodeGeneration`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 253-254
```cpp
253:       if (ModifiedByCodeGen) {
254:         ModifiedIR = true;
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 256-258
```cpp
256:         // Discard old polly::Scop objects because they may refer to invalidated
257:         // LLVM-IR instructions and SCEV expressions. ScopInfo will recreate
258:         // them on demand.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 259-259
```cpp
259:         Info.invalidate();
```
- **EN**: Introduces or continues `invalidate`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `invalidate`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 260-260
```cpp
260:       }
```
- **EN**: Closes the current block or scope and hands control back to the surrounding context.
- **CN**: 这里关闭当前代码块或作用域，并将控制流交还给外层上下文。

### Lines 261-261
```cpp
261:     }
```
- **EN**: Closes the current block or scope and hands control back to the surrounding context.
- **CN**: 这里关闭当前代码块或作用域，并将控制流交还给外层上下文。

### Lines 263-263
```cpp
263:     return ModifiedIR;
```
- **EN**: Returns the computed value or forwards the current result to the caller.
- **CN**: 这里返回计算结果，或将当前结果继续传递给调用方。

### Lines 264-264
```cpp
264:   }
```
- **EN**: Closes the current block or scope and hands control back to the surrounding context.
- **CN**: 这里关闭当前代码块或作用域，并将控制流交还给外层上下文。

### Lines 265-265
```cpp
265: };
```
- **EN**: Closes the current block or scope and hands control back to the surrounding context.
- **CN**: 这里关闭当前代码块或作用域，并将控制流交还给外层上下文。

### Lines 266-266
```cpp
266: } // namespace
```
- **EN**: Closes the current block or scope and hands control back to the surrounding context.
- **CN**: 这里关闭当前代码块或作用域，并将控制流交还给外层上下文。

### Lines 268-321
```cpp
268: StringRef polly::getPhaseName(PassPhase Phase) {
269:   switch (Phase) {
270:   case PassPhase::Prepare:
271:     return "prepare";
272:   case PassPhase::Detection:
273:     return "detect";
274:   case PassPhase::PrintDetect:
275:     return "print-detect";
276:   case PassPhase::DotScops:
277:     return "dot-scops";
278:   case PassPhase::DotScopsOnly:
279:     return "dot-scops-only";
280:   case PassPhase::ViewScops:
281:     return "view-scops";
282:   case PassPhase::ViewScopsOnly:
283:     return "view-scops-only";
284:   case PassPhase::ScopInfo:
285:     return "scops";
286:   case PassPhase::PrintScopInfo:
287:     return "print-scops";
288:   case PassPhase::Flatten:
289:     return "flatten";
290:   case PassPhase::Dependences:
291:     return "deps";
292:   case PassPhase::PrintDependences:
293:     return "print-deps";
294:   case PassPhase::ImportJScop:
295:     return "import-jscop";
296:   case PassPhase::Simplify0:
297:     return "simplify-0";
298:   case PassPhase::Optree:
299:     return "optree";
300:   case PassPhase::DeLICM:
301:     return "delicm";
302:   case PassPhase::Simplify1:
303:     return "simplify-1";
304:   case PassPhase::DeadCodeElimination:
305:     return "dce";
306:   case PassPhase::MaximumStaticExtension:
307:     return "mse";
308:   case PassPhase::PruneUnprofitable:
309:     return "prune";
310:   case PassPhase::Optimization:
311:     return "opt-isl"; // "opt" would conflict with the llvm executable
312:   case PassPhase::ExportJScop:
313:     return "export-jscop";
314:   case PassPhase::AstGen:
315:     return "ast";
316:   case PassPhase::CodeGen:
317:     return "codegen";
318:   default:
319:     llvm_unreachable("Unexpected phase");
320:   }
321: }
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 323-350
```cpp
323: PassPhase polly::parsePhase(StringRef Name) {
324:   return StringSwitch<PassPhase>(Name)
325:       .Case("prepare", PassPhase::Prepare)
326:       .Case("detect", PassPhase::Detection)
327:       .Case("print-detect", PassPhase::PrintDetect)
328:       .Case("dot-scops", PassPhase::DotScops)
329:       .Case("dot-scops-only", PassPhase::DotScopsOnly)
330:       .Case("view-scops", PassPhase::ViewScops)
331:       .Case("view-scops-only", PassPhase::ViewScopsOnly)
332:       .Case("scops", PassPhase::ScopInfo)
333:       .Case("print-scops", PassPhase::PrintScopInfo)
334:       .Case("flatten", PassPhase::Flatten)
335:       .Case("deps", PassPhase::Dependences)
336:       .Case("print-deps", PassPhase::PrintDependences)
337:       .Case("import-jscop", PassPhase::ImportJScop)
338:       .Case("simplify-0", PassPhase::Simplify0)
339:       .Case("optree", PassPhase::Optree)
340:       .Case("delicm", PassPhase::DeLICM)
341:       .Case("simplify-1", PassPhase::Simplify1)
342:       .Case("dce", PassPhase::DeadCodeElimination)
343:       .Case("mse", PassPhase::MaximumStaticExtension)
344:       .Case("prune", PassPhase::PruneUnprofitable)
345:       .Case("opt-isl", PassPhase::Optimization)
346:       .Case("export-jscop", PassPhase::ExportJScop)
347:       .Case("ast", PassPhase::AstGen)
348:       .Case("codegen", PassPhase::CodeGen)
349:       .Default(PassPhase::None);
350: }
```
- **EN**: Introduces or continues `polly::parsePhase`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `polly::parsePhase`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 352-355
```cpp
352: bool polly::dependsOnDependenceInfo(PassPhase Phase) {
353:   // Nothing before dep phase can depend on it
354:   if (static_cast<size_t>(Phase) <= static_cast<size_t>(PassPhase::Dependences))
355:     return false;
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 357-370
```cpp
357:   switch (Phase) {
358:   case PassPhase::Simplify0:
359:   case PassPhase::Optree:
360:   case PassPhase::DeLICM:
361:   case PassPhase::Simplify1:
362:   case PassPhase::PruneUnprofitable:
363:   case PassPhase::ImportJScop:
364:   case PassPhase::ExportJScop:
365:   case PassPhase::AstGen: // transitively through codegen
366:   case PassPhase::CodeGen:
367:     return false;
368:   default:
369:     return true;
370:   }
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 371-371
```cpp
371: }
```
- **EN**: Closes the current block or scope and hands control back to the surrounding context.
- **CN**: 这里关闭当前代码块或作用域，并将控制流交还给外层上下文。

### Lines 373-379
```cpp
373: void PollyPassOptions::enableEnd2End() {
374:   setPhaseEnabled(PassPhase::Detection);
375:   setPhaseEnabled(PassPhase::ScopInfo);
376:   setPhaseEnabled(PassPhase::Dependences);
377:   setPhaseEnabled(PassPhase::AstGen);
378:   setPhaseEnabled(PassPhase::CodeGen);
379: }
```
- **EN**: Introduces or continues `PollyPassOptions::enableEnd2End`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `PollyPassOptions::enableEnd2End`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 381-389
```cpp
381: void PollyPassOptions::enableDefaultOpts() {
382:   setPhaseEnabled(PassPhase::Prepare);
383:   setPhaseEnabled(PassPhase::Simplify0);
384:   setPhaseEnabled(PassPhase::Optree);
385:   setPhaseEnabled(PassPhase::DeLICM);
386:   setPhaseEnabled(PassPhase::Simplify1);
387:   setPhaseEnabled(PassPhase::PruneUnprofitable);
388:   setPhaseEnabled(PassPhase::Optimization);
389: }
```
- **EN**: Introduces or continues `PollyPassOptions::enableDefaultOpts`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `PollyPassOptions::enableDefaultOpts`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 391-398
```cpp
391: void PollyPassOptions::disableAfter(PassPhase Phase) {
392:   assert(Phase != PassPhase::None);
393:   for (PassPhase P : enum_seq_inclusive(Phase, PassPhase::PassPhaseLast)) {
394:     if (P == Phase)
395:       continue;
396:     setPhaseEnabled(P, false);
397:   }
398: }
```
- **EN**: Executes an iterative step of the algorithm, usually scanning statements, dimensions, accesses, operands, or schedule nodes.
- **CN**: 这里执行算法中的迭代步骤，通常遍历语句、维度、访问、操作数或调度节点。

### Lines 400-404
```cpp
400: Error PollyPassOptions::checkConsistency() const {
401:   for (PassPhase P : enum_seq_inclusive(PassPhase::PassPhaseFirst,
402:                                         PassPhase::PassPhaseLast)) {
403:     if (!isPhaseEnabled(P))
404:       continue;
```
- **EN**: Executes an iterative step of the algorithm, usually scanning statements, dimensions, accesses, operands, or schedule nodes.
- **CN**: 这里执行算法中的迭代步骤，通常遍历语句、维度、访问、操作数或调度节点。

### Lines 406-406
```cpp
406:     // Prepare and Detection have no requirements
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 407-408
```cpp
407:     if (P == PassPhase::Prepare || P == PassPhase::Detection)
408:       continue;
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 410-414
```cpp
410:     if (!isPhaseEnabled(PassPhase::Detection))
411:       return make_error<StringError>(
412:           formatv("'{0}' requires 'detect' to be enabled", getPhaseName(P))
413:               .str(),
414:           inconvertibleErrorCode());
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 416-417
```cpp
416:     if (static_cast<size_t>(P) < static_cast<size_t>(PassPhase::ScopInfo))
417:       continue;
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 419-423
```cpp
419:     if (!isPhaseEnabled(PassPhase::ScopInfo))
420:       return make_error<StringError>(
421:           formatv("'{0}' requires 'scops' to be enabled", getPhaseName(P))
422:               .str(),
423:           inconvertibleErrorCode());
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 425-428
```cpp
425:     if (dependsOnDependenceInfo(P) && !isPhaseEnabled(PassPhase::Dependences))
426:       return make_error<StringError>(
427:           formatv("'{0}' requires 'deps' to be enabled", getPhaseName(P)).str(),
428:           inconvertibleErrorCode());
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 429-429
```cpp
429:   }
```
- **EN**: Closes the current block or scope and hands control back to the surrounding context.
- **CN**: 这里关闭当前代码块或作用域，并将控制流交还给外层上下文。

### Lines 431-433
```cpp
431:   if (isPhaseEnabled(PassPhase::CodeGen) && !isPhaseEnabled(PassPhase::AstGen))
432:     return make_error<StringError>("'codegen' requires 'ast' to be enabled",
433:                                    inconvertibleErrorCode());
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 435-435
```cpp
435:   return Error::success();
```
- **EN**: Introduces or continues `Error::success`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `Error::success`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 436-436
```cpp
436: }
```
- **EN**: Closes the current block or scope and hands control back to the surrounding context.
- **CN**: 这里关闭当前代码块或作用域，并将控制流交还给外层上下文。

### Lines 438-441
```cpp
438: bool polly::runPollyPass(Function &F, FunctionAnalysisManager &FAM,
439:                          PollyPassOptions Opts) {
440:   return PhaseManager(F, FAM, std::move(Opts)).run();
441: }
```
- **EN**: Introduces or continues `polly::runPollyPass`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `polly::runPollyPass`，它是该文件分析或变换流水线中的一个步骤函数。

## Key Concepts / 关键概念

- **Phase sequencing** / **阶段编排**
- **Analysis invalidation** / **分析失效管理**
- **Pass orchestration** / **Pass 编排**
- **ScalarEvolution reasoning** / **ScalarEvolution 推理**
- **Schedule manipulation** / **调度操作**
- **Memory/access reasoning** / **内存/访问推理**

## Dependencies / 依赖关系

- **EN**: Polly headers such as polly/Pass/PhaseManager.h, polly/CodeGen/CodeGeneration.h, polly/CodeGen/IslAst.h, polly/CodePreparation.h
- **CN**: Polly 头文件，例如 polly/Pass/PhaseManager.h, polly/CodeGen/CodeGeneration.h, polly/CodeGen/IslAst.h, polly/CodePreparation.h
- **EN**: LLVM infrastructure headers such as llvm/ADT/PriorityWorklist.h, llvm/Analysis/AssumptionCache.h, llvm/Analysis/OptimizationRemarkEmitter.h, llvm/Analysis/TargetTransformInfo.h
- **CN**: LLVM 基础设施头文件，例如 llvm/ADT/PriorityWorklist.h, llvm/Analysis/AssumptionCache.h, llvm/Analysis/OptimizationRemarkEmitter.h, llvm/Analysis/TargetTransformInfo.h
