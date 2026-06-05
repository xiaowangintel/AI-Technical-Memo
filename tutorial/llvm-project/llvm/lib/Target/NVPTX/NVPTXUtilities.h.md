# NVPTXUtilities.h — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/NVPTX/NVPTXUtilities.h`
- Repository: `llvm-project`
- Purpose (EN): This file contains declarations for PTX-specific utility functions.
- 目的（中文）: 该文件声明 LLVM 目标后端使用的数据结构、接口或辅助类型。
- Language: C++ header

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-40
```cpp
 1: //===-- NVPTXUtilities - Utilities -----------------------------*- C++ -*-====//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // This file contains declarations for PTX-specific utility functions.
10: //
11: //===----------------------------------------------------------------------===//
12:
13: #ifndef LLVM_LIB_TARGET_NVPTX_NVPTXUTILITIES_H
14: #define LLVM_LIB_TARGET_NVPTX_NVPTXUTILITIES_H
15:
16: #include "NVPTX.h"
17: #include "llvm/ADT/SmallVector.h"
18: #include "llvm/ADT/StringExtras.h"
19: #include "llvm/CodeGen/ValueTypes.h"
20: #include "llvm/IR/Function.h"
21: #include "llvm/IR/IntrinsicInst.h"
22: #include "llvm/IR/Value.h"
23: #include "llvm/Support/Alignment.h"
24: #include "llvm/Support/FormatVariadic.h"
25: #include <cstdarg>
26: #include <string>
27:
28: namespace llvm {
29:
30: class DataLayout;
31: class TargetMachine;
32:
33: Function *getMaybeBitcastedCallee(const CallBase *CB);
34:
35: /// Since function arguments are passed via .param space, we may want to
36: /// increase their alignment in a way that ensures that we can effectively
37: /// vectorize their loads & stores. We can increase alignment only if the
38: /// function has internal or private linkage as for other linkage types callers
39: /// may already rely on default alignment. To allow using 128-bit vectorized
40: /// loads/stores, this function ensures that alignment is 16 or greater.
```
- EN: This range defines or declares important types such as DataLayout, TargetMachine, getMaybeBitcastedCallee, shaping the data model used by NVPTXUtilities.h.
- CN: 这一段定义或声明了 DataLayout、TargetMachine、getMaybeBitcastedCallee 等关键类型，构成 NVPTXUtilities.h 使用的数据模型。

### Lines 41-80
```cpp
41: Align getFunctionParamOptimizedAlign(const Function *F, Type *ArgTy,
42:                                      const DataLayout &DL);
43:
44: Align getFunctionArgumentAlignment(const Function *F, Type *Ty, unsigned Idx,
45:                                    const DataLayout &DL);
46:
47: Align getFunctionByValParamAlign(const Function *F, Type *ArgTy,
48:                                  Align InitialAlign, const DataLayout &DL);
49:
50: // PTX ABI requires all scalar argument/return values to have
51: // bit-size as a power of two of at least 32 bits.
52: inline unsigned promoteScalarArgumentSize(unsigned size) {
53:   if (size <= 32)
54:     return 32;
55:   if (size <= 64)
56:     return 64;
57:   if (size <= 128)
58:     return 128;
59:   return size;
60: }
61:
62: bool shouldEmitPTXNoReturn(const Value *V, const TargetMachine &TM);
63:
64: inline bool shouldPassAsArray(Type *Ty) {
65:   return Ty->isAggregateType() || Ty->isVectorTy() ||
66:          Ty->getScalarSizeInBits() >= 128 || Ty->isHalfTy() || Ty->isBFloatTy();
67: }
68:
69: namespace NVPTX {
70: // Returns a list of vector types that we prefer to fit into a single PTX
71: // register. NOTE: This must be kept in sync with the register classes
72: // defined in NVPTXRegisterInfo.td.
73: inline auto packed_types() {
74:   static const auto PackedTypes = {MVT::v4i8,  MVT::v2f16, MVT::v2bf16,
75:                                    MVT::v2i16, MVT::v2f32, MVT::v2i32};
76:   return PackedTypes;
77: }
78:
79: // Checks if the type VT can fit into a single register.
80: inline bool isPackedVectorTy(EVT VT) {
```
- EN: This range declares interfaces or inline helpers such as promoteScalarArgumentSize, shouldEmitPTXNoReturn, shouldPassAsArray, getScalarSizeInBits, defining how other backend pieces interact with this header.
- CN: 这一段声明了 promoteScalarArgumentSize、shouldEmitPTXNoReturn、shouldPassAsArray、getScalarSizeInBits 等接口或内联辅助函数，定义其他后端组件如何与该头文件协作。

### Lines 81-120
```cpp
 81:   return any_of(packed_types(), equal_to(VT));
 82: }
 83:
 84: // Checks if two or more of the type ET can fit into a single register.
 85: inline bool isPackedElementTy(EVT ET) {
 86:   return any_of(packed_types(),
 87:                 [ET](EVT OVT) { return OVT.getVectorElementType() == ET; });
 88: }
 89:
 90: inline std::string getValidPTXIdentifier(StringRef Name) {
 91:   std::string ValidName;
 92:   ValidName.reserve(Name.size() + 4);
 93:   for (char C : Name)
 94:     // While PTX also allows '%' at the start of identifiers, LLVM will throw a
 95:     // fatal error for '%' in symbol names in MCSymbol::print. Exclude for now.
 96:     if (isAlnum(C) || C == '_' || C == '$')
 97:       ValidName.push_back(C);
 98:     else
 99:       ValidName.append({'_', '$', '_'});
100:
101:   return ValidName;
102: }
103:
104: inline std::string OrderingToString(Ordering Order) {
105:   switch (Order) {
106:   case Ordering::NotAtomic:
107:     return "NotAtomic";
108:   case Ordering::Relaxed:
109:     return "Relaxed";
110:   case Ordering::Acquire:
111:     return "Acquire";
112:   case Ordering::Release:
113:     return "Release";
114:   case Ordering::AcquireRelease:
115:     return "AcquireRelease";
116:   case Ordering::SequentiallyConsistent:
117:     return "SequentiallyConsistent";
118:   case Ordering::Volatile:
119:     return "Volatile";
120:   case Ordering::RelaxedMMIO:
```
- EN: This range declares interfaces or inline helpers such as any_of, isPackedElementTy, getValidPTXIdentifier, reserve, defining how other backend pieces interact with this header.
- CN: 这一段声明了 any_of、isPackedElementTy、getValidPTXIdentifier、reserve 等接口或内联辅助函数，定义其他后端组件如何与该头文件协作。

### Lines 121-160
```cpp
121:     return "RelaxedMMIO";
122:   }
123:   report_fatal_error(formatv("Unknown NVPTX::Ordering \"{}\".",
124:                              static_cast<OrderingUnderlyingType>(Order)));
125: }
126:
127: inline raw_ostream &operator<<(raw_ostream &O, Ordering Order) {
128:   O << OrderingToString(Order);
129:   return O;
130: }
131:
132: inline std::string ScopeToString(Scope S) {
133:   switch (S) {
134:   case Scope::Thread:
135:     return "Thread";
136:   case Scope::System:
137:     return "System";
138:   case Scope::Block:
139:     return "Block";
140:   case Scope::Cluster:
141:     return "Cluster";
142:   case Scope::Device:
143:     return "Device";
144:   case Scope::DefaultDevice:
145:     return "DefaultDevice";
146:   }
147:   report_fatal_error(formatv("Unknown NVPTX::Scope \"{}\".",
148:                              static_cast<ScopeUnderlyingType>(S)));
149: }
150:
151: inline raw_ostream &operator<<(raw_ostream &O, Scope S) {
152:   O << ScopeToString(S);
153:   return O;
154: }
155:
156: inline const char *addressSpaceToString(AddressSpace A,
157:                                         bool UseParamSubqualifiers = false) {
158:   switch (A) {
159:   case AddressSpace::Generic:
160:     return "generic";
```
- EN: This range declares interfaces or inline helpers such as OrderingToString, ScopeToString, defining how other backend pieces interact with this header.
- CN: 这一段声明了 OrderingToString、ScopeToString 等接口或内联辅助函数，定义其他后端组件如何与该头文件协作。

### Lines 161-188
```cpp
161:   case AddressSpace::Global:
162:     return "global";
163:   case AddressSpace::Const:
164:     return "const";
165:   case AddressSpace::Shared:
166:     return "shared";
167:   case AddressSpace::SharedCluster:
168:     return "shared::cluster";
169:   case AddressSpace::EntryParam:
170:     return UseParamSubqualifiers ? "param::entry" : "param";
171:   case AddressSpace::DeviceParam:
172:     return UseParamSubqualifiers ? "param::func" : "param";
173:   case AddressSpace::Local:
174:     return "local";
175:   }
176:   report_fatal_error(formatv("Unknown NVPTX::AddressSpace \"{}\".",
177:                              static_cast<AddressSpaceUnderlyingType>(A)));
178: }
179:
180: inline raw_ostream &operator<<(raw_ostream &O, AddressSpace A) {
181:   O << addressSpaceToString(A);
182:   return O;
183: }
184:
185: } // namespace NVPTX
186: } // namespace llvm
187:
188: #endif
```
- EN: This range declares interfaces or inline helpers such as addressSpaceToString, defining how other backend pieces interact with this header.
- CN: 这一段声明了 addressSpaceToString 等接口或内联辅助函数，定义其他后端组件如何与该头文件协作。

## Key Concepts / 关键概念

- EN: Utility files package reusable helpers so other backend components can stay focused on core compilation steps.
  - CN: 工具类文件封装可复用辅助逻辑，使其他后端组件能够聚焦核心编译步骤。
- EN: NVPTX backend code reflects GPU execution concerns such as address spaces, intrinsics, and PTX-specific lowering.
  - CN: NVPTX 后端代码体现了 GPU 执行特性，例如地址空间、内建函数以及 PTX 特定降级。
- EN: Key symbols in this file include DataLayout, TargetMachine, getMaybeBitcastedCallee, promoteScalarArgumentSize, shouldEmitPTXNoReturn, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 DataLayout, TargetMachine, getMaybeBitcastedCallee, promoteScalarArgumentSize, shouldEmitPTXNoReturn，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `NVPTX.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/ADT/SmallVector.h`
  - `llvm/ADT/StringExtras.h`
  - `llvm/CodeGen/ValueTypes.h`
  - `llvm/IR/Function.h`
  - `llvm/IR/IntrinsicInst.h`
  - `llvm/IR/Value.h`
  - `llvm/Support/Alignment.h`
  - `llvm/Support/FormatVariadic.h`
- System/standard headers / 系统或标准头文件:
  - `cstdarg`
  - `string`
