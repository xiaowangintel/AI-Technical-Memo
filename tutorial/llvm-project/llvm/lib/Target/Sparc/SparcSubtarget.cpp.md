# SparcSubtarget.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/Sparc/SparcSubtarget.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Captures CPU features and per-function subtarget configuration.
  - **CN**: 封装 CPU 特性以及按函数区分的子目标配置。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- SparcSubtarget.cpp - SPARC Subtarget Information ------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 8-12
```cpp
//
// This file implements the SPARC specific subclass of TargetSubtargetInfo.
//
//===----------------------------------------------------------------------===//

```
- **EN**: Documents the next group of declarations or explains a target-specific rule.
- **CN**: 为接下来的声明分组提供说明，或解释目标相关规则。

### Lines 13-18
```cpp
#include "SparcSubtarget.h"
#include "SparcSelectionDAGInfo.h"
#include "llvm/ADT/StringRef.h"
#include "llvm/MC/TargetRegistry.h"
#include "llvm/Support/MathExtras.h"

```
- **EN**: Pulls in the headers needed for this implementation, including `SparcSubtarget.h`, `SparcSelectionDAGInfo.h`, `llvm/ADT/StringRef.h`, `llvm/MC/TargetRegistry.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `SparcSubtarget.h`, `SparcSelectionDAGInfo.h`, `llvm/ADT/StringRef.h`, `llvm/MC/TargetRegistry.h`。

### Lines 19-26
```cpp
using namespace llvm;

#define DEBUG_TYPE "sparc-subtarget"

#define GET_SUBTARGETINFO_TARGET_DESC
#define GET_SUBTARGETINFO_CTOR
#include "SparcGenSubtargetInfo.inc"

```
- **EN**: Pulls in the headers needed for this implementation, including `SparcGenSubtargetInfo.inc`.
- **CN**: 引入该实现所需的头文件，其中包括 `SparcGenSubtargetInfo.inc`。

### Lines 27-36
```cpp
void SparcSubtarget::anchor() { }

SparcSubtarget &SparcSubtarget::initializeSubtargetDependencies(
    StringRef CPU, StringRef TuneCPU, StringRef FS) {
  const Triple &TT = getTargetTriple();
  // Determine default and user specified characteristics
  std::string CPUName = std::string(CPU);
  if (CPUName.empty())
    CPUName = TT.isSPARC64() ? "v9" : "v8";

```
- **EN**: Implements logic around `anchor`, `initializeSubtargetDependencies`, `getTargetTriple`, `string`, ...; this block applies conditional target rules.
- **CN**: 围绕 `anchor`, `initializeSubtargetDependencies`, `getTargetTriple`, `string`, ... 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 37-42
```cpp
  if (TuneCPU.empty())
    TuneCPU = CPUName;

  // Parse features string.
  ParseSubtargetFeatures(CPUName, TuneCPU, FS);

```
- **EN**: Implements logic around `ParseSubtargetFeatures`; this block applies conditional target rules.
- **CN**: 围绕 `ParseSubtargetFeatures` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 43-48
```cpp
  if (!Is64Bit && TT.isSPARC64()) {
    FeatureBitset Features = getFeatureBits();
    setFeatureBits(Features.set(Sparc::Feature64Bit));
    Is64Bit = true;
  }

```
- **EN**: Implements logic around `getFeatureBits`, `setFeatureBits`; this block applies conditional target rules.
- **CN**: 围绕 `getFeatureBits`, `setFeatureBits` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 49-55
```cpp
  // Popc is a v9-only instruction.
  if (!IsV9)
    UsePopc = false;

  return *this;
}

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 56-64
```cpp
SparcSubtarget::SparcSubtarget(const StringRef &CPU, const StringRef &TuneCPU,
                               const StringRef &FS, const TargetMachine &TM)
    : SparcGenSubtargetInfo(TM.getTargetTriple(), CPU, TuneCPU, FS),
      ReserveRegister(TM.getMCRegisterInfo().getNumRegs()),
      InstrInfo(initializeSubtargetDependencies(CPU, TuneCPU, FS)),
      TLInfo(TM, *this), FrameLowering(*this) {
  TSInfo = std::make_unique<SparcSelectionDAGInfo>();
}

```
- **EN**: Implements logic around `SparcSubtarget`, `SparcGenSubtargetInfo`, `ReserveRegister`, `InstrInfo`, ...; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `SparcSubtarget`, `SparcGenSubtargetInfo`, `ReserveRegister`, `InstrInfo`, ... 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 65-70
```cpp
SparcSubtarget::~SparcSubtarget() = default;

const SelectionDAGTargetInfo *SparcSubtarget::getSelectionDAGInfo() const {
  return TSInfo.get();
}

```
- **EN**: Implements logic around `~SparcSubtarget`, `getSelectionDAGInfo`, `get`; this block returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `~SparcSubtarget`, `getSelectionDAGInfo`, `get` 实现具体逻辑；这一段返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 71-80
```cpp
void SparcSubtarget::initLibcallLoweringInfo(LibcallLoweringInfo &Info) const {
  if (hasHardQuad())
    return;

  // Setup Runtime library names.
  if (is64Bit() && !useSoftFloat()) {
    Info.setLibcallImpl(RTLIB::ADD_F128, RTLIB::impl__Qp_add);
    Info.setLibcallImpl(RTLIB::SUB_F128, RTLIB::impl__Qp_sub);
    Info.setLibcallImpl(RTLIB::MUL_F128, RTLIB::impl__Qp_mul);
    Info.setLibcallImpl(RTLIB::DIV_F128, RTLIB::impl__Qp_div);
```
- **EN**: Implements logic around `initLibcallLoweringInfo`, `setLibcallImpl`; this block applies conditional target rules.
- **CN**: 围绕 `initLibcallLoweringInfo`, `setLibcallImpl` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 81-90
```cpp
    Info.setLibcallImpl(RTLIB::SQRT_F128, RTLIB::impl__Qp_sqrt);
    Info.setLibcallImpl(RTLIB::FPTOSINT_F128_I32, RTLIB::impl__Qp_qtoi);
    Info.setLibcallImpl(RTLIB::FPTOUINT_F128_I32, RTLIB::impl__Qp_qtoui);
    Info.setLibcallImpl(RTLIB::SINTTOFP_I32_F128, RTLIB::impl__Qp_itoq);
    Info.setLibcallImpl(RTLIB::UINTTOFP_I32_F128, RTLIB::impl__Qp_uitoq);
    Info.setLibcallImpl(RTLIB::FPTOSINT_F128_I64, RTLIB::impl__Qp_qtox);
    Info.setLibcallImpl(RTLIB::FPTOUINT_F128_I64, RTLIB::impl__Qp_qtoux);
    Info.setLibcallImpl(RTLIB::SINTTOFP_I64_F128, RTLIB::impl__Qp_xtoq);
    Info.setLibcallImpl(RTLIB::UINTTOFP_I64_F128, RTLIB::impl__Qp_uxtoq);
    Info.setLibcallImpl(RTLIB::FPEXT_F32_F128, RTLIB::impl__Qp_stoq);
```
- **EN**: Implements logic around `setLibcallImpl`.
- **CN**: 围绕 `setLibcallImpl` 实现具体逻辑。

### Lines 91-100
```cpp
    Info.setLibcallImpl(RTLIB::FPEXT_F64_F128, RTLIB::impl__Qp_dtoq);
    Info.setLibcallImpl(RTLIB::FPROUND_F128_F32, RTLIB::impl__Qp_qtos);
    Info.setLibcallImpl(RTLIB::FPROUND_F128_F64, RTLIB::impl__Qp_qtod);
  } else if (!useSoftFloat()) {
    Info.setLibcallImpl(RTLIB::ADD_F128, RTLIB::impl__Q_add);
    Info.setLibcallImpl(RTLIB::SUB_F128, RTLIB::impl__Q_sub);
    Info.setLibcallImpl(RTLIB::MUL_F128, RTLIB::impl__Q_mul);
    Info.setLibcallImpl(RTLIB::DIV_F128, RTLIB::impl__Q_div);
    Info.setLibcallImpl(RTLIB::SQRT_F128, RTLIB::impl__Q_sqrt);
    Info.setLibcallImpl(RTLIB::FPTOSINT_F128_I32, RTLIB::impl__Q_qtoi);
```
- **EN**: Implements logic around `setLibcallImpl`; this block applies conditional target rules.
- **CN**: 围绕 `setLibcallImpl` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 101-110
```cpp
    Info.setLibcallImpl(RTLIB::FPTOUINT_F128_I32, RTLIB::impl__Q_qtou);
    Info.setLibcallImpl(RTLIB::SINTTOFP_I32_F128, RTLIB::impl__Q_itoq);
    Info.setLibcallImpl(RTLIB::UINTTOFP_I32_F128, RTLIB::impl__Q_utoq);
    Info.setLibcallImpl(RTLIB::FPEXT_F32_F128, RTLIB::impl__Q_stoq);
    Info.setLibcallImpl(RTLIB::FPEXT_F64_F128, RTLIB::impl__Q_dtoq);
    Info.setLibcallImpl(RTLIB::FPROUND_F128_F32, RTLIB::impl__Q_qtos);
    Info.setLibcallImpl(RTLIB::FPROUND_F128_F64, RTLIB::impl__Q_qtod);
  }
}

```
- **EN**: Implements logic around `setLibcallImpl`.
- **CN**: 围绕 `setLibcallImpl` 实现具体逻辑。

### Lines 111-120
```cpp
int SparcSubtarget::getAdjustedFrameSize(int frameSize) const {

  if (is64Bit()) {
    // All 64-bit stack frames must be 16-byte aligned, and must reserve space
    // for spilling the 16 window registers at %sp+BIAS..%sp+BIAS+128.
    frameSize += 128;
    // Frames with calls must also reserve space for 6 outgoing arguments
    // whether they are used or not. LowerCall_64 takes care of that.
    frameSize = alignTo(frameSize, 16);
  } else {
```
- **EN**: Implements logic around `getAdjustedFrameSize`, `alignTo`; this block applies conditional target rules.
- **CN**: 围绕 `getAdjustedFrameSize`, `alignTo` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 121-129
```cpp
    // Emit the correct save instruction based on the number of bytes in
    // the frame. Minimum stack frame size according to V8 ABI is:
    //   16 words for register window spill
    //    1 word for address of returned aggregate-value
    // +  6 words for passing parameters on the stack
    // ----------
    //   23 words * 4 bytes per word = 92 bytes
    frameSize += 92;

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 130-136
```cpp
    // Round up to next doubleword boundary -- a double-word boundary
    // is required by the ABI.
    frameSize = alignTo(frameSize, 8);
  }
  return frameSize;
}

```
- **EN**: Implements logic around `alignTo`; this block returns target-specific results.
- **CN**: 围绕 `alignTo` 实现具体逻辑；这一段返回目标相关结果。

### Lines 137-139
```cpp
bool SparcSubtarget::enableMachineScheduler() const {
  return true;
}
```
- **EN**: Implements logic around `enableMachineScheduler`; this block returns target-specific results.
- **CN**: 围绕 `enableMachineScheduler` 实现具体逻辑；这一段返回目标相关结果。

## Key Concepts / 关键概念

- **Subtarget features / 子目标特性**:
  - **EN**: Tracks CPU variants and feature bits
  - **CN**: 跟踪 CPU 变体与特性位
- **DAG legalization / DAG 合法化**:
  - **EN**: Explains how generic IR-style operations are rewritten for this target
  - **CN**: 说明如何把通用操作改写为该目标可接受的形式

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `SparcSubtarget.h`, `SparcSelectionDAGInfo.h`, `llvm/ADT/StringRef.h`, `llvm/MC/TargetRegistry.h`, `llvm/Support/MathExtras.h`, `SparcGenSubtargetInfo.inc`
- **LLVM subsystems / LLVM 子系统**: MC, Support
- **Generated macros / 生成宏**: `GET_SUBTARGETINFO_CTOR`, `GET_SUBTARGETINFO_TARGET_DESC`
