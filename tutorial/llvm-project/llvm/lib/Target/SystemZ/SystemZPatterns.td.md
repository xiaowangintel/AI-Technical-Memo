# SystemZPatterns.td — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/SystemZPatterns.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **用途 (CN)**: 使用 LLVM TableGen DSL 定义声明式的 SystemZ 后端记录与匹配模式。
- **Note / 说明**: This file is written in LLVM TableGen DSL rather than standard C++ source. / 该文件使用 LLVM TableGen DSL，而不是普通 C++ 源码。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行
```tablegen
   1: //===-- SystemZPatterns.td - SystemZ-specific pattern rules ---*- tblgen-*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
   9: // Record that INSN performs a 64-bit version of unary operator OPERATOR
  10: // in which the operand is sign-extended from 32 to 64 bits.
  11: multiclass SXU<SDPatternOperator operator, Instruction insn> {
  12:   def : Pat<(operator (sext (i32 GR32:$src))),
  13:             (insn GR32:$src)>;
  14:   def : Pat<(operator (sext_inreg GR64:$src, i32)),
  15:             (insn (EXTRACT_SUBREG GR64:$src, subreg_l32))>;
  16: }
  17: 
  18: // Record that INSN performs a 64-bit version of binary operator OPERATOR
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. This block declares or refines TableGen records such as `SXU`.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 该代码块声明或细化了 `SXU` 等 TableGen 记录。

### Lines 19-36 / 第 19-36 行
```tablegen
  19: // in which the first operand has class CLS and which the second operand
  20: // is sign-extended from a 32-bit register.
  21: multiclass SXB<SDPatternOperator operator, RegisterOperand cls,
  22:                Instruction insn> {
  23:   def : Pat<(operator cls:$src1, (sext GR32:$src2)),
  24:             (insn cls:$src1, GR32:$src2)>;
  25:   def : Pat<(operator cls:$src1, (sext_inreg GR64:$src2, i32)),
  26:             (insn cls:$src1, (EXTRACT_SUBREG GR64:$src2, subreg_l32))>;
  27: }
  28: 
  29: // Like SXB, but for zero extension.
  30: multiclass ZXB<SDPatternOperator operator, RegisterOperand cls,
  31:                Instruction insn> {
  32:   def : Pat<(operator cls:$src1, (zext GR32:$src2)),
  33:             (insn cls:$src1, GR32:$src2)>;
  34:   def : Pat<(operator cls:$src1, (and GR64:$src2, 0xffffffff)),
  35:             (insn cls:$src1, (EXTRACT_SUBREG GR64:$src2, subreg_l32))>;
  36: }
```
- **EN**: This block declares or refines TableGen records such as `SXB`, `ZXB`.
- **CN**: 该代码块声明或细化了 `SXB`, `ZXB` 等 TableGen 记录。

### Lines 37-54 / 第 37-54 行
```tablegen
  37: 
  38: // Record that INSN performs a binary read-modify-write operation,
  39: // with LOAD, OPERATOR and STORE being the read, modify and write
  40: // respectively.  MODE is the addressing mode and IMM is the type
  41: // of the second operand.
  42: class RMWI<SDPatternOperator load, SDPatternOperator operator,
  43:            SDPatternOperator store, AddressingMode mode,
  44:            PatFrag imm, Instruction insn>
  45:   : Pat<(store (operator (load mode:$addr), imm:$src), mode:$addr),
  46:         (insn mode:$addr, (UIMM8 imm:$src))>;
  47: 
  48: // Record that INSN performs binary operation OPERATION on a byte
  49: // memory location.  IMM is the type of the second operand.
  50: multiclass RMWIByte<SDPatternOperator operator, AddressingMode mode,
  51:                     Instruction insn> {
  52:   def : RMWI<z_anyextloadi8, operator, truncstorei8, mode, imm32, insn>;
  53:   def : RMWI<z_anyextloadi8, operator, truncstorei8, mode, imm64, insn>;
  54: }
```
- **EN**: This block declares or refines TableGen records such as `RMWI`, `RMWIByte`.
- **CN**: 该代码块声明或细化了 `RMWI`, `RMWIByte` 等 TableGen 记录。

### Lines 55-72 / 第 55-72 行
```tablegen
  55: 
  56: // Record that INSN performs insertion TYPE into a register of class CLS.
  57: // The inserted operand is loaded using LOAD from an address of mode MODE.
  58: multiclass InsertMem<string type, Instruction insn, RegisterOperand cls,
  59:                      SDPatternOperator load, AddressingMode mode> {
  60:   def : Pat<(!cast<SDPatternOperator>("or_as_"#type)
  61:               cls:$src1, (load mode:$src2)),
  62:             (insn cls:$src1, mode:$src2)>;
  63:   def : Pat<(!cast<SDPatternOperator>("or_as_rev"#type)
  64:               (load mode:$src2), cls:$src1),
  65:             (insn cls:$src1, mode:$src2)>;
  66: }
  67: 
  68: // INSN stores the low 32 bits of a GPR to a memory with addressing mode MODE.
  69: // Record that it is equivalent to using OPERATOR to store a GR64.
  70: class StoreGR64<Instruction insn, SDPatternOperator operator,
  71:                 AddressingMode mode>
  72:   : Pat<(operator GR64:$R1, mode:$XBD2),
```
- **EN**: This block declares or refines TableGen records such as `InsertMem`, `StoreGR64`.
- **CN**: 该代码块声明或细化了 `InsertMem`, `StoreGR64` 等 TableGen 记录。

### Lines 73-90 / 第 73-90 行
```tablegen
  73:         (insn (EXTRACT_SUBREG GR64:$R1, subreg_l32), mode:$XBD2)>;
  74: 
  75: // INSN and INSNY are an RX/RXY pair of instructions that store the low
  76: // 32 bits of a GPR to memory.  Record that they are equivalent to using
  77: // OPERATOR to store a GR64.
  78: multiclass StoreGR64Pair<Instruction insn, Instruction insny,
  79:                          SDPatternOperator operator> {
  80:   def : StoreGR64<insn, operator, bdxaddr12pair>;
  81:   def : StoreGR64<insny, operator, bdxaddr20pair>;
  82: }
  83: 
  84: // INSN stores the low 32 bits of a GPR using PC-relative addressing.
  85: // Record that it is equivalent to using OPERATOR to store a GR64.
  86: class StoreGR64PC<Instruction insn, SDPatternOperator operator>
  87:   : Pat<(operator GR64:$R1, pcrel32:$XBD2),
  88:         (insn (EXTRACT_SUBREG GR64:$R1, subreg_l32), pcrel32:$XBD2)> {
  89:   // We want PC-relative addresses to be tried ahead of BD and BDX addresses.
  90:   // However, BDXs have two extra operands and are therefore 6 units more
```
- **EN**: This block declares or refines TableGen records such as `StoreGR64Pair`, `StoreGR64PC`.
- **CN**: 该代码块声明或细化了 `StoreGR64Pair`, `StoreGR64PC` 等 TableGen 记录。

### Lines 91-108 / 第 91-108 行
```tablegen
  91:   // complex.
  92:   let AddedComplexity = 7;
  93: }
  94: 
  95: // INSN and INSNINV conditionally store the low 32 bits of a GPR to memory,
  96: // with INSN storing when the condition is true and INSNINV storing when the
  97: // condition is false.  Record that they are equivalent to a LOAD/select/STORE
  98: // sequence for GR64s.
  99: multiclass CondStores64<Instruction insn, Instruction insninv,
 100:                         SDPatternOperator store, SDPatternOperator load,
 101:                         AddressingMode mode> {
 102:   def : Pat<(store (z_select_ccmask GR64:$new, (load mode:$addr),
 103:                                     imm32zx4_timm:$valid, imm32zx4_timm:$cc),
 104:                    mode:$addr),
 105:             (insn (EXTRACT_SUBREG GR64:$new, subreg_l32), mode:$addr,
 106:                   imm32zx4:$valid, imm32zx4:$cc)>;
 107:   def : Pat<(store (z_select_ccmask (load mode:$addr), GR64:$new,
 108:                                     imm32zx4_timm:$valid, imm32zx4_timm:$cc),
```
- **EN**: This block declares or refines TableGen records such as `CondStores64`.
- **CN**: 该代码块声明或细化了 `CondStores64` 等 TableGen 记录。

### Lines 109-126 / 第 109-126 行
```tablegen
 109:                    mode:$addr),
 110:             (insninv (EXTRACT_SUBREG GR64:$new, subreg_l32), mode:$addr,
 111:                      imm32zx4:$valid, imm32zx4:$cc)>;
 112: }
 113: 
 114: // Try to use MVC instruction INSN for a load of type LOAD followed by a store
 115: // of the same size.  VT is the type of the intermediate (legalized) value and
 116: // LENGTH is the number of bytes loaded by LOAD.
 117: multiclass MVCLoadStore<SDPatternOperator load, ValueType vt, Instruction insn,
 118:                         bits<5> length> {
 119:   def : Pat<(mvc_store (vt (load bdaddr12only:$src)), bdaddr12only:$dest),
 120:             (insn bdaddr12only:$dest, bdaddr12only:$src, length)>;
 121: }
 122: 
 123: // Use NC-like instruction INSN for block_op operation OPERATOR.
 124: // The other operand is a load of type LOAD, which accesses LENGTH bytes.
 125: // VT is the intermediate legalized type in which the binary operation
 126: // is actually done.
```
- **EN**: This block declares or refines TableGen records such as `MVCLoadStore`.
- **CN**: 该代码块声明或细化了 `MVCLoadStore` 等 TableGen 记录。

### Lines 127-144 / 第 127-144 行
```tablegen
 127: multiclass BinaryLoadStore<SDPatternOperator operator, SDPatternOperator load,
 128:                            ValueType vt, Instruction insn, bits<5> length> {
 129:   def : Pat<(operator (vt (load bdaddr12only:$src)), bdaddr12only:$dest),
 130:             (insn bdaddr12only:$dest, bdaddr12only:$src, length)>;
 131: }
 132: 
 133: // A convenient way of generating all block peepholes for a particular
 134: // LOAD/VT/LENGTH combination.
 135: multiclass BlockLoadStore<SDPatternOperator load, ValueType vt,
 136:                           Instruction mvc, Instruction nc, Instruction oc,
 137:                           Instruction xc, bits<5> length> {
 138:   defm : MVCLoadStore<load, vt, mvc, length>;
 139:   defm : BinaryLoadStore<block_and1, load, vt, nc, length>;
 140:   defm : BinaryLoadStore<block_and2, load, vt, nc, length>;
 141:   defm : BinaryLoadStore<block_or1,  load, vt, oc, length>;
 142:   defm : BinaryLoadStore<block_or2,  load, vt, oc, length>;
 143:   defm : BinaryLoadStore<block_xor1, load, vt, xc, length>;
 144:   defm : BinaryLoadStore<block_xor2, load, vt, xc, length>;
```
- **EN**: This block declares or refines TableGen records such as `BinaryLoadStore`, `BlockLoadStore`.
- **CN**: 该代码块声明或细化了 `BinaryLoadStore`, `BlockLoadStore` 等 TableGen 记录。

### Lines 145-162 / 第 145-162 行
```tablegen
 145: }
 146: 
 147: // Record that INSN is a LOAD AND TEST that can be used to compare
 148: // registers in CLS against zero.
 149: multiclass CompareZeroFP<Instruction insn, RegisterOperand cls> {
 150:   def : Pat<(z_any_fcmp cls:$reg, (fpimm0)), (insn cls:$reg)>;
 151:   // The sign of the zero makes no difference.
 152:   def : Pat<(z_any_fcmp cls:$reg, (fpimmneg0)), (insn cls:$reg)>;
 153: }
 154: 
 155: // Use INSN for performing binary operation OPERATION of type VT
 156: // on registers of class CLS.
 157: class BinaryRRWithType<Instruction insn, RegisterOperand cls,
 158:                        SDPatternOperator operator, ValueType vt>
 159:   : Pat<(vt (operator cls:$x, cls:$y)), (insn cls:$x, cls:$y)>;
 160: 
 161: // Use INSN to perform conversion operation OPERATOR, with the input being
 162: // TR2 and the output being TR1.  SUPPRESS is 4 to suppress inexact conditions
```
- **EN**: This block declares or refines TableGen records such as `CompareZeroFP`, `BinaryRRWithType`.
- **CN**: 该代码块声明或细化了 `CompareZeroFP`, `BinaryRRWithType` 等 TableGen 记录。

### Lines 163-174 / 第 163-174 行
```tablegen
 163: // and 0 to allow them.  MODE is the rounding mode to use.
 164: class FPConversion<Instruction insn, SDPatternOperator operator, TypedReg tr1,
 165:                    TypedReg tr2, bits<3> suppress, bits<4> mode>
 166:   : Pat<(tr1.vt (operator (tr2.vt tr2.op:$vec))),
 167:         (insn tr2.op:$vec, suppress, mode)>;
 168: 
 169: // Use INSN to perform minimum/maximum operation OPERATOR on type TR.
 170: // FUNCTION is the type of minimum/maximum function to perform.
 171: class FPMinMax<Instruction insn, SDPatternOperator operator, TypedReg tr,
 172:                bits<4> function>
 173:   : Pat<(tr.vt (operator (tr.vt tr.op:$vec1), (tr.vt tr.op:$vec2))),
 174:         (insn tr.op:$vec1, tr.op:$vec2, function)>;
```
- **EN**: This block declares or refines TableGen records such as `FPConversion`, `FPMinMax`.
- **CN**: 该代码块声明或细化了 `FPConversion`, `FPMinMax` 等 TableGen 记录。

## Key Concepts / 关键概念
- **TableGen DSL**: Describes backend metadata with declarative records and multiclasses. / 使用声明式记录和 multiclass 描述后端元数据。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。

## Dependencies / 依赖关系
- No explicit include directives were found; dependencies are primarily implicit through LLVM build structure or generated files. / 未发现显式 include 指令；依赖主要通过 LLVM 构建结构或生成文件隐式提供。
