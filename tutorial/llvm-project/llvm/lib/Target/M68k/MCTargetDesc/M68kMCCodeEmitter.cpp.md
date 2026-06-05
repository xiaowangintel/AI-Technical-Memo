# M68kMCCodeEmitter.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/M68k/MCTargetDesc/M68kMCCodeEmitter.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file connects the backend to LLVM MC layer descriptors for the M68k backend.
- **用途 (CN)**: 该文件用于 M68k 后端，负责将后端连接到 LLVM MC 层描述符。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行
```cpp
   1: //===-- M68kMCCodeEmitter.cpp - Convert M68k code emitter -------*- C++ -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: ///
   9: /// \file
  10: /// This file contains definitions for M68k code emitter.
  11: ///
  12: //===----------------------------------------------------------------------===//
  13: 
  14: #include "MCTargetDesc/M68kMCCodeEmitter.h"
  15: #include "MCTargetDesc/M68kBaseInfo.h"
  16: #include "MCTargetDesc/M68kFixupKinds.h"
  17: #include "MCTargetDesc/M68kMCTargetDesc.h"
  18: 
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `M68kMCCodeEmitter.h`, `M68kBaseInfo.h`, `M68kFixupKinds.h`, `M68kMCTargetDesc.h` that expose the LLVM and target interfaces used in later logic.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `M68kMCCodeEmitter.h`, `M68kBaseInfo.h`, `M68kFixupKinds.h`, `M68kMCTargetDesc.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。

### Lines 19-36 / 第 19-36 行
```cpp
  19: #include "llvm/MC/MCCodeEmitter.h"
  20: #include "llvm/MC/MCContext.h"
  21: #include "llvm/MC/MCExpr.h"
  22: #include "llvm/MC/MCInst.h"
  23: #include "llvm/MC/MCInstrInfo.h"
  24: #include "llvm/MC/MCRegisterInfo.h"
  25: #include "llvm/MC/MCSubtargetInfo.h"
  26: #include "llvm/MC/MCSymbol.h"
  27: #include "llvm/Support/Debug.h"
  28: #include "llvm/Support/EndianStream.h"
  29: #include "llvm/Support/raw_ostream.h"
  30: #include <type_traits>
  31: 
  32: using namespace llvm;
  33: 
  34: #define DEBUG_TYPE "m68k-mccodeemitter"
  35: 
  36: namespace {
```
- **EN**: It imports dependencies such as `MCCodeEmitter.h`, `MCContext.h`, `MCExpr.h`, `MCInst.h`, `MCInstrInfo.h`, `MCRegisterInfo.h` that expose the LLVM and target interfaces used in later logic. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 它引入了 `MCCodeEmitter.h`, `MCContext.h`, `MCExpr.h`, `MCInst.h`, `MCInstrInfo.h`, `MCRegisterInfo.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 37-54 / 第 37-54 行
```cpp
  37: class M68kMCCodeEmitter : public MCCodeEmitter {
  38:   M68kMCCodeEmitter(const M68kMCCodeEmitter &) = delete;
  39:   void operator=(const M68kMCCodeEmitter &) = delete;
  40:   const MCInstrInfo &MCII;
  41:   MCContext &Ctx;
  42: 
  43:   void getBinaryCodeForInstr(const MCInst &MI, SmallVectorImpl<MCFixup> &Fixups,
  44:                              APInt &Inst, APInt &Scratch,
  45:                              const MCSubtargetInfo &STI) const;
  46: 
  47:   void getMachineOpValue(const MCInst &MI, const MCOperand &Op,
  48:                          unsigned InsertPos, APInt &Value,
  49:                          SmallVectorImpl<MCFixup> &Fixups,
  50:                          const MCSubtargetInfo &STI) const;
  51: 
  52:   template <unsigned Size>
  53:   void encodeRelocImm(const MCInst &MI, unsigned OpIdx, unsigned InsertPos,
  54:                       APInt &Value, SmallVectorImpl<MCFixup> &Fixups,
```
- **EN**: This block declares or refines TableGen records such as `M68kMCCodeEmitter`.
- **CN**: 该代码块声明或细化了 `M68kMCCodeEmitter` 等 TableGen 记录。

### Lines 55-72 / 第 55-72 行
```cpp
  55:                       const MCSubtargetInfo &STI) const;
  56: 
  57:   template <unsigned Size>
  58:   void encodePCRelImm(const MCInst &MI, unsigned OpIdx, unsigned InsertPos,
  59:                       APInt &Value, SmallVectorImpl<MCFixup> &Fixups,
  60:                       const MCSubtargetInfo &STI) const;
  61: 
  62:   void encodeFPSYSSelect(const MCInst &MI, unsigned OpIdx, unsigned InsertPos,
  63:                          APInt &Value, SmallVectorImpl<MCFixup> &Fixups,
  64:                          const MCSubtargetInfo &STI) const;
  65: 
  66:   void encodeInverseMoveMask(const MCInst &MI, unsigned OpIdx,
  67:                              unsigned InsertPos, APInt &Value,
  68:                              SmallVectorImpl<MCFixup> &Fixups,
  69:                              const MCSubtargetInfo &STI) const;
  70: 
  71: public:
  72:   M68kMCCodeEmitter(const MCInstrInfo &mcii, MCContext &ctx)
```
- **EN**: The range implements or declares functions including `M68kMCCodeEmitter`.
- **CN**: 这一段实现或声明了 `M68kMCCodeEmitter` 等函数。

### Lines 73-90 / 第 73-90 行
```cpp
  73:       : MCII(mcii), Ctx(ctx) {}
  74: 
  75:   ~M68kMCCodeEmitter() override {}
  76: 
  77:   void encodeInstruction(const MCInst &MI, SmallVectorImpl<char> &CB,
  78:                          SmallVectorImpl<MCFixup> &Fixups,
  79:                          const MCSubtargetInfo &STI) const override;
  80: };
  81: 
  82: } // end anonymous namespace
  83: 
  84: #include "M68kGenMCCodeEmitter.inc"
  85: 
  86: // Select the proper unsigned integer type from a bit size.
  87: template <unsigned Size> struct select_uint_t {
  88:   using type = typename std::conditional<
  89:       Size == 8, uint8_t,
  90:       typename std::conditional<
```
- **EN**: It imports dependencies such as `M68kGenMCCodeEmitter.inc` that expose the LLVM and target interfaces used in later logic. The range implements or declares functions including `MCII`.
- **CN**: 它引入了 `M68kGenMCCodeEmitter.inc` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段实现或声明了 `MCII` 等函数。

### Lines 91-108 / 第 91-108 行
```cpp
  91:           Size == 16, uint16_t,
  92:           typename std::conditional<Size == 32, uint32_t,
  93:                                     uint64_t>::type>::type>::type;
  94: };
  95: 
  96: // Figure out which byte we're at in big endian mode.
  97: template <unsigned Size> static unsigned getBytePosition(unsigned BitPos) {
  98:   if (Size % 16) {
  99:     return static_cast<unsigned>(BitPos / 8 + ((BitPos & 0b1111) < 8 ? 1 : -1));
 100:   } else {
 101:     assert(!(BitPos & 0b1111) && "Not aligned to word boundary?");
 102:     return BitPos / 8;
 103:   }
 104: }
 105: 
 106: static void addFixup(SmallVectorImpl<MCFixup> &Fixups, uint32_t Offset,
 107:                      const MCExpr *Value, uint16_t Kind, bool PCRel = false) {
 108:   Fixups.push_back(MCFixup::create(Offset, Value, Kind, PCRel));
```
- **EN**: The range implements or declares functions including `getBytePosition`, `addFixup`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `getBytePosition`, `addFixup` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 109-126 / 第 109-126 行
```cpp
 109: }
 110: 
 111: // We need special handlings for relocatable & pc-relative operands that are
 112: // larger than a word.
 113: // A M68k instruction is aligned by word (16 bits). That means, 32-bit
 114: // (& 64-bit) immediate values are separated into hi & lo words and placed
 115: // at lower & higher addresses, respectively. For immediate values that can
 116: // be easily expressed in TG, we explicitly rotate the word ordering like
 117: // this:
 118: // ```
 119: // (ascend (slice "$imm", 31, 16), (slice "$imm", 15, 0))
 120: // ```
 121: // For operands that call into encoder functions, we need to use the `swapWord`
 122: // function to assure the correct word ordering on LE host. Note that
 123: // M68kMCCodeEmitter does massage _byte_ ordering of the final encoded
 124: // instruction but it assumes everything aligns on word boundaries. So things
 125: // will go wrong if we don't take care of the _word_ ordering here.
 126: template <unsigned Size>
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 127-144 / 第 127-144 行
```cpp
 127: void M68kMCCodeEmitter::encodeRelocImm(const MCInst &MI, unsigned OpIdx,
 128:                                        unsigned InsertPos, APInt &Value,
 129:                                        SmallVectorImpl<MCFixup> &Fixups,
 130:                                        const MCSubtargetInfo &STI) const {
 131:   using value_t = typename select_uint_t<Size>::type;
 132:   const MCOperand &MCO = MI.getOperand(OpIdx);
 133:   if (MCO.isImm()) {
 134:     Value |= M68k::swapWord<value_t>(static_cast<value_t>(MCO.getImm()));
 135:   } else if (MCO.isExpr()) {
 136:     const MCExpr *Expr = MCO.getExpr();
 137: 
 138:     // Absolute address
 139:     int64_t Addr;
 140:     if (Expr->evaluateAsAbsolute(Addr)) {
 141:       Value |= M68k::swapWord<value_t>(static_cast<value_t>(Addr));
 142:       return;
 143:     }
 144: 
```
- **EN**: The range implements or declares functions including `M68kMCCodeEmitter::encodeRelocImm`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `M68kMCCodeEmitter::encodeRelocImm` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 145-162 / 第 145-162 行
```cpp
 145:     // Relocatable address
 146:     unsigned InsertByte = getBytePosition<Size>(InsertPos);
 147:     addFixup(Fixups, InsertByte, Expr, MCFixup::getDataKindForSize(Size / 8));
 148:   }
 149: }
 150: 
 151: template <unsigned Size>
 152: void M68kMCCodeEmitter::encodePCRelImm(const MCInst &MI, unsigned OpIdx,
 153:                                        unsigned InsertPos, APInt &Value,
 154:                                        SmallVectorImpl<MCFixup> &Fixups,
 155:                                        const MCSubtargetInfo &STI) const {
 156:   const MCOperand &MCO = MI.getOperand(OpIdx);
 157:   if (MCO.isImm()) {
 158:     using value_t = typename select_uint_t<Size>::type;
 159:     Value |= M68k::swapWord<value_t>(static_cast<value_t>(MCO.getImm()));
 160:   } else if (MCO.isExpr()) {
 161:     const MCExpr *Expr = MCO.getExpr();
 162:     unsigned InsertByte = getBytePosition<Size>(InsertPos);
```
- **EN**: The range implements or declares functions including `M68kMCCodeEmitter::encodePCRelImm`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `M68kMCCodeEmitter::encodePCRelImm` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 163-180 / 第 163-180 行
```cpp
 163: 
 164:     // Special handlings for sizes smaller than a word.
 165:     if (Size < 16) {
 166:       int LabelOffset = 0;
 167:       if (InsertPos < 16)
 168:         // If the patch point is at the first word, PC is pointing at the
 169:         // next word.
 170:         LabelOffset = InsertByte - 2;
 171:       else if (InsertByte % 2)
 172:         // Otherwise the PC is pointing at the first byte of this word.
 173:         // So we need to consider the offset between PC and the fixup byte.
 174:         LabelOffset = 1;
 175: 
 176:       if (LabelOffset)
 177:         Expr = MCBinaryExpr::createAdd(
 178:             Expr, MCConstantExpr::create(LabelOffset, Ctx), Ctx);
 179:     }
 180: 
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 181-198 / 第 181-198 行
```cpp
 181:     addFixup(Fixups, InsertByte, Expr, MCFixup::getDataKindForSize(Size / 8),
 182:              true);
 183:   }
 184: }
 185: 
 186: void M68kMCCodeEmitter::encodeFPSYSSelect(const MCInst &MI, unsigned OpIdx,
 187:                                           unsigned InsertPos, APInt &Value,
 188:                                           SmallVectorImpl<MCFixup> &Fixups,
 189:                                           const MCSubtargetInfo &STI) const {
 190:   MCRegister FPSysReg = MI.getOperand(OpIdx).getReg();
 191:   switch (FPSysReg) {
 192:   case M68k::FPC:
 193:     Value = 0b100;
 194:     break;
 195:   case M68k::FPS:
 196:     Value = 0b010;
 197:     break;
 198:   case M68k::FPIAR:
```
- **EN**: The range implements or declares functions including `M68kMCCodeEmitter::encodeFPSYSSelect`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories.
- **CN**: 这一段实现或声明了 `M68kMCCodeEmitter::encodeFPSYSSelect` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。

### Lines 199-216 / 第 199-216 行
```cpp
 199:     Value = 0b001;
 200:     break;
 201:   default:
 202:     llvm_unreachable("Unrecognized FPSYS register");
 203:   }
 204: }
 205: 
 206: void M68kMCCodeEmitter::encodeInverseMoveMask(
 207:     const MCInst &MI, unsigned OpIdx, unsigned InsertPos, APInt &Value,
 208:     SmallVectorImpl<MCFixup> &Fixups, const MCSubtargetInfo &STI) const {
 209:   const MCOperand &Op = MI.getOperand(OpIdx);
 210:   Value = llvm::reverseBits<uint16_t>((uint16_t)Op.getImm());
 211: }
 212: 
 213: void M68kMCCodeEmitter::getMachineOpValue(const MCInst &MI, const MCOperand &Op,
 214:                                           unsigned InsertPos, APInt &Value,
 215:                                           SmallVectorImpl<MCFixup> &Fixups,
 216:                                           const MCSubtargetInfo &STI) const {
```
- **EN**: The range implements or declares functions including `M68kMCCodeEmitter::encodeInverseMoveMask`, `M68kMCCodeEmitter::getMachineOpValue`. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `M68kMCCodeEmitter::encodeInverseMoveMask`, `M68kMCCodeEmitter::getMachineOpValue` 等函数。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 217-234 / 第 217-234 行
```cpp
 217:   // Register
 218:   if (Op.isReg()) {
 219:     unsigned RegNum = Op.getReg();
 220:     const auto *RI = Ctx.getRegisterInfo();
 221:     Value |= RI->getEncodingValue(RegNum);
 222:     // Setup the D/A bit
 223:     if (M68kII::isAddressRegister(RegNum))
 224:       Value |= 0b1000;
 225:   } else if (Op.isImm()) {
 226:     // Immediate
 227:     Value |= static_cast<uint64_t>(Op.getImm());
 228:   } else if (Op.isExpr()) {
 229:     // Absolute address
 230:     int64_t Addr;
 231:     if (!Op.getExpr()->evaluateAsAbsolute(Addr))
 232:       report_fatal_error("Unsupported asm expression. Only absolute address "
 233:                          "can be placed here.");
 234:     Value |= static_cast<uint64_t>(Addr);
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 235-252 / 第 235-252 行
```cpp
 235:   } else {
 236:     llvm_unreachable("Unsupported operand type");
 237:   }
 238: }
 239: 
 240: void M68kMCCodeEmitter::encodeInstruction(const MCInst &MI,
 241:                                           SmallVectorImpl<char> &CB,
 242:                                           SmallVectorImpl<MCFixup> &Fixups,
 243:                                           const MCSubtargetInfo &STI) const {
 244:   LLVM_DEBUG(dbgs() << "EncodeInstruction: " << MCII.getName(MI.getOpcode())
 245:                     << "(" << MI.getOpcode() << ")\n");
 246:   (void)MCII;
 247: 
 248:   // Try using the new method first.
 249:   APInt EncodedInst(16, 0U);
 250:   APInt Scratch(64, 0U); // One APInt word is enough.
 251:   getBinaryCodeForInstr(MI, Fixups, EncodedInst, Scratch, STI);
 252: 
```
- **EN**: The range implements or declares functions including `M68kMCCodeEmitter::encodeInstruction`. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `M68kMCCodeEmitter::encodeInstruction` 等函数。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 253-263 / 第 253-263 行
```cpp
 253:   unsigned InstSize = EncodedInst.getBitWidth();
 254:   for (unsigned i = 0; i != InstSize; i += 16)
 255:     support::endian::write<uint16_t>(
 256:         CB, static_cast<uint16_t>(EncodedInst.extractBitsAsZExtValue(16, i)),
 257:         llvm::endianness::big);
 258: }
 259: 
 260: MCCodeEmitter *llvm::createM68kMCCodeEmitter(const MCInstrInfo &MCII,
 261:                                              MCContext &Ctx) {
 262:   return new M68kMCCodeEmitter(MCII, Ctx);
 263: }
```
- **EN**: Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这里使用迭代来遍历操作数、记录或与指令相关的集合。

## Key Concepts / 关键概念
- **MC layer**: Uses LLVM MC structures for encoding, printing, parsing, or disassembly. / 使用 LLVM MC 结构进行编码、打印、解析或反汇编。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Instruction metadata**: Captures opcodes, operands, patterns, and helper routines. / 描述操作码、操作数、匹配模式和辅助例程。
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。

## Dependencies / 依赖关系
- `MCTargetDesc/M68kMCCodeEmitter.h`
- `MCTargetDesc/M68kBaseInfo.h`
- `MCTargetDesc/M68kFixupKinds.h`
- `MCTargetDesc/M68kMCTargetDesc.h`
- `llvm/MC/MCCodeEmitter.h`
- `llvm/MC/MCContext.h`
- `llvm/MC/MCExpr.h`
- `llvm/MC/MCInst.h`
- `llvm/MC/MCInstrInfo.h`
- `llvm/MC/MCRegisterInfo.h`
- `llvm/MC/MCSubtargetInfo.h`
- `llvm/MC/MCSymbol.h`
- `llvm/Support/Debug.h`
- `llvm/Support/EndianStream.h`
- `llvm/Support/raw_ostream.h`
- `type_traits`
- `...` (1 more include dependencies omitted for brevity / 其余 1 个 include 依赖已省略)
