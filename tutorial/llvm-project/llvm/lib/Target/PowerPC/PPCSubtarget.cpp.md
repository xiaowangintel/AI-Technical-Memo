# PPCSubtarget.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/PowerPC/PPCSubtarget.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file provides subtarget feature modeling for the PowerPC backend.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/PowerPC/PPCSubtarget.cpp`，主要负责 PowerPC 后端的子目标特性建模。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//===-- PowerPCSubtarget.cpp - PPC Subtarget Information ------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file. Subtarget feature gating influences the behavior here.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。 子目标特性裁剪会影响这里的行为。

### Lines 7-10

```cpp
//===----------------------------------------------------------------------===//
//
// This file implements the PPC specific subclass of TargetSubtargetInfo.
//
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "This file implements the PPC specific subclass of TargetSubtargetInfo.". Subtarget feature gating influences the behavior here.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“This file implements the PPC specific subclass of TargetSubtargetInfo.”。 子目标特性裁剪会影响这里的行为。

### Lines 11-17

```cpp
//===----------------------------------------------------------------------===//

#include "PPCSubtarget.h"
#include "GISel/PPCCallLowering.h"
#include "GISel/PPCLegalizerInfo.h"
#include "GISel/PPCRegisterBankInfo.h"
#include "PPC.h"
```
- **EN**: Pulls in direct dependencies required by this subtarget feature modeling, so later declarations can reuse LLVM infrastructure and target-specific helpers. Subtarget feature gating influences the behavior here.
- **CN**: 这一段引入该子目标特性建模所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 子目标特性裁剪会影响这里的行为。

### Lines 18-24

```cpp
#include "PPCRegisterInfo.h"
#include "PPCSelectionDAGInfo.h"
#include "PPCTargetMachine.h"
#include "llvm/CodeGen/GlobalISel/InstructionSelect.h"
#include "llvm/CodeGen/GlobalISel/InstructionSelector.h"
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/MachineScheduler.h"
```
- **EN**: Pulls in direct dependencies required by this subtarget feature modeling, so later declarations can reuse LLVM infrastructure and target-specific helpers. This range participates in the GlobalISel pipeline.
- **CN**: 这一段引入该子目标特性建模所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 这一段参与 GlobalISel 流水线。

### Lines 25-31

```cpp
#include "llvm/IR/GlobalAlias.h"
#include "llvm/IR/GlobalValue.h"
#include "llvm/IR/GlobalVariable.h"
#include "llvm/MC/TargetRegistry.h"
#include "llvm/Support/CommandLine.h"
#include "llvm/Target/TargetMachine.h"
#include "llvm/TargetParser/PPCTargetParser.h"
```
- **EN**: Pulls in direct dependencies required by this subtarget feature modeling, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该子目标特性建模所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 32-38

```cpp
#include <cstdlib>

using namespace llvm;

#define DEBUG_TYPE "ppc-subtarget"

#define GET_SUBTARGETINFO_TARGET_DESC
```
- **EN**: Pulls in direct dependencies required by this subtarget feature modeling, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该子目标特性建模所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 39-76

```cpp
#define GET_SUBTARGETINFO_CTOR
#include "PPCGenSubtargetInfo.inc"

static cl::opt<bool>
    EnableMachinePipeliner("ppc-enable-pipeliner",
                           cl::desc("Enable Machine Pipeliner for PPC"),
                           cl::init(false), cl::Hidden);

PPCSubtarget &PPCSubtarget::initializeSubtargetDependencies(StringRef CPU,
                                                            StringRef TuneCPU,
                                                            StringRef FS) {
  initializeEnvironment();
  initSubtargetFeatures(CPU, TuneCPU, FS);
  return *this;
}

PPCSubtarget::PPCSubtarget(const Triple &TT, StringRef CPU, StringRef TuneCPU,
                           StringRef FS, const PPCTargetMachine &TM)
    : PPCGenSubtargetInfo(TT, CPU, TuneCPU, FS), TM(TM),
      FrameLowering(initializeSubtargetDependencies(CPU, TuneCPU, FS)),
      InstrInfo(*this), TLInfo(TM, *this) {
  TSInfo = std::make_unique<PPCSelectionDAGInfo>();

  CallLoweringInfo.reset(new PPCCallLowering(*getTargetLowering()));
  Legalizer.reset(new PPCLegalizerInfo(*this));
  auto *RBI = new PPCRegisterBankInfo(*getRegisterInfo());
  RegBankInfo.reset(RBI);

  InstSelector.reset(createPPCInstructionSelector(TM, *this, *RBI));
}

PPCSubtarget::~PPCSubtarget() = default;

const SelectionDAGTargetInfo *PPCSubtarget::getSelectionDAGInfo() const {
  return TSInfo.get();
}

void PPCSubtarget::initializeEnvironment() {
```
- **EN**: Pulls in direct dependencies required by this subtarget feature modeling, so later declarations can reuse LLVM infrastructure and target-specific helpers. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这一段引入该子目标特性建模所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 77-114

```cpp
  StackAlignment = Align(16);
  CPUDirective = PPC::DIR_NONE;
  HasPOPCNTD = POPCNTD_Unavailable;
}

void PPCSubtarget::initSubtargetFeatures(StringRef CPU, StringRef TuneCPU,
                                         StringRef FS) {
  // Determine default and user specified characteristics
  std::string CPUName = std::string(CPU);
  if (CPUName.empty() || CPU == "generic") {
    if (getTargetTriple().getSubArch() == Triple::PPCSubArch_spe)
      CPUName = "e500";
    else
      CPUName = std::string(PPC::getNormalizedPPCTargetCPU(getTargetTriple()));
  }

  // Determine the CPU to schedule for.
  if (TuneCPU.empty()) TuneCPU = CPUName;

  // Initialize scheduling itinerary for the specified CPU.
  InstrItins = getInstrItineraryForCPU(CPUName);

  // Parse features string.
  ParseSubtargetFeatures(CPUName, TuneCPU, FS);

  // If the user requested use of 64-bit regs, but the cpu selected doesn't
  // support it, ignore.
  if (IsPPC64 && has64BitSupport())
    Use64BitRegs = true;

  if (getTargetTriple().isPPC32SecurePlt())
    IsSecurePlt = true;

  if (HasSPE && IsPPC64)
    report_fatal_error( "SPE is only supported for 32-bit targets.\n", false);
  if (HasSPE && (HasAltivec || HasVSX || HasFPU))
    report_fatal_error(
        "SPE and traditional floating point cannot both be enabled.\n", false);
```
- **EN**: Implements helper routine(s) `Align`, `initSubtargetFeatures`, `string` for this portion of the PowerPC backend subtarget feature modeling. Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里实现了 PowerPC 后端该部分子目标特性建模所需的辅助例程 `Align`, `initSubtargetFeatures`, `string`。 该区间与栈帧布局或栈访问相关。

### Lines 115-152

```cpp

  // If not SPE, set standard FPU
  if (!HasSPE)
    HasFPU = true;

  StackAlignment = getPlatformStackAlignment();

  // Determine endianness.
  IsLittleEndian = TM.isLittleEndian();

  if (HasAIXSmallLocalExecTLS || HasAIXSmallLocalDynamicTLS) {
    if (!getTargetTriple().isOSAIX() || !IsPPC64)
      report_fatal_error("The aix-small-local-[exec|dynamic]-tls attribute is "
                         "only supported on AIX in "
                         "64-bit mode.\n",
                         false);
    // The aix-small-local-[exec|dynamic]-tls attribute should only be used with
    // -data-sections, as having data sections turned off with this option
    // is not ideal for performance. Moreover, the
    // small-local-[exec|dynamic]-tls region is a limited resource, and should
    // not be used for variables that may be replaced.
    if (!TM.getDataSections())
      report_fatal_error("The aix-small-local-[exec|dynamic]-tls attribute can "
                         "only be specified with "
                         "-data-sections.\n",
                         false);
  }

  if (HasAIXShLibTLSModelOpt && (!getTargetTriple().isOSAIX() || !IsPPC64))
    report_fatal_error("The aix-shared-lib-tls-model-opt attribute "
                       "is only supported on AIX in 64-bit mode.\n",
                       false);

  if (UsePointerGlueHelper && !getTargetTriple().isOSAIX())
    report_fatal_error("use-ptrgl-helper feature is only supported on AIX\n",
                       false);
}
```
- **EN**: Implements helper routine(s) `getPlatformStackAlignment`, `isLittleEndian`, `getTargetTriple` for this portion of the PowerPC backend subtarget feature modeling. Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里实现了 PowerPC 后端该部分子目标特性建模所需的辅助例程 `getPlatformStackAlignment`, `isLittleEndian`, `getTargetTriple`。 该区间与栈帧布局或栈访问相关。

### Lines 153-190

```cpp
bool PPCSubtarget::enableMachineScheduler() const { return true; }

bool PPCSubtarget::enableMachinePipeliner() const {
  return getSchedModel().hasInstrSchedModel() && EnableMachinePipeliner;
}

bool PPCSubtarget::useDFAforSMS() const { return false; }

// This overrides the PostRAScheduler bit in the SchedModel for each CPU.
bool PPCSubtarget::enablePostRAScheduler() const { return true; }

PPCGenSubtargetInfo::AntiDepBreakMode PPCSubtarget::getAntiDepBreakMode() const {
  return TargetSubtargetInfo::ANTIDEP_ALL;
}

void PPCSubtarget::getCriticalPathRCs(RegClassVector &CriticalPathRCs) const {
  CriticalPathRCs.clear();
  CriticalPathRCs.push_back(isPPC64() ?
                            &PPC::G8RCRegClass : &PPC::GPRCRegClass);
}

void PPCSubtarget::overrideSchedPolicy(MachineSchedPolicy &Policy,
                                       const SchedRegion &Region) const {
  // The GenericScheduler that we use defaults to scheduling bottom up only.
  // We want to schedule from both the top and the bottom and so we set
  // OnlyBottomUp to false.
  // We want to do bi-directional scheduling since it provides a more balanced
  // schedule leading to better performance.
  Policy.OnlyBottomUp = false;
  // Spilling is generally expensive on all PPC cores, so always enable
  // register-pressure tracking.
  Policy.ShouldTrackPressure = true;
}

bool PPCSubtarget::useAA() const {
  return true;
}
```
- **EN**: Implements helper routine(s) `enableMachineScheduler`, `enableMachinePipeliner`, `getSchedModel` for this portion of the PowerPC backend subtarget feature modeling. Subtarget feature gating influences the behavior here.
- **CN**: 这里实现了 PowerPC 后端该部分子目标特性建模所需的辅助例程 `enableMachineScheduler`, `enableMachinePipeliner`, `getSchedModel`。 子目标特性裁剪会影响这里的行为。

### Lines 191-228

```cpp
bool PPCSubtarget::enableSubRegLiveness() const { return true; }

bool PPCSubtarget::isGVIndirectSymbol(const GlobalValue *GV) const {
  if (isAIXABI()) {
    if (const GlobalVariable *GVar = dyn_cast<GlobalVariable>(GV))
      // On AIX the only symbols that aren't indirect are toc-data.
      return !GVar->hasAttribute("toc-data");

    return true;
  }

  // Large code model always uses the TOC even for local symbols.
  if (TM.getCodeModel() == CodeModel::Large)
    return true;

  if (TM.shouldAssumeDSOLocal(GV))
    return false;
  return true;
}

CodeModel::Model PPCSubtarget::getCodeModel(const TargetMachine &TM,
                                            const GlobalValue *GV) const {
  // If there isn't an attribute to override the module code model
  // this will be the effective code model.
  CodeModel::Model ModuleModel = TM.getCodeModel();

  // Initially support per global code model for AIX only.
  if (!isAIXABI())
    return ModuleModel;

  // Only GlobalVariables carry an attribute which can override the module code
  // model.
  assert(GV && "Unexpected NULL GlobalValue");
  const GlobalVariable *GlobalVar =
      [](const GlobalValue *GV) -> const GlobalVariable * {
    const GlobalVariable *Var = dyn_cast<GlobalVariable>(GV);
    if (Var)
      return Var;
```
- **EN**: Implements helper routine(s) `enableSubRegLiveness`, `isGVIndirectSymbol`, `isAIXABI` for this portion of the PowerPC backend subtarget feature modeling. Subtarget feature gating influences the behavior here.
- **CN**: 这里实现了 PowerPC 后端该部分子目标特性建模所需的辅助例程 `enableSubRegLiveness`, `isGVIndirectSymbol`, `isAIXABI`。 子目标特性裁剪会影响这里的行为。

### Lines 229-266

```cpp

    const GlobalAlias *Alias = dyn_cast<GlobalAlias>(GV);
    if (Alias)
      return dyn_cast<GlobalVariable>(Alias->getAliaseeObject());

    return nullptr;
  }(GV);

  if (!GlobalVar)
    return ModuleModel;

  std::optional<CodeModel::Model> MaybeCodeModel = GlobalVar->getCodeModel();
  if (MaybeCodeModel) {
    CodeModel::Model CM = *MaybeCodeModel;
    assert((CM == CodeModel::Small || CM == CodeModel::Large) &&
           "invalid code model for AIX");
    return CM;
  }

  return ModuleModel;
}

bool PPCSubtarget::isELFv2ABI() const { return TM.isELFv2ABI(); }

bool PPCSubtarget::isUsingPCRelativeCalls() const {
  return isPPC64() && hasPCRelativeMemops() && isELFv2ABI() &&
         CodeModel::Medium == getTargetMachine().getCodeModel();
}

// GlobalISEL
const CallLowering *PPCSubtarget::getCallLowering() const {
  return CallLoweringInfo.get();
}

const RegisterBankInfo *PPCSubtarget::getRegBankInfo() const {
  return RegBankInfo.get();
}
```
- **EN**: Implements helper routine(s) `getAliaseeObject`, `getCodeModel`, `isELFv2ABI` for this portion of the PowerPC backend subtarget feature modeling. Subtarget feature gating influences the behavior here.
- **CN**: 这里实现了 PowerPC 后端该部分子目标特性建模所需的辅助例程 `getAliaseeObject`, `getCodeModel`, `isELFv2ABI`。 子目标特性裁剪会影响这里的行为。

### Lines 267-273

```cpp
const LegalizerInfo *PPCSubtarget::getLegalizerInfo() const {
  return Legalizer.get();
}

InstructionSelector *PPCSubtarget::getInstructionSelector() const {
  return InstSelector.get();
}
```
- **EN**: Implements helper routine(s) `getLegalizerInfo`, `get`, `getInstructionSelector` for this portion of the PowerPC backend subtarget feature modeling. Subtarget feature gating influences the behavior here.
- **CN**: 这里实现了 PowerPC 后端该部分子目标特性建模所需的辅助例程 `getLegalizerInfo`, `get`, `getInstructionSelector`。 子目标特性裁剪会影响这里的行为。

## Key Concepts / 关键概念

- PowerPC backend integration / PowerPC 后端集成
- Subtarget feature modeling / 子目标特性建模
- GlobalISel pipeline / GlobalISel 流水线
- SelectionDAG lowering / SelectionDAG lowering
- Calling convention handling / 调用约定处理
- Register modeling / 寄存器建模
- Stack frame management / 栈帧管理
- Subtarget features / 子目标特性

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `PPCSubtarget.h`
- `GISel/PPCCallLowering.h`
- `GISel/PPCLegalizerInfo.h`
- `GISel/PPCRegisterBankInfo.h`
- `PPC.h`
- `PPCRegisterInfo.h`
- `PPCSelectionDAGInfo.h`
- `PPCTargetMachine.h`
- `llvm/CodeGen/GlobalISel/InstructionSelect.h`
- `llvm/CodeGen/GlobalISel/InstructionSelector.h`
- `llvm/CodeGen/MachineFunction.h`
- `llvm/CodeGen/MachineScheduler.h`
- `llvm/IR/GlobalAlias.h`
- `llvm/IR/GlobalValue.h`
- `llvm/IR/GlobalVariable.h`
- `llvm/MC/TargetRegistry.h`
- `llvm/Support/CommandLine.h`
- `llvm/Target/TargetMachine.h`
- `llvm/TargetParser/PPCTargetParser.h`
- `cstdlib`
- `PPCGenSubtargetInfo.inc`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- LLVM Support utilities / LLVM Support 工具层
- PowerPC target-specific helpers / PowerPC 目标专用辅助组件
