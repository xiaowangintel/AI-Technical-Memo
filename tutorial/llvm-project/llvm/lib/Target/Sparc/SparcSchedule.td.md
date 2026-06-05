# SparcSchedule.td — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/Sparc/SparcSchedule.td`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Defines processor scheduling models and itinerary information. This file is written in TableGen DSL rather than C++.
  - **CN**: 定义处理器调度模型和 itinerary 信息。 该文件使用的是 TableGen DSL，而不是 C++。
- **Note / 说明**:
  - **EN**: This file uses LLVM TableGen DSL, so many definitions are declarative records rather than executable C++ statements.
  - **CN**: 该文件使用 LLVM TableGen DSL，因此许多内容是声明式记录，而不是可执行的 C++ 语句。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```tablegen
//===-- SparcSchedule.td - Describe the Sparc Itineraries ----*- tablegen -*-=//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 8-17
```tablegen
//
//
//===----------------------------------------------------------------------===//

def IIC_iu_or_fpu_instr : InstrItinClass;
def IIC_iu_instr : InstrItinClass;
def IIC_fpu_normal_instr : InstrItinClass;
def IIC_fpu_fast_instr : InstrItinClass;
def IIC_jmp_or_call : InstrItinClass;
def IIC_ldd : InstrItinClass;
```
- **EN**: Declares TableGen records such as `IIC_iu_or_fpu_instr`, `IIC_iu_instr`, `IIC_fpu_normal_instr`, `IIC_fpu_fast_instr`, ... for the backend description.
- **CN**: 为后端描述声明了 `IIC_iu_or_fpu_instr`, `IIC_iu_instr`, `IIC_fpu_normal_instr`, `IIC_fpu_fast_instr`, ... 等 TableGen 记录。

### Lines 18-27
```tablegen
def IIC_st : InstrItinClass;
def IIC_std : InstrItinClass;
def IIC_iu_smul : InstrItinClass;
def IIC_iu_umul : InstrItinClass;
def IIC_iu_div : InstrItinClass;
def IIC_ticc : InstrItinClass;
def IIC_ldstub : InstrItinClass;
def IIC_fpu_muls : InstrItinClass;
def IIC_fpu_muld : InstrItinClass;
def IIC_fpu_divs : InstrItinClass;
```
- **EN**: Declares TableGen records such as `IIC_st`, `IIC_std`, `IIC_iu_smul`, `IIC_iu_umul`, ... for the backend description.
- **CN**: 为后端描述声明了 `IIC_st`, `IIC_std`, `IIC_iu_smul`, `IIC_iu_umul`, ... 等 TableGen 记录。

### Lines 28-36
```tablegen
def IIC_fpu_divd : InstrItinClass;
def IIC_fpu_sqrts : InstrItinClass;
def IIC_fpu_sqrtd : InstrItinClass;
def IIC_fpu_abs : InstrItinClass;
def IIC_fpu_movs : InstrItinClass;
def IIC_fpu_negs : InstrItinClass;
def IIC_smac_umac : InstrItinClass;
def IIC_fpu_stod : InstrItinClass;

```
- **EN**: Declares TableGen records such as `IIC_fpu_divd`, `IIC_fpu_sqrts`, `IIC_fpu_sqrtd`, `IIC_fpu_abs`, ... for the backend description.
- **CN**: 为后端描述声明了 `IIC_fpu_divd`, `IIC_fpu_sqrts`, `IIC_fpu_sqrtd`, `IIC_fpu_abs`, ... 等 TableGen 记录。

### Lines 37-41
```tablegen
def LEONIU : FuncUnit; // integer unit
def LEONFPU : FuncUnit; // floating-point unit

// Ref: http://www.atmel.com/Images/doc4226.pdf

```
- **EN**: Declares TableGen records such as `LEONIU`, `LEONFPU` for the backend description.
- **CN**: 为后端描述声明了 `LEONIU`, `LEONFPU` 等 TableGen 记录。

### Lines 42-51
```tablegen
def LEON2Itineraries : ProcessorItineraries<
[LEONIU, LEONFPU], [], [
  InstrItinData<IIC_iu_or_fpu_instr, [InstrStage<1, [LEONIU, LEONFPU]>], [1, 1]>,
  InstrItinData<IIC_iu_instr, [InstrStage<1, [LEONIU]>], [1, 1]>,
  InstrItinData<IIC_fpu_normal_instr, [InstrStage<1, [LEONFPU]>], [7, 1]>,
  InstrItinData<IIC_fpu_fast_instr, [InstrStage<1, [LEONFPU]>], [7, 1]>,
  InstrItinData<IIC_jmp_or_call, [InstrStage<1, [LEONIU, LEONFPU]>], [2, 1]>,
  InstrItinData<IIC_ldd, [InstrStage<1, [LEONIU, LEONFPU]>], [2, 1]>,
  InstrItinData<IIC_st, [InstrStage<1, [LEONIU, LEONFPU]>], [2, 1]>,
  InstrItinData<IIC_std, [InstrStage<1, [LEONIU, LEONFPU]>], [3, 1]>,
```
- **EN**: Declares TableGen records such as `LEON2Itineraries` for the backend description.
- **CN**: 为后端描述声明了 `LEON2Itineraries` 等 TableGen 记录。

### Lines 52-61
```tablegen
  InstrItinData<IIC_iu_smul, [InstrStage<1, [LEONIU]>], [5, 1]>,
  InstrItinData<IIC_iu_umul, [InstrStage<1, [LEONIU]>], [5, 1]>,
  InstrItinData<IIC_iu_div, [InstrStage<1, [LEONIU]>], [35, 1]>,
  InstrItinData<IIC_ticc, [InstrStage<1, [LEONIU, LEONFPU]>], [4, 1]>,
  InstrItinData<IIC_ldstub, [InstrStage<1, [LEONIU, LEONFPU]>], [3, 1]>,
  InstrItinData<IIC_fpu_muls, [InstrStage<1, [LEONFPU]>], [16, 1]>,
  InstrItinData<IIC_fpu_muld, [InstrStage<1, [LEONFPU]>], [21, 1]>,
  InstrItinData<IIC_fpu_divs, [InstrStage<1, [LEONFPU]>], [20, 1]>,
  InstrItinData<IIC_fpu_divd, [InstrStage<1, [LEONFPU]>], [36, 1]>,
  InstrItinData<IIC_fpu_sqrts, [InstrStage<1, [LEONFPU]>], [37, 1]>,
```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 62-68
```tablegen
  InstrItinData<IIC_fpu_sqrtd, [InstrStage<1, [LEONFPU]>], [65, 1]>,
  InstrItinData<IIC_fpu_abs, [InstrStage<1, [LEONFPU]>], [2, 1]>,
  InstrItinData<IIC_fpu_movs, [InstrStage<1, [LEONFPU]>], [2, 1]>,
  InstrItinData<IIC_fpu_negs, [InstrStage<1, [LEONFPU]>], [2, 1]>,
  InstrItinData<IIC_fpu_stod, [InstrStage<1, [LEONFPU]>], [2, 1]>
]>;

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 69-78
```tablegen
def LEON3Itineraries : ProcessorItineraries<
[LEONIU, LEONFPU], [], [
  InstrItinData<IIC_iu_or_fpu_instr, [InstrStage<1, [LEONIU, LEONFPU]>], [1, 1]>,
  InstrItinData<IIC_iu_instr, [InstrStage<1, [LEONIU]>], [1, 1]>,
  InstrItinData<IIC_fpu_normal_instr, [InstrStage<1, [LEONFPU]>], [7, 1]>,
  InstrItinData<IIC_fpu_fast_instr, [InstrStage<1, [LEONFPU]>], [4, 1]>,
  InstrItinData<IIC_jmp_or_call, [InstrStage<1, [LEONIU, LEONFPU]>], [3, 1]>,
  InstrItinData<IIC_ldd, [InstrStage<1, [LEONIU, LEONFPU]>], [2, 1]>,
  InstrItinData<IIC_st, [InstrStage<1, [LEONIU, LEONFPU]>], [4, 1]>,
  InstrItinData<IIC_std, [InstrStage<1, [LEONIU, LEONFPU]>], [5, 1]>,
```
- **EN**: Declares TableGen records such as `LEON3Itineraries` for the backend description.
- **CN**: 为后端描述声明了 `LEON3Itineraries` 等 TableGen 记录。

### Lines 79-88
```tablegen
  InstrItinData<IIC_iu_smul, [InstrStage<1, [LEONIU]>], [1, 1]>,
  InstrItinData<IIC_iu_umul, [InstrStage<1, [LEONIU]>], [4, 1]>,
  InstrItinData<IIC_iu_div, [InstrStage<1, [LEONIU]>], [35, 1]>,
  InstrItinData<IIC_smac_umac, [InstrStage<1, [LEONIU]>], [2, 1]>,
  InstrItinData<IIC_ticc, [InstrStage<1, [LEONIU, LEONFPU]>], [5, 1]>,
  InstrItinData<IIC_ldstub, [InstrStage<1, [LEONIU, LEONFPU]>], [3, 1]>,
  InstrItinData<IIC_fpu_muls, [InstrStage<1, [LEONFPU]>], [4, 1]>,
  InstrItinData<IIC_fpu_muld, [InstrStage<1, [LEONFPU]>], [4, 1]>,
  InstrItinData<IIC_fpu_divs, [InstrStage<1, [LEONFPU]>], [16, 1]>,
  InstrItinData<IIC_fpu_divd, [InstrStage<1, [LEONFPU]>], [17, 1]>,
```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 89-96
```tablegen
  InstrItinData<IIC_fpu_sqrts, [InstrStage<1, [LEONFPU]>], [24, 1]>,
  InstrItinData<IIC_fpu_sqrtd, [InstrStage<1, [LEONFPU]>], [25, 1]>,
  InstrItinData<IIC_fpu_abs, [InstrStage<1, [LEONFPU]>], [2, 1]>,
  InstrItinData<IIC_fpu_movs, [InstrStage<1, [LEONFPU]>], [2, 1]>,
  InstrItinData<IIC_fpu_negs, [InstrStage<1, [LEONFPU]>], [2, 1]>,
  InstrItinData<IIC_fpu_stod, [InstrStage<1, [LEONFPU]>], [4, 1]>
]>;

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 97-106
```tablegen
def LEON4Itineraries : ProcessorItineraries<
[LEONIU, LEONFPU], [], [
  InstrItinData<IIC_iu_or_fpu_instr, [InstrStage<1, [LEONIU, LEONFPU]>], [1, 1]>,
  InstrItinData<IIC_iu_instr, [InstrStage<1, [LEONIU]>], [1, 1]>,
  InstrItinData<IIC_fpu_normal_instr, [InstrStage<1, [LEONFPU]>], [7, 1]>,
  InstrItinData<IIC_fpu_fast_instr, [InstrStage<1, [LEONFPU]>], [4, 1]>,
  InstrItinData<IIC_jmp_or_call, [InstrStage<1, [LEONIU, LEONFPU]>], [3, 1]>,
  InstrItinData<IIC_ldd, [InstrStage<1, [LEONIU, LEONFPU]>], [1, 1]>,
  InstrItinData<IIC_st, [InstrStage<1, [LEONIU, LEONFPU]>], [1, 1]>,
  InstrItinData<IIC_std, [InstrStage<1, [LEONIU, LEONFPU]>], [1, 1]>,
```
- **EN**: Declares TableGen records such as `LEON4Itineraries` for the backend description.
- **CN**: 为后端描述声明了 `LEON4Itineraries` 等 TableGen 记录。

### Lines 107-116
```tablegen
  InstrItinData<IIC_iu_smul, [InstrStage<1, [LEONIU]>], [1, 1]>,
  InstrItinData<IIC_iu_umul, [InstrStage<1, [LEONIU]>], [4, 1]>,
  InstrItinData<IIC_iu_div, [InstrStage<1, [LEONIU]>], [35, 1]>,
  InstrItinData<IIC_smac_umac, [InstrStage<1, [LEONIU]>], [2, 1]>,
  InstrItinData<IIC_ticc, [InstrStage<1, [LEONIU, LEONFPU]>], [5, 1]>,
  InstrItinData<IIC_ldstub, [InstrStage<1, [LEONIU, LEONFPU]>], [3, 1]>,
  InstrItinData<IIC_fpu_muls, [InstrStage<1, [LEONFPU]>], [4, 1]>,
  InstrItinData<IIC_fpu_muld, [InstrStage<1, [LEONFPU]>], [4, 1]>,
  InstrItinData<IIC_fpu_divs, [InstrStage<1, [LEONFPU]>], [16, 1]>,
  InstrItinData<IIC_fpu_divd, [InstrStage<1, [LEONFPU]>], [17, 1]>,
```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 117-123
```tablegen
  InstrItinData<IIC_fpu_sqrts, [InstrStage<1, [LEONFPU]>], [24, 1]>,
  InstrItinData<IIC_fpu_sqrtd, [InstrStage<1, [LEONFPU]>], [25, 1]>,
  InstrItinData<IIC_fpu_abs, [InstrStage<1, [LEONFPU]>], [2, 1]>,
  InstrItinData<IIC_fpu_movs, [InstrStage<1, [LEONFPU]>], [2, 1]>,
  InstrItinData<IIC_fpu_negs, [InstrStage<1, [LEONFPU]>], [2, 1]>,
  InstrItinData<IIC_fpu_stod, [InstrStage<1, [LEONFPU]>], [4, 1]>
]>;
```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

## Key Concepts / 关键概念

- **TableGen DSL / TableGen 领域专用语言**:
  - **EN**: Declarative backend description expanded into generated tables
  - **CN**: 以声明式方式描述后端并生成表格代码
- **Generated backend tables / 生成的后端表**:
  - **EN**: The records here feed TableGen generators that emit `.inc` matcher/data tables
  - **CN**: 此处记录会喂给 TableGen 生成器，产出 `.inc` 匹配器/数据表

## Dependencies / 依赖关系

- **LLVM subsystems / LLVM 子系统**: TableGen generators
