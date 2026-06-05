# SPIRVTargetTransformInfo.h — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/SPIRV/SPIRVTargetTransformInfo.h`
- Repository: `llvm-project`
- Purpose (EN): This file contains a TargetTransformInfoImplBase conforming object specific to the SPIRV target machine.
- 目的（中文）: 该文件声明 LLVM 目标后端使用的数据结构、接口或辅助类型。
- Language: C++ header

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-40
```cpp
 1: //===- SPIRVTargetTransformInfo.h - SPIR-V specific TTI ---------*- C++ -*-===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: // \file
 9: // This file contains a TargetTransformInfoImplBase conforming object specific
10: // to the SPIRV target machine. It uses the target's detailed information to
11: // provide more precise answers to certain TTI queries, while letting the
12: // target independent and default TTI implementations handle the rest.
13: //
14: //===----------------------------------------------------------------------===//
15:
16: #ifndef LLVM_LIB_TARGET_SPIRV_SPIRVTARGETTRANSFORMINFO_H
17: #define LLVM_LIB_TARGET_SPIRV_SPIRVTARGETTRANSFORMINFO_H
18:
19: #include "SPIRV.h"
20: #include "SPIRVTargetMachine.h"
21: #include "llvm/Analysis/TargetTransformInfo.h"
22: #include "llvm/CodeGen/BasicTTIImpl.h"
23:
24: namespace llvm {
25: class SPIRVTTIImpl final : public BasicTTIImplBase<SPIRVTTIImpl> {
26:   using BaseT = BasicTTIImplBase<SPIRVTTIImpl>;
27:   using TTI = TargetTransformInfo;
28:
29:   friend BaseT;
30:
31:   const SPIRVSubtarget *ST;
32:   const SPIRVTargetLowering *TLI;
33:
34:   const TargetSubtargetInfo *getST() const { return ST; }
35:   const SPIRVTargetLowering *getTLI() const { return TLI; }
36:
37: public:
38:   explicit SPIRVTTIImpl(const SPIRVTargetMachine *TM, const Function &F)
39:       : BaseT(TM, F.getDataLayout()), ST(TM->getSubtargetImpl(F)),
40:         TLI(ST->getTargetLowering()) {}
```
- EN: This range defines or declares important types such as SPIRVTTIImpl, getST, getTLI, TLI, shaping the data model used by SPIRVTargetTransformInfo.h.
- CN: 这一段定义或声明了 SPIRVTTIImpl、getST、getTLI、TLI 等关键类型，构成 SPIRVTargetTransformInfo.h 使用的数据模型。

### Lines 41-80
```cpp
41:
42:   TTI::PopcntSupportKind getPopcntSupport(unsigned TyWidth) const override {
43:     // SPIR-V natively supports OpBitcount, per 3.53.14 in the spec, as such it
44:     // is reasonable to assume the Op is fast / preferable to the expanded loop.
45:     // Furthermore, this prevents information being lost if transforms are
46:     // applied to SPIR-V before lowering to a concrete target.
47:     if (!isPowerOf2_32(TyWidth) || TyWidth > 64)
48:       return TTI::PSK_Software; // Arbitrary bit-width INT is not core SPIR-V.
49:     return TTI::PSK_FastHardware;
50:   }
51:
52:   unsigned getFlatAddressSpace() const override {
53:     // Clang has 2 distinct address space maps. One where
54:     // default=4=Generic, and one with default=0=Function. This depends on the
55:     // environment.
56:     return ST->isShader() ? 0 : 4;
57:   }
58:   bool collectFlatAddressOperands(SmallVectorImpl<int> &OpIndexes,
59:                                   Intrinsic::ID IID) const override;
60:   Value *rewriteIntrinsicWithAddressSpace(IntrinsicInst *II, Value *OldV,
61:                                           Value *NewV) const override;
62:
63:   bool allowVectorElementIndexingUsingGEP() const override { return false; }
64:
65:   bool isLegalMaskedGather(Type *DataType, Align Alignment) const override;
66:   bool isLegalMaskedScatter(Type *DataType, Align Alignment) const override;
67:
68:   InstructionCost getPartialReductionCost(
69:       unsigned Opcode, Type *InputTypeA, Type *InputTypeB, Type *AccumType,
70:       ElementCount VF, TTI::PartialReductionExtendKind OpAExtend,
71:       TTI::PartialReductionExtendKind OpBExtend, std::optional<unsigned> BinOp,
72:       TTI::TargetCostKind CostKind,
73:       std::optional<FastMathFlags> FMF) const override {
74:     return InstructionCost::getInvalid();
75:   }
76: };
77:
78: } // namespace llvm
79:
80: #endif // LLVM_LIB_TARGET_SPIRV_SPIRVTARGETTRANSFORMINFO_H
```
- EN: This range declares interfaces or inline helpers such as getPopcntSupport, getFlatAddressSpace, allowVectorElementIndexingUsingGEP, isLegalMaskedGather, defining how other backend pieces interact with this header.
- CN: 这一段声明了 getPopcntSupport、getFlatAddressSpace、allowVectorElementIndexingUsingGEP、isLegalMaskedGather 等接口或内联辅助函数，定义其他后端组件如何与该头文件协作。

## Key Concepts / 关键概念

- EN: SPIR-V backend code must preserve rich type information and encode capabilities/extensions explicitly.
  - CN: SPIR-V 后端需要保留丰富的类型信息，并显式编码 capability 与 extension。
- EN: Key symbols in this file include SPIRVTTIImpl, getST, getTLI, TLI, getPopcntSupport, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 SPIRVTTIImpl, getST, getTLI, TLI, getPopcntSupport，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `SPIRV.h`
  - `SPIRVTargetMachine.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/Analysis/TargetTransformInfo.h`
  - `llvm/CodeGen/BasicTTIImpl.h`
