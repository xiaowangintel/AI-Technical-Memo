# RISCVInstrInfoZfa.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVInstrInfoZfa.td`
- **Repository**: llvm/llvm-project
- **Purpose**: Defines TableGen records for declarative instruction records, selection patterns, and extension-specific metadata for the RISC-V backend. / 使用 TableGen 定义RISC-V 后端的声明式指令记录、选择模式与扩展专用元数据。
- **Note**: This file uses the TableGen DSL to describe target metadata declaratively. / 该文件使用 TableGen DSL 以声明式方式描述目标元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12: Commentary and design intent / 注释与设计意图
```tablegen
//===-- RISCVInstrInfoZfa.td - RISC-V 'Zfa' instructions ---*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file describes the RISC-V instructions from the standard 'Zfa'
// additional floating-point extension, version 1.0.
//
//===----------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 13-23: TableGen record SDT_RISCVFLI / TableGen 记录 SDT_RISCVFLI
```tablegen

//===----------------------------------------------------------------------===//
// RISC-V specific DAG Nodes.
//===----------------------------------------------------------------------===//

def SDT_RISCVFLI
    : SDTypeProfile<1, 1, [SDTCisFP<0>, SDTCisVT<1, XLenVT>]>;

// Zfa fli instruction for constant materialization.
def riscv_fli : RVSDNode<"FLI", SDT_RISCVFLI>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 24-35: TableGen record LoadFPImmOperand / TableGen 记录 LoadFPImmOperand
```tablegen
//===----------------------------------------------------------------------===//
// Operand and SDNode transformation definitions.
//===----------------------------------------------------------------------===//

// 5-bit floating-point immediate encodings.
def LoadFPImmOperand : AsmOperandClass {
  let Name = "LoadFPImm";
  let ParserMethod = "parseFPImm";
  let RenderMethod = "addFPImmOperands";
  let DiagnosticType = "InvalidLoadFPImm";
  let DiagnosticString = "operand must be a valid floating-point constant";
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 36-50: TableGen record loadfpimm / TableGen 记录 loadfpimm
```tablegen

def loadfpimm : Operand<XLenVT> {
  let ParserMatchClass = LoadFPImmOperand;
  let PrintMethod = "printFPImmOperand";
  let OperandType = "OPERAND_UIMM5";
  let OperandNamespace = "RISCVOp";
}

def RTZArg : AsmOperandClass {
  let Name = "RTZArg";
  let RenderMethod = "addFRMArgOperands";
  let DiagnosticType = "InvalidRTZArg";
  let DiagnosticString = "operand must be 'rtz' floating-point rounding mode";
  let ParserMethod = "parseFRMArg";
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 51-60: TableGen record rtzarg / TableGen 记录 rtzarg
```tablegen

def rtzarg : Operand<XLenVT> {
  let ParserMatchClass = RTZArg;
  let PrintMethod = "printFRMArg";
  let DecoderMethod = "decodeFRMArg";
  let OperandType = "OPERAND_RTZARG";
  let OperandNamespace = "RISCVOp";
}

//===----------------------------------------------------------------------===//
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 61-76: TableGen class templates / TableGen 类 templates
```tablegen
// Instruction class templates
//===----------------------------------------------------------------------===//

let hasSideEffects = 0, mayLoad = 0, mayStore = 0, mayRaiseFPException = 1 in
class FPBinaryOp_rr<bits<7> funct7, bits<3> funct3, DAGOperand rdty,
                    DAGOperand rsty, string opcodestr>
    : RVInstR<funct7, funct3, OPC_OP_FP, (outs rdty:$rd),
              (ins rsty:$rs1, rsty:$rs2), opcodestr, "$rd, $rs1, $rs2">;

let hasSideEffects = 0, mayLoad = 0, mayStore = 0 in
class FPFLI_r<bits<7> funct7, bits<5> rs2val, bits<3> funct3,
              DAGOperand rdty, string opcodestr>
    : RVInstR<funct7, funct3, OPC_OP_FP, (outs rdty:$rd),
              (ins loadfpimm:$imm), opcodestr, "$rd, $imm"> {
  bits<5> imm;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 77-90: TableGen class FPUnaryOp_r_rtz<bits<7> / TableGen 类 FPUnaryOp_r_rtz<bits<7>
```tablegen
  let rs2 = rs2val;
  let rs1 = imm;
}

let hasSideEffects = 0, mayLoad = 0, mayStore = 0, mayRaiseFPException = 1,
    UseNamedOperandTable = 1, hasPostISelHook = 1 in
class FPUnaryOp_r_rtz<bits<7> funct7, bits<5> rs2val, DAGOperand rdty,
                      DAGOperand rs1ty, string opcodestr>
    : RVInstRFrm<funct7, OPC_OP_FP, (outs rdty:$rd),
                 (ins rs1ty:$rs1, rtzarg:$frm), opcodestr,
                  "$rd, $rs1$frm"> {
  let rs2 = rs2val;
  let frm = FRM_RTZ;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 91-100: TableGen record FLI_S / TableGen 记录 FLI_S
```tablegen

//===----------------------------------------------------------------------===//
// Instructions
//===----------------------------------------------------------------------===//

let Predicates = [HasStdExtZfa] in {
let isReMaterializable = 1, isAsCheapAsAMove = 1 in
def FLI_S : FPFLI_r<0b1111000, 0b00001, 0b000, FPR32, "fli.s">,
            Sched<[WriteFLI32]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 101-110: TableGen record FMINM_S: / TableGen 记录 FMINM_S:
```tablegen
let SchedRW = [WriteFMinMax32, ReadFMinMax32, ReadFMinMax32] in {
def FMINM_S: FPALU_rr<0b0010100, 0b010, "fminm.s", FPR32, Commutable=1>;
def FMAXM_S: FPALU_rr<0b0010100, 0b011, "fmaxm.s", FPR32, Commutable=1>;
}

def FROUND_S : FPUnaryOp_r_frm<0b0100000, 0b00100, FPR32, FPR32, "fround.s">,
               Sched<[WriteFRoundF32, ReadFRoundF32]>;
def FROUNDNX_S : FPUnaryOp_r_frm<0b0100000, 0b00101, FPR32, FPR32, "froundnx.s">,
                 Sched<[WriteFRoundF32, ReadFRoundF32]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 111-121: TableGen record FLTQ_S / TableGen 记录 FLTQ_S
```tablegen
let SchedRW = [WriteFCmp32, ReadFCmp32, ReadFCmp32] in {
def FLTQ_S : FPCmp_rr<0b1010000, 0b101, "fltq.s", FPR32>;
def FLEQ_S : FPCmp_rr<0b1010000, 0b100, "fleq.s", FPR32>;
}
} // Predicates = [HasStdExtZfa]

let Predicates = [HasStdExtZfa, HasStdExtD] in {
let isReMaterializable = 1, isAsCheapAsAMove = 1 in
def FLI_D : FPFLI_r<0b1111001, 0b00001, 0b000, FPR64, "fli.d">,
            Sched<[WriteFLI64]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 122-131: TableGen record FMINM_D: / TableGen 记录 FMINM_D:
```tablegen
let SchedRW = [WriteFMinMax64, ReadFMinMax64, ReadFMinMax64] in {
def FMINM_D: FPALU_rr<0b0010101, 0b010, "fminm.d", FPR64, Commutable=1>;
def FMAXM_D: FPALU_rr<0b0010101, 0b011, "fmaxm.d", FPR64, Commutable=1>;
}

def FROUND_D : FPUnaryOp_r_frm<0b0100001, 0b00100, FPR64, FPR64, "fround.d">,
               Sched<[WriteFRoundF64, ReadFRoundF64]>;
def FROUNDNX_D : FPUnaryOp_r_frm<0b0100001, 0b00101, FPR64, FPR64, "froundnx.d">,
                 Sched<[WriteFRoundF64, ReadFRoundF64]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 132-142: TableGen record FCVTMOD_W_D / TableGen 记录 FCVTMOD_W_D
```tablegen
let IsSignExtendingOpW = 1 in
def FCVTMOD_W_D
    : FPUnaryOp_r_rtz<0b1100001, 0b01000, GPR, FPR64, "fcvtmod.w.d">,
      Sched<[WriteFCvtF64ToI32, ReadFCvtF64ToI32]>;

let SchedRW = [WriteFCmp64, ReadFCmp64, ReadFCmp64] in {
def FLTQ_D : FPCmp_rr<0b1010001, 0b101, "fltq.d", FPR64>;
def FLEQ_D : FPCmp_rr<0b1010001, 0b100, "fleq.d", FPR64>;
}
} // Predicates = [HasStdExtZfa, HasStdExtD]
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 143-156: TableGen record FMVH_X_D / TableGen 记录 FMVH_X_D
```tablegen
let Predicates = [HasStdExtZfa, HasStdExtD, IsRV32] in {
let mayRaiseFPException = 0 in {
def FMVH_X_D : FPUnaryOp_r<0b1110001, 0b00001, 0b000, GPR, FPR64, "fmvh.x.d">,
               Sched<[WriteFMovF64ToI64, ReadFMovF64ToI64]>;
def FMVP_D_X : FPBinaryOp_rr<0b1011001, 0b000, FPR64, GPR, "fmvp.d.x">,
               Sched<[WriteFMovI64ToF64, ReadFMovI64ToF64, ReadFMovI64ToF64]>;
}

let isCodeGenOnly = 1, mayRaiseFPException = 0 in
def FMV_X_W_FPR64 : FPUnaryOp_r<0b1110000, 0b00000, 0b000, GPR, FPR64,
                                "fmv.x.w">,
                    Sched<[WriteFMovF64ToI64, ReadFMovF64ToI64]>;
} // Predicates = [HasStdExtZfa, HasStdExtD, IsRV32]
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 157-166: TableGen record FLI_H / TableGen 记录 FLI_H
```tablegen
let Predicates = [HasStdExtZfa, HasStdExtZfhOrZvfh] in
let isReMaterializable = 1, isAsCheapAsAMove = 1 in
def FLI_H : FPFLI_r<0b1111010, 0b00001, 0b000, FPR16, "fli.h">,
            Sched<[WriteFLI16]>;

let Predicates = [HasStdExtZfa, HasStdExtZfh] in {
let SchedRW = [WriteFMinMax16, ReadFMinMax16, ReadFMinMax16] in {
def FMINM_H: FPALU_rr<0b0010110, 0b010, "fminm.h", FPR16, Commutable=1>;
def FMAXM_H: FPALU_rr<0b0010110, 0b011, "fmaxm.h", FPR16, Commutable=1>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 167-176: TableGen record FROUND_H / TableGen 记录 FROUND_H
```tablegen

def FROUND_H : FPUnaryOp_r_frm<0b0100010, 0b00100, FPR16, FPR16, "fround.h">,
               Sched<[WriteFRoundF16, ReadFRoundF16]>;
def FROUNDNX_H : FPUnaryOp_r_frm<0b0100010, 0b00101, FPR16, FPR16, "froundnx.h">,
                 Sched<[WriteFRoundF16, ReadFRoundF16]>;

let SchedRW = [WriteFCmp16, ReadFCmp16, ReadFCmp16] in {
def FLTQ_H : FPCmp_rr<0b1010010, 0b101, "fltq.h", FPR16>;
def FLEQ_H : FPCmp_rr<0b1010010, 0b100, "fleq.h", FPR16>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 177-187: TableGen record FLI_Q / TableGen 记录 FLI_Q
```tablegen
} // Predicates = [HasStdExtZfa, HasStdExtZfh]

let Predicates = [HasStdExtZfa, HasStdExtQ] in {
let isReMaterializable = 1, isAsCheapAsAMove = 1 in
def FLI_Q : FPFLI_r<0b1111011, 0b00001, 0b000, FPR128, "fli.q">,
            Sched<[WriteFLI128]>;

let SchedRW = [WriteFMinMax128, ReadFMinMax128, ReadFMinMax128] in {
def FMINM_Q: FPALU_rr<0b0010111, 0b010, "fminm.q", FPR128, Commutable=1>;
def FMAXM_Q: FPALU_rr<0b0010111, 0b011, "fmaxm.q", FPR128, Commutable=1>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 188-198: TableGen record FROUND_Q / TableGen 记录 FROUND_Q
```tablegen

def FROUND_Q : FPUnaryOp_r_frm<0b0100011, 0b00100, FPR128, FPR128, "fround.q">,
               Sched<[WriteFRoundF128, ReadFRoundF128]>;
def FROUNDNX_Q : FPUnaryOp_r_frm<0b0100011, 0b00101, FPR128, FPR128, 
                                 "froundnx.q">,
                 Sched<[WriteFRoundF128, ReadFRoundF128]>;

let SchedRW = [WriteFCmp128, ReadFCmp128, ReadFCmp128] in {
def FLTQ_Q : FPCmp_rr<0b1010011, 0b101, "fltq.q", FPR128>;
def FLEQ_Q : FPCmp_rr<0b1010011, 0b100, "fleq.q", FPR128>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 199-208: TableGen record FMVH_X_Q / TableGen 记录 FMVH_X_Q
```tablegen
} // Predicates = [HasStdExtZfa, HasStdExtQ]

let Predicates = [HasStdExtZfa, HasStdExtQ, IsRV64] in {
  let mayRaiseFPException = 0 in {
    def FMVH_X_Q : FPUnaryOp_r<0b1110011, 0b00001, 0b000, GPR, FPR128, "fmvh.x.q">;
    def FMVP_Q_X : FPBinaryOp_rr<0b1011011, 0b000, FPR128, GPR, "fmvp.q.x">;
  }
} // Predicates = [HasStdExtZfa, HasStdExtQ, IsRV64]

//===----------------------------------------------------------------------===//
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 209-218: TableGen record definition / TableGen 记录定义
```tablegen
// Pseudo-instructions and codegen patterns
//===----------------------------------------------------------------------===//

let Predicates = [HasStdExtZfa] in {
def : InstAlias<"fgtq.s $rd, $rs, $rt",
                (FLTQ_S GPR:$rd, FPR32:$rt, FPR32:$rs), 0>;
def : InstAlias<"fgeq.s $rd, $rs, $rt",
                (FLEQ_S GPR:$rd, FPR32:$rt, FPR32:$rs), 0>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 219-231: TableGen record definition / TableGen 记录定义
```tablegen
let Predicates = [HasStdExtZfa, HasStdExtD] in {
def : InstAlias<"fgtq.d $rd, $rs, $rt",
                (FLTQ_D GPR:$rd, FPR64:$rt, FPR64:$rs), 0>;
def : InstAlias<"fgeq.d $rd, $rs, $rt",
                (FLEQ_D GPR:$rd, FPR64:$rt, FPR64:$rs), 0>;
}

let Predicates = [HasStdExtZfa, HasStdExtZfh] in {
def : InstAlias<"fgtq.h $rd, $rs, $rt",
                (FLTQ_H GPR:$rd, FPR16:$rt, FPR16:$rs), 0>;
def : InstAlias<"fgeq.h $rd, $rs, $rt",
                (FLEQ_H GPR:$rd, FPR16:$rt, FPR16:$rs), 0>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 232-242: TableGen record definition / TableGen 记录定义
```tablegen

let Predicates = [HasStdExtZfa, HasStdExtQ] in {
def : InstAlias<"fgtq.q $rd, $rs, $rt",
                (FLTQ_Q GPR:$rd, FPR128:$rt, FPR128:$rs), 0>;
def : InstAlias<"fgeq.q $rd, $rs, $rt",
                (FLEQ_Q GPR:$rd, FPR128:$rt, FPR128:$rs), 0>;
}

//===----------------------------------------------------------------------===//
// Codegen patterns
//===----------------------------------------------------------------------===//
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 243-253: TableGen field customization and iteration / TableGen 字段定制与迭代
```tablegen

let Predicates = [HasStdExtZfa] in {
def: Pat<(f32 (riscv_fli timm:$imm)), (FLI_S timm:$imm)>;

def: PatFprFpr<fminimum, FMINM_S, FPR32, f32>;
def: PatFprFpr<fmaximum, FMAXM_S, FPR32, f32>;

// frint rounds according to the current rounding mode and detects
// inexact conditions.
def: Pat<(any_frint FPR32:$rs1), (FROUNDNX_S FPR32:$rs1, FRM_DYN)>;
```
**EN:** This block adjusts inherited fields or iterates over parameter sets to shape many related records consistently.

**CN:** 该区段调整继承字段或遍历参数集合，以一致地塑造多条相关记录。

### Lines 254-268: Definitions and supporting logic / 定义与支撑逻辑
```tablegen
// fnearbyint is like frint but does not detect inexact conditions.
def: Pat<(any_fnearbyint FPR32:$rs1), (FROUND_S FPR32:$rs1, FRM_DYN)>;

def: Pat<(any_fround FPR32:$rs1), (FROUND_S FPR32:$rs1, FRM_RMM)>;
def: Pat<(any_froundeven FPR32:$rs1), (FROUND_S FPR32:$rs1, FRM_RNE)>;
def: Pat<(any_ffloor FPR32:$rs1), (FROUND_S FPR32:$rs1, FRM_RDN)>;
def: Pat<(any_fceil FPR32:$rs1), (FROUND_S FPR32:$rs1, FRM_RUP)>;
def: Pat<(any_ftrunc FPR32:$rs1), (FROUND_S FPR32:$rs1, FRM_RTZ)>;

def: PatSetCC<FPR32, strict_fsetcc, SETLT, FLTQ_S, f32>;
def: PatSetCC<FPR32, strict_fsetcc, SETOLT, FLTQ_S, f32>;
def: PatSetCC<FPR32, strict_fsetcc, SETLE, FLEQ_S, f32>;
def: PatSetCC<FPR32, strict_fsetcc, SETOLE, FLEQ_S, f32>;
} // Predicates = [HasStdExtZfa]
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 269-278: TableGen field customization and iteration / TableGen 字段定制与迭代
```tablegen
let Predicates = [HasStdExtZfa, HasStdExtD] in {
def: Pat<(f64 (riscv_fli timm:$imm)), (FLI_D timm:$imm)>;

def: PatFprFpr<fminimum, FMINM_D, FPR64, f64>;
def: PatFprFpr<fmaximum, FMAXM_D, FPR64, f64>;

// frint rounds according to the current rounding mode and detects
// inexact conditions.
def: Pat<(any_frint FPR64:$rs1), (FROUNDNX_D FPR64:$rs1, FRM_DYN)>;
```
**EN:** This block adjusts inherited fields or iterates over parameter sets to shape many related records consistently.

**CN:** 该区段调整继承字段或遍历参数集合，以一致地塑造多条相关记录。

### Lines 279-293: Definitions and supporting logic / 定义与支撑逻辑
```tablegen
// fnearbyint is like frint but does not detect inexact conditions.
def: Pat<(any_fnearbyint FPR64:$rs1), (FROUND_D FPR64:$rs1, FRM_DYN)>;

def: Pat<(any_fround FPR64:$rs1), (FROUND_D FPR64:$rs1, FRM_RMM)>;
def: Pat<(any_froundeven FPR64:$rs1), (FROUND_D FPR64:$rs1, FRM_RNE)>;
def: Pat<(any_ffloor FPR64:$rs1), (FROUND_D FPR64:$rs1, FRM_RDN)>;
def: Pat<(any_fceil FPR64:$rs1), (FROUND_D FPR64:$rs1, FRM_RUP)>;
def: Pat<(any_ftrunc FPR64:$rs1), (FROUND_D FPR64:$rs1, FRM_RTZ)>;

def: PatSetCC<FPR64, strict_fsetcc, SETLT, FLTQ_D, f64>;
def: PatSetCC<FPR64, strict_fsetcc, SETOLT, FLTQ_D, f64>;
def: PatSetCC<FPR64, strict_fsetcc, SETLE, FLEQ_D, f64>;
def: PatSetCC<FPR64, strict_fsetcc, SETOLE, FLEQ_D, f64>;
} // Predicates = [HasStdExtZfa, HasStdExtD]
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 294-304: TableGen record definition / TableGen 记录定义
```tablegen
let Predicates = [HasStdExtZfa, HasStdExtD, IsRV32] in {
def : Pat<(RISCVBuildPairF64 GPR:$rs1, GPR:$rs2),
          (FMVP_D_X GPR:$rs1, GPR:$rs2)>;
}

let Predicates = [HasStdExtZfa, HasStdExtZfh] in {
def: Pat<(f16 (riscv_fli timm:$imm)), (FLI_H timm:$imm)>;

def: PatFprFpr<fminimum, FMINM_H, FPR16, f16>;
def: PatFprFpr<fmaximum, FMAXM_H, FPR16, f16>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 305-317: Definitions and supporting logic / 定义与支撑逻辑
```tablegen
// frint rounds according to the current rounding mode and detects
// inexact conditions.
def: Pat<(f16 (any_frint FPR16:$rs1)), (FROUNDNX_H FPR16:$rs1, FRM_DYN)>;

// fnearbyint is like frint but does not detect inexact conditions.
def: Pat<(f16 (any_fnearbyint FPR16:$rs1)), (FROUND_H FPR16:$rs1, FRM_DYN)>;

def: Pat<(f16 (any_fround FPR16:$rs1)), (FROUND_H FPR16:$rs1, FRM_RMM)>;
def: Pat<(f16 (any_froundeven FPR16:$rs1)), (FROUND_H FPR16:$rs1, FRM_RNE)>;
def: Pat<(f16 (any_ffloor FPR16:$rs1)), (FROUND_H FPR16:$rs1, FRM_RDN)>;
def: Pat<(f16 (any_fceil FPR16:$rs1)), (FROUND_H FPR16:$rs1, FRM_RUP)>;
def: Pat<(f16 (any_ftrunc FPR16:$rs1)), (FROUND_H FPR16:$rs1, FRM_RTZ)>;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 318-322: Definitions and supporting logic / 定义与支撑逻辑
```tablegen
def: PatSetCC<FPR16, strict_fsetcc, SETLT, FLTQ_H, f16>;
def: PatSetCC<FPR16, strict_fsetcc, SETOLT, FLTQ_H, f16>;
def: PatSetCC<FPR16, strict_fsetcc, SETLE, FLEQ_H, f16>;
def: PatSetCC<FPR16, strict_fsetcc, SETOLE, FLEQ_H, f16>;
} // Predicates = [HasStdExtZfa, HasStdExtZfh]
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

## Key Concepts / 关键概念
- **Instruction definitions** / **指令定义**
- **TableGen DSL** / **TableGen DSL**

## Dependencies / 依赖关系
- No explicit source includes in this file; dependencies are mostly implicit through surrounding backend integration. / 该文件没有显式源码级 include，依赖主要通过周边后端集成隐式体现。
