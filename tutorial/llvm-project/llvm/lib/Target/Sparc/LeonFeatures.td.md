# LeonFeatures.td — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/Sparc/LeonFeatures.td`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Defines backend metadata in TableGen DSL that is later expanded into generated tables and matcher code.
  - **CN**: 使用 TableGen DSL 定义后端元数据，随后会展开为生成的表和匹配代码。
- **Note / 说明**:
  - **EN**: This file uses LLVM TableGen DSL, so many definitions are declarative records rather than executable C++ statements.
  - **CN**: 该文件使用 LLVM TableGen DSL，因此许多内容是声明式记录，而不是可执行的 C++ 语句。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```tablegen
//===-- LeonFeatures.td - Describe the Leon Features -------*- tablegen -*-===//
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
```tablegen
//
//
//===----------------------------------------------------------------------===//


```
- **EN**: Documents the next group of declarations or explains a target-specific rule.
- **CN**: 为接下来的声明分组提供说明，或解释目标相关规则。

### Lines 13-22
```tablegen
//===----------------------------------------------------------------------===//
// UMAC and SMAC support for LEON3 and LEON4 processors.
//===----------------------------------------------------------------------===//

//support to casa instruction; for leon3 subtarget only
def UMACSMACSupport : SubtargetFeature<
  "hasumacsmac",
  "HasUmacSmac",
  "true",
  "Enable UMAC and SMAC for LEON3 and LEON4 processors"
```
- **EN**: Declares TableGen records such as `UMACSMACSupport` for the backend description.
- **CN**: 为后端描述声明了 `UMACSMACSupport` 等 TableGen 记录。

### Lines 23-29
```tablegen
>;


//===----------------------------------------------------------------------===//
// CASA Support differs between LEON3-FT GR712RC and LEON3-FT UT699
// We need to have the option to switch this on and off.
//===----------------------------------------------------------------------===//
```
- **EN**: Implements target-specific case analysis using a `switch` over opcodes, fixups, or enum values.
- **CN**: 通过对 opcode、fixup 或枚举值执行 `switch` 分析来实现目标相关逻辑。

### Lines 30-38
```tablegen

//support to casa instruction; for leon3 subtarget only
def LeonCASA : SubtargetFeature<
  "hasleoncasa",
  "HasLeonCasa",
  "true",
  "Enable CASA instruction for LEON3 and LEON4 processors"
>;

```
- **EN**: Declares TableGen records such as `LeonCASA` for the backend description.
- **CN**: 为后端描述声明了 `LeonCASA` 等 TableGen 记录。

### Lines 39-45
```tablegen
def InsertNOPLoad: SubtargetFeature<
  "insertnopload",
  "InsertNOPLoad",
  "true",
  "LEON3 erratum fix: Insert a NOP instruction after every single-cycle load instruction when the next instruction is another load/store instruction"
>;

```
- **EN**: Declares TableGen records such as `InsertNOPLoad` for the backend description.
- **CN**: 为后端描述声明了 `InsertNOPLoad` 等 TableGen 记录。

### Lines 46-53
```tablegen
def DetectRoundChange : SubtargetFeature<
  "detectroundchange",
  "DetectRoundChange",
  "true",
  "LEON3 erratum detection: Detects any rounding mode change "
  "request: use only the round-to-nearest rounding mode"
>;

```
- **EN**: Declares TableGen records such as `DetectRoundChange` for the backend description.
- **CN**: 为后端描述声明了 `DetectRoundChange` 等 TableGen 记录。

### Lines 54-60
```tablegen
def FixAllFDIVSQRT : SubtargetFeature<
  "fixallfdivsqrt",
  "FixAllFDIVSQRT",
  "true",
  "LEON erratum fix: Fix FDIVS/FDIVD/FSQRTS/FSQRTD instructions with NOPs and floating-point store"
>;

```
- **EN**: Declares TableGen records such as `FixAllFDIVSQRT` for the backend description.
- **CN**: 为后端描述声明了 `FixAllFDIVSQRT` 等 TableGen 记录。

### Lines 61-70
```tablegen
def LeonCycleCounter
  : SubtargetFeature<"leoncyclecounter", "HasLeonCycleCounter", "true",
                     "Use the Leon cycle counter register">;

def FixTN0009 : SubtargetFeature<
  "fix-tn0009",
  "FixTN0009",
  "true",
  "Enable workaround for errata described in GRLIB-TN-0009"
>;
```
- **EN**: Declares TableGen records such as `LeonCycleCounter`, `FixTN0009` for the backend description.
- **CN**: 为后端描述声明了 `LeonCycleCounter`, `FixTN0009` 等 TableGen 记录。

### Lines 71-78
```tablegen

def FixTN0010 : SubtargetFeature<
  "fix-tn0010",
  "FixTN0010",
  "true",
  "Enable workaround for errata described in GRLIB-TN-0010"
>;

```
- **EN**: Declares TableGen records such as `FixTN0010` for the backend description.
- **CN**: 为后端描述声明了 `FixTN0010` 等 TableGen 记录。

### Lines 79-85
```tablegen
def FixTN0011 : SubtargetFeature<
  "fix-tn0011",
  "FixTN0011",
  "true",
  "Enable workaround for errata described in GRLIB-TN-0011"
>;

```
- **EN**: Declares TableGen records such as `FixTN0011` for the backend description.
- **CN**: 为后端描述声明了 `FixTN0011` 等 TableGen 记录。

### Lines 86-92
```tablegen
def FixTN0012 : SubtargetFeature<
  "fix-tn0012",
  "FixTN0012",
  "true",
  "Enable workaround for errata described in GRLIB-TN-0012"
>;

```
- **EN**: Declares TableGen records such as `FixTN0012` for the backend description.
- **CN**: 为后端描述声明了 `FixTN0012` 等 TableGen 记录。

### Lines 93-98
```tablegen
def FixTN0013 : SubtargetFeature<
  "fix-tn0013",
  "FixTN0013",
  "true",
  "Enable workaround for errata described in GRLIB-TN-0013"
>;
```
- **EN**: Declares TableGen records such as `FixTN0013` for the backend description.
- **CN**: 为后端描述声明了 `FixTN0013` 等 TableGen 记录。

## Key Concepts / 关键概念

- **TableGen DSL / TableGen 领域专用语言**:
  - **EN**: Declarative backend description expanded into generated tables
  - **CN**: 以声明式方式描述后端并生成表格代码
- **Generated backend tables / 生成的后端表**:
  - **EN**: The records here feed TableGen generators that emit `.inc` matcher/data tables
  - **CN**: 此处记录会喂给 TableGen 生成器，产出 `.inc` 匹配器/数据表

## Dependencies / 依赖关系

- **LLVM subsystems / LLVM 子系统**: TableGen generators
