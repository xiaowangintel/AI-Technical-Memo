# SPIRVLegalizerInfo.h — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/SPIRV/SPIRVLegalizerInfo.h`
- Repository: `llvm-project`
- Purpose (EN): This file declares the targeting of the MachineLegalizer class for SPIR-V.
- 目的（中文）: 该文件声明 LLVM 目标后端使用的数据结构、接口或辅助类型。
- Language: C++ header

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-40
```cpp
 1: //===- SPIRVLegalizerInfo.h --- SPIR-V Legalization Rules --------*- C++ -*-==//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // This file declares the targeting of the MachineLegalizer class for SPIR-V.
10: //
11: //===----------------------------------------------------------------------===//
12:
13: #ifndef LLVM_LIB_TARGET_SPIRV_SPIRVLEGALIZERINFO_H
14: #define LLVM_LIB_TARGET_SPIRV_SPIRVLEGALIZERINFO_H
15:
16: #include "SPIRVGlobalRegistry.h"
17: #include "llvm/CodeGen/GlobalISel/LegalizerInfo.h"
18:
19: namespace llvm {
20:
21: class LLVMContext;
22: class SPIRVSubtarget;
23:
24: // This class provides the information for legalizing SPIR-V instructions.
25: class SPIRVLegalizerInfo : public LegalizerInfo {
26:   const SPIRVSubtarget *ST;
27:   SPIRVGlobalRegistry *GR;
28:
29: public:
30:   bool legalizeCustom(LegalizerHelper &Helper, MachineInstr &MI,
31:                       LostDebugLocObserver &LocObserver) const override;
32:   bool legalizeIntrinsic(LegalizerHelper &Helper,
33:                          MachineInstr &MI) const override;
34:
35:   SPIRVLegalizerInfo(const SPIRVSubtarget &ST);
36:
37: private:
38:   bool legalizeIsFPClass(LegalizerHelper &Helper, MachineInstr &MI,
39:                          LostDebugLocObserver &LocObserver) const;
40:   bool legalizeBitcast(LegalizerHelper &Helper, MachineInstr &MI) const;
```
- EN: This range defines or declares important types such as LLVMContext, SPIRVSubtarget, SPIRVLegalizerInfo, legalizeBitcast, shaping the data model used by SPIRVLegalizerInfo.h.
- CN: 这一段定义或声明了 LLVMContext、SPIRVSubtarget、SPIRVLegalizerInfo、legalizeBitcast 等关键类型，构成 SPIRVLegalizerInfo.h 使用的数据模型。

### Lines 41-43
```cpp
41: };
42: } // namespace llvm
43: #endif // LLVM_LIB_TARGET_SPIRV_SPIRVLEGALIZERINFO_H
```
- EN: This range contains connective backend logic, including helper statements, control flow, and data movement needed by the surrounding implementation.
- CN: 这一段包含衔接性的后端逻辑，包括辅助语句、控制流和数据处理，用于支撑周围实现。

## Key Concepts / 关键概念

- EN: Legalization logic rewrites unsupported operations into forms that the target can handle.
  - CN: 合法化逻辑会把目标不支持的操作重写为可处理的形式。
- EN: SPIR-V backend code must preserve rich type information and encode capabilities/extensions explicitly.
  - CN: SPIR-V 后端需要保留丰富的类型信息，并显式编码 capability 与 extension。
- EN: Key symbols in this file include LLVMContext, SPIRVSubtarget, SPIRVLegalizerInfo, legalizeBitcast, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 LLVMContext, SPIRVSubtarget, SPIRVLegalizerInfo, legalizeBitcast，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `SPIRVGlobalRegistry.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/CodeGen/GlobalISel/LegalizerInfo.h`
