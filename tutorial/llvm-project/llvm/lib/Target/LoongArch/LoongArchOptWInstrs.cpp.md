# LoongArchOptWInstrs.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/LoongArch/LoongArchOptWInstrs.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Provides target-specific implementation details for the LoongArch backend.
- **用途 (CN)**: 提供 LoongArch 后端的目标相关实现细节。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28 / 第 1-28 行
```cpp
   1: //===- LoongArchOptWInstrs.cpp - MI W instruction optimizations ----------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===---------------------------------------------------------------------===//
   8: //
   9: // This pass does some optimizations for *W instructions at the MI level.
  10: //
  11: // First it removes unneeded sext(addi.w rd, rs, 0) instructions. Either
  12: // because the sign extended bits aren't consumed or because the input was
  13: // already sign extended by an earlier instruction.
  14: //
  15: // Then:
  16: // 1. Unless explicit disabled or the target prefers instructions with W suffix,
  17: //    it removes the -w suffix from opw instructions whenever all users are
  18: //    dependent only on the lower word of the result of the instruction.
  19: //    The cases handled are:
  20: //    * addi.w because it helps reduce test differences between LA32 and LA64
  21: //      w/o being a pessimization.
  22: //
  23: // 2. Or if explicit enabled or the target prefers instructions with W suffix,
  24: //    it adds the W suffix to the instruction whenever all users are dependent
  25: //    only on the lower word of the result of the instruction.
  26: //    The cases handled are:
  27: //    * add.d/addi.d/sub.d/mul.d.
  28: //    * slli.d with imm < 32.
```
- **EN**: This range is mainly descriptive commentary or banner text that frames the file before executable definitions begin.
- **CN**: 这一段主要是说明性注释或横幅文本，用于在可执行定义开始前说明文件背景。

### Lines 29-56 / 第 29-56 行
```cpp
  29: //    * ld.d/ld.wu.
  30: //===---------------------------------------------------------------------===//
  31: 
  32: #include "LoongArch.h"
  33: #include "LoongArchMachineFunctionInfo.h"
  34: #include "LoongArchSubtarget.h"
  35: #include "llvm/ADT/SmallSet.h"
  36: #include "llvm/ADT/Statistic.h"
  37: #include "llvm/CodeGen/MachineFunctionPass.h"
  38: #include "llvm/CodeGen/TargetInstrInfo.h"
  39: 
  40: using namespace llvm;
  41: 
  42: #define DEBUG_TYPE "loongarch-opt-w-instrs"
  43: #define LOONGARCH_OPT_W_INSTRS_NAME "LoongArch Optimize W Instructions"
  44: 
  45: STATISTIC(NumRemovedSExtW, "Number of removed sign-extensions");
  46: STATISTIC(NumTransformedToWInstrs,
  47:           "Number of instructions transformed to W-ops");
  48: 
  49: static cl::opt<bool>
  50:     DisableSExtWRemoval("loongarch-disable-sextw-removal",
  51:                         cl::desc("Disable removal of sign-extend insn"),
  52:                         cl::init(false), cl::Hidden);
  53: static cl::opt<bool>
  54:     DisableCvtToDSuffix("loongarch-disable-cvt-to-d-suffix",
  55:                         cl::desc("Disable convert to D suffix"),
  56:                         cl::init(false), cl::Hidden);
```
- **EN**: It imports dependencies such as `LoongArch.h`, `LoongArchMachineFunctionInfo.h`, `LoongArchSubtarget.h`, `SmallSet.h`, `Statistic.h`, `MachineFunctionPass.h` that expose the LLVM and target interfaces used in later logic. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 它引入了 `LoongArch.h`, `LoongArchMachineFunctionInfo.h`, `LoongArchSubtarget.h`, `SmallSet.h`, `Statistic.h`, `MachineFunctionPass.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 57-84 / 第 57-84 行
```cpp
  57: 
  58: namespace {
  59: 
  60: class LoongArchOptWInstrs : public MachineFunctionPass {
  61: public:
  62:   static char ID;
  63: 
  64:   LoongArchOptWInstrs() : MachineFunctionPass(ID) {}
  65: 
  66:   bool runOnMachineFunction(MachineFunction &MF) override;
  67:   bool removeSExtWInstrs(MachineFunction &MF, const LoongArchInstrInfo &TII,
  68:                          const LoongArchSubtarget &ST,
  69:                          MachineRegisterInfo &MRI);
  70:   bool convertToDSuffixes(MachineFunction &MF, const LoongArchInstrInfo &TII,
  71:                           const LoongArchSubtarget &ST,
  72:                           MachineRegisterInfo &MRI);
  73:   bool convertToWSuffixes(MachineFunction &MF, const LoongArchInstrInfo &TII,
  74:                           const LoongArchSubtarget &ST,
  75:                           MachineRegisterInfo &MRI);
  76: 
  77:   void getAnalysisUsage(AnalysisUsage &AU) const override {
  78:     AU.setPreservesCFG();
  79:     MachineFunctionPass::getAnalysisUsage(AU);
  80:   }
  81: 
  82:   StringRef getPassName() const override { return LOONGARCH_OPT_W_INSTRS_NAME; }
  83: };
  84: 
```
- **EN**: This block declares or refines TableGen records such as `LoongArchOptWInstrs`. The range implements or declares functions including `LoongArchOptWInstrs`.
- **CN**: 该代码块声明或细化了 `LoongArchOptWInstrs` 等 TableGen 记录。 这一段实现或声明了 `LoongArchOptWInstrs` 等函数。

### Lines 85-112 / 第 85-112 行
```cpp
  85: } // end anonymous namespace
  86: 
  87: char LoongArchOptWInstrs::ID = 0;
  88: INITIALIZE_PASS(LoongArchOptWInstrs, DEBUG_TYPE, LOONGARCH_OPT_W_INSTRS_NAME,
  89:                 false, false)
  90: 
  91: FunctionPass *llvm::createLoongArchOptWInstrsPass() {
  92:   return new LoongArchOptWInstrs();
  93: }
  94: 
  95: // Checks if all users only demand the lower \p OrigBits of the original
  96: // instruction's result.
  97: // TODO: handle multiple interdependent transformations
  98: static bool hasAllNBitUsers(const MachineInstr &OrigMI,
  99:                             const LoongArchSubtarget &ST,
 100:                             const MachineRegisterInfo &MRI, unsigned OrigBits) {
 101: 
 102:   SmallSet<std::pair<const MachineInstr *, unsigned>, 4> Visited;
 103:   SmallVector<std::pair<const MachineInstr *, unsigned>, 4> Worklist;
 104: 
 105:   Worklist.push_back(std::make_pair(&OrigMI, OrigBits));
 106: 
 107:   while (!Worklist.empty()) {
 108:     auto P = Worklist.pop_back_val();
 109:     const MachineInstr *MI = P.first;
 110:     unsigned Bits = P.second;
 111: 
 112:     if (!Visited.insert(P).second)
```
- **EN**: The range implements or declares functions including `hasAllNBitUsers`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `hasAllNBitUsers` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 113-140 / 第 113-140 行
```cpp
 113:       continue;
 114: 
 115:     // Only handle instructions with one def.
 116:     if (MI->getNumExplicitDefs() != 1)
 117:       return false;
 118: 
 119:     Register DestReg = MI->getOperand(0).getReg();
 120:     if (!DestReg.isVirtual())
 121:       return false;
 122: 
 123:     for (auto &UserOp : MRI.use_nodbg_operands(DestReg)) {
 124:       const MachineInstr *UserMI = UserOp.getParent();
 125:       unsigned OpIdx = UserOp.getOperandNo();
 126: 
 127:       switch (UserMI->getOpcode()) {
 128:       default:
 129:         return false;
 130: 
 131:       case LoongArch::ADD_W:
 132:       case LoongArch::ADDI_W:
 133:       case LoongArch::SUB_W:
 134:       case LoongArch::ALSL_W:
 135:       case LoongArch::ALSL_WU:
 136:       case LoongArch::MUL_W:
 137:       case LoongArch::MULH_W:
 138:       case LoongArch::MULH_WU:
 139:       case LoongArch::MULW_D_W:
 140:       case LoongArch::MULW_D_WU:
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 141-168 / 第 141-168 行
```cpp
 141:       case LoongArch::SLL_W:
 142:       case LoongArch::SLLI_W:
 143:       case LoongArch::SRL_W:
 144:       case LoongArch::SRLI_W:
 145:       case LoongArch::SRA_W:
 146:       case LoongArch::SRAI_W:
 147:       case LoongArch::ROTR_W:
 148:       case LoongArch::ROTRI_W:
 149:       case LoongArch::CLO_W:
 150:       case LoongArch::CLZ_W:
 151:       case LoongArch::CTO_W:
 152:       case LoongArch::CTZ_W:
 153:       case LoongArch::BYTEPICK_W:
 154:       case LoongArch::REVB_2H:
 155:       case LoongArch::BITREV_4B:
 156:       case LoongArch::BITREV_W:
 157:       case LoongArch::BSTRINS_W:
 158:       case LoongArch::BSTRPICK_W:
 159:       case LoongArch::CRC_W_W_W:
 160:       case LoongArch::CRCC_W_W_W:
 161:       case LoongArch::MOVGR2FCSR:
 162:       case LoongArch::MOVGR2FRH_W:
 163:       case LoongArch::MOVGR2FR_W_64:
 164:       case LoongArch::VINSGR2VR_W:
 165:       case LoongArch::XVINSGR2VR_W:
 166:       case LoongArch::VREPLGR2VR_W:
 167:       case LoongArch::XVREPLGR2VR_W:
 168:         if (Bits >= 32)
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 169-196 / 第 169-196 行
```cpp
 169:           break;
 170:         return false;
 171:       // {DIV,MOD}.W{U} consumes the upper 32 bits if the div32
 172:       // feature is not enabled.
 173:       case LoongArch::DIV_W:
 174:       case LoongArch::DIV_WU:
 175:       case LoongArch::MOD_W:
 176:       case LoongArch::MOD_WU:
 177:         if (Bits >= 32 && ST.hasDiv32())
 178:           break;
 179:         return false;
 180:       case LoongArch::MOVGR2CF:
 181:       case LoongArch::VREPLVE_D:
 182:       case LoongArch::XVREPLVE_D:
 183:         if (Bits >= 1)
 184:           break;
 185:         return false;
 186:       case LoongArch::VREPLVE_W:
 187:       case LoongArch::XVREPLVE_W:
 188:         if (Bits >= 2)
 189:           break;
 190:         return false;
 191:       case LoongArch::VREPLVE_H:
 192:       case LoongArch::XVREPLVE_H:
 193:         if (Bits >= 3)
 194:           break;
 195:         return false;
 196:       case LoongArch::VREPLVE_B:
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 197-224 / 第 197-224 行
```cpp
 197:       case LoongArch::XVREPLVE_B:
 198:         if (Bits >= 4)
 199:           break;
 200:         return false;
 201:       case LoongArch::EXT_W_B:
 202:       case LoongArch::VINSGR2VR_B:
 203:       case LoongArch::VREPLGR2VR_B:
 204:       case LoongArch::XVREPLGR2VR_B:
 205:         if (Bits >= 8)
 206:           break;
 207:         return false;
 208:       case LoongArch::EXT_W_H:
 209:       case LoongArch::VINSGR2VR_H:
 210:       case LoongArch::VREPLGR2VR_H:
 211:       case LoongArch::XVREPLGR2VR_H:
 212:         if (Bits >= 16)
 213:           break;
 214:         return false;
 215: 
 216:       case LoongArch::SRLI_D: {
 217:         // If we are shifting right by less than Bits, and users don't demand
 218:         // any bits that were shifted into [Bits-1:0], then we can consider this
 219:         // as an N-Bit user.
 220:         unsigned ShAmt = UserMI->getOperand(2).getImm();
 221:         if (Bits > ShAmt) {
 222:           Worklist.push_back(std::make_pair(UserMI, Bits - ShAmt));
 223:           break;
 224:         }
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 225-252 / 第 225-252 行
```cpp
 225:         return false;
 226:       }
 227: 
 228:       // these overwrite higher input bits, otherwise the lower word of output
 229:       // depends only on the lower word of input. So check their uses read W.
 230:       case LoongArch::SLLI_D:
 231:         if (Bits >= (ST.getGRLen() - UserMI->getOperand(2).getImm()))
 232:           break;
 233:         Worklist.push_back(std::make_pair(UserMI, Bits));
 234:         break;
 235:       case LoongArch::ANDI: {
 236:         uint64_t Imm = UserMI->getOperand(2).getImm();
 237:         if (Bits >= (unsigned)llvm::bit_width(Imm))
 238:           break;
 239:         Worklist.push_back(std::make_pair(UserMI, Bits));
 240:         break;
 241:       }
 242:       case LoongArch::ORI: {
 243:         uint64_t Imm = UserMI->getOperand(2).getImm();
 244:         if (Bits >= (unsigned)llvm::bit_width<uint64_t>(~Imm))
 245:           break;
 246:         Worklist.push_back(std::make_pair(UserMI, Bits));
 247:         break;
 248:       }
 249: 
 250:       case LoongArch::SLL_D:
 251:         // Operand 2 is the shift amount which uses log2(grlen) bits.
 252:         if (OpIdx == 2) {
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 253-280 / 第 253-280 行
```cpp
 253:           if (Bits >= Log2_32(ST.getGRLen()))
 254:             break;
 255:           return false;
 256:         }
 257:         Worklist.push_back(std::make_pair(UserMI, Bits));
 258:         break;
 259: 
 260:       case LoongArch::SRA_D:
 261:       case LoongArch::SRL_D:
 262:       case LoongArch::ROTR_D:
 263:         // Operand 2 is the shift amount which uses 6 bits.
 264:         if (OpIdx == 2 && Bits >= Log2_32(ST.getGRLen()))
 265:           break;
 266:         return false;
 267: 
 268:       case LoongArch::ST_B:
 269:       case LoongArch::STX_B:
 270:       case LoongArch::STGT_B:
 271:       case LoongArch::STLE_B:
 272:       case LoongArch::IOCSRWR_B:
 273:         // The first argument is the value to store.
 274:         if (OpIdx == 0 && Bits >= 8)
 275:           break;
 276:         return false;
 277:       case LoongArch::ST_H:
 278:       case LoongArch::STX_H:
 279:       case LoongArch::STGT_H:
 280:       case LoongArch::STLE_H:
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 281-308 / 第 281-308 行
```cpp
 281:       case LoongArch::IOCSRWR_H:
 282:         // The first argument is the value to store.
 283:         if (OpIdx == 0 && Bits >= 16)
 284:           break;
 285:         return false;
 286:       case LoongArch::ST_W:
 287:       case LoongArch::STX_W:
 288:       case LoongArch::SCREL_W:
 289:       case LoongArch::STPTR_W:
 290:       case LoongArch::STGT_W:
 291:       case LoongArch::STLE_W:
 292:       case LoongArch::IOCSRWR_W:
 293:         // The first argument is the value to store.
 294:         if (OpIdx == 0 && Bits >= 32)
 295:           break;
 296:         return false;
 297: 
 298:       case LoongArch::CRC_W_B_W:
 299:       case LoongArch::CRCC_W_B_W:
 300:         if ((OpIdx == 1 && Bits >= 8) || (OpIdx == 2 && Bits >= 32))
 301:           break;
 302:         return false;
 303:       case LoongArch::CRC_W_H_W:
 304:       case LoongArch::CRCC_W_H_W:
 305:         if ((OpIdx == 1 && Bits >= 16) || (OpIdx == 2 && Bits >= 32))
 306:           break;
 307:         return false;
 308:       case LoongArch::CRC_W_D_W:
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 309-336 / 第 309-336 行
```cpp
 309:       case LoongArch::CRCC_W_D_W:
 310:         if (OpIdx == 2 && Bits >= 32)
 311:           break;
 312:         return false;
 313: 
 314:       // For these, lower word of output in these operations, depends only on
 315:       // the lower word of input. So, we check all uses only read lower word.
 316:       case LoongArch::COPY:
 317:       case LoongArch::PHI:
 318:       case LoongArch::ADD_D:
 319:       case LoongArch::ADDI_D:
 320:       case LoongArch::SUB_D:
 321:       case LoongArch::MUL_D:
 322:       case LoongArch::AND:
 323:       case LoongArch::OR:
 324:       case LoongArch::NOR:
 325:       case LoongArch::XOR:
 326:       case LoongArch::XORI:
 327:       case LoongArch::ANDN:
 328:       case LoongArch::ORN:
 329:         Worklist.push_back(std::make_pair(UserMI, Bits));
 330:         break;
 331: 
 332:       case LoongArch::MASKNEZ:
 333:       case LoongArch::MASKEQZ:
 334:         if (OpIdx != 1)
 335:           return false;
 336:         Worklist.push_back(std::make_pair(UserMI, Bits));
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 337-364 / 第 337-364 行
```cpp
 337:         break;
 338:       }
 339:     }
 340:   }
 341: 
 342:   return true;
 343: }
 344: 
 345: static bool hasAllWUsers(const MachineInstr &OrigMI,
 346:                          const LoongArchSubtarget &ST,
 347:                          const MachineRegisterInfo &MRI) {
 348:   return hasAllNBitUsers(OrigMI, ST, MRI, 32);
 349: }
 350: 
 351: // This function returns true if the machine instruction always outputs a value
 352: // where bits 63:32 match bit 31.
 353: static bool isSignExtendingOpW(const MachineInstr &MI,
 354:                                const MachineRegisterInfo &MRI, unsigned OpNo) {
 355:   switch (MI.getOpcode()) {
 356:   // Normal cases
 357:   case LoongArch::ADD_W:
 358:   case LoongArch::SUB_W:
 359:   case LoongArch::ADDI_W:
 360:   case LoongArch::ALSL_W:
 361:   case LoongArch::LU12I_W:
 362:   case LoongArch::SLT:
 363:   case LoongArch::SLTU:
 364:   case LoongArch::SLTI:
```
- **EN**: The range implements or declares functions including `hasAllWUsers`, `isSignExtendingOpW`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `hasAllWUsers`, `isSignExtendingOpW` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 365-392 / 第 365-392 行
```cpp
 365:   case LoongArch::SLTUI:
 366:   case LoongArch::ANDI:
 367:   case LoongArch::MUL_W:
 368:   case LoongArch::MULH_W:
 369:   case LoongArch::MULH_WU:
 370:   case LoongArch::DIV_W:
 371:   case LoongArch::MOD_W:
 372:   case LoongArch::DIV_WU:
 373:   case LoongArch::MOD_WU:
 374:   case LoongArch::SLL_W:
 375:   case LoongArch::SRL_W:
 376:   case LoongArch::SRA_W:
 377:   case LoongArch::ROTR_W:
 378:   case LoongArch::SLLI_W:
 379:   case LoongArch::SRLI_W:
 380:   case LoongArch::SRAI_W:
 381:   case LoongArch::ROTRI_W:
 382:   case LoongArch::EXT_W_B:
 383:   case LoongArch::EXT_W_H:
 384:   case LoongArch::CLO_W:
 385:   case LoongArch::CLZ_W:
 386:   case LoongArch::CTO_W:
 387:   case LoongArch::CTZ_W:
 388:   case LoongArch::BYTEPICK_W:
 389:   case LoongArch::REVB_2H:
 390:   case LoongArch::BITREV_4B:
 391:   case LoongArch::BITREV_W:
 392:   case LoongArch::BSTRINS_W:
```
- **EN**: This span continues the file's main responsibility: provides target-specific implementation details for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 393-420 / 第 393-420 行
```cpp
 393:   case LoongArch::BSTRPICK_W:
 394:   case LoongArch::LD_B:
 395:   case LoongArch::LD_H:
 396:   case LoongArch::LD_W:
 397:   case LoongArch::LD_BU:
 398:   case LoongArch::LD_HU:
 399:   case LoongArch::LL_W:
 400:   case LoongArch::LLACQ_W:
 401:   case LoongArch::RDTIMEL_W:
 402:   case LoongArch::RDTIMEH_W:
 403:   case LoongArch::CPUCFG:
 404:   case LoongArch::LDX_B:
 405:   case LoongArch::LDX_H:
 406:   case LoongArch::LDX_W:
 407:   case LoongArch::LDX_BU:
 408:   case LoongArch::LDX_HU:
 409:   case LoongArch::LDPTR_W:
 410:   case LoongArch::LDGT_B:
 411:   case LoongArch::LDGT_H:
 412:   case LoongArch::LDGT_W:
 413:   case LoongArch::LDLE_B:
 414:   case LoongArch::LDLE_H:
 415:   case LoongArch::LDLE_W:
 416:   case LoongArch::AMSWAP_B:
 417:   case LoongArch::AMSWAP_H:
 418:   case LoongArch::AMSWAP_W:
 419:   case LoongArch::AMADD_B:
 420:   case LoongArch::AMADD_H:
```
- **EN**: This span continues the file's main responsibility: provides target-specific implementation details for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 421-448 / 第 421-448 行
```cpp
 421:   case LoongArch::AMADD_W:
 422:   case LoongArch::AMAND_W:
 423:   case LoongArch::AMOR_W:
 424:   case LoongArch::AMXOR_W:
 425:   case LoongArch::AMMAX_W:
 426:   case LoongArch::AMMIN_W:
 427:   case LoongArch::AMMAX_WU:
 428:   case LoongArch::AMMIN_WU:
 429:   case LoongArch::AMSWAP__DB_B:
 430:   case LoongArch::AMSWAP__DB_H:
 431:   case LoongArch::AMSWAP__DB_W:
 432:   case LoongArch::AMADD__DB_B:
 433:   case LoongArch::AMADD__DB_H:
 434:   case LoongArch::AMADD__DB_W:
 435:   case LoongArch::AMAND__DB_W:
 436:   case LoongArch::AMOR__DB_W:
 437:   case LoongArch::AMXOR__DB_W:
 438:   case LoongArch::AMMAX__DB_W:
 439:   case LoongArch::AMMIN__DB_W:
 440:   case LoongArch::AMMAX__DB_WU:
 441:   case LoongArch::AMMIN__DB_WU:
 442:   case LoongArch::AMCAS_B:
 443:   case LoongArch::AMCAS_H:
 444:   case LoongArch::AMCAS_W:
 445:   case LoongArch::AMCAS__DB_B:
 446:   case LoongArch::AMCAS__DB_H:
 447:   case LoongArch::AMCAS__DB_W:
 448:   case LoongArch::CRC_W_B_W:
```
- **EN**: This span continues the file's main responsibility: provides target-specific implementation details for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 449-476 / 第 449-476 行
```cpp
 449:   case LoongArch::CRC_W_H_W:
 450:   case LoongArch::CRC_W_W_W:
 451:   case LoongArch::CRC_W_D_W:
 452:   case LoongArch::CRCC_W_B_W:
 453:   case LoongArch::CRCC_W_H_W:
 454:   case LoongArch::CRCC_W_W_W:
 455:   case LoongArch::CRCC_W_D_W:
 456:   case LoongArch::IOCSRRD_B:
 457:   case LoongArch::IOCSRRD_H:
 458:   case LoongArch::IOCSRRD_W:
 459:   case LoongArch::MOVFR2GR_S:
 460:   case LoongArch::MOVFCSR2GR:
 461:   case LoongArch::MOVCF2GR:
 462:   case LoongArch::MOVFRH2GR_S:
 463:   case LoongArch::MOVFR2GR_S_64:
 464:   case LoongArch::VPICKVE2GR_W:
 465:   case LoongArch::XVPICKVE2GR_W:
 466:     return true;
 467:   // Special cases that require checking operands.
 468:   // shifting right sufficiently makes the value 32-bit sign-extended
 469:   case LoongArch::SRAI_D:
 470:     return MI.getOperand(2).getImm() >= 32;
 471:   case LoongArch::SRLI_D:
 472:     return MI.getOperand(2).getImm() > 32;
 473:   // The LI pattern ADDI rd, R0, imm and ORI rd, R0, imm are sign extended.
 474:   case LoongArch::ADDI_D:
 475:   case LoongArch::ORI:
 476:     return MI.getOperand(1).isReg() &&
```
- **EN**: This span continues the file's main responsibility: provides target-specific implementation details for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 477-504 / 第 477-504 行
```cpp
 477:            MI.getOperand(1).getReg() == LoongArch::R0;
 478:   // A bits extract is sign extended if the msb is less than 31.
 479:   case LoongArch::BSTRPICK_D:
 480:     return MI.getOperand(2).getImm() < 31;
 481:   // Copying from R0 produces zero.
 482:   case LoongArch::COPY:
 483:     return MI.getOperand(1).getReg() == LoongArch::R0;
 484:   // Ignore the scratch register destination.
 485:   case LoongArch::PseudoMaskedAtomicSwap32:
 486:   case LoongArch::PseudoAtomicSwap32:
 487:   case LoongArch::PseudoMaskedAtomicLoadAdd32:
 488:   case LoongArch::PseudoMaskedAtomicLoadSub32:
 489:   case LoongArch::PseudoAtomicLoadNand32:
 490:   case LoongArch::PseudoMaskedAtomicLoadNand32:
 491:   case LoongArch::PseudoAtomicLoadAdd32:
 492:   case LoongArch::PseudoAtomicLoadSub32:
 493:   case LoongArch::PseudoAtomicLoadAnd32:
 494:   case LoongArch::PseudoAtomicLoadOr32:
 495:   case LoongArch::PseudoAtomicLoadXor32:
 496:   case LoongArch::PseudoMaskedAtomicLoadUMax32:
 497:   case LoongArch::PseudoMaskedAtomicLoadUMin32:
 498:   case LoongArch::PseudoCmpXchg32:
 499:   case LoongArch::PseudoMaskedCmpXchg32:
 500:   case LoongArch::PseudoMaskedAtomicLoadMax32:
 501:   case LoongArch::PseudoMaskedAtomicLoadMin32:
 502:     return OpNo == 0;
 503:   }
 504: 
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 505-532 / 第 505-532 行
```cpp
 505:   return false;
 506: }
 507: 
 508: static bool isSignExtendedW(Register SrcReg, const LoongArchSubtarget &ST,
 509:                             const MachineRegisterInfo &MRI,
 510:                             SmallPtrSetImpl<MachineInstr *> &FixableDef) {
 511:   SmallSet<Register, 4> Visited;
 512:   SmallVector<Register, 4> Worklist;
 513: 
 514:   auto AddRegToWorkList = [&](Register SrcReg) {
 515:     if (!SrcReg.isVirtual())
 516:       return false;
 517:     Worklist.push_back(SrcReg);
 518:     return true;
 519:   };
 520: 
 521:   if (!AddRegToWorkList(SrcReg))
 522:     return false;
 523: 
 524:   while (!Worklist.empty()) {
 525:     Register Reg = Worklist.pop_back_val();
 526: 
 527:     // If we already visited this register, we don't need to check it again.
 528:     if (!Visited.insert(Reg).second)
 529:       continue;
 530: 
 531:     MachineInstr *MI = MRI.getVRegDef(Reg);
 532:     if (!MI)
```
- **EN**: The range implements or declares functions including `isSignExtendedW`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `isSignExtendedW` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 533-560 / 第 533-560 行
```cpp
 533:       continue;
 534: 
 535:     int OpNo = MI->findRegisterDefOperandIdx(Reg, /*TRI=*/nullptr);
 536:     assert(OpNo != -1 && "Couldn't find register");
 537: 
 538:     // If this is a sign extending operation we don't need to look any further.
 539:     if (isSignExtendingOpW(*MI, MRI, OpNo))
 540:       continue;
 541: 
 542:     // Is this an instruction that propagates sign extend?
 543:     switch (MI->getOpcode()) {
 544:     default:
 545:       // Unknown opcode, give up.
 546:       return false;
 547:     case LoongArch::COPY: {
 548:       const MachineFunction *MF = MI->getMF();
 549:       const LoongArchMachineFunctionInfo *LAFI =
 550:           MF->getInfo<LoongArchMachineFunctionInfo>();
 551: 
 552:       // If this is the entry block and the register is livein, see if we know
 553:       // it is sign extended.
 554:       if (MI->getParent() == &MF->front()) {
 555:         Register VReg = MI->getOperand(0).getReg();
 556:         if (MF->getRegInfo().isLiveIn(VReg) && LAFI->isSExt32Register(VReg))
 557:           continue;
 558:       }
 559: 
 560:       Register CopySrcReg = MI->getOperand(1).getReg();
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 561-588 / 第 561-588 行
```cpp
 561:       if (CopySrcReg == LoongArch::R4) {
 562:         // For a method return value, we check the ZExt/SExt flags in attribute.
 563:         // We assume the following code sequence for method call.
 564:         // PseudoCALL_SMALL @bar, ...
 565:         // ADJCALLSTACKUP 0, 0, implicit-def dead $r3, implicit $r3
 566:         // %0:gpr = COPY $r4
 567:         //
 568:         // We use the PseudoCall to look up the IR function being called to find
 569:         // its return attributes.
 570:         const MachineBasicBlock *MBB = MI->getParent();
 571:         auto II = MI->getIterator();
 572:         if (II == MBB->instr_begin() ||
 573:             (--II)->getOpcode() != LoongArch::ADJCALLSTACKUP)
 574:           return false;
 575: 
 576:         const MachineInstr &CallMI = *(--II);
 577:         if (!CallMI.isCall() || !CallMI.getOperand(0).isGlobal())
 578:           return false;
 579: 
 580:         auto *CalleeFn =
 581:             dyn_cast_if_present<Function>(CallMI.getOperand(0).getGlobal());
 582:         if (!CalleeFn)
 583:           return false;
 584: 
 585:         auto *IntTy = dyn_cast<IntegerType>(CalleeFn->getReturnType());
 586:         if (!IntTy)
 587:           return false;
 588: 
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 589-616 / 第 589-616 行
```cpp
 589:         const AttributeSet &Attrs = CalleeFn->getAttributes().getRetAttrs();
 590:         unsigned BitWidth = IntTy->getBitWidth();
 591:         if ((BitWidth <= 32 && Attrs.hasAttribute(Attribute::SExt)) ||
 592:             (BitWidth < 32 && Attrs.hasAttribute(Attribute::ZExt)))
 593:           continue;
 594:       }
 595: 
 596:       if (!AddRegToWorkList(CopySrcReg))
 597:         return false;
 598: 
 599:       break;
 600:     }
 601: 
 602:     // For these, we just need to check if the 1st operand is sign extended.
 603:     case LoongArch::MOD_D:
 604:     case LoongArch::ANDI:
 605:     case LoongArch::ORI:
 606:     case LoongArch::XORI:
 607:       // |Remainder| is always <= |Dividend|. If D is 32-bit, then so is R.
 608:       // DIV doesn't work because of the edge case 0xf..f 8000 0000 / (long)-1
 609:       // Logical operations use a sign extended 12-bit immediate.
 610:       if (!AddRegToWorkList(MI->getOperand(1).getReg()))
 611:         return false;
 612: 
 613:       break;
 614:     case LoongArch::MOD_DU:
 615:     case LoongArch::AND:
 616:     case LoongArch::OR:
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 617-644 / 第 617-644 行
```cpp
 617:     case LoongArch::XOR:
 618:     case LoongArch::ANDN:
 619:     case LoongArch::ORN:
 620:     case LoongArch::PHI: {
 621:       // If all incoming values are sign-extended, the output of AND, OR, XOR,
 622:       // or PHI is also sign-extended.
 623: 
 624:       // The input registers for PHI are operand 1, 3, ...
 625:       // The input registers for others are operand 1 and 2.
 626:       unsigned B = 1, E = 3, D = 1;
 627:       switch (MI->getOpcode()) {
 628:       case LoongArch::PHI:
 629:         E = MI->getNumOperands();
 630:         D = 2;
 631:         break;
 632:       }
 633: 
 634:       for (unsigned I = B; I != E; I += D) {
 635:         if (!MI->getOperand(I).isReg())
 636:           return false;
 637: 
 638:         if (!AddRegToWorkList(MI->getOperand(I).getReg()))
 639:           return false;
 640:       }
 641: 
 642:       break;
 643:     }
 644: 
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 645-672 / 第 645-672 行
```cpp
 645:     case LoongArch::MASKEQZ:
 646:     case LoongArch::MASKNEZ:
 647:       // Instructions return zero or operand 1. Result is sign extended if
 648:       // operand 1 is sign extended.
 649:       if (!AddRegToWorkList(MI->getOperand(1).getReg()))
 650:         return false;
 651:       break;
 652: 
 653:     // With these opcode, we can "fix" them with the W-version
 654:     // if we know all users of the result only rely on bits 31:0
 655:     case LoongArch::SLLI_D:
 656:       // SLLI_W reads the lowest 5 bits, while SLLI_D reads lowest 6 bits
 657:       if (MI->getOperand(2).getImm() >= 32)
 658:         return false;
 659:       [[fallthrough]];
 660:     case LoongArch::ADDI_D:
 661:     case LoongArch::ADD_D:
 662:     case LoongArch::LD_D:
 663:     case LoongArch::LD_WU:
 664:     case LoongArch::MUL_D:
 665:     case LoongArch::SUB_D:
 666:       if (hasAllWUsers(*MI, ST, MRI)) {
 667:         FixableDef.insert(MI);
 668:         break;
 669:       }
 670:       return false;
 671:     // If all incoming values are sign-extended and all users only use
 672:     // the lower 32 bits, then convert them to W versions.
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 673-700 / 第 673-700 行
```cpp
 673:     case LoongArch::DIV_D: {
 674:       if (!AddRegToWorkList(MI->getOperand(1).getReg()))
 675:         return false;
 676:       if (!AddRegToWorkList(MI->getOperand(2).getReg()))
 677:         return false;
 678:       if (hasAllWUsers(*MI, ST, MRI)) {
 679:         FixableDef.insert(MI);
 680:         break;
 681:       }
 682:       return false;
 683:     }
 684:     }
 685:   }
 686: 
 687:   // If we get here, then every node we visited produces a sign extended value
 688:   // or propagated sign extended values. So the result must be sign extended.
 689:   return true;
 690: }
 691: 
 692: static unsigned getWOp(unsigned Opcode) {
 693:   switch (Opcode) {
 694:   case LoongArch::ADDI_D:
 695:     return LoongArch::ADDI_W;
 696:   case LoongArch::ADD_D:
 697:     return LoongArch::ADD_W;
 698:   case LoongArch::DIV_D:
 699:     return LoongArch::DIV_W;
 700:   case LoongArch::LD_D:
```
- **EN**: The range implements or declares functions including `getWOp`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `getWOp` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 701-728 / 第 701-728 行
```cpp
 701:   case LoongArch::LD_WU:
 702:     return LoongArch::LD_W;
 703:   case LoongArch::MUL_D:
 704:     return LoongArch::MUL_W;
 705:   case LoongArch::SLLI_D:
 706:     return LoongArch::SLLI_W;
 707:   case LoongArch::SUB_D:
 708:     return LoongArch::SUB_W;
 709:   default:
 710:     llvm_unreachable("Unexpected opcode for replacement with W variant");
 711:   }
 712: }
 713: 
 714: bool LoongArchOptWInstrs::removeSExtWInstrs(MachineFunction &MF,
 715:                                             const LoongArchInstrInfo &TII,
 716:                                             const LoongArchSubtarget &ST,
 717:                                             MachineRegisterInfo &MRI) {
 718:   if (DisableSExtWRemoval)
 719:     return false;
 720: 
 721:   bool MadeChange = false;
 722:   for (MachineBasicBlock &MBB : MF) {
 723:     for (MachineInstr &MI : llvm::make_early_inc_range(MBB)) {
 724:       // We're looking for the sext.w pattern ADDI.W rd, rs, 0.
 725:       if (!LoongArch::isSEXT_W(MI))
 726:         continue;
 727: 
 728:       Register SrcReg = MI.getOperand(1).getReg();
```
- **EN**: The range implements or declares functions including `LoongArchOptWInstrs::removeSExtWInstrs`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `LoongArchOptWInstrs::removeSExtWInstrs` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 729-756 / 第 729-756 行
```cpp
 729: 
 730:       SmallPtrSet<MachineInstr *, 4> FixableDefs;
 731: 
 732:       // If all users only use the lower bits, this sext.w is redundant.
 733:       // Or if all definitions reaching MI sign-extend their output,
 734:       // then sext.w is redundant.
 735:       if (!hasAllWUsers(MI, ST, MRI) &&
 736:           !isSignExtendedW(SrcReg, ST, MRI, FixableDefs))
 737:         continue;
 738: 
 739:       Register DstReg = MI.getOperand(0).getReg();
 740:       if (!SrcReg.isVirtual() ||
 741:           !MRI.constrainRegClass(SrcReg, MRI.getRegClass(DstReg)))
 742:         continue;
 743: 
 744:       // Convert Fixable instructions to their W versions.
 745:       for (MachineInstr *Fixable : FixableDefs) {
 746:         LLVM_DEBUG(dbgs() << "Replacing " << *Fixable);
 747:         Fixable->setDesc(TII.get(getWOp(Fixable->getOpcode())));
 748:         Fixable->clearFlag(MachineInstr::MIFlag::NoSWrap);
 749:         Fixable->clearFlag(MachineInstr::MIFlag::NoUWrap);
 750:         Fixable->clearFlag(MachineInstr::MIFlag::IsExact);
 751:         LLVM_DEBUG(dbgs() << "     with " << *Fixable);
 752:         ++NumTransformedToWInstrs;
 753:       }
 754: 
 755:       LLVM_DEBUG(dbgs() << "Removing redundant sign-extension\n");
 756:       MRI.replaceRegWith(DstReg, SrcReg);
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 757-784 / 第 757-784 行
```cpp
 757:       MRI.clearKillFlags(SrcReg);
 758:       MI.eraseFromParent();
 759:       ++NumRemovedSExtW;
 760:       MadeChange = true;
 761:     }
 762:   }
 763: 
 764:   return MadeChange;
 765: }
 766: 
 767: bool LoongArchOptWInstrs::convertToDSuffixes(MachineFunction &MF,
 768:                                              const LoongArchInstrInfo &TII,
 769:                                              const LoongArchSubtarget &ST,
 770:                                              MachineRegisterInfo &MRI) {
 771:   bool MadeChange = false;
 772:   for (MachineBasicBlock &MBB : MF) {
 773:     for (MachineInstr &MI : MBB) {
 774:       unsigned Opc;
 775:       switch (MI.getOpcode()) {
 776:       default:
 777:         continue;
 778:       case LoongArch::ADDI_W:
 779:         Opc = LoongArch::ADDI_D;
 780:         break;
 781:       }
 782: 
 783:       if (hasAllWUsers(MI, ST, MRI)) {
 784:         MI.setDesc(TII.get(Opc));
```
- **EN**: The range implements or declares functions including `LoongArchOptWInstrs::convertToDSuffixes`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchOptWInstrs::convertToDSuffixes` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 785-812 / 第 785-812 行
```cpp
 785:         MadeChange = true;
 786:       }
 787:     }
 788:   }
 789: 
 790:   return MadeChange;
 791: }
 792: 
 793: bool LoongArchOptWInstrs::convertToWSuffixes(MachineFunction &MF,
 794:                                              const LoongArchInstrInfo &TII,
 795:                                              const LoongArchSubtarget &ST,
 796:                                              MachineRegisterInfo &MRI) {
 797:   bool MadeChange = false;
 798:   for (MachineBasicBlock &MBB : MF) {
 799:     for (MachineInstr &MI : MBB) {
 800:       unsigned WOpc;
 801:       // TODO: Add more?
 802:       switch (MI.getOpcode()) {
 803:       default:
 804:         continue;
 805:       case LoongArch::ADD_D:
 806:         WOpc = LoongArch::ADD_W;
 807:         break;
 808:       case LoongArch::ADDI_D:
 809:         WOpc = LoongArch::ADDI_W;
 810:         break;
 811:       case LoongArch::SUB_D:
 812:         WOpc = LoongArch::SUB_W;
```
- **EN**: The range implements or declares functions including `LoongArchOptWInstrs::convertToWSuffixes`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `LoongArchOptWInstrs::convertToWSuffixes` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 813-840 / 第 813-840 行
```cpp
 813:         break;
 814:       case LoongArch::MUL_D:
 815:         WOpc = LoongArch::MUL_W;
 816:         break;
 817:       case LoongArch::SLLI_D:
 818:         // SLLI.W reads the lowest 5 bits, while SLLI.D reads lowest 6 bits
 819:         if (MI.getOperand(2).getImm() >= 32)
 820:           continue;
 821:         WOpc = LoongArch::SLLI_W;
 822:         break;
 823:       case LoongArch::LD_D:
 824:       case LoongArch::LD_WU:
 825:         WOpc = LoongArch::LD_W;
 826:         break;
 827:       }
 828: 
 829:       if (hasAllWUsers(MI, ST, MRI)) {
 830:         LLVM_DEBUG(dbgs() << "Replacing " << MI);
 831:         MI.setDesc(TII.get(WOpc));
 832:         MI.clearFlag(MachineInstr::MIFlag::NoSWrap);
 833:         MI.clearFlag(MachineInstr::MIFlag::NoUWrap);
 834:         MI.clearFlag(MachineInstr::MIFlag::IsExact);
 835:         LLVM_DEBUG(dbgs() << "     with " << MI);
 836:         ++NumTransformedToWInstrs;
 837:         MadeChange = true;
 838:       }
 839:     }
 840:   }
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 841-866 / 第 841-866 行
```cpp
 841: 
 842:   return MadeChange;
 843: }
 844: 
 845: bool LoongArchOptWInstrs::runOnMachineFunction(MachineFunction &MF) {
 846:   if (skipFunction(MF.getFunction()))
 847:     return false;
 848: 
 849:   MachineRegisterInfo &MRI = MF.getRegInfo();
 850:   const LoongArchSubtarget &ST = MF.getSubtarget<LoongArchSubtarget>();
 851:   const LoongArchInstrInfo &TII = *ST.getInstrInfo();
 852: 
 853:   if (!ST.is64Bit())
 854:     return false;
 855: 
 856:   bool MadeChange = false;
 857:   MadeChange |= removeSExtWInstrs(MF, TII, ST, MRI);
 858: 
 859:   if (!(DisableCvtToDSuffix || ST.preferWInst()))
 860:     MadeChange |= convertToDSuffixes(MF, TII, ST, MRI);
 861: 
 862:   if (ST.preferWInst())
 863:     MadeChange |= convertToWSuffixes(MF, TII, ST, MRI);
 864: 
 865:   return MadeChange;
 866: }
```
- **EN**: The range implements or declares functions including `LoongArchOptWInstrs::runOnMachineFunction`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchOptWInstrs::runOnMachineFunction` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

## Key Concepts / 关键概念
- **MachineInstr**: Represents target-aware machine instructions during late code generation. / 表示代码生成后期的目标相关机器指令。
- **MachineFunction**: Carries per-function machine-level state and basic blocks. / 保存每个函数的机器级状态和基本块。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Register classes**: Groups registers by capability so instruction operands and allocators can constrain them. / 按能力对寄存器分组，以便指令操作数和分配器施加约束。
- **Instruction metadata**: Captures opcodes, operands, patterns, and helper routines. / 描述操作码、操作数、匹配模式和辅助例程。
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。

## Dependencies / 依赖关系
- `LoongArch.h`
- `LoongArchMachineFunctionInfo.h`
- `LoongArchSubtarget.h`
- `llvm/ADT/SmallSet.h`
- `llvm/ADT/Statistic.h`
- `llvm/CodeGen/MachineFunctionPass.h`
- `llvm/CodeGen/TargetInstrInfo.h`
