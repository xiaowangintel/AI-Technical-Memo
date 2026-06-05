# AMDGPUAsmUtils.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/Utils/AMDGPUAsmUtils.h`
- **Repository**: llvm/llvm-project
- **Purpose**: This header declares the interfaces, data structures, and pass entry points for AMDGPUAsmUtils in the LLVM backend utilities. It defines the contracts consumed by other AMDGPU backend components. / 该头文件声明 LLVM 后端工具中 AMDGPUAsmUtils 的接口、数据结构与 Pass 入口。它定义了其他 AMDGPU 后端组件依赖的契约。

## Line-by-Line Analysis / 逐行分析
### Lines 1-22: File banner, includes, and setup
```cpp
//===-- AMDGPUAsmUtils.h - AsmParser/InstPrinter common ---------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_AMDGPU_UTILS_AMDGPUASMUTILS_H
#define LLVM_LIB_TARGET_AMDGPU_UTILS_AMDGPUASMUTILS_H

#include "SIDefines.h"

#include "llvm/ADT/StringRef.h"

namespace llvm {

class StringLiteral;
class MCSubtargetInfo;

namespace AMDGPU {

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation. Main symbols: `StringLiteral`, `MCSubtargetInfo`.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。 主要符号：`StringLiteral`, `MCSubtargetInfo`。

### Lines 23-46: Declares struct CustomOperand
```cpp
const int OPR_ID_UNKNOWN = -1;
const int OPR_ID_UNSUPPORTED = -2;
const int OPR_ID_DUPLICATE = -3;
const int OPR_VAL_INVALID = -4;

struct CustomOperand {
  StringLiteral Name;
  unsigned Encoding = 0;
  bool (*Cond)(const MCSubtargetInfo &STI) = nullptr;
};

struct CustomOperandVal {
  StringLiteral Name;
  unsigned Max;
  unsigned Default;
  unsigned Shift;
  unsigned Width;
  bool (*Cond)(const MCSubtargetInfo &STI) = nullptr;
  unsigned Mask = (1 << Width) - 1;

  unsigned decode(unsigned Code) const { return (Code >> Shift) & Mask; }

  unsigned encode(unsigned Val) const { return (Val & Mask) << Shift; }

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `CustomOperand`, `CustomOperandVal`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`CustomOperand`, `CustomOperandVal`。

### Lines 47-70: Defines getMask
```cpp
  unsigned getMask() const { return Mask << Shift; }

  bool isValid(unsigned Val) const { return Val <= Max; }

  bool isSupported(const MCSubtargetInfo &STI) const {
    return !Cond || Cond(STI);
  }
};

namespace DepCtr {

extern const CustomOperandVal DepCtrInfo[];
extern const int DEP_CTR_SIZE;

} // namespace DepCtr

// Symbolic names for the sendmsg(msg_id, operation, stream) syntax.
namespace SendMsg {

/// Map from a symbolic name for a msg_id to the message portion of the
/// immediate encoding. A negative return value indicates that the Name was
/// unknown or unsupported on this target.
int64_t getMsgId(StringRef Name, const MCSubtargetInfo &STI);

```
**EN:** This section contains concrete logic for getMask. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 getMask 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 71-93: Declares getMsgName
```cpp
/// Map from an encoding to the symbolic name for a msg_id immediate. This is
/// doing opposite of getMsgId().
StringRef getMsgName(uint64_t Encoding, const MCSubtargetInfo &STI);

/// Map from a symbolic name for a sendmsg operation to the operation portion of
/// the immediate encoding. A negative return value indicates that the Name was
/// unknown or unsupported on this target.
int64_t getMsgOpId(int64_t MsgId, StringRef Name, const MCSubtargetInfo &STI);

/// Map from an encoding to the symbolic name for a sendmsg operation. This is
/// doing opposite of getMsgOpId().
StringRef getMsgOpName(int64_t MsgId, uint64_t Encoding,
                       const MCSubtargetInfo &STI);

} // namespace SendMsg

namespace WaitEvent {
int64_t getWaitEventMask(StringRef Name, const MCSubtargetInfo &STI);
StringRef getWaitEventMaskName(uint64_t Encoding, const MCSubtargetInfo &STI);
} // namespace WaitEvent

namespace Hwreg { // Symbolic names for the hwreg(...) syntax.

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 94-117: Declares getHwregId
```cpp
int64_t getHwregId(StringRef Name, const MCSubtargetInfo &STI);
StringRef getHwreg(uint64_t Encoding, const MCSubtargetInfo &STI);

} // namespace Hwreg

namespace MTBUFFormat {

extern StringLiteral const DfmtSymbolic[];
extern StringLiteral const NfmtSymbolicGFX10[];
extern StringLiteral const NfmtSymbolicSICI[];
extern StringLiteral const NfmtSymbolicVI[];
extern StringLiteral const UfmtSymbolicGFX10[];
extern StringLiteral const UfmtSymbolicGFX11[];
extern unsigned const DfmtNfmt2UFmtGFX10[];
extern unsigned const DfmtNfmt2UFmtGFX11[];

} // namespace MTBUFFormat

namespace Swizzle { // Symbolic names for the swizzle(...) syntax.

extern const char* const IdSymbolic[];

} // namespace Swizzle

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 118-141: Declares struct GFXVersion
```cpp
namespace VGPRIndexMode { // Symbolic names for the gpr_idx(...) syntax.

extern const char* const IdSymbolic[];

} // namespace VGPRIndexMode

namespace UCVersion {

struct GFXVersion {
  StringLiteral Symbol;
  unsigned Code;
};

ArrayRef<GFXVersion> getGFXVersions();

} // namespace UCVersion

namespace WMMAMods {
// These should match enum values in SIDefines.h

constexpr const char *const ModMatrixFmt[] = {
    "MATRIX_FMT_FP8", "MATRIX_FMT_BF8", "MATRIX_FMT_FP6", "MATRIX_FMT_BF6",
    "MATRIX_FMT_FP4"};

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `GFXVersion`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`GFXVersion`。

### Lines 142-152: Preprocessor guards and macros
```cpp
constexpr const char *const ModMatrixScale[] = {"MATRIX_SCALE_ROW0",
                                                "MATRIX_SCALE_ROW1"};

constexpr const char *const ModMatrixScaleFmt[] = {
    "MATRIX_SCALE_FMT_E8", "MATRIX_SCALE_FMT_E5M3", "MATRIX_SCALE_FMT_E4M3"};
} // namespace WMMAMods

} // namespace AMDGPU
} // namespace llvm

#endif
```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。

## Key Concepts / 关键概念
- **Language / 语言**: C++ header
- **Primary symbols / 主要符号**: `StringLiteral`, `MCSubtargetInfo`, `CustomOperand`, `CustomOperandVal`, `GFXVersion`
- **Main themes / 核心主题**: subtarget modeling / 子目标建模; assembly handling / 汇编处理
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"SIDefines.h"`
- `"llvm/ADT/StringRef.h"`
