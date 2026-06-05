# HexagonTargetTransformInfo.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonTargetTransformInfo.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): This file implements a TargetTransformInfo analysis pass specific to the
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-50 / 第 1-50 行

```cpp
     1: //===- HexagonTargetTransformInfo.cpp - Hexagon specific TTI pass ---------===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: /// \file
     8: /// This file implements a TargetTransformInfo analysis pass specific to the
     9: /// Hexagon target machine. It uses the target's detailed information to provide
    10: /// more precise answers to certain TTI queries, while letting the target
    11: /// independent and default TTI implementations handle the rest.
    12: ///
    13: //===----------------------------------------------------------------------===//
    14: 
    15: #include "HexagonTargetTransformInfo.h"
    16: #include "HexagonSubtarget.h"
    17: #include "llvm/Analysis/TargetTransformInfo.h"
    18: #include "llvm/CodeGen/ValueTypes.h"
    19: #include "llvm/IR/InstrTypes.h"
    20: #include "llvm/IR/Instructions.h"
    21: #include "llvm/IR/User.h"
    22: #include "llvm/Support/Casting.h"
    23: #include "llvm/Support/CommandLine.h"
    24: #include "llvm/Transforms/Utils/LoopPeel.h"
    25: #include "llvm/Transforms/Utils/UnrollLoop.h"
    26: 
    27: using namespace llvm;
    28: 
    29: #define DEBUG_TYPE "hexagontti"
    30: 
    31: static cl::opt<bool> HexagonAutoHVX("hexagon-autohvx", cl::init(false),
    32:     cl::Hidden, cl::desc("Enable loop vectorizer for HVX"));
    33: 
    34: cl::opt<bool> HexagonAllowScatterGatherHVX(
    35:     "hexagon-allow-scatter-gather-hvx", cl::init(false), cl::Hidden,
    36:     cl::desc("Allow auto-generation of HVX scatter-gather"));
    37: 
    38: static cl::opt<bool> EnableV68FloatAutoHVX(
    39:     "force-hvx-float", cl::Hidden,
    40:     cl::desc("Enable auto-vectorization of floatint point types on v68."));
    41: 
    42: static cl::opt<bool> EmitLookupTables("hexagon-emit-lookup-tables",
    43:     cl::init(true), cl::Hidden,
    44:     cl::desc("Control lookup table emission on Hexagon target"));
    45: 
    46: static cl::opt<bool> HexagonMaskedVMem("hexagon-masked-vmem", cl::init(true),
    47:     cl::Hidden, cl::desc("Enable masked loads/stores for HVX"));
    48: 
    49: // Constant "cost factor" to make floating point operations more expensive
    50: // in terms of vectorization cost. This isn't the best way, but it should
```
- EN: It imports headers such as HexagonTargetTransformInfo.h, HexagonSubtarget.h, llvm/Analysis/TargetTransformInfo.h, llvm/CodeGen/ValueTypes.h, ... (11 total), establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as HexagonAutoHVX, HexagonAllowScatterGatherHVX, EnableV68FloatAutoHVX, EmitLookupTables, ... (5 total), translating Hexagon-specific policy into reusable code paths.
- CN: 这里引入了 HexagonTargetTransformInfo.h, HexagonSubtarget.h, llvm/Analysis/TargetTransformInfo.h, llvm/CodeGen/ValueTypes.h, ... (11 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 HexagonAutoHVX, HexagonAllowScatterGatherHVX, EnableV68FloatAutoHVX, EmitLookupTables, ... (5 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

### Lines 51-100 / 第 51-100 行

```cpp
    51: // do. Ultimately, the cost should use cycles.
    52: static const unsigned FloatFactor = 4;
    53: 
    54: bool HexagonTTIImpl::useHVX() const {
    55:   return ST.useHVXOps() && HexagonAutoHVX;
    56: }
    57: 
    58: bool HexagonTTIImpl::isHVXVectorType(Type *Ty) const {
    59:   auto *VecTy = dyn_cast<VectorType>(Ty);
    60:   if (!VecTy)
    61:     return false;
    62:   if (!ST.isTypeForHVX(VecTy))
    63:     return false;
    64:   if (ST.useHVXV69Ops() || !VecTy->getElementType()->isFloatingPointTy())
    65:     return true;
    66:   return ST.useHVXV68Ops() && EnableV68FloatAutoHVX;
    67: }
    68: 
    69: unsigned HexagonTTIImpl::getTypeNumElements(Type *Ty) const {
    70:   if (auto *VTy = dyn_cast<FixedVectorType>(Ty))
    71:     return VTy->getNumElements();
    72:   assert((Ty->isIntegerTy() || Ty->isFloatingPointTy()) &&
    73:          "Expecting scalar type");
    74:   return 1;
    75: }
    76: 
    77: TargetTransformInfo::PopcntSupportKind
    78: HexagonTTIImpl::getPopcntSupport(unsigned IntTyWidthInBit) const {
    79:   // Return fast hardware support as every input < 64 bits will be promoted
    80:   // to 64 bits.
    81:   return TargetTransformInfo::PSK_FastHardware;
    82: }
    83: 
    84: // The Hexagon target can unroll loops with run-time trip counts.
    85: void HexagonTTIImpl::getUnrollingPreferences(
    86:     Loop *L, ScalarEvolution &SE, TTI::UnrollingPreferences &UP,
    87:     OptimizationRemarkEmitter *ORE) const {
    88:   UP.Runtime = UP.Partial = true;
    89: }
    90: 
    91: void HexagonTTIImpl::getPeelingPreferences(Loop *L, ScalarEvolution &SE,
    92:                                            TTI::PeelingPreferences &PP) const {
    93:   BaseT::getPeelingPreferences(L, SE, PP);
    94:   // Only try to peel innermost loops with small runtime trip counts.
    95:   if (L && L->isInnermost() && canPeel(L) &&
    96:       SE.getSmallConstantTripCount(L) == 0 &&
    97:       SE.getSmallConstantMaxTripCount(L) > 0 &&
    98:       SE.getSmallConstantMaxTripCount(L) <= 5) {
    99:     PP.PeelCount = 2;
   100:   }
```
- EN: It declares or implements routines such as HexagonTTIImpl::useHVX, HexagonTTIImpl::isHVXVectorType, dyn_cast<VectorType>, HexagonTTIImpl::getTypeNumElements, ... (9 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonTTIImpl, HexagonAutoHVX, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonTTIImpl::useHVX, HexagonTTIImpl::isHVXVectorType, dyn_cast<VectorType>, HexagonTTIImpl::getTypeNumElements, ... (9 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonTTIImpl, HexagonAutoHVX，说明了它与同级后端组件的连接关系。

### Lines 101-150 / 第 101-150 行

```cpp
   101: }
   102: 
   103: TTI::AddressingModeKind
   104: HexagonTTIImpl::getPreferredAddressingMode(const Loop *L,
   105:                                            ScalarEvolution *SE) const {
   106:   return TTI::AMK_PostIndexed;
   107: }
   108: 
   109: /// --- Vector TTI begin ---
   110: 
   111: unsigned HexagonTTIImpl::getNumberOfRegisters(unsigned ClassID) const {
   112:   bool Vector = ClassID == 1;
   113:   if (Vector)
   114:     return useHVX() ? 32 : 0;
   115:   return 32;
   116: }
   117: 
   118: unsigned HexagonTTIImpl::getMaxInterleaveFactor(ElementCount VF) const {
   119:   return useHVX() ? 2 : 1;
   120: }
   121: 
   122: TypeSize
   123: HexagonTTIImpl::getRegisterBitWidth(TargetTransformInfo::RegisterKind K) const {
   124:   switch (K) {
   125:   case TargetTransformInfo::RGK_Scalar:
   126:     return TypeSize::getFixed(32);
   127:   case TargetTransformInfo::RGK_FixedWidthVector:
   128:     return TypeSize::getFixed(getMinVectorRegisterBitWidth());
   129:   case TargetTransformInfo::RGK_ScalableVector:
   130:     return TypeSize::getScalable(0);
   131:   }
   132: 
   133:   llvm_unreachable("Unsupported register kind");
   134: }
   135: 
   136: unsigned HexagonTTIImpl::getMinVectorRegisterBitWidth() const {
   137:   return useHVX() ? ST.getVectorLength()*8 : 32;
   138: }
   139: 
   140: ElementCount HexagonTTIImpl::getMinimumVF(unsigned ElemWidth,
   141:                                           bool IsScalable) const {
   142:   assert(!IsScalable && "Scalable VFs are not supported for Hexagon");
   143:   return ElementCount::getFixed((8 * ST.getVectorLength()) / ElemWidth);
   144: }
   145: 
   146: InstructionCost
   147: HexagonTTIImpl::getCallInstrCost(Function *F, Type *RetTy, ArrayRef<Type *> Tys,
   148:                                  TTI::TargetCostKind CostKind) const {
   149:   return BaseT::getCallInstrCost(F, RetTy, Tys, CostKind);
   150: }
```
- EN: It declares or implements routines such as HexagonTTIImpl::getPreferredAddressingMode, HexagonTTIImpl::getNumberOfRegisters, HexagonTTIImpl::getMaxInterleaveFactor, HexagonTTIImpl::getRegisterBitWidth, ... (13 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonTTIImpl, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonTTIImpl::getPreferredAddressingMode, HexagonTTIImpl::getNumberOfRegisters, HexagonTTIImpl::getMaxInterleaveFactor, HexagonTTIImpl::getRegisterBitWidth, ... (13 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonTTIImpl，说明了它与同级后端组件的连接关系。

### Lines 151-200 / 第 151-200 行

```cpp
   151: 
   152: InstructionCost
   153: HexagonTTIImpl::getIntrinsicInstrCost(const IntrinsicCostAttributes &ICA,
   154:                                       TTI::TargetCostKind CostKind) const {
   155:   if (ICA.getID() == Intrinsic::bswap) {
   156:     std::pair<InstructionCost, MVT> LT =
   157:         getTypeLegalizationCost(ICA.getReturnType());
   158:     return LT.first + 2;
   159:   }
   160:   return BaseT::getIntrinsicInstrCost(ICA, CostKind);
   161: }
   162: 
   163: InstructionCost
   164: HexagonTTIImpl::getAddressComputationCost(Type *PtrTy, ScalarEvolution *SE,
   165:                                           const SCEV *S,
   166:                                           TTI::TargetCostKind CostKind) const {
   167:   return 0;
   168: }
   169: 
   170: InstructionCost HexagonTTIImpl::getMemoryOpCost(unsigned Opcode, Type *Src,
   171:                                                 Align Alignment,
   172:                                                 unsigned AddressSpace,
   173:                                                 TTI::TargetCostKind CostKind,
   174:                                                 TTI::OperandValueInfo OpInfo,
   175:                                                 const Instruction *I) const {
   176:   assert(Opcode == Instruction::Load || Opcode == Instruction::Store);
   177:   // TODO: Handle other cost kinds.
   178:   if (CostKind != TTI::TCK_RecipThroughput)
   179:     return 1;
   180: 
   181:   if (Opcode == Instruction::Store)
   182:     return BaseT::getMemoryOpCost(Opcode, Src, Alignment, AddressSpace,
   183:                                   CostKind, OpInfo, I);
   184: 
   185:   if (Src->isVectorTy()) {
   186:     VectorType *VecTy = cast<VectorType>(Src);
   187:     unsigned VecWidth = VecTy->getPrimitiveSizeInBits().getFixedValue();
   188:     if (isHVXVectorType(VecTy)) {
   189:       unsigned RegWidth =
   190:           getRegisterBitWidth(TargetTransformInfo::RGK_FixedWidthVector)
   191:               .getFixedValue();
   192:       assert(RegWidth && "Non-zero vector register width expected");
   193:       // Cost of HVX loads.
   194:       if (VecWidth % RegWidth == 0)
   195:         return VecWidth / RegWidth;
   196:       // Cost of constructing HVX vector from scalar loads
   197:       const Align RegAlign(RegWidth / 8);
   198:       if (Alignment > RegAlign)
   199:         Alignment = RegAlign;
   200:       unsigned AlignWidth = 8 * Alignment.value();
```
- EN: It declares or implements routines such as HexagonTTIImpl::getIntrinsicInstrCost, getTypeLegalizationCost, BaseT::getIntrinsicInstrCost, HexagonTTIImpl::getAddressComputationCost, ... (11 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonTTIImpl, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonTTIImpl::getIntrinsicInstrCost, getTypeLegalizationCost, BaseT::getIntrinsicInstrCost, HexagonTTIImpl::getAddressComputationCost, ... (11 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonTTIImpl，说明了它与同级后端组件的连接关系。

### Lines 201-250 / 第 201-250 行

```cpp
   201:       unsigned NumLoads = alignTo(VecWidth, AlignWidth) / AlignWidth;
   202:       return 3 * NumLoads;
   203:     }
   204: 
   205:     // Non-HVX vectors.
   206:     // Add extra cost for floating point types.
   207:     unsigned Cost =
   208:         VecTy->getElementType()->isFloatingPointTy() ? FloatFactor : 1;
   209: 
   210:     // At this point unspecified alignment is considered as Align(1).
   211:     const Align BoundAlignment = std::min(Alignment, Align(8));
   212:     unsigned AlignWidth = 8 * BoundAlignment.value();
   213:     unsigned NumLoads = alignTo(VecWidth, AlignWidth) / AlignWidth;
   214:     if (Alignment == Align(4) || Alignment == Align(8))
   215:       return Cost * NumLoads;
   216:     // Loads of less than 32 bits will need extra inserts to compose a vector.
   217:     assert(BoundAlignment <= Align(8));
   218:     unsigned LogA = Log2(BoundAlignment);
   219:     return (3 - LogA) * Cost * NumLoads;
   220:   }
   221: 
   222:   return BaseT::getMemoryOpCost(Opcode, Src, Alignment, AddressSpace, CostKind,
   223:                                 OpInfo, I);
   224: }
   225: 
   226: InstructionCost
   227: HexagonTTIImpl::getShuffleCost(TTI::ShuffleKind Kind, VectorType *DstTy,
   228:                                VectorType *SrcTy, ArrayRef<int> Mask,
   229:                                TTI::TargetCostKind CostKind, int Index,
   230:                                VectorType *SubTp, ArrayRef<const Value *> Args,
   231:                                const Instruction *CxtI) const {
   232:   return 1;
   233: }
   234: 
   235: InstructionCost HexagonTTIImpl::getInterleavedMemoryOpCost(
   236:     unsigned Opcode, Type *VecTy, unsigned Factor, ArrayRef<unsigned> Indices,
   237:     Align Alignment, unsigned AddressSpace, TTI::TargetCostKind CostKind,
   238:     bool UseMaskForCond, bool UseMaskForGaps) const {
   239:   if (Indices.size() != Factor || UseMaskForCond || UseMaskForGaps)
   240:     return BaseT::getInterleavedMemoryOpCost(Opcode, VecTy, Factor, Indices,
   241:                                              Alignment, AddressSpace,
   242:                                              CostKind,
   243:                                              UseMaskForCond, UseMaskForGaps);
   244:   return getMemoryOpCost(Opcode, VecTy, Alignment, AddressSpace, CostKind);
   245: }
   246: 
   247: InstructionCost HexagonTTIImpl::getCmpSelInstrCost(
   248:     unsigned Opcode, Type *ValTy, Type *CondTy, CmpInst::Predicate VecPred,
   249:     TTI::TargetCostKind CostKind, TTI::OperandValueInfo Op1Info,
   250:     TTI::OperandValueInfo Op2Info, const Instruction *I) const {
```
- EN: It declares or implements routines such as Align, value, assert, Log2, ... (9 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonTTIImpl, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 Align, value, assert, Log2, ... (9 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonTTIImpl，说明了它与同级后端组件的连接关系。

### Lines 251-300 / 第 251-300 行

```cpp
   251:   if (ValTy->isVectorTy() && CostKind == TTI::TCK_RecipThroughput) {
   252:     if (!isHVXVectorType(ValTy) && ValTy->isFPOrFPVectorTy())
   253:       return InstructionCost::getMax();
   254:     std::pair<InstructionCost, MVT> LT = getTypeLegalizationCost(ValTy);
   255:     if (Opcode == Instruction::FCmp)
   256:       return LT.first + FloatFactor * getTypeNumElements(ValTy);
   257:   }
   258:   return BaseT::getCmpSelInstrCost(Opcode, ValTy, CondTy, VecPred, CostKind,
   259:                                    Op1Info, Op2Info, I);
   260: }
   261: 
   262: InstructionCost HexagonTTIImpl::getArithmeticInstrCost(
   263:     unsigned Opcode, Type *Ty, TTI::TargetCostKind CostKind,
   264:     TTI::OperandValueInfo Op1Info, TTI::OperandValueInfo Op2Info,
   265:     ArrayRef<const Value *> Args, const Instruction *CxtI) const {
   266:   // TODO: Handle more cost kinds.
   267:   if (CostKind != TTI::TCK_RecipThroughput)
   268:     return BaseT::getArithmeticInstrCost(Opcode, Ty, CostKind, Op1Info,
   269:                                          Op2Info, Args, CxtI);
   270: 
   271:   if (Ty->isVectorTy()) {
   272:     if (!isHVXVectorType(Ty) && Ty->isFPOrFPVectorTy())
   273:       return InstructionCost::getMax();
   274:     std::pair<InstructionCost, MVT> LT = getTypeLegalizationCost(Ty);
   275:     if (LT.second.isFloatingPoint())
   276:       return LT.first + FloatFactor * getTypeNumElements(Ty);
   277:   }
   278:   return BaseT::getArithmeticInstrCost(Opcode, Ty, CostKind, Op1Info, Op2Info,
   279:                                        Args, CxtI);
   280: }
   281: 
   282: InstructionCost HexagonTTIImpl::getCastInstrCost(unsigned Opcode, Type *DstTy,
   283:                                                  Type *SrcTy,
   284:                                                  TTI::CastContextHint CCH,
   285:                                                  TTI::TargetCostKind CostKind,
   286:                                                  const Instruction *I) const {
   287:   auto isNonHVXFP = [this] (Type *Ty) {
   288:     return Ty->isVectorTy() && !isHVXVectorType(Ty) && Ty->isFPOrFPVectorTy();
   289:   };
   290:   if (isNonHVXFP(SrcTy) || isNonHVXFP(DstTy))
   291:     return InstructionCost::getMax();
   292: 
   293:   if (SrcTy->isFPOrFPVectorTy() || DstTy->isFPOrFPVectorTy()) {
   294:     unsigned SrcN = SrcTy->isFPOrFPVectorTy() ? getTypeNumElements(SrcTy) : 0;
   295:     unsigned DstN = DstTy->isFPOrFPVectorTy() ? getTypeNumElements(DstTy) : 0;
   296: 
   297:     std::pair<InstructionCost, MVT> SrcLT = getTypeLegalizationCost(SrcTy);
   298:     std::pair<InstructionCost, MVT> DstLT = getTypeLegalizationCost(DstTy);
   299:     InstructionCost Cost =
   300:         std::max(SrcLT.first, DstLT.first) + FloatFactor * (SrcN + DstN);
```
- EN: It declares or implements routines such as getTypeLegalizationCost, BaseT::getCmpSelInstrCost, HexagonTTIImpl::getArithmeticInstrCost, BaseT::getArithmeticInstrCost, ... (7 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonTTIImpl, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getTypeLegalizationCost, BaseT::getCmpSelInstrCost, HexagonTTIImpl::getArithmeticInstrCost, BaseT::getArithmeticInstrCost, ... (7 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonTTIImpl，说明了它与同级后端组件的连接关系。

### Lines 301-350 / 第 301-350 行

```cpp
   301:     // TODO: Allow non-throughput costs that aren't binary.
   302:     if (CostKind != TTI::TCK_RecipThroughput)
   303:       return Cost == 0 ? 0 : 1;
   304:     return Cost;
   305:   }
   306:   return 1;
   307: }
   308: 
   309: InstructionCost HexagonTTIImpl::getVectorInstrCost(
   310:     unsigned Opcode, Type *Val, TTI::TargetCostKind CostKind, unsigned Index,
   311:     const Value *Op0, const Value *Op1, TTI::VectorInstrContext VIC) const {
   312:   Type *ElemTy = Val->isVectorTy() ? cast<VectorType>(Val)->getElementType()
   313:                                    : Val;
   314:   if (Opcode == Instruction::InsertElement) {
   315:     // Need two rotations for non-zero index.
   316:     unsigned Cost = (Index != 0) ? 2 : 0;
   317:     if (ElemTy->isIntegerTy(32))
   318:       return Cost;
   319:     // If it's not a 32-bit value, there will need to be an extract.
   320:     return Cost + getVectorInstrCost(Instruction::ExtractElement, Val, CostKind,
   321:                                      Index, Op0, Op1, VIC);
   322:   }
   323: 
   324:   if (Opcode == Instruction::ExtractElement)
   325:     return 2;
   326: 
   327:   return 1;
   328: }
   329: 
   330: bool HexagonTTIImpl::shouldExpandReduction(const IntrinsicInst *II) const {
   331:   switch (II->getIntrinsicID()) {
   332:   case Intrinsic::vector_reduce_add:
   333:     return false;
   334:   }
   335:   return true;
   336: }
   337: 
   338: bool HexagonTTIImpl::isLegalMaskedStore(Type *DataType, Align /*Alignment*/,
   339:                                         unsigned /*AddressSpace*/,
   340:                                         TTI::MaskKind /*MaskKind*/) const {
   341:   // This function is called from scalarize-masked-mem-intrin, which runs
   342:   // in pre-isel. Use ST directly instead of calling isHVXVectorType.
   343:   return HexagonMaskedVMem && ST.isTypeForHVX(DataType);
   344: }
   345: 
   346: bool HexagonTTIImpl::isLegalMaskedLoad(Type *DataType, Align /*Alignment*/,
   347:                                        unsigned /*AddressSpace*/,
   348:                                        TTI::MaskKind /*MaskKind*/) const {
   349:   // This function is called from scalarize-masked-mem-intrin, which runs
   350:   // in pre-isel. Use ST directly instead of calling isHVXVectorType.
```
- EN: It declares or implements routines such as HexagonTTIImpl::getVectorInstrCost, getVectorInstrCost, HexagonTTIImpl::shouldExpandReduction, HexagonTTIImpl::isLegalMaskedStore, ... (6 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonTTIImpl, HexagonMaskedVMem, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonTTIImpl::getVectorInstrCost, getVectorInstrCost, HexagonTTIImpl::shouldExpandReduction, HexagonTTIImpl::isLegalMaskedStore, ... (6 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonTTIImpl, HexagonMaskedVMem，说明了它与同级后端组件的连接关系。

### Lines 351-400 / 第 351-400 行

```cpp
   351:   return HexagonMaskedVMem && ST.isTypeForHVX(DataType);
   352: }
   353: 
   354: bool HexagonTTIImpl::isLegalMaskedGather(Type *Ty, Align Alignment) const {
   355:   // For now assume we can not deal with all HVX datatypes.
   356:   if (!Ty->isVectorTy() || !ST.isTypeForHVX(Ty) ||
   357:       !HexagonAllowScatterGatherHVX)
   358:     return false;
   359:   // This must be in sync with HexagonVectorCombine pass.
   360:   switch (Ty->getScalarSizeInBits()) {
   361:   case 8:
   362:     return (getTypeNumElements(Ty) == 128);
   363:   case 16:
   364:     if (getTypeNumElements(Ty) == 64 || getTypeNumElements(Ty) == 32)
   365:       return (Alignment >= 2);
   366:     break;
   367:   case 32:
   368:     if (getTypeNumElements(Ty) == 32)
   369:       return (Alignment >= 4);
   370:     break;
   371:   default:
   372:     break;
   373:   }
   374:   return false;
   375: }
   376: 
   377: bool HexagonTTIImpl::isLegalMaskedScatter(Type *Ty, Align Alignment) const {
   378:   if (!Ty->isVectorTy() || !ST.isTypeForHVX(Ty) ||
   379:       !HexagonAllowScatterGatherHVX)
   380:     return false;
   381:   // This must be in sync with HexagonVectorCombine pass.
   382:   switch (Ty->getScalarSizeInBits()) {
   383:   case 8:
   384:     return (getTypeNumElements(Ty) == 128);
   385:   case 16:
   386:     if (getTypeNumElements(Ty) == 64)
   387:       return (Alignment >= 2);
   388:     break;
   389:   case 32:
   390:     if (getTypeNumElements(Ty) == 32)
   391:       return (Alignment >= 4);
   392:     break;
   393:   default:
   394:     break;
   395:   }
   396:   return false;
   397: }
   398: 
   399: bool HexagonTTIImpl::forceScalarizeMaskedGather(VectorType *VTy,
   400:                                                 Align Alignment) const {
```
- EN: It declares or implements routines such as isTypeForHVX, HexagonTTIImpl::isLegalMaskedGather, HexagonTTIImpl::isLegalMaskedScatter, HexagonTTIImpl::forceScalarizeMaskedGather, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonMaskedVMem, HexagonTTIImpl, HexagonAllowScatterGatherHVX, HexagonVectorCombine, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 isTypeForHVX, HexagonTTIImpl::isLegalMaskedGather, HexagonTTIImpl::isLegalMaskedScatter, HexagonTTIImpl::forceScalarizeMaskedGather 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonMaskedVMem, HexagonTTIImpl, HexagonAllowScatterGatherHVX, HexagonVectorCombine，说明了它与同级后端组件的连接关系。

### Lines 401-449 / 第 401-449 行

```cpp
   401:   return !isLegalMaskedGather(VTy, Alignment);
   402: }
   403: 
   404: bool HexagonTTIImpl::forceScalarizeMaskedScatter(VectorType *VTy,
   405:                                                  Align Alignment) const {
   406:   return !isLegalMaskedScatter(VTy, Alignment);
   407: }
   408: 
   409: /// --- Vector TTI end ---
   410: 
   411: unsigned HexagonTTIImpl::getPrefetchDistance() const {
   412:   return ST.getL1PrefetchDistance();
   413: }
   414: 
   415: unsigned HexagonTTIImpl::getCacheLineSize() const {
   416:   return ST.getL1CacheLineSize();
   417: }
   418: 
   419: InstructionCost
   420: HexagonTTIImpl::getInstructionCost(const User *U,
   421:                                    ArrayRef<const Value *> Operands,
   422:                                    TTI::TargetCostKind CostKind) const {
   423:   auto isCastFoldedIntoLoad = [this](const CastInst *CI) -> bool {
   424:     if (!CI->isIntegerCast())
   425:       return false;
   426:     // Only extensions from an integer type shorter than 32-bit to i32
   427:     // can be folded into the load.
   428:     const DataLayout &DL = getDataLayout();
   429:     unsigned SBW = DL.getTypeSizeInBits(CI->getSrcTy());
   430:     unsigned DBW = DL.getTypeSizeInBits(CI->getDestTy());
   431:     if (DBW != 32 || SBW >= DBW)
   432:       return false;
   433: 
   434:     const LoadInst *LI = dyn_cast<const LoadInst>(CI->getOperand(0));
   435:     // Technically, this code could allow multiple uses of the load, and
   436:     // check if all the uses are the same extension operation, but this
   437:     // should be sufficient for most cases.
   438:     return LI && LI->hasOneUse();
   439:   };
   440: 
   441:   if (const CastInst *CI = dyn_cast<const CastInst>(U))
   442:     if (isCastFoldedIntoLoad(CI))
   443:       return TargetTransformInfo::TCC_Free;
   444:   return BaseT::getInstructionCost(U, Operands, CostKind);
   445: }
   446: 
   447: bool HexagonTTIImpl::shouldBuildLookupTables() const {
   448:   return EmitLookupTables;
   449: }
```
- EN: It declares or implements routines such as isLegalMaskedGather, HexagonTTIImpl::forceScalarizeMaskedScatter, isLegalMaskedScatter, HexagonTTIImpl::getPrefetchDistance, ... (14 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonTTIImpl, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 isLegalMaskedGather, HexagonTTIImpl::forceScalarizeMaskedScatter, isLegalMaskedScatter, HexagonTTIImpl::getPrefetchDistance, ... (14 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonTTIImpl，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- HVX vector ISA modeling / HVX 向量 ISA 建模
- loop-aware code generation / 循环相关代码生成
- instruction semantics / 指令语义
- subtarget feature gating / 子目标特性控制

## Dependencies / 依赖关系

- Direct includes / 直接包含: `HexagonTargetTransformInfo.h, HexagonSubtarget.h, llvm/Analysis/TargetTransformInfo.h, llvm/CodeGen/ValueTypes.h, llvm/IR/InstrTypes.h, llvm/IR/Instructions.h, llvm/IR/User.h, llvm/Support/Casting.h, llvm/Support/CommandLine.h, llvm/Transforms/Utils/LoopPeel.h, ... (11 total)`
- Hexagon symbols / Hexagon 符号: `HexagonTargetTransformInfo, HexagonSubtarget, HexagonAutoHVX, HexagonAllowScatterGatherHVX, HexagonMaskedVMem, HexagonTTIImpl, HexagonVectorCombine`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
