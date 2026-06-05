# CommandLineOpts.h — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `bolt/include/bolt/Utils/CommandLineOpts.h`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: BOLT CLI options. It also sits in code that belongs to the BOLT post-link optimizer subsystem. / 该文件属于 BOLT 链接后优化器子系统。 源码头部说明其职责是：BOLT CLI options。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12

```cpp
//===- bolt/Utils/CommandLineOpts.h - BOLT CLI options ----------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// BOLT CLI options
//
//===----------------------------------------------------------------------===//
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 13-23

```cpp
#ifndef BOLT_UTILS_COMMAND_LINE_OPTS_H
#define BOLT_UTILS_COMMAND_LINE_OPTS_H

#include "llvm/Support/CommandLine.h"

namespace llvm {
namespace bolt {
class BinaryFunction;
}
} // namespace llvm
```

- EN: Pulls in 1 header(s) from LLVM dependencies needed by this range. Establishes header-guard or prologue macros so the file can be compiled safely. Works inside namespace scope `llvm`, `bolt` to organize symbols. Introduces type definitions such as `BinaryFunction`.
- CN: 这里引入 1 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里建立头文件保护或前导宏，保证源码可被安全编译与重复包含控制。这里位于命名空间 `llvm`, `bolt` 中，用于组织符号作用域。这里引入类型定义，例如 `BinaryFunction`。

### Lines 24-31

```cpp
namespace opts {

enum HeatmapModeKind {
  HM_None = 0,
  HM_Exclusive, // llvm-bolt-heatmap
  HM_Optional   // perf2bolt --heatmap
};
```

- EN: Works inside namespace scope `opts` to organize symbols. Defines enumerations such as `HeatmapModeKind` to encode states or modes. Notable symbols here include `HeatmapModeKind`, `opts`.
- CN: 这里位于命名空间 `opts` 中，用于组织符号作用域。这里定义枚举 `HeatmapModeKind`，用于表达状态或模式。这里较值得关注的符号包括 `HeatmapModeKind`, `opts`。

### Lines 32-49

```cpp
/// Strategy used to partition blocks into fragments.
enum SplitFunctionsStrategy : char {
  /// Split each function into a hot and cold fragment using profiling
  /// information.
  Profile2 = 0,
  /// Split each function into a hot, warm, and cold fragment using
  /// profiling information.
  CDSplit,
  /// Split each function into a hot and cold fragment at a randomly chosen
  /// split point (ignoring any available profiling information).
  Random2,
  /// Split each function into N fragments at a randomly chosen split points
  /// (ignoring any available profiling information).
  RandomN,
  /// Split all basic blocks of each function into fragments such that each
  /// fragment contains exactly a single basic block.
  All
};
```

- EN: Defines enumerations such as `SplitFunctionsStrategy` to encode states or modes. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `SplitFunctionsStrategy`.
- CN: 这里定义枚举 `SplitFunctionsStrategy`，用于表达状态或模式。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `SplitFunctionsStrategy`。

### Lines 50-59

```cpp

using HeatmapBlockSizes = std::vector<unsigned>;
struct HeatmapBlockSpecParser : public llvm::cl::parser<HeatmapBlockSizes> {
  explicit HeatmapBlockSpecParser(llvm::cl::Option &O)
      : llvm::cl::parser<HeatmapBlockSizes>(O) {}
  // Return true on error.
  bool parse(llvm::cl::Option &O, llvm::StringRef ArgName, llvm::StringRef Arg,
             HeatmapBlockSizes &Val);
};
```

- EN: Introduces type definitions such as `HeatmapBlockSpecParser`. Declares or implements routines including `HeatmapBlockSpecParser`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `HeatmapBlockSpecParser`.
- CN: 这里引入类型定义，例如 `HeatmapBlockSpecParser`。这里声明或实现函数，例如 `HeatmapBlockSpecParser`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `HeatmapBlockSpecParser`。

### Lines 60-72

```cpp
extern HeatmapModeKind HeatmapMode;
extern bool BinaryAnalysisMode;

extern llvm::cl::OptionCategory BoltCategory;
extern llvm::cl::OptionCategory BoltDiffCategory;
extern llvm::cl::OptionCategory BoltOptCategory;
extern llvm::cl::OptionCategory BoltRelocCategory;
extern llvm::cl::OptionCategory BoltOutputCategory;
extern llvm::cl::OptionCategory AggregatorCategory;
extern llvm::cl::OptionCategory BoltInstrCategory;
extern llvm::cl::OptionCategory HeatmapCategory;
extern llvm::cl::OptionCategory BinaryAnalysisCategory;
```

- EN: Continues the surrounding implementation, wiring declarations, state, and helper logic together.
- CN: 这里继续展开周边实现，衔接声明、状态与辅助逻辑。

### Lines 73-90

```cpp
extern llvm::cl::opt<unsigned> AlignText;
extern llvm::cl::opt<unsigned> AlignFunctions;
extern llvm::cl::opt<bool> AggregateOnly;
extern llvm::cl::opt<bool> ArmSPE;
extern llvm::cl::opt<unsigned> BucketsPerLine;
extern llvm::cl::opt<bool> CompactCodeModel;
extern llvm::cl::opt<bool> DiffOnly;
extern llvm::cl::opt<bool> EnableBAT;
extern llvm::cl::opt<bool> EqualizeBBCounts;
extern llvm::cl::opt<bool> ForcePatch;
extern llvm::cl::opt<bool> RemoveSymtab;
extern llvm::cl::opt<unsigned> ExecutionCountThreshold;
extern llvm::cl::opt<HeatmapBlockSizes, false, HeatmapBlockSpecParser>
    HeatmapBlock;
extern llvm::cl::opt<unsigned long long> HeatmapMaxAddress;
extern llvm::cl::opt<unsigned long long> HeatmapMinAddress;
extern llvm::cl::opt<bool> HeatmapPrintMappings;
extern llvm::cl::opt<std::string> HeatmapOutput;
```

- EN: Continues the surrounding implementation, wiring declarations, state, and helper logic together.
- CN: 这里继续展开周边实现，衔接声明、状态与辅助逻辑。

### Lines 91-102

```cpp
extern llvm::cl::opt<bool> HotData;
extern llvm::cl::opt<bool> HotFunctionsAtEnd;
extern llvm::cl::opt<bool> HotText;
extern llvm::cl::opt<bool> Hugify;
extern llvm::cl::opt<bool> Instrument;
extern llvm::cl::opt<std::string> OutputFilename;
extern llvm::cl::opt<std::string> PerfData;
extern llvm::cl::opt<bool> PrintCacheMetrics;
extern llvm::cl::opt<bool> PrintSections;
extern llvm::cl::opt<bool> UpdateBranchProtection;
extern llvm::cl::opt<SplitFunctionsStrategy> SplitStrategy;
```

- EN: Continues the surrounding implementation, wiring declarations, state, and helper logic together.
- CN: 这里继续展开周边实现，衔接声明、状态与辅助逻辑。

### Lines 103-113

```cpp
// The format to use with -o in aggregation mode (perf2bolt)
enum ProfileFormatKind { PF_Fdata, PF_YAML };

extern llvm::cl::opt<ProfileFormatKind> ProfileFormat;
extern llvm::cl::opt<bool> ShowDensity;
extern llvm::cl::opt<bool> SplitEH;
extern llvm::cl::opt<bool> StrictMode;
extern llvm::cl::opt<bool> TimeOpts;
extern llvm::cl::opt<bool> UseOldText;
extern llvm::cl::opt<bool> UpdateDebugSections;
```

- EN: Defines enumerations such as `ProfileFormatKind` to encode states or modes. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `ProfileFormatKind`.
- CN: 这里定义枚举 `ProfileFormatKind`，用于表达状态或模式。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `ProfileFormatKind`。

### Lines 114-126

```cpp
// The default verbosity level (0) is pretty terse, level 1 is fairly
// verbose and usually prints some informational message for every
// function processed.  Level 2 is for the noisiest of messages and
// often prints a message per basic block.
// Error messages should never be suppressed by the verbosity level.
// Only warnings and info messages should be affected.
//
// The rationale behind stream usage is as follows:
// outs() for info and debugging controlled by command line flags.
// errs() for errors and warnings.
// dbgs() for output within DEBUG().
extern llvm::cl::opt<unsigned> Verbosity;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 127-144

```cpp
/// Return true if we should process all functions in the binary.
bool processAllFunctions();

/// Return true if we should dump dot graphs for the given function.
bool shouldDumpDot(const llvm::bolt::BinaryFunction &Function);

/// Bitmask representing a subset of possible gadget kinds.
enum GadgetKindBitmask : unsigned {
  /// Scan for unprotected backward control-flow (return instructions).
  GS_PTRAUTH_RETURN_TARGETS = (1 << 0),
  /// Scan for tail calls performed with untrusted link register.
  GS_PTRAUTH_TAIL_CALLS = (1 << 1),
  /// Scan for unprotected forward control-flow (branch and call instructions).
  GS_PTRAUTH_BRANCH_AND_CALL_TARGETS = (1 << 2),
  /// Scan for signing oracles.
  GS_PTRAUTH_SIGN_ORACLES = (1 << 3),
  /// Scan for authentication oracles.
  GS_PTRAUTH_AUTH_ORACLES = (1 << 4),
```

- EN: Defines enumerations such as `GadgetKindBitmask` to encode states or modes. Declares or implements routines including `processAllFunctions`, `shouldDumpDot`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `GadgetKindBitmask`, `processAllFunctions`, `shouldDumpDot`.
- CN: 这里定义枚举 `GadgetKindBitmask`，用于表达状态或模式。这里声明或实现函数，例如 `processAllFunctions`, `shouldDumpDot`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `GadgetKindBitmask`, `processAllFunctions`, `shouldDumpDot`。

### Lines 145-154

```cpp

  /// Scan for all Pointer Authentication issues.
  GS_PTRAUTH_ALL_MASK = GS_PTRAUTH_RETURN_TARGETS | GS_PTRAUTH_TAIL_CALLS |
                        GS_PTRAUTH_BRANCH_AND_CALL_TARGETS |
                        GS_PTRAUTH_SIGN_ORACLES | GS_PTRAUTH_AUTH_ORACLES,

  /// Run all implemented scanners.
  GS_ALL_MASK = GS_PTRAUTH_ALL_MASK,
};
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 155-162

```cpp
} // namespace opts

namespace llvm {
namespace bolt {
extern const char *BoltRevision;
}
} // namespace llvm
```

- EN: Works inside namespace scope `opts`, `llvm`, `bolt` to organize symbols. Notable symbols here include `opts`, `llvm`, `bolt`.
- CN: 这里位于命名空间 `opts`, `llvm`, `bolt` 中，用于组织符号作用域。这里较值得关注的符号包括 `opts`, `llvm`, `bolt`。

### Lines 163-163

```cpp
#endif
```

- EN: Uses conditional compilation to choose platform- or feature-specific code paths. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里使用条件编译，根据平台或特性切换不同代码路径。这里的注释记录了设计假设、不变量或使用说明。

## Key Concepts / 关键概念

- `BinaryFunction`: class or struct interface / 类或结构体接口
- `HeatmapBlockSpecParser`: class or struct interface / 类或结构体接口
- `HeatmapModeKind`: enumeration of modes or states / 模式或状态枚举
- `SplitFunctionsStrategy`: enumeration of modes or states / 模式或状态枚举
- `ProfileFormatKind`: enumeration of modes or states / 模式或状态枚举
- `HeatmapBlockSpecParser`: function or method entry point / 函数或方法入口
- `processAllFunctions`: function or method entry point / 函数或方法入口
- `shouldDumpDot`: function or method entry point / 函数或方法入口

## Dependencies / 依赖关系

- LLVM headers / LLVM 头文件: `llvm/Support/CommandLine.h`
- Directory context / 目录上下文: `bolt/include/bolt/Utils` neighbors usually cooperate with this file to provide the surrounding subsystem / `bolt/include/bolt/Utils` 下的相邻文件通常与本文件协作组成对应子系统
