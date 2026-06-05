# RISCVSubtarget.h — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVSubtarget.h`
- **Repository**: llvm/llvm-project
- **Purpose**: Declares interfaces, data structures, and helper APIs for CPU-specific and feature-specific subtarget modeling for RISC-V. / 声明RISC-V 的 CPU 专用与特性专用子目标建模所需的接口、数据结构与辅助 API。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: Commentary and design intent / 注释与设计意图
```cpp
//===-- RISCVSubtarget.h - Define Subtarget for the RISC-V ------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file declares the RISC-V specific subclass of TargetSubtargetInfo.
//
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_RISCV_RISCVSUBTARGET_H
#define LLVM_LIB_TARGET_RISCV_RISCVSUBTARGET_H
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 16-32: Header imports and compile-time dependencies / 头文件导入与编译期依赖
```cpp
#include "GISel/RISCVRegisterBankInfo.h"
#include "MCTargetDesc/RISCVBaseInfo.h"
#include "RISCVFrameLowering.h"
#include "RISCVISelLowering.h"
#include "RISCVInstrInfo.h"
#include "llvm/CodeGen/GlobalISel/CallLowering.h"
#include "llvm/CodeGen/GlobalISel/InlineAsmLowering.h"
#include "llvm/CodeGen/GlobalISel/InstructionSelector.h"
#include "llvm/CodeGen/GlobalISel/LegalizerInfo.h"
#include "llvm/CodeGen/MachineScheduler.h"
#include "llvm/CodeGen/TargetSubtargetInfo.h"
#include "llvm/IR/DataLayout.h"
#include "llvm/Support/Compiler.h"
#include "llvm/Target/TargetMachine.h"
#include <bitset>
#include <memory>
```
**EN:** This block gathers the headers required by the file, revealing which LLVM layers and helper utilities the implementation relies on.

**CN:** 该区段汇集文件所需的头文件，体现实现依赖的 LLVM 层次与辅助工具。

### Lines 33-48: Header imports and compile-time dependencies / 头文件导入与编译期依赖
```cpp
#define GET_RISCV_MACRO_FUSION_PRED_DECL
#include "RISCVGenMacroFusion.inc"

#define GET_SUBTARGETINFO_HEADER
#include "RISCVGenSubtargetInfo.inc"

namespace llvm {
class StringRef;

namespace RISCVTuneInfoTable {

struct RISCVTuneInfo {
  const char *Name;
  uint8_t PrefFunctionAlignment;
  uint8_t PrefLoopAlignment;
```
**EN:** This block gathers the headers required by the file, revealing which LLVM layers and helper utilities the implementation relies on.

**CN:** 该区段汇集文件所需的头文件，体现实现依赖的 LLVM 层次与辅助工具。

### Lines 49-62: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  // Information needed by LoopDataPrefetch.
  uint16_t CacheLineSize;
  uint16_t PrefetchDistance;
  uint16_t MinPrefetchStride;
  unsigned MaxPrefetchIterationsAhead;

  unsigned MinimumJumpTableEntries;

  // Tail duplication threshold at -O3.
  unsigned TailDupAggressiveThreshold;

  unsigned MaxStoresPerMemsetOptSize;
  unsigned MaxStoresPerMemset;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 63-76: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  unsigned MaxGluedStoresPerMemcpy;
  unsigned MaxStoresPerMemcpyOptSize;
  unsigned MaxStoresPerMemcpy;

  unsigned MaxStoresPerMemmoveOptSize;
  unsigned MaxStoresPerMemmove;

  unsigned MaxLoadsPerMemcmpOptSize;
  unsigned MaxLoadsPerMemcmp;

  // The direction of PostRA scheduling.
  MISched::Direction PostRASchedDirection;
};
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 77-90: File prologue and imported dependencies / 文件前言与导入依赖
```cpp
#define GET_RISCVTuneInfoTable_DECL
#include "RISCVGenSearchableTables.inc"
} // namespace RISCVTuneInfoTable

class RISCVSubtarget : public RISCVGenSubtargetInfo {
public:
  // clang-format off
  enum RISCVProcFamilyEnum : uint8_t {
    Others,
    SiFive7,
    VentanaVeyron,
    MIPSP8700,
    Andes45,
  };
```
**EN:** This opening block combines banner comments with the first wave of includes, giving readers both context and the main compile-time dependencies.

**CN:** 该开头区段同时包含说明性注释与首批 include，让读者先看到文件定位，再看到主要编译期依赖。

### Lines 91-107: File prologue and imported dependencies / 文件前言与导入依赖
```cpp
  enum RISCVVRGatherCostModelEnum : uint8_t {
    Quadratic,
    NLog2N,
  };
  // clang-format on
private:
  virtual void anchor();

  RISCVProcFamilyEnum RISCVProcFamily = Others;
  RISCVVRGatherCostModelEnum RISCVVRGatherCostModel = Quadratic;

  bool IsLittleEndian = true;

#define GET_SUBTARGETINFO_MACRO(ATTRIBUTE, DEFAULT, GETTER) \
  bool ATTRIBUTE = DEFAULT;
#include "RISCVGenSubtargetInfo.inc"
```
**EN:** This opening block combines banner comments with the first wave of includes, giving readers both context and the main compile-time dependencies.

**CN:** 该开头区段同时包含说明性注释与首批 include，让读者先看到文件定位，再看到主要编译期依赖。

### Lines 108-128: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  unsigned XSfmmTE = 0;
  unsigned ZvlLen = 0;
  unsigned RVVVectorBitsMin;
  unsigned RVVVectorBitsMax;
  uint8_t MaxInterleaveFactor = 2;
  RISCVABI::ABI TargetABI = RISCVABI::ABI_Unknown;
  std::bitset<RISCV::NUM_TARGET_REGS> UserReservedRegister;
  const RISCVTuneInfoTable::RISCVTuneInfo *TuneInfo;

  RISCVFrameLowering FrameLowering;
  RISCVInstrInfo InstrInfo;
  RISCVTargetLowering TLInfo;

  /// Initializes using the passed in CPU and feature strings so that we can
  /// use initializer lists for subtarget initialization.
  RISCVSubtarget &initializeSubtargetDependencies(const Triple &TT,
                                                  StringRef CPU,
                                                  StringRef TuneCPU,
                                                  StringRef FS,
                                                  StringRef ABIName);
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 129-143: Function implementation: getFrameLowering / 函数实现：getFrameLowering
```cpp
public:
  // Initializes the data members to match that of the specified triple.
  RISCVSubtarget(const Triple &TT, StringRef CPU, StringRef TuneCPU,
                 StringRef FS, StringRef ABIName, unsigned RVVVectorBitsMin,
                 unsigned RVVVectorLMULMax, const TargetMachine &TM);

  ~RISCVSubtarget() override;

  // Parses features string setting specified subtarget options. The
  // definition of this function is auto-generated by tblgen.
  void ParseSubtargetFeatures(StringRef CPU, StringRef TuneCPU, StringRef FS);

  const RISCVFrameLowering *getFrameLowering() const override {
    return &FrameLowering;
  }
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 144-157: Function implementation: getInstrInfo / 函数实现：getInstrInfo
```cpp
  const RISCVInstrInfo *getInstrInfo() const override { return &InstrInfo; }
  const RISCVRegisterInfo *getRegisterInfo() const override {
    return &InstrInfo.getRegisterInfo();
  }
  const RISCVTargetLowering *getTargetLowering() const override {
    return &TLInfo;
  }

  void mirFileLoaded(MachineFunction &MF) const override;

  bool enableMachineScheduler() const override { return true; }

  bool enablePostRAScheduler() const override { return UsePostRAScheduler; }
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 158-172: Type declaration for getPrefFunctionAlignment / getPrefFunctionAlignment 的类型声明
```cpp
  Align getPrefFunctionAlignment() const {
    return Align(TuneInfo->PrefFunctionAlignment);
  }
  Align getPrefLoopAlignment() const {
    return Align(TuneInfo->PrefLoopAlignment);
  }

  /// Returns RISC-V processor family.
  /// Avoid this function! CPU specifics should be kept local to this class
  /// and preferably modeled with SubtargetFeatures or properties in
  /// initializeProperties().
  RISCVProcFamilyEnum getProcFamily() const { return RISCVProcFamily; }

  RISCVVRGatherCostModelEnum getVRGatherCostModel() const { return RISCVVRGatherCostModel; }
```
**EN:** This block declares a type, its members, and its responsibilities so later code can implement or consume the interface.

**CN:** 该区段声明类型、成员及其职责，供后续代码实现或使用该接口。

### Lines 173-187: Header imports and compile-time dependencies / 头文件导入与编译期依赖
```cpp
#define GET_SUBTARGETINFO_MACRO(ATTRIBUTE, DEFAULT, GETTER) \
  bool GETTER() const { return ATTRIBUTE; }
#include "RISCVGenSubtargetInfo.inc"

  LLVM_DEPRECATED("Now Equivalent to hasStdExtZcd", "hasStdExtZcd")
  bool hasStdExtCOrZcd() const { return HasStdExtZcd; }
  LLVM_DEPRECATED("Now Equivalent to hasStdExtZcf", "hasStdExtZcf")
  bool hasStdExtCOrZcfOrZce() const { return HasStdExtZcf; }
  bool hasStdExtZvl() const { return ZvlLen != 0; }
  bool hasStdExtFOrZfinx() const { return HasStdExtF || HasStdExtZfinx; }
  bool hasStdExtDOrZdinx() const { return HasStdExtD || HasStdExtZdinx; }
  bool hasStdExtZfhOrZhinx() const { return HasStdExtZfh || HasStdExtZhinx; }
  bool hasStdExtZfhminOrZhinxmin() const {
    return HasStdExtZfhmin || HasStdExtZhinxmin;
  }
```
**EN:** This block gathers the headers required by the file, revealing which LLVM layers and helper utilities the implementation relies on.

**CN:** 该区段汇集文件所需的头文件，体现实现依赖的 LLVM 层次与辅助工具。

### Lines 188-201: Function implementation: hasHalfFPLoadStoreMove / 函数实现：hasHalfFPLoadStoreMove
```cpp
  bool hasHalfFPLoadStoreMove() const {
    return HasStdExtZfhmin || HasStdExtZfbfmin;
  }

  bool hasCLZLike() const {
    return HasStdExtZbb || HasVendorXTHeadBb ||
           (HasVendorXCVbitmanip && !IsRV64);
  }
  bool hasCTZLike() const {
    return HasStdExtZbb || (HasVendorXCVbitmanip && !IsRV64);
  }
  bool hasCPOPLike() const {
    return HasStdExtZbb || (HasVendorXCVbitmanip && !IsRV64);
  }
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 202-219: Function implementation: hasREV8Like / 函数实现：hasREV8Like
```cpp
  bool hasREV8Like() const {
    return HasStdExtZbb || HasStdExtZbkb || HasVendorXTHeadBb;
  }
  bool hasREVLike() const {
    return HasStdExtP || ((HasVendorXCVbitmanip || HasVendorXqcibm) && !IsRV64);
  }

  bool hasBEXTILike() const { return HasStdExtZbs || HasVendorXTHeadBs; }

  bool hasCZEROLike() const {
    return HasStdExtZicond || HasVendorXVentanaCondOps;
  }

  bool hasConditionalMoveFusion() const {
    // Do we support fusing a branch+mv or branch+c.mv as a conditional move.
    return (hasConditionalCompressedMoveFusion() && hasStdExtZca()) ||
           hasShortForwardBranchIALU();
  }
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 220-233: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

  bool hasShlAdd(int64_t ShAmt) const {
    if (ShAmt <= 0)
      return false;
    if (ShAmt <= 3)
      return HasStdExtZba || HasVendorXAndesPerf || HasVendorXTHeadBa;
    return ShAmt <= 31 && HasVendorXqciac;
  }

  bool is64Bit() const { return IsRV64; }
  bool isLittleEndian() const { return IsLittleEndian; }
  MVT getXLenVT() const {
    return is64Bit() ? MVT::i64 : MVT::i32;
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 234-247: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  unsigned getXLen() const {
    return is64Bit() ? 64 : 32;
  }
  bool useMIPSLoadStorePairs() const;
  bool useMIPSCCMovInsn() const;
  unsigned getFLen() const {
    if (HasStdExtD)
      return 64;

    if (HasStdExtF)
      return 32;

    return 0;
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 248-262: Function implementation: getZilsdAlign / 函数实现：getZilsdAlign
```cpp

  Align getZilsdAlign() const {
    return Align(enableUnalignedScalarMem() ? 1
                 : allowZilsd4ByteAlign()   ? 4
                                            : 8);
  }

  unsigned getELen() const {
    assert(hasVInstructions() && "Expected V extension");
    return hasVInstructionsI64() ? 64 : 32;
  }
  unsigned getRealMinVLen() const {
    unsigned VLen = getMinRVVVectorSizeInBits();
    return VLen == 0 ? ZvlLen : VLen;
  }
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 263-282: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  unsigned getRealMaxVLen() const {
    unsigned VLen = getMaxRVVVectorSizeInBits();
    return VLen == 0 ? 65536 : VLen;
  }
  // If we know the exact VLEN, return it.  Otherwise, return std::nullopt.
  std::optional<unsigned> getRealVLen() const {
    unsigned Min = getRealMinVLen();
    if (Min != getRealMaxVLen())
      return std::nullopt;
    return Min;
  }

  /// If the ElementCount or TypeSize \p X is scalable and VScale (VLEN) is
  /// exactly known, returns \p X converted to a fixed quantity. Otherwise
  /// returns \p X unmodified.
  template <typename Quantity> Quantity expandVScale(Quantity X) const {
    if (auto VLen = getRealVLen(); VLen && X.isScalable()) {
      const unsigned VScale = *VLen / RISCV::RVVBitsPerBlock;
      X = Quantity::getFixed(X.getKnownMinValue() * VScale);
    }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 283-296: Function implementation: getTargetABI / 函数实现：getTargetABI
```cpp
    return X;
  }

  RISCVABI::ABI getTargetABI() const { return TargetABI; }
  bool isSoftFPABI() const {
    return TargetABI == RISCVABI::ABI_LP64 ||
           TargetABI == RISCVABI::ABI_ILP32 ||
           TargetABI == RISCVABI::ABI_ILP32E;
  }
  bool isRegisterReservedByUser(Register i) const override {
    assert(i.id() < RISCV::NUM_TARGET_REGS && "Register out of range");
    return UserReservedRegister[i.id()];
  }
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 297-316: Function implementation: isXRaySupported / 函数实现：isXRaySupported
```cpp
  // XRay support - require D and C extensions.
  bool isXRaySupported() const override { return hasStdExtD() && hasStdExtC(); }

  // Vector codegen related methods.
  bool hasVInstructions() const { return HasStdExtZve32x; }
  bool hasVInstructionsI64() const { return HasStdExtZve64x; }
  bool hasVInstructionsF16Minimal() const { return HasStdExtZvfhmin; }
  bool hasVInstructionsF16() const { return HasStdExtZvfh; }
  bool hasVInstructionsBF16Minimal() const {
    return HasStdExtZvfbfmin || HasStdExtZvfbfa;
  }
  bool hasVInstructionsF32() const { return HasStdExtZve32f; }
  bool hasVInstructionsF64() const { return HasStdExtZve64d; }
  bool hasVInstructionsBF16() const { return HasStdExtZvfbfa; }
  // F16 and F64 both require F32.
  bool hasVInstructionsAnyF() const { return hasVInstructionsF32(); }
  bool hasVInstructionsFullMultiply() const { return HasStdExtV; }
  unsigned getMaxInterleaveFactor() const {
    return hasVInstructions() ? MaxInterleaveFactor : 1;
  }
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 317-336: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

  bool hasOptimizedSegmentLoadStore(unsigned NF) const {
    switch (NF) {
    case 2:
      return hasOptimizedNF2SegmentLoadStore();
    case 3:
      return hasOptimizedNF3SegmentLoadStore();
    case 4:
      return hasOptimizedNF4SegmentLoadStore();
    case 5:
      return hasOptimizedNF5SegmentLoadStore();
    case 6:
      return hasOptimizedNF6SegmentLoadStore();
    case 7:
      return hasOptimizedNF7SegmentLoadStore();
    case 8:
      return hasOptimizedNF8SegmentLoadStore();
    default:
      llvm_unreachable("Unexpected NF");
    }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 337-353: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  }

  bool isPExtPackedType(MVT VT) const;
  bool isPExtPackedDoubleType(MVT VT) const;

  // Returns VLEN divided by DLEN. Where DLEN is the datapath width of the
  // vector hardware implementation which may be less than VLEN.
  unsigned getDLenFactor() const {
    if (DLenFactor2)
      return 2;
    return 1;
  }

protected:
  // SelectionDAGISel related APIs.
  std::unique_ptr<const SelectionDAGTargetInfo> TSInfo;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 354-367: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  // GlobalISel related APIs.
  mutable std::unique_ptr<CallLowering> CallLoweringInfo;
  mutable std::unique_ptr<InstructionSelector> InstSelector;
  mutable std::unique_ptr<LegalizerInfo> Legalizer;
  mutable std::unique_ptr<RISCVRegisterBankInfo> RegBankInfo;
  mutable std::unique_ptr<InlineAsmLowering> InlineAsmLoweringInfo;

  // Return the known range for the bit length of RVV data registers as set
  // at the command line. A value of 0 means nothing is known about that particular
  // limit beyond what's implied by the architecture.
  // NOTE: Please use getRealMinVLen and getRealMaxVLen instead!
  unsigned getMaxRVVVectorSizeInBits() const;
  unsigned getMinRVVVectorSizeInBits() const;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 368-384: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
public:
  const SelectionDAGTargetInfo *getSelectionDAGInfo() const override;
  const CallLowering *getCallLowering() const override;
  InstructionSelector *getInstructionSelector() const override;
  const LegalizerInfo *getLegalizerInfo() const override;
  const RISCVRegisterBankInfo *getRegBankInfo() const override;
  const InlineAsmLowering *getInlineAsmLowering() const override;

  bool isTargetAndroid() const { return getTargetTriple().isAndroid(); }
  bool isTargetFuchsia() const { return getTargetTriple().isOSFuchsia(); }

  bool useConstantPoolForLargeInts() const;

  // Maximum cost used for building integers, integers will be put into constant
  // pool if exceeded.
  unsigned getMaxBuildIntsCost() const;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 385-398: Function implementation: useDFAforSMS / 函数实现：useDFAforSMS
```cpp
  unsigned getMaxLMULForFixedLengthVectors() const;
  bool useRVVForFixedLengthVectors() const;

  bool enableSubRegLiveness() const override;

  bool enableMachinePipeliner() const override;

  bool useDFAforSMS() const override { return false; }

  bool useAA() const override;

  unsigned getCacheLineSize() const override {
    return TuneInfo->CacheLineSize;
  };
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 399-412: Function implementation: getPrefetchDistance / 函数实现：getPrefetchDistance
```cpp
  unsigned getPrefetchDistance() const override {
    return TuneInfo->PrefetchDistance;
  };
  unsigned getMinPrefetchStride(unsigned NumMemAccesses,
                                unsigned NumStridedMemAccesses,
                                unsigned NumPrefetches,
                                bool HasCall) const override {
    return TuneInfo->MinPrefetchStride;
  };
  unsigned getMaxPrefetchIterationsAhead() const override {
    return TuneInfo->MaxPrefetchIterationsAhead;
  };
  bool enableWritePrefetching() const override { return true; }
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 413-426: Function implementation: getTailDupAggressiveThreshold / 函数实现：getTailDupAggressiveThreshold
```cpp
  unsigned getMinimumJumpTableEntries() const;

  unsigned getTailDupAggressiveThreshold() const {
    return TuneInfo->TailDupAggressiveThreshold;
  }

  unsigned getMaxStoresPerMemset(bool OptSize) const {
    return OptSize ? TuneInfo->MaxStoresPerMemsetOptSize
                   : TuneInfo->MaxStoresPerMemset;
  }

  unsigned getMaxGluedStoresPerMemcpy() const {
    return TuneInfo->MaxGluedStoresPerMemcpy;
  }
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 427-441: Function implementation: getMaxStoresPerMemcpy / 函数实现：getMaxStoresPerMemcpy
```cpp

  unsigned getMaxStoresPerMemcpy(bool OptSize) const {
    return OptSize ? TuneInfo->MaxStoresPerMemcpyOptSize
                   : TuneInfo->MaxStoresPerMemcpy;
  }

  unsigned getMaxStoresPerMemmove(bool OptSize) const {
    return OptSize ? TuneInfo->MaxStoresPerMemmoveOptSize
                   : TuneInfo->MaxStoresPerMemmove;
  }

  unsigned getMaxLoadsPerMemcmp(bool OptSize) const {
    return OptSize ? TuneInfo->MaxLoadsPerMemcmpOptSize
                   : TuneInfo->MaxLoadsPerMemcmp;
  }
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 442-455: Header guard and interface framing / 头文件保护与接口框架
```cpp

  MISched::Direction getPostRASchedDirection() const {
    return TuneInfo->PostRASchedDirection;
  }

  void overrideSchedPolicy(MachineSchedPolicy &Policy,
                           const SchedRegion &Region) const override;

  void overridePostRASchedPolicy(MachineSchedPolicy &Policy,
                                 const SchedRegion &Region) const override;
};
} // namespace llvm

#endif
```
**EN:** This block establishes include guards and the outer structure of the public interface.

**CN:** 该区段建立头文件保护并给出公共接口的外层结构。

## Key Concepts / 关键概念
- **CPU and feature modeling** / **CPU 与特性建模**

## Dependencies / 依赖关系
- `GISel/RISCVRegisterBankInfo.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `MCTargetDesc/RISCVBaseInfo.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVFrameLowering.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVISelLowering.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVInstrInfo.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/GlobalISel/CallLowering.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/GlobalISel/InlineAsmLowering.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/GlobalISel/InstructionSelector.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/GlobalISel/LegalizerInfo.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/MachineScheduler.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/TargetSubtargetInfo.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/IR/DataLayout.h` — Directly referenced by this file. / 该文件直接引用的依赖。
