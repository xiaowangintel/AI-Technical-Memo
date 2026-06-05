# AArch64Subtarget.h — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/AArch64/AArch64Subtarget.h`
- **Repository**: llvm/llvm-project
- **Purpose**: This file declares the AArch64 specific subclass of TargetSubtarget. / 该文件实现 AArch64 后端中的子目标特性建模。
## Line-by-Line Analysis / 逐行分析
### Lines 1-32: Documented code section
```cpp
//===--- AArch64Subtarget.h - Define Subtarget for the AArch64 -*- C++ -*--===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file declares the AArch64 specific subclass of TargetSubtarget.
//
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_AARCH64_AARCH64SUBTARGET_H
#define LLVM_LIB_TARGET_AARCH64_AARCH64SUBTARGET_H

#include "AArch64FrameLowering.h"
#include "AArch64ISelLowering.h"
#include "AArch64InstrInfo.h"
#include "AArch64PointerAuth.h"
#include "AArch64RegisterInfo.h"
#include "AArch64SelectionDAGInfo.h"
#include "llvm/CodeGen/GlobalISel/CallLowering.h"
#include "llvm/CodeGen/GlobalISel/InlineAsmLowering.h"
#include "llvm/CodeGen/GlobalISel/InstructionSelector.h"
#include "llvm/CodeGen/GlobalISel/LegalizerInfo.h"
#include "llvm/CodeGen/RegisterBankInfo.h"
#include "llvm/CodeGen/TargetSubtargetInfo.h"
#include "llvm/IR/DataLayout.h"
#include "llvm/TargetParser/Triple.h"

#define GET_SUBTARGETINFO_HEADER
#include "AArch64GenSubtargetInfo.inc"
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
### Lines 33-57: Namespace llvm
```cpp

namespace llvm {
class GlobalValue;
class StringRef;

class AArch64Subtarget final : public AArch64GenSubtargetInfo {
public:
  enum ARMProcFamilyEnum : uint8_t {
    Generic,
#define ARM_PROCESSOR_FAMILY(ENUM) ENUM,
#include "llvm/TargetParser/AArch64TargetParserDef.inc"
#undef ARM_PROCESSOR_FAMILY
  };

protected:
  /// ARMProcFamily - ARM processor family: Cortex-A53, Cortex-A57, and others.
  ARMProcFamilyEnum ARMProcFamily = Generic;

  // Enable 64-bit vectorization in SLP.
  unsigned MinVectorRegisterBitWidth = 64;

// Bool members corresponding to the SubtargetFeatures defined in tablegen
#define GET_SUBTARGETINFO_MACRO(ATTRIBUTE, DEFAULT, GETTER)                    \
  bool ATTRIBUTE = DEFAULT;
#include "AArch64GenSubtargetInfo.inc"
```
**EN:** This block scopes the following declarations to the intended namespace boundary.  
**CN:** 该代码块将后续声明限定在目标命名空间边界内。
### Lines 58-84: Core AArch64 backend logic
```cpp

  unsigned EpilogueVectorizationMinVF = 16;
  uint8_t MaxInterleaveFactor = 2;
  uint8_t VectorInsertExtractBaseCost = 2;
  uint16_t CacheLineSize = 0;
  // Default scatter/gather overhead.
  unsigned ScatterOverhead = 10;
  unsigned GatherOverhead = 10;
  uint16_t PrefetchDistance = 0;
  uint16_t MinPrefetchStride = 1;
  unsigned MaxPrefetchIterationsAhead = UINT_MAX;
  Align PrefFunctionAlignment;
  Align PrefLoopAlignment;
  unsigned MaxBytesForLoopAlignment = 0;
  unsigned MinimumJumpTableEntries = 4;
  unsigned MaxJumpTableSize = 0;

  // ReserveXRegister[i] - X#i is not available as a general purpose register.
  BitVector ReserveXRegister;

  // ReserveXRegisterForRA[i] - X#i is not available for register allocator.
  BitVector ReserveXRegisterForRA;

  // CustomCallUsedXRegister[i] - X#i call saved.
  BitVector CustomCallSavedXRegs;

  bool IsLittle;
```
**EN:** This block continues the file's main subtarget feature modeling logic and connects local helpers with the wider AArch64 backend.  
**CN:** 该代码块延续本文件的子目标特性建模主线逻辑，并把本地辅助实现连接到更广泛的 AArch64 后端中。
### Lines 85-119: Core AArch64 backend logic
```cpp

  bool IsStreaming;
  bool IsStreamingCompatible;
  std::optional<unsigned> StreamingHazardSize;
  unsigned MinSVEVectorSizeInBits;
  unsigned MaxSVEVectorSizeInBits;
  bool EnableSRLTSubregToRegMitigation;
  unsigned VScaleForTuning = 1;
  TailFoldingOpts DefaultSVETFOpts = TailFoldingOpts::Disabled;

  bool EnableSubregLiveness;

  /// TargetTriple - What processor and OS we're targeting.
  Triple TargetTriple;

  AArch64FrameLowering FrameLowering;
  AArch64InstrInfo InstrInfo;
  AArch64SelectionDAGInfo TSInfo;
  AArch64TargetLowering TLInfo;

  /// GlobalISel related APIs.
  std::unique_ptr<CallLowering> CallLoweringInfo;
  std::unique_ptr<InlineAsmLowering> InlineAsmLoweringInfo;
  std::unique_ptr<InstructionSelector> InstSelector;
  std::unique_ptr<LegalizerInfo> Legalizer;
  std::unique_ptr<RegisterBankInfo> RegBankInfo;

private:
  /// initializeSubtargetDependencies - Initializes using CPUString and the
  /// passed in feature string so that we can use initializer lists for
  /// subtarget initialization.
  AArch64Subtarget &initializeSubtargetDependencies(StringRef FS,
                                                    StringRef CPUString,
                                                    StringRef TuneCPUString,
                                                    bool HasMinSize);
```
**EN:** This block continues the file's main subtarget feature modeling logic and connects local helpers with the wider AArch64 backend.  
**CN:** 该代码块延续本文件的子目标特性建模主线逻辑，并把本地辅助实现连接到更广泛的 AArch64 后端中。
### Lines 120-138: Documented code section
```cpp

  /// Initialize properties based on the selected processor family.
  void initializeProperties(bool HasMinSize);

public:
  /// This constructor initializes the data members to match that
  /// of the specified triple.
  AArch64Subtarget(const Triple &TT, StringRef CPU, StringRef TuneCPU,
                   StringRef FS, const TargetMachine &TM, bool LittleEndian,
                   unsigned MinSVEVectorSizeInBitsOverride = 0,
                   unsigned MaxSVEVectorSizeInBitsOverride = 0,
                   bool IsStreaming = false, bool IsStreamingCompatible = false,
                   bool HasMinSize = false,
                   bool EnableSRLTSubregToRegMitigation = false);

// Getters for SubtargetFeatures defined in tablegen
#define GET_SUBTARGETINFO_MACRO(ATTRIBUTE, DEFAULT, GETTER)                    \
  bool GETTER() const { return ATTRIBUTE; }
#include "AArch64GenSubtargetInfo.inc"
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
### Lines 139-173: Function getSelectionDAGInfo
```cpp

  const AArch64SelectionDAGInfo *getSelectionDAGInfo() const override {
    return &TSInfo;
  }
  const AArch64FrameLowering *getFrameLowering() const override {
    return &FrameLowering;
  }
  const AArch64TargetLowering *getTargetLowering() const override {
    return &TLInfo;
  }
  const AArch64InstrInfo *getInstrInfo() const override { return &InstrInfo; }
  const AArch64RegisterInfo *getRegisterInfo() const override {
    return &getInstrInfo()->getRegisterInfo();
  }
  const CallLowering *getCallLowering() const override;
  const InlineAsmLowering *getInlineAsmLowering() const override;
  InstructionSelector *getInstructionSelector() const override;
  const LegalizerInfo *getLegalizerInfo() const override;
  const RegisterBankInfo *getRegBankInfo() const override;
  const Triple &getTargetTriple() const { return TargetTriple; }
  bool enableMachineScheduler() const override { return true; }
  bool enablePostRAScheduler() const override { return usePostRAScheduler(); }
  bool enableSubRegLiveness() const override { return EnableSubregLiveness; }
  bool enableSpillageCopyElimination() const override { return true; }

  bool enableMachinePipeliner() const override;
  bool useDFAforSMS() const override { return false; }

  /// Returns ARM processor family.
  /// Avoid this function! CPU specifics should be kept local to this class
  /// and preferably modeled with SubtargetFeatures or properties in
  /// initializeProperties().
  ARMProcFamilyEnum getProcFamily() const {
    return ARMProcFamily;
  }
```
**EN:** This block implements getSelectionDAGInfo, advancing the file's subtarget feature modeling flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 getSelectionDAGInfo，通过 AArch64 专用的决策与数据处理推进本文件的子目标特性建模流程。
### Lines 174-204: Function isAppleMLike
```cpp

  /// Returns true if the processor is an Apple M-series or aligned A-series
  /// (A14 or newer).
  bool isAppleMLike() const {
    switch (ARMProcFamily) {
    case AppleA14:
    case AppleA15:
    case AppleA16:
    case AppleA17:
    case AppleM4:
    case AppleM5:
      return true;
    default:
      return false;
    }
  }

  bool isXRaySupported() const override { return true; }

  /// Returns true if the function has a streaming body.
  bool isStreaming() const { return IsStreaming; }

  /// Returns true if the function has a streaming-compatible body.
  bool isStreamingCompatible() const { return IsStreamingCompatible; }

  /// Returns the size of memory region that if accessed by both the CPU and
  /// the SME unit could result in a hazard. 0 = disabled.
  unsigned getStreamingHazardSize() const {
    return StreamingHazardSize.value_or(
        !hasSMEFA64() && hasSME() && hasSVE() ? 1024 : 0);
  }
```
**EN:** This block implements isAppleMLike, advancing the file's subtarget feature modeling flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 isAppleMLike，通过 AArch64 专用的决策与数据处理推进本文件的子目标特性建模流程。
### Lines 205-237: Function enabled
```cpp

  /// Returns true if the target has NEON and the function at runtime is known
  /// to have NEON enabled (e.g. the function is known not to be in streaming-SVE
  /// mode, which disables NEON instructions).
  bool isNeonAvailable() const {
    return hasNEON() &&
           (hasSMEFA64() || (!isStreaming() && !isStreamingCompatible()));
  }

  /// Returns true if the target has SVE and can use the full range of SVE
  /// instructions, for example because it knows the function is known not to be
  /// in streaming-SVE mode or when the target has FEAT_FA64 enabled.
  bool isSVEAvailable() const {
    return hasSVE() &&
           (hasSMEFA64() || (!isStreaming() && !isStreamingCompatible()));
  }

  /// Returns true if the target has access to the streaming-compatible subset
  /// of SVE instructions.
  bool isStreamingSVEAvailable() const { return hasSME() && isStreaming(); }

  /// Returns true if the target has access to either the full range of SVE
  /// instructions, or the streaming-compatible subset of SVE instructions.
  bool isSVEorStreamingSVEAvailable() const {
    return hasSVE() || isStreamingSVEAvailable();
  }

  /// Returns true if the target has access to either the full range of SVE
  /// instructions, or the streaming-compatible subset of SVE instructions
  /// available to SME2.
  bool isNonStreamingSVEorSME2Available() const {
    return isSVEAvailable() || (isSVEorStreamingSVEAvailable() && hasSME2());
  }
```
**EN:** This block implements enabled, advancing the file's subtarget feature modeling flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 enabled，通过 AArch64 专用的决策与数据处理推进本文件的子目标特性建模流程。
### Lines 238-269: Function getMinVectorRegisterBitWidth
```cpp

  unsigned getMinVectorRegisterBitWidth() const {
    // Don't assume any minimum vector size when PSTATE.SM may not be 0, because
    // we don't yet support streaming-compatible codegen support that we trust
    // is safe for functions that may be executed in streaming-SVE mode.
    // By returning '0' here, we disable vectorization.
    if (!isSVEAvailable() && !isNeonAvailable())
      return 0;
    return MinVectorRegisterBitWidth;
  }

  bool isXRegisterReserved(size_t i) const { return ReserveXRegister[i]; }
  bool isXRegisterReservedForRA(size_t i) const { return ReserveXRegisterForRA[i]; }
  unsigned getNumXRegisterReserved() const {
    BitVector AllReservedX(AArch64::GPR64commonRegClass.getNumRegs());
    AllReservedX |= ReserveXRegister;
    AllReservedX |= ReserveXRegisterForRA;
    return AllReservedX.count();
  }
  bool isLRReservedForRA() const { return ReserveLRForRA; }
  bool isXRegCustomCalleeSaved(size_t i) const {
    return CustomCallSavedXRegs[i];
  }
  bool hasCustomCallingConv() const { return CustomCallSavedXRegs.any(); }

  /// Return true if the CPU supports any kind of instruction fusion.
  bool hasFusion() const {
    return hasArithmeticBccFusion() || hasArithmeticCbzFusion() ||
           hasFuseAES() || hasFuseArithmeticLogic() || hasFuseCmpCSel() ||
           hasFuseFCmpFCSel() || hasFuseCmpCSet() || hasFuseAdrpAdd() ||
           hasFuseLiterals();
  }
```
**EN:** This block implements getMinVectorRegisterBitWidth, advancing the file's subtarget feature modeling flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 getMinVectorRegisterBitWidth，通过 AArch64 专用的决策与数据处理推进本文件的子目标特性建模流程。
### Lines 270-301: Function getEpilogueVectorizationMinVF
```cpp

  unsigned getEpilogueVectorizationMinVF() const {
    return EpilogueVectorizationMinVF;
  }
  unsigned getMaxInterleaveFactor() const { return MaxInterleaveFactor; }
  unsigned getVectorInsertExtractBaseCost() const;
  unsigned getCacheLineSize() const override { return CacheLineSize; }
  unsigned getScatterOverhead() const { return ScatterOverhead; }
  unsigned getGatherOverhead() const { return GatherOverhead; }
  unsigned getPrefetchDistance() const override { return PrefetchDistance; }
  unsigned getMinPrefetchStride(unsigned NumMemAccesses,
                                unsigned NumStridedMemAccesses,
                                unsigned NumPrefetches,
                                bool HasCall) const override {
    return MinPrefetchStride;
  }
  unsigned getMaxPrefetchIterationsAhead() const override {
    return MaxPrefetchIterationsAhead;
  }
  Align getPrefFunctionAlignment() const {
    return PrefFunctionAlignment;
  }
  Align getPrefLoopAlignment() const { return PrefLoopAlignment; }

  unsigned getMaxBytesForLoopAlignment() const {
    return MaxBytesForLoopAlignment;
  }

  unsigned getMaximumJumpTableSize() const { return MaxJumpTableSize; }
  unsigned getMinimumJumpTableEntries() const {
    return MinimumJumpTableEntries;
  }
```
**EN:** This block implements getEpilogueVectorizationMinVF, advancing the file's subtarget feature modeling flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 getEpilogueVectorizationMinVF，通过 AArch64 专用的决策与数据处理推进本文件的子目标特性建模流程。
### Lines 302-333: Function TBI
```cpp

  /// CPU has TBI (top byte of addresses is ignored during HW address
  /// translation) and OS enables it.
  bool supportsAddressTopByteIgnored() const;

  bool isLittleEndian() const { return IsLittle; }

  bool isTargetDarwin() const { return TargetTriple.isOSDarwin(); }
  bool isTargetIOS() const { return TargetTriple.isiOS(); }
  bool isTargetLinux() const { return TargetTriple.isOSLinux(); }
  bool isTargetWindows() const { return TargetTriple.isOSWindows(); }
  bool isTargetAndroid() const { return TargetTriple.isAndroid(); }
  bool isTargetFuchsia() const { return TargetTriple.isOSFuchsia(); }
  bool isWindowsArm64EC() const { return TargetTriple.isWindowsArm64EC(); }
  bool isLFI() const { return TargetTriple.isLFI(); }

  bool isTargetCOFF() const { return TargetTriple.isOSBinFormatCOFF(); }
  bool isTargetELF() const { return TargetTriple.isOSBinFormatELF(); }
  bool isTargetMachO() const { return TargetTriple.isOSBinFormatMachO(); }

  bool isTargetILP32() const {
    return TargetTriple.isArch32Bit() ||
           TargetTriple.getEnvironment() == Triple::GNUILP32;
  }

  bool useAA() const override;

  bool addrSinkUsingGEPs() const override {
    // Keeping GEPs inbounds is important for exploiting AArch64
    // addressing-modes in ILP32 mode.
    return useAA() || isTargetILP32();
  }
```
**EN:** This block implements TBI, advancing the file's subtarget feature modeling flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 TBI，通过 AArch64 专用的决策与数据处理推进本文件的子目标特性建模流程。
### Lines 334-361: Function useSmallAddressing
```cpp

  bool useSmallAddressing() const {
    switch (TLInfo.getTargetMachine().getCodeModel()) {
      case CodeModel::Kernel:
        // Kernel is currently allowed only for Fuchsia targets,
        // where it is the same as Small for almost all purposes.
      case CodeModel::Small:
        return true;
      default:
        return false;
    }
  }

  /// Returns whether the operating system makes it safer to store sensitive
  /// values in x16 and x17 as opposed to other registers.
  bool isX16X17Safer() const;

  /// ParseSubtargetFeatures - Parses features string setting specified
  /// subtarget options.  Definition of function is auto generated by tblgen.
  void ParseSubtargetFeatures(StringRef CPU, StringRef TuneCPU, StringRef FS);

  /// ClassifyGlobalReference - Find the target operand flags that describe
  /// how a global value should be referenced for the current subtarget.
  unsigned ClassifyGlobalReference(const GlobalValue *GV,
                                   const TargetMachine &TM) const;

  unsigned classifyGlobalFunctionReference(const GlobalValue *GV,
                                           const TargetMachine &TM) const;
```
**EN:** This block implements useSmallAddressing, advancing the file's subtarget feature modeling flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 useSmallAddressing，通过 AArch64 专用的决策与数据处理推进本文件的子目标特性建模流程。
### Lines 362-396: Class TargetSubtargetInfo
```cpp

  /// This function is design to compatible with the function def in other
  /// targets and escape build error about the virtual function def in base
  /// class TargetSubtargetInfo. Updeate me if AArch64 target need to use it.
  unsigned char
  classifyGlobalFunctionReference(const GlobalValue *GV) const override {
    return 0;
  }

  void overrideSchedPolicy(MachineSchedPolicy &Policy,
                           const SchedRegion &Region) const override;

  void adjustSchedDependency(SUnit *Def, int DefOpIdx, SUnit *Use, int UseOpIdx,
                             SDep &Dep,
                             const TargetSchedModel *SchedModel) const override;

  bool enableEarlyIfConversion() const override;

  std::unique_ptr<PBQPRAConstraint> getCustomPBQPConstraints() const override;

  bool isCallingConvWin64(CallingConv::ID CC, bool IsVarArg) const {
    switch (CC) {
    case CallingConv::C:
    case CallingConv::Fast:
    case CallingConv::Swift:
    case CallingConv::SwiftTail:
      return isTargetWindows();
    case CallingConv::PreserveNone:
      return IsVarArg && isTargetWindows();
    case CallingConv::Win64:
      return true;
    default:
      return false;
    }
  }
```
**EN:** This block defines TargetSubtargetInfo, packaging state and behavior that the file reuses for subtarget feature modeling.  
**CN:** 该代码块定义 TargetSubtargetInfo，把 子目标特性建模 所需的状态与行为封装在一起供后续复用。
### Lines 397-431: Function swiftAsyncContextIsDynamicallySet
```cpp

  /// Return whether FrameLowering should always set the "extended frame
  /// present" bit in FP, or set it based on a symbol in the runtime.
  bool swiftAsyncContextIsDynamicallySet() const {
    // Older OS versions (particularly system unwinders) are confused by the
    // Swift extended frame, so when building code that might be run on them we
    // must dynamically query the concurrency library to determine whether
    // extended frames should be flagged as present.
    const Triple &TT = getTargetTriple();

    unsigned Major = TT.getOSVersion().getMajor();
    switch(TT.getOS()) {
    default:
      return false;
    case Triple::IOS:
    case Triple::TvOS:
      return Major < 15;
    case Triple::WatchOS:
      return Major < 8;
    case Triple::MacOSX:
    case Triple::Darwin:
      return Major < 12;
    }
  }

  void mirFileLoaded(MachineFunction &MF) const override;

  // Return the known range for the bit length of SVE data registers. A value
  // of 0 means nothing is known about that particular limit beyond what's
  // implied by the architecture.
  unsigned getMaxSVEVectorSizeInBits() const {
    assert(isSVEorStreamingSVEAvailable() &&
           "Tried to get SVE vector length without SVE support!");
    return MaxSVEVectorSizeInBits;
  }
```
**EN:** This block implements swiftAsyncContextIsDynamicallySet, advancing the file's subtarget feature modeling flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 swiftAsyncContextIsDynamicallySet，通过 AArch64 专用的决策与数据处理推进本文件的子目标特性建模流程。
### Lines 432-464: Function getMinSVEVectorSizeInBits
```cpp

  unsigned getMinSVEVectorSizeInBits() const {
    assert(isSVEorStreamingSVEAvailable() &&
           "Tried to get SVE vector length without SVE support!");
    return MinSVEVectorSizeInBits;
  }

  // Return the known bit length of SVE data registers. A value of 0 means the
  // length is unknown beyond what's implied by the architecture.
  unsigned getSVEVectorSizeInBits() const {
    assert(isSVEorStreamingSVEAvailable() &&
           "Tried to get SVE vector length without SVE support!");
    if (MinSVEVectorSizeInBits == MaxSVEVectorSizeInBits)
      return MaxSVEVectorSizeInBits;
    return 0;
  }

  bool useSVEForFixedLengthVectors() const {
    if (!isSVEorStreamingSVEAvailable())
      return false;

    // Prefer NEON unless larger SVE registers are available.
    return !isNeonAvailable() || getMinSVEVectorSizeInBits() >= 256;
  }

  bool useSVEForFixedLengthVectors(EVT VT) const {
    if (!useSVEForFixedLengthVectors() || !VT.isFixedLengthVector())
      return false;
    return VT.getFixedSizeInBits() > AArch64::SVEBitsPerBlock ||
           !isNeonAvailable();
  }

  unsigned getVScaleForTuning() const { return VScaleForTuning; }
```
**EN:** This block implements getMinSVEVectorSizeInBits, advancing the file's subtarget feature modeling flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 getMinSVEVectorSizeInBits，通过 AArch64 专用的决策与数据处理推进本文件的子目标特性建模流程。
### Lines 465-496: Function getSVETailFoldingDefaultOpts
```cpp

  TailFoldingOpts getSVETailFoldingDefaultOpts() const {
    return DefaultSVETFOpts;
  }

  /// Returns true to use the addvl/inc/dec instructions, as opposed to separate
  /// add + cnt instructions.
  bool useScalarIncVL() const;

  bool enableSRLTSubregToRegMitigation() const {
    return EnableSRLTSubregToRegMitigation;
  }

  /// Choose a method of checking LR before performing a tail call.
  AArch64PAuth::AuthCheckMethod
  getAuthenticatedLRCheckMethod(const MachineFunction &MF) const;

  /// Compute the integer discriminator for a given BlockAddress constant, if
  /// blockaddress signing is enabled, or std::nullopt otherwise.
  /// Blockaddress signing is controlled by the function attribute
  /// "ptrauth-indirect-gotos" on the parent function.
  /// Note that this assumes the discriminator is independent of the indirect
  /// goto branch site itself, i.e., it's the same for all BlockAddresses in
  /// a function.
  std::optional<uint16_t>
  getPtrAuthBlockAddressDiscriminatorIfEnabled(const Function &ParentFn) const;

  bool enableAggressiveInterleaving() const { return AggressiveInterleaving; }
};
} // End llvm namespace

#endif
```
**EN:** This block implements getSVETailFoldingDefaultOpts, advancing the file's subtarget feature modeling flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 getSVETailFoldingDefaultOpts，通过 AArch64 专用的决策与数据处理推进本文件的子目标特性建模流程。
## Key Concepts / 关键概念
- **EN:** AArch64 target backend structure **CN:** AArch64 目标后端结构
- **EN:** CPU feature gating and micro-architecture tuning **CN:** CPU 特性开关与微架构调优
## Dependencies / 依赖关系
- **EN:** Target-local includes: AArch64FrameLowering.h, AArch64ISelLowering.h, AArch64InstrInfo.h, AArch64PointerAuth.h, AArch64RegisterInfo.h, AArch64SelectionDAGInfo.h, AArch64GenSubtargetInfo.inc, AArch64GenSubtargetInfo.inc, AArch64GenSubtargetInfo.inc **CN:** 目标本地依赖：AArch64FrameLowering.h, AArch64ISelLowering.h, AArch64InstrInfo.h, AArch64PointerAuth.h, AArch64RegisterInfo.h, AArch64SelectionDAGInfo.h, AArch64GenSubtargetInfo.inc, AArch64GenSubtargetInfo.inc, AArch64GenSubtargetInfo.inc
- **EN:** Core LLVM interfaces: llvm/CodeGen/GlobalISel/CallLowering.h, llvm/CodeGen/GlobalISel/InlineAsmLowering.h, llvm/CodeGen/GlobalISel/InstructionSelector.h, llvm/CodeGen/GlobalISel/LegalizerInfo.h, llvm/CodeGen/RegisterBankInfo.h, llvm/CodeGen/TargetSubtargetInfo.h, llvm/IR/DataLayout.h, llvm/TargetParser/Triple.h, llvm/TargetParser/AArch64TargetParserDef.inc **CN:** 核心 LLVM 接口：llvm/CodeGen/GlobalISel/CallLowering.h, llvm/CodeGen/GlobalISel/InlineAsmLowering.h, llvm/CodeGen/GlobalISel/InstructionSelector.h, llvm/CodeGen/GlobalISel/LegalizerInfo.h, llvm/CodeGen/RegisterBankInfo.h, llvm/CodeGen/TargetSubtargetInfo.h, llvm/IR/DataLayout.h, llvm/TargetParser/Triple.h, llvm/TargetParser/AArch64TargetParserDef.inc
- **EN:** Closely connected with neighboring AArch64 backend components responsible for subtarget feature modeling. **CN:** 与周边负责子目标特性建模的 AArch64 后端组件紧密协作。
