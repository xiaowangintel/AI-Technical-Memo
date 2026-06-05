# DXILPrepare.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/DirectX/DXILPrepare.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file contains passes and utilities to convert a modern LLVM module into a module compatible with the LLVM 3.7-based DirectX Intermediate Language (DXIL).
- 目的（中文）: 该文件用于支撑 LLVM 目标后端的相关功能。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-40
```cpp
 1: //===- DXILPrepare.cpp - Prepare LLVM Module for DXIL encoding ------------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: ///
 9: /// \file This file contains passes and utilities to convert a modern LLVM
10: /// module into a module compatible with the LLVM 3.7-based DirectX Intermediate
11: /// Language (DXIL).
12: //===----------------------------------------------------------------------===//
13:
14: #include "DXILRootSignature.h"
15: #include "DXILShaderFlags.h"
16: #include "DirectX.h"
17: #include "DirectXIRPasses/PointerTypeAnalysis.h"
18: #include "llvm/ADT/STLExtras.h"
19: #include "llvm/ADT/StringSet.h"
20: #include "llvm/Analysis/DXILMetadataAnalysis.h"
21: #include "llvm/Analysis/DXILResource.h"
22: #include "llvm/CodeGen/Passes.h"
23: #include "llvm/IR/AttributeMask.h"
24: #include "llvm/IR/IRBuilder.h"
25: #include "llvm/IR/Instruction.h"
26: #include "llvm/IR/Module.h"
27: #include "llvm/InitializePasses.h"
28: #include "llvm/Pass.h"
29: #include "llvm/Support/VersionTuple.h"
30:
31: #define DEBUG_TYPE "dxil-prepare"
32:
33: using namespace llvm;
34: using namespace llvm::dxil;
35:
36: namespace {
37:
38: constexpr bool isValidForDXIL(Attribute::AttrKind Attr) {
39:   return is_contained({Attribute::Alignment,
40:                        Attribute::AlwaysInline,
```
- EN: This range implements operational logic in helpers such as isValidForDXIL, translating backend policy into executable code.
- CN: 这一段实现了 isValidForDXIL 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 41-80
```cpp
41:                        Attribute::Builtin,
42:                        Attribute::ByVal,
43:                        Attribute::InAlloca,
44:                        Attribute::Cold,
45:                        Attribute::Convergent,
46:                        Attribute::InlineHint,
47:                        Attribute::InReg,
48:                        Attribute::JumpTable,
49:                        Attribute::MinSize,
50:                        Attribute::Naked,
51:                        Attribute::Nest,
52:                        Attribute::NoAlias,
53:                        Attribute::NoBuiltin,
54:                        Attribute::NoDuplicate,
55:                        Attribute::NoImplicitFloat,
56:                        Attribute::NoInline,
57:                        Attribute::NonLazyBind,
58:                        Attribute::NonNull,
59:                        Attribute::Dereferenceable,
60:                        Attribute::DereferenceableOrNull,
61:                        Attribute::Memory,
62:                        Attribute::NoRedZone,
63:                        Attribute::NoReturn,
64:                        Attribute::NoUnwind,
65:                        Attribute::OptimizeForSize,
66:                        Attribute::OptimizeNone,
67:                        Attribute::ReadNone,
68:                        Attribute::ReadOnly,
69:                        Attribute::Returned,
70:                        Attribute::ReturnsTwice,
71:                        Attribute::SExt,
72:                        Attribute::StackAlignment,
73:                        Attribute::StackProtect,
74:                        Attribute::StackProtectReq,
75:                        Attribute::StackProtectStrong,
76:                        Attribute::SafeStack,
77:                        Attribute::StructRet,
78:                        Attribute::SanitizeAddress,
79:                        Attribute::SanitizeThread,
80:                        Attribute::SanitizeMemory,
```
- EN: This range contains connective backend logic, including helper statements, control flow, and data movement needed by the surrounding implementation.
- CN: 这一段包含衔接性的后端逻辑，包括辅助语句、控制流和数据处理，用于支撑周围实现。

### Lines 81-120
```cpp
 81:                        Attribute::UWTable,
 82:                        Attribute::ZExt},
 83:                       Attr);
 84: }
 85:
 86: static void collectDeadStringAttrs(AttributeMask &DeadAttrs, AttributeSet &&AS,
 87:                                    const StringSet<> &LiveKeys,
 88:                                    bool AllowExperimental) {
 89:   for (auto &Attr : AS) {
 90:     if (!Attr.isStringAttribute())
 91:       continue;
 92:     StringRef Key = Attr.getKindAsString();
 93:     if (LiveKeys.contains(Key))
 94:       continue;
 95:     if (AllowExperimental && Key.starts_with("exp-"))
 96:       continue;
 97:     DeadAttrs.addAttribute(Key);
 98:   }
 99: }
100:
101: static void removeStringFunctionAttributes(Function &F,
102:                                            bool AllowExperimental) {
103:   AttributeList Attrs = F.getAttributes();
104:   const StringSet<> LiveKeys = {"waveops-include-helper-lanes",
105:                                 "fp32-denorm-mode"};
106:   // Collect DeadKeys in FnAttrs.
107:   AttributeMask DeadAttrs;
108:   collectDeadStringAttrs(DeadAttrs, Attrs.getFnAttrs(), LiveKeys,
109:                          AllowExperimental);
110:   collectDeadStringAttrs(DeadAttrs, Attrs.getRetAttrs(), LiveKeys,
111:                          AllowExperimental);
112:
113:   F.removeFnAttrs(DeadAttrs);
114:   F.removeRetAttrs(DeadAttrs);
115: }
116:
117: class DXILPrepareModule : public ModulePass {
118:
119:   static Value *maybeGenerateBitcast(IRBuilder<> &Builder,
120:                                      PointerTypeMap &PointerTypes,
```
- EN: This range defines or declares important types such as getKindAsString, addAttribute, getAttributes, removeFnAttrs, shaping the data model used by DXILPrepare.cpp.
- CN: 这一段定义或声明了 getKindAsString、addAttribute、getAttributes、removeFnAttrs 等关键类型，构成 DXILPrepare.cpp 使用的数据模型。

### Lines 121-160
```cpp
121:                                      Instruction &Inst, Value *Operand,
122:                                      Type *Ty) {
123:     // Omit bitcasts if the incoming value matches the instruction type.
124:     auto It = PointerTypes.find(Operand);
125:     if (It != PointerTypes.end()) {
126:       auto *OpTy = cast<TypedPointerType>(It->second)->getElementType();
127:       if (OpTy == Ty)
128:         return nullptr;
129:     }
130:
131:     Type *ValTy = Operand->getType();
132:     // Also omit the bitcast for matching global array types
133:     if (auto *GlobalVar = dyn_cast<GlobalVariable>(Operand))
134:       ValTy = GlobalVar->getValueType();
135:
136:     if (auto *AI = dyn_cast<AllocaInst>(Operand))
137:       ValTy = AI->getAllocatedType();
138:
139:     if (auto *ArrTy = dyn_cast<ArrayType>(ValTy)) {
140:       Type *ElTy = ArrTy->getElementType();
141:       if (ElTy == Ty)
142:         return nullptr;
143:     }
144:
145:     // finally, drill down GEP instructions until we get the array
146:     // that is being accessed, and compare element types
147:     if (ConstantExpr *GEPInstr = dyn_cast<ConstantExpr>(Operand)) {
148:       while (GEPInstr->getOpcode() == Instruction::GetElementPtr) {
149:         Value *OpArg = GEPInstr->getOperand(0);
150:         if (ConstantExpr *NewGEPInstr = dyn_cast<ConstantExpr>(OpArg)) {
151:           GEPInstr = NewGEPInstr;
152:           continue;
153:         }
154:
155:         if (auto *GlobalVar = dyn_cast<GlobalVariable>(OpArg))
156:           ValTy = GlobalVar->getValueType();
157:         if (auto *AI = dyn_cast<AllocaInst>(Operand))
158:           ValTy = AI->getAllocatedType();
159:         if (auto *ArrTy = dyn_cast<ArrayType>(ValTy)) {
160:           Type *ElTy = ArrTy->getElementType();
```
- EN: This range implements operational logic in helpers such as find, getElementType, getType, getValueType, translating backend policy into executable code.
- CN: 这一段实现了 find、getElementType、getType、getValueType 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 161-200
```cpp
161:           if (ElTy == Ty)
162:             return nullptr;
163:         }
164:         break;
165:       }
166:     }
167:
168:     // Insert bitcasts where we are removing the instruction.
169:     Builder.SetInsertPoint(&Inst);
170:     // This code only gets hit in opaque-pointer mode, so the type of the
171:     // pointer doesn't matter.
172:     PointerType *PtrTy = cast<PointerType>(Operand->getType());
173:     return Builder.Insert(
174:         CastInst::Create(Instruction::BitCast, Operand,
175:                          Builder.getPtrTy(PtrTy->getAddressSpace())));
176:   }
177:
178: public:
179:   bool runOnModule(Module &M) override {
180:     M.convertFromNewDbgValues();
181:
182:     PointerTypeMap PointerTypes = PointerTypeAnalysis::run(M);
183:     AttributeMask AttrMask;
184:     for (Attribute::AttrKind I = Attribute::None; I != Attribute::EndAttrKinds;
185:          I = Attribute::AttrKind(I + 1)) {
186:       if (!isValidForDXIL(I))
187:         AttrMask.addAttribute(I);
188:     }
189:
190:     const dxil::ModuleMetadataInfo MetadataInfo =
191:         getAnalysis<DXILMetadataAnalysisWrapperPass>().getModuleMetadata();
192:     VersionTuple ValVer = MetadataInfo.ValidatorVersion;
193:     bool AllowExperimental = ValVer.getMajor() == 0 && ValVer.getMinor() == 0;
194:
195:     for (auto &F : M.functions()) {
196:       F.removeFnAttrs(AttrMask);
197:       F.removeRetAttrs(AttrMask);
198:       // Only remove string attributes if we are not skipping validation.
199:       // This will reserve the experimental attributes when validation version
200:       // is 0.0 for experiment mode.
```
- EN: This range implements operational logic in helpers such as SetInsertPoint, getType, getPtrTy, runOnModule, translating backend policy into executable code.
- CN: 这一段实现了 SetInsertPoint、getType、getPtrTy、runOnModule 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 201-240
```cpp
201:       removeStringFunctionAttributes(F, AllowExperimental);
202:       for (size_t Idx = 0, End = F.arg_size(); Idx < End; ++Idx)
203:         F.removeParamAttrs(Idx, AttrMask);
204:
205:       for (auto &BB : F) {
206:         IRBuilder<> Builder(&BB);
207:         for (auto &I : make_early_inc_range(BB)) {
208:
209:           if (auto *CB = dyn_cast<CallBase>(&I)) {
210:             CB->removeFnAttrs(AttrMask);
211:             CB->removeRetAttrs(AttrMask);
212:             for (size_t Idx = 0, End = CB->arg_size(); Idx < End; ++Idx)
213:               CB->removeParamAttrs(Idx, AttrMask);
214:             continue;
215:           }
216:
217:           // Emtting NoOp bitcast instructions allows the ValueEnumerator to be
218:           // unmodified as it reserves instruction IDs during contruction.
219:           if (auto *LI = dyn_cast<LoadInst>(&I)) {
220:             if (Value *NoOpBitcast = maybeGenerateBitcast(
221:                     Builder, PointerTypes, I, LI->getPointerOperand(),
222:                     LI->getType())) {
223:               LI->replaceAllUsesWith(
224:                   Builder.CreateLoad(LI->getType(), NoOpBitcast));
225:               LI->eraseFromParent();
226:             }
227:             continue;
228:           }
229:           if (auto *SI = dyn_cast<StoreInst>(&I)) {
230:             if (Value *NoOpBitcast = maybeGenerateBitcast(
231:                     Builder, PointerTypes, I, SI->getPointerOperand(),
232:                     SI->getValueOperand()->getType())) {
233:
234:               SI->replaceAllUsesWith(
235:                   Builder.CreateStore(SI->getValueOperand(), NoOpBitcast));
236:               SI->eraseFromParent();
237:             }
238:             continue;
239:           }
240:           if (auto *GEP = dyn_cast<GetElementPtrInst>(&I)) {
```
- EN: This range implements operational logic in helpers such as removeStringFunctionAttributes, removeParamAttrs, Builder, removeFnAttrs, translating backend policy into executable code.
- CN: 这一段实现了 removeStringFunctionAttributes、removeParamAttrs、Builder、removeFnAttrs 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 241-277
```cpp
241:             if (Value *NoOpBitcast = maybeGenerateBitcast(
242:                     Builder, PointerTypes, I, GEP->getPointerOperand(),
243:                     GEP->getSourceElementType()))
244:               GEP->setOperand(0, NoOpBitcast);
245:             continue;
246:           }
247:         }
248:       }
249:     }
250:
251:     return true;
252:   }
253:
254:   DXILPrepareModule() : ModulePass(ID) {}
255:   void getAnalysisUsage(AnalysisUsage &AU) const override {
256:     AU.addRequired<DXILMetadataAnalysisWrapperPass>();
257:
258:     AU.addPreserved<DXILMetadataAnalysisWrapperPass>();
259:     AU.addPreserved<DXILResourceWrapperPass>();
260:     AU.addPreserved<RootSignatureAnalysisWrapper>();
261:     AU.addPreserved<ShaderFlagsAnalysisWrapper>();
262:   }
263:   static char ID; // Pass identification.
264: };
265: char DXILPrepareModule::ID = 0;
266:
267: } // end anonymous namespace
268:
269: INITIALIZE_PASS_BEGIN(DXILPrepareModule, DEBUG_TYPE, "DXIL Prepare Module",
270:                       false, false)
271: INITIALIZE_PASS_DEPENDENCY(DXILMetadataAnalysisWrapperPass)
272: INITIALIZE_PASS_END(DXILPrepareModule, DEBUG_TYPE, "DXIL Prepare Module", false,
273:                     false)
274:
275: ModulePass *llvm::createDXILPrepareModulePass() {
276:   return new DXILPrepareModule();
277: }
```
- EN: This range implements operational logic in helpers such as getSourceElementType, setOperand, DXILPrepareModule, getAnalysisUsage, translating backend policy into executable code.
- CN: 这一段实现了 getSourceElementType、setOperand、DXILPrepareModule、getAnalysisUsage 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: DirectX/DXIL backend code models HLSL/DXIL semantics, including resources, shader stages, and validation-friendly IR forms.
  - CN: DirectX/DXIL 后端代码建模了 HLSL/DXIL 语义，包括资源、着色器阶段以及便于验证的 IR 形式。
- EN: Key symbols in this file include isValidForDXIL, getKindAsString, addAttribute, getAttributes, removeFnAttrs, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 isValidForDXIL, getKindAsString, addAttribute, getAttributes, removeFnAttrs，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `DXILRootSignature.h`
  - `DXILShaderFlags.h`
  - `DirectX.h`
  - `DirectXIRPasses/PointerTypeAnalysis.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/ADT/STLExtras.h`
  - `llvm/ADT/StringSet.h`
  - `llvm/Analysis/DXILMetadataAnalysis.h`
  - `llvm/Analysis/DXILResource.h`
  - `llvm/CodeGen/Passes.h`
  - `llvm/IR/AttributeMask.h`
  - `llvm/IR/IRBuilder.h`
  - `llvm/IR/Instruction.h`
  - `llvm/IR/Module.h`
  - `llvm/InitializePasses.h`
  - `llvm/Pass.h`
  - `llvm/Support/VersionTuple.h`
