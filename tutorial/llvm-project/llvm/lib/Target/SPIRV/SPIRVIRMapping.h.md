# SPIRVIRMapping.h — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/SPIRV/SPIRVIRMapping.h`
- Repository: `llvm-project`
- Purpose (EN): SPIRVMapping.h - SPIR-V Duplicates Tracker ----*- C++ -*-===.
- 目的（中文）: 该文件声明 LLVM 目标后端使用的数据结构、接口或辅助类型。
- Language: C++ header

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-60
```cpp
 1: //===------------ SPIRVMapping.h - SPIR-V Duplicates Tracker ----*- C++ -*-===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // General infrastructure for keeping track of the values that according to
10: // the SPIR-V binary layout should be global to the whole module.
11: //
12: //===----------------------------------------------------------------------===//
13:
14: #ifndef LLVM_LIB_TARGET_SPIRV_SPIRVIRMAPPING_H
15: #define LLVM_LIB_TARGET_SPIRV_SPIRVIRMAPPING_H
16:
17: #include "MCTargetDesc/SPIRVBaseInfo.h"
18: #include "MCTargetDesc/SPIRVMCTargetDesc.h"
19: #include "SPIRVUtils.h"
20: #include "llvm/ADT/DenseMap.h"
21: #include "llvm/ADT/Hashing.h"
22: #include "llvm/CodeGen/GlobalISel/MachineIRBuilder.h"
23: #include "llvm/CodeGen/MachineModuleInfo.h"
24:
25: namespace llvm {
26: namespace SPIRV {
27:
28: inline size_t to_hash(const MachineInstr *MI) {
29:   hash_code H = llvm::hash_combine(MI->getOpcode(), MI->getNumOperands());
30:   for (unsigned I = MI->getNumDefs(); I < MI->getNumOperands(); ++I) {
31:     const MachineOperand &MO = MI->getOperand(I);
32:     if (MO.getType() == MachineOperand::MO_CImmediate)
33:       H = llvm::hash_combine(H, MO.getType(), MO.getCImm());
34:     else if (MO.getType() == MachineOperand::MO_FPImmediate)
35:       H = llvm::hash_combine(H, MO.getType(), MO.getFPImm());
36:     else
37:       H = llvm::hash_combine(H, MO.getType());
38:   }
39:   return H;
40: }
41:
42: using MIHandle = std::tuple<const MachineInstr *, Register, size_t>;
43:
44: inline MIHandle getMIKey(const MachineInstr *MI) {
45:   return std::make_tuple(MI, MI->getOperand(0).getReg(), SPIRV::to_hash(MI));
46: }
47:
48: using IRHandle = std::tuple<const void *, unsigned, unsigned>;
49: using IRHandleMF = std::pair<IRHandle, const MachineFunction *>;
50:
51: inline IRHandleMF getIRHandleMF(IRHandle Handle, const MachineFunction *MF) {
52:   return std::make_pair(Handle, MF);
53: }
54:
55: enum SpecialTypeKind {
56:   STK_Empty = 0,
57:   STK_Image,
58:   STK_SampledImage,
59:   STK_Sampler,
60:   STK_Pipe,
```
- EN: This range defines or declares important types such as to_hash, llvm::hash_combine, getOperand, getMIKey, shaping the data model used by SPIRVIRMapping.h.
- CN: 这一段定义或声明了 to_hash、llvm::hash_combine、getOperand、getMIKey 等关键类型，构成 SPIRVIRMapping.h 使用的数据模型。

### Lines 61-120
```cpp
 61:   STK_DeviceEvent,
 62:   STK_ElementPointer,
 63:   STK_Type,
 64:   STK_Value,
 65:   STK_MachineInstr,
 66:   STK_VkBuffer,
 67:   STK_Padding,
 68:   STK_ExplictLayoutType,
 69:   STK_Last = -1
 70: };
 71:
 72: union ImageAttrs {
 73:   struct BitFlags {
 74:     unsigned Dim : 3;
 75:     unsigned Depth : 2;
 76:     unsigned Arrayed : 1;
 77:     unsigned MS : 1;
 78:     unsigned Sampled : 2;
 79:     unsigned ImageFormat : 6;
 80:     unsigned AQ : 2;
 81:   } Flags;
 82:   unsigned Val;
 83:
 84:   ImageAttrs(unsigned Dim, unsigned Depth, unsigned Arrayed, unsigned MS,
 85:              unsigned Sampled, unsigned ImageFormat, unsigned AQ = 0) {
 86:     Val = 0;
 87:     Flags.Dim = Dim;
 88:     Flags.Depth = Depth;
 89:     Flags.Arrayed = Arrayed;
 90:     Flags.MS = MS;
 91:     Flags.Sampled = Sampled;
 92:     Flags.ImageFormat = ImageFormat;
 93:     Flags.AQ = AQ;
 94:   }
 95: };
 96:
 97: inline IRHandle irhandle_image(const Type *SampledTy, unsigned Dim,
 98:                                unsigned Depth, unsigned Arrayed, unsigned MS,
 99:                                unsigned Sampled, unsigned ImageFormat,
100:                                unsigned AQ = 0) {
101:   return std::make_tuple(
102:       SampledTy,
103:       ImageAttrs(Dim, Depth, Arrayed, MS, Sampled, ImageFormat, AQ).Val,
104:       SpecialTypeKind::STK_Image);
105: }
106:
107: inline IRHandle irhandle_sampled_image(const Type *SampledTy,
108:                                        const MachineInstr *ImageTy) {
109:   assert(ImageTy->getOpcode() == SPIRV::OpTypeImage);
110:   unsigned AC = AccessQualifier::AccessQualifier::None;
111:   if (ImageTy->getNumOperands() > 8)
112:     AC = ImageTy->getOperand(8).getImm();
113:   return std::make_tuple(
114:       SampledTy,
115:       ImageAttrs(
116:           ImageTy->getOperand(2).getImm(), ImageTy->getOperand(3).getImm(),
117:           ImageTy->getOperand(4).getImm(), ImageTy->getOperand(5).getImm(),
118:           ImageTy->getOperand(6).getImm(), ImageTy->getOperand(7).getImm(), AC)
119:           .Val,
120:       SpecialTypeKind::STK_SampledImage);
```
- EN: This range defines or declares important types such as BitFlags, assert, getOperand, shaping the data model used by SPIRVIRMapping.h.
- CN: 这一段定义或声明了 BitFlags、assert、getOperand 等关键类型，构成 SPIRVIRMapping.h 使用的数据模型。

### Lines 121-180
```cpp
121: }
122:
123: inline IRHandle irhandle_sampler() {
124:   return std::make_tuple(nullptr, 0U, SpecialTypeKind::STK_Sampler);
125: }
126:
127: inline IRHandle irhandle_pipe(uint8_t AQ) {
128:   return std::make_tuple(nullptr, AQ, SpecialTypeKind::STK_Pipe);
129: }
130:
131: inline IRHandle irhandle_event() {
132:   return std::make_tuple(nullptr, 0U, SpecialTypeKind::STK_DeviceEvent);
133: }
134:
135: inline IRHandle irhandle_pointee(const Type *ElementType,
136:                                  unsigned AddressSpace) {
137:   return std::make_tuple(unifyPtrType(ElementType), AddressSpace,
138:                          SpecialTypeKind::STK_ElementPointer);
139: }
140:
141: inline IRHandle irhandle_ptr(const void *Ptr, unsigned Arg,
142:                              enum SpecialTypeKind STK) {
143:   return std::make_tuple(Ptr, Arg, STK);
144: }
145:
146: inline IRHandle irhandle_vkbuffer(const Type *ElementType,
147:                                   StorageClass::StorageClass SC,
148:                                   bool IsWriteable) {
149:   return std::make_tuple(ElementType, (SC << 1) | IsWriteable,
150:                          SpecialTypeKind::STK_VkBuffer);
151: }
152:
153: inline IRHandle irhandle_padding() {
154:   return std::make_tuple(nullptr, 0, SpecialTypeKind::STK_Padding);
155: }
156:
157: inline IRHandle irhandle_explict_layout_type(const Type *Ty) {
158:   const Type *WrpTy = unifyPtrType(Ty);
159:   return irhandle_ptr(WrpTy, Ty->getTypeID(), STK_ExplictLayoutType);
160: }
161:
162: inline IRHandle handle(const Type *Ty) {
163:   const Type *WrpTy = unifyPtrType(Ty);
164:   return irhandle_ptr(WrpTy, Ty->getTypeID(), STK_Type);
165: }
166:
167: inline IRHandle handle(const Value *V) {
168:   return irhandle_ptr(V, V->getValueID(), STK_Value);
169: }
170:
171: inline IRHandle handle(const MachineInstr *KeyMI) {
172:   return irhandle_ptr(KeyMI, SPIRV::to_hash(KeyMI), STK_MachineInstr);
173: }
174:
175: inline bool type_has_layout_decoration(const Type *T) {
176:   return (isa<StructType>(T) || isa<ArrayType>(T));
177: }
178:
179: } // namespace SPIRV
180:
```
- EN: This range defines or declares important types such as irhandle_sampler, std::make_tuple, irhandle_pipe, irhandle_event, shaping the data model used by SPIRVIRMapping.h.
- CN: 这一段定义或声明了 irhandle_sampler、std::make_tuple、irhandle_pipe、irhandle_event 等关键类型，构成 SPIRVIRMapping.h 使用的数据模型。

### Lines 181-240
```cpp
181: // Bi-directional mappings between LLVM entities and (v-reg, machine function)
182: // pairs support management of unique SPIR-V definitions per machine function
183: // per an LLVM/GlobalISel entity (e.g., Type, Constant, Machine Instruction).
184: class SPIRVIRMapping {
185:   DenseMap<SPIRV::IRHandleMF, SPIRV::MIHandle> Vregs;
186:   DenseMap<const MachineInstr *, SPIRV::IRHandleMF> Defs;
187:
188: public:
189:   bool add(SPIRV::IRHandle Handle, const MachineInstr *MI) {
190:     if (auto DefIt = Defs.find(MI); DefIt != Defs.end()) {
191:       auto [ExistHandle, ExistMF] = DefIt->second;
192:       if (Handle == ExistHandle && MI->getMF() == ExistMF)
193:         return false; // already exists
194:       // invalidate the record
195:       Vregs.erase(DefIt->second);
196:       Defs.erase(DefIt);
197:     }
198:     SPIRV::IRHandleMF HandleMF = SPIRV::getIRHandleMF(Handle, MI->getMF());
199:     SPIRV::MIHandle MIKey = SPIRV::getMIKey(MI);
200:     auto It1 = Vregs.try_emplace(HandleMF, MIKey);
201:     if (!It1.second) {
202:       // there is an expired record that we need to invalidate
203:       Defs.erase(std::get<0>(It1.first->second));
204:       // update the record
205:       It1.first->second = MIKey;
206:     }
207:     [[maybe_unused]] auto It2 = Defs.try_emplace(MI, HandleMF);
208:     assert(It2.second);
209:     return true;
210:   }
211:   bool erase(const MachineInstr *MI) {
212:     bool Res = false;
213:     if (auto It = Defs.find(MI); It != Defs.end()) {
214:       Res = Vregs.erase(It->second);
215:       Defs.erase(It);
216:     }
217:     return Res;
218:   }
219:   const MachineInstr *findMI(SPIRV::IRHandle Handle,
220:                              const MachineFunction *MF) {
221:     SPIRV::IRHandleMF HandleMF = SPIRV::getIRHandleMF(Handle, MF);
222:     auto It = Vregs.find(HandleMF);
223:     if (It == Vregs.end())
224:       return nullptr;
225:     auto [MI, Reg, Hash] = It->second;
226:     const MachineInstr *Def = MF->getRegInfo().getVRegDef(Reg);
227:     if (!Def || Def != MI || SPIRV::to_hash(MI) != Hash) {
228:       // there is an expired record that we need to invalidate
229:       erase(MI);
230:       return nullptr;
231:     }
232:     assert(Defs.contains(MI) && Defs.find(MI)->second == HandleMF);
233:     return MI;
234:   }
235:   Register find(SPIRV::IRHandle Handle, const MachineFunction *MF) {
236:     const MachineInstr *MI = findMI(Handle, MF);
237:     return MI ? MI->getOperand(0).getReg() : Register();
238:   }
239:
240:   // helpers
```
- EN: This range defines or declares important types such as SPIRVIRMapping, add, erase, SPIRV::getIRHandleMF, shaping the data model used by SPIRVIRMapping.h.
- CN: 这一段定义或声明了 SPIRVIRMapping、add、erase、SPIRV::getIRHandleMF 等关键类型，构成 SPIRVIRMapping.h 使用的数据模型。

### Lines 241-300
```cpp
241:   bool add(const Type *PointeeTy, unsigned AddressSpace,
242:            const MachineInstr *MI) {
243:     return add(SPIRV::irhandle_pointee(PointeeTy, AddressSpace), MI);
244:   }
245:   Register find(const Type *PointeeTy, unsigned AddressSpace,
246:                 const MachineFunction *MF) {
247:     return find(SPIRV::irhandle_pointee(PointeeTy, AddressSpace), MF);
248:   }
249:   const MachineInstr *findMI(const Type *PointeeTy, unsigned AddressSpace,
250:                              const MachineFunction *MF) {
251:     return findMI(SPIRV::irhandle_pointee(PointeeTy, AddressSpace), MF);
252:   }
253:
254:   bool add(const Value *V, const MachineInstr *MI) {
255:     return add(SPIRV::handle(V), MI);
256:   }
257:
258:   bool add(const Type *T, bool RequiresExplicitLayout, const MachineInstr *MI) {
259:     if (RequiresExplicitLayout && SPIRV::type_has_layout_decoration(T)) {
260:       return add(SPIRV::irhandle_explict_layout_type(T), MI);
261:     }
262:     return add(SPIRV::handle(T), MI);
263:   }
264:
265:   bool add(const MachineInstr *Obj, const MachineInstr *MI) {
266:     return add(SPIRV::handle(Obj), MI);
267:   }
268:
269:   Register find(const Value *V, const MachineFunction *MF) {
270:     return find(SPIRV::handle(V), MF);
271:   }
272:
273:   Register find(const Type *T, bool RequiresExplicitLayout,
274:                 const MachineFunction *MF) {
275:     if (RequiresExplicitLayout && SPIRV::type_has_layout_decoration(T))
276:       return find(SPIRV::irhandle_explict_layout_type(T), MF);
277:     return find(SPIRV::handle(T), MF);
278:   }
279:
280:   Register find(const MachineInstr *MI, const MachineFunction *MF) {
281:     return find(SPIRV::handle(MI), MF);
282:   }
283:
284:   const MachineInstr *findMI(const Value *Obj, const MachineFunction *MF) {
285:     return findMI(SPIRV::handle(Obj), MF);
286:   }
287:
288:   const MachineInstr *findMI(const Type *T, bool RequiresExplicitLayout,
289:                              const MachineFunction *MF) {
290:     if (RequiresExplicitLayout && SPIRV::type_has_layout_decoration(T))
291:       return findMI(SPIRV::irhandle_explict_layout_type(T), MF);
292:     return findMI(SPIRV::handle(T), MF);
293:   }
294:
295:   const MachineInstr *findMI(const MachineInstr *Obj,
296:                              const MachineFunction *MF) {
297:     return findMI(SPIRV::handle(Obj), MF);
298:   }
299: };
300: } // namespace llvm
```
- EN: This range declares interfaces or inline helpers such as add, find, findMI, defining how other backend pieces interact with this header.
- CN: 这一段声明了 add、find、findMI 等接口或内联辅助函数，定义其他后端组件如何与该头文件协作。

### Lines 301-301
```cpp
301: #endif // LLVM_LIB_TARGET_SPIRV_SPIRVIRMAPPING_H
```
- EN: This range contains connective backend logic, including helper statements, control flow, and data movement needed by the surrounding implementation.
- CN: 这一段包含衔接性的后端逻辑，包括辅助语句、控制流和数据处理，用于支撑周围实现。

## Key Concepts / 关键概念

- EN: SPIR-V backend code must preserve rich type information and encode capabilities/extensions explicitly.
  - CN: SPIR-V 后端需要保留丰富的类型信息，并显式编码 capability 与 extension。
- EN: Key symbols in this file include to_hash, llvm::hash_combine, getOperand, getMIKey, std::make_tuple, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 to_hash, llvm::hash_combine, getOperand, getMIKey, std::make_tuple，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `MCTargetDesc/SPIRVBaseInfo.h`
  - `MCTargetDesc/SPIRVMCTargetDesc.h`
  - `SPIRVUtils.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/ADT/DenseMap.h`
  - `llvm/ADT/Hashing.h`
  - `llvm/CodeGen/GlobalISel/MachineIRBuilder.h`
  - `llvm/CodeGen/MachineModuleInfo.h`
