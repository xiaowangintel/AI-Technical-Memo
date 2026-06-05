# HexagonISelLowering.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonISelLowering.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): HexagonISelLowering.cpp - Hexagon DAG Lowering Implementation
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-200 / 第 1-200 行

```cpp
     1: //===-- HexagonISelLowering.cpp - Hexagon DAG Lowering Implementation -----===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: //
     9: // This file implements the interfaces that Hexagon uses to lower LLVM code
    10: // into a selection DAG.
    11: //
    12: //===----------------------------------------------------------------------===//
    13: 
    14: #include "HexagonISelLowering.h"
    15: #include "Hexagon.h"
    16: #include "HexagonMachineFunctionInfo.h"
    17: #include "HexagonRegisterInfo.h"
    18: #include "HexagonSubtarget.h"
    19: #include "HexagonTargetMachine.h"
    20: #include "HexagonTargetObjectFile.h"
    21: #include "llvm/ADT/APInt.h"
    22: #include "llvm/ADT/ArrayRef.h"
    23: #include "llvm/ADT/SmallVector.h"
    24: #include "llvm/ADT/StringSwitch.h"
    25: #include "llvm/CodeGen/CallingConvLower.h"
    26: #include "llvm/CodeGen/MachineFrameInfo.h"
    27: #include "llvm/CodeGen/MachineFunction.h"
    28: #include "llvm/CodeGen/MachineMemOperand.h"
    29: #include "llvm/CodeGen/MachineRegisterInfo.h"
    30: #include "llvm/CodeGen/SelectionDAG.h"
    31: #include "llvm/CodeGen/TargetCallingConv.h"
    32: #include "llvm/CodeGen/ValueTypes.h"
    33: #include "llvm/IR/BasicBlock.h"
    34: #include "llvm/IR/CallingConv.h"
    35: #include "llvm/IR/DataLayout.h"
    36: #include "llvm/IR/DerivedTypes.h"
    37: #include "llvm/IR/DiagnosticInfo.h"
    38: #include "llvm/IR/DiagnosticPrinter.h"
    39: #include "llvm/IR/Function.h"
    40: #include "llvm/IR/GlobalValue.h"
    41: #include "llvm/IR/IRBuilder.h"
    42: #include "llvm/IR/InlineAsm.h"
    43: #include "llvm/IR/Instructions.h"
    44: #include "llvm/IR/IntrinsicInst.h"
    45: #include "llvm/IR/Intrinsics.h"
    46: #include "llvm/IR/IntrinsicsHexagon.h"
    47: #include "llvm/IR/Module.h"
    48: #include "llvm/IR/Type.h"
    49: #include "llvm/IR/Value.h"
    50: #include "llvm/Support/Casting.h"
    51: #include "llvm/Support/CodeGen.h"
    52: #include "llvm/Support/CommandLine.h"
    53: #include "llvm/Support/Debug.h"
    54: #include "llvm/Support/ErrorHandling.h"
    55: #include "llvm/Support/MathExtras.h"
    56: #include "llvm/Support/raw_ostream.h"
    57: #include "llvm/Target/TargetMachine.h"
    58: #include <algorithm>
    59: #include <cassert>
    60: #include <cstdint>
    61: #include <limits>
    62: #include <utility>
    63: 
    64: using namespace llvm;
    65: 
    66: #define DEBUG_TYPE "hexagon-lowering"
    67: 
    68: static cl::opt<bool> EmitJumpTables("hexagon-emit-jump-tables",
    69:   cl::init(true), cl::Hidden,
    70:   cl::desc("Control jump table emission on Hexagon target"));
    71: 
    72: static cl::opt<bool>
    73:     EnableHexSDNodeSched("enable-hexagon-sdnode-sched", cl::Hidden,
    74:                          cl::desc("Enable Hexagon SDNode scheduling"));
    75: 
    76: static cl::opt<int> MinimumJumpTables("minimum-jump-tables", cl::Hidden,
    77:                                       cl::init(5),
    78:                                       cl::desc("Set minimum jump tables"));
    79: 
    80: static cl::opt<bool>
    81:     ConstantLoadsToImm("constant-loads-to-imm", cl::Hidden, cl::init(true),
    82:                        cl::desc("Convert constant loads to immediate values."));
    83: 
    84: static cl::opt<bool> AlignLoads("hexagon-align-loads",
    85:   cl::Hidden, cl::init(false),
    86:   cl::desc("Rewrite unaligned loads as a pair of aligned loads"));
    87: 
    88: static cl::opt<bool>
    89:     DisableArgsMinAlignment("hexagon-disable-args-min-alignment", cl::Hidden,
    90:                             cl::init(false),
    91:                             cl::desc("Disable minimum alignment of 1 for "
    92:                                      "arguments passed by value on stack"));
    93: 
    94: // Implement calling convention for Hexagon.
    95: 
    96: static bool CC_SkipOdd(unsigned &ValNo, MVT &ValVT, MVT &LocVT,
    97:                        CCValAssign::LocInfo &LocInfo,
    98:                        ISD::ArgFlagsTy &ArgFlags, CCState &State) {
    99:   static const MCPhysReg ArgRegs[] = {
   100:     Hexagon::R0, Hexagon::R1, Hexagon::R2,
   101:     Hexagon::R3, Hexagon::R4, Hexagon::R5
   102:   };
   103:   const unsigned NumArgRegs = std::size(ArgRegs);
   104:   unsigned RegNum = State.getFirstUnallocated(ArgRegs);
   105: 
   106:   // RegNum is an index into ArgRegs: skip a register if RegNum is odd.
   107:   if (RegNum != NumArgRegs && RegNum % 2 == 1)
   108:     State.AllocateReg(ArgRegs[RegNum]);
   109: 
   110:   // Always return false here, as this function only makes sure that the first
   111:   // unallocated register has an even register number and does not actually
   112:   // allocate a register for the current argument.
   113:   return false;
   114: }
   115: 
   116: #include "HexagonGenCallingConv.inc"
   117: 
   118: unsigned HexagonTargetLowering::getVectorTypeBreakdownForCallingConv(
   119:     LLVMContext &Context, CallingConv::ID CC, EVT VT, EVT &IntermediateVT,
   120:     unsigned &NumIntermediates, MVT &RegisterVT) const {
   121: 
   122:   bool isBoolVector = VT.getVectorElementType() == MVT::i1;
   123:   bool isPowerOf2 = VT.isPow2VectorType();
   124:   unsigned NumElts = VT.getVectorNumElements();
   125: 
   126:   // Split vectors of type vXi1 into (X/8) vectors of type v8i1,
   127:   // where X is divisible by 8.
   128:   if (isBoolVector && !Subtarget.useHVXOps() && isPowerOf2 && NumElts >= 8) {
   129:     RegisterVT = MVT::v8i8;
   130:     IntermediateVT = MVT::v8i1;
   131:     NumIntermediates = NumElts / 8;
   132:     return NumIntermediates;
   133:   }
   134: 
   135:   // In HVX 64-byte mode, vectors of type vXi1 are split into (X / 64) vectors
   136:   // of type v64i1, provided that X is divisible by 64.
   137:   if (isBoolVector && Subtarget.useHVX64BOps() && isPowerOf2 && NumElts >= 64) {
   138:     RegisterVT = MVT::v64i8;
   139:     IntermediateVT = MVT::v64i1;
   140:     NumIntermediates = NumElts / 64;
   141:     return NumIntermediates;
   142:   }
   143: 
   144:   // In HVX 128-byte mode, vectors of type vXi1 are split into (X / 128) vectors
   145:   // of type v128i1, provided that X is divisible by 128.
   146:   if (isBoolVector && Subtarget.useHVX128BOps() && isPowerOf2 &&
   147:       NumElts >= 128) {
   148:     RegisterVT = MVT::v128i8;
   149:     IntermediateVT = MVT::v128i1;
   150:     NumIntermediates = NumElts / 128;
   151:     return NumIntermediates;
   152:   }
   153: 
   154:   return TargetLowering::getVectorTypeBreakdownForCallingConv(
   155:       Context, CC, VT, IntermediateVT, NumIntermediates, RegisterVT);
   156: }
   157: 
   158: std::pair<MVT, unsigned>
   159: HexagonTargetLowering::handleMaskRegisterForCallingConv(
   160:     const HexagonSubtarget &Subtarget, EVT VT) const {
   161:   assert(VT.getVectorElementType() == MVT::i1);
   162: 
   163:   const unsigned NumElems = VT.getVectorNumElements();
   164: 
   165:   if (!VT.isPow2VectorType())
   166:     return {MVT::INVALID_SIMPLE_VALUE_TYPE, 0};
   167: 
   168:   if (!Subtarget.useHVXOps() && NumElems >= 8)
   169:     return {MVT::v8i8, NumElems / 8};
   170: 
   171:   if (Subtarget.useHVX64BOps() && NumElems >= 64)
   172:     return {MVT::v64i8, NumElems / 64};
   173: 
   174:   if (Subtarget.useHVX128BOps() && NumElems >= 128)
   175:     return {MVT::v128i8, NumElems / 128};
   176: 
   177:   return {MVT::INVALID_SIMPLE_VALUE_TYPE, 0};
   178: }
   179: 
   180: MVT HexagonTargetLowering::getRegisterTypeForCallingConv(LLVMContext &Context,
   181:                                                          CallingConv::ID CC,
   182:                                                          EVT VT) const {
   183: 
   184:   if (VT.isVector() && VT.getVectorElementType() == MVT::i1) {
   185:     auto [RegisterVT, NumRegisters] =
   186:         handleMaskRegisterForCallingConv(Subtarget, VT);
   187:     if (RegisterVT != MVT::INVALID_SIMPLE_VALUE_TYPE)
   188:       return RegisterVT;
   189:   }
   190: 
   191:   return TargetLowering::getRegisterTypeForCallingConv(Context, CC, VT);
   192: }
   193: 
   194: SDValue
   195: HexagonTargetLowering::LowerINTRINSIC_WO_CHAIN(SDValue Op, SelectionDAG &DAG)
   196:       const {
   197:   unsigned IntNo = Op.getConstantOperandVal(0);
   198:   SDLoc dl(Op);
   199:   switch (IntNo) {
   200:   default:
```
- EN: It imports headers such as HexagonISelLowering.h, Hexagon.h, HexagonMachineFunctionInfo.h, HexagonRegisterInfo.h, ... (50 total), establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as EmitJumpTables, EnableHexSDNodeSched, MinimumJumpTables, ConstantLoadsToImm, ... (22 total), translating Hexagon-specific policy into reusable code paths.
- CN: 这里引入了 HexagonISelLowering.h, Hexagon.h, HexagonMachineFunctionInfo.h, HexagonRegisterInfo.h, ... (50 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 EmitJumpTables, EnableHexSDNodeSched, MinimumJumpTables, ConstantLoadsToImm, ... (22 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

### Lines 201-400 / 第 201-400 行

```cpp
   201:     return SDValue(); // Don't custom lower most intrinsics.
   202:   case Intrinsic::thread_pointer: {
   203:     EVT PtrVT = getPointerTy(DAG.getDataLayout());
   204:     return DAG.getNode(HexagonISD::THREAD_POINTER, dl, PtrVT);
   205:   }
   206:   }
   207: }
   208: 
   209: /// CreateCopyOfByValArgument - Make a copy of an aggregate at address specified
   210: /// by "Src" to address "Dst" of size "Size".  Alignment information is
   211: /// specified by the specific parameter attribute. The copy will be passed as
   212: /// a byval function parameter.  Sometimes what we are copying is the end of a
   213: /// larger object, the part that does not fit in registers.
   214: static SDValue CreateCopyOfByValArgument(SDValue Src, SDValue Dst,
   215:                                          SDValue Chain, ISD::ArgFlagsTy Flags,
   216:                                          SelectionDAG &DAG, const SDLoc &dl) {
   217:   SDValue SizeNode = DAG.getConstant(Flags.getByValSize(), dl, MVT::i32);
   218:   return DAG.getMemcpy(
   219:       Chain, dl, Dst, Src, SizeNode, Flags.getNonZeroByValAlign(),
   220:       /*isVolatile=*/false, /*AlwaysInline=*/false,
   221:       /*CI=*/nullptr, std::nullopt, MachinePointerInfo(), MachinePointerInfo());
   222: }
   223: 
   224: bool
   225: HexagonTargetLowering::CanLowerReturn(
   226:     CallingConv::ID CallConv, MachineFunction &MF, bool IsVarArg,
   227:     const SmallVectorImpl<ISD::OutputArg> &Outs,
   228:     LLVMContext &Context, const Type *RetTy) const {
   229:   SmallVector<CCValAssign, 16> RVLocs;
   230:   CCState CCInfo(CallConv, IsVarArg, MF, RVLocs, Context);
   231: 
   232:   if (MF.getSubtarget<HexagonSubtarget>().useHVXOps())
   233:     return CCInfo.CheckReturn(Outs, RetCC_Hexagon_HVX);
   234:   return CCInfo.CheckReturn(Outs, RetCC_Hexagon);
   235: }
   236: 
   237: // LowerReturn - Lower ISD::RET. If a struct is larger than 8 bytes and is
   238: // passed by value, the function prototype is modified to return void and
   239: // the value is stored in memory pointed by a pointer passed by caller.
   240: SDValue
   241: HexagonTargetLowering::LowerReturn(SDValue Chain, CallingConv::ID CallConv,
   242:                                    bool IsVarArg,
   243:                                    const SmallVectorImpl<ISD::OutputArg> &Outs,
   244:                                    const SmallVectorImpl<SDValue> &OutVals,
   245:                                    const SDLoc &dl, SelectionDAG &DAG) const {
   246:   // CCValAssign - represent the assignment of the return value to locations.
   247:   SmallVector<CCValAssign, 16> RVLocs;
   248: 
   249:   // CCState - Info about the registers and stack slot.
   250:   CCState CCInfo(CallConv, IsVarArg, DAG.getMachineFunction(), RVLocs,
   251:                  *DAG.getContext());
   252: 
   253:   // Analyze return values of ISD::RET
   254:   if (Subtarget.useHVXOps())
   255:     CCInfo.AnalyzeReturn(Outs, RetCC_Hexagon_HVX);
   256:   else
   257:     CCInfo.AnalyzeReturn(Outs, RetCC_Hexagon);
   258: 
   259:   SDValue Glue;
   260:   SmallVector<SDValue, 4> RetOps(1, Chain);
   261: 
   262:   // Copy the result values into the output registers.
   263:   for (unsigned i = 0; i != RVLocs.size(); ++i) {
   264:     CCValAssign &VA = RVLocs[i];
   265:     SDValue Val = OutVals[i];
   266: 
   267:     switch (VA.getLocInfo()) {
   268:       default:
   269:         // Loc info must be one of Full, BCvt, SExt, ZExt, or AExt.
   270:         llvm_unreachable("Unknown loc info!");
   271:       case CCValAssign::Full:
   272:         break;
   273:       case CCValAssign::BCvt:
   274:         Val = DAG.getBitcast(VA.getLocVT(), Val);
   275:         break;
   276:       case CCValAssign::SExt:
   277:         Val = DAG.getNode(ISD::SIGN_EXTEND, dl, VA.getLocVT(), Val);
   278:         break;
   279:       case CCValAssign::ZExt:
   280:         Val = DAG.getNode(ISD::ZERO_EXTEND, dl, VA.getLocVT(), Val);
   281:         break;
   282:       case CCValAssign::AExt:
   283:         Val = DAG.getNode(ISD::ANY_EXTEND, dl, VA.getLocVT(), Val);
   284:         break;
   285:     }
   286: 
   287:     Chain = DAG.getCopyToReg(Chain, dl, VA.getLocReg(), Val, Glue);
   288: 
   289:     // Guarantee that all emitted copies are stuck together with flags.
   290:     Glue = Chain.getValue(1);
   291:     RetOps.push_back(DAG.getRegister(VA.getLocReg(), VA.getLocVT()));
   292:   }
   293: 
   294:   RetOps[0] = Chain;  // Update chain.
   295: 
   296:   // Add the glue if we have it.
   297:   if (Glue.getNode())
   298:     RetOps.push_back(Glue);
   299: 
   300:   return DAG.getNode(HexagonISD::RET_GLUE, dl, MVT::Other, RetOps);
   301: }
   302: 
   303: bool HexagonTargetLowering::mayBeEmittedAsTailCall(const CallInst *CI) const {
   304:   // If either no tail call or told not to tail call at all, don't.
   305:   return CI->isTailCall();
   306: }
   307: 
   308: Register HexagonTargetLowering::getRegisterByName(
   309:       const char* RegName, LLT VT, const MachineFunction &) const {
   310:   // Just support r19, the linux kernel uses it.
   311:   Register Reg = StringSwitch<Register>(RegName)
   312:                      .Case("r0", Hexagon::R0)
   313:                      .Case("r1", Hexagon::R1)
   314:                      .Case("r2", Hexagon::R2)
   315:                      .Case("r3", Hexagon::R3)
   316:                      .Case("r4", Hexagon::R4)
   317:                      .Case("r5", Hexagon::R5)
   318:                      .Case("r6", Hexagon::R6)
   319:                      .Case("r7", Hexagon::R7)
   320:                      .Case("r8", Hexagon::R8)
   321:                      .Case("r9", Hexagon::R9)
   322:                      .Case("r10", Hexagon::R10)
   323:                      .Case("r11", Hexagon::R11)
   324:                      .Case("r12", Hexagon::R12)
   325:                      .Case("r13", Hexagon::R13)
   326:                      .Case("r14", Hexagon::R14)
   327:                      .Case("r15", Hexagon::R15)
   328:                      .Case("r16", Hexagon::R16)
   329:                      .Case("r17", Hexagon::R17)
   330:                      .Case("r18", Hexagon::R18)
   331:                      .Case("r19", Hexagon::R19)
   332:                      .Case("r20", Hexagon::R20)
   333:                      .Case("r21", Hexagon::R21)
   334:                      .Case("r22", Hexagon::R22)
   335:                      .Case("r23", Hexagon::R23)
   336:                      .Case("r24", Hexagon::R24)
   337:                      .Case("r25", Hexagon::R25)
   338:                      .Case("r26", Hexagon::R26)
   339:                      .Case("r27", Hexagon::R27)
   340:                      .Case("r28", Hexagon::R28)
   341:                      .Case("r29", Hexagon::R29)
   342:                      .Case("r30", Hexagon::R30)
   343:                      .Case("r31", Hexagon::R31)
   344:                      .Case("r1:0", Hexagon::D0)
   345:                      .Case("r3:2", Hexagon::D1)
   346:                      .Case("r5:4", Hexagon::D2)
   347:                      .Case("r7:6", Hexagon::D3)
   348:                      .Case("r9:8", Hexagon::D4)
   349:                      .Case("r11:10", Hexagon::D5)
   350:                      .Case("r13:12", Hexagon::D6)
   351:                      .Case("r15:14", Hexagon::D7)
   352:                      .Case("r17:16", Hexagon::D8)
   353:                      .Case("r19:18", Hexagon::D9)
   354:                      .Case("r21:20", Hexagon::D10)
   355:                      .Case("r23:22", Hexagon::D11)
   356:                      .Case("r25:24", Hexagon::D12)
   357:                      .Case("r27:26", Hexagon::D13)
   358:                      .Case("r29:28", Hexagon::D14)
   359:                      .Case("r31:30", Hexagon::D15)
   360:                      .Case("sp", Hexagon::R29)
   361:                      .Case("fp", Hexagon::R30)
   362:                      .Case("lr", Hexagon::R31)
   363:                      .Case("p0", Hexagon::P0)
   364:                      .Case("p1", Hexagon::P1)
   365:                      .Case("p2", Hexagon::P2)
   366:                      .Case("p3", Hexagon::P3)
   367:                      .Case("sa0", Hexagon::SA0)
   368:                      .Case("lc0", Hexagon::LC0)
   369:                      .Case("sa1", Hexagon::SA1)
   370:                      .Case("lc1", Hexagon::LC1)
   371:                      .Case("m0", Hexagon::M0)
   372:                      .Case("m1", Hexagon::M1)
   373:                      .Case("usr", Hexagon::USR)
   374:                      .Case("ugp", Hexagon::UGP)
   375:                      .Case("cs0", Hexagon::CS0)
   376:                      .Case("cs1", Hexagon::CS1)
   377:                      .Default(Register());
   378:   return Reg;
   379: }
   380: 
   381: /// LowerCallResult - Lower the result values of an ISD::CALL into the
   382: /// appropriate copies out of appropriate physical registers.  This assumes that
   383: /// Chain/Glue are the input chain/glue to use, and that TheCall is the call
   384: /// being lowered. Returns a SDNode with the same number of values as the
   385: /// ISD::CALL.
   386: SDValue HexagonTargetLowering::LowerCallResult(
   387:     SDValue Chain, SDValue Glue, CallingConv::ID CallConv, bool IsVarArg,
   388:     const SmallVectorImpl<ISD::InputArg> &Ins, const SDLoc &dl,
   389:     SelectionDAG &DAG, SmallVectorImpl<SDValue> &InVals,
   390:     const SmallVectorImpl<SDValue> &OutVals, SDValue Callee) const {
   391:   // Assign locations to each value returned by this call.
   392:   SmallVector<CCValAssign, 16> RVLocs;
   393: 
   394:   CCState CCInfo(CallConv, IsVarArg, DAG.getMachineFunction(), RVLocs,
   395:                  *DAG.getContext());
   396: 
   397:   if (Subtarget.useHVXOps())
   398:     CCInfo.AnalyzeCallResult(Ins, RetCC_Hexagon_HVX);
   399:   else
   400:     CCInfo.AnalyzeCallResult(Ins, RetCC_Hexagon);
```
- EN: It declares or implements routines such as SDValue, getPointerTy, getNode, CreateCopyOfByValArgument, ... (24 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonISD, HexagonTargetLowering, HexagonSubtarget, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 SDValue, getPointerTy, getNode, CreateCopyOfByValArgument, ... (24 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonISD, HexagonTargetLowering, HexagonSubtarget，说明了它与同级后端组件的连接关系。

### Lines 401-600 / 第 401-600 行

```cpp
   401: 
   402:   // Copy all of the result registers out of their specified physreg.
   403:   for (unsigned i = 0; i != RVLocs.size(); ++i) {
   404:     SDValue RetVal;
   405:     if (RVLocs[i].getValVT() == MVT::i1) {
   406:       // Return values of type MVT::i1 require special handling. The reason
   407:       // is that MVT::i1 is associated with the PredRegs register class, but
   408:       // values of that type are still returned in R0. Generate an explicit
   409:       // copy into a predicate register from R0, and treat the value of the
   410:       // predicate register as the call result.
   411:       auto &MRI = DAG.getMachineFunction().getRegInfo();
   412:       SDValue FR0 = DAG.getCopyFromReg(Chain, dl, RVLocs[i].getLocReg(),
   413:                                        MVT::i32, Glue);
   414:       // FR0 = (Value, Chain, Glue)
   415:       Register PredR = MRI.createVirtualRegister(&Hexagon::PredRegsRegClass);
   416:       SDValue TPR = DAG.getCopyToReg(FR0.getValue(1), dl, PredR,
   417:                                      FR0.getValue(0), FR0.getValue(2));
   418:       // TPR = (Chain, Glue)
   419:       // Don't glue this CopyFromReg, because it copies from a virtual
   420:       // register. If it is glued to the call, InstrEmitter will add it
   421:       // as an implicit def to the call (EmitMachineNode).
   422:       RetVal = DAG.getCopyFromReg(TPR.getValue(0), dl, PredR, MVT::i1);
   423:       Glue = TPR.getValue(1);
   424:       Chain = TPR.getValue(0);
   425:     } else {
   426:       RetVal = DAG.getCopyFromReg(Chain, dl, RVLocs[i].getLocReg(),
   427:                                   RVLocs[i].getValVT(), Glue);
   428:       Glue = RetVal.getValue(2);
   429:       Chain = RetVal.getValue(1);
   430:     }
   431:     InVals.push_back(RetVal.getValue(0));
   432:   }
   433: 
   434:   return Chain;
   435: }
   436: 
   437: /// LowerCall - Functions arguments are copied from virtual regs to
   438: /// (physical regs)/(stack frame), CALLSEQ_START and CALLSEQ_END are emitted.
   439: SDValue
   440: HexagonTargetLowering::LowerCall(TargetLowering::CallLoweringInfo &CLI,
   441:                                  SmallVectorImpl<SDValue> &InVals) const {
   442:   SelectionDAG &DAG                     = CLI.DAG;
   443:   SDLoc &dl                             = CLI.DL;
   444:   SmallVectorImpl<ISD::OutputArg> &Outs = CLI.Outs;
   445:   SmallVectorImpl<SDValue> &OutVals     = CLI.OutVals;
   446:   SmallVectorImpl<ISD::InputArg> &Ins   = CLI.Ins;
   447:   SDValue Chain                         = CLI.Chain;
   448:   SDValue Callee                        = CLI.Callee;
   449:   CallingConv::ID CallConv              = CLI.CallConv;
   450:   bool IsVarArg                         = CLI.IsVarArg;
   451:   bool DoesNotReturn                    = CLI.DoesNotReturn;
   452: 
   453:   bool IsStructRet    = Outs.empty() ? false : Outs[0].Flags.isSRet();
   454:   MachineFunction &MF = DAG.getMachineFunction();
   455:   MachineFrameInfo &MFI = MF.getFrameInfo();
   456:   auto PtrVT = getPointerTy(MF.getDataLayout());
   457: 
   458:   if (GlobalAddressSDNode *GAN = dyn_cast<GlobalAddressSDNode>(Callee))
   459:     Callee = DAG.getTargetGlobalAddress(GAN->getGlobal(), dl, MVT::i32);
   460: 
   461:   // Linux ABI treats var-arg calls the same way as regular ones.
   462:   bool TreatAsVarArg = !Subtarget.isEnvironmentMusl() && IsVarArg;
   463: 
   464:   // Analyze operands of the call, assigning locations to each operand.
   465:   SmallVector<CCValAssign, 16> ArgLocs;
   466:   CCState CCInfo(CallConv, TreatAsVarArg, MF, ArgLocs, *DAG.getContext());
   467: 
   468:   if (Subtarget.useHVXOps())
   469:     CCInfo.AnalyzeCallOperands(Outs, CC_Hexagon_HVX);
   470:   else if (DisableArgsMinAlignment)
   471:     CCInfo.AnalyzeCallOperands(Outs, CC_Hexagon_Legacy);
   472:   else
   473:     CCInfo.AnalyzeCallOperands(Outs, CC_Hexagon);
   474: 
   475:   if (CLI.IsTailCall) {
   476:     bool StructAttrFlag = MF.getFunction().hasStructRetAttr();
   477:     CLI.IsTailCall = IsEligibleForTailCallOptimization(Callee, CallConv,
   478:                         IsVarArg, IsStructRet, StructAttrFlag, Outs,
   479:                         OutVals, Ins, DAG);
   480:     for (const CCValAssign &VA : ArgLocs) {
   481:       if (VA.isMemLoc()) {
   482:         CLI.IsTailCall = false;
   483:         break;
   484:       }
   485:     }
   486:     LLVM_DEBUG(dbgs() << (CLI.IsTailCall ? "Eligible for Tail Call\n"
   487:                                          : "Argument must be passed on stack. "
   488:                                            "Not eligible for Tail Call\n"));
   489:   }
   490:   // Get a count of how many bytes are to be pushed on the stack.
   491:   unsigned NumBytes = CCInfo.getStackSize();
   492:   SmallVector<std::pair<unsigned, SDValue>, 16> RegsToPass;
   493:   SmallVector<SDValue, 8> MemOpChains;
   494: 
   495:   const HexagonRegisterInfo &HRI = *Subtarget.getRegisterInfo();
   496:   SDValue StackPtr =
   497:       DAG.getCopyFromReg(Chain, dl, HRI.getStackRegister(), PtrVT);
   498: 
   499:   bool NeedsArgAlign = false;
   500:   Align LargestAlignSeen;
   501:   // Walk the register/memloc assignments, inserting copies/loads.
   502:   for (unsigned i = 0, e = ArgLocs.size(); i != e; ++i) {
   503:     CCValAssign &VA = ArgLocs[i];
   504:     SDValue Arg = OutVals[i];
   505:     ISD::ArgFlagsTy Flags = Outs[i].Flags;
   506:     // Record if we need > 8 byte alignment on an argument.
   507:     bool ArgAlign = Subtarget.isHVXVectorType(VA.getValVT());
   508:     NeedsArgAlign |= ArgAlign;
   509: 
   510:     // Promote the value if needed.
   511:     switch (VA.getLocInfo()) {
   512:       default:
   513:         // Loc info must be one of Full, BCvt, SExt, ZExt, or AExt.
   514:         llvm_unreachable("Unknown loc info!");
   515:       case CCValAssign::Full:
   516:         break;
   517:       case CCValAssign::BCvt:
   518:         Arg = DAG.getBitcast(VA.getLocVT(), Arg);
   519:         break;
   520:       case CCValAssign::SExt:
   521:         Arg = DAG.getNode(ISD::SIGN_EXTEND, dl, VA.getLocVT(), Arg);
   522:         break;
   523:       case CCValAssign::ZExt:
   524:         Arg = DAG.getNode(ISD::ZERO_EXTEND, dl, VA.getLocVT(), Arg);
   525:         break;
   526:       case CCValAssign::AExt:
   527:         Arg = DAG.getNode(ISD::ANY_EXTEND, dl, VA.getLocVT(), Arg);
   528:         break;
   529:     }
   530: 
   531:     if (VA.isMemLoc()) {
   532:       unsigned LocMemOffset = VA.getLocMemOffset();
   533:       SDValue MemAddr = DAG.getConstant(LocMemOffset, dl,
   534:                                         StackPtr.getValueType());
   535:       MemAddr = DAG.getNode(ISD::ADD, dl, MVT::i32, StackPtr, MemAddr);
   536:       if (ArgAlign)
   537:         LargestAlignSeen = std::max(
   538:             LargestAlignSeen, Align(VA.getLocVT().getStoreSizeInBits() / 8));
   539:       if (Flags.isByVal()) {
   540:         // The argument is a struct passed by value. According to LLVM, "Arg"
   541:         // is a pointer.
   542:         MemOpChains.push_back(CreateCopyOfByValArgument(Arg, MemAddr, Chain,
   543:                                                         Flags, DAG, dl));
   544:       } else {
   545:         MachinePointerInfo LocPI = MachinePointerInfo::getStack(
   546:             DAG.getMachineFunction(), LocMemOffset);
   547:         SDValue S = DAG.getStore(Chain, dl, Arg, MemAddr, LocPI);
   548:         MemOpChains.push_back(S);
   549:       }
   550:       continue;
   551:     }
   552: 
   553:     // Arguments that can be passed on register must be kept at RegsToPass
   554:     // vector.
   555:     if (VA.isRegLoc())
   556:       RegsToPass.push_back(std::make_pair(VA.getLocReg(), Arg));
   557:   }
   558: 
   559:   if (NeedsArgAlign && Subtarget.hasV60Ops()) {
   560:     LLVM_DEBUG(dbgs() << "Function needs byte stack align due to call args\n");
   561:     Align VecAlign = HRI.getSpillAlign(Hexagon::HvxVRRegClass);
   562:     LargestAlignSeen = std::max(LargestAlignSeen, VecAlign);
   563:     MFI.ensureMaxAlignment(LargestAlignSeen);
   564:   }
   565:   // Transform all store nodes into one single node because all store
   566:   // nodes are independent of each other.
   567:   if (!MemOpChains.empty())
   568:     Chain = DAG.getNode(ISD::TokenFactor, dl, MVT::Other, MemOpChains);
   569: 
   570:   SDValue Glue;
   571:   if (!CLI.IsTailCall) {
   572:     Chain = DAG.getCALLSEQ_START(Chain, NumBytes, 0, dl);
   573:     Glue = Chain.getValue(1);
   574:   }
   575: 
   576:   // Build a sequence of copy-to-reg nodes chained together with token
   577:   // chain and flag operands which copy the outgoing args into registers.
   578:   // The Glue is necessary since all emitted instructions must be
   579:   // stuck together.
   580:   if (!CLI.IsTailCall) {
   581:     for (const auto &R : RegsToPass) {
   582:       Chain = DAG.getCopyToReg(Chain, dl, R.first, R.second, Glue);
   583:       Glue = Chain.getValue(1);
   584:     }
   585:   } else {
   586:     // For tail calls lower the arguments to the 'real' stack slot.
   587:     //
   588:     // Force all the incoming stack arguments to be loaded from the stack
   589:     // before any new outgoing arguments are stored to the stack, because the
   590:     // outgoing stack slots may alias the incoming argument stack slots, and
   591:     // the alias isn't otherwise explicit. This is slightly more conservative
   592:     // than necessary, because it means that each store effectively depends
   593:     // on every argument instead of just those arguments it would clobber.
   594:     //
   595:     // Do not flag preceding copytoreg stuff together with the following stuff.
   596:     Glue = SDValue();
   597:     for (const auto &R : RegsToPass) {
   598:       Chain = DAG.getCopyToReg(Chain, dl, R.first, R.second, Glue);
   599:       Glue = Chain.getValue(1);
   600:     }
```
- EN: It declares or implements routines such as size, getMachineFunction, getCopyFromReg, createVirtualRegister, ... (32 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonTargetLowering, HexagonRegisterInfo, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 size, getMachineFunction, getCopyFromReg, createVirtualRegister, ... (32 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonTargetLowering, HexagonRegisterInfo，说明了它与同级后端组件的连接关系。

### Lines 601-800 / 第 601-800 行

```cpp
   601:     Glue = SDValue();
   602:   }
   603: 
   604:   bool LongCalls = MF.getSubtarget<HexagonSubtarget>().useLongCalls();
   605:   unsigned Flags = LongCalls ? HexagonII::HMOTF_ConstExtended : 0;
   606: 
   607:   // If the callee is a GlobalAddress/ExternalSymbol node (quite common, every
   608:   // direct call is) turn it into a TargetGlobalAddress/TargetExternalSymbol
   609:   // node so that legalize doesn't hack it.
   610:   if (GlobalAddressSDNode *G = dyn_cast<GlobalAddressSDNode>(Callee)) {
   611:     Callee = DAG.getTargetGlobalAddress(G->getGlobal(), dl, PtrVT, 0, Flags);
   612:   } else if (ExternalSymbolSDNode *S =
   613:              dyn_cast<ExternalSymbolSDNode>(Callee)) {
   614:     Callee = DAG.getTargetExternalSymbol(S->getSymbol(), PtrVT, Flags);
   615:   }
   616: 
   617:   // Returns a chain & a flag for retval copy to use.
   618:   SmallVector<SDValue, 8> Ops;
   619:   Ops.push_back(Chain);
   620:   Ops.push_back(Callee);
   621: 
   622:   // Add argument registers to the end of the list so that they are
   623:   // known live into the call.
   624:   for (const auto &R : RegsToPass)
   625:     Ops.push_back(DAG.getRegister(R.first, R.second.getValueType()));
   626: 
   627:   const uint32_t *Mask = HRI.getCallPreservedMask(MF, CallConv);
   628:   assert(Mask && "Missing call preserved mask for calling convention");
   629:   Ops.push_back(DAG.getRegisterMask(Mask));
   630: 
   631:   if (Glue.getNode())
   632:     Ops.push_back(Glue);
   633: 
   634:   if (CLI.IsTailCall) {
   635:     MFI.setHasTailCall();
   636:     return DAG.getNode(HexagonISD::TC_RETURN, dl, MVT::Other, Ops);
   637:   }
   638: 
   639:   // Set this here because we need to know this for "hasFP" in frame lowering.
   640:   // The target-independent code calls getFrameRegister before setting it, and
   641:   // getFrameRegister uses hasFP to determine whether the function has FP.
   642:   MFI.setHasCalls(true);
   643: 
   644:   unsigned OpCode = DoesNotReturn ? HexagonISD::CALLnr : HexagonISD::CALL;
   645:   Chain = DAG.getNode(OpCode, dl, {MVT::Other, MVT::Glue}, Ops);
   646:   Glue = Chain.getValue(1);
   647: 
   648:   // Create the CALLSEQ_END node.
   649:   Chain = DAG.getCALLSEQ_END(Chain, NumBytes, 0, Glue, dl);
   650:   Glue = Chain.getValue(1);
   651: 
   652:   // Handle result values, copying them out of physregs into vregs that we
   653:   // return.
   654:   return LowerCallResult(Chain, Glue, CallConv, IsVarArg, Ins, dl, DAG,
   655:                          InVals, OutVals, Callee);
   656: }
   657: 
   658: /// Returns true by value, base pointer and offset pointer and addressing
   659: /// mode by reference if this node can be combined with a load / store to
   660: /// form a post-indexed load / store.
   661: bool HexagonTargetLowering::getPostIndexedAddressParts(SDNode *N, SDNode *Op,
   662:       SDValue &Base, SDValue &Offset, ISD::MemIndexedMode &AM,
   663:       SelectionDAG &DAG) const {
   664:   LSBaseSDNode *LSN = dyn_cast<LSBaseSDNode>(N);
   665:   if (!LSN)
   666:     return false;
   667:   EVT VT = LSN->getMemoryVT();
   668:   if (!VT.isSimple())
   669:     return false;
   670:   bool IsLegalType = VT == MVT::i8 || VT == MVT::i16 || VT == MVT::i32 ||
   671:                      VT == MVT::i64 || VT == MVT::f32 || VT == MVT::f64 ||
   672:                      VT == MVT::v2i16 || VT == MVT::v2i32 || VT == MVT::v4i8 ||
   673:                      VT == MVT::v4i16 || VT == MVT::v8i8 ||
   674:                      Subtarget.isHVXVectorType(VT.getSimpleVT());
   675:   if (!IsLegalType)
   676:     return false;
   677: 
   678:   if (Op->getOpcode() != ISD::ADD)
   679:     return false;
   680:   Base = Op->getOperand(0);
   681:   Offset = Op->getOperand(1);
   682:   if (!isa<ConstantSDNode>(Offset.getNode()))
   683:     return false;
   684:   AM = ISD::POST_INC;
   685: 
   686:   int32_t V = cast<ConstantSDNode>(Offset.getNode())->getSExtValue();
   687:   return Subtarget.getInstrInfo()->isValidAutoIncImm(VT, V);
   688: }
   689: 
   690: SDValue HexagonTargetLowering::LowerFDIV(SDValue Op, SelectionDAG &DAG) const {
   691:   if (DAG.getMachineFunction().getFunction().hasOptSize())
   692:     return SDValue();
   693:   else
   694:     return Op;
   695: }
   696: 
   697: SDValue
   698: HexagonTargetLowering::LowerINLINEASM(SDValue Op, SelectionDAG &DAG) const {
   699:   MachineFunction &MF = DAG.getMachineFunction();
   700:   auto &HMFI = *MF.getInfo<HexagonMachineFunctionInfo>();
   701:   const HexagonRegisterInfo &HRI = *Subtarget.getRegisterInfo();
   702:   unsigned LR = HRI.getRARegister();
   703: 
   704:   if ((Op.getOpcode() != ISD::INLINEASM &&
   705:        Op.getOpcode() != ISD::INLINEASM_BR) || HMFI.hasClobberLR())
   706:     return Op;
   707: 
   708:   unsigned NumOps = Op.getNumOperands();
   709:   if (Op.getOperand(NumOps-1).getValueType() == MVT::Glue)
   710:     --NumOps;  // Ignore the flag operand.
   711: 
   712:   for (unsigned i = InlineAsm::Op_FirstOperand; i != NumOps;) {
   713:     const InlineAsm::Flag Flags(Op.getConstantOperandVal(i));
   714:     unsigned NumVals = Flags.getNumOperandRegisters();
   715:     ++i;  // Skip the ID value.
   716: 
   717:     switch (Flags.getKind()) {
   718:     default:
   719:       llvm_unreachable("Bad flags!");
   720:     case InlineAsm::Kind::RegUse:
   721:     case InlineAsm::Kind::Imm:
   722:     case InlineAsm::Kind::Mem:
   723:       i += NumVals;
   724:       break;
   725:     case InlineAsm::Kind::Clobber:
   726:     case InlineAsm::Kind::RegDef:
   727:     case InlineAsm::Kind::RegDefEarlyClobber: {
   728:       for (; NumVals; --NumVals, ++i) {
   729:         Register Reg = cast<RegisterSDNode>(Op.getOperand(i))->getReg();
   730:         if (Reg != LR)
   731:           continue;
   732:         HMFI.setHasClobberLR(true);
   733:         return Op;
   734:       }
   735:       break;
   736:       }
   737:       }
   738:   }
   739: 
   740:   return Op;
   741: }
   742: 
   743: // Need to transform ISD::PREFETCH into something that doesn't inherit
   744: // all of the properties of ISD::PREFETCH, specifically SDNPMayLoad and
   745: // SDNPMayStore.
   746: SDValue HexagonTargetLowering::LowerPREFETCH(SDValue Op,
   747:                                              SelectionDAG &DAG) const {
   748:   SDValue Chain = Op.getOperand(0);
   749:   SDValue Addr = Op.getOperand(1);
   750:   // Lower it to DCFETCH($reg, #0).  A "pat" will try to merge the offset in,
   751:   // if the "reg" is fed by an "add".
   752:   SDLoc DL(Op);
   753:   SDValue Zero = DAG.getConstant(0, DL, MVT::i32);
   754:   return DAG.getNode(HexagonISD::DCFETCH, DL, MVT::Other, Chain, Addr, Zero);
   755: }
   756: 
   757: SDValue HexagonTargetLowering::LowerINTRINSIC_VOID(SDValue Op,
   758:                                                    SelectionDAG &DAG) const {
   759:   SDValue Chain = Op.getOperand(0);
   760:   unsigned IntNo = Op.getConstantOperandVal(1);
   761:   // Lower the hexagon_prefetch builtin to DCFETCH, as above.
   762:   if (IntNo == Intrinsic::hexagon_prefetch) {
   763:     SDValue Addr = Op.getOperand(2);
   764:     SDLoc DL(Op);
   765:     SDValue Zero = DAG.getConstant(0, DL, MVT::i32);
   766:     return DAG.getNode(HexagonISD::DCFETCH, DL, MVT::Other, Chain, Addr, Zero);
   767:   }
   768:   return SDValue();
   769: }
   770: 
   771: SDValue
   772: HexagonTargetLowering::LowerDYNAMIC_STACKALLOC(SDValue Op,
   773:                                                SelectionDAG &DAG) const {
   774:   SDValue Chain = Op.getOperand(0);
   775:   SDValue Size = Op.getOperand(1);
   776:   SDValue Align = Op.getOperand(2);
   777:   SDLoc dl(Op);
   778: 
   779:   ConstantSDNode *AlignConst = dyn_cast<ConstantSDNode>(Align);
   780:   assert(AlignConst && "Non-constant Align in LowerDYNAMIC_STACKALLOC");
   781: 
   782:   unsigned A = AlignConst->getSExtValue();
   783:   auto &HFI = *Subtarget.getFrameLowering();
   784:   // "Zero" means natural stack alignment.
   785:   if (A == 0)
   786:     A = HFI.getStackAlign().value();
   787: 
   788:   LLVM_DEBUG({
   789:     dbgs () << __func__ << " Align: " << A << " Size: ";
   790:     Size.getNode()->dump(&DAG);
   791:     dbgs() << "\n";
   792:   });
   793: 
   794:   SDValue AC = DAG.getConstant(A, dl, MVT::i32);
   795:   SDVTList VTs = DAG.getVTList(MVT::i32, MVT::Other);
   796:   SDValue AA = DAG.getNode(HexagonISD::ALLOCA, dl, VTs, Chain, Size, AC);
   797: 
   798:   DAG.ReplaceAllUsesOfValueWith(Op, AA);
   799:   return AA;
   800: }
```
- EN: It declares or implements routines such as SDValue, getSubtarget<HexagonSubtarget>, node, getTargetGlobalAddress, ... (46 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonSubtarget, HexagonII, HexagonISD, HexagonTargetLowering, ... (6 total), showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 SDValue, getSubtarget<HexagonSubtarget>, node, getTargetGlobalAddress, ... (46 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonSubtarget, HexagonII, HexagonISD, HexagonTargetLowering, ... (6 total)，说明了它与同级后端组件的连接关系。

### Lines 801-1000 / 第 801-1000 行

```cpp
   801: 
   802: SDValue HexagonTargetLowering::LowerFormalArguments(
   803:     SDValue Chain, CallingConv::ID CallConv, bool IsVarArg,
   804:     const SmallVectorImpl<ISD::InputArg> &Ins, const SDLoc &dl,
   805:     SelectionDAG &DAG, SmallVectorImpl<SDValue> &InVals) const {
   806:   MachineFunction &MF = DAG.getMachineFunction();
   807:   MachineFrameInfo &MFI = MF.getFrameInfo();
   808:   MachineRegisterInfo &MRI = MF.getRegInfo();
   809: 
   810:   // Linux ABI treats var-arg calls the same way as regular ones.
   811:   bool TreatAsVarArg = !Subtarget.isEnvironmentMusl() && IsVarArg;
   812: 
   813:   // Assign locations to all of the incoming arguments.
   814:   SmallVector<CCValAssign, 16> ArgLocs;
   815:   CCState CCInfo(CallConv, TreatAsVarArg, MF, ArgLocs, *DAG.getContext());
   816: 
   817:   if (Subtarget.useHVXOps())
   818:     CCInfo.AnalyzeFormalArguments(Ins, CC_Hexagon_HVX);
   819:   else if (DisableArgsMinAlignment)
   820:     CCInfo.AnalyzeFormalArguments(Ins, CC_Hexagon_Legacy);
   821:   else
   822:     CCInfo.AnalyzeFormalArguments(Ins, CC_Hexagon);
   823: 
   824:   // For LLVM, in the case when returning a struct by value (>8byte),
   825:   // the first argument is a pointer that points to the location on caller's
   826:   // stack where the return value will be stored. For Hexagon, the location on
   827:   // caller's stack is passed only when the struct size is smaller than (and
   828:   // equal to) 8 bytes. If not, no address will be passed into callee and
   829:   // callee return the result directly through R0/R1.
   830:   auto NextSingleReg = [] (const TargetRegisterClass &RC, unsigned Reg) {
   831:     switch (RC.getID()) {
   832:     case Hexagon::IntRegsRegClassID:
   833:       return Reg - Hexagon::R0 + 1;
   834:     case Hexagon::DoubleRegsRegClassID:
   835:       return (Reg - Hexagon::D0 + 1) * 2;
   836:     case Hexagon::HvxVRRegClassID:
   837:       return Reg - Hexagon::V0 + 1;
   838:     case Hexagon::HvxWRRegClassID:
   839:       return (Reg - Hexagon::W0 + 1) * 2;
   840:     }
   841:     llvm_unreachable("Unexpected register class");
   842:   };
   843: 
   844:   auto &HFL = const_cast<HexagonFrameLowering&>(*Subtarget.getFrameLowering());
   845:   auto &HMFI = *MF.getInfo<HexagonMachineFunctionInfo>();
   846:   HFL.FirstVarArgSavedReg = 0;
   847:   HMFI.setFirstNamedArgFrameIndex(-int(MFI.getNumFixedObjects()));
   848: 
   849:   for (unsigned i = 0, e = ArgLocs.size(); i != e; ++i) {
   850:     CCValAssign &VA = ArgLocs[i];
   851:     ISD::ArgFlagsTy Flags = Ins[i].Flags;
   852:     bool ByVal = Flags.isByVal();
   853: 
   854:     // Arguments passed in registers:
   855:     // 1. 32- and 64-bit values and HVX vectors are passed directly,
   856:     // 2. Large structs are passed via an address, and the address is
   857:     //    passed in a register.
   858:     if (VA.isRegLoc() && ByVal && Flags.getByValSize() <= 8)
   859:       llvm_unreachable("ByValSize must be bigger than 8 bytes");
   860: 
   861:     bool InReg = VA.isRegLoc() &&
   862:                  (!ByVal || (ByVal && Flags.getByValSize() > 8));
   863: 
   864:     if (InReg) {
   865:       MVT RegVT = VA.getLocVT();
   866:       if (VA.getLocInfo() == CCValAssign::BCvt)
   867:         RegVT = VA.getValVT();
   868: 
   869:       const TargetRegisterClass *RC = getRegClassFor(RegVT);
   870:       Register VReg = MRI.createVirtualRegister(RC);
   871:       SDValue Copy = DAG.getCopyFromReg(Chain, dl, VReg, RegVT);
   872: 
   873:       // Treat values of type MVT::i1 specially: they are passed in
   874:       // registers of type i32, but they need to remain as values of
   875:       // type i1 for consistency of the argument lowering.
   876:       if (VA.getValVT() == MVT::i1) {
   877:         assert(RegVT.getSizeInBits() <= 32);
   878:         SDValue T = DAG.getNode(ISD::AND, dl, RegVT,
   879:                                 Copy, DAG.getConstant(1, dl, RegVT));
   880:         Copy = DAG.getSetCC(dl, MVT::i1, T, DAG.getConstant(0, dl, RegVT),
   881:                             ISD::SETNE);
   882:       } else {
   883: #ifndef NDEBUG
   884:         unsigned RegSize = RegVT.getSizeInBits();
   885:         assert(RegSize == 32 || RegSize == 64 ||
   886:                Subtarget.isHVXVectorType(RegVT));
   887: #endif
   888:       }
   889:       InVals.push_back(Copy);
   890:       MRI.addLiveIn(VA.getLocReg(), VReg);
   891:       HFL.FirstVarArgSavedReg = NextSingleReg(*RC, VA.getLocReg());
   892:     } else {
   893:       assert(VA.isMemLoc() && "Argument should be passed in memory");
   894: 
   895:       // If it's a byval parameter, then we need to compute the
   896:       // "real" size, not the size of the pointer.
   897:       unsigned ObjSize = Flags.isByVal()
   898:                             ? Flags.getByValSize()
   899:                             : VA.getLocVT().getStoreSizeInBits() / 8;
   900: 
   901:       // Create the frame index object for this incoming parameter.
   902:       int Offset = HEXAGON_LRFP_SIZE + VA.getLocMemOffset();
   903:       int FI = MFI.CreateFixedObject(ObjSize, Offset, true);
   904:       SDValue FIN = DAG.getFrameIndex(FI, MVT::i32);
   905: 
   906:       if (Flags.isByVal()) {
   907:         // If it's a pass-by-value aggregate, then do not dereference the stack
   908:         // location. Instead, we should generate a reference to the stack
   909:         // location.
   910:         InVals.push_back(FIN);
   911:       } else {
   912:         SDValue L = DAG.getLoad(VA.getValVT(), dl, Chain, FIN,
   913:                                 MachinePointerInfo::getFixedStack(MF, FI, 0));
   914:         InVals.push_back(L);
   915:       }
   916:     }
   917:   }
   918: 
   919:   if (IsVarArg && Subtarget.isEnvironmentMusl()) {
   920:     for (int i = HFL.FirstVarArgSavedReg; i < 6; i++)
   921:       MRI.addLiveIn(Hexagon::R0+i);
   922:   }
   923: 
   924:   if (IsVarArg && Subtarget.isEnvironmentMusl()) {
   925:     HMFI.setFirstNamedArgFrameIndex(HMFI.getFirstNamedArgFrameIndex() - 1);
   926:     HMFI.setLastNamedArgFrameIndex(-int(MFI.getNumFixedObjects()));
   927: 
   928:     // Create Frame index for the start of register saved area.
   929:     int NumVarArgRegs = 6 - HFL.FirstVarArgSavedReg;
   930:     bool RequiresPadding = (NumVarArgRegs & 1);
   931:     int RegSaveAreaSizePlusPadding = RequiresPadding
   932:                                         ? (NumVarArgRegs + 1) * 4
   933:                                         : NumVarArgRegs * 4;
   934: 
   935:     if (RegSaveAreaSizePlusPadding > 0) {
   936:       // The offset to saved register area should be 8 byte aligned.
   937:       int RegAreaStart = HEXAGON_LRFP_SIZE + CCInfo.getStackSize();
   938:       if (!(RegAreaStart % 8))
   939:         RegAreaStart = (RegAreaStart + 7) & -8;
   940: 
   941:       int RegSaveAreaFrameIndex =
   942:         MFI.CreateFixedObject(RegSaveAreaSizePlusPadding, RegAreaStart, true);
   943:       HMFI.setRegSavedAreaStartFrameIndex(RegSaveAreaFrameIndex);
   944: 
   945:       // This will point to the next argument passed via stack.
   946:       int Offset = RegAreaStart + RegSaveAreaSizePlusPadding;
   947:       int FI = MFI.CreateFixedObject(Hexagon_PointerSize, Offset, true);
   948:       HMFI.setVarArgsFrameIndex(FI);
   949:     } else {
   950:       // This will point to the next argument passed via stack, when
   951:       // there is no saved register area.
   952:       int Offset = HEXAGON_LRFP_SIZE + CCInfo.getStackSize();
   953:       int FI = MFI.CreateFixedObject(Hexagon_PointerSize, Offset, true);
   954:       HMFI.setRegSavedAreaStartFrameIndex(FI);
   955:       HMFI.setVarArgsFrameIndex(FI);
   956:     }
   957:   }
   958: 
   959: 
   960:   if (IsVarArg && !Subtarget.isEnvironmentMusl()) {
   961:     // This will point to the next argument passed via stack.
   962:     int Offset = HEXAGON_LRFP_SIZE + CCInfo.getStackSize();
   963:     int FI = MFI.CreateFixedObject(Hexagon_PointerSize, Offset, true);
   964:     HMFI.setVarArgsFrameIndex(FI);
   965:   }
   966: 
   967:   return Chain;
   968: }
   969: 
   970: SDValue
   971: HexagonTargetLowering::LowerVASTART(SDValue Op, SelectionDAG &DAG) const {
   972:   // VASTART stores the address of the VarArgsFrameIndex slot into the
   973:   // memory location argument.
   974:   MachineFunction &MF = DAG.getMachineFunction();
   975:   HexagonMachineFunctionInfo *QFI = MF.getInfo<HexagonMachineFunctionInfo>();
   976:   SDValue Addr = DAG.getFrameIndex(QFI->getVarArgsFrameIndex(), MVT::i32);
   977:   const Value *SV = cast<SrcValueSDNode>(Op.getOperand(2))->getValue();
   978: 
   979:   if (!Subtarget.isEnvironmentMusl()) {
   980:     return DAG.getStore(Op.getOperand(0), SDLoc(Op), Addr, Op.getOperand(1),
   981:                         MachinePointerInfo(SV));
   982:   }
   983:   auto &FuncInfo = *MF.getInfo<HexagonMachineFunctionInfo>();
   984:   auto &HFL = *Subtarget.getFrameLowering();
   985:   SDLoc DL(Op);
   986:   SmallVector<SDValue, 8> MemOps;
   987: 
   988:   // Get frame index of va_list.
   989:   SDValue FIN = Op.getOperand(1);
   990: 
   991:   // If first Vararg register is odd, add 4 bytes to start of
   992:   // saved register area to point to the first register location.
   993:   // This is because the saved register area has to be 8 byte aligned.
   994:   // In case of an odd start register, there will be 4 bytes of padding in
   995:   // the beginning of saved register area. If all registers area used up,
   996:   // the following condition will handle it correctly.
   997:   SDValue SavedRegAreaStartFrameIndex =
   998:     DAG.getFrameIndex(FuncInfo.getRegSavedAreaStartFrameIndex(), MVT::i32);
   999: 
  1000:   auto PtrVT = getPointerTy(DAG.getDataLayout());
```
- EN: Header guards in this range prevent duplicate inclusion and define the interface boundary. It declares or implements routines such as HexagonTargetLowering::LowerFormalArguments, getMachineFunction, getFrameInfo, getRegInfo, ... (38 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里声明或实现了 HexagonTargetLowering::LowerFormalArguments, getMachineFunction, getFrameInfo, getRegInfo, ... (38 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 1001-1200 / 第 1001-1200 行

```cpp
  1001: 
  1002:   if (HFL.FirstVarArgSavedReg & 1)
  1003:     SavedRegAreaStartFrameIndex =
  1004:       DAG.getNode(ISD::ADD, DL, PtrVT,
  1005:                   DAG.getFrameIndex(FuncInfo.getRegSavedAreaStartFrameIndex(),
  1006:                                     MVT::i32),
  1007:                   DAG.getIntPtrConstant(4, DL));
  1008: 
  1009:   // Store the saved register area start pointer.
  1010:   SDValue Store =
  1011:     DAG.getStore(Op.getOperand(0), DL,
  1012:                  SavedRegAreaStartFrameIndex,
  1013:                  FIN, MachinePointerInfo(SV));
  1014:   MemOps.push_back(Store);
  1015: 
  1016:   // Store saved register area end pointer.
  1017:   FIN = DAG.getNode(ISD::ADD, DL, PtrVT,
  1018:                     FIN, DAG.getIntPtrConstant(4, DL));
  1019:   Store = DAG.getStore(Op.getOperand(0), DL,
  1020:                        DAG.getFrameIndex(FuncInfo.getVarArgsFrameIndex(),
  1021:                                          PtrVT),
  1022:                        FIN, MachinePointerInfo(SV, 4));
  1023:   MemOps.push_back(Store);
  1024: 
  1025:   // Store overflow area pointer.
  1026:   FIN = DAG.getNode(ISD::ADD, DL, PtrVT,
  1027:                     FIN, DAG.getIntPtrConstant(4, DL));
  1028:   Store = DAG.getStore(Op.getOperand(0), DL,
  1029:                        DAG.getFrameIndex(FuncInfo.getVarArgsFrameIndex(),
  1030:                                          PtrVT),
  1031:                        FIN, MachinePointerInfo(SV, 8));
  1032:   MemOps.push_back(Store);
  1033: 
  1034:   return DAG.getNode(ISD::TokenFactor, DL, MVT::Other, MemOps);
  1035: }
  1036: 
  1037: SDValue
  1038: HexagonTargetLowering::LowerVACOPY(SDValue Op, SelectionDAG &DAG) const {
  1039:   // Assert that the linux ABI is enabled for the current compilation.
  1040:   assert(Subtarget.isEnvironmentMusl() && "Linux ABI should be enabled");
  1041:   SDValue Chain = Op.getOperand(0);
  1042:   SDValue DestPtr = Op.getOperand(1);
  1043:   SDValue SrcPtr = Op.getOperand(2);
  1044:   const Value *DestSV = cast<SrcValueSDNode>(Op.getOperand(3))->getValue();
  1045:   const Value *SrcSV = cast<SrcValueSDNode>(Op.getOperand(4))->getValue();
  1046:   SDLoc DL(Op);
  1047:   // Size of the va_list is 12 bytes as it has 3 pointers. Therefore,
  1048:   // we need to memcopy 12 bytes from va_list to another similar list.
  1049:   return DAG.getMemcpy(
  1050:       Chain, DL, DestPtr, SrcPtr, DAG.getIntPtrConstant(12, DL), Align(4),
  1051:       /*isVolatile*/ false, false, /*CI=*/nullptr, std::nullopt,
  1052:       MachinePointerInfo(DestSV), MachinePointerInfo(SrcSV));
  1053: }
  1054: 
  1055: SDValue HexagonTargetLowering::LowerSETCC(SDValue Op, SelectionDAG &DAG) const {
  1056:   const SDLoc &dl(Op);
  1057:   SDValue LHS = Op.getOperand(0);
  1058:   SDValue RHS = Op.getOperand(1);
  1059:   ISD::CondCode CC = cast<CondCodeSDNode>(Op.getOperand(2))->get();
  1060:   MVT ResTy = ty(Op);
  1061:   MVT OpTy = ty(LHS);
  1062: 
  1063:   if (OpTy == MVT::v2i16 || OpTy == MVT::v4i8) {
  1064:     MVT ElemTy = OpTy.getVectorElementType();
  1065:     assert(ElemTy.isScalarInteger());
  1066:     MVT WideTy = MVT::getVectorVT(MVT::getIntegerVT(2*ElemTy.getSizeInBits()),
  1067:                                   OpTy.getVectorNumElements());
  1068:     return DAG.getSetCC(dl, ResTy,
  1069:                         DAG.getSExtOrTrunc(LHS, SDLoc(LHS), WideTy),
  1070:                         DAG.getSExtOrTrunc(RHS, SDLoc(RHS), WideTy), CC);
  1071:   }
  1072: 
  1073:   // Treat all other vector types as legal.
  1074:   if (ResTy.isVector())
  1075:     return Op;
  1076: 
  1077:   // Comparisons of short integers should use sign-extend, not zero-extend,
  1078:   // since we can represent small negative values in the compare instructions.
  1079:   // The LLVM default is to use zero-extend arbitrarily in these cases.
  1080:   auto isSExtFree = [this](SDValue N) {
  1081:     switch (N.getOpcode()) {
  1082:       case ISD::TRUNCATE: {
  1083:         // A sign-extend of a truncate of a sign-extend is free.
  1084:         SDValue Op = N.getOperand(0);
  1085:         if (Op.getOpcode() != ISD::AssertSext)
  1086:           return false;
  1087:         EVT OrigTy = cast<VTSDNode>(Op.getOperand(1))->getVT();
  1088:         unsigned ThisBW = ty(N).getSizeInBits();
  1089:         unsigned OrigBW = OrigTy.getSizeInBits();
  1090:         // The type that was sign-extended to get the AssertSext must be
  1091:         // narrower than the type of N (so that N has still the same value
  1092:         // as the original).
  1093:         return ThisBW >= OrigBW;
  1094:       }
  1095:       case ISD::LOAD:
  1096:         // We have sign-extended loads.
  1097:         return true;
  1098:     }
  1099:     return false;
  1100:   };
  1101: 
  1102:   if (OpTy == MVT::i8 || OpTy == MVT::i16) {
  1103:     ConstantSDNode *C = dyn_cast<ConstantSDNode>(RHS);
  1104:     bool IsNegative = C && C->getAPIntValue().isNegative();
  1105:     if (IsNegative || isSExtFree(LHS) || isSExtFree(RHS))
  1106:       return DAG.getSetCC(dl, ResTy,
  1107:                           DAG.getSExtOrTrunc(LHS, SDLoc(LHS), MVT::i32),
  1108:                           DAG.getSExtOrTrunc(RHS, SDLoc(RHS), MVT::i32), CC);
  1109:   }
  1110: 
  1111:   return SDValue();
  1112: }
  1113: 
  1114: SDValue
  1115: HexagonTargetLowering::LowerVSELECT(SDValue Op, SelectionDAG &DAG) const {
  1116:   SDValue PredOp = Op.getOperand(0);
  1117:   SDValue Op1 = Op.getOperand(1), Op2 = Op.getOperand(2);
  1118:   MVT OpTy = ty(Op1);
  1119:   const SDLoc &dl(Op);
  1120: 
  1121:   if (OpTy == MVT::v2i16 || OpTy == MVT::v4i8) {
  1122:     MVT ElemTy = OpTy.getVectorElementType();
  1123:     assert(ElemTy.isScalarInteger());
  1124:     MVT WideTy = MVT::getVectorVT(MVT::getIntegerVT(2*ElemTy.getSizeInBits()),
  1125:                                   OpTy.getVectorNumElements());
  1126:     // Generate (trunc (select (_, sext, sext))).
  1127:     return DAG.getSExtOrTrunc(
  1128:               DAG.getSelect(dl, WideTy, PredOp,
  1129:                             DAG.getSExtOrTrunc(Op1, dl, WideTy),
  1130:                             DAG.getSExtOrTrunc(Op2, dl, WideTy)),
  1131:               dl, OpTy);
  1132:   }
  1133: 
  1134:   return SDValue();
  1135: }
  1136: 
  1137: SDValue
  1138: HexagonTargetLowering::LowerConstantPool(SDValue Op, SelectionDAG &DAG) const {
  1139:   EVT ValTy = Op.getValueType();
  1140:   ConstantPoolSDNode *CPN = cast<ConstantPoolSDNode>(Op);
  1141:   Constant *CVal = nullptr;
  1142:   bool isVTi1Type = false;
  1143:   if (auto *CV = dyn_cast<ConstantVector>(CPN->getConstVal())) {
  1144:     if (cast<VectorType>(CV->getType())->getElementType()->isIntegerTy(1)) {
  1145:       IRBuilder<> IRB(CV->getContext());
  1146:       SmallVector<Constant*, 128> NewConst;
  1147:       unsigned VecLen = CV->getNumOperands();
  1148:       assert(isPowerOf2_32(VecLen) &&
  1149:              "conversion only supported for pow2 VectorSize");
  1150:       for (unsigned i = 0; i < VecLen; ++i)
  1151:         NewConst.push_back(IRB.getInt8(CV->getOperand(i)->isNullValue()));
  1152: 
  1153:       CVal = ConstantVector::get(NewConst);
  1154:       isVTi1Type = true;
  1155:     }
  1156:   }
  1157:   Align Alignment = CPN->getAlign();
  1158:   bool IsPositionIndependent = isPositionIndependent();
  1159:   unsigned char TF = IsPositionIndependent ? HexagonII::MO_PCREL : 0;
  1160: 
  1161:   unsigned Offset = 0;
  1162:   SDValue T;
  1163:   if (CPN->isMachineConstantPoolEntry())
  1164:     T = DAG.getTargetConstantPool(CPN->getMachineCPVal(), ValTy, Alignment,
  1165:                                   Offset, TF);
  1166:   else if (isVTi1Type)
  1167:     T = DAG.getTargetConstantPool(CVal, ValTy, Alignment, Offset, TF);
  1168:   else
  1169:     T = DAG.getTargetConstantPool(CPN->getConstVal(), ValTy, Alignment, Offset,
  1170:                                   TF);
  1171: 
  1172:   assert(cast<ConstantPoolSDNode>(T)->getTargetFlags() == TF &&
  1173:          "Inconsistent target flag encountered");
  1174: 
  1175:   if (IsPositionIndependent)
  1176:     return DAG.getNode(HexagonISD::AT_PCREL, SDLoc(Op), ValTy, T);
  1177:   return DAG.getNode(HexagonISD::CP, SDLoc(Op), ValTy, T);
  1178: }
  1179: 
  1180: SDValue
  1181: HexagonTargetLowering::LowerJumpTable(SDValue Op, SelectionDAG &DAG) const {
  1182:   EVT VT = Op.getValueType();
  1183:   int Idx = cast<JumpTableSDNode>(Op)->getIndex();
  1184:   if (isPositionIndependent()) {
  1185:     SDValue T = DAG.getTargetJumpTable(Idx, VT, HexagonII::MO_PCREL);
  1186:     return DAG.getNode(HexagonISD::AT_PCREL, SDLoc(Op), VT, T);
  1187:   }
  1188: 
  1189:   SDValue T = DAG.getTargetJumpTable(Idx, VT);
  1190:   return DAG.getNode(HexagonISD::JT, SDLoc(Op), VT, T);
  1191: }
  1192: 
  1193: SDValue
  1194: HexagonTargetLowering::LowerRETURNADDR(SDValue Op, SelectionDAG &DAG) const {
  1195:   const HexagonRegisterInfo &HRI = *Subtarget.getRegisterInfo();
  1196:   MachineFunction &MF = DAG.getMachineFunction();
  1197:   MachineFrameInfo &MFI = MF.getFrameInfo();
  1198:   MFI.setReturnAddressIsTaken(true);
  1199: 
  1200:   EVT VT = Op.getValueType();
```
- EN: It declares or implements routines such as getStore, push_back, getNode, HexagonTargetLowering::LowerVACOPY, ... (40 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonTargetLowering, HexagonII, HexagonISD, HexagonRegisterInfo, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getStore, push_back, getNode, HexagonTargetLowering::LowerVACOPY, ... (40 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonTargetLowering, HexagonII, HexagonISD, HexagonRegisterInfo，说明了它与同级后端组件的连接关系。

### Lines 1201-1400 / 第 1201-1400 行

```cpp
  1201:   SDLoc dl(Op);
  1202:   unsigned Depth = Op.getConstantOperandVal(0);
  1203:   if (Depth) {
  1204:     SDValue FrameAddr = LowerFRAMEADDR(Op, DAG);
  1205:     SDValue Offset = DAG.getConstant(4, dl, MVT::i32);
  1206:     return DAG.getLoad(VT, dl, DAG.getEntryNode(),
  1207:                        DAG.getNode(ISD::ADD, dl, VT, FrameAddr, Offset),
  1208:                        MachinePointerInfo());
  1209:   }
  1210: 
  1211:   // Return LR, which contains the return address. Mark it an implicit live-in.
  1212:   Register Reg = MF.addLiveIn(HRI.getRARegister(), getRegClassFor(MVT::i32));
  1213:   return DAG.getCopyFromReg(DAG.getEntryNode(), dl, Reg, VT);
  1214: }
  1215: 
  1216: SDValue
  1217: HexagonTargetLowering::LowerFRAMEADDR(SDValue Op, SelectionDAG &DAG) const {
  1218:   const HexagonRegisterInfo &HRI = *Subtarget.getRegisterInfo();
  1219:   MachineFrameInfo &MFI = DAG.getMachineFunction().getFrameInfo();
  1220:   MFI.setFrameAddressIsTaken(true);
  1221: 
  1222:   EVT VT = Op.getValueType();
  1223:   SDLoc dl(Op);
  1224:   unsigned Depth = Op.getConstantOperandVal(0);
  1225:   SDValue FrameAddr = DAG.getCopyFromReg(DAG.getEntryNode(), dl,
  1226:                                          HRI.getFrameRegister(), VT);
  1227:   while (Depth--)
  1228:     FrameAddr = DAG.getLoad(VT, dl, DAG.getEntryNode(), FrameAddr,
  1229:                             MachinePointerInfo());
  1230:   return FrameAddr;
  1231: }
  1232: 
  1233: SDValue
  1234: HexagonTargetLowering::LowerATOMIC_FENCE(SDValue Op, SelectionDAG& DAG) const {
  1235:   SDLoc dl(Op);
  1236:   return DAG.getNode(HexagonISD::BARRIER, dl, MVT::Other, Op.getOperand(0));
  1237: }
  1238: 
  1239: SDValue
  1240: HexagonTargetLowering::LowerGLOBALADDRESS(SDValue Op, SelectionDAG &DAG) const {
  1241:   SDLoc dl(Op);
  1242:   auto *GAN = cast<GlobalAddressSDNode>(Op);
  1243:   auto PtrVT = getPointerTy(DAG.getDataLayout());
  1244:   auto *GV = GAN->getGlobal();
  1245:   int64_t Offset = GAN->getOffset();
  1246: 
  1247:   auto &HLOF = *HTM.getObjFileLowering();
  1248:   Reloc::Model RM = HTM.getRelocationModel();
  1249: 
  1250:   if (RM == Reloc::Static) {
  1251:     SDValue GA = DAG.getTargetGlobalAddress(GV, dl, PtrVT, Offset);
  1252:     const GlobalObject *GO = GV->getAliaseeObject();
  1253:     if (GO && Subtarget.useSmallData() && HLOF.isGlobalInSmallSection(GO, HTM))
  1254:       return DAG.getNode(HexagonISD::CONST32_GP, dl, PtrVT, GA);
  1255:     return DAG.getNode(HexagonISD::CONST32, dl, PtrVT, GA);
  1256:   }
  1257: 
  1258:   bool UsePCRel = getTargetMachine().shouldAssumeDSOLocal(GV);
  1259:   if (UsePCRel) {
  1260:     SDValue GA = DAG.getTargetGlobalAddress(GV, dl, PtrVT, Offset,
  1261:                                             HexagonII::MO_PCREL);
  1262:     return DAG.getNode(HexagonISD::AT_PCREL, dl, PtrVT, GA);
  1263:   }
  1264: 
  1265:   // Use GOT index.
  1266:   SDValue GOT = DAG.getGLOBAL_OFFSET_TABLE(PtrVT);
  1267:   SDValue GA = DAG.getTargetGlobalAddress(GV, dl, PtrVT, 0, HexagonII::MO_GOT);
  1268:   SDValue Off = DAG.getConstant(Offset, dl, MVT::i32);
  1269:   return DAG.getNode(HexagonISD::AT_GOT, dl, PtrVT, GOT, GA, Off);
  1270: }
  1271: 
  1272: // Specifies that for loads and stores VT can be promoted to PromotedLdStVT.
  1273: SDValue
  1274: HexagonTargetLowering::LowerBlockAddress(SDValue Op, SelectionDAG &DAG) const {
  1275:   const BlockAddress *BA = cast<BlockAddressSDNode>(Op)->getBlockAddress();
  1276:   SDLoc dl(Op);
  1277:   EVT PtrVT = getPointerTy(DAG.getDataLayout());
  1278: 
  1279:   Reloc::Model RM = HTM.getRelocationModel();
  1280:   if (RM == Reloc::Static) {
  1281:     SDValue A = DAG.getTargetBlockAddress(BA, PtrVT);
  1282:     return DAG.getNode(HexagonISD::CONST32_GP, dl, PtrVT, A);
  1283:   }
  1284: 
  1285:   SDValue A = DAG.getTargetBlockAddress(BA, PtrVT, 0, HexagonII::MO_PCREL);
  1286:   return DAG.getNode(HexagonISD::AT_PCREL, dl, PtrVT, A);
  1287: }
  1288: 
  1289: SDValue
  1290: HexagonTargetLowering::LowerGLOBAL_OFFSET_TABLE(SDValue Op, SelectionDAG &DAG)
  1291:       const {
  1292:   EVT PtrVT = getPointerTy(DAG.getDataLayout());
  1293:   SDValue GOTSym = DAG.getTargetExternalSymbol(HEXAGON_GOT_SYM_NAME, PtrVT,
  1294:                                                HexagonII::MO_PCREL);
  1295:   return DAG.getNode(HexagonISD::AT_PCREL, SDLoc(Op), PtrVT, GOTSym);
  1296: }
  1297: 
  1298: SDValue
  1299: HexagonTargetLowering::GetDynamicTLSAddr(SelectionDAG &DAG, SDValue Chain,
  1300:       GlobalAddressSDNode *GA, SDValue Glue, EVT PtrVT, unsigned ReturnReg,
  1301:       unsigned char OperandFlags) const {
  1302:   MachineFunction &MF = DAG.getMachineFunction();
  1303:   MachineFrameInfo &MFI = MF.getFrameInfo();
  1304:   SDVTList NodeTys = DAG.getVTList(MVT::Other, MVT::Glue);
  1305:   SDLoc dl(GA);
  1306:   SDValue TGA = DAG.getTargetGlobalAddress(GA->getGlobal(), dl,
  1307:                                            GA->getValueType(0),
  1308:                                            GA->getOffset(),
  1309:                                            OperandFlags);
  1310:   // Create Operands for the call.The Operands should have the following:
  1311:   // 1. Chain SDValue
  1312:   // 2. Callee which in this case is the Global address value.
  1313:   // 3. Registers live into the call.In this case its R0, as we
  1314:   //    have just one argument to be passed.
  1315:   // 4. Glue.
  1316:   // Note: The order is important.
  1317: 
  1318:   const auto &HRI = *Subtarget.getRegisterInfo();
  1319:   const uint32_t *Mask = HRI.getCallPreservedMask(MF, CallingConv::C);
  1320:   assert(Mask && "Missing call preserved mask for calling convention");
  1321:   SDValue Ops[] = { Chain, TGA, DAG.getRegister(Hexagon::R0, PtrVT),
  1322:                     DAG.getRegisterMask(Mask), Glue };
  1323:   Chain = DAG.getNode(HexagonISD::CALL, dl, NodeTys, Ops);
  1324: 
  1325:   // Inform MFI that function has calls.
  1326:   MFI.setAdjustsStack(true);
  1327: 
  1328:   Glue = Chain.getValue(1);
  1329:   return DAG.getCopyFromReg(Chain, dl, ReturnReg, PtrVT, Glue);
  1330: }
  1331: 
  1332: //
  1333: // Lower using the initial executable model for TLS addresses
  1334: //
  1335: SDValue
  1336: HexagonTargetLowering::LowerToTLSInitialExecModel(GlobalAddressSDNode *GA,
  1337:       SelectionDAG &DAG) const {
  1338:   SDLoc dl(GA);
  1339:   int64_t Offset = GA->getOffset();
  1340:   auto PtrVT = getPointerTy(DAG.getDataLayout());
  1341: 
  1342:   // Get the thread pointer.
  1343:   SDValue TP = DAG.getCopyFromReg(DAG.getEntryNode(), dl, Hexagon::UGP, PtrVT);
  1344: 
  1345:   bool IsPositionIndependent = isPositionIndependent();
  1346:   unsigned char TF =
  1347:       IsPositionIndependent ? HexagonII::MO_IEGOT : HexagonII::MO_IE;
  1348: 
  1349:   // First generate the TLS symbol address
  1350:   SDValue TGA = DAG.getTargetGlobalAddress(GA->getGlobal(), dl, PtrVT,
  1351:                                            Offset, TF);
  1352: 
  1353:   SDValue Sym = DAG.getNode(HexagonISD::CONST32, dl, PtrVT, TGA);
  1354: 
  1355:   if (IsPositionIndependent) {
  1356:     // Generate the GOT pointer in case of position independent code
  1357:     SDValue GOT = LowerGLOBAL_OFFSET_TABLE(Sym, DAG);
  1358: 
  1359:     // Add the TLS Symbol address to GOT pointer.This gives
  1360:     // GOT relative relocation for the symbol.
  1361:     Sym = DAG.getNode(ISD::ADD, dl, PtrVT, GOT, Sym);
  1362:   }
  1363: 
  1364:   // Load the offset value for TLS symbol.This offset is relative to
  1365:   // thread pointer.
  1366:   SDValue LoadOffset =
  1367:       DAG.getLoad(PtrVT, dl, DAG.getEntryNode(), Sym, MachinePointerInfo());
  1368: 
  1369:   // Address of the thread local variable is the add of thread
  1370:   // pointer and the offset of the variable.
  1371:   return DAG.getNode(ISD::ADD, dl, PtrVT, TP, LoadOffset);
  1372: }
  1373: 
  1374: //
  1375: // Lower using the local executable model for TLS addresses
  1376: //
  1377: SDValue
  1378: HexagonTargetLowering::LowerToTLSLocalExecModel(GlobalAddressSDNode *GA,
  1379:       SelectionDAG &DAG) const {
  1380:   SDLoc dl(GA);
  1381:   int64_t Offset = GA->getOffset();
  1382:   auto PtrVT = getPointerTy(DAG.getDataLayout());
  1383: 
  1384:   // Get the thread pointer.
  1385:   SDValue TP = DAG.getCopyFromReg(DAG.getEntryNode(), dl, Hexagon::UGP, PtrVT);
  1386:   // Generate the TLS symbol address
  1387:   SDValue TGA = DAG.getTargetGlobalAddress(GA->getGlobal(), dl, PtrVT, Offset,
  1388:                                            HexagonII::MO_TPREL);
  1389:   SDValue Sym = DAG.getNode(HexagonISD::CONST32, dl, PtrVT, TGA);
  1390: 
  1391:   // Address of the thread local variable is the add of thread
  1392:   // pointer and the offset of the variable.
  1393:   return DAG.getNode(ISD::ADD, dl, PtrVT, TP, Sym);
  1394: }
  1395: 
  1396: //
  1397: // Lower using the general dynamic model for TLS addresses
  1398: //
  1399: SDValue
  1400: HexagonTargetLowering::LowerToTLSGeneralDynamicModel(GlobalAddressSDNode *GA,
```
- EN: It declares or implements routines such as dl, getConstantOperandVal, LowerFRAMEADDR, getConstant, ... (41 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonTargetLowering, HexagonRegisterInfo, HexagonISD, HexagonII, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 dl, getConstantOperandVal, LowerFRAMEADDR, getConstant, ... (41 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonTargetLowering, HexagonRegisterInfo, HexagonISD, HexagonII，说明了它与同级后端组件的连接关系。

### Lines 1401-1600 / 第 1401-1600 行

```cpp
  1401:       SelectionDAG &DAG) const {
  1402:   SDLoc dl(GA);
  1403:   int64_t Offset = GA->getOffset();
  1404:   auto PtrVT = getPointerTy(DAG.getDataLayout());
  1405: 
  1406:   // First generate the TLS symbol address
  1407:   SDValue TGA = DAG.getTargetGlobalAddress(GA->getGlobal(), dl, PtrVT, Offset,
  1408:                                            HexagonII::MO_GDGOT);
  1409: 
  1410:   // Then, generate the GOT pointer
  1411:   SDValue GOT = LowerGLOBAL_OFFSET_TABLE(TGA, DAG);
  1412: 
  1413:   // Add the TLS symbol and the GOT pointer
  1414:   SDValue Sym = DAG.getNode(HexagonISD::CONST32, dl, PtrVT, TGA);
  1415:   SDValue Chain = DAG.getNode(ISD::ADD, dl, PtrVT, GOT, Sym);
  1416: 
  1417:   // Copy over the argument to R0
  1418:   SDValue InGlue;
  1419:   Chain = DAG.getCopyToReg(DAG.getEntryNode(), dl, Hexagon::R0, Chain, InGlue);
  1420:   InGlue = Chain.getValue(1);
  1421: 
  1422:   unsigned Flags = DAG.getSubtarget<HexagonSubtarget>().useLongCalls()
  1423:                        ? HexagonII::MO_GDPLT | HexagonII::HMOTF_ConstExtended
  1424:                        : HexagonII::MO_GDPLT;
  1425: 
  1426:   return GetDynamicTLSAddr(DAG, Chain, GA, InGlue, PtrVT,
  1427:                            Hexagon::R0, Flags);
  1428: }
  1429: 
  1430: //
  1431: // Lower TLS addresses.
  1432: //
  1433: // For now for dynamic models, we only support the general dynamic model.
  1434: //
  1435: SDValue
  1436: HexagonTargetLowering::LowerGlobalTLSAddress(SDValue Op,
  1437:       SelectionDAG &DAG) const {
  1438:   GlobalAddressSDNode *GA = cast<GlobalAddressSDNode>(Op);
  1439: 
  1440:   switch (HTM.getTLSModel(GA->getGlobal())) {
  1441:     case TLSModel::GeneralDynamic:
  1442:     case TLSModel::LocalDynamic:
  1443:       return LowerToTLSGeneralDynamicModel(GA, DAG);
  1444:     case TLSModel::InitialExec:
  1445:       return LowerToTLSInitialExecModel(GA, DAG);
  1446:     case TLSModel::LocalExec:
  1447:       return LowerToTLSLocalExecModel(GA, DAG);
  1448:   }
  1449:   llvm_unreachable("Bogus TLS model");
  1450: }
  1451: 
  1452: //===----------------------------------------------------------------------===//
  1453: // TargetLowering Implementation
  1454: //===----------------------------------------------------------------------===//
  1455: 
  1456: HexagonTargetLowering::HexagonTargetLowering(const TargetMachine &TM,
  1457:                                              const HexagonSubtarget &ST)
  1458:     : TargetLowering(TM, ST),
  1459:       HTM(static_cast<const HexagonTargetMachine &>(TM)), Subtarget(ST) {
  1460:   auto &HRI = *Subtarget.getRegisterInfo();
  1461: 
  1462:   setPrefLoopAlignment(Align(16));
  1463:   setMinFunctionAlignment(Align(4));
  1464:   setPrefFunctionAlignment(Align(16));
  1465:   setStackPointerRegisterToSaveRestore(HRI.getStackRegister());
  1466:   setBooleanContents(TargetLoweringBase::UndefinedBooleanContent);
  1467:   setBooleanVectorContents(TargetLoweringBase::UndefinedBooleanContent);
  1468: 
  1469:   setMaxAtomicSizeInBitsSupported(64);
  1470:   setMinCmpXchgSizeInBits(32);
  1471: 
  1472:   if (EnableHexSDNodeSched)
  1473:     setSchedulingPreference(Sched::VLIW);
  1474:   else
  1475:     setSchedulingPreference(Sched::Source);
  1476: 
  1477:   // Limits for inline expansion of memcpy/memmove
  1478:   MaxStoresPerMemcpy = 6;
  1479:   MaxStoresPerMemcpyOptSize = 4;
  1480:   MaxStoresPerMemmove = 6;
  1481:   MaxStoresPerMemmoveOptSize = 4;
  1482:   MaxStoresPerMemset = 8;
  1483:   MaxStoresPerMemsetOptSize = 4;
  1484: 
  1485:   setTargetDAGCombine(ISD::VECREDUCE_ADD);
  1486: 
  1487:   //
  1488:   // Set up register classes.
  1489:   //
  1490: 
  1491:   addRegisterClass(MVT::i1,    &Hexagon::PredRegsRegClass);
  1492:   addRegisterClass(MVT::v2i1,  &Hexagon::PredRegsRegClass);  // bbbbaaaa
  1493:   addRegisterClass(MVT::v4i1,  &Hexagon::PredRegsRegClass);  // ddccbbaa
  1494:   addRegisterClass(MVT::v8i1,  &Hexagon::PredRegsRegClass);  // hgfedcba
  1495:   addRegisterClass(MVT::i32,   &Hexagon::IntRegsRegClass);
  1496:   addRegisterClass(MVT::v2i16, &Hexagon::IntRegsRegClass);
  1497:   addRegisterClass(MVT::v4i8,  &Hexagon::IntRegsRegClass);
  1498:   addRegisterClass(MVT::i64,   &Hexagon::DoubleRegsRegClass);
  1499:   addRegisterClass(MVT::v8i8,  &Hexagon::DoubleRegsRegClass);
  1500:   addRegisterClass(MVT::v4i16, &Hexagon::DoubleRegsRegClass);
  1501:   addRegisterClass(MVT::v2i32, &Hexagon::DoubleRegsRegClass);
  1502: 
  1503:   addRegisterClass(MVT::f32, &Hexagon::IntRegsRegClass);
  1504:   addRegisterClass(MVT::f64, &Hexagon::DoubleRegsRegClass);
  1505: 
  1506:   //
  1507:   // Handling of scalar operations.
  1508:   //
  1509:   // All operations default to "legal", except:
  1510:   // - indexed loads and stores (pre-/post-incremented),
  1511:   // - ANY_EXTEND_VECTOR_INREG, ATOMIC_CMP_SWAP_WITH_SUCCESS, CONCAT_VECTORS,
  1512:   //   ConstantFP, FCEIL, FCOPYSIGN, FEXP, FEXP2, FFLOOR, FGETSIGN,
  1513:   //   FLOG, FLOG2, FLOG10, FMAXIMUMNUM, FMINIMUMNUM, FNEARBYINT, FRINT, FROUND,
  1514:   //   TRAP, FTRUNC, PREFETCH, SIGN_EXTEND_VECTOR_INREG,
  1515:   //   ZERO_EXTEND_VECTOR_INREG,
  1516:   // which default to "expand" for at least one type.
  1517: 
  1518:   // Misc operations.
  1519:   setOperationAction(ISD::ConstantFP,           MVT::f32,   Legal);
  1520:   setOperationAction(ISD::ConstantFP,           MVT::f64,   Legal);
  1521:   setOperationAction(ISD::TRAP,                 MVT::Other, Legal);
  1522:   setOperationAction(ISD::DEBUGTRAP,            MVT::Other, Legal);
  1523:   setOperationAction(ISD::ConstantPool,         MVT::i32,   Custom);
  1524:   setOperationAction(ISD::JumpTable,            MVT::i32,   Custom);
  1525:   setOperationAction(ISD::BUILD_PAIR,           MVT::i64,   Expand);
  1526:   setOperationAction(ISD::SIGN_EXTEND_INREG,    MVT::i1,    Expand);
  1527:   setOperationAction(ISD::INLINEASM,            MVT::Other, Custom);
  1528:   setOperationAction(ISD::INLINEASM_BR,         MVT::Other, Custom);
  1529:   setOperationAction(ISD::PREFETCH,             MVT::Other, Custom);
  1530:   setOperationAction(ISD::READCYCLECOUNTER, MVT::i64, Legal);
  1531:   setOperationAction(ISD::READSTEADYCOUNTER, MVT::i64, Legal);
  1532:   setOperationAction(ISD::INTRINSIC_WO_CHAIN, MVT::Other, Custom);
  1533:   setOperationAction(ISD::INTRINSIC_VOID,       MVT::Other, Custom);
  1534:   setOperationAction(ISD::EH_RETURN,            MVT::Other, Custom);
  1535:   setOperationAction(ISD::GLOBAL_OFFSET_TABLE,  MVT::i32,   Custom);
  1536:   setOperationAction(ISD::GlobalTLSAddress,     MVT::i32,   Custom);
  1537:   setOperationAction(ISD::ATOMIC_FENCE,         MVT::Other, Custom);
  1538: 
  1539:   // Custom legalize GlobalAddress nodes into CONST32.
  1540:   setOperationAction(ISD::GlobalAddress, MVT::i32, Custom);
  1541:   setOperationAction(ISD::GlobalAddress, MVT::i8,  Custom);
  1542:   setOperationAction(ISD::BlockAddress,  MVT::i32, Custom);
  1543: 
  1544:   // Hexagon needs to optimize cases with negative constants.
  1545:   setOperationAction(ISD::SETCC, MVT::i8,    Custom);
  1546:   setOperationAction(ISD::SETCC, MVT::i16,   Custom);
  1547:   setOperationAction(ISD::SETCC, MVT::v4i8,  Custom);
  1548:   setOperationAction(ISD::SETCC, MVT::v2i16, Custom);
  1549: 
  1550:   // VASTART needs to be custom lowered to use the VarArgsFrameIndex.
  1551:   setOperationAction(ISD::VASTART, MVT::Other, Custom);
  1552:   setOperationAction(ISD::VAEND,   MVT::Other, Expand);
  1553:   setOperationAction(ISD::VAARG,   MVT::Other, Expand);
  1554:   if (Subtarget.isEnvironmentMusl())
  1555:     setOperationAction(ISD::VACOPY, MVT::Other, Custom);
  1556:   else
  1557:     setOperationAction(ISD::VACOPY,  MVT::Other, Expand);
  1558: 
  1559:   setOperationAction(ISD::STACKSAVE, MVT::Other, Expand);
  1560:   setOperationAction(ISD::STACKRESTORE, MVT::Other, Expand);
  1561:   setOperationAction(ISD::DYNAMIC_STACKALLOC, MVT::i32, Custom);
  1562: 
  1563:   if (EmitJumpTables)
  1564:     setMinimumJumpTableEntries(MinimumJumpTables);
  1565:   else
  1566:     setMinimumJumpTableEntries(std::numeric_limits<unsigned>::max());
  1567:   setOperationAction(ISD::BR_JT, MVT::Other, Expand);
  1568: 
  1569:   for (unsigned LegalIntOp :
  1570:        {ISD::ABS, ISD::SMIN, ISD::SMAX, ISD::UMIN, ISD::UMAX}) {
  1571:     setOperationAction(LegalIntOp, MVT::i32, Legal);
  1572:     setOperationAction(LegalIntOp, MVT::i64, Legal);
  1573:   }
  1574: 
  1575:   // Hexagon has A4_addp_c and A4_subp_c that take and generate a carry bit,
  1576:   // but they only operate on i64.
  1577:   for (MVT VT : MVT::integer_valuetypes()) {
  1578:     setOperationAction(ISD::UADDO, VT, Custom);
  1579:     setOperationAction(ISD::USUBO, VT, Custom);
  1580:     setOperationAction(ISD::SADDO, VT, Expand);
  1581:     setOperationAction(ISD::SSUBO, VT, Expand);
  1582:     setOperationAction(ISD::UADDO_CARRY, VT, Expand);
  1583:     setOperationAction(ISD::USUBO_CARRY, VT, Expand);
  1584:   }
  1585:   setOperationAction(ISD::UADDO_CARRY, MVT::i64, Custom);
  1586:   setOperationAction(ISD::USUBO_CARRY, MVT::i64, Custom);
  1587: 
  1588:   setOperationAction(ISD::CTLZ, MVT::i8,  Promote);
  1589:   setOperationAction(ISD::CTLZ, MVT::i16, Promote);
  1590:   setOperationAction(ISD::CTTZ, MVT::i8,  Promote);
  1591:   setOperationAction(ISD::CTTZ, MVT::i16, Promote);
  1592: 
  1593:   // Popcount can count # of 1s in i64 but returns i32.
  1594:   setOperationAction(ISD::CTPOP, MVT::i8,  Promote);
  1595:   setOperationAction(ISD::CTPOP, MVT::i16, Promote);
  1596:   setOperationAction(ISD::CTPOP, MVT::i32, Promote);
  1597:   setOperationAction(ISD::CTPOP, MVT::i64, Legal);
  1598: 
  1599:   setOperationAction(ISD::BITREVERSE, MVT::i32, Legal);
  1600:   setOperationAction(ISD::BITREVERSE, MVT::i64, Legal);
```
- EN: It declares or implements routines such as dl, getOffset, getPointerTy, getTargetGlobalAddress, ... (31 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonII, HexagonISD, HexagonSubtarget, HexagonTargetLowering, ... (5 total), showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 dl, getOffset, getPointerTy, getTargetGlobalAddress, ... (31 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonII, HexagonISD, HexagonSubtarget, HexagonTargetLowering, ... (5 total)，说明了它与同级后端组件的连接关系。

### Lines 1601-1800 / 第 1601-1800 行

```cpp
  1601:   setOperationAction(ISD::BSWAP, MVT::i32, Legal);
  1602:   setOperationAction(ISD::BSWAP, MVT::i64, Legal);
  1603: 
  1604:   setOperationAction(ISD::FSHL, MVT::i32, Legal);
  1605:   setOperationAction(ISD::FSHL, MVT::i64, Legal);
  1606:   setOperationAction(ISD::FSHR, MVT::i32, Legal);
  1607:   setOperationAction(ISD::FSHR, MVT::i64, Legal);
  1608: 
  1609:   for (unsigned IntExpOp :
  1610:        {ISD::SDIV,      ISD::UDIV,      ISD::SREM,      ISD::UREM,
  1611:         ISD::SDIVREM,   ISD::UDIVREM,   ISD::ROTL,      ISD::ROTR,
  1612:         ISD::SHL_PARTS, ISD::SRA_PARTS, ISD::SRL_PARTS,
  1613:         ISD::SMUL_LOHI, ISD::UMUL_LOHI}) {
  1614:     for (MVT VT : MVT::integer_valuetypes())
  1615:       setOperationAction(IntExpOp, VT, Expand);
  1616:   }
  1617:   for (MVT VT : MVT::fp_valuetypes()) {
  1618:     for (unsigned FPExpOp : {ISD::FDIV, ISD::FSQRT, ISD::FSIN, ISD::FCOS,
  1619:                              ISD::FSINCOS, ISD::FPOW, ISD::FCOPYSIGN})
  1620:       setOperationAction(FPExpOp, VT, Expand);
  1621: 
  1622:     setOperationAction(ISD::FREM, VT, LibCall);
  1623:   }
  1624: 
  1625:   // No extending loads from i32.
  1626:   for (MVT VT : MVT::integer_valuetypes()) {
  1627:     setLoadExtAction(ISD::ZEXTLOAD, VT, MVT::i32, Expand);
  1628:     setLoadExtAction(ISD::SEXTLOAD, VT, MVT::i32, Expand);
  1629:     setLoadExtAction(ISD::EXTLOAD,  VT, MVT::i32, Expand);
  1630:   }
  1631:   // Turn FP truncstore into trunc + store.
  1632:   setTruncStoreAction(MVT::f64, MVT::f32, Expand);
  1633:   setTruncStoreAction(MVT::f32, MVT::bf16, Expand);
  1634:   setTruncStoreAction(MVT::f64, MVT::bf16, Expand);
  1635:   // Turn FP extload into load/fpextend.
  1636:   for (MVT VT : MVT::fp_valuetypes())
  1637:     setLoadExtAction(ISD::EXTLOAD, VT, MVT::f32, Expand);
  1638: 
  1639:   // Expand BR_CC and SELECT_CC for all integer and fp types.
  1640:   for (MVT VT : MVT::integer_valuetypes()) {
  1641:     setOperationAction(ISD::BR_CC,     VT, Expand);
  1642:     setOperationAction(ISD::SELECT_CC, VT, Expand);
  1643:   }
  1644:   for (MVT VT : MVT::fp_valuetypes()) {
  1645:     setOperationAction(ISD::BR_CC,     VT, Expand);
  1646:     setOperationAction(ISD::SELECT_CC, VT, Expand);
  1647:   }
  1648:   setOperationAction(ISD::BR_CC, MVT::Other, Expand);
  1649: 
  1650:   //
  1651:   // Handling of vector operations.
  1652:   //
  1653: 
  1654:   // Set the action for vector operations to "expand", then override it with
  1655:   // either "custom" or "legal" for specific cases.
  1656:   // clang-format off
  1657:   static const unsigned VectExpOps[] = {
  1658:     // Integer arithmetic:
  1659:     ISD::ADD,     ISD::SUB,     ISD::MUL,     ISD::SDIV,      ISD::UDIV,
  1660:     ISD::SREM,    ISD::UREM,    ISD::SDIVREM, ISD::UDIVREM,   ISD::SADDO,
  1661:     ISD::UADDO,   ISD::SSUBO,   ISD::USUBO,   ISD::SMUL_LOHI, ISD::UMUL_LOHI,
  1662:     // Logical/bit:
  1663:     ISD::AND,     ISD::OR,      ISD::XOR,     ISD::ROTL,    ISD::ROTR,
  1664:     ISD::CTPOP,   ISD::CTLZ,    ISD::CTTZ,    ISD::BSWAP,   ISD::BITREVERSE,
  1665:     // Floating point arithmetic/math functions:
  1666:     ISD::FADD,    ISD::FSUB,    ISD::FMUL,    ISD::FMA,     ISD::FDIV,
  1667:     ISD::FREM,    ISD::FNEG,    ISD::FABS,    ISD::FSQRT,   ISD::FSIN,
  1668:     ISD::FCOS,    ISD::FPOW,    ISD::FLOG,    ISD::FLOG2,
  1669:     ISD::FLOG10,  ISD::FEXP,    ISD::FEXP2,   ISD::FCEIL,   ISD::FTRUNC,
  1670:     ISD::FRINT,   ISD::FNEARBYINT,            ISD::FROUND,  ISD::FFLOOR,
  1671:     ISD::FMINIMUMNUM,           ISD::FMAXIMUMNUM,
  1672:     ISD::FSINCOS, ISD::FLDEXP,
  1673:     // Misc:
  1674:     ISD::BR_CC,   ISD::SELECT_CC,             ISD::ConstantPool,
  1675:     // Vector:
  1676:     ISD::BUILD_VECTOR,          ISD::SCALAR_TO_VECTOR,
  1677:     ISD::EXTRACT_VECTOR_ELT,    ISD::INSERT_VECTOR_ELT,
  1678:     ISD::EXTRACT_SUBVECTOR,     ISD::INSERT_SUBVECTOR,
  1679:     ISD::CONCAT_VECTORS,        ISD::VECTOR_SHUFFLE,
  1680:     ISD::SPLAT_VECTOR,
  1681:   };
  1682:   // clang-format on
  1683: 
  1684:   for (MVT VT : MVT::fixedlen_vector_valuetypes()) {
  1685:     for (unsigned VectExpOp : VectExpOps)
  1686:       setOperationAction(VectExpOp, VT, Expand);
  1687: 
  1688:     // Expand all extending loads and truncating stores:
  1689:     for (MVT TargetVT : MVT::fixedlen_vector_valuetypes()) {
  1690:       if (TargetVT == VT)
  1691:         continue;
  1692:       setLoadExtAction(ISD::EXTLOAD, TargetVT, VT, Expand);
  1693:       setLoadExtAction(ISD::ZEXTLOAD, TargetVT, VT, Expand);
  1694:       setLoadExtAction(ISD::SEXTLOAD, TargetVT, VT, Expand);
  1695:       setTruncStoreAction(VT, TargetVT, Expand);
  1696:     }
  1697: 
  1698:     // Normalize all inputs to SELECT to be vectors of i32.
  1699:     if (VT.getVectorElementType() != MVT::i32) {
  1700:       MVT VT32 = MVT::getVectorVT(MVT::i32, VT.getSizeInBits()/32);
  1701:       setOperationAction(ISD::SELECT, VT, Promote);
  1702:       AddPromotedToType(ISD::SELECT, VT, VT32);
  1703:     }
  1704:     setOperationAction(ISD::SRA, VT, Custom);
  1705:     setOperationAction(ISD::SHL, VT, Custom);
  1706:     setOperationAction(ISD::SRL, VT, Custom);
  1707:   }
  1708: 
  1709:   setOperationAction(ISD::SADDSAT, MVT::i32, Legal);
  1710:   setOperationAction(ISD::SADDSAT, MVT::i64, Legal);
  1711: 
  1712:   // Extending loads from (native) vectors of i8 into (native) vectors of i16
  1713:   // are legal.
  1714:   setLoadExtAction(ISD::EXTLOAD,  MVT::v2i16, MVT::v2i8, Legal);
  1715:   setLoadExtAction(ISD::ZEXTLOAD, MVT::v2i16, MVT::v2i8, Legal);
  1716:   setLoadExtAction(ISD::SEXTLOAD, MVT::v2i16, MVT::v2i8, Legal);
  1717:   setLoadExtAction(ISD::EXTLOAD,  MVT::v4i16, MVT::v4i8, Legal);
  1718:   setLoadExtAction(ISD::ZEXTLOAD, MVT::v4i16, MVT::v4i8, Legal);
  1719:   setLoadExtAction(ISD::SEXTLOAD, MVT::v4i16, MVT::v4i8, Legal);
  1720: 
  1721:   setOperationAction(ISD::SIGN_EXTEND_INREG, MVT::v2i8,  Legal);
  1722:   setOperationAction(ISD::SIGN_EXTEND_INREG, MVT::v2i16, Legal);
  1723:   setOperationAction(ISD::SIGN_EXTEND_INREG, MVT::v2i32, Legal);
  1724: 
  1725:   // Types natively supported:
  1726:   for (MVT NativeVT : {MVT::v8i1, MVT::v4i1, MVT::v2i1, MVT::v4i8,
  1727:                        MVT::v8i8, MVT::v2i16, MVT::v4i16, MVT::v2i32}) {
  1728:     setOperationAction(ISD::BUILD_VECTOR,       NativeVT, Custom);
  1729:     setOperationAction(ISD::EXTRACT_VECTOR_ELT, NativeVT, Custom);
  1730:     setOperationAction(ISD::INSERT_VECTOR_ELT,  NativeVT, Custom);
  1731:     setOperationAction(ISD::EXTRACT_SUBVECTOR,  NativeVT, Custom);
  1732:     setOperationAction(ISD::INSERT_SUBVECTOR,   NativeVT, Custom);
  1733:     setOperationAction(ISD::CONCAT_VECTORS,     NativeVT, Custom);
  1734: 
  1735:     setOperationAction(ISD::ADD, NativeVT, Legal);
  1736:     setOperationAction(ISD::SUB, NativeVT, Legal);
  1737:     setOperationAction(ISD::MUL, NativeVT, Legal);
  1738:     setOperationAction(ISD::AND, NativeVT, Legal);
  1739:     setOperationAction(ISD::OR,  NativeVT, Legal);
  1740:     setOperationAction(ISD::XOR, NativeVT, Legal);
  1741: 
  1742:     if (NativeVT.getVectorElementType() != MVT::i1) {
  1743:       setOperationAction(ISD::SPLAT_VECTOR, NativeVT, Legal);
  1744:       setOperationAction(ISD::BSWAP,        NativeVT, Legal);
  1745:       setOperationAction(ISD::BITREVERSE,   NativeVT, Legal);
  1746:     }
  1747:   }
  1748: 
  1749:   for (MVT VT : {MVT::v8i8, MVT::v4i16, MVT::v2i32}) {
  1750:     setOperationAction(ISD::SMIN, VT, Legal);
  1751:     setOperationAction(ISD::SMAX, VT, Legal);
  1752:     setOperationAction(ISD::UMIN, VT, Legal);
  1753:     setOperationAction(ISD::UMAX, VT, Legal);
  1754:   }
  1755: 
  1756:   // Custom lower unaligned loads.
  1757:   // Also, for both loads and stores, verify the alignment of the address
  1758:   // in case it is a compile-time constant. This is a usability feature to
  1759:   // provide a meaningful error message to users.
  1760:   for (MVT VT : {MVT::i16, MVT::i32, MVT::v4i8, MVT::i64, MVT::v8i8,
  1761:                  MVT::v2i16, MVT::v4i16, MVT::v2i32}) {
  1762:     setOperationAction(ISD::LOAD,  VT, Custom);
  1763:     setOperationAction(ISD::STORE, VT, Custom);
  1764:   }
  1765: 
  1766:   // Custom-lower load/stores of boolean vectors.
  1767:   for (MVT VT : {MVT::v2i1, MVT::v4i1, MVT::v8i1}) {
  1768:     setOperationAction(ISD::LOAD,  VT, Custom);
  1769:     setOperationAction(ISD::STORE, VT, Custom);
  1770:   }
  1771: 
  1772:   // Normalize integer compares to EQ/GT/UGT
  1773:   for (MVT VT : {MVT::v2i16, MVT::v4i8, MVT::v8i8, MVT::v2i32, MVT::v4i16,
  1774:                  MVT::v2i32}) {
  1775:     setCondCodeAction(ISD::SETNE,  VT, Expand);
  1776:     setCondCodeAction(ISD::SETLE,  VT, Expand);
  1777:     setCondCodeAction(ISD::SETGE,  VT, Expand);
  1778:     setCondCodeAction(ISD::SETLT,  VT, Expand);
  1779:     setCondCodeAction(ISD::SETULE, VT, Expand);
  1780:     setCondCodeAction(ISD::SETUGE, VT, Expand);
  1781:     setCondCodeAction(ISD::SETULT, VT, Expand);
  1782:   }
  1783: 
  1784:   // Normalize boolean compares to [U]LE/[U]LT
  1785:   for (MVT VT : {MVT::i1, MVT::v2i1, MVT::v4i1, MVT::v8i1}) {
  1786:     setCondCodeAction(ISD::SETGE,  VT, Expand);
  1787:     setCondCodeAction(ISD::SETGT,  VT, Expand);
  1788:     setCondCodeAction(ISD::SETUGE, VT, Expand);
  1789:     setCondCodeAction(ISD::SETUGT, VT, Expand);
  1790:   }
  1791: 
  1792:   // Custom-lower bitcasts from i8 to v8i1.
  1793:   setOperationAction(ISD::BITCAST,        MVT::i8,    Custom);
  1794:   setOperationAction(ISD::SETCC,          MVT::v2i16, Custom);
  1795:   setOperationAction(ISD::VSELECT,        MVT::v4i8,  Custom);
  1796:   setOperationAction(ISD::VSELECT,        MVT::v2i16, Custom);
  1797:   setOperationAction(ISD::VECTOR_SHUFFLE, MVT::v4i8,  Custom);
  1798:   setOperationAction(ISD::VECTOR_SHUFFLE, MVT::v4i16, Custom);
  1799:   setOperationAction(ISD::VECTOR_SHUFFLE, MVT::v8i8,  Custom);
  1800: 
```
- EN: It declares or implements routines such as setOperationAction, setLoadExtAction, setTruncStoreAction, MVT::getVectorVT, ... (7 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明或实现了 setOperationAction, setLoadExtAction, setTruncStoreAction, MVT::getVectorVT, ... (7 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 1801-2000 / 第 1801-2000 行

```cpp
  1801:   // V5+.
  1802:   setOperationAction(ISD::FMA,  MVT::f64, Expand);
  1803:   setOperationAction(ISD::FADD, MVT::f64, Expand);
  1804:   setOperationAction(ISD::FSUB, MVT::f64, Expand);
  1805:   setOperationAction(ISD::FMUL, MVT::f64, Expand);
  1806:   setOperationAction(ISD::FDIV, MVT::f32, Custom);
  1807: 
  1808:   setOperationAction(ISD::FMINIMUMNUM, MVT::f32, Legal);
  1809:   setOperationAction(ISD::FMAXIMUMNUM, MVT::f32, Legal);
  1810: 
  1811:   setOperationAction(ISD::FP_TO_UINT, MVT::i1,  Promote);
  1812:   setOperationAction(ISD::FP_TO_UINT, MVT::i8,  Promote);
  1813:   setOperationAction(ISD::FP_TO_UINT, MVT::i16, Promote);
  1814:   setOperationAction(ISD::FP_TO_SINT, MVT::i1,  Promote);
  1815:   setOperationAction(ISD::FP_TO_SINT, MVT::i8,  Promote);
  1816:   setOperationAction(ISD::FP_TO_SINT, MVT::i16, Promote);
  1817:   setOperationAction(ISD::UINT_TO_FP, MVT::i1,  Promote);
  1818:   setOperationAction(ISD::UINT_TO_FP, MVT::i8,  Promote);
  1819:   setOperationAction(ISD::UINT_TO_FP, MVT::i16, Promote);
  1820:   setOperationAction(ISD::SINT_TO_FP, MVT::i1,  Promote);
  1821:   setOperationAction(ISD::SINT_TO_FP, MVT::i8,  Promote);
  1822:   setOperationAction(ISD::SINT_TO_FP, MVT::i16, Promote);
  1823: 
  1824:   // Special handling for half-precision floating point conversions.
  1825:   // Lower half float conversions into library calls.
  1826:   setOperationAction(ISD::FP16_TO_FP, MVT::f32, Expand);
  1827:   setOperationAction(ISD::FP16_TO_FP, MVT::f64, Expand);
  1828:   setOperationAction(ISD::FP_TO_FP16, MVT::f32, Expand);
  1829:   setOperationAction(ISD::FP_TO_FP16, MVT::f64, Expand);
  1830:   setOperationAction(ISD::BF16_TO_FP, MVT::f32, Expand);
  1831:   setOperationAction(ISD::BF16_TO_FP, MVT::f64, Expand);
  1832:   setOperationAction(ISD::FP_TO_BF16, MVT::f64, Expand);
  1833: 
  1834:   setLoadExtAction(ISD::EXTLOAD, MVT::f32, MVT::f16, Expand);
  1835:   setLoadExtAction(ISD::EXTLOAD, MVT::f64, MVT::f16, Expand);
  1836:   setLoadExtAction(ISD::EXTLOAD, MVT::f32, MVT::bf16, Expand);
  1837:   setLoadExtAction(ISD::EXTLOAD, MVT::f64, MVT::bf16, Expand);
  1838: 
  1839:   setTruncStoreAction(MVT::f32, MVT::f16, Expand);
  1840:   setTruncStoreAction(MVT::f64, MVT::f16, Expand);
  1841: 
  1842:   // Handling of indexed loads/stores: default is "expand".
  1843:   //
  1844:   for (MVT VT : {MVT::i8, MVT::i16, MVT::i32, MVT::i64, MVT::f32, MVT::f64,
  1845:                  MVT::v2i16, MVT::v2i32, MVT::v4i8, MVT::v4i16, MVT::v8i8}) {
  1846:     setIndexedLoadAction(ISD::POST_INC, VT, Legal);
  1847:     setIndexedStoreAction(ISD::POST_INC, VT, Legal);
  1848:   }
  1849: 
  1850:   // Subtarget-specific operation actions.
  1851:   //
  1852:   if (Subtarget.hasV60Ops()) {
  1853:     setOperationAction(ISD::ROTL, MVT::i32, Legal);
  1854:     setOperationAction(ISD::ROTL, MVT::i64, Legal);
  1855:     setOperationAction(ISD::ROTR, MVT::i32, Legal);
  1856:     setOperationAction(ISD::ROTR, MVT::i64, Legal);
  1857:   }
  1858:   if (Subtarget.hasV66Ops()) {
  1859:     setOperationAction(ISD::FADD, MVT::f64, Legal);
  1860:     setOperationAction(ISD::FSUB, MVT::f64, Legal);
  1861:   }
  1862:   if (Subtarget.hasV67Ops()) {
  1863:     setOperationAction(ISD::FMINIMUMNUM, MVT::f64, Legal);
  1864:     setOperationAction(ISD::FMAXIMUMNUM, MVT::f64, Legal);
  1865:     setOperationAction(ISD::FMUL,    MVT::f64, Legal);
  1866:   }
  1867: 
  1868:   setTargetDAGCombine(ISD::OR);
  1869:   setTargetDAGCombine(ISD::TRUNCATE);
  1870:   setTargetDAGCombine(ISD::VSELECT);
  1871: 
  1872:   if (Subtarget.useHVXOps())
  1873:     initializeHVXLowering();
  1874: 
  1875:   computeRegisterProperties(&HRI);
  1876: }
  1877: 
  1878: bool
  1879: HexagonTargetLowering::validateConstPtrAlignment(SDValue Ptr, Align NeedAlign,
  1880:       const SDLoc &dl, SelectionDAG &DAG) const {
  1881:   auto *CA = dyn_cast<ConstantSDNode>(Ptr);
  1882:   if (!CA)
  1883:     return true;
  1884:   unsigned Addr = CA->getZExtValue();
  1885:   Align HaveAlign =
  1886:       Addr != 0 ? Align(1ull << llvm::countr_zero(Addr)) : NeedAlign;
  1887:   if (HaveAlign >= NeedAlign)
  1888:     return true;
  1889: 
  1890:   static int DK_MisalignedTrap = llvm::getNextAvailablePluginDiagnosticKind();
  1891: 
  1892:   struct DiagnosticInfoMisalignedTrap : public DiagnosticInfo {
  1893:     DiagnosticInfoMisalignedTrap(StringRef M)
  1894:       : DiagnosticInfo(DK_MisalignedTrap, DS_Remark), Msg(M) {}
  1895:     void print(DiagnosticPrinter &DP) const override {
  1896:       DP << Msg;
  1897:     }
  1898:     static bool classof(const DiagnosticInfo *DI) {
  1899:       return DI->getKind() == DK_MisalignedTrap;
  1900:     }
  1901:     StringRef Msg;
  1902:   };
  1903: 
  1904:   std::string ErrMsg;
  1905:   raw_string_ostream O(ErrMsg);
  1906:   O << "Misaligned constant address: " << format_hex(Addr, 10)
  1907:     << " has alignment " << HaveAlign.value()
  1908:     << ", but the memory access requires " << NeedAlign.value();
  1909:   if (DebugLoc DL = dl.getDebugLoc())
  1910:     DL.print(O << ", at ");
  1911:   O << ". The instruction has been replaced with a trap.";
  1912: 
  1913:   DAG.getContext()->diagnose(DiagnosticInfoMisalignedTrap(O.str()));
  1914:   return false;
  1915: }
  1916: 
  1917: SDValue
  1918: HexagonTargetLowering::replaceMemWithUndef(SDValue Op, SelectionDAG &DAG)
  1919:       const {
  1920:   const SDLoc &dl(Op);
  1921:   auto *LS = cast<LSBaseSDNode>(Op.getNode());
  1922:   assert(!LS->isIndexed() && "Not expecting indexed ops on constant address");
  1923: 
  1924:   SDValue Chain = LS->getChain();
  1925:   SDValue Trap = DAG.getNode(ISD::TRAP, dl, MVT::Other, Chain);
  1926:   if (LS->getOpcode() == ISD::LOAD)
  1927:     return DAG.getMergeValues({DAG.getUNDEF(ty(Op)), Trap}, dl);
  1928:   return Trap;
  1929: }
  1930: 
  1931: // Bit-reverse Load Intrinsic: Check if the instruction is a bit reverse load
  1932: // intrinsic.
  1933: static bool isBrevLdIntrinsic(const Value *Inst) {
  1934:   unsigned ID = cast<IntrinsicInst>(Inst)->getIntrinsicID();
  1935:   return (ID == Intrinsic::hexagon_L2_loadrd_pbr ||
  1936:           ID == Intrinsic::hexagon_L2_loadri_pbr ||
  1937:           ID == Intrinsic::hexagon_L2_loadrh_pbr ||
  1938:           ID == Intrinsic::hexagon_L2_loadruh_pbr ||
  1939:           ID == Intrinsic::hexagon_L2_loadrb_pbr ||
  1940:           ID == Intrinsic::hexagon_L2_loadrub_pbr);
  1941: }
  1942: 
  1943: // Bit-reverse Load Intrinsic :Crawl up and figure out the object from previous
  1944: // instruction. So far we only handle bitcast, extract value and bit reverse
  1945: // load intrinsic instructions. Should we handle CGEP ?
  1946: static Value *getBrevLdObject(Value *V) {
  1947:   if (Operator::getOpcode(V) == Instruction::ExtractValue ||
  1948:       Operator::getOpcode(V) == Instruction::BitCast)
  1949:     V = cast<Operator>(V)->getOperand(0);
  1950:   else if (isa<IntrinsicInst>(V) && isBrevLdIntrinsic(V))
  1951:     V = cast<Instruction>(V)->getOperand(0);
  1952:   return V;
  1953: }
  1954: 
  1955: // Bit-reverse Load Intrinsic: For a PHI Node return either an incoming edge or
  1956: // a back edge. If the back edge comes from the intrinsic itself, the incoming
  1957: // edge is returned.
  1958: static Value *returnEdge(const PHINode *PN, Value *IntrBaseVal) {
  1959:   const BasicBlock *Parent = PN->getParent();
  1960:   int Idx = -1;
  1961:   for (unsigned i = 0, e = PN->getNumIncomingValues(); i < e; ++i) {
  1962:     BasicBlock *Blk = PN->getIncomingBlock(i);
  1963:     // Determine if the back edge is originated from intrinsic.
  1964:     if (Blk == Parent) {
  1965:       Value *BackEdgeVal = PN->getIncomingValue(i);
  1966:       Value *BaseVal;
  1967:       // Loop over till we return the same Value or we hit the IntrBaseVal.
  1968:       do {
  1969:         BaseVal = BackEdgeVal;
  1970:         BackEdgeVal = getBrevLdObject(BackEdgeVal);
  1971:       } while ((BaseVal != BackEdgeVal) && (IntrBaseVal != BackEdgeVal));
  1972:       // If the getBrevLdObject returns IntrBaseVal, we should return the
  1973:       // incoming edge.
  1974:       if (IntrBaseVal == BackEdgeVal)
  1975:         continue;
  1976:       Idx = i;
  1977:       break;
  1978:     } else // Set the node to incoming edge.
  1979:       Idx = i;
  1980:   }
  1981:   assert(Idx >= 0 && "Unexpected index to incoming argument in PHI");
  1982:   return PN->getIncomingValue(Idx);
  1983: }
  1984: 
  1985: // Bit-reverse Load Intrinsic: Figure out the underlying object the base
  1986: // pointer points to, for the bit-reverse load intrinsic. Setting this to
  1987: // memoperand might help alias analysis to figure out the dependencies.
  1988: static Value *getUnderLyingObjectForBrevLdIntr(Value *V) {
  1989:   Value *IntrBaseVal = V;
  1990:   Value *BaseVal;
  1991:   // Loop over till we return the same Value, implies we either figure out
  1992:   // the object or we hit a PHI
  1993:   do {
  1994:     BaseVal = V;
  1995:     V = getBrevLdObject(V);
  1996:   } while (BaseVal != V);
  1997: 
  1998:   // Identify the object from PHINode.
  1999:   if (const PHINode *PN = dyn_cast<PHINode>(V))
  2000:     return returnEdge(PN, IntrBaseVal);
```
- EN: It declares types such as DiagnosticInfoMisalignedTrap, which carry the state or API of this component. It declares or implements routines such as setOperationAction, setLoadExtAction, setTruncStoreAction, setIndexedLoadAction, ... (30 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明了 DiagnosticInfoMisalignedTrap 等类型，用来承载该组件的状态或接口。 这里声明或实现了 setOperationAction, setLoadExtAction, setTruncStoreAction, setIndexedLoadAction, ... (30 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 2001-2200 / 第 2001-2200 行

```cpp
  2001:   // For non PHI nodes, the object is the last value returned by getBrevLdObject
  2002:   else
  2003:     return V;
  2004: }
  2005: 
  2006: /// Given an intrinsic, checks if on the target the intrinsic will need to map
  2007: /// to a MemIntrinsicNode (touches memory). If this is the case, it stores
  2008: /// the intrinsic information into the Infos vector.
  2009: void HexagonTargetLowering::getTgtMemIntrinsic(
  2010:     SmallVectorImpl<IntrinsicInfo> &Infos, const CallBase &I,
  2011:     MachineFunction &MF, unsigned Intrinsic) const {
  2012:   IntrinsicInfo Info;
  2013:   switch (Intrinsic) {
  2014:   case Intrinsic::hexagon_L2_loadrd_pbr:
  2015:   case Intrinsic::hexagon_L2_loadri_pbr:
  2016:   case Intrinsic::hexagon_L2_loadrh_pbr:
  2017:   case Intrinsic::hexagon_L2_loadruh_pbr:
  2018:   case Intrinsic::hexagon_L2_loadrb_pbr:
  2019:   case Intrinsic::hexagon_L2_loadrub_pbr: {
  2020:     Info.opc = ISD::INTRINSIC_W_CHAIN;
  2021:     auto &DL = I.getDataLayout();
  2022:     auto &Cont = I.getCalledFunction()->getParent()->getContext();
  2023:     // The intrinsic function call is of the form { ElTy, i8* }
  2024:     // @llvm.hexagon.L2.loadXX.pbr(i8*, i32). The pointer and memory access type
  2025:     // should be derived from ElTy.
  2026:     Type *ElTy = I.getCalledFunction()->getReturnType()->getStructElementType(0);
  2027:     Info.memVT = MVT::getVT(ElTy);
  2028:     llvm::Value *BasePtrVal = I.getOperand(0);
  2029:     Info.ptrVal = getUnderLyingObjectForBrevLdIntr(BasePtrVal);
  2030:     // The offset value comes through Modifier register. For now, assume the
  2031:     // offset is 0.
  2032:     Info.offset = 0;
  2033:     Info.align = DL.getABITypeAlign(Info.memVT.getTypeForEVT(Cont));
  2034:     Info.flags = MachineMemOperand::MOLoad;
  2035:     Infos.push_back(Info);
  2036:     return;
  2037:   }
  2038:   case Intrinsic::hexagon_V6_vgathermw:
  2039:   case Intrinsic::hexagon_V6_vgathermw_128B:
  2040:   case Intrinsic::hexagon_V6_vgathermh:
  2041:   case Intrinsic::hexagon_V6_vgathermh_128B:
  2042:   case Intrinsic::hexagon_V6_vgathermhw:
  2043:   case Intrinsic::hexagon_V6_vgathermhw_128B:
  2044:   case Intrinsic::hexagon_V6_vgathermwq:
  2045:   case Intrinsic::hexagon_V6_vgathermwq_128B:
  2046:   case Intrinsic::hexagon_V6_vgathermhq:
  2047:   case Intrinsic::hexagon_V6_vgathermhq_128B:
  2048:   case Intrinsic::hexagon_V6_vgathermhwq:
  2049:   case Intrinsic::hexagon_V6_vgathermhwq_128B:
  2050:   case Intrinsic::hexagon_V6_vgather_vscattermh:
  2051:   case Intrinsic::hexagon_V6_vgather_vscattermh_128B: {
  2052:     const Module &M = *I.getParent()->getParent()->getParent();
  2053:     Info.opc = ISD::INTRINSIC_W_CHAIN;
  2054:     Type *VecTy = I.getArgOperand(I.arg_size() - 1)->getType();
  2055:     assert(VecTy->isVectorTy() && "Expected vector operand for vgather");
  2056:     Info.memVT = MVT::getVT(VecTy);
  2057:     Info.ptrVal = I.getArgOperand(0);
  2058:     Info.offset = 0;
  2059:     Info.align =
  2060:         MaybeAlign(M.getDataLayout().getTypeAllocSizeInBits(VecTy) / 8);
  2061:     Info.flags = MachineMemOperand::MOLoad | MachineMemOperand::MOStore |
  2062:                  MachineMemOperand::MOVolatile;
  2063:     Infos.push_back(Info);
  2064:     return;
  2065:   }
  2066:   default:
  2067:     break;
  2068:   }
  2069: }
  2070: 
  2071: bool HexagonTargetLowering::hasBitTest(SDValue X, SDValue Y) const {
  2072:   return X.getValueType().isScalarInteger(); // 'tstbit'
  2073: }
  2074: 
  2075: bool HexagonTargetLowering::isTruncateFree(Type *Ty1, Type *Ty2) const {
  2076:   return isTruncateFree(EVT::getEVT(Ty1), EVT::getEVT(Ty2));
  2077: }
  2078: 
  2079: bool HexagonTargetLowering::isTruncateFree(EVT VT1, EVT VT2) const {
  2080:   if (!VT1.isSimple() || !VT2.isSimple())
  2081:     return false;
  2082:   return VT1.getSimpleVT() == MVT::i64 && VT2.getSimpleVT() == MVT::i32;
  2083: }
  2084: 
  2085: bool HexagonTargetLowering::isFMAFasterThanFMulAndFAdd(
  2086:     const MachineFunction &MF, EVT VT) const {
  2087:   return isOperationLegalOrCustom(ISD::FMA, VT);
  2088: }
  2089: 
  2090: // Should we expand the build vector with shuffles?
  2091: bool HexagonTargetLowering::shouldExpandBuildVectorWithShuffles(EVT VT,
  2092:       unsigned DefinedValues) const {
  2093:   return false;
  2094: }
  2095: 
  2096: bool HexagonTargetLowering::isExtractSubvectorCheap(EVT ResVT, EVT SrcVT,
  2097:       unsigned Index) const {
  2098:   assert(ResVT.getVectorElementType() == SrcVT.getVectorElementType());
  2099:   if (!ResVT.isSimple() || !SrcVT.isSimple())
  2100:     return false;
  2101: 
  2102:   MVT ResTy = ResVT.getSimpleVT(), SrcTy = SrcVT.getSimpleVT();
  2103:   if (ResTy.getVectorElementType() != MVT::i1)
  2104:     return true;
  2105: 
  2106:   // Non-HVX bool vectors are relatively cheap.
  2107:   return SrcTy.getVectorNumElements() <= 8;
  2108: }
  2109: 
  2110: bool HexagonTargetLowering::isTargetCanonicalConstantNode(SDValue Op) const {
  2111:   return Op.getOpcode() == ISD::CONCAT_VECTORS ||
  2112:          TargetLowering::isTargetCanonicalConstantNode(Op);
  2113: }
  2114: 
  2115: bool HexagonTargetLowering::isShuffleMaskLegal(ArrayRef<int> Mask,
  2116:                                                EVT VT) const {
  2117:   return true;
  2118: }
  2119: 
  2120: TargetLoweringBase::LegalizeTypeAction
  2121: HexagonTargetLowering::getPreferredVectorAction(MVT VT) const {
  2122:   unsigned VecLen = VT.getVectorMinNumElements();
  2123:   MVT ElemTy = VT.getVectorElementType();
  2124: 
  2125:   if (VecLen == 1 || VT.isScalableVector())
  2126:     return TargetLoweringBase::TypeScalarizeVector;
  2127: 
  2128:   if (Subtarget.useHVXOps()) {
  2129:     unsigned Action = getPreferredHvxVectorAction(VT);
  2130:     if (Action != ~0u)
  2131:       return static_cast<TargetLoweringBase::LegalizeTypeAction>(Action);
  2132:   }
  2133: 
  2134:   // Always widen (remaining) vectors of i1.
  2135:   if (ElemTy == MVT::i1)
  2136:     return TargetLoweringBase::TypeWidenVector;
  2137:   // Widen non-power-of-2 vectors. Such types cannot be split right now,
  2138:   // and computeRegisterProperties will override "split" with "widen",
  2139:   // which can cause other issues.
  2140:   if (!isPowerOf2_32(VecLen))
  2141:     return TargetLoweringBase::TypeWidenVector;
  2142: 
  2143:   return TargetLoweringBase::TypeSplitVector;
  2144: }
  2145: 
  2146: TargetLoweringBase::LegalizeAction
  2147: HexagonTargetLowering::getCustomOperationAction(SDNode &Op) const {
  2148:   if (Subtarget.useHVXOps()) {
  2149:     unsigned Action = getCustomHvxOperationAction(Op);
  2150:     if (Action != ~0u)
  2151:       return static_cast<TargetLoweringBase::LegalizeAction>(Action);
  2152:   }
  2153:   return TargetLoweringBase::Legal;
  2154: }
  2155: 
  2156: std::pair<SDValue, int>
  2157: HexagonTargetLowering::getBaseAndOffset(SDValue Addr) const {
  2158:   if (Addr.getOpcode() == ISD::ADD) {
  2159:     SDValue Op1 = Addr.getOperand(1);
  2160:     if (auto *CN = dyn_cast<const ConstantSDNode>(Op1.getNode()))
  2161:       return { Addr.getOperand(0), CN->getSExtValue() };
  2162:   }
  2163:   return { Addr, 0 };
  2164: }
  2165: 
  2166: // Lower a vector shuffle (V1, V2, V3).  V1 and V2 are the two vectors
  2167: // to select data from, V3 is the permutation.
  2168: SDValue
  2169: HexagonTargetLowering::LowerVECTOR_SHUFFLE(SDValue Op, SelectionDAG &DAG)
  2170:       const {
  2171:   const auto *SVN = cast<ShuffleVectorSDNode>(Op);
  2172:   ArrayRef<int> AM = SVN->getMask();
  2173:   assert(AM.size() <= 8 && "Unexpected shuffle mask");
  2174:   unsigned VecLen = AM.size();
  2175: 
  2176:   MVT VecTy = ty(Op);
  2177:   assert(!Subtarget.isHVXVectorType(VecTy, true) &&
  2178:          "HVX shuffles should be legal");
  2179:   assert(VecTy.getSizeInBits() <= 64 && "Unexpected vector length");
  2180: 
  2181:   SDValue Op0 = Op.getOperand(0);
  2182:   SDValue Op1 = Op.getOperand(1);
  2183:   const SDLoc &dl(Op);
  2184: 
  2185:   // If the inputs are not the same as the output, bail. This is not an
  2186:   // error situation, but complicates the handling and the default expansion
  2187:   // (into BUILD_VECTOR) should be adequate.
  2188:   if (ty(Op0) != VecTy || ty(Op1) != VecTy)
  2189:     return SDValue();
  2190: 
  2191:   // Normalize the mask so that the first non-negative index comes from
  2192:   // the first operand.
  2193:   SmallVector<int, 8> Mask(AM);
  2194:   unsigned F = llvm::find_if(AM, [](int M) { return M >= 0; }) - AM.data();
  2195:   if (F == AM.size())
  2196:     return DAG.getUNDEF(VecTy);
  2197:   if (AM[F] >= int(VecLen)) {
  2198:     ShuffleVectorSDNode::commuteMask(Mask);
  2199:     std::swap(Op0, Op1);
  2200:   }
```
- EN: It declares or implements routines such as MemIntrinsicNode, getDataLayout, getCalledFunction, pbr, ... (43 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonTargetLowering, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 MemIntrinsicNode, getDataLayout, getCalledFunction, pbr, ... (43 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonTargetLowering，说明了它与同级后端组件的连接关系。

### Lines 2201-2400 / 第 2201-2400 行

```cpp
  2201: 
  2202:   // Express the shuffle mask in terms of bytes.
  2203:   SmallVector<int,8> ByteMask;
  2204:   unsigned ElemBytes = VecTy.getVectorElementType().getSizeInBits() / 8;
  2205:   for (int M : Mask) {
  2206:     if (M < 0) {
  2207:       for (unsigned j = 0; j != ElemBytes; ++j)
  2208:         ByteMask.push_back(-1);
  2209:     } else {
  2210:       for (unsigned j = 0; j != ElemBytes; ++j)
  2211:         ByteMask.push_back(M*ElemBytes + j);
  2212:     }
  2213:   }
  2214:   assert(ByteMask.size() <= 8);
  2215: 
  2216:   // All non-undef (non-negative) indexes are well within [0..127], so they
  2217:   // fit in a single byte. Build two 64-bit words:
  2218:   // - MaskIdx where each byte is the corresponding index (for non-negative
  2219:   //   indexes), and 0xFF for negative indexes, and
  2220:   // - MaskUnd that has 0xFF for each negative index.
  2221:   uint64_t MaskIdx = 0;
  2222:   uint64_t MaskUnd = 0;
  2223:   for (unsigned i = 0, e = ByteMask.size(); i != e; ++i) {
  2224:     unsigned S = 8*i;
  2225:     uint64_t M = ByteMask[i] & 0xFF;
  2226:     if (M == 0xFF)
  2227:       MaskUnd |= M << S;
  2228:     MaskIdx |= M << S;
  2229:   }
  2230: 
  2231:   if (ByteMask.size() == 4) {
  2232:     // Identity.
  2233:     if (MaskIdx == (0x03020100 | MaskUnd))
  2234:       return Op0;
  2235:     // Byte swap.
  2236:     if (MaskIdx == (0x00010203 | MaskUnd)) {
  2237:       SDValue T0 = DAG.getBitcast(MVT::i32, Op0);
  2238:       SDValue T1 = DAG.getNode(ISD::BSWAP, dl, MVT::i32, T0);
  2239:       return DAG.getBitcast(VecTy, T1);
  2240:     }
  2241: 
  2242:     // Byte packs.
  2243:     SDValue Concat10 =
  2244:         getCombine(Op1, Op0, dl, typeJoin({ty(Op1), ty(Op0)}), DAG);
  2245:     if (MaskIdx == (0x06040200 | MaskUnd))
  2246:       return getInstr(Hexagon::S2_vtrunehb, dl, VecTy, {Concat10}, DAG);
  2247:     if (MaskIdx == (0x07050301 | MaskUnd))
  2248:       return getInstr(Hexagon::S2_vtrunohb, dl, VecTy, {Concat10}, DAG);
  2249: 
  2250:     SDValue Concat01 =
  2251:         getCombine(Op0, Op1, dl, typeJoin({ty(Op0), ty(Op1)}), DAG);
  2252:     if (MaskIdx == (0x02000604 | MaskUnd))
  2253:       return getInstr(Hexagon::S2_vtrunehb, dl, VecTy, {Concat01}, DAG);
  2254:     if (MaskIdx == (0x03010705 | MaskUnd))
  2255:       return getInstr(Hexagon::S2_vtrunohb, dl, VecTy, {Concat01}, DAG);
  2256:   }
  2257: 
  2258:   if (ByteMask.size() == 8) {
  2259:     // Identity.
  2260:     if (MaskIdx == (0x0706050403020100ull | MaskUnd))
  2261:       return Op0;
  2262:     // Byte swap.
  2263:     if (MaskIdx == (0x0001020304050607ull | MaskUnd)) {
  2264:       SDValue T0 = DAG.getBitcast(MVT::i64, Op0);
  2265:       SDValue T1 = DAG.getNode(ISD::BSWAP, dl, MVT::i64, T0);
  2266:       return DAG.getBitcast(VecTy, T1);
  2267:     }
  2268: 
  2269:     // Halfword picks.
  2270:     if (MaskIdx == (0x0d0c050409080100ull | MaskUnd))
  2271:       return getInstr(Hexagon::S2_shuffeh, dl, VecTy, {Op1, Op0}, DAG);
  2272:     if (MaskIdx == (0x0f0e07060b0a0302ull | MaskUnd))
  2273:       return getInstr(Hexagon::S2_shuffoh, dl, VecTy, {Op1, Op0}, DAG);
  2274:     if (MaskIdx == (0x0d0c090805040100ull | MaskUnd))
  2275:       return getInstr(Hexagon::S2_vtrunewh, dl, VecTy, {Op1, Op0}, DAG);
  2276:     if (MaskIdx == (0x0f0e0b0a07060302ull | MaskUnd))
  2277:       return getInstr(Hexagon::S2_vtrunowh, dl, VecTy, {Op1, Op0}, DAG);
  2278:     if (MaskIdx == (0x0706030205040100ull | MaskUnd)) {
  2279:       VectorPair P = opSplit(Op0, dl, DAG);
  2280:       return getInstr(Hexagon::S2_packhl, dl, VecTy, {P.second, P.first}, DAG);
  2281:     }
  2282: 
  2283:     // Byte packs.
  2284:     if (MaskIdx == (0x0e060c040a020800ull | MaskUnd))
  2285:       return getInstr(Hexagon::S2_shuffeb, dl, VecTy, {Op1, Op0}, DAG);
  2286:     if (MaskIdx == (0x0f070d050b030901ull | MaskUnd))
  2287:       return getInstr(Hexagon::S2_shuffob, dl, VecTy, {Op1, Op0}, DAG);
  2288:   }
  2289: 
  2290:   return SDValue();
  2291: }
  2292: 
  2293: SDValue
  2294: HexagonTargetLowering::getSplatValue(SDValue Op, SelectionDAG &DAG) const {
  2295:   switch (Op.getOpcode()) {
  2296:     case ISD::BUILD_VECTOR:
  2297:       if (SDValue S = cast<BuildVectorSDNode>(Op)->getSplatValue())
  2298:         return S;
  2299:       break;
  2300:     case ISD::SPLAT_VECTOR:
  2301:       return Op.getOperand(0);
  2302:   }
  2303:   return SDValue();
  2304: }
  2305: 
  2306: // Create a Hexagon-specific node for shifting a vector by an integer.
  2307: SDValue
  2308: HexagonTargetLowering::getVectorShiftByInt(SDValue Op, SelectionDAG &DAG)
  2309:       const {
  2310:   unsigned NewOpc;
  2311:   switch (Op.getOpcode()) {
  2312:     case ISD::SHL:
  2313:       NewOpc = HexagonISD::VASL;
  2314:       break;
  2315:     case ISD::SRA:
  2316:       NewOpc = HexagonISD::VASR;
  2317:       break;
  2318:     case ISD::SRL:
  2319:       NewOpc = HexagonISD::VLSR;
  2320:       break;
  2321:     default:
  2322:       llvm_unreachable("Unexpected shift opcode");
  2323:   }
  2324:   if (SDValue Sp = getSplatValue(Op.getOperand(1), DAG)) {
  2325:     const SDLoc dl(Op);
  2326:     // Canonicalize shift amount to i32 as required.
  2327:     SDValue Sh = Sp;
  2328:     if (Sh.getValueType() != MVT::i32)
  2329:       Sh = DAG.getZExtOrTrunc(Sh, dl, MVT::i32);
  2330: 
  2331:     assert(Sh.getValueType() == MVT::i32 &&
  2332:            "Hexagon vector shift-by-int must use i32 shift operand");
  2333:     return DAG.getNode(NewOpc, dl, ty(Op), Op.getOperand(0), Sh);
  2334:   }
  2335: 
  2336:   return SDValue();
  2337: }
  2338: 
  2339: SDValue
  2340: HexagonTargetLowering::LowerVECTOR_SHIFT(SDValue Op, SelectionDAG &DAG) const {
  2341:   const SDLoc &dl(Op);
  2342: 
  2343:   // First try to convert the shift (by vector) to a shift by a scalar.
  2344:   // If we first split the shift, the shift amount will become 'extract
  2345:   // subvector', and will no longer be recognized as scalar.
  2346:   SDValue Res = Op;
  2347:   if (SDValue S = getVectorShiftByInt(Op, DAG))
  2348:     Res = S;
  2349: 
  2350:   unsigned Opc = Res.getOpcode();
  2351:   switch (Opc) {
  2352:   case HexagonISD::VASR:
  2353:   case HexagonISD::VLSR:
  2354:   case HexagonISD::VASL:
  2355:     break;
  2356:   default:
  2357:     // No instructions for shifts by non-scalars.
  2358:     return SDValue();
  2359:   }
  2360: 
  2361:   MVT ResTy = ty(Res);
  2362:   if (ResTy.getVectorElementType() != MVT::i8)
  2363:     return Res;
  2364: 
  2365:   // For shifts of i8, extend the inputs to i16, then truncate back to i8.
  2366:   assert(ResTy.getVectorElementType() == MVT::i8);
  2367:   SDValue Val = Res.getOperand(0), Amt = Res.getOperand(1);
  2368: 
  2369:   auto ShiftPartI8 = [&dl, &DAG, this](unsigned Opc, SDValue V, SDValue A) {
  2370:     MVT Ty = ty(V);
  2371:     MVT ExtTy = MVT::getVectorVT(MVT::i16, Ty.getVectorNumElements());
  2372:     SDValue ExtV = Opc == HexagonISD::VASR ? DAG.getSExtOrTrunc(V, dl, ExtTy)
  2373:                                            : DAG.getZExtOrTrunc(V, dl, ExtTy);
  2374:     SDValue ExtS = DAG.getNode(Opc, dl, ExtTy, {ExtV, A});
  2375:     return DAG.getZExtOrTrunc(ExtS, dl, Ty);
  2376:   };
  2377: 
  2378:   if (ResTy.getSizeInBits() == 32)
  2379:     return ShiftPartI8(Opc, Val, Amt);
  2380: 
  2381:   auto [LoV, HiV] = opSplit(Val, dl, DAG);
  2382:   return DAG.getNode(ISD::CONCAT_VECTORS, dl, ResTy,
  2383:                      {ShiftPartI8(Opc, LoV, Amt), ShiftPartI8(Opc, HiV, Amt)});
  2384: }
  2385: 
  2386: SDValue
  2387: HexagonTargetLowering::LowerROTL(SDValue Op, SelectionDAG &DAG) const {
  2388:   if (isa<ConstantSDNode>(Op.getOperand(1).getNode()))
  2389:     return Op;
  2390:   return SDValue();
  2391: }
  2392: 
  2393: SDValue
  2394: HexagonTargetLowering::LowerBITCAST(SDValue Op, SelectionDAG &DAG) const {
  2395:   MVT ResTy = ty(Op);
  2396:   SDValue InpV = Op.getOperand(0);
  2397:   MVT InpTy = ty(InpV);
  2398:   assert(ResTy.getSizeInBits() == InpTy.getSizeInBits());
  2399:   const SDLoc &dl(Op);
  2400: 
```
- EN: It declares or implements routines such as push_back, assert, getBitcast, getNode, ... (19 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonTargetLowering, HexagonISD, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 push_back, assert, getBitcast, getNode, ... (19 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonTargetLowering, HexagonISD，说明了它与同级后端组件的连接关系。

### Lines 2401-2600 / 第 2401-2600 行

```cpp
  2401:   // Handle conversion from i8 to v8i1.
  2402:   if (InpTy == MVT::i8) {
  2403:     if (ResTy == MVT::v8i1) {
  2404:       SDValue Sc = DAG.getBitcast(tyScalar(InpTy), InpV);
  2405:       SDValue Ext = DAG.getZExtOrTrunc(Sc, dl, MVT::i32);
  2406:       return getInstr(Hexagon::C2_tfrrp, dl, ResTy, Ext, DAG);
  2407:     }
  2408:     return SDValue();
  2409:   }
  2410: 
  2411:   return Op;
  2412: }
  2413: 
  2414: bool
  2415: HexagonTargetLowering::getBuildVectorConstInts(ArrayRef<SDValue> Values,
  2416:       MVT VecTy, SelectionDAG &DAG,
  2417:       MutableArrayRef<ConstantInt*> Consts) const {
  2418:   MVT ElemTy = VecTy.getVectorElementType();
  2419:   unsigned ElemWidth = ElemTy.getSizeInBits();
  2420:   IntegerType *IntTy = IntegerType::get(*DAG.getContext(), ElemWidth);
  2421:   bool AllConst = true;
  2422: 
  2423:   for (unsigned i = 0, e = Values.size(); i != e; ++i) {
  2424:     SDValue V = Values[i];
  2425:     if (V.isUndef()) {
  2426:       Consts[i] = ConstantInt::get(IntTy, 0);
  2427:       continue;
  2428:     }
  2429:     // Make sure to always cast to IntTy.
  2430:     if (auto *CN = dyn_cast<ConstantSDNode>(V.getNode())) {
  2431:       const ConstantInt *CI = CN->getConstantIntValue();
  2432:       Consts[i] = cast<ConstantInt>(
  2433:           ConstantInt::get(IntTy, CI->getValue().trunc(ElemWidth)));
  2434:     } else if (auto *CN = dyn_cast<ConstantFPSDNode>(V.getNode())) {
  2435:       const ConstantFP *CF = CN->getConstantFPValue();
  2436:       APInt A = CF->getValueAPF().bitcastToAPInt();
  2437:       Consts[i] = ConstantInt::get(IntTy, A.getZExtValue());
  2438:     } else {
  2439:       AllConst = false;
  2440:     }
  2441:   }
  2442:   return AllConst;
  2443: }
  2444: 
  2445: SDValue
  2446: HexagonTargetLowering::buildVector32(ArrayRef<SDValue> Elem, const SDLoc &dl,
  2447:                                      MVT VecTy, SelectionDAG &DAG) const {
  2448:   MVT ElemTy = VecTy.getVectorElementType();
  2449:   assert(VecTy.getVectorNumElements() == Elem.size());
  2450: 
  2451:   SmallVector<ConstantInt*,4> Consts(Elem.size());
  2452:   bool AllConst = getBuildVectorConstInts(Elem, VecTy, DAG, Consts);
  2453: 
  2454:   unsigned First, Num = Elem.size();
  2455:   for (First = 0; First != Num; ++First) {
  2456:     if (!isUndef(Elem[First]))
  2457:       break;
  2458:   }
  2459:   if (First == Num)
  2460:     return DAG.getUNDEF(VecTy);
  2461: 
  2462:   if (AllConst &&
  2463:       llvm::all_of(Consts, [](ConstantInt *CI) { return CI->isZero(); }))
  2464:     return getZero(dl, VecTy, DAG);
  2465: 
  2466:   if (ElemTy == MVT::i16 || ElemTy == MVT::f16) {
  2467:     assert(Elem.size() == 2);
  2468:     if (AllConst) {
  2469:       // The 'Consts' array will have all values as integers regardless
  2470:       // of the vector element type.
  2471:       uint32_t V = (Consts[0]->getZExtValue() & 0xFFFF) |
  2472:                    Consts[1]->getZExtValue() << 16;
  2473:       return DAG.getBitcast(VecTy, DAG.getConstant(V, dl, MVT::i32));
  2474:     }
  2475:     SDValue E0, E1;
  2476:     if (ElemTy == MVT::f16) {
  2477:       E0 = DAG.getZExtOrTrunc(DAG.getBitcast(MVT::i16, Elem[0]), dl, MVT::i32);
  2478:       E1 = DAG.getZExtOrTrunc(DAG.getBitcast(MVT::i16, Elem[1]), dl, MVT::i32);
  2479:     } else {
  2480:       E0 = Elem[0];
  2481:       E1 = Elem[1];
  2482:     }
  2483:     SDValue N = getInstr(Hexagon::A2_combine_ll, dl, MVT::i32, {E1, E0}, DAG);
  2484:     return DAG.getBitcast(VecTy, N);
  2485:   }
  2486: 
  2487:   if (ElemTy == MVT::i8) {
  2488:     // First try generating a constant.
  2489:     if (AllConst) {
  2490:       uint32_t V = (Consts[0]->getZExtValue() & 0xFF) |
  2491:                    (Consts[1]->getZExtValue() & 0xFF) << 8 |
  2492:                    (Consts[2]->getZExtValue() & 0xFF) << 16 |
  2493:                    Consts[3]->getZExtValue() << 24;
  2494:       return DAG.getBitcast(MVT::v4i8, DAG.getConstant(V, dl, MVT::i32));
  2495:     }
  2496: 
  2497:     // Then try splat.
  2498:     bool IsSplat = true;
  2499:     for (unsigned i = First+1; i != Num; ++i) {
  2500:       if (Elem[i] == Elem[First] || isUndef(Elem[i]))
  2501:         continue;
  2502:       IsSplat = false;
  2503:       break;
  2504:     }
  2505:     if (IsSplat) {
  2506:       // Legalize the operand of SPLAT_VECTOR.
  2507:       SDValue Ext = DAG.getZExtOrTrunc(Elem[First], dl, MVT::i32);
  2508:       return DAG.getNode(ISD::SPLAT_VECTOR, dl, VecTy, Ext);
  2509:     }
  2510: 
  2511:     // Generate
  2512:     //   (zxtb(Elem[0]) | (zxtb(Elem[1]) << 8)) |
  2513:     //   (zxtb(Elem[2]) | (zxtb(Elem[3]) << 8)) << 16
  2514:     assert(Elem.size() == 4);
  2515:     SDValue Vs[4];
  2516:     for (unsigned i = 0; i != 4; ++i) {
  2517:       Vs[i] = DAG.getZExtOrTrunc(Elem[i], dl, MVT::i32);
  2518:       Vs[i] = DAG.getZeroExtendInReg(Vs[i], dl, MVT::i8);
  2519:     }
  2520:     SDValue S8 = DAG.getConstant(8, dl, MVT::i32);
  2521:     SDValue T0 = DAG.getNode(ISD::SHL, dl, MVT::i32, {Vs[1], S8});
  2522:     SDValue T1 = DAG.getNode(ISD::SHL, dl, MVT::i32, {Vs[3], S8});
  2523:     SDValue B0 = DAG.getNode(ISD::OR, dl, MVT::i32, {Vs[0], T0});
  2524:     SDValue B1 = DAG.getNode(ISD::OR, dl, MVT::i32, {Vs[2], T1});
  2525: 
  2526:     SDValue R = getInstr(Hexagon::A2_combine_ll, dl, MVT::i32, {B1, B0}, DAG);
  2527:     return DAG.getBitcast(MVT::v4i8, R);
  2528:   }
  2529: 
  2530: #ifndef NDEBUG
  2531:   dbgs() << "VecTy: " << VecTy << '\n';
  2532: #endif
  2533:   llvm_unreachable("Unexpected vector element type");
  2534: }
  2535: 
  2536: SDValue
  2537: HexagonTargetLowering::buildVector64(ArrayRef<SDValue> Elem, const SDLoc &dl,
  2538:                                      MVT VecTy, SelectionDAG &DAG) const {
  2539:   MVT ElemTy = VecTy.getVectorElementType();
  2540:   assert(VecTy.getVectorNumElements() == Elem.size());
  2541: 
  2542:   SmallVector<ConstantInt*,8> Consts(Elem.size());
  2543:   bool AllConst = getBuildVectorConstInts(Elem, VecTy, DAG, Consts);
  2544: 
  2545:   unsigned First, Num = Elem.size();
  2546:   for (First = 0; First != Num; ++First) {
  2547:     if (!isUndef(Elem[First]))
  2548:       break;
  2549:   }
  2550:   if (First == Num)
  2551:     return DAG.getUNDEF(VecTy);
  2552: 
  2553:   if (AllConst &&
  2554:       llvm::all_of(Consts, [](ConstantInt *CI) { return CI->isZero(); }))
  2555:     return getZero(dl, VecTy, DAG);
  2556: 
  2557:   // First try splat if possible.
  2558:   if (ElemTy == MVT::i16 || ElemTy == MVT::f16) {
  2559:     bool IsSplat = true;
  2560:     for (unsigned i = First+1; i != Num; ++i) {
  2561:       if (Elem[i] == Elem[First] || isUndef(Elem[i]))
  2562:         continue;
  2563:       IsSplat = false;
  2564:       break;
  2565:     }
  2566:     if (IsSplat) {
  2567:       // Legalize the operand of SPLAT_VECTOR
  2568:       SDValue S = ElemTy == MVT::f16 ? DAG.getBitcast(MVT::i16, Elem[First])
  2569:                                      : Elem[First];
  2570:       SDValue Ext = DAG.getZExtOrTrunc(S, dl, MVT::i32);
  2571:       return DAG.getNode(ISD::SPLAT_VECTOR, dl, VecTy, Ext);
  2572:     }
  2573:   }
  2574: 
  2575:   // Then try constant.
  2576:   if (AllConst) {
  2577:     uint64_t Val = 0;
  2578:     unsigned W = ElemTy.getSizeInBits();
  2579:     uint64_t Mask = (1ull << W) - 1;
  2580:     for (unsigned i = 0; i != Num; ++i)
  2581:       Val = (Val << W) | (Consts[Num-1-i]->getZExtValue() & Mask);
  2582:     SDValue V0 = DAG.getConstant(Val, dl, MVT::i64);
  2583:     return DAG.getBitcast(VecTy, V0);
  2584:   }
  2585: 
  2586:   // Build two 32-bit vectors and concatenate.
  2587:   MVT HalfTy = MVT::getVectorVT(ElemTy, Num/2);
  2588:   SDValue L = (ElemTy == MVT::i32)
  2589:                 ? Elem[0]
  2590:                 : buildVector32(Elem.take_front(Num/2), dl, HalfTy, DAG);
  2591:   SDValue H = (ElemTy == MVT::i32)
  2592:                 ? Elem[1]
  2593:                 : buildVector32(Elem.drop_front(Num/2), dl, HalfTy, DAG);
  2594:   return getCombine(H, L, dl, VecTy, DAG);
  2595: }
  2596: 
  2597: SDValue
  2598: HexagonTargetLowering::extractVector(SDValue VecV, SDValue IdxV,
  2599:                                      const SDLoc &dl, MVT ValTy, MVT ResTy,
  2600:                                      SelectionDAG &DAG) const {
```
- EN: Header guards in this range prevent duplicate inclusion and define the interface boundary. It declares or implements routines such as getBitcast, getZExtOrTrunc, getInstr, SDValue, ... (31 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里声明或实现了 getBitcast, getZExtOrTrunc, getInstr, SDValue, ... (31 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 2601-2800 / 第 2601-2800 行

```cpp
  2601:   MVT VecTy = ty(VecV);
  2602:   assert(!ValTy.isVector() ||
  2603:          VecTy.getVectorElementType() == ValTy.getVectorElementType());
  2604:   if (VecTy.getVectorElementType() == MVT::i1)
  2605:     return extractVectorPred(VecV, IdxV, dl, ValTy, ResTy, DAG);
  2606: 
  2607:   unsigned VecWidth = VecTy.getSizeInBits();
  2608:   unsigned ValWidth = ValTy.getSizeInBits();
  2609:   unsigned ElemWidth = VecTy.getVectorElementType().getSizeInBits();
  2610:   assert((VecWidth % ElemWidth) == 0);
  2611:   assert(VecWidth == 32 || VecWidth == 64);
  2612: 
  2613:   // Cast everything to scalar integer types.
  2614:   MVT ScalarTy = tyScalar(VecTy);
  2615:   VecV = DAG.getBitcast(ScalarTy, VecV);
  2616: 
  2617:   SDValue WidthV = DAG.getConstant(ValWidth, dl, MVT::i32);
  2618:   SDValue ExtV;
  2619: 
  2620:   if (auto *IdxN = dyn_cast<ConstantSDNode>(IdxV)) {
  2621:     unsigned Off = IdxN->getZExtValue() * ElemWidth;
  2622:     if (VecWidth == 64 && ValWidth == 32) {
  2623:       assert(Off == 0 || Off == 32);
  2624:       ExtV = Off == 0 ? LoHalf(VecV, DAG) : HiHalf(VecV, DAG);
  2625:     } else if (Off == 0 && (ValWidth % 8) == 0) {
  2626:       ExtV = DAG.getZeroExtendInReg(VecV, dl, tyScalar(ValTy));
  2627:     } else {
  2628:       SDValue OffV = DAG.getConstant(Off, dl, MVT::i32);
  2629:       // The return type of EXTRACTU must be the same as the type of the
  2630:       // input vector.
  2631:       ExtV = DAG.getNode(HexagonISD::EXTRACTU, dl, ScalarTy,
  2632:                          {VecV, WidthV, OffV});
  2633:     }
  2634:   } else {
  2635:     if (ty(IdxV) != MVT::i32)
  2636:       IdxV = DAG.getZExtOrTrunc(IdxV, dl, MVT::i32);
  2637:     SDValue OffV = DAG.getNode(ISD::MUL, dl, MVT::i32, IdxV,
  2638:                                DAG.getConstant(ElemWidth, dl, MVT::i32));
  2639:     ExtV = DAG.getNode(HexagonISD::EXTRACTU, dl, ScalarTy,
  2640:                        {VecV, WidthV, OffV});
  2641:   }
  2642: 
  2643:   // Cast ExtV to the requested result type.
  2644:   ExtV = DAG.getZExtOrTrunc(ExtV, dl, tyScalar(ResTy));
  2645:   ExtV = DAG.getBitcast(ResTy, ExtV);
  2646:   return ExtV;
  2647: }
  2648: 
  2649: SDValue
  2650: HexagonTargetLowering::extractVectorPred(SDValue VecV, SDValue IdxV,
  2651:                                          const SDLoc &dl, MVT ValTy, MVT ResTy,
  2652:                                          SelectionDAG &DAG) const {
  2653:   // Special case for v{8,4,2}i1 (the only boolean vectors legal in Hexagon
  2654:   // without any coprocessors).
  2655:   MVT VecTy = ty(VecV);
  2656:   unsigned VecWidth = VecTy.getSizeInBits();
  2657:   unsigned ValWidth = ValTy.getSizeInBits();
  2658:   assert(VecWidth == VecTy.getVectorNumElements() &&
  2659:          "Vector elements should equal vector width size");
  2660:   assert(VecWidth == 8 || VecWidth == 4 || VecWidth == 2);
  2661: 
  2662:   // Check if this is an extract of the lowest bit.
  2663:   if (isNullConstant(IdxV) && ValTy.getSizeInBits() == 1) {
  2664:     // Extracting the lowest bit is a no-op, but it changes the type,
  2665:     // so it must be kept as an operation to avoid errors related to
  2666:     // type mismatches.
  2667:     return DAG.getNode(HexagonISD::TYPECAST, dl, MVT::i1, VecV);
  2668:   }
  2669: 
  2670:   // If the value extracted is a single bit, use tstbit.
  2671:   if (ValWidth == 1) {
  2672:     SDValue A0 = getInstr(Hexagon::C2_tfrpr, dl, MVT::i32, {VecV}, DAG);
  2673:     SDValue M0 = DAG.getConstant(8 / VecWidth, dl, MVT::i32);
  2674:     SDValue I0 = DAG.getNode(ISD::MUL, dl, MVT::i32, IdxV, M0);
  2675:     return DAG.getNode(HexagonISD::TSTBIT, dl, MVT::i1, A0, I0);
  2676:   }
  2677: 
  2678:   // Each bool vector (v2i1, v4i1, v8i1) always occupies 8 bits in
  2679:   // a predicate register. The elements of the vector are repeated
  2680:   // in the register (if necessary) so that the total number is 8.
  2681:   // The extracted subvector will need to be expanded in such a way.
  2682:   unsigned Scale = VecWidth / ValWidth;
  2683: 
  2684:   // Generate (p2d VecV) >> 8*Idx to move the interesting bytes to
  2685:   // position 0.
  2686:   assert(ty(IdxV) == MVT::i32);
  2687:   unsigned VecRep = 8 / VecWidth;
  2688:   SDValue S0 = DAG.getNode(ISD::MUL, dl, MVT::i32, IdxV,
  2689:                            DAG.getConstant(8*VecRep, dl, MVT::i32));
  2690:   SDValue T0 = DAG.getNode(HexagonISD::P2D, dl, MVT::i64, VecV);
  2691:   SDValue T1 = DAG.getNode(ISD::SRL, dl, MVT::i64, T0, S0);
  2692:   while (Scale > 1) {
  2693:     // The longest possible subvector is at most 32 bits, so it is always
  2694:     // contained in the low subregister.
  2695:     T1 = LoHalf(T1, DAG);
  2696:     T1 = expandPredicate(T1, dl, DAG);
  2697:     Scale /= 2;
  2698:   }
  2699: 
  2700:   return DAG.getNode(HexagonISD::D2P, dl, ResTy, T1);
  2701: }
  2702: 
  2703: SDValue
  2704: HexagonTargetLowering::insertVector(SDValue VecV, SDValue ValV, SDValue IdxV,
  2705:                                     const SDLoc &dl, MVT ValTy,
  2706:                                     SelectionDAG &DAG) const {
  2707:   MVT VecTy = ty(VecV);
  2708:   if (VecTy.getVectorElementType() == MVT::i1)
  2709:     return insertVectorPred(VecV, ValV, IdxV, dl, ValTy, DAG);
  2710: 
  2711:   unsigned VecWidth = VecTy.getSizeInBits();
  2712:   unsigned ValWidth = ValTy.getSizeInBits();
  2713:   assert(VecWidth == 32 || VecWidth == 64);
  2714:   assert((VecWidth % ValWidth) == 0);
  2715: 
  2716:   // Cast everything to scalar integer types.
  2717:   MVT ScalarTy = MVT::getIntegerVT(VecWidth);
  2718:   // The actual type of ValV may be different than ValTy (which is related
  2719:   // to the vector type).
  2720:   unsigned VW = ty(ValV).getSizeInBits();
  2721:   ValV = DAG.getBitcast(MVT::getIntegerVT(VW), ValV);
  2722:   VecV = DAG.getBitcast(ScalarTy, VecV);
  2723:   if (VW != VecWidth)
  2724:     ValV = DAG.getAnyExtOrTrunc(ValV, dl, ScalarTy);
  2725: 
  2726:   SDValue WidthV = DAG.getConstant(ValWidth, dl, MVT::i32);
  2727:   SDValue InsV;
  2728: 
  2729:   if (ConstantSDNode *C = dyn_cast<ConstantSDNode>(IdxV)) {
  2730:     unsigned W = C->getZExtValue() * ValWidth;
  2731:     SDValue OffV = DAG.getConstant(W, dl, MVT::i32);
  2732:     InsV = DAG.getNode(HexagonISD::INSERT, dl, ScalarTy,
  2733:                        {VecV, ValV, WidthV, OffV});
  2734:   } else {
  2735:     if (ty(IdxV) != MVT::i32)
  2736:       IdxV = DAG.getZExtOrTrunc(IdxV, dl, MVT::i32);
  2737:     SDValue OffV = DAG.getNode(ISD::MUL, dl, MVT::i32, IdxV, WidthV);
  2738:     InsV = DAG.getNode(HexagonISD::INSERT, dl, ScalarTy,
  2739:                        {VecV, ValV, WidthV, OffV});
  2740:   }
  2741: 
  2742:   return DAG.getNode(ISD::BITCAST, dl, VecTy, InsV);
  2743: }
  2744: 
  2745: SDValue
  2746: HexagonTargetLowering::insertVectorPred(SDValue VecV, SDValue ValV,
  2747:                                         SDValue IdxV, const SDLoc &dl,
  2748:                                         MVT ValTy, SelectionDAG &DAG) const {
  2749:   MVT VecTy = ty(VecV);
  2750:   unsigned VecLen = VecTy.getVectorNumElements();
  2751: 
  2752:   if (ValTy == MVT::i1) {
  2753:     SDValue ToReg = getInstr(Hexagon::C2_tfrpr, dl, MVT::i32, {VecV}, DAG);
  2754:     SDValue Ext = DAG.getSExtOrTrunc(ValV, dl, MVT::i32);
  2755:     SDValue Width = DAG.getConstant(8 / VecLen, dl, MVT::i32);
  2756:     SDValue Idx = DAG.getNode(ISD::MUL, dl, MVT::i32, IdxV, Width);
  2757:     SDValue Ins =
  2758:         DAG.getNode(HexagonISD::INSERT, dl, MVT::i32, {ToReg, Ext, Width, Idx});
  2759:     return getInstr(Hexagon::C2_tfrrp, dl, VecTy, {Ins}, DAG);
  2760:   }
  2761: 
  2762:   assert(ValTy.getVectorElementType() == MVT::i1);
  2763:   SDValue ValR = ValTy.isVector()
  2764:                      ? DAG.getNode(HexagonISD::P2D, dl, MVT::i64, ValV)
  2765:                      : DAG.getSExtOrTrunc(ValV, dl, MVT::i64);
  2766: 
  2767:   unsigned Scale = VecLen / ValTy.getVectorNumElements();
  2768:   assert(Scale > 1);
  2769: 
  2770:   for (unsigned R = Scale; R > 1; R /= 2) {
  2771:     ValR = contractPredicate(ValR, dl, DAG);
  2772:     ValR = getCombine(DAG.getUNDEF(MVT::i32), ValR, dl, MVT::i64, DAG);
  2773:   }
  2774: 
  2775:   SDValue Width = DAG.getConstant(64 / Scale, dl, MVT::i32);
  2776:   SDValue Idx = DAG.getNode(ISD::MUL, dl, MVT::i32, IdxV, Width);
  2777:   SDValue VecR = DAG.getNode(HexagonISD::P2D, dl, MVT::i64, VecV);
  2778:   SDValue Ins =
  2779:       DAG.getNode(HexagonISD::INSERT, dl, MVT::i64, {VecR, ValR, Width, Idx});
  2780:   return DAG.getNode(HexagonISD::D2P, dl, VecTy, Ins);
  2781: }
  2782: 
  2783: SDValue
  2784: HexagonTargetLowering::expandPredicate(SDValue Vec32, const SDLoc &dl,
  2785:                                        SelectionDAG &DAG) const {
  2786:   assert(ty(Vec32).getSizeInBits() == 32);
  2787:   if (isUndef(Vec32))
  2788:     return DAG.getUNDEF(MVT::i64);
  2789:   SDValue P = DAG.getBitcast(MVT::v4i8, Vec32);
  2790:   SDValue X = DAG.getNode(ISD::SIGN_EXTEND, dl, MVT::v4i16, P);
  2791:   return DAG.getBitcast(MVT::i64, X);
  2792: }
  2793: 
  2794: SDValue
  2795: HexagonTargetLowering::contractPredicate(SDValue Vec64, const SDLoc &dl,
  2796:                                          SelectionDAG &DAG) const {
  2797:   assert(ty(Vec64).getSizeInBits() == 64);
  2798:   if (isUndef(Vec64))
  2799:     return DAG.getUNDEF(MVT::i32);
  2800:   // Collect even bytes:
```
- EN: It declares or implements routines such as ty, assert, getSizeInBits, getVectorElementType, ... (26 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonISD, HexagonTargetLowering, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 ty, assert, getSizeInBits, getVectorElementType, ... (26 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonISD, HexagonTargetLowering，说明了它与同级后端组件的连接关系。

### Lines 2801-3000 / 第 2801-3000 行

```cpp
  2801:   SDValue A = DAG.getBitcast(MVT::v8i8, Vec64);
  2802:   SDValue S = DAG.getVectorShuffle(MVT::v8i8, dl, A, DAG.getUNDEF(MVT::v8i8),
  2803:                                    {0, 2, 4, 6, 1, 3, 5, 7});
  2804:   return extractVector(S, DAG.getConstant(0, dl, MVT::i32), dl, MVT::v4i8,
  2805:                        MVT::i32, DAG);
  2806: }
  2807: 
  2808: SDValue
  2809: HexagonTargetLowering::getZero(const SDLoc &dl, MVT Ty, SelectionDAG &DAG)
  2810:       const {
  2811:   if (Ty.isVector()) {
  2812:     unsigned W = Ty.getSizeInBits();
  2813:     if (W <= 64)
  2814:       return DAG.getBitcast(Ty, DAG.getConstant(0, dl, MVT::getIntegerVT(W)));
  2815:     return DAG.getNode(ISD::SPLAT_VECTOR, dl, Ty, getZero(dl, MVT::i32, DAG));
  2816:   }
  2817: 
  2818:   if (Ty.isInteger())
  2819:     return DAG.getConstant(0, dl, Ty);
  2820:   if (Ty.isFloatingPoint())
  2821:     return DAG.getConstantFP(0.0, dl, Ty);
  2822:   llvm_unreachable("Invalid type for zero");
  2823: }
  2824: 
  2825: SDValue
  2826: HexagonTargetLowering::appendUndef(SDValue Val, MVT ResTy, SelectionDAG &DAG)
  2827:       const {
  2828:   MVT ValTy = ty(Val);
  2829:   assert(ValTy.getVectorElementType() == ResTy.getVectorElementType());
  2830: 
  2831:   unsigned ValLen = ValTy.getVectorNumElements();
  2832:   unsigned ResLen = ResTy.getVectorNumElements();
  2833:   if (ValLen == ResLen)
  2834:     return Val;
  2835: 
  2836:   const SDLoc &dl(Val);
  2837:   assert(ValLen < ResLen);
  2838:   assert(ResLen % ValLen == 0);
  2839: 
  2840:   SmallVector<SDValue, 4> Concats = {Val};
  2841:   for (unsigned i = 1, e = ResLen / ValLen; i < e; ++i)
  2842:     Concats.push_back(DAG.getUNDEF(ValTy));
  2843: 
  2844:   return DAG.getNode(ISD::CONCAT_VECTORS, dl, ResTy, Concats);
  2845: }
  2846: 
  2847: SDValue
  2848: HexagonTargetLowering::getCombine(SDValue Hi, SDValue Lo, const SDLoc &dl,
  2849:                                   MVT ResTy, SelectionDAG &DAG) const {
  2850:   MVT ElemTy = ty(Hi);
  2851:   assert(ElemTy == ty(Lo));
  2852: 
  2853:   if (!ElemTy.isVector()) {
  2854:     assert(ElemTy.isScalarInteger());
  2855:     MVT PairTy = MVT::getIntegerVT(2 * ElemTy.getSizeInBits());
  2856:     SDValue Pair = DAG.getNode(ISD::BUILD_PAIR, dl, PairTy, Lo, Hi);
  2857:     return DAG.getBitcast(ResTy, Pair);
  2858:   }
  2859: 
  2860:   unsigned Width = ElemTy.getSizeInBits();
  2861:   MVT IntTy = MVT::getIntegerVT(Width);
  2862:   MVT PairTy = MVT::getIntegerVT(2 * Width);
  2863:   SDValue Pair =
  2864:       DAG.getNode(ISD::BUILD_PAIR, dl, PairTy,
  2865:                   {DAG.getBitcast(IntTy, Lo), DAG.getBitcast(IntTy, Hi)});
  2866:   return DAG.getBitcast(ResTy, Pair);
  2867: }
  2868: 
  2869: SDValue
  2870: HexagonTargetLowering::LowerBUILD_VECTOR(SDValue Op, SelectionDAG &DAG) const {
  2871:   MVT VecTy = ty(Op);
  2872:   unsigned BW = VecTy.getSizeInBits();
  2873:   const SDLoc &dl(Op);
  2874:   SmallVector<SDValue,8> Ops;
  2875:   for (unsigned i = 0, e = Op.getNumOperands(); i != e; ++i)
  2876:     Ops.push_back(Op.getOperand(i));
  2877: 
  2878:   if (BW == 32)
  2879:     return buildVector32(Ops, dl, VecTy, DAG);
  2880:   if (BW == 64)
  2881:     return buildVector64(Ops, dl, VecTy, DAG);
  2882: 
  2883:   if (VecTy == MVT::v8i1 || VecTy == MVT::v4i1 || VecTy == MVT::v2i1) {
  2884:     // Check if this is a special case or all-0 or all-1.
  2885:     bool All0 = true, All1 = true;
  2886:     for (SDValue P : Ops) {
  2887:       auto *CN = dyn_cast<ConstantSDNode>(P.getNode());
  2888:       if (CN == nullptr) {
  2889:         All0 = All1 = false;
  2890:         break;
  2891:       }
  2892:       uint32_t C = CN->getZExtValue();
  2893:       All0 &= (C == 0);
  2894:       All1 &= (C == 1);
  2895:     }
  2896:     if (All0)
  2897:       return DAG.getNode(HexagonISD::PFALSE, dl, VecTy);
  2898:     if (All1)
  2899:       return DAG.getNode(HexagonISD::PTRUE, dl, VecTy);
  2900: 
  2901:     // For each i1 element in the resulting predicate register, put 1
  2902:     // shifted by the index of the element into a general-purpose register,
  2903:     // then or them together and transfer it back into a predicate register.
  2904:     SDValue Rs[8];
  2905:     SDValue Z = getZero(dl, MVT::i32, DAG);
  2906:     // Always produce 8 bits, repeat inputs if necessary.
  2907:     unsigned Rep = 8 / VecTy.getVectorNumElements();
  2908:     for (unsigned i = 0; i != 8; ++i) {
  2909:       SDValue S = DAG.getConstant(1ull << i, dl, MVT::i32);
  2910:       Rs[i] = DAG.getSelect(dl, MVT::i32, Ops[i/Rep], S, Z);
  2911:     }
  2912:     for (ArrayRef<SDValue> A(Rs); A.size() != 1; A = A.drop_back(A.size()/2)) {
  2913:       for (unsigned i = 0, e = A.size()/2; i != e; ++i)
  2914:         Rs[i] = DAG.getNode(ISD::OR, dl, MVT::i32, Rs[2*i], Rs[2*i+1]);
  2915:     }
  2916:     // Move the value directly to a predicate register.
  2917:     return getInstr(Hexagon::C2_tfrrp, dl, VecTy, {Rs[0]}, DAG);
  2918:   }
  2919: 
  2920:   return SDValue();
  2921: }
  2922: 
  2923: SDValue
  2924: HexagonTargetLowering::LowerCONCAT_VECTORS(SDValue Op,
  2925:                                            SelectionDAG &DAG) const {
  2926:   MVT VecTy = ty(Op);
  2927:   const SDLoc &dl(Op);
  2928:   if (VecTy.getSizeInBits() == 64) {
  2929:     assert(Op.getNumOperands() == 2);
  2930:     return getCombine(Op.getOperand(1), Op.getOperand(0), dl, VecTy, DAG);
  2931:   }
  2932: 
  2933:   MVT ElemTy = VecTy.getVectorElementType();
  2934:   if (ElemTy == MVT::i1) {
  2935:     assert(VecTy == MVT::v2i1 || VecTy == MVT::v4i1 || VecTy == MVT::v8i1);
  2936:     MVT OpTy = ty(Op.getOperand(0));
  2937:     // Scale is how many times the operands need to be contracted to match
  2938:     // the representation in the target register.
  2939:     unsigned Scale = VecTy.getVectorNumElements() / OpTy.getVectorNumElements();
  2940:     assert(Scale == Op.getNumOperands() && Scale > 1);
  2941: 
  2942:     // First, convert all bool vectors to integers, then generate pairwise
  2943:     // inserts to form values of doubled length. Up until there are only
  2944:     // two values left to concatenate, all of these values will fit in a
  2945:     // 32-bit integer, so keep them as i32 to use 32-bit inserts.
  2946:     SmallVector<SDValue,4> Words[2];
  2947:     unsigned IdxW = 0;
  2948: 
  2949:     for (SDValue P : Op.getNode()->op_values()) {
  2950:       SDValue W = DAG.getNode(HexagonISD::P2D, dl, MVT::i64, P);
  2951:       for (unsigned R = Scale; R > 1; R /= 2) {
  2952:         W = contractPredicate(W, dl, DAG);
  2953:         W = getCombine(DAG.getUNDEF(MVT::i32), W, dl, MVT::i64, DAG);
  2954:       }
  2955:       W = LoHalf(W, DAG);
  2956:       Words[IdxW].push_back(W);
  2957:     }
  2958: 
  2959:     while (Scale > 2) {
  2960:       SDValue WidthV = DAG.getConstant(64 / Scale, dl, MVT::i32);
  2961:       Words[IdxW ^ 1].clear();
  2962: 
  2963:       for (unsigned i = 0, e = Words[IdxW].size(); i != e; i += 2) {
  2964:         SDValue W0 = Words[IdxW][i], W1 = Words[IdxW][i+1];
  2965:         // Insert W1 into W0 right next to the significant bits of W0.
  2966:         SDValue T = DAG.getNode(HexagonISD::INSERT, dl, MVT::i32,
  2967:                                 {W0, W1, WidthV, WidthV});
  2968:         Words[IdxW ^ 1].push_back(T);
  2969:       }
  2970:       IdxW ^= 1;
  2971:       Scale /= 2;
  2972:     }
  2973: 
  2974:     // At this point there should only be two words left, and Scale should be 2.
  2975:     assert(Scale == 2 && Words[IdxW].size() == 2);
  2976: 
  2977:     SDValue WW = getCombine(Words[IdxW][1], Words[IdxW][0], dl, MVT::i64, DAG);
  2978:     return DAG.getNode(HexagonISD::D2P, dl, VecTy, WW);
  2979:   }
  2980: 
  2981:   return SDValue();
  2982: }
  2983: 
  2984: SDValue
  2985: HexagonTargetLowering::LowerEXTRACT_VECTOR_ELT(SDValue Op,
  2986:                                                SelectionDAG &DAG) const {
  2987:   SDValue Vec = Op.getOperand(0);
  2988:   MVT ElemTy = ty(Vec).getVectorElementType();
  2989:   return extractVector(Vec, Op.getOperand(1), SDLoc(Op), ElemTy, ty(Op), DAG);
  2990: }
  2991: 
  2992: SDValue
  2993: HexagonTargetLowering::LowerEXTRACT_SUBVECTOR(SDValue Op,
  2994:                                               SelectionDAG &DAG) const {
  2995:   return extractVector(Op.getOperand(0), Op.getOperand(1), SDLoc(Op),
  2996:                        ty(Op), ty(Op), DAG);
  2997: }
  2998: 
  2999: SDValue
  3000: HexagonTargetLowering::LowerINSERT_VECTOR_ELT(SDValue Op,
```
- EN: It declares or implements routines such as getBitcast, extractVector, HexagonTargetLowering::getZero, getSizeInBits, ... (31 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonTargetLowering, HexagonISD, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getBitcast, extractVector, HexagonTargetLowering::getZero, getSizeInBits, ... (31 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonTargetLowering, HexagonISD，说明了它与同级后端组件的连接关系。

### Lines 3001-3200 / 第 3001-3200 行

```cpp
  3001:                                               SelectionDAG &DAG) const {
  3002:   return insertVector(Op.getOperand(0), Op.getOperand(1), Op.getOperand(2),
  3003:                       SDLoc(Op), ty(Op).getVectorElementType(), DAG);
  3004: }
  3005: 
  3006: SDValue
  3007: HexagonTargetLowering::LowerINSERT_SUBVECTOR(SDValue Op,
  3008:                                              SelectionDAG &DAG) const {
  3009:   SDValue ValV = Op.getOperand(1);
  3010:   return insertVector(Op.getOperand(0), ValV, Op.getOperand(2),
  3011:                       SDLoc(Op), ty(ValV), DAG);
  3012: }
  3013: 
  3014: bool
  3015: HexagonTargetLowering::allowTruncateForTailCall(Type *Ty1, Type *Ty2) const {
  3016:   // Assuming the caller does not have either a signext or zeroext modifier, and
  3017:   // only one value is accepted, any reasonable truncation is allowed.
  3018:   if (!Ty1->isIntegerTy() || !Ty2->isIntegerTy())
  3019:     return false;
  3020: 
  3021:   // FIXME: in principle up to 64-bit could be made safe, but it would be very
  3022:   // fragile at the moment: any support for multiple value returns would be
  3023:   // liable to disallow tail calls involving i64 -> iN truncation in many cases.
  3024:   return Ty1->getPrimitiveSizeInBits() <= 32;
  3025: }
  3026: 
  3027: SDValue
  3028: HexagonTargetLowering::LowerLoad(SDValue Op, SelectionDAG &DAG) const {
  3029:   MVT Ty = ty(Op);
  3030:   const SDLoc &dl(Op);
  3031:   LoadSDNode *LN = cast<LoadSDNode>(Op.getNode());
  3032:   MVT MemTy = LN->getMemoryVT().getSimpleVT();
  3033:   ISD::LoadExtType ET = LN->getExtensionType();
  3034: 
  3035:   bool LoadPred = MemTy == MVT::v2i1 || MemTy == MVT::v4i1 || MemTy == MVT::v8i1;
  3036:   if (LoadPred) {
  3037:     SDValue NL = DAG.getLoad(
  3038:         LN->getAddressingMode(), ISD::ZEXTLOAD, MVT::i32, dl, LN->getChain(),
  3039:         LN->getBasePtr(), LN->getOffset(), LN->getPointerInfo(),
  3040:         /*MemoryVT*/ MVT::i8, LN->getAlign(), LN->getMemOperand()->getFlags(),
  3041:         LN->getAAInfo(), LN->getRanges());
  3042:     LN = cast<LoadSDNode>(NL.getNode());
  3043:   }
  3044: 
  3045:   Align ClaimAlign = LN->getAlign();
  3046:   if (!validateConstPtrAlignment(LN->getBasePtr(), ClaimAlign, dl, DAG))
  3047:     return replaceMemWithUndef(Op, DAG);
  3048: 
  3049:   // Call LowerUnalignedLoad for all loads, it recognizes loads that
  3050:   // don't need extra aligning.
  3051:   SDValue LU = LowerUnalignedLoad(SDValue(LN, 0), DAG);
  3052:   if (LoadPred) {
  3053:     SDValue TP = getInstr(Hexagon::C2_tfrrp, dl, MemTy, {LU}, DAG);
  3054:     if (ET == ISD::SEXTLOAD) {
  3055:       TP = DAG.getSExtOrTrunc(TP, dl, Ty);
  3056:     } else if (ET != ISD::NON_EXTLOAD) {
  3057:       TP = DAG.getZExtOrTrunc(TP, dl, Ty);
  3058:     }
  3059:     SDValue Ch = cast<LoadSDNode>(LU.getNode())->getChain();
  3060:     return DAG.getMergeValues({TP, Ch}, dl);
  3061:   }
  3062:   return LU;
  3063: }
  3064: 
  3065: SDValue
  3066: HexagonTargetLowering::LowerStore(SDValue Op, SelectionDAG &DAG) const {
  3067:   const SDLoc &dl(Op);
  3068:   StoreSDNode *SN = cast<StoreSDNode>(Op.getNode());
  3069:   SDValue Val = SN->getValue();
  3070:   MVT Ty = ty(Val);
  3071: 
  3072:   if (Ty == MVT::v2i1 || Ty == MVT::v4i1 || Ty == MVT::v8i1) {
  3073:     // Store the exact predicate (all bits).
  3074:     SDValue TR = getInstr(Hexagon::C2_tfrpr, dl, MVT::i32, {Val}, DAG);
  3075:     SDValue NS = DAG.getTruncStore(SN->getChain(), dl, TR, SN->getBasePtr(),
  3076:                                    MVT::i8, SN->getMemOperand());
  3077:     if (SN->isIndexed()) {
  3078:       NS = DAG.getIndexedStore(NS, dl, SN->getBasePtr(), SN->getOffset(),
  3079:                                SN->getAddressingMode());
  3080:     }
  3081:     SN = cast<StoreSDNode>(NS.getNode());
  3082:   }
  3083: 
  3084:   Align ClaimAlign = SN->getAlign();
  3085:   if (!validateConstPtrAlignment(SN->getBasePtr(), ClaimAlign, dl, DAG))
  3086:     return replaceMemWithUndef(Op, DAG);
  3087: 
  3088:   MVT StoreTy = SN->getMemoryVT().getSimpleVT();
  3089:   Align NeedAlign = Subtarget.getTypeAlignment(StoreTy);
  3090:   if (ClaimAlign < NeedAlign)
  3091:     return expandUnalignedStore(SN, DAG);
  3092:   return SDValue(SN, 0);
  3093: }
  3094: 
  3095: SDValue
  3096: HexagonTargetLowering::LowerUnalignedLoad(SDValue Op, SelectionDAG &DAG)
  3097:       const {
  3098:   LoadSDNode *LN = cast<LoadSDNode>(Op.getNode());
  3099:   MVT LoadTy = ty(Op);
  3100:   unsigned NeedAlign = Subtarget.getTypeAlignment(LoadTy).value();
  3101:   unsigned HaveAlign = LN->getAlign().value();
  3102:   if (HaveAlign >= NeedAlign)
  3103:     return Op;
  3104: 
  3105:   const SDLoc &dl(Op);
  3106:   const DataLayout &DL = DAG.getDataLayout();
  3107:   LLVMContext &Ctx = *DAG.getContext();
  3108: 
  3109:   // If the load aligning is disabled or the load can be broken up into two
  3110:   // smaller legal loads, do the default (target-independent) expansion.
  3111:   bool DoDefault = false;
  3112:   // Handle it in the default way if this is an indexed load.
  3113:   if (!LN->isUnindexed())
  3114:     DoDefault = true;
  3115: 
  3116:   if (!AlignLoads) {
  3117:     if (allowsMemoryAccessForAlignment(Ctx, DL, LN->getMemoryVT(),
  3118:                                        *LN->getMemOperand()))
  3119:       return Op;
  3120:     DoDefault = true;
  3121:   }
  3122:   if (!DoDefault && (2 * HaveAlign) == NeedAlign) {
  3123:     // The PartTy is the equivalent of "getLoadableTypeOfSize(HaveAlign)".
  3124:     MVT PartTy = HaveAlign <= 8 ? MVT::getIntegerVT(8 * HaveAlign)
  3125:                                 : MVT::getVectorVT(MVT::i8, HaveAlign);
  3126:     DoDefault =
  3127:         allowsMemoryAccessForAlignment(Ctx, DL, PartTy, *LN->getMemOperand());
  3128:   }
  3129:   if (DoDefault) {
  3130:     std::pair<SDValue, SDValue> P = expandUnalignedLoad(LN, DAG);
  3131:     return DAG.getMergeValues({P.first, P.second}, dl);
  3132:   }
  3133: 
  3134:   // The code below generates two loads, both aligned as NeedAlign, and
  3135:   // with the distance of NeedAlign between them. For that to cover the
  3136:   // bits that need to be loaded (and without overlapping), the size of
  3137:   // the loads should be equal to NeedAlign. This is true for all loadable
  3138:   // types, but add an assertion in case something changes in the future.
  3139:   assert(LoadTy.getSizeInBits() == 8*NeedAlign);
  3140: 
  3141:   unsigned LoadLen = NeedAlign;
  3142:   SDValue Base = LN->getBasePtr();
  3143:   SDValue Chain = LN->getChain();
  3144:   auto BO = getBaseAndOffset(Base);
  3145:   unsigned BaseOpc = BO.first.getOpcode();
  3146:   if (BaseOpc == HexagonISD::VALIGNADDR && BO.second % LoadLen == 0)
  3147:     return Op;
  3148: 
  3149:   if (BO.second % LoadLen != 0) {
  3150:     BO.first = DAG.getNode(ISD::ADD, dl, MVT::i32, BO.first,
  3151:                            DAG.getConstant(BO.second % LoadLen, dl, MVT::i32));
  3152:     BO.second -= BO.second % LoadLen;
  3153:   }
  3154:   SDValue BaseNoOff = (BaseOpc != HexagonISD::VALIGNADDR)
  3155:       ? DAG.getNode(HexagonISD::VALIGNADDR, dl, MVT::i32, BO.first,
  3156:                     DAG.getConstant(NeedAlign, dl, MVT::i32))
  3157:       : BO.first;
  3158:   SDValue Base0 =
  3159:       DAG.getMemBasePlusOffset(BaseNoOff, TypeSize::getFixed(BO.second), dl);
  3160:   SDValue Base1 = DAG.getMemBasePlusOffset(
  3161:       BaseNoOff, TypeSize::getFixed(BO.second + LoadLen), dl);
  3162: 
  3163:   MachineMemOperand *WideMMO = nullptr;
  3164:   if (MachineMemOperand *MMO = LN->getMemOperand()) {
  3165:     MachineFunction &MF = DAG.getMachineFunction();
  3166:     WideMMO = MF.getMachineMemOperand(
  3167:         MMO->getPointerInfo(), MMO->getFlags(), 2 * LoadLen, Align(LoadLen),
  3168:         MMO->getAAInfo(), MMO->getRanges(), MMO->getSyncScopeID(),
  3169:         MMO->getSuccessOrdering(), MMO->getFailureOrdering());
  3170:   }
  3171: 
  3172:   SDValue Load0 = DAG.getLoad(LoadTy, dl, Chain, Base0, WideMMO);
  3173:   SDValue Load1 = DAG.getLoad(LoadTy, dl, Chain, Base1, WideMMO);
  3174: 
  3175:   SDValue Aligned = DAG.getNode(HexagonISD::VALIGN, dl, LoadTy,
  3176:                                 {Load1, Load0, BaseNoOff.getOperand(0)});
  3177:   SDValue NewChain = DAG.getNode(ISD::TokenFactor, dl, MVT::Other,
  3178:                                  Load0.getValue(1), Load1.getValue(1));
  3179:   SDValue M = DAG.getMergeValues({Aligned, NewChain}, dl);
  3180:   return M;
  3181: }
  3182: 
  3183: SDValue
  3184: HexagonTargetLowering::LowerUAddSubO(SDValue Op, SelectionDAG &DAG) const {
  3185:   SDValue X = Op.getOperand(0), Y = Op.getOperand(1);
  3186:   auto *CY = dyn_cast<ConstantSDNode>(Y);
  3187:   if (!CY)
  3188:     return SDValue();
  3189: 
  3190:   const SDLoc &dl(Op);
  3191:   SDVTList VTs = Op.getNode()->getVTList();
  3192:   assert(VTs.NumVTs == 2);
  3193:   assert(VTs.VTs[1] == MVT::i1);
  3194:   unsigned Opc = Op.getOpcode();
  3195: 
  3196:   if (CY) {
  3197:     uint64_t VY = CY->getZExtValue();
  3198:     assert(VY != 0 && "This should have been folded");
  3199:     // X +/- 1
  3200:     if (VY != 1)
```
- EN: It declares or implements routines such as insertVector, HexagonTargetLowering::LowerINSERT_SUBVECTOR, getOperand, HexagonTargetLowering::allowTruncateForTailCall, ... (41 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonTargetLowering, HexagonISD, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 insertVector, HexagonTargetLowering::LowerINSERT_SUBVECTOR, getOperand, HexagonTargetLowering::allowTruncateForTailCall, ... (41 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonTargetLowering, HexagonISD，说明了它与同级后端组件的连接关系。

### Lines 3201-3400 / 第 3201-3400 行

```cpp
  3201:       return SDValue();
  3202: 
  3203:     if (Opc == ISD::UADDO) {
  3204:       SDValue Op = DAG.getNode(ISD::ADD, dl, VTs.VTs[0], {X, Y});
  3205:       SDValue Ov = DAG.getSetCC(dl, MVT::i1, Op, getZero(dl, ty(Op), DAG),
  3206:                                 ISD::SETEQ);
  3207:       return DAG.getMergeValues({Op, Ov}, dl);
  3208:     }
  3209:     if (Opc == ISD::USUBO) {
  3210:       SDValue Op = DAG.getNode(ISD::SUB, dl, VTs.VTs[0], {X, Y});
  3211:       SDValue Ov = DAG.getSetCC(dl, MVT::i1, Op,
  3212:                                 DAG.getAllOnesConstant(dl, ty(Op)), ISD::SETEQ);
  3213:       return DAG.getMergeValues({Op, Ov}, dl);
  3214:     }
  3215:   }
  3216: 
  3217:   return SDValue();
  3218: }
  3219: 
  3220: SDValue HexagonTargetLowering::LowerUAddSubOCarry(SDValue Op,
  3221:                                                   SelectionDAG &DAG) const {
  3222:   const SDLoc &dl(Op);
  3223:   unsigned Opc = Op.getOpcode();
  3224:   SDValue X = Op.getOperand(0), Y = Op.getOperand(1), C = Op.getOperand(2);
  3225: 
  3226:   if (Opc == ISD::UADDO_CARRY)
  3227:     return DAG.getNode(HexagonISD::ADDC, dl, Op.getNode()->getVTList(),
  3228:                        { X, Y, C });
  3229: 
  3230:   EVT CarryTy = C.getValueType();
  3231:   SDValue SubC = DAG.getNode(HexagonISD::SUBC, dl, Op.getNode()->getVTList(),
  3232:                              { X, Y, DAG.getLogicalNOT(dl, C, CarryTy) });
  3233:   SDValue Out[] = { SubC.getValue(0),
  3234:                     DAG.getLogicalNOT(dl, SubC.getValue(1), CarryTy) };
  3235:   return DAG.getMergeValues(Out, dl);
  3236: }
  3237: 
  3238: SDValue
  3239: HexagonTargetLowering::LowerEH_RETURN(SDValue Op, SelectionDAG &DAG) const {
  3240:   SDValue Chain     = Op.getOperand(0);
  3241:   SDValue Offset    = Op.getOperand(1);
  3242:   SDValue Handler   = Op.getOperand(2);
  3243:   SDLoc dl(Op);
  3244:   auto PtrVT = getPointerTy(DAG.getDataLayout());
  3245: 
  3246:   // Mark function as containing a call to EH_RETURN.
  3247:   HexagonMachineFunctionInfo *FuncInfo =
  3248:     DAG.getMachineFunction().getInfo<HexagonMachineFunctionInfo>();
  3249:   FuncInfo->setHasEHReturn();
  3250: 
  3251:   unsigned OffsetReg = Hexagon::R28;
  3252: 
  3253:   SDValue StoreAddr =
  3254:       DAG.getNode(ISD::ADD, dl, PtrVT, DAG.getRegister(Hexagon::R30, PtrVT),
  3255:                   DAG.getIntPtrConstant(4, dl));
  3256:   Chain = DAG.getStore(Chain, dl, Handler, StoreAddr, MachinePointerInfo());
  3257:   Chain = DAG.getCopyToReg(Chain, dl, OffsetReg, Offset);
  3258: 
  3259:   // Not needed we already use it as explicit input to EH_RETURN.
  3260:   // MF.getRegInfo().addLiveOut(OffsetReg);
  3261: 
  3262:   return DAG.getNode(HexagonISD::EH_RETURN, dl, MVT::Other, Chain);
  3263: }
  3264: 
  3265: SDValue
  3266: HexagonTargetLowering::LowerOperation(SDValue Op, SelectionDAG &DAG) const {
  3267:   unsigned Opc = Op.getOpcode();
  3268:   // Handle INLINEASM first.
  3269:   if (Opc == ISD::INLINEASM || Opc == ISD::INLINEASM_BR)
  3270:     return LowerINLINEASM(Op, DAG);
  3271: 
  3272:   if (isHvxOperation(Op.getNode(), DAG)) {
  3273:     // If HVX lowering returns nothing, try the default lowering.
  3274:     if (SDValue V = LowerHvxOperation(Op, DAG))
  3275:       return V;
  3276:   }
  3277: 
  3278:   switch (Opc) {
  3279:     default:
  3280: #ifndef NDEBUG
  3281:       Op.getNode()->dumpr(&DAG);
  3282: #endif
  3283:       llvm_unreachable("Should not custom lower this!");
  3284: 
  3285:     case ISD::FDIV:
  3286:       return LowerFDIV(Op, DAG);
  3287:     case ISD::CONCAT_VECTORS:       return LowerCONCAT_VECTORS(Op, DAG);
  3288:     case ISD::INSERT_SUBVECTOR:     return LowerINSERT_SUBVECTOR(Op, DAG);
  3289:     case ISD::INSERT_VECTOR_ELT:    return LowerINSERT_VECTOR_ELT(Op, DAG);
  3290:     case ISD::EXTRACT_SUBVECTOR:    return LowerEXTRACT_SUBVECTOR(Op, DAG);
  3291:     case ISD::EXTRACT_VECTOR_ELT:   return LowerEXTRACT_VECTOR_ELT(Op, DAG);
  3292:     case ISD::BUILD_VECTOR:         return LowerBUILD_VECTOR(Op, DAG);
  3293:     case ISD::VECTOR_SHUFFLE:       return LowerVECTOR_SHUFFLE(Op, DAG);
  3294:     case ISD::BITCAST:              return LowerBITCAST(Op, DAG);
  3295:     case ISD::LOAD:                 return LowerLoad(Op, DAG);
  3296:     case ISD::STORE:                return LowerStore(Op, DAG);
  3297:     case ISD::UADDO:
  3298:     case ISD::USUBO:                return LowerUAddSubO(Op, DAG);
  3299:     case ISD::UADDO_CARRY:
  3300:     case ISD::USUBO_CARRY:          return LowerUAddSubOCarry(Op, DAG);
  3301:     case ISD::SRA:
  3302:     case ISD::SHL:
  3303:     case ISD::SRL:                  return LowerVECTOR_SHIFT(Op, DAG);
  3304:     case ISD::ROTL:                 return LowerROTL(Op, DAG);
  3305:     case ISD::ConstantPool:         return LowerConstantPool(Op, DAG);
  3306:     case ISD::JumpTable:            return LowerJumpTable(Op, DAG);
  3307:     case ISD::EH_RETURN:            return LowerEH_RETURN(Op, DAG);
  3308:     case ISD::RETURNADDR:           return LowerRETURNADDR(Op, DAG);
  3309:     case ISD::FRAMEADDR:            return LowerFRAMEADDR(Op, DAG);
  3310:     case ISD::GlobalTLSAddress:     return LowerGlobalTLSAddress(Op, DAG);
  3311:     case ISD::ATOMIC_FENCE:         return LowerATOMIC_FENCE(Op, DAG);
  3312:     case ISD::GlobalAddress:        return LowerGLOBALADDRESS(Op, DAG);
  3313:     case ISD::BlockAddress:         return LowerBlockAddress(Op, DAG);
  3314:     case ISD::GLOBAL_OFFSET_TABLE:  return LowerGLOBAL_OFFSET_TABLE(Op, DAG);
  3315:     case ISD::VACOPY:               return LowerVACOPY(Op, DAG);
  3316:     case ISD::VASTART:              return LowerVASTART(Op, DAG);
  3317:     case ISD::DYNAMIC_STACKALLOC:   return LowerDYNAMIC_STACKALLOC(Op, DAG);
  3318:     case ISD::SETCC:                return LowerSETCC(Op, DAG);
  3319:     case ISD::VSELECT:              return LowerVSELECT(Op, DAG);
  3320:     case ISD::INTRINSIC_WO_CHAIN:   return LowerINTRINSIC_WO_CHAIN(Op, DAG);
  3321:     case ISD::INTRINSIC_VOID:       return LowerINTRINSIC_VOID(Op, DAG);
  3322:     case ISD::PREFETCH:
  3323:       return LowerPREFETCH(Op, DAG);
  3324:       break;
  3325:   }
  3326: 
  3327:   return SDValue();
  3328: }
  3329: 
  3330: void
  3331: HexagonTargetLowering::LowerOperationWrapper(SDNode *N,
  3332:                                              SmallVectorImpl<SDValue> &Results,
  3333:                                              SelectionDAG &DAG) const {
  3334:   if (isHvxOperation(N, DAG)) {
  3335:     LowerHvxOperationWrapper(N, Results, DAG);
  3336:     if (!Results.empty())
  3337:       return;
  3338:   }
  3339: 
  3340:   SDValue Op(N, 0);
  3341:   unsigned Opc = N->getOpcode();
  3342: 
  3343:   switch (Opc) {
  3344:     case HexagonISD::SSAT:
  3345:     case HexagonISD::USAT:
  3346:       Results.push_back(opJoin(SplitVectorOp(Op, DAG), SDLoc(Op), DAG));
  3347:       break;
  3348:     case ISD::STORE:
  3349:       // We are only custom-lowering stores to verify the alignment of the
  3350:       // address if it is a compile-time constant. Since a store can be
  3351:       // modified during type-legalization (the value being stored may need
  3352:       // legalization), return empty Results here to indicate that we don't
  3353:       // really make any changes in the custom lowering.
  3354:       return;
  3355:     default:
  3356:       TargetLowering::LowerOperationWrapper(N, Results, DAG);
  3357:       break;
  3358:   }
  3359: }
  3360: 
  3361: void
  3362: HexagonTargetLowering::ReplaceNodeResults(SDNode *N,
  3363:                                           SmallVectorImpl<SDValue> &Results,
  3364:                                           SelectionDAG &DAG) const {
  3365:   if (isHvxOperation(N, DAG)) {
  3366:     ReplaceHvxNodeResults(N, Results, DAG);
  3367:     if (!Results.empty())
  3368:       return;
  3369:   }
  3370: 
  3371:   const SDLoc &dl(N);
  3372:   switch (N->getOpcode()) {
  3373:     case ISD::SRL:
  3374:     case ISD::SRA:
  3375:     case ISD::SHL:
  3376:       return;
  3377:     case ISD::BITCAST:
  3378:       // Handle a bitcast from v8i1 to i8.
  3379:       if (N->getValueType(0) == MVT::i8) {
  3380:         if (N->getOperand(0).getValueType() == MVT::v8i1) {
  3381:           SDValue P = getInstr(Hexagon::C2_tfrpr, dl, MVT::i32,
  3382:                                N->getOperand(0), DAG);
  3383:           SDValue T = DAG.getAnyExtOrTrunc(P, dl, MVT::i8);
  3384:           Results.push_back(T);
  3385:         }
  3386:       }
  3387:       break;
  3388:   }
  3389: }
  3390: 
  3391: SDValue
  3392: HexagonTargetLowering::PerformDAGCombine(SDNode *N,
  3393:                                          DAGCombinerInfo &DCI) const {
  3394:   SDValue Op(N, 0);
  3395:   const SDLoc &dl(Op);
  3396:   unsigned Opc = Op.getOpcode();
  3397: 
  3398:   // Combining transformations applicable for arbitrary vector sizes.
  3399:   if (DCI.isBeforeLegalizeOps()) {
  3400:     switch (Opc) {
```
- EN: Header guards in this range prevent duplicate inclusion and define the interface boundary. It declares or implements routines such as SDValue, getSetCC, HexagonTargetLowering::LowerUAddSubOCarry, dl, ... (61 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonTargetLowering, HexagonISD, HexagonMachineFunctionInfo, showing how the code connects to sibling backend components.
- CN: 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里声明或实现了 SDValue, getSetCC, HexagonTargetLowering::LowerUAddSubOCarry, dl, ... (61 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonTargetLowering, HexagonISD, HexagonMachineFunctionInfo，说明了它与同级后端组件的连接关系。

### Lines 3401-3600 / 第 3401-3600 行

```cpp
  3401:     case ISD::VECREDUCE_ADD:
  3402:       if (SDValue V = splitVecReduceAdd(N, DCI.DAG))
  3403:         return V;
  3404:       if (SDValue V = expandVecReduceAdd(N, DCI.DAG))
  3405:         return V;
  3406:       return SDValue();
  3407:     case ISD::PARTIAL_REDUCE_SMLA:
  3408:     case ISD::PARTIAL_REDUCE_UMLA:
  3409:     case ISD::PARTIAL_REDUCE_SUMLA:
  3410:       if (SDValue V = splitExtendingPartialReduceMLA(N, DCI.DAG))
  3411:         return V;
  3412:       return SDValue();
  3413:     }
  3414:   } else {
  3415:     switch (Opc) {
  3416:     case ISD::VSELECT: {
  3417:       // (vselect (xor x, ptrue), v0, v1) -> (vselect x, v1, v0)
  3418:       SDValue Cond = Op.getOperand(0);
  3419:       if (Cond->getOpcode() == ISD::XOR) {
  3420:         SDValue C0 = Cond.getOperand(0), C1 = Cond.getOperand(1);
  3421:         if (C1->getOpcode() == HexagonISD::PTRUE) {
  3422:           SDValue VSel = DCI.DAG.getNode(ISD::VSELECT, dl, ty(Op), C0,
  3423:                                          Op.getOperand(2), Op.getOperand(1));
  3424:           return VSel;
  3425:         }
  3426:       }
  3427:       return SDValue();
  3428:     }
  3429:     }
  3430:   }
  3431: 
  3432:   if (isHvxOperation(N, DCI.DAG)) {
  3433:     if (SDValue V = PerformHvxDAGCombine(N, DCI))
  3434:       return V;
  3435:     return SDValue();
  3436:   }
  3437: 
  3438:   if (Opc == ISD::TRUNCATE) {
  3439:     SDValue Op0 = Op.getOperand(0);
  3440:     // fold (truncate (build pair x, y)) -> (truncate x) or x
  3441:     if (Op0.getOpcode() == ISD::BUILD_PAIR) {
  3442:       EVT TruncTy = Op.getValueType();
  3443:       SDValue Elem0 = Op0.getOperand(0);
  3444:       // if we match the low element of the pair, just return it.
  3445:       if (Elem0.getValueType() == TruncTy)
  3446:         return Elem0;
  3447:       // otherwise, if the low part is still too large, apply the truncate.
  3448:       if (Elem0.getValueType().bitsGT(TruncTy))
  3449:         return DCI.DAG.getNode(ISD::TRUNCATE, dl, TruncTy, Elem0);
  3450:     }
  3451:   }
  3452: 
  3453:   if (DCI.isBeforeLegalizeOps())
  3454:     return SDValue();
  3455: 
  3456:   switch (Opc) {
  3457:   case HexagonISD::P2D: {
  3458:     SDValue P = Op.getOperand(0);
  3459:     switch (P.getOpcode()) {
  3460:     case HexagonISD::PTRUE:
  3461:       return DCI.DAG.getAllOnesConstant(dl, ty(Op));
  3462:     case HexagonISD::PFALSE:
  3463:       return getZero(dl, ty(Op), DCI.DAG);
  3464:     default:
  3465:       break;
  3466:     }
  3467:     break;
  3468:   }
  3469:   case ISD::TRUNCATE: {
  3470:     SDValue Op0 = Op.getOperand(0);
  3471:     // fold (truncate (build pair x, y)) -> (truncate x) or x
  3472:     if (Op0.getOpcode() == ISD::BUILD_PAIR) {
  3473:       MVT TruncTy = ty(Op);
  3474:       SDValue Elem0 = Op0.getOperand(0);
  3475:       // if we match the low element of the pair, just return it.
  3476:       if (ty(Elem0) == TruncTy)
  3477:         return Elem0;
  3478:       // otherwise, if the low part is still too large, apply the truncate.
  3479:       if (ty(Elem0).bitsGT(TruncTy))
  3480:         return DCI.DAG.getNode(ISD::TRUNCATE, dl, TruncTy, Elem0);
  3481:     }
  3482:     break;
  3483:   }
  3484:   case ISD::OR: {
  3485:     // fold (or (shl xx, s), (zext y)) -> (COMBINE (shl xx, s-32), y)
  3486:     // if s >= 32
  3487:     auto fold0 = [&, this](SDValue Op) {
  3488:       if (ty(Op) != MVT::i64)
  3489:         return SDValue();
  3490:       SDValue Shl = Op.getOperand(0);
  3491:       SDValue Zxt = Op.getOperand(1);
  3492:       if (Shl.getOpcode() != ISD::SHL)
  3493:         std::swap(Shl, Zxt);
  3494: 
  3495:       if (Shl.getOpcode() != ISD::SHL || Zxt.getOpcode() != ISD::ZERO_EXTEND)
  3496:         return SDValue();
  3497: 
  3498:       SDValue Z = Zxt.getOperand(0);
  3499:       auto *Amt = dyn_cast<ConstantSDNode>(Shl.getOperand(1));
  3500:       if (Amt && Amt->getZExtValue() >= 32 && ty(Z).getSizeInBits() <= 32) {
  3501:         unsigned A = Amt->getZExtValue();
  3502:         SDValue S = Shl.getOperand(0);
  3503:         SDValue T0 = DCI.DAG.getNode(ISD::SHL, dl, ty(S), S,
  3504:                                      DCI.DAG.getConstant(A - 32, dl, MVT::i32));
  3505:         SDValue T1 = DCI.DAG.getZExtOrTrunc(T0, dl, MVT::i32);
  3506:         SDValue T2 = DCI.DAG.getZExtOrTrunc(Z, dl, MVT::i32);
  3507:         return DCI.DAG.getNode(HexagonISD::COMBINE, dl, MVT::i64, {T1, T2});
  3508:       }
  3509:       return SDValue();
  3510:     };
  3511: 
  3512:     if (SDValue R = fold0(Op))
  3513:       return R;
  3514:     break;
  3515:   }
  3516:   }
  3517: 
  3518:   return SDValue();
  3519: }
  3520: 
  3521: /// Returns relocation base for the given PIC jumptable.
  3522: SDValue
  3523: HexagonTargetLowering::getPICJumpTableRelocBase(SDValue Table,
  3524:                                                 SelectionDAG &DAG) const {
  3525:   int Idx = cast<JumpTableSDNode>(Table)->getIndex();
  3526:   EVT VT = Table.getValueType();
  3527:   SDValue T = DAG.getTargetJumpTable(Idx, VT, HexagonII::MO_PCREL);
  3528:   return DAG.getNode(HexagonISD::AT_PCREL, SDLoc(Table), VT, T);
  3529: }
  3530: 
  3531: //===----------------------------------------------------------------------===//
  3532: // Inline Assembly Support
  3533: //===----------------------------------------------------------------------===//
  3534: 
  3535: TargetLowering::ConstraintType
  3536: HexagonTargetLowering::getConstraintType(StringRef Constraint) const {
  3537:   if (Constraint.size() == 1) {
  3538:     switch (Constraint[0]) {
  3539:       case 'q':
  3540:       case 'v':
  3541:         if (Subtarget.useHVXOps())
  3542:           return C_RegisterClass;
  3543:         break;
  3544:       case 'a':
  3545:         return C_RegisterClass;
  3546:       default:
  3547:         break;
  3548:     }
  3549:   }
  3550:   return TargetLowering::getConstraintType(Constraint);
  3551: }
  3552: 
  3553: std::pair<unsigned, const TargetRegisterClass*>
  3554: HexagonTargetLowering::getRegForInlineAsmConstraint(
  3555:     const TargetRegisterInfo *TRI, StringRef Constraint, MVT VT) const {
  3556: 
  3557:   if (Constraint.size() == 1) {
  3558:     switch (Constraint[0]) {
  3559:     case 'r':   // R0-R31
  3560:       switch (VT.SimpleTy) {
  3561:       default:
  3562:         return {0u, nullptr};
  3563:       case MVT::i1:
  3564:       case MVT::i8:
  3565:       case MVT::i16:
  3566:       case MVT::i32:
  3567:       case MVT::f32:
  3568:         return {0u, &Hexagon::IntRegsRegClass};
  3569:       case MVT::i64:
  3570:       case MVT::f64:
  3571:         return {0u, &Hexagon::DoubleRegsRegClass};
  3572:       }
  3573:       break;
  3574:     case 'a': // M0-M1
  3575:       if (VT != MVT::i32)
  3576:         return {0u, nullptr};
  3577:       return {0u, &Hexagon::ModRegsRegClass};
  3578:     case 'q': // q0-q3
  3579:       switch (VT.getSizeInBits()) {
  3580:       default:
  3581:         return {0u, nullptr};
  3582:       case 64:
  3583:       case 128:
  3584:         return {0u, &Hexagon::HvxQRRegClass};
  3585:       }
  3586:       break;
  3587:     case 'v': // V0-V31
  3588:       switch (VT.getSizeInBits()) {
  3589:       default:
  3590:         return {0u, nullptr};
  3591:       case 512:
  3592:         return {0u, &Hexagon::HvxVRRegClass};
  3593:       case 1024:
  3594:         if (Subtarget.hasV60Ops() && Subtarget.useHVX128BOps())
  3595:           return {0u, &Hexagon::HvxVRRegClass};
  3596:         return {0u, &Hexagon::HvxWRRegClass};
  3597:       case 2048:
  3598:         return {0u, &Hexagon::HvxWRRegClass};
  3599:       }
  3600:       break;
```
- EN: It declares or implements routines such as SDValue, vselect, getOperand, getNode, ... (18 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonISD, HexagonTargetLowering, HexagonII, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 SDValue, vselect, getOperand, getNode, ... (18 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonISD, HexagonTargetLowering, HexagonII，说明了它与同级后端组件的连接关系。

### Lines 3601-3800 / 第 3601-3800 行

```cpp
  3601:     default:
  3602:       return {0u, nullptr};
  3603:     }
  3604:   }
  3605: 
  3606:   return TargetLowering::getRegForInlineAsmConstraint(TRI, Constraint, VT);
  3607: }
  3608: 
  3609: /// isFPImmLegal - Returns true if the target can instruction select the
  3610: /// specified FP immediate natively. If false, the legalizer will
  3611: /// materialize the FP immediate as a load from a constant pool.
  3612: bool HexagonTargetLowering::isFPImmLegal(const APFloat &Imm, EVT VT,
  3613:                                          bool ForCodeSize) const {
  3614:   return true;
  3615: }
  3616: 
  3617: /// Returns true if it is beneficial to convert a load of a constant
  3618: /// to just the constant itself.
  3619: bool HexagonTargetLowering::shouldConvertConstantLoadToIntImm(const APInt &Imm,
  3620:                                                               Type *Ty) const {
  3621:   if (!ConstantLoadsToImm)
  3622:     return false;
  3623: 
  3624:   assert(Ty->isIntegerTy());
  3625:   unsigned BitSize = Ty->getPrimitiveSizeInBits();
  3626:   return (BitSize > 0 && BitSize <= 64);
  3627: }
  3628: 
  3629: /// isLegalAddressingMode - Return true if the addressing mode represented by
  3630: /// AM is legal for this target, for a load/store of the specified type.
  3631: bool HexagonTargetLowering::isLegalAddressingMode(const DataLayout &DL,
  3632:                                                   const AddrMode &AM, Type *Ty,
  3633:                                                   unsigned AS, Instruction *I) const {
  3634:   if (Ty->isSized()) {
  3635:     // When LSR detects uses of the same base address to access different
  3636:     // types (e.g. unions), it will assume a conservative type for these
  3637:     // uses:
  3638:     //   LSR Use: Kind=Address of void in addrspace(4294967295), ...
  3639:     // The type Ty passed here would then be "void". Skip the alignment
  3640:     // checks, but do not return false right away, since that confuses
  3641:     // LSR into crashing.
  3642:     Align A = DL.getABITypeAlign(Ty);
  3643:     // The base offset must be a multiple of the alignment.
  3644:     if (!isAligned(A, AM.BaseOffs))
  3645:       return false;
  3646:     // The shifted offset must fit in 11 bits.
  3647:     if (!isInt<11>(AM.BaseOffs >> Log2(A)))
  3648:       return false;
  3649:   }
  3650: 
  3651:   // No global is ever allowed as a base.
  3652:   if (AM.BaseGV)
  3653:     return false;
  3654: 
  3655:   int Scale = AM.Scale;
  3656:   if (Scale < 0)
  3657:     Scale = -Scale;
  3658:   switch (Scale) {
  3659:   case 0:  // No scale reg, "r+i", "r", or just "i".
  3660:     break;
  3661:   default: // No scaled addressing mode.
  3662:     return false;
  3663:   }
  3664:   return true;
  3665: }
  3666: 
  3667: /// Return true if folding a constant offset with the given GlobalAddress is
  3668: /// legal.  It is frequently not legal in PIC relocation models.
  3669: bool HexagonTargetLowering::isOffsetFoldingLegal(const GlobalAddressSDNode *GA)
  3670:       const {
  3671:   return HTM.getRelocationModel() == Reloc::Static;
  3672: }
  3673: 
  3674: /// isLegalICmpImmediate - Return true if the specified immediate is legal
  3675: /// icmp immediate, that is the target has icmp instructions which can compare
  3676: /// a register against the immediate without having to materialize the
  3677: /// immediate into a register.
  3678: bool HexagonTargetLowering::isLegalICmpImmediate(int64_t Imm) const {
  3679:   return Imm >= -512 && Imm <= 511;
  3680: }
  3681: 
  3682: /// IsEligibleForTailCallOptimization - Check whether the call is eligible
  3683: /// for tail call optimization. Targets which want to do tail call
  3684: /// optimization should implement this function.
  3685: bool HexagonTargetLowering::IsEligibleForTailCallOptimization(
  3686:                                  SDValue Callee,
  3687:                                  CallingConv::ID CalleeCC,
  3688:                                  bool IsVarArg,
  3689:                                  bool IsCalleeStructRet,
  3690:                                  bool IsCallerStructRet,
  3691:                                  const SmallVectorImpl<ISD::OutputArg> &Outs,
  3692:                                  const SmallVectorImpl<SDValue> &OutVals,
  3693:                                  const SmallVectorImpl<ISD::InputArg> &Ins,
  3694:                                  SelectionDAG& DAG) const {
  3695:   const Function &CallerF = DAG.getMachineFunction().getFunction();
  3696:   CallingConv::ID CallerCC = CallerF.getCallingConv();
  3697:   bool CCMatch = CallerCC == CalleeCC;
  3698: 
  3699:   // ***************************************************************************
  3700:   //  Look for obvious safe cases to perform tail call optimization that do not
  3701:   //  require ABI changes.
  3702:   // ***************************************************************************
  3703: 
  3704:   // If this is a tail call via a function pointer, then don't do it!
  3705:   if (!isa<GlobalAddressSDNode>(Callee) &&
  3706:       !isa<ExternalSymbolSDNode>(Callee)) {
  3707:     return false;
  3708:   }
  3709: 
  3710:   // Do not optimize if the calling conventions do not match and the conventions
  3711:   // used are not C or Fast.
  3712:   if (!CCMatch) {
  3713:     bool R = (CallerCC == CallingConv::C || CallerCC == CallingConv::Fast);
  3714:     bool E = (CalleeCC == CallingConv::C || CalleeCC == CallingConv::Fast);
  3715:     // If R & E, then ok.
  3716:     if (!R || !E)
  3717:       return false;
  3718:   }
  3719: 
  3720:   // Do not tail call optimize vararg calls.
  3721:   if (IsVarArg)
  3722:     return false;
  3723: 
  3724:   // Also avoid tail call optimization if either caller or callee uses struct
  3725:   // return semantics.
  3726:   if (IsCalleeStructRet || IsCallerStructRet)
  3727:     return false;
  3728: 
  3729:   // In addition to the cases above, we also disable Tail Call Optimization if
  3730:   // the calling convention code that at least one outgoing argument needs to
  3731:   // go on the stack. We cannot check that here because at this point that
  3732:   // information is not available.
  3733:   return true;
  3734: }
  3735: 
  3736: /// Returns the target specific optimal type for load and store operations as
  3737: /// a result of memset, memcpy, and memmove lowering.
  3738: ///
  3739: /// If DstAlign is zero that means it's safe to destination alignment can
  3740: /// satisfy any constraint. Similarly if SrcAlign is zero it means there isn't
  3741: /// a need to check it against alignment requirement, probably because the
  3742: /// source does not need to be loaded. If 'IsMemset' is true, that means it's
  3743: /// expanding a memset. If 'ZeroMemset' is true, that means it's a memset of
  3744: /// zero. 'MemcpyStrSrc' indicates whether the memcpy source is constant so it
  3745: /// does not need to be loaded.  It returns EVT::Other if the type should be
  3746: /// determined using generic target-independent logic.
  3747: EVT HexagonTargetLowering::getOptimalMemOpType(
  3748:     LLVMContext &Context, const MemOp &Op,
  3749:     const AttributeList &FuncAttributes) const {
  3750:   if (Op.size() >= 8 && Op.isAligned(Align(8)))
  3751:     return MVT::i64;
  3752:   if (Op.size() >= 4 && Op.isAligned(Align(4)))
  3753:     return MVT::i32;
  3754:   if (Op.size() >= 2 && Op.isAligned(Align(2)))
  3755:     return MVT::i16;
  3756:   return MVT::Other;
  3757: }
  3758: 
  3759: // The helpers below are versions of llvm::getShuffleReduction and
  3760: // llvm::getOrderedReduction, adapted to use during DAG passes and simplified as
  3761: // follows:
  3762: // - ICmp and FCmp are not handled;
  3763: // - in every step in getShuffleReduction, the input is split into halves (not
  3764: // pairwise).
  3765: 
  3766: static SDValue getOrderedReduction(SDValue Vec, unsigned Op,
  3767:                                    SelectionDAG &DAG) {
  3768:   assert(Op != Instruction::ICmp && Op != Instruction::FCmp);
  3769: 
  3770:   EVT VT = Vec.getValueType();
  3771:   EVT EltT = VT.getVectorElementType();
  3772:   unsigned VF = VT.getVectorNumElements();
  3773:   assert(VF > 0 &&
  3774:          "Reduction emission only supported for non-zero length vectors!");
  3775: 
  3776:   SDLoc DL(Vec);
  3777:   SDValue Result = DAG.getExtractVectorElt(DL, EltT, Vec, 0);
  3778:   for (unsigned ExtractIdx = 1; ExtractIdx < VF; ++ExtractIdx) {
  3779:     SDValue Ext = DAG.getExtractVectorElt(DL, EltT, Vec, ExtractIdx);
  3780:     Result = DAG.getNode(Op, DL, EltT, {Result, Ext});
  3781:   }
  3782: 
  3783:   return Result;
  3784: }
  3785: 
  3786: static SDValue getShuffleReduction(SDValue Vec, unsigned Op,
  3787:                                    SelectionDAG &DAG) {
  3788:   assert(Op != Instruction::ICmp && Op != Instruction::FCmp);
  3789: 
  3790:   EVT VT = Vec.getValueType();
  3791:   unsigned VF = VT.getVectorNumElements();
  3792:   if (VF == 0)
  3793:     llvm_unreachable("Vector must be non-zero length");
  3794:   // VF is a power of 2 so we can emit the reduction using log2(VF) shuffles
  3795:   // and vector ops, reducing the set of values being computed by half each
  3796:   // round.
  3797:   assert(isPowerOf2_32(VF) &&
  3798:          "Reduction emission only supported for pow2 vectors!");
  3799: 
  3800:   SDLoc DL(Vec);
```
- EN: It declares or implements routines such as TargetLowering::getRegForInlineAsmConstraint, HexagonTargetLowering::isFPImmLegal, HexagonTargetLowering::shouldConvertConstantLoadToIntImm, assert, ... (21 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonTargetLowering, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 TargetLowering::getRegForInlineAsmConstraint, HexagonTargetLowering::isFPImmLegal, HexagonTargetLowering::shouldConvertConstantLoadToIntImm, assert, ... (21 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonTargetLowering，说明了它与同级后端组件的连接关系。

### Lines 3801-4000 / 第 3801-4000 行

```cpp
  3801:   // TODO: Is it correct to create double-vector shuffle and fill 3/4 of it with
  3802:   // undefs?
  3803:   SmallVector<int, 32> ShuffleMask(VF);
  3804:   for (unsigned i = VF; i > 1; i >>= 1) {
  3805:     // Move the upper half of the vector to the lower half.
  3806:     for (unsigned j = 0; j != i / 2; ++j)
  3807:       ShuffleMask[j] = i / 2 + j;
  3808:     // Fill the rest of the mask with undef.
  3809:     std::fill(&ShuffleMask[i / 2], ShuffleMask.end(), -1);
  3810: 
  3811:     SDValue Shuf =
  3812:         DAG.getVectorShuffle(VT, DL, Vec, DAG.getUNDEF(VT), ShuffleMask);
  3813: 
  3814:     Vec = DAG.getNode(Op, DL, VT, {Vec, Shuf});
  3815:   }
  3816:   // The result is in the first element of the vector.
  3817:   return DAG.getExtractVectorElt(DL, VT.getVectorElementType(), Vec, 0);
  3818: }
  3819: 
  3820: SDValue HexagonTargetLowering::expandVecReduceAdd(SDNode *N,
  3821:                                                   SelectionDAG &DAG) const {
  3822:   // Since we disabled automatic reduction expansion, generate log2 ladder code
  3823:   // if the vector is of a power-of-two length.
  3824:   SDValue Input = N->getOperand(0);
  3825:   if (isPowerOf2_32(Input.getValueType().getVectorNumElements()))
  3826:     return getShuffleReduction(Input, ISD::ADD, DAG);
  3827:   // Otherwise, reduction will be scalarized.
  3828:   return getOrderedReduction(Input, ISD::ADD, DAG);
  3829: }
  3830: 
  3831: bool HexagonTargetLowering::allowsMemoryAccess(
  3832:     LLVMContext &Context, const DataLayout &DL, EVT VT, unsigned AddrSpace,
  3833:     Align Alignment, MachineMemOperand::Flags Flags, unsigned *Fast) const {
  3834:   if (!VT.isSimple())
  3835:     return false;
  3836:   MVT SVT = VT.getSimpleVT();
  3837:   if (Subtarget.isHVXVectorType(SVT, true))
  3838:     return allowsHvxMemoryAccess(SVT, Flags, Fast);
  3839:   return TargetLoweringBase::allowsMemoryAccess(
  3840:               Context, DL, VT, AddrSpace, Alignment, Flags, Fast);
  3841: }
  3842: 
  3843: bool HexagonTargetLowering::allowsMisalignedMemoryAccesses(
  3844:     EVT VT, unsigned AddrSpace, Align Alignment, MachineMemOperand::Flags Flags,
  3845:     unsigned *Fast) const {
  3846:   if (!VT.isSimple())
  3847:     return false;
  3848:   MVT SVT = VT.getSimpleVT();
  3849:   if (Subtarget.isHVXVectorType(SVT, true))
  3850:     return allowsHvxMisalignedMemoryAccesses(SVT, Flags, Fast);
  3851:   if (Fast)
  3852:     *Fast = 0;
  3853:   return false;
  3854: }
  3855: 
  3856: std::pair<const TargetRegisterClass*, uint8_t>
  3857: HexagonTargetLowering::findRepresentativeClass(const TargetRegisterInfo *TRI,
  3858:       MVT VT) const {
  3859:   if (Subtarget.isHVXVectorType(VT, true)) {
  3860:     unsigned BitWidth = VT.getSizeInBits();
  3861:     unsigned VecWidth = Subtarget.getVectorLength() * 8;
  3862: 
  3863:     if (VT.getVectorElementType() == MVT::i1)
  3864:       return std::make_pair(&Hexagon::HvxQRRegClass, 1);
  3865:     if (BitWidth == VecWidth)
  3866:       return std::make_pair(&Hexagon::HvxVRRegClass, 1);
  3867:     assert(BitWidth == 2 * VecWidth);
  3868:     return std::make_pair(&Hexagon::HvxWRRegClass, 1);
  3869:   }
  3870: 
  3871:   return TargetLowering::findRepresentativeClass(TRI, VT);
  3872: }
  3873: 
  3874: bool HexagonTargetLowering::shouldReduceLoadWidth(
  3875:     SDNode *Load, ISD::LoadExtType ExtTy, EVT NewVT,
  3876:     std::optional<unsigned> ByteOffset) const {
  3877:   // TODO: This may be worth removing. Check regression tests for diffs.
  3878:   if (!TargetLoweringBase::shouldReduceLoadWidth(Load, ExtTy, NewVT,
  3879:                                                  ByteOffset))
  3880:     return false;
  3881: 
  3882:   auto *L = cast<LoadSDNode>(Load);
  3883:   std::pair<SDValue, int> BO = getBaseAndOffset(L->getBasePtr());
  3884:   // Small-data object, do not shrink.
  3885:   if (BO.first.getOpcode() == HexagonISD::CONST32_GP)
  3886:     return false;
  3887:   if (GlobalAddressSDNode *GA = dyn_cast<GlobalAddressSDNode>(BO.first)) {
  3888:     auto &HTM = static_cast<const HexagonTargetMachine &>(getTargetMachine());
  3889:     const auto *GO = dyn_cast_or_null<const GlobalObject>(GA->getGlobal());
  3890:     return !GO || !HTM.getObjFileLowering()->isGlobalInSmallSection(GO, HTM);
  3891:   }
  3892:   return true;
  3893: }
  3894: 
  3895: void HexagonTargetLowering::AdjustInstrPostInstrSelection(MachineInstr &MI,
  3896:       SDNode *Node) const {
  3897:   AdjustHvxInstrPostInstrSelection(MI, Node);
  3898: }
  3899: 
  3900: Value *HexagonTargetLowering::emitLoadLinked(IRBuilderBase &Builder,
  3901:                                              Type *ValueTy, Value *Addr,
  3902:                                              AtomicOrdering Ord) const {
  3903:   unsigned SZ = ValueTy->getPrimitiveSizeInBits();
  3904:   assert((SZ == 32 || SZ == 64) && "Only 32/64-bit atomic loads supported");
  3905:   Intrinsic::ID IntID = (SZ == 32) ? Intrinsic::hexagon_L2_loadw_locked
  3906:                                    : Intrinsic::hexagon_L4_loadd_locked;
  3907: 
  3908:   Value *Call =
  3909:       Builder.CreateIntrinsic(IntID, Addr, /*FMFSource=*/nullptr, "larx");
  3910: 
  3911:   return Builder.CreateBitCast(Call, ValueTy);
  3912: }
  3913: 
  3914: /// Perform a store-conditional operation to Addr. Return the status of the
  3915: /// store. This should be 0 if the store succeeded, non-zero otherwise.
  3916: Value *HexagonTargetLowering::emitStoreConditional(IRBuilderBase &Builder,
  3917:                                                    Value *Val, Value *Addr,
  3918:                                                    AtomicOrdering Ord) const {
  3919:   BasicBlock *BB = Builder.GetInsertBlock();
  3920:   Module *M = BB->getParent()->getParent();
  3921:   Type *Ty = Val->getType();
  3922:   unsigned SZ = Ty->getPrimitiveSizeInBits();
  3923: 
  3924:   Type *CastTy = Builder.getIntNTy(SZ);
  3925:   assert((SZ == 32 || SZ == 64) && "Only 32/64-bit atomic stores supported");
  3926:   Intrinsic::ID IntID = (SZ == 32) ? Intrinsic::hexagon_S2_storew_locked
  3927:                                    : Intrinsic::hexagon_S4_stored_locked;
  3928: 
  3929:   Val = Builder.CreateBitCast(Val, CastTy);
  3930: 
  3931:   Value *Call = Builder.CreateIntrinsic(IntID, {Addr, Val},
  3932:                                         /*FMFSource=*/nullptr, "stcx");
  3933:   Value *Cmp = Builder.CreateICmpEQ(Call, Builder.getInt32(0), "");
  3934:   Value *Ext = Builder.CreateZExt(Cmp, Type::getInt32Ty(M->getContext()));
  3935:   return Ext;
  3936: }
  3937: 
  3938: TargetLowering::AtomicExpansionKind
  3939: HexagonTargetLowering::shouldExpandAtomicLoadInIR(LoadInst *LI) const {
  3940:   // Do not expand loads and stores that don't exceed 64 bits.
  3941:   return LI->getType()->getPrimitiveSizeInBits() > 64
  3942:              ? AtomicExpansionKind::LLOnly
  3943:              : AtomicExpansionKind::None;
  3944: }
  3945: 
  3946: TargetLowering::AtomicExpansionKind
  3947: HexagonTargetLowering::shouldExpandAtomicStoreInIR(StoreInst *SI) const {
  3948:   // Do not expand loads and stores that don't exceed 64 bits.
  3949:   return SI->getValueOperand()->getType()->getPrimitiveSizeInBits() > 64
  3950:              ? AtomicExpansionKind::Expand
  3951:              : AtomicExpansionKind::None;
  3952: }
  3953: 
  3954: TargetLowering::AtomicExpansionKind
  3955: HexagonTargetLowering::shouldExpandAtomicCmpXchgInIR(
  3956:     const AtomicCmpXchgInst *AI) const {
  3957:   return AtomicExpansionKind::LLSC;
  3958: }
  3959: 
  3960: MachineBasicBlock *HexagonTargetLowering::EmitInstrWithCustomInserter(
  3961:     MachineInstr &MI, MachineBasicBlock *BB) const {
  3962:   switch (MI.getOpcode()) {
  3963:   case TargetOpcode::PATCHABLE_EVENT_CALL:
  3964:   case TargetOpcode::PATCHABLE_TYPED_EVENT_CALL:
  3965:     // These are lowered in the AsmPrinter.
  3966:     return BB;
  3967:   default:
  3968:     llvm_unreachable("Unexpected instruction with custom inserter");
  3969:   }
  3970: }
  3971: 
  3972: bool HexagonTargetLowering::isMaskAndCmp0FoldingBeneficial(
  3973:     const Instruction &AndI) const {
  3974:   // Only sink 'and' mask to cmp use block if it is masking a single bit since
  3975:   // this will fold the and/cmp/br into a single tstbit instruction.
  3976:   ConstantInt *Mask = dyn_cast<ConstantInt>(AndI.getOperand(1));
  3977:   if (!Mask)
  3978:     return false;
  3979:   return Mask->getValue().isPowerOf2();
  3980: }
  3981: 
  3982: // Check if the result of the node is only used as a return value, as
  3983: // otherwise we can't perform a tail-call.
  3984: bool HexagonTargetLowering::isUsedByReturnOnly(SDNode *N,
  3985:                                                SDValue &Chain) const {
  3986:   if (N->getNumValues() != 1)
  3987:     return false;
  3988:   if (!N->hasNUsesOfValue(1, 0))
  3989:     return false;
  3990: 
  3991:   SDNode *Copy = *N->user_begin();
  3992: 
  3993:   if (Copy->getOpcode() == ISD::BITCAST) {
  3994:     return isUsedByReturnOnly(Copy, Chain);
  3995:   }
  3996: 
  3997:   if (Copy->getOpcode() != ISD::CopyToReg) {
  3998:     return false;
  3999:   }
  4000: 
```
- EN: It declares or implements routines such as ShuffleMask, std::fill, getVectorShuffle, getExtractVectorElt, ... (46 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonTargetLowering, HexagonISD, HexagonTargetMachine, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 ShuffleMask, std::fill, getVectorShuffle, getExtractVectorElt, ... (46 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonTargetLowering, HexagonISD, HexagonTargetMachine，说明了它与同级后端组件的连接关系。

### Lines 4001-4018 / 第 4001-4018 行

```cpp
  4001:   // If the ISD::CopyToReg has a glue operand, we conservatively assume it
  4002:   // isn't safe to perform a tail call.
  4003:   if (Copy->getOperand(Copy->getNumOperands() - 1).getValueType() == MVT::Glue)
  4004:     return false;
  4005: 
  4006:   // The copy must be used by a HexagonISD::RET_GLUE, and nothing else.
  4007:   bool HasRet = false;
  4008:   for (SDNode *Node : Copy->users()) {
  4009:     if (Node->getOpcode() != HexagonISD::RET_GLUE)
  4010:       return false;
  4011:     HasRet = true;
  4012:   }
  4013:   if (!HasRet)
  4014:     return false;
  4015: 
  4016:   Chain = Copy->getOperand(0);
  4017:   return true;
  4018: }
```
- EN: It declares or implements routines such as getOperand, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonISD, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getOperand 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonISD，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- register modeling / 寄存器建模
- instruction semantics / 指令语义
- assembly/MC integration / 汇编/MC 集成
- MC-layer target description / MC 层目标描述
- target pipeline configuration / 目标流水线配置
- subtarget feature gating / 子目标特性控制

## Dependencies / 依赖关系

- Direct includes / 直接包含: `HexagonISelLowering.h, Hexagon.h, HexagonMachineFunctionInfo.h, HexagonRegisterInfo.h, HexagonSubtarget.h, HexagonTargetMachine.h, HexagonTargetObjectFile.h, llvm/ADT/APInt.h, llvm/ADT/ArrayRef.h, llvm/ADT/SmallVector.h, ... (50 total)`
- Hexagon symbols / Hexagon 符号: `HexagonISelLowering, HexagonMachineFunctionInfo, HexagonRegisterInfo, HexagonSubtarget, HexagonTargetMachine, HexagonTargetObjectFile, HexagonGenCallingConv, HexagonTargetLowering, HexagonISD, HexagonII, ... (12 total)`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
