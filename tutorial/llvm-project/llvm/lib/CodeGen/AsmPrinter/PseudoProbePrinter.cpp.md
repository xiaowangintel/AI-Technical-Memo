# PseudoProbePrinter.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/CodeGen/AsmPrinter/PseudoProbePrinter.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: This file contains `Pseudo Probe Emission` logic inside LLVM CodeGen.
- **Purpose (CN)**: 该文件在 LLVM CodeGen 中实现与“Pseudo Probe Emission”相关的逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

````cpp
//===- llvm/CodeGen/PseudoProbePrinter.cpp - Pseudo Probe Emission -------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains support for writing pseudo probe info into asm files.
//
//===----------------------------------------------------------------------===//

#include "PseudoProbePrinter.h"
#include "llvm/CodeGen/AsmPrinter.h"
#include "llvm/IR/DebugInfoMetadata.h"
#include "llvm/IR/Function.h"
#include "llvm/IR/PseudoProbe.h"
#include "llvm/MC/MCPseudoProbe.h"
#include "llvm/MC/MCStreamer.h"
#include "llvm/ProfileData/SampleProf.h"
````
- **L1 EN**: Comment documents: `===- llvm/CodeGen/PseudoProbePrinter.cpp - Pseudo Probe Emission -------…`.
  **L1 CN**: 注释说明：`===- llvm/CodeGen/PseudoProbePrinter.cpp - Pseudo Probe Emission -------…`。
- **L2 EN**: Continues the surrounding comment block.
  **L2 CN**: 延续周围的注释块。
- **L3 EN**: Comment documents: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Except…`.
  **L3 CN**: 注释说明：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Except…`。
- **L4 EN**: Comment documents: `See https://llvm.org/LICENSE.txt for license information.`.
  **L4 CN**: 注释说明：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5 EN**: Comment documents: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`.
  **L5 CN**: 注释说明：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6 EN**: Continues the surrounding comment block.
  **L6 CN**: 延续周围的注释块。
- **L7 EN**: Comment documents: `===---------------------------------------------------------------------…`.
  **L7 CN**: 注释说明：`===---------------------------------------------------------------------…`。
- **L8 EN**: Continues the surrounding comment block.
  **L8 CN**: 延续周围的注释块。
- **L9 EN**: Comment documents: `This file contains support for writing pseudo probe info into asm files.`.
  **L9 CN**: 注释说明：`This file contains support for writing pseudo probe info into asm files.`。
- **L10 EN**: Continues the surrounding comment block.
  **L10 CN**: 延续周围的注释块。
- **L11 EN**: Comment documents: `===---------------------------------------------------------------------…`.
  **L11 CN**: 注释说明：`===---------------------------------------------------------------------…`。
- **L12 EN**: Separates nearby statements for readability.
  **L12 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L13 EN**: Includes system header `PseudoProbePrinter.h`.
  **L13 CN**: 引入系统头文件 `PseudoProbePrinter.h`。
- **L14 EN**: Includes LLVM header `llvm/CodeGen/AsmPrinter.h` for AsmPrinter support.
  **L14 CN**: 引入 LLVM 头文件 `llvm/CodeGen/AsmPrinter.h`，用于 AsmPrinter 相关支持。
- **L15 EN**: Includes LLVM header `llvm/IR/DebugInfoMetadata.h` for DebugInfoMetadata support.
  **L15 CN**: 引入 LLVM 头文件 `llvm/IR/DebugInfoMetadata.h`，用于 DebugInfoMetadata 相关支持。
- **L16 EN**: Includes LLVM header `llvm/IR/Function.h` for Function support.
  **L16 CN**: 引入 LLVM 头文件 `llvm/IR/Function.h`，用于 Function 相关支持。
- **L17 EN**: Includes LLVM header `llvm/IR/PseudoProbe.h` for PseudoProbe support.
  **L17 CN**: 引入 LLVM 头文件 `llvm/IR/PseudoProbe.h`，用于 PseudoProbe 相关支持。
- **L18 EN**: Includes LLVM header `llvm/MC/MCPseudoProbe.h` for MCPseudoProbe support.
  **L18 CN**: 引入 LLVM 头文件 `llvm/MC/MCPseudoProbe.h`，用于 MCPseudoProbe 相关支持。
- **L19 EN**: Includes LLVM header `llvm/MC/MCStreamer.h` for MCStreamer support.
  **L19 CN**: 引入 LLVM 头文件 `llvm/MC/MCStreamer.h`，用于 MCStreamer 相关支持。
- **L20 EN**: Includes LLVM header `llvm/ProfileData/SampleProf.h` for SampleProf support.
  **L20 CN**: 引入 LLVM 头文件 `llvm/ProfileData/SampleProf.h`，用于 SampleProf 相关支持。

### Lines 21-40

````cpp

#ifndef NDEBUG
#include "llvm/IR/Module.h"
#include "llvm/Support/WithColor.h"
#endif

using namespace llvm;

#ifndef NDEBUG
// Deprecated with ThinLTO. For some modules compiled with ThinLTO, certain
// pseudo probe descriptors may not be imported, resulting in false positive
// warning.
static cl::opt<bool> VerifyGuidExistence(
    "pseudo-probe-verify-guid-existence-in-desc",
    cl::desc("Verify whether GUID exists in the .pseudo_probe_desc."),
    cl::Hidden, cl::init(false));
#endif

void PseudoProbeHandler::emitPseudoProbe(uint64_t Guid, uint64_t Index,
                                         uint64_t Type, uint64_t Attr,
````
- **L21 EN**: Separates nearby statements for readability.
  **L21 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L22 EN**: Starts a preprocessor conditional block.
  **L22 CN**: 开始一个预处理条件块。
- **L23 EN**: Includes LLVM header `llvm/IR/Module.h` for Module support.
  **L23 CN**: 引入 LLVM 头文件 `llvm/IR/Module.h`，用于 Module 相关支持。
- **L24 EN**: Includes LLVM header `llvm/Support/WithColor.h` for WithColor support.
  **L24 CN**: 引入 LLVM 头文件 `llvm/Support/WithColor.h`，用于 WithColor 相关支持。
- **L25 EN**: Ends the current preprocessor conditional block.
  **L25 CN**: 结束当前的预处理条件块。
- **L26 EN**: Separates nearby statements for readability.
  **L26 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L27 EN**: Imports namespace `llvm` into this translation unit.
  **L27 CN**: 将命名空间 `llvm` 引入当前编译单元。
- **L28 EN**: Separates nearby statements for readability.
  **L28 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L29 EN**: Starts a preprocessor conditional block.
  **L29 CN**: 开始一个预处理条件块。
- **L30 EN**: Comment documents: `Deprecated with ThinLTO. For some modules compiled with ThinLTO, certain`.
  **L30 CN**: 注释说明：`Deprecated with ThinLTO. For some modules compiled with ThinLTO, certain`。
- **L31 EN**: Comment documents: `pseudo probe descriptors may not be imported, resulting in false positiv…`.
  **L31 CN**: 注释说明：`pseudo probe descriptors may not be imported, resulting in false positiv…`。
- **L32 EN**: Comment documents: `warning.`.
  **L32 CN**: 注释说明：`warning.`。
- **L33 EN**: Declares LLVM command-line option `command-line option`.
  **L33 CN**: 声明 LLVM 命令行选项 `command-line option`。
- **L34 EN**: Continues logic with `"pseudo-probe-verify-guid-existence-in-desc",`.
  **L34 CN**: 继续处理逻辑：`"pseudo-probe-verify-guid-existence-in-desc",`。
- **L35 EN**: Provides part of the signature for `desc`.
  **L35 CN**: 给出 `desc` 的一部分签名。
- **L36 EN**: Declares function or method `init`.
  **L36 CN**: 声明函数或方法 `init`。
- **L37 EN**: Ends the current preprocessor conditional block.
  **L37 CN**: 结束当前的预处理条件块。
- **L38 EN**: Separates nearby statements for readability.
  **L38 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L39 EN**: Provides part of the signature for `emitPseudoProbe`.
  **L39 CN**: 给出 `emitPseudoProbe` 的一部分签名。
- **L40 EN**: Continues logic with `uint64_t Type, uint64_t Attr,`.
  **L40 CN**: 继续处理逻辑：`uint64_t Type, uint64_t Attr,`。

### Lines 41-60

````cpp
                                         const DILocation *DebugLoc) {
  // Gather all the inlined-at nodes.
  // When it's done ReversedInlineStack looks like ([66, B], [88, A])
  // which means, Function A inlines function B at calliste with a probe id 88,
  // and B inlines C at probe 66 where C is represented by Guid.
  SmallVector<InlineSite, 8> ReversedInlineStack;
  auto *InlinedAt = DebugLoc ? DebugLoc->getInlinedAt() : nullptr;
  while (InlinedAt) {
    auto Name = InlinedAt->getSubprogramLinkageName();
    // Strip Coroutine suffixes from CoroSplit Pass, since pseudo probes are
    // generated in an earlier stage.
    Name = FunctionSamples::getCanonicalCoroFnName(Name);
    // Use caching to avoid redundant md5 computation for build speed.
    uint64_t &CallerGuid = NameGuidMap[Name];
    if (!CallerGuid)
      CallerGuid = Function::getGUIDAssumingExternalLinkage(Name);
#ifndef NDEBUG
    if (VerifyGuidExistence)
      verifyGuidExistenceInDesc(CallerGuid, Name);
#endif
````
- **L41 EN**: Starts block `const DILocation *DebugLoc)`.
  **L41 CN**: 开始代码块 `const DILocation *DebugLoc)`。
- **L42 EN**: Comment documents: `Gather all the inlined-at nodes.`.
  **L42 CN**: 注释说明：`Gather all the inlined-at nodes.`。
- **L43 EN**: Comment documents: `When it's done ReversedInlineStack looks like ([66, B], [88, A])`.
  **L43 CN**: 注释说明：`When it's done ReversedInlineStack looks like ([66, B], [88, A])`。
- **L44 EN**: Comment documents: `which means, Function A inlines function B at calliste with a probe id 8…`.
  **L44 CN**: 注释说明：`which means, Function A inlines function B at calliste with a probe id 8…`。
- **L45 EN**: Comment documents: `and B inlines C at probe 66 where C is represented by Guid.`.
  **L45 CN**: 注释说明：`and B inlines C at probe 66 where C is represented by Guid.`。
- **L46 EN**: Executes statement `SmallVector<InlineSite, 8> ReversedInlineStack;`.
  **L46 CN**: 执行语句 `SmallVector<InlineSite, 8> ReversedInlineStack;`。
- **L47 EN**: Assigns or initializes `auto *InlinedAt`.
  **L47 CN**: 对 `auto *InlinedAt` 进行赋值或初始化。
- **L48 EN**: Starts a while loop controlled by a condition.
  **L48 CN**: 开始一个由条件控制的 while 循环。
- **L49 EN**: Assigns or initializes `auto Name`.
  **L49 CN**: 对 `auto Name` 进行赋值或初始化。
- **L50 EN**: Comment documents: `Strip Coroutine suffixes from CoroSplit Pass, since pseudo probes are`.
  **L50 CN**: 注释说明：`Strip Coroutine suffixes from CoroSplit Pass, since pseudo probes are`。
- **L51 EN**: Comment documents: `generated in an earlier stage.`.
  **L51 CN**: 注释说明：`generated in an earlier stage.`。
- **L52 EN**: Declares function or method `getCanonicalCoroFnName`.
  **L52 CN**: 声明函数或方法 `getCanonicalCoroFnName`。
- **L53 EN**: Comment documents: `Use caching to avoid redundant md5 computation for build speed.`.
  **L53 CN**: 注释说明：`Use caching to avoid redundant md5 computation for build speed.`。
- **L54 EN**: Assigns or initializes `uint64_t &CallerGuid`.
  **L54 CN**: 对 `uint64_t &CallerGuid` 进行赋值或初始化。
- **L55 EN**: Begins a conditional branch.
  **L55 CN**: 开始一个条件分支。
- **L56 EN**: Declares function or method `getGUIDAssumingExternalLinkage`.
  **L56 CN**: 声明函数或方法 `getGUIDAssumingExternalLinkage`。
- **L57 EN**: Starts a preprocessor conditional block.
  **L57 CN**: 开始一个预处理条件块。
- **L58 EN**: Begins a conditional branch.
  **L58 CN**: 开始一个条件分支。
- **L59 EN**: Executes statement `verifyGuidExistenceInDesc(CallerGuid, Name);`.
  **L59 CN**: 执行语句 `verifyGuidExistenceInDesc(CallerGuid, Name);`。
- **L60 EN**: Ends the current preprocessor conditional block.
  **L60 CN**: 结束当前的预处理条件块。

### Lines 61-80

````cpp
    uint64_t CallerProbeId = PseudoProbeDwarfDiscriminator::extractProbeIndex(
        InlinedAt->getDiscriminator());
    ReversedInlineStack.emplace_back(CallerGuid, CallerProbeId);
    InlinedAt = InlinedAt->getInlinedAt();
  }
  uint64_t Discriminator = 0;
  // For now only block probes have FS discriminators. See
  // MIRFSDiscriminator.cpp for more details.
  if (EnableFSDiscriminator && DebugLoc &&
      (Type == (uint64_t)PseudoProbeType::Block))
    Discriminator = DebugLoc->getDiscriminator();
  assert((EnableFSDiscriminator || Discriminator == 0) &&
         "Discriminator should not be set in non-FSAFDO mode");
  SmallVector<InlineSite, 8> InlineStack(llvm::reverse(ReversedInlineStack));
  Asm->OutStreamer->emitPseudoProbe(Guid, Index, Type, Attr, Discriminator,
                                    InlineStack, Asm->CurrentFnSym);
#ifndef NDEBUG
  if (VerifyGuidExistence)
    verifyGuidExistenceInDesc(
        Guid, DebugLoc ? DebugLoc->getSubprogramLinkageName() : "");
````
- **L61 EN**: Provides part of the signature for `extractProbeIndex`.
  **L61 CN**: 给出 `extractProbeIndex` 的一部分签名。
- **L62 EN**: Executes statement `InlinedAt->getDiscriminator());`.
  **L62 CN**: 执行语句 `InlinedAt->getDiscriminator());`。
- **L63 EN**: Executes statement `ReversedInlineStack.emplace_back(CallerGuid, CallerProbeId);`.
  **L63 CN**: 执行语句 `ReversedInlineStack.emplace_back(CallerGuid, CallerProbeId);`。
- **L64 EN**: Assigns or initializes `InlinedAt`.
  **L64 CN**: 对 `InlinedAt` 进行赋值或初始化。
- **L65 EN**: Closes the current scope.
  **L65 CN**: 关闭当前作用域。
- **L66 EN**: Assigns or initializes `uint64_t Discriminator`.
  **L66 CN**: 对 `uint64_t Discriminator` 进行赋值或初始化。
- **L67 EN**: Comment documents: `For now only block probes have FS discriminators. See`.
  **L67 CN**: 注释说明：`For now only block probes have FS discriminators. See`。
- **L68 EN**: Comment documents: `MIRFSDiscriminator.cpp for more details.`.
  **L68 CN**: 注释说明：`MIRFSDiscriminator.cpp for more details.`。
- **L69 EN**: Begins a conditional branch.
  **L69 CN**: 开始一个条件分支。
- **L70 EN**: Continues logic with `(Type == (uint64_t)PseudoProbeType::Block))`.
  **L70 CN**: 继续处理逻辑：`(Type == (uint64_t)PseudoProbeType::Block))`。
- **L71 EN**: Assigns or initializes `Discriminator`.
  **L71 CN**: 对 `Discriminator` 进行赋值或初始化。
- **L72 EN**: Checks an invariant in debug builds.
  **L72 CN**: 在调试构建中检查一个不变量。
- **L73 EN**: Executes statement `"Discriminator should not be set in non-FSAFDO mode");`.
  **L73 CN**: 执行语句 `"Discriminator should not be set in non-FSAFDO mode");`。
- **L74 EN**: Declares function or method `InlineStack`.
  **L74 CN**: 声明函数或方法 `InlineStack`。
- **L75 EN**: Continues logic with `Asm->OutStreamer->emitPseudoProbe(Guid, Index, Type, Attr, Discriminator…`.
  **L75 CN**: 继续处理逻辑：`Asm->OutStreamer->emitPseudoProbe(Guid, Index, Type, Attr, Discriminator…`。
- **L76 EN**: Executes statement `InlineStack, Asm->CurrentFnSym);`.
  **L76 CN**: 执行语句 `InlineStack, Asm->CurrentFnSym);`。
- **L77 EN**: Starts a preprocessor conditional block.
  **L77 CN**: 开始一个预处理条件块。
- **L78 EN**: Begins a conditional branch.
  **L78 CN**: 开始一个条件分支。
- **L79 EN**: Continues logic with `verifyGuidExistenceInDesc(`.
  **L79 CN**: 继续处理逻辑：`verifyGuidExistenceInDesc(`。
- **L80 EN**: Executes statement `Guid, DebugLoc ? DebugLoc->getSubprogramLinkageName() : "");`.
  **L80 CN**: 执行语句 `Guid, DebugLoc ? DebugLoc->getSubprogramLinkageName() : "");`。

### Lines 81-100

````cpp
#endif
}

#ifndef NDEBUG
void PseudoProbeHandler::verifyGuidExistenceInDesc(uint64_t Guid,
                                                   StringRef FuncName) {
  NamedMDNode *Desc = Asm->MF->getFunction().getParent()->getNamedMetadata(
      PseudoProbeDescMetadataName);
  assert(Desc && "pseudo probe does not exist");

  // Keep DescGuidSet up to date.
  for (size_t I = DescGuidSet.size(), E = Desc->getNumOperands(); I != E; ++I) {
    const auto *MD = cast<MDNode>(Desc->getOperand(I));
    auto *ID = mdconst::extract<ConstantInt>(MD->getOperand(0));
    DescGuidSet.insert(ID->getZExtValue());
  }

  if (!DescGuidSet.contains(Guid))
    WithColor::warning() << "Guid:" << Guid << " Name:" << FuncName
                         << " does not exist in pseudo probe desc\n";
````
- **L81 EN**: Ends the current preprocessor conditional block.
  **L81 CN**: 结束当前的预处理条件块。
- **L82 EN**: Closes the current scope.
  **L82 CN**: 关闭当前作用域。
- **L83 EN**: Separates nearby statements for readability.
  **L83 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L84 EN**: Starts a preprocessor conditional block.
  **L84 CN**: 开始一个预处理条件块。
- **L85 EN**: Provides part of the signature for `verifyGuidExistenceInDesc`.
  **L85 CN**: 给出 `verifyGuidExistenceInDesc` 的一部分签名。
- **L86 EN**: Starts block `StringRef FuncName)`.
  **L86 CN**: 开始代码块 `StringRef FuncName)`。
- **L87 EN**: Continues logic with `NamedMDNode *Desc = Asm->MF->getFunction().getParent()->getNamedMetadata…`.
  **L87 CN**: 继续处理逻辑：`NamedMDNode *Desc = Asm->MF->getFunction().getParent()->getNamedMetadata…`。
- **L88 EN**: Executes statement `PseudoProbeDescMetadataName);`.
  **L88 CN**: 执行语句 `PseudoProbeDescMetadataName);`。
- **L89 EN**: Checks an invariant in debug builds.
  **L89 CN**: 在调试构建中检查一个不变量。
- **L90 EN**: Separates nearby statements for readability.
  **L90 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L91 EN**: Comment documents: `Keep DescGuidSet up to date.`.
  **L91 CN**: 注释说明：`Keep DescGuidSet up to date.`。
- **L92 EN**: Starts a loop over a sequence or range.
  **L92 CN**: 开始遍历序列或范围的循环。
- **L93 EN**: Assigns or initializes `const auto *MD`.
  **L93 CN**: 对 `const auto *MD` 进行赋值或初始化。
- **L94 EN**: Declares function or method `getOperand`.
  **L94 CN**: 声明函数或方法 `getOperand`。
- **L95 EN**: Executes statement `DescGuidSet.insert(ID->getZExtValue());`.
  **L95 CN**: 执行语句 `DescGuidSet.insert(ID->getZExtValue());`。
- **L96 EN**: Closes the current scope.
  **L96 CN**: 关闭当前作用域。
- **L97 EN**: Separates nearby statements for readability.
  **L97 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L98 EN**: Begins a conditional branch.
  **L98 CN**: 开始一个条件分支。
- **L99 EN**: Provides part of the signature for `warning`.
  **L99 CN**: 给出 `warning` 的一部分签名。
- **L100 EN**: Executes statement `<< " does not exist in pseudo probe desc\n";`.
  **L100 CN**: 执行语句 `<< " does not exist in pseudo probe desc\n";`。

### Lines 101-102

````cpp
}
#endif
````
- **L101 EN**: Closes the current scope.
  **L101 CN**: 关闭当前作用域。
- **L102 EN**: Ends the current preprocessor conditional block.
  **L102 CN**: 结束当前的预处理条件块。

## Key Concepts / 关键概念
- **Stack frame management** / **栈帧管理**
- **Debug information handling** / **调试信息处理**
- **Pseudo-instruction expansion** / **伪指令展开**

## Dependencies / 依赖关系
- **LLVM headers / LLVM 头文件**: `llvm/CodeGen/AsmPrinter.h`, `llvm/IR/DebugInfoMetadata.h`, `llvm/IR/Function.h`, `llvm/IR/PseudoProbe.h`, `llvm/MC/MCPseudoProbe.h`, `llvm/MC/MCStreamer.h`, `llvm/ProfileData/SampleProf.h`, `llvm/IR/Module.h`, `llvm/Support/WithColor.h`
- **System headers / 系统头文件**: `PseudoProbePrinter.h`
- **Primary dependency domains / 主要依赖域**: CodeGen internals, LLVM support utilities, and C++ runtime helpers. / CodeGen 内部组件、LLVM 支持工具以及 C++ 运行时辅助设施。
