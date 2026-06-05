# SystemZISelDAGToDAG.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/SystemZISelDAGToDAG.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file selects target instructions from SelectionDAG nodes for the SystemZ backend.
- **用途 (CN)**: 该文件用于 SystemZ 后端，负责从 SelectionDAG 节点选择目标指令。

## Line-by-Line Analysis / 逐行分析
### Lines 1-40 / 第 1-40 行
```cpp
   1: //===-- SystemZISelDAGToDAG.cpp - A dag to dag inst selector for SystemZ --===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file defines an instruction selector for the SystemZ target.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: #include "SystemZTargetMachine.h"
  14: #include "SystemZISelLowering.h"
  15: #include "llvm/Analysis/AliasAnalysis.h"
  16: #include "llvm/CodeGen/SelectionDAGISel.h"
  17: #include "llvm/Support/Debug.h"
  18: #include "llvm/Support/KnownBits.h"
  19: #include "llvm/Support/raw_ostream.h"
  20: 
  21: using namespace llvm;
  22: 
  23: #define DEBUG_TYPE "systemz-isel"
  24: #define PASS_NAME "SystemZ DAG->DAG Pattern Instruction Selection"
  25: 
  26: namespace {
  27: // Used to build addressing modes.
  28: struct SystemZAddressingMode {
  29:   // The shape of the address.
  30:   enum AddrForm {
  31:     // base+displacement
  32:     FormBD,
  33: 
  34:     // base+displacement+index for load and store operands
  35:     FormBDXNormal,
  36: 
  37:     // base+displacement+index for load address operands
  38:     FormBDXLA,
  39: 
  40:     // base+displacement+index+ADJDYNALLOC
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `SystemZTargetMachine.h`, `SystemZISelLowering.h`, `AliasAnalysis.h`, `SelectionDAGISel.h`, `Debug.h`, `KnownBits.h` that expose the LLVM and target interfaces used in later logic. It introduces interface types such as `SystemZAddressingMode`, shaping how other backend components interact with this file.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `SystemZTargetMachine.h`, `SystemZISelLowering.h`, `AliasAnalysis.h`, `SelectionDAGISel.h`, `Debug.h`, `KnownBits.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 它引入了 `SystemZAddressingMode` 等接口类型，定义了其他后端组件与本文件交互的方式。

### Lines 41-80 / 第 41-80 行
```cpp
  41:     FormBDXDynAlloc
  42:   };
  43:   AddrForm Form;
  44: 
  45:   // The type of displacement.  The enum names here correspond directly
  46:   // to the definitions in SystemZOperand.td.  We could split them into
  47:   // flags -- single/pair, 128-bit, etc. -- but it hardly seems worth it.
  48:   enum DispRange {
  49:     Disp12Only,
  50:     Disp12Pair,
  51:     Disp20Only,
  52:     Disp20Only128,
  53:     Disp20Pair
  54:   };
  55:   DispRange DR;
  56: 
  57:   // The parts of the address.  The address is equivalent to:
  58:   //
  59:   //     Base + Disp + Index + (IncludesDynAlloc ? ADJDYNALLOC : 0)
  60:   SDValue Base;
  61:   int64_t Disp;
  62:   SDValue Index;
  63:   bool IncludesDynAlloc;
  64: 
  65:   SystemZAddressingMode(AddrForm form, DispRange dr)
  66:       : Form(form), DR(dr), Disp(0), IncludesDynAlloc(false) {}
  67: 
  68:   // True if the address can have an index register.
  69:   bool hasIndexField() { return Form != FormBD; }
  70: 
  71:   // True if the address can (and must) include ADJDYNALLOC.
  72:   bool isDynAlloc() { return Form == FormBDXDynAlloc; }
  73: 
  74:   void dump(const llvm::SelectionDAG *DAG) {
  75:     errs() << "SystemZAddressingMode " << this << '\n';
  76: 
  77:     errs() << " Base ";
  78:     if (Base.getNode())
  79:       Base.getNode()->dump(DAG);
  80:     else
```
- **EN**: The range implements or declares functions including `SystemZAddressingMode`, `isDynAlloc`, `dump`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZAddressingMode`, `isDynAlloc`, `dump` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 81-120 / 第 81-120 行
```cpp
  81:       errs() << "null\n";
  82: 
  83:     if (hasIndexField()) {
  84:       errs() << " Index ";
  85:       if (Index.getNode())
  86:         Index.getNode()->dump(DAG);
  87:       else
  88:         errs() << "null\n";
  89:     }
  90: 
  91:     errs() << " Disp " << Disp;
  92:     if (IncludesDynAlloc)
  93:       errs() << " + ADJDYNALLOC";
  94:     errs() << '\n';
  95:   }
  96: };
  97: 
  98: // Return a mask with Count low bits set.
  99: static uint64_t allOnes(unsigned int Count) {
 100:   assert(Count <= 64);
 101:   if (Count > 63)
 102:     return UINT64_MAX;
 103:   return (uint64_t(1) << Count) - 1;
 104: }
 105: 
 106: // Represents operands 2 to 5 of the ROTATE AND ... SELECTED BITS operation
 107: // given by Opcode.  The operands are: Input (R2), Start (I3), End (I4) and
 108: // Rotate (I5).  The combined operand value is effectively:
 109: //
 110: //   (or (rotl Input, Rotate), ~Mask)
 111: //
 112: // for RNSBG and:
 113: //
 114: //   (and (rotl Input, Rotate), Mask)
 115: //
 116: // otherwise.  The output value has BitSize bits, although Input may be
 117: // narrower (in which case the upper bits are don't care), or wider (in which
 118: // case the result will be truncated as part of the operation).
 119: struct RxSBGOperands {
 120:   RxSBGOperands(unsigned Op, SDValue N)
```
- **EN**: It introduces interface types such as `RxSBGOperands`, shaping how other backend components interact with this file. The range implements or declares functions including `allOnes`, `RxSBGOperands`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 它引入了 `RxSBGOperands` 等接口类型，定义了其他后端组件与本文件交互的方式。 这一段实现或声明了 `allOnes`, `RxSBGOperands` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 121-160 / 第 121-160 行
```cpp
 121:     : Opcode(Op), BitSize(N.getValueSizeInBits()),
 122:       Mask(allOnes(BitSize)), Input(N), Start(64 - BitSize), End(63),
 123:       Rotate(0) {}
 124: 
 125:   unsigned Opcode;
 126:   unsigned BitSize;
 127:   uint64_t Mask;
 128:   SDValue Input;
 129:   unsigned Start;
 130:   unsigned End;
 131:   unsigned Rotate;
 132: };
 133: 
 134: class SystemZDAGToDAGISel : public SelectionDAGISel {
 135:   const SystemZSubtarget *Subtarget;
 136: 
 137:   // Used by SystemZOperands.td to create integer constants.
 138:   inline SDValue getImm(const SDNode *Node, uint64_t Imm) const {
 139:     return CurDAG->getTargetConstant(Imm, SDLoc(Node), Node->getValueType(0));
 140:   }
 141: 
 142:   const SystemZTargetMachine &getTargetMachine() const {
 143:     return static_cast<const SystemZTargetMachine &>(TM);
 144:   }
 145: 
 146:   const SystemZInstrInfo *getInstrInfo() const {
 147:     return Subtarget->getInstrInfo();
 148:   }
 149: 
 150:   // Try to fold more of the base or index of AM into AM, where IsBase
 151:   // selects between the base and index.
 152:   bool expandAddress(SystemZAddressingMode &AM, bool IsBase) const;
 153: 
 154:   // Try to describe N in AM, returning true on success.
 155:   bool selectAddress(SDValue N, SystemZAddressingMode &AM) const;
 156: 
 157:   // Extract individual target operands from matched address AM.
 158:   void getAddressOperands(const SystemZAddressingMode &AM, EVT VT,
 159:                           SDValue &Base, SDValue &Disp) const;
 160:   void getAddressOperands(const SystemZAddressingMode &AM, EVT VT,
```
- **EN**: This block declares or refines TableGen records such as `SystemZDAGToDAGISel`. The range implements or declares functions including `Opcode`, `getImm`.
- **CN**: 该代码块声明或细化了 `SystemZDAGToDAGISel` 等 TableGen 记录。 这一段实现或声明了 `Opcode`, `getImm` 等函数。

### Lines 161-200 / 第 161-200 行
```cpp
 161:                           SDValue &Base, SDValue &Disp, SDValue &Index) const;
 162: 
 163:   // Try to match Addr as a FormBD address with displacement type DR.
 164:   // Return true on success, storing the base and displacement in
 165:   // Base and Disp respectively.
 166:   bool selectBDAddr(SystemZAddressingMode::DispRange DR, SDValue Addr,
 167:                     SDValue &Base, SDValue &Disp) const;
 168: 
 169:   // Try to match Addr as a FormBDX address with displacement type DR.
 170:   // Return true on success and if the result had no index.  Store the
 171:   // base and displacement in Base and Disp respectively.
 172:   bool selectMVIAddr(SystemZAddressingMode::DispRange DR, SDValue Addr,
 173:                      SDValue &Base, SDValue &Disp) const;
 174: 
 175:   // Try to match Addr as a FormBDX* address of form Form with
 176:   // displacement type DR.  Return true on success, storing the base,
 177:   // displacement and index in Base, Disp and Index respectively.
 178:   bool selectBDXAddr(SystemZAddressingMode::AddrForm Form,
 179:                      SystemZAddressingMode::DispRange DR, SDValue Addr,
 180:                      SDValue &Base, SDValue &Disp, SDValue &Index) const;
 181: 
 182:   // PC-relative address matching routines used by SystemZOperands.td.
 183:   bool selectPCRelAddress(SDValue Addr, SDValue &Target) const {
 184:     if (SystemZISD::isPCREL(Addr.getOpcode())) {
 185:       Target = Addr.getOperand(0);
 186:       return true;
 187:     }
 188:     return false;
 189:   }
 190: 
 191:   // BD matching routines used by SystemZOperands.td.
 192:   bool selectBDAddr12Only(SDValue Addr, SDValue &Base, SDValue &Disp) const {
 193:     return selectBDAddr(SystemZAddressingMode::Disp12Only, Addr, Base, Disp);
 194:   }
 195:   bool selectBDAddr12Pair(SDValue Addr, SDValue &Base, SDValue &Disp) const {
 196:     return selectBDAddr(SystemZAddressingMode::Disp12Pair, Addr, Base, Disp);
 197:   }
 198:   bool selectBDAddr20Only(SDValue Addr, SDValue &Base, SDValue &Disp) const {
 199:     return selectBDAddr(SystemZAddressingMode::Disp20Only, Addr, Base, Disp);
 200:   }
```
- **EN**: The range implements or declares functions including `selectPCRelAddress`, `selectBDAddr12Only`, `selectBDAddr12Pair`, `selectBDAddr20Only`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `selectPCRelAddress`, `selectBDAddr12Only`, `selectBDAddr12Pair`, `selectBDAddr20Only` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 201-240 / 第 201-240 行
```cpp
 201:   bool selectBDAddr20Pair(SDValue Addr, SDValue &Base, SDValue &Disp) const {
 202:     return selectBDAddr(SystemZAddressingMode::Disp20Pair, Addr, Base, Disp);
 203:   }
 204: 
 205:   // MVI matching routines used by SystemZOperands.td.
 206:   bool selectMVIAddr12Pair(SDValue Addr, SDValue &Base, SDValue &Disp) const {
 207:     return selectMVIAddr(SystemZAddressingMode::Disp12Pair, Addr, Base, Disp);
 208:   }
 209:   bool selectMVIAddr20Pair(SDValue Addr, SDValue &Base, SDValue &Disp) const {
 210:     return selectMVIAddr(SystemZAddressingMode::Disp20Pair, Addr, Base, Disp);
 211:   }
 212: 
 213:   // BDX matching routines used by SystemZOperands.td.
 214:   bool selectBDXAddr12Only(SDValue Addr, SDValue &Base, SDValue &Disp,
 215:                            SDValue &Index) const {
 216:     return selectBDXAddr(SystemZAddressingMode::FormBDXNormal,
 217:                          SystemZAddressingMode::Disp12Only,
 218:                          Addr, Base, Disp, Index);
 219:   }
 220:   bool selectBDXAddr12Pair(SDValue Addr, SDValue &Base, SDValue &Disp,
 221:                            SDValue &Index) const {
 222:     return selectBDXAddr(SystemZAddressingMode::FormBDXNormal,
 223:                          SystemZAddressingMode::Disp12Pair,
 224:                          Addr, Base, Disp, Index);
 225:   }
 226:   bool selectDynAlloc12Only(SDValue Addr, SDValue &Base, SDValue &Disp,
 227:                             SDValue &Index) const {
 228:     return selectBDXAddr(SystemZAddressingMode::FormBDXDynAlloc,
 229:                          SystemZAddressingMode::Disp12Only,
 230:                          Addr, Base, Disp, Index);
 231:   }
 232:   bool selectBDXAddr20Only(SDValue Addr, SDValue &Base, SDValue &Disp,
 233:                            SDValue &Index) const {
 234:     return selectBDXAddr(SystemZAddressingMode::FormBDXNormal,
 235:                          SystemZAddressingMode::Disp20Only,
 236:                          Addr, Base, Disp, Index);
 237:   }
 238:   bool selectBDXAddr20Only128(SDValue Addr, SDValue &Base, SDValue &Disp,
 239:                               SDValue &Index) const {
 240:     return selectBDXAddr(SystemZAddressingMode::FormBDXNormal,
```
- **EN**: The range implements or declares functions including `selectBDAddr20Pair`, `selectMVIAddr12Pair`, `selectMVIAddr20Pair`, `selectBDXAddr12Only`, `selectBDXAddr12Pair`.
- **CN**: 这一段实现或声明了 `selectBDAddr20Pair`, `selectMVIAddr12Pair`, `selectMVIAddr20Pair`, `selectBDXAddr12Only`, `selectBDXAddr12Pair` 等函数。

### Lines 241-280 / 第 241-280 行
```cpp
 241:                          SystemZAddressingMode::Disp20Only128,
 242:                          Addr, Base, Disp, Index);
 243:   }
 244:   bool selectBDXAddr20Pair(SDValue Addr, SDValue &Base, SDValue &Disp,
 245:                            SDValue &Index) const {
 246:     return selectBDXAddr(SystemZAddressingMode::FormBDXNormal,
 247:                          SystemZAddressingMode::Disp20Pair,
 248:                          Addr, Base, Disp, Index);
 249:   }
 250:   bool selectLAAddr12Pair(SDValue Addr, SDValue &Base, SDValue &Disp,
 251:                           SDValue &Index) const {
 252:     return selectBDXAddr(SystemZAddressingMode::FormBDXLA,
 253:                          SystemZAddressingMode::Disp12Pair,
 254:                          Addr, Base, Disp, Index);
 255:   }
 256:   bool selectLAAddr20Pair(SDValue Addr, SDValue &Base, SDValue &Disp,
 257:                           SDValue &Index) const {
 258:     return selectBDXAddr(SystemZAddressingMode::FormBDXLA,
 259:                          SystemZAddressingMode::Disp20Pair,
 260:                          Addr, Base, Disp, Index);
 261:   }
 262: 
 263:   // Try to match Addr as an address with a base, 12-bit displacement
 264:   // and index, where the index is element Elem of a vector.
 265:   // Return true on success, storing the base, displacement and vector
 266:   // in Base, Disp and Index respectively.
 267:   bool selectBDVAddr12Only(SDValue Addr, SDValue Elem, SDValue &Base,
 268:                            SDValue &Disp, SDValue &Index) const;
 269: 
 270:   // Check whether (or Op (and X InsertMask)) is effectively an insertion
 271:   // of X into bits InsertMask of some Y != Op.  Return true if so and
 272:   // set Op to that Y.
 273:   bool detectOrAndInsertion(SDValue &Op, uint64_t InsertMask) const;
 274: 
 275:   // Try to update RxSBG so that only the bits of RxSBG.Input in Mask are used.
 276:   // Return true on success.
 277:   bool refineRxSBGMask(RxSBGOperands &RxSBG, uint64_t Mask) const;
 278: 
 279:   // Try to fold some of RxSBG.Input into other fields of RxSBG.
 280:   // Return true on success.
```
- **EN**: The range implements or declares functions including `selectBDXAddr20Pair`, `selectLAAddr12Pair`, `selectLAAddr20Pair`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `selectBDXAddr20Pair`, `selectLAAddr12Pair`, `selectLAAddr20Pair` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 281-320 / 第 281-320 行
```cpp
 281:   bool expandRxSBG(RxSBGOperands &RxSBG) const;
 282: 
 283:   // Return an undefined value of type VT.
 284:   SDValue getUNDEF(const SDLoc &DL, EVT VT) const;
 285: 
 286:   // Convert N to VT, if it isn't already.
 287:   SDValue convertTo(const SDLoc &DL, EVT VT, SDValue N) const;
 288: 
 289:   // Try to implement AND or shift node N using RISBG with the zero flag set.
 290:   // Return the selected node on success, otherwise return null.
 291:   bool tryRISBGZero(SDNode *N);
 292: 
 293:   // Try to use RISBG or Opcode to implement OR or XOR node N.
 294:   // Return the selected node on success, otherwise return null.
 295:   bool tryRxSBG(SDNode *N, unsigned Opcode);
 296: 
 297:   // If Op0 is null, then Node is a constant that can be loaded using:
 298:   //
 299:   //   (Opcode UpperVal LowerVal)
 300:   //
 301:   // If Op0 is nonnull, then Node can be implemented using:
 302:   //
 303:   //   (Opcode (Opcode Op0 UpperVal) LowerVal)
 304:   void splitLargeImmediate(unsigned Opcode, SDNode *Node, SDValue Op0,
 305:                            uint64_t UpperVal, uint64_t LowerVal);
 306: 
 307:   void loadVectorConstant(const SystemZVectorConstantInfo &VCI,
 308:                           SDNode *Node);
 309: 
 310:   SDNode *loadPoolVectorConstant(APInt Val, EVT VT, SDLoc DL);
 311: 
 312:   // Try to use gather instruction Opcode to implement vector insertion N.
 313:   bool tryGather(SDNode *N, unsigned Opcode);
 314: 
 315:   // Try to use scatter instruction Opcode to implement store Store.
 316:   bool tryScatter(StoreSDNode *Store, unsigned Opcode);
 317: 
 318:   // Change a chain of {load; op; store} of the same value into a simple op
 319:   // through memory of that value, if the uses of the modified value and its
 320:   // address are suitable.
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 321-360 / 第 321-360 行
```cpp
 321:   bool tryFoldLoadStoreIntoMemOperand(SDNode *Node);
 322: 
 323:   // Return true if Load and Store are loads and stores of the same size
 324:   // and are guaranteed not to overlap.  Such operations can be implemented
 325:   // using block (SS-format) instructions.
 326:   //
 327:   // Partial overlap would lead to incorrect code, since the block operations
 328:   // are logically bytewise, even though they have a fast path for the
 329:   // non-overlapping case.  We also need to avoid full overlap (i.e. two
 330:   // addresses that might be equal at run time) because although that case
 331:   // would be handled correctly, it might be implemented by millicode.
 332:   bool canUseBlockOperation(StoreSDNode *Store, LoadSDNode *Load) const;
 333: 
 334:   // N is a (store (load Y), X) pattern.  Return true if it can use an MVC
 335:   // from Y to X.
 336:   bool storeLoadCanUseMVC(SDNode *N) const;
 337: 
 338:   // N is a (store (op (load A[0]), (load A[1])), X) pattern.  Return true
 339:   // if A[1 - I] == X and if N can use a block operation like NC from A[I]
 340:   // to X.
 341:   bool storeLoadCanUseBlockBinary(SDNode *N, unsigned I) const;
 342: 
 343:   // Return true if N (a load or a store) fullfills the alignment
 344:   // requirements for a PC-relative access.
 345:   bool storeLoadIsAligned(SDNode *N) const;
 346: 
 347:   // Return the load extension type of a load or atomic load.
 348:   ISD::LoadExtType getLoadExtType(SDNode *N) const;
 349: 
 350:   // Try to expand a boolean SELECT_CCMASK using an IPM sequence.
 351:   SDValue expandSelectBoolean(SDNode *Node);
 352: 
 353:   // Return true if the flags of N and the subtarget allows for
 354:   // reassociation, in which case a reg/reg opcode is needed as input to the
 355:   // MachineCombiner.
 356:   bool shouldSelectForReassoc(SDNode *N) const;
 357: 
 358: public:
 359:   SystemZDAGToDAGISel() = delete;
 360: 
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 361-400 / 第 361-400 行
```cpp
 361:   SystemZDAGToDAGISel(SystemZTargetMachine &TM, CodeGenOptLevel OptLevel)
 362:       : SelectionDAGISel(TM, OptLevel) {}
 363: 
 364:   bool runOnMachineFunction(MachineFunction &MF) override {
 365:     const Function &F = MF.getFunction();
 366:     if (F.getFnAttribute("fentry-call").getValueAsString() != "true") {
 367:       if (F.hasFnAttribute("mnop-mcount"))
 368:         report_fatal_error("mnop-mcount only supported with fentry-call");
 369:       if (F.hasFnAttribute("mrecord-mcount"))
 370:         report_fatal_error("mrecord-mcount only supported with fentry-call");
 371:     }
 372: 
 373:     Subtarget = &MF.getSubtarget<SystemZSubtarget>();
 374:     return SelectionDAGISel::runOnMachineFunction(MF);
 375:   }
 376: 
 377:   // Override SelectionDAGISel.
 378:   void Select(SDNode *Node) override;
 379:   bool SelectInlineAsmMemoryOperand(const SDValue &Op,
 380:                                     InlineAsm::ConstraintCode ConstraintID,
 381:                                     std::vector<SDValue> &OutOps) override;
 382:   bool IsProfitableToFold(SDValue N, SDNode *U, SDNode *Root) const override;
 383:   void PreprocessISelDAG() override;
 384: 
 385:   // Include the pieces autogenerated from the target description.
 386:   #include "SystemZGenDAGISel.inc"
 387: };
 388: 
 389: class SystemZDAGToDAGISelLegacy : public SelectionDAGISelLegacy {
 390: public:
 391:   static char ID;
 392:   explicit SystemZDAGToDAGISelLegacy(SystemZTargetMachine &TM,
 393:                                      CodeGenOptLevel OptLevel)
 394:       : SelectionDAGISelLegacy(
 395:             ID, std::make_unique<SystemZDAGToDAGISel>(TM, OptLevel)) {}
 396: };
 397: } // end anonymous namespace
 398: 
 399: char SystemZDAGToDAGISelLegacy::ID = 0;
 400: 
```
- **EN**: It imports dependencies such as `SystemZGenDAGISel.inc` that expose the LLVM and target interfaces used in later logic. This block declares or refines TableGen records such as `SystemZDAGToDAGISelLegacy`. The range implements or declares functions including `SystemZDAGToDAGISel`, `SystemZDAGToDAGISelLegacy`.
- **CN**: 它引入了 `SystemZGenDAGISel.inc` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 该代码块声明或细化了 `SystemZDAGToDAGISelLegacy` 等 TableGen 记录。 这一段实现或声明了 `SystemZDAGToDAGISel`, `SystemZDAGToDAGISelLegacy` 等函数。

### Lines 401-440 / 第 401-440 行
```cpp
 401: INITIALIZE_PASS(SystemZDAGToDAGISelLegacy, DEBUG_TYPE, PASS_NAME, false, false)
 402: 
 403: FunctionPass *llvm::createSystemZISelDag(SystemZTargetMachine &TM,
 404:                                          CodeGenOptLevel OptLevel) {
 405:   return new SystemZDAGToDAGISelLegacy(TM, OptLevel);
 406: }
 407: 
 408: // Return true if Val should be selected as a displacement for an address
 409: // with range DR.  Here we're interested in the range of both the instruction
 410: // described by DR and of any pairing instruction.
 411: static bool selectDisp(SystemZAddressingMode::DispRange DR, int64_t Val) {
 412:   switch (DR) {
 413:   case SystemZAddressingMode::Disp12Only:
 414:     return isUInt<12>(Val);
 415: 
 416:   case SystemZAddressingMode::Disp12Pair:
 417:   case SystemZAddressingMode::Disp20Only:
 418:   case SystemZAddressingMode::Disp20Pair:
 419:     return isInt<20>(Val);
 420: 
 421:   case SystemZAddressingMode::Disp20Only128:
 422:     return isInt<20>(Val) && isInt<20>(Val + 8);
 423:   }
 424:   llvm_unreachable("Unhandled displacement range");
 425: }
 426: 
 427: // Change the base or index in AM to Value, where IsBase selects
 428: // between the base and index.
 429: static void changeComponent(SystemZAddressingMode &AM, bool IsBase,
 430:                             SDValue Value) {
 431:   if (IsBase)
 432:     AM.Base = Value;
 433:   else
 434:     AM.Index = Value;
 435: }
 436: 
 437: // The base or index of AM is equivalent to Value + ADJDYNALLOC,
 438: // where IsBase selects between the base and index.  Try to fold the
 439: // ADJDYNALLOC into AM.
 440: static bool expandAdjDynAlloc(SystemZAddressingMode &AM, bool IsBase,
```
- **EN**: The range implements or declares functions including `selectDisp`, `changeComponent`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `selectDisp`, `changeComponent` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 441-480 / 第 441-480 行
```cpp
 441:                               SDValue Value) {
 442:   if (AM.isDynAlloc() && !AM.IncludesDynAlloc) {
 443:     changeComponent(AM, IsBase, Value);
 444:     AM.IncludesDynAlloc = true;
 445:     return true;
 446:   }
 447:   return false;
 448: }
 449: 
 450: // The base of AM is equivalent to Base + Index.  Try to use Index as
 451: // the index register.
 452: static bool expandIndex(SystemZAddressingMode &AM, SDValue Base,
 453:                         SDValue Index) {
 454:   if (AM.hasIndexField() && !AM.Index.getNode()) {
 455:     AM.Base = Base;
 456:     AM.Index = Index;
 457:     return true;
 458:   }
 459:   return false;
 460: }
 461: 
 462: // The base or index of AM is equivalent to Op0 + Op1, where IsBase selects
 463: // between the base and index.  Try to fold Op1 into AM's displacement.
 464: static bool expandDisp(SystemZAddressingMode &AM, bool IsBase,
 465:                        SDValue Op0, uint64_t Op1) {
 466:   // First try adjusting the displacement.
 467:   int64_t TestDisp = AM.Disp + Op1;
 468:   if (selectDisp(AM.DR, TestDisp)) {
 469:     changeComponent(AM, IsBase, Op0);
 470:     AM.Disp = TestDisp;
 471:     return true;
 472:   }
 473: 
 474:   // We could consider forcing the displacement into a register and
 475:   // using it as an index, but it would need to be carefully tuned.
 476:   return false;
 477: }
 478: 
 479: bool SystemZDAGToDAGISel::expandAddress(SystemZAddressingMode &AM,
 480:                                         bool IsBase) const {
```
- **EN**: The range implements or declares functions including `expandIndex`, `expandDisp`, `SystemZDAGToDAGISel::expandAddress`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `expandIndex`, `expandDisp`, `SystemZDAGToDAGISel::expandAddress` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 481-520 / 第 481-520 行
```cpp
 481:   SDValue N = IsBase ? AM.Base : AM.Index;
 482:   unsigned Opcode = N.getOpcode();
 483:   // Look through no-op truncations.
 484:   if (Opcode == ISD::TRUNCATE && N.getOperand(0).getValueSizeInBits() <= 64) {
 485:     N = N.getOperand(0);
 486:     Opcode = N.getOpcode();
 487:   }
 488:   if (Opcode == ISD::ADD || CurDAG->isBaseWithConstantOffset(N)) {
 489:     SDValue Op0 = N.getOperand(0);
 490:     SDValue Op1 = N.getOperand(1);
 491: 
 492:     unsigned Op0Code = Op0->getOpcode();
 493:     unsigned Op1Code = Op1->getOpcode();
 494: 
 495:     if (Op0Code == SystemZISD::ADJDYNALLOC)
 496:       return expandAdjDynAlloc(AM, IsBase, Op1);
 497:     if (Op1Code == SystemZISD::ADJDYNALLOC)
 498:       return expandAdjDynAlloc(AM, IsBase, Op0);
 499: 
 500:     if (Op0Code == ISD::Constant)
 501:       return expandDisp(AM, IsBase, Op1,
 502:                         cast<ConstantSDNode>(Op0)->getSExtValue());
 503:     if (Op1Code == ISD::Constant)
 504:       return expandDisp(AM, IsBase, Op0,
 505:                         cast<ConstantSDNode>(Op1)->getSExtValue());
 506: 
 507:     if (IsBase && expandIndex(AM, Op0, Op1))
 508:       return true;
 509:   }
 510:   if (Opcode == SystemZISD::PCREL_OFFSET) {
 511:     SDValue Full = N.getOperand(0);
 512:     SDValue Base = N.getOperand(1);
 513:     SDValue Anchor = Base.getOperand(0);
 514:     uint64_t Offset = (cast<GlobalAddressSDNode>(Full)->getOffset() -
 515:                        cast<GlobalAddressSDNode>(Anchor)->getOffset());
 516:     return expandDisp(AM, IsBase, Base, Offset);
 517:   }
 518:   return false;
 519: }
 520: 
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 521-560 / 第 521-560 行
```cpp
 521: // Return true if an instruction with displacement range DR should be
 522: // used for displacement value Val.  selectDisp(DR, Val) must already hold.
 523: static bool isValidDisp(SystemZAddressingMode::DispRange DR, int64_t Val) {
 524:   assert(selectDisp(DR, Val) && "Invalid displacement");
 525:   switch (DR) {
 526:   case SystemZAddressingMode::Disp12Only:
 527:   case SystemZAddressingMode::Disp20Only:
 528:   case SystemZAddressingMode::Disp20Only128:
 529:     return true;
 530: 
 531:   case SystemZAddressingMode::Disp12Pair:
 532:     // Use the other instruction if the displacement is too large.
 533:     return isUInt<12>(Val);
 534: 
 535:   case SystemZAddressingMode::Disp20Pair:
 536:     // Use the other instruction if the displacement is small enough.
 537:     return !isUInt<12>(Val);
 538:   }
 539:   llvm_unreachable("Unhandled displacement range");
 540: }
 541: 
 542: // Return true if Base + Disp + Index should be performed by LA(Y).
 543: static bool shouldUseLA(SDNode *Base, int64_t Disp, SDNode *Index) {
 544:   // Don't use LA(Y) for constants.
 545:   if (!Base)
 546:     return false;
 547: 
 548:   // Always use LA(Y) for frame addresses, since we know that the destination
 549:   // register is almost always (perhaps always) going to be different from
 550:   // the frame register.
 551:   if (Base->getOpcode() == ISD::FrameIndex)
 552:     return true;
 553: 
 554:   if (Disp) {
 555:     // Always use LA(Y) if there is a base, displacement and index.
 556:     if (Index)
 557:       return true;
 558: 
 559:     // Always use LA if the displacement is small enough.  It should always
 560:     // be no worse than AGHI (and better if it avoids a move).
```
- **EN**: The range implements or declares functions including `isValidDisp`, `shouldUseLA`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `isValidDisp`, `shouldUseLA` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 561-600 / 第 561-600 行
```cpp
 561:     if (isUInt<12>(Disp))
 562:       return true;
 563: 
 564:     // For similar reasons, always use LAY if the constant is too big for AGHI.
 565:     // LAY should be no worse than AGFI.
 566:     if (!isInt<16>(Disp))
 567:       return true;
 568:   } else {
 569:     // Don't use LA for plain registers.
 570:     if (!Index)
 571:       return false;
 572: 
 573:     // Don't use LA for plain addition if the index operand is only used
 574:     // once.  It should be a natural two-operand addition in that case.
 575:     if (Index->hasOneUse())
 576:       return false;
 577: 
 578:     // Prefer addition if the second operation is sign-extended, in the
 579:     // hope of using AGF.
 580:     unsigned IndexOpcode = Index->getOpcode();
 581:     if (IndexOpcode == ISD::SIGN_EXTEND ||
 582:         IndexOpcode == ISD::SIGN_EXTEND_INREG)
 583:       return false;
 584:   }
 585: 
 586:   // Don't use LA for two-operand addition if either operand is only
 587:   // used once.  The addition instructions are better in that case.
 588:   if (Base->hasOneUse())
 589:     return false;
 590: 
 591:   return true;
 592: }
 593: 
 594: // Return true if Addr is suitable for AM, updating AM if so.
 595: bool SystemZDAGToDAGISel::selectAddress(SDValue Addr,
 596:                                         SystemZAddressingMode &AM) const {
 597:   // Start out assuming that the address will need to be loaded separately,
 598:   // then try to extend it as much as we can.
 599:   AM.Base = Addr;
 600: 
```
- **EN**: The range implements or declares functions including `SystemZDAGToDAGISel::selectAddress`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZDAGToDAGISel::selectAddress` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 601-640 / 第 601-640 行
```cpp
 601:   // First try treating the address as a constant.
 602:   if (Addr.getOpcode() == ISD::Constant &&
 603:       expandDisp(AM, true, SDValue(),
 604:                  cast<ConstantSDNode>(Addr)->getSExtValue()))
 605:     ;
 606:   // Also see if it's a bare ADJDYNALLOC.
 607:   else if (Addr.getOpcode() == SystemZISD::ADJDYNALLOC &&
 608:            expandAdjDynAlloc(AM, true, SDValue()))
 609:     ;
 610:   else
 611:     // Otherwise try expanding each component.
 612:     while (expandAddress(AM, true) ||
 613:            (AM.Index.getNode() && expandAddress(AM, false)))
 614:       continue;
 615: 
 616:   // Reject cases where it isn't profitable to use LA(Y).
 617:   if (AM.Form == SystemZAddressingMode::FormBDXLA &&
 618:       !shouldUseLA(AM.Base.getNode(), AM.Disp, AM.Index.getNode()))
 619:     return false;
 620: 
 621:   // Reject cases where the other instruction in a pair should be used.
 622:   if (!isValidDisp(AM.DR, AM.Disp))
 623:     return false;
 624: 
 625:   // Make sure that ADJDYNALLOC is included where necessary.
 626:   if (AM.isDynAlloc() && !AM.IncludesDynAlloc)
 627:     return false;
 628: 
 629:   LLVM_DEBUG(AM.dump(CurDAG));
 630:   return true;
 631: }
 632: 
 633: // Insert a node into the DAG at least before Pos.  This will reposition
 634: // the node as needed, and will assign it a node ID that is <= Pos's ID.
 635: // Note that this does *not* preserve the uniqueness of node IDs!
 636: // The selection DAG must no longer depend on their uniqueness when this
 637: // function is used.
 638: static void insertDAGNode(SelectionDAG *DAG, SDNode *Pos, SDValue N) {
 639:   if (N->getNodeId() == -1 ||
 640:       (SelectionDAGISel::getUninvalidatedNodeId(N.getNode()) >
```
- **EN**: The range implements or declares functions including `insertDAGNode`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `insertDAGNode` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 641-680 / 第 641-680 行
```cpp
 641:        SelectionDAGISel::getUninvalidatedNodeId(Pos))) {
 642:     DAG->RepositionNode(Pos->getIterator(), N.getNode());
 643:     // Mark Node as invalid for pruning as after this it may be a successor to a
 644:     // selected node but otherwise be in the same position of Pos.
 645:     // Conservatively mark it with the same -abs(Id) to assure node id
 646:     // invariant is preserved.
 647:     N->setNodeId(Pos->getNodeId());
 648:     SelectionDAGISel::InvalidateNodeId(N.getNode());
 649:   }
 650: }
 651: 
 652: void SystemZDAGToDAGISel::getAddressOperands(const SystemZAddressingMode &AM,
 653:                                              EVT VT, SDValue &Base,
 654:                                              SDValue &Disp) const {
 655:   Base = AM.Base;
 656:   if (!Base.getNode())
 657:     // Register 0 means "no base".  This is mostly useful for shifts.
 658:     Base = CurDAG->getRegister(0, VT);
 659:   else if (Base.getOpcode() == ISD::FrameIndex) {
 660:     // Lower a FrameIndex to a TargetFrameIndex.
 661:     int64_t FrameIndex = cast<FrameIndexSDNode>(Base)->getIndex();
 662:     Base = CurDAG->getTargetFrameIndex(FrameIndex, VT);
 663:   } else if (Base.getValueType() != VT) {
 664:     // Truncate values from i64 to i32, for shifts.
 665:     assert(VT == MVT::i32 && Base.getValueType() == MVT::i64 &&
 666:            "Unexpected truncation");
 667:     SDLoc DL(Base);
 668:     SDValue Trunc = CurDAG->getNode(ISD::TRUNCATE, DL, VT, Base);
 669:     insertDAGNode(CurDAG, Base.getNode(), Trunc);
 670:     Base = Trunc;
 671:   }
 672: 
 673:   // Lower the displacement to a TargetConstant.
 674:   Disp = CurDAG->getSignedTargetConstant(AM.Disp, SDLoc(Base), VT);
 675: }
 676: 
 677: void SystemZDAGToDAGISel::getAddressOperands(const SystemZAddressingMode &AM,
 678:                                              EVT VT, SDValue &Base,
 679:                                              SDValue &Disp,
 680:                                              SDValue &Index) const {
```
- **EN**: The range implements or declares functions including `SelectionDAGISel::getUninvalidatedNodeId`, `SystemZDAGToDAGISel::getAddressOperands`, `SystemZDAGToDAGISel::getAddressOperands`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `SelectionDAGISel::getUninvalidatedNodeId`, `SystemZDAGToDAGISel::getAddressOperands`, `SystemZDAGToDAGISel::getAddressOperands` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 681-720 / 第 681-720 行
```cpp
 681:   getAddressOperands(AM, VT, Base, Disp);
 682: 
 683:   Index = AM.Index;
 684:   if (!Index.getNode())
 685:     // Register 0 means "no index".
 686:     Index = CurDAG->getRegister(0, VT);
 687: }
 688: 
 689: bool SystemZDAGToDAGISel::selectBDAddr(SystemZAddressingMode::DispRange DR,
 690:                                        SDValue Addr, SDValue &Base,
 691:                                        SDValue &Disp) const {
 692:   SystemZAddressingMode AM(SystemZAddressingMode::FormBD, DR);
 693:   if (!selectAddress(Addr, AM))
 694:     return false;
 695: 
 696:   getAddressOperands(AM, Addr.getValueType(), Base, Disp);
 697:   return true;
 698: }
 699: 
 700: bool SystemZDAGToDAGISel::selectMVIAddr(SystemZAddressingMode::DispRange DR,
 701:                                         SDValue Addr, SDValue &Base,
 702:                                         SDValue &Disp) const {
 703:   SystemZAddressingMode AM(SystemZAddressingMode::FormBDXNormal, DR);
 704:   if (!selectAddress(Addr, AM) || AM.Index.getNode())
 705:     return false;
 706: 
 707:   getAddressOperands(AM, Addr.getValueType(), Base, Disp);
 708:   return true;
 709: }
 710: 
 711: bool SystemZDAGToDAGISel::selectBDXAddr(SystemZAddressingMode::AddrForm Form,
 712:                                         SystemZAddressingMode::DispRange DR,
 713:                                         SDValue Addr, SDValue &Base,
 714:                                         SDValue &Disp, SDValue &Index) const {
 715:   SystemZAddressingMode AM(Form, DR);
 716:   if (!selectAddress(Addr, AM))
 717:     return false;
 718: 
 719:   getAddressOperands(AM, Addr.getValueType(), Base, Disp, Index);
 720:   return true;
```
- **EN**: The range implements or declares functions including `SystemZDAGToDAGISel::selectBDAddr`, `SystemZDAGToDAGISel::selectMVIAddr`, `SystemZDAGToDAGISel::selectBDXAddr`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZDAGToDAGISel::selectBDAddr`, `SystemZDAGToDAGISel::selectMVIAddr`, `SystemZDAGToDAGISel::selectBDXAddr` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 721-760 / 第 721-760 行
```cpp
 721: }
 722: 
 723: bool SystemZDAGToDAGISel::selectBDVAddr12Only(SDValue Addr, SDValue Elem,
 724:                                               SDValue &Base,
 725:                                               SDValue &Disp,
 726:                                               SDValue &Index) const {
 727:   SDValue Regs[2];
 728:   if (selectBDXAddr12Only(Addr, Regs[0], Disp, Regs[1]) &&
 729:       Regs[0].getNode() && Regs[1].getNode()) {
 730:     for (unsigned int I = 0; I < 2; ++I) {
 731:       Base = Regs[I];
 732:       Index = Regs[1 - I];
 733:       // We can't tell here whether the index vector has the right type
 734:       // for the access; the caller needs to do that instead.
 735:       if (Index.getOpcode() == ISD::ZERO_EXTEND)
 736:         Index = Index.getOperand(0);
 737:       if (Index.getOpcode() == ISD::EXTRACT_VECTOR_ELT &&
 738:           Index.getOperand(1) == Elem) {
 739:         Index = Index.getOperand(0);
 740:         return true;
 741:       }
 742:     }
 743:   }
 744:   return false;
 745: }
 746: 
 747: bool SystemZDAGToDAGISel::detectOrAndInsertion(SDValue &Op,
 748:                                                uint64_t InsertMask) const {
 749:   // We're only interested in cases where the insertion is into some operand
 750:   // of Op, rather than into Op itself.  The only useful case is an AND.
 751:   if (Op.getOpcode() != ISD::AND)
 752:     return false;
 753: 
 754:   // We need a constant mask.
 755:   auto *MaskNode = dyn_cast<ConstantSDNode>(Op.getOperand(1).getNode());
 756:   if (!MaskNode)
 757:     return false;
 758: 
 759:   // It's not an insertion of Op.getOperand(0) if the two masks overlap.
 760:   uint64_t AndMask = MaskNode->getZExtValue();
```
- **EN**: The range implements or declares functions including `SystemZDAGToDAGISel::selectBDVAddr12Only`, `SystemZDAGToDAGISel::detectOrAndInsertion`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZDAGToDAGISel::selectBDVAddr12Only`, `SystemZDAGToDAGISel::detectOrAndInsertion` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 761-800 / 第 761-800 行
```cpp
 761:   if (InsertMask & AndMask)
 762:     return false;
 763: 
 764:   // It's only an insertion if all bits are covered or are known to be zero.
 765:   // The inner check covers all cases but is more expensive.
 766:   uint64_t Used = allOnes(Op.getValueSizeInBits());
 767:   if (Used != (AndMask | InsertMask)) {
 768:     KnownBits Known = CurDAG->computeKnownBits(Op.getOperand(0));
 769:     if (Used != (AndMask | InsertMask | Known.Zero.getZExtValue()))
 770:       return false;
 771:   }
 772: 
 773:   Op = Op.getOperand(0);
 774:   return true;
 775: }
 776: 
 777: bool SystemZDAGToDAGISel::refineRxSBGMask(RxSBGOperands &RxSBG,
 778:                                           uint64_t Mask) const {
 779:   const SystemZInstrInfo *TII = getInstrInfo();
 780:   if (RxSBG.Rotate != 0)
 781:     Mask = (Mask << RxSBG.Rotate) | (Mask >> (64 - RxSBG.Rotate));
 782:   Mask &= RxSBG.Mask;
 783:   if (TII->isRxSBGMask(Mask, RxSBG.BitSize, RxSBG.Start, RxSBG.End)) {
 784:     RxSBG.Mask = Mask;
 785:     return true;
 786:   }
 787:   return false;
 788: }
 789: 
 790: // Return true if any bits of (RxSBG.Input & Mask) are significant.
 791: static bool maskMatters(RxSBGOperands &RxSBG, uint64_t Mask) {
 792:   // Rotate the mask in the same way as RxSBG.Input is rotated.
 793:   if (RxSBG.Rotate != 0)
 794:     Mask = ((Mask << RxSBG.Rotate) | (Mask >> (64 - RxSBG.Rotate)));
 795:   return (Mask & RxSBG.Mask) != 0;
 796: }
 797: 
 798: bool SystemZDAGToDAGISel::expandRxSBG(RxSBGOperands &RxSBG) const {
 799:   SDValue N = RxSBG.Input;
 800:   unsigned Opcode = N.getOpcode();
```
- **EN**: The range implements or declares functions including `SystemZDAGToDAGISel::refineRxSBGMask`, `maskMatters`, `SystemZDAGToDAGISel::expandRxSBG`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZDAGToDAGISel::refineRxSBGMask`, `maskMatters`, `SystemZDAGToDAGISel::expandRxSBG` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 801-840 / 第 801-840 行
```cpp
 801:   switch (Opcode) {
 802:   case ISD::TRUNCATE: {
 803:     if (RxSBG.Opcode == SystemZ::RNSBG)
 804:       return false;
 805:     if (N.getOperand(0).getValueSizeInBits() > 64)
 806:       return false;
 807:     uint64_t BitSize = N.getValueSizeInBits();
 808:     uint64_t Mask = allOnes(BitSize);
 809:     if (!refineRxSBGMask(RxSBG, Mask))
 810:       return false;
 811:     RxSBG.Input = N.getOperand(0);
 812:     return true;
 813:   }
 814:   case ISD::AND: {
 815:     if (RxSBG.Opcode == SystemZ::RNSBG)
 816:       return false;
 817: 
 818:     auto *MaskNode = dyn_cast<ConstantSDNode>(N.getOperand(1).getNode());
 819:     if (!MaskNode)
 820:       return false;
 821: 
 822:     SDValue Input = N.getOperand(0);
 823:     uint64_t Mask = MaskNode->getZExtValue();
 824:     if (!refineRxSBGMask(RxSBG, Mask)) {
 825:       // If some bits of Input are already known zeros, those bits will have
 826:       // been removed from the mask.  See if adding them back in makes the
 827:       // mask suitable.
 828:       KnownBits Known = CurDAG->computeKnownBits(Input);
 829:       Mask |= Known.Zero.getZExtValue();
 830:       if (!refineRxSBGMask(RxSBG, Mask))
 831:         return false;
 832:     }
 833:     RxSBG.Input = Input;
 834:     return true;
 835:   }
 836: 
 837:   case ISD::OR: {
 838:     if (RxSBG.Opcode != SystemZ::RNSBG)
 839:       return false;
 840: 
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 841-880 / 第 841-880 行
```cpp
 841:     auto *MaskNode = dyn_cast<ConstantSDNode>(N.getOperand(1).getNode());
 842:     if (!MaskNode)
 843:       return false;
 844: 
 845:     SDValue Input = N.getOperand(0);
 846:     uint64_t Mask = ~MaskNode->getZExtValue();
 847:     if (!refineRxSBGMask(RxSBG, Mask)) {
 848:       // If some bits of Input are already known ones, those bits will have
 849:       // been removed from the mask.  See if adding them back in makes the
 850:       // mask suitable.
 851:       KnownBits Known = CurDAG->computeKnownBits(Input);
 852:       Mask &= ~Known.One.getZExtValue();
 853:       if (!refineRxSBGMask(RxSBG, Mask))
 854:         return false;
 855:     }
 856:     RxSBG.Input = Input;
 857:     return true;
 858:   }
 859: 
 860:   case ISD::ROTL: {
 861:     // Any 64-bit rotate left can be merged into the RxSBG.
 862:     if (RxSBG.BitSize != 64 || N.getValueType() != MVT::i64)
 863:       return false;
 864:     auto *CountNode = dyn_cast<ConstantSDNode>(N.getOperand(1).getNode());
 865:     if (!CountNode)
 866:       return false;
 867: 
 868:     RxSBG.Rotate = (RxSBG.Rotate + CountNode->getZExtValue()) & 63;
 869:     RxSBG.Input = N.getOperand(0);
 870:     return true;
 871:   }
 872: 
 873:   case ISD::ANY_EXTEND:
 874:     // Bits above the extended operand are don't-care.
 875:     RxSBG.Input = N.getOperand(0);
 876:     return true;
 877: 
 878:   case ISD::ZERO_EXTEND:
 879:     if (RxSBG.Opcode != SystemZ::RNSBG) {
 880:       // Restrict the mask to the extended operand.
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 881-920 / 第 881-920 行
```cpp
 881:       unsigned InnerBitSize = N.getOperand(0).getValueSizeInBits();
 882:       if (!refineRxSBGMask(RxSBG, allOnes(InnerBitSize)))
 883:         return false;
 884: 
 885:       RxSBG.Input = N.getOperand(0);
 886:       return true;
 887:     }
 888:     [[fallthrough]];
 889: 
 890:   case ISD::SIGN_EXTEND: {
 891:     // Check that the extension bits are don't-care (i.e. are masked out
 892:     // by the final mask).
 893:     unsigned BitSize = N.getValueSizeInBits();
 894:     unsigned InnerBitSize = N.getOperand(0).getValueSizeInBits();
 895:     if (maskMatters(RxSBG, allOnes(BitSize) - allOnes(InnerBitSize))) {
 896:       // In the case where only the sign bit is active, increase Rotate with
 897:       // the extension width.
 898:       if (RxSBG.Mask == 1 && RxSBG.Rotate == 1)
 899:         RxSBG.Rotate += (BitSize - InnerBitSize);
 900:       else
 901:         return false;
 902:     }
 903: 
 904:     RxSBG.Input = N.getOperand(0);
 905:     return true;
 906:   }
 907: 
 908:   case ISD::SHL: {
 909:     auto *CountNode = dyn_cast<ConstantSDNode>(N.getOperand(1).getNode());
 910:     if (!CountNode)
 911:       return false;
 912: 
 913:     uint64_t Count = CountNode->getZExtValue();
 914:     unsigned BitSize = N.getValueSizeInBits();
 915:     if (Count < 1 || Count >= BitSize)
 916:       return false;
 917: 
 918:     if (RxSBG.Opcode == SystemZ::RNSBG) {
 919:       // Treat (shl X, count) as (rotl X, size-count) as long as the bottom
 920:       // count bits from RxSBG.Input are ignored.
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 921-960 / 第 921-960 行
```cpp
 921:       if (maskMatters(RxSBG, allOnes(Count)))
 922:         return false;
 923:     } else {
 924:       // Treat (shl X, count) as (and (rotl X, count), ~0<<count).
 925:       if (!refineRxSBGMask(RxSBG, allOnes(BitSize - Count) << Count))
 926:         return false;
 927:     }
 928: 
 929:     RxSBG.Rotate = (RxSBG.Rotate + Count) & 63;
 930:     RxSBG.Input = N.getOperand(0);
 931:     return true;
 932:   }
 933: 
 934:   case ISD::SRL:
 935:   case ISD::SRA: {
 936:     auto *CountNode = dyn_cast<ConstantSDNode>(N.getOperand(1).getNode());
 937:     if (!CountNode)
 938:       return false;
 939: 
 940:     uint64_t Count = CountNode->getZExtValue();
 941:     unsigned BitSize = N.getValueSizeInBits();
 942:     if (Count < 1 || Count >= BitSize)
 943:       return false;
 944: 
 945:     if (RxSBG.Opcode == SystemZ::RNSBG || Opcode == ISD::SRA) {
 946:       // Treat (srl|sra X, count) as (rotl X, size-count) as long as the top
 947:       // count bits from RxSBG.Input are ignored.
 948:       if (maskMatters(RxSBG, allOnes(Count) << (BitSize - Count)))
 949:         return false;
 950:     } else {
 951:       // Treat (srl X, count), mask) as (and (rotl X, size-count), ~0>>count),
 952:       // which is similar to SLL above.
 953:       if (!refineRxSBGMask(RxSBG, allOnes(BitSize - Count)))
 954:         return false;
 955:     }
 956: 
 957:     RxSBG.Rotate = (RxSBG.Rotate - Count) & 63;
 958:     RxSBG.Input = N.getOperand(0);
 959:     return true;
 960:   }
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 961-1000 / 第 961-1000 行
```cpp
 961:   default:
 962:     return false;
 963:   }
 964: }
 965: 
 966: SDValue SystemZDAGToDAGISel::getUNDEF(const SDLoc &DL, EVT VT) const {
 967:   SDNode *N = CurDAG->getMachineNode(TargetOpcode::IMPLICIT_DEF, DL, VT);
 968:   return SDValue(N, 0);
 969: }
 970: 
 971: SDValue SystemZDAGToDAGISel::convertTo(const SDLoc &DL, EVT VT,
 972:                                        SDValue N) const {
 973:   if (N.getValueType() == MVT::i32 && VT == MVT::i64)
 974:     return CurDAG->getTargetInsertSubreg(SystemZ::subreg_l32,
 975:                                          DL, VT, getUNDEF(DL, MVT::i64), N);
 976:   if (N.getValueType() == MVT::i64 && VT == MVT::i32)
 977:     return CurDAG->getTargetExtractSubreg(SystemZ::subreg_l32, DL, VT, N);
 978:   assert(N.getValueType() == VT && "Unexpected value types");
 979:   return N;
 980: }
 981: 
 982: bool SystemZDAGToDAGISel::tryRISBGZero(SDNode *N) {
 983:   SDLoc DL(N);
 984:   EVT VT = N->getValueType(0);
 985:   if (!VT.isInteger() || VT.getSizeInBits() > 64)
 986:     return false;
 987:   RxSBGOperands RISBG(SystemZ::RISBG, SDValue(N, 0));
 988:   unsigned Count = 0;
 989:   while (expandRxSBG(RISBG))
 990:     // The widening or narrowing is expected to be free.
 991:     // Counting widening or narrowing as a saved operation will result in
 992:     // preferring an R*SBG over a simple shift/logical instruction.
 993:     if (RISBG.Input.getOpcode() != ISD::ANY_EXTEND &&
 994:         RISBG.Input.getOpcode() != ISD::TRUNCATE)
 995:       Count += 1;
 996:   if (Count == 0 || isa<ConstantSDNode>(RISBG.Input))
 997:     return false;
 998: 
 999:   // Prefer to use normal shift instructions over RISBG, since they can handle
1000:   // all cases and are sometimes shorter.
```
- **EN**: The range implements or declares functions including `SystemZDAGToDAGISel::getUNDEF`, `SystemZDAGToDAGISel::convertTo`, `SystemZDAGToDAGISel::tryRISBGZero`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZDAGToDAGISel::getUNDEF`, `SystemZDAGToDAGISel::convertTo`, `SystemZDAGToDAGISel::tryRISBGZero` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 1001-1040 / 第 1001-1040 行
```cpp
1001:   if (Count == 1 && N->getOpcode() != ISD::AND)
1002:     return false;
1003: 
1004:   // Prefer LOAD LOGICAL INDEXED ADDRESS over RISBG in the case where we
1005:   // can use its displacement to pull in an addition.
1006:   if (Subtarget->hasMiscellaneousExtensions4() &&
1007:       RISBG.Rotate >= 1 && RISBG.Rotate <= 4 &&
1008:       RISBG.Mask == (((uint64_t)1 << 32) - 1) << RISBG.Rotate &&
1009:       RISBG.Input.getOpcode() == ISD::ADD)
1010:     if (auto *C = dyn_cast<ConstantSDNode>(RISBG.Input.getOperand(1)))
1011:       if (isInt<20>(C->getSExtValue()))
1012:         return false;
1013: 
1014:   // Prefer register extensions like LLC over RISBG.  Also prefer to start
1015:   // out with normal ANDs if one instruction would be enough.  We can convert
1016:   // these ANDs into an RISBG later if a three-address instruction is useful.
1017:   if (RISBG.Rotate == 0) {
1018:     bool PreferAnd = false;
1019:     // Prefer AND for any 32-bit and-immediate operation.
1020:     if (VT == MVT::i32)
1021:       PreferAnd = true;
1022:     // As well as for any 64-bit operation that can be implemented via LLC(R),
1023:     // LLH(R), LLGT(R), or one of the and-immediate instructions.
1024:     else if (RISBG.Mask == 0xff ||
1025:              RISBG.Mask == 0xffff ||
1026:              RISBG.Mask == 0x7fffffff ||
1027:              SystemZ::isImmLF(~RISBG.Mask) ||
1028:              SystemZ::isImmHF(~RISBG.Mask))
1029:      PreferAnd = true;
1030:     // And likewise for the LLZRGF instruction, which doesn't have a register
1031:     // to register version.
1032:     else if (auto *Load = dyn_cast<LoadSDNode>(RISBG.Input)) {
1033:       if (Load->getMemoryVT() == MVT::i32 &&
1034:           (Load->getExtensionType() == ISD::EXTLOAD ||
1035:            Load->getExtensionType() == ISD::ZEXTLOAD) &&
1036:           RISBG.Mask == 0xffffff00 &&
1037:           Subtarget->hasLoadAndZeroRightmostByte())
1038:       PreferAnd = true;
1039:     }
1040:     if (PreferAnd) {
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1041-1080 / 第 1041-1080 行
```cpp
1041:       // Replace the current node with an AND.  Note that the current node
1042:       // might already be that same AND, in which case it is already CSE'd
1043:       // with it, and we must not call ReplaceNode.
1044:       SDValue In = convertTo(DL, VT, RISBG.Input);
1045:       SDValue Mask = CurDAG->getConstant(RISBG.Mask, DL, VT);
1046:       SDValue New = CurDAG->getNode(ISD::AND, DL, VT, In, Mask);
1047:       if (N != New.getNode()) {
1048:         insertDAGNode(CurDAG, N, Mask);
1049:         insertDAGNode(CurDAG, N, New);
1050:         ReplaceNode(N, New.getNode());
1051:         N = New.getNode();
1052:       }
1053:       // Now, select the machine opcode to implement this operation.
1054:       if (!N->isMachineOpcode())
1055:         SelectCode(N);
1056:       return true;
1057:     }
1058:   }
1059: 
1060:   unsigned Opcode = SystemZ::RISBG;
1061:   // Prefer RISBGN if available, since it does not clobber CC.
1062:   if (Subtarget->hasMiscellaneousExtensions())
1063:     Opcode = SystemZ::RISBGN;
1064:   EVT OpcodeVT = MVT::i64;
1065:   if (VT == MVT::i32 && Subtarget->hasHighWord() &&
1066:       // We can only use the 32-bit instructions if all source bits are
1067:       // in the low 32 bits without wrapping, both after rotation (because
1068:       // of the smaller range for Start and End) and before rotation
1069:       // (because the input value is truncated).
1070:       RISBG.Start >= 32 && RISBG.End >= RISBG.Start &&
1071:       ((RISBG.Start + RISBG.Rotate) & 63) >= 32 &&
1072:       ((RISBG.End + RISBG.Rotate) & 63) >=
1073:       ((RISBG.Start + RISBG.Rotate) & 63)) {
1074:     Opcode = SystemZ::RISBMux;
1075:     OpcodeVT = MVT::i32;
1076:     RISBG.Start &= 31;
1077:     RISBG.End &= 31;
1078:   }
1079:   SDValue Ops[5] = {
1080:     getUNDEF(DL, OpcodeVT),
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1081-1120 / 第 1081-1120 行
```cpp
1081:     convertTo(DL, OpcodeVT, RISBG.Input),
1082:     CurDAG->getTargetConstant(RISBG.Start, DL, MVT::i32),
1083:     CurDAG->getTargetConstant(RISBG.End | 128, DL, MVT::i32),
1084:     CurDAG->getTargetConstant(RISBG.Rotate, DL, MVT::i32)
1085:   };
1086:   SDValue New = convertTo(
1087:       DL, VT, SDValue(CurDAG->getMachineNode(Opcode, DL, OpcodeVT, Ops), 0));
1088:   ReplaceNode(N, New.getNode());
1089:   return true;
1090: }
1091: 
1092: bool SystemZDAGToDAGISel::tryRxSBG(SDNode *N, unsigned Opcode) {
1093:   SDLoc DL(N);
1094:   EVT VT = N->getValueType(0);
1095:   if (!VT.isInteger() || VT.getSizeInBits() > 64)
1096:     return false;
1097:   // Try treating each operand of N as the second operand of the RxSBG
1098:   // and see which goes deepest.
1099:   RxSBGOperands RxSBG[] = {
1100:     RxSBGOperands(Opcode, N->getOperand(0)),
1101:     RxSBGOperands(Opcode, N->getOperand(1))
1102:   };
1103:   unsigned Count[] = { 0, 0 };
1104:   for (unsigned I = 0; I < 2; ++I)
1105:     while (RxSBG[I].Input->hasOneUse() && expandRxSBG(RxSBG[I]))
1106:       // In cases of multiple users it seems better to keep the simple
1107:       // instruction as they are one cycle faster, and it also helps in cases
1108:       // where both inputs share a common node.
1109:       // The widening or narrowing is expected to be free.  Counting widening
1110:       // or narrowing as a saved operation will result in preferring an R*SBG
1111:       // over a simple shift/logical instruction.
1112:       if (RxSBG[I].Input.getOpcode() != ISD::ANY_EXTEND &&
1113:           RxSBG[I].Input.getOpcode() != ISD::TRUNCATE)
1114:         Count[I] += 1;
1115: 
1116:   // Do nothing if neither operand is suitable.
1117:   if (Count[0] == 0 && Count[1] == 0)
1118:     return false;
1119: 
1120:   // Pick the deepest second operand.
```
- **EN**: The range implements or declares functions including `convertTo`, `SystemZDAGToDAGISel::tryRxSBG`, `RxSBGOperands`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `convertTo`, `SystemZDAGToDAGISel::tryRxSBG`, `RxSBGOperands` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 1121-1160 / 第 1121-1160 行
```cpp
1121:   unsigned I = Count[0] > Count[1] ? 0 : 1;
1122:   SDValue Op0 = N->getOperand(I ^ 1);
1123: 
1124:   // Prefer IC for character insertions from memory.
1125:   if (Opcode == SystemZ::ROSBG && (RxSBG[I].Mask & 0xff) == 0)
1126:     if (auto *Load = dyn_cast<LoadSDNode>(Op0.getNode()))
1127:       if (Load->getMemoryVT() == MVT::i8)
1128:         return false;
1129: 
1130:   // See whether we can avoid an AND in the first operand by converting
1131:   // ROSBG to RISBG.
1132:   if (Opcode == SystemZ::ROSBG && detectOrAndInsertion(Op0, RxSBG[I].Mask)) {
1133:     Opcode = SystemZ::RISBG;
1134:     // Prefer RISBGN if available, since it does not clobber CC.
1135:     if (Subtarget->hasMiscellaneousExtensions())
1136:       Opcode = SystemZ::RISBGN;
1137:   }
1138: 
1139:   SDValue Ops[5] = {
1140:     convertTo(DL, MVT::i64, Op0),
1141:     convertTo(DL, MVT::i64, RxSBG[I].Input),
1142:     CurDAG->getTargetConstant(RxSBG[I].Start, DL, MVT::i32),
1143:     CurDAG->getTargetConstant(RxSBG[I].End, DL, MVT::i32),
1144:     CurDAG->getTargetConstant(RxSBG[I].Rotate, DL, MVT::i32)
1145:   };
1146:   SDValue New = convertTo(
1147:       DL, VT, SDValue(CurDAG->getMachineNode(Opcode, DL, MVT::i64, Ops), 0));
1148:   ReplaceNode(N, New.getNode());
1149:   return true;
1150: }
1151: 
1152: void SystemZDAGToDAGISel::splitLargeImmediate(unsigned Opcode, SDNode *Node,
1153:                                               SDValue Op0, uint64_t UpperVal,
1154:                                               uint64_t LowerVal) {
1155:   EVT VT = Node->getValueType(0);
1156:   SDLoc DL(Node);
1157:   SDValue Upper = CurDAG->getConstant(UpperVal, DL, VT);
1158:   if (Op0.getNode())
1159:     Upper = CurDAG->getNode(Opcode, DL, VT, Op0, Upper);
1160: 
```
- **EN**: The range implements or declares functions including `convertTo`, `SystemZDAGToDAGISel::splitLargeImmediate`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `convertTo`, `SystemZDAGToDAGISel::splitLargeImmediate` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1161-1200 / 第 1161-1200 行
```cpp
1161:   {
1162:     // When we haven't passed in Op0, Upper will be a constant. In order to
1163:     // prevent folding back to the large immediate in `Or = getNode(...)` we run
1164:     // SelectCode first and end up with an opaque machine node. This means that
1165:     // we need to use a handle to keep track of Upper in case it gets CSE'd by
1166:     // SelectCode.
1167:     //
1168:     // Note that in the case where Op0 is passed in we could just call
1169:     // SelectCode(Upper) later, along with the SelectCode(Or), and avoid needing
1170:     // the handle at all, but it's fine to do it here.
1171:     //
1172:     // TODO: This is a pretty hacky way to do this. Can we do something that
1173:     // doesn't require a two paragraph explanation?
1174:     HandleSDNode Handle(Upper);
1175:     SelectCode(Upper.getNode());
1176:     Upper = Handle.getValue();
1177:   }
1178: 
1179:   SDValue Lower = CurDAG->getConstant(LowerVal, DL, VT);
1180:   SDValue Or = CurDAG->getNode(Opcode, DL, VT, Upper, Lower);
1181: 
1182:   ReplaceNode(Node, Or.getNode());
1183: 
1184:   SelectCode(Or.getNode());
1185: }
1186: 
1187: void SystemZDAGToDAGISel::loadVectorConstant(
1188:     const SystemZVectorConstantInfo &VCI, SDNode *Node) {
1189:   assert((VCI.Opcode == SystemZISD::BYTE_MASK ||
1190:           VCI.Opcode == SystemZISD::REPLICATE ||
1191:           VCI.Opcode == SystemZISD::ROTATE_MASK) &&
1192:          "Bad opcode!");
1193:   assert(VCI.VecVT.getSizeInBits() == 128 && "Expected a vector type");
1194:   EVT VT = Node->getValueType(0);
1195:   SDLoc DL(Node);
1196:   SmallVector<SDValue, 2> Ops;
1197:   for (unsigned OpVal : VCI.OpVals)
1198:     Ops.push_back(CurDAG->getTargetConstant(OpVal, DL, MVT::i32));
1199:   SDValue Op = CurDAG->getNode(VCI.Opcode, DL, VCI.VecVT, Ops);
1200: 
```
- **EN**: The range implements or declares functions including `SystemZDAGToDAGISel::loadVectorConstant`. Iteration is used to walk operands, records, or instruction-related collections. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `SystemZDAGToDAGISel::loadVectorConstant` 等函数。 这里使用迭代来遍历操作数、记录或与指令相关的集合。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 1201-1240 / 第 1201-1240 行
```cpp
1201:   if (VCI.VecVT == VT.getSimpleVT())
1202:     ReplaceNode(Node, Op.getNode());
1203:   else if (VT.getSizeInBits() == 128) {
1204:     SDValue BitCast = CurDAG->getNode(ISD::BITCAST, DL, VT, Op);
1205:     ReplaceNode(Node, BitCast.getNode());
1206:     SelectCode(BitCast.getNode());
1207:   } else { // half, float or double
1208:     unsigned SubRegIdx = (VT.getSizeInBits() == 16   ? SystemZ::subreg_h16
1209:                           : VT.getSizeInBits() == 32 ? SystemZ::subreg_h32
1210:                                                      : SystemZ::subreg_h64);
1211:     ReplaceNode(
1212:         Node, CurDAG->getTargetExtractSubreg(SubRegIdx, DL, VT, Op).getNode());
1213:   }
1214:   SelectCode(Op.getNode());
1215: }
1216: 
1217: SDNode *SystemZDAGToDAGISel::loadPoolVectorConstant(APInt Val, EVT VT, SDLoc DL) {
1218:   SDNode *ResNode;
1219:   assert (VT.getSizeInBits() == 128);
1220: 
1221:   SDValue CP = CurDAG->getTargetConstantPool(
1222:       ConstantInt::get(Type::getInt128Ty(*CurDAG->getContext()), Val),
1223:       TLI->getPointerTy(CurDAG->getDataLayout()));
1224: 
1225:   EVT PtrVT = CP.getValueType();
1226:   SDValue Ops[] = {
1227:     SDValue(CurDAG->getMachineNode(SystemZ::LARL, DL, PtrVT, CP), 0),
1228:     CurDAG->getTargetConstant(0, DL, PtrVT),
1229:     CurDAG->getRegister(0, PtrVT),
1230:     CurDAG->getEntryNode()
1231:   };
1232:   ResNode = CurDAG->getMachineNode(SystemZ::VL, DL, VT, MVT::Other, Ops);
1233: 
1234:   // Annotate ResNode with memory operand information so that MachineInstr
1235:   // queries work properly. This e.g. gives the register allocation the
1236:   // required information for rematerialization.
1237:   MachineFunction& MF = CurDAG->getMachineFunction();
1238:   MachineMemOperand *MemOp =
1239:       MF.getMachineMemOperand(MachinePointerInfo::getConstantPool(MF),
1240:                               MachineMemOperand::MOLoad, 16, Align(8));
```
- **EN**: The range implements or declares functions including `SDValue`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SDValue` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1241-1280 / 第 1241-1280 行
```cpp
1241: 
1242:   CurDAG->setNodeMemRefs(cast<MachineSDNode>(ResNode), {MemOp});
1243:   return ResNode;
1244: }
1245: 
1246: bool SystemZDAGToDAGISel::tryGather(SDNode *N, unsigned Opcode) {
1247:   SDValue ElemV = N->getOperand(2);
1248:   auto *ElemN = dyn_cast<ConstantSDNode>(ElemV);
1249:   if (!ElemN)
1250:     return false;
1251: 
1252:   unsigned Elem = ElemN->getZExtValue();
1253:   EVT VT = N->getValueType(0);
1254:   if (Elem >= VT.getVectorNumElements())
1255:     return false;
1256: 
1257:   auto *Load = dyn_cast<LoadSDNode>(N->getOperand(1));
1258:   if (!Load || !Load->hasNUsesOfValue(1, 0))
1259:     return false;
1260:   if (Load->getMemoryVT().getSizeInBits() !=
1261:       Load->getValueType(0).getSizeInBits())
1262:     return false;
1263: 
1264:   SDValue Base, Disp, Index;
1265:   if (!selectBDVAddr12Only(Load->getBasePtr(), ElemV, Base, Disp, Index) ||
1266:       Index.getValueType() != VT.changeVectorElementTypeToInteger())
1267:     return false;
1268: 
1269:   SDLoc DL(Load);
1270:   SDValue Ops[] = {
1271:     N->getOperand(0), Base, Disp, Index,
1272:     CurDAG->getTargetConstant(Elem, DL, MVT::i32), Load->getChain()
1273:   };
1274:   SDNode *Res = CurDAG->getMachineNode(Opcode, DL, VT, MVT::Other, Ops);
1275:   ReplaceUses(SDValue(Load, 1), SDValue(Res, 1));
1276:   ReplaceNode(N, Res);
1277:   return true;
1278: }
1279: 
1280: bool SystemZDAGToDAGISel::tryScatter(StoreSDNode *Store, unsigned Opcode) {
```
- **EN**: The range implements or declares functions including `SystemZDAGToDAGISel::tryGather`, `SystemZDAGToDAGISel::tryScatter`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZDAGToDAGISel::tryGather`, `SystemZDAGToDAGISel::tryScatter` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1281-1320 / 第 1281-1320 行
```cpp
1281:   SDValue Value = Store->getValue();
1282:   if (Value.getOpcode() != ISD::EXTRACT_VECTOR_ELT)
1283:     return false;
1284:   if (Store->getMemoryVT().getSizeInBits() != Value.getValueSizeInBits())
1285:     return false;
1286: 
1287:   SDValue ElemV = Value.getOperand(1);
1288:   auto *ElemN = dyn_cast<ConstantSDNode>(ElemV);
1289:   if (!ElemN)
1290:     return false;
1291: 
1292:   SDValue Vec = Value.getOperand(0);
1293:   EVT VT = Vec.getValueType();
1294:   unsigned Elem = ElemN->getZExtValue();
1295:   if (Elem >= VT.getVectorNumElements())
1296:     return false;
1297: 
1298:   SDValue Base, Disp, Index;
1299:   if (!selectBDVAddr12Only(Store->getBasePtr(), ElemV, Base, Disp, Index) ||
1300:       Index.getValueType() != VT.changeVectorElementTypeToInteger())
1301:     return false;
1302: 
1303:   SDLoc DL(Store);
1304:   SDValue Ops[] = {
1305:     Vec, Base, Disp, Index, CurDAG->getTargetConstant(Elem, DL, MVT::i32),
1306:     Store->getChain()
1307:   };
1308:   ReplaceNode(Store, CurDAG->getMachineNode(Opcode, DL, MVT::Other, Ops));
1309:   return true;
1310: }
1311: 
1312: // Check whether or not the chain ending in StoreNode is suitable for doing
1313: // the {load; op; store} to modify transformation.
1314: static bool isFusableLoadOpStorePattern(StoreSDNode *StoreNode,
1315:                                         SDValue StoredVal, SelectionDAG *CurDAG,
1316:                                         LoadSDNode *&LoadNode,
1317:                                         SDValue &InputChain) {
1318:   // Is the stored value result 0 of the operation?
1319:   if (StoredVal.getResNo() != 0)
1320:     return false;
```
- **EN**: The range implements or declares functions including `isFusableLoadOpStorePattern`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `isFusableLoadOpStorePattern` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1321-1360 / 第 1321-1360 行
```cpp
1321: 
1322:   // Are there other uses of the loaded value than the operation?
1323:   if (!StoredVal.getNode()->hasNUsesOfValue(1, 0))
1324:     return false;
1325: 
1326:   // Is the store non-extending and non-indexed?
1327:   if (!ISD::isNormalStore(StoreNode) || StoreNode->isNonTemporal())
1328:     return false;
1329: 
1330:   SDValue Load = StoredVal->getOperand(0);
1331:   // Is the stored value a non-extending and non-indexed load?
1332:   if (!ISD::isNormalLoad(Load.getNode()))
1333:     return false;
1334: 
1335:   // Return LoadNode by reference.
1336:   LoadNode = cast<LoadSDNode>(Load);
1337: 
1338:   // Is store the only read of the loaded value?
1339:   if (!Load.hasOneUse())
1340:     return false;
1341: 
1342:   // Is the address of the store the same as the load?
1343:   if (LoadNode->getBasePtr() != StoreNode->getBasePtr() ||
1344:       LoadNode->getOffset() != StoreNode->getOffset())
1345:     return false;
1346: 
1347:   // Check if the chain is produced by the load or is a TokenFactor with
1348:   // the load output chain as an operand. Return InputChain by reference.
1349:   SDValue Chain = StoreNode->getChain();
1350: 
1351:   bool ChainCheck = false;
1352:   if (Chain == Load.getValue(1)) {
1353:     ChainCheck = true;
1354:     InputChain = LoadNode->getChain();
1355:   } else if (Chain.getOpcode() == ISD::TokenFactor) {
1356:     SmallVector<SDValue, 4> ChainOps;
1357:     SmallVector<const SDNode *, 4> LoopWorklist;
1358:     SmallPtrSet<const SDNode *, 16> Visited;
1359:     const unsigned int Max = 1024;
1360:     for (unsigned i = 0, e = Chain.getNumOperands(); i != e; ++i) {
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 1361-1400 / 第 1361-1400 行
```cpp
1361:       SDValue Op = Chain.getOperand(i);
1362:       if (Op == Load.getValue(1)) {
1363:         ChainCheck = true;
1364:         // Drop Load, but keep its chain. No cycle check necessary.
1365:         ChainOps.push_back(Load.getOperand(0));
1366:         continue;
1367:       }
1368:       LoopWorklist.push_back(Op.getNode());
1369:       ChainOps.push_back(Op);
1370:     }
1371: 
1372:     if (ChainCheck) {
1373:       // Add the other operand of StoredVal to worklist.
1374:       for (SDValue Op : StoredVal->ops())
1375:         if (Op.getNode() != LoadNode)
1376:           LoopWorklist.push_back(Op.getNode());
1377: 
1378:       // Check if Load is reachable from any of the nodes in the worklist.
1379:       if (SDNode::hasPredecessorHelper(Load.getNode(), Visited, LoopWorklist, Max,
1380:                                        true))
1381:         return false;
1382: 
1383:       // Make a new TokenFactor with all the other input chains except
1384:       // for the load.
1385:       InputChain = CurDAG->getNode(ISD::TokenFactor, SDLoc(Chain),
1386:                                    MVT::Other, ChainOps);
1387:     }
1388:   }
1389:   if (!ChainCheck)
1390:     return false;
1391: 
1392:   return true;
1393: }
1394: 
1395: // Change a chain of {load; op; store} of the same value into a simple op
1396: // through memory of that value, if the uses of the modified value and its
1397: // address are suitable.
1398: //
1399: // The tablegen pattern memory operand pattern is currently not able to match
1400: // the case where the CC on the original operation are used.
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 1401-1440 / 第 1401-1440 行
```cpp
1401: //
1402: // See the equivalent routine in X86ISelDAGToDAG for further comments.
1403: bool SystemZDAGToDAGISel::tryFoldLoadStoreIntoMemOperand(SDNode *Node) {
1404:   StoreSDNode *StoreNode = cast<StoreSDNode>(Node);
1405:   SDValue StoredVal = StoreNode->getOperand(1);
1406:   unsigned Opc = StoredVal->getOpcode();
1407:   SDLoc DL(StoreNode);
1408: 
1409:   // Before we try to select anything, make sure this is memory operand size
1410:   // and opcode we can handle. Note that this must match the code below that
1411:   // actually lowers the opcodes.
1412:   EVT MemVT = StoreNode->getMemoryVT();
1413:   unsigned NewOpc = 0;
1414:   bool NegateOperand = false;
1415:   switch (Opc) {
1416:   default:
1417:     return false;
1418:   case SystemZISD::SSUBO:
1419:     NegateOperand = true;
1420:     [[fallthrough]];
1421:   case SystemZISD::SADDO:
1422:     if (MemVT == MVT::i32)
1423:       NewOpc = SystemZ::ASI;
1424:     else if (MemVT == MVT::i64)
1425:       NewOpc = SystemZ::AGSI;
1426:     else
1427:       return false;
1428:     break;
1429:   case SystemZISD::USUBO:
1430:     NegateOperand = true;
1431:     [[fallthrough]];
1432:   case SystemZISD::UADDO:
1433:     if (MemVT == MVT::i32)
1434:       NewOpc = SystemZ::ALSI;
1435:     else if (MemVT == MVT::i64)
1436:       NewOpc = SystemZ::ALGSI;
1437:     else
1438:       return false;
1439:     break;
1440:   }
```
- **EN**: The range implements or declares functions including `SystemZDAGToDAGISel::tryFoldLoadStoreIntoMemOperand`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZDAGToDAGISel::tryFoldLoadStoreIntoMemOperand` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1441-1480 / 第 1441-1480 行
```cpp
1441: 
1442:   LoadSDNode *LoadNode = nullptr;
1443:   SDValue InputChain;
1444:   if (!isFusableLoadOpStorePattern(StoreNode, StoredVal, CurDAG, LoadNode,
1445:                                    InputChain))
1446:     return false;
1447: 
1448:   SDValue Operand = StoredVal.getOperand(1);
1449:   auto *OperandC = dyn_cast<ConstantSDNode>(Operand);
1450:   if (!OperandC)
1451:     return false;
1452:   auto OperandV = OperandC->getAPIntValue();
1453:   if (NegateOperand)
1454:     OperandV = -OperandV;
1455:   if (OperandV.getSignificantBits() > 8)
1456:     return false;
1457:   Operand = CurDAG->getTargetConstant(OperandV, DL, MemVT);
1458: 
1459:   SDValue Base, Disp;
1460:   if (!selectBDAddr20Only(StoreNode->getBasePtr(), Base, Disp))
1461:     return false;
1462: 
1463:   SDValue Ops[] = { Base, Disp, Operand, InputChain };
1464:   MachineSDNode *Result =
1465:     CurDAG->getMachineNode(NewOpc, DL, MVT::i32, MVT::Other, Ops);
1466:   CurDAG->setNodeMemRefs(
1467:       Result, {StoreNode->getMemOperand(), LoadNode->getMemOperand()});
1468: 
1469:   ReplaceUses(SDValue(StoreNode, 0), SDValue(Result, 1));
1470:   ReplaceUses(SDValue(StoredVal.getNode(), 1), SDValue(Result, 0));
1471:   CurDAG->RemoveDeadNode(Node);
1472:   return true;
1473: }
1474: 
1475: bool SystemZDAGToDAGISel::canUseBlockOperation(StoreSDNode *Store,
1476:                                                LoadSDNode *Load) const {
1477:   // Check that the two memory operands have the same size.
1478:   if (Load->getMemoryVT() != Store->getMemoryVT())
1479:     return false;
1480: 
```
- **EN**: The range implements or declares functions including `SystemZDAGToDAGISel::canUseBlockOperation`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZDAGToDAGISel::canUseBlockOperation` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1481-1520 / 第 1481-1520 行
```cpp
1481:   // Volatility stops an access from being decomposed.
1482:   if (Load->isVolatile() || Store->isVolatile())
1483:     return false;
1484: 
1485:   // There's no chance of overlap if the load is invariant.
1486:   if (Load->isInvariant() && Load->isDereferenceable())
1487:     return true;
1488: 
1489:   // Otherwise we need to check whether there's an alias.
1490:   const Value *V1 = Load->getMemOperand()->getValue();
1491:   const Value *V2 = Store->getMemOperand()->getValue();
1492:   if (!V1 || !V2)
1493:     return false;
1494: 
1495:   // Reject equality.
1496:   uint64_t Size = Load->getMemoryVT().getStoreSize();
1497:   int64_t End1 = Load->getSrcValueOffset() + Size;
1498:   int64_t End2 = Store->getSrcValueOffset() + Size;
1499:   if (V1 == V2 && End1 == End2)
1500:     return false;
1501: 
1502:   return BatchAA->isNoAlias(MemoryLocation(V1, End1, Load->getAAInfo()),
1503:                             MemoryLocation(V2, End2, Store->getAAInfo()));
1504: }
1505: 
1506: bool SystemZDAGToDAGISel::storeLoadCanUseMVC(SDNode *N) const {
1507:   auto *Store = cast<StoreSDNode>(N);
1508:   auto *Load = cast<LoadSDNode>(Store->getValue());
1509: 
1510:   // Prefer not to use MVC if either address can use ... RELATIVE LONG
1511:   // instructions.
1512:   uint64_t Size = Load->getMemoryVT().getStoreSize();
1513:   if (Size > 1 && Size <= 8) {
1514:     // Prefer LHRL, LRL and LGRL.
1515:     if (SystemZISD::isPCREL(Load->getBasePtr().getOpcode()))
1516:       return false;
1517:     // Prefer STHRL, STRL and STGRL.
1518:     if (SystemZISD::isPCREL(Store->getBasePtr().getOpcode()))
1519:       return false;
1520:   }
```
- **EN**: The range implements or declares functions including `SystemZDAGToDAGISel::storeLoadCanUseMVC`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZDAGToDAGISel::storeLoadCanUseMVC` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1521-1560 / 第 1521-1560 行
```cpp
1521: 
1522:   return canUseBlockOperation(Store, Load);
1523: }
1524: 
1525: bool SystemZDAGToDAGISel::storeLoadCanUseBlockBinary(SDNode *N,
1526:                                                      unsigned I) const {
1527:   auto *StoreA = cast<StoreSDNode>(N);
1528:   auto *LoadA = cast<LoadSDNode>(StoreA->getValue().getOperand(1 - I));
1529:   auto *LoadB = cast<LoadSDNode>(StoreA->getValue().getOperand(I));
1530:   return !LoadA->isVolatile() && LoadA->getMemoryVT() == LoadB->getMemoryVT() &&
1531:          canUseBlockOperation(StoreA, LoadB);
1532: }
1533: 
1534: bool SystemZDAGToDAGISel::storeLoadIsAligned(SDNode *N) const {
1535: 
1536:   auto *MemAccess = cast<MemSDNode>(N);
1537:   auto *LdSt = dyn_cast<LSBaseSDNode>(MemAccess);
1538:   TypeSize StoreSize = MemAccess->getMemoryVT().getStoreSize();
1539:   SDValue BasePtr = MemAccess->getBasePtr();
1540:   MachineMemOperand *MMO = MemAccess->getMemOperand();
1541:   assert(MMO && "Expected a memory operand.");
1542: 
1543:   // The memory access must have a proper alignment and no index register.
1544:   // Only load and store nodes have the offset operand (atomic loads do not).
1545:   if (MemAccess->getAlign().value() < StoreSize ||
1546:       (LdSt && !LdSt->getOffset().isUndef()))
1547:     return false;
1548: 
1549:   // The MMO must not have an unaligned offset.
1550:   if (MMO->getOffset() % StoreSize != 0)
1551:     return false;
1552: 
1553:   // An access to GOT or the Constant Pool is aligned.
1554:   if (const PseudoSourceValue *PSV = MMO->getPseudoValue())
1555:     if ((PSV->isGOT() || PSV->isConstantPool()))
1556:       return true;
1557: 
1558:   // Check the alignment of a Global Address.
1559:   if (BasePtr.getNumOperands())
1560:     if (GlobalAddressSDNode *GA =
```
- **EN**: The range implements or declares functions including `SystemZDAGToDAGISel::storeLoadCanUseBlockBinary`, `SystemZDAGToDAGISel::storeLoadIsAligned`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `SystemZDAGToDAGISel::storeLoadCanUseBlockBinary`, `SystemZDAGToDAGISel::storeLoadIsAligned` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 1561-1600 / 第 1561-1600 行
```cpp
1561:         dyn_cast<GlobalAddressSDNode>(BasePtr.getOperand(0))) {
1562:       // The immediate offset must be aligned.
1563:       if (GA->getOffset() % StoreSize != 0)
1564:         return false;
1565: 
1566:       // The alignment of the symbol itself must be at least the store size.
1567:       const GlobalValue *GV = GA->getGlobal();
1568:       const DataLayout &DL = GV->getDataLayout();
1569:       if (GV->getPointerAlignment(DL).value() < StoreSize)
1570:         return false;
1571:     }
1572: 
1573:   return true;
1574: }
1575: 
1576: ISD::LoadExtType SystemZDAGToDAGISel::getLoadExtType(SDNode *N) const {
1577:   ISD::LoadExtType ETy;
1578:   if (auto *L = dyn_cast<LoadSDNode>(N))
1579:     ETy = L->getExtensionType();
1580:   else if (auto *AL = dyn_cast<AtomicSDNode>(N))
1581:     ETy = AL->getExtensionType();
1582:   else
1583:     llvm_unreachable("Unkown load node type.");
1584:   return ETy;
1585: }
1586: 
1587: void SystemZDAGToDAGISel::Select(SDNode *Node) {
1588:   // If we have a custom node, we already have selected!
1589:   if (Node->isMachineOpcode()) {
1590:     LLVM_DEBUG(errs() << "== "; Node->dump(CurDAG); errs() << "\n");
1591:     Node->setNodeId(-1);
1592:     return;
1593:   }
1594: 
1595:   unsigned Opcode = Node->getOpcode();
1596:   switch (Opcode) {
1597:   case ISD::OR:
1598:     if (Node->getOperand(1).getOpcode() != ISD::Constant)
1599:       if (tryRxSBG(Node, SystemZ::ROSBG))
1600:         return;
```
- **EN**: The range implements or declares functions including `SystemZDAGToDAGISel::getLoadExtType`, `SystemZDAGToDAGISel::Select`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZDAGToDAGISel::getLoadExtType`, `SystemZDAGToDAGISel::Select` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1601-1640 / 第 1601-1640 行
```cpp
1601:     goto or_xor;
1602: 
1603:   case ISD::XOR:
1604:     if (Node->getOperand(1).getOpcode() != ISD::Constant)
1605:       if (tryRxSBG(Node, SystemZ::RXSBG))
1606:         return;
1607:     // Fall through.
1608:   or_xor:
1609:     // If this is a 64-bit operation in which both 32-bit halves are nonzero,
1610:     // split the operation into two.  If both operands here happen to be
1611:     // constant, leave this to common code to optimize.
1612:     if (Node->getValueType(0) == MVT::i64 &&
1613:         Node->getOperand(0).getOpcode() != ISD::Constant)
1614:       if (auto *Op1 = dyn_cast<ConstantSDNode>(Node->getOperand(1))) {
1615:         uint64_t Val = Op1->getZExtValue();
1616:         // Don't split the operation if we can match one of the combined
1617:         // logical operations provided by miscellaneous-extensions-3.
1618:         if (Subtarget->hasMiscellaneousExtensions3()) {
1619:           unsigned ChildOpcode = Node->getOperand(0).getOpcode();
1620:           // Check whether this expression matches NAND/NOR/NXOR.
1621:           if (Val == (uint64_t)-1 && Opcode == ISD::XOR)
1622:             if (ChildOpcode == ISD::AND || ChildOpcode == ISD::OR ||
1623:                 ChildOpcode == ISD::XOR)
1624:               break;
1625:           // Check whether this expression matches OR-with-complement
1626:           // (or matches an alternate pattern for NXOR).
1627:           if (ChildOpcode == ISD::XOR) {
1628:             auto Op0 = Node->getOperand(0);
1629:             if (auto *Op0Op1 = dyn_cast<ConstantSDNode>(Op0->getOperand(1)))
1630:               if (Op0Op1->getZExtValue() == (uint64_t)-1)
1631:                 break;
1632:           }
1633:         }
1634:         // Don't split an XOR with -1 as LCGR/AGHI is more compact.
1635:         if (Opcode == ISD::XOR && Op1->isAllOnes())
1636:           break;
1637:         if (!SystemZ::isImmLF(Val) && !SystemZ::isImmHF(Val)) {
1638:           splitLargeImmediate(Opcode, Node, Node->getOperand(0),
1639:                               Val - uint32_t(Val), uint32_t(Val));
1640:           return;
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1641-1680 / 第 1641-1680 行
```cpp
1641:         }
1642:       }
1643:     break;
1644: 
1645:   case ISD::AND:
1646:     if (Node->getOperand(1).getOpcode() != ISD::Constant)
1647:       if (tryRxSBG(Node, SystemZ::RNSBG))
1648:         return;
1649:     [[fallthrough]];
1650:   case ISD::ROTL:
1651:   case ISD::SHL:
1652:   case ISD::SRL:
1653:   case ISD::ZERO_EXTEND:
1654:     if (tryRISBGZero(Node))
1655:       return;
1656:     break;
1657: 
1658:   case ISD::BSWAP:
1659:     if (Node->getValueType(0) == MVT::i128) {
1660:       SDLoc DL(Node);
1661:       SDValue Src = Node->getOperand(0);
1662:       Src = CurDAG->getNode(ISD::BITCAST, DL, MVT::v16i8, Src);
1663: 
1664:       uint64_t Bytes[2] = { 0x0706050403020100ULL, 0x0f0e0d0c0b0a0908ULL };
1665:       SDNode *Mask = loadPoolVectorConstant(APInt(128, Bytes), MVT::v16i8, DL);
1666:       SDValue Ops[] = { Src, Src, SDValue(Mask, 0) };
1667:       SDValue Res = SDValue(CurDAG->getMachineNode(SystemZ::VPERM, DL,
1668:                                                    MVT::v16i8, Ops), 0);
1669: 
1670:       Res = CurDAG->getNode(ISD::BITCAST, DL, MVT::i128, Res);
1671:       SDNode *ResNode = Res.getNode();
1672:       ReplaceNode(Node, ResNode);
1673:       SelectCode(Src.getNode());
1674:       SelectCode(ResNode);
1675:       return;
1676:     }
1677:     break;
1678: 
1679:   case ISD::Constant:
1680:     // If this is a 64-bit constant that is out of the range of LLILF,
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1681-1720 / 第 1681-1720 行
```cpp
1681:     // LLIHF and LGFI, split it into two 32-bit pieces.
1682:     if (Node->getValueType(0) == MVT::i64) {
1683:       uint64_t Val = Node->getAsZExtVal();
1684:       if (!SystemZ::isImmLF(Val) && !SystemZ::isImmHF(Val) && !isInt<32>(Val)) {
1685:         splitLargeImmediate(ISD::OR, Node, SDValue(), Val - uint32_t(Val),
1686:                             uint32_t(Val));
1687:         return;
1688:       }
1689:     }
1690:     if (Node->getValueType(0) == MVT::i128) {
1691:       const APInt &Val = Node->getAsAPIntVal();
1692:       SystemZVectorConstantInfo VCI(Val);
1693:       if (VCI.isVectorConstantLegal(*Subtarget)) {
1694:         loadVectorConstant(VCI, Node);
1695:         return;
1696:       }
1697:       // If we can't materialize the constant we need to use a literal pool.
1698:       SDNode *ResNode = loadPoolVectorConstant(Val, MVT::i128, SDLoc(Node));
1699:       ReplaceNode(Node, ResNode);
1700:       return;
1701:     }
1702:     break;
1703: 
1704:   case SystemZISD::SELECT_CCMASK: {
1705:     SDValue Op0 = Node->getOperand(0);
1706:     SDValue Op1 = Node->getOperand(1);
1707:     // Prefer to put any load first, so that it can be matched as a
1708:     // conditional load.  Likewise for constants in range for LOCHI.
1709:     if ((Op1.getOpcode() == ISD::LOAD && Op0.getOpcode() != ISD::LOAD) ||
1710:         (Subtarget->hasLoadStoreOnCond2() &&
1711:          Node->getValueType(0).isInteger() &&
1712:          Node->getValueType(0).getSizeInBits() <= 64 &&
1713:          Op1.getOpcode() == ISD::Constant &&
1714:          isInt<16>(cast<ConstantSDNode>(Op1)->getSExtValue()) &&
1715:          !(Op0.getOpcode() == ISD::Constant &&
1716:            isInt<16>(cast<ConstantSDNode>(Op0)->getSExtValue())))) {
1717:       SDValue CCValid = Node->getOperand(2);
1718:       SDValue CCMask = Node->getOperand(3);
1719:       uint64_t ConstCCValid = CCValid.getNode()->getAsZExtVal();
1720:       uint64_t ConstCCMask = CCMask.getNode()->getAsZExtVal();
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1721-1760 / 第 1721-1760 行
```cpp
1721:       // Invert the condition.
1722:       CCMask = CurDAG->getTargetConstant(ConstCCValid ^ ConstCCMask,
1723:                                          SDLoc(Node), CCMask.getValueType());
1724:       SDValue Op4 = Node->getOperand(4);
1725:       SDNode *UpdatedNode =
1726:         CurDAG->UpdateNodeOperands(Node, Op1, Op0, CCValid, CCMask, Op4);
1727:       if (UpdatedNode != Node) {
1728:         // In case this node already exists then replace Node with it.
1729:         ReplaceNode(Node, UpdatedNode);
1730:         Node = UpdatedNode;
1731:       }
1732:     }
1733:     break;
1734:   }
1735: 
1736:   case ISD::INSERT_VECTOR_ELT: {
1737:     EVT VT = Node->getValueType(0);
1738:     unsigned ElemBitSize = VT.getScalarSizeInBits();
1739:     if (ElemBitSize == 32) {
1740:       if (tryGather(Node, SystemZ::VGEF))
1741:         return;
1742:     } else if (ElemBitSize == 64) {
1743:       if (tryGather(Node, SystemZ::VGEG))
1744:         return;
1745:     }
1746:     break;
1747:   }
1748: 
1749:   case ISD::BUILD_VECTOR: {
1750:     auto *BVN = cast<BuildVectorSDNode>(Node);
1751:     SystemZVectorConstantInfo VCI(BVN);
1752:     if (VCI.isVectorConstantLegal(*Subtarget)) {
1753:       loadVectorConstant(VCI, Node);
1754:       return;
1755:     }
1756:     break;
1757:   }
1758: 
1759:   case ISD::ConstantFP: {
1760:     APFloat Imm = cast<ConstantFPSDNode>(Node)->getValueAPF();
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1761-1800 / 第 1761-1800 行
```cpp
1761:     if (Imm.isZero() || Imm.isNegZero())
1762:       break;
1763:     SystemZVectorConstantInfo VCI(Imm);
1764:     bool Success = VCI.isVectorConstantLegal(*Subtarget); (void)Success;
1765:     assert(Success && "Expected legal FP immediate");
1766:     loadVectorConstant(VCI, Node);
1767:     return;
1768:   }
1769: 
1770:   case ISD::STORE: {
1771:     if (tryFoldLoadStoreIntoMemOperand(Node))
1772:       return;
1773:     auto *Store = cast<StoreSDNode>(Node);
1774:     unsigned ElemBitSize = Store->getValue().getValueSizeInBits();
1775:     if (ElemBitSize == 32) {
1776:       if (tryScatter(Store, SystemZ::VSCEF))
1777:         return;
1778:     } else if (ElemBitSize == 64) {
1779:       if (tryScatter(Store, SystemZ::VSCEG))
1780:         return;
1781:     }
1782:     break;
1783:   }
1784: 
1785:   case ISD::ATOMIC_STORE: {
1786:     auto *AtomOp = cast<AtomicSDNode>(Node);
1787:     // Replace the atomic_store with a regular store and select it. This is
1788:     // ok since we know all store instructions <= 8 bytes are atomic, and the
1789:     // 16 byte case is already handled during lowering.
1790:     StoreSDNode *St = cast<StoreSDNode>(CurDAG->getTruncStore(
1791:          AtomOp->getChain(), SDLoc(AtomOp), AtomOp->getVal(),
1792:          AtomOp->getBasePtr(), AtomOp->getMemoryVT(), AtomOp->getMemOperand()));
1793:     assert(St->getMemOperand()->isAtomic() && "Broken MMO.");
1794:     SDNode *Chain = St;
1795:     // We have to enforce sequential consistency by performing a
1796:     // serialization operation after the store.
1797:     if (AtomOp->getSuccessOrdering() == AtomicOrdering::SequentiallyConsistent)
1798:       Chain = CurDAG->getMachineNode(SystemZ::Serialize, SDLoc(AtomOp),
1799:                                      MVT::Other, SDValue(Chain, 0));
1800:     ReplaceNode(Node, Chain);
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 1801-1840 / 第 1801-1840 行
```cpp
1801:     SelectCode(St);
1802:     return;
1803:   }
1804:   }
1805: 
1806:   SelectCode(Node);
1807: }
1808: 
1809: bool SystemZDAGToDAGISel::SelectInlineAsmMemoryOperand(
1810:     const SDValue &Op, InlineAsm::ConstraintCode ConstraintID,
1811:     std::vector<SDValue> &OutOps) {
1812:   SystemZAddressingMode::AddrForm Form;
1813:   SystemZAddressingMode::DispRange DispRange;
1814:   SDValue Base, Disp, Index;
1815: 
1816:   switch(ConstraintID) {
1817:   default:
1818:     llvm_unreachable("Unexpected asm memory constraint");
1819:   case InlineAsm::ConstraintCode::i:
1820:   case InlineAsm::ConstraintCode::Q:
1821:   case InlineAsm::ConstraintCode::ZQ:
1822:     // Accept an address with a short displacement, but no index.
1823:     Form = SystemZAddressingMode::FormBD;
1824:     DispRange = SystemZAddressingMode::Disp12Only;
1825:     break;
1826:   case InlineAsm::ConstraintCode::R:
1827:   case InlineAsm::ConstraintCode::ZR:
1828:     // Accept an address with a short displacement and an index.
1829:     Form = SystemZAddressingMode::FormBDXNormal;
1830:     DispRange = SystemZAddressingMode::Disp12Only;
1831:     break;
1832:   case InlineAsm::ConstraintCode::S:
1833:   case InlineAsm::ConstraintCode::ZS:
1834:     // Accept an address with a long displacement, but no index.
1835:     Form = SystemZAddressingMode::FormBD;
1836:     DispRange = SystemZAddressingMode::Disp20Only;
1837:     break;
1838:   case InlineAsm::ConstraintCode::T:
1839:   case InlineAsm::ConstraintCode::m:
1840:   case InlineAsm::ConstraintCode::o:
```
- **EN**: The range implements or declares functions including `SystemZDAGToDAGISel::SelectInlineAsmMemoryOperand`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `SystemZDAGToDAGISel::SelectInlineAsmMemoryOperand` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 1841-1880 / 第 1841-1880 行
```cpp
1841:   case InlineAsm::ConstraintCode::p:
1842:   case InlineAsm::ConstraintCode::ZT:
1843:     // Accept an address with a long displacement and an index.
1844:     // m works the same as T, as this is the most general case.
1845:     // We don't really have any special handling of "offsettable"
1846:     // memory addresses, so just treat o the same as m.
1847:     Form = SystemZAddressingMode::FormBDXNormal;
1848:     DispRange = SystemZAddressingMode::Disp20Only;
1849:     break;
1850:   }
1851: 
1852:   if (selectBDXAddr(Form, DispRange, Op, Base, Disp, Index)) {
1853:     const TargetRegisterClass *TRC =
1854:         Subtarget->getRegisterInfo()->getPointerRegClass();
1855:     SDLoc DL(Base);
1856:     SDValue RC = CurDAG->getTargetConstant(TRC->getID(), DL, MVT::i32);
1857: 
1858:     // Make sure that the base address doesn't go into %r0.
1859:     // If it's a TargetFrameIndex or a fixed register, we shouldn't do anything.
1860:     if (Base.getOpcode() != ISD::TargetFrameIndex &&
1861:         Base.getOpcode() != ISD::Register) {
1862:       Base =
1863:         SDValue(CurDAG->getMachineNode(TargetOpcode::COPY_TO_REGCLASS,
1864:                                        DL, Base.getValueType(),
1865:                                        Base, RC), 0);
1866:     }
1867: 
1868:     // Make sure that the index register isn't assigned to %r0 either.
1869:     if (Index.getOpcode() != ISD::Register) {
1870:       Index =
1871:         SDValue(CurDAG->getMachineNode(TargetOpcode::COPY_TO_REGCLASS,
1872:                                        DL, Index.getValueType(),
1873:                                        Index, RC), 0);
1874:     }
1875: 
1876:     OutOps.push_back(Base);
1877:     OutOps.push_back(Disp);
1878:     OutOps.push_back(Index);
1879:     return false;
1880:   }
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1881-1920 / 第 1881-1920 行
```cpp
1881: 
1882:   return true;
1883: }
1884: 
1885: // IsProfitableToFold - Returns true if is profitable to fold the specific
1886: // operand node N of U during instruction selection that starts at Root.
1887: bool
1888: SystemZDAGToDAGISel::IsProfitableToFold(SDValue N, SDNode *U,
1889:                                         SDNode *Root) const {
1890:   // We want to avoid folding a LOAD into an ICMP node if as a result
1891:   // we would be forced to spill the condition code into a GPR.
1892:   if (N.getOpcode() == ISD::LOAD && U->getOpcode() == SystemZISD::ICMP) {
1893:     if (!N.hasOneUse() || !U->hasOneUse())
1894:       return false;
1895: 
1896:     // The user of the CC value will usually be a CopyToReg into the
1897:     // physical CC register, which in turn is glued and chained to the
1898:     // actual instruction that uses the CC value.  Bail out if we have
1899:     // anything else than that.
1900:     SDNode *CCUser = *U->user_begin();
1901:     SDNode *CCRegUser = nullptr;
1902:     if (CCUser->getOpcode() == ISD::CopyToReg ||
1903:         cast<RegisterSDNode>(CCUser->getOperand(1))->getReg() == SystemZ::CC) {
1904:       for (auto *U : CCUser->users()) {
1905:         if (CCRegUser == nullptr)
1906:           CCRegUser = U;
1907:         else if (CCRegUser != U)
1908:           return false;
1909:       }
1910:     }
1911:     if (CCRegUser == nullptr)
1912:       return false;
1913: 
1914:     // If the actual instruction is a branch, the only thing that remains to be
1915:     // checked is whether the CCUser chain is a predecessor of the load.
1916:     if (CCRegUser->isMachineOpcode() &&
1917:         CCRegUser->getMachineOpcode() == SystemZ::BRC)
1918:       return !N->isPredecessorOf(CCUser->getOperand(0).getNode());
1919: 
1920:     // Otherwise, the instruction may have multiple operands, and we need to
```
- **EN**: The range implements or declares functions including `SystemZDAGToDAGISel::IsProfitableToFold`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZDAGToDAGISel::IsProfitableToFold` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 1921-1960 / 第 1921-1960 行
```cpp
1921:     // verify that none of them are a predecessor of the load.  This is exactly
1922:     // the same check that would be done by common code if the CC setter were
1923:     // glued to the CC user, so simply invoke that check here.
1924:     if (!IsLegalToFold(N, U, CCRegUser, OptLevel, false))
1925:       return false;
1926:   }
1927: 
1928:   return true;
1929: }
1930: 
1931: namespace {
1932: // Represents a sequence for extracting a 0/1 value from an IPM result:
1933: // (((X ^ XORValue) + AddValue) >> Bit)
1934: struct IPMConversion {
1935:   IPMConversion(unsigned xorValue, int64_t addValue, unsigned bit)
1936:     : XORValue(xorValue), AddValue(addValue), Bit(bit) {}
1937: 
1938:   int64_t XORValue;
1939:   int64_t AddValue;
1940:   unsigned Bit;
1941: };
1942: } // end anonymous namespace
1943: 
1944: // Return a sequence for getting a 1 from an IPM result when CC has a
1945: // value in CCMask and a 0 when CC has a value in CCValid & ~CCMask.
1946: // The handling of CC values outside CCValid doesn't matter.
1947: static IPMConversion getIPMConversion(unsigned CCValid, unsigned CCMask) {
1948:   // Deal with cases where the result can be taken directly from a bit
1949:   // of the IPM result.
1950:   if (CCMask == (CCValid & (SystemZ::CCMASK_1 | SystemZ::CCMASK_3)))
1951:     return IPMConversion(0, 0, SystemZ::IPM_CC);
1952:   if (CCMask == (CCValid & (SystemZ::CCMASK_2 | SystemZ::CCMASK_3)))
1953:     return IPMConversion(0, 0, SystemZ::IPM_CC + 1);
1954: 
1955:   // Deal with cases where we can add a value to force the sign bit
1956:   // to contain the right value.  Putting the bit in 31 means we can
1957:   // use SRL rather than RISBG(L), and also makes it easier to get a
1958:   // 0/-1 value, so it has priority over the other tests below.
1959:   //
1960:   // These sequences rely on the fact that the upper two bits of the
```
- **EN**: It introduces interface types such as `IPMConversion`, shaping how other backend components interact with this file. The range implements or declares functions including `IPMConversion`, `getIPMConversion`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 它引入了 `IPMConversion` 等接口类型，定义了其他后端组件与本文件交互的方式。 这一段实现或声明了 `IPMConversion`, `getIPMConversion` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1961-2000 / 第 1961-2000 行
```cpp
1961:   // IPM result are zero.
1962:   uint64_t TopBit = uint64_t(1) << 31;
1963:   if (CCMask == (CCValid & SystemZ::CCMASK_0))
1964:     return IPMConversion(0, -(1 << SystemZ::IPM_CC), 31);
1965:   if (CCMask == (CCValid & (SystemZ::CCMASK_0 | SystemZ::CCMASK_1)))
1966:     return IPMConversion(0, -(2 << SystemZ::IPM_CC), 31);
1967:   if (CCMask == (CCValid & (SystemZ::CCMASK_0
1968:                             | SystemZ::CCMASK_1
1969:                             | SystemZ::CCMASK_2)))
1970:     return IPMConversion(0, -(3 << SystemZ::IPM_CC), 31);
1971:   if (CCMask == (CCValid & SystemZ::CCMASK_3))
1972:     return IPMConversion(0, TopBit - (3 << SystemZ::IPM_CC), 31);
1973:   if (CCMask == (CCValid & (SystemZ::CCMASK_1
1974:                             | SystemZ::CCMASK_2
1975:                             | SystemZ::CCMASK_3)))
1976:     return IPMConversion(0, TopBit - (1 << SystemZ::IPM_CC), 31);
1977: 
1978:   // Next try inverting the value and testing a bit.  0/1 could be
1979:   // handled this way too, but we dealt with that case above.
1980:   if (CCMask == (CCValid & (SystemZ::CCMASK_0 | SystemZ::CCMASK_2)))
1981:     return IPMConversion(-1, 0, SystemZ::IPM_CC);
1982: 
1983:   // Handle cases where adding a value forces a non-sign bit to contain
1984:   // the right value.
1985:   if (CCMask == (CCValid & (SystemZ::CCMASK_1 | SystemZ::CCMASK_2)))
1986:     return IPMConversion(0, 1 << SystemZ::IPM_CC, SystemZ::IPM_CC + 1);
1987:   if (CCMask == (CCValid & (SystemZ::CCMASK_0 | SystemZ::CCMASK_3)))
1988:     return IPMConversion(0, -(1 << SystemZ::IPM_CC), SystemZ::IPM_CC + 1);
1989: 
1990:   // The remaining cases are 1, 2, 0/1/3 and 0/2/3.  All these are
1991:   // can be done by inverting the low CC bit and applying one of the
1992:   // sign-based extractions above.
1993:   if (CCMask == (CCValid & SystemZ::CCMASK_1))
1994:     return IPMConversion(1 << SystemZ::IPM_CC, -(1 << SystemZ::IPM_CC), 31);
1995:   if (CCMask == (CCValid & SystemZ::CCMASK_2))
1996:     return IPMConversion(1 << SystemZ::IPM_CC,
1997:                          TopBit - (3 << SystemZ::IPM_CC), 31);
1998:   if (CCMask == (CCValid & (SystemZ::CCMASK_0
1999:                             | SystemZ::CCMASK_1
2000:                             | SystemZ::CCMASK_3)))
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 2001-2040 / 第 2001-2040 行
```cpp
2001:     return IPMConversion(1 << SystemZ::IPM_CC, -(3 << SystemZ::IPM_CC), 31);
2002:   if (CCMask == (CCValid & (SystemZ::CCMASK_0
2003:                             | SystemZ::CCMASK_2
2004:                             | SystemZ::CCMASK_3)))
2005:     return IPMConversion(1 << SystemZ::IPM_CC,
2006:                          TopBit - (1 << SystemZ::IPM_CC), 31);
2007: 
2008:   llvm_unreachable("Unexpected CC combination");
2009: }
2010: 
2011: SDValue SystemZDAGToDAGISel::expandSelectBoolean(SDNode *Node) {
2012:   auto *TrueOp = dyn_cast<ConstantSDNode>(Node->getOperand(0));
2013:   auto *FalseOp = dyn_cast<ConstantSDNode>(Node->getOperand(1));
2014:   if (!TrueOp || !FalseOp)
2015:     return SDValue();
2016:   if (FalseOp->getZExtValue() != 0)
2017:     return SDValue();
2018:   if (TrueOp->getSExtValue() != 1 && TrueOp->getSExtValue() != -1)
2019:     return SDValue();
2020: 
2021:   auto *CCValidOp = dyn_cast<ConstantSDNode>(Node->getOperand(2));
2022:   auto *CCMaskOp = dyn_cast<ConstantSDNode>(Node->getOperand(3));
2023:   if (!CCValidOp || !CCMaskOp)
2024:     return SDValue();
2025:   int CCValid = CCValidOp->getZExtValue();
2026:   int CCMask = CCMaskOp->getZExtValue();
2027: 
2028:   SDLoc DL(Node);
2029:   SDValue CCReg = Node->getOperand(4);
2030:   IPMConversion IPM = getIPMConversion(CCValid, CCMask);
2031:   SDValue Result = CurDAG->getNode(SystemZISD::IPM, DL, MVT::i32, CCReg);
2032: 
2033:   if (IPM.XORValue)
2034:     Result = CurDAG->getNode(ISD::XOR, DL, MVT::i32, Result,
2035:                              CurDAG->getConstant(IPM.XORValue, DL, MVT::i32));
2036: 
2037:   if (IPM.AddValue)
2038:     Result =
2039:         CurDAG->getNode(ISD::ADD, DL, MVT::i32, Result,
2040:                         CurDAG->getSignedConstant(IPM.AddValue, DL, MVT::i32));
```
- **EN**: The range implements or declares functions including `SystemZDAGToDAGISel::expandSelectBoolean`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `SystemZDAGToDAGISel::expandSelectBoolean` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 2041-2080 / 第 2041-2080 行
```cpp
2041: 
2042:   EVT VT = Node->getValueType(0);
2043:   if (VT == MVT::i32 && IPM.Bit == 31) {
2044:     unsigned ShiftOp = TrueOp->getSExtValue() == 1 ? ISD::SRL : ISD::SRA;
2045:     Result = CurDAG->getNode(ShiftOp, DL, MVT::i32, Result,
2046:                              CurDAG->getConstant(IPM.Bit, DL, MVT::i32));
2047:   } else {
2048:     if (VT != MVT::i32)
2049:       Result = CurDAG->getNode(ISD::ANY_EXTEND, DL, VT, Result);
2050: 
2051:     if (TrueOp->getSExtValue() == 1) {
2052:       // The SHR/AND sequence should get optimized to an RISBG.
2053:       Result = CurDAG->getNode(ISD::SRL, DL, VT, Result,
2054:                                CurDAG->getConstant(IPM.Bit, DL, MVT::i32));
2055:       Result = CurDAG->getNode(ISD::AND, DL, VT, Result,
2056:                                CurDAG->getConstant(1, DL, VT));
2057:     } else {
2058:       // Sign-extend from IPM.Bit using a pair of shifts.
2059:       int ShlAmt = VT.getSizeInBits() - 1 - IPM.Bit;
2060:       int SraAmt = VT.getSizeInBits() - 1;
2061:       Result = CurDAG->getNode(ISD::SHL, DL, VT, Result,
2062:                                CurDAG->getConstant(ShlAmt, DL, MVT::i32));
2063:       Result = CurDAG->getNode(ISD::SRA, DL, VT, Result,
2064:                                CurDAG->getConstant(SraAmt, DL, MVT::i32));
2065:     }
2066:   }
2067: 
2068:   return Result;
2069: }
2070: 
2071: bool SystemZDAGToDAGISel::shouldSelectForReassoc(SDNode *N) const {
2072:   EVT VT = N->getValueType(0);
2073:   assert(VT.isFloatingPoint() && "Expected FP SDNode");
2074:   return N->getFlags().hasAllowReassociation() &&
2075:          N->getFlags().hasNoSignedZeros() && Subtarget->hasVector() &&
2076:          (VT != MVT::f32 || Subtarget->hasVectorEnhancements1()) &&
2077:          !N->isStrictFPOpcode();
2078: }
2079: 
2080: void SystemZDAGToDAGISel::PreprocessISelDAG() {
```
- **EN**: The range implements or declares functions including `SystemZDAGToDAGISel::shouldSelectForReassoc`, `SystemZDAGToDAGISel::PreprocessISelDAG`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `SystemZDAGToDAGISel::shouldSelectForReassoc`, `SystemZDAGToDAGISel::PreprocessISelDAG` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 2081-2117 / 第 2081-2117 行
```cpp
2081:   // If we have conditional immediate loads, we always prefer
2082:   // using those over an IPM sequence.
2083:   if (Subtarget->hasLoadStoreOnCond2())
2084:     return;
2085: 
2086:   bool MadeChange = false;
2087: 
2088:   for (SelectionDAG::allnodes_iterator I = CurDAG->allnodes_begin(),
2089:                                        E = CurDAG->allnodes_end();
2090:        I != E;) {
2091:     SDNode *N = &*I++;
2092:     if (N->use_empty())
2093:       continue;
2094: 
2095:     SDValue Res;
2096:     switch (N->getOpcode()) {
2097:     default: break;
2098:     case SystemZISD::SELECT_CCMASK:
2099:       Res = expandSelectBoolean(N);
2100:       break;
2101:     }
2102: 
2103:     if (Res) {
2104:       LLVM_DEBUG(dbgs() << "SystemZ DAG preprocessing replacing:\nOld:    ");
2105:       LLVM_DEBUG(N->dump(CurDAG));
2106:       LLVM_DEBUG(dbgs() << "\nNew: ");
2107:       LLVM_DEBUG(Res.getNode()->dump(CurDAG));
2108:       LLVM_DEBUG(dbgs() << "\n");
2109: 
2110:       CurDAG->ReplaceAllUsesOfValueWith(SDValue(N, 0), Res);
2111:       MadeChange = true;
2112:     }
2113:   }
2114: 
2115:   if (MadeChange)
2116:     CurDAG->RemoveDeadNodes();
2117: }
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

## Key Concepts / 关键概念
- **SelectionDAG lowering**: Bridges LLVM IR and target-specific DAG nodes/instructions. / 连接 LLVM IR 与目标相关 DAG 节点/指令。
- **GlobalISel**: Uses legalization, register banks, and instruction selection after generic IR lowering. / 在通用 IR 降低后使用合法化、寄存器银行和指令选择。
- **MachineInstr**: Represents target-aware machine instructions during late code generation. / 表示代码生成后期的目标相关机器指令。
- **MachineFunction**: Carries per-function machine-level state and basic blocks. / 保存每个函数的机器级状态和基本块。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Register classes**: Groups registers by capability so instruction operands and allocators can constrain them. / 按能力对寄存器分组，以便指令操作数和分配器施加约束。

## Dependencies / 依赖关系
- `SystemZTargetMachine.h`
- `SystemZISelLowering.h`
- `llvm/Analysis/AliasAnalysis.h`
- `llvm/CodeGen/SelectionDAGISel.h`
- `llvm/Support/Debug.h`
- `llvm/Support/KnownBits.h`
- `llvm/Support/raw_ostream.h`
- `SystemZGenDAGISel.inc`
