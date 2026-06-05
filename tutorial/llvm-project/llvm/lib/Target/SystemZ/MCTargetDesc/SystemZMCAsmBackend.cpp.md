# SystemZMCAsmBackend.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/MCTargetDesc/SystemZMCAsmBackend.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file connects the backend to LLVM MC layer descriptors for the SystemZ backend.
- **用途 (CN)**: 该文件用于 SystemZ 后端，负责将后端连接到 LLVM MC 层描述符。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行
```cpp
   1: //===-- SystemZMCAsmBackend.cpp - SystemZ assembler backend ---------------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
   9: #include "MCTargetDesc/SystemZMCFixups.h"
  10: #include "MCTargetDesc/SystemZMCTargetDesc.h"
  11: #include "llvm/ADT/StringSwitch.h"
  12: #include "llvm/MC/MCAsmBackend.h"
  13: #include "llvm/MC/MCAssembler.h"
  14: #include "llvm/MC/MCContext.h"
  15: #include "llvm/MC/MCELFObjectWriter.h"
  16: #include "llvm/MC/MCInst.h"
  17: #include "llvm/MC/MCObjectWriter.h"
  18: #include "llvm/MC/MCSubtargetInfo.h"
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `SystemZMCFixups.h`, `SystemZMCTargetDesc.h`, `StringSwitch.h`, `MCAsmBackend.h`, `MCAssembler.h`, `MCContext.h` that expose the LLVM and target interfaces used in later logic.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `SystemZMCFixups.h`, `SystemZMCTargetDesc.h`, `StringSwitch.h`, `MCAsmBackend.h`, `MCAssembler.h`, `MCContext.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。

### Lines 19-36 / 第 19-36 行
```cpp
  19: #include "llvm/MC/MCValue.h"
  20: 
  21: using namespace llvm;
  22: 
  23: // Value is a fully-resolved relocation value: Symbol + Addend [- Pivot].
  24: // Return the bits that should be installed in a relocation field for
  25: // fixup kind Kind.
  26: static uint64_t extractBitsForFixup(MCFixupKind Kind, uint64_t Value,
  27:                                     const MCFixup &Fixup, MCContext &Ctx) {
  28:   if (Kind < FirstTargetFixupKind)
  29:     return Value;
  30: 
  31:   auto checkFixupInRange = [&](int64_t Min, int64_t Max) -> bool {
  32:     int64_t SVal = int64_t(Value);
  33:     if (SVal < Min || SVal > Max) {
  34:       Ctx.reportError(Fixup.getLoc(), "operand out of range (" + Twine(SVal) +
  35:                                           " not between " + Twine(Min) +
  36:                                           " and " + Twine(Max) + ")");
```
- **EN**: It imports dependencies such as `MCValue.h` that expose the LLVM and target interfaces used in later logic. The range implements or declares functions including `extractBitsForFixup`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 它引入了 `MCValue.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段实现或声明了 `extractBitsForFixup` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 37-54 / 第 37-54 行
```cpp
  37:       return false;
  38:     }
  39:     return true;
  40:   };
  41: 
  42:   auto handlePCRelFixupValue = [&](unsigned W) -> uint64_t {
  43:     if (Value % 2 != 0)
  44:       Ctx.reportError(Fixup.getLoc(), "Non-even PC relative offset.");
  45:     if (!checkFixupInRange(minIntN(W) * 2, maxIntN(W) * 2))
  46:       return 0;
  47:     return (int64_t)Value / 2;
  48:   };
  49: 
  50:   auto handleImmValue = [&](bool IsSigned, unsigned W) -> uint64_t {
  51:     if (!(IsSigned ? checkFixupInRange(minIntN(W), maxIntN(W))
  52:                    : checkFixupInRange(0, maxUIntN(W))))
  53:       return 0;
  54:     return Value;
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 55-72 / 第 55-72 行
```cpp
  55:   };
  56: 
  57:   switch (unsigned(Kind)) {
  58:   case SystemZ::FK_390_PC12DBL:
  59:     return handlePCRelFixupValue(12);
  60:   case SystemZ::FK_390_PC16DBL:
  61:     return handlePCRelFixupValue(16);
  62:   case SystemZ::FK_390_PC24DBL:
  63:     return handlePCRelFixupValue(24);
  64:   case SystemZ::FK_390_PC32DBL:
  65:     return handlePCRelFixupValue(32);
  66: 
  67:   case SystemZ::FK_390_TLS_CALL:
  68:     return 0;
  69: 
  70:   case SystemZ::FK_390_S8Imm:
  71:     return handleImmValue(true, 8);
  72:   case SystemZ::FK_390_S16Imm:
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。

### Lines 73-90 / 第 73-90 行
```cpp
  73:     return handleImmValue(true, 16);
  74:   case SystemZ::FK_390_S20Imm: {
  75:     Value = handleImmValue(true, 20);
  76:     // S20Imm is used only for signed 20-bit displacements.
  77:     // The high byte of a 20 bit displacement value comes first.
  78:     uint64_t DLo = Value & 0xfff;
  79:     uint64_t DHi = (Value >> 12) & 0xff;
  80:     return (DLo << 8) | DHi;
  81:   }
  82:   case SystemZ::FK_390_S32Imm:
  83:     return handleImmValue(true, 32);
  84:   case SystemZ::FK_390_U1Imm:
  85:     return handleImmValue(false, 1);
  86:   case SystemZ::FK_390_U2Imm:
  87:     return handleImmValue(false, 2);
  88:   case SystemZ::FK_390_U3Imm:
  89:     return handleImmValue(false, 3);
  90:   case SystemZ::FK_390_U4Imm:
```
- **EN**: This span continues the file's main responsibility: this file connects the backend to LLVM MC layer descriptors for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 91-108 / 第 91-108 行
```cpp
  91:     return handleImmValue(false, 4);
  92:   case SystemZ::FK_390_U8Imm:
  93:     return handleImmValue(false, 8);
  94:   case SystemZ::FK_390_U12Imm:
  95:     return handleImmValue(false, 12);
  96:   case SystemZ::FK_390_U16Imm:
  97:     return handleImmValue(false, 16);
  98:   case SystemZ::FK_390_U32Imm:
  99:     return handleImmValue(false, 32);
 100:   case SystemZ::FK_390_U48Imm:
 101:     return handleImmValue(false, 48);
 102:   }
 103: 
 104:   llvm_unreachable("Unknown fixup kind!");
 105: }
 106: 
 107: namespace {
 108: class SystemZMCAsmBackend : public MCAsmBackend {
```
- **EN**: This block declares or refines TableGen records such as `SystemZMCAsmBackend`. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 该代码块声明或细化了 `SystemZMCAsmBackend` 等 TableGen 记录。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 109-126 / 第 109-126 行
```cpp
 109: public:
 110:   SystemZMCAsmBackend() : MCAsmBackend(llvm::endianness::big) {}
 111: 
 112:   // Override MCAsmBackend
 113:   std::optional<MCFixupKind> getFixupKind(StringRef Name) const override;
 114:   MCFixupKindInfo getFixupKindInfo(MCFixupKind Kind) const override;
 115:   void applyFixup(const MCFragment &, const MCFixup &, const MCValue &Target,
 116:                   uint8_t *Data, uint64_t Value, bool IsResolved) override;
 117:   bool writeNopData(raw_ostream &OS, uint64_t Count,
 118:                     const MCSubtargetInfo *STI) const override;
 119: };
 120: } // end anonymous namespace
 121: 
 122: std::optional<MCFixupKind>
 123: SystemZMCAsmBackend::getFixupKind(StringRef Name) const {
 124:   unsigned Type = llvm::StringSwitch<unsigned>(Name)
 125: #define ELF_RELOC(X, Y) .Case(#X, Y)
 126: #include "llvm/BinaryFormat/ELFRelocs/SystemZ.def"
```
- **EN**: It imports dependencies such as `SystemZ.def` that expose the LLVM and target interfaces used in later logic. The range implements or declares functions including `SystemZMCAsmBackend`, `SystemZMCAsmBackend::getFixupKind`. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 它引入了 `SystemZ.def` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段实现或声明了 `SystemZMCAsmBackend`, `SystemZMCAsmBackend::getFixupKind` 等函数。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 127-144 / 第 127-144 行
```cpp
 127: #undef ELF_RELOC
 128: 			.Case("BFD_RELOC_NONE", ELF::R_390_NONE)
 129: 			.Case("BFD_RELOC_8", ELF::R_390_8)
 130: 			.Case("BFD_RELOC_16", ELF::R_390_16)
 131: 			.Case("BFD_RELOC_32", ELF::R_390_32)
 132: 			.Case("BFD_RELOC_64", ELF::R_390_64)
 133: 			.Default(-1u);
 134:   if (Type != -1u)
 135:     return static_cast<MCFixupKind>(FirstLiteralRelocationKind + Type);
 136:   return std::nullopt;
 137: }
 138: 
 139: MCFixupKindInfo SystemZMCAsmBackend::getFixupKindInfo(MCFixupKind Kind) const {
 140:   // Fixup kinds from .reloc directive are like R_390_NONE. They
 141:   // do not require any extra processing.
 142:   if (mc::isRelocation(Kind))
 143:     return {};
 144: 
```
- **EN**: The range implements or declares functions including `SystemZMCAsmBackend::getFixupKindInfo`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 这一段实现或声明了 `SystemZMCAsmBackend::getFixupKindInfo` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 145-162 / 第 145-162 行
```cpp
 145:   if (Kind < FirstTargetFixupKind)
 146:     return MCAsmBackend::getFixupKindInfo(Kind);
 147: 
 148:   assert(unsigned(Kind - FirstTargetFixupKind) < SystemZ::NumTargetFixupKinds &&
 149:          "Invalid kind!");
 150:   return SystemZ::MCFixupKindInfos[Kind - FirstTargetFixupKind];
 151: }
 152: 
 153: void SystemZMCAsmBackend::applyFixup(const MCFragment &F, const MCFixup &Fixup,
 154:                                      const MCValue &Target, uint8_t *Data,
 155:                                      uint64_t Value, bool IsResolved) {
 156:   if (Target.getSpecifier())
 157:     IsResolved = false;
 158:   maybeAddReloc(F, Fixup, Target, Value, IsResolved);
 159:   MCFixupKind Kind = Fixup.getKind();
 160:   if (mc::isRelocation(Kind))
 161:     return;
 162:   unsigned BitSize = getFixupKindInfo(Kind).TargetSize;
```
- **EN**: The range implements or declares functions including `SystemZMCAsmBackend::applyFixup`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `SystemZMCAsmBackend::applyFixup` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 163-180 / 第 163-180 行
```cpp
 163:   unsigned Size = (BitSize + 7) / 8;
 164: 
 165:   assert(Fixup.getOffset() + Size <= F.getSize() && "Invalid fixup offset!");
 166: 
 167:   // Big-endian insertion of Size bytes.
 168:   Value = extractBitsForFixup(Kind, Value, Fixup, getContext());
 169:   if (BitSize < 64)
 170:     Value &= ((uint64_t)1 << BitSize) - 1;
 171:   unsigned ShiftValue = (Size * 8) - 8;
 172:   for (unsigned I = 0; I != Size; ++I) {
 173:     Data[I] |= uint8_t(Value >> ShiftValue);
 174:     ShiftValue -= 8;
 175:   }
 176: }
 177: 
 178: bool SystemZMCAsmBackend::writeNopData(raw_ostream &OS, uint64_t Count,
 179:                                        const MCSubtargetInfo *STI) const {
 180:   for (uint64_t I = 0; I != Count; ++I)
```
- **EN**: The range implements or declares functions including `SystemZMCAsmBackend::writeNopData`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZMCAsmBackend::writeNopData` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 181-198 / 第 181-198 行
```cpp
 181:     OS << '\x7';
 182:   return true;
 183: }
 184: 
 185: namespace {
 186: class ELFSystemZAsmBackend : public SystemZMCAsmBackend {
 187:   uint8_t OSABI;
 188: 
 189: public:
 190:   ELFSystemZAsmBackend(uint8_t OsABI) : SystemZMCAsmBackend(), OSABI(OsABI){};
 191: 
 192:   std::unique_ptr<MCObjectTargetWriter>
 193:   createObjectTargetWriter() const override {
 194:     return createSystemZELFObjectWriter(OSABI);
 195:   }
 196: };
 197: 
 198: class GOFFSystemZAsmBackend : public SystemZMCAsmBackend {
```
- **EN**: This block declares or refines TableGen records such as `ELFSystemZAsmBackend`, `GOFFSystemZAsmBackend`. The range implements or declares functions including `ELFSystemZAsmBackend`.
- **CN**: 该代码块声明或细化了 `ELFSystemZAsmBackend`, `GOFFSystemZAsmBackend` 等 TableGen 记录。 这一段实现或声明了 `ELFSystemZAsmBackend` 等函数。

### Lines 199-216 / 第 199-216 行
```cpp
 199: public:
 200:   GOFFSystemZAsmBackend() : SystemZMCAsmBackend(){};
 201: 
 202:   std::unique_ptr<MCObjectTargetWriter>
 203:   createObjectTargetWriter() const override {
 204:     return createSystemZGOFFObjectWriter();
 205:   }
 206: };
 207: } // namespace
 208: 
 209: MCAsmBackend *llvm::createSystemZMCAsmBackend(const Target &T,
 210:                                               const MCSubtargetInfo &STI,
 211:                                               const MCRegisterInfo &MRI,
 212:                                               const MCTargetOptions &Options) {
 213:   if (STI.getTargetTriple().isOSzOS()) {
 214:     return new GOFFSystemZAsmBackend();
 215:   }
 216: 
```
- **EN**: The range implements or declares functions including `GOFFSystemZAsmBackend`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `GOFFSystemZAsmBackend` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 217-220 / 第 217-220 行
```cpp
 217:   uint8_t OSABI =
 218:       MCELFObjectTargetWriter::getOSABI(STI.getTargetTriple().getOS());
 219:   return new ELFSystemZAsmBackend(OSABI);
 220: }
```
- **EN**: This span continues the file's main responsibility: this file connects the backend to LLVM MC layer descriptors for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

## Key Concepts / 关键概念
- **MC layer**: Uses LLVM MC structures for encoding, printing, parsing, or disassembly. / 使用 LLVM MC 结构进行编码、打印、解析或反汇编。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。

## Dependencies / 依赖关系
- `MCTargetDesc/SystemZMCFixups.h`
- `MCTargetDesc/SystemZMCTargetDesc.h`
- `llvm/ADT/StringSwitch.h`
- `llvm/MC/MCAsmBackend.h`
- `llvm/MC/MCAssembler.h`
- `llvm/MC/MCContext.h`
- `llvm/MC/MCELFObjectWriter.h`
- `llvm/MC/MCInst.h`
- `llvm/MC/MCObjectWriter.h`
- `llvm/MC/MCSubtargetInfo.h`
- `llvm/MC/MCValue.h`
- `llvm/BinaryFormat/ELFRelocs/SystemZ.def`
