# RISCVInstrInfoXSfmm.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVInstrInfoXSfmm.td`
- **Repository**: llvm/llvm-project
- **Purpose**: Defines TableGen records for declarative instruction records, selection patterns, and extension-specific metadata for the RISC-V backend. / 使用 TableGen 定义RISC-V 后端的声明式指令记录、选择模式与扩展专用元数据。
- **Note**: This file uses the TableGen DSL to describe target metadata declaratively. / 该文件使用 TableGen DSL 以声明式方式描述目标元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17: Commentary and design intent / 注释与设计意图
```tablegen
//===-- RISCVInstrInfoXsfmm.td - SiFive matrix multiply ----*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file describes the Xsfmm* vendor extensions defined by SiFive.
//
//===----------------------------------------------------------------------===//

def XSfmmVTypeAsmOperand : AsmOperandClass {
  let Name = "XSfmmVType";
  let ParserMethod = "parseXSfmmVType";
  let RenderMethod = "addVTypeIOperands";
}
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 18-31: TableGen record XSfmmVTypeOp / TableGen 记录 XSfmmVTypeOp
```tablegen

def XSfmmVTypeOp : RISCVOp {
  let ParserMatchClass = XSfmmVTypeAsmOperand;
  let PrintMethod = "printXSfmmVType";
  let OperandType = "OPERAND_XSFMM_VTYPE";
  let MCOperandPredicate = [{
    int64_t Imm;
    if (!MCOp.evaluateAsConstantImm(Imm))
      return false;
    if (!isUInt<32>(Imm))
      return false;
    return RISCVVType::isValidXSfmmVType(Imm);
  }];
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 32-45: TableGen class SFInstSetSingle<dag / TableGen 类 SFInstSetSingle<dag
```tablegen

// TWiden operand should be named $twiden for getNamedOperandIdx.
// ATK operand should be named $atk for getNamedOperandIdx.
// ATM operand should be named $atm for getNamedOperandIdx.
// ATN operand should be named $atn for getNamedOperandIdx.
def twiden : RISCVOp {
  let OperandType = "OPERAND_XSFMM_TWIDEN";
}

let hasSideEffects = 1, mayLoad = 0, mayStore = 0 in
class SFInstSetSingle<dag outs, dag ins, bits<5> rs2, string opcodestr,
                      string argstr>
    : RVInstIBase<OPCFG.Value, OPC_OP_V, outs, ins, opcodestr, argstr> {
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 46-67: TableGen class SFInstTileMemOp<dag / TableGen 类 SFInstTileMemOp<dag
```tablegen
  let Inst{31-25} = 0b1000010;
  let Inst{24-20} = rs2;

  let Defs = [VL, VTYPE];
}

class SFInstTileMemOp<dag outs, dag ins, bits<3> nf, RISCVOpcode opcode,
                      string opcodestr, string argstr>
    : RVInst<outs, ins, opcodestr, argstr, [], InstFormatR> {
  bits<5> rs2;
  bits<5> rs1;

  let Inst{31-29} = nf;
  let Inst{28} = 1;
  let Inst{27-26} = MOPLDUnitStride.Value;
  let Inst{25} = 1;
  let Inst{24-20} = rs2;
  let Inst{19-15} = rs1;
  let Inst{14-12} = 0b111;
  let Inst{11-7} = 0b00000;
  let Inst{6-0} = opcode.Value;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 68-81: TableGen class SFInstTileLoad<bits<3> / TableGen 类 SFInstTileLoad<bits<3>
```tablegen
  let Uses = [VL, VTYPE];
  let ReadsPastVL = 1;
}

let hasSideEffects = 0, mayLoad = 1, mayStore = 0 in
class SFInstTileLoad<bits<3> nf, string opcodestr>
    : SFInstTileMemOp<(outs), (ins GPR:$rs2, GPRMemZeroOffset:$rs1), nf,
                      OPC_LOAD_FP, opcodestr, "$rs2, ${rs1}">;

let hasSideEffects = 0, mayLoad = 0, mayStore = 1 in
class SFInstTileStore<bits<3> nf, string opcodestr>
    : SFInstTileMemOp<(outs), (ins GPR:$rs2, GPRMemZeroOffset:$rs1), nf,
                      OPC_STORE_FP, opcodestr, "$rs2, ${rs1}">;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 82-97: TableGen class SFInstTileMoveOp<bits<6> / TableGen 类 SFInstTileMoveOp<bits<6>
```tablegen
let hasSideEffects = 0, mayLoad = 0, mayStore = 0 in
class SFInstTileMoveOp<bits<6> funct6, dag outs, dag ins, string opcodestr,
                       string argstr>
    : RVInst<outs, ins, opcodestr, argstr, [], InstFormatR> {
  bits<5> rs2;
  bits<5> rs1;
  bits<5> vd;

  let Inst{31-26} = funct6;
  let Inst{25} = 1;
  let Inst{24-20} = rs2;
  let Inst{19-15} = rs1;
  let Inst{14-12} = OPMVX.Value;
  let Inst{11-7} = vd;
  let Inst{6-0} = OPC_OP_V.Value;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 98-117: TableGen class SFInstMatmulF<dag / TableGen 类 SFInstMatmulF<dag
```tablegen
  let Uses = [VL, VTYPE];
  let ReadsPastVL = 1;
}

let hasSideEffects = 0, mayLoad = 0, mayStore = 0 in
class SFInstMatmulF<dag outs, dag ins, string opcodestr, string argstr>
    : RVInst<outs, ins, opcodestr, argstr, [], InstFormatR> {
  bits<5> vs2;
  bits<5> vs1;
  bits<4> rd;

  let Inst{31-26} = 0b111100;
  let Inst{25} = 1;
  let Inst{24-20} = vs2;
  let Inst{19-15} = vs1;
  let Inst{14-12} = OPFVV.Value;
  let Inst{11-9} = rd{3-1};
  let Inst{8-7} = 0b00;
  let Inst{6-0} = OPC_OP_VE.Value;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 118-140: TableGen class SFInstMatmulF8<bit / TableGen 类 SFInstMatmulF8<bit
```tablegen
  let Uses = [VL, VTYPE];
  let ReadsPastVL = 1;
}

let hasSideEffects = 0, mayLoad = 0, mayStore = 0 in
class SFInstMatmulF8<bit a, bit b, dag outs, dag ins,
                     string opcodestr, string argstr>
    : RVInst<outs, ins, opcodestr, argstr, [], InstFormatR> {
  bits<5> vs2;
  bits<5> vs1;
  bits<4> rd;

  let Inst{31-27} = 0b11111;
  let Inst{26} = a;
  let Inst{25} = 1;
  let Inst{24-20} = vs2;
  let Inst{19-15} = vs1;
  let Inst{14-12} = OPFVV.Value;
  let Inst{11-10} = rd{3-2};
  let Inst{9-8} = 0b00;
  let Inst{7} = b;
  let Inst{6-0} = OPC_OP_VE.Value;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 141-161: TableGen class F8Encode<bit / TableGen 类 F8Encode<bit
```tablegen
  let Uses = [VL, VTYPE];
  let ReadsPastVL = 1;
}


class F8Encode<bit encoding, string name> {
  bit Encoding = encoding;
  string Name = name;
}

defvar F8Encodes = [F8Encode<0b0, "e5m2">,
                    F8Encode<0b1, "e4m3">];

let hasSideEffects = 0, mayLoad = 0, mayStore = 0 in
class SFInstMatmulI8<bit funct6_1, bit a, bit b, dag outs, dag ins,
                     string opcodestr, string argstr>
    : RVInst<outs, ins, opcodestr, argstr, [], InstFormatR> {
  bits<5> vs2;
  bits<5> vs1;
  bits<4> rd;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 162-176: TableGen field customization and iteration / TableGen 字段定制与迭代
```tablegen
  let Inst{31-28} = 0b1111;
  let Inst{27} = funct6_1;
  let Inst{26} = a;
  let Inst{25} = 1;
  let Inst{24-20} = vs2;
  let Inst{19-15} = vs1;
  let Inst{14-12} = OPIVV.Value;
  let Inst{11-10} = rd{3-2};
  let Inst{9-8} = 0b00;
  let Inst{7} = b;
  let Inst{6-0} = OPC_OP_VE.Value;

  let Uses = [VL, VTYPE];
  let ReadsPastVL = 1;
}
```
**EN:** This block adjusts inherited fields or iterates over parameter sets to shape many related records consistently.

**CN:** 该区段调整继承字段或遍历参数集合，以一致地塑造多条相关记录。

### Lines 177-192: TableGen class I8Encode<bit / TableGen 类 I8Encode<bit
```tablegen

class I8Encode<bit encoding, string name> {
  bit Encoding = encoding;
  string Name = name;
}

defvar I8Encodes = [I8Encode<0, "u">,
                    I8Encode<1, "s">];

let hasSideEffects = 0, mayLoad = 0, mayStore = 0 in
class SFInstSetZero<dag outs, dag ins, string opcodestr, string argstr>
    : RVInst<outs, ins, opcodestr, argstr, [], InstFormatR> {
  bits<5> vs2;
  bits<5> vs1;
  bits<4> rd;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 193-215: TableGen class SFInstVtDiscard<string / TableGen 类 SFInstVtDiscard<string
```tablegen
  let Inst{31-26} = 0b010000;
  let Inst{25} = 1;
  let Inst{24-20} = 0b11110;
  let Inst{19-15} = 0b00000;
  let Inst{14-12} = OPMVX.Value;
  let Inst{11-8} = rd;
  let Inst{7} = 0;
  let Inst{6-0} = OPC_OP_V.Value;

  let Uses = [VL, VTYPE];
}

let hasSideEffects = 1, mayLoad = 0, mayStore = 0 in
class SFInstVtDiscard<string opcodestr>
    : RVInst<(outs), (ins), opcodestr, "", [], InstFormatR> {
  let Inst{31-26} = 0b010000;
  let Inst{25} = 1;
  let Inst{24-20} = 0b11100;
  let Inst{19-15} = 0b00000;
  let Inst{14-12} = OPMVX.Value;
  let Inst{11-7} = 0b00000;
  let Inst{6-0} = OPC_OP_V.Value;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 216-231: TableGen record SF_VSETTN / TableGen 记录 SF_VSETTN
```tablegen

let Predicates = [HasVendorXSfmmbase] in
def : InstAlias<"sf.vsettnt $rd, $rs1, $vtypei",
                (VSETVLI GPR:$rd, GPR:$rs1, XSfmmVTypeOp:$vtypei)>;

let DecoderNamespace = "XSfvector" in {

let Predicates = [HasVendorXSfmmbase] in {
  def SF_VSETTN : SFInstSetSingle<(outs GPR:$rd), (ins GPR:$rs1), 0b00000,
                                  "sf.vsettn", "$rd, $rs1">;
  def SF_VSETTM : SFInstSetSingle<(outs GPR:$rd), (ins GPR:$rs1), 0b00001,
                                  "sf.vsettm", "$rd, $rs1">;
  def SF_VSETTK : SFInstSetSingle<(outs GPR:$rd), (ins GPR:$rs1), 0b00010,
                                  "sf.vsettk", "$rd, $rs1">;
  def SF_VTDISCARD : SFInstVtDiscard<"sf.vtdiscard">;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 232-247: TableGen record SF_VTMV_V_T / TableGen 记录 SF_VTMV_V_T
```tablegen
  def SF_VTMV_V_T : SFInstTileMoveOp<0b010000, (outs VR:$vd), (ins GPR:$rs1),
                                     "sf.vtmv.v.t", "$vd, $rs1"> {
    let rs2 = 0b11111;
  }
  def SF_VTMV_T_V : SFInstTileMoveOp<0b010111, (outs), (ins GPR:$rs1, VR:$rs2),
                                     "sf.vtmv.t.v", "$rs1, $rs2"> {
    let vd = 0b00000;
  }

  def SF_VTZERO_T : SFInstSetZero<(outs), (ins TR:$rd), "sf.vtzero.t", "$rd">;

  def SF_VLTE8  : SFInstTileLoad<0b000, "sf.vlte8">;
  def SF_VLTE16 : SFInstTileLoad<0b001, "sf.vlte16">;
  def SF_VLTE32 : SFInstTileLoad<0b010, "sf.vlte32">;
  def SF_VLTE64 : SFInstTileLoad<0b011, "sf.vlte64">;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 248-268: TableGen record SF_VSTE8 / TableGen 记录 SF_VSTE8
```tablegen
  def SF_VSTE8  : SFInstTileStore<0b000, "sf.vste8">;
  def SF_VSTE16 : SFInstTileStore<0b001, "sf.vste16">;
  def SF_VSTE32 : SFInstTileStore<0b010, "sf.vste32">;
  def SF_VSTE64 : SFInstTileStore<0b011, "sf.vste64">;
} // Predicates = [HasVendorXSfmmbase]

let Predicates = [HasVendorXSfmm32a16fOrXSfmm32a32fOrXSfmm64a64f] in {
  let Uses = [FRM], mayRaiseFPException = true in
  def SF_MM_F_F   : SFInstMatmulF<(outs), (ins TRM2:$rd, VR:$vs2, VR:$vs1),
                                  "sf.mm.f.f", "$rd, $vs2, $vs1">;
} // Predicates = [HasVendorXSfmm32a16fOrXSfmm32a32fOrXSfmm64a64f]

let Predicates = [HasVendorXSfmm32a8i] in {
  foreach a = I8Encodes in
    foreach b = I8Encodes in
      def SF_MM_#!toupper(a.Name)#_#!toupper(b.Name)
          : SFInstMatmulI8<0, a.Encoding, b.Encoding,
                           (outs), (ins TRM4:$rd, VR:$vs2, VR:$vs1),
                           "sf.mm."#a.Name#"."#b.Name, "$rd, $vs2, $vs1">;
} // Predicates = [HasVendorXSfmm32a8i]
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 269-291: TableGen class VPseudoSF_VTileLoad / TableGen 类 VPseudoSF_VTileLoad
```tablegen
let Predicates = [HasVendorXSfmm32a8f] in {
let Uses = [FRM], mayRaiseFPException = true in {
  foreach a = F8Encodes in
    foreach b = F8Encodes in
      def SF_MM_#!toupper(a.Name)#_#!toupper(b.Name)
          : SFInstMatmulF8<a.Encoding, b.Encoding,
                           (outs), (ins TRM4:$rd, VR:$vs2, VR:$vs1),
                           "sf.mm."#a.Name#"."#b.Name, "$rd, $vs2, $vs1">;
}
} // Predicates = [HasVendorXSfmm32a8f]

} // DecoderNamespace = "XSfvector"

class VPseudoSF_VTileLoad
    : RISCVVPseudo<(outs), (ins GPR:$rs2, GPR:$rs1, GPRNoX0:$atn, sew:$sew,
                                twiden:$twiden)> {
  let mayLoad = 1;
  let mayStore = 0;
  let HasVLOp = 1; // Tn
  let HasSEWOp = 1;
  let HasTWidenOp = 1;
  let hasSideEffects = 1;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 292-313: TableGen class VPseudoSF_VTileStore / TableGen 类 VPseudoSF_VTileStore
```tablegen

class VPseudoSF_VTileStore
    : RISCVVPseudo<(outs), (ins GPR:$rs2, GPR:$rs1, GPRNoX0:$atn, sew:$sew,
                                twiden:$twiden)> {
  let mayLoad = 0;
  let mayStore = 1;
  let HasVLOp = 1; // Tn
  let HasSEWOp = 1;
  let HasTWidenOp = 1;
  let hasSideEffects = 1;
}

class VPseudoSF_VTileMove_V_T
    : RISCVVPseudo<(outs VRM8:$vd), (ins GPR:$rs1, GPRNoX0:$atn, sew:$sew,
                                         twiden:$twiden)> {
  let mayLoad = 0;
  let mayStore = 0;
  let HasVLOp = 1; // Tn
  let HasSEWOp = 1;
  let HasTWidenOp = 1;
  let hasSideEffects = 1;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 314-339: TableGen class VPseudoSF_VTileMove_T_V / TableGen 类 VPseudoSF_VTileMove_T_V
```tablegen

class VPseudoSF_VTileMove_T_V
    : RISCVVPseudo<(outs), (ins GPR:$rs1, VRM8:$vs2, GPRNoX0:$atn, sew:$sew,
                                twiden:$twiden)> {
  let mayLoad = 0;
  let mayStore = 0;
  let HasVLOp = 1; // Tn
  let HasSEWOp = 1;
  let HasTWidenOp = 1;
  let hasSideEffects = 1;
}

class VPseudoSF_MatMul<RegisterClass mtd_class>
    : RISCVVPseudo<(outs),
                   (ins mtd_class:$rd, VRM8:$vs2, VRM8:$vs1, GPRNoX0:$atm,
                        GPRNoX0:$atn, GPRNoX0:$atk, sew:$sew,
                        twiden:$twiden)> {
  let mayLoad = 0;
  let mayStore = 0;
  let HasTmOp = 1;
  let HasVLOp = 1; // Tn
  let HasTkOp = 1;
  let HasSEWOp = 1;
  let HasTWidenOp = 1;
  let hasSideEffects = 1;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 340-357: TableGen class VPseudoSF_MatMul_FRM<RegisterClass / TableGen 类 VPseudoSF_MatMul_FRM<RegisterClass
```tablegen

class VPseudoSF_MatMul_FRM<RegisterClass mtd_class>
    : RISCVVPseudo<(outs),
                   (ins mtd_class:$rd, VRM8:$vs2, VRM8:$vs1, vec_rm:$rm,
                        GPRNoX0:$atm, GPRNoX0:$atn, GPRNoX0:$atk, sew:$sew,
                        twiden:$twiden), []> {
  let mayLoad = 0;
  let mayStore = 0;
  let HasTmOp = 1;
  let HasVLOp = 1; // Tn
  let HasTkOp = 1;
  let HasSEWOp = 1;
  let HasRoundModeOp = 1;
  let hasPostISelHook = 1;
  let HasTWidenOp = 1;
  let hasSideEffects = 1;
  let BaseInstr = !cast<Instruction>(PseudoToVInst<NAME>.VInst);
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 358-376: TableGen record PseudoSF_VSETTNT / TableGen 记录 PseudoSF_VSETTNT
```tablegen

let hasSideEffects = 0, mayLoad = 0, mayStore = 0 in {
let Defs = [VL, VTYPE] in {
  def PseudoSF_VSETTNT
      : Pseudo<(outs GPR:$rd),
               (ins GPRNoX0:$rs1, XSfmmVTypeOp:$vtypei), []>,
        PseudoInstExpansion<(VSETVLI GPR:$rd, GPR:$rs1, VTypeIOp11:$vtypei)>,
        Sched<[WriteVSETVLI, ReadVSETVLI]>;
  def PseudoSF_VSETTNTX0
      : Pseudo<(outs GPRNoX0:$rd),
               (ins GPRX0:$rs1, XSfmmVTypeOp:$vtypei), []>,
        PseudoInstExpansion<(VSETVLI GPR:$rd, GPR:$rs1, VTypeIOp11:$vtypei)>,
        Sched<[WriteVSETVLI, ReadVSETVLI]>;
  def PseudoSF_VSETTNTX0X0
      : Pseudo<(outs GPRX0:$rd),
               (ins GPRX0:$rs1, XSfmmVTypeOp:$vtypei), []>,
        PseudoInstExpansion<(VSETVLI GPR:$rd, GPR:$rs1, VTypeIOp11:$vtypei)>,
        Sched<[WriteVSETVLI, ReadVSETVLI]>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 377-390: TableGen record PseudoSF_VSETTM / TableGen 记录 PseudoSF_VSETTM
```tablegen

let Defs = [VTYPE], Uses = [VTYPE], HasTWidenOp = 1, HasSEWOp = 1 in {
  def PseudoSF_VSETTM
      : Pseudo<(outs GPR:$rd),
               (ins GPR:$rs1, sew:$sew, twiden:$twiden), []>,
        PseudoInstExpansion<(SF_VSETTM GPR:$rd, GPR:$rs1)>,
        Sched<[WriteVSETVLI, ReadVSETVLI]>;
  def PseudoSF_VSETTK
      : Pseudo<(outs GPR:$rd),
               (ins GPR:$rs1, sew:$sew, twiden:$twiden), []>,
        PseudoInstExpansion<(SF_VSETTK GPR:$rd, GPR:$rs1)>,
        Sched<[WriteVSETVLI, ReadVSETVLI]>;
}
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 391-404: TableGen record eew / TableGen 记录 eew
```tablegen

foreach eew = [8, 16, 32, 64] in {
  def PseudoSF_VLTE # eew : VPseudoSF_VTileLoad;
  def PseudoSF_VSTE # eew : VPseudoSF_VTileStore;
}

def PseudoSF_VTMV_T_V : VPseudoSF_VTileMove_T_V;
def PseudoSF_VTMV_V_T : VPseudoSF_VTileMove_V_T;

foreach a = I8Encodes in
  foreach b = I8Encodes in
    def PseudoSF_MM_ # !toupper(a.Name) # _ # !toupper(b.Name)
        : VPseudoSF_MatMul<TRM4>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 405-422: TableGen record PseudoSF_MM_F_F / TableGen 记录 PseudoSF_MM_F_F
```tablegen
let AltFmtType = IS_NOT_ALTFMT in
  def PseudoSF_MM_F_F : VPseudoSF_MatMul_FRM<TRM2>;
let AltFmtType = IS_ALTFMT in
  def PseudoSF_MM_F_F_ALT : VPseudoSF_MatMul_FRM<TRM2>;

foreach e1 = [5, 4] in
  foreach e2 = [5, 4] in
    def PseudoSF_MM_E # e1 # M # !sub(7, e1) # _E # e2 # M # !sub(7, e2)
        : VPseudoSF_MatMul_FRM<TRM4>;

let hasSideEffects = 1, mayLoad = 0, mayStore = 0 in {
  let HasVLOp = 1, HasTmOp = 1, HasTWidenOp = 1, HasSEWOp = 1 in
    def PseudoSF_VTZERO_T
        : RISCVVPseudo<(outs),
                       (ins TR:$rd, GPRNoX0:$atm, GPRNoX0:$atn, sew:$sew,
                            twiden:$twiden)>;
  def PseudoSF_VTDISCARD : RISCVVPseudo<(outs), (ins), []>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 423-448: TableGen class VPatXSfmmTileStore<string / TableGen 类 VPatXSfmmTileStore<string
```tablegen

class VPatXSfmmTileStore<string intrinsic_name,
                         string inst_name,
                         int log2sew> :
  Pat<(!cast<Intrinsic>(intrinsic_name)
       (XLenVT GPR:$rs2),
       (XLenVT GPR:$rs1),
       (XLenVT GPRNoX0:$tn)),
      (!cast<Instruction>(inst_name)
       (XLenVT GPR:$rs2),
       (XLenVT GPR:$rs1),
       GPR:$tn, log2sew, 1)>;

class VPatXSfmmTileMove_T_V<string intrinsic_name,
                            string inst_name,
                            ValueType reg_type,
                            int log2sew> :
  Pat<(!cast<Intrinsic>(intrinsic_name)
                    (XLenVT GPR:$rs1),
                    (reg_type VRM8:$vs2),
                    (XLenVT GPRNoX0:$atn)),
      (!cast<Instruction>(inst_name)
       (XLenVT GPR:$rs1),
       (reg_type VRM8:$vs2),
       GPR:$atn, log2sew, 1)>;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 449-464: TableGen class VPatXSfmmTileMove_V_T<string / TableGen 类 VPatXSfmmTileMove_V_T<string
```tablegen
class VPatXSfmmTileMove_V_T<string intrinsic_name,
                            string inst_name,
                            ValueType result_type,
                            int log2sew> :
  Pat<(result_type (!cast<Intrinsic>(intrinsic_name)
                    (XLenVT GPR:$rs1),
                    (XLenVT GPRNoX0:$atn))),
      (!cast<Instruction>(inst_name)
       (XLenVT GPR:$rs1),
       GPR:$atn, log2sew, 1)>;

class VPatXSfmmVTDiscard<string intrinsic_name,
                         string inst_name> :
  Pat<(!cast<Intrinsic>(intrinsic_name)),
      (!cast<Instruction>(inst_name))>;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 465-473: TableGen record eew / TableGen 记录 eew
```tablegen
foreach eew = [8, 16, 32, 64] in
  def : VPatXSfmmTileStore<"int_riscv_sf_vste" # eew, "PseudoSF_VSTE" # eew, !logtwo(eew)>;

foreach vti = [VI8M8, VI16M8, VI32M8, VI64M8, VF16M8, VF32M8, VF64M8, VBF16M8] in {
  def : VPatXSfmmTileMove_T_V<"int_riscv_sf_vtmv_t_v", "PseudoSF_VTMV_T_V", vti.Vector, vti.Log2SEW>;
  def : VPatXSfmmTileMove_V_T<"int_riscv_sf_vtmv_v_t", "PseudoSF_VTMV_V_T", vti.Vector, vti.Log2SEW>;
}

def : VPatXSfmmVTDiscard<"int_riscv_sf_vtdiscard", "PseudoSF_VTDISCARD">;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

## Key Concepts / 关键概念
- **Instruction definitions** / **指令定义**
- **TableGen DSL** / **TableGen DSL**

## Dependencies / 依赖关系
- No explicit source includes in this file; dependencies are mostly implicit through surrounding backend integration. / 该文件没有显式源码级 include，依赖主要通过周边后端集成隐式体现。
