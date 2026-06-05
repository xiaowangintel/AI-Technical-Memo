# AMDGPUMCExpr.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/MCTargetDesc/AMDGPUMCExpr.h`
- **Repository**: llvm/llvm-project
- **Purpose**: This header declares the interfaces, data structures, and pass entry points for AMDGPUMCExpr in the LLVM MC target description layer. It defines the contracts consumed by other AMDGPU backend components. / 该头文件声明 LLVM MC 目标描述层中 AMDGPUMCExpr 的接口、数据结构与 Pass 入口。它定义了其他 AMDGPU 后端组件依赖的契约。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20: File banner, includes, and setup
```cpp
//===- AMDGPUMCExpr.h - AMDGPU specific MC expression classes ---*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_AMDGPU_MCTARGETDESC_AMDGPUMCEXPR_H
#define LLVM_LIB_TARGET_AMDGPU_MCTARGETDESC_AMDGPUMCEXPR_H

#include "llvm/ADT/ArrayRef.h"
#include "llvm/MC/MCExpr.h"

namespace llvm {

class Function;

enum class LitModifier { None, Lit, Lit64 };

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation. Main symbols: `Function`, `LitModifier`.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。 主要符号：`Function`, `LitModifier`。

### Lines 21-44: Declares class AMDGPUMCExpr
```cpp
/// AMDGPU target specific MCExpr operations.
///
/// Takes in a minimum of 1 argument to be used with an operation. The supported
/// operations are:
///   - (bitwise) or
///   - max
///
/// \note If the 'or'/'max' operations are provided only a single argument, the
/// operation will act as a no-op and simply resolve as the provided argument.
///
class AMDGPUMCExpr : public MCTargetExpr {
public:
  enum VariantKind {
    AGVK_None,
    AGVK_Or,
    AGVK_Max,
    AGVK_ExtraSGPRs,
    AGVK_TotalNumVGPRs,
    AGVK_AlignTo,
    AGVK_Occupancy,
    AGVK_Lit,
    AGVK_Lit64,
  };

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `AMDGPUMCExpr`, `VariantKind`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`AMDGPUMCExpr`, `VariantKind`。

### Lines 45-67: Type declarations and aliases
```cpp
  // Relocation specifiers.
  enum Specifier {
    S_None,
    S_GOTPCREL,      // symbol@gotpcrel
    S_GOTPCREL32_LO, // symbol@gotpcrel32@lo
    S_GOTPCREL32_HI, // symbol@gotpcrel32@hi
    S_REL32_LO,      // symbol@rel32@lo
    S_REL32_HI,      // symbol@rel32@hi
    S_REL64,         // symbol@rel64
    S_ABS32_LO,      // symbol@abs32@lo
    S_ABS32_HI,      // symbol@abs32@hi
    S_ABS64,         // symbol@abs64
  };

private:
  VariantKind Kind;
  MCContext &Ctx;
  const MCExpr **RawArgs;
  ArrayRef<const MCExpr *> Args;

  AMDGPUMCExpr(VariantKind Kind, ArrayRef<const MCExpr *> Args, MCContext &Ctx);
  ~AMDGPUMCExpr() override;

```
**EN:** This section introduces supporting types or aliases that simplify later declarations and backend logic. Main symbols: `Specifier`.
**CN:** 本节引入辅助类型或别名，以简化后续声明与后端逻辑。 主要符号：`Specifier`。

### Lines 68-90: Declares evaluateExtraSGPRs
```cpp
  bool evaluateExtraSGPRs(MCValue &Res, const MCAssembler *Asm) const;
  bool evaluateTotalNumVGPR(MCValue &Res, const MCAssembler *Asm) const;
  bool evaluateAlignTo(MCValue &Res, const MCAssembler *Asm) const;
  bool evaluateOccupancy(MCValue &Res, const MCAssembler *Asm) const;

public:
  static const AMDGPUMCExpr *
  create(VariantKind Kind, ArrayRef<const MCExpr *> Args, MCContext &Ctx);

  static const AMDGPUMCExpr *createOr(ArrayRef<const MCExpr *> Args,
                                      MCContext &Ctx) {
    return create(VariantKind::AGVK_Or, Args, Ctx);
  }

  static const AMDGPUMCExpr *createMax(ArrayRef<const MCExpr *> Args,
                                       MCContext &Ctx) {
    return create(VariantKind::AGVK_Max, Args, Ctx);
  }

  static const AMDGPUMCExpr *createExtraSGPRs(const MCExpr *VCCUsed,
                                              const MCExpr *FlatScrUsed,
                                              bool XNACKUsed, MCContext &Ctx);

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 91-112: Defines createTotalNumVGPR
```cpp
  static const AMDGPUMCExpr *createTotalNumVGPR(const MCExpr *NumAGPR,
                                                const MCExpr *NumVGPR,
                                                MCContext &Ctx);

  static const AMDGPUMCExpr *
  createAlignTo(const MCExpr *Value, const MCExpr *Align, MCContext &Ctx) {
    return create(VariantKind::AGVK_AlignTo, {Value, Align}, Ctx);
  }

  static const AMDGPUMCExpr *createLit(LitModifier Lit, int64_t Value,
                                       MCContext &Ctx);

  ArrayRef<const MCExpr *> getArgs() const { return Args; }
  VariantKind getKind() const { return Kind; }
  const MCExpr *getSubExpr(size_t Index) const;

  void printImpl(raw_ostream &OS, const MCAsmInfo *MAI) const override;
  bool evaluateAsRelocatableImpl(MCValue &Res,
                                 const MCAssembler *Asm) const override;
  void visitUsedExpr(MCStreamer &Streamer) const override;
  MCFragment *findAssociatedFragment() const override;

```
**EN:** This section contains concrete logic for createTotalNumVGPR. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 createTotalNumVGPR 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 113-135: Defines classof
```cpp
  static bool classof(const MCExpr *E) {
    return E->getKind() == MCExpr::Target;
  }
  static bool isSymbolUsedInExpression(const MCSymbol *Sym, const MCExpr *E);
};

namespace AMDGPU {
// Tries to leverage KnownBits for MCExprs to reduce and limit any composed
// MCExprs printing. E.g., for an expression such as
// ((unevaluatable_sym | 1) & 1) won't evaluate due to unevaluatable_sym and
// would verbosely print the full expression; however, KnownBits should deduce
// the value to be 1. Particularly useful for AMDGPU metadata MCExprs.
void printAMDGPUMCExpr(const MCExpr *Expr, raw_ostream &OS,
                       const MCAsmInfo *MAI);

const MCExpr *foldAMDGPUMCExpr(const MCExpr *Expr, MCContext &Ctx);

static inline AMDGPUMCExpr::Specifier getSpecifier(const MCSymbolRefExpr *SRE) {
  return AMDGPUMCExpr::Specifier(SRE->getKind());
}

LLVM_READONLY bool isLitExpr(const MCExpr *Expr);

```
**EN:** This section contains concrete logic for classof. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUMCExpr::Specifier`.
**CN:** 本节包含与 classof 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUMCExpr::Specifier`。

### Lines 136-143: Preprocessor guards and macros
```cpp
LLVM_READONLY int64_t getLitValue(const MCExpr *Expr);

LLVM_READONLY AMDGPUMCExpr::VariantKind getExprKind(const MCExpr *Expr);

} // end namespace AMDGPU
} // end namespace llvm

#endif // LLVM_LIB_TARGET_AMDGPU_MCTARGETDESC_AMDGPUMCEXPR_H
```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。

## Key Concepts / 关键概念
- **Language / 语言**: C++ header
- **Primary symbols / 主要符号**: `Function`, `LitModifier`, `AMDGPUMCExpr`, `VariantKind`, `Specifier`, `AMDGPUMCExpr::Specifier`
- **Main themes / 核心主题**: assembly handling / 汇编处理
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"llvm/ADT/ArrayRef.h"`
- `"llvm/MC/MCExpr.h"`
