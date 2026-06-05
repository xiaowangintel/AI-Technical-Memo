# VirtualInstruction.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `polly/lib/Support/VirtualInstruction.cpp`
- **Repository**: `llvm-project` / `llvm-project`
- **Purpose (EN)**: Implements virtual-instruction abstractions used to reason about synthesized scalar computations inside Polly.
- **用途（CN）**: 实现虚拟指令抽象，用于在 Polly 中推理合成的标量计算。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
1: //===------ VirtualInstruction.cpp ------------------------------*- C++ -*-===//
2: //
3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
4: // See https://llvm.org/LICENSE.txt for license information.
5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
6: //
7: //===----------------------------------------------------------------------===//
8: //
9: // Tools for determining which instructions are within a statement and the
10: // nature of their operands.
11: //
12: //===----------------------------------------------------------------------===//
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 14-14
```cpp
14: #include "polly/Support/VirtualInstruction.h"
```
- **EN**: Brings in the headers required by this compilation unit, covering Polly, LLVM, ISL, or standard-library facilities.
- **CN**: 这里引入当前编译单元所需的头文件，覆盖 Polly、LLVM、ISL 或标准库能力。

### Lines 16-16
```cpp
16: using namespace polly;
```
- **EN**: Introduces namespace aliases/shorthands so later code can refer to LLVM or Polly entities more compactly.
- **CN**: 这里引入命名空间简写，使后续代码能更紧凑地引用 LLVM 或 Polly 实体。

### Lines 17-17
```cpp
17: using namespace llvm;
```
- **EN**: Introduces namespace aliases/shorthands so later code can refer to LLVM or Polly entities more compactly.
- **CN**: 这里引入命名空间简写，使后续代码能更紧凑地引用 LLVM 或 Polly 实体。

### Lines 19-24
```cpp
19: VirtualUse VirtualUse::create(Scop *S, const Use &U, LoopInfo *LI,
20:                               bool Virtual) {
21:   auto *UserBB = getUseBlock(U);
22:   Loop *UserScope = LI->getLoopFor(UserBB);
23:   Instruction *UI = dyn_cast<Instruction>(U.getUser());
24:   ScopStmt *UserStmt = S->getStmtFor(UI);
```
- **EN**: Introduces or continues `VirtualUse::create`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `VirtualUse::create`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 26-27
```cpp
26:   // Uses by PHI nodes are always reading values written by other statements,
27:   // except it is within a region statement.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 28-31
```cpp
28:   if (PHINode *PHI = dyn_cast<PHINode>(UI)) {
29:     // Handle PHI in exit block.
30:     if (S->getRegion().getExit() == PHI->getParent())
31:       return VirtualUse(UserStmt, U.get(), Inter, nullptr, nullptr);
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 33-34
```cpp
33:     if (UserStmt->getEntryBlock() != PHI->getParent())
34:       return VirtualUse(UserStmt, U.get(), Intra, nullptr, nullptr);
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 36-36
```cpp
36:     // The MemoryAccess is expected to be set if @p Virtual is true.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 37-37
```cpp
37:     MemoryAccess *IncomingMA = nullptr;
```
- **EN**: Carries supporting computation, data movement, or bookkeeping for the surrounding algorithm.
- **CN**: 这里承担周边算法所需的辅助计算、数据传递或簿记工作。

### Lines 38-44
```cpp
38:     if (Virtual) {
39:       if (const ScopArrayInfo *SAI =
40:               S->getScopArrayInfoOrNull(PHI, MemoryKind::PHI)) {
41:         IncomingMA = S->getPHIRead(SAI);
42:         assert(IncomingMA->getStatement() == UserStmt);
43:       }
44:     }
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 46-46
```cpp
46:     return VirtualUse(UserStmt, U.get(), Inter, nullptr, IncomingMA);
```
- **EN**: Introduces or continues `VirtualUse`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `VirtualUse`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 47-47
```cpp
47:   }
```
- **EN**: Closes the current block or scope and hands control back to the surrounding context.
- **CN**: 这里关闭当前代码块或作用域，并将控制流交还给外层上下文。

### Lines 49-49
```cpp
49:   return create(S, UserStmt, UserScope, U.get(), Virtual);
```
- **EN**: Introduces or continues `create`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `create`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 50-50
```cpp
50: }
```
- **EN**: Closes the current block or scope and hands control back to the surrounding context.
- **CN**: 这里关闭当前代码块或作用域，并将控制流交还给外层上下文。

### Lines 52-54
```cpp
52: VirtualUse VirtualUse::create(Scop *S, ScopStmt *UserStmt, Loop *UserScope,
53:                               Value *Val, bool Virtual) {
54:   assert(!isa<StoreInst>(Val) && "a StoreInst cannot be used");
```
- **EN**: Introduces or continues `VirtualUse::create`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `VirtualUse::create`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 56-57
```cpp
56:   if (isa<BasicBlock>(Val))
57:     return VirtualUse(UserStmt, Val, Block, nullptr, nullptr);
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 59-61
```cpp
59:   if (isa<llvm::Constant>(Val) || isa<MetadataAsValue>(Val) ||
60:       isa<InlineAsm>(Val))
61:     return VirtualUse(UserStmt, Val, Constant, nullptr, nullptr);
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 63-65
```cpp
63:   // Is the value synthesizable? If the user has been pruned
64:   // (UserStmt == nullptr), it is either not used anywhere or is synthesizable.
65:   // We assume synthesizable which practically should have the same effect.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 66-66
```cpp
66:   auto *SE = S->getSE();
```
- **EN**: Introduces or continues `getSE`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `getSE`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 67-71
```cpp
67:   if (SE->isSCEVable(Val->getType())) {
68:     const SCEV *ScevExpr = SE->getSCEVAtScope(Val, UserScope);
69:     if (!UserStmt || canSynthesize(Val, *UserStmt->getParent(), SE, UserScope))
70:       return VirtualUse(UserStmt, Val, Synthesizable, ScevExpr, nullptr);
71:   }
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 73-74
```cpp
73:   // FIXME: Inconsistency between lookupInvariantEquivClass and
74:   // getRequiredInvariantLoads. Querying one of them should be enough.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 75-75
```cpp
75:   auto &RIL = S->getRequiredInvariantLoads();
```
- **EN**: Introduces or continues `getRequiredInvariantLoads`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `getRequiredInvariantLoads`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 76-77
```cpp
76:   if (S->lookupInvariantEquivClass(Val) || RIL.count(dyn_cast<LoadInst>(Val)))
77:     return VirtualUse(UserStmt, Val, Hoisted, nullptr, nullptr);
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 79-80
```cpp
79:   // ReadOnly uses may have MemoryAccesses that we want to associate with the
80:   // use. This is why we look for a MemoryAccess here already.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 81-81
```cpp
81:   MemoryAccess *InputMA = nullptr;
```
- **EN**: Carries supporting computation, data movement, or bookkeeping for the surrounding algorithm.
- **CN**: 这里承担周边算法所需的辅助计算、数据传递或簿记工作。

### Lines 82-83
```cpp
82:   if (UserStmt && Virtual)
83:     InputMA = UserStmt->lookupValueReadOf(Val);
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 85-89
```cpp
85:   // Uses are read-only if they have been defined before the SCoP, i.e., they
86:   // cannot be written to inside the SCoP. Arguments are defined before any
87:   // instructions, hence also before the SCoP. If the user has been pruned
88:   // (UserStmt == nullptr) and is not SCEVable, assume it is read-only as it is
89:   // neither an intra- nor an inter-use.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 90-91
```cpp
90:   if (!UserStmt || isa<Argument>(Val))
91:     return VirtualUse(UserStmt, Val, ReadOnly, nullptr, InputMA);
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 93-93
```cpp
93:   auto Inst = cast<Instruction>(Val);
```
- **EN**: Introduces or continues `cast<Instruction>`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `cast<Instruction>`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 94-95
```cpp
94:   if (!S->contains(Inst))
95:     return VirtualUse(UserStmt, Val, ReadOnly, nullptr, InputMA);
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 97-99
```cpp
97:   // A use is inter-statement if either it is defined in another statement, or
98:   // there is a MemoryAccess that reads its value that has been written by
99:   // another statement.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 100-101
```cpp
100:   if (InputMA || (!Virtual && UserStmt != S->getStmtFor(Inst)))
101:     return VirtualUse(UserStmt, Val, Inter, nullptr, InputMA);
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 103-103
```cpp
103:   return VirtualUse(UserStmt, Val, Intra, nullptr, nullptr);
```
- **EN**: Introduces or continues `VirtualUse`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `VirtualUse`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 104-104
```cpp
104: }
```
- **EN**: Closes the current block or scope and hands control back to the surrounding context.
- **CN**: 这里关闭当前代码块或作用域，并将控制流交还给外层上下文。

### Lines 106-130
```cpp
106: void VirtualUse::print(raw_ostream &OS, bool Reproducible) const {
107:   OS << "User: [" << User->getBaseName() << "] ";
108:   switch (Kind) {
109:   case VirtualUse::Constant:
110:     OS << "Constant Op:";
111:     break;
112:   case VirtualUse::Block:
113:     OS << "BasicBlock Op:";
114:     break;
115:   case VirtualUse::Synthesizable:
116:     OS << "Synthesizable Op:";
117:     break;
118:   case VirtualUse::Hoisted:
119:     OS << "Hoisted load Op:";
120:     break;
121:   case VirtualUse::ReadOnly:
122:     OS << "Read-Only Op:";
123:     break;
124:   case VirtualUse::Intra:
125:     OS << "Intra Op:";
126:     break;
127:   case VirtualUse::Inter:
128:     OS << "Inter Op:";
129:     break;
130:   }
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 132-138
```cpp
132:   if (Val) {
133:     OS << ' ';
134:     if (Reproducible)
135:       OS << '"' << Val->getName() << '"';
136:     else
137:       Val->print(OS, true);
138:   }
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 139-142
```cpp
139:   if (ScevExpr) {
140:     OS << ' ';
141:     ScevExpr->print(OS);
142:   }
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 143-144
```cpp
143:   if (InputMA && !Reproducible)
144:     OS << ' ' << InputMA;
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 145-145
```cpp
145: }
```
- **EN**: Closes the current block or scope and hands control back to the surrounding context.
- **CN**: 这里关闭当前代码块或作用域，并将控制流交还给外层上下文。

### Lines 147-147
```cpp
147: #if !defined(NDEBUG) || defined(LLVM_ENABLE_DUMP)
```
- **EN**: Uses preprocessor directives to control compilation, feature selection, debugging tags, or header guards.
- **CN**: 这里使用预处理指令来控制编译、特性选择、调试标签或头文件保护。

### Lines 148-151
```cpp
148: LLVM_DUMP_METHOD void VirtualUse::dump() const {
149:   print(errs(), false);
150:   errs() << '\n';
151: }
```
- **EN**: Introduces or continues `VirtualUse::dump`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `VirtualUse::dump`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 152-152
```cpp
152: #endif
```
- **EN**: Uses preprocessor directives to control compilation, feature selection, debugging tags, or header guards.
- **CN**: 这里使用预处理指令来控制编译、特性选择、调试标签或头文件保护。

### Lines 154-158
```cpp
154: void VirtualInstruction::print(raw_ostream &OS, bool Reproducible) const {
155:   if (!Stmt || !Inst) {
156:     OS << "[null VirtualInstruction]";
157:     return;
158:   }
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 160-160
```cpp
160:   OS << "[" << Stmt->getBaseName() << "]";
```
- **EN**: Carries supporting computation, data movement, or bookkeeping for the surrounding algorithm.
- **CN**: 这里承担周边算法所需的辅助计算、数据传递或簿记工作。

### Lines 161-161
```cpp
161:   Inst->print(OS, !Reproducible);
```
- **EN**: Introduces or continues `print`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `print`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 162-162
```cpp
162: }
```
- **EN**: Closes the current block or scope and hands control back to the surrounding context.
- **CN**: 这里关闭当前代码块或作用域，并将控制流交还给外层上下文。

### Lines 164-164
```cpp
164: #if !defined(NDEBUG) || defined(LLVM_ENABLE_DUMP)
```
- **EN**: Uses preprocessor directives to control compilation, feature selection, debugging tags, or header guards.
- **CN**: 这里使用预处理指令来控制编译、特性选择、调试标签或头文件保护。

### Lines 165-168
```cpp
165: LLVM_DUMP_METHOD void VirtualInstruction::dump() const {
166:   print(errs(), false);
167:   errs() << '\n';
168: }
```
- **EN**: Introduces or continues `VirtualInstruction::dump`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `VirtualInstruction::dump`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 169-169
```cpp
169: #endif
```
- **EN**: Uses preprocessor directives to control compilation, feature selection, debugging tags, or header guards.
- **CN**: 这里使用预处理指令来控制编译、特性选择、调试标签或头文件保护。

### Lines 171-171
```cpp
171: /// Return true if @p Inst cannot be removed, even if it is nowhere referenced.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 172-176
```cpp
172: static bool isRoot(const Instruction *Inst) {
173:   // The store is handled by its MemoryAccess. The load must be reached from the
174:   // roots in order to be marked as used.
175:   if (isa<LoadInst>(Inst) || isa<StoreInst>(Inst))
176:     return false;
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 178-179
```cpp
178:   // Terminator instructions (in region statements) are required for control
179:   // flow.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 180-181
```cpp
180:   if (Inst->isTerminator())
181:     return true;
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 183-183
```cpp
183:   // Writes to memory must be honored.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 184-185
```cpp
184:   if (Inst->mayWriteToMemory())
185:     return true;
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 187-187
```cpp
187:   return false;
```
- **EN**: Returns the computed value or forwards the current result to the caller.
- **CN**: 这里返回计算结果，或将当前结果继续传递给调用方。

### Lines 188-188
```cpp
188: }
```
- **EN**: Closes the current block or scope and hands control back to the surrounding context.
- **CN**: 这里关闭当前代码块或作用域，并将控制流交还给外层上下文。

### Lines 190-191
```cpp
190: /// Return true for MemoryAccesses that cannot be removed because it represents
191: /// an llvm::Value that is used after the SCoP.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 192-196
```cpp
192: static bool isEscaping(MemoryAccess *MA) {
193:   assert(MA->isOriginalValueKind());
194:   Scop *S = MA->getStatement()->getParent();
195:   return S->isEscaping(cast<Instruction>(MA->getAccessValue()));
196: }
```
- **EN**: Introduces or continues `isEscaping`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `isEscaping`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 198-198
```cpp
198: /// Add non-removable virtual instructions in @p Stmt to @p RootInsts.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 199-213
```cpp
199: static void
200: addInstructionRoots(ScopStmt *Stmt,
201:                     SmallVectorImpl<VirtualInstruction> &RootInsts) {
202:   if (!Stmt->isBlockStmt()) {
203:     // In region statements the terminator statement and all statements that
204:     // are not in the entry block cannot be eliminated and consequently must
205:     // be roots.
206:     RootInsts.emplace_back(Stmt,
207:                            Stmt->getRegion()->getEntry()->getTerminator());
208:     for (BasicBlock *BB : Stmt->getRegion()->blocks())
209:       if (Stmt->getRegion()->getEntry() != BB)
210:         for (Instruction &Inst : *BB)
211:           RootInsts.emplace_back(Stmt, &Inst);
212:     return;
213:   }
```
- **EN**: Executes an iterative step of the algorithm, usually scanning statements, dimensions, accesses, operands, or schedule nodes.
- **CN**: 这里执行算法中的迭代步骤，通常遍历语句、维度、访问、操作数或调度节点。

### Lines 215-217
```cpp
215:   for (Instruction *Inst : Stmt->getInstructions())
216:     if (isRoot(Inst))
217:       RootInsts.emplace_back(Stmt, Inst);
```
- **EN**: Executes an iterative step of the algorithm, usually scanning statements, dimensions, accesses, operands, or schedule nodes.
- **CN**: 这里执行算法中的迭代步骤，通常遍历语句、维度、访问、操作数或调度节点。

### Lines 218-218
```cpp
218: }
```
- **EN**: Closes the current block or scope and hands control back to the surrounding context.
- **CN**: 这里关闭当前代码块或作用域，并将控制流交还给外层上下文。

### Lines 220-227
```cpp
220: /// Add non-removable memory accesses in @p Stmt to @p RootInsts.
221: ///
222: /// @param Local If true, all writes are assumed to escape. markAndSweep
223: /// algorithms can use this to be applicable to a single ScopStmt only without
224: /// the risk of removing definitions required by other statements.
225: ///              If false, only writes for SCoP-escaping values are roots.  This
226: ///              is global mode, where such writes must be marked by theirs uses
227: ///              in order to be reachable.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 228-233
```cpp
228: static void addAccessRoots(ScopStmt *Stmt,
229:                            SmallVectorImpl<MemoryAccess *> &RootAccs,
230:                            bool Local) {
231:   for (auto *MA : *Stmt) {
232:     if (!MA->isWrite())
233:       continue;
```
- **EN**: Executes an iterative step of the algorithm, usually scanning statements, dimensions, accesses, operands, or schedule nodes.
- **CN**: 这里执行算法中的迭代步骤，通常遍历语句、维度、访问、操作数或调度节点。

### Lines 235-235
```cpp
235:     // Writes to arrays are always used.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 236-237
```cpp
236:     if (MA->isLatestArrayKind())
237:       RootAccs.push_back(MA);
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 239-239
```cpp
239:     // Values are roots if they are escaping.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 240-243
```cpp
240:     else if (MA->isLatestValueKind()) {
241:       if (Local || isEscaping(MA))
242:         RootAccs.push_back(MA);
243:     }
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 245-245
```cpp
245:     // Exit phis are, by definition, escaping.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 246-247
```cpp
246:     else if (MA->isLatestExitPHIKind())
247:       RootAccs.push_back(MA);
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 249-250
```cpp
249:     // phi writes are only roots if we are not visiting the statement
250:     // containing the PHINode.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 251-252
```cpp
251:     else if (Local && MA->isLatestPHIKind())
252:       RootAccs.push_back(MA);
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 253-253
```cpp
253:   }
```
- **EN**: Closes the current block or scope and hands control back to the surrounding context.
- **CN**: 这里关闭当前代码块或作用域，并将控制流交还给外层上下文。

### Lines 254-254
```cpp
254: }
```
- **EN**: Closes the current block or scope and hands control back to the surrounding context.
- **CN**: 这里关闭当前代码块或作用域，并将控制流交还给外层上下文。

### Lines 256-256
```cpp
256: /// Determine all instruction and access roots.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 257-262
```cpp
257: static void addRoots(ScopStmt *Stmt,
258:                      SmallVectorImpl<VirtualInstruction> &RootInsts,
259:                      SmallVectorImpl<MemoryAccess *> &RootAccs, bool Local) {
260:   addInstructionRoots(Stmt, RootInsts);
261:   addAccessRoots(Stmt, RootAccs, Local);
262: }
```
- **EN**: Introduces or continues `addRoots`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `addRoots`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 264-275
```cpp
264: /// Mark accesses and instructions as used if they are reachable from a root,
265: /// walking the operand trees.
266: ///
267: /// @param S              The SCoP to walk.
268: /// @param LI             The LoopInfo Analysis.
269: /// @param RootInsts      List of root instructions.
270: /// @param RootAccs       List of root accesses.
271: /// @param UsesInsts[out] Receives all reachable instructions, including the
272: /// roots.
273: /// @param UsedAccs[out]  Receives all reachable accesses, including the roots.
274: /// @param OnlyLocal      If non-nullptr, restricts walking to a single
275: /// statement.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 276-283
```cpp
276: static void walkReachable(Scop *S, LoopInfo *LI,
277:                           ArrayRef<VirtualInstruction> RootInsts,
278:                           ArrayRef<MemoryAccess *> RootAccs,
279:                           DenseSet<VirtualInstruction> &UsedInsts,
280:                           DenseSet<MemoryAccess *> &UsedAccs,
281:                           ScopStmt *OnlyLocal = nullptr) {
282:   UsedInsts.clear();
283:   UsedAccs.clear();
```
- **EN**: Introduces or continues `walkReachable`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `walkReachable`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 285-285
```cpp
285:   SmallVector<VirtualInstruction, 32> WorklistInsts;
```
- **EN**: Carries supporting computation, data movement, or bookkeeping for the surrounding algorithm.
- **CN**: 这里承担周边算法所需的辅助计算、数据传递或簿记工作。

### Lines 286-286
```cpp
286:   SmallVector<MemoryAccess *, 32> WorklistAccs;
```
- **EN**: Carries supporting computation, data movement, or bookkeeping for the surrounding algorithm.
- **CN**: 这里承担周边算法所需的辅助计算、数据传递或簿记工作。

### Lines 288-288
```cpp
288:   WorklistInsts.append(RootInsts.begin(), RootInsts.end());
```
- **EN**: Introduces or continues `append`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `append`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 289-289
```cpp
289:   WorklistAccs.append(RootAccs.begin(), RootAccs.end());
```
- **EN**: Introduces or continues `append`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `append`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 291-313
```cpp
291:   auto AddToWorklist = [&](VirtualUse VUse) {
292:     switch (VUse.getKind()) {
293:     case VirtualUse::Block:
294:     case VirtualUse::Constant:
295:     case VirtualUse::Synthesizable:
296:     case VirtualUse::Hoisted:
297:       break;
298:     case VirtualUse::ReadOnly:
299:       // Read-only scalars only have MemoryAccesses if ModelReadOnlyScalars is
300:       // enabled.
301:       if (!VUse.getMemoryAccess())
302:         break;
303:       [[fallthrough]];
304:     case VirtualUse::Inter:
305:       assert(VUse.getMemoryAccess());
306:       WorklistAccs.push_back(VUse.getMemoryAccess());
307:       break;
308:     case VirtualUse::Intra:
309:       WorklistInsts.emplace_back(VUse.getUser(),
310:                                  cast<Instruction>(VUse.getValue()));
311:       break;
312:     }
313:   };
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 315-317
```cpp
315:   while (true) {
316:     // We have two worklists to process: Only when the MemoryAccess worklist is
317:     // empty, we process the instruction worklist.
```
- **EN**: Carries supporting computation, data movement, or bookkeeping for the surrounding algorithm.
- **CN**: 这里承担周边算法所需的辅助计算、数据传递或簿记工作。

### Lines 319-320
```cpp
319:     while (!WorklistAccs.empty()) {
320:       auto *Acc = WorklistAccs.pop_back_val();
```
- **EN**: Introduces or continues `pop_back_val`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `pop_back_val`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 322-322
```cpp
322:       ScopStmt *Stmt = Acc->getStatement();
```
- **EN**: Introduces or continues `getStatement`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `getStatement`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 323-324
```cpp
323:       if (OnlyLocal && Stmt != OnlyLocal)
324:         continue;
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 326-326
```cpp
326:       auto Inserted = UsedAccs.insert(Acc);
```
- **EN**: Introduces or continues `insert`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `insert`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 327-328
```cpp
327:       if (!Inserted.second)
328:         continue;
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 330-331
```cpp
330:       if (Acc->isRead()) {
331:         const ScopArrayInfo *SAI = Acc->getScopArrayInfo();
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 333-334
```cpp
333:         if (Acc->isLatestValueKind()) {
334:           MemoryAccess *DefAcc = S->getValueDef(SAI);
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 336-336
```cpp
336:           // Accesses to read-only values do not have a definition.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 337-338
```cpp
337:           if (DefAcc)
338:             WorklistAccs.push_back(S->getValueDef(SAI));
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 339-339
```cpp
339:         }
```
- **EN**: Closes the current block or scope and hands control back to the surrounding context.
- **CN**: 这里关闭当前代码块或作用域，并将控制流交还给外层上下文。

### Lines 341-344
```cpp
341:         if (Acc->isLatestAnyPHIKind()) {
342:           auto IncomingMAs = S->getPHIIncomings(SAI);
343:           WorklistAccs.append(IncomingMAs.begin(), IncomingMAs.end());
344:         }
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 345-345
```cpp
345:       }
```
- **EN**: Closes the current block or scope and hands control back to the surrounding context.
- **CN**: 这里关闭当前代码块或作用域，并将控制流交还给外层上下文。

### Lines 347-354
```cpp
347:       if (Acc->isWrite()) {
348:         if (Acc->isOriginalValueKind() ||
349:             (Acc->isOriginalArrayKind() && Acc->getAccessValue())) {
350:           Loop *Scope = Stmt->getSurroundingLoop();
351:           VirtualUse VUse =
352:               VirtualUse::create(S, Stmt, Scope, Acc->getAccessValue(), true);
353:           AddToWorklist(VUse);
354:         }
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 356-362
```cpp
356:         if (Acc->isOriginalAnyPHIKind()) {
357:           for (auto Incoming : Acc->getIncoming()) {
358:             VirtualUse VUse = VirtualUse::create(
359:                 S, Stmt, LI->getLoopFor(Incoming.first), Incoming.second, true);
360:             AddToWorklist(VUse);
361:           }
362:         }
```
- **EN**: Executes an iterative step of the algorithm, usually scanning statements, dimensions, accesses, operands, or schedule nodes.
- **CN**: 这里执行算法中的迭代步骤，通常遍历语句、维度、访问、操作数或调度节点。

### Lines 364-365
```cpp
364:         if (Acc->isOriginalArrayKind())
365:           WorklistInsts.emplace_back(Stmt, Acc->getAccessInstruction());
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 366-366
```cpp
366:       }
```
- **EN**: Closes the current block or scope and hands control back to the surrounding context.
- **CN**: 这里关闭当前代码块或作用域，并将控制流交还给外层上下文。

### Lines 367-367
```cpp
367:     }
```
- **EN**: Closes the current block or scope and hands control back to the surrounding context.
- **CN**: 这里关闭当前代码块或作用域，并将控制流交还给外层上下文。

### Lines 369-369
```cpp
369:     // If both worklists are empty, stop walking.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 370-371
```cpp
370:     if (WorklistInsts.empty())
371:       break;
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 373-373
```cpp
373:     VirtualInstruction VInst = WorklistInsts.pop_back_val();
```
- **EN**: Introduces or continues `pop_back_val`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `pop_back_val`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 374-374
```cpp
374:     ScopStmt *Stmt = VInst.getStmt();
```
- **EN**: Introduces or continues `getStmt`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `getStmt`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 375-375
```cpp
375:     Instruction *Inst = VInst.getInstruction();
```
- **EN**: Introduces or continues `getInstruction`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `getInstruction`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 377-377
```cpp
377:     // Do not process statements other than the local.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 378-379
```cpp
378:     if (OnlyLocal && Stmt != OnlyLocal)
379:       continue;
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 381-381
```cpp
381:     auto InsertResult = UsedInsts.insert(VInst);
```
- **EN**: Introduces or continues `insert`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `insert`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 382-383
```cpp
382:     if (!InsertResult.second)
383:       continue;
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 385-385
```cpp
385:     // Add all operands to the worklists.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 386-386
```cpp
386:     PHINode *PHI = dyn_cast<PHINode>(Inst);
```
- **EN**: Introduces or continues `dyn_cast<PHINode>`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `dyn_cast<PHINode>`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 387-393
```cpp
387:     if (PHI && PHI->getParent() == Stmt->getEntryBlock()) {
388:       if (MemoryAccess *PHIRead = Stmt->lookupPHIReadOf(PHI))
389:         WorklistAccs.push_back(PHIRead);
390:     } else {
391:       for (VirtualUse VUse : VInst.operands())
392:         AddToWorklist(VUse);
393:     }
```
- **EN**: Executes an iterative step of the algorithm, usually scanning statements, dimensions, accesses, operands, or schedule nodes.
- **CN**: 这里执行算法中的迭代步骤，通常遍历语句、维度、访问、操作数或调度节点。

### Lines 395-395
```cpp
395:     // If there is an array access, also add its MemoryAccesses to the worklist.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 396-396
```cpp
396:     const MemoryAccessList *Accs = Stmt->lookupArrayAccessesFor(Inst);
```
- **EN**: Introduces or continues `lookupArrayAccessesFor`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `lookupArrayAccessesFor`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 397-398
```cpp
397:     if (!Accs)
398:       continue;
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 400-401
```cpp
400:     for (MemoryAccess *Acc : *Accs)
401:       WorklistAccs.push_back(Acc);
```
- **EN**: Executes an iterative step of the algorithm, usually scanning statements, dimensions, accesses, operands, or schedule nodes.
- **CN**: 这里执行算法中的迭代步骤，通常遍历语句、维度、访问、操作数或调度节点。

### Lines 402-402
```cpp
402:   }
```
- **EN**: Closes the current block or scope and hands control back to the surrounding context.
- **CN**: 这里关闭当前代码块或作用域，并将控制流交还给外层上下文。

### Lines 403-403
```cpp
403: }
```
- **EN**: Closes the current block or scope and hands control back to the surrounding context.
- **CN**: 这里关闭当前代码块或作用域，并将控制流交还给外层上下文。

### Lines 405-410
```cpp
405: void polly::markReachable(Scop *S, LoopInfo *LI,
406:                           DenseSet<VirtualInstruction> &UsedInsts,
407:                           DenseSet<MemoryAccess *> &UsedAccs,
408:                           ScopStmt *OnlyLocal) {
409:   SmallVector<VirtualInstruction, 32> RootInsts;
410:   SmallVector<MemoryAccess *, 32> RootAccs;
```
- **EN**: Introduces or continues `polly::markReachable`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `polly::markReachable`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 412-417
```cpp
412:   if (OnlyLocal) {
413:     addRoots(OnlyLocal, RootInsts, RootAccs, true);
414:   } else {
415:     for (auto &Stmt : *S)
416:       addRoots(&Stmt, RootInsts, RootAccs, false);
417:   }
```
- **EN**: Executes an iterative step of the algorithm, usually scanning statements, dimensions, accesses, operands, or schedule nodes.
- **CN**: 这里执行算法中的迭代步骤，通常遍历语句、维度、访问、操作数或调度节点。

### Lines 419-419
```cpp
419:   walkReachable(S, LI, RootInsts, RootAccs, UsedInsts, UsedAccs, OnlyLocal);
```
- **EN**: Introduces or continues `walkReachable`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `walkReachable`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 420-420
```cpp
420: }
```
- **EN**: Closes the current block or scope and hands control back to the surrounding context.
- **CN**: 这里关闭当前代码块或作用域，并将控制流交还给外层上下文。

## Key Concepts / 关键概念

- **Virtual instruction model** / **虚拟指令模型**
- **Scalar reasoning** / **标量推理**
- **Instruction abstraction** / **指令抽象**
- **ScalarEvolution reasoning** / **ScalarEvolution 推理**
- **Memory/access reasoning** / **内存/访问推理**

## Dependencies / 依赖关系

- **EN**: Polly headers such as polly/Support/VirtualInstruction.h
- **CN**: Polly 头文件，例如 polly/Support/VirtualInstruction.h
