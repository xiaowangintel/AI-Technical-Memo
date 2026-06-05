# RISCVInstrPredicates.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVInstrPredicates.td`
- **Repository**: llvm/llvm-project
- **Purpose**: Defines TableGen records for declarative instruction records, selection patterns, and extension-specific metadata for the RISC-V backend. / 使用 TableGen 定义RISC-V 后端的声明式指令记录、选择模式与扩展专用元数据。
- **Note**: This file uses the TableGen DSL to describe target metadata declaratively. / 该文件使用 TableGen DSL 以声明式方式描述目标元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11: Commentary and design intent / 注释与设计意图
```tablegen
//===-- RISCVInstrPredicates.td - Instruction Predicates ---*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file describes the RISC-V instruction predicates.
//
//===----------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 12-21: Commentary and design intent / 注释与设计意图
```tablegen

// This predicate is true when the rs2 operand of vlse or vsse is x0, false
// otherwise.
// Note: in our pseudo RVV MachinInstr, rs2 is the fourth operand. However,
// it's the third operand of its MCInst form, so this predicate will not
// be triggered by llvm-mca, which checks against MCInst.
def VLDSX0Pred
    : AllOfSchedPreds<[FeatureSchedPredicate<TuneOptimizedZeroStrideLoad>,
                       MCSchedPredicate<CheckRegOperand<3, X0>>]>;
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 22-39: TableGen record SingleElementVecFP64SchedPred / TableGen 记录 SingleElementVecFP64SchedPred
```tablegen
// This scheduling predicate is true when subtarget feature TuneHasSingleElementVecFP64
// is enabled.
def SingleElementVecFP64SchedPred : FeatureSchedPredicate<TuneHasSingleElementVecFP64>;

// This scheduling predicate is true when the subtarget is RV32 (i.e. Feature64Bit
// is not enabled).
def RV32SchedPred : NotSchedPred<FeatureSchedPredicate<Feature64Bit>>;

// Returns true if this is the sext.w pattern, addiw rd, rs1, 0.
def isSEXT_W
    : TIIPredicate<"isSEXT_W",
                   MCReturnStatement<CheckAll<[
                     CheckOpcode<[ADDIW]>,
                     CheckIsRegOperand<1>,
                     CheckIsImmOperand<2>,
                     CheckImmOperand<2, 0>
                   ]>>>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 40-49: TableGen record isZEXT_W / TableGen 记录 isZEXT_W
```tablegen
// Returns true if this is the zext.w pattern, adduw rd, rs1, x0.
def isZEXT_W
    : TIIPredicate<"isZEXT_W",
                   MCReturnStatement<CheckAll<[
                     CheckOpcode<[ADD_UW]>,
                     CheckIsRegOperand<1>,
                     CheckIsRegOperand<2>,
                     CheckRegOperand<2, X0>
                   ]>>>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 50-59: TableGen record isZEXT_B / TableGen 记录 isZEXT_B
```tablegen
// Returns true if this is the zext.b pattern, andi rd, rs1, 255.
def isZEXT_B
    : TIIPredicate<"isZEXT_B",
                   MCReturnStatement<CheckAll<[
                     CheckOpcode<[ANDI]>,
                     CheckIsRegOperand<1>,
                     CheckIsImmOperand<2>,
                     CheckImmOperand<2, 255>
                   ]>>>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 60-79: TableGen record isSelectPseudo / TableGen 记录 isSelectPseudo
```tablegen
def isSelectPseudo
    : TIIPredicate<"isSelectPseudo",
                   MCReturnStatement<
                     CheckOpcode<[
                       Select_GPR_Using_CC_GPR,
                       Select_GPR_Using_CC_Imm5_Zibi,
                       Select_GPR_Using_CC_SImm5_CV,
                       Select_GPRNoX0_Using_CC_SImm5NonZero_QC,
                       Select_GPRNoX0_Using_CC_UImm5NonZero_QC,
                       Select_GPRNoX0_Using_CC_SImm16NonZero_QC,
                       Select_GPRNoX0_Using_CC_UImm16NonZero_QC,
                       Select_GPR_Using_CC_UImmLog2XLen_NDS,
                       Select_GPR_Using_CC_UImm7_NDS,
                       Select_FPR16_Using_CC_GPR,
                       Select_FPR16INX_Using_CC_GPR,
                       Select_FPR32_Using_CC_GPR,
                       Select_FPR32INX_Using_CC_GPR,
                       Select_FPR64_Using_CC_GPR,
                       Select_FPR64INX_Using_CC_GPR,
                       Select_FPR64IN32X_Using_CC_GPR
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 80-95: TableGen record isVectorConfigInstr / TableGen 记录 isVectorConfigInstr
```tablegen
                     ]>>>;

// Returns true if this is a vector configuration instruction.
def isVectorConfigInstr
    : TIIPredicate<"isVectorConfigInstr",
                   MCReturnStatement<
                     CheckOpcode<[
                       PseudoVSETVLI,
                       PseudoVSETVLIX0,
                       PseudoVSETVLIX0X0,
                       PseudoVSETIVLI,
                       PseudoSF_VSETTNT,
                       PseudoSF_VSETTNTX0,
                       PseudoSF_VSETTNTX0X0
                     ]>>>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 96-105: TableGen record isXSfmmVectorConfigTNInstr / TableGen 记录 isXSfmmVectorConfigTNInstr
```tablegen
// Returns true if this is a PseudoSF_VSETTNT* instructions.
def isXSfmmVectorConfigTNInstr
    : TIIPredicate<"isXSfmmVectorConfigTNInstr",
                   MCReturnStatement<
                     CheckOpcode<[
                       PseudoSF_VSETTNT,
                       PseudoSF_VSETTNTX0,
                       PseudoSF_VSETTNTX0X0
                     ]>>>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 106-125: TableGen record isXSfmmVectorConfigTMTKInstr / TableGen 记录 isXSfmmVectorConfigTMTKInstr
```tablegen
// Returns true if this is PseudoSF_VSETTM or PseudoSF_VSETTK.
def isXSfmmVectorConfigTMTKInstr
    : TIIPredicate<"isXSfmmVectorConfigTMTKInstr",
                   MCReturnStatement<
                     CheckOpcode<[
                       PseudoSF_VSETTM,
                       PseudoSF_VSETTK
                     ]>>>;

// Returns true if this is a XSfmm vector configuration instruction.
def isXSfmmVectorConfigInstr
    : TIIPredicate<"isXSfmmVectorConfigInstr",
                   MCReturnStatement<
                     CheckOpcode<[
                       PseudoSF_VSETTNT,
                       PseudoSF_VSETTNTX0,
                       PseudoSF_VSETTNTX0X0,
                       PseudoSF_VSETTM,
                       PseudoSF_VSETTK
                     ]>>>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 126-140: TableGen record isVLPreservingConfig / TableGen 记录 isVLPreservingConfig
```tablegen

// Return true if this is 'vsetvli x0, x0, vtype' which preserves
// VL and only sets VTYPE.
def isVLPreservingConfig
    : TIIPredicate<"isVLPreservingConfig",
                   MCReturnStatement<CheckOpcode<[PseudoVSETVLIX0X0]>>>;

def isFloatScalarMoveOrScalarSplatInstr
    : TIIPredicate<"isFloatScalarMoveOrScalarSplatInstr",
                   MCReturnStatement<
                     CheckOpcode<!listflatten([
                      !instances<Pseudo>("^PseudoVFMV_S_F.*"),
                      !instances<Pseudo>("^PseudoVFMV_V_F.*")
                     ])>>>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 141-156: TableGen record isScalarExtractInstr / TableGen 记录 isScalarExtractInstr
```tablegen
def isScalarExtractInstr
    : TIIPredicate<"isScalarExtractInstr",
                   MCReturnStatement<
                     CheckOpcode<!listflatten([
                      !instances<Pseudo>("^PseudoVMV_X_S.*"),
                      !instances<Pseudo>("^PseudoVFMV_F.*_S.*")
                     ])>>>;

def isScalarInsertInstr
    : TIIPredicate<"isScalarInsertInstr",
                   MCReturnStatement<
                     CheckOpcode<!listflatten([
                      !instances<Pseudo>("^PseudoVMV_S_X.*"),
                      !instances<Pseudo>("^PseudoVFMV_S_F.*")
                     ])>>>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 157-175: TableGen record isScalarSplatInstr / TableGen 记录 isScalarSplatInstr
```tablegen
def isScalarSplatInstr
    : TIIPredicate<"isScalarSplatInstr",
                   MCReturnStatement<
                     CheckOpcode<!listflatten([
                      !instances<Pseudo>("^PseudoVMV_V_I.*"),
                      !instances<Pseudo>("^PseudoVMV_V_X.*"),
                      !instances<Pseudo>("^PseudoVFMV_V_F.*")
                     ])>>>;

def isVSlideInstr
    : TIIPredicate<"isVSlideInstr",
                   MCReturnStatement<
                     CheckOpcode<!listflatten([
                      !instances<Pseudo>("^PseudoVSLIDEDOWN_VX.*"),
                      !instances<Pseudo>("^PseudoVSLIDEDOWN_VI.*"),
                      !instances<Pseudo>("^PseudoVSLIDEUP_VX.*"),
                      !instances<Pseudo>("^PseudoVSLIDEUP_VI.*")
                     ])>>>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 176-191: TableGen record isFaultOnlyFirstLoad / TableGen 记录 isFaultOnlyFirstLoad
```tablegen
def isFaultOnlyFirstLoad
    : TIIPredicate<"isFaultOnlyFirstLoad",
                    MCReturnStatement<
                      CheckOpcode<
                       !instances<Pseudo>(
                          "^PseudoVL(SEG[2-8])?E(8|16|32|64)FF_V.*")>>>;

def isLoadImmediate
    : TIIPredicate<"isLoadImmediate",
                   MCReturnStatement<CheckAll<[
                     CheckOpcode<[ADDI]>,
                     CheckIsRegOperand<1>,
                     CheckRegOperand<1, X0>,
                     CheckIsImmOperand<2>
                   ]>>>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 192-201: TableGen record isNonZeroLoadImmediate / TableGen 记录 isNonZeroLoadImmediate
```tablegen
def isNonZeroLoadImmediate
    : TIIPredicate<"isNonZeroLoadImmediate",
                   MCReturnStatement<CheckAll<[
                     CheckOpcode<[ADDI]>,
                     CheckIsRegOperand<1>,
                     CheckRegOperand<1, X0>,
                     CheckIsImmOperand<2>,
                     CheckNot<CheckImmOperand<2, 0>>
                   ]>>>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 202-219: TableGen record isLPAD / TableGen 记录 isLPAD
```tablegen
def isLPAD
    : TIIPredicate<"isLPAD",
                   MCReturnStatement<CheckAll<[
                     CheckOpcode<[AUIPC]>,
                     CheckIsRegOperand<0>,
                     CheckRegOperand<0, X0>,
                   ]>>>;

def ignoresVXRM
    : TIIPredicate<"ignoresVXRM",
                   MCOpcodeSwitchStatement<
                     [MCOpcodeSwitchCase<
                        !listflatten([
                          !instances<Pseudo>("^PseudoVNCLIP_WI.*"),
                          !instances<Pseudo>("^PseudoVNCLIPU_WI.*")
                        ]),
                        MCReturnStatement<CheckImmOperand<3, 0>>>],
                      MCReturnStatement<FalsePred>>>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

## Key Concepts / 关键概念
- **TableGen DSL** / **TableGen DSL**

## Dependencies / 依赖关系
- No explicit source includes in this file; dependencies are mostly implicit through surrounding backend integration. / 该文件没有显式源码级 include，依赖主要通过周边后端集成隐式体现。
