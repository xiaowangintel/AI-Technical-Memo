# NVPTXSubtarget.h — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/NVPTX/NVPTXSubtarget.h`
- Repository: `llvm-project`
- Purpose (EN): This file declares the NVPTX specific subclass of TargetSubtarget.
- 目的（中文）: 该文件声明 LLVM 目标后端使用的数据结构、接口或辅助类型。
- Language: C++ header

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-60
```cpp
 1: //=====-- NVPTXSubtarget.h - Define Subtarget for the NVPTX ---*- C++ -*--====//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // This file declares the NVPTX specific subclass of TargetSubtarget.
10: //
11: //===----------------------------------------------------------------------===//
12:
13: #ifndef LLVM_LIB_TARGET_NVPTX_NVPTXSUBTARGET_H
14: #define LLVM_LIB_TARGET_NVPTX_NVPTXSUBTARGET_H
15:
16: #include "NVPTX.h"
17: #include "NVPTXFrameLowering.h"
18: #include "NVPTXISelLowering.h"
19: #include "NVPTXInstrInfo.h"
20: #include "NVPTXRegisterInfo.h"
21: #include "llvm/CodeGen/TargetSubtargetInfo.h"
22: #include "llvm/IR/DataLayout.h"
23: #include "llvm/IR/NVVMIntrinsicUtils.h"
24: #include "llvm/Support/NVPTXAddrSpace.h"
25: #include <string>
26:
27: #define GET_SUBTARGETINFO_HEADER
28: #include "NVPTXGenSubtargetInfo.inc"
29:
30: namespace llvm {
31:
32: // FullSmVersion encoding: SM * 10 + ArchSuffixOffset
33: // ArchSuffixOffset: 0 (base), 2 ('f'), 3 ('a')
34: // e.g. sm_100 -> 1000, sm_100f -> 1002, sm_100a -> 1003
35:
36: class NVPTXSubtarget : public NVPTXGenSubtargetInfo {
37:   virtual void anchor();
38:   std::string TargetName;
39:
40:   // PTX version x.y is represented as 10*x+y, e.g. 3.1 == 31
41:   unsigned PTXVersion;
42:
43:   // FullSmVersion encoding: SM * 10 + ArchSuffixOffset
44:   // ArchSuffixOffset: 0 (base), 2 ('f'), 3 ('a')
45:   // e.g. sm_30 -> 300, sm_90a -> 903, sm_100f -> 1002
46:   unsigned int FullSmVersion;
47:
48:   // SM version x.y is represented as 10*x+y, e.g. 3.1 == 31. Derived from
49:   // FullSmVersion.
50:   unsigned int SmVersion;
51:
52:   NVPTXInstrInfo InstrInfo;
53:   NVPTXTargetLowering TLInfo;
54:   std::unique_ptr<const SelectionDAGTargetInfo> TSInfo;
55:
56:   // NVPTX does not have any call stack frame, but need a NVPTX specific
57:   // FrameLowering class because TargetFrameLowering is abstract.
58:   NVPTXFrameLowering FrameLowering;
59:
60: public:
```
- EN: This range defines or declares important types such as NVPTXSubtarget, anchor, shaping the data model used by NVPTXSubtarget.h.
- CN: 这一段定义或声明了 NVPTXSubtarget、anchor 等关键类型，构成 NVPTXSubtarget.h 使用的数据模型。

### Lines 61-120
```cpp
 61:   /// This constructor initializes the data members to match that
 62:   /// of the specified module.
 63:   ///
 64:   NVPTXSubtarget(const Triple &TT, const std::string &CPU,
 65:                  const std::string &FS, const NVPTXTargetMachine &TM);
 66:
 67:   ~NVPTXSubtarget() override;
 68:
 69:   const TargetFrameLowering *getFrameLowering() const override {
 70:     return &FrameLowering;
 71:   }
 72:   const NVPTXInstrInfo *getInstrInfo() const override { return &InstrInfo; }
 73:   const NVPTXRegisterInfo *getRegisterInfo() const override {
 74:     return &InstrInfo.getRegisterInfo();
 75:   }
 76:   const NVPTXTargetLowering *getTargetLowering() const override {
 77:     return &TLInfo;
 78:   }
 79:
 80:   const SelectionDAGTargetInfo *getSelectionDAGInfo() const override;
 81:
 82:   // Checks PTX version and family-specific and architecture-specific SM
 83:   // versions. For example, sm_100{f/a} and any future variants in the same
 84:   // family will match for any PTX version greater than or equal to
 85:   // `PTXVersion`.
 86:   bool hasPTXWithFamilySMs(unsigned PTXVersion,
 87:                            ArrayRef<unsigned> SMVersions) const;
 88:   // Checks PTX version and architecture-specific SM versions.
 89:   // For example, sm_100{a} will match for any PTX version greater than or equal
 90:   // to `PTXVersion`.
 91:   bool hasPTXWithAccelSMs(unsigned PTXVersion,
 92:                           ArrayRef<unsigned> SMVersions) const;
 93:
 94:   bool has256BitVectorLoadStore(unsigned AS) const {
 95:     return SmVersion >= 100 && PTXVersion >= 88 &&
 96:            AS == NVPTXAS::ADDRESS_SPACE_GLOBAL;
 97:   }
 98:   bool hasUsedBytesMaskPragma() const {
 99:     return SmVersion >= 50 && PTXVersion >= 83;
100:   }
101:   bool hasAtomAddF64() const { return SmVersion >= 60; }
102:   bool hasAtomScope() const { return SmVersion >= 60; }
103:   bool hasAtomBitwise64() const { return SmVersion >= 32; }
104:   bool hasAtomMinMax64() const { return SmVersion >= 32; }
105:   bool hasAtomCas16() const { return SmVersion >= 70 && PTXVersion >= 63; }
106:   bool hasAtomSwap128() const { return SmVersion >= 90 && PTXVersion >= 83; }
107:   bool hasClusters() const { return SmVersion >= 90 && PTXVersion >= 78; }
108:   bool hasLDG() const { return SmVersion >= 32; }
109:   bool hasHWROT32() const { return SmVersion >= 32; }
110:   bool hasBrx() const { return SmVersion >= 30 && PTXVersion >= 60; }
111:   bool hasFP16Math() const { return SmVersion >= 53; }
112:   bool hasBF16Math() const { return SmVersion >= 80; }
113:   bool allowFP16Math() const;
114:   bool hasMaskOperator() const { return PTXVersion >= 71; }
115:   bool hasNoReturn() const { return SmVersion >= 30 && PTXVersion >= 64; }
116:   // Does SM & PTX support memory orderings (weak and atomic: relaxed, acquire,
117:   // release, acq_rel, sc) ?
118:   bool hasMemoryOrdering() const { return SmVersion >= 70 && PTXVersion >= 60; }
119:   // Does SM & PTX support .acquire and .release qualifiers for fence?
120:   bool hasSplitAcquireAndReleaseFences() const {
```
- EN: This range declares interfaces or inline helpers such as ~NVPTXSubtarget, getFrameLowering, getInstrInfo, getRegisterInfo, defining how other backend pieces interact with this header.
- CN: 这一段声明了 ~NVPTXSubtarget、getFrameLowering、getInstrInfo、getRegisterInfo 等接口或内联辅助函数，定义其他后端组件如何与该头文件协作。

### Lines 121-180
```cpp
121:     return SmVersion >= 90 && PTXVersion >= 86;
122:   }
123:   // Does SM & PTX support atomic relaxed MMIO operations ?
124:   bool hasRelaxedMMIO() const { return SmVersion >= 70 && PTXVersion >= 82; }
125:   bool hasDotInstructions() const {
126:     return SmVersion >= 61 && PTXVersion >= 50;
127:   }
128:
129:   // Checks following instructions support:
130:   // - tcgen05.ld/st
131:   // - tcgen05.alloc/dealloc/relinquish
132:   // - tcgen05.cp
133:   // - tcgen05.fence/wait
134:   // - tcgen05.commit
135:   // - tcgen05.mma
136:   bool hasTcgen05InstSupport() const {
137:     // sm_101 renamed to sm_110 in PTX 9.0
138:     return hasPTXWithFamilySMs(90, {100, 110}) ||
139:            hasPTXWithFamilySMs(88, {100, 101}) ||
140:            hasPTXWithAccelSMs(86, {100, 101});
141:   }
142:
143:   // Checks tcgen05.shift instruction support.
144:   bool hasTcgen05ShiftSupport() const {
145:     // sm_101 renamed to sm_110 in PTX 9.0
146:     return hasPTXWithAccelSMs(90, {100, 110, 103}) ||
147:            hasPTXWithAccelSMs(88, {100, 101, 103}) ||
148:            hasPTXWithAccelSMs(86, {100, 101});
149:   }
150:
151:   bool hasTcgen05MMAScaleInputDImm() const {
152:     return hasPTXWithFamilySMs(88, {100}) || hasPTXWithAccelSMs(86, {100});
153:   }
154:
155:   bool hasTcgen05MMAI8Kind() const {
156:     return hasPTXWithAccelSMs(90, {100, 110}) ||
157:            hasPTXWithAccelSMs(86, {100, 101});
158:   }
159:
160:   bool hasTcgen05MMASparseMxf4nvf4() const {
161:     return hasPTXWithAccelSMs(90, {100, 110, 103}) ||
162:            hasPTXWithAccelSMs(87, {100, 101, 103});
163:   }
164:
165:   bool hasTcgen05MMASparseMxf4() const {
166:     return hasPTXWithAccelSMs(90, {100, 110, 103}) ||
167:            hasPTXWithAccelSMs(86, {100, 101, 103});
168:   }
169:
170:   bool hasTcgen05LdRedSupport() const {
171:     return hasPTXWithFamilySMs(90, {110, 103}) ||
172:            hasPTXWithFamilySMs(88, {101, 103});
173:   }
174:
175:   bool hasReduxSyncF32() const {
176:     return hasPTXWithFamilySMs(88, {100}) || hasPTXWithAccelSMs(86, {100});
177:   }
178:
179:   bool hasMMABlockScale() const {
180:     return hasPTXWithFamilySMs(88, {120}) || hasPTXWithAccelSMs(87, {120});
```
- EN: This range declares interfaces or inline helpers such as hasRelaxedMMIO, hasDotInstructions, hasTcgen05InstSupport, hasPTXWithAccelSMs, defining how other backend pieces interact with this header.
- CN: 这一段声明了 hasRelaxedMMIO、hasDotInstructions、hasTcgen05InstSupport、hasPTXWithAccelSMs 等接口或内联辅助函数，定义其他后端组件如何与该头文件协作。

### Lines 181-240
```cpp
181:   }
182:
183:   bool hasMMASparseBlockScaleF4() const {
184:     return hasPTXWithAccelSMs(87, {120, 121});
185:   }
186:
187:   bool hasMMAWithMXF4NVF4Scale4xE8M0() const {
188:     return hasPTXWithFamilySMs(91, {120});
189:   }
190:
191:   bool hasMMASparseWithMXF4NVF4Scale4xE8M0() const {
192:     return hasPTXWithAccelSMs(91, {120, 121});
193:   }
194:
195:   // f32x2 instructions in Blackwell family
196:   bool hasF32x2Instructions() const;
197:
198:   // Checks support for following in TMA:
199:   //  - cta_group::1/2 support
200:   //  - im2col_w/w_128 mode support
201:   //  - tile_gather4 mode support
202:   //  - tile_scatter4 mode support
203:   bool hasTMABlackwellSupport() const {
204:     return hasPTXWithFamilySMs(90, {100, 110}) ||
205:            hasPTXWithFamilySMs(88, {100, 101}) ||
206:            hasPTXWithAccelSMs(86, {100, 101});
207:   }
208:
209:   // Checks support for conversions involving e4m3x2 and e5m2x2.
210:   bool hasFP8ConversionSupport() const {
211:     if (PTXVersion >= 81)
212:       return SmVersion >= 89;
213:
214:     if (PTXVersion >= 78)
215:       return SmVersion >= 90;
216:
217:     return false;
218:   }
219:
220:   // Checks support for conversions involving the following types:
221:   // - e2m3x2/e3m2x2
222:   // - e2m1x2
223:   // - ue8m0x2
224:   bool hasNarrowFPConversionSupport() const {
225:     return hasPTXWithFamilySMs(90, {100, 110, 120}) ||
226:            hasPTXWithFamilySMs(88, {100, 101, 120}) ||
227:            hasPTXWithAccelSMs(86, {100, 101, 120});
228:   }
229:
230:   // Checks support for conversions involving the following types:
231:   // - bf16x2 -> f8x2
232:   // - f16x2 -> f6x2
233:   // - bf16x2 -> f6x2
234:   // - f16x2 -> f4x2
235:   // - bf16x2 -> f4x2
236:   bool hasFP16X2ToNarrowFPConversionSupport() const {
237:     return hasPTXWithFamilySMs(91, {100, 110, 120});
238:   }
239:
240:   bool hasS2F6X2ConversionSupport() const {
```
- EN: This range declares interfaces or inline helpers such as hasMMASparseBlockScaleF4, hasPTXWithAccelSMs, hasMMAWithMXF4NVF4Scale4xE8M0, hasPTXWithFamilySMs, defining how other backend pieces interact with this header.
- CN: 这一段声明了 hasMMASparseBlockScaleF4、hasPTXWithAccelSMs、hasMMAWithMXF4NVF4Scale4xE8M0、hasPTXWithFamilySMs 等接口或内联辅助函数，定义其他后端组件如何与该头文件协作。

### Lines 241-300
```cpp
241:     return hasPTXWithAccelSMs(91, {100, 103, 110, 120, 121});
242:   }
243:
244:   // Checks support for conversions from narrow FP types to bf16x2.
245:   bool hasNarrowFPToBF16x2ConversionSupport() const {
246:     return hasPTXWithFamilySMs(92, {100, 110, 120});
247:   }
248:
249:   bool hasTensormapReplaceSupport() const {
250:     return hasPTXWithFamilySMs(90, {90, 100, 110, 120}) ||
251:            hasPTXWithFamilySMs(88, {90, 100, 101, 120}) ||
252:            hasPTXWithAccelSMs(83, {90, 100, 101, 120});
253:   }
254:
255:   bool hasTensormapReplaceElemtypeSupport(unsigned value) const {
256:     if (value >= static_cast<unsigned>(nvvm::TensormapElemType::B4x16))
257:       return hasPTXWithFamilySMs(90, {100, 110, 120}) ||
258:              hasPTXWithFamilySMs(88, {100, 101, 120}) ||
259:              hasPTXWithAccelSMs(87, {100, 101, 120});
260:
261:     return hasTensormapReplaceSupport();
262:   }
263:
264:   bool hasTensormapReplaceSwizzleAtomicitySupport() const {
265:     return hasPTXWithFamilySMs(90, {100, 110, 120}) ||
266:            hasPTXWithFamilySMs(88, {100, 101, 120}) ||
267:            hasPTXWithAccelSMs(87, {100, 101, 120});
268:   }
269:
270:   bool hasTensormapReplaceSwizzleModeSupport(unsigned value) const {
271:     if (value == static_cast<unsigned>(nvvm::TensormapSwizzleMode::SWIZZLE_96B))
272:       return hasPTXWithAccelSMs(88, {103});
273:
274:     return hasTensormapReplaceSupport();
275:   }
276:
277:   bool hasClusterLaunchControlTryCancelMulticastSupport() const {
278:     return hasPTXWithFamilySMs(90, {100, 110, 120}) ||
279:            hasPTXWithFamilySMs(88, {100, 101, 120}) ||
280:            hasPTXWithAccelSMs(86, {100, 101, 120});
281:   }
282:
283:   bool hasSetMaxNRegSupport() const {
284:     return hasPTXWithFamilySMs(90, {100, 110, 120}) ||
285:            hasPTXWithFamilySMs(88, {100, 101, 120}) ||
286:            hasPTXWithAccelSMs(86, {100, 101, 120}) ||
287:            hasPTXWithAccelSMs(80, {90});
288:   }
289:
290:   bool hasLdStmatrixBlackwellSupport() const {
291:     return hasPTXWithFamilySMs(90, {100, 110, 120}) ||
292:            hasPTXWithFamilySMs(88, {100, 101, 120}) ||
293:            hasPTXWithAccelSMs(86, {100, 101, 120});
294:   }
295:
296:   // Prior to CUDA 12.3 ptxas did not recognize that the trap instruction
297:   // terminates a basic block. Instead, it would assume that control flow
298:   // continued to the next instruction. The next instruction could be in the
299:   // block that's lexically below it. This would lead to a phantom CFG edges
300:   // being created within ptxas. This issue was fixed in CUDA 12.3. Thus, when
```
- EN: This range declares interfaces or inline helpers such as hasPTXWithAccelSMs, hasNarrowFPToBF16x2ConversionSupport, hasPTXWithFamilySMs, hasTensormapReplaceSupport, defining how other backend pieces interact with this header.
- CN: 这一段声明了 hasPTXWithAccelSMs、hasNarrowFPToBF16x2ConversionSupport、hasPTXWithFamilySMs、hasTensormapReplaceSupport 等接口或内联辅助函数，定义其他后端组件如何与该头文件协作。

### Lines 301-358
```cpp
301:   // PTX ISA versions 8.3+ we can confidently say that the bug will not be
302:   // present.
303:   bool hasPTXASUnreachableBug() const { return PTXVersion < 83; }
304:   bool hasCvtaParam() const { return SmVersion >= 70 && PTXVersion >= 77; }
305:   bool hasConvertWithStochasticRounding() const {
306:     return hasPTXWithAccelSMs(87, {100, 103});
307:   }
308:   unsigned int getFullSmVersion() const { return FullSmVersion; }
309:   unsigned int getSmVersion() const { return getFullSmVersion() / 10; }
310:   unsigned int getSmFamilyVersion() const { return getFullSmVersion() / 100; }
311:   // GPUs with "a" suffix have architecture-accelerated features that are
312:   // supported on the specified architecture only, hence such targets do not
313:   // follow the onion layer model. hasArchAccelFeatures() allows distinguishing
314:   // such GPU variants from the base GPU architecture.
315:   // - false represents non-accelerated architecture.
316:   // - true represents architecture-accelerated variant.
317:   bool hasArchAccelFeatures() const {
318:     return (getFullSmVersion() & 1) && PTXVersion >= 80;
319:   }
320:   // GPUs with 'f' suffix have architecture-accelerated features which are
321:   // portable across all future architectures under same SM major. For example,
322:   // sm_100f features will work for sm_10X*f*/sm_10X*a* future architectures.
323:   // - false represents non-family-specific architecture.
324:   // - true represents family-specific variant.
325:   bool hasFamilySpecificFeatures() const {
326:     return getFullSmVersion() % 10 == 2 ? PTXVersion >= 88
327:                                         : hasArchAccelFeatures();
328:   }
329:   // If the user did not provide a target we default to the `sm_75` target.
330:   std::string getTargetName() const {
331:     return TargetName.empty() ? "sm_75" : TargetName;
332:   }
333:   bool hasTargetName() const { return !TargetName.empty(); }
334:
335:   bool hasNativeBF16Support(int Opcode) const;
336:
337:   // Get maximum value of required alignments among the supported data types.
338:   // From the PTX ISA doc, section 8.2.3:
339:   //  The memory consistency model relates operations executed on memory
340:   //  locations with scalar data-types, which have a maximum size and alignment
341:   //  of 64 bits. Memory operations with a vector data-type are modelled as a
342:   //  set of equivalent memory operations with a scalar data-type, executed in
343:   //  an unspecified order on the elements in the vector.
344:   unsigned getMaxRequiredAlignment() const { return 8; }
345:   // Get the smallest cmpxchg word size that the hardware supports.
346:   unsigned getMinCmpXchgSizeInBits() const { return 32; }
347:
348:   unsigned getPTXVersion() const { return PTXVersion; }
349:
350:   NVPTXSubtarget &initializeSubtargetDependencies(StringRef CPU, StringRef FS);
351:   void ParseSubtargetFeatures(StringRef CPU, StringRef TuneCPU, StringRef FS);
352:
353:   void failIfClustersUnsupported(std::string const &FailureMessage) const;
354: };
355:
356: } // End llvm namespace
357:
358: #endif
```
- EN: This range declares interfaces or inline helpers such as hasPTXASUnreachableBug, hasCvtaParam, hasConvertWithStochasticRounding, hasPTXWithAccelSMs, defining how other backend pieces interact with this header.
- CN: 这一段声明了 hasPTXASUnreachableBug、hasCvtaParam、hasConvertWithStochasticRounding、hasPTXWithAccelSMs 等接口或内联辅助函数，定义其他后端组件如何与该头文件协作。

## Key Concepts / 关键概念

- EN: Subtarget objects record per-environment or per-CPU capabilities that specialize backend behavior.
  - CN: Subtarget 对象记录按环境或 CPU 细分的能力，用于特化后端行为。
- EN: NVPTX backend code reflects GPU execution concerns such as address spaces, intrinsics, and PTX-specific lowering.
  - CN: NVPTX 后端代码体现了 GPU 执行特性，例如地址空间、内建函数以及 PTX 特定降级。
- EN: Key symbols in this file include NVPTXSubtarget, anchor, ~NVPTXSubtarget, getFrameLowering, getInstrInfo, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 NVPTXSubtarget, anchor, ~NVPTXSubtarget, getFrameLowering, getInstrInfo，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `NVPTX.h`
  - `NVPTXFrameLowering.h`
  - `NVPTXISelLowering.h`
  - `NVPTXInstrInfo.h`
  - `NVPTXRegisterInfo.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/CodeGen/TargetSubtargetInfo.h`
  - `llvm/IR/DataLayout.h`
  - `llvm/IR/NVVMIntrinsicUtils.h`
  - `llvm/Support/NVPTXAddrSpace.h`
- System/standard headers / 系统或标准头文件:
  - `string`
  - `NVPTXGenSubtargetInfo.inc`
