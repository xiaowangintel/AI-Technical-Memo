# SPIRVBuiltins.h — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/SPIRV/SPIRVBuiltins.h`
- Repository: `llvm-project`
- Purpose (EN): Lowering builtin function calls and types using their demangled names.
- 目的（中文）: 该文件声明 LLVM 目标后端使用的数据结构、接口或辅助类型。
- Language: C++ header

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-40
```cpp
 1: //===-- SPIRVBuiltins.h - SPIR-V Built-in Functions -------------*- C++ -*-===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // Lowering builtin function calls and types using their demangled names.
10: //
11: //===----------------------------------------------------------------------===//
12:
13: #ifndef LLVM_LIB_TARGET_SPIRV_SPIRVBUILTINS_H
14: #define LLVM_LIB_TARGET_SPIRV_SPIRVBUILTINS_H
15:
16: #include "SPIRVGlobalRegistry.h"
17: #include "llvm/CodeGen/GlobalISel/CallLowering.h"
18: #include "llvm/CodeGen/GlobalISel/MachineIRBuilder.h"
19:
20: namespace llvm {
21: namespace SPIRV {
22: /// Parses the name part of the demangled builtin call.
23: std::string lookupBuiltinNameHelper(StringRef DemangledCall,
24:                                     FPDecorationId *DecorationId = nullptr);
25: /// Lowers a builtin function call using the provided \p DemangledCall skeleton
26: /// and external instruction \p Set.
27: ///
28: /// \return the lowering success status if the called function is a recognized
29: /// builtin, std::nullopt otherwise.
30: ///
31: /// \p DemangledCall is the skeleton of the lowered builtin function call.
32: /// \p Set is the external instruction set containing the given builtin.
33: /// \p OrigRet is the single original virtual return register if defined,
34: /// Register(0) otherwise.
35: /// \p OrigRetTy is the type of the \p OrigRet.
36: /// \p Args are the arguments of the lowered builtin call.
37: std::optional<bool> lowerBuiltin(const StringRef DemangledCall,
38:                                  InstructionSet::InstructionSet Set,
39:                                  MachineIRBuilder &MIRBuilder,
40:                                  const Register OrigRet, const Type *OrigRetTy,
```
- EN: This range declares interfaces or inline helpers such as backend logic, defining how other backend pieces interact with this header.
- CN: 这一段声明了 后端逻辑 等接口或内联辅助函数，定义其他后端组件如何与该头文件协作。

### Lines 41-80
```cpp
41:                                  const SmallVectorImpl<Register> &Args,
42:                                  SPIRVGlobalRegistry *GR, const CallBase &CB);
43:
44: /// Helper function for finding a builtin function attributes
45: /// by a demangled function name. Defined in SPIRVBuiltins.cpp.
46: std::tuple<int, unsigned, unsigned>
47: mapBuiltinToOpcode(const StringRef DemangledCall,
48:                    SPIRV::InstructionSet::InstructionSet Set);
49:
50: /// Parses the provided \p ArgIdx argument base type in the \p DemangledCall
51: /// skeleton. A base type is either a basic type (e.g. i32 for int), pointer
52: /// element type (e.g. i8 for char*), or builtin type (TargetExtType).
53: ///
54: /// \return LLVM Type or nullptr if unrecognized
55: ///
56: /// \p DemangledCall is the skeleton of the lowered builtin function call.
57: /// \p ArgIdx is the index of the argument to parse.
58: Type *parseBuiltinCallArgumentBaseType(const StringRef DemangledCall,
59:                                        unsigned ArgIdx, LLVMContext &Ctx);
60: bool parseBuiltinTypeStr(SmallVector<StringRef, 10> &BuiltinArgsTypeStrs,
61:                          const StringRef DemangledCall, LLVMContext &Ctx);
62: Type *parseBuiltinCallArgumentType(StringRef TypeStr, LLVMContext &Ctx);
63:
64: /// Translates a string representing a SPIR-V or OpenCL builtin type to a
65: /// TargetExtType that can be further lowered with lowerBuiltinType().
66: ///
67: /// \return A TargetExtType representing the builtin SPIR-V type.
68: ///
69: /// \p TypeName is the full string representation of the SPIR-V or OpenCL
70: /// builtin type.
71: TargetExtType *parseBuiltinTypeNameToTargetExtType(std::string TypeName,
72:                                                    LLVMContext &Context);
73:
74: /// Handles the translation of the provided special opaque/builtin type \p Type
75: /// to SPIR-V type. Generates the corresponding machine instructions for the
76: /// target type or gets the already existing OpType<...> register from the
77: /// global registry \p GR.
78: ///
79: /// \return A machine instruction representing the OpType<...> SPIR-V type.
80: ///
```
- EN: This range declares interfaces or inline helpers such as parseBuiltinCallArgumentType, defining how other backend pieces interact with this header.
- CN: 这一段声明了 parseBuiltinCallArgumentType 等接口或内联辅助函数，定义其他后端组件如何与该头文件协作。

### Lines 81-88
```cpp
81: /// \p Type is the special opaque/builtin type to be lowered.
82: SPIRVTypeInst lowerBuiltinType(const Type *Type,
83:                                AccessQualifier::AccessQualifier AccessQual,
84:                                MachineIRBuilder &MIRBuilder,
85:                                SPIRVGlobalRegistry *GR);
86: } // namespace SPIRV
87: } // namespace llvm
88: #endif // LLVM_LIB_TARGET_SPIRV_SPIRVBUILTINS_H
```
- EN: This range declares interfaces or inline helpers such as backend logic, defining how other backend pieces interact with this header.
- CN: 这一段声明了 后端逻辑 等接口或内联辅助函数，定义其他后端组件如何与该头文件协作。

## Key Concepts / 关键概念

- EN: SPIR-V backend code must preserve rich type information and encode capabilities/extensions explicitly.
  - CN: SPIR-V 后端需要保留丰富的类型信息，并显式编码 capability 与 extension。
- EN: Key symbols in this file include parseBuiltinCallArgumentType, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 parseBuiltinCallArgumentType，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `SPIRVGlobalRegistry.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/CodeGen/GlobalISel/CallLowering.h`
  - `llvm/CodeGen/GlobalISel/MachineIRBuilder.h`
