# HexagonBitTracker.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonBitTracker.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): If (RC.contains(Reg))
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-100 / 第 1-100 行

```cpp
     1: //===- HexagonBitTracker.cpp ----------------------------------------------===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: 
     9: #include "HexagonBitTracker.h"
    10: #include "HexagonInstrInfo.h"
    11: #include "HexagonRegisterInfo.h"
    12: #include "HexagonSubtarget.h"
    13: #include "llvm/CodeGen/MachineFrameInfo.h"
    14: #include "llvm/CodeGen/MachineFunction.h"
    15: #include "llvm/CodeGen/MachineInstr.h"
    16: #include "llvm/CodeGen/MachineOperand.h"
    17: #include "llvm/CodeGen/MachineRegisterInfo.h"
    18: #include "llvm/IR/Argument.h"
    19: #include "llvm/IR/Attributes.h"
    20: #include "llvm/IR/Function.h"
    21: #include "llvm/IR/Type.h"
    22: #include "llvm/Support/Compiler.h"
    23: #include "llvm/Support/Debug.h"
    24: #include "llvm/Support/ErrorHandling.h"
    25: 
    26: using namespace llvm;
    27: 
    28: using BT = BitTracker;
    29: 
    30: HexagonEvaluator::HexagonEvaluator(const HexagonRegisterInfo &tri,
    31:                                    MachineRegisterInfo &mri,
    32:                                    const HexagonInstrInfo &tii,
    33:                                    MachineFunction &mf)
    34:     : MachineEvaluator(tri, mri), MF(mf), MFI(mf.getFrameInfo()), TII(tii) {
    35:   // Populate the VRX map (VR to extension-type).
    36:   // Go over all the formal parameters of the function. If a given parameter
    37:   // P is sign- or zero-extended, locate the virtual register holding that
    38:   // parameter and create an entry in the VRX map indicating the type of ex-
    39:   // tension (and the source type).
    40:   // This is a bit complicated to do accurately, since the memory layout in-
    41:   // formation is necessary to precisely determine whether an aggregate para-
    42:   // meter will be passed in a register or in memory. What is given in MRI
    43:   // is the association between the physical register that is live-in (i.e.
    44:   // holds an argument), and the virtual register that this value will be
    45:   // copied into. This, by itself, is not sufficient to map back the virtual
    46:   // register to a formal parameter from Function (since consecutive live-ins
    47:   // from MRI may not correspond to consecutive formal parameters from Func-
    48:   // tion). To avoid the complications with in-memory arguments, only consi-
    49:   // der the initial sequence of formal parameters that are known to be
    50:   // passed via registers.
    51:   unsigned InVirtReg, InPhysReg = 0;
    52: 
    53:   for (const Argument &Arg : MF.getFunction().args()) {
    54:     Type *ATy = Arg.getType();
    55:     unsigned Width = 0;
    56:     if (ATy->isIntegerTy())
    57:       Width = ATy->getIntegerBitWidth();
    58:     else if (ATy->isPointerTy())
    59:       Width = 32;
    60:     // If pointer size is not set through target data, it will default to
    61:     // Module::AnyPointerSize.
    62:     if (Width == 0 || Width > 64)
    63:       break;
    64:     if (Arg.hasAttribute(Attribute::ByVal))
    65:       continue;
    66:     InPhysReg = getNextPhysReg(InPhysReg, Width);
    67:     if (!InPhysReg)
    68:       break;
    69:     InVirtReg = getVirtRegFor(InPhysReg);
    70:     if (!InVirtReg)
    71:       continue;
    72:     if (Arg.hasAttribute(Attribute::SExt))
    73:       VRX.insert(std::make_pair(InVirtReg, ExtType(ExtType::SExt, Width)));
    74:     else if (Arg.hasAttribute(Attribute::ZExt))
    75:       VRX.insert(std::make_pair(InVirtReg, ExtType(ExtType::ZExt, Width)));
    76:   }
    77: }
    78: 
    79: BT::BitMask HexagonEvaluator::mask(Register Reg, unsigned Sub) const {
    80:   if (Sub == 0)
    81:     return MachineEvaluator::mask(Reg, 0);
    82:   const TargetRegisterClass &RC = *MRI.getRegClass(Reg);
    83:   unsigned ID = RC.getID();
    84:   uint16_t RW = getRegBitWidth(RegisterRef(Reg, Sub));
    85:   const auto &HRI = static_cast<const HexagonRegisterInfo&>(TRI);
    86:   bool IsSubLo = (Sub == HRI.getHexagonSubRegIndex(RC, Hexagon::ps_sub_lo));
    87:   switch (ID) {
    88:     case Hexagon::DoubleRegsRegClassID:
    89:     case Hexagon::DoubleRegs_with_isub_hi_in_IntRegsLow8RegClassID:
    90:     case Hexagon::HvxWRRegClassID:
    91:     case Hexagon::HvxVQRRegClassID:
    92:       return IsSubLo ? BT::BitMask(0, RW-1)
    93:                      : BT::BitMask(RW, 2*RW-1);
    94:     default:
    95:       break;
    96:   }
    97: #ifndef NDEBUG
    98:   dbgs() << printReg(Reg, &TRI, Sub) << " in reg class "
    99:          << TRI.getRegClassName(&RC) << '\n';
   100: #endif
```
- EN: It imports headers such as HexagonBitTracker.h, HexagonInstrInfo.h, HexagonRegisterInfo.h, HexagonSubtarget.h, ... (16 total), establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as HexagonEvaluator::HexagonEvaluator, getType, getNextPhysReg, getVirtRegFor, ... (10 total), translating Hexagon-specific policy into reusable code paths.
- CN: 这里引入了 HexagonBitTracker.h, HexagonInstrInfo.h, HexagonRegisterInfo.h, HexagonSubtarget.h, ... (16 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 HexagonEvaluator::HexagonEvaluator, getType, getNextPhysReg, getVirtRegFor, ... (10 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

### Lines 101-200 / 第 101-200 行

```cpp
   101:   llvm_unreachable("Unexpected register/subregister");
   102: }
   103: 
   104: uint16_t HexagonEvaluator::getPhysRegBitWidth(MCRegister Reg) const {
   105:   using namespace Hexagon;
   106:   const auto &HST = MF.getSubtarget<HexagonSubtarget>();
   107:   if (HST.useHVXOps()) {
   108:     for (auto &RC : {HvxVRRegClass, HvxWRRegClass, HvxQRRegClass,
   109:                      HvxVQRRegClass})
   110:       if (RC.contains(Reg))
   111:         return TRI.getRegSizeInBits(RC);
   112:   }
   113:   // Default treatment for other physical registers.
   114:   if (const TargetRegisterClass *RC = TRI.getMinimalPhysRegClass(Reg))
   115:     return TRI.getRegSizeInBits(*RC);
   116: 
   117:   llvm_unreachable(
   118:       (Twine("Unhandled physical register") + TRI.getName(Reg)).str().c_str());
   119: }
   120: 
   121: const TargetRegisterClass &HexagonEvaluator::composeWithSubRegIndex(
   122:       const TargetRegisterClass &RC, unsigned Idx) const {
   123:   if (Idx == 0)
   124:     return RC;
   125: 
   126: #ifndef NDEBUG
   127:   const auto &HRI = static_cast<const HexagonRegisterInfo&>(TRI);
   128:   bool IsSubLo = (Idx == HRI.getHexagonSubRegIndex(RC, Hexagon::ps_sub_lo));
   129:   bool IsSubHi = (Idx == HRI.getHexagonSubRegIndex(RC, Hexagon::ps_sub_hi));
   130:   assert(IsSubLo != IsSubHi && "Must refer to either low or high subreg");
   131: #endif
   132: 
   133:   switch (RC.getID()) {
   134:     case Hexagon::DoubleRegsRegClassID:
   135:     case Hexagon::DoubleRegs_with_isub_hi_in_IntRegsLow8RegClassID:
   136:       return Hexagon::IntRegsRegClass;
   137:     case Hexagon::HvxWRRegClassID:
   138:       return Hexagon::HvxVRRegClass;
   139:     case Hexagon::HvxVQRRegClassID:
   140:       return Hexagon::HvxWRRegClass;
   141:     default:
   142:       break;
   143:   }
   144: #ifndef NDEBUG
   145:   dbgs() << "Reg class id: " << RC.getID() << " idx: " << Idx << '\n';
   146: #endif
   147:   llvm_unreachable("Unimplemented combination of reg class/subreg idx");
   148: }
   149: 
   150: namespace {
   151: 
   152: class RegisterRefs {
   153:   std::vector<BT::RegisterRef> Vector;
   154: 
   155: public:
   156:   RegisterRefs(const MachineInstr &MI) : Vector(MI.getNumOperands()) {
   157:     for (unsigned i = 0, n = Vector.size(); i < n; ++i) {
   158:       const MachineOperand &MO = MI.getOperand(i);
   159:       if (MO.isReg())
   160:         Vector[i] = BT::RegisterRef(MO);
   161:       // For indices that don't correspond to registers, the entry will
   162:       // remain constructed via the default constructor.
   163:     }
   164:   }
   165: 
   166:   size_t size() const { return Vector.size(); }
   167: 
   168:   const BT::RegisterRef &operator[](unsigned n) const {
   169:     // The main purpose of this operator is to assert with bad argument.
   170:     assert(n < Vector.size());
   171:     return Vector[n];
   172:   }
   173: };
   174: 
   175: } // end anonymous namespace
   176: 
   177: bool HexagonEvaluator::evaluate(const MachineInstr &MI,
   178:                                 const CellMapType &Inputs,
   179:                                 CellMapType &Outputs) const {
   180:   using namespace Hexagon;
   181: 
   182:   unsigned NumDefs = 0;
   183: 
   184:   // Basic correctness check: there should not be any defs with subregisters.
   185:   for (const MachineOperand &MO : MI.operands()) {
   186:     if (!MO.isReg() || !MO.isDef())
   187:       continue;
   188:     NumDefs++;
   189:     assert(MO.getSubReg() == 0);
   190:   }
   191: 
   192:   if (NumDefs == 0)
   193:     return false;
   194: 
   195:   unsigned Opc = MI.getOpcode();
   196: 
   197:   if (MI.mayLoad()) {
   198:     switch (Opc) {
   199:       // These instructions may be marked as mayLoad, but they are generating
   200:       // immediate values, so skip them.
```
- EN: Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (Hexagon, bool) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as RegisterRefs, which carry the state or API of this component. It defines declarative TableGen records like RegisterRefs; these records are consumed by TableGen instead of executed directly.
- CN: 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（Hexagon, bool），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 RegisterRefs 等类型，用来承载该组件的状态或接口。 这里定义了 RegisterRefs 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 201-300 / 第 201-300 行

```cpp
   201:       case CONST32:
   202:       case CONST64:
   203:         break;
   204:       default:
   205:         return evaluateLoad(MI, Inputs, Outputs);
   206:     }
   207:   }
   208: 
   209:   // Check COPY instructions that copy formal parameters into virtual
   210:   // registers. Such parameters can be sign- or zero-extended at the
   211:   // call site, and we should take advantage of this knowledge. The MRI
   212:   // keeps a list of pairs of live-in physical and virtual registers,
   213:   // which provides information about which virtual registers will hold
   214:   // the argument values. The function will still contain instructions
   215:   // defining those virtual registers, and in practice those are COPY
   216:   // instructions from a physical to a virtual register. In such cases,
   217:   // applying the argument extension to the virtual register can be seen
   218:   // as simply mirroring the extension that had already been applied to
   219:   // the physical register at the call site. If the defining instruction
   220:   // was not a COPY, it would not be clear how to mirror that extension
   221:   // on the callee's side. For that reason, only check COPY instructions
   222:   // for potential extensions.
   223:   if (MI.isCopy()) {
   224:     if (evaluateFormalCopy(MI, Inputs, Outputs))
   225:       return true;
   226:   }
   227: 
   228:   // Beyond this point, if any operand is a global, skip that instruction.
   229:   // The reason is that certain instructions that can take an immediate
   230:   // operand can also have a global symbol in that operand. To avoid
   231:   // checking what kind of operand a given instruction has individually
   232:   // for each instruction, do it here. Global symbols as operands gene-
   233:   // rally do not provide any useful information.
   234:   for (const MachineOperand &MO : MI.operands()) {
   235:     if (MO.isGlobal() || MO.isBlockAddress() || MO.isSymbol() || MO.isJTI() ||
   236:         MO.isCPI())
   237:       return false;
   238:   }
   239: 
   240:   RegisterRefs Reg(MI);
   241: #define op(i) MI.getOperand(i)
   242: #define rc(i) RegisterCell::ref(getCell(Reg[i], Inputs))
   243: #define im(i) MI.getOperand(i).getImm()
   244: 
   245:   // If the instruction has no register operands, skip it.
   246:   if (Reg.size() == 0)
   247:     return false;
   248: 
   249:   // Record result for register in operand 0.
   250:   auto rr0 = [this,Reg] (const BT::RegisterCell &Val, CellMapType &Outputs)
   251:         -> bool {
   252:     putCell(Reg[0], Val, Outputs);
   253:     return true;
   254:   };
   255:   // Get the cell corresponding to the N-th operand.
   256:   auto cop = [this, &Reg, &MI, &Inputs](unsigned N,
   257:                                         uint16_t W) -> BT::RegisterCell {
   258:     const MachineOperand &Op = MI.getOperand(N);
   259:     if (Op.isImm())
   260:       return eIMM(Op.getImm(), W);
   261:     if (!Op.isReg())
   262:       return RegisterCell::self(0, W);
   263:     assert(getRegBitWidth(Reg[N]) == W && "Register width mismatch");
   264:     return rc(N);
   265:   };
   266:   // Extract RW low bits of the cell.
   267:   auto lo = [this] (const BT::RegisterCell &RC, uint16_t RW)
   268:         -> BT::RegisterCell {
   269:     assert(RW <= RC.width());
   270:     return eXTR(RC, 0, RW);
   271:   };
   272:   // Extract RW high bits of the cell.
   273:   auto hi = [this] (const BT::RegisterCell &RC, uint16_t RW)
   274:         -> BT::RegisterCell {
   275:     uint16_t W = RC.width();
   276:     assert(RW <= W);
   277:     return eXTR(RC, W-RW, W);
   278:   };
   279:   // Extract N-th halfword (counting from the least significant position).
   280:   auto half = [this] (const BT::RegisterCell &RC, unsigned N)
   281:         -> BT::RegisterCell {
   282:     assert(N*16+16 <= RC.width());
   283:     return eXTR(RC, N*16, N*16+16);
   284:   };
   285:   // Shuffle bits (pick even/odd from cells and merge into result).
   286:   auto shuffle = [this] (const BT::RegisterCell &Rs, const BT::RegisterCell &Rt,
   287:                          uint16_t BW, bool Odd) -> BT::RegisterCell {
   288:     uint16_t I = Odd, Ws = Rs.width();
   289:     assert(Ws == Rt.width());
   290:     RegisterCell RC = eXTR(Rt, I*BW, I*BW+BW).cat(eXTR(Rs, I*BW, I*BW+BW));
   291:     I += 2;
   292:     while (I*BW < Ws) {
   293:       RC.cat(eXTR(Rt, I*BW, I*BW+BW)).cat(eXTR(Rs, I*BW, I*BW+BW));
   294:       I += 2;
   295:     }
   296:     return RC;
   297:   };
   298: 
   299:   // The bitwidth of the 0th operand. In most (if not all) of the
   300:   // instructions below, the 0th operand is the defined register.
```
- EN: Header guards in this range prevent duplicate inclusion and define the interface boundary. It declares or implements routines such as evaluateLoad, Reg, putCell, getOperand, ... (9 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里声明或实现了 evaluateLoad, Reg, putCell, getOperand, ... (9 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 301-400 / 第 301-400 行

```cpp
   301:   // Pre-compute the bitwidth here, because it is needed in many cases
   302:   // cases below.
   303:   uint16_t W0 = (Reg[0].Reg != 0) ? getRegBitWidth(Reg[0]) : 0;
   304: 
   305:   // Register id of the 0th operand. It can be 0.
   306:   unsigned Reg0 = Reg[0].Reg;
   307: 
   308:   switch (Opc) {
   309:     // Transfer immediate:
   310: 
   311:     case A2_tfrsi:
   312:     case A2_tfrpi:
   313:     case CONST32:
   314:     case CONST64:
   315:       return rr0(eIMM(im(1), W0), Outputs);
   316:     case PS_false:
   317:       return rr0(RegisterCell(W0).fill(0, W0, BT::BitValue::Zero), Outputs);
   318:     case PS_true:
   319:       return rr0(RegisterCell(W0).fill(0, W0, BT::BitValue::One), Outputs);
   320:     case PS_fi: {
   321:       int FI = op(1).getIndex();
   322:       int Off = op(2).getImm();
   323:       unsigned A = MFI.getObjectAlign(FI).value() + std::abs(Off);
   324:       unsigned L = llvm::countr_zero(A);
   325:       RegisterCell RC = RegisterCell::self(Reg[0].Reg, W0);
   326:       RC.fill(0, L, BT::BitValue::Zero);
   327:       return rr0(RC, Outputs);
   328:     }
   329: 
   330:     // Transfer register:
   331: 
   332:     case A2_tfr:
   333:     case A2_tfrp:
   334:     case C2_pxfer_map:
   335:       return rr0(rc(1), Outputs);
   336:     case C2_tfrpr: {
   337:       uint16_t RW = W0;
   338:       uint16_t PW = 8; // XXX Pred size: getRegBitWidth(Reg[1]);
   339:       assert(PW <= RW);
   340:       RegisterCell PC = eXTR(rc(1), 0, PW);
   341:       RegisterCell RC = RegisterCell(RW).insert(PC, BT::BitMask(0, PW-1));
   342:       RC.fill(PW, RW, BT::BitValue::Zero);
   343:       return rr0(RC, Outputs);
   344:     }
   345:     case C2_tfrrp: {
   346:       uint16_t RW = W0;
   347:       uint16_t PW = 8; // XXX Pred size: getRegBitWidth(Reg[1]);
   348:       RegisterCell RC = RegisterCell::self(Reg[0].Reg, RW);
   349:       RC.fill(PW, RW, BT::BitValue::Zero);
   350:       return rr0(eINS(RC, eXTR(rc(1), 0, PW), 0), Outputs);
   351:     }
   352: 
   353:     // Arithmetic:
   354: 
   355:     case A2_abs:
   356:     case A2_absp:
   357:       // TODO
   358:       break;
   359: 
   360:     case A2_addsp: {
   361:       uint16_t W1 = getRegBitWidth(Reg[1]);
   362:       assert(W0 == 64 && W1 == 32);
   363:       RegisterCell CW = RegisterCell(W0).insert(rc(1), BT::BitMask(0, W1-1));
   364:       RegisterCell RC = eADD(eSXT(CW, W1), rc(2));
   365:       return rr0(RC, Outputs);
   366:     }
   367:     case A2_add:
   368:     case A2_addp:
   369:       return rr0(eADD(rc(1), rc(2)), Outputs);
   370:     case A2_addi:
   371:       return rr0(eADD(rc(1), eIMM(im(2), W0)), Outputs);
   372:     case S4_addi_asl_ri: {
   373:       RegisterCell RC = eADD(eIMM(im(1), W0), eASL(rc(2), im(3)));
   374:       return rr0(RC, Outputs);
   375:     }
   376:     case S4_addi_lsr_ri: {
   377:       RegisterCell RC = eADD(eIMM(im(1), W0), eLSR(rc(2), im(3)));
   378:       return rr0(RC, Outputs);
   379:     }
   380:     case S4_addaddi: {
   381:       RegisterCell RC = eADD(rc(1), eADD(rc(2), eIMM(im(3), W0)));
   382:       return rr0(RC, Outputs);
   383:     }
   384:     case M4_mpyri_addi: {
   385:       RegisterCell M = eMLS(rc(2), eIMM(im(3), W0));
   386:       RegisterCell RC = eADD(eIMM(im(1), W0), lo(M, W0));
   387:       return rr0(RC, Outputs);
   388:     }
   389:     case M4_mpyrr_addi: {
   390:       RegisterCell M = eMLS(rc(2), rc(3));
   391:       RegisterCell RC = eADD(eIMM(im(1), W0), lo(M, W0));
   392:       return rr0(RC, Outputs);
   393:     }
   394:     case M4_mpyri_addr_u2: {
   395:       RegisterCell M = eMLS(eIMM(im(2), W0), rc(3));
   396:       RegisterCell RC = eADD(rc(1), lo(M, W0));
   397:       return rr0(RC, Outputs);
   398:     }
   399:     case M4_mpyri_addr: {
   400:       RegisterCell M = eMLS(rc(2), eIMM(im(3), W0));
```
- EN: It declares or implements routines such as rr0, op, getObjectAlign, llvm::countr_zero, ... (12 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明或实现了 rr0, op, getObjectAlign, llvm::countr_zero, ... (12 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 401-500 / 第 401-500 行

```cpp
   401:       RegisterCell RC = eADD(rc(1), lo(M, W0));
   402:       return rr0(RC, Outputs);
   403:     }
   404:     case M4_mpyrr_addr: {
   405:       RegisterCell M = eMLS(rc(2), rc(3));
   406:       RegisterCell RC = eADD(rc(1), lo(M, W0));
   407:       return rr0(RC, Outputs);
   408:     }
   409:     case S4_subaddi: {
   410:       RegisterCell RC = eADD(rc(1), eSUB(eIMM(im(2), W0), rc(3)));
   411:       return rr0(RC, Outputs);
   412:     }
   413:     case M2_accii: {
   414:       RegisterCell RC = eADD(rc(1), eADD(rc(2), eIMM(im(3), W0)));
   415:       return rr0(RC, Outputs);
   416:     }
   417:     case M2_acci: {
   418:       RegisterCell RC = eADD(rc(1), eADD(rc(2), rc(3)));
   419:       return rr0(RC, Outputs);
   420:     }
   421:     case M2_subacc: {
   422:       RegisterCell RC = eADD(rc(1), eSUB(rc(2), rc(3)));
   423:       return rr0(RC, Outputs);
   424:     }
   425:     case S2_addasl_rrri: {
   426:       RegisterCell RC = eADD(rc(1), eASL(rc(2), im(3)));
   427:       return rr0(RC, Outputs);
   428:     }
   429:     case C4_addipc: {
   430:       RegisterCell RPC = RegisterCell::self(Reg[0].Reg, W0);
   431:       RPC.fill(0, 2, BT::BitValue::Zero);
   432:       return rr0(eADD(RPC, eIMM(im(2), W0)), Outputs);
   433:     }
   434:     case A2_sub:
   435:     case A2_subp:
   436:       return rr0(eSUB(rc(1), rc(2)), Outputs);
   437:     case A2_subri:
   438:       return rr0(eSUB(eIMM(im(1), W0), rc(2)), Outputs);
   439:     case S4_subi_asl_ri: {
   440:       RegisterCell RC = eSUB(eIMM(im(1), W0), eASL(rc(2), im(3)));
   441:       return rr0(RC, Outputs);
   442:     }
   443:     case S4_subi_lsr_ri: {
   444:       RegisterCell RC = eSUB(eIMM(im(1), W0), eLSR(rc(2), im(3)));
   445:       return rr0(RC, Outputs);
   446:     }
   447:     case M2_naccii: {
   448:       RegisterCell RC = eSUB(rc(1), eADD(rc(2), eIMM(im(3), W0)));
   449:       return rr0(RC, Outputs);
   450:     }
   451:     case M2_nacci: {
   452:       RegisterCell RC = eSUB(rc(1), eADD(rc(2), rc(3)));
   453:       return rr0(RC, Outputs);
   454:     }
   455:     // 32-bit negation is done by "Rd = A2_subri 0, Rs"
   456:     case A2_negp:
   457:       return rr0(eSUB(eIMM(0, W0), rc(1)), Outputs);
   458: 
   459:     case M2_mpy_up: {
   460:       RegisterCell M = eMLS(rc(1), rc(2));
   461:       return rr0(hi(M, W0), Outputs);
   462:     }
   463:     case M2_dpmpyss_s0:
   464:       return rr0(eMLS(rc(1), rc(2)), Outputs);
   465:     case M2_dpmpyss_acc_s0:
   466:       return rr0(eADD(rc(1), eMLS(rc(2), rc(3))), Outputs);
   467:     case M2_dpmpyss_nac_s0:
   468:       return rr0(eSUB(rc(1), eMLS(rc(2), rc(3))), Outputs);
   469:     case M2_mpyi: {
   470:       RegisterCell M = eMLS(rc(1), rc(2));
   471:       return rr0(lo(M, W0), Outputs);
   472:     }
   473:     case M2_macsip: {
   474:       RegisterCell M = eMLS(rc(2), eIMM(im(3), W0));
   475:       RegisterCell RC = eADD(rc(1), lo(M, W0));
   476:       return rr0(RC, Outputs);
   477:     }
   478:     case M2_macsin: {
   479:       RegisterCell M = eMLS(rc(2), eIMM(im(3), W0));
   480:       RegisterCell RC = eSUB(rc(1), lo(M, W0));
   481:       return rr0(RC, Outputs);
   482:     }
   483:     case M2_maci: {
   484:       RegisterCell M = eMLS(rc(2), rc(3));
   485:       RegisterCell RC = eADD(rc(1), lo(M, W0));
   486:       return rr0(RC, Outputs);
   487:     }
   488:     case M2_mnaci: {
   489:       RegisterCell M = eMLS(rc(2), rc(3));
   490:       RegisterCell RC = eSUB(rc(1), lo(M, W0));
   491:       return rr0(RC, Outputs);
   492:     }
   493:     case M2_mpysmi: {
   494:       RegisterCell M = eMLS(rc(1), eIMM(im(2), W0));
   495:       return rr0(lo(M, 32), Outputs);
   496:     }
   497:     case M2_mpysin: {
   498:       RegisterCell M = eMLS(rc(1), eIMM(-im(2), W0));
   499:       return rr0(lo(M, 32), Outputs);
   500:     }
```
- EN: It declares or implements routines such as eADD, rr0, eMLS, RegisterCell::self, ... (6 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明或实现了 eADD, rr0, eMLS, RegisterCell::self, ... (6 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 501-600 / 第 501-600 行

```cpp
   501:     case M2_mpysip: {
   502:       RegisterCell M = eMLS(rc(1), eIMM(im(2), W0));
   503:       return rr0(lo(M, 32), Outputs);
   504:     }
   505:     case M2_mpyu_up: {
   506:       RegisterCell M = eMLU(rc(1), rc(2));
   507:       return rr0(hi(M, W0), Outputs);
   508:     }
   509:     case M2_dpmpyuu_s0:
   510:       return rr0(eMLU(rc(1), rc(2)), Outputs);
   511:     case M2_dpmpyuu_acc_s0:
   512:       return rr0(eADD(rc(1), eMLU(rc(2), rc(3))), Outputs);
   513:     case M2_dpmpyuu_nac_s0:
   514:       return rr0(eSUB(rc(1), eMLU(rc(2), rc(3))), Outputs);
   515:     //case M2_mpysu_up:
   516: 
   517:     // Logical/bitwise:
   518: 
   519:     case A2_andir:
   520:       return rr0(eAND(rc(1), eIMM(im(2), W0)), Outputs);
   521:     case A2_and:
   522:     case A2_andp:
   523:       return rr0(eAND(rc(1), rc(2)), Outputs);
   524:     case A4_andn:
   525:     case A4_andnp:
   526:       return rr0(eAND(rc(1), eNOT(rc(2))), Outputs);
   527:     case S4_andi_asl_ri: {
   528:       RegisterCell RC = eAND(eIMM(im(1), W0), eASL(rc(2), im(3)));
   529:       return rr0(RC, Outputs);
   530:     }
   531:     case S4_andi_lsr_ri: {
   532:       RegisterCell RC = eAND(eIMM(im(1), W0), eLSR(rc(2), im(3)));
   533:       return rr0(RC, Outputs);
   534:     }
   535:     case M4_and_and:
   536:       return rr0(eAND(rc(1), eAND(rc(2), rc(3))), Outputs);
   537:     case M4_and_andn:
   538:       return rr0(eAND(rc(1), eAND(rc(2), eNOT(rc(3)))), Outputs);
   539:     case M4_and_or:
   540:       return rr0(eAND(rc(1), eORL(rc(2), rc(3))), Outputs);
   541:     case M4_and_xor:
   542:       return rr0(eAND(rc(1), eXOR(rc(2), rc(3))), Outputs);
   543:     case A2_orir:
   544:       return rr0(eORL(rc(1), eIMM(im(2), W0)), Outputs);
   545:     case A2_or:
   546:     case A2_orp:
   547:       return rr0(eORL(rc(1), rc(2)), Outputs);
   548:     case A4_orn:
   549:     case A4_ornp:
   550:       return rr0(eORL(rc(1), eNOT(rc(2))), Outputs);
   551:     case S4_ori_asl_ri: {
   552:       RegisterCell RC = eORL(eIMM(im(1), W0), eASL(rc(2), im(3)));
   553:       return rr0(RC, Outputs);
   554:     }
   555:     case S4_ori_lsr_ri: {
   556:       RegisterCell RC = eORL(eIMM(im(1), W0), eLSR(rc(2), im(3)));
   557:       return rr0(RC, Outputs);
   558:     }
   559:     case M4_or_and:
   560:       return rr0(eORL(rc(1), eAND(rc(2), rc(3))), Outputs);
   561:     case M4_or_andn:
   562:       return rr0(eORL(rc(1), eAND(rc(2), eNOT(rc(3)))), Outputs);
   563:     case S4_or_andi:
   564:     case S4_or_andix: {
   565:       RegisterCell RC = eORL(rc(1), eAND(rc(2), eIMM(im(3), W0)));
   566:       return rr0(RC, Outputs);
   567:     }
   568:     case S4_or_ori: {
   569:       RegisterCell RC = eORL(rc(1), eORL(rc(2), eIMM(im(3), W0)));
   570:       return rr0(RC, Outputs);
   571:     }
   572:     case M4_or_or:
   573:       return rr0(eORL(rc(1), eORL(rc(2), rc(3))), Outputs);
   574:     case M4_or_xor:
   575:       return rr0(eORL(rc(1), eXOR(rc(2), rc(3))), Outputs);
   576:     case A2_xor:
   577:     case A2_xorp:
   578:       return rr0(eXOR(rc(1), rc(2)), Outputs);
   579:     case M4_xor_and:
   580:       return rr0(eXOR(rc(1), eAND(rc(2), rc(3))), Outputs);
   581:     case M4_xor_andn:
   582:       return rr0(eXOR(rc(1), eAND(rc(2), eNOT(rc(3)))), Outputs);
   583:     case M4_xor_or:
   584:       return rr0(eXOR(rc(1), eORL(rc(2), rc(3))), Outputs);
   585:     case M4_xor_xacc:
   586:       return rr0(eXOR(rc(1), eXOR(rc(2), rc(3))), Outputs);
   587:     case A2_not:
   588:     case A2_notp:
   589:       return rr0(eNOT(rc(1)), Outputs);
   590: 
   591:     case S2_asl_i_r:
   592:     case S2_asl_i_p:
   593:       return rr0(eASL(rc(1), im(2)), Outputs);
   594:     case A2_aslh:
   595:       return rr0(eASL(rc(1), 16), Outputs);
   596:     case S2_asl_i_r_acc:
   597:     case S2_asl_i_p_acc:
   598:       return rr0(eADD(rc(1), eASL(rc(2), im(3))), Outputs);
   599:     case S2_asl_i_r_nac:
   600:     case S2_asl_i_p_nac:
```
- EN: It declares or implements routines such as eMLS, rr0, eMLU, eAND, ... (5 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明或实现了 eMLS, rr0, eMLU, eAND, ... (5 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 601-700 / 第 601-700 行

```cpp
   601:       return rr0(eSUB(rc(1), eASL(rc(2), im(3))), Outputs);
   602:     case S2_asl_i_r_and:
   603:     case S2_asl_i_p_and:
   604:       return rr0(eAND(rc(1), eASL(rc(2), im(3))), Outputs);
   605:     case S2_asl_i_r_or:
   606:     case S2_asl_i_p_or:
   607:       return rr0(eORL(rc(1), eASL(rc(2), im(3))), Outputs);
   608:     case S2_asl_i_r_xacc:
   609:     case S2_asl_i_p_xacc:
   610:       return rr0(eXOR(rc(1), eASL(rc(2), im(3))), Outputs);
   611:     case S2_asl_i_vh:
   612:     case S2_asl_i_vw:
   613:       // TODO
   614:       break;
   615: 
   616:     case S2_asr_i_r:
   617:     case S2_asr_i_p:
   618:       return rr0(eASR(rc(1), im(2)), Outputs);
   619:     case A2_asrh:
   620:       return rr0(eASR(rc(1), 16), Outputs);
   621:     case S2_asr_i_r_acc:
   622:     case S2_asr_i_p_acc:
   623:       return rr0(eADD(rc(1), eASR(rc(2), im(3))), Outputs);
   624:     case S2_asr_i_r_nac:
   625:     case S2_asr_i_p_nac:
   626:       return rr0(eSUB(rc(1), eASR(rc(2), im(3))), Outputs);
   627:     case S2_asr_i_r_and:
   628:     case S2_asr_i_p_and:
   629:       return rr0(eAND(rc(1), eASR(rc(2), im(3))), Outputs);
   630:     case S2_asr_i_r_or:
   631:     case S2_asr_i_p_or:
   632:       return rr0(eORL(rc(1), eASR(rc(2), im(3))), Outputs);
   633:     case S2_asr_i_r_rnd: {
   634:       // The input is first sign-extended to 64 bits, then the output
   635:       // is truncated back to 32 bits.
   636:       assert(W0 == 32);
   637:       RegisterCell XC = eSXT(rc(1).cat(eIMM(0, W0)), W0);
   638:       RegisterCell RC = eASR(eADD(eASR(XC, im(2)), eIMM(1, 2*W0)), 1);
   639:       return rr0(eXTR(RC, 0, W0), Outputs);
   640:     }
   641:     case S2_asr_i_r_rnd_goodsyntax: {
   642:       int64_t S = im(2);
   643:       if (S == 0)
   644:         return rr0(rc(1), Outputs);
   645:       // Result: S2_asr_i_r_rnd Rs, u5-1
   646:       RegisterCell XC = eSXT(rc(1).cat(eIMM(0, W0)), W0);
   647:       RegisterCell RC = eLSR(eADD(eASR(XC, S-1), eIMM(1, 2*W0)), 1);
   648:       return rr0(eXTR(RC, 0, W0), Outputs);
   649:     }
   650:     case S2_asr_r_vh:
   651:     case S2_asr_i_vw:
   652:     case S2_asr_i_svw_trun:
   653:       // TODO
   654:       break;
   655: 
   656:     case S2_lsr_i_r:
   657:     case S2_lsr_i_p:
   658:       return rr0(eLSR(rc(1), im(2)), Outputs);
   659:     case S2_lsr_i_r_acc:
   660:     case S2_lsr_i_p_acc:
   661:       return rr0(eADD(rc(1), eLSR(rc(2), im(3))), Outputs);
   662:     case S2_lsr_i_r_nac:
   663:     case S2_lsr_i_p_nac:
   664:       return rr0(eSUB(rc(1), eLSR(rc(2), im(3))), Outputs);
   665:     case S2_lsr_i_r_and:
   666:     case S2_lsr_i_p_and:
   667:       return rr0(eAND(rc(1), eLSR(rc(2), im(3))), Outputs);
   668:     case S2_lsr_i_r_or:
   669:     case S2_lsr_i_p_or:
   670:       return rr0(eORL(rc(1), eLSR(rc(2), im(3))), Outputs);
   671:     case S2_lsr_i_r_xacc:
   672:     case S2_lsr_i_p_xacc:
   673:       return rr0(eXOR(rc(1), eLSR(rc(2), im(3))), Outputs);
   674: 
   675:     case S2_clrbit_i: {
   676:       RegisterCell RC = rc(1);
   677:       RC[im(2)] = BT::BitValue::Zero;
   678:       return rr0(RC, Outputs);
   679:     }
   680:     case S2_setbit_i: {
   681:       RegisterCell RC = rc(1);
   682:       RC[im(2)] = BT::BitValue::One;
   683:       return rr0(RC, Outputs);
   684:     }
   685:     case S2_togglebit_i: {
   686:       RegisterCell RC = rc(1);
   687:       uint16_t BX = im(2);
   688:       RC[BX] = RC[BX].is(0) ? BT::BitValue::One
   689:                             : RC[BX].is(1) ? BT::BitValue::Zero
   690:                                            : BT::BitValue::self();
   691:       return rr0(RC, Outputs);
   692:     }
   693: 
   694:     case A4_bitspliti: {
   695:       uint16_t W1 = getRegBitWidth(Reg[1]);
   696:       uint16_t BX = im(2);
   697:       // Res.uw[1] = Rs[bx+1:], Res.uw[0] = Rs[0:bx]
   698:       const BT::BitValue Zero = BT::BitValue::Zero;
   699:       RegisterCell RZ = RegisterCell(W0).fill(BX, W1, Zero)
   700:                                         .fill(W1+(W1-BX), W0, Zero);
```
- EN: It declares or implements routines such as rr0, assert, eSXT, eASR, ... (10 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明或实现了 rr0, assert, eSXT, eASR, ... (10 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 701-800 / 第 701-800 行

```cpp
   701:       RegisterCell BF1 = eXTR(rc(1), 0, BX), BF2 = eXTR(rc(1), BX, W1);
   702:       RegisterCell RC = eINS(eINS(RZ, BF1, 0), BF2, W1);
   703:       return rr0(RC, Outputs);
   704:     }
   705:     case S4_extract:
   706:     case S4_extractp:
   707:     case S2_extractu:
   708:     case S2_extractup: {
   709:       uint16_t Wd = im(2), Of = im(3);
   710:       assert(Wd <= W0);
   711:       if (Wd == 0)
   712:         return rr0(eIMM(0, W0), Outputs);
   713:       // If the width extends beyond the register size, pad the register
   714:       // with 0 bits.
   715:       RegisterCell Pad = (Wd+Of > W0) ? rc(1).cat(eIMM(0, Wd+Of-W0)) : rc(1);
   716:       RegisterCell Ext = eXTR(Pad, Of, Wd+Of);
   717:       // Ext is short, need to extend it with 0s or sign bit.
   718:       RegisterCell RC = RegisterCell(W0).insert(Ext, BT::BitMask(0, Wd-1));
   719:       if (Opc == S2_extractu || Opc == S2_extractup)
   720:         return rr0(eZXT(RC, Wd), Outputs);
   721:       return rr0(eSXT(RC, Wd), Outputs);
   722:     }
   723:     case S2_insert:
   724:     case S2_insertp: {
   725:       uint16_t Wd = im(3), Of = im(4);
   726:       assert(Wd < W0 && Of < W0);
   727:       // If Wd+Of exceeds W0, the inserted bits are truncated.
   728:       if (Wd+Of > W0)
   729:         Wd = W0-Of;
   730:       if (Wd == 0)
   731:         return rr0(rc(1), Outputs);
   732:       return rr0(eINS(rc(1), eXTR(rc(2), 0, Wd), Of), Outputs);
   733:     }
   734: 
   735:     // Bit permutations:
   736: 
   737:     case A2_combineii:
   738:     case A4_combineii:
   739:     case A4_combineir:
   740:     case A4_combineri:
   741:     case A2_combinew:
   742:     case V6_vcombine:
   743:       assert(W0 % 2 == 0);
   744:       return rr0(cop(2, W0/2).cat(cop(1, W0/2)), Outputs);
   745:     case A2_combine_ll:
   746:     case A2_combine_lh:
   747:     case A2_combine_hl:
   748:     case A2_combine_hh: {
   749:       assert(W0 == 32);
   750:       assert(getRegBitWidth(Reg[1]) == 32 && getRegBitWidth(Reg[2]) == 32);
   751:       // Low half in the output is 0 for _ll and _hl, 1 otherwise:
   752:       unsigned LoH = !(Opc == A2_combine_ll || Opc == A2_combine_hl);
   753:       // High half in the output is 0 for _ll and _lh, 1 otherwise:
   754:       unsigned HiH = !(Opc == A2_combine_ll || Opc == A2_combine_lh);
   755:       RegisterCell R1 = rc(1);
   756:       RegisterCell R2 = rc(2);
   757:       RegisterCell RC = half(R2, LoH).cat(half(R1, HiH));
   758:       return rr0(RC, Outputs);
   759:     }
   760:     case S2_packhl: {
   761:       assert(W0 == 64);
   762:       assert(getRegBitWidth(Reg[1]) == 32 && getRegBitWidth(Reg[2]) == 32);
   763:       RegisterCell R1 = rc(1);
   764:       RegisterCell R2 = rc(2);
   765:       RegisterCell RC = half(R2, 0).cat(half(R1, 0)).cat(half(R2, 1))
   766:                                    .cat(half(R1, 1));
   767:       return rr0(RC, Outputs);
   768:     }
   769:     case S2_shuffeb: {
   770:       RegisterCell RC = shuffle(rc(1), rc(2), 8, false);
   771:       return rr0(RC, Outputs);
   772:     }
   773:     case S2_shuffeh: {
   774:       RegisterCell RC = shuffle(rc(1), rc(2), 16, false);
   775:       return rr0(RC, Outputs);
   776:     }
   777:     case S2_shuffob: {
   778:       RegisterCell RC = shuffle(rc(1), rc(2), 8, true);
   779:       return rr0(RC, Outputs);
   780:     }
   781:     case S2_shuffoh: {
   782:       RegisterCell RC = shuffle(rc(1), rc(2), 16, true);
   783:       return rr0(RC, Outputs);
   784:     }
   785:     case C2_mask: {
   786:       uint16_t WR = W0;
   787:       uint16_t WP = 8; // XXX Pred size: getRegBitWidth(Reg[1]);
   788:       assert(WR == 64 && WP == 8);
   789:       RegisterCell R1 = rc(1);
   790:       RegisterCell RC(WR);
   791:       for (uint16_t i = 0; i < WP; ++i) {
   792:         const BT::BitValue &V = R1[i];
   793:         BT::BitValue F = (V.is(0) || V.is(1)) ? V : BT::BitValue::self();
   794:         RC.fill(i*8, i*8+8, F);
   795:       }
   796:       return rr0(RC, Outputs);
   797:     }
   798: 
   799:     // Mux:
   800: 
```
- EN: It declares or implements routines such as eXTR, eINS, rr0, im, ... (13 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明或实现了 eXTR, eINS, rr0, im, ... (13 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 801-900 / 第 801-900 行

```cpp
   801:     case C2_muxii:
   802:     case C2_muxir:
   803:     case C2_muxri:
   804:     case C2_mux: {
   805:       BT::BitValue PC0 = rc(1)[0];
   806:       RegisterCell R2 = cop(2, W0);
   807:       RegisterCell R3 = cop(3, W0);
   808:       if (PC0.is(0) || PC0.is(1))
   809:         return rr0(RegisterCell::ref(PC0 ? R2 : R3), Outputs);
   810:       R2.meet(R3, Reg[0].Reg);
   811:       return rr0(R2, Outputs);
   812:     }
   813:     case C2_vmux:
   814:       // TODO
   815:       break;
   816: 
   817:     // Sign- and zero-extension:
   818: 
   819:     case A2_sxtb:
   820:       return rr0(eSXT(rc(1), 8), Outputs);
   821:     case A2_sxth:
   822:       return rr0(eSXT(rc(1), 16), Outputs);
   823:     case A2_sxtw: {
   824:       uint16_t W1 = getRegBitWidth(Reg[1]);
   825:       assert(W0 == 64 && W1 == 32);
   826:       RegisterCell RC = eSXT(rc(1).cat(eIMM(0, W1)), W1);
   827:       return rr0(RC, Outputs);
   828:     }
   829:     case A2_zxtb:
   830:       return rr0(eZXT(rc(1), 8), Outputs);
   831:     case A2_zxth:
   832:       return rr0(eZXT(rc(1), 16), Outputs);
   833: 
   834:     // Saturations
   835: 
   836:     case A2_satb:
   837:       return rr0(eSXT(RegisterCell::self(0, W0).regify(Reg0), 8), Outputs);
   838:     case A2_sath:
   839:       return rr0(eSXT(RegisterCell::self(0, W0).regify(Reg0), 16), Outputs);
   840:     case A2_satub:
   841:       return rr0(eZXT(RegisterCell::self(0, W0).regify(Reg0), 8), Outputs);
   842:     case A2_satuh:
   843:       return rr0(eZXT(RegisterCell::self(0, W0).regify(Reg0), 16), Outputs);
   844: 
   845:     // Bit count:
   846: 
   847:     case S2_cl0:
   848:     case S2_cl0p:
   849:       // Always produce a 32-bit result.
   850:       return rr0(eCLB(rc(1), false/*bit*/, 32), Outputs);
   851:     case S2_cl1:
   852:     case S2_cl1p:
   853:       return rr0(eCLB(rc(1), true/*bit*/, 32), Outputs);
   854:     case S2_clb:
   855:     case S2_clbp: {
   856:       uint16_t W1 = getRegBitWidth(Reg[1]);
   857:       RegisterCell R1 = rc(1);
   858:       BT::BitValue TV = R1[W1-1];
   859:       if (TV.is(0) || TV.is(1))
   860:         return rr0(eCLB(R1, TV, 32), Outputs);
   861:       break;
   862:     }
   863:     case S2_ct0:
   864:     case S2_ct0p:
   865:       return rr0(eCTB(rc(1), false/*bit*/, 32), Outputs);
   866:     case S2_ct1:
   867:     case S2_ct1p:
   868:       return rr0(eCTB(rc(1), true/*bit*/, 32), Outputs);
   869:     case S5_popcountp:
   870:       // TODO
   871:       break;
   872: 
   873:     case C2_all8: {
   874:       RegisterCell P1 = rc(1);
   875:       bool Has0 = false, All1 = true;
   876:       for (uint16_t i = 0; i < 8/*XXX*/; ++i) {
   877:         if (!P1[i].is(1))
   878:           All1 = false;
   879:         if (!P1[i].is(0))
   880:           continue;
   881:         Has0 = true;
   882:         break;
   883:       }
   884:       if (!Has0 && !All1)
   885:         break;
   886:       RegisterCell RC(W0);
   887:       RC.fill(0, W0, (All1 ? BT::BitValue::One : BT::BitValue::Zero));
   888:       return rr0(RC, Outputs);
   889:     }
   890:     case C2_any8: {
   891:       RegisterCell P1 = rc(1);
   892:       bool Has1 = false, All0 = true;
   893:       for (uint16_t i = 0; i < 8/*XXX*/; ++i) {
   894:         if (!P1[i].is(0))
   895:           All0 = false;
   896:         if (!P1[i].is(1))
   897:           continue;
   898:         Has1 = true;
   899:         break;
   900:       }
```
- EN: It declares or implements routines such as cop, meet, rr0, getRegBitWidth, ... (9 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明或实现了 cop, meet, rr0, getRegBitWidth, ... (9 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 901-1000 / 第 901-1000 行

```cpp
   901:       if (!Has1 && !All0)
   902:         break;
   903:       RegisterCell RC(W0);
   904:       RC.fill(0, W0, (Has1 ? BT::BitValue::One : BT::BitValue::Zero));
   905:       return rr0(RC, Outputs);
   906:     }
   907:     case C2_and:
   908:       return rr0(eAND(rc(1), rc(2)), Outputs);
   909:     case C2_andn:
   910:       return rr0(eAND(rc(1), eNOT(rc(2))), Outputs);
   911:     case C2_not:
   912:       return rr0(eNOT(rc(1)), Outputs);
   913:     case C2_or:
   914:       return rr0(eORL(rc(1), rc(2)), Outputs);
   915:     case C2_orn:
   916:       return rr0(eORL(rc(1), eNOT(rc(2))), Outputs);
   917:     case C2_xor:
   918:       return rr0(eXOR(rc(1), rc(2)), Outputs);
   919:     case C4_and_and:
   920:       return rr0(eAND(rc(1), eAND(rc(2), rc(3))), Outputs);
   921:     case C4_and_andn:
   922:       return rr0(eAND(rc(1), eAND(rc(2), eNOT(rc(3)))), Outputs);
   923:     case C4_and_or:
   924:       return rr0(eAND(rc(1), eORL(rc(2), rc(3))), Outputs);
   925:     case C4_and_orn:
   926:       return rr0(eAND(rc(1), eORL(rc(2), eNOT(rc(3)))), Outputs);
   927:     case C4_or_and:
   928:       return rr0(eORL(rc(1), eAND(rc(2), rc(3))), Outputs);
   929:     case C4_or_andn:
   930:       return rr0(eORL(rc(1), eAND(rc(2), eNOT(rc(3)))), Outputs);
   931:     case C4_or_or:
   932:       return rr0(eORL(rc(1), eORL(rc(2), rc(3))), Outputs);
   933:     case C4_or_orn:
   934:       return rr0(eORL(rc(1), eORL(rc(2), eNOT(rc(3)))), Outputs);
   935:     case C2_bitsclr:
   936:     case C2_bitsclri:
   937:     case C2_bitsset:
   938:     case C4_nbitsclr:
   939:     case C4_nbitsclri:
   940:     case C4_nbitsset:
   941:       // TODO
   942:       break;
   943:     case S2_tstbit_i:
   944:     case S4_ntstbit_i: {
   945:       BT::BitValue V = rc(1)[im(2)];
   946:       if (V.is(0) || V.is(1)) {
   947:         // If instruction is S2_tstbit_i, test for 1, otherwise test for 0.
   948:         bool TV = (Opc == S2_tstbit_i);
   949:         BT::BitValue F = V.is(TV) ? BT::BitValue::One : BT::BitValue::Zero;
   950:         return rr0(RegisterCell(W0).fill(0, W0, F), Outputs);
   951:       }
   952:       break;
   953:     }
   954: 
   955:     default:
   956:       // For instructions that define a single predicate registers, store
   957:       // the low 8 bits of the register only.
   958:       if (unsigned DefR = getUniqueDefVReg(MI)) {
   959:         if (MRI.getRegClass(DefR) == &Hexagon::PredRegsRegClass) {
   960:           BT::RegisterRef PD(DefR, 0);
   961:           uint16_t RW = getRegBitWidth(PD);
   962:           uint16_t PW = 8; // XXX Pred size: getRegBitWidth(Reg[1]);
   963:           RegisterCell RC = RegisterCell::self(DefR, RW);
   964:           RC.fill(PW, RW, BT::BitValue::Zero);
   965:           putCell(PD, RC, Outputs);
   966:           return true;
   967:         }
   968:       }
   969:       return MachineEvaluator::evaluate(MI, Inputs, Outputs);
   970:   }
   971:   #undef im
   972:   #undef rc
   973:   #undef op
   974:   return false;
   975: }
   976: 
   977: bool HexagonEvaluator::evaluate(const MachineInstr &BI,
   978:                                 const CellMapType &Inputs,
   979:                                 BranchTargetList &Targets,
   980:                                 bool &FallsThru) const {
   981:   // We need to evaluate one branch at a time. TII::analyzeBranch checks
   982:   // all the branches in a basic block at once, so we cannot use it.
   983:   unsigned Opc = BI.getOpcode();
   984:   bool SimpleBranch = false;
   985:   bool Negated = false;
   986:   switch (Opc) {
   987:     case Hexagon::J2_jumpf:
   988:     case Hexagon::J2_jumpfpt:
   989:     case Hexagon::J2_jumpfnew:
   990:     case Hexagon::J2_jumpfnewpt:
   991:       Negated = true;
   992:       [[fallthrough]];
   993:     case Hexagon::J2_jumpt:
   994:     case Hexagon::J2_jumptpt:
   995:     case Hexagon::J2_jumptnew:
   996:     case Hexagon::J2_jumptnewpt:
   997:       // Simple branch:  if([!]Pn) jump ...
   998:       // i.e. Op0 = predicate, Op1 = branch target.
   999:       SimpleBranch = true;
  1000:       break;
```
- EN: It declares or implements routines such as RC, fill, rr0, PD, ... (10 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonEvaluator, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 RC, fill, rr0, PD, ... (10 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonEvaluator，说明了它与同级后端组件的连接关系。

### Lines 1001-1100 / 第 1001-1100 行

```cpp
  1001:     case Hexagon::J2_jump:
  1002:       Targets.insert(BI.getOperand(0).getMBB());
  1003:       FallsThru = false;
  1004:       return true;
  1005:     default:
  1006:       // If the branch is of unknown type, assume that all successors are
  1007:       // executable.
  1008:       return false;
  1009:   }
  1010: 
  1011:   if (!SimpleBranch)
  1012:     return false;
  1013: 
  1014:   // BI is a conditional branch if we got here.
  1015:   RegisterRef PR = BI.getOperand(0);
  1016:   RegisterCell PC = getCell(PR, Inputs);
  1017:   const BT::BitValue &Test = PC[0];
  1018: 
  1019:   // If the condition is neither true nor false, then it's unknown.
  1020:   if (!Test.is(0) && !Test.is(1))
  1021:     return false;
  1022: 
  1023:   // "Test.is(!Negated)" means "branch condition is true".
  1024:   if (!Test.is(!Negated)) {
  1025:     // Condition known to be false.
  1026:     FallsThru = true;
  1027:     return true;
  1028:   }
  1029: 
  1030:   Targets.insert(BI.getOperand(1).getMBB());
  1031:   FallsThru = false;
  1032:   return true;
  1033: }
  1034: 
  1035: unsigned HexagonEvaluator::getUniqueDefVReg(const MachineInstr &MI) const {
  1036:   unsigned DefReg = 0;
  1037:   for (const MachineOperand &Op : MI.operands()) {
  1038:     if (!Op.isReg() || !Op.isDef())
  1039:       continue;
  1040:     Register R = Op.getReg();
  1041:     if (!R.isVirtual())
  1042:       continue;
  1043:     if (DefReg != 0)
  1044:       return 0;
  1045:     DefReg = R;
  1046:   }
  1047:   return DefReg;
  1048: }
  1049: 
  1050: bool HexagonEvaluator::evaluateLoad(const MachineInstr &MI,
  1051:                                     const CellMapType &Inputs,
  1052:                                     CellMapType &Outputs) const {
  1053:   using namespace Hexagon;
  1054: 
  1055:   if (TII.isPredicated(MI))
  1056:     return false;
  1057:   assert(MI.mayLoad() && "A load that mayn't?");
  1058:   unsigned Opc = MI.getOpcode();
  1059: 
  1060:   uint16_t BitNum;
  1061:   bool SignEx;
  1062: 
  1063:   switch (Opc) {
  1064:     default:
  1065:       return false;
  1066: 
  1067: #if 0
  1068:     // memb_fifo
  1069:     case L2_loadalignb_pbr:
  1070:     case L2_loadalignb_pcr:
  1071:     case L2_loadalignb_pi:
  1072:     // memh_fifo
  1073:     case L2_loadalignh_pbr:
  1074:     case L2_loadalignh_pcr:
  1075:     case L2_loadalignh_pi:
  1076:     // membh
  1077:     case L2_loadbsw2_pbr:
  1078:     case L2_loadbsw2_pci:
  1079:     case L2_loadbsw2_pcr:
  1080:     case L2_loadbsw2_pi:
  1081:     case L2_loadbsw4_pbr:
  1082:     case L2_loadbsw4_pci:
  1083:     case L2_loadbsw4_pcr:
  1084:     case L2_loadbsw4_pi:
  1085:     // memubh
  1086:     case L2_loadbzw2_pbr:
  1087:     case L2_loadbzw2_pci:
  1088:     case L2_loadbzw2_pcr:
  1089:     case L2_loadbzw2_pi:
  1090:     case L2_loadbzw4_pbr:
  1091:     case L2_loadbzw4_pci:
  1092:     case L2_loadbzw4_pcr:
  1093:     case L2_loadbzw4_pi:
  1094: #endif
  1095: 
  1096:     case L2_loadrbgp:
  1097:     case L2_loadrb_io:
  1098:     case L2_loadrb_pbr:
  1099:     case L2_loadrb_pci:
  1100:     case L2_loadrb_pcr:
```
- EN: It opens namespaces (Hexagon) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as insert, getOperand, getCell, is, ... (9 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里打开了命名空间（Hexagon），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 insert, getOperand, getCell, is, ... (9 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 1101-1200 / 第 1101-1200 行

```cpp
  1101:     case L2_loadrb_pi:
  1102:     case PS_loadrbabs:
  1103:     case L4_loadrb_ap:
  1104:     case L4_loadrb_rr:
  1105:     case L4_loadrb_ur:
  1106:       BitNum = 8;
  1107:       SignEx = true;
  1108:       break;
  1109: 
  1110:     case L2_loadrubgp:
  1111:     case L2_loadrub_io:
  1112:     case L2_loadrub_pbr:
  1113:     case L2_loadrub_pci:
  1114:     case L2_loadrub_pcr:
  1115:     case L2_loadrub_pi:
  1116:     case PS_loadrubabs:
  1117:     case L4_loadrub_ap:
  1118:     case L4_loadrub_rr:
  1119:     case L4_loadrub_ur:
  1120:       BitNum = 8;
  1121:       SignEx = false;
  1122:       break;
  1123: 
  1124:     case L2_loadrhgp:
  1125:     case L2_loadrh_io:
  1126:     case L2_loadrh_pbr:
  1127:     case L2_loadrh_pci:
  1128:     case L2_loadrh_pcr:
  1129:     case L2_loadrh_pi:
  1130:     case PS_loadrhabs:
  1131:     case L4_loadrh_ap:
  1132:     case L4_loadrh_rr:
  1133:     case L4_loadrh_ur:
  1134:       BitNum = 16;
  1135:       SignEx = true;
  1136:       break;
  1137: 
  1138:     case L2_loadruhgp:
  1139:     case L2_loadruh_io:
  1140:     case L2_loadruh_pbr:
  1141:     case L2_loadruh_pci:
  1142:     case L2_loadruh_pcr:
  1143:     case L2_loadruh_pi:
  1144:     case L4_loadruh_rr:
  1145:     case PS_loadruhabs:
  1146:     case L4_loadruh_ap:
  1147:     case L4_loadruh_ur:
  1148:       BitNum = 16;
  1149:       SignEx = false;
  1150:       break;
  1151: 
  1152:     case L2_loadrigp:
  1153:     case L2_loadri_io:
  1154:     case L2_loadri_pbr:
  1155:     case L2_loadri_pci:
  1156:     case L2_loadri_pcr:
  1157:     case L2_loadri_pi:
  1158:     case L2_loadw_locked:
  1159:     case PS_loadriabs:
  1160:     case L4_loadri_ap:
  1161:     case L4_loadri_rr:
  1162:     case L4_loadri_ur:
  1163:     case LDriw_pred:
  1164:       BitNum = 32;
  1165:       SignEx = true;
  1166:       break;
  1167: 
  1168:     case L2_loadrdgp:
  1169:     case L2_loadrd_io:
  1170:     case L2_loadrd_pbr:
  1171:     case L2_loadrd_pci:
  1172:     case L2_loadrd_pcr:
  1173:     case L2_loadrd_pi:
  1174:     case L4_loadd_locked:
  1175:     case PS_loadrdabs:
  1176:     case L4_loadrd_ap:
  1177:     case L4_loadrd_rr:
  1178:     case L4_loadrd_ur:
  1179:       BitNum = 64;
  1180:       SignEx = true;
  1181:       break;
  1182:   }
  1183: 
  1184:   const MachineOperand &MD = MI.getOperand(0);
  1185:   assert(MD.isReg() && MD.isDef());
  1186:   RegisterRef RD = MD;
  1187: 
  1188:   uint16_t W = getRegBitWidth(RD);
  1189:   assert(W >= BitNum && BitNum > 0);
  1190:   RegisterCell Res(W);
  1191: 
  1192:   for (uint16_t i = 0; i < BitNum; ++i)
  1193:     Res[i] = BT::BitValue::self(BT::BitRef(RD.Reg, i));
  1194: 
  1195:   if (SignEx) {
  1196:     const BT::BitValue &Sign = Res[BitNum-1];
  1197:     for (uint16_t i = BitNum; i < W; ++i)
  1198:       Res[i] = BT::BitValue::ref(Sign);
  1199:   } else {
  1200:     for (uint16_t i = BitNum; i < W; ++i)
```
- EN: It declares or implements routines such as getOperand, assert, getRegBitWidth, Res, ... (6 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明或实现了 getOperand, assert, getRegBitWidth, Res, ... (6 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 1201-1286 / 第 1201-1286 行

```cpp
  1201:       Res[i] = BT::BitValue::Zero;
  1202:   }
  1203: 
  1204:   putCell(RD, Res, Outputs);
  1205:   return true;
  1206: }
  1207: 
  1208: bool HexagonEvaluator::evaluateFormalCopy(const MachineInstr &MI,
  1209:                                           const CellMapType &Inputs,
  1210:                                           CellMapType &Outputs) const {
  1211:   // If MI defines a formal parameter, but is not a copy (loads are handled
  1212:   // in evaluateLoad), then it's not clear what to do.
  1213:   assert(MI.isCopy());
  1214: 
  1215:   RegisterRef RD = MI.getOperand(0);
  1216:   RegisterRef RS = MI.getOperand(1);
  1217:   assert(RD.Sub == 0);
  1218:   if (!RS.Reg.isPhysical())
  1219:     return false;
  1220:   RegExtMap::const_iterator F = VRX.find(RD.Reg);
  1221:   if (F == VRX.end())
  1222:     return false;
  1223: 
  1224:   uint16_t EW = F->second.Width;
  1225:   // Store RD's cell into the map. This will associate the cell with a virtual
  1226:   // register, and make zero-/sign-extends possible (otherwise we would be ex-
  1227:   // tending "self" bit values, which will have no effect, since "self" values
  1228:   // cannot be references to anything).
  1229:   putCell(RD, getCell(RS, Inputs), Outputs);
  1230: 
  1231:   RegisterCell Res;
  1232:   // Read RD's cell from the outputs instead of RS's cell from the inputs:
  1233:   if (F->second.Type == ExtType::SExt)
  1234:     Res = eSXT(getCell(RD, Outputs), EW);
  1235:   else if (F->second.Type == ExtType::ZExt)
  1236:     Res = eZXT(getCell(RD, Outputs), EW);
  1237: 
  1238:   putCell(RD, Res, Outputs);
  1239:   return true;
  1240: }
  1241: 
  1242: unsigned HexagonEvaluator::getNextPhysReg(unsigned PReg, unsigned Width) const {
  1243:   using namespace Hexagon;
  1244: 
  1245:   bool Is64 = DoubleRegsRegClass.contains(PReg);
  1246:   assert(PReg == 0 || Is64 || IntRegsRegClass.contains(PReg));
  1247: 
  1248:   static const unsigned Phys32[] = { R0, R1, R2, R3, R4, R5 };
  1249:   static const unsigned Phys64[] = { D0, D1, D2 };
  1250:   const unsigned Num32 = sizeof(Phys32)/sizeof(unsigned);
  1251:   const unsigned Num64 = sizeof(Phys64)/sizeof(unsigned);
  1252: 
  1253:   // Return the first parameter register of the required width.
  1254:   if (PReg == 0)
  1255:     return (Width <= 32) ? Phys32[0] : Phys64[0];
  1256: 
  1257:   // Set Idx32, Idx64 in such a way that Idx+1 would give the index of the
  1258:   // next register.
  1259:   unsigned Idx32 = 0, Idx64 = 0;
  1260:   if (!Is64) {
  1261:     while (Idx32 < Num32) {
  1262:       if (Phys32[Idx32] == PReg)
  1263:         break;
  1264:       Idx32++;
  1265:     }
  1266:     Idx64 = Idx32/2;
  1267:   } else {
  1268:     while (Idx64 < Num64) {
  1269:       if (Phys64[Idx64] == PReg)
  1270:         break;
  1271:       Idx64++;
  1272:     }
  1273:     Idx32 = Idx64*2+1;
  1274:   }
  1275: 
  1276:   if (Width <= 32)
  1277:     return (Idx32+1 < Num32) ? Phys32[Idx32+1] : 0;
  1278:   return (Idx64+1 < Num64) ? Phys64[Idx64+1] : 0;
  1279: }
  1280: 
  1281: unsigned HexagonEvaluator::getVirtRegFor(unsigned PReg) const {
  1282:   for (std::pair<MCRegister, Register> P : MRI.liveins())
  1283:     if (P.first == PReg)
  1284:       return P.second;
  1285:   return 0;
  1286: }
```
- EN: It opens namespaces (Hexagon) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as putCell, HexagonEvaluator::evaluateFormalCopy, copy, getOperand, ... (10 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里打开了命名空间（Hexagon），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 putCell, HexagonEvaluator::evaluateFormalCopy, copy, getOperand, ... (10 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

## Key Concepts / 关键概念

- register modeling / 寄存器建模
- instruction semantics / 指令语义
- subtarget feature gating / 子目标特性控制
- ABI and stack-frame lowering / ABI 与栈帧下沉

## Dependencies / 依赖关系

- Direct includes / 直接包含: `HexagonBitTracker.h, HexagonInstrInfo.h, HexagonRegisterInfo.h, HexagonSubtarget.h, llvm/CodeGen/MachineFrameInfo.h, llvm/CodeGen/MachineFunction.h, llvm/CodeGen/MachineInstr.h, llvm/CodeGen/MachineOperand.h, llvm/CodeGen/MachineRegisterInfo.h, llvm/IR/Argument.h, ... (16 total)`
- Hexagon symbols / Hexagon 符号: `HexagonBitTracker, HexagonInstrInfo, HexagonRegisterInfo, HexagonSubtarget, HexagonEvaluator`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
