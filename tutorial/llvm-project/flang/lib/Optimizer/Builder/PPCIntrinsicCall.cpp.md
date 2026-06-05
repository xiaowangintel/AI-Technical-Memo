# PPCIntrinsicCall.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `flang/lib/Optimizer/Builder/PPCIntrinsicCall.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: Helper routines for constructing the FIR dialect of MLIR for PowerPC intrinsics. Extensive use of MLIR interfaces and MLIR's coding style (https://mlir.llvm.org/getting_started/DeveloperGuide/) is used in this module.
- **Purpose (CN)**: 实现 PPC Intrinsic Call 相关的 FIR/MLIR 优化器辅助工具与基础设施。

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

````cpp
//===-- PPCIntrinsicCall.cpp ----------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Helper routines for constructing the FIR dialect of MLIR for PowerPC
// intrinsics. Extensive use of MLIR interfaces and MLIR's coding style
// (https://mlir.llvm.org/getting_started/DeveloperGuide/) is used in this
// module.
//
//===----------------------------------------------------------------------===//

#include "flang/Optimizer/Builder/PPCIntrinsicCall.h"
#include "flang/Evaluate/common.h"
#include "flang/Lower/AbstractConverter.h"
#include "flang/Optimizer/Builder/FIRBuilder.h"
#include "flang/Optimizer/Builder/MutableBox.h"
#include "mlir/Dialect/Index/IR/IndexOps.h"
#include "mlir/Dialect/Vector/IR/VectorOps.h"

namespace fir {
````
- **L1 EN**: Banner comment marking a file or section boundary.
  **L1 CN**: 横幅注释，用于标记文件或章节边界。
- **L2 EN**: Separator comment used for visual grouping.
  **L2 CN**: 用于视觉分组的分隔注释。
- **L3 EN**: Comment explains nearby logic, intent, or metadata: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`.
  **L3 CN**: 注释说明附近代码的逻辑、意图或元数据：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L4 EN**: Comment explains nearby logic, intent, or metadata: `See https://llvm.org/LICENSE.txt for license information.`.
  **L4 CN**: 注释说明附近代码的逻辑、意图或元数据：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5 EN**: Comment explains nearby logic, intent, or metadata: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`.
  **L5 CN**: 注释说明附近代码的逻辑、意图或元数据：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6 EN**: Separator comment used for visual grouping.
  **L6 CN**: 用于视觉分组的分隔注释。
- **L7 EN**: Banner comment marking a file or section boundary.
  **L7 CN**: 横幅注释，用于标记文件或章节边界。
- **L8 EN**: Separator comment used for visual grouping.
  **L8 CN**: 用于视觉分组的分隔注释。
- **L9 EN**: Comment explains nearby logic, intent, or metadata: `Helper routines for constructing the FIR dialect of MLIR for PowerPC`.
  **L9 CN**: 注释说明附近代码的逻辑、意图或元数据：`Helper routines for constructing the FIR dialect of MLIR for PowerPC`。
- **L10 EN**: Comment explains nearby logic, intent, or metadata: `intrinsics. Extensive use of MLIR interfaces and MLIR's coding style`.
  **L10 CN**: 注释说明附近代码的逻辑、意图或元数据：`intrinsics. Extensive use of MLIR interfaces and MLIR's coding style`。
- **L11 EN**: Comment explains nearby logic, intent, or metadata: `(https://mlir.llvm.org/getting_started/DeveloperGuide/) is used in this`.
  **L11 CN**: 注释说明附近代码的逻辑、意图或元数据：`(https://mlir.llvm.org/getting_started/DeveloperGuide/) is used in this`。
- **L12 EN**: Comment explains nearby logic, intent, or metadata: `module.`.
  **L12 CN**: 注释说明附近代码的逻辑、意图或元数据：`module.`。
- **L13 EN**: Separator comment used for visual grouping.
  **L13 CN**: 用于视觉分组的分隔注释。
- **L14 EN**: Banner comment marking a file or section boundary.
  **L14 CN**: 横幅注释，用于标记文件或章节边界。
- **L15 EN**: Blank line separating nearby declarations or logic blocks.
  **L15 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L16 EN**: Includes "flang/Optimizer/Builder/PPCIntrinsicCall.h" to access FIR builder helpers and runtime-construction utilities.
  **L16 CN**: 引入 "flang/Optimizer/Builder/PPCIntrinsicCall.h" 以使用FIR Builder 辅助工具与运行时构造能力。
- **L17 EN**: Includes "flang/Evaluate/common.h" to access Fortran constant-folding and evaluation facilities.
  **L17 CN**: 引入 "flang/Evaluate/common.h" 以使用Fortran 常量折叠与求值能力。
- **L18 EN**: Includes "flang/Lower/AbstractConverter.h" to access lowering utilities that map semantics into FIR/MLIR.
  **L18 CN**: 引入 "flang/Lower/AbstractConverter.h" 以使用将语义映射到 FIR/MLIR 的 lowering 工具。
- **L19 EN**: Includes "flang/Optimizer/Builder/FIRBuilder.h" to access FIR builder helpers and runtime-construction utilities.
  **L19 CN**: 引入 "flang/Optimizer/Builder/FIRBuilder.h" 以使用FIR Builder 辅助工具与运行时构造能力。
- **L20 EN**: Includes "flang/Optimizer/Builder/MutableBox.h" to access FIR builder helpers and runtime-construction utilities.
  **L20 CN**: 引入 "flang/Optimizer/Builder/MutableBox.h" 以使用FIR Builder 辅助工具与运行时构造能力。
- **L21 EN**: Includes "mlir/Dialect/Index/IR/IndexOps.h" to access MLIR core IR, pass, or dialect infrastructure.
  **L21 CN**: 引入 "mlir/Dialect/Index/IR/IndexOps.h" 以使用MLIR 核心 IR、Pass 或方言基础设施。
- **L22 EN**: Includes "mlir/Dialect/Vector/IR/VectorOps.h" to access MLIR core IR, pass, or dialect infrastructure.
  **L22 CN**: 引入 "mlir/Dialect/Vector/IR/VectorOps.h" 以使用MLIR 核心 IR、Pass 或方言基础设施。
- **L23 EN**: Blank line separating nearby declarations or logic blocks.
  **L23 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L24 EN**: Opens namespace scope `fir`.
  **L24 CN**: 打开命名空间作用域 `fir`。

### Lines 25-48

````cpp

using PI = PPCIntrinsicLibrary;

// PPC specific intrinsic handlers.
static constexpr IntrinsicHandler ppcHandlers[]{
    {"__ppc_mma_assemble_acc",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genMmaIntr<MMAOp::AssembleAcc, MMAHandlerOp::SubToFunc>),
     {{{"acc", asAddr},
       {"arg1", asValue},
       {"arg2", asValue},
       {"arg3", asValue},
       {"arg4", asValue}}},
     /*isElemental=*/true},
    {"__ppc_mma_assemble_pair",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genMmaIntr<MMAOp::AssemblePair, MMAHandlerOp::SubToFunc>),
     {{{"pair", asAddr}, {"arg1", asValue}, {"arg2", asValue}}},
     /*isElemental=*/true},
    {"__ppc_mma_build_acc",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genMmaIntr<MMAOp::AssembleAcc,
                         MMAHandlerOp::SubToFuncReverseArgOnLE>),
     {{{"acc", asAddr},
````
- **L25 EN**: Blank line separating nearby declarations or logic blocks.
  **L25 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L26 EN**: Defines alias `PI` to simplify later code.
  **L26 CN**: 定义别名 `PI` 以简化后续代码。
- **L27 EN**: Blank line separating nearby declarations or logic blocks.
  **L27 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L28 EN**: Comment explains nearby logic, intent, or metadata: `PPC specific intrinsic handlers.`.
  **L28 CN**: 注释说明附近代码的逻辑、意图或元数据：`PPC specific intrinsic handlers.`。
- **L29 EN**: Continues the surrounding expression or declaration: `static constexpr IntrinsicHandler ppcHandlers[]{`.
  **L29 CN**: 继续构造周围的表达式或声明：`static constexpr IntrinsicHandler ppcHandlers[]{`。
- **L30 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mma_assemble_acc",`.
  **L30 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mma_assemble_acc",`。
- **L31 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L31 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L32 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genMmaIntr<MMAOp::AssembleAcc, MMAHandlerOp::SubToFunc>),`.
  **L32 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genMmaIntr<MMAOp::AssembleAcc, MMAHandlerOp::SubToFunc>),`。
- **L33 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"acc", asAddr},`.
  **L33 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"acc", asAddr},`。
- **L34 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"arg1", asValue},`.
  **L34 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"arg1", asValue},`。
- **L35 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"arg2", asValue},`.
  **L35 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"arg2", asValue},`。
- **L36 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"arg3", asValue},`.
  **L36 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"arg3", asValue},`。
- **L37 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"arg4", asValue}}},`.
  **L37 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"arg4", asValue}}},`。
- **L38 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L38 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L39 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mma_assemble_pair",`.
  **L39 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mma_assemble_pair",`。
- **L40 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L40 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L41 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genMmaIntr<MMAOp::AssemblePair, MMAHandlerOp::SubToFunc>),`.
  **L41 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genMmaIntr<MMAOp::AssemblePair, MMAHandlerOp::SubToFunc>),`。
- **L42 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"pair", asAddr}, {"arg1", asValue}, {"arg2", asValue}}},`.
  **L42 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"pair", asAddr}, {"arg1", asValue}, {"arg2", asValue}}},`。
- **L43 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L43 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L44 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mma_build_acc",`.
  **L44 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mma_build_acc",`。
- **L45 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L45 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L46 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genMmaIntr<MMAOp::AssembleAcc,`.
  **L46 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genMmaIntr<MMAOp::AssembleAcc,`。
- **L47 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MMAHandlerOp::SubToFuncReverseArgOnLE>),`.
  **L47 CN**: 继续一个多行参数列表、初始化器或聚合项：`MMAHandlerOp::SubToFuncReverseArgOnLE>),`。
- **L48 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"acc", asAddr},`.
  **L48 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"acc", asAddr},`。

### Lines 49-72

````cpp
       {"arg1", asValue},
       {"arg2", asValue},
       {"arg3", asValue},
       {"arg4", asValue}}},
     /*isElemental=*/true},
    {"__ppc_mma_disassemble_acc",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genMmaIntr<MMAOp::DisassembleAcc, MMAHandlerOp::SubToFunc>),
     {{{"data", asAddr}, {"acc", asValue}}},
     /*isElemental=*/true},
    {"__ppc_mma_disassemble_pair",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genMmaIntr<MMAOp::DisassemblePair, MMAHandlerOp::SubToFunc>),
     {{{"data", asAddr}, {"pair", asValue}}},
     /*isElemental=*/true},
    {"__ppc_mma_pmxvbf16ger2_",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genMmaIntr<MMAOp::Pmxvbf16ger2, MMAHandlerOp::SubToFunc>),
     {{{"acc", asAddr},
       {"a", asValue},
       {"b", asValue},
       {"xmask", asValue},
       {"ymask", asValue},
       {"pmask", asValue}}},
````
- **L49 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"arg1", asValue},`.
  **L49 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"arg1", asValue},`。
- **L50 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"arg2", asValue},`.
  **L50 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"arg2", asValue},`。
- **L51 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"arg3", asValue},`.
  **L51 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"arg3", asValue},`。
- **L52 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"arg4", asValue}}},`.
  **L52 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"arg4", asValue}}},`。
- **L53 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L53 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L54 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mma_disassemble_acc",`.
  **L54 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mma_disassemble_acc",`。
- **L55 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L55 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L56 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genMmaIntr<MMAOp::DisassembleAcc, MMAHandlerOp::SubToFunc>),`.
  **L56 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genMmaIntr<MMAOp::DisassembleAcc, MMAHandlerOp::SubToFunc>),`。
- **L57 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"data", asAddr}, {"acc", asValue}}},`.
  **L57 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"data", asAddr}, {"acc", asValue}}},`。
- **L58 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L58 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L59 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mma_disassemble_pair",`.
  **L59 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mma_disassemble_pair",`。
- **L60 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L60 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L61 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genMmaIntr<MMAOp::DisassemblePair, MMAHandlerOp::SubToFunc>),`.
  **L61 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genMmaIntr<MMAOp::DisassemblePair, MMAHandlerOp::SubToFunc>),`。
- **L62 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"data", asAddr}, {"pair", asValue}}},`.
  **L62 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"data", asAddr}, {"pair", asValue}}},`。
- **L63 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L63 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L64 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mma_pmxvbf16ger2_",`.
  **L64 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mma_pmxvbf16ger2_",`。
- **L65 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L65 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L66 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genMmaIntr<MMAOp::Pmxvbf16ger2, MMAHandlerOp::SubToFunc>),`.
  **L66 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genMmaIntr<MMAOp::Pmxvbf16ger2, MMAHandlerOp::SubToFunc>),`。
- **L67 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"acc", asAddr},`.
  **L67 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"acc", asAddr},`。
- **L68 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"a", asValue},`.
  **L68 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"a", asValue},`。
- **L69 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"b", asValue},`.
  **L69 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"b", asValue},`。
- **L70 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"xmask", asValue},`.
  **L70 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"xmask", asValue},`。
- **L71 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"ymask", asValue},`.
  **L71 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"ymask", asValue},`。
- **L72 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"pmask", asValue}}},`.
  **L72 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"pmask", asValue}}},`。

### Lines 73-96

````cpp
     /*isElemental=*/true},
    {"__ppc_mma_pmxvbf16ger2nn",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genMmaIntr<MMAOp::Pmxvbf16ger2nn,
                         MMAHandlerOp::FirstArgIsResult>),
     {{{"acc", asAddr},
       {"a", asValue},
       {"b", asValue},
       {"xmask", asValue},
       {"ymask", asValue},
       {"pmask", asValue}}},
     /*isElemental=*/true},
    {"__ppc_mma_pmxvbf16ger2np",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genMmaIntr<MMAOp::Pmxvbf16ger2np,
                         MMAHandlerOp::FirstArgIsResult>),
     {{{"acc", asAddr},
       {"a", asValue},
       {"b", asValue},
       {"xmask", asValue},
       {"ymask", asValue},
       {"pmask", asValue}}},
     /*isElemental=*/true},
    {"__ppc_mma_pmxvbf16ger2pn",
````
- **L73 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L73 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L74 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mma_pmxvbf16ger2nn",`.
  **L74 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mma_pmxvbf16ger2nn",`。
- **L75 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L75 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L76 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genMmaIntr<MMAOp::Pmxvbf16ger2nn,`.
  **L76 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genMmaIntr<MMAOp::Pmxvbf16ger2nn,`。
- **L77 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MMAHandlerOp::FirstArgIsResult>),`.
  **L77 CN**: 继续一个多行参数列表、初始化器或聚合项：`MMAHandlerOp::FirstArgIsResult>),`。
- **L78 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"acc", asAddr},`.
  **L78 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"acc", asAddr},`。
- **L79 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"a", asValue},`.
  **L79 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"a", asValue},`。
- **L80 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"b", asValue},`.
  **L80 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"b", asValue},`。
- **L81 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"xmask", asValue},`.
  **L81 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"xmask", asValue},`。
- **L82 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"ymask", asValue},`.
  **L82 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"ymask", asValue},`。
- **L83 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"pmask", asValue}}},`.
  **L83 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"pmask", asValue}}},`。
- **L84 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L84 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L85 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mma_pmxvbf16ger2np",`.
  **L85 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mma_pmxvbf16ger2np",`。
- **L86 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L86 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L87 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genMmaIntr<MMAOp::Pmxvbf16ger2np,`.
  **L87 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genMmaIntr<MMAOp::Pmxvbf16ger2np,`。
- **L88 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MMAHandlerOp::FirstArgIsResult>),`.
  **L88 CN**: 继续一个多行参数列表、初始化器或聚合项：`MMAHandlerOp::FirstArgIsResult>),`。
- **L89 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"acc", asAddr},`.
  **L89 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"acc", asAddr},`。
- **L90 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"a", asValue},`.
  **L90 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"a", asValue},`。
- **L91 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"b", asValue},`.
  **L91 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"b", asValue},`。
- **L92 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"xmask", asValue},`.
  **L92 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"xmask", asValue},`。
- **L93 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"ymask", asValue},`.
  **L93 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"ymask", asValue},`。
- **L94 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"pmask", asValue}}},`.
  **L94 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"pmask", asValue}}},`。
- **L95 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L95 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L96 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mma_pmxvbf16ger2pn",`.
  **L96 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mma_pmxvbf16ger2pn",`。

### Lines 97-120

````cpp
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genMmaIntr<MMAOp::Pmxvbf16ger2pn,
                         MMAHandlerOp::FirstArgIsResult>),
     {{{"acc", asAddr},
       {"a", asValue},
       {"b", asValue},
       {"xmask", asValue},
       {"ymask", asValue},
       {"pmask", asValue}}},
     /*isElemental=*/true},
    {"__ppc_mma_pmxvbf16ger2pp",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genMmaIntr<MMAOp::Pmxvbf16ger2pp,
                         MMAHandlerOp::FirstArgIsResult>),
     {{{"acc", asAddr},
       {"a", asValue},
       {"b", asValue},
       {"xmask", asValue},
       {"ymask", asValue},
       {"pmask", asValue}}},
     /*isElemental=*/true},
    {"__ppc_mma_pmxvf16ger2_",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genMmaIntr<MMAOp::Pmxvf16ger2, MMAHandlerOp::SubToFunc>),
````
- **L97 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L97 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L98 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genMmaIntr<MMAOp::Pmxvbf16ger2pn,`.
  **L98 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genMmaIntr<MMAOp::Pmxvbf16ger2pn,`。
- **L99 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MMAHandlerOp::FirstArgIsResult>),`.
  **L99 CN**: 继续一个多行参数列表、初始化器或聚合项：`MMAHandlerOp::FirstArgIsResult>),`。
- **L100 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"acc", asAddr},`.
  **L100 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"acc", asAddr},`。
- **L101 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"a", asValue},`.
  **L101 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"a", asValue},`。
- **L102 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"b", asValue},`.
  **L102 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"b", asValue},`。
- **L103 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"xmask", asValue},`.
  **L103 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"xmask", asValue},`。
- **L104 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"ymask", asValue},`.
  **L104 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"ymask", asValue},`。
- **L105 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"pmask", asValue}}},`.
  **L105 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"pmask", asValue}}},`。
- **L106 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L106 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L107 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mma_pmxvbf16ger2pp",`.
  **L107 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mma_pmxvbf16ger2pp",`。
- **L108 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L108 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L109 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genMmaIntr<MMAOp::Pmxvbf16ger2pp,`.
  **L109 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genMmaIntr<MMAOp::Pmxvbf16ger2pp,`。
- **L110 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MMAHandlerOp::FirstArgIsResult>),`.
  **L110 CN**: 继续一个多行参数列表、初始化器或聚合项：`MMAHandlerOp::FirstArgIsResult>),`。
- **L111 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"acc", asAddr},`.
  **L111 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"acc", asAddr},`。
- **L112 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"a", asValue},`.
  **L112 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"a", asValue},`。
- **L113 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"b", asValue},`.
  **L113 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"b", asValue},`。
- **L114 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"xmask", asValue},`.
  **L114 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"xmask", asValue},`。
- **L115 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"ymask", asValue},`.
  **L115 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"ymask", asValue},`。
- **L116 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"pmask", asValue}}},`.
  **L116 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"pmask", asValue}}},`。
- **L117 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L117 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L118 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mma_pmxvf16ger2_",`.
  **L118 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mma_pmxvf16ger2_",`。
- **L119 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L119 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L120 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genMmaIntr<MMAOp::Pmxvf16ger2, MMAHandlerOp::SubToFunc>),`.
  **L120 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genMmaIntr<MMAOp::Pmxvf16ger2, MMAHandlerOp::SubToFunc>),`。

### Lines 121-144

````cpp
     {{{"acc", asAddr},
       {"a", asValue},
       {"b", asValue},
       {"xmask", asValue},
       {"ymask", asValue},
       {"pmask", asValue}}},
     /*isElemental=*/true},
    {"__ppc_mma_pmxvf16ger2nn",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genMmaIntr<MMAOp::Pmxvf16ger2nn, MMAHandlerOp::FirstArgIsResult>),
     {{{"acc", asAddr},
       {"a", asValue},
       {"b", asValue},
       {"xmask", asValue},
       {"ymask", asValue},
       {"pmask", asValue}}},
     /*isElemental=*/true},
    {"__ppc_mma_pmxvf16ger2np",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genMmaIntr<MMAOp::Pmxvf16ger2np, MMAHandlerOp::FirstArgIsResult>),
     {{{"acc", asAddr},
       {"a", asValue},
       {"b", asValue},
       {"xmask", asValue},
````
- **L121 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"acc", asAddr},`.
  **L121 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"acc", asAddr},`。
- **L122 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"a", asValue},`.
  **L122 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"a", asValue},`。
- **L123 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"b", asValue},`.
  **L123 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"b", asValue},`。
- **L124 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"xmask", asValue},`.
  **L124 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"xmask", asValue},`。
- **L125 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"ymask", asValue},`.
  **L125 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"ymask", asValue},`。
- **L126 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"pmask", asValue}}},`.
  **L126 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"pmask", asValue}}},`。
- **L127 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L127 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L128 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mma_pmxvf16ger2nn",`.
  **L128 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mma_pmxvf16ger2nn",`。
- **L129 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L129 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L130 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genMmaIntr<MMAOp::Pmxvf16ger2nn, MMAHandlerOp::FirstArgIsResult>),`.
  **L130 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genMmaIntr<MMAOp::Pmxvf16ger2nn, MMAHandlerOp::FirstArgIsResult>),`。
- **L131 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"acc", asAddr},`.
  **L131 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"acc", asAddr},`。
- **L132 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"a", asValue},`.
  **L132 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"a", asValue},`。
- **L133 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"b", asValue},`.
  **L133 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"b", asValue},`。
- **L134 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"xmask", asValue},`.
  **L134 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"xmask", asValue},`。
- **L135 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"ymask", asValue},`.
  **L135 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"ymask", asValue},`。
- **L136 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"pmask", asValue}}},`.
  **L136 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"pmask", asValue}}},`。
- **L137 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L137 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L138 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mma_pmxvf16ger2np",`.
  **L138 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mma_pmxvf16ger2np",`。
- **L139 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L139 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L140 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genMmaIntr<MMAOp::Pmxvf16ger2np, MMAHandlerOp::FirstArgIsResult>),`.
  **L140 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genMmaIntr<MMAOp::Pmxvf16ger2np, MMAHandlerOp::FirstArgIsResult>),`。
- **L141 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"acc", asAddr},`.
  **L141 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"acc", asAddr},`。
- **L142 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"a", asValue},`.
  **L142 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"a", asValue},`。
- **L143 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"b", asValue},`.
  **L143 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"b", asValue},`。
- **L144 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"xmask", asValue},`.
  **L144 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"xmask", asValue},`。

### Lines 145-168

````cpp
       {"ymask", asValue},
       {"pmask", asValue}}},
     /*isElemental=*/true},
    {"__ppc_mma_pmxvf16ger2pn",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genMmaIntr<MMAOp::Pmxvf16ger2pn, MMAHandlerOp::FirstArgIsResult>),
     {{{"acc", asAddr},
       {"a", asValue},
       {"b", asValue},
       {"xmask", asValue},
       {"ymask", asValue},
       {"pmask", asValue}}},
     /*isElemental=*/true},
    {"__ppc_mma_pmxvf16ger2pp",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genMmaIntr<MMAOp::Pmxvf16ger2pp, MMAHandlerOp::FirstArgIsResult>),
     {{{"acc", asAddr},
       {"a", asValue},
       {"b", asValue},
       {"xmask", asValue},
       {"ymask", asValue},
       {"pmask", asValue}}},
     /*isElemental=*/true},
    {"__ppc_mma_pmxvf32ger",
````
- **L145 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"ymask", asValue},`.
  **L145 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"ymask", asValue},`。
- **L146 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"pmask", asValue}}},`.
  **L146 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"pmask", asValue}}},`。
- **L147 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L147 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L148 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mma_pmxvf16ger2pn",`.
  **L148 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mma_pmxvf16ger2pn",`。
- **L149 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L149 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L150 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genMmaIntr<MMAOp::Pmxvf16ger2pn, MMAHandlerOp::FirstArgIsResult>),`.
  **L150 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genMmaIntr<MMAOp::Pmxvf16ger2pn, MMAHandlerOp::FirstArgIsResult>),`。
- **L151 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"acc", asAddr},`.
  **L151 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"acc", asAddr},`。
- **L152 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"a", asValue},`.
  **L152 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"a", asValue},`。
- **L153 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"b", asValue},`.
  **L153 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"b", asValue},`。
- **L154 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"xmask", asValue},`.
  **L154 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"xmask", asValue},`。
- **L155 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"ymask", asValue},`.
  **L155 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"ymask", asValue},`。
- **L156 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"pmask", asValue}}},`.
  **L156 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"pmask", asValue}}},`。
- **L157 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L157 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L158 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mma_pmxvf16ger2pp",`.
  **L158 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mma_pmxvf16ger2pp",`。
- **L159 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L159 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L160 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genMmaIntr<MMAOp::Pmxvf16ger2pp, MMAHandlerOp::FirstArgIsResult>),`.
  **L160 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genMmaIntr<MMAOp::Pmxvf16ger2pp, MMAHandlerOp::FirstArgIsResult>),`。
- **L161 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"acc", asAddr},`.
  **L161 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"acc", asAddr},`。
- **L162 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"a", asValue},`.
  **L162 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"a", asValue},`。
- **L163 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"b", asValue},`.
  **L163 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"b", asValue},`。
- **L164 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"xmask", asValue},`.
  **L164 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"xmask", asValue},`。
- **L165 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"ymask", asValue},`.
  **L165 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"ymask", asValue},`。
- **L166 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"pmask", asValue}}},`.
  **L166 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"pmask", asValue}}},`。
- **L167 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L167 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L168 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mma_pmxvf32ger",`.
  **L168 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mma_pmxvf32ger",`。

### Lines 169-192

````cpp
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genMmaIntr<MMAOp::Pmxvf32ger, MMAHandlerOp::SubToFunc>),
     {{{"acc", asAddr},
       {"a", asValue},
       {"b", asValue},
       {"xmask", asValue},
       {"ymask", asValue}}},
     /*isElemental=*/true},
    {"__ppc_mma_pmxvf32gernn",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genMmaIntr<MMAOp::Pmxvf32gernn, MMAHandlerOp::FirstArgIsResult>),
     {{{"acc", asAddr},
       {"a", asValue},
       {"b", asValue},
       {"xmask", asValue},
       {"ymask", asValue}}},
     /*isElemental=*/true},
    {"__ppc_mma_pmxvf32gernp",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genMmaIntr<MMAOp::Pmxvf32gernp, MMAHandlerOp::FirstArgIsResult>),
     {{{"acc", asAddr},
       {"a", asValue},
       {"b", asValue},
       {"xmask", asValue},
````
- **L169 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L169 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L170 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genMmaIntr<MMAOp::Pmxvf32ger, MMAHandlerOp::SubToFunc>),`.
  **L170 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genMmaIntr<MMAOp::Pmxvf32ger, MMAHandlerOp::SubToFunc>),`。
- **L171 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"acc", asAddr},`.
  **L171 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"acc", asAddr},`。
- **L172 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"a", asValue},`.
  **L172 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"a", asValue},`。
- **L173 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"b", asValue},`.
  **L173 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"b", asValue},`。
- **L174 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"xmask", asValue},`.
  **L174 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"xmask", asValue},`。
- **L175 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"ymask", asValue}}},`.
  **L175 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"ymask", asValue}}},`。
- **L176 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L176 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L177 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mma_pmxvf32gernn",`.
  **L177 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mma_pmxvf32gernn",`。
- **L178 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L178 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L179 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genMmaIntr<MMAOp::Pmxvf32gernn, MMAHandlerOp::FirstArgIsResult>),`.
  **L179 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genMmaIntr<MMAOp::Pmxvf32gernn, MMAHandlerOp::FirstArgIsResult>),`。
- **L180 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"acc", asAddr},`.
  **L180 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"acc", asAddr},`。
- **L181 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"a", asValue},`.
  **L181 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"a", asValue},`。
- **L182 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"b", asValue},`.
  **L182 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"b", asValue},`。
- **L183 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"xmask", asValue},`.
  **L183 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"xmask", asValue},`。
- **L184 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"ymask", asValue}}},`.
  **L184 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"ymask", asValue}}},`。
- **L185 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L185 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L186 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mma_pmxvf32gernp",`.
  **L186 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mma_pmxvf32gernp",`。
- **L187 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L187 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L188 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genMmaIntr<MMAOp::Pmxvf32gernp, MMAHandlerOp::FirstArgIsResult>),`.
  **L188 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genMmaIntr<MMAOp::Pmxvf32gernp, MMAHandlerOp::FirstArgIsResult>),`。
- **L189 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"acc", asAddr},`.
  **L189 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"acc", asAddr},`。
- **L190 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"a", asValue},`.
  **L190 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"a", asValue},`。
- **L191 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"b", asValue},`.
  **L191 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"b", asValue},`。
- **L192 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"xmask", asValue},`.
  **L192 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"xmask", asValue},`。

### Lines 193-216

````cpp
       {"ymask", asValue}}},
     /*isElemental=*/true},
    {"__ppc_mma_pmxvf32gerpn",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genMmaIntr<MMAOp::Pmxvf32gerpn, MMAHandlerOp::FirstArgIsResult>),
     {{{"acc", asAddr},
       {"a", asValue},
       {"b", asValue},
       {"xmask", asValue},
       {"ymask", asValue}}},
     /*isElemental=*/true},
    {"__ppc_mma_pmxvf32gerpp",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genMmaIntr<MMAOp::Pmxvf32gerpp, MMAHandlerOp::FirstArgIsResult>),
     {{{"acc", asAddr},
       {"a", asValue},
       {"b", asValue},
       {"xmask", asValue},
       {"ymask", asValue}}},
     /*isElemental=*/true},
    {"__ppc_mma_pmxvf64ger",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genMmaIntr<MMAOp::Pmxvf64ger, MMAHandlerOp::SubToFunc>),
     {{{"acc", asAddr},
````
- **L193 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"ymask", asValue}}},`.
  **L193 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"ymask", asValue}}},`。
- **L194 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L194 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L195 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mma_pmxvf32gerpn",`.
  **L195 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mma_pmxvf32gerpn",`。
- **L196 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L196 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L197 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genMmaIntr<MMAOp::Pmxvf32gerpn, MMAHandlerOp::FirstArgIsResult>),`.
  **L197 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genMmaIntr<MMAOp::Pmxvf32gerpn, MMAHandlerOp::FirstArgIsResult>),`。
- **L198 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"acc", asAddr},`.
  **L198 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"acc", asAddr},`。
- **L199 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"a", asValue},`.
  **L199 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"a", asValue},`。
- **L200 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"b", asValue},`.
  **L200 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"b", asValue},`。
- **L201 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"xmask", asValue},`.
  **L201 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"xmask", asValue},`。
- **L202 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"ymask", asValue}}},`.
  **L202 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"ymask", asValue}}},`。
- **L203 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L203 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L204 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mma_pmxvf32gerpp",`.
  **L204 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mma_pmxvf32gerpp",`。
- **L205 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L205 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L206 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genMmaIntr<MMAOp::Pmxvf32gerpp, MMAHandlerOp::FirstArgIsResult>),`.
  **L206 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genMmaIntr<MMAOp::Pmxvf32gerpp, MMAHandlerOp::FirstArgIsResult>),`。
- **L207 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"acc", asAddr},`.
  **L207 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"acc", asAddr},`。
- **L208 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"a", asValue},`.
  **L208 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"a", asValue},`。
- **L209 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"b", asValue},`.
  **L209 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"b", asValue},`。
- **L210 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"xmask", asValue},`.
  **L210 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"xmask", asValue},`。
- **L211 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"ymask", asValue}}},`.
  **L211 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"ymask", asValue}}},`。
- **L212 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L212 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L213 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mma_pmxvf64ger",`.
  **L213 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mma_pmxvf64ger",`。
- **L214 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L214 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L215 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genMmaIntr<MMAOp::Pmxvf64ger, MMAHandlerOp::SubToFunc>),`.
  **L215 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genMmaIntr<MMAOp::Pmxvf64ger, MMAHandlerOp::SubToFunc>),`。
- **L216 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"acc", asAddr},`.
  **L216 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"acc", asAddr},`。

### Lines 217-240

````cpp
       {"a", asValue},
       {"b", asValue},
       {"xmask", asValue},
       {"ymask", asValue}}},
     /*isElemental=*/true},
    {"__ppc_mma_pmxvf64gernn",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genMmaIntr<MMAOp::Pmxvf64gernn, MMAHandlerOp::FirstArgIsResult>),
     {{{"acc", asAddr},
       {"a", asValue},
       {"b", asValue},
       {"xmask", asValue},
       {"ymask", asValue}}},
     /*isElemental=*/true},
    {"__ppc_mma_pmxvf64gernp",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genMmaIntr<MMAOp::Pmxvf64gernp, MMAHandlerOp::FirstArgIsResult>),
     {{{"acc", asAddr},
       {"a", asValue},
       {"b", asValue},
       {"xmask", asValue},
       {"ymask", asValue}}},
     /*isElemental=*/true},
    {"__ppc_mma_pmxvf64gerpn",
````
- **L217 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"a", asValue},`.
  **L217 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"a", asValue},`。
- **L218 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"b", asValue},`.
  **L218 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"b", asValue},`。
- **L219 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"xmask", asValue},`.
  **L219 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"xmask", asValue},`。
- **L220 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"ymask", asValue}}},`.
  **L220 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"ymask", asValue}}},`。
- **L221 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L221 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L222 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mma_pmxvf64gernn",`.
  **L222 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mma_pmxvf64gernn",`。
- **L223 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L223 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L224 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genMmaIntr<MMAOp::Pmxvf64gernn, MMAHandlerOp::FirstArgIsResult>),`.
  **L224 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genMmaIntr<MMAOp::Pmxvf64gernn, MMAHandlerOp::FirstArgIsResult>),`。
- **L225 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"acc", asAddr},`.
  **L225 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"acc", asAddr},`。
- **L226 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"a", asValue},`.
  **L226 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"a", asValue},`。
- **L227 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"b", asValue},`.
  **L227 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"b", asValue},`。
- **L228 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"xmask", asValue},`.
  **L228 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"xmask", asValue},`。
- **L229 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"ymask", asValue}}},`.
  **L229 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"ymask", asValue}}},`。
- **L230 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L230 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L231 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mma_pmxvf64gernp",`.
  **L231 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mma_pmxvf64gernp",`。
- **L232 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L232 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L233 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genMmaIntr<MMAOp::Pmxvf64gernp, MMAHandlerOp::FirstArgIsResult>),`.
  **L233 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genMmaIntr<MMAOp::Pmxvf64gernp, MMAHandlerOp::FirstArgIsResult>),`。
- **L234 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"acc", asAddr},`.
  **L234 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"acc", asAddr},`。
- **L235 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"a", asValue},`.
  **L235 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"a", asValue},`。
- **L236 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"b", asValue},`.
  **L236 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"b", asValue},`。
- **L237 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"xmask", asValue},`.
  **L237 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"xmask", asValue},`。
- **L238 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"ymask", asValue}}},`.
  **L238 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"ymask", asValue}}},`。
- **L239 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L239 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L240 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mma_pmxvf64gerpn",`.
  **L240 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mma_pmxvf64gerpn",`。

### Lines 241-264

````cpp
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genMmaIntr<MMAOp::Pmxvf64gerpn, MMAHandlerOp::FirstArgIsResult>),
     {{{"acc", asAddr},
       {"a", asValue},
       {"b", asValue},
       {"xmask", asValue},
       {"ymask", asValue}}},
     /*isElemental=*/true},
    {"__ppc_mma_pmxvf64gerpp",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genMmaIntr<MMAOp::Pmxvf64gerpp, MMAHandlerOp::FirstArgIsResult>),
     {{{"acc", asAddr},
       {"a", asValue},
       {"b", asValue},
       {"xmask", asValue},
       {"ymask", asValue}}},
     /*isElemental=*/true},
    {"__ppc_mma_pmxvi16ger2_",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genMmaIntr<MMAOp::Pmxvi16ger2, MMAHandlerOp::SubToFunc>),
     {{{"acc", asAddr},
       {"a", asValue},
       {"b", asValue},
       {"xmask", asValue},
````
- **L241 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L241 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L242 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genMmaIntr<MMAOp::Pmxvf64gerpn, MMAHandlerOp::FirstArgIsResult>),`.
  **L242 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genMmaIntr<MMAOp::Pmxvf64gerpn, MMAHandlerOp::FirstArgIsResult>),`。
- **L243 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"acc", asAddr},`.
  **L243 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"acc", asAddr},`。
- **L244 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"a", asValue},`.
  **L244 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"a", asValue},`。
- **L245 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"b", asValue},`.
  **L245 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"b", asValue},`。
- **L246 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"xmask", asValue},`.
  **L246 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"xmask", asValue},`。
- **L247 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"ymask", asValue}}},`.
  **L247 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"ymask", asValue}}},`。
- **L248 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L248 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L249 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mma_pmxvf64gerpp",`.
  **L249 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mma_pmxvf64gerpp",`。
- **L250 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L250 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L251 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genMmaIntr<MMAOp::Pmxvf64gerpp, MMAHandlerOp::FirstArgIsResult>),`.
  **L251 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genMmaIntr<MMAOp::Pmxvf64gerpp, MMAHandlerOp::FirstArgIsResult>),`。
- **L252 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"acc", asAddr},`.
  **L252 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"acc", asAddr},`。
- **L253 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"a", asValue},`.
  **L253 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"a", asValue},`。
- **L254 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"b", asValue},`.
  **L254 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"b", asValue},`。
- **L255 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"xmask", asValue},`.
  **L255 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"xmask", asValue},`。
- **L256 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"ymask", asValue}}},`.
  **L256 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"ymask", asValue}}},`。
- **L257 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L257 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L258 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mma_pmxvi16ger2_",`.
  **L258 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mma_pmxvi16ger2_",`。
- **L259 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L259 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L260 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genMmaIntr<MMAOp::Pmxvi16ger2, MMAHandlerOp::SubToFunc>),`.
  **L260 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genMmaIntr<MMAOp::Pmxvi16ger2, MMAHandlerOp::SubToFunc>),`。
- **L261 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"acc", asAddr},`.
  **L261 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"acc", asAddr},`。
- **L262 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"a", asValue},`.
  **L262 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"a", asValue},`。
- **L263 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"b", asValue},`.
  **L263 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"b", asValue},`。
- **L264 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"xmask", asValue},`.
  **L264 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"xmask", asValue},`。

### Lines 265-288

````cpp
       {"ymask", asValue},
       {"pmask", asValue}}},
     /*isElemental=*/true},
    {"__ppc_mma_pmxvi16ger2pp",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genMmaIntr<MMAOp::Pmxvi16ger2pp, MMAHandlerOp::FirstArgIsResult>),
     {{{"acc", asAddr},
       {"a", asValue},
       {"b", asValue},
       {"xmask", asValue},
       {"ymask", asValue},
       {"pmask", asValue}}},
     /*isElemental=*/true},
    {"__ppc_mma_pmxvi16ger2s",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genMmaIntr<MMAOp::Pmxvi16ger2s, MMAHandlerOp::SubToFunc>),
     {{{"acc", asAddr},
       {"a", asValue},
       {"b", asValue},
       {"xmask", asValue},
       {"ymask", asValue},
       {"pmask", asValue}}},
     /*isElemental=*/true},
    {"__ppc_mma_pmxvi16ger2spp",
````
- **L265 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"ymask", asValue},`.
  **L265 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"ymask", asValue},`。
- **L266 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"pmask", asValue}}},`.
  **L266 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"pmask", asValue}}},`。
- **L267 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L267 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L268 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mma_pmxvi16ger2pp",`.
  **L268 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mma_pmxvi16ger2pp",`。
- **L269 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L269 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L270 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genMmaIntr<MMAOp::Pmxvi16ger2pp, MMAHandlerOp::FirstArgIsResult>),`.
  **L270 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genMmaIntr<MMAOp::Pmxvi16ger2pp, MMAHandlerOp::FirstArgIsResult>),`。
- **L271 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"acc", asAddr},`.
  **L271 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"acc", asAddr},`。
- **L272 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"a", asValue},`.
  **L272 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"a", asValue},`。
- **L273 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"b", asValue},`.
  **L273 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"b", asValue},`。
- **L274 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"xmask", asValue},`.
  **L274 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"xmask", asValue},`。
- **L275 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"ymask", asValue},`.
  **L275 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"ymask", asValue},`。
- **L276 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"pmask", asValue}}},`.
  **L276 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"pmask", asValue}}},`。
- **L277 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L277 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L278 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mma_pmxvi16ger2s",`.
  **L278 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mma_pmxvi16ger2s",`。
- **L279 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L279 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L280 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genMmaIntr<MMAOp::Pmxvi16ger2s, MMAHandlerOp::SubToFunc>),`.
  **L280 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genMmaIntr<MMAOp::Pmxvi16ger2s, MMAHandlerOp::SubToFunc>),`。
- **L281 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"acc", asAddr},`.
  **L281 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"acc", asAddr},`。
- **L282 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"a", asValue},`.
  **L282 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"a", asValue},`。
- **L283 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"b", asValue},`.
  **L283 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"b", asValue},`。
- **L284 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"xmask", asValue},`.
  **L284 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"xmask", asValue},`。
- **L285 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"ymask", asValue},`.
  **L285 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"ymask", asValue},`。
- **L286 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"pmask", asValue}}},`.
  **L286 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"pmask", asValue}}},`。
- **L287 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L287 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L288 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mma_pmxvi16ger2spp",`.
  **L288 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mma_pmxvi16ger2spp",`。

### Lines 289-312

````cpp
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genMmaIntr<MMAOp::Pmxvi16ger2spp,
                         MMAHandlerOp::FirstArgIsResult>),
     {{{"acc", asAddr},
       {"a", asValue},
       {"b", asValue},
       {"xmask", asValue},
       {"ymask", asValue},
       {"pmask", asValue}}},
     /*isElemental=*/true},
    {"__ppc_mma_pmxvi4ger8_",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genMmaIntr<MMAOp::Pmxvi4ger8, MMAHandlerOp::SubToFunc>),
     {{{"acc", asAddr},
       {"a", asValue},
       {"b", asValue},
       {"xmask", asValue},
       {"ymask", asValue},
       {"pmask", asValue}}},
     /*isElemental=*/true},
    {"__ppc_mma_pmxvi4ger8pp",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genMmaIntr<MMAOp::Pmxvi4ger8pp, MMAHandlerOp::FirstArgIsResult>),
     {{{"acc", asAddr},
````
- **L289 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L289 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L290 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genMmaIntr<MMAOp::Pmxvi16ger2spp,`.
  **L290 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genMmaIntr<MMAOp::Pmxvi16ger2spp,`。
- **L291 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MMAHandlerOp::FirstArgIsResult>),`.
  **L291 CN**: 继续一个多行参数列表、初始化器或聚合项：`MMAHandlerOp::FirstArgIsResult>),`。
- **L292 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"acc", asAddr},`.
  **L292 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"acc", asAddr},`。
- **L293 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"a", asValue},`.
  **L293 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"a", asValue},`。
- **L294 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"b", asValue},`.
  **L294 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"b", asValue},`。
- **L295 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"xmask", asValue},`.
  **L295 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"xmask", asValue},`。
- **L296 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"ymask", asValue},`.
  **L296 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"ymask", asValue},`。
- **L297 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"pmask", asValue}}},`.
  **L297 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"pmask", asValue}}},`。
- **L298 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L298 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L299 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mma_pmxvi4ger8_",`.
  **L299 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mma_pmxvi4ger8_",`。
- **L300 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L300 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L301 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genMmaIntr<MMAOp::Pmxvi4ger8, MMAHandlerOp::SubToFunc>),`.
  **L301 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genMmaIntr<MMAOp::Pmxvi4ger8, MMAHandlerOp::SubToFunc>),`。
- **L302 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"acc", asAddr},`.
  **L302 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"acc", asAddr},`。
- **L303 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"a", asValue},`.
  **L303 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"a", asValue},`。
- **L304 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"b", asValue},`.
  **L304 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"b", asValue},`。
- **L305 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"xmask", asValue},`.
  **L305 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"xmask", asValue},`。
- **L306 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"ymask", asValue},`.
  **L306 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"ymask", asValue},`。
- **L307 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"pmask", asValue}}},`.
  **L307 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"pmask", asValue}}},`。
- **L308 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L308 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L309 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mma_pmxvi4ger8pp",`.
  **L309 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mma_pmxvi4ger8pp",`。
- **L310 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L310 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L311 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genMmaIntr<MMAOp::Pmxvi4ger8pp, MMAHandlerOp::FirstArgIsResult>),`.
  **L311 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genMmaIntr<MMAOp::Pmxvi4ger8pp, MMAHandlerOp::FirstArgIsResult>),`。
- **L312 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"acc", asAddr},`.
  **L312 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"acc", asAddr},`。

### Lines 313-336

````cpp
       {"a", asValue},
       {"b", asValue},
       {"xmask", asValue},
       {"ymask", asValue},
       {"pmask", asValue}}},
     /*isElemental=*/true},
    {"__ppc_mma_pmxvi8ger4_",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genMmaIntr<MMAOp::Pmxvi8ger4, MMAHandlerOp::SubToFunc>),
     {{{"acc", asAddr},
       {"a", asValue},
       {"b", asValue},
       {"xmask", asValue},
       {"ymask", asValue},
       {"pmask", asValue}}},
     /*isElemental=*/true},
    {"__ppc_mma_pmxvi8ger4pp",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genMmaIntr<MMAOp::Pmxvi8ger4pp, MMAHandlerOp::FirstArgIsResult>),
     {{{"acc", asAddr},
       {"a", asValue},
       {"b", asValue},
       {"xmask", asValue},
       {"ymask", asValue},
````
- **L313 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"a", asValue},`.
  **L313 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"a", asValue},`。
- **L314 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"b", asValue},`.
  **L314 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"b", asValue},`。
- **L315 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"xmask", asValue},`.
  **L315 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"xmask", asValue},`。
- **L316 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"ymask", asValue},`.
  **L316 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"ymask", asValue},`。
- **L317 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"pmask", asValue}}},`.
  **L317 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"pmask", asValue}}},`。
- **L318 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L318 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L319 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mma_pmxvi8ger4_",`.
  **L319 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mma_pmxvi8ger4_",`。
- **L320 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L320 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L321 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genMmaIntr<MMAOp::Pmxvi8ger4, MMAHandlerOp::SubToFunc>),`.
  **L321 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genMmaIntr<MMAOp::Pmxvi8ger4, MMAHandlerOp::SubToFunc>),`。
- **L322 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"acc", asAddr},`.
  **L322 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"acc", asAddr},`。
- **L323 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"a", asValue},`.
  **L323 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"a", asValue},`。
- **L324 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"b", asValue},`.
  **L324 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"b", asValue},`。
- **L325 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"xmask", asValue},`.
  **L325 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"xmask", asValue},`。
- **L326 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"ymask", asValue},`.
  **L326 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"ymask", asValue},`。
- **L327 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"pmask", asValue}}},`.
  **L327 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"pmask", asValue}}},`。
- **L328 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L328 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L329 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mma_pmxvi8ger4pp",`.
  **L329 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mma_pmxvi8ger4pp",`。
- **L330 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L330 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L331 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genMmaIntr<MMAOp::Pmxvi8ger4pp, MMAHandlerOp::FirstArgIsResult>),`.
  **L331 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genMmaIntr<MMAOp::Pmxvi8ger4pp, MMAHandlerOp::FirstArgIsResult>),`。
- **L332 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"acc", asAddr},`.
  **L332 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"acc", asAddr},`。
- **L333 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"a", asValue},`.
  **L333 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"a", asValue},`。
- **L334 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"b", asValue},`.
  **L334 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"b", asValue},`。
- **L335 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"xmask", asValue},`.
  **L335 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"xmask", asValue},`。
- **L336 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"ymask", asValue},`.
  **L336 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"ymask", asValue},`。

### Lines 337-360

````cpp
       {"pmask", asValue}}},
     /*isElemental=*/true},
    {"__ppc_mma_pmxvi8ger4spp",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genMmaIntr<MMAOp::Pmxvi8ger4spp, MMAHandlerOp::FirstArgIsResult>),
     {{{"acc", asAddr},
       {"a", asValue},
       {"b", asValue},
       {"xmask", asValue},
       {"ymask", asValue},
       {"pmask", asValue}}},
     /*isElemental=*/true},
    {"__ppc_mma_xvbf16ger2_",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genMmaIntr<MMAOp::Xvbf16ger2, MMAHandlerOp::SubToFunc>),
     {{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},
     /*isElemental=*/true},
    {"__ppc_mma_xvbf16ger2nn",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genMmaIntr<MMAOp::Xvbf16ger2nn, MMAHandlerOp::FirstArgIsResult>),
     {{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},
     /*isElemental=*/true},
    {"__ppc_mma_xvbf16ger2np",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
````
- **L337 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"pmask", asValue}}},`.
  **L337 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"pmask", asValue}}},`。
- **L338 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L338 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L339 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mma_pmxvi8ger4spp",`.
  **L339 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mma_pmxvi8ger4spp",`。
- **L340 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L340 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L341 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genMmaIntr<MMAOp::Pmxvi8ger4spp, MMAHandlerOp::FirstArgIsResult>),`.
  **L341 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genMmaIntr<MMAOp::Pmxvi8ger4spp, MMAHandlerOp::FirstArgIsResult>),`。
- **L342 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"acc", asAddr},`.
  **L342 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"acc", asAddr},`。
- **L343 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"a", asValue},`.
  **L343 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"a", asValue},`。
- **L344 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"b", asValue},`.
  **L344 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"b", asValue},`。
- **L345 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"xmask", asValue},`.
  **L345 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"xmask", asValue},`。
- **L346 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"ymask", asValue},`.
  **L346 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"ymask", asValue},`。
- **L347 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"pmask", asValue}}},`.
  **L347 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"pmask", asValue}}},`。
- **L348 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L348 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L349 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mma_xvbf16ger2_",`.
  **L349 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mma_xvbf16ger2_",`。
- **L350 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L350 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L351 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genMmaIntr<MMAOp::Xvbf16ger2, MMAHandlerOp::SubToFunc>),`.
  **L351 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genMmaIntr<MMAOp::Xvbf16ger2, MMAHandlerOp::SubToFunc>),`。
- **L352 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},`.
  **L352 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},`。
- **L353 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L353 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L354 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mma_xvbf16ger2nn",`.
  **L354 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mma_xvbf16ger2nn",`。
- **L355 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L355 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L356 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genMmaIntr<MMAOp::Xvbf16ger2nn, MMAHandlerOp::FirstArgIsResult>),`.
  **L356 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genMmaIntr<MMAOp::Xvbf16ger2nn, MMAHandlerOp::FirstArgIsResult>),`。
- **L357 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},`.
  **L357 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},`。
- **L358 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L358 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L359 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mma_xvbf16ger2np",`.
  **L359 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mma_xvbf16ger2np",`。
- **L360 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L360 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。

### Lines 361-384

````cpp
         &PI::genMmaIntr<MMAOp::Xvbf16ger2np, MMAHandlerOp::FirstArgIsResult>),
     {{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},
     /*isElemental=*/true},
    {"__ppc_mma_xvbf16ger2pn",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genMmaIntr<MMAOp::Xvbf16ger2pn, MMAHandlerOp::FirstArgIsResult>),
     {{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},
     /*isElemental=*/true},
    {"__ppc_mma_xvbf16ger2pp",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genMmaIntr<MMAOp::Xvbf16ger2pp, MMAHandlerOp::FirstArgIsResult>),
     {{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},
     /*isElemental=*/true},
    {"__ppc_mma_xvf16ger2_",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genMmaIntr<MMAOp::Xvf16ger2, MMAHandlerOp::SubToFunc>),
     {{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},
     /*isElemental=*/true},
    {"__ppc_mma_xvf16ger2nn",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genMmaIntr<MMAOp::Xvf16ger2nn, MMAHandlerOp::FirstArgIsResult>),
     {{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},
     /*isElemental=*/true},
    {"__ppc_mma_xvf16ger2np",
````
- **L361 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genMmaIntr<MMAOp::Xvbf16ger2np, MMAHandlerOp::FirstArgIsResult>),`.
  **L361 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genMmaIntr<MMAOp::Xvbf16ger2np, MMAHandlerOp::FirstArgIsResult>),`。
- **L362 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},`.
  **L362 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},`。
- **L363 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L363 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L364 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mma_xvbf16ger2pn",`.
  **L364 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mma_xvbf16ger2pn",`。
- **L365 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L365 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L366 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genMmaIntr<MMAOp::Xvbf16ger2pn, MMAHandlerOp::FirstArgIsResult>),`.
  **L366 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genMmaIntr<MMAOp::Xvbf16ger2pn, MMAHandlerOp::FirstArgIsResult>),`。
- **L367 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},`.
  **L367 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},`。
- **L368 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L368 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L369 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mma_xvbf16ger2pp",`.
  **L369 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mma_xvbf16ger2pp",`。
- **L370 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L370 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L371 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genMmaIntr<MMAOp::Xvbf16ger2pp, MMAHandlerOp::FirstArgIsResult>),`.
  **L371 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genMmaIntr<MMAOp::Xvbf16ger2pp, MMAHandlerOp::FirstArgIsResult>),`。
- **L372 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},`.
  **L372 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},`。
- **L373 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L373 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L374 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mma_xvf16ger2_",`.
  **L374 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mma_xvf16ger2_",`。
- **L375 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L375 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L376 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genMmaIntr<MMAOp::Xvf16ger2, MMAHandlerOp::SubToFunc>),`.
  **L376 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genMmaIntr<MMAOp::Xvf16ger2, MMAHandlerOp::SubToFunc>),`。
- **L377 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},`.
  **L377 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},`。
- **L378 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L378 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L379 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mma_xvf16ger2nn",`.
  **L379 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mma_xvf16ger2nn",`。
- **L380 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L380 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L381 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genMmaIntr<MMAOp::Xvf16ger2nn, MMAHandlerOp::FirstArgIsResult>),`.
  **L381 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genMmaIntr<MMAOp::Xvf16ger2nn, MMAHandlerOp::FirstArgIsResult>),`。
- **L382 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},`.
  **L382 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},`。
- **L383 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L383 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L384 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mma_xvf16ger2np",`.
  **L384 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mma_xvf16ger2np",`。

### Lines 385-408

````cpp
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genMmaIntr<MMAOp::Xvf16ger2np, MMAHandlerOp::FirstArgIsResult>),
     {{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},
     /*isElemental=*/true},
    {"__ppc_mma_xvf16ger2pn",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genMmaIntr<MMAOp::Xvf16ger2pn, MMAHandlerOp::FirstArgIsResult>),
     {{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},
     /*isElemental=*/true},
    {"__ppc_mma_xvf16ger2pp",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genMmaIntr<MMAOp::Xvf16ger2pp, MMAHandlerOp::FirstArgIsResult>),
     {{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},
     /*isElemental=*/true},
    {"__ppc_mma_xvf32ger",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genMmaIntr<MMAOp::Xvf32ger, MMAHandlerOp::SubToFunc>),
     {{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},
     /*isElemental=*/true},
    {"__ppc_mma_xvf32gernn",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genMmaIntr<MMAOp::Xvf32gernn, MMAHandlerOp::FirstArgIsResult>),
     {{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},
     /*isElemental=*/true},
````
- **L385 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L385 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L386 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genMmaIntr<MMAOp::Xvf16ger2np, MMAHandlerOp::FirstArgIsResult>),`.
  **L386 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genMmaIntr<MMAOp::Xvf16ger2np, MMAHandlerOp::FirstArgIsResult>),`。
- **L387 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},`.
  **L387 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},`。
- **L388 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L388 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L389 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mma_xvf16ger2pn",`.
  **L389 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mma_xvf16ger2pn",`。
- **L390 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L390 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L391 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genMmaIntr<MMAOp::Xvf16ger2pn, MMAHandlerOp::FirstArgIsResult>),`.
  **L391 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genMmaIntr<MMAOp::Xvf16ger2pn, MMAHandlerOp::FirstArgIsResult>),`。
- **L392 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},`.
  **L392 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},`。
- **L393 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L393 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L394 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mma_xvf16ger2pp",`.
  **L394 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mma_xvf16ger2pp",`。
- **L395 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L395 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L396 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genMmaIntr<MMAOp::Xvf16ger2pp, MMAHandlerOp::FirstArgIsResult>),`.
  **L396 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genMmaIntr<MMAOp::Xvf16ger2pp, MMAHandlerOp::FirstArgIsResult>),`。
- **L397 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},`.
  **L397 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},`。
- **L398 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L398 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L399 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mma_xvf32ger",`.
  **L399 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mma_xvf32ger",`。
- **L400 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L400 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L401 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genMmaIntr<MMAOp::Xvf32ger, MMAHandlerOp::SubToFunc>),`.
  **L401 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genMmaIntr<MMAOp::Xvf32ger, MMAHandlerOp::SubToFunc>),`。
- **L402 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},`.
  **L402 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},`。
- **L403 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L403 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L404 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mma_xvf32gernn",`.
  **L404 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mma_xvf32gernn",`。
- **L405 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L405 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L406 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genMmaIntr<MMAOp::Xvf32gernn, MMAHandlerOp::FirstArgIsResult>),`.
  **L406 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genMmaIntr<MMAOp::Xvf32gernn, MMAHandlerOp::FirstArgIsResult>),`。
- **L407 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},`.
  **L407 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},`。
- **L408 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L408 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。

### Lines 409-432

````cpp
    {"__ppc_mma_xvf32gernp",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genMmaIntr<MMAOp::Xvf32gernp, MMAHandlerOp::FirstArgIsResult>),
     {{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},
     /*isElemental=*/true},
    {"__ppc_mma_xvf32gerpn",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genMmaIntr<MMAOp::Xvf32gerpn, MMAHandlerOp::FirstArgIsResult>),
     {{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},
     /*isElemental=*/true},
    {"__ppc_mma_xvf32gerpp",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genMmaIntr<MMAOp::Xvf32gerpp, MMAHandlerOp::FirstArgIsResult>),
     {{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},
     /*isElemental=*/true},
    {"__ppc_mma_xvf64ger",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genMmaIntr<MMAOp::Xvf64ger, MMAHandlerOp::SubToFunc>),
     {{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},
     /*isElemental=*/true},
    {"__ppc_mma_xvf64gernn",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genMmaIntr<MMAOp::Xvf64gernn, MMAHandlerOp::FirstArgIsResult>),
     {{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},
````
- **L409 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mma_xvf32gernp",`.
  **L409 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mma_xvf32gernp",`。
- **L410 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L410 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L411 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genMmaIntr<MMAOp::Xvf32gernp, MMAHandlerOp::FirstArgIsResult>),`.
  **L411 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genMmaIntr<MMAOp::Xvf32gernp, MMAHandlerOp::FirstArgIsResult>),`。
- **L412 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},`.
  **L412 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},`。
- **L413 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L413 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L414 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mma_xvf32gerpn",`.
  **L414 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mma_xvf32gerpn",`。
- **L415 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L415 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L416 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genMmaIntr<MMAOp::Xvf32gerpn, MMAHandlerOp::FirstArgIsResult>),`.
  **L416 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genMmaIntr<MMAOp::Xvf32gerpn, MMAHandlerOp::FirstArgIsResult>),`。
- **L417 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},`.
  **L417 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},`。
- **L418 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L418 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L419 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mma_xvf32gerpp",`.
  **L419 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mma_xvf32gerpp",`。
- **L420 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L420 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L421 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genMmaIntr<MMAOp::Xvf32gerpp, MMAHandlerOp::FirstArgIsResult>),`.
  **L421 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genMmaIntr<MMAOp::Xvf32gerpp, MMAHandlerOp::FirstArgIsResult>),`。
- **L422 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},`.
  **L422 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},`。
- **L423 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L423 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L424 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mma_xvf64ger",`.
  **L424 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mma_xvf64ger",`。
- **L425 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L425 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L426 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genMmaIntr<MMAOp::Xvf64ger, MMAHandlerOp::SubToFunc>),`.
  **L426 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genMmaIntr<MMAOp::Xvf64ger, MMAHandlerOp::SubToFunc>),`。
- **L427 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},`.
  **L427 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},`。
- **L428 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L428 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L429 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mma_xvf64gernn",`.
  **L429 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mma_xvf64gernn",`。
- **L430 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L430 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L431 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genMmaIntr<MMAOp::Xvf64gernn, MMAHandlerOp::FirstArgIsResult>),`.
  **L431 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genMmaIntr<MMAOp::Xvf64gernn, MMAHandlerOp::FirstArgIsResult>),`。
- **L432 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},`.
  **L432 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},`。

### Lines 433-456

````cpp
     /*isElemental=*/true},
    {"__ppc_mma_xvf64gernp",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genMmaIntr<MMAOp::Xvf64gernp, MMAHandlerOp::FirstArgIsResult>),
     {{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},
     /*isElemental=*/true},
    {"__ppc_mma_xvf64gerpn",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genMmaIntr<MMAOp::Xvf64gerpn, MMAHandlerOp::FirstArgIsResult>),
     {{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},
     /*isElemental=*/true},
    {"__ppc_mma_xvf64gerpp",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genMmaIntr<MMAOp::Xvf64gerpp, MMAHandlerOp::FirstArgIsResult>),
     {{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},
     /*isElemental=*/true},
    {"__ppc_mma_xvi16ger2_",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genMmaIntr<MMAOp::Xvi16ger2, MMAHandlerOp::SubToFunc>),
     {{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},
     /*isElemental=*/true},
    {"__ppc_mma_xvi16ger2pp",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genMmaIntr<MMAOp::Xvi16ger2pp, MMAHandlerOp::FirstArgIsResult>),
````
- **L433 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L433 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L434 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mma_xvf64gernp",`.
  **L434 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mma_xvf64gernp",`。
- **L435 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L435 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L436 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genMmaIntr<MMAOp::Xvf64gernp, MMAHandlerOp::FirstArgIsResult>),`.
  **L436 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genMmaIntr<MMAOp::Xvf64gernp, MMAHandlerOp::FirstArgIsResult>),`。
- **L437 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},`.
  **L437 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},`。
- **L438 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L438 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L439 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mma_xvf64gerpn",`.
  **L439 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mma_xvf64gerpn",`。
- **L440 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L440 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L441 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genMmaIntr<MMAOp::Xvf64gerpn, MMAHandlerOp::FirstArgIsResult>),`.
  **L441 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genMmaIntr<MMAOp::Xvf64gerpn, MMAHandlerOp::FirstArgIsResult>),`。
- **L442 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},`.
  **L442 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},`。
- **L443 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L443 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L444 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mma_xvf64gerpp",`.
  **L444 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mma_xvf64gerpp",`。
- **L445 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L445 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L446 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genMmaIntr<MMAOp::Xvf64gerpp, MMAHandlerOp::FirstArgIsResult>),`.
  **L446 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genMmaIntr<MMAOp::Xvf64gerpp, MMAHandlerOp::FirstArgIsResult>),`。
- **L447 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},`.
  **L447 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},`。
- **L448 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L448 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L449 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mma_xvi16ger2_",`.
  **L449 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mma_xvi16ger2_",`。
- **L450 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L450 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L451 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genMmaIntr<MMAOp::Xvi16ger2, MMAHandlerOp::SubToFunc>),`.
  **L451 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genMmaIntr<MMAOp::Xvi16ger2, MMAHandlerOp::SubToFunc>),`。
- **L452 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},`.
  **L452 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},`。
- **L453 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L453 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L454 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mma_xvi16ger2pp",`.
  **L454 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mma_xvi16ger2pp",`。
- **L455 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L455 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L456 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genMmaIntr<MMAOp::Xvi16ger2pp, MMAHandlerOp::FirstArgIsResult>),`.
  **L456 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genMmaIntr<MMAOp::Xvi16ger2pp, MMAHandlerOp::FirstArgIsResult>),`。

### Lines 457-480

````cpp
     {{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},
     /*isElemental=*/true},
    {"__ppc_mma_xvi16ger2s",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genMmaIntr<MMAOp::Xvi16ger2s, MMAHandlerOp::SubToFunc>),
     {{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},
     /*isElemental=*/true},
    {"__ppc_mma_xvi16ger2spp",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genMmaIntr<MMAOp::Xvi16ger2spp, MMAHandlerOp::FirstArgIsResult>),
     {{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},
     /*isElemental=*/true},
    {"__ppc_mma_xvi4ger8_",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genMmaIntr<MMAOp::Xvi4ger8, MMAHandlerOp::SubToFunc>),
     {{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},
     /*isElemental=*/true},
    {"__ppc_mma_xvi4ger8pp",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genMmaIntr<MMAOp::Xvi4ger8pp, MMAHandlerOp::FirstArgIsResult>),
     {{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},
     /*isElemental=*/true},
    {"__ppc_mma_xvi8ger4_",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
````
- **L457 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},`.
  **L457 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},`。
- **L458 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L458 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L459 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mma_xvi16ger2s",`.
  **L459 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mma_xvi16ger2s",`。
- **L460 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L460 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L461 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genMmaIntr<MMAOp::Xvi16ger2s, MMAHandlerOp::SubToFunc>),`.
  **L461 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genMmaIntr<MMAOp::Xvi16ger2s, MMAHandlerOp::SubToFunc>),`。
- **L462 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},`.
  **L462 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},`。
- **L463 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L463 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L464 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mma_xvi16ger2spp",`.
  **L464 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mma_xvi16ger2spp",`。
- **L465 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L465 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L466 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genMmaIntr<MMAOp::Xvi16ger2spp, MMAHandlerOp::FirstArgIsResult>),`.
  **L466 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genMmaIntr<MMAOp::Xvi16ger2spp, MMAHandlerOp::FirstArgIsResult>),`。
- **L467 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},`.
  **L467 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},`。
- **L468 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L468 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L469 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mma_xvi4ger8_",`.
  **L469 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mma_xvi4ger8_",`。
- **L470 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L470 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L471 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genMmaIntr<MMAOp::Xvi4ger8, MMAHandlerOp::SubToFunc>),`.
  **L471 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genMmaIntr<MMAOp::Xvi4ger8, MMAHandlerOp::SubToFunc>),`。
- **L472 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},`.
  **L472 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},`。
- **L473 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L473 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L474 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mma_xvi4ger8pp",`.
  **L474 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mma_xvi4ger8pp",`。
- **L475 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L475 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L476 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genMmaIntr<MMAOp::Xvi4ger8pp, MMAHandlerOp::FirstArgIsResult>),`.
  **L476 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genMmaIntr<MMAOp::Xvi4ger8pp, MMAHandlerOp::FirstArgIsResult>),`。
- **L477 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},`.
  **L477 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},`。
- **L478 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L478 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L479 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mma_xvi8ger4_",`.
  **L479 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mma_xvi8ger4_",`。
- **L480 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L480 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。

### Lines 481-504

````cpp
         &PI::genMmaIntr<MMAOp::Xvi8ger4, MMAHandlerOp::SubToFunc>),
     {{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},
     /*isElemental=*/true},
    {"__ppc_mma_xvi8ger4pp",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genMmaIntr<MMAOp::Xvi8ger4pp, MMAHandlerOp::FirstArgIsResult>),
     {{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},
     /*isElemental=*/true},
    {"__ppc_mma_xvi8ger4spp",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genMmaIntr<MMAOp::Xvi8ger4spp, MMAHandlerOp::FirstArgIsResult>),
     {{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},
     /*isElemental=*/true},
    {"__ppc_mma_xxmfacc",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genMmaIntr<MMAOp::Xxmfacc, MMAHandlerOp::FirstArgIsResult>),
     {{{"acc", asAddr}}},
     /*isElemental=*/true},
    {"__ppc_mma_xxmtacc",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genMmaIntr<MMAOp::Xxmtacc, MMAHandlerOp::FirstArgIsResult>),
     {{{"acc", asAddr}}},
     /*isElemental=*/true},
    {"__ppc_mma_xxsetaccz",
````
- **L481 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genMmaIntr<MMAOp::Xvi8ger4, MMAHandlerOp::SubToFunc>),`.
  **L481 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genMmaIntr<MMAOp::Xvi8ger4, MMAHandlerOp::SubToFunc>),`。
- **L482 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},`.
  **L482 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},`。
- **L483 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L483 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L484 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mma_xvi8ger4pp",`.
  **L484 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mma_xvi8ger4pp",`。
- **L485 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L485 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L486 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genMmaIntr<MMAOp::Xvi8ger4pp, MMAHandlerOp::FirstArgIsResult>),`.
  **L486 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genMmaIntr<MMAOp::Xvi8ger4pp, MMAHandlerOp::FirstArgIsResult>),`。
- **L487 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},`.
  **L487 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},`。
- **L488 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L488 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L489 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mma_xvi8ger4spp",`.
  **L489 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mma_xvi8ger4spp",`。
- **L490 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L490 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L491 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genMmaIntr<MMAOp::Xvi8ger4spp, MMAHandlerOp::FirstArgIsResult>),`.
  **L491 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genMmaIntr<MMAOp::Xvi8ger4spp, MMAHandlerOp::FirstArgIsResult>),`。
- **L492 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},`.
  **L492 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"acc", asAddr}, {"a", asValue}, {"b", asValue}}},`。
- **L493 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L493 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L494 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mma_xxmfacc",`.
  **L494 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mma_xxmfacc",`。
- **L495 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L495 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L496 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genMmaIntr<MMAOp::Xxmfacc, MMAHandlerOp::FirstArgIsResult>),`.
  **L496 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genMmaIntr<MMAOp::Xxmfacc, MMAHandlerOp::FirstArgIsResult>),`。
- **L497 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"acc", asAddr}}},`.
  **L497 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"acc", asAddr}}},`。
- **L498 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L498 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L499 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mma_xxmtacc",`.
  **L499 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mma_xxmtacc",`。
- **L500 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L500 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L501 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genMmaIntr<MMAOp::Xxmtacc, MMAHandlerOp::FirstArgIsResult>),`.
  **L501 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genMmaIntr<MMAOp::Xxmtacc, MMAHandlerOp::FirstArgIsResult>),`。
- **L502 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"acc", asAddr}}},`.
  **L502 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"acc", asAddr}}},`。
- **L503 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L503 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L504 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mma_xxsetaccz",`.
  **L504 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mma_xxsetaccz",`。

### Lines 505-528

````cpp
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genMmaIntr<MMAOp::Xxsetaccz, MMAHandlerOp::SubToFunc>),
     {{{"acc", asAddr}}},
     /*isElemental=*/true},
    {"__ppc_mtfsf",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(&PI::genMtfsf<false>),
     {{{"mask", asValue}, {"r", asValue}}},
     /*isElemental=*/false},
    {"__ppc_mtfsfi",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(&PI::genMtfsf<true>),
     {{{"bf", asValue}, {"i", asValue}}},
     /*isElemental=*/false},
    {"__ppc_vec_abs",
     static_cast<IntrinsicLibrary::ExtendedGenerator>(&PI::genVecAbs),
     {{{"arg1", asValue}}},
     /*isElemental=*/true},
    {"__ppc_vec_add",
     static_cast<IntrinsicLibrary::ExtendedGenerator>(
         &PI::genVecAddAndMulSubXor<VecOp::Add>),
     {{{"arg1", asValue}, {"arg2", asValue}}},
     /*isElemental=*/true},
    {"__ppc_vec_and",
     static_cast<IntrinsicLibrary::ExtendedGenerator>(
         &PI::genVecAddAndMulSubXor<VecOp::And>),
````
- **L505 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L505 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L506 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genMmaIntr<MMAOp::Xxsetaccz, MMAHandlerOp::SubToFunc>),`.
  **L506 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genMmaIntr<MMAOp::Xxsetaccz, MMAHandlerOp::SubToFunc>),`。
- **L507 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"acc", asAddr}}},`.
  **L507 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"acc", asAddr}}},`。
- **L508 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L508 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L509 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mtfsf",`.
  **L509 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mtfsf",`。
- **L510 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static_cast<IntrinsicLibrary::SubroutineGenerator>(&PI::genMtfsf<false>),`.
  **L510 CN**: 继续一个多行参数列表、初始化器或聚合项：`static_cast<IntrinsicLibrary::SubroutineGenerator>(&PI::genMtfsf<false>),`。
- **L511 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"mask", asValue}, {"r", asValue}}},`.
  **L511 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"mask", asValue}, {"r", asValue}}},`。
- **L512 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/false},`.
  **L512 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/false},`。
- **L513 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_mtfsfi",`.
  **L513 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_mtfsfi",`。
- **L514 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static_cast<IntrinsicLibrary::SubroutineGenerator>(&PI::genMtfsf<true>),`.
  **L514 CN**: 继续一个多行参数列表、初始化器或聚合项：`static_cast<IntrinsicLibrary::SubroutineGenerator>(&PI::genMtfsf<true>),`。
- **L515 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"bf", asValue}, {"i", asValue}}},`.
  **L515 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"bf", asValue}, {"i", asValue}}},`。
- **L516 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/false},`.
  **L516 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/false},`。
- **L517 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_abs",`.
  **L517 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_abs",`。
- **L518 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static_cast<IntrinsicLibrary::ExtendedGenerator>(&PI::genVecAbs),`.
  **L518 CN**: 继续一个多行参数列表、初始化器或聚合项：`static_cast<IntrinsicLibrary::ExtendedGenerator>(&PI::genVecAbs),`。
- **L519 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"arg1", asValue}}},`.
  **L519 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"arg1", asValue}}},`。
- **L520 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L520 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L521 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_add",`.
  **L521 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_add",`。
- **L522 EN**: Continues logic associated with callable symbol `ExtendedGenerator>`.
  **L522 CN**: 继续与可调用符号 `ExtendedGenerator>` 相关的逻辑。
- **L523 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genVecAddAndMulSubXor<VecOp::Add>),`.
  **L523 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genVecAddAndMulSubXor<VecOp::Add>),`。
- **L524 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"arg1", asValue}, {"arg2", asValue}}},`.
  **L524 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"arg1", asValue}, {"arg2", asValue}}},`。
- **L525 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L525 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L526 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_and",`.
  **L526 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_and",`。
- **L527 EN**: Continues logic associated with callable symbol `ExtendedGenerator>`.
  **L527 CN**: 继续与可调用符号 `ExtendedGenerator>` 相关的逻辑。
- **L528 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genVecAddAndMulSubXor<VecOp::And>),`.
  **L528 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genVecAddAndMulSubXor<VecOp::And>),`。

### Lines 529-552

````cpp
     {{{"arg1", asValue}, {"arg2", asValue}}},
     /*isElemental=*/true},
    {"__ppc_vec_any_ge",
     static_cast<IntrinsicLibrary::ExtendedGenerator>(
         &PI::genVecAnyCompare<VecOp::Anyge>),
     {{{"arg1", asValue}, {"arg2", asValue}}},
     /*isElemental=*/true},
    {"__ppc_vec_cmpge",
     static_cast<IntrinsicLibrary::ExtendedGenerator>(
         &PI::genVecCmp<VecOp::Cmpge>),
     {{{"arg1", asValue}, {"arg2", asValue}}},
     /*isElemental=*/true},
    {"__ppc_vec_cmpgt",
     static_cast<IntrinsicLibrary::ExtendedGenerator>(
         &PI::genVecCmp<VecOp::Cmpgt>),
     {{{"arg1", asValue}, {"arg2", asValue}}},
     /*isElemental=*/true},
    {"__ppc_vec_cmple",
     static_cast<IntrinsicLibrary::ExtendedGenerator>(
         &PI::genVecCmp<VecOp::Cmple>),
     {{{"arg1", asValue}, {"arg2", asValue}}},
     /*isElemental=*/true},
    {"__ppc_vec_cmplt",
     static_cast<IntrinsicLibrary::ExtendedGenerator>(
````
- **L529 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"arg1", asValue}, {"arg2", asValue}}},`.
  **L529 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"arg1", asValue}, {"arg2", asValue}}},`。
- **L530 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L530 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L531 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_any_ge",`.
  **L531 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_any_ge",`。
- **L532 EN**: Continues logic associated with callable symbol `ExtendedGenerator>`.
  **L532 CN**: 继续与可调用符号 `ExtendedGenerator>` 相关的逻辑。
- **L533 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genVecAnyCompare<VecOp::Anyge>),`.
  **L533 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genVecAnyCompare<VecOp::Anyge>),`。
- **L534 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"arg1", asValue}, {"arg2", asValue}}},`.
  **L534 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"arg1", asValue}, {"arg2", asValue}}},`。
- **L535 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L535 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L536 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_cmpge",`.
  **L536 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_cmpge",`。
- **L537 EN**: Continues logic associated with callable symbol `ExtendedGenerator>`.
  **L537 CN**: 继续与可调用符号 `ExtendedGenerator>` 相关的逻辑。
- **L538 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genVecCmp<VecOp::Cmpge>),`.
  **L538 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genVecCmp<VecOp::Cmpge>),`。
- **L539 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"arg1", asValue}, {"arg2", asValue}}},`.
  **L539 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"arg1", asValue}, {"arg2", asValue}}},`。
- **L540 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L540 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L541 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_cmpgt",`.
  **L541 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_cmpgt",`。
- **L542 EN**: Continues logic associated with callable symbol `ExtendedGenerator>`.
  **L542 CN**: 继续与可调用符号 `ExtendedGenerator>` 相关的逻辑。
- **L543 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genVecCmp<VecOp::Cmpgt>),`.
  **L543 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genVecCmp<VecOp::Cmpgt>),`。
- **L544 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"arg1", asValue}, {"arg2", asValue}}},`.
  **L544 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"arg1", asValue}, {"arg2", asValue}}},`。
- **L545 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L545 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L546 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_cmple",`.
  **L546 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_cmple",`。
- **L547 EN**: Continues logic associated with callable symbol `ExtendedGenerator>`.
  **L547 CN**: 继续与可调用符号 `ExtendedGenerator>` 相关的逻辑。
- **L548 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genVecCmp<VecOp::Cmple>),`.
  **L548 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genVecCmp<VecOp::Cmple>),`。
- **L549 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"arg1", asValue}, {"arg2", asValue}}},`.
  **L549 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"arg1", asValue}, {"arg2", asValue}}},`。
- **L550 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L550 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L551 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_cmplt",`.
  **L551 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_cmplt",`。
- **L552 EN**: Continues logic associated with callable symbol `ExtendedGenerator>`.
  **L552 CN**: 继续与可调用符号 `ExtendedGenerator>` 相关的逻辑。

### Lines 553-576

````cpp
         &PI::genVecCmp<VecOp::Cmplt>),
     {{{"arg1", asValue}, {"arg2", asValue}}},
     /*isElemental=*/true},
    {"__ppc_vec_convert",
     static_cast<IntrinsicLibrary::ExtendedGenerator>(
         &PI::genVecConvert<VecOp::Convert>),
     {{{"v", asValue}, {"mold", asValue}}},
     /*isElemental=*/false},
    {"__ppc_vec_ctf",
     static_cast<IntrinsicLibrary::ExtendedGenerator>(
         &PI::genVecConvert<VecOp::Ctf>),
     {{{"arg1", asValue}, {"arg2", asValue}}},
     /*isElemental=*/true},
    {"__ppc_vec_cvf",
     static_cast<IntrinsicLibrary::ExtendedGenerator>(
         &PI::genVecConvert<VecOp::Cvf>),
     {{{"arg1", asValue}}},
     /*isElemental=*/true},
    {"__ppc_vec_extract",
     static_cast<IntrinsicLibrary::ExtendedGenerator>(&PI::genVecExtract),
     {{{"arg1", asValue}, {"arg2", asValue}}},
     /*isElemental=*/true},
    {"__ppc_vec_insert",
     static_cast<IntrinsicLibrary::ExtendedGenerator>(&PI::genVecInsert),
````
- **L553 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genVecCmp<VecOp::Cmplt>),`.
  **L553 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genVecCmp<VecOp::Cmplt>),`。
- **L554 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"arg1", asValue}, {"arg2", asValue}}},`.
  **L554 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"arg1", asValue}, {"arg2", asValue}}},`。
- **L555 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L555 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L556 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_convert",`.
  **L556 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_convert",`。
- **L557 EN**: Continues logic associated with callable symbol `ExtendedGenerator>`.
  **L557 CN**: 继续与可调用符号 `ExtendedGenerator>` 相关的逻辑。
- **L558 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genVecConvert<VecOp::Convert>),`.
  **L558 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genVecConvert<VecOp::Convert>),`。
- **L559 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"v", asValue}, {"mold", asValue}}},`.
  **L559 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"v", asValue}, {"mold", asValue}}},`。
- **L560 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/false},`.
  **L560 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/false},`。
- **L561 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_ctf",`.
  **L561 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_ctf",`。
- **L562 EN**: Continues logic associated with callable symbol `ExtendedGenerator>`.
  **L562 CN**: 继续与可调用符号 `ExtendedGenerator>` 相关的逻辑。
- **L563 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genVecConvert<VecOp::Ctf>),`.
  **L563 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genVecConvert<VecOp::Ctf>),`。
- **L564 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"arg1", asValue}, {"arg2", asValue}}},`.
  **L564 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"arg1", asValue}, {"arg2", asValue}}},`。
- **L565 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L565 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L566 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_cvf",`.
  **L566 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_cvf",`。
- **L567 EN**: Continues logic associated with callable symbol `ExtendedGenerator>`.
  **L567 CN**: 继续与可调用符号 `ExtendedGenerator>` 相关的逻辑。
- **L568 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genVecConvert<VecOp::Cvf>),`.
  **L568 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genVecConvert<VecOp::Cvf>),`。
- **L569 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"arg1", asValue}}},`.
  **L569 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"arg1", asValue}}},`。
- **L570 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L570 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L571 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_extract",`.
  **L571 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_extract",`。
- **L572 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static_cast<IntrinsicLibrary::ExtendedGenerator>(&PI::genVecExtract),`.
  **L572 CN**: 继续一个多行参数列表、初始化器或聚合项：`static_cast<IntrinsicLibrary::ExtendedGenerator>(&PI::genVecExtract),`。
- **L573 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"arg1", asValue}, {"arg2", asValue}}},`.
  **L573 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"arg1", asValue}, {"arg2", asValue}}},`。
- **L574 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L574 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L575 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_insert",`.
  **L575 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_insert",`。
- **L576 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static_cast<IntrinsicLibrary::ExtendedGenerator>(&PI::genVecInsert),`.
  **L576 CN**: 继续一个多行参数列表、初始化器或聚合项：`static_cast<IntrinsicLibrary::ExtendedGenerator>(&PI::genVecInsert),`。

### Lines 577-600

````cpp
     {{{"arg1", asValue}, {"arg2", asValue}, {"arg3", asValue}}},
     /*isElemental=*/true},
    {"__ppc_vec_ld",
     static_cast<IntrinsicLibrary::ExtendedGenerator>(
         &PI::genVecLdCallGrp<VecOp::Ld>),
     {{{"arg1", asValue}, {"arg2", asAddr}}},
     /*isElemental=*/false},
    {"__ppc_vec_lde",
     static_cast<IntrinsicLibrary::ExtendedGenerator>(
         &PI::genVecLdCallGrp<VecOp::Lde>),
     {{{"arg1", asValue}, {"arg2", asAddr}}},
     /*isElemental=*/false},
    {"__ppc_vec_ldl",
     static_cast<IntrinsicLibrary::ExtendedGenerator>(
         &PI::genVecLdCallGrp<VecOp::Ldl>),
     {{{"arg1", asValue}, {"arg2", asAddr}}},
     /*isElemental=*/false},
    {"__ppc_vec_lvsl",
     static_cast<IntrinsicLibrary::ExtendedGenerator>(
         &PI::genVecLvsGrp<VecOp::Lvsl>),
     {{{"arg1", asValue}, {"arg2", asAddr}}},
     /*isElemental=*/false},
    {"__ppc_vec_lvsr",
     static_cast<IntrinsicLibrary::ExtendedGenerator>(
````
- **L577 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"arg1", asValue}, {"arg2", asValue}, {"arg3", asValue}}},`.
  **L577 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"arg1", asValue}, {"arg2", asValue}, {"arg3", asValue}}},`。
- **L578 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L578 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L579 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_ld",`.
  **L579 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_ld",`。
- **L580 EN**: Continues logic associated with callable symbol `ExtendedGenerator>`.
  **L580 CN**: 继续与可调用符号 `ExtendedGenerator>` 相关的逻辑。
- **L581 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genVecLdCallGrp<VecOp::Ld>),`.
  **L581 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genVecLdCallGrp<VecOp::Ld>),`。
- **L582 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"arg1", asValue}, {"arg2", asAddr}}},`.
  **L582 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"arg1", asValue}, {"arg2", asAddr}}},`。
- **L583 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/false},`.
  **L583 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/false},`。
- **L584 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_lde",`.
  **L584 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_lde",`。
- **L585 EN**: Continues logic associated with callable symbol `ExtendedGenerator>`.
  **L585 CN**: 继续与可调用符号 `ExtendedGenerator>` 相关的逻辑。
- **L586 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genVecLdCallGrp<VecOp::Lde>),`.
  **L586 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genVecLdCallGrp<VecOp::Lde>),`。
- **L587 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"arg1", asValue}, {"arg2", asAddr}}},`.
  **L587 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"arg1", asValue}, {"arg2", asAddr}}},`。
- **L588 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/false},`.
  **L588 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/false},`。
- **L589 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_ldl",`.
  **L589 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_ldl",`。
- **L590 EN**: Continues logic associated with callable symbol `ExtendedGenerator>`.
  **L590 CN**: 继续与可调用符号 `ExtendedGenerator>` 相关的逻辑。
- **L591 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genVecLdCallGrp<VecOp::Ldl>),`.
  **L591 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genVecLdCallGrp<VecOp::Ldl>),`。
- **L592 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"arg1", asValue}, {"arg2", asAddr}}},`.
  **L592 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"arg1", asValue}, {"arg2", asAddr}}},`。
- **L593 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/false},`.
  **L593 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/false},`。
- **L594 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_lvsl",`.
  **L594 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_lvsl",`。
- **L595 EN**: Continues logic associated with callable symbol `ExtendedGenerator>`.
  **L595 CN**: 继续与可调用符号 `ExtendedGenerator>` 相关的逻辑。
- **L596 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genVecLvsGrp<VecOp::Lvsl>),`.
  **L596 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genVecLvsGrp<VecOp::Lvsl>),`。
- **L597 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"arg1", asValue}, {"arg2", asAddr}}},`.
  **L597 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"arg1", asValue}, {"arg2", asAddr}}},`。
- **L598 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/false},`.
  **L598 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/false},`。
- **L599 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_lvsr",`.
  **L599 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_lvsr",`。
- **L600 EN**: Continues logic associated with callable symbol `ExtendedGenerator>`.
  **L600 CN**: 继续与可调用符号 `ExtendedGenerator>` 相关的逻辑。

### Lines 601-624

````cpp
         &PI::genVecLvsGrp<VecOp::Lvsr>),
     {{{"arg1", asValue}, {"arg2", asAddr}}},
     /*isElemental=*/false},
    {"__ppc_vec_lxv",
     static_cast<IntrinsicLibrary::ExtendedGenerator>(
         &PI::genVecLdNoCallGrp<VecOp::Lxv>),
     {{{"arg1", asValue}, {"arg2", asAddr}}},
     /*isElemental=*/false},
    {"__ppc_vec_lxvp",
     static_cast<IntrinsicLibrary::ExtendedGenerator>(
         &PI::genVecLdCallGrp<VecOp::Lxvp>),
     {{{"arg1", asValue}, {"arg2", asAddr}}},
     /*isElemental=*/false},
    {"__ppc_vec_mergeh",
     static_cast<IntrinsicLibrary::ExtendedGenerator>(
         &PI::genVecMerge<VecOp::Mergeh>),
     {{{"arg1", asValue}, {"arg2", asValue}}},
     /*isElemental=*/true},
    {"__ppc_vec_mergel",
     static_cast<IntrinsicLibrary::ExtendedGenerator>(
         &PI::genVecMerge<VecOp::Mergel>),
     {{{"arg1", asValue}, {"arg2", asValue}}},
     /*isElemental=*/true},
    {"__ppc_vec_msub",
````
- **L601 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genVecLvsGrp<VecOp::Lvsr>),`.
  **L601 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genVecLvsGrp<VecOp::Lvsr>),`。
- **L602 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"arg1", asValue}, {"arg2", asAddr}}},`.
  **L602 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"arg1", asValue}, {"arg2", asAddr}}},`。
- **L603 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/false},`.
  **L603 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/false},`。
- **L604 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_lxv",`.
  **L604 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_lxv",`。
- **L605 EN**: Continues logic associated with callable symbol `ExtendedGenerator>`.
  **L605 CN**: 继续与可调用符号 `ExtendedGenerator>` 相关的逻辑。
- **L606 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genVecLdNoCallGrp<VecOp::Lxv>),`.
  **L606 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genVecLdNoCallGrp<VecOp::Lxv>),`。
- **L607 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"arg1", asValue}, {"arg2", asAddr}}},`.
  **L607 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"arg1", asValue}, {"arg2", asAddr}}},`。
- **L608 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/false},`.
  **L608 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/false},`。
- **L609 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_lxvp",`.
  **L609 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_lxvp",`。
- **L610 EN**: Continues logic associated with callable symbol `ExtendedGenerator>`.
  **L610 CN**: 继续与可调用符号 `ExtendedGenerator>` 相关的逻辑。
- **L611 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genVecLdCallGrp<VecOp::Lxvp>),`.
  **L611 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genVecLdCallGrp<VecOp::Lxvp>),`。
- **L612 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"arg1", asValue}, {"arg2", asAddr}}},`.
  **L612 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"arg1", asValue}, {"arg2", asAddr}}},`。
- **L613 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/false},`.
  **L613 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/false},`。
- **L614 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_mergeh",`.
  **L614 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_mergeh",`。
- **L615 EN**: Continues logic associated with callable symbol `ExtendedGenerator>`.
  **L615 CN**: 继续与可调用符号 `ExtendedGenerator>` 相关的逻辑。
- **L616 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genVecMerge<VecOp::Mergeh>),`.
  **L616 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genVecMerge<VecOp::Mergeh>),`。
- **L617 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"arg1", asValue}, {"arg2", asValue}}},`.
  **L617 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"arg1", asValue}, {"arg2", asValue}}},`。
- **L618 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L618 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L619 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_mergel",`.
  **L619 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_mergel",`。
- **L620 EN**: Continues logic associated with callable symbol `ExtendedGenerator>`.
  **L620 CN**: 继续与可调用符号 `ExtendedGenerator>` 相关的逻辑。
- **L621 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genVecMerge<VecOp::Mergel>),`.
  **L621 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genVecMerge<VecOp::Mergel>),`。
- **L622 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"arg1", asValue}, {"arg2", asValue}}},`.
  **L622 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"arg1", asValue}, {"arg2", asValue}}},`。
- **L623 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L623 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L624 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_msub",`.
  **L624 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_msub",`。

### Lines 625-648

````cpp
     static_cast<IntrinsicLibrary::ExtendedGenerator>(
         &PI::genVecNmaddMsub<VecOp::Msub>),
     {{{"arg1", asValue}, {"arg2", asValue}, {"arg3", asValue}}},
     /*isElemental=*/true},
    {"__ppc_vec_mul",
     static_cast<IntrinsicLibrary::ExtendedGenerator>(
         &PI::genVecAddAndMulSubXor<VecOp::Mul>),
     {{{"arg1", asValue}, {"arg2", asValue}}},
     /*isElemental=*/true},
    {"__ppc_vec_nmadd",
     static_cast<IntrinsicLibrary::ExtendedGenerator>(
         &PI::genVecNmaddMsub<VecOp::Nmadd>),
     {{{"arg1", asValue}, {"arg2", asValue}, {"arg3", asValue}}},
     /*isElemental=*/true},
    {"__ppc_vec_perm",
     static_cast<IntrinsicLibrary::ExtendedGenerator>(
         &PI::genVecPerm<VecOp::Perm>),
     {{{"arg1", asValue}, {"arg2", asValue}, {"arg3", asValue}}},
     /*isElemental=*/true},
    {"__ppc_vec_permi",
     static_cast<IntrinsicLibrary::ExtendedGenerator>(
         &PI::genVecPerm<VecOp::Permi>),
     {{{"arg1", asValue}, {"arg2", asValue}, {"arg3", asValue}}},
     /*isElemental=*/true},
````
- **L625 EN**: Continues logic associated with callable symbol `ExtendedGenerator>`.
  **L625 CN**: 继续与可调用符号 `ExtendedGenerator>` 相关的逻辑。
- **L626 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genVecNmaddMsub<VecOp::Msub>),`.
  **L626 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genVecNmaddMsub<VecOp::Msub>),`。
- **L627 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"arg1", asValue}, {"arg2", asValue}, {"arg3", asValue}}},`.
  **L627 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"arg1", asValue}, {"arg2", asValue}, {"arg3", asValue}}},`。
- **L628 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L628 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L629 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_mul",`.
  **L629 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_mul",`。
- **L630 EN**: Continues logic associated with callable symbol `ExtendedGenerator>`.
  **L630 CN**: 继续与可调用符号 `ExtendedGenerator>` 相关的逻辑。
- **L631 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genVecAddAndMulSubXor<VecOp::Mul>),`.
  **L631 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genVecAddAndMulSubXor<VecOp::Mul>),`。
- **L632 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"arg1", asValue}, {"arg2", asValue}}},`.
  **L632 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"arg1", asValue}, {"arg2", asValue}}},`。
- **L633 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L633 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L634 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_nmadd",`.
  **L634 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_nmadd",`。
- **L635 EN**: Continues logic associated with callable symbol `ExtendedGenerator>`.
  **L635 CN**: 继续与可调用符号 `ExtendedGenerator>` 相关的逻辑。
- **L636 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genVecNmaddMsub<VecOp::Nmadd>),`.
  **L636 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genVecNmaddMsub<VecOp::Nmadd>),`。
- **L637 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"arg1", asValue}, {"arg2", asValue}, {"arg3", asValue}}},`.
  **L637 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"arg1", asValue}, {"arg2", asValue}, {"arg3", asValue}}},`。
- **L638 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L638 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L639 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_perm",`.
  **L639 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_perm",`。
- **L640 EN**: Continues logic associated with callable symbol `ExtendedGenerator>`.
  **L640 CN**: 继续与可调用符号 `ExtendedGenerator>` 相关的逻辑。
- **L641 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genVecPerm<VecOp::Perm>),`.
  **L641 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genVecPerm<VecOp::Perm>),`。
- **L642 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"arg1", asValue}, {"arg2", asValue}, {"arg3", asValue}}},`.
  **L642 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"arg1", asValue}, {"arg2", asValue}, {"arg3", asValue}}},`。
- **L643 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L643 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L644 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_permi",`.
  **L644 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_permi",`。
- **L645 EN**: Continues logic associated with callable symbol `ExtendedGenerator>`.
  **L645 CN**: 继续与可调用符号 `ExtendedGenerator>` 相关的逻辑。
- **L646 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genVecPerm<VecOp::Permi>),`.
  **L646 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genVecPerm<VecOp::Permi>),`。
- **L647 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"arg1", asValue}, {"arg2", asValue}, {"arg3", asValue}}},`.
  **L647 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"arg1", asValue}, {"arg2", asValue}, {"arg3", asValue}}},`。
- **L648 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L648 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。

### Lines 649-672

````cpp
    {"__ppc_vec_sel",
     static_cast<IntrinsicLibrary::ExtendedGenerator>(&PI::genVecSel),
     {{{"arg1", asValue}, {"arg2", asValue}, {"arg3", asValue}}},
     /*isElemental=*/true},
    {"__ppc_vec_sl",
     static_cast<IntrinsicLibrary::ExtendedGenerator>(
         &PI::genVecShift<VecOp::Sl>),
     {{{"arg1", asValue}, {"arg2", asValue}}},
     /*isElemental=*/true},
    {"__ppc_vec_sld",
     static_cast<IntrinsicLibrary::ExtendedGenerator>(
         &PI::genVecShift<VecOp::Sld>),
     {{{"arg1", asValue}, {"arg2", asValue}, {"arg3", asValue}}},
     /*isElemental=*/true},
    {"__ppc_vec_sldw",
     static_cast<IntrinsicLibrary::ExtendedGenerator>(
         &PI::genVecShift<VecOp::Sldw>),
     {{{"arg1", asValue}, {"arg2", asValue}, {"arg3", asValue}}},
     /*isElemental=*/true},
    {"__ppc_vec_sll",
     static_cast<IntrinsicLibrary::ExtendedGenerator>(
         &PI::genVecShift<VecOp::Sll>),
     {{{"arg1", asValue}, {"arg2", asValue}}},
     /*isElemental=*/true},
````
- **L649 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_sel",`.
  **L649 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_sel",`。
- **L650 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static_cast<IntrinsicLibrary::ExtendedGenerator>(&PI::genVecSel),`.
  **L650 CN**: 继续一个多行参数列表、初始化器或聚合项：`static_cast<IntrinsicLibrary::ExtendedGenerator>(&PI::genVecSel),`。
- **L651 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"arg1", asValue}, {"arg2", asValue}, {"arg3", asValue}}},`.
  **L651 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"arg1", asValue}, {"arg2", asValue}, {"arg3", asValue}}},`。
- **L652 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L652 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L653 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_sl",`.
  **L653 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_sl",`。
- **L654 EN**: Continues logic associated with callable symbol `ExtendedGenerator>`.
  **L654 CN**: 继续与可调用符号 `ExtendedGenerator>` 相关的逻辑。
- **L655 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genVecShift<VecOp::Sl>),`.
  **L655 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genVecShift<VecOp::Sl>),`。
- **L656 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"arg1", asValue}, {"arg2", asValue}}},`.
  **L656 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"arg1", asValue}, {"arg2", asValue}}},`。
- **L657 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L657 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L658 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_sld",`.
  **L658 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_sld",`。
- **L659 EN**: Continues logic associated with callable symbol `ExtendedGenerator>`.
  **L659 CN**: 继续与可调用符号 `ExtendedGenerator>` 相关的逻辑。
- **L660 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genVecShift<VecOp::Sld>),`.
  **L660 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genVecShift<VecOp::Sld>),`。
- **L661 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"arg1", asValue}, {"arg2", asValue}, {"arg3", asValue}}},`.
  **L661 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"arg1", asValue}, {"arg2", asValue}, {"arg3", asValue}}},`。
- **L662 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L662 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L663 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_sldw",`.
  **L663 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_sldw",`。
- **L664 EN**: Continues logic associated with callable symbol `ExtendedGenerator>`.
  **L664 CN**: 继续与可调用符号 `ExtendedGenerator>` 相关的逻辑。
- **L665 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genVecShift<VecOp::Sldw>),`.
  **L665 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genVecShift<VecOp::Sldw>),`。
- **L666 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"arg1", asValue}, {"arg2", asValue}, {"arg3", asValue}}},`.
  **L666 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"arg1", asValue}, {"arg2", asValue}, {"arg3", asValue}}},`。
- **L667 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L667 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L668 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_sll",`.
  **L668 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_sll",`。
- **L669 EN**: Continues logic associated with callable symbol `ExtendedGenerator>`.
  **L669 CN**: 继续与可调用符号 `ExtendedGenerator>` 相关的逻辑。
- **L670 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genVecShift<VecOp::Sll>),`.
  **L670 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genVecShift<VecOp::Sll>),`。
- **L671 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"arg1", asValue}, {"arg2", asValue}}},`.
  **L671 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"arg1", asValue}, {"arg2", asValue}}},`。
- **L672 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L672 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。

### Lines 673-696

````cpp
    {"__ppc_vec_slo",
     static_cast<IntrinsicLibrary::ExtendedGenerator>(
         &PI::genVecShift<VecOp::Slo>),
     {{{"arg1", asValue}, {"arg2", asValue}}},
     /*isElemental=*/true},
    {"__ppc_vec_splat",
     static_cast<IntrinsicLibrary::ExtendedGenerator>(
         &PI::genVecSplat<VecOp::Splat>),
     {{{"arg1", asValue}, {"arg2", asValue}}},
     /*isElemental=*/true},
    {"__ppc_vec_splat_s32_",
     static_cast<IntrinsicLibrary::ExtendedGenerator>(
         &PI::genVecSplat<VecOp::Splat_s32>),
     {{{"arg1", asValue}}},
     /*isElemental=*/true},
    {"__ppc_vec_splats",
     static_cast<IntrinsicLibrary::ExtendedGenerator>(
         &PI::genVecSplat<VecOp::Splats>),
     {{{"arg1", asValue}}},
     /*isElemental=*/true},
    {"__ppc_vec_sr",
     static_cast<IntrinsicLibrary::ExtendedGenerator>(
         &PI::genVecShift<VecOp::Sr>),
     {{{"arg1", asValue}, {"arg2", asValue}}},
````
- **L673 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_slo",`.
  **L673 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_slo",`。
- **L674 EN**: Continues logic associated with callable symbol `ExtendedGenerator>`.
  **L674 CN**: 继续与可调用符号 `ExtendedGenerator>` 相关的逻辑。
- **L675 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genVecShift<VecOp::Slo>),`.
  **L675 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genVecShift<VecOp::Slo>),`。
- **L676 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"arg1", asValue}, {"arg2", asValue}}},`.
  **L676 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"arg1", asValue}, {"arg2", asValue}}},`。
- **L677 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L677 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L678 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_splat",`.
  **L678 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_splat",`。
- **L679 EN**: Continues logic associated with callable symbol `ExtendedGenerator>`.
  **L679 CN**: 继续与可调用符号 `ExtendedGenerator>` 相关的逻辑。
- **L680 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genVecSplat<VecOp::Splat>),`.
  **L680 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genVecSplat<VecOp::Splat>),`。
- **L681 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"arg1", asValue}, {"arg2", asValue}}},`.
  **L681 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"arg1", asValue}, {"arg2", asValue}}},`。
- **L682 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L682 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L683 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_splat_s32_",`.
  **L683 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_splat_s32_",`。
- **L684 EN**: Continues logic associated with callable symbol `ExtendedGenerator>`.
  **L684 CN**: 继续与可调用符号 `ExtendedGenerator>` 相关的逻辑。
- **L685 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genVecSplat<VecOp::Splat_s32>),`.
  **L685 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genVecSplat<VecOp::Splat_s32>),`。
- **L686 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"arg1", asValue}}},`.
  **L686 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"arg1", asValue}}},`。
- **L687 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L687 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L688 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_splats",`.
  **L688 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_splats",`。
- **L689 EN**: Continues logic associated with callable symbol `ExtendedGenerator>`.
  **L689 CN**: 继续与可调用符号 `ExtendedGenerator>` 相关的逻辑。
- **L690 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genVecSplat<VecOp::Splats>),`.
  **L690 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genVecSplat<VecOp::Splats>),`。
- **L691 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"arg1", asValue}}},`.
  **L691 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"arg1", asValue}}},`。
- **L692 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L692 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L693 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_sr",`.
  **L693 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_sr",`。
- **L694 EN**: Continues logic associated with callable symbol `ExtendedGenerator>`.
  **L694 CN**: 继续与可调用符号 `ExtendedGenerator>` 相关的逻辑。
- **L695 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genVecShift<VecOp::Sr>),`.
  **L695 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genVecShift<VecOp::Sr>),`。
- **L696 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"arg1", asValue}, {"arg2", asValue}}},`.
  **L696 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"arg1", asValue}, {"arg2", asValue}}},`。

### Lines 697-720

````cpp
     /*isElemental=*/true},
    {"__ppc_vec_srl",
     static_cast<IntrinsicLibrary::ExtendedGenerator>(
         &PI::genVecShift<VecOp::Srl>),
     {{{"arg1", asValue}, {"arg2", asValue}}},
     /*isElemental=*/true},
    {"__ppc_vec_sro",
     static_cast<IntrinsicLibrary::ExtendedGenerator>(
         &PI::genVecShift<VecOp::Sro>),
     {{{"arg1", asValue}, {"arg2", asValue}}},
     /*isElemental=*/true},
    {"__ppc_vec_st",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genVecStore<VecOp::St>),
     {{{"arg1", asValue}, {"arg2", asValue}, {"arg3", asAddr}}},
     /*isElemental=*/false},
    {"__ppc_vec_ste",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genVecStore<VecOp::Ste>),
     {{{"arg1", asValue}, {"arg2", asValue}, {"arg3", asAddr}}},
     /*isElemental=*/false},
    {"__ppc_vec_stxv",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genVecXStore<VecOp::Stxv>),
````
- **L697 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L697 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L698 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_srl",`.
  **L698 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_srl",`。
- **L699 EN**: Continues logic associated with callable symbol `ExtendedGenerator>`.
  **L699 CN**: 继续与可调用符号 `ExtendedGenerator>` 相关的逻辑。
- **L700 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genVecShift<VecOp::Srl>),`.
  **L700 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genVecShift<VecOp::Srl>),`。
- **L701 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"arg1", asValue}, {"arg2", asValue}}},`.
  **L701 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"arg1", asValue}, {"arg2", asValue}}},`。
- **L702 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L702 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L703 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_sro",`.
  **L703 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_sro",`。
- **L704 EN**: Continues logic associated with callable symbol `ExtendedGenerator>`.
  **L704 CN**: 继续与可调用符号 `ExtendedGenerator>` 相关的逻辑。
- **L705 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genVecShift<VecOp::Sro>),`.
  **L705 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genVecShift<VecOp::Sro>),`。
- **L706 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"arg1", asValue}, {"arg2", asValue}}},`.
  **L706 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"arg1", asValue}, {"arg2", asValue}}},`。
- **L707 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L707 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L708 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_st",`.
  **L708 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_st",`。
- **L709 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L709 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L710 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genVecStore<VecOp::St>),`.
  **L710 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genVecStore<VecOp::St>),`。
- **L711 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"arg1", asValue}, {"arg2", asValue}, {"arg3", asAddr}}},`.
  **L711 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"arg1", asValue}, {"arg2", asValue}, {"arg3", asAddr}}},`。
- **L712 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/false},`.
  **L712 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/false},`。
- **L713 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_ste",`.
  **L713 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_ste",`。
- **L714 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L714 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L715 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genVecStore<VecOp::Ste>),`.
  **L715 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genVecStore<VecOp::Ste>),`。
- **L716 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"arg1", asValue}, {"arg2", asValue}, {"arg3", asAddr}}},`.
  **L716 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"arg1", asValue}, {"arg2", asValue}, {"arg3", asAddr}}},`。
- **L717 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/false},`.
  **L717 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/false},`。
- **L718 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_stxv",`.
  **L718 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_stxv",`。
- **L719 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L719 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L720 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genVecXStore<VecOp::Stxv>),`.
  **L720 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genVecXStore<VecOp::Stxv>),`。

### Lines 721-744

````cpp
     {{{"arg1", asValue}, {"arg2", asValue}, {"arg3", asAddr}}},
     /*isElemental=*/false},
    {"__ppc_vec_stxvp",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genVecStore<VecOp::Stxvp>),
     {{{"arg1", asValue}, {"arg2", asValue}, {"arg3", asAddr}}},
     /*isElemental=*/false},
    {"__ppc_vec_sub",
     static_cast<IntrinsicLibrary::ExtendedGenerator>(
         &PI::genVecAddAndMulSubXor<VecOp::Sub>),
     {{{"arg1", asValue}, {"arg2", asValue}}},
     /*isElemental=*/true},
    {"__ppc_vec_xl",
     static_cast<IntrinsicLibrary::ExtendedGenerator>(&PI::genVecXlGrp),
     {{{"arg1", asValue}, {"arg2", asAddr}}},
     /*isElemental=*/false},
    {"__ppc_vec_xl_be",
     static_cast<IntrinsicLibrary::ExtendedGenerator>(
         &PI::genVecLdNoCallGrp<VecOp::Xlbe>),
     {{{"arg1", asValue}, {"arg2", asAddr}}},
     /*isElemental=*/false},
    {"__ppc_vec_xld2_",
     static_cast<IntrinsicLibrary::ExtendedGenerator>(
         &PI::genVecLdCallGrp<VecOp::Xld2>),
````
- **L721 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"arg1", asValue}, {"arg2", asValue}, {"arg3", asAddr}}},`.
  **L721 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"arg1", asValue}, {"arg2", asValue}, {"arg3", asAddr}}},`。
- **L722 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/false},`.
  **L722 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/false},`。
- **L723 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_stxvp",`.
  **L723 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_stxvp",`。
- **L724 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L724 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L725 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genVecStore<VecOp::Stxvp>),`.
  **L725 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genVecStore<VecOp::Stxvp>),`。
- **L726 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"arg1", asValue}, {"arg2", asValue}, {"arg3", asAddr}}},`.
  **L726 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"arg1", asValue}, {"arg2", asValue}, {"arg3", asAddr}}},`。
- **L727 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/false},`.
  **L727 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/false},`。
- **L728 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_sub",`.
  **L728 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_sub",`。
- **L729 EN**: Continues logic associated with callable symbol `ExtendedGenerator>`.
  **L729 CN**: 继续与可调用符号 `ExtendedGenerator>` 相关的逻辑。
- **L730 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genVecAddAndMulSubXor<VecOp::Sub>),`.
  **L730 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genVecAddAndMulSubXor<VecOp::Sub>),`。
- **L731 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"arg1", asValue}, {"arg2", asValue}}},`.
  **L731 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"arg1", asValue}, {"arg2", asValue}}},`。
- **L732 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L732 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L733 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_xl",`.
  **L733 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_xl",`。
- **L734 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static_cast<IntrinsicLibrary::ExtendedGenerator>(&PI::genVecXlGrp),`.
  **L734 CN**: 继续一个多行参数列表、初始化器或聚合项：`static_cast<IntrinsicLibrary::ExtendedGenerator>(&PI::genVecXlGrp),`。
- **L735 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"arg1", asValue}, {"arg2", asAddr}}},`.
  **L735 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"arg1", asValue}, {"arg2", asAddr}}},`。
- **L736 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/false},`.
  **L736 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/false},`。
- **L737 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_xl_be",`.
  **L737 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_xl_be",`。
- **L738 EN**: Continues logic associated with callable symbol `ExtendedGenerator>`.
  **L738 CN**: 继续与可调用符号 `ExtendedGenerator>` 相关的逻辑。
- **L739 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genVecLdNoCallGrp<VecOp::Xlbe>),`.
  **L739 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genVecLdNoCallGrp<VecOp::Xlbe>),`。
- **L740 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"arg1", asValue}, {"arg2", asAddr}}},`.
  **L740 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"arg1", asValue}, {"arg2", asAddr}}},`。
- **L741 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/false},`.
  **L741 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/false},`。
- **L742 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_xld2_",`.
  **L742 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_xld2_",`。
- **L743 EN**: Continues logic associated with callable symbol `ExtendedGenerator>`.
  **L743 CN**: 继续与可调用符号 `ExtendedGenerator>` 相关的逻辑。
- **L744 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genVecLdCallGrp<VecOp::Xld2>),`.
  **L744 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genVecLdCallGrp<VecOp::Xld2>),`。

### Lines 745-768

````cpp
     {{{"arg1", asValue}, {"arg2", asAddr}}},
     /*isElemental=*/false},
    {"__ppc_vec_xlds",
     static_cast<IntrinsicLibrary::ExtendedGenerator>(&PI::genVecXlds),
     {{{"arg1", asValue}, {"arg2", asAddr}}},
     /*isElemental=*/false},
    {"__ppc_vec_xlw4_",
     static_cast<IntrinsicLibrary::ExtendedGenerator>(
         &PI::genVecLdCallGrp<VecOp::Xlw4>),
     {{{"arg1", asValue}, {"arg2", asAddr}}},
     /*isElemental=*/false},
    {"__ppc_vec_xor",
     static_cast<IntrinsicLibrary::ExtendedGenerator>(
         &PI::genVecAddAndMulSubXor<VecOp::Xor>),
     {{{"arg1", asValue}, {"arg2", asValue}}},
     /*isElemental=*/true},
    {"__ppc_vec_xst",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genVecXStore<VecOp::Xst>),
     {{{"arg1", asValue}, {"arg2", asValue}, {"arg3", asAddr}}},
     /*isElemental=*/false},
    {"__ppc_vec_xst_be",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genVecXStore<VecOp::Xst_be>),
````
- **L745 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"arg1", asValue}, {"arg2", asAddr}}},`.
  **L745 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"arg1", asValue}, {"arg2", asAddr}}},`。
- **L746 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/false},`.
  **L746 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/false},`。
- **L747 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_xlds",`.
  **L747 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_xlds",`。
- **L748 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static_cast<IntrinsicLibrary::ExtendedGenerator>(&PI::genVecXlds),`.
  **L748 CN**: 继续一个多行参数列表、初始化器或聚合项：`static_cast<IntrinsicLibrary::ExtendedGenerator>(&PI::genVecXlds),`。
- **L749 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"arg1", asValue}, {"arg2", asAddr}}},`.
  **L749 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"arg1", asValue}, {"arg2", asAddr}}},`。
- **L750 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/false},`.
  **L750 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/false},`。
- **L751 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_xlw4_",`.
  **L751 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_xlw4_",`。
- **L752 EN**: Continues logic associated with callable symbol `ExtendedGenerator>`.
  **L752 CN**: 继续与可调用符号 `ExtendedGenerator>` 相关的逻辑。
- **L753 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genVecLdCallGrp<VecOp::Xlw4>),`.
  **L753 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genVecLdCallGrp<VecOp::Xlw4>),`。
- **L754 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"arg1", asValue}, {"arg2", asAddr}}},`.
  **L754 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"arg1", asValue}, {"arg2", asAddr}}},`。
- **L755 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/false},`.
  **L755 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/false},`。
- **L756 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_xor",`.
  **L756 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_xor",`。
- **L757 EN**: Continues logic associated with callable symbol `ExtendedGenerator>`.
  **L757 CN**: 继续与可调用符号 `ExtendedGenerator>` 相关的逻辑。
- **L758 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genVecAddAndMulSubXor<VecOp::Xor>),`.
  **L758 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genVecAddAndMulSubXor<VecOp::Xor>),`。
- **L759 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"arg1", asValue}, {"arg2", asValue}}},`.
  **L759 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"arg1", asValue}, {"arg2", asValue}}},`。
- **L760 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/true},`.
  **L760 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/true},`。
- **L761 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_xst",`.
  **L761 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_xst",`。
- **L762 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L762 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L763 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genVecXStore<VecOp::Xst>),`.
  **L763 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genVecXStore<VecOp::Xst>),`。
- **L764 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"arg1", asValue}, {"arg2", asValue}, {"arg3", asAddr}}},`.
  **L764 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"arg1", asValue}, {"arg2", asValue}, {"arg3", asAddr}}},`。
- **L765 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/false},`.
  **L765 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/false},`。
- **L766 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_xst_be",`.
  **L766 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_xst_be",`。
- **L767 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L767 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L768 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genVecXStore<VecOp::Xst_be>),`.
  **L768 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genVecXStore<VecOp::Xst_be>),`。

### Lines 769-792

````cpp
     {{{"arg1", asValue}, {"arg2", asValue}, {"arg3", asAddr}}},
     /*isElemental=*/false},
    {"__ppc_vec_xstd2_",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genVecXStore<VecOp::Xstd2>),
     {{{"arg1", asValue}, {"arg2", asValue}, {"arg3", asAddr}}},
     /*isElemental=*/false},
    {"__ppc_vec_xstw4_",
     static_cast<IntrinsicLibrary::SubroutineGenerator>(
         &PI::genVecXStore<VecOp::Xstw4>),
     {{{"arg1", asValue}, {"arg2", asValue}, {"arg3", asAddr}}},
     /*isElemental=*/false},
};

static constexpr MathOperation ppcMathOperations[] = {
    // fcfi is just another name for fcfid, there is no llvm.ppc.fcfi.
    {"__ppc_fcfi", "llvm.ppc.fcfid", genFuncType<Ty::Real<8>, Ty::Real<8>>,
     genLibCall},
    {"__ppc_fcfid", "llvm.ppc.fcfid", genFuncType<Ty::Real<8>, Ty::Real<8>>,
     genLibCall},
    {"__ppc_fcfud", "llvm.ppc.fcfud", genFuncType<Ty::Real<8>, Ty::Real<8>>,
     genLibCall},
    {"__ppc_fctid", "llvm.ppc.fctid", genFuncType<Ty::Real<8>, Ty::Real<8>>,
     genLibCall},
````
- **L769 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"arg1", asValue}, {"arg2", asValue}, {"arg3", asAddr}}},`.
  **L769 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"arg1", asValue}, {"arg2", asValue}, {"arg3", asAddr}}},`。
- **L770 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/false},`.
  **L770 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/false},`。
- **L771 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_xstd2_",`.
  **L771 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_xstd2_",`。
- **L772 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L772 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L773 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genVecXStore<VecOp::Xstd2>),`.
  **L773 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genVecXStore<VecOp::Xstd2>),`。
- **L774 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"arg1", asValue}, {"arg2", asValue}, {"arg3", asAddr}}},`.
  **L774 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"arg1", asValue}, {"arg2", asValue}, {"arg3", asAddr}}},`。
- **L775 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/false},`.
  **L775 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/false},`。
- **L776 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_xstw4_",`.
  **L776 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_xstw4_",`。
- **L777 EN**: Continues logic associated with callable symbol `SubroutineGenerator>`.
  **L777 CN**: 继续与可调用符号 `SubroutineGenerator>` 相关的逻辑。
- **L778 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&PI::genVecXStore<VecOp::Xstw4>),`.
  **L778 CN**: 继续一个多行参数列表、初始化器或聚合项：`&PI::genVecXStore<VecOp::Xstw4>),`。
- **L779 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{{"arg1", asValue}, {"arg2", asValue}, {"arg3", asAddr}}},`.
  **L779 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{{"arg1", asValue}, {"arg2", asValue}, {"arg3", asAddr}}},`。
- **L780 EN**: Comment explains nearby logic, intent, or metadata: `isElemental=*/false},`.
  **L780 CN**: 注释说明附近代码的逻辑、意图或元数据：`isElemental=*/false},`。
- **L781 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L781 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L782 EN**: Blank line separating nearby declarations or logic blocks.
  **L782 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L783 EN**: Continues the surrounding expression or declaration: `static constexpr MathOperation ppcMathOperations[] = {`.
  **L783 CN**: 继续构造周围的表达式或声明：`static constexpr MathOperation ppcMathOperations[] = {`。
- **L784 EN**: Comment explains nearby logic, intent, or metadata: `fcfi is just another name for fcfid, there is no llvm.ppc.fcfi.`.
  **L784 CN**: 注释说明附近代码的逻辑、意图或元数据：`fcfi is just another name for fcfid, there is no llvm.ppc.fcfi.`。
- **L785 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_fcfi", "llvm.ppc.fcfid", genFuncType<Ty::Real<8>, Ty::Real<8>>,`.
  **L785 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_fcfi", "llvm.ppc.fcfid", genFuncType<Ty::Real<8>, Ty::Real<8>>,`。
- **L786 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genLibCall},`.
  **L786 CN**: 继续一个多行参数列表、初始化器或聚合项：`genLibCall},`。
- **L787 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_fcfid", "llvm.ppc.fcfid", genFuncType<Ty::Real<8>, Ty::Real<8>>,`.
  **L787 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_fcfid", "llvm.ppc.fcfid", genFuncType<Ty::Real<8>, Ty::Real<8>>,`。
- **L788 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genLibCall},`.
  **L788 CN**: 继续一个多行参数列表、初始化器或聚合项：`genLibCall},`。
- **L789 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_fcfud", "llvm.ppc.fcfud", genFuncType<Ty::Real<8>, Ty::Real<8>>,`.
  **L789 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_fcfud", "llvm.ppc.fcfud", genFuncType<Ty::Real<8>, Ty::Real<8>>,`。
- **L790 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genLibCall},`.
  **L790 CN**: 继续一个多行参数列表、初始化器或聚合项：`genLibCall},`。
- **L791 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_fctid", "llvm.ppc.fctid", genFuncType<Ty::Real<8>, Ty::Real<8>>,`.
  **L791 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_fctid", "llvm.ppc.fctid", genFuncType<Ty::Real<8>, Ty::Real<8>>,`。
- **L792 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genLibCall},`.
  **L792 CN**: 继续一个多行参数列表、初始化器或聚合项：`genLibCall},`。

### Lines 793-816

````cpp
    {"__ppc_fctidz", "llvm.ppc.fctidz", genFuncType<Ty::Real<8>, Ty::Real<8>>,
     genLibCall},
    {"__ppc_fctiw", "llvm.ppc.fctiw", genFuncType<Ty::Real<8>, Ty::Real<8>>,
     genLibCall},
    {"__ppc_fctiwz", "llvm.ppc.fctiwz", genFuncType<Ty::Real<8>, Ty::Real<8>>,
     genLibCall},
    {"__ppc_fctudz", "llvm.ppc.fctudz", genFuncType<Ty::Real<8>, Ty::Real<8>>,
     genLibCall},
    {"__ppc_fctuwz", "llvm.ppc.fctuwz", genFuncType<Ty::Real<8>, Ty::Real<8>>,
     genLibCall},
    {"__ppc_fmadd", "llvm.fma.f32",
     genFuncType<Ty::Real<4>, Ty::Real<4>, Ty::Real<4>, Ty::Real<4>>,
     genMathOp<mlir::math::FmaOp>},
    {"__ppc_fmadd", "llvm.fma.f64",
     genFuncType<Ty::Real<8>, Ty::Real<8>, Ty::Real<8>, Ty::Real<8>>,
     genMathOp<mlir::math::FmaOp>},
    {"__ppc_fmsub", "llvm.ppc.fmsubs",
     genFuncType<Ty::Real<4>, Ty::Real<4>, Ty::Real<4>, Ty::Real<4>>,
     genLibCall},
    {"__ppc_fmsub", "llvm.ppc.fmsub",
     genFuncType<Ty::Real<8>, Ty::Real<8>, Ty::Real<8>, Ty::Real<8>>,
     genLibCall},
    {"__ppc_fnabs", "llvm.ppc.fnabss", genFuncType<Ty::Real<4>, Ty::Real<4>>,
     genLibCall},
````
- **L793 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_fctidz", "llvm.ppc.fctidz", genFuncType<Ty::Real<8>, Ty::Real<8>>,`.
  **L793 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_fctidz", "llvm.ppc.fctidz", genFuncType<Ty::Real<8>, Ty::Real<8>>,`。
- **L794 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genLibCall},`.
  **L794 CN**: 继续一个多行参数列表、初始化器或聚合项：`genLibCall},`。
- **L795 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_fctiw", "llvm.ppc.fctiw", genFuncType<Ty::Real<8>, Ty::Real<8>>,`.
  **L795 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_fctiw", "llvm.ppc.fctiw", genFuncType<Ty::Real<8>, Ty::Real<8>>,`。
- **L796 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genLibCall},`.
  **L796 CN**: 继续一个多行参数列表、初始化器或聚合项：`genLibCall},`。
- **L797 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_fctiwz", "llvm.ppc.fctiwz", genFuncType<Ty::Real<8>, Ty::Real<8>>,`.
  **L797 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_fctiwz", "llvm.ppc.fctiwz", genFuncType<Ty::Real<8>, Ty::Real<8>>,`。
- **L798 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genLibCall},`.
  **L798 CN**: 继续一个多行参数列表、初始化器或聚合项：`genLibCall},`。
- **L799 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_fctudz", "llvm.ppc.fctudz", genFuncType<Ty::Real<8>, Ty::Real<8>>,`.
  **L799 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_fctudz", "llvm.ppc.fctudz", genFuncType<Ty::Real<8>, Ty::Real<8>>,`。
- **L800 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genLibCall},`.
  **L800 CN**: 继续一个多行参数列表、初始化器或聚合项：`genLibCall},`。
- **L801 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_fctuwz", "llvm.ppc.fctuwz", genFuncType<Ty::Real<8>, Ty::Real<8>>,`.
  **L801 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_fctuwz", "llvm.ppc.fctuwz", genFuncType<Ty::Real<8>, Ty::Real<8>>,`。
- **L802 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genLibCall},`.
  **L802 CN**: 继续一个多行参数列表、初始化器或聚合项：`genLibCall},`。
- **L803 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_fmadd", "llvm.fma.f32",`.
  **L803 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_fmadd", "llvm.fma.f32",`。
- **L804 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genFuncType<Ty::Real<4>, Ty::Real<4>, Ty::Real<4>, Ty::Real<4>>,`.
  **L804 CN**: 继续一个多行参数列表、初始化器或聚合项：`genFuncType<Ty::Real<4>, Ty::Real<4>, Ty::Real<4>, Ty::Real<4>>,`。
- **L805 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genMathOp<mlir::math::FmaOp>},`.
  **L805 CN**: 继续一个多行参数列表、初始化器或聚合项：`genMathOp<mlir::math::FmaOp>},`。
- **L806 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_fmadd", "llvm.fma.f64",`.
  **L806 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_fmadd", "llvm.fma.f64",`。
- **L807 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genFuncType<Ty::Real<8>, Ty::Real<8>, Ty::Real<8>, Ty::Real<8>>,`.
  **L807 CN**: 继续一个多行参数列表、初始化器或聚合项：`genFuncType<Ty::Real<8>, Ty::Real<8>, Ty::Real<8>, Ty::Real<8>>,`。
- **L808 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genMathOp<mlir::math::FmaOp>},`.
  **L808 CN**: 继续一个多行参数列表、初始化器或聚合项：`genMathOp<mlir::math::FmaOp>},`。
- **L809 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_fmsub", "llvm.ppc.fmsubs",`.
  **L809 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_fmsub", "llvm.ppc.fmsubs",`。
- **L810 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genFuncType<Ty::Real<4>, Ty::Real<4>, Ty::Real<4>, Ty::Real<4>>,`.
  **L810 CN**: 继续一个多行参数列表、初始化器或聚合项：`genFuncType<Ty::Real<4>, Ty::Real<4>, Ty::Real<4>, Ty::Real<4>>,`。
- **L811 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genLibCall},`.
  **L811 CN**: 继续一个多行参数列表、初始化器或聚合项：`genLibCall},`。
- **L812 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_fmsub", "llvm.ppc.fmsub",`.
  **L812 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_fmsub", "llvm.ppc.fmsub",`。
- **L813 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genFuncType<Ty::Real<8>, Ty::Real<8>, Ty::Real<8>, Ty::Real<8>>,`.
  **L813 CN**: 继续一个多行参数列表、初始化器或聚合项：`genFuncType<Ty::Real<8>, Ty::Real<8>, Ty::Real<8>, Ty::Real<8>>,`。
- **L814 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genLibCall},`.
  **L814 CN**: 继续一个多行参数列表、初始化器或聚合项：`genLibCall},`。
- **L815 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_fnabs", "llvm.ppc.fnabss", genFuncType<Ty::Real<4>, Ty::Real<4>>,`.
  **L815 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_fnabs", "llvm.ppc.fnabss", genFuncType<Ty::Real<4>, Ty::Real<4>>,`。
- **L816 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genLibCall},`.
  **L816 CN**: 继续一个多行参数列表、初始化器或聚合项：`genLibCall},`。

### Lines 817-840

````cpp
    {"__ppc_fnabs", "llvm.ppc.fnabs", genFuncType<Ty::Real<8>, Ty::Real<8>>,
     genLibCall},
    {"__ppc_fnmadd", "llvm.ppc.fnmadds",
     genFuncType<Ty::Real<4>, Ty::Real<4>, Ty::Real<4>, Ty::Real<4>>,
     genLibCall},
    {"__ppc_fnmadd", "llvm.ppc.fnmadd",
     genFuncType<Ty::Real<8>, Ty::Real<8>, Ty::Real<8>, Ty::Real<8>>,
     genLibCall},
    {"__ppc_fnmsub", "llvm.ppc.fnmsub.f32",
     genFuncType<Ty::Real<4>, Ty::Real<4>, Ty::Real<4>, Ty::Real<4>>,
     genLibCall},
    {"__ppc_fnmsub", "llvm.ppc.fnmsub.f64",
     genFuncType<Ty::Real<8>, Ty::Real<8>, Ty::Real<8>, Ty::Real<8>>,
     genLibCall},
    {"__ppc_fre", "llvm.ppc.fre", genFuncType<Ty::Real<8>, Ty::Real<8>>,
     genLibCall},
    {"__ppc_fres", "llvm.ppc.fres", genFuncType<Ty::Real<4>, Ty::Real<4>>,
     genLibCall},
    {"__ppc_frsqrte", "llvm.ppc.frsqrte", genFuncType<Ty::Real<8>, Ty::Real<8>>,
     genLibCall},
    {"__ppc_frsqrtes", "llvm.ppc.frsqrtes",
     genFuncType<Ty::Real<4>, Ty::Real<4>>, genLibCall},
    {"__ppc_vec_cvbf16spn", "llvm.ppc.vsx.xvcvbf16spn",
     genFuncType<Ty::UnsignedVector<1>, Ty::UnsignedVector<1>>, genLibCall},
````
- **L817 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_fnabs", "llvm.ppc.fnabs", genFuncType<Ty::Real<8>, Ty::Real<8>>,`.
  **L817 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_fnabs", "llvm.ppc.fnabs", genFuncType<Ty::Real<8>, Ty::Real<8>>,`。
- **L818 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genLibCall},`.
  **L818 CN**: 继续一个多行参数列表、初始化器或聚合项：`genLibCall},`。
- **L819 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_fnmadd", "llvm.ppc.fnmadds",`.
  **L819 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_fnmadd", "llvm.ppc.fnmadds",`。
- **L820 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genFuncType<Ty::Real<4>, Ty::Real<4>, Ty::Real<4>, Ty::Real<4>>,`.
  **L820 CN**: 继续一个多行参数列表、初始化器或聚合项：`genFuncType<Ty::Real<4>, Ty::Real<4>, Ty::Real<4>, Ty::Real<4>>,`。
- **L821 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genLibCall},`.
  **L821 CN**: 继续一个多行参数列表、初始化器或聚合项：`genLibCall},`。
- **L822 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_fnmadd", "llvm.ppc.fnmadd",`.
  **L822 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_fnmadd", "llvm.ppc.fnmadd",`。
- **L823 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genFuncType<Ty::Real<8>, Ty::Real<8>, Ty::Real<8>, Ty::Real<8>>,`.
  **L823 CN**: 继续一个多行参数列表、初始化器或聚合项：`genFuncType<Ty::Real<8>, Ty::Real<8>, Ty::Real<8>, Ty::Real<8>>,`。
- **L824 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genLibCall},`.
  **L824 CN**: 继续一个多行参数列表、初始化器或聚合项：`genLibCall},`。
- **L825 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_fnmsub", "llvm.ppc.fnmsub.f32",`.
  **L825 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_fnmsub", "llvm.ppc.fnmsub.f32",`。
- **L826 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genFuncType<Ty::Real<4>, Ty::Real<4>, Ty::Real<4>, Ty::Real<4>>,`.
  **L826 CN**: 继续一个多行参数列表、初始化器或聚合项：`genFuncType<Ty::Real<4>, Ty::Real<4>, Ty::Real<4>, Ty::Real<4>>,`。
- **L827 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genLibCall},`.
  **L827 CN**: 继续一个多行参数列表、初始化器或聚合项：`genLibCall},`。
- **L828 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_fnmsub", "llvm.ppc.fnmsub.f64",`.
  **L828 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_fnmsub", "llvm.ppc.fnmsub.f64",`。
- **L829 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genFuncType<Ty::Real<8>, Ty::Real<8>, Ty::Real<8>, Ty::Real<8>>,`.
  **L829 CN**: 继续一个多行参数列表、初始化器或聚合项：`genFuncType<Ty::Real<8>, Ty::Real<8>, Ty::Real<8>, Ty::Real<8>>,`。
- **L830 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genLibCall},`.
  **L830 CN**: 继续一个多行参数列表、初始化器或聚合项：`genLibCall},`。
- **L831 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_fre", "llvm.ppc.fre", genFuncType<Ty::Real<8>, Ty::Real<8>>,`.
  **L831 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_fre", "llvm.ppc.fre", genFuncType<Ty::Real<8>, Ty::Real<8>>,`。
- **L832 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genLibCall},`.
  **L832 CN**: 继续一个多行参数列表、初始化器或聚合项：`genLibCall},`。
- **L833 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_fres", "llvm.ppc.fres", genFuncType<Ty::Real<4>, Ty::Real<4>>,`.
  **L833 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_fres", "llvm.ppc.fres", genFuncType<Ty::Real<4>, Ty::Real<4>>,`。
- **L834 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genLibCall},`.
  **L834 CN**: 继续一个多行参数列表、初始化器或聚合项：`genLibCall},`。
- **L835 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_frsqrte", "llvm.ppc.frsqrte", genFuncType<Ty::Real<8>, Ty::Real<8>>,`.
  **L835 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_frsqrte", "llvm.ppc.frsqrte", genFuncType<Ty::Real<8>, Ty::Real<8>>,`。
- **L836 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genLibCall},`.
  **L836 CN**: 继续一个多行参数列表、初始化器或聚合项：`genLibCall},`。
- **L837 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_frsqrtes", "llvm.ppc.frsqrtes",`.
  **L837 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_frsqrtes", "llvm.ppc.frsqrtes",`。
- **L838 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genFuncType<Ty::Real<4>, Ty::Real<4>>, genLibCall},`.
  **L838 CN**: 继续一个多行参数列表、初始化器或聚合项：`genFuncType<Ty::Real<4>, Ty::Real<4>>, genLibCall},`。
- **L839 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_cvbf16spn", "llvm.ppc.vsx.xvcvbf16spn",`.
  **L839 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_cvbf16spn", "llvm.ppc.vsx.xvcvbf16spn",`。
- **L840 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genFuncType<Ty::UnsignedVector<1>, Ty::UnsignedVector<1>>, genLibCall},`.
  **L840 CN**: 继续一个多行参数列表、初始化器或聚合项：`genFuncType<Ty::UnsignedVector<1>, Ty::UnsignedVector<1>>, genLibCall},`。

### Lines 841-864

````cpp
    {"__ppc_vec_cvspbf16_", "llvm.ppc.vsx.xvcvspbf16",
     genFuncType<Ty::UnsignedVector<1>, Ty::UnsignedVector<1>>, genLibCall},
    {"__ppc_vec_madd", "llvm.fma.v4f32",
     genFuncType<Ty::RealVector<4>, Ty::RealVector<4>, Ty::RealVector<4>,
                 Ty::RealVector<4>>,
     genLibCall},
    {"__ppc_vec_madd", "llvm.fma.v2f64",
     genFuncType<Ty::RealVector<8>, Ty::RealVector<8>, Ty::RealVector<8>,
                 Ty::RealVector<8>>,
     genLibCall},
    {"__ppc_vec_max", "llvm.ppc.altivec.vmaxsb",
     genFuncType<Ty::IntegerVector<1>, Ty::IntegerVector<1>,
                 Ty::IntegerVector<1>>,
     genLibCall},
    {"__ppc_vec_max", "llvm.ppc.altivec.vmaxsh",
     genFuncType<Ty::IntegerVector<2>, Ty::IntegerVector<2>,
                 Ty::IntegerVector<2>>,
     genLibCall},
    {"__ppc_vec_max", "llvm.ppc.altivec.vmaxsw",
     genFuncType<Ty::IntegerVector<4>, Ty::IntegerVector<4>,
                 Ty::IntegerVector<4>>,
     genLibCall},
    {"__ppc_vec_max", "llvm.ppc.altivec.vmaxsd",
     genFuncType<Ty::IntegerVector<8>, Ty::IntegerVector<8>,
````
- **L841 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_cvspbf16_", "llvm.ppc.vsx.xvcvspbf16",`.
  **L841 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_cvspbf16_", "llvm.ppc.vsx.xvcvspbf16",`。
- **L842 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genFuncType<Ty::UnsignedVector<1>, Ty::UnsignedVector<1>>, genLibCall},`.
  **L842 CN**: 继续一个多行参数列表、初始化器或聚合项：`genFuncType<Ty::UnsignedVector<1>, Ty::UnsignedVector<1>>, genLibCall},`。
- **L843 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_madd", "llvm.fma.v4f32",`.
  **L843 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_madd", "llvm.fma.v4f32",`。
- **L844 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genFuncType<Ty::RealVector<4>, Ty::RealVector<4>, Ty::RealVector<4>,`.
  **L844 CN**: 继续一个多行参数列表、初始化器或聚合项：`genFuncType<Ty::RealVector<4>, Ty::RealVector<4>, Ty::RealVector<4>,`。
- **L845 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Ty::RealVector<4>>,`.
  **L845 CN**: 继续一个多行参数列表、初始化器或聚合项：`Ty::RealVector<4>>,`。
- **L846 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genLibCall},`.
  **L846 CN**: 继续一个多行参数列表、初始化器或聚合项：`genLibCall},`。
- **L847 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_madd", "llvm.fma.v2f64",`.
  **L847 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_madd", "llvm.fma.v2f64",`。
- **L848 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genFuncType<Ty::RealVector<8>, Ty::RealVector<8>, Ty::RealVector<8>,`.
  **L848 CN**: 继续一个多行参数列表、初始化器或聚合项：`genFuncType<Ty::RealVector<8>, Ty::RealVector<8>, Ty::RealVector<8>,`。
- **L849 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Ty::RealVector<8>>,`.
  **L849 CN**: 继续一个多行参数列表、初始化器或聚合项：`Ty::RealVector<8>>,`。
- **L850 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genLibCall},`.
  **L850 CN**: 继续一个多行参数列表、初始化器或聚合项：`genLibCall},`。
- **L851 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_max", "llvm.ppc.altivec.vmaxsb",`.
  **L851 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_max", "llvm.ppc.altivec.vmaxsb",`。
- **L852 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genFuncType<Ty::IntegerVector<1>, Ty::IntegerVector<1>,`.
  **L852 CN**: 继续一个多行参数列表、初始化器或聚合项：`genFuncType<Ty::IntegerVector<1>, Ty::IntegerVector<1>,`。
- **L853 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Ty::IntegerVector<1>>,`.
  **L853 CN**: 继续一个多行参数列表、初始化器或聚合项：`Ty::IntegerVector<1>>,`。
- **L854 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genLibCall},`.
  **L854 CN**: 继续一个多行参数列表、初始化器或聚合项：`genLibCall},`。
- **L855 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_max", "llvm.ppc.altivec.vmaxsh",`.
  **L855 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_max", "llvm.ppc.altivec.vmaxsh",`。
- **L856 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genFuncType<Ty::IntegerVector<2>, Ty::IntegerVector<2>,`.
  **L856 CN**: 继续一个多行参数列表、初始化器或聚合项：`genFuncType<Ty::IntegerVector<2>, Ty::IntegerVector<2>,`。
- **L857 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Ty::IntegerVector<2>>,`.
  **L857 CN**: 继续一个多行参数列表、初始化器或聚合项：`Ty::IntegerVector<2>>,`。
- **L858 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genLibCall},`.
  **L858 CN**: 继续一个多行参数列表、初始化器或聚合项：`genLibCall},`。
- **L859 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_max", "llvm.ppc.altivec.vmaxsw",`.
  **L859 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_max", "llvm.ppc.altivec.vmaxsw",`。
- **L860 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genFuncType<Ty::IntegerVector<4>, Ty::IntegerVector<4>,`.
  **L860 CN**: 继续一个多行参数列表、初始化器或聚合项：`genFuncType<Ty::IntegerVector<4>, Ty::IntegerVector<4>,`。
- **L861 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Ty::IntegerVector<4>>,`.
  **L861 CN**: 继续一个多行参数列表、初始化器或聚合项：`Ty::IntegerVector<4>>,`。
- **L862 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genLibCall},`.
  **L862 CN**: 继续一个多行参数列表、初始化器或聚合项：`genLibCall},`。
- **L863 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_max", "llvm.ppc.altivec.vmaxsd",`.
  **L863 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_max", "llvm.ppc.altivec.vmaxsd",`。
- **L864 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genFuncType<Ty::IntegerVector<8>, Ty::IntegerVector<8>,`.
  **L864 CN**: 继续一个多行参数列表、初始化器或聚合项：`genFuncType<Ty::IntegerVector<8>, Ty::IntegerVector<8>,`。

### Lines 865-888

````cpp
                 Ty::IntegerVector<8>>,
     genLibCall},
    {"__ppc_vec_max", "llvm.ppc.altivec.vmaxub",
     genFuncType<Ty::UnsignedVector<1>, Ty::UnsignedVector<1>,
                 Ty::UnsignedVector<1>>,
     genLibCall},
    {"__ppc_vec_max", "llvm.ppc.altivec.vmaxuh",
     genFuncType<Ty::UnsignedVector<2>, Ty::UnsignedVector<2>,
                 Ty::UnsignedVector<2>>,
     genLibCall},
    {"__ppc_vec_max", "llvm.ppc.altivec.vmaxuw",
     genFuncType<Ty::UnsignedVector<4>, Ty::UnsignedVector<4>,
                 Ty::UnsignedVector<4>>,
     genLibCall},
    {"__ppc_vec_max", "llvm.ppc.altivec.vmaxud",
     genFuncType<Ty::UnsignedVector<8>, Ty::UnsignedVector<8>,
                 Ty::UnsignedVector<8>>,
     genLibCall},
    {"__ppc_vec_max", "llvm.ppc.vsx.xvmaxsp",
     genFuncType<Ty::RealVector<4>, Ty::RealVector<4>, Ty::RealVector<4>>,
     genLibCall},
    {"__ppc_vec_max", "llvm.ppc.vsx.xvmaxdp",
     genFuncType<Ty::RealVector<8>, Ty::RealVector<8>, Ty::RealVector<8>>,
     genLibCall},
````
- **L865 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Ty::IntegerVector<8>>,`.
  **L865 CN**: 继续一个多行参数列表、初始化器或聚合项：`Ty::IntegerVector<8>>,`。
- **L866 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genLibCall},`.
  **L866 CN**: 继续一个多行参数列表、初始化器或聚合项：`genLibCall},`。
- **L867 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_max", "llvm.ppc.altivec.vmaxub",`.
  **L867 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_max", "llvm.ppc.altivec.vmaxub",`。
- **L868 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genFuncType<Ty::UnsignedVector<1>, Ty::UnsignedVector<1>,`.
  **L868 CN**: 继续一个多行参数列表、初始化器或聚合项：`genFuncType<Ty::UnsignedVector<1>, Ty::UnsignedVector<1>,`。
- **L869 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Ty::UnsignedVector<1>>,`.
  **L869 CN**: 继续一个多行参数列表、初始化器或聚合项：`Ty::UnsignedVector<1>>,`。
- **L870 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genLibCall},`.
  **L870 CN**: 继续一个多行参数列表、初始化器或聚合项：`genLibCall},`。
- **L871 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_max", "llvm.ppc.altivec.vmaxuh",`.
  **L871 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_max", "llvm.ppc.altivec.vmaxuh",`。
- **L872 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genFuncType<Ty::UnsignedVector<2>, Ty::UnsignedVector<2>,`.
  **L872 CN**: 继续一个多行参数列表、初始化器或聚合项：`genFuncType<Ty::UnsignedVector<2>, Ty::UnsignedVector<2>,`。
- **L873 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Ty::UnsignedVector<2>>,`.
  **L873 CN**: 继续一个多行参数列表、初始化器或聚合项：`Ty::UnsignedVector<2>>,`。
- **L874 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genLibCall},`.
  **L874 CN**: 继续一个多行参数列表、初始化器或聚合项：`genLibCall},`。
- **L875 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_max", "llvm.ppc.altivec.vmaxuw",`.
  **L875 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_max", "llvm.ppc.altivec.vmaxuw",`。
- **L876 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genFuncType<Ty::UnsignedVector<4>, Ty::UnsignedVector<4>,`.
  **L876 CN**: 继续一个多行参数列表、初始化器或聚合项：`genFuncType<Ty::UnsignedVector<4>, Ty::UnsignedVector<4>,`。
- **L877 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Ty::UnsignedVector<4>>,`.
  **L877 CN**: 继续一个多行参数列表、初始化器或聚合项：`Ty::UnsignedVector<4>>,`。
- **L878 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genLibCall},`.
  **L878 CN**: 继续一个多行参数列表、初始化器或聚合项：`genLibCall},`。
- **L879 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_max", "llvm.ppc.altivec.vmaxud",`.
  **L879 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_max", "llvm.ppc.altivec.vmaxud",`。
- **L880 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genFuncType<Ty::UnsignedVector<8>, Ty::UnsignedVector<8>,`.
  **L880 CN**: 继续一个多行参数列表、初始化器或聚合项：`genFuncType<Ty::UnsignedVector<8>, Ty::UnsignedVector<8>,`。
- **L881 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Ty::UnsignedVector<8>>,`.
  **L881 CN**: 继续一个多行参数列表、初始化器或聚合项：`Ty::UnsignedVector<8>>,`。
- **L882 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genLibCall},`.
  **L882 CN**: 继续一个多行参数列表、初始化器或聚合项：`genLibCall},`。
- **L883 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_max", "llvm.ppc.vsx.xvmaxsp",`.
  **L883 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_max", "llvm.ppc.vsx.xvmaxsp",`。
- **L884 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genFuncType<Ty::RealVector<4>, Ty::RealVector<4>, Ty::RealVector<4>>,`.
  **L884 CN**: 继续一个多行参数列表、初始化器或聚合项：`genFuncType<Ty::RealVector<4>, Ty::RealVector<4>, Ty::RealVector<4>>,`。
- **L885 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genLibCall},`.
  **L885 CN**: 继续一个多行参数列表、初始化器或聚合项：`genLibCall},`。
- **L886 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_max", "llvm.ppc.vsx.xvmaxdp",`.
  **L886 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_max", "llvm.ppc.vsx.xvmaxdp",`。
- **L887 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genFuncType<Ty::RealVector<8>, Ty::RealVector<8>, Ty::RealVector<8>>,`.
  **L887 CN**: 继续一个多行参数列表、初始化器或聚合项：`genFuncType<Ty::RealVector<8>, Ty::RealVector<8>, Ty::RealVector<8>>,`。
- **L888 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genLibCall},`.
  **L888 CN**: 继续一个多行参数列表、初始化器或聚合项：`genLibCall},`。

### Lines 889-912

````cpp
    {"__ppc_vec_min", "llvm.ppc.altivec.vminsb",
     genFuncType<Ty::IntegerVector<1>, Ty::IntegerVector<1>,
                 Ty::IntegerVector<1>>,
     genLibCall},
    {"__ppc_vec_min", "llvm.ppc.altivec.vminsh",
     genFuncType<Ty::IntegerVector<2>, Ty::IntegerVector<2>,
                 Ty::IntegerVector<2>>,
     genLibCall},
    {"__ppc_vec_min", "llvm.ppc.altivec.vminsw",
     genFuncType<Ty::IntegerVector<4>, Ty::IntegerVector<4>,
                 Ty::IntegerVector<4>>,
     genLibCall},
    {"__ppc_vec_min", "llvm.ppc.altivec.vminsd",
     genFuncType<Ty::IntegerVector<8>, Ty::IntegerVector<8>,
                 Ty::IntegerVector<8>>,
     genLibCall},
    {"__ppc_vec_min", "llvm.ppc.altivec.vminub",
     genFuncType<Ty::UnsignedVector<1>, Ty::UnsignedVector<1>,
                 Ty::UnsignedVector<1>>,
     genLibCall},
    {"__ppc_vec_min", "llvm.ppc.altivec.vminuh",
     genFuncType<Ty::UnsignedVector<2>, Ty::UnsignedVector<2>,
                 Ty::UnsignedVector<2>>,
     genLibCall},
````
- **L889 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_min", "llvm.ppc.altivec.vminsb",`.
  **L889 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_min", "llvm.ppc.altivec.vminsb",`。
- **L890 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genFuncType<Ty::IntegerVector<1>, Ty::IntegerVector<1>,`.
  **L890 CN**: 继续一个多行参数列表、初始化器或聚合项：`genFuncType<Ty::IntegerVector<1>, Ty::IntegerVector<1>,`。
- **L891 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Ty::IntegerVector<1>>,`.
  **L891 CN**: 继续一个多行参数列表、初始化器或聚合项：`Ty::IntegerVector<1>>,`。
- **L892 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genLibCall},`.
  **L892 CN**: 继续一个多行参数列表、初始化器或聚合项：`genLibCall},`。
- **L893 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_min", "llvm.ppc.altivec.vminsh",`.
  **L893 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_min", "llvm.ppc.altivec.vminsh",`。
- **L894 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genFuncType<Ty::IntegerVector<2>, Ty::IntegerVector<2>,`.
  **L894 CN**: 继续一个多行参数列表、初始化器或聚合项：`genFuncType<Ty::IntegerVector<2>, Ty::IntegerVector<2>,`。
- **L895 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Ty::IntegerVector<2>>,`.
  **L895 CN**: 继续一个多行参数列表、初始化器或聚合项：`Ty::IntegerVector<2>>,`。
- **L896 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genLibCall},`.
  **L896 CN**: 继续一个多行参数列表、初始化器或聚合项：`genLibCall},`。
- **L897 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_min", "llvm.ppc.altivec.vminsw",`.
  **L897 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_min", "llvm.ppc.altivec.vminsw",`。
- **L898 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genFuncType<Ty::IntegerVector<4>, Ty::IntegerVector<4>,`.
  **L898 CN**: 继续一个多行参数列表、初始化器或聚合项：`genFuncType<Ty::IntegerVector<4>, Ty::IntegerVector<4>,`。
- **L899 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Ty::IntegerVector<4>>,`.
  **L899 CN**: 继续一个多行参数列表、初始化器或聚合项：`Ty::IntegerVector<4>>,`。
- **L900 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genLibCall},`.
  **L900 CN**: 继续一个多行参数列表、初始化器或聚合项：`genLibCall},`。
- **L901 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_min", "llvm.ppc.altivec.vminsd",`.
  **L901 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_min", "llvm.ppc.altivec.vminsd",`。
- **L902 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genFuncType<Ty::IntegerVector<8>, Ty::IntegerVector<8>,`.
  **L902 CN**: 继续一个多行参数列表、初始化器或聚合项：`genFuncType<Ty::IntegerVector<8>, Ty::IntegerVector<8>,`。
- **L903 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Ty::IntegerVector<8>>,`.
  **L903 CN**: 继续一个多行参数列表、初始化器或聚合项：`Ty::IntegerVector<8>>,`。
- **L904 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genLibCall},`.
  **L904 CN**: 继续一个多行参数列表、初始化器或聚合项：`genLibCall},`。
- **L905 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_min", "llvm.ppc.altivec.vminub",`.
  **L905 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_min", "llvm.ppc.altivec.vminub",`。
- **L906 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genFuncType<Ty::UnsignedVector<1>, Ty::UnsignedVector<1>,`.
  **L906 CN**: 继续一个多行参数列表、初始化器或聚合项：`genFuncType<Ty::UnsignedVector<1>, Ty::UnsignedVector<1>,`。
- **L907 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Ty::UnsignedVector<1>>,`.
  **L907 CN**: 继续一个多行参数列表、初始化器或聚合项：`Ty::UnsignedVector<1>>,`。
- **L908 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genLibCall},`.
  **L908 CN**: 继续一个多行参数列表、初始化器或聚合项：`genLibCall},`。
- **L909 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_min", "llvm.ppc.altivec.vminuh",`.
  **L909 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_min", "llvm.ppc.altivec.vminuh",`。
- **L910 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genFuncType<Ty::UnsignedVector<2>, Ty::UnsignedVector<2>,`.
  **L910 CN**: 继续一个多行参数列表、初始化器或聚合项：`genFuncType<Ty::UnsignedVector<2>, Ty::UnsignedVector<2>,`。
- **L911 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Ty::UnsignedVector<2>>,`.
  **L911 CN**: 继续一个多行参数列表、初始化器或聚合项：`Ty::UnsignedVector<2>>,`。
- **L912 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genLibCall},`.
  **L912 CN**: 继续一个多行参数列表、初始化器或聚合项：`genLibCall},`。

### Lines 913-936

````cpp
    {"__ppc_vec_min", "llvm.ppc.altivec.vminuw",
     genFuncType<Ty::UnsignedVector<4>, Ty::UnsignedVector<4>,
                 Ty::UnsignedVector<4>>,
     genLibCall},
    {"__ppc_vec_min", "llvm.ppc.altivec.vminud",
     genFuncType<Ty::UnsignedVector<8>, Ty::UnsignedVector<8>,
                 Ty::UnsignedVector<8>>,
     genLibCall},
    {"__ppc_vec_min", "llvm.ppc.vsx.xvminsp",
     genFuncType<Ty::RealVector<4>, Ty::RealVector<4>, Ty::RealVector<4>>,
     genLibCall},
    {"__ppc_vec_min", "llvm.ppc.vsx.xvmindp",
     genFuncType<Ty::RealVector<8>, Ty::RealVector<8>, Ty::RealVector<8>>,
     genLibCall},
    {"__ppc_vec_nmsub", "llvm.ppc.fnmsub.v4f32",
     genFuncType<Ty::RealVector<4>, Ty::RealVector<4>, Ty::RealVector<4>,
                 Ty::RealVector<4>>,
     genLibCall},
    {"__ppc_vec_nmsub", "llvm.ppc.fnmsub.v2f64",
     genFuncType<Ty::RealVector<8>, Ty::RealVector<8>, Ty::RealVector<8>,
                 Ty::RealVector<8>>,
     genLibCall},
};

````
- **L913 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_min", "llvm.ppc.altivec.vminuw",`.
  **L913 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_min", "llvm.ppc.altivec.vminuw",`。
- **L914 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genFuncType<Ty::UnsignedVector<4>, Ty::UnsignedVector<4>,`.
  **L914 CN**: 继续一个多行参数列表、初始化器或聚合项：`genFuncType<Ty::UnsignedVector<4>, Ty::UnsignedVector<4>,`。
- **L915 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Ty::UnsignedVector<4>>,`.
  **L915 CN**: 继续一个多行参数列表、初始化器或聚合项：`Ty::UnsignedVector<4>>,`。
- **L916 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genLibCall},`.
  **L916 CN**: 继续一个多行参数列表、初始化器或聚合项：`genLibCall},`。
- **L917 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_min", "llvm.ppc.altivec.vminud",`.
  **L917 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_min", "llvm.ppc.altivec.vminud",`。
- **L918 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genFuncType<Ty::UnsignedVector<8>, Ty::UnsignedVector<8>,`.
  **L918 CN**: 继续一个多行参数列表、初始化器或聚合项：`genFuncType<Ty::UnsignedVector<8>, Ty::UnsignedVector<8>,`。
- **L919 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Ty::UnsignedVector<8>>,`.
  **L919 CN**: 继续一个多行参数列表、初始化器或聚合项：`Ty::UnsignedVector<8>>,`。
- **L920 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genLibCall},`.
  **L920 CN**: 继续一个多行参数列表、初始化器或聚合项：`genLibCall},`。
- **L921 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_min", "llvm.ppc.vsx.xvminsp",`.
  **L921 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_min", "llvm.ppc.vsx.xvminsp",`。
- **L922 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genFuncType<Ty::RealVector<4>, Ty::RealVector<4>, Ty::RealVector<4>>,`.
  **L922 CN**: 继续一个多行参数列表、初始化器或聚合项：`genFuncType<Ty::RealVector<4>, Ty::RealVector<4>, Ty::RealVector<4>>,`。
- **L923 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genLibCall},`.
  **L923 CN**: 继续一个多行参数列表、初始化器或聚合项：`genLibCall},`。
- **L924 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_min", "llvm.ppc.vsx.xvmindp",`.
  **L924 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_min", "llvm.ppc.vsx.xvmindp",`。
- **L925 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genFuncType<Ty::RealVector<8>, Ty::RealVector<8>, Ty::RealVector<8>>,`.
  **L925 CN**: 继续一个多行参数列表、初始化器或聚合项：`genFuncType<Ty::RealVector<8>, Ty::RealVector<8>, Ty::RealVector<8>>,`。
- **L926 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genLibCall},`.
  **L926 CN**: 继续一个多行参数列表、初始化器或聚合项：`genLibCall},`。
- **L927 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_nmsub", "llvm.ppc.fnmsub.v4f32",`.
  **L927 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_nmsub", "llvm.ppc.fnmsub.v4f32",`。
- **L928 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genFuncType<Ty::RealVector<4>, Ty::RealVector<4>, Ty::RealVector<4>,`.
  **L928 CN**: 继续一个多行参数列表、初始化器或聚合项：`genFuncType<Ty::RealVector<4>, Ty::RealVector<4>, Ty::RealVector<4>,`。
- **L929 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Ty::RealVector<4>>,`.
  **L929 CN**: 继续一个多行参数列表、初始化器或聚合项：`Ty::RealVector<4>>,`。
- **L930 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genLibCall},`.
  **L930 CN**: 继续一个多行参数列表、初始化器或聚合项：`genLibCall},`。
- **L931 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{"__ppc_vec_nmsub", "llvm.ppc.fnmsub.v2f64",`.
  **L931 CN**: 继续一个多行参数列表、初始化器或聚合项：`{"__ppc_vec_nmsub", "llvm.ppc.fnmsub.v2f64",`。
- **L932 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genFuncType<Ty::RealVector<8>, Ty::RealVector<8>, Ty::RealVector<8>,`.
  **L932 CN**: 继续一个多行参数列表、初始化器或聚合项：`genFuncType<Ty::RealVector<8>, Ty::RealVector<8>, Ty::RealVector<8>,`。
- **L933 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Ty::RealVector<8>>,`.
  **L933 CN**: 继续一个多行参数列表、初始化器或聚合项：`Ty::RealVector<8>>,`。
- **L934 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genLibCall},`.
  **L934 CN**: 继续一个多行参数列表、初始化器或聚合项：`genLibCall},`。
- **L935 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L935 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L936 EN**: Blank line separating nearby declarations or logic blocks.
  **L936 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 937-960

````cpp
const IntrinsicHandler *findPPCIntrinsicHandler(llvm::StringRef name) {
  auto compare = [](const IntrinsicHandler &ppcHandler, llvm::StringRef name) {
    return name.compare(ppcHandler.name) > 0;
  };
  auto result = llvm::lower_bound(ppcHandlers, name, compare);
  return result != std::end(ppcHandlers) && result->name == name ? result
                                                                 : nullptr;
}

using RtMap = Fortran::common::StaticMultimapView<MathOperation>;
static constexpr RtMap ppcMathOps(ppcMathOperations);
static_assert(ppcMathOps.Verify() && "map must be sorted");

std::pair<const MathOperation *, const MathOperation *>
checkPPCMathOperationsRange(llvm::StringRef name) {
  return ppcMathOps.equal_range(name);
}

// Helper functions for vector element ordering.
bool PPCIntrinsicLibrary::isBEVecElemOrderOnLE() {
  const auto triple{fir::getTargetTriple(builder.getModule())};
  return (triple.isLittleEndian() &&
          converter->getLoweringOptions().getNoPPCNativeVecElemOrder());
}
````
- **L937 EN**: Starts a function, method, lambda, or structured scope: `const IntrinsicHandler *findPPCIntrinsicHandler(llvm::StringRef name) {`.
  **L937 CN**: 开始一个函数、方法、lambda 或结构化作用域：`const IntrinsicHandler *findPPCIntrinsicHandler(llvm::StringRef name) {`。
- **L938 EN**: Starts a function, method, lambda, or structured scope: `auto compare = [](const IntrinsicHandler &ppcHandler, llvm::StringRef name) {`.
  **L938 CN**: 开始一个函数、方法、lambda 或结构化作用域：`auto compare = [](const IntrinsicHandler &ppcHandler, llvm::StringRef name) {`。
- **L939 EN**: Returns from the current function with `name.compare(ppcHandler.name) > 0`.
  **L939 CN**: 以 `name.compare(ppcHandler.name) > 0` 从当前函数返回。
- **L940 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L940 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L941 EN**: Initializes variable `result` from the right-hand expression.
  **L941 CN**: 使用右侧表达式初始化变量 `result`。
- **L942 EN**: Returns from the current function with `result != std::end(ppcHandlers) && result->name == name ? result`.
  **L942 CN**: 以 `result != std::end(ppcHandlers) && result->name == name ? result` 从当前函数返回。
- **L943 EN**: Executes a standalone statement or declaration: `: nullptr;`.
  **L943 CN**: 执行一条独立语句或声明：`: nullptr;`。
- **L944 EN**: Closes the current lexical scope or compound statement.
  **L944 CN**: 结束当前词法作用域或复合语句块。
- **L945 EN**: Blank line separating nearby declarations or logic blocks.
  **L945 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L946 EN**: Defines alias `RtMap` to simplify later code.
  **L946 CN**: 定义别名 `RtMap` 以简化后续代码。
- **L947 EN**: Executes a call or declaration centered on `ppcMathOps`.
  **L947 CN**: 执行以 `ppcMathOps` 为核心的调用或声明。
- **L948 EN**: Executes a call or declaration centered on `static_assert`.
  **L948 CN**: 执行以 `static_assert` 为核心的调用或声明。
- **L949 EN**: Blank line separating nearby declarations or logic blocks.
  **L949 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L950 EN**: Continues the surrounding expression or declaration: `std::pair<const MathOperation *, const MathOperation *>`.
  **L950 CN**: 继续构造周围的表达式或声明：`std::pair<const MathOperation *, const MathOperation *>`。
- **L951 EN**: Starts a function, method, lambda, or structured scope: `checkPPCMathOperationsRange(llvm::StringRef name) {`.
  **L951 CN**: 开始一个函数、方法、lambda 或结构化作用域：`checkPPCMathOperationsRange(llvm::StringRef name) {`。
- **L952 EN**: Returns from the current function with `ppcMathOps.equal_range(name)`.
  **L952 CN**: 以 `ppcMathOps.equal_range(name)` 从当前函数返回。
- **L953 EN**: Closes the current lexical scope or compound statement.
  **L953 CN**: 结束当前词法作用域或复合语句块。
- **L954 EN**: Blank line separating nearby declarations or logic blocks.
  **L954 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L955 EN**: Comment explains nearby logic, intent, or metadata: `Helper functions for vector element ordering.`.
  **L955 CN**: 注释说明附近代码的逻辑、意图或元数据：`Helper functions for vector element ordering.`。
- **L956 EN**: Starts a function, method, lambda, or structured scope: `bool PPCIntrinsicLibrary::isBEVecElemOrderOnLE() {`.
  **L956 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool PPCIntrinsicLibrary::isBEVecElemOrderOnLE() {`。
- **L957 EN**: Executes a call or declaration centered on `triple{fir::getTargetTriple`.
  **L957 CN**: 执行以 `triple{fir::getTargetTriple` 为核心的调用或声明。
- **L958 EN**: Returns from the current function with `(triple.isLittleEndian() &&`.
  **L958 CN**: 以 `(triple.isLittleEndian() &&` 从当前函数返回。
- **L959 EN**: Executes a call or declaration centered on `converter->getLoweringOptions`.
  **L959 CN**: 执行以 `converter->getLoweringOptions` 为核心的调用或声明。
- **L960 EN**: Closes the current lexical scope or compound statement.
  **L960 CN**: 结束当前词法作用域或复合语句块。

### Lines 961-984

````cpp
bool PPCIntrinsicLibrary::isNativeVecElemOrderOnLE() {
  const auto triple{fir::getTargetTriple(builder.getModule())};
  return (triple.isLittleEndian() &&
          !converter->getLoweringOptions().getNoPPCNativeVecElemOrder());
}
bool PPCIntrinsicLibrary::changeVecElemOrder() {
  const auto triple{fir::getTargetTriple(builder.getModule())};
  return (triple.isLittleEndian() !=
          converter->getLoweringOptions().getNoPPCNativeVecElemOrder());
}

static mlir::FunctionType genMmaVpFuncType(mlir::MLIRContext *context,
                                           int quadCnt, int pairCnt, int vecCnt,
                                           int intCnt = 0,
                                           int vecElemBitSize = 8,
                                           int intBitSize = 32) {
  // Constructs a function type with the following signature:
  // Result type: __vector_pair
  // Arguments:
  //   quadCnt: number of arguments that has __vector_quad type, followed by
  //   pairCnt: number of arguments that has __vector_pair type, followed by
  //   vecCnt: number of arguments that has vector(integer) type, followed by
  //   intCnt: number of arguments that has integer type
  //   vecElemBitSize: specifies the size of vector elements in bits
````
- **L961 EN**: Starts a function, method, lambda, or structured scope: `bool PPCIntrinsicLibrary::isNativeVecElemOrderOnLE() {`.
  **L961 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool PPCIntrinsicLibrary::isNativeVecElemOrderOnLE() {`。
- **L962 EN**: Executes a call or declaration centered on `triple{fir::getTargetTriple`.
  **L962 CN**: 执行以 `triple{fir::getTargetTriple` 为核心的调用或声明。
- **L963 EN**: Returns from the current function with `(triple.isLittleEndian() &&`.
  **L963 CN**: 以 `(triple.isLittleEndian() &&` 从当前函数返回。
- **L964 EN**: Executes a call or declaration centered on `!converter->getLoweringOptions`.
  **L964 CN**: 执行以 `!converter->getLoweringOptions` 为核心的调用或声明。
- **L965 EN**: Closes the current lexical scope or compound statement.
  **L965 CN**: 结束当前词法作用域或复合语句块。
- **L966 EN**: Starts a function, method, lambda, or structured scope: `bool PPCIntrinsicLibrary::changeVecElemOrder() {`.
  **L966 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool PPCIntrinsicLibrary::changeVecElemOrder() {`。
- **L967 EN**: Executes a call or declaration centered on `triple{fir::getTargetTriple`.
  **L967 CN**: 执行以 `triple{fir::getTargetTriple` 为核心的调用或声明。
- **L968 EN**: Returns from the current function with `(triple.isLittleEndian() !=`.
  **L968 CN**: 以 `(triple.isLittleEndian() !=` 从当前函数返回。
- **L969 EN**: Executes a call or declaration centered on `converter->getLoweringOptions`.
  **L969 CN**: 执行以 `converter->getLoweringOptions` 为核心的调用或声明。
- **L970 EN**: Closes the current lexical scope or compound statement.
  **L970 CN**: 结束当前词法作用域或复合语句块。
- **L971 EN**: Blank line separating nearby declarations or logic blocks.
  **L971 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L972 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static mlir::FunctionType genMmaVpFuncType(mlir::MLIRContext *context,`.
  **L972 CN**: 继续一个多行参数列表、初始化器或聚合项：`static mlir::FunctionType genMmaVpFuncType(mlir::MLIRContext *context,`。
- **L973 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `int quadCnt, int pairCnt, int vecCnt,`.
  **L973 CN**: 继续一个多行参数列表、初始化器或聚合项：`int quadCnt, int pairCnt, int vecCnt,`。
- **L974 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `int intCnt = 0,`.
  **L974 CN**: 继续一个多行参数列表、初始化器或聚合项：`int intCnt = 0,`。
- **L975 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `int vecElemBitSize = 8,`.
  **L975 CN**: 继续一个多行参数列表、初始化器或聚合项：`int vecElemBitSize = 8,`。
- **L976 EN**: Continues the surrounding expression or declaration: `int intBitSize = 32) {`.
  **L976 CN**: 继续构造周围的表达式或声明：`int intBitSize = 32) {`。
- **L977 EN**: Comment explains nearby logic, intent, or metadata: `Constructs a function type with the following signature:`.
  **L977 CN**: 注释说明附近代码的逻辑、意图或元数据：`Constructs a function type with the following signature:`。
- **L978 EN**: Comment explains nearby logic, intent, or metadata: `Result type: __vector_pair`.
  **L978 CN**: 注释说明附近代码的逻辑、意图或元数据：`Result type: __vector_pair`。
- **L979 EN**: Comment explains nearby logic, intent, or metadata: `Arguments:`.
  **L979 CN**: 注释说明附近代码的逻辑、意图或元数据：`Arguments:`。
- **L980 EN**: Comment explains nearby logic, intent, or metadata: `quadCnt: number of arguments that has __vector_quad type, followed by`.
  **L980 CN**: 注释说明附近代码的逻辑、意图或元数据：`quadCnt: number of arguments that has __vector_quad type, followed by`。
- **L981 EN**: Comment explains nearby logic, intent, or metadata: `pairCnt: number of arguments that has __vector_pair type, followed by`.
  **L981 CN**: 注释说明附近代码的逻辑、意图或元数据：`pairCnt: number of arguments that has __vector_pair type, followed by`。
- **L982 EN**: Comment explains nearby logic, intent, or metadata: `vecCnt: number of arguments that has vector(integer) type, followed by`.
  **L982 CN**: 注释说明附近代码的逻辑、意图或元数据：`vecCnt: number of arguments that has vector(integer) type, followed by`。
- **L983 EN**: Comment explains nearby logic, intent, or metadata: `intCnt: number of arguments that has integer type`.
  **L983 CN**: 注释说明附近代码的逻辑、意图或元数据：`intCnt: number of arguments that has integer type`。
- **L984 EN**: Comment explains nearby logic, intent, or metadata: `vecElemBitSize: specifies the size of vector elements in bits`.
  **L984 CN**: 注释说明附近代码的逻辑、意图或元数据：`vecElemBitSize: specifies the size of vector elements in bits`。

### Lines 985-1008

````cpp
  //   intBitSize: specifies the size of integer arguments in bits
  auto vType{mlir::VectorType::get(
      128 / vecElemBitSize, mlir::IntegerType::get(context, vecElemBitSize))};
  auto vpType{fir::VectorType::get(256, mlir::IntegerType::get(context, 1))};
  auto vqType{fir::VectorType::get(512, mlir::IntegerType::get(context, 1))};
  auto iType{mlir::IntegerType::get(context, intBitSize)};
  llvm::SmallVector<mlir::Type> argTypes;
  for (int i = 0; i < quadCnt; ++i) {
    argTypes.push_back(vqType);
  }
  for (int i = 0; i < pairCnt; ++i) {
    argTypes.push_back(vpType);
  }
  for (int i = 0; i < vecCnt; ++i) {
    argTypes.push_back(vType);
  }
  for (int i = 0; i < intCnt; ++i) {
    argTypes.push_back(iType);
  }

  return mlir::FunctionType::get(context, argTypes, {vpType});
}

static mlir::FunctionType genMmaVqFuncType(mlir::MLIRContext *context,
````
- **L985 EN**: Comment explains nearby logic, intent, or metadata: `intBitSize: specifies the size of integer arguments in bits`.
  **L985 CN**: 注释说明附近代码的逻辑、意图或元数据：`intBitSize: specifies the size of integer arguments in bits`。
- **L986 EN**: Continues logic associated with callable symbol `get`.
  **L986 CN**: 继续与可调用符号 `get` 相关的逻辑。
- **L987 EN**: Executes a call or declaration centered on `mlir::IntegerType::get`.
  **L987 CN**: 执行以 `mlir::IntegerType::get` 为核心的调用或声明。
- **L988 EN**: Executes a call or declaration centered on `vpType{fir::VectorType::get`.
  **L988 CN**: 执行以 `vpType{fir::VectorType::get` 为核心的调用或声明。
- **L989 EN**: Executes a call or declaration centered on `vqType{fir::VectorType::get`.
  **L989 CN**: 执行以 `vqType{fir::VectorType::get` 为核心的调用或声明。
- **L990 EN**: Executes a call or declaration centered on `iType{mlir::IntegerType::get`.
  **L990 CN**: 执行以 `iType{mlir::IntegerType::get` 为核心的调用或声明。
- **L991 EN**: Executes a standalone statement or declaration: `llvm::SmallVector<mlir::Type> argTypes;`.
  **L991 CN**: 执行一条独立语句或声明：`llvm::SmallVector<mlir::Type> argTypes;`。
- **L992 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L992 CN**: 开始 `for` 控制流语句并计算其条件。
- **L993 EN**: Executes a call or declaration centered on `argTypes.push_back`.
  **L993 CN**: 执行以 `argTypes.push_back` 为核心的调用或声明。
- **L994 EN**: Closes the current lexical scope or compound statement.
  **L994 CN**: 结束当前词法作用域或复合语句块。
- **L995 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L995 CN**: 开始 `for` 控制流语句并计算其条件。
- **L996 EN**: Executes a call or declaration centered on `argTypes.push_back`.
  **L996 CN**: 执行以 `argTypes.push_back` 为核心的调用或声明。
- **L997 EN**: Closes the current lexical scope or compound statement.
  **L997 CN**: 结束当前词法作用域或复合语句块。
- **L998 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L998 CN**: 开始 `for` 控制流语句并计算其条件。
- **L999 EN**: Executes a call or declaration centered on `argTypes.push_back`.
  **L999 CN**: 执行以 `argTypes.push_back` 为核心的调用或声明。
- **L1000 EN**: Closes the current lexical scope or compound statement.
  **L1000 CN**: 结束当前词法作用域或复合语句块。
- **L1001 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1001 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1002 EN**: Executes a call or declaration centered on `argTypes.push_back`.
  **L1002 CN**: 执行以 `argTypes.push_back` 为核心的调用或声明。
- **L1003 EN**: Closes the current lexical scope or compound statement.
  **L1003 CN**: 结束当前词法作用域或复合语句块。
- **L1004 EN**: Blank line separating nearby declarations or logic blocks.
  **L1004 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1005 EN**: Returns from the current function with `mlir::FunctionType::get(context, argTypes, {vpType})`.
  **L1005 CN**: 以 `mlir::FunctionType::get(context, argTypes, {vpType})` 从当前函数返回。
- **L1006 EN**: Closes the current lexical scope or compound statement.
  **L1006 CN**: 结束当前词法作用域或复合语句块。
- **L1007 EN**: Blank line separating nearby declarations or logic blocks.
  **L1007 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1008 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static mlir::FunctionType genMmaVqFuncType(mlir::MLIRContext *context,`.
  **L1008 CN**: 继续一个多行参数列表、初始化器或聚合项：`static mlir::FunctionType genMmaVqFuncType(mlir::MLIRContext *context,`。

### Lines 1009-1032

````cpp
                                           int quadCnt, int pairCnt, int vecCnt,
                                           int intCnt = 0,
                                           int vecElemBitSize = 8,
                                           int intBitSize = 32) {
  // Constructs a function type with the following signature:
  // Result type: __vector_quad
  // Arguments:
  //   quadCnt: number of arguments that has __vector_quad type, followed by
  //   pairCnt: number of arguments that has __vector_pair type, followed by
  //   vecCnt: number of arguments that has vector(integer) type, followed by
  //   intCnt: number of arguments that has integer type
  //   vecElemBitSize: specifies the size of vector elements in bits
  //   intBitSize: specifies the size of integer arguments in bits
  auto vType{mlir::VectorType::get(
      128 / vecElemBitSize, mlir::IntegerType::get(context, vecElemBitSize))};
  auto vpType{fir::VectorType::get(256, mlir::IntegerType::get(context, 1))};
  auto vqType{fir::VectorType::get(512, mlir::IntegerType::get(context, 1))};
  auto iType{mlir::IntegerType::get(context, intBitSize)};
  llvm::SmallVector<mlir::Type> argTypes;
  for (int i = 0; i < quadCnt; ++i) {
    argTypes.push_back(vqType);
  }
  for (int i = 0; i < pairCnt; ++i) {
    argTypes.push_back(vpType);
````
- **L1009 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `int quadCnt, int pairCnt, int vecCnt,`.
  **L1009 CN**: 继续一个多行参数列表、初始化器或聚合项：`int quadCnt, int pairCnt, int vecCnt,`。
- **L1010 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `int intCnt = 0,`.
  **L1010 CN**: 继续一个多行参数列表、初始化器或聚合项：`int intCnt = 0,`。
- **L1011 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `int vecElemBitSize = 8,`.
  **L1011 CN**: 继续一个多行参数列表、初始化器或聚合项：`int vecElemBitSize = 8,`。
- **L1012 EN**: Continues the surrounding expression or declaration: `int intBitSize = 32) {`.
  **L1012 CN**: 继续构造周围的表达式或声明：`int intBitSize = 32) {`。
- **L1013 EN**: Comment explains nearby logic, intent, or metadata: `Constructs a function type with the following signature:`.
  **L1013 CN**: 注释说明附近代码的逻辑、意图或元数据：`Constructs a function type with the following signature:`。
- **L1014 EN**: Comment explains nearby logic, intent, or metadata: `Result type: __vector_quad`.
  **L1014 CN**: 注释说明附近代码的逻辑、意图或元数据：`Result type: __vector_quad`。
- **L1015 EN**: Comment explains nearby logic, intent, or metadata: `Arguments:`.
  **L1015 CN**: 注释说明附近代码的逻辑、意图或元数据：`Arguments:`。
- **L1016 EN**: Comment explains nearby logic, intent, or metadata: `quadCnt: number of arguments that has __vector_quad type, followed by`.
  **L1016 CN**: 注释说明附近代码的逻辑、意图或元数据：`quadCnt: number of arguments that has __vector_quad type, followed by`。
- **L1017 EN**: Comment explains nearby logic, intent, or metadata: `pairCnt: number of arguments that has __vector_pair type, followed by`.
  **L1017 CN**: 注释说明附近代码的逻辑、意图或元数据：`pairCnt: number of arguments that has __vector_pair type, followed by`。
- **L1018 EN**: Comment explains nearby logic, intent, or metadata: `vecCnt: number of arguments that has vector(integer) type, followed by`.
  **L1018 CN**: 注释说明附近代码的逻辑、意图或元数据：`vecCnt: number of arguments that has vector(integer) type, followed by`。
- **L1019 EN**: Comment explains nearby logic, intent, or metadata: `intCnt: number of arguments that has integer type`.
  **L1019 CN**: 注释说明附近代码的逻辑、意图或元数据：`intCnt: number of arguments that has integer type`。
- **L1020 EN**: Comment explains nearby logic, intent, or metadata: `vecElemBitSize: specifies the size of vector elements in bits`.
  **L1020 CN**: 注释说明附近代码的逻辑、意图或元数据：`vecElemBitSize: specifies the size of vector elements in bits`。
- **L1021 EN**: Comment explains nearby logic, intent, or metadata: `intBitSize: specifies the size of integer arguments in bits`.
  **L1021 CN**: 注释说明附近代码的逻辑、意图或元数据：`intBitSize: specifies the size of integer arguments in bits`。
- **L1022 EN**: Continues logic associated with callable symbol `get`.
  **L1022 CN**: 继续与可调用符号 `get` 相关的逻辑。
- **L1023 EN**: Executes a call or declaration centered on `mlir::IntegerType::get`.
  **L1023 CN**: 执行以 `mlir::IntegerType::get` 为核心的调用或声明。
- **L1024 EN**: Executes a call or declaration centered on `vpType{fir::VectorType::get`.
  **L1024 CN**: 执行以 `vpType{fir::VectorType::get` 为核心的调用或声明。
- **L1025 EN**: Executes a call or declaration centered on `vqType{fir::VectorType::get`.
  **L1025 CN**: 执行以 `vqType{fir::VectorType::get` 为核心的调用或声明。
- **L1026 EN**: Executes a call or declaration centered on `iType{mlir::IntegerType::get`.
  **L1026 CN**: 执行以 `iType{mlir::IntegerType::get` 为核心的调用或声明。
- **L1027 EN**: Executes a standalone statement or declaration: `llvm::SmallVector<mlir::Type> argTypes;`.
  **L1027 CN**: 执行一条独立语句或声明：`llvm::SmallVector<mlir::Type> argTypes;`。
- **L1028 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1028 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1029 EN**: Executes a call or declaration centered on `argTypes.push_back`.
  **L1029 CN**: 执行以 `argTypes.push_back` 为核心的调用或声明。
- **L1030 EN**: Closes the current lexical scope or compound statement.
  **L1030 CN**: 结束当前词法作用域或复合语句块。
- **L1031 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1031 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1032 EN**: Executes a call or declaration centered on `argTypes.push_back`.
  **L1032 CN**: 执行以 `argTypes.push_back` 为核心的调用或声明。

### Lines 1033-1056

````cpp
  }
  for (int i = 0; i < vecCnt; ++i) {
    argTypes.push_back(vType);
  }
  for (int i = 0; i < intCnt; ++i) {
    argTypes.push_back(iType);
  }

  return mlir::FunctionType::get(context, argTypes, {vqType});
}

mlir::FunctionType genMmaDisassembleFuncType(mlir::MLIRContext *context,
                                             MMAOp mmaOp) {
  auto vType{mlir::VectorType::get(16, mlir::IntegerType::get(context, 8))};
  llvm::SmallVector<mlir::Type> members;

  if (mmaOp == MMAOp::DisassembleAcc) {
    auto vqType{fir::VectorType::get(512, mlir::IntegerType::get(context, 1))};
    members.push_back(vType);
    members.push_back(vType);
    members.push_back(vType);
    members.push_back(vType);
    auto resType{mlir::LLVM::LLVMStructType::getLiteral(context, members)};
    return mlir::FunctionType::get(context, {vqType}, {resType});
````
- **L1033 EN**: Closes the current lexical scope or compound statement.
  **L1033 CN**: 结束当前词法作用域或复合语句块。
- **L1034 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1034 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1035 EN**: Executes a call or declaration centered on `argTypes.push_back`.
  **L1035 CN**: 执行以 `argTypes.push_back` 为核心的调用或声明。
- **L1036 EN**: Closes the current lexical scope or compound statement.
  **L1036 CN**: 结束当前词法作用域或复合语句块。
- **L1037 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1037 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1038 EN**: Executes a call or declaration centered on `argTypes.push_back`.
  **L1038 CN**: 执行以 `argTypes.push_back` 为核心的调用或声明。
- **L1039 EN**: Closes the current lexical scope or compound statement.
  **L1039 CN**: 结束当前词法作用域或复合语句块。
- **L1040 EN**: Blank line separating nearby declarations or logic blocks.
  **L1040 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1041 EN**: Returns from the current function with `mlir::FunctionType::get(context, argTypes, {vqType})`.
  **L1041 CN**: 以 `mlir::FunctionType::get(context, argTypes, {vqType})` 从当前函数返回。
- **L1042 EN**: Closes the current lexical scope or compound statement.
  **L1042 CN**: 结束当前词法作用域或复合语句块。
- **L1043 EN**: Blank line separating nearby declarations or logic blocks.
  **L1043 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1044 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::FunctionType genMmaDisassembleFuncType(mlir::MLIRContext *context,`.
  **L1044 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::FunctionType genMmaDisassembleFuncType(mlir::MLIRContext *context,`。
- **L1045 EN**: Continues the surrounding expression or declaration: `MMAOp mmaOp) {`.
  **L1045 CN**: 继续构造周围的表达式或声明：`MMAOp mmaOp) {`。
- **L1046 EN**: Executes a call or declaration centered on `vType{mlir::VectorType::get`.
  **L1046 CN**: 执行以 `vType{mlir::VectorType::get` 为核心的调用或声明。
- **L1047 EN**: Executes a standalone statement or declaration: `llvm::SmallVector<mlir::Type> members;`.
  **L1047 CN**: 执行一条独立语句或声明：`llvm::SmallVector<mlir::Type> members;`。
- **L1048 EN**: Blank line separating nearby declarations or logic blocks.
  **L1048 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1049 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1049 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1050 EN**: Executes a call or declaration centered on `vqType{fir::VectorType::get`.
  **L1050 CN**: 执行以 `vqType{fir::VectorType::get` 为核心的调用或声明。
- **L1051 EN**: Executes a call or declaration centered on `members.push_back`.
  **L1051 CN**: 执行以 `members.push_back` 为核心的调用或声明。
- **L1052 EN**: Executes a call or declaration centered on `members.push_back`.
  **L1052 CN**: 执行以 `members.push_back` 为核心的调用或声明。
- **L1053 EN**: Executes a call or declaration centered on `members.push_back`.
  **L1053 CN**: 执行以 `members.push_back` 为核心的调用或声明。
- **L1054 EN**: Executes a call or declaration centered on `members.push_back`.
  **L1054 CN**: 执行以 `members.push_back` 为核心的调用或声明。
- **L1055 EN**: Executes a call or declaration centered on `resType{mlir::LLVM::LLVMStructType::getLiteral`.
  **L1055 CN**: 执行以 `resType{mlir::LLVM::LLVMStructType::getLiteral` 为核心的调用或声明。
- **L1056 EN**: Returns from the current function with `mlir::FunctionType::get(context, {vqType}, {resType})`.
  **L1056 CN**: 以 `mlir::FunctionType::get(context, {vqType}, {resType})` 从当前函数返回。

### Lines 1057-1080

````cpp
  } else if (mmaOp == MMAOp::DisassemblePair) {
    auto vpType{fir::VectorType::get(256, mlir::IntegerType::get(context, 1))};
    members.push_back(vType);
    members.push_back(vType);
    auto resType{mlir::LLVM::LLVMStructType::getLiteral(context, members)};
    return mlir::FunctionType::get(context, {vpType}, {resType});
  } else {
    llvm_unreachable(
        "Unsupported intrinsic code for function signature generator");
  }
}

//===----------------------------------------------------------------------===//
// PowerPC specific intrinsic handlers.
//===----------------------------------------------------------------------===//

// MTFSF, MTFSFI
template <bool isImm>
void PPCIntrinsicLibrary::genMtfsf(llvm::ArrayRef<fir::ExtendedValue> args) {
  assert(args.size() == 2);
  llvm::SmallVector<mlir::Value> scalarArgs;
  for (const fir::ExtendedValue &arg : args)
    if (arg.getUnboxed())
      scalarArgs.emplace_back(fir::getBase(arg));
````
- **L1057 EN**: Transitions from the previous branch into an `else if` condition.
  **L1057 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1058 EN**: Executes a call or declaration centered on `vpType{fir::VectorType::get`.
  **L1058 CN**: 执行以 `vpType{fir::VectorType::get` 为核心的调用或声明。
- **L1059 EN**: Executes a call or declaration centered on `members.push_back`.
  **L1059 CN**: 执行以 `members.push_back` 为核心的调用或声明。
- **L1060 EN**: Executes a call or declaration centered on `members.push_back`.
  **L1060 CN**: 执行以 `members.push_back` 为核心的调用或声明。
- **L1061 EN**: Executes a call or declaration centered on `resType{mlir::LLVM::LLVMStructType::getLiteral`.
  **L1061 CN**: 执行以 `resType{mlir::LLVM::LLVMStructType::getLiteral` 为核心的调用或声明。
- **L1062 EN**: Returns from the current function with `mlir::FunctionType::get(context, {vpType}, {resType})`.
  **L1062 CN**: 以 `mlir::FunctionType::get(context, {vpType}, {resType})` 从当前函数返回。
- **L1063 EN**: Transitions from the previous branch into the alternative path.
  **L1063 CN**: 从前一个分支过渡到备选路径。
- **L1064 EN**: Marks this control path as unreachable to LLVM.
  **L1064 CN**: 将该控制路径标记为 LLVM 认为不可达。
- **L1065 EN**: Executes a standalone statement or declaration: `"Unsupported intrinsic code for function signature generator");`.
  **L1065 CN**: 执行一条独立语句或声明：`"Unsupported intrinsic code for function signature generator");`。
- **L1066 EN**: Closes the current lexical scope or compound statement.
  **L1066 CN**: 结束当前词法作用域或复合语句块。
- **L1067 EN**: Closes the current lexical scope or compound statement.
  **L1067 CN**: 结束当前词法作用域或复合语句块。
- **L1068 EN**: Blank line separating nearby declarations or logic blocks.
  **L1068 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1069 EN**: Banner comment marking a file or section boundary.
  **L1069 CN**: 横幅注释，用于标记文件或章节边界。
- **L1070 EN**: Comment explains nearby logic, intent, or metadata: `PowerPC specific intrinsic handlers.`.
  **L1070 CN**: 注释说明附近代码的逻辑、意图或元数据：`PowerPC specific intrinsic handlers.`。
- **L1071 EN**: Banner comment marking a file or section boundary.
  **L1071 CN**: 横幅注释，用于标记文件或章节边界。
- **L1072 EN**: Blank line separating nearby declarations or logic blocks.
  **L1072 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1073 EN**: Comment explains nearby logic, intent, or metadata: `MTFSF, MTFSFI`.
  **L1073 CN**: 注释说明附近代码的逻辑、意图或元数据：`MTFSF, MTFSFI`。
- **L1074 EN**: Introduces template parameters or specialization context: `template <bool isImm>`.
  **L1074 CN**: 为后续声明引入模板参数或特化上下文：`template <bool isImm>`。
- **L1075 EN**: Starts a function, method, lambda, or structured scope: `void PPCIntrinsicLibrary::genMtfsf(llvm::ArrayRef<fir::ExtendedValue> args) {`.
  **L1075 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void PPCIntrinsicLibrary::genMtfsf(llvm::ArrayRef<fir::ExtendedValue> args) {`。
- **L1076 EN**: Checks an internal invariant in debug builds.
  **L1076 CN**: 在调试构建中检查内部不变式。
- **L1077 EN**: Executes a standalone statement or declaration: `llvm::SmallVector<mlir::Value> scalarArgs;`.
  **L1077 CN**: 执行一条独立语句或声明：`llvm::SmallVector<mlir::Value> scalarArgs;`。
- **L1078 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1078 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1079 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1079 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1080 EN**: Executes a call or declaration centered on `scalarArgs.emplace_back`.
  **L1080 CN**: 执行以 `scalarArgs.emplace_back` 为核心的调用或声明。

### Lines 1081-1104

````cpp
    else
      mlir::emitError(loc, "nonscalar intrinsic argument");

  mlir::FunctionType libFuncType;
  mlir::func::FuncOp funcOp;
  if (isImm) {
    libFuncType = genFuncType<Ty::Void, Ty::Integer<4>, Ty::Integer<4>>(
        builder.getContext(), builder);
    funcOp = builder.createFunction(loc, "llvm.ppc.mtfsfi", libFuncType);
  } else {
    libFuncType = genFuncType<Ty::Void, Ty::Integer<4>, Ty::Real<8>>(
        builder.getContext(), builder);
    funcOp = builder.createFunction(loc, "llvm.ppc.mtfsf", libFuncType);
  }
  fir::CallOp::create(builder, loc, funcOp, scalarArgs);
}

// VEC_ABS
fir::ExtendedValue
PPCIntrinsicLibrary::genVecAbs(mlir::Type resultType,
                               llvm::ArrayRef<fir::ExtendedValue> args) {
  assert(args.size() == 1);
  auto context{builder.getContext()};
  auto argBases{getBasesForArgs(args)};
````
- **L1081 EN**: Transitions from the previous branch into the alternative path.
  **L1081 CN**: 从前一个分支过渡到备选路径。
- **L1082 EN**: Executes a call or declaration centered on `mlir::emitError`.
  **L1082 CN**: 执行以 `mlir::emitError` 为核心的调用或声明。
- **L1083 EN**: Blank line separating nearby declarations or logic blocks.
  **L1083 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1084 EN**: Executes a standalone statement or declaration: `mlir::FunctionType libFuncType;`.
  **L1084 CN**: 执行一条独立语句或声明：`mlir::FunctionType libFuncType;`。
- **L1085 EN**: Executes a standalone statement or declaration: `mlir::func::FuncOp funcOp;`.
  **L1085 CN**: 执行一条独立语句或声明：`mlir::func::FuncOp funcOp;`。
- **L1086 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1086 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1087 EN**: Continues logic associated with callable symbol `Integer<4>>`.
  **L1087 CN**: 继续与可调用符号 `Integer<4>>` 相关的逻辑。
- **L1088 EN**: Executes a call or declaration centered on `builder.getContext`.
  **L1088 CN**: 执行以 `builder.getContext` 为核心的调用或声明。
- **L1089 EN**: Executes a call or declaration centered on `builder.createFunction`.
  **L1089 CN**: 执行以 `builder.createFunction` 为核心的调用或声明。
- **L1090 EN**: Transitions from the previous branch into the alternative path.
  **L1090 CN**: 从前一个分支过渡到备选路径。
- **L1091 EN**: Continues logic associated with callable symbol `Real<8>>`.
  **L1091 CN**: 继续与可调用符号 `Real<8>>` 相关的逻辑。
- **L1092 EN**: Executes a call or declaration centered on `builder.getContext`.
  **L1092 CN**: 执行以 `builder.getContext` 为核心的调用或声明。
- **L1093 EN**: Executes a call or declaration centered on `builder.createFunction`.
  **L1093 CN**: 执行以 `builder.createFunction` 为核心的调用或声明。
- **L1094 EN**: Closes the current lexical scope or compound statement.
  **L1094 CN**: 结束当前词法作用域或复合语句块。
- **L1095 EN**: Executes a call or declaration centered on `fir::CallOp::create`.
  **L1095 CN**: 执行以 `fir::CallOp::create` 为核心的调用或声明。
- **L1096 EN**: Closes the current lexical scope or compound statement.
  **L1096 CN**: 结束当前词法作用域或复合语句块。
- **L1097 EN**: Blank line separating nearby declarations or logic blocks.
  **L1097 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1098 EN**: Comment explains nearby logic, intent, or metadata: `VEC_ABS`.
  **L1098 CN**: 注释说明附近代码的逻辑、意图或元数据：`VEC_ABS`。
- **L1099 EN**: Continues the surrounding expression or declaration: `fir::ExtendedValue`.
  **L1099 CN**: 继续构造周围的表达式或声明：`fir::ExtendedValue`。
- **L1100 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PPCIntrinsicLibrary::genVecAbs(mlir::Type resultType,`.
  **L1100 CN**: 继续一个多行参数列表、初始化器或聚合项：`PPCIntrinsicLibrary::genVecAbs(mlir::Type resultType,`。
- **L1101 EN**: Continues the surrounding expression or declaration: `llvm::ArrayRef<fir::ExtendedValue> args) {`.
  **L1101 CN**: 继续构造周围的表达式或声明：`llvm::ArrayRef<fir::ExtendedValue> args) {`。
- **L1102 EN**: Checks an internal invariant in debug builds.
  **L1102 CN**: 在调试构建中检查内部不变式。
- **L1103 EN**: Executes a call or declaration centered on `context{builder.getContext`.
  **L1103 CN**: 执行以 `context{builder.getContext` 为核心的调用或声明。
- **L1104 EN**: Executes a call or declaration centered on `argBases{getBasesForArgs`.
  **L1104 CN**: 执行以 `argBases{getBasesForArgs` 为核心的调用或声明。

### Lines 1105-1128

````cpp
  auto vTypeInfo{getVecTypeFromFir(argBases[0])};

  mlir::func::FuncOp funcOp{nullptr};
  mlir::FunctionType ftype;
  llvm::StringRef fname{};
  if (vTypeInfo.isFloat()) {
    if (vTypeInfo.isFloat32()) {
      fname = "llvm.fabs.v4f32";
      ftype =
          genFuncType<Ty::RealVector<4>, Ty::RealVector<4>>(context, builder);
    } else if (vTypeInfo.isFloat64()) {
      fname = "llvm.fabs.v2f64";
      ftype =
          genFuncType<Ty::RealVector<8>, Ty::RealVector<8>>(context, builder);
    }

    funcOp = builder.createFunction(loc, fname, ftype);
    auto callOp{fir::CallOp::create(builder, loc, funcOp, argBases[0])};
    return callOp.getResult(0);
  } else if (auto eleTy = mlir::dyn_cast<mlir::IntegerType>(vTypeInfo.eleTy)) {
    // vec_abs(arg1) = max(0 - arg1, arg1)

    auto newVecTy{mlir::VectorType::get(vTypeInfo.len, eleTy)};
    auto varg1{builder.createConvert(loc, newVecTy, argBases[0])};
````
- **L1105 EN**: Executes a call or declaration centered on `vTypeInfo{getVecTypeFromFir`.
  **L1105 CN**: 执行以 `vTypeInfo{getVecTypeFromFir` 为核心的调用或声明。
- **L1106 EN**: Blank line separating nearby declarations or logic blocks.
  **L1106 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1107 EN**: Executes a standalone statement or declaration: `mlir::func::FuncOp funcOp{nullptr};`.
  **L1107 CN**: 执行一条独立语句或声明：`mlir::func::FuncOp funcOp{nullptr};`。
- **L1108 EN**: Executes a standalone statement or declaration: `mlir::FunctionType ftype;`.
  **L1108 CN**: 执行一条独立语句或声明：`mlir::FunctionType ftype;`。
- **L1109 EN**: Executes a standalone statement or declaration: `llvm::StringRef fname{};`.
  **L1109 CN**: 执行一条独立语句或声明：`llvm::StringRef fname{};`。
- **L1110 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1110 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1111 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1111 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1112 EN**: Executes a standalone statement or declaration: `fname = "llvm.fabs.v4f32";`.
  **L1112 CN**: 执行一条独立语句或声明：`fname = "llvm.fabs.v4f32";`。
- **L1113 EN**: Continues the surrounding expression or declaration: `ftype =`.
  **L1113 CN**: 继续构造周围的表达式或声明：`ftype =`。
- **L1114 EN**: Executes a call or declaration centered on `Ty::RealVector<4>>`.
  **L1114 CN**: 执行以 `Ty::RealVector<4>>` 为核心的调用或声明。
- **L1115 EN**: Transitions from the previous branch into an `else if` condition.
  **L1115 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1116 EN**: Executes a standalone statement or declaration: `fname = "llvm.fabs.v2f64";`.
  **L1116 CN**: 执行一条独立语句或声明：`fname = "llvm.fabs.v2f64";`。
- **L1117 EN**: Continues the surrounding expression or declaration: `ftype =`.
  **L1117 CN**: 继续构造周围的表达式或声明：`ftype =`。
- **L1118 EN**: Executes a call or declaration centered on `Ty::RealVector<8>>`.
  **L1118 CN**: 执行以 `Ty::RealVector<8>>` 为核心的调用或声明。
- **L1119 EN**: Closes the current lexical scope or compound statement.
  **L1119 CN**: 结束当前词法作用域或复合语句块。
- **L1120 EN**: Blank line separating nearby declarations or logic blocks.
  **L1120 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1121 EN**: Executes a call or declaration centered on `builder.createFunction`.
  **L1121 CN**: 执行以 `builder.createFunction` 为核心的调用或声明。
- **L1122 EN**: Executes a call or declaration centered on `callOp{fir::CallOp::create`.
  **L1122 CN**: 执行以 `callOp{fir::CallOp::create` 为核心的调用或声明。
- **L1123 EN**: Returns from the current function with `callOp.getResult(0)`.
  **L1123 CN**: 以 `callOp.getResult(0)` 从当前函数返回。
- **L1124 EN**: Transitions from the previous branch into an `else if` condition.
  **L1124 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1125 EN**: Comment explains nearby logic, intent, or metadata: `vec_abs(arg1) = max(0 - arg1, arg1)`.
  **L1125 CN**: 注释说明附近代码的逻辑、意图或元数据：`vec_abs(arg1) = max(0 - arg1, arg1)`。
- **L1126 EN**: Blank line separating nearby declarations or logic blocks.
  **L1126 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1127 EN**: Executes a call or declaration centered on `newVecTy{mlir::VectorType::get`.
  **L1127 CN**: 执行以 `newVecTy{mlir::VectorType::get` 为核心的调用或声明。
- **L1128 EN**: Executes a call or declaration centered on `varg1{builder.createConvert`.
  **L1128 CN**: 执行以 `varg1{builder.createConvert` 为核心的调用或声明。

### Lines 1129-1152

````cpp
    // construct vector(0,..)
    auto zeroVal{builder.createIntegerConstant(loc, eleTy, 0)};
    auto vZero{
        mlir::vector::BroadcastOp::create(builder, loc, newVecTy, zeroVal)};
    auto zeroSubVarg1{mlir::arith::SubIOp::create(builder, loc, vZero, varg1)};

    mlir::func::FuncOp funcOp{nullptr};
    switch (eleTy.getWidth()) {
    case 8:
      fname = "llvm.ppc.altivec.vmaxsb";
      ftype = genFuncType<Ty::IntegerVector<1>, Ty::IntegerVector<1>,
                          Ty::IntegerVector<1>>(context, builder);
      break;
    case 16:
      fname = "llvm.ppc.altivec.vmaxsh";
      ftype = genFuncType<Ty::IntegerVector<2>, Ty::IntegerVector<2>,
                          Ty::IntegerVector<2>>(context, builder);
      break;
    case 32:
      fname = "llvm.ppc.altivec.vmaxsw";
      ftype = genFuncType<Ty::IntegerVector<4>, Ty::IntegerVector<4>,
                          Ty::IntegerVector<4>>(context, builder);
      break;
    case 64:
````
- **L1129 EN**: Comment explains nearby logic, intent, or metadata: `construct vector(0,..)`.
  **L1129 CN**: 注释说明附近代码的逻辑、意图或元数据：`construct vector(0,..)`。
- **L1130 EN**: Executes a call or declaration centered on `zeroVal{builder.createIntegerConstant`.
  **L1130 CN**: 执行以 `zeroVal{builder.createIntegerConstant` 为核心的调用或声明。
- **L1131 EN**: Continues the surrounding expression or declaration: `auto vZero{`.
  **L1131 CN**: 继续构造周围的表达式或声明：`auto vZero{`。
- **L1132 EN**: Executes a call or declaration centered on `mlir::vector::BroadcastOp::create`.
  **L1132 CN**: 执行以 `mlir::vector::BroadcastOp::create` 为核心的调用或声明。
- **L1133 EN**: Executes a call or declaration centered on `zeroSubVarg1{mlir::arith::SubIOp::create`.
  **L1133 CN**: 执行以 `zeroSubVarg1{mlir::arith::SubIOp::create` 为核心的调用或声明。
- **L1134 EN**: Blank line separating nearby declarations or logic blocks.
  **L1134 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1135 EN**: Executes a standalone statement or declaration: `mlir::func::FuncOp funcOp{nullptr};`.
  **L1135 CN**: 执行一条独立语句或声明：`mlir::func::FuncOp funcOp{nullptr};`。
- **L1136 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L1136 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L1137 EN**: Introduces a switch dispatch label: `case 8:`.
  **L1137 CN**: 引入一个 switch 分发标签：`case 8:`。
- **L1138 EN**: Executes a standalone statement or declaration: `fname = "llvm.ppc.altivec.vmaxsb";`.
  **L1138 CN**: 执行一条独立语句或声明：`fname = "llvm.ppc.altivec.vmaxsb";`。
- **L1139 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ftype = genFuncType<Ty::IntegerVector<1>, Ty::IntegerVector<1>,`.
  **L1139 CN**: 继续一个多行参数列表、初始化器或聚合项：`ftype = genFuncType<Ty::IntegerVector<1>, Ty::IntegerVector<1>,`。
- **L1140 EN**: Executes a call or declaration centered on `Ty::IntegerVector<1>>`.
  **L1140 CN**: 执行以 `Ty::IntegerVector<1>>` 为核心的调用或声明。
- **L1141 EN**: Exits the nearest loop or switch statement.
  **L1141 CN**: 退出最近的循环或 switch 语句。
- **L1142 EN**: Introduces a switch dispatch label: `case 16:`.
  **L1142 CN**: 引入一个 switch 分发标签：`case 16:`。
- **L1143 EN**: Executes a standalone statement or declaration: `fname = "llvm.ppc.altivec.vmaxsh";`.
  **L1143 CN**: 执行一条独立语句或声明：`fname = "llvm.ppc.altivec.vmaxsh";`。
- **L1144 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ftype = genFuncType<Ty::IntegerVector<2>, Ty::IntegerVector<2>,`.
  **L1144 CN**: 继续一个多行参数列表、初始化器或聚合项：`ftype = genFuncType<Ty::IntegerVector<2>, Ty::IntegerVector<2>,`。
- **L1145 EN**: Executes a call or declaration centered on `Ty::IntegerVector<2>>`.
  **L1145 CN**: 执行以 `Ty::IntegerVector<2>>` 为核心的调用或声明。
- **L1146 EN**: Exits the nearest loop or switch statement.
  **L1146 CN**: 退出最近的循环或 switch 语句。
- **L1147 EN**: Introduces a switch dispatch label: `case 32:`.
  **L1147 CN**: 引入一个 switch 分发标签：`case 32:`。
- **L1148 EN**: Executes a standalone statement or declaration: `fname = "llvm.ppc.altivec.vmaxsw";`.
  **L1148 CN**: 执行一条独立语句或声明：`fname = "llvm.ppc.altivec.vmaxsw";`。
- **L1149 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ftype = genFuncType<Ty::IntegerVector<4>, Ty::IntegerVector<4>,`.
  **L1149 CN**: 继续一个多行参数列表、初始化器或聚合项：`ftype = genFuncType<Ty::IntegerVector<4>, Ty::IntegerVector<4>,`。
- **L1150 EN**: Executes a call or declaration centered on `Ty::IntegerVector<4>>`.
  **L1150 CN**: 执行以 `Ty::IntegerVector<4>>` 为核心的调用或声明。
- **L1151 EN**: Exits the nearest loop or switch statement.
  **L1151 CN**: 退出最近的循环或 switch 语句。
- **L1152 EN**: Introduces a switch dispatch label: `case 64:`.
  **L1152 CN**: 引入一个 switch 分发标签：`case 64:`。

### Lines 1153-1176

````cpp
      fname = "llvm.ppc.altivec.vmaxsd";
      ftype = genFuncType<Ty::IntegerVector<8>, Ty::IntegerVector<8>,
                          Ty::IntegerVector<8>>(context, builder);
      break;
    default:
      llvm_unreachable("invalid integer size");
    }
    funcOp = builder.createFunction(loc, fname, ftype);

    mlir::Value args[] = {zeroSubVarg1, varg1};
    auto callOp{fir::CallOp::create(builder, loc, funcOp, args)};
    return builder.createConvert(loc, argBases[0].getType(),
                                 callOp.getResult(0));
  }

  llvm_unreachable("unknown vector type");
}

// VEC_ADD, VEC_AND, VEC_SUB, VEC_MUL, VEC_XOR
template <VecOp vop>
fir::ExtendedValue PPCIntrinsicLibrary::genVecAddAndMulSubXor(
    mlir::Type resultType, llvm::ArrayRef<fir::ExtendedValue> args) {
  assert(args.size() == 2);
  auto argBases{getBasesForArgs(args)};
````
- **L1153 EN**: Executes a standalone statement or declaration: `fname = "llvm.ppc.altivec.vmaxsd";`.
  **L1153 CN**: 执行一条独立语句或声明：`fname = "llvm.ppc.altivec.vmaxsd";`。
- **L1154 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ftype = genFuncType<Ty::IntegerVector<8>, Ty::IntegerVector<8>,`.
  **L1154 CN**: 继续一个多行参数列表、初始化器或聚合项：`ftype = genFuncType<Ty::IntegerVector<8>, Ty::IntegerVector<8>,`。
- **L1155 EN**: Executes a call or declaration centered on `Ty::IntegerVector<8>>`.
  **L1155 CN**: 执行以 `Ty::IntegerVector<8>>` 为核心的调用或声明。
- **L1156 EN**: Exits the nearest loop or switch statement.
  **L1156 CN**: 退出最近的循环或 switch 语句。
- **L1157 EN**: Introduces a switch dispatch label: `default:`.
  **L1157 CN**: 引入一个 switch 分发标签：`default:`。
- **L1158 EN**: Marks this control path as unreachable to LLVM.
  **L1158 CN**: 将该控制路径标记为 LLVM 认为不可达。
- **L1159 EN**: Closes the current lexical scope or compound statement.
  **L1159 CN**: 结束当前词法作用域或复合语句块。
- **L1160 EN**: Executes a call or declaration centered on `builder.createFunction`.
  **L1160 CN**: 执行以 `builder.createFunction` 为核心的调用或声明。
- **L1161 EN**: Blank line separating nearby declarations or logic blocks.
  **L1161 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1162 EN**: Executes a standalone statement or declaration: `mlir::Value args[] = {zeroSubVarg1, varg1};`.
  **L1162 CN**: 执行一条独立语句或声明：`mlir::Value args[] = {zeroSubVarg1, varg1};`。
- **L1163 EN**: Executes a call or declaration centered on `callOp{fir::CallOp::create`.
  **L1163 CN**: 执行以 `callOp{fir::CallOp::create` 为核心的调用或声明。
- **L1164 EN**: Returns from the current function with `builder.createConvert(loc, argBases[0].getType(),`.
  **L1164 CN**: 以 `builder.createConvert(loc, argBases[0].getType(),` 从当前函数返回。
- **L1165 EN**: Executes a call or declaration centered on `callOp.getResult`.
  **L1165 CN**: 执行以 `callOp.getResult` 为核心的调用或声明。
- **L1166 EN**: Closes the current lexical scope or compound statement.
  **L1166 CN**: 结束当前词法作用域或复合语句块。
- **L1167 EN**: Blank line separating nearby declarations or logic blocks.
  **L1167 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1168 EN**: Marks this control path as unreachable to LLVM.
  **L1168 CN**: 将该控制路径标记为 LLVM 认为不可达。
- **L1169 EN**: Closes the current lexical scope or compound statement.
  **L1169 CN**: 结束当前词法作用域或复合语句块。
- **L1170 EN**: Blank line separating nearby declarations or logic blocks.
  **L1170 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1171 EN**: Comment explains nearby logic, intent, or metadata: `VEC_ADD, VEC_AND, VEC_SUB, VEC_MUL, VEC_XOR`.
  **L1171 CN**: 注释说明附近代码的逻辑、意图或元数据：`VEC_ADD, VEC_AND, VEC_SUB, VEC_MUL, VEC_XOR`。
- **L1172 EN**: Introduces template parameters or specialization context: `template <VecOp vop>`.
  **L1172 CN**: 为后续声明引入模板参数或特化上下文：`template <VecOp vop>`。
- **L1173 EN**: Continues logic associated with callable symbol `genVecAddAndMulSubXor`.
  **L1173 CN**: 继续与可调用符号 `genVecAddAndMulSubXor` 相关的逻辑。
- **L1174 EN**: Continues the surrounding expression or declaration: `mlir::Type resultType, llvm::ArrayRef<fir::ExtendedValue> args) {`.
  **L1174 CN**: 继续构造周围的表达式或声明：`mlir::Type resultType, llvm::ArrayRef<fir::ExtendedValue> args) {`。
- **L1175 EN**: Checks an internal invariant in debug builds.
  **L1175 CN**: 在调试构建中检查内部不变式。
- **L1176 EN**: Executes a call or declaration centered on `argBases{getBasesForArgs`.
  **L1176 CN**: 执行以 `argBases{getBasesForArgs` 为核心的调用或声明。

### Lines 1177-1200

````cpp
  auto argsTy{getTypesForArgs(argBases)};
  assert(mlir::isa<fir::VectorType>(argsTy[0]) &&
         mlir::isa<fir::VectorType>(argsTy[1]));

  auto vecTyInfo{getVecTypeFromFir(argBases[0])};

  const auto isInteger{mlir::isa<mlir::IntegerType>(vecTyInfo.eleTy)};
  const auto isFloat{mlir::isa<mlir::FloatType>(vecTyInfo.eleTy)};
  assert((isInteger || isFloat) && "unknown vector type");

  auto vargs{convertVecArgs(builder, loc, vecTyInfo, argBases)};

  mlir::Value r{nullptr};
  switch (vop) {
  case VecOp::Add:
    if (isInteger)
      r = mlir::arith::AddIOp::create(builder, loc, vargs[0], vargs[1]);
    else if (isFloat)
      r = mlir::arith::AddFOp::create(builder, loc, vargs[0], vargs[1]);
    break;
  case VecOp::Mul:
    if (isInteger)
      r = mlir::arith::MulIOp::create(builder, loc, vargs[0], vargs[1]);
    else if (isFloat)
````
- **L1177 EN**: Executes a call or declaration centered on `argsTy{getTypesForArgs`.
  **L1177 CN**: 执行以 `argsTy{getTypesForArgs` 为核心的调用或声明。
- **L1178 EN**: Checks an internal invariant in debug builds.
  **L1178 CN**: 在调试构建中检查内部不变式。
- **L1179 EN**: Executes a call or declaration centered on `mlir::isa<fir::VectorType>`.
  **L1179 CN**: 执行以 `mlir::isa<fir::VectorType>` 为核心的调用或声明。
- **L1180 EN**: Blank line separating nearby declarations or logic blocks.
  **L1180 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1181 EN**: Executes a call or declaration centered on `vecTyInfo{getVecTypeFromFir`.
  **L1181 CN**: 执行以 `vecTyInfo{getVecTypeFromFir` 为核心的调用或声明。
- **L1182 EN**: Blank line separating nearby declarations or logic blocks.
  **L1182 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1183 EN**: Executes a call or declaration centered on `isInteger{mlir::isa<mlir::IntegerType>`.
  **L1183 CN**: 执行以 `isInteger{mlir::isa<mlir::IntegerType>` 为核心的调用或声明。
- **L1184 EN**: Executes a call or declaration centered on `isFloat{mlir::isa<mlir::FloatType>`.
  **L1184 CN**: 执行以 `isFloat{mlir::isa<mlir::FloatType>` 为核心的调用或声明。
- **L1185 EN**: Checks an internal invariant in debug builds.
  **L1185 CN**: 在调试构建中检查内部不变式。
- **L1186 EN**: Blank line separating nearby declarations or logic blocks.
  **L1186 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1187 EN**: Executes a call or declaration centered on `vargs{convertVecArgs`.
  **L1187 CN**: 执行以 `vargs{convertVecArgs` 为核心的调用或声明。
- **L1188 EN**: Blank line separating nearby declarations or logic blocks.
  **L1188 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1189 EN**: Executes a standalone statement or declaration: `mlir::Value r{nullptr};`.
  **L1189 CN**: 执行一条独立语句或声明：`mlir::Value r{nullptr};`。
- **L1190 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L1190 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L1191 EN**: Introduces a switch dispatch label: `case VecOp::Add:`.
  **L1191 CN**: 引入一个 switch 分发标签：`case VecOp::Add:`。
- **L1192 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1192 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1193 EN**: Executes a call or declaration centered on `mlir::arith::AddIOp::create`.
  **L1193 CN**: 执行以 `mlir::arith::AddIOp::create` 为核心的调用或声明。
- **L1194 EN**: Starts the alternative branch of the preceding conditional.
  **L1194 CN**: 开始前一个条件语句的备选分支。
- **L1195 EN**: Executes a call or declaration centered on `mlir::arith::AddFOp::create`.
  **L1195 CN**: 执行以 `mlir::arith::AddFOp::create` 为核心的调用或声明。
- **L1196 EN**: Exits the nearest loop or switch statement.
  **L1196 CN**: 退出最近的循环或 switch 语句。
- **L1197 EN**: Introduces a switch dispatch label: `case VecOp::Mul:`.
  **L1197 CN**: 引入一个 switch 分发标签：`case VecOp::Mul:`。
- **L1198 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1198 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1199 EN**: Executes a call or declaration centered on `mlir::arith::MulIOp::create`.
  **L1199 CN**: 执行以 `mlir::arith::MulIOp::create` 为核心的调用或声明。
- **L1200 EN**: Starts the alternative branch of the preceding conditional.
  **L1200 CN**: 开始前一个条件语句的备选分支。

### Lines 1201-1224

````cpp
      r = mlir::arith::MulFOp::create(builder, loc, vargs[0], vargs[1]);
    break;
  case VecOp::Sub:
    if (isInteger)
      r = mlir::arith::SubIOp::create(builder, loc, vargs[0], vargs[1]);
    else if (isFloat)
      r = mlir::arith::SubFOp::create(builder, loc, vargs[0], vargs[1]);
    break;
  case VecOp::And:
  case VecOp::Xor: {
    mlir::Value arg1{nullptr};
    mlir::Value arg2{nullptr};
    if (isInteger) {
      arg1 = vargs[0];
      arg2 = vargs[1];
    } else if (isFloat) {
      // bitcast the arguments to integer
      auto wd{mlir::dyn_cast<mlir::FloatType>(vecTyInfo.eleTy).getWidth()};
      auto ftype{builder.getIntegerType(wd)};
      auto bcVecTy{mlir::VectorType::get(vecTyInfo.len, ftype)};
      arg1 = mlir::vector::BitCastOp::create(builder, loc, bcVecTy, vargs[0]);
      arg2 = mlir::vector::BitCastOp::create(builder, loc, bcVecTy, vargs[1]);
    }
    if (vop == VecOp::And)
````
- **L1201 EN**: Executes a call or declaration centered on `mlir::arith::MulFOp::create`.
  **L1201 CN**: 执行以 `mlir::arith::MulFOp::create` 为核心的调用或声明。
- **L1202 EN**: Exits the nearest loop or switch statement.
  **L1202 CN**: 退出最近的循环或 switch 语句。
- **L1203 EN**: Introduces a switch dispatch label: `case VecOp::Sub:`.
  **L1203 CN**: 引入一个 switch 分发标签：`case VecOp::Sub:`。
- **L1204 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1204 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1205 EN**: Executes a call or declaration centered on `mlir::arith::SubIOp::create`.
  **L1205 CN**: 执行以 `mlir::arith::SubIOp::create` 为核心的调用或声明。
- **L1206 EN**: Starts the alternative branch of the preceding conditional.
  **L1206 CN**: 开始前一个条件语句的备选分支。
- **L1207 EN**: Executes a call or declaration centered on `mlir::arith::SubFOp::create`.
  **L1207 CN**: 执行以 `mlir::arith::SubFOp::create` 为核心的调用或声明。
- **L1208 EN**: Exits the nearest loop or switch statement.
  **L1208 CN**: 退出最近的循环或 switch 语句。
- **L1209 EN**: Introduces a switch dispatch label: `case VecOp::And:`.
  **L1209 CN**: 引入一个 switch 分发标签：`case VecOp::And:`。
- **L1210 EN**: Introduces a switch dispatch label: `case VecOp::Xor: {`.
  **L1210 CN**: 引入一个 switch 分发标签：`case VecOp::Xor: {`。
- **L1211 EN**: Executes a standalone statement or declaration: `mlir::Value arg1{nullptr};`.
  **L1211 CN**: 执行一条独立语句或声明：`mlir::Value arg1{nullptr};`。
- **L1212 EN**: Executes a standalone statement or declaration: `mlir::Value arg2{nullptr};`.
  **L1212 CN**: 执行一条独立语句或声明：`mlir::Value arg2{nullptr};`。
- **L1213 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1213 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1214 EN**: Executes a standalone statement or declaration: `arg1 = vargs[0];`.
  **L1214 CN**: 执行一条独立语句或声明：`arg1 = vargs[0];`。
- **L1215 EN**: Executes a standalone statement or declaration: `arg2 = vargs[1];`.
  **L1215 CN**: 执行一条独立语句或声明：`arg2 = vargs[1];`。
- **L1216 EN**: Transitions from the previous branch into an `else if` condition.
  **L1216 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1217 EN**: Comment explains nearby logic, intent, or metadata: `bitcast the arguments to integer`.
  **L1217 CN**: 注释说明附近代码的逻辑、意图或元数据：`bitcast the arguments to integer`。
- **L1218 EN**: Executes a call or declaration centered on `wd{mlir::dyn_cast<mlir::FloatType>`.
  **L1218 CN**: 执行以 `wd{mlir::dyn_cast<mlir::FloatType>` 为核心的调用或声明。
- **L1219 EN**: Executes a call or declaration centered on `ftype{builder.getIntegerType`.
  **L1219 CN**: 执行以 `ftype{builder.getIntegerType` 为核心的调用或声明。
- **L1220 EN**: Executes a call or declaration centered on `bcVecTy{mlir::VectorType::get`.
  **L1220 CN**: 执行以 `bcVecTy{mlir::VectorType::get` 为核心的调用或声明。
- **L1221 EN**: Executes a call or declaration centered on `mlir::vector::BitCastOp::create`.
  **L1221 CN**: 执行以 `mlir::vector::BitCastOp::create` 为核心的调用或声明。
- **L1222 EN**: Executes a call or declaration centered on `mlir::vector::BitCastOp::create`.
  **L1222 CN**: 执行以 `mlir::vector::BitCastOp::create` 为核心的调用或声明。
- **L1223 EN**: Closes the current lexical scope or compound statement.
  **L1223 CN**: 结束当前词法作用域或复合语句块。
- **L1224 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1224 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 1225-1248

````cpp
      r = mlir::arith::AndIOp::create(builder, loc, arg1, arg2);
    else if (vop == VecOp::Xor)
      r = mlir::arith::XOrIOp::create(builder, loc, arg1, arg2);

    if (isFloat)
      r = mlir::vector::BitCastOp::create(builder, loc, vargs[0].getType(), r);

    break;
  }
  }

  return builder.createConvert(loc, argsTy[0], r);
}

// VEC_ANY_GE
template <VecOp vop>
fir::ExtendedValue
PPCIntrinsicLibrary::genVecAnyCompare(mlir::Type resultType,
                                      llvm::ArrayRef<fir::ExtendedValue> args) {
  assert(args.size() == 2);
  assert(vop == VecOp::Anyge && "unknown vector compare operation");
  auto argBases{getBasesForArgs(args)};
  VecTypeInfo vTypeInfo{getVecTypeFromFir(argBases[0])};
  [[maybe_unused]] const auto isSupportedTy{
````
- **L1225 EN**: Executes a call or declaration centered on `mlir::arith::AndIOp::create`.
  **L1225 CN**: 执行以 `mlir::arith::AndIOp::create` 为核心的调用或声明。
- **L1226 EN**: Starts the alternative branch of the preceding conditional.
  **L1226 CN**: 开始前一个条件语句的备选分支。
- **L1227 EN**: Executes a call or declaration centered on `mlir::arith::XOrIOp::create`.
  **L1227 CN**: 执行以 `mlir::arith::XOrIOp::create` 为核心的调用或声明。
- **L1228 EN**: Blank line separating nearby declarations or logic blocks.
  **L1228 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1229 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1229 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1230 EN**: Executes a call or declaration centered on `mlir::vector::BitCastOp::create`.
  **L1230 CN**: 执行以 `mlir::vector::BitCastOp::create` 为核心的调用或声明。
- **L1231 EN**: Blank line separating nearby declarations or logic blocks.
  **L1231 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1232 EN**: Exits the nearest loop or switch statement.
  **L1232 CN**: 退出最近的循环或 switch 语句。
- **L1233 EN**: Closes the current lexical scope or compound statement.
  **L1233 CN**: 结束当前词法作用域或复合语句块。
- **L1234 EN**: Closes the current lexical scope or compound statement.
  **L1234 CN**: 结束当前词法作用域或复合语句块。
- **L1235 EN**: Blank line separating nearby declarations or logic blocks.
  **L1235 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1236 EN**: Returns from the current function with `builder.createConvert(loc, argsTy[0], r)`.
  **L1236 CN**: 以 `builder.createConvert(loc, argsTy[0], r)` 从当前函数返回。
- **L1237 EN**: Closes the current lexical scope or compound statement.
  **L1237 CN**: 结束当前词法作用域或复合语句块。
- **L1238 EN**: Blank line separating nearby declarations or logic blocks.
  **L1238 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1239 EN**: Comment explains nearby logic, intent, or metadata: `VEC_ANY_GE`.
  **L1239 CN**: 注释说明附近代码的逻辑、意图或元数据：`VEC_ANY_GE`。
- **L1240 EN**: Introduces template parameters or specialization context: `template <VecOp vop>`.
  **L1240 CN**: 为后续声明引入模板参数或特化上下文：`template <VecOp vop>`。
- **L1241 EN**: Continues the surrounding expression or declaration: `fir::ExtendedValue`.
  **L1241 CN**: 继续构造周围的表达式或声明：`fir::ExtendedValue`。
- **L1242 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PPCIntrinsicLibrary::genVecAnyCompare(mlir::Type resultType,`.
  **L1242 CN**: 继续一个多行参数列表、初始化器或聚合项：`PPCIntrinsicLibrary::genVecAnyCompare(mlir::Type resultType,`。
- **L1243 EN**: Continues the surrounding expression or declaration: `llvm::ArrayRef<fir::ExtendedValue> args) {`.
  **L1243 CN**: 继续构造周围的表达式或声明：`llvm::ArrayRef<fir::ExtendedValue> args) {`。
- **L1244 EN**: Checks an internal invariant in debug builds.
  **L1244 CN**: 在调试构建中检查内部不变式。
- **L1245 EN**: Checks an internal invariant in debug builds.
  **L1245 CN**: 在调试构建中检查内部不变式。
- **L1246 EN**: Executes a call or declaration centered on `argBases{getBasesForArgs`.
  **L1246 CN**: 执行以 `argBases{getBasesForArgs` 为核心的调用或声明。
- **L1247 EN**: Executes a call or declaration centered on `vTypeInfo{getVecTypeFromFir`.
  **L1247 CN**: 执行以 `vTypeInfo{getVecTypeFromFir` 为核心的调用或声明。
- **L1248 EN**: Continues the surrounding expression or declaration: `[[maybe_unused]] const auto isSupportedTy{`.
  **L1248 CN**: 继续构造周围的表达式或声明：`[[maybe_unused]] const auto isSupportedTy{`。

### Lines 1249-1272

````cpp
      mlir::isa<mlir::Float32Type, mlir::Float64Type, mlir::IntegerType>(
          vTypeInfo.eleTy)};
  assert(isSupportedTy && "unsupported vector type");

  // Constants for mapping CR6 bits to predicate result
  enum { CR6_EQ_REV = 1, CR6_LT_REV = 3 };

  auto context{builder.getContext()};

  static std::map<std::pair<ParamTypeId, unsigned>,
                  std::pair<llvm::StringRef, mlir::FunctionType>>
      uiBuiltin{
          {std::make_pair(ParamTypeId::IntegerVector, 8),
           std::make_pair(
               "llvm.ppc.altivec.vcmpgtsb.p",
               genFuncType<Ty::Integer<4>, Ty::Integer<4>, Ty::IntegerVector<1>,
                           Ty::IntegerVector<1>>(context, builder))},
          {std::make_pair(ParamTypeId::IntegerVector, 16),
           std::make_pair(
               "llvm.ppc.altivec.vcmpgtsh.p",
               genFuncType<Ty::Integer<4>, Ty::Integer<4>, Ty::IntegerVector<2>,
                           Ty::IntegerVector<2>>(context, builder))},
          {std::make_pair(ParamTypeId::IntegerVector, 32),
           std::make_pair(
````
- **L1249 EN**: Continues logic associated with callable symbol `IntegerType>`.
  **L1249 CN**: 继续与可调用符号 `IntegerType>` 相关的逻辑。
- **L1250 EN**: Executes a standalone statement or declaration: `vTypeInfo.eleTy)};`.
  **L1250 CN**: 执行一条独立语句或声明：`vTypeInfo.eleTy)};`。
- **L1251 EN**: Checks an internal invariant in debug builds.
  **L1251 CN**: 在调试构建中检查内部不变式。
- **L1252 EN**: Blank line separating nearby declarations or logic blocks.
  **L1252 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1253 EN**: Comment explains nearby logic, intent, or metadata: `Constants for mapping CR6 bits to predicate result`.
  **L1253 CN**: 注释说明附近代码的逻辑、意图或元数据：`Constants for mapping CR6 bits to predicate result`。
- **L1254 EN**: Declares enum ``.
  **L1254 CN**: 声明 enum ``。
- **L1255 EN**: Blank line separating nearby declarations or logic blocks.
  **L1255 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1256 EN**: Executes a call or declaration centered on `context{builder.getContext`.
  **L1256 CN**: 执行以 `context{builder.getContext` 为核心的调用或声明。
- **L1257 EN**: Blank line separating nearby declarations or logic blocks.
  **L1257 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1258 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static std::map<std::pair<ParamTypeId, unsigned>,`.
  **L1258 CN**: 继续一个多行参数列表、初始化器或聚合项：`static std::map<std::pair<ParamTypeId, unsigned>,`。
- **L1259 EN**: Continues the surrounding expression or declaration: `std::pair<llvm::StringRef, mlir::FunctionType>>`.
  **L1259 CN**: 继续构造周围的表达式或声明：`std::pair<llvm::StringRef, mlir::FunctionType>>`。
- **L1260 EN**: Continues the surrounding expression or declaration: `uiBuiltin{`.
  **L1260 CN**: 继续构造周围的表达式或声明：`uiBuiltin{`。
- **L1261 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{std::make_pair(ParamTypeId::IntegerVector, 8),`.
  **L1261 CN**: 继续一个多行参数列表、初始化器或聚合项：`{std::make_pair(ParamTypeId::IntegerVector, 8),`。
- **L1262 EN**: Continues logic associated with callable symbol `make_pair`.
  **L1262 CN**: 继续与可调用符号 `make_pair` 相关的逻辑。
- **L1263 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"llvm.ppc.altivec.vcmpgtsb.p",`.
  **L1263 CN**: 继续一个多行参数列表、初始化器或聚合项：`"llvm.ppc.altivec.vcmpgtsb.p",`。
- **L1264 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genFuncType<Ty::Integer<4>, Ty::Integer<4>, Ty::IntegerVector<1>,`.
  **L1264 CN**: 继续一个多行参数列表、初始化器或聚合项：`genFuncType<Ty::Integer<4>, Ty::Integer<4>, Ty::IntegerVector<1>,`。
- **L1265 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Ty::IntegerVector<1>>(context, builder))},`.
  **L1265 CN**: 继续一个多行参数列表、初始化器或聚合项：`Ty::IntegerVector<1>>(context, builder))},`。
- **L1266 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{std::make_pair(ParamTypeId::IntegerVector, 16),`.
  **L1266 CN**: 继续一个多行参数列表、初始化器或聚合项：`{std::make_pair(ParamTypeId::IntegerVector, 16),`。
- **L1267 EN**: Continues logic associated with callable symbol `make_pair`.
  **L1267 CN**: 继续与可调用符号 `make_pair` 相关的逻辑。
- **L1268 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"llvm.ppc.altivec.vcmpgtsh.p",`.
  **L1268 CN**: 继续一个多行参数列表、初始化器或聚合项：`"llvm.ppc.altivec.vcmpgtsh.p",`。
- **L1269 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genFuncType<Ty::Integer<4>, Ty::Integer<4>, Ty::IntegerVector<2>,`.
  **L1269 CN**: 继续一个多行参数列表、初始化器或聚合项：`genFuncType<Ty::Integer<4>, Ty::Integer<4>, Ty::IntegerVector<2>,`。
- **L1270 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Ty::IntegerVector<2>>(context, builder))},`.
  **L1270 CN**: 继续一个多行参数列表、初始化器或聚合项：`Ty::IntegerVector<2>>(context, builder))},`。
- **L1271 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{std::make_pair(ParamTypeId::IntegerVector, 32),`.
  **L1271 CN**: 继续一个多行参数列表、初始化器或聚合项：`{std::make_pair(ParamTypeId::IntegerVector, 32),`。
- **L1272 EN**: Continues logic associated with callable symbol `make_pair`.
  **L1272 CN**: 继续与可调用符号 `make_pair` 相关的逻辑。

### Lines 1273-1296

````cpp
               "llvm.ppc.altivec.vcmpgtsw.p",
               genFuncType<Ty::Integer<4>, Ty::Integer<4>, Ty::IntegerVector<4>,
                           Ty::IntegerVector<4>>(context, builder))},
          {std::make_pair(ParamTypeId::IntegerVector, 64),
           std::make_pair(
               "llvm.ppc.altivec.vcmpgtsd.p",
               genFuncType<Ty::Integer<4>, Ty::Integer<4>, Ty::IntegerVector<8>,
                           Ty::IntegerVector<8>>(context, builder))},
          {std::make_pair(ParamTypeId::UnsignedVector, 8),
           std::make_pair(
               "llvm.ppc.altivec.vcmpgtub.p",
               genFuncType<Ty::Integer<4>, Ty::Integer<4>,
                           Ty::UnsignedVector<1>, Ty::UnsignedVector<1>>(
                   context, builder))},
          {std::make_pair(ParamTypeId::UnsignedVector, 16),
           std::make_pair(
               "llvm.ppc.altivec.vcmpgtuh.p",
               genFuncType<Ty::Integer<4>, Ty::Integer<4>,
                           Ty::UnsignedVector<2>, Ty::UnsignedVector<2>>(
                   context, builder))},
          {std::make_pair(ParamTypeId::UnsignedVector, 32),
           std::make_pair(
               "llvm.ppc.altivec.vcmpgtuw.p",
               genFuncType<Ty::Integer<4>, Ty::Integer<4>,
````
- **L1273 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"llvm.ppc.altivec.vcmpgtsw.p",`.
  **L1273 CN**: 继续一个多行参数列表、初始化器或聚合项：`"llvm.ppc.altivec.vcmpgtsw.p",`。
- **L1274 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genFuncType<Ty::Integer<4>, Ty::Integer<4>, Ty::IntegerVector<4>,`.
  **L1274 CN**: 继续一个多行参数列表、初始化器或聚合项：`genFuncType<Ty::Integer<4>, Ty::Integer<4>, Ty::IntegerVector<4>,`。
- **L1275 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Ty::IntegerVector<4>>(context, builder))},`.
  **L1275 CN**: 继续一个多行参数列表、初始化器或聚合项：`Ty::IntegerVector<4>>(context, builder))},`。
- **L1276 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{std::make_pair(ParamTypeId::IntegerVector, 64),`.
  **L1276 CN**: 继续一个多行参数列表、初始化器或聚合项：`{std::make_pair(ParamTypeId::IntegerVector, 64),`。
- **L1277 EN**: Continues logic associated with callable symbol `make_pair`.
  **L1277 CN**: 继续与可调用符号 `make_pair` 相关的逻辑。
- **L1278 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"llvm.ppc.altivec.vcmpgtsd.p",`.
  **L1278 CN**: 继续一个多行参数列表、初始化器或聚合项：`"llvm.ppc.altivec.vcmpgtsd.p",`。
- **L1279 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genFuncType<Ty::Integer<4>, Ty::Integer<4>, Ty::IntegerVector<8>,`.
  **L1279 CN**: 继续一个多行参数列表、初始化器或聚合项：`genFuncType<Ty::Integer<4>, Ty::Integer<4>, Ty::IntegerVector<8>,`。
- **L1280 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Ty::IntegerVector<8>>(context, builder))},`.
  **L1280 CN**: 继续一个多行参数列表、初始化器或聚合项：`Ty::IntegerVector<8>>(context, builder))},`。
- **L1281 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{std::make_pair(ParamTypeId::UnsignedVector, 8),`.
  **L1281 CN**: 继续一个多行参数列表、初始化器或聚合项：`{std::make_pair(ParamTypeId::UnsignedVector, 8),`。
- **L1282 EN**: Continues logic associated with callable symbol `make_pair`.
  **L1282 CN**: 继续与可调用符号 `make_pair` 相关的逻辑。
- **L1283 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"llvm.ppc.altivec.vcmpgtub.p",`.
  **L1283 CN**: 继续一个多行参数列表、初始化器或聚合项：`"llvm.ppc.altivec.vcmpgtub.p",`。
- **L1284 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genFuncType<Ty::Integer<4>, Ty::Integer<4>,`.
  **L1284 CN**: 继续一个多行参数列表、初始化器或聚合项：`genFuncType<Ty::Integer<4>, Ty::Integer<4>,`。
- **L1285 EN**: Continues logic associated with callable symbol `UnsignedVector<1>>`.
  **L1285 CN**: 继续与可调用符号 `UnsignedVector<1>>` 相关的逻辑。
- **L1286 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context, builder))},`.
  **L1286 CN**: 继续一个多行参数列表、初始化器或聚合项：`context, builder))},`。
- **L1287 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{std::make_pair(ParamTypeId::UnsignedVector, 16),`.
  **L1287 CN**: 继续一个多行参数列表、初始化器或聚合项：`{std::make_pair(ParamTypeId::UnsignedVector, 16),`。
- **L1288 EN**: Continues logic associated with callable symbol `make_pair`.
  **L1288 CN**: 继续与可调用符号 `make_pair` 相关的逻辑。
- **L1289 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"llvm.ppc.altivec.vcmpgtuh.p",`.
  **L1289 CN**: 继续一个多行参数列表、初始化器或聚合项：`"llvm.ppc.altivec.vcmpgtuh.p",`。
- **L1290 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genFuncType<Ty::Integer<4>, Ty::Integer<4>,`.
  **L1290 CN**: 继续一个多行参数列表、初始化器或聚合项：`genFuncType<Ty::Integer<4>, Ty::Integer<4>,`。
- **L1291 EN**: Continues logic associated with callable symbol `UnsignedVector<2>>`.
  **L1291 CN**: 继续与可调用符号 `UnsignedVector<2>>` 相关的逻辑。
- **L1292 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context, builder))},`.
  **L1292 CN**: 继续一个多行参数列表、初始化器或聚合项：`context, builder))},`。
- **L1293 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{std::make_pair(ParamTypeId::UnsignedVector, 32),`.
  **L1293 CN**: 继续一个多行参数列表、初始化器或聚合项：`{std::make_pair(ParamTypeId::UnsignedVector, 32),`。
- **L1294 EN**: Continues logic associated with callable symbol `make_pair`.
  **L1294 CN**: 继续与可调用符号 `make_pair` 相关的逻辑。
- **L1295 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"llvm.ppc.altivec.vcmpgtuw.p",`.
  **L1295 CN**: 继续一个多行参数列表、初始化器或聚合项：`"llvm.ppc.altivec.vcmpgtuw.p",`。
- **L1296 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genFuncType<Ty::Integer<4>, Ty::Integer<4>,`.
  **L1296 CN**: 继续一个多行参数列表、初始化器或聚合项：`genFuncType<Ty::Integer<4>, Ty::Integer<4>,`。

### Lines 1297-1320

````cpp
                           Ty::UnsignedVector<4>, Ty::UnsignedVector<4>>(
                   context, builder))},
          {std::make_pair(ParamTypeId::UnsignedVector, 64),
           std::make_pair(
               "llvm.ppc.altivec.vcmpgtud.p",
               genFuncType<Ty::Integer<4>, Ty::Integer<4>,
                           Ty::UnsignedVector<8>, Ty::UnsignedVector<8>>(
                   context, builder))},
      };

  mlir::FunctionType ftype{nullptr};
  llvm::StringRef fname;
  const auto i32Ty{mlir::IntegerType::get(context, 32)};
  llvm::SmallVector<mlir::Value> cmpArgs;
  mlir::Value op{nullptr};
  const auto width{vTypeInfo.eleTy.getIntOrFloatBitWidth()};

  if (auto elementTy = mlir::dyn_cast<mlir::IntegerType>(vTypeInfo.eleTy)) {
    std::pair<llvm::StringRef, mlir::FunctionType> bi;
    bi = (elementTy.isUnsignedInteger())
             ? uiBuiltin[std::pair(ParamTypeId::UnsignedVector, width)]
             : uiBuiltin[std::pair(ParamTypeId::IntegerVector, width)];

    fname = std::get<0>(bi);
````
- **L1297 EN**: Continues logic associated with callable symbol `UnsignedVector<4>>`.
  **L1297 CN**: 继续与可调用符号 `UnsignedVector<4>>` 相关的逻辑。
- **L1298 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context, builder))},`.
  **L1298 CN**: 继续一个多行参数列表、初始化器或聚合项：`context, builder))},`。
- **L1299 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{std::make_pair(ParamTypeId::UnsignedVector, 64),`.
  **L1299 CN**: 继续一个多行参数列表、初始化器或聚合项：`{std::make_pair(ParamTypeId::UnsignedVector, 64),`。
- **L1300 EN**: Continues logic associated with callable symbol `make_pair`.
  **L1300 CN**: 继续与可调用符号 `make_pair` 相关的逻辑。
- **L1301 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"llvm.ppc.altivec.vcmpgtud.p",`.
  **L1301 CN**: 继续一个多行参数列表、初始化器或聚合项：`"llvm.ppc.altivec.vcmpgtud.p",`。
- **L1302 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genFuncType<Ty::Integer<4>, Ty::Integer<4>,`.
  **L1302 CN**: 继续一个多行参数列表、初始化器或聚合项：`genFuncType<Ty::Integer<4>, Ty::Integer<4>,`。
- **L1303 EN**: Continues logic associated with callable symbol `UnsignedVector<8>>`.
  **L1303 CN**: 继续与可调用符号 `UnsignedVector<8>>` 相关的逻辑。
- **L1304 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context, builder))},`.
  **L1304 CN**: 继续一个多行参数列表、初始化器或聚合项：`context, builder))},`。
- **L1305 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L1305 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L1306 EN**: Blank line separating nearby declarations or logic blocks.
  **L1306 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1307 EN**: Executes a standalone statement or declaration: `mlir::FunctionType ftype{nullptr};`.
  **L1307 CN**: 执行一条独立语句或声明：`mlir::FunctionType ftype{nullptr};`。
- **L1308 EN**: Executes a standalone statement or declaration: `llvm::StringRef fname;`.
  **L1308 CN**: 执行一条独立语句或声明：`llvm::StringRef fname;`。
- **L1309 EN**: Executes a call or declaration centered on `i32Ty{mlir::IntegerType::get`.
  **L1309 CN**: 执行以 `i32Ty{mlir::IntegerType::get` 为核心的调用或声明。
- **L1310 EN**: Executes a standalone statement or declaration: `llvm::SmallVector<mlir::Value> cmpArgs;`.
  **L1310 CN**: 执行一条独立语句或声明：`llvm::SmallVector<mlir::Value> cmpArgs;`。
- **L1311 EN**: Executes a standalone statement or declaration: `mlir::Value op{nullptr};`.
  **L1311 CN**: 执行一条独立语句或声明：`mlir::Value op{nullptr};`。
- **L1312 EN**: Executes a call or declaration centered on `width{vTypeInfo.eleTy.getIntOrFloatBitWidth`.
  **L1312 CN**: 执行以 `width{vTypeInfo.eleTy.getIntOrFloatBitWidth` 为核心的调用或声明。
- **L1313 EN**: Blank line separating nearby declarations or logic blocks.
  **L1313 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1314 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1314 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1315 EN**: Executes a standalone statement or declaration: `std::pair<llvm::StringRef, mlir::FunctionType> bi;`.
  **L1315 CN**: 执行一条独立语句或声明：`std::pair<llvm::StringRef, mlir::FunctionType> bi;`。
- **L1316 EN**: Continues logic associated with callable symbol `isUnsignedInteger`.
  **L1316 CN**: 继续与可调用符号 `isUnsignedInteger` 相关的逻辑。
- **L1317 EN**: Continues logic associated with callable symbol `pair`.
  **L1317 CN**: 继续与可调用符号 `pair` 相关的逻辑。
- **L1318 EN**: Executes a call or declaration centered on `uiBuiltin[std::pair`.
  **L1318 CN**: 执行以 `uiBuiltin[std::pair` 为核心的调用或声明。
- **L1319 EN**: Blank line separating nearby declarations or logic blocks.
  **L1319 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1320 EN**: Executes a call or declaration centered on `std::get<0>`.
  **L1320 CN**: 执行以 `std::get<0>` 为核心的调用或声明。

### Lines 1321-1344

````cpp
    ftype = std::get<1>(bi);

    op = builder.createIntegerConstant(loc, i32Ty, CR6_LT_REV);
    cmpArgs.emplace_back(op);
    // reverse the argument order
    cmpArgs.emplace_back(argBases[1]);
    cmpArgs.emplace_back(argBases[0]);
  } else if (vTypeInfo.isFloat()) {
    if (vTypeInfo.isFloat32()) {
      fname = "llvm.ppc.vsx.xvcmpgesp.p";
      ftype = genFuncType<Ty::Integer<4>, Ty::Integer<4>, Ty::RealVector<4>,
                          Ty::RealVector<4>>(context, builder);
    } else {
      fname = "llvm.ppc.vsx.xvcmpgedp.p";
      ftype = genFuncType<Ty::Integer<4>, Ty::Integer<4>, Ty::RealVector<8>,
                          Ty::RealVector<8>>(context, builder);
    }
    op = builder.createIntegerConstant(loc, i32Ty, CR6_EQ_REV);
    cmpArgs.emplace_back(op);
    cmpArgs.emplace_back(argBases[0]);
    cmpArgs.emplace_back(argBases[1]);
  }
  assert((!fname.empty() && ftype) && "invalid type");

````
- **L1321 EN**: Executes a call or declaration centered on `std::get<1>`.
  **L1321 CN**: 执行以 `std::get<1>` 为核心的调用或声明。
- **L1322 EN**: Blank line separating nearby declarations or logic blocks.
  **L1322 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1323 EN**: Executes a call or declaration centered on `builder.createIntegerConstant`.
  **L1323 CN**: 执行以 `builder.createIntegerConstant` 为核心的调用或声明。
- **L1324 EN**: Executes a call or declaration centered on `cmpArgs.emplace_back`.
  **L1324 CN**: 执行以 `cmpArgs.emplace_back` 为核心的调用或声明。
- **L1325 EN**: Comment explains nearby logic, intent, or metadata: `reverse the argument order`.
  **L1325 CN**: 注释说明附近代码的逻辑、意图或元数据：`reverse the argument order`。
- **L1326 EN**: Executes a call or declaration centered on `cmpArgs.emplace_back`.
  **L1326 CN**: 执行以 `cmpArgs.emplace_back` 为核心的调用或声明。
- **L1327 EN**: Executes a call or declaration centered on `cmpArgs.emplace_back`.
  **L1327 CN**: 执行以 `cmpArgs.emplace_back` 为核心的调用或声明。
- **L1328 EN**: Transitions from the previous branch into an `else if` condition.
  **L1328 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1329 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1329 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1330 EN**: Executes a standalone statement or declaration: `fname = "llvm.ppc.vsx.xvcmpgesp.p";`.
  **L1330 CN**: 执行一条独立语句或声明：`fname = "llvm.ppc.vsx.xvcmpgesp.p";`。
- **L1331 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ftype = genFuncType<Ty::Integer<4>, Ty::Integer<4>, Ty::RealVector<4>,`.
  **L1331 CN**: 继续一个多行参数列表、初始化器或聚合项：`ftype = genFuncType<Ty::Integer<4>, Ty::Integer<4>, Ty::RealVector<4>,`。
- **L1332 EN**: Executes a call or declaration centered on `Ty::RealVector<4>>`.
  **L1332 CN**: 执行以 `Ty::RealVector<4>>` 为核心的调用或声明。
- **L1333 EN**: Transitions from the previous branch into the alternative path.
  **L1333 CN**: 从前一个分支过渡到备选路径。
- **L1334 EN**: Executes a standalone statement or declaration: `fname = "llvm.ppc.vsx.xvcmpgedp.p";`.
  **L1334 CN**: 执行一条独立语句或声明：`fname = "llvm.ppc.vsx.xvcmpgedp.p";`。
- **L1335 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ftype = genFuncType<Ty::Integer<4>, Ty::Integer<4>, Ty::RealVector<8>,`.
  **L1335 CN**: 继续一个多行参数列表、初始化器或聚合项：`ftype = genFuncType<Ty::Integer<4>, Ty::Integer<4>, Ty::RealVector<8>,`。
- **L1336 EN**: Executes a call or declaration centered on `Ty::RealVector<8>>`.
  **L1336 CN**: 执行以 `Ty::RealVector<8>>` 为核心的调用或声明。
- **L1337 EN**: Closes the current lexical scope or compound statement.
  **L1337 CN**: 结束当前词法作用域或复合语句块。
- **L1338 EN**: Executes a call or declaration centered on `builder.createIntegerConstant`.
  **L1338 CN**: 执行以 `builder.createIntegerConstant` 为核心的调用或声明。
- **L1339 EN**: Executes a call or declaration centered on `cmpArgs.emplace_back`.
  **L1339 CN**: 执行以 `cmpArgs.emplace_back` 为核心的调用或声明。
- **L1340 EN**: Executes a call or declaration centered on `cmpArgs.emplace_back`.
  **L1340 CN**: 执行以 `cmpArgs.emplace_back` 为核心的调用或声明。
- **L1341 EN**: Executes a call or declaration centered on `cmpArgs.emplace_back`.
  **L1341 CN**: 执行以 `cmpArgs.emplace_back` 为核心的调用或声明。
- **L1342 EN**: Closes the current lexical scope or compound statement.
  **L1342 CN**: 结束当前词法作用域或复合语句块。
- **L1343 EN**: Checks an internal invariant in debug builds.
  **L1343 CN**: 在调试构建中检查内部不变式。
- **L1344 EN**: Blank line separating nearby declarations or logic blocks.
  **L1344 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 1345-1368

````cpp
  mlir::func::FuncOp funcOp{builder.createFunction(loc, fname, ftype)};
  auto callOp{fir::CallOp::create(builder, loc, funcOp, cmpArgs)};
  return callOp.getResult(0);
}

static std::pair<llvm::StringRef, mlir::FunctionType>
getVecCmpFuncTypeAndName(VecTypeInfo &vTypeInfo, VecOp vop,
                         fir::FirOpBuilder &builder) {
  auto context{builder.getContext()};
  static std::map<std::pair<ParamTypeId, unsigned>,
                  std::pair<llvm::StringRef, mlir::FunctionType>>
      iuBuiltinName{
          {std::make_pair(ParamTypeId::IntegerVector, 8),
           std::make_pair(
               "llvm.ppc.altivec.vcmpgtsb",
               genFuncType<Ty::UnsignedVector<1>, Ty::IntegerVector<1>,
                           Ty::IntegerVector<1>>(context, builder))},
          {std::make_pair(ParamTypeId::IntegerVector, 16),
           std::make_pair(
               "llvm.ppc.altivec.vcmpgtsh",
               genFuncType<Ty::UnsignedVector<2>, Ty::IntegerVector<2>,
                           Ty::IntegerVector<2>>(context, builder))},
          {std::make_pair(ParamTypeId::IntegerVector, 32),
           std::make_pair(
````
- **L1345 EN**: Executes a call or declaration centered on `funcOp{builder.createFunction`.
  **L1345 CN**: 执行以 `funcOp{builder.createFunction` 为核心的调用或声明。
- **L1346 EN**: Executes a call or declaration centered on `callOp{fir::CallOp::create`.
  **L1346 CN**: 执行以 `callOp{fir::CallOp::create` 为核心的调用或声明。
- **L1347 EN**: Returns from the current function with `callOp.getResult(0)`.
  **L1347 CN**: 以 `callOp.getResult(0)` 从当前函数返回。
- **L1348 EN**: Closes the current lexical scope or compound statement.
  **L1348 CN**: 结束当前词法作用域或复合语句块。
- **L1349 EN**: Blank line separating nearby declarations or logic blocks.
  **L1349 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1350 EN**: Continues the surrounding expression or declaration: `static std::pair<llvm::StringRef, mlir::FunctionType>`.
  **L1350 CN**: 继续构造周围的表达式或声明：`static std::pair<llvm::StringRef, mlir::FunctionType>`。
- **L1351 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `getVecCmpFuncTypeAndName(VecTypeInfo &vTypeInfo, VecOp vop,`.
  **L1351 CN**: 继续一个多行参数列表、初始化器或聚合项：`getVecCmpFuncTypeAndName(VecTypeInfo &vTypeInfo, VecOp vop,`。
- **L1352 EN**: Continues the surrounding expression or declaration: `fir::FirOpBuilder &builder) {`.
  **L1352 CN**: 继续构造周围的表达式或声明：`fir::FirOpBuilder &builder) {`。
- **L1353 EN**: Executes a call or declaration centered on `context{builder.getContext`.
  **L1353 CN**: 执行以 `context{builder.getContext` 为核心的调用或声明。
- **L1354 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static std::map<std::pair<ParamTypeId, unsigned>,`.
  **L1354 CN**: 继续一个多行参数列表、初始化器或聚合项：`static std::map<std::pair<ParamTypeId, unsigned>,`。
- **L1355 EN**: Continues the surrounding expression or declaration: `std::pair<llvm::StringRef, mlir::FunctionType>>`.
  **L1355 CN**: 继续构造周围的表达式或声明：`std::pair<llvm::StringRef, mlir::FunctionType>>`。
- **L1356 EN**: Continues the surrounding expression or declaration: `iuBuiltinName{`.
  **L1356 CN**: 继续构造周围的表达式或声明：`iuBuiltinName{`。
- **L1357 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{std::make_pair(ParamTypeId::IntegerVector, 8),`.
  **L1357 CN**: 继续一个多行参数列表、初始化器或聚合项：`{std::make_pair(ParamTypeId::IntegerVector, 8),`。
- **L1358 EN**: Continues logic associated with callable symbol `make_pair`.
  **L1358 CN**: 继续与可调用符号 `make_pair` 相关的逻辑。
- **L1359 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"llvm.ppc.altivec.vcmpgtsb",`.
  **L1359 CN**: 继续一个多行参数列表、初始化器或聚合项：`"llvm.ppc.altivec.vcmpgtsb",`。
- **L1360 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genFuncType<Ty::UnsignedVector<1>, Ty::IntegerVector<1>,`.
  **L1360 CN**: 继续一个多行参数列表、初始化器或聚合项：`genFuncType<Ty::UnsignedVector<1>, Ty::IntegerVector<1>,`。
- **L1361 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Ty::IntegerVector<1>>(context, builder))},`.
  **L1361 CN**: 继续一个多行参数列表、初始化器或聚合项：`Ty::IntegerVector<1>>(context, builder))},`。
- **L1362 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{std::make_pair(ParamTypeId::IntegerVector, 16),`.
  **L1362 CN**: 继续一个多行参数列表、初始化器或聚合项：`{std::make_pair(ParamTypeId::IntegerVector, 16),`。
- **L1363 EN**: Continues logic associated with callable symbol `make_pair`.
  **L1363 CN**: 继续与可调用符号 `make_pair` 相关的逻辑。
- **L1364 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"llvm.ppc.altivec.vcmpgtsh",`.
  **L1364 CN**: 继续一个多行参数列表、初始化器或聚合项：`"llvm.ppc.altivec.vcmpgtsh",`。
- **L1365 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genFuncType<Ty::UnsignedVector<2>, Ty::IntegerVector<2>,`.
  **L1365 CN**: 继续一个多行参数列表、初始化器或聚合项：`genFuncType<Ty::UnsignedVector<2>, Ty::IntegerVector<2>,`。
- **L1366 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Ty::IntegerVector<2>>(context, builder))},`.
  **L1366 CN**: 继续一个多行参数列表、初始化器或聚合项：`Ty::IntegerVector<2>>(context, builder))},`。
- **L1367 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{std::make_pair(ParamTypeId::IntegerVector, 32),`.
  **L1367 CN**: 继续一个多行参数列表、初始化器或聚合项：`{std::make_pair(ParamTypeId::IntegerVector, 32),`。
- **L1368 EN**: Continues logic associated with callable symbol `make_pair`.
  **L1368 CN**: 继续与可调用符号 `make_pair` 相关的逻辑。

### Lines 1369-1392

````cpp
               "llvm.ppc.altivec.vcmpgtsw",
               genFuncType<Ty::UnsignedVector<4>, Ty::IntegerVector<4>,
                           Ty::IntegerVector<4>>(context, builder))},
          {std::make_pair(ParamTypeId::IntegerVector, 64),
           std::make_pair(
               "llvm.ppc.altivec.vcmpgtsd",
               genFuncType<Ty::UnsignedVector<8>, Ty::IntegerVector<8>,
                           Ty::IntegerVector<8>>(context, builder))},
          {std::make_pair(ParamTypeId::UnsignedVector, 8),
           std::make_pair(
               "llvm.ppc.altivec.vcmpgtub",
               genFuncType<Ty::UnsignedVector<1>, Ty::UnsignedVector<1>,
                           Ty::UnsignedVector<1>>(context, builder))},
          {std::make_pair(ParamTypeId::UnsignedVector, 16),
           std::make_pair(
               "llvm.ppc.altivec.vcmpgtuh",
               genFuncType<Ty::UnsignedVector<2>, Ty::UnsignedVector<2>,
                           Ty::UnsignedVector<2>>(context, builder))},
          {std::make_pair(ParamTypeId::UnsignedVector, 32),
           std::make_pair(
               "llvm.ppc.altivec.vcmpgtuw",
               genFuncType<Ty::UnsignedVector<4>, Ty::UnsignedVector<4>,
                           Ty::UnsignedVector<4>>(context, builder))},
          {std::make_pair(ParamTypeId::UnsignedVector, 64),
````
- **L1369 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"llvm.ppc.altivec.vcmpgtsw",`.
  **L1369 CN**: 继续一个多行参数列表、初始化器或聚合项：`"llvm.ppc.altivec.vcmpgtsw",`。
- **L1370 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genFuncType<Ty::UnsignedVector<4>, Ty::IntegerVector<4>,`.
  **L1370 CN**: 继续一个多行参数列表、初始化器或聚合项：`genFuncType<Ty::UnsignedVector<4>, Ty::IntegerVector<4>,`。
- **L1371 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Ty::IntegerVector<4>>(context, builder))},`.
  **L1371 CN**: 继续一个多行参数列表、初始化器或聚合项：`Ty::IntegerVector<4>>(context, builder))},`。
- **L1372 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{std::make_pair(ParamTypeId::IntegerVector, 64),`.
  **L1372 CN**: 继续一个多行参数列表、初始化器或聚合项：`{std::make_pair(ParamTypeId::IntegerVector, 64),`。
- **L1373 EN**: Continues logic associated with callable symbol `make_pair`.
  **L1373 CN**: 继续与可调用符号 `make_pair` 相关的逻辑。
- **L1374 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"llvm.ppc.altivec.vcmpgtsd",`.
  **L1374 CN**: 继续一个多行参数列表、初始化器或聚合项：`"llvm.ppc.altivec.vcmpgtsd",`。
- **L1375 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genFuncType<Ty::UnsignedVector<8>, Ty::IntegerVector<8>,`.
  **L1375 CN**: 继续一个多行参数列表、初始化器或聚合项：`genFuncType<Ty::UnsignedVector<8>, Ty::IntegerVector<8>,`。
- **L1376 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Ty::IntegerVector<8>>(context, builder))},`.
  **L1376 CN**: 继续一个多行参数列表、初始化器或聚合项：`Ty::IntegerVector<8>>(context, builder))},`。
- **L1377 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{std::make_pair(ParamTypeId::UnsignedVector, 8),`.
  **L1377 CN**: 继续一个多行参数列表、初始化器或聚合项：`{std::make_pair(ParamTypeId::UnsignedVector, 8),`。
- **L1378 EN**: Continues logic associated with callable symbol `make_pair`.
  **L1378 CN**: 继续与可调用符号 `make_pair` 相关的逻辑。
- **L1379 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"llvm.ppc.altivec.vcmpgtub",`.
  **L1379 CN**: 继续一个多行参数列表、初始化器或聚合项：`"llvm.ppc.altivec.vcmpgtub",`。
- **L1380 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genFuncType<Ty::UnsignedVector<1>, Ty::UnsignedVector<1>,`.
  **L1380 CN**: 继续一个多行参数列表、初始化器或聚合项：`genFuncType<Ty::UnsignedVector<1>, Ty::UnsignedVector<1>,`。
- **L1381 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Ty::UnsignedVector<1>>(context, builder))},`.
  **L1381 CN**: 继续一个多行参数列表、初始化器或聚合项：`Ty::UnsignedVector<1>>(context, builder))},`。
- **L1382 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{std::make_pair(ParamTypeId::UnsignedVector, 16),`.
  **L1382 CN**: 继续一个多行参数列表、初始化器或聚合项：`{std::make_pair(ParamTypeId::UnsignedVector, 16),`。
- **L1383 EN**: Continues logic associated with callable symbol `make_pair`.
  **L1383 CN**: 继续与可调用符号 `make_pair` 相关的逻辑。
- **L1384 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"llvm.ppc.altivec.vcmpgtuh",`.
  **L1384 CN**: 继续一个多行参数列表、初始化器或聚合项：`"llvm.ppc.altivec.vcmpgtuh",`。
- **L1385 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genFuncType<Ty::UnsignedVector<2>, Ty::UnsignedVector<2>,`.
  **L1385 CN**: 继续一个多行参数列表、初始化器或聚合项：`genFuncType<Ty::UnsignedVector<2>, Ty::UnsignedVector<2>,`。
- **L1386 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Ty::UnsignedVector<2>>(context, builder))},`.
  **L1386 CN**: 继续一个多行参数列表、初始化器或聚合项：`Ty::UnsignedVector<2>>(context, builder))},`。
- **L1387 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{std::make_pair(ParamTypeId::UnsignedVector, 32),`.
  **L1387 CN**: 继续一个多行参数列表、初始化器或聚合项：`{std::make_pair(ParamTypeId::UnsignedVector, 32),`。
- **L1388 EN**: Continues logic associated with callable symbol `make_pair`.
  **L1388 CN**: 继续与可调用符号 `make_pair` 相关的逻辑。
- **L1389 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"llvm.ppc.altivec.vcmpgtuw",`.
  **L1389 CN**: 继续一个多行参数列表、初始化器或聚合项：`"llvm.ppc.altivec.vcmpgtuw",`。
- **L1390 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genFuncType<Ty::UnsignedVector<4>, Ty::UnsignedVector<4>,`.
  **L1390 CN**: 继续一个多行参数列表、初始化器或聚合项：`genFuncType<Ty::UnsignedVector<4>, Ty::UnsignedVector<4>,`。
- **L1391 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Ty::UnsignedVector<4>>(context, builder))},`.
  **L1391 CN**: 继续一个多行参数列表、初始化器或聚合项：`Ty::UnsignedVector<4>>(context, builder))},`。
- **L1392 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{std::make_pair(ParamTypeId::UnsignedVector, 64),`.
  **L1392 CN**: 继续一个多行参数列表、初始化器或聚合项：`{std::make_pair(ParamTypeId::UnsignedVector, 64),`。

### Lines 1393-1416

````cpp
           std::make_pair(
               "llvm.ppc.altivec.vcmpgtud",
               genFuncType<Ty::UnsignedVector<8>, Ty::UnsignedVector<8>,
                           Ty::UnsignedVector<8>>(context, builder))}};

  // VSX only defines GE and GT builtins. Cmple and Cmplt use GE and GT with
  // arguments revsered.
  enum class Cmp { gtOrLt, geOrLe };
  static std::map<std::pair<Cmp, int>,
                  std::pair<llvm::StringRef, mlir::FunctionType>>
      rGBI{{std::make_pair(Cmp::geOrLe, 32),
            std::make_pair("llvm.ppc.vsx.xvcmpgesp",
                           genFuncType<Ty::UnsignedVector<4>, Ty::RealVector<4>,
                                       Ty::RealVector<4>>(context, builder))},
           {std::make_pair(Cmp::geOrLe, 64),
            std::make_pair("llvm.ppc.vsx.xvcmpgedp",
                           genFuncType<Ty::UnsignedVector<8>, Ty::RealVector<8>,
                                       Ty::RealVector<8>>(context, builder))},
           {std::make_pair(Cmp::gtOrLt, 32),
            std::make_pair("llvm.ppc.vsx.xvcmpgtsp",
                           genFuncType<Ty::UnsignedVector<4>, Ty::RealVector<4>,
                                       Ty::RealVector<4>>(context, builder))},
           {std::make_pair(Cmp::gtOrLt, 64),
            std::make_pair("llvm.ppc.vsx.xvcmpgtdp",
````
- **L1393 EN**: Continues logic associated with callable symbol `make_pair`.
  **L1393 CN**: 继续与可调用符号 `make_pair` 相关的逻辑。
- **L1394 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"llvm.ppc.altivec.vcmpgtud",`.
  **L1394 CN**: 继续一个多行参数列表、初始化器或聚合项：`"llvm.ppc.altivec.vcmpgtud",`。
- **L1395 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genFuncType<Ty::UnsignedVector<8>, Ty::UnsignedVector<8>,`.
  **L1395 CN**: 继续一个多行参数列表、初始化器或聚合项：`genFuncType<Ty::UnsignedVector<8>, Ty::UnsignedVector<8>,`。
- **L1396 EN**: Executes a call or declaration centered on `Ty::UnsignedVector<8>>`.
  **L1396 CN**: 执行以 `Ty::UnsignedVector<8>>` 为核心的调用或声明。
- **L1397 EN**: Blank line separating nearby declarations or logic blocks.
  **L1397 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1398 EN**: Comment explains nearby logic, intent, or metadata: `VSX only defines GE and GT builtins. Cmple and Cmplt use GE and GT with`.
  **L1398 CN**: 注释说明附近代码的逻辑、意图或元数据：`VSX only defines GE and GT builtins. Cmple and Cmplt use GE and GT with`。
- **L1399 EN**: Comment explains nearby logic, intent, or metadata: `arguments revsered.`.
  **L1399 CN**: 注释说明附近代码的逻辑、意图或元数据：`arguments revsered.`。
- **L1400 EN**: Declares enum `class`.
  **L1400 CN**: 声明 enum `class`。
- **L1401 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static std::map<std::pair<Cmp, int>,`.
  **L1401 CN**: 继续一个多行参数列表、初始化器或聚合项：`static std::map<std::pair<Cmp, int>,`。
- **L1402 EN**: Continues the surrounding expression or declaration: `std::pair<llvm::StringRef, mlir::FunctionType>>`.
  **L1402 CN**: 继续构造周围的表达式或声明：`std::pair<llvm::StringRef, mlir::FunctionType>>`。
- **L1403 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rGBI{{std::make_pair(Cmp::geOrLe, 32),`.
  **L1403 CN**: 继续一个多行参数列表、初始化器或聚合项：`rGBI{{std::make_pair(Cmp::geOrLe, 32),`。
- **L1404 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `std::make_pair("llvm.ppc.vsx.xvcmpgesp",`.
  **L1404 CN**: 继续一个多行参数列表、初始化器或聚合项：`std::make_pair("llvm.ppc.vsx.xvcmpgesp",`。
- **L1405 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genFuncType<Ty::UnsignedVector<4>, Ty::RealVector<4>,`.
  **L1405 CN**: 继续一个多行参数列表、初始化器或聚合项：`genFuncType<Ty::UnsignedVector<4>, Ty::RealVector<4>,`。
- **L1406 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Ty::RealVector<4>>(context, builder))},`.
  **L1406 CN**: 继续一个多行参数列表、初始化器或聚合项：`Ty::RealVector<4>>(context, builder))},`。
- **L1407 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{std::make_pair(Cmp::geOrLe, 64),`.
  **L1407 CN**: 继续一个多行参数列表、初始化器或聚合项：`{std::make_pair(Cmp::geOrLe, 64),`。
- **L1408 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `std::make_pair("llvm.ppc.vsx.xvcmpgedp",`.
  **L1408 CN**: 继续一个多行参数列表、初始化器或聚合项：`std::make_pair("llvm.ppc.vsx.xvcmpgedp",`。
- **L1409 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genFuncType<Ty::UnsignedVector<8>, Ty::RealVector<8>,`.
  **L1409 CN**: 继续一个多行参数列表、初始化器或聚合项：`genFuncType<Ty::UnsignedVector<8>, Ty::RealVector<8>,`。
- **L1410 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Ty::RealVector<8>>(context, builder))},`.
  **L1410 CN**: 继续一个多行参数列表、初始化器或聚合项：`Ty::RealVector<8>>(context, builder))},`。
- **L1411 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{std::make_pair(Cmp::gtOrLt, 32),`.
  **L1411 CN**: 继续一个多行参数列表、初始化器或聚合项：`{std::make_pair(Cmp::gtOrLt, 32),`。
- **L1412 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `std::make_pair("llvm.ppc.vsx.xvcmpgtsp",`.
  **L1412 CN**: 继续一个多行参数列表、初始化器或聚合项：`std::make_pair("llvm.ppc.vsx.xvcmpgtsp",`。
- **L1413 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genFuncType<Ty::UnsignedVector<4>, Ty::RealVector<4>,`.
  **L1413 CN**: 继续一个多行参数列表、初始化器或聚合项：`genFuncType<Ty::UnsignedVector<4>, Ty::RealVector<4>,`。
- **L1414 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Ty::RealVector<4>>(context, builder))},`.
  **L1414 CN**: 继续一个多行参数列表、初始化器或聚合项：`Ty::RealVector<4>>(context, builder))},`。
- **L1415 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{std::make_pair(Cmp::gtOrLt, 64),`.
  **L1415 CN**: 继续一个多行参数列表、初始化器或聚合项：`{std::make_pair(Cmp::gtOrLt, 64),`。
- **L1416 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `std::make_pair("llvm.ppc.vsx.xvcmpgtdp",`.
  **L1416 CN**: 继续一个多行参数列表、初始化器或聚合项：`std::make_pair("llvm.ppc.vsx.xvcmpgtdp",`。

### Lines 1417-1440

````cpp
                           genFuncType<Ty::UnsignedVector<8>, Ty::RealVector<8>,
                                       Ty::RealVector<8>>(context, builder))}};

  const auto width{vTypeInfo.eleTy.getIntOrFloatBitWidth()};
  std::pair<llvm::StringRef, mlir::FunctionType> specFunc;
  if (auto elementTy = mlir::dyn_cast<mlir::IntegerType>(vTypeInfo.eleTy))
    specFunc =
        (elementTy.isUnsignedInteger())
            ? iuBuiltinName[std::make_pair(ParamTypeId::UnsignedVector, width)]
            : iuBuiltinName[std::make_pair(ParamTypeId::IntegerVector, width)];
  else if (vTypeInfo.isFloat())
    specFunc = (vop == VecOp::Cmpge || vop == VecOp::Cmple)
                   ? rGBI[std::make_pair(Cmp::geOrLe, width)]
                   : rGBI[std::make_pair(Cmp::gtOrLt, width)];

  assert(!std::get<0>(specFunc).empty() && "unknown builtin name");
  assert(std::get<1>(specFunc) && "unknown function type");
  return specFunc;
}

// VEC_CMPGE, VEC_CMPGT, VEC_CMPLE, VEC_CMPLT
template <VecOp vop>
fir::ExtendedValue
PPCIntrinsicLibrary::genVecCmp(mlir::Type resultType,
````
- **L1417 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genFuncType<Ty::UnsignedVector<8>, Ty::RealVector<8>,`.
  **L1417 CN**: 继续一个多行参数列表、初始化器或聚合项：`genFuncType<Ty::UnsignedVector<8>, Ty::RealVector<8>,`。
- **L1418 EN**: Executes a call or declaration centered on `Ty::RealVector<8>>`.
  **L1418 CN**: 执行以 `Ty::RealVector<8>>` 为核心的调用或声明。
- **L1419 EN**: Blank line separating nearby declarations or logic blocks.
  **L1419 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1420 EN**: Executes a call or declaration centered on `width{vTypeInfo.eleTy.getIntOrFloatBitWidth`.
  **L1420 CN**: 执行以 `width{vTypeInfo.eleTy.getIntOrFloatBitWidth` 为核心的调用或声明。
- **L1421 EN**: Executes a standalone statement or declaration: `std::pair<llvm::StringRef, mlir::FunctionType> specFunc;`.
  **L1421 CN**: 执行一条独立语句或声明：`std::pair<llvm::StringRef, mlir::FunctionType> specFunc;`。
- **L1422 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1422 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1423 EN**: Continues the surrounding expression or declaration: `specFunc =`.
  **L1423 CN**: 继续构造周围的表达式或声明：`specFunc =`。
- **L1424 EN**: Continues logic associated with callable symbol `isUnsignedInteger`.
  **L1424 CN**: 继续与可调用符号 `isUnsignedInteger` 相关的逻辑。
- **L1425 EN**: Continues logic associated with callable symbol `make_pair`.
  **L1425 CN**: 继续与可调用符号 `make_pair` 相关的逻辑。
- **L1426 EN**: Executes a call or declaration centered on `iuBuiltinName[std::make_pair`.
  **L1426 CN**: 执行以 `iuBuiltinName[std::make_pair` 为核心的调用或声明。
- **L1427 EN**: Starts the alternative branch of the preceding conditional.
  **L1427 CN**: 开始前一个条件语句的备选分支。
- **L1428 EN**: Continues the surrounding expression or declaration: `specFunc = (vop == VecOp::Cmpge || vop == VecOp::Cmple)`.
  **L1428 CN**: 继续构造周围的表达式或声明：`specFunc = (vop == VecOp::Cmpge || vop == VecOp::Cmple)`。
- **L1429 EN**: Continues logic associated with callable symbol `make_pair`.
  **L1429 CN**: 继续与可调用符号 `make_pair` 相关的逻辑。
- **L1430 EN**: Executes a call or declaration centered on `rGBI[std::make_pair`.
  **L1430 CN**: 执行以 `rGBI[std::make_pair` 为核心的调用或声明。
- **L1431 EN**: Blank line separating nearby declarations or logic blocks.
  **L1431 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1432 EN**: Checks an internal invariant in debug builds.
  **L1432 CN**: 在调试构建中检查内部不变式。
- **L1433 EN**: Checks an internal invariant in debug builds.
  **L1433 CN**: 在调试构建中检查内部不变式。
- **L1434 EN**: Returns from the current function with `specFunc`.
  **L1434 CN**: 以 `specFunc` 从当前函数返回。
- **L1435 EN**: Closes the current lexical scope or compound statement.
  **L1435 CN**: 结束当前词法作用域或复合语句块。
- **L1436 EN**: Blank line separating nearby declarations or logic blocks.
  **L1436 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1437 EN**: Comment explains nearby logic, intent, or metadata: `VEC_CMPGE, VEC_CMPGT, VEC_CMPLE, VEC_CMPLT`.
  **L1437 CN**: 注释说明附近代码的逻辑、意图或元数据：`VEC_CMPGE, VEC_CMPGT, VEC_CMPLE, VEC_CMPLT`。
- **L1438 EN**: Introduces template parameters or specialization context: `template <VecOp vop>`.
  **L1438 CN**: 为后续声明引入模板参数或特化上下文：`template <VecOp vop>`。
- **L1439 EN**: Continues the surrounding expression or declaration: `fir::ExtendedValue`.
  **L1439 CN**: 继续构造周围的表达式或声明：`fir::ExtendedValue`。
- **L1440 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PPCIntrinsicLibrary::genVecCmp(mlir::Type resultType,`.
  **L1440 CN**: 继续一个多行参数列表、初始化器或聚合项：`PPCIntrinsicLibrary::genVecCmp(mlir::Type resultType,`。

### Lines 1441-1464

````cpp
                               llvm::ArrayRef<fir::ExtendedValue> args) {
  assert(args.size() == 2);
  auto context{builder.getContext()};
  auto argBases{getBasesForArgs(args)};
  VecTypeInfo vecTyInfo{getVecTypeFromFir(argBases[0])};
  auto varg{convertVecArgs(builder, loc, vecTyInfo, argBases)};

  std::pair<llvm::StringRef, mlir::FunctionType> funcTyNam{
      getVecCmpFuncTypeAndName(vecTyInfo, vop, builder)};

  mlir::func::FuncOp funcOp = builder.createFunction(
      loc, std::get<0>(funcTyNam), std::get<1>(funcTyNam));

  mlir::Value res{nullptr};

  if (auto eTy = mlir::dyn_cast<mlir::IntegerType>(vecTyInfo.eleTy)) {
    constexpr int firstArg{0};
    constexpr int secondArg{1};
    std::map<VecOp, std::array<int, 2>> argOrder{
        {VecOp::Cmpge, {secondArg, firstArg}},
        {VecOp::Cmple, {firstArg, secondArg}},
        {VecOp::Cmpgt, {firstArg, secondArg}},
        {VecOp::Cmplt, {secondArg, firstArg}}};

````
- **L1441 EN**: Continues the surrounding expression or declaration: `llvm::ArrayRef<fir::ExtendedValue> args) {`.
  **L1441 CN**: 继续构造周围的表达式或声明：`llvm::ArrayRef<fir::ExtendedValue> args) {`。
- **L1442 EN**: Checks an internal invariant in debug builds.
  **L1442 CN**: 在调试构建中检查内部不变式。
- **L1443 EN**: Executes a call or declaration centered on `context{builder.getContext`.
  **L1443 CN**: 执行以 `context{builder.getContext` 为核心的调用或声明。
- **L1444 EN**: Executes a call or declaration centered on `argBases{getBasesForArgs`.
  **L1444 CN**: 执行以 `argBases{getBasesForArgs` 为核心的调用或声明。
- **L1445 EN**: Executes a call or declaration centered on `vecTyInfo{getVecTypeFromFir`.
  **L1445 CN**: 执行以 `vecTyInfo{getVecTypeFromFir` 为核心的调用或声明。
- **L1446 EN**: Executes a call or declaration centered on `varg{convertVecArgs`.
  **L1446 CN**: 执行以 `varg{convertVecArgs` 为核心的调用或声明。
- **L1447 EN**: Blank line separating nearby declarations or logic blocks.
  **L1447 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1448 EN**: Continues the surrounding expression or declaration: `std::pair<llvm::StringRef, mlir::FunctionType> funcTyNam{`.
  **L1448 CN**: 继续构造周围的表达式或声明：`std::pair<llvm::StringRef, mlir::FunctionType> funcTyNam{`。
- **L1449 EN**: Executes a call or declaration centered on `getVecCmpFuncTypeAndName`.
  **L1449 CN**: 执行以 `getVecCmpFuncTypeAndName` 为核心的调用或声明。
- **L1450 EN**: Blank line separating nearby declarations or logic blocks.
  **L1450 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1451 EN**: Continues logic associated with callable symbol `createFunction`.
  **L1451 CN**: 继续与可调用符号 `createFunction` 相关的逻辑。
- **L1452 EN**: Executes a call or declaration centered on `std::get<0>`.
  **L1452 CN**: 执行以 `std::get<0>` 为核心的调用或声明。
- **L1453 EN**: Blank line separating nearby declarations or logic blocks.
  **L1453 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1454 EN**: Executes a standalone statement or declaration: `mlir::Value res{nullptr};`.
  **L1454 CN**: 执行一条独立语句或声明：`mlir::Value res{nullptr};`。
- **L1455 EN**: Blank line separating nearby declarations or logic blocks.
  **L1455 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1456 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1456 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1457 EN**: Executes a standalone statement or declaration: `constexpr int firstArg{0};`.
  **L1457 CN**: 执行一条独立语句或声明：`constexpr int firstArg{0};`。
- **L1458 EN**: Executes a standalone statement or declaration: `constexpr int secondArg{1};`.
  **L1458 CN**: 执行一条独立语句或声明：`constexpr int secondArg{1};`。
- **L1459 EN**: Continues the surrounding expression or declaration: `std::map<VecOp, std::array<int, 2>> argOrder{`.
  **L1459 CN**: 继续构造周围的表达式或声明：`std::map<VecOp, std::array<int, 2>> argOrder{`。
- **L1460 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{VecOp::Cmpge, {secondArg, firstArg}},`.
  **L1460 CN**: 继续一个多行参数列表、初始化器或聚合项：`{VecOp::Cmpge, {secondArg, firstArg}},`。
- **L1461 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{VecOp::Cmple, {firstArg, secondArg}},`.
  **L1461 CN**: 继续一个多行参数列表、初始化器或聚合项：`{VecOp::Cmple, {firstArg, secondArg}},`。
- **L1462 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{VecOp::Cmpgt, {firstArg, secondArg}},`.
  **L1462 CN**: 继续一个多行参数列表、初始化器或聚合项：`{VecOp::Cmpgt, {firstArg, secondArg}},`。
- **L1463 EN**: Executes a standalone statement or declaration: `{VecOp::Cmplt, {secondArg, firstArg}}};`.
  **L1463 CN**: 执行一条独立语句或声明：`{VecOp::Cmplt, {secondArg, firstArg}}};`。
- **L1464 EN**: Blank line separating nearby declarations or logic blocks.
  **L1464 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 1465-1488

````cpp
    // Construct the function return type, unsigned vector, for conversion.
    auto itype = mlir::IntegerType::get(context, eTy.getWidth(),
                                        mlir::IntegerType::Unsigned);
    auto returnType = fir::VectorType::get(vecTyInfo.len, itype);

    switch (vop) {
    case VecOp::Cmpgt:
    case VecOp::Cmplt: {
      // arg1 > arg2 --> vcmpgt(arg1, arg2)
      // arg1 < arg2 --> vcmpgt(arg2, arg1)
      mlir::Value vargs[]{argBases[argOrder[vop][0]],
                          argBases[argOrder[vop][1]]};
      auto callOp{fir::CallOp::create(builder, loc, funcOp, vargs)};
      res = callOp.getResult(0);
      break;
    }
    case VecOp::Cmpge:
    case VecOp::Cmple: {
      // arg1 >= arg2 --> vcmpge(arg2, arg1) xor vector(-1)
      // arg1 <= arg2 --> vcmpge(arg1, arg2) xor vector(-1)
      mlir::Value vargs[]{argBases[argOrder[vop][0]],
                          argBases[argOrder[vop][1]]};

      // Construct a constant vector(-1)
````
- **L1465 EN**: Comment explains nearby logic, intent, or metadata: `Construct the function return type, unsigned vector, for conversion.`.
  **L1465 CN**: 注释说明附近代码的逻辑、意图或元数据：`Construct the function return type, unsigned vector, for conversion.`。
- **L1466 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto itype = mlir::IntegerType::get(context, eTy.getWidth(),`.
  **L1466 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto itype = mlir::IntegerType::get(context, eTy.getWidth(),`。
- **L1467 EN**: Executes a standalone statement or declaration: `mlir::IntegerType::Unsigned);`.
  **L1467 CN**: 执行一条独立语句或声明：`mlir::IntegerType::Unsigned);`。
- **L1468 EN**: Initializes variable `returnType` from the right-hand expression.
  **L1468 CN**: 使用右侧表达式初始化变量 `returnType`。
- **L1469 EN**: Blank line separating nearby declarations or logic blocks.
  **L1469 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1470 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L1470 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L1471 EN**: Introduces a switch dispatch label: `case VecOp::Cmpgt:`.
  **L1471 CN**: 引入一个 switch 分发标签：`case VecOp::Cmpgt:`。
- **L1472 EN**: Introduces a switch dispatch label: `case VecOp::Cmplt: {`.
  **L1472 CN**: 引入一个 switch 分发标签：`case VecOp::Cmplt: {`。
- **L1473 EN**: Comment explains nearby logic, intent, or metadata: `arg1 > arg2 --> vcmpgt(arg1, arg2)`.
  **L1473 CN**: 注释说明附近代码的逻辑、意图或元数据：`arg1 > arg2 --> vcmpgt(arg1, arg2)`。
- **L1474 EN**: Comment explains nearby logic, intent, or metadata: `arg1 < arg2 --> vcmpgt(arg2, arg1)`.
  **L1474 CN**: 注释说明附近代码的逻辑、意图或元数据：`arg1 < arg2 --> vcmpgt(arg2, arg1)`。
- **L1475 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Value vargs[]{argBases[argOrder[vop][0]],`.
  **L1475 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Value vargs[]{argBases[argOrder[vop][0]],`。
- **L1476 EN**: Executes a standalone statement or declaration: `argBases[argOrder[vop][1]]};`.
  **L1476 CN**: 执行一条独立语句或声明：`argBases[argOrder[vop][1]]};`。
- **L1477 EN**: Executes a call or declaration centered on `callOp{fir::CallOp::create`.
  **L1477 CN**: 执行以 `callOp{fir::CallOp::create` 为核心的调用或声明。
- **L1478 EN**: Executes a call or declaration centered on `callOp.getResult`.
  **L1478 CN**: 执行以 `callOp.getResult` 为核心的调用或声明。
- **L1479 EN**: Exits the nearest loop or switch statement.
  **L1479 CN**: 退出最近的循环或 switch 语句。
- **L1480 EN**: Closes the current lexical scope or compound statement.
  **L1480 CN**: 结束当前词法作用域或复合语句块。
- **L1481 EN**: Introduces a switch dispatch label: `case VecOp::Cmpge:`.
  **L1481 CN**: 引入一个 switch 分发标签：`case VecOp::Cmpge:`。
- **L1482 EN**: Introduces a switch dispatch label: `case VecOp::Cmple: {`.
  **L1482 CN**: 引入一个 switch 分发标签：`case VecOp::Cmple: {`。
- **L1483 EN**: Comment explains nearby logic, intent, or metadata: `arg1 >= arg2 --> vcmpge(arg2, arg1) xor vector(-1)`.
  **L1483 CN**: 注释说明附近代码的逻辑、意图或元数据：`arg1 >= arg2 --> vcmpge(arg2, arg1) xor vector(-1)`。
- **L1484 EN**: Comment explains nearby logic, intent, or metadata: `arg1 <= arg2 --> vcmpge(arg1, arg2) xor vector(-1)`.
  **L1484 CN**: 注释说明附近代码的逻辑、意图或元数据：`arg1 <= arg2 --> vcmpge(arg1, arg2) xor vector(-1)`。
- **L1485 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Value vargs[]{argBases[argOrder[vop][0]],`.
  **L1485 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Value vargs[]{argBases[argOrder[vop][0]],`。
- **L1486 EN**: Executes a standalone statement or declaration: `argBases[argOrder[vop][1]]};`.
  **L1486 CN**: 执行一条独立语句或声明：`argBases[argOrder[vop][1]]};`。
- **L1487 EN**: Blank line separating nearby declarations or logic blocks.
  **L1487 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1488 EN**: Comment explains nearby logic, intent, or metadata: `Construct a constant vector(-1)`.
  **L1488 CN**: 注释说明附近代码的逻辑、意图或元数据：`Construct a constant vector(-1)`。

### Lines 1489-1512

````cpp
      auto negOneVal{builder.createIntegerConstant(
          loc, getConvertedElementType(context, eTy), -1)};
      auto vNegOne{mlir::vector::BroadcastOp::create(
          builder, loc, vecTyInfo.toMlirVectorType(context), negOneVal)};

      auto callOp{fir::CallOp::create(builder, loc, funcOp, vargs)};
      mlir::Value callRes{callOp.getResult(0)};
      auto vargs2{
          convertVecArgs(builder, loc, vecTyInfo, mlir::ValueRange{callRes})};
      auto xorRes{
          mlir::arith::XOrIOp::create(builder, loc, vargs2[0], vNegOne)};

      res = builder.createConvert(loc, returnType, xorRes);
      break;
    }
    default:
      llvm_unreachable("Invalid vector operation for generator");
    }
  } else if (vecTyInfo.isFloat()) {
    mlir::Value vargs[2];
    switch (vop) {
    case VecOp::Cmpge:
    case VecOp::Cmpgt:
      vargs[0] = argBases[0];
````
- **L1489 EN**: Continues logic associated with callable symbol `createIntegerConstant`.
  **L1489 CN**: 继续与可调用符号 `createIntegerConstant` 相关的逻辑。
- **L1490 EN**: Executes a call or declaration centered on `getConvertedElementType`.
  **L1490 CN**: 执行以 `getConvertedElementType` 为核心的调用或声明。
- **L1491 EN**: Continues logic associated with callable symbol `create`.
  **L1491 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L1492 EN**: Executes a call or declaration centered on `vecTyInfo.toMlirVectorType`.
  **L1492 CN**: 执行以 `vecTyInfo.toMlirVectorType` 为核心的调用或声明。
- **L1493 EN**: Blank line separating nearby declarations or logic blocks.
  **L1493 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1494 EN**: Executes a call or declaration centered on `callOp{fir::CallOp::create`.
  **L1494 CN**: 执行以 `callOp{fir::CallOp::create` 为核心的调用或声明。
- **L1495 EN**: Executes a call or declaration centered on `callRes{callOp.getResult`.
  **L1495 CN**: 执行以 `callRes{callOp.getResult` 为核心的调用或声明。
- **L1496 EN**: Continues the surrounding expression or declaration: `auto vargs2{`.
  **L1496 CN**: 继续构造周围的表达式或声明：`auto vargs2{`。
- **L1497 EN**: Executes a call or declaration centered on `convertVecArgs`.
  **L1497 CN**: 执行以 `convertVecArgs` 为核心的调用或声明。
- **L1498 EN**: Continues the surrounding expression or declaration: `auto xorRes{`.
  **L1498 CN**: 继续构造周围的表达式或声明：`auto xorRes{`。
- **L1499 EN**: Executes a call or declaration centered on `mlir::arith::XOrIOp::create`.
  **L1499 CN**: 执行以 `mlir::arith::XOrIOp::create` 为核心的调用或声明。
- **L1500 EN**: Blank line separating nearby declarations or logic blocks.
  **L1500 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1501 EN**: Executes a call or declaration centered on `builder.createConvert`.
  **L1501 CN**: 执行以 `builder.createConvert` 为核心的调用或声明。
- **L1502 EN**: Exits the nearest loop or switch statement.
  **L1502 CN**: 退出最近的循环或 switch 语句。
- **L1503 EN**: Closes the current lexical scope or compound statement.
  **L1503 CN**: 结束当前词法作用域或复合语句块。
- **L1504 EN**: Introduces a switch dispatch label: `default:`.
  **L1504 CN**: 引入一个 switch 分发标签：`default:`。
- **L1505 EN**: Marks this control path as unreachable to LLVM.
  **L1505 CN**: 将该控制路径标记为 LLVM 认为不可达。
- **L1506 EN**: Closes the current lexical scope or compound statement.
  **L1506 CN**: 结束当前词法作用域或复合语句块。
- **L1507 EN**: Transitions from the previous branch into an `else if` condition.
  **L1507 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1508 EN**: Executes a standalone statement or declaration: `mlir::Value vargs[2];`.
  **L1508 CN**: 执行一条独立语句或声明：`mlir::Value vargs[2];`。
- **L1509 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L1509 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L1510 EN**: Introduces a switch dispatch label: `case VecOp::Cmpge:`.
  **L1510 CN**: 引入一个 switch 分发标签：`case VecOp::Cmpge:`。
- **L1511 EN**: Introduces a switch dispatch label: `case VecOp::Cmpgt:`.
  **L1511 CN**: 引入一个 switch 分发标签：`case VecOp::Cmpgt:`。
- **L1512 EN**: Executes a standalone statement or declaration: `vargs[0] = argBases[0];`.
  **L1512 CN**: 执行一条独立语句或声明：`vargs[0] = argBases[0];`。

### Lines 1513-1536

````cpp
      vargs[1] = argBases[1];
      break;
    case VecOp::Cmple:
    case VecOp::Cmplt:
      // Swap the arguments as xvcmpg[et] is used
      vargs[0] = argBases[1];
      vargs[1] = argBases[0];
      break;
    default:
      llvm_unreachable("Invalid vector operation for generator");
    }
    auto callOp{fir::CallOp::create(builder, loc, funcOp, vargs)};
    res = callOp.getResult(0);
  } else
    llvm_unreachable("invalid vector type");

  return res;
}

static inline mlir::Value swapVectorWordPairs(fir::FirOpBuilder &builder,
                                              mlir::Location loc,
                                              mlir::Value arg) {
  auto ty = arg.getType();
  auto context{builder.getContext()};
````
- **L1513 EN**: Executes a standalone statement or declaration: `vargs[1] = argBases[1];`.
  **L1513 CN**: 执行一条独立语句或声明：`vargs[1] = argBases[1];`。
- **L1514 EN**: Exits the nearest loop or switch statement.
  **L1514 CN**: 退出最近的循环或 switch 语句。
- **L1515 EN**: Introduces a switch dispatch label: `case VecOp::Cmple:`.
  **L1515 CN**: 引入一个 switch 分发标签：`case VecOp::Cmple:`。
- **L1516 EN**: Introduces a switch dispatch label: `case VecOp::Cmplt:`.
  **L1516 CN**: 引入一个 switch 分发标签：`case VecOp::Cmplt:`。
- **L1517 EN**: Comment explains nearby logic, intent, or metadata: `Swap the arguments as xvcmpg[et] is used`.
  **L1517 CN**: 注释说明附近代码的逻辑、意图或元数据：`Swap the arguments as xvcmpg[et] is used`。
- **L1518 EN**: Executes a standalone statement or declaration: `vargs[0] = argBases[1];`.
  **L1518 CN**: 执行一条独立语句或声明：`vargs[0] = argBases[1];`。
- **L1519 EN**: Executes a standalone statement or declaration: `vargs[1] = argBases[0];`.
  **L1519 CN**: 执行一条独立语句或声明：`vargs[1] = argBases[0];`。
- **L1520 EN**: Exits the nearest loop or switch statement.
  **L1520 CN**: 退出最近的循环或 switch 语句。
- **L1521 EN**: Introduces a switch dispatch label: `default:`.
  **L1521 CN**: 引入一个 switch 分发标签：`default:`。
- **L1522 EN**: Marks this control path as unreachable to LLVM.
  **L1522 CN**: 将该控制路径标记为 LLVM 认为不可达。
- **L1523 EN**: Closes the current lexical scope or compound statement.
  **L1523 CN**: 结束当前词法作用域或复合语句块。
- **L1524 EN**: Executes a call or declaration centered on `callOp{fir::CallOp::create`.
  **L1524 CN**: 执行以 `callOp{fir::CallOp::create` 为核心的调用或声明。
- **L1525 EN**: Executes a call or declaration centered on `callOp.getResult`.
  **L1525 CN**: 执行以 `callOp.getResult` 为核心的调用或声明。
- **L1526 EN**: Transitions from the previous branch into the alternative path.
  **L1526 CN**: 从前一个分支过渡到备选路径。
- **L1527 EN**: Marks this control path as unreachable to LLVM.
  **L1527 CN**: 将该控制路径标记为 LLVM 认为不可达。
- **L1528 EN**: Blank line separating nearby declarations or logic blocks.
  **L1528 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1529 EN**: Returns from the current function with `res`.
  **L1529 CN**: 以 `res` 从当前函数返回。
- **L1530 EN**: Closes the current lexical scope or compound statement.
  **L1530 CN**: 结束当前词法作用域或复合语句块。
- **L1531 EN**: Blank line separating nearby declarations or logic blocks.
  **L1531 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1532 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static inline mlir::Value swapVectorWordPairs(fir::FirOpBuilder &builder,`.
  **L1532 CN**: 继续一个多行参数列表、初始化器或聚合项：`static inline mlir::Value swapVectorWordPairs(fir::FirOpBuilder &builder,`。
- **L1533 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Location loc,`.
  **L1533 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Location loc,`。
- **L1534 EN**: Continues the surrounding expression or declaration: `mlir::Value arg) {`.
  **L1534 CN**: 继续构造周围的表达式或声明：`mlir::Value arg) {`。
- **L1535 EN**: Initializes variable `ty` from the right-hand expression.
  **L1535 CN**: 使用右侧表达式初始化变量 `ty`。
- **L1536 EN**: Executes a call or declaration centered on `context{builder.getContext`.
  **L1536 CN**: 执行以 `context{builder.getContext` 为核心的调用或声明。

### Lines 1537-1560

````cpp
  auto vtype{mlir::VectorType::get(16, mlir::IntegerType::get(context, 8))};

  if (ty != vtype)
    arg = mlir::LLVM::BitcastOp::create(builder, loc, vtype, arg).getResult();

  llvm::SmallVector<int64_t, 16> mask{4,  5,  6,  7,  0, 1, 2,  3,
                                      12, 13, 14, 15, 8, 9, 10, 11};
  arg = mlir::vector::ShuffleOp::create(builder, loc, arg, arg, mask);
  if (ty != vtype)
    arg = mlir::LLVM::BitcastOp::create(builder, loc, ty, arg);
  return arg;
}

// VEC_CONVERT, VEC_CTF, VEC_CVF
template <VecOp vop>
fir::ExtendedValue
PPCIntrinsicLibrary::genVecConvert(mlir::Type resultType,
                                   llvm::ArrayRef<fir::ExtendedValue> args) {
  auto context{builder.getContext()};
  auto argBases{getBasesForArgs(args)};
  auto vecTyInfo{getVecTypeFromFir(argBases[0])};
  auto mlirTy{vecTyInfo.toMlirVectorType(context)};
  auto vArg1{builder.createConvert(loc, mlirTy, argBases[0])};
  const auto i32Ty{mlir::IntegerType::get(context, 32)};
````
- **L1537 EN**: Executes a call or declaration centered on `vtype{mlir::VectorType::get`.
  **L1537 CN**: 执行以 `vtype{mlir::VectorType::get` 为核心的调用或声明。
- **L1538 EN**: Blank line separating nearby declarations or logic blocks.
  **L1538 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1539 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1539 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1540 EN**: Executes a call or declaration centered on `mlir::LLVM::BitcastOp::create`.
  **L1540 CN**: 执行以 `mlir::LLVM::BitcastOp::create` 为核心的调用或声明。
- **L1541 EN**: Blank line separating nearby declarations or logic blocks.
  **L1541 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1542 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm::SmallVector<int64_t, 16> mask{4,  5,  6,  7,  0, 1, 2,  3,`.
  **L1542 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm::SmallVector<int64_t, 16> mask{4,  5,  6,  7,  0, 1, 2,  3,`。
- **L1543 EN**: Executes a standalone statement or declaration: `12, 13, 14, 15, 8, 9, 10, 11};`.
  **L1543 CN**: 执行一条独立语句或声明：`12, 13, 14, 15, 8, 9, 10, 11};`。
- **L1544 EN**: Executes a call or declaration centered on `mlir::vector::ShuffleOp::create`.
  **L1544 CN**: 执行以 `mlir::vector::ShuffleOp::create` 为核心的调用或声明。
- **L1545 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1545 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1546 EN**: Executes a call or declaration centered on `mlir::LLVM::BitcastOp::create`.
  **L1546 CN**: 执行以 `mlir::LLVM::BitcastOp::create` 为核心的调用或声明。
- **L1547 EN**: Returns from the current function with `arg`.
  **L1547 CN**: 以 `arg` 从当前函数返回。
- **L1548 EN**: Closes the current lexical scope or compound statement.
  **L1548 CN**: 结束当前词法作用域或复合语句块。
- **L1549 EN**: Blank line separating nearby declarations or logic blocks.
  **L1549 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1550 EN**: Comment explains nearby logic, intent, or metadata: `VEC_CONVERT, VEC_CTF, VEC_CVF`.
  **L1550 CN**: 注释说明附近代码的逻辑、意图或元数据：`VEC_CONVERT, VEC_CTF, VEC_CVF`。
- **L1551 EN**: Introduces template parameters or specialization context: `template <VecOp vop>`.
  **L1551 CN**: 为后续声明引入模板参数或特化上下文：`template <VecOp vop>`。
- **L1552 EN**: Continues the surrounding expression or declaration: `fir::ExtendedValue`.
  **L1552 CN**: 继续构造周围的表达式或声明：`fir::ExtendedValue`。
- **L1553 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PPCIntrinsicLibrary::genVecConvert(mlir::Type resultType,`.
  **L1553 CN**: 继续一个多行参数列表、初始化器或聚合项：`PPCIntrinsicLibrary::genVecConvert(mlir::Type resultType,`。
- **L1554 EN**: Continues the surrounding expression or declaration: `llvm::ArrayRef<fir::ExtendedValue> args) {`.
  **L1554 CN**: 继续构造周围的表达式或声明：`llvm::ArrayRef<fir::ExtendedValue> args) {`。
- **L1555 EN**: Executes a call or declaration centered on `context{builder.getContext`.
  **L1555 CN**: 执行以 `context{builder.getContext` 为核心的调用或声明。
- **L1556 EN**: Executes a call or declaration centered on `argBases{getBasesForArgs`.
  **L1556 CN**: 执行以 `argBases{getBasesForArgs` 为核心的调用或声明。
- **L1557 EN**: Executes a call or declaration centered on `vecTyInfo{getVecTypeFromFir`.
  **L1557 CN**: 执行以 `vecTyInfo{getVecTypeFromFir` 为核心的调用或声明。
- **L1558 EN**: Executes a call or declaration centered on `mlirTy{vecTyInfo.toMlirVectorType`.
  **L1558 CN**: 执行以 `mlirTy{vecTyInfo.toMlirVectorType` 为核心的调用或声明。
- **L1559 EN**: Executes a call or declaration centered on `vArg1{builder.createConvert`.
  **L1559 CN**: 执行以 `vArg1{builder.createConvert` 为核心的调用或声明。
- **L1560 EN**: Executes a call or declaration centered on `i32Ty{mlir::IntegerType::get`.
  **L1560 CN**: 执行以 `i32Ty{mlir::IntegerType::get` 为核心的调用或声明。

### Lines 1561-1584

````cpp

  switch (vop) {
  case VecOp::Ctf: {
    assert(args.size() == 2);
    auto convArg{builder.createConvert(loc, i32Ty, argBases[1])};
    auto eTy{mlir::dyn_cast<mlir::IntegerType>(vecTyInfo.eleTy)};
    assert(eTy && "Unsupported vector type");
    const auto isUnsigned{eTy.isUnsignedInteger()};
    const auto width{eTy.getWidth()};

    if (width == 32) {
      auto ftype{(isUnsigned)
                     ? genFuncType<Ty::RealVector<4>, Ty::UnsignedVector<4>,
                                   Ty::Integer<4>>(context, builder)
                     : genFuncType<Ty::RealVector<4>, Ty::IntegerVector<4>,
                                   Ty::Integer<4>>(context, builder)};
      const llvm::StringRef fname{(isUnsigned) ? "llvm.ppc.altivec.vcfux"
                                               : "llvm.ppc.altivec.vcfsx"};
      auto funcOp{builder.createFunction(loc, fname, ftype)};
      mlir::Value newArgs[] = {argBases[0], convArg};
      auto callOp{fir::CallOp::create(builder, loc, funcOp, newArgs)};

      return callOp.getResult(0);
    } else if (width == 64) {
````
- **L1561 EN**: Blank line separating nearby declarations or logic blocks.
  **L1561 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1562 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L1562 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L1563 EN**: Introduces a switch dispatch label: `case VecOp::Ctf: {`.
  **L1563 CN**: 引入一个 switch 分发标签：`case VecOp::Ctf: {`。
- **L1564 EN**: Checks an internal invariant in debug builds.
  **L1564 CN**: 在调试构建中检查内部不变式。
- **L1565 EN**: Executes a call or declaration centered on `convArg{builder.createConvert`.
  **L1565 CN**: 执行以 `convArg{builder.createConvert` 为核心的调用或声明。
- **L1566 EN**: Executes a call or declaration centered on `eTy{mlir::dyn_cast<mlir::IntegerType>`.
  **L1566 CN**: 执行以 `eTy{mlir::dyn_cast<mlir::IntegerType>` 为核心的调用或声明。
- **L1567 EN**: Checks an internal invariant in debug builds.
  **L1567 CN**: 在调试构建中检查内部不变式。
- **L1568 EN**: Executes a call or declaration centered on `isUnsigned{eTy.isUnsignedInteger`.
  **L1568 CN**: 执行以 `isUnsigned{eTy.isUnsignedInteger` 为核心的调用或声明。
- **L1569 EN**: Executes a call or declaration centered on `width{eTy.getWidth`.
  **L1569 CN**: 执行以 `width{eTy.getWidth` 为核心的调用或声明。
- **L1570 EN**: Blank line separating nearby declarations or logic blocks.
  **L1570 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1571 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1571 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1572 EN**: Continues the surrounding expression or declaration: `auto ftype{(isUnsigned)`.
  **L1572 CN**: 继续构造周围的表达式或声明：`auto ftype{(isUnsigned)`。
- **L1573 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `? genFuncType<Ty::RealVector<4>, Ty::UnsignedVector<4>,`.
  **L1573 CN**: 继续一个多行参数列表、初始化器或聚合项：`? genFuncType<Ty::RealVector<4>, Ty::UnsignedVector<4>,`。
- **L1574 EN**: Continues logic associated with callable symbol `Integer<4>>`.
  **L1574 CN**: 继续与可调用符号 `Integer<4>>` 相关的逻辑。
- **L1575 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: genFuncType<Ty::RealVector<4>, Ty::IntegerVector<4>,`.
  **L1575 CN**: 继续一个多行参数列表、初始化器或聚合项：`: genFuncType<Ty::RealVector<4>, Ty::IntegerVector<4>,`。
- **L1576 EN**: Executes a call or declaration centered on `Ty::Integer<4>>`.
  **L1576 CN**: 执行以 `Ty::Integer<4>>` 为核心的调用或声明。
- **L1577 EN**: Continues the surrounding expression or declaration: `const llvm::StringRef fname{(isUnsigned) ? "llvm.ppc.altivec.vcfux"`.
  **L1577 CN**: 继续构造周围的表达式或声明：`const llvm::StringRef fname{(isUnsigned) ? "llvm.ppc.altivec.vcfux"`。
- **L1578 EN**: Executes a standalone statement or declaration: `: "llvm.ppc.altivec.vcfsx"};`.
  **L1578 CN**: 执行一条独立语句或声明：`: "llvm.ppc.altivec.vcfsx"};`。
- **L1579 EN**: Executes a call or declaration centered on `funcOp{builder.createFunction`.
  **L1579 CN**: 执行以 `funcOp{builder.createFunction` 为核心的调用或声明。
- **L1580 EN**: Executes a standalone statement or declaration: `mlir::Value newArgs[] = {argBases[0], convArg};`.
  **L1580 CN**: 执行一条独立语句或声明：`mlir::Value newArgs[] = {argBases[0], convArg};`。
- **L1581 EN**: Executes a call or declaration centered on `callOp{fir::CallOp::create`.
  **L1581 CN**: 执行以 `callOp{fir::CallOp::create` 为核心的调用或声明。
- **L1582 EN**: Blank line separating nearby declarations or logic blocks.
  **L1582 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1583 EN**: Returns from the current function with `callOp.getResult(0)`.
  **L1583 CN**: 以 `callOp.getResult(0)` 从当前函数返回。
- **L1584 EN**: Transitions from the previous branch into an `else if` condition.
  **L1584 CN**: 从前一个分支过渡到 `else if` 条件判断。

### Lines 1585-1608

````cpp
      auto fTy{mlir::Float64Type::get(context)};
      auto ty{mlir::VectorType::get(2, fTy)};

      // vec_vtf(arg1, arg2) = fmul(1.0 / (1 << arg2), llvm.sitofp(arg1))
      auto convOp{(isUnsigned)
                      ? mlir::LLVM::UIToFPOp::create(builder, loc, ty, vArg1)
                      : mlir::LLVM::SIToFPOp::create(builder, loc, ty, vArg1)};

      // construct vector<1./(1<<arg1), 1.0/(1<<arg1)>
      auto constInt{mlir::dyn_cast_or_null<mlir::IntegerAttr>(
          mlir::dyn_cast<mlir::arith::ConstantOp>(argBases[1].getDefiningOp())
              .getValue())};
      assert(constInt && "expected integer constant argument");
      double f{1.0 / (1 << constInt.getInt())};
      llvm::SmallVector<double> vals{f, f};
      auto constOp{mlir::arith::ConstantOp::create(
          builder, loc, ty, builder.getF64VectorAttr(vals))};

      auto mulOp{mlir::LLVM::FMulOp::create(builder, loc, ty,
                                            convOp->getResult(0), constOp)};

      return builder.createConvert(loc, fir::VectorType::get(2, fTy), mulOp);
    }
    llvm_unreachable("invalid element integer kind");
````
- **L1585 EN**: Executes a call or declaration centered on `fTy{mlir::Float64Type::get`.
  **L1585 CN**: 执行以 `fTy{mlir::Float64Type::get` 为核心的调用或声明。
- **L1586 EN**: Executes a call or declaration centered on `ty{mlir::VectorType::get`.
  **L1586 CN**: 执行以 `ty{mlir::VectorType::get` 为核心的调用或声明。
- **L1587 EN**: Blank line separating nearby declarations or logic blocks.
  **L1587 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1588 EN**: Comment explains nearby logic, intent, or metadata: `vec_vtf(arg1, arg2) = fmul(1.0 / (1 << arg2), llvm.sitofp(arg1))`.
  **L1588 CN**: 注释说明附近代码的逻辑、意图或元数据：`vec_vtf(arg1, arg2) = fmul(1.0 / (1 << arg2), llvm.sitofp(arg1))`。
- **L1589 EN**: Continues the surrounding expression or declaration: `auto convOp{(isUnsigned)`.
  **L1589 CN**: 继续构造周围的表达式或声明：`auto convOp{(isUnsigned)`。
- **L1590 EN**: Continues logic associated with callable symbol `create`.
  **L1590 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L1591 EN**: Executes a call or declaration centered on `mlir::LLVM::SIToFPOp::create`.
  **L1591 CN**: 执行以 `mlir::LLVM::SIToFPOp::create` 为核心的调用或声明。
- **L1592 EN**: Blank line separating nearby declarations or logic blocks.
  **L1592 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1593 EN**: Comment explains nearby logic, intent, or metadata: `construct vector<1./(1<<arg1), 1.0/(1<<arg1)>`.
  **L1593 CN**: 注释说明附近代码的逻辑、意图或元数据：`construct vector<1./(1<<arg1), 1.0/(1<<arg1)>`。
- **L1594 EN**: Continues logic associated with callable symbol `IntegerAttr>`.
  **L1594 CN**: 继续与可调用符号 `IntegerAttr>` 相关的逻辑。
- **L1595 EN**: Continues logic associated with callable symbol `ConstantOp>`.
  **L1595 CN**: 继续与可调用符号 `ConstantOp>` 相关的逻辑。
- **L1596 EN**: Executes a call or declaration centered on `.getValue`.
  **L1596 CN**: 执行以 `.getValue` 为核心的调用或声明。
- **L1597 EN**: Checks an internal invariant in debug builds.
  **L1597 CN**: 在调试构建中检查内部不变式。
- **L1598 EN**: Executes a call or declaration centered on `/`.
  **L1598 CN**: 执行以 `/` 为核心的调用或声明。
- **L1599 EN**: Executes a standalone statement or declaration: `llvm::SmallVector<double> vals{f, f};`.
  **L1599 CN**: 执行一条独立语句或声明：`llvm::SmallVector<double> vals{f, f};`。
- **L1600 EN**: Continues logic associated with callable symbol `create`.
  **L1600 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L1601 EN**: Executes a call or declaration centered on `builder.getF64VectorAttr`.
  **L1601 CN**: 执行以 `builder.getF64VectorAttr` 为核心的调用或声明。
- **L1602 EN**: Blank line separating nearby declarations or logic blocks.
  **L1602 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1603 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto mulOp{mlir::LLVM::FMulOp::create(builder, loc, ty,`.
  **L1603 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto mulOp{mlir::LLVM::FMulOp::create(builder, loc, ty,`。
- **L1604 EN**: Executes a call or declaration centered on `convOp->getResult`.
  **L1604 CN**: 执行以 `convOp->getResult` 为核心的调用或声明。
- **L1605 EN**: Blank line separating nearby declarations or logic blocks.
  **L1605 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1606 EN**: Returns from the current function with `builder.createConvert(loc, fir::VectorType::get(2, fTy), mulOp)`.
  **L1606 CN**: 以 `builder.createConvert(loc, fir::VectorType::get(2, fTy), mulOp)` 从当前函数返回。
- **L1607 EN**: Closes the current lexical scope or compound statement.
  **L1607 CN**: 结束当前词法作用域或复合语句块。
- **L1608 EN**: Marks this control path as unreachable to LLVM.
  **L1608 CN**: 将该控制路径标记为 LLVM 认为不可达。

### Lines 1609-1632

````cpp
  }
  case VecOp::Convert: {
    assert(args.size() == 2);
    // resultType has mold type (if scalar) or element type (if array)
    auto resTyInfo{getVecTypeFromFirType(resultType)};
    auto moldTy{resTyInfo.toMlirVectorType(context)};
    auto firTy{resTyInfo.toFirVectorType()};

    // vec_convert(v, mold) = bitcast v to "type of mold"
    auto conv{mlir::LLVM::BitcastOp::create(builder, loc, moldTy, vArg1)};

    return builder.createConvert(loc, firTy, conv);
  }
  case VecOp::Cvf: {
    assert(args.size() == 1);

    mlir::Value newArgs[]{vArg1};
    if (vecTyInfo.isFloat32()) {
      if (changeVecElemOrder())
        newArgs[0] = swapVectorWordPairs(builder, loc, newArgs[0]);

      const llvm::StringRef fname{"llvm.ppc.vsx.xvcvspdp"};
      auto ftype{
          genFuncType<Ty::RealVector<8>, Ty::RealVector<4>>(context, builder)};
````
- **L1609 EN**: Closes the current lexical scope or compound statement.
  **L1609 CN**: 结束当前词法作用域或复合语句块。
- **L1610 EN**: Introduces a switch dispatch label: `case VecOp::Convert: {`.
  **L1610 CN**: 引入一个 switch 分发标签：`case VecOp::Convert: {`。
- **L1611 EN**: Checks an internal invariant in debug builds.
  **L1611 CN**: 在调试构建中检查内部不变式。
- **L1612 EN**: Comment explains nearby logic, intent, or metadata: `resultType has mold type (if scalar) or element type (if array)`.
  **L1612 CN**: 注释说明附近代码的逻辑、意图或元数据：`resultType has mold type (if scalar) or element type (if array)`。
- **L1613 EN**: Executes a call or declaration centered on `resTyInfo{getVecTypeFromFirType`.
  **L1613 CN**: 执行以 `resTyInfo{getVecTypeFromFirType` 为核心的调用或声明。
- **L1614 EN**: Executes a call or declaration centered on `moldTy{resTyInfo.toMlirVectorType`.
  **L1614 CN**: 执行以 `moldTy{resTyInfo.toMlirVectorType` 为核心的调用或声明。
- **L1615 EN**: Executes a call or declaration centered on `firTy{resTyInfo.toFirVectorType`.
  **L1615 CN**: 执行以 `firTy{resTyInfo.toFirVectorType` 为核心的调用或声明。
- **L1616 EN**: Blank line separating nearby declarations or logic blocks.
  **L1616 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1617 EN**: Comment explains nearby logic, intent, or metadata: `vec_convert(v, mold) = bitcast v to "type of mold"`.
  **L1617 CN**: 注释说明附近代码的逻辑、意图或元数据：`vec_convert(v, mold) = bitcast v to "type of mold"`。
- **L1618 EN**: Executes a call or declaration centered on `conv{mlir::LLVM::BitcastOp::create`.
  **L1618 CN**: 执行以 `conv{mlir::LLVM::BitcastOp::create` 为核心的调用或声明。
- **L1619 EN**: Blank line separating nearby declarations or logic blocks.
  **L1619 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1620 EN**: Returns from the current function with `builder.createConvert(loc, firTy, conv)`.
  **L1620 CN**: 以 `builder.createConvert(loc, firTy, conv)` 从当前函数返回。
- **L1621 EN**: Closes the current lexical scope or compound statement.
  **L1621 CN**: 结束当前词法作用域或复合语句块。
- **L1622 EN**: Introduces a switch dispatch label: `case VecOp::Cvf: {`.
  **L1622 CN**: 引入一个 switch 分发标签：`case VecOp::Cvf: {`。
- **L1623 EN**: Checks an internal invariant in debug builds.
  **L1623 CN**: 在调试构建中检查内部不变式。
- **L1624 EN**: Blank line separating nearby declarations or logic blocks.
  **L1624 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1625 EN**: Executes a standalone statement or declaration: `mlir::Value newArgs[]{vArg1};`.
  **L1625 CN**: 执行一条独立语句或声明：`mlir::Value newArgs[]{vArg1};`。
- **L1626 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1626 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1627 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1627 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1628 EN**: Executes a call or declaration centered on `swapVectorWordPairs`.
  **L1628 CN**: 执行以 `swapVectorWordPairs` 为核心的调用或声明。
- **L1629 EN**: Blank line separating nearby declarations or logic blocks.
  **L1629 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1630 EN**: Executes a standalone statement or declaration: `const llvm::StringRef fname{"llvm.ppc.vsx.xvcvspdp"};`.
  **L1630 CN**: 执行一条独立语句或声明：`const llvm::StringRef fname{"llvm.ppc.vsx.xvcvspdp"};`。
- **L1631 EN**: Continues the surrounding expression or declaration: `auto ftype{`.
  **L1631 CN**: 继续构造周围的表达式或声明：`auto ftype{`。
- **L1632 EN**: Executes a call or declaration centered on `Ty::RealVector<4>>`.
  **L1632 CN**: 执行以 `Ty::RealVector<4>>` 为核心的调用或声明。

### Lines 1633-1656

````cpp
      auto funcOp{builder.createFunction(loc, fname, ftype)};
      auto callOp{fir::CallOp::create(builder, loc, funcOp, newArgs)};

      return callOp.getResult(0);
    } else if (vecTyInfo.isFloat64()) {
      const llvm::StringRef fname{"llvm.ppc.vsx.xvcvdpsp"};
      auto ftype{
          genFuncType<Ty::RealVector<4>, Ty::RealVector<8>>(context, builder)};
      auto funcOp{builder.createFunction(loc, fname, ftype)};
      newArgs[0] =
          fir::CallOp::create(builder, loc, funcOp, newArgs).getResult(0);
      auto fvf32Ty{newArgs[0].getType()};
      auto f32type{mlir::Float32Type::get(context)};
      auto mvf32Ty{mlir::VectorType::get(4, f32type)};
      newArgs[0] = builder.createConvert(loc, mvf32Ty, newArgs[0]);

      if (changeVecElemOrder())
        newArgs[0] = swapVectorWordPairs(builder, loc, newArgs[0]);

      return builder.createConvert(loc, fvf32Ty, newArgs[0]);
    }
    llvm_unreachable("invalid element integer kind");
  }
  default:
````
- **L1633 EN**: Executes a call or declaration centered on `funcOp{builder.createFunction`.
  **L1633 CN**: 执行以 `funcOp{builder.createFunction` 为核心的调用或声明。
- **L1634 EN**: Executes a call or declaration centered on `callOp{fir::CallOp::create`.
  **L1634 CN**: 执行以 `callOp{fir::CallOp::create` 为核心的调用或声明。
- **L1635 EN**: Blank line separating nearby declarations or logic blocks.
  **L1635 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1636 EN**: Returns from the current function with `callOp.getResult(0)`.
  **L1636 CN**: 以 `callOp.getResult(0)` 从当前函数返回。
- **L1637 EN**: Transitions from the previous branch into an `else if` condition.
  **L1637 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1638 EN**: Executes a standalone statement or declaration: `const llvm::StringRef fname{"llvm.ppc.vsx.xvcvdpsp"};`.
  **L1638 CN**: 执行一条独立语句或声明：`const llvm::StringRef fname{"llvm.ppc.vsx.xvcvdpsp"};`。
- **L1639 EN**: Continues the surrounding expression or declaration: `auto ftype{`.
  **L1639 CN**: 继续构造周围的表达式或声明：`auto ftype{`。
- **L1640 EN**: Executes a call or declaration centered on `Ty::RealVector<8>>`.
  **L1640 CN**: 执行以 `Ty::RealVector<8>>` 为核心的调用或声明。
- **L1641 EN**: Executes a call or declaration centered on `funcOp{builder.createFunction`.
  **L1641 CN**: 执行以 `funcOp{builder.createFunction` 为核心的调用或声明。
- **L1642 EN**: Continues the surrounding expression or declaration: `newArgs[0] =`.
  **L1642 CN**: 继续构造周围的表达式或声明：`newArgs[0] =`。
- **L1643 EN**: Executes a call or declaration centered on `fir::CallOp::create`.
  **L1643 CN**: 执行以 `fir::CallOp::create` 为核心的调用或声明。
- **L1644 EN**: Executes a call or declaration centered on `fvf32Ty{newArgs[0].getType`.
  **L1644 CN**: 执行以 `fvf32Ty{newArgs[0].getType` 为核心的调用或声明。
- **L1645 EN**: Executes a call or declaration centered on `f32type{mlir::Float32Type::get`.
  **L1645 CN**: 执行以 `f32type{mlir::Float32Type::get` 为核心的调用或声明。
- **L1646 EN**: Executes a call or declaration centered on `mvf32Ty{mlir::VectorType::get`.
  **L1646 CN**: 执行以 `mvf32Ty{mlir::VectorType::get` 为核心的调用或声明。
- **L1647 EN**: Executes a call or declaration centered on `builder.createConvert`.
  **L1647 CN**: 执行以 `builder.createConvert` 为核心的调用或声明。
- **L1648 EN**: Blank line separating nearby declarations or logic blocks.
  **L1648 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1649 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1649 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1650 EN**: Executes a call or declaration centered on `swapVectorWordPairs`.
  **L1650 CN**: 执行以 `swapVectorWordPairs` 为核心的调用或声明。
- **L1651 EN**: Blank line separating nearby declarations or logic blocks.
  **L1651 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1652 EN**: Returns from the current function with `builder.createConvert(loc, fvf32Ty, newArgs[0])`.
  **L1652 CN**: 以 `builder.createConvert(loc, fvf32Ty, newArgs[0])` 从当前函数返回。
- **L1653 EN**: Closes the current lexical scope or compound statement.
  **L1653 CN**: 结束当前词法作用域或复合语句块。
- **L1654 EN**: Marks this control path as unreachable to LLVM.
  **L1654 CN**: 将该控制路径标记为 LLVM 认为不可达。
- **L1655 EN**: Closes the current lexical scope or compound statement.
  **L1655 CN**: 结束当前词法作用域或复合语句块。
- **L1656 EN**: Introduces a switch dispatch label: `default:`.
  **L1656 CN**: 引入一个 switch 分发标签：`default:`。

### Lines 1657-1680

````cpp
    llvm_unreachable("Invalid vector operation for generator");
  }
}

static mlir::Value convertVectorElementOrder(fir::FirOpBuilder &builder,
                                             mlir::Location loc,
                                             VecTypeInfo vecInfo,
                                             mlir::Value idx) {
  mlir::Value numSub1{
      builder.createIntegerConstant(loc, idx.getType(), vecInfo.len - 1)};
  return mlir::LLVM::SubOp::create(builder, loc, idx.getType(), numSub1, idx);
}

// VEC_EXTRACT
fir::ExtendedValue
PPCIntrinsicLibrary::genVecExtract(mlir::Type resultType,
                                   llvm::ArrayRef<fir::ExtendedValue> args) {
  assert(args.size() == 2);
  auto argBases{getBasesForArgs(args)};
  auto argTypes{getTypesForArgs(argBases)};
  auto vecTyInfo{getVecTypeFromFir(argBases[0])};

  auto mlirTy{vecTyInfo.toMlirVectorType(builder.getContext())};
  auto varg0{builder.createConvert(loc, mlirTy, argBases[0])};
````
- **L1657 EN**: Marks this control path as unreachable to LLVM.
  **L1657 CN**: 将该控制路径标记为 LLVM 认为不可达。
- **L1658 EN**: Closes the current lexical scope or compound statement.
  **L1658 CN**: 结束当前词法作用域或复合语句块。
- **L1659 EN**: Closes the current lexical scope or compound statement.
  **L1659 CN**: 结束当前词法作用域或复合语句块。
- **L1660 EN**: Blank line separating nearby declarations or logic blocks.
  **L1660 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1661 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static mlir::Value convertVectorElementOrder(fir::FirOpBuilder &builder,`.
  **L1661 CN**: 继续一个多行参数列表、初始化器或聚合项：`static mlir::Value convertVectorElementOrder(fir::FirOpBuilder &builder,`。
- **L1662 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Location loc,`.
  **L1662 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Location loc,`。
- **L1663 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VecTypeInfo vecInfo,`.
  **L1663 CN**: 继续一个多行参数列表、初始化器或聚合项：`VecTypeInfo vecInfo,`。
- **L1664 EN**: Continues the surrounding expression or declaration: `mlir::Value idx) {`.
  **L1664 CN**: 继续构造周围的表达式或声明：`mlir::Value idx) {`。
- **L1665 EN**: Continues the surrounding expression or declaration: `mlir::Value numSub1{`.
  **L1665 CN**: 继续构造周围的表达式或声明：`mlir::Value numSub1{`。
- **L1666 EN**: Executes a call or declaration centered on `builder.createIntegerConstant`.
  **L1666 CN**: 执行以 `builder.createIntegerConstant` 为核心的调用或声明。
- **L1667 EN**: Returns from the current function with `mlir::LLVM::SubOp::create(builder, loc, idx.getType(), numSub1, idx)`.
  **L1667 CN**: 以 `mlir::LLVM::SubOp::create(builder, loc, idx.getType(), numSub1, idx)` 从当前函数返回。
- **L1668 EN**: Closes the current lexical scope or compound statement.
  **L1668 CN**: 结束当前词法作用域或复合语句块。
- **L1669 EN**: Blank line separating nearby declarations or logic blocks.
  **L1669 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1670 EN**: Comment explains nearby logic, intent, or metadata: `VEC_EXTRACT`.
  **L1670 CN**: 注释说明附近代码的逻辑、意图或元数据：`VEC_EXTRACT`。
- **L1671 EN**: Continues the surrounding expression or declaration: `fir::ExtendedValue`.
  **L1671 CN**: 继续构造周围的表达式或声明：`fir::ExtendedValue`。
- **L1672 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PPCIntrinsicLibrary::genVecExtract(mlir::Type resultType,`.
  **L1672 CN**: 继续一个多行参数列表、初始化器或聚合项：`PPCIntrinsicLibrary::genVecExtract(mlir::Type resultType,`。
- **L1673 EN**: Continues the surrounding expression or declaration: `llvm::ArrayRef<fir::ExtendedValue> args) {`.
  **L1673 CN**: 继续构造周围的表达式或声明：`llvm::ArrayRef<fir::ExtendedValue> args) {`。
- **L1674 EN**: Checks an internal invariant in debug builds.
  **L1674 CN**: 在调试构建中检查内部不变式。
- **L1675 EN**: Executes a call or declaration centered on `argBases{getBasesForArgs`.
  **L1675 CN**: 执行以 `argBases{getBasesForArgs` 为核心的调用或声明。
- **L1676 EN**: Executes a call or declaration centered on `argTypes{getTypesForArgs`.
  **L1676 CN**: 执行以 `argTypes{getTypesForArgs` 为核心的调用或声明。
- **L1677 EN**: Executes a call or declaration centered on `vecTyInfo{getVecTypeFromFir`.
  **L1677 CN**: 执行以 `vecTyInfo{getVecTypeFromFir` 为核心的调用或声明。
- **L1678 EN**: Blank line separating nearby declarations or logic blocks.
  **L1678 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1679 EN**: Executes a call or declaration centered on `mlirTy{vecTyInfo.toMlirVectorType`.
  **L1679 CN**: 执行以 `mlirTy{vecTyInfo.toMlirVectorType` 为核心的调用或声明。
- **L1680 EN**: Executes a call or declaration centered on `varg0{builder.createConvert`.
  **L1680 CN**: 执行以 `varg0{builder.createConvert` 为核心的调用或声明。

### Lines 1681-1704

````cpp

  // arg2 modulo the number of elements in arg1 to determine the element
  // position
  auto numEle{builder.createIntegerConstant(loc, argTypes[1], vecTyInfo.len)};
  mlir::Value uremOp{
      mlir::LLVM::URemOp::create(builder, loc, argBases[1], numEle)};

  if (!isNativeVecElemOrderOnLE())
    uremOp = convertVectorElementOrder(builder, loc, vecTyInfo, uremOp);

  mlir::Value index = builder.createOrFold<mlir::index::CastUOp>(
      loc, builder.getIndexType(), uremOp);
  return mlir::vector::ExtractOp::create(builder, loc, varg0, index);
}

// VEC_INSERT
fir::ExtendedValue
PPCIntrinsicLibrary::genVecInsert(mlir::Type resultType,
                                  llvm::ArrayRef<fir::ExtendedValue> args) {
  assert(args.size() == 3);
  auto argBases{getBasesForArgs(args)};
  auto argTypes{getTypesForArgs(argBases)};
  auto vecTyInfo{getVecTypeFromFir(argBases[1])};
  auto mlirTy{vecTyInfo.toMlirVectorType(builder.getContext())};
````
- **L1681 EN**: Blank line separating nearby declarations or logic blocks.
  **L1681 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1682 EN**: Comment explains nearby logic, intent, or metadata: `arg2 modulo the number of elements in arg1 to determine the element`.
  **L1682 CN**: 注释说明附近代码的逻辑、意图或元数据：`arg2 modulo the number of elements in arg1 to determine the element`。
- **L1683 EN**: Comment explains nearby logic, intent, or metadata: `position`.
  **L1683 CN**: 注释说明附近代码的逻辑、意图或元数据：`position`。
- **L1684 EN**: Executes a call or declaration centered on `numEle{builder.createIntegerConstant`.
  **L1684 CN**: 执行以 `numEle{builder.createIntegerConstant` 为核心的调用或声明。
- **L1685 EN**: Continues the surrounding expression or declaration: `mlir::Value uremOp{`.
  **L1685 CN**: 继续构造周围的表达式或声明：`mlir::Value uremOp{`。
- **L1686 EN**: Executes a call or declaration centered on `mlir::LLVM::URemOp::create`.
  **L1686 CN**: 执行以 `mlir::LLVM::URemOp::create` 为核心的调用或声明。
- **L1687 EN**: Blank line separating nearby declarations or logic blocks.
  **L1687 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1688 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1688 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1689 EN**: Executes a call or declaration centered on `convertVectorElementOrder`.
  **L1689 CN**: 执行以 `convertVectorElementOrder` 为核心的调用或声明。
- **L1690 EN**: Blank line separating nearby declarations or logic blocks.
  **L1690 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1691 EN**: Continues logic associated with callable symbol `CastUOp>`.
  **L1691 CN**: 继续与可调用符号 `CastUOp>` 相关的逻辑。
- **L1692 EN**: Executes a call or declaration centered on `builder.getIndexType`.
  **L1692 CN**: 执行以 `builder.getIndexType` 为核心的调用或声明。
- **L1693 EN**: Returns from the current function with `mlir::vector::ExtractOp::create(builder, loc, varg0, index)`.
  **L1693 CN**: 以 `mlir::vector::ExtractOp::create(builder, loc, varg0, index)` 从当前函数返回。
- **L1694 EN**: Closes the current lexical scope or compound statement.
  **L1694 CN**: 结束当前词法作用域或复合语句块。
- **L1695 EN**: Blank line separating nearby declarations or logic blocks.
  **L1695 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1696 EN**: Comment explains nearby logic, intent, or metadata: `VEC_INSERT`.
  **L1696 CN**: 注释说明附近代码的逻辑、意图或元数据：`VEC_INSERT`。
- **L1697 EN**: Continues the surrounding expression or declaration: `fir::ExtendedValue`.
  **L1697 CN**: 继续构造周围的表达式或声明：`fir::ExtendedValue`。
- **L1698 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PPCIntrinsicLibrary::genVecInsert(mlir::Type resultType,`.
  **L1698 CN**: 继续一个多行参数列表、初始化器或聚合项：`PPCIntrinsicLibrary::genVecInsert(mlir::Type resultType,`。
- **L1699 EN**: Continues the surrounding expression or declaration: `llvm::ArrayRef<fir::ExtendedValue> args) {`.
  **L1699 CN**: 继续构造周围的表达式或声明：`llvm::ArrayRef<fir::ExtendedValue> args) {`。
- **L1700 EN**: Checks an internal invariant in debug builds.
  **L1700 CN**: 在调试构建中检查内部不变式。
- **L1701 EN**: Executes a call or declaration centered on `argBases{getBasesForArgs`.
  **L1701 CN**: 执行以 `argBases{getBasesForArgs` 为核心的调用或声明。
- **L1702 EN**: Executes a call or declaration centered on `argTypes{getTypesForArgs`.
  **L1702 CN**: 执行以 `argTypes{getTypesForArgs` 为核心的调用或声明。
- **L1703 EN**: Executes a call or declaration centered on `vecTyInfo{getVecTypeFromFir`.
  **L1703 CN**: 执行以 `vecTyInfo{getVecTypeFromFir` 为核心的调用或声明。
- **L1704 EN**: Executes a call or declaration centered on `mlirTy{vecTyInfo.toMlirVectorType`.
  **L1704 CN**: 执行以 `mlirTy{vecTyInfo.toMlirVectorType` 为核心的调用或声明。

### Lines 1705-1728

````cpp
  auto varg1{builder.createConvert(loc, mlirTy, argBases[1])};

  auto numEle{builder.createIntegerConstant(loc, argTypes[2], vecTyInfo.len)};
  mlir::Value uremOp{
      mlir::LLVM::URemOp::create(builder, loc, argBases[2], numEle)};

  if (!isNativeVecElemOrderOnLE())
    uremOp = convertVectorElementOrder(builder, loc, vecTyInfo, uremOp);

  mlir::Value index = builder.createOrFold<mlir::index::CastUOp>(
      loc, builder.getIndexType(), uremOp);
  mlir::Value res =
      mlir::vector::InsertOp::create(builder, loc, argBases[0], varg1, index);
  return fir::ConvertOp::create(builder, loc, vecTyInfo.toFirVectorType(), res);
}

// VEC_MERGEH, VEC_MERGEL
template <VecOp vop>
fir::ExtendedValue
PPCIntrinsicLibrary::genVecMerge(mlir::Type resultType,
                                 llvm::ArrayRef<fir::ExtendedValue> args) {
  assert(args.size() == 2);
  auto argBases{getBasesForArgs(args)};
  auto vecTyInfo{getVecTypeFromFir(argBases[0])};
````
- **L1705 EN**: Executes a call or declaration centered on `varg1{builder.createConvert`.
  **L1705 CN**: 执行以 `varg1{builder.createConvert` 为核心的调用或声明。
- **L1706 EN**: Blank line separating nearby declarations or logic blocks.
  **L1706 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1707 EN**: Executes a call or declaration centered on `numEle{builder.createIntegerConstant`.
  **L1707 CN**: 执行以 `numEle{builder.createIntegerConstant` 为核心的调用或声明。
- **L1708 EN**: Continues the surrounding expression or declaration: `mlir::Value uremOp{`.
  **L1708 CN**: 继续构造周围的表达式或声明：`mlir::Value uremOp{`。
- **L1709 EN**: Executes a call or declaration centered on `mlir::LLVM::URemOp::create`.
  **L1709 CN**: 执行以 `mlir::LLVM::URemOp::create` 为核心的调用或声明。
- **L1710 EN**: Blank line separating nearby declarations or logic blocks.
  **L1710 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1711 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1711 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1712 EN**: Executes a call or declaration centered on `convertVectorElementOrder`.
  **L1712 CN**: 执行以 `convertVectorElementOrder` 为核心的调用或声明。
- **L1713 EN**: Blank line separating nearby declarations or logic blocks.
  **L1713 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1714 EN**: Continues logic associated with callable symbol `CastUOp>`.
  **L1714 CN**: 继续与可调用符号 `CastUOp>` 相关的逻辑。
- **L1715 EN**: Executes a call or declaration centered on `builder.getIndexType`.
  **L1715 CN**: 执行以 `builder.getIndexType` 为核心的调用或声明。
- **L1716 EN**: Continues the surrounding expression or declaration: `mlir::Value res =`.
  **L1716 CN**: 继续构造周围的表达式或声明：`mlir::Value res =`。
- **L1717 EN**: Executes a call or declaration centered on `mlir::vector::InsertOp::create`.
  **L1717 CN**: 执行以 `mlir::vector::InsertOp::create` 为核心的调用或声明。
- **L1718 EN**: Returns from the current function with `fir::ConvertOp::create(builder, loc, vecTyInfo.toFirVectorType(), res)`.
  **L1718 CN**: 以 `fir::ConvertOp::create(builder, loc, vecTyInfo.toFirVectorType(), res)` 从当前函数返回。
- **L1719 EN**: Closes the current lexical scope or compound statement.
  **L1719 CN**: 结束当前词法作用域或复合语句块。
- **L1720 EN**: Blank line separating nearby declarations or logic blocks.
  **L1720 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1721 EN**: Comment explains nearby logic, intent, or metadata: `VEC_MERGEH, VEC_MERGEL`.
  **L1721 CN**: 注释说明附近代码的逻辑、意图或元数据：`VEC_MERGEH, VEC_MERGEL`。
- **L1722 EN**: Introduces template parameters or specialization context: `template <VecOp vop>`.
  **L1722 CN**: 为后续声明引入模板参数或特化上下文：`template <VecOp vop>`。
- **L1723 EN**: Continues the surrounding expression or declaration: `fir::ExtendedValue`.
  **L1723 CN**: 继续构造周围的表达式或声明：`fir::ExtendedValue`。
- **L1724 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PPCIntrinsicLibrary::genVecMerge(mlir::Type resultType,`.
  **L1724 CN**: 继续一个多行参数列表、初始化器或聚合项：`PPCIntrinsicLibrary::genVecMerge(mlir::Type resultType,`。
- **L1725 EN**: Continues the surrounding expression or declaration: `llvm::ArrayRef<fir::ExtendedValue> args) {`.
  **L1725 CN**: 继续构造周围的表达式或声明：`llvm::ArrayRef<fir::ExtendedValue> args) {`。
- **L1726 EN**: Checks an internal invariant in debug builds.
  **L1726 CN**: 在调试构建中检查内部不变式。
- **L1727 EN**: Executes a call or declaration centered on `argBases{getBasesForArgs`.
  **L1727 CN**: 执行以 `argBases{getBasesForArgs` 为核心的调用或声明。
- **L1728 EN**: Executes a call or declaration centered on `vecTyInfo{getVecTypeFromFir`.
  **L1728 CN**: 执行以 `vecTyInfo{getVecTypeFromFir` 为核心的调用或声明。

### Lines 1729-1752

````cpp
  llvm::SmallVector<int64_t, 16> mMask; // native vector element order mask
  llvm::SmallVector<int64_t, 16> rMask; // non-native vector element order mask

  switch (vop) {
  case VecOp::Mergeh: {
    switch (vecTyInfo.len) {
    case 2: {
      enum { V1 = 0, V2 = 2 };
      mMask = {V1 + 0, V2 + 0};
      rMask = {V2 + 1, V1 + 1};
      break;
    }
    case 4: {
      enum { V1 = 0, V2 = 4 };
      mMask = {V1 + 0, V2 + 0, V1 + 1, V2 + 1};
      rMask = {V2 + 2, V1 + 2, V2 + 3, V1 + 3};
      break;
    }
    case 8: {
      enum { V1 = 0, V2 = 8 };
      mMask = {V1 + 0, V2 + 0, V1 + 1, V2 + 1, V1 + 2, V2 + 2, V1 + 3, V2 + 3};
      rMask = {V2 + 4, V1 + 4, V2 + 5, V1 + 5, V2 + 6, V1 + 6, V2 + 7, V1 + 7};
      break;
    }
````
- **L1729 EN**: Continues the surrounding expression or declaration: `llvm::SmallVector<int64_t, 16> mMask; // native vector element order mask`.
  **L1729 CN**: 继续构造周围的表达式或声明：`llvm::SmallVector<int64_t, 16> mMask; // native vector element order mask`。
- **L1730 EN**: Continues the surrounding expression or declaration: `llvm::SmallVector<int64_t, 16> rMask; // non-native vector element order mask`.
  **L1730 CN**: 继续构造周围的表达式或声明：`llvm::SmallVector<int64_t, 16> rMask; // non-native vector element order mask`。
- **L1731 EN**: Blank line separating nearby declarations or logic blocks.
  **L1731 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1732 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L1732 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L1733 EN**: Introduces a switch dispatch label: `case VecOp::Mergeh: {`.
  **L1733 CN**: 引入一个 switch 分发标签：`case VecOp::Mergeh: {`。
- **L1734 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L1734 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L1735 EN**: Introduces a switch dispatch label: `case 2: {`.
  **L1735 CN**: 引入一个 switch 分发标签：`case 2: {`。
- **L1736 EN**: Declares enum ``.
  **L1736 CN**: 声明 enum ``。
- **L1737 EN**: Executes a standalone statement or declaration: `mMask = {V1 + 0, V2 + 0};`.
  **L1737 CN**: 执行一条独立语句或声明：`mMask = {V1 + 0, V2 + 0};`。
- **L1738 EN**: Executes a standalone statement or declaration: `rMask = {V2 + 1, V1 + 1};`.
  **L1738 CN**: 执行一条独立语句或声明：`rMask = {V2 + 1, V1 + 1};`。
- **L1739 EN**: Exits the nearest loop or switch statement.
  **L1739 CN**: 退出最近的循环或 switch 语句。
- **L1740 EN**: Closes the current lexical scope or compound statement.
  **L1740 CN**: 结束当前词法作用域或复合语句块。
- **L1741 EN**: Introduces a switch dispatch label: `case 4: {`.
  **L1741 CN**: 引入一个 switch 分发标签：`case 4: {`。
- **L1742 EN**: Declares enum ``.
  **L1742 CN**: 声明 enum ``。
- **L1743 EN**: Executes a standalone statement or declaration: `mMask = {V1 + 0, V2 + 0, V1 + 1, V2 + 1};`.
  **L1743 CN**: 执行一条独立语句或声明：`mMask = {V1 + 0, V2 + 0, V1 + 1, V2 + 1};`。
- **L1744 EN**: Executes a standalone statement or declaration: `rMask = {V2 + 2, V1 + 2, V2 + 3, V1 + 3};`.
  **L1744 CN**: 执行一条独立语句或声明：`rMask = {V2 + 2, V1 + 2, V2 + 3, V1 + 3};`。
- **L1745 EN**: Exits the nearest loop or switch statement.
  **L1745 CN**: 退出最近的循环或 switch 语句。
- **L1746 EN**: Closes the current lexical scope or compound statement.
  **L1746 CN**: 结束当前词法作用域或复合语句块。
- **L1747 EN**: Introduces a switch dispatch label: `case 8: {`.
  **L1747 CN**: 引入一个 switch 分发标签：`case 8: {`。
- **L1748 EN**: Declares enum ``.
  **L1748 CN**: 声明 enum ``。
- **L1749 EN**: Executes a standalone statement or declaration: `mMask = {V1 + 0, V2 + 0, V1 + 1, V2 + 1, V1 + 2, V2 + 2, V1 + 3, V2 + 3};`.
  **L1749 CN**: 执行一条独立语句或声明：`mMask = {V1 + 0, V2 + 0, V1 + 1, V2 + 1, V1 + 2, V2 + 2, V1 + 3, V2 + 3};`。
- **L1750 EN**: Executes a standalone statement or declaration: `rMask = {V2 + 4, V1 + 4, V2 + 5, V1 + 5, V2 + 6, V1 + 6, V2 + 7, V1 + 7};`.
  **L1750 CN**: 执行一条独立语句或声明：`rMask = {V2 + 4, V1 + 4, V2 + 5, V1 + 5, V2 + 6, V1 + 6, V2 + 7, V1 + 7};`。
- **L1751 EN**: Exits the nearest loop or switch statement.
  **L1751 CN**: 退出最近的循环或 switch 语句。
- **L1752 EN**: Closes the current lexical scope or compound statement.
  **L1752 CN**: 结束当前词法作用域或复合语句块。

### Lines 1753-1776

````cpp
    case 16:
      mMask = {0x00, 0x10, 0x01, 0x11, 0x02, 0x12, 0x03, 0x13,
               0x04, 0x14, 0x05, 0x15, 0x06, 0x16, 0x07, 0x17};
      rMask = {0x18, 0x08, 0x19, 0x09, 0x1A, 0x0A, 0x1B, 0x0B,
               0x1C, 0x0C, 0x1D, 0x0D, 0x1E, 0x0E, 0x1F, 0x0F};
      break;
    default:
      llvm_unreachable("unexpected vector length");
    }
    break;
  }
  case VecOp::Mergel: {
    switch (vecTyInfo.len) {
    case 2: {
      enum { V1 = 0, V2 = 2 };
      mMask = {V1 + 1, V2 + 1};
      rMask = {V2 + 0, V1 + 0};
      break;
    }
    case 4: {
      enum { V1 = 0, V2 = 4 };
      mMask = {V1 + 2, V2 + 2, V1 + 3, V2 + 3};
      rMask = {V2 + 0, V1 + 0, V2 + 1, V1 + 1};
      break;
````
- **L1753 EN**: Introduces a switch dispatch label: `case 16:`.
  **L1753 CN**: 引入一个 switch 分发标签：`case 16:`。
- **L1754 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mMask = {0x00, 0x10, 0x01, 0x11, 0x02, 0x12, 0x03, 0x13,`.
  **L1754 CN**: 继续一个多行参数列表、初始化器或聚合项：`mMask = {0x00, 0x10, 0x01, 0x11, 0x02, 0x12, 0x03, 0x13,`。
- **L1755 EN**: Executes a standalone statement or declaration: `0x04, 0x14, 0x05, 0x15, 0x06, 0x16, 0x07, 0x17};`.
  **L1755 CN**: 执行一条独立语句或声明：`0x04, 0x14, 0x05, 0x15, 0x06, 0x16, 0x07, 0x17};`。
- **L1756 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rMask = {0x18, 0x08, 0x19, 0x09, 0x1A, 0x0A, 0x1B, 0x0B,`.
  **L1756 CN**: 继续一个多行参数列表、初始化器或聚合项：`rMask = {0x18, 0x08, 0x19, 0x09, 0x1A, 0x0A, 0x1B, 0x0B,`。
- **L1757 EN**: Executes a standalone statement or declaration: `0x1C, 0x0C, 0x1D, 0x0D, 0x1E, 0x0E, 0x1F, 0x0F};`.
  **L1757 CN**: 执行一条独立语句或声明：`0x1C, 0x0C, 0x1D, 0x0D, 0x1E, 0x0E, 0x1F, 0x0F};`。
- **L1758 EN**: Exits the nearest loop or switch statement.
  **L1758 CN**: 退出最近的循环或 switch 语句。
- **L1759 EN**: Introduces a switch dispatch label: `default:`.
  **L1759 CN**: 引入一个 switch 分发标签：`default:`。
- **L1760 EN**: Marks this control path as unreachable to LLVM.
  **L1760 CN**: 将该控制路径标记为 LLVM 认为不可达。
- **L1761 EN**: Closes the current lexical scope or compound statement.
  **L1761 CN**: 结束当前词法作用域或复合语句块。
- **L1762 EN**: Exits the nearest loop or switch statement.
  **L1762 CN**: 退出最近的循环或 switch 语句。
- **L1763 EN**: Closes the current lexical scope or compound statement.
  **L1763 CN**: 结束当前词法作用域或复合语句块。
- **L1764 EN**: Introduces a switch dispatch label: `case VecOp::Mergel: {`.
  **L1764 CN**: 引入一个 switch 分发标签：`case VecOp::Mergel: {`。
- **L1765 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L1765 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L1766 EN**: Introduces a switch dispatch label: `case 2: {`.
  **L1766 CN**: 引入一个 switch 分发标签：`case 2: {`。
- **L1767 EN**: Declares enum ``.
  **L1767 CN**: 声明 enum ``。
- **L1768 EN**: Executes a standalone statement or declaration: `mMask = {V1 + 1, V2 + 1};`.
  **L1768 CN**: 执行一条独立语句或声明：`mMask = {V1 + 1, V2 + 1};`。
- **L1769 EN**: Executes a standalone statement or declaration: `rMask = {V2 + 0, V1 + 0};`.
  **L1769 CN**: 执行一条独立语句或声明：`rMask = {V2 + 0, V1 + 0};`。
- **L1770 EN**: Exits the nearest loop or switch statement.
  **L1770 CN**: 退出最近的循环或 switch 语句。
- **L1771 EN**: Closes the current lexical scope or compound statement.
  **L1771 CN**: 结束当前词法作用域或复合语句块。
- **L1772 EN**: Introduces a switch dispatch label: `case 4: {`.
  **L1772 CN**: 引入一个 switch 分发标签：`case 4: {`。
- **L1773 EN**: Declares enum ``.
  **L1773 CN**: 声明 enum ``。
- **L1774 EN**: Executes a standalone statement or declaration: `mMask = {V1 + 2, V2 + 2, V1 + 3, V2 + 3};`.
  **L1774 CN**: 执行一条独立语句或声明：`mMask = {V1 + 2, V2 + 2, V1 + 3, V2 + 3};`。
- **L1775 EN**: Executes a standalone statement or declaration: `rMask = {V2 + 0, V1 + 0, V2 + 1, V1 + 1};`.
  **L1775 CN**: 执行一条独立语句或声明：`rMask = {V2 + 0, V1 + 0, V2 + 1, V1 + 1};`。
- **L1776 EN**: Exits the nearest loop or switch statement.
  **L1776 CN**: 退出最近的循环或 switch 语句。

### Lines 1777-1800

````cpp
    }
    case 8: {
      enum { V1 = 0, V2 = 8 };
      mMask = {V1 + 4, V2 + 4, V1 + 5, V2 + 5, V1 + 6, V2 + 6, V1 + 7, V2 + 7};
      rMask = {V2 + 0, V1 + 0, V2 + 1, V1 + 1, V2 + 2, V1 + 2, V2 + 3, V1 + 3};
      break;
    }
    case 16:
      mMask = {0x08, 0x18, 0x09, 0x19, 0x0A, 0x1A, 0x0B, 0x1B,
               0x0C, 0x1C, 0x0D, 0x1D, 0x0E, 0x1E, 0x0F, 0x1F};
      rMask = {0x10, 0x00, 0x11, 0x01, 0x12, 0x02, 0x13, 0x03,
               0x14, 0x04, 0x15, 0x05, 0x16, 0x06, 0x17, 0x07};
      break;
    default:
      llvm_unreachable("unexpected vector length");
    }
    break;
  }
  default:
    llvm_unreachable("invalid vector operation for generator");
  }

  auto vargs{convertVecArgs(builder, loc, vecTyInfo, argBases)};

````
- **L1777 EN**: Closes the current lexical scope or compound statement.
  **L1777 CN**: 结束当前词法作用域或复合语句块。
- **L1778 EN**: Introduces a switch dispatch label: `case 8: {`.
  **L1778 CN**: 引入一个 switch 分发标签：`case 8: {`。
- **L1779 EN**: Declares enum ``.
  **L1779 CN**: 声明 enum ``。
- **L1780 EN**: Executes a standalone statement or declaration: `mMask = {V1 + 4, V2 + 4, V1 + 5, V2 + 5, V1 + 6, V2 + 6, V1 + 7, V2 + 7};`.
  **L1780 CN**: 执行一条独立语句或声明：`mMask = {V1 + 4, V2 + 4, V1 + 5, V2 + 5, V1 + 6, V2 + 6, V1 + 7, V2 + 7};`。
- **L1781 EN**: Executes a standalone statement or declaration: `rMask = {V2 + 0, V1 + 0, V2 + 1, V1 + 1, V2 + 2, V1 + 2, V2 + 3, V1 + 3};`.
  **L1781 CN**: 执行一条独立语句或声明：`rMask = {V2 + 0, V1 + 0, V2 + 1, V1 + 1, V2 + 2, V1 + 2, V2 + 3, V1 + 3};`。
- **L1782 EN**: Exits the nearest loop or switch statement.
  **L1782 CN**: 退出最近的循环或 switch 语句。
- **L1783 EN**: Closes the current lexical scope or compound statement.
  **L1783 CN**: 结束当前词法作用域或复合语句块。
- **L1784 EN**: Introduces a switch dispatch label: `case 16:`.
  **L1784 CN**: 引入一个 switch 分发标签：`case 16:`。
- **L1785 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mMask = {0x08, 0x18, 0x09, 0x19, 0x0A, 0x1A, 0x0B, 0x1B,`.
  **L1785 CN**: 继续一个多行参数列表、初始化器或聚合项：`mMask = {0x08, 0x18, 0x09, 0x19, 0x0A, 0x1A, 0x0B, 0x1B,`。
- **L1786 EN**: Executes a standalone statement or declaration: `0x0C, 0x1C, 0x0D, 0x1D, 0x0E, 0x1E, 0x0F, 0x1F};`.
  **L1786 CN**: 执行一条独立语句或声明：`0x0C, 0x1C, 0x0D, 0x1D, 0x0E, 0x1E, 0x0F, 0x1F};`。
- **L1787 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rMask = {0x10, 0x00, 0x11, 0x01, 0x12, 0x02, 0x13, 0x03,`.
  **L1787 CN**: 继续一个多行参数列表、初始化器或聚合项：`rMask = {0x10, 0x00, 0x11, 0x01, 0x12, 0x02, 0x13, 0x03,`。
- **L1788 EN**: Executes a standalone statement or declaration: `0x14, 0x04, 0x15, 0x05, 0x16, 0x06, 0x17, 0x07};`.
  **L1788 CN**: 执行一条独立语句或声明：`0x14, 0x04, 0x15, 0x05, 0x16, 0x06, 0x17, 0x07};`。
- **L1789 EN**: Exits the nearest loop or switch statement.
  **L1789 CN**: 退出最近的循环或 switch 语句。
- **L1790 EN**: Introduces a switch dispatch label: `default:`.
  **L1790 CN**: 引入一个 switch 分发标签：`default:`。
- **L1791 EN**: Marks this control path as unreachable to LLVM.
  **L1791 CN**: 将该控制路径标记为 LLVM 认为不可达。
- **L1792 EN**: Closes the current lexical scope or compound statement.
  **L1792 CN**: 结束当前词法作用域或复合语句块。
- **L1793 EN**: Exits the nearest loop or switch statement.
  **L1793 CN**: 退出最近的循环或 switch 语句。
- **L1794 EN**: Closes the current lexical scope or compound statement.
  **L1794 CN**: 结束当前词法作用域或复合语句块。
- **L1795 EN**: Introduces a switch dispatch label: `default:`.
  **L1795 CN**: 引入一个 switch 分发标签：`default:`。
- **L1796 EN**: Marks this control path as unreachable to LLVM.
  **L1796 CN**: 将该控制路径标记为 LLVM 认为不可达。
- **L1797 EN**: Closes the current lexical scope or compound statement.
  **L1797 CN**: 结束当前词法作用域或复合语句块。
- **L1798 EN**: Blank line separating nearby declarations or logic blocks.
  **L1798 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1799 EN**: Executes a call or declaration centered on `vargs{convertVecArgs`.
  **L1799 CN**: 执行以 `vargs{convertVecArgs` 为核心的调用或声明。
- **L1800 EN**: Blank line separating nearby declarations or logic blocks.
  **L1800 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 1801-1824

````cpp
  llvm::SmallVector<int64_t, 16> &mergeMask =
      (isBEVecElemOrderOnLE()) ? rMask : mMask;

  auto callOp{mlir::vector::ShuffleOp::create(builder, loc, vargs[0], vargs[1],
                                              mergeMask)};
  return builder.createConvert(loc, resultType, callOp);
}

static mlir::Value addOffsetToAddress(fir::FirOpBuilder &builder,
                                      mlir::Location loc, mlir::Value baseAddr,
                                      mlir::Value offset) {
  auto typeExtent{fir::SequenceType::getUnknownExtent()};
  // Construct an !fir.ref<!ref.array<?xi8>> type
  auto arrRefTy{builder.getRefType(fir::SequenceType::get(
      {typeExtent}, mlir::IntegerType::get(builder.getContext(), 8)))};
  // Convert arg to !fir.ref<!ref.array<?xi8>>
  auto resAddr{fir::ConvertOp::create(builder, loc, arrRefTy, baseAddr)};

  return fir::CoordinateOp::create(builder, loc, arrRefTy, resAddr, offset);
}

static mlir::Value reverseVectorElements(fir::FirOpBuilder &builder,
                                         mlir::Location loc, mlir::Value v,
                                         int64_t len) {
````
- **L1801 EN**: Continues the surrounding expression or declaration: `llvm::SmallVector<int64_t, 16> &mergeMask =`.
  **L1801 CN**: 继续构造周围的表达式或声明：`llvm::SmallVector<int64_t, 16> &mergeMask =`。
- **L1802 EN**: Executes a call or declaration centered on `statement`.
  **L1802 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1803 EN**: Blank line separating nearby declarations or logic blocks.
  **L1803 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1804 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto callOp{mlir::vector::ShuffleOp::create(builder, loc, vargs[0], vargs[1],`.
  **L1804 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto callOp{mlir::vector::ShuffleOp::create(builder, loc, vargs[0], vargs[1],`。
- **L1805 EN**: Executes a standalone statement or declaration: `mergeMask)};`.
  **L1805 CN**: 执行一条独立语句或声明：`mergeMask)};`。
- **L1806 EN**: Returns from the current function with `builder.createConvert(loc, resultType, callOp)`.
  **L1806 CN**: 以 `builder.createConvert(loc, resultType, callOp)` 从当前函数返回。
- **L1807 EN**: Closes the current lexical scope or compound statement.
  **L1807 CN**: 结束当前词法作用域或复合语句块。
- **L1808 EN**: Blank line separating nearby declarations or logic blocks.
  **L1808 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1809 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static mlir::Value addOffsetToAddress(fir::FirOpBuilder &builder,`.
  **L1809 CN**: 继续一个多行参数列表、初始化器或聚合项：`static mlir::Value addOffsetToAddress(fir::FirOpBuilder &builder,`。
- **L1810 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Location loc, mlir::Value baseAddr,`.
  **L1810 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Location loc, mlir::Value baseAddr,`。
- **L1811 EN**: Continues the surrounding expression or declaration: `mlir::Value offset) {`.
  **L1811 CN**: 继续构造周围的表达式或声明：`mlir::Value offset) {`。
- **L1812 EN**: Executes a call or declaration centered on `typeExtent{fir::SequenceType::getUnknownExtent`.
  **L1812 CN**: 执行以 `typeExtent{fir::SequenceType::getUnknownExtent` 为核心的调用或声明。
- **L1813 EN**: Comment explains nearby logic, intent, or metadata: `Construct an !fir.ref<!ref.array<?xi8>> type`.
  **L1813 CN**: 注释说明附近代码的逻辑、意图或元数据：`Construct an !fir.ref<!ref.array<?xi8>> type`。
- **L1814 EN**: Continues logic associated with callable symbol `getRefType`.
  **L1814 CN**: 继续与可调用符号 `getRefType` 相关的逻辑。
- **L1815 EN**: Executes a call or declaration centered on `mlir::IntegerType::get`.
  **L1815 CN**: 执行以 `mlir::IntegerType::get` 为核心的调用或声明。
- **L1816 EN**: Comment explains nearby logic, intent, or metadata: `Convert arg to !fir.ref<!ref.array<?xi8>>`.
  **L1816 CN**: 注释说明附近代码的逻辑、意图或元数据：`Convert arg to !fir.ref<!ref.array<?xi8>>`。
- **L1817 EN**: Executes a call or declaration centered on `resAddr{fir::ConvertOp::create`.
  **L1817 CN**: 执行以 `resAddr{fir::ConvertOp::create` 为核心的调用或声明。
- **L1818 EN**: Blank line separating nearby declarations or logic blocks.
  **L1818 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1819 EN**: Returns from the current function with `fir::CoordinateOp::create(builder, loc, arrRefTy, resAddr, offset)`.
  **L1819 CN**: 以 `fir::CoordinateOp::create(builder, loc, arrRefTy, resAddr, offset)` 从当前函数返回。
- **L1820 EN**: Closes the current lexical scope or compound statement.
  **L1820 CN**: 结束当前词法作用域或复合语句块。
- **L1821 EN**: Blank line separating nearby declarations or logic blocks.
  **L1821 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1822 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static mlir::Value reverseVectorElements(fir::FirOpBuilder &builder,`.
  **L1822 CN**: 继续一个多行参数列表、初始化器或聚合项：`static mlir::Value reverseVectorElements(fir::FirOpBuilder &builder,`。
- **L1823 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::Location loc, mlir::Value v,`.
  **L1823 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::Location loc, mlir::Value v,`。
- **L1824 EN**: Continues the surrounding expression or declaration: `int64_t len) {`.
  **L1824 CN**: 继续构造周围的表达式或声明：`int64_t len) {`。

### Lines 1825-1848

````cpp
  assert(mlir::isa<mlir::VectorType>(v.getType()));
  assert(len > 0);
  llvm::SmallVector<int64_t, 16> mask;
  for (int64_t i = 0; i < len; ++i) {
    mask.push_back(len - 1 - i);
  }
  auto undefVec{fir::UndefOp::create(builder, loc, v.getType())};
  return mlir::vector::ShuffleOp::create(builder, loc, v, undefVec, mask);
}

static mlir::NamedAttribute getAlignmentAttr(fir::FirOpBuilder &builder,
                                             const int val) {
  auto i64ty{mlir::IntegerType::get(builder.getContext(), 64)};
  auto alignAttr{mlir::IntegerAttr::get(i64ty, val)};
  return builder.getNamedAttr("alignment", alignAttr);
}

fir::ExtendedValue
PPCIntrinsicLibrary::genVecXlGrp(mlir::Type resultType,
                                 llvm::ArrayRef<fir::ExtendedValue> args) {
  VecTypeInfo vecTyInfo{getVecTypeFromFirType(resultType)};
  switch (vecTyInfo.eleTy.getIntOrFloatBitWidth()) {
  case 8:
    // vec_xlb1
````
- **L1825 EN**: Checks an internal invariant in debug builds.
  **L1825 CN**: 在调试构建中检查内部不变式。
- **L1826 EN**: Checks an internal invariant in debug builds.
  **L1826 CN**: 在调试构建中检查内部不变式。
- **L1827 EN**: Executes a standalone statement or declaration: `llvm::SmallVector<int64_t, 16> mask;`.
  **L1827 CN**: 执行一条独立语句或声明：`llvm::SmallVector<int64_t, 16> mask;`。
- **L1828 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1828 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1829 EN**: Executes a call or declaration centered on `mask.push_back`.
  **L1829 CN**: 执行以 `mask.push_back` 为核心的调用或声明。
- **L1830 EN**: Closes the current lexical scope or compound statement.
  **L1830 CN**: 结束当前词法作用域或复合语句块。
- **L1831 EN**: Executes a call or declaration centered on `undefVec{fir::UndefOp::create`.
  **L1831 CN**: 执行以 `undefVec{fir::UndefOp::create` 为核心的调用或声明。
- **L1832 EN**: Returns from the current function with `mlir::vector::ShuffleOp::create(builder, loc, v, undefVec, mask)`.
  **L1832 CN**: 以 `mlir::vector::ShuffleOp::create(builder, loc, v, undefVec, mask)` 从当前函数返回。
- **L1833 EN**: Closes the current lexical scope or compound statement.
  **L1833 CN**: 结束当前词法作用域或复合语句块。
- **L1834 EN**: Blank line separating nearby declarations or logic blocks.
  **L1834 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1835 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static mlir::NamedAttribute getAlignmentAttr(fir::FirOpBuilder &builder,`.
  **L1835 CN**: 继续一个多行参数列表、初始化器或聚合项：`static mlir::NamedAttribute getAlignmentAttr(fir::FirOpBuilder &builder,`。
- **L1836 EN**: Continues the surrounding expression or declaration: `const int val) {`.
  **L1836 CN**: 继续构造周围的表达式或声明：`const int val) {`。
- **L1837 EN**: Executes a call or declaration centered on `i64ty{mlir::IntegerType::get`.
  **L1837 CN**: 执行以 `i64ty{mlir::IntegerType::get` 为核心的调用或声明。
- **L1838 EN**: Executes a call or declaration centered on `alignAttr{mlir::IntegerAttr::get`.
  **L1838 CN**: 执行以 `alignAttr{mlir::IntegerAttr::get` 为核心的调用或声明。
- **L1839 EN**: Returns from the current function with `builder.getNamedAttr("alignment", alignAttr)`.
  **L1839 CN**: 以 `builder.getNamedAttr("alignment", alignAttr)` 从当前函数返回。
- **L1840 EN**: Closes the current lexical scope or compound statement.
  **L1840 CN**: 结束当前词法作用域或复合语句块。
- **L1841 EN**: Blank line separating nearby declarations or logic blocks.
  **L1841 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1842 EN**: Continues the surrounding expression or declaration: `fir::ExtendedValue`.
  **L1842 CN**: 继续构造周围的表达式或声明：`fir::ExtendedValue`。
- **L1843 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PPCIntrinsicLibrary::genVecXlGrp(mlir::Type resultType,`.
  **L1843 CN**: 继续一个多行参数列表、初始化器或聚合项：`PPCIntrinsicLibrary::genVecXlGrp(mlir::Type resultType,`。
- **L1844 EN**: Continues the surrounding expression or declaration: `llvm::ArrayRef<fir::ExtendedValue> args) {`.
  **L1844 CN**: 继续构造周围的表达式或声明：`llvm::ArrayRef<fir::ExtendedValue> args) {`。
- **L1845 EN**: Executes a call or declaration centered on `vecTyInfo{getVecTypeFromFirType`.
  **L1845 CN**: 执行以 `vecTyInfo{getVecTypeFromFirType` 为核心的调用或声明。
- **L1846 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L1846 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L1847 EN**: Introduces a switch dispatch label: `case 8:`.
  **L1847 CN**: 引入一个 switch 分发标签：`case 8:`。
- **L1848 EN**: Comment explains nearby logic, intent, or metadata: `vec_xlb1`.
  **L1848 CN**: 注释说明附近代码的逻辑、意图或元数据：`vec_xlb1`。

### Lines 1849-1872

````cpp
    return genVecLdNoCallGrp<VecOp::Xl>(resultType, args);
  case 16:
    // vec_xlh8
    return genVecLdNoCallGrp<VecOp::Xl>(resultType, args);
  case 32:
    // vec_xlw4
    return genVecLdCallGrp<VecOp::Xlw4>(resultType, args);
  case 64:
    // vec_xld2
    return genVecLdCallGrp<VecOp::Xld2>(resultType, args);
  default:
    llvm_unreachable("invalid kind");
  }
  llvm_unreachable("invalid vector operation for generator");
}

template <VecOp vop>
fir::ExtendedValue PPCIntrinsicLibrary::genVecLdNoCallGrp(
    mlir::Type resultType, llvm::ArrayRef<fir::ExtendedValue> args) {
  assert(args.size() == 2);
  auto arg0{getBase(args[0])};
  auto arg1{getBase(args[1])};

  auto vecTyInfo{getVecTypeFromFirType(resultType)};
````
- **L1849 EN**: Returns from the current function with `genVecLdNoCallGrp<VecOp::Xl>(resultType, args)`.
  **L1849 CN**: 以 `genVecLdNoCallGrp<VecOp::Xl>(resultType, args)` 从当前函数返回。
- **L1850 EN**: Introduces a switch dispatch label: `case 16:`.
  **L1850 CN**: 引入一个 switch 分发标签：`case 16:`。
- **L1851 EN**: Comment explains nearby logic, intent, or metadata: `vec_xlh8`.
  **L1851 CN**: 注释说明附近代码的逻辑、意图或元数据：`vec_xlh8`。
- **L1852 EN**: Returns from the current function with `genVecLdNoCallGrp<VecOp::Xl>(resultType, args)`.
  **L1852 CN**: 以 `genVecLdNoCallGrp<VecOp::Xl>(resultType, args)` 从当前函数返回。
- **L1853 EN**: Introduces a switch dispatch label: `case 32:`.
  **L1853 CN**: 引入一个 switch 分发标签：`case 32:`。
- **L1854 EN**: Comment explains nearby logic, intent, or metadata: `vec_xlw4`.
  **L1854 CN**: 注释说明附近代码的逻辑、意图或元数据：`vec_xlw4`。
- **L1855 EN**: Returns from the current function with `genVecLdCallGrp<VecOp::Xlw4>(resultType, args)`.
  **L1855 CN**: 以 `genVecLdCallGrp<VecOp::Xlw4>(resultType, args)` 从当前函数返回。
- **L1856 EN**: Introduces a switch dispatch label: `case 64:`.
  **L1856 CN**: 引入一个 switch 分发标签：`case 64:`。
- **L1857 EN**: Comment explains nearby logic, intent, or metadata: `vec_xld2`.
  **L1857 CN**: 注释说明附近代码的逻辑、意图或元数据：`vec_xld2`。
- **L1858 EN**: Returns from the current function with `genVecLdCallGrp<VecOp::Xld2>(resultType, args)`.
  **L1858 CN**: 以 `genVecLdCallGrp<VecOp::Xld2>(resultType, args)` 从当前函数返回。
- **L1859 EN**: Introduces a switch dispatch label: `default:`.
  **L1859 CN**: 引入一个 switch 分发标签：`default:`。
- **L1860 EN**: Marks this control path as unreachable to LLVM.
  **L1860 CN**: 将该控制路径标记为 LLVM 认为不可达。
- **L1861 EN**: Closes the current lexical scope or compound statement.
  **L1861 CN**: 结束当前词法作用域或复合语句块。
- **L1862 EN**: Marks this control path as unreachable to LLVM.
  **L1862 CN**: 将该控制路径标记为 LLVM 认为不可达。
- **L1863 EN**: Closes the current lexical scope or compound statement.
  **L1863 CN**: 结束当前词法作用域或复合语句块。
- **L1864 EN**: Blank line separating nearby declarations or logic blocks.
  **L1864 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1865 EN**: Introduces template parameters or specialization context: `template <VecOp vop>`.
  **L1865 CN**: 为后续声明引入模板参数或特化上下文：`template <VecOp vop>`。
- **L1866 EN**: Continues logic associated with callable symbol `genVecLdNoCallGrp`.
  **L1866 CN**: 继续与可调用符号 `genVecLdNoCallGrp` 相关的逻辑。
- **L1867 EN**: Continues the surrounding expression or declaration: `mlir::Type resultType, llvm::ArrayRef<fir::ExtendedValue> args) {`.
  **L1867 CN**: 继续构造周围的表达式或声明：`mlir::Type resultType, llvm::ArrayRef<fir::ExtendedValue> args) {`。
- **L1868 EN**: Checks an internal invariant in debug builds.
  **L1868 CN**: 在调试构建中检查内部不变式。
- **L1869 EN**: Executes a call or declaration centered on `arg0{getBase`.
  **L1869 CN**: 执行以 `arg0{getBase` 为核心的调用或声明。
- **L1870 EN**: Executes a call or declaration centered on `arg1{getBase`.
  **L1870 CN**: 执行以 `arg1{getBase` 为核心的调用或声明。
- **L1871 EN**: Blank line separating nearby declarations or logic blocks.
  **L1871 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1872 EN**: Executes a call or declaration centered on `vecTyInfo{getVecTypeFromFirType`.
  **L1872 CN**: 执行以 `vecTyInfo{getVecTypeFromFirType` 为核心的调用或声明。

### Lines 1873-1896

````cpp
  auto mlirTy{vecTyInfo.toMlirVectorType(builder.getContext())};
  auto firTy{vecTyInfo.toFirVectorType()};

  // Add the %val of arg0 to %addr of arg1
  auto addr{addOffsetToAddress(builder, loc, arg1, arg0)};

  const auto triple{fir::getTargetTriple(builder.getModule())};
  // Need to get align 1.
  auto result{fir::LoadOp::create(builder, loc, mlirTy, addr,
                                  getAlignmentAttr(builder, 1))};
  if ((vop == VecOp::Xl && isBEVecElemOrderOnLE()) ||
      (vop == VecOp::Xlbe && triple.isLittleEndian()))
    return builder.createConvert(
        loc, firTy, reverseVectorElements(builder, loc, result, vecTyInfo.len));

  return builder.createConvert(loc, firTy, result);
}

// VEC_LD, VEC_LDE, VEC_LDL, VEC_LXVP, VEC_XLD2, VEC_XLW4
template <VecOp vop>
fir::ExtendedValue
PPCIntrinsicLibrary::genVecLdCallGrp(mlir::Type resultType,
                                     llvm::ArrayRef<fir::ExtendedValue> args) {
  assert(args.size() == 2);
````
- **L1873 EN**: Executes a call or declaration centered on `mlirTy{vecTyInfo.toMlirVectorType`.
  **L1873 CN**: 执行以 `mlirTy{vecTyInfo.toMlirVectorType` 为核心的调用或声明。
- **L1874 EN**: Executes a call or declaration centered on `firTy{vecTyInfo.toFirVectorType`.
  **L1874 CN**: 执行以 `firTy{vecTyInfo.toFirVectorType` 为核心的调用或声明。
- **L1875 EN**: Blank line separating nearby declarations or logic blocks.
  **L1875 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1876 EN**: Comment explains nearby logic, intent, or metadata: `Add the %val of arg0 to %addr of arg1`.
  **L1876 CN**: 注释说明附近代码的逻辑、意图或元数据：`Add the %val of arg0 to %addr of arg1`。
- **L1877 EN**: Executes a call or declaration centered on `addr{addOffsetToAddress`.
  **L1877 CN**: 执行以 `addr{addOffsetToAddress` 为核心的调用或声明。
- **L1878 EN**: Blank line separating nearby declarations or logic blocks.
  **L1878 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1879 EN**: Executes a call or declaration centered on `triple{fir::getTargetTriple`.
  **L1879 CN**: 执行以 `triple{fir::getTargetTriple` 为核心的调用或声明。
- **L1880 EN**: Comment explains nearby logic, intent, or metadata: `Need to get align 1.`.
  **L1880 CN**: 注释说明附近代码的逻辑、意图或元数据：`Need to get align 1.`。
- **L1881 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto result{fir::LoadOp::create(builder, loc, mlirTy, addr,`.
  **L1881 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto result{fir::LoadOp::create(builder, loc, mlirTy, addr,`。
- **L1882 EN**: Executes a call or declaration centered on `getAlignmentAttr`.
  **L1882 CN**: 执行以 `getAlignmentAttr` 为核心的调用或声明。
- **L1883 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1883 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1884 EN**: Continues logic associated with callable symbol `isLittleEndian`.
  **L1884 CN**: 继续与可调用符号 `isLittleEndian` 相关的逻辑。
- **L1885 EN**: Returns from the current function with `builder.createConvert(`.
  **L1885 CN**: 以 `builder.createConvert(` 从当前函数返回。
- **L1886 EN**: Executes a call or declaration centered on `reverseVectorElements`.
  **L1886 CN**: 执行以 `reverseVectorElements` 为核心的调用或声明。
- **L1887 EN**: Blank line separating nearby declarations or logic blocks.
  **L1887 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1888 EN**: Returns from the current function with `builder.createConvert(loc, firTy, result)`.
  **L1888 CN**: 以 `builder.createConvert(loc, firTy, result)` 从当前函数返回。
- **L1889 EN**: Closes the current lexical scope or compound statement.
  **L1889 CN**: 结束当前词法作用域或复合语句块。
- **L1890 EN**: Blank line separating nearby declarations or logic blocks.
  **L1890 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1891 EN**: Comment explains nearby logic, intent, or metadata: `VEC_LD, VEC_LDE, VEC_LDL, VEC_LXVP, VEC_XLD2, VEC_XLW4`.
  **L1891 CN**: 注释说明附近代码的逻辑、意图或元数据：`VEC_LD, VEC_LDE, VEC_LDL, VEC_LXVP, VEC_XLD2, VEC_XLW4`。
- **L1892 EN**: Introduces template parameters or specialization context: `template <VecOp vop>`.
  **L1892 CN**: 为后续声明引入模板参数或特化上下文：`template <VecOp vop>`。
- **L1893 EN**: Continues the surrounding expression or declaration: `fir::ExtendedValue`.
  **L1893 CN**: 继续构造周围的表达式或声明：`fir::ExtendedValue`。
- **L1894 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PPCIntrinsicLibrary::genVecLdCallGrp(mlir::Type resultType,`.
  **L1894 CN**: 继续一个多行参数列表、初始化器或聚合项：`PPCIntrinsicLibrary::genVecLdCallGrp(mlir::Type resultType,`。
- **L1895 EN**: Continues the surrounding expression or declaration: `llvm::ArrayRef<fir::ExtendedValue> args) {`.
  **L1895 CN**: 继续构造周围的表达式或声明：`llvm::ArrayRef<fir::ExtendedValue> args) {`。
- **L1896 EN**: Checks an internal invariant in debug builds.
  **L1896 CN**: 在调试构建中检查内部不变式。

### Lines 1897-1920

````cpp
  auto context{builder.getContext()};
  auto arg0{getBase(args[0])};
  auto arg1{getBase(args[1])};

  // Prepare the return type in FIR.
  auto vecResTyInfo{getVecTypeFromFirType(resultType)};
  auto mlirTy{vecResTyInfo.toMlirVectorType(context)};
  auto firTy{vecResTyInfo.toFirVectorType()};

  // llvm.ppc.altivec.lvx* returns <4xi32>
  // Others, like "llvm.ppc.altivec.lvebx" too if arg2 is not of Integer type
  const auto i32Ty{mlir::IntegerType::get(builder.getContext(), 32)};
  const auto mVecI32Ty{mlir::VectorType::get(4, i32Ty)};

  // For vec_ld, need to convert arg0 from i64 to i32
  if (vop == VecOp::Ld && arg0.getType().getIntOrFloatBitWidth() == 64)
    arg0 = builder.createConvert(loc, i32Ty, arg0);

  // Add the %val of arg0 to %addr of arg1
  auto addr{addOffsetToAddress(builder, loc, arg1, arg0)};
  llvm::SmallVector<mlir::Value, 4> parsedArgs{addr};

  mlir::Type intrinResTy{nullptr};
  llvm::StringRef fname{};
````
- **L1897 EN**: Executes a call or declaration centered on `context{builder.getContext`.
  **L1897 CN**: 执行以 `context{builder.getContext` 为核心的调用或声明。
- **L1898 EN**: Executes a call or declaration centered on `arg0{getBase`.
  **L1898 CN**: 执行以 `arg0{getBase` 为核心的调用或声明。
- **L1899 EN**: Executes a call or declaration centered on `arg1{getBase`.
  **L1899 CN**: 执行以 `arg1{getBase` 为核心的调用或声明。
- **L1900 EN**: Blank line separating nearby declarations or logic blocks.
  **L1900 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1901 EN**: Comment explains nearby logic, intent, or metadata: `Prepare the return type in FIR.`.
  **L1901 CN**: 注释说明附近代码的逻辑、意图或元数据：`Prepare the return type in FIR.`。
- **L1902 EN**: Executes a call or declaration centered on `vecResTyInfo{getVecTypeFromFirType`.
  **L1902 CN**: 执行以 `vecResTyInfo{getVecTypeFromFirType` 为核心的调用或声明。
- **L1903 EN**: Executes a call or declaration centered on `mlirTy{vecResTyInfo.toMlirVectorType`.
  **L1903 CN**: 执行以 `mlirTy{vecResTyInfo.toMlirVectorType` 为核心的调用或声明。
- **L1904 EN**: Executes a call or declaration centered on `firTy{vecResTyInfo.toFirVectorType`.
  **L1904 CN**: 执行以 `firTy{vecResTyInfo.toFirVectorType` 为核心的调用或声明。
- **L1905 EN**: Blank line separating nearby declarations or logic blocks.
  **L1905 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1906 EN**: Comment explains nearby logic, intent, or metadata: `llvm.ppc.altivec.lvx* returns <4xi32>`.
  **L1906 CN**: 注释说明附近代码的逻辑、意图或元数据：`llvm.ppc.altivec.lvx* returns <4xi32>`。
- **L1907 EN**: Comment explains nearby logic, intent, or metadata: `Others, like "llvm.ppc.altivec.lvebx" too if arg2 is not of Integer type`.
  **L1907 CN**: 注释说明附近代码的逻辑、意图或元数据：`Others, like "llvm.ppc.altivec.lvebx" too if arg2 is not of Integer type`。
- **L1908 EN**: Executes a call or declaration centered on `i32Ty{mlir::IntegerType::get`.
  **L1908 CN**: 执行以 `i32Ty{mlir::IntegerType::get` 为核心的调用或声明。
- **L1909 EN**: Executes a call or declaration centered on `mVecI32Ty{mlir::VectorType::get`.
  **L1909 CN**: 执行以 `mVecI32Ty{mlir::VectorType::get` 为核心的调用或声明。
- **L1910 EN**: Blank line separating nearby declarations or logic blocks.
  **L1910 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1911 EN**: Comment explains nearby logic, intent, or metadata: `For vec_ld, need to convert arg0 from i64 to i32`.
  **L1911 CN**: 注释说明附近代码的逻辑、意图或元数据：`For vec_ld, need to convert arg0 from i64 to i32`。
- **L1912 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1912 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1913 EN**: Executes a call or declaration centered on `builder.createConvert`.
  **L1913 CN**: 执行以 `builder.createConvert` 为核心的调用或声明。
- **L1914 EN**: Blank line separating nearby declarations or logic blocks.
  **L1914 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1915 EN**: Comment explains nearby logic, intent, or metadata: `Add the %val of arg0 to %addr of arg1`.
  **L1915 CN**: 注释说明附近代码的逻辑、意图或元数据：`Add the %val of arg0 to %addr of arg1`。
- **L1916 EN**: Executes a call or declaration centered on `addr{addOffsetToAddress`.
  **L1916 CN**: 执行以 `addr{addOffsetToAddress` 为核心的调用或声明。
- **L1917 EN**: Executes a standalone statement or declaration: `llvm::SmallVector<mlir::Value, 4> parsedArgs{addr};`.
  **L1917 CN**: 执行一条独立语句或声明：`llvm::SmallVector<mlir::Value, 4> parsedArgs{addr};`。
- **L1918 EN**: Blank line separating nearby declarations or logic blocks.
  **L1918 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1919 EN**: Executes a standalone statement or declaration: `mlir::Type intrinResTy{nullptr};`.
  **L1919 CN**: 执行一条独立语句或声明：`mlir::Type intrinResTy{nullptr};`。
- **L1920 EN**: Executes a standalone statement or declaration: `llvm::StringRef fname{};`.
  **L1920 CN**: 执行一条独立语句或声明：`llvm::StringRef fname{};`。

### Lines 1921-1944

````cpp
  switch (vop) {
  case VecOp::Ld:
    fname = "llvm.ppc.altivec.lvx";
    intrinResTy = mVecI32Ty;
    break;
  case VecOp::Lde:
    switch (vecResTyInfo.eleTy.getIntOrFloatBitWidth()) {
    case 8:
      fname = "llvm.ppc.altivec.lvebx";
      intrinResTy = mlirTy;
      break;
    case 16:
      fname = "llvm.ppc.altivec.lvehx";
      intrinResTy = mlirTy;
      break;
    case 32:
      fname = "llvm.ppc.altivec.lvewx";
      if (mlir::isa<mlir::IntegerType>(vecResTyInfo.eleTy))
        intrinResTy = mlirTy;
      else
        intrinResTy = mVecI32Ty;
      break;
    default:
      llvm_unreachable("invalid vector for vec_lde");
````
- **L1921 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L1921 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L1922 EN**: Introduces a switch dispatch label: `case VecOp::Ld:`.
  **L1922 CN**: 引入一个 switch 分发标签：`case VecOp::Ld:`。
- **L1923 EN**: Executes a standalone statement or declaration: `fname = "llvm.ppc.altivec.lvx";`.
  **L1923 CN**: 执行一条独立语句或声明：`fname = "llvm.ppc.altivec.lvx";`。
- **L1924 EN**: Executes a standalone statement or declaration: `intrinResTy = mVecI32Ty;`.
  **L1924 CN**: 执行一条独立语句或声明：`intrinResTy = mVecI32Ty;`。
- **L1925 EN**: Exits the nearest loop or switch statement.
  **L1925 CN**: 退出最近的循环或 switch 语句。
- **L1926 EN**: Introduces a switch dispatch label: `case VecOp::Lde:`.
  **L1926 CN**: 引入一个 switch 分发标签：`case VecOp::Lde:`。
- **L1927 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L1927 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L1928 EN**: Introduces a switch dispatch label: `case 8:`.
  **L1928 CN**: 引入一个 switch 分发标签：`case 8:`。
- **L1929 EN**: Executes a standalone statement or declaration: `fname = "llvm.ppc.altivec.lvebx";`.
  **L1929 CN**: 执行一条独立语句或声明：`fname = "llvm.ppc.altivec.lvebx";`。
- **L1930 EN**: Executes a standalone statement or declaration: `intrinResTy = mlirTy;`.
  **L1930 CN**: 执行一条独立语句或声明：`intrinResTy = mlirTy;`。
- **L1931 EN**: Exits the nearest loop or switch statement.
  **L1931 CN**: 退出最近的循环或 switch 语句。
- **L1932 EN**: Introduces a switch dispatch label: `case 16:`.
  **L1932 CN**: 引入一个 switch 分发标签：`case 16:`。
- **L1933 EN**: Executes a standalone statement or declaration: `fname = "llvm.ppc.altivec.lvehx";`.
  **L1933 CN**: 执行一条独立语句或声明：`fname = "llvm.ppc.altivec.lvehx";`。
- **L1934 EN**: Executes a standalone statement or declaration: `intrinResTy = mlirTy;`.
  **L1934 CN**: 执行一条独立语句或声明：`intrinResTy = mlirTy;`。
- **L1935 EN**: Exits the nearest loop or switch statement.
  **L1935 CN**: 退出最近的循环或 switch 语句。
- **L1936 EN**: Introduces a switch dispatch label: `case 32:`.
  **L1936 CN**: 引入一个 switch 分发标签：`case 32:`。
- **L1937 EN**: Executes a standalone statement or declaration: `fname = "llvm.ppc.altivec.lvewx";`.
  **L1937 CN**: 执行一条独立语句或声明：`fname = "llvm.ppc.altivec.lvewx";`。
- **L1938 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1938 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1939 EN**: Executes a standalone statement or declaration: `intrinResTy = mlirTy;`.
  **L1939 CN**: 执行一条独立语句或声明：`intrinResTy = mlirTy;`。
- **L1940 EN**: Transitions from the previous branch into the alternative path.
  **L1940 CN**: 从前一个分支过渡到备选路径。
- **L1941 EN**: Executes a standalone statement or declaration: `intrinResTy = mVecI32Ty;`.
  **L1941 CN**: 执行一条独立语句或声明：`intrinResTy = mVecI32Ty;`。
- **L1942 EN**: Exits the nearest loop or switch statement.
  **L1942 CN**: 退出最近的循环或 switch 语句。
- **L1943 EN**: Introduces a switch dispatch label: `default:`.
  **L1943 CN**: 引入一个 switch 分发标签：`default:`。
- **L1944 EN**: Marks this control path as unreachable to LLVM.
  **L1944 CN**: 将该控制路径标记为 LLVM 认为不可达。

### Lines 1945-1968

````cpp
    }
    break;
  case VecOp::Ldl:
    fname = "llvm.ppc.altivec.lvxl";
    intrinResTy = mVecI32Ty;
    break;
  case VecOp::Lxvp:
    fname = "llvm.ppc.vsx.lxvp";
    intrinResTy = fir::VectorType::get(256, mlir::IntegerType::get(context, 1));
    break;
  case VecOp::Xld2: {
    fname = isBEVecElemOrderOnLE() ? "llvm.ppc.vsx.lxvd2x.be"
                                   : "llvm.ppc.vsx.lxvd2x";
    // llvm.ppc.altivec.lxvd2x* returns <2 x double>
    intrinResTy = mlir::VectorType::get(2, mlir::Float64Type::get(context));
  } break;
  case VecOp::Xlw4:
    fname = isBEVecElemOrderOnLE() ? "llvm.ppc.vsx.lxvw4x.be"
                                   : "llvm.ppc.vsx.lxvw4x";
    // llvm.ppc.altivec.lxvw4x* returns <4xi32>
    intrinResTy = mVecI32Ty;
    break;
  default:
    llvm_unreachable("invalid vector operation for generator");
````
- **L1945 EN**: Closes the current lexical scope or compound statement.
  **L1945 CN**: 结束当前词法作用域或复合语句块。
- **L1946 EN**: Exits the nearest loop or switch statement.
  **L1946 CN**: 退出最近的循环或 switch 语句。
- **L1947 EN**: Introduces a switch dispatch label: `case VecOp::Ldl:`.
  **L1947 CN**: 引入一个 switch 分发标签：`case VecOp::Ldl:`。
- **L1948 EN**: Executes a standalone statement or declaration: `fname = "llvm.ppc.altivec.lvxl";`.
  **L1948 CN**: 执行一条独立语句或声明：`fname = "llvm.ppc.altivec.lvxl";`。
- **L1949 EN**: Executes a standalone statement or declaration: `intrinResTy = mVecI32Ty;`.
  **L1949 CN**: 执行一条独立语句或声明：`intrinResTy = mVecI32Ty;`。
- **L1950 EN**: Exits the nearest loop or switch statement.
  **L1950 CN**: 退出最近的循环或 switch 语句。
- **L1951 EN**: Introduces a switch dispatch label: `case VecOp::Lxvp:`.
  **L1951 CN**: 引入一个 switch 分发标签：`case VecOp::Lxvp:`。
- **L1952 EN**: Executes a standalone statement or declaration: `fname = "llvm.ppc.vsx.lxvp";`.
  **L1952 CN**: 执行一条独立语句或声明：`fname = "llvm.ppc.vsx.lxvp";`。
- **L1953 EN**: Executes a call or declaration centered on `fir::VectorType::get`.
  **L1953 CN**: 执行以 `fir::VectorType::get` 为核心的调用或声明。
- **L1954 EN**: Exits the nearest loop or switch statement.
  **L1954 CN**: 退出最近的循环或 switch 语句。
- **L1955 EN**: Introduces a switch dispatch label: `case VecOp::Xld2: {`.
  **L1955 CN**: 引入一个 switch 分发标签：`case VecOp::Xld2: {`。
- **L1956 EN**: Continues logic associated with callable symbol `isBEVecElemOrderOnLE`.
  **L1956 CN**: 继续与可调用符号 `isBEVecElemOrderOnLE` 相关的逻辑。
- **L1957 EN**: Executes a standalone statement or declaration: `: "llvm.ppc.vsx.lxvd2x";`.
  **L1957 CN**: 执行一条独立语句或声明：`: "llvm.ppc.vsx.lxvd2x";`。
- **L1958 EN**: Comment explains nearby logic, intent, or metadata: `llvm.ppc.altivec.lxvd2x* returns <2 x double>`.
  **L1958 CN**: 注释说明附近代码的逻辑、意图或元数据：`llvm.ppc.altivec.lxvd2x* returns <2 x double>`。
- **L1959 EN**: Executes a call or declaration centered on `mlir::VectorType::get`.
  **L1959 CN**: 执行以 `mlir::VectorType::get` 为核心的调用或声明。
- **L1960 EN**: Executes a standalone statement or declaration: `} break;`.
  **L1960 CN**: 执行一条独立语句或声明：`} break;`。
- **L1961 EN**: Introduces a switch dispatch label: `case VecOp::Xlw4:`.
  **L1961 CN**: 引入一个 switch 分发标签：`case VecOp::Xlw4:`。
- **L1962 EN**: Continues logic associated with callable symbol `isBEVecElemOrderOnLE`.
  **L1962 CN**: 继续与可调用符号 `isBEVecElemOrderOnLE` 相关的逻辑。
- **L1963 EN**: Executes a standalone statement or declaration: `: "llvm.ppc.vsx.lxvw4x";`.
  **L1963 CN**: 执行一条独立语句或声明：`: "llvm.ppc.vsx.lxvw4x";`。
- **L1964 EN**: Comment explains nearby logic, intent, or metadata: `llvm.ppc.altivec.lxvw4x* returns <4xi32>`.
  **L1964 CN**: 注释说明附近代码的逻辑、意图或元数据：`llvm.ppc.altivec.lxvw4x* returns <4xi32>`。
- **L1965 EN**: Executes a standalone statement or declaration: `intrinResTy = mVecI32Ty;`.
  **L1965 CN**: 执行一条独立语句或声明：`intrinResTy = mVecI32Ty;`。
- **L1966 EN**: Exits the nearest loop or switch statement.
  **L1966 CN**: 退出最近的循环或 switch 语句。
- **L1967 EN**: Introduces a switch dispatch label: `default:`.
  **L1967 CN**: 引入一个 switch 分发标签：`default:`。
- **L1968 EN**: Marks this control path as unreachable to LLVM.
  **L1968 CN**: 将该控制路径标记为 LLVM 认为不可达。

### Lines 1969-1992

````cpp
  }

  auto funcType{
      mlir::FunctionType::get(context, {addr.getType()}, {intrinResTy})};
  auto funcOp{builder.createFunction(loc, fname, funcType)};
  auto result{
      fir::CallOp::create(builder, loc, funcOp, parsedArgs).getResult(0)};

  if (vop == VecOp::Lxvp)
    return result;

  if (intrinResTy != mlirTy)
    result = mlir::vector::BitCastOp::create(builder, loc, mlirTy, result);

  if (vop != VecOp::Xld2 && vop != VecOp::Xlw4 && isBEVecElemOrderOnLE())
    return builder.createConvert(
        loc, firTy,
        reverseVectorElements(builder, loc, result, vecResTyInfo.len));

  return builder.createConvert(loc, firTy, result);
}

// VEC_LVSL, VEC_LVSR
template <VecOp vop>
````
- **L1969 EN**: Closes the current lexical scope or compound statement.
  **L1969 CN**: 结束当前词法作用域或复合语句块。
- **L1970 EN**: Blank line separating nearby declarations or logic blocks.
  **L1970 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1971 EN**: Continues the surrounding expression or declaration: `auto funcType{`.
  **L1971 CN**: 继续构造周围的表达式或声明：`auto funcType{`。
- **L1972 EN**: Executes a call or declaration centered on `mlir::FunctionType::get`.
  **L1972 CN**: 执行以 `mlir::FunctionType::get` 为核心的调用或声明。
- **L1973 EN**: Executes a call or declaration centered on `funcOp{builder.createFunction`.
  **L1973 CN**: 执行以 `funcOp{builder.createFunction` 为核心的调用或声明。
- **L1974 EN**: Continues the surrounding expression or declaration: `auto result{`.
  **L1974 CN**: 继续构造周围的表达式或声明：`auto result{`。
- **L1975 EN**: Executes a call or declaration centered on `fir::CallOp::create`.
  **L1975 CN**: 执行以 `fir::CallOp::create` 为核心的调用或声明。
- **L1976 EN**: Blank line separating nearby declarations or logic blocks.
  **L1976 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1977 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1977 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1978 EN**: Returns from the current function with `result`.
  **L1978 CN**: 以 `result` 从当前函数返回。
- **L1979 EN**: Blank line separating nearby declarations or logic blocks.
  **L1979 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1980 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1980 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1981 EN**: Executes a call or declaration centered on `mlir::vector::BitCastOp::create`.
  **L1981 CN**: 执行以 `mlir::vector::BitCastOp::create` 为核心的调用或声明。
- **L1982 EN**: Blank line separating nearby declarations or logic blocks.
  **L1982 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1983 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1983 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1984 EN**: Returns from the current function with `builder.createConvert(`.
  **L1984 CN**: 以 `builder.createConvert(` 从当前函数返回。
- **L1985 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `loc, firTy,`.
  **L1985 CN**: 继续一个多行参数列表、初始化器或聚合项：`loc, firTy,`。
- **L1986 EN**: Executes a call or declaration centered on `reverseVectorElements`.
  **L1986 CN**: 执行以 `reverseVectorElements` 为核心的调用或声明。
- **L1987 EN**: Blank line separating nearby declarations or logic blocks.
  **L1987 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1988 EN**: Returns from the current function with `builder.createConvert(loc, firTy, result)`.
  **L1988 CN**: 以 `builder.createConvert(loc, firTy, result)` 从当前函数返回。
- **L1989 EN**: Closes the current lexical scope or compound statement.
  **L1989 CN**: 结束当前词法作用域或复合语句块。
- **L1990 EN**: Blank line separating nearby declarations or logic blocks.
  **L1990 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1991 EN**: Comment explains nearby logic, intent, or metadata: `VEC_LVSL, VEC_LVSR`.
  **L1991 CN**: 注释说明附近代码的逻辑、意图或元数据：`VEC_LVSL, VEC_LVSR`。
- **L1992 EN**: Introduces template parameters or specialization context: `template <VecOp vop>`.
  **L1992 CN**: 为后续声明引入模板参数或特化上下文：`template <VecOp vop>`。

### Lines 1993-2016

````cpp
fir::ExtendedValue
PPCIntrinsicLibrary::genVecLvsGrp(mlir::Type resultType,
                                  llvm::ArrayRef<fir::ExtendedValue> args) {
  assert(args.size() == 2);
  auto context{builder.getContext()};
  auto arg0{getBase(args[0])};
  auto arg1{getBase(args[1])};

  auto vecTyInfo{getVecTypeFromFirType(resultType)};
  auto mlirTy{vecTyInfo.toMlirVectorType(context)};
  auto firTy{vecTyInfo.toFirVectorType()};

  // Convert arg0 to i64 type if needed
  auto i64ty{mlir::IntegerType::get(context, 64)};
  if (arg0.getType() != i64ty)
    arg0 = fir::ConvertOp::create(builder, loc, i64ty, arg0);

  // offset is modulo 16, so shift left 56 bits and then right 56 bits to clear
  //   upper 56 bit while preserving sign
  auto shiftVal{builder.createIntegerConstant(loc, i64ty, 56)};
  auto offset{mlir::arith::ShLIOp::create(builder, loc, arg0, shiftVal)};
  auto offset2{mlir::arith::ShRSIOp::create(builder, loc, offset, shiftVal)};

  // Add the offsetArg to %addr of arg1
````
- **L1993 EN**: Continues the surrounding expression or declaration: `fir::ExtendedValue`.
  **L1993 CN**: 继续构造周围的表达式或声明：`fir::ExtendedValue`。
- **L1994 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PPCIntrinsicLibrary::genVecLvsGrp(mlir::Type resultType,`.
  **L1994 CN**: 继续一个多行参数列表、初始化器或聚合项：`PPCIntrinsicLibrary::genVecLvsGrp(mlir::Type resultType,`。
- **L1995 EN**: Continues the surrounding expression or declaration: `llvm::ArrayRef<fir::ExtendedValue> args) {`.
  **L1995 CN**: 继续构造周围的表达式或声明：`llvm::ArrayRef<fir::ExtendedValue> args) {`。
- **L1996 EN**: Checks an internal invariant in debug builds.
  **L1996 CN**: 在调试构建中检查内部不变式。
- **L1997 EN**: Executes a call or declaration centered on `context{builder.getContext`.
  **L1997 CN**: 执行以 `context{builder.getContext` 为核心的调用或声明。
- **L1998 EN**: Executes a call or declaration centered on `arg0{getBase`.
  **L1998 CN**: 执行以 `arg0{getBase` 为核心的调用或声明。
- **L1999 EN**: Executes a call or declaration centered on `arg1{getBase`.
  **L1999 CN**: 执行以 `arg1{getBase` 为核心的调用或声明。
- **L2000 EN**: Blank line separating nearby declarations or logic blocks.
  **L2000 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2001 EN**: Executes a call or declaration centered on `vecTyInfo{getVecTypeFromFirType`.
  **L2001 CN**: 执行以 `vecTyInfo{getVecTypeFromFirType` 为核心的调用或声明。
- **L2002 EN**: Executes a call or declaration centered on `mlirTy{vecTyInfo.toMlirVectorType`.
  **L2002 CN**: 执行以 `mlirTy{vecTyInfo.toMlirVectorType` 为核心的调用或声明。
- **L2003 EN**: Executes a call or declaration centered on `firTy{vecTyInfo.toFirVectorType`.
  **L2003 CN**: 执行以 `firTy{vecTyInfo.toFirVectorType` 为核心的调用或声明。
- **L2004 EN**: Blank line separating nearby declarations or logic blocks.
  **L2004 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2005 EN**: Comment explains nearby logic, intent, or metadata: `Convert arg0 to i64 type if needed`.
  **L2005 CN**: 注释说明附近代码的逻辑、意图或元数据：`Convert arg0 to i64 type if needed`。
- **L2006 EN**: Executes a call or declaration centered on `i64ty{mlir::IntegerType::get`.
  **L2006 CN**: 执行以 `i64ty{mlir::IntegerType::get` 为核心的调用或声明。
- **L2007 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2007 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2008 EN**: Executes a call or declaration centered on `fir::ConvertOp::create`.
  **L2008 CN**: 执行以 `fir::ConvertOp::create` 为核心的调用或声明。
- **L2009 EN**: Blank line separating nearby declarations or logic blocks.
  **L2009 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2010 EN**: Comment explains nearby logic, intent, or metadata: `offset is modulo 16, so shift left 56 bits and then right 56 bits to clear`.
  **L2010 CN**: 注释说明附近代码的逻辑、意图或元数据：`offset is modulo 16, so shift left 56 bits and then right 56 bits to clear`。
- **L2011 EN**: Comment explains nearby logic, intent, or metadata: `upper 56 bit while preserving sign`.
  **L2011 CN**: 注释说明附近代码的逻辑、意图或元数据：`upper 56 bit while preserving sign`。
- **L2012 EN**: Executes a call or declaration centered on `shiftVal{builder.createIntegerConstant`.
  **L2012 CN**: 执行以 `shiftVal{builder.createIntegerConstant` 为核心的调用或声明。
- **L2013 EN**: Executes a call or declaration centered on `offset{mlir::arith::ShLIOp::create`.
  **L2013 CN**: 执行以 `offset{mlir::arith::ShLIOp::create` 为核心的调用或声明。
- **L2014 EN**: Executes a call or declaration centered on `offset2{mlir::arith::ShRSIOp::create`.
  **L2014 CN**: 执行以 `offset2{mlir::arith::ShRSIOp::create` 为核心的调用或声明。
- **L2015 EN**: Blank line separating nearby declarations or logic blocks.
  **L2015 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2016 EN**: Comment explains nearby logic, intent, or metadata: `Add the offsetArg to %addr of arg1`.
  **L2016 CN**: 注释说明附近代码的逻辑、意图或元数据：`Add the offsetArg to %addr of arg1`。

### Lines 2017-2040

````cpp
  auto addr{addOffsetToAddress(builder, loc, arg1, offset2)};
  llvm::SmallVector<mlir::Value, 4> parsedArgs{addr};

  llvm::StringRef fname{};
  switch (vop) {
  case VecOp::Lvsl:
    fname = "llvm.ppc.altivec.lvsl";
    break;
  case VecOp::Lvsr:
    fname = "llvm.ppc.altivec.lvsr";
    break;
  default:
    llvm_unreachable("invalid vector operation for generator");
  }
  auto funcType{mlir::FunctionType::get(context, {addr.getType()}, {mlirTy})};
  auto funcOp{builder.createFunction(loc, fname, funcType)};
  auto result{
      fir::CallOp::create(builder, loc, funcOp, parsedArgs).getResult(0)};

  if (isNativeVecElemOrderOnLE())
    return builder.createConvert(
        loc, firTy, reverseVectorElements(builder, loc, result, vecTyInfo.len));

  return builder.createConvert(loc, firTy, result);
````
- **L2017 EN**: Executes a call or declaration centered on `addr{addOffsetToAddress`.
  **L2017 CN**: 执行以 `addr{addOffsetToAddress` 为核心的调用或声明。
- **L2018 EN**: Executes a standalone statement or declaration: `llvm::SmallVector<mlir::Value, 4> parsedArgs{addr};`.
  **L2018 CN**: 执行一条独立语句或声明：`llvm::SmallVector<mlir::Value, 4> parsedArgs{addr};`。
- **L2019 EN**: Blank line separating nearby declarations or logic blocks.
  **L2019 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2020 EN**: Executes a standalone statement or declaration: `llvm::StringRef fname{};`.
  **L2020 CN**: 执行一条独立语句或声明：`llvm::StringRef fname{};`。
- **L2021 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L2021 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L2022 EN**: Introduces a switch dispatch label: `case VecOp::Lvsl:`.
  **L2022 CN**: 引入一个 switch 分发标签：`case VecOp::Lvsl:`。
- **L2023 EN**: Executes a standalone statement or declaration: `fname = "llvm.ppc.altivec.lvsl";`.
  **L2023 CN**: 执行一条独立语句或声明：`fname = "llvm.ppc.altivec.lvsl";`。
- **L2024 EN**: Exits the nearest loop or switch statement.
  **L2024 CN**: 退出最近的循环或 switch 语句。
- **L2025 EN**: Introduces a switch dispatch label: `case VecOp::Lvsr:`.
  **L2025 CN**: 引入一个 switch 分发标签：`case VecOp::Lvsr:`。
- **L2026 EN**: Executes a standalone statement or declaration: `fname = "llvm.ppc.altivec.lvsr";`.
  **L2026 CN**: 执行一条独立语句或声明：`fname = "llvm.ppc.altivec.lvsr";`。
- **L2027 EN**: Exits the nearest loop or switch statement.
  **L2027 CN**: 退出最近的循环或 switch 语句。
- **L2028 EN**: Introduces a switch dispatch label: `default:`.
  **L2028 CN**: 引入一个 switch 分发标签：`default:`。
- **L2029 EN**: Marks this control path as unreachable to LLVM.
  **L2029 CN**: 将该控制路径标记为 LLVM 认为不可达。
- **L2030 EN**: Closes the current lexical scope or compound statement.
  **L2030 CN**: 结束当前词法作用域或复合语句块。
- **L2031 EN**: Executes a call or declaration centered on `funcType{mlir::FunctionType::get`.
  **L2031 CN**: 执行以 `funcType{mlir::FunctionType::get` 为核心的调用或声明。
- **L2032 EN**: Executes a call or declaration centered on `funcOp{builder.createFunction`.
  **L2032 CN**: 执行以 `funcOp{builder.createFunction` 为核心的调用或声明。
- **L2033 EN**: Continues the surrounding expression or declaration: `auto result{`.
  **L2033 CN**: 继续构造周围的表达式或声明：`auto result{`。
- **L2034 EN**: Executes a call or declaration centered on `fir::CallOp::create`.
  **L2034 CN**: 执行以 `fir::CallOp::create` 为核心的调用或声明。
- **L2035 EN**: Blank line separating nearby declarations or logic blocks.
  **L2035 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2036 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2036 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2037 EN**: Returns from the current function with `builder.createConvert(`.
  **L2037 CN**: 以 `builder.createConvert(` 从当前函数返回。
- **L2038 EN**: Executes a call or declaration centered on `reverseVectorElements`.
  **L2038 CN**: 执行以 `reverseVectorElements` 为核心的调用或声明。
- **L2039 EN**: Blank line separating nearby declarations or logic blocks.
  **L2039 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2040 EN**: Returns from the current function with `builder.createConvert(loc, firTy, result)`.
  **L2040 CN**: 以 `builder.createConvert(loc, firTy, result)` 从当前函数返回。

### Lines 2041-2064

````cpp
}

// VEC_NMADD, VEC_MSUB
template <VecOp vop>
fir::ExtendedValue
PPCIntrinsicLibrary::genVecNmaddMsub(mlir::Type resultType,
                                     llvm::ArrayRef<fir::ExtendedValue> args) {
  assert(args.size() == 3);
  auto context{builder.getContext()};
  auto argBases{getBasesForArgs(args)};
  auto vTypeInfo{getVecTypeFromFir(argBases[0])};
  auto newArgs{convertVecArgs(builder, loc, vTypeInfo, argBases)};
  const auto width{vTypeInfo.eleTy.getIntOrFloatBitWidth()};

  static std::map<int, std::pair<llvm::StringRef, mlir::FunctionType>> fmaMap{
      {32,
       std::make_pair(
           "llvm.fma.v4f32",
           genFuncType<Ty::RealVector<4>, Ty::RealVector<4>, Ty::RealVector<4>>(
               context, builder))},
      {64,
       std::make_pair(
           "llvm.fma.v2f64",
           genFuncType<Ty::RealVector<8>, Ty::RealVector<8>, Ty::RealVector<8>>(
````
- **L2041 EN**: Closes the current lexical scope or compound statement.
  **L2041 CN**: 结束当前词法作用域或复合语句块。
- **L2042 EN**: Blank line separating nearby declarations or logic blocks.
  **L2042 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2043 EN**: Comment explains nearby logic, intent, or metadata: `VEC_NMADD, VEC_MSUB`.
  **L2043 CN**: 注释说明附近代码的逻辑、意图或元数据：`VEC_NMADD, VEC_MSUB`。
- **L2044 EN**: Introduces template parameters or specialization context: `template <VecOp vop>`.
  **L2044 CN**: 为后续声明引入模板参数或特化上下文：`template <VecOp vop>`。
- **L2045 EN**: Continues the surrounding expression or declaration: `fir::ExtendedValue`.
  **L2045 CN**: 继续构造周围的表达式或声明：`fir::ExtendedValue`。
- **L2046 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PPCIntrinsicLibrary::genVecNmaddMsub(mlir::Type resultType,`.
  **L2046 CN**: 继续一个多行参数列表、初始化器或聚合项：`PPCIntrinsicLibrary::genVecNmaddMsub(mlir::Type resultType,`。
- **L2047 EN**: Continues the surrounding expression or declaration: `llvm::ArrayRef<fir::ExtendedValue> args) {`.
  **L2047 CN**: 继续构造周围的表达式或声明：`llvm::ArrayRef<fir::ExtendedValue> args) {`。
- **L2048 EN**: Checks an internal invariant in debug builds.
  **L2048 CN**: 在调试构建中检查内部不变式。
- **L2049 EN**: Executes a call or declaration centered on `context{builder.getContext`.
  **L2049 CN**: 执行以 `context{builder.getContext` 为核心的调用或声明。
- **L2050 EN**: Executes a call or declaration centered on `argBases{getBasesForArgs`.
  **L2050 CN**: 执行以 `argBases{getBasesForArgs` 为核心的调用或声明。
- **L2051 EN**: Executes a call or declaration centered on `vTypeInfo{getVecTypeFromFir`.
  **L2051 CN**: 执行以 `vTypeInfo{getVecTypeFromFir` 为核心的调用或声明。
- **L2052 EN**: Executes a call or declaration centered on `newArgs{convertVecArgs`.
  **L2052 CN**: 执行以 `newArgs{convertVecArgs` 为核心的调用或声明。
- **L2053 EN**: Executes a call or declaration centered on `width{vTypeInfo.eleTy.getIntOrFloatBitWidth`.
  **L2053 CN**: 执行以 `width{vTypeInfo.eleTy.getIntOrFloatBitWidth` 为核心的调用或声明。
- **L2054 EN**: Blank line separating nearby declarations or logic blocks.
  **L2054 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2055 EN**: Continues the surrounding expression or declaration: `static std::map<int, std::pair<llvm::StringRef, mlir::FunctionType>> fmaMap{`.
  **L2055 CN**: 继续构造周围的表达式或声明：`static std::map<int, std::pair<llvm::StringRef, mlir::FunctionType>> fmaMap{`。
- **L2056 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{32,`.
  **L2056 CN**: 继续一个多行参数列表、初始化器或聚合项：`{32,`。
- **L2057 EN**: Continues logic associated with callable symbol `make_pair`.
  **L2057 CN**: 继续与可调用符号 `make_pair` 相关的逻辑。
- **L2058 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"llvm.fma.v4f32",`.
  **L2058 CN**: 继续一个多行参数列表、初始化器或聚合项：`"llvm.fma.v4f32",`。
- **L2059 EN**: Continues logic associated with callable symbol `RealVector<4>>`.
  **L2059 CN**: 继续与可调用符号 `RealVector<4>>` 相关的逻辑。
- **L2060 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context, builder))},`.
  **L2060 CN**: 继续一个多行参数列表、初始化器或聚合项：`context, builder))},`。
- **L2061 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{64,`.
  **L2061 CN**: 继续一个多行参数列表、初始化器或聚合项：`{64,`。
- **L2062 EN**: Continues logic associated with callable symbol `make_pair`.
  **L2062 CN**: 继续与可调用符号 `make_pair` 相关的逻辑。
- **L2063 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"llvm.fma.v2f64",`.
  **L2063 CN**: 继续一个多行参数列表、初始化器或聚合项：`"llvm.fma.v2f64",`。
- **L2064 EN**: Continues logic associated with callable symbol `RealVector<8>>`.
  **L2064 CN**: 继续与可调用符号 `RealVector<8>>` 相关的逻辑。

### Lines 2065-2088

````cpp
               context, builder))}};

  auto funcOp{builder.createFunction(loc, std::get<0>(fmaMap[width]),
                                     std::get<1>(fmaMap[width]))};
  if (vop == VecOp::Nmadd) {
    // vec_nmadd(arg1, arg2, arg3) = -fma(arg1, arg2, arg3)
    auto callOp{fir::CallOp::create(builder, loc, funcOp, newArgs)};

    // We need to convert fir.vector to MLIR vector to use fneg and then back
    // to fir.vector to store.
    auto vCall{builder.createConvert(loc, vTypeInfo.toMlirVectorType(context),
                                     callOp.getResult(0))};
    auto neg{mlir::arith::NegFOp::create(builder, loc, vCall)};
    return builder.createConvert(loc, vTypeInfo.toFirVectorType(), neg);
  } else if (vop == VecOp::Msub) {
    // vec_msub(arg1, arg2, arg3) = fma(arg1, arg2, -arg3)
    newArgs[2] = mlir::arith::NegFOp::create(builder, loc, newArgs[2]);

    auto callOp{fir::CallOp::create(builder, loc, funcOp, newArgs)};
    return callOp.getResult(0);
  }
  llvm_unreachable("Invalid vector operation for generator");
}

````
- **L2065 EN**: Executes a standalone statement or declaration: `context, builder))}};`.
  **L2065 CN**: 执行一条独立语句或声明：`context, builder))}};`。
- **L2066 EN**: Blank line separating nearby declarations or logic blocks.
  **L2066 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2067 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto funcOp{builder.createFunction(loc, std::get<0>(fmaMap[width]),`.
  **L2067 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto funcOp{builder.createFunction(loc, std::get<0>(fmaMap[width]),`。
- **L2068 EN**: Executes a call or declaration centered on `std::get<1>`.
  **L2068 CN**: 执行以 `std::get<1>` 为核心的调用或声明。
- **L2069 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2069 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2070 EN**: Comment explains nearby logic, intent, or metadata: `vec_nmadd(arg1, arg2, arg3) = -fma(arg1, arg2, arg3)`.
  **L2070 CN**: 注释说明附近代码的逻辑、意图或元数据：`vec_nmadd(arg1, arg2, arg3) = -fma(arg1, arg2, arg3)`。
- **L2071 EN**: Executes a call or declaration centered on `callOp{fir::CallOp::create`.
  **L2071 CN**: 执行以 `callOp{fir::CallOp::create` 为核心的调用或声明。
- **L2072 EN**: Blank line separating nearby declarations or logic blocks.
  **L2072 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2073 EN**: Comment explains nearby logic, intent, or metadata: `We need to convert fir.vector to MLIR vector to use fneg and then back`.
  **L2073 CN**: 注释说明附近代码的逻辑、意图或元数据：`We need to convert fir.vector to MLIR vector to use fneg and then back`。
- **L2074 EN**: Comment explains nearby logic, intent, or metadata: `to fir.vector to store.`.
  **L2074 CN**: 注释说明附近代码的逻辑、意图或元数据：`to fir.vector to store.`。
- **L2075 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto vCall{builder.createConvert(loc, vTypeInfo.toMlirVectorType(context),`.
  **L2075 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto vCall{builder.createConvert(loc, vTypeInfo.toMlirVectorType(context),`。
- **L2076 EN**: Executes a call or declaration centered on `callOp.getResult`.
  **L2076 CN**: 执行以 `callOp.getResult` 为核心的调用或声明。
- **L2077 EN**: Executes a call or declaration centered on `neg{mlir::arith::NegFOp::create`.
  **L2077 CN**: 执行以 `neg{mlir::arith::NegFOp::create` 为核心的调用或声明。
- **L2078 EN**: Returns from the current function with `builder.createConvert(loc, vTypeInfo.toFirVectorType(), neg)`.
  **L2078 CN**: 以 `builder.createConvert(loc, vTypeInfo.toFirVectorType(), neg)` 从当前函数返回。
- **L2079 EN**: Transitions from the previous branch into an `else if` condition.
  **L2079 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2080 EN**: Comment explains nearby logic, intent, or metadata: `vec_msub(arg1, arg2, arg3) = fma(arg1, arg2, -arg3)`.
  **L2080 CN**: 注释说明附近代码的逻辑、意图或元数据：`vec_msub(arg1, arg2, arg3) = fma(arg1, arg2, -arg3)`。
- **L2081 EN**: Executes a call or declaration centered on `mlir::arith::NegFOp::create`.
  **L2081 CN**: 执行以 `mlir::arith::NegFOp::create` 为核心的调用或声明。
- **L2082 EN**: Blank line separating nearby declarations or logic blocks.
  **L2082 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2083 EN**: Executes a call or declaration centered on `callOp{fir::CallOp::create`.
  **L2083 CN**: 执行以 `callOp{fir::CallOp::create` 为核心的调用或声明。
- **L2084 EN**: Returns from the current function with `callOp.getResult(0)`.
  **L2084 CN**: 以 `callOp.getResult(0)` 从当前函数返回。
- **L2085 EN**: Closes the current lexical scope or compound statement.
  **L2085 CN**: 结束当前词法作用域或复合语句块。
- **L2086 EN**: Marks this control path as unreachable to LLVM.
  **L2086 CN**: 将该控制路径标记为 LLVM 认为不可达。
- **L2087 EN**: Closes the current lexical scope or compound statement.
  **L2087 CN**: 结束当前词法作用域或复合语句块。
- **L2088 EN**: Blank line separating nearby declarations or logic blocks.
  **L2088 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 2089-2112

````cpp
// VEC_PERM, VEC_PERMI
template <VecOp vop>
fir::ExtendedValue
PPCIntrinsicLibrary::genVecPerm(mlir::Type resultType,
                                llvm::ArrayRef<fir::ExtendedValue> args) {
  assert(args.size() == 3);
  auto context{builder.getContext()};
  auto argBases{getBasesForArgs(args)};
  auto argTypes{getTypesForArgs(argBases)};
  auto vecTyInfo{getVecTypeFromFir(argBases[0])};
  auto mlirTy{vecTyInfo.toMlirVectorType(context)};

  auto vi32Ty{mlir::VectorType::get(4, mlir::IntegerType::get(context, 32))};
  auto vf64Ty{mlir::VectorType::get(2, mlir::Float64Type::get(context))};

  auto mArg0{builder.createConvert(loc, mlirTy, argBases[0])};
  auto mArg1{builder.createConvert(loc, mlirTy, argBases[1])};

  switch (vop) {
  case VecOp::Perm: {
    VecTypeInfo maskVecTyInfo{getVecTypeFromFir(argBases[2])};
    auto mlirMaskTy{maskVecTyInfo.toMlirVectorType(context)};
    auto mMask{builder.createConvert(loc, mlirMaskTy, argBases[2])};

````
- **L2089 EN**: Comment explains nearby logic, intent, or metadata: `VEC_PERM, VEC_PERMI`.
  **L2089 CN**: 注释说明附近代码的逻辑、意图或元数据：`VEC_PERM, VEC_PERMI`。
- **L2090 EN**: Introduces template parameters or specialization context: `template <VecOp vop>`.
  **L2090 CN**: 为后续声明引入模板参数或特化上下文：`template <VecOp vop>`。
- **L2091 EN**: Continues the surrounding expression or declaration: `fir::ExtendedValue`.
  **L2091 CN**: 继续构造周围的表达式或声明：`fir::ExtendedValue`。
- **L2092 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PPCIntrinsicLibrary::genVecPerm(mlir::Type resultType,`.
  **L2092 CN**: 继续一个多行参数列表、初始化器或聚合项：`PPCIntrinsicLibrary::genVecPerm(mlir::Type resultType,`。
- **L2093 EN**: Continues the surrounding expression or declaration: `llvm::ArrayRef<fir::ExtendedValue> args) {`.
  **L2093 CN**: 继续构造周围的表达式或声明：`llvm::ArrayRef<fir::ExtendedValue> args) {`。
- **L2094 EN**: Checks an internal invariant in debug builds.
  **L2094 CN**: 在调试构建中检查内部不变式。
- **L2095 EN**: Executes a call or declaration centered on `context{builder.getContext`.
  **L2095 CN**: 执行以 `context{builder.getContext` 为核心的调用或声明。
- **L2096 EN**: Executes a call or declaration centered on `argBases{getBasesForArgs`.
  **L2096 CN**: 执行以 `argBases{getBasesForArgs` 为核心的调用或声明。
- **L2097 EN**: Executes a call or declaration centered on `argTypes{getTypesForArgs`.
  **L2097 CN**: 执行以 `argTypes{getTypesForArgs` 为核心的调用或声明。
- **L2098 EN**: Executes a call or declaration centered on `vecTyInfo{getVecTypeFromFir`.
  **L2098 CN**: 执行以 `vecTyInfo{getVecTypeFromFir` 为核心的调用或声明。
- **L2099 EN**: Executes a call or declaration centered on `mlirTy{vecTyInfo.toMlirVectorType`.
  **L2099 CN**: 执行以 `mlirTy{vecTyInfo.toMlirVectorType` 为核心的调用或声明。
- **L2100 EN**: Blank line separating nearby declarations or logic blocks.
  **L2100 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2101 EN**: Executes a call or declaration centered on `vi32Ty{mlir::VectorType::get`.
  **L2101 CN**: 执行以 `vi32Ty{mlir::VectorType::get` 为核心的调用或声明。
- **L2102 EN**: Executes a call or declaration centered on `vf64Ty{mlir::VectorType::get`.
  **L2102 CN**: 执行以 `vf64Ty{mlir::VectorType::get` 为核心的调用或声明。
- **L2103 EN**: Blank line separating nearby declarations or logic blocks.
  **L2103 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2104 EN**: Executes a call or declaration centered on `mArg0{builder.createConvert`.
  **L2104 CN**: 执行以 `mArg0{builder.createConvert` 为核心的调用或声明。
- **L2105 EN**: Executes a call or declaration centered on `mArg1{builder.createConvert`.
  **L2105 CN**: 执行以 `mArg1{builder.createConvert` 为核心的调用或声明。
- **L2106 EN**: Blank line separating nearby declarations or logic blocks.
  **L2106 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2107 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L2107 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L2108 EN**: Introduces a switch dispatch label: `case VecOp::Perm: {`.
  **L2108 CN**: 引入一个 switch 分发标签：`case VecOp::Perm: {`。
- **L2109 EN**: Executes a call or declaration centered on `maskVecTyInfo{getVecTypeFromFir`.
  **L2109 CN**: 执行以 `maskVecTyInfo{getVecTypeFromFir` 为核心的调用或声明。
- **L2110 EN**: Executes a call or declaration centered on `mlirMaskTy{maskVecTyInfo.toMlirVectorType`.
  **L2110 CN**: 执行以 `mlirMaskTy{maskVecTyInfo.toMlirVectorType` 为核心的调用或声明。
- **L2111 EN**: Executes a call or declaration centered on `mMask{builder.createConvert`.
  **L2111 CN**: 执行以 `mMask{builder.createConvert` 为核心的调用或声明。
- **L2112 EN**: Blank line separating nearby declarations or logic blocks.
  **L2112 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 2113-2136

````cpp
    if (mlirTy != vi32Ty) {
      mArg0 = mlir::LLVM::BitcastOp::create(builder, loc, vi32Ty, mArg0)
                  .getResult();
      mArg1 = mlir::LLVM::BitcastOp::create(builder, loc, vi32Ty, mArg1)
                  .getResult();
    }

    auto funcOp{builder.createFunction(
        loc, "llvm.ppc.altivec.vperm",
        genFuncType<Ty::IntegerVector<4>, Ty::IntegerVector<4>,
                    Ty::IntegerVector<4>, Ty::IntegerVector<1>>(context,
                                                                builder))};

    llvm::SmallVector<mlir::Value> newArgs;
    if (isNativeVecElemOrderOnLE()) {
      auto i8Ty{mlir::IntegerType::get(context, 8)};
      auto v8Ty{mlir::VectorType::get(16, i8Ty)};
      auto negOne{builder.createMinusOneInteger(loc, i8Ty)};
      auto vNegOne{
          mlir::vector::BroadcastOp::create(builder, loc, v8Ty, negOne)};

      mMask = mlir::arith::XOrIOp::create(builder, loc, mMask, vNegOne);
      newArgs = {mArg1, mArg0, mMask};
    } else {
````
- **L2113 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2113 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2114 EN**: Continues logic associated with callable symbol `create`.
  **L2114 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L2115 EN**: Executes a call or declaration centered on `.getResult`.
  **L2115 CN**: 执行以 `.getResult` 为核心的调用或声明。
- **L2116 EN**: Continues logic associated with callable symbol `create`.
  **L2116 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L2117 EN**: Executes a call or declaration centered on `.getResult`.
  **L2117 CN**: 执行以 `.getResult` 为核心的调用或声明。
- **L2118 EN**: Closes the current lexical scope or compound statement.
  **L2118 CN**: 结束当前词法作用域或复合语句块。
- **L2119 EN**: Blank line separating nearby declarations or logic blocks.
  **L2119 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2120 EN**: Continues logic associated with callable symbol `createFunction`.
  **L2120 CN**: 继续与可调用符号 `createFunction` 相关的逻辑。
- **L2121 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `loc, "llvm.ppc.altivec.vperm",`.
  **L2121 CN**: 继续一个多行参数列表、初始化器或聚合项：`loc, "llvm.ppc.altivec.vperm",`。
- **L2122 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genFuncType<Ty::IntegerVector<4>, Ty::IntegerVector<4>,`.
  **L2122 CN**: 继续一个多行参数列表、初始化器或聚合项：`genFuncType<Ty::IntegerVector<4>, Ty::IntegerVector<4>,`。
- **L2123 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Ty::IntegerVector<4>, Ty::IntegerVector<1>>(context,`.
  **L2123 CN**: 继续一个多行参数列表、初始化器或聚合项：`Ty::IntegerVector<4>, Ty::IntegerVector<1>>(context,`。
- **L2124 EN**: Executes a standalone statement or declaration: `builder))};`.
  **L2124 CN**: 执行一条独立语句或声明：`builder))};`。
- **L2125 EN**: Blank line separating nearby declarations or logic blocks.
  **L2125 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2126 EN**: Executes a standalone statement or declaration: `llvm::SmallVector<mlir::Value> newArgs;`.
  **L2126 CN**: 执行一条独立语句或声明：`llvm::SmallVector<mlir::Value> newArgs;`。
- **L2127 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2127 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2128 EN**: Executes a call or declaration centered on `i8Ty{mlir::IntegerType::get`.
  **L2128 CN**: 执行以 `i8Ty{mlir::IntegerType::get` 为核心的调用或声明。
- **L2129 EN**: Executes a call or declaration centered on `v8Ty{mlir::VectorType::get`.
  **L2129 CN**: 执行以 `v8Ty{mlir::VectorType::get` 为核心的调用或声明。
- **L2130 EN**: Executes a call or declaration centered on `negOne{builder.createMinusOneInteger`.
  **L2130 CN**: 执行以 `negOne{builder.createMinusOneInteger` 为核心的调用或声明。
- **L2131 EN**: Continues the surrounding expression or declaration: `auto vNegOne{`.
  **L2131 CN**: 继续构造周围的表达式或声明：`auto vNegOne{`。
- **L2132 EN**: Executes a call or declaration centered on `mlir::vector::BroadcastOp::create`.
  **L2132 CN**: 执行以 `mlir::vector::BroadcastOp::create` 为核心的调用或声明。
- **L2133 EN**: Blank line separating nearby declarations or logic blocks.
  **L2133 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2134 EN**: Executes a call or declaration centered on `mlir::arith::XOrIOp::create`.
  **L2134 CN**: 执行以 `mlir::arith::XOrIOp::create` 为核心的调用或声明。
- **L2135 EN**: Executes a standalone statement or declaration: `newArgs = {mArg1, mArg0, mMask};`.
  **L2135 CN**: 执行一条独立语句或声明：`newArgs = {mArg1, mArg0, mMask};`。
- **L2136 EN**: Transitions from the previous branch into the alternative path.
  **L2136 CN**: 从前一个分支过渡到备选路径。

### Lines 2137-2160

````cpp
      newArgs = {mArg0, mArg1, mMask};
    }

    auto res{fir::CallOp::create(builder, loc, funcOp, newArgs).getResult(0)};

    if (res.getType() != argTypes[0]) {
      // fir.call llvm.ppc.altivec.vperm returns !fir.vector<i4:32>
      // convert the result back to the original type
      res = builder.createConvert(loc, vi32Ty, res);
      if (mlirTy != vi32Ty)
        res = mlir::LLVM::BitcastOp::create(builder, loc, mlirTy, res)
                  .getResult();
    }
    return builder.createConvert(loc, resultType, res);
  }
  case VecOp::Permi: {
    // arg3 is a constant
    auto constIntOp{mlir::dyn_cast_or_null<mlir::IntegerAttr>(
        mlir::dyn_cast<mlir::arith::ConstantOp>(argBases[2].getDefiningOp())
            .getValue())};
    assert(constIntOp && "expected integer constant argument");
    auto constInt{constIntOp.getInt()};
    // arg1, arg2, and result type share same VecTypeInfo
    if (vecTyInfo.isFloat()) {
````
- **L2137 EN**: Executes a standalone statement or declaration: `newArgs = {mArg0, mArg1, mMask};`.
  **L2137 CN**: 执行一条独立语句或声明：`newArgs = {mArg0, mArg1, mMask};`。
- **L2138 EN**: Closes the current lexical scope or compound statement.
  **L2138 CN**: 结束当前词法作用域或复合语句块。
- **L2139 EN**: Blank line separating nearby declarations or logic blocks.
  **L2139 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2140 EN**: Executes a call or declaration centered on `res{fir::CallOp::create`.
  **L2140 CN**: 执行以 `res{fir::CallOp::create` 为核心的调用或声明。
- **L2141 EN**: Blank line separating nearby declarations or logic blocks.
  **L2141 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2142 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2142 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2143 EN**: Comment explains nearby logic, intent, or metadata: `fir.call llvm.ppc.altivec.vperm returns !fir.vector<i4:32>`.
  **L2143 CN**: 注释说明附近代码的逻辑、意图或元数据：`fir.call llvm.ppc.altivec.vperm returns !fir.vector<i4:32>`。
- **L2144 EN**: Comment explains nearby logic, intent, or metadata: `convert the result back to the original type`.
  **L2144 CN**: 注释说明附近代码的逻辑、意图或元数据：`convert the result back to the original type`。
- **L2145 EN**: Executes a call or declaration centered on `builder.createConvert`.
  **L2145 CN**: 执行以 `builder.createConvert` 为核心的调用或声明。
- **L2146 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2146 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2147 EN**: Continues logic associated with callable symbol `create`.
  **L2147 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L2148 EN**: Executes a call or declaration centered on `.getResult`.
  **L2148 CN**: 执行以 `.getResult` 为核心的调用或声明。
- **L2149 EN**: Closes the current lexical scope or compound statement.
  **L2149 CN**: 结束当前词法作用域或复合语句块。
- **L2150 EN**: Returns from the current function with `builder.createConvert(loc, resultType, res)`.
  **L2150 CN**: 以 `builder.createConvert(loc, resultType, res)` 从当前函数返回。
- **L2151 EN**: Closes the current lexical scope or compound statement.
  **L2151 CN**: 结束当前词法作用域或复合语句块。
- **L2152 EN**: Introduces a switch dispatch label: `case VecOp::Permi: {`.
  **L2152 CN**: 引入一个 switch 分发标签：`case VecOp::Permi: {`。
- **L2153 EN**: Comment explains nearby logic, intent, or metadata: `arg3 is a constant`.
  **L2153 CN**: 注释说明附近代码的逻辑、意图或元数据：`arg3 is a constant`。
- **L2154 EN**: Continues logic associated with callable symbol `IntegerAttr>`.
  **L2154 CN**: 继续与可调用符号 `IntegerAttr>` 相关的逻辑。
- **L2155 EN**: Continues logic associated with callable symbol `ConstantOp>`.
  **L2155 CN**: 继续与可调用符号 `ConstantOp>` 相关的逻辑。
- **L2156 EN**: Executes a call or declaration centered on `.getValue`.
  **L2156 CN**: 执行以 `.getValue` 为核心的调用或声明。
- **L2157 EN**: Checks an internal invariant in debug builds.
  **L2157 CN**: 在调试构建中检查内部不变式。
- **L2158 EN**: Executes a call or declaration centered on `constInt{constIntOp.getInt`.
  **L2158 CN**: 执行以 `constInt{constIntOp.getInt` 为核心的调用或声明。
- **L2159 EN**: Comment explains nearby logic, intent, or metadata: `arg1, arg2, and result type share same VecTypeInfo`.
  **L2159 CN**: 注释说明附近代码的逻辑、意图或元数据：`arg1, arg2, and result type share same VecTypeInfo`。
- **L2160 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2160 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 2161-2184

````cpp
      mArg0 = mlir::LLVM::BitcastOp::create(builder, loc, vf64Ty, mArg0)
                  .getResult();
      mArg1 = mlir::LLVM::BitcastOp::create(builder, loc, vf64Ty, mArg1)
                  .getResult();
    }

    llvm::SmallVector<int64_t, 2> nMask; // native vector element order mask
    llvm::SmallVector<int64_t, 2> rMask; // non-native vector element order mask
    enum { V1 = 0, V2 = 2 };
    switch (constInt) {
    case 0:
      nMask = {V1 + 0, V2 + 0};
      rMask = {V2 + 1, V1 + 1};
      break;
    case 1:
      nMask = {V1 + 0, V2 + 1};
      rMask = {V2 + 0, V1 + 1};
      break;
    case 2:
      nMask = {V1 + 1, V2 + 0};
      rMask = {V2 + 1, V1 + 0};
      break;
    case 3:
      nMask = {V1 + 1, V2 + 1};
````
- **L2161 EN**: Continues logic associated with callable symbol `create`.
  **L2161 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L2162 EN**: Executes a call or declaration centered on `.getResult`.
  **L2162 CN**: 执行以 `.getResult` 为核心的调用或声明。
- **L2163 EN**: Continues logic associated with callable symbol `create`.
  **L2163 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L2164 EN**: Executes a call or declaration centered on `.getResult`.
  **L2164 CN**: 执行以 `.getResult` 为核心的调用或声明。
- **L2165 EN**: Closes the current lexical scope or compound statement.
  **L2165 CN**: 结束当前词法作用域或复合语句块。
- **L2166 EN**: Blank line separating nearby declarations or logic blocks.
  **L2166 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2167 EN**: Continues the surrounding expression or declaration: `llvm::SmallVector<int64_t, 2> nMask; // native vector element order mask`.
  **L2167 CN**: 继续构造周围的表达式或声明：`llvm::SmallVector<int64_t, 2> nMask; // native vector element order mask`。
- **L2168 EN**: Continues the surrounding expression or declaration: `llvm::SmallVector<int64_t, 2> rMask; // non-native vector element order mask`.
  **L2168 CN**: 继续构造周围的表达式或声明：`llvm::SmallVector<int64_t, 2> rMask; // non-native vector element order mask`。
- **L2169 EN**: Declares enum ``.
  **L2169 CN**: 声明 enum ``。
- **L2170 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L2170 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L2171 EN**: Introduces a switch dispatch label: `case 0:`.
  **L2171 CN**: 引入一个 switch 分发标签：`case 0:`。
- **L2172 EN**: Executes a standalone statement or declaration: `nMask = {V1 + 0, V2 + 0};`.
  **L2172 CN**: 执行一条独立语句或声明：`nMask = {V1 + 0, V2 + 0};`。
- **L2173 EN**: Executes a standalone statement or declaration: `rMask = {V2 + 1, V1 + 1};`.
  **L2173 CN**: 执行一条独立语句或声明：`rMask = {V2 + 1, V1 + 1};`。
- **L2174 EN**: Exits the nearest loop or switch statement.
  **L2174 CN**: 退出最近的循环或 switch 语句。
- **L2175 EN**: Introduces a switch dispatch label: `case 1:`.
  **L2175 CN**: 引入一个 switch 分发标签：`case 1:`。
- **L2176 EN**: Executes a standalone statement or declaration: `nMask = {V1 + 0, V2 + 1};`.
  **L2176 CN**: 执行一条独立语句或声明：`nMask = {V1 + 0, V2 + 1};`。
- **L2177 EN**: Executes a standalone statement or declaration: `rMask = {V2 + 0, V1 + 1};`.
  **L2177 CN**: 执行一条独立语句或声明：`rMask = {V2 + 0, V1 + 1};`。
- **L2178 EN**: Exits the nearest loop or switch statement.
  **L2178 CN**: 退出最近的循环或 switch 语句。
- **L2179 EN**: Introduces a switch dispatch label: `case 2:`.
  **L2179 CN**: 引入一个 switch 分发标签：`case 2:`。
- **L2180 EN**: Executes a standalone statement or declaration: `nMask = {V1 + 1, V2 + 0};`.
  **L2180 CN**: 执行一条独立语句或声明：`nMask = {V1 + 1, V2 + 0};`。
- **L2181 EN**: Executes a standalone statement or declaration: `rMask = {V2 + 1, V1 + 0};`.
  **L2181 CN**: 执行一条独立语句或声明：`rMask = {V2 + 1, V1 + 0};`。
- **L2182 EN**: Exits the nearest loop or switch statement.
  **L2182 CN**: 退出最近的循环或 switch 语句。
- **L2183 EN**: Introduces a switch dispatch label: `case 3:`.
  **L2183 CN**: 引入一个 switch 分发标签：`case 3:`。
- **L2184 EN**: Executes a standalone statement or declaration: `nMask = {V1 + 1, V2 + 1};`.
  **L2184 CN**: 执行一条独立语句或声明：`nMask = {V1 + 1, V2 + 1};`。

### Lines 2185-2208

````cpp
      rMask = {V2 + 0, V1 + 0};
      break;
    default:
      llvm_unreachable("unexpected arg3 value for vec_permi");
    }

    llvm::SmallVector<int64_t, 2> mask =
        (isBEVecElemOrderOnLE()) ? rMask : nMask;
    auto res{mlir::vector::ShuffleOp::create(builder, loc, mArg0, mArg1, mask)};
    if (res.getType() != mlirTy) {
      auto cast{mlir::LLVM::BitcastOp::create(builder, loc, mlirTy, res)};
      return builder.createConvert(loc, resultType, cast);
    }
    return builder.createConvert(loc, resultType, res);
  }
  default:
    llvm_unreachable("invalid vector operation for generator");
  }
}

// VEC_SEL
fir::ExtendedValue
PPCIntrinsicLibrary::genVecSel(mlir::Type resultType,
                               llvm::ArrayRef<fir::ExtendedValue> args) {
````
- **L2185 EN**: Executes a standalone statement or declaration: `rMask = {V2 + 0, V1 + 0};`.
  **L2185 CN**: 执行一条独立语句或声明：`rMask = {V2 + 0, V1 + 0};`。
- **L2186 EN**: Exits the nearest loop or switch statement.
  **L2186 CN**: 退出最近的循环或 switch 语句。
- **L2187 EN**: Introduces a switch dispatch label: `default:`.
  **L2187 CN**: 引入一个 switch 分发标签：`default:`。
- **L2188 EN**: Marks this control path as unreachable to LLVM.
  **L2188 CN**: 将该控制路径标记为 LLVM 认为不可达。
- **L2189 EN**: Closes the current lexical scope or compound statement.
  **L2189 CN**: 结束当前词法作用域或复合语句块。
- **L2190 EN**: Blank line separating nearby declarations or logic blocks.
  **L2190 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2191 EN**: Continues the surrounding expression or declaration: `llvm::SmallVector<int64_t, 2> mask =`.
  **L2191 CN**: 继续构造周围的表达式或声明：`llvm::SmallVector<int64_t, 2> mask =`。
- **L2192 EN**: Executes a call or declaration centered on `statement`.
  **L2192 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2193 EN**: Executes a call or declaration centered on `res{mlir::vector::ShuffleOp::create`.
  **L2193 CN**: 执行以 `res{mlir::vector::ShuffleOp::create` 为核心的调用或声明。
- **L2194 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2194 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2195 EN**: Executes a call or declaration centered on `cast{mlir::LLVM::BitcastOp::create`.
  **L2195 CN**: 执行以 `cast{mlir::LLVM::BitcastOp::create` 为核心的调用或声明。
- **L2196 EN**: Returns from the current function with `builder.createConvert(loc, resultType, cast)`.
  **L2196 CN**: 以 `builder.createConvert(loc, resultType, cast)` 从当前函数返回。
- **L2197 EN**: Closes the current lexical scope or compound statement.
  **L2197 CN**: 结束当前词法作用域或复合语句块。
- **L2198 EN**: Returns from the current function with `builder.createConvert(loc, resultType, res)`.
  **L2198 CN**: 以 `builder.createConvert(loc, resultType, res)` 从当前函数返回。
- **L2199 EN**: Closes the current lexical scope or compound statement.
  **L2199 CN**: 结束当前词法作用域或复合语句块。
- **L2200 EN**: Introduces a switch dispatch label: `default:`.
  **L2200 CN**: 引入一个 switch 分发标签：`default:`。
- **L2201 EN**: Marks this control path as unreachable to LLVM.
  **L2201 CN**: 将该控制路径标记为 LLVM 认为不可达。
- **L2202 EN**: Closes the current lexical scope or compound statement.
  **L2202 CN**: 结束当前词法作用域或复合语句块。
- **L2203 EN**: Closes the current lexical scope or compound statement.
  **L2203 CN**: 结束当前词法作用域或复合语句块。
- **L2204 EN**: Blank line separating nearby declarations or logic blocks.
  **L2204 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2205 EN**: Comment explains nearby logic, intent, or metadata: `VEC_SEL`.
  **L2205 CN**: 注释说明附近代码的逻辑、意图或元数据：`VEC_SEL`。
- **L2206 EN**: Continues the surrounding expression or declaration: `fir::ExtendedValue`.
  **L2206 CN**: 继续构造周围的表达式或声明：`fir::ExtendedValue`。
- **L2207 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PPCIntrinsicLibrary::genVecSel(mlir::Type resultType,`.
  **L2207 CN**: 继续一个多行参数列表、初始化器或聚合项：`PPCIntrinsicLibrary::genVecSel(mlir::Type resultType,`。
- **L2208 EN**: Continues the surrounding expression or declaration: `llvm::ArrayRef<fir::ExtendedValue> args) {`.
  **L2208 CN**: 继续构造周围的表达式或声明：`llvm::ArrayRef<fir::ExtendedValue> args) {`。

### Lines 2209-2232

````cpp
  assert(args.size() == 3);
  auto argBases{getBasesForArgs(args)};
  llvm::SmallVector<VecTypeInfo, 4> vecTyInfos;
  for (size_t i = 0; i < argBases.size(); i++) {
    vecTyInfos.push_back(getVecTypeFromFir(argBases[i]));
  }
  auto vargs{convertVecArgs(builder, loc, vecTyInfos, argBases)};

  auto i8Ty{mlir::IntegerType::get(builder.getContext(), 8)};
  auto negOne{builder.createMinusOneInteger(loc, i8Ty)};

  // construct a constant <16 x i8> vector with value -1 for bitcast
  auto bcVecTy{mlir::VectorType::get(16, i8Ty)};
  auto vNegOne{
      mlir::vector::BroadcastOp::create(builder, loc, bcVecTy, negOne)};

  // bitcast arguments to bcVecTy
  auto arg1{mlir::vector::BitCastOp::create(builder, loc, bcVecTy, vargs[0])};
  auto arg2{mlir::vector::BitCastOp::create(builder, loc, bcVecTy, vargs[1])};
  auto arg3{mlir::vector::BitCastOp::create(builder, loc, bcVecTy, vargs[2])};

  // vec_sel(arg1, arg2, arg3) =
  //   (arg2 and arg3) or (arg1 and (arg3 xor vector(-1,...)))
  auto comp{mlir::arith::XOrIOp::create(builder, loc, arg3, vNegOne)};
````
- **L2209 EN**: Checks an internal invariant in debug builds.
  **L2209 CN**: 在调试构建中检查内部不变式。
- **L2210 EN**: Executes a call or declaration centered on `argBases{getBasesForArgs`.
  **L2210 CN**: 执行以 `argBases{getBasesForArgs` 为核心的调用或声明。
- **L2211 EN**: Executes a standalone statement or declaration: `llvm::SmallVector<VecTypeInfo, 4> vecTyInfos;`.
  **L2211 CN**: 执行一条独立语句或声明：`llvm::SmallVector<VecTypeInfo, 4> vecTyInfos;`。
- **L2212 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L2212 CN**: 开始 `for` 控制流语句并计算其条件。
- **L2213 EN**: Executes a call or declaration centered on `vecTyInfos.push_back`.
  **L2213 CN**: 执行以 `vecTyInfos.push_back` 为核心的调用或声明。
- **L2214 EN**: Closes the current lexical scope or compound statement.
  **L2214 CN**: 结束当前词法作用域或复合语句块。
- **L2215 EN**: Executes a call or declaration centered on `vargs{convertVecArgs`.
  **L2215 CN**: 执行以 `vargs{convertVecArgs` 为核心的调用或声明。
- **L2216 EN**: Blank line separating nearby declarations or logic blocks.
  **L2216 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2217 EN**: Executes a call or declaration centered on `i8Ty{mlir::IntegerType::get`.
  **L2217 CN**: 执行以 `i8Ty{mlir::IntegerType::get` 为核心的调用或声明。
- **L2218 EN**: Executes a call or declaration centered on `negOne{builder.createMinusOneInteger`.
  **L2218 CN**: 执行以 `negOne{builder.createMinusOneInteger` 为核心的调用或声明。
- **L2219 EN**: Blank line separating nearby declarations or logic blocks.
  **L2219 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2220 EN**: Comment explains nearby logic, intent, or metadata: `construct a constant <16 x i8> vector with value -1 for bitcast`.
  **L2220 CN**: 注释说明附近代码的逻辑、意图或元数据：`construct a constant <16 x i8> vector with value -1 for bitcast`。
- **L2221 EN**: Executes a call or declaration centered on `bcVecTy{mlir::VectorType::get`.
  **L2221 CN**: 执行以 `bcVecTy{mlir::VectorType::get` 为核心的调用或声明。
- **L2222 EN**: Continues the surrounding expression or declaration: `auto vNegOne{`.
  **L2222 CN**: 继续构造周围的表达式或声明：`auto vNegOne{`。
- **L2223 EN**: Executes a call or declaration centered on `mlir::vector::BroadcastOp::create`.
  **L2223 CN**: 执行以 `mlir::vector::BroadcastOp::create` 为核心的调用或声明。
- **L2224 EN**: Blank line separating nearby declarations or logic blocks.
  **L2224 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2225 EN**: Comment explains nearby logic, intent, or metadata: `bitcast arguments to bcVecTy`.
  **L2225 CN**: 注释说明附近代码的逻辑、意图或元数据：`bitcast arguments to bcVecTy`。
- **L2226 EN**: Executes a call or declaration centered on `arg1{mlir::vector::BitCastOp::create`.
  **L2226 CN**: 执行以 `arg1{mlir::vector::BitCastOp::create` 为核心的调用或声明。
- **L2227 EN**: Executes a call or declaration centered on `arg2{mlir::vector::BitCastOp::create`.
  **L2227 CN**: 执行以 `arg2{mlir::vector::BitCastOp::create` 为核心的调用或声明。
- **L2228 EN**: Executes a call or declaration centered on `arg3{mlir::vector::BitCastOp::create`.
  **L2228 CN**: 执行以 `arg3{mlir::vector::BitCastOp::create` 为核心的调用或声明。
- **L2229 EN**: Blank line separating nearby declarations or logic blocks.
  **L2229 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2230 EN**: Comment explains nearby logic, intent, or metadata: `vec_sel(arg1, arg2, arg3) =`.
  **L2230 CN**: 注释说明附近代码的逻辑、意图或元数据：`vec_sel(arg1, arg2, arg3) =`。
- **L2231 EN**: Comment explains nearby logic, intent, or metadata: `(arg2 and arg3) or (arg1 and (arg3 xor vector(-1,...)))`.
  **L2231 CN**: 注释说明附近代码的逻辑、意图或元数据：`(arg2 and arg3) or (arg1 and (arg3 xor vector(-1,...)))`。
- **L2232 EN**: Executes a call or declaration centered on `comp{mlir::arith::XOrIOp::create`.
  **L2232 CN**: 执行以 `comp{mlir::arith::XOrIOp::create` 为核心的调用或声明。

### Lines 2233-2256

````cpp
  auto a1AndComp{mlir::arith::AndIOp::create(builder, loc, arg1, comp)};
  auto a1OrA2{mlir::arith::AndIOp::create(builder, loc, arg2, arg3)};
  auto res{mlir::arith::OrIOp::create(builder, loc, a1AndComp, a1OrA2)};

  auto bcRes{
      mlir::vector::BitCastOp::create(builder, loc, vargs[0].getType(), res)};

  return builder.createConvert(loc, vecTyInfos[0].toFirVectorType(), bcRes);
}

// VEC_SL, VEC_SLD, VEC_SLDW, VEC_SLL, VEC_SLO, VEC_SR, VEC_SRL, VEC_SRO
template <VecOp vop>
fir::ExtendedValue
PPCIntrinsicLibrary::genVecShift(mlir::Type resultType,
                                 llvm::ArrayRef<fir::ExtendedValue> args) {
  auto context{builder.getContext()};
  auto argBases{getBasesForArgs(args)};
  auto argTypes{getTypesForArgs(argBases)};

  llvm::SmallVector<VecTypeInfo, 2> vecTyInfoArgs;
  vecTyInfoArgs.push_back(getVecTypeFromFir(argBases[0]));
  vecTyInfoArgs.push_back(getVecTypeFromFir(argBases[1]));

  // Convert the first two arguments to MLIR vectors
````
- **L2233 EN**: Executes a call or declaration centered on `a1AndComp{mlir::arith::AndIOp::create`.
  **L2233 CN**: 执行以 `a1AndComp{mlir::arith::AndIOp::create` 为核心的调用或声明。
- **L2234 EN**: Executes a call or declaration centered on `a1OrA2{mlir::arith::AndIOp::create`.
  **L2234 CN**: 执行以 `a1OrA2{mlir::arith::AndIOp::create` 为核心的调用或声明。
- **L2235 EN**: Executes a call or declaration centered on `res{mlir::arith::OrIOp::create`.
  **L2235 CN**: 执行以 `res{mlir::arith::OrIOp::create` 为核心的调用或声明。
- **L2236 EN**: Blank line separating nearby declarations or logic blocks.
  **L2236 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2237 EN**: Continues the surrounding expression or declaration: `auto bcRes{`.
  **L2237 CN**: 继续构造周围的表达式或声明：`auto bcRes{`。
- **L2238 EN**: Executes a call or declaration centered on `mlir::vector::BitCastOp::create`.
  **L2238 CN**: 执行以 `mlir::vector::BitCastOp::create` 为核心的调用或声明。
- **L2239 EN**: Blank line separating nearby declarations or logic blocks.
  **L2239 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2240 EN**: Returns from the current function with `builder.createConvert(loc, vecTyInfos[0].toFirVectorType(), bcRes)`.
  **L2240 CN**: 以 `builder.createConvert(loc, vecTyInfos[0].toFirVectorType(), bcRes)` 从当前函数返回。
- **L2241 EN**: Closes the current lexical scope or compound statement.
  **L2241 CN**: 结束当前词法作用域或复合语句块。
- **L2242 EN**: Blank line separating nearby declarations or logic blocks.
  **L2242 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2243 EN**: Comment explains nearby logic, intent, or metadata: `VEC_SL, VEC_SLD, VEC_SLDW, VEC_SLL, VEC_SLO, VEC_SR, VEC_SRL, VEC_SRO`.
  **L2243 CN**: 注释说明附近代码的逻辑、意图或元数据：`VEC_SL, VEC_SLD, VEC_SLDW, VEC_SLL, VEC_SLO, VEC_SR, VEC_SRL, VEC_SRO`。
- **L2244 EN**: Introduces template parameters or specialization context: `template <VecOp vop>`.
  **L2244 CN**: 为后续声明引入模板参数或特化上下文：`template <VecOp vop>`。
- **L2245 EN**: Continues the surrounding expression or declaration: `fir::ExtendedValue`.
  **L2245 CN**: 继续构造周围的表达式或声明：`fir::ExtendedValue`。
- **L2246 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PPCIntrinsicLibrary::genVecShift(mlir::Type resultType,`.
  **L2246 CN**: 继续一个多行参数列表、初始化器或聚合项：`PPCIntrinsicLibrary::genVecShift(mlir::Type resultType,`。
- **L2247 EN**: Continues the surrounding expression or declaration: `llvm::ArrayRef<fir::ExtendedValue> args) {`.
  **L2247 CN**: 继续构造周围的表达式或声明：`llvm::ArrayRef<fir::ExtendedValue> args) {`。
- **L2248 EN**: Executes a call or declaration centered on `context{builder.getContext`.
  **L2248 CN**: 执行以 `context{builder.getContext` 为核心的调用或声明。
- **L2249 EN**: Executes a call or declaration centered on `argBases{getBasesForArgs`.
  **L2249 CN**: 执行以 `argBases{getBasesForArgs` 为核心的调用或声明。
- **L2250 EN**: Executes a call or declaration centered on `argTypes{getTypesForArgs`.
  **L2250 CN**: 执行以 `argTypes{getTypesForArgs` 为核心的调用或声明。
- **L2251 EN**: Blank line separating nearby declarations or logic blocks.
  **L2251 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2252 EN**: Executes a standalone statement or declaration: `llvm::SmallVector<VecTypeInfo, 2> vecTyInfoArgs;`.
  **L2252 CN**: 执行一条独立语句或声明：`llvm::SmallVector<VecTypeInfo, 2> vecTyInfoArgs;`。
- **L2253 EN**: Executes a call or declaration centered on `vecTyInfoArgs.push_back`.
  **L2253 CN**: 执行以 `vecTyInfoArgs.push_back` 为核心的调用或声明。
- **L2254 EN**: Executes a call or declaration centered on `vecTyInfoArgs.push_back`.
  **L2254 CN**: 执行以 `vecTyInfoArgs.push_back` 为核心的调用或声明。
- **L2255 EN**: Blank line separating nearby declarations or logic blocks.
  **L2255 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2256 EN**: Comment explains nearby logic, intent, or metadata: `Convert the first two arguments to MLIR vectors`.
  **L2256 CN**: 注释说明附近代码的逻辑、意图或元数据：`Convert the first two arguments to MLIR vectors`。

### Lines 2257-2280

````cpp
  llvm::SmallVector<mlir::Type, 2> mlirTyArgs;
  mlirTyArgs.push_back(vecTyInfoArgs[0].toMlirVectorType(context));
  mlirTyArgs.push_back(vecTyInfoArgs[1].toMlirVectorType(context));

  llvm::SmallVector<mlir::Value, 2> mlirVecArgs;
  mlirVecArgs.push_back(builder.createConvert(loc, mlirTyArgs[0], argBases[0]));
  mlirVecArgs.push_back(builder.createConvert(loc, mlirTyArgs[1], argBases[1]));

  mlir::Value shftRes{nullptr};

  if (vop == VecOp::Sl || vop == VecOp::Sr) {
    assert(args.size() == 2);
    // Construct the mask
    auto width{
        mlir::dyn_cast<mlir::IntegerType>(vecTyInfoArgs[1].eleTy).getWidth()};
    auto vecVal{builder.createIntegerConstant(
        loc, getConvertedElementType(context, vecTyInfoArgs[0].eleTy), width)};
    auto mask{
        mlir::vector::BroadcastOp::create(builder, loc, mlirTyArgs[1], vecVal)};
    auto shft{mlir::arith::RemUIOp::create(builder, loc, mlirVecArgs[1], mask)};

    mlir::Value res{nullptr};
    if (vop == VecOp::Sr)
      res = mlir::arith::ShRUIOp::create(builder, loc, mlirVecArgs[0], shft);
````
- **L2257 EN**: Executes a standalone statement or declaration: `llvm::SmallVector<mlir::Type, 2> mlirTyArgs;`.
  **L2257 CN**: 执行一条独立语句或声明：`llvm::SmallVector<mlir::Type, 2> mlirTyArgs;`。
- **L2258 EN**: Executes a call or declaration centered on `mlirTyArgs.push_back`.
  **L2258 CN**: 执行以 `mlirTyArgs.push_back` 为核心的调用或声明。
- **L2259 EN**: Executes a call or declaration centered on `mlirTyArgs.push_back`.
  **L2259 CN**: 执行以 `mlirTyArgs.push_back` 为核心的调用或声明。
- **L2260 EN**: Blank line separating nearby declarations or logic blocks.
  **L2260 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2261 EN**: Executes a standalone statement or declaration: `llvm::SmallVector<mlir::Value, 2> mlirVecArgs;`.
  **L2261 CN**: 执行一条独立语句或声明：`llvm::SmallVector<mlir::Value, 2> mlirVecArgs;`。
- **L2262 EN**: Executes a call or declaration centered on `mlirVecArgs.push_back`.
  **L2262 CN**: 执行以 `mlirVecArgs.push_back` 为核心的调用或声明。
- **L2263 EN**: Executes a call or declaration centered on `mlirVecArgs.push_back`.
  **L2263 CN**: 执行以 `mlirVecArgs.push_back` 为核心的调用或声明。
- **L2264 EN**: Blank line separating nearby declarations or logic blocks.
  **L2264 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2265 EN**: Executes a standalone statement or declaration: `mlir::Value shftRes{nullptr};`.
  **L2265 CN**: 执行一条独立语句或声明：`mlir::Value shftRes{nullptr};`。
- **L2266 EN**: Blank line separating nearby declarations or logic blocks.
  **L2266 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2267 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2267 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2268 EN**: Checks an internal invariant in debug builds.
  **L2268 CN**: 在调试构建中检查内部不变式。
- **L2269 EN**: Comment explains nearby logic, intent, or metadata: `Construct the mask`.
  **L2269 CN**: 注释说明附近代码的逻辑、意图或元数据：`Construct the mask`。
- **L2270 EN**: Continues the surrounding expression or declaration: `auto width{`.
  **L2270 CN**: 继续构造周围的表达式或声明：`auto width{`。
- **L2271 EN**: Executes a call or declaration centered on `mlir::dyn_cast<mlir::IntegerType>`.
  **L2271 CN**: 执行以 `mlir::dyn_cast<mlir::IntegerType>` 为核心的调用或声明。
- **L2272 EN**: Continues logic associated with callable symbol `createIntegerConstant`.
  **L2272 CN**: 继续与可调用符号 `createIntegerConstant` 相关的逻辑。
- **L2273 EN**: Executes a call or declaration centered on `getConvertedElementType`.
  **L2273 CN**: 执行以 `getConvertedElementType` 为核心的调用或声明。
- **L2274 EN**: Continues the surrounding expression or declaration: `auto mask{`.
  **L2274 CN**: 继续构造周围的表达式或声明：`auto mask{`。
- **L2275 EN**: Executes a call or declaration centered on `mlir::vector::BroadcastOp::create`.
  **L2275 CN**: 执行以 `mlir::vector::BroadcastOp::create` 为核心的调用或声明。
- **L2276 EN**: Executes a call or declaration centered on `shft{mlir::arith::RemUIOp::create`.
  **L2276 CN**: 执行以 `shft{mlir::arith::RemUIOp::create` 为核心的调用或声明。
- **L2277 EN**: Blank line separating nearby declarations or logic blocks.
  **L2277 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2278 EN**: Executes a standalone statement or declaration: `mlir::Value res{nullptr};`.
  **L2278 CN**: 执行一条独立语句或声明：`mlir::Value res{nullptr};`。
- **L2279 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2279 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2280 EN**: Executes a call or declaration centered on `mlir::arith::ShRUIOp::create`.
  **L2280 CN**: 执行以 `mlir::arith::ShRUIOp::create` 为核心的调用或声明。

### Lines 2281-2304

````cpp
    else if (vop == VecOp::Sl)
      res = mlir::arith::ShLIOp::create(builder, loc, mlirVecArgs[0], shft);

    shftRes = builder.createConvert(loc, argTypes[0], res);
  } else if (vop == VecOp::Sll || vop == VecOp::Slo || vop == VecOp::Srl ||
             vop == VecOp::Sro) {
    assert(args.size() == 2);

    // Bitcast to vector<4xi32>
    auto bcVecTy{mlir::VectorType::get(4, builder.getIntegerType(32))};
    if (mlirTyArgs[0] != bcVecTy)
      mlirVecArgs[0] = mlir::vector::BitCastOp::create(builder, loc, bcVecTy,
                                                       mlirVecArgs[0]);
    if (mlirTyArgs[1] != bcVecTy)
      mlirVecArgs[1] = mlir::vector::BitCastOp::create(builder, loc, bcVecTy,
                                                       mlirVecArgs[1]);

    llvm::StringRef funcName;
    switch (vop) {
    case VecOp::Srl:
      funcName = "llvm.ppc.altivec.vsr";
      break;
    case VecOp::Sro:
      funcName = "llvm.ppc.altivec.vsro";
````
- **L2281 EN**: Starts the alternative branch of the preceding conditional.
  **L2281 CN**: 开始前一个条件语句的备选分支。
- **L2282 EN**: Executes a call or declaration centered on `mlir::arith::ShLIOp::create`.
  **L2282 CN**: 执行以 `mlir::arith::ShLIOp::create` 为核心的调用或声明。
- **L2283 EN**: Blank line separating nearby declarations or logic blocks.
  **L2283 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2284 EN**: Executes a call or declaration centered on `builder.createConvert`.
  **L2284 CN**: 执行以 `builder.createConvert` 为核心的调用或声明。
- **L2285 EN**: Transitions from the previous branch into an `else if` condition.
  **L2285 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2286 EN**: Continues the surrounding expression or declaration: `vop == VecOp::Sro) {`.
  **L2286 CN**: 继续构造周围的表达式或声明：`vop == VecOp::Sro) {`。
- **L2287 EN**: Checks an internal invariant in debug builds.
  **L2287 CN**: 在调试构建中检查内部不变式。
- **L2288 EN**: Blank line separating nearby declarations or logic blocks.
  **L2288 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2289 EN**: Comment explains nearby logic, intent, or metadata: `Bitcast to vector<4xi32>`.
  **L2289 CN**: 注释说明附近代码的逻辑、意图或元数据：`Bitcast to vector<4xi32>`。
- **L2290 EN**: Executes a call or declaration centered on `bcVecTy{mlir::VectorType::get`.
  **L2290 CN**: 执行以 `bcVecTy{mlir::VectorType::get` 为核心的调用或声明。
- **L2291 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2291 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2292 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlirVecArgs[0] = mlir::vector::BitCastOp::create(builder, loc, bcVecTy,`.
  **L2292 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlirVecArgs[0] = mlir::vector::BitCastOp::create(builder, loc, bcVecTy,`。
- **L2293 EN**: Executes a standalone statement or declaration: `mlirVecArgs[0]);`.
  **L2293 CN**: 执行一条独立语句或声明：`mlirVecArgs[0]);`。
- **L2294 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2294 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2295 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlirVecArgs[1] = mlir::vector::BitCastOp::create(builder, loc, bcVecTy,`.
  **L2295 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlirVecArgs[1] = mlir::vector::BitCastOp::create(builder, loc, bcVecTy,`。
- **L2296 EN**: Executes a standalone statement or declaration: `mlirVecArgs[1]);`.
  **L2296 CN**: 执行一条独立语句或声明：`mlirVecArgs[1]);`。
- **L2297 EN**: Blank line separating nearby declarations or logic blocks.
  **L2297 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2298 EN**: Executes a standalone statement or declaration: `llvm::StringRef funcName;`.
  **L2298 CN**: 执行一条独立语句或声明：`llvm::StringRef funcName;`。
- **L2299 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L2299 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L2300 EN**: Introduces a switch dispatch label: `case VecOp::Srl:`.
  **L2300 CN**: 引入一个 switch 分发标签：`case VecOp::Srl:`。
- **L2301 EN**: Executes a standalone statement or declaration: `funcName = "llvm.ppc.altivec.vsr";`.
  **L2301 CN**: 执行一条独立语句或声明：`funcName = "llvm.ppc.altivec.vsr";`。
- **L2302 EN**: Exits the nearest loop or switch statement.
  **L2302 CN**: 退出最近的循环或 switch 语句。
- **L2303 EN**: Introduces a switch dispatch label: `case VecOp::Sro:`.
  **L2303 CN**: 引入一个 switch 分发标签：`case VecOp::Sro:`。
- **L2304 EN**: Executes a standalone statement or declaration: `funcName = "llvm.ppc.altivec.vsro";`.
  **L2304 CN**: 执行一条独立语句或声明：`funcName = "llvm.ppc.altivec.vsro";`。

### Lines 2305-2328

````cpp
      break;
    case VecOp::Sll:
      funcName = "llvm.ppc.altivec.vsl";
      break;
    case VecOp::Slo:
      funcName = "llvm.ppc.altivec.vslo";
      break;
    default:
      llvm_unreachable("unknown vector shift operation");
    }
    auto funcTy{genFuncType<Ty::IntegerVector<4>, Ty::IntegerVector<4>,
                            Ty::IntegerVector<4>>(context, builder)};
    mlir::func::FuncOp funcOp{builder.createFunction(loc, funcName, funcTy)};
    auto callOp{fir::CallOp::create(builder, loc, funcOp, mlirVecArgs)};

    // If the result vector type is different from the original type, need
    // to convert to mlir vector, bitcast and then convert back to fir vector.
    if (callOp.getResult(0).getType() != argTypes[0]) {
      auto res = builder.createConvert(loc, bcVecTy, callOp.getResult(0));
      res = mlir::vector::BitCastOp::create(builder, loc, mlirTyArgs[0], res);
      shftRes = builder.createConvert(loc, argTypes[0], res);
    } else {
      shftRes = callOp.getResult(0);
    }
````
- **L2305 EN**: Exits the nearest loop or switch statement.
  **L2305 CN**: 退出最近的循环或 switch 语句。
- **L2306 EN**: Introduces a switch dispatch label: `case VecOp::Sll:`.
  **L2306 CN**: 引入一个 switch 分发标签：`case VecOp::Sll:`。
- **L2307 EN**: Executes a standalone statement or declaration: `funcName = "llvm.ppc.altivec.vsl";`.
  **L2307 CN**: 执行一条独立语句或声明：`funcName = "llvm.ppc.altivec.vsl";`。
- **L2308 EN**: Exits the nearest loop or switch statement.
  **L2308 CN**: 退出最近的循环或 switch 语句。
- **L2309 EN**: Introduces a switch dispatch label: `case VecOp::Slo:`.
  **L2309 CN**: 引入一个 switch 分发标签：`case VecOp::Slo:`。
- **L2310 EN**: Executes a standalone statement or declaration: `funcName = "llvm.ppc.altivec.vslo";`.
  **L2310 CN**: 执行一条独立语句或声明：`funcName = "llvm.ppc.altivec.vslo";`。
- **L2311 EN**: Exits the nearest loop or switch statement.
  **L2311 CN**: 退出最近的循环或 switch 语句。
- **L2312 EN**: Introduces a switch dispatch label: `default:`.
  **L2312 CN**: 引入一个 switch 分发标签：`default:`。
- **L2313 EN**: Marks this control path as unreachable to LLVM.
  **L2313 CN**: 将该控制路径标记为 LLVM 认为不可达。
- **L2314 EN**: Closes the current lexical scope or compound statement.
  **L2314 CN**: 结束当前词法作用域或复合语句块。
- **L2315 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto funcTy{genFuncType<Ty::IntegerVector<4>, Ty::IntegerVector<4>,`.
  **L2315 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto funcTy{genFuncType<Ty::IntegerVector<4>, Ty::IntegerVector<4>,`。
- **L2316 EN**: Executes a call or declaration centered on `Ty::IntegerVector<4>>`.
  **L2316 CN**: 执行以 `Ty::IntegerVector<4>>` 为核心的调用或声明。
- **L2317 EN**: Executes a call or declaration centered on `funcOp{builder.createFunction`.
  **L2317 CN**: 执行以 `funcOp{builder.createFunction` 为核心的调用或声明。
- **L2318 EN**: Executes a call or declaration centered on `callOp{fir::CallOp::create`.
  **L2318 CN**: 执行以 `callOp{fir::CallOp::create` 为核心的调用或声明。
- **L2319 EN**: Blank line separating nearby declarations or logic blocks.
  **L2319 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2320 EN**: Comment explains nearby logic, intent, or metadata: `If the result vector type is different from the original type, need`.
  **L2320 CN**: 注释说明附近代码的逻辑、意图或元数据：`If the result vector type is different from the original type, need`。
- **L2321 EN**: Comment explains nearby logic, intent, or metadata: `to convert to mlir vector, bitcast and then convert back to fir vector.`.
  **L2321 CN**: 注释说明附近代码的逻辑、意图或元数据：`to convert to mlir vector, bitcast and then convert back to fir vector.`。
- **L2322 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2322 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2323 EN**: Initializes variable `res` from the right-hand expression.
  **L2323 CN**: 使用右侧表达式初始化变量 `res`。
- **L2324 EN**: Executes a call or declaration centered on `mlir::vector::BitCastOp::create`.
  **L2324 CN**: 执行以 `mlir::vector::BitCastOp::create` 为核心的调用或声明。
- **L2325 EN**: Executes a call or declaration centered on `builder.createConvert`.
  **L2325 CN**: 执行以 `builder.createConvert` 为核心的调用或声明。
- **L2326 EN**: Transitions from the previous branch into the alternative path.
  **L2326 CN**: 从前一个分支过渡到备选路径。
- **L2327 EN**: Executes a call or declaration centered on `callOp.getResult`.
  **L2327 CN**: 执行以 `callOp.getResult` 为核心的调用或声明。
- **L2328 EN**: Closes the current lexical scope or compound statement.
  **L2328 CN**: 结束当前词法作用域或复合语句块。

### Lines 2329-2352

````cpp
  } else if (vop == VecOp::Sld || vop == VecOp::Sldw) {
    assert(args.size() == 3);
    auto constIntOp = mlir::dyn_cast_or_null<mlir::IntegerAttr>(
        mlir::dyn_cast<mlir::arith::ConstantOp>(argBases[2].getDefiningOp())
            .getValue());
    assert(constIntOp && "expected integer constant argument");

    // Bitcast to vector<16xi8>
    auto vi8Ty{mlir::VectorType::get(16, builder.getIntegerType(8))};
    if (mlirTyArgs[0] != vi8Ty) {
      mlirVecArgs[0] =
          mlir::LLVM::BitcastOp::create(builder, loc, vi8Ty, mlirVecArgs[0])
              .getResult();
      mlirVecArgs[1] =
          mlir::LLVM::BitcastOp::create(builder, loc, vi8Ty, mlirVecArgs[1])
              .getResult();
    }

    // Construct the mask for shuffling
    auto shiftVal{constIntOp.getInt()};
    if (vop == VecOp::Sldw)
      shiftVal = shiftVal << 2;
    shiftVal &= 0xF;
    llvm::SmallVector<int64_t, 16> mask;
````
- **L2329 EN**: Transitions from the previous branch into an `else if` condition.
  **L2329 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2330 EN**: Checks an internal invariant in debug builds.
  **L2330 CN**: 在调试构建中检查内部不变式。
- **L2331 EN**: Continues logic associated with callable symbol `IntegerAttr>`.
  **L2331 CN**: 继续与可调用符号 `IntegerAttr>` 相关的逻辑。
- **L2332 EN**: Continues logic associated with callable symbol `ConstantOp>`.
  **L2332 CN**: 继续与可调用符号 `ConstantOp>` 相关的逻辑。
- **L2333 EN**: Executes a call or declaration centered on `.getValue`.
  **L2333 CN**: 执行以 `.getValue` 为核心的调用或声明。
- **L2334 EN**: Checks an internal invariant in debug builds.
  **L2334 CN**: 在调试构建中检查内部不变式。
- **L2335 EN**: Blank line separating nearby declarations or logic blocks.
  **L2335 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2336 EN**: Comment explains nearby logic, intent, or metadata: `Bitcast to vector<16xi8>`.
  **L2336 CN**: 注释说明附近代码的逻辑、意图或元数据：`Bitcast to vector<16xi8>`。
- **L2337 EN**: Executes a call or declaration centered on `vi8Ty{mlir::VectorType::get`.
  **L2337 CN**: 执行以 `vi8Ty{mlir::VectorType::get` 为核心的调用或声明。
- **L2338 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2338 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2339 EN**: Continues the surrounding expression or declaration: `mlirVecArgs[0] =`.
  **L2339 CN**: 继续构造周围的表达式或声明：`mlirVecArgs[0] =`。
- **L2340 EN**: Continues logic associated with callable symbol `create`.
  **L2340 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L2341 EN**: Executes a call or declaration centered on `.getResult`.
  **L2341 CN**: 执行以 `.getResult` 为核心的调用或声明。
- **L2342 EN**: Continues the surrounding expression or declaration: `mlirVecArgs[1] =`.
  **L2342 CN**: 继续构造周围的表达式或声明：`mlirVecArgs[1] =`。
- **L2343 EN**: Continues logic associated with callable symbol `create`.
  **L2343 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L2344 EN**: Executes a call or declaration centered on `.getResult`.
  **L2344 CN**: 执行以 `.getResult` 为核心的调用或声明。
- **L2345 EN**: Closes the current lexical scope or compound statement.
  **L2345 CN**: 结束当前词法作用域或复合语句块。
- **L2346 EN**: Blank line separating nearby declarations or logic blocks.
  **L2346 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2347 EN**: Comment explains nearby logic, intent, or metadata: `Construct the mask for shuffling`.
  **L2347 CN**: 注释说明附近代码的逻辑、意图或元数据：`Construct the mask for shuffling`。
- **L2348 EN**: Executes a call or declaration centered on `shiftVal{constIntOp.getInt`.
  **L2348 CN**: 执行以 `shiftVal{constIntOp.getInt` 为核心的调用或声明。
- **L2349 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2349 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2350 EN**: Executes a standalone statement or declaration: `shiftVal = shiftVal << 2;`.
  **L2350 CN**: 执行一条独立语句或声明：`shiftVal = shiftVal << 2;`。
- **L2351 EN**: Executes a standalone statement or declaration: `shiftVal &= 0xF;`.
  **L2351 CN**: 执行一条独立语句或声明：`shiftVal &= 0xF;`。
- **L2352 EN**: Executes a standalone statement or declaration: `llvm::SmallVector<int64_t, 16> mask;`.
  **L2352 CN**: 执行一条独立语句或声明：`llvm::SmallVector<int64_t, 16> mask;`。

### Lines 2353-2376

````cpp
    // Shuffle with mask based on the endianness
    const auto triple{fir::getTargetTriple(builder.getModule())};
    if (triple.isLittleEndian()) {
      for (int i = 16; i < 32; ++i)
        mask.push_back(i - shiftVal);
      shftRes = mlir::vector::ShuffleOp::create(builder, loc, mlirVecArgs[1],
                                                mlirVecArgs[0], mask);
    } else {
      for (int i = 0; i < 16; ++i)
        mask.push_back(i + shiftVal);
      shftRes = mlir::vector::ShuffleOp::create(builder, loc, mlirVecArgs[0],
                                                mlirVecArgs[1], mask);
    }

    // Bitcast to the original type
    if (shftRes.getType() != mlirTyArgs[0])
      shftRes =
          mlir::LLVM::BitcastOp::create(builder, loc, mlirTyArgs[0], shftRes);

    return builder.createConvert(loc, resultType, shftRes);
  } else
    llvm_unreachable("Invalid vector operation for generator");

  return shftRes;
````
- **L2353 EN**: Comment explains nearby logic, intent, or metadata: `Shuffle with mask based on the endianness`.
  **L2353 CN**: 注释说明附近代码的逻辑、意图或元数据：`Shuffle with mask based on the endianness`。
- **L2354 EN**: Executes a call or declaration centered on `triple{fir::getTargetTriple`.
  **L2354 CN**: 执行以 `triple{fir::getTargetTriple` 为核心的调用或声明。
- **L2355 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2355 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2356 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L2356 CN**: 开始 `for` 控制流语句并计算其条件。
- **L2357 EN**: Executes a call or declaration centered on `mask.push_back`.
  **L2357 CN**: 执行以 `mask.push_back` 为核心的调用或声明。
- **L2358 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `shftRes = mlir::vector::ShuffleOp::create(builder, loc, mlirVecArgs[1],`.
  **L2358 CN**: 继续一个多行参数列表、初始化器或聚合项：`shftRes = mlir::vector::ShuffleOp::create(builder, loc, mlirVecArgs[1],`。
- **L2359 EN**: Executes a standalone statement or declaration: `mlirVecArgs[0], mask);`.
  **L2359 CN**: 执行一条独立语句或声明：`mlirVecArgs[0], mask);`。
- **L2360 EN**: Transitions from the previous branch into the alternative path.
  **L2360 CN**: 从前一个分支过渡到备选路径。
- **L2361 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L2361 CN**: 开始 `for` 控制流语句并计算其条件。
- **L2362 EN**: Executes a call or declaration centered on `mask.push_back`.
  **L2362 CN**: 执行以 `mask.push_back` 为核心的调用或声明。
- **L2363 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `shftRes = mlir::vector::ShuffleOp::create(builder, loc, mlirVecArgs[0],`.
  **L2363 CN**: 继续一个多行参数列表、初始化器或聚合项：`shftRes = mlir::vector::ShuffleOp::create(builder, loc, mlirVecArgs[0],`。
- **L2364 EN**: Executes a standalone statement or declaration: `mlirVecArgs[1], mask);`.
  **L2364 CN**: 执行一条独立语句或声明：`mlirVecArgs[1], mask);`。
- **L2365 EN**: Closes the current lexical scope or compound statement.
  **L2365 CN**: 结束当前词法作用域或复合语句块。
- **L2366 EN**: Blank line separating nearby declarations or logic blocks.
  **L2366 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2367 EN**: Comment explains nearby logic, intent, or metadata: `Bitcast to the original type`.
  **L2367 CN**: 注释说明附近代码的逻辑、意图或元数据：`Bitcast to the original type`。
- **L2368 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2368 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2369 EN**: Continues the surrounding expression or declaration: `shftRes =`.
  **L2369 CN**: 继续构造周围的表达式或声明：`shftRes =`。
- **L2370 EN**: Executes a call or declaration centered on `mlir::LLVM::BitcastOp::create`.
  **L2370 CN**: 执行以 `mlir::LLVM::BitcastOp::create` 为核心的调用或声明。
- **L2371 EN**: Blank line separating nearby declarations or logic blocks.
  **L2371 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2372 EN**: Returns from the current function with `builder.createConvert(loc, resultType, shftRes)`.
  **L2372 CN**: 以 `builder.createConvert(loc, resultType, shftRes)` 从当前函数返回。
- **L2373 EN**: Transitions from the previous branch into the alternative path.
  **L2373 CN**: 从前一个分支过渡到备选路径。
- **L2374 EN**: Marks this control path as unreachable to LLVM.
  **L2374 CN**: 将该控制路径标记为 LLVM 认为不可达。
- **L2375 EN**: Blank line separating nearby declarations or logic blocks.
  **L2375 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2376 EN**: Returns from the current function with `shftRes`.
  **L2376 CN**: 以 `shftRes` 从当前函数返回。

### Lines 2377-2400

````cpp
}

// VEC_SPLAT, VEC_SPLATS, VEC_SPLAT_S32
template <VecOp vop>
fir::ExtendedValue
PPCIntrinsicLibrary::genVecSplat(mlir::Type resultType,
                                 llvm::ArrayRef<fir::ExtendedValue> args) {
  auto context{builder.getContext()};
  auto argBases{getBasesForArgs(args)};

  mlir::vector::BroadcastOp splatOp{nullptr};
  mlir::Type retTy{nullptr};
  switch (vop) {
  case VecOp::Splat: {
    assert(args.size() == 2);
    auto vecTyInfo{getVecTypeFromFir(argBases[0])};

    auto extractOp{genVecExtract(resultType, args)};
    splatOp = mlir::vector::BroadcastOp::create(
        builder, loc, vecTyInfo.toMlirVectorType(context),
        *(extractOp.getUnboxed()));
    retTy = vecTyInfo.toFirVectorType();
    break;
  }
````
- **L2377 EN**: Closes the current lexical scope or compound statement.
  **L2377 CN**: 结束当前词法作用域或复合语句块。
- **L2378 EN**: Blank line separating nearby declarations or logic blocks.
  **L2378 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2379 EN**: Comment explains nearby logic, intent, or metadata: `VEC_SPLAT, VEC_SPLATS, VEC_SPLAT_S32`.
  **L2379 CN**: 注释说明附近代码的逻辑、意图或元数据：`VEC_SPLAT, VEC_SPLATS, VEC_SPLAT_S32`。
- **L2380 EN**: Introduces template parameters or specialization context: `template <VecOp vop>`.
  **L2380 CN**: 为后续声明引入模板参数或特化上下文：`template <VecOp vop>`。
- **L2381 EN**: Continues the surrounding expression or declaration: `fir::ExtendedValue`.
  **L2381 CN**: 继续构造周围的表达式或声明：`fir::ExtendedValue`。
- **L2382 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PPCIntrinsicLibrary::genVecSplat(mlir::Type resultType,`.
  **L2382 CN**: 继续一个多行参数列表、初始化器或聚合项：`PPCIntrinsicLibrary::genVecSplat(mlir::Type resultType,`。
- **L2383 EN**: Continues the surrounding expression or declaration: `llvm::ArrayRef<fir::ExtendedValue> args) {`.
  **L2383 CN**: 继续构造周围的表达式或声明：`llvm::ArrayRef<fir::ExtendedValue> args) {`。
- **L2384 EN**: Executes a call or declaration centered on `context{builder.getContext`.
  **L2384 CN**: 执行以 `context{builder.getContext` 为核心的调用或声明。
- **L2385 EN**: Executes a call or declaration centered on `argBases{getBasesForArgs`.
  **L2385 CN**: 执行以 `argBases{getBasesForArgs` 为核心的调用或声明。
- **L2386 EN**: Blank line separating nearby declarations or logic blocks.
  **L2386 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2387 EN**: Executes a standalone statement or declaration: `mlir::vector::BroadcastOp splatOp{nullptr};`.
  **L2387 CN**: 执行一条独立语句或声明：`mlir::vector::BroadcastOp splatOp{nullptr};`。
- **L2388 EN**: Executes a standalone statement or declaration: `mlir::Type retTy{nullptr};`.
  **L2388 CN**: 执行一条独立语句或声明：`mlir::Type retTy{nullptr};`。
- **L2389 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L2389 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L2390 EN**: Introduces a switch dispatch label: `case VecOp::Splat: {`.
  **L2390 CN**: 引入一个 switch 分发标签：`case VecOp::Splat: {`。
- **L2391 EN**: Checks an internal invariant in debug builds.
  **L2391 CN**: 在调试构建中检查内部不变式。
- **L2392 EN**: Executes a call or declaration centered on `vecTyInfo{getVecTypeFromFir`.
  **L2392 CN**: 执行以 `vecTyInfo{getVecTypeFromFir` 为核心的调用或声明。
- **L2393 EN**: Blank line separating nearby declarations or logic blocks.
  **L2393 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2394 EN**: Executes a call or declaration centered on `extractOp{genVecExtract`.
  **L2394 CN**: 执行以 `extractOp{genVecExtract` 为核心的调用或声明。
- **L2395 EN**: Continues logic associated with callable symbol `create`.
  **L2395 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L2396 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `builder, loc, vecTyInfo.toMlirVectorType(context),`.
  **L2396 CN**: 继续一个多行参数列表、初始化器或聚合项：`builder, loc, vecTyInfo.toMlirVectorType(context),`。
- **L2397 EN**: Comment explains nearby logic, intent, or metadata: `(extractOp.getUnboxed()));`.
  **L2397 CN**: 注释说明附近代码的逻辑、意图或元数据：`(extractOp.getUnboxed()));`。
- **L2398 EN**: Executes a call or declaration centered on `vecTyInfo.toFirVectorType`.
  **L2398 CN**: 执行以 `vecTyInfo.toFirVectorType` 为核心的调用或声明。
- **L2399 EN**: Exits the nearest loop or switch statement.
  **L2399 CN**: 退出最近的循环或 switch 语句。
- **L2400 EN**: Closes the current lexical scope or compound statement.
  **L2400 CN**: 结束当前词法作用域或复合语句块。

### Lines 2401-2424

````cpp
  case VecOp::Splats: {
    assert(args.size() == 1);
    auto vecTyInfo{getVecTypeFromEle(argBases[0])};

    splatOp = mlir::vector::BroadcastOp::create(
        builder, loc, vecTyInfo.toMlirVectorType(context), argBases[0]);
    retTy = vecTyInfo.toFirVectorType();
    break;
  }
  case VecOp::Splat_s32: {
    assert(args.size() == 1);
    auto eleTy{builder.getIntegerType(32)};
    auto intOp{builder.createConvert(loc, eleTy, argBases[0])};

    // the intrinsic always returns vector(integer(4))
    splatOp = mlir::vector::BroadcastOp::create(
        builder, loc, mlir::VectorType::get(4, eleTy), intOp);
    retTy = fir::VectorType::get(4, eleTy);
    break;
  }
  default:
    llvm_unreachable("invalid vector operation for generator");
  }
  return builder.createConvert(loc, retTy, splatOp);
````
- **L2401 EN**: Introduces a switch dispatch label: `case VecOp::Splats: {`.
  **L2401 CN**: 引入一个 switch 分发标签：`case VecOp::Splats: {`。
- **L2402 EN**: Checks an internal invariant in debug builds.
  **L2402 CN**: 在调试构建中检查内部不变式。
- **L2403 EN**: Executes a call or declaration centered on `vecTyInfo{getVecTypeFromEle`.
  **L2403 CN**: 执行以 `vecTyInfo{getVecTypeFromEle` 为核心的调用或声明。
- **L2404 EN**: Blank line separating nearby declarations or logic blocks.
  **L2404 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2405 EN**: Continues logic associated with callable symbol `create`.
  **L2405 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L2406 EN**: Executes a call or declaration centered on `vecTyInfo.toMlirVectorType`.
  **L2406 CN**: 执行以 `vecTyInfo.toMlirVectorType` 为核心的调用或声明。
- **L2407 EN**: Executes a call or declaration centered on `vecTyInfo.toFirVectorType`.
  **L2407 CN**: 执行以 `vecTyInfo.toFirVectorType` 为核心的调用或声明。
- **L2408 EN**: Exits the nearest loop or switch statement.
  **L2408 CN**: 退出最近的循环或 switch 语句。
- **L2409 EN**: Closes the current lexical scope or compound statement.
  **L2409 CN**: 结束当前词法作用域或复合语句块。
- **L2410 EN**: Introduces a switch dispatch label: `case VecOp::Splat_s32: {`.
  **L2410 CN**: 引入一个 switch 分发标签：`case VecOp::Splat_s32: {`。
- **L2411 EN**: Checks an internal invariant in debug builds.
  **L2411 CN**: 在调试构建中检查内部不变式。
- **L2412 EN**: Executes a call or declaration centered on `eleTy{builder.getIntegerType`.
  **L2412 CN**: 执行以 `eleTy{builder.getIntegerType` 为核心的调用或声明。
- **L2413 EN**: Executes a call or declaration centered on `intOp{builder.createConvert`.
  **L2413 CN**: 执行以 `intOp{builder.createConvert` 为核心的调用或声明。
- **L2414 EN**: Blank line separating nearby declarations or logic blocks.
  **L2414 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2415 EN**: Comment explains nearby logic, intent, or metadata: `the intrinsic always returns vector(integer(4))`.
  **L2415 CN**: 注释说明附近代码的逻辑、意图或元数据：`the intrinsic always returns vector(integer(4))`。
- **L2416 EN**: Continues logic associated with callable symbol `create`.
  **L2416 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L2417 EN**: Executes a call or declaration centered on `mlir::VectorType::get`.
  **L2417 CN**: 执行以 `mlir::VectorType::get` 为核心的调用或声明。
- **L2418 EN**: Executes a call or declaration centered on `fir::VectorType::get`.
  **L2418 CN**: 执行以 `fir::VectorType::get` 为核心的调用或声明。
- **L2419 EN**: Exits the nearest loop or switch statement.
  **L2419 CN**: 退出最近的循环或 switch 语句。
- **L2420 EN**: Closes the current lexical scope or compound statement.
  **L2420 CN**: 结束当前词法作用域或复合语句块。
- **L2421 EN**: Introduces a switch dispatch label: `default:`.
  **L2421 CN**: 引入一个 switch 分发标签：`default:`。
- **L2422 EN**: Marks this control path as unreachable to LLVM.
  **L2422 CN**: 将该控制路径标记为 LLVM 认为不可达。
- **L2423 EN**: Closes the current lexical scope or compound statement.
  **L2423 CN**: 结束当前词法作用域或复合语句块。
- **L2424 EN**: Returns from the current function with `builder.createConvert(loc, retTy, splatOp)`.
  **L2424 CN**: 以 `builder.createConvert(loc, retTy, splatOp)` 从当前函数返回。

### Lines 2425-2448

````cpp
}

fir::ExtendedValue
PPCIntrinsicLibrary::genVecXlds(mlir::Type resultType,
                                llvm::ArrayRef<fir::ExtendedValue> args) {
  assert(args.size() == 2);
  auto arg0{getBase(args[0])};
  auto arg1{getBase(args[1])};

  // Prepare the return type in FIR.
  auto vecTyInfo{getVecTypeFromFirType(resultType)};
  auto mlirTy{vecTyInfo.toMlirVectorType(builder.getContext())};
  auto firTy{vecTyInfo.toFirVectorType()};

  // Add the %val of arg0 to %addr of arg1
  auto addr{addOffsetToAddress(builder, loc, arg1, arg0)};

  auto i64Ty{mlir::IntegerType::get(builder.getContext(), 64)};
  auto i64VecTy{mlir::VectorType::get(2, i64Ty)};
  auto i64RefTy{builder.getRefType(i64Ty)};
  auto addrConv{fir::ConvertOp::create(builder, loc, i64RefTy, addr)};

  auto addrVal{fir::LoadOp::create(builder, loc, addrConv)};
  auto splatRes{
````
- **L2425 EN**: Closes the current lexical scope or compound statement.
  **L2425 CN**: 结束当前词法作用域或复合语句块。
- **L2426 EN**: Blank line separating nearby declarations or logic blocks.
  **L2426 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2427 EN**: Continues the surrounding expression or declaration: `fir::ExtendedValue`.
  **L2427 CN**: 继续构造周围的表达式或声明：`fir::ExtendedValue`。
- **L2428 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PPCIntrinsicLibrary::genVecXlds(mlir::Type resultType,`.
  **L2428 CN**: 继续一个多行参数列表、初始化器或聚合项：`PPCIntrinsicLibrary::genVecXlds(mlir::Type resultType,`。
- **L2429 EN**: Continues the surrounding expression or declaration: `llvm::ArrayRef<fir::ExtendedValue> args) {`.
  **L2429 CN**: 继续构造周围的表达式或声明：`llvm::ArrayRef<fir::ExtendedValue> args) {`。
- **L2430 EN**: Checks an internal invariant in debug builds.
  **L2430 CN**: 在调试构建中检查内部不变式。
- **L2431 EN**: Executes a call or declaration centered on `arg0{getBase`.
  **L2431 CN**: 执行以 `arg0{getBase` 为核心的调用或声明。
- **L2432 EN**: Executes a call or declaration centered on `arg1{getBase`.
  **L2432 CN**: 执行以 `arg1{getBase` 为核心的调用或声明。
- **L2433 EN**: Blank line separating nearby declarations or logic blocks.
  **L2433 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2434 EN**: Comment explains nearby logic, intent, or metadata: `Prepare the return type in FIR.`.
  **L2434 CN**: 注释说明附近代码的逻辑、意图或元数据：`Prepare the return type in FIR.`。
- **L2435 EN**: Executes a call or declaration centered on `vecTyInfo{getVecTypeFromFirType`.
  **L2435 CN**: 执行以 `vecTyInfo{getVecTypeFromFirType` 为核心的调用或声明。
- **L2436 EN**: Executes a call or declaration centered on `mlirTy{vecTyInfo.toMlirVectorType`.
  **L2436 CN**: 执行以 `mlirTy{vecTyInfo.toMlirVectorType` 为核心的调用或声明。
- **L2437 EN**: Executes a call or declaration centered on `firTy{vecTyInfo.toFirVectorType`.
  **L2437 CN**: 执行以 `firTy{vecTyInfo.toFirVectorType` 为核心的调用或声明。
- **L2438 EN**: Blank line separating nearby declarations or logic blocks.
  **L2438 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2439 EN**: Comment explains nearby logic, intent, or metadata: `Add the %val of arg0 to %addr of arg1`.
  **L2439 CN**: 注释说明附近代码的逻辑、意图或元数据：`Add the %val of arg0 to %addr of arg1`。
- **L2440 EN**: Executes a call or declaration centered on `addr{addOffsetToAddress`.
  **L2440 CN**: 执行以 `addr{addOffsetToAddress` 为核心的调用或声明。
- **L2441 EN**: Blank line separating nearby declarations or logic blocks.
  **L2441 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2442 EN**: Executes a call or declaration centered on `i64Ty{mlir::IntegerType::get`.
  **L2442 CN**: 执行以 `i64Ty{mlir::IntegerType::get` 为核心的调用或声明。
- **L2443 EN**: Executes a call or declaration centered on `i64VecTy{mlir::VectorType::get`.
  **L2443 CN**: 执行以 `i64VecTy{mlir::VectorType::get` 为核心的调用或声明。
- **L2444 EN**: Executes a call or declaration centered on `i64RefTy{builder.getRefType`.
  **L2444 CN**: 执行以 `i64RefTy{builder.getRefType` 为核心的调用或声明。
- **L2445 EN**: Executes a call or declaration centered on `addrConv{fir::ConvertOp::create`.
  **L2445 CN**: 执行以 `addrConv{fir::ConvertOp::create` 为核心的调用或声明。
- **L2446 EN**: Blank line separating nearby declarations or logic blocks.
  **L2446 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2447 EN**: Executes a call or declaration centered on `addrVal{fir::LoadOp::create`.
  **L2447 CN**: 执行以 `addrVal{fir::LoadOp::create` 为核心的调用或声明。
- **L2448 EN**: Continues the surrounding expression or declaration: `auto splatRes{`.
  **L2448 CN**: 继续构造周围的表达式或声明：`auto splatRes{`。

### Lines 2449-2472

````cpp
      mlir::vector::BroadcastOp::create(builder, loc, i64VecTy, addrVal)};

  mlir::Value result{nullptr};
  if (mlirTy != splatRes.getType()) {
    result = mlir::vector::BitCastOp::create(builder, loc, mlirTy, splatRes);
  } else
    result = splatRes;

  return builder.createConvert(loc, firTy, result);
}

const char *getMmaIrIntrName(MMAOp mmaOp) {
  switch (mmaOp) {
  case MMAOp::AssembleAcc:
    return "llvm.ppc.mma.assemble.acc";
  case MMAOp::AssemblePair:
    return "llvm.ppc.vsx.assemble.pair";
  case MMAOp::DisassembleAcc:
    return "llvm.ppc.mma.disassemble.acc";
  case MMAOp::DisassemblePair:
    return "llvm.ppc.vsx.disassemble.pair";
  case MMAOp::Xxmfacc:
    return "llvm.ppc.mma.xxmfacc";
  case MMAOp::Xxmtacc:
````
- **L2449 EN**: Executes a call or declaration centered on `mlir::vector::BroadcastOp::create`.
  **L2449 CN**: 执行以 `mlir::vector::BroadcastOp::create` 为核心的调用或声明。
- **L2450 EN**: Blank line separating nearby declarations or logic blocks.
  **L2450 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2451 EN**: Executes a standalone statement or declaration: `mlir::Value result{nullptr};`.
  **L2451 CN**: 执行一条独立语句或声明：`mlir::Value result{nullptr};`。
- **L2452 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2452 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2453 EN**: Executes a call or declaration centered on `mlir::vector::BitCastOp::create`.
  **L2453 CN**: 执行以 `mlir::vector::BitCastOp::create` 为核心的调用或声明。
- **L2454 EN**: Transitions from the previous branch into the alternative path.
  **L2454 CN**: 从前一个分支过渡到备选路径。
- **L2455 EN**: Executes a standalone statement or declaration: `result = splatRes;`.
  **L2455 CN**: 执行一条独立语句或声明：`result = splatRes;`。
- **L2456 EN**: Blank line separating nearby declarations or logic blocks.
  **L2456 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2457 EN**: Returns from the current function with `builder.createConvert(loc, firTy, result)`.
  **L2457 CN**: 以 `builder.createConvert(loc, firTy, result)` 从当前函数返回。
- **L2458 EN**: Closes the current lexical scope or compound statement.
  **L2458 CN**: 结束当前词法作用域或复合语句块。
- **L2459 EN**: Blank line separating nearby declarations or logic blocks.
  **L2459 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2460 EN**: Starts a function, method, lambda, or structured scope: `const char *getMmaIrIntrName(MMAOp mmaOp) {`.
  **L2460 CN**: 开始一个函数、方法、lambda 或结构化作用域：`const char *getMmaIrIntrName(MMAOp mmaOp) {`。
- **L2461 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L2461 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L2462 EN**: Introduces a switch dispatch label: `case MMAOp::AssembleAcc:`.
  **L2462 CN**: 引入一个 switch 分发标签：`case MMAOp::AssembleAcc:`。
- **L2463 EN**: Returns from the current function with `"llvm.ppc.mma.assemble.acc"`.
  **L2463 CN**: 以 `"llvm.ppc.mma.assemble.acc"` 从当前函数返回。
- **L2464 EN**: Introduces a switch dispatch label: `case MMAOp::AssemblePair:`.
  **L2464 CN**: 引入一个 switch 分发标签：`case MMAOp::AssemblePair:`。
- **L2465 EN**: Returns from the current function with `"llvm.ppc.vsx.assemble.pair"`.
  **L2465 CN**: 以 `"llvm.ppc.vsx.assemble.pair"` 从当前函数返回。
- **L2466 EN**: Introduces a switch dispatch label: `case MMAOp::DisassembleAcc:`.
  **L2466 CN**: 引入一个 switch 分发标签：`case MMAOp::DisassembleAcc:`。
- **L2467 EN**: Returns from the current function with `"llvm.ppc.mma.disassemble.acc"`.
  **L2467 CN**: 以 `"llvm.ppc.mma.disassemble.acc"` 从当前函数返回。
- **L2468 EN**: Introduces a switch dispatch label: `case MMAOp::DisassemblePair:`.
  **L2468 CN**: 引入一个 switch 分发标签：`case MMAOp::DisassemblePair:`。
- **L2469 EN**: Returns from the current function with `"llvm.ppc.vsx.disassemble.pair"`.
  **L2469 CN**: 以 `"llvm.ppc.vsx.disassemble.pair"` 从当前函数返回。
- **L2470 EN**: Introduces a switch dispatch label: `case MMAOp::Xxmfacc:`.
  **L2470 CN**: 引入一个 switch 分发标签：`case MMAOp::Xxmfacc:`。
- **L2471 EN**: Returns from the current function with `"llvm.ppc.mma.xxmfacc"`.
  **L2471 CN**: 以 `"llvm.ppc.mma.xxmfacc"` 从当前函数返回。
- **L2472 EN**: Introduces a switch dispatch label: `case MMAOp::Xxmtacc:`.
  **L2472 CN**: 引入一个 switch 分发标签：`case MMAOp::Xxmtacc:`。

### Lines 2473-2496

````cpp
    return "llvm.ppc.mma.xxmtacc";
  case MMAOp::Xxsetaccz:
    return "llvm.ppc.mma.xxsetaccz";
  case MMAOp::Pmxvbf16ger2:
    return "llvm.ppc.mma.pmxvbf16ger2";
  case MMAOp::Pmxvbf16ger2nn:
    return "llvm.ppc.mma.pmxvbf16ger2nn";
  case MMAOp::Pmxvbf16ger2np:
    return "llvm.ppc.mma.pmxvbf16ger2np";
  case MMAOp::Pmxvbf16ger2pn:
    return "llvm.ppc.mma.pmxvbf16ger2pn";
  case MMAOp::Pmxvbf16ger2pp:
    return "llvm.ppc.mma.pmxvbf16ger2pp";
  case MMAOp::Pmxvf16ger2:
    return "llvm.ppc.mma.pmxvf16ger2";
  case MMAOp::Pmxvf16ger2nn:
    return "llvm.ppc.mma.pmxvf16ger2nn";
  case MMAOp::Pmxvf16ger2np:
    return "llvm.ppc.mma.pmxvf16ger2np";
  case MMAOp::Pmxvf16ger2pn:
    return "llvm.ppc.mma.pmxvf16ger2pn";
  case MMAOp::Pmxvf16ger2pp:
    return "llvm.ppc.mma.pmxvf16ger2pp";
  case MMAOp::Pmxvf32ger:
````
- **L2473 EN**: Returns from the current function with `"llvm.ppc.mma.xxmtacc"`.
  **L2473 CN**: 以 `"llvm.ppc.mma.xxmtacc"` 从当前函数返回。
- **L2474 EN**: Introduces a switch dispatch label: `case MMAOp::Xxsetaccz:`.
  **L2474 CN**: 引入一个 switch 分发标签：`case MMAOp::Xxsetaccz:`。
- **L2475 EN**: Returns from the current function with `"llvm.ppc.mma.xxsetaccz"`.
  **L2475 CN**: 以 `"llvm.ppc.mma.xxsetaccz"` 从当前函数返回。
- **L2476 EN**: Introduces a switch dispatch label: `case MMAOp::Pmxvbf16ger2:`.
  **L2476 CN**: 引入一个 switch 分发标签：`case MMAOp::Pmxvbf16ger2:`。
- **L2477 EN**: Returns from the current function with `"llvm.ppc.mma.pmxvbf16ger2"`.
  **L2477 CN**: 以 `"llvm.ppc.mma.pmxvbf16ger2"` 从当前函数返回。
- **L2478 EN**: Introduces a switch dispatch label: `case MMAOp::Pmxvbf16ger2nn:`.
  **L2478 CN**: 引入一个 switch 分发标签：`case MMAOp::Pmxvbf16ger2nn:`。
- **L2479 EN**: Returns from the current function with `"llvm.ppc.mma.pmxvbf16ger2nn"`.
  **L2479 CN**: 以 `"llvm.ppc.mma.pmxvbf16ger2nn"` 从当前函数返回。
- **L2480 EN**: Introduces a switch dispatch label: `case MMAOp::Pmxvbf16ger2np:`.
  **L2480 CN**: 引入一个 switch 分发标签：`case MMAOp::Pmxvbf16ger2np:`。
- **L2481 EN**: Returns from the current function with `"llvm.ppc.mma.pmxvbf16ger2np"`.
  **L2481 CN**: 以 `"llvm.ppc.mma.pmxvbf16ger2np"` 从当前函数返回。
- **L2482 EN**: Introduces a switch dispatch label: `case MMAOp::Pmxvbf16ger2pn:`.
  **L2482 CN**: 引入一个 switch 分发标签：`case MMAOp::Pmxvbf16ger2pn:`。
- **L2483 EN**: Returns from the current function with `"llvm.ppc.mma.pmxvbf16ger2pn"`.
  **L2483 CN**: 以 `"llvm.ppc.mma.pmxvbf16ger2pn"` 从当前函数返回。
- **L2484 EN**: Introduces a switch dispatch label: `case MMAOp::Pmxvbf16ger2pp:`.
  **L2484 CN**: 引入一个 switch 分发标签：`case MMAOp::Pmxvbf16ger2pp:`。
- **L2485 EN**: Returns from the current function with `"llvm.ppc.mma.pmxvbf16ger2pp"`.
  **L2485 CN**: 以 `"llvm.ppc.mma.pmxvbf16ger2pp"` 从当前函数返回。
- **L2486 EN**: Introduces a switch dispatch label: `case MMAOp::Pmxvf16ger2:`.
  **L2486 CN**: 引入一个 switch 分发标签：`case MMAOp::Pmxvf16ger2:`。
- **L2487 EN**: Returns from the current function with `"llvm.ppc.mma.pmxvf16ger2"`.
  **L2487 CN**: 以 `"llvm.ppc.mma.pmxvf16ger2"` 从当前函数返回。
- **L2488 EN**: Introduces a switch dispatch label: `case MMAOp::Pmxvf16ger2nn:`.
  **L2488 CN**: 引入一个 switch 分发标签：`case MMAOp::Pmxvf16ger2nn:`。
- **L2489 EN**: Returns from the current function with `"llvm.ppc.mma.pmxvf16ger2nn"`.
  **L2489 CN**: 以 `"llvm.ppc.mma.pmxvf16ger2nn"` 从当前函数返回。
- **L2490 EN**: Introduces a switch dispatch label: `case MMAOp::Pmxvf16ger2np:`.
  **L2490 CN**: 引入一个 switch 分发标签：`case MMAOp::Pmxvf16ger2np:`。
- **L2491 EN**: Returns from the current function with `"llvm.ppc.mma.pmxvf16ger2np"`.
  **L2491 CN**: 以 `"llvm.ppc.mma.pmxvf16ger2np"` 从当前函数返回。
- **L2492 EN**: Introduces a switch dispatch label: `case MMAOp::Pmxvf16ger2pn:`.
  **L2492 CN**: 引入一个 switch 分发标签：`case MMAOp::Pmxvf16ger2pn:`。
- **L2493 EN**: Returns from the current function with `"llvm.ppc.mma.pmxvf16ger2pn"`.
  **L2493 CN**: 以 `"llvm.ppc.mma.pmxvf16ger2pn"` 从当前函数返回。
- **L2494 EN**: Introduces a switch dispatch label: `case MMAOp::Pmxvf16ger2pp:`.
  **L2494 CN**: 引入一个 switch 分发标签：`case MMAOp::Pmxvf16ger2pp:`。
- **L2495 EN**: Returns from the current function with `"llvm.ppc.mma.pmxvf16ger2pp"`.
  **L2495 CN**: 以 `"llvm.ppc.mma.pmxvf16ger2pp"` 从当前函数返回。
- **L2496 EN**: Introduces a switch dispatch label: `case MMAOp::Pmxvf32ger:`.
  **L2496 CN**: 引入一个 switch 分发标签：`case MMAOp::Pmxvf32ger:`。

### Lines 2497-2520

````cpp
    return "llvm.ppc.mma.pmxvf32ger";
  case MMAOp::Pmxvf32gernn:
    return "llvm.ppc.mma.pmxvf32gernn";
  case MMAOp::Pmxvf32gernp:
    return "llvm.ppc.mma.pmxvf32gernp";
  case MMAOp::Pmxvf32gerpn:
    return "llvm.ppc.mma.pmxvf32gerpn";
  case MMAOp::Pmxvf32gerpp:
    return "llvm.ppc.mma.pmxvf32gerpp";
  case MMAOp::Pmxvf64ger:
    return "llvm.ppc.mma.pmxvf64ger";
  case MMAOp::Pmxvf64gernn:
    return "llvm.ppc.mma.pmxvf64gernn";
  case MMAOp::Pmxvf64gernp:
    return "llvm.ppc.mma.pmxvf64gernp";
  case MMAOp::Pmxvf64gerpn:
    return "llvm.ppc.mma.pmxvf64gerpn";
  case MMAOp::Pmxvf64gerpp:
    return "llvm.ppc.mma.pmxvf64gerpp";
  case MMAOp::Pmxvi16ger2:
    return "llvm.ppc.mma.pmxvi16ger2";
  case MMAOp::Pmxvi16ger2pp:
    return "llvm.ppc.mma.pmxvi16ger2pp";
  case MMAOp::Pmxvi16ger2s:
````
- **L2497 EN**: Returns from the current function with `"llvm.ppc.mma.pmxvf32ger"`.
  **L2497 CN**: 以 `"llvm.ppc.mma.pmxvf32ger"` 从当前函数返回。
- **L2498 EN**: Introduces a switch dispatch label: `case MMAOp::Pmxvf32gernn:`.
  **L2498 CN**: 引入一个 switch 分发标签：`case MMAOp::Pmxvf32gernn:`。
- **L2499 EN**: Returns from the current function with `"llvm.ppc.mma.pmxvf32gernn"`.
  **L2499 CN**: 以 `"llvm.ppc.mma.pmxvf32gernn"` 从当前函数返回。
- **L2500 EN**: Introduces a switch dispatch label: `case MMAOp::Pmxvf32gernp:`.
  **L2500 CN**: 引入一个 switch 分发标签：`case MMAOp::Pmxvf32gernp:`。
- **L2501 EN**: Returns from the current function with `"llvm.ppc.mma.pmxvf32gernp"`.
  **L2501 CN**: 以 `"llvm.ppc.mma.pmxvf32gernp"` 从当前函数返回。
- **L2502 EN**: Introduces a switch dispatch label: `case MMAOp::Pmxvf32gerpn:`.
  **L2502 CN**: 引入一个 switch 分发标签：`case MMAOp::Pmxvf32gerpn:`。
- **L2503 EN**: Returns from the current function with `"llvm.ppc.mma.pmxvf32gerpn"`.
  **L2503 CN**: 以 `"llvm.ppc.mma.pmxvf32gerpn"` 从当前函数返回。
- **L2504 EN**: Introduces a switch dispatch label: `case MMAOp::Pmxvf32gerpp:`.
  **L2504 CN**: 引入一个 switch 分发标签：`case MMAOp::Pmxvf32gerpp:`。
- **L2505 EN**: Returns from the current function with `"llvm.ppc.mma.pmxvf32gerpp"`.
  **L2505 CN**: 以 `"llvm.ppc.mma.pmxvf32gerpp"` 从当前函数返回。
- **L2506 EN**: Introduces a switch dispatch label: `case MMAOp::Pmxvf64ger:`.
  **L2506 CN**: 引入一个 switch 分发标签：`case MMAOp::Pmxvf64ger:`。
- **L2507 EN**: Returns from the current function with `"llvm.ppc.mma.pmxvf64ger"`.
  **L2507 CN**: 以 `"llvm.ppc.mma.pmxvf64ger"` 从当前函数返回。
- **L2508 EN**: Introduces a switch dispatch label: `case MMAOp::Pmxvf64gernn:`.
  **L2508 CN**: 引入一个 switch 分发标签：`case MMAOp::Pmxvf64gernn:`。
- **L2509 EN**: Returns from the current function with `"llvm.ppc.mma.pmxvf64gernn"`.
  **L2509 CN**: 以 `"llvm.ppc.mma.pmxvf64gernn"` 从当前函数返回。
- **L2510 EN**: Introduces a switch dispatch label: `case MMAOp::Pmxvf64gernp:`.
  **L2510 CN**: 引入一个 switch 分发标签：`case MMAOp::Pmxvf64gernp:`。
- **L2511 EN**: Returns from the current function with `"llvm.ppc.mma.pmxvf64gernp"`.
  **L2511 CN**: 以 `"llvm.ppc.mma.pmxvf64gernp"` 从当前函数返回。
- **L2512 EN**: Introduces a switch dispatch label: `case MMAOp::Pmxvf64gerpn:`.
  **L2512 CN**: 引入一个 switch 分发标签：`case MMAOp::Pmxvf64gerpn:`。
- **L2513 EN**: Returns from the current function with `"llvm.ppc.mma.pmxvf64gerpn"`.
  **L2513 CN**: 以 `"llvm.ppc.mma.pmxvf64gerpn"` 从当前函数返回。
- **L2514 EN**: Introduces a switch dispatch label: `case MMAOp::Pmxvf64gerpp:`.
  **L2514 CN**: 引入一个 switch 分发标签：`case MMAOp::Pmxvf64gerpp:`。
- **L2515 EN**: Returns from the current function with `"llvm.ppc.mma.pmxvf64gerpp"`.
  **L2515 CN**: 以 `"llvm.ppc.mma.pmxvf64gerpp"` 从当前函数返回。
- **L2516 EN**: Introduces a switch dispatch label: `case MMAOp::Pmxvi16ger2:`.
  **L2516 CN**: 引入一个 switch 分发标签：`case MMAOp::Pmxvi16ger2:`。
- **L2517 EN**: Returns from the current function with `"llvm.ppc.mma.pmxvi16ger2"`.
  **L2517 CN**: 以 `"llvm.ppc.mma.pmxvi16ger2"` 从当前函数返回。
- **L2518 EN**: Introduces a switch dispatch label: `case MMAOp::Pmxvi16ger2pp:`.
  **L2518 CN**: 引入一个 switch 分发标签：`case MMAOp::Pmxvi16ger2pp:`。
- **L2519 EN**: Returns from the current function with `"llvm.ppc.mma.pmxvi16ger2pp"`.
  **L2519 CN**: 以 `"llvm.ppc.mma.pmxvi16ger2pp"` 从当前函数返回。
- **L2520 EN**: Introduces a switch dispatch label: `case MMAOp::Pmxvi16ger2s:`.
  **L2520 CN**: 引入一个 switch 分发标签：`case MMAOp::Pmxvi16ger2s:`。

### Lines 2521-2544

````cpp
    return "llvm.ppc.mma.pmxvi16ger2s";
  case MMAOp::Pmxvi16ger2spp:
    return "llvm.ppc.mma.pmxvi16ger2spp";
  case MMAOp::Pmxvi4ger8:
    return "llvm.ppc.mma.pmxvi4ger8";
  case MMAOp::Pmxvi4ger8pp:
    return "llvm.ppc.mma.pmxvi4ger8pp";
  case MMAOp::Pmxvi8ger4:
    return "llvm.ppc.mma.pmxvi8ger4";
  case MMAOp::Pmxvi8ger4pp:
    return "llvm.ppc.mma.pmxvi8ger4pp";
  case MMAOp::Pmxvi8ger4spp:
    return "llvm.ppc.mma.pmxvi8ger4spp";
  case MMAOp::Xvbf16ger2:
    return "llvm.ppc.mma.xvbf16ger2";
  case MMAOp::Xvbf16ger2nn:
    return "llvm.ppc.mma.xvbf16ger2nn";
  case MMAOp::Xvbf16ger2np:
    return "llvm.ppc.mma.xvbf16ger2np";
  case MMAOp::Xvbf16ger2pn:
    return "llvm.ppc.mma.xvbf16ger2pn";
  case MMAOp::Xvbf16ger2pp:
    return "llvm.ppc.mma.xvbf16ger2pp";
  case MMAOp::Xvf16ger2:
````
- **L2521 EN**: Returns from the current function with `"llvm.ppc.mma.pmxvi16ger2s"`.
  **L2521 CN**: 以 `"llvm.ppc.mma.pmxvi16ger2s"` 从当前函数返回。
- **L2522 EN**: Introduces a switch dispatch label: `case MMAOp::Pmxvi16ger2spp:`.
  **L2522 CN**: 引入一个 switch 分发标签：`case MMAOp::Pmxvi16ger2spp:`。
- **L2523 EN**: Returns from the current function with `"llvm.ppc.mma.pmxvi16ger2spp"`.
  **L2523 CN**: 以 `"llvm.ppc.mma.pmxvi16ger2spp"` 从当前函数返回。
- **L2524 EN**: Introduces a switch dispatch label: `case MMAOp::Pmxvi4ger8:`.
  **L2524 CN**: 引入一个 switch 分发标签：`case MMAOp::Pmxvi4ger8:`。
- **L2525 EN**: Returns from the current function with `"llvm.ppc.mma.pmxvi4ger8"`.
  **L2525 CN**: 以 `"llvm.ppc.mma.pmxvi4ger8"` 从当前函数返回。
- **L2526 EN**: Introduces a switch dispatch label: `case MMAOp::Pmxvi4ger8pp:`.
  **L2526 CN**: 引入一个 switch 分发标签：`case MMAOp::Pmxvi4ger8pp:`。
- **L2527 EN**: Returns from the current function with `"llvm.ppc.mma.pmxvi4ger8pp"`.
  **L2527 CN**: 以 `"llvm.ppc.mma.pmxvi4ger8pp"` 从当前函数返回。
- **L2528 EN**: Introduces a switch dispatch label: `case MMAOp::Pmxvi8ger4:`.
  **L2528 CN**: 引入一个 switch 分发标签：`case MMAOp::Pmxvi8ger4:`。
- **L2529 EN**: Returns from the current function with `"llvm.ppc.mma.pmxvi8ger4"`.
  **L2529 CN**: 以 `"llvm.ppc.mma.pmxvi8ger4"` 从当前函数返回。
- **L2530 EN**: Introduces a switch dispatch label: `case MMAOp::Pmxvi8ger4pp:`.
  **L2530 CN**: 引入一个 switch 分发标签：`case MMAOp::Pmxvi8ger4pp:`。
- **L2531 EN**: Returns from the current function with `"llvm.ppc.mma.pmxvi8ger4pp"`.
  **L2531 CN**: 以 `"llvm.ppc.mma.pmxvi8ger4pp"` 从当前函数返回。
- **L2532 EN**: Introduces a switch dispatch label: `case MMAOp::Pmxvi8ger4spp:`.
  **L2532 CN**: 引入一个 switch 分发标签：`case MMAOp::Pmxvi8ger4spp:`。
- **L2533 EN**: Returns from the current function with `"llvm.ppc.mma.pmxvi8ger4spp"`.
  **L2533 CN**: 以 `"llvm.ppc.mma.pmxvi8ger4spp"` 从当前函数返回。
- **L2534 EN**: Introduces a switch dispatch label: `case MMAOp::Xvbf16ger2:`.
  **L2534 CN**: 引入一个 switch 分发标签：`case MMAOp::Xvbf16ger2:`。
- **L2535 EN**: Returns from the current function with `"llvm.ppc.mma.xvbf16ger2"`.
  **L2535 CN**: 以 `"llvm.ppc.mma.xvbf16ger2"` 从当前函数返回。
- **L2536 EN**: Introduces a switch dispatch label: `case MMAOp::Xvbf16ger2nn:`.
  **L2536 CN**: 引入一个 switch 分发标签：`case MMAOp::Xvbf16ger2nn:`。
- **L2537 EN**: Returns from the current function with `"llvm.ppc.mma.xvbf16ger2nn"`.
  **L2537 CN**: 以 `"llvm.ppc.mma.xvbf16ger2nn"` 从当前函数返回。
- **L2538 EN**: Introduces a switch dispatch label: `case MMAOp::Xvbf16ger2np:`.
  **L2538 CN**: 引入一个 switch 分发标签：`case MMAOp::Xvbf16ger2np:`。
- **L2539 EN**: Returns from the current function with `"llvm.ppc.mma.xvbf16ger2np"`.
  **L2539 CN**: 以 `"llvm.ppc.mma.xvbf16ger2np"` 从当前函数返回。
- **L2540 EN**: Introduces a switch dispatch label: `case MMAOp::Xvbf16ger2pn:`.
  **L2540 CN**: 引入一个 switch 分发标签：`case MMAOp::Xvbf16ger2pn:`。
- **L2541 EN**: Returns from the current function with `"llvm.ppc.mma.xvbf16ger2pn"`.
  **L2541 CN**: 以 `"llvm.ppc.mma.xvbf16ger2pn"` 从当前函数返回。
- **L2542 EN**: Introduces a switch dispatch label: `case MMAOp::Xvbf16ger2pp:`.
  **L2542 CN**: 引入一个 switch 分发标签：`case MMAOp::Xvbf16ger2pp:`。
- **L2543 EN**: Returns from the current function with `"llvm.ppc.mma.xvbf16ger2pp"`.
  **L2543 CN**: 以 `"llvm.ppc.mma.xvbf16ger2pp"` 从当前函数返回。
- **L2544 EN**: Introduces a switch dispatch label: `case MMAOp::Xvf16ger2:`.
  **L2544 CN**: 引入一个 switch 分发标签：`case MMAOp::Xvf16ger2:`。

### Lines 2545-2568

````cpp
    return "llvm.ppc.mma.xvf16ger2";
  case MMAOp::Xvf16ger2nn:
    return "llvm.ppc.mma.xvf16ger2nn";
  case MMAOp::Xvf16ger2np:
    return "llvm.ppc.mma.xvf16ger2np";
  case MMAOp::Xvf16ger2pn:
    return "llvm.ppc.mma.xvf16ger2pn";
  case MMAOp::Xvf16ger2pp:
    return "llvm.ppc.mma.xvf16ger2pp";
  case MMAOp::Xvf32ger:
    return "llvm.ppc.mma.xvf32ger";
  case MMAOp::Xvf32gernn:
    return "llvm.ppc.mma.xvf32gernn";
  case MMAOp::Xvf32gernp:
    return "llvm.ppc.mma.xvf32gernp";
  case MMAOp::Xvf32gerpn:
    return "llvm.ppc.mma.xvf32gerpn";
  case MMAOp::Xvf32gerpp:
    return "llvm.ppc.mma.xvf32gerpp";
  case MMAOp::Xvf64ger:
    return "llvm.ppc.mma.xvf64ger";
  case MMAOp::Xvf64gernn:
    return "llvm.ppc.mma.xvf64gernn";
  case MMAOp::Xvf64gernp:
````
- **L2545 EN**: Returns from the current function with `"llvm.ppc.mma.xvf16ger2"`.
  **L2545 CN**: 以 `"llvm.ppc.mma.xvf16ger2"` 从当前函数返回。
- **L2546 EN**: Introduces a switch dispatch label: `case MMAOp::Xvf16ger2nn:`.
  **L2546 CN**: 引入一个 switch 分发标签：`case MMAOp::Xvf16ger2nn:`。
- **L2547 EN**: Returns from the current function with `"llvm.ppc.mma.xvf16ger2nn"`.
  **L2547 CN**: 以 `"llvm.ppc.mma.xvf16ger2nn"` 从当前函数返回。
- **L2548 EN**: Introduces a switch dispatch label: `case MMAOp::Xvf16ger2np:`.
  **L2548 CN**: 引入一个 switch 分发标签：`case MMAOp::Xvf16ger2np:`。
- **L2549 EN**: Returns from the current function with `"llvm.ppc.mma.xvf16ger2np"`.
  **L2549 CN**: 以 `"llvm.ppc.mma.xvf16ger2np"` 从当前函数返回。
- **L2550 EN**: Introduces a switch dispatch label: `case MMAOp::Xvf16ger2pn:`.
  **L2550 CN**: 引入一个 switch 分发标签：`case MMAOp::Xvf16ger2pn:`。
- **L2551 EN**: Returns from the current function with `"llvm.ppc.mma.xvf16ger2pn"`.
  **L2551 CN**: 以 `"llvm.ppc.mma.xvf16ger2pn"` 从当前函数返回。
- **L2552 EN**: Introduces a switch dispatch label: `case MMAOp::Xvf16ger2pp:`.
  **L2552 CN**: 引入一个 switch 分发标签：`case MMAOp::Xvf16ger2pp:`。
- **L2553 EN**: Returns from the current function with `"llvm.ppc.mma.xvf16ger2pp"`.
  **L2553 CN**: 以 `"llvm.ppc.mma.xvf16ger2pp"` 从当前函数返回。
- **L2554 EN**: Introduces a switch dispatch label: `case MMAOp::Xvf32ger:`.
  **L2554 CN**: 引入一个 switch 分发标签：`case MMAOp::Xvf32ger:`。
- **L2555 EN**: Returns from the current function with `"llvm.ppc.mma.xvf32ger"`.
  **L2555 CN**: 以 `"llvm.ppc.mma.xvf32ger"` 从当前函数返回。
- **L2556 EN**: Introduces a switch dispatch label: `case MMAOp::Xvf32gernn:`.
  **L2556 CN**: 引入一个 switch 分发标签：`case MMAOp::Xvf32gernn:`。
- **L2557 EN**: Returns from the current function with `"llvm.ppc.mma.xvf32gernn"`.
  **L2557 CN**: 以 `"llvm.ppc.mma.xvf32gernn"` 从当前函数返回。
- **L2558 EN**: Introduces a switch dispatch label: `case MMAOp::Xvf32gernp:`.
  **L2558 CN**: 引入一个 switch 分发标签：`case MMAOp::Xvf32gernp:`。
- **L2559 EN**: Returns from the current function with `"llvm.ppc.mma.xvf32gernp"`.
  **L2559 CN**: 以 `"llvm.ppc.mma.xvf32gernp"` 从当前函数返回。
- **L2560 EN**: Introduces a switch dispatch label: `case MMAOp::Xvf32gerpn:`.
  **L2560 CN**: 引入一个 switch 分发标签：`case MMAOp::Xvf32gerpn:`。
- **L2561 EN**: Returns from the current function with `"llvm.ppc.mma.xvf32gerpn"`.
  **L2561 CN**: 以 `"llvm.ppc.mma.xvf32gerpn"` 从当前函数返回。
- **L2562 EN**: Introduces a switch dispatch label: `case MMAOp::Xvf32gerpp:`.
  **L2562 CN**: 引入一个 switch 分发标签：`case MMAOp::Xvf32gerpp:`。
- **L2563 EN**: Returns from the current function with `"llvm.ppc.mma.xvf32gerpp"`.
  **L2563 CN**: 以 `"llvm.ppc.mma.xvf32gerpp"` 从当前函数返回。
- **L2564 EN**: Introduces a switch dispatch label: `case MMAOp::Xvf64ger:`.
  **L2564 CN**: 引入一个 switch 分发标签：`case MMAOp::Xvf64ger:`。
- **L2565 EN**: Returns from the current function with `"llvm.ppc.mma.xvf64ger"`.
  **L2565 CN**: 以 `"llvm.ppc.mma.xvf64ger"` 从当前函数返回。
- **L2566 EN**: Introduces a switch dispatch label: `case MMAOp::Xvf64gernn:`.
  **L2566 CN**: 引入一个 switch 分发标签：`case MMAOp::Xvf64gernn:`。
- **L2567 EN**: Returns from the current function with `"llvm.ppc.mma.xvf64gernn"`.
  **L2567 CN**: 以 `"llvm.ppc.mma.xvf64gernn"` 从当前函数返回。
- **L2568 EN**: Introduces a switch dispatch label: `case MMAOp::Xvf64gernp:`.
  **L2568 CN**: 引入一个 switch 分发标签：`case MMAOp::Xvf64gernp:`。

### Lines 2569-2592

````cpp
    return "llvm.ppc.mma.xvf64gernp";
  case MMAOp::Xvf64gerpn:
    return "llvm.ppc.mma.xvf64gerpn";
  case MMAOp::Xvf64gerpp:
    return "llvm.ppc.mma.xvf64gerpp";
  case MMAOp::Xvi16ger2:
    return "llvm.ppc.mma.xvi16ger2";
  case MMAOp::Xvi16ger2pp:
    return "llvm.ppc.mma.xvi16ger2pp";
  case MMAOp::Xvi16ger2s:
    return "llvm.ppc.mma.xvi16ger2s";
  case MMAOp::Xvi16ger2spp:
    return "llvm.ppc.mma.xvi16ger2spp";
  case MMAOp::Xvi4ger8:
    return "llvm.ppc.mma.xvi4ger8";
  case MMAOp::Xvi4ger8pp:
    return "llvm.ppc.mma.xvi4ger8pp";
  case MMAOp::Xvi8ger4:
    return "llvm.ppc.mma.xvi8ger4";
  case MMAOp::Xvi8ger4pp:
    return "llvm.ppc.mma.xvi8ger4pp";
  case MMAOp::Xvi8ger4spp:
    return "llvm.ppc.mma.xvi8ger4spp";
  }
````
- **L2569 EN**: Returns from the current function with `"llvm.ppc.mma.xvf64gernp"`.
  **L2569 CN**: 以 `"llvm.ppc.mma.xvf64gernp"` 从当前函数返回。
- **L2570 EN**: Introduces a switch dispatch label: `case MMAOp::Xvf64gerpn:`.
  **L2570 CN**: 引入一个 switch 分发标签：`case MMAOp::Xvf64gerpn:`。
- **L2571 EN**: Returns from the current function with `"llvm.ppc.mma.xvf64gerpn"`.
  **L2571 CN**: 以 `"llvm.ppc.mma.xvf64gerpn"` 从当前函数返回。
- **L2572 EN**: Introduces a switch dispatch label: `case MMAOp::Xvf64gerpp:`.
  **L2572 CN**: 引入一个 switch 分发标签：`case MMAOp::Xvf64gerpp:`。
- **L2573 EN**: Returns from the current function with `"llvm.ppc.mma.xvf64gerpp"`.
  **L2573 CN**: 以 `"llvm.ppc.mma.xvf64gerpp"` 从当前函数返回。
- **L2574 EN**: Introduces a switch dispatch label: `case MMAOp::Xvi16ger2:`.
  **L2574 CN**: 引入一个 switch 分发标签：`case MMAOp::Xvi16ger2:`。
- **L2575 EN**: Returns from the current function with `"llvm.ppc.mma.xvi16ger2"`.
  **L2575 CN**: 以 `"llvm.ppc.mma.xvi16ger2"` 从当前函数返回。
- **L2576 EN**: Introduces a switch dispatch label: `case MMAOp::Xvi16ger2pp:`.
  **L2576 CN**: 引入一个 switch 分发标签：`case MMAOp::Xvi16ger2pp:`。
- **L2577 EN**: Returns from the current function with `"llvm.ppc.mma.xvi16ger2pp"`.
  **L2577 CN**: 以 `"llvm.ppc.mma.xvi16ger2pp"` 从当前函数返回。
- **L2578 EN**: Introduces a switch dispatch label: `case MMAOp::Xvi16ger2s:`.
  **L2578 CN**: 引入一个 switch 分发标签：`case MMAOp::Xvi16ger2s:`。
- **L2579 EN**: Returns from the current function with `"llvm.ppc.mma.xvi16ger2s"`.
  **L2579 CN**: 以 `"llvm.ppc.mma.xvi16ger2s"` 从当前函数返回。
- **L2580 EN**: Introduces a switch dispatch label: `case MMAOp::Xvi16ger2spp:`.
  **L2580 CN**: 引入一个 switch 分发标签：`case MMAOp::Xvi16ger2spp:`。
- **L2581 EN**: Returns from the current function with `"llvm.ppc.mma.xvi16ger2spp"`.
  **L2581 CN**: 以 `"llvm.ppc.mma.xvi16ger2spp"` 从当前函数返回。
- **L2582 EN**: Introduces a switch dispatch label: `case MMAOp::Xvi4ger8:`.
  **L2582 CN**: 引入一个 switch 分发标签：`case MMAOp::Xvi4ger8:`。
- **L2583 EN**: Returns from the current function with `"llvm.ppc.mma.xvi4ger8"`.
  **L2583 CN**: 以 `"llvm.ppc.mma.xvi4ger8"` 从当前函数返回。
- **L2584 EN**: Introduces a switch dispatch label: `case MMAOp::Xvi4ger8pp:`.
  **L2584 CN**: 引入一个 switch 分发标签：`case MMAOp::Xvi4ger8pp:`。
- **L2585 EN**: Returns from the current function with `"llvm.ppc.mma.xvi4ger8pp"`.
  **L2585 CN**: 以 `"llvm.ppc.mma.xvi4ger8pp"` 从当前函数返回。
- **L2586 EN**: Introduces a switch dispatch label: `case MMAOp::Xvi8ger4:`.
  **L2586 CN**: 引入一个 switch 分发标签：`case MMAOp::Xvi8ger4:`。
- **L2587 EN**: Returns from the current function with `"llvm.ppc.mma.xvi8ger4"`.
  **L2587 CN**: 以 `"llvm.ppc.mma.xvi8ger4"` 从当前函数返回。
- **L2588 EN**: Introduces a switch dispatch label: `case MMAOp::Xvi8ger4pp:`.
  **L2588 CN**: 引入一个 switch 分发标签：`case MMAOp::Xvi8ger4pp:`。
- **L2589 EN**: Returns from the current function with `"llvm.ppc.mma.xvi8ger4pp"`.
  **L2589 CN**: 以 `"llvm.ppc.mma.xvi8ger4pp"` 从当前函数返回。
- **L2590 EN**: Introduces a switch dispatch label: `case MMAOp::Xvi8ger4spp:`.
  **L2590 CN**: 引入一个 switch 分发标签：`case MMAOp::Xvi8ger4spp:`。
- **L2591 EN**: Returns from the current function with `"llvm.ppc.mma.xvi8ger4spp"`.
  **L2591 CN**: 以 `"llvm.ppc.mma.xvi8ger4spp"` 从当前函数返回。
- **L2592 EN**: Closes the current lexical scope or compound statement.
  **L2592 CN**: 结束当前词法作用域或复合语句块。

### Lines 2593-2616

````cpp
  llvm_unreachable("getMmaIrIntrName");
}

mlir::FunctionType getMmaIrFuncType(mlir::MLIRContext *context, MMAOp mmaOp) {
  switch (mmaOp) {
  case MMAOp::AssembleAcc:
    return genMmaVqFuncType(context, /*Quad*/ 0, /*Pair*/ 0, /*Vector*/ 4);
  case MMAOp::AssemblePair:
    return genMmaVpFuncType(context, /*Quad*/ 0, /*Pair*/ 0, /*Vector*/ 2);
  case MMAOp::DisassembleAcc:
    return genMmaDisassembleFuncType(context, mmaOp);
  case MMAOp::DisassemblePair:
    return genMmaDisassembleFuncType(context, mmaOp);
  case MMAOp::Xxmfacc:
    return genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 0);
  case MMAOp::Xxmtacc:
    return genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 0);
  case MMAOp::Xxsetaccz:
    return genMmaVqFuncType(context, /*Quad*/ 0, /*Pair*/ 0, /*Vector*/ 0);
  case MMAOp::Pmxvbf16ger2:
    return genMmaVqFuncType(context, /*Quad*/ 0, /*Pair*/ 0, /*Vector*/ 2,
                            /*Integer*/ 3);
  case MMAOp::Pmxvbf16ger2nn:
    return genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2,
````
- **L2593 EN**: Marks this control path as unreachable to LLVM.
  **L2593 CN**: 将该控制路径标记为 LLVM 认为不可达。
- **L2594 EN**: Closes the current lexical scope or compound statement.
  **L2594 CN**: 结束当前词法作用域或复合语句块。
- **L2595 EN**: Blank line separating nearby declarations or logic blocks.
  **L2595 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2596 EN**: Starts a function, method, lambda, or structured scope: `mlir::FunctionType getMmaIrFuncType(mlir::MLIRContext *context, MMAOp mmaOp) {`.
  **L2596 CN**: 开始一个函数、方法、lambda 或结构化作用域：`mlir::FunctionType getMmaIrFuncType(mlir::MLIRContext *context, MMAOp mmaOp) {`。
- **L2597 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L2597 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L2598 EN**: Introduces a switch dispatch label: `case MMAOp::AssembleAcc:`.
  **L2598 CN**: 引入一个 switch 分发标签：`case MMAOp::AssembleAcc:`。
- **L2599 EN**: Returns from the current function with `genMmaVqFuncType(context, /*Quad*/ 0, /*Pair*/ 0, /*Vector*/ 4)`.
  **L2599 CN**: 以 `genMmaVqFuncType(context, /*Quad*/ 0, /*Pair*/ 0, /*Vector*/ 4)` 从当前函数返回。
- **L2600 EN**: Introduces a switch dispatch label: `case MMAOp::AssemblePair:`.
  **L2600 CN**: 引入一个 switch 分发标签：`case MMAOp::AssemblePair:`。
- **L2601 EN**: Returns from the current function with `genMmaVpFuncType(context, /*Quad*/ 0, /*Pair*/ 0, /*Vector*/ 2)`.
  **L2601 CN**: 以 `genMmaVpFuncType(context, /*Quad*/ 0, /*Pair*/ 0, /*Vector*/ 2)` 从当前函数返回。
- **L2602 EN**: Introduces a switch dispatch label: `case MMAOp::DisassembleAcc:`.
  **L2602 CN**: 引入一个 switch 分发标签：`case MMAOp::DisassembleAcc:`。
- **L2603 EN**: Returns from the current function with `genMmaDisassembleFuncType(context, mmaOp)`.
  **L2603 CN**: 以 `genMmaDisassembleFuncType(context, mmaOp)` 从当前函数返回。
- **L2604 EN**: Introduces a switch dispatch label: `case MMAOp::DisassemblePair:`.
  **L2604 CN**: 引入一个 switch 分发标签：`case MMAOp::DisassemblePair:`。
- **L2605 EN**: Returns from the current function with `genMmaDisassembleFuncType(context, mmaOp)`.
  **L2605 CN**: 以 `genMmaDisassembleFuncType(context, mmaOp)` 从当前函数返回。
- **L2606 EN**: Introduces a switch dispatch label: `case MMAOp::Xxmfacc:`.
  **L2606 CN**: 引入一个 switch 分发标签：`case MMAOp::Xxmfacc:`。
- **L2607 EN**: Returns from the current function with `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 0)`.
  **L2607 CN**: 以 `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 0)` 从当前函数返回。
- **L2608 EN**: Introduces a switch dispatch label: `case MMAOp::Xxmtacc:`.
  **L2608 CN**: 引入一个 switch 分发标签：`case MMAOp::Xxmtacc:`。
- **L2609 EN**: Returns from the current function with `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 0)`.
  **L2609 CN**: 以 `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 0)` 从当前函数返回。
- **L2610 EN**: Introduces a switch dispatch label: `case MMAOp::Xxsetaccz:`.
  **L2610 CN**: 引入一个 switch 分发标签：`case MMAOp::Xxsetaccz:`。
- **L2611 EN**: Returns from the current function with `genMmaVqFuncType(context, /*Quad*/ 0, /*Pair*/ 0, /*Vector*/ 0)`.
  **L2611 CN**: 以 `genMmaVqFuncType(context, /*Quad*/ 0, /*Pair*/ 0, /*Vector*/ 0)` 从当前函数返回。
- **L2612 EN**: Introduces a switch dispatch label: `case MMAOp::Pmxvbf16ger2:`.
  **L2612 CN**: 引入一个 switch 分发标签：`case MMAOp::Pmxvbf16ger2:`。
- **L2613 EN**: Returns from the current function with `genMmaVqFuncType(context, /*Quad*/ 0, /*Pair*/ 0, /*Vector*/ 2,`.
  **L2613 CN**: 以 `genMmaVqFuncType(context, /*Quad*/ 0, /*Pair*/ 0, /*Vector*/ 2,` 从当前函数返回。
- **L2614 EN**: Comment explains nearby logic, intent, or metadata: `Integer*/ 3);`.
  **L2614 CN**: 注释说明附近代码的逻辑、意图或元数据：`Integer*/ 3);`。
- **L2615 EN**: Introduces a switch dispatch label: `case MMAOp::Pmxvbf16ger2nn:`.
  **L2615 CN**: 引入一个 switch 分发标签：`case MMAOp::Pmxvbf16ger2nn:`。
- **L2616 EN**: Returns from the current function with `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2,`.
  **L2616 CN**: 以 `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2,` 从当前函数返回。

### Lines 2617-2640

````cpp
                            /*Integer*/ 3);
  case MMAOp::Pmxvbf16ger2np:
    return genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2,
                            /*Integer*/ 3);
  case MMAOp::Pmxvbf16ger2pn:
    return genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2,
                            /*Integer*/ 3);
  case MMAOp::Pmxvbf16ger2pp:
    return genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2,
                            /*Integer*/ 3);
  case MMAOp::Pmxvf16ger2:
    return genMmaVqFuncType(context, /*Quad*/ 0, /*Pair*/ 0, /*Vector*/ 2,
                            /*Integer*/ 3);
  case MMAOp::Pmxvf16ger2nn:
    return genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2,
                            /*Integer*/ 3);
  case MMAOp::Pmxvf16ger2np:
    return genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2,
                            /*Integer*/ 3);
  case MMAOp::Pmxvf16ger2pn:
    return genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2,
                            /*Integer*/ 3);
  case MMAOp::Pmxvf16ger2pp:
    return genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2,
````
- **L2617 EN**: Comment explains nearby logic, intent, or metadata: `Integer*/ 3);`.
  **L2617 CN**: 注释说明附近代码的逻辑、意图或元数据：`Integer*/ 3);`。
- **L2618 EN**: Introduces a switch dispatch label: `case MMAOp::Pmxvbf16ger2np:`.
  **L2618 CN**: 引入一个 switch 分发标签：`case MMAOp::Pmxvbf16ger2np:`。
- **L2619 EN**: Returns from the current function with `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2,`.
  **L2619 CN**: 以 `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2,` 从当前函数返回。
- **L2620 EN**: Comment explains nearby logic, intent, or metadata: `Integer*/ 3);`.
  **L2620 CN**: 注释说明附近代码的逻辑、意图或元数据：`Integer*/ 3);`。
- **L2621 EN**: Introduces a switch dispatch label: `case MMAOp::Pmxvbf16ger2pn:`.
  **L2621 CN**: 引入一个 switch 分发标签：`case MMAOp::Pmxvbf16ger2pn:`。
- **L2622 EN**: Returns from the current function with `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2,`.
  **L2622 CN**: 以 `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2,` 从当前函数返回。
- **L2623 EN**: Comment explains nearby logic, intent, or metadata: `Integer*/ 3);`.
  **L2623 CN**: 注释说明附近代码的逻辑、意图或元数据：`Integer*/ 3);`。
- **L2624 EN**: Introduces a switch dispatch label: `case MMAOp::Pmxvbf16ger2pp:`.
  **L2624 CN**: 引入一个 switch 分发标签：`case MMAOp::Pmxvbf16ger2pp:`。
- **L2625 EN**: Returns from the current function with `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2,`.
  **L2625 CN**: 以 `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2,` 从当前函数返回。
- **L2626 EN**: Comment explains nearby logic, intent, or metadata: `Integer*/ 3);`.
  **L2626 CN**: 注释说明附近代码的逻辑、意图或元数据：`Integer*/ 3);`。
- **L2627 EN**: Introduces a switch dispatch label: `case MMAOp::Pmxvf16ger2:`.
  **L2627 CN**: 引入一个 switch 分发标签：`case MMAOp::Pmxvf16ger2:`。
- **L2628 EN**: Returns from the current function with `genMmaVqFuncType(context, /*Quad*/ 0, /*Pair*/ 0, /*Vector*/ 2,`.
  **L2628 CN**: 以 `genMmaVqFuncType(context, /*Quad*/ 0, /*Pair*/ 0, /*Vector*/ 2,` 从当前函数返回。
- **L2629 EN**: Comment explains nearby logic, intent, or metadata: `Integer*/ 3);`.
  **L2629 CN**: 注释说明附近代码的逻辑、意图或元数据：`Integer*/ 3);`。
- **L2630 EN**: Introduces a switch dispatch label: `case MMAOp::Pmxvf16ger2nn:`.
  **L2630 CN**: 引入一个 switch 分发标签：`case MMAOp::Pmxvf16ger2nn:`。
- **L2631 EN**: Returns from the current function with `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2,`.
  **L2631 CN**: 以 `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2,` 从当前函数返回。
- **L2632 EN**: Comment explains nearby logic, intent, or metadata: `Integer*/ 3);`.
  **L2632 CN**: 注释说明附近代码的逻辑、意图或元数据：`Integer*/ 3);`。
- **L2633 EN**: Introduces a switch dispatch label: `case MMAOp::Pmxvf16ger2np:`.
  **L2633 CN**: 引入一个 switch 分发标签：`case MMAOp::Pmxvf16ger2np:`。
- **L2634 EN**: Returns from the current function with `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2,`.
  **L2634 CN**: 以 `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2,` 从当前函数返回。
- **L2635 EN**: Comment explains nearby logic, intent, or metadata: `Integer*/ 3);`.
  **L2635 CN**: 注释说明附近代码的逻辑、意图或元数据：`Integer*/ 3);`。
- **L2636 EN**: Introduces a switch dispatch label: `case MMAOp::Pmxvf16ger2pn:`.
  **L2636 CN**: 引入一个 switch 分发标签：`case MMAOp::Pmxvf16ger2pn:`。
- **L2637 EN**: Returns from the current function with `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2,`.
  **L2637 CN**: 以 `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2,` 从当前函数返回。
- **L2638 EN**: Comment explains nearby logic, intent, or metadata: `Integer*/ 3);`.
  **L2638 CN**: 注释说明附近代码的逻辑、意图或元数据：`Integer*/ 3);`。
- **L2639 EN**: Introduces a switch dispatch label: `case MMAOp::Pmxvf16ger2pp:`.
  **L2639 CN**: 引入一个 switch 分发标签：`case MMAOp::Pmxvf16ger2pp:`。
- **L2640 EN**: Returns from the current function with `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2,`.
  **L2640 CN**: 以 `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2,` 从当前函数返回。

### Lines 2641-2664

````cpp
                            /*Integer*/ 3);
  case MMAOp::Pmxvf32ger:
    return genMmaVqFuncType(context, /*Quad*/ 0, /*Pair*/ 0, /*Vector*/ 2,
                            /*Integer*/ 2);
  case MMAOp::Pmxvf32gernn:
    return genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2,
                            /*Integer*/ 2);
  case MMAOp::Pmxvf32gernp:
    return genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2,
                            /*Integer*/ 2);
  case MMAOp::Pmxvf32gerpn:
    return genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2,
                            /*Integer*/ 2);
  case MMAOp::Pmxvf32gerpp:
    return genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2,
                            /*Integer*/ 2);
  case MMAOp::Pmxvf64ger:
    return genMmaVqFuncType(context, /*Quad*/ 0, /*Pair*/ 1, /*Vector*/ 1,
                            /*Integer*/ 2);
  case MMAOp::Pmxvf64gernn:
    return genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 1, /*Vector*/ 1,
                            /*Integer*/ 2);
  case MMAOp::Pmxvf64gernp:
    return genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 1, /*Vector*/ 1,
````
- **L2641 EN**: Comment explains nearby logic, intent, or metadata: `Integer*/ 3);`.
  **L2641 CN**: 注释说明附近代码的逻辑、意图或元数据：`Integer*/ 3);`。
- **L2642 EN**: Introduces a switch dispatch label: `case MMAOp::Pmxvf32ger:`.
  **L2642 CN**: 引入一个 switch 分发标签：`case MMAOp::Pmxvf32ger:`。
- **L2643 EN**: Returns from the current function with `genMmaVqFuncType(context, /*Quad*/ 0, /*Pair*/ 0, /*Vector*/ 2,`.
  **L2643 CN**: 以 `genMmaVqFuncType(context, /*Quad*/ 0, /*Pair*/ 0, /*Vector*/ 2,` 从当前函数返回。
- **L2644 EN**: Comment explains nearby logic, intent, or metadata: `Integer*/ 2);`.
  **L2644 CN**: 注释说明附近代码的逻辑、意图或元数据：`Integer*/ 2);`。
- **L2645 EN**: Introduces a switch dispatch label: `case MMAOp::Pmxvf32gernn:`.
  **L2645 CN**: 引入一个 switch 分发标签：`case MMAOp::Pmxvf32gernn:`。
- **L2646 EN**: Returns from the current function with `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2,`.
  **L2646 CN**: 以 `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2,` 从当前函数返回。
- **L2647 EN**: Comment explains nearby logic, intent, or metadata: `Integer*/ 2);`.
  **L2647 CN**: 注释说明附近代码的逻辑、意图或元数据：`Integer*/ 2);`。
- **L2648 EN**: Introduces a switch dispatch label: `case MMAOp::Pmxvf32gernp:`.
  **L2648 CN**: 引入一个 switch 分发标签：`case MMAOp::Pmxvf32gernp:`。
- **L2649 EN**: Returns from the current function with `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2,`.
  **L2649 CN**: 以 `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2,` 从当前函数返回。
- **L2650 EN**: Comment explains nearby logic, intent, or metadata: `Integer*/ 2);`.
  **L2650 CN**: 注释说明附近代码的逻辑、意图或元数据：`Integer*/ 2);`。
- **L2651 EN**: Introduces a switch dispatch label: `case MMAOp::Pmxvf32gerpn:`.
  **L2651 CN**: 引入一个 switch 分发标签：`case MMAOp::Pmxvf32gerpn:`。
- **L2652 EN**: Returns from the current function with `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2,`.
  **L2652 CN**: 以 `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2,` 从当前函数返回。
- **L2653 EN**: Comment explains nearby logic, intent, or metadata: `Integer*/ 2);`.
  **L2653 CN**: 注释说明附近代码的逻辑、意图或元数据：`Integer*/ 2);`。
- **L2654 EN**: Introduces a switch dispatch label: `case MMAOp::Pmxvf32gerpp:`.
  **L2654 CN**: 引入一个 switch 分发标签：`case MMAOp::Pmxvf32gerpp:`。
- **L2655 EN**: Returns from the current function with `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2,`.
  **L2655 CN**: 以 `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2,` 从当前函数返回。
- **L2656 EN**: Comment explains nearby logic, intent, or metadata: `Integer*/ 2);`.
  **L2656 CN**: 注释说明附近代码的逻辑、意图或元数据：`Integer*/ 2);`。
- **L2657 EN**: Introduces a switch dispatch label: `case MMAOp::Pmxvf64ger:`.
  **L2657 CN**: 引入一个 switch 分发标签：`case MMAOp::Pmxvf64ger:`。
- **L2658 EN**: Returns from the current function with `genMmaVqFuncType(context, /*Quad*/ 0, /*Pair*/ 1, /*Vector*/ 1,`.
  **L2658 CN**: 以 `genMmaVqFuncType(context, /*Quad*/ 0, /*Pair*/ 1, /*Vector*/ 1,` 从当前函数返回。
- **L2659 EN**: Comment explains nearby logic, intent, or metadata: `Integer*/ 2);`.
  **L2659 CN**: 注释说明附近代码的逻辑、意图或元数据：`Integer*/ 2);`。
- **L2660 EN**: Introduces a switch dispatch label: `case MMAOp::Pmxvf64gernn:`.
  **L2660 CN**: 引入一个 switch 分发标签：`case MMAOp::Pmxvf64gernn:`。
- **L2661 EN**: Returns from the current function with `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 1, /*Vector*/ 1,`.
  **L2661 CN**: 以 `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 1, /*Vector*/ 1,` 从当前函数返回。
- **L2662 EN**: Comment explains nearby logic, intent, or metadata: `Integer*/ 2);`.
  **L2662 CN**: 注释说明附近代码的逻辑、意图或元数据：`Integer*/ 2);`。
- **L2663 EN**: Introduces a switch dispatch label: `case MMAOp::Pmxvf64gernp:`.
  **L2663 CN**: 引入一个 switch 分发标签：`case MMAOp::Pmxvf64gernp:`。
- **L2664 EN**: Returns from the current function with `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 1, /*Vector*/ 1,`.
  **L2664 CN**: 以 `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 1, /*Vector*/ 1,` 从当前函数返回。

### Lines 2665-2688

````cpp
                            /*Integer*/ 2);
  case MMAOp::Pmxvf64gerpn:
    return genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 1, /*Vector*/ 1,
                            /*Integer*/ 2);
  case MMAOp::Pmxvf64gerpp:
    return genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 1, /*Vector*/ 1,
                            /*Integer*/ 2);
  case MMAOp::Pmxvi16ger2:
    return genMmaVqFuncType(context, /*Quad*/ 0, /*Pair*/ 0, /*Vector*/ 2,
                            /*Integer*/ 3);
  case MMAOp::Pmxvi16ger2pp:
    return genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2,
                            /*Integer*/ 3);
  case MMAOp::Pmxvi16ger2s:
    return genMmaVqFuncType(context, /*Quad*/ 0, /*Pair*/ 0, /*Vector*/ 2,
                            /*Integer*/ 3);
  case MMAOp::Pmxvi16ger2spp:
    return genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2,
                            /*Integer*/ 3);
  case MMAOp::Pmxvi4ger8:
    return genMmaVqFuncType(context, /*Quad*/ 0, /*Pair*/ 0, /*Vector*/ 2,
                            /*Integer*/ 3);
  case MMAOp::Pmxvi4ger8pp:
    return genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2,
````
- **L2665 EN**: Comment explains nearby logic, intent, or metadata: `Integer*/ 2);`.
  **L2665 CN**: 注释说明附近代码的逻辑、意图或元数据：`Integer*/ 2);`。
- **L2666 EN**: Introduces a switch dispatch label: `case MMAOp::Pmxvf64gerpn:`.
  **L2666 CN**: 引入一个 switch 分发标签：`case MMAOp::Pmxvf64gerpn:`。
- **L2667 EN**: Returns from the current function with `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 1, /*Vector*/ 1,`.
  **L2667 CN**: 以 `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 1, /*Vector*/ 1,` 从当前函数返回。
- **L2668 EN**: Comment explains nearby logic, intent, or metadata: `Integer*/ 2);`.
  **L2668 CN**: 注释说明附近代码的逻辑、意图或元数据：`Integer*/ 2);`。
- **L2669 EN**: Introduces a switch dispatch label: `case MMAOp::Pmxvf64gerpp:`.
  **L2669 CN**: 引入一个 switch 分发标签：`case MMAOp::Pmxvf64gerpp:`。
- **L2670 EN**: Returns from the current function with `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 1, /*Vector*/ 1,`.
  **L2670 CN**: 以 `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 1, /*Vector*/ 1,` 从当前函数返回。
- **L2671 EN**: Comment explains nearby logic, intent, or metadata: `Integer*/ 2);`.
  **L2671 CN**: 注释说明附近代码的逻辑、意图或元数据：`Integer*/ 2);`。
- **L2672 EN**: Introduces a switch dispatch label: `case MMAOp::Pmxvi16ger2:`.
  **L2672 CN**: 引入一个 switch 分发标签：`case MMAOp::Pmxvi16ger2:`。
- **L2673 EN**: Returns from the current function with `genMmaVqFuncType(context, /*Quad*/ 0, /*Pair*/ 0, /*Vector*/ 2,`.
  **L2673 CN**: 以 `genMmaVqFuncType(context, /*Quad*/ 0, /*Pair*/ 0, /*Vector*/ 2,` 从当前函数返回。
- **L2674 EN**: Comment explains nearby logic, intent, or metadata: `Integer*/ 3);`.
  **L2674 CN**: 注释说明附近代码的逻辑、意图或元数据：`Integer*/ 3);`。
- **L2675 EN**: Introduces a switch dispatch label: `case MMAOp::Pmxvi16ger2pp:`.
  **L2675 CN**: 引入一个 switch 分发标签：`case MMAOp::Pmxvi16ger2pp:`。
- **L2676 EN**: Returns from the current function with `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2,`.
  **L2676 CN**: 以 `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2,` 从当前函数返回。
- **L2677 EN**: Comment explains nearby logic, intent, or metadata: `Integer*/ 3);`.
  **L2677 CN**: 注释说明附近代码的逻辑、意图或元数据：`Integer*/ 3);`。
- **L2678 EN**: Introduces a switch dispatch label: `case MMAOp::Pmxvi16ger2s:`.
  **L2678 CN**: 引入一个 switch 分发标签：`case MMAOp::Pmxvi16ger2s:`。
- **L2679 EN**: Returns from the current function with `genMmaVqFuncType(context, /*Quad*/ 0, /*Pair*/ 0, /*Vector*/ 2,`.
  **L2679 CN**: 以 `genMmaVqFuncType(context, /*Quad*/ 0, /*Pair*/ 0, /*Vector*/ 2,` 从当前函数返回。
- **L2680 EN**: Comment explains nearby logic, intent, or metadata: `Integer*/ 3);`.
  **L2680 CN**: 注释说明附近代码的逻辑、意图或元数据：`Integer*/ 3);`。
- **L2681 EN**: Introduces a switch dispatch label: `case MMAOp::Pmxvi16ger2spp:`.
  **L2681 CN**: 引入一个 switch 分发标签：`case MMAOp::Pmxvi16ger2spp:`。
- **L2682 EN**: Returns from the current function with `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2,`.
  **L2682 CN**: 以 `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2,` 从当前函数返回。
- **L2683 EN**: Comment explains nearby logic, intent, or metadata: `Integer*/ 3);`.
  **L2683 CN**: 注释说明附近代码的逻辑、意图或元数据：`Integer*/ 3);`。
- **L2684 EN**: Introduces a switch dispatch label: `case MMAOp::Pmxvi4ger8:`.
  **L2684 CN**: 引入一个 switch 分发标签：`case MMAOp::Pmxvi4ger8:`。
- **L2685 EN**: Returns from the current function with `genMmaVqFuncType(context, /*Quad*/ 0, /*Pair*/ 0, /*Vector*/ 2,`.
  **L2685 CN**: 以 `genMmaVqFuncType(context, /*Quad*/ 0, /*Pair*/ 0, /*Vector*/ 2,` 从当前函数返回。
- **L2686 EN**: Comment explains nearby logic, intent, or metadata: `Integer*/ 3);`.
  **L2686 CN**: 注释说明附近代码的逻辑、意图或元数据：`Integer*/ 3);`。
- **L2687 EN**: Introduces a switch dispatch label: `case MMAOp::Pmxvi4ger8pp:`.
  **L2687 CN**: 引入一个 switch 分发标签：`case MMAOp::Pmxvi4ger8pp:`。
- **L2688 EN**: Returns from the current function with `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2,`.
  **L2688 CN**: 以 `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2,` 从当前函数返回。

### Lines 2689-2712

````cpp
                            /*Integer*/ 3);
  case MMAOp::Pmxvi8ger4:
    return genMmaVqFuncType(context, /*Quad*/ 0, /*Pair*/ 0, /*Vector*/ 2,
                            /*Integer*/ 3);
  case MMAOp::Pmxvi8ger4pp:
    return genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2,
                            /*Integer*/ 3);
  case MMAOp::Pmxvi8ger4spp:
    return genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2,
                            /*Integer*/ 3);
  case MMAOp::Xvbf16ger2:
    return genMmaVqFuncType(context, /*Quad*/ 0, /*Pair*/ 0, /*Vector*/ 2);
  case MMAOp::Xvbf16ger2nn:
    return genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2);
  case MMAOp::Xvbf16ger2np:
    return genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2);
  case MMAOp::Xvbf16ger2pn:
    return genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2);
  case MMAOp::Xvbf16ger2pp:
    return genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2);
  case MMAOp::Xvf16ger2:
    return genMmaVqFuncType(context, /*Quad*/ 0, /*Pair*/ 0, /*Vector*/ 2);
  case MMAOp::Xvf16ger2nn:
    return genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2);
````
- **L2689 EN**: Comment explains nearby logic, intent, or metadata: `Integer*/ 3);`.
  **L2689 CN**: 注释说明附近代码的逻辑、意图或元数据：`Integer*/ 3);`。
- **L2690 EN**: Introduces a switch dispatch label: `case MMAOp::Pmxvi8ger4:`.
  **L2690 CN**: 引入一个 switch 分发标签：`case MMAOp::Pmxvi8ger4:`。
- **L2691 EN**: Returns from the current function with `genMmaVqFuncType(context, /*Quad*/ 0, /*Pair*/ 0, /*Vector*/ 2,`.
  **L2691 CN**: 以 `genMmaVqFuncType(context, /*Quad*/ 0, /*Pair*/ 0, /*Vector*/ 2,` 从当前函数返回。
- **L2692 EN**: Comment explains nearby logic, intent, or metadata: `Integer*/ 3);`.
  **L2692 CN**: 注释说明附近代码的逻辑、意图或元数据：`Integer*/ 3);`。
- **L2693 EN**: Introduces a switch dispatch label: `case MMAOp::Pmxvi8ger4pp:`.
  **L2693 CN**: 引入一个 switch 分发标签：`case MMAOp::Pmxvi8ger4pp:`。
- **L2694 EN**: Returns from the current function with `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2,`.
  **L2694 CN**: 以 `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2,` 从当前函数返回。
- **L2695 EN**: Comment explains nearby logic, intent, or metadata: `Integer*/ 3);`.
  **L2695 CN**: 注释说明附近代码的逻辑、意图或元数据：`Integer*/ 3);`。
- **L2696 EN**: Introduces a switch dispatch label: `case MMAOp::Pmxvi8ger4spp:`.
  **L2696 CN**: 引入一个 switch 分发标签：`case MMAOp::Pmxvi8ger4spp:`。
- **L2697 EN**: Returns from the current function with `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2,`.
  **L2697 CN**: 以 `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2,` 从当前函数返回。
- **L2698 EN**: Comment explains nearby logic, intent, or metadata: `Integer*/ 3);`.
  **L2698 CN**: 注释说明附近代码的逻辑、意图或元数据：`Integer*/ 3);`。
- **L2699 EN**: Introduces a switch dispatch label: `case MMAOp::Xvbf16ger2:`.
  **L2699 CN**: 引入一个 switch 分发标签：`case MMAOp::Xvbf16ger2:`。
- **L2700 EN**: Returns from the current function with `genMmaVqFuncType(context, /*Quad*/ 0, /*Pair*/ 0, /*Vector*/ 2)`.
  **L2700 CN**: 以 `genMmaVqFuncType(context, /*Quad*/ 0, /*Pair*/ 0, /*Vector*/ 2)` 从当前函数返回。
- **L2701 EN**: Introduces a switch dispatch label: `case MMAOp::Xvbf16ger2nn:`.
  **L2701 CN**: 引入一个 switch 分发标签：`case MMAOp::Xvbf16ger2nn:`。
- **L2702 EN**: Returns from the current function with `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2)`.
  **L2702 CN**: 以 `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2)` 从当前函数返回。
- **L2703 EN**: Introduces a switch dispatch label: `case MMAOp::Xvbf16ger2np:`.
  **L2703 CN**: 引入一个 switch 分发标签：`case MMAOp::Xvbf16ger2np:`。
- **L2704 EN**: Returns from the current function with `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2)`.
  **L2704 CN**: 以 `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2)` 从当前函数返回。
- **L2705 EN**: Introduces a switch dispatch label: `case MMAOp::Xvbf16ger2pn:`.
  **L2705 CN**: 引入一个 switch 分发标签：`case MMAOp::Xvbf16ger2pn:`。
- **L2706 EN**: Returns from the current function with `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2)`.
  **L2706 CN**: 以 `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2)` 从当前函数返回。
- **L2707 EN**: Introduces a switch dispatch label: `case MMAOp::Xvbf16ger2pp:`.
  **L2707 CN**: 引入一个 switch 分发标签：`case MMAOp::Xvbf16ger2pp:`。
- **L2708 EN**: Returns from the current function with `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2)`.
  **L2708 CN**: 以 `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2)` 从当前函数返回。
- **L2709 EN**: Introduces a switch dispatch label: `case MMAOp::Xvf16ger2:`.
  **L2709 CN**: 引入一个 switch 分发标签：`case MMAOp::Xvf16ger2:`。
- **L2710 EN**: Returns from the current function with `genMmaVqFuncType(context, /*Quad*/ 0, /*Pair*/ 0, /*Vector*/ 2)`.
  **L2710 CN**: 以 `genMmaVqFuncType(context, /*Quad*/ 0, /*Pair*/ 0, /*Vector*/ 2)` 从当前函数返回。
- **L2711 EN**: Introduces a switch dispatch label: `case MMAOp::Xvf16ger2nn:`.
  **L2711 CN**: 引入一个 switch 分发标签：`case MMAOp::Xvf16ger2nn:`。
- **L2712 EN**: Returns from the current function with `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2)`.
  **L2712 CN**: 以 `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2)` 从当前函数返回。

### Lines 2713-2736

````cpp
  case MMAOp::Xvf16ger2np:
    return genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2);
  case MMAOp::Xvf16ger2pn:
    return genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2);
  case MMAOp::Xvf16ger2pp:
    return genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2);
  case MMAOp::Xvf32ger:
    return genMmaVqFuncType(context, /*Quad*/ 0, /*Pair*/ 0, /*Vector*/ 2);
  case MMAOp::Xvf32gernn:
    return genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2);
  case MMAOp::Xvf32gernp:
    return genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2);
  case MMAOp::Xvf32gerpn:
    return genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2);
  case MMAOp::Xvf32gerpp:
    return genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2);
  case MMAOp::Xvf64ger:
    return genMmaVqFuncType(context, /*Quad*/ 0, /*Pair*/ 1, /*Vector*/ 1);
  case MMAOp::Xvf64gernn:
    return genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 1, /*Vector*/ 1);
  case MMAOp::Xvf64gernp:
    return genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 1, /*Vector*/ 1);
  case MMAOp::Xvf64gerpn:
    return genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 1, /*Vector*/ 1);
````
- **L2713 EN**: Introduces a switch dispatch label: `case MMAOp::Xvf16ger2np:`.
  **L2713 CN**: 引入一个 switch 分发标签：`case MMAOp::Xvf16ger2np:`。
- **L2714 EN**: Returns from the current function with `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2)`.
  **L2714 CN**: 以 `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2)` 从当前函数返回。
- **L2715 EN**: Introduces a switch dispatch label: `case MMAOp::Xvf16ger2pn:`.
  **L2715 CN**: 引入一个 switch 分发标签：`case MMAOp::Xvf16ger2pn:`。
- **L2716 EN**: Returns from the current function with `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2)`.
  **L2716 CN**: 以 `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2)` 从当前函数返回。
- **L2717 EN**: Introduces a switch dispatch label: `case MMAOp::Xvf16ger2pp:`.
  **L2717 CN**: 引入一个 switch 分发标签：`case MMAOp::Xvf16ger2pp:`。
- **L2718 EN**: Returns from the current function with `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2)`.
  **L2718 CN**: 以 `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2)` 从当前函数返回。
- **L2719 EN**: Introduces a switch dispatch label: `case MMAOp::Xvf32ger:`.
  **L2719 CN**: 引入一个 switch 分发标签：`case MMAOp::Xvf32ger:`。
- **L2720 EN**: Returns from the current function with `genMmaVqFuncType(context, /*Quad*/ 0, /*Pair*/ 0, /*Vector*/ 2)`.
  **L2720 CN**: 以 `genMmaVqFuncType(context, /*Quad*/ 0, /*Pair*/ 0, /*Vector*/ 2)` 从当前函数返回。
- **L2721 EN**: Introduces a switch dispatch label: `case MMAOp::Xvf32gernn:`.
  **L2721 CN**: 引入一个 switch 分发标签：`case MMAOp::Xvf32gernn:`。
- **L2722 EN**: Returns from the current function with `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2)`.
  **L2722 CN**: 以 `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2)` 从当前函数返回。
- **L2723 EN**: Introduces a switch dispatch label: `case MMAOp::Xvf32gernp:`.
  **L2723 CN**: 引入一个 switch 分发标签：`case MMAOp::Xvf32gernp:`。
- **L2724 EN**: Returns from the current function with `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2)`.
  **L2724 CN**: 以 `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2)` 从当前函数返回。
- **L2725 EN**: Introduces a switch dispatch label: `case MMAOp::Xvf32gerpn:`.
  **L2725 CN**: 引入一个 switch 分发标签：`case MMAOp::Xvf32gerpn:`。
- **L2726 EN**: Returns from the current function with `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2)`.
  **L2726 CN**: 以 `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2)` 从当前函数返回。
- **L2727 EN**: Introduces a switch dispatch label: `case MMAOp::Xvf32gerpp:`.
  **L2727 CN**: 引入一个 switch 分发标签：`case MMAOp::Xvf32gerpp:`。
- **L2728 EN**: Returns from the current function with `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2)`.
  **L2728 CN**: 以 `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2)` 从当前函数返回。
- **L2729 EN**: Introduces a switch dispatch label: `case MMAOp::Xvf64ger:`.
  **L2729 CN**: 引入一个 switch 分发标签：`case MMAOp::Xvf64ger:`。
- **L2730 EN**: Returns from the current function with `genMmaVqFuncType(context, /*Quad*/ 0, /*Pair*/ 1, /*Vector*/ 1)`.
  **L2730 CN**: 以 `genMmaVqFuncType(context, /*Quad*/ 0, /*Pair*/ 1, /*Vector*/ 1)` 从当前函数返回。
- **L2731 EN**: Introduces a switch dispatch label: `case MMAOp::Xvf64gernn:`.
  **L2731 CN**: 引入一个 switch 分发标签：`case MMAOp::Xvf64gernn:`。
- **L2732 EN**: Returns from the current function with `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 1, /*Vector*/ 1)`.
  **L2732 CN**: 以 `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 1, /*Vector*/ 1)` 从当前函数返回。
- **L2733 EN**: Introduces a switch dispatch label: `case MMAOp::Xvf64gernp:`.
  **L2733 CN**: 引入一个 switch 分发标签：`case MMAOp::Xvf64gernp:`。
- **L2734 EN**: Returns from the current function with `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 1, /*Vector*/ 1)`.
  **L2734 CN**: 以 `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 1, /*Vector*/ 1)` 从当前函数返回。
- **L2735 EN**: Introduces a switch dispatch label: `case MMAOp::Xvf64gerpn:`.
  **L2735 CN**: 引入一个 switch 分发标签：`case MMAOp::Xvf64gerpn:`。
- **L2736 EN**: Returns from the current function with `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 1, /*Vector*/ 1)`.
  **L2736 CN**: 以 `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 1, /*Vector*/ 1)` 从当前函数返回。

### Lines 2737-2760

````cpp
  case MMAOp::Xvf64gerpp:
    return genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 1, /*Vector*/ 1);
  case MMAOp::Xvi16ger2:
    return genMmaVqFuncType(context, /*Quad*/ 0, /*Pair*/ 0, /*Vector*/ 2);
  case MMAOp::Xvi16ger2pp:
    return genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2);
  case MMAOp::Xvi16ger2s:
    return genMmaVqFuncType(context, /*Quad*/ 0, /*Pair*/ 0, /*Vector*/ 2);
  case MMAOp::Xvi16ger2spp:
    return genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2);
  case MMAOp::Xvi4ger8:
    return genMmaVqFuncType(context, /*Quad*/ 0, /*Pair*/ 0, /*Vector*/ 2);
  case MMAOp::Xvi4ger8pp:
    return genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2);
  case MMAOp::Xvi8ger4:
    return genMmaVqFuncType(context, /*Quad*/ 0, /*Pair*/ 0, /*Vector*/ 2);
  case MMAOp::Xvi8ger4pp:
    return genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2);
  case MMAOp::Xvi8ger4spp:
    return genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2);
  }
  llvm_unreachable("getMmaIrFuncType");
}

````
- **L2737 EN**: Introduces a switch dispatch label: `case MMAOp::Xvf64gerpp:`.
  **L2737 CN**: 引入一个 switch 分发标签：`case MMAOp::Xvf64gerpp:`。
- **L2738 EN**: Returns from the current function with `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 1, /*Vector*/ 1)`.
  **L2738 CN**: 以 `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 1, /*Vector*/ 1)` 从当前函数返回。
- **L2739 EN**: Introduces a switch dispatch label: `case MMAOp::Xvi16ger2:`.
  **L2739 CN**: 引入一个 switch 分发标签：`case MMAOp::Xvi16ger2:`。
- **L2740 EN**: Returns from the current function with `genMmaVqFuncType(context, /*Quad*/ 0, /*Pair*/ 0, /*Vector*/ 2)`.
  **L2740 CN**: 以 `genMmaVqFuncType(context, /*Quad*/ 0, /*Pair*/ 0, /*Vector*/ 2)` 从当前函数返回。
- **L2741 EN**: Introduces a switch dispatch label: `case MMAOp::Xvi16ger2pp:`.
  **L2741 CN**: 引入一个 switch 分发标签：`case MMAOp::Xvi16ger2pp:`。
- **L2742 EN**: Returns from the current function with `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2)`.
  **L2742 CN**: 以 `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2)` 从当前函数返回。
- **L2743 EN**: Introduces a switch dispatch label: `case MMAOp::Xvi16ger2s:`.
  **L2743 CN**: 引入一个 switch 分发标签：`case MMAOp::Xvi16ger2s:`。
- **L2744 EN**: Returns from the current function with `genMmaVqFuncType(context, /*Quad*/ 0, /*Pair*/ 0, /*Vector*/ 2)`.
  **L2744 CN**: 以 `genMmaVqFuncType(context, /*Quad*/ 0, /*Pair*/ 0, /*Vector*/ 2)` 从当前函数返回。
- **L2745 EN**: Introduces a switch dispatch label: `case MMAOp::Xvi16ger2spp:`.
  **L2745 CN**: 引入一个 switch 分发标签：`case MMAOp::Xvi16ger2spp:`。
- **L2746 EN**: Returns from the current function with `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2)`.
  **L2746 CN**: 以 `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2)` 从当前函数返回。
- **L2747 EN**: Introduces a switch dispatch label: `case MMAOp::Xvi4ger8:`.
  **L2747 CN**: 引入一个 switch 分发标签：`case MMAOp::Xvi4ger8:`。
- **L2748 EN**: Returns from the current function with `genMmaVqFuncType(context, /*Quad*/ 0, /*Pair*/ 0, /*Vector*/ 2)`.
  **L2748 CN**: 以 `genMmaVqFuncType(context, /*Quad*/ 0, /*Pair*/ 0, /*Vector*/ 2)` 从当前函数返回。
- **L2749 EN**: Introduces a switch dispatch label: `case MMAOp::Xvi4ger8pp:`.
  **L2749 CN**: 引入一个 switch 分发标签：`case MMAOp::Xvi4ger8pp:`。
- **L2750 EN**: Returns from the current function with `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2)`.
  **L2750 CN**: 以 `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2)` 从当前函数返回。
- **L2751 EN**: Introduces a switch dispatch label: `case MMAOp::Xvi8ger4:`.
  **L2751 CN**: 引入一个 switch 分发标签：`case MMAOp::Xvi8ger4:`。
- **L2752 EN**: Returns from the current function with `genMmaVqFuncType(context, /*Quad*/ 0, /*Pair*/ 0, /*Vector*/ 2)`.
  **L2752 CN**: 以 `genMmaVqFuncType(context, /*Quad*/ 0, /*Pair*/ 0, /*Vector*/ 2)` 从当前函数返回。
- **L2753 EN**: Introduces a switch dispatch label: `case MMAOp::Xvi8ger4pp:`.
  **L2753 CN**: 引入一个 switch 分发标签：`case MMAOp::Xvi8ger4pp:`。
- **L2754 EN**: Returns from the current function with `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2)`.
  **L2754 CN**: 以 `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2)` 从当前函数返回。
- **L2755 EN**: Introduces a switch dispatch label: `case MMAOp::Xvi8ger4spp:`.
  **L2755 CN**: 引入一个 switch 分发标签：`case MMAOp::Xvi8ger4spp:`。
- **L2756 EN**: Returns from the current function with `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2)`.
  **L2756 CN**: 以 `genMmaVqFuncType(context, /*Quad*/ 1, /*Pair*/ 0, /*Vector*/ 2)` 从当前函数返回。
- **L2757 EN**: Closes the current lexical scope or compound statement.
  **L2757 CN**: 结束当前词法作用域或复合语句块。
- **L2758 EN**: Marks this control path as unreachable to LLVM.
  **L2758 CN**: 将该控制路径标记为 LLVM 认为不可达。
- **L2759 EN**: Closes the current lexical scope or compound statement.
  **L2759 CN**: 结束当前词法作用域或复合语句块。
- **L2760 EN**: Blank line separating nearby declarations or logic blocks.
  **L2760 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 2761-2784

````cpp
template <MMAOp IntrId, MMAHandlerOp HandlerOp>
void PPCIntrinsicLibrary::genMmaIntr(llvm::ArrayRef<fir::ExtendedValue> args) {
  auto context{builder.getContext()};
  mlir::FunctionType intrFuncType{getMmaIrFuncType(context, IntrId)};
  mlir::func::FuncOp funcOp{
      builder.createFunction(loc, getMmaIrIntrName(IntrId), intrFuncType)};
  llvm::SmallVector<mlir::Value> intrArgs;

  // Depending on SubToFunc, change the subroutine call to a function call.
  // First argument represents the result. Rest of the arguments
  // are shifted one position to form the actual argument list.
  size_t argStart{0};
  size_t argStep{1};
  size_t e{args.size()};
  if (HandlerOp == MMAHandlerOp::SubToFunc) {
    // The first argument becomes function result. Start from the second
    // argument.
    argStart = 1;
  } else if (HandlerOp == MMAHandlerOp::SubToFuncReverseArgOnLE) {
    // Reverse argument order on little-endian target only.
    // The reversal does not depend on the setting of non-native-order option.
    const auto triple{fir::getTargetTriple(builder.getModule())};
    if (triple.isLittleEndian()) {
      // Load the arguments in reverse order.
````
- **L2761 EN**: Introduces template parameters or specialization context: `template <MMAOp IntrId, MMAHandlerOp HandlerOp>`.
  **L2761 CN**: 为后续声明引入模板参数或特化上下文：`template <MMAOp IntrId, MMAHandlerOp HandlerOp>`。
- **L2762 EN**: Starts a function, method, lambda, or structured scope: `void PPCIntrinsicLibrary::genMmaIntr(llvm::ArrayRef<fir::ExtendedValue> args) {`.
  **L2762 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void PPCIntrinsicLibrary::genMmaIntr(llvm::ArrayRef<fir::ExtendedValue> args) {`。
- **L2763 EN**: Executes a call or declaration centered on `context{builder.getContext`.
  **L2763 CN**: 执行以 `context{builder.getContext` 为核心的调用或声明。
- **L2764 EN**: Executes a call or declaration centered on `intrFuncType{getMmaIrFuncType`.
  **L2764 CN**: 执行以 `intrFuncType{getMmaIrFuncType` 为核心的调用或声明。
- **L2765 EN**: Continues the surrounding expression or declaration: `mlir::func::FuncOp funcOp{`.
  **L2765 CN**: 继续构造周围的表达式或声明：`mlir::func::FuncOp funcOp{`。
- **L2766 EN**: Executes a call or declaration centered on `builder.createFunction`.
  **L2766 CN**: 执行以 `builder.createFunction` 为核心的调用或声明。
- **L2767 EN**: Executes a standalone statement or declaration: `llvm::SmallVector<mlir::Value> intrArgs;`.
  **L2767 CN**: 执行一条独立语句或声明：`llvm::SmallVector<mlir::Value> intrArgs;`。
- **L2768 EN**: Blank line separating nearby declarations or logic blocks.
  **L2768 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2769 EN**: Comment explains nearby logic, intent, or metadata: `Depending on SubToFunc, change the subroutine call to a function call.`.
  **L2769 CN**: 注释说明附近代码的逻辑、意图或元数据：`Depending on SubToFunc, change the subroutine call to a function call.`。
- **L2770 EN**: Comment explains nearby logic, intent, or metadata: `First argument represents the result. Rest of the arguments`.
  **L2770 CN**: 注释说明附近代码的逻辑、意图或元数据：`First argument represents the result. Rest of the arguments`。
- **L2771 EN**: Comment explains nearby logic, intent, or metadata: `are shifted one position to form the actual argument list.`.
  **L2771 CN**: 注释说明附近代码的逻辑、意图或元数据：`are shifted one position to form the actual argument list.`。
- **L2772 EN**: Executes a standalone statement or declaration: `size_t argStart{0};`.
  **L2772 CN**: 执行一条独立语句或声明：`size_t argStart{0};`。
- **L2773 EN**: Executes a standalone statement or declaration: `size_t argStep{1};`.
  **L2773 CN**: 执行一条独立语句或声明：`size_t argStep{1};`。
- **L2774 EN**: Executes a call or declaration centered on `e{args.size`.
  **L2774 CN**: 执行以 `e{args.size` 为核心的调用或声明。
- **L2775 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2775 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2776 EN**: Comment explains nearby logic, intent, or metadata: `The first argument becomes function result. Start from the second`.
  **L2776 CN**: 注释说明附近代码的逻辑、意图或元数据：`The first argument becomes function result. Start from the second`。
- **L2777 EN**: Comment explains nearby logic, intent, or metadata: `argument.`.
  **L2777 CN**: 注释说明附近代码的逻辑、意图或元数据：`argument.`。
- **L2778 EN**: Executes a standalone statement or declaration: `argStart = 1;`.
  **L2778 CN**: 执行一条独立语句或声明：`argStart = 1;`。
- **L2779 EN**: Transitions from the previous branch into an `else if` condition.
  **L2779 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2780 EN**: Comment explains nearby logic, intent, or metadata: `Reverse argument order on little-endian target only.`.
  **L2780 CN**: 注释说明附近代码的逻辑、意图或元数据：`Reverse argument order on little-endian target only.`。
- **L2781 EN**: Comment explains nearby logic, intent, or metadata: `The reversal does not depend on the setting of non-native-order option.`.
  **L2781 CN**: 注释说明附近代码的逻辑、意图或元数据：`The reversal does not depend on the setting of non-native-order option.`。
- **L2782 EN**: Executes a call or declaration centered on `triple{fir::getTargetTriple`.
  **L2782 CN**: 执行以 `triple{fir::getTargetTriple` 为核心的调用或声明。
- **L2783 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2783 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2784 EN**: Comment explains nearby logic, intent, or metadata: `Load the arguments in reverse order.`.
  **L2784 CN**: 注释说明附近代码的逻辑、意图或元数据：`Load the arguments in reverse order.`。

### Lines 2785-2808

````cpp
      argStart = args.size() - 1;
      // The first argument becomes function result. Stop at the second
      // argument.
      e = 0;
      argStep = -1;
    } else {
      // Load the arguments in natural order.
      // The first argument becomes function result. Start from the second
      // argument.
      argStart = 1;
    }
  }

  for (size_t i = argStart, j = 0; i != e; i += argStep, ++j) {
    auto v{fir::getBase(args[i])};
    if (i == 0 && HandlerOp == MMAHandlerOp::FirstArgIsResult) {
      // First argument is passed in as an address. We need to load
      // the content to match the LLVM interface.
      v = fir::LoadOp::create(builder, loc, v);
    }
    auto vType{v.getType()};
    mlir::Type targetType{intrFuncType.getInput(j)};
    if (vType != targetType) {
      if (mlir::isa<mlir::VectorType>(targetType)) {
````
- **L2785 EN**: Executes a call or declaration centered on `args.size`.
  **L2785 CN**: 执行以 `args.size` 为核心的调用或声明。
- **L2786 EN**: Comment explains nearby logic, intent, or metadata: `The first argument becomes function result. Stop at the second`.
  **L2786 CN**: 注释说明附近代码的逻辑、意图或元数据：`The first argument becomes function result. Stop at the second`。
- **L2787 EN**: Comment explains nearby logic, intent, or metadata: `argument.`.
  **L2787 CN**: 注释说明附近代码的逻辑、意图或元数据：`argument.`。
- **L2788 EN**: Executes a standalone statement or declaration: `e = 0;`.
  **L2788 CN**: 执行一条独立语句或声明：`e = 0;`。
- **L2789 EN**: Executes a standalone statement or declaration: `argStep = -1;`.
  **L2789 CN**: 执行一条独立语句或声明：`argStep = -1;`。
- **L2790 EN**: Transitions from the previous branch into the alternative path.
  **L2790 CN**: 从前一个分支过渡到备选路径。
- **L2791 EN**: Comment explains nearby logic, intent, or metadata: `Load the arguments in natural order.`.
  **L2791 CN**: 注释说明附近代码的逻辑、意图或元数据：`Load the arguments in natural order.`。
- **L2792 EN**: Comment explains nearby logic, intent, or metadata: `The first argument becomes function result. Start from the second`.
  **L2792 CN**: 注释说明附近代码的逻辑、意图或元数据：`The first argument becomes function result. Start from the second`。
- **L2793 EN**: Comment explains nearby logic, intent, or metadata: `argument.`.
  **L2793 CN**: 注释说明附近代码的逻辑、意图或元数据：`argument.`。
- **L2794 EN**: Executes a standalone statement or declaration: `argStart = 1;`.
  **L2794 CN**: 执行一条独立语句或声明：`argStart = 1;`。
- **L2795 EN**: Closes the current lexical scope or compound statement.
  **L2795 CN**: 结束当前词法作用域或复合语句块。
- **L2796 EN**: Closes the current lexical scope or compound statement.
  **L2796 CN**: 结束当前词法作用域或复合语句块。
- **L2797 EN**: Blank line separating nearby declarations or logic blocks.
  **L2797 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2798 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L2798 CN**: 开始 `for` 控制流语句并计算其条件。
- **L2799 EN**: Executes a call or declaration centered on `v{fir::getBase`.
  **L2799 CN**: 执行以 `v{fir::getBase` 为核心的调用或声明。
- **L2800 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2800 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2801 EN**: Comment explains nearby logic, intent, or metadata: `First argument is passed in as an address. We need to load`.
  **L2801 CN**: 注释说明附近代码的逻辑、意图或元数据：`First argument is passed in as an address. We need to load`。
- **L2802 EN**: Comment explains nearby logic, intent, or metadata: `the content to match the LLVM interface.`.
  **L2802 CN**: 注释说明附近代码的逻辑、意图或元数据：`the content to match the LLVM interface.`。
- **L2803 EN**: Executes a call or declaration centered on `fir::LoadOp::create`.
  **L2803 CN**: 执行以 `fir::LoadOp::create` 为核心的调用或声明。
- **L2804 EN**: Closes the current lexical scope or compound statement.
  **L2804 CN**: 结束当前词法作用域或复合语句块。
- **L2805 EN**: Executes a call or declaration centered on `vType{v.getType`.
  **L2805 CN**: 执行以 `vType{v.getType` 为核心的调用或声明。
- **L2806 EN**: Executes a call or declaration centered on `targetType{intrFuncType.getInput`.
  **L2806 CN**: 执行以 `targetType{intrFuncType.getInput` 为核心的调用或声明。
- **L2807 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2807 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2808 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2808 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 2809-2832

````cpp
        // Perform vector type conversion for arguments passed by value.
        auto eleTy{mlir::dyn_cast<fir::VectorType>(vType).getElementType()};
        auto len{mlir::dyn_cast<fir::VectorType>(vType).getLen()};
        mlir::VectorType mlirType = mlir::VectorType::get(len, eleTy);
        auto v0{builder.createConvert(loc, mlirType, v)};
        auto v1{mlir::vector::BitCastOp::create(builder, loc, targetType, v0)};
        intrArgs.push_back(v1);
      } else if (mlir::isa<mlir::IntegerType>(targetType) &&
                 mlir::isa<mlir::IntegerType>(vType)) {
        auto v0{builder.createConvert(loc, targetType, v)};
        intrArgs.push_back(v0);
      } else {
        llvm::errs() << "\nUnexpected type conversion requested: "
                     << " from " << vType << " to " << targetType << "\n";
        llvm_unreachable("Unsupported type conversion for argument to PowerPC "
                         "MMA intrinsic");
      }
    } else {
      intrArgs.push_back(v);
    }
  }
  auto callSt{fir::CallOp::create(builder, loc, funcOp, intrArgs)};
  if (HandlerOp == MMAHandlerOp::SubToFunc ||
      HandlerOp == MMAHandlerOp::SubToFuncReverseArgOnLE ||
````
- **L2809 EN**: Comment explains nearby logic, intent, or metadata: `Perform vector type conversion for arguments passed by value.`.
  **L2809 CN**: 注释说明附近代码的逻辑、意图或元数据：`Perform vector type conversion for arguments passed by value.`。
- **L2810 EN**: Executes a call or declaration centered on `eleTy{mlir::dyn_cast<fir::VectorType>`.
  **L2810 CN**: 执行以 `eleTy{mlir::dyn_cast<fir::VectorType>` 为核心的调用或声明。
- **L2811 EN**: Executes a call or declaration centered on `len{mlir::dyn_cast<fir::VectorType>`.
  **L2811 CN**: 执行以 `len{mlir::dyn_cast<fir::VectorType>` 为核心的调用或声明。
- **L2812 EN**: Initializes variable `mlirType` from the right-hand expression.
  **L2812 CN**: 使用右侧表达式初始化变量 `mlirType`。
- **L2813 EN**: Executes a call or declaration centered on `v0{builder.createConvert`.
  **L2813 CN**: 执行以 `v0{builder.createConvert` 为核心的调用或声明。
- **L2814 EN**: Executes a call or declaration centered on `v1{mlir::vector::BitCastOp::create`.
  **L2814 CN**: 执行以 `v1{mlir::vector::BitCastOp::create` 为核心的调用或声明。
- **L2815 EN**: Executes a call or declaration centered on `intrArgs.push_back`.
  **L2815 CN**: 执行以 `intrArgs.push_back` 为核心的调用或声明。
- **L2816 EN**: Transitions from the previous branch into an `else if` condition.
  **L2816 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2817 EN**: Starts a function, method, lambda, or structured scope: `mlir::isa<mlir::IntegerType>(vType)) {`.
  **L2817 CN**: 开始一个函数、方法、lambda 或结构化作用域：`mlir::isa<mlir::IntegerType>(vType)) {`。
- **L2818 EN**: Executes a call or declaration centered on `v0{builder.createConvert`.
  **L2818 CN**: 执行以 `v0{builder.createConvert` 为核心的调用或声明。
- **L2819 EN**: Executes a call or declaration centered on `intrArgs.push_back`.
  **L2819 CN**: 执行以 `intrArgs.push_back` 为核心的调用或声明。
- **L2820 EN**: Transitions from the previous branch into the alternative path.
  **L2820 CN**: 从前一个分支过渡到备选路径。
- **L2821 EN**: Continues logic associated with callable symbol `errs`.
  **L2821 CN**: 继续与可调用符号 `errs` 相关的逻辑。
- **L2822 EN**: Executes a standalone statement or declaration: `<< " from " << vType << " to " << targetType << "\n";`.
  **L2822 CN**: 执行一条独立语句或声明：`<< " from " << vType << " to " << targetType << "\n";`。
- **L2823 EN**: Marks this control path as unreachable to LLVM.
  **L2823 CN**: 将该控制路径标记为 LLVM 认为不可达。
- **L2824 EN**: Executes a standalone statement or declaration: `"MMA intrinsic");`.
  **L2824 CN**: 执行一条独立语句或声明：`"MMA intrinsic");`。
- **L2825 EN**: Closes the current lexical scope or compound statement.
  **L2825 CN**: 结束当前词法作用域或复合语句块。
- **L2826 EN**: Transitions from the previous branch into the alternative path.
  **L2826 CN**: 从前一个分支过渡到备选路径。
- **L2827 EN**: Executes a call or declaration centered on `intrArgs.push_back`.
  **L2827 CN**: 执行以 `intrArgs.push_back` 为核心的调用或声明。
- **L2828 EN**: Closes the current lexical scope or compound statement.
  **L2828 CN**: 结束当前词法作用域或复合语句块。
- **L2829 EN**: Closes the current lexical scope or compound statement.
  **L2829 CN**: 结束当前词法作用域或复合语句块。
- **L2830 EN**: Executes a call or declaration centered on `callSt{fir::CallOp::create`.
  **L2830 CN**: 执行以 `callSt{fir::CallOp::create` 为核心的调用或声明。
- **L2831 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2831 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2832 EN**: Continues the surrounding expression or declaration: `HandlerOp == MMAHandlerOp::SubToFuncReverseArgOnLE ||`.
  **L2832 CN**: 继续构造周围的表达式或声明：`HandlerOp == MMAHandlerOp::SubToFuncReverseArgOnLE ||`。

### Lines 2833-2856

````cpp
      HandlerOp == MMAHandlerOp::FirstArgIsResult) {
    // Convert pointer type if needed.
    mlir::Value callResult{callSt.getResult(0)};
    mlir::Value destPtr{fir::getBase(args[0])};
    mlir::Type callResultPtrType{builder.getRefType(callResult.getType())};
    if (destPtr.getType() != callResultPtrType) {
      destPtr =
          fir::ConvertOp::create(builder, loc, callResultPtrType, destPtr);
    }
    // Copy the result.
    fir::StoreOp::create(builder, loc, callResult, destPtr);
  }
}

// VEC_ST, VEC_STE
template <VecOp vop>
void PPCIntrinsicLibrary::genVecStore(llvm::ArrayRef<fir::ExtendedValue> args) {
  assert(args.size() == 3);

  auto context{builder.getContext()};
  auto argBases{getBasesForArgs(args)};
  auto arg1TyInfo{getVecTypeFromFir(argBases[0])};

  auto addr{addOffsetToAddress(builder, loc, argBases[2], argBases[1])};
````
- **L2833 EN**: Continues the surrounding expression or declaration: `HandlerOp == MMAHandlerOp::FirstArgIsResult) {`.
  **L2833 CN**: 继续构造周围的表达式或声明：`HandlerOp == MMAHandlerOp::FirstArgIsResult) {`。
- **L2834 EN**: Comment explains nearby logic, intent, or metadata: `Convert pointer type if needed.`.
  **L2834 CN**: 注释说明附近代码的逻辑、意图或元数据：`Convert pointer type if needed.`。
- **L2835 EN**: Executes a call or declaration centered on `callResult{callSt.getResult`.
  **L2835 CN**: 执行以 `callResult{callSt.getResult` 为核心的调用或声明。
- **L2836 EN**: Executes a call or declaration centered on `destPtr{fir::getBase`.
  **L2836 CN**: 执行以 `destPtr{fir::getBase` 为核心的调用或声明。
- **L2837 EN**: Executes a call or declaration centered on `callResultPtrType{builder.getRefType`.
  **L2837 CN**: 执行以 `callResultPtrType{builder.getRefType` 为核心的调用或声明。
- **L2838 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2838 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2839 EN**: Continues the surrounding expression or declaration: `destPtr =`.
  **L2839 CN**: 继续构造周围的表达式或声明：`destPtr =`。
- **L2840 EN**: Executes a call or declaration centered on `fir::ConvertOp::create`.
  **L2840 CN**: 执行以 `fir::ConvertOp::create` 为核心的调用或声明。
- **L2841 EN**: Closes the current lexical scope or compound statement.
  **L2841 CN**: 结束当前词法作用域或复合语句块。
- **L2842 EN**: Comment explains nearby logic, intent, or metadata: `Copy the result.`.
  **L2842 CN**: 注释说明附近代码的逻辑、意图或元数据：`Copy the result.`。
- **L2843 EN**: Executes a call or declaration centered on `fir::StoreOp::create`.
  **L2843 CN**: 执行以 `fir::StoreOp::create` 为核心的调用或声明。
- **L2844 EN**: Closes the current lexical scope or compound statement.
  **L2844 CN**: 结束当前词法作用域或复合语句块。
- **L2845 EN**: Closes the current lexical scope or compound statement.
  **L2845 CN**: 结束当前词法作用域或复合语句块。
- **L2846 EN**: Blank line separating nearby declarations or logic blocks.
  **L2846 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2847 EN**: Comment explains nearby logic, intent, or metadata: `VEC_ST, VEC_STE`.
  **L2847 CN**: 注释说明附近代码的逻辑、意图或元数据：`VEC_ST, VEC_STE`。
- **L2848 EN**: Introduces template parameters or specialization context: `template <VecOp vop>`.
  **L2848 CN**: 为后续声明引入模板参数或特化上下文：`template <VecOp vop>`。
- **L2849 EN**: Starts a function, method, lambda, or structured scope: `void PPCIntrinsicLibrary::genVecStore(llvm::ArrayRef<fir::ExtendedValue> args) {`.
  **L2849 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void PPCIntrinsicLibrary::genVecStore(llvm::ArrayRef<fir::ExtendedValue> args) {`。
- **L2850 EN**: Checks an internal invariant in debug builds.
  **L2850 CN**: 在调试构建中检查内部不变式。
- **L2851 EN**: Blank line separating nearby declarations or logic blocks.
  **L2851 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2852 EN**: Executes a call or declaration centered on `context{builder.getContext`.
  **L2852 CN**: 执行以 `context{builder.getContext` 为核心的调用或声明。
- **L2853 EN**: Executes a call or declaration centered on `argBases{getBasesForArgs`.
  **L2853 CN**: 执行以 `argBases{getBasesForArgs` 为核心的调用或声明。
- **L2854 EN**: Executes a call or declaration centered on `arg1TyInfo{getVecTypeFromFir`.
  **L2854 CN**: 执行以 `arg1TyInfo{getVecTypeFromFir` 为核心的调用或声明。
- **L2855 EN**: Blank line separating nearby declarations or logic blocks.
  **L2855 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2856 EN**: Executes a call or declaration centered on `addr{addOffsetToAddress`.
  **L2856 CN**: 执行以 `addr{addOffsetToAddress` 为核心的调用或声明。

### Lines 2857-2880

````cpp

  llvm::StringRef fname{};
  mlir::VectorType stTy{nullptr};
  auto i32ty{mlir::IntegerType::get(context, 32)};
  switch (vop) {
  case VecOp::St:
    stTy = mlir::VectorType::get(4, i32ty);
    fname = "llvm.ppc.altivec.stvx";
    break;
  case VecOp::Ste: {
    const auto width{arg1TyInfo.eleTy.getIntOrFloatBitWidth()};
    const auto len{arg1TyInfo.len};

    if (arg1TyInfo.isFloat32()) {
      stTy = mlir::VectorType::get(len, i32ty);
      fname = "llvm.ppc.altivec.stvewx";
    } else if (mlir::isa<mlir::IntegerType>(arg1TyInfo.eleTy)) {
      stTy = mlir::VectorType::get(len, mlir::IntegerType::get(context, width));

      switch (width) {
      case 8:
        fname = "llvm.ppc.altivec.stvebx";
        break;
      case 16:
````
- **L2857 EN**: Blank line separating nearby declarations or logic blocks.
  **L2857 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2858 EN**: Executes a standalone statement or declaration: `llvm::StringRef fname{};`.
  **L2858 CN**: 执行一条独立语句或声明：`llvm::StringRef fname{};`。
- **L2859 EN**: Executes a standalone statement or declaration: `mlir::VectorType stTy{nullptr};`.
  **L2859 CN**: 执行一条独立语句或声明：`mlir::VectorType stTy{nullptr};`。
- **L2860 EN**: Executes a call or declaration centered on `i32ty{mlir::IntegerType::get`.
  **L2860 CN**: 执行以 `i32ty{mlir::IntegerType::get` 为核心的调用或声明。
- **L2861 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L2861 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L2862 EN**: Introduces a switch dispatch label: `case VecOp::St:`.
  **L2862 CN**: 引入一个 switch 分发标签：`case VecOp::St:`。
- **L2863 EN**: Executes a call or declaration centered on `mlir::VectorType::get`.
  **L2863 CN**: 执行以 `mlir::VectorType::get` 为核心的调用或声明。
- **L2864 EN**: Executes a standalone statement or declaration: `fname = "llvm.ppc.altivec.stvx";`.
  **L2864 CN**: 执行一条独立语句或声明：`fname = "llvm.ppc.altivec.stvx";`。
- **L2865 EN**: Exits the nearest loop or switch statement.
  **L2865 CN**: 退出最近的循环或 switch 语句。
- **L2866 EN**: Introduces a switch dispatch label: `case VecOp::Ste: {`.
  **L2866 CN**: 引入一个 switch 分发标签：`case VecOp::Ste: {`。
- **L2867 EN**: Executes a call or declaration centered on `width{arg1TyInfo.eleTy.getIntOrFloatBitWidth`.
  **L2867 CN**: 执行以 `width{arg1TyInfo.eleTy.getIntOrFloatBitWidth` 为核心的调用或声明。
- **L2868 EN**: Executes a standalone statement or declaration: `const auto len{arg1TyInfo.len};`.
  **L2868 CN**: 执行一条独立语句或声明：`const auto len{arg1TyInfo.len};`。
- **L2869 EN**: Blank line separating nearby declarations or logic blocks.
  **L2869 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2870 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2870 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2871 EN**: Executes a call or declaration centered on `mlir::VectorType::get`.
  **L2871 CN**: 执行以 `mlir::VectorType::get` 为核心的调用或声明。
- **L2872 EN**: Executes a standalone statement or declaration: `fname = "llvm.ppc.altivec.stvewx";`.
  **L2872 CN**: 执行一条独立语句或声明：`fname = "llvm.ppc.altivec.stvewx";`。
- **L2873 EN**: Transitions from the previous branch into an `else if` condition.
  **L2873 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L2874 EN**: Executes a call or declaration centered on `mlir::VectorType::get`.
  **L2874 CN**: 执行以 `mlir::VectorType::get` 为核心的调用或声明。
- **L2875 EN**: Blank line separating nearby declarations or logic blocks.
  **L2875 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2876 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L2876 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L2877 EN**: Introduces a switch dispatch label: `case 8:`.
  **L2877 CN**: 引入一个 switch 分发标签：`case 8:`。
- **L2878 EN**: Executes a standalone statement or declaration: `fname = "llvm.ppc.altivec.stvebx";`.
  **L2878 CN**: 执行一条独立语句或声明：`fname = "llvm.ppc.altivec.stvebx";`。
- **L2879 EN**: Exits the nearest loop or switch statement.
  **L2879 CN**: 退出最近的循环或 switch 语句。
- **L2880 EN**: Introduces a switch dispatch label: `case 16:`.
  **L2880 CN**: 引入一个 switch 分发标签：`case 16:`。

### Lines 2881-2904

````cpp
        fname = "llvm.ppc.altivec.stvehx";
        break;
      case 32:
        fname = "llvm.ppc.altivec.stvewx";
        break;
      default:
        assert(false && "invalid element size");
      }
    } else
      assert(false && "unknown type");
    break;
  }
  case VecOp::Stxvp:
    // __vector_pair type
    stTy = mlir::VectorType::get(256, mlir::IntegerType::get(context, 1));
    fname = "llvm.ppc.vsx.stxvp";
    break;
  default:
    llvm_unreachable("invalid vector operation for generator");
  }

  auto funcType{mlir::FunctionType::get(context, {stTy, addr.getType()}, {})};
  mlir::func::FuncOp funcOp = builder.createFunction(loc, fname, funcType);

````
- **L2881 EN**: Executes a standalone statement or declaration: `fname = "llvm.ppc.altivec.stvehx";`.
  **L2881 CN**: 执行一条独立语句或声明：`fname = "llvm.ppc.altivec.stvehx";`。
- **L2882 EN**: Exits the nearest loop or switch statement.
  **L2882 CN**: 退出最近的循环或 switch 语句。
- **L2883 EN**: Introduces a switch dispatch label: `case 32:`.
  **L2883 CN**: 引入一个 switch 分发标签：`case 32:`。
- **L2884 EN**: Executes a standalone statement or declaration: `fname = "llvm.ppc.altivec.stvewx";`.
  **L2884 CN**: 执行一条独立语句或声明：`fname = "llvm.ppc.altivec.stvewx";`。
- **L2885 EN**: Exits the nearest loop or switch statement.
  **L2885 CN**: 退出最近的循环或 switch 语句。
- **L2886 EN**: Introduces a switch dispatch label: `default:`.
  **L2886 CN**: 引入一个 switch 分发标签：`default:`。
- **L2887 EN**: Checks an internal invariant in debug builds.
  **L2887 CN**: 在调试构建中检查内部不变式。
- **L2888 EN**: Closes the current lexical scope or compound statement.
  **L2888 CN**: 结束当前词法作用域或复合语句块。
- **L2889 EN**: Transitions from the previous branch into the alternative path.
  **L2889 CN**: 从前一个分支过渡到备选路径。
- **L2890 EN**: Checks an internal invariant in debug builds.
  **L2890 CN**: 在调试构建中检查内部不变式。
- **L2891 EN**: Exits the nearest loop or switch statement.
  **L2891 CN**: 退出最近的循环或 switch 语句。
- **L2892 EN**: Closes the current lexical scope or compound statement.
  **L2892 CN**: 结束当前词法作用域或复合语句块。
- **L2893 EN**: Introduces a switch dispatch label: `case VecOp::Stxvp:`.
  **L2893 CN**: 引入一个 switch 分发标签：`case VecOp::Stxvp:`。
- **L2894 EN**: Comment explains nearby logic, intent, or metadata: `__vector_pair type`.
  **L2894 CN**: 注释说明附近代码的逻辑、意图或元数据：`__vector_pair type`。
- **L2895 EN**: Executes a call or declaration centered on `mlir::VectorType::get`.
  **L2895 CN**: 执行以 `mlir::VectorType::get` 为核心的调用或声明。
- **L2896 EN**: Executes a standalone statement or declaration: `fname = "llvm.ppc.vsx.stxvp";`.
  **L2896 CN**: 执行一条独立语句或声明：`fname = "llvm.ppc.vsx.stxvp";`。
- **L2897 EN**: Exits the nearest loop or switch statement.
  **L2897 CN**: 退出最近的循环或 switch 语句。
- **L2898 EN**: Introduces a switch dispatch label: `default:`.
  **L2898 CN**: 引入一个 switch 分发标签：`default:`。
- **L2899 EN**: Marks this control path as unreachable to LLVM.
  **L2899 CN**: 将该控制路径标记为 LLVM 认为不可达。
- **L2900 EN**: Closes the current lexical scope or compound statement.
  **L2900 CN**: 结束当前词法作用域或复合语句块。
- **L2901 EN**: Blank line separating nearby declarations or logic blocks.
  **L2901 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2902 EN**: Executes a call or declaration centered on `funcType{mlir::FunctionType::get`.
  **L2902 CN**: 执行以 `funcType{mlir::FunctionType::get` 为核心的调用或声明。
- **L2903 EN**: Initializes variable `funcOp` from the right-hand expression.
  **L2903 CN**: 使用右侧表达式初始化变量 `funcOp`。
- **L2904 EN**: Blank line separating nearby declarations or logic blocks.
  **L2904 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 2905-2928

````cpp
  llvm::SmallVector<mlir::Value, 4> biArgs;

  if (vop == VecOp::Stxvp) {
    biArgs.push_back(argBases[0]);
    biArgs.push_back(addr);
    fir::CallOp::create(builder, loc, funcOp, biArgs);
    return;
  }

  auto vecTyInfo{getVecTypeFromFirType(argBases[0].getType())};
  auto cnv{builder.createConvert(loc, vecTyInfo.toMlirVectorType(context),
                                 argBases[0])};

  mlir::Value newArg1{nullptr};
  if (stTy != arg1TyInfo.toMlirVectorType(context))
    newArg1 = mlir::vector::BitCastOp::create(builder, loc, stTy, cnv);
  else
    newArg1 = cnv;

  if (isBEVecElemOrderOnLE())
    newArg1 = builder.createConvert(
        loc, stTy, reverseVectorElements(builder, loc, newArg1, 4));

  biArgs.push_back(newArg1);
````
- **L2905 EN**: Executes a standalone statement or declaration: `llvm::SmallVector<mlir::Value, 4> biArgs;`.
  **L2905 CN**: 执行一条独立语句或声明：`llvm::SmallVector<mlir::Value, 4> biArgs;`。
- **L2906 EN**: Blank line separating nearby declarations or logic blocks.
  **L2906 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2907 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2907 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2908 EN**: Executes a call or declaration centered on `biArgs.push_back`.
  **L2908 CN**: 执行以 `biArgs.push_back` 为核心的调用或声明。
- **L2909 EN**: Executes a call or declaration centered on `biArgs.push_back`.
  **L2909 CN**: 执行以 `biArgs.push_back` 为核心的调用或声明。
- **L2910 EN**: Executes a call or declaration centered on `fir::CallOp::create`.
  **L2910 CN**: 执行以 `fir::CallOp::create` 为核心的调用或声明。
- **L2911 EN**: Returns from the current function with `void`.
  **L2911 CN**: 以 `void` 从当前函数返回。
- **L2912 EN**: Closes the current lexical scope or compound statement.
  **L2912 CN**: 结束当前词法作用域或复合语句块。
- **L2913 EN**: Blank line separating nearby declarations or logic blocks.
  **L2913 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2914 EN**: Executes a call or declaration centered on `vecTyInfo{getVecTypeFromFirType`.
  **L2914 CN**: 执行以 `vecTyInfo{getVecTypeFromFirType` 为核心的调用或声明。
- **L2915 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto cnv{builder.createConvert(loc, vecTyInfo.toMlirVectorType(context),`.
  **L2915 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto cnv{builder.createConvert(loc, vecTyInfo.toMlirVectorType(context),`。
- **L2916 EN**: Executes a standalone statement or declaration: `argBases[0])};`.
  **L2916 CN**: 执行一条独立语句或声明：`argBases[0])};`。
- **L2917 EN**: Blank line separating nearby declarations or logic blocks.
  **L2917 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2918 EN**: Executes a standalone statement or declaration: `mlir::Value newArg1{nullptr};`.
  **L2918 CN**: 执行一条独立语句或声明：`mlir::Value newArg1{nullptr};`。
- **L2919 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2919 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2920 EN**: Executes a call or declaration centered on `mlir::vector::BitCastOp::create`.
  **L2920 CN**: 执行以 `mlir::vector::BitCastOp::create` 为核心的调用或声明。
- **L2921 EN**: Transitions from the previous branch into the alternative path.
  **L2921 CN**: 从前一个分支过渡到备选路径。
- **L2922 EN**: Executes a standalone statement or declaration: `newArg1 = cnv;`.
  **L2922 CN**: 执行一条独立语句或声明：`newArg1 = cnv;`。
- **L2923 EN**: Blank line separating nearby declarations or logic blocks.
  **L2923 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2924 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2924 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2925 EN**: Continues logic associated with callable symbol `createConvert`.
  **L2925 CN**: 继续与可调用符号 `createConvert` 相关的逻辑。
- **L2926 EN**: Executes a call or declaration centered on `reverseVectorElements`.
  **L2926 CN**: 执行以 `reverseVectorElements` 为核心的调用或声明。
- **L2927 EN**: Blank line separating nearby declarations or logic blocks.
  **L2927 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2928 EN**: Executes a call or declaration centered on `biArgs.push_back`.
  **L2928 CN**: 执行以 `biArgs.push_back` 为核心的调用或声明。

### Lines 2929-2952

````cpp
  biArgs.push_back(addr);

  fir::CallOp::create(builder, loc, funcOp, biArgs);
}

// VEC_XST, VEC_XST_BE, VEC_STXV, VEC_XSTD2, VEC_XSTW4
template <VecOp vop>
void PPCIntrinsicLibrary::genVecXStore(
    llvm::ArrayRef<fir::ExtendedValue> args) {
  assert(args.size() == 3);
  auto context{builder.getContext()};
  auto argBases{getBasesForArgs(args)};
  VecTypeInfo arg1TyInfo{getVecTypeFromFir(argBases[0])};

  auto addr{addOffsetToAddress(builder, loc, argBases[2], argBases[1])};

  mlir::Value trg{nullptr};
  mlir::Value src{nullptr};

  switch (vop) {
  case VecOp::Xst:
  case VecOp::Xst_be: {
    src = argBases[0];
    trg = builder.createConvert(loc, builder.getRefType(argBases[0].getType()),
````
- **L2929 EN**: Executes a call or declaration centered on `biArgs.push_back`.
  **L2929 CN**: 执行以 `biArgs.push_back` 为核心的调用或声明。
- **L2930 EN**: Blank line separating nearby declarations or logic blocks.
  **L2930 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2931 EN**: Executes a call or declaration centered on `fir::CallOp::create`.
  **L2931 CN**: 执行以 `fir::CallOp::create` 为核心的调用或声明。
- **L2932 EN**: Closes the current lexical scope or compound statement.
  **L2932 CN**: 结束当前词法作用域或复合语句块。
- **L2933 EN**: Blank line separating nearby declarations or logic blocks.
  **L2933 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2934 EN**: Comment explains nearby logic, intent, or metadata: `VEC_XST, VEC_XST_BE, VEC_STXV, VEC_XSTD2, VEC_XSTW4`.
  **L2934 CN**: 注释说明附近代码的逻辑、意图或元数据：`VEC_XST, VEC_XST_BE, VEC_STXV, VEC_XSTD2, VEC_XSTW4`。
- **L2935 EN**: Introduces template parameters or specialization context: `template <VecOp vop>`.
  **L2935 CN**: 为后续声明引入模板参数或特化上下文：`template <VecOp vop>`。
- **L2936 EN**: Continues logic associated with callable symbol `genVecXStore`.
  **L2936 CN**: 继续与可调用符号 `genVecXStore` 相关的逻辑。
- **L2937 EN**: Continues the surrounding expression or declaration: `llvm::ArrayRef<fir::ExtendedValue> args) {`.
  **L2937 CN**: 继续构造周围的表达式或声明：`llvm::ArrayRef<fir::ExtendedValue> args) {`。
- **L2938 EN**: Checks an internal invariant in debug builds.
  **L2938 CN**: 在调试构建中检查内部不变式。
- **L2939 EN**: Executes a call or declaration centered on `context{builder.getContext`.
  **L2939 CN**: 执行以 `context{builder.getContext` 为核心的调用或声明。
- **L2940 EN**: Executes a call or declaration centered on `argBases{getBasesForArgs`.
  **L2940 CN**: 执行以 `argBases{getBasesForArgs` 为核心的调用或声明。
- **L2941 EN**: Executes a call or declaration centered on `arg1TyInfo{getVecTypeFromFir`.
  **L2941 CN**: 执行以 `arg1TyInfo{getVecTypeFromFir` 为核心的调用或声明。
- **L2942 EN**: Blank line separating nearby declarations or logic blocks.
  **L2942 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2943 EN**: Executes a call or declaration centered on `addr{addOffsetToAddress`.
  **L2943 CN**: 执行以 `addr{addOffsetToAddress` 为核心的调用或声明。
- **L2944 EN**: Blank line separating nearby declarations or logic blocks.
  **L2944 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2945 EN**: Executes a standalone statement or declaration: `mlir::Value trg{nullptr};`.
  **L2945 CN**: 执行一条独立语句或声明：`mlir::Value trg{nullptr};`。
- **L2946 EN**: Executes a standalone statement or declaration: `mlir::Value src{nullptr};`.
  **L2946 CN**: 执行一条独立语句或声明：`mlir::Value src{nullptr};`。
- **L2947 EN**: Blank line separating nearby declarations or logic blocks.
  **L2947 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2948 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L2948 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L2949 EN**: Introduces a switch dispatch label: `case VecOp::Xst:`.
  **L2949 CN**: 引入一个 switch 分发标签：`case VecOp::Xst:`。
- **L2950 EN**: Introduces a switch dispatch label: `case VecOp::Xst_be: {`.
  **L2950 CN**: 引入一个 switch 分发标签：`case VecOp::Xst_be: {`。
- **L2951 EN**: Executes a standalone statement or declaration: `src = argBases[0];`.
  **L2951 CN**: 执行一条独立语句或声明：`src = argBases[0];`。
- **L2952 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `trg = builder.createConvert(loc, builder.getRefType(argBases[0].getType()),`.
  **L2952 CN**: 继续一个多行参数列表、初始化器或聚合项：`trg = builder.createConvert(loc, builder.getRefType(argBases[0].getType()),`。

### Lines 2953-2976

````cpp
                                addr);

    if (vop == VecOp::Xst_be || isBEVecElemOrderOnLE()) {
      auto cnv{builder.createConvert(loc, arg1TyInfo.toMlirVectorType(context),
                                     argBases[0])};
      auto shf{reverseVectorElements(builder, loc, cnv, arg1TyInfo.len)};

      src = builder.createConvert(loc, arg1TyInfo.toFirVectorType(), shf);
    }
    break;
  }
  case VecOp::Xstd2:
  case VecOp::Xstw4: {
    // an 16-byte vector arg1 is treated as two 8-byte elements or
    // four 4-byte elements
    mlir::IntegerType elemTy;
    uint64_t numElem = (vop == VecOp::Xstd2) ? 2 : 4;
    elemTy = builder.getIntegerType(128 / numElem);

    mlir::VectorType mlirVecTy{mlir::VectorType::get(numElem, elemTy)};
    fir::VectorType firVecTy{fir::VectorType::get(numElem, elemTy)};

    auto cnv{builder.createConvert(loc, arg1TyInfo.toMlirVectorType(context),
                                   argBases[0])};
````
- **L2953 EN**: Executes a standalone statement or declaration: `addr);`.
  **L2953 CN**: 执行一条独立语句或声明：`addr);`。
- **L2954 EN**: Blank line separating nearby declarations or logic blocks.
  **L2954 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2955 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2955 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2956 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto cnv{builder.createConvert(loc, arg1TyInfo.toMlirVectorType(context),`.
  **L2956 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto cnv{builder.createConvert(loc, arg1TyInfo.toMlirVectorType(context),`。
- **L2957 EN**: Executes a standalone statement or declaration: `argBases[0])};`.
  **L2957 CN**: 执行一条独立语句或声明：`argBases[0])};`。
- **L2958 EN**: Executes a call or declaration centered on `shf{reverseVectorElements`.
  **L2958 CN**: 执行以 `shf{reverseVectorElements` 为核心的调用或声明。
- **L2959 EN**: Blank line separating nearby declarations or logic blocks.
  **L2959 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2960 EN**: Executes a call or declaration centered on `builder.createConvert`.
  **L2960 CN**: 执行以 `builder.createConvert` 为核心的调用或声明。
- **L2961 EN**: Closes the current lexical scope or compound statement.
  **L2961 CN**: 结束当前词法作用域或复合语句块。
- **L2962 EN**: Exits the nearest loop or switch statement.
  **L2962 CN**: 退出最近的循环或 switch 语句。
- **L2963 EN**: Closes the current lexical scope or compound statement.
  **L2963 CN**: 结束当前词法作用域或复合语句块。
- **L2964 EN**: Introduces a switch dispatch label: `case VecOp::Xstd2:`.
  **L2964 CN**: 引入一个 switch 分发标签：`case VecOp::Xstd2:`。
- **L2965 EN**: Introduces a switch dispatch label: `case VecOp::Xstw4: {`.
  **L2965 CN**: 引入一个 switch 分发标签：`case VecOp::Xstw4: {`。
- **L2966 EN**: Comment explains nearby logic, intent, or metadata: `an 16-byte vector arg1 is treated as two 8-byte elements or`.
  **L2966 CN**: 注释说明附近代码的逻辑、意图或元数据：`an 16-byte vector arg1 is treated as two 8-byte elements or`。
- **L2967 EN**: Comment explains nearby logic, intent, or metadata: `four 4-byte elements`.
  **L2967 CN**: 注释说明附近代码的逻辑、意图或元数据：`four 4-byte elements`。
- **L2968 EN**: Executes a standalone statement or declaration: `mlir::IntegerType elemTy;`.
  **L2968 CN**: 执行一条独立语句或声明：`mlir::IntegerType elemTy;`。
- **L2969 EN**: Initializes variable `numElem` from the right-hand expression.
  **L2969 CN**: 使用右侧表达式初始化变量 `numElem`。
- **L2970 EN**: Executes a call or declaration centered on `builder.getIntegerType`.
  **L2970 CN**: 执行以 `builder.getIntegerType` 为核心的调用或声明。
- **L2971 EN**: Blank line separating nearby declarations or logic blocks.
  **L2971 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2972 EN**: Executes a call or declaration centered on `mlirVecTy{mlir::VectorType::get`.
  **L2972 CN**: 执行以 `mlirVecTy{mlir::VectorType::get` 为核心的调用或声明。
- **L2973 EN**: Executes a call or declaration centered on `firVecTy{fir::VectorType::get`.
  **L2973 CN**: 执行以 `firVecTy{fir::VectorType::get` 为核心的调用或声明。
- **L2974 EN**: Blank line separating nearby declarations or logic blocks.
  **L2974 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2975 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto cnv{builder.createConvert(loc, arg1TyInfo.toMlirVectorType(context),`.
  **L2975 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto cnv{builder.createConvert(loc, arg1TyInfo.toMlirVectorType(context),`。
- **L2976 EN**: Executes a standalone statement or declaration: `argBases[0])};`.
  **L2976 CN**: 执行一条独立语句或声明：`argBases[0])};`。

### Lines 2977-3000

````cpp

    mlir::Type srcTy{nullptr};
    if (numElem != arg1TyInfo.len) {
      cnv = mlir::vector::BitCastOp::create(builder, loc, mlirVecTy, cnv);
      srcTy = firVecTy;
    } else {
      srcTy = arg1TyInfo.toFirVectorType();
    }

    trg = builder.createConvert(loc, builder.getRefType(srcTy), addr);

    if (isBEVecElemOrderOnLE()) {
      cnv = reverseVectorElements(builder, loc, cnv, numElem);
    }

    src = builder.createConvert(loc, srcTy, cnv);
    break;
  }
  case VecOp::Stxv:
    src = argBases[0];
    trg = builder.createConvert(loc, builder.getRefType(argBases[0].getType()),
                                addr);
    break;
  default:
````
- **L2977 EN**: Blank line separating nearby declarations or logic blocks.
  **L2977 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2978 EN**: Executes a standalone statement or declaration: `mlir::Type srcTy{nullptr};`.
  **L2978 CN**: 执行一条独立语句或声明：`mlir::Type srcTy{nullptr};`。
- **L2979 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2979 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2980 EN**: Executes a call or declaration centered on `mlir::vector::BitCastOp::create`.
  **L2980 CN**: 执行以 `mlir::vector::BitCastOp::create` 为核心的调用或声明。
- **L2981 EN**: Executes a standalone statement or declaration: `srcTy = firVecTy;`.
  **L2981 CN**: 执行一条独立语句或声明：`srcTy = firVecTy;`。
- **L2982 EN**: Transitions from the previous branch into the alternative path.
  **L2982 CN**: 从前一个分支过渡到备选路径。
- **L2983 EN**: Executes a call or declaration centered on `arg1TyInfo.toFirVectorType`.
  **L2983 CN**: 执行以 `arg1TyInfo.toFirVectorType` 为核心的调用或声明。
- **L2984 EN**: Closes the current lexical scope or compound statement.
  **L2984 CN**: 结束当前词法作用域或复合语句块。
- **L2985 EN**: Blank line separating nearby declarations or logic blocks.
  **L2985 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2986 EN**: Executes a call or declaration centered on `builder.createConvert`.
  **L2986 CN**: 执行以 `builder.createConvert` 为核心的调用或声明。
- **L2987 EN**: Blank line separating nearby declarations or logic blocks.
  **L2987 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2988 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L2988 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2989 EN**: Executes a call or declaration centered on `reverseVectorElements`.
  **L2989 CN**: 执行以 `reverseVectorElements` 为核心的调用或声明。
- **L2990 EN**: Closes the current lexical scope or compound statement.
  **L2990 CN**: 结束当前词法作用域或复合语句块。
- **L2991 EN**: Blank line separating nearby declarations or logic blocks.
  **L2991 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2992 EN**: Executes a call or declaration centered on `builder.createConvert`.
  **L2992 CN**: 执行以 `builder.createConvert` 为核心的调用或声明。
- **L2993 EN**: Exits the nearest loop or switch statement.
  **L2993 CN**: 退出最近的循环或 switch 语句。
- **L2994 EN**: Closes the current lexical scope or compound statement.
  **L2994 CN**: 结束当前词法作用域或复合语句块。
- **L2995 EN**: Introduces a switch dispatch label: `case VecOp::Stxv:`.
  **L2995 CN**: 引入一个 switch 分发标签：`case VecOp::Stxv:`。
- **L2996 EN**: Executes a standalone statement or declaration: `src = argBases[0];`.
  **L2996 CN**: 执行一条独立语句或声明：`src = argBases[0];`。
- **L2997 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `trg = builder.createConvert(loc, builder.getRefType(argBases[0].getType()),`.
  **L2997 CN**: 继续一个多行参数列表、初始化器或聚合项：`trg = builder.createConvert(loc, builder.getRefType(argBases[0].getType()),`。
- **L2998 EN**: Executes a standalone statement or declaration: `addr);`.
  **L2998 CN**: 执行一条独立语句或声明：`addr);`。
- **L2999 EN**: Exits the nearest loop or switch statement.
  **L2999 CN**: 退出最近的循环或 switch 语句。
- **L3000 EN**: Introduces a switch dispatch label: `default:`.
  **L3000 CN**: 引入一个 switch 分发标签：`default:`。

### Lines 3001-3008

````cpp
    assert(false && "Invalid vector operation for generator");
  }
  fir::StoreOp::create(builder, loc, mlir::TypeRange{},
                       mlir::ValueRange{src, trg},
                       getAlignmentAttr(builder, 1));
}

} // namespace fir
````
- **L3001 EN**: Checks an internal invariant in debug builds.
  **L3001 CN**: 在调试构建中检查内部不变式。
- **L3002 EN**: Closes the current lexical scope or compound statement.
  **L3002 CN**: 结束当前词法作用域或复合语句块。
- **L3003 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `fir::StoreOp::create(builder, loc, mlir::TypeRange{},`.
  **L3003 CN**: 继续一个多行参数列表、初始化器或聚合项：`fir::StoreOp::create(builder, loc, mlir::TypeRange{},`。
- **L3004 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::ValueRange{src, trg},`.
  **L3004 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::ValueRange{src, trg},`。
- **L3005 EN**: Executes a call or declaration centered on `getAlignmentAttr`.
  **L3005 CN**: 执行以 `getAlignmentAttr` 为核心的调用或声明。
- **L3006 EN**: Closes the current lexical scope or compound statement.
  **L3006 CN**: 结束当前词法作用域或复合语句块。
- **L3007 EN**: Blank line separating nearby declarations or logic blocks.
  **L3007 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3008 EN**: Closes a namespace scope with a trailing comment: `} // namespace fir`.
  **L3008 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace fir`。

## Key Concepts / 关键概念

- **FIR/MLIR optimizer infrastructure / FIR/MLIR 优化器基础设施**
- **Lowering converter orchestration / lowering 转换器编排**
- **FIR dialect operations and helpers / FIR 方言操作与辅助能力**
- **MLIR IR and pass infrastructure / MLIR IR 与 Pass 基础设施**
- **IR builder orchestration / IR Builder 编排**

## Dependencies / 依赖关系

- `flang/Optimizer/Builder/PPCIntrinsicCall.h`: Provides FIR builder helpers and runtime-construction utilities. / 提供FIR Builder 辅助工具与运行时构造能力。
- `flang/Evaluate/common.h`: Provides Fortran constant-folding and evaluation facilities. / 提供Fortran 常量折叠与求值能力。
- `flang/Lower/AbstractConverter.h`: Provides lowering utilities that map semantics into FIR/MLIR. / 提供将语义映射到 FIR/MLIR 的 lowering 工具。
- `flang/Optimizer/Builder/FIRBuilder.h`: Provides FIR builder helpers and runtime-construction utilities. / 提供FIR Builder 辅助工具与运行时构造能力。
- `flang/Optimizer/Builder/MutableBox.h`: Provides FIR builder helpers and runtime-construction utilities. / 提供FIR Builder 辅助工具与运行时构造能力。
- `mlir/Dialect/Index/IR/IndexOps.h`: Provides MLIR core IR, pass, or dialect infrastructure. / 提供MLIR 核心 IR、Pass 或方言基础设施。
- `mlir/Dialect/Vector/IR/VectorOps.h`: Provides MLIR core IR, pass, or dialect infrastructure. / 提供MLIR 核心 IR、Pass 或方言基础设施。
