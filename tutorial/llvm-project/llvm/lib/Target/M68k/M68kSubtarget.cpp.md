# M68kSubtarget.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/M68k/M68kSubtarget.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file models CPU/subtarget features and scheduling properties for the M68k backend.
- **用途 (CN)**: 该文件用于 M68k 后端，负责建模 CPU/子目标特性与调度属性。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行
```cpp
   1: //===-- M68kSubtarget.cpp - M68k Subtarget Information ----------*- C++ -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: ///
   9: /// \file
  10: /// This file implements the M68k specific subclass of TargetSubtargetInfo.
  11: ///
  12: //===----------------------------------------------------------------------===//
  13: 
  14: #include "M68kSubtarget.h"
  15: #include "GISel/M68kCallLowering.h"
  16: #include "GISel/M68kLegalizerInfo.h"
  17: #include "GISel/M68kRegisterBankInfo.h"
  18: #include "M68k.h"
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `M68kSubtarget.h`, `M68kCallLowering.h`, `M68kLegalizerInfo.h`, `M68kRegisterBankInfo.h`, `M68k.h` that expose the LLVM and target interfaces used in later logic.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `M68kSubtarget.h`, `M68kCallLowering.h`, `M68kLegalizerInfo.h`, `M68kRegisterBankInfo.h`, `M68k.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。

### Lines 19-36 / 第 19-36 行
```cpp
  19: #include "M68kMachineFunction.h"
  20: #include "M68kRegisterInfo.h"
  21: #include "M68kSelectionDAGInfo.h"
  22: #include "M68kTargetMachine.h"
  23: #include "llvm/CodeGen/MachineJumpTableInfo.h"
  24: #include "llvm/IR/Attributes.h"
  25: #include "llvm/IR/Function.h"
  26: #include "llvm/MC/TargetRegistry.h"
  27: #include "llvm/Support/CommandLine.h"
  28: #include "llvm/Support/ErrorHandling.h"
  29: 
  30: using namespace llvm;
  31: 
  32: #define DEBUG_TYPE "m68k-subtarget"
  33: 
  34: #define GET_SUBTARGETINFO_TARGET_DESC
  35: #define GET_SUBTARGETINFO_CTOR
  36: #include "M68kGenSubtargetInfo.inc"
```
- **EN**: It imports dependencies such as `M68kMachineFunction.h`, `M68kRegisterInfo.h`, `M68kSelectionDAGInfo.h`, `M68kTargetMachine.h`, `MachineJumpTableInfo.h`, `Attributes.h` that expose the LLVM and target interfaces used in later logic. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file. Generated `.inc` fragments are pulled in here, a common LLVM technique for TableGen-produced code.
- **CN**: 它引入了 `M68kMachineFunction.h`, `M68kRegisterInfo.h`, `M68kSelectionDAGInfo.h`, `M68kTargetMachine.h`, `MachineJumpTableInfo.h`, `Attributes.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。 这里引入了生成的 `.inc` 片段，这是 LLVM 使用 TableGen 产物的常见方式。

### Lines 37-54 / 第 37-54 行
```cpp
  37: 
  38: extern bool FixGlobalBaseReg;
  39: 
  40: /// Select the M68k CPU for the given triple and cpu name.
  41: static StringRef selectM68kCPU(Triple TT, StringRef CPU) {
  42:   if (CPU.empty() || CPU == "generic") {
  43:     CPU = "M68000";
  44:   }
  45:   return CPU;
  46: }
  47: 
  48: void M68kSubtarget::anchor() {}
  49: 
  50: M68kSubtarget::M68kSubtarget(const Triple &TT, StringRef CPU, StringRef FS,
  51:                              const M68kTargetMachine &TM)
  52:     : M68kGenSubtargetInfo(TT, CPU, /*TuneCPU*/ CPU, FS), TM(TM),
  53:       InstrInfo(initializeSubtargetDependencies(CPU, TT, FS, TM)),
  54:       FrameLowering(*this, this->getStackAlignment()), TLInfo(TM, *this),
```
- **EN**: The range implements or declares functions including `selectM68kCPU`, `M68kSubtarget::anchor`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `selectM68kCPU`, `M68kSubtarget::anchor` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 55-72 / 第 55-72 行
```cpp
  55:       TargetTriple(TT) {
  56:   TSInfo = std::make_unique<M68kSelectionDAGInfo>();
  57: 
  58:   CallLoweringInfo.reset(new M68kCallLowering(*getTargetLowering()));
  59:   Legalizer.reset(new M68kLegalizerInfo(*this));
  60: 
  61:   auto *RBI = new M68kRegisterBankInfo(*getRegisterInfo());
  62:   RegBankInfo.reset(RBI);
  63:   InstSelector.reset(createM68kInstructionSelector(TM, *this, *RBI));
  64: }
  65: 
  66: M68kSubtarget::~M68kSubtarget() = default;
  67: 
  68: const SelectionDAGTargetInfo *M68kSubtarget::getSelectionDAGInfo() const {
  69:   return TSInfo.get();
  70: }
  71: 
  72: const CallLowering *M68kSubtarget::getCallLowering() const {
```
- **EN**: The range implements or declares functions including `TargetTriple`.
- **CN**: 这一段实现或声明了 `TargetTriple` 等函数。

### Lines 73-90 / 第 73-90 行
```cpp
  73:   return CallLoweringInfo.get();
  74: }
  75: 
  76: InstructionSelector *M68kSubtarget::getInstructionSelector() const {
  77:   return InstSelector.get();
  78: }
  79: 
  80: const LegalizerInfo *M68kSubtarget::getLegalizerInfo() const {
  81:   return Legalizer.get();
  82: }
  83: 
  84: const RegisterBankInfo *M68kSubtarget::getRegBankInfo() const {
  85:   return RegBankInfo.get();
  86: }
  87: 
  88: bool M68kSubtarget::isPositionIndependent() const {
  89:   return TM.isPositionIndependent();
  90: }
```
- **EN**: The range implements or declares functions including `M68kSubtarget::isPositionIndependent`.
- **CN**: 这一段实现或声明了 `M68kSubtarget::isPositionIndependent` 等函数。

### Lines 91-108 / 第 91-108 行
```cpp
  91: 
  92: bool M68kSubtarget::isLegalToCallImmediateAddr() const { return true; }
  93: 
  94: M68kSubtarget &M68kSubtarget::initializeSubtargetDependencies(
  95:     StringRef CPU, Triple TT, StringRef FS, const M68kTargetMachine &TM) {
  96:   std::string CPUName = selectM68kCPU(TT, CPU).str();
  97: 
  98:   // Parse features string.
  99:   ParseSubtargetFeatures(CPUName, CPUName, FS);
 100: 
 101:   // Initialize scheduling itinerary for the specified CPU.
 102:   InstrItins = getInstrItineraryForCPU(CPUName);
 103: 
 104:   stackAlignment = 8;
 105: 
 106:   return *this;
 107: }
 108: 
```
- **EN**: The range implements or declares functions including `M68kSubtarget::isLegalToCallImmediateAddr`.
- **CN**: 这一段实现或声明了 `M68kSubtarget::isLegalToCallImmediateAddr` 等函数。

### Lines 109-126 / 第 109-126 行
```cpp
 109: //===----------------------------------------------------------------------===//
 110: // Code Model
 111: //
 112: // Key assumptions:
 113: //  - Whenever possible we use pc-rel encoding since it is smaller(16 bit) than
 114: //    absolute(32 bit).
 115: //  - GOT is reachable within 16 bit offset for both Small and Medium models.
 116: //  - Code section is reachable within 16 bit offset for both models.
 117: //
 118: //  ---------------------+-------------------------+--------------------------
 119: //                       |          Small          |          Medium
 120: //                       +-------------------------+------------+-------------
 121: //                       |   Static   |    PIC     |   Static   |    PIC
 122: //  ---------------------+------------+------------+------------+-------------
 123: //                branch |   pc-rel   |   pc-rel   |   pc-rel   |   pc-rel
 124: //  ---------------------+------------+------------+------------+-------------
 125: //           call global |  absolute  |    @PLT    |  absolute  |    @PLT
 126: //  ---------------------+------------+------------+------------+-------------
```
- **EN**: This range is mainly descriptive commentary or banner text that frames the file before executable definitions begin.
- **CN**: 这一段主要是说明性注释或横幅文本，用于在可执行定义开始前说明文件背景。

### Lines 127-144 / 第 127-144 行
```cpp
 127: //         call internal |   pc-rel   |   pc-rel   |   pc-rel   |   pc-rel
 128: //  ---------------------+------------+------------+------------+-------------
 129: //            data local |   pc-rel   |   pc-rel   |  ~pc-rel   |  ^pc-rel
 130: //  ---------------------+------------+------------+------------+-------------
 131: //       data local big* |   pc-rel   |   pc-rel   |  absolute  |  @GOTOFF
 132: //  ---------------------+------------+------------+------------+-------------
 133: //           data global |   pc-rel   |  @GOTPCREL |  ~pc-rel   |  @GOTPCREL
 134: //  ---------------------+------------+------------+------------+-------------
 135: //      data global big* |   pc-rel   |  @GOTPCREL |  absolute  |  @GOTPCREL
 136: //  ---------------------+------------+------------+------------+-------------
 137: //                       |          Large          |
 138: //                       +-------------------------+
 139: //                       |   Static   |    PIC     |
 140: //  ---------------------+------------+------------+
 141: //                branch |  absolute  |   pc-rel   |
 142: //  ---------------------+------------+------------+
 143: //           call global |  absolute  |    @PLT    |
 144: //  ---------------------+------------+------------+
```
- **EN**: This range is mainly descriptive commentary or banner text that frames the file before executable definitions begin.
- **CN**: 这一段主要是说明性注释或横幅文本，用于在可执行定义开始前说明文件背景。

### Lines 145-162 / 第 145-162 行
```cpp
 145: //         call internal |  absolute  |   pc-rel   |
 146: //  ---------------------+------------+------------+
 147: //            data local |  absolute  |  @GOTOFF   |
 148: //  ---------------------+------------+------------+
 149: //       data local big* |  absolute  |  @GOTOFF   |
 150: //  ---------------------+------------+------------+
 151: //           data global |  absolute  |  @GOTOFF   |
 152: //  ---------------------+------------+------------+
 153: //      data global big* |  absolute  |  @GOTOFF   |
 154: //  ---------------------+------------+------------+
 155: //
 156: // * Big data potentially cannot be reached within 16 bit offset and requires
 157: //   special handling for old(x00 and x10) CPUs. Normally these symbols go into
 158: //   separate .ldata section which mapped after normal .data and .text, but I
 159: //   don't really know how this must be done for M68k atm... will try to dig
 160: //   this info out from GCC. For now CPUs prior to M68020 will use static ref
 161: //   for Static Model and @GOT based references for PIC.
 162: //
```
- **EN**: This range is mainly descriptive commentary or banner text that frames the file before executable definitions begin.
- **CN**: 这一段主要是说明性注释或横幅文本，用于在可执行定义开始前说明文件背景。

### Lines 163-180 / 第 163-180 行
```cpp
 163: // ~ These are absolute for older CPUs for now.
 164: // ^ These are @GOTOFF for older CPUs for now.
 165: //===----------------------------------------------------------------------===//
 166: 
 167: /// Classify a blockaddress reference for the current subtarget according to how
 168: /// we should reference it in a non-pcrel context.
 169: unsigned char M68kSubtarget::classifyBlockAddressReference() const {
 170:   switch (TM.getCodeModel()) {
 171:   default:
 172:     llvm_unreachable("Unsupported code model");
 173:   case CodeModel::Small:
 174:   case CodeModel::Kernel:
 175:   case CodeModel::Medium: {
 176:     return M68kII::MO_PC_RELATIVE_ADDRESS;
 177:   }
 178:   case CodeModel::Large: {
 179:     if (isPositionIndependent()) {
 180:       return M68kII::MO_PC_RELATIVE_ADDRESS;
```
- **EN**: The range implements or declares functions including `M68kSubtarget::classifyBlockAddressReference`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `M68kSubtarget::classifyBlockAddressReference` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 181-198 / 第 181-198 行
```cpp
 181:     } else {
 182:       return M68kII::MO_ABSOLUTE_ADDRESS;
 183:     }
 184:   }
 185:   }
 186: }
 187: 
 188: unsigned char
 189: M68kSubtarget::classifyLocalReference(const GlobalValue *GV) const {
 190:   switch (TM.getCodeModel()) {
 191:   default:
 192:     llvm_unreachable("Unsupported code model");
 193:   case CodeModel::Small:
 194:   case CodeModel::Kernel: {
 195:     return M68kII::MO_PC_RELATIVE_ADDRESS;
 196:   }
 197:   case CodeModel::Medium: {
 198:     if (isPositionIndependent()) {
```
- **EN**: The range implements or declares functions including `M68kSubtarget::classifyLocalReference`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `M68kSubtarget::classifyLocalReference` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 199-216 / 第 199-216 行
```cpp
 199:       // On M68020 and better we can fit big any data offset into dips field.
 200:       if (atLeastM68020()) {
 201:         return M68kII::MO_PC_RELATIVE_ADDRESS;
 202:       }
 203:       // Otherwise we could check the data size and make sure it will fit into
 204:       // 16 bit offset. For now we will be conservative and go with @GOTOFF
 205:       return M68kII::MO_GOTOFF;
 206:     } else {
 207:       if (atLeastM68020()) {
 208:         return M68kII::MO_PC_RELATIVE_ADDRESS;
 209:       }
 210:       return M68kII::MO_ABSOLUTE_ADDRESS;
 211:     }
 212:   }
 213:   case CodeModel::Large: {
 214:     if (isPositionIndependent()) {
 215:       return M68kII::MO_GOTOFF;
 216:     } else {
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 217-234 / 第 217-234 行
```cpp
 217:       return M68kII::MO_ABSOLUTE_ADDRESS;
 218:     }
 219:   }
 220:   }
 221: }
 222: 
 223: unsigned char M68kSubtarget::classifyExternalReference(const Module &M) const {
 224:   if (TM.shouldAssumeDSOLocal(nullptr))
 225:     return classifyLocalReference(nullptr);
 226: 
 227:   if (isPositionIndependent())
 228:     return M68kII::MO_GOTPCREL;
 229: 
 230:   return M68kII::MO_GOT;
 231: }
 232: 
 233: unsigned char
 234: M68kSubtarget::classifyGlobalReference(const GlobalValue *GV) const {
```
- **EN**: The range implements or declares functions including `M68kSubtarget::classifyExternalReference`, `M68kSubtarget::classifyGlobalReference`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `M68kSubtarget::classifyExternalReference`, `M68kSubtarget::classifyGlobalReference` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 235-252 / 第 235-252 行
```cpp
 235:   return classifyGlobalReference(GV, *GV->getParent());
 236: }
 237: 
 238: unsigned char M68kSubtarget::classifyGlobalReference(const GlobalValue *GV,
 239:                                                      const Module &M) const {
 240:   if (TM.shouldAssumeDSOLocal(GV))
 241:     return classifyLocalReference(GV);
 242: 
 243:   switch (TM.getCodeModel()) {
 244:   default:
 245:     llvm_unreachable("Unsupported code model");
 246:   case CodeModel::Small:
 247:   case CodeModel::Kernel: {
 248:     if (isPositionIndependent())
 249:       return M68kII::MO_GOTPCREL;
 250:     return M68kII::MO_PC_RELATIVE_ADDRESS;
 251:   }
 252:   case CodeModel::Medium: {
```
- **EN**: The range implements or declares functions including `M68kSubtarget::classifyGlobalReference`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `M68kSubtarget::classifyGlobalReference` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 253-270 / 第 253-270 行
```cpp
 253:     if (isPositionIndependent())
 254:       return M68kII::MO_GOTPCREL;
 255: 
 256:     if (atLeastM68020())
 257:       return M68kII::MO_PC_RELATIVE_ADDRESS;
 258: 
 259:     return M68kII::MO_ABSOLUTE_ADDRESS;
 260:   }
 261:   case CodeModel::Large: {
 262:     if (isPositionIndependent())
 263:       return M68kII::MO_GOTOFF;
 264: 
 265:     return M68kII::MO_ABSOLUTE_ADDRESS;
 266:   }
 267:   }
 268: }
 269: 
 270: unsigned M68kSubtarget::getJumpTableEncoding() const {
```
- **EN**: The range implements or declares functions including `M68kSubtarget::getJumpTableEncoding`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `M68kSubtarget::getJumpTableEncoding` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 271-288 / 第 271-288 行
```cpp
 271:   if (isPositionIndependent()) {
 272:     // The only time we want to use GOTOFF(used when with EK_Custom32) is when
 273:     // the potential delta between the jump target and table base can be larger
 274:     // than displacement field, which is True for older CPUs(16 bit disp)
 275:     // in Medium model(can have large data way beyond 16 bit).
 276:     if ((TM.getCodeModel() == CodeModel::Medium && !atLeastM68020()) ||
 277:         TM.getCodeModel() == CodeModel::Large)
 278:       return MachineJumpTableInfo::EK_Custom32;
 279: 
 280:     return MachineJumpTableInfo::EK_LabelDifference32;
 281:   }
 282: 
 283:   // In non-pic modes, just use the address of a block.
 284:   return MachineJumpTableInfo::EK_BlockAddress;
 285: }
 286: 
 287: unsigned char
 288: M68kSubtarget::classifyGlobalFunctionReference(const GlobalValue *GV) const {
```
- **EN**: The range implements or declares functions including `M68kSubtarget::classifyGlobalFunctionReference`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `M68kSubtarget::classifyGlobalFunctionReference` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 289-306 / 第 289-306 行
```cpp
 289:   return classifyGlobalFunctionReference(GV, *GV->getParent());
 290: }
 291: 
 292: unsigned char
 293: M68kSubtarget::classifyGlobalFunctionReference(const GlobalValue *GV,
 294:                                                const Module &M) const {
 295:   // local always use pc-rel referencing
 296:   if (TM.shouldAssumeDSOLocal(GV))
 297:     return M68kII::MO_NO_FLAG;
 298: 
 299:   // If the function is marked as non-lazy, generate an indirect call
 300:   // which loads from the GOT directly. This avoids run-time overhead
 301:   // at the cost of eager binding.
 302:   auto *F = dyn_cast_or_null<Function>(GV);
 303:   if (F && F->hasFnAttribute(Attribute::NonLazyBind)) {
 304:     return M68kII::MO_GOTPCREL;
 305:   }
 306: 
```
- **EN**: The range implements or declares functions including `M68kSubtarget::classifyGlobalFunctionReference`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `M68kSubtarget::classifyGlobalFunctionReference` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 307-309 / 第 307-309 行
```cpp
 307:   // Ensure that we don't emit PLT relocations when in non-pic modes.
 308:   return isPositionIndependent() ? M68kII::MO_PLT : M68kII::MO_ABSOLUTE_ADDRESS;
 309: }
```
- **EN**: This span continues the file's main responsibility: this file models CPU/subtarget features and scheduling properties for the M68k backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

## Key Concepts / 关键概念
- **SelectionDAG lowering**: Bridges LLVM IR and target-specific DAG nodes/instructions. / 连接 LLVM IR 与目标相关 DAG 节点/指令。
- **GlobalISel**: Uses legalization, register banks, and instruction selection after generic IR lowering. / 在通用 IR 降低后使用合法化、寄存器银行和指令选择。
- **MachineFunction**: Carries per-function machine-level state and basic blocks. / 保存每个函数的机器级状态和基本块。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Instruction metadata**: Captures opcodes, operands, patterns, and helper routines. / 描述操作码、操作数、匹配模式和辅助例程。
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。

## Dependencies / 依赖关系
- `M68kSubtarget.h`
- `GISel/M68kCallLowering.h`
- `GISel/M68kLegalizerInfo.h`
- `GISel/M68kRegisterBankInfo.h`
- `M68k.h`
- `M68kMachineFunction.h`
- `M68kRegisterInfo.h`
- `M68kSelectionDAGInfo.h`
- `M68kTargetMachine.h`
- `llvm/CodeGen/MachineJumpTableInfo.h`
- `llvm/IR/Attributes.h`
- `llvm/IR/Function.h`
- `llvm/MC/TargetRegistry.h`
- `llvm/Support/CommandLine.h`
- `llvm/Support/ErrorHandling.h`
- `M68kGenSubtargetInfo.inc`
