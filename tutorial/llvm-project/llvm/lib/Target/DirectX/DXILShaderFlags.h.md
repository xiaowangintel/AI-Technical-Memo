# DXILShaderFlags.h — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/DirectX/DXILShaderFlags.h`
- Repository: `llvm-project`
- Purpose (EN): This file contains helper objects and APIs for working with DXIL Shader Flags.
- 目的（中文）: 该文件声明 LLVM 目标后端使用的数据结构、接口或辅助类型。
- Language: C++ header

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-40
```cpp
 1: //===- DXILShaderFlags.h - DXIL Shader Flags helper objects ---------------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: ///
 9: /// \file This file contains helper objects and APIs for working with DXIL
10: ///       Shader Flags.
11: ///
12: //===----------------------------------------------------------------------===//
13:
14: #ifndef LLVM_TARGET_DIRECTX_DXILSHADERFLAGS_H
15: #define LLVM_TARGET_DIRECTX_DXILSHADERFLAGS_H
16:
17: #include "llvm/Analysis/DXILMetadataAnalysis.h"
18: #include "llvm/IR/Function.h"
19: #include "llvm/IR/PassManager.h"
20: #include "llvm/Pass.h"
21: #include "llvm/Support/Compiler.h"
22: #include "llvm/Support/Debug.h"
23: #include "llvm/Support/raw_ostream.h"
24: #include <cstdint>
25:
26: namespace llvm {
27: class Module;
28: class GlobalVariable;
29: class DXILResourceTypeMap;
30: class DXILResourceMap;
31:
32: namespace dxil {
33:
34: struct ComputedShaderFlags {
35: #define SHADER_FEATURE_FLAG(FeatureBit, DxilModuleBit, FlagName, Str)          \
36:   bool FlagName : 1;
37: #define DXIL_MODULE_FLAG(DxilModuleBit, FlagName, Str) bool FlagName : 1;
38: #include "llvm/BinaryFormat/DXContainerConstants.def"
39:
40: #define SHADER_FEATURE_FLAG(FeatureBit, DxilModuleBit, FlagName, Str)          \
```
- EN: This range defines or declares important types such as Module, GlobalVariable, DXILResourceTypeMap, DXILResourceMap, shaping the data model used by DXILShaderFlags.h.
- CN: 这一段定义或声明了 Module、GlobalVariable、DXILResourceTypeMap、DXILResourceMap 等关键类型，构成 DXILShaderFlags.h 使用的数据模型。

### Lines 41-80
```cpp
41:   FlagName = false;
42: #define DXIL_MODULE_FLAG(DxilModuleBit, FlagName, Str) FlagName = false;
43:   ComputedShaderFlags() {
44: #include "llvm/BinaryFormat/DXContainerConstants.def"
45:   }
46:
47:   constexpr uint64_t getMask(int Bit) const {
48:     return Bit != -1 ? 1ull << Bit : 0;
49:   }
50:
51:   uint64_t getModuleFlags() const {
52:     uint64_t ModuleFlags = 0;
53: #define DXIL_MODULE_FLAG(DxilModuleBit, FlagName, Str)                         \
54:   ModuleFlags |= FlagName ? getMask(DxilModuleBit) : 0ull;
55: #include "llvm/BinaryFormat/DXContainerConstants.def"
56:     return ModuleFlags;
57:   }
58:
59:   operator uint64_t() const {
60:     uint64_t FlagValue = getModuleFlags();
61: #define SHADER_FEATURE_FLAG(FeatureBit, DxilModuleBit, FlagName, Str)          \
62:   FlagValue |= FlagName ? getMask(DxilModuleBit) : 0ull;
63: #include "llvm/BinaryFormat/DXContainerConstants.def"
64:     return FlagValue;
65:   }
66:
67:   uint64_t getFeatureFlags() const {
68:     uint64_t FeatureFlags = 0;
69: #define SHADER_FEATURE_FLAG(FeatureBit, DxilModuleBit, FlagName, Str)          \
70:   FeatureFlags |= FlagName ? getMask(FeatureBit) : 0ull;
71: #include "llvm/BinaryFormat/DXContainerConstants.def"
72:     return FeatureFlags;
73:   }
74:
75:   void merge(const ComputedShaderFlags CSF) {
76: #define SHADER_FEATURE_FLAG(FeatureBit, DxilModuleBit, FlagName, Str)          \
77:   FlagName |= CSF.FlagName;
78: #define DXIL_MODULE_FLAG(DxilModuleBit, FlagName, Str) FlagName |= CSF.FlagName;
79: #include "llvm/BinaryFormat/DXContainerConstants.def"
80:   }
```
- EN: This range declares interfaces or inline helpers such as ComputedShaderFlags, getMask, getModuleFlags, uint64_t, defining how other backend pieces interact with this header.
- CN: 这一段声明了 ComputedShaderFlags、getMask、getModuleFlags、uint64_t 等接口或内联辅助函数，定义其他后端组件如何与该头文件协作。

### Lines 81-120
```cpp
 81:
 82:   void print(raw_ostream &OS = dbgs()) const;
 83:   LLVM_DUMP_METHOD void dump() const { print(); }
 84: };
 85:
 86: struct ModuleShaderFlags {
 87:   void initialize(Module &, DXILResourceTypeMap &DRTM,
 88:                   const DXILResourceMap &DRM, const ModuleMetadataInfo &MMDI);
 89:   const ComputedShaderFlags &getFunctionFlags(const Function *) const;
 90:   const ComputedShaderFlags &getCombinedFlags() const { return CombinedSFMask; }
 91:
 92: private:
 93:   // This boolean is inversely set by the LLVM module flag dx.resmayalias to
 94:   // determine whether or not the ResMayNotAlias DXIL module flag can be set
 95:   bool CanSetResMayNotAlias;
 96:
 97:   /// Map of Function-Shader Flag Mask pairs representing properties of each of
 98:   /// the functions in the module. Shader Flags of each function represent both
 99:   /// module-level and function-level flags
100:   DenseMap<const Function *, ComputedShaderFlags> FunctionFlags;
101:   /// Combined Shader Flag Mask of all functions of the module
102:   ComputedShaderFlags CombinedSFMask{};
103:   ComputedShaderFlags gatherGlobalModuleFlags(const Module &M,
104:                                               const DXILResourceMap &,
105:                                               const ModuleMetadataInfo &);
106:   void updateFunctionFlags(ComputedShaderFlags &, const Instruction &,
107:                            DXILResourceTypeMap &, const ModuleMetadataInfo &);
108: };
109:
110: class ShaderFlagsAnalysis : public AnalysisInfoMixin<ShaderFlagsAnalysis> {
111:   friend AnalysisInfoMixin<ShaderFlagsAnalysis>;
112:   static AnalysisKey Key;
113:
114: public:
115:   ShaderFlagsAnalysis() = default;
116:
117:   using Result = ModuleShaderFlags;
118:
119:   ModuleShaderFlags run(Module &M, ModuleAnalysisManager &AM);
120: };
```
- EN: This range defines or declares important types such as print, dump, ModuleShaderFlags, getFunctionFlags, shaping the data model used by DXILShaderFlags.h.
- CN: 这一段定义或声明了 print、dump、ModuleShaderFlags、getFunctionFlags 等关键类型，构成 DXILShaderFlags.h 使用的数据模型。

### Lines 121-154
```cpp
121:
122: /// Printer pass for ShaderFlagsAnalysis results.
123: class ShaderFlagsAnalysisPrinter
124:     : public OptionalPassInfoMixin<ShaderFlagsAnalysisPrinter> {
125:   raw_ostream &OS;
126:
127: public:
128:   explicit ShaderFlagsAnalysisPrinter(raw_ostream &OS) : OS(OS) {}
129:   PreservedAnalyses run(Module &M, ModuleAnalysisManager &AM);
130: };
131:
132: /// Wrapper pass for the legacy pass manager.
133: ///
134: /// This is required because the passes that will depend on this are codegen
135: /// passes which run through the legacy pass manager.
136: class ShaderFlagsAnalysisWrapper : public ModulePass {
137:   ModuleShaderFlags MSFI;
138:
139: public:
140:   static char ID;
141:
142:   ShaderFlagsAnalysisWrapper() : ModulePass(ID) {}
143:
144:   const ModuleShaderFlags &getShaderFlags() { return MSFI; }
145:
146:   bool runOnModule(Module &M) override;
147:
148:   void getAnalysisUsage(AnalysisUsage &AU) const override;
149: };
150:
151: } // namespace dxil
152: } // namespace llvm
153:
154: #endif // LLVM_TARGET_DIRECTX_DXILSHADERFLAGS_H
```
- EN: This range defines or declares important types such as ShaderFlagsAnalysisPrinter, run, ShaderFlagsAnalysisWrapper, getShaderFlags, shaping the data model used by DXILShaderFlags.h.
- CN: 这一段定义或声明了 ShaderFlagsAnalysisPrinter、run、ShaderFlagsAnalysisWrapper、getShaderFlags 等关键类型，构成 DXILShaderFlags.h 使用的数据模型。

## Key Concepts / 关键概念

- EN: DirectX/DXIL backend code models HLSL/DXIL semantics, including resources, shader stages, and validation-friendly IR forms.
  - CN: DirectX/DXIL 后端代码建模了 HLSL/DXIL 语义，包括资源、着色器阶段以及便于验证的 IR 形式。
- EN: Key symbols in this file include Module, GlobalVariable, DXILResourceTypeMap, DXILResourceMap, ComputedShaderFlags, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 Module, GlobalVariable, DXILResourceTypeMap, DXILResourceMap, ComputedShaderFlags，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- LLVM infrastructure / LLVM 基础设施:
  - `llvm/Analysis/DXILMetadataAnalysis.h`
  - `llvm/IR/Function.h`
  - `llvm/IR/PassManager.h`
  - `llvm/Pass.h`
  - `llvm/Support/Compiler.h`
  - `llvm/Support/Debug.h`
  - `llvm/Support/raw_ostream.h`
  - `llvm/BinaryFormat/DXContainerConstants.def`
  - `llvm/BinaryFormat/DXContainerConstants.def`
  - `llvm/BinaryFormat/DXContainerConstants.def`
  - `llvm/BinaryFormat/DXContainerConstants.def`
  - `llvm/BinaryFormat/DXContainerConstants.def`
  - `llvm/BinaryFormat/DXContainerConstants.def`
- System/standard headers / 系统或标准头文件:
  - `cstdint`
