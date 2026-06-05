# AArch64InstrGISel.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/AArch64/AArch64InstrGISel.td`
- **Repository**: llvm/llvm-project
- **Purpose**: AArch64 GlobalISel target pseudo instruction definitions. This is kept separately from the other tablegen files for organizational purposes, but share the same infrastructure. It is written in the TableGen DSL. / 该文件实现 AArch64 后端中的AArch64 后端逻辑。 它使用 TableGen DSL 描述目标规则。
- **Note**: Written in the TableGen DSL / 使用 TableGen DSL 编写
## Line-by-Line Analysis / 逐行分析
### Lines 1-35: Documented TableGen section
```tablegen
//=----- AArch64InstrGISel.td - AArch64 GISel target pseudos -*- tablegen -*-=//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// AArch64 GlobalISel target pseudo instruction definitions. This is kept
// separately from the other tablegen files for organizational purposes, but
// share the same infrastructure.
//
//===----------------------------------------------------------------------===//


class AArch64GenericInstruction : GenericInstruction {
  let Namespace = "AArch64";
}

// A pseudo to represent a relocatable add instruction as part of address
// computation.
def G_ADD_LOW : AArch64GenericInstruction {
  let OutOperandList = (outs type0:$dst);
  let InOperandList = (ins type1:$src, type2:$imm);
  let hasSideEffects = 0;
}

// Pseudo for a rev32 instruction. Produced post-legalization from
// G_SHUFFLE_VECTORs with appropriate masks.
def G_REV32 : AArch64GenericInstruction {
  let OutOperandList = (outs type0:$dst);
  let InOperandList = (ins type0:$src);
  let hasSideEffects = 0;
  let GISelMatchGenericTypes = 1;
}
```
**EN:** This comment block marks a new TableGen section and explains the intent of the records that follow.  
**CN:** 该注释块标记了新的 TableGen 章节，并说明后续记录的设计意图。
### Lines 36-62: TableGen definition G_REV64
```tablegen

// Pseudo for a rev64 instruction. Produced post-legalization from
// G_SHUFFLE_VECTORs with appropriate masks.
def G_REV64 : AArch64GenericInstruction {
  let OutOperandList = (outs type0:$dst);
  let InOperandList = (ins type0:$src);
  let hasSideEffects = 0;
  let GISelMatchGenericTypes = 1;
}

// Represents an uzp1 instruction. Produced post-legalization from
// G_SHUFFLE_VECTORs with appropriate masks.
def G_UZP1 : AArch64GenericInstruction {
  let OutOperandList = (outs type0:$dst);
  let InOperandList = (ins type0:$v1, type0:$v2);
  let hasSideEffects = 0;
  let GISelMatchGenericTypes = 1;
}

// Represents an uzp2 instruction. Produced post-legalization from
// G_SHUFFLE_VECTORs with appropriate masks.
def G_UZP2 : AArch64GenericInstruction {
  let OutOperandList = (outs type0:$dst);
  let InOperandList = (ins type0:$v1, type0:$v2);
  let hasSideEffects = 0;
  let GISelMatchGenericTypes = 1;
}
```
**EN:** This definition materializes G_REV64 as a concrete backend record that TableGen can later turn into generated matchers, encoders, or metadata.  
**CN:** 该定义将 G_REV64 实例化为具体后端记录，后续 TableGen 可据此生成匹配器、编码器或元数据。
### Lines 63-89: TableGen definition G_ZIP1
```tablegen

// Represents a zip1 instruction. Produced post-legalization from
// G_SHUFFLE_VECTORs with appropriate masks.
def G_ZIP1 : AArch64GenericInstruction {
  let OutOperandList = (outs type0:$dst);
  let InOperandList = (ins type0:$v1, type0:$v2);
  let hasSideEffects = 0;
  let GISelMatchGenericTypes = 1;
}

// Represents a zip2 instruction. Produced post-legalization from
// G_SHUFFLE_VECTORs with appropriate masks.
def G_ZIP2 : AArch64GenericInstruction {
  let OutOperandList = (outs type0:$dst);
  let InOperandList = (ins type0:$v1, type0:$v2);
  let hasSideEffects = 0;
  let GISelMatchGenericTypes = 1;
}

// Represents a dup instruction. Produced post-legalization from
// G_SHUFFLE_VECTORs with appropriate masks.
def G_DUP: AArch64GenericInstruction {
  let OutOperandList = (outs type0:$dst);
  let InOperandList = (ins type1:$lane);
  let hasSideEffects = 0;
  let GISelMatchGenericTypes = 1;
}
```
**EN:** This definition materializes G_ZIP1 as a concrete backend record that TableGen can later turn into generated matchers, encoders, or metadata.  
**CN:** 该定义将 G_ZIP1 实例化为具体后端记录，后续 TableGen 可据此生成匹配器、编码器或元数据。
### Lines 90-124: TableGen definition G_DUPLANE8
```tablegen

// Represents a lane duplicate operation.
def G_DUPLANE8 : AArch64GenericInstruction {
  let OutOperandList = (outs type0:$dst);
  let InOperandList = (ins type0:$src, type1:$lane);
  let hasSideEffects = 0;
  let GISelMatchGenericTypes = 1;
}
def G_DUPLANE16 : AArch64GenericInstruction {
  let OutOperandList = (outs type0:$dst);
  let InOperandList = (ins type0:$src, type1:$lane);
  let hasSideEffects = 0;
  let GISelMatchGenericTypes = 1;
}
def G_DUPLANE32 : AArch64GenericInstruction {
  let OutOperandList = (outs type0:$dst);
  let InOperandList = (ins type0:$src, type1:$lane);
  let hasSideEffects = 0;
  let GISelMatchGenericTypes = 1;
}
def G_DUPLANE64 : AArch64GenericInstruction {
  let OutOperandList = (outs type0:$dst);
  let InOperandList = (ins type0:$src, type1:$lane);
  let hasSideEffects = 0;
  let GISelMatchGenericTypes = 1;
}

// Represents a trn1 instruction. Produced post-legalization from
// G_SHUFFLE_VECTORs with appropriate masks.
def G_TRN1 : AArch64GenericInstruction {
  let OutOperandList = (outs type0:$dst);
  let InOperandList = (ins type0:$v1, type0:$v2);
  let hasSideEffects = 0;
  let GISelMatchGenericTypes = 1;
}
```
**EN:** This definition materializes G_DUPLANE8 as a concrete backend record that TableGen can later turn into generated matchers, encoders, or metadata.  
**CN:** 该定义将 G_DUPLANE8 实例化为具体后端记录，后续 TableGen 可据此生成匹配器、编码器或元数据。
### Lines 125-156: TableGen definition G_TRN2
```tablegen

// Represents a trn2 instruction. Produced post-legalization from
// G_SHUFFLE_VECTORs with appropriate masks.
def G_TRN2 : AArch64GenericInstruction {
  let OutOperandList = (outs type0:$dst);
  let InOperandList = (ins type0:$v1, type0:$v2);
  let hasSideEffects = 0;
  let GISelMatchGenericTypes = 1;
}

// Represents an ext instruction. Produced post-legalization from
// G_SHUFFLE_VECTORs with appropriate masks.
def G_EXT: AArch64GenericInstruction {
  let OutOperandList = (outs type0:$dst);
  let InOperandList = (ins type0:$v1, type0:$v2, untyped_imm_0:$imm);
  let hasSideEffects = 0;
  let GISelMatchGenericTypes = 1;
}

// Represents a vector G_ASHR with an immediate.
def G_VASHR : AArch64GenericInstruction {
  let OutOperandList = (outs type0:$dst);
  let InOperandList = (ins type0:$src1, untyped_imm_0:$imm);
  let hasSideEffects = 0;
}

// Represents a vector G_LSHR with an immediate.
def G_VLSHR : AArch64GenericInstruction {
  let OutOperandList = (outs type0:$dst);
  let InOperandList = (ins type0:$src1, untyped_imm_0:$imm);
  let hasSideEffects = 0;
}
```
**EN:** This definition materializes G_TRN2 as a concrete backend record that TableGen can later turn into generated matchers, encoders, or metadata.  
**CN:** 该定义将 G_TRN2 实例化为具体后端记录，后续 TableGen 可据此生成匹配器、编码器或元数据。
### Lines 157-187: TableGen definition G_FPTRUNC_ODD
```tablegen

// Float truncation using round to odd
def G_FPTRUNC_ODD : AArch64GenericInstruction {
  let OutOperandList = (outs type0:$dst);
  let InOperandList = (ins type1:$src);
  let hasSideEffects = false;
}

// Represents an integer to FP conversion on the FPR bank.
def G_SITOF : AArch64GenericInstruction {
  let OutOperandList = (outs type0:$dst);
  let InOperandList = (ins type0:$src);
  let hasSideEffects = 0;
}
def G_UITOF : AArch64GenericInstruction {
  let OutOperandList = (outs type0:$dst);
  let InOperandList = (ins type0:$src);
  let hasSideEffects = 0;
}

def G_FCMEQ : AArch64GenericInstruction {
  let OutOperandList = (outs type0:$dst);
  let InOperandList = (ins type0:$src1, type1:$src2);
  let hasSideEffects = 0;
}

def G_FCMGE : AArch64GenericInstruction {
  let OutOperandList = (outs type0:$dst);
  let InOperandList = (ins type0:$src1, type1:$src2);
  let hasSideEffects = 0;
}
```
**EN:** This definition materializes G_FPTRUNC_ODD as a concrete backend record that TableGen can later turn into generated matchers, encoders, or metadata.  
**CN:** 该定义将 G_FPTRUNC_ODD 实例化为具体后端记录，后续 TableGen 可据此生成匹配器、编码器或元数据。
### Lines 188-217: TableGen definition G_FCMGT
```tablegen

def G_FCMGT : AArch64GenericInstruction {
  let OutOperandList = (outs type0:$dst);
  let InOperandList = (ins type0:$src1, type1:$src2);
  let hasSideEffects = 0;
}

def G_AARCH64_PREFETCH : AArch64GenericInstruction {
  let OutOperandList = (outs);
  let InOperandList = (ins type0:$imm, ptype0:$src1);
  let hasSideEffects = 1;
}

def G_AARCH64_RANGE_PREFETCH : AArch64GenericInstruction {
  let OutOperandList = (outs);
  let InOperandList = (ins type0:$imm, ptype0:$src1, type1:$src2);
  let hasSideEffects = 1;
}

def G_UMULL : AArch64GenericInstruction {
  let OutOperandList = (outs type0:$dst);
  let InOperandList = (ins type0:$src1, type0:$src2);
  let hasSideEffects = 0;
}

def G_SMULL : AArch64GenericInstruction {
  let OutOperandList = (outs type0:$dst);
  let InOperandList = (ins type0:$src1, type0:$src2);
  let hasSideEffects = 0;
}
```
**EN:** This definition materializes G_FCMGT as a concrete backend record that TableGen can later turn into generated matchers, encoders, or metadata.  
**CN:** 该定义将 G_FCMGT 实例化为具体后端记录，后续 TableGen 可据此生成匹配器、编码器或元数据。
### Lines 218-247: TableGen definition G_PMULL
```tablegen

def G_PMULL : AArch64GenericInstruction {
  let OutOperandList = (outs type0:$dst);
  let InOperandList = (ins type1:$src1, type1:$src2);
  let hasSideEffects = 0;
}

def G_UADDLP : AArch64GenericInstruction {
  let OutOperandList = (outs type0:$dst);
  let InOperandList = (ins type0:$src1);
  let hasSideEffects = 0;
}

def G_SADDLP : AArch64GenericInstruction {
  let OutOperandList = (outs type0:$dst);
  let InOperandList = (ins type0:$src1);
  let hasSideEffects = 0;
}

def G_UADDLV : AArch64GenericInstruction {
  let OutOperandList = (outs type0:$dst);
  let InOperandList = (ins type0:$src1);
  let hasSideEffects = 0;
}

def G_SADDLV : AArch64GenericInstruction {
  let OutOperandList = (outs type0:$dst);
  let InOperandList = (ins type0:$src1);
  let hasSideEffects = 0;
}
```
**EN:** This definition materializes G_PMULL as a concrete backend record that TableGen can later turn into generated matchers, encoders, or metadata.  
**CN:** 该定义将 G_PMULL 实例化为具体后端记录，后续 TableGen 可据此生成匹配器、编码器或元数据。
### Lines 248-277: TableGen definition G_UDOT
```tablegen

def G_UDOT : AArch64GenericInstruction {
  let OutOperandList = (outs type0:$dst);
  let InOperandList = (ins type0:$src1, type0:$src2, type0:$src3);
  let hasSideEffects = 0;
}

def G_SDOT : AArch64GenericInstruction {
  let OutOperandList = (outs type0:$dst);
  let InOperandList = (ins type0:$src1, type0:$src2, type0:$src3);
  let hasSideEffects = 0;
}

def G_USDOT : AArch64GenericInstruction {
  let OutOperandList = (outs type0:$dst);
  let InOperandList = (ins type0:$src1, type0:$src2, type0:$src3);
  let hasSideEffects = 0;
}

def G_SQSHLU_I : AArch64GenericInstruction {
  let OutOperandList = (outs type0:$dst);
  let InOperandList = (ins type0:$src1, type0:$src2);
  let hasSideEffects = 0;
}

def G_SRSHR_I: AArch64GenericInstruction {
  let OutOperandList = (outs type0:$dst);
  let InOperandList = (ins type0:$src1, type0:$src2);
  let hasSideEffects = 0;
}
```
**EN:** This definition materializes G_UDOT as a concrete backend record that TableGen can later turn into generated matchers, encoders, or metadata.  
**CN:** 该定义将 G_UDOT 实例化为具体后端记录，后续 TableGen 可据此生成匹配器、编码器或元数据。
### Lines 278-304: TableGen definition G_URSHR_I
```tablegen

def G_URSHR_I: AArch64GenericInstruction {
  let OutOperandList = (outs type0:$dst);
  let InOperandList = (ins type0:$src1, type0:$src2);
  let hasSideEffects = 0;
}

def G_SLI: AArch64GenericInstruction {
  let OutOperandList = (outs type0:$dst);
  let InOperandList = (ins type0:$src1, type0:$src2, type1:$src3);
  let hasSideEffects = 0;
}

def G_SRI: AArch64GenericInstruction {
  let OutOperandList = (outs type0:$dst);
  let InOperandList = (ins type0:$src1, type0:$src2, type1:$src3);
  let hasSideEffects = 0;
}

// Generic instruction for the BSP pseudo. It is expanded into BSP, which
// expands into BSL/BIT/BIF after register allocation.
def G_BSP : AArch64GenericInstruction {
  let OutOperandList = (outs type0:$dst);
  let InOperandList = (ins type0:$src1, type0:$src2, type0:$src3);
  let hasSideEffects = 0;
  let GISelMatchGenericTypes = 1;
}
```
**EN:** This definition materializes G_URSHR_I as a concrete backend record that TableGen can later turn into generated matchers, encoders, or metadata.  
**CN:** 该定义将 G_URSHR_I 实例化为具体后端记录，后续 TableGen 可据此生成匹配器、编码器或元数据。
### Lines 305-339: TableGen definition
```tablegen

def : GINodeEquiv<G_REV32, AArch64rev32>;
def : GINodeEquiv<G_REV64, AArch64rev64>;
def : GINodeEquiv<G_UZP1, AArch64uzp1>;
def : GINodeEquiv<G_UZP2, AArch64uzp2>;
def : GINodeEquiv<G_ZIP1, AArch64zip1>;
def : GINodeEquiv<G_ZIP2, AArch64zip2>;
def : GINodeEquiv<G_DUP, AArch64dup>;
def : GINodeEquiv<G_DUPLANE8, AArch64duplane8>;
def : GINodeEquiv<G_DUPLANE16, AArch64duplane16>;
def : GINodeEquiv<G_DUPLANE32, AArch64duplane32>;
def : GINodeEquiv<G_DUPLANE64, AArch64duplane64>;
def : GINodeEquiv<G_TRN1, AArch64trn1>;
def : GINodeEquiv<G_TRN2, AArch64trn2>;
def : GINodeEquiv<G_EXT, AArch64ext>;
def : GINodeEquiv<G_VASHR, AArch64vashr>;
def : GINodeEquiv<G_VLSHR, AArch64vlshr>;
def : GINodeEquiv<G_SITOF, AArch64sitof>;
def : GINodeEquiv<G_UITOF, AArch64uitof>;

def : GINodeEquiv<G_FCMEQ, AArch64fcmeq>;
def : GINodeEquiv<G_FCMGE, AArch64fcmge>;
def : GINodeEquiv<G_FCMGT, AArch64fcmgt>;

def : GINodeEquiv<G_BSP, AArch64bsp>;

def : GINodeEquiv<G_PMULL, AArch64pmull>;
def : GINodeEquiv<G_UMULL, AArch64umull>;
def : GINodeEquiv<G_SMULL, AArch64smull>;

def : GINodeEquiv<G_SADDLP, AArch64saddlp_n>;
def : GINodeEquiv<G_UADDLP, AArch64uaddlp_n>;

def : GINodeEquiv<G_SADDLV, AArch64saddlv>;
def : GINodeEquiv<G_UADDLV, AArch64uaddlv>;
```
**EN:** This definition materializes this definition as a concrete backend record that TableGen can later turn into generated matchers, encoders, or metadata.  
**CN:** 该定义将 this definition 实例化为具体后端记录，后续 TableGen 可据此生成匹配器、编码器或元数据。
### Lines 340-371: TableGen definition
```tablegen

def : GINodeEquiv<G_UDOT, AArch64udot>;
def : GINodeEquiv<G_SDOT, AArch64sdot>;
def : GINodeEquiv<G_USDOT, AArch64usdot>;

def : GINodeEquiv<G_SQSHLU_I, AArch64sqshlui>;
def : GINodeEquiv<G_SRSHR_I, AArch64srshri>;
def : GINodeEquiv<G_URSHR_I, AArch64urshri>;
def : GINodeEquiv<G_SLI, AArch64vsli>;
def : GINodeEquiv<G_SRI, AArch64vsri>;

def : GINodeEquiv<G_EXTRACT_VECTOR_ELT, vector_extract>;

def : GINodeEquiv<G_AARCH64_PREFETCH, AArch64Prefetch>;
def : GINodeEquiv<G_AARCH64_RANGE_PREFETCH, AArch64RangePrefetch>;

def : GINodeEquiv<G_FPTRUNC_ODD, AArch64fcvtxn_n>;

// These are patterns that we only use for GlobalISel via the importer.
def : Pat<(f32 (fadd (vector_extract (v2f32 FPR64:$Rn), (i64 0)),
                     (vector_extract (v2f32 FPR64:$Rn), (i64 1)))),
           (f32 (FADDPv2i32p (v2f32 FPR64:$Rn)))>;

let Predicates = [HasNoLSE] in {
def : Pat<(atomic_cmp_swap_i8 GPR64:$addr, GPR32:$desired, GPR32:$new),
          (CMP_SWAP_8 GPR64:$addr, GPR32:$desired, GPR32:$new)>;

def : Pat<(atomic_cmp_swap_i16 GPR64:$addr, GPR32:$desired, GPR32:$new),
          (CMP_SWAP_16 GPR64:$addr, GPR32:$desired, GPR32:$new)>;

def : Pat<(atomic_cmp_swap_i32 GPR64:$addr, GPR32:$desired, GPR32:$new),
          (CMP_SWAP_32 GPR64:$addr, GPR32:$desired, GPR32:$new)>;
```
**EN:** This definition materializes this definition as a concrete backend record that TableGen can later turn into generated matchers, encoders, or metadata.  
**CN:** 该定义将 this definition 实例化为具体后端记录，后续 TableGen 可据此生成匹配器、编码器或元数据。
### Lines 372-400: TableGen definition
```tablegen

def : Pat<(atomic_cmp_swap_i64 GPR64:$addr, GPR64:$desired, GPR64:$new),
          (CMP_SWAP_64 GPR64:$addr, GPR64:$desired, GPR64:$new)>;
}

def : Pat<(int_aarch64_stlxp GPR64:$lo, GPR64:$hi, GPR64:$addr),
          (STLXPX GPR64:$lo, GPR64:$hi, GPR64:$addr)>;
def : Pat<(int_aarch64_stxp GPR64:$lo, GPR64:$hi, GPR64:$addr),
          (STXPX GPR64:$lo, GPR64:$hi, GPR64:$addr)>;

let GIIgnoreCopies = 1 in
class PatIgnoreCopies<dag pattern, dag result> : Pat<pattern, result>, GISelFlags;

multiclass SIMDAcrossLanesSignedIntrinsicBHS<string baseOpc, Intrinsic intOp> {
  def : PatIgnoreCopies<(i32 (sext (i8 (intOp (v8i8 V64:$Rn))))),
        (i32 (SMOVvi8to32
          (INSERT_SUBREG (v16i8 (IMPLICIT_DEF)),
           (!cast<Instruction>(!strconcat(baseOpc, "v8i8v")) V64:$Rn), bsub),
          (i64 0)))>;
  def : Pat<(i8 (intOp (v8i8 V64:$Rn))),
        (!cast<Instruction>(!strconcat(baseOpc, "v8i8v")) V64:$Rn)>;

  def : PatIgnoreCopies<(i32 (sext (i8 (intOp (v16i8 V128:$Rn))))),
        (i32 (SMOVvi8to32
          (INSERT_SUBREG (v16i8 (IMPLICIT_DEF)),
           (!cast<Instruction>(!strconcat(baseOpc, "v16i8v")) V128:$Rn), bsub),
          (i64 0)))>;
  def : Pat<(i8 (intOp (v16i8 V128:$Rn))),
        (!cast<Instruction>(!strconcat(baseOpc, "v16i8v")) V128:$Rn)>;
```
**EN:** This definition materializes this definition as a concrete backend record that TableGen can later turn into generated matchers, encoders, or metadata.  
**CN:** 该定义将 this definition 实例化为具体后端记录，后续 TableGen 可据此生成匹配器、编码器或元数据。
### Lines 401-435: TableGen definition
```tablegen

  def : PatIgnoreCopies<(i32 (sext (i16 (intOp (v4i16 V64:$Rn))))),
        (i32 (SMOVvi16to32
          (INSERT_SUBREG (v16i8 (IMPLICIT_DEF)),
           (!cast<Instruction>(!strconcat(baseOpc, "v4i16v")) V64:$Rn), hsub),
          (i64 0)))>;
  def : Pat<(i16 (intOp (v4i16 V64:$Rn))),
        (!cast<Instruction>(!strconcat(baseOpc, "v4i16v")) V64:$Rn)>;

  def : PatIgnoreCopies<(i32 (sext (i16 (intOp (v8i16 V128:$Rn))))),
        (i32 (SMOVvi16to32
          (INSERT_SUBREG (v16i8 (IMPLICIT_DEF)),
           (!cast<Instruction>(!strconcat(baseOpc, "v8i16v")) V128:$Rn), hsub),
          (i64 0)))>;
  def : Pat<(i16 (intOp (v8i16 V128:$Rn))),
        (!cast<Instruction>(!strconcat(baseOpc, "v8i16v")) V128:$Rn)>;

  def : PatIgnoreCopies<(i32 (intOp (v4i32 V128:$Rn))),
        (i32 (EXTRACT_SUBREG
          (INSERT_SUBREG (v16i8 (IMPLICIT_DEF)),
           (!cast<Instruction>(!strconcat(baseOpc, "v4i32v")) V128:$Rn), ssub),
          ssub))>;
}

multiclass SIMDAcrossLanesUnsignedIntrinsicBHS<string baseOpc,
                                                Intrinsic intOp> {
  def : PatIgnoreCopies<(i32 (zext (i8 (intOp (v8i8 V64:$Rn))))),
        (COPY_TO_REGCLASS
          (i32 (EXTRACT_SUBREG
            (INSERT_SUBREG (v16i8 (IMPLICIT_DEF)),
              (!cast<Instruction>(!strconcat(baseOpc, "v8i8v")) V64:$Rn), bsub),
            ssub)),
          GPR32)>;
  def : Pat<(i8 (intOp (v8i8 V64:$Rn))),
        (!cast<Instruction>(!strconcat(baseOpc, "v8i8v")) V64:$Rn)>;
```
**EN:** This definition materializes this definition as a concrete backend record that TableGen can later turn into generated matchers, encoders, or metadata.  
**CN:** 该定义将 this definition 实例化为具体后端记录，后续 TableGen 可据此生成匹配器、编码器或元数据。
### Lines 436-466: TableGen definition
```tablegen

  def : PatIgnoreCopies<(i32 (zext (i8 (intOp (v16i8 V128:$Rn))))),
        (COPY_TO_REGCLASS
          (i32 (EXTRACT_SUBREG
            (INSERT_SUBREG (v16i8 (IMPLICIT_DEF)),
              (!cast<Instruction>(!strconcat(baseOpc, "v16i8v")) V128:$Rn), bsub),
            ssub)),
        GPR32)>;
  def : Pat<(i8 (intOp (v16i8 V128:$Rn))),
        (!cast<Instruction>(!strconcat(baseOpc, "v16i8v")) V128:$Rn)>;


  def : PatIgnoreCopies<(i32 (zext (i16 (intOp (v4i16 V64:$Rn))))),
        (COPY_TO_REGCLASS
          (i32 (EXTRACT_SUBREG
            (INSERT_SUBREG (v16i8 (IMPLICIT_DEF)),
              (!cast<Instruction>(!strconcat(baseOpc, "v4i16v")) V64:$Rn), hsub),
            ssub)),
          GPR32)>;
  def : Pat<(i16 (intOp (v4i16 V64:$Rn))),
        (!cast<Instruction>(!strconcat(baseOpc, "v4i16v")) V64:$Rn)>;

  def : PatIgnoreCopies<(i32 (zext (i16 (intOp (v8i16 V128:$Rn))))),
        (COPY_TO_REGCLASS
          (i32 (EXTRACT_SUBREG
            (INSERT_SUBREG (v16i8 (IMPLICIT_DEF)),
              (!cast<Instruction>(!strconcat(baseOpc, "v8i16v")) V128:$Rn), hsub),
            ssub)),
        GPR32)>;
  def : Pat<(i16 (intOp (v8i16 V128:$Rn))),
        (!cast<Instruction>(!strconcat(baseOpc, "v8i16v")) V128:$Rn)>;
```
**EN:** This definition materializes this definition as a concrete backend record that TableGen can later turn into generated matchers, encoders, or metadata.  
**CN:** 该定义将 this definition 实例化为具体后端记录，后续 TableGen 可据此生成匹配器、编码器或元数据。
### Lines 467-500: TableGen definition
```tablegen

  def : PatIgnoreCopies<(i32 (intOp (v4i32 V128:$Rn))),
        (i32 (EXTRACT_SUBREG
          (INSERT_SUBREG (v16i8 (IMPLICIT_DEF)),
            (!cast<Instruction>(!strconcat(baseOpc, "v4i32v")) V128:$Rn), ssub),
          ssub))>;
}


defm : SIMDAcrossLanesSignedIntrinsicBHS<"ADDV", int_aarch64_neon_saddv>;
// vaddv_[su]32 is special; -> ADDP Vd.2S,Vn.2S,Vm.2S; return Vd.s[0];Vn==Vm
def : Pat<(i32 (int_aarch64_neon_saddv (v2i32 V64:$Rn))),
          (i32 (EXTRACT_SUBREG
            (INSERT_SUBREG (v16i8 (IMPLICIT_DEF)),
              (ADDPv2i32 V64:$Rn, V64:$Rn), dsub),
            ssub))>;

def : Pat<(i64 (int_aarch64_neon_saddv (v2i64 V128:$Rn))),
          (i64 (EXTRACT_SUBREG
          (INSERT_SUBREG (v2i64 (IMPLICIT_DEF)),
              (ADDPv2i64p V128:$Rn), dsub),
            dsub))>;

defm : SIMDAcrossLanesUnsignedIntrinsicBHS<"ADDV", int_aarch64_neon_uaddv>;
def : Pat<(i32 (int_aarch64_neon_uaddv (v2i32 V64:$Rn))),
          (i32 (EXTRACT_SUBREG
            (INSERT_SUBREG (v16i8 (IMPLICIT_DEF)),
              (ADDPv2i32 V64:$Rn, V64:$Rn), dsub),
            ssub))>;
def : Pat<(i64 (int_aarch64_neon_uaddv (v2i64 V128:$Rn))),
          (i64 (EXTRACT_SUBREG
          (INSERT_SUBREG (v2i64 (IMPLICIT_DEF)),
              (ADDPv2i64p V128:$Rn), dsub),
            dsub))>;
```
**EN:** This definition materializes this definition as a concrete backend record that TableGen can later turn into generated matchers, encoders, or metadata.  
**CN:** 该定义将 this definition 实例化为具体后端记录，后续 TableGen 可据此生成匹配器、编码器或元数据。
### Lines 501-528: TableGen defm expansion
```tablegen

defm : SIMDAcrossLanesSignedIntrinsicBHS<"SMAXV", int_aarch64_neon_smaxv>;
def : Pat<(i32 (int_aarch64_neon_smaxv (v2i32 V64:$Rn))),
          (i32 (EXTRACT_SUBREG
            (INSERT_SUBREG (v16i8 (IMPLICIT_DEF)),
              (SMAXPv2i32 V64:$Rn, V64:$Rn), dsub),
            ssub))>;

defm : SIMDAcrossLanesSignedIntrinsicBHS<"SMINV", int_aarch64_neon_sminv>;
def : Pat<(i32 (int_aarch64_neon_sminv (v2i32 V64:$Rn))),
          (i32 (EXTRACT_SUBREG
            (INSERT_SUBREG (v16i8 (IMPLICIT_DEF)),
              (SMINPv2i32 V64:$Rn, V64:$Rn), dsub),
            ssub))>;

defm : SIMDAcrossLanesUnsignedIntrinsicBHS<"UMAXV", int_aarch64_neon_umaxv>;
def : Pat<(i32 (int_aarch64_neon_umaxv (v2i32 V64:$Rn))),
          (i32 (EXTRACT_SUBREG
            (INSERT_SUBREG (v16i8 (IMPLICIT_DEF)),
              (UMAXPv2i32 V64:$Rn, V64:$Rn), dsub),
            ssub))>;

defm : SIMDAcrossLanesUnsignedIntrinsicBHS<"UMINV", int_aarch64_neon_uminv>;
def : Pat<(i32 (int_aarch64_neon_uminv (v2i32 V64:$Rn))),
          (i32 (EXTRACT_SUBREG
            (INSERT_SUBREG (v16i8 (IMPLICIT_DEF)),
              (UMINPv2i32 V64:$Rn, V64:$Rn), dsub),
            ssub))>;
```
**EN:** This defm block expands this defm expansion into multiple concrete records, letting TableGen synthesize repeated backend data.  
**CN:** 该 defm 代码块把 this defm expansion 展开为多个具体记录，便于 TableGen 自动生成重复的后端数据。
### Lines 529-560: TableGen multiclass VecStoreLane64_0Pat
```tablegen

// Match stores from lane 0 to the appropriate subreg's store.
multiclass VecStoreLane64_0Pat<ComplexPattern UIAddrMode, SDPatternOperator storeop,
                            ValueType VTy, ValueType STy,
                            SubRegIndex SubRegIdx, Operand IndexType,
                            Instruction STR> {
  def : Pat<(storeop (STy (vector_extract (VTy VecListOne64:$Vt), (i64 0))),
                     (UIAddrMode GPR64sp:$Rn, IndexType:$offset)),
            (STR (EXTRACT_SUBREG VecListOne64:$Vt, SubRegIdx),
                 GPR64sp:$Rn, IndexType:$offset)>;
}
multiclass VecStoreULane64_0Pat<SDPatternOperator StoreOp,
                             ValueType VTy, ValueType STy,
                             SubRegIndex SubRegIdx, Instruction STR> {
  defm : VecStoreLane64_0Pat<am_unscaled64, StoreOp, VTy, STy, SubRegIdx, simm9, STR>;
}

multiclass VecROStoreLane64_0Pat<ROAddrMode ro, SDPatternOperator storeop,
                              ValueType VecTy, ValueType STy,
                              SubRegIndex SubRegIdx,
                              Instruction STRW, Instruction STRX> {

  def : Pat<(storeop (STy (vector_extract (VecTy VecListOne64:$Vt), (i64 0))),
                     (ro.Wpat GPR64sp:$Rn, GPR32:$Rm, ro.Wext:$extend)),
            (STRW (EXTRACT_SUBREG VecListOne64:$Vt, SubRegIdx),
                  GPR64sp:$Rn, GPR32:$Rm, ro.Wext:$extend)>;

  def : Pat<(storeop (STy (vector_extract (VecTy VecListOne64:$Vt), (i64 0))),
                     (ro.Xpat GPR64sp:$Rn, GPR64:$Rm, ro.Xext:$extend)),
            (STRX (EXTRACT_SUBREG VecListOne64:$Vt, SubRegIdx),
                  GPR64sp:$Rn, GPR64:$Rm, ro.Xext:$extend)>;
}
```
**EN:** This TableGen multiclass packages a family of related records into VecStoreLane64_0Pat for repeated instantiation.  
**CN:** 该 TableGen multiclass 将一组相关记录封装进 VecStoreLane64_0Pat，便于重复实例化。
### Lines 561-590: TableGen defm expansion
```tablegen

let AddedComplexity = 19 in {
  def : St1Lane128Pat<store, VectorIndexB, v16i8, i8,  ST1i8>;
  def : St1Lane64Pat<store, VectorIndexB, v8i8,  i8,  ST1i8>;

  defm : VecStoreLane64_0Pat<am_indexed16, store, v4i16, i16, hsub, uimm12s2, STRHui>;
  defm : VecStoreLane64_0Pat<am_indexed32, store, v2i32, i32, ssub, uimm12s4, STRSui>;

  defm : VecStoreULane64_0Pat<store, v4i16, i16, hsub, STURHi>;
  defm : VecStoreULane64_0Pat<store, v2i32, i32, ssub, STURSi>;
  defm : VecROStoreLane64_0Pat<ro16, store, v4i16, i16, hsub, STRHroW, STRHroX>;
  defm : VecROStoreLane64_0Pat<ro32, store, v2i32, i32, ssub, STRSroW, STRSroX>;
}

def : Pat<(v8i8 (AArch64dup (i8 (load GPR64sp:$Rn)))),
          (LD1Rv8b GPR64sp:$Rn)>;
def : Pat<(v16i8 (AArch64dup (i8 (load GPR64sp:$Rn)))),
          (LD1Rv16b GPR64sp:$Rn)>;
def : Pat<(v4i16 (AArch64dup (i16 (load GPR64sp:$Rn)))),
          (LD1Rv4h GPR64sp:$Rn)>;
def : Pat<(v8i16 (AArch64dup (i16 (load GPR64sp:$Rn)))),
          (LD1Rv8h GPR64sp:$Rn)>;
def : Pat<(v2i32 (AArch64dup (i32 (load GPR64sp:$Rn)))),
          (LD1Rv2s GPR64sp:$Rn)>;
def : Pat<(v4i32 (AArch64dup (i32 (load GPR64sp:$Rn)))),
          (LD1Rv4s GPR64sp:$Rn)>;
def : Pat<(v2i64 (AArch64dup (i64 (load GPR64sp:$Rn)))),
          (LD1Rv2d GPR64sp:$Rn)>;
def : Pat<(v1i64 (AArch64dup (i64 (load GPR64sp:$Rn)))),
          (LD1Rv1d GPR64sp:$Rn)>;
```
**EN:** This defm block expands this defm expansion into multiple concrete records, letting TableGen synthesize repeated backend data.  
**CN:** 该 defm 代码块把 this defm expansion 展开为多个具体记录，便于 TableGen 自动生成重复的后端数据。
## Key Concepts / 关键概念
- **EN:** AArch64 target backend structure **CN:** AArch64 目标后端结构
- **EN:** Declarative backend description in TableGen DSL **CN:** 使用 TableGen DSL 的声明式后端描述
## Dependencies / 依赖关系
- **EN:** Consumed by LLVM TableGen to produce generated matchers, encoders, register info, or scheduling tables. **CN:** 该文件会被 LLVM TableGen 消费，用于生成匹配器、编码器、寄存器信息或调度表。
- **EN:** Closely connected with neighboring AArch64 backend components responsible for AArch64 backend logic. **CN:** 与周边负责AArch64 后端逻辑的 AArch64 后端组件紧密协作。
