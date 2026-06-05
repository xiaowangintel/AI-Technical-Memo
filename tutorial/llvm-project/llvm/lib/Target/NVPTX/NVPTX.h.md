# NVPTX.h — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/NVPTX/NVPTX.h`
- Repository: `llvm-project`
- Purpose (EN): This file contains the entry points for global functions defined in the LLVM NVPTX back-end.
- 目的（中文）: 该文件声明 LLVM 目标后端使用的数据结构、接口或辅助类型。
- Language: C++ header

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-40
```cpp
 1: //===-- NVPTX.h - Top-level interface for NVPTX representation --*- C++ -*-===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // This file contains the entry points for global functions defined in
10: // the LLVM NVPTX back-end.
11: //
12: //===----------------------------------------------------------------------===//
13:
14: #ifndef LLVM_LIB_TARGET_NVPTX_NVPTX_H
15: #define LLVM_LIB_TARGET_NVPTX_NVPTX_H
16:
17: #include "llvm/IR/PassManager.h"
18: #include "llvm/Pass.h"
19: #include "llvm/Support/AtomicOrdering.h"
20: #include "llvm/Support/CodeGen.h"
21: #include "llvm/Support/NVPTXAddrSpace.h"
22: #include "llvm/Target/TargetMachine.h"
23:
24: namespace llvm {
25: class FunctionPass;
26: class MachineFunctionPass;
27: class NVPTXTargetMachine;
28: class PassRegistry;
29:
30: namespace NVPTXCC {
31: enum CondCodes {
32:   EQ,
33:   NE,
34:   LT,
35:   LE,
36:   GT,
37:   GE
38: };
39: }
40:
```
- EN: This range defines or declares important types such as FunctionPass, MachineFunctionPass, NVPTXTargetMachine, PassRegistry, shaping the data model used by NVPTX.h.
- CN: 这一段定义或声明了 FunctionPass、MachineFunctionPass、NVPTXTargetMachine、PassRegistry 等关键类型，构成 NVPTX.h 使用的数据模型。

### Lines 41-80
```cpp
41: FunctionPass *createNVPTXISelDag(NVPTXTargetMachine &TM,
42:                                  llvm::CodeGenOptLevel OptLevel);
43: ModulePass *createNVPTXAssignValidGlobalNamesPass();
44: ModulePass *createGenericToNVVMLegacyPass();
45: ModulePass *createNVPTXCtorDtorLoweringLegacyPass();
46: FunctionPass *createNVVMIntrRangePass();
47: ModulePass *createNVVMReflectPass(unsigned int SmVersion);
48: MachineFunctionPass *createNVPTXPrologEpilogPass();
49: MachineFunctionPass *createNVPTXReplaceImageHandlesPass();
50: FunctionPass *createNVPTXImageOptimizerPass();
51: FunctionPass *createNVPTXLowerArgsPass();
52: FunctionPass *createNVPTXSetByValParamAlignPass();
53: FunctionPass *createNVPTXLowerAllocaPass();
54: FunctionPass *createNVPTXLowerUnreachablePass(bool TrapUnreachable,
55:                                               bool NoTrapAfterNoreturn);
56: FunctionPass *createNVPTXMarkKernelPtrsGlobalPass();
57: FunctionPass *createNVPTXTagInvariantLoadsPass();
58: FunctionPass *createNVPTXIRPeepholePass();
59: MachineFunctionPass *createNVPTXPeephole();
60: MachineFunctionPass *createNVPTXProxyRegErasurePass();
61: MachineFunctionPass *createNVPTXForwardParamsPass();
62:
63: void initializeNVVMReflectLegacyPassPass(PassRegistry &);
64: void initializeGenericToNVVMLegacyPassPass(PassRegistry &);
65: void initializeNVPTXAllocaHoistingPass(PassRegistry &);
66: void initializeNVPTXAsmPrinterPass(PassRegistry &);
67: void initializeNVPTXAssignValidGlobalNamesPass(PassRegistry &);
68: void initializeNVPTXAtomicLowerPass(PassRegistry &);
69: void initializeNVPTXCtorDtorLoweringLegacyPass(PassRegistry &);
70: void initializeNVPTXLowerAggrCopiesPass(PassRegistry &);
71: void initializeNVPTXLowerAllocaPass(PassRegistry &);
72: void initializeNVPTXLowerUnreachablePass(PassRegistry &);
73: void initializeNVPTXLowerArgsLegacyPassPass(PassRegistry &);
74: void initializeNVPTXSetByValParamAlignLegacyPassPass(PassRegistry &);
75: void initializeNVPTXProxyRegErasurePass(PassRegistry &);
76: void initializeNVPTXForwardParamsPassPass(PassRegistry &);
77: void initializeNVVMIntrRangePass(PassRegistry &);
78: void initializeNVVMReflectPass(PassRegistry &);
79: void initializeNVPTXAAWrapperPassPass(PassRegistry &);
80: void initializeNVPTXExternalAAWrapperPass(PassRegistry &);
```
- EN: This range declares interfaces or inline helpers such as createNVPTXAssignValidGlobalNamesPass, createGenericToNVVMLegacyPass, createNVPTXCtorDtorLoweringLegacyPass, createNVVMIntrRangePass, defining how other backend pieces interact with this header.
- CN: 这一段声明了 createNVPTXAssignValidGlobalNamesPass、createGenericToNVVMLegacyPass、createNVPTXCtorDtorLoweringLegacyPass、createNVVMIntrRangePass 等接口或内联辅助函数，定义其他后端组件如何与该头文件协作。

### Lines 81-120
```cpp
 81: void initializeNVPTXPeepholePass(PassRegistry &);
 82: void initializeNVPTXMarkKernelPtrsGlobalLegacyPassPass(PassRegistry &);
 83: void initializeNVPTXTagInvariantLoadLegacyPassPass(PassRegistry &);
 84: void initializeNVPTXIRPeepholePass(PassRegistry &);
 85: void initializeNVPTXPrologEpilogPassPass(PassRegistry &);
 86:
 87: struct NVVMIntrRangePass : OptionalPassInfoMixin<NVVMIntrRangePass> {
 88:   PreservedAnalyses run(Function &F, FunctionAnalysisManager &AM);
 89: };
 90:
 91: struct NVPTXIRPeepholePass : OptionalPassInfoMixin<NVPTXIRPeepholePass> {
 92:   PreservedAnalyses run(Function &F, FunctionAnalysisManager &AM);
 93: };
 94:
 95: struct NVVMReflectPass : OptionalPassInfoMixin<NVVMReflectPass> {
 96:   NVVMReflectPass() : SmVersion(0) {}
 97:   NVVMReflectPass(unsigned SmVersion) : SmVersion(SmVersion) {}
 98:   PreservedAnalyses run(Module &F, ModuleAnalysisManager &AM);
 99:
100: private:
101:   unsigned SmVersion;
102: };
103:
104: struct GenericToNVVMPass : OptionalPassInfoMixin<GenericToNVVMPass> {
105:   PreservedAnalyses run(Module &M, ModuleAnalysisManager &AM);
106: };
107:
108: struct NVPTXCopyByValArgsPass : OptionalPassInfoMixin<NVPTXCopyByValArgsPass> {
109:   PreservedAnalyses run(Function &F, FunctionAnalysisManager &AM);
110: };
111:
112: struct NVPTXSetByValParamAlignPass
113:     : OptionalPassInfoMixin<NVPTXSetByValParamAlignPass> {
114:   PreservedAnalyses run(Function &F, FunctionAnalysisManager &AM);
115: };
116:
117: struct NVPTXLowerArgsPass : OptionalPassInfoMixin<NVPTXLowerArgsPass> {
118: private:
119:   TargetMachine &TM;
120:
```
- EN: This range defines or declares important types such as initializeNVPTXPeepholePass, initializeNVPTXMarkKernelPtrsGlobalLegacyPassPass, initializeNVPTXTagInvariantLoadLegacyPassPass, initializeNVPTXIRPeepholePass, shaping the data model used by NVPTX.h.
- CN: 这一段定义或声明了 initializeNVPTXPeepholePass、initializeNVPTXMarkKernelPtrsGlobalLegacyPassPass、initializeNVPTXTagInvariantLoadLegacyPassPass、initializeNVPTXIRPeepholePass 等关键类型，构成 NVPTX.h 使用的数据模型。

### Lines 121-160
```cpp
121: public:
122:   NVPTXLowerArgsPass(TargetMachine &TM) : TM(TM) {};
123:   PreservedAnalyses run(Function &F, FunctionAnalysisManager &AM);
124: };
125:
126: struct NVPTXMarkKernelPtrsGlobalPass
127:     : OptionalPassInfoMixin<NVPTXMarkKernelPtrsGlobalPass> {
128:   PreservedAnalyses run(Function &F, FunctionAnalysisManager &AM);
129: };
130:
131: struct NVPTXTagInvariantLoadsPass
132:     : OptionalPassInfoMixin<NVPTXTagInvariantLoadsPass> {
133:   PreservedAnalyses run(Function &F, FunctionAnalysisManager &AM);
134: };
135:
136: namespace NVPTX {
137: enum DrvInterface {
138:   NVCL,
139:   CUDA
140: };
141:
142: // A field inside TSFlags needs a shift and a mask. The usage is
143: // always as follows :
144: // ((TSFlags & fieldMask) >> fieldShift)
145: // The enum keeps the mask, the shift, and all valid values of the
146: // field in one place.
147: enum VecInstType {
148:   VecInstTypeShift = 0,
149:   VecInstTypeMask = 0xF,
150:
151:   VecNOP = 0,
152:   VecLoad = 1,
153:   VecStore = 2,
154:   VecBuild = 3,
155:   VecShuffle = 4,
156:   VecExtract = 5,
157:   VecInsert = 6,
158:   VecDest = 7,
159:   VecOther = 15
160: };
```
- EN: This range defines or declares important types such as NVPTXLowerArgsPass, run, NVPTXMarkKernelPtrsGlobalPass, NVPTXTagInvariantLoadsPass, shaping the data model used by NVPTX.h.
- CN: 这一段定义或声明了 NVPTXLowerArgsPass、run、NVPTXMarkKernelPtrsGlobalPass、NVPTXTagInvariantLoadsPass 等关键类型，构成 NVPTX.h 使用的数据模型。

### Lines 161-200
```cpp
161:
162: enum SimpleMove {
163:   SimpleMoveMask = 0x10,
164:   SimpleMoveShift = 4
165: };
166: enum LoadStore {
167:   isLoadMask = 0x20,
168:   isLoadShift = 5,
169:   isStoreMask = 0x40,
170:   isStoreShift = 6
171: };
172:
173: // Extends LLVM AtomicOrdering with PTX Orderings:
174: using OrderingUnderlyingType = unsigned int;
175: enum Ordering : OrderingUnderlyingType {
176:   NotAtomic = (OrderingUnderlyingType)
177:       AtomicOrdering::NotAtomic, // PTX calls these: "Weak"
178:   // Unordered = 1, // NVPTX maps LLVM Unorderd to Relaxed
179:   Relaxed = (OrderingUnderlyingType)AtomicOrdering::Monotonic,
180:   // Consume = 3,   // Unimplemented in LLVM; NVPTX would map to "Acquire"
181:   Acquire = (OrderingUnderlyingType)AtomicOrdering::Acquire,
182:   Release = (OrderingUnderlyingType)AtomicOrdering::Release,
183:   AcquireRelease = (OrderingUnderlyingType)AtomicOrdering::AcquireRelease,
184:   SequentiallyConsistent =
185:       (OrderingUnderlyingType)AtomicOrdering::SequentiallyConsistent,
186:   Volatile = SequentiallyConsistent + 1,
187:   RelaxedMMIO = Volatile + 1,
188: };
189:
190: using ScopeUnderlyingType = unsigned int;
191: enum Scope : ScopeUnderlyingType {
192:   Thread = 0,
193:   Block = 1,
194:   Cluster = 2,
195:   Device = 3,
196:   System = 4,
197:   DefaultDevice = 5, //  For SM < 70: denotes PTX op implicit/default .gpu scope
198:   LASTSCOPE = DefaultDevice
199: };
200:
```
- EN: This range defines or declares important types such as SimpleMove, LoadStore, Ordering, Scope, shaping the data model used by NVPTX.h.
- CN: 这一段定义或声明了 SimpleMove、LoadStore、Ordering、Scope 等关键类型，构成 NVPTX.h 使用的数据模型。

### Lines 201-240
```cpp
201: using AddressSpaceUnderlyingType = unsigned int;
202: enum AddressSpace : AddressSpaceUnderlyingType {
203:   Generic = NVPTXAS::ADDRESS_SPACE_GENERIC,
204:   Global = NVPTXAS::ADDRESS_SPACE_GLOBAL,
205:   Shared = NVPTXAS::ADDRESS_SPACE_SHARED,
206:   Const = NVPTXAS::ADDRESS_SPACE_CONST,
207:   Local = NVPTXAS::ADDRESS_SPACE_LOCAL,
208:   SharedCluster = NVPTXAS::ADDRESS_SPACE_SHARED_CLUSTER,
209:   EntryParam = NVPTXAS::ADDRESS_SPACE_ENTRY_PARAM,
210:
211:   // DeviceParam is not a real address space, as it does not support pointers
212:   // and instead can only be referenced by param+offset. For this reason it is
213:   // only used in MIR as an instruction modifier and should not be used in LLVM
214:   // IR.
215:   DeviceParam
216: };
217:
218: namespace PTXLdStInstCode {
219: enum FromType { Unsigned = 0, Signed, Float, Untyped };
220: } // namespace PTXLdStInstCode
221:
222: /// PTXCvtMode - Conversion code enumeration
223: namespace PTXCvtMode {
224: enum CvtMode {
225:   NONE = 0,
226:   RNI,
227:   RZI,
228:   RMI,
229:   RPI,
230:   RN,
231:   RZ,
232:   RM,
233:   RP,
234:   RNA,
235:   RS,
236:
237:   BASE_MASK = 0x0F,
238:   FTZ_FLAG = 0x10,
239:   SAT_FLAG = 0x20,
240:   RELU_FLAG = 0x40,
```
- EN: This range defines or declares important types such as AddressSpace, FromType, CvtMode, shaping the data model used by NVPTX.h.
- CN: 这一段定义或声明了 AddressSpace、FromType、CvtMode 等关键类型，构成 NVPTX.h 使用的数据模型。

### Lines 241-280
```cpp
241:   SATFINITE_FLAG = 0x80
242: };
243: }
244:
245: /// PTXCmpMode - Comparison mode enumeration
246: namespace PTXCmpMode {
247: enum CmpMode {
248:   EQ = 0,
249:   NE,
250:   LT,
251:   LE,
252:   GT,
253:   GE,
254:   EQU,
255:   NEU,
256:   LTU,
257:   LEU,
258:   GTU,
259:   GEU,
260:   NUM,
261:   // NAN is a MACRO
262:   NotANumber,
263: };
264: }
265:
266: namespace PTXPrmtMode {
267: enum PrmtMode {
268:   NONE,
269:   F4E,
270:   B4E,
271:   RC8,
272:   ECL,
273:   ECR,
274:   RC16,
275: };
276: }
277:
278: enum class DivPrecisionLevel : unsigned {
279:   Approx = 0,
280:   Full = 1,
```
- EN: This range defines or declares important types such as CmpMode, PrmtMode, DivPrecisionLevel, shaping the data model used by NVPTX.h.
- CN: 这一段定义或声明了 CmpMode、PrmtMode、DivPrecisionLevel 等关键类型，构成 NVPTX.h 使用的数据模型。

### Lines 281-299
```cpp
281:   IEEE754 = 2,
282:   IEEE754_NoFTZ = 3,
283: };
284:
285: } // namespace NVPTX
286: void initializeNVPTXDAGToDAGISelLegacyPass(PassRegistry &);
287: } // namespace llvm
288:
289: // Defines symbolic names for NVPTX registers.  This defines a mapping from
290: // register name to register number.
291: #define GET_REGINFO_ENUM
292: #include "NVPTXGenRegisterInfo.inc"
293:
294: // Defines symbolic names for the NVPTX instructions.
295: #define GET_INSTRINFO_ENUM
296: #define GET_INSTRINFO_MC_HELPER_DECLS
297: #include "NVPTXGenInstrInfo.inc"
298:
299: #endif
```
- EN: This range declares interfaces or inline helpers such as initializeNVPTXDAGToDAGISelLegacyPass, defining how other backend pieces interact with this header.
- CN: 这一段声明了 initializeNVPTXDAGToDAGISelLegacyPass 等接口或内联辅助函数，定义其他后端组件如何与该头文件协作。

## Key Concepts / 关键概念

- EN: NVPTX backend code reflects GPU execution concerns such as address spaces, intrinsics, and PTX-specific lowering.
  - CN: NVPTX 后端代码体现了 GPU 执行特性，例如地址空间、内建函数以及 PTX 特定降级。
- EN: Key symbols in this file include FunctionPass, MachineFunctionPass, NVPTXTargetMachine, PassRegistry, CondCodes, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 FunctionPass, MachineFunctionPass, NVPTXTargetMachine, PassRegistry, CondCodes，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- LLVM infrastructure / LLVM 基础设施:
  - `llvm/IR/PassManager.h`
  - `llvm/Pass.h`
  - `llvm/Support/AtomicOrdering.h`
  - `llvm/Support/CodeGen.h`
  - `llvm/Support/NVPTXAddrSpace.h`
  - `llvm/Target/TargetMachine.h`
- System/standard headers / 系统或标准头文件:
  - `NVPTXGenRegisterInfo.inc`
  - `NVPTXGenInstrInfo.inc`
