# AMDGPUDisassembler.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/Disassembler/AMDGPUDisassembler.h`
- **Repository**: llvm/llvm-project
- **Purpose**: This header declares the interfaces, data structures, and pass entry points for AMDGPUDisassembler in the LLVM disassembler. It defines the contracts consumed by other AMDGPU backend components. / 该头文件声明 LLVM 反汇编器中 AMDGPUDisassembler 的接口、数据结构与 Pass 入口。它定义了其他 AMDGPU 后端组件依赖的契约。

## Line-by-Line Analysis / 逐行分析
### Lines 1-34: File banner, includes, and setup
```cpp
//===- AMDGPUDisassembler.hpp - Disassembler for AMDGPU ISA -----*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file
///
/// This file contains declaration for AMDGPU ISA disassembler
//
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_AMDGPU_DISASSEMBLER_AMDGPUDISASSEMBLER_H
#define LLVM_LIB_TARGET_AMDGPU_DISASSEMBLER_AMDGPUDISASSEMBLER_H

#include "SIDefines.h"
#include "llvm/ADT/APInt.h"
#include "llvm/ADT/SmallString.h"
#include "llvm/MC/MCDisassembler/MCDisassembler.h"
#include "llvm/MC/MCInst.h"
#include "llvm/MC/MCInstrInfo.h"
#include "llvm/Support/DataExtractor.h"
#include <memory>

namespace llvm {

class MCAsmInfo;
class MCInst;
class MCOperand;
class MCSubtargetInfo;
class Twine;

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation. Main symbols: `MCAsmInfo`, `MCInst`, `MCOperand`.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。 主要符号：`MCAsmInfo`, `MCInst`, `MCOperand`。

### Lines 35-65: Declares class AMDGPUDisassembler
```cpp
//===----------------------------------------------------------------------===//
// AMDGPUDisassembler
//===----------------------------------------------------------------------===//

class AMDGPUDisassembler : public MCDisassembler {
private:
  std::unique_ptr<MCInstrInfo const> const MCII;
  const MCRegisterInfo &MRI;
  const MCAsmInfo &MAI;
  const unsigned HwModeRegClass;
  const unsigned TargetMaxInstBytes;
  mutable ArrayRef<uint8_t> Bytes;
  mutable uint64_t Literal;
  mutable bool HasLiteral;
  mutable std::optional<bool> EnableWavefrontSize32;
  unsigned CodeObjectVersion;
  const MCExpr *UCVersionW64Expr;
  const MCExpr *UCVersionW32Expr;
  const MCExpr *UCVersionMDPExpr;

  const MCExpr *createConstantSymbolExpr(StringRef Id, int64_t Val);

  void decodeImmOperands(MCInst &MI, const MCInstrInfo &MCII) const;

public:
  AMDGPUDisassembler(const MCSubtargetInfo &STI, MCContext &Ctx,
                     MCInstrInfo const *MCII);
  ~AMDGPUDisassembler() override = default;

  void setABIVersion(unsigned Version) override;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `AMDGPUDisassembler`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`AMDGPUDisassembler`。

### Lines 66-99: Defines getInstruction
```cpp
  DecodeStatus getInstruction(MCInst &MI, uint64_t &Size,
                              ArrayRef<uint8_t> Bytes, uint64_t Address,
                              raw_ostream &CS) const override;

  const char* getRegClassName(unsigned RegClassID) const;

  MCOperand createRegOperand(MCRegister Reg) const;
  MCOperand createRegOperand(unsigned RegClassID, unsigned Val) const;
  MCOperand createSRegOperand(unsigned SRegClassID, unsigned Val) const;
  MCOperand createVGPR16Operand(unsigned RegIdx, bool IsHi) const;

  MCOperand errOperand(unsigned V, const Twine& ErrMsg) const;

  template <typename InsnType>
  DecodeStatus tryDecodeInst(const uint8_t *Table, MCInst &MI, InsnType Inst,
                             uint64_t Address, raw_ostream &Comments) const;
  template <typename InsnType>
  DecodeStatus tryDecodeInst(const uint8_t *Table1, const uint8_t *Table2,
                             MCInst &MI, InsnType Inst, uint64_t Address,
                             raw_ostream &Comments) const;

  Expected<bool> onSymbolStart(SymbolInfoTy &Symbol, uint64_t &Size,
                               ArrayRef<uint8_t> Bytes,
                               uint64_t Address) const override;

  Expected<bool> decodeKernelDescriptor(StringRef KdName,
                                        ArrayRef<uint8_t> Bytes,
                                        uint64_t KdAddress) const;

  Expected<bool>
  decodeKernelDescriptorDirective(DataExtractor::Cursor &Cursor,
                                  ArrayRef<uint8_t> Bytes,
                                  raw_string_ostream &KdStream) const;

```
**EN:** This section contains concrete logic for getInstruction. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 getInstruction 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 100-120: Defines decodeCOMPUTE_PGM_RSRC1
```cpp
  /// Decode as directives that handle COMPUTE_PGM_RSRC1.
  /// \param FourByteBuffer - Bytes holding contents of COMPUTE_PGM_RSRC1.
  /// \param KdStream       - Stream to write the disassembled directives to.
  // NOLINTNEXTLINE(readability-identifier-naming)
  Expected<bool> decodeCOMPUTE_PGM_RSRC1(uint32_t FourByteBuffer,
                                         raw_string_ostream &KdStream) const;

  /// Decode as directives that handle COMPUTE_PGM_RSRC2.
  /// \param FourByteBuffer - Bytes holding contents of COMPUTE_PGM_RSRC2.
  /// \param KdStream       - Stream to write the disassembled directives to.
  // NOLINTNEXTLINE(readability-identifier-naming)
  Expected<bool> decodeCOMPUTE_PGM_RSRC2(uint32_t FourByteBuffer,
                                         raw_string_ostream &KdStream) const;

  /// Decode as directives that handle COMPUTE_PGM_RSRC3.
  /// \param FourByteBuffer - Bytes holding contents of COMPUTE_PGM_RSRC3.
  /// \param KdStream       - Stream to write the disassembled directives to.
  // NOLINTNEXTLINE(readability-identifier-naming)
  Expected<bool> decodeCOMPUTE_PGM_RSRC3(uint32_t FourByteBuffer,
                                         raw_string_ostream &KdStream) const;

```
**EN:** This section contains concrete logic for decodeCOMPUTE_PGM_RSRC1. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 decodeCOMPUTE_PGM_RSRC1 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 121-153: Declares convertEXPInst
```cpp
  void convertEXPInst(MCInst &MI) const;
  void convertVINTERPInst(MCInst &MI) const;
  void convertFMAanyK(MCInst &MI) const;
  void convertSDWAInst(MCInst &MI) const;
  void convertMAIInst(MCInst &MI) const;
  void convertWMMAInst(MCInst &MI) const;
  void convertDPP8Inst(MCInst &MI) const;
  void convertMIMGInst(MCInst &MI) const;
  void convertVOP3DPPInst(MCInst &MI) const;
  void convertVOP3PDPPInst(MCInst &MI) const;
  void convertVOPCDPPInst(MCInst &MI) const;
  void convertVOPC64DPPInst(MCInst &MI) const;
  void convertMacDPPInst(MCInst &MI) const;
  void convertTrue16OpSel(MCInst &MI) const;

  unsigned getVgprClassId(unsigned Width) const;
  unsigned getAgprClassId(unsigned Width) const;
  unsigned getSgprClassId(unsigned Width) const;
  unsigned getTtmpClassId(unsigned Width) const;

  static MCOperand decodeIntImmed(unsigned Imm);

  MCOperand decodeMandatoryLiteralConstant(unsigned Imm) const;
  MCOperand decodeMandatoryLiteral64Constant(uint64_t Imm) const;
  MCOperand decodeLiteralConstant(const MCInstrDesc &Desc,
                                  const MCOperandInfo &OpDesc) const;
  MCOperand decodeLiteral64Constant() const;

  MCOperand decodeSrcOp(const MCInst &Inst, unsigned Width, unsigned Val) const;

  MCOperand decodeNonVGPRSrcOp(const MCInst &Inst, unsigned Width,
                               unsigned Val) const;

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 154-173: Declares decodeVOPDDstYOp
```cpp
  MCOperand decodeVOPDDstYOp(MCInst &Inst, unsigned Val) const;
  MCOperand decodeSpecialReg32(unsigned Val) const;
  MCOperand decodeSpecialReg64(unsigned Val) const;
  MCOperand decodeSpecialReg96Plus(unsigned Val) const;

  MCOperand decodeSDWASrc(unsigned Width, unsigned Val) const;
  MCOperand decodeSDWASrc16(unsigned Val) const;
  MCOperand decodeSDWASrc32(unsigned Val) const;
  MCOperand decodeSDWAVopcDst(unsigned Val) const;

  MCOperand decodeBoolReg(const MCInst &Inst, unsigned Val) const;
  MCOperand decodeSplitBarrier(const MCInst &Inst, unsigned Val) const;
  MCOperand decodeDpp8FI(unsigned Val) const;

  MCOperand decodeVersionImm(unsigned Imm) const;

  int getTTmpIdx(unsigned Val) const;

  const MCInstrInfo *getMCII() const { return MCII.get(); }

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 174-207: Declares class AMDGPUSymbolizer
```cpp
  bool isVI() const;
  bool isGFX9() const;
  bool isGFX90A() const;
  bool isGFX9Plus() const;
  bool isGFX10() const;
  bool isGFX10Plus() const;
  bool isGFX11() const;
  bool isGFX1170() const;
  bool isGFX11Plus() const;
  bool isGFX12() const;
  bool isGFX12Plus() const;
  bool isGFX1250() const;
  bool isGFX1250Plus() const;
  bool isGFX13() const;
  bool isGFX13Plus() const;

  bool hasArchitectedFlatScratch() const;
  bool hasKernargPreload() const;

  bool isMacDPP(MCInst &MI) const;

  /// Check if the instruction is a buffer operation (MUBUF, MTBUF, or S_BUFFER)
  bool isBufferInstruction(const MCInst &MI) const;
};

//===----------------------------------------------------------------------===//
// AMDGPUSymbolizer
//===----------------------------------------------------------------------===//

class AMDGPUSymbolizer : public MCSymbolizer {
private:
  void *DisInfo;
  std::vector<uint64_t> ReferencedAddresses;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `AMDGPUSymbolizer`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`AMDGPUSymbolizer`。

### Lines 208-229: Preprocessor guards and macros
```cpp
public:
  AMDGPUSymbolizer(MCContext &Ctx, std::unique_ptr<MCRelocationInfo> &&RelInfo,
                   void *disInfo)
                   : MCSymbolizer(Ctx, std::move(RelInfo)), DisInfo(disInfo) {}

  bool tryAddingSymbolicOperand(MCInst &Inst, raw_ostream &cStream,
                                int64_t Value, uint64_t Address, bool IsBranch,
                                uint64_t Offset, uint64_t OpSize,
                                uint64_t InstSize) override;

  void tryAddingPcLoadReferenceComment(raw_ostream &cStream,
                                       int64_t Value,
                                       uint64_t Address) override;

  ArrayRef<uint64_t> getReferencedAddresses() const override {
    return ReferencedAddresses;
  }
};

} // end namespace llvm

#endif // LLVM_LIB_TARGET_AMDGPU_DISASSEMBLER_AMDGPUDISASSEMBLER_H
```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled. Main symbols: `std::move`.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。 主要符号：`std::move`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ header
- **Primary symbols / 主要符号**: `MCAsmInfo`, `MCInst`, `MCOperand`, `MCSubtargetInfo`, `Twine`, `AMDGPUDisassembler`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; subtarget modeling / 子目标建模; assembly handling / 汇编处理
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"SIDefines.h"`
- `"llvm/ADT/APInt.h"`
- `"llvm/ADT/SmallString.h"`
- `"llvm/MC/MCDisassembler/MCDisassembler.h"`
- `"llvm/MC/MCInst.h"`
- `"llvm/MC/MCInstrInfo.h"`
- `"llvm/Support/DataExtractor.h"`
- `<memory>`
