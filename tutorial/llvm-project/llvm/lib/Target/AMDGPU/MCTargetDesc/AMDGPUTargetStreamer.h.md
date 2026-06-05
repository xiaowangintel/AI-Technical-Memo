# AMDGPUTargetStreamer.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/MCTargetDesc/AMDGPUTargetStreamer.h`
- **Repository**: llvm/llvm-project
- **Purpose**: This header declares the interfaces, data structures, and pass entry points for AMDGPUTargetStreamer in the LLVM MC target description layer. It defines the contracts consumed by other AMDGPU backend components. / 该头文件声明 LLVM MC 目标描述层中 AMDGPUTargetStreamer 的接口、数据结构与 Pass 入口。它定义了其他 AMDGPU 后端组件依赖的契约。

## Line-by-Line Analysis / 逐行分析
### Lines 1-32: File banner, includes, and setup
```cpp
//===-- AMDGPUTargetStreamer.h - AMDGPU Target Streamer --------*- C++ -*--===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_AMDGPU_MCTARGETDESC_AMDGPUTARGETSTREAMER_H
#define LLVM_LIB_TARGET_AMDGPU_MCTARGETDESC_AMDGPUTARGETSTREAMER_H

#include "Utils/AMDGPUBaseInfo.h"
#include "Utils/AMDGPUPALMetadata.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/MC/MCStreamer.h"
#include <string>
#include <utility>

namespace llvm {

class MCELFStreamer;
class MCSymbol;
class formatted_raw_ostream;

namespace AMDGPU {

struct AMDGPUMCKernelCodeT;
struct MCKernelDescriptor;
namespace HSAMD {
struct Metadata;
}

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation. Main symbols: `MCELFStreamer`, `MCSymbol`, `formatted_raw_ostream`.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。 主要符号：`MCELFStreamer`, `MCSymbol`, `formatted_raw_ostream`。

### Lines 33-64: Declares struct FuncInfo
```cpp
struct FuncInfo {
  uint32_t NumSGPR = 0;
  uint32_t NumArchVGPR = 0;
  uint32_t NumAccVGPR = 0;
  uint32_t PrivateSegmentSize = 0;
  bool UsesVCC = false;
  bool UsesFlatScratch = false;
  bool HasDynStack = false;

  MCSymbol *Sym = nullptr;
};

struct InfoSectionData {
  SmallVector<FuncInfo, 8> Funcs;
  SmallVector<std::pair<MCSymbol *, MCSymbol *>, 4> Uses;
  SmallVector<std::pair<MCSymbol *, MCSymbol *>, 8> Calls;
  SmallVector<std::pair<MCSymbol *, std::string>, 4> IndirectCalls;
  SmallVector<std::pair<MCSymbol *, std::string>, 4> TypeIds;
};

} // namespace AMDGPU

class AMDGPUTargetStreamer : public MCTargetStreamer {
  AMDGPUPALMetadata PALMetadata;

protected:
  // TODO: Move HSAMetadataStream to AMDGPUTargetStreamer.
  std::optional<AMDGPU::IsaInfo::AMDGPUTargetID> TargetID;
  unsigned CodeObjectVersion;

  MCContext &getContext() const { return Streamer.getContext(); }

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `FuncInfo`, `InfoSectionData`, `AMDGPUTargetStreamer`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`FuncInfo`, `InfoSectionData`, `AMDGPUTargetStreamer`。

### Lines 65-98: Defines AMDGPUTargetStreamer
```cpp
public:
  AMDGPUTargetStreamer(MCStreamer &S)
      : MCTargetStreamer(S),
        // Assume the default COV for now, EmitDirectiveAMDHSACodeObjectVersion
        // will update this if it is encountered.
        CodeObjectVersion(AMDGPU::getDefaultAMDHSACodeObjectVersion()) {}

  AMDGPUPALMetadata *getPALMetadata() { return &PALMetadata; }

  virtual void EmitDirectiveAMDGCNTarget(){};

  virtual void EmitDirectiveAMDHSACodeObjectVersion(unsigned COV) {
    CodeObjectVersion = COV;
  }

  virtual void EmitAMDKernelCodeT(AMDGPU::AMDGPUMCKernelCodeT &Header) {};

  virtual void EmitAMDGPUSymbolType(StringRef SymbolName, unsigned Type){};

  virtual void emitAMDGPULDS(MCSymbol *Symbol, unsigned Size, Align Alignment) {
  }

  virtual void EmitMCResourceInfo(
      const MCSymbol *NumVGPR, const MCSymbol *NumAGPR,
      const MCSymbol *NumExplicitSGPR, const MCSymbol *NumNamedBarrier,
      const MCSymbol *PrivateSegmentSize, const MCSymbol *UsesVCC,
      const MCSymbol *UsesFlatScratch, const MCSymbol *HasDynamicallySizedStack,
      const MCSymbol *HasRecursion, const MCSymbol *HasIndirectCall) {};

  virtual void EmitMCResourceMaximums(const MCSymbol *MaxVGPR,
                                      const MCSymbol *MaxAGPR,
                                      const MCSymbol *MaxSGPR,
                                      const MCSymbol *MaxNamedBarrier) {};

```
**EN:** This section contains concrete logic for AMDGPUTargetStreamer. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPU::getDefaultAMDHSACodeObjectVersion`.
**CN:** 本节包含与 AMDGPUTargetStreamer 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPU::getDefaultAMDHSACodeObjectVersion`。

### Lines 99-132: Defines EmitISAVersion
```cpp
  /// \returns True on success, false on failure.
  virtual bool EmitISAVersion() { return true; }

  /// \returns True on success, false on failure.
  virtual bool EmitHSAMetadataV3(StringRef HSAMetadataString);

  /// Emit HSA Metadata
  ///
  /// When \p Strict is true, known metadata elements must already be
  /// well-typed. When \p Strict is false, known types are inferred and
  /// the \p HSAMetadata structure is updated with the correct types.
  ///
  /// \returns True on success, false on failure.
  virtual bool EmitHSAMetadata(msgpack::Document &HSAMetadata, bool Strict) {
    return true;
  }

  /// \returns True on success, false on failure.
  virtual bool EmitHSAMetadata(const AMDGPU::HSAMD::Metadata &HSAMetadata) {
    return true;
  }

  /// \returns True on success, false on failure.
  virtual bool EmitCodeEnd(const MCSubtargetInfo &STI) { return true; }

  virtual void
  EmitAmdhsaKernelDescriptor(const MCSubtargetInfo &STI, StringRef KernelName,
                             const AMDGPU::MCKernelDescriptor &KernelDescriptor,
                             const MCExpr *NextVGPR, const MCExpr *NextSGPR,
                             const MCExpr *ReserveVCC,
                             const MCExpr *ReserveFlatScr) {}

  virtual void emitAMDGPUInfo(const AMDGPU::InfoSectionData &Data) {}

```
**EN:** This section contains concrete logic for EmitISAVersion. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 EmitISAVersion 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 133-166: Declares class AMDGPUTargetAsmStreamer
```cpp
  static StringRef getArchNameFromElfMach(unsigned ElfMach);
  static unsigned getElfMach(StringRef GPU);

  const std::optional<AMDGPU::IsaInfo::AMDGPUTargetID> &getTargetID() const {
    return TargetID;
  }
  std::optional<AMDGPU::IsaInfo::AMDGPUTargetID> &getTargetID() {
    return TargetID;
  }
  void initializeTargetID(const MCSubtargetInfo &STI) {
    assert(TargetID == std::nullopt && "TargetID can only be initialized once");
    TargetID.emplace(STI);
  }
  void initializeTargetID(const MCSubtargetInfo &STI, StringRef FeatureString) {
    initializeTargetID(STI);

    assert(getTargetID() != std::nullopt && "TargetID is None");
    getTargetID()->setTargetIDFromFeaturesString(FeatureString);
  }
};

class AMDGPUTargetAsmStreamer final : public AMDGPUTargetStreamer {
  formatted_raw_ostream &OS;
public:
  AMDGPUTargetAsmStreamer(MCStreamer &S, formatted_raw_ostream &OS);

  void finish() override;

  void EmitDirectiveAMDGCNTarget() override;

  void EmitDirectiveAMDHSACodeObjectVersion(unsigned COV) override;

  void EmitAMDKernelCodeT(AMDGPU::AMDGPUMCKernelCodeT &Header) override;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `AMDGPUTargetAsmStreamer`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`AMDGPUTargetAsmStreamer`。

### Lines 167-200: Declares EmitAMDGPUSymbolType
```cpp
  void EmitAMDGPUSymbolType(StringRef SymbolName, unsigned Type) override;

  void emitAMDGPULDS(MCSymbol *Sym, unsigned Size, Align Alignment) override;

  void EmitMCResourceInfo(
      const MCSymbol *NumVGPR, const MCSymbol *NumAGPR,
      const MCSymbol *NumExplicitSGPR, const MCSymbol *NumNamedBarrier,
      const MCSymbol *PrivateSegmentSize, const MCSymbol *UsesVCC,
      const MCSymbol *UsesFlatScratch, const MCSymbol *HasDynamicallySizedStack,
      const MCSymbol *HasRecursion, const MCSymbol *HasIndirectCall) override;

  void EmitMCResourceMaximums(const MCSymbol *MaxVGPR, const MCSymbol *MaxAGPR,
                              const MCSymbol *MaxSGPR,
                              const MCSymbol *MaxNamedBarrier) override;

  /// \returns True on success, false on failure.
  bool EmitISAVersion() override;

  /// \returns True on success, false on failure.
  bool EmitHSAMetadata(msgpack::Document &HSAMetadata, bool Strict) override;

  /// \returns True on success, false on failure.
  bool EmitCodeEnd(const MCSubtargetInfo &STI) override;

  void
  EmitAmdhsaKernelDescriptor(const MCSubtargetInfo &STI, StringRef KernelName,
                             const AMDGPU::MCKernelDescriptor &KernelDescriptor,
                             const MCExpr *NextVGPR, const MCExpr *NextSGPR,
                             const MCExpr *ReserveVCC,
                             const MCExpr *ReserveFlatScr) override;

  void emitAMDGPUInfo(const AMDGPU::InfoSectionData &Data) override;
};

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 201-234: Declares class AMDGPUTargetELFStreamer
```cpp
class AMDGPUTargetELFStreamer final : public AMDGPUTargetStreamer {
  const MCSubtargetInfo &STI;
  MCStreamer &Streamer;

  void EmitNote(StringRef Name, const MCExpr *DescSize, unsigned NoteType,
                function_ref<void(MCELFStreamer &)> EmitDesc);

  unsigned getEFlags();

  unsigned getEFlagsR600();
  unsigned getEFlagsAMDGCN();

  unsigned getEFlagsUnknownOS();
  unsigned getEFlagsAMDHSA();
  unsigned getEFlagsAMDPAL();
  unsigned getEFlagsMesa3D();

  unsigned getEFlagsV3();
  unsigned getEFlagsV4();
  unsigned getEFlagsV6();

public:
  AMDGPUTargetELFStreamer(MCStreamer &S, const MCSubtargetInfo &STI);

  MCELFStreamer &getStreamer();

  void finish() override;

  void EmitDirectiveAMDGCNTarget() override;

  void EmitAMDKernelCodeT(AMDGPU::AMDGPUMCKernelCodeT &Header) override;

  void EmitAMDGPUSymbolType(StringRef SymbolName, unsigned Type) override;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `AMDGPUTargetELFStreamer`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`AMDGPUTargetELFStreamer`。

### Lines 235-256: Preprocessor guards and macros
```cpp
  void emitAMDGPULDS(MCSymbol *Sym, unsigned Size, Align Alignment) override;

  /// \returns True on success, false on failure.
  bool EmitISAVersion() override;

  /// \returns True on success, false on failure.
  bool EmitHSAMetadata(msgpack::Document &HSAMetadata, bool Strict) override;

  /// \returns True on success, false on failure.
  bool EmitCodeEnd(const MCSubtargetInfo &STI) override;

  void
  EmitAmdhsaKernelDescriptor(const MCSubtargetInfo &STI, StringRef KernelName,
                             const AMDGPU::MCKernelDescriptor &KernelDescriptor,
                             const MCExpr *NextVGPR, const MCExpr *NextSGPR,
                             const MCExpr *ReserveVCC,
                             const MCExpr *ReserveFlatScr) override;

  void emitAMDGPUInfo(const AMDGPU::InfoSectionData &Data) override;
};
}
#endif
```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。

## Key Concepts / 关键概念
- **Language / 语言**: C++ header
- **Primary symbols / 主要符号**: `MCELFStreamer`, `MCSymbol`, `formatted_raw_ostream`, `AMDGPUMCKernelCodeT`, `MCKernelDescriptor`, `Metadata`
- **Main themes / 核心主题**: subtarget modeling / 子目标建模; hardware resource modeling / 硬件资源建模
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"Utils/AMDGPUBaseInfo.h"`
- `"Utils/AMDGPUPALMetadata.h"`
- `"llvm/ADT/SmallVector.h"`
- `"llvm/MC/MCStreamer.h"`
- `<string>`
- `<utility>`
