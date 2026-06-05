# DWARFCFIAnalysis.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `llvm/lib/DWARFCFIChecker/DWARFCFIAnalysis.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements DWARF call-frame analysis and validation support.
  - **CN**: 实现 DWARF 调用帧分析与校验支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20
```cpp
//===----------------------------------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "llvm/DWARFCFIChecker/DWARFCFIAnalysis.h"
#include "Registers.h"
#include "llvm/ADT/ArrayRef.h"
#include "llvm/ADT/SmallSet.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/ADT/Twine.h"
#include "llvm/DWARFCFIChecker/DWARFCFIState.h"
#include "llvm/DebugInfo/DWARF/LowLevel/DWARFUnwindTable.h"
#include "llvm/MC/MCAsmInfo.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCDwarf.h"
#include "llvm/MC/MCExpr.h"
```
- **EN**: Pulls in the headers needed by this translation unit, including `llvm/DWARFCFIChecker/DWARFCFIAnalysis.h`, `Registers.h`, `llvm/ADT/ArrayRef.h`, `llvm/ADT/SmallSet.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `llvm/DWARFCFIChecker/DWARFCFIAnalysis.h`, `Registers.h`, `llvm/ADT/ArrayRef.h`, `llvm/ADT/SmallSet.h`。

### Lines 21-31
```cpp
#include "llvm/MC/MCInst.h"
#include "llvm/MC/MCInstrInfo.h"
#include "llvm/MC/MCRegister.h"
#include "llvm/MC/MCRegisterInfo.h"
#include "llvm/MC/MCStreamer.h"
#include "llvm/MC/MCSubtargetInfo.h"
#include "llvm/MC/TargetRegistry.h"
#include "llvm/Support/ErrorHandling.h"
#include "llvm/Support/FormatVariadic.h"
#include <optional>

```
- **EN**: Pulls in the headers needed by this translation unit, including `llvm/MC/MCInst.h`, `llvm/MC/MCInstrInfo.h`, `llvm/MC/MCRegister.h`, `llvm/MC/MCRegisterInfo.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `llvm/MC/MCInst.h`, `llvm/MC/MCInstrInfo.h`, `llvm/MC/MCRegister.h`, `llvm/MC/MCRegisterInfo.h`。

### Lines 32-45
```cpp
using namespace llvm;

struct CFARegOffsetInfo {
  DWARFRegNum Reg;
  int64_t Offset;

  CFARegOffsetInfo(DWARFRegNum Reg, int64_t Offset)
      : Reg(Reg), Offset(Offset) {}

  bool operator==(const CFARegOffsetInfo &RHS) const {
    return Reg == RHS.Reg && Offset == RHS.Offset;
  }
};

```
- **EN**: Introduces declarations for `llvm`, `CFARegOffsetInfo`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `llvm`, `CFARegOffsetInfo` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 46-55
```cpp
static std::optional<CFARegOffsetInfo>
getCFARegOffsetInfo(const dwarf::UnwindRow &UnwindRow) {
  auto CFALocation = UnwindRow.getCFAValue();
  if (CFALocation.getLocation() !=
      dwarf::UnwindLocation::Location::RegPlusOffset)
    return std::nullopt;

  return CFARegOffsetInfo(CFALocation.getRegister(), CFALocation.getOffset());
}

```
- **EN**: Implements logic around `getCFARegOffsetInfo`, `getCFAValue`, `getLocation`, `CFARegOffsetInfo`; this block manipulates DWARF/debug-info concepts.
- **CN**: 围绕 `getCFARegOffsetInfo`, `getCFAValue`, `getLocation`, `CFARegOffsetInfo` 实现具体逻辑；该代码块处理 DWARF/调试信息概念。

### Lines 56-75
```cpp
static SmallSet<DWARFRegNum, 4>
getUnwindRuleRegSet(const dwarf::UnwindRow &UnwindRow, DWARFRegNum Reg) {
  auto MaybeLoc = UnwindRow.getRegisterLocations().getRegisterLocation(Reg);
  assert(MaybeLoc && "the register should be included in the unwinding row");
  auto Loc = *MaybeLoc;

  switch (Loc.getLocation()) {
  case dwarf::UnwindLocation::Location::Unspecified:
  case dwarf::UnwindLocation::Location::Undefined:
  case dwarf::UnwindLocation::Location::Constant:
  case dwarf::UnwindLocation::Location::CFAPlusOffset:
    // [CFA + offset] does not depend on any register because the CFA value is
    // constant throughout the entire frame; only the way to calculate it might
    // change.
  case dwarf::UnwindLocation::Location::DWARFExpr:
    // TODO: Expressions are not supported yet, but if they were to be
    // supported, all the registers used in an expression should extracted and
    // returned here.
    return {};
  case dwarf::UnwindLocation::Location::Same:
```
- **EN**: Implements logic around `getUnwindRuleRegSet`, `getRegisterLocations`, `assert`, `getLocation`; this block uses `switch`-style dispatch; manipulates DWARF/debug-info concepts.
- **CN**: 围绕 `getUnwindRuleRegSet`, `getRegisterLocations`, `assert`, `getLocation` 实现具体逻辑；该代码块使用 `switch` 风格分派，并处理 DWARF/调试信息概念。

### Lines 76-88
```cpp
    return {Reg};
  case dwarf::UnwindLocation::Location::RegPlusOffset:
    return {Loc.getRegister()};
  }
  llvm_unreachable("Unknown dwarf::UnwindLocation::Location enum");
}

DWARFCFIAnalysis::DWARFCFIAnalysis(MCContext *Context, MCInstrInfo const &MCII,
                                   bool IsEH,
                                   ArrayRef<MCCFIInstruction> Prologue)
    : State(Context), Context(Context), MCII(MCII),
      MCRI(Context->getRegisterInfo()), IsEH(IsEH) {

```
- **EN**: Implements logic around `getRegister`, `llvm_unreachable`, `DWARFCFIAnalysis`, `State`, and 1 more symbols; this block manipulates DWARF/debug-info concepts.
- **CN**: 围绕 `getRegister`, `llvm_unreachable`, `DWARFCFIAnalysis`, `State`, and 1 more symbols 实现具体逻辑；该代码块处理 DWARF/调试信息概念。

### Lines 89-100
```cpp
  for (auto LLVMReg : getTrackingRegs(MCRI)) {
    if (MCRI->get(LLVMReg).IsArtificial || MCRI->get(LLVMReg).IsConstant)
      continue;

    DWARFRegNum Reg = MCRI->getDwarfRegNum(LLVMReg, IsEH);
    // TODO: this should be `undefined` instead of `same_value`, but because
    // initial frame state doesn't have any directives about callee saved
    // registers, every register is tracked. After initial frame state is
    // corrected, this should be changed.
    State.update(MCCFIInstruction::createSameValue(nullptr, Reg));
  }

```
- **EN**: Implements logic around `getTrackingRegs`, `get`, `getDwarfRegNum`, `update`.
- **CN**: 围绕 `getTrackingRegs`, `get`, `getDwarfRegNum`, `update` 实现具体逻辑。

### Lines 101-115
```cpp
  // TODO: Ignoring PC should be in the initial frame state.
  State.update(MCCFIInstruction::createUndefined(
      nullptr, MCRI->getDwarfRegNum(MCRI->getProgramCounter(), IsEH)));

  for (auto &&InitialFrameStateCFIDirective :
       Context->getAsmInfo().getInitialFrameState())
    State.update(InitialFrameStateCFIDirective);

  auto MaybeCurrentRow = State.getCurrentUnwindRow();
  assert(MaybeCurrentRow && "there should be at least one row");
  auto MaybeCFA = getCFARegOffsetInfo(*MaybeCurrentRow);
  assert(MaybeCFA &&
         "the CFA information should be describable in [reg + offset] in here");
  auto CFA = *MaybeCFA;

```
- **EN**: Implements logic around `update`, `getDwarfRegNum`, `getAsmInfo`, `getCurrentUnwindRow`, and 2 more symbols.
- **CN**: 围绕 `update`, `getDwarfRegNum`, `getAsmInfo`, `getCurrentUnwindRow`, and 2 more symbols 实现具体逻辑。

### Lines 116-128
```cpp
  // TODO: CFA register callee value is CFA's value, this should be in initial
  // frame state.
  State.update(MCCFIInstruction::createOffset(nullptr, CFA.Reg, 0));

  // Applying the prologue after default assumptions to overwrite them.
  for (auto &&Directive : Prologue)
    State.update(Directive);
}

void DWARFCFIAnalysis::update(const MCInst &Inst,
                              ArrayRef<MCCFIInstruction> Directives) {
  const MCInstrDesc &MCInstInfo = MCII.get(Inst.getOpcode());

```
- **EN**: Implements logic around `update`, `get`.
- **CN**: 围绕 `update`, `get` 实现具体逻辑。

### Lines 129-141
```cpp
  auto MaybePrevRow = State.getCurrentUnwindRow();
  assert(MaybePrevRow && "the analysis should have initialized the "
                         "state with at least one row by now");
  auto PrevRow = *MaybePrevRow;

  for (auto &&Directive : Directives)
    State.update(Directive);

  SmallSet<DWARFRegNum, 4> Writes;
  for (unsigned I = 0; I < MCInstInfo.NumImplicitDefs; I++)
    Writes.insert(MCRI->getDwarfRegNum(
        getSuperReg(MCRI, MCInstInfo.implicit_defs()[I]), IsEH));

```
- **EN**: Implements logic around `getCurrentUnwindRow`, `assert`, `update`, `insert`, and 1 more symbols; this block emits or serializes data to an external representation.
- **CN**: 围绕 `getCurrentUnwindRow`, `assert`, `update`, `insert`, and 1 more symbols 实现具体逻辑；该代码块把数据输出或序列化为外部表示。

### Lines 142-154
```cpp
  for (unsigned I = 0; I < Inst.getNumOperands(); I++) {
    auto &&Op = Inst.getOperand(I);
    if (Op.isReg()) {
      if (I < MCInstInfo.getNumDefs())
        Writes.insert(
            MCRI->getDwarfRegNum(getSuperReg(MCRI, Op.getReg()), IsEH));
    }
  }

  auto MaybeNextRow = State.getCurrentUnwindRow();
  assert(MaybeNextRow && "previous row existed, so should the current row");
  auto NextRow = *MaybeNextRow;

```
- **EN**: Implements logic around `getNumOperands`, `getOperand`, `isReg`, `getNumDefs`, and 4 more symbols; this block emits or serializes data to an external representation.
- **CN**: 围绕 `getNumOperands`, `getOperand`, `isReg`, `getNumDefs`, and 4 more symbols 实现具体逻辑；该代码块把数据输出或序列化为外部表示。

### Lines 155-170
```cpp
  checkCFADiff(Inst, PrevRow, NextRow, Writes);

  for (auto LLVMReg : getTrackingRegs(MCRI)) {
    DWARFRegNum Reg = MCRI->getDwarfRegNum(LLVMReg, IsEH);

    checkRegDiff(Inst, Reg, PrevRow, NextRow, Writes);
  }
}

void DWARFCFIAnalysis::checkRegDiff(const MCInst &Inst, DWARFRegNum Reg,
                                    const dwarf::UnwindRow &PrevRow,
                                    const dwarf::UnwindRow &NextRow,
                                    const SmallSet<DWARFRegNum, 4> &Writes) {
  auto MaybePrevLoc = PrevRow.getRegisterLocations().getRegisterLocation(Reg);
  auto MaybeNextLoc = NextRow.getRegisterLocations().getRegisterLocation(Reg);

```
- **EN**: Implements logic around `checkCFADiff`, `getTrackingRegs`, `getDwarfRegNum`, `checkRegDiff`, and 1 more symbols; this block emits or serializes data to an external representation; manipulates DWARF/debug-info concepts.
- **CN**: 围绕 `checkCFADiff`, `getTrackingRegs`, `getDwarfRegNum`, `checkRegDiff`, and 1 more symbols 实现具体逻辑；该代码块把数据输出或序列化为外部表示，并处理 DWARF/调试信息概念。

### Lines 171-181
```cpp
  // All the tracked registers are added during initiation. So if a register is
  // not added, should stay the same during execution and vice versa.
  if (!MaybePrevLoc) {
    assert(!MaybeNextLoc && "the register unwind info suddenly appeared here");
    return;
  }
  assert(MaybeNextLoc && "the register unwind info suddenly vanished here");

  auto PrevLoc = MaybePrevLoc.value();
  auto NextLoc = MaybeNextLoc.value();

```
- **EN**: Implements logic around `assert`, `value`.
- **CN**: 围绕 `assert`, `value` 实现具体逻辑。

### Lines 182-193
```cpp
  auto MaybeLLVMReg = MCRI->getLLVMRegNum(Reg, IsEH);
  if (!MaybeLLVMReg) {
    if (!(PrevLoc == NextLoc))
      Context->reportWarning(
          Inst.getLoc(),
          formatv("the dwarf register {0} does not have a LLVM number, but its "
                  "unwind info changed. Ignoring this change",
                  Reg));
    return;
  }
  const char *RegName = MCRI->getName(*MaybeLLVMReg);

```
- **EN**: Implements logic around `getLLVMRegNum`, `reportWarning`, `getLoc`, `formatv`, and 1 more symbols; this block manipulates DWARF/debug-info concepts.
- **CN**: 围绕 `getLLVMRegNum`, `reportWarning`, `getLoc`, `formatv`, and 1 more symbols 实现具体逻辑；该代码块处理 DWARF/调试信息概念。

### Lines 194-213
```cpp
  // Each case is annotated with its corresponding number as described in
  // `llvm/include/llvm/DWARFCFIChecker/DWARFCFIAnalysis.h`.

  // TODO: Expressions are not supported yet, but if they were to be supported,
  // note that structure equality for expressions is defined as follows: Two
  // expressions are structurally equal if they become the same after you
  // replace every operand with a placeholder.

  if (PrevLoc == NextLoc) { // Case 1
    for (DWARFRegNum UsedReg : getUnwindRuleRegSet(PrevRow, Reg))
      if (Writes.count(UsedReg)) { // Case 1.b
        auto MaybeLLVMUsedReg = MCRI->getLLVMRegNum(UsedReg, IsEH);
        assert(MaybeLLVMUsedReg && "instructions will always write to a "
                                   "register that has an LLVM register number");
        Context->reportError(
            Inst.getLoc(),
            formatv("changed register {1}, that register {0}'s unwinding rule "
                    "uses, but there is no CFI directives about it",
                    RegName, MCRI->getName(*MaybeLLVMUsedReg)));
        return;
```
- **EN**: Implements logic around `getUnwindRuleRegSet`, `count`, `getLLVMRegNum`, `assert`, and 4 more symbols; this block emits or serializes data to an external representation; manipulates DWARF/debug-info concepts.
- **CN**: 围绕 `getUnwindRuleRegSet`, `count`, `getLLVMRegNum`, `assert`, and 4 more symbols 实现具体逻辑；该代码块把数据输出或序列化为外部表示，并处理 DWARF/调试信息概念。

### Lines 214-233
```cpp
      }
    return; // Case 1.a
  }
  // Case 2
  if (PrevLoc.getLocation() != NextLoc.getLocation()) { // Case 2.a
    Context->reportWarning(
        Inst.getLoc(),
        formatv("validating changes happening to register {0} unwinding "
                "rule structure is not implemented yet",
                RegName));
    return;
  }
  auto &&PrevRegSet = getUnwindRuleRegSet(PrevRow, Reg);
  if (PrevRegSet != getUnwindRuleRegSet(NextRow, Reg)) { // Case 2.b
    Context->reportWarning(
        Inst.getLoc(),
        formatv("validating changes happening to register {0} unwinding "
                "rule register set is not implemented yet",
                RegName));
    return;
```
- **EN**: Implements logic around `getLocation`, `reportWarning`, `getLoc`, `formatv`, and 1 more symbols.
- **CN**: 围绕 `getLocation`, `reportWarning`, `getLoc`, `formatv`, and 1 more symbols 实现具体逻辑。

### Lines 234-252
```cpp
  }
  // Case 2.c
  for (DWARFRegNum UsedReg : PrevRegSet)
    if (Writes.count(UsedReg)) { // Case 2.c.i
      Context->reportWarning(
          Inst.getLoc(),
          formatv("register {0} unwinding rule's offset is changed, and one of "
                  "the rule's registers is modified, but validating the "
                  "modification amount is not implemented yet",
                  RegName));
      return;
    }
  // Case 2.c.ii
  Context->reportError(
      Inst.getLoc(), formatv("register {0} unwinding rule's offset is changed, "
                             "but not any of the rule's registers are modified",
                             RegName));
}

```
- **EN**: Implements logic around `count`, `reportWarning`, `getLoc`, `formatv`, and 1 more symbols; this block emits or serializes data to an external representation.
- **CN**: 围绕 `count`, `reportWarning`, `getLoc`, `formatv`, and 1 more symbols 实现具体逻辑；该代码块把数据输出或序列化为外部表示。

### Lines 253-268
```cpp
void DWARFCFIAnalysis::checkCFADiff(const MCInst &Inst,
                                    const dwarf::UnwindRow &PrevRow,
                                    const dwarf::UnwindRow &NextRow,
                                    const SmallSet<DWARFRegNum, 4> &Writes) {

  auto MaybePrevCFA = getCFARegOffsetInfo(PrevRow);
  auto MaybeNextCFA = getCFARegOffsetInfo(NextRow);

  if (!MaybePrevCFA) {
    if (MaybeNextCFA) {
      Context->reportWarning(Inst.getLoc(),
                             "CFA rule changed to [reg + offset], this "
                             "transition will not be checked");
      return;
    }

```
- **EN**: Implements logic around `checkCFADiff`, `getCFARegOffsetInfo`, `reportWarning`; this block emits or serializes data to an external representation; manipulates DWARF/debug-info concepts.
- **CN**: 围绕 `checkCFADiff`, `getCFARegOffsetInfo`, `reportWarning` 实现具体逻辑；该代码块把数据输出或序列化为外部表示，并处理 DWARF/调试信息概念。

### Lines 269-280
```cpp
    Context->reportWarning(Inst.getLoc(),
                           "CFA rule is not [reg + offset], not checking it");
    return;
  }

  if (!MaybeNextCFA) {
    Context->reportWarning(Inst.getLoc(),
                           "CFA rule changed from [reg + offset], this "
                           "transition will not be checked");
    return;
  }

```
- **EN**: Implements logic around `reportWarning`.
- **CN**: 围绕 `reportWarning` 实现具体逻辑。

### Lines 281-290
```cpp
  auto PrevCFA = *MaybePrevCFA;
  auto NextCFA = *MaybeNextCFA;

  auto MaybeLLVMPrevReg = MCRI->getLLVMRegNum(PrevCFA.Reg, IsEH);
  const char *PrevCFARegName =
      MaybeLLVMPrevReg ? MCRI->getName(*MaybeLLVMPrevReg) : "";
  auto MaybeLLVMNextReg = MCRI->getLLVMRegNum(NextCFA.Reg, IsEH);
  const char *NextCFARegName =
      MaybeLLVMNextReg ? MCRI->getName(*MaybeLLVMNextReg) : "";

```
- **EN**: Implements logic around `getLLVMRegNum`, `getName`.
- **CN**: 围绕 `getLLVMRegNum`, `getName` 实现具体逻辑。

### Lines 291-301
```cpp
  if (PrevCFA == NextCFA) {         // Case 1
    if (!Writes.count(PrevCFA.Reg)) // Case 1.a
      return;
    // Case 1.b
    Context->reportError(
        Inst.getLoc(),
        formatv("modified CFA register {0} but not changed CFA rule",
                PrevCFARegName));
    return;
  }

```
- **EN**: Implements logic around `count`, `reportError`, `getLoc`, `formatv`; this block emits or serializes data to an external representation.
- **CN**: 围绕 `count`, `reportError`, `getLoc`, `formatv` 实现具体逻辑；该代码块把数据输出或序列化为外部表示。

### Lines 302-321
```cpp
  if (PrevCFA.Reg != NextCFA.Reg) { // Case 2.b
    Context->reportWarning(
        Inst.getLoc(),
        formatv("CFA register changed from register {0} to register {1}, "
                "validating this change is not implemented yet",
                PrevCFARegName, NextCFARegName));
    return;
  }
  // Case 2.c
  if (Writes.count(PrevCFA.Reg)) { // Case 2.c.i
    Context->reportWarning(
        Inst.getLoc(), formatv("CFA offset is changed from {0} to {1}, and CFA "
                               "register {2} is modified, but validating the "
                               "modification amount is not implemented yet",
                               PrevCFA.Offset, NextCFA.Offset, PrevCFARegName));
    return;
  }
  // Case 2.c.ii
  Context->reportError(
      Inst.getLoc(),
```
- **EN**: Implements logic around `reportWarning`, `getLoc`, `formatv`, `count`, and 1 more symbols; this block emits or serializes data to an external representation.
- **CN**: 围绕 `reportWarning`, `getLoc`, `formatv`, `count`, and 1 more symbols 实现具体逻辑；该代码块把数据输出或序列化为外部表示。

### Lines 322-324
```cpp
      formatv("did not modify CFA register {0} but changed CFA rule",
              PrevCFARegName));
}
```
- **EN**: Implements logic around `formatv`.
- **CN**: 围绕 `formatv` 实现具体逻辑。

## Key Concepts / 关键概念

- **CFI validation / CFI 校验**:
  - **EN**: Checks DWARF call frame information against function behavior.
  - **CN**: 将 DWARF 调用帧信息与函数行为进行比对校验。
- **LLVM container usage / LLVM 容器使用**:
  - **EN**: Relies on LLVM ADT containers for performance-conscious in-memory data management.
  - **CN**: 依赖 LLVM ADT 容器来进行注重性能的内存数据管理。

## Dependencies / 依赖关系

- **Direct LLVM/local includes / 直接的 LLVM/本地包含**: `llvm/DWARFCFIChecker/DWARFCFIAnalysis.h`, `Registers.h`, `llvm/ADT/ArrayRef.h`, `llvm/ADT/SmallSet.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/Twine.h`, `llvm/DWARFCFIChecker/DWARFCFIState.h`, `llvm/DebugInfo/DWARF/LowLevel/DWARFUnwindTable.h`, `llvm/MC/MCAsmInfo.h`, `llvm/MC/MCContext.h` ... (+11 more)
- **Standard-library headers / 标准库头文件**: `<optional>`
- **Subsystem categories / 子系统类别**: machine-code layer support / 机器码层支持 (11), LLVM ADT containers and utility types / LLVM ADT 容器与工具类型 (4), support-library helpers / Support 库辅助功能 (2)
