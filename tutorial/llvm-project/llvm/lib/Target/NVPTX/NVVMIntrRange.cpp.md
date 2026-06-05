# NVVMIntrRange.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/NVPTX/NVVMIntrRange.cpp`
- Repository: `llvm-project`
- Purpose (EN): This pass adds appropriate range attributes for calls to NVVM intrinsics that return a limited range of values.
- 目的（中文）: 该文件用于支撑 LLVM 目标后端的相关功能。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-40
```cpp
 1: //===- NVVMIntrRange.cpp - Set range attributes for NVVM intrinsics -------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // This pass adds appropriate range attributes for calls to NVVM
10: // intrinsics that return a limited range of values.
11: //
12: //===----------------------------------------------------------------------===//
13:
14: #include "NVPTX.h"
15: #include "NVVMProperties.h"
16: #include "llvm/IR/InstIterator.h"
17: #include "llvm/IR/Instructions.h"
18: #include "llvm/IR/IntrinsicInst.h"
19: #include "llvm/IR/Intrinsics.h"
20: #include "llvm/IR/IntrinsicsNVPTX.h"
21: #include "llvm/IR/PassManager.h"
22: #include <cstdint>
23:
24: using namespace llvm;
25:
26: #define DEBUG_TYPE "nvvm-intr-range"
27:
28: namespace {
29: class NVVMIntrRange : public FunctionPass {
30: public:
31:   static char ID;
32:   NVVMIntrRange() : FunctionPass(ID) {}
33:
34:   bool runOnFunction(Function &) override;
35: };
36: } // namespace
37:
38: FunctionPass *llvm::createNVVMIntrRangePass() { return new NVVMIntrRange(); }
39:
40: char NVVMIntrRange::ID = 0;
```
- EN: This range defines or declares important types such as NVVMIntrRange, runOnFunction, llvm::createNVVMIntrRangePass, shaping the data model used by NVVMIntrRange.cpp.
- CN: 这一段定义或声明了 NVVMIntrRange、runOnFunction、llvm::createNVVMIntrRangePass 等关键类型，构成 NVVMIntrRange.cpp 使用的数据模型。

### Lines 41-80
```cpp
41: INITIALIZE_PASS(NVVMIntrRange, "nvvm-intr-range",
42:                 "Add !range metadata to NVVM intrinsics.", false, false)
43:
44: // Adds the passed-in [Low,High) range information as metadata to the
45: // passed-in call instruction.
46: static bool addRangeAttr(uint64_t Low, uint64_t High, IntrinsicInst *II) {
47:   if (II->getMetadata(LLVMContext::MD_range))
48:     return false;
49:
50:   const uint64_t BitWidth = II->getType()->getIntegerBitWidth();
51:   ConstantRange Range(APInt(BitWidth, Low), APInt(BitWidth, High));
52:
53:   if (auto CurrentRange = II->getRange())
54:     Range = Range.intersectWith(CurrentRange.value());
55:
56:   II->addRangeRetAttr(Range);
57:   return true;
58: }
59:
60: static bool runNVVMIntrRange(Function &F) {
61:   struct Vector3 {
62:     unsigned X, Y, Z;
63:   };
64:
65:   // All these annotations are only valid for kernel functions.
66:   if (!isKernelFunction(F))
67:     return false;
68:
69:   auto ReqNTID = getReqNTID(F);
70:   const auto OverallMaxNTID = getOverallMaxNTID(F);
71:   const auto OverallClusterRank = getOverallClusterRank(F);
72:
73:   // If this function lacks any range information, do nothing.
74:   if (!(!ReqNTID.empty() || OverallMaxNTID || OverallClusterRank))
75:     return false;
76:
77:   const unsigned MaxNTID =
78:       OverallMaxNTID.value_or(std::numeric_limits<unsigned>::max());
79:
80:   const unsigned FunctionClusterRank =
```
- EN: This range defines or declares important types such as addRangeAttr, getType, Range, intersectWith, shaping the data model used by NVVMIntrRange.cpp.
- CN: 这一段定义或声明了 addRangeAttr、getType、Range、intersectWith 等关键类型，构成 NVVMIntrRange.cpp 使用的数据模型。

### Lines 81-120
```cpp
 81:       OverallClusterRank.value_or(std::numeric_limits<unsigned>::max());
 82:
 83:   // When reqntid is specified, block dimensions are exact compile-time
 84:   // constants. Otherwise, use maxntid (capped at hardware limits) as upper
 85:   // bounds.
 86:   Vector3 MinBlockDim, MaxBlockDim;
 87:   if (!ReqNTID.empty()) {
 88:     ReqNTID.resize(3, 1);
 89:     MinBlockDim = MaxBlockDim = {ReqNTID[0], ReqNTID[1], ReqNTID[2]};
 90:   } else {
 91:     MinBlockDim = {1, 1, 1};
 92:     MaxBlockDim = {std::min(1024u, MaxNTID), std::min(1024u, MaxNTID),
 93:                    std::min(64u, MaxNTID)};
 94:   }
 95:
 96:   // We conservatively use the maximum grid size as an upper bound for the
 97:   // cluster rank.
 98:   const Vector3 MaxClusterRank{std::min(0x7fffffffu, FunctionClusterRank),
 99:                                std::min(0xffffu, FunctionClusterRank),
100:                                std::min(0xffffu, FunctionClusterRank)};
101:
102:   const auto ProcessIntrinsic = [&](IntrinsicInst *II) -> bool {
103:     switch (II->getIntrinsicID()) {
104:     // Index within block
105:     case Intrinsic::nvvm_read_ptx_sreg_tid_x:
106:       return addRangeAttr(0, MaxBlockDim.X, II);
107:     case Intrinsic::nvvm_read_ptx_sreg_tid_y:
108:       return addRangeAttr(0, MaxBlockDim.Y, II);
109:     case Intrinsic::nvvm_read_ptx_sreg_tid_z:
110:       return addRangeAttr(0, MaxBlockDim.Z, II);
111:
112:     // Block size: use single-value range when reqntid is specified;
113:     // InstCombine will fold these to constants later.
114:     case Intrinsic::nvvm_read_ptx_sreg_ntid_x:
115:       return addRangeAttr(MinBlockDim.X, MaxBlockDim.X + 1, II);
116:     case Intrinsic::nvvm_read_ptx_sreg_ntid_y:
117:       return addRangeAttr(MinBlockDim.Y, MaxBlockDim.Y + 1, II);
118:     case Intrinsic::nvvm_read_ptx_sreg_ntid_z:
119:       return addRangeAttr(MinBlockDim.Z, MaxBlockDim.Z + 1, II);
120:
```
- EN: This range implements operational logic in helpers such as value_or, resize, addRangeAttr, translating backend policy into executable code.
- CN: 这一段实现了 value_or、resize、addRangeAttr 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 121-160
```cpp
121:     // Cluster size
122:     case Intrinsic::nvvm_read_ptx_sreg_cluster_ctaid_x:
123:       return addRangeAttr(0, MaxClusterRank.X, II);
124:     case Intrinsic::nvvm_read_ptx_sreg_cluster_ctaid_y:
125:       return addRangeAttr(0, MaxClusterRank.Y, II);
126:     case Intrinsic::nvvm_read_ptx_sreg_cluster_ctaid_z:
127:       return addRangeAttr(0, MaxClusterRank.Z, II);
128:     case Intrinsic::nvvm_read_ptx_sreg_cluster_nctaid_x:
129:       return addRangeAttr(1, MaxClusterRank.X + 1, II);
130:     case Intrinsic::nvvm_read_ptx_sreg_cluster_nctaid_y:
131:       return addRangeAttr(1, MaxClusterRank.Y + 1, II);
132:     case Intrinsic::nvvm_read_ptx_sreg_cluster_nctaid_z:
133:       return addRangeAttr(1, MaxClusterRank.Z + 1, II);
134:
135:     case Intrinsic::nvvm_read_ptx_sreg_cluster_ctarank:
136:       if (OverallClusterRank)
137:         return addRangeAttr(0, FunctionClusterRank, II);
138:       break;
139:     case Intrinsic::nvvm_read_ptx_sreg_cluster_nctarank:
140:       if (OverallClusterRank)
141:         return addRangeAttr(1, FunctionClusterRank + 1, II);
142:       break;
143:     default:
144:       return false;
145:     }
146:     return false;
147:   };
148:
149:   // Go through the calls in this function.
150:   bool Changed = false;
151:   for (Instruction &I : instructions(F))
152:     if (IntrinsicInst *II = dyn_cast<IntrinsicInst>(&I))
153:       Changed |= ProcessIntrinsic(II);
154:
155:   return Changed;
156: }
157:
158: bool NVVMIntrRange::runOnFunction(Function &F) { return runNVVMIntrRange(F); }
159:
160: PreservedAnalyses NVVMIntrRangePass::run(Function &F,
```
- EN: This range implements operational logic in helpers such as addRangeAttr, ProcessIntrinsic, NVVMIntrRange::runOnFunction, translating backend policy into executable code.
- CN: 这一段实现了 addRangeAttr、ProcessIntrinsic、NVVMIntrRange::runOnFunction 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 161-164
```cpp
161:                                          FunctionAnalysisManager &AM) {
162:   return runNVVMIntrRange(F) ? PreservedAnalyses::none()
163:                              : PreservedAnalyses::all();
164: }
```
- EN: This range implements operational logic in helpers such as runNVVMIntrRange, PreservedAnalyses::all, translating backend policy into executable code.
- CN: 这一段实现了 runNVVMIntrRange、PreservedAnalyses::all 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: NVPTX backend code reflects GPU execution concerns such as address spaces, intrinsics, and PTX-specific lowering.
  - CN: NVPTX 后端代码体现了 GPU 执行特性，例如地址空间、内建函数以及 PTX 特定降级。
- EN: Key symbols in this file include NVVMIntrRange, runOnFunction, llvm::createNVVMIntrRangePass, addRangeAttr, getType, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 NVVMIntrRange, runOnFunction, llvm::createNVVMIntrRangePass, addRangeAttr, getType，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `NVPTX.h`
  - `NVVMProperties.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/IR/InstIterator.h`
  - `llvm/IR/Instructions.h`
  - `llvm/IR/IntrinsicInst.h`
  - `llvm/IR/Intrinsics.h`
  - `llvm/IR/IntrinsicsNVPTX.h`
  - `llvm/IR/PassManager.h`
- System/standard headers / 系统或标准头文件:
  - `cstdint`
