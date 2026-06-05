# HexagonMCShuffler.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/MCTargetDesc/HexagonMCShuffler.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Implements Hexagon MC-layer target description glue.
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-50 / 第 1-50 行

```cpp
     1: //===----- HexagonMCShuffler.cpp - MC bundle shuffling --------------------===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: //
     9: // This implements the shuffling of insns inside a bundle according to the
    10: // packet formation rules of the Hexagon ISA.
    11: //
    12: //===----------------------------------------------------------------------===//
    13: 
    14: #include "MCTargetDesc/HexagonMCShuffler.h"
    15: #include "MCTargetDesc/HexagonMCInstrInfo.h"
    16: #include "MCTargetDesc/HexagonShuffler.h"
    17: #include "llvm/MC/MCInst.h"
    18: #include "llvm/MC/MCInstrDesc.h"
    19: #include "llvm/MC/MCInstrInfo.h"
    20: #include "llvm/Support/CommandLine.h"
    21: #include "llvm/Support/Debug.h"
    22: #include "llvm/Support/raw_ostream.h"
    23: #include <cassert>
    24: 
    25: #define DEBUG_TYPE "hexagon-shuffle"
    26: 
    27: using namespace llvm;
    28: 
    29: static cl::opt<bool>
    30:     DisableShuffle("disable-hexagon-shuffle", cl::Hidden, cl::init(false),
    31:                    cl::desc("Disable Hexagon instruction shuffling"));
    32: 
    33: void HexagonMCShuffler::init(MCInst &MCB) {
    34:   if (HexagonMCInstrInfo::isBundle(MCB)) {
    35:     MCInst const *Extender = nullptr;
    36:     // Copy the bundle for the shuffling.
    37:     for (const auto &I : HexagonMCInstrInfo::bundleInstructions(MCB)) {
    38:       MCInst &MI = *const_cast<MCInst *>(I.getInst());
    39:       LLVM_DEBUG(dbgs() << "Shuffling: " << MCII.getName(MI.getOpcode())
    40:                         << '\n');
    41:       assert(!HexagonMCInstrInfo::getDesc(MCII, MI).isPseudo());
    42: 
    43:       if (!HexagonMCInstrInfo::isImmext(MI)) {
    44:         append(MI, Extender, HexagonMCInstrInfo::getUnits(MCII, STI, MI));
    45:         Extender = nullptr;
    46:       } else
    47:         Extender = &MI;
    48:     }
    49:   }
    50: 
```
- EN: It imports headers such as MCTargetDesc/HexagonMCShuffler.h, MCTargetDesc/HexagonMCInstrInfo.h, MCTargetDesc/HexagonShuffler.h, llvm/MC/MCInst.h, ... (10 total), establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as DisableShuffle, HexagonMCShuffler::init, getInst, LLVM_DEBUG, ... (6 total), translating Hexagon-specific policy into reusable code paths.
- CN: 这里引入了 MCTargetDesc/HexagonMCShuffler.h, MCTargetDesc/HexagonMCInstrInfo.h, MCTargetDesc/HexagonShuffler.h, llvm/MC/MCInst.h, ... (10 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 DisableShuffle, HexagonMCShuffler::init, getInst, LLVM_DEBUG, ... (6 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

### Lines 51-100 / 第 51-100 行

```cpp
    51:   Loc = MCB.getLoc();
    52:   BundleFlags = MCB.getOperand(0).getImm();
    53: }
    54: 
    55: void HexagonMCShuffler::init(MCInst &MCB, MCInst const &AddMI,
    56:                              bool bInsertAtFront) {
    57:   if (HexagonMCInstrInfo::isBundle(MCB)) {
    58:     if (bInsertAtFront)
    59:       append(AddMI, nullptr, HexagonMCInstrInfo::getUnits(MCII, STI, AddMI));
    60:     MCInst const *Extender = nullptr;
    61:     // Copy the bundle for the shuffling.
    62:     for (auto const &I : HexagonMCInstrInfo::bundleInstructions(MCB)) {
    63:       assert(!HexagonMCInstrInfo::getDesc(MCII, *I.getInst()).isPseudo());
    64:       MCInst &MI = *const_cast<MCInst *>(I.getInst());
    65:       if (!HexagonMCInstrInfo::isImmext(MI)) {
    66:         append(MI, Extender, HexagonMCInstrInfo::getUnits(MCII, STI, MI));
    67:         Extender = nullptr;
    68:       } else
    69:         Extender = &MI;
    70:     }
    71:     if (!bInsertAtFront)
    72:       append(AddMI, nullptr, HexagonMCInstrInfo::getUnits(MCII, STI, AddMI));
    73:   }
    74: 
    75:   Loc = MCB.getLoc();
    76:   BundleFlags = MCB.getOperand(0).getImm();
    77: }
    78: 
    79: void HexagonMCShuffler::copyTo(MCInst &MCB) {
    80:   MCB.clear();
    81:   MCB.addOperand(MCOperand::createImm(BundleFlags));
    82:   MCB.setLoc(Loc);
    83:   // Copy the results into the bundle.
    84:   for (auto &I : *this) {
    85:     MCInst const &MI = I.getDesc();
    86:     MCInst const *Extender = I.getExtender();
    87:     if (Extender)
    88:       MCB.addOperand(MCOperand::createInst(Extender));
    89:     MCB.addOperand(MCOperand::createInst(&MI));
    90:   }
    91: }
    92: 
    93: bool HexagonMCShuffler::reshuffleTo(MCInst &MCB) {
    94:   if (shuffle()) {
    95:     // Copy the results into the bundle.
    96:     copyTo(MCB);
    97:     return true;
    98:   }
    99:   LLVM_DEBUG(MCB.dump());
   100:   return false;
```
- EN: It declares or implements routines such as getLoc, getOperand, HexagonMCShuffler::init, assert, ... (15 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonMCShuffler, HexagonMCInstrInfo, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getLoc, getOperand, HexagonMCShuffler::init, assert, ... (15 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonMCShuffler, HexagonMCInstrInfo，说明了它与同级后端组件的连接关系。

### Lines 101-150 / 第 101-150 行

```cpp
   101: }
   102: 
   103: bool llvm::HexagonMCShuffle(MCContext &Context, bool ReportErrors,
   104:                             MCInstrInfo const &MCII, MCSubtargetInfo const &STI,
   105:                             MCInst &MCB) {
   106:   HexagonMCShuffler MCS(Context, ReportErrors, MCII, STI, MCB);
   107: 
   108:   if (DisableShuffle)
   109:     // Ignore if user chose so.
   110:     return false;
   111: 
   112:   if (!HexagonMCInstrInfo::bundleSize(MCB)) {
   113:     // There once was a bundle:
   114:     //    BUNDLE implicit-def %d2, implicit-def %r4, implicit-def %r5,
   115:     //    implicit-def %d7, ...
   116:     //      * %d2 = IMPLICIT_DEF; flags:
   117:     //      * %d7 = IMPLICIT_DEF; flags:
   118:     // After the IMPLICIT_DEFs were removed by the asm printer, the bundle
   119:     // became empty.
   120:     LLVM_DEBUG(dbgs() << "Skipping empty bundle");
   121:     return false;
   122:   } else if (!HexagonMCInstrInfo::isBundle(MCB)) {
   123:     LLVM_DEBUG(dbgs() << "Skipping stand-alone insn");
   124:     return false;
   125:   }
   126: 
   127:   return MCS.reshuffleTo(MCB);
   128: }
   129: 
   130: bool llvm::HexagonMCShuffle(MCContext &Context, MCInstrInfo const &MCII,
   131:                             MCSubtargetInfo const &STI, MCInst &MCB,
   132:                             SmallVector<DuplexCandidate, 8> possibleDuplexes) {
   133: 
   134:   if (DisableShuffle || possibleDuplexes.size() == 0)
   135:     return false;
   136: 
   137:   if (!HexagonMCInstrInfo::bundleSize(MCB)) {
   138:     // There once was a bundle:
   139:     //    BUNDLE implicit-def %d2, implicit-def %r4, implicit-def %r5,
   140:     //    implicit-def %d7, ...
   141:     //      * %d2 = IMPLICIT_DEF; flags:
   142:     //      * %d7 = IMPLICIT_DEF; flags:
   143:     // After the IMPLICIT_DEFs were removed by the asm printer, the bundle
   144:     // became empty.
   145:     LLVM_DEBUG(dbgs() << "Skipping empty bundle");
   146:     return false;
   147:   } else if (!HexagonMCInstrInfo::isBundle(MCB)) {
   148:     LLVM_DEBUG(dbgs() << "Skipping stand-alone insn");
   149:     return false;
   150:   }
```
- EN: It declares or implements routines such as llvm::HexagonMCShuffle, MCS, LLVM_DEBUG, reshuffleTo, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonMCShuffle, HexagonMCShuffler, HexagonMCInstrInfo, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 llvm::HexagonMCShuffle, MCS, LLVM_DEBUG, reshuffleTo 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonMCShuffle, HexagonMCShuffler, HexagonMCInstrInfo，说明了它与同级后端组件的连接关系。

### Lines 151-200 / 第 151-200 行

```cpp
   151: 
   152:   bool doneShuffling = false;
   153:   while (possibleDuplexes.size() > 0 && (!doneShuffling)) {
   154:     // case of Duplex Found
   155:     DuplexCandidate duplexToTry = possibleDuplexes.pop_back_val();
   156:     MCInst Attempt(MCB);
   157:     HexagonMCInstrInfo::replaceDuplex(Context, Attempt, duplexToTry);
   158:     HexagonMCShuffler MCS(Context, false, MCII, STI, Attempt); // copy packet to the shuffler
   159:     if (MCS.size() == 1) {                     // case of one duplex
   160:       // copy the created duplex in the shuffler to the bundle
   161:       MCS.copyTo(MCB);
   162:       return false;
   163:     }
   164:     // try shuffle with this duplex
   165:     doneShuffling = MCS.reshuffleTo(MCB);
   166: 
   167:     if (doneShuffling)
   168:       break;
   169:   }
   170: 
   171:   if (!doneShuffling) {
   172:     HexagonMCShuffler MCS(Context, false, MCII, STI, MCB);
   173:     doneShuffling = MCS.reshuffleTo(MCB); // shuffle
   174:   }
   175: 
   176:   return doneShuffling;
   177: }
   178: 
   179: bool llvm::HexagonMCShuffle(MCContext &Context, MCInstrInfo const &MCII,
   180:                             MCSubtargetInfo const &STI, MCInst &MCB,
   181:                             MCInst const &AddMI, int fixupCount) {
   182:   if (!HexagonMCInstrInfo::isBundle(MCB))
   183:     return false;
   184: 
   185:   // if fixups present, make sure we don't insert too many nops that would
   186:   // later prevent an extender from being inserted.
   187:   unsigned int bundleSize = HexagonMCInstrInfo::bundleSize(MCB);
   188:   if (bundleSize >= HEXAGON_PACKET_SIZE)
   189:     return false;
   190:   bool bhasDuplex = HexagonMCInstrInfo::hasDuplex(MCII, MCB);
   191:   if (fixupCount >= 2) {
   192:     if (bhasDuplex) {
   193:       if (bundleSize >= HEXAGON_PACKET_SIZE - 1) {
   194:         return false;
   195:       }
   196:     } else {
   197:       return false;
   198:     }
   199:   } else {
   200:     if (bundleSize == HEXAGON_PACKET_SIZE - 1 && fixupCount)
```
- EN: It declares or implements routines such as pop_back_val, Attempt, HexagonMCInstrInfo::replaceDuplex, MCS, ... (9 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonMCInstrInfo, HexagonMCShuffler, HexagonMCShuffle, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 pop_back_val, Attempt, HexagonMCInstrInfo::replaceDuplex, MCS, ... (9 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonMCInstrInfo, HexagonMCShuffler, HexagonMCShuffle，说明了它与同级后端组件的连接关系。

### Lines 201-218 / 第 201-218 行

```cpp
   201:       return false;
   202:   }
   203: 
   204:   if (DisableShuffle)
   205:     return false;
   206: 
   207:   // mgl: temporary code (shuffler doesn't take into account the fact that
   208:   // a duplex takes up two slots.  for example, 3 nops can be put into a packet
   209:   // containing a duplex oversubscribing slots by 1).
   210:   unsigned maxBundleSize = (HexagonMCInstrInfo::hasImmExt(MCB))
   211:                                ? HEXAGON_PACKET_SIZE
   212:                                : HEXAGON_PACKET_SIZE - 1;
   213:   if (bhasDuplex && bundleSize >= maxBundleSize)
   214:     return false;
   215: 
   216:   HexagonMCShuffler MCS(Context, false, MCII, STI, MCB, AddMI, false);
   217:   return MCS.reshuffleTo(MCB);
   218: }
```
- EN: It declares or implements routines such as MCS, reshuffleTo, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonMCInstrInfo, HexagonMCShuffler, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 MCS, reshuffleTo 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonMCInstrInfo, HexagonMCShuffler，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- instruction semantics / 指令语义
- MC-layer target description / MC 层目标描述
- subtarget feature gating / 子目标特性控制

## Dependencies / 依赖关系

- Direct includes / 直接包含: `MCTargetDesc/HexagonMCShuffler.h, MCTargetDesc/HexagonMCInstrInfo.h, MCTargetDesc/HexagonShuffler.h, llvm/MC/MCInst.h, llvm/MC/MCInstrDesc.h, llvm/MC/MCInstrInfo.h, llvm/Support/CommandLine.h, llvm/Support/Debug.h, llvm/Support/raw_ostream.h, cassert`
- Hexagon symbols / Hexagon 符号: `HexagonMCShuffler, HexagonMCInstrInfo, HexagonShuffler, HexagonMCShuffle`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
