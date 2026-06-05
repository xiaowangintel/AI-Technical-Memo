# MIRFSDiscriminator.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/CodeGen/MIRFSDiscriminator.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: This file contains `See https://llvm.org/LICENSE.txt for license information` logic inside LLVM CodeGen.
- **Purpose (CN)**: 该文件在 LLVM CodeGen 中实现与“See https://llvm.org/LICENSE.txt for license information”相关的逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

````cpp
//===-------- MIRFSDiscriminator.cpp: Flow Sensitive Discriminator --------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file provides the implementation of a machine pass that adds the flow
// sensitive discriminator to the instruction debug information.
//
//===----------------------------------------------------------------------===//

#include "llvm/CodeGen/MIRFSDiscriminator.h"
#include "llvm/ADT/DenseMap.h"
#include "llvm/ADT/DenseSet.h"
#include "llvm/Analysis/BlockFrequencyInfoImpl.h"
#include "llvm/CodeGen/MIRFSDiscriminatorOptions.h"
#include "llvm/CodeGen/Passes.h"
#include "llvm/IR/DebugInfoMetadata.h"
````
- **L1 EN**: Comment documents: `===-------- MIRFSDiscriminator.cpp: Flow Sensitive Discriminator -------…`.
  **L1 CN**: 注释说明：`===-------- MIRFSDiscriminator.cpp: Flow Sensitive Discriminator -------…`。
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
- **L9 EN**: Comment documents: `This file provides the implementation of a machine pass that adds the fl…`.
  **L9 CN**: 注释说明：`This file provides the implementation of a machine pass that adds the fl…`。
- **L10 EN**: Comment documents: `sensitive discriminator to the instruction debug information.`.
  **L10 CN**: 注释说明：`sensitive discriminator to the instruction debug information.`。
- **L11 EN**: Continues the surrounding comment block.
  **L11 CN**: 延续周围的注释块。
- **L12 EN**: Comment documents: `===---------------------------------------------------------------------…`.
  **L12 CN**: 注释说明：`===---------------------------------------------------------------------…`。
- **L13 EN**: Separates nearby statements for readability.
  **L13 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L14 EN**: Includes LLVM header `llvm/CodeGen/MIRFSDiscriminator.h` for MIRFSDiscriminator support.
  **L14 CN**: 引入 LLVM 头文件 `llvm/CodeGen/MIRFSDiscriminator.h`，用于 MIRFSDiscriminator 相关支持。
- **L15 EN**: Includes LLVM header `llvm/ADT/DenseMap.h` for DenseMap support.
  **L15 CN**: 引入 LLVM 头文件 `llvm/ADT/DenseMap.h`，用于 DenseMap 相关支持。
- **L16 EN**: Includes LLVM header `llvm/ADT/DenseSet.h` for DenseSet support.
  **L16 CN**: 引入 LLVM 头文件 `llvm/ADT/DenseSet.h`，用于 DenseSet 相关支持。
- **L17 EN**: Includes LLVM header `llvm/Analysis/BlockFrequencyInfoImpl.h` for BlockFrequencyInfoImpl support.
  **L17 CN**: 引入 LLVM 头文件 `llvm/Analysis/BlockFrequencyInfoImpl.h`，用于 BlockFrequencyInfoImpl 相关支持。
- **L18 EN**: Includes LLVM header `llvm/CodeGen/MIRFSDiscriminatorOptions.h` for MIRFSDiscriminatorOptions support.
  **L18 CN**: 引入 LLVM 头文件 `llvm/CodeGen/MIRFSDiscriminatorOptions.h`，用于 MIRFSDiscriminatorOptions 相关支持。
- **L19 EN**: Includes LLVM header `llvm/CodeGen/Passes.h` for Passes support.
  **L19 CN**: 引入 LLVM 头文件 `llvm/CodeGen/Passes.h`，用于 Passes 相关支持。
- **L20 EN**: Includes LLVM header `llvm/IR/DebugInfoMetadata.h` for DebugInfoMetadata support.
  **L20 CN**: 引入 LLVM 头文件 `llvm/IR/DebugInfoMetadata.h`，用于 DebugInfoMetadata 相关支持。

### Lines 21-40

````cpp
#include "llvm/IR/Function.h"
#include "llvm/IR/Module.h"
#include "llvm/IR/PseudoProbe.h"
#include "llvm/InitializePasses.h"
#include "llvm/Support/CommandLine.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/raw_ostream.h"
#include "llvm/Support/xxhash.h"
#include "llvm/Transforms/Utils/SampleProfileLoaderBaseUtil.h"

using namespace llvm;
using namespace sampleprof;
using namespace sampleprofutil;

#define DEBUG_TYPE "mirfs-discriminators"

// TODO(xur): Remove this option and related code once we make true as the
// default.
cl::opt<bool> llvm::ImprovedFSDiscriminator(
    "improved-fs-discriminator", cl::Hidden, cl::init(false),
````
- **L21 EN**: Includes LLVM header `llvm/IR/Function.h` for Function support.
  **L21 CN**: 引入 LLVM 头文件 `llvm/IR/Function.h`，用于 Function 相关支持。
- **L22 EN**: Includes LLVM header `llvm/IR/Module.h` for Module support.
  **L22 CN**: 引入 LLVM 头文件 `llvm/IR/Module.h`，用于 Module 相关支持。
- **L23 EN**: Includes LLVM header `llvm/IR/PseudoProbe.h` for PseudoProbe support.
  **L23 CN**: 引入 LLVM 头文件 `llvm/IR/PseudoProbe.h`，用于 PseudoProbe 相关支持。
- **L24 EN**: Includes LLVM header `llvm/InitializePasses.h` for InitializePasses support.
  **L24 CN**: 引入 LLVM 头文件 `llvm/InitializePasses.h`，用于 InitializePasses 相关支持。
- **L25 EN**: Includes LLVM header `llvm/Support/CommandLine.h` for CommandLine support.
  **L25 CN**: 引入 LLVM 头文件 `llvm/Support/CommandLine.h`，用于 CommandLine 相关支持。
- **L26 EN**: Includes LLVM header `llvm/Support/Debug.h` for Debug support.
  **L26 CN**: 引入 LLVM 头文件 `llvm/Support/Debug.h`，用于 Debug 相关支持。
- **L27 EN**: Includes LLVM header `llvm/Support/raw_ostream.h` for raw_ostream support.
  **L27 CN**: 引入 LLVM 头文件 `llvm/Support/raw_ostream.h`，用于 raw_ostream 相关支持。
- **L28 EN**: Includes LLVM header `llvm/Support/xxhash.h` for xxhash support.
  **L28 CN**: 引入 LLVM 头文件 `llvm/Support/xxhash.h`，用于 xxhash 相关支持。
- **L29 EN**: Includes LLVM header `llvm/Transforms/Utils/SampleProfileLoaderBaseUtil.h` for SampleProfileLoaderBaseUtil support.
  **L29 CN**: 引入 LLVM 头文件 `llvm/Transforms/Utils/SampleProfileLoaderBaseUtil.h`，用于 SampleProfileLoaderBaseUtil 相关支持。
- **L30 EN**: Separates nearby statements for readability.
  **L30 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L31 EN**: Imports namespace `llvm` into this translation unit.
  **L31 CN**: 将命名空间 `llvm` 引入当前编译单元。
- **L32 EN**: Imports namespace `sampleprof` into this translation unit.
  **L32 CN**: 将命名空间 `sampleprof` 引入当前编译单元。
- **L33 EN**: Imports namespace `sampleprofutil` into this translation unit.
  **L33 CN**: 将命名空间 `sampleprofutil` 引入当前编译单元。
- **L34 EN**: Separates nearby statements for readability.
  **L34 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L35 EN**: Defines the LLVM debug channel used by this file.
  **L35 CN**: 定义该文件使用的 LLVM 调试通道。
- **L36 EN**: Separates nearby statements for readability.
  **L36 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L37 EN**: Comment documents: `TODO(xur): Remove this option and related code once we make true as the`.
  **L37 CN**: 注释说明：`TODO(xur): Remove this option and related code once we make true as the`。
- **L38 EN**: Comment documents: `default.`.
  **L38 CN**: 注释说明：`default.`。
- **L39 EN**: Declares LLVM command-line option `command-line option`.
  **L39 CN**: 声明 LLVM 命令行选项 `command-line option`。
- **L40 EN**: Provides part of the signature for `init`.
  **L40 CN**: 给出 `init` 的一部分签名。

### Lines 41-60

````cpp
    cl::desc("New FS discriminators encoding (incompatible with the original "
             "encoding)"));
char MIRAddFSDiscriminators::ID = 0;

INITIALIZE_PASS(MIRAddFSDiscriminators, DEBUG_TYPE,
                "Add MIR Flow Sensitive Discriminators",
                /* cfg = */ false, /* is_analysis = */ false)

char &llvm::MIRAddFSDiscriminatorsID = MIRAddFSDiscriminators::ID;

FunctionPass *llvm::createMIRAddFSDiscriminatorsPass(FSDiscriminatorPass P) {
  return new MIRAddFSDiscriminators(P);
}

// TODO(xur): Remove this once we switch to ImprovedFSDiscriminator.
// Compute a hash value using debug line number, and the line numbers from the
// inline stack.
static uint64_t getCallStackHashV0(const MachineBasicBlock &BB,
                                   const MachineInstr &MI,
                                   const DILocation *DIL) {
````
- **L41 EN**: Provides part of the signature for `desc`.
  **L41 CN**: 给出 `desc` 的一部分签名。
- **L42 EN**: Executes statement `"encoding)"));`.
  **L42 CN**: 执行语句 `"encoding)"));`。
- **L43 EN**: Assigns or initializes `char MIRAddFSDiscriminators::ID`.
  **L43 CN**: 对 `char MIRAddFSDiscriminators::ID` 进行赋值或初始化。
- **L44 EN**: Separates nearby statements for readability.
  **L44 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L45 EN**: Continues logic with `INITIALIZE_PASS(MIRAddFSDiscriminators, DEBUG_TYPE,`.
  **L45 CN**: 继续处理逻辑：`INITIALIZE_PASS(MIRAddFSDiscriminators, DEBUG_TYPE,`。
- **L46 EN**: Continues logic with `"Add MIR Flow Sensitive Discriminators",`.
  **L46 CN**: 继续处理逻辑：`"Add MIR Flow Sensitive Discriminators",`。
- **L47 EN**: Comment documents: `cfg = */ false, /* is_analysis = */ false)`.
  **L47 CN**: 注释说明：`cfg = */ false, /* is_analysis = */ false)`。
- **L48 EN**: Separates nearby statements for readability.
  **L48 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L49 EN**: Assigns or initializes `char &llvm::MIRAddFSDiscriminatorsID`.
  **L49 CN**: 对 `char &llvm::MIRAddFSDiscriminatorsID` 进行赋值或初始化。
- **L50 EN**: Separates nearby statements for readability.
  **L50 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L51 EN**: Begins the definition of `createMIRAddFSDiscriminatorsPass`.
  **L51 CN**: 开始定义 `createMIRAddFSDiscriminatorsPass`。
- **L52 EN**: Returns `new MIRAddFSDiscriminators(P)` to the caller.
  **L52 CN**: 向调用者返回 `new MIRAddFSDiscriminators(P)`。
- **L53 EN**: Closes the current scope.
  **L53 CN**: 关闭当前作用域。
- **L54 EN**: Separates nearby statements for readability.
  **L54 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L55 EN**: Comment documents: `TODO(xur): Remove this once we switch to ImprovedFSDiscriminator.`.
  **L55 CN**: 注释说明：`TODO(xur): Remove this once we switch to ImprovedFSDiscriminator.`。
- **L56 EN**: Comment documents: `Compute a hash value using debug line number, and the line numbers from …`.
  **L56 CN**: 注释说明：`Compute a hash value using debug line number, and the line numbers from …`。
- **L57 EN**: Comment documents: `inline stack.`.
  **L57 CN**: 注释说明：`inline stack.`。
- **L58 EN**: Provides part of the signature for `getCallStackHashV0`.
  **L58 CN**: 给出 `getCallStackHashV0` 的一部分签名。
- **L59 EN**: Continues logic with `const MachineInstr &MI,`.
  **L59 CN**: 继续处理逻辑：`const MachineInstr &MI,`。
- **L60 EN**: Starts block `const DILocation *DIL)`.
  **L60 CN**: 开始代码块 `const DILocation *DIL)`。

### Lines 61-80

````cpp
  auto updateHash = [](const StringRef &Str) -> uint64_t {
    if (Str.empty())
      return 0;
    return MD5Hash(Str);
  };
  uint64_t Ret = updateHash(std::to_string(DIL->getLine()));
  Ret ^= updateHash(BB.getName());
  Ret ^= updateHash(DIL->getScope()->getSubprogram()->getLinkageName());
  for (DIL = DIL->getInlinedAt(); DIL; DIL = DIL->getInlinedAt()) {
    Ret ^= updateHash(std::to_string(DIL->getLine()));
    Ret ^= updateHash(DIL->getScope()->getSubprogram()->getLinkageName());
  }
  return Ret;
}

static uint64_t getCallStackHash(const DILocation *DIL) {
  auto hashCombine = [](const uint64_t Seed, const uint64_t Val) {
    std::hash<uint64_t> Hasher;
    return Seed ^ (Hasher(Val) + 0x9e3779b9 + (Seed << 6) + (Seed >> 2));
  };
````
- **L61 EN**: Starts block `auto updateHash = [](const StringRef &Str) -> uint64_t`.
  **L61 CN**: 开始代码块 `auto updateHash = [](const StringRef &Str) -> uint64_t`。
- **L62 EN**: Begins a conditional branch.
  **L62 CN**: 开始一个条件分支。
- **L63 EN**: Returns `0` to the caller.
  **L63 CN**: 向调用者返回 `0`。
- **L64 EN**: Returns `MD5Hash(Str)` to the caller.
  **L64 CN**: 向调用者返回 `MD5Hash(Str)`。
- **L65 EN**: Closes the current scope.
  **L65 CN**: 关闭当前作用域。
- **L66 EN**: Declares function or method `updateHash`.
  **L66 CN**: 声明函数或方法 `updateHash`。
- **L67 EN**: Assigns or initializes `Ret ^`.
  **L67 CN**: 对 `Ret ^` 进行赋值或初始化。
- **L68 EN**: Assigns or initializes `Ret ^`.
  **L68 CN**: 对 `Ret ^` 进行赋值或初始化。
- **L69 EN**: Starts a loop over a sequence or range.
  **L69 CN**: 开始遍历序列或范围的循环。
- **L70 EN**: Declares function or method `updateHash`.
  **L70 CN**: 声明函数或方法 `updateHash`。
- **L71 EN**: Assigns or initializes `Ret ^`.
  **L71 CN**: 对 `Ret ^` 进行赋值或初始化。
- **L72 EN**: Closes the current scope.
  **L72 CN**: 关闭当前作用域。
- **L73 EN**: Returns `Ret` to the caller.
  **L73 CN**: 向调用者返回 `Ret`。
- **L74 EN**: Closes the current scope.
  **L74 CN**: 关闭当前作用域。
- **L75 EN**: Separates nearby statements for readability.
  **L75 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L76 EN**: Begins the definition of `getCallStackHash`.
  **L76 CN**: 开始定义 `getCallStackHash`。
- **L77 EN**: Starts block `auto hashCombine = [](const uint64_t Seed, const uint64_t Val)`.
  **L77 CN**: 开始代码块 `auto hashCombine = [](const uint64_t Seed, const uint64_t Val)`。
- **L78 EN**: Executes statement `std::hash<uint64_t> Hasher;`.
  **L78 CN**: 执行语句 `std::hash<uint64_t> Hasher;`。
- **L79 EN**: Returns `Seed ^ (Hasher(Val) + 0x9e3779b9 + (Seed << 6) + (Seed >> 2))` to the caller.
  **L79 CN**: 向调用者返回 `Seed ^ (Hasher(Val) + 0x9e3779b9 + (Seed << 6) + (Seed >> 2))`。
- **L80 EN**: Closes the current scope.
  **L80 CN**: 关闭当前作用域。

### Lines 81-100

````cpp
  uint64_t Ret = 0;
  for (DIL = DIL->getInlinedAt(); DIL; DIL = DIL->getInlinedAt()) {
    Ret = hashCombine(Ret, xxh3_64bits(ArrayRef<uint8_t>(DIL->getLine())));
    Ret = hashCombine(Ret, xxh3_64bits(DIL->getSubprogramLinkageName()));
  }
  return Ret;
}

// Traverse the CFG and assign FD discriminators. If two instructions
// have the same lineno and discriminator, but residing in different BBs,
// the latter instruction will get a new discriminator value. The new
// discriminator keeps the existing discriminator value but sets new bits
// b/w LowBit and HighBit.
bool MIRAddFSDiscriminators::runOnMachineFunction(MachineFunction &MF) {
  if (!EnableFSDiscriminator)
    return false;

  bool HasPseudoProbe = MF.getFunction().getParent()->getNamedMetadata(
      PseudoProbeDescMetadataName);

````
- **L81 EN**: Assigns or initializes `uint64_t Ret`.
  **L81 CN**: 对 `uint64_t Ret` 进行赋值或初始化。
- **L82 EN**: Starts a loop over a sequence or range.
  **L82 CN**: 开始遍历序列或范围的循环。
- **L83 EN**: Assigns or initializes `Ret`.
  **L83 CN**: 对 `Ret` 进行赋值或初始化。
- **L84 EN**: Assigns or initializes `Ret`.
  **L84 CN**: 对 `Ret` 进行赋值或初始化。
- **L85 EN**: Closes the current scope.
  **L85 CN**: 关闭当前作用域。
- **L86 EN**: Returns `Ret` to the caller.
  **L86 CN**: 向调用者返回 `Ret`。
- **L87 EN**: Closes the current scope.
  **L87 CN**: 关闭当前作用域。
- **L88 EN**: Separates nearby statements for readability.
  **L88 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L89 EN**: Comment documents: `Traverse the CFG and assign FD discriminators. If two instructions`.
  **L89 CN**: 注释说明：`Traverse the CFG and assign FD discriminators. If two instructions`。
- **L90 EN**: Comment documents: `have the same lineno and discriminator, but residing in different BBs,`.
  **L90 CN**: 注释说明：`have the same lineno and discriminator, but residing in different BBs,`。
- **L91 EN**: Comment documents: `the latter instruction will get a new discriminator value. The new`.
  **L91 CN**: 注释说明：`the latter instruction will get a new discriminator value. The new`。
- **L92 EN**: Comment documents: `discriminator keeps the existing discriminator value but sets new bits`.
  **L92 CN**: 注释说明：`discriminator keeps the existing discriminator value but sets new bits`。
- **L93 EN**: Comment documents: `b/w LowBit and HighBit.`.
  **L93 CN**: 注释说明：`b/w LowBit and HighBit.`。
- **L94 EN**: Begins the definition of `runOnMachineFunction`.
  **L94 CN**: 开始定义 `runOnMachineFunction`。
- **L95 EN**: Begins a conditional branch.
  **L95 CN**: 开始一个条件分支。
- **L96 EN**: Returns `false` to the caller.
  **L96 CN**: 向调用者返回 `false`。
- **L97 EN**: Separates nearby statements for readability.
  **L97 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L98 EN**: Continues logic with `bool HasPseudoProbe = MF.getFunction().getParent()->getNamedMetadata(`.
  **L98 CN**: 继续处理逻辑：`bool HasPseudoProbe = MF.getFunction().getParent()->getNamedMetadata(`。
- **L99 EN**: Executes statement `PseudoProbeDescMetadataName);`.
  **L99 CN**: 执行语句 `PseudoProbeDescMetadataName);`。
- **L100 EN**: Separates nearby statements for readability.
  **L100 CN**: 空行，用于分隔相邻语句并提升可读性。

### Lines 101-120

````cpp
  if (!HasPseudoProbe && !MF.getFunction().shouldEmitDebugInfoForProfiling())
    return false;

  bool Changed = false;
  using LocationDiscriminator =
      std::tuple<StringRef, unsigned, unsigned, uint64_t>;
  using BBSet = DenseSet<const MachineBasicBlock *>;
  using LocationDiscriminatorBBMap = DenseMap<LocationDiscriminator, BBSet>;
  using LocationDiscriminatorCurrPassMap =
      DenseMap<LocationDiscriminator, unsigned>;

  LocationDiscriminatorBBMap LDBM;
  LocationDiscriminatorCurrPassMap LDCM;

  // Mask of discriminators before this pass.
  // TODO(xur): simplify this once we switch to ImprovedFSDiscriminator.
  unsigned LowBitTemp = LowBit;
  assert(LowBit > 0 && "LowBit in FSDiscriminator cannot be 0");
  if (ImprovedFSDiscriminator)
    LowBitTemp -= 1;
````
- **L101 EN**: Begins a conditional branch.
  **L101 CN**: 开始一个条件分支。
- **L102 EN**: Returns `false` to the caller.
  **L102 CN**: 向调用者返回 `false`。
- **L103 EN**: Separates nearby statements for readability.
  **L103 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L104 EN**: Assigns or initializes `bool Changed`.
  **L104 CN**: 对 `bool Changed` 进行赋值或初始化。
- **L105 EN**: Continues logic with `using LocationDiscriminator =`.
  **L105 CN**: 继续处理逻辑：`using LocationDiscriminator =`。
- **L106 EN**: Executes statement `std::tuple<StringRef, unsigned, unsigned, uint64_t>;`.
  **L106 CN**: 执行语句 `std::tuple<StringRef, unsigned, unsigned, uint64_t>;`。
- **L107 EN**: Introduces alias or using-declaration `using BBSet = DenseSet<const MachineBasicBlock *>`.
  **L107 CN**: 引入别名或 using 声明 `using BBSet = DenseSet<const MachineBasicBlock *>`。
- **L108 EN**: Introduces alias or using-declaration `using LocationDiscriminatorBBMap = DenseMap<LocationDiscriminator, BBSet>`.
  **L108 CN**: 引入别名或 using 声明 `using LocationDiscriminatorBBMap = DenseMap<LocationDiscriminator, BBSet>`。
- **L109 EN**: Continues logic with `using LocationDiscriminatorCurrPassMap =`.
  **L109 CN**: 继续处理逻辑：`using LocationDiscriminatorCurrPassMap =`。
- **L110 EN**: Executes statement `DenseMap<LocationDiscriminator, unsigned>;`.
  **L110 CN**: 执行语句 `DenseMap<LocationDiscriminator, unsigned>;`。
- **L111 EN**: Separates nearby statements for readability.
  **L111 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L112 EN**: Executes statement `LocationDiscriminatorBBMap LDBM;`.
  **L112 CN**: 执行语句 `LocationDiscriminatorBBMap LDBM;`。
- **L113 EN**: Executes statement `LocationDiscriminatorCurrPassMap LDCM;`.
  **L113 CN**: 执行语句 `LocationDiscriminatorCurrPassMap LDCM;`。
- **L114 EN**: Separates nearby statements for readability.
  **L114 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L115 EN**: Comment documents: `Mask of discriminators before this pass.`.
  **L115 CN**: 注释说明：`Mask of discriminators before this pass.`。
- **L116 EN**: Comment documents: `TODO(xur): simplify this once we switch to ImprovedFSDiscriminator.`.
  **L116 CN**: 注释说明：`TODO(xur): simplify this once we switch to ImprovedFSDiscriminator.`。
- **L117 EN**: Assigns or initializes `unsigned LowBitTemp`.
  **L117 CN**: 对 `unsigned LowBitTemp` 进行赋值或初始化。
- **L118 EN**: Checks an invariant in debug builds.
  **L118 CN**: 在调试构建中检查一个不变量。
- **L119 EN**: Begins a conditional branch.
  **L119 CN**: 开始一个条件分支。
- **L120 EN**: Assigns or initializes `LowBitTemp -`.
  **L120 CN**: 对 `LowBitTemp -` 进行赋值或初始化。

### Lines 121-140

````cpp
  unsigned BitMaskBefore = getN1Bits(LowBitTemp);
  // Mask of discriminators including this pass.
  unsigned BitMaskNow = getN1Bits(HighBit);
  // Mask of discriminators for bits specific to this pass.
  unsigned BitMaskThisPass = BitMaskNow ^ BitMaskBefore;
  unsigned NumNewD = 0;

  LLVM_DEBUG(dbgs() << "MIRAddFSDiscriminators working on Func: "
                    << MF.getFunction().getName() << " Highbit=" << HighBit
                    << "\n");

  for (MachineBasicBlock &BB : MF) {
    for (MachineInstr &I : BB) {
      if (HasPseudoProbe) {
        // Only assign discriminators to pseudo probe instructions. Call
        // instructions are excluded since their dwarf discriminators are used
        // for other purposes, i.e, storing probe ids.
        if (!I.isPseudoProbe())
          continue;
      } else if (ImprovedFSDiscriminator && I.isMetaInstruction()) {
````
- **L121 EN**: Assigns or initializes `unsigned BitMaskBefore`.
  **L121 CN**: 对 `unsigned BitMaskBefore` 进行赋值或初始化。
- **L122 EN**: Comment documents: `Mask of discriminators including this pass.`.
  **L122 CN**: 注释说明：`Mask of discriminators including this pass.`。
- **L123 EN**: Assigns or initializes `unsigned BitMaskNow`.
  **L123 CN**: 对 `unsigned BitMaskNow` 进行赋值或初始化。
- **L124 EN**: Comment documents: `Mask of discriminators for bits specific to this pass.`.
  **L124 CN**: 注释说明：`Mask of discriminators for bits specific to this pass.`。
- **L125 EN**: Assigns or initializes `unsigned BitMaskThisPass`.
  **L125 CN**: 对 `unsigned BitMaskThisPass` 进行赋值或初始化。
- **L126 EN**: Assigns or initializes `unsigned NumNewD`.
  **L126 CN**: 对 `unsigned NumNewD` 进行赋值或初始化。
- **L127 EN**: Separates nearby statements for readability.
  **L127 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L128 EN**: Emits debug-only tracing logic.
  **L128 CN**: 发出仅在调试时启用的跟踪逻辑。
- **L129 EN**: Continues logic with `<< MF.getFunction().getName() << " Highbit=" << HighBit`.
  **L129 CN**: 继续处理逻辑：`<< MF.getFunction().getName() << " Highbit=" << HighBit`。
- **L130 EN**: Executes statement `<< "\n");`.
  **L130 CN**: 执行语句 `<< "\n");`。
- **L131 EN**: Separates nearby statements for readability.
  **L131 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L132 EN**: Starts a loop over a sequence or range.
  **L132 CN**: 开始遍历序列或范围的循环。
- **L133 EN**: Starts a loop over a sequence or range.
  **L133 CN**: 开始遍历序列或范围的循环。
- **L134 EN**: Begins a conditional branch.
  **L134 CN**: 开始一个条件分支。
- **L135 EN**: Comment documents: `Only assign discriminators to pseudo probe instructions. Call`.
  **L135 CN**: 注释说明：`Only assign discriminators to pseudo probe instructions. Call`。
- **L136 EN**: Comment documents: `instructions are excluded since their dwarf discriminators are used`.
  **L136 CN**: 注释说明：`instructions are excluded since their dwarf discriminators are used`。
- **L137 EN**: Comment documents: `for other purposes, i.e, storing probe ids.`.
  **L137 CN**: 注释说明：`for other purposes, i.e, storing probe ids.`。
- **L138 EN**: Begins a conditional branch.
  **L138 CN**: 开始一个条件分支。
- **L139 EN**: Skips to the next loop iteration.
  **L139 CN**: 跳到下一次循环迭代。
- **L140 EN**: Starts block `} else if (ImprovedFSDiscriminator && I.isMetaInstruction())`.
  **L140 CN**: 开始代码块 `} else if (ImprovedFSDiscriminator && I.isMetaInstruction())`。

### Lines 141-160

````cpp
        continue;
      }
      const DILocation *DIL = I.getDebugLoc().get();
      if (!DIL)
        continue;

      // Use the id of pseudo probe to compute the discriminator.
      unsigned LineNo =
          I.isPseudoProbe() ? I.getOperand(1).getImm() : DIL->getLine();
      if (LineNo == 0)
        continue;
      unsigned Discriminator = DIL->getDiscriminator();
      // Clean up discriminators for pseudo probes at the first FS discriminator
      // pass as their discriminators should not ever be used.
      if ((Pass == FSDiscriminatorPass::Pass1) && I.isPseudoProbe()) {
        Discriminator = 0;
        I.setDebugLoc(DIL->cloneWithDiscriminator(0));
      }
      uint64_t CallStackHashVal = 0;
      if (ImprovedFSDiscriminator)
````
- **L141 EN**: Skips to the next loop iteration.
  **L141 CN**: 跳到下一次循环迭代。
- **L142 EN**: Closes the current scope.
  **L142 CN**: 关闭当前作用域。
- **L143 EN**: Assigns or initializes `const DILocation *DIL`.
  **L143 CN**: 对 `const DILocation *DIL` 进行赋值或初始化。
- **L144 EN**: Begins a conditional branch.
  **L144 CN**: 开始一个条件分支。
- **L145 EN**: Skips to the next loop iteration.
  **L145 CN**: 跳到下一次循环迭代。
- **L146 EN**: Separates nearby statements for readability.
  **L146 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L147 EN**: Comment documents: `Use the id of pseudo probe to compute the discriminator.`.
  **L147 CN**: 注释说明：`Use the id of pseudo probe to compute the discriminator.`。
- **L148 EN**: Continues logic with `unsigned LineNo =`.
  **L148 CN**: 继续处理逻辑：`unsigned LineNo =`。
- **L149 EN**: Executes statement `I.isPseudoProbe() ? I.getOperand(1).getImm() : DIL->getLine();`.
  **L149 CN**: 执行语句 `I.isPseudoProbe() ? I.getOperand(1).getImm() : DIL->getLine();`。
- **L150 EN**: Begins a conditional branch.
  **L150 CN**: 开始一个条件分支。
- **L151 EN**: Skips to the next loop iteration.
  **L151 CN**: 跳到下一次循环迭代。
- **L152 EN**: Assigns or initializes `unsigned Discriminator`.
  **L152 CN**: 对 `unsigned Discriminator` 进行赋值或初始化。
- **L153 EN**: Comment documents: `Clean up discriminators for pseudo probes at the first FS discriminator`.
  **L153 CN**: 注释说明：`Clean up discriminators for pseudo probes at the first FS discriminator`。
- **L154 EN**: Comment documents: `pass as their discriminators should not ever be used.`.
  **L154 CN**: 注释说明：`pass as their discriminators should not ever be used.`。
- **L155 EN**: Begins a conditional branch.
  **L155 CN**: 开始一个条件分支。
- **L156 EN**: Assigns or initializes `Discriminator`.
  **L156 CN**: 对 `Discriminator` 进行赋值或初始化。
- **L157 EN**: Executes statement `I.setDebugLoc(DIL->cloneWithDiscriminator(0));`.
  **L157 CN**: 执行语句 `I.setDebugLoc(DIL->cloneWithDiscriminator(0));`。
- **L158 EN**: Closes the current scope.
  **L158 CN**: 关闭当前作用域。
- **L159 EN**: Assigns or initializes `uint64_t CallStackHashVal`.
  **L159 CN**: 对 `uint64_t CallStackHashVal` 进行赋值或初始化。
- **L160 EN**: Begins a conditional branch.
  **L160 CN**: 开始一个条件分支。

### Lines 161-180

````cpp
        CallStackHashVal = getCallStackHash(DIL);

      LocationDiscriminator LD{DIL->getFilename(), LineNo, Discriminator,
                               CallStackHashVal};
      auto &BBMap = LDBM[LD];
      auto R = BBMap.insert(&BB);
      if (BBMap.size() == 1)
        continue;

      unsigned DiscriminatorCurrPass;
      DiscriminatorCurrPass = R.second ? ++LDCM[LD] : LDCM[LD];
      DiscriminatorCurrPass = DiscriminatorCurrPass << LowBit;
      if (!ImprovedFSDiscriminator)
        DiscriminatorCurrPass += getCallStackHashV0(BB, I, DIL);
      DiscriminatorCurrPass &= BitMaskThisPass;
      unsigned NewD = Discriminator | DiscriminatorCurrPass;
      const auto *const NewDIL = DIL->cloneWithDiscriminator(NewD);
      if (!NewDIL) {
        LLVM_DEBUG(dbgs() << "Could not encode discriminator: "
                          << DIL->getFilename() << ":" << DIL->getLine() << ":"
````
- **L161 EN**: Assigns or initializes `CallStackHashVal`.
  **L161 CN**: 对 `CallStackHashVal` 进行赋值或初始化。
- **L162 EN**: Separates nearby statements for readability.
  **L162 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L163 EN**: Continues logic with `LocationDiscriminator LD{DIL->getFilename(), LineNo, Discriminator,`.
  **L163 CN**: 继续处理逻辑：`LocationDiscriminator LD{DIL->getFilename(), LineNo, Discriminator,`。
- **L164 EN**: Executes statement `CallStackHashVal};`.
  **L164 CN**: 执行语句 `CallStackHashVal};`。
- **L165 EN**: Assigns or initializes `auto &BBMap`.
  **L165 CN**: 对 `auto &BBMap` 进行赋值或初始化。
- **L166 EN**: Assigns or initializes `auto R`.
  **L166 CN**: 对 `auto R` 进行赋值或初始化。
- **L167 EN**: Begins a conditional branch.
  **L167 CN**: 开始一个条件分支。
- **L168 EN**: Skips to the next loop iteration.
  **L168 CN**: 跳到下一次循环迭代。
- **L169 EN**: Separates nearby statements for readability.
  **L169 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L170 EN**: Executes statement `unsigned DiscriminatorCurrPass;`.
  **L170 CN**: 执行语句 `unsigned DiscriminatorCurrPass;`。
- **L171 EN**: Assigns or initializes `DiscriminatorCurrPass`.
  **L171 CN**: 对 `DiscriminatorCurrPass` 进行赋值或初始化。
- **L172 EN**: Assigns or initializes `DiscriminatorCurrPass`.
  **L172 CN**: 对 `DiscriminatorCurrPass` 进行赋值或初始化。
- **L173 EN**: Begins a conditional branch.
  **L173 CN**: 开始一个条件分支。
- **L174 EN**: Assigns or initializes `DiscriminatorCurrPass +`.
  **L174 CN**: 对 `DiscriminatorCurrPass +` 进行赋值或初始化。
- **L175 EN**: Assigns or initializes `DiscriminatorCurrPass &`.
  **L175 CN**: 对 `DiscriminatorCurrPass &` 进行赋值或初始化。
- **L176 EN**: Assigns or initializes `unsigned NewD`.
  **L176 CN**: 对 `unsigned NewD` 进行赋值或初始化。
- **L177 EN**: Assigns or initializes `const auto *const NewDIL`.
  **L177 CN**: 对 `const auto *const NewDIL` 进行赋值或初始化。
- **L178 EN**: Begins a conditional branch.
  **L178 CN**: 开始一个条件分支。
- **L179 EN**: Emits debug-only tracing logic.
  **L179 CN**: 发出仅在调试时启用的跟踪逻辑。
- **L180 EN**: Continues logic with `<< DIL->getFilename() << ":" << DIL->getLine() << ":"`.
  **L180 CN**: 继续处理逻辑：`<< DIL->getFilename() << ":" << DIL->getLine() << ":"`。

### Lines 181-200

````cpp
                          << DIL->getColumn() << ":" << Discriminator << " "
                          << I << "\n");
        continue;
      }

      I.setDebugLoc(NewDIL);
      NumNewD++;
      LLVM_DEBUG(dbgs() << DIL->getFilename() << ":" << DIL->getLine() << ":"
                        << DIL->getColumn() << ": add FS discriminator, from "
                        << Discriminator << " -> " << NewD << "\n");
      Changed = true;
    }
  }

  if (Changed) {
    createFSDiscriminatorVariable(MF.getFunction().getParent());
    LLVM_DEBUG(dbgs() << "Num of FS Discriminators: " << NumNewD << "\n");
    (void) NumNewD;
  }

````
- **L181 EN**: Continues logic with `<< DIL->getColumn() << ":" << Discriminator << " "`.
  **L181 CN**: 继续处理逻辑：`<< DIL->getColumn() << ":" << Discriminator << " "`。
- **L182 EN**: Executes statement `<< I << "\n");`.
  **L182 CN**: 执行语句 `<< I << "\n");`。
- **L183 EN**: Skips to the next loop iteration.
  **L183 CN**: 跳到下一次循环迭代。
- **L184 EN**: Closes the current scope.
  **L184 CN**: 关闭当前作用域。
- **L185 EN**: Separates nearby statements for readability.
  **L185 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L186 EN**: Executes statement `I.setDebugLoc(NewDIL);`.
  **L186 CN**: 执行语句 `I.setDebugLoc(NewDIL);`。
- **L187 EN**: Executes statement `NumNewD++;`.
  **L187 CN**: 执行语句 `NumNewD++;`。
- **L188 EN**: Emits debug-only tracing logic.
  **L188 CN**: 发出仅在调试时启用的跟踪逻辑。
- **L189 EN**: Continues logic with `<< DIL->getColumn() << ": add FS discriminator, from "`.
  **L189 CN**: 继续处理逻辑：`<< DIL->getColumn() << ": add FS discriminator, from "`。
- **L190 EN**: Executes statement `<< Discriminator << " -> " << NewD << "\n");`.
  **L190 CN**: 执行语句 `<< Discriminator << " -> " << NewD << "\n");`。
- **L191 EN**: Assigns or initializes `Changed`.
  **L191 CN**: 对 `Changed` 进行赋值或初始化。
- **L192 EN**: Closes the current scope.
  **L192 CN**: 关闭当前作用域。
- **L193 EN**: Closes the current scope.
  **L193 CN**: 关闭当前作用域。
- **L194 EN**: Separates nearby statements for readability.
  **L194 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L195 EN**: Begins a conditional branch.
  **L195 CN**: 开始一个条件分支。
- **L196 EN**: Executes statement `createFSDiscriminatorVariable(MF.getFunction().getParent());`.
  **L196 CN**: 执行语句 `createFSDiscriminatorVariable(MF.getFunction().getParent());`。
- **L197 EN**: Emits debug-only tracing logic.
  **L197 CN**: 发出仅在调试时启用的跟踪逻辑。
- **L198 EN**: Executes statement `(void) NumNewD;`.
  **L198 CN**: 执行语句 `(void) NumNewD;`。
- **L199 EN**: Closes the current scope.
  **L199 CN**: 关闭当前作用域。
- **L200 EN**: Separates nearby statements for readability.
  **L200 CN**: 空行，用于分隔相邻语句并提升可读性。

### Lines 201-202

````cpp
  return Changed;
}
````
- **L201 EN**: Returns `Changed` to the caller.
  **L201 CN**: 向调用者返回 `Changed`。
- **L202 EN**: Closes the current scope.
  **L202 CN**: 关闭当前作用域。

## Key Concepts / 关键概念
- **Machine function processing** / **MachineFunction 处理**
- **Machine basic block handling** / **MachineBasicBlock 处理**
- **Machine instruction manipulation** / **机器指令操作**
- **Stack frame management** / **栈帧管理**
- **Debug information handling** / **调试信息处理**
- **Pseudo-instruction expansion** / **伪指令展开**

## Dependencies / 依赖关系
- **LLVM headers / LLVM 头文件**: `llvm/CodeGen/MIRFSDiscriminator.h`, `llvm/ADT/DenseMap.h`, `llvm/ADT/DenseSet.h`, `llvm/Analysis/BlockFrequencyInfoImpl.h`, `llvm/CodeGen/MIRFSDiscriminatorOptions.h`, `llvm/CodeGen/Passes.h`, `llvm/IR/DebugInfoMetadata.h`, `llvm/IR/Function.h`, `llvm/IR/Module.h`, `llvm/IR/PseudoProbe.h`, `llvm/InitializePasses.h`, `llvm/Support/CommandLine.h`, `llvm/Support/Debug.h`, `llvm/Support/raw_ostream.h`, `llvm/Support/xxhash.h`, `llvm/Transforms/Utils/SampleProfileLoaderBaseUtil.h`
- **Primary dependency domains / 主要依赖域**: CodeGen internals, LLVM support utilities, and C++ runtime helpers. / CodeGen 内部组件、LLVM 支持工具以及 C++ 运行时辅助设施。
