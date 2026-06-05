# SystemZSubtarget.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/SystemZSubtarget.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file models CPU/subtarget features and scheduling properties for the SystemZ backend.
- **用途 (CN)**: 该文件用于 SystemZ 后端，负责建模 CPU/子目标特性与调度属性。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行
```cpp
   1: //===-- SystemZSubtarget.cpp - SystemZ subtarget information --------------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
   9: #include "SystemZSubtarget.h"
  10: #include "llvm/CodeGen/TargetLoweringObjectFileImpl.h"
  11: #include "llvm/IR/GlobalVariable.h"
  12: #include "llvm/Target/TargetMachine.h"
  13: 
  14: using namespace llvm;
  15: 
  16: #define DEBUG_TYPE "systemz-subtarget"
  17: 
  18: #define GET_SUBTARGETINFO_TARGET_DESC
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `SystemZSubtarget.h`, `TargetLoweringObjectFileImpl.h`, `GlobalVariable.h`, `TargetMachine.h` that expose the LLVM and target interfaces used in later logic. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `SystemZSubtarget.h`, `TargetLoweringObjectFileImpl.h`, `GlobalVariable.h`, `TargetMachine.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 19-36 / 第 19-36 行
```cpp
  19: #define GET_SUBTARGETINFO_CTOR
  20: #include "SystemZGenSubtargetInfo.inc"
  21: 
  22: static cl::opt<bool> UseSubRegLiveness(
  23:     "systemz-subreg-liveness",
  24:     cl::desc("Enable subregister liveness tracking for SystemZ (experimental)"),
  25:     cl::Hidden);
  26: 
  27: // Pin the vtable to this file.
  28: void SystemZSubtarget::anchor() {}
  29: 
  30: SystemZSubtarget &SystemZSubtarget::initializeSubtargetDependencies(
  31:     StringRef CPU, StringRef TuneCPU, StringRef FS) {
  32:   if (CPU.empty())
  33:     CPU = "generic";
  34:   if (TuneCPU.empty())
  35:     TuneCPU = CPU;
  36:   // Parse features string.
```
- **EN**: It imports dependencies such as `SystemZGenSubtargetInfo.inc` that expose the LLVM and target interfaces used in later logic. The range implements or declares functions including `SystemZSubtarget::anchor`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 它引入了 `SystemZGenSubtargetInfo.inc` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段实现或声明了 `SystemZSubtarget::anchor` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 37-54 / 第 37-54 行
```cpp
  37:   ParseSubtargetFeatures(CPU, TuneCPU, FS);
  38: 
  39:   // -msoft-float implies -mno-vx.
  40:   if (HasSoftFloat)
  41:     HasVector = false;
  42: 
  43:   // -mno-vx implicitly disables all vector-related features.
  44:   if (!HasVector) {
  45:     HasVectorEnhancements1 = false;
  46:     HasVectorEnhancements2 = false;
  47:     HasVectorEnhancements3 = false;
  48:     HasVectorPackedDecimal = false;
  49:     HasVectorPackedDecimalEnhancement = false;
  50:     HasVectorPackedDecimalEnhancement2 = false;
  51:     HasVectorPackedDecimalEnhancement3 = false;
  52:   }
  53: 
  54:   return *this;
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 55-72 / 第 55-72 行
```cpp
  55: }
  56: 
  57: SystemZCallingConventionRegisters *
  58: SystemZSubtarget::initializeSpecialRegisters() {
  59:   if (isTargetXPLINK64())
  60:     return new SystemZXPLINK64Registers;
  61:   else if (isTargetELF())
  62:     return new SystemZELFRegisters;
  63:   llvm_unreachable("Invalid Calling Convention. Cannot initialize Special "
  64:                    "Call Registers!");
  65: }
  66: 
  67: SystemZSubtarget::SystemZSubtarget(const Triple &TT, const std::string &CPU,
  68:                                    const std::string &TuneCPU,
  69:                                    const std::string &FS,
  70:                                    const TargetMachine &TM)
  71:     : SystemZGenSubtargetInfo(TT, CPU, TuneCPU, FS), TargetTriple(TT),
  72:       SpecialRegisters(initializeSpecialRegisters()),
```
- **EN**: The range implements or declares functions including `SystemZSubtarget::initializeSpecialRegisters`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `SystemZSubtarget::initializeSpecialRegisters` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 73-90 / 第 73-90 行
```cpp
  73:       InstrInfo(initializeSubtargetDependencies(CPU, TuneCPU, FS)),
  74:       TLInfo(TM, *this), FrameLowering(SystemZFrameLowering::create(*this)) {}
  75: 
  76: bool SystemZSubtarget::enableSubRegLiveness() const {
  77:   return UseSubRegLiveness;
  78: }
  79: 
  80: bool SystemZSubtarget::isAddressedViaADA(const GlobalValue *GV) const {
  81:   if (const auto *GO = dyn_cast<GlobalObject>(GV)) {
  82:     // A R/O variable is placed in code section. If the R/O variable has as
  83:     // least two byte alignment, then generated code can use relative
  84:     // instructions to address the variable. Otherwise, use the ADA to address
  85:     // the variable.
  86:     if (auto *GV = dyn_cast<GlobalVariable>(GO))
  87:       if (GV->getAlign() && (*GV->getAlign()).value() & 0x1)
  88:         return true;
  89: 
  90:     // getKindForGlobal only works with definitions
```
- **EN**: The range implements or declares functions including `InstrInfo`, `SystemZSubtarget::isAddressedViaADA`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `InstrInfo`, `SystemZSubtarget::isAddressedViaADA` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 91-108 / 第 91-108 行
```cpp
  91:     if (GO->isDeclaration()) {
  92:       return true;
  93:     }
  94: 
  95:     // check AvailableExternallyLinkage here as getKindForGlobal() asserts
  96:     if (GO->hasAvailableExternallyLinkage()) {
  97:       return true;
  98:     }
  99: 
 100:     SectionKind GOKind = TargetLoweringObjectFile::getKindForGlobal(
 101:         GO, TLInfo.getTargetMachine());
 102:     if (!GOKind.isReadOnly()) {
 103:       return true;
 104:     }
 105: 
 106:     return false; // R/O variable with multiple of 2 byte alignment
 107:   }
 108:   return true;
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 109-126 / 第 109-126 行
```cpp
 109: }
 110: 
 111: bool SystemZSubtarget::isPC32DBLSymbol(const GlobalValue *GV,
 112:                                        CodeModel::Model CM) const {
 113:   if (isTargetzOS())
 114:     return !isAddressedViaADA(GV);
 115: 
 116:   // PC32DBL accesses require the low bit to be clear.
 117:   //
 118:   // FIXME: Explicitly check for functions: the datalayout is currently
 119:   // missing information about function pointers.
 120:   const DataLayout &DL = GV->getDataLayout();
 121:   if (GV->getPointerAlignment(DL) == 1 && !GV->getValueType()->isFunctionTy())
 122:     return false;
 123: 
 124:   // For the small model, all locally-binding symbols are in range.
 125:   if (CM == CodeModel::Small)
 126:     return TLInfo.getTargetMachine().shouldAssumeDSOLocal(GV);
```
- **EN**: The range implements or declares functions including `SystemZSubtarget::isPC32DBLSymbol`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZSubtarget::isPC32DBLSymbol` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 127-132 / 第 127-132 行
```cpp
 127: 
 128:   // For Medium and above, assume that the symbol is not within the 4GB range.
 129:   // Taking the address of locally-defined text would be OK, but that
 130:   // case isn't easy to detect.
 131:   return false;
 132: }
```
- **EN**: This span continues the file's main responsibility: this file models CPU/subtarget features and scheduling properties for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

## Key Concepts / 关键概念
- **Calling convention**: Defines how arguments, returns, and preserved registers are assigned. / 定义参数、返回值和保留寄存器的分配方式。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Instruction metadata**: Captures opcodes, operands, patterns, and helper routines. / 描述操作码、操作数、匹配模式和辅助例程。
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。
- **Frame lowering**: Builds stack frames, callee-save handling, and prologue/epilogue sequences. / 构建栈帧、被调用者保存寄存器处理以及序言/尾声序列。
- **Target machine**: Owns data layout, pass configuration, and backend-wide policy. / 管理数据布局、Pass 配置以及整个后端策略。

## Dependencies / 依赖关系
- `SystemZSubtarget.h`
- `llvm/CodeGen/TargetLoweringObjectFileImpl.h`
- `llvm/IR/GlobalVariable.h`
- `llvm/Target/TargetMachine.h`
- `SystemZGenSubtargetInfo.inc`
