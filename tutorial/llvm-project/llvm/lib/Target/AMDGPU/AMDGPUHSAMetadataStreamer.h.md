# AMDGPUHSAMetadataStreamer.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/AMDGPUHSAMetadataStreamer.h`
- **Repository**: llvm/llvm-project
- **Purpose**: This header declares the interfaces, data structures, and pass entry points for AMDGPUHSAMetadataStreamer in the LLVM AMDGPU backend. It defines the contracts consumed by other AMDGPU backend components. / 该头文件声明 LLVM AMDGPU 后端中 AMDGPUHSAMetadataStreamer 的接口、数据结构与 Pass 入口。它定义了其他 AMDGPU 后端组件依赖的契约。

## Line-by-Line Analysis / 逐行分析
### Lines 1-23: File banner, includes, and setup
```cpp
//===--- AMDGPUHSAMetadataStreamer.h ----------------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file
/// AMDGPU HSA Metadata Streamer.
///
//
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_AMDGPU_MCTARGETDESC_AMDGPUHSAMETADATASTREAMER_H
#define LLVM_LIB_TARGET_AMDGPU_MCTARGETDESC_AMDGPUHSAMETADATASTREAMER_H

#include "Utils/AMDGPUDelayedMCExpr.h"
#include "llvm/BinaryFormat/MsgPackDocument.h"
#include "llvm/Support/AMDGPUMetadata.h"
#include "llvm/Support/Alignment.h"
#include "llvm/Support/Compiler.h"

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。

### Lines 24-44: Declares class AMDGPUTargetMachine
```cpp
namespace llvm {

class AMDGPUTargetMachine;
class AMDGPUTargetStreamer;
class Argument;
class DataLayout;
class Function;
class MachineFunction;
class MDNode;
class Module;
struct SIProgramInfo;
class Type;

namespace AMDGPU {

namespace IsaInfo {
class AMDGPUTargetID;
}

namespace HSAMD {

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `AMDGPUTargetMachine`, `AMDGPUTargetStreamer`, `Argument`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`AMDGPUTargetMachine`, `AMDGPUTargetStreamer`, `Argument`。

### Lines 45-67: Declares class MetadataStreamer
```cpp
class MetadataStreamer {
public:
  virtual ~MetadataStreamer() = default;

  virtual bool emitTo(AMDGPUTargetStreamer &TargetStreamer) = 0;

  virtual void begin(const Module &Mod,
                     const IsaInfo::AMDGPUTargetID &TargetID) = 0;

  virtual void end() = 0;

  virtual void emitKernel(const MachineFunction &MF,
                          const SIProgramInfo &ProgramInfo) = 0;

protected:
  virtual void emitVersion() = 0;
  virtual void emitHiddenKernelArgs(const MachineFunction &MF, unsigned &Offset,
                                    msgpack::ArrayDocNode Args) = 0;
  virtual void emitKernelAttrs(const AMDGPUTargetMachine &TM,
                               const MachineFunction &MF,
                               msgpack::MapDocNode Kern) = 0;
};

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `MetadataStreamer`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`MetadataStreamer`。

### Lines 68-90: Declares class LLVM_EXTERNAL_VISIBILITY
```cpp
class LLVM_EXTERNAL_VISIBILITY MetadataStreamerMsgPackV4
    : public MetadataStreamer {
protected:
  std::unique_ptr<DelayedMCExprs> DelayedExprs =
      std::make_unique<DelayedMCExprs>();

  std::unique_ptr<msgpack::Document> HSAMetadataDoc =
      std::make_unique<msgpack::Document>();

  void dump(StringRef HSAMetadataString) const;

  void verify(StringRef HSAMetadataString) const;

  std::optional<StringRef> getAccessQualifier(StringRef AccQual) const;

  std::optional<StringRef>
  getAddressSpaceQualifier(unsigned AddressSpace) const;

  StringRef getValueKind(Type *Ty, StringRef TypeQual,
                         StringRef BaseTypeName) const;

  std::string getTypeName(Type *Ty, bool Signed) const;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `LLVM_EXTERNAL_VISIBILITY`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`LLVM_EXTERNAL_VISIBILITY`。

### Lines 91-112: Declares getWorkGroupDimensions
```cpp
  msgpack::ArrayDocNode getWorkGroupDimensions(MDNode *Node) const;

  msgpack::MapDocNode getHSAKernelProps(const MachineFunction &MF,
                                        const SIProgramInfo &ProgramInfo,
                                        unsigned CodeObjectVersion) const;

  void emitVersion() override;

  void emitTargetID(const IsaInfo::AMDGPUTargetID &TargetID);

  void emitPrintf(const Module &Mod);

  void emitKernelLanguage(const Function &Func, msgpack::MapDocNode Kern);

  void emitKernelAttrs(const AMDGPUTargetMachine &TM, const MachineFunction &MF,
                       msgpack::MapDocNode Kern) override;

  void emitKernelArgs(const MachineFunction &MF, msgpack::MapDocNode Kern);

  void emitKernelArg(const Argument &Arg, unsigned &Offset,
                     msgpack::ArrayDocNode Args);

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 113-135: Defines emitKernelArg
```cpp
  void emitKernelArg(const DataLayout &DL, Type *Ty, Align Alignment,
                     StringRef ValueKind, unsigned &Offset,
                     msgpack::ArrayDocNode Args,
                     MaybeAlign PointeeAlign = std::nullopt,
                     StringRef Name = "", StringRef TypeName = "",
                     StringRef BaseTypeName = "", StringRef ActAccQual = "",
                     StringRef AccQual = "", StringRef TypeQual = "");

  void emitHiddenKernelArgs(const MachineFunction &MF, unsigned &Offset,
                            msgpack::ArrayDocNode Args) override;

  msgpack::DocNode &getRootMetadata(StringRef Key) {
    return HSAMetadataDoc->getRoot().getMap(/*Convert=*/true)[Key];
  }

  msgpack::DocNode &getHSAMetadataRoot() {
    return HSAMetadataDoc->getRoot();
  }

public:
  MetadataStreamerMsgPackV4() = default;
  ~MetadataStreamerMsgPackV4() override = default;

```
**EN:** This section contains concrete logic for emitKernelArg. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 emitKernelArg 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 136-159: Declares class MetadataStreamerMsgPackV5
```cpp
  bool emitTo(AMDGPUTargetStreamer &TargetStreamer) override;

  void begin(const Module &Mod,
             const IsaInfo::AMDGPUTargetID &TargetID) override;

  void end() override;

  void emitKernel(const MachineFunction &MF,
                  const SIProgramInfo &ProgramInfo) override;
};

class MetadataStreamerMsgPackV5 : public MetadataStreamerMsgPackV4 {
protected:
  void emitVersion() override;
  void emitHiddenKernelArgs(const MachineFunction &MF, unsigned &Offset,
                            msgpack::ArrayDocNode Args) override;
  void emitKernelAttrs(const AMDGPUTargetMachine &TM, const MachineFunction &MF,
                       msgpack::MapDocNode Kern) override;

public:
  MetadataStreamerMsgPackV5() = default;
  ~MetadataStreamerMsgPackV5() override = default;
};

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `MetadataStreamerMsgPackV5`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`MetadataStreamerMsgPackV5`。

### Lines 160-176: Preprocessor guards and macros
```cpp
class MetadataStreamerMsgPackV6 final : public MetadataStreamerMsgPackV5 {
protected:
  void emitVersion() override;

public:
  MetadataStreamerMsgPackV6() = default;
  ~MetadataStreamerMsgPackV6() override = default;

  void emitKernelAttrs(const AMDGPUTargetMachine &TM, const MachineFunction &MF,
                       msgpack::MapDocNode Kern) override;
};

} // end namespace HSAMD
} // end namespace AMDGPU
} // end namespace llvm

#endif // LLVM_LIB_TARGET_AMDGPU_MCTARGETDESC_AMDGPUHSAMETADATASTREAMER_H
```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled. Main symbols: `MetadataStreamerMsgPackV6`.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。 主要符号：`MetadataStreamerMsgPackV6`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ header
- **Primary symbols / 主要符号**: `AMDGPUTargetMachine`, `AMDGPUTargetStreamer`, `Argument`, `DataLayout`, `Function`, `MachineFunction`
- **Main themes / 核心主题**: machine-function state / MachineFunction 状态
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"Utils/AMDGPUDelayedMCExpr.h"`
- `"llvm/BinaryFormat/MsgPackDocument.h"`
- `"llvm/Support/AMDGPUMetadata.h"`
- `"llvm/Support/Alignment.h"`
- `"llvm/Support/Compiler.h"`
