# SparcInstr64Bit.td — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/Sparc/SparcInstr64Bit.td`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Defines backend metadata in TableGen DSL that is later expanded into generated tables and matcher code.
  - **CN**: 使用 TableGen DSL 定义后端元数据，随后会展开为生成的表和匹配代码。
- **Note / 说明**:
  - **EN**: This file uses LLVM TableGen DSL, so many definitions are declarative records rather than executable C++ statements.
  - **CN**: 该文件使用 LLVM TableGen DSL，因此许多内容是声明式记录，而不是可执行的 C++ 语句。

## Line-by-Line Analysis / 逐行分析

### Lines 1-15
```tablegen
//===-- SparcInstr64Bit.td - 64-bit instructions for Sparc Target ---------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains instruction definitions and patterns needed for 64-bit
// code generation on SPARC v9.
//
// Some SPARC v9 instructions are defined in SparcInstrInfo.td because they can
// also be used in 32-bit code running on a SPARC v9 CPU.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 16-25
```tablegen

let Predicates = [Is64Bit] in {
// The same integer registers are used for i32 and i64 values.
// When registers hold i32 values, the high bits are don't care.
// This give us free trunc and anyext.
def : Pat<(i64 (anyext i32:$val)), (COPY_TO_REGCLASS $val, I64Regs)>;
def : Pat<(i32 (trunc i64:$val)), (COPY_TO_REGCLASS $val, IntRegs)>;

} // Predicates = [Is64Bit]

```
- **EN**: Implements logic around `Pat<`.
- **CN**: 围绕 `Pat<` 实现具体逻辑。

### Lines 26-36
```tablegen

//===----------------------------------------------------------------------===//
// 64-bit Shift Instructions.
//===----------------------------------------------------------------------===//
//
// The 32-bit shift instructions are still available. The left shift srl
// instructions shift all 64 bits, but it only accepts a 5-bit shift amount.
//
// The srl instructions only shift the low 32 bits and clear the high 32 bits.
// Finally, sra shifts the low 32 bits and sign-extends to 64 bits.

```
- **EN**: Documents the next group of declarations or explains a target-specific rule.
- **CN**: 为接下来的声明分组提供说明，或解释目标相关规则。

### Lines 37-46
```tablegen
let Predicates = [Is64Bit] in {

def : Pat<(i64 (zext i32:$val)), (SRLri $val, 0)>;
def : Pat<(i64 (sext i32:$val)), (SRAri $val, 0)>;

def : Pat<(i64 (and i64:$val, 0xffffffff)), (SRLri $val, 0)>;
def : Pat<(i64 (sext_inreg i64:$val, i32)), (SRAri $val, 0)>;

} // Predicates = [Is64Bit]

```
- **EN**: Implements logic around `Pat<`.
- **CN**: 围绕 `Pat<` 实现具体逻辑。

### Lines 47-55
```tablegen
let Predicates = [HasV9] in {

defm SLLX : F3_S<"sllx", 0b100101, 1, shl, i64, shift_imm6, I64Regs>;
defm SRLX : F3_S<"srlx", 0b100110, 1, srl, i64, shift_imm6, I64Regs>;
defm SRAX : F3_S<"srax", 0b100111, 1, sra, i64, shift_imm6, I64Regs>;

} // Predicates = [HasV9]

//===----------------------------------------------------------------------===//
```
- **EN**: Declares TableGen records such as `SLLX`, `SRLX`, `SRAX`; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `SLLX`, `SRLX`, `SRAX` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

### Lines 56-65
```tablegen
// 64-bit Immediates.
//===----------------------------------------------------------------------===//
//
// All 32-bit immediates can be materialized with sethi+or, but 64-bit
// immediates may require more code. There may be a point where it is
// preferable to use a constant pool load instead, depending on the
// microarchitecture.

// Single-instruction patterns.

```
- **EN**: Documents the next group of declarations or explains a target-specific rule.
- **CN**: 为接下来的声明分组提供说明，或解释目标相关规则。

### Lines 66-77
```tablegen
// Zero immediate.
def : Pat<(i64 0), (COPY (i64 G0))>,
  Requires<[Is64Bit]>;

// The ALU instructions want their simm13 operands as i32 immediates.
// FIXME: This is no longer true, they are now pointer-sized.
def as_i32imm : SDNodeXForm<imm, [{
  return CurDAG->getSignedTargetConstant(N->getSExtValue(), SDLoc(N), MVT::i32);
}]>;
def : Pat<(i64 simm13:$val), (ORri (i64 G0), (as_i32imm $val))>;
def : Pat<(i64 SETHIimm:$val), (SETHIi (HI22 $val))>;

```
- **EN**: Declares TableGen records such as `as_i32imm` for the backend description.
- **CN**: 为后端描述声明了 `as_i32imm` 等 TableGen 记录。

### Lines 78-95
```tablegen
// Double-instruction patterns.

// All unsigned i32 immediates can be handled by sethi+or.
def uimm32 : PatLeaf<(imm), [{ return isUInt<32>(N->getZExtValue()); }]>;
def : Pat<(i64 uimm32:$val), (ORri (SETHIi (HI22 $val)), (LO10 $val))>,
      Requires<[Is64Bit]>;

// All negative i33 immediates can be handled by sethi+xor.
def nimm33 : PatLeaf<(imm), [{
  int64_t Imm = N->getSExtValue();
  return Imm < 0 && isInt<33>(Imm);
}]>;
// Bits 10-31 inverted. Same as assembler's %hix.
def HIX22 : SDNodeXForm<imm, [{
  uint64_t Val = (~N->getZExtValue() >> 10) & ((1u << 22) - 1);
  return CurDAG->getTargetConstant(Val, SDLoc(N), MVT::i32);
}]>;
// Bits 0-9 with ones in bits 10-31. Same as assembler's %lox.
```
- **EN**: Declares TableGen records such as `uimm32`, `nimm33`, `HIX22`; this block attaches SelectionDAG patterns.
- **CN**: 声明了 `uimm32`, `nimm33`, `HIX22` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式。

### Lines 96-113
```tablegen
def LOX10 : SDNodeXForm<imm, [{
  return CurDAG->getSignedTargetConstant(~(~N->getZExtValue() & 0x3ff),
                                         SDLoc(N), MVT::i32);
}]>;
def : Pat<(i64 nimm33:$val), (XORri (SETHIi (HIX22 $val)), (LOX10 $val))>,
      Requires<[Is64Bit]>;

// More possible patterns:
//
//   (sllx sethi, n)
//   (sllx simm13, n)
//
// 3 instrs:
//
//   (xor (sllx sethi), simm13)
//   (sllx (xor sethi, simm13))
//
// 4 instrs:
```
- **EN**: Declares TableGen records such as `LOX10`; this block attaches SelectionDAG patterns.
- **CN**: 声明了 `LOX10` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式。

### Lines 114-128
```tablegen
//
//   (or sethi, (sllx sethi))
//   (xnor sethi, (sllx sethi))
//
// 5 instrs:
//
//   (or (sllx sethi), (or sethi, simm13))
//   (xnor (sllx sethi), (or sethi, simm13))
//   (or (sllx sethi), (sllx sethi))
//   (xnor (sllx sethi), (sllx sethi))
//
// Worst case is 6 instrs:
//
//   (or (sllx (or sethi, simmm13)), (or sethi, simm13))

```
- **EN**: Documents the next group of declarations or explains a target-specific rule.
- **CN**: 为接下来的声明分组提供说明，或解释目标相关规则。

### Lines 129-143
```tablegen
// Bits 42-63, same as assembler's %hh.
def HH22 : SDNodeXForm<imm, [{
  uint64_t Val = (N->getZExtValue() >> 42) & ((1u << 22) - 1);
  return CurDAG->getTargetConstant(Val, SDLoc(N), MVT::i32);
}]>;
// Bits 32-41, same as assembler's %hm.
def HM10 : SDNodeXForm<imm, [{
  uint64_t Val = (N->getZExtValue() >> 32) & ((1u << 10) - 1);
  return CurDAG->getTargetConstant(Val, SDLoc(N), MVT::i32);
}]>;
def : Pat<(i64 imm:$val),
          (ORrr (SLLXri (ORri (SETHIi (HH22 $val)), (HM10 $val)), (i32 32)),
                (ORri (SETHIi (HI22 $val)), (LO10 $val)))>,
      Requires<[Is64Bit]>;

```
- **EN**: Declares TableGen records such as `HH22`, `HM10` for the backend description.
- **CN**: 为后端描述声明了 `HH22`, `HM10` 等 TableGen 记录。

### Lines 144-154
```tablegen

//===----------------------------------------------------------------------===//
// 64-bit Integer Arithmetic and Logic.
//===----------------------------------------------------------------------===//

let Predicates = [Is64Bit] in {

def : Pat<(and i64:$lhs, i64:$rhs), (ANDrr $lhs, $rhs)>;
def : Pat<(or  i64:$lhs, i64:$rhs), (ORrr  $lhs, $rhs)>;
def : Pat<(xor i64:$lhs, i64:$rhs), (XORrr $lhs, $rhs)>;

```
- **EN**: Implements logic around `Pat<`.
- **CN**: 围绕 `Pat<` 实现具体逻辑。

### Lines 155-166
```tablegen
def : Pat<(and i64:$lhs, (i64 simm13:$rhs)), (ANDri $lhs, imm:$rhs)>;
def : Pat<(or  i64:$lhs, (i64 simm13:$rhs)), (ORri  $lhs, imm:$rhs)>;
def : Pat<(xor i64:$lhs, (i64 simm13:$rhs)), (XORri $lhs, imm:$rhs)>;

def : Pat<(and i64:$lhs, (not i64:$rhs)), (ANDNrr $lhs, $rhs)>;
def : Pat<(or  i64:$lhs, (not i64:$rhs)), (ORNrr  $lhs, $rhs)>;
def : Pat<(not (xor i64:$lhs, i64:$rhs)), (XNORrr $lhs, $rhs)>;

def : Pat<(addc i64:$lhs, i64:$rhs), (ADDCCrr $lhs, $rhs)>, Requires<[HasVIS3]>;
def : Pat<(add i64:$lhs, i64:$rhs), (ADDrr $lhs, $rhs)>;
def : Pat<(sub i64:$lhs, i64:$rhs), (SUBrr $lhs, $rhs)>;

```
- **EN**: Implements logic around `Pat<`.
- **CN**: 围绕 `Pat<` 实现具体逻辑。

### Lines 167-177
```tablegen
def : Pat<(addc i64:$lhs, (i64 simm13:$rhs)), (ADDCCri $lhs, imm:$rhs)>, Requires<[HasVIS3]>;
def : Pat<(add i64:$lhs, (i64 simm13:$rhs)), (ADDri $lhs, imm:$rhs)>;
def : Pat<(sub i64:$lhs, (i64 simm13:$rhs)), (SUBri $lhs, imm:$rhs)>;

def : Pat<(tlsadd i64:$rs1, i64:$rs2, tglobaltlsaddr:$sym),
          (TLS_ADDrr $rs1, $rs2, $sym)>;

def : Pat<(SPcmpicc i64:$lhs, i64:$rhs), (SUBCCrr $lhs, $rhs)>;
def : Pat<(SPcmpicc i64:$lhs, (i64 simm13:$rhs)), (SUBCCri $lhs, imm:$rhs)>;
def : Pat<(i64 (ctpop i64:$src)), (POPCrr $src)>;

```
- **EN**: Implements logic around `Pat<`.
- **CN**: 围绕 `Pat<` 实现具体逻辑。

### Lines 178-187
```tablegen
} // Predicates = [Is64Bit]


//===----------------------------------------------------------------------===//
// 64-bit Integer Multiply and Divide.
//===----------------------------------------------------------------------===//

let Predicates = [HasV9] in {
defm MULX    : F3_12<"mulx", 0b001001, mul, I64Regs, i64, i64imm>;

```
- **EN**: Declares TableGen records such as `MULX`; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `MULX` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

### Lines 188-196
```tablegen
// Division can trap.
let hasSideEffects = 1 in {
defm SDIVX    : F3_12<"sdivx", 0b101101, sdiv, I64Regs, i64, i64imm>;
defm UDIVX    : F3_12<"udivx", 0b001101, udiv, I64Regs, i64, i64imm>;
} // hasSideEffects = 1
} // Predicates = [HasV9]


//===----------------------------------------------------------------------===//
```
- **EN**: Declares TableGen records such as `SDIVX`, `UDIVX`; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `SDIVX`, `UDIVX` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

### Lines 197-206
```tablegen
// 64-bit Loads and Stores.
//===----------------------------------------------------------------------===//
//
// All the 32-bit loads and stores are available. The extending loads are sign
// or zero-extending to 64 bits. The LDrr and LDri instructions load 32 bits
// zero-extended to i64. Their mnemonic is lduw in SPARC v9 (Load Unsigned
// Word).
//
// SPARC v9 adds 64-bit loads as well as a sign-extending ldsw i32 loads.

```
- **EN**: Documents the next group of declarations or explains a target-specific rule.
- **CN**: 为接下来的声明分组提供说明，或解释目标相关规则。

### Lines 207-224
```tablegen
let Predicates = [HasV9] in {

// 64-bit loads.
defm LDX   : LoadA<"ldx", 0b001011, 0b011011, load, I64Regs, i64>;

let mayLoad = 1, isAsmParserOnly = 1 in {
  def TLS_LDXrr : F3_1<3, 0b001011,
                       (outs IntRegs:$rd),
                       (ins (MEMrr $rs1, $rs2):$addr, TailRelocSymTLSLoad:$sym),
                       "ldx [$addr], $rd, $sym",
                       [(set i64:$rd,
                           (tlsld ADDRrr:$addr, tglobaltlsaddr:$sym))]>;
  def GDOP_LDXrr : F3_1<3, 0b001011,
                       (outs I64Regs:$rd),
                       (ins (MEMrr $rs1, $rs2):$addr, TailRelocSymGOTLoad:$sym),
                       "ldx [$addr], $rd, $sym",
                       [(set i64:$rd,
                           (load_gdop ADDRrr:$addr, tglobaladdr:$sym))]>;
```
- **EN**: Declares TableGen records such as `LDX`, `TLS_LDXrr`, `GDOP_LDXrr`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments.
- **CN**: 声明了 `LDX`, `TLS_LDXrr`, `GDOP_LDXrr` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段。

### Lines 225-234
```tablegen
}

// Sign-extending load of i32 into i64 is a new SPARC v9 instruction.
defm LDSW   : LoadA<"ldsw", 0b001000, 0b011000, sextloadi32, I64Regs, i64>;

// 64-bit stores.
defm STX    : StoreA<"stx", 0b001110, 0b011110, store, I64Regs, i64>;

} // Predicates = [HasV9]

```
- **EN**: Declares TableGen records such as `LDSW`, `STX` for the backend description.
- **CN**: 为后端描述声明了 `LDSW`, `STX` 等 TableGen 记录。

### Lines 235-249
```tablegen
let Predicates = [Is64Bit] in {

// Extending loads to i64.
def : Pat<(i64 (zextloadi1 ADDRrr:$addr)), (LDUBrr ADDRrr:$addr)>;
def : Pat<(i64 (zextloadi1 ADDRri:$addr)), (LDUBri ADDRri:$addr)>;
def : Pat<(i64 (extloadi1 ADDRrr:$addr)), (LDUBrr ADDRrr:$addr)>;
def : Pat<(i64 (extloadi1 ADDRri:$addr)), (LDUBri ADDRri:$addr)>;

def : Pat<(i64 (zextloadi8 ADDRrr:$addr)), (LDUBrr ADDRrr:$addr)>;
def : Pat<(i64 (zextloadi8 ADDRri:$addr)), (LDUBri ADDRri:$addr)>;
def : Pat<(i64 (extloadi8 ADDRrr:$addr)),  (LDUBrr ADDRrr:$addr)>;
def : Pat<(i64 (extloadi8 ADDRri:$addr)),  (LDUBri ADDRri:$addr)>;
def : Pat<(i64 (sextloadi8 ADDRrr:$addr)), (LDSBrr ADDRrr:$addr)>;
def : Pat<(i64 (sextloadi8 ADDRri:$addr)), (LDSBri ADDRri:$addr)>;

```
- **EN**: Implements logic around `Pat<`.
- **CN**: 围绕 `Pat<` 实现具体逻辑。

### Lines 250-261
```tablegen
def : Pat<(i64 (zextloadi16 ADDRrr:$addr)), (LDUHrr ADDRrr:$addr)>;
def : Pat<(i64 (zextloadi16 ADDRri:$addr)), (LDUHri ADDRri:$addr)>;
def : Pat<(i64 (extloadi16 ADDRrr:$addr)),  (LDUHrr ADDRrr:$addr)>;
def : Pat<(i64 (extloadi16 ADDRri:$addr)),  (LDUHri ADDRri:$addr)>;
def : Pat<(i64 (sextloadi16 ADDRrr:$addr)), (LDSHrr ADDRrr:$addr)>;
def : Pat<(i64 (sextloadi16 ADDRri:$addr)), (LDSHri ADDRri:$addr)>;

def : Pat<(i64 (zextloadi32 ADDRrr:$addr)), (LDrr ADDRrr:$addr)>;
def : Pat<(i64 (zextloadi32 ADDRri:$addr)), (LDri ADDRri:$addr)>;
def : Pat<(i64 (extloadi32 ADDRrr:$addr)),  (LDrr ADDRrr:$addr)>;
def : Pat<(i64 (extloadi32 ADDRri:$addr)),  (LDri ADDRri:$addr)>;

```
- **EN**: Implements logic around `Pat<`.
- **CN**: 围绕 `Pat<` 实现具体逻辑。

### Lines 262-273
```tablegen
// Truncating stores from i64 are identical to the i32 stores.
def : Pat<(truncstorei8  i64:$src, ADDRrr:$addr), (STBrr ADDRrr:$addr, $src)>;
def : Pat<(truncstorei8  i64:$src, ADDRri:$addr), (STBri ADDRri:$addr, $src)>;
def : Pat<(truncstorei16 i64:$src, ADDRrr:$addr), (STHrr ADDRrr:$addr, $src)>;
def : Pat<(truncstorei16 i64:$src, ADDRri:$addr), (STHri ADDRri:$addr, $src)>;
def : Pat<(truncstorei32 i64:$src, ADDRrr:$addr), (STrr  ADDRrr:$addr, $src)>;
def : Pat<(truncstorei32 i64:$src, ADDRri:$addr), (STri  ADDRri:$addr, $src)>;

// store 0, addr -> store %g0, addr
def : Pat<(store (i64 0), ADDRrr:$dst), (STXrr ADDRrr:$dst, (i64 G0))>;
def : Pat<(store (i64 0), ADDRri:$dst), (STXri ADDRri:$dst, (i64 G0))>;

```
- **EN**: Implements logic around `Pat<`.
- **CN**: 围绕 `Pat<` 实现具体逻辑。

### Lines 274-288
```tablegen
} // Predicates = [Is64Bit]


//===----------------------------------------------------------------------===//
// 64-bit Conditionals.
//===----------------------------------------------------------------------===//

//
// Flag-setting instructions like subcc and addcc set both icc and xcc flags.
// The icc flags correspond to the 32-bit result, and the xcc are for the
// full 64-bit result.
//
// We reuse CMPICC SDNodes for compares, but use new BPXCC branch nodes for
// 64-bit compares. See LowerBR_CC.

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 289-306
```tablegen
let Predicates = [Is64Bit] in {

let Uses = [ICC], cc = 0b10 in
  defm BPX : IPredBranch<"%xcc", [(SPbpxcc bb:$imm19, imm:$cond)]>;

// Conditional moves on %xcc.
let Uses = [ICC], Constraints = "$f = $rd" in {
let intcc = 1, cc = 0b10 in {
def MOVXCCrr : F4_1<0b101100, (outs IntRegs:$rd),
                      (ins IntRegs:$rs2, IntRegs:$f, CCOp:$cond),
                      "mov$cond %xcc, $rs2, $rd",
                      [(set i32:$rd,
                       (SPselectxcc i32:$rs2, i32:$f, imm:$cond))]>;
def MOVXCCri : F4_2<0b101100, (outs IntRegs:$rd),
                      (ins i32imm:$simm11, IntRegs:$f, CCOp:$cond),
                      "mov$cond %xcc, $simm11, $rd",
                      [(set i32:$rd,
                       (SPselectxcc simm11:$simm11, i32:$f, imm:$cond))]>;
```
- **EN**: Declares TableGen records such as `BPX`, `MOVXCCrr`, `MOVXCCri`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments.
- **CN**: 声明了 `BPX`, `MOVXCCrr`, `MOVXCCri` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段。

### Lines 307-324
```tablegen
} // cc

let intcc = 1, opf_cc = 0b10 in {
def FMOVS_XCC : F4_3<0b110101, 0b000001, (outs FPRegs:$rd),
                      (ins FPRegs:$rs2, FPRegs:$f, CCOp:$cond),
                      "fmovs$cond %xcc, $rs2, $rd",
                      [(set f32:$rd,
                       (SPselectxcc f32:$rs2, f32:$f, imm:$cond))]>;
def FMOVD_XCC : F4_3<0b110101, 0b000010, (outs DFPRegs:$rd),
                      (ins DFPRegs:$rs2, DFPRegs:$f, CCOp:$cond),
                      "fmovd$cond %xcc, $rs2, $rd",
                      [(set f64:$rd,
                       (SPselectxcc f64:$rs2, f64:$f, imm:$cond))]>;
let append Predicates = [HasHardQuad] in
def FMOVQ_XCC : F4_3<0b110101, 0b000011, (outs QFPRegs:$rd),
                      (ins QFPRegs:$rs2, QFPRegs:$f, CCOp:$cond),
                      "fmovq$cond %xcc, $rs2, $rd",
                      [(set f128:$rd,
```
- **EN**: Declares TableGen records such as `FMOVS_XCC`, `FMOVD_XCC`, `FMOVQ_XCC`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments.
- **CN**: 声明了 `FMOVS_XCC`, `FMOVD_XCC`, `FMOVQ_XCC` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段。

### Lines 325-342
```tablegen
                       (SPselectxcc f128:$rs2, f128:$f, imm:$cond))]>;
} // opf_cc
} // Uses, Constraints

} // Predicates = [Is64Bit]

// Branch On integer register with Prediction (BPr).
let isBranch = 1, isTerminator = 1, hasDelaySlot = 1 in
multiclass BranchOnReg<list<dag> CCPattern> {
  def R    : F2_4<0, 1, (outs), (ins bprtarget16:$imm16, RegCCOp:$rcond, I64Regs:$rs1),
             "br$rcond $rs1, $imm16", CCPattern>;
  def RA   : F2_4<1, 1, (outs), (ins bprtarget16:$imm16, RegCCOp:$rcond, I64Regs:$rs1),
             "br$rcond,a $rs1, $imm16", []>;
  def RNT  : F2_4<0, 0, (outs), (ins bprtarget16:$imm16, RegCCOp:$rcond, I64Regs:$rs1),
             "br$rcond,pn $rs1, $imm16", []>;
  def RANT : F2_4<1, 0, (outs), (ins bprtarget16:$imm16, RegCCOp:$rcond, I64Regs:$rs1),
             "br$rcond,a,pn $rs1, $imm16", []>;
}
```
- **EN**: Declares TableGen records such as `BranchOnReg`, `R`, `RA`, `RNT`, ...; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments.
- **CN**: 声明了 `BranchOnReg`, `R`, `RA`, `RNT`, ... 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段。

### Lines 343-353
```tablegen

multiclass bpr_alias<string OpcStr, Instruction NAPT, Instruction APT> {
  def : InstAlias<!strconcat(OpcStr, ",pt $rs1, $imm16"),
                  (NAPT I64Regs:$rs1, bprtarget16:$imm16), 0>;
  def : InstAlias<!strconcat(OpcStr, ",a,pt $rs1, $imm16"),
                  (APT I64Regs:$rs1, bprtarget16:$imm16), 0>;
}

let Predicates = [HasV9] in
  defm BP : BranchOnReg<[(SPbrreg bb:$imm16, imm:$rcond, i64:$rs1)]>;

```
- **EN**: Declares TableGen records such as `bpr_alias`, `BP`; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `bpr_alias`, `BP` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

### Lines 354-366
```tablegen
// Move integer register on register condition (MOVr).
let Predicates = [HasV9], Constraints = "$f = $rd" in {
  def MOVRrr : F4_4r<0b101111, 0b00000, (outs IntRegs:$rd),
                   (ins I64Regs:$rs1, IntRegs:$rs2, IntRegs:$f, RegCCOp:$rcond),
                   "movr$rcond $rs1, $rs2, $rd",
                   [(set i32:$rd, (SPselectreg i32:$rs2, i32:$f, imm:$rcond, i64:$rs1))]>;

  def MOVRri : F4_4i<0b101111, (outs IntRegs:$rd),
                   (ins I64Regs:$rs1, i32imm:$simm10, IntRegs:$f, RegCCOp:$rcond),
                   "movr$rcond $rs1, $simm10, $rd",
                   [(set i32:$rd, (SPselectreg simm10:$simm10, i32:$f, imm:$rcond, i64:$rs1))]>;
}

```
- **EN**: Declares TableGen records such as `MOVRrr`, `MOVRri`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments.
- **CN**: 声明了 `MOVRrr`, `MOVRri` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段。

### Lines 367-383
```tablegen
// Move FP register on integer register condition (FMOVr).
let Predicates = [HasV9], Constraints = "$f = $rd" in {
  def FMOVRS : F4_4r<0b110101, 0b00101,
                (outs FPRegs:$rd), (ins I64Regs:$rs1, FPRegs:$rs2, FPRegs:$f,  RegCCOp:$rcond),
                "fmovrs$rcond $rs1, $rs2, $rd",
                [(set f32:$rd, (SPselectreg f32:$rs2, f32:$f, imm:$rcond, i64:$rs1))]>;
  def FMOVRD : F4_4r<0b110101, 0b00110,
                (outs DFPRegs:$rd), (ins I64Regs:$rs1, DFPRegs:$rs2, DFPRegs:$f, RegCCOp:$rcond),
                "fmovrd$rcond $rs1, $rs2, $rd",
                [(set f64:$rd, (SPselectreg f64:$rs2, f64:$f, imm:$rcond, i64:$rs1))]>;
  let append Predicates = [HasHardQuad] in
  def FMOVRQ : F4_4r<0b110101, 0b00111,
                (outs QFPRegs:$rd), (ins I64Regs:$rs1, QFPRegs:$rs2, QFPRegs:$f, RegCCOp:$rcond),
                "fmovrq$rcond $rs1, $rs2, $rd",
                [(set f128:$rd, (SPselectreg f128:$rs2, f128:$f, imm:$rcond, i64:$rs1))]>;
}

```
- **EN**: Declares TableGen records such as `FMOVRS`, `FMOVRD`, `FMOVRQ`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments.
- **CN**: 声明了 `FMOVRS`, `FMOVRD`, `FMOVRQ` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段。

### Lines 384-401
```tablegen
//===----------------------------------------------------------------------===//
// 64-bit Floating Point Conversions.
//===----------------------------------------------------------------------===//

let Predicates = [HasV9] in {

def FXTOS : F3_3u<2, 0b110100, 0b010000100,
                 (outs FPRegs:$rd), (ins DFPRegs:$rs2),
                 "fxtos $rs2, $rd",
                 [(set FPRegs:$rd, (SPxtof DFPRegs:$rs2))]>;
def FXTOD : F3_3u<2, 0b110100, 0b010001000,
                 (outs DFPRegs:$rd), (ins DFPRegs:$rs2),
                 "fxtod $rs2, $rd",
                 [(set DFPRegs:$rd, (SPxtof DFPRegs:$rs2))]>;
let append Predicates = [HasHardQuad] in
def FXTOQ : F3_3u<2, 0b110100, 0b010001100,
                 (outs QFPRegs:$rd), (ins DFPRegs:$rs2),
                 "fxtoq $rs2, $rd",
```
- **EN**: Declares TableGen records such as `FXTOS`, `FXTOD`, `FXTOQ`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments.
- **CN**: 声明了 `FXTOS`, `FXTOD`, `FXTOQ` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段。

### Lines 402-417
```tablegen
                 [(set QFPRegs:$rd, (SPxtof DFPRegs:$rs2))]>;

def FSTOX : F3_3u<2, 0b110100, 0b010000001,
                 (outs DFPRegs:$rd), (ins FPRegs:$rs2),
                 "fstox $rs2, $rd",
                 [(set DFPRegs:$rd, (SPftox FPRegs:$rs2))]>;
def FDTOX : F3_3u<2, 0b110100, 0b010000010,
                 (outs DFPRegs:$rd), (ins DFPRegs:$rs2),
                 "fdtox $rs2, $rd",
                 [(set DFPRegs:$rd, (SPftox DFPRegs:$rs2))]>;
let append Predicates = [HasHardQuad] in
def FQTOX : F3_3u<2, 0b110100, 0b010000011,
                 (outs DFPRegs:$rd), (ins QFPRegs:$rs2),
                 "fqtox $rs2, $rd",
                 [(set DFPRegs:$rd, (SPftox QFPRegs:$rs2))]>;

```
- **EN**: Declares TableGen records such as `FSTOX`, `FDTOX`, `FQTOX`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments.
- **CN**: 声明了 `FSTOX`, `FDTOX`, `FQTOX` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段。

### Lines 418-426
```tablegen
} // Predicates = [HasV9]

// Conditional move patterns.
let Predicates = [Is64Bit] in {
def : Pat<(SPselectxcc i64:$t, i64:$f, imm:$cond),
          (MOVXCCrr $t, $f, imm:$cond)>;
def : Pat<(SPselectxcc (i64 simm11:$t), i64:$f, imm:$cond),
          (MOVXCCri (as_i32imm $t), $f, imm:$cond)>;

```
- **EN**: Implements logic around `Pat<`, `MOVXCCri`.
- **CN**: 围绕 `Pat<`, `MOVXCCri` 实现具体逻辑。

### Lines 427-436
```tablegen
def : Pat<(SPselecticc i64:$t, i64:$f, imm:$cond),
          (MOVICCrr $t, $f, imm:$cond)>;
def : Pat<(SPselecticc (i64 simm11:$t), i64:$f, imm:$cond),
          (MOVICCri (as_i32imm $t), $f, imm:$cond)>;

def : Pat<(SPselectfcc i64:$t, i64:$f, imm:$cond),
          (MOVFCCrr $t, $f, imm:$cond)>;
def : Pat<(SPselectfcc (i64 simm11:$t), i64:$f, imm:$cond),
          (MOVFCCri (as_i32imm $t), $f, imm:$cond)>;

```
- **EN**: Implements logic around `Pat<`, `MOVICCri`, `MOVFCCri`.
- **CN**: 围绕 `Pat<`, `MOVICCri`, `MOVFCCri` 实现具体逻辑。

### Lines 437-450
```tablegen
def : Pat<(SPselectreg i64:$t, i64:$f, imm:$rcond, i64:$rs1),
          (MOVRrr $rs1, $t, $f, imm:$rcond)>;
def : Pat<(SPselectreg (i64 simm10:$t), i64:$f, imm:$rcond, i64:$rs1),
          (MOVRri $rs1, (as_i32imm $t), $f, imm:$rcond)>;
} // Predicates = [Is64Bit]

// ATOMICS.
let Predicates = [HasV9], Constraints = "$swap = $rd" in {
  def CASXArr: F3_1_asi<3, 0b111110,
                (outs I64Regs:$rd), (ins I64Regs:$rs1, I64Regs:$rs2,
                                     I64Regs:$swap, ASITag:$asi),
                 "casxa [$rs1] $asi, $rs2, $rd",
                 []>;

```
- **EN**: Declares TableGen records such as `CASXArr`; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `CASXArr` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

### Lines 451-460
```tablegen
  let Uses = [ASR3] in
    def CASXAri: F3_1_cas_asi<3, 0b111110,
                (outs I64Regs:$rd), (ins I64Regs:$rs1, I64Regs:$rs2,
                                     I64Regs:$swap),
                 "casxa [$rs1] %asi, $rs2, $rd",
                 []>;
} // Predicates = [Is64Bit], Constraints = ...

let Predicates = [Is64Bit] in {

```
- **EN**: Declares TableGen records such as `CASXAri`; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `CASXAri` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

### Lines 461-471
```tablegen
// atomic_load_nonext_64 addr -> load addr
def : Pat<(i64 (atomic_load_nonext_64 ADDRrr:$src)), (LDXrr ADDRrr:$src)>;
def : Pat<(i64 (atomic_load_nonext_64 ADDRri:$src)), (LDXri ADDRri:$src)>;

// atomic_store_64 val, addr -> store val, addr
def : Pat<(atomic_store_64 i64:$val, ADDRrr:$dst), (STXrr ADDRrr:$dst, $val)>;
def : Pat<(atomic_store_64 i64:$val, ADDRri:$dst), (STXri ADDRri:$dst, $val)>;

def : Pat<(atomic_cmp_swap_i64 i64:$rs1, i64:$rs2, i64:$swap),
          (CASXArr $rs1, $rs2, $swap, 0x80)>;

```
- **EN**: Implements logic around `Pat<`.
- **CN**: 围绕 `Pat<` 实现具体逻辑。

### Lines 472-484
```tablegen
} // Predicates = [Is64Bit]

let Predicates = [Is64Bit], hasSideEffects = 1, Uses = [ICC], cc = 0b10 in
 defm TXCC : TRAP<"%xcc">;

// Global addresses, constant pool entries
let Predicates = [Is64Bit] in {

def : Pat<(SPhi tglobaladdr:$in), (SETHIi tglobaladdr:$in)>;
def : Pat<(SPlo tglobaladdr:$in), (ORri (i64 G0), tglobaladdr:$in)>;
def : Pat<(SPhi tconstpool:$in), (SETHIi tconstpool:$in)>;
def : Pat<(SPlo tconstpool:$in), (ORri (i64 G0), tconstpool:$in)>;

```
- **EN**: Declares TableGen records such as `TXCC`; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `TXCC` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

### Lines 485-496
```tablegen
// GlobalTLS addresses
def : Pat<(SPhi tglobaltlsaddr:$in), (SETHIi tglobaltlsaddr:$in)>;
def : Pat<(SPlo tglobaltlsaddr:$in), (ORri (i64 G0), tglobaltlsaddr:$in)>;
def : Pat<(add (SPhi tglobaltlsaddr:$in1), (SPlo tglobaltlsaddr:$in2)),
          (ADDri (SETHIi tglobaltlsaddr:$in1), (tglobaltlsaddr:$in2))>;
def : Pat<(xor (SPhi tglobaltlsaddr:$in1), (SPlo tglobaltlsaddr:$in2)),
          (XORri  (SETHIi tglobaltlsaddr:$in1), (tglobaltlsaddr:$in2))>;

// Blockaddress
def : Pat<(SPhi tblockaddress:$in), (SETHIi tblockaddress:$in)>;
def : Pat<(SPlo tblockaddress:$in), (ORri (i64 G0), tblockaddress:$in)>;

```
- **EN**: Implements logic around `Pat<`, `ADDri`, `XORri`.
- **CN**: 围绕 `Pat<`, `ADDri`, `XORri` 实现具体逻辑。

### Lines 497-502
```tablegen
// Add reg, lo.  This is used when taking the addr of a global/constpool entry.
def : Pat<(add iPTR:$r, (SPlo tglobaladdr:$in)), (ADDri $r, tglobaladdr:$in)>;
def : Pat<(add iPTR:$r, (SPlo tconstpool:$in)),  (ADDri $r, tconstpool:$in)>;
def : Pat<(add iPTR:$r, (SPlo tblockaddress:$in)),
                        (ADDri $r, tblockaddress:$in)>;
}
```
- **EN**: Implements logic around `Pat<`.
- **CN**: 围绕 `Pat<` 实现具体逻辑。

## Key Concepts / 关键概念

- **TableGen DSL / TableGen 领域专用语言**:
  - **EN**: Declarative backend description expanded into generated tables
  - **CN**: 以声明式方式描述后端并生成表格代码
- **Fixups and relocations / Fixup 与重定位**:
  - **EN**: Bridges symbolic references to concrete relocation records
  - **CN**: 把符号引用连接到具体重定位记录
- **Generated backend tables / 生成的后端表**:
  - **EN**: The records here feed TableGen generators that emit `.inc` matcher/data tables
  - **CN**: 此处记录会喂给 TableGen 生成器，产出 `.inc` 匹配器/数据表

## Dependencies / 依赖关系

- **LLVM subsystems / LLVM 子系统**: TableGen generators
