# SPIRVMetadata.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/SPIRV/SPIRVMetadata.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file contains functions needed for parsing LLVM IR metadata relevant to the SPIR-V target.
- 目的（中文）: 该文件用于支撑 LLVM 目标后端的相关功能。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-40
```cpp
 1: //===--- SPIRVMetadata.cpp ---- IR Metadata Parsing Funcs -------*- C++ -*-===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // This file contains functions needed for parsing LLVM IR metadata relevant
10: // to the SPIR-V target.
11: //
12: //===----------------------------------------------------------------------===//
13:
14: #include "SPIRVMetadata.h"
15:
16: using namespace llvm;
17:
18: static MDString *getOCLKernelArgAttribute(const Function &F, unsigned ArgIdx,
19:                                           const StringRef AttributeName) {
20:   assert(
21:       F.getCallingConv() == CallingConv::SPIR_KERNEL &&
22:       "Kernel attributes are attached/belong only to OpenCL kernel functions");
23:
24:   // Lookup the argument attribute in metadata attached to the kernel function.
25:   MDNode *Node = F.getMetadata(AttributeName);
26:   if (Node && ArgIdx < Node->getNumOperands())
27:     return cast<MDString>(Node->getOperand(ArgIdx));
28:
29:   // Sometimes metadata containing kernel attributes is not attached to the
30:   // function, but can be found in the named module-level metadata instead.
31:   // For example:
32:   //   !opencl.kernels = !{!0}
33:   //   !0 = !{void ()* @someKernelFunction, !1, ...}
34:   //   !1 = !{!"kernel_arg_addr_space", ...}
35:   // In this case the actual index of searched argument attribute is ArgIdx + 1,
36:   // since the first metadata node operand is occupied by attribute name
37:   // ("kernel_arg_addr_space" in the example above).
38:   unsigned MDArgIdx = ArgIdx + 1;
39:   NamedMDNode *OpenCLKernelsMD =
40:       F.getParent()->getNamedMetadata("opencl.kernels");
```
- EN: This range implements operational logic in helpers such as getMetadata, getOperand, getParent, translating backend policy into executable code.
- CN: 这一段实现了 getMetadata、getOperand、getParent 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 41-80
```cpp
41:   if (!OpenCLKernelsMD || OpenCLKernelsMD->getNumOperands() == 0)
42:     return nullptr;
43:
44:   // KernelToMDNodeList contains kernel function declarations followed by
45:   // corresponding MDNodes for each attribute. Search only MDNodes "belonging"
46:   // to the currently lowered kernel function.
47:   MDNode *KernelToMDNodeList = OpenCLKernelsMD->getOperand(0);
48:   bool FoundLoweredKernelFunction = false;
49:   for (const MDOperand &Operand : KernelToMDNodeList->operands()) {
50:     ValueAsMetadata *MaybeValue = dyn_cast<ValueAsMetadata>(Operand);
51:     if (MaybeValue &&
52:         dyn_cast<Function>(MaybeValue->getValue())->getName() == F.getName()) {
53:       FoundLoweredKernelFunction = true;
54:       continue;
55:     }
56:     if (MaybeValue && FoundLoweredKernelFunction)
57:       return nullptr;
58:
59:     MDNode *MaybeNode = dyn_cast<MDNode>(Operand);
60:     if (FoundLoweredKernelFunction && MaybeNode &&
61:         cast<MDString>(MaybeNode->getOperand(0))->getString() ==
62:             AttributeName &&
63:         MDArgIdx < MaybeNode->getNumOperands())
64:       return cast<MDString>(MaybeNode->getOperand(MDArgIdx));
65:   }
66:   return nullptr;
67: }
68:
69: namespace llvm {
70:
71: MDString *getOCLKernelArgAccessQual(const Function &F, unsigned ArgIdx) {
72:   assert(
73:       F.getCallingConv() == CallingConv::SPIR_KERNEL &&
74:       "Kernel attributes are attached/belong only to OpenCL kernel functions");
75:   return getOCLKernelArgAttribute(F, ArgIdx, "kernel_arg_access_qual");
76: }
77:
78: MDString *getOCLKernelArgTypeQual(const Function &F, unsigned ArgIdx) {
79:   assert(
80:       F.getCallingConv() == CallingConv::SPIR_KERNEL &&
```
- EN: This range implements operational logic in helpers such as getOperand, getValue, getNumOperands, getOCLKernelArgAccessQual, translating backend policy into executable code.
- CN: 这一段实现了 getOperand、getValue、getNumOperands、getOCLKernelArgAccessQual 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 81-85
```cpp
81:       "Kernel attributes are attached/belong only to OpenCL kernel functions");
82:   return getOCLKernelArgAttribute(F, ArgIdx, "kernel_arg_type_qual");
83: }
84:
85: } // namespace llvm
```
- EN: This range implements operational logic in helpers such as getOCLKernelArgAttribute, translating backend policy into executable code.
- CN: 这一段实现了 getOCLKernelArgAttribute 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: SPIR-V backend code must preserve rich type information and encode capabilities/extensions explicitly.
  - CN: SPIR-V 后端需要保留丰富的类型信息，并显式编码 capability 与 extension。
- EN: Key symbols in this file include getMetadata, getOperand, getParent, getValue, getNumOperands, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 getMetadata, getOperand, getParent, getValue, getNumOperands，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `SPIRVMetadata.h`
