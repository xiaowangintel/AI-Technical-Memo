# SanitizerBinaryMetadata.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/CodeGen/SanitizerBinaryMetadata.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: This file contains `----------------------------------------------===//` logic inside LLVM CodeGen.
- **Purpose (CN)**: 该文件在 LLVM CodeGen 中实现与“----------------------------------------------===//”相关的逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

````cpp
//===- SanitizerBinaryMetadata.cpp
//----------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file is a part of SanitizerBinaryMetadata.
//
//===----------------------------------------------------------------------===//

#include "llvm/CodeGen/SanitizerBinaryMetadata.h"
#include "llvm/CodeGen/MachineFrameInfo.h"
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/Passes.h"
#include "llvm/IR/IRBuilder.h"
#include "llvm/IR/MDBuilder.h"
````
- **L1 EN**: Comment documents: `===- SanitizerBinaryMetadata.cpp`.
  **L1 CN**: 注释说明：`===- SanitizerBinaryMetadata.cpp`。
- **L2 EN**: Comment documents: `----------------------------------------------===`.
  **L2 CN**: 注释说明：`----------------------------------------------===`。
- **L3 EN**: Continues the surrounding comment block.
  **L3 CN**: 延续周围的注释块。
- **L4 EN**: Comment documents: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Except…`.
  **L4 CN**: 注释说明：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Except…`。
- **L5 EN**: Comment documents: `See https://llvm.org/LICENSE.txt for license information.`.
  **L5 CN**: 注释说明：`See https://llvm.org/LICENSE.txt for license information.`。
- **L6 EN**: Comment documents: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`.
  **L6 CN**: 注释说明：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L7 EN**: Continues the surrounding comment block.
  **L7 CN**: 延续周围的注释块。
- **L8 EN**: Comment documents: `===---------------------------------------------------------------------…`.
  **L8 CN**: 注释说明：`===---------------------------------------------------------------------…`。
- **L9 EN**: Continues the surrounding comment block.
  **L9 CN**: 延续周围的注释块。
- **L10 EN**: Comment documents: `This file is a part of SanitizerBinaryMetadata.`.
  **L10 CN**: 注释说明：`This file is a part of SanitizerBinaryMetadata.`。
- **L11 EN**: Continues the surrounding comment block.
  **L11 CN**: 延续周围的注释块。
- **L12 EN**: Comment documents: `===---------------------------------------------------------------------…`.
  **L12 CN**: 注释说明：`===---------------------------------------------------------------------…`。
- **L13 EN**: Separates nearby statements for readability.
  **L13 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L14 EN**: Includes LLVM header `llvm/CodeGen/SanitizerBinaryMetadata.h` for SanitizerBinaryMetadata support.
  **L14 CN**: 引入 LLVM 头文件 `llvm/CodeGen/SanitizerBinaryMetadata.h`，用于 SanitizerBinaryMetadata 相关支持。
- **L15 EN**: Includes LLVM header `llvm/CodeGen/MachineFrameInfo.h` for MachineFrameInfo support.
  **L15 CN**: 引入 LLVM 头文件 `llvm/CodeGen/MachineFrameInfo.h`，用于 MachineFrameInfo 相关支持。
- **L16 EN**: Includes LLVM header `llvm/CodeGen/MachineFunction.h` for MachineFunction support.
  **L16 CN**: 引入 LLVM 头文件 `llvm/CodeGen/MachineFunction.h`，用于 MachineFunction 相关支持。
- **L17 EN**: Includes LLVM header `llvm/CodeGen/MachineFunctionPass.h` for MachineFunctionPass support.
  **L17 CN**: 引入 LLVM 头文件 `llvm/CodeGen/MachineFunctionPass.h`，用于 MachineFunctionPass 相关支持。
- **L18 EN**: Includes LLVM header `llvm/CodeGen/Passes.h` for Passes support.
  **L18 CN**: 引入 LLVM 头文件 `llvm/CodeGen/Passes.h`，用于 Passes 相关支持。
- **L19 EN**: Includes LLVM header `llvm/IR/IRBuilder.h` for IRBuilder support.
  **L19 CN**: 引入 LLVM 头文件 `llvm/IR/IRBuilder.h`，用于 IRBuilder 相关支持。
- **L20 EN**: Includes LLVM header `llvm/IR/MDBuilder.h` for MDBuilder support.
  **L20 CN**: 引入 LLVM 头文件 `llvm/IR/MDBuilder.h`，用于 MDBuilder 相关支持。

### Lines 21-40

````cpp
#include "llvm/InitializePasses.h"
#include "llvm/Pass.h"
#include "llvm/Support/VirtualFileSystem.h"
#include "llvm/Transforms/Instrumentation/SanitizerBinaryMetadata.h"
#include <algorithm>

using namespace llvm;

namespace {
// FIXME: This pass modifies Function metadata, which is not to be done in
// MachineFunctionPass. It should probably be moved to a FunctionPass.
class MachineSanitizerBinaryMetadataLegacy : public MachineFunctionPass {
public:
  static char ID;

  MachineSanitizerBinaryMetadataLegacy();
  bool runOnMachineFunction(MachineFunction &F) override;
};

struct MachineSanitizerBinaryMetadata {
````
- **L21 EN**: Includes LLVM header `llvm/InitializePasses.h` for InitializePasses support.
  **L21 CN**: 引入 LLVM 头文件 `llvm/InitializePasses.h`，用于 InitializePasses 相关支持。
- **L22 EN**: Includes LLVM header `llvm/Pass.h` for Pass support.
  **L22 CN**: 引入 LLVM 头文件 `llvm/Pass.h`，用于 Pass 相关支持。
- **L23 EN**: Includes LLVM header `llvm/Support/VirtualFileSystem.h` for VirtualFileSystem support.
  **L23 CN**: 引入 LLVM 头文件 `llvm/Support/VirtualFileSystem.h`，用于 VirtualFileSystem 相关支持。
- **L24 EN**: Includes LLVM header `llvm/Transforms/Instrumentation/SanitizerBinaryMetadata.h` for SanitizerBinaryMetadata support.
  **L24 CN**: 引入 LLVM 头文件 `llvm/Transforms/Instrumentation/SanitizerBinaryMetadata.h`，用于 SanitizerBinaryMetadata 相关支持。
- **L25 EN**: Includes system header `algorithm`.
  **L25 CN**: 引入系统头文件 `algorithm`。
- **L26 EN**: Separates nearby statements for readability.
  **L26 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L27 EN**: Imports namespace `llvm` into this translation unit.
  **L27 CN**: 将命名空间 `llvm` 引入当前编译单元。
- **L28 EN**: Separates nearby statements for readability.
  **L28 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L29 EN**: Opens namespace ``.
  **L29 CN**: 打开命名空间 ``。
- **L30 EN**: Comment documents: `FIXME: This pass modifies Function metadata, which is not to be done in`.
  **L30 CN**: 注释说明：`FIXME: This pass modifies Function metadata, which is not to be done in`。
- **L31 EN**: Comment documents: `MachineFunctionPass. It should probably be moved to a FunctionPass.`.
  **L31 CN**: 注释说明：`MachineFunctionPass. It should probably be moved to a FunctionPass.`。
- **L32 EN**: Starts the declaration of class `MachineSanitizerBinaryMetadataLegacy`.
  **L32 CN**: 开始声明 class `MachineSanitizerBinaryMetadataLegacy`。
- **L33 EN**: Continues logic with `public:`.
  **L33 CN**: 继续处理逻辑：`public:`。
- **L34 EN**: Executes statement `static char ID;`.
  **L34 CN**: 执行语句 `static char ID;`。
- **L35 EN**: Separates nearby statements for readability.
  **L35 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L36 EN**: Executes statement `MachineSanitizerBinaryMetadataLegacy();`.
  **L36 CN**: 执行语句 `MachineSanitizerBinaryMetadataLegacy();`。
- **L37 EN**: Declares function or method `runOnMachineFunction`.
  **L37 CN**: 声明函数或方法 `runOnMachineFunction`。
- **L38 EN**: Closes the current scope.
  **L38 CN**: 关闭当前作用域。
- **L39 EN**: Separates nearby statements for readability.
  **L39 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L40 EN**: Starts the declaration of struct `MachineSanitizerBinaryMetadata`.
  **L40 CN**: 开始声明 struct `MachineSanitizerBinaryMetadata`。

### Lines 41-60

````cpp
  bool run(MachineFunction &MF);
};

} // namespace

INITIALIZE_PASS(MachineSanitizerBinaryMetadataLegacy, "machine-sanmd",
                "Machine Sanitizer Binary Metadata", false, false)

char MachineSanitizerBinaryMetadataLegacy::ID = 0;
char &llvm::MachineSanitizerBinaryMetadataID =
    MachineSanitizerBinaryMetadataLegacy::ID;

MachineSanitizerBinaryMetadataLegacy::MachineSanitizerBinaryMetadataLegacy()
    : MachineFunctionPass(ID) {}

bool MachineSanitizerBinaryMetadataLegacy::runOnMachineFunction(
    MachineFunction &MF) {
  return MachineSanitizerBinaryMetadata().run(MF);
}

````
- **L41 EN**: Declares function or method `run`.
  **L41 CN**: 声明函数或方法 `run`。
- **L42 EN**: Closes the current scope.
  **L42 CN**: 关闭当前作用域。
- **L43 EN**: Separates nearby statements for readability.
  **L43 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L44 EN**: Continues logic with `} // namespace`.
  **L44 CN**: 继续处理逻辑：`} // namespace`。
- **L45 EN**: Separates nearby statements for readability.
  **L45 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L46 EN**: Continues logic with `INITIALIZE_PASS(MachineSanitizerBinaryMetadataLegacy, "machine-sanmd",`.
  **L46 CN**: 继续处理逻辑：`INITIALIZE_PASS(MachineSanitizerBinaryMetadataLegacy, "machine-sanmd",`。
- **L47 EN**: Continues logic with `"Machine Sanitizer Binary Metadata", false, false)`.
  **L47 CN**: 继续处理逻辑：`"Machine Sanitizer Binary Metadata", false, false)`。
- **L48 EN**: Separates nearby statements for readability.
  **L48 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L49 EN**: Assigns or initializes `char MachineSanitizerBinaryMetadataLegacy::ID`.
  **L49 CN**: 对 `char MachineSanitizerBinaryMetadataLegacy::ID` 进行赋值或初始化。
- **L50 EN**: Continues logic with `char &llvm::MachineSanitizerBinaryMetadataID =`.
  **L50 CN**: 继续处理逻辑：`char &llvm::MachineSanitizerBinaryMetadataID =`。
- **L51 EN**: Executes statement `MachineSanitizerBinaryMetadataLegacy::ID;`.
  **L51 CN**: 执行语句 `MachineSanitizerBinaryMetadataLegacy::ID;`。
- **L52 EN**: Separates nearby statements for readability.
  **L52 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L53 EN**: Provides part of the signature for `MachineSanitizerBinaryMetadataLegacy`.
  **L53 CN**: 给出 `MachineSanitizerBinaryMetadataLegacy` 的一部分签名。
- **L54 EN**: Provides part of the signature for `MachineFunctionPass`.
  **L54 CN**: 给出 `MachineFunctionPass` 的一部分签名。
- **L55 EN**: Separates nearby statements for readability.
  **L55 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L56 EN**: Provides part of the signature for `runOnMachineFunction`.
  **L56 CN**: 给出 `runOnMachineFunction` 的一部分签名。
- **L57 EN**: Starts block `MachineFunction &MF)`.
  **L57 CN**: 开始代码块 `MachineFunction &MF)`。
- **L58 EN**: Returns `MachineSanitizerBinaryMetadata().run(MF)` to the caller.
  **L58 CN**: 向调用者返回 `MachineSanitizerBinaryMetadata().run(MF)`。
- **L59 EN**: Closes the current scope.
  **L59 CN**: 关闭当前作用域。
- **L60 EN**: Separates nearby statements for readability.
  **L60 CN**: 空行，用于分隔相邻语句并提升可读性。

### Lines 61-80

````cpp
PreservedAnalyses
MachineSanitizerBinaryMetadataPass::run(MachineFunction &MF,
                                        MachineFunctionAnalysisManager &MFAM) {
  if (!MachineSanitizerBinaryMetadata().run(MF))
    return PreservedAnalyses::all();

  return getMachineFunctionPassPreservedAnalyses();
}

bool MachineSanitizerBinaryMetadata::run(MachineFunction &MF) {
  MDNode *MD = MF.getFunction().getMetadata(LLVMContext::MD_pcsections);
  if (!MD)
    return false;
  const auto &Section = *cast<MDString>(MD->getOperand(0));
  if (!Section.getString().starts_with(kSanitizerBinaryMetadataCoveredSection))
    return false;
  auto &AuxMDs = *cast<MDTuple>(MD->getOperand(1));
  // Assume it currently only has features.
  assert(AuxMDs.getNumOperands() == 1);
  Constant *Features =
````
- **L61 EN**: Continues logic with `PreservedAnalyses`.
  **L61 CN**: 继续处理逻辑：`PreservedAnalyses`。
- **L62 EN**: Provides part of the signature for `run`.
  **L62 CN**: 给出 `run` 的一部分签名。
- **L63 EN**: Starts block `MachineFunctionAnalysisManager &MFAM)`.
  **L63 CN**: 开始代码块 `MachineFunctionAnalysisManager &MFAM)`。
- **L64 EN**: Begins a conditional branch.
  **L64 CN**: 开始一个条件分支。
- **L65 EN**: Returns `PreservedAnalyses::all()` to the caller.
  **L65 CN**: 向调用者返回 `PreservedAnalyses::all()`。
- **L66 EN**: Separates nearby statements for readability.
  **L66 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L67 EN**: Returns `getMachineFunctionPassPreservedAnalyses()` to the caller.
  **L67 CN**: 向调用者返回 `getMachineFunctionPassPreservedAnalyses()`。
- **L68 EN**: Closes the current scope.
  **L68 CN**: 关闭当前作用域。
- **L69 EN**: Separates nearby statements for readability.
  **L69 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L70 EN**: Begins the definition of `run`.
  **L70 CN**: 开始定义 `run`。
- **L71 EN**: Assigns or initializes `MDNode *MD`.
  **L71 CN**: 对 `MDNode *MD` 进行赋值或初始化。
- **L72 EN**: Begins a conditional branch.
  **L72 CN**: 开始一个条件分支。
- **L73 EN**: Returns `false` to the caller.
  **L73 CN**: 向调用者返回 `false`。
- **L74 EN**: Assigns or initializes `const auto &Section`.
  **L74 CN**: 对 `const auto &Section` 进行赋值或初始化。
- **L75 EN**: Begins a conditional branch.
  **L75 CN**: 开始一个条件分支。
- **L76 EN**: Returns `false` to the caller.
  **L76 CN**: 向调用者返回 `false`。
- **L77 EN**: Assigns or initializes `auto &AuxMDs`.
  **L77 CN**: 对 `auto &AuxMDs` 进行赋值或初始化。
- **L78 EN**: Comment documents: `Assume it currently only has features.`.
  **L78 CN**: 注释说明：`Assume it currently only has features.`。
- **L79 EN**: Checks an invariant in debug builds.
  **L79 CN**: 在调试构建中检查一个不变量。
- **L80 EN**: Continues logic with `Constant *Features =`.
  **L80 CN**: 继续处理逻辑：`Constant *Features =`。

### Lines 81-100

````cpp
      cast<ConstantAsMetadata>(AuxMDs.getOperand(0))->getValue();
  if (!Features->getUniqueInteger()[kSanitizerBinaryMetadataUARBit])
    return false;
  // Calculate size of stack args for the function.
  int64_t Size = 0;
  uint64_t Align = 0;
  const MachineFrameInfo &MFI = MF.getFrameInfo();
  for (int i = -1; i >= (int)-MFI.getNumFixedObjects(); --i) {
    Size = std::max(Size, MFI.getObjectOffset(i) + MFI.getObjectSize(i));
    Align = std::max(Align, MFI.getObjectAlign(i).value());
  }
  Size = (Size + Align - 1) & ~(Align - 1);
  if (!Size)
    return false;
  // Non-zero size, update metadata.
  auto &F = MF.getFunction();
  IRBuilder<> IRB(F.getContext());
  MDBuilder MDB(F.getContext());
  // Keep the features and append size of stack args to the metadata.
  APInt NewFeatures = Features->getUniqueInteger();
````
- **L81 EN**: Executes statement `cast<ConstantAsMetadata>(AuxMDs.getOperand(0))->getValue();`.
  **L81 CN**: 执行语句 `cast<ConstantAsMetadata>(AuxMDs.getOperand(0))->getValue();`。
- **L82 EN**: Begins a conditional branch.
  **L82 CN**: 开始一个条件分支。
- **L83 EN**: Returns `false` to the caller.
  **L83 CN**: 向调用者返回 `false`。
- **L84 EN**: Comment documents: `Calculate size of stack args for the function.`.
  **L84 CN**: 注释说明：`Calculate size of stack args for the function.`。
- **L85 EN**: Assigns or initializes `int64_t Size`.
  **L85 CN**: 对 `int64_t Size` 进行赋值或初始化。
- **L86 EN**: Assigns or initializes `uint64_t Align`.
  **L86 CN**: 对 `uint64_t Align` 进行赋值或初始化。
- **L87 EN**: Assigns or initializes `const MachineFrameInfo &MFI`.
  **L87 CN**: 对 `const MachineFrameInfo &MFI` 进行赋值或初始化。
- **L88 EN**: Starts a loop over a sequence or range.
  **L88 CN**: 开始遍历序列或范围的循环。
- **L89 EN**: Declares function or method `max`.
  **L89 CN**: 声明函数或方法 `max`。
- **L90 EN**: Declares function or method `max`.
  **L90 CN**: 声明函数或方法 `max`。
- **L91 EN**: Closes the current scope.
  **L91 CN**: 关闭当前作用域。
- **L92 EN**: Assigns or initializes `Size`.
  **L92 CN**: 对 `Size` 进行赋值或初始化。
- **L93 EN**: Begins a conditional branch.
  **L93 CN**: 开始一个条件分支。
- **L94 EN**: Returns `false` to the caller.
  **L94 CN**: 向调用者返回 `false`。
- **L95 EN**: Comment documents: `Non-zero size, update metadata.`.
  **L95 CN**: 注释说明：`Non-zero size, update metadata.`。
- **L96 EN**: Assigns or initializes `auto &F`.
  **L96 CN**: 对 `auto &F` 进行赋值或初始化。
- **L97 EN**: Declares function or method `IRB`.
  **L97 CN**: 声明函数或方法 `IRB`。
- **L98 EN**: Declares function or method `MDB`.
  **L98 CN**: 声明函数或方法 `MDB`。
- **L99 EN**: Comment documents: `Keep the features and append size of stack args to the metadata.`.
  **L99 CN**: 注释说明：`Keep the features and append size of stack args to the metadata.`。
- **L100 EN**: Assigns or initializes `APInt NewFeatures`.
  **L100 CN**: 对 `APInt NewFeatures` 进行赋值或初始化。

### Lines 101-107

````cpp
  NewFeatures.setBit(kSanitizerBinaryMetadataUARHasSizeBit);
  F.setMetadata(
      LLVMContext::MD_pcsections,
      MDB.createPCSections({{Section.getString(),
                             {IRB.getInt(NewFeatures), IRB.getInt32(Size)}}}));
  return false;
}
````
- **L101 EN**: Executes statement `NewFeatures.setBit(kSanitizerBinaryMetadataUARHasSizeBit);`.
  **L101 CN**: 执行语句 `NewFeatures.setBit(kSanitizerBinaryMetadataUARHasSizeBit);`。
- **L102 EN**: Continues logic with `F.setMetadata(`.
  **L102 CN**: 继续处理逻辑：`F.setMetadata(`。
- **L103 EN**: Continues logic with `LLVMContext::MD_pcsections,`.
  **L103 CN**: 继续处理逻辑：`LLVMContext::MD_pcsections,`。
- **L104 EN**: Continues logic with `MDB.createPCSections({{Section.getString(),`.
  **L104 CN**: 继续处理逻辑：`MDB.createPCSections({{Section.getString(),`。
- **L105 EN**: Executes statement `{IRB.getInt(NewFeatures), IRB.getInt32(Size)}}}));`.
  **L105 CN**: 执行语句 `{IRB.getInt(NewFeatures), IRB.getInt32(Size)}}}));`。
- **L106 EN**: Returns `false` to the caller.
  **L106 CN**: 向调用者返回 `false`。
- **L107 EN**: Closes the current scope.
  **L107 CN**: 关闭当前作用域。

## Key Concepts / 关键概念
- **Machine function processing** / **MachineFunction 处理**
- **Stack frame management** / **栈帧管理**

## Dependencies / 依赖关系
- **LLVM headers / LLVM 头文件**: `llvm/CodeGen/SanitizerBinaryMetadata.h`, `llvm/CodeGen/MachineFrameInfo.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/CodeGen/MachineFunctionPass.h`, `llvm/CodeGen/Passes.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/MDBuilder.h`, `llvm/InitializePasses.h`, `llvm/Pass.h`, `llvm/Support/VirtualFileSystem.h`, `llvm/Transforms/Instrumentation/SanitizerBinaryMetadata.h`
- **System headers / 系统头文件**: `algorithm`
- **Primary dependency domains / 主要依赖域**: CodeGen internals, LLVM support utilities, and C++ runtime helpers. / CodeGen 内部组件、LLVM 支持工具以及 C++ 运行时辅助设施。
