# AMDGPUSubtarget.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/AMDGPUSubtarget.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements AMDGPUSubtarget for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 AMDGPUSubtarget 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-34: File banner, includes, and setup
```cpp
//===-- AMDGPUSubtarget.cpp - AMDGPU Subtarget Information ----------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file
/// Implements the AMDGPU specific subclass of TargetSubtarget.
//
//===----------------------------------------------------------------------===//

#include "AMDGPUSubtarget.h"
#include "AMDGPUCallLowering.h"
#include "AMDGPUInstructionSelector.h"
#include "AMDGPULegalizerInfo.h"
#include "AMDGPURegisterBankInfo.h"
#include "R600Subtarget.h"
#include "SIMachineFunctionInfo.h"
#include "Utils/AMDGPUBaseInfo.h"
#include "llvm/CodeGen/GlobalISel/InlineAsmLowering.h"
#include "llvm/CodeGen/MachineScheduler.h"
#include "llvm/CodeGen/TargetFrameLowering.h"
#include "llvm/IR/DiagnosticInfo.h"
#include "llvm/IR/IntrinsicsAMDGPU.h"
#include "llvm/IR/IntrinsicsR600.h"
#include "llvm/IR/MDBuilder.h"
#include <algorithm>

using namespace llvm;

#define DEBUG_TYPE "amdgpu-subtarget"

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。

### Lines 35-66: Implements AMDGPUSubtarget::getMaxLocalMemSizeWithWaveCount
```cpp
// Returns the maximum per-workgroup LDS allocation size (in bytes) that still
// allows the given function to achieve an occupancy of NWaves waves per
// SIMD / EU, taking into account only the function's *maximum* workgroup size.
unsigned
AMDGPUSubtarget::getMaxLocalMemSizeWithWaveCount(unsigned NWaves,
                                                 const Function &F) const {
  const unsigned WaveSize = getWavefrontSize();
  const unsigned WorkGroupSize = getFlatWorkGroupSizes(F).second;
  const unsigned WavesPerWorkgroup =
      std::max(1u, (WorkGroupSize + WaveSize - 1) / WaveSize);

  const unsigned WorkGroupsPerCU =
      std::max(1u, (NWaves * getEUsPerCU()) / WavesPerWorkgroup);

  return getLocalMemorySize() / WorkGroupsPerCU;
}

std::pair<unsigned, unsigned> AMDGPUSubtarget::getOccupancyWithWorkGroupSizes(
    uint32_t LDSBytes, std::pair<unsigned, unsigned> FlatWorkGroupSizes) const {

  // FIXME: We should take into account the LDS allocation granularity.
  const unsigned MaxWGsLDS = getLocalMemorySize() / std::max(LDSBytes, 1u);

  // Queried LDS size may be larger than available on a CU, in which case we
  // consider the only achievable occupancy to be 1, in line with what we
  // consider the occupancy to be when the number of requested registers in a
  // particular bank is higher than the number of available ones in that bank.
  if (!MaxWGsLDS)
    return {1, 1};

  const unsigned WaveSize = getWavefrontSize(), WavesPerEU = getMaxWavesPerEU();

```
**EN:** This section contains concrete logic for AMDGPUSubtarget::getMaxLocalMemSizeWithWaveCount. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUSubtarget::getMaxLocalMemSizeWithWaveCount`, `std::max`, `AMDGPUSubtarget::getOccupancyWithWorkGroupSizes`.
**CN:** 本节包含与 AMDGPUSubtarget::getMaxLocalMemSizeWithWaveCount 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUSubtarget::getMaxLocalMemSizeWithWaveCount`, `std::max`, `AMDGPUSubtarget::getOccupancyWithWorkGroupSizes`。

### Lines 67-89: Conditional logic and checks
```cpp
  auto PropsFromWGSize = [=](unsigned WGSize)
      -> std::tuple<const unsigned, const unsigned, unsigned> {
    unsigned WavesPerWG = divideCeil(WGSize, WaveSize);
    unsigned WGsPerCU = std::min(getMaxWorkGroupsPerCU(WGSize), MaxWGsLDS);
    return {WavesPerWG, WGsPerCU, WavesPerWG * WGsPerCU};
  };

  // The maximum group size will generally yield the minimum number of
  // workgroups, maximum number of waves, and minimum occupancy. The opposite is
  // generally true for the minimum group size. LDS or barrier ressource
  // limitations can flip those minimums/maximums.
  const auto [MinWGSize, MaxWGSize] = FlatWorkGroupSizes;
  auto [MinWavesPerWG, MaxWGsPerCU, MaxWavesPerCU] = PropsFromWGSize(MinWGSize);
  auto [MaxWavesPerWG, MinWGsPerCU, MinWavesPerCU] = PropsFromWGSize(MaxWGSize);

  // It is possible that we end up with flipped minimum and maximum number of
  // waves per CU when the number of minimum/maximum concurrent groups on the CU
  // is limited by LDS usage or barrier resources.
  if (MinWavesPerCU >= MaxWavesPerCU) {
    std::swap(MinWavesPerCU, MaxWavesPerCU);
  } else {
    const unsigned WaveSlotsPerCU = WavesPerEU * getEUsPerCU();

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `std::min`, `std::swap`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`std::min`, `std::swap`。

### Lines 90-108: Conditional logic and checks
```cpp
    // Look for a potential smaller group size than the maximum which decreases
    // the concurrent number of waves on the CU for the same number of
    // concurrent workgroups on the CU.
    unsigned MinWavesPerCUForWGSize =
        divideCeil(WaveSlotsPerCU, MinWGsPerCU + 1) * MinWGsPerCU;
    if (MinWavesPerCU > MinWavesPerCUForWGSize) {
      unsigned ExcessSlots = MinWavesPerCU - MinWavesPerCUForWGSize;
      if (unsigned ExcessSlotsPerWG = ExcessSlots / MinWGsPerCU) {
        // There may exist a smaller group size than the maximum that achieves
        // the minimum number of waves per CU. This group size is the largest
        // possible size that requires MaxWavesPerWG - E waves where E is
        // maximized under the following constraints.
        // 1. 0 <= E <= ExcessSlotsPerWG
        // 2. (MaxWavesPerWG - E) * WaveSize >= MinWGSize
        MinWavesPerCU -= MinWGsPerCU * std::min(ExcessSlotsPerWG,
                                                MaxWavesPerWG - MinWavesPerWG);
      }
    }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `std::min`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`std::min`。

### Lines 109-137: Conditional logic and checks
```cpp
    // Look for a potential larger group size than the minimum which increases
    // the concurrent number of waves on the CU for the same number of
    // concurrent workgroups on the CU.
    unsigned LeftoverSlots = WaveSlotsPerCU - MaxWGsPerCU * MinWavesPerWG;
    if (unsigned LeftoverSlotsPerWG = LeftoverSlots / MaxWGsPerCU) {
      // There may exist a larger group size than the minimum that achieves the
      // maximum number of waves per CU. This group size is the smallest
      // possible size that requires MinWavesPerWG + L waves where L is
      // maximized under the following constraints.
      // 1. 0 <= L <= LeftoverSlotsPerWG
      // 2. (MinWavesPerWG + L - 1) * WaveSize <= MaxWGSize
      MaxWavesPerCU += MaxWGsPerCU * std::min(LeftoverSlotsPerWG,
                                              ((MaxWGSize - 1) / WaveSize) + 1 -
                                                  MinWavesPerWG);
    }
  }

  // Return the minimum/maximum number of waves on any EU, assuming that all
  // wavefronts are spread across all EUs as evenly as possible.
  return {std::clamp(MinWavesPerCU / getEUsPerCU(), 1U, WavesPerEU),
          std::clamp(divideCeil(MaxWavesPerCU, getEUsPerCU()), 1U, WavesPerEU)};
}

std::pair<unsigned, unsigned> AMDGPUSubtarget::getOccupancyWithWorkGroupSizes(
    const MachineFunction &MF) const {
  const auto *MFI = MF.getInfo<SIMachineFunctionInfo>();
  return getOccupancyWithWorkGroupSizes(MFI->getLDSSize(), MF.getFunction());
}

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `std::min`, `std::clamp`, `AMDGPUSubtarget::getOccupancyWithWorkGroupSizes`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`std::min`, `std::clamp`, `AMDGPUSubtarget::getOccupancyWithWorkGroupSizes`。

### Lines 138-166: Implements AMDGPUSubtarget::getDefaultFlatWorkGroupSize
```cpp
std::pair<unsigned, unsigned>
AMDGPUSubtarget::getDefaultFlatWorkGroupSize(CallingConv::ID CC) const {
  switch (CC) {
  case CallingConv::AMDGPU_VS:
  case CallingConv::AMDGPU_LS:
  case CallingConv::AMDGPU_HS:
  case CallingConv::AMDGPU_ES:
  case CallingConv::AMDGPU_GS:
  case CallingConv::AMDGPU_PS:
    return std::pair(1, getWavefrontSize());
  default:
    return std::pair(1u, getMaxFlatWorkGroupSize());
  }
}

std::pair<unsigned, unsigned> AMDGPUSubtarget::getFlatWorkGroupSizes(
  const Function &F) const {
  // Default minimum/maximum flat work group sizes.
  std::pair<unsigned, unsigned> Default =
    getDefaultFlatWorkGroupSize(F.getCallingConv());

  // Requested minimum/maximum flat work group sizes.
  std::pair<unsigned, unsigned> Requested = AMDGPU::getIntegerPairAttribute(
    F, "amdgpu-flat-work-group-size", Default);

  // Make sure requested minimum is less than requested maximum.
  if (Requested.first > Requested.second)
    return Default;

```
**EN:** This section contains concrete logic for AMDGPUSubtarget::getDefaultFlatWorkGroupSize. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUSubtarget::getDefaultFlatWorkGroupSize`, `std::pair`, `AMDGPUSubtarget::getFlatWorkGroupSizes`.
**CN:** 本节包含与 AMDGPUSubtarget::getDefaultFlatWorkGroupSize 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUSubtarget::getDefaultFlatWorkGroupSize`, `std::pair`, `AMDGPUSubtarget::getFlatWorkGroupSizes`。

### Lines 167-199: Implements AMDGPUSubtarget::isSingleWavefrontWorkgroup
```cpp
  // Make sure requested values do not violate subtarget's specifications.
  if (Requested.first < getMinFlatWorkGroupSize())
    return Default;
  if (Requested.second > getMaxFlatWorkGroupSize())
    return Default;

  return Requested;
}

bool AMDGPUSubtarget::isSingleWavefrontWorkgroup(const Function &F) const {
  return getFlatWorkGroupSizes(F).second <= getWavefrontSize();
}

std::pair<unsigned, unsigned> AMDGPUSubtarget::getEffectiveWavesPerEU(
    std::pair<unsigned, unsigned> RequestedWavesPerEU,
    std::pair<unsigned, unsigned> FlatWorkGroupSizes, unsigned LDSBytes) const {
  // Default minimum/maximum number of waves per EU. The range of flat workgroup
  // sizes limits the achievable maximum, and we aim to support enough waves per
  // EU so that we can concurrently execute all waves of a single workgroup of
  // maximum size on a CU.
  std::pair<unsigned, unsigned> Default = {
      getWavesPerEUForWorkGroup(FlatWorkGroupSizes.second),
      getOccupancyWithWorkGroupSizes(LDSBytes, FlatWorkGroupSizes).second};
  Default.first = std::min(Default.first, Default.second);

  // Make sure requested minimum is within the default range and lower than the
  // requested maximum. The latter must not violate target specification.
  if (RequestedWavesPerEU.first < Default.first ||
      RequestedWavesPerEU.first > Default.second ||
      RequestedWavesPerEU.first > RequestedWavesPerEU.second ||
      RequestedWavesPerEU.second > getMaxWavesPerEU())
    return Default;

```
**EN:** This section contains concrete logic for AMDGPUSubtarget::isSingleWavefrontWorkgroup. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUSubtarget::isSingleWavefrontWorkgroup`, `AMDGPUSubtarget::getEffectiveWavesPerEU`, `std::min`.
**CN:** 本节包含与 AMDGPUSubtarget::isSingleWavefrontWorkgroup 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUSubtarget::isSingleWavefrontWorkgroup`, `AMDGPUSubtarget::getEffectiveWavesPerEU`, `std::min`。

### Lines 200-229: Implements AMDGPUSubtarget::getWavesPerEU
```cpp
  // We cannot exceed maximum occupancy implied by flat workgroup size and LDS.
  RequestedWavesPerEU.second =
      std::min(RequestedWavesPerEU.second, Default.second);
  return RequestedWavesPerEU;
}

std::pair<unsigned, unsigned>
AMDGPUSubtarget::getWavesPerEU(const Function &F) const {
  // Default/requested minimum/maximum flat work group sizes.
  std::pair<unsigned, unsigned> FlatWorkGroupSizes = getFlatWorkGroupSizes(F);
  // Minimum number of bytes allocated in the LDS.
  unsigned LDSBytes =
      AMDGPU::getIntegerPairAttribute(F, "amdgpu-lds-size", {0, UINT32_MAX},
                                      /*OnlyFirstRequired=*/true)
          .first;
  return getWavesPerEU(FlatWorkGroupSizes, LDSBytes, F);
}

std::pair<unsigned, unsigned>
AMDGPUSubtarget::getWavesPerEU(std::pair<unsigned, unsigned> FlatWorkGroupSizes,
                               unsigned LDSBytes, const Function &F) const {
  // Default minimum/maximum number of waves per execution unit.
  std::pair<unsigned, unsigned> Default(1, getMaxWavesPerEU());

  // Requested minimum/maximum number of waves per execution unit.
  std::pair<unsigned, unsigned> Requested =
      AMDGPU::getIntegerPairAttribute(F, "amdgpu-waves-per-eu", Default, true);
  return getEffectiveWavesPerEU(Requested, FlatWorkGroupSizes, LDSBytes);
}

```
**EN:** This section contains concrete logic for AMDGPUSubtarget::getWavesPerEU. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `std::min`, `AMDGPUSubtarget::getWavesPerEU`, `AMDGPU::getIntegerPairAttribute`.
**CN:** 本节包含与 AMDGPUSubtarget::getWavesPerEU 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`std::min`, `AMDGPUSubtarget::getWavesPerEU`, `AMDGPU::getIntegerPairAttribute`。

### Lines 230-260: Implements AMDGPUSubtarget::getReqdWorkGroupSize
```cpp
std::optional<unsigned>
AMDGPUSubtarget::getReqdWorkGroupSize(const Function &Kernel,
                                      unsigned Dim) const {
  auto *Node = Kernel.getMetadata("reqd_work_group_size");
  if (Node && Node->getNumOperands() == 3)
    return mdconst::extract<ConstantInt>(Node->getOperand(Dim))->getZExtValue();
  return std::nullopt;
}

bool AMDGPUSubtarget::hasWavefrontsEvenlySplittingXDim(
    const Function &F, bool RequiresUniformYZ) const {
  auto *Node = F.getMetadata("reqd_work_group_size");
  if (!Node || Node->getNumOperands() != 3)
    return false;
  unsigned XLen =
      mdconst::extract<ConstantInt>(Node->getOperand(0))->getZExtValue();
  unsigned YLen =
      mdconst::extract<ConstantInt>(Node->getOperand(1))->getZExtValue();
  unsigned ZLen =
      mdconst::extract<ConstantInt>(Node->getOperand(2))->getZExtValue();

  bool Is1D = YLen <= 1 && ZLen <= 1;
  bool IsXLargeEnough =
      isPowerOf2_32(XLen) && (!RequiresUniformYZ || XLen >= getWavefrontSize());
  return Is1D || IsXLargeEnough;
}

bool AMDGPUSubtarget::isMesaKernel(const Function &F) const {
  return isMesa3DOS() && !AMDGPU::isShader(F.getCallingConv());
}

```
**EN:** This section contains concrete logic for AMDGPUSubtarget::getReqdWorkGroupSize. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUSubtarget::getReqdWorkGroupSize`, `AMDGPUSubtarget::hasWavefrontsEvenlySplittingXDim`, `AMDGPUSubtarget::isMesaKernel`.
**CN:** 本节包含与 AMDGPUSubtarget::getReqdWorkGroupSize 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUSubtarget::getReqdWorkGroupSize`, `AMDGPUSubtarget::hasWavefrontsEvenlySplittingXDim`, `AMDGPUSubtarget::isMesaKernel`。

### Lines 261-288: Implements AMDGPUSubtarget::getMaxWorkitemID
```cpp
unsigned AMDGPUSubtarget::getMaxWorkitemID(const Function &Kernel,
                                           unsigned Dimension) const {
  std::optional<unsigned> ReqdSize = getReqdWorkGroupSize(Kernel, Dimension);
  if (ReqdSize)
    return *ReqdSize - 1;
  return getFlatWorkGroupSizes(Kernel).second - 1;
}

bool AMDGPUSubtarget::isSingleLaneExecution(const Function &Func) const {
  for (int I = 0; I < 3; ++I) {
    if (getMaxWorkitemID(Func, I) > 0)
      return false;
  }

  // If the function may call the WWM intrinsic, just return false as
  // all threads will be active at some point
  if (!Func.hasFnAttribute("amdgpu-no-wwm"))
    return false;

  return true;
}

bool AMDGPUSubtarget::makeLIDRangeMetadata(Instruction *I) const {
  Function *Kernel = I->getFunction();
  unsigned MinSize = 0;
  unsigned MaxSize = getFlatWorkGroupSizes(*Kernel).second;
  bool IdQuery = false;

```
**EN:** This section contains concrete logic for AMDGPUSubtarget::getMaxWorkitemID. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUSubtarget::getMaxWorkitemID`, `AMDGPUSubtarget::isSingleLaneExecution`, `AMDGPUSubtarget::makeLIDRangeMetadata`.
**CN:** 本节包含与 AMDGPUSubtarget::getMaxWorkitemID 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUSubtarget::getMaxWorkitemID`, `AMDGPUSubtarget::isSingleLaneExecution`, `AMDGPUSubtarget::makeLIDRangeMetadata`。

### Lines 289-319: Switch-based control flow
```cpp
  // If reqd_work_group_size is present it narrows value down.
  if (auto *CI = dyn_cast<CallInst>(I)) {
    const Function *F = CI->getCalledFunction();
    if (F) {
      unsigned Dim = UINT_MAX;
      switch (F->getIntrinsicID()) {
      case Intrinsic::amdgcn_workitem_id_x:
      case Intrinsic::r600_read_tidig_x:
        IdQuery = true;
        [[fallthrough]];
      case Intrinsic::r600_read_local_size_x:
        Dim = 0;
        break;
      case Intrinsic::amdgcn_workitem_id_y:
      case Intrinsic::r600_read_tidig_y:
        IdQuery = true;
        [[fallthrough]];
      case Intrinsic::r600_read_local_size_y:
        Dim = 1;
        break;
      case Intrinsic::amdgcn_workitem_id_z:
      case Intrinsic::r600_read_tidig_z:
        IdQuery = true;
        [[fallthrough]];
      case Intrinsic::r600_read_local_size_z:
        Dim = 2;
        break;
      default:
        break;
      }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 320-352: Conditional logic and checks
```cpp
      if (Dim <= 3) {
        std::optional<unsigned> ReqdSize = getReqdWorkGroupSize(*Kernel, Dim);
        if (ReqdSize)
          MinSize = MaxSize = *ReqdSize;
      }
    }
  }

  if (!MaxSize)
    return false;

  // Range metadata is [Lo, Hi). For ID query we need to pass max size
  // as Hi. For size query we need to pass Hi + 1.
  if (IdQuery)
    MinSize = 0;
  else
    ++MaxSize;

  APInt Lower{32, MinSize};
  APInt Upper{32, MaxSize};
  if (auto *CI = dyn_cast<CallBase>(I)) {
    ConstantRange Range(Lower, Upper);
    CI->addRangeRetAttr(Range);
  } else {
    MDBuilder MDB(I->getContext());
    MDNode *MaxWorkGroupSizeRange = MDB.createRange(Lower, Upper);
    I->setMetadata(LLVMContext::MD_range, MaxWorkGroupSizeRange);
  }
  return true;
}

unsigned AMDGPUSubtarget::getImplicitArgNumBytes(const Function &F) const {

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `AMDGPUSubtarget::getImplicitArgNumBytes`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`AMDGPUSubtarget::getImplicitArgNumBytes`。

### Lines 353-381: Declares getParent
```cpp
  // We don't allocate the segment if we know the implicit arguments weren't
  // used, even if the ABI implies we need them.
  if (F.hasFnAttribute("amdgpu-no-implicitarg-ptr"))
    return 0;

  if (isMesaKernel(F))
    return 16;

  // Assume all implicit inputs are used by default
  const Module *M = F.getParent();
  unsigned NBytes =
      AMDGPU::getAMDHSACodeObjectVersion(*M) >= AMDGPU::AMDHSA_COV5 ? 256 : 56;
  return F.getFnAttributeAsParsedInteger("amdgpu-implicitarg-num-bytes",
                                         NBytes);
}

uint64_t AMDGPUSubtarget::getExplicitKernArgSize(const Function &F,
                                                 Align &MaxAlign) const {
  assert(F.getCallingConv() == CallingConv::AMDGPU_KERNEL ||
         F.getCallingConv() == CallingConv::SPIR_KERNEL);

  const DataLayout &DL = F.getDataLayout();
  uint64_t ExplicitArgBytes = 0;
  MaxAlign = Align(1);

  for (const Argument &Arg : F.args()) {
    if (Arg.hasAttribute("amdgpu-hidden-argument"))
      continue;

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `AMDGPU::getAMDHSACodeObjectVersion`, `AMDGPUSubtarget::getExplicitKernArgSize`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`AMDGPU::getAMDHSACodeObjectVersion`, `AMDGPUSubtarget::getExplicitKernArgSize`。

### Lines 382-415: Conditional logic and checks
```cpp
    const bool IsByRef = Arg.hasByRefAttr();
    Type *ArgTy = IsByRef ? Arg.getParamByRefType() : Arg.getType();
    Align Alignment = DL.getValueOrABITypeAlignment(
        IsByRef ? Arg.getParamAlign() : std::nullopt, ArgTy);
    uint64_t AllocSize = DL.getTypeAllocSize(ArgTy);
    ExplicitArgBytes = alignTo(ExplicitArgBytes, Alignment) + AllocSize;
    MaxAlign = std::max(MaxAlign, Alignment);
  }

  return ExplicitArgBytes;
}

unsigned AMDGPUSubtarget::getKernArgSegmentSize(const Function &F,
                                                Align &MaxAlign) const {
  if (F.getCallingConv() != CallingConv::AMDGPU_KERNEL &&
      F.getCallingConv() != CallingConv::SPIR_KERNEL)
    return 0;

  uint64_t ExplicitArgBytes = getExplicitKernArgSize(F, MaxAlign);

  unsigned ExplicitOffset = getExplicitKernelArgOffset();

  uint64_t TotalSize = ExplicitOffset + ExplicitArgBytes;
  unsigned ImplicitBytes = getImplicitArgNumBytes(F);
  if (ImplicitBytes != 0) {
    const Align Alignment = getAlignmentForImplicitArgPtr();
    TotalSize = alignTo(ExplicitArgBytes, Alignment) + ImplicitBytes;
    MaxAlign = std::max(MaxAlign, Alignment);
  }

  // Being able to dereference past the end is useful for emitting scalar loads.
  return alignTo(TotalSize, 4);
}

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `std::max`, `AMDGPUSubtarget::getKernArgSegmentSize`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`std::max`, `AMDGPUSubtarget::getKernArgSegmentSize`。

### Lines 416-439: Implements AMDGPUSubtarget::getAMDGPUDwarfFlavour
```cpp
AMDGPUDwarfFlavour AMDGPUSubtarget::getAMDGPUDwarfFlavour() const {
  return getWavefrontSize() == 32 ? AMDGPUDwarfFlavour::Wave32
                                  : AMDGPUDwarfFlavour::Wave64;
}

const AMDGPUSubtarget &AMDGPUSubtarget::get(const MachineFunction &MF) {
  if (MF.getTarget().getTargetTriple().isAMDGCN())
    return static_cast<const AMDGPUSubtarget&>(MF.getSubtarget<GCNSubtarget>());
  return static_cast<const AMDGPUSubtarget &>(MF.getSubtarget<R600Subtarget>());
}

const AMDGPUSubtarget &AMDGPUSubtarget::get(const TargetMachine &TM, const Function &F) {
  if (TM.getTargetTriple().isAMDGCN())
    return static_cast<const AMDGPUSubtarget&>(TM.getSubtarget<GCNSubtarget>(F));
  return static_cast<const AMDGPUSubtarget &>(
      TM.getSubtarget<R600Subtarget>(F));
}

// FIXME: This has no reason to be in subtarget
SmallVector<unsigned>
AMDGPUSubtarget::getMaxNumWorkGroups(const Function &F) const {
  return AMDGPU::getIntegerVecAttribute(F, "amdgpu-max-num-workgroups", 3,
                                        std::numeric_limits<uint32_t>::max());
}
```
**EN:** This section contains concrete logic for AMDGPUSubtarget::getAMDGPUDwarfFlavour. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUSubtarget::getAMDGPUDwarfFlavour`, `AMDGPUSubtarget::get`, `AMDGPUSubtarget::getMaxNumWorkGroups`.
**CN:** 本节包含与 AMDGPUSubtarget::getAMDGPUDwarfFlavour 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUSubtarget::getAMDGPUDwarfFlavour`, `AMDGPUSubtarget::get`, `AMDGPUSubtarget::getMaxNumWorkGroups`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `AMDGPUSubtarget::getMaxLocalMemSizeWithWaveCount`, `std::max`, `AMDGPUSubtarget::getOccupancyWithWorkGroupSizes`, `std::min`, `std::swap`, `std::clamp`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; subtarget modeling / 子目标建模; lowering / 降低
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"AMDGPUSubtarget.h"`
- `"AMDGPUCallLowering.h"`
- `"AMDGPUInstructionSelector.h"`
- `"AMDGPULegalizerInfo.h"`
- `"AMDGPURegisterBankInfo.h"`
- `"R600Subtarget.h"`
- `"SIMachineFunctionInfo.h"`
- `"Utils/AMDGPUBaseInfo.h"`
- `"llvm/CodeGen/GlobalISel/InlineAsmLowering.h"`
- `"llvm/CodeGen/MachineScheduler.h"`
- `"llvm/CodeGen/TargetFrameLowering.h"`
- `"llvm/IR/DiagnosticInfo.h"`
- `"llvm/IR/IntrinsicsAMDGPU.h"`
- `"llvm/IR/IntrinsicsR600.h"`
- `"llvm/IR/MDBuilder.h"`
- `<algorithm>`
