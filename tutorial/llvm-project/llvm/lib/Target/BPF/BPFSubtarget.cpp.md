# BPFSubtarget.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/BPF/BPFSubtarget.cpp`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): This file implements the BPF specific subclass of TargetSubtargetInfo.
- 目的（中文）: 定义目标子架构的 CPU、特性位以及调优信息。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===-- BPFSubtarget.cpp - BPF Subtarget Information ----------------------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file implements the BPF specific subclass of TargetSubtargetInfo.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: #include "BPFSubtarget.h"
  14: #include "BPF.h"
  15: #include "BPFTargetMachine.h"
  16: #include "GISel/BPFCallLowering.h"
  17: #include "GISel/BPFLegalizerInfo.h"
  18: #include "GISel/BPFRegisterBankInfo.h"
  19: #include "llvm/MC/TargetRegistry.h"
  20: #include "llvm/TargetParser/Host.h"
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。

### Lines 21-40

```cpp
  21: 
  22: using namespace llvm;
  23: 
  24: #define DEBUG_TYPE "bpf-subtarget"
  25: 
  26: #define GET_SUBTARGETINFO_TARGET_DESC
  27: #define GET_SUBTARGETINFO_CTOR
  28: #include "BPFGenSubtargetInfo.inc"
  29: 
  30: static cl::opt<bool> Disable_ldsx("disable-ldsx", cl::Hidden, cl::init(false),
  31:   cl::desc("Disable ldsx insns"));
  32: static cl::opt<bool> Disable_movsx("disable-movsx", cl::Hidden, cl::init(false),
  33:   cl::desc("Disable movsx insns"));
  34: static cl::opt<bool> Disable_bswap("disable-bswap", cl::Hidden, cl::init(false),
  35:   cl::desc("Disable bswap insns"));
  36: static cl::opt<bool> Disable_sdiv_smod("disable-sdiv-smod", cl::Hidden,
  37:   cl::init(false), cl::desc("Disable sdiv/smod insns"));
  38: static cl::opt<bool> Disable_gotol("disable-gotol", cl::Hidden, cl::init(false),
  39:   cl::desc("Disable gotol insn"));
  40: static cl::opt<bool>
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。

### Lines 41-60

```cpp
  41:     Disable_StoreImm("disable-storeimm", cl::Hidden, cl::init(false),
  42:                      cl::desc("Disable BPF_ST (immediate store) insn"));
  43: static cl::opt<bool> Disable_load_acq_store_rel(
  44:     "disable-load-acq-store-rel", cl::Hidden, cl::init(false),
  45:     cl::desc("Disable load-acquire and store-release insns"));
  46: static cl::opt<bool> Disable_gotox("disable-gotox", cl::Hidden, cl::init(false),
  47:                                    cl::desc("Disable gotox insn"));
  48: 
  49: void BPFSubtarget::anchor() {}
  50: 
  51: BPFSubtarget &BPFSubtarget::initializeSubtargetDependencies(StringRef CPU,
  52:                                                             StringRef FS) {
  53:   initializeEnvironment();
  54:   initSubtargetFeatures(CPU, FS);
  55:   ParseSubtargetFeatures(CPU, /*TuneCPU*/ CPU, FS);
  56:   return *this;
  57: }
  58: 
  59: void BPFSubtarget::initializeEnvironment() {
  60:   HasJmpExt = false;
```

- EN: Function bodies or method definitions such as anchor, initializeEnvironment contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: anchor, initializeEnvironment 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 61-80

```cpp
  61:   HasJmp32 = false;
  62:   HasAlu32 = false;
  63:   UseDwarfRIS = false;
  64:   HasLdsx = false;
  65:   HasMovsx = false;
  66:   HasBswap = false;
  67:   HasSdivSmod = false;
  68:   HasGotol = false;
  69:   HasStoreImm = false;
  70:   HasLoadAcqStoreRel = false;
  71:   HasGotox = false;
  72:   AllowsMisalignedMemAccess = false;
  73: }
  74: 
  75: void BPFSubtarget::initSubtargetFeatures(StringRef CPU, StringRef FS) {
  76:   if (CPU.empty())
  77:     CPU = "v3";
  78:   if (CPU == "probe")
  79:     CPU = sys::detail::getHostCPUNameForBPF();
  80:   if (CPU == "generic" || CPU == "v1")
```

- EN: Function bodies or method definitions such as initSubtargetFeatures contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: initSubtargetFeatures 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 81-100

```cpp
  81:     return;
  82:   if (CPU == "v2") {
  83:     HasJmpExt = true;
  84:     return;
  85:   }
  86:   if (CPU == "v3") {
  87:     HasJmpExt = true;
  88:     HasJmp32 = true;
  89:     HasAlu32 = true;
  90:     return;
  91:   }
  92:   if (CPU == "v4") {
  93:     HasJmpExt = true;
  94:     HasJmp32 = true;
  95:     HasAlu32 = true;
  96:     HasLdsx = !Disable_ldsx;
  97:     HasMovsx = !Disable_movsx;
  98:     HasBswap = !Disable_bswap;
  99:     HasSdivSmod = !Disable_sdiv_smod;
 100:     HasGotol = !Disable_gotol;
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 101-120

```cpp
 101:     HasStoreImm = !Disable_StoreImm;
 102:     HasLoadAcqStoreRel = !Disable_load_acq_store_rel;
 103:     HasGotox = !Disable_gotox;
 104:     return;
 105:   }
 106: }
 107: 
 108: BPFSubtarget::BPFSubtarget(const Triple &TT, const std::string &CPU,
 109:                            const std::string &FS, const TargetMachine &TM)
 110:     : BPFGenSubtargetInfo(TT, CPU, /*TuneCPU*/ CPU, FS),
 111:       InstrInfo(initializeSubtargetDependencies(CPU, FS)), FrameLowering(*this),
 112:       TLInfo(TM, *this) {
 113:   IsLittleEndian = TT.isLittleEndian();
 114: 
 115:   CallLoweringInfo.reset(new BPFCallLowering(*getTargetLowering()));
 116:   Legalizer.reset(new BPFLegalizerInfo(*this));
 117:   auto *RBI = new BPFRegisterBankInfo(*getRegisterInfo());
 118:   RegBankInfo.reset(RBI);
 119: 
 120:   InstSelector.reset(createBPFInstructionSelector(
```

- EN: Function bodies or method definitions such as BPFGenSubtargetInfo contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: BPFGenSubtargetInfo 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 121-138

```cpp
 121:       *static_cast<const BPFTargetMachine *>(&TM), *this, *RBI));
 122: }
 123: 
 124: const CallLowering *BPFSubtarget::getCallLowering() const {
 125:   return CallLoweringInfo.get();
 126: }
 127: 
 128: InstructionSelector *BPFSubtarget::getInstructionSelector() const {
 129:   return InstSelector.get();
 130: }
 131: 
 132: const LegalizerInfo *BPFSubtarget::getLegalizerInfo() const {
 133:   return Legalizer.get();
 134: }
 135: 
 136: const RegisterBankInfo *BPFSubtarget::getRegBankInfo() const {
 137:   return RegBankInfo.get();
 138: }
```

- EN: This range continues the implementation of the backend component described by BPFSubtarget.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

## Key Concepts / 关键概念

- Feature bits / 特性位
- CPU tuning / CPU 调优
- Register banks / 寄存器银行
- IR to target lowering / IR 到目标降级
- Target machine configuration / 目标机器配置
- CPU feature modelling / CPU 特性建模
- Stack frame lowering / 栈帧降级

## Dependencies / 依赖关系

- Direct includes / 直接包含: `BPFSubtarget.h`, `BPF.h`, `BPFTargetMachine.h`, `GISel/BPFCallLowering.h`, `GISel/BPFLegalizerInfo.h`, `GISel/BPFRegisterBankInfo.h`, `llvm/MC/TargetRegistry.h`, `llvm/TargetParser/Host.h`
- LLVM subsystems / LLVM 子系统: SelectionDAG
- Generated or companion files / 生成或配套文件: `BPFGenSubtargetInfo.inc`
- Local companions / 本地配套文件: `BPFSubtarget.h`
