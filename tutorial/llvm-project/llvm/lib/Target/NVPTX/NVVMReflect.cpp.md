# NVVMReflect.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/NVPTX/NVVMReflect.cpp`
- Repository: `llvm-project`
- Purpose (EN): This pass replaces occurrences of __nvvm_reflect("foo") and llvm.nvvm.reflect with an integer.
- 目的（中文）: 该文件用于支撑 LLVM 目标后端的相关功能。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-40
```cpp
 1: //===- NVVMReflect.cpp - NVVM Emulate conditional compilation -------------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // This pass replaces occurrences of __nvvm_reflect("foo") and llvm.nvvm.reflect
10: // with an integer.
11: //
12: // We choose the value we use by looking at metadata in the module itself.  Note
13: // that we intentionally only have one way to choose these values, because other
14: // parts of LLVM (particularly, InstCombineCall) rely on being able to predict
15: // the values chosen by this pass.
16: //
17: // If we see an unknown string, we replace its call with 0.
18: //
19: //===----------------------------------------------------------------------===//
20:
21: #include "NVPTX.h"
22: #include "llvm/ADT/SmallVector.h"
23: #include "llvm/ADT/StringExtras.h"
24: #include "llvm/Analysis/ConstantFolding.h"
25: #include "llvm/CodeGen/CommandFlags.h"
26: #include "llvm/IR/Constants.h"
27: #include "llvm/IR/DerivedTypes.h"
28: #include "llvm/IR/Function.h"
29: #include "llvm/IR/Instructions.h"
30: #include "llvm/IR/Intrinsics.h"
31: #include "llvm/IR/IntrinsicsNVPTX.h"
32: #include "llvm/IR/Module.h"
33: #include "llvm/IR/PassManager.h"
34: #include "llvm/IR/Type.h"
35: #include "llvm/Pass.h"
36: #include "llvm/Support/CommandLine.h"
37: #include "llvm/Support/Debug.h"
38: #include "llvm/Support/raw_ostream.h"
39: #include "llvm/Transforms/Scalar.h"
40: #include "llvm/Transforms/Utils/BasicBlockUtils.h"
```
- EN: This range pulls in local backend headers and core LLVM infrastructure needed by the rest of the file.
- CN: 这一段引入本地后端头文件以及后续实现所依赖的 LLVM 基础设施。

### Lines 41-80
```cpp
41: #include "llvm/Transforms/Utils/Local.h"
42: #define NVVM_REFLECT_FUNCTION "__nvvm_reflect"
43: #define NVVM_REFLECT_OCL_FUNCTION "__nvvm_reflect_ocl"
44: // Argument of reflect call to retrive arch number
45: #define CUDA_ARCH_NAME "__CUDA_ARCH"
46: // Argument of reflect call to retrive ftz mode
47: #define CUDA_FTZ_NAME "__CUDA_FTZ"
48: // Name of module metadata where ftz mode is stored
49: #define CUDA_FTZ_MODULE_NAME "nvvm-reflect-ftz"
50:
51: using namespace llvm;
52:
53: #define DEBUG_TYPE "nvvm-reflect"
54:
55: namespace {
56: class NVVMReflect {
57:   // Map from reflect function call arguments to the value to replace the call
58:   // with. Should include __CUDA_FTZ and __CUDA_ARCH values.
59:   StringMap<unsigned> ReflectMap;
60:   bool handleReflectFunction(Module &M, StringRef ReflectName);
61:   void populateReflectMap(Module &M);
62:   void foldReflectCall(CallInst *Call, Constant *NewValue);
63:
64: public:
65:   // __CUDA_FTZ is assigned in `runOnModule` by checking nvvm-reflect-ftz module
66:   // metadata.
67:   explicit NVVMReflect(unsigned SmVersion)
68:       : ReflectMap({{CUDA_ARCH_NAME, SmVersion * 10}}) {}
69:   bool runOnModule(Module &M);
70: };
71:
72: class NVVMReflectLegacyPass : public ModulePass {
73:   NVVMReflect Impl;
74:
75: public:
76:   static char ID;
77:   NVVMReflectLegacyPass(unsigned SmVersion) : ModulePass(ID), Impl(SmVersion) {}
78:   bool runOnModule(Module &M) override;
79: };
80: } // namespace
```
- EN: This range defines or declares important types such as NVVMReflect, handleReflectFunction, populateReflectMap, foldReflectCall, shaping the data model used by NVVMReflect.cpp.
- CN: 这一段定义或声明了 NVVMReflect、handleReflectFunction、populateReflectMap、foldReflectCall 等关键类型，构成 NVVMReflect.cpp 使用的数据模型。

### Lines 81-120
```cpp
 81:
 82: ModulePass *llvm::createNVVMReflectPass(unsigned SmVersion) {
 83:   return new NVVMReflectLegacyPass(SmVersion);
 84: }
 85:
 86: static cl::opt<bool>
 87:     NVVMReflectEnabled("nvvm-reflect-enable", cl::init(true), cl::Hidden,
 88:                        cl::desc("NVVM reflection, enabled by default"));
 89:
 90: char NVVMReflectLegacyPass::ID = 0;
 91: INITIALIZE_PASS(NVVMReflectLegacyPass, "nvvm-reflect",
 92:                 "Replace occurrences of __nvvm_reflect() calls with 0/1", false,
 93:                 false)
 94:
 95: // Allow users to specify additional key/value pairs to reflect. These key/value
 96: // pairs are the last to be added to the ReflectMap, and therefore will take
 97: // precedence over initial values (i.e. __CUDA_FTZ from module medadata and
 98: // __CUDA_ARCH from SmVersion).
 99: static cl::list<std::string> ReflectList(
100:     "nvvm-reflect-add", cl::value_desc("name=<int>"), cl::Hidden,
101:     cl::desc("A key=value pair. Replace __nvvm_reflect(name) with value."),
102:     cl::ValueRequired);
103:
104: // Set the ReflectMap with, first, the value of __CUDA_FTZ from module metadata,
105: // and then the key/value pairs from the command line.
106: void NVVMReflect::populateReflectMap(Module &M) {
107:   if (auto *Flag = mdconst::extract_or_null<ConstantInt>(
108:           M.getModuleFlag(CUDA_FTZ_MODULE_NAME)))
109:     ReflectMap[CUDA_FTZ_NAME] = Flag->getSExtValue();
110:
111:   for (auto &Option : ReflectList) {
112:     LLVM_DEBUG(dbgs() << "ReflectOption : " << Option << "\n");
113:     StringRef OptionRef(Option);
114:     auto [Name, Val] = OptionRef.split('=');
115:     if (Name.empty())
116:       report_fatal_error(Twine("Empty name in nvvm-reflect-add option '") +
117:                          Option + "'");
118:     if (Val.empty())
119:       report_fatal_error(Twine("Missing value in nvvm-reflect-add option '") +
120:                          Option + "'");
```
- EN: This range defines command-line tuning knobs that influence backend lowering, code generation, or diagnostics.
- CN: 这一段定义命令行调优选项，用来影响后端的降级、代码生成或诊断行为。

### Lines 121-160
```cpp
121:     unsigned ValInt;
122:     if (!to_integer(Val.trim(), ValInt, 10))
123:       report_fatal_error(
124:           Twine("integer value expected in nvvm-reflect-add option '") +
125:           Option + "'");
126:     ReflectMap[Name] = ValInt;
127:   }
128: }
129:
130: /// Process a reflect function by finding all its calls and replacing them with
131: /// appropriate constant values. For __CUDA_FTZ, uses the module flag value.
132: /// For __CUDA_ARCH, uses SmVersion * 10. For all other strings, uses 0.
133: bool NVVMReflect::handleReflectFunction(Module &M, StringRef ReflectName) {
134:   Function *F = M.getFunction(ReflectName);
135:   if (!F)
136:     return false;
137:   assert(F->isDeclaration() && "_reflect function should not have a body");
138:   assert(F->getReturnType()->isIntegerTy() &&
139:          "_reflect's return type should be integer");
140:
141:   const bool Changed = !F->use_empty();
142:   for (User *U : make_early_inc_range(F->users())) {
143:     // Reflect function calls look like:
144:     // @arch = private unnamed_addr addrspace(1) constant [12 x i8]
145:     // c"__CUDA_ARCH\00" call i32 @__nvvm_reflect(ptr addrspacecast (ptr
146:     // addrspace(1) @arch to ptr)) We need to extract the string argument from
147:     // the call (i.e. "__CUDA_ARCH")
148:     auto *Call = dyn_cast<CallInst>(U);
149:     if (!Call)
150:       report_fatal_error(
151:           "__nvvm_reflect can only be used in a call instruction");
152:     if (Call->getNumOperands() != 2)
153:       report_fatal_error("__nvvm_reflect requires exactly one argument");
154:
155:     auto *GlobalStr =
156:         dyn_cast<Constant>(Call->getArgOperand(0)->stripPointerCasts());
157:     if (!GlobalStr)
158:       report_fatal_error("__nvvm_reflect argument must be a constant string");
159:
160:     auto *ConstantStr =
```
- EN: This range implements operational logic in helpers such as NVVMReflect::handleReflectFunction, getFunction, assert, use_empty, translating backend policy into executable code.
- CN: 这一段实现了 NVVMReflect::handleReflectFunction、getFunction、assert、use_empty 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 161-200
```cpp
161:         dyn_cast<ConstantDataSequential>(GlobalStr->getOperand(0));
162:     if (!ConstantStr)
163:       report_fatal_error("__nvvm_reflect argument must be a string constant");
164:     if (!ConstantStr->isCString())
165:       report_fatal_error(
166:           "__nvvm_reflect argument must be a null-terminated string");
167:
168:     StringRef ReflectArg = ConstantStr->getAsString().drop_back();
169:     if (ReflectArg.empty())
170:       report_fatal_error("__nvvm_reflect argument cannot be empty");
171:     // Now that we have extracted the string argument, we can look it up in the
172:     // ReflectMap
173:     unsigned ReflectVal = 0; // The default value is 0
174:     if (ReflectMap.contains(ReflectArg))
175:       ReflectVal = ReflectMap[ReflectArg];
176:
177:     LLVM_DEBUG(dbgs() << "Replacing call of reflect function " << F->getName()
178:                       << "(" << ReflectArg << ") with value " << ReflectVal
179:                       << "\n");
180:     auto *NewValue = ConstantInt::get(Call->getType(), ReflectVal);
181:     foldReflectCall(Call, NewValue);
182:     Call->eraseFromParent();
183:   }
184:
185:   // Remove the __nvvm_reflect function from the module
186:   F->eraseFromParent();
187:   return Changed;
188: }
189:
190: void NVVMReflect::foldReflectCall(CallInst *Call, Constant *NewValue) {
191:   SmallVector<Instruction *, 8> Worklist;
192:   // Replace an instruction with a constant and add all users of the instruction
193:   // to the worklist
194:   auto ReplaceInstructionWithConst = [&](Instruction *I, Constant *C) {
195:     for (auto *U : I->users())
196:       if (auto *UI = dyn_cast<Instruction>(U))
197:         Worklist.push_back(UI);
198:     I->replaceAllUsesWith(C);
199:   };
200:
```
- EN: This range implements operational logic in helpers such as getOperand, report_fatal_error, getAsString, LLVM_DEBUG, translating backend policy into executable code.
- CN: 这一段实现了 getOperand、report_fatal_error、getAsString、LLVM_DEBUG 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 201-235
```cpp
201:   ReplaceInstructionWithConst(Call, NewValue);
202:
203:   auto &DL = Call->getModule()->getDataLayout();
204:   while (!Worklist.empty()) {
205:     auto *I = Worklist.pop_back_val();
206:     if (auto *C = ConstantFoldInstruction(I, DL)) {
207:       ReplaceInstructionWithConst(I, C);
208:       if (isInstructionTriviallyDead(I))
209:         I->eraseFromParent();
210:     } else if (I->isTerminator()) {
211:       ConstantFoldTerminator(I->getParent());
212:     }
213:   }
214: }
215:
216: bool NVVMReflect::runOnModule(Module &M) {
217:   if (!NVVMReflectEnabled)
218:     return false;
219:   populateReflectMap(M);
220:   bool Changed = true;
221:   Changed |= handleReflectFunction(M, NVVM_REFLECT_FUNCTION);
222:   Changed |= handleReflectFunction(M, NVVM_REFLECT_OCL_FUNCTION);
223:   Changed |=
224:       handleReflectFunction(M, Intrinsic::getName(Intrinsic::nvvm_reflect));
225:   return Changed;
226: }
227:
228: bool NVVMReflectLegacyPass::runOnModule(Module &M) {
229:   return Impl.runOnModule(M);
230: }
231:
232: PreservedAnalyses NVVMReflectPass::run(Module &M, ModuleAnalysisManager &AM) {
233:   return NVVMReflect(SmVersion).runOnModule(M) ? PreservedAnalyses::none()
234:                                                : PreservedAnalyses::all();
235: }
```
- EN: This range implements operational logic in helpers such as ReplaceInstructionWithConst, getModule, pop_back_val, eraseFromParent, translating backend policy into executable code.
- CN: 这一段实现了 ReplaceInstructionWithConst、getModule、pop_back_val、eraseFromParent 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: NVPTX backend code reflects GPU execution concerns such as address spaces, intrinsics, and PTX-specific lowering.
  - CN: NVPTX 后端代码体现了 GPU 执行特性，例如地址空间、内建函数以及 PTX 特定降级。
- EN: Key symbols in this file include NVVMReflect, handleReflectFunction, populateReflectMap, foldReflectCall, ReflectMap, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 NVVMReflect, handleReflectFunction, populateReflectMap, foldReflectCall, ReflectMap，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `NVPTX.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/ADT/SmallVector.h`
  - `llvm/ADT/StringExtras.h`
  - `llvm/Analysis/ConstantFolding.h`
  - `llvm/CodeGen/CommandFlags.h`
  - `llvm/IR/Constants.h`
  - `llvm/IR/DerivedTypes.h`
  - `llvm/IR/Function.h`
  - `llvm/IR/Instructions.h`
  - `llvm/IR/Intrinsics.h`
  - `llvm/IR/IntrinsicsNVPTX.h`
  - `llvm/IR/Module.h`
  - `llvm/IR/PassManager.h`
  - `llvm/IR/Type.h`
  - `llvm/Pass.h`
  - `llvm/Support/CommandLine.h`
  - `llvm/Support/Debug.h`
