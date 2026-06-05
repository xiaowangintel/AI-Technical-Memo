# M68kISelDAGToDAG.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/M68k/M68kISelDAGToDAG.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file selects target instructions from SelectionDAG nodes for the M68k backend.
- **用途 (CN)**: 该文件用于 M68k 后端，负责从 SelectionDAG 节点选择目标指令。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28 / 第 1-28 行
```cpp
   1: //===-- M68kISelDAGToDAG.cpp - M68k Dag to Dag Inst Selector ----*- C++ -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: ///
   9: /// \file
  10: /// This file defines an instruction selector for the M68K target.
  11: ///
  12: //===----------------------------------------------------------------------===//
  13: 
  14: #include "M68k.h"
  15: 
  16: #include "M68kMachineFunction.h"
  17: #include "M68kRegisterInfo.h"
  18: #include "M68kSelectionDAGInfo.h"
  19: #include "M68kTargetMachine.h"
  20: #include "llvm/CodeGen/MachineConstantPool.h"
  21: #include "llvm/CodeGen/MachineFrameInfo.h"
  22: #include "llvm/CodeGen/MachineFunction.h"
  23: #include "llvm/CodeGen/MachineInstrBuilder.h"
  24: #include "llvm/CodeGen/MachineRegisterInfo.h"
  25: #include "llvm/CodeGen/SelectionDAGISel.h"
  26: #include "llvm/CodeGen/SelectionDAGNodes.h"
  27: #include "llvm/IR/CFG.h"
  28: #include "llvm/IR/GlobalValue.h"
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `M68k.h`, `M68kMachineFunction.h`, `M68kRegisterInfo.h`, `M68kSelectionDAGInfo.h`, `M68kTargetMachine.h`, `MachineConstantPool.h` that expose the LLVM and target interfaces used in later logic.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `M68k.h`, `M68kMachineFunction.h`, `M68kRegisterInfo.h`, `M68kSelectionDAGInfo.h`, `M68kTargetMachine.h`, `MachineConstantPool.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。

### Lines 29-56 / 第 29-56 行
```cpp
  29: #include "llvm/IR/Instructions.h"
  30: #include "llvm/IR/Intrinsics.h"
  31: #include "llvm/IR/Type.h"
  32: #include "llvm/Support/Alignment.h"
  33: #include "llvm/Support/Debug.h"
  34: #include "llvm/Support/ErrorHandling.h"
  35: #include "llvm/Support/MathExtras.h"
  36: #include "llvm/Support/raw_ostream.h"
  37: #include "llvm/Target/TargetMachine.h"
  38: 
  39: using namespace llvm;
  40: 
  41: #define DEBUG_TYPE "m68k-isel"
  42: #define PASS_NAME "M68k DAG->DAG Pattern Instruction Selection"
  43: 
  44: namespace {
  45: 
  46: // For reference, the full order of operands for memory references is:
  47: // (Operand), Displacement, Base, Index, Scale
  48: struct M68kISelAddressMode {
  49:   enum class AddrType {
  50:     ARI,   // Address Register Indirect
  51:     ARIPI, // Address Register Indirect with Postincrement
  52:     ARIPD, // Address Register Indirect with Postdecrement
  53:     ARID,  // Address Register Indirect with Displacement
  54:     ARII,  // Address Register Indirect with Index
  55:     PCD,   // Program Counter Indirect with Displacement
  56:     PCI,   // Program Counter Indirect with Index
```
- **EN**: It imports dependencies such as `Instructions.h`, `Intrinsics.h`, `Type.h`, `Alignment.h`, `Debug.h`, `ErrorHandling.h` that expose the LLVM and target interfaces used in later logic. It introduces interface types such as `M68kISelAddressMode`, shaping how other backend components interact with this file. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 它引入了 `Instructions.h`, `Intrinsics.h`, `Type.h`, `Alignment.h`, `Debug.h`, `ErrorHandling.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 它引入了 `M68kISelAddressMode` 等接口类型，定义了其他后端组件与本文件交互的方式。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 57-84 / 第 57-84 行
```cpp
  57:     AL,    // Absolute
  58:   };
  59:   AddrType AM;
  60: 
  61:   enum class Base { RegBase, FrameIndexBase };
  62:   Base BaseType;
  63: 
  64:   int64_t Disp;
  65: 
  66:   // This is really a union, discriminated by BaseType!
  67:   SDValue BaseReg;
  68:   int BaseFrameIndex;
  69: 
  70:   SDValue IndexReg;
  71:   unsigned Scale;
  72: 
  73:   const GlobalValue *GV;
  74:   const Constant *CP;
  75:   const BlockAddress *BlockAddr;
  76:   const char *ES;
  77:   MCSymbol *MCSym;
  78:   int JT;
  79:   Align Alignment; // CP alignment.
  80: 
  81:   unsigned char SymbolFlags; // M68kII::MO_*
  82: 
  83:   M68kISelAddressMode(AddrType AT)
  84:       : AM(AT), BaseType(Base::RegBase), Disp(0), BaseFrameIndex(0), IndexReg(),
```
- **EN**: The range implements or declares functions including `M68kISelAddressMode`.
- **CN**: 这一段实现或声明了 `M68kISelAddressMode` 等函数。

### Lines 85-112 / 第 85-112 行
```cpp
  85:         Scale(1), GV(nullptr), CP(nullptr), BlockAddr(nullptr), ES(nullptr),
  86:         MCSym(nullptr), JT(-1), Alignment(), SymbolFlags(M68kII::MO_NO_FLAG) {}
  87: 
  88:   bool hasSymbolicDisplacement() const {
  89:     return GV != nullptr || CP != nullptr || ES != nullptr ||
  90:            MCSym != nullptr || JT != -1 || BlockAddr != nullptr;
  91:   }
  92: 
  93:   bool hasBase() const {
  94:     return BaseType == Base::FrameIndexBase || BaseReg.getNode() != nullptr;
  95:   }
  96: 
  97:   bool hasFrameIndex() const { return BaseType == Base::FrameIndexBase; }
  98: 
  99:   bool hasBaseReg() const {
 100:     return BaseType == Base::RegBase && BaseReg.getNode() != nullptr;
 101:   }
 102: 
 103:   bool hasIndexReg() const {
 104:     return BaseType == Base::RegBase && IndexReg.getNode() != nullptr;
 105:   }
 106: 
 107:   /// True if address mode type supports displacement
 108:   bool isDispAddrType() const {
 109:     return AM == AddrType::ARII || AM == AddrType::PCI ||
 110:            AM == AddrType::ARID || AM == AddrType::PCD || AM == AddrType::AL;
 111:   }
 112: 
```
- **EN**: The range implements or declares functions including `Scale`, `hasBase`, `hasFrameIndex`, `hasBaseReg`, `hasIndexReg`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `Scale`, `hasBase`, `hasFrameIndex`, `hasBaseReg`, `hasIndexReg` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 113-140 / 第 113-140 行
```cpp
 113:   unsigned getDispSize() const {
 114:     switch (AM) {
 115:     default:
 116:       return 0;
 117:     case AddrType::ARII:
 118:     case AddrType::PCI:
 119:       return 8;
 120:     // These two in the next chip generations can hold upto 32 bit
 121:     case AddrType::ARID:
 122:     case AddrType::PCD:
 123:       return 16;
 124:     case AddrType::AL:
 125:       return 32;
 126:     }
 127:   }
 128: 
 129:   bool hasDisp() const { return getDispSize() != 0; }
 130:   bool isDisp8() const { return getDispSize() == 8; }
 131:   bool isDisp16() const { return getDispSize() == 16; }
 132:   bool isDisp32() const { return getDispSize() == 32; }
 133: 
 134:   /// Return true if this addressing mode is already PC-relative.
 135:   bool isPCRelative() const {
 136:     if (BaseType != Base::RegBase)
 137:       return false;
 138:     if (auto *RegNode = dyn_cast_or_null<RegisterSDNode>(BaseReg.getNode()))
 139:       return RegNode->getReg() == M68k::PC;
 140:     return false;
```
- **EN**: The range implements or declares functions including `getDispSize`, `hasDisp`, `isDisp8`, `isDisp16`, `isDisp32`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `getDispSize`, `hasDisp`, `isDisp8`, `isDisp16`, `isDisp32` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 141-168 / 第 141-168 行
```cpp
 141:   }
 142: 
 143:   void setBaseReg(SDValue Reg) {
 144:     BaseType = Base::RegBase;
 145:     BaseReg = Reg;
 146:   }
 147: 
 148:   void setIndexReg(SDValue Reg) { IndexReg = Reg; }
 149: 
 150: #if !defined(NDEBUG) || defined(LLVM_ENABLE_DUMP)
 151:   void dump() {
 152:     dbgs() << "M68kISelAddressMode " << this;
 153:     dbgs() << "\nDisp: " << Disp;
 154:     dbgs() << ", BaseReg: ";
 155:     if (BaseReg.getNode())
 156:       BaseReg.getNode()->dump();
 157:     else
 158:       dbgs() << "null";
 159:     dbgs() << ", BaseFI: " << BaseFrameIndex;
 160:     dbgs() << ", IndexReg: ";
 161:     if (IndexReg.getNode()) {
 162:       IndexReg.getNode()->dump();
 163:     } else {
 164:       dbgs() << "null";
 165:       dbgs() << ", Scale: " << Scale;
 166:     }
 167:     dbgs() << '\n';
 168:   }
```
- **EN**: The range implements or declares functions including `setBaseReg`, `setIndexReg`, `dump`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `setBaseReg`, `setIndexReg`, `dump` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 169-196 / 第 169-196 行
```cpp
 169: #endif
 170: };
 171: } // end anonymous namespace
 172: 
 173: namespace {
 174: 
 175: // Helper type used by isSafeStoreLoad. Used to determine if
 176: // it is safe to fold a load and store into a single operation.
 177: struct CallSeqChainInfo {
 178:   // The nearest callseq_{start/end} (or lowered equivalent)
 179:   // in the chain of the load or store currently being analyzed.
 180:   SDNode *Node = nullptr;
 181:   // True when a TokenFactor introduces a dependency on more than one
 182:   // chain with a callseq_{start/end} (or lowered equivalent) to the load
 183:   // or store currently being analyzed
 184:   bool Multiple = false;
 185: };
 186: 
 187: static bool isCallSeqNode(const SDNode *N) {
 188:   if (N->getOpcode() == ISD::CALLSEQ_START ||
 189:       N->getOpcode() == ISD::CALLSEQ_END)
 190:     return true;
 191:   if (N->isMachineOpcode()) {
 192:     unsigned Opc = N->getMachineOpcode();
 193:     return Opc == M68k::ADJCALLSTACKDOWN || Opc == M68k::ADJCALLSTACKUP;
 194:   }
 195:   return false;
 196: }
```
- **EN**: It introduces interface types such as `CallSeqChainInfo`, shaping how other backend components interact with this file. The range implements or declares functions including `isCallSeqNode`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 它引入了 `CallSeqChainInfo` 等接口类型，定义了其他后端组件与本文件交互的方式。 这一段实现或声明了 `isCallSeqNode` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 197-224 / 第 197-224 行
```cpp
 197: 
 198: static CallSeqChainInfo getCallSeqChainInfo(SDValue Chain) {
 199:   SmallVector<SDValue, 8> Worklist = {Chain};
 200:   SmallPtrSet<SDNode *, 16> Visited;
 201:   SDNode *Found = nullptr;
 202: 
 203:   while (!Worklist.empty()) {
 204:     SDNode *CN = Worklist.pop_back_val().getNode();
 205:     if (!CN || !Visited.insert(CN).second)
 206:       continue;
 207: 
 208:     if (isCallSeqNode(CN)) {
 209:       if (!Found)
 210:         Found = CN;
 211:       else if (Found != CN)
 212:         return CallSeqChainInfo{nullptr, true};
 213:     }
 214: 
 215:     if (CN->getOpcode() == ISD::TokenFactor) {
 216:       for (const SDValue &Op : CN->op_values())
 217:         if (Op.getValueType() == MVT::Other)
 218:           Worklist.push_back(Op);
 219:       continue;
 220:     }
 221: 
 222:     for (const SDValue &Op : CN->op_values()) {
 223:       if (Op.getValueType() == MVT::Other) {
 224:         if (Worklist.size() == 8) {
```
- **EN**: The range implements or declares functions including `getCallSeqChainInfo`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `getCallSeqChainInfo` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 225-252 / 第 225-252 行
```cpp
 225:           // We can't actually evaluate all branches,
 226:           // be pessimistic and fail out.
 227:           return CallSeqChainInfo{nullptr, true};
 228:         }
 229:         Worklist.push_back(Op);
 230:         break;
 231:       }
 232:     }
 233:   }
 234: 
 235:   return CallSeqChainInfo{Found, false};
 236: }
 237: 
 238: // Helper for use in TableGen. We can't safely use a combined load/store in the
 239: // case where a token factor can cause a chain dep on a different call sequence.
 240: // Look for that case and return false if we can't confirm it's safe. This is
 241: // necessary due to the nesting level tracking in
 242: // ScheduleDAGRRList::FindCallSeqStart.
 243: static bool isSafeStoreLoad(SDNode *N) {
 244:   auto *ST = dyn_cast<StoreSDNode>(N);
 245:   if (!ST)
 246:     return false;
 247:   auto *LD = dyn_cast<LoadSDNode>(ST->getValue());
 248:   if (!LD)
 249:     return false;
 250:   // Load and store chains can be unrelated; guard against either side
 251:   // depending on a different call sequence boundary.
 252:   CallSeqChainInfo LoadInfo = getCallSeqChainInfo(LD->getChain());
```
- **EN**: The range implements or declares functions including `isSafeStoreLoad`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `isSafeStoreLoad` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 253-280 / 第 253-280 行
```cpp
 253:   CallSeqChainInfo StoreInfo = getCallSeqChainInfo(ST->getChain());
 254:   if (LoadInfo.Multiple || StoreInfo.Multiple)
 255:     return false;
 256:   if (!LoadInfo.Node && !StoreInfo.Node)
 257:     return true;
 258:   return LoadInfo.Node && StoreInfo.Node && LoadInfo.Node == StoreInfo.Node;
 259: }
 260: 
 261: class M68kDAGToDAGISel : public SelectionDAGISel {
 262: public:
 263:   M68kDAGToDAGISel() = delete;
 264: 
 265:   explicit M68kDAGToDAGISel(M68kTargetMachine &TM)
 266:       : SelectionDAGISel(TM), Subtarget(nullptr) {}
 267: 
 268:   bool runOnMachineFunction(MachineFunction &MF) override;
 269:   bool IsProfitableToFold(SDValue N, SDNode *U, SDNode *Root) const override;
 270: 
 271: private:
 272:   /// Keep a pointer to the M68kSubtarget around so that we can
 273:   /// make the right decision when generating code for different targets.
 274:   const M68kSubtarget *Subtarget;
 275: 
 276: // Include the pieces autogenerated from the target description.
 277: #include "M68kGenDAGISel.inc"
 278: 
 279:   /// getTargetMachine - Return a reference to the TargetMachine, casted
 280:   /// to the target-specific type.
```
- **EN**: It imports dependencies such as `M68kGenDAGISel.inc` that expose the LLVM and target interfaces used in later logic. This block declares or refines TableGen records such as `M68kDAGToDAGISel`. The range implements or declares functions including `M68kDAGToDAGISel`.
- **CN**: 它引入了 `M68kGenDAGISel.inc` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 该代码块声明或细化了 `M68kDAGToDAGISel` 等 TableGen 记录。 这一段实现或声明了 `M68kDAGToDAGISel` 等函数。

### Lines 281-308 / 第 281-308 行
```cpp
 281:   const M68kTargetMachine &getTargetMachine() {
 282:     return static_cast<const M68kTargetMachine &>(TM);
 283:   }
 284: 
 285:   void Select(SDNode *N) override;
 286: 
 287:   // Insert instructions to initialize the global base register in the
 288:   // first MBB of the function.
 289:   // HMM... do i need this?
 290:   void initGlobalBaseReg(MachineFunction &MF);
 291: 
 292:   bool foldOffsetIntoAddress(uint64_t Offset, M68kISelAddressMode &AM);
 293: 
 294:   bool matchLoadInAddress(LoadSDNode *N, M68kISelAddressMode &AM);
 295:   bool matchAddress(SDValue N, M68kISelAddressMode &AM);
 296:   bool matchAddressBase(SDValue N, M68kISelAddressMode &AM);
 297:   bool matchAddressRecursively(SDValue N, M68kISelAddressMode &AM,
 298:                                unsigned Depth);
 299:   bool matchADD(SDValue &N, M68kISelAddressMode &AM, unsigned Depth);
 300:   bool matchWrapper(SDValue N, M68kISelAddressMode &AM);
 301: 
 302:   std::pair<bool, SDNode *> selectNode(SDNode *Node);
 303: 
 304:   bool SelectARI(SDNode *Parent, SDValue N, SDValue &Base);
 305:   bool SelectARIPI(SDNode *Parent, SDValue N, SDValue &Base);
 306:   bool SelectARIPD(SDNode *Parent, SDValue N, SDValue &Base);
 307:   bool SelectARID(SDNode *Parent, SDValue N, SDValue &Imm, SDValue &Base);
 308:   bool SelectARII(SDNode *Parent, SDValue N, SDValue &Imm, SDValue &Base,
```
- **EN**: This span continues the file's main responsibility: this file selects target instructions from SelectionDAG nodes for the M68k backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 309-336 / 第 309-336 行
```cpp
 309:                   SDValue &Index);
 310:   bool SelectAL(SDNode *Parent, SDValue N, SDValue &Sym);
 311:   bool SelectPCD(SDNode *Parent, SDValue N, SDValue &Imm);
 312:   bool SelectPCI(SDNode *Parent, SDValue N, SDValue &Imm, SDValue &Index);
 313: 
 314:   bool SelectInlineAsmMemoryOperand(const SDValue &Op,
 315:                                     InlineAsm::ConstraintCode ConstraintID,
 316:                                     std::vector<SDValue> &OutOps) override;
 317: 
 318:   // If Address Mode represents Frame Index store FI in Disp and
 319:   // Displacement bit size in Base. These values are read symmetrically by
 320:   // M68kRegisterInfo::eliminateFrameIndex method
 321:   inline bool getFrameIndexAddress(M68kISelAddressMode &AM, const SDLoc &DL,
 322:                                    SDValue &Disp, SDValue &Base) {
 323:     if (AM.BaseType == M68kISelAddressMode::Base::FrameIndexBase) {
 324:       Disp = getI32Imm(AM.Disp, DL);
 325:       Base = CurDAG->getTargetFrameIndex(
 326:           AM.BaseFrameIndex, TLI->getPointerTy(CurDAG->getDataLayout()));
 327:       return true;
 328:     }
 329: 
 330:     return false;
 331:   }
 332: 
 333:   // Gets a symbol plus optional displacement
 334:   inline bool getSymbolicDisplacement(M68kISelAddressMode &AM, const SDLoc &DL,
 335:                                       SDValue &Sym) {
 336:     if (AM.GV) {
```
- **EN**: The range implements or declares functions including `getFrameIndexAddress`, `getSymbolicDisplacement`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `getFrameIndexAddress`, `getSymbolicDisplacement` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 337-364 / 第 337-364 行
```cpp
 337:       Sym = CurDAG->getTargetGlobalAddress(AM.GV, SDLoc(), MVT::i32, AM.Disp,
 338:                                            AM.SymbolFlags);
 339:       return true;
 340:     }
 341: 
 342:     if (AM.CP) {
 343:       Sym = CurDAG->getTargetConstantPool(AM.CP, MVT::i32, AM.Alignment,
 344:                                           AM.Disp, AM.SymbolFlags);
 345:       return true;
 346:     }
 347: 
 348:     if (AM.ES) {
 349:       assert(!AM.Disp && "Non-zero displacement is ignored with ES.");
 350:       Sym = CurDAG->getTargetExternalSymbol(AM.ES, MVT::i32, AM.SymbolFlags);
 351:       return true;
 352:     }
 353: 
 354:     if (AM.MCSym) {
 355:       assert(!AM.Disp && "Non-zero displacement is ignored with MCSym.");
 356:       assert(AM.SymbolFlags == 0 && "oo");
 357:       Sym = CurDAG->getMCSymbol(AM.MCSym, MVT::i32);
 358:       return true;
 359:     }
 360: 
 361:     if (AM.JT != -1) {
 362:       assert(!AM.Disp && "Non-zero displacement is ignored with JT.");
 363:       Sym = CurDAG->getTargetJumpTable(AM.JT, MVT::i32, AM.SymbolFlags);
 364:       return true;
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 365-392 / 第 365-392 行
```cpp
 365:     }
 366: 
 367:     if (AM.BlockAddr) {
 368:       Sym = CurDAG->getTargetBlockAddress(AM.BlockAddr, MVT::i32, AM.Disp,
 369:                                           AM.SymbolFlags);
 370:       return true;
 371:     }
 372: 
 373:     return false;
 374:   }
 375: 
 376:   /// Return a target constant with the specified value of type i8.
 377:   inline SDValue getI8Imm(int64_t Imm, const SDLoc &DL) {
 378:     return CurDAG->getSignedTargetConstant(Imm, DL, MVT::i8);
 379:   }
 380: 
 381:   /// Return a target constant with the specified value of type i8.
 382:   inline SDValue getI16Imm(int64_t Imm, const SDLoc &DL) {
 383:     return CurDAG->getSignedTargetConstant(Imm, DL, MVT::i16);
 384:   }
 385: 
 386:   /// Return a target constant with the specified value, of type i32.
 387:   inline SDValue getI32Imm(int64_t Imm, const SDLoc &DL) {
 388:     return CurDAG->getSignedTargetConstant(Imm, DL, MVT::i32);
 389:   }
 390: 
 391:   /// Return a reference to the TargetInstrInfo, casted to the target-specific
 392:   /// type.
```
- **EN**: The range implements or declares functions including `getI8Imm`, `getI16Imm`, `getI32Imm`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `getI8Imm`, `getI16Imm`, `getI32Imm` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 393-420 / 第 393-420 行
```cpp
 393:   const M68kInstrInfo *getInstrInfo() const {
 394:     return Subtarget->getInstrInfo();
 395:   }
 396: 
 397:   /// Return an SDNode that returns the value of the global base register.
 398:   /// Output instructions required to initialize the global base register,
 399:   /// if necessary.
 400:   SDNode *getGlobalBaseReg();
 401: };
 402: 
 403: class M68kDAGToDAGISelLegacy : public SelectionDAGISelLegacy {
 404: public:
 405:   static char ID;
 406:   explicit M68kDAGToDAGISelLegacy(M68kTargetMachine &TM)
 407:       : SelectionDAGISelLegacy(ID, std::make_unique<M68kDAGToDAGISel>(TM)) {}
 408: };
 409: 
 410: char M68kDAGToDAGISelLegacy::ID;
 411: 
 412: } // namespace
 413: 
 414: INITIALIZE_PASS(M68kDAGToDAGISelLegacy, DEBUG_TYPE, PASS_NAME, false, false)
 415: 
 416: bool M68kDAGToDAGISel::IsProfitableToFold(SDValue N, SDNode *U,
 417:                                           SDNode *Root) const {
 418:   if (OptLevel == CodeGenOptLevel::None)
 419:     return false;
 420: 
```
- **EN**: This block declares or refines TableGen records such as `M68kDAGToDAGISelLegacy`. The range implements or declares functions including `M68kDAGToDAGISelLegacy`, `M68kDAGToDAGISel::IsProfitableToFold`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 该代码块声明或细化了 `M68kDAGToDAGISelLegacy` 等 TableGen 记录。 这一段实现或声明了 `M68kDAGToDAGISelLegacy`, `M68kDAGToDAGISel::IsProfitableToFold` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 421-448 / 第 421-448 行
```cpp
 421:   if (U == Root) {
 422:     switch (U->getOpcode()) {
 423:     default:
 424:       return true;
 425:     case M68kISD::SUB:
 426:     case ISD::SUB:
 427:       // Prefer NEG instruction when zero subtracts a value.
 428:       // e.g.
 429:       //   move.l	#0, %d0
 430:       //   sub.l	(4,%sp), %d0
 431:       // vs.
 432:       //   move.l	(4,%sp), %d0
 433:       //   neg.l	%d0
 434:       if (llvm::isNullConstant(U->getOperand(0)))
 435:         return false;
 436:       break;
 437:     }
 438:   }
 439: 
 440:   return true;
 441: }
 442: 
 443: bool M68kDAGToDAGISel::runOnMachineFunction(MachineFunction &MF) {
 444:   Subtarget = &MF.getSubtarget<M68kSubtarget>();
 445:   return SelectionDAGISel::runOnMachineFunction(MF);
 446: }
 447: 
 448: /// This pass converts a legalized DAG into a M68k-specific DAG,
```
- **EN**: The range implements or declares functions including `M68kDAGToDAGISel::runOnMachineFunction`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `M68kDAGToDAGISel::runOnMachineFunction` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 449-476 / 第 449-476 行
```cpp
 449: /// ready for instruction scheduling.
 450: FunctionPass *llvm::createM68kISelDag(M68kTargetMachine &TM) {
 451:   return new M68kDAGToDAGISelLegacy(TM);
 452: }
 453: 
 454: static bool doesDispFitFI(M68kISelAddressMode &AM) {
 455:   if (!AM.isDispAddrType())
 456:     return false;
 457:   // -1 to make sure that resolved FI will fit into Disp field
 458:   return isIntN(AM.getDispSize() - 1, AM.Disp);
 459: }
 460: 
 461: static bool doesDispFit(M68kISelAddressMode &AM, int64_t Val) {
 462:   if (!AM.isDispAddrType())
 463:     return false;
 464:   return isIntN(AM.getDispSize(), Val);
 465: }
 466: 
 467: /// Return an SDNode that returns the value of the global base register.
 468: /// Output instructions required to initialize the global base register,
 469: /// if necessary.
 470: SDNode *M68kDAGToDAGISel::getGlobalBaseReg() {
 471:   unsigned GlobalBaseReg = getInstrInfo()->getGlobalBaseReg(MF);
 472:   auto &DL = MF->getDataLayout();
 473:   return CurDAG->getRegister(GlobalBaseReg, TLI->getPointerTy(DL)).getNode();
 474: }
 475: 
 476: bool M68kDAGToDAGISel::foldOffsetIntoAddress(uint64_t Offset,
```
- **EN**: The range implements or declares functions including `doesDispFitFI`, `doesDispFit`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `doesDispFitFI`, `doesDispFit` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 477-504 / 第 477-504 行
```cpp
 477:                                              M68kISelAddressMode &AM) {
 478:   // Cannot combine ExternalSymbol displacements with integer offsets.
 479:   if (Offset != 0 && (AM.ES || AM.MCSym))
 480:     return false;
 481: 
 482:   int64_t Val = AM.Disp + Offset;
 483: 
 484:   if (doesDispFit(AM, Val)) {
 485:     AM.Disp = Val;
 486:     return true;
 487:   }
 488: 
 489:   return false;
 490: }
 491: 
 492: //===----------------------------------------------------------------------===//
 493: // Matchers
 494: //===----------------------------------------------------------------------===//
 495: 
 496: /// Helper for MatchAddress. Add the specified node to the
 497: /// specified addressing mode without any further recursion.
 498: bool M68kDAGToDAGISel::matchAddressBase(SDValue N, M68kISelAddressMode &AM) {
 499:   // Is the base register already occupied?
 500:   if (AM.hasBase()) {
 501:     // If so, check to see if the scale index register is set.
 502:     if (!AM.hasIndexReg()) {
 503:       AM.IndexReg = N;
 504:       AM.Scale = 1;
```
- **EN**: The range implements or declares functions including `M68kDAGToDAGISel::matchAddressBase`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `M68kDAGToDAGISel::matchAddressBase` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 505-532 / 第 505-532 行
```cpp
 505:       return true;
 506:     }
 507: 
 508:     // Otherwise, we cannot select it.
 509:     return false;
 510:   }
 511: 
 512:   // Default, generate it as a register.
 513:   AM.BaseType = M68kISelAddressMode::Base::RegBase;
 514:   AM.BaseReg = N;
 515:   return true;
 516: }
 517: 
 518: /// TODO Add TLS support
 519: bool M68kDAGToDAGISel::matchLoadInAddress(LoadSDNode *N,
 520:                                           M68kISelAddressMode &AM) {
 521:   return false;
 522: }
 523: 
 524: bool M68kDAGToDAGISel::matchAddressRecursively(SDValue N,
 525:                                                M68kISelAddressMode &AM,
 526:                                                unsigned Depth) {
 527:   SDLoc DL(N);
 528: 
 529:   // Limit recursion.
 530:   if (Depth > 5)
 531:     return matchAddressBase(N, AM);
 532: 
```
- **EN**: The range implements or declares functions including `M68kDAGToDAGISel::matchLoadInAddress`, `M68kDAGToDAGISel::matchAddressRecursively`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `M68kDAGToDAGISel::matchLoadInAddress`, `M68kDAGToDAGISel::matchAddressRecursively` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 533-560 / 第 533-560 行
```cpp
 533:   // If this is already a %PC relative address, we can only merge immediates
 534:   // into it.  Instead of handling this in every case, we handle it here.
 535:   // PC relative addressing: %PC + 16-bit displacement!
 536:   if (AM.isPCRelative()) {
 537:     // FIXME JumpTable and ExternalSymbol address currently don't like
 538:     // displacements.  It isn't very important, but should be fixed for
 539:     // consistency.
 540: 
 541:     if (ConstantSDNode *Cst = dyn_cast<ConstantSDNode>(N))
 542:       if (foldOffsetIntoAddress(Cst->getSExtValue(), AM))
 543:         return true;
 544:     return false;
 545:   }
 546: 
 547:   switch (N.getOpcode()) {
 548:   default:
 549:     break;
 550: 
 551:   case ISD::Constant: {
 552:     uint64_t Val = cast<ConstantSDNode>(N)->getSExtValue();
 553:     if (foldOffsetIntoAddress(Val, AM))
 554:       return true;
 555:     break;
 556:   }
 557: 
 558:   case M68kISD::Wrapper:
 559:   case M68kISD::WrapperPC:
 560:     if (matchWrapper(N, AM))
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 561-588 / 第 561-588 行
```cpp
 561:       return true;
 562:     break;
 563: 
 564:   case ISD::LOAD:
 565:     if (matchLoadInAddress(cast<LoadSDNode>(N), AM))
 566:       return true;
 567:     break;
 568: 
 569:   case ISD::OR:
 570:     // We want to look through a transform in InstCombine and DAGCombiner that
 571:     // turns 'add' into 'or', so we can treat this 'or' exactly like an 'add'.
 572:     // Example: (or (and x, 1), (shl y, 3)) --> (add (and x, 1), (shl y, 3))
 573:     // An 'lea' can then be used to match the shift (multiply) and add:
 574:     // and $1, %esi
 575:     // lea (%rsi, %rdi, 8), %rax
 576:     if (CurDAG->haveNoCommonBitsSet(N.getOperand(0), N.getOperand(1)) &&
 577:         matchADD(N, AM, Depth))
 578:       return true;
 579:     break;
 580: 
 581:   case ISD::ADD:
 582:     if (matchADD(N, AM, Depth))
 583:       return true;
 584:     break;
 585: 
 586:   case ISD::FrameIndex:
 587:     if (AM.isDispAddrType() &&
 588:         AM.BaseType == M68kISelAddressMode::Base::RegBase &&
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 589-616 / 第 589-616 行
```cpp
 589:         AM.BaseReg.getNode() == nullptr && doesDispFitFI(AM)) {
 590:       AM.BaseType = M68kISelAddressMode::Base::FrameIndexBase;
 591:       AM.BaseFrameIndex = cast<FrameIndexSDNode>(N)->getIndex();
 592:       return true;
 593:     }
 594:     break;
 595: 
 596:   case ISD::TargetGlobalTLSAddress: {
 597:     GlobalAddressSDNode *GA = cast<GlobalAddressSDNode>(N);
 598:     AM.GV = GA->getGlobal();
 599:     AM.SymbolFlags = GA->getTargetFlags();
 600:     return true;
 601:   }
 602:   }
 603: 
 604:   return matchAddressBase(N, AM);
 605: }
 606: 
 607: /// Add the specified node to the specified addressing mode, returning true if
 608: /// it cannot be done. This just pattern matches for the addressing mode.
 609: bool M68kDAGToDAGISel::matchAddress(SDValue N, M68kISelAddressMode &AM) {
 610:   // TODO: Post-processing: Convert lea(,%reg,2) to lea(%reg,%reg), which has
 611:   // a smaller encoding and avoids a scaled-index.
 612:   // And make sure it is an indexed mode
 613: 
 614:   // TODO: Post-processing: Convert foo to foo(%pc), even in non-PIC mode,
 615:   // because it has a smaller encoding.
 616:   // Make sure this must be done only if PC* modes are currently being matched
```
- **EN**: The range implements or declares functions including `M68kDAGToDAGISel::matchAddress`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `M68kDAGToDAGISel::matchAddress` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 617-644 / 第 617-644 行
```cpp
 617:   return matchAddressRecursively(N, AM, 0);
 618: }
 619: 
 620: bool M68kDAGToDAGISel::matchADD(SDValue &N, M68kISelAddressMode &AM,
 621:                                 unsigned Depth) {
 622:   // Add an artificial use to this node so that we can keep track of
 623:   // it if it gets CSE'd with a different node.
 624:   HandleSDNode Handle(N);
 625: 
 626:   M68kISelAddressMode Backup = AM;
 627:   if (matchAddressRecursively(N.getOperand(0), AM, Depth + 1) &&
 628:       matchAddressRecursively(Handle.getValue().getOperand(1), AM, Depth + 1)) {
 629:     return true;
 630:   }
 631:   AM = Backup;
 632: 
 633:   // Try again after commuting the operands.
 634:   if (matchAddressRecursively(Handle.getValue().getOperand(1), AM, Depth + 1) &&
 635:       matchAddressRecursively(Handle.getValue().getOperand(0), AM, Depth + 1)) {
 636:     return true;
 637:   }
 638:   AM = Backup;
 639: 
 640:   // If we couldn't fold both operands into the address at the same time,
 641:   // see if we can just put each operand into a register and fold at least
 642:   // the add.
 643:   if (!AM.hasBase() && !AM.hasIndexReg()) {
 644:     N = Handle.getValue();
```
- **EN**: The range implements or declares functions including `M68kDAGToDAGISel::matchADD`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `M68kDAGToDAGISel::matchADD` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 645-672 / 第 645-672 行
```cpp
 645:     AM.BaseReg = N.getOperand(0);
 646:     AM.IndexReg = N.getOperand(1);
 647:     AM.Scale = 1;
 648:     return true;
 649:   }
 650: 
 651:   N = Handle.getValue();
 652:   return false;
 653: }
 654: 
 655: /// Try to match M68kISD::Wrapper and M68kISD::WrapperPC nodes into an
 656: /// addressing mode. These wrap things that will resolve down into a symbol
 657: /// reference. If no match is possible, this returns true, otherwise it returns
 658: /// false.
 659: bool M68kDAGToDAGISel::matchWrapper(SDValue N, M68kISelAddressMode &AM) {
 660:   // If the addressing mode already has a symbol as the displacement, we can
 661:   // never match another symbol.
 662:   if (AM.hasSymbolicDisplacement())
 663:     return false;
 664: 
 665:   SDValue N0 = N.getOperand(0);
 666: 
 667:   if (N.getOpcode() == M68kISD::WrapperPC) {
 668: 
 669:     // If cannot match here just restore the old version
 670:     M68kISelAddressMode Backup = AM;
 671: 
 672:     if (AM.hasBase()) {
```
- **EN**: The range implements or declares functions including `M68kDAGToDAGISel::matchWrapper`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `M68kDAGToDAGISel::matchWrapper` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 673-700 / 第 673-700 行
```cpp
 673:       return false;
 674:     }
 675: 
 676:     if (auto *G = dyn_cast<GlobalAddressSDNode>(N0)) {
 677:       AM.GV = G->getGlobal();
 678:       AM.SymbolFlags = G->getTargetFlags();
 679:       if (!foldOffsetIntoAddress(G->getOffset(), AM)) {
 680:         AM = Backup;
 681:         return false;
 682:       }
 683:     } else if (auto *CP = dyn_cast<ConstantPoolSDNode>(N0)) {
 684:       AM.CP = CP->getConstVal();
 685:       AM.Alignment = CP->getAlign();
 686:       AM.SymbolFlags = CP->getTargetFlags();
 687:       if (!foldOffsetIntoAddress(CP->getOffset(), AM)) {
 688:         AM = Backup;
 689:         return false;
 690:       }
 691:     } else if (auto *S = dyn_cast<ExternalSymbolSDNode>(N0)) {
 692:       AM.ES = S->getSymbol();
 693:       AM.SymbolFlags = S->getTargetFlags();
 694:     } else if (auto *S = dyn_cast<MCSymbolSDNode>(N0)) {
 695:       AM.MCSym = S->getMCSymbol();
 696:     } else if (auto *J = dyn_cast<JumpTableSDNode>(N0)) {
 697:       AM.JT = J->getIndex();
 698:       AM.SymbolFlags = J->getTargetFlags();
 699:     } else if (auto *BA = dyn_cast<BlockAddressSDNode>(N0)) {
 700:       AM.BlockAddr = BA->getBlockAddress();
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 701-728 / 第 701-728 行
```cpp
 701:       AM.SymbolFlags = BA->getTargetFlags();
 702:       if (!foldOffsetIntoAddress(BA->getOffset(), AM)) {
 703:         AM = Backup;
 704:         return false;
 705:       }
 706:     } else
 707:       llvm_unreachable("Unhandled symbol reference node.");
 708: 
 709:     AM.setBaseReg(CurDAG->getRegister(M68k::PC, MVT::i32));
 710:     return true;
 711:   }
 712: 
 713:   // This wrapper requires 32bit disp/imm field for Medium CM
 714:   if (!AM.isDisp32()) {
 715:     return false;
 716:   }
 717: 
 718:   if (N.getOpcode() == M68kISD::Wrapper) {
 719:     if (auto *G = dyn_cast<GlobalAddressSDNode>(N0)) {
 720:       AM.GV = G->getGlobal();
 721:       AM.Disp += G->getOffset();
 722:       AM.SymbolFlags = G->getTargetFlags();
 723:     } else if (auto *CP = dyn_cast<ConstantPoolSDNode>(N0)) {
 724:       AM.CP = CP->getConstVal();
 725:       AM.Alignment = CP->getAlign();
 726:       AM.Disp += CP->getOffset();
 727:       AM.SymbolFlags = CP->getTargetFlags();
 728:     } else if (auto *S = dyn_cast<ExternalSymbolSDNode>(N0)) {
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 729-756 / 第 729-756 行
```cpp
 729:       AM.ES = S->getSymbol();
 730:       AM.SymbolFlags = S->getTargetFlags();
 731:     } else if (auto *S = dyn_cast<MCSymbolSDNode>(N0)) {
 732:       AM.MCSym = S->getMCSymbol();
 733:     } else if (auto *J = dyn_cast<JumpTableSDNode>(N0)) {
 734:       AM.JT = J->getIndex();
 735:       AM.SymbolFlags = J->getTargetFlags();
 736:     } else if (auto *BA = dyn_cast<BlockAddressSDNode>(N0)) {
 737:       AM.BlockAddr = BA->getBlockAddress();
 738:       AM.Disp += BA->getOffset();
 739:       AM.SymbolFlags = BA->getTargetFlags();
 740:     } else
 741:       llvm_unreachable("Unhandled symbol reference node.");
 742:     return true;
 743:   }
 744: 
 745:   return false;
 746: }
 747: 
 748: //===----------------------------------------------------------------------===//
 749: // Selectors
 750: //===----------------------------------------------------------------------===//
 751: 
 752: void M68kDAGToDAGISel::Select(SDNode *Node) {
 753:   unsigned Opcode = Node->getOpcode();
 754:   SDLoc DL(Node);
 755: 
 756:   LLVM_DEBUG(dbgs() << "Selecting: "; Node->dump(CurDAG); dbgs() << '\n');
```
- **EN**: The range implements or declares functions including `M68kDAGToDAGISel::Select`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `M68kDAGToDAGISel::Select` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 757-784 / 第 757-784 行
```cpp
 757: 
 758:   if (Node->isMachineOpcode()) {
 759:     LLVM_DEBUG(dbgs() << "== "; Node->dump(CurDAG); dbgs() << '\n');
 760:     Node->setNodeId(-1);
 761:     return; // Already selected.
 762:   }
 763: 
 764:   switch (Opcode) {
 765:   default:
 766:     break;
 767: 
 768:   case ISD::GLOBAL_OFFSET_TABLE: {
 769:     SDValue GOT = CurDAG->getTargetExternalSymbol(
 770:         "_GLOBAL_OFFSET_TABLE_", MVT::i32, M68kII::MO_GOTPCREL);
 771:     MachineSDNode *Res =
 772:         CurDAG->getMachineNode(M68k::LEA32q, DL, MVT::i32, GOT);
 773:     ReplaceNode(Node, Res);
 774:     return;
 775:   }
 776: 
 777:   case M68kISD::GLOBAL_BASE_REG:
 778:     ReplaceNode(Node, getGlobalBaseReg());
 779:     return;
 780:   }
 781: 
 782:   SelectCode(Node);
 783: }
 784: 
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 785-812 / 第 785-812 行
```cpp
 785: bool M68kDAGToDAGISel::SelectARIPI(SDNode *Parent, SDValue N, SDValue &Base) {
 786:   LLVM_DEBUG(dbgs() << "Selecting AddrType::ARIPI: ");
 787:   LLVM_DEBUG(dbgs() << "NOT IMPLEMENTED\n");
 788:   return false;
 789: }
 790: 
 791: bool M68kDAGToDAGISel::SelectARIPD(SDNode *Parent, SDValue N, SDValue &Base) {
 792:   LLVM_DEBUG(dbgs() << "Selecting AddrType::ARIPD: ");
 793:   LLVM_DEBUG(dbgs() << "NOT IMPLEMENTED\n");
 794:   return false;
 795: }
 796: 
 797: [[maybe_unused]] static bool allowARIDWithDisp(SDNode *Parent) {
 798:   if (!Parent)
 799:     return false;
 800:   switch (Parent->getOpcode()) {
 801:   case ISD::LOAD:
 802:   case ISD::STORE:
 803:   case ISD::ATOMIC_LOAD:
 804:   case ISD::ATOMIC_STORE:
 805:     return true;
 806:   default:
 807:     return false;
 808:   }
 809: }
 810: 
 811: bool M68kDAGToDAGISel::SelectARID(SDNode *Parent, SDValue N, SDValue &Disp,
 812:                                   SDValue &Base) {
```
- **EN**: The range implements or declares functions including `M68kDAGToDAGISel::SelectARIPI`, `M68kDAGToDAGISel::SelectARIPD`, `M68kDAGToDAGISel::SelectARID`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `M68kDAGToDAGISel::SelectARIPI`, `M68kDAGToDAGISel::SelectARIPD`, `M68kDAGToDAGISel::SelectARID` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 813-840 / 第 813-840 行
```cpp
 813:   LLVM_DEBUG(dbgs() << "Selecting AddrType::ARID: ");
 814:   M68kISelAddressMode AM(M68kISelAddressMode::AddrType::ARID);
 815: 
 816:   if (!matchAddress(N, AM))
 817:     return false;
 818: 
 819:   if (AM.isPCRelative()) {
 820:     LLVM_DEBUG(dbgs() << "REJECT: Cannot match PC relative address\n");
 821:     return false;
 822:   }
 823: 
 824:   // If this is a frame index, grab it
 825:   if (getFrameIndexAddress(AM, SDLoc(N), Disp, Base)) {
 826:     LLVM_DEBUG(dbgs() << "SUCCESS matched FI\n");
 827:     return true;
 828:   }
 829: 
 830:   if (AM.hasIndexReg()) {
 831:     LLVM_DEBUG(dbgs() << "REJECT: Cannot match Index\n");
 832:     return false;
 833:   }
 834: 
 835:   if (!AM.hasBaseReg()) {
 836:     LLVM_DEBUG(dbgs() << "REJECT: No Base reg\n");
 837:     return false;
 838:   }
 839: 
 840:   Base = AM.BaseReg;
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 841-868 / 第 841-868 行
```cpp
 841: 
 842:   if (getSymbolicDisplacement(AM, SDLoc(N), Disp)) {
 843:     assert((!AM.Disp || allowARIDWithDisp(Parent)) &&
 844:            "Should not be any displacement");
 845:     LLVM_DEBUG(dbgs() << "SUCCESS, matched Symbol\n");
 846:     return true;
 847:   }
 848: 
 849:   // Give a chance to AddrType::ARI
 850:   if (AM.Disp == 0) {
 851:     LLVM_DEBUG(dbgs() << "REJECT: No displacement\n");
 852:     return false;
 853:   }
 854: 
 855:   Disp = getI16Imm(AM.Disp, SDLoc(N));
 856: 
 857:   LLVM_DEBUG(dbgs() << "SUCCESS\n");
 858:   return true;
 859: }
 860: 
 861: static bool isAddressBase(const SDValue &N) {
 862:   switch (N.getOpcode()) {
 863:   case ISD::ADD:
 864:   case ISD::ADDC:
 865:     return llvm::any_of(N.getNode()->ops(),
 866:                         [](const SDUse &U) { return isAddressBase(U.get()); });
 867:   case M68kISD::Wrapper:
 868:   case M68kISD::WrapperPC:
```
- **EN**: The range implements or declares functions including `isAddressBase`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `isAddressBase` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 869-896 / 第 869-896 行
```cpp
 869:   case M68kISD::GLOBAL_BASE_REG:
 870:     return true;
 871:   default:
 872:     return false;
 873:   }
 874: }
 875: 
 876: static bool AllowARIIWithZeroDisp(SDNode *Parent) {
 877:   if (!Parent)
 878:     return false;
 879:   switch (Parent->getOpcode()) {
 880:   case ISD::LOAD:
 881:   case ISD::STORE:
 882:   case ISD::ATOMIC_LOAD:
 883:   case ISD::ATOMIC_STORE:
 884:   case ISD::ATOMIC_CMP_SWAP:
 885:     return true;
 886:   default:
 887:     return false;
 888:   }
 889: }
 890: 
 891: bool M68kDAGToDAGISel::SelectARII(SDNode *Parent, SDValue N, SDValue &Disp,
 892:                                   SDValue &Base, SDValue &Index) {
 893:   M68kISelAddressMode AM(M68kISelAddressMode::AddrType::ARII);
 894:   LLVM_DEBUG(dbgs() << "Selecting AddrType::ARII: ");
 895: 
 896:   if (!matchAddress(N, AM))
```
- **EN**: The range implements or declares functions including `AllowARIIWithZeroDisp`, `M68kDAGToDAGISel::SelectARII`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `AllowARIIWithZeroDisp`, `M68kDAGToDAGISel::SelectARII` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 897-924 / 第 897-924 行
```cpp
 897:     return false;
 898: 
 899:   if (AM.isPCRelative()) {
 900:     LLVM_DEBUG(dbgs() << "REJECT: PC relative\n");
 901:     return false;
 902:   }
 903: 
 904:   if (!AM.hasIndexReg()) {
 905:     LLVM_DEBUG(dbgs() << "REJECT: No Index\n");
 906:     return false;
 907:   }
 908: 
 909:   if (!AM.hasBaseReg()) {
 910:     LLVM_DEBUG(dbgs() << "REJECT: No Base\n");
 911:     return false;
 912:   }
 913: 
 914:   if (!isAddressBase(AM.BaseReg) && isAddressBase(AM.IndexReg)) {
 915:     Base = AM.IndexReg;
 916:     Index = AM.BaseReg;
 917:   } else {
 918:     Base = AM.BaseReg;
 919:     Index = AM.IndexReg;
 920:   }
 921: 
 922:   if (AM.hasSymbolicDisplacement()) {
 923:     LLVM_DEBUG(dbgs() << "REJECT, Cannot match symbolic displacement\n");
 924:     return false;
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 925-952 / 第 925-952 行
```cpp
 925:   }
 926: 
 927:   // The idea here is that we want to use AddrType::ARII without displacement
 928:   // only if necessary like memory operations, otherwise this must be lowered
 929:   // into addition
 930:   if (AM.Disp == 0 && !AllowARIIWithZeroDisp(Parent)) {
 931:     LLVM_DEBUG(dbgs() << "REJECT: Displacement is Zero\n");
 932:     return false;
 933:   }
 934: 
 935:   Disp = getI8Imm(AM.Disp, SDLoc(N));
 936: 
 937:   LLVM_DEBUG(dbgs() << "SUCCESS\n");
 938:   return true;
 939: }
 940: 
 941: bool M68kDAGToDAGISel::SelectAL(SDNode *Parent, SDValue N, SDValue &Sym) {
 942:   LLVM_DEBUG(dbgs() << "Selecting AddrType::AL: ");
 943:   M68kISelAddressMode AM(M68kISelAddressMode::AddrType::AL);
 944: 
 945:   if (!matchAddress(N, AM)) {
 946:     LLVM_DEBUG(dbgs() << "REJECT: Match failed\n");
 947:     return false;
 948:   }
 949: 
 950:   if (AM.isPCRelative()) {
 951:     LLVM_DEBUG(dbgs() << "REJECT: Cannot match PC relative address\n");
 952:     return false;
```
- **EN**: The range implements or declares functions including `M68kDAGToDAGISel::SelectAL`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `M68kDAGToDAGISel::SelectAL` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 953-980 / 第 953-980 行
```cpp
 953:   }
 954: 
 955:   if (AM.hasBase()) {
 956:     LLVM_DEBUG(dbgs() << "REJECT: Cannot match Base\n");
 957:     return false;
 958:   }
 959: 
 960:   if (AM.hasIndexReg()) {
 961:     LLVM_DEBUG(dbgs() << "REJECT: Cannot match Index\n");
 962:     return false;
 963:   }
 964: 
 965:   if (getSymbolicDisplacement(AM, SDLoc(N), Sym)) {
 966:     LLVM_DEBUG(dbgs() << "SUCCESS: Matched symbol\n");
 967:     return true;
 968:   }
 969: 
 970:   if (AM.Disp) {
 971:     Sym = getI32Imm(AM.Disp, SDLoc(N));
 972:     LLVM_DEBUG(dbgs() << "SUCCESS\n");
 973:     return true;
 974:   }
 975: 
 976:   LLVM_DEBUG(dbgs() << "REJECT: Not Symbol or Disp\n");
 977:   return false;
 978:   ;
 979: }
 980: 
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 981-1008 / 第 981-1008 行
```cpp
 981: bool M68kDAGToDAGISel::SelectPCD(SDNode *Parent, SDValue N, SDValue &Disp) {
 982:   LLVM_DEBUG(dbgs() << "Selecting AddrType::PCD: ");
 983:   M68kISelAddressMode AM(M68kISelAddressMode::AddrType::PCD);
 984: 
 985:   if (!matchAddress(N, AM))
 986:     return false;
 987: 
 988:   if (!AM.isPCRelative()) {
 989:     LLVM_DEBUG(dbgs() << "REJECT: Not PC relative\n");
 990:     return false;
 991:   }
 992: 
 993:   if (AM.hasIndexReg()) {
 994:     LLVM_DEBUG(dbgs() << "REJECT: Cannot match Index\n");
 995:     return false;
 996:   }
 997: 
 998:   if (getSymbolicDisplacement(AM, SDLoc(N), Disp)) {
 999:     LLVM_DEBUG(dbgs() << "SUCCESS, matched Symbol\n");
1000:     return true;
1001:   }
1002: 
1003:   Disp = getI16Imm(AM.Disp, SDLoc(N));
1004: 
1005:   LLVM_DEBUG(dbgs() << "SUCCESS\n");
1006:   return true;
1007: }
1008: 
```
- **EN**: The range implements or declares functions including `M68kDAGToDAGISel::SelectPCD`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `M68kDAGToDAGISel::SelectPCD` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1009-1036 / 第 1009-1036 行
```cpp
1009: bool M68kDAGToDAGISel::SelectPCI(SDNode *Parent, SDValue N, SDValue &Disp,
1010:                                  SDValue &Index) {
1011:   LLVM_DEBUG(dbgs() << "Selecting AddrType::PCI: ");
1012:   M68kISelAddressMode AM(M68kISelAddressMode::AddrType::PCI);
1013: 
1014:   if (!matchAddress(N, AM))
1015:     return false;
1016: 
1017:   if (!AM.isPCRelative()) {
1018:     LLVM_DEBUG(dbgs() << "REJECT: Not PC relative\n");
1019:     return false;
1020:   }
1021: 
1022:   if (!AM.hasIndexReg()) {
1023:     LLVM_DEBUG(dbgs() << "REJECT: No Index\n");
1024:     return false;
1025:   }
1026: 
1027:   Index = AM.IndexReg;
1028: 
1029:   if (getSymbolicDisplacement(AM, SDLoc(N), Disp)) {
1030:     assert(!AM.Disp && "Should not be any displacement");
1031:     LLVM_DEBUG(dbgs() << "SUCCESS, matched Symbol\n");
1032:     return true;
1033:   }
1034: 
1035:   Disp = getI8Imm(AM.Disp, SDLoc(N));
1036: 
```
- **EN**: The range implements or declares functions including `M68kDAGToDAGISel::SelectPCI`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `M68kDAGToDAGISel::SelectPCI` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 1037-1064 / 第 1037-1064 行
```cpp
1037:   LLVM_DEBUG(dbgs() << "SUCCESS\n");
1038:   return true;
1039: }
1040: 
1041: bool M68kDAGToDAGISel::SelectARI(SDNode *Parent, SDValue N, SDValue &Base) {
1042:   LLVM_DEBUG(dbgs() << "Selecting AddrType::ARI: ");
1043:   M68kISelAddressMode AM(M68kISelAddressMode::AddrType::ARI);
1044: 
1045:   if (!matchAddress(N, AM)) {
1046:     LLVM_DEBUG(dbgs() << "REJECT: Match failed\n");
1047:     return false;
1048:   }
1049: 
1050:   if (AM.isPCRelative()) {
1051:     LLVM_DEBUG(dbgs() << "REJECT: Cannot match PC relative address\n");
1052:     return false;
1053:   }
1054: 
1055:   // AddrType::ARI does not use these
1056:   if (AM.hasIndexReg() || AM.Disp != 0) {
1057:     LLVM_DEBUG(dbgs() << "REJECT: Cannot match Index or Disp\n");
1058:     return false;
1059:   }
1060: 
1061:   // Must be matched by AddrType::AL
1062:   if (AM.hasSymbolicDisplacement()) {
1063:     LLVM_DEBUG(dbgs() << "REJECT: Cannot match Symbolic Disp\n");
1064:     return false;
```
- **EN**: The range implements or declares functions including `M68kDAGToDAGISel::SelectARI`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `M68kDAGToDAGISel::SelectARI` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1065-1092 / 第 1065-1092 行
```cpp
1065:   }
1066: 
1067:   if (AM.hasBaseReg()) {
1068:     Base = AM.BaseReg;
1069:     LLVM_DEBUG(dbgs() << "SUCCESS\n");
1070:     return true;
1071:   }
1072: 
1073:   return false;
1074: }
1075: 
1076: bool M68kDAGToDAGISel::SelectInlineAsmMemoryOperand(
1077:     const SDValue &Op, InlineAsm::ConstraintCode ConstraintID,
1078:     std::vector<SDValue> &OutOps) {
1079:   // In order to tell AsmPrinter the exact addressing mode we select here, which
1080:   // might comprise of multiple SDValues (hence MachineOperands), a 32-bit
1081:   // immediate value is prepended to the list of selected SDValues to indicate
1082:   // the addressing mode kind.
1083:   using AMK = M68k::MemAddrModeKind;
1084:   auto addKind = [this](SDValue &Opnd, AMK Kind) -> bool {
1085:     Opnd = CurDAG->getTargetConstant(unsigned(Kind), SDLoc(), MVT::i32);
1086:     return true;
1087:   };
1088: 
1089:   switch (ConstraintID) {
1090:   // Generic memory operand.
1091:   case InlineAsm::ConstraintCode::m: {
1092:     // Try every supported (memory) addressing modes.
```
- **EN**: The range implements or declares functions including `M68kDAGToDAGISel::SelectInlineAsmMemoryOperand`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `M68kDAGToDAGISel::SelectInlineAsmMemoryOperand` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1093-1120 / 第 1093-1120 行
```cpp
1093:     SDValue Operands[4];
1094: 
1095:     // TODO: The ordering of the following SelectXXX is relatively...arbitrary,
1096:     // right now we simply sort them by descending complexity. Maybe we should
1097:     // adjust this by code model and/or relocation mode in the future.
1098:     if (SelectARII(nullptr, Op, Operands[1], Operands[2], Operands[3]) &&
1099:         addKind(Operands[0], AMK::f)) {
1100:       OutOps.insert(OutOps.end(), &Operands[0], Operands + 4);
1101:       return false;
1102:     }
1103: 
1104:     if ((SelectPCI(nullptr, Op, Operands[1], Operands[2]) &&
1105:          addKind(Operands[0], AMK::k)) ||
1106:         (SelectARID(nullptr, Op, Operands[1], Operands[2]) &&
1107:          addKind(Operands[0], AMK::p))) {
1108:       OutOps.insert(OutOps.end(), &Operands[0], Operands + 3);
1109:       return false;
1110:     }
1111: 
1112:     if ((SelectPCD(nullptr, Op, Operands[1]) && addKind(Operands[0], AMK::q)) ||
1113:         (SelectARI(nullptr, Op, Operands[1]) && addKind(Operands[0], AMK::j)) ||
1114:         (SelectAL(nullptr, Op, Operands[1]) && addKind(Operands[0], AMK::b))) {
1115:       OutOps.insert(OutOps.end(), {Operands[0], Operands[1]});
1116:       return false;
1117:     }
1118: 
1119:     return true;
1120:   }
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1121-1146 / 第 1121-1146 行
```cpp
1121:   // 'Q': Address register indirect addressing.
1122:   case InlineAsm::ConstraintCode::Q: {
1123:     SDValue AMKind, Base;
1124:     // 'j' addressing mode.
1125:     // TODO: Add support for 'o' and 'e' after their
1126:     // select functions are implemented.
1127:     if (SelectARI(nullptr, Op, Base) && addKind(AMKind, AMK::j)) {
1128:       OutOps.insert(OutOps.end(), {AMKind, Base});
1129:       return false;
1130:     }
1131:     return true;
1132:   }
1133:   // 'U': Address register indirect w/ constant offset addressing.
1134:   case InlineAsm::ConstraintCode::Um: {
1135:     SDValue AMKind, Base, Offset;
1136:     // 'p' addressing mode.
1137:     if (SelectARID(nullptr, Op, Offset, Base) && addKind(AMKind, AMK::p)) {
1138:       OutOps.insert(OutOps.end(), {AMKind, Offset, Base});
1139:       return false;
1140:     }
1141:     return true;
1142:   }
1143:   default:
1144:     return true;
1145:   }
1146: }
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

## Key Concepts / 关键概念
- **TableGen DSL**: Describes backend metadata with declarative records and multiclasses. / 使用声明式记录和 multiclass 描述后端元数据。
- **SelectionDAG lowering**: Bridges LLVM IR and target-specific DAG nodes/instructions. / 连接 LLVM IR 与目标相关 DAG 节点/指令。
- **GlobalISel**: Uses legalization, register banks, and instruction selection after generic IR lowering. / 在通用 IR 降低后使用合法化、寄存器银行和指令选择。
- **MachineInstr**: Represents target-aware machine instructions during late code generation. / 表示代码生成后期的目标相关机器指令。
- **MachineFunction**: Carries per-function machine-level state and basic blocks. / 保存每个函数的机器级状态和基本块。
- **Assembly emission**: Prints target instructions and directives in textual assembly form. / 以文本汇编形式输出目标指令和伪指令。

## Dependencies / 依赖关系
- `M68k.h`
- `M68kMachineFunction.h`
- `M68kRegisterInfo.h`
- `M68kSelectionDAGInfo.h`
- `M68kTargetMachine.h`
- `llvm/CodeGen/MachineConstantPool.h`
- `llvm/CodeGen/MachineFrameInfo.h`
- `llvm/CodeGen/MachineFunction.h`
- `llvm/CodeGen/MachineInstrBuilder.h`
- `llvm/CodeGen/MachineRegisterInfo.h`
- `llvm/CodeGen/SelectionDAGISel.h`
- `llvm/CodeGen/SelectionDAGNodes.h`
- `llvm/IR/CFG.h`
- `llvm/IR/GlobalValue.h`
- `llvm/IR/Instructions.h`
- `llvm/IR/Intrinsics.h`
- `...` (8 more include dependencies omitted for brevity / 其余 8 个 include 依赖已省略)
