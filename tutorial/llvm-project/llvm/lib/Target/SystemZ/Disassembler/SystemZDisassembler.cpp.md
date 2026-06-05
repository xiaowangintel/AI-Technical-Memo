# SystemZDisassembler.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/Disassembler/SystemZDisassembler.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file decodes target machine code into MC instructions for the SystemZ backend.
- **用途 (CN)**: 该文件用于 SystemZ 后端，负责将目标机器码解码为 MC 指令。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行
```cpp
   1: //===-- SystemZDisassembler.cpp - Disassembler for SystemZ ------*- C++ -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
   9: #include "MCTargetDesc/SystemZMCTargetDesc.h"
  10: #include "TargetInfo/SystemZTargetInfo.h"
  11: #include "llvm/MC/MCDecoder.h"
  12: #include "llvm/MC/MCDecoderOps.h"
  13: #include "llvm/MC/MCDisassembler/MCDisassembler.h"
  14: #include "llvm/MC/MCInst.h"
  15: #include "llvm/MC/MCSubtargetInfo.h"
  16: #include "llvm/MC/TargetRegistry.h"
  17: #include "llvm/Support/Compiler.h"
  18: #include "llvm/Support/MathExtras.h"
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `SystemZMCTargetDesc.h`, `SystemZTargetInfo.h`, `MCDecoder.h`, `MCDecoderOps.h`, `MCDisassembler.h`, `MCInst.h` that expose the LLVM and target interfaces used in later logic.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `SystemZMCTargetDesc.h`, `SystemZTargetInfo.h`, `MCDecoder.h`, `MCDecoderOps.h`, `MCDisassembler.h`, `MCInst.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。

### Lines 19-36 / 第 19-36 行
```cpp
  19: #include <cassert>
  20: #include <cstdint>
  21: 
  22: using namespace llvm;
  23: using namespace llvm::MCD;
  24: 
  25: #define DEBUG_TYPE "systemz-disassembler"
  26: 
  27: typedef MCDisassembler::DecodeStatus DecodeStatus;
  28: 
  29: namespace {
  30: 
  31: class SystemZDisassembler : public MCDisassembler {
  32: public:
  33:   SystemZDisassembler(const MCSubtargetInfo &STI, MCContext &Ctx)
  34:     : MCDisassembler(STI, Ctx) {}
  35:   ~SystemZDisassembler() override = default;
  36: 
```
- **EN**: It imports dependencies such as `cassert`, `cstdint` that expose the LLVM and target interfaces used in later logic. This block declares or refines TableGen records such as `SystemZDisassembler`. The range implements or declares functions including `SystemZDisassembler`.
- **CN**: 它引入了 `cassert`, `cstdint` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 该代码块声明或细化了 `SystemZDisassembler` 等 TableGen 记录。 这一段实现或声明了 `SystemZDisassembler` 等函数。

### Lines 37-54 / 第 37-54 行
```cpp
  37:   DecodeStatus getInstruction(MCInst &Instr, uint64_t &Size,
  38:                               ArrayRef<uint8_t> Bytes, uint64_t Address,
  39:                               raw_ostream &CStream) const override;
  40: };
  41: 
  42: } // end anonymous namespace
  43: 
  44: static MCDisassembler *createSystemZDisassembler(const Target &T,
  45:                                                  const MCSubtargetInfo &STI,
  46:                                                  MCContext &Ctx) {
  47:   return new SystemZDisassembler(STI, Ctx);
  48: }
  49: 
  50: // NOLINTNEXTLINE(readability-identifier-naming)
  51: extern "C" LLVM_ABI LLVM_EXTERNAL_VISIBILITY void
  52: LLVMInitializeSystemZDisassembler() {
  53:   // Register the disassembler.
  54:   TargetRegistry::RegisterMCDisassembler(getTheSystemZTarget(),
```
- **EN**: This span continues the file's main responsibility: this file decodes target machine code into MC instructions for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 55-72 / 第 55-72 行
```cpp
  55:                                          createSystemZDisassembler);
  56: }
  57: 
  58: /// tryAddingSymbolicOperand - trys to add a symbolic operand in place of the
  59: /// immediate Value in the MCInst.
  60: ///
  61: /// @param Value      - The immediate Value, has had any PC adjustment made by
  62: ///                     the caller.
  63: /// @param isBranch   - If the instruction is a branch instruction
  64: /// @param Address    - The starting address of the instruction
  65: /// @param Offset     - The byte offset to this immediate in the instruction
  66: /// @param Width      - The byte width of this immediate in the instruction
  67: ///
  68: /// If the getOpInfo() function was set when setupForSymbolicDisassembly() was
  69: /// called then that function is called to get any symbolic information for the
  70: /// immediate in the instruction using the Address, Offset and Width.  If that
  71: /// returns non-zero then the symbolic information it returns is used to create
  72: /// an MCExpr and that is added as an operand to the MCInst.  If getOpInfo()
```
- **EN**: This span continues the file's main responsibility: this file decodes target machine code into MC instructions for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 73-90 / 第 73-90 行
```cpp
  73: /// returns zero and isBranch is true then a symbol look up for immediate Value
  74: /// is done and if a symbol is found an MCExpr is created with that, else
  75: /// an MCExpr with the immediate Value is created.  This function returns true
  76: /// if it adds an operand to the MCInst and false otherwise.
  77: static bool tryAddingSymbolicOperand(int64_t Value, bool IsBranch,
  78:                                      uint64_t Address, uint64_t Offset,
  79:                                      uint64_t Width, MCInst &MI,
  80:                                      const MCDisassembler *Decoder) {
  81:   return Decoder->tryAddingSymbolicOperand(MI, Value, Address, IsBranch, Offset,
  82:                                            Width, /*InstSize=*/0);
  83: }
  84: 
  85: static DecodeStatus decodeRegisterClass(MCInst &Inst, uint64_t RegNo,
  86:                                         const unsigned *Regs, unsigned Size,
  87:                                         bool IsAddr = false) {
  88:   assert(RegNo < Size && "Invalid register");
  89:   if (IsAddr && RegNo == 0) {
  90:     RegNo = SystemZ::NoRegister;
```
- **EN**: The range implements or declares functions including `tryAddingSymbolicOperand`, `decodeRegisterClass`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `tryAddingSymbolicOperand`, `decodeRegisterClass` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 91-108 / 第 91-108 行
```cpp
  91:   } else {
  92:     RegNo = Regs[RegNo];
  93:     if (RegNo == 0)
  94:       return MCDisassembler::Fail;
  95:   }
  96:   Inst.addOperand(MCOperand::createReg(RegNo));
  97:   return MCDisassembler::Success;
  98: }
  99: 
 100: static DecodeStatus DecodeGR32BitRegisterClass(MCInst &Inst, uint64_t RegNo,
 101:                                                uint64_t Address,
 102:                                                const MCDisassembler *Decoder) {
 103:   return decodeRegisterClass(Inst, RegNo, SystemZMC::GR32Regs, 16);
 104: }
 105: 
 106: static DecodeStatus DecodeGRH32BitRegisterClass(MCInst &Inst, uint64_t RegNo,
 107:                                                 uint64_t Address,
 108:                                                 const MCDisassembler *Decoder) {
```
- **EN**: The range implements or declares functions including `DecodeGR32BitRegisterClass`, `DecodeGRH32BitRegisterClass`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `DecodeGR32BitRegisterClass`, `DecodeGRH32BitRegisterClass` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 109-126 / 第 109-126 行
```cpp
 109:   return decodeRegisterClass(Inst, RegNo, SystemZMC::GRH32Regs, 16);
 110: }
 111: 
 112: static DecodeStatus DecodeGR64BitRegisterClass(MCInst &Inst, uint64_t RegNo,
 113:                                                uint64_t Address,
 114:                                                const MCDisassembler *Decoder) {
 115:   return decodeRegisterClass(Inst, RegNo, SystemZMC::GR64Regs, 16);
 116: }
 117: 
 118: static DecodeStatus DecodeGR128BitRegisterClass(MCInst &Inst, uint64_t RegNo,
 119:                                                 uint64_t Address,
 120:                                                 const MCDisassembler *Decoder) {
 121:   return decodeRegisterClass(Inst, RegNo, SystemZMC::GR128Regs, 16);
 122: }
 123: 
 124: static DecodeStatus
 125: DecodeADDR32BitRegisterClass(MCInst &Inst, uint64_t RegNo, uint64_t Address,
 126:                              const MCDisassembler *Decoder) {
```
- **EN**: The range implements or declares functions including `DecodeGR64BitRegisterClass`, `DecodeGR128BitRegisterClass`, `DecodeADDR32BitRegisterClass`.
- **CN**: 这一段实现或声明了 `DecodeGR64BitRegisterClass`, `DecodeGR128BitRegisterClass`, `DecodeADDR32BitRegisterClass` 等函数。

### Lines 127-144 / 第 127-144 行
```cpp
 127:   return decodeRegisterClass(Inst, RegNo, SystemZMC::GR32Regs, 16, true);
 128: }
 129: 
 130: static DecodeStatus
 131: DecodeADDR64BitRegisterClass(MCInst &Inst, uint64_t RegNo, uint64_t Address,
 132:                              const MCDisassembler *Decoder) {
 133:   return decodeRegisterClass(Inst, RegNo, SystemZMC::GR64Regs, 16, true);
 134: }
 135: 
 136: static DecodeStatus DecodeFP32BitRegisterClass(MCInst &Inst, uint64_t RegNo,
 137:                                                uint64_t Address,
 138:                                                const MCDisassembler *Decoder) {
 139:   return decodeRegisterClass(Inst, RegNo, SystemZMC::FP32Regs, 16);
 140: }
 141: 
 142: static DecodeStatus DecodeFP64BitRegisterClass(MCInst &Inst, uint64_t RegNo,
 143:                                                uint64_t Address,
 144:                                                const MCDisassembler *Decoder) {
```
- **EN**: The range implements or declares functions including `DecodeADDR64BitRegisterClass`, `DecodeFP32BitRegisterClass`, `DecodeFP64BitRegisterClass`.
- **CN**: 这一段实现或声明了 `DecodeADDR64BitRegisterClass`, `DecodeFP32BitRegisterClass`, `DecodeFP64BitRegisterClass` 等函数。

### Lines 145-162 / 第 145-162 行
```cpp
 145:   return decodeRegisterClass(Inst, RegNo, SystemZMC::FP64Regs, 16);
 146: }
 147: 
 148: static DecodeStatus DecodeFP128BitRegisterClass(MCInst &Inst, uint64_t RegNo,
 149:                                                 uint64_t Address,
 150:                                                 const MCDisassembler *Decoder) {
 151:   return decodeRegisterClass(Inst, RegNo, SystemZMC::FP128Regs, 16);
 152: }
 153: 
 154: static DecodeStatus DecodeVR32BitRegisterClass(MCInst &Inst, uint64_t RegNo,
 155:                                                uint64_t Address,
 156:                                                const MCDisassembler *Decoder) {
 157:   return decodeRegisterClass(Inst, RegNo, SystemZMC::VR32Regs, 32);
 158: }
 159: 
 160: static DecodeStatus DecodeVR64BitRegisterClass(MCInst &Inst, uint64_t RegNo,
 161:                                                uint64_t Address,
 162:                                                const MCDisassembler *Decoder) {
```
- **EN**: The range implements or declares functions including `DecodeFP128BitRegisterClass`, `DecodeVR32BitRegisterClass`, `DecodeVR64BitRegisterClass`.
- **CN**: 这一段实现或声明了 `DecodeFP128BitRegisterClass`, `DecodeVR32BitRegisterClass`, `DecodeVR64BitRegisterClass` 等函数。

### Lines 163-180 / 第 163-180 行
```cpp
 163:   return decodeRegisterClass(Inst, RegNo, SystemZMC::VR64Regs, 32);
 164: }
 165: 
 166: static DecodeStatus DecodeVR128BitRegisterClass(MCInst &Inst, uint64_t RegNo,
 167:                                                 uint64_t Address,
 168:                                                 const MCDisassembler *Decoder) {
 169:   return decodeRegisterClass(Inst, RegNo, SystemZMC::VR128Regs, 32);
 170: }
 171: 
 172: static DecodeStatus DecodeAR32BitRegisterClass(MCInst &Inst, uint64_t RegNo,
 173:                                                uint64_t Address,
 174:                                                const MCDisassembler *Decoder) {
 175:   return decodeRegisterClass(Inst, RegNo, SystemZMC::AR32Regs, 16);
 176: }
 177: 
 178: static DecodeStatus DecodeCR64BitRegisterClass(MCInst &Inst, uint64_t RegNo,
 179:                                                uint64_t Address,
 180:                                                const MCDisassembler *Decoder) {
```
- **EN**: The range implements or declares functions including `DecodeVR128BitRegisterClass`, `DecodeAR32BitRegisterClass`, `DecodeCR64BitRegisterClass`.
- **CN**: 这一段实现或声明了 `DecodeVR128BitRegisterClass`, `DecodeAR32BitRegisterClass`, `DecodeCR64BitRegisterClass` 等函数。

### Lines 181-198 / 第 181-198 行
```cpp
 181:   return decodeRegisterClass(Inst, RegNo, SystemZMC::CR64Regs, 16);
 182: }
 183: 
 184: template<unsigned N>
 185: static DecodeStatus decodeUImmOperand(MCInst &Inst, uint64_t Imm) {
 186:   if (!isUInt<N>(Imm))
 187:     return MCDisassembler::Fail;
 188:   Inst.addOperand(MCOperand::createImm(Imm));
 189:   return MCDisassembler::Success;
 190: }
 191: 
 192: template<unsigned N>
 193: static DecodeStatus decodeSImmOperand(MCInst &Inst, uint64_t Imm) {
 194:   if (!isUInt<N>(Imm))
 195:     return MCDisassembler::Fail;
 196:   Inst.addOperand(MCOperand::createImm(SignExtend64<N>(Imm)));
 197:   return MCDisassembler::Success;
 198: }
```
- **EN**: The range implements or declares functions including `decodeUImmOperand`, `decodeSImmOperand`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `decodeUImmOperand`, `decodeSImmOperand` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 199-216 / 第 199-216 行
```cpp
 199: 
 200: static DecodeStatus decodeU1ImmOperand(MCInst &Inst, uint64_t Imm,
 201:                                        uint64_t Address,
 202:                                        const MCDisassembler *Decoder) {
 203:   return decodeUImmOperand<1>(Inst, Imm);
 204: }
 205: 
 206: static DecodeStatus decodeU2ImmOperand(MCInst &Inst, uint64_t Imm,
 207:                                        uint64_t Address,
 208:                                        const MCDisassembler *Decoder) {
 209:   return decodeUImmOperand<2>(Inst, Imm);
 210: }
 211: 
 212: static DecodeStatus decodeU3ImmOperand(MCInst &Inst, uint64_t Imm,
 213:                                        uint64_t Address,
 214:                                        const MCDisassembler *Decoder) {
 215:   return decodeUImmOperand<3>(Inst, Imm);
 216: }
```
- **EN**: The range implements or declares functions including `decodeU1ImmOperand`, `decodeU2ImmOperand`, `decodeU3ImmOperand`.
- **CN**: 这一段实现或声明了 `decodeU1ImmOperand`, `decodeU2ImmOperand`, `decodeU3ImmOperand` 等函数。

### Lines 217-234 / 第 217-234 行
```cpp
 217: 
 218: static DecodeStatus decodeU4ImmOperand(MCInst &Inst, uint64_t Imm,
 219:                                        uint64_t Address,
 220:                                        const MCDisassembler *Decoder) {
 221:   return decodeUImmOperand<4>(Inst, Imm);
 222: }
 223: 
 224: static DecodeStatus decodeU8ImmOperand(MCInst &Inst, uint64_t Imm,
 225:                                        uint64_t Address,
 226:                                        const MCDisassembler *Decoder) {
 227:   return decodeUImmOperand<8>(Inst, Imm);
 228: }
 229: 
 230: static DecodeStatus decodeU12ImmOperand(MCInst &Inst, uint64_t Imm,
 231:                                         uint64_t Address,
 232:                                         const MCDisassembler *Decoder) {
 233:   return decodeUImmOperand<12>(Inst, Imm);
 234: }
```
- **EN**: The range implements or declares functions including `decodeU4ImmOperand`, `decodeU8ImmOperand`, `decodeU12ImmOperand`.
- **CN**: 这一段实现或声明了 `decodeU4ImmOperand`, `decodeU8ImmOperand`, `decodeU12ImmOperand` 等函数。

### Lines 235-252 / 第 235-252 行
```cpp
 235: 
 236: static DecodeStatus decodeU16ImmOperand(MCInst &Inst, uint64_t Imm,
 237:                                         uint64_t Address,
 238:                                         const MCDisassembler *Decoder) {
 239:   return decodeUImmOperand<16>(Inst, Imm);
 240: }
 241: 
 242: static DecodeStatus decodeU32ImmOperand(MCInst &Inst, uint64_t Imm,
 243:                                         uint64_t Address,
 244:                                         const MCDisassembler *Decoder) {
 245:   return decodeUImmOperand<32>(Inst, Imm);
 246: }
 247: 
 248: static DecodeStatus decodeS8ImmOperand(MCInst &Inst, uint64_t Imm,
 249:                                        uint64_t Address,
 250:                                        const MCDisassembler *Decoder) {
 251:   return decodeSImmOperand<8>(Inst, Imm);
 252: }
```
- **EN**: The range implements or declares functions including `decodeU16ImmOperand`, `decodeU32ImmOperand`, `decodeS8ImmOperand`.
- **CN**: 这一段实现或声明了 `decodeU16ImmOperand`, `decodeU32ImmOperand`, `decodeS8ImmOperand` 等函数。

### Lines 253-270 / 第 253-270 行
```cpp
 253: 
 254: static DecodeStatus decodeS16ImmOperand(MCInst &Inst, uint64_t Imm,
 255:                                         uint64_t Address,
 256:                                         const MCDisassembler *Decoder) {
 257:   return decodeSImmOperand<16>(Inst, Imm);
 258: }
 259: 
 260: static DecodeStatus decodeS20ImmOperand(MCInst &Inst, uint64_t Imm,
 261:                                         uint64_t Address,
 262:                                         const MCDisassembler *Decoder) {
 263:   return decodeSImmOperand<20>(Inst, Imm);
 264: }
 265: 
 266: static DecodeStatus decodeS32ImmOperand(MCInst &Inst, uint64_t Imm,
 267:                                         uint64_t Address,
 268:                                         const MCDisassembler *Decoder) {
 269:   return decodeSImmOperand<32>(Inst, Imm);
 270: }
```
- **EN**: The range implements or declares functions including `decodeS16ImmOperand`, `decodeS20ImmOperand`, `decodeS32ImmOperand`.
- **CN**: 这一段实现或声明了 `decodeS16ImmOperand`, `decodeS20ImmOperand`, `decodeS32ImmOperand` 等函数。

### Lines 271-288 / 第 271-288 行
```cpp
 271: 
 272: template <unsigned N>
 273: static DecodeStatus decodeLenOperand(MCInst &Inst, uint64_t Imm,
 274:                                      uint64_t Address,
 275:                                      const MCDisassembler *Decoder) {
 276:   if (!isUInt<N>(Imm))
 277:     return MCDisassembler::Fail;
 278:   Inst.addOperand(MCOperand::createImm(Imm + 1));
 279:   return MCDisassembler::Success;
 280: }
 281: 
 282: template <unsigned N>
 283: static DecodeStatus decodePCDBLOperand(MCInst &Inst, uint64_t Imm,
 284:                                        uint64_t Address, bool isBranch,
 285:                                        const MCDisassembler *Decoder) {
 286:   assert(isUInt<N>(Imm) && "Invalid PC-relative offset");
 287:   uint64_t Value = SignExtend64<N>(Imm) * 2;
 288: 
```
- **EN**: The range implements or declares functions including `decodeLenOperand`, `decodePCDBLOperand`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `decodeLenOperand`, `decodePCDBLOperand` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 289-306 / 第 289-306 行
```cpp
 289:   if (!tryAddingSymbolicOperand(Value + Address, isBranch, Address, 2, N / 8,
 290:                                 Inst, Decoder))
 291:     Inst.addOperand(MCOperand::createImm(Value));
 292: 
 293:   return MCDisassembler::Success;
 294: }
 295: 
 296: static DecodeStatus decodePC12DBLBranchOperand(MCInst &Inst, uint64_t Imm,
 297:                                                uint64_t Address,
 298:                                                const MCDisassembler *Decoder) {
 299:   return decodePCDBLOperand<12>(Inst, Imm, Address, true, Decoder);
 300: }
 301: 
 302: static DecodeStatus decodePC16DBLBranchOperand(MCInst &Inst, uint64_t Imm,
 303:                                                uint64_t Address,
 304:                                                const MCDisassembler *Decoder) {
 305:   return decodePCDBLOperand<16>(Inst, Imm, Address, true, Decoder);
 306: }
```
- **EN**: The range implements or declares functions including `decodePC12DBLBranchOperand`, `decodePC16DBLBranchOperand`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `decodePC12DBLBranchOperand`, `decodePC16DBLBranchOperand` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 307-324 / 第 307-324 行
```cpp
 307: 
 308: static DecodeStatus decodePC24DBLBranchOperand(MCInst &Inst, uint64_t Imm,
 309:                                                uint64_t Address,
 310:                                                const MCDisassembler *Decoder) {
 311:   return decodePCDBLOperand<24>(Inst, Imm, Address, true, Decoder);
 312: }
 313: 
 314: static DecodeStatus decodePC32DBLBranchOperand(MCInst &Inst, uint64_t Imm,
 315:                                                uint64_t Address,
 316:                                                const MCDisassembler *Decoder) {
 317:   return decodePCDBLOperand<32>(Inst, Imm, Address, true, Decoder);
 318: }
 319: 
 320: static DecodeStatus decodePC32DBLOperand(MCInst &Inst, uint64_t Imm,
 321:                                          uint64_t Address,
 322:                                          const MCDisassembler *Decoder) {
 323:   return decodePCDBLOperand<32>(Inst, Imm, Address, false, Decoder);
 324: }
```
- **EN**: The range implements or declares functions including `decodePC24DBLBranchOperand`, `decodePC32DBLBranchOperand`, `decodePC32DBLOperand`.
- **CN**: 这一段实现或声明了 `decodePC24DBLBranchOperand`, `decodePC32DBLBranchOperand`, `decodePC32DBLOperand` 等函数。

### Lines 325-342 / 第 325-342 行
```cpp
 325: 
 326: #include "SystemZGenDisassemblerTables.inc"
 327: 
 328: DecodeStatus SystemZDisassembler::getInstruction(MCInst &MI, uint64_t &Size,
 329:                                                  ArrayRef<uint8_t> Bytes,
 330:                                                  uint64_t Address,
 331:                                                  raw_ostream &CS) const {
 332:   CommentStream = &CS;
 333: 
 334:   // Get the first two bytes of the instruction.
 335:   Size = 0;
 336:   if (Bytes.size() < 2)
 337:     return MCDisassembler::Fail;
 338: 
 339:   // The top 2 bits of the first byte specify the size.
 340:   const uint8_t *Table;
 341:   if (Bytes[0] < 0x40) {
 342:     Size = 2;
```
- **EN**: It imports dependencies such as `SystemZGenDisassemblerTables.inc` that expose the LLVM and target interfaces used in later logic. The range implements or declares functions including `SystemZDisassembler::getInstruction`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 它引入了 `SystemZGenDisassemblerTables.inc` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段实现或声明了 `SystemZDisassembler::getInstruction` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 343-360 / 第 343-360 行
```cpp
 343:     Table = DecoderTable16;
 344:   } else if (Bytes[0] < 0xc0) {
 345:     Size = 4;
 346:     Table = DecoderTable32;
 347:   } else {
 348:     Size = 6;
 349:     Table = DecoderTable48;
 350:   }
 351: 
 352:   // Read any remaining bytes.
 353:   if (Bytes.size() < Size) {
 354:     Size = Bytes.size();
 355:     return MCDisassembler::Fail;
 356:   }
 357: 
 358:   // Construct the instruction.
 359:   uint64_t Inst = 0;
 360:   for (uint64_t I = 0; I < Size; ++I)
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 361-364 / 第 361-364 行
```cpp
 361:     Inst = (Inst << 8) | Bytes[I];
 362: 
 363:   return decodeInstruction(Table, MI, Inst, Address, this, STI);
 364: }
```
- **EN**: This span continues the file's main responsibility: this file decodes target machine code into MC instructions for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

## Key Concepts / 关键概念
- **MC layer**: Uses LLVM MC structures for encoding, printing, parsing, or disassembly. / 使用 LLVM MC 结构进行编码、打印、解析或反汇编。
- **Disassembly**: Decodes binary encodings back into symbolic instructions. / 将二进制编码解码回符号化指令。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。

## Dependencies / 依赖关系
- `MCTargetDesc/SystemZMCTargetDesc.h`
- `TargetInfo/SystemZTargetInfo.h`
- `llvm/MC/MCDecoder.h`
- `llvm/MC/MCDecoderOps.h`
- `llvm/MC/MCDisassembler/MCDisassembler.h`
- `llvm/MC/MCInst.h`
- `llvm/MC/MCSubtargetInfo.h`
- `llvm/MC/TargetRegistry.h`
- `llvm/Support/Compiler.h`
- `llvm/Support/MathExtras.h`
- `cassert`
- `cstdint`
- `SystemZGenDisassemblerTables.inc`
