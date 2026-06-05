# AMDGPUMCExpr.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/MCTargetDesc/AMDGPUMCExpr.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements AMDGPUMCExpr for the LLVM MC target description layer. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM MC 目标描述层中的 AMDGPUMCExpr 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-30: File banner, includes, and setup
```cpp
//===- AMDGPUMCExpr.cpp - AMDGPU specific MC expression classes -----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "AMDGPUMCExpr.h"
#include "Utils/AMDGPUBaseInfo.h"
#include "llvm/MC/MCAsmInfo.h"
#include "llvm/MC/MCAssembler.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCStreamer.h"
#include "llvm/MC/MCSymbol.h"
#include "llvm/MC/MCValue.h"
#include "llvm/Support/KnownBits.h"
#include "llvm/Support/raw_ostream.h"
#include <functional>
#include <optional>

using namespace llvm;
using namespace llvm::AMDGPU;

AMDGPUMCExpr::AMDGPUMCExpr(VariantKind Kind, ArrayRef<const MCExpr *> Args,
                           MCContext &Ctx)
    : Kind(Kind), Ctx(Ctx) {
  assert(Args.size() >= 1 && "Needs a minimum of one expression.");
  assert(Kind != AGVK_None && "Cannot construct AMDGPUMCExpr of kind none.");

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation. Main symbols: `AMDGPUMCExpr::AMDGPUMCExpr`.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。 主要符号：`AMDGPUMCExpr::AMDGPUMCExpr`。

### Lines 31-54: Implements llvm::uninitialized_copy
```cpp
  // Allocating the variadic arguments through the same allocation mechanism
  // that the object itself is allocated with so they end up in the same memory.
  //
  // Will result in an asan failure if allocated on the heap through standard
  // allocation (e.g., through SmallVector's grow).
  RawArgs = static_cast<const MCExpr **>(
      Ctx.allocate(sizeof(const MCExpr *) * Args.size()));
  llvm::uninitialized_copy(Args, RawArgs);
  this->Args = ArrayRef<const MCExpr *>(RawArgs, Args.size());
}

AMDGPUMCExpr::~AMDGPUMCExpr() { Ctx.deallocate(RawArgs); }

const AMDGPUMCExpr *AMDGPUMCExpr::create(VariantKind Kind,
                                         ArrayRef<const MCExpr *> Args,
                                         MCContext &Ctx) {
  return new (Ctx) AMDGPUMCExpr(Kind, Args, Ctx);
}

const MCExpr *AMDGPUMCExpr::getSubExpr(size_t Index) const {
  assert(Index < Args.size() && "Indexing out of bounds AMDGPUMCExpr sub-expr");
  return Args[Index];
}

```
**EN:** This section contains concrete logic for llvm::uninitialized_copy. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `llvm::uninitialized_copy`, `AMDGPUMCExpr::~AMDGPUMCExpr`, `AMDGPUMCExpr::create`.
**CN:** 本节包含与 llvm::uninitialized_copy 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`llvm::uninitialized_copy`, `AMDGPUMCExpr::~AMDGPUMCExpr`, `AMDGPUMCExpr::create`。

### Lines 55-88: Implements AMDGPUMCExpr::printImpl
```cpp
void AMDGPUMCExpr::printImpl(raw_ostream &OS, const MCAsmInfo *MAI) const {
  switch (Kind) {
  default:
    llvm_unreachable("Unknown AMDGPUMCExpr kind.");
  case AGVK_Or:
    OS << "or(";
    break;
  case AGVK_Max:
    OS << "max(";
    break;
  case AGVK_ExtraSGPRs:
    OS << "extrasgprs(";
    break;
  case AGVK_TotalNumVGPRs:
    OS << "totalnumvgprs(";
    break;
  case AGVK_AlignTo:
    OS << "alignto(";
    break;
  case AGVK_Occupancy:
    OS << "occupancy(";
    break;
  case AGVK_Lit:
    OS << "lit(";
    break;
  case AGVK_Lit64:
    OS << "lit64(";
    break;
  }
  for (const auto *It = Args.begin(); It != Args.end(); ++It) {
    MAI->printExpr(OS, **It);
    if ((It + 1) != Args.end())
      OS << ", ";
  }
```
**EN:** This section contains concrete logic for AMDGPUMCExpr::printImpl. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUMCExpr::printImpl`.
**CN:** 本节包含与 AMDGPUMCExpr::printImpl 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUMCExpr::printImpl`。

### Lines 89-121: Defines op
```cpp
  OS << ')';
}

static int64_t op(AMDGPUMCExpr::VariantKind Kind, int64_t Arg1, int64_t Arg2) {
  switch (Kind) {
  default:
    llvm_unreachable("Unknown AMDGPUMCExpr kind.");
  case AMDGPUMCExpr::AGVK_Max:
    return std::max(Arg1, Arg2);
  case AMDGPUMCExpr::AGVK_Or:
    return Arg1 | Arg2;
  }
}

static bool
evaluateMCExprs(ArrayRef<const MCExpr *> Exprs, const MCAssembler *Asm,
                std::initializer_list<std::reference_wrapper<uint64_t>> Vals) {
  return llvm::all_of(llvm::zip_equal(Exprs, Vals), [&](const auto &Pair) {
    auto [Expr, ValRef] = Pair;
    uint64_t &Val = ValRef.get();
    MCValue MCVal;
    if (!Expr->evaluateAsRelocatable(MCVal, Asm) || !MCVal.isAbsolute())
      return false;
    Val = MCVal.getConstant();
    return true;
  });
}

bool AMDGPUMCExpr::evaluateExtraSGPRs(MCValue &Res,
                                      const MCAssembler *Asm) const {
  const MCSubtargetInfo *STI = Ctx.getSubtargetInfo();
  uint64_t VCCUsed = 0, FlatScrUsed = 0, XNACKUsed = 0;

```
**EN:** This section contains concrete logic for op. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `std::max`, `llvm::all_of`, `llvm::zip_equal`.
**CN:** 本节包含与 op 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`std::max`, `llvm::all_of`, `llvm::zip_equal`。

### Lines 122-155: Implements IsaInfo::getNumExtraSGPRs
```cpp
  if (!evaluateMCExprs(Args, Asm, {VCCUsed, FlatScrUsed, XNACKUsed}))
    return false;

  uint64_t ExtraSGPRs = IsaInfo::getNumExtraSGPRs(
      STI, (bool)VCCUsed, (bool)FlatScrUsed, (bool)XNACKUsed);
  Res = MCValue::get(ExtraSGPRs);
  return true;
}

bool AMDGPUMCExpr::evaluateTotalNumVGPR(MCValue &Res,
                                        const MCAssembler *Asm) const {
  const MCSubtargetInfo *STI = Ctx.getSubtargetInfo();
  uint64_t NumAGPR = 0, NumVGPR = 0;

  bool Has90AInsts = AMDGPU::isGFX90A(*STI);

  if (!evaluateMCExprs(Args, Asm, {NumAGPR, NumVGPR}))
    return false;

  uint64_t TotalNum = Has90AInsts && NumAGPR ? alignTo(NumVGPR, 4) + NumAGPR
                                             : std::max(NumVGPR, NumAGPR);
  Res = MCValue::get(TotalNum);
  return true;
}

bool AMDGPUMCExpr::evaluateAlignTo(MCValue &Res, const MCAssembler *Asm) const {
  uint64_t Value = 0, Align = 0;
  if (!evaluateMCExprs(Args, Asm, {Value, Align}))
    return false;

  Res = MCValue::get(alignTo(Value, Align));
  return true;
}

```
**EN:** This section contains concrete logic for IsaInfo::getNumExtraSGPRs. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `IsaInfo::getNumExtraSGPRs`, `MCValue::get`, `AMDGPUMCExpr::evaluateTotalNumVGPR`.
**CN:** 本节包含与 IsaInfo::getNumExtraSGPRs 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`IsaInfo::getNumExtraSGPRs`, `MCValue::get`, `AMDGPUMCExpr::evaluateTotalNumVGPR`。

### Lines 156-184: Implements AMDGPUMCExpr::evaluateOccupancy
```cpp
bool AMDGPUMCExpr::evaluateOccupancy(MCValue &Res,
                                     const MCAssembler *Asm) const {
  uint64_t InitOccupancy, MaxWaves, Granule, TargetTotalNumVGPRs, Generation,
      NumSGPRs, NumVGPRs;

  bool Success = evaluateMCExprs(
      Args.slice(0, 5), Asm,
      {MaxWaves, Granule, TargetTotalNumVGPRs, Generation, InitOccupancy});

  assert(Success && "Arguments 1 to 5 for Occupancy should be known constants");

  if (!Success || !evaluateMCExprs(Args.slice(5, 2), Asm, {NumSGPRs, NumVGPRs}))
    return false;

  unsigned Occupancy = InitOccupancy;
  if (NumSGPRs)
    Occupancy = std::min(
        Occupancy, IsaInfo::getOccupancyWithNumSGPRs(
                       NumSGPRs, MaxWaves,
                       static_cast<AMDGPUSubtarget::Generation>(Generation)));
  if (NumVGPRs)
    Occupancy = std::min(Occupancy,
                         IsaInfo::getNumWavesPerEUWithNumVGPRs(
                             NumVGPRs, Granule, MaxWaves, TargetTotalNumVGPRs));

  Res = MCValue::get(Occupancy);
  return true;
}

```
**EN:** This section contains concrete logic for AMDGPUMCExpr::evaluateOccupancy. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUMCExpr::evaluateOccupancy`, `std::min`, `IsaInfo::getOccupancyWithNumSGPRs`.
**CN:** 本节包含与 AMDGPUMCExpr::evaluateOccupancy 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUMCExpr::evaluateOccupancy`, `std::min`, `IsaInfo::getOccupancyWithNumSGPRs`。

### Lines 185-215: Implements AMDGPUMCExpr::isSymbolUsedInExpression
```cpp
bool AMDGPUMCExpr::isSymbolUsedInExpression(const MCSymbol *Sym,
                                            const MCExpr *E) {
  switch (E->getKind()) {
  case MCExpr::Constant:
    return false;
  case MCExpr::Unary:
    return isSymbolUsedInExpression(
        Sym, static_cast<const MCUnaryExpr *>(E)->getSubExpr());
  case MCExpr::Binary: {
    const MCBinaryExpr *BE = static_cast<const MCBinaryExpr *>(E);
    return isSymbolUsedInExpression(Sym, BE->getLHS()) ||
           isSymbolUsedInExpression(Sym, BE->getRHS());
  }
  case MCExpr::SymbolRef: {
    const MCSymbol &S = static_cast<const MCSymbolRefExpr *>(E)->getSymbol();
    if (S.isVariable())
      return isSymbolUsedInExpression(Sym, S.getVariableValue());
    return &S == Sym;
  }
  case MCExpr::Specifier:
  case MCExpr::Target: {
    auto *TE = static_cast<const AMDGPUMCExpr *>(E);
    for (const MCExpr *E : TE->getArgs())
      if (isSymbolUsedInExpression(Sym, E))
        return true;
    return false;
  }
  }
  llvm_unreachable("Unknown expr kind!");
}

```
**EN:** This section contains concrete logic for AMDGPUMCExpr::isSymbolUsedInExpression. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUMCExpr::isSymbolUsedInExpression`.
**CN:** 本节包含与 AMDGPUMCExpr::isSymbolUsedInExpression 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUMCExpr::isSymbolUsedInExpression`。

### Lines 216-248: Implements AMDGPUMCExpr::evaluateAsRelocatableImpl
```cpp
bool AMDGPUMCExpr::evaluateAsRelocatableImpl(MCValue &Res,
                                             const MCAssembler *Asm) const {
  std::optional<int64_t> Total;
  switch (Kind) {
  default:
    break;
  case AGVK_ExtraSGPRs:
    return evaluateExtraSGPRs(Res, Asm);
  case AGVK_AlignTo:
    return evaluateAlignTo(Res, Asm);
  case AGVK_TotalNumVGPRs:
    return evaluateTotalNumVGPR(Res, Asm);
  case AGVK_Occupancy:
    return evaluateOccupancy(Res, Asm);
  case AGVK_Lit:
  case AGVK_Lit64:
    return Args[0]->evaluateAsRelocatable(Res, Asm);
  }

  for (const MCExpr *Arg : Args) {
    MCValue ArgRes;
    if (!Arg->evaluateAsRelocatable(ArgRes, Asm) || !ArgRes.isAbsolute())
      return false;

    if (!Total.has_value())
      Total = ArgRes.getConstant();
    Total = op(Kind, *Total, ArgRes.getConstant());
  }

  Res = MCValue::get(*Total);
  return true;
}

```
**EN:** This section contains concrete logic for AMDGPUMCExpr::evaluateAsRelocatableImpl. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUMCExpr::evaluateAsRelocatableImpl`, `MCValue::get`.
**CN:** 本节包含与 AMDGPUMCExpr::evaluateAsRelocatableImpl 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUMCExpr::evaluateAsRelocatableImpl`, `MCValue::get`。

### Lines 249-281: Implements AMDGPUMCExpr::visitUsedExpr
```cpp
void AMDGPUMCExpr::visitUsedExpr(MCStreamer &Streamer) const {
  for (const MCExpr *Arg : Args)
    Streamer.visitUsedExpr(*Arg);
}

MCFragment *AMDGPUMCExpr::findAssociatedFragment() const {
  for (const MCExpr *Arg : Args) {
    if (Arg->findAssociatedFragment())
      return Arg->findAssociatedFragment();
  }
  return nullptr;
}

/// Allow delayed MCExpr resolve of ExtraSGPRs (in case VCCUsed or FlatScrUsed
/// are unresolvable but needed for further MCExprs). Derived from
/// implementation of IsaInfo::getNumExtraSGPRs in AMDGPUBaseInfo.cpp.
///
const AMDGPUMCExpr *AMDGPUMCExpr::createExtraSGPRs(const MCExpr *VCCUsed,
                                                   const MCExpr *FlatScrUsed,
                                                   bool XNACKUsed,
                                                   MCContext &Ctx) {

  return create(AGVK_ExtraSGPRs,
                {VCCUsed, FlatScrUsed, MCConstantExpr::create(XNACKUsed, Ctx)},
                Ctx);
}

const AMDGPUMCExpr *AMDGPUMCExpr::createTotalNumVGPR(const MCExpr *NumAGPR,
                                                     const MCExpr *NumVGPR,
                                                     MCContext &Ctx) {
  return create(AGVK_TotalNumVGPRs, {NumAGPR, NumVGPR}, Ctx);
}

```
**EN:** This section contains concrete logic for AMDGPUMCExpr::visitUsedExpr. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUMCExpr::visitUsedExpr`, `AMDGPUMCExpr::findAssociatedFragment`, `AMDGPUMCExpr::createExtraSGPRs`.
**CN:** 本节包含与 AMDGPUMCExpr::visitUsedExpr 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUMCExpr::visitUsedExpr`, `AMDGPUMCExpr::findAssociatedFragment`, `AMDGPUMCExpr::createExtraSGPRs`。

### Lines 282-313: Implements AMDGPUMCExpr::createLit
```cpp
const AMDGPUMCExpr *AMDGPUMCExpr::createLit(LitModifier Lit, int64_t Value,
                                            MCContext &Ctx) {
  assert(Lit == LitModifier::Lit || Lit == LitModifier::Lit64);
  return create(Lit == LitModifier::Lit ? VariantKind::AGVK_Lit
                                        : VariantKind::AGVK_Lit64,
                {MCConstantExpr::create(Value, Ctx, /*PrintInHex=*/true)}, Ctx);
}

static KnownBits fromOptionalToKnownBits(std::optional<bool> CompareResult) {
  static constexpr unsigned BitWidth = 64;
  const APInt True(BitWidth, 1);
  const APInt False(BitWidth, 0);
  if (CompareResult) {
    return *CompareResult ? KnownBits::makeConstant(True)
                          : KnownBits::makeConstant(False);
  }

  KnownBits UnknownBool(/*BitWidth=*/1);
  return UnknownBool.zext(BitWidth);
}

using KnownBitsMap = DenseMap<const MCExpr *, KnownBits>;
static void knownBitsMapHelper(const MCExpr *Expr, KnownBitsMap &KBM,
                               unsigned Depth = 0);

static void binaryOpKnownBitsMapHelper(const MCExpr *Expr, KnownBitsMap &KBM,
                                       unsigned Depth) {
  static constexpr unsigned BitWidth = 64;
  const MCBinaryExpr *BExpr = cast<MCBinaryExpr>(Expr);
  const MCExpr *LHS = BExpr->getLHS();
  const MCExpr *RHS = BExpr->getRHS();

```
**EN:** This section contains concrete logic for AMDGPUMCExpr::createLit. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUMCExpr::createLit`, `MCConstantExpr::create`, `KnownBits::makeConstant`.
**CN:** 本节包含与 AMDGPUMCExpr::createLit 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUMCExpr::createLit`, `MCConstantExpr::create`, `KnownBits::makeConstant`。

### Lines 314-347: Declares knownBitsMapHelper
```cpp
  knownBitsMapHelper(LHS, KBM, Depth + 1);
  knownBitsMapHelper(RHS, KBM, Depth + 1);
  KnownBits LHSKnown = KBM[LHS];
  KnownBits RHSKnown = KBM[RHS];

  switch (BExpr->getOpcode()) {
  default:
    KBM[Expr] = KnownBits(BitWidth);
    return;
  case MCBinaryExpr::Opcode::Add:
    KBM[Expr] = KnownBits::add(LHSKnown, RHSKnown);
    return;
  case MCBinaryExpr::Opcode::And:
    KBM[Expr] = LHSKnown & RHSKnown;
    return;
  case MCBinaryExpr::Opcode::Div:
    KBM[Expr] = KnownBits::sdiv(LHSKnown, RHSKnown);
    return;
  case MCBinaryExpr::Opcode::EQ: {
    std::optional<bool> CompareRes = KnownBits::eq(LHSKnown, RHSKnown);
    KBM[Expr] = fromOptionalToKnownBits(CompareRes);
    return;
  }
  case MCBinaryExpr::Opcode::NE: {
    std::optional<bool> CompareRes = KnownBits::ne(LHSKnown, RHSKnown);
    KBM[Expr] = fromOptionalToKnownBits(CompareRes);
    return;
  }
  case MCBinaryExpr::Opcode::GT: {
    std::optional<bool> CompareRes = KnownBits::sgt(LHSKnown, RHSKnown);
    KBM[Expr] = fromOptionalToKnownBits(CompareRes);
    return;
  }
  case MCBinaryExpr::Opcode::GTE: {
```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `KnownBits::add`, `KnownBits::sdiv`, `KnownBits::eq`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`KnownBits::add`, `KnownBits::sdiv`, `KnownBits::eq`。

### Lines 348-381: Conditional logic and checks
```cpp
    std::optional<bool> CompareRes = KnownBits::sge(LHSKnown, RHSKnown);
    KBM[Expr] = fromOptionalToKnownBits(CompareRes);
    return;
  }
  case MCBinaryExpr::Opcode::LAnd: {
    std::optional<bool> CompareRes;
    const APInt False(BitWidth, 0);
    std::optional<bool> LHSBool =
        KnownBits::ne(LHSKnown, KnownBits::makeConstant(False));
    std::optional<bool> RHSBool =
        KnownBits::ne(RHSKnown, KnownBits::makeConstant(False));
    if (LHSBool && RHSBool)
      CompareRes = *LHSBool && *RHSBool;
    KBM[Expr] = fromOptionalToKnownBits(CompareRes);
    return;
  }
  case MCBinaryExpr::Opcode::LOr: {
    const APInt False(BitWidth, 0);
    KnownBits Bits = LHSKnown | RHSKnown;
    std::optional<bool> CompareRes =
        KnownBits::ne(Bits, KnownBits::makeConstant(False));
    KBM[Expr] = fromOptionalToKnownBits(CompareRes);
    return;
  }
  case MCBinaryExpr::Opcode::LT: {
    std::optional<bool> CompareRes = KnownBits::slt(LHSKnown, RHSKnown);
    KBM[Expr] = fromOptionalToKnownBits(CompareRes);
    return;
  }
  case MCBinaryExpr::Opcode::LTE: {
    std::optional<bool> CompareRes = KnownBits::sle(LHSKnown, RHSKnown);
    KBM[Expr] = fromOptionalToKnownBits(CompareRes);
    return;
  }
```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `KnownBits::sge`, `KnownBits::ne`, `KnownBits::makeConstant`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`KnownBits::sge`, `KnownBits::ne`, `KnownBits::makeConstant`。

### Lines 382-415: Implementation details and local logic
```cpp
  case MCBinaryExpr::Opcode::Mod:
    KBM[Expr] = KnownBits::srem(LHSKnown, RHSKnown);
    return;
  case MCBinaryExpr::Opcode::Mul:
    KBM[Expr] = KnownBits::mul(LHSKnown, RHSKnown);
    return;
  case MCBinaryExpr::Opcode::Or:
    KBM[Expr] = LHSKnown | RHSKnown;
    return;
  case MCBinaryExpr::Opcode::Shl:
    KBM[Expr] = KnownBits::shl(LHSKnown, RHSKnown);
    return;
  case MCBinaryExpr::Opcode::AShr:
    KBM[Expr] = KnownBits::ashr(LHSKnown, RHSKnown);
    return;
  case MCBinaryExpr::Opcode::LShr:
    KBM[Expr] = KnownBits::lshr(LHSKnown, RHSKnown);
    return;
  case MCBinaryExpr::Opcode::Sub:
    KBM[Expr] = KnownBits::sub(LHSKnown, RHSKnown);
    return;
  case MCBinaryExpr::Opcode::Xor:
    KBM[Expr] = LHSKnown ^ RHSKnown;
    return;
  }
}

static void unaryOpKnownBitsMapHelper(const MCExpr *Expr, KnownBitsMap &KBM,
                                      unsigned Depth) {
  static constexpr unsigned BitWidth = 64;
  const MCUnaryExpr *UExpr = cast<MCUnaryExpr>(Expr);
  knownBitsMapHelper(UExpr->getSubExpr(), KBM, Depth + 1);
  KnownBits KB = KBM[UExpr->getSubExpr()];

```
**EN:** This section advances the file implementation by combining declarations, control flow, and helper operations that support the AMDGPU backend. Main symbols: `KnownBits::srem`, `KnownBits::mul`, `KnownBits::shl`.
**CN:** 本节通过组合声明、控制流与辅助操作来推进文件实现，支撑 AMDGPU 后端的整体功能。 主要符号：`KnownBits::srem`, `KnownBits::mul`, `KnownBits::shl`。

### Lines 416-443: Switch-based control flow
```cpp
  switch (UExpr->getOpcode()) {
  default:
    KBM[Expr] = KnownBits(BitWidth);
    return;
  case MCUnaryExpr::Opcode::Minus: {
    KB.makeNegative();
    KBM[Expr] = std::move(KB);
    return;
  }
  case MCUnaryExpr::Opcode::Not: {
    KnownBits AllOnes(BitWidth);
    AllOnes.setAllOnes();
    KBM[Expr] = KB ^ AllOnes;
    return;
  }
  case MCUnaryExpr::Opcode::Plus: {
    KB.makeNonNegative();
    KBM[Expr] = std::move(KB);
    return;
  }
  }
}

static void targetOpKnownBitsMapHelper(const MCExpr *Expr, KnownBitsMap &KBM,
                                       unsigned Depth) {
  static constexpr unsigned BitWidth = 64;
  const AMDGPUMCExpr *AGVK = cast<AMDGPUMCExpr>(Expr);

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `std::move`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`std::move`。

### Lines 444-477: Switch-based control flow
```cpp
  switch (AGVK->getKind()) {
  default:
    KBM[Expr] = KnownBits(BitWidth);
    return;
  case AMDGPUMCExpr::VariantKind::AGVK_Or: {
    knownBitsMapHelper(AGVK->getSubExpr(0), KBM, Depth + 1);
    KnownBits KB = KBM[AGVK->getSubExpr(0)];
    for (const MCExpr *Arg : AGVK->getArgs()) {
      knownBitsMapHelper(Arg, KBM, Depth + 1);
      KB |= KBM[Arg];
    }
    KBM[Expr] = std::move(KB);
    return;
  }
  case AMDGPUMCExpr::VariantKind::AGVK_Max: {
    knownBitsMapHelper(AGVK->getSubExpr(0), KBM, Depth + 1);
    KnownBits KB = KBM[AGVK->getSubExpr(0)];
    for (const MCExpr *Arg : AGVK->getArgs()) {
      knownBitsMapHelper(Arg, KBM, Depth + 1);
      KB = KnownBits::umax(KB, KBM[Arg]);
    }
    KBM[Expr] = std::move(KB);
    return;
  }
  case AMDGPUMCExpr::VariantKind::AGVK_ExtraSGPRs:
  case AMDGPUMCExpr::VariantKind::AGVK_TotalNumVGPRs:
  case AMDGPUMCExpr::VariantKind::AGVK_AlignTo:
  case AMDGPUMCExpr::VariantKind::AGVK_Occupancy:
  case AMDGPUMCExpr::VariantKind::AGVK_Lit:
  case AMDGPUMCExpr::VariantKind::AGVK_Lit64: {
    int64_t Val;
    if (AGVK->evaluateAsAbsolute(Val)) {
      APInt APValue(BitWidth, Val);
      KBM[Expr] = KnownBits::makeConstant(APValue);
```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `std::move`, `KnownBits::umax`, `KnownBits::makeConstant`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`std::move`, `KnownBits::umax`, `KnownBits::makeConstant`。

### Lines 478-501: Defines knownBitsMapHelper
```cpp
      return;
    }
    KBM[Expr] = KnownBits(BitWidth);
    return;
  }
  }
}

static void knownBitsMapHelper(const MCExpr *Expr, KnownBitsMap &KBM,
                               unsigned Depth) {
  static constexpr unsigned BitWidth = 64;

  int64_t Val;
  if (Expr->evaluateAsAbsolute(Val)) {
    APInt APValue(BitWidth, Val, /*isSigned=*/true);
    KBM[Expr] = KnownBits::makeConstant(APValue);
    return;
  }

  if (Depth == 16) {
    KBM[Expr] = KnownBits(BitWidth);
    return;
  }

```
**EN:** This section contains concrete logic for knownBitsMapHelper. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `KnownBits::makeConstant`.
**CN:** 本节包含与 knownBitsMapHelper 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`KnownBits::makeConstant`。

### Lines 502-525: Switch-based control flow
```cpp
  switch (Expr->getKind()) {
  case MCExpr::ExprKind::Binary: {
    binaryOpKnownBitsMapHelper(Expr, KBM, Depth);
    return;
  }
  case MCExpr::ExprKind::Constant: {
    const MCConstantExpr *CE = cast<MCConstantExpr>(Expr);
    APInt APValue(BitWidth, CE->getValue(), /*isSigned=*/true);
    KBM[Expr] = KnownBits::makeConstant(APValue);
    return;
  }
  case MCExpr::ExprKind::SymbolRef: {
    const MCSymbolRefExpr *RExpr = cast<MCSymbolRefExpr>(Expr);
    const MCSymbol &Sym = RExpr->getSymbol();
    if (!Sym.isVariable()) {
      KBM[Expr] = KnownBits(BitWidth);
      return;
    }

    // Variable value retrieval is not for actual use but only for knownbits
    // analysis.
    const MCExpr *SymVal = Sym.getVariableValue();
    knownBitsMapHelper(SymVal, KBM, Depth + 1);

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `KnownBits::makeConstant`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`KnownBits::makeConstant`。

### Lines 526-558: Conditional logic and checks
```cpp
    // Explicitly copy-construct so that there exists a local KnownBits in case
    // KBM[SymVal] gets invalidated after a potential growth through KBM[Expr].
    KBM[Expr] = KnownBits(KBM[SymVal]);
    return;
  }
  case MCExpr::ExprKind::Unary: {
    unaryOpKnownBitsMapHelper(Expr, KBM, Depth);
    return;
  }
  case MCExpr::ExprKind::Target: {
    targetOpKnownBitsMapHelper(Expr, KBM, Depth);
    return;
  case MCExpr::Specifier:
    llvm_unreachable("unused by this backend");
  }
  }
}

static const MCExpr *tryFoldHelper(const MCExpr *Expr, KnownBitsMap &KBM,
                                   MCContext &Ctx) {
  if (!KBM.count(Expr))
    return Expr;

  auto ValueCheckKnownBits = [](KnownBits &KB, unsigned Value) -> bool {
    if (!KB.isConstant())
      return false;

    return Value == KB.getConstant();
  };

  if (Expr->getKind() == MCExpr::ExprKind::Constant)
    return Expr;

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 559-580: Switch-based control flow
```cpp
  // Resolving unary operations to constants may make the value more ambiguous.
  // For example, `~62` becomes `-63`; however, to me it's more ambiguous if a
  // bit mask value is represented through a negative number.
  if (Expr->getKind() != MCExpr::ExprKind::Unary) {
    if (KBM[Expr].isConstant()) {
      APInt ConstVal = KBM[Expr].getConstant();
      return MCConstantExpr::create(ConstVal.getSExtValue(), Ctx);
    }

    int64_t EvalValue;
    if (Expr->evaluateAsAbsolute(EvalValue))
      return MCConstantExpr::create(EvalValue, Ctx);
  }

  switch (Expr->getKind()) {
  default:
    return Expr;
  case MCExpr::ExprKind::Binary: {
    const MCBinaryExpr *BExpr = cast<MCBinaryExpr>(Expr);
    const MCExpr *LHS = BExpr->getLHS();
    const MCExpr *RHS = BExpr->getRHS();

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `MCConstantExpr::create`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`MCConstantExpr::create`。

### Lines 581-614: Switch-based control flow
```cpp
    switch (BExpr->getOpcode()) {
    default:
      return Expr;
    case MCBinaryExpr::Opcode::Sub: {
      if (ValueCheckKnownBits(KBM[RHS], 0))
        return tryFoldHelper(LHS, KBM, Ctx);
      break;
    }
    case MCBinaryExpr::Opcode::Add:
    case MCBinaryExpr::Opcode::Or: {
      if (ValueCheckKnownBits(KBM[LHS], 0))
        return tryFoldHelper(RHS, KBM, Ctx);
      if (ValueCheckKnownBits(KBM[RHS], 0))
        return tryFoldHelper(LHS, KBM, Ctx);
      break;
    }
    case MCBinaryExpr::Opcode::Mul: {
      if (ValueCheckKnownBits(KBM[LHS], 1))
        return tryFoldHelper(RHS, KBM, Ctx);
      if (ValueCheckKnownBits(KBM[RHS], 1))
        return tryFoldHelper(LHS, KBM, Ctx);
      break;
    }
    case MCBinaryExpr::Opcode::Shl:
    case MCBinaryExpr::Opcode::AShr:
    case MCBinaryExpr::Opcode::LShr: {
      if (ValueCheckKnownBits(KBM[RHS], 0))
        return tryFoldHelper(LHS, KBM, Ctx);
      if (ValueCheckKnownBits(KBM[LHS], 0))
        return MCConstantExpr::create(0, Ctx);
      break;
    }
    case MCBinaryExpr::Opcode::And: {
      if (ValueCheckKnownBits(KBM[LHS], 0) || ValueCheckKnownBits(KBM[RHS], 0))
```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `MCConstantExpr::create`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`MCConstantExpr::create`。

### Lines 615-648: Conditional logic and checks
```cpp
        return MCConstantExpr::create(0, Ctx);
      break;
    }
    }
    const MCExpr *NewLHS = tryFoldHelper(LHS, KBM, Ctx);
    const MCExpr *NewRHS = tryFoldHelper(RHS, KBM, Ctx);
    if (NewLHS != LHS || NewRHS != RHS)
      return MCBinaryExpr::create(BExpr->getOpcode(), NewLHS, NewRHS, Ctx,
                                  BExpr->getLoc());
    return Expr;
  }
  case MCExpr::ExprKind::Unary: {
    const MCUnaryExpr *UExpr = cast<MCUnaryExpr>(Expr);
    const MCExpr *SubExpr = UExpr->getSubExpr();
    const MCExpr *NewSubExpr = tryFoldHelper(SubExpr, KBM, Ctx);
    if (SubExpr != NewSubExpr)
      return MCUnaryExpr::create(UExpr->getOpcode(), NewSubExpr, Ctx,
                                 UExpr->getLoc());
    return Expr;
  }
  case MCExpr::ExprKind::Target: {
    const AMDGPUMCExpr *AGVK = cast<AMDGPUMCExpr>(Expr);
    SmallVector<const MCExpr *, 8> NewArgs;
    bool Changed = false;
    for (const MCExpr *Arg : AGVK->getArgs()) {
      const MCExpr *NewArg = tryFoldHelper(Arg, KBM, Ctx);
      NewArgs.push_back(NewArg);
      Changed |= Arg != NewArg;
    }
    return Changed ? AMDGPUMCExpr::create(AGVK->getKind(), NewArgs, Ctx) : Expr;
  }
  }
  return Expr;
}
```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `MCConstantExpr::create`, `MCBinaryExpr::create`, `MCUnaryExpr::create`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`MCConstantExpr::create`, `MCBinaryExpr::create`, `MCUnaryExpr::create`。

### Lines 649-681: Implements AMDGPU::foldAMDGPUMCExpr
```cpp

const MCExpr *llvm::AMDGPU::foldAMDGPUMCExpr(const MCExpr *Expr,
                                             MCContext &Ctx) {
  KnownBitsMap KBM;
  knownBitsMapHelper(Expr, KBM);
  const MCExpr *NewExpr = tryFoldHelper(Expr, KBM, Ctx);

  return Expr != NewExpr ? NewExpr : Expr;
}

void llvm::AMDGPU::printAMDGPUMCExpr(const MCExpr *Expr, raw_ostream &OS,
                                     const MCAsmInfo *MAI) {
  int64_t Val;
  if (Expr->evaluateAsAbsolute(Val)) {
    OS << Val;
    return;
  }

  MAI->printExpr(OS, *Expr);
}

bool AMDGPU::isLitExpr(const MCExpr *Expr) {
  const auto *E = dyn_cast<AMDGPUMCExpr>(Expr);
  return E && (E->getKind() == AMDGPUMCExpr::AGVK_Lit ||
               E->getKind() == AMDGPUMCExpr::AGVK_Lit64);
}

int64_t AMDGPU::getLitValue(const MCExpr *Expr) {
  assert(isLitExpr(Expr));
  return cast<MCConstantExpr>(cast<AMDGPUMCExpr>(Expr)->getArgs()[0])
      ->getValue();
}

```
**EN:** This section contains concrete logic for AMDGPU::foldAMDGPUMCExpr. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPU::foldAMDGPUMCExpr`, `AMDGPU::printAMDGPUMCExpr`, `AMDGPU::isLitExpr`.
**CN:** 本节包含与 AMDGPU::foldAMDGPUMCExpr 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPU::foldAMDGPUMCExpr`, `AMDGPU::printAMDGPUMCExpr`, `AMDGPU::isLitExpr`。

### Lines 682-687: Implements AMDGPU::getExprKind
```cpp
AMDGPUMCExpr::VariantKind AMDGPU::getExprKind(const MCExpr *Expr) {
  const auto *E = dyn_cast<AMDGPUMCExpr>(Expr);
  if (!E)
    return AMDGPUMCExpr::AGVK_None;
  return E->getKind();
}
```
**EN:** This section contains concrete logic for AMDGPU::getExprKind. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPU::getExprKind`.
**CN:** 本节包含与 AMDGPU::getExprKind 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPU::getExprKind`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `AMDGPUMCExpr::AMDGPUMCExpr`, `llvm::uninitialized_copy`, `AMDGPUMCExpr::~AMDGPUMCExpr`, `AMDGPUMCExpr::create`, `AMDGPUMCExpr::getSubExpr`, `AMDGPUMCExpr::printImpl`
- **Main themes / 核心主题**: subtarget modeling / 子目标建模; assembly handling / 汇编处理
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"AMDGPUMCExpr.h"`
- `"Utils/AMDGPUBaseInfo.h"`
- `"llvm/MC/MCAsmInfo.h"`
- `"llvm/MC/MCAssembler.h"`
- `"llvm/MC/MCContext.h"`
- `"llvm/MC/MCStreamer.h"`
- `"llvm/MC/MCSymbol.h"`
- `"llvm/MC/MCValue.h"`
- `"llvm/Support/KnownBits.h"`
- `"llvm/Support/raw_ostream.h"`
- `<functional>`
- `<optional>`
