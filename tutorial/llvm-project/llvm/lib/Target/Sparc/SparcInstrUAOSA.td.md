# SparcInstrUAOSA.td — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/Sparc/SparcInstrUAOSA.td`
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
//=== SparcInstrUAOSA.td - UltraSPARC/Oracle SPARC Architecture extensions ===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 8-11
```tablegen
//
// This file contains instruction formats, definitions and patterns needed for
// UA 2005, UA 2007, and OSA 2011 instructions on SPARC.
//===----------------------------------------------------------------------===//
```
- **EN**: Documents the next group of declarations or explains a target-specific rule.
- **CN**: 为接下来的声明分组提供说明，或解释目标相关规则。

### Lines 12-19
```tablegen

class UA2005RegWin<string asmstr, bits<5> fcn>
    : F3_1<2, 0b110001, (outs), (ins), asmstr, []> {
  let rd = fcn;
  let rs1 = 0;
  let rs2 = 0;
}

```
- **EN**: Declares TableGen records such as `UA2005RegWin`; this block specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `UA2005RegWin` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 20-25
```tablegen
// Convenience template for 4-operand instructions
class FourOp<string OpcStr, bits<6> op3val, bits<4> op5val,
        RegisterClass RC>
      : F3_4<op3val, op5val, (outs RC:$rd), (ins RC:$rs1, RC:$rs2, RC:$rs3),
        !strconcat(OpcStr, " $rs1, $rs2, $rs3, $rd")>;

```
- **EN**: Declares TableGen records such as `FourOp`; this block describes instruction encoding bits.
- **CN**: 声明了 `FourOp` 等 TableGen 记录；这一段同时描述指令编码位。

### Lines 26-33
```tablegen
/// F2_56 multiclass - Define a F2_5/F2_6 pattern in one shot.
multiclass F2_56<string OpcStr, bits<1> cc> {
  def rr  : F2_5<cc, (outs),
                 (ins cbtarget:$imm10, CCOp:$cond, IntRegs:$rs1, IntRegs:$rs2),
                 !strconcat(OpcStr, "$cond $rs1, $rs2, $imm10")>;
  def ri  : F2_6<cc, (outs),
                 (ins cbtarget:$imm10, CCOp:$cond, IntRegs:$rs1, simm5Op:$simm5),
                 !strconcat(OpcStr, "$cond $rs1, $simm5, $imm10")>;
```
- **EN**: Declares TableGen records such as `F2_56`, `rr`, `ri`; this block attaches SelectionDAG patterns; describes instruction encoding bits.
- **CN**: 声明了 `F2_56`, `rr`, `ri` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、描述指令编码位。

### Lines 34-41
```tablegen
}

// UltraSPARC Architecture 2005 Instructions
let Predicates = [HasUA2005] in {
let hasSideEffects = 1 in {
def ALLCLEAN : UA2005RegWin<"allclean", 0b00010>;
def INVALW   : UA2005RegWin<"invalw", 0b00101>;
def NORMALW  : UA2005RegWin<"normalw", 0b00100>;
```
- **EN**: Declares TableGen records such as `ALLCLEAN`, `INVALW`, `NORMALW`; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `ALLCLEAN`, `INVALW`, `NORMALW` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

### Lines 42-45
```tablegen
def OTHERW   : UA2005RegWin<"otherw", 0b00011>;
}
} // Predicates = [HasUA2005]

```
- **EN**: Declares TableGen records such as `OTHERW` for the backend description.
- **CN**: 为后端描述声明了 `OTHERW` 等 TableGen 记录。

### Lines 46-52
```tablegen
// UltraSPARC Architecture 2007 Instructions
let Predicates = [HasUA2007] in {
def FMADDS : FourOp<"fmadds", 0b110111, 0b0001, FPRegs>;
def FMADDD : FourOp<"fmaddd", 0b110111, 0b0010, DFPRegs>;
def FMSUBS : FourOp<"fmsubs", 0b110111, 0b0101, FPRegs>;
def FMSUBD : FourOp<"fmsubd", 0b110111, 0b0110, DFPRegs>;

```
- **EN**: Declares TableGen records such as `FMADDS`, `FMADDD`, `FMSUBS`, `FMSUBD`; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `FMADDS`, `FMADDD`, `FMSUBS`, `FMSUBD` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

### Lines 53-58
```tablegen
def FNMADDS : FourOp<"fnmadds", 0b110111, 0b1101, FPRegs>;
def FNMADDD : FourOp<"fnmaddd", 0b110111, 0b1110, DFPRegs>;
def FNMSUBS : FourOp<"fnmsubs", 0b110111, 0b1001, FPRegs>;
def FNMSUBD : FourOp<"fnmsubd", 0b110111, 0b1010, DFPRegs>;
} // Predicates = [HasUA2007]

```
- **EN**: Declares TableGen records such as `FNMADDS`, `FNMADDD`, `FNMSUBS`, `FNMSUBD` for the backend description.
- **CN**: 为后端描述声明了 `FNMADDS`, `FNMADDD`, `FNMSUBS`, `FNMSUBD` 等 TableGen 记录。

### Lines 59-65
```tablegen
// Oracle SPARC Architecture 2011 Instructions
let Predicates = [HasOSA2011] in {
let isBranch = 1, isTerminator = 1, hasDelaySlot = 0 in {
defm CWBCOND : F2_56<"cwb", 0>;
defm CXBCOND : F2_56<"cxb", 1>;
}

```
- **EN**: Declares TableGen records such as `CWBCOND`, `CXBCOND`; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `CWBCOND`, `CXBCOND` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

### Lines 66-69
```tablegen
def FPMADDX   : FourOp<"fpmaddx", 0b110111, 0b0000, DFPRegs>;
def FPMADDXHI : FourOp<"fpmaddxhi", 0b110111, 0b0100, DFPRegs>;
} // Predicates = [HasOSA2011]

```
- **EN**: Declares TableGen records such as `FPMADDX`, `FPMADDXHI` for the backend description.
- **CN**: 为后端描述声明了 `FPMADDX`, `FPMADDXHI` 等 TableGen 记录。

### Lines 70-77
```tablegen
// UA2007 instruction patterns.
let Predicates = [HasUA2007] in {
def : Pat<(f32 (any_fma f32:$rs1, f32:$rs2, f32:$add)), (FMADDS $rs1, $rs2, $add)>;
def : Pat<(f64 (any_fma f64:$rs1, f64:$rs2, f64:$add)), (FMADDD $rs1, $rs2, $add)>;
def : Pat<(f32 (any_fma f32:$rs1, f32:$rs2, (fneg f32:$sub))), (FMSUBS $rs1, $rs2, $sub)>;
def : Pat<(f64 (any_fma f64:$rs1, f64:$rs2, (fneg f64:$sub))), (FMSUBD $rs1, $rs2, $sub)>;
def : Pat<(f32 (fneg (any_fma f32:$rs1, f32:$rs2, f32:$add))), (FNMADDS $rs1, $rs2, $add)>;
def : Pat<(f64 (fneg (any_fma f64:$rs1, f64:$rs2, f64:$add))), (FNMADDD $rs1, $rs2, $add)>;
```
- **EN**: Implements logic around `Pat<`.
- **CN**: 围绕 `Pat<` 实现具体逻辑。

### Lines 78-80
```tablegen
def : Pat<(f32 (fneg (any_fma f32:$rs1, f32:$rs2, (fneg f32:$sub)))), (FNMSUBS $rs1, $rs2, $sub)>;
def : Pat<(f64 (fneg (any_fma f64:$rs1, f64:$rs2, (fneg f64:$sub)))), (FNMSUBD $rs1, $rs2, $sub)>;
} // Predicates = [HasUA2007]
```
- **EN**: Implements logic around `Pat<`.
- **CN**: 围绕 `Pat<` 实现具体逻辑。

## Key Concepts / 关键概念

- **TableGen DSL / TableGen 领域专用语言**:
  - **EN**: Declarative backend description expanded into generated tables
  - **CN**: 以声明式方式描述后端并生成表格代码
- **Generated backend tables / 生成的后端表**:
  - **EN**: The records here feed TableGen generators that emit `.inc` matcher/data tables
  - **CN**: 此处记录会喂给 TableGen 生成器，产出 `.inc` 匹配器/数据表

## Dependencies / 依赖关系

- **LLVM subsystems / LLVM 子系统**: TableGen generators
