# AMDGPUSubtarget.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/AMDGPUSubtarget.h`
- **Repository**: llvm/llvm-project
- **Purpose**: This header declares the interfaces, data structures, and pass entry points for AMDGPUSubtarget in the LLVM AMDGPU backend. It defines the contracts consumed by other AMDGPU backend components. / 该头文件声明 LLVM AMDGPU 后端中 AMDGPUSubtarget 的接口、数据结构与 Pass 入口。它定义了其他 AMDGPU 后端组件依赖的契约。

## Line-by-Line Analysis / 逐行分析
### Lines 1-29: File banner, includes, and setup
```cpp
//=====-- AMDGPUSubtarget.h - Define Subtarget for AMDGPU -------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//==-----------------------------------------------------------------------===//
//
/// \file
/// Base class for AMDGPU specific classes of TargetSubtarget.
//
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_AMDGPU_AMDGPUSUBTARGET_H
#define LLVM_LIB_TARGET_AMDGPU_AMDGPUSUBTARGET_H

#include "llvm/ADT/SmallVector.h"
#include "llvm/IR/CallingConv.h"
#include "llvm/Support/Alignment.h"
#include "llvm/TargetParser/Triple.h"

namespace llvm {

enum AMDGPUDwarfFlavour : unsigned;
class Function;
class Instruction;
class MachineFunction;
class TargetMachine;

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation. Main symbols: `AMDGPUDwarfFlavour`, `Function`, `Instruction`.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。 主要符号：`AMDGPUDwarfFlavour`, `Function`, `Instruction`。

### Lines 30-63: Declares class AMDGPUSubtarget
```cpp
class AMDGPUSubtarget {
public:
  enum Generation {
    INVALID = 0,
    R600 = 1,
    R700 = 2,
    EVERGREEN = 3,
    NORTHERN_ISLANDS = 4,
    SOUTHERN_ISLANDS = 5,
    SEA_ISLANDS = 6,
    VOLCANIC_ISLANDS = 7,
    GFX9 = 8,
    GFX10 = 9,
    GFX11 = 10,
    GFX12 = 11,
    GFX13 = 12,
  };

private:
  const Triple &TargetTriple;

protected:
  bool HasMulI24 = true;
  bool HasMulU24 = true;
  bool HasSMulHi = false;
  bool HasFminFmaxLegacy = true;

  unsigned EUsPerCU = 4;
  unsigned MaxWavesPerEU = 10;
  unsigned LocalMemorySize = 0;
  unsigned AddressableLocalMemorySize = 0;
  char WavefrontSizeLog2 = 0;
  unsigned FlatOffsetBitWidth = 0;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `AMDGPUSubtarget`, `Generation`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`AMDGPUSubtarget`, `Generation`。

### Lines 64-92: Defines AMDGPUSubtarget
```cpp
public:
  AMDGPUSubtarget(const Triple &TT) : TargetTriple(TT) {}

  static const AMDGPUSubtarget &get(const MachineFunction &MF);
  static const AMDGPUSubtarget &get(const TargetMachine &TM,
                                    const Function &F);

  /// \returns Default range flat work group size for a calling convention.
  std::pair<unsigned, unsigned> getDefaultFlatWorkGroupSize(CallingConv::ID CC) const;

  /// \returns Subtarget's default pair of minimum/maximum flat work group sizes
  /// for function \p F, or minimum/maximum flat work group sizes explicitly
  /// requested using "amdgpu-flat-work-group-size" attribute attached to
  /// function \p F.
  ///
  /// \returns Subtarget's default values if explicitly requested values cannot
  /// be converted to integer, or violate subtarget's specifications.
  std::pair<unsigned, unsigned> getFlatWorkGroupSizes(const Function &F) const;

  /// \returns true if the maximum flat work-group size for \p F is at most the
  /// wavefront size, so a work-group may fit in a single wavefront.
  bool isSingleWavefrontWorkgroup(const Function &F) const;

  /// \returns The required size of workgroups that will be used to execute \p F
  /// in the \p Dim dimension, if it is known (from `!reqd_work_group_size`
  /// metadata. Otherwise, returns std::nullopt.
  std::optional<unsigned> getReqdWorkGroupSize(const Function &F,
                                               unsigned Dim) const;

```
**EN:** This section contains concrete logic for AMDGPUSubtarget. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 AMDGPUSubtarget 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 93-125: Defines hasWavefrontsEvenlySplittingXDim
```cpp
  /// \returns true if \p F will execute in a manner that leaves the X
  /// dimensions of the workitem ID evenly tiling wavefronts - that is, if X /
  /// wavefrontsize is uniform. This is true if either the Y and Z block
  /// dimensions are known to always be 1 or if the X dimension will always be a
  /// power of 2. If \p RequireUniformYZ is true, it also ensures that the Y and
  /// Z workitem IDs will be uniform (so, while a (32, 2, 1) launch with
  /// wavesize64 would ordinarily pass this test, it won't with
  /// \pRequiresUniformYZ).
  ///
  /// This information is currently only gathered from the !reqd_work_group_size
  /// metadata on \p F, but this may be improved in the future.
  bool hasWavefrontsEvenlySplittingXDim(const Function &F,
                                        bool REquiresUniformYZ = false) const;

  /// \returns Subtarget's default pair of minimum/maximum number of waves per
  /// execution unit for function \p F, or minimum/maximum number of waves per
  /// execution unit explicitly requested using "amdgpu-waves-per-eu" attribute
  /// attached to function \p F.
  ///
  /// \returns Subtarget's default values if explicitly requested values cannot
  /// be converted to integer, violate subtarget's specifications, or are not
  /// compatible with minimum/maximum number of waves limited by flat work group
  /// size, register usage, and/or lds usage.
  std::pair<unsigned, unsigned> getWavesPerEU(const Function &F) const;

  /// Overload which uses the specified values for the flat workgroup sizes and
  /// LDS space rather than querying the function itself. \p FlatWorkGroupSizes
  /// should correspond to the function's value for getFlatWorkGroupSizes and \p
  /// LDSBytes to the per-workgroup LDS allocation.
  std::pair<unsigned, unsigned>
  getWavesPerEU(std::pair<unsigned, unsigned> FlatWorkGroupSizes,
                unsigned LDSBytes, const Function &F) const;

```
**EN:** This section contains concrete logic for hasWavefrontsEvenlySplittingXDim. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 hasWavefrontsEvenlySplittingXDim 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 126-156: Defines getEffectiveWavesPerEU
```cpp
  /// Returns the target minimum/maximum number of waves per EU. This is based
  /// on the minimum/maximum number of \p RequestedWavesPerEU and further
  /// limited by the maximum achievable occupancy derived from the range of \p
  /// FlatWorkGroupSizes and number of \p LDSBytes per workgroup.
  std::pair<unsigned, unsigned>
  getEffectiveWavesPerEU(std::pair<unsigned, unsigned> RequestedWavesPerEU,
                         std::pair<unsigned, unsigned> FlatWorkGroupSizes,
                         unsigned LDSBytes) const;

  /// Return the amount of LDS that can be used that will not restrict the
  /// occupancy lower than WaveCount.
  unsigned getMaxLocalMemSizeWithWaveCount(unsigned WaveCount,
                                           const Function &) const;

  /// Subtarget's minimum/maximum occupancy, in number of waves per EU, that can
  /// be achieved when the only function running on a CU is \p F and each
  /// workgroup running the function requires \p LDSBytes bytes of LDS space.
  /// This notably depends on the range of allowed flat group sizes for the
  /// function and hardware characteristics.
  std::pair<unsigned, unsigned>
  getOccupancyWithWorkGroupSizes(uint32_t LDSBytes, const Function &F) const {
    return getOccupancyWithWorkGroupSizes(LDSBytes, getFlatWorkGroupSizes(F));
  }

  /// Overload which uses the specified values for the flat work group sizes,
  /// rather than querying the function itself. \p FlatWorkGroupSizes should
  /// correspond to the function's value for getFlatWorkGroupSizes.
  std::pair<unsigned, unsigned> getOccupancyWithWorkGroupSizes(
      uint32_t LDSBytes,
      std::pair<unsigned, unsigned> FlatWorkGroupSizes) const;

```
**EN:** This section contains concrete logic for getEffectiveWavesPerEU. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 getEffectiveWavesPerEU 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 157-184: Declares getOccupancyWithWorkGroupSizes
```cpp
  /// Subtarget's minimum/maximum occupancy, in number of waves per EU, that can
  /// be achieved when the only function running on a CU is \p MF. This notably
  /// depends on the range of allowed flat group sizes for the function, the
  /// amount of per-workgroup LDS space required by the function, and hardware
  /// characteristics.
  std::pair<unsigned, unsigned>
  getOccupancyWithWorkGroupSizes(const MachineFunction &MF) const;

  bool isAmdHsaOS() const {
    return TargetTriple.getOS() == Triple::AMDHSA;
  }

  bool isAmdPalOS() const {
    return TargetTriple.getOS() == Triple::AMDPAL;
  }

  bool isMesa3DOS() const {
    return TargetTriple.getOS() == Triple::Mesa3D;
  }

  bool isMesaKernel(const Function &F) const;

  bool isAmdHsaOrMesa(const Function &F) const {
    return isAmdHsaOS() || isMesaKernel(F);
  }

  bool isGCN() const { return TargetTriple.isAMDGCN(); }

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 185-218: Header dependencies and setup
```cpp
  //==---------------------------------------------------------------------===//
  // TableGen-generated feature getters.
  //==---------------------------------------------------------------------===//
#define GET_SUBTARGETINFO_MACRO(ATTRIBUTE, DEFAULT, GETTER)                    \
  virtual bool GETTER() const { return false; }
#include "AMDGPUGenSubtargetInfo.inc"
  //==---------------------------------------------------------------------===//

  /// Return true if real (non-fake) variants of True16 instructions using
  /// 16-bit registers should be code-generated. Fake True16 instructions are
  /// identical to non-fake ones except that they take 32-bit registers as
  /// operands and always use their low halves.
  // TODO: Remove and use hasTrue16BitInsts() instead once True16 is fully
  // supported and the support for fake True16 instructions is removed.
  bool useRealTrue16Insts() const {
    return hasTrue16BitInsts() && enableRealTrue16Insts();
  }

  bool hasMulI24() const {
    return HasMulI24;
  }

  bool hasMulU24() const {
    return HasMulU24;
  }

  bool hasSMulHi() const {
    return HasSMulHi;
  }

  bool hasFminFmaxLegacy() const {
    return HasFminFmaxLegacy;
  }

```
**EN:** These includes pull in the LLVM and AMDGPU declarations required by the rest of the file, making later symbols and helper APIs available.
**CN:** 这些头文件引入本文件后续实现所需的 LLVM 与 AMDGPU 声明，使后面的符号和辅助 API 可以直接使用。

### Lines 219-251: Defines getWavefrontSize
```cpp
  unsigned getWavefrontSize() const {
    return 1 << WavefrontSizeLog2;
  }

  unsigned getWavefrontSizeLog2() const {
    return WavefrontSizeLog2;
  }

  /// Return the maximum number of bytes of LDS available for all workgroups
  /// running on the same WGP or CU.
  /// For GFX10-GFX12 in WGP mode this is 128k even though each workgroup is
  /// limited to 64k.
  unsigned getLocalMemorySize() const {
    return LocalMemorySize;
  }

  /// Return the maximum number of bytes of LDS that can be allocated to a
  /// single workgroup.
  /// For GFX10-GFX12 in WGP mode this is limited to 64k even though the WGP has
  /// 128k in total.
  unsigned getAddressableLocalMemorySize() const {
    return AddressableLocalMemorySize;
  }

  /// Number of SIMDs/EUs (execution units) per "CU" ("compute unit"), where the
  /// "CU" is the unit onto which workgroups are mapped. This takes WGP mode vs.
  /// CU mode into account.
  unsigned getEUsPerCU() const { return EUsPerCU; }

  Align getAlignmentForImplicitArgPtr() const {
    return isAmdHsaOS() ? Align(8) : Align(4);
  }

```
**EN:** This section contains concrete logic for getWavefrontSize. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 getWavefrontSize 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 252-284: Defines getExplicitKernelArgOffset
```cpp
  /// Returns the offset in bytes from the start of the input buffer
  ///        of the first explicit kernel argument.
  unsigned getExplicitKernelArgOffset() const {
    switch (TargetTriple.getOS()) {
    case Triple::AMDHSA:
    case Triple::AMDPAL:
    case Triple::Mesa3D:
      return 0;
    case Triple::UnknownOS:
    default:
      // For legacy reasons unknown/other is treated as a different version of
      // mesa.
      return 36;
    }

    llvm_unreachable("invalid triple OS");
  }

  /// \returns Maximum number of work groups per compute unit supported by the
  /// subtarget and limited by given \p FlatWorkGroupSize.
  virtual unsigned getMaxWorkGroupsPerCU(unsigned FlatWorkGroupSize) const = 0;

  /// \returns Minimum flat work group size supported by the subtarget.
  virtual unsigned getMinFlatWorkGroupSize() const = 0;

  /// \returns Maximum flat work group size supported by the subtarget.
  virtual unsigned getMaxFlatWorkGroupSize() const = 0;

  /// \returns Number of waves per execution unit required to support the given
  /// \p FlatWorkGroupSize.
  virtual unsigned
  getWavesPerEUForWorkGroup(unsigned FlatWorkGroupSize) const = 0;

```
**EN:** This section contains concrete logic for getExplicitKernelArgOffset. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 getExplicitKernelArgOffset 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 285-318: Declares getMinWavesPerEU
```cpp
  /// \returns Minimum number of waves per execution unit supported by the
  /// subtarget.
  virtual unsigned getMinWavesPerEU() const = 0;

  /// \returns Maximum number of waves per execution unit supported by the
  /// subtarget without any kind of limitation.
  unsigned getMaxWavesPerEU() const { return MaxWavesPerEU; }

  /// Return the maximum workitem ID value in the function, for the given (0, 1,
  /// 2) dimension.
  unsigned getMaxWorkitemID(const Function &Kernel, unsigned Dimension) const;

  /// Return the number of work groups for the function.
  SmallVector<unsigned> getMaxNumWorkGroups(const Function &F) const;

  /// Return true if only a single workitem can be active in a wave.
  bool isSingleLaneExecution(const Function &Kernel) const;

  /// Creates value range metadata on an workitemid.* intrinsic call or load.
  bool makeLIDRangeMetadata(Instruction *I) const;

  /// \returns Number of bytes of arguments that are passed to a shader or
  /// kernel in addition to the explicit ones declared for the function.
  unsigned getImplicitArgNumBytes(const Function &F) const;
  uint64_t getExplicitKernArgSize(const Function &F, Align &MaxAlign) const;
  unsigned getKernArgSegmentSize(const Function &F, Align &MaxAlign) const;

  /// \returns Corresponding DWARF register number mapping flavour for the
  /// \p WavefrontSize.
  AMDGPUDwarfFlavour getAMDGPUDwarfFlavour() const;

  virtual ~AMDGPUSubtarget() = default;
};

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 319-321: Preprocessor guards and macros
```cpp
} // end namespace llvm

#endif // LLVM_LIB_TARGET_AMDGPU_AMDGPUSUBTARGET_H
```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。

## Key Concepts / 关键概念
- **Language / 语言**: C++ header
- **Primary symbols / 主要符号**: `AMDGPUDwarfFlavour`, `Function`, `Instruction`, `MachineFunction`, `TargetMachine`, `AMDGPUSubtarget`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; subtarget modeling / 子目标建模; assembly handling / 汇编处理
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"llvm/ADT/SmallVector.h"`
- `"llvm/IR/CallingConv.h"`
- `"llvm/Support/Alignment.h"`
- `"llvm/TargetParser/Triple.h"`
- `"AMDGPUGenSubtargetInfo.inc"`
