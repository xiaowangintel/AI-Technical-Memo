# HexagonISelLowering.h — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonISelLowering.h`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): This file defines the interfaces that Hexagon uses to lower LLVM code into a
- Purpose / 作用 (CN): 该文件声明 Hexagon 后端组件的接口、类型或辅助入口。

## Line-by-Line Analysis / 逐行分析

### Lines 1-50 / 第 1-50 行

```cpp
     1: //===-- HexagonISelLowering.h - Hexagon DAG Lowering Interface --*- C++ -*-===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: //
     9: // This file defines the interfaces that Hexagon uses to lower LLVM code into a
    10: // selection DAG.
    11: //
    12: //===----------------------------------------------------------------------===//
    13: 
    14: #ifndef LLVM_LIB_TARGET_HEXAGON_HEXAGONISELLOWERING_H
    15: #define LLVM_LIB_TARGET_HEXAGON_HEXAGONISELLOWERING_H
    16: 
    17: #include "Hexagon.h"
    18: #include "MCTargetDesc/HexagonMCTargetDesc.h"
    19: #include "llvm/ADT/StringRef.h"
    20: #include "llvm/CodeGen/ISDOpcodes.h"
    21: #include "llvm/CodeGen/SelectionDAGNodes.h"
    22: #include "llvm/CodeGen/TargetLowering.h"
    23: #include "llvm/CodeGen/ValueTypes.h"
    24: #include "llvm/CodeGenTypes/MachineValueType.h"
    25: #include "llvm/IR/CallingConv.h"
    26: #include "llvm/IR/InlineAsm.h"
    27: #include <cstdint>
    28: #include <utility>
    29: 
    30: namespace llvm {
    31: 
    32: class HexagonSubtarget;
    33: 
    34: class HexagonTargetLowering : public TargetLowering {
    35:   int VarArgsFrameOffset;   // Frame offset to start of varargs area.
    36:   const HexagonTargetMachine &HTM;
    37:   const HexagonSubtarget &Subtarget;
    38: 
    39: public:
    40:   explicit HexagonTargetLowering(const TargetMachine &TM,
    41:                                  const HexagonSubtarget &ST);
    42: 
    43:   /// IsEligibleForTailCallOptimization - Check whether the call is eligible
    44:   /// for tail call optimization. Targets which want to do tail call
    45:   /// optimization should implement this function.
    46:   bool IsEligibleForTailCallOptimization(SDValue Callee,
    47:       CallingConv::ID CalleeCC, bool isVarArg, bool isCalleeStructRet,
    48:       bool isCallerStructRet, const SmallVectorImpl<ISD::OutputArg> &Outs,
    49:       const SmallVectorImpl<SDValue> &OutVals,
    50:       const SmallVectorImpl<ISD::InputArg> &Ins, SelectionDAG& DAG) const;
```
- EN: It imports headers such as Hexagon.h, MCTargetDesc/HexagonMCTargetDesc.h, llvm/ADT/StringRef.h, llvm/CodeGen/ISDOpcodes.h, ... (12 total), establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as HexagonSubtarget, HexagonTargetLowering, which carry the state or API of this component.
- CN: 这里引入了 Hexagon.h, MCTargetDesc/HexagonMCTargetDesc.h, llvm/ADT/StringRef.h, llvm/CodeGen/ISDOpcodes.h, ... (12 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 HexagonSubtarget, HexagonTargetLowering 等类型，用来承载该组件的状态或接口。

### Lines 51-100 / 第 51-100 行

```cpp
    51: 
    52:   void getTgtMemIntrinsic(SmallVectorImpl<IntrinsicInfo> &Infos,
    53:                           const CallBase &I, MachineFunction &MF,
    54:                           unsigned Intrinsic) const override;
    55: 
    56:   bool isTruncateFree(Type *Ty1, Type *Ty2) const override;
    57:   bool isTruncateFree(EVT VT1, EVT VT2) const override;
    58: 
    59:   bool isCheapToSpeculateCttz(Type *) const override { return true; }
    60:   bool isCheapToSpeculateCtlz(Type *) const override { return true; }
    61:   bool isCtlzFast() const override { return true; }
    62: 
    63:   bool hasBitTest(SDValue X, SDValue Y) const override;
    64: 
    65:   bool allowTruncateForTailCall(Type *Ty1, Type *Ty2) const override;
    66: 
    67:   bool isMaskAndCmp0FoldingBeneficial(const Instruction &AndI) const override;
    68: 
    69:   bool isUsedByReturnOnly(SDNode *N, SDValue &Chain) const override;
    70: 
    71:   /// Return true if an FMA operation is faster than a pair of mul and add
    72:   /// instructions. fmuladd intrinsics will be expanded to FMAs when this
    73:   /// method returns true (and FMAs are legal), otherwise fmuladd is
    74:   /// expanded to mul + add.
    75:   bool isFMAFasterThanFMulAndFAdd(const MachineFunction &,
    76:                                   EVT) const override;
    77: 
    78:   // Should we expand the build vector with shuffles?
    79:   bool shouldExpandBuildVectorWithShuffles(EVT VT,
    80:       unsigned DefinedValues) const override;
    81:   bool isExtractSubvectorCheap(EVT ResVT, EVT SrcVT,
    82:       unsigned Index) const override;
    83: 
    84:   bool isTargetCanonicalConstantNode(SDValue Op) const override;
    85: 
    86:   bool isShuffleMaskLegal(ArrayRef<int> Mask, EVT VT) const override;
    87:   LegalizeTypeAction getPreferredVectorAction(MVT VT) const override;
    88:   LegalizeAction getCustomOperationAction(SDNode &Op) const override;
    89: 
    90:   SDValue LowerOperation(SDValue Op, SelectionDAG &DAG) const override;
    91:   void LowerOperationWrapper(SDNode *N, SmallVectorImpl<SDValue> &Results,
    92:                              SelectionDAG &DAG) const override;
    93:   void ReplaceNodeResults(SDNode *N, SmallVectorImpl<SDValue> &Results,
    94:                           SelectionDAG &DAG) const override;
    95: 
    96:   std::pair<MVT, unsigned>
    97:   handleMaskRegisterForCallingConv(const HexagonSubtarget &Subtarget,
    98:                                    EVT VT) const;
    99: 
   100:   SDValue LowerBUILD_VECTOR(SDValue Op, SelectionDAG &DAG) const;
```
- EN: It declares or implements routines such as getTgtMemIntrinsic, isTruncateFree, isCheapToSpeculateCttz, isCheapToSpeculateCtlz, ... (21 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonSubtarget, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getTgtMemIntrinsic, isTruncateFree, isCheapToSpeculateCttz, isCheapToSpeculateCtlz, ... (21 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonSubtarget，说明了它与同级后端组件的连接关系。

### Lines 101-150 / 第 101-150 行

```cpp
   101:   SDValue LowerCONCAT_VECTORS(SDValue Op, SelectionDAG &DAG) const;
   102:   SDValue LowerEXTRACT_VECTOR_ELT(SDValue Op, SelectionDAG &DAG) const;
   103:   SDValue LowerEXTRACT_SUBVECTOR(SDValue Op, SelectionDAG &DAG) const;
   104:   SDValue LowerINSERT_VECTOR_ELT(SDValue Op, SelectionDAG &DAG) const;
   105:   SDValue LowerINSERT_SUBVECTOR(SDValue Op, SelectionDAG &DAG) const;
   106:   SDValue LowerVECTOR_SHUFFLE(SDValue Op, SelectionDAG &DAG) const;
   107:   SDValue LowerVECTOR_SHIFT(SDValue Op, SelectionDAG &DAG) const;
   108:   SDValue LowerROTL(SDValue Op, SelectionDAG &DAG) const;
   109:   SDValue LowerBITCAST(SDValue Op, SelectionDAG &DAG) const;
   110:   SDValue LowerANY_EXTEND(SDValue Op, SelectionDAG &DAG) const;
   111:   SDValue LowerSIGN_EXTEND(SDValue Op, SelectionDAG &DAG) const;
   112:   SDValue LowerZERO_EXTEND(SDValue Op, SelectionDAG &DAG) const;
   113:   SDValue LowerLoad(SDValue Op, SelectionDAG &DAG) const;
   114:   SDValue LowerStore(SDValue Op, SelectionDAG &DAG) const;
   115:   SDValue LowerUnalignedLoad(SDValue Op, SelectionDAG &DAG) const;
   116:   SDValue LowerUAddSubO(SDValue Op, SelectionDAG &DAG) const;
   117:   SDValue LowerUAddSubOCarry(SDValue Op, SelectionDAG &DAG) const;
   118: 
   119:   SDValue LowerDYNAMIC_STACKALLOC(SDValue Op, SelectionDAG &DAG) const;
   120:   SDValue LowerINLINEASM(SDValue Op, SelectionDAG &DAG) const;
   121:   SDValue LowerFDIV(SDValue Op, SelectionDAG &DAG) const;
   122:   SDValue LowerPREFETCH(SDValue Op, SelectionDAG &DAG) const;
   123:   SDValue LowerEH_LABEL(SDValue Op, SelectionDAG &DAG) const;
   124:   SDValue LowerEH_RETURN(SDValue Op, SelectionDAG &DAG) const;
   125:   SDValue
   126:   LowerFormalArguments(SDValue Chain, CallingConv::ID CallConv, bool isVarArg,
   127:                        const SmallVectorImpl<ISD::InputArg> &Ins,
   128:                        const SDLoc &dl, SelectionDAG &DAG,
   129:                        SmallVectorImpl<SDValue> &InVals) const override;
   130:   SDValue LowerGLOBALADDRESS(SDValue Op, SelectionDAG &DAG) const;
   131:   SDValue LowerBlockAddress(SDValue Op, SelectionDAG &DAG) const;
   132:   SDValue LowerGlobalTLSAddress(SDValue Op, SelectionDAG &DAG) const;
   133:   SDValue LowerToTLSGeneralDynamicModel(GlobalAddressSDNode *GA,
   134:       SelectionDAG &DAG) const;
   135:   SDValue LowerToTLSInitialExecModel(GlobalAddressSDNode *GA,
   136:       SelectionDAG &DAG) const;
   137:   SDValue LowerToTLSLocalExecModel(GlobalAddressSDNode *GA,
   138:       SelectionDAG &DAG) const;
   139:   SDValue GetDynamicTLSAddr(SelectionDAG &DAG, SDValue Chain,
   140:       GlobalAddressSDNode *GA, SDValue InGlue, EVT PtrVT,
   141:       unsigned ReturnReg, unsigned char OperandGlues) const;
   142:   SDValue LowerGLOBAL_OFFSET_TABLE(SDValue Op, SelectionDAG &DAG) const;
   143: 
   144:   SDValue LowerCall(TargetLowering::CallLoweringInfo &CLI,
   145:       SmallVectorImpl<SDValue> &InVals) const override;
   146:   SDValue LowerCallResult(SDValue Chain, SDValue InGlue,
   147:                           CallingConv::ID CallConv, bool isVarArg,
   148:                           const SmallVectorImpl<ISD::InputArg> &Ins,
   149:                           const SDLoc &dl, SelectionDAG &DAG,
   150:                           SmallVectorImpl<SDValue> &InVals,
```
- EN: It declares or implements routines such as LowerCONCAT_VECTORS, LowerEXTRACT_VECTOR_ELT, LowerEXTRACT_SUBVECTOR, LowerINSERT_VECTOR_ELT, ... (33 total), translating Hexagon-specific policy into reusable code paths.
- CN: 这里声明或实现了 LowerCONCAT_VECTORS, LowerEXTRACT_VECTOR_ELT, LowerEXTRACT_SUBVECTOR, LowerINSERT_VECTOR_ELT, ... (33 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

### Lines 151-200 / 第 151-200 行

```cpp
   151:                           const SmallVectorImpl<SDValue> &OutVals,
   152:                           SDValue Callee) const;
   153: 
   154:   SDValue LowerSETCC(SDValue Op, SelectionDAG &DAG) const;
   155:   SDValue LowerVSELECT(SDValue Op, SelectionDAG &DAG) const;
   156:   SDValue LowerFRAMEADDR(SDValue Op, SelectionDAG &DAG) const;
   157:   SDValue LowerATOMIC_FENCE(SDValue Op, SelectionDAG& DAG) const;
   158:   SDValue LowerRETURNADDR(SDValue Op, SelectionDAG &DAG) const;
   159: 
   160:   bool CanLowerReturn(CallingConv::ID CallConv,
   161:                       MachineFunction &MF, bool isVarArg,
   162:                       const SmallVectorImpl<ISD::OutputArg> &Outs,
   163:                       LLVMContext &Context, const Type *RetTy) const override;
   164: 
   165:   SDValue LowerReturn(SDValue Chain, CallingConv::ID CallConv, bool isVarArg,
   166:                       const SmallVectorImpl<ISD::OutputArg> &Outs,
   167:                       const SmallVectorImpl<SDValue> &OutVals,
   168:                       const SDLoc &dl, SelectionDAG &DAG) const override;
   169: 
   170:   SDValue PerformDAGCombine(SDNode *N, DAGCombinerInfo &DCI) const override;
   171: 
   172:   bool mayBeEmittedAsTailCall(const CallInst *CI) const override;
   173: 
   174:   Register getRegisterByName(const char* RegName, LLT VT,
   175:                              const MachineFunction &MF) const override;
   176: 
   177:   unsigned getVectorTypeBreakdownForCallingConv(LLVMContext &Context,
   178:                                                 CallingConv::ID CC, EVT VT,
   179:                                                 EVT &IntermediateVT,
   180:                                                 unsigned &NumIntermediates,
   181:                                                 MVT &RegisterVT) const override;
   182: 
   183:   MVT getRegisterTypeForCallingConv(LLVMContext &Context, CallingConv::ID CC,
   184:                                     EVT VT) const override;
   185:   /// If a physical register, this returns the register that receives the
   186:   /// exception address on entry to an EH pad.
   187:   Register
   188:   getExceptionPointerRegister(const Constant *PersonalityFn) const override {
   189:     return Hexagon::R0;
   190:   }
   191: 
   192:   /// If a physical register, this returns the register that receives the
   193:   /// exception typeid on entry to a landing pad.
   194:   Register
   195:   getExceptionSelectorRegister(const Constant *PersonalityFn) const override {
   196:     return Hexagon::R1;
   197:   }
   198: 
   199:   SDValue LowerVASTART(SDValue Op, SelectionDAG &DAG) const;
   200:   SDValue LowerVACOPY(SDValue Op, SelectionDAG &DAG) const;
```
- EN: It declares or implements routines such as LowerSETCC, LowerVSELECT, LowerFRAMEADDR, LowerATOMIC_FENCE, ... (16 total), translating Hexagon-specific policy into reusable code paths.
- CN: 这里声明或实现了 LowerSETCC, LowerVSELECT, LowerFRAMEADDR, LowerATOMIC_FENCE, ... (16 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

### Lines 201-250 / 第 201-250 行

```cpp
   201:   SDValue LowerConstantPool(SDValue Op, SelectionDAG &DAG) const;
   202:   SDValue LowerJumpTable(SDValue Op, SelectionDAG &DAG) const;
   203: 
   204:   EVT getSetCCResultType(const DataLayout &, LLVMContext &C,
   205:                          EVT VT) const override {
   206:     if (!VT.isVector())
   207:       return MVT::i1;
   208:     else
   209:       return EVT::getVectorVT(C, MVT::i1, VT.getVectorNumElements());
   210:   }
   211: 
   212:   bool getPostIndexedAddressParts(SDNode *N, SDNode *Op,
   213:                                   SDValue &Base, SDValue &Offset,
   214:                                   ISD::MemIndexedMode &AM,
   215:                                   SelectionDAG &DAG) const override;
   216: 
   217:   ConstraintType getConstraintType(StringRef Constraint) const override;
   218: 
   219:   std::pair<unsigned, const TargetRegisterClass *>
   220:   getRegForInlineAsmConstraint(const TargetRegisterInfo *TRI,
   221:                                StringRef Constraint, MVT VT) const override;
   222: 
   223:   // Intrinsics
   224:   SDValue LowerINTRINSIC_WO_CHAIN(SDValue Op, SelectionDAG &DAG) const;
   225:   SDValue LowerINTRINSIC_VOID(SDValue Op, SelectionDAG &DAG) const;
   226:   /// isLegalAddressingMode - Return true if the addressing mode represented
   227:   /// by AM is legal for this target, for a load/store of the specified type.
   228:   /// The type may be VoidTy, in which case only return true if the addressing
   229:   /// mode is legal for a load/store of any legal type.
   230:   /// TODO: Handle pre/postinc as well.
   231:   bool isLegalAddressingMode(const DataLayout &DL, const AddrMode &AM,
   232:                              Type *Ty, unsigned AS,
   233:                              Instruction *I = nullptr) const override;
   234:   /// Return true if folding a constant offset with the given GlobalAddress
   235:   /// is legal.  It is frequently not legal in PIC relocation models.
   236:   bool isOffsetFoldingLegal(const GlobalAddressSDNode *GA) const override;
   237: 
   238:   bool isFPImmLegal(const APFloat &Imm, EVT VT,
   239:                     bool ForCodeSize) const override;
   240: 
   241:   /// isLegalICmpImmediate - Return true if the specified immediate is legal
   242:   /// icmp immediate, that is the target has icmp instructions which can
   243:   /// compare a register against the immediate without having to materialize
   244:   /// the immediate into a register.
   245:   bool isLegalICmpImmediate(int64_t Imm) const override;
   246: 
   247:   EVT getOptimalMemOpType(LLVMContext &Context, const MemOp &Op,
   248:                           const AttributeList &FuncAttributes) const override;
   249: 
   250:   bool allowsMemoryAccess(LLVMContext &Context, const DataLayout &DL, EVT VT,
```
- EN: It declares or implements routines such as LowerConstantPool, LowerJumpTable, getSetCCResultType, EVT::getVectorVT, ... (14 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明或实现了 LowerConstantPool, LowerJumpTable, getSetCCResultType, EVT::getVectorVT, ... (14 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 251-300 / 第 251-300 行

```cpp
   251:                           unsigned AddrSpace, Align Alignment,
   252:                           MachineMemOperand::Flags Flags,
   253:                           unsigned *Fast) const override;
   254: 
   255:   bool allowsMisalignedMemoryAccesses(EVT VT, unsigned AddrSpace,
   256:                                       Align Alignment,
   257:                                       MachineMemOperand::Flags Flags,
   258:                                       unsigned *Fast) const override;
   259: 
   260:   /// Returns relocation base for the given PIC jumptable.
   261:   SDValue getPICJumpTableRelocBase(SDValue Table, SelectionDAG &DAG)
   262:                                    const override;
   263: 
   264:   /// Returns true if it is beneficial to convert a load of a constant
   265:   /// to just the constant itself.
   266:   bool shouldConvertConstantLoadToIntImm(const APInt &Imm,
   267:                                          Type *Ty) const override;
   268: 
   269:   bool shouldReduceLoadWidth(SDNode *Load, ISD::LoadExtType ExtTy, EVT NewVT,
   270:                              std::optional<unsigned> ByteOffset) const override;
   271: 
   272:   void AdjustInstrPostInstrSelection(MachineInstr &MI,
   273:                                      SDNode *Node) const override;
   274: 
   275:   // Handling of atomic RMW instructions.
   276:   Value *emitLoadLinked(IRBuilderBase &Builder, Type *ValueTy, Value *Addr,
   277:                         AtomicOrdering Ord) const override;
   278:   Value *emitStoreConditional(IRBuilderBase &Builder, Value *Val, Value *Addr,
   279:                               AtomicOrdering Ord) const override;
   280:   AtomicExpansionKind shouldExpandAtomicLoadInIR(LoadInst *LI) const override;
   281:   AtomicExpansionKind shouldExpandAtomicStoreInIR(StoreInst *SI) const override;
   282:   AtomicExpansionKind
   283:   shouldExpandAtomicCmpXchgInIR(const AtomicCmpXchgInst *AI) const override;
   284: 
   285:   AtomicExpansionKind
   286:   shouldExpandAtomicRMWInIR(const AtomicRMWInst *AI) const override {
   287:     return AtomicExpansionKind::LLSC;
   288:   }
   289: 
   290:   MachineBasicBlock *
   291:   EmitInstrWithCustomInserter(MachineInstr &MI,
   292:                               MachineBasicBlock *BB) const override;
   293: 
   294: private:
   295:   void initializeHVXLowering();
   296:   unsigned getPreferredHvxVectorAction(MVT VecTy) const;
   297:   unsigned getCustomHvxOperationAction(SDNode &Op) const;
   298: 
   299:   bool validateConstPtrAlignment(SDValue Ptr, Align NeedAlign, const SDLoc &dl,
   300:                                  SelectionDAG &DAG) const;
```
- EN: It declares or implements routines such as allowsMisalignedMemoryAccesses, getPICJumpTableRelocBase, shouldConvertConstantLoadToIntImm, shouldReduceLoadWidth, ... (16 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明或实现了 allowsMisalignedMemoryAccesses, getPICJumpTableRelocBase, shouldConvertConstantLoadToIntImm, shouldReduceLoadWidth, ... (16 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 301-350 / 第 301-350 行

```cpp
   301:   SDValue replaceMemWithUndef(SDValue Op, SelectionDAG &DAG) const;
   302: 
   303:   std::pair<SDValue,int> getBaseAndOffset(SDValue Addr) const;
   304: 
   305:   bool getBuildVectorConstInts(ArrayRef<SDValue> Values, MVT VecTy,
   306:                                SelectionDAG &DAG,
   307:                                MutableArrayRef<ConstantInt*> Consts) const;
   308:   SDValue buildVector32(ArrayRef<SDValue> Elem, const SDLoc &dl, MVT VecTy,
   309:                         SelectionDAG &DAG) const;
   310:   SDValue buildVector64(ArrayRef<SDValue> Elem, const SDLoc &dl, MVT VecTy,
   311:                         SelectionDAG &DAG) const;
   312:   SDValue extractVector(SDValue VecV, SDValue IdxV, const SDLoc &dl,
   313:                         MVT ValTy, MVT ResTy, SelectionDAG &DAG) const;
   314:   SDValue extractVectorPred(SDValue VecV, SDValue IdxV, const SDLoc &dl,
   315:                             MVT ValTy, MVT ResTy, SelectionDAG &DAG) const;
   316:   SDValue insertVector(SDValue VecV, SDValue ValV, SDValue IdxV,
   317:                        const SDLoc &dl, MVT ValTy, SelectionDAG &DAG) const;
   318:   SDValue insertVectorPred(SDValue VecV, SDValue ValV, SDValue IdxV,
   319:                            const SDLoc &dl, MVT ValTy, SelectionDAG &DAG) const;
   320:   SDValue expandPredicate(SDValue Vec32, const SDLoc &dl,
   321:                           SelectionDAG &DAG) const;
   322:   SDValue contractPredicate(SDValue Vec64, const SDLoc &dl,
   323:                             SelectionDAG &DAG) const;
   324:   SDValue getSplatValue(SDValue Op, SelectionDAG &DAG) const;
   325:   SDValue getVectorShiftByInt(SDValue Op, SelectionDAG &DAG) const;
   326:   SDValue appendUndef(SDValue Val, MVT ResTy, SelectionDAG &DAG) const;
   327:   SDValue getCombine(SDValue Hi, SDValue Lo, const SDLoc &dl, MVT ResTy,
   328:                      SelectionDAG &DAG) const;
   329: 
   330:   bool isUndef(SDValue Op) const {
   331:     if (Op.isMachineOpcode())
   332:       return Op.getMachineOpcode() == TargetOpcode::IMPLICIT_DEF;
   333:     return Op.getOpcode() == ISD::UNDEF;
   334:   }
   335:   SDValue getInstr(unsigned MachineOpc, const SDLoc &dl, MVT Ty,
   336:                    ArrayRef<SDValue> Ops, SelectionDAG &DAG) const {
   337:     SDNode *N = DAG.getMachineNode(MachineOpc, dl, Ty, Ops);
   338:     return SDValue(N, 0);
   339:   }
   340:   SDValue getZero(const SDLoc &dl, MVT Ty, SelectionDAG &DAG) const;
   341: 
   342:   using VectorPair = std::pair<SDValue, SDValue>;
   343:   using TypePair = std::pair<MVT, MVT>;
   344: 
   345:   SDValue getInt(unsigned IntId, MVT ResTy, ArrayRef<SDValue> Ops,
   346:                  const SDLoc &dl, SelectionDAG &DAG) const;
   347: 
   348:   MVT ty(SDValue Op) const {
   349:     return Op.getValueType().getSimpleVT();
   350:   }
```
- EN: It declares or implements routines such as replaceMemWithUndef, getBaseAndOffset, getBuildVectorConstInts, buildVector32, ... (23 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明或实现了 replaceMemWithUndef, getBaseAndOffset, getBuildVectorConstInts, buildVector32, ... (23 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 351-400 / 第 351-400 行

```cpp
   351:   TypePair ty(const VectorPair &Ops) const {
   352:     return { Ops.first.getValueType().getSimpleVT(),
   353:              Ops.second.getValueType().getSimpleVT() };
   354:   }
   355:   MVT tyScalar(MVT Ty) const {
   356:     if (!Ty.isVector())
   357:       return Ty;
   358:     return MVT::getIntegerVT(Ty.getSizeInBits());
   359:   }
   360:   MVT tyVector(MVT Ty, MVT ElemTy) const {
   361:     if (Ty.isVector() && Ty.getVectorElementType() == ElemTy)
   362:       return Ty;
   363:     unsigned TyWidth = Ty.getSizeInBits();
   364:     unsigned ElemWidth = ElemTy.getSizeInBits();
   365:     assert((TyWidth % ElemWidth) == 0);
   366:     return MVT::getVectorVT(ElemTy, TyWidth/ElemWidth);
   367:   }
   368: 
   369:   MVT typeJoin(const TypePair &Tys) const;
   370:   TypePair typeSplit(MVT Ty) const;
   371:   MVT typeExtElem(MVT VecTy, unsigned Factor) const;
   372:   MVT typeTruncElem(MVT VecTy, unsigned Factor) const;
   373:   TypePair typeExtendToWider(MVT Ty0, MVT Ty1) const;
   374:   TypePair typeWidenToWider(MVT Ty0, MVT Ty1) const;
   375:   MVT typeLegalize(MVT Ty, SelectionDAG &DAG) const;
   376:   MVT typeWidenToHvx(MVT Ty) const;
   377: 
   378:   SDValue opJoin(const VectorPair &Ops, const SDLoc &dl,
   379:                  SelectionDAG &DAG) const;
   380:   VectorPair opSplit(SDValue Vec, const SDLoc &dl, SelectionDAG &DAG) const;
   381:   SDValue opCastElem(SDValue Vec, MVT ElemTy, SelectionDAG &DAG) const;
   382: 
   383:   SDValue LoHalf(SDValue V, SelectionDAG &DAG) const {
   384:     MVT Ty = ty(V);
   385:     const SDLoc &dl(V);
   386:     if (!Ty.isVector()) {
   387:       assert(Ty.getSizeInBits() == 64);
   388:       return DAG.getTargetExtractSubreg(Hexagon::isub_lo, dl, MVT::i32, V);
   389:     }
   390:     MVT HalfTy = typeSplit(Ty).first;
   391:     SDValue Idx = getZero(dl, MVT::i32, DAG);
   392:     return DAG.getNode(ISD::EXTRACT_SUBVECTOR, dl, HalfTy, V, Idx);
   393:   }
   394:   SDValue HiHalf(SDValue V, SelectionDAG &DAG) const {
   395:     MVT Ty = ty(V);
   396:     const SDLoc &dl(V);
   397:     if (!Ty.isVector()) {
   398:       assert(Ty.getSizeInBits() == 64);
   399:       return DAG.getTargetExtractSubreg(Hexagon::isub_hi, dl, MVT::i32, V);
   400:     }
```
- EN: It declares or implements routines such as ty, tyScalar, MVT::getIntegerVT, tyVector, ... (24 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明或实现了 ty, tyScalar, MVT::getIntegerVT, tyVector, ... (24 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 401-450 / 第 401-450 行

```cpp
   401:     MVT HalfTy = typeSplit(Ty).first;
   402:     SDValue Idx = DAG.getConstant(HalfTy.getVectorNumElements(), dl, MVT::i32);
   403:     return DAG.getNode(ISD::EXTRACT_SUBVECTOR, dl, HalfTy, V, Idx);
   404:   }
   405: 
   406:   bool allowsHvxMemoryAccess(MVT VecTy, MachineMemOperand::Flags Flags,
   407:                              unsigned *Fast) const;
   408:   bool allowsHvxMisalignedMemoryAccesses(MVT VecTy,
   409:                                          MachineMemOperand::Flags Flags,
   410:                                          unsigned *Fast) const;
   411:   void AdjustHvxInstrPostInstrSelection(MachineInstr &MI, SDNode *Node) const;
   412: 
   413:   bool isHvxSingleTy(MVT Ty) const;
   414:   bool isHvxPairTy(MVT Ty) const;
   415:   bool isHvxBoolTy(MVT Ty) const;
   416:   SDValue convertToByteIndex(SDValue ElemIdx, MVT ElemTy,
   417:                              SelectionDAG &DAG) const;
   418:   SDValue getIndexInWord32(SDValue Idx, MVT ElemTy, SelectionDAG &DAG) const;
   419:   SDValue getByteShuffle(const SDLoc &dl, SDValue Op0, SDValue Op1,
   420:                          ArrayRef<int> Mask, SelectionDAG &DAG) const;
   421: 
   422:   SDValue buildHvxVectorReg(ArrayRef<SDValue> Values, const SDLoc &dl,
   423:                             MVT VecTy, SelectionDAG &DAG) const;
   424:   SDValue buildHvxVectorPred(ArrayRef<SDValue> Values, const SDLoc &dl,
   425:                              MVT VecTy, SelectionDAG &DAG) const;
   426:   SDValue createHvxPrefixPred(SDValue PredV, const SDLoc &dl,
   427:                               unsigned BitBytes, bool ZeroFill,
   428:                               SelectionDAG &DAG) const;
   429:   SDValue extractHvxElementReg(SDValue VecV, SDValue IdxV, const SDLoc &dl,
   430:                                MVT ResTy, SelectionDAG &DAG) const;
   431:   SDValue extractHvxElementPred(SDValue VecV, SDValue IdxV, const SDLoc &dl,
   432:                                 MVT ResTy, SelectionDAG &DAG) const;
   433:   SDValue insertHvxElementReg(SDValue VecV, SDValue IdxV, SDValue ValV,
   434:                               const SDLoc &dl, SelectionDAG &DAG) const;
   435:   SDValue insertHvxElementPred(SDValue VecV, SDValue IdxV, SDValue ValV,
   436:                                const SDLoc &dl, SelectionDAG &DAG) const;
   437:   SDValue extractHvxSubvectorReg(SDValue OrigOp, SDValue VecV, SDValue IdxV,
   438:                                  const SDLoc &dl, MVT ResTy, SelectionDAG &DAG)
   439:                                  const;
   440:   SDValue extractHvxSubvectorPred(SDValue VecV, SDValue IdxV, const SDLoc &dl,
   441:                                   MVT ResTy, SelectionDAG &DAG) const;
   442:   SDValue insertHvxSubvectorReg(SDValue VecV, SDValue SubV, SDValue IdxV,
   443:                                 const SDLoc &dl, SelectionDAG &DAG) const;
   444:   SDValue insertHvxSubvectorPred(SDValue VecV, SDValue SubV, SDValue IdxV,
   445:                                  const SDLoc &dl, SelectionDAG &DAG) const;
   446:   SDValue extendHvxVectorPred(SDValue VecV, const SDLoc &dl, MVT ResTy,
   447:                               bool ZeroExt, SelectionDAG &DAG) const;
   448:   SDValue compressHvxPred(SDValue VecQ, const SDLoc &dl, MVT ResTy,
   449:                           SelectionDAG &DAG) const;
   450:   SDValue resizeToWidth(SDValue VecV, MVT ResTy, bool Signed, const SDLoc &dl,
```
- EN: It declares or implements routines such as getConstant, getNode, allowsHvxMemoryAccess, allowsHvxMisalignedMemoryAccesses, ... (24 total), translating Hexagon-specific policy into reusable code paths.
- CN: 这里声明或实现了 getConstant, getNode, allowsHvxMemoryAccess, allowsHvxMisalignedMemoryAccesses, ... (24 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

### Lines 451-500 / 第 451-500 行

```cpp
   451:                         SelectionDAG &DAG) const;
   452:   SDValue extractSubvector(SDValue Vec, MVT SubTy, unsigned SubIdx,
   453:                            SelectionDAG &DAG) const;
   454:   VectorPair emitHvxAddWithOverflow(SDValue A, SDValue B, const SDLoc &dl,
   455:                                     bool Signed, SelectionDAG &DAG) const;
   456:   VectorPair emitHvxShiftRightRnd(SDValue Val, unsigned Amt, bool Signed,
   457:                                   SelectionDAG &DAG) const;
   458:   SDValue emitHvxMulHsV60(SDValue A, SDValue B, const SDLoc &dl,
   459:                           SelectionDAG &DAG) const;
   460:   SDValue emitHvxMulLoHiV60(SDValue A, bool SignedA, SDValue B, bool SignedB,
   461:                             const SDLoc &dl, SelectionDAG &DAG) const;
   462:   SDValue emitHvxMulLoHiV62(SDValue A, bool SignedA, SDValue B, bool SignedB,
   463:                             const SDLoc &dl, SelectionDAG &DAG) const;
   464: 
   465:   SDValue LowerHvxBuildVector(SDValue Op, SelectionDAG &DAG) const;
   466:   SDValue LowerHvxSplatVector(SDValue Op, SelectionDAG &DAG) const;
   467:   SDValue LowerHvxConcatVectors(SDValue Op, SelectionDAG &DAG) const;
   468:   SDValue LowerHvxExtractElement(SDValue Op, SelectionDAG &DAG) const;
   469:   SDValue LowerHvxInsertElement(SDValue Op, SelectionDAG &DAG) const;
   470:   SDValue LowerHvxExtractSubvector(SDValue Op, SelectionDAG &DAG) const;
   471:   SDValue LowerHvxInsertSubvector(SDValue Op, SelectionDAG &DAG) const;
   472:   SDValue LowerHvxBitcast(SDValue Op, SelectionDAG &DAG) const;
   473:   SDValue LowerHvxAnyExt(SDValue Op, SelectionDAG &DAG) const;
   474:   SDValue LowerHvxSignExt(SDValue Op, SelectionDAG &DAG) const;
   475:   SDValue LowerHvxZeroExt(SDValue Op, SelectionDAG &DAG) const;
   476:   SDValue LowerHvxCttz(SDValue Op, SelectionDAG &DAG) const;
   477:   SDValue LowerHvxMulh(SDValue Op, SelectionDAG &DAG) const;
   478:   SDValue LowerHvxMulLoHi(SDValue Op, SelectionDAG &DAG) const;
   479:   SDValue LowerHvxExtend(SDValue Op, SelectionDAG &DAG) const;
   480:   SDValue LowerHvxSelect(SDValue Op, SelectionDAG &DAG) const;
   481:   SDValue LowerHvxShift(SDValue Op, SelectionDAG &DAG) const;
   482:   SDValue LowerHvxFunnelShift(SDValue Op, SelectionDAG &DAG) const;
   483:   SDValue LowerHvxIntrinsic(SDValue Op, SelectionDAG &DAG) const;
   484:   SDValue LowerHvxMaskedOp(SDValue Op, SelectionDAG &DAG) const;
   485:   SDValue LowerHvxFpExtend(SDValue Op, SelectionDAG &DAG) const;
   486:   SDValue LowerHvxFpToInt(SDValue Op, SelectionDAG &DAG) const;
   487:   SDValue LowerHvxIntToFp(SDValue Op, SelectionDAG &DAG) const;
   488:   SDValue LowerHvxPred32ToFp(SDValue Op, SelectionDAG &DAG) const;
   489:   SDValue LowerHvxPred64ToFp(SDValue Op, SelectionDAG &DAG) const;
   490:   SDValue LowerHvxPartialReduceMLA(SDValue Op, SelectionDAG &DAG) const;
   491:   SDValue ExpandHvxFpToInt(SDValue Op, SelectionDAG &DAG) const;
   492:   SDValue ExpandHvxIntToFp(SDValue Op, SelectionDAG &DAG) const;
   493:   SDValue LowerHvxStore(SDValue Op, SelectionDAG &DAG) const;
   494:   SDValue LowerHvxLoad(SDValue Op, SelectionDAG &DAG) const;
   495: 
   496:   VectorPair SplitVectorOp(SDValue Op, SelectionDAG &DAG) const;
   497: 
   498:   SDValue SplitHvxMemOp(SDValue Op, SelectionDAG &DAG) const;
   499:   SDValue WidenHvxLoad(SDValue Op, SelectionDAG &DAG) const;
   500:   SDValue WidenHvxStore(SDValue Op, SelectionDAG &DAG) const;
```
- EN: It declares or implements routines such as extractSubvector, emitHvxAddWithOverflow, emitHvxShiftRightRnd, emitHvxMulHsV60, ... (40 total), translating Hexagon-specific policy into reusable code paths.
- CN: 这里声明或实现了 extractSubvector, emitHvxAddWithOverflow, emitHvxShiftRightRnd, emitHvxMulHsV60, ... (40 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

### Lines 501-542 / 第 501-542 行

```cpp
   501:   SDValue WidenHvxSetCC(SDValue Op, SelectionDAG &DAG) const;
   502:   SDValue LegalizeHvxResize(SDValue Op, SelectionDAG &DAG) const;
   503:   SDValue ExpandHvxResizeIntoSteps(SDValue Op, SelectionDAG &DAG) const;
   504:   SDValue EqualizeFpIntConversion(SDValue Op, SelectionDAG &DAG) const;
   505: 
   506:   SDValue CreateTLWrapper(SDValue Op, SelectionDAG &DAG) const;
   507:   SDValue RemoveTLWrapper(SDValue Op, SelectionDAG &DAG) const;
   508:   SDValue WidenHvxTruncateToBool(SDValue Op, SelectionDAG &DAG) const;
   509: 
   510:   std::pair<const TargetRegisterClass*, uint8_t>
   511:   findRepresentativeClass(const TargetRegisterInfo *TRI, MVT VT)
   512:       const override;
   513: 
   514:   bool shouldSplitToHvx(MVT Ty, SelectionDAG &DAG) const;
   515:   bool shouldWidenToHvx(MVT Ty, SelectionDAG &DAG) const;
   516:   bool isHvxOperation(SDNode *N, SelectionDAG &DAG) const;
   517:   SDValue LowerHvxOperation(SDValue Op, SelectionDAG &DAG) const;
   518:   void LowerHvxOperationWrapper(SDNode *N, SmallVectorImpl<SDValue> &Results,
   519:                                 SelectionDAG &DAG) const;
   520:   void ReplaceHvxNodeResults(SDNode *N, SmallVectorImpl<SDValue> &Results,
   521:                              SelectionDAG &DAG) const;
   522: 
   523:   SDValue combineTruncateBeforeLegal(SDValue Op, DAGCombinerInfo &DCI) const;
   524: 
   525:   SDValue combineConcatOfShuffles(SDValue Op, SelectionDAG &DAG) const;
   526:   SDValue combineConcatOfScalarPreds(SDValue Op, unsigned BitBytes,
   527:                                      SelectionDAG &DAG) const;
   528:   SDValue combineConcatVectorsBeforeLegal(SDValue Op, DAGCombinerInfo & DCI)
   529:       const;
   530:   SDValue expandVecReduceAdd(SDNode *N, SelectionDAG &DAG) const;
   531:   SDValue createExtendingPartialReduceMLA(
   532:       unsigned Opcode, EVT AccEltType, unsigned AccNumElements, EVT InputType,
   533:       const SDValue &A, const SDValue &B, unsigned &RemainingReductionRatio,
   534:       const SDLoc &DL, SelectionDAG &DAG) const;
   535:   SDValue splitVecReduceAdd(SDNode *N, SelectionDAG &DAG) const;
   536:   SDValue splitExtendingPartialReduceMLA(SDNode *N, SelectionDAG &DAG) const;
   537:   SDValue PerformHvxDAGCombine(SDNode *N, DAGCombinerInfo &DCI) const;
   538: };
   539: 
   540: } // end namespace llvm
   541: 
   542: #endif // LLVM_LIB_TARGET_HEXAGON_HEXAGONISELLOWERING_H
```
- EN: It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as WidenHvxSetCC, LegalizeHvxResize, ExpandHvxResizeIntoSteps, EqualizeFpIntConversion, ... (23 total), translating Hexagon-specific policy into reusable code paths.
- CN: 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 WidenHvxSetCC, LegalizeHvxResize, ExpandHvxResizeIntoSteps, EqualizeFpIntConversion, ... (23 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

## Key Concepts / 关键概念

- instruction semantics / 指令语义
- assembly/MC integration / 汇编/MC 集成
- MC-layer target description / MC 层目标描述
- target pipeline configuration / 目标流水线配置
- subtarget feature gating / 子目标特性控制
- ABI and stack-frame lowering / ABI 与栈帧下沉

## Dependencies / 依赖关系

- Direct includes / 直接包含: `Hexagon.h, MCTargetDesc/HexagonMCTargetDesc.h, llvm/ADT/StringRef.h, llvm/CodeGen/ISDOpcodes.h, llvm/CodeGen/SelectionDAGNodes.h, llvm/CodeGen/TargetLowering.h, llvm/CodeGen/ValueTypes.h, llvm/CodeGenTypes/MachineValueType.h, llvm/IR/CallingConv.h, llvm/IR/InlineAsm.h, ... (12 total)`
- Hexagon symbols / Hexagon 符号: `HexagonISelLowering, HexagonMCTargetDesc, HexagonSubtarget, HexagonTargetLowering, HexagonTargetMachine`
- Interface role / 接口角色: included by sibling Hexagon implementation files to share declarations safely. / 由同级 Hexagon 实现文件包含，以共享声明。
