# SPIRVCombinerHelper.h — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/SPIRV/SPIRVCombinerHelper.h`
- Repository: `llvm-project`
- Purpose (EN): This contains common combine transformations that may be used in a combine pass.
- 目的（中文）: 该文件声明 LLVM 目标后端使用的数据结构、接口或辅助类型。
- Language: C++ header

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-40
```cpp
 1: //===-- SPIRVCombinerHelper.h -----------------------------------*- C++ -*-===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: ///
 9: /// This contains common combine transformations that may be used in a combine
10: /// pass.
11: ///
12: //===----------------------------------------------------------------------===//
13:
14: #ifndef LLVM_LIB_TARGET_SPIRV_SPIRVCOMBINERHELPER_H
15: #define LLVM_LIB_TARGET_SPIRV_SPIRVCOMBINERHELPER_H
16:
17: #include "SPIRVSubtarget.h"
18: #include "llvm/CodeGen/GlobalISel/CombinerHelper.h"
19:
20: namespace llvm {
21: class SPIRVCombinerHelper : public CombinerHelper {
22: protected:
23:   const SPIRVSubtarget &STI;
24:
25: public:
26:   using CombinerHelper::CombinerHelper;
27:   SPIRVCombinerHelper(GISelChangeObserver &Observer, MachineIRBuilder &B,
28:                       bool IsPreLegalize, GISelValueTracking *VT,
29:                       MachineDominatorTree *MDT, const LegalizerInfo *LI,
30:                       const SPIRVSubtarget &STI);
31:
32:   bool matchLengthToDistance(MachineInstr &MI) const;
33:   void applySPIRVDistance(MachineInstr &MI) const;
34:   bool matchSelectToFaceForward(MachineInstr &MI) const;
35:   void applySPIRVFaceForward(MachineInstr &MI) const;
36:   bool matchMatrixTranspose(MachineInstr &MI) const;
37:   void applyMatrixTranspose(MachineInstr &MI) const;
38:   bool matchMatrixMultiply(MachineInstr &MI) const;
39:   void applyMatrixMultiply(MachineInstr &MI) const;
40:
```
- EN: This range defines or declares important types such as SPIRVCombinerHelper, matchLengthToDistance, applySPIRVDistance, matchSelectToFaceForward, shaping the data model used by SPIRVCombinerHelper.h.
- CN: 这一段定义或声明了 SPIRVCombinerHelper、matchLengthToDistance、applySPIRVDistance、matchSelectToFaceForward 等关键类型，构成 SPIRVCombinerHelper.h 使用的数据模型。

### Lines 41-62
```cpp
41: private:
42:   SPIRVTypeInst getDotProductVectorType(Register ResReg, uint32_t K,
43:                                         SPIRVGlobalRegistry *GR) const;
44:   SmallVector<Register, 4> extractColumns(Register BReg, uint32_t N,
45:                                           SPIRVTypeInst SpvVecType,
46:                                           SPIRVGlobalRegistry *GR) const;
47:   SmallVector<Register, 4> extractRows(Register AReg, uint32_t NumRows,
48:                                        uint32_t NumCols,
49:                                        SPIRVTypeInst SpvRowType,
50:                                        SPIRVGlobalRegistry *GR) const;
51:   SmallVector<Register, 16>
52:   computeDotProducts(const SmallVector<Register, 4> &RowsA,
53:                      const SmallVector<Register, 4> &ColsB,
54:                      SPIRVTypeInst SpvVecType, SPIRVGlobalRegistry *GR) const;
55:   Register computeDotProduct(Register RowA, Register ColB,
56:                              SPIRVTypeInst SpvVecType,
57:                              SPIRVGlobalRegistry *GR) const;
58: };
59:
60: } // end namespace llvm
61:
62: #endif // LLVM_LIB_TARGET_SPIRV_SPIRVCOMBINERHELPER_H
```
- EN: This range declares interfaces or inline helpers such as backend logic, defining how other backend pieces interact with this header.
- CN: 这一段声明了 后端逻辑 等接口或内联辅助函数，定义其他后端组件如何与该头文件协作。

## Key Concepts / 关键概念

- EN: SPIR-V backend code must preserve rich type information and encode capabilities/extensions explicitly.
  - CN: SPIR-V 后端需要保留丰富的类型信息，并显式编码 capability 与 extension。
- EN: Key symbols in this file include SPIRVCombinerHelper, matchLengthToDistance, applySPIRVDistance, matchSelectToFaceForward, applySPIRVFaceForward, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 SPIRVCombinerHelper, matchLengthToDistance, applySPIRVDistance, matchSelectToFaceForward, applySPIRVFaceForward，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `SPIRVSubtarget.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/CodeGen/GlobalISel/CombinerHelper.h`
