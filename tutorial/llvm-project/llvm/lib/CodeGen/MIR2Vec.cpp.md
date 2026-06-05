# MIR2Vec.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/CodeGen/MIR2Vec.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: This file contains `Implementation of MIR2Vec` logic inside LLVM CodeGen.
- **Purpose (CN)**: 该文件在 LLVM CodeGen 中实现与“Implementation of MIR2Vec”相关的逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

````cpp
//===- MIR2Vec.cpp - Implementation of MIR2Vec ---------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM
// Exceptions. See the LICENSE file for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
///
/// \file
/// This file implements the MIR2Vec algorithm for Machine IR embeddings.
///
//===----------------------------------------------------------------------===//

#include "llvm/CodeGen/MIR2Vec.h"
#include "llvm/ADT/DepthFirstIterator.h"
#include "llvm/ADT/Statistic.h"
#include "llvm/CodeGen/TargetInstrInfo.h"
#include "llvm/IR/Module.h"
#include "llvm/InitializePasses.h"
#include "llvm/Pass.h"
````
- **L1 EN**: Comment documents: `===- MIR2Vec.cpp - Implementation of MIR2Vec ---------------------------…`.
  **L1 CN**: 注释说明：`===- MIR2Vec.cpp - Implementation of MIR2Vec ---------------------------…`。
- **L2 EN**: Continues the surrounding comment block.
  **L2 CN**: 延续周围的注释块。
- **L3 EN**: Comment documents: `Part of the LLVM Project, under the Apache License v2.0 with LLVM`.
  **L3 CN**: 注释说明：`Part of the LLVM Project, under the Apache License v2.0 with LLVM`。
- **L4 EN**: Comment documents: `Exceptions. See the LICENSE file for license information.`.
  **L4 CN**: 注释说明：`Exceptions. See the LICENSE file for license information.`。
- **L5 EN**: Comment documents: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`.
  **L5 CN**: 注释说明：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6 EN**: Continues the surrounding comment block.
  **L6 CN**: 延续周围的注释块。
- **L7 EN**: Comment documents: `===---------------------------------------------------------------------…`.
  **L7 CN**: 注释说明：`===---------------------------------------------------------------------…`。
- **L8 EN**: Continues the surrounding comment block.
  **L8 CN**: 延续周围的注释块。
- **L9 EN**: Comment documents: `\file`.
  **L9 CN**: 注释说明：`\file`。
- **L10 EN**: Comment documents: `This file implements the MIR2Vec algorithm for Machine IR embeddings.`.
  **L10 CN**: 注释说明：`This file implements the MIR2Vec algorithm for Machine IR embeddings.`。
- **L11 EN**: Continues the surrounding comment block.
  **L11 CN**: 延续周围的注释块。
- **L12 EN**: Comment documents: `===---------------------------------------------------------------------…`.
  **L12 CN**: 注释说明：`===---------------------------------------------------------------------…`。
- **L13 EN**: Separates nearby statements for readability.
  **L13 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L14 EN**: Includes LLVM header `llvm/CodeGen/MIR2Vec.h` for MIR2Vec support.
  **L14 CN**: 引入 LLVM 头文件 `llvm/CodeGen/MIR2Vec.h`，用于 MIR2Vec 相关支持。
- **L15 EN**: Includes LLVM header `llvm/ADT/DepthFirstIterator.h` for DepthFirstIterator support.
  **L15 CN**: 引入 LLVM 头文件 `llvm/ADT/DepthFirstIterator.h`，用于 DepthFirstIterator 相关支持。
- **L16 EN**: Includes LLVM header `llvm/ADT/Statistic.h` for Statistic support.
  **L16 CN**: 引入 LLVM 头文件 `llvm/ADT/Statistic.h`，用于 Statistic 相关支持。
- **L17 EN**: Includes LLVM header `llvm/CodeGen/TargetInstrInfo.h` for TargetInstrInfo support.
  **L17 CN**: 引入 LLVM 头文件 `llvm/CodeGen/TargetInstrInfo.h`，用于 TargetInstrInfo 相关支持。
- **L18 EN**: Includes LLVM header `llvm/IR/Module.h` for Module support.
  **L18 CN**: 引入 LLVM 头文件 `llvm/IR/Module.h`，用于 Module 相关支持。
- **L19 EN**: Includes LLVM header `llvm/InitializePasses.h` for InitializePasses support.
  **L19 CN**: 引入 LLVM 头文件 `llvm/InitializePasses.h`，用于 InitializePasses 相关支持。
- **L20 EN**: Includes LLVM header `llvm/Pass.h` for Pass support.
  **L20 CN**: 引入 LLVM 头文件 `llvm/Pass.h`，用于 Pass 相关支持。

### Lines 21-40

````cpp
#include "llvm/Support/Errc.h"
#include "llvm/Support/MemoryBuffer.h"
#include "llvm/Support/Regex.h"

using namespace llvm;
using namespace mir2vec;

#define DEBUG_TYPE "mir2vec"

STATISTIC(MIRVocabMissCounter,
          "Number of lookups to MIR entities not present in the vocabulary");

namespace llvm {
namespace mir2vec {
cl::OptionCategory MIR2VecCategory("MIR2Vec Options");

// FIXME: Use a default vocab when not specified
static cl::opt<std::string>
    VocabFile("mir2vec-vocab-path", cl::Optional,
              cl::desc("Path to the vocabulary file for MIR2Vec"), cl::init(""),
````
- **L21 EN**: Includes LLVM header `llvm/Support/Errc.h` for Errc support.
  **L21 CN**: 引入 LLVM 头文件 `llvm/Support/Errc.h`，用于 Errc 相关支持。
- **L22 EN**: Includes LLVM header `llvm/Support/MemoryBuffer.h` for MemoryBuffer support.
  **L22 CN**: 引入 LLVM 头文件 `llvm/Support/MemoryBuffer.h`，用于 MemoryBuffer 相关支持。
- **L23 EN**: Includes LLVM header `llvm/Support/Regex.h` for Regex support.
  **L23 CN**: 引入 LLVM 头文件 `llvm/Support/Regex.h`，用于 Regex 相关支持。
- **L24 EN**: Separates nearby statements for readability.
  **L24 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L25 EN**: Imports namespace `llvm` into this translation unit.
  **L25 CN**: 将命名空间 `llvm` 引入当前编译单元。
- **L26 EN**: Imports namespace `mir2vec` into this translation unit.
  **L26 CN**: 将命名空间 `mir2vec` 引入当前编译单元。
- **L27 EN**: Separates nearby statements for readability.
  **L27 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L28 EN**: Defines the LLVM debug channel used by this file.
  **L28 CN**: 定义该文件使用的 LLVM 调试通道。
- **L29 EN**: Separates nearby statements for readability.
  **L29 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L30 EN**: Registers a pass statistic counter.
  **L30 CN**: 注册一个 pass 统计计数器。
- **L31 EN**: Executes statement `"Number of lookups to MIR entities not present in the vocabulary");`.
  **L31 CN**: 执行语句 `"Number of lookups to MIR entities not present in the vocabulary");`。
- **L32 EN**: Separates nearby statements for readability.
  **L32 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L33 EN**: Opens namespace `llvm`.
  **L33 CN**: 打开命名空间 `llvm`。
- **L34 EN**: Opens namespace `mir2vec`.
  **L34 CN**: 打开命名空间 `mir2vec`。
- **L35 EN**: Declares function or method `MIR2VecCategory`.
  **L35 CN**: 声明函数或方法 `MIR2VecCategory`。
- **L36 EN**: Separates nearby statements for readability.
  **L36 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L37 EN**: Comment documents: `FIXME: Use a default vocab when not specified`.
  **L37 CN**: 注释说明：`FIXME: Use a default vocab when not specified`。
- **L38 EN**: Declares LLVM command-line option `command-line option`.
  **L38 CN**: 声明 LLVM 命令行选项 `command-line option`。
- **L39 EN**: Continues logic with `VocabFile("mir2vec-vocab-path", cl::Optional,`.
  **L39 CN**: 继续处理逻辑：`VocabFile("mir2vec-vocab-path", cl::Optional,`。
- **L40 EN**: Provides part of the signature for `desc`.
  **L40 CN**: 给出 `desc` 的一部分签名。

### Lines 41-60

````cpp
              cl::cat(MIR2VecCategory));
cl::opt<float> OpcWeight("mir2vec-opc-weight", cl::Optional, cl::init(1.0),
                         cl::desc("Weight for machine opcode embeddings"),
                         cl::cat(MIR2VecCategory));
cl::opt<float> CommonOperandWeight(
    "mir2vec-common-operand-weight", cl::Optional, cl::init(1.0),
    cl::desc("Weight for common operand embeddings"), cl::cat(MIR2VecCategory));
cl::opt<float>
    RegOperandWeight("mir2vec-reg-operand-weight", cl::Optional, cl::init(1.0),
                     cl::desc("Weight for register operand embeddings"),
                     cl::cat(MIR2VecCategory));
cl::opt<MIR2VecKind> MIR2VecEmbeddingKind(
    "mir2vec-kind", cl::Optional,
    cl::values(clEnumValN(MIR2VecKind::Symbolic, "symbolic",
                          "Generate symbolic embeddings for MIR")),
    cl::init(MIR2VecKind::Symbolic), cl::desc("MIR2Vec embedding kind"),
    cl::cat(MIR2VecCategory));

static cl::opt<bool> PrintAllVocabEntries(
    "mir2vec-print-all-vocab-entries", cl::Optional, cl::init(false),
````
- **L41 EN**: Declares function or method `cat`.
  **L41 CN**: 声明函数或方法 `cat`。
- **L42 EN**: Declares LLVM command-line option `mir2vec-opc-weight`.
  **L42 CN**: 声明 LLVM 命令行选项 `mir2vec-opc-weight`。
- **L43 EN**: Provides part of the signature for `desc`.
  **L43 CN**: 给出 `desc` 的一部分签名。
- **L44 EN**: Declares function or method `cat`.
  **L44 CN**: 声明函数或方法 `cat`。
- **L45 EN**: Declares LLVM command-line option `command-line option`.
  **L45 CN**: 声明 LLVM 命令行选项 `command-line option`。
- **L46 EN**: Provides part of the signature for `init`.
  **L46 CN**: 给出 `init` 的一部分签名。
- **L47 EN**: Declares function or method `desc`.
  **L47 CN**: 声明函数或方法 `desc`。
- **L48 EN**: Declares LLVM command-line option `command-line option`.
  **L48 CN**: 声明 LLVM 命令行选项 `command-line option`。
- **L49 EN**: Provides part of the signature for `RegOperandWeight`.
  **L49 CN**: 给出 `RegOperandWeight` 的一部分签名。
- **L50 EN**: Provides part of the signature for `desc`.
  **L50 CN**: 给出 `desc` 的一部分签名。
- **L51 EN**: Declares function or method `cat`.
  **L51 CN**: 声明函数或方法 `cat`。
- **L52 EN**: Declares LLVM command-line option `command-line option`.
  **L52 CN**: 声明 LLVM 命令行选项 `command-line option`。
- **L53 EN**: Continues logic with `"mir2vec-kind", cl::Optional,`.
  **L53 CN**: 继续处理逻辑：`"mir2vec-kind", cl::Optional,`。
- **L54 EN**: Provides part of the signature for `values`.
  **L54 CN**: 给出 `values` 的一部分签名。
- **L55 EN**: Continues logic with `"Generate symbolic embeddings for MIR")),`.
  **L55 CN**: 继续处理逻辑：`"Generate symbolic embeddings for MIR")),`。
- **L56 EN**: Provides part of the signature for `init`.
  **L56 CN**: 给出 `init` 的一部分签名。
- **L57 EN**: Declares function or method `cat`.
  **L57 CN**: 声明函数或方法 `cat`。
- **L58 EN**: Separates nearby statements for readability.
  **L58 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L59 EN**: Declares LLVM command-line option `command-line option`.
  **L59 CN**: 声明 LLVM 命令行选项 `command-line option`。
- **L60 EN**: Provides part of the signature for `init`.
  **L60 CN**: 给出 `init` 的一部分签名。

### Lines 61-80

````cpp
    cl::desc("Print all vocabulary entries including zero embeddings"),
    cl::cat(MIR2VecCategory));

} // namespace mir2vec
} // namespace llvm

//===----------------------------------------------------------------------===//
// Vocabulary
//===----------------------------------------------------------------------===//

MIRVocabulary::MIRVocabulary(VocabMap &&OpcodeMap, VocabMap &&CommonOperandMap,
                             VocabMap &&PhysicalRegisterMap,
                             VocabMap &&VirtualRegisterMap,
                             const TargetInstrInfo &TII,
                             const TargetRegisterInfo &TRI,
                             const MachineRegisterInfo &MRI)
    : TII(TII), TRI(TRI), MRI(MRI) {
  buildCanonicalOpcodeMapping();
  unsigned CanonicalOpcodeCount = UniqueBaseOpcodeNames.size();
  assert(CanonicalOpcodeCount > 0 &&
````
- **L61 EN**: Provides part of the signature for `desc`.
  **L61 CN**: 给出 `desc` 的一部分签名。
- **L62 EN**: Declares function or method `cat`.
  **L62 CN**: 声明函数或方法 `cat`。
- **L63 EN**: Separates nearby statements for readability.
  **L63 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L64 EN**: Continues logic with `} // namespace mir2vec`.
  **L64 CN**: 继续处理逻辑：`} // namespace mir2vec`。
- **L65 EN**: Continues logic with `} // namespace llvm`.
  **L65 CN**: 继续处理逻辑：`} // namespace llvm`。
- **L66 EN**: Separates nearby statements for readability.
  **L66 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L67 EN**: Comment documents: `===---------------------------------------------------------------------…`.
  **L67 CN**: 注释说明：`===---------------------------------------------------------------------…`。
- **L68 EN**: Comment documents: `Vocabulary`.
  **L68 CN**: 注释说明：`Vocabulary`。
- **L69 EN**: Comment documents: `===---------------------------------------------------------------------…`.
  **L69 CN**: 注释说明：`===---------------------------------------------------------------------…`。
- **L70 EN**: Separates nearby statements for readability.
  **L70 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L71 EN**: Provides part of the signature for `MIRVocabulary`.
  **L71 CN**: 给出 `MIRVocabulary` 的一部分签名。
- **L72 EN**: Continues logic with `VocabMap &&PhysicalRegisterMap,`.
  **L72 CN**: 继续处理逻辑：`VocabMap &&PhysicalRegisterMap,`。
- **L73 EN**: Continues logic with `VocabMap &&VirtualRegisterMap,`.
  **L73 CN**: 继续处理逻辑：`VocabMap &&VirtualRegisterMap,`。
- **L74 EN**: Continues logic with `const TargetInstrInfo &TII,`.
  **L74 CN**: 继续处理逻辑：`const TargetInstrInfo &TII,`。
- **L75 EN**: Continues logic with `const TargetRegisterInfo &TRI,`.
  **L75 CN**: 继续处理逻辑：`const TargetRegisterInfo &TRI,`。
- **L76 EN**: Continues logic with `const MachineRegisterInfo &MRI)`.
  **L76 CN**: 继续处理逻辑：`const MachineRegisterInfo &MRI)`。
- **L77 EN**: Begins the definition of `TII`.
  **L77 CN**: 开始定义 `TII`。
- **L78 EN**: Executes statement `buildCanonicalOpcodeMapping();`.
  **L78 CN**: 执行语句 `buildCanonicalOpcodeMapping();`。
- **L79 EN**: Assigns or initializes `unsigned CanonicalOpcodeCount`.
  **L79 CN**: 对 `unsigned CanonicalOpcodeCount` 进行赋值或初始化。
- **L80 EN**: Checks an invariant in debug builds.
  **L80 CN**: 在调试构建中检查一个不变量。

### Lines 81-100

````cpp
         "No canonical opcodes found for target - invalid vocabulary");

  buildRegisterOperandMapping();

  // Define layout of vocabulary sections
  Layout.OpcodeBase = 0;
  Layout.CommonOperandBase = CanonicalOpcodeCount;
  // We expect same classes for physical and virtual registers
  Layout.PhyRegBase = Layout.CommonOperandBase + std::size(CommonOperandNames);
  Layout.VirtRegBase = Layout.PhyRegBase + RegisterOperandNames.size();

  generateStorage(OpcodeMap, CommonOperandMap, PhysicalRegisterMap,
                  VirtualRegisterMap);
  Layout.TotalEntries = Storage.size();
}

Expected<MIRVocabulary>
MIRVocabulary::create(VocabMap &&OpcodeMap, VocabMap &&CommonOperandMap,
                      VocabMap &&PhyRegMap, VocabMap &&VirtRegMap,
                      const TargetInstrInfo &TII, const TargetRegisterInfo &TRI,
````
- **L81 EN**: Executes statement `"No canonical opcodes found for target - invalid vocabulary");`.
  **L81 CN**: 执行语句 `"No canonical opcodes found for target - invalid vocabulary");`。
- **L82 EN**: Separates nearby statements for readability.
  **L82 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L83 EN**: Executes statement `buildRegisterOperandMapping();`.
  **L83 CN**: 执行语句 `buildRegisterOperandMapping();`。
- **L84 EN**: Separates nearby statements for readability.
  **L84 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L85 EN**: Comment documents: `Define layout of vocabulary sections`.
  **L85 CN**: 注释说明：`Define layout of vocabulary sections`。
- **L86 EN**: Assigns or initializes `Layout.OpcodeBase`.
  **L86 CN**: 对 `Layout.OpcodeBase` 进行赋值或初始化。
- **L87 EN**: Assigns or initializes `Layout.CommonOperandBase`.
  **L87 CN**: 对 `Layout.CommonOperandBase` 进行赋值或初始化。
- **L88 EN**: Comment documents: `We expect same classes for physical and virtual registers`.
  **L88 CN**: 注释说明：`We expect same classes for physical and virtual registers`。
- **L89 EN**: Declares function or method `size`.
  **L89 CN**: 声明函数或方法 `size`。
- **L90 EN**: Assigns or initializes `Layout.VirtRegBase`.
  **L90 CN**: 对 `Layout.VirtRegBase` 进行赋值或初始化。
- **L91 EN**: Separates nearby statements for readability.
  **L91 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L92 EN**: Continues logic with `generateStorage(OpcodeMap, CommonOperandMap, PhysicalRegisterMap,`.
  **L92 CN**: 继续处理逻辑：`generateStorage(OpcodeMap, CommonOperandMap, PhysicalRegisterMap,`。
- **L93 EN**: Executes statement `VirtualRegisterMap);`.
  **L93 CN**: 执行语句 `VirtualRegisterMap);`。
- **L94 EN**: Assigns or initializes `Layout.TotalEntries`.
  **L94 CN**: 对 `Layout.TotalEntries` 进行赋值或初始化。
- **L95 EN**: Closes the current scope.
  **L95 CN**: 关闭当前作用域。
- **L96 EN**: Separates nearby statements for readability.
  **L96 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L97 EN**: Continues logic with `Expected<MIRVocabulary>`.
  **L97 CN**: 继续处理逻辑：`Expected<MIRVocabulary>`。
- **L98 EN**: Provides part of the signature for `create`.
  **L98 CN**: 给出 `create` 的一部分签名。
- **L99 EN**: Continues logic with `VocabMap &&PhyRegMap, VocabMap &&VirtRegMap,`.
  **L99 CN**: 继续处理逻辑：`VocabMap &&PhyRegMap, VocabMap &&VirtRegMap,`。
- **L100 EN**: Continues logic with `const TargetInstrInfo &TII, const TargetRegisterInfo &TRI,`.
  **L100 CN**: 继续处理逻辑：`const TargetInstrInfo &TII, const TargetRegisterInfo &TRI,`。

### Lines 101-120

````cpp
                      const MachineRegisterInfo &MRI) {
  if (OpcodeMap.empty() || CommonOperandMap.empty() || PhyRegMap.empty() ||
      VirtRegMap.empty())
    return createStringError(errc::invalid_argument,
                             "Empty vocabulary entries provided");

  MIRVocabulary Vocab(std::move(OpcodeMap), std::move(CommonOperandMap),
                      std::move(PhyRegMap), std::move(VirtRegMap), TII, TRI,
                      MRI);

  // Validate Storage after construction
  if (!Vocab.Storage.isValid())
    return createStringError(errc::invalid_argument,
                             "Failed to create valid vocabulary storage");
  Vocab.ZeroEmbedding = Embedding(Vocab.Storage.getDimension(), 0.0);
  return std::move(Vocab);
}

std::string MIRVocabulary::extractBaseOpcodeName(StringRef InstrName) {
  // Extract base instruction name using regex to capture letters and
````
- **L101 EN**: Starts block `const MachineRegisterInfo &MRI)`.
  **L101 CN**: 开始代码块 `const MachineRegisterInfo &MRI)`。
- **L102 EN**: Begins a conditional branch.
  **L102 CN**: 开始一个条件分支。
- **L103 EN**: Continues logic with `VirtRegMap.empty())`.
  **L103 CN**: 继续处理逻辑：`VirtRegMap.empty())`。
- **L104 EN**: Returns `createStringError(errc::invalid_argument,` to the caller.
  **L104 CN**: 向调用者返回 `createStringError(errc::invalid_argument,`。
- **L105 EN**: Executes statement `"Empty vocabulary entries provided");`.
  **L105 CN**: 执行语句 `"Empty vocabulary entries provided");`。
- **L106 EN**: Separates nearby statements for readability.
  **L106 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L107 EN**: Provides part of the signature for `Vocab`.
  **L107 CN**: 给出 `Vocab` 的一部分签名。
- **L108 EN**: Provides part of the signature for `move`.
  **L108 CN**: 给出 `move` 的一部分签名。
- **L109 EN**: Executes statement `MRI);`.
  **L109 CN**: 执行语句 `MRI);`。
- **L110 EN**: Separates nearby statements for readability.
  **L110 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L111 EN**: Comment documents: `Validate Storage after construction`.
  **L111 CN**: 注释说明：`Validate Storage after construction`。
- **L112 EN**: Begins a conditional branch.
  **L112 CN**: 开始一个条件分支。
- **L113 EN**: Returns `createStringError(errc::invalid_argument,` to the caller.
  **L113 CN**: 向调用者返回 `createStringError(errc::invalid_argument,`。
- **L114 EN**: Executes statement `"Failed to create valid vocabulary storage");`.
  **L114 CN**: 执行语句 `"Failed to create valid vocabulary storage");`。
- **L115 EN**: Assigns or initializes `Vocab.ZeroEmbedding`.
  **L115 CN**: 对 `Vocab.ZeroEmbedding` 进行赋值或初始化。
- **L116 EN**: Returns `std::move(Vocab)` to the caller.
  **L116 CN**: 向调用者返回 `std::move(Vocab)`。
- **L117 EN**: Closes the current scope.
  **L117 CN**: 关闭当前作用域。
- **L118 EN**: Separates nearby statements for readability.
  **L118 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L119 EN**: Begins the definition of `extractBaseOpcodeName`.
  **L119 CN**: 开始定义 `extractBaseOpcodeName`。
- **L120 EN**: Comment documents: `Extract base instruction name using regex to capture letters and`.
  **L120 CN**: 注释说明：`Extract base instruction name using regex to capture letters and`。

### Lines 121-140

````cpp
  // underscores Examples: "ADD32rr" -> "ADD", "ARITH_FENCE" -> "ARITH_FENCE"
  //
  // TODO: Consider more sophisticated extraction:
  // - Handle complex prefixes like "AVX1_SETALLONES" correctly (Currently, it
  // would naively map to "AVX")
  // - Extract width suffixes (8,16,32,64) as separate features
  // - Capture addressing mode suffixes (r,i,m,ri,etc.) for better analysis
  // (Currently, instances like "MOV32mi" map to "MOV", but "ADDPDrr" would map
  // to "ADDPDrr")

  assert(!InstrName.empty() && "Instruction name should not be empty");

  // Use regex to extract initial sequence of letters and underscores
  static const Regex BaseOpcodeRegex("([a-zA-Z_]+)");
  SmallVector<StringRef, 2> Matches;

  if (BaseOpcodeRegex.match(InstrName, &Matches) && Matches.size() > 1) {
    StringRef Match = Matches[1];
    // Trim trailing underscores
    while (!Match.empty() && Match.back() == '_')
````
- **L121 EN**: Comment documents: `underscores Examples: "ADD32rr" -> "ADD", "ARITH_FENCE" -> "ARITH_FENCE"`.
  **L121 CN**: 注释说明：`underscores Examples: "ADD32rr" -> "ADD", "ARITH_FENCE" -> "ARITH_FENCE"`。
- **L122 EN**: Continues the surrounding comment block.
  **L122 CN**: 延续周围的注释块。
- **L123 EN**: Comment documents: `TODO: Consider more sophisticated extraction:`.
  **L123 CN**: 注释说明：`TODO: Consider more sophisticated extraction:`。
- **L124 EN**: Comment documents: `- Handle complex prefixes like "AVX1_SETALLONES" correctly (Currently, i…`.
  **L124 CN**: 注释说明：`- Handle complex prefixes like "AVX1_SETALLONES" correctly (Currently, i…`。
- **L125 EN**: Comment documents: `would naively map to "AVX")`.
  **L125 CN**: 注释说明：`would naively map to "AVX")`。
- **L126 EN**: Comment documents: `- Extract width suffixes (8,16,32,64) as separate features`.
  **L126 CN**: 注释说明：`- Extract width suffixes (8,16,32,64) as separate features`。
- **L127 EN**: Comment documents: `- Capture addressing mode suffixes (r,i,m,ri,etc.) for better analysis`.
  **L127 CN**: 注释说明：`- Capture addressing mode suffixes (r,i,m,ri,etc.) for better analysis`。
- **L128 EN**: Comment documents: `(Currently, instances like "MOV32mi" map to "MOV", but "ADDPDrr" would m…`.
  **L128 CN**: 注释说明：`(Currently, instances like "MOV32mi" map to "MOV", but "ADDPDrr" would m…`。
- **L129 EN**: Comment documents: `to "ADDPDrr")`.
  **L129 CN**: 注释说明：`to "ADDPDrr")`。
- **L130 EN**: Separates nearby statements for readability.
  **L130 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L131 EN**: Checks an invariant in debug builds.
  **L131 CN**: 在调试构建中检查一个不变量。
- **L132 EN**: Separates nearby statements for readability.
  **L132 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L133 EN**: Comment documents: `Use regex to extract initial sequence of letters and underscores`.
  **L133 CN**: 注释说明：`Use regex to extract initial sequence of letters and underscores`。
- **L134 EN**: Declares function or method `BaseOpcodeRegex`.
  **L134 CN**: 声明函数或方法 `BaseOpcodeRegex`。
- **L135 EN**: Executes statement `SmallVector<StringRef, 2> Matches;`.
  **L135 CN**: 执行语句 `SmallVector<StringRef, 2> Matches;`。
- **L136 EN**: Separates nearby statements for readability.
  **L136 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L137 EN**: Begins a conditional branch.
  **L137 CN**: 开始一个条件分支。
- **L138 EN**: Assigns or initializes `StringRef Match`.
  **L138 CN**: 对 `StringRef Match` 进行赋值或初始化。
- **L139 EN**: Comment documents: `Trim trailing underscores`.
  **L139 CN**: 注释说明：`Trim trailing underscores`。
- **L140 EN**: Starts a while loop controlled by a condition.
  **L140 CN**: 开始一个由条件控制的 while 循环。

### Lines 141-160

````cpp
      Match = Match.drop_back();
    return Match.str();
  }

  // Fallback to original name if no pattern matches
  return InstrName.str();
}

unsigned MIRVocabulary::getCanonicalIndexForBaseName(StringRef BaseName) const {
  assert(!UniqueBaseOpcodeNames.empty() && "Canonical mapping not built");
  auto It = std::find(UniqueBaseOpcodeNames.begin(),
                      UniqueBaseOpcodeNames.end(), BaseName.str());
  assert(It != UniqueBaseOpcodeNames.end() &&
         "Base name not found in unique opcodes");
  return std::distance(UniqueBaseOpcodeNames.begin(), It);
}

unsigned MIRVocabulary::getCanonicalOpcodeIndex(unsigned Opcode) const {
  auto BaseOpcode = extractBaseOpcodeName(TII.getName(Opcode));
  return getCanonicalIndexForBaseName(BaseOpcode);
````
- **L141 EN**: Assigns or initializes `Match`.
  **L141 CN**: 对 `Match` 进行赋值或初始化。
- **L142 EN**: Returns `Match.str()` to the caller.
  **L142 CN**: 向调用者返回 `Match.str()`。
- **L143 EN**: Closes the current scope.
  **L143 CN**: 关闭当前作用域。
- **L144 EN**: Separates nearby statements for readability.
  **L144 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L145 EN**: Comment documents: `Fallback to original name if no pattern matches`.
  **L145 CN**: 注释说明：`Fallback to original name if no pattern matches`。
- **L146 EN**: Returns `InstrName.str()` to the caller.
  **L146 CN**: 向调用者返回 `InstrName.str()`。
- **L147 EN**: Closes the current scope.
  **L147 CN**: 关闭当前作用域。
- **L148 EN**: Separates nearby statements for readability.
  **L148 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L149 EN**: Begins the definition of `getCanonicalIndexForBaseName`.
  **L149 CN**: 开始定义 `getCanonicalIndexForBaseName`。
- **L150 EN**: Checks an invariant in debug builds.
  **L150 CN**: 在调试构建中检查一个不变量。
- **L151 EN**: Provides part of the signature for `find`.
  **L151 CN**: 给出 `find` 的一部分签名。
- **L152 EN**: Executes statement `UniqueBaseOpcodeNames.end(), BaseName.str());`.
  **L152 CN**: 执行语句 `UniqueBaseOpcodeNames.end(), BaseName.str());`。
- **L153 EN**: Checks an invariant in debug builds.
  **L153 CN**: 在调试构建中检查一个不变量。
- **L154 EN**: Executes statement `"Base name not found in unique opcodes");`.
  **L154 CN**: 执行语句 `"Base name not found in unique opcodes");`。
- **L155 EN**: Returns `std::distance(UniqueBaseOpcodeNames.begin(), It)` to the caller.
  **L155 CN**: 向调用者返回 `std::distance(UniqueBaseOpcodeNames.begin(), It)`。
- **L156 EN**: Closes the current scope.
  **L156 CN**: 关闭当前作用域。
- **L157 EN**: Separates nearby statements for readability.
  **L157 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L158 EN**: Begins the definition of `getCanonicalOpcodeIndex`.
  **L158 CN**: 开始定义 `getCanonicalOpcodeIndex`。
- **L159 EN**: Assigns or initializes `auto BaseOpcode`.
  **L159 CN**: 对 `auto BaseOpcode` 进行赋值或初始化。
- **L160 EN**: Returns `getCanonicalIndexForBaseName(BaseOpcode)` to the caller.
  **L160 CN**: 向调用者返回 `getCanonicalIndexForBaseName(BaseOpcode)`。

### Lines 161-180

````cpp
}

unsigned
MIRVocabulary::getCanonicalIndexForOperandName(StringRef OperandName) const {
  auto It = std::find(std::begin(CommonOperandNames),
                      std::end(CommonOperandNames), OperandName);
  assert(It != std::end(CommonOperandNames) &&
         "Operand name not found in common operands");
  return Layout.CommonOperandBase +
         std::distance(std::begin(CommonOperandNames), It);
}

unsigned
MIRVocabulary::getCanonicalIndexForRegisterClass(StringRef RegName,
                                                 bool IsPhysical) const {
  auto It = std::find(RegisterOperandNames.begin(), RegisterOperandNames.end(),
                      RegName);
  assert(It != RegisterOperandNames.end() &&
         "Register name not found in register operands");
  unsigned LocalIndex = std::distance(RegisterOperandNames.begin(), It);
````
- **L161 EN**: Closes the current scope.
  **L161 CN**: 关闭当前作用域。
- **L162 EN**: Separates nearby statements for readability.
  **L162 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L163 EN**: Continues logic with `unsigned`.
  **L163 CN**: 继续处理逻辑：`unsigned`。
- **L164 EN**: Begins the definition of `getCanonicalIndexForOperandName`.
  **L164 CN**: 开始定义 `getCanonicalIndexForOperandName`。
- **L165 EN**: Provides part of the signature for `find`.
  **L165 CN**: 给出 `find` 的一部分签名。
- **L166 EN**: Declares function or method `end`.
  **L166 CN**: 声明函数或方法 `end`。
- **L167 EN**: Checks an invariant in debug builds.
  **L167 CN**: 在调试构建中检查一个不变量。
- **L168 EN**: Executes statement `"Operand name not found in common operands");`.
  **L168 CN**: 执行语句 `"Operand name not found in common operands");`。
- **L169 EN**: Returns `Layout.CommonOperandBase +` to the caller.
  **L169 CN**: 向调用者返回 `Layout.CommonOperandBase +`。
- **L170 EN**: Declares function or method `distance`.
  **L170 CN**: 声明函数或方法 `distance`。
- **L171 EN**: Closes the current scope.
  **L171 CN**: 关闭当前作用域。
- **L172 EN**: Separates nearby statements for readability.
  **L172 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L173 EN**: Continues logic with `unsigned`.
  **L173 CN**: 继续处理逻辑：`unsigned`。
- **L174 EN**: Provides part of the signature for `getCanonicalIndexForRegisterClass`.
  **L174 CN**: 给出 `getCanonicalIndexForRegisterClass` 的一部分签名。
- **L175 EN**: Starts block `bool IsPhysical) const`.
  **L175 CN**: 开始代码块 `bool IsPhysical) const`。
- **L176 EN**: Provides part of the signature for `find`.
  **L176 CN**: 给出 `find` 的一部分签名。
- **L177 EN**: Executes statement `RegName);`.
  **L177 CN**: 执行语句 `RegName);`。
- **L178 EN**: Checks an invariant in debug builds.
  **L178 CN**: 在调试构建中检查一个不变量。
- **L179 EN**: Executes statement `"Register name not found in register operands");`.
  **L179 CN**: 执行语句 `"Register name not found in register operands");`。
- **L180 EN**: Declares function or method `distance`.
  **L180 CN**: 声明函数或方法 `distance`。

### Lines 181-200

````cpp
  return (IsPhysical ? Layout.PhyRegBase : Layout.VirtRegBase) + LocalIndex;
}

std::string MIRVocabulary::getStringKey(unsigned Pos) const {
  assert(Pos < Layout.TotalEntries && "Position out of bounds in vocabulary");

  // Handle opcodes section
  if (Pos < Layout.CommonOperandBase) {
    // Convert canonical index back to base opcode name
    auto It = UniqueBaseOpcodeNames.begin();
    std::advance(It, Pos);
    assert(It != UniqueBaseOpcodeNames.end() &&
           "Canonical index out of bounds in opcode section");
    return *It;
  }

  auto getLocalIndex = [](unsigned Pos, size_t BaseOffset, size_t Bound,
                          const char *Msg) {
    unsigned LocalIndex = Pos - BaseOffset;
    assert(LocalIndex < Bound && Msg);
````
- **L181 EN**: Returns `(IsPhysical ? Layout.PhyRegBase : Layout.VirtRegBase) + LocalIndex` to the caller.
  **L181 CN**: 向调用者返回 `(IsPhysical ? Layout.PhyRegBase : Layout.VirtRegBase) + LocalIndex`。
- **L182 EN**: Closes the current scope.
  **L182 CN**: 关闭当前作用域。
- **L183 EN**: Separates nearby statements for readability.
  **L183 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L184 EN**: Begins the definition of `getStringKey`.
  **L184 CN**: 开始定义 `getStringKey`。
- **L185 EN**: Checks an invariant in debug builds.
  **L185 CN**: 在调试构建中检查一个不变量。
- **L186 EN**: Separates nearby statements for readability.
  **L186 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L187 EN**: Comment documents: `Handle opcodes section`.
  **L187 CN**: 注释说明：`Handle opcodes section`。
- **L188 EN**: Begins a conditional branch.
  **L188 CN**: 开始一个条件分支。
- **L189 EN**: Comment documents: `Convert canonical index back to base opcode name`.
  **L189 CN**: 注释说明：`Convert canonical index back to base opcode name`。
- **L190 EN**: Assigns or initializes `auto It`.
  **L190 CN**: 对 `auto It` 进行赋值或初始化。
- **L191 EN**: Declares function or method `advance`.
  **L191 CN**: 声明函数或方法 `advance`。
- **L192 EN**: Checks an invariant in debug builds.
  **L192 CN**: 在调试构建中检查一个不变量。
- **L193 EN**: Executes statement `"Canonical index out of bounds in opcode section");`.
  **L193 CN**: 执行语句 `"Canonical index out of bounds in opcode section");`。
- **L194 EN**: Returns `*It` to the caller.
  **L194 CN**: 向调用者返回 `*It`。
- **L195 EN**: Closes the current scope.
  **L195 CN**: 关闭当前作用域。
- **L196 EN**: Separates nearby statements for readability.
  **L196 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L197 EN**: Continues logic with `auto getLocalIndex = [](unsigned Pos, size_t BaseOffset, size_t Bound,`.
  **L197 CN**: 继续处理逻辑：`auto getLocalIndex = [](unsigned Pos, size_t BaseOffset, size_t Bound,`。
- **L198 EN**: Starts block `const char *Msg)`.
  **L198 CN**: 开始代码块 `const char *Msg)`。
- **L199 EN**: Assigns or initializes `unsigned LocalIndex`.
  **L199 CN**: 对 `unsigned LocalIndex` 进行赋值或初始化。
- **L200 EN**: Checks an invariant in debug builds.
  **L200 CN**: 在调试构建中检查一个不变量。

### Lines 201-220

````cpp
    return LocalIndex;
  };

  // Handle common operands section
  if (Pos < Layout.PhyRegBase) {
    unsigned LocalIndex = getLocalIndex(
        Pos, Layout.CommonOperandBase, std::size(CommonOperandNames),
        "Local index out of bounds in common operands");
    return CommonOperandNames[LocalIndex].str();
  }

  // Handle physical registers section
  if (Pos < Layout.VirtRegBase) {
    unsigned LocalIndex =
        getLocalIndex(Pos, Layout.PhyRegBase, RegisterOperandNames.size(),
                      "Local index out of bounds in physical registers");
    return "PhyReg_" + RegisterOperandNames[LocalIndex];
  }

  // Handle virtual registers section
````
- **L201 EN**: Returns `LocalIndex` to the caller.
  **L201 CN**: 向调用者返回 `LocalIndex`。
- **L202 EN**: Closes the current scope.
  **L202 CN**: 关闭当前作用域。
- **L203 EN**: Separates nearby statements for readability.
  **L203 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L204 EN**: Comment documents: `Handle common operands section`.
  **L204 CN**: 注释说明：`Handle common operands section`。
- **L205 EN**: Begins a conditional branch.
  **L205 CN**: 开始一个条件分支。
- **L206 EN**: Continues logic with `unsigned LocalIndex = getLocalIndex(`.
  **L206 CN**: 继续处理逻辑：`unsigned LocalIndex = getLocalIndex(`。
- **L207 EN**: Provides part of the signature for `size`.
  **L207 CN**: 给出 `size` 的一部分签名。
- **L208 EN**: Executes statement `"Local index out of bounds in common operands");`.
  **L208 CN**: 执行语句 `"Local index out of bounds in common operands");`。
- **L209 EN**: Returns `CommonOperandNames[LocalIndex].str()` to the caller.
  **L209 CN**: 向调用者返回 `CommonOperandNames[LocalIndex].str()`。
- **L210 EN**: Closes the current scope.
  **L210 CN**: 关闭当前作用域。
- **L211 EN**: Separates nearby statements for readability.
  **L211 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L212 EN**: Comment documents: `Handle physical registers section`.
  **L212 CN**: 注释说明：`Handle physical registers section`。
- **L213 EN**: Begins a conditional branch.
  **L213 CN**: 开始一个条件分支。
- **L214 EN**: Continues logic with `unsigned LocalIndex =`.
  **L214 CN**: 继续处理逻辑：`unsigned LocalIndex =`。
- **L215 EN**: Continues logic with `getLocalIndex(Pos, Layout.PhyRegBase, RegisterOperandNames.size(),`.
  **L215 CN**: 继续处理逻辑：`getLocalIndex(Pos, Layout.PhyRegBase, RegisterOperandNames.size(),`。
- **L216 EN**: Executes statement `"Local index out of bounds in physical registers");`.
  **L216 CN**: 执行语句 `"Local index out of bounds in physical registers");`。
- **L217 EN**: Returns `"PhyReg_" + RegisterOperandNames[LocalIndex]` to the caller.
  **L217 CN**: 向调用者返回 `"PhyReg_" + RegisterOperandNames[LocalIndex]`。
- **L218 EN**: Closes the current scope.
  **L218 CN**: 关闭当前作用域。
- **L219 EN**: Separates nearby statements for readability.
  **L219 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L220 EN**: Comment documents: `Handle virtual registers section`.
  **L220 CN**: 注释说明：`Handle virtual registers section`。

### Lines 221-240

````cpp
  unsigned LocalIndex =
      getLocalIndex(Pos, Layout.VirtRegBase, RegisterOperandNames.size(),
                    "Local index out of bounds in virtual registers");
  return "VirtReg_" + RegisterOperandNames[LocalIndex];
}

void MIRVocabulary::generateStorage(const VocabMap &OpcodeMap,
                                    const VocabMap &CommonOperandsMap,
                                    const VocabMap &PhyRegMap,
                                    const VocabMap &VirtRegMap) {

  // Helper for handling missing entities in the vocabulary.
  // Currently, we use a zero vector. In the future, we will throw an error to
  // ensure that *all* known entities are present in the vocabulary.
  auto handleMissingEntity = [](StringRef Key) {
    LLVM_DEBUG(errs() << "MIR2Vec: Missing vocabulary entry for " << Key
                      << "; using zero vector. This will result in an error "
                         "in the future.\n");
    ++MIRVocabMissCounter;
  };
````
- **L221 EN**: Continues logic with `unsigned LocalIndex =`.
  **L221 CN**: 继续处理逻辑：`unsigned LocalIndex =`。
- **L222 EN**: Continues logic with `getLocalIndex(Pos, Layout.VirtRegBase, RegisterOperandNames.size(),`.
  **L222 CN**: 继续处理逻辑：`getLocalIndex(Pos, Layout.VirtRegBase, RegisterOperandNames.size(),`。
- **L223 EN**: Executes statement `"Local index out of bounds in virtual registers");`.
  **L223 CN**: 执行语句 `"Local index out of bounds in virtual registers");`。
- **L224 EN**: Returns `"VirtReg_" + RegisterOperandNames[LocalIndex]` to the caller.
  **L224 CN**: 向调用者返回 `"VirtReg_" + RegisterOperandNames[LocalIndex]`。
- **L225 EN**: Closes the current scope.
  **L225 CN**: 关闭当前作用域。
- **L226 EN**: Separates nearby statements for readability.
  **L226 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L227 EN**: Provides part of the signature for `generateStorage`.
  **L227 CN**: 给出 `generateStorage` 的一部分签名。
- **L228 EN**: Continues logic with `const VocabMap &CommonOperandsMap,`.
  **L228 CN**: 继续处理逻辑：`const VocabMap &CommonOperandsMap,`。
- **L229 EN**: Continues logic with `const VocabMap &PhyRegMap,`.
  **L229 CN**: 继续处理逻辑：`const VocabMap &PhyRegMap,`。
- **L230 EN**: Starts block `const VocabMap &VirtRegMap)`.
  **L230 CN**: 开始代码块 `const VocabMap &VirtRegMap)`。
- **L231 EN**: Separates nearby statements for readability.
  **L231 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L232 EN**: Comment documents: `Helper for handling missing entities in the vocabulary.`.
  **L232 CN**: 注释说明：`Helper for handling missing entities in the vocabulary.`。
- **L233 EN**: Comment documents: `Currently, we use a zero vector. In the future, we will throw an error t…`.
  **L233 CN**: 注释说明：`Currently, we use a zero vector. In the future, we will throw an error t…`。
- **L234 EN**: Comment documents: `ensure that *all* known entities are present in the vocabulary.`.
  **L234 CN**: 注释说明：`ensure that *all* known entities are present in the vocabulary.`。
- **L235 EN**: Starts block `auto handleMissingEntity = [](StringRef Key)`.
  **L235 CN**: 开始代码块 `auto handleMissingEntity = [](StringRef Key)`。
- **L236 EN**: Emits debug-only tracing logic.
  **L236 CN**: 发出仅在调试时启用的跟踪逻辑。
- **L237 EN**: Continues logic with `<< "; using zero vector. This will result in an error "`.
  **L237 CN**: 继续处理逻辑：`<< "; using zero vector. This will result in an error "`。
- **L238 EN**: Executes statement `"in the future.\n");`.
  **L238 CN**: 执行语句 `"in the future.\n");`。
- **L239 EN**: Executes statement `++MIRVocabMissCounter;`.
  **L239 CN**: 执行语句 `++MIRVocabMissCounter;`。
- **L240 EN**: Closes the current scope.
  **L240 CN**: 关闭当前作用域。

### Lines 241-260

````cpp

  // Initialize opcode embeddings section
  unsigned EmbeddingDim = OpcodeMap.begin()->second.size();
  std::vector<Embedding> OpcodeEmbeddings(Layout.CommonOperandBase,
                                          Embedding(EmbeddingDim));

  // Populate opcode embeddings using canonical mapping
  for (auto COpcodeName : UniqueBaseOpcodeNames) {
    if (auto It = OpcodeMap.find(COpcodeName); It != OpcodeMap.end()) {
      auto COpcodeIndex = getCanonicalIndexForBaseName(COpcodeName);
      assert(COpcodeIndex < Layout.CommonOperandBase &&
             "Canonical index out of bounds");
      OpcodeEmbeddings[COpcodeIndex] = It->second;
    } else {
      handleMissingEntity(COpcodeName);
    }
  }

  // Initialize common operand embeddings section
  std::vector<Embedding> CommonOperandEmbeddings(std::size(CommonOperandNames),
````
- **L241 EN**: Separates nearby statements for readability.
  **L241 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L242 EN**: Comment documents: `Initialize opcode embeddings section`.
  **L242 CN**: 注释说明：`Initialize opcode embeddings section`。
- **L243 EN**: Assigns or initializes `unsigned EmbeddingDim`.
  **L243 CN**: 对 `unsigned EmbeddingDim` 进行赋值或初始化。
- **L244 EN**: Provides part of the signature for `OpcodeEmbeddings`.
  **L244 CN**: 给出 `OpcodeEmbeddings` 的一部分签名。
- **L245 EN**: Executes statement `Embedding(EmbeddingDim));`.
  **L245 CN**: 执行语句 `Embedding(EmbeddingDim));`。
- **L246 EN**: Separates nearby statements for readability.
  **L246 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L247 EN**: Comment documents: `Populate opcode embeddings using canonical mapping`.
  **L247 CN**: 注释说明：`Populate opcode embeddings using canonical mapping`。
- **L248 EN**: Starts a loop over a sequence or range.
  **L248 CN**: 开始遍历序列或范围的循环。
- **L249 EN**: Begins a conditional branch.
  **L249 CN**: 开始一个条件分支。
- **L250 EN**: Assigns or initializes `auto COpcodeIndex`.
  **L250 CN**: 对 `auto COpcodeIndex` 进行赋值或初始化。
- **L251 EN**: Checks an invariant in debug builds.
  **L251 CN**: 在调试构建中检查一个不变量。
- **L252 EN**: Executes statement `"Canonical index out of bounds");`.
  **L252 CN**: 执行语句 `"Canonical index out of bounds");`。
- **L253 EN**: Assigns or initializes `OpcodeEmbeddings[COpcodeIndex]`.
  **L253 CN**: 对 `OpcodeEmbeddings[COpcodeIndex]` 进行赋值或初始化。
- **L254 EN**: Starts block `} else`.
  **L254 CN**: 开始代码块 `} else`。
- **L255 EN**: Executes statement `handleMissingEntity(COpcodeName);`.
  **L255 CN**: 执行语句 `handleMissingEntity(COpcodeName);`。
- **L256 EN**: Closes the current scope.
  **L256 CN**: 关闭当前作用域。
- **L257 EN**: Closes the current scope.
  **L257 CN**: 关闭当前作用域。
- **L258 EN**: Separates nearby statements for readability.
  **L258 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L259 EN**: Comment documents: `Initialize common operand embeddings section`.
  **L259 CN**: 注释说明：`Initialize common operand embeddings section`。
- **L260 EN**: Provides part of the signature for `CommonOperandEmbeddings`.
  **L260 CN**: 给出 `CommonOperandEmbeddings` 的一部分签名。

### Lines 261-280

````cpp
                                                 Embedding(EmbeddingDim));
  unsigned OperandIndex = 0;
  for (const auto &CommonOperandName : CommonOperandNames) {
    if (auto It = CommonOperandsMap.find(CommonOperandName.str());
        It != CommonOperandsMap.end()) {
      CommonOperandEmbeddings[OperandIndex] = It->second;
    } else {
      handleMissingEntity(CommonOperandName);
    }
    ++OperandIndex;
  }

  // Helper lambda for creating register operand embeddings
  auto createRegisterEmbeddings = [&](const VocabMap &RegMap) {
    std::vector<Embedding> RegEmbeddings(TRI.getNumRegClasses(),
                                         Embedding(EmbeddingDim));
    unsigned RegOperandIndex = 0;
    for (const auto &RegOperandName : RegisterOperandNames) {
      if (auto It = RegMap.find(RegOperandName); It != RegMap.end())
        RegEmbeddings[RegOperandIndex] = It->second;
````
- **L261 EN**: Executes statement `Embedding(EmbeddingDim));`.
  **L261 CN**: 执行语句 `Embedding(EmbeddingDim));`。
- **L262 EN**: Assigns or initializes `unsigned OperandIndex`.
  **L262 CN**: 对 `unsigned OperandIndex` 进行赋值或初始化。
- **L263 EN**: Starts a loop over a sequence or range.
  **L263 CN**: 开始遍历序列或范围的循环。
- **L264 EN**: Begins a conditional branch.
  **L264 CN**: 开始一个条件分支。
- **L265 EN**: Starts block `It != CommonOperandsMap.end())`.
  **L265 CN**: 开始代码块 `It != CommonOperandsMap.end())`。
- **L266 EN**: Assigns or initializes `CommonOperandEmbeddings[OperandIndex]`.
  **L266 CN**: 对 `CommonOperandEmbeddings[OperandIndex]` 进行赋值或初始化。
- **L267 EN**: Starts block `} else`.
  **L267 CN**: 开始代码块 `} else`。
- **L268 EN**: Executes statement `handleMissingEntity(CommonOperandName);`.
  **L268 CN**: 执行语句 `handleMissingEntity(CommonOperandName);`。
- **L269 EN**: Closes the current scope.
  **L269 CN**: 关闭当前作用域。
- **L270 EN**: Executes statement `++OperandIndex;`.
  **L270 CN**: 执行语句 `++OperandIndex;`。
- **L271 EN**: Closes the current scope.
  **L271 CN**: 关闭当前作用域。
- **L272 EN**: Separates nearby statements for readability.
  **L272 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L273 EN**: Comment documents: `Helper lambda for creating register operand embeddings`.
  **L273 CN**: 注释说明：`Helper lambda for creating register operand embeddings`。
- **L274 EN**: Starts block `auto createRegisterEmbeddings = [&](const VocabMap &RegMap)`.
  **L274 CN**: 开始代码块 `auto createRegisterEmbeddings = [&](const VocabMap &RegMap)`。
- **L275 EN**: Provides part of the signature for `RegEmbeddings`.
  **L275 CN**: 给出 `RegEmbeddings` 的一部分签名。
- **L276 EN**: Executes statement `Embedding(EmbeddingDim));`.
  **L276 CN**: 执行语句 `Embedding(EmbeddingDim));`。
- **L277 EN**: Assigns or initializes `unsigned RegOperandIndex`.
  **L277 CN**: 对 `unsigned RegOperandIndex` 进行赋值或初始化。
- **L278 EN**: Starts a loop over a sequence or range.
  **L278 CN**: 开始遍历序列或范围的循环。
- **L279 EN**: Begins a conditional branch.
  **L279 CN**: 开始一个条件分支。
- **L280 EN**: Assigns or initializes `RegEmbeddings[RegOperandIndex]`.
  **L280 CN**: 对 `RegEmbeddings[RegOperandIndex]` 进行赋值或初始化。

### Lines 281-300

````cpp
      else
        handleMissingEntity(RegOperandName);
      ++RegOperandIndex;
    }
    return RegEmbeddings;
  };

  // Initialize register operand embeddings sections
  std::vector<Embedding> PhyRegEmbeddings = createRegisterEmbeddings(PhyRegMap);
  std::vector<Embedding> VirtRegEmbeddings =
      createRegisterEmbeddings(VirtRegMap);

  // Scale the vocabulary sections based on the provided weights
  auto scaleVocabSection = [](std::vector<Embedding> &Embeddings,
                              double Weight) {
    for (auto &Embedding : Embeddings)
      Embedding *= Weight;
  };
  scaleVocabSection(OpcodeEmbeddings, OpcWeight);
  scaleVocabSection(CommonOperandEmbeddings, CommonOperandWeight);
````
- **L281 EN**: Handles the fallback branch.
  **L281 CN**: 处理兜底分支。
- **L282 EN**: Executes statement `handleMissingEntity(RegOperandName);`.
  **L282 CN**: 执行语句 `handleMissingEntity(RegOperandName);`。
- **L283 EN**: Executes statement `++RegOperandIndex;`.
  **L283 CN**: 执行语句 `++RegOperandIndex;`。
- **L284 EN**: Closes the current scope.
  **L284 CN**: 关闭当前作用域。
- **L285 EN**: Returns `RegEmbeddings` to the caller.
  **L285 CN**: 向调用者返回 `RegEmbeddings`。
- **L286 EN**: Closes the current scope.
  **L286 CN**: 关闭当前作用域。
- **L287 EN**: Separates nearby statements for readability.
  **L287 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L288 EN**: Comment documents: `Initialize register operand embeddings sections`.
  **L288 CN**: 注释说明：`Initialize register operand embeddings sections`。
- **L289 EN**: Assigns or initializes `std::vector<Embedding> PhyRegEmbeddings`.
  **L289 CN**: 对 `std::vector<Embedding> PhyRegEmbeddings` 进行赋值或初始化。
- **L290 EN**: Continues logic with `std::vector<Embedding> VirtRegEmbeddings =`.
  **L290 CN**: 继续处理逻辑：`std::vector<Embedding> VirtRegEmbeddings =`。
- **L291 EN**: Executes statement `createRegisterEmbeddings(VirtRegMap);`.
  **L291 CN**: 执行语句 `createRegisterEmbeddings(VirtRegMap);`。
- **L292 EN**: Separates nearby statements for readability.
  **L292 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L293 EN**: Comment documents: `Scale the vocabulary sections based on the provided weights`.
  **L293 CN**: 注释说明：`Scale the vocabulary sections based on the provided weights`。
- **L294 EN**: Continues logic with `auto scaleVocabSection = [](std::vector<Embedding> &Embeddings,`.
  **L294 CN**: 继续处理逻辑：`auto scaleVocabSection = [](std::vector<Embedding> &Embeddings,`。
- **L295 EN**: Starts block `double Weight)`.
  **L295 CN**: 开始代码块 `double Weight)`。
- **L296 EN**: Starts a loop over a sequence or range.
  **L296 CN**: 开始遍历序列或范围的循环。
- **L297 EN**: Assigns or initializes `Embedding *`.
  **L297 CN**: 对 `Embedding *` 进行赋值或初始化。
- **L298 EN**: Closes the current scope.
  **L298 CN**: 关闭当前作用域。
- **L299 EN**: Executes statement `scaleVocabSection(OpcodeEmbeddings, OpcWeight);`.
  **L299 CN**: 执行语句 `scaleVocabSection(OpcodeEmbeddings, OpcWeight);`。
- **L300 EN**: Executes statement `scaleVocabSection(CommonOperandEmbeddings, CommonOperandWeight);`.
  **L300 CN**: 执行语句 `scaleVocabSection(CommonOperandEmbeddings, CommonOperandWeight);`。

### Lines 301-320

````cpp
  scaleVocabSection(PhyRegEmbeddings, RegOperandWeight);
  scaleVocabSection(VirtRegEmbeddings, RegOperandWeight);

  std::vector<std::vector<Embedding>> Sections(
      static_cast<unsigned>(Section::MaxSections));
  Sections[static_cast<unsigned>(Section::Opcodes)] =
      std::move(OpcodeEmbeddings);
  Sections[static_cast<unsigned>(Section::CommonOperands)] =
      std::move(CommonOperandEmbeddings);
  Sections[static_cast<unsigned>(Section::PhyRegisters)] =
      std::move(PhyRegEmbeddings);
  Sections[static_cast<unsigned>(Section::VirtRegisters)] =
      std::move(VirtRegEmbeddings);

  Storage = ir2vec::VocabStorage(std::move(Sections));
}

void MIRVocabulary::buildCanonicalOpcodeMapping() {
  // Check if already built
  if (!UniqueBaseOpcodeNames.empty())
````
- **L301 EN**: Executes statement `scaleVocabSection(PhyRegEmbeddings, RegOperandWeight);`.
  **L301 CN**: 执行语句 `scaleVocabSection(PhyRegEmbeddings, RegOperandWeight);`。
- **L302 EN**: Executes statement `scaleVocabSection(VirtRegEmbeddings, RegOperandWeight);`.
  **L302 CN**: 执行语句 `scaleVocabSection(VirtRegEmbeddings, RegOperandWeight);`。
- **L303 EN**: Separates nearby statements for readability.
  **L303 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L304 EN**: Provides part of the signature for `Sections`.
  **L304 CN**: 给出 `Sections` 的一部分签名。
- **L305 EN**: Executes statement `static_cast<unsigned>(Section::MaxSections));`.
  **L305 CN**: 执行语句 `static_cast<unsigned>(Section::MaxSections));`。
- **L306 EN**: Continues logic with `Sections[static_cast<unsigned>(Section::Opcodes)] =`.
  **L306 CN**: 继续处理逻辑：`Sections[static_cast<unsigned>(Section::Opcodes)] =`。
- **L307 EN**: Declares function or method `move`.
  **L307 CN**: 声明函数或方法 `move`。
- **L308 EN**: Continues logic with `Sections[static_cast<unsigned>(Section::CommonOperands)] =`.
  **L308 CN**: 继续处理逻辑：`Sections[static_cast<unsigned>(Section::CommonOperands)] =`。
- **L309 EN**: Declares function or method `move`.
  **L309 CN**: 声明函数或方法 `move`。
- **L310 EN**: Continues logic with `Sections[static_cast<unsigned>(Section::PhyRegisters)] =`.
  **L310 CN**: 继续处理逻辑：`Sections[static_cast<unsigned>(Section::PhyRegisters)] =`。
- **L311 EN**: Declares function or method `move`.
  **L311 CN**: 声明函数或方法 `move`。
- **L312 EN**: Continues logic with `Sections[static_cast<unsigned>(Section::VirtRegisters)] =`.
  **L312 CN**: 继续处理逻辑：`Sections[static_cast<unsigned>(Section::VirtRegisters)] =`。
- **L313 EN**: Declares function or method `move`.
  **L313 CN**: 声明函数或方法 `move`。
- **L314 EN**: Separates nearby statements for readability.
  **L314 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L315 EN**: Declares function or method `VocabStorage`.
  **L315 CN**: 声明函数或方法 `VocabStorage`。
- **L316 EN**: Closes the current scope.
  **L316 CN**: 关闭当前作用域。
- **L317 EN**: Separates nearby statements for readability.
  **L317 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L318 EN**: Begins the definition of `buildCanonicalOpcodeMapping`.
  **L318 CN**: 开始定义 `buildCanonicalOpcodeMapping`。
- **L319 EN**: Comment documents: `Check if already built`.
  **L319 CN**: 注释说明：`Check if already built`。
- **L320 EN**: Begins a conditional branch.
  **L320 CN**: 开始一个条件分支。

### Lines 321-340

````cpp
    return;

  // Build mapping from opcodes to canonical base opcode indices
  for (unsigned Opcode = 0; Opcode < TII.getNumOpcodes(); ++Opcode) {
    std::string BaseOpcode = extractBaseOpcodeName(TII.getName(Opcode));
    UniqueBaseOpcodeNames.insert(BaseOpcode);
  }

  LLVM_DEBUG(dbgs() << "MIR2Vec: Built canonical mapping for target with "
                    << UniqueBaseOpcodeNames.size()
                    << " unique base opcodes\n");
}

void MIRVocabulary::buildRegisterOperandMapping() {
  // Check if already built
  if (!RegisterOperandNames.empty())
    return;

  for (unsigned RC = 0; RC < TRI.getNumRegClasses(); ++RC) {
    const TargetRegisterClass *RegClass = TRI.getRegClass(RC);
````
- **L321 EN**: Returns control to the caller.
  **L321 CN**: 将控制流返回给调用者。
- **L322 EN**: Separates nearby statements for readability.
  **L322 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L323 EN**: Comment documents: `Build mapping from opcodes to canonical base opcode indices`.
  **L323 CN**: 注释说明：`Build mapping from opcodes to canonical base opcode indices`。
- **L324 EN**: Starts a loop over a sequence or range.
  **L324 CN**: 开始遍历序列或范围的循环。
- **L325 EN**: Assigns or initializes `std::string BaseOpcode`.
  **L325 CN**: 对 `std::string BaseOpcode` 进行赋值或初始化。
- **L326 EN**: Executes statement `UniqueBaseOpcodeNames.insert(BaseOpcode);`.
  **L326 CN**: 执行语句 `UniqueBaseOpcodeNames.insert(BaseOpcode);`。
- **L327 EN**: Closes the current scope.
  **L327 CN**: 关闭当前作用域。
- **L328 EN**: Separates nearby statements for readability.
  **L328 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L329 EN**: Emits debug-only tracing logic.
  **L329 CN**: 发出仅在调试时启用的跟踪逻辑。
- **L330 EN**: Continues logic with `<< UniqueBaseOpcodeNames.size()`.
  **L330 CN**: 继续处理逻辑：`<< UniqueBaseOpcodeNames.size()`。
- **L331 EN**: Executes statement `<< " unique base opcodes\n");`.
  **L331 CN**: 执行语句 `<< " unique base opcodes\n");`。
- **L332 EN**: Closes the current scope.
  **L332 CN**: 关闭当前作用域。
- **L333 EN**: Separates nearby statements for readability.
  **L333 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L334 EN**: Begins the definition of `buildRegisterOperandMapping`.
  **L334 CN**: 开始定义 `buildRegisterOperandMapping`。
- **L335 EN**: Comment documents: `Check if already built`.
  **L335 CN**: 注释说明：`Check if already built`。
- **L336 EN**: Begins a conditional branch.
  **L336 CN**: 开始一个条件分支。
- **L337 EN**: Returns control to the caller.
  **L337 CN**: 将控制流返回给调用者。
- **L338 EN**: Separates nearby statements for readability.
  **L338 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L339 EN**: Starts a loop over a sequence or range.
  **L339 CN**: 开始遍历序列或范围的循环。
- **L340 EN**: Assigns or initializes `const TargetRegisterClass *RegClass`.
  **L340 CN**: 对 `const TargetRegisterClass *RegClass` 进行赋值或初始化。

### Lines 341-360

````cpp
    if (!RegClass)
      continue;

    // Get the register class name
    StringRef ClassName = TRI.getRegClassName(RegClass);
    RegisterOperandNames.push_back(ClassName.str());
  }
}

unsigned MIRVocabulary::getCommonOperandIndex(
    MachineOperand::MachineOperandType OperandType) const {
  assert(OperandType != MachineOperand::MO_Register &&
         "Expected non-register operand type");
  assert(OperandType > MachineOperand::MO_Register &&
         OperandType < MachineOperand::MO_Last && "Operand type out of bounds");
  return static_cast<unsigned>(OperandType) - 1;
}

unsigned MIRVocabulary::getRegisterOperandIndex(Register Reg) const {
  assert(!RegisterOperandNames.empty() && "Register operand mapping not built");
````
- **L341 EN**: Begins a conditional branch.
  **L341 CN**: 开始一个条件分支。
- **L342 EN**: Skips to the next loop iteration.
  **L342 CN**: 跳到下一次循环迭代。
- **L343 EN**: Separates nearby statements for readability.
  **L343 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L344 EN**: Comment documents: `Get the register class name`.
  **L344 CN**: 注释说明：`Get the register class name`。
- **L345 EN**: Assigns or initializes `StringRef ClassName`.
  **L345 CN**: 对 `StringRef ClassName` 进行赋值或初始化。
- **L346 EN**: Executes statement `RegisterOperandNames.push_back(ClassName.str());`.
  **L346 CN**: 执行语句 `RegisterOperandNames.push_back(ClassName.str());`。
- **L347 EN**: Closes the current scope.
  **L347 CN**: 关闭当前作用域。
- **L348 EN**: Closes the current scope.
  **L348 CN**: 关闭当前作用域。
- **L349 EN**: Separates nearby statements for readability.
  **L349 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L350 EN**: Provides part of the signature for `getCommonOperandIndex`.
  **L350 CN**: 给出 `getCommonOperandIndex` 的一部分签名。
- **L351 EN**: Starts block `MachineOperand::MachineOperandType OperandType) const`.
  **L351 CN**: 开始代码块 `MachineOperand::MachineOperandType OperandType) const`。
- **L352 EN**: Checks an invariant in debug builds.
  **L352 CN**: 在调试构建中检查一个不变量。
- **L353 EN**: Executes statement `"Expected non-register operand type");`.
  **L353 CN**: 执行语句 `"Expected non-register operand type");`。
- **L354 EN**: Checks an invariant in debug builds.
  **L354 CN**: 在调试构建中检查一个不变量。
- **L355 EN**: Executes statement `OperandType < MachineOperand::MO_Last && "Operand type out of bounds");`.
  **L355 CN**: 执行语句 `OperandType < MachineOperand::MO_Last && "Operand type out of bounds");`。
- **L356 EN**: Returns `static_cast<unsigned>(OperandType) - 1` to the caller.
  **L356 CN**: 向调用者返回 `static_cast<unsigned>(OperandType) - 1`。
- **L357 EN**: Closes the current scope.
  **L357 CN**: 关闭当前作用域。
- **L358 EN**: Separates nearby statements for readability.
  **L358 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L359 EN**: Begins the definition of `getRegisterOperandIndex`.
  **L359 CN**: 开始定义 `getRegisterOperandIndex`。
- **L360 EN**: Checks an invariant in debug builds.
  **L360 CN**: 在调试构建中检查一个不变量。

### Lines 361-380

````cpp
  assert(Reg.isValid() && "Invalid register; not expected here");
  assert((Reg.isPhysical() || Reg.isVirtual()) &&
         "Expected a physical or virtual register");

  const TargetRegisterClass *RegClass = nullptr;

  // For physical registers, use TRI to get minimal register class as a
  // physical register can belong to multiple classes. For virtual
  // registers, use MRI to uniquely identify the assigned register class.
  if (Reg.isPhysical())
    RegClass = TRI.getMinimalPhysRegClass(Reg);
  else
    RegClass = MRI.getRegClass(Reg);

  if (RegClass)
    return RegClass->getID();
  // Fallback for registers without a class (shouldn't happen)
  llvm_unreachable("Register operand without a valid register class");
  return 0;
}
````
- **L361 EN**: Checks an invariant in debug builds.
  **L361 CN**: 在调试构建中检查一个不变量。
- **L362 EN**: Checks an invariant in debug builds.
  **L362 CN**: 在调试构建中检查一个不变量。
- **L363 EN**: Executes statement `"Expected a physical or virtual register");`.
  **L363 CN**: 执行语句 `"Expected a physical or virtual register");`。
- **L364 EN**: Separates nearby statements for readability.
  **L364 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L365 EN**: Assigns or initializes `const TargetRegisterClass *RegClass`.
  **L365 CN**: 对 `const TargetRegisterClass *RegClass` 进行赋值或初始化。
- **L366 EN**: Separates nearby statements for readability.
  **L366 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L367 EN**: Comment documents: `For physical registers, use TRI to get minimal register class as a`.
  **L367 CN**: 注释说明：`For physical registers, use TRI to get minimal register class as a`。
- **L368 EN**: Comment documents: `physical register can belong to multiple classes. For virtual`.
  **L368 CN**: 注释说明：`physical register can belong to multiple classes. For virtual`。
- **L369 EN**: Comment documents: `registers, use MRI to uniquely identify the assigned register class.`.
  **L369 CN**: 注释说明：`registers, use MRI to uniquely identify the assigned register class.`。
- **L370 EN**: Begins a conditional branch.
  **L370 CN**: 开始一个条件分支。
- **L371 EN**: Assigns or initializes `RegClass`.
  **L371 CN**: 对 `RegClass` 进行赋值或初始化。
- **L372 EN**: Handles the fallback branch.
  **L372 CN**: 处理兜底分支。
- **L373 EN**: Assigns or initializes `RegClass`.
  **L373 CN**: 对 `RegClass` 进行赋值或初始化。
- **L374 EN**: Separates nearby statements for readability.
  **L374 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L375 EN**: Begins a conditional branch.
  **L375 CN**: 开始一个条件分支。
- **L376 EN**: Returns `RegClass->getID()` to the caller.
  **L376 CN**: 向调用者返回 `RegClass->getID()`。
- **L377 EN**: Comment documents: `Fallback for registers without a class (shouldn't happen)`.
  **L377 CN**: 注释说明：`Fallback for registers without a class (shouldn't happen)`。
- **L378 EN**: Executes statement `llvm_unreachable("Register operand without a valid register class");`.
  **L378 CN**: 执行语句 `llvm_unreachable("Register operand without a valid register class");`。
- **L379 EN**: Returns `0` to the caller.
  **L379 CN**: 向调用者返回 `0`。
- **L380 EN**: Closes the current scope.
  **L380 CN**: 关闭当前作用域。

### Lines 381-400

````cpp

Expected<MIRVocabulary> MIRVocabulary::createDummyVocabForTest(
    const TargetInstrInfo &TII, const TargetRegisterInfo &TRI,
    const MachineRegisterInfo &MRI, unsigned Dim) {
  assert(Dim > 0 && "Dimension must be greater than zero");

  float DummyVal = 0.1f;

  VocabMap DummyOpcMap, DummyOperandMap, DummyPhyRegMap, DummyVirtRegMap;

  // Process opcodes directly without creating temporary vocabulary
  for (unsigned Opcode = 0; Opcode < TII.getNumOpcodes(); ++Opcode) {
    std::string BaseOpcode = extractBaseOpcodeName(TII.getName(Opcode));
    if (DummyOpcMap.count(BaseOpcode) == 0) { // Only add if not already present
      DummyOpcMap[BaseOpcode] = Embedding(Dim, DummyVal);
      DummyVal += 0.1f;
    }
  }

  // Add common operands
````
- **L381 EN**: Separates nearby statements for readability.
  **L381 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L382 EN**: Provides part of the signature for `createDummyVocabForTest`.
  **L382 CN**: 给出 `createDummyVocabForTest` 的一部分签名。
- **L383 EN**: Continues logic with `const TargetInstrInfo &TII, const TargetRegisterInfo &TRI,`.
  **L383 CN**: 继续处理逻辑：`const TargetInstrInfo &TII, const TargetRegisterInfo &TRI,`。
- **L384 EN**: Starts block `const MachineRegisterInfo &MRI, unsigned Dim)`.
  **L384 CN**: 开始代码块 `const MachineRegisterInfo &MRI, unsigned Dim)`。
- **L385 EN**: Checks an invariant in debug builds.
  **L385 CN**: 在调试构建中检查一个不变量。
- **L386 EN**: Separates nearby statements for readability.
  **L386 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L387 EN**: Assigns or initializes `float DummyVal`.
  **L387 CN**: 对 `float DummyVal` 进行赋值或初始化。
- **L388 EN**: Separates nearby statements for readability.
  **L388 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L389 EN**: Executes statement `VocabMap DummyOpcMap, DummyOperandMap, DummyPhyRegMap, DummyVirtRegMap;`.
  **L389 CN**: 执行语句 `VocabMap DummyOpcMap, DummyOperandMap, DummyPhyRegMap, DummyVirtRegMap;`。
- **L390 EN**: Separates nearby statements for readability.
  **L390 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L391 EN**: Comment documents: `Process opcodes directly without creating temporary vocabulary`.
  **L391 CN**: 注释说明：`Process opcodes directly without creating temporary vocabulary`。
- **L392 EN**: Starts a loop over a sequence or range.
  **L392 CN**: 开始遍历序列或范围的循环。
- **L393 EN**: Assigns or initializes `std::string BaseOpcode`.
  **L393 CN**: 对 `std::string BaseOpcode` 进行赋值或初始化。
- **L394 EN**: Begins a conditional branch.
  **L394 CN**: 开始一个条件分支。
- **L395 EN**: Assigns or initializes `DummyOpcMap[BaseOpcode]`.
  **L395 CN**: 对 `DummyOpcMap[BaseOpcode]` 进行赋值或初始化。
- **L396 EN**: Assigns or initializes `DummyVal +`.
  **L396 CN**: 对 `DummyVal +` 进行赋值或初始化。
- **L397 EN**: Closes the current scope.
  **L397 CN**: 关闭当前作用域。
- **L398 EN**: Closes the current scope.
  **L398 CN**: 关闭当前作用域。
- **L399 EN**: Separates nearby statements for readability.
  **L399 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L400 EN**: Comment documents: `Add common operands`.
  **L400 CN**: 注释说明：`Add common operands`。

### Lines 401-420

````cpp
  for (const auto &CommonOperandName : CommonOperandNames) {
    DummyOperandMap[CommonOperandName.str()] = Embedding(Dim, DummyVal);
    DummyVal += 0.1f;
  }

  // Process register classes directly
  for (unsigned RC = 0; RC < TRI.getNumRegClasses(); ++RC) {
    const TargetRegisterClass *RegClass = TRI.getRegClass(RC);
    if (!RegClass)
      continue;

    std::string ClassName = TRI.getRegClassName(RegClass);
    DummyPhyRegMap[ClassName] = Embedding(Dim, DummyVal);
    DummyVirtRegMap[ClassName] = Embedding(Dim, DummyVal);
    DummyVal += 0.1f;
  }

  // Create vocabulary directly without temporary instance
  return MIRVocabulary::create(
      std::move(DummyOpcMap), std::move(DummyOperandMap),
````
- **L401 EN**: Starts a loop over a sequence or range.
  **L401 CN**: 开始遍历序列或范围的循环。
- **L402 EN**: Assigns or initializes `DummyOperandMap[CommonOperandName.str()]`.
  **L402 CN**: 对 `DummyOperandMap[CommonOperandName.str()]` 进行赋值或初始化。
- **L403 EN**: Assigns or initializes `DummyVal +`.
  **L403 CN**: 对 `DummyVal +` 进行赋值或初始化。
- **L404 EN**: Closes the current scope.
  **L404 CN**: 关闭当前作用域。
- **L405 EN**: Separates nearby statements for readability.
  **L405 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L406 EN**: Comment documents: `Process register classes directly`.
  **L406 CN**: 注释说明：`Process register classes directly`。
- **L407 EN**: Starts a loop over a sequence or range.
  **L407 CN**: 开始遍历序列或范围的循环。
- **L408 EN**: Assigns or initializes `const TargetRegisterClass *RegClass`.
  **L408 CN**: 对 `const TargetRegisterClass *RegClass` 进行赋值或初始化。
- **L409 EN**: Begins a conditional branch.
  **L409 CN**: 开始一个条件分支。
- **L410 EN**: Skips to the next loop iteration.
  **L410 CN**: 跳到下一次循环迭代。
- **L411 EN**: Separates nearby statements for readability.
  **L411 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L412 EN**: Assigns or initializes `std::string ClassName`.
  **L412 CN**: 对 `std::string ClassName` 进行赋值或初始化。
- **L413 EN**: Assigns or initializes `DummyPhyRegMap[ClassName]`.
  **L413 CN**: 对 `DummyPhyRegMap[ClassName]` 进行赋值或初始化。
- **L414 EN**: Assigns or initializes `DummyVirtRegMap[ClassName]`.
  **L414 CN**: 对 `DummyVirtRegMap[ClassName]` 进行赋值或初始化。
- **L415 EN**: Assigns or initializes `DummyVal +`.
  **L415 CN**: 对 `DummyVal +` 进行赋值或初始化。
- **L416 EN**: Closes the current scope.
  **L416 CN**: 关闭当前作用域。
- **L417 EN**: Separates nearby statements for readability.
  **L417 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L418 EN**: Comment documents: `Create vocabulary directly without temporary instance`.
  **L418 CN**: 注释说明：`Create vocabulary directly without temporary instance`。
- **L419 EN**: Returns `MIRVocabulary::create(` to the caller.
  **L419 CN**: 向调用者返回 `MIRVocabulary::create(`。
- **L420 EN**: Provides part of the signature for `move`.
  **L420 CN**: 给出 `move` 的一部分签名。

### Lines 421-440

````cpp
      std::move(DummyPhyRegMap), std::move(DummyVirtRegMap), TII, TRI, MRI);
}

//===----------------------------------------------------------------------===//
// MIR2VecVocabProvider and MIR2VecVocabLegacyAnalysis
//===----------------------------------------------------------------------===//

Expected<mir2vec::MIRVocabulary>
MIR2VecVocabProvider::getVocabulary(const Module &M) {
  VocabMap OpcVocab, CommonOperandVocab, PhyRegVocabMap, VirtRegVocabMap;

  if (Error Err = readVocabulary(OpcVocab, CommonOperandVocab, PhyRegVocabMap,
                                 VirtRegVocabMap))
    return std::move(Err);

  for (const auto &F : M) {
    if (F.isDeclaration())
      continue;

    if (auto *MF = MMI.getMachineFunction(F)) {
````
- **L421 EN**: Declares function or method `move`.
  **L421 CN**: 声明函数或方法 `move`。
- **L422 EN**: Closes the current scope.
  **L422 CN**: 关闭当前作用域。
- **L423 EN**: Separates nearby statements for readability.
  **L423 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L424 EN**: Comment documents: `===---------------------------------------------------------------------…`.
  **L424 CN**: 注释说明：`===---------------------------------------------------------------------…`。
- **L425 EN**: Comment documents: `MIR2VecVocabProvider and MIR2VecVocabLegacyAnalysis`.
  **L425 CN**: 注释说明：`MIR2VecVocabProvider and MIR2VecVocabLegacyAnalysis`。
- **L426 EN**: Comment documents: `===---------------------------------------------------------------------…`.
  **L426 CN**: 注释说明：`===---------------------------------------------------------------------…`。
- **L427 EN**: Separates nearby statements for readability.
  **L427 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L428 EN**: Continues logic with `Expected<mir2vec::MIRVocabulary>`.
  **L428 CN**: 继续处理逻辑：`Expected<mir2vec::MIRVocabulary>`。
- **L429 EN**: Begins the definition of `getVocabulary`.
  **L429 CN**: 开始定义 `getVocabulary`。
- **L430 EN**: Executes statement `VocabMap OpcVocab, CommonOperandVocab, PhyRegVocabMap, VirtRegVocabMap;`.
  **L430 CN**: 执行语句 `VocabMap OpcVocab, CommonOperandVocab, PhyRegVocabMap, VirtRegVocabMap;`。
- **L431 EN**: Separates nearby statements for readability.
  **L431 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L432 EN**: Begins a conditional branch.
  **L432 CN**: 开始一个条件分支。
- **L433 EN**: Continues logic with `VirtRegVocabMap))`.
  **L433 CN**: 继续处理逻辑：`VirtRegVocabMap))`。
- **L434 EN**: Returns `std::move(Err)` to the caller.
  **L434 CN**: 向调用者返回 `std::move(Err)`。
- **L435 EN**: Separates nearby statements for readability.
  **L435 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L436 EN**: Starts a loop over a sequence or range.
  **L436 CN**: 开始遍历序列或范围的循环。
- **L437 EN**: Begins a conditional branch.
  **L437 CN**: 开始一个条件分支。
- **L438 EN**: Skips to the next loop iteration.
  **L438 CN**: 跳到下一次循环迭代。
- **L439 EN**: Separates nearby statements for readability.
  **L439 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L440 EN**: Begins a conditional branch.
  **L440 CN**: 开始一个条件分支。

### Lines 441-460

````cpp
      auto &Subtarget = MF->getSubtarget();
      if (const auto *TII = Subtarget.getInstrInfo())
        if (const auto *TRI = Subtarget.getRegisterInfo())
          return mir2vec::MIRVocabulary::create(
              std::move(OpcVocab), std::move(CommonOperandVocab),
              std::move(PhyRegVocabMap), std::move(VirtRegVocabMap), *TII, *TRI,
              MF->getRegInfo());
    }
  }
  return createStringError(errc::invalid_argument,
                           "No machine functions found in module");
}

Error MIR2VecVocabProvider::readVocabulary(VocabMap &OpcodeVocab,
                                           VocabMap &CommonOperandVocab,
                                           VocabMap &PhyRegVocabMap,
                                           VocabMap &VirtRegVocabMap) {
  if (VocabFile.empty())
    return createStringError(
        errc::invalid_argument,
````
- **L441 EN**: Assigns or initializes `auto &Subtarget`.
  **L441 CN**: 对 `auto &Subtarget` 进行赋值或初始化。
- **L442 EN**: Begins a conditional branch.
  **L442 CN**: 开始一个条件分支。
- **L443 EN**: Begins a conditional branch.
  **L443 CN**: 开始一个条件分支。
- **L444 EN**: Returns `mir2vec::MIRVocabulary::create(` to the caller.
  **L444 CN**: 向调用者返回 `mir2vec::MIRVocabulary::create(`。
- **L445 EN**: Provides part of the signature for `move`.
  **L445 CN**: 给出 `move` 的一部分签名。
- **L446 EN**: Provides part of the signature for `move`.
  **L446 CN**: 给出 `move` 的一部分签名。
- **L447 EN**: Executes statement `MF->getRegInfo());`.
  **L447 CN**: 执行语句 `MF->getRegInfo());`。
- **L448 EN**: Closes the current scope.
  **L448 CN**: 关闭当前作用域。
- **L449 EN**: Closes the current scope.
  **L449 CN**: 关闭当前作用域。
- **L450 EN**: Returns `createStringError(errc::invalid_argument,` to the caller.
  **L450 CN**: 向调用者返回 `createStringError(errc::invalid_argument,`。
- **L451 EN**: Executes statement `"No machine functions found in module");`.
  **L451 CN**: 执行语句 `"No machine functions found in module");`。
- **L452 EN**: Closes the current scope.
  **L452 CN**: 关闭当前作用域。
- **L453 EN**: Separates nearby statements for readability.
  **L453 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L454 EN**: Provides part of the signature for `readVocabulary`.
  **L454 CN**: 给出 `readVocabulary` 的一部分签名。
- **L455 EN**: Continues logic with `VocabMap &CommonOperandVocab,`.
  **L455 CN**: 继续处理逻辑：`VocabMap &CommonOperandVocab,`。
- **L456 EN**: Continues logic with `VocabMap &PhyRegVocabMap,`.
  **L456 CN**: 继续处理逻辑：`VocabMap &PhyRegVocabMap,`。
- **L457 EN**: Starts block `VocabMap &VirtRegVocabMap)`.
  **L457 CN**: 开始代码块 `VocabMap &VirtRegVocabMap)`。
- **L458 EN**: Begins a conditional branch.
  **L458 CN**: 开始一个条件分支。
- **L459 EN**: Returns `createStringError(` to the caller.
  **L459 CN**: 向调用者返回 `createStringError(`。
- **L460 EN**: Continues logic with `errc::invalid_argument,`.
  **L460 CN**: 继续处理逻辑：`errc::invalid_argument,`。

### Lines 461-480

````cpp
        "MIR2Vec vocabulary file path not specified; set it "
        "using --mir2vec-vocab-path");

  auto BufOrError = MemoryBuffer::getFileOrSTDIN(VocabFile, /*IsText=*/true);
  if (!BufOrError)
    return createFileError(VocabFile, BufOrError.getError());

  auto Content = BufOrError.get()->getBuffer();

  Expected<json::Value> ParsedVocabValue = json::parse(Content);
  if (!ParsedVocabValue)
    return ParsedVocabValue.takeError();

  unsigned OpcodeDim = 0, CommonOperandDim = 0, PhyRegOperandDim = 0,
           VirtRegOperandDim = 0;
  if (auto Err = ir2vec::VocabStorage::parseVocabSection(
          "Opcodes", *ParsedVocabValue, OpcodeVocab, OpcodeDim))
    return Err;

  if (auto Err = ir2vec::VocabStorage::parseVocabSection(
````
- **L461 EN**: Continues logic with `"MIR2Vec vocabulary file path not specified; set it "`.
  **L461 CN**: 继续处理逻辑：`"MIR2Vec vocabulary file path not specified; set it "`。
- **L462 EN**: Executes statement `"using --mir2vec-vocab-path");`.
  **L462 CN**: 执行语句 `"using --mir2vec-vocab-path");`。
- **L463 EN**: Separates nearby statements for readability.
  **L463 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L464 EN**: Declares function or method `getFileOrSTDIN`.
  **L464 CN**: 声明函数或方法 `getFileOrSTDIN`。
- **L465 EN**: Begins a conditional branch.
  **L465 CN**: 开始一个条件分支。
- **L466 EN**: Returns `createFileError(VocabFile, BufOrError.getError())` to the caller.
  **L466 CN**: 向调用者返回 `createFileError(VocabFile, BufOrError.getError())`。
- **L467 EN**: Separates nearby statements for readability.
  **L467 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L468 EN**: Assigns or initializes `auto Content`.
  **L468 CN**: 对 `auto Content` 进行赋值或初始化。
- **L469 EN**: Separates nearby statements for readability.
  **L469 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L470 EN**: Declares function or method `parse`.
  **L470 CN**: 声明函数或方法 `parse`。
- **L471 EN**: Begins a conditional branch.
  **L471 CN**: 开始一个条件分支。
- **L472 EN**: Returns `ParsedVocabValue.takeError()` to the caller.
  **L472 CN**: 向调用者返回 `ParsedVocabValue.takeError()`。
- **L473 EN**: Separates nearby statements for readability.
  **L473 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L474 EN**: Continues logic with `unsigned OpcodeDim = 0, CommonOperandDim = 0, PhyRegOperandDim = 0,`.
  **L474 CN**: 继续处理逻辑：`unsigned OpcodeDim = 0, CommonOperandDim = 0, PhyRegOperandDim = 0,`。
- **L475 EN**: Assigns or initializes `VirtRegOperandDim`.
  **L475 CN**: 对 `VirtRegOperandDim` 进行赋值或初始化。
- **L476 EN**: Begins a conditional branch.
  **L476 CN**: 开始一个条件分支。
- **L477 EN**: Continues logic with `"Opcodes", *ParsedVocabValue, OpcodeVocab, OpcodeDim))`.
  **L477 CN**: 继续处理逻辑：`"Opcodes", *ParsedVocabValue, OpcodeVocab, OpcodeDim))`。
- **L478 EN**: Returns `Err` to the caller.
  **L478 CN**: 向调用者返回 `Err`。
- **L479 EN**: Separates nearby statements for readability.
  **L479 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L480 EN**: Begins a conditional branch.
  **L480 CN**: 开始一个条件分支。

### Lines 481-500

````cpp
          "CommonOperands", *ParsedVocabValue, CommonOperandVocab,
          CommonOperandDim))
    return Err;

  if (auto Err = ir2vec::VocabStorage::parseVocabSection(
          "PhysicalRegisters", *ParsedVocabValue, PhyRegVocabMap,
          PhyRegOperandDim))
    return Err;

  if (auto Err = ir2vec::VocabStorage::parseVocabSection(
          "VirtualRegisters", *ParsedVocabValue, VirtRegVocabMap,
          VirtRegOperandDim))
    return Err;

  // All sections must have the same embedding dimension
  if (!(OpcodeDim == CommonOperandDim && CommonOperandDim == PhyRegOperandDim &&
        PhyRegOperandDim == VirtRegOperandDim)) {
    return createStringError(
        errc::illegal_byte_sequence,
        "MIR2Vec vocabulary sections have different dimensions");
````
- **L481 EN**: Continues logic with `"CommonOperands", *ParsedVocabValue, CommonOperandVocab,`.
  **L481 CN**: 继续处理逻辑：`"CommonOperands", *ParsedVocabValue, CommonOperandVocab,`。
- **L482 EN**: Continues logic with `CommonOperandDim))`.
  **L482 CN**: 继续处理逻辑：`CommonOperandDim))`。
- **L483 EN**: Returns `Err` to the caller.
  **L483 CN**: 向调用者返回 `Err`。
- **L484 EN**: Separates nearby statements for readability.
  **L484 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L485 EN**: Begins a conditional branch.
  **L485 CN**: 开始一个条件分支。
- **L486 EN**: Continues logic with `"PhysicalRegisters", *ParsedVocabValue, PhyRegVocabMap,`.
  **L486 CN**: 继续处理逻辑：`"PhysicalRegisters", *ParsedVocabValue, PhyRegVocabMap,`。
- **L487 EN**: Continues logic with `PhyRegOperandDim))`.
  **L487 CN**: 继续处理逻辑：`PhyRegOperandDim))`。
- **L488 EN**: Returns `Err` to the caller.
  **L488 CN**: 向调用者返回 `Err`。
- **L489 EN**: Separates nearby statements for readability.
  **L489 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L490 EN**: Begins a conditional branch.
  **L490 CN**: 开始一个条件分支。
- **L491 EN**: Continues logic with `"VirtualRegisters", *ParsedVocabValue, VirtRegVocabMap,`.
  **L491 CN**: 继续处理逻辑：`"VirtualRegisters", *ParsedVocabValue, VirtRegVocabMap,`。
- **L492 EN**: Continues logic with `VirtRegOperandDim))`.
  **L492 CN**: 继续处理逻辑：`VirtRegOperandDim))`。
- **L493 EN**: Returns `Err` to the caller.
  **L493 CN**: 向调用者返回 `Err`。
- **L494 EN**: Separates nearby statements for readability.
  **L494 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L495 EN**: Comment documents: `All sections must have the same embedding dimension`.
  **L495 CN**: 注释说明：`All sections must have the same embedding dimension`。
- **L496 EN**: Begins a conditional branch.
  **L496 CN**: 开始一个条件分支。
- **L497 EN**: Starts block `PhyRegOperandDim == VirtRegOperandDim))`.
  **L497 CN**: 开始代码块 `PhyRegOperandDim == VirtRegOperandDim))`。
- **L498 EN**: Returns `createStringError(` to the caller.
  **L498 CN**: 向调用者返回 `createStringError(`。
- **L499 EN**: Continues logic with `errc::illegal_byte_sequence,`.
  **L499 CN**: 继续处理逻辑：`errc::illegal_byte_sequence,`。
- **L500 EN**: Executes statement `"MIR2Vec vocabulary sections have different dimensions");`.
  **L500 CN**: 执行语句 `"MIR2Vec vocabulary sections have different dimensions");`。

### Lines 501-520

````cpp
  }

  return Error::success();
}

char MIR2VecVocabLegacyAnalysis::ID = 0;
INITIALIZE_PASS_BEGIN(MIR2VecVocabLegacyAnalysis, "mir2vec-vocab-analysis",
                      "MIR2Vec Vocabulary Analysis", false, true)
INITIALIZE_PASS_DEPENDENCY(MachineModuleInfoWrapperPass)
INITIALIZE_PASS_END(MIR2VecVocabLegacyAnalysis, "mir2vec-vocab-analysis",
                    "MIR2Vec Vocabulary Analysis", false, true)

StringRef MIR2VecVocabLegacyAnalysis::getPassName() const {
  return "MIR2Vec Vocabulary Analysis";
}

//===----------------------------------------------------------------------===//
// MIREmbedder and its subclasses
//===----------------------------------------------------------------------===//

````
- **L501 EN**: Closes the current scope.
  **L501 CN**: 关闭当前作用域。
- **L502 EN**: Separates nearby statements for readability.
  **L502 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L503 EN**: Returns `Error::success()` to the caller.
  **L503 CN**: 向调用者返回 `Error::success()`。
- **L504 EN**: Closes the current scope.
  **L504 CN**: 关闭当前作用域。
- **L505 EN**: Separates nearby statements for readability.
  **L505 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L506 EN**: Assigns or initializes `char MIR2VecVocabLegacyAnalysis::ID`.
  **L506 CN**: 对 `char MIR2VecVocabLegacyAnalysis::ID` 进行赋值或初始化。
- **L507 EN**: Continues logic with `INITIALIZE_PASS_BEGIN(MIR2VecVocabLegacyAnalysis, "mir2vec-vocab-analysi…`.
  **L507 CN**: 继续处理逻辑：`INITIALIZE_PASS_BEGIN(MIR2VecVocabLegacyAnalysis, "mir2vec-vocab-analysi…`。
- **L508 EN**: Continues logic with `"MIR2Vec Vocabulary Analysis", false, true)`.
  **L508 CN**: 继续处理逻辑：`"MIR2Vec Vocabulary Analysis", false, true)`。
- **L509 EN**: Continues logic with `INITIALIZE_PASS_DEPENDENCY(MachineModuleInfoWrapperPass)`.
  **L509 CN**: 继续处理逻辑：`INITIALIZE_PASS_DEPENDENCY(MachineModuleInfoWrapperPass)`。
- **L510 EN**: Continues logic with `INITIALIZE_PASS_END(MIR2VecVocabLegacyAnalysis, "mir2vec-vocab-analysis"…`.
  **L510 CN**: 继续处理逻辑：`INITIALIZE_PASS_END(MIR2VecVocabLegacyAnalysis, "mir2vec-vocab-analysis"…`。
- **L511 EN**: Continues logic with `"MIR2Vec Vocabulary Analysis", false, true)`.
  **L511 CN**: 继续处理逻辑：`"MIR2Vec Vocabulary Analysis", false, true)`。
- **L512 EN**: Separates nearby statements for readability.
  **L512 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L513 EN**: Begins the definition of `getPassName`.
  **L513 CN**: 开始定义 `getPassName`。
- **L514 EN**: Returns `"MIR2Vec Vocabulary Analysis"` to the caller.
  **L514 CN**: 向调用者返回 `"MIR2Vec Vocabulary Analysis"`。
- **L515 EN**: Closes the current scope.
  **L515 CN**: 关闭当前作用域。
- **L516 EN**: Separates nearby statements for readability.
  **L516 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L517 EN**: Comment documents: `===---------------------------------------------------------------------…`.
  **L517 CN**: 注释说明：`===---------------------------------------------------------------------…`。
- **L518 EN**: Comment documents: `MIREmbedder and its subclasses`.
  **L518 CN**: 注释说明：`MIREmbedder and its subclasses`。
- **L519 EN**: Comment documents: `===---------------------------------------------------------------------…`.
  **L519 CN**: 注释说明：`===---------------------------------------------------------------------…`。
- **L520 EN**: Separates nearby statements for readability.
  **L520 CN**: 空行，用于分隔相邻语句并提升可读性。

### Lines 521-540

````cpp
std::unique_ptr<MIREmbedder> MIREmbedder::create(MIR2VecKind Mode,
                                                 const MachineFunction &MF,
                                                 const MIRVocabulary &Vocab) {
  switch (Mode) {
  case MIR2VecKind::Symbolic:
    return std::make_unique<SymbolicMIREmbedder>(MF, Vocab);
  }
  return nullptr;
}

Embedding MIREmbedder::computeEmbeddings(const MachineBasicBlock &MBB) const {
  Embedding MBBVector(Dimension, 0);

  // Get instruction info for opcode name resolution
  const auto &Subtarget = MF.getSubtarget();
  const auto *TII = Subtarget.getInstrInfo();
  if (!TII) {
    MF.getFunction().getContext().emitError(
        "MIR2Vec: No TargetInstrInfo available; cannot compute embeddings");
    return MBBVector;
````
- **L521 EN**: Provides part of the signature for `create`.
  **L521 CN**: 给出 `create` 的一部分签名。
- **L522 EN**: Continues logic with `const MachineFunction &MF,`.
  **L522 CN**: 继续处理逻辑：`const MachineFunction &MF,`。
- **L523 EN**: Starts block `const MIRVocabulary &Vocab)`.
  **L523 CN**: 开始代码块 `const MIRVocabulary &Vocab)`。
- **L524 EN**: Starts a multi-way branch.
  **L524 CN**: 开始一个多路分支。
- **L525 EN**: Handles one switch case.
  **L525 CN**: 处理一个 switch 分支。
- **L526 EN**: Returns `std::make_unique<SymbolicMIREmbedder>(MF, Vocab)` to the caller.
  **L526 CN**: 向调用者返回 `std::make_unique<SymbolicMIREmbedder>(MF, Vocab)`。
- **L527 EN**: Closes the current scope.
  **L527 CN**: 关闭当前作用域。
- **L528 EN**: Returns `nullptr` to the caller.
  **L528 CN**: 向调用者返回 `nullptr`。
- **L529 EN**: Closes the current scope.
  **L529 CN**: 关闭当前作用域。
- **L530 EN**: Separates nearby statements for readability.
  **L530 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L531 EN**: Begins the definition of `computeEmbeddings`.
  **L531 CN**: 开始定义 `computeEmbeddings`。
- **L532 EN**: Declares function or method `MBBVector`.
  **L532 CN**: 声明函数或方法 `MBBVector`。
- **L533 EN**: Separates nearby statements for readability.
  **L533 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L534 EN**: Comment documents: `Get instruction info for opcode name resolution`.
  **L534 CN**: 注释说明：`Get instruction info for opcode name resolution`。
- **L535 EN**: Assigns or initializes `const auto &Subtarget`.
  **L535 CN**: 对 `const auto &Subtarget` 进行赋值或初始化。
- **L536 EN**: Assigns or initializes `const auto *TII`.
  **L536 CN**: 对 `const auto *TII` 进行赋值或初始化。
- **L537 EN**: Begins a conditional branch.
  **L537 CN**: 开始一个条件分支。
- **L538 EN**: Continues logic with `MF.getFunction().getContext().emitError(`.
  **L538 CN**: 继续处理逻辑：`MF.getFunction().getContext().emitError(`。
- **L539 EN**: Executes statement `"MIR2Vec: No TargetInstrInfo available; cannot compute embeddings");`.
  **L539 CN**: 执行语句 `"MIR2Vec: No TargetInstrInfo available; cannot compute embeddings");`。
- **L540 EN**: Returns `MBBVector` to the caller.
  **L540 CN**: 向调用者返回 `MBBVector`。

### Lines 541-560

````cpp
  }

  // Process each machine instruction in the basic block
  for (const auto &MI : MBB) {
    // Skip debug instructions and other metadata
    if (MI.isDebugInstr())
      continue;
    MBBVector += computeEmbeddings(MI);
  }

  return MBBVector;
}

Embedding MIREmbedder::computeEmbeddings() const {
  Embedding MFuncVector(Dimension, 0);

  // Consider all reachable machine basic blocks in the function
  for (const auto *MBB : depth_first(&MF))
    MFuncVector += computeEmbeddings(*MBB);
  return MFuncVector;
````
- **L541 EN**: Closes the current scope.
  **L541 CN**: 关闭当前作用域。
- **L542 EN**: Separates nearby statements for readability.
  **L542 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L543 EN**: Comment documents: `Process each machine instruction in the basic block`.
  **L543 CN**: 注释说明：`Process each machine instruction in the basic block`。
- **L544 EN**: Starts a loop over a sequence or range.
  **L544 CN**: 开始遍历序列或范围的循环。
- **L545 EN**: Comment documents: `Skip debug instructions and other metadata`.
  **L545 CN**: 注释说明：`Skip debug instructions and other metadata`。
- **L546 EN**: Begins a conditional branch.
  **L546 CN**: 开始一个条件分支。
- **L547 EN**: Skips to the next loop iteration.
  **L547 CN**: 跳到下一次循环迭代。
- **L548 EN**: Assigns or initializes `MBBVector +`.
  **L548 CN**: 对 `MBBVector +` 进行赋值或初始化。
- **L549 EN**: Closes the current scope.
  **L549 CN**: 关闭当前作用域。
- **L550 EN**: Separates nearby statements for readability.
  **L550 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L551 EN**: Returns `MBBVector` to the caller.
  **L551 CN**: 向调用者返回 `MBBVector`。
- **L552 EN**: Closes the current scope.
  **L552 CN**: 关闭当前作用域。
- **L553 EN**: Separates nearby statements for readability.
  **L553 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L554 EN**: Begins the definition of `computeEmbeddings`.
  **L554 CN**: 开始定义 `computeEmbeddings`。
- **L555 EN**: Declares function or method `MFuncVector`.
  **L555 CN**: 声明函数或方法 `MFuncVector`。
- **L556 EN**: Separates nearby statements for readability.
  **L556 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L557 EN**: Comment documents: `Consider all reachable machine basic blocks in the function`.
  **L557 CN**: 注释说明：`Consider all reachable machine basic blocks in the function`。
- **L558 EN**: Starts a loop over a sequence or range.
  **L558 CN**: 开始遍历序列或范围的循环。
- **L559 EN**: Assigns or initializes `MFuncVector +`.
  **L559 CN**: 对 `MFuncVector +` 进行赋值或初始化。
- **L560 EN**: Returns `MFuncVector` to the caller.
  **L560 CN**: 向调用者返回 `MFuncVector`。

### Lines 561-580

````cpp
}

SymbolicMIREmbedder::SymbolicMIREmbedder(const MachineFunction &MF,
                                         const MIRVocabulary &Vocab)
    : MIREmbedder(MF, Vocab) {}

std::unique_ptr<SymbolicMIREmbedder>
SymbolicMIREmbedder::create(const MachineFunction &MF,
                            const MIRVocabulary &Vocab) {
  return std::make_unique<SymbolicMIREmbedder>(MF, Vocab);
}

Embedding SymbolicMIREmbedder::computeEmbeddings(const MachineInstr &MI) const {
  // Skip debug instructions and other metadata
  if (MI.isDebugInstr())
    return Embedding(Dimension, 0);

  // Opcode embedding
  Embedding InstructionEmbedding = Vocab[MI.getOpcode()];

````
- **L561 EN**: Closes the current scope.
  **L561 CN**: 关闭当前作用域。
- **L562 EN**: Separates nearby statements for readability.
  **L562 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L563 EN**: Provides part of the signature for `SymbolicMIREmbedder`.
  **L563 CN**: 给出 `SymbolicMIREmbedder` 的一部分签名。
- **L564 EN**: Continues logic with `const MIRVocabulary &Vocab)`.
  **L564 CN**: 继续处理逻辑：`const MIRVocabulary &Vocab)`。
- **L565 EN**: Provides part of the signature for `MIREmbedder`.
  **L565 CN**: 给出 `MIREmbedder` 的一部分签名。
- **L566 EN**: Separates nearby statements for readability.
  **L566 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L567 EN**: Continues logic with `std::unique_ptr<SymbolicMIREmbedder>`.
  **L567 CN**: 继续处理逻辑：`std::unique_ptr<SymbolicMIREmbedder>`。
- **L568 EN**: Provides part of the signature for `create`.
  **L568 CN**: 给出 `create` 的一部分签名。
- **L569 EN**: Starts block `const MIRVocabulary &Vocab)`.
  **L569 CN**: 开始代码块 `const MIRVocabulary &Vocab)`。
- **L570 EN**: Returns `std::make_unique<SymbolicMIREmbedder>(MF, Vocab)` to the caller.
  **L570 CN**: 向调用者返回 `std::make_unique<SymbolicMIREmbedder>(MF, Vocab)`。
- **L571 EN**: Closes the current scope.
  **L571 CN**: 关闭当前作用域。
- **L572 EN**: Separates nearby statements for readability.
  **L572 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L573 EN**: Begins the definition of `computeEmbeddings`.
  **L573 CN**: 开始定义 `computeEmbeddings`。
- **L574 EN**: Comment documents: `Skip debug instructions and other metadata`.
  **L574 CN**: 注释说明：`Skip debug instructions and other metadata`。
- **L575 EN**: Begins a conditional branch.
  **L575 CN**: 开始一个条件分支。
- **L576 EN**: Returns `Embedding(Dimension, 0)` to the caller.
  **L576 CN**: 向调用者返回 `Embedding(Dimension, 0)`。
- **L577 EN**: Separates nearby statements for readability.
  **L577 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L578 EN**: Comment documents: `Opcode embedding`.
  **L578 CN**: 注释说明：`Opcode embedding`。
- **L579 EN**: Assigns or initializes `Embedding InstructionEmbedding`.
  **L579 CN**: 对 `Embedding InstructionEmbedding` 进行赋值或初始化。
- **L580 EN**: Separates nearby statements for readability.
  **L580 CN**: 空行，用于分隔相邻语句并提升可读性。

### Lines 581-600

````cpp
  // Add operand contributions
  for (const MachineOperand &MO : MI.operands())
    InstructionEmbedding += Vocab[MO];

  return InstructionEmbedding;
}

//===----------------------------------------------------------------------===//
// Printer Passes
//===----------------------------------------------------------------------===//

char MIR2VecVocabPrinterLegacyPass::ID = 0;
INITIALIZE_PASS_BEGIN(MIR2VecVocabPrinterLegacyPass, "print-mir2vec-vocab",
                      "MIR2Vec Vocabulary Printer Pass", false, true)
INITIALIZE_PASS_DEPENDENCY(MIR2VecVocabLegacyAnalysis)
INITIALIZE_PASS_DEPENDENCY(MachineModuleInfoWrapperPass)
INITIALIZE_PASS_END(MIR2VecVocabPrinterLegacyPass, "print-mir2vec-vocab",
                    "MIR2Vec Vocabulary Printer Pass", false, true)

bool MIR2VecVocabPrinterLegacyPass::runOnMachineFunction(MachineFunction &MF) {
````
- **L581 EN**: Comment documents: `Add operand contributions`.
  **L581 CN**: 注释说明：`Add operand contributions`。
- **L582 EN**: Starts a loop over a sequence or range.
  **L582 CN**: 开始遍历序列或范围的循环。
- **L583 EN**: Assigns or initializes `InstructionEmbedding +`.
  **L583 CN**: 对 `InstructionEmbedding +` 进行赋值或初始化。
- **L584 EN**: Separates nearby statements for readability.
  **L584 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L585 EN**: Returns `InstructionEmbedding` to the caller.
  **L585 CN**: 向调用者返回 `InstructionEmbedding`。
- **L586 EN**: Closes the current scope.
  **L586 CN**: 关闭当前作用域。
- **L587 EN**: Separates nearby statements for readability.
  **L587 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L588 EN**: Comment documents: `===---------------------------------------------------------------------…`.
  **L588 CN**: 注释说明：`===---------------------------------------------------------------------…`。
- **L589 EN**: Comment documents: `Printer Passes`.
  **L589 CN**: 注释说明：`Printer Passes`。
- **L590 EN**: Comment documents: `===---------------------------------------------------------------------…`.
  **L590 CN**: 注释说明：`===---------------------------------------------------------------------…`。
- **L591 EN**: Separates nearby statements for readability.
  **L591 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L592 EN**: Assigns or initializes `char MIR2VecVocabPrinterLegacyPass::ID`.
  **L592 CN**: 对 `char MIR2VecVocabPrinterLegacyPass::ID` 进行赋值或初始化。
- **L593 EN**: Continues logic with `INITIALIZE_PASS_BEGIN(MIR2VecVocabPrinterLegacyPass, "print-mir2vec-voca…`.
  **L593 CN**: 继续处理逻辑：`INITIALIZE_PASS_BEGIN(MIR2VecVocabPrinterLegacyPass, "print-mir2vec-voca…`。
- **L594 EN**: Continues logic with `"MIR2Vec Vocabulary Printer Pass", false, true)`.
  **L594 CN**: 继续处理逻辑：`"MIR2Vec Vocabulary Printer Pass", false, true)`。
- **L595 EN**: Continues logic with `INITIALIZE_PASS_DEPENDENCY(MIR2VecVocabLegacyAnalysis)`.
  **L595 CN**: 继续处理逻辑：`INITIALIZE_PASS_DEPENDENCY(MIR2VecVocabLegacyAnalysis)`。
- **L596 EN**: Continues logic with `INITIALIZE_PASS_DEPENDENCY(MachineModuleInfoWrapperPass)`.
  **L596 CN**: 继续处理逻辑：`INITIALIZE_PASS_DEPENDENCY(MachineModuleInfoWrapperPass)`。
- **L597 EN**: Continues logic with `INITIALIZE_PASS_END(MIR2VecVocabPrinterLegacyPass, "print-mir2vec-vocab"…`.
  **L597 CN**: 继续处理逻辑：`INITIALIZE_PASS_END(MIR2VecVocabPrinterLegacyPass, "print-mir2vec-vocab"…`。
- **L598 EN**: Continues logic with `"MIR2Vec Vocabulary Printer Pass", false, true)`.
  **L598 CN**: 继续处理逻辑：`"MIR2Vec Vocabulary Printer Pass", false, true)`。
- **L599 EN**: Separates nearby statements for readability.
  **L599 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L600 EN**: Begins the definition of `runOnMachineFunction`.
  **L600 CN**: 开始定义 `runOnMachineFunction`。

### Lines 601-620

````cpp
  return false;
}

bool MIR2VecVocabPrinterLegacyPass::doFinalization(Module &M) {
  auto &Analysis = getAnalysis<MIR2VecVocabLegacyAnalysis>();
  auto MIR2VecVocabOrErr = Analysis.getMIR2VecVocabulary(M);

  if (!MIR2VecVocabOrErr) {
    OS << "MIR2Vec Vocabulary Printer: Failed to get vocabulary - "
       << toString(MIR2VecVocabOrErr.takeError()) << "\n";
    return false;
  }

  auto &MIR2VecVocab = *MIR2VecVocabOrErr;
  unsigned Pos = 0;
  for (const auto &Entry : MIR2VecVocab) {
    // Skip zero embeddings to avoid printing entries not in the vocabulary.
    // This makes the output stable across changes to the opcode list.
    if (PrintAllVocabEntries || !Entry.isZero()) {
      OS << "Key: " << MIR2VecVocab.getStringKey(Pos) << ": ";
````
- **L601 EN**: Returns `false` to the caller.
  **L601 CN**: 向调用者返回 `false`。
- **L602 EN**: Closes the current scope.
  **L602 CN**: 关闭当前作用域。
- **L603 EN**: Separates nearby statements for readability.
  **L603 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L604 EN**: Begins the definition of `doFinalization`.
  **L604 CN**: 开始定义 `doFinalization`。
- **L605 EN**: Assigns or initializes `auto &Analysis`.
  **L605 CN**: 对 `auto &Analysis` 进行赋值或初始化。
- **L606 EN**: Assigns or initializes `auto MIR2VecVocabOrErr`.
  **L606 CN**: 对 `auto MIR2VecVocabOrErr` 进行赋值或初始化。
- **L607 EN**: Separates nearby statements for readability.
  **L607 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L608 EN**: Begins a conditional branch.
  **L608 CN**: 开始一个条件分支。
- **L609 EN**: Continues logic with `OS << "MIR2Vec Vocabulary Printer: Failed to get vocabulary - "`.
  **L609 CN**: 继续处理逻辑：`OS << "MIR2Vec Vocabulary Printer: Failed to get vocabulary - "`。
- **L610 EN**: Declares function or method `toString`.
  **L610 CN**: 声明函数或方法 `toString`。
- **L611 EN**: Returns `false` to the caller.
  **L611 CN**: 向调用者返回 `false`。
- **L612 EN**: Closes the current scope.
  **L612 CN**: 关闭当前作用域。
- **L613 EN**: Separates nearby statements for readability.
  **L613 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L614 EN**: Assigns or initializes `auto &MIR2VecVocab`.
  **L614 CN**: 对 `auto &MIR2VecVocab` 进行赋值或初始化。
- **L615 EN**: Assigns or initializes `unsigned Pos`.
  **L615 CN**: 对 `unsigned Pos` 进行赋值或初始化。
- **L616 EN**: Starts a loop over a sequence or range.
  **L616 CN**: 开始遍历序列或范围的循环。
- **L617 EN**: Comment documents: `Skip zero embeddings to avoid printing entries not in the vocabulary.`.
  **L617 CN**: 注释说明：`Skip zero embeddings to avoid printing entries not in the vocabulary.`。
- **L618 EN**: Comment documents: `This makes the output stable across changes to the opcode list.`.
  **L618 CN**: 注释说明：`This makes the output stable across changes to the opcode list.`。
- **L619 EN**: Begins a conditional branch.
  **L619 CN**: 开始一个条件分支。
- **L620 EN**: Executes statement `OS << "Key: " << MIR2VecVocab.getStringKey(Pos) << ": ";`.
  **L620 CN**: 执行语句 `OS << "Key: " << MIR2VecVocab.getStringKey(Pos) << ": ";`。

### Lines 621-640

````cpp
      Entry.print(OS);
    }
    ++Pos;
  }

  return false;
}

MachineFunctionPass *
llvm::createMIR2VecVocabPrinterLegacyPass(raw_ostream &OS) {
  return new MIR2VecVocabPrinterLegacyPass(OS);
}

char MIR2VecPrinterLegacyPass::ID = 0;
INITIALIZE_PASS_BEGIN(MIR2VecPrinterLegacyPass, "print-mir2vec",
                      "MIR2Vec Embedder Printer Pass", false, true)
INITIALIZE_PASS_DEPENDENCY(MIR2VecVocabLegacyAnalysis)
INITIALIZE_PASS_DEPENDENCY(MachineModuleInfoWrapperPass)
INITIALIZE_PASS_END(MIR2VecPrinterLegacyPass, "print-mir2vec",
                    "MIR2Vec Embedder Printer Pass", false, true)
````
- **L621 EN**: Executes statement `Entry.print(OS);`.
  **L621 CN**: 执行语句 `Entry.print(OS);`。
- **L622 EN**: Closes the current scope.
  **L622 CN**: 关闭当前作用域。
- **L623 EN**: Executes statement `++Pos;`.
  **L623 CN**: 执行语句 `++Pos;`。
- **L624 EN**: Closes the current scope.
  **L624 CN**: 关闭当前作用域。
- **L625 EN**: Separates nearby statements for readability.
  **L625 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L626 EN**: Returns `false` to the caller.
  **L626 CN**: 向调用者返回 `false`。
- **L627 EN**: Closes the current scope.
  **L627 CN**: 关闭当前作用域。
- **L628 EN**: Separates nearby statements for readability.
  **L628 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L629 EN**: Continues logic with `MachineFunctionPass *`.
  **L629 CN**: 继续处理逻辑：`MachineFunctionPass *`。
- **L630 EN**: Begins the definition of `createMIR2VecVocabPrinterLegacyPass`.
  **L630 CN**: 开始定义 `createMIR2VecVocabPrinterLegacyPass`。
- **L631 EN**: Returns `new MIR2VecVocabPrinterLegacyPass(OS)` to the caller.
  **L631 CN**: 向调用者返回 `new MIR2VecVocabPrinterLegacyPass(OS)`。
- **L632 EN**: Closes the current scope.
  **L632 CN**: 关闭当前作用域。
- **L633 EN**: Separates nearby statements for readability.
  **L633 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L634 EN**: Assigns or initializes `char MIR2VecPrinterLegacyPass::ID`.
  **L634 CN**: 对 `char MIR2VecPrinterLegacyPass::ID` 进行赋值或初始化。
- **L635 EN**: Continues logic with `INITIALIZE_PASS_BEGIN(MIR2VecPrinterLegacyPass, "print-mir2vec",`.
  **L635 CN**: 继续处理逻辑：`INITIALIZE_PASS_BEGIN(MIR2VecPrinterLegacyPass, "print-mir2vec",`。
- **L636 EN**: Continues logic with `"MIR2Vec Embedder Printer Pass", false, true)`.
  **L636 CN**: 继续处理逻辑：`"MIR2Vec Embedder Printer Pass", false, true)`。
- **L637 EN**: Continues logic with `INITIALIZE_PASS_DEPENDENCY(MIR2VecVocabLegacyAnalysis)`.
  **L637 CN**: 继续处理逻辑：`INITIALIZE_PASS_DEPENDENCY(MIR2VecVocabLegacyAnalysis)`。
- **L638 EN**: Continues logic with `INITIALIZE_PASS_DEPENDENCY(MachineModuleInfoWrapperPass)`.
  **L638 CN**: 继续处理逻辑：`INITIALIZE_PASS_DEPENDENCY(MachineModuleInfoWrapperPass)`。
- **L639 EN**: Continues logic with `INITIALIZE_PASS_END(MIR2VecPrinterLegacyPass, "print-mir2vec",`.
  **L639 CN**: 继续处理逻辑：`INITIALIZE_PASS_END(MIR2VecPrinterLegacyPass, "print-mir2vec",`。
- **L640 EN**: Continues logic with `"MIR2Vec Embedder Printer Pass", false, true)`.
  **L640 CN**: 继续处理逻辑：`"MIR2Vec Embedder Printer Pass", false, true)`。

### Lines 641-660

````cpp

bool MIR2VecPrinterLegacyPass::runOnMachineFunction(MachineFunction &MF) {
  auto &Analysis = getAnalysis<MIR2VecVocabLegacyAnalysis>();
  auto VocabOrErr =
      Analysis.getMIR2VecVocabulary(*MF.getFunction().getParent());
  assert(VocabOrErr && "Failed to get MIR2Vec vocabulary");
  auto &MIRVocab = *VocabOrErr;

  auto Emb = mir2vec::MIREmbedder::create(MIR2VecEmbeddingKind, MF, MIRVocab);
  if (!Emb) {
    OS << "Error creating MIR2Vec embeddings for function " << MF.getName()
       << "\n";
    return false;
  }

  OS << "MIR2Vec embeddings for machine function " << MF.getName() << ":\n";
  OS << "Machine Function vector: ";
  Emb->getMFunctionVector().print(OS);

  OS << "Machine basic block vectors:\n";
````
- **L641 EN**: Separates nearby statements for readability.
  **L641 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L642 EN**: Begins the definition of `runOnMachineFunction`.
  **L642 CN**: 开始定义 `runOnMachineFunction`。
- **L643 EN**: Assigns or initializes `auto &Analysis`.
  **L643 CN**: 对 `auto &Analysis` 进行赋值或初始化。
- **L644 EN**: Continues logic with `auto VocabOrErr =`.
  **L644 CN**: 继续处理逻辑：`auto VocabOrErr =`。
- **L645 EN**: Executes statement `Analysis.getMIR2VecVocabulary(*MF.getFunction().getParent());`.
  **L645 CN**: 执行语句 `Analysis.getMIR2VecVocabulary(*MF.getFunction().getParent());`。
- **L646 EN**: Checks an invariant in debug builds.
  **L646 CN**: 在调试构建中检查一个不变量。
- **L647 EN**: Assigns or initializes `auto &MIRVocab`.
  **L647 CN**: 对 `auto &MIRVocab` 进行赋值或初始化。
- **L648 EN**: Separates nearby statements for readability.
  **L648 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L649 EN**: Declares function or method `create`.
  **L649 CN**: 声明函数或方法 `create`。
- **L650 EN**: Begins a conditional branch.
  **L650 CN**: 开始一个条件分支。
- **L651 EN**: Continues logic with `OS << "Error creating MIR2Vec embeddings for function " << MF.getName()`.
  **L651 CN**: 继续处理逻辑：`OS << "Error creating MIR2Vec embeddings for function " << MF.getName()`。
- **L652 EN**: Executes statement `<< "\n";`.
  **L652 CN**: 执行语句 `<< "\n";`。
- **L653 EN**: Returns `false` to the caller.
  **L653 CN**: 向调用者返回 `false`。
- **L654 EN**: Closes the current scope.
  **L654 CN**: 关闭当前作用域。
- **L655 EN**: Separates nearby statements for readability.
  **L655 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L656 EN**: Executes statement `OS << "MIR2Vec embeddings for machine function " << MF.getName() << ":\n…`.
  **L656 CN**: 执行语句 `OS << "MIR2Vec embeddings for machine function " << MF.getName() << ":\n…`。
- **L657 EN**: Executes statement `OS << "Machine Function vector: ";`.
  **L657 CN**: 执行语句 `OS << "Machine Function vector: ";`。
- **L658 EN**: Executes statement `Emb->getMFunctionVector().print(OS);`.
  **L658 CN**: 执行语句 `Emb->getMFunctionVector().print(OS);`。
- **L659 EN**: Separates nearby statements for readability.
  **L659 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L660 EN**: Executes statement `OS << "Machine basic block vectors:\n";`.
  **L660 CN**: 执行语句 `OS << "Machine basic block vectors:\n";`。

### Lines 661-680

````cpp
  for (const MachineBasicBlock &MBB : MF) {
    OS << "Machine basic block: " << MBB.getFullName() << ":\n";
    Emb->getMBBVector(MBB).print(OS);
  }

  OS << "Machine instruction vectors:\n";
  for (const MachineBasicBlock &MBB : MF) {
    for (const MachineInstr &MI : MBB) {
      // Skip debug instructions as they are not
      // embedded
      if (MI.isDebugInstr())
        continue;

      OS << "Machine instruction: ";
      MI.print(OS);
      Emb->getMInstVector(MI).print(OS);
    }
  }

  return false;
````
- **L661 EN**: Starts a loop over a sequence or range.
  **L661 CN**: 开始遍历序列或范围的循环。
- **L662 EN**: Executes statement `OS << "Machine basic block: " << MBB.getFullName() << ":\n";`.
  **L662 CN**: 执行语句 `OS << "Machine basic block: " << MBB.getFullName() << ":\n";`。
- **L663 EN**: Executes statement `Emb->getMBBVector(MBB).print(OS);`.
  **L663 CN**: 执行语句 `Emb->getMBBVector(MBB).print(OS);`。
- **L664 EN**: Closes the current scope.
  **L664 CN**: 关闭当前作用域。
- **L665 EN**: Separates nearby statements for readability.
  **L665 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L666 EN**: Executes statement `OS << "Machine instruction vectors:\n";`.
  **L666 CN**: 执行语句 `OS << "Machine instruction vectors:\n";`。
- **L667 EN**: Starts a loop over a sequence or range.
  **L667 CN**: 开始遍历序列或范围的循环。
- **L668 EN**: Starts a loop over a sequence or range.
  **L668 CN**: 开始遍历序列或范围的循环。
- **L669 EN**: Comment documents: `Skip debug instructions as they are not`.
  **L669 CN**: 注释说明：`Skip debug instructions as they are not`。
- **L670 EN**: Comment documents: `embedded`.
  **L670 CN**: 注释说明：`embedded`。
- **L671 EN**: Begins a conditional branch.
  **L671 CN**: 开始一个条件分支。
- **L672 EN**: Skips to the next loop iteration.
  **L672 CN**: 跳到下一次循环迭代。
- **L673 EN**: Separates nearby statements for readability.
  **L673 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L674 EN**: Executes statement `OS << "Machine instruction: ";`.
  **L674 CN**: 执行语句 `OS << "Machine instruction: ";`。
- **L675 EN**: Executes statement `MI.print(OS);`.
  **L675 CN**: 执行语句 `MI.print(OS);`。
- **L676 EN**: Executes statement `Emb->getMInstVector(MI).print(OS);`.
  **L676 CN**: 执行语句 `Emb->getMInstVector(MI).print(OS);`。
- **L677 EN**: Closes the current scope.
  **L677 CN**: 关闭当前作用域。
- **L678 EN**: Closes the current scope.
  **L678 CN**: 关闭当前作用域。
- **L679 EN**: Separates nearby statements for readability.
  **L679 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L680 EN**: Returns `false` to the caller.
  **L680 CN**: 向调用者返回 `false`。

### Lines 681-685

````cpp
}

MachineFunctionPass *llvm::createMIR2VecPrinterLegacyPass(raw_ostream &OS) {
  return new MIR2VecPrinterLegacyPass(OS);
}
````
- **L681 EN**: Closes the current scope.
  **L681 CN**: 关闭当前作用域。
- **L682 EN**: Separates nearby statements for readability.
  **L682 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L683 EN**: Begins the definition of `createMIR2VecPrinterLegacyPass`.
  **L683 CN**: 开始定义 `createMIR2VecPrinterLegacyPass`。
- **L684 EN**: Returns `new MIR2VecPrinterLegacyPass(OS)` to the caller.
  **L684 CN**: 向调用者返回 `new MIR2VecPrinterLegacyPass(OS)`。
- **L685 EN**: Closes the current scope.
  **L685 CN**: 关闭当前作用域。

## Key Concepts / 关键概念
- **Register management** / **寄存器管理**
- **Machine function processing** / **MachineFunction 处理**
- **Machine basic block handling** / **MachineBasicBlock 处理**
- **Machine instruction manipulation** / **机器指令操作**
- **Debug information handling** / **调试信息处理**
- **SSA/PHI lowering** / **SSA/PHI 降低**
- **Target-specific hooks** / **目标相关钩子**
- **Garbage-collection support** / **垃圾回收支持**

## Dependencies / 依赖关系
- **LLVM headers / LLVM 头文件**: `llvm/CodeGen/MIR2Vec.h`, `llvm/ADT/DepthFirstIterator.h`, `llvm/ADT/Statistic.h`, `llvm/CodeGen/TargetInstrInfo.h`, `llvm/IR/Module.h`, `llvm/InitializePasses.h`, `llvm/Pass.h`, `llvm/Support/Errc.h`, `llvm/Support/MemoryBuffer.h`, `llvm/Support/Regex.h`
- **Primary dependency domains / 主要依赖域**: CodeGen internals, LLVM support utilities, and C++ runtime helpers. / CodeGen 内部组件、LLVM 支持工具以及 C++ 运行时辅助设施。
