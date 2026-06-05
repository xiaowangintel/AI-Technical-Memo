# HexagonISelDAGToDAG.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonISelDAGToDAG.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): This file defines an instruction selector for the Hexagon target.
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。 重点涉及指令语义与选择。

## Line-by-Line Analysis / 逐行分析

### Lines 1-200 / 第 1-200 行

```cpp
     1: //===-- HexagonISelDAGToDAG.cpp - A dag to dag inst selector for Hexagon --===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: //
     9: // This file defines an instruction selector for the Hexagon target.
    10: //
    11: //===----------------------------------------------------------------------===//
    12: 
    13: #include "HexagonISelDAGToDAG.h"
    14: #include "Hexagon.h"
    15: #include "HexagonISelLowering.h"
    16: #include "HexagonMachineFunctionInfo.h"
    17: #include "HexagonTargetMachine.h"
    18: #include "llvm/CodeGen/FunctionLoweringInfo.h"
    19: #include "llvm/CodeGen/MachineInstrBuilder.h"
    20: #include "llvm/CodeGen/SelectionDAGISel.h"
    21: #include "llvm/IR/Intrinsics.h"
    22: #include "llvm/IR/IntrinsicsHexagon.h"
    23: #include "llvm/Support/CommandLine.h"
    24: #include "llvm/Support/Debug.h"
    25: using namespace llvm;
    26: 
    27: #define DEBUG_TYPE "hexagon-isel"
    28: #define PASS_NAME "Hexagon DAG->DAG Pattern Instruction Selection"
    29: 
    30: static
    31: cl::opt<bool>
    32: EnableAddressRebalancing("isel-rebalance-addr", cl::Hidden, cl::init(true),
    33:   cl::desc("Rebalance address calculation trees to improve "
    34:           "instruction selection"));
    35: 
    36: // Rebalance only if this allows e.g. combining a GA with an offset or
    37: // factoring out a shift.
    38: static
    39: cl::opt<bool>
    40: RebalanceOnlyForOptimizations("rebalance-only-opt", cl::Hidden, cl::init(false),
    41:   cl::desc("Rebalance address tree only if this allows optimizations"));
    42: 
    43: static
    44: cl::opt<bool>
    45: RebalanceOnlyImbalancedTrees("rebalance-only-imbal", cl::Hidden,
    46:   cl::init(false), cl::desc("Rebalance address tree only if it is imbalanced"));
    47: 
    48: static cl::opt<bool> CheckSingleUse("hexagon-isel-su", cl::Hidden,
    49:   cl::init(true), cl::desc("Enable checking of SDNode's single-use status"));
    50: 
    51: //===----------------------------------------------------------------------===//
    52: // Instruction Selector Implementation
    53: //===----------------------------------------------------------------------===//
    54: 
    55: #define GET_DAGISEL_BODY HexagonDAGToDAGISel
    56: #include "HexagonGenDAGISel.inc"
    57: 
    58: namespace llvm {
    59: /// createHexagonISelDag - This pass converts a legalized DAG into a
    60: /// Hexagon-specific DAG, ready for instruction scheduling.
    61: FunctionPass *createHexagonISelDag(HexagonTargetMachine &TM,
    62:                                    CodeGenOptLevel OptLevel) {
    63:   return new HexagonDAGToDAGISelLegacy(TM, OptLevel);
    64: }
    65: }
    66: 
    67: HexagonDAGToDAGISelLegacy::HexagonDAGToDAGISelLegacy(HexagonTargetMachine &tm,
    68:                                                      CodeGenOptLevel OptLevel)
    69:     : SelectionDAGISelLegacy(
    70:           ID, std::make_unique<HexagonDAGToDAGISel>(tm, OptLevel)) {}
    71: 
    72: char HexagonDAGToDAGISelLegacy::ID = 0;
    73: 
    74: INITIALIZE_PASS(HexagonDAGToDAGISelLegacy, DEBUG_TYPE, PASS_NAME, false, false)
    75: 
    76: void HexagonDAGToDAGISel::SelectIndexedLoad(LoadSDNode *LD, const SDLoc &dl) {
    77:   SDValue Chain = LD->getChain();
    78:   SDValue Base = LD->getBasePtr();
    79:   SDValue Offset = LD->getOffset();
    80:   int32_t Inc = cast<ConstantSDNode>(Offset.getNode())->getSExtValue();
    81:   EVT LoadedVT = LD->getMemoryVT();
    82:   unsigned Opcode = 0;
    83: 
    84:   // Check for zero extended loads. Treat any-extend loads as zero extended
    85:   // loads.
    86:   ISD::LoadExtType ExtType = LD->getExtensionType();
    87:   bool IsZeroExt = (ExtType == ISD::ZEXTLOAD || ExtType == ISD::EXTLOAD);
    88:   bool IsValidInc = HII->isValidAutoIncImm(LoadedVT, Inc);
    89: 
    90:   assert(LoadedVT.isSimple());
    91:   switch (LoadedVT.getSimpleVT().SimpleTy) {
    92:   case MVT::i8:
    93:     if (IsZeroExt)
    94:       Opcode = IsValidInc ? Hexagon::L2_loadrub_pi : Hexagon::L2_loadrub_io;
    95:     else
    96:       Opcode = IsValidInc ? Hexagon::L2_loadrb_pi : Hexagon::L2_loadrb_io;
    97:     break;
    98:   case MVT::i16:
    99:     if (IsZeroExt)
   100:       Opcode = IsValidInc ? Hexagon::L2_loadruh_pi : Hexagon::L2_loadruh_io;
   101:     else
   102:       Opcode = IsValidInc ? Hexagon::L2_loadrh_pi : Hexagon::L2_loadrh_io;
   103:     break;
   104:   case MVT::i32:
   105:   case MVT::f32:
   106:   case MVT::v2i16:
   107:   case MVT::v4i8:
   108:     Opcode = IsValidInc ? Hexagon::L2_loadri_pi : Hexagon::L2_loadri_io;
   109:     break;
   110:   case MVT::i64:
   111:   case MVT::f64:
   112:   case MVT::v2i32:
   113:   case MVT::v4i16:
   114:   case MVT::v8i8:
   115:     Opcode = IsValidInc ? Hexagon::L2_loadrd_pi : Hexagon::L2_loadrd_io;
   116:     break;
   117:   case MVT::v64i8:
   118:   case MVT::v32i16:
   119:   case MVT::v16i32:
   120:   case MVT::v8i64:
   121:   case MVT::v128i8:
   122:   case MVT::v64i16:
   123:   case MVT::v32i32:
   124:   case MVT::v16i64:
   125:     if (isAlignedMemNode(LD)) {
   126:       if (LD->isNonTemporal())
   127:         Opcode = IsValidInc ? Hexagon::V6_vL32b_nt_pi : Hexagon::V6_vL32b_nt_ai;
   128:       else
   129:         Opcode = IsValidInc ? Hexagon::V6_vL32b_pi : Hexagon::V6_vL32b_ai;
   130:     } else {
   131:       Opcode = IsValidInc ? Hexagon::V6_vL32Ub_pi : Hexagon::V6_vL32Ub_ai;
   132:     }
   133:     break;
   134:   default:
   135:     llvm_unreachable("Unexpected memory type in indexed load");
   136:   }
   137: 
   138:   SDValue IncV = CurDAG->getSignedTargetConstant(Inc, dl, MVT::i32);
   139:   MachineMemOperand *MemOp = LD->getMemOperand();
   140: 
   141:   auto getExt64 = [this,ExtType] (MachineSDNode *N, const SDLoc &dl)
   142:         -> MachineSDNode* {
   143:     if (ExtType == ISD::ZEXTLOAD || ExtType == ISD::EXTLOAD) {
   144:       SDValue Zero = CurDAG->getTargetConstant(0, dl, MVT::i32);
   145:       return CurDAG->getMachineNode(Hexagon::A4_combineir, dl, MVT::i64,
   146:                                     Zero, SDValue(N, 0));
   147:     }
   148:     if (ExtType == ISD::SEXTLOAD)
   149:       return CurDAG->getMachineNode(Hexagon::A2_sxtw, dl, MVT::i64,
   150:                                     SDValue(N, 0));
   151:     return N;
   152:   };
   153: 
   154:   //                  Loaded value   Next address   Chain
   155:   SDValue From[3] = { SDValue(LD,0), SDValue(LD,1), SDValue(LD,2) };
   156:   SDValue To[3];
   157: 
   158:   EVT ValueVT = LD->getValueType(0);
   159:   if (ValueVT == MVT::i64 && ExtType != ISD::NON_EXTLOAD) {
   160:     // A load extending to i64 will actually produce i32, which will then
   161:     // need to be extended to i64.
   162:     assert(LoadedVT.getSizeInBits() <= 32);
   163:     ValueVT = MVT::i32;
   164:   }
   165: 
   166:   if (IsValidInc) {
   167:     MachineSDNode *L = CurDAG->getMachineNode(Opcode, dl, ValueVT,
   168:                                               MVT::i32, MVT::Other, Base,
   169:                                               IncV, Chain);
   170:     CurDAG->setNodeMemRefs(L, {MemOp});
   171:     To[1] = SDValue(L, 1); // Next address.
   172:     To[2] = SDValue(L, 2); // Chain.
   173:     // Handle special case for extension to i64.
   174:     if (LD->getValueType(0) == MVT::i64)
   175:       L = getExt64(L, dl);
   176:     To[0] = SDValue(L, 0); // Loaded (extended) value.
   177:   } else {
   178:     SDValue Zero = CurDAG->getTargetConstant(0, dl, MVT::i32);
   179:     MachineSDNode *L = CurDAG->getMachineNode(Opcode, dl, ValueVT, MVT::Other,
   180:                                               Base, Zero, Chain);
   181:     CurDAG->setNodeMemRefs(L, {MemOp});
   182:     To[2] = SDValue(L, 1); // Chain.
   183:     MachineSDNode *A = CurDAG->getMachineNode(Hexagon::A2_addi, dl, MVT::i32,
   184:                                               Base, IncV);
   185:     To[1] = SDValue(A, 0); // Next address.
   186:     // Handle special case for extension to i64.
   187:     if (LD->getValueType(0) == MVT::i64)
   188:       L = getExt64(L, dl);
   189:     To[0] = SDValue(L, 0); // Loaded (extended) value.
   190:   }
   191:   ReplaceUses(From, To, 3);
   192:   CurDAG->RemoveDeadNode(LD);
   193: }
   194: 
   195: MachineSDNode *HexagonDAGToDAGISel::LoadInstrForLoadIntrinsic(SDNode *IntN) {
   196:   if (IntN->getOpcode() != ISD::INTRINSIC_W_CHAIN)
   197:     return nullptr;
   198: 
   199:   SDLoc dl(IntN);
   200:   unsigned IntNo = IntN->getConstantOperandVal(1);
```
- EN: It imports headers such as HexagonISelDAGToDAG.h, Hexagon.h, HexagonISelLowering.h, HexagonMachineFunctionInfo.h, ... (13 total), establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as EnableAddressRebalancing, RebalanceOnlyForOptimizations, RebalanceOnlyImbalancedTrees, CheckSingleUse, ... (28 total), translating Hexagon-specific policy into reusable code paths.
- CN: 这里引入了 HexagonISelDAGToDAG.h, Hexagon.h, HexagonISelLowering.h, HexagonMachineFunctionInfo.h, ... (13 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 EnableAddressRebalancing, RebalanceOnlyForOptimizations, RebalanceOnlyImbalancedTrees, CheckSingleUse, ... (28 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

### Lines 201-400 / 第 201-400 行

```cpp
   201: 
   202:   static std::map<unsigned,unsigned> LoadPciMap = {
   203:     { Intrinsic::hexagon_circ_ldb,  Hexagon::L2_loadrb_pci  },
   204:     { Intrinsic::hexagon_circ_ldub, Hexagon::L2_loadrub_pci },
   205:     { Intrinsic::hexagon_circ_ldh,  Hexagon::L2_loadrh_pci  },
   206:     { Intrinsic::hexagon_circ_lduh, Hexagon::L2_loadruh_pci },
   207:     { Intrinsic::hexagon_circ_ldw,  Hexagon::L2_loadri_pci  },
   208:     { Intrinsic::hexagon_circ_ldd,  Hexagon::L2_loadrd_pci  },
   209:   };
   210:   auto FLC = LoadPciMap.find(IntNo);
   211:   if (FLC != LoadPciMap.end()) {
   212:     EVT ValTy = (IntNo == Intrinsic::hexagon_circ_ldd) ? MVT::i64 : MVT::i32;
   213:     EVT RTys[] = { ValTy, MVT::i32, MVT::Other };
   214:     // Operands: { Base, Increment, Modifier, Chain }
   215:     auto Inc = cast<ConstantSDNode>(IntN->getOperand(5));
   216:     SDValue I =
   217:         CurDAG->getSignedTargetConstant(Inc->getSExtValue(), dl, MVT::i32);
   218:     MachineSDNode *Res = CurDAG->getMachineNode(FLC->second, dl, RTys,
   219:           { IntN->getOperand(2), I, IntN->getOperand(4),
   220:             IntN->getOperand(0) });
   221:     return Res;
   222:   }
   223: 
   224:   return nullptr;
   225: }
   226: 
   227: SDNode *HexagonDAGToDAGISel::StoreInstrForLoadIntrinsic(MachineSDNode *LoadN,
   228:       SDNode *IntN) {
   229:   // The "LoadN" is just a machine load instruction. The intrinsic also
   230:   // involves storing it. Generate an appropriate store to the location
   231:   // given in the intrinsic's operand(3).
   232:   uint64_t F = HII->get(LoadN->getMachineOpcode()).TSFlags;
   233:   unsigned SizeBits = (F >> HexagonII::MemAccessSizePos) &
   234:                       HexagonII::MemAccesSizeMask;
   235:   unsigned Size = 1U << (SizeBits-1);
   236: 
   237:   SDLoc dl(IntN);
   238:   MachinePointerInfo PI;
   239:   SDValue TS;
   240:   SDValue Loc = IntN->getOperand(3);
   241: 
   242:   if (Size >= 4)
   243:     TS = CurDAG->getStore(SDValue(LoadN, 2), dl, SDValue(LoadN, 0), Loc, PI,
   244:                           Align(Size));
   245:   else
   246:     TS = CurDAG->getTruncStore(SDValue(LoadN, 2), dl, SDValue(LoadN, 0), Loc,
   247:                                PI, MVT::getIntegerVT(Size * 8), Align(Size));
   248: 
   249:   SDNode *StoreN;
   250:   {
   251:     HandleSDNode Handle(TS);
   252:     SelectStore(TS.getNode());
   253:     StoreN = Handle.getValue().getNode();
   254:   }
   255: 
   256:   // Load's results are { Loaded value, Updated pointer, Chain }
   257:   ReplaceUses(SDValue(IntN, 0), SDValue(LoadN, 1));
   258:   ReplaceUses(SDValue(IntN, 1), SDValue(StoreN, 0));
   259:   return StoreN;
   260: }
   261: 
   262: bool HexagonDAGToDAGISel::tryLoadOfLoadIntrinsic(LoadSDNode *N) {
   263:   // The intrinsics for load circ/brev perform two operations:
   264:   // 1. Load a value V from the specified location, using the addressing
   265:   //    mode corresponding to the intrinsic.
   266:   // 2. Store V into a specified location. This location is typically a
   267:   //    local, temporary object.
   268:   // In many cases, the program using these intrinsics will immediately
   269:   // load V again from the local object. In those cases, when certain
   270:   // conditions are met, the last load can be removed.
   271:   // This function identifies and optimizes this pattern. If the pattern
   272:   // cannot be optimized, it returns nullptr, which will cause the load
   273:   // to be selected separately from the intrinsic (which will be handled
   274:   // in SelectIntrinsicWChain).
   275: 
   276:   SDValue Ch = N->getOperand(0);
   277:   SDValue Loc = N->getOperand(1);
   278: 
   279:   // Assume that the load and the intrinsic are connected directly with a
   280:   // chain:
   281:   //   t1: i32,ch = int.load ..., ..., ..., Loc, ...    // <-- C
   282:   //   t2: i32,ch = load t1:1, Loc, ...
   283:   SDNode *C = Ch.getNode();
   284: 
   285:   if (C->getOpcode() != ISD::INTRINSIC_W_CHAIN)
   286:     return false;
   287: 
   288:   // The second load can only be eliminated if its extension type matches
   289:   // that of the load instruction corresponding to the intrinsic. The user
   290:   // can provide an address of an unsigned variable to store the result of
   291:   // a sign-extending intrinsic into (or the other way around).
   292:   ISD::LoadExtType IntExt;
   293:   switch (C->getConstantOperandVal(1)) {
   294:   case Intrinsic::hexagon_circ_ldub:
   295:   case Intrinsic::hexagon_circ_lduh:
   296:     IntExt = ISD::ZEXTLOAD;
   297:     break;
   298:   case Intrinsic::hexagon_circ_ldw:
   299:   case Intrinsic::hexagon_circ_ldd:
   300:     IntExt = ISD::NON_EXTLOAD;
   301:     break;
   302:   default:
   303:     IntExt = ISD::SEXTLOAD;
   304:     break;
   305:   }
   306:   if (N->getExtensionType() != IntExt)
   307:     return false;
   308: 
   309:   // Make sure the target location for the loaded value in the load intrinsic
   310:   // is the location from which LD (or N) is loading.
   311:   if (C->getNumOperands() < 4 || Loc.getNode() != C->getOperand(3).getNode())
   312:     return false;
   313: 
   314:   if (MachineSDNode *L = LoadInstrForLoadIntrinsic(C)) {
   315:     SDNode *S = StoreInstrForLoadIntrinsic(L, C);
   316:     SDValue F[] = { SDValue(N,0), SDValue(N,1), SDValue(C,0), SDValue(C,1) };
   317:     SDValue T[] = { SDValue(L,0), SDValue(S,0), SDValue(L,1), SDValue(S,0) };
   318:     ReplaceUses(F, T, std::size(T));
   319:     // This transformation will leave the intrinsic dead. If it remains in
   320:     // the DAG, the selection code will see it again, but without the load,
   321:     // and it will generate a store that is normally required for it.
   322:     CurDAG->RemoveDeadNode(C);
   323:     return true;
   324:   }
   325:   return false;
   326: }
   327: 
   328: // Convert the bit-reverse load intrinsic to appropriate target instruction.
   329: bool HexagonDAGToDAGISel::SelectBrevLdIntrinsic(SDNode *IntN) {
   330:   if (IntN->getOpcode() != ISD::INTRINSIC_W_CHAIN)
   331:     return false;
   332: 
   333:   const SDLoc &dl(IntN);
   334:   unsigned IntNo = IntN->getConstantOperandVal(1);
   335: 
   336:   static const std::map<unsigned, unsigned> LoadBrevMap = {
   337:     { Intrinsic::hexagon_L2_loadrb_pbr, Hexagon::L2_loadrb_pbr },
   338:     { Intrinsic::hexagon_L2_loadrub_pbr, Hexagon::L2_loadrub_pbr },
   339:     { Intrinsic::hexagon_L2_loadrh_pbr, Hexagon::L2_loadrh_pbr },
   340:     { Intrinsic::hexagon_L2_loadruh_pbr, Hexagon::L2_loadruh_pbr },
   341:     { Intrinsic::hexagon_L2_loadri_pbr, Hexagon::L2_loadri_pbr },
   342:     { Intrinsic::hexagon_L2_loadrd_pbr, Hexagon::L2_loadrd_pbr }
   343:   };
   344:   auto FLI = LoadBrevMap.find(IntNo);
   345:   if (FLI != LoadBrevMap.end()) {
   346:     EVT ValTy =
   347:         (IntNo == Intrinsic::hexagon_L2_loadrd_pbr) ? MVT::i64 : MVT::i32;
   348:     EVT RTys[] = { ValTy, MVT::i32, MVT::Other };
   349:     // Operands of Intrinsic: {chain, enum ID of intrinsic, baseptr,
   350:     // modifier}.
   351:     // Operands of target instruction: { Base, Modifier, Chain }.
   352:     MachineSDNode *Res = CurDAG->getMachineNode(
   353:         FLI->second, dl, RTys,
   354:         {IntN->getOperand(2), IntN->getOperand(3), IntN->getOperand(0)});
   355: 
   356:     MachineMemOperand *MemOp = cast<MemIntrinsicSDNode>(IntN)->getMemOperand();
   357:     CurDAG->setNodeMemRefs(Res, {MemOp});
   358: 
   359:     ReplaceUses(SDValue(IntN, 0), SDValue(Res, 0));
   360:     ReplaceUses(SDValue(IntN, 1), SDValue(Res, 1));
   361:     ReplaceUses(SDValue(IntN, 2), SDValue(Res, 2));
   362:     CurDAG->RemoveDeadNode(IntN);
   363:     return true;
   364:   }
   365:   return false;
   366: }
   367: 
   368: /// Generate a machine instruction node for the new circular buffer intrinsics.
   369: /// The new versions use a CSx register instead of the K field.
   370: bool HexagonDAGToDAGISel::SelectNewCircIntrinsic(SDNode *IntN) {
   371:   if (IntN->getOpcode() != ISD::INTRINSIC_W_CHAIN)
   372:     return false;
   373: 
   374:   SDLoc DL(IntN);
   375:   unsigned IntNo = IntN->getConstantOperandVal(1);
   376:   SmallVector<SDValue, 7> Ops;
   377: 
   378:   static std::map<unsigned,unsigned> LoadNPcMap = {
   379:     { Intrinsic::hexagon_L2_loadrub_pci, Hexagon::PS_loadrub_pci },
   380:     { Intrinsic::hexagon_L2_loadrb_pci, Hexagon::PS_loadrb_pci },
   381:     { Intrinsic::hexagon_L2_loadruh_pci, Hexagon::PS_loadruh_pci },
   382:     { Intrinsic::hexagon_L2_loadrh_pci, Hexagon::PS_loadrh_pci },
   383:     { Intrinsic::hexagon_L2_loadri_pci, Hexagon::PS_loadri_pci },
   384:     { Intrinsic::hexagon_L2_loadrd_pci, Hexagon::PS_loadrd_pci },
   385:     { Intrinsic::hexagon_L2_loadrub_pcr, Hexagon::PS_loadrub_pcr },
   386:     { Intrinsic::hexagon_L2_loadrb_pcr, Hexagon::PS_loadrb_pcr },
   387:     { Intrinsic::hexagon_L2_loadruh_pcr, Hexagon::PS_loadruh_pcr },
   388:     { Intrinsic::hexagon_L2_loadrh_pcr, Hexagon::PS_loadrh_pcr },
   389:     { Intrinsic::hexagon_L2_loadri_pcr, Hexagon::PS_loadri_pcr },
   390:     { Intrinsic::hexagon_L2_loadrd_pcr, Hexagon::PS_loadrd_pcr }
   391:   };
   392:   auto FLI = LoadNPcMap.find (IntNo);
   393:   if (FLI != LoadNPcMap.end()) {
   394:     EVT ValTy = MVT::i32;
   395:     if (IntNo == Intrinsic::hexagon_L2_loadrd_pci ||
   396:         IntNo == Intrinsic::hexagon_L2_loadrd_pcr)
   397:       ValTy = MVT::i64;
   398:     EVT RTys[] = { ValTy, MVT::i32, MVT::Other };
   399:     // Handle load.*_pci case which has 6 operands.
   400:     if (IntN->getNumOperands() == 6) {
```
- EN: It declares or implements routines such as find, cast<ConstantSDNode>, getSignedTargetConstant, HexagonDAGToDAGISel::StoreInstrForLoadIntrinsic, ... (21 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonDAGToDAGISel, HexagonII, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 find, cast<ConstantSDNode>, getSignedTargetConstant, HexagonDAGToDAGISel::StoreInstrForLoadIntrinsic, ... (21 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonDAGToDAGISel, HexagonII，说明了它与同级后端组件的连接关系。

### Lines 401-600 / 第 401-600 行

```cpp
   401:       auto Inc = cast<ConstantSDNode>(IntN->getOperand(3));
   402:       SDValue I = CurDAG->getTargetConstant(Inc->getSExtValue(), DL, MVT::i32);
   403:       // Operands: { Base, Increment, Modifier, Start, Chain }.
   404:       Ops = { IntN->getOperand(2), I, IntN->getOperand(4), IntN->getOperand(5),
   405:               IntN->getOperand(0) };
   406:     } else
   407:       // Handle load.*_pcr case which has 5 operands.
   408:       // Operands: { Base, Modifier, Start, Chain }.
   409:       Ops = { IntN->getOperand(2), IntN->getOperand(3), IntN->getOperand(4),
   410:               IntN->getOperand(0) };
   411:     MachineSDNode *Res = CurDAG->getMachineNode(FLI->second, DL, RTys, Ops);
   412:     ReplaceUses(SDValue(IntN, 0), SDValue(Res, 0));
   413:     ReplaceUses(SDValue(IntN, 1), SDValue(Res, 1));
   414:     ReplaceUses(SDValue(IntN, 2), SDValue(Res, 2));
   415:     CurDAG->RemoveDeadNode(IntN);
   416:     return true;
   417:   }
   418: 
   419:   static std::map<unsigned,unsigned> StoreNPcMap = {
   420:     { Intrinsic::hexagon_S2_storerb_pci, Hexagon::PS_storerb_pci },
   421:     { Intrinsic::hexagon_S2_storerh_pci, Hexagon::PS_storerh_pci },
   422:     { Intrinsic::hexagon_S2_storerf_pci, Hexagon::PS_storerf_pci },
   423:     { Intrinsic::hexagon_S2_storeri_pci, Hexagon::PS_storeri_pci },
   424:     { Intrinsic::hexagon_S2_storerd_pci, Hexagon::PS_storerd_pci },
   425:     { Intrinsic::hexagon_S2_storerb_pcr, Hexagon::PS_storerb_pcr },
   426:     { Intrinsic::hexagon_S2_storerh_pcr, Hexagon::PS_storerh_pcr },
   427:     { Intrinsic::hexagon_S2_storerf_pcr, Hexagon::PS_storerf_pcr },
   428:     { Intrinsic::hexagon_S2_storeri_pcr, Hexagon::PS_storeri_pcr },
   429:     { Intrinsic::hexagon_S2_storerd_pcr, Hexagon::PS_storerd_pcr }
   430:   };
   431:   auto FSI = StoreNPcMap.find (IntNo);
   432:   if (FSI != StoreNPcMap.end()) {
   433:     EVT RTys[] = { MVT::i32, MVT::Other };
   434:     // Handle store.*_pci case which has 7 operands.
   435:     if (IntN->getNumOperands() == 7) {
   436:       auto Inc = cast<ConstantSDNode>(IntN->getOperand(3));
   437:       SDValue I = CurDAG->getTargetConstant(Inc->getSExtValue(), DL, MVT::i32);
   438:       // Operands: { Base, Increment, Modifier, Value, Start, Chain }.
   439:       Ops = { IntN->getOperand(2), I, IntN->getOperand(4), IntN->getOperand(5),
   440:               IntN->getOperand(6), IntN->getOperand(0) };
   441:     } else
   442:       // Handle store.*_pcr case which has 6 operands.
   443:       // Operands: { Base, Modifier, Value, Start, Chain }.
   444:       Ops = { IntN->getOperand(2), IntN->getOperand(3), IntN->getOperand(4),
   445:               IntN->getOperand(5), IntN->getOperand(0) };
   446:     MachineSDNode *Res = CurDAG->getMachineNode(FSI->second, DL, RTys, Ops);
   447:     ReplaceUses(SDValue(IntN, 0), SDValue(Res, 0));
   448:     ReplaceUses(SDValue(IntN, 1), SDValue(Res, 1));
   449:     CurDAG->RemoveDeadNode(IntN);
   450:     return true;
   451:   }
   452: 
   453:   return false;
   454: }
   455: 
   456: void HexagonDAGToDAGISel::SelectLoad(SDNode *N) {
   457:   SDLoc dl(N);
   458:   LoadSDNode *LD = cast<LoadSDNode>(N);
   459: 
   460:   // Handle indexed loads.
   461:   ISD::MemIndexedMode AM = LD->getAddressingMode();
   462:   if (AM != ISD::UNINDEXED) {
   463:     SelectIndexedLoad(LD, dl);
   464:     return;
   465:   }
   466: 
   467:   // Handle patterns using circ/brev load intrinsics.
   468:   if (tryLoadOfLoadIntrinsic(LD))
   469:     return;
   470: 
   471:   SelectCode(LD);
   472: }
   473: 
   474: void HexagonDAGToDAGISel::SelectIndexedStore(StoreSDNode *ST, const SDLoc &dl) {
   475:   SDValue Chain = ST->getChain();
   476:   SDValue Base = ST->getBasePtr();
   477:   SDValue Offset = ST->getOffset();
   478:   SDValue Value = ST->getValue();
   479:   // Get the constant value.
   480:   int32_t Inc = cast<ConstantSDNode>(Offset.getNode())->getSExtValue();
   481:   EVT StoredVT = ST->getMemoryVT();
   482:   EVT ValueVT = Value.getValueType();
   483: 
   484:   bool IsValidInc = HII->isValidAutoIncImm(StoredVT, Inc);
   485:   unsigned Opcode = 0;
   486: 
   487:   assert(StoredVT.isSimple());
   488:   switch (StoredVT.getSimpleVT().SimpleTy) {
   489:   case MVT::i8:
   490:     Opcode = IsValidInc ? Hexagon::S2_storerb_pi : Hexagon::S2_storerb_io;
   491:     break;
   492:   case MVT::i16:
   493:     Opcode = IsValidInc ? Hexagon::S2_storerh_pi : Hexagon::S2_storerh_io;
   494:     break;
   495:   case MVT::i32:
   496:   case MVT::f32:
   497:   case MVT::v2i16:
   498:   case MVT::v4i8:
   499:     Opcode = IsValidInc ? Hexagon::S2_storeri_pi : Hexagon::S2_storeri_io;
   500:     break;
   501:   case MVT::i64:
   502:   case MVT::f64:
   503:   case MVT::v2i32:
   504:   case MVT::v4i16:
   505:   case MVT::v8i8:
   506:     Opcode = IsValidInc ? Hexagon::S2_storerd_pi : Hexagon::S2_storerd_io;
   507:     break;
   508:   case MVT::v64i8:
   509:   case MVT::v32i16:
   510:   case MVT::v16i32:
   511:   case MVT::v8i64:
   512:   case MVT::v128i8:
   513:   case MVT::v64i16:
   514:   case MVT::v32i32:
   515:   case MVT::v16i64:
   516:     if (isAlignedMemNode(ST)) {
   517:       if (ST->isNonTemporal())
   518:         Opcode = IsValidInc ? Hexagon::V6_vS32b_nt_pi : Hexagon::V6_vS32b_nt_ai;
   519:       else
   520:         Opcode = IsValidInc ? Hexagon::V6_vS32b_pi : Hexagon::V6_vS32b_ai;
   521:     } else {
   522:       Opcode = IsValidInc ? Hexagon::V6_vS32Ub_pi : Hexagon::V6_vS32Ub_ai;
   523:     }
   524:     break;
   525:   default:
   526:     llvm_unreachable("Unexpected memory type in indexed store");
   527:   }
   528: 
   529:   if (ST->isTruncatingStore() && ValueVT.getSizeInBits() == 64) {
   530:     assert(StoredVT.getSizeInBits() < 64 && "Not a truncating store");
   531:     Value = CurDAG->getTargetExtractSubreg(Hexagon::isub_lo,
   532:                                            dl, MVT::i32, Value);
   533:   }
   534: 
   535:   SDValue IncV = CurDAG->getSignedTargetConstant(Inc, dl, MVT::i32);
   536:   MachineMemOperand *MemOp = ST->getMemOperand();
   537: 
   538:   //                  Next address   Chain
   539:   SDValue From[2] = { SDValue(ST,0), SDValue(ST,1) };
   540:   SDValue To[2];
   541: 
   542:   if (IsValidInc) {
   543:     // Build post increment store.
   544:     SDValue Ops[] = { Base, IncV, Value, Chain };
   545:     MachineSDNode *S = CurDAG->getMachineNode(Opcode, dl, MVT::i32, MVT::Other,
   546:                                               Ops);
   547:     CurDAG->setNodeMemRefs(S, {MemOp});
   548:     To[0] = SDValue(S, 0);
   549:     To[1] = SDValue(S, 1);
   550:   } else {
   551:     SDValue Zero = CurDAG->getTargetConstant(0, dl, MVT::i32);
   552:     SDValue Ops[] = { Base, Zero, Value, Chain };
   553:     MachineSDNode *S = CurDAG->getMachineNode(Opcode, dl, MVT::Other, Ops);
   554:     CurDAG->setNodeMemRefs(S, {MemOp});
   555:     To[1] = SDValue(S, 0);
   556:     MachineSDNode *A = CurDAG->getMachineNode(Hexagon::A2_addi, dl, MVT::i32,
   557:                                               Base, IncV);
   558:     To[0] = SDValue(A, 0);
   559:   }
   560: 
   561:   ReplaceUses(From, To, 2);
   562:   CurDAG->RemoveDeadNode(ST);
   563: }
   564: 
   565: void HexagonDAGToDAGISel::SelectStore(SDNode *N) {
   566:   SDLoc dl(N);
   567:   StoreSDNode *ST = cast<StoreSDNode>(N);
   568: 
   569:   // Handle indexed stores.
   570:   ISD::MemIndexedMode AM = ST->getAddressingMode();
   571:   if (AM != ISD::UNINDEXED) {
   572:     SelectIndexedStore(ST, dl);
   573:     return;
   574:   }
   575: 
   576:   SelectCode(ST);
   577: }
   578: 
   579: void HexagonDAGToDAGISel::SelectSHL(SDNode *N) {
   580:   SDLoc dl(N);
   581:   SDValue Shl_0 = N->getOperand(0);
   582:   SDValue Shl_1 = N->getOperand(1);
   583: 
   584:   auto Default = [this,N] () -> void { SelectCode(N); };
   585: 
   586:   if (N->getValueType(0) != MVT::i32 || Shl_1.getOpcode() != ISD::Constant)
   587:     return Default();
   588: 
   589:   // RHS is const.
   590:   int32_t ShlConst = cast<ConstantSDNode>(Shl_1)->getSExtValue();
   591: 
   592:   if (Shl_0.getOpcode() == ISD::MUL) {
   593:     SDValue Mul_0 = Shl_0.getOperand(0); // Val
   594:     SDValue Mul_1 = Shl_0.getOperand(1); // Const
   595:     // RHS of mul is const.
   596:     if (ConstantSDNode *C = dyn_cast<ConstantSDNode>(Mul_1)) {
   597:       int32_t ValConst = static_cast<int32_t>(
   598:           static_cast<uint32_t>(C->getSExtValue()) << ShlConst);
   599:       if (isInt<9>(ValConst)) {
   600:         SDValue Val = CurDAG->getTargetConstant(ValConst, dl, MVT::i32);
```
- EN: It declares or implements routines such as cast<ConstantSDNode>, getTargetConstant, getMachineNode, ReplaceUses, ... (32 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonDAGToDAGISel, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 cast<ConstantSDNode>, getTargetConstant, getMachineNode, ReplaceUses, ... (32 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonDAGToDAGISel，说明了它与同级后端组件的连接关系。

### Lines 601-800 / 第 601-800 行

```cpp
   601:         SDNode *Result = CurDAG->getMachineNode(Hexagon::M2_mpysmi, dl,
   602:                                                 MVT::i32, Mul_0, Val);
   603:         ReplaceNode(N, Result);
   604:         return;
   605:       }
   606:     }
   607:     return Default();
   608:   }
   609: 
   610:   if (Shl_0.getOpcode() == ISD::SUB) {
   611:     SDValue Sub_0 = Shl_0.getOperand(0); // Const 0
   612:     SDValue Sub_1 = Shl_0.getOperand(1); // Val
   613:     if (ConstantSDNode *C1 = dyn_cast<ConstantSDNode>(Sub_0)) {
   614:       if (C1->getSExtValue() != 0 || Sub_1.getOpcode() != ISD::SHL)
   615:         return Default();
   616:       SDValue Shl2_0 = Sub_1.getOperand(0); // Val
   617:       SDValue Shl2_1 = Sub_1.getOperand(1); // Const
   618:       if (ConstantSDNode *C2 = dyn_cast<ConstantSDNode>(Shl2_1)) {
   619:         int32_t ValConst =
   620:             static_cast<int32_t>(1U << (ShlConst + C2->getSExtValue()));
   621:         if (isInt<9>(-ValConst)) {
   622:           SDValue Val =
   623:               CurDAG->getSignedTargetConstant(-ValConst, dl, MVT::i32);
   624:           SDNode *Result = CurDAG->getMachineNode(Hexagon::M2_mpysmi, dl,
   625:                                                   MVT::i32, Shl2_0, Val);
   626:           ReplaceNode(N, Result);
   627:           return;
   628:         }
   629:       }
   630:     }
   631:   }
   632: 
   633:   return Default();
   634: }
   635: 
   636: //
   637: // Handling intrinsics for circular load and bitreverse load.
   638: //
   639: void HexagonDAGToDAGISel::SelectIntrinsicWChain(SDNode *N) {
   640:   if (MachineSDNode *L = LoadInstrForLoadIntrinsic(N)) {
   641:     StoreInstrForLoadIntrinsic(L, N);
   642:     CurDAG->RemoveDeadNode(N);
   643:     return;
   644:   }
   645: 
   646:   // Handle bit-reverse load intrinsics.
   647:   if (SelectBrevLdIntrinsic(N))
   648:     return;
   649: 
   650:   if (SelectNewCircIntrinsic(N))
   651:     return;
   652: 
   653:   unsigned IntNo = N->getConstantOperandVal(1);
   654:   if (IntNo == Intrinsic::hexagon_V6_vgathermw ||
   655:       IntNo == Intrinsic::hexagon_V6_vgathermw_128B ||
   656:       IntNo == Intrinsic::hexagon_V6_vgathermh ||
   657:       IntNo == Intrinsic::hexagon_V6_vgathermh_128B ||
   658:       IntNo == Intrinsic::hexagon_V6_vgathermhw ||
   659:       IntNo == Intrinsic::hexagon_V6_vgathermhw_128B ||
   660:       IntNo == Intrinsic::hexagon_V6_vgather_vscattermh ||
   661:       IntNo == Intrinsic::hexagon_V6_vgather_vscattermh_128B) {
   662:     SelectV65Gather(N);
   663:     return;
   664:   }
   665:   if (IntNo == Intrinsic::hexagon_V6_vgathermwq ||
   666:       IntNo == Intrinsic::hexagon_V6_vgathermwq_128B ||
   667:       IntNo == Intrinsic::hexagon_V6_vgathermhq ||
   668:       IntNo == Intrinsic::hexagon_V6_vgathermhq_128B ||
   669:       IntNo == Intrinsic::hexagon_V6_vgathermhwq ||
   670:       IntNo == Intrinsic::hexagon_V6_vgathermhwq_128B) {
   671:     SelectV65GatherPred(N);
   672:     return;
   673:   }
   674: 
   675:   SelectCode(N);
   676: }
   677: 
   678: void HexagonDAGToDAGISel::SelectIntrinsicWOChain(SDNode *N) {
   679:   unsigned IID = N->getConstantOperandVal(0);
   680:   unsigned Bits;
   681:   switch (IID) {
   682:   case Intrinsic::hexagon_S2_vsplatrb:
   683:     Bits = 8;
   684:     break;
   685:   case Intrinsic::hexagon_S2_vsplatrh:
   686:     Bits = 16;
   687:     break;
   688:   case Intrinsic::hexagon_V6_vaddcarry:
   689:   case Intrinsic::hexagon_V6_vaddcarry_128B:
   690:   case Intrinsic::hexagon_V6_vsubcarry:
   691:   case Intrinsic::hexagon_V6_vsubcarry_128B:
   692:     SelectHVXDualOutput(N);
   693:     return;
   694:   default:
   695:     SelectCode(N);
   696:     return;
   697:   }
   698: 
   699:   SDValue V = N->getOperand(1);
   700:   SDValue U;
   701:   // Splat intrinsics.
   702:   if (keepsLowBits(V, Bits, U)) {
   703:     SDValue R = CurDAG->getNode(N->getOpcode(), SDLoc(N), N->getValueType(0),
   704:                                 N->getOperand(0), U);
   705:     ReplaceNode(N, R.getNode());
   706:     SelectCode(R.getNode());
   707:     return;
   708:   }
   709:   SelectCode(N);
   710: }
   711: 
   712: void HexagonDAGToDAGISel::SelectExtractSubvector(SDNode *N) {
   713:   SDValue Inp = N->getOperand(0);
   714:   MVT ResTy = N->getValueType(0).getSimpleVT();
   715:   unsigned Idx = N->getConstantOperandVal(1);
   716: 
   717:   [[maybe_unused]] MVT InpTy = Inp.getValueType().getSimpleVT();
   718:   [[maybe_unused]] unsigned ResLen = ResTy.getVectorNumElements();
   719:   assert(InpTy.getVectorElementType() == ResTy.getVectorElementType());
   720:   assert(2 * ResLen == InpTy.getVectorNumElements());
   721:   assert(ResTy.getSizeInBits() == 32);
   722:   assert(Idx == 0 || Idx == ResLen);
   723: 
   724:   unsigned SubReg = Idx == 0 ? Hexagon::isub_lo : Hexagon::isub_hi;
   725:   SDValue Ext = CurDAG->getTargetExtractSubreg(SubReg, SDLoc(N), ResTy, Inp);
   726: 
   727:   ReplaceNode(N, Ext.getNode());
   728: }
   729: 
   730: //
   731: // Map floating point constant values.
   732: //
   733: void HexagonDAGToDAGISel::SelectConstantFP(SDNode *N) {
   734:   SDLoc dl(N);
   735:   auto *CN = cast<ConstantFPSDNode>(N);
   736:   APInt A = CN->getValueAPF().bitcastToAPInt();
   737:   if (N->getValueType(0) == MVT::f32) {
   738:     SDValue V = CurDAG->getTargetConstant(A.getZExtValue(), dl, MVT::i32);
   739:     ReplaceNode(N, CurDAG->getMachineNode(Hexagon::A2_tfrsi, dl, MVT::f32, V));
   740:     return;
   741:   }
   742:   if (N->getValueType(0) == MVT::f64) {
   743:     SDValue V = CurDAG->getTargetConstant(A.getZExtValue(), dl, MVT::i64);
   744:     ReplaceNode(N, CurDAG->getMachineNode(Hexagon::CONST64, dl, MVT::f64, V));
   745:     return;
   746:   }
   747: 
   748:   SelectCode(N);
   749: }
   750: 
   751: //
   752: // Map boolean values.
   753: //
   754: void HexagonDAGToDAGISel::SelectConstant(SDNode *N) {
   755:   if (N->getValueType(0) == MVT::i1) {
   756:     assert(!(N->getAsZExtVal() >> 1));
   757:     unsigned Opc = (cast<ConstantSDNode>(N)->getSExtValue() != 0)
   758:                       ? Hexagon::PS_true
   759:                       : Hexagon::PS_false;
   760:     ReplaceNode(N, CurDAG->getMachineNode(Opc, SDLoc(N), MVT::i1));
   761:     return;
   762:   }
   763: 
   764:   SelectCode(N);
   765: }
   766: 
   767: void HexagonDAGToDAGISel::SelectFrameIndex(SDNode *N) {
   768:   MachineFrameInfo &MFI = MF->getFrameInfo();
   769:   const HexagonFrameLowering *HFI = HST->getFrameLowering();
   770:   int FX = cast<FrameIndexSDNode>(N)->getIndex();
   771:   Align StkA = HFI->getStackAlign();
   772:   Align MaxA = MFI.getMaxAlign();
   773:   SDValue FI = CurDAG->getTargetFrameIndex(FX, MVT::i32);
   774:   SDLoc DL(N);
   775:   SDValue Zero = CurDAG->getTargetConstant(0, DL, MVT::i32);
   776:   SDNode *R = nullptr;
   777: 
   778:   // Use PS_fi when:
   779:   // - the object is fixed, or
   780:   // - there are no objects with higher-than-default alignment, or
   781:   // - there are no dynamically allocated objects.
   782:   // Otherwise, use PS_fia.
   783:   if (FX < 0 || MaxA <= StkA || !MFI.hasVarSizedObjects()) {
   784:     R = CurDAG->getMachineNode(Hexagon::PS_fi, DL, MVT::i32, FI, Zero);
   785:   } else {
   786:     auto &HMFI = *MF->getInfo<HexagonMachineFunctionInfo>();
   787:     Register AR = HMFI.getStackAlignBaseReg();
   788:     SDValue CH = CurDAG->getEntryNode();
   789:     SDValue Ops[] = { CurDAG->getCopyFromReg(CH, DL, AR, MVT::i32), FI, Zero };
   790:     R = CurDAG->getMachineNode(Hexagon::PS_fia, DL, MVT::i32, Ops);
   791:   }
   792: 
   793:   ReplaceNode(N, R);
   794: }
   795: 
   796: void HexagonDAGToDAGISel::SelectAddSubCarry(SDNode *N) {
   797:   unsigned OpcCarry = N->getOpcode() == HexagonISD::ADDC ? Hexagon::A4_addp_c
   798:                                                          : Hexagon::A4_subp_c;
   799:   SDNode *C = CurDAG->getMachineNode(OpcCarry, SDLoc(N), N->getVTList(),
   800:                                      { N->getOperand(0), N->getOperand(1),
```
- EN: It declares or implements routines such as getMachineNode, ReplaceNode, Default, getOperand, ... (39 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonDAGToDAGISel, HexagonFrameLowering, HexagonMachineFunctionInfo, HexagonISD, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getMachineNode, ReplaceNode, Default, getOperand, ... (39 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonDAGToDAGISel, HexagonFrameLowering, HexagonMachineFunctionInfo, HexagonISD，说明了它与同级后端组件的连接关系。

### Lines 801-1000 / 第 801-1000 行

```cpp
   801:                                        N->getOperand(2) });
   802:   ReplaceNode(N, C);
   803: }
   804: 
   805: void HexagonDAGToDAGISel::SelectVAlign(SDNode *N) {
   806:   MVT ResTy = N->getValueType(0).getSimpleVT();
   807:   if (HST->isHVXVectorType(ResTy, true))
   808:     return SelectHvxVAlign(N);
   809: 
   810:   const SDLoc &dl(N);
   811:   unsigned VecLen = ResTy.getSizeInBits();
   812:   if (VecLen == 32) {
   813:     SDValue Ops[] = {
   814:       CurDAG->getTargetConstant(Hexagon::DoubleRegsRegClassID, dl, MVT::i32),
   815:       N->getOperand(0),
   816:       CurDAG->getTargetConstant(Hexagon::isub_hi, dl, MVT::i32),
   817:       N->getOperand(1),
   818:       CurDAG->getTargetConstant(Hexagon::isub_lo, dl, MVT::i32)
   819:     };
   820:     SDNode *R = CurDAG->getMachineNode(TargetOpcode::REG_SEQUENCE, dl,
   821:                                        MVT::i64, Ops);
   822: 
   823:     // Shift right by "(Addr & 0x3) * 8" bytes.
   824:     SDNode *C;
   825:     SDValue M0 = CurDAG->getTargetConstant(0x18, dl, MVT::i32);
   826:     SDValue M1 = CurDAG->getTargetConstant(0x03, dl, MVT::i32);
   827:     if (HST->useCompound()) {
   828:       C = CurDAG->getMachineNode(Hexagon::S4_andi_asl_ri, dl, MVT::i32,
   829:                                  M0, N->getOperand(2), M1);
   830:     } else {
   831:       SDNode *T = CurDAG->getMachineNode(Hexagon::S2_asl_i_r, dl, MVT::i32,
   832:                                          N->getOperand(2), M1);
   833:       C = CurDAG->getMachineNode(Hexagon::A2_andir, dl, MVT::i32,
   834:                                  SDValue(T, 0), M0);
   835:     }
   836:     SDNode *S = CurDAG->getMachineNode(Hexagon::S2_lsr_r_p, dl, MVT::i64,
   837:                                        SDValue(R, 0), SDValue(C, 0));
   838:     SDValue E = CurDAG->getTargetExtractSubreg(Hexagon::isub_lo, dl, ResTy,
   839:                                                SDValue(S, 0));
   840:     ReplaceNode(N, E.getNode());
   841:   } else {
   842:     assert(VecLen == 64);
   843:     SDNode *Pu = CurDAG->getMachineNode(Hexagon::C2_tfrrp, dl, MVT::v8i1,
   844:                                         N->getOperand(2));
   845:     SDNode *VA = CurDAG->getMachineNode(Hexagon::S2_valignrb, dl, ResTy,
   846:                                         N->getOperand(0), N->getOperand(1),
   847:                                         SDValue(Pu,0));
   848:     ReplaceNode(N, VA);
   849:   }
   850: }
   851: 
   852: void HexagonDAGToDAGISel::SelectVAlignAddr(SDNode *N) {
   853:   const SDLoc &dl(N);
   854:   SDValue A = N->getOperand(1);
   855:   int Mask = -cast<ConstantSDNode>(A.getNode())->getSExtValue();
   856:   assert(isPowerOf2_32(-Mask));
   857: 
   858:   SDValue M = CurDAG->getTargetConstant(Mask, dl, MVT::i32);
   859:   SDNode *AA = CurDAG->getMachineNode(Hexagon::A2_andir, dl, MVT::i32,
   860:                                       N->getOperand(0), M);
   861:   ReplaceNode(N, AA);
   862: }
   863: 
   864: // Handle these nodes here to avoid having to write patterns for all
   865: // combinations of input/output types. In all cases, the resulting
   866: // instruction is the same.
   867: void HexagonDAGToDAGISel::SelectTypecast(SDNode *N) {
   868:   SDValue Op = N->getOperand(0);
   869:   MVT OpTy = Op.getValueType().getSimpleVT();
   870:   SDNode *T = CurDAG->MorphNodeTo(N, N->getOpcode(),
   871:                                   CurDAG->getVTList(OpTy), {Op});
   872:   ReplaceNode(T, Op.getNode());
   873: }
   874: 
   875: void HexagonDAGToDAGISel::SelectP2D(SDNode *N) {
   876:   MVT ResTy = N->getValueType(0).getSimpleVT();
   877:   SDNode *T = CurDAG->getMachineNode(Hexagon::C2_mask, SDLoc(N), ResTy,
   878:                                      N->getOperand(0));
   879:   ReplaceNode(N, T);
   880: }
   881: 
   882: void HexagonDAGToDAGISel::SelectD2P(SDNode *N) {
   883:   const SDLoc &dl(N);
   884:   MVT ResTy = N->getValueType(0).getSimpleVT();
   885:   SDValue Zero = CurDAG->getTargetConstant(0, dl, MVT::i32);
   886:   SDNode *T = CurDAG->getMachineNode(Hexagon::A4_vcmpbgtui, dl, ResTy,
   887:                                      N->getOperand(0), Zero);
   888:   ReplaceNode(N, T);
   889: }
   890: 
   891: void HexagonDAGToDAGISel::SelectV2Q(SDNode *N) {
   892:   const SDLoc &dl(N);
   893:   MVT ResTy = N->getValueType(0).getSimpleVT();
   894:   // The argument to V2Q should be a single vector.
   895:   MVT OpTy = N->getOperand(0).getValueType().getSimpleVT(); (void)OpTy;
   896:   assert(HST->getVectorLength() * 8 == OpTy.getSizeInBits());
   897: 
   898:   SDValue C = CurDAG->getSignedTargetConstant(-1, dl, MVT::i32);
   899:   SDNode *R = CurDAG->getMachineNode(Hexagon::A2_tfrsi, dl, MVT::i32, C);
   900:   SDNode *T = CurDAG->getMachineNode(Hexagon::V6_vandvrt, dl, ResTy,
   901:                                      N->getOperand(0), SDValue(R,0));
   902:   ReplaceNode(N, T);
   903: }
   904: 
   905: void HexagonDAGToDAGISel::SelectQ2V(SDNode *N) {
   906:   const SDLoc &dl(N);
   907:   MVT ResTy = N->getValueType(0).getSimpleVT();
   908:   // The result of V2Q should be a single vector.
   909:   assert(HST->getVectorLength() * 8 == ResTy.getSizeInBits());
   910: 
   911:   SDValue C = CurDAG->getSignedTargetConstant(-1, dl, MVT::i32);
   912:   SDNode *R = CurDAG->getMachineNode(Hexagon::A2_tfrsi, dl, MVT::i32, C);
   913:   SDNode *T = CurDAG->getMachineNode(Hexagon::V6_vandqrt, dl, ResTy,
   914:                                      N->getOperand(0), SDValue(R,0));
   915:   ReplaceNode(N, T);
   916: }
   917: 
   918: void HexagonDAGToDAGISel::FDiv(SDNode *N) {
   919:   const SDLoc &dl(N);
   920:   ArrayRef<EVT> ResultType(N->value_begin(), N->value_end());
   921:   SmallVector<SDValue, 2> Ops;
   922:   Ops = {N->getOperand(0), N->getOperand(1)};
   923:   SDVTList VTs;
   924:   VTs = CurDAG->getVTList(MVT::f32, MVT::f32);
   925:   SDNode *ResScale = CurDAG->getMachineNode(Hexagon::F2_sfrecipa, dl, VTs, Ops);
   926:   SDNode *D = CurDAG->getMachineNode(Hexagon::F2_sffixupd, dl, MVT::f32, Ops);
   927: 
   928:   SDValue C = CurDAG->getTargetConstant(0x3f800000, dl, MVT::i32);
   929:   SDNode *constNode =
   930:       CurDAG->getMachineNode(Hexagon::A2_tfrsi, dl, MVT::f32, C);
   931: 
   932:   SDNode *n = CurDAG->getMachineNode(Hexagon::F2_sffixupn, dl, MVT::f32, Ops);
   933:   SDNode *Err = CurDAG->getMachineNode(Hexagon::F2_sffms_lib, dl, MVT::f32,
   934:                                        SDValue(constNode, 0), SDValue(D, 0),
   935:                                        SDValue(ResScale, 0));
   936:   SDNode *NewRec = CurDAG->getMachineNode(Hexagon::F2_sffma_lib, dl, MVT::f32,
   937:                                           SDValue(ResScale, 0), SDValue(Err, 0),
   938:                                           SDValue(ResScale, 0));
   939:   SDNode *newErr = CurDAG->getMachineNode(Hexagon::F2_sffms_lib, dl, MVT::f32,
   940:                                           SDValue(constNode, 0), SDValue(D, 0),
   941:                                           SDValue(NewRec, 0));
   942:   SDNode *q = CurDAG->getMachineNode(
   943:       Hexagon::A2_andir, dl, MVT::f32, SDValue(n, 0),
   944:       CurDAG->getTargetConstant(0x80000000, dl, MVT::i32));
   945:   SDNode *NewQ =
   946:       CurDAG->getMachineNode(Hexagon::F2_sffma_lib, dl, MVT::f32, SDValue(q, 0),
   947:                              SDValue(n, 0), SDValue(NewRec, 0));
   948:   SDNode *NNewRec = CurDAG->getMachineNode(
   949:       Hexagon::F2_sffma_lib, dl, MVT::f32, SDValue(NewRec, 0),
   950:       SDValue(newErr, 0), SDValue(NewRec, 0));
   951:   SDNode *qErr =
   952:       CurDAG->getMachineNode(Hexagon::F2_sffms_lib, dl, MVT::f32, SDValue(n, 0),
   953:                              SDValue(D, 0), SDValue(NewQ, 0));
   954:   SDNode *NNewQ = CurDAG->getMachineNode(Hexagon::F2_sffma_lib, dl, MVT::f32,
   955:                                          SDValue(NewQ, 0), SDValue(qErr, 0),
   956:                                          SDValue(NNewRec, 0));
   957: 
   958:   SDNode *NqErr =
   959:       CurDAG->getMachineNode(Hexagon::F2_sffms_lib, dl, MVT::f32, SDValue(n, 0),
   960:                              SDValue(NNewQ, 0), SDValue(D, 0));
   961:   std::array<SDValue, 4> temp1 = {SDValue(NNewQ, 0), SDValue(NqErr, 0),
   962:                                   SDValue(NNewRec, 0), SDValue(ResScale, 1)};
   963:   ArrayRef<SDValue> OpValue1(temp1);
   964:   SDNode *FinalNewQ =
   965:       CurDAG->getMachineNode(Hexagon::F2_sffma_sc, dl, MVT::f32, OpValue1);
   966:   ReplaceNode(N, FinalNewQ);
   967: }
   968: 
   969: void HexagonDAGToDAGISel::FastFDiv(SDNode *N) {
   970:   const SDLoc &dl(N);
   971:   ArrayRef<EVT> ResultType(N->value_begin(), N->value_end());
   972:   SmallVector<SDValue, 2> Ops;
   973:   Ops = {N->getOperand(0), N->getOperand(1)};
   974:   SDVTList VTs;
   975:   VTs = CurDAG->getVTList(MVT::f32, MVT::f32);
   976:   SDNode *ResScale = CurDAG->getMachineNode(Hexagon::F2_sfrecipa, dl, VTs, Ops);
   977:   SDNode *D = CurDAG->getMachineNode(Hexagon::F2_sffixupd, dl, MVT::f32, Ops);
   978: 
   979:   SDValue C = CurDAG->getTargetConstant(0x3f800000, dl, MVT::i32);
   980:   SDNode *constNode =
   981:       CurDAG->getMachineNode(Hexagon::A2_tfrsi, dl, MVT::f32, C);
   982: 
   983:   SDNode *n = CurDAG->getMachineNode(Hexagon::F2_sffixupn, dl, MVT::f32, Ops);
   984:   SDNode *Err = CurDAG->getMachineNode(Hexagon::F2_sffms_lib, dl, MVT::f32,
   985:                                        SDValue(constNode, 0), SDValue(D, 0),
   986:                                        SDValue(ResScale, 0));
   987:   SDNode *NewRec = CurDAG->getMachineNode(Hexagon::F2_sffma_lib, dl, MVT::f32,
   988:                                           SDValue(ResScale, 0), SDValue(Err, 0),
   989:                                           SDValue(ResScale, 0));
   990:   SDNode *newErr = CurDAG->getMachineNode(Hexagon::F2_sffms_lib, dl, MVT::f32,
   991:                                           SDValue(constNode, 0), SDValue(D, 0),
   992:                                           SDValue(NewRec, 0));
   993: 
   994:   SDNode *NNewRec = CurDAG->getMachineNode(
   995:       Hexagon::F2_sffma_lib, dl, MVT::f32, SDValue(NewRec, 0),
   996:       SDValue(newErr, 0), SDValue(NewRec, 0));
   997:   SDNode *FinalNewQ = CurDAG->getMachineNode(
   998:       Hexagon::F2_sfmpy, dl, MVT::f32, SDValue(NNewRec, 0), SDValue(n, 0));
   999:   ReplaceNode(N, FinalNewQ);
  1000: }
```
- EN: It declares or implements routines such as ReplaceNode, HexagonDAGToDAGISel::SelectVAlign, getValueType, dl, ... (23 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonDAGToDAGISel, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 ReplaceNode, HexagonDAGToDAGISel::SelectVAlign, getValueType, dl, ... (23 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonDAGToDAGISel，说明了它与同级后端组件的连接关系。

### Lines 1001-1200 / 第 1001-1200 行

```cpp
  1001: 
  1002: void HexagonDAGToDAGISel::SelectFDiv(SDNode *N) {
  1003:   if (N->getFlags().hasAllowReassociation())
  1004:     FastFDiv(N);
  1005:   else
  1006:     FDiv(N);
  1007: }
  1008: 
  1009: void HexagonDAGToDAGISel::Select(SDNode *N) {
  1010:   if (N->isMachineOpcode())
  1011:     return N->setNodeId(-1);  // Already selected.
  1012: 
  1013:   auto isHvxOp = [this](SDNode *N) {
  1014:     for (unsigned i = 0, e = N->getNumValues(); i != e; ++i) {
  1015:       if (HST->isHVXVectorType(N->getValueType(i), true))
  1016:         return true;
  1017:     }
  1018:     for (SDValue I : N->ops()) {
  1019:       if (HST->isHVXVectorType(I.getValueType(), true))
  1020:         return true;
  1021:     }
  1022:     return false;
  1023:   };
  1024: 
  1025:   if (HST->useHVXOps() && isHvxOp(N)) {
  1026:     switch (N->getOpcode()) {
  1027:     case ISD::EXTRACT_SUBVECTOR:  return SelectHvxExtractSubvector(N);
  1028:     case ISD::VECTOR_SHUFFLE:     return SelectHvxShuffle(N);
  1029: 
  1030:     case HexagonISD::VROR:        return SelectHvxRor(N);
  1031:     }
  1032:   }
  1033: 
  1034:   switch (N->getOpcode()) {
  1035:   case ISD::Constant:             return SelectConstant(N);
  1036:   case ISD::ConstantFP:           return SelectConstantFP(N);
  1037:   case ISD::FrameIndex:           return SelectFrameIndex(N);
  1038:   case ISD::SHL:                  return SelectSHL(N);
  1039:   case ISD::LOAD:                 return SelectLoad(N);
  1040:   case ISD::STORE:                return SelectStore(N);
  1041:   case ISD::INTRINSIC_W_CHAIN:    return SelectIntrinsicWChain(N);
  1042:   case ISD::INTRINSIC_WO_CHAIN:   return SelectIntrinsicWOChain(N);
  1043:   case ISD::EXTRACT_SUBVECTOR:    return SelectExtractSubvector(N);
  1044: 
  1045:   case HexagonISD::ADDC:
  1046:   case HexagonISD::SUBC:          return SelectAddSubCarry(N);
  1047:   case HexagonISD::VALIGN:        return SelectVAlign(N);
  1048:   case HexagonISD::VALIGNADDR:    return SelectVAlignAddr(N);
  1049:   case HexagonISD::TYPECAST:      return SelectTypecast(N);
  1050:   case HexagonISD::P2D:           return SelectP2D(N);
  1051:   case HexagonISD::D2P:           return SelectD2P(N);
  1052:   case HexagonISD::Q2V:           return SelectQ2V(N);
  1053:   case HexagonISD::V2Q:           return SelectV2Q(N);
  1054:   case ISD::FDIV:
  1055:     return SelectFDiv(N);
  1056:   }
  1057: 
  1058:   SelectCode(N);
  1059: }
  1060: 
  1061: bool HexagonDAGToDAGISel::SelectInlineAsmMemoryOperand(
  1062:     const SDValue &Op, InlineAsm::ConstraintCode ConstraintID,
  1063:     std::vector<SDValue> &OutOps) {
  1064:   SDValue Inp = Op, Res;
  1065: 
  1066:   switch (ConstraintID) {
  1067:   default:
  1068:     return true;
  1069:   case InlineAsm::ConstraintCode::o: // Offsetable.
  1070:   case InlineAsm::ConstraintCode::v: // Not offsetable.
  1071:   case InlineAsm::ConstraintCode::m: // Memory.
  1072:     if (SelectAddrFI(Inp, Res))
  1073:       OutOps.push_back(Res);
  1074:     else
  1075:       OutOps.push_back(Inp);
  1076:     break;
  1077:   }
  1078: 
  1079:   OutOps.push_back(CurDAG->getTargetConstant(0, SDLoc(Op), MVT::i32));
  1080:   return false;
  1081: }
  1082: 
  1083: static bool isMemOPCandidate(SDNode *I, SDNode *U) {
  1084:   // I is an operand of U. Check if U is an arithmetic (binary) operation
  1085:   // usable in a memop, where the other operand is a loaded value, and the
  1086:   // result of U is stored in the same location.
  1087: 
  1088:   if (!U->hasOneUse())
  1089:     return false;
  1090:   unsigned Opc = U->getOpcode();
  1091:   switch (Opc) {
  1092:     case ISD::ADD:
  1093:     case ISD::SUB:
  1094:     case ISD::AND:
  1095:     case ISD::OR:
  1096:       break;
  1097:     default:
  1098:       return false;
  1099:   }
  1100: 
  1101:   SDValue S0 = U->getOperand(0);
  1102:   SDValue S1 = U->getOperand(1);
  1103:   SDValue SY = (S0.getNode() == I) ? S1 : S0;
  1104: 
  1105:   SDNode *UUse = *U->user_begin();
  1106:   if (UUse->getNumValues() != 1)
  1107:     return false;
  1108: 
  1109:   // Check if one of the inputs to U is a load instruction and the output
  1110:   // is used by a store instruction. If so and they also have the same
  1111:   // base pointer, then don't preoprocess this node sequence as it
  1112:   // can be matched to a memop.
  1113:   SDNode *SYNode = SY.getNode();
  1114:   if (UUse->getOpcode() == ISD::STORE && SYNode->getOpcode() == ISD::LOAD) {
  1115:     SDValue LDBasePtr = cast<MemSDNode>(SYNode)->getBasePtr();
  1116:     SDValue STBasePtr = cast<MemSDNode>(UUse)->getBasePtr();
  1117:     if (LDBasePtr == STBasePtr)
  1118:       return true;
  1119:   }
  1120:   return false;
  1121: }
  1122: 
  1123: 
  1124: // Transform: (or (select c x 0) z)  ->  (select c (or x z) z)
  1125: //            (or (select c 0 y) z)  ->  (select c z (or y z))
  1126: void HexagonDAGToDAGISel::ppSimplifyOrSelect0(std::vector<SDNode*> &&Nodes) {
  1127:   SelectionDAG &DAG = *CurDAG;
  1128: 
  1129:   for (auto *I : Nodes) {
  1130:     if (I->getOpcode() != ISD::OR)
  1131:       continue;
  1132: 
  1133:     auto IsSelect0 = [](const SDValue &Op) -> bool {
  1134:       if (Op.getOpcode() != ISD::SELECT)
  1135:         return false;
  1136:       return isNullConstant(Op.getOperand(1)) ||
  1137:              isNullConstant(Op.getOperand(2));
  1138:     };
  1139: 
  1140:     SDValue N0 = I->getOperand(0), N1 = I->getOperand(1);
  1141:     EVT VT = I->getValueType(0);
  1142:     bool SelN0 = IsSelect0(N0);
  1143:     SDValue SOp = SelN0 ? N0 : N1;
  1144:     SDValue VOp = SelN0 ? N1 : N0;
  1145: 
  1146:     if (SOp.getOpcode() == ISD::SELECT && SOp.getNode()->hasOneUse()) {
  1147:       SDValue SC = SOp.getOperand(0);
  1148:       SDValue SX = SOp.getOperand(1);
  1149:       SDValue SY = SOp.getOperand(2);
  1150:       SDLoc DLS = SOp;
  1151:       if (isNullConstant(SY)) {
  1152:         SDValue NewOr = DAG.getNode(ISD::OR, DLS, VT, SX, VOp);
  1153:         SDValue NewSel = DAG.getNode(ISD::SELECT, DLS, VT, SC, NewOr, VOp);
  1154:         DAG.ReplaceAllUsesWith(I, NewSel.getNode());
  1155:       } else if (isNullConstant(SX)) {
  1156:         SDValue NewOr = DAG.getNode(ISD::OR, DLS, VT, SY, VOp);
  1157:         SDValue NewSel = DAG.getNode(ISD::SELECT, DLS, VT, SC, VOp, NewOr);
  1158:         DAG.ReplaceAllUsesWith(I, NewSel.getNode());
  1159:       }
  1160:     }
  1161:   }
  1162: }
  1163: 
  1164: // Transform: (store ch val (add x (add (shl y c) e)))
  1165: //        to: (store ch val (add x (shl (add y d) c))),
  1166: // where e = (shl d c) for some integer d.
  1167: // The purpose of this is to enable generation of loads/stores with
  1168: // shifted addressing mode, i.e. mem(x+y<<#c). For that, the shift
  1169: // value c must be 0, 1 or 2.
  1170: void HexagonDAGToDAGISel::ppAddrReorderAddShl(std::vector<SDNode*> &&Nodes) {
  1171:   SelectionDAG &DAG = *CurDAG;
  1172: 
  1173:   for (auto *I : Nodes) {
  1174:     if (I->getOpcode() != ISD::STORE)
  1175:       continue;
  1176: 
  1177:     // I matched: (store ch val Off)
  1178:     SDValue Off = I->getOperand(2);
  1179:     // Off needs to match: (add x (add (shl y c) (shl d c))))
  1180:     if (Off.getOpcode() != ISD::ADD)
  1181:       continue;
  1182:     // Off matched: (add x T0)
  1183:     SDValue T0 = Off.getOperand(1);
  1184:     // T0 needs to match: (add T1 T2):
  1185:     if (T0.getOpcode() != ISD::ADD)
  1186:       continue;
  1187:     // T0 matched: (add T1 T2)
  1188:     SDValue T1 = T0.getOperand(0);
  1189:     SDValue T2 = T0.getOperand(1);
  1190:     // T1 needs to match: (shl y c)
  1191:     if (T1.getOpcode() != ISD::SHL)
  1192:       continue;
  1193:     SDValue C = T1.getOperand(1);
  1194:     ConstantSDNode *CN = dyn_cast<ConstantSDNode>(C.getNode());
  1195:     if (CN == nullptr)
  1196:       continue;
  1197:     unsigned CV = CN->getZExtValue();
  1198:     if (CV > 2)
  1199:       continue;
  1200:     // T2 needs to match e, where e = (shl d c) for some d.
```
- EN: It declares or implements routines such as HexagonDAGToDAGISel::SelectFDiv, FDiv, HexagonDAGToDAGISel::Select, SelectHvxExtractSubvector, ... (41 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonDAGToDAGISel, HexagonISD, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonDAGToDAGISel::SelectFDiv, FDiv, HexagonDAGToDAGISel::Select, SelectHvxExtractSubvector, ... (41 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonDAGToDAGISel, HexagonISD，说明了它与同级后端组件的连接关系。

### Lines 1201-1400 / 第 1201-1400 行

```cpp
  1201:     ConstantSDNode *EN = dyn_cast<ConstantSDNode>(T2.getNode());
  1202:     if (EN == nullptr)
  1203:       continue;
  1204:     unsigned EV = EN->getZExtValue();
  1205:     if (EV % (1 << CV) != 0)
  1206:       continue;
  1207:     unsigned DV = EV / (1 << CV);
  1208: 
  1209:     // Replace T0 with: (shl (add y d) c)
  1210:     SDLoc DL = SDLoc(I);
  1211:     EVT VT = T0.getValueType();
  1212:     SDValue D = DAG.getConstant(DV, DL, VT);
  1213:     // NewAdd = (add y d)
  1214:     SDValue NewAdd = DAG.getNode(ISD::ADD, DL, VT, T1.getOperand(0), D);
  1215:     // NewShl = (shl NewAdd c)
  1216:     SDValue NewShl = DAG.getNode(ISD::SHL, DL, VT, NewAdd, C);
  1217:     ReplaceNode(T0.getNode(), NewShl.getNode());
  1218:   }
  1219: }
  1220: 
  1221: // Transform: (load ch (add x (and (srl y c) Mask)))
  1222: //        to: (load ch (add x (shl (srl y d) d-c)))
  1223: // where
  1224: // Mask = 00..0 111..1 0.0
  1225: //          |     |     +-- d-c 0s, and d-c is 0, 1 or 2.
  1226: //          |     +-------- 1s
  1227: //          +-------------- at most c 0s
  1228: // Motivating example:
  1229: // DAG combiner optimizes (add x (shl (srl y 5) 2))
  1230: //                     to (add x (and (srl y 3) 1FFFFFFC))
  1231: // which results in a constant-extended and(##...,lsr). This transformation
  1232: // undoes this simplification for cases where the shl can be folded into
  1233: // an addressing mode.
  1234: void HexagonDAGToDAGISel::ppAddrRewriteAndSrl(std::vector<SDNode*> &&Nodes) {
  1235:   SelectionDAG &DAG = *CurDAG;
  1236: 
  1237:   for (SDNode *N : Nodes) {
  1238:     unsigned Opc = N->getOpcode();
  1239:     if (Opc != ISD::LOAD && Opc != ISD::STORE)
  1240:       continue;
  1241:     SDValue Addr = Opc == ISD::LOAD ? N->getOperand(1) : N->getOperand(2);
  1242:     // Addr must match: (add x T0)
  1243:     if (Addr.getOpcode() != ISD::ADD)
  1244:       continue;
  1245:     SDValue T0 = Addr.getOperand(1);
  1246:     // T0 must match: (and T1 Mask)
  1247:     if (T0.getOpcode() != ISD::AND)
  1248:       continue;
  1249: 
  1250:     // We have an AND.
  1251:     //
  1252:     // Check the first operand. It must be: (srl y c).
  1253:     SDValue S = T0.getOperand(0);
  1254:     if (S.getOpcode() != ISD::SRL)
  1255:       continue;
  1256:     ConstantSDNode *SN = dyn_cast<ConstantSDNode>(S.getOperand(1).getNode());
  1257:     if (SN == nullptr)
  1258:       continue;
  1259:     if (SN->getAPIntValue().getBitWidth() != 32)
  1260:       continue;
  1261:     uint32_t CV = SN->getZExtValue();
  1262: 
  1263:     // Check the second operand: the supposed mask.
  1264:     ConstantSDNode *MN = dyn_cast<ConstantSDNode>(T0.getOperand(1).getNode());
  1265:     if (MN == nullptr)
  1266:       continue;
  1267:     if (MN->getAPIntValue().getBitWidth() != 32)
  1268:       continue;
  1269:     uint32_t Mask = MN->getZExtValue();
  1270:     // Examine the mask.
  1271:     uint32_t TZ = llvm::countr_zero(Mask);
  1272:     uint32_t M1 = llvm::countr_one(Mask >> TZ);
  1273:     uint32_t LZ = llvm::countl_zero(Mask);
  1274:     // Trailing zeros + middle ones + leading zeros must equal the width.
  1275:     if (TZ + M1 + LZ != 32)
  1276:       continue;
  1277:     // The number of trailing zeros will be encoded in the addressing mode.
  1278:     if (TZ > 2)
  1279:       continue;
  1280:     // The number of leading zeros must be at most c.
  1281:     if (LZ > CV)
  1282:       continue;
  1283: 
  1284:     // All looks good.
  1285:     SDValue Y = S.getOperand(0);
  1286:     EVT VT = Addr.getValueType();
  1287:     SDLoc dl(S);
  1288:     // TZ = D-C, so D = TZ+C.
  1289:     SDValue D = DAG.getConstant(TZ+CV, dl, VT);
  1290:     SDValue DC = DAG.getConstant(TZ, dl, VT);
  1291:     SDValue NewSrl = DAG.getNode(ISD::SRL, dl, VT, Y, D);
  1292:     SDValue NewShl = DAG.getNode(ISD::SHL, dl, VT, NewSrl, DC);
  1293:     ReplaceNode(T0.getNode(), NewShl.getNode());
  1294:   }
  1295: }
  1296: 
  1297: // Transform: (op ... (zext i1 c) ...) -> (select c (op ... 0 ...)
  1298: //                                                  (op ... 1 ...))
  1299: void HexagonDAGToDAGISel::ppHoistZextI1(std::vector<SDNode*> &&Nodes) {
  1300:   SelectionDAG &DAG = *CurDAG;
  1301: 
  1302:   for (SDNode *N : Nodes) {
  1303:     unsigned Opc = N->getOpcode();
  1304:     if (Opc != ISD::ZERO_EXTEND)
  1305:       continue;
  1306:     SDValue OpI1 = N->getOperand(0);
  1307:     EVT OpVT = OpI1.getValueType();
  1308:     if (!OpVT.isSimple() || OpVT.getSimpleVT() != MVT::i1)
  1309:       continue;
  1310:     for (SDUse &Use : N->uses()) {
  1311:       SDNode *U = Use.getUser();
  1312:       if (U->getNumValues() != 1)
  1313:         continue;
  1314:       EVT UVT = U->getValueType(0);
  1315:       if (!UVT.isSimple() || !UVT.isInteger() || UVT.getSimpleVT() == MVT::i1)
  1316:         continue;
  1317:       // Do not generate select for all i1 vector type.
  1318:       if (UVT.isVector() && UVT.getVectorElementType() == MVT::i1)
  1319:         continue;
  1320:       if (isMemOPCandidate(N, U))
  1321:         continue;
  1322: 
  1323:       // Potentially simplifiable operation.
  1324:       unsigned I1N = Use.getOperandNo();
  1325:       SmallVector<SDValue,2> Ops(U->getNumOperands());
  1326:       for (unsigned i = 0, n = U->getNumOperands(); i != n; ++i)
  1327:         Ops[i] = U->getOperand(i);
  1328:       EVT BVT = Ops[I1N].getValueType();
  1329: 
  1330:       const SDLoc &dl(U);
  1331:       SDValue C0 = DAG.getConstant(0, dl, BVT);
  1332:       SDValue C1 = DAG.getConstant(1, dl, BVT);
  1333:       SDValue If0, If1;
  1334: 
  1335:       if (isa<MachineSDNode>(U)) {
  1336:         unsigned UseOpc = U->getMachineOpcode();
  1337:         Ops[I1N] = C0;
  1338:         If0 = SDValue(DAG.getMachineNode(UseOpc, dl, UVT, Ops), 0);
  1339:         Ops[I1N] = C1;
  1340:         If1 = SDValue(DAG.getMachineNode(UseOpc, dl, UVT, Ops), 0);
  1341:       } else {
  1342:         unsigned UseOpc = U->getOpcode();
  1343:         Ops[I1N] = C0;
  1344:         If0 = DAG.getNode(UseOpc, dl, UVT, Ops);
  1345:         Ops[I1N] = C1;
  1346:         If1 = DAG.getNode(UseOpc, dl, UVT, Ops);
  1347:       }
  1348:       // We're generating a SELECT way after legalization, so keep the types
  1349:       // simple.
  1350:       unsigned UW = UVT.getSizeInBits();
  1351:       EVT SVT = (UW == 32 || UW == 64) ? MVT::getIntegerVT(UW) : UVT;
  1352:       SDValue Sel = DAG.getNode(ISD::SELECT, dl, SVT, OpI1,
  1353:                                 DAG.getBitcast(SVT, If1),
  1354:                                 DAG.getBitcast(SVT, If0));
  1355:       SDValue Ret = DAG.getBitcast(UVT, Sel);
  1356:       DAG.ReplaceAllUsesWith(U, Ret.getNode());
  1357:     }
  1358:   }
  1359: }
  1360: 
  1361: void HexagonDAGToDAGISel::PreprocessISelDAG() {
  1362:   // Repack all nodes before calling each preprocessing function,
  1363:   // because each of them can modify the set of nodes.
  1364:   auto getNodes = [this]() -> std::vector<SDNode *> {
  1365:     std::vector<SDNode *> T;
  1366:     T.reserve(CurDAG->allnodes_size());
  1367:     for (SDNode &N : CurDAG->allnodes())
  1368:       T.push_back(&N);
  1369:     return T;
  1370:   };
  1371: 
  1372:   if (HST->useHVXOps())
  1373:     PreprocessHvxISelDAG();
  1374: 
  1375:   // Transform: (or (select c x 0) z)  ->  (select c (or x z) z)
  1376:   //            (or (select c 0 y) z)  ->  (select c z (or y z))
  1377:   ppSimplifyOrSelect0(getNodes());
  1378: 
  1379:   // Transform: (store ch val (add x (add (shl y c) e)))
  1380:   //        to: (store ch val (add x (shl (add y d) c))),
  1381:   // where e = (shl d c) for some integer d.
  1382:   // The purpose of this is to enable generation of loads/stores with
  1383:   // shifted addressing mode, i.e. mem(x+y<<#c). For that, the shift
  1384:   // value c must be 0, 1 or 2.
  1385:   ppAddrReorderAddShl(getNodes());
  1386: 
  1387:   // Transform: (load ch (add x (and (srl y c) Mask)))
  1388:   //        to: (load ch (add x (shl (srl y d) d-c)))
  1389:   // where
  1390:   // Mask = 00..0 111..1 0.0
  1391:   //          |     |     +-- d-c 0s, and d-c is 0, 1 or 2.
  1392:   //          |     +-------- 1s
  1393:   //          +-------------- at most c 0s
  1394:   // Motivating example:
  1395:   // DAG combiner optimizes (add x (shl (srl y 5) 2))
  1396:   //                     to (add x (and (srl y 3) 1FFFFFFC))
  1397:   // which results in a constant-extended and(##...,lsr). This transformation
  1398:   // undoes this simplification for cases where the shl can be folded into
  1399:   // an addressing mode.
  1400:   ppAddrRewriteAndSrl(getNodes());
```
- EN: It declares or implements routines such as dyn_cast<ConstantSDNode>, getZExtValue, with:, getValueType, ... (25 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonDAGToDAGISel, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 dyn_cast<ConstantSDNode>, getZExtValue, with:, getValueType, ... (25 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonDAGToDAGISel，说明了它与同级后端组件的连接关系。

### Lines 1401-1600 / 第 1401-1600 行

```cpp
  1401: 
  1402:   // Transform: (op ... (zext i1 c) ...) -> (select c (op ... 0 ...)
  1403:   //                                                  (op ... 1 ...))
  1404:   ppHoistZextI1(getNodes());
  1405: 
  1406:   DEBUG_WITH_TYPE("isel", {
  1407:     dbgs() << "Preprocessed (Hexagon) selection DAG:";
  1408:     CurDAG->dump();
  1409:   });
  1410: 
  1411:   if (EnableAddressRebalancing) {
  1412:     rebalanceAddressTrees();
  1413: 
  1414:     DEBUG_WITH_TYPE("isel", {
  1415:       dbgs() << "Address tree balanced selection DAG:";
  1416:       CurDAG->dump();
  1417:     });
  1418:   }
  1419: }
  1420: 
  1421: void HexagonDAGToDAGISel::emitFunctionEntryCode() {
  1422:   auto &HST = MF->getSubtarget<HexagonSubtarget>();
  1423:   auto &HFI = *HST.getFrameLowering();
  1424:   if (!HFI.needsAligna(*MF))
  1425:     return;
  1426: 
  1427:   MachineFrameInfo &MFI = MF->getFrameInfo();
  1428:   MachineBasicBlock *EntryBB = &MF->front();
  1429:   Align EntryMaxA = MFI.getMaxAlign();
  1430: 
  1431:   // Reserve the first non-volatile register.
  1432:   Register AP = 0;
  1433:   auto &HRI = *HST.getRegisterInfo();
  1434:   BitVector Reserved = HRI.getReservedRegs(*MF);
  1435:   for (const MCPhysReg *R = HRI.getCalleeSavedRegs(MF); *R; ++R) {
  1436:     if (Reserved[*R])
  1437:       continue;
  1438:     AP = *R;
  1439:     break;
  1440:   }
  1441:   assert(AP.isValid() && "Couldn't reserve stack align register");
  1442:   BuildMI(EntryBB, DebugLoc(), HII->get(Hexagon::PS_aligna), AP)
  1443:       .addImm(EntryMaxA.value());
  1444:   MF->getInfo<HexagonMachineFunctionInfo>()->setStackAlignBaseReg(AP);
  1445: }
  1446: 
  1447: void HexagonDAGToDAGISel::updateAligna() {
  1448:   auto &HFI = *MF->getSubtarget<HexagonSubtarget>().getFrameLowering();
  1449:   if (!HFI.needsAligna(*MF))
  1450:     return;
  1451:   auto *AlignaI = const_cast<MachineInstr*>(HFI.getAlignaInstr(*MF));
  1452:   assert(AlignaI != nullptr);
  1453:   unsigned MaxA = MF->getFrameInfo().getMaxAlign().value();
  1454:   if (AlignaI->getOperand(1).getImm() < MaxA)
  1455:     AlignaI->getOperand(1).setImm(MaxA);
  1456: }
  1457: 
  1458: // Match a frame index that can be used in an addressing mode.
  1459: bool HexagonDAGToDAGISel::SelectAddrFI(SDValue &N, SDValue &R) {
  1460:   if (N.getOpcode() != ISD::FrameIndex)
  1461:     return false;
  1462:   auto &HFI = *HST->getFrameLowering();
  1463:   MachineFrameInfo &MFI = MF->getFrameInfo();
  1464:   int FX = cast<FrameIndexSDNode>(N)->getIndex();
  1465:   if (!MFI.isFixedObjectIndex(FX) && HFI.needsAligna(*MF))
  1466:     return false;
  1467:   R = CurDAG->getTargetFrameIndex(FX, MVT::i32);
  1468:   return true;
  1469: }
  1470: 
  1471: inline bool HexagonDAGToDAGISel::SelectAddrGA(SDValue &N, SDValue &R) {
  1472:   return SelectGlobalAddress(N, R, false, Align(1));
  1473: }
  1474: 
  1475: inline bool HexagonDAGToDAGISel::SelectAddrGP(SDValue &N, SDValue &R) {
  1476:   return SelectGlobalAddress(N, R, true, Align(1));
  1477: }
  1478: 
  1479: inline bool HexagonDAGToDAGISel::SelectAnyImm(SDValue &N, SDValue &R) {
  1480:   return SelectAnyImmediate(N, R, Align(1));
  1481: }
  1482: 
  1483: inline bool HexagonDAGToDAGISel::SelectAnyImm0(SDValue &N, SDValue &R) {
  1484:   return SelectAnyImmediate(N, R, Align(1));
  1485: }
  1486: inline bool HexagonDAGToDAGISel::SelectAnyImm1(SDValue &N, SDValue &R) {
  1487:   return SelectAnyImmediate(N, R, Align(2));
  1488: }
  1489: inline bool HexagonDAGToDAGISel::SelectAnyImm2(SDValue &N, SDValue &R) {
  1490:   return SelectAnyImmediate(N, R, Align(4));
  1491: }
  1492: inline bool HexagonDAGToDAGISel::SelectAnyImm3(SDValue &N, SDValue &R) {
  1493:   return SelectAnyImmediate(N, R, Align(8));
  1494: }
  1495: 
  1496: inline bool HexagonDAGToDAGISel::SelectAnyInt(SDValue &N, SDValue &R) {
  1497:   EVT T = N.getValueType();
  1498:   if (!T.isInteger() || T.getSizeInBits() != 32 || !isa<ConstantSDNode>(N))
  1499:     return false;
  1500:   uint32_t V = cast<const ConstantSDNode>(N)->getZExtValue();
  1501:   R = CurDAG->getTargetConstant(V, SDLoc(N), N.getValueType());
  1502:   return true;
  1503: }
  1504: 
  1505: bool HexagonDAGToDAGISel::SelectAnyImmediate(SDValue &N, SDValue &R,
  1506:                                              Align Alignment) {
  1507:   switch (N.getOpcode()) {
  1508:   case ISD::Constant: {
  1509:     if (N.getValueType() != MVT::i32)
  1510:       return false;
  1511:     uint32_t V = cast<const ConstantSDNode>(N)->getZExtValue();
  1512:     if (!isAligned(Alignment, V))
  1513:       return false;
  1514:     R = CurDAG->getTargetConstant(V, SDLoc(N), N.getValueType());
  1515:     return true;
  1516:   }
  1517:   case HexagonISD::JT:
  1518:   case HexagonISD::CP:
  1519:     // These are assumed to always be aligned at least 8-byte boundary.
  1520:     if (Alignment > Align(8))
  1521:       return false;
  1522:     R = N.getOperand(0);
  1523:     return true;
  1524:   case ISD::ExternalSymbol:
  1525:     // Symbols may be aligned at any boundary.
  1526:     if (Alignment > Align(1))
  1527:       return false;
  1528:     R = N;
  1529:     return true;
  1530:   case ISD::BlockAddress:
  1531:     // Block address is always aligned at least 4-byte boundary.
  1532:     if (Alignment > Align(4) ||
  1533:         !isAligned(Alignment, cast<BlockAddressSDNode>(N)->getOffset()))
  1534:       return false;
  1535:     R = N;
  1536:     return true;
  1537:   }
  1538: 
  1539:   if (SelectGlobalAddress(N, R, false, Alignment) ||
  1540:       SelectGlobalAddress(N, R, true, Alignment))
  1541:     return true;
  1542: 
  1543:   return false;
  1544: }
  1545: 
  1546: bool HexagonDAGToDAGISel::SelectGlobalAddress(SDValue &N, SDValue &R,
  1547:                                               bool UseGP, Align Alignment) {
  1548:   switch (N.getOpcode()) {
  1549:   case ISD::ADD: {
  1550:     SDValue N0 = N.getOperand(0);
  1551:     SDValue N1 = N.getOperand(1);
  1552:     unsigned GAOpc = N0.getOpcode();
  1553:     if (UseGP && GAOpc != HexagonISD::CONST32_GP)
  1554:       return false;
  1555:     if (!UseGP && GAOpc != HexagonISD::CONST32)
  1556:       return false;
  1557:     if (ConstantSDNode *Const = dyn_cast<ConstantSDNode>(N1)) {
  1558:       if (!isAligned(Alignment, Const->getZExtValue()))
  1559:         return false;
  1560:       SDValue Addr = N0.getOperand(0);
  1561:       if (GlobalAddressSDNode *GA = dyn_cast<GlobalAddressSDNode>(Addr)) {
  1562:         if (GA->getOpcode() == ISD::TargetGlobalAddress) {
  1563:           uint64_t NewOff = GA->getOffset() + (uint64_t)Const->getSExtValue();
  1564:           R = CurDAG->getTargetGlobalAddress(GA->getGlobal(), SDLoc(Const),
  1565:                                              N.getValueType(), NewOff);
  1566:           return true;
  1567:         }
  1568:       }
  1569:     }
  1570:     break;
  1571:   }
  1572:   case HexagonISD::CP:
  1573:   case HexagonISD::JT:
  1574:   case HexagonISD::CONST32:
  1575:     // The operand(0) of CONST32 is TargetGlobalAddress, which is what we
  1576:     // want in the instruction.
  1577:     if (!UseGP)
  1578:       R = N.getOperand(0);
  1579:     return !UseGP;
  1580:   case HexagonISD::CONST32_GP:
  1581:     if (UseGP)
  1582:       R = N.getOperand(0);
  1583:     return UseGP;
  1584:   default:
  1585:     return false;
  1586:   }
  1587: 
  1588:   return false;
  1589: }
  1590: 
  1591: bool HexagonDAGToDAGISel::DetectUseSxtw(SDValue &N, SDValue &R) {
  1592:   // This (complex pattern) function is meant to detect a sign-extension
  1593:   // i32->i64 on a per-operand basis. This would allow writing single
  1594:   // patterns that would cover a number of combinations of different ways
  1595:   // a sign-extensions could be written. For example:
  1596:   //   (mul (DetectUseSxtw x) (DetectUseSxtw y)) -> (M2_dpmpyss_s0 x y)
  1597:   // could match either one of these:
  1598:   //   (mul (sext x) (sext_inreg y))
  1599:   //   (mul (sext-load *p) (sext_inreg y))
  1600:   //   (mul (sext_inreg x) (sext y))
```
- EN: It declares or implements routines such as Transform:, dump, rebalanceAddressTrees, HexagonDAGToDAGISel::emitFunctionEntryCode, ... (40 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonDAGToDAGISel, HexagonSubtarget, HexagonMachineFunctionInfo, HexagonISD, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 Transform:, dump, rebalanceAddressTrees, HexagonDAGToDAGISel::emitFunctionEntryCode, ... (40 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonDAGToDAGISel, HexagonSubtarget, HexagonMachineFunctionInfo, HexagonISD，说明了它与同级后端组件的连接关系。

### Lines 1601-1800 / 第 1601-1800 行

```cpp
  1601:   // etc.
  1602:   //
  1603:   // The returned value will have type i64 and its low word will
  1604:   // contain the value being extended. The high bits are not specified.
  1605:   // The returned type is i64 because the original type of N was i64,
  1606:   // but the users of this function should only use the low-word of the
  1607:   // result, e.g.
  1608:   //  (mul sxtw:x, sxtw:y) -> (M2_dpmpyss_s0 (LoReg sxtw:x), (LoReg sxtw:y))
  1609: 
  1610:   if (N.getValueType() != MVT::i64)
  1611:     return false;
  1612:   unsigned Opc = N.getOpcode();
  1613:   switch (Opc) {
  1614:     case ISD::SIGN_EXTEND:
  1615:     case ISD::SIGN_EXTEND_INREG: {
  1616:       // sext_inreg has the source type as a separate operand.
  1617:       EVT T = Opc == ISD::SIGN_EXTEND
  1618:                 ? N.getOperand(0).getValueType()
  1619:                 : cast<VTSDNode>(N.getOperand(1))->getVT();
  1620:       unsigned SW = T.getSizeInBits();
  1621:       if (SW == 32)
  1622:         R = N.getOperand(0);
  1623:       else if (SW < 32)
  1624:         R = N;
  1625:       else
  1626:         return false;
  1627:       break;
  1628:     }
  1629:     case ISD::LOAD: {
  1630:       LoadSDNode *L = cast<LoadSDNode>(N);
  1631:       if (L->getExtensionType() != ISD::SEXTLOAD)
  1632:         return false;
  1633:       // All extending loads extend to i32, so even if the value in
  1634:       // memory is shorter than 32 bits, it will be i32 after the load.
  1635:       if (L->getMemoryVT().getSizeInBits() > 32)
  1636:         return false;
  1637:       R = N;
  1638:       break;
  1639:     }
  1640:     case ISD::SRA: {
  1641:       auto *S = dyn_cast<ConstantSDNode>(N.getOperand(1));
  1642:       if (!S || S->getZExtValue() != 32)
  1643:         return false;
  1644:       R = N;
  1645:       break;
  1646:     }
  1647:     case ISD::AssertSext: {
  1648:       EVT T = cast<VTSDNode>(N.getOperand(1))->getVT();
  1649:       if (T.getSizeInBits() == 32)
  1650:         R = N.getOperand(0);
  1651:       else
  1652:         return false;
  1653:       break;
  1654:     }
  1655: 
  1656:     default:
  1657:       return false;
  1658:   }
  1659:   EVT RT = R.getValueType();
  1660:   if (RT == MVT::i64)
  1661:     return true;
  1662:   assert(RT == MVT::i32);
  1663:   // This is only to produce a value of type i64. Do not rely on the
  1664:   // high bits produced by this.
  1665:   const SDLoc &dl(N);
  1666:   SDValue Ops[] = {
  1667:     CurDAG->getTargetConstant(Hexagon::DoubleRegsRegClassID, dl, MVT::i32),
  1668:     R, CurDAG->getTargetConstant(Hexagon::isub_hi, dl, MVT::i32),
  1669:     R, CurDAG->getTargetConstant(Hexagon::isub_lo, dl, MVT::i32)
  1670:   };
  1671:   SDNode *T = CurDAG->getMachineNode(TargetOpcode::REG_SEQUENCE, dl,
  1672:                                      MVT::i64, Ops);
  1673:   R = SDValue(T, 0);
  1674:   return true;
  1675: }
  1676: 
  1677: bool HexagonDAGToDAGISel::keepsLowBits(const SDValue &Val, unsigned NumBits,
  1678:       SDValue &Src) {
  1679:   unsigned Opc = Val.getOpcode();
  1680:   switch (Opc) {
  1681:   case ISD::SIGN_EXTEND:
  1682:   case ISD::ZERO_EXTEND:
  1683:   case ISD::ANY_EXTEND: {
  1684:     const SDValue &Op0 = Val.getOperand(0);
  1685:     EVT T = Op0.getValueType();
  1686:     if (T.isInteger() && T.getSizeInBits() == NumBits) {
  1687:       Src = Op0;
  1688:       return true;
  1689:     }
  1690:     break;
  1691:   }
  1692:   case ISD::SIGN_EXTEND_INREG:
  1693:   case ISD::AssertSext:
  1694:   case ISD::AssertZext:
  1695:     if (Val.getOperand(0).getValueType().isInteger()) {
  1696:       VTSDNode *T = cast<VTSDNode>(Val.getOperand(1));
  1697:       if (T->getVT().getSizeInBits() == NumBits) {
  1698:         Src = Val.getOperand(0);
  1699:         return true;
  1700:       }
  1701:     }
  1702:     break;
  1703:   case ISD::AND: {
  1704:     // Check if this is an AND with NumBits of lower bits set to 1.
  1705:     uint64_t Mask = (1ULL << NumBits) - 1;
  1706:     if (ConstantSDNode *C = dyn_cast<ConstantSDNode>(Val.getOperand(0))) {
  1707:       if (C->getZExtValue() == Mask) {
  1708:         Src = Val.getOperand(1);
  1709:         return true;
  1710:       }
  1711:     }
  1712:     if (ConstantSDNode *C = dyn_cast<ConstantSDNode>(Val.getOperand(1))) {
  1713:       if (C->getZExtValue() == Mask) {
  1714:         Src = Val.getOperand(0);
  1715:         return true;
  1716:       }
  1717:     }
  1718:     break;
  1719:   }
  1720:   case ISD::OR:
  1721:   case ISD::XOR: {
  1722:     // OR/XOR with the lower NumBits bits set to 0.
  1723:     uint64_t Mask = (1ULL << NumBits) - 1;
  1724:     if (ConstantSDNode *C = dyn_cast<ConstantSDNode>(Val.getOperand(0))) {
  1725:       if ((C->getZExtValue() & Mask) == 0) {
  1726:         Src = Val.getOperand(1);
  1727:         return true;
  1728:       }
  1729:     }
  1730:     if (ConstantSDNode *C = dyn_cast<ConstantSDNode>(Val.getOperand(1))) {
  1731:       if ((C->getZExtValue() & Mask) == 0) {
  1732:         Src = Val.getOperand(0);
  1733:         return true;
  1734:       }
  1735:     }
  1736:     break;
  1737:   }
  1738:   default:
  1739:     break;
  1740:   }
  1741:   return false;
  1742: }
  1743: 
  1744: bool HexagonDAGToDAGISel::isAlignedMemNode(const MemSDNode *N) const {
  1745:   return N->getAlign().value() >= N->getMemoryVT().getStoreSize();
  1746: }
  1747: 
  1748: bool HexagonDAGToDAGISel::isSmallStackStore(const StoreSDNode *N) const {
  1749:   unsigned StackSize = MF->getFrameInfo().estimateStackSize(*MF);
  1750:   switch (N->getMemoryVT().getStoreSize()) {
  1751:     case 1:
  1752:       return StackSize <= 56;   // 1*2^6 - 8
  1753:     case 2:
  1754:       return StackSize <= 120;  // 2*2^6 - 8
  1755:     case 4:
  1756:       return StackSize <= 248;  // 4*2^6 - 8
  1757:     default:
  1758:       return false;
  1759:   }
  1760: }
  1761: 
  1762: // Return true when the given node fits in a positive half word.
  1763: bool HexagonDAGToDAGISel::isPositiveHalfWord(const SDNode *N) const {
  1764:   if (const ConstantSDNode *CN = dyn_cast<const ConstantSDNode>(N)) {
  1765:     int64_t V = CN->getSExtValue();
  1766:     return V > 0 && isInt<16>(V);
  1767:   }
  1768:   if (N->getOpcode() == ISD::SIGN_EXTEND_INREG) {
  1769:     const VTSDNode *VN = dyn_cast<const VTSDNode>(N->getOperand(1));
  1770:     return VN->getVT().getSizeInBits() <= 16;
  1771:   }
  1772:   return false;
  1773: }
  1774: 
  1775: bool HexagonDAGToDAGISel::hasOneUse(const SDNode *N) const {
  1776:   return !CheckSingleUse || N->hasOneUse();
  1777: }
  1778: 
  1779: ////////////////////////////////////////////////////////////////////////////////
  1780: // Rebalancing of address calculation trees
  1781: 
  1782: static bool isOpcodeHandled(const SDNode *N) {
  1783:   switch (N->getOpcode()) {
  1784:     case ISD::ADD:
  1785:     case ISD::MUL:
  1786:       return true;
  1787:     case ISD::SHL:
  1788:       // We only handle constant shifts because these can be easily flattened
  1789:       // into multiplications by 2^Op1.
  1790:       return N->getNumOperands() >= 2 &&
  1791:              isa<ConstantSDNode>(N->getOperand(1).getNode());
  1792:     default:
  1793:       return false;
  1794:   }
  1795: }
  1796: 
  1797: /// Return the weight of an SDNode
  1798: int HexagonDAGToDAGISel::getWeight(SDNode *N) {
  1799:   if (!isOpcodeHandled(N))
  1800:     return 1;
```
- EN: It declares or implements routines such as getOpcode, getOperand, getSizeInBits, cast<LoadSDNode>, ... (25 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonDAGToDAGISel, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getOpcode, getOperand, getSizeInBits, cast<LoadSDNode>, ... (25 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonDAGToDAGISel，说明了它与同级后端组件的连接关系。

### Lines 1801-2000 / 第 1801-2000 行

```cpp
  1801:   assert(RootWeights.count(N) && "Cannot get weight of unseen root!");
  1802:   assert(RootWeights[N] != -1 && "Cannot get weight of unvisited root!");
  1803:   assert(RootWeights[N] != -2 && "Cannot get weight of RAWU'd root!");
  1804:   return RootWeights[N];
  1805: }
  1806: 
  1807: int HexagonDAGToDAGISel::getHeight(SDNode *N) {
  1808:   if (!isOpcodeHandled(N))
  1809:     return 0;
  1810:   assert(RootWeights.count(N) && RootWeights[N] >= 0 &&
  1811:       "Cannot query height of unvisited/RAUW'd node!");
  1812:   return RootHeights[N];
  1813: }
  1814: 
  1815: namespace {
  1816: struct WeightedLeaf {
  1817:   SDValue Value;
  1818:   int Weight;
  1819:   int InsertionOrder;
  1820: 
  1821:   WeightedLeaf() = default;
  1822: 
  1823:   WeightedLeaf(SDValue Value, int Weight, int InsertionOrder) :
  1824:     Value(Value), Weight(Weight), InsertionOrder(InsertionOrder) {
  1825:     assert(Weight >= 0 && "Weight must be >= 0");
  1826:   }
  1827: 
  1828:   static bool Compare(const WeightedLeaf &A, const WeightedLeaf &B) {
  1829:     assert(A.Value.getNode() && B.Value.getNode());
  1830:     return A.Weight == B.Weight ?
  1831:             (A.InsertionOrder > B.InsertionOrder) :
  1832:             (A.Weight > B.Weight);
  1833:   }
  1834: };
  1835: 
  1836: /// A specialized priority queue for WeigthedLeaves. It automatically folds
  1837: /// constants and allows removal of non-top elements while maintaining the
  1838: /// priority order.
  1839: class LeafPrioQueue {
  1840:   SmallVector<WeightedLeaf, 8> Q;
  1841:   bool HaveConst;
  1842:   WeightedLeaf ConstElt;
  1843:   unsigned Opcode;
  1844: 
  1845: public:
  1846:   bool empty() {
  1847:     return (!HaveConst && Q.empty());
  1848:   }
  1849: 
  1850:   size_t size() {
  1851:     return Q.size() + HaveConst;
  1852:   }
  1853: 
  1854:   bool hasConst() {
  1855:     return HaveConst;
  1856:   }
  1857: 
  1858:   const WeightedLeaf &top() {
  1859:     if (HaveConst)
  1860:       return ConstElt;
  1861:     return Q.front();
  1862:   }
  1863: 
  1864:   WeightedLeaf pop() {
  1865:     if (HaveConst) {
  1866:       HaveConst = false;
  1867:       return ConstElt;
  1868:     }
  1869:     std::pop_heap(Q.begin(), Q.end(), WeightedLeaf::Compare);
  1870:     return Q.pop_back_val();
  1871:   }
  1872: 
  1873:   void push(WeightedLeaf L, bool SeparateConst=true) {
  1874:     if (!HaveConst && SeparateConst && isa<ConstantSDNode>(L.Value)) {
  1875:       if (Opcode == ISD::MUL &&
  1876:           cast<ConstantSDNode>(L.Value)->getSExtValue() == 1)
  1877:         return;
  1878:       if (Opcode == ISD::ADD &&
  1879:           cast<ConstantSDNode>(L.Value)->getSExtValue() == 0)
  1880:         return;
  1881: 
  1882:       HaveConst = true;
  1883:       ConstElt = L;
  1884:     } else {
  1885:       Q.push_back(L);
  1886:       std::push_heap(Q.begin(), Q.end(), WeightedLeaf::Compare);
  1887:     }
  1888:   }
  1889: 
  1890:   /// Push L to the bottom of the queue regardless of its weight. If L is
  1891:   /// constant, it will not be folded with other constants in the queue.
  1892:   void pushToBottom(WeightedLeaf L) {
  1893:     L.Weight = 1000;
  1894:     push(L, false);
  1895:   }
  1896: 
  1897:   /// Search for a SHL(x, [<=MaxAmount]) subtree in the queue, return the one of
  1898:   /// lowest weight and remove it from the queue.
  1899:   WeightedLeaf findSHL(uint64_t MaxAmount);
  1900: 
  1901:   WeightedLeaf findMULbyConst();
  1902: 
  1903:   LeafPrioQueue(unsigned Opcode) :
  1904:     HaveConst(false), Opcode(Opcode) { }
  1905: };
  1906: } // end anonymous namespace
  1907: 
  1908: WeightedLeaf LeafPrioQueue::findSHL(uint64_t MaxAmount) {
  1909:   int ResultPos;
  1910:   WeightedLeaf Result;
  1911: 
  1912:   for (int Pos = 0, End = Q.size(); Pos != End; ++Pos) {
  1913:     const WeightedLeaf &L = Q[Pos];
  1914:     const SDValue &Val = L.Value;
  1915:     if (Val.getOpcode() != ISD::SHL || Val.getNumOperands() < 2 ||
  1916:         !isa<ConstantSDNode>(Val.getOperand(1)) ||
  1917:         Val.getConstantOperandVal(1) > MaxAmount)
  1918:       continue;
  1919:     if (!Result.Value.getNode() || Result.Weight > L.Weight ||
  1920:         (Result.Weight == L.Weight && Result.InsertionOrder > L.InsertionOrder))
  1921:     {
  1922:       Result = L;
  1923:       ResultPos = Pos;
  1924:     }
  1925:   }
  1926: 
  1927:   if (Result.Value.getNode()) {
  1928:     Q.erase(&Q[ResultPos]);
  1929:     std::make_heap(Q.begin(), Q.end(), WeightedLeaf::Compare);
  1930:   }
  1931: 
  1932:   return Result;
  1933: }
  1934: 
  1935: WeightedLeaf LeafPrioQueue::findMULbyConst() {
  1936:   int ResultPos;
  1937:   WeightedLeaf Result;
  1938: 
  1939:   for (int Pos = 0, End = Q.size(); Pos != End; ++Pos) {
  1940:     const WeightedLeaf &L = Q[Pos];
  1941:     const SDValue &Val = L.Value;
  1942:     if (Val.getOpcode() != ISD::MUL || Val.getNumOperands() < 2 ||
  1943:         !isa<ConstantSDNode>(Val.getOperand(1)) ||
  1944:         Val.getConstantOperandVal(1) > 127)
  1945:       continue;
  1946:     if (!Result.Value.getNode() || Result.Weight > L.Weight ||
  1947:         (Result.Weight == L.Weight && Result.InsertionOrder > L.InsertionOrder))
  1948:     {
  1949:       Result = L;
  1950:       ResultPos = Pos;
  1951:     }
  1952:   }
  1953: 
  1954:   if (Result.Value.getNode()) {
  1955:     Q.erase(&Q[ResultPos]);
  1956:     std::make_heap(Q.begin(), Q.end(), WeightedLeaf::Compare);
  1957:   }
  1958: 
  1959:   return Result;
  1960: }
  1961: 
  1962: SDValue HexagonDAGToDAGISel::getMultiplierForSHL(SDNode *N) {
  1963:   if (N->getNumOperands() < 2)
  1964:     return SDValue();
  1965:   uint64_t MulFactor = 1ull << N->getConstantOperandVal(1);
  1966:   return CurDAG->getConstant(MulFactor, SDLoc(N),
  1967:                              N->getOperand(1).getValueType());
  1968: }
  1969: 
  1970: /// @returns the value x for which 2^x is a factor of Val
  1971: static unsigned getPowerOf2Factor(SDValue Val) {
  1972:   if (Val.getOpcode() == ISD::MUL) {
  1973:     if (Val.getNumOperands() < 2)
  1974:       return 0;
  1975:     unsigned MaxFactor = 0;
  1976:     for (int i = 0; i < 2; ++i) {
  1977:       ConstantSDNode *C = dyn_cast<ConstantSDNode>(Val.getOperand(i));
  1978:       if (!C)
  1979:         continue;
  1980:       const APInt &CInt = C->getAPIntValue();
  1981:       if (CInt.getBoolValue())
  1982:         MaxFactor = CInt.countr_zero();
  1983:     }
  1984:     return MaxFactor;
  1985:   }
  1986:   if (Val.getOpcode() == ISD::SHL) {
  1987:     if (Val.getNumOperands() < 2 ||
  1988:         !isa<ConstantSDNode>(Val.getOperand(1).getNode()))
  1989:       return 0;
  1990:     return (unsigned) Val.getConstantOperandVal(1);
  1991:   }
  1992: 
  1993:   return 0;
  1994: }
  1995: 
  1996: /// @returns true if V>>Amount will eliminate V's operation on its child
  1997: static bool willShiftRightEliminate(SDValue V, unsigned Amount) {
  1998:   if (V.getOpcode() == ISD::MUL) {
  1999:     if (V.getNumOperands() < 2)
  2000:       return false;
```
- EN: It opens namespaces (WeightedLeaf) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as WeightedLeaf, LeafPrioQueue, which carry the state or API of this component. It defines declarative TableGen records like LeafPrioQueue; these records are consumed by TableGen instead of executed directly. It declares or implements routines such as assert, HexagonDAGToDAGISel::getHeight, WeightedLeaf, Compare, ... (30 total), translating Hexagon-specific policy into reusable code paths.
- CN: 这里打开了命名空间（WeightedLeaf），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 WeightedLeaf, LeafPrioQueue 等类型，用来承载该组件的状态或接口。 这里定义了 LeafPrioQueue 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里声明或实现了 assert, HexagonDAGToDAGISel::getHeight, WeightedLeaf, Compare, ... (30 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

### Lines 2001-2200 / 第 2001-2200 行

```cpp
  2001:     SDValue Ops[] = { V.getOperand(0), V.getOperand(1) };
  2002:     for (int i = 0; i < 2; ++i)
  2003:       if (isa<ConstantSDNode>(Ops[i].getNode()) &&
  2004:           V.getConstantOperandVal(i) % (1ULL << Amount) == 0) {
  2005:         uint64_t NewConst = V.getConstantOperandVal(i) >> Amount;
  2006:         return (NewConst == 1);
  2007:       }
  2008:   } else if (V.getOpcode() == ISD::SHL) {
  2009:     if (V.getNumOperands() < 2 ||
  2010:         !isa<ConstantSDNode>(V.getOperand(1).getNode()))
  2011:       return false;
  2012:     return (Amount == V.getConstantOperandVal(1));
  2013:   }
  2014: 
  2015:   return false;
  2016: }
  2017: 
  2018: SDValue HexagonDAGToDAGISel::factorOutPowerOf2(SDValue V, unsigned Power) {
  2019:   // Ensure the node has at least 2 operands before accessing them.
  2020:   if (V.getNumOperands() < 2)
  2021:     return V;
  2022: 
  2023:   SDValue Ops[] = { V.getOperand(0), V.getOperand(1) };
  2024:   if (V.getOpcode() == ISD::MUL) {
  2025:     for (int i=0; i < 2; ++i) {
  2026:       if (isa<ConstantSDNode>(Ops[i].getNode()) &&
  2027:           V.getConstantOperandVal(i) % ((uint64_t)1 << Power) == 0) {
  2028:         uint64_t NewConst = V.getConstantOperandVal(i) >> Power;
  2029:         if (NewConst == 1)
  2030:           return Ops[!i];
  2031:         Ops[i] = CurDAG->getConstant(NewConst,
  2032:                                      SDLoc(V), V.getValueType());
  2033:         break;
  2034:       }
  2035:     }
  2036:   } else if (V.getOpcode() == ISD::SHL) {
  2037:     if (!isa<ConstantSDNode>(V.getOperand(1).getNode()))
  2038:       return V;
  2039:     uint64_t ShiftAmount = V.getConstantOperandVal(1);
  2040:     if (ShiftAmount == Power)
  2041:       return Ops[0];
  2042:     Ops[1] = CurDAG->getConstant(ShiftAmount - Power,
  2043:                                  SDLoc(V), V.getValueType());
  2044:   }
  2045: 
  2046:   return CurDAG->getNode(V.getOpcode(), SDLoc(V), V.getValueType(), Ops);
  2047: }
  2048: 
  2049: static bool isTargetConstant(const SDValue &V) {
  2050:   return V.getOpcode() == HexagonISD::CONST32 ||
  2051:          V.getOpcode() == HexagonISD::CONST32_GP;
  2052: }
  2053: 
  2054: unsigned HexagonDAGToDAGISel::getUsesInFunction(const Value *V) {
  2055:   auto [It, Inserted] = GAUsesInFunction.try_emplace(V);
  2056:   if (!Inserted)
  2057:     return It->second;
  2058: 
  2059:   unsigned Result = 0;
  2060:   const Function &CurF = CurDAG->getMachineFunction().getFunction();
  2061:   for (const User *U : V->users()) {
  2062:     if (isa<Instruction>(U) &&
  2063:         cast<Instruction>(U)->getParent()->getParent() == &CurF)
  2064:       ++Result;
  2065:   }
  2066: 
  2067:   It->second = Result;
  2068: 
  2069:   return Result;
  2070: }
  2071: 
  2072: /// Note - After calling this, N may be dead. It may have been replaced by a
  2073: /// new node, so always use the returned value in place of N.
  2074: ///
  2075: /// @returns The SDValue taking the place of N (which could be N if it is
  2076: /// unchanged)
  2077: SDValue HexagonDAGToDAGISel::balanceSubTree(SDNode *N, bool TopLevel) {
  2078:   assert(RootWeights.count(N) && "Cannot balance non-root node.");
  2079:   assert(RootWeights[N] != -2 && "This node was RAUW'd!");
  2080:   assert(!TopLevel || N->getOpcode() == ISD::ADD);
  2081: 
  2082:   // Return early if this node was already visited
  2083:   if (RootWeights[N] != -1)
  2084:     return SDValue(N, 0);
  2085: 
  2086:   assert(isOpcodeHandled(N));
  2087: 
  2088:   if (N->getNumOperands() < 2)
  2089:     return SDValue(N, 0);
  2090: 
  2091:   SDValue Op0 = N->getOperand(0);
  2092:   SDValue Op1 = N->getOperand(1);
  2093: 
  2094:   // Return early if the operands will remain unchanged or are all roots
  2095:   if ((!isOpcodeHandled(Op0.getNode()) || RootWeights.count(Op0.getNode())) &&
  2096:       (!isOpcodeHandled(Op1.getNode()) || RootWeights.count(Op1.getNode()))) {
  2097:     SDNode *Op0N = Op0.getNode();
  2098:     int Weight;
  2099:     if (isOpcodeHandled(Op0N) && RootWeights[Op0N] == -1) {
  2100:       Weight = getWeight(balanceSubTree(Op0N).getNode());
  2101:       // Weight = calculateWeight(Op0N);
  2102:     } else
  2103:       Weight = getWeight(Op0N);
  2104: 
  2105:     SDNode *Op1N = Op1.getNode();
  2106:     if (isOpcodeHandled(Op1N) && RootWeights[Op1N] == -1) {
  2107:       Weight += getWeight(balanceSubTree(Op1N).getNode());
  2108:       // Weight += calculateWeight(Op1N);
  2109:     } else
  2110:       Weight += getWeight(Op1N);
  2111: 
  2112:     RootWeights[N] = Weight;
  2113: 
  2114:     // After recursive calls, check if Op0/Op1 are still valid before getting
  2115:     // height
  2116:     int Height0 = 0, Height1 = 0;
  2117:     if (isOpcodeHandled(Op0N) && RootWeights.count(Op0N) &&
  2118:         RootWeights[Op0N] >= 0)
  2119:       Height0 = getHeight(Op0N);
  2120:     if (isOpcodeHandled(Op1N) && RootWeights.count(Op1N) &&
  2121:         RootWeights[Op1N] >= 0)
  2122:       Height1 = getHeight(Op1N);
  2123: 
  2124:     RootHeights[N] = std::max(Height0, Height1) + 1;
  2125: 
  2126:     LLVM_DEBUG(dbgs() << "--> No need to balance root (Weight=" << Weight
  2127:                       << " Height=" << RootHeights[N] << "): ");
  2128:     LLVM_DEBUG(N->dump(CurDAG));
  2129: 
  2130:     return SDValue(N, 0);
  2131:   }
  2132: 
  2133:   LLVM_DEBUG(dbgs() << "** Balancing root node: ");
  2134:   LLVM_DEBUG(N->dump(CurDAG));
  2135: 
  2136:   unsigned NOpcode = N->getOpcode();
  2137: 
  2138:   LeafPrioQueue Leaves(NOpcode);
  2139:   SmallVector<SDValue, 4> Worklist;
  2140:   Worklist.push_back(SDValue(N, 0));
  2141: 
  2142:   // SHL nodes will be converted to MUL nodes
  2143:   if (NOpcode == ISD::SHL)
  2144:     NOpcode = ISD::MUL;
  2145: 
  2146:   bool CanFactorize = false;
  2147:   WeightedLeaf Mul1, Mul2;
  2148:   unsigned MaxPowerOf2 = 0;
  2149:   WeightedLeaf GA;
  2150: 
  2151:   // Do not try to factor out a shift if there is already a shift at the tip of
  2152:   // the tree.
  2153:   bool HaveTopLevelShift = false;
  2154:   if (TopLevel &&
  2155:       ((isOpcodeHandled(Op0.getNode()) && Op0.getOpcode() == ISD::SHL &&
  2156:         Op0.getNumOperands() >= 2 && Op0.getConstantOperandVal(1) < 4) ||
  2157:        (isOpcodeHandled(Op1.getNode()) && Op1.getOpcode() == ISD::SHL &&
  2158:         Op1.getNumOperands() >= 2 && Op1.getConstantOperandVal(1) < 4)))
  2159:     HaveTopLevelShift = true;
  2160: 
  2161:   // Flatten the subtree into an ordered list of leaves; at the same time
  2162:   // determine whether the tree is already balanced.
  2163:   int InsertionOrder = 0;
  2164:   SmallDenseMap<SDValue, int> NodeHeights;
  2165:   bool Imbalanced = false;
  2166:   int CurrentWeight = 0;
  2167:   while (!Worklist.empty()) {
  2168:     SDValue Child = Worklist.pop_back_val();
  2169: 
  2170:     if (Child.getNode() != N && RootWeights.count(Child.getNode())) {
  2171:       // CASE 1: Child is a root note
  2172: 
  2173:       int Weight = RootWeights[Child.getNode()];
  2174:       if (Weight == -1) {
  2175:         Child = balanceSubTree(Child.getNode());
  2176:         // calculateWeight(Child.getNode());
  2177:         Weight = getWeight(Child.getNode());
  2178:       } else if (Weight == -2) {
  2179:         // Whoops, this node was RAUWd by one of the balanceSubTree calls we
  2180:         // made. Our worklist isn't up to date anymore.
  2181:         // Restart the whole process.
  2182:         LLVM_DEBUG(dbgs() << "--> Subtree was RAUWd. Restarting...\n");
  2183:         return balanceSubTree(N, TopLevel);
  2184:       }
  2185: 
  2186:       NodeHeights[Child] = 1;
  2187:       CurrentWeight += Weight;
  2188: 
  2189:       unsigned PowerOf2;
  2190:       if (TopLevel && !CanFactorize && !HaveTopLevelShift &&
  2191:           (Child.getOpcode() == ISD::MUL || Child.getOpcode() == ISD::SHL) &&
  2192:           Child.hasOneUse() && (PowerOf2 = getPowerOf2Factor(Child))) {
  2193:         // Try to identify two factorizable MUL/SHL children greedily. Leave
  2194:         // them out of the priority queue for now so we can deal with them
  2195:         // after.
  2196:         if (!Mul1.Value.getNode()) {
  2197:           Mul1 = WeightedLeaf(Child, Weight, InsertionOrder++);
  2198:           MaxPowerOf2 = PowerOf2;
  2199:         } else {
  2200:           Mul2 = WeightedLeaf(Child, Weight, InsertionOrder++);
```
- EN: It declares or implements routines such as HexagonDAGToDAGISel::factorOutPowerOf2, getConstant, getConstantOperandVal, getNode, ... (21 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonDAGToDAGISel, HexagonISD, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonDAGToDAGISel::factorOutPowerOf2, getConstant, getConstantOperandVal, getNode, ... (21 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonDAGToDAGISel, HexagonISD，说明了它与同级后端组件的连接关系。

### Lines 2201-2400 / 第 2201-2400 行

```cpp
  2201:           MaxPowerOf2 = std::min(MaxPowerOf2, PowerOf2);
  2202: 
  2203:           // Our addressing modes can only shift by a maximum of 3
  2204:           if (MaxPowerOf2 > 3)
  2205:             MaxPowerOf2 = 3;
  2206: 
  2207:           CanFactorize = true;
  2208:         }
  2209:       } else
  2210:         Leaves.push(WeightedLeaf(Child, Weight, InsertionOrder++));
  2211:     } else if (!isOpcodeHandled(Child.getNode())) {
  2212:       // CASE 2: Child is an unhandled kind of node (e.g. constant)
  2213:       int Weight = getWeight(Child.getNode());
  2214: 
  2215:       NodeHeights[Child] = getHeight(Child.getNode());
  2216:       CurrentWeight += Weight;
  2217: 
  2218:       if (isTargetConstant(Child) && !GA.Value.getNode())
  2219:         GA = WeightedLeaf(Child, Weight, InsertionOrder++);
  2220:       else
  2221:         Leaves.push(WeightedLeaf(Child, Weight, InsertionOrder++));
  2222:     } else {
  2223:       // CASE 3: Child is a subtree of same opcode
  2224:       // Visit children first, then flatten.
  2225:       unsigned ChildOpcode = Child.getOpcode();
  2226:       assert(ChildOpcode == NOpcode ||
  2227:              (NOpcode == ISD::MUL && ChildOpcode == ISD::SHL));
  2228: 
  2229:       if (Child->getNumOperands() < 2) {
  2230:         // Treat as a leaf if not enough operands
  2231:         int Weight = getWeight(Child.getNode());
  2232:         NodeHeights[Child] = getHeight(Child.getNode());
  2233:         CurrentWeight += Weight;
  2234:         Leaves.push(WeightedLeaf(Child, Weight, InsertionOrder++));
  2235:         continue;
  2236:       }
  2237: 
  2238:       // Convert SHL to MUL
  2239:       SDValue Op1;
  2240:       if (ChildOpcode == ISD::SHL) {
  2241:         Op1 = getMultiplierForSHL(Child.getNode());
  2242:         assert(Op1.getNode() && "getMultiplierForSHL returned null");
  2243:       } else
  2244:         Op1 = Child->getOperand(1);
  2245: 
  2246:       if (!NodeHeights.count(Op1) || !NodeHeights.count(Child->getOperand(0))) {
  2247:         assert(!NodeHeights.count(Child) && "Parent visited before children?");
  2248:         // Visit children first, then re-visit this node
  2249:         Worklist.push_back(Child);
  2250:         Worklist.push_back(Op1);
  2251:         Worklist.push_back(Child->getOperand(0));
  2252:       } else {
  2253:         // Back at this node after visiting the children
  2254:         if (std::abs(NodeHeights[Op1] - NodeHeights[Child->getOperand(0)]) > 1)
  2255:           Imbalanced = true;
  2256: 
  2257:         NodeHeights[Child] = std::max(NodeHeights[Op1],
  2258:                                       NodeHeights[Child->getOperand(0)]) + 1;
  2259:       }
  2260:     }
  2261:   }
  2262: 
  2263:   LLVM_DEBUG(dbgs() << "--> Current height=" << NodeHeights[SDValue(N, 0)]
  2264:                     << " weight=" << CurrentWeight
  2265:                     << " imbalanced=" << Imbalanced << "\n");
  2266: 
  2267:   // Transform MUL(x, C * 2^Y) + SHL(z, Y) -> SHL(ADD(MUL(x, C), z), Y)
  2268:   //  This factors out a shift in order to match memw(a<<Y+b).
  2269:   if (CanFactorize && (willShiftRightEliminate(Mul1.Value, MaxPowerOf2) ||
  2270:                        willShiftRightEliminate(Mul2.Value, MaxPowerOf2))) {
  2271:     LLVM_DEBUG(dbgs() << "--> Found common factor for two MUL children!\n");
  2272:     int Weight = Mul1.Weight + Mul2.Weight;
  2273:     int Height = std::max(NodeHeights[Mul1.Value], NodeHeights[Mul2.Value]) + 1;
  2274:     SDValue Mul1Factored = factorOutPowerOf2(Mul1.Value, MaxPowerOf2);
  2275:     SDValue Mul2Factored = factorOutPowerOf2(Mul2.Value, MaxPowerOf2);
  2276:     SDValue Sum = CurDAG->getNode(ISD::ADD, SDLoc(N), Mul1.Value.getValueType(),
  2277:                                   Mul1Factored, Mul2Factored);
  2278:     SDValue Const = CurDAG->getConstant(MaxPowerOf2, SDLoc(N),
  2279:                                         Mul1.Value.getValueType());
  2280:     SDValue New = CurDAG->getNode(ISD::SHL, SDLoc(N), Mul1.Value.getValueType(),
  2281:                                   Sum, Const);
  2282:     NodeHeights[New] = Height;
  2283:     Leaves.push(WeightedLeaf(New, Weight, Mul1.InsertionOrder));
  2284:   } else if (Mul1.Value.getNode()) {
  2285:     // We failed to factorize two MULs, so now the Muls are left outside the
  2286:     // queue... add them back.
  2287:     Leaves.push(Mul1);
  2288:     if (Mul2.Value.getNode())
  2289:       Leaves.push(Mul2);
  2290:     CanFactorize = false;
  2291:   }
  2292: 
  2293:   // Combine GA + Constant -> GA+Offset, but only if GA is not used elsewhere
  2294:   // and the root node itself is not used more than twice. This reduces the
  2295:   // amount of additional constant extenders introduced by this optimization.
  2296:   bool CombinedGA = false;
  2297:   if (NOpcode == ISD::ADD && GA.Value.getNode() && Leaves.hasConst() &&
  2298:       GA.Value.hasOneUse() && N->use_size() < 3 &&
  2299:       GA.Value.getNumOperands() >= 1) {
  2300:     GlobalAddressSDNode *GANode =
  2301:       cast<GlobalAddressSDNode>(GA.Value.getOperand(0));
  2302:     ConstantSDNode *Offset = cast<ConstantSDNode>(Leaves.top().Value);
  2303: 
  2304:     if (getUsesInFunction(GANode->getGlobal()) == 1 && Offset->hasOneUse() &&
  2305:         getTargetLowering()->isOffsetFoldingLegal(GANode)) {
  2306:       LLVM_DEBUG(dbgs() << "--> Combining GA and offset ("
  2307:                         << Offset->getSExtValue() << "): ");
  2308:       LLVM_DEBUG(GANode->dump(CurDAG));
  2309: 
  2310:       SDValue NewTGA =
  2311:         CurDAG->getTargetGlobalAddress(GANode->getGlobal(), SDLoc(GA.Value),
  2312:             GANode->getValueType(0),
  2313:             GANode->getOffset() + (uint64_t)Offset->getSExtValue());
  2314:       GA.Value = CurDAG->getNode(GA.Value.getOpcode(), SDLoc(GA.Value),
  2315:           GA.Value.getValueType(), NewTGA);
  2316:       GA.Weight += Leaves.top().Weight;
  2317: 
  2318:       NodeHeights[GA.Value] = getHeight(GA.Value.getNode());
  2319:       CombinedGA = true;
  2320: 
  2321:       Leaves.pop(); // Remove the offset constant from the queue
  2322:     }
  2323:   }
  2324: 
  2325:   if ((RebalanceOnlyForOptimizations && !CanFactorize && !CombinedGA) ||
  2326:       (RebalanceOnlyImbalancedTrees && !Imbalanced)) {
  2327:     RootWeights[N] = CurrentWeight;
  2328:     RootHeights[N] = NodeHeights[SDValue(N, 0)];
  2329: 
  2330:     return SDValue(N, 0);
  2331:   }
  2332: 
  2333:   // Combine GA + SHL(x, C<=31) so we will match Rx=add(#u8,asl(Rx,#U5))
  2334:   if (NOpcode == ISD::ADD && GA.Value.getNode()) {
  2335:     WeightedLeaf SHL = Leaves.findSHL(31);
  2336:     if (SHL.Value.getNode()) {
  2337:       int Height = std::max(NodeHeights[GA.Value], NodeHeights[SHL.Value]) + 1;
  2338:       GA.Value = CurDAG->getNode(ISD::ADD, SDLoc(GA.Value),
  2339:                                  GA.Value.getValueType(),
  2340:                                  GA.Value, SHL.Value);
  2341:       GA.Weight = SHL.Weight; // Specifically ignore the GA weight here
  2342:       NodeHeights[GA.Value] = Height;
  2343:     }
  2344:   }
  2345: 
  2346:   if (GA.Value.getNode())
  2347:     Leaves.push(GA);
  2348: 
  2349:   // If this is the top level and we haven't factored out a shift, we should try
  2350:   // to move a constant to the bottom to match addressing modes like memw(rX+C)
  2351:   if (TopLevel && !CanFactorize && Leaves.hasConst()) {
  2352:     LLVM_DEBUG(dbgs() << "--> Pushing constant to tip of tree.");
  2353:     Leaves.pushToBottom(Leaves.pop());
  2354:   }
  2355: 
  2356:   const DataLayout &DL = CurDAG->getDataLayout();
  2357:   const TargetLowering &TLI = *getTargetLowering();
  2358: 
  2359:   // Rebuild the tree using Huffman's algorithm
  2360:   while (Leaves.size() > 1) {
  2361:     WeightedLeaf L0 = Leaves.pop();
  2362: 
  2363:     // See whether we can grab a MUL to form an add(Rx,mpyi(Ry,#u6)),
  2364:     // otherwise just get the next leaf
  2365:     WeightedLeaf L1 = Leaves.findMULbyConst();
  2366:     if (!L1.Value.getNode())
  2367:       L1 = Leaves.pop();
  2368: 
  2369:     assert(L0.Weight <= L1.Weight && "Priority queue is broken!");
  2370: 
  2371:     SDValue V0 = L0.Value;
  2372:     int V0Weight = L0.Weight;
  2373:     SDValue V1 = L1.Value;
  2374:     int V1Weight = L1.Weight;
  2375: 
  2376:     // Make sure that none of these nodes have been RAUW'd
  2377:     if ((RootWeights.count(V0.getNode()) && RootWeights[V0.getNode()] == -2) ||
  2378:         (RootWeights.count(V1.getNode()) && RootWeights[V1.getNode()] == -2)) {
  2379:       LLVM_DEBUG(dbgs() << "--> Subtree was RAUWd. Restarting...\n");
  2380:       return balanceSubTree(N, TopLevel);
  2381:     }
  2382: 
  2383:     ConstantSDNode *V0C = dyn_cast<ConstantSDNode>(V0);
  2384:     ConstantSDNode *V1C = dyn_cast<ConstantSDNode>(V1);
  2385:     EVT VT = N->getValueType(0);
  2386:     SDValue NewNode;
  2387: 
  2388:     if (V0C && !V1C) {
  2389:       std::swap(V0, V1);
  2390:       std::swap(V0C, V1C);
  2391:     }
  2392: 
  2393:     // Calculate height of this node
  2394:     assert(NodeHeights.count(V0) && NodeHeights.count(V1) &&
  2395:            "Children must have been visited before re-combining them!");
  2396:     int Height = std::max(NodeHeights[V0], NodeHeights[V1]) + 1;
  2397: 
  2398:     // Rebuild this node (and restore SHL from MUL if needed)
  2399:     if (V1C && NOpcode == ISD::MUL && V1C->getAPIntValue().isPowerOf2())
  2400:       NewNode = CurDAG->getNode(
```
- EN: It declares or implements routines such as std::min, push, node, getHeight, ... (31 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明或实现了 std::min, push, node, getHeight, ... (31 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 2401-2525 / 第 2401-2525 行

```cpp
  2401:           ISD::SHL, SDLoc(V0), VT, V0,
  2402:           CurDAG->getConstant(
  2403:               V1C->getAPIntValue().logBase2(), SDLoc(N),
  2404:               TLI.getScalarShiftAmountTy(DL, V0.getValueType())));
  2405:     else
  2406:       NewNode = CurDAG->getNode(NOpcode, SDLoc(N), VT, V0, V1);
  2407: 
  2408:     NodeHeights[NewNode] = Height;
  2409: 
  2410:     int Weight = V0Weight + V1Weight;
  2411:     Leaves.push(WeightedLeaf(NewNode, Weight, L0.InsertionOrder));
  2412: 
  2413:     LLVM_DEBUG(dbgs() << "--> Built new node (Weight=" << Weight
  2414:                       << ",Height=" << Height << "):\n");
  2415:     LLVM_DEBUG(NewNode.dump());
  2416:   }
  2417: 
  2418:   assert(Leaves.size() == 1);
  2419:   SDValue NewRoot = Leaves.top().Value;
  2420: 
  2421:   assert(NodeHeights.count(NewRoot));
  2422:   int Height = NodeHeights[NewRoot];
  2423: 
  2424:   // Restore SHL if we earlier converted it to a MUL
  2425:   if (NewRoot.getOpcode() == ISD::MUL && NewRoot->getNumOperands() >= 2) {
  2426:     ConstantSDNode *V1C = dyn_cast<ConstantSDNode>(NewRoot.getOperand(1));
  2427:     if (V1C && V1C->getAPIntValue().isPowerOf2()) {
  2428:       EVT VT = NewRoot.getValueType();
  2429:       SDValue V0 = NewRoot.getOperand(0);
  2430:       NewRoot = CurDAG->getNode(
  2431:           ISD::SHL, SDLoc(NewRoot), VT, V0,
  2432:           CurDAG->getConstant(
  2433:               V1C->getAPIntValue().logBase2(), SDLoc(NewRoot),
  2434:               TLI.getScalarShiftAmountTy(DL, V0.getValueType())));
  2435:     }
  2436:   }
  2437: 
  2438:   if (N != NewRoot.getNode()) {
  2439:     LLVM_DEBUG(dbgs() << "--> Root is now: ");
  2440:     LLVM_DEBUG(NewRoot.dump());
  2441: 
  2442:     // Replace all uses of old root by new root
  2443:     CurDAG->ReplaceAllUsesWith(N, NewRoot.getNode());
  2444:     // Mark that we have RAUW'd N
  2445:     RootWeights[N] = -2;
  2446:   } else {
  2447:     LLVM_DEBUG(dbgs() << "--> Root unchanged.\n");
  2448:   }
  2449: 
  2450:   RootWeights[NewRoot.getNode()] = Leaves.top().Weight;
  2451:   RootHeights[NewRoot.getNode()] = Height;
  2452: 
  2453:   return NewRoot;
  2454: }
  2455: 
  2456: void HexagonDAGToDAGISel::rebalanceAddressTrees() {
  2457:   for (SDNode &Node : llvm::make_early_inc_range(CurDAG->allnodes())) {
  2458:     SDNode *N = &Node;
  2459:     if (N->getOpcode() != ISD::LOAD && N->getOpcode() != ISD::STORE)
  2460:       continue;
  2461: 
  2462:     SDValue BasePtr = cast<MemSDNode>(N)->getBasePtr();
  2463:     if (BasePtr.getOpcode() != ISD::ADD)
  2464:       continue;
  2465: 
  2466:     // We've already processed this node
  2467:     if (RootWeights.count(BasePtr.getNode()))
  2468:       continue;
  2469: 
  2470:     LLVM_DEBUG(dbgs() << "** Rebalancing address calculation in node: ");
  2471:     LLVM_DEBUG(N->dump(CurDAG));
  2472: 
  2473:     // FindRoots
  2474:     SmallVector<SDNode *, 4> Worklist;
  2475: 
  2476:     if (BasePtr->getNumOperands() < 2)
  2477:       continue;
  2478: 
  2479:     Worklist.push_back(BasePtr.getOperand(0).getNode());
  2480:     Worklist.push_back(BasePtr.getOperand(1).getNode());
  2481: 
  2482:     while (!Worklist.empty()) {
  2483:       SDNode *N = Worklist.pop_back_val();
  2484:       unsigned Opcode = N->getOpcode();
  2485: 
  2486:       if (!isOpcodeHandled(N))
  2487:         continue;
  2488: 
  2489:       if (N->getNumOperands() < 2)
  2490:         continue;
  2491: 
  2492:       Worklist.push_back(N->getOperand(0).getNode());
  2493:       Worklist.push_back(N->getOperand(1).getNode());
  2494: 
  2495:       // Not a root if it has only one use and same opcode as its parent
  2496:       if (N->hasOneUse() && Opcode == N->user_begin()->getOpcode())
  2497:         continue;
  2498: 
  2499:       // This root node has already been processed
  2500:       RootWeights.try_emplace(N, -1);
  2501:     }
  2502: 
  2503:     // Balance node itself
  2504:     RootWeights[BasePtr.getNode()] = -1;
  2505:     SDValue NewBasePtr = balanceSubTree(BasePtr.getNode(), /*TopLevel=*/ true);
  2506: 
  2507:     if (N->getOpcode() == ISD::LOAD) {
  2508:       if (N->getNumOperands() >= 3)
  2509:         N = CurDAG->UpdateNodeOperands(N, N->getOperand(0), NewBasePtr,
  2510:                                        N->getOperand(2));
  2511:     } else {
  2512:       if (N->getNumOperands() >= 4)
  2513:         N = CurDAG->UpdateNodeOperands(N, N->getOperand(0), N->getOperand(1),
  2514:                                        NewBasePtr, N->getOperand(3));
  2515:     }
  2516: 
  2517:     LLVM_DEBUG(dbgs() << "--> Final node: ");
  2518:     LLVM_DEBUG(N->dump(CurDAG));
  2519:   }
  2520: 
  2521:   CurDAG->RemoveDeadNodes();
  2522:   GAUsesInFunction.clear();
  2523:   RootHeights.clear();
  2524:   RootWeights.clear();
  2525: }
```
- EN: It declares or implements routines such as SDLoc, getNode, push, LLVM_DEBUG, ... (18 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonDAGToDAGISel, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 SDLoc, getNode, push, LLVM_DEBUG, ... (18 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonDAGToDAGISel，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- instruction semantics / 指令语义
- target pipeline configuration / 目标流水线配置
- pattern-driven instruction selection / 基于模式的指令选择
- intrinsic mapping / Intrinsic 映射

## Dependencies / 依赖关系

- Direct includes / 直接包含: `HexagonISelDAGToDAG.h, Hexagon.h, HexagonISelLowering.h, HexagonMachineFunctionInfo.h, HexagonTargetMachine.h, llvm/CodeGen/FunctionLoweringInfo.h, llvm/CodeGen/MachineInstrBuilder.h, llvm/CodeGen/SelectionDAGISel.h, llvm/IR/Intrinsics.h, llvm/IR/IntrinsicsHexagon.h, ... (13 total)`
- Hexagon symbols / Hexagon 符号: `HexagonISelDAGToDAG, HexagonISelLowering, HexagonMachineFunctionInfo, HexagonTargetMachine, HexagonDAGToDAGISel, HexagonGenDAGISel, HexagonDAGToDAGISelLegacy, HexagonII, HexagonFrameLowering, HexagonISD, ... (11 total)`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
