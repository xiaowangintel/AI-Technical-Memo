# LanaiAsmBackend.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/Lanai/MCTargetDesc/LanaiAsmBackend.cpp`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): Implements the MC assembler backend, including fixups, relaxation, and object emission policies.
- 目的（中文）: 实现 MC 汇编后端，包括重定位修正、指令松弛和目标文件输出策略。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===-- LanaiAsmBackend.cpp - Lanai Assembler Backend ---------------------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
   9: #include "LanaiFixupKinds.h"
  10: #include "MCTargetDesc/LanaiMCTargetDesc.h"
  11: #include "llvm/MC/MCAsmBackend.h"
  12: #include "llvm/MC/MCAssembler.h"
  13: #include "llvm/MC/MCELFObjectWriter.h"
  14: #include "llvm/MC/MCObjectWriter.h"
  15: #include "llvm/MC/MCSubtargetInfo.h"
  16: #include "llvm/MC/MCValue.h"
  17: #include "llvm/Support/ErrorHandling.h"
  18: #include "llvm/Support/raw_ostream.h"
  19: 
  20: using namespace llvm;
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。

### Lines 21-40

```cpp
  21: 
  22: // Prepare value for the target space
  23: static unsigned adjustFixupValue(unsigned Kind, uint64_t Value) {
  24:   switch (Kind) {
  25:   case FK_Data_1:
  26:   case FK_Data_2:
  27:   case FK_Data_4:
  28:   case FK_Data_8:
  29:     return Value;
  30:   case Lanai::FIXUP_LANAI_21:
  31:   case Lanai::FIXUP_LANAI_21_F:
  32:   case Lanai::FIXUP_LANAI_25:
  33:   case Lanai::FIXUP_LANAI_32:
  34:   case Lanai::FIXUP_LANAI_HI16:
  35:   case Lanai::FIXUP_LANAI_LO16:
  36:     return Value;
  37:   default:
  38:     llvm_unreachable("Unknown fixup kind!");
  39:   }
  40: }
```

- EN: Function bodies or method definitions such as adjustFixupValue contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition.
- 中文: adjustFixupValue 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。

### Lines 41-60

```cpp
  41: 
  42: namespace {
  43: class LanaiAsmBackend : public MCAsmBackend {
  44:   Triple::OSType OSType;
  45: 
  46: public:
  47:   LanaiAsmBackend(const Target &T, Triple::OSType OST)
  48:       : MCAsmBackend(llvm::endianness::big), OSType(OST) {}
  49: 
  50:   void applyFixup(const MCFragment &, const MCFixup &, const MCValue &Target,
  51:                   uint8_t *Data, uint64_t Value, bool IsResolved) override;
  52: 
  53:   std::unique_ptr<MCObjectTargetWriter>
  54:   createObjectTargetWriter() const override;
  55: 
  56:   MCFixupKindInfo getFixupKindInfo(MCFixupKind Kind) const override;
  57: 
  58:   bool writeNopData(raw_ostream &OS, uint64_t Count,
  59:                     const MCSubtargetInfo *STI) const override;
  60: };
```

- EN: This chunk introduces interfaces or data structures such as LanaiAsmBackend, which organize the target-specific behavior exposed by the file. Function bodies or method definitions such as LanaiAsmBackend contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: 这一段引入了 LanaiAsmBackend 等接口或数据结构，用于组织该文件暴露的目标专用行为。 LanaiAsmBackend 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 61-80

```cpp
  61: 
  62: bool LanaiAsmBackend::writeNopData(raw_ostream &OS, uint64_t Count,
  63:                                    const MCSubtargetInfo *STI) const {
  64:   if ((Count % 4) != 0)
  65:     return false;
  66: 
  67:   for (uint64_t i = 0; i < Count; i += 4)
  68:     OS.write("\x15\0\0\0", 4);
  69: 
  70:   return true;
  71: }
  72: 
  73: void LanaiAsmBackend::applyFixup(const MCFragment &F, const MCFixup &Fixup,
  74:                                  const MCValue &Target, uint8_t *Data,
  75:                                  uint64_t Value, bool IsResolved) {
  76:   if (!IsResolved)
  77:     Asm->getWriter().recordRelocation(F, Fixup, Target, Value);
  78: 
  79:   MCFixupKind Kind = Fixup.getKind();
  80:   Value = adjustFixupValue(static_cast<unsigned>(Kind), Value);
```

- EN: Function bodies or method definitions such as writeNopData, applyFixup contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: writeNopData, applyFixup 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 81-100

```cpp
  81:   if (!Value)
  82:     return; // This value doesn't change the encoding
  83: 
  84:   // Where in the object and where the number of bytes that need
  85:   // fixing up
  86:   unsigned NumBytes = (getFixupKindInfo(Kind).TargetSize + 7) / 8;
  87:   unsigned FullSize = 4;
  88: 
  89:   // Grab current value, if any, from bits.
  90:   uint64_t CurVal = 0;
  91: 
  92:   // Load instruction and apply value
  93:   for (unsigned i = 0; i != NumBytes; ++i) {
  94:     unsigned Idx = (FullSize - 1 - i);
  95:     CurVal |= static_cast<uint64_t>(static_cast<uint8_t>(Data[Idx])) << (i * 8);
  96:   }
  97: 
  98:   uint64_t Mask =
  99:       (static_cast<uint64_t>(-1) >> (64 - getFixupKindInfo(Kind).TargetSize));
 100:   CurVal |= Value & Mask;
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 101-120

```cpp
 101: 
 102:   // Write out the fixed up bytes back to the code/data bits.
 103:   for (unsigned i = 0; i != NumBytes; ++i) {
 104:     unsigned Idx = (FullSize - 1 - i);
 105:     Data[Idx] = static_cast<uint8_t>((CurVal >> (i * 8)) & 0xff);
 106:   }
 107: }
 108: 
 109: std::unique_ptr<MCObjectTargetWriter>
 110: LanaiAsmBackend::createObjectTargetWriter() const {
 111:   return createLanaiELFObjectWriter(MCELFObjectTargetWriter::getOSABI(OSType));
 112: }
 113: 
 114: MCFixupKindInfo LanaiAsmBackend::getFixupKindInfo(MCFixupKind Kind) const {
 115:   static const MCFixupKindInfo Infos[Lanai::NumTargetFixupKinds] = {
 116:       // This table *must* be in same the order of fixup_* kinds in
 117:       // LanaiFixupKinds.h.
 118:       // Note: The number of bits indicated here are assumed to be contiguous.
 119:       //   This does not hold true for LANAI_21 and LANAI_21_F which are applied
 120:       //   to bits 0x7cffff and 0x7cfffc, respectively. Since the 'bits' counts
```

- EN: Function bodies or method definitions such as createObjectTargetWriter, getFixupKindInfo contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis.
- 中文: createObjectTargetWriter, getFixupKindInfo 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。

### Lines 121-140

```cpp
 121:       //   here are used only for cosmetic purposes, we set the size to 16 bits
 122:       //   for these 21-bit relocation as llvm/lib/MC/MCAsmStreamer.cpp checks
 123:       //   no bits are set in the fixup range.
 124:       //
 125:       // name          offset bits flags
 126:       {"FIXUP_LANAI_NONE", 0, 32, 0},
 127:       {"FIXUP_LANAI_21", 16, 16 /*21*/, 0},
 128:       {"FIXUP_LANAI_21_F", 16, 16 /*21*/, 0},
 129:       {"FIXUP_LANAI_25", 7, 25, 0},
 130:       {"FIXUP_LANAI_32", 0, 32, 0},
 131:       {"FIXUP_LANAI_HI16", 16, 16, 0},
 132:       {"FIXUP_LANAI_LO16", 16, 16, 0}};
 133: 
 134:   if (Kind < FirstTargetFixupKind)
 135:     return MCAsmBackend::getFixupKindInfo(Kind);
 136: 
 137:   assert(unsigned(Kind - FirstTargetFixupKind) < Lanai::NumTargetFixupKinds &&
 138:          "Invalid kind!");
 139:   return Infos[Kind - FirstTargetFixupKind];
 140: }
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 141-153

```cpp
 141: 
 142: } // namespace
 143: 
 144: MCAsmBackend *llvm::createLanaiAsmBackend(const Target &T,
 145:                                           const MCSubtargetInfo &STI,
 146:                                           const MCRegisterInfo & /*MRI*/,
 147:                                           const MCTargetOptions & /*Options*/) {
 148:   const Triple &TT = STI.getTargetTriple();
 149:   if (!TT.isOSBinFormatELF())
 150:     llvm_unreachable("OS not supported");
 151: 
 152:   return new LanaiAsmBackend(T, TT.getOS());
 153: }
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

## Key Concepts / 关键概念

- Fixups and relaxation / 修正与松弛
- Object emission rules / 目标文件输出规则
- Assembler backend policies / 汇编后端策略
- CPU feature modelling / CPU 特性建模
- ELF object support / ELF 目标文件支持

## Dependencies / 依赖关系

- Direct includes / 直接包含: `LanaiFixupKinds.h`, `MCTargetDesc/LanaiMCTargetDesc.h`, `llvm/MC/MCAsmBackend.h`, `llvm/MC/MCAssembler.h`, `llvm/MC/MCELFObjectWriter.h`, `llvm/MC/MCObjectWriter.h`, `llvm/MC/MCSubtargetInfo.h`, `llvm/MC/MCValue.h`
- LLVM subsystems / LLVM 子系统: LLVM MC
