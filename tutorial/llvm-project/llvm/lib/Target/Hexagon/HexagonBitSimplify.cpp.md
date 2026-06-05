# HexagonBitSimplify.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonBitSimplify.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Implements Hexagon bit-level peephole simplification.
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-200 / 第 1-200 行

```cpp
     1: //===- HexagonBitSimplify.cpp ---------------------------------------------===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: 
     9: #include "BitTracker.h"
    10: #include "Hexagon.h"
    11: #include "HexagonBitTracker.h"
    12: #include "HexagonInstrInfo.h"
    13: #include "HexagonRegisterInfo.h"
    14: #include "HexagonSubtarget.h"
    15: #include "llvm/ADT/BitVector.h"
    16: #include "llvm/ADT/SmallVector.h"
    17: #include "llvm/ADT/StringRef.h"
    18: #include "llvm/CodeGen/MachineBasicBlock.h"
    19: #include "llvm/CodeGen/MachineDominators.h"
    20: #include "llvm/CodeGen/MachineFunction.h"
    21: #include "llvm/CodeGen/MachineFunctionPass.h"
    22: #include "llvm/CodeGen/MachineInstr.h"
    23: #include "llvm/CodeGen/MachineOperand.h"
    24: #include "llvm/CodeGen/MachineRegisterInfo.h"
    25: #include "llvm/CodeGen/TargetRegisterInfo.h"
    26: #include "llvm/IR/DebugLoc.h"
    27: #include "llvm/InitializePasses.h"
    28: #include "llvm/MC/MCInstrDesc.h"
    29: #include "llvm/Pass.h"
    30: #include "llvm/Support/Debug.h"
    31: #include "llvm/Support/raw_ostream.h"
    32: 
    33: #define DEBUG_TYPE "hexbit"
    34: 
    35: using namespace llvm;
    36: 
    37: static cl::opt<bool> PreserveTiedOps("hexbit-keep-tied", cl::Hidden,
    38:   cl::init(true), cl::desc("Preserve subregisters in tied operands"));
    39: static cl::opt<bool> GenExtract("hexbit-extract", cl::Hidden,
    40:   cl::init(true), cl::desc("Generate extract instructions"));
    41: static cl::opt<bool> GenBitSplit("hexbit-bitsplit", cl::Hidden,
    42:   cl::init(true), cl::desc("Generate bitsplit instructions"));
    43: 
    44: static cl::opt<unsigned> MaxExtract("hexbit-max-extract", cl::Hidden,
    45:   cl::init(std::numeric_limits<unsigned>::max()));
    46: static unsigned CountExtract = 0;
    47: static cl::opt<unsigned> MaxBitSplit("hexbit-max-bitsplit", cl::Hidden,
    48:   cl::init(std::numeric_limits<unsigned>::max()));
    49: static unsigned CountBitSplit = 0;
    50: 
    51: static cl::opt<unsigned> RegisterSetLimit("hexbit-registerset-limit",
    52:   cl::Hidden, cl::init(1000));
    53: 
    54: namespace {
    55: 
    56:   // Set of virtual registers, based on BitVector.
    57:   struct RegisterSet {
    58:     RegisterSet() = default;
    59:     explicit RegisterSet(unsigned s, bool t = false) : Bits(s, t) {}
    60:     RegisterSet(const RegisterSet &RS) = default;
    61: 
    62:     void clear() {
    63:       Bits.clear();
    64:       LRU.clear();
    65:     }
    66: 
    67:     unsigned count() const {
    68:       return Bits.count();
    69:     }
    70: 
    71:     unsigned find_first() const {
    72:       int First = Bits.find_first();
    73:       if (First < 0)
    74:         return 0;
    75:       return x2v(First);
    76:     }
    77: 
    78:     unsigned find_next(unsigned Prev) const {
    79:       int Next = Bits.find_next(v2x(Prev));
    80:       if (Next < 0)
    81:         return 0;
    82:       return x2v(Next);
    83:     }
    84: 
    85:     RegisterSet &insert(unsigned R) {
    86:       unsigned Idx = v2x(R);
    87:       ensure(Idx);
    88:       bool Exists = Bits.test(Idx);
    89:       Bits.set(Idx);
    90:       if (!Exists) {
    91:         LRU.push_back(Idx);
    92:         if (LRU.size() > RegisterSetLimit) {
    93:           unsigned T = LRU.front();
    94:           Bits.reset(T);
    95:           LRU.pop_front();
    96:         }
    97:       }
    98:       return *this;
    99:     }
   100:     RegisterSet &remove(unsigned R) {
   101:       unsigned Idx = v2x(R);
   102:       if (Idx < Bits.size()) {
   103:         bool Exists = Bits.test(Idx);
   104:         Bits.reset(Idx);
   105:         if (Exists) {
   106:           auto F = llvm::find(LRU, Idx);
   107:           assert(F != LRU.end());
   108:           LRU.erase(F);
   109:         }
   110:       }
   111:       return *this;
   112:     }
   113: 
   114:     RegisterSet &insert(const RegisterSet &Rs) {
   115:       for (unsigned R = Rs.find_first(); R; R = Rs.find_next(R))
   116:         insert(R);
   117:       return *this;
   118:     }
   119:     RegisterSet &remove(const RegisterSet &Rs) {
   120:       for (unsigned R = Rs.find_first(); R; R = Rs.find_next(R))
   121:         remove(R);
   122:       return *this;
   123:     }
   124: 
   125:     bool operator[](unsigned R) const {
   126:       unsigned Idx = v2x(R);
   127:       return Idx < Bits.size() ? Bits[Idx] : false;
   128:     }
   129:     bool has(unsigned R) const {
   130:       unsigned Idx = v2x(R);
   131:       if (Idx >= Bits.size())
   132:         return false;
   133:       return Bits.test(Idx);
   134:     }
   135: 
   136:     bool empty() const {
   137:       return !Bits.any();
   138:     }
   139:     bool includes(const RegisterSet &Rs) const {
   140:       return Rs.Bits.subsetOf(Bits);
   141:     }
   142:     bool intersects(const RegisterSet &Rs) const {
   143:       return Bits.anyCommon(Rs.Bits);
   144:     }
   145: 
   146:   private:
   147:     BitVector Bits;
   148:     std::deque<unsigned> LRU;
   149: 
   150:     void ensure(unsigned Idx) {
   151:       if (Bits.size() <= Idx)
   152:         Bits.resize(std::max(Idx+1, 32U));
   153:     }
   154: 
   155:     static inline unsigned v2x(unsigned v) {
   156:       return Register(v).virtRegIndex();
   157:     }
   158: 
   159:     static inline unsigned x2v(unsigned x) {
   160:       return Register::index2VirtReg(x);
   161:     }
   162:   };
   163: 
   164:   struct PrintRegSet {
   165:     PrintRegSet(const RegisterSet &S, const TargetRegisterInfo *RI)
   166:       : RS(S), TRI(RI) {}
   167: 
   168:     friend raw_ostream &operator<< (raw_ostream &OS,
   169:           const PrintRegSet &P);
   170: 
   171:   private:
   172:     const RegisterSet &RS;
   173:     const TargetRegisterInfo *TRI;
   174:   };
   175: 
   176:   [[maybe_unused]] raw_ostream &operator<<(raw_ostream &OS,
   177:                                            const PrintRegSet &P);
   178:   raw_ostream &operator<< (raw_ostream &OS, const PrintRegSet &P) {
   179:     OS << '{';
   180:     for (unsigned R = P.RS.find_first(); R; R = P.RS.find_next(R))
   181:       OS << ' ' << printReg(R, P.TRI);
   182:     OS << " }";
   183:     return OS;
   184:   }
   185: 
   186:   class Transformation;
   187: 
   188:   class HexagonBitSimplify : public MachineFunctionPass {
   189:   public:
   190:     static char ID;
   191: 
   192:     HexagonBitSimplify() : MachineFunctionPass(ID) {}
   193: 
   194:     StringRef getPassName() const override {
   195:       return "Hexagon bit simplification";
   196:     }
   197: 
   198:     void getAnalysisUsage(AnalysisUsage &AU) const override {
   199:       AU.addRequired<MachineDominatorTreeWrapperPass>();
   200:       AU.addPreserved<MachineDominatorTreeWrapperPass>();
```
- EN: It imports headers such as BitTracker.h, Hexagon.h, HexagonBitTracker.h, HexagonInstrInfo.h, ... (23 total), establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as RegisterSet, PrintRegSet, Transformation, HexagonBitSimplify, which carry the state or API of this component.
- CN: 这里引入了 BitTracker.h, Hexagon.h, HexagonBitTracker.h, HexagonInstrInfo.h, ... (23 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 RegisterSet, PrintRegSet, Transformation, HexagonBitSimplify 等类型，用来承载该组件的状态或接口。

### Lines 201-400 / 第 201-400 行

```cpp
   201:       MachineFunctionPass::getAnalysisUsage(AU);
   202:     }
   203: 
   204:     bool runOnMachineFunction(MachineFunction &MF) override;
   205: 
   206:     static void getInstrDefs(const MachineInstr &MI, RegisterSet &Defs);
   207:     static void getInstrUses(const MachineInstr &MI, RegisterSet &Uses);
   208:     static bool isEqual(const BitTracker::RegisterCell &RC1, uint16_t B1,
   209:         const BitTracker::RegisterCell &RC2, uint16_t B2, uint16_t W);
   210:     static bool isZero(const BitTracker::RegisterCell &RC, uint16_t B,
   211:         uint16_t W);
   212:     static bool getConst(const BitTracker::RegisterCell &RC, uint16_t B,
   213:         uint16_t W, uint64_t &U);
   214:     static bool replaceReg(Register OldR, Register NewR,
   215:                            MachineRegisterInfo &MRI);
   216:     static bool getSubregMask(const BitTracker::RegisterRef &RR,
   217:         unsigned &Begin, unsigned &Width, MachineRegisterInfo &MRI);
   218:     static bool replaceRegWithSub(Register OldR, Register NewR, unsigned NewSR,
   219:                                   MachineRegisterInfo &MRI);
   220:     static bool replaceSubWithSub(Register OldR, unsigned OldSR, Register NewR,
   221:                                   unsigned NewSR, MachineRegisterInfo &MRI);
   222:     static bool parseRegSequence(const MachineInstr &I,
   223:         BitTracker::RegisterRef &SL, BitTracker::RegisterRef &SH,
   224:         const MachineRegisterInfo &MRI);
   225: 
   226:     static bool getUsedBitsInStore(unsigned Opc, BitVector &Bits,
   227:         uint16_t Begin);
   228:     static bool getUsedBits(unsigned Opc, unsigned OpN, BitVector &Bits,
   229:         uint16_t Begin, const HexagonInstrInfo &HII);
   230: 
   231:     static const TargetRegisterClass *getFinalVRegClass(
   232:         const BitTracker::RegisterRef &RR, MachineRegisterInfo &MRI);
   233:     static bool isTransparentCopy(const BitTracker::RegisterRef &RD,
   234:         const BitTracker::RegisterRef &RS, MachineRegisterInfo &MRI);
   235: 
   236:   private:
   237:     MachineDominatorTree *MDT = nullptr;
   238: 
   239:     bool visitBlock(MachineBasicBlock &B, Transformation &T, RegisterSet &AVs);
   240:     static bool hasTiedUse(unsigned Reg, MachineRegisterInfo &MRI,
   241:         unsigned NewSub = Hexagon::NoSubRegister);
   242:   };
   243: 
   244:   using HBS = HexagonBitSimplify;
   245: 
   246:   // The purpose of this class is to provide a common facility to traverse
   247:   // the function top-down or bottom-up via the dominator tree, and keep
   248:   // track of the available registers.
   249:   class Transformation {
   250:   public:
   251:     bool TopDown;
   252: 
   253:     Transformation(bool TD) : TopDown(TD) {}
   254:     virtual ~Transformation() = default;
   255: 
   256:     virtual bool processBlock(MachineBasicBlock &B, const RegisterSet &AVs) = 0;
   257:   };
   258: 
   259: } // end anonymous namespace
   260: 
   261: char HexagonBitSimplify::ID = 0;
   262: 
   263: INITIALIZE_PASS_BEGIN(HexagonBitSimplify, "hexagon-bit-simplify",
   264:       "Hexagon bit simplification", false, false)
   265: INITIALIZE_PASS_DEPENDENCY(MachineDominatorTreeWrapperPass)
   266: INITIALIZE_PASS_END(HexagonBitSimplify, "hexagon-bit-simplify",
   267:       "Hexagon bit simplification", false, false)
   268: 
   269: bool HexagonBitSimplify::visitBlock(MachineBasicBlock &B, Transformation &T,
   270:       RegisterSet &AVs) {
   271:   bool Changed = false;
   272: 
   273:   if (T.TopDown)
   274:     Changed = T.processBlock(B, AVs);
   275: 
   276:   RegisterSet Defs;
   277:   for (auto &I : B)
   278:     getInstrDefs(I, Defs);
   279:   RegisterSet NewAVs = AVs;
   280:   NewAVs.insert(Defs);
   281: 
   282:   for (auto *DTN : children<MachineDomTreeNode*>(MDT->getNode(&B)))
   283:     Changed |= visitBlock(*(DTN->getBlock()), T, NewAVs);
   284: 
   285:   if (!T.TopDown)
   286:     Changed |= T.processBlock(B, AVs);
   287: 
   288:   return Changed;
   289: }
   290: 
   291: //
   292: // Utility functions:
   293: //
   294: void HexagonBitSimplify::getInstrDefs(const MachineInstr &MI,
   295:       RegisterSet &Defs) {
   296:   for (auto &Op : MI.operands()) {
   297:     if (!Op.isReg() || !Op.isDef())
   298:       continue;
   299:     Register R = Op.getReg();
   300:     if (!R.isVirtual())
   301:       continue;
   302:     Defs.insert(R);
   303:   }
   304: }
   305: 
   306: void HexagonBitSimplify::getInstrUses(const MachineInstr &MI,
   307:       RegisterSet &Uses) {
   308:   for (auto &Op : MI.operands()) {
   309:     if (!Op.isReg() || !Op.isUse())
   310:       continue;
   311:     Register R = Op.getReg();
   312:     if (!R.isVirtual())
   313:       continue;
   314:     Uses.insert(R);
   315:   }
   316: }
   317: 
   318: // Check if all the bits in range [B, E) in both cells are equal.
   319: bool HexagonBitSimplify::isEqual(const BitTracker::RegisterCell &RC1,
   320:       uint16_t B1, const BitTracker::RegisterCell &RC2, uint16_t B2,
   321:       uint16_t W) {
   322:   for (uint16_t i = 0; i < W; ++i) {
   323:     // If RC1[i] is "bottom", it cannot be proven equal to RC2[i].
   324:     if (RC1[B1+i].Type == BitTracker::BitValue::Ref && RC1[B1+i].RefI.Reg == 0)
   325:       return false;
   326:     // Same for RC2[i].
   327:     if (RC2[B2+i].Type == BitTracker::BitValue::Ref && RC2[B2+i].RefI.Reg == 0)
   328:       return false;
   329:     if (RC1[B1+i] != RC2[B2+i])
   330:       return false;
   331:   }
   332:   return true;
   333: }
   334: 
   335: bool HexagonBitSimplify::isZero(const BitTracker::RegisterCell &RC,
   336:       uint16_t B, uint16_t W) {
   337:   assert(B < RC.width() && B+W <= RC.width());
   338:   for (uint16_t i = B; i < B+W; ++i)
   339:     if (!RC[i].is(0))
   340:       return false;
   341:   return true;
   342: }
   343: 
   344: bool HexagonBitSimplify::getConst(const BitTracker::RegisterCell &RC,
   345:         uint16_t B, uint16_t W, uint64_t &U) {
   346:   assert(B < RC.width() && B+W <= RC.width());
   347:   int64_t T = 0;
   348:   for (uint16_t i = B+W; i > B; --i) {
   349:     const BitTracker::BitValue &BV = RC[i-1];
   350:     T <<= 1;
   351:     if (BV.is(1))
   352:       T |= 1;
   353:     else if (!BV.is(0))
   354:       return false;
   355:   }
   356:   U = T;
   357:   return true;
   358: }
   359: 
   360: bool HexagonBitSimplify::replaceReg(Register OldR, Register NewR,
   361:                                     MachineRegisterInfo &MRI) {
   362:   if (!OldR.isVirtual() || !NewR.isVirtual())
   363:     return false;
   364:   auto Begin = MRI.use_begin(OldR), End = MRI.use_end();
   365:   decltype(End) NextI;
   366:   for (auto I = Begin; I != End; I = NextI) {
   367:     NextI = std::next(I);
   368:     I->setReg(NewR);
   369:   }
   370:   return Begin != End;
   371: }
   372: 
   373: bool HexagonBitSimplify::replaceRegWithSub(Register OldR, Register NewR,
   374:                                            unsigned NewSR,
   375:                                            MachineRegisterInfo &MRI) {
   376:   if (!OldR.isVirtual() || !NewR.isVirtual())
   377:     return false;
   378:   if (hasTiedUse(OldR, MRI, NewSR))
   379:     return false;
   380:   auto Begin = MRI.use_begin(OldR), End = MRI.use_end();
   381:   decltype(End) NextI;
   382:   for (auto I = Begin; I != End; I = NextI) {
   383:     NextI = std::next(I);
   384:     I->setReg(NewR);
   385:     I->setSubReg(NewSR);
   386:   }
   387:   return Begin != End;
   388: }
   389: 
   390: bool HexagonBitSimplify::replaceSubWithSub(Register OldR, unsigned OldSR,
   391:                                            Register NewR, unsigned NewSR,
   392:                                            MachineRegisterInfo &MRI) {
   393:   if (!OldR.isVirtual() || !NewR.isVirtual())
   394:     return false;
   395:   if (OldSR != NewSR && hasTiedUse(OldR, MRI, NewSR))
   396:     return false;
   397:   auto Begin = MRI.use_begin(OldR), End = MRI.use_end();
   398:   decltype(End) NextI;
   399:   for (auto I = Begin; I != End; I = NextI) {
   400:     NextI = std::next(I);
```
- EN: It opens namespaces (char) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as Transformation, which carry the state or API of this component. It defines declarative TableGen records like Transformation; these records are consumed by TableGen instead of executed directly. It declares or implements routines such as MachineFunctionPass::getAnalysisUsage, runOnMachineFunction, getInstrDefs, getInstrUses, ... (35 total), translating Hexagon-specific policy into reusable code paths.
- CN: 这里打开了命名空间（char），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 Transformation 等类型，用来承载该组件的状态或接口。 这里定义了 Transformation 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里声明或实现了 MachineFunctionPass::getAnalysisUsage, runOnMachineFunction, getInstrDefs, getInstrUses, ... (35 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

### Lines 401-600 / 第 401-600 行

```cpp
   401:     if (I->getSubReg() != OldSR)
   402:       continue;
   403:     I->setReg(NewR);
   404:     I->setSubReg(NewSR);
   405:   }
   406:   return Begin != End;
   407: }
   408: 
   409: // For a register ref (pair Reg:Sub), set Begin to the position of the LSB
   410: // of Sub in Reg, and set Width to the size of Sub in bits. Return true,
   411: // if this succeeded, otherwise return false.
   412: bool HexagonBitSimplify::getSubregMask(const BitTracker::RegisterRef &RR,
   413:       unsigned &Begin, unsigned &Width, MachineRegisterInfo &MRI) {
   414:   const TargetRegisterClass *RC = MRI.getRegClass(RR.Reg);
   415:   if (RR.Sub == 0) {
   416:     Begin = 0;
   417:     Width = MRI.getTargetRegisterInfo()->getRegSizeInBits(*RC);
   418:     return true;
   419:   }
   420: 
   421:   Begin = 0;
   422: 
   423:   switch (RC->getID()) {
   424:     case Hexagon::DoubleRegsRegClassID:
   425:     case Hexagon::HvxWRRegClassID:
   426:       Width = MRI.getTargetRegisterInfo()->getRegSizeInBits(*RC) / 2;
   427:       if (RR.Sub == Hexagon::isub_hi || RR.Sub == Hexagon::vsub_hi)
   428:         Begin = Width;
   429:       break;
   430:     default:
   431:       return false;
   432:   }
   433:   return true;
   434: }
   435: 
   436: 
   437: // For a REG_SEQUENCE, set SL to the low subregister and SH to the high
   438: // subregister.
   439: bool HexagonBitSimplify::parseRegSequence(const MachineInstr &I,
   440:       BitTracker::RegisterRef &SL, BitTracker::RegisterRef &SH,
   441:       const MachineRegisterInfo &MRI) {
   442:   assert(I.getOpcode() == TargetOpcode::REG_SEQUENCE);
   443:   unsigned Sub1 = I.getOperand(2).getImm(), Sub2 = I.getOperand(4).getImm();
   444:   auto &DstRC = *MRI.getRegClass(I.getOperand(0).getReg());
   445:   auto &HRI = static_cast<const HexagonRegisterInfo&>(
   446:                   *MRI.getTargetRegisterInfo());
   447:   unsigned SubLo = HRI.getHexagonSubRegIndex(DstRC, Hexagon::ps_sub_lo);
   448:   unsigned SubHi = HRI.getHexagonSubRegIndex(DstRC, Hexagon::ps_sub_hi);
   449:   assert((Sub1 == SubLo && Sub2 == SubHi) || (Sub1 == SubHi && Sub2 == SubLo));
   450:   if (Sub1 == SubLo && Sub2 == SubHi) {
   451:     SL = I.getOperand(1);
   452:     SH = I.getOperand(3);
   453:     return true;
   454:   }
   455:   if (Sub1 == SubHi && Sub2 == SubLo) {
   456:     SH = I.getOperand(1);
   457:     SL = I.getOperand(3);
   458:     return true;
   459:   }
   460:   return false;
   461: }
   462: 
   463: // All stores (except 64-bit stores) take a 32-bit register as the source
   464: // of the value to be stored. If the instruction stores into a location
   465: // that is shorter than 32 bits, some bits of the source register are not
   466: // used. For each store instruction, calculate the set of used bits in
   467: // the source register, and set appropriate bits in Bits. Return true if
   468: // the bits are calculated, false otherwise.
   469: bool HexagonBitSimplify::getUsedBitsInStore(unsigned Opc, BitVector &Bits,
   470:       uint16_t Begin) {
   471:   using namespace Hexagon;
   472: 
   473:   switch (Opc) {
   474:     // Store byte
   475:     case S2_storerb_io:           // memb(Rs32+#s11:0)=Rt32
   476:     case S2_storerbnew_io:        // memb(Rs32+#s11:0)=Nt8.new
   477:     case S2_pstorerbt_io:         // if (Pv4) memb(Rs32+#u6:0)=Rt32
   478:     case S2_pstorerbf_io:         // if (!Pv4) memb(Rs32+#u6:0)=Rt32
   479:     case S4_pstorerbtnew_io:      // if (Pv4.new) memb(Rs32+#u6:0)=Rt32
   480:     case S4_pstorerbfnew_io:      // if (!Pv4.new) memb(Rs32+#u6:0)=Rt32
   481:     case S2_pstorerbnewt_io:      // if (Pv4) memb(Rs32+#u6:0)=Nt8.new
   482:     case S2_pstorerbnewf_io:      // if (!Pv4) memb(Rs32+#u6:0)=Nt8.new
   483:     case S4_pstorerbnewtnew_io:   // if (Pv4.new) memb(Rs32+#u6:0)=Nt8.new
   484:     case S4_pstorerbnewfnew_io:   // if (!Pv4.new) memb(Rs32+#u6:0)=Nt8.new
   485:     case S2_storerb_pi:           // memb(Rx32++#s4:0)=Rt32
   486:     case S2_storerbnew_pi:        // memb(Rx32++#s4:0)=Nt8.new
   487:     case S2_pstorerbt_pi:         // if (Pv4) memb(Rx32++#s4:0)=Rt32
   488:     case S2_pstorerbf_pi:         // if (!Pv4) memb(Rx32++#s4:0)=Rt32
   489:     case S2_pstorerbtnew_pi:      // if (Pv4.new) memb(Rx32++#s4:0)=Rt32
   490:     case S2_pstorerbfnew_pi:      // if (!Pv4.new) memb(Rx32++#s4:0)=Rt32
   491:     case S2_pstorerbnewt_pi:      // if (Pv4) memb(Rx32++#s4:0)=Nt8.new
   492:     case S2_pstorerbnewf_pi:      // if (!Pv4) memb(Rx32++#s4:0)=Nt8.new
   493:     case S2_pstorerbnewtnew_pi:   // if (Pv4.new) memb(Rx32++#s4:0)=Nt8.new
   494:     case S2_pstorerbnewfnew_pi:   // if (!Pv4.new) memb(Rx32++#s4:0)=Nt8.new
   495:     case S4_storerb_ap:           // memb(Re32=#U6)=Rt32
   496:     case S4_storerbnew_ap:        // memb(Re32=#U6)=Nt8.new
   497:     case S2_storerb_pr:           // memb(Rx32++Mu2)=Rt32
   498:     case S2_storerbnew_pr:        // memb(Rx32++Mu2)=Nt8.new
   499:     case S4_storerb_ur:           // memb(Ru32<<#u2+#U6)=Rt32
   500:     case S4_storerbnew_ur:        // memb(Ru32<<#u2+#U6)=Nt8.new
   501:     case S2_storerb_pbr:          // memb(Rx32++Mu2:brev)=Rt32
   502:     case S2_storerbnew_pbr:       // memb(Rx32++Mu2:brev)=Nt8.new
   503:     case S2_storerb_pci:          // memb(Rx32++#s4:0:circ(Mu2))=Rt32
   504:     case S2_storerbnew_pci:       // memb(Rx32++#s4:0:circ(Mu2))=Nt8.new
   505:     case S2_storerb_pcr:          // memb(Rx32++I:circ(Mu2))=Rt32
   506:     case S2_storerbnew_pcr:       // memb(Rx32++I:circ(Mu2))=Nt8.new
   507:     case S4_storerb_rr:           // memb(Rs32+Ru32<<#u2)=Rt32
   508:     case S4_storerbnew_rr:        // memb(Rs32+Ru32<<#u2)=Nt8.new
   509:     case S4_pstorerbt_rr:         // if (Pv4) memb(Rs32+Ru32<<#u2)=Rt32
   510:     case S4_pstorerbf_rr:         // if (!Pv4) memb(Rs32+Ru32<<#u2)=Rt32
   511:     case S4_pstorerbtnew_rr:      // if (Pv4.new) memb(Rs32+Ru32<<#u2)=Rt32
   512:     case S4_pstorerbfnew_rr:      // if (!Pv4.new) memb(Rs32+Ru32<<#u2)=Rt32
   513:     case S4_pstorerbnewt_rr:      // if (Pv4) memb(Rs32+Ru32<<#u2)=Nt8.new
   514:     case S4_pstorerbnewf_rr:      // if (!Pv4) memb(Rs32+Ru32<<#u2)=Nt8.new
   515:     case S4_pstorerbnewtnew_rr:   // if (Pv4.new) memb(Rs32+Ru32<<#u2)=Nt8.new
   516:     case S4_pstorerbnewfnew_rr:   // if (!Pv4.new) memb(Rs32+Ru32<<#u2)=Nt8.new
   517:     case S2_storerbgp:            // memb(gp+#u16:0)=Rt32
   518:     case S2_storerbnewgp:         // memb(gp+#u16:0)=Nt8.new
   519:     case S4_pstorerbt_abs:        // if (Pv4) memb(#u6)=Rt32
   520:     case S4_pstorerbf_abs:        // if (!Pv4) memb(#u6)=Rt32
   521:     case S4_pstorerbtnew_abs:     // if (Pv4.new) memb(#u6)=Rt32
   522:     case S4_pstorerbfnew_abs:     // if (!Pv4.new) memb(#u6)=Rt32
   523:     case S4_pstorerbnewt_abs:     // if (Pv4) memb(#u6)=Nt8.new
   524:     case S4_pstorerbnewf_abs:     // if (!Pv4) memb(#u6)=Nt8.new
   525:     case S4_pstorerbnewtnew_abs:  // if (Pv4.new) memb(#u6)=Nt8.new
   526:     case S4_pstorerbnewfnew_abs:  // if (!Pv4.new) memb(#u6)=Nt8.new
   527:       Bits.set(Begin, Begin+8);
   528:       return true;
   529: 
   530:     // Store low half
   531:     case S2_storerh_io:           // memh(Rs32+#s11:1)=Rt32
   532:     case S2_storerhnew_io:        // memh(Rs32+#s11:1)=Nt8.new
   533:     case S2_pstorerht_io:         // if (Pv4) memh(Rs32+#u6:1)=Rt32
   534:     case S2_pstorerhf_io:         // if (!Pv4) memh(Rs32+#u6:1)=Rt32
   535:     case S4_pstorerhtnew_io:      // if (Pv4.new) memh(Rs32+#u6:1)=Rt32
   536:     case S4_pstorerhfnew_io:      // if (!Pv4.new) memh(Rs32+#u6:1)=Rt32
   537:     case S2_pstorerhnewt_io:      // if (Pv4) memh(Rs32+#u6:1)=Nt8.new
   538:     case S2_pstorerhnewf_io:      // if (!Pv4) memh(Rs32+#u6:1)=Nt8.new
   539:     case S4_pstorerhnewtnew_io:   // if (Pv4.new) memh(Rs32+#u6:1)=Nt8.new
   540:     case S4_pstorerhnewfnew_io:   // if (!Pv4.new) memh(Rs32+#u6:1)=Nt8.new
   541:     case S2_storerh_pi:           // memh(Rx32++#s4:1)=Rt32
   542:     case S2_storerhnew_pi:        // memh(Rx32++#s4:1)=Nt8.new
   543:     case S2_pstorerht_pi:         // if (Pv4) memh(Rx32++#s4:1)=Rt32
   544:     case S2_pstorerhf_pi:         // if (!Pv4) memh(Rx32++#s4:1)=Rt32
   545:     case S2_pstorerhtnew_pi:      // if (Pv4.new) memh(Rx32++#s4:1)=Rt32
   546:     case S2_pstorerhfnew_pi:      // if (!Pv4.new) memh(Rx32++#s4:1)=Rt32
   547:     case S2_pstorerhnewt_pi:      // if (Pv4) memh(Rx32++#s4:1)=Nt8.new
   548:     case S2_pstorerhnewf_pi:      // if (!Pv4) memh(Rx32++#s4:1)=Nt8.new
   549:     case S2_pstorerhnewtnew_pi:   // if (Pv4.new) memh(Rx32++#s4:1)=Nt8.new
   550:     case S2_pstorerhnewfnew_pi:   // if (!Pv4.new) memh(Rx32++#s4:1)=Nt8.new
   551:     case S4_storerh_ap:           // memh(Re32=#U6)=Rt32
   552:     case S4_storerhnew_ap:        // memh(Re32=#U6)=Nt8.new
   553:     case S2_storerh_pr:           // memh(Rx32++Mu2)=Rt32
   554:     case S2_storerhnew_pr:        // memh(Rx32++Mu2)=Nt8.new
   555:     case S4_storerh_ur:           // memh(Ru32<<#u2+#U6)=Rt32
   556:     case S4_storerhnew_ur:        // memh(Ru32<<#u2+#U6)=Nt8.new
   557:     case S2_storerh_pbr:          // memh(Rx32++Mu2:brev)=Rt32
   558:     case S2_storerhnew_pbr:       // memh(Rx32++Mu2:brev)=Nt8.new
   559:     case S2_storerh_pci:          // memh(Rx32++#s4:1:circ(Mu2))=Rt32
   560:     case S2_storerhnew_pci:       // memh(Rx32++#s4:1:circ(Mu2))=Nt8.new
   561:     case S2_storerh_pcr:          // memh(Rx32++I:circ(Mu2))=Rt32
   562:     case S2_storerhnew_pcr:       // memh(Rx32++I:circ(Mu2))=Nt8.new
   563:     case S4_storerh_rr:           // memh(Rs32+Ru32<<#u2)=Rt32
   564:     case S4_pstorerht_rr:         // if (Pv4) memh(Rs32+Ru32<<#u2)=Rt32
   565:     case S4_pstorerhf_rr:         // if (!Pv4) memh(Rs32+Ru32<<#u2)=Rt32
   566:     case S4_pstorerhtnew_rr:      // if (Pv4.new) memh(Rs32+Ru32<<#u2)=Rt32
   567:     case S4_pstorerhfnew_rr:      // if (!Pv4.new) memh(Rs32+Ru32<<#u2)=Rt32
   568:     case S4_storerhnew_rr:        // memh(Rs32+Ru32<<#u2)=Nt8.new
   569:     case S4_pstorerhnewt_rr:      // if (Pv4) memh(Rs32+Ru32<<#u2)=Nt8.new
   570:     case S4_pstorerhnewf_rr:      // if (!Pv4) memh(Rs32+Ru32<<#u2)=Nt8.new
   571:     case S4_pstorerhnewtnew_rr:   // if (Pv4.new) memh(Rs32+Ru32<<#u2)=Nt8.new
   572:     case S4_pstorerhnewfnew_rr:   // if (!Pv4.new) memh(Rs32+Ru32<<#u2)=Nt8.new
   573:     case S2_storerhgp:            // memh(gp+#u16:1)=Rt32
   574:     case S2_storerhnewgp:         // memh(gp+#u16:1)=Nt8.new
   575:     case S4_pstorerht_abs:        // if (Pv4) memh(#u6)=Rt32
   576:     case S4_pstorerhf_abs:        // if (!Pv4) memh(#u6)=Rt32
   577:     case S4_pstorerhtnew_abs:     // if (Pv4.new) memh(#u6)=Rt32
   578:     case S4_pstorerhfnew_abs:     // if (!Pv4.new) memh(#u6)=Rt32
   579:     case S4_pstorerhnewt_abs:     // if (Pv4) memh(#u6)=Nt8.new
   580:     case S4_pstorerhnewf_abs:     // if (!Pv4) memh(#u6)=Nt8.new
   581:     case S4_pstorerhnewtnew_abs:  // if (Pv4.new) memh(#u6)=Nt8.new
   582:     case S4_pstorerhnewfnew_abs:  // if (!Pv4.new) memh(#u6)=Nt8.new
   583:       Bits.set(Begin, Begin+16);
   584:       return true;
   585: 
   586:     // Store high half
   587:     case S2_storerf_io:           // memh(Rs32+#s11:1)=Rt.H32
   588:     case S2_pstorerft_io:         // if (Pv4) memh(Rs32+#u6:1)=Rt.H32
   589:     case S2_pstorerff_io:         // if (!Pv4) memh(Rs32+#u6:1)=Rt.H32
   590:     case S4_pstorerftnew_io:      // if (Pv4.new) memh(Rs32+#u6:1)=Rt.H32
   591:     case S4_pstorerffnew_io:      // if (!Pv4.new) memh(Rs32+#u6:1)=Rt.H32
   592:     case S2_storerf_pi:           // memh(Rx32++#s4:1)=Rt.H32
   593:     case S2_pstorerft_pi:         // if (Pv4) memh(Rx32++#s4:1)=Rt.H32
   594:     case S2_pstorerff_pi:         // if (!Pv4) memh(Rx32++#s4:1)=Rt.H32
   595:     case S2_pstorerftnew_pi:      // if (Pv4.new) memh(Rx32++#s4:1)=Rt.H32
   596:     case S2_pstorerffnew_pi:      // if (!Pv4.new) memh(Rx32++#s4:1)=Rt.H32
   597:     case S4_storerf_ap:           // memh(Re32=#U6)=Rt.H32
   598:     case S2_storerf_pr:           // memh(Rx32++Mu2)=Rt.H32
   599:     case S4_storerf_ur:           // memh(Ru32<<#u2+#U6)=Rt.H32
   600:     case S2_storerf_pbr:          // memh(Rx32++Mu2:brev)=Rt.H32
```
- EN: It opens namespaces (Hexagon) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as setReg, setSubReg, ref, getRegClass, ... (12 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里打开了命名空间（Hexagon），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 setReg, setSubReg, ref, getRegClass, ... (12 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 601-800 / 第 601-800 行

```cpp
   601:     case S2_storerf_pci:          // memh(Rx32++#s4:1:circ(Mu2))=Rt.H32
   602:     case S2_storerf_pcr:          // memh(Rx32++I:circ(Mu2))=Rt.H32
   603:     case S4_storerf_rr:           // memh(Rs32+Ru32<<#u2)=Rt.H32
   604:     case S4_pstorerft_rr:         // if (Pv4) memh(Rs32+Ru32<<#u2)=Rt.H32
   605:     case S4_pstorerff_rr:         // if (!Pv4) memh(Rs32+Ru32<<#u2)=Rt.H32
   606:     case S4_pstorerftnew_rr:      // if (Pv4.new) memh(Rs32+Ru32<<#u2)=Rt.H32
   607:     case S4_pstorerffnew_rr:      // if (!Pv4.new) memh(Rs32+Ru32<<#u2)=Rt.H32
   608:     case S2_storerfgp:            // memh(gp+#u16:1)=Rt.H32
   609:     case S4_pstorerft_abs:        // if (Pv4) memh(#u6)=Rt.H32
   610:     case S4_pstorerff_abs:        // if (!Pv4) memh(#u6)=Rt.H32
   611:     case S4_pstorerftnew_abs:     // if (Pv4.new) memh(#u6)=Rt.H32
   612:     case S4_pstorerffnew_abs:     // if (!Pv4.new) memh(#u6)=Rt.H32
   613:       Bits.set(Begin+16, Begin+32);
   614:       return true;
   615:   }
   616: 
   617:   return false;
   618: }
   619: 
   620: // For an instruction with opcode Opc, calculate the set of bits that it
   621: // uses in a register in operand OpN. This only calculates the set of used
   622: // bits for cases where it does not depend on any operands (as is the case
   623: // in shifts, for example). For concrete instructions from a program, the
   624: // operand may be a subregister of a larger register, while Bits would
   625: // correspond to the larger register in its entirety. Because of that,
   626: // the parameter Begin can be used to indicate which bit of Bits should be
   627: // considered the LSB of the operand.
   628: bool HexagonBitSimplify::getUsedBits(unsigned Opc, unsigned OpN,
   629:       BitVector &Bits, uint16_t Begin, const HexagonInstrInfo &HII) {
   630:   using namespace Hexagon;
   631: 
   632:   const MCInstrDesc &D = HII.get(Opc);
   633:   if (D.mayStore()) {
   634:     if (OpN == D.getNumOperands()-1)
   635:       return getUsedBitsInStore(Opc, Bits, Begin);
   636:     return false;
   637:   }
   638: 
   639:   switch (Opc) {
   640:     // One register source. Used bits: R1[0-7].
   641:     case A2_sxtb:
   642:     case A2_zxtb:
   643:     case A4_cmpbeqi:
   644:     case A4_cmpbgti:
   645:     case A4_cmpbgtui:
   646:       if (OpN == 1) {
   647:         Bits.set(Begin, Begin+8);
   648:         return true;
   649:       }
   650:       break;
   651: 
   652:     // One register source. Used bits: R1[0-15].
   653:     case A2_aslh:
   654:     case A2_sxth:
   655:     case A2_zxth:
   656:     case A4_cmpheqi:
   657:     case A4_cmphgti:
   658:     case A4_cmphgtui:
   659:       if (OpN == 1) {
   660:         Bits.set(Begin, Begin+16);
   661:         return true;
   662:       }
   663:       break;
   664: 
   665:     // One register source. Used bits: R1[16-31].
   666:     case A2_asrh:
   667:       if (OpN == 1) {
   668:         Bits.set(Begin+16, Begin+32);
   669:         return true;
   670:       }
   671:       break;
   672: 
   673:     // Two register sources. Used bits: R1[0-7], R2[0-7].
   674:     case A4_cmpbeq:
   675:     case A4_cmpbgt:
   676:     case A4_cmpbgtu:
   677:       if (OpN == 1) {
   678:         Bits.set(Begin, Begin+8);
   679:         return true;
   680:       }
   681:       break;
   682: 
   683:     // Two register sources. Used bits: R1[0-15], R2[0-15].
   684:     case A4_cmpheq:
   685:     case A4_cmphgt:
   686:     case A4_cmphgtu:
   687:     case A2_addh_h16_ll:
   688:     case A2_addh_h16_sat_ll:
   689:     case A2_addh_l16_ll:
   690:     case A2_addh_l16_sat_ll:
   691:     case A2_combine_ll:
   692:     case A2_subh_h16_ll:
   693:     case A2_subh_h16_sat_ll:
   694:     case A2_subh_l16_ll:
   695:     case A2_subh_l16_sat_ll:
   696:     case M2_mpy_acc_ll_s0:
   697:     case M2_mpy_acc_ll_s1:
   698:     case M2_mpy_acc_sat_ll_s0:
   699:     case M2_mpy_acc_sat_ll_s1:
   700:     case M2_mpy_ll_s0:
   701:     case M2_mpy_ll_s1:
   702:     case M2_mpy_nac_ll_s0:
   703:     case M2_mpy_nac_ll_s1:
   704:     case M2_mpy_nac_sat_ll_s0:
   705:     case M2_mpy_nac_sat_ll_s1:
   706:     case M2_mpy_rnd_ll_s0:
   707:     case M2_mpy_rnd_ll_s1:
   708:     case M2_mpy_sat_ll_s0:
   709:     case M2_mpy_sat_ll_s1:
   710:     case M2_mpy_sat_rnd_ll_s0:
   711:     case M2_mpy_sat_rnd_ll_s1:
   712:     case M2_mpyd_acc_ll_s0:
   713:     case M2_mpyd_acc_ll_s1:
   714:     case M2_mpyd_ll_s0:
   715:     case M2_mpyd_ll_s1:
   716:     case M2_mpyd_nac_ll_s0:
   717:     case M2_mpyd_nac_ll_s1:
   718:     case M2_mpyd_rnd_ll_s0:
   719:     case M2_mpyd_rnd_ll_s1:
   720:     case M2_mpyu_acc_ll_s0:
   721:     case M2_mpyu_acc_ll_s1:
   722:     case M2_mpyu_ll_s0:
   723:     case M2_mpyu_ll_s1:
   724:     case M2_mpyu_nac_ll_s0:
   725:     case M2_mpyu_nac_ll_s1:
   726:     case M2_mpyud_acc_ll_s0:
   727:     case M2_mpyud_acc_ll_s1:
   728:     case M2_mpyud_ll_s0:
   729:     case M2_mpyud_ll_s1:
   730:     case M2_mpyud_nac_ll_s0:
   731:     case M2_mpyud_nac_ll_s1:
   732:       if (OpN == 1 || OpN == 2) {
   733:         Bits.set(Begin, Begin+16);
   734:         return true;
   735:       }
   736:       break;
   737: 
   738:     // Two register sources. Used bits: R1[0-15], R2[16-31].
   739:     case A2_addh_h16_lh:
   740:     case A2_addh_h16_sat_lh:
   741:     case A2_combine_lh:
   742:     case A2_subh_h16_lh:
   743:     case A2_subh_h16_sat_lh:
   744:     case M2_mpy_acc_lh_s0:
   745:     case M2_mpy_acc_lh_s1:
   746:     case M2_mpy_acc_sat_lh_s0:
   747:     case M2_mpy_acc_sat_lh_s1:
   748:     case M2_mpy_lh_s0:
   749:     case M2_mpy_lh_s1:
   750:     case M2_mpy_nac_lh_s0:
   751:     case M2_mpy_nac_lh_s1:
   752:     case M2_mpy_nac_sat_lh_s0:
   753:     case M2_mpy_nac_sat_lh_s1:
   754:     case M2_mpy_rnd_lh_s0:
   755:     case M2_mpy_rnd_lh_s1:
   756:     case M2_mpy_sat_lh_s0:
   757:     case M2_mpy_sat_lh_s1:
   758:     case M2_mpy_sat_rnd_lh_s0:
   759:     case M2_mpy_sat_rnd_lh_s1:
   760:     case M2_mpyd_acc_lh_s0:
   761:     case M2_mpyd_acc_lh_s1:
   762:     case M2_mpyd_lh_s0:
   763:     case M2_mpyd_lh_s1:
   764:     case M2_mpyd_nac_lh_s0:
   765:     case M2_mpyd_nac_lh_s1:
   766:     case M2_mpyd_rnd_lh_s0:
   767:     case M2_mpyd_rnd_lh_s1:
   768:     case M2_mpyu_acc_lh_s0:
   769:     case M2_mpyu_acc_lh_s1:
   770:     case M2_mpyu_lh_s0:
   771:     case M2_mpyu_lh_s1:
   772:     case M2_mpyu_nac_lh_s0:
   773:     case M2_mpyu_nac_lh_s1:
   774:     case M2_mpyud_acc_lh_s0:
   775:     case M2_mpyud_acc_lh_s1:
   776:     case M2_mpyud_lh_s0:
   777:     case M2_mpyud_lh_s1:
   778:     case M2_mpyud_nac_lh_s0:
   779:     case M2_mpyud_nac_lh_s1:
   780:     // These four are actually LH.
   781:     case A2_addh_l16_hl:
   782:     case A2_addh_l16_sat_hl:
   783:     case A2_subh_l16_hl:
   784:     case A2_subh_l16_sat_hl:
   785:       if (OpN == 1) {
   786:         Bits.set(Begin, Begin+16);
   787:         return true;
   788:       }
   789:       if (OpN == 2) {
   790:         Bits.set(Begin+16, Begin+32);
   791:         return true;
   792:       }
   793:       break;
   794: 
   795:     // Two register sources, used bits: R1[16-31], R2[0-15].
   796:     case A2_addh_h16_hl:
   797:     case A2_addh_h16_sat_hl:
   798:     case A2_combine_hl:
   799:     case A2_subh_h16_hl:
   800:     case A2_subh_h16_sat_hl:
```
- EN: It opens namespaces (Hexagon) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as memh, operands, get, set, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonBitSimplify, HexagonInstrInfo, showing how the code connects to sibling backend components.
- CN: 这里打开了命名空间（Hexagon），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 memh, operands, get, set 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonBitSimplify, HexagonInstrInfo，说明了它与同级后端组件的连接关系。

### Lines 801-1000 / 第 801-1000 行

```cpp
   801:     case M2_mpy_acc_hl_s0:
   802:     case M2_mpy_acc_hl_s1:
   803:     case M2_mpy_acc_sat_hl_s0:
   804:     case M2_mpy_acc_sat_hl_s1:
   805:     case M2_mpy_hl_s0:
   806:     case M2_mpy_hl_s1:
   807:     case M2_mpy_nac_hl_s0:
   808:     case M2_mpy_nac_hl_s1:
   809:     case M2_mpy_nac_sat_hl_s0:
   810:     case M2_mpy_nac_sat_hl_s1:
   811:     case M2_mpy_rnd_hl_s0:
   812:     case M2_mpy_rnd_hl_s1:
   813:     case M2_mpy_sat_hl_s0:
   814:     case M2_mpy_sat_hl_s1:
   815:     case M2_mpy_sat_rnd_hl_s0:
   816:     case M2_mpy_sat_rnd_hl_s1:
   817:     case M2_mpyd_acc_hl_s0:
   818:     case M2_mpyd_acc_hl_s1:
   819:     case M2_mpyd_hl_s0:
   820:     case M2_mpyd_hl_s1:
   821:     case M2_mpyd_nac_hl_s0:
   822:     case M2_mpyd_nac_hl_s1:
   823:     case M2_mpyd_rnd_hl_s0:
   824:     case M2_mpyd_rnd_hl_s1:
   825:     case M2_mpyu_acc_hl_s0:
   826:     case M2_mpyu_acc_hl_s1:
   827:     case M2_mpyu_hl_s0:
   828:     case M2_mpyu_hl_s1:
   829:     case M2_mpyu_nac_hl_s0:
   830:     case M2_mpyu_nac_hl_s1:
   831:     case M2_mpyud_acc_hl_s0:
   832:     case M2_mpyud_acc_hl_s1:
   833:     case M2_mpyud_hl_s0:
   834:     case M2_mpyud_hl_s1:
   835:     case M2_mpyud_nac_hl_s0:
   836:     case M2_mpyud_nac_hl_s1:
   837:       if (OpN == 1) {
   838:         Bits.set(Begin+16, Begin+32);
   839:         return true;
   840:       }
   841:       if (OpN == 2) {
   842:         Bits.set(Begin, Begin+16);
   843:         return true;
   844:       }
   845:       break;
   846: 
   847:     // Two register sources, used bits: R1[16-31], R2[16-31].
   848:     case A2_addh_h16_hh:
   849:     case A2_addh_h16_sat_hh:
   850:     case A2_combine_hh:
   851:     case A2_subh_h16_hh:
   852:     case A2_subh_h16_sat_hh:
   853:     case M2_mpy_acc_hh_s0:
   854:     case M2_mpy_acc_hh_s1:
   855:     case M2_mpy_acc_sat_hh_s0:
   856:     case M2_mpy_acc_sat_hh_s1:
   857:     case M2_mpy_hh_s0:
   858:     case M2_mpy_hh_s1:
   859:     case M2_mpy_nac_hh_s0:
   860:     case M2_mpy_nac_hh_s1:
   861:     case M2_mpy_nac_sat_hh_s0:
   862:     case M2_mpy_nac_sat_hh_s1:
   863:     case M2_mpy_rnd_hh_s0:
   864:     case M2_mpy_rnd_hh_s1:
   865:     case M2_mpy_sat_hh_s0:
   866:     case M2_mpy_sat_hh_s1:
   867:     case M2_mpy_sat_rnd_hh_s0:
   868:     case M2_mpy_sat_rnd_hh_s1:
   869:     case M2_mpyd_acc_hh_s0:
   870:     case M2_mpyd_acc_hh_s1:
   871:     case M2_mpyd_hh_s0:
   872:     case M2_mpyd_hh_s1:
   873:     case M2_mpyd_nac_hh_s0:
   874:     case M2_mpyd_nac_hh_s1:
   875:     case M2_mpyd_rnd_hh_s0:
   876:     case M2_mpyd_rnd_hh_s1:
   877:     case M2_mpyu_acc_hh_s0:
   878:     case M2_mpyu_acc_hh_s1:
   879:     case M2_mpyu_hh_s0:
   880:     case M2_mpyu_hh_s1:
   881:     case M2_mpyu_nac_hh_s0:
   882:     case M2_mpyu_nac_hh_s1:
   883:     case M2_mpyud_acc_hh_s0:
   884:     case M2_mpyud_acc_hh_s1:
   885:     case M2_mpyud_hh_s0:
   886:     case M2_mpyud_hh_s1:
   887:     case M2_mpyud_nac_hh_s0:
   888:     case M2_mpyud_nac_hh_s1:
   889:       if (OpN == 1 || OpN == 2) {
   890:         Bits.set(Begin+16, Begin+32);
   891:         return true;
   892:       }
   893:       break;
   894:   }
   895: 
   896:   return false;
   897: }
   898: 
   899: // Calculate the register class that matches Reg:Sub. For example, if
   900: // %1 is a double register, then %1:isub_hi would match the "int"
   901: // register class.
   902: const TargetRegisterClass *HexagonBitSimplify::getFinalVRegClass(
   903:       const BitTracker::RegisterRef &RR, MachineRegisterInfo &MRI) {
   904:   if (!RR.Reg.isVirtual())
   905:     return nullptr;
   906:   auto *RC = MRI.getRegClass(RR.Reg);
   907:   if (RR.Sub == 0)
   908:     return RC;
   909:   auto &HRI = static_cast<const HexagonRegisterInfo&>(
   910:                   *MRI.getTargetRegisterInfo());
   911: 
   912:   auto VerifySR = [&HRI] (const TargetRegisterClass *RC, unsigned Sub) -> void {
   913:     (void)HRI;
   914:     assert(Sub == HRI.getHexagonSubRegIndex(*RC, Hexagon::ps_sub_lo) ||
   915:            Sub == HRI.getHexagonSubRegIndex(*RC, Hexagon::ps_sub_hi));
   916:   };
   917: 
   918:   switch (RC->getID()) {
   919:     case Hexagon::DoubleRegsRegClassID:
   920:       VerifySR(RC, RR.Sub);
   921:       return &Hexagon::IntRegsRegClass;
   922:     case Hexagon::HvxWRRegClassID:
   923:       VerifySR(RC, RR.Sub);
   924:       return &Hexagon::HvxVRRegClass;
   925:   }
   926:   return nullptr;
   927: }
   928: 
   929: // Check if RD could be replaced with RS at any possible use of RD.
   930: // For example a predicate register cannot be replaced with a integer
   931: // register, but a 64-bit register with a subregister can be replaced
   932: // with a 32-bit register.
   933: bool HexagonBitSimplify::isTransparentCopy(const BitTracker::RegisterRef &RD,
   934:       const BitTracker::RegisterRef &RS, MachineRegisterInfo &MRI) {
   935:   if (!RD.Reg.isVirtual() || !RS.Reg.isVirtual())
   936:     return false;
   937:   // Return false if one (or both) classes are nullptr.
   938:   auto *DRC = getFinalVRegClass(RD, MRI);
   939:   if (!DRC)
   940:     return false;
   941: 
   942:   return DRC == getFinalVRegClass(RS, MRI);
   943: }
   944: 
   945: bool HexagonBitSimplify::hasTiedUse(unsigned Reg, MachineRegisterInfo &MRI,
   946:       unsigned NewSub) {
   947:   if (!PreserveTiedOps)
   948:     return false;
   949:   return llvm::any_of(MRI.use_operands(Reg),
   950:                       [NewSub] (const MachineOperand &Op) -> bool {
   951:                         return Op.getSubReg() != NewSub && Op.isTied();
   952:                       });
   953: }
   954: 
   955: namespace {
   956: 
   957:   class DeadCodeElimination {
   958:   public:
   959:     DeadCodeElimination(MachineFunction &mf, MachineDominatorTree &mdt)
   960:       : MF(mf), HII(*MF.getSubtarget<HexagonSubtarget>().getInstrInfo()),
   961:         MDT(mdt), MRI(mf.getRegInfo()) {}
   962: 
   963:     bool run() {
   964:       return runOnNode(MDT.getRootNode());
   965:     }
   966: 
   967:   private:
   968:     bool isDead(unsigned R) const;
   969:     bool runOnNode(MachineDomTreeNode *N);
   970: 
   971:     MachineFunction &MF;
   972:     const HexagonInstrInfo &HII;
   973:     MachineDominatorTree &MDT;
   974:     MachineRegisterInfo &MRI;
   975:   };
   976: 
   977: } // end anonymous namespace
   978: 
   979: bool DeadCodeElimination::isDead(unsigned R) const {
   980:   for (const MachineOperand &MO : MRI.use_operands(R)) {
   981:     const MachineInstr *UseI = MO.getParent();
   982:     if (UseI->isDebugInstr())
   983:       continue;
   984:     if (UseI->isPHI()) {
   985:       assert(!UseI->getOperand(0).getSubReg());
   986:       Register DR = UseI->getOperand(0).getReg();
   987:       if (DR == R)
   988:         continue;
   989:     }
   990:     return false;
   991:   }
   992:   return true;
   993: }
   994: 
   995: bool DeadCodeElimination::runOnNode(MachineDomTreeNode *N) {
   996:   bool Changed = false;
   997: 
   998:   for (auto *DTN : children<MachineDomTreeNode*>(N))
   999:     Changed |= runOnNode(DTN);
  1000: 
```
- EN: It opens namespaces (bool) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as DeadCodeElimination, which carry the state or API of this component. It defines declarative TableGen records like DeadCodeElimination; these records are consumed by TableGen instead of executed directly. It declares or implements routines such as set, HexagonBitSimplify::getFinalVRegClass, getRegClass, getTargetRegisterInfo, ... (19 total), translating Hexagon-specific policy into reusable code paths.
- CN: 这里打开了命名空间（bool），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 DeadCodeElimination 等类型，用来承载该组件的状态或接口。 这里定义了 DeadCodeElimination 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里声明或实现了 set, HexagonBitSimplify::getFinalVRegClass, getRegClass, getTargetRegisterInfo, ... (19 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

### Lines 1001-1200 / 第 1001-1200 行

```cpp
  1001:   MachineBasicBlock *B = N->getBlock();
  1002:   std::vector<MachineInstr*> Instrs;
  1003:   for (MachineInstr &MI : llvm::reverse(*B))
  1004:     Instrs.push_back(&MI);
  1005: 
  1006:   for (auto *MI : Instrs) {
  1007:     unsigned Opc = MI->getOpcode();
  1008:     // Do not touch lifetime markers. This is why the target-independent DCE
  1009:     // cannot be used.
  1010:     if (Opc == TargetOpcode::LIFETIME_START ||
  1011:         Opc == TargetOpcode::LIFETIME_END)
  1012:       continue;
  1013:     bool Store = false;
  1014:     if (MI->isInlineAsm())
  1015:       continue;
  1016:     // Delete PHIs if possible.
  1017:     if (!MI->isPHI() && !MI->isSafeToMove(Store))
  1018:       continue;
  1019: 
  1020:     bool AllDead = true;
  1021:     SmallVector<unsigned,2> Regs;
  1022:     for (auto &Op : MI->operands()) {
  1023:       if (!Op.isReg() || !Op.isDef())
  1024:         continue;
  1025:       Register R = Op.getReg();
  1026:       if (!R.isVirtual() || !isDead(R)) {
  1027:         AllDead = false;
  1028:         break;
  1029:       }
  1030:       Regs.push_back(R);
  1031:     }
  1032:     if (!AllDead)
  1033:       continue;
  1034: 
  1035:     B->erase(MI);
  1036:     for (unsigned Reg : Regs)
  1037:       MRI.markUsesInDebugValueAsUndef(Reg);
  1038:     Changed = true;
  1039:   }
  1040: 
  1041:   return Changed;
  1042: }
  1043: 
  1044: namespace {
  1045: 
  1046: // Eliminate redundant instructions
  1047: //
  1048: // This transformation will identify instructions where the output register
  1049: // is the same as one of its input registers. This only works on instructions
  1050: // that define a single register (unlike post-increment loads, for example).
  1051: // The equality check is actually more detailed: the code calculates which
  1052: // bits of the output are used, and only compares these bits with the input
  1053: // registers.
  1054: // If the output matches an input, the instruction is replaced with COPY.
  1055: // The copies will be removed by another transformation.
  1056:   class RedundantInstrElimination : public Transformation {
  1057:   public:
  1058:     RedundantInstrElimination(BitTracker &bt, const HexagonInstrInfo &hii,
  1059:           const HexagonRegisterInfo &hri, MachineRegisterInfo &mri)
  1060:         : Transformation(true), HII(hii), HRI(hri), MRI(mri), BT(bt) {}
  1061: 
  1062:     bool processBlock(MachineBasicBlock &B, const RegisterSet &AVs) override;
  1063: 
  1064:   private:
  1065:     bool isLossyShiftLeft(const MachineInstr &MI, unsigned OpN,
  1066:           unsigned &LostB, unsigned &LostE);
  1067:     bool isLossyShiftRight(const MachineInstr &MI, unsigned OpN,
  1068:           unsigned &LostB, unsigned &LostE);
  1069:     bool computeUsedBits(unsigned Reg, BitVector &Bits);
  1070:     bool computeUsedBits(const MachineInstr &MI, unsigned OpN, BitVector &Bits,
  1071:           uint16_t Begin);
  1072:     bool usedBitsEqual(BitTracker::RegisterRef RD, BitTracker::RegisterRef RS);
  1073: 
  1074:     const HexagonInstrInfo &HII;
  1075:     const HexagonRegisterInfo &HRI;
  1076:     MachineRegisterInfo &MRI;
  1077:     BitTracker &BT;
  1078:   };
  1079: 
  1080: } // end anonymous namespace
  1081: 
  1082: // Check if the instruction is a lossy shift left, where the input being
  1083: // shifted is the operand OpN of MI. If true, [LostB, LostE) is the range
  1084: // of bit indices that are lost.
  1085: bool RedundantInstrElimination::isLossyShiftLeft(const MachineInstr &MI,
  1086:       unsigned OpN, unsigned &LostB, unsigned &LostE) {
  1087:   using namespace Hexagon;
  1088: 
  1089:   unsigned Opc = MI.getOpcode();
  1090:   unsigned ImN, RegN, Width;
  1091:   switch (Opc) {
  1092:     case S2_asl_i_p:
  1093:       ImN = 2;
  1094:       RegN = 1;
  1095:       Width = 64;
  1096:       break;
  1097:     case S2_asl_i_p_acc:
  1098:     case S2_asl_i_p_and:
  1099:     case S2_asl_i_p_nac:
  1100:     case S2_asl_i_p_or:
  1101:     case S2_asl_i_p_xacc:
  1102:       ImN = 3;
  1103:       RegN = 2;
  1104:       Width = 64;
  1105:       break;
  1106:     case S2_asl_i_r:
  1107:       ImN = 2;
  1108:       RegN = 1;
  1109:       Width = 32;
  1110:       break;
  1111:     case S2_addasl_rrri:
  1112:     case S4_andi_asl_ri:
  1113:     case S4_ori_asl_ri:
  1114:     case S4_addi_asl_ri:
  1115:     case S4_subi_asl_ri:
  1116:     case S2_asl_i_r_acc:
  1117:     case S2_asl_i_r_and:
  1118:     case S2_asl_i_r_nac:
  1119:     case S2_asl_i_r_or:
  1120:     case S2_asl_i_r_sat:
  1121:     case S2_asl_i_r_xacc:
  1122:       ImN = 3;
  1123:       RegN = 2;
  1124:       Width = 32;
  1125:       break;
  1126:     default:
  1127:       return false;
  1128:   }
  1129: 
  1130:   if (RegN != OpN)
  1131:     return false;
  1132: 
  1133:   assert(MI.getOperand(ImN).isImm());
  1134:   unsigned S = MI.getOperand(ImN).getImm();
  1135:   if (S == 0)
  1136:     return false;
  1137:   LostB = Width-S;
  1138:   LostE = Width;
  1139:   return true;
  1140: }
  1141: 
  1142: // Check if the instruction is a lossy shift right, where the input being
  1143: // shifted is the operand OpN of MI. If true, [LostB, LostE) is the range
  1144: // of bit indices that are lost.
  1145: bool RedundantInstrElimination::isLossyShiftRight(const MachineInstr &MI,
  1146:       unsigned OpN, unsigned &LostB, unsigned &LostE) {
  1147:   using namespace Hexagon;
  1148: 
  1149:   unsigned Opc = MI.getOpcode();
  1150:   unsigned ImN, RegN;
  1151:   switch (Opc) {
  1152:     case S2_asr_i_p:
  1153:     case S2_lsr_i_p:
  1154:       ImN = 2;
  1155:       RegN = 1;
  1156:       break;
  1157:     case S2_asr_i_p_acc:
  1158:     case S2_asr_i_p_and:
  1159:     case S2_asr_i_p_nac:
  1160:     case S2_asr_i_p_or:
  1161:     case S2_lsr_i_p_acc:
  1162:     case S2_lsr_i_p_and:
  1163:     case S2_lsr_i_p_nac:
  1164:     case S2_lsr_i_p_or:
  1165:     case S2_lsr_i_p_xacc:
  1166:       ImN = 3;
  1167:       RegN = 2;
  1168:       break;
  1169:     case S2_asr_i_r:
  1170:     case S2_lsr_i_r:
  1171:       ImN = 2;
  1172:       RegN = 1;
  1173:       break;
  1174:     case S4_andi_lsr_ri:
  1175:     case S4_ori_lsr_ri:
  1176:     case S4_addi_lsr_ri:
  1177:     case S4_subi_lsr_ri:
  1178:     case S2_asr_i_r_acc:
  1179:     case S2_asr_i_r_and:
  1180:     case S2_asr_i_r_nac:
  1181:     case S2_asr_i_r_or:
  1182:     case S2_lsr_i_r_acc:
  1183:     case S2_lsr_i_r_and:
  1184:     case S2_lsr_i_r_nac:
  1185:     case S2_lsr_i_r_or:
  1186:     case S2_lsr_i_r_xacc:
  1187:       ImN = 3;
  1188:       RegN = 2;
  1189:       break;
  1190: 
  1191:     default:
  1192:       return false;
  1193:   }
  1194: 
  1195:   if (RegN != OpN)
  1196:     return false;
  1197: 
  1198:   assert(MI.getOperand(ImN).isImm());
  1199:   unsigned S = MI.getOperand(ImN).getImm();
  1200:   LostB = 0;
```
- EN: It opens namespaces (Hexagon) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as RedundantInstrElimination, which carry the state or API of this component. It defines declarative TableGen records like RedundantInstrElimination; these records are consumed by TableGen instead of executed directly. It declares or implements routines such as getBlock, getOpcode, getReg, push_back, ... (15 total), translating Hexagon-specific policy into reusable code paths.
- CN: 这里打开了命名空间（Hexagon），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 RedundantInstrElimination 等类型，用来承载该组件的状态或接口。 这里定义了 RedundantInstrElimination 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里声明或实现了 getBlock, getOpcode, getReg, push_back, ... (15 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

### Lines 1201-1400 / 第 1201-1400 行

```cpp
  1201:   LostE = S;
  1202:   return true;
  1203: }
  1204: 
  1205: // Calculate the bit vector that corresponds to the used bits of register Reg.
  1206: // The vector Bits has the same size, as the size of Reg in bits. If the cal-
  1207: // culation fails (i.e. the used bits are unknown), it returns false. Other-
  1208: // wise, it returns true and sets the corresponding bits in Bits.
  1209: bool RedundantInstrElimination::computeUsedBits(unsigned Reg, BitVector &Bits) {
  1210:   BitVector Used(Bits.size());
  1211:   RegisterSet Visited;
  1212:   std::vector<unsigned> Pending;
  1213:   Pending.push_back(Reg);
  1214: 
  1215:   for (unsigned i = 0; i < Pending.size(); ++i) {
  1216:     unsigned R = Pending[i];
  1217:     if (Visited.has(R))
  1218:       continue;
  1219:     Visited.insert(R);
  1220:     for (auto I = MRI.use_begin(R), E = MRI.use_end(); I != E; ++I) {
  1221:       BitTracker::RegisterRef UR = *I;
  1222:       unsigned B, W;
  1223:       if (!HBS::getSubregMask(UR, B, W, MRI))
  1224:         return false;
  1225:       MachineInstr &UseI = *I->getParent();
  1226:       if (UseI.isPHI() || UseI.isCopy()) {
  1227:         Register DefR = UseI.getOperand(0).getReg();
  1228:         if (!DefR.isVirtual())
  1229:           return false;
  1230:         Pending.push_back(DefR);
  1231:       } else {
  1232:         if (!computeUsedBits(UseI, I.getOperandNo(), Used, B))
  1233:           return false;
  1234:       }
  1235:     }
  1236:   }
  1237:   Bits |= Used;
  1238:   return true;
  1239: }
  1240: 
  1241: // Calculate the bits used by instruction MI in a register in operand OpN.
  1242: // Return true/false if the calculation succeeds/fails. If is succeeds, set
  1243: // used bits in Bits. This function does not reset any bits in Bits, so
  1244: // subsequent calls over different instructions will result in the union
  1245: // of the used bits in all these instructions.
  1246: // The register in question may be used with a sub-register, whereas Bits
  1247: // holds the bits for the entire register. To keep track of that, the
  1248: // argument Begin indicates where in Bits is the lowest-significant bit
  1249: // of the register used in operand OpN. For example, in instruction:
  1250: //   %1 = S2_lsr_i_r %2:isub_hi, 10
  1251: // the operand 1 is a 32-bit register, which happens to be a subregister
  1252: // of the 64-bit register %2, and that subregister starts at position 32.
  1253: // In this case Begin=32, since Bits[32] would be the lowest-significant bit
  1254: // of %2:isub_hi.
  1255: bool RedundantInstrElimination::computeUsedBits(const MachineInstr &MI,
  1256:       unsigned OpN, BitVector &Bits, uint16_t Begin) {
  1257:   unsigned Opc = MI.getOpcode();
  1258:   BitVector T(Bits.size());
  1259:   bool GotBits = HBS::getUsedBits(Opc, OpN, T, Begin, HII);
  1260:   // Even if we don't have bits yet, we could still provide some information
  1261:   // if the instruction is a lossy shift: the lost bits will be marked as
  1262:   // not used.
  1263:   unsigned LB, LE;
  1264:   if (isLossyShiftLeft(MI, OpN, LB, LE) || isLossyShiftRight(MI, OpN, LB, LE)) {
  1265:     assert(MI.getOperand(OpN).isReg());
  1266:     BitTracker::RegisterRef RR = MI.getOperand(OpN);
  1267:     const TargetRegisterClass *RC = HBS::getFinalVRegClass(RR, MRI);
  1268:     uint16_t Width = HRI.getRegSizeInBits(*RC);
  1269: 
  1270:     if (!GotBits)
  1271:       T.set(Begin, Begin+Width);
  1272:     assert(LB <= LE && LB < Width && LE <= Width);
  1273:     T.reset(Begin+LB, Begin+LE);
  1274:     GotBits = true;
  1275:   }
  1276:   if (GotBits)
  1277:     Bits |= T;
  1278:   return GotBits;
  1279: }
  1280: 
  1281: // Calculates the used bits in RD ("defined register"), and checks if these
  1282: // bits in RS ("used register") and RD are identical.
  1283: bool RedundantInstrElimination::usedBitsEqual(BitTracker::RegisterRef RD,
  1284:       BitTracker::RegisterRef RS) {
  1285:   const BitTracker::RegisterCell &DC = BT.lookup(RD.Reg);
  1286:   const BitTracker::RegisterCell &SC = BT.lookup(RS.Reg);
  1287: 
  1288:   unsigned DB, DW;
  1289:   if (!HBS::getSubregMask(RD, DB, DW, MRI))
  1290:     return false;
  1291:   unsigned SB, SW;
  1292:   if (!HBS::getSubregMask(RS, SB, SW, MRI))
  1293:     return false;
  1294:   if (SW != DW)
  1295:     return false;
  1296: 
  1297:   BitVector Used(DC.width());
  1298:   if (!computeUsedBits(RD.Reg, Used))
  1299:     return false;
  1300: 
  1301:   for (unsigned i = 0; i != DW; ++i)
  1302:     if (Used[i+DB] && DC[DB+i] != SC[SB+i])
  1303:       return false;
  1304:   return true;
  1305: }
  1306: 
  1307: bool RedundantInstrElimination::processBlock(MachineBasicBlock &B,
  1308:       const RegisterSet&) {
  1309:   if (!BT.reached(&B))
  1310:     return false;
  1311:   bool Changed = false;
  1312: 
  1313:   for (auto I = B.begin(), E = B.end(); I != E; ++I) {
  1314:     MachineInstr *MI = &*I;
  1315: 
  1316:     if (MI->getOpcode() == TargetOpcode::COPY)
  1317:       continue;
  1318:     if (MI->isPHI() || MI->hasUnmodeledSideEffects() || MI->isInlineAsm())
  1319:       continue;
  1320:     unsigned NumD = MI->getDesc().getNumDefs();
  1321:     if (NumD != 1)
  1322:       continue;
  1323: 
  1324:     BitTracker::RegisterRef RD = MI->getOperand(0);
  1325:     if (!BT.has(RD.Reg))
  1326:       continue;
  1327:     const BitTracker::RegisterCell &DC = BT.lookup(RD.Reg);
  1328:     auto At = MachineBasicBlock::iterator(MI);
  1329: 
  1330:     // Find a source operand that is equal to the result.
  1331:     for (auto &Op : MI->uses()) {
  1332:       if (!Op.isReg())
  1333:         continue;
  1334:       BitTracker::RegisterRef RS = Op;
  1335:       if (!BT.has(RS.Reg))
  1336:         continue;
  1337:       if (!HBS::isTransparentCopy(RD, RS, MRI))
  1338:         continue;
  1339: 
  1340:       unsigned BN, BW;
  1341:       if (!HBS::getSubregMask(RS, BN, BW, MRI))
  1342:         continue;
  1343: 
  1344:       const BitTracker::RegisterCell &SC = BT.lookup(RS.Reg);
  1345:       if (!usedBitsEqual(RD, RS) && !HBS::isEqual(DC, 0, SC, BN, BW))
  1346:         continue;
  1347: 
  1348:       // If found, replace the instruction with a COPY.
  1349:       const DebugLoc &DL = MI->getDebugLoc();
  1350:       const TargetRegisterClass *FRC = HBS::getFinalVRegClass(RD, MRI);
  1351:       Register NewR = MRI.createVirtualRegister(FRC);
  1352:       MachineInstr *CopyI =
  1353:           BuildMI(B, At, DL, HII.get(TargetOpcode::COPY), NewR)
  1354:               .addReg(RS.Reg, {}, RS.Sub);
  1355:       HBS::replaceSubWithSub(RD.Reg, RD.Sub, NewR, 0, MRI);
  1356:       // This pass can create copies between registers that don't have the
  1357:       // exact same values. Updating the tracker has to involve updating
  1358:       // all dependent cells. Example:
  1359:       //   %1  = inst %2     ; %1 != %2, but used bits are equal
  1360:       //
  1361:       //   %3  = copy %2     ; <- inserted
  1362:       //   ... = %3          ; <- replaced from %2
  1363:       // Indirectly, we can create a "copy" between %1 and %2 even
  1364:       // though their exact values do not match.
  1365:       BT.visit(*CopyI);
  1366:       Changed = true;
  1367:       break;
  1368:     }
  1369:   }
  1370: 
  1371:   return Changed;
  1372: }
  1373: 
  1374: namespace {
  1375: 
  1376: // Recognize instructions that produce constant values known at compile-time.
  1377: // Replace them with register definitions that load these constants directly.
  1378:   class ConstGeneration : public Transformation {
  1379:   public:
  1380:     ConstGeneration(BitTracker &bt, const HexagonInstrInfo &hii,
  1381:         MachineRegisterInfo &mri)
  1382:       : Transformation(true), HII(hii), MRI(mri), BT(bt) {}
  1383: 
  1384:     bool processBlock(MachineBasicBlock &B, const RegisterSet &AVs) override;
  1385:     static bool isTfrConst(const MachineInstr &MI);
  1386: 
  1387:   private:
  1388:     Register genTfrConst(const TargetRegisterClass *RC, int64_t C,
  1389:                          MachineBasicBlock &B, MachineBasicBlock::iterator At,
  1390:                          DebugLoc &DL);
  1391: 
  1392:     const HexagonInstrInfo &HII;
  1393:     MachineRegisterInfo &MRI;
  1394:     BitTracker &BT;
  1395:   };
  1396: 
  1397: } // end anonymous namespace
  1398: 
  1399: bool ConstGeneration::isTfrConst(const MachineInstr &MI) {
  1400:   unsigned Opc = MI.getOpcode();
```
- EN: It opens namespaces (bool) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as ConstGeneration, which carry the state or API of this component. It defines declarative TableGen records like ConstGeneration; these records are consumed by TableGen instead of executed directly. It declares or implements routines such as fails, Used, push_back, size, ... (29 total), translating Hexagon-specific policy into reusable code paths.
- CN: 这里打开了命名空间（bool），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 ConstGeneration 等类型，用来承载该组件的状态或接口。 这里定义了 ConstGeneration 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里声明或实现了 fails, Used, push_back, size, ... (29 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

### Lines 1401-1600 / 第 1401-1600 行

```cpp
  1401:   switch (Opc) {
  1402:     case Hexagon::A2_combineii:
  1403:     case Hexagon::A4_combineii:
  1404:     case Hexagon::A2_tfrsi:
  1405:     case Hexagon::A2_tfrpi:
  1406:     case Hexagon::PS_true:
  1407:     case Hexagon::PS_false:
  1408:     case Hexagon::CONST32:
  1409:     case Hexagon::CONST64:
  1410:       return true;
  1411:   }
  1412:   return false;
  1413: }
  1414: 
  1415: // Generate a transfer-immediate instruction that is appropriate for the
  1416: // register class and the actual value being transferred.
  1417: Register ConstGeneration::genTfrConst(const TargetRegisterClass *RC, int64_t C,
  1418:                                       MachineBasicBlock &B,
  1419:                                       MachineBasicBlock::iterator At,
  1420:                                       DebugLoc &DL) {
  1421:   Register Reg = MRI.createVirtualRegister(RC);
  1422:   if (RC == &Hexagon::IntRegsRegClass) {
  1423:     BuildMI(B, At, DL, HII.get(Hexagon::A2_tfrsi), Reg)
  1424:         .addImm(int32_t(C));
  1425:     return Reg;
  1426:   }
  1427: 
  1428:   if (RC == &Hexagon::DoubleRegsRegClass) {
  1429:     if (isInt<8>(C)) {
  1430:       BuildMI(B, At, DL, HII.get(Hexagon::A2_tfrpi), Reg)
  1431:           .addImm(C);
  1432:       return Reg;
  1433:     }
  1434: 
  1435:     unsigned Lo = Lo_32(C), Hi = Hi_32(C);
  1436:     if (isInt<8>(Lo) || isInt<8>(Hi)) {
  1437:       unsigned Opc = isInt<8>(Lo) ? Hexagon::A2_combineii
  1438:                                   : Hexagon::A4_combineii;
  1439:       BuildMI(B, At, DL, HII.get(Opc), Reg)
  1440:           .addImm(int32_t(Hi))
  1441:           .addImm(int32_t(Lo));
  1442:       return Reg;
  1443:     }
  1444:     MachineFunction *MF = B.getParent();
  1445:     auto &HST = MF->getSubtarget<HexagonSubtarget>();
  1446: 
  1447:     // Disable CONST64 for tiny core since it takes a LD resource.
  1448:     if (!HST.isTinyCore() ||
  1449:         MF->getFunction().hasOptSize()) {
  1450:       BuildMI(B, At, DL, HII.get(Hexagon::CONST64), Reg)
  1451:           .addImm(C);
  1452:       return Reg;
  1453:     }
  1454:   }
  1455: 
  1456:   if (RC == &Hexagon::PredRegsRegClass) {
  1457:     unsigned Opc;
  1458:     if (C == 0)
  1459:       Opc = Hexagon::PS_false;
  1460:     else if ((C & 0xFF) == 0xFF)
  1461:       Opc = Hexagon::PS_true;
  1462:     else
  1463:       return 0;
  1464:     BuildMI(B, At, DL, HII.get(Opc), Reg);
  1465:     return Reg;
  1466:   }
  1467: 
  1468:   return 0;
  1469: }
  1470: 
  1471: bool ConstGeneration::processBlock(MachineBasicBlock &B, const RegisterSet&) {
  1472:   if (!BT.reached(&B))
  1473:     return false;
  1474:   bool Changed = false;
  1475:   RegisterSet Defs;
  1476: 
  1477:   for (auto I = B.begin(), E = B.end(); I != E; ++I) {
  1478:     if (isTfrConst(*I))
  1479:       continue;
  1480:     Defs.clear();
  1481:     HBS::getInstrDefs(*I, Defs);
  1482:     if (Defs.count() != 1)
  1483:       continue;
  1484:     Register DR = Defs.find_first();
  1485:     if (!DR.isVirtual())
  1486:       continue;
  1487:     uint64_t U;
  1488:     const BitTracker::RegisterCell &DRC = BT.lookup(DR);
  1489:     if (HBS::getConst(DRC, 0, DRC.width(), U)) {
  1490:       int64_t C = U;
  1491:       DebugLoc DL = I->getDebugLoc();
  1492:       auto At = I->isPHI() ? B.getFirstNonPHI() : I;
  1493:       Register ImmReg = genTfrConst(MRI.getRegClass(DR), C, B, At, DL);
  1494:       if (ImmReg) {
  1495:         HBS::replaceReg(DR, ImmReg, MRI);
  1496:         BT.put(ImmReg, DRC);
  1497:         Changed = true;
  1498:       }
  1499:     }
  1500:   }
  1501:   return Changed;
  1502: }
  1503: 
  1504: namespace {
  1505: 
  1506: // Identify pairs of available registers which hold identical values.
  1507: // In such cases, only one of them needs to be calculated, the other one
  1508: // will be defined as a copy of the first.
  1509:   class CopyGeneration : public Transformation {
  1510:   public:
  1511:     CopyGeneration(BitTracker &bt, const HexagonInstrInfo &hii,
  1512:         const HexagonRegisterInfo &hri, MachineRegisterInfo &mri)
  1513:       : Transformation(true), HII(hii), HRI(hri), MRI(mri), BT(bt) {}
  1514: 
  1515:     bool processBlock(MachineBasicBlock &B, const RegisterSet &AVs) override;
  1516: 
  1517:   private:
  1518:     bool findMatch(const BitTracker::RegisterRef &Inp,
  1519:         BitTracker::RegisterRef &Out, const RegisterSet &AVs);
  1520: 
  1521:     const HexagonInstrInfo &HII;
  1522:     const HexagonRegisterInfo &HRI;
  1523:     MachineRegisterInfo &MRI;
  1524:     BitTracker &BT;
  1525:     RegisterSet Forbidden;
  1526:   };
  1527: 
  1528: // Eliminate register copies RD = RS, by replacing the uses of RD with
  1529: // with uses of RS.
  1530:   class CopyPropagation : public Transformation {
  1531:   public:
  1532:     CopyPropagation(const HexagonRegisterInfo &hri, MachineRegisterInfo &mri)
  1533:         : Transformation(false), HRI(hri), MRI(mri) {}
  1534: 
  1535:     bool processBlock(MachineBasicBlock &B, const RegisterSet &AVs) override;
  1536: 
  1537:     static bool isCopyReg(unsigned Opc, bool NoConv);
  1538: 
  1539:   private:
  1540:     bool propagateRegCopy(MachineInstr &MI);
  1541: 
  1542:     const HexagonRegisterInfo &HRI;
  1543:     MachineRegisterInfo &MRI;
  1544:   };
  1545: 
  1546: } // end anonymous namespace
  1547: 
  1548: /// Check if there is a register in AVs that is identical to Inp. If so,
  1549: /// set Out to the found register. The output may be a pair Reg:Sub.
  1550: bool CopyGeneration::findMatch(const BitTracker::RegisterRef &Inp,
  1551:       BitTracker::RegisterRef &Out, const RegisterSet &AVs) {
  1552:   if (!BT.has(Inp.Reg))
  1553:     return false;
  1554:   const BitTracker::RegisterCell &InpRC = BT.lookup(Inp.Reg);
  1555:   auto *FRC = HBS::getFinalVRegClass(Inp, MRI);
  1556:   unsigned B, W;
  1557:   if (!HBS::getSubregMask(Inp, B, W, MRI))
  1558:     return false;
  1559: 
  1560:   for (Register R = AVs.find_first(); R; R = AVs.find_next(R)) {
  1561:     if (!BT.has(R) || Forbidden[R])
  1562:       continue;
  1563:     const BitTracker::RegisterCell &RC = BT.lookup(R);
  1564:     unsigned RW = RC.width();
  1565:     if (W == RW) {
  1566:       if (FRC != MRI.getRegClass(R))
  1567:         continue;
  1568:       if (!HBS::isTransparentCopy(R, Inp, MRI))
  1569:         continue;
  1570:       if (!HBS::isEqual(InpRC, B, RC, 0, W))
  1571:         continue;
  1572:       Out.Reg = R;
  1573:       Out.Sub = 0;
  1574:       return true;
  1575:     }
  1576:     // Check if there is a super-register, whose part (with a subregister)
  1577:     // is equal to the input.
  1578:     // Only do double registers for now.
  1579:     if (W*2 != RW)
  1580:       continue;
  1581:     if (MRI.getRegClass(R) != &Hexagon::DoubleRegsRegClass)
  1582:       continue;
  1583: 
  1584:     if (HBS::isEqual(InpRC, B, RC, 0, W))
  1585:       Out.Sub = Hexagon::isub_lo;
  1586:     else if (HBS::isEqual(InpRC, B, RC, W, W))
  1587:       Out.Sub = Hexagon::isub_hi;
  1588:     else
  1589:       continue;
  1590:     Out.Reg = R;
  1591:     if (HBS::isTransparentCopy(Out, Inp, MRI))
  1592:       return true;
  1593:   }
  1594:   return false;
  1595: }
  1596: 
  1597: bool CopyGeneration::processBlock(MachineBasicBlock &B,
  1598:       const RegisterSet &AVs) {
  1599:   if (!BT.reached(&B))
  1600:     return false;
```
- EN: It declares types such as CopyGeneration, CopyPropagation, which carry the state or API of this component. It defines declarative TableGen records like CopyGeneration, CopyPropagation; these records are consumed by TableGen instead of executed directly. It declares or implements routines such as ConstGeneration::genTfrConst, createVirtualRegister, BuildMI, Lo_32, ... (26 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明了 CopyGeneration, CopyPropagation 等类型，用来承载该组件的状态或接口。 这里定义了 CopyGeneration, CopyPropagation 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里声明或实现了 ConstGeneration::genTfrConst, createVirtualRegister, BuildMI, Lo_32, ... (26 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 1601-1800 / 第 1601-1800 行

```cpp
  1601:   RegisterSet AVB(AVs);
  1602:   bool Changed = false;
  1603:   RegisterSet Defs;
  1604: 
  1605:   for (auto I = B.begin(), E = B.end(); I != E; ++I, AVB.insert(Defs)) {
  1606:     Defs.clear();
  1607:     HBS::getInstrDefs(*I, Defs);
  1608: 
  1609:     unsigned Opc = I->getOpcode();
  1610:     if (CopyPropagation::isCopyReg(Opc, false) ||
  1611:         ConstGeneration::isTfrConst(*I))
  1612:       continue;
  1613: 
  1614:     DebugLoc DL = I->getDebugLoc();
  1615:     auto At = I->isPHI() ? B.getFirstNonPHI() : I;
  1616: 
  1617:     for (Register R = Defs.find_first(); R; R = Defs.find_next(R)) {
  1618:       BitTracker::RegisterRef MR;
  1619:       auto *FRC = HBS::getFinalVRegClass(R, MRI);
  1620: 
  1621:       if (findMatch(R, MR, AVB)) {
  1622:         Register NewR = MRI.createVirtualRegister(FRC);
  1623:         BuildMI(B, At, DL, HII.get(TargetOpcode::COPY), NewR)
  1624:             .addReg(MR.Reg, {}, MR.Sub);
  1625:         BT.put(BitTracker::RegisterRef(NewR), BT.get(MR));
  1626:         HBS::replaceReg(R, NewR, MRI);
  1627:         Forbidden.insert(R);
  1628:         continue;
  1629:       }
  1630: 
  1631:       if (FRC == &Hexagon::DoubleRegsRegClass ||
  1632:           FRC == &Hexagon::HvxWRRegClass) {
  1633:         // Try to generate REG_SEQUENCE.
  1634:         unsigned SubLo = HRI.getHexagonSubRegIndex(*FRC, Hexagon::ps_sub_lo);
  1635:         unsigned SubHi = HRI.getHexagonSubRegIndex(*FRC, Hexagon::ps_sub_hi);
  1636:         BitTracker::RegisterRef TL = { R, SubLo };
  1637:         BitTracker::RegisterRef TH = { R, SubHi };
  1638:         BitTracker::RegisterRef ML, MH;
  1639:         if (findMatch(TL, ML, AVB) && findMatch(TH, MH, AVB)) {
  1640:           auto *FRC = HBS::getFinalVRegClass(R, MRI);
  1641:           Register NewR = MRI.createVirtualRegister(FRC);
  1642:           BuildMI(B, At, DL, HII.get(TargetOpcode::REG_SEQUENCE), NewR)
  1643:               .addReg(ML.Reg, {}, ML.Sub)
  1644:               .addImm(SubLo)
  1645:               .addReg(MH.Reg, {}, MH.Sub)
  1646:               .addImm(SubHi);
  1647:           BT.put(BitTracker::RegisterRef(NewR), BT.get(R));
  1648:           HBS::replaceReg(R, NewR, MRI);
  1649:           Forbidden.insert(R);
  1650:         }
  1651:       }
  1652:     }
  1653:   }
  1654: 
  1655:   return Changed;
  1656: }
  1657: 
  1658: bool CopyPropagation::isCopyReg(unsigned Opc, bool NoConv) {
  1659:   switch (Opc) {
  1660:     case TargetOpcode::COPY:
  1661:     case TargetOpcode::REG_SEQUENCE:
  1662:     case Hexagon::A4_combineir:
  1663:     case Hexagon::A4_combineri:
  1664:       return true;
  1665:     case Hexagon::A2_tfr:
  1666:     case Hexagon::A2_tfrp:
  1667:     case Hexagon::A2_combinew:
  1668:     case Hexagon::V6_vcombine:
  1669:       return NoConv;
  1670:     default:
  1671:       break;
  1672:   }
  1673:   return false;
  1674: }
  1675: 
  1676: bool CopyPropagation::propagateRegCopy(MachineInstr &MI) {
  1677:   bool Changed = false;
  1678:   unsigned Opc = MI.getOpcode();
  1679:   BitTracker::RegisterRef RD = MI.getOperand(0);
  1680:   assert(MI.getOperand(0).getSubReg() == 0);
  1681: 
  1682:   switch (Opc) {
  1683:     case TargetOpcode::COPY:
  1684:     case Hexagon::A2_tfr:
  1685:     case Hexagon::A2_tfrp: {
  1686:       BitTracker::RegisterRef RS = MI.getOperand(1);
  1687:       if (!HBS::isTransparentCopy(RD, RS, MRI))
  1688:         break;
  1689:       if (RS.Sub != 0)
  1690:         Changed = HBS::replaceRegWithSub(RD.Reg, RS.Reg, RS.Sub, MRI);
  1691:       else
  1692:         Changed = HBS::replaceReg(RD.Reg, RS.Reg, MRI);
  1693:       break;
  1694:     }
  1695:     case TargetOpcode::REG_SEQUENCE: {
  1696:       BitTracker::RegisterRef SL, SH;
  1697:       if (HBS::parseRegSequence(MI, SL, SH, MRI)) {
  1698:         const TargetRegisterClass &RC = *MRI.getRegClass(RD.Reg);
  1699:         unsigned SubLo = HRI.getHexagonSubRegIndex(RC, Hexagon::ps_sub_lo);
  1700:         unsigned SubHi = HRI.getHexagonSubRegIndex(RC, Hexagon::ps_sub_hi);
  1701:         Changed  = HBS::replaceSubWithSub(RD.Reg, SubLo, SL.Reg, SL.Sub, MRI);
  1702:         Changed |= HBS::replaceSubWithSub(RD.Reg, SubHi, SH.Reg, SH.Sub, MRI);
  1703:       }
  1704:       break;
  1705:     }
  1706:     case Hexagon::A2_combinew:
  1707:     case Hexagon::V6_vcombine: {
  1708:       const TargetRegisterClass &RC = *MRI.getRegClass(RD.Reg);
  1709:       unsigned SubLo = HRI.getHexagonSubRegIndex(RC, Hexagon::ps_sub_lo);
  1710:       unsigned SubHi = HRI.getHexagonSubRegIndex(RC, Hexagon::ps_sub_hi);
  1711:       BitTracker::RegisterRef RH = MI.getOperand(1), RL = MI.getOperand(2);
  1712:       Changed  = HBS::replaceSubWithSub(RD.Reg, SubLo, RL.Reg, RL.Sub, MRI);
  1713:       Changed |= HBS::replaceSubWithSub(RD.Reg, SubHi, RH.Reg, RH.Sub, MRI);
  1714:       break;
  1715:     }
  1716:     case Hexagon::A4_combineir:
  1717:     case Hexagon::A4_combineri: {
  1718:       unsigned SrcX = (Opc == Hexagon::A4_combineir) ? 2 : 1;
  1719:       unsigned Sub = (Opc == Hexagon::A4_combineir) ? Hexagon::isub_lo
  1720:                                                     : Hexagon::isub_hi;
  1721:       BitTracker::RegisterRef RS = MI.getOperand(SrcX);
  1722:       Changed = HBS::replaceSubWithSub(RD.Reg, Sub, RS.Reg, RS.Sub, MRI);
  1723:       break;
  1724:     }
  1725:   }
  1726:   return Changed;
  1727: }
  1728: 
  1729: bool CopyPropagation::processBlock(MachineBasicBlock &B, const RegisterSet&) {
  1730:   std::vector<MachineInstr*> Instrs;
  1731:   for (MachineInstr &MI : llvm::reverse(B))
  1732:     Instrs.push_back(&MI);
  1733: 
  1734:   bool Changed = false;
  1735:   for (auto *I : Instrs) {
  1736:     unsigned Opc = I->getOpcode();
  1737:     if (!CopyPropagation::isCopyReg(Opc, true))
  1738:       continue;
  1739:     Changed |= propagateRegCopy(*I);
  1740:   }
  1741: 
  1742:   return Changed;
  1743: }
  1744: 
  1745: namespace {
  1746: 
  1747: // Recognize patterns that can be simplified and replace them with the
  1748: // simpler forms.
  1749: // This is by no means complete
  1750:   class BitSimplification : public Transformation {
  1751:   public:
  1752:     BitSimplification(BitTracker &bt, const MachineDominatorTree &mdt,
  1753:                       const HexagonInstrInfo &hii,
  1754:                       const HexagonRegisterInfo &hri, MachineRegisterInfo &mri,
  1755:                       MachineFunction &mf)
  1756:         : Transformation(true), MDT(mdt), HII(hii), HRI(hri), MRI(mri), BT(bt) {
  1757:     }
  1758: 
  1759:     bool processBlock(MachineBasicBlock &B, const RegisterSet &AVs) override;
  1760: 
  1761:   private:
  1762:     struct RegHalf : public BitTracker::RegisterRef {
  1763:       bool Low;  // Low/High halfword.
  1764:     };
  1765: 
  1766:     bool matchHalf(unsigned SelfR, const BitTracker::RegisterCell &RC,
  1767:           unsigned B, RegHalf &RH);
  1768:     bool validateReg(BitTracker::RegisterRef R, unsigned Opc, unsigned OpNum);
  1769: 
  1770:     bool matchPackhl(unsigned SelfR, const BitTracker::RegisterCell &RC,
  1771:           BitTracker::RegisterRef &Rs, BitTracker::RegisterRef &Rt);
  1772:     unsigned getCombineOpcode(bool HLow, bool LLow);
  1773: 
  1774:     bool genStoreUpperHalf(MachineInstr *MI);
  1775:     bool genStoreImmediate(MachineInstr *MI);
  1776:     bool genPackhl(MachineInstr *MI, BitTracker::RegisterRef RD,
  1777:           const BitTracker::RegisterCell &RC);
  1778:     bool genExtractHalf(MachineInstr *MI, BitTracker::RegisterRef RD,
  1779:           const BitTracker::RegisterCell &RC);
  1780:     bool genCombineHalf(MachineInstr *MI, BitTracker::RegisterRef RD,
  1781:           const BitTracker::RegisterCell &RC);
  1782:     bool genExtractLow(MachineInstr *MI, BitTracker::RegisterRef RD,
  1783:           const BitTracker::RegisterCell &RC);
  1784:     bool genBitSplit(MachineInstr *MI, BitTracker::RegisterRef RD,
  1785:           const BitTracker::RegisterCell &RC, const RegisterSet &AVs);
  1786:     bool simplifyTstbit(MachineInstr *MI, BitTracker::RegisterRef RD,
  1787:           const BitTracker::RegisterCell &RC);
  1788:     bool simplifyExtractLow(MachineInstr *MI, BitTracker::RegisterRef RD,
  1789:           const BitTracker::RegisterCell &RC, const RegisterSet &AVs);
  1790:     bool simplifyRCmp0(MachineInstr *MI, BitTracker::RegisterRef RD);
  1791: 
  1792:     // Cache of created instructions to avoid creating duplicates.
  1793:     // XXX Currently only used by genBitSplit.
  1794:     std::vector<MachineInstr*> NewMIs;
  1795: 
  1796:     const MachineDominatorTree &MDT;
  1797:     const HexagonInstrInfo &HII;
  1798:     [[maybe_unused]] const HexagonRegisterInfo &HRI;
  1799:     MachineRegisterInfo &MRI;
  1800:     BitTracker &BT;
```
- EN: It declares types such as BitSimplification, RegHalf, which carry the state or API of this component. It defines declarative TableGen records like BitSimplification; these records are consumed by TableGen instead of executed directly. It declares or implements routines such as AVB, insert, clear, HBS::getInstrDefs, ... (37 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明了 BitSimplification, RegHalf 等类型，用来承载该组件的状态或接口。 这里定义了 BitSimplification 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里声明或实现了 AVB, insert, clear, HBS::getInstrDefs, ... (37 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 1801-2000 / 第 1801-2000 行

```cpp
  1801:   };
  1802: 
  1803: } // end anonymous namespace
  1804: 
  1805: // Check if the bits [B..B+16) in register cell RC form a valid halfword,
  1806: // i.e. [0..16), [16..32), etc. of some register. If so, return true and
  1807: // set the information about the found register in RH.
  1808: bool BitSimplification::matchHalf(unsigned SelfR,
  1809:       const BitTracker::RegisterCell &RC, unsigned B, RegHalf &RH) {
  1810:   // XXX This could be searching in the set of available registers, in case
  1811:   // the match is not exact.
  1812: 
  1813:   // Match 16-bit chunks, where the RC[B..B+15] references exactly one
  1814:   // register and all the bits B..B+15 match between RC and the register.
  1815:   // This is meant to match "v1[0-15]", where v1 = { [0]:0 [1-15]:v1... },
  1816:   // and RC = { [0]:0 [1-15]:v1[1-15]... }.
  1817:   bool Low = false;
  1818:   unsigned I = B;
  1819:   while (I < B+16 && RC[I].num())
  1820:     I++;
  1821:   if (I == B+16)
  1822:     return false;
  1823: 
  1824:   Register Reg = RC[I].RefI.Reg;
  1825:   unsigned P = RC[I].RefI.Pos;    // The RefI.Pos will be advanced by I-B.
  1826:   if (P < I-B)
  1827:     return false;
  1828:   unsigned Pos = P - (I-B);
  1829: 
  1830:   if (Reg == 0 || Reg == SelfR)    // Don't match "self".
  1831:     return false;
  1832:   if (!Reg.isVirtual())
  1833:     return false;
  1834:   if (!BT.has(Reg))
  1835:     return false;
  1836: 
  1837:   const BitTracker::RegisterCell &SC = BT.lookup(Reg);
  1838:   if (Pos+16 > SC.width())
  1839:     return false;
  1840: 
  1841:   for (unsigned i = 0; i < 16; ++i) {
  1842:     const BitTracker::BitValue &RV = RC[i+B];
  1843:     if (RV.Type == BitTracker::BitValue::Ref) {
  1844:       if (RV.RefI.Reg != Reg)
  1845:         return false;
  1846:       if (RV.RefI.Pos != i+Pos)
  1847:         return false;
  1848:       continue;
  1849:     }
  1850:     if (RC[i+B] != SC[i+Pos])
  1851:       return false;
  1852:   }
  1853: 
  1854:   unsigned Sub = 0;
  1855:   switch (Pos) {
  1856:     case 0:
  1857:       Sub = Hexagon::isub_lo;
  1858:       Low = true;
  1859:       break;
  1860:     case 16:
  1861:       Sub = Hexagon::isub_lo;
  1862:       Low = false;
  1863:       break;
  1864:     case 32:
  1865:       Sub = Hexagon::isub_hi;
  1866:       Low = true;
  1867:       break;
  1868:     case 48:
  1869:       Sub = Hexagon::isub_hi;
  1870:       Low = false;
  1871:       break;
  1872:     default:
  1873:       return false;
  1874:   }
  1875: 
  1876:   RH.Reg = Reg;
  1877:   RH.Sub = Sub;
  1878:   RH.Low = Low;
  1879:   // If the subregister is not valid with the register, set it to 0.
  1880:   if (!HBS::getFinalVRegClass(RH, MRI))
  1881:     RH.Sub = 0;
  1882: 
  1883:   return true;
  1884: }
  1885: 
  1886: bool BitSimplification::validateReg(BitTracker::RegisterRef R, unsigned Opc,
  1887:       unsigned OpNum) {
  1888:   auto *OpRC = HII.getRegClass(HII.get(Opc), OpNum);
  1889:   auto *RRC = HBS::getFinalVRegClass(R, MRI);
  1890:   return OpRC->hasSubClassEq(RRC);
  1891: }
  1892: 
  1893: // Check if RC matches the pattern of a S2_packhl. If so, return true and
  1894: // set the inputs Rs and Rt.
  1895: bool BitSimplification::matchPackhl(unsigned SelfR,
  1896:       const BitTracker::RegisterCell &RC, BitTracker::RegisterRef &Rs,
  1897:       BitTracker::RegisterRef &Rt) {
  1898:   RegHalf L1, H1, L2, H2;
  1899: 
  1900:   if (!matchHalf(SelfR, RC, 0, L2)  || !matchHalf(SelfR, RC, 16, L1))
  1901:     return false;
  1902:   if (!matchHalf(SelfR, RC, 32, H2) || !matchHalf(SelfR, RC, 48, H1))
  1903:     return false;
  1904: 
  1905:   // Rs = H1.L1, Rt = H2.L2
  1906:   if (H1.Reg != L1.Reg || H1.Sub != L1.Sub || H1.Low || !L1.Low)
  1907:     return false;
  1908:   if (H2.Reg != L2.Reg || H2.Sub != L2.Sub || H2.Low || !L2.Low)
  1909:     return false;
  1910: 
  1911:   Rs = H1;
  1912:   Rt = H2;
  1913:   return true;
  1914: }
  1915: 
  1916: unsigned BitSimplification::getCombineOpcode(bool HLow, bool LLow) {
  1917:   return HLow ? LLow ? Hexagon::A2_combine_ll
  1918:                      : Hexagon::A2_combine_lh
  1919:               : LLow ? Hexagon::A2_combine_hl
  1920:                      : Hexagon::A2_combine_hh;
  1921: }
  1922: 
  1923: // If MI stores the upper halfword of a register (potentially obtained via
  1924: // shifts or extracts), replace it with a storerf instruction. This could
  1925: // cause the "extraction" code to become dead.
  1926: bool BitSimplification::genStoreUpperHalf(MachineInstr *MI) {
  1927:   unsigned Opc = MI->getOpcode();
  1928:   if (Opc != Hexagon::S2_storerh_io)
  1929:     return false;
  1930: 
  1931:   MachineOperand &ValOp = MI->getOperand(2);
  1932:   BitTracker::RegisterRef RS = ValOp;
  1933:   if (!BT.has(RS.Reg))
  1934:     return false;
  1935:   const BitTracker::RegisterCell &RC = BT.lookup(RS.Reg);
  1936:   RegHalf H;
  1937:   unsigned B = (RS.Sub == Hexagon::isub_hi) ? 32 : 0;
  1938:   if (!matchHalf(0, RC, B, H))
  1939:     return false;
  1940:   if (H.Low)
  1941:     return false;
  1942:   MI->setDesc(HII.get(Hexagon::S2_storerf_io));
  1943:   ValOp.setReg(H.Reg);
  1944:   ValOp.setSubReg(H.Sub);
  1945:   return true;
  1946: }
  1947: 
  1948: // If MI stores a value known at compile-time, and the value is within a range
  1949: // that avoids using constant-extenders, replace it with a store-immediate.
  1950: bool BitSimplification::genStoreImmediate(MachineInstr *MI) {
  1951:   unsigned Opc = MI->getOpcode();
  1952:   unsigned Align = 0;
  1953:   switch (Opc) {
  1954:     case Hexagon::S2_storeri_io:
  1955:       Align++;
  1956:       [[fallthrough]];
  1957:     case Hexagon::S2_storerh_io:
  1958:       Align++;
  1959:       [[fallthrough]];
  1960:     case Hexagon::S2_storerb_io:
  1961:       break;
  1962:     default:
  1963:       return false;
  1964:   }
  1965: 
  1966:   // Avoid stores to frame-indices (due to an unknown offset).
  1967:   if (!MI->getOperand(0).isReg())
  1968:     return false;
  1969:   MachineOperand &OffOp = MI->getOperand(1);
  1970:   if (!OffOp.isImm())
  1971:     return false;
  1972: 
  1973:   int64_t Off = OffOp.getImm();
  1974:   // Offset is u6:a. Sadly, there is no isShiftedUInt(n,x).
  1975:   if (!isUIntN(6+Align, Off) || (Off & ((1<<Align)-1)))
  1976:     return false;
  1977:   // Source register:
  1978:   BitTracker::RegisterRef RS = MI->getOperand(2);
  1979:   if (!BT.has(RS.Reg))
  1980:     return false;
  1981:   const BitTracker::RegisterCell &RC = BT.lookup(RS.Reg);
  1982:   uint64_t U;
  1983:   if (!HBS::getConst(RC, 0, RC.width(), U))
  1984:     return false;
  1985: 
  1986:   // Only consider 8-bit values to avoid constant-extenders.
  1987:   int V;
  1988:   switch (Opc) {
  1989:     case Hexagon::S2_storerb_io:
  1990:       V = int8_t(U);
  1991:       break;
  1992:     case Hexagon::S2_storerh_io:
  1993:       V = int16_t(U);
  1994:       break;
  1995:     case Hexagon::S2_storeri_io:
  1996:       V = int32_t(U);
  1997:       break;
  1998:     default:
  1999:       // Opc is already checked above to be one of the three store instructions.
  2000:       // This silences a -Wuninitialized false positive on GCC 5.4.
```
- EN: It declares or implements routines such as BitSimplification::matchHalf, lookup, BitSimplification::validateReg, getRegClass, ... (19 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明或实现了 BitSimplification::matchHalf, lookup, BitSimplification::validateReg, getRegClass, ... (19 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 2001-2200 / 第 2001-2200 行

```cpp
  2001:       llvm_unreachable("Unexpected store opcode");
  2002:   }
  2003:   if (!isInt<8>(V))
  2004:     return false;
  2005: 
  2006:   MI->removeOperand(2);
  2007:   switch (Opc) {
  2008:     case Hexagon::S2_storerb_io:
  2009:       MI->setDesc(HII.get(Hexagon::S4_storeirb_io));
  2010:       break;
  2011:     case Hexagon::S2_storerh_io:
  2012:       MI->setDesc(HII.get(Hexagon::S4_storeirh_io));
  2013:       break;
  2014:     case Hexagon::S2_storeri_io:
  2015:       MI->setDesc(HII.get(Hexagon::S4_storeiri_io));
  2016:       break;
  2017:   }
  2018:   MI->addOperand(MachineOperand::CreateImm(V));
  2019:   return true;
  2020: }
  2021: 
  2022: // If MI is equivalent o S2_packhl, generate the S2_packhl. MI could be the
  2023: // last instruction in a sequence that results in something equivalent to
  2024: // the pack-halfwords. The intent is to cause the entire sequence to become
  2025: // dead.
  2026: bool BitSimplification::genPackhl(MachineInstr *MI,
  2027:       BitTracker::RegisterRef RD, const BitTracker::RegisterCell &RC) {
  2028:   unsigned Opc = MI->getOpcode();
  2029:   if (Opc == Hexagon::S2_packhl)
  2030:     return false;
  2031:   BitTracker::RegisterRef Rs, Rt;
  2032:   if (!matchPackhl(RD.Reg, RC, Rs, Rt))
  2033:     return false;
  2034:   if (!validateReg(Rs, Hexagon::S2_packhl, 1) ||
  2035:       !validateReg(Rt, Hexagon::S2_packhl, 2))
  2036:     return false;
  2037: 
  2038:   MachineBasicBlock &B = *MI->getParent();
  2039:   Register NewR = MRI.createVirtualRegister(&Hexagon::DoubleRegsRegClass);
  2040:   DebugLoc DL = MI->getDebugLoc();
  2041:   auto At = MI->isPHI() ? B.getFirstNonPHI()
  2042:                         : MachineBasicBlock::iterator(MI);
  2043:   BuildMI(B, At, DL, HII.get(Hexagon::S2_packhl), NewR)
  2044:       .addReg(Rs.Reg, {}, Rs.Sub)
  2045:       .addReg(Rt.Reg, {}, Rt.Sub);
  2046:   HBS::replaceSubWithSub(RD.Reg, RD.Sub, NewR, 0, MRI);
  2047:   BT.put(BitTracker::RegisterRef(NewR), RC);
  2048:   return true;
  2049: }
  2050: 
  2051: // If MI produces halfword of the input in the low half of the output,
  2052: // replace it with zero-extend or extractu.
  2053: bool BitSimplification::genExtractHalf(MachineInstr *MI,
  2054:       BitTracker::RegisterRef RD, const BitTracker::RegisterCell &RC) {
  2055:   RegHalf L;
  2056:   // Check for halfword in low 16 bits, zeros elsewhere.
  2057:   if (!matchHalf(RD.Reg, RC, 0, L) || !HBS::isZero(RC, 16, 16))
  2058:     return false;
  2059: 
  2060:   unsigned Opc = MI->getOpcode();
  2061:   MachineBasicBlock &B = *MI->getParent();
  2062:   DebugLoc DL = MI->getDebugLoc();
  2063: 
  2064:   // Prefer zxth, since zxth can go in any slot, while extractu only in
  2065:   // slots 2 and 3.
  2066:   unsigned NewR = 0;
  2067:   auto At = MI->isPHI() ? B.getFirstNonPHI()
  2068:                         : MachineBasicBlock::iterator(MI);
  2069:   if (L.Low && Opc != Hexagon::A2_zxth) {
  2070:     if (validateReg(L, Hexagon::A2_zxth, 1)) {
  2071:       NewR = MRI.createVirtualRegister(&Hexagon::IntRegsRegClass);
  2072:       BuildMI(B, At, DL, HII.get(Hexagon::A2_zxth), NewR)
  2073:           .addReg(L.Reg, {}, L.Sub);
  2074:     }
  2075:   } else if (!L.Low && Opc != Hexagon::S2_lsr_i_r) {
  2076:     if (validateReg(L, Hexagon::S2_lsr_i_r, 1)) {
  2077:       NewR = MRI.createVirtualRegister(&Hexagon::IntRegsRegClass);
  2078:       BuildMI(B, MI, DL, HII.get(Hexagon::S2_lsr_i_r), NewR)
  2079:           .addReg(L.Reg, {}, L.Sub)
  2080:           .addImm(16);
  2081:     }
  2082:   }
  2083:   if (NewR == 0)
  2084:     return false;
  2085:   HBS::replaceSubWithSub(RD.Reg, RD.Sub, NewR, 0, MRI);
  2086:   BT.put(BitTracker::RegisterRef(NewR), RC);
  2087:   return true;
  2088: }
  2089: 
  2090: // If MI is equivalent to a combine(.L/.H, .L/.H) replace with with the
  2091: // combine.
  2092: bool BitSimplification::genCombineHalf(MachineInstr *MI,
  2093:       BitTracker::RegisterRef RD, const BitTracker::RegisterCell &RC) {
  2094:   RegHalf L, H;
  2095:   // Check for combine h/l
  2096:   if (!matchHalf(RD.Reg, RC, 0, L) || !matchHalf(RD.Reg, RC, 16, H))
  2097:     return false;
  2098:   // Do nothing if this is just a reg copy.
  2099:   if (L.Reg == H.Reg && L.Sub == H.Sub && !H.Low && L.Low)
  2100:     return false;
  2101: 
  2102:   unsigned Opc = MI->getOpcode();
  2103:   unsigned COpc = getCombineOpcode(H.Low, L.Low);
  2104:   if (COpc == Opc)
  2105:     return false;
  2106:   if (!validateReg(H, COpc, 1) || !validateReg(L, COpc, 2))
  2107:     return false;
  2108: 
  2109:   MachineBasicBlock &B = *MI->getParent();
  2110:   DebugLoc DL = MI->getDebugLoc();
  2111:   Register NewR = MRI.createVirtualRegister(&Hexagon::IntRegsRegClass);
  2112:   auto At = MI->isPHI() ? B.getFirstNonPHI()
  2113:                         : MachineBasicBlock::iterator(MI);
  2114:   BuildMI(B, At, DL, HII.get(COpc), NewR)
  2115:       .addReg(H.Reg, {}, H.Sub)
  2116:       .addReg(L.Reg, {}, L.Sub);
  2117:   HBS::replaceSubWithSub(RD.Reg, RD.Sub, NewR, 0, MRI);
  2118:   BT.put(BitTracker::RegisterRef(NewR), RC);
  2119:   return true;
  2120: }
  2121: 
  2122: // If MI resets high bits of a register and keeps the lower ones, replace it
  2123: // with zero-extend byte/half, and-immediate, or extractu, as appropriate.
  2124: bool BitSimplification::genExtractLow(MachineInstr *MI,
  2125:       BitTracker::RegisterRef RD, const BitTracker::RegisterCell &RC) {
  2126:   unsigned Opc = MI->getOpcode();
  2127:   switch (Opc) {
  2128:     case Hexagon::A2_zxtb:
  2129:     case Hexagon::A2_zxth:
  2130:     case Hexagon::S2_extractu:
  2131:       return false;
  2132:   }
  2133:   if (Opc == Hexagon::A2_andir && MI->getOperand(2).isImm()) {
  2134:     int32_t Imm = MI->getOperand(2).getImm();
  2135:     if (isInt<10>(Imm))
  2136:       return false;
  2137:   }
  2138: 
  2139:   if (MI->hasUnmodeledSideEffects() || MI->isInlineAsm())
  2140:     return false;
  2141:   unsigned W = RC.width();
  2142:   while (W > 0 && RC[W-1].is(0))
  2143:     W--;
  2144:   if (W == 0 || W == RC.width())
  2145:     return false;
  2146:   unsigned NewOpc = (W == 8)  ? Hexagon::A2_zxtb
  2147:                   : (W == 16) ? Hexagon::A2_zxth
  2148:                   : (W < 10)  ? Hexagon::A2_andir
  2149:                   : Hexagon::S2_extractu;
  2150:   MachineBasicBlock &B = *MI->getParent();
  2151:   DebugLoc DL = MI->getDebugLoc();
  2152: 
  2153:   for (auto &Op : MI->uses()) {
  2154:     if (!Op.isReg())
  2155:       continue;
  2156:     BitTracker::RegisterRef RS = Op;
  2157:     if (!BT.has(RS.Reg))
  2158:       continue;
  2159:     const BitTracker::RegisterCell &SC = BT.lookup(RS.Reg);
  2160:     unsigned BN, BW;
  2161:     if (!HBS::getSubregMask(RS, BN, BW, MRI))
  2162:       continue;
  2163:     if (BW < W || !HBS::isEqual(RC, 0, SC, BN, W))
  2164:       continue;
  2165:     if (!validateReg(RS, NewOpc, 1))
  2166:       continue;
  2167: 
  2168:     Register NewR = MRI.createVirtualRegister(&Hexagon::IntRegsRegClass);
  2169:     auto At = MI->isPHI() ? B.getFirstNonPHI()
  2170:                           : MachineBasicBlock::iterator(MI);
  2171:     auto MIB =
  2172:         BuildMI(B, At, DL, HII.get(NewOpc), NewR).addReg(RS.Reg, {}, RS.Sub);
  2173:     if (NewOpc == Hexagon::A2_andir)
  2174:       MIB.addImm((1 << W) - 1);
  2175:     else if (NewOpc == Hexagon::S2_extractu)
  2176:       MIB.addImm(W).addImm(0);
  2177:     HBS::replaceSubWithSub(RD.Reg, RD.Sub, NewR, 0, MRI);
  2178:     BT.put(BitTracker::RegisterRef(NewR), RC);
  2179:     return true;
  2180:   }
  2181:   return false;
  2182: }
  2183: 
  2184: bool BitSimplification::genBitSplit(MachineInstr *MI,
  2185:       BitTracker::RegisterRef RD, const BitTracker::RegisterCell &RC,
  2186:       const RegisterSet &AVs) {
  2187:   if (!GenBitSplit)
  2188:     return false;
  2189:   if (MaxBitSplit.getNumOccurrences()) {
  2190:     if (CountBitSplit >= MaxBitSplit)
  2191:       return false;
  2192:   }
  2193: 
  2194:   unsigned Opc = MI->getOpcode();
  2195:   switch (Opc) {
  2196:     case Hexagon::A4_bitsplit:
  2197:     case Hexagon::A4_bitspliti:
  2198:       return false;
  2199:   }
  2200: 
```
- EN: It declares or implements routines such as llvm_unreachable, removeOperand, setDesc, addOperand, ... (21 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明或实现了 llvm_unreachable, removeOperand, setDesc, addOperand, ... (21 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 2201-2400 / 第 2201-2400 行

```cpp
  2201:   unsigned W = RC.width();
  2202:   if (W != 32)
  2203:     return false;
  2204: 
  2205:   auto ctlz = [] (const BitTracker::RegisterCell &C) -> unsigned {
  2206:     unsigned Z = C.width();
  2207:     while (Z > 0 && C[Z-1].is(0))
  2208:       --Z;
  2209:     return C.width() - Z;
  2210:   };
  2211: 
  2212:   // Count the number of leading zeros in the target RC.
  2213:   unsigned Z = ctlz(RC);
  2214:   if (Z == 0 || Z == W)
  2215:     return false;
  2216: 
  2217:   // A simplistic analysis: assume the source register (the one being split)
  2218:   // is fully unknown, and that all its bits are self-references.
  2219:   const BitTracker::BitValue &B0 = RC[0];
  2220:   if (B0.Type != BitTracker::BitValue::Ref)
  2221:     return false;
  2222: 
  2223:   unsigned SrcR = B0.RefI.Reg;
  2224:   unsigned SrcSR = 0;
  2225:   unsigned Pos = B0.RefI.Pos;
  2226: 
  2227:   // All the non-zero bits should be consecutive bits from the same register.
  2228:   for (unsigned i = 1; i < W-Z; ++i) {
  2229:     const BitTracker::BitValue &V = RC[i];
  2230:     if (V.Type != BitTracker::BitValue::Ref)
  2231:       return false;
  2232:     if (V.RefI.Reg != SrcR || V.RefI.Pos != Pos+i)
  2233:       return false;
  2234:   }
  2235: 
  2236:   // Now, find the other bitfield among AVs.
  2237:   for (unsigned S = AVs.find_first(); S; S = AVs.find_next(S)) {
  2238:     // The number of leading zeros here should be the number of trailing
  2239:     // non-zeros in RC.
  2240:     unsigned SRC = MRI.getRegClass(S)->getID();
  2241:     if (SRC != Hexagon::IntRegsRegClassID &&
  2242:         SRC != Hexagon::DoubleRegsRegClassID)
  2243:       continue;
  2244:     if (!BT.has(S))
  2245:       continue;
  2246:     const BitTracker::RegisterCell &SC = BT.lookup(S);
  2247:     if (SC.width() != W || ctlz(SC) != W-Z)
  2248:       continue;
  2249:     // The Z lower bits should now match SrcR.
  2250:     const BitTracker::BitValue &S0 = SC[0];
  2251:     if (S0.Type != BitTracker::BitValue::Ref || S0.RefI.Reg != SrcR)
  2252:       continue;
  2253:     unsigned P = S0.RefI.Pos;
  2254: 
  2255:     if (Pos <= P && (Pos + W-Z) != P)
  2256:       continue;
  2257:     if (P < Pos && (P + Z) != Pos)
  2258:       continue;
  2259:     // The starting bitfield position must be at a subregister boundary.
  2260:     if (std::min(P, Pos) != 0 && std::min(P, Pos) != 32)
  2261:       continue;
  2262: 
  2263:     unsigned I;
  2264:     for (I = 1; I < Z; ++I) {
  2265:       const BitTracker::BitValue &V = SC[I];
  2266:       if (V.Type != BitTracker::BitValue::Ref)
  2267:         break;
  2268:       if (V.RefI.Reg != SrcR || V.RefI.Pos != P+I)
  2269:         break;
  2270:     }
  2271:     if (I != Z)
  2272:       continue;
  2273: 
  2274:     // Generate bitsplit where S is defined.
  2275:     if (MaxBitSplit.getNumOccurrences())
  2276:       CountBitSplit++;
  2277:     MachineInstr *DefS = MRI.getVRegDef(S);
  2278:     assert(DefS != nullptr);
  2279:     DebugLoc DL = DefS->getDebugLoc();
  2280:     MachineBasicBlock &B = *DefS->getParent();
  2281:     auto At = DefS->isPHI() ? B.getFirstNonPHI()
  2282:                             : MachineBasicBlock::iterator(DefS);
  2283:     if (MRI.getRegClass(SrcR)->getID() == Hexagon::DoubleRegsRegClassID)
  2284:       SrcSR = (std::min(Pos, P) == 32) ? Hexagon::isub_hi : Hexagon::isub_lo;
  2285:     if (!validateReg({SrcR,SrcSR}, Hexagon::A4_bitspliti, 1))
  2286:       continue;
  2287:     unsigned ImmOp = Pos <= P ? W-Z : Z;
  2288: 
  2289:     // Find an existing bitsplit instruction if one already exists.
  2290:     unsigned NewR = 0;
  2291:     for (MachineInstr *In : NewMIs) {
  2292:       if (In->getOpcode() != Hexagon::A4_bitspliti)
  2293:         continue;
  2294:       MachineOperand &Op1 = In->getOperand(1);
  2295:       if (Op1.getReg() != SrcR || Op1.getSubReg() != SrcSR)
  2296:         continue;
  2297:       if (In->getOperand(2).getImm() != ImmOp)
  2298:         continue;
  2299:       // Check if the target register is available here.
  2300:       MachineOperand &Op0 = In->getOperand(0);
  2301:       MachineInstr *DefI = MRI.getVRegDef(Op0.getReg());
  2302:       assert(DefI != nullptr);
  2303:       if (!MDT.dominates(DefI, &*At))
  2304:         continue;
  2305: 
  2306:       // Found one that can be reused.
  2307:       assert(Op0.getSubReg() == 0);
  2308:       NewR = Op0.getReg();
  2309:       break;
  2310:     }
  2311:     if (!NewR) {
  2312:       NewR = MRI.createVirtualRegister(&Hexagon::DoubleRegsRegClass);
  2313:       auto NewBS = BuildMI(B, At, DL, HII.get(Hexagon::A4_bitspliti), NewR)
  2314:                        .addReg(SrcR, {}, SrcSR)
  2315:                        .addImm(ImmOp);
  2316:       NewMIs.push_back(NewBS);
  2317:     }
  2318:     if (Pos <= P) {
  2319:       HBS::replaceRegWithSub(RD.Reg, NewR, Hexagon::isub_lo, MRI);
  2320:       HBS::replaceRegWithSub(S,      NewR, Hexagon::isub_hi, MRI);
  2321:     } else {
  2322:       HBS::replaceRegWithSub(S,      NewR, Hexagon::isub_lo, MRI);
  2323:       HBS::replaceRegWithSub(RD.Reg, NewR, Hexagon::isub_hi, MRI);
  2324:     }
  2325:     return true;
  2326:   }
  2327: 
  2328:   return false;
  2329: }
  2330: 
  2331: // Check for tstbit simplification opportunity, where the bit being checked
  2332: // can be tracked back to another register. For example:
  2333: //   %2 = S2_lsr_i_r  %1, 5
  2334: //   %3 = S2_tstbit_i %2, 0
  2335: // =>
  2336: //   %3 = S2_tstbit_i %1, 5
  2337: bool BitSimplification::simplifyTstbit(MachineInstr *MI,
  2338:       BitTracker::RegisterRef RD, const BitTracker::RegisterCell &RC) {
  2339:   unsigned Opc = MI->getOpcode();
  2340:   if (Opc != Hexagon::S2_tstbit_i)
  2341:     return false;
  2342: 
  2343:   unsigned BN = MI->getOperand(2).getImm();
  2344:   BitTracker::RegisterRef RS = MI->getOperand(1);
  2345:   unsigned F, W;
  2346:   DebugLoc DL = MI->getDebugLoc();
  2347:   if (!BT.has(RS.Reg) || !HBS::getSubregMask(RS, F, W, MRI))
  2348:     return false;
  2349:   MachineBasicBlock &B = *MI->getParent();
  2350:   auto At = MI->isPHI() ? B.getFirstNonPHI()
  2351:                         : MachineBasicBlock::iterator(MI);
  2352: 
  2353:   const BitTracker::RegisterCell &SC = BT.lookup(RS.Reg);
  2354:   const BitTracker::BitValue &V = SC[F+BN];
  2355:   if (V.Type == BitTracker::BitValue::Ref && V.RefI.Reg != RS.Reg) {
  2356:     const TargetRegisterClass *TC = MRI.getRegClass(V.RefI.Reg);
  2357:     // Need to map V.RefI.Reg to a 32-bit register, i.e. if it is
  2358:     // a double register, need to use a subregister and adjust bit
  2359:     // number.
  2360:     unsigned P = std::numeric_limits<unsigned>::max();
  2361:     BitTracker::RegisterRef RR(V.RefI.Reg, 0);
  2362:     if (TC == &Hexagon::DoubleRegsRegClass) {
  2363:       P = V.RefI.Pos;
  2364:       RR.Sub = Hexagon::isub_lo;
  2365:       if (P >= 32) {
  2366:         P -= 32;
  2367:         RR.Sub = Hexagon::isub_hi;
  2368:       }
  2369:     } else if (TC == &Hexagon::IntRegsRegClass) {
  2370:       P = V.RefI.Pos;
  2371:     }
  2372:     if (P != std::numeric_limits<unsigned>::max()) {
  2373:       Register NewR = MRI.createVirtualRegister(&Hexagon::PredRegsRegClass);
  2374:       BuildMI(B, At, DL, HII.get(Hexagon::S2_tstbit_i), NewR)
  2375:           .addReg(RR.Reg, {}, RR.Sub)
  2376:           .addImm(P);
  2377:       HBS::replaceReg(RD.Reg, NewR, MRI);
  2378:       BT.put(NewR, RC);
  2379:       return true;
  2380:     }
  2381:   } else if (V.is(0) || V.is(1)) {
  2382:     Register NewR = MRI.createVirtualRegister(&Hexagon::PredRegsRegClass);
  2383:     unsigned NewOpc = V.is(0) ? Hexagon::PS_false : Hexagon::PS_true;
  2384:     BuildMI(B, At, DL, HII.get(NewOpc), NewR);
  2385:     HBS::replaceReg(RD.Reg, NewR, MRI);
  2386:     return true;
  2387:   }
  2388: 
  2389:   return false;
  2390: }
  2391: 
  2392: // Detect whether RD is a bitfield extract (sign- or zero-extended) of
  2393: // some register from the AVs set. Create a new corresponding instruction
  2394: // at the location of MI. The intent is to recognize situations where
  2395: // a sequence of instructions performs an operation that is equivalent to
  2396: // an extract operation, such as a shift left followed by a shift right.
  2397: bool BitSimplification::simplifyExtractLow(MachineInstr *MI,
  2398:       BitTracker::RegisterRef RD, const BitTracker::RegisterCell &RC,
  2399:       const RegisterSet &AVs) {
  2400:   if (!GenExtract)
```
- EN: It declares or implements routines such as width, ctlz, find_next, getRegClass, ... (24 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明或实现了 width, ctlz, find_next, getRegClass, ... (24 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 2401-2600 / 第 2401-2600 行

```cpp
  2401:     return false;
  2402:   if (MaxExtract.getNumOccurrences()) {
  2403:     if (CountExtract >= MaxExtract)
  2404:       return false;
  2405:     CountExtract++;
  2406:   }
  2407: 
  2408:   unsigned W = RC.width();
  2409:   unsigned RW = W;
  2410:   unsigned Len;
  2411:   bool Signed;
  2412: 
  2413:   // The code is mostly class-independent, except for the part that generates
  2414:   // the extract instruction, and establishes the source register (in case it
  2415:   // needs to use a subregister).
  2416:   const TargetRegisterClass *FRC = HBS::getFinalVRegClass(RD, MRI);
  2417:   if (FRC != &Hexagon::IntRegsRegClass && FRC != &Hexagon::DoubleRegsRegClass)
  2418:     return false;
  2419:   assert(RD.Sub == 0);
  2420: 
  2421:   // Observation:
  2422:   // If the cell has a form of 00..0xx..x with k zeros and n remaining
  2423:   // bits, this could be an extractu of the n bits, but it could also be
  2424:   // an extractu of a longer field which happens to have 0s in the top
  2425:   // bit positions.
  2426:   // The same logic applies to sign-extended fields.
  2427:   //
  2428:   // Do not check for the extended extracts, since it would expand the
  2429:   // search space quite a bit. The search may be expensive as it is.
  2430: 
  2431:   const BitTracker::BitValue &TopV = RC[W-1];
  2432: 
  2433:   // Eliminate candidates that have self-referential bits, since they
  2434:   // cannot be extracts from other registers. Also, skip registers that
  2435:   // have compile-time constant values.
  2436:   bool IsConst = true;
  2437:   for (unsigned I = 0; I != W; ++I) {
  2438:     const BitTracker::BitValue &V = RC[I];
  2439:     if (V.Type == BitTracker::BitValue::Ref && V.RefI.Reg == RD.Reg)
  2440:       return false;
  2441:     IsConst = IsConst && (V.is(0) || V.is(1));
  2442:   }
  2443:   if (IsConst)
  2444:     return false;
  2445: 
  2446:   if (TopV.is(0) || TopV.is(1)) {
  2447:     bool S = TopV.is(1);
  2448:     for (--W; W > 0 && RC[W-1].is(S); --W)
  2449:       ;
  2450:     Len = W;
  2451:     Signed = S;
  2452:     // The sign bit must be a part of the field being extended.
  2453:     if (Signed)
  2454:       ++Len;
  2455:   } else {
  2456:     // This could still be a sign-extended extract.
  2457:     assert(TopV.Type == BitTracker::BitValue::Ref);
  2458:     if (TopV.RefI.Reg == RD.Reg || TopV.RefI.Pos == W-1)
  2459:       return false;
  2460:     for (--W; W > 0 && RC[W-1] == TopV; --W)
  2461:       ;
  2462:     // The top bits of RC are copies of TopV. One occurrence of TopV will
  2463:     // be a part of the field.
  2464:     Len = W + 1;
  2465:     Signed = true;
  2466:   }
  2467: 
  2468:   // This would be just a copy. It should be handled elsewhere.
  2469:   if (Len == RW)
  2470:     return false;
  2471: 
  2472:   LLVM_DEBUG({
  2473:     dbgs() << __func__ << " on reg: " << printReg(RD.Reg, &HRI, RD.Sub)
  2474:            << ", MI: " << *MI;
  2475:     dbgs() << "Cell: " << RC << '\n';
  2476:     dbgs() << "Expected bitfield size: " << Len << " bits, "
  2477:            << (Signed ? "sign" : "zero") << "-extended\n";
  2478:   });
  2479: 
  2480:   bool Changed = false;
  2481: 
  2482:   for (unsigned R = AVs.find_first(); R != 0; R = AVs.find_next(R)) {
  2483:     if (!BT.has(R))
  2484:       continue;
  2485:     const BitTracker::RegisterCell &SC = BT.lookup(R);
  2486:     unsigned SW = SC.width();
  2487: 
  2488:     // The source can be longer than the destination, as long as its size is
  2489:     // a multiple of the size of the destination. Also, we would need to be
  2490:     // able to refer to the subregister in the source that would be of the
  2491:     // same size as the destination, but only check the sizes here.
  2492:     if (SW < RW || (SW % RW) != 0)
  2493:       continue;
  2494: 
  2495:     // The field can start at any offset in SC as long as it contains Len
  2496:     // bits and does not cross subregister boundary (if the source register
  2497:     // is longer than the destination).
  2498:     unsigned Off = 0;
  2499:     while (Off <= SW-Len) {
  2500:       unsigned OE = (Off+Len)/RW;
  2501:       if (OE != Off/RW) {
  2502:         // The assumption here is that if the source (R) is longer than the
  2503:         // destination, then the destination is a sequence of words of
  2504:         // size RW, and each such word in R can be accessed via a subregister.
  2505:         //
  2506:         // If the beginning and the end of the field cross the subregister
  2507:         // boundary, advance to the next subregister.
  2508:         Off = OE*RW;
  2509:         continue;
  2510:       }
  2511:       if (HBS::isEqual(RC, 0, SC, Off, Len))
  2512:         break;
  2513:       ++Off;
  2514:     }
  2515: 
  2516:     if (Off > SW-Len)
  2517:       continue;
  2518: 
  2519:     // Found match.
  2520:     unsigned ExtOpc = 0;
  2521:     if (Off == 0) {
  2522:       if (Len == 8)
  2523:         ExtOpc = Signed ? Hexagon::A2_sxtb : Hexagon::A2_zxtb;
  2524:       else if (Len == 16)
  2525:         ExtOpc = Signed ? Hexagon::A2_sxth : Hexagon::A2_zxth;
  2526:       else if (Len < 10 && !Signed)
  2527:         ExtOpc = Hexagon::A2_andir;
  2528:     }
  2529:     if (ExtOpc == 0) {
  2530:       ExtOpc =
  2531:           Signed ? (RW == 32 ? Hexagon::S4_extract  : Hexagon::S4_extractp)
  2532:                  : (RW == 32 ? Hexagon::S2_extractu : Hexagon::S2_extractup);
  2533:     }
  2534:     unsigned SR = 0;
  2535:     // This only recognizes isub_lo and isub_hi.
  2536:     if (RW != SW && RW*2 != SW)
  2537:       continue;
  2538:     if (RW != SW)
  2539:       SR = (Off/RW == 0) ? Hexagon::isub_lo : Hexagon::isub_hi;
  2540:     Off = Off % RW;
  2541: 
  2542:     if (!validateReg({R,SR}, ExtOpc, 1))
  2543:       continue;
  2544: 
  2545:     // Don't generate the same instruction as the one being optimized.
  2546:     if (MI->getOpcode() == ExtOpc) {
  2547:       // All possible ExtOpc's have the source in operand(1).
  2548:       const MachineOperand &SrcOp = MI->getOperand(1);
  2549:       if (SrcOp.getReg() == R)
  2550:         continue;
  2551:     }
  2552: 
  2553:     DebugLoc DL = MI->getDebugLoc();
  2554:     MachineBasicBlock &B = *MI->getParent();
  2555:     Register NewR = MRI.createVirtualRegister(FRC);
  2556:     auto At = MI->isPHI() ? B.getFirstNonPHI()
  2557:                           : MachineBasicBlock::iterator(MI);
  2558:     auto MIB = BuildMI(B, At, DL, HII.get(ExtOpc), NewR).addReg(R, {}, SR);
  2559:     switch (ExtOpc) {
  2560:       case Hexagon::A2_sxtb:
  2561:       case Hexagon::A2_zxtb:
  2562:       case Hexagon::A2_sxth:
  2563:       case Hexagon::A2_zxth:
  2564:         break;
  2565:       case Hexagon::A2_andir:
  2566:         MIB.addImm((1u << Len) - 1);
  2567:         break;
  2568:       case Hexagon::S4_extract:
  2569:       case Hexagon::S2_extractu:
  2570:       case Hexagon::S4_extractp:
  2571:       case Hexagon::S2_extractup:
  2572:         MIB.addImm(Len)
  2573:            .addImm(Off);
  2574:         break;
  2575:       default:
  2576:         llvm_unreachable("Unexpected opcode");
  2577:     }
  2578: 
  2579:     HBS::replaceReg(RD.Reg, NewR, MRI);
  2580:     BT.put(BitTracker::RegisterRef(NewR), RC);
  2581:     Changed = true;
  2582:     break;
  2583:   }
  2584: 
  2585:   return Changed;
  2586: }
  2587: 
  2588: bool BitSimplification::simplifyRCmp0(MachineInstr *MI,
  2589:       BitTracker::RegisterRef RD) {
  2590:   unsigned Opc = MI->getOpcode();
  2591:   if (Opc != Hexagon::A4_rcmpeqi && Opc != Hexagon::A4_rcmpneqi)
  2592:     return false;
  2593:   MachineOperand &CmpOp = MI->getOperand(2);
  2594:   if (!CmpOp.isImm() || CmpOp.getImm() != 0)
  2595:     return false;
  2596: 
  2597:   const TargetRegisterClass *FRC = HBS::getFinalVRegClass(RD, MRI);
  2598:   if (FRC != &Hexagon::IntRegsRegClass && FRC != &Hexagon::DoubleRegsRegClass)
  2599:     return false;
  2600:   assert(RD.Sub == 0);
```
- EN: It declares or implements routines such as width, register, assert, is, ... (19 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明或实现了 width, register, assert, is, ... (19 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 2601-2800 / 第 2601-2800 行

```cpp
  2601: 
  2602:   MachineBasicBlock &B = *MI->getParent();
  2603:   const DebugLoc &DL = MI->getDebugLoc();
  2604:   auto At = MI->isPHI() ? B.getFirstNonPHI()
  2605:                         : MachineBasicBlock::iterator(MI);
  2606:   bool KnownZ = true;
  2607:   bool KnownNZ = false;
  2608: 
  2609:   BitTracker::RegisterRef SR = MI->getOperand(1);
  2610:   if (!BT.has(SR.Reg))
  2611:     return false;
  2612:   const BitTracker::RegisterCell &SC = BT.lookup(SR.Reg);
  2613:   unsigned F, W;
  2614:   if (!HBS::getSubregMask(SR, F, W, MRI))
  2615:     return false;
  2616: 
  2617:   for (uint16_t I = F; I != F+W; ++I) {
  2618:     const BitTracker::BitValue &V = SC[I];
  2619:     if (!V.is(0))
  2620:       KnownZ = false;
  2621:     if (V.is(1))
  2622:       KnownNZ = true;
  2623:   }
  2624: 
  2625:   auto ReplaceWithConst = [&](int C) {
  2626:     Register NewR = MRI.createVirtualRegister(FRC);
  2627:     BuildMI(B, At, DL, HII.get(Hexagon::A2_tfrsi), NewR)
  2628:       .addImm(C);
  2629:     HBS::replaceReg(RD.Reg, NewR, MRI);
  2630:     BitTracker::RegisterCell NewRC(W);
  2631:     for (uint16_t I = 0; I != W; ++I) {
  2632:       NewRC[I] = BitTracker::BitValue(C & 1);
  2633:       C = unsigned(C) >> 1;
  2634:     }
  2635:     BT.put(BitTracker::RegisterRef(NewR), NewRC);
  2636:     return true;
  2637:   };
  2638: 
  2639:   auto IsNonZero = [] (const MachineOperand &Op) {
  2640:     if (Op.isGlobal() || Op.isBlockAddress())
  2641:       return true;
  2642:     if (Op.isImm())
  2643:       return Op.getImm() != 0;
  2644:     if (Op.isCImm())
  2645:       return !Op.getCImm()->isZero();
  2646:     if (Op.isFPImm())
  2647:       return !Op.getFPImm()->isZero();
  2648:     return false;
  2649:   };
  2650: 
  2651:   auto IsZero = [] (const MachineOperand &Op) {
  2652:     if (Op.isGlobal() || Op.isBlockAddress())
  2653:       return false;
  2654:     if (Op.isImm())
  2655:       return Op.getImm() == 0;
  2656:     if (Op.isCImm())
  2657:       return Op.getCImm()->isZero();
  2658:     if (Op.isFPImm())
  2659:       return Op.getFPImm()->isZero();
  2660:     return false;
  2661:   };
  2662: 
  2663:   // If the source register is known to be 0 or non-0, the comparison can
  2664:   // be folded to a load of a constant.
  2665:   if (KnownZ || KnownNZ) {
  2666:     assert(KnownZ != KnownNZ && "Register cannot be both 0 and non-0");
  2667:     return ReplaceWithConst(KnownZ == (Opc == Hexagon::A4_rcmpeqi));
  2668:   }
  2669: 
  2670:   // Special case: if the compare comes from a C2_muxii, then we know the
  2671:   // two possible constants that can be the source value.
  2672:   MachineInstr *InpDef = MRI.getVRegDef(SR.Reg);
  2673:   if (!InpDef)
  2674:     return false;
  2675:   if (SR.Sub == 0 && InpDef->getOpcode() == Hexagon::C2_muxii) {
  2676:     MachineOperand &Src1 = InpDef->getOperand(2);
  2677:     MachineOperand &Src2 = InpDef->getOperand(3);
  2678:     // Check if both are non-zero.
  2679:     bool KnownNZ1 = IsNonZero(Src1), KnownNZ2 = IsNonZero(Src2);
  2680:     if (KnownNZ1 && KnownNZ2)
  2681:       return ReplaceWithConst(Opc == Hexagon::A4_rcmpneqi);
  2682:     // Check if both are zero.
  2683:     bool KnownZ1 = IsZero(Src1), KnownZ2 = IsZero(Src2);
  2684:     if (KnownZ1 && KnownZ2)
  2685:       return ReplaceWithConst(Opc == Hexagon::A4_rcmpeqi);
  2686: 
  2687:     // If for both operands we know that they are either 0 or non-0,
  2688:     // replace the comparison with a C2_muxii, using the same predicate
  2689:     // register, but with operands substituted with 0/1 accordingly.
  2690:     if ((KnownZ1 || KnownNZ1) && (KnownZ2 || KnownNZ2)) {
  2691:       Register NewR = MRI.createVirtualRegister(FRC);
  2692:       BuildMI(B, At, DL, HII.get(Hexagon::C2_muxii), NewR)
  2693:         .addReg(InpDef->getOperand(1).getReg())
  2694:         .addImm(KnownZ1 == (Opc == Hexagon::A4_rcmpeqi))
  2695:         .addImm(KnownZ2 == (Opc == Hexagon::A4_rcmpeqi));
  2696:       HBS::replaceReg(RD.Reg, NewR, MRI);
  2697:       // Create a new cell with only the least significant bit unknown.
  2698:       BitTracker::RegisterCell NewRC(W);
  2699:       NewRC[0] = BitTracker::BitValue::self();
  2700:       NewRC.fill(1, W, BitTracker::BitValue::Zero);
  2701:       BT.put(BitTracker::RegisterRef(NewR), NewRC);
  2702:       return true;
  2703:     }
  2704:   }
  2705: 
  2706:   return false;
  2707: }
  2708: 
  2709: bool BitSimplification::processBlock(MachineBasicBlock &B,
  2710:       const RegisterSet &AVs) {
  2711:   if (!BT.reached(&B))
  2712:     return false;
  2713:   bool Changed = false;
  2714:   RegisterSet AVB = AVs;
  2715:   RegisterSet Defs;
  2716: 
  2717:   for (auto I = B.begin(), E = B.end(); I != E; ++I, AVB.insert(Defs)) {
  2718:     MachineInstr *MI = &*I;
  2719:     Defs.clear();
  2720:     HBS::getInstrDefs(*MI, Defs);
  2721: 
  2722:     unsigned Opc = MI->getOpcode();
  2723:     if (Opc == TargetOpcode::COPY || Opc == TargetOpcode::REG_SEQUENCE)
  2724:       continue;
  2725: 
  2726:     if (MI->mayStore()) {
  2727:       bool T = genStoreUpperHalf(MI);
  2728:       T = T || genStoreImmediate(MI);
  2729:       Changed |= T;
  2730:       continue;
  2731:     }
  2732: 
  2733:     if (Defs.count() != 1)
  2734:       continue;
  2735:     const MachineOperand &Op0 = MI->getOperand(0);
  2736:     if (!Op0.isReg() || !Op0.isDef())
  2737:       continue;
  2738:     BitTracker::RegisterRef RD = Op0;
  2739:     if (!BT.has(RD.Reg))
  2740:       continue;
  2741:     const TargetRegisterClass *FRC = HBS::getFinalVRegClass(RD, MRI);
  2742:     const BitTracker::RegisterCell &RC = BT.lookup(RD.Reg);
  2743: 
  2744:     if (FRC->getID() == Hexagon::DoubleRegsRegClassID) {
  2745:       bool T = genPackhl(MI, RD, RC);
  2746:       T = T || simplifyExtractLow(MI, RD, RC, AVB);
  2747:       Changed |= T;
  2748:       continue;
  2749:     }
  2750: 
  2751:     if (FRC->getID() == Hexagon::IntRegsRegClassID) {
  2752:       bool T = genBitSplit(MI, RD, RC, AVB);
  2753:       T = T || simplifyExtractLow(MI, RD, RC, AVB);
  2754:       T = T || genExtractHalf(MI, RD, RC);
  2755:       T = T || genCombineHalf(MI, RD, RC);
  2756:       T = T || genExtractLow(MI, RD, RC);
  2757:       T = T || simplifyRCmp0(MI, RD);
  2758:       Changed |= T;
  2759:       continue;
  2760:     }
  2761: 
  2762:     if (FRC->getID() == Hexagon::PredRegsRegClassID) {
  2763:       bool T = simplifyTstbit(MI, RD, RC);
  2764:       Changed |= T;
  2765:       continue;
  2766:     }
  2767:   }
  2768:   return Changed;
  2769: }
  2770: 
  2771: bool HexagonBitSimplify::runOnMachineFunction(MachineFunction &MF) {
  2772:   if (skipFunction(MF.getFunction()))
  2773:     return false;
  2774: 
  2775:   auto &HST = MF.getSubtarget<HexagonSubtarget>();
  2776:   auto &HRI = *HST.getRegisterInfo();
  2777:   auto &HII = *HST.getInstrInfo();
  2778: 
  2779:   MDT = &getAnalysis<MachineDominatorTreeWrapperPass>().getDomTree();
  2780:   MachineRegisterInfo &MRI = MF.getRegInfo();
  2781:   bool Changed;
  2782: 
  2783:   Changed = DeadCodeElimination(MF, *MDT).run();
  2784: 
  2785:   const HexagonEvaluator HE(HRI, MRI, HII, MF);
  2786:   BitTracker BT(HE, MF);
  2787:   LLVM_DEBUG(BT.trace(true));
  2788:   BT.run();
  2789: 
  2790:   MachineBasicBlock &Entry = MF.front();
  2791: 
  2792:   RegisterSet AIG;  // Available registers for IG.
  2793:   ConstGeneration ImmG(BT, HII, MRI);
  2794:   Changed |= visitBlock(Entry, ImmG, AIG);
  2795: 
  2796:   RegisterSet ARE;  // Available registers for RIE.
  2797:   RedundantInstrElimination RIE(BT, HII, HRI, MRI);
  2798:   bool Ried = visitBlock(Entry, RIE, ARE);
  2799:   if (Ried) {
  2800:     Changed = true;
```
- EN: It declares or implements routines such as getParent, getDebugLoc, isPHI, getOperand, ... (49 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonBitSimplify, HexagonSubtarget, HexagonEvaluator, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getParent, getDebugLoc, isPHI, getOperand, ... (49 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonBitSimplify, HexagonSubtarget, HexagonEvaluator，说明了它与同级后端组件的连接关系。

### Lines 2801-3000 / 第 2801-3000 行

```cpp
  2801:     BT.run();
  2802:   }
  2803: 
  2804:   RegisterSet ACG;  // Available registers for CG.
  2805:   CopyGeneration CopyG(BT, HII, HRI, MRI);
  2806:   Changed |= visitBlock(Entry, CopyG, ACG);
  2807: 
  2808:   RegisterSet ACP;  // Available registers for CP.
  2809:   CopyPropagation CopyP(HRI, MRI);
  2810:   Changed |= visitBlock(Entry, CopyP, ACP);
  2811: 
  2812:   Changed = DeadCodeElimination(MF, *MDT).run() || Changed;
  2813: 
  2814:   BT.run();
  2815:   RegisterSet ABS;  // Available registers for BS.
  2816:   BitSimplification BitS(BT, *MDT, HII, HRI, MRI, MF);
  2817:   Changed |= visitBlock(Entry, BitS, ABS);
  2818: 
  2819:   Changed = DeadCodeElimination(MF, *MDT).run() || Changed;
  2820: 
  2821:   if (Changed) {
  2822:     for (auto &B : MF)
  2823:       for (auto &I : B)
  2824:         I.clearKillInfo();
  2825:     DeadCodeElimination(MF, *MDT).run();
  2826:   }
  2827:   return Changed;
  2828: }
  2829: 
  2830: // Recognize loops where the code at the end of the loop matches the code
  2831: // before the entry of the loop, and the matching code is such that is can
  2832: // be simplified. This pass relies on the bit simplification above and only
  2833: // prepares code in a way that can be handled by the bit simplification.
  2834: //
  2835: // This is the motivating testcase (and explanation):
  2836: //
  2837: // {
  2838: //   loop0(.LBB0_2, r1)      // %for.body.preheader
  2839: //   r5:4 = memd(r0++#8)
  2840: // }
  2841: // {
  2842: //   r3 = lsr(r4, #16)
  2843: //   r7:6 = combine(r5, r5)
  2844: // }
  2845: // {
  2846: //   r3 = insert(r5, #16, #16)
  2847: //   r7:6 = vlsrw(r7:6, #16)
  2848: // }
  2849: // .LBB0_2:
  2850: // {
  2851: //   memh(r2+#4) = r5
  2852: //   memh(r2+#6) = r6            # R6 is really R5.H
  2853: // }
  2854: // {
  2855: //   r2 = add(r2, #8)
  2856: //   memh(r2+#0) = r4
  2857: //   memh(r2+#2) = r3            # R3 is really R4.H
  2858: // }
  2859: // {
  2860: //   r5:4 = memd(r0++#8)
  2861: // }
  2862: // {                             # "Shuffling" code that sets up R3 and R6
  2863: //   r3 = lsr(r4, #16)           # so that their halves can be stored in the
  2864: //   r7:6 = combine(r5, r5)      # next iteration. This could be folded into
  2865: // }                             # the stores if the code was at the beginning
  2866: // {                             # of the loop iteration. Since the same code
  2867: //   r3 = insert(r5, #16, #16)   # precedes the loop, it can actually be moved
  2868: //   r7:6 = vlsrw(r7:6, #16)     # there.
  2869: // }:endloop0
  2870: //
  2871: //
  2872: // The outcome:
  2873: //
  2874: // {
  2875: //   loop0(.LBB0_2, r1)
  2876: //   r5:4 = memd(r0++#8)
  2877: // }
  2878: // .LBB0_2:
  2879: // {
  2880: //   memh(r2+#4) = r5
  2881: //   memh(r2+#6) = r5.h
  2882: // }
  2883: // {
  2884: //   r2 = add(r2, #8)
  2885: //   memh(r2+#0) = r4
  2886: //   memh(r2+#2) = r4.h
  2887: // }
  2888: // {
  2889: //   r5:4 = memd(r0++#8)
  2890: // }:endloop0
  2891: 
  2892: namespace {
  2893: 
  2894:   class HexagonLoopRescheduling : public MachineFunctionPass {
  2895:   public:
  2896:     static char ID;
  2897: 
  2898:     HexagonLoopRescheduling() : MachineFunctionPass(ID) {}
  2899: 
  2900:     bool runOnMachineFunction(MachineFunction &MF) override;
  2901: 
  2902:   private:
  2903:     const HexagonInstrInfo *HII = nullptr;
  2904:     const HexagonRegisterInfo *HRI = nullptr;
  2905:     MachineRegisterInfo *MRI = nullptr;
  2906:     BitTracker *BTP = nullptr;
  2907: 
  2908:     struct LoopCand {
  2909:       LoopCand(MachineBasicBlock *lb, MachineBasicBlock *pb,
  2910:             MachineBasicBlock *eb) : LB(lb), PB(pb), EB(eb) {}
  2911: 
  2912:       MachineBasicBlock *LB, *PB, *EB;
  2913:     };
  2914:     using InstrList = std::vector<MachineInstr *>;
  2915:     struct InstrGroup {
  2916:       BitTracker::RegisterRef Inp, Out;
  2917:       InstrList Ins;
  2918:     };
  2919:     struct PhiInfo {
  2920:       PhiInfo(MachineInstr &P, MachineBasicBlock &B);
  2921: 
  2922:       unsigned DefR;
  2923:       BitTracker::RegisterRef LR, PR; // Loop Register, Preheader Register
  2924:       MachineBasicBlock *LB, *PB;     // Loop Block, Preheader Block
  2925:     };
  2926: 
  2927:     static unsigned getDefReg(const MachineInstr *MI);
  2928:     bool isConst(unsigned Reg) const;
  2929:     bool isBitShuffle(const MachineInstr *MI, unsigned DefR) const;
  2930:     bool isStoreInput(const MachineInstr *MI, unsigned DefR) const;
  2931:     bool isShuffleOf(unsigned OutR, unsigned InpR) const;
  2932:     bool isSameShuffle(unsigned OutR1, unsigned InpR1, unsigned OutR2,
  2933:         unsigned &InpR2) const;
  2934:     void moveGroup(InstrGroup &G, MachineBasicBlock &LB, MachineBasicBlock &PB,
  2935:         MachineBasicBlock::iterator At, unsigned OldPhiR, unsigned NewPredR);
  2936:     bool processLoop(LoopCand &C);
  2937:   };
  2938: 
  2939: } // end anonymous namespace
  2940: 
  2941: char HexagonLoopRescheduling::ID = 0;
  2942: 
  2943: INITIALIZE_PASS(HexagonLoopRescheduling, "hexagon-loop-resched-pass",
  2944:                 "Hexagon Loop Rescheduling", false, false)
  2945: 
  2946: HexagonLoopRescheduling::PhiInfo::PhiInfo(MachineInstr &P,
  2947:       MachineBasicBlock &B) {
  2948:   DefR = HexagonLoopRescheduling::getDefReg(&P);
  2949:   LB = &B;
  2950:   PB = nullptr;
  2951:   for (unsigned i = 1, n = P.getNumOperands(); i < n; i += 2) {
  2952:     const MachineOperand &OpB = P.getOperand(i+1);
  2953:     if (OpB.getMBB() == &B) {
  2954:       LR = P.getOperand(i);
  2955:       continue;
  2956:     }
  2957:     PB = OpB.getMBB();
  2958:     PR = P.getOperand(i);
  2959:   }
  2960: }
  2961: 
  2962: unsigned HexagonLoopRescheduling::getDefReg(const MachineInstr *MI) {
  2963:   RegisterSet Defs;
  2964:   HBS::getInstrDefs(*MI, Defs);
  2965:   if (Defs.count() != 1)
  2966:     return 0;
  2967:   return Defs.find_first();
  2968: }
  2969: 
  2970: bool HexagonLoopRescheduling::isConst(unsigned Reg) const {
  2971:   if (!BTP->has(Reg))
  2972:     return false;
  2973:   const BitTracker::RegisterCell &RC = BTP->lookup(Reg);
  2974:   for (unsigned i = 0, w = RC.width(); i < w; ++i) {
  2975:     const BitTracker::BitValue &V = RC[i];
  2976:     if (!V.is(0) && !V.is(1))
  2977:       return false;
  2978:   }
  2979:   return true;
  2980: }
  2981: 
  2982: bool HexagonLoopRescheduling::isBitShuffle(const MachineInstr *MI,
  2983:       unsigned DefR) const {
  2984:   unsigned Opc = MI->getOpcode();
  2985:   switch (Opc) {
  2986:     case TargetOpcode::COPY:
  2987:     case Hexagon::S2_lsr_i_r:
  2988:     case Hexagon::S2_asr_i_r:
  2989:     case Hexagon::S2_asl_i_r:
  2990:     case Hexagon::S2_lsr_i_p:
  2991:     case Hexagon::S2_asr_i_p:
  2992:     case Hexagon::S2_asl_i_p:
  2993:     case Hexagon::S2_insert:
  2994:     case Hexagon::A2_or:
  2995:     case Hexagon::A2_orp:
  2996:     case Hexagon::A2_and:
  2997:     case Hexagon::A2_andp:
  2998:     case Hexagon::A2_combinew:
  2999:     case Hexagon::A4_combineri:
  3000:     case Hexagon::A4_combineir:
```
- EN: It opens namespaces (char) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as HexagonLoopRescheduling, LoopCand, InstrGroup, PhiInfo, which carry the state or API of this component. It defines declarative TableGen records like HexagonLoopRescheduling; these records are consumed by TableGen instead of executed directly. It declares or implements routines such as run, CopyG, visitBlock, CopyP, ... (28 total), translating Hexagon-specific policy into reusable code paths.
- CN: 这里打开了命名空间（char），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 HexagonLoopRescheduling, LoopCand, InstrGroup, PhiInfo 等类型，用来承载该组件的状态或接口。 这里定义了 HexagonLoopRescheduling 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里声明或实现了 run, CopyG, visitBlock, CopyP, ... (28 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

### Lines 3001-3200 / 第 3001-3200 行

```cpp
  3001:     case Hexagon::A2_combineii:
  3002:     case Hexagon::A4_combineii:
  3003:     case Hexagon::A2_combine_ll:
  3004:     case Hexagon::A2_combine_lh:
  3005:     case Hexagon::A2_combine_hl:
  3006:     case Hexagon::A2_combine_hh:
  3007:       return true;
  3008:   }
  3009:   return false;
  3010: }
  3011: 
  3012: bool HexagonLoopRescheduling::isStoreInput(const MachineInstr *MI,
  3013:       unsigned InpR) const {
  3014:   for (unsigned i = 0, n = MI->getNumOperands(); i < n; ++i) {
  3015:     const MachineOperand &Op = MI->getOperand(i);
  3016:     if (!Op.isReg())
  3017:       continue;
  3018:     if (Op.getReg() == InpR)
  3019:       return i == n-1;
  3020:   }
  3021:   return false;
  3022: }
  3023: 
  3024: bool HexagonLoopRescheduling::isShuffleOf(unsigned OutR, unsigned InpR) const {
  3025:   if (!BTP->has(OutR) || !BTP->has(InpR))
  3026:     return false;
  3027:   const BitTracker::RegisterCell &OutC = BTP->lookup(OutR);
  3028:   for (unsigned i = 0, w = OutC.width(); i < w; ++i) {
  3029:     const BitTracker::BitValue &V = OutC[i];
  3030:     if (V.Type != BitTracker::BitValue::Ref)
  3031:       continue;
  3032:     if (V.RefI.Reg != InpR)
  3033:       return false;
  3034:   }
  3035:   return true;
  3036: }
  3037: 
  3038: bool HexagonLoopRescheduling::isSameShuffle(unsigned OutR1, unsigned InpR1,
  3039:       unsigned OutR2, unsigned &InpR2) const {
  3040:   if (!BTP->has(OutR1) || !BTP->has(InpR1) || !BTP->has(OutR2))
  3041:     return false;
  3042:   const BitTracker::RegisterCell &OutC1 = BTP->lookup(OutR1);
  3043:   const BitTracker::RegisterCell &OutC2 = BTP->lookup(OutR2);
  3044:   unsigned W = OutC1.width();
  3045:   unsigned MatchR = 0;
  3046:   if (W != OutC2.width())
  3047:     return false;
  3048:   for (unsigned i = 0; i < W; ++i) {
  3049:     const BitTracker::BitValue &V1 = OutC1[i], &V2 = OutC2[i];
  3050:     if (V1.Type != V2.Type || V1.Type == BitTracker::BitValue::One)
  3051:       return false;
  3052:     if (V1.Type != BitTracker::BitValue::Ref)
  3053:       continue;
  3054:     if (V1.RefI.Pos != V2.RefI.Pos)
  3055:       return false;
  3056:     if (V1.RefI.Reg != InpR1)
  3057:       return false;
  3058:     if (V2.RefI.Reg == 0 || V2.RefI.Reg == OutR2)
  3059:       return false;
  3060:     if (!MatchR)
  3061:       MatchR = V2.RefI.Reg;
  3062:     else if (V2.RefI.Reg != MatchR)
  3063:       return false;
  3064:   }
  3065:   InpR2 = MatchR;
  3066:   return true;
  3067: }
  3068: 
  3069: void HexagonLoopRescheduling::moveGroup(InstrGroup &G, MachineBasicBlock &LB,
  3070:       MachineBasicBlock &PB, MachineBasicBlock::iterator At, unsigned OldPhiR,
  3071:       unsigned NewPredR) {
  3072:   DenseMap<unsigned,unsigned> RegMap;
  3073: 
  3074:   const TargetRegisterClass *PhiRC = MRI->getRegClass(NewPredR);
  3075:   Register PhiR = MRI->createVirtualRegister(PhiRC);
  3076:   BuildMI(LB, At, At->getDebugLoc(), HII->get(TargetOpcode::PHI), PhiR)
  3077:     .addReg(NewPredR)
  3078:     .addMBB(&PB)
  3079:     .addReg(G.Inp.Reg)
  3080:     .addMBB(&LB);
  3081:   RegMap.insert(std::make_pair(G.Inp.Reg, PhiR));
  3082: 
  3083:   for (const MachineInstr *SI : llvm::reverse(G.Ins)) {
  3084:     unsigned DR = getDefReg(SI);
  3085:     const TargetRegisterClass *RC = MRI->getRegClass(DR);
  3086:     Register NewDR = MRI->createVirtualRegister(RC);
  3087:     DebugLoc DL = SI->getDebugLoc();
  3088: 
  3089:     auto MIB = BuildMI(LB, At, DL, HII->get(SI->getOpcode()), NewDR);
  3090:     for (const MachineOperand &Op : SI->operands()) {
  3091:       if (!Op.isReg()) {
  3092:         MIB.add(Op);
  3093:         continue;
  3094:       }
  3095:       if (!Op.isUse())
  3096:         continue;
  3097:       unsigned UseR = RegMap[Op.getReg()];
  3098:       MIB.addReg(UseR, {}, Op.getSubReg());
  3099:     }
  3100:     RegMap.insert(std::make_pair(DR, NewDR));
  3101:   }
  3102: 
  3103:   HBS::replaceReg(OldPhiR, RegMap[G.Out.Reg], *MRI);
  3104: }
  3105: 
  3106: bool HexagonLoopRescheduling::processLoop(LoopCand &C) {
  3107:   LLVM_DEBUG(dbgs() << "Processing loop in " << printMBBReference(*C.LB)
  3108:                     << "\n");
  3109:   std::vector<PhiInfo> Phis;
  3110:   for (auto &I : *C.LB) {
  3111:     if (!I.isPHI())
  3112:       break;
  3113:     unsigned PR = getDefReg(&I);
  3114:     if (isConst(PR))
  3115:       continue;
  3116:     bool BadUse = false, GoodUse = false;
  3117:     for (const MachineOperand &MO : MRI->use_operands(PR)) {
  3118:       const MachineInstr *UseI = MO.getParent();
  3119:       if (UseI->getParent() != C.LB) {
  3120:         BadUse = true;
  3121:         break;
  3122:       }
  3123:       if (isBitShuffle(UseI, PR) || isStoreInput(UseI, PR))
  3124:         GoodUse = true;
  3125:     }
  3126:     if (BadUse || !GoodUse)
  3127:       continue;
  3128: 
  3129:     Phis.push_back(PhiInfo(I, *C.LB));
  3130:   }
  3131: 
  3132:   LLVM_DEBUG({
  3133:     dbgs() << "Phis: {";
  3134:     for (auto &I : Phis) {
  3135:       dbgs() << ' ' << printReg(I.DefR, HRI) << "=phi("
  3136:              << printReg(I.PR.Reg, HRI, I.PR.Sub) << ":b" << I.PB->getNumber()
  3137:              << ',' << printReg(I.LR.Reg, HRI, I.LR.Sub) << ":b"
  3138:              << I.LB->getNumber() << ')';
  3139:     }
  3140:     dbgs() << " }\n";
  3141:   });
  3142: 
  3143:   if (Phis.empty())
  3144:     return false;
  3145: 
  3146:   bool Changed = false;
  3147:   InstrList ShufIns;
  3148: 
  3149:   // Go backwards in the block: for each bit shuffling instruction, check
  3150:   // if that instruction could potentially be moved to the front of the loop:
  3151:   // the output of the loop cannot be used in a non-shuffling instruction
  3152:   // in this loop.
  3153:   for (MachineInstr &MI : llvm::reverse(*C.LB)) {
  3154:     if (MI.isTerminator())
  3155:       continue;
  3156:     if (MI.isPHI())
  3157:       break;
  3158: 
  3159:     RegisterSet Defs;
  3160:     HBS::getInstrDefs(MI, Defs);
  3161:     if (Defs.count() != 1)
  3162:       continue;
  3163:     Register DefR = Defs.find_first();
  3164:     if (!DefR.isVirtual())
  3165:       continue;
  3166:     if (!isBitShuffle(&MI, DefR))
  3167:       continue;
  3168: 
  3169:     bool BadUse = false;
  3170:     for (auto UI = MRI->use_begin(DefR), UE = MRI->use_end(); UI != UE; ++UI) {
  3171:       MachineInstr *UseI = UI->getParent();
  3172:       if (UseI->getParent() == C.LB) {
  3173:         if (UseI->isPHI()) {
  3174:           // If the use is in a phi node in this loop, then it should be
  3175:           // the value corresponding to the back edge.
  3176:           unsigned Idx = UI.getOperandNo();
  3177:           if (UseI->getOperand(Idx+1).getMBB() != C.LB)
  3178:             BadUse = true;
  3179:         } else {
  3180:           if (!llvm::is_contained(ShufIns, UseI))
  3181:             BadUse = true;
  3182:         }
  3183:       } else {
  3184:         // There is a use outside of the loop, but there is no epilog block
  3185:         // suitable for a copy-out.
  3186:         if (C.EB == nullptr)
  3187:           BadUse = true;
  3188:       }
  3189:       if (BadUse)
  3190:         break;
  3191:     }
  3192: 
  3193:     if (BadUse)
  3194:       continue;
  3195:     ShufIns.push_back(&MI);
  3196:   }
  3197: 
  3198:   // Partition the list of shuffling instructions into instruction groups,
  3199:   // where each group has to be moved as a whole (i.e. a group is a chain of
  3200:   // dependent instructions). A group produces a single live output register,
```
- EN: It declares or implements routines such as HexagonLoopRescheduling::isStoreInput, getOperand, HexagonLoopRescheduling::isShuffleOf, lookup, ... (23 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonLoopRescheduling, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonLoopRescheduling::isStoreInput, getOperand, HexagonLoopRescheduling::isShuffleOf, lookup, ... (23 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonLoopRescheduling，说明了它与同级后端组件的连接关系。

### Lines 3201-3370 / 第 3201-3370 行

```cpp
  3201:   // which is meant to be the input of the loop phi node (although this is
  3202:   // not checked here yet). It also uses a single register as its input,
  3203:   // which is some value produced in the loop body. After moving the group
  3204:   // to the beginning of the loop, that input register would need to be
  3205:   // the loop-carried register (through a phi node) instead of the (currently
  3206:   // loop-carried) output register.
  3207:   using InstrGroupList = std::vector<InstrGroup>;
  3208:   InstrGroupList Groups;
  3209: 
  3210:   for (unsigned i = 0, n = ShufIns.size(); i < n; ++i) {
  3211:     MachineInstr *SI = ShufIns[i];
  3212:     if (SI == nullptr)
  3213:       continue;
  3214: 
  3215:     InstrGroup G;
  3216:     G.Ins.push_back(SI);
  3217:     G.Out.Reg = getDefReg(SI);
  3218:     RegisterSet Inputs;
  3219:     HBS::getInstrUses(*SI, Inputs);
  3220: 
  3221:     for (unsigned j = i+1; j < n; ++j) {
  3222:       MachineInstr *MI = ShufIns[j];
  3223:       if (MI == nullptr)
  3224:         continue;
  3225:       RegisterSet Defs;
  3226:       HBS::getInstrDefs(*MI, Defs);
  3227:       // If this instruction does not define any pending inputs, skip it.
  3228:       if (!Defs.intersects(Inputs))
  3229:         continue;
  3230:       // Otherwise, add it to the current group and remove the inputs that
  3231:       // are defined by MI.
  3232:       G.Ins.push_back(MI);
  3233:       Inputs.remove(Defs);
  3234:       // Then add all registers used by MI.
  3235:       HBS::getInstrUses(*MI, Inputs);
  3236:       ShufIns[j] = nullptr;
  3237:     }
  3238: 
  3239:     // Only add a group if it requires at most one register.
  3240:     if (Inputs.count() > 1)
  3241:       continue;
  3242:     auto LoopInpEq = [G] (const PhiInfo &P) -> bool {
  3243:       return G.Out.Reg == P.LR.Reg;
  3244:     };
  3245:     if (llvm::none_of(Phis, LoopInpEq))
  3246:       continue;
  3247: 
  3248:     G.Inp.Reg = Inputs.find_first();
  3249:     Groups.push_back(G);
  3250:   }
  3251: 
  3252:   LLVM_DEBUG({
  3253:     for (unsigned i = 0, n = Groups.size(); i < n; ++i) {
  3254:       InstrGroup &G = Groups[i];
  3255:       dbgs() << "Group[" << i << "] inp: "
  3256:              << printReg(G.Inp.Reg, HRI, G.Inp.Sub)
  3257:              << "  out: " << printReg(G.Out.Reg, HRI, G.Out.Sub) << "\n";
  3258:       for (const MachineInstr *MI : G.Ins)
  3259:         dbgs() << "  " << MI;
  3260:     }
  3261:   });
  3262: 
  3263:   for (InstrGroup &G : Groups) {
  3264:     if (!isShuffleOf(G.Out.Reg, G.Inp.Reg))
  3265:       continue;
  3266:     auto LoopInpEq = [G] (const PhiInfo &P) -> bool {
  3267:       return G.Out.Reg == P.LR.Reg;
  3268:     };
  3269:     auto F = llvm::find_if(Phis, LoopInpEq);
  3270:     if (F == Phis.end())
  3271:       continue;
  3272:     unsigned PrehR = 0;
  3273:     if (!isSameShuffle(G.Out.Reg, G.Inp.Reg, F->PR.Reg, PrehR)) {
  3274:       const MachineInstr *DefPrehR = MRI->getVRegDef(F->PR.Reg);
  3275:       unsigned Opc = DefPrehR->getOpcode();
  3276:       if (Opc != Hexagon::A2_tfrsi && Opc != Hexagon::A2_tfrpi)
  3277:         continue;
  3278:       if (!DefPrehR->getOperand(1).isImm())
  3279:         continue;
  3280:       if (DefPrehR->getOperand(1).getImm() != 0)
  3281:         continue;
  3282:       const TargetRegisterClass *RC = MRI->getRegClass(G.Inp.Reg);
  3283:       if (RC != MRI->getRegClass(F->PR.Reg)) {
  3284:         PrehR = MRI->createVirtualRegister(RC);
  3285:         unsigned TfrI = (RC == &Hexagon::IntRegsRegClass) ? Hexagon::A2_tfrsi
  3286:                                                           : Hexagon::A2_tfrpi;
  3287:         auto T = C.PB->getFirstTerminator();
  3288:         DebugLoc DL = (T != C.PB->end()) ? T->getDebugLoc() : DebugLoc();
  3289:         BuildMI(*C.PB, T, DL, HII->get(TfrI), PrehR)
  3290:           .addImm(0);
  3291:       } else {
  3292:         PrehR = F->PR.Reg;
  3293:       }
  3294:     }
  3295:     // isSameShuffle could match with PrehR being of a wider class than
  3296:     // G.Inp.Reg, for example if G shuffles the low 32 bits of its input,
  3297:     // it would match for the input being a 32-bit register, and PrehR
  3298:     // being a 64-bit register (where the low 32 bits match). This could
  3299:     // be handled, but for now skip these cases.
  3300:     if (MRI->getRegClass(PrehR) != MRI->getRegClass(G.Inp.Reg))
  3301:       continue;
  3302:     moveGroup(G, *F->LB, *F->PB, F->LB->getFirstNonPHI(), F->DefR, PrehR);
  3303:     Changed = true;
  3304:   }
  3305: 
  3306:   return Changed;
  3307: }
  3308: 
  3309: bool HexagonLoopRescheduling::runOnMachineFunction(MachineFunction &MF) {
  3310:   if (skipFunction(MF.getFunction()))
  3311:     return false;
  3312: 
  3313:   auto &HST = MF.getSubtarget<HexagonSubtarget>();
  3314:   HII = HST.getInstrInfo();
  3315:   HRI = HST.getRegisterInfo();
  3316:   MRI = &MF.getRegInfo();
  3317:   const HexagonEvaluator HE(*HRI, *MRI, *HII, MF);
  3318:   BitTracker BT(HE, MF);
  3319:   LLVM_DEBUG(BT.trace(true));
  3320:   BT.run();
  3321:   BTP = &BT;
  3322: 
  3323:   std::vector<LoopCand> Cand;
  3324: 
  3325:   for (auto &B : MF) {
  3326:     if (B.pred_size() != 2 || B.succ_size() != 2)
  3327:       continue;
  3328:     MachineBasicBlock *PB = nullptr;
  3329:     bool IsLoop = false;
  3330:     for (MachineBasicBlock *Pred : B.predecessors()) {
  3331:       if (Pred != &B)
  3332:         PB = Pred;
  3333:       else
  3334:         IsLoop = true;
  3335:     }
  3336:     if (!IsLoop)
  3337:       continue;
  3338: 
  3339:     MachineBasicBlock *EB = nullptr;
  3340:     for (MachineBasicBlock *Succ : B.successors()) {
  3341:       if (Succ == &B)
  3342:         continue;
  3343:       // Set EP to the epilog block, if it has only 1 predecessor (i.e. the
  3344:       // edge from B to EP is non-critical.
  3345:       if (Succ->pred_size() == 1)
  3346:         EB = Succ;
  3347:       break;
  3348:     }
  3349: 
  3350:     Cand.push_back(LoopCand(&B, PB, EB));
  3351:   }
  3352: 
  3353:   bool Changed = false;
  3354:   for (auto &C : Cand)
  3355:     Changed |= processLoop(C);
  3356: 
  3357:   return Changed;
  3358: }
  3359: 
  3360: //===----------------------------------------------------------------------===//
  3361: //                         Public Constructor Functions
  3362: //===----------------------------------------------------------------------===//
  3363: 
  3364: FunctionPass *llvm::createHexagonLoopRescheduling() {
  3365:   return new HexagonLoopRescheduling();
  3366: }
  3367: 
  3368: FunctionPass *llvm::createHexagonBitSimplify() {
  3369:   return new HexagonBitSimplify();
  3370: }
```
- EN: It declares or implements routines such as push_back, getDefReg, HBS::getInstrUses, HBS::getInstrDefs, ... (28 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonLoopRescheduling, HexagonSubtarget, HexagonEvaluator, HexagonBitSimplify, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 push_back, getDefReg, HBS::getInstrUses, HBS::getInstrDefs, ... (28 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonLoopRescheduling, HexagonSubtarget, HexagonEvaluator, HexagonBitSimplify，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- register modeling / 寄存器建模
- instruction semantics / 指令语义
- MC-layer target description / MC 层目标描述
- subtarget feature gating / 子目标特性控制

## Dependencies / 依赖关系

- Direct includes / 直接包含: `BitTracker.h, Hexagon.h, HexagonBitTracker.h, HexagonInstrInfo.h, HexagonRegisterInfo.h, HexagonSubtarget.h, llvm/ADT/BitVector.h, llvm/ADT/SmallVector.h, llvm/ADT/StringRef.h, llvm/CodeGen/MachineBasicBlock.h, ... (23 total)`
- Hexagon symbols / Hexagon 符号: `HexagonBitSimplify, HexagonBitTracker, HexagonInstrInfo, HexagonRegisterInfo, HexagonSubtarget, HexagonEvaluator, HexagonLoopRescheduling`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
