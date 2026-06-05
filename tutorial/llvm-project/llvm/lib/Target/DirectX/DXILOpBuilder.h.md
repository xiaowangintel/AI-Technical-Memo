# DXILOpBuilder.h — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/DirectX/DXILOpBuilder.h`
- Repository: `llvm-project`
- Purpose (EN): This file contains class to help build DXIL op functions.
- 目的（中文）: 该文件声明 LLVM 目标后端使用的数据结构、接口或辅助类型。
- Language: C++ header

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-40
```cpp
 1: //===- DXILOpBuilder.h - Helper class for build DIXLOp functions ----------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: ///
 9: /// \file This file contains class to help build DXIL op functions.
10: //===----------------------------------------------------------------------===//
11:
12: #ifndef LLVM_LIB_TARGET_DIRECTX_DXILOPBUILDER_H
13: #define LLVM_LIB_TARGET_DIRECTX_DXILOPBUILDER_H
14:
15: #include "DXILConstants.h"
16: #include "llvm/ADT/SmallVector.h"
17: #include "llvm/IR/IRBuilder.h"
18: #include "llvm/Support/DXILABI.h"
19: #include "llvm/Support/Error.h"
20: #include "llvm/TargetParser/Triple.h"
21:
22: namespace llvm {
23: class Module;
24: class IRBuilderBase;
25: class CallInst;
26: class Constant;
27: class Value;
28: class Type;
29: class FunctionType;
30:
31: namespace dxil {
32:
33: class DXILOpBuilder {
34: public:
35:   DXILOpBuilder(Module &M);
36:
37:   IRBuilder<> &getIRB() { return IRB; }
38:
39:   /// Create a call instruction for the given DXIL op. The arguments
40:   /// must be valid for an overload of the operation.
```
- EN: This range defines or declares important types such as Module, IRBuilderBase, CallInst, Constant, shaping the data model used by DXILOpBuilder.h.
- CN: 这一段定义或声明了 Module、IRBuilderBase、CallInst、Constant 等关键类型，构成 DXILOpBuilder.h 使用的数据模型。

### Lines 41-80
```cpp
41:   CallInst *createOp(dxil::OpCode Op, ArrayRef<Value *> Args,
42:                      const Twine &Name = "", Type *RetTy = nullptr);
43:
44:   /// Try to create a call instruction for the given DXIL op. Fails if the
45:   /// overload is invalid.
46:   Expected<CallInst *> tryCreateOp(dxil::OpCode Op, ArrayRef<Value *> Args,
47:                                    const Twine &Name = "",
48:                                    Type *RetTy = nullptr);
49:
50:   /// Get a `%dx.types.ResRet` type with the given element type.
51:   StructType *getResRetType(Type *ElementTy);
52:
53:   /// Get a `%dx.types.CBufRet` type with the given element type.
54:   StructType *getCBufRetType(Type *ElementTy);
55:
56:   /// Get the `%dx.types.Handle` type.
57:   StructType *getHandleType();
58:
59:   /// Get a constant `%dx.types.ResBind` value.
60:   Constant *getResBind(uint32_t LowerBound, uint32_t UpperBound,
61:                        uint32_t SpaceID, dxil::ResourceClass RC);
62:   /// Get a constant `%dx.types.ResourceProperties` value.
63:   Constant *getResProps(uint32_t Word0, uint32_t Word1);
64:
65:   /// Return the name of the given opcode.
66:   static const char *getOpCodeName(dxil::OpCode DXILOp);
67:
68: private:
69:   /// Gets a specific overload type of the function for the given DXIL op. If
70:   /// the operation is not overloaded, \c OverloadType may be nullptr.
71:   FunctionType *getOpFunctionType(dxil::OpCode OpCode,
72:                                   Type *OverloadType = nullptr);
73:
74:   Module &M;
75:   IRBuilder<> IRB;
76:   VersionTuple DXILVersion;
77:   Triple::EnvironmentType ShaderStage;
78: };
79:
80: } // namespace dxil
```
- EN: This range declares interfaces or inline helpers such as getResRetType, getCBufRetType, getHandleType, getResProps, defining how other backend pieces interact with this header.
- CN: 这一段声明了 getResRetType、getCBufRetType、getHandleType、getResProps 等接口或内联辅助函数，定义其他后端组件如何与该头文件协作。

### Lines 81-83
```cpp
81: } // namespace llvm
82:
83: #endif
```
- EN: This range contains connective backend logic, including helper statements, control flow, and data movement needed by the surrounding implementation.
- CN: 这一段包含衔接性的后端逻辑，包括辅助语句、控制流和数据处理，用于支撑周围实现。

## Key Concepts / 关键概念

- EN: DirectX/DXIL backend code models HLSL/DXIL semantics, including resources, shader stages, and validation-friendly IR forms.
  - CN: DirectX/DXIL 后端代码建模了 HLSL/DXIL 语义，包括资源、着色器阶段以及便于验证的 IR 形式。
- EN: Key symbols in this file include Module, IRBuilderBase, CallInst, Constant, Value, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 Module, IRBuilderBase, CallInst, Constant, Value，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `DXILConstants.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/ADT/SmallVector.h`
  - `llvm/IR/IRBuilder.h`
  - `llvm/Support/DXILABI.h`
  - `llvm/Support/Error.h`
  - `llvm/TargetParser/Triple.h`
