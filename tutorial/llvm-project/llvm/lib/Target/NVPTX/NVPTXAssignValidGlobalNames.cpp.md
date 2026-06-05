# NVPTXAssignValidGlobalNames.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/NVPTX/NVPTXAssignValidGlobalNames.cpp`
- Repository: `llvm-project`
- Purpose (EN): Clean up the names of global variables in the module to not contain symbols that are invalid in PTX.
- 目的（中文）: 该文件用于支撑 LLVM 目标后端的相关功能。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-40
```cpp
 1: //===-- NVPTXAssignValidGlobalNames.cpp - Assign valid names to globals ---===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // Clean up the names of global variables in the module to not contain symbols
10: // that are invalid in PTX.
11: //
12: // Currently NVPTX, like other backends, relies on generic symbol name
13: // sanitizing done by MC. However, the ptxas assembler is more stringent and
14: // disallows some additional characters in symbol names. This pass makes sure
15: // such names do not reach MC at all.
16: //
17: //===----------------------------------------------------------------------===//
18:
19: #include "NVPTX.h"
20: #include "NVPTXUtilities.h"
21: #include "llvm/IR/Function.h"
22: #include "llvm/IR/GlobalVariable.h"
23: #include "llvm/IR/LegacyPassManager.h"
24: #include "llvm/IR/Module.h"
25:
26: using namespace llvm;
27:
28: namespace {
29: /// NVPTXAssignValidGlobalNames
30: class NVPTXAssignValidGlobalNames : public ModulePass {
31: public:
32:   static char ID;
33:   NVPTXAssignValidGlobalNames() : ModulePass(ID) {}
34:
35:   bool runOnModule(Module &M) override;
36: };
37: } // namespace
38:
39: char NVPTXAssignValidGlobalNames::ID = 0;
40:
```
- EN: This range defines or declares important types such as NVPTXAssignValidGlobalNames, runOnModule, shaping the data model used by NVPTXAssignValidGlobalNames.cpp.
- CN: 这一段定义或声明了 NVPTXAssignValidGlobalNames、runOnModule 等关键类型，构成 NVPTXAssignValidGlobalNames.cpp 使用的数据模型。

### Lines 41-71
```cpp
41: INITIALIZE_PASS(NVPTXAssignValidGlobalNames, "nvptx-assign-valid-global-names",
42:                 "Assign valid PTX names to globals", false, false)
43:
44: bool NVPTXAssignValidGlobalNames::runOnModule(Module &M) {
45:   for (GlobalVariable &GV : M.globals()) {
46:     // We are only allowed to rename symbols that are not externally linked by
47:     // name
48:     // - local symbols, as all references will be renamed
49:     // - .extern .shared symbols, as they're the same regardless of name
50:     if (GV.hasLocalLinkage() ||
51:         (GV.hasExternalLinkage() &&
52:          GV.getAddressSpace() == NVPTX::AddressSpace::Shared)) {
53:       // setName doesn't do extra work if the name does not change.
54:       // Note: this does not create collisions - if setName is asked to set the
55:       // name to something that already exists, it adds a proper postfix to
56:       // avoid collisions.
57:       GV.setName(NVPTX::getValidPTXIdentifier(GV.getName()));
58:     }
59:   }
60:
61:   // Do the same for local functions.
62:   for (Function &F : M.functions())
63:     if (F.hasLocalLinkage())
64:       F.setName(NVPTX::getValidPTXIdentifier(F.getName()));
65:
66:   return true;
67: }
68:
69: ModulePass *llvm::createNVPTXAssignValidGlobalNamesPass() {
70:   return new NVPTXAssignValidGlobalNames();
71: }
```
- EN: This range implements operational logic in helpers such as NVPTXAssignValidGlobalNames::runOnModule, getAddressSpace, setName, llvm::createNVPTXAssignValidGlobalNamesPass, translating backend policy into executable code.
- CN: 这一段实现了 NVPTXAssignValidGlobalNames::runOnModule、getAddressSpace、setName、llvm::createNVPTXAssignValidGlobalNamesPass 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: NVPTX backend code reflects GPU execution concerns such as address spaces, intrinsics, and PTX-specific lowering.
  - CN: NVPTX 后端代码体现了 GPU 执行特性，例如地址空间、内建函数以及 PTX 特定降级。
- EN: Key symbols in this file include NVPTXAssignValidGlobalNames, runOnModule, NVPTXAssignValidGlobalNames::runOnModule, getAddressSpace, setName, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 NVPTXAssignValidGlobalNames, runOnModule, NVPTXAssignValidGlobalNames::runOnModule, getAddressSpace, setName，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `NVPTX.h`
  - `NVPTXUtilities.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/IR/Function.h`
  - `llvm/IR/GlobalVariable.h`
  - `llvm/IR/LegacyPassManager.h`
  - `llvm/IR/Module.h`
