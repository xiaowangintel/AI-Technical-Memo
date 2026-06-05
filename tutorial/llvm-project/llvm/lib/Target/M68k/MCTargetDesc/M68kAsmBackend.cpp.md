# M68kAsmBackend.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/M68k/MCTargetDesc/M68kAsmBackend.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file connects the backend to LLVM MC layer descriptors for the M68k backend.
- **用途 (CN)**: 该文件用于 M68k 后端，负责将后端连接到 LLVM MC 层描述符。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行
```cpp
   1: //===-- M68kAsmBackend.cpp - M68k Assembler Backend -------------*- C++ -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: ///
   9: /// \file
  10: /// This file contains definitions for M68k assembler backend.
  11: ///
  12: //===----------------------------------------------------------------------===//
  13: 
  14: #include "MCTargetDesc/M68kBaseInfo.h"
  15: #include "MCTargetDesc/M68kFixupKinds.h"
  16: 
  17: #include "llvm/ADT/StringSwitch.h"
  18: #include "llvm/BinaryFormat/ELF.h"
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `M68kBaseInfo.h`, `M68kFixupKinds.h`, `StringSwitch.h`, `ELF.h` that expose the LLVM and target interfaces used in later logic.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `M68kBaseInfo.h`, `M68kFixupKinds.h`, `StringSwitch.h`, `ELF.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。

### Lines 19-36 / 第 19-36 行
```cpp
  19: #include "llvm/BinaryFormat/MachO.h"
  20: #include "llvm/MC/MCAsmBackend.h"
  21: #include "llvm/MC/MCAssembler.h"
  22: #include "llvm/MC/MCELFObjectWriter.h"
  23: #include "llvm/MC/MCExpr.h"
  24: #include "llvm/MC/MCInst.h"
  25: #include "llvm/MC/MCMachObjectWriter.h"
  26: #include "llvm/MC/MCObjectWriter.h"
  27: #include "llvm/MC/MCRegisterInfo.h"
  28: #include "llvm/MC/MCSectionCOFF.h"
  29: #include "llvm/MC/MCSectionELF.h"
  30: #include "llvm/MC/MCSectionMachO.h"
  31: #include "llvm/MC/MCSubtargetInfo.h"
  32: #include "llvm/MC/MCValue.h"
  33: #include "llvm/MC/TargetRegistry.h"
  34: #include "llvm/Support/Debug.h"
  35: #include "llvm/Support/ErrorHandling.h"
  36: #include "llvm/Support/MathExtras.h"
```
- **EN**: It imports dependencies such as `MachO.h`, `MCAsmBackend.h`, `MCAssembler.h`, `MCELFObjectWriter.h`, `MCExpr.h`, `MCInst.h` that expose the LLVM and target interfaces used in later logic.
- **CN**: 它引入了 `MachO.h`, `MCAsmBackend.h`, `MCAssembler.h`, `MCELFObjectWriter.h`, `MCExpr.h`, `MCInst.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。

### Lines 37-54 / 第 37-54 行
```cpp
  37: #include "llvm/Support/raw_ostream.h"
  38: 
  39: using namespace llvm;
  40: 
  41: #define DEBUG_TYPE "M68k-asm-backend"
  42: 
  43: namespace {
  44: 
  45: class M68kAsmBackend : public MCAsmBackend {
  46:   bool Allows32BitBranch;
  47: 
  48: public:
  49:   M68kAsmBackend(const Target &T, const MCSubtargetInfo &STI)
  50:       : MCAsmBackend(llvm::endianness::big),
  51:         Allows32BitBranch(llvm::StringSwitch<bool>(STI.getCPU())
  52:                               .CasesLower({"m68020", "m68030", "m68040"}, true)
  53:                               .Default(false)) {}
  54: 
```
- **EN**: It imports dependencies such as `raw_ostream.h` that expose the LLVM and target interfaces used in later logic. This block declares or refines TableGen records such as `M68kAsmBackend`. The range implements or declares functions including `M68kAsmBackend`.
- **CN**: 它引入了 `raw_ostream.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 该代码块声明或细化了 `M68kAsmBackend` 等 TableGen 记录。 这一段实现或声明了 `M68kAsmBackend` 等函数。

### Lines 55-72 / 第 55-72 行
```cpp
  55:   void applyFixup(const MCFragment &, const MCFixup &, const MCValue &,
  56:                   uint8_t *Data, uint64_t Value, bool IsResolved) override;
  57: 
  58:   bool mayNeedRelaxation(unsigned Opcode, ArrayRef<MCOperand> Operands,
  59:                          const MCSubtargetInfo &STI) const override;
  60: 
  61:   bool fixupNeedsRelaxationAdvanced(const MCFragment &, const MCFixup &,
  62:                                     const MCValue &, uint64_t,
  63:                                     bool) const override;
  64: 
  65:   void relaxInstruction(MCInst &Inst,
  66:                         const MCSubtargetInfo &STI) const override;
  67: 
  68:   /// Returns the minimum size of a nop in bytes on this target. The assembler
  69:   /// will use this to emit excess padding in situations where the padding
  70:   /// required for simple alignment would be less than the minimum nop size.
  71:   unsigned getMinimumNopSize() const override { return 2; }
  72: 
```
- **EN**: This span continues the file's main responsibility: this file connects the backend to LLVM MC layer descriptors for the M68k backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 73-90 / 第 73-90 行
```cpp
  73:   /// Write a sequence of optimal nops to the output, covering \p Count bytes.
  74:   /// \return - true on success, false on failure
  75:   bool writeNopData(raw_ostream &OS, uint64_t Count,
  76:                     const MCSubtargetInfo *STI) const override;
  77: };
  78: } // end anonymous namespace
  79: 
  80: void M68kAsmBackend::applyFixup(const MCFragment &F, const MCFixup &Fixup,
  81:                                 const MCValue &Target, uint8_t *Data,
  82:                                 uint64_t Value, bool IsResolved) {
  83:   if (!IsResolved)
  84:     Asm->getWriter().recordRelocation(F, Fixup, Target, Value);
  85: 
  86:   unsigned Size = 1 << getFixupKindLog2Size(Fixup.getKind());
  87:   assert(Fixup.getOffset() + Size <= F.getSize() && "Invalid fixup offset!");
  88:   // Check that uppper bits are either all zeros or all ones.
  89:   // Specifically ignore overflow/underflow as long as the leakage is
  90:   // limited to the lower bits. This is to remain compatible with
```
- **EN**: The range implements or declares functions including `M68kAsmBackend::applyFixup`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `M68kAsmBackend::applyFixup` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 91-108 / 第 91-108 行
```cpp
  91:   // other assemblers.
  92:   assert(isIntN(Size * 8 + 1, static_cast<int64_t>(Value)) &&
  93:          "Value does not fit in the Fixup field");
  94: 
  95:   // Write in Big Endian
  96:   for (unsigned i = 0; i != Size; ++i)
  97:     Data[i] = uint8_t(static_cast<int64_t>(Value) >> ((Size - i - 1) * 8));
  98: }
  99: 
 100: /// cc—Carry clear      GE—Greater than or equal
 101: /// LS—Lower or same    PL—Plus
 102: /// CS—Carry set        GT—Greater than
 103: /// LT—Less than
 104: /// EQ—Equal            HI—Higher
 105: /// MI—Minus            VC—Overflow clear
 106: ///                     LE—Less than or equal
 107: /// NE—Not equal        VS—Overflow set
 108: static unsigned getRelaxedOpcodeBranch(unsigned Op) {
```
- **EN**: The range implements or declares functions including `getRelaxedOpcodeBranch`. Iteration is used to walk operands, records, or instruction-related collections. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `getRelaxedOpcodeBranch` 等函数。 这里使用迭代来遍历操作数、记录或与指令相关的集合。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 109-126 / 第 109-126 行
```cpp
 109:   switch (Op) {
 110:   default:
 111:     return Op;
 112: 
 113:   // 8 -> 16
 114:   case M68k::BRA8:
 115:     return M68k::BRA16;
 116:   case M68k::Bcc8:
 117:     return M68k::Bcc16;
 118:   case M68k::Bls8:
 119:     return M68k::Bls16;
 120:   case M68k::Blt8:
 121:     return M68k::Blt16;
 122:   case M68k::Beq8:
 123:     return M68k::Beq16;
 124:   case M68k::Bmi8:
 125:     return M68k::Bmi16;
 126:   case M68k::Bne8:
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。

### Lines 127-144 / 第 127-144 行
```cpp
 127:     return M68k::Bne16;
 128:   case M68k::Bge8:
 129:     return M68k::Bge16;
 130:   case M68k::Bcs8:
 131:     return M68k::Bcs16;
 132:   case M68k::Bpl8:
 133:     return M68k::Bpl16;
 134:   case M68k::Bgt8:
 135:     return M68k::Bgt16;
 136:   case M68k::Bhi8:
 137:     return M68k::Bhi16;
 138:   case M68k::Bvc8:
 139:     return M68k::Bvc16;
 140:   case M68k::Ble8:
 141:     return M68k::Ble16;
 142:   case M68k::Bvs8:
 143:     return M68k::Bvs16;
 144: 
```
- **EN**: This span continues the file's main responsibility: this file connects the backend to LLVM MC layer descriptors for the M68k backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 145-162 / 第 145-162 行
```cpp
 145:   // 16 -> 32
 146:   case M68k::BRA16:
 147:     return M68k::BRA32;
 148:   case M68k::Bcc16:
 149:     return M68k::Bcc32;
 150:   case M68k::Bls16:
 151:     return M68k::Bls32;
 152:   case M68k::Blt16:
 153:     return M68k::Blt32;
 154:   case M68k::Beq16:
 155:     return M68k::Beq32;
 156:   case M68k::Bmi16:
 157:     return M68k::Bmi32;
 158:   case M68k::Bne16:
 159:     return M68k::Bne32;
 160:   case M68k::Bge16:
 161:     return M68k::Bge32;
 162:   case M68k::Bcs16:
```
- **EN**: This span continues the file's main responsibility: this file connects the backend to LLVM MC layer descriptors for the M68k backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 163-180 / 第 163-180 行
```cpp
 163:     return M68k::Bcs32;
 164:   case M68k::Bpl16:
 165:     return M68k::Bpl32;
 166:   case M68k::Bgt16:
 167:     return M68k::Bgt32;
 168:   case M68k::Bhi16:
 169:     return M68k::Bhi32;
 170:   case M68k::Bvc16:
 171:     return M68k::Bvc32;
 172:   case M68k::Ble16:
 173:     return M68k::Ble32;
 174:   case M68k::Bvs16:
 175:     return M68k::Bvs32;
 176:   }
 177: }
 178: 
 179: static unsigned getRelaxedOpcode(unsigned Opcode) {
 180:   // NOTE there will be some relaxations for PCD and ARD mem for x20
```
- **EN**: The range implements or declares functions including `getRelaxedOpcode`.
- **CN**: 这一段实现或声明了 `getRelaxedOpcode` 等函数。

### Lines 181-198 / 第 181-198 行
```cpp
 181:   return getRelaxedOpcodeBranch(Opcode);
 182: }
 183: 
 184: bool M68kAsmBackend::mayNeedRelaxation(unsigned Opcode, ArrayRef<MCOperand>,
 185:                                        const MCSubtargetInfo &STI) const {
 186:   // Branches can always be relaxed in either mode.
 187:   return getRelaxedOpcode(Opcode) != Opcode;
 188: 
 189:   // NOTE will change for x20 mem
 190: }
 191: 
 192: bool M68kAsmBackend::fixupNeedsRelaxationAdvanced(const MCFragment &,
 193:                                                   const MCFixup &Fixup,
 194:                                                   const MCValue &,
 195:                                                   uint64_t UnsignedValue,
 196:                                                   bool Resolved) const {
 197:   if (!Resolved)
 198:     return true;
```
- **EN**: The range implements or declares functions including `M68kAsmBackend::mayNeedRelaxation`, `M68kAsmBackend::fixupNeedsRelaxationAdvanced`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `M68kAsmBackend::mayNeedRelaxation`, `M68kAsmBackend::fixupNeedsRelaxationAdvanced` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 199-216 / 第 199-216 行
```cpp
 199:   int64_t Value = static_cast<int64_t>(UnsignedValue);
 200: 
 201:   if (!isInt<32>(Value) || (!Allows32BitBranch && !isInt<16>(Value)))
 202:     llvm_unreachable("Cannot relax the instruction, value does not fit");
 203: 
 204:   // Relax if the value is too big for a (signed) i8
 205:   // (or signed i16 if 32 bit branches can be used). This means
 206:   // that byte-wide instructions have to matched by default
 207:   unsigned KindLog2Size = getFixupKindLog2Size(Fixup.getKind());
 208:   bool FixupFieldTooSmall = false;
 209:   if (!isInt<8>(Value) && KindLog2Size == 0)
 210:     FixupFieldTooSmall = true;
 211:   else if (!isInt<16>(Value) && KindLog2Size <= 1)
 212:     FixupFieldTooSmall = true;
 213: 
 214:   // NOTE
 215:   // A branch to the immediately following instruction automatically
 216:   // uses the 16-bit displacement format because the 8-bit
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 217-234 / 第 217-234 行
```cpp
 217:   // displacement field contains $00 (zero offset).
 218:   bool ZeroDisplacementNeedsFixup = Value == 0 && KindLog2Size == 0;
 219: 
 220:   return ZeroDisplacementNeedsFixup || FixupFieldTooSmall;
 221: }
 222: 
 223: // NOTE Can tblgen help at all here to verify there aren't other instructions
 224: // we can relax?
 225: void M68kAsmBackend::relaxInstruction(MCInst &Inst,
 226:                                       const MCSubtargetInfo &STI) const {
 227:   unsigned RelaxedOp = getRelaxedOpcode(Inst.getOpcode());
 228:   assert(RelaxedOp != Inst.getOpcode());
 229:   Inst.setOpcode(RelaxedOp);
 230: }
 231: 
 232: bool M68kAsmBackend::writeNopData(raw_ostream &OS, uint64_t Count,
 233:                                   const MCSubtargetInfo *STI) const {
 234:   // Cannot emit NOP with size being not multiple of 16 bits.
```
- **EN**: The range implements or declares functions including `M68kAsmBackend::relaxInstruction`, `M68kAsmBackend::writeNopData`. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `M68kAsmBackend::relaxInstruction`, `M68kAsmBackend::writeNopData` 等函数。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 235-252 / 第 235-252 行
```cpp
 235:   if (Count % 2 != 0)
 236:     return false;
 237: 
 238:   uint64_t NumNops = Count / 2;
 239:   for (uint64_t i = 0; i != NumNops; ++i) {
 240:     OS << "\x4E\x71";
 241:   }
 242: 
 243:   return true;
 244: }
 245: 
 246: namespace {
 247: 
 248: class M68kELFAsmBackend : public M68kAsmBackend {
 249: public:
 250:   uint8_t OSABI;
 251:   M68kELFAsmBackend(const Target &T, const MCSubtargetInfo &STI, uint8_t OSABI)
 252:       : M68kAsmBackend(T, STI), OSABI(OSABI) {}
```
- **EN**: This block declares or refines TableGen records such as `M68kELFAsmBackend`. The range implements or declares functions including `M68kELFAsmBackend`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 该代码块声明或细化了 `M68kELFAsmBackend` 等 TableGen 记录。 这一段实现或声明了 `M68kELFAsmBackend` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 253-269 / 第 253-269 行
```cpp
 253: 
 254:   std::unique_ptr<MCObjectTargetWriter>
 255:   createObjectTargetWriter() const override {
 256:     return createM68kELFObjectWriter(OSABI);
 257:   }
 258: };
 259: 
 260: } // end anonymous namespace
 261: 
 262: MCAsmBackend *llvm::createM68kAsmBackend(const Target &T,
 263:                                          const MCSubtargetInfo &STI,
 264:                                          const MCRegisterInfo &MRI,
 265:                                          const MCTargetOptions &Options) {
 266:   const Triple &TheTriple = STI.getTargetTriple();
 267:   uint8_t OSABI = MCELFObjectTargetWriter::getOSABI(TheTriple.getOS());
 268:   return new M68kELFAsmBackend(T, STI, OSABI);
 269: }
```
- **EN**: This span continues the file's main responsibility: this file connects the backend to LLVM MC layer descriptors for the M68k backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

## Key Concepts / 关键概念
- **MC layer**: Uses LLVM MC structures for encoding, printing, parsing, or disassembly. / 使用 LLVM MC 结构进行编码、打印、解析或反汇编。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。

## Dependencies / 依赖关系
- `MCTargetDesc/M68kBaseInfo.h`
- `MCTargetDesc/M68kFixupKinds.h`
- `llvm/ADT/StringSwitch.h`
- `llvm/BinaryFormat/ELF.h`
- `llvm/BinaryFormat/MachO.h`
- `llvm/MC/MCAsmBackend.h`
- `llvm/MC/MCAssembler.h`
- `llvm/MC/MCELFObjectWriter.h`
- `llvm/MC/MCExpr.h`
- `llvm/MC/MCInst.h`
- `llvm/MC/MCMachObjectWriter.h`
- `llvm/MC/MCObjectWriter.h`
- `llvm/MC/MCRegisterInfo.h`
- `llvm/MC/MCSectionCOFF.h`
- `llvm/MC/MCSectionELF.h`
- `llvm/MC/MCSectionMachO.h`
- `...` (7 more include dependencies omitted for brevity / 其余 7 个 include 依赖已省略)
