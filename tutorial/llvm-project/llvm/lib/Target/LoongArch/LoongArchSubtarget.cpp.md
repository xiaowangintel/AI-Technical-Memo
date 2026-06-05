# LoongArchSubtarget.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/LoongArch/LoongArchSubtarget.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file models CPU/subtarget features and scheduling properties for the LoongArch backend.
- **用途 (CN)**: 该文件用于 LoongArch 后端，负责建模 CPU/子目标特性与调度属性。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12 / 第 1-12 行
```cpp
   1: //===-- LoongArchSubtarget.cpp - LoongArch Subtarget Information -*- C++ -*--=//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file implements the LoongArch specific subclass of TargetSubtargetInfo.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
```
- **EN**: This range is mainly descriptive commentary or banner text that frames the file before executable definitions begin.
- **CN**: 这一段主要是说明性注释或横幅文本，用于在可执行定义开始前说明文件背景。

### Lines 13-24 / 第 13-24 行
```cpp
  13: #include "LoongArchSubtarget.h"
  14: #include "LoongArchFrameLowering.h"
  15: #include "LoongArchSelectionDAGInfo.h"
  16: #include "MCTargetDesc/LoongArchBaseInfo.h"
  17: 
  18: using namespace llvm;
  19: 
  20: #define DEBUG_TYPE "loongarch-subtarget"
  21: 
  22: #define GET_SUBTARGETINFO_TARGET_DESC
  23: #define GET_SUBTARGETINFO_CTOR
  24: #include "LoongArchGenSubtargetInfo.inc"
```
- **EN**: It imports dependencies such as `LoongArchSubtarget.h`, `LoongArchFrameLowering.h`, `LoongArchSelectionDAGInfo.h`, `LoongArchBaseInfo.h`, `LoongArchGenSubtargetInfo.inc` that expose the LLVM and target interfaces used in later logic. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file. Generated `.inc` fragments are pulled in here, a common LLVM technique for TableGen-produced code.
- **CN**: 它引入了 `LoongArchSubtarget.h`, `LoongArchFrameLowering.h`, `LoongArchSelectionDAGInfo.h`, `LoongArchBaseInfo.h`, `LoongArchGenSubtargetInfo.inc` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。 这里引入了生成的 `.inc` 片段，这是 LLVM 使用 TableGen 产物的常见方式。

### Lines 25-36 / 第 25-36 行
```cpp
  25: 
  26: static cl::opt<bool> UseAA("loongarch-use-aa", cl::init(true),
  27:                            cl::desc("Enable the use of AA during codegen."));
  28: 
  29: void LoongArchSubtarget::anchor() {}
  30: 
  31: // Enable use of alias analysis during code generation (during MI scheduling,
  32: // DAGCombine, etc.).
  33: bool LoongArchSubtarget::useAA() const { return UseAA; }
  34: 
  35: LoongArchSubtarget &LoongArchSubtarget::initializeSubtargetDependencies(
  36:     const Triple &TT, StringRef CPU, StringRef TuneCPU, StringRef FS,
```
- **EN**: The range implements or declares functions including `LoongArchSubtarget::anchor`.
- **CN**: 这一段实现或声明了 `LoongArchSubtarget::anchor` 等函数。

### Lines 37-48 / 第 37-48 行
```cpp
  37:     StringRef ABIName) {
  38:   bool Is64Bit = TT.isArch64Bit();
  39:   if (CPU.empty() || CPU == "generic")
  40:     CPU = Is64Bit ? "generic-la64" : "generic-la32";
  41: 
  42:   if (TuneCPU.empty())
  43:     TuneCPU = CPU;
  44: 
  45:   ParseSubtargetFeatures(CPU, TuneCPU, FS);
  46:   initializeProperties(TuneCPU);
  47:   if (Is64Bit) {
  48:     GRLenVT = MVT::i64;
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 49-60 / 第 49-60 行
```cpp
  49:     GRLen = 64;
  50:   }
  51: 
  52:   if (HasLA32 == HasLA64)
  53:     report_fatal_error("Please use one feature of 32bit and 64bit.");
  54: 
  55:   if (Is64Bit && HasLA32)
  56:     report_fatal_error("Feature 32bit should be used for loongarch32 target.");
  57: 
  58:   if (!Is64Bit && HasLA64)
  59:     report_fatal_error("Feature 64bit should be used for loongarch64 target.");
  60: 
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 61-72 / 第 61-72 行
```cpp
  61:   TargetABI = LoongArchABI::computeTargetABI(TT, getFeatureBits(), ABIName);
  62: 
  63:   return *this;
  64: }
  65: 
  66: void LoongArchSubtarget::initializeProperties(StringRef TuneCPU) {
  67:   // Initialize CPU specific properties. We should add a tablegen feature for
  68:   // this in the future so we can specify it together with the subtarget
  69:   // features.
  70: 
  71:   // TODO: Check TuneCPU and override defaults (that are for LA464) once we
  72:   // support optimizing for more uarchs.
```
- **EN**: The range implements or declares functions including `LoongArchSubtarget::initializeProperties`.
- **CN**: 这一段实现或声明了 `LoongArchSubtarget::initializeProperties` 等函数。

### Lines 73-84 / 第 73-84 行
```cpp
  73: 
  74:   // Default to the alignment settings empirically confirmed to perform best
  75:   // on LA464, with 4-wide instruction fetch and decode stages. These settings
  76:   // can also be overridden in initializeProperties.
  77:   //
  78:   // We default to such higher-than-minimum alignments because we assume that:
  79:   //
  80:   // * these settings should benefit most existing uarchs/users,
  81:   // * future general-purpose LoongArch cores are likely to have issue widths
  82:   //   equal to or wider than 4,
  83:   // * instruction sequences best for LA464 should not pessimize other future
  84:   //   uarchs, and
```
- **EN**: This range is mainly descriptive commentary or banner text that frames the file before executable definitions begin.
- **CN**: 这一段主要是说明性注释或横幅文本，用于在可执行定义开始前说明文件背景。

### Lines 85-96 / 第 85-96 行
```cpp
  85:   // * narrower cores would not suffer much (aside from slightly increased
  86:   //   ICache footprint maybe), compared to the gains everywhere else.
  87:   PrefFunctionAlignment = Align(32);
  88:   PrefLoopAlignment = Align(16);
  89:   MaxBytesForAlignment = 16;
  90: }
  91: 
  92: LoongArchSubtarget::LoongArchSubtarget(const Triple &TT, StringRef CPU,
  93:                                        StringRef TuneCPU, StringRef FS,
  94:                                        StringRef ABIName,
  95:                                        const TargetMachine &TM)
  96:     : LoongArchGenSubtargetInfo(TT, CPU, TuneCPU, FS),
```
- **EN**: This span continues the file's main responsibility: this file models CPU/subtarget features and scheduling properties for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 97-107 / 第 97-107 行
```cpp
  97:       FrameLowering(
  98:           initializeSubtargetDependencies(TT, CPU, TuneCPU, FS, ABIName)),
  99:       InstrInfo(*this), TLInfo(TM, *this) {
 100:   TSInfo = std::make_unique<LoongArchSelectionDAGInfo>();
 101: }
 102: 
 103: LoongArchSubtarget::~LoongArchSubtarget() = default;
 104: 
 105: const SelectionDAGTargetInfo *LoongArchSubtarget::getSelectionDAGInfo() const {
 106:   return TSInfo.get();
 107: }
```
- **EN**: The range implements or declares functions including `FrameLowering`.
- **CN**: 这一段实现或声明了 `FrameLowering` 等函数。

## Key Concepts / 关键概念
- **SelectionDAG lowering**: Bridges LLVM IR and target-specific DAG nodes/instructions. / 连接 LLVM IR 与目标相关 DAG 节点/指令。
- **Instruction metadata**: Captures opcodes, operands, patterns, and helper routines. / 描述操作码、操作数、匹配模式和辅助例程。
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。
- **Frame lowering**: Builds stack frames, callee-save handling, and prologue/epilogue sequences. / 构建栈帧、被调用者保存寄存器处理以及序言/尾声序列。
- **Target machine**: Owns data layout, pass configuration, and backend-wide policy. / 管理数据布局、Pass 配置以及整个后端策略。

## Dependencies / 依赖关系
- `LoongArchSubtarget.h`
- `LoongArchFrameLowering.h`
- `LoongArchSelectionDAGInfo.h`
- `MCTargetDesc/LoongArchBaseInfo.h`
- `LoongArchGenSubtargetInfo.inc`
