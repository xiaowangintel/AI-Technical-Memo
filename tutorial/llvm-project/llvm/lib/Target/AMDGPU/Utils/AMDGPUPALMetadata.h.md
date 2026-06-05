# AMDGPUPALMetadata.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/Utils/AMDGPUPALMetadata.h`
- **Repository**: llvm/llvm-project
- **Purpose**: This header declares the interfaces, data structures, and pass entry points for AMDGPUPALMetadata in the LLVM backend utilities. It defines the contracts consumed by other AMDGPU backend components. / 该头文件声明 LLVM 后端工具中 AMDGPUPALMetadata 的接口、数据结构与 Pass 入口。它定义了其他 AMDGPU 后端组件依赖的契约。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28: File banner, includes, and setup
```cpp
//===-- AMDGPUPALMetadata.h - PAL metadata handling -------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file
/// PAL metadata handling
//
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_AMDGPU_AMDGPUPALMETADATA_H
#define LLVM_LIB_TARGET_AMDGPU_AMDGPUPALMETADATA_H
#include "AMDGPUDelayedMCExpr.h"
#include "llvm/ADT/DenseMap.h"
#include "llvm/BinaryFormat/MsgPackDocument.h"
#include "llvm/MC/MCContext.h"

namespace llvm {

class Module;

class AMDGPUPALMetadata {
public:
  using RegisterExprMap = DenseMap<unsigned, const MCExpr *>;

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation. Main symbols: `Module`, `AMDGPUPALMetadata`.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。 主要符号：`Module`, `AMDGPUPALMetadata`。

### Lines 29-59: Implementation details and local logic
```cpp
private:
  unsigned BlobType = 0;
  msgpack::Document MsgPackDoc;
  msgpack::DocNode Registers;
  msgpack::DocNode HwStages;
  msgpack::DocNode ShaderFunctions;
  bool VersionChecked = false;
  msgpack::DocNode Version;
  // From PAL version >= 3.0
  msgpack::DocNode ComputeRegisters;
  msgpack::DocNode GraphicsRegisters;

  DelayedMCExprs DelayedExprs;
  RegisterExprMap REM;
  bool ResolvedAll = true;

public:
  // Read the amdgpu.pal.metadata supplied by the frontend, ready for
  // per-function modification.
  void readFromIR(Module &M);

  // Set PAL metadata from a binary blob from the applicable .note record.
  // Returns false if bad format.  Blob must remain valid for the lifetime of
  // the Metadata.
  bool setFromBlob(unsigned Type, StringRef Blob);

  // Set the rsrc1 register in the metadata for a particular shader stage.
  // In fact this ORs the value into any previous setting of the register.
  void setRsrc1(unsigned CC, unsigned Val);
  void setRsrc1(unsigned CC, const MCExpr *Val, MCContext &Ctx);

```
**EN:** This section advances the file implementation by combining declarations, control flow, and helper operations that support the AMDGPU backend.
**CN:** 本节通过组合声明、控制流与辅助操作来推进文件实现，支撑 AMDGPU 后端的整体功能。

### Lines 60-89: Declares setRsrc2
```cpp
  // Set the rsrc2 register in the metadata for a particular shader stage.
  // In fact this ORs the value into any previous setting of the register.
  void setRsrc2(unsigned CC, unsigned Val);
  void setRsrc2(unsigned CC, const MCExpr *Val, MCContext &Ctx);

  // Set the SPI_PS_INPUT_ENA register in the metadata.
  // In fact this ORs the value into any previous setting of the register.
  void setSpiPsInputEna(unsigned Val);

  // Set the SPI_PS_INPUT_ADDR register in the metadata.
  // In fact this ORs the value into any previous setting of the register.
  void setSpiPsInputAddr(unsigned Val);

  // Get a register from the metadata, or 0 if not currently set.
  unsigned getRegister(unsigned Reg);

  // Set a register in the metadata.
  // In fact this ORs the value into any previous setting of the register.
  void setRegister(unsigned Reg, unsigned Val);
  void setRegister(unsigned Reg, const MCExpr *Val, MCContext &Ctx);

  // Set the entry point name for one shader.
  void setEntryPoint(unsigned CC, StringRef Name);

  // Set the number of used vgprs in the metadata. This is an optional advisory
  // record for logging etc; wave dispatch actually uses the rsrc1 register for
  // the shader stage to determine the number of vgprs to allocate.
  void setNumUsedVgprs(unsigned CC, unsigned Val);
  void setNumUsedVgprs(unsigned CC, const MCExpr *Val, MCContext &Ctx);

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 90-118: Declares setNumUsedAgprs
```cpp
  // Set the number of used agprs in the metadata. This is an optional advisory
  // record for logging etc;
  void setNumUsedAgprs(unsigned CC, unsigned Val);
  void setNumUsedAgprs(unsigned CC, const MCExpr *Val);

  // Set the number of used sgprs in the metadata. This is an optional advisory
  // record for logging etc; wave dispatch actually uses the rsrc1 register for
  // the shader stage to determine the number of sgprs to allocate.
  void setNumUsedSgprs(unsigned CC, unsigned Val);
  void setNumUsedSgprs(unsigned CC, const MCExpr *Val, MCContext &Ctx);

  // Set the scratch size in the metadata.
  void setScratchSize(unsigned CC, unsigned Val);
  void setScratchSize(unsigned CC, const MCExpr *Val, MCContext &Ctx);

  // Set the stack frame size of a function in the metadata.
  void setFunctionScratchSize(StringRef FnName, unsigned Val);

  // Set the amount of LDS used in bytes in the metadata. This is an optional
  // advisory record for logging etc; wave dispatch actually uses the rsrc1
  // register for the shader stage to determine the amount of LDS to allocate.
  void setFunctionLdsSize(StringRef FnName, unsigned Val);

  // Set the number of used vgprs in the metadata. This is an optional advisory
  // record for logging etc; wave dispatch actually uses the rsrc1 register for
  // the shader stage to determine the number of vgprs to allocate.
  void setFunctionNumUsedVgprs(StringRef FnName, unsigned Val);
  void setFunctionNumUsedVgprs(StringRef FnName, const MCExpr *Val);

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 119-151: Declares setFunctionNumUsedSgprs
```cpp
  // Set the number of used sgprs in the metadata. This is an optional advisory
  // record for logging etc; wave dispatch actually uses the rsrc1 register for
  // the shader stage to determine the number of sgprs to allocate.
  void setFunctionNumUsedSgprs(StringRef FnName, unsigned Val);
  void setFunctionNumUsedSgprs(StringRef FnName, const MCExpr *Val);

  // Set the hardware register bit in PAL metadata to enable wave32 on the
  // shader of the given calling convention.
  void setWave32(unsigned CC);

  // Emit the accumulated PAL metadata as asm directives.
  // This is called from AMDGPUTargetAsmStreamer::Finish().
  void toString(std::string &S);

  // Set PAL metadata from YAML text.
  bool setFromString(StringRef S);

  // Get .note record vendor name of metadata blob to be emitted.
  const char *getVendor() const;

  // Get .note record type of metadata blob to be emitted:
  // ELF::NT_AMD_PAL_METADATA (legacy key=val format), or
  // ELF::NT_AMDGPU_METADATA (MsgPack format), or
  // 0 (no PAL metadata).
  unsigned getType() const;

  // Emit the accumulated PAL metadata as a binary blob.
  // This is called from AMDGPUTargetELFStreamer::Finish().
  void toBlob(unsigned Type, std::string &S);

  // Get the msgpack::Document for the PAL metadata.
  msgpack::Document *getMsgPackDoc() { return &MsgPackDoc; }

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `AMDGPUTargetAsmStreamer::Finish`, `ELF::NT_AMD_PAL_METADATA`, `ELF::NT_AMDGPU_METADATA`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`AMDGPUTargetAsmStreamer::Finish`, `ELF::NT_AMD_PAL_METADATA`, `ELF::NT_AMDGPU_METADATA`。

### Lines 152-183: Declares setLegacy
```cpp
  // Set legacy PAL metadata format.
  void setLegacy();

  unsigned getPALMajorVersion();
  unsigned getPALMinorVersion();
  VersionTuple getPALVersion();

  void updateHwStageMaximum(unsigned CC, StringRef field, unsigned Val);
  void setHwStage(unsigned CC, StringRef field, unsigned Val);
  void setHwStage(unsigned CC, StringRef field, bool Val);
  void setHwStage(unsigned CC, StringRef field, msgpack::Type Type,
                  const MCExpr *Val);

  void setComputeRegisters(StringRef field, unsigned Val);
  void setComputeRegisters(StringRef field, bool Val);

  // If the field does not exist will return nullptr rather than creating a new
  // entry (which is the behaviour of the other functions).
  msgpack::DocNode *refComputeRegister(StringRef field);
  bool checkComputeRegisters(StringRef field, unsigned Val);
  bool checkComputeRegisters(StringRef field, bool Val);

  void setGraphicsRegisters(StringRef field, unsigned Val);
  void setGraphicsRegisters(StringRef field, bool Val);
  void setGraphicsRegisters(StringRef field1, StringRef field2, unsigned Val);
  void setGraphicsRegisters(StringRef field1, StringRef field2, bool Val);

  // Erase all PAL metadata.
  void reset();

  bool resolvedAllMCExpr();

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 184-217: Declares isLegacy
```cpp
private:
  // Return whether the blob type is legacy PAL metadata.
  bool isLegacy() const;

  // Reference (create if necessary) the node for the registers map.
  msgpack::DocNode &refRegisters();

  // Get (create if necessary) the registers map.
  msgpack::MapDocNode getRegisters();

  // Reference (create if necessary) the node for the shader functions map.
  msgpack::DocNode &refShaderFunctions();

  // Get (create if necessary) the shader functions map.
  msgpack::MapDocNode getShaderFunctions();

  // Get (create if necessary) a function in the shader functions map.
  msgpack::MapDocNode getShaderFunction(StringRef Name);

  // Reference (create if necessary) the node for the compute_registers map.
  msgpack::DocNode &refComputeRegisters();

  // Get (create if necessary) the .compute_registers entry.
  msgpack::MapDocNode getComputeRegisters();

  // Reference (create if necessary) the node for the graphics registers map.
  msgpack::DocNode &refGraphicsRegisters();

  // Get (create if necessary) the .graphics_registers entry.
  msgpack::MapDocNode getGraphicsRegisters();

  // Reference (create if necessary) the node for the hardware_stages map.
  msgpack::DocNode &refHwStage();

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 218-234: Preprocessor guards and macros
```cpp
  // Get (create if necessary) the .hardware_stages entry for the given calling
  // convention.
  msgpack::MapDocNode getHwStage(unsigned CC);

  // Get the PAL version major (idx 0) or minor (idx 1). This is an internal
  // helper for the public wrapper functions that request Major or Minor
  unsigned getPALVersion(unsigned idx);

  bool setFromLegacyBlob(StringRef Blob);
  bool setFromMsgPackBlob(StringRef Blob);
  void toLegacyBlob(std::string &Blob);
  void toMsgPackBlob(std::string &Blob);
};

} // end namespace llvm

#endif // LLVM_LIB_TARGET_AMDGPU_AMDGPUPALMETADATA_H
```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。

## Key Concepts / 关键概念
- **Language / 语言**: C++ header
- **Primary symbols / 主要符号**: `Module`, `AMDGPUPALMetadata`, `AMDGPUTargetAsmStreamer::Finish`, `ELF::NT_AMD_PAL_METADATA`, `ELF::NT_AMDGPU_METADATA`, `AMDGPUTargetELFStreamer::Finish`
- **Main themes / 核心主题**: register management / 寄存器管理; SelectionDAG processing / SelectionDAG 处理
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"AMDGPUDelayedMCExpr.h"`
- `"llvm/ADT/DenseMap.h"`
- `"llvm/BinaryFormat/MsgPackDocument.h"`
- `"llvm/MC/MCContext.h"`
