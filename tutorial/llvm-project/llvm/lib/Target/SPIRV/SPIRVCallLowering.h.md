# SPIRVCallLowering.h — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/SPIRV/SPIRVCallLowering.h`
- Repository: `llvm-project`
- Purpose (EN): This file describes how to lower LLVM calls to machine code calls.
- 目的（中文）: 该文件声明 LLVM 目标后端使用的数据结构、接口或辅助类型。
- Language: C++ header

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-40
```cpp
 1: //===--- SPIRVCallLowering.h - Call lowering --------------------*- C++ -*-===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // This file describes how to lower LLVM calls to machine code calls.
10: //
11: //===----------------------------------------------------------------------===//
12:
13: #ifndef LLVM_LIB_TARGET_SPIRV_SPIRVCALLLOWERING_H
14: #define LLVM_LIB_TARGET_SPIRV_SPIRVCALLLOWERING_H
15:
16: #include "SPIRVGlobalRegistry.h"
17: #include "llvm/CodeGen/GlobalISel/CallLowering.h"
18:
19: namespace llvm {
20:
21: class SPIRVGlobalRegistry;
22: class SPIRVTargetLowering;
23:
24: class SPIRVCallLowering : public CallLowering {
25: private:
26:   // Used to create and assign function, argument, and return type information.
27:   SPIRVGlobalRegistry *GR;
28:
29:   // Used to postpone producing of indirect function pointer types
30:   // after all indirect calls info is collected
31:   struct SPIRVIndirectCall {
32:     const Type *RetTy = nullptr;
33:     SmallVector<Type *> ArgTys;
34:     SmallVector<Register> ArgRegs;
35:     Register Callee;
36:   };
37:   void produceIndirectPtrType(MachineIRBuilder &MIRBuilder,
38:                               const SPIRVIndirectCall &IC) const;
39:
40: public:
```
- EN: This range defines or declares important types such as SPIRVGlobalRegistry, SPIRVTargetLowering, SPIRVCallLowering, SPIRVIndirectCall, shaping the data model used by SPIRVCallLowering.h.
- CN: 这一段定义或声明了 SPIRVGlobalRegistry、SPIRVTargetLowering、SPIRVCallLowering、SPIRVIndirectCall 等关键类型，构成 SPIRVCallLowering.h 使用的数据模型。

### Lines 41-59
```cpp
41:   SPIRVCallLowering(const SPIRVTargetLowering &TLI, SPIRVGlobalRegistry *GR);
42:
43:   // Built OpReturn or OpReturnValue.
44:   bool lowerReturn(MachineIRBuilder &MIRBuiler, const Value *Val,
45:                    ArrayRef<Register> VRegs, FunctionLoweringInfo &FLI,
46:                    Register SwiftErrorVReg) const override;
47:
48:   // Build OpFunction, OpFunctionParameter, and any EntryPoint or Linkage data.
49:   bool lowerFormalArguments(MachineIRBuilder &MIRBuilder, const Function &F,
50:                             ArrayRef<ArrayRef<Register>> VRegs,
51:                             FunctionLoweringInfo &FLI) const override;
52:
53:   // Build OpCall, or replace with a builtin function.
54:   bool lowerCall(MachineIRBuilder &MIRBuilder,
55:                  CallLoweringInfo &Info) const override;
56: };
57: } // end namespace llvm
58:
59: #endif // LLVM_LIB_TARGET_SPIRV_SPIRVCALLLOWERING_H
```
- EN: This range declares interfaces or inline helpers such as SPIRVCallLowering, defining how other backend pieces interact with this header.
- CN: 这一段声明了 SPIRVCallLowering 等接口或内联辅助函数，定义其他后端组件如何与该头文件协作。

## Key Concepts / 关键概念

- EN: Lowering turns higher-level IR constructs into forms accepted by the target pipeline.
  - CN: 降级过程会把更高层的 IR 构造转换成目标流水线可接受的形式。
- EN: SPIR-V backend code must preserve rich type information and encode capabilities/extensions explicitly.
  - CN: SPIR-V 后端需要保留丰富的类型信息，并显式编码 capability 与 extension。
- EN: Key symbols in this file include SPIRVGlobalRegistry, SPIRVTargetLowering, SPIRVCallLowering, SPIRVIndirectCall, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 SPIRVGlobalRegistry, SPIRVTargetLowering, SPIRVCallLowering, SPIRVIndirectCall，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `SPIRVGlobalRegistry.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/CodeGen/GlobalISel/CallLowering.h`
