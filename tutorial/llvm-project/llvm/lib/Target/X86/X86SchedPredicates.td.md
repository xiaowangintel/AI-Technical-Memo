# X86SchedPredicates.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/X86/X86SchedPredicates.td`
- **Repository**: llvm/llvm-project
- **Purpose**: Defines scheduling models for the core X86 backend using LLVM TableGen DSL. / 使用 LLVM TableGen DSL 为X86 后端核心定义调度模型。

## Line-by-Line Analysis / 逐行分析
### Lines 1-30: File header, licensing, and opening context / 文件头、许可证与开场上下文
```tablegen
//===-- X86SchedPredicates.td - X86 Scheduling Predicates --*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines scheduling predicate definitions that are common to
// all X86 subtargets.
//
//===----------------------------------------------------------------------===//

// A predicate used to identify dependency-breaking instructions that clear the
// content of the destination register. Note that this predicate only checks if
// input registers are the same. This predicate doesn't make any assumptions on
// the expected instruction opcodes, because different processors may implement
// different zero-idioms.
def ZeroIdiomPredicate : CheckSameRegOperand<1, 2>;

// A predicate used to identify VPERM that have bits 3 and 7 of their mask set.
// On some processors, these VPERM instructions are zero-idioms.
def ZeroIdiomVPERMPredicate : CheckAll<[
  ZeroIdiomPredicate,
  CheckImmOperand<3, 0x88>
]>;

// A predicate used to check if a LEA instruction uses all three source
// operands: base, index, and offset.
def IsThreeOperandsLEAPredicate: CheckAll<[
```
**EN:** This section uses LLVM TableGen DSL to describe file header, licensing, and opening context for the core X86 backend. Key symbols include ZeroIdiomPredicate, ZeroIdiomVPERMPredicate, IsThreeOperandsLEAPredicate. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的文件头、许可证与开场上下文。关键符号包括 ZeroIdiomPredicate, ZeroIdiomVPERMPredicate, IsThreeOperandsLEAPredicate。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 31-60: TableGen scheduling model records / TableGen 调度模型记录
```tablegen
  // isRegOperand(Base)
  CheckIsRegOperand<1>,
  CheckNot<CheckInvalidRegOperand<1>>,

  // isRegOperand(Index)
  CheckIsRegOperand<3>,
  CheckNot<CheckInvalidRegOperand<3>>,

  // hasLEAOffset(Offset)
  CheckAny<[
    CheckAll<[
      CheckIsImmOperand<4>,
      CheckNot<CheckZeroOperand<4>>
    ]>,
    CheckNonPortable<"MI.getOperand(4).isGlobal()">
  ]>
]>;

def LEACases : MCOpcodeSwitchCase<
    [LEA32r, LEA64r, LEA64_32r, LEA16r],
    MCReturnStatement<IsThreeOperandsLEAPredicate>
>;

// Used to generate the body of a TII member function.
def IsThreeOperandsLEABody :
    MCOpcodeSwitchStatement<[LEACases], MCReturnStatement<FalsePred>>;

// This predicate evaluates to true only if the input machine instruction is a
// 3-operands LEA.  Tablegen automatically generates a new method for it in
// X86GenInstrInfo.
```
**EN:** This section uses LLVM TableGen DSL to describe tablegen scheduling model records for the core X86 backend. Key symbols include LEACases, IsThreeOperandsLEABody. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 调度模型记录。关键符号包括 LEACases, IsThreeOperandsLEABody。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 61-90: TableGen scheduling model records / TableGen 调度模型记录
```tablegen
def IsThreeOperandsLEAFn :
    TIIPredicate<"isThreeOperandsLEA", IsThreeOperandsLEABody>;

// A predicate to check for COND_A and COND_BE CMOVs which have an extra uop
// on recent Intel CPUs.
def IsCMOVArr_Or_CMOVBErr : CheckAny<[
  CheckImmOperand_s<3, "X86::COND_A">,
  CheckImmOperand_s<3, "X86::COND_BE">
]>;

def IsCMOVArm_Or_CMOVBErm : CheckAny<[
  CheckImmOperand_s<7, "X86::COND_A">,
  CheckImmOperand_s<7, "X86::COND_BE">
]>;

// A predicate to check for COND_A and COND_BE SETCCs which have an extra uop
// on recent Intel CPUs.
def IsSETAr_Or_SETBEr : CheckAny<[
  CheckImmOperand_s<1, "X86::COND_A">,
  CheckImmOperand_s<1, "X86::COND_BE">
]>;

def IsSETAm_Or_SETBEm : CheckAny<[
  CheckImmOperand_s<5, "X86::COND_A">,
  CheckImmOperand_s<5, "X86::COND_BE">
]>;

// A predicate used to check if an instruction has a LOCK prefix.
def CheckLockPrefix : CheckFunctionPredicate<
  "X86_MC::hasLockPrefix",
```
**EN:** This section uses LLVM TableGen DSL to describe tablegen scheduling model records for the core X86 backend. Key symbols include IsThreeOperandsLEAFn, IsCMOVArr_Or_CMOVBErr, IsCMOVArm_Or_CMOVBErm, IsSETAr_Or_SETBEr, IsSETAm_Or_SETBEm, CheckLockPrefix. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 调度模型记录。关键符号包括 IsThreeOperandsLEAFn, IsCMOVArr_Or_CMOVBErr, IsCMOVArm_Or_CMOVBErm, IsSETAr_Or_SETBEr, IsSETAm_Or_SETBEm, CheckLockPrefix。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 91-120: TableGen scheduling model records / TableGen 调度模型记录
```tablegen
  "X86InstrInfo::hasLockPrefix"
>;

def IsRegRegCompareAndSwap_8 : CheckOpcode<[ CMPXCHG8rr ]>;

def IsRegMemCompareAndSwap_8 : CheckOpcode<[
  LCMPXCHG8, CMPXCHG8rm
]>;

def IsRegRegCompareAndSwap_16_32_64  : CheckOpcode<[
  CMPXCHG16rr, CMPXCHG32rr, CMPXCHG64rr
]>;

def IsRegMemCompareAndSwap_16_32_64  : CheckOpcode<[
  CMPXCHG16rm, CMPXCHG32rm, CMPXCHG64rm,
  LCMPXCHG16, LCMPXCHG32, LCMPXCHG64,
  LCMPXCHG8B, LCMPXCHG16B
]>;

def IsCompareAndSwap8B  : CheckOpcode<[ CMPXCHG8B, LCMPXCHG8B ]>;
def IsCompareAndSwap16B : CheckOpcode<[ CMPXCHG16B, LCMPXCHG16B ]>;

def IsRegMemCompareAndSwap  : CheckOpcode<
  !listconcat(
    IsRegMemCompareAndSwap_8.ValidOpcodes,
    IsRegMemCompareAndSwap_16_32_64.ValidOpcodes
  )>;

def IsRegRegCompareAndSwap  : CheckOpcode<
  !listconcat(
```
**EN:** This section uses LLVM TableGen DSL to describe tablegen scheduling model records for the core X86 backend. Key symbols include IsRegRegCompareAndSwap_8, IsRegMemCompareAndSwap_8, IsRegRegCompareAndSwap_16_32_64, IsRegMemCompareAndSwap_16_32_64, IsCompareAndSwap8B, IsCompareAndSwap16B. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 调度模型记录。关键符号包括 IsRegRegCompareAndSwap_8, IsRegMemCompareAndSwap_8, IsRegRegCompareAndSwap_16_32_64, IsRegMemCompareAndSwap_16_32_64, IsCompareAndSwap8B, IsCompareAndSwap16B。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 121-143: TableGen scheduling model records / TableGen 调度模型记录
```tablegen
    IsRegRegCompareAndSwap_8.ValidOpcodes,
    IsRegRegCompareAndSwap_16_32_64.ValidOpcodes
  )>;

def IsAtomicCompareAndSwap_8 : CheckAll<[
  CheckLockPrefix,
  IsRegMemCompareAndSwap_8
]>;

def IsAtomicCompareAndSwap : CheckAll<[
  CheckLockPrefix,
  IsRegMemCompareAndSwap
]>;

def IsAtomicCompareAndSwap8B : CheckAll<[
  CheckLockPrefix,
  IsCompareAndSwap8B
]>;

def IsAtomicCompareAndSwap16B : CheckAll<[
  CheckLockPrefix,
  IsCompareAndSwap16B
]>;
```
**EN:** This section uses LLVM TableGen DSL to describe tablegen scheduling model records for the core X86 backend. Key symbols include IsAtomicCompareAndSwap_8, IsAtomicCompareAndSwap, IsAtomicCompareAndSwap8B, IsAtomicCompareAndSwap16B. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 调度模型记录。关键符号包括 IsAtomicCompareAndSwap_8, IsAtomicCompareAndSwap, IsAtomicCompareAndSwap8B, IsAtomicCompareAndSwap16B。这些记录随后会展开为 X86 后端使用的生成表。

## Key Concepts / 关键概念
- Primary topic: scheduling models. / 核心主题：调度模型。
- Subsystem: the core X86 backend. / 所属子系统：X86 后端核心。
- Notable symbols: ZeroIdiomPredicate, ZeroIdiomVPERMPredicate, IsThreeOperandsLEAPredicate, LEACases, IsThreeOperandsLEABody, IsThreeOperandsLEAFn. / 重要符号：ZeroIdiomPredicate, ZeroIdiomVPERMPredicate, IsThreeOperandsLEAPredicate, LEACases, IsThreeOperandsLEABody, IsThreeOperandsLEAFn。
- This file is written in TableGen DSL and contributes generated target tables. / 该文件使用 TableGen DSL 编写，并参与生成目标描述表。

## Dependencies / 依赖关系
- Consumed by LLVM TableGen to emit generated matcher, encoding, register, or scheduling tables. / 由 LLVM TableGen 消费，以生成匹配器、编码、寄存器或调度相关表。
- Closely coupled with sibling X86 .td fragments that assemble the full target description. / 与同级 X86 .td 片段紧密协作，共同组成完整目标描述。
