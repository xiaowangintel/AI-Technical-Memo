# NVPTXSubtarget.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/NVPTX/NVPTXSubtarget.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file implements the NVPTX specific subclass of TargetSubtarget.
- 目的（中文）: 该文件实现 LLVM 目标后端中的相关逻辑。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-40
```cpp
 1: //===- NVPTXSubtarget.cpp - NVPTX Subtarget Information -------------------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // This file implements the NVPTX specific subclass of TargetSubtarget.
10: //
11: //===----------------------------------------------------------------------===//
12:
13: #include "NVPTXSubtarget.h"
14: #include "NVPTXSelectionDAGInfo.h"
15: #include "NVPTXTargetMachine.h"
16: #include "llvm/Support/ErrorHandling.h"
17: #include "llvm/Support/FormatVariadic.h"
18:
19: using namespace llvm;
20:
21: #define DEBUG_TYPE "nvptx-subtarget"
22:
23: #define GET_SUBTARGETINFO_ENUM
24: #define GET_SUBTARGETINFO_TARGET_DESC
25: #define GET_SUBTARGETINFO_CTOR
26: #include "NVPTXGenSubtargetInfo.inc"
27:
28: static cl::opt<bool>
29:     NoF16Math("nvptx-no-f16-math", cl::Hidden,
30:               cl::desc("NVPTX Specific: Disable generation of f16 math ops."),
31:               cl::init(false));
32:
33: static cl::opt<bool> NoF32x2("nvptx-no-f32x2", cl::Hidden,
34:                              cl::desc("NVPTX Specific: Disable generation of "
35:                                       "f32x2 instructions and registers."),
36:                              cl::init(false));
37:
38: // FullSmVersion encoding helpers: SM * 10 + suffix offset
39: // (0 = base, 2 = 'f', 3 = 'a').
40: static constexpr unsigned SM(unsigned Version) { return Version * 10; }
```
- EN: This range defines command-line tuning knobs that influence backend lowering, code generation, or diagnostics.
- CN: 这一段定义命令行调优选项，用来影响后端的降级、代码生成或诊断行为。

### Lines 41-80
```cpp
41: static constexpr unsigned SMF(unsigned Version) { return SM(Version) + 2; }
42: static constexpr unsigned SMA(unsigned Version) { return SM(Version) + 3; }
43:
44: // Pin the vtable to this file.
45: void NVPTXSubtarget::anchor() {}
46:
47: // Returns the minimum PTX version required for a given SM target.
48: // This must be kept in sync with the "Supported Targets" column of the
49: // "PTX Release History" table in the PTX ISA documentation:
50: // https://docs.nvidia.com/cuda/parallel-thread-execution/index.html#release-notes-ptx-release-history
51: //
52: // Note: LLVM's minimum supported PTX version is 3.2 (see FeaturePTX in
53: // NVPTX.td), so older SMs that supported earlier PTX versions instead use 3.2
54: // as their effective minimum.
55: static unsigned getMinPTXVersionForSM(unsigned FullSmVersion) {
56:   switch (FullSmVersion) {
57:   case SM(20):
58:   case SM(21):
59:   case SM(30):
60:   case SM(35):
61:     return 32;
62:   case SM(32):
63:   case SM(50):
64:     return 40;
65:   case SM(37):
66:   case SM(52):
67:     return 41;
68:   case SM(53):
69:     return 42;
70:   case SM(60):
71:   case SM(61):
72:   case SM(62):
73:     return 50;
74:   case SM(70):
75:     return 60;
76:   case SM(72):
77:     return 61;
78:   case SM(75):
79:     return 63;
80:   case SM(80):
```
- EN: This range implements operational logic in helpers such as SMF, SMA, NVPTXSubtarget::anchor, getMinPTXVersionForSM, translating backend policy into executable code.
- CN: 这一段实现了 SMF、SMA、NVPTXSubtarget::anchor、getMinPTXVersionForSM 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 81-120
```cpp
 81:     return 70;
 82:   case SM(86):
 83:     return 71;
 84:   case SM(87):
 85:     return 74;
 86:   case SM(89):
 87:   case SM(90):
 88:     return 78;
 89:   case SMA(90):
 90:     return 80;
 91:   case SM(100):
 92:   case SMA(100):
 93:   case SM(101):
 94:   case SMA(101):
 95:     return 86;
 96:   case SM(120):
 97:   case SMA(120):
 98:     return 87;
 99:   case SMF(100):
100:   case SMF(101):
101:   case SM(103):
102:   case SMF(103):
103:   case SMA(103):
104:   case SMF(120):
105:   case SM(121):
106:   case SMF(121):
107:   case SMA(121):
108:     return 88;
109:   case SM(88):
110:   case SM(110):
111:   case SMF(110):
112:   case SMA(110):
113:     return 90;
114:   default:
115:     llvm_unreachable("Unknown SM version");
116:   }
117: }
118:
119: NVPTXSubtarget &NVPTXSubtarget::initializeSubtargetDependencies(StringRef CPU,
120:                                                                 StringRef FS) {
```
- EN: This range implements operational logic in helpers such as llvm_unreachable, translating backend policy into executable code.
- CN: 这一段实现了 llvm_unreachable 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 121-160
```cpp
121:   TargetName = std::string(CPU);
122:
123:   ParseSubtargetFeatures(getTargetName(), /*TuneCPU=*/getTargetName(), FS);
124:
125:   // Re-map SM version numbers, SmVersion carries the regular SMs which do
126:   // have relative order, while FullSmVersion allows distinguishing sm_90 from
127:   // sm_90a, which would *not* be a subset of sm_91.
128:   SmVersion = getSmVersion();
129:
130:   unsigned MinPTX = getMinPTXVersionForSM(FullSmVersion);
131:
132:   if (PTXVersion == 0) {
133:     // User didn't request a specific PTX version; use the minimum for this SM.
134:     PTXVersion = MinPTX;
135:   } else if (PTXVersion < MinPTX) {
136:     // User explicitly requested an insufficient PTX version.
137:     reportFatalUsageError(
138:         formatv("PTX version {0}.{1} does not support target '{2}'. "
139:                 "Minimum required PTX version is {3}.{4}. "
140:                 "Either remove the PTX version to use the default, "
141:                 "or increase it to at least {3}.{4}.",
142:                 PTXVersion / 10, PTXVersion % 10, getTargetName(), MinPTX / 10,
143:                 MinPTX % 10));
144:   }
145:
146:   return *this;
147: }
148:
149: NVPTXSubtarget::NVPTXSubtarget(const Triple &TT, const std::string &CPU,
150:                                const std::string &FS,
151:                                const NVPTXTargetMachine &TM)
152:     : NVPTXGenSubtargetInfo(TT, CPU, /*TuneCPU*/ CPU, FS), PTXVersion(0),
153:       FullSmVersion(200), SmVersion(getSmVersion()),
154:       InstrInfo(initializeSubtargetDependencies(CPU, FS)), TLInfo(TM, *this) {
155:   TSInfo = std::make_unique<NVPTXSelectionDAGInfo>();
156: }
157:
158: NVPTXSubtarget::~NVPTXSubtarget() = default;
159:
160: const SelectionDAGTargetInfo *NVPTXSubtarget::getSelectionDAGInfo() const {
```
- EN: This range implements operational logic in helpers such as std::string, ParseSubtargetFeatures, getSmVersion, getMinPTXVersionForSM, translating backend policy into executable code.
- CN: 这一段实现了 std::string、ParseSubtargetFeatures、getSmVersion、getMinPTXVersionForSM 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 161-200
```cpp
161:   return TSInfo.get();
162: }
163:
164: bool NVPTXSubtarget::hasPTXWithFamilySMs(unsigned PTXVersion,
165:                                          ArrayRef<unsigned> SMVersions) const {
166:   unsigned PTXVer = getPTXVersion();
167:   if (!hasFamilySpecificFeatures() || PTXVer < PTXVersion)
168:     return false;
169:
170:   unsigned SMVer = getSmVersion();
171:   return llvm::any_of(SMVersions, [&](unsigned SM) {
172:     // sm_101 is a different family, never group it with sm_10x.
173:     if (SMVer == 101 || SM == 101)
174:       return SMVer == SM &&
175:              // PTX 9.0 and later renamed sm_101 to sm_110, so sm_101 is not
176:              // supported.
177:              !(PTXVer >= 90 && SMVer == 101);
178:
179:     return getSmFamilyVersion() == SM / 10 && SMVer >= SM;
180:   });
181: }
182:
183: bool NVPTXSubtarget::hasPTXWithAccelSMs(unsigned PTXVersion,
184:                                         ArrayRef<unsigned> SMVersions) const {
185:   unsigned PTXVer = getPTXVersion();
186:   if (!hasArchAccelFeatures() || PTXVer < PTXVersion)
187:     return false;
188:
189:   unsigned SMVer = getSmVersion();
190:   return llvm::any_of(SMVersions, [&](unsigned SM) {
191:     return SMVer == SM &&
192:            // PTX 9.0 and later renamed sm_101 to sm_110, so sm_101 is not
193:            // supported.
194:            !(PTXVer >= 90 && SMVer == 101);
195:   });
196: }
197:
198: bool NVPTXSubtarget::allowFP16Math() const {
199:   return hasFP16Math() && NoF16Math == false;
200: }
```
- EN: This range implements operational logic in helpers such as get, getPTXVersion, getSmVersion, llvm::any_of, translating backend policy into executable code.
- CN: 这一段实现了 get、getPTXVersion、getSmVersion、llvm::any_of 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 201-240
```cpp
201:
202: bool NVPTXSubtarget::hasF32x2Instructions() const {
203:   return SmVersion >= 100 && PTXVersion >= 86 && !NoF32x2;
204: }
205:
206: bool NVPTXSubtarget::hasNativeBF16Support(int Opcode) const {
207:   if (!hasBF16Math())
208:     return false;
209:
210:   switch (Opcode) {
211:   // Several BF16 instructions are available on sm_90 only.
212:   case ISD::FADD:
213:   case ISD::FMUL:
214:   case ISD::FSUB:
215:   case ISD::SELECT:
216:   case ISD::SELECT_CC:
217:   case ISD::SETCC:
218:   case ISD::FEXP2:
219:   case ISD::FCEIL:
220:   case ISD::FFLOOR:
221:   case ISD::FNEARBYINT:
222:   case ISD::FRINT:
223:   case ISD::FROUNDEVEN:
224:   case ISD::FTRUNC:
225:     return getSmVersion() >= 90 && getPTXVersion() >= 78;
226:   // Several BF16 instructions are available on sm_80 only.
227:   case ISD::FMINNUM:
228:   case ISD::FMAXNUM:
229:   case ISD::FMAXNUM_IEEE:
230:   case ISD::FMINNUM_IEEE:
231:   case ISD::FMAXIMUM:
232:   case ISD::FMINIMUM:
233:     return getSmVersion() >= 80 && getPTXVersion() >= 70;
234:   }
235:   return true;
236: }
237:
238: void NVPTXSubtarget::failIfClustersUnsupported(
239:     std::string const &FailureMessage) const {
240:   if (hasClusters())
```
- EN: This range implements operational logic in helpers such as NVPTXSubtarget::hasF32x2Instructions, NVPTXSubtarget::hasNativeBF16Support, translating backend policy into executable code.
- CN: 这一段实现了 NVPTXSubtarget::hasF32x2Instructions、NVPTXSubtarget::hasNativeBF16Support 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 241-247
```cpp
241:     return;
242:
243:   report_fatal_error(formatv(
244:       "NVPTX SM architecture \"{}\" and PTX version \"{}\" do not support {}. "
245:       "Requires SM >= 90 and PTX >= 78.",
246:       getFullSmVersion(), PTXVersion, FailureMessage));
247: }
```
- EN: This range implements operational logic in helpers such as getFullSmVersion, translating backend policy into executable code.
- CN: 这一段实现了 getFullSmVersion 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: Subtarget objects record per-environment or per-CPU capabilities that specialize backend behavior.
  - CN: Subtarget 对象记录按环境或 CPU 细分的能力，用于特化后端行为。
- EN: NVPTX backend code reflects GPU execution concerns such as address spaces, intrinsics, and PTX-specific lowering.
  - CN: NVPTX 后端代码体现了 GPU 执行特性，例如地址空间、内建函数以及 PTX 特定降级。
- EN: Key symbols in this file include cl::init, SM, SMF, SMA, NVPTXSubtarget::anchor, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 cl::init, SM, SMF, SMA, NVPTXSubtarget::anchor，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `NVPTXSubtarget.h`
  - `NVPTXSelectionDAGInfo.h`
  - `NVPTXTargetMachine.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/Support/ErrorHandling.h`
  - `llvm/Support/FormatVariadic.h`
- System/standard headers / 系统或标准头文件:
  - `NVPTXGenSubtargetInfo.inc`
