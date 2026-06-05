# RISCVInstrInfoZvk.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVInstrInfoZvk.td`
- **Repository**: llvm/llvm-project
- **Purpose**: Defines TableGen records for declarative instruction records, selection patterns, and extension-specific metadata for the RISC-V backend. / 使用 TableGen 定义RISC-V 后端的声明式指令记录、选择模式与扩展专用元数据。
- **Note**: This file uses the TableGen DSL to describe target metadata declaratively. / 该文件使用 TableGen DSL 以声明式方式描述目标元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: Commentary and design intent / 注释与设计意图
```tablegen
//===-- RISCVInstrInfoZvk.td - RISC-V 'Zvk' instructions ---*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file describes the RISC-V instructions from the standard 'Zvk',
// Vector Cryptography Instructions extension, version Release 1.0.0.
//
//===----------------------------------------------------------------------===//

//===----------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 15-30: Reusable TableGen multiclass templates / 可复用的 TableGen 多类 templates
```tablegen
// Operand and SDNode transformation definitions.
//===----------------------------------------------------------------------===//

def tuimm5 : RISCVOp, TImmLeaf<XLenVT, [{return isUInt<5>(Imm);}]>;

//===----------------------------------------------------------------------===//
// Instruction class templates
//===----------------------------------------------------------------------===//

let hasSideEffects = 0, mayLoad = 0, mayStore = 0 in {
multiclass VCLMUL_MV_V_X<string opcodestr, bits<6> funct6> {
  def V  : VALUVV<funct6, OPMVV, opcodestr # "." # "vv">,
           SchedBinaryMC<"WriteVCLMULV", "ReadVCLMULV", "ReadVCLMULV">;
  def X  : VALUVX<funct6, OPMVX, opcodestr # "." # "vx">,
           SchedBinaryMC<"WriteVCLMULX", "ReadVCLMULV", "ReadVCLMULX">;
}
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 31-44: TableGen class RVInstIVI_VROR<bits<6> / TableGen 类 RVInstIVI_VROR<bits<6>
```tablegen

class RVInstIVI_VROR<bits<6> funct6, dag outs, dag ins, string opcodestr,
                     string argstr>
    : RVInstVBase<funct6, OPIVI, outs, ins, opcodestr, argstr> {
  bits<5> vs2;
  bits<6> imm;

  // This overwrites the lsb of funct6 in the RVInstVBase base class with imm{5}
  assert !eq(funct6{0}, 0), "funct6 LSB should be zero";
  let Inst{26} = imm{5};
  let Inst{24-20} = vs2;
  let Inst{19-15} = imm{4-0};
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 45-58: Reusable TableGen multiclass ZvkALUVVNoVm<bits<6> / 可复用的 TableGen 多类 ZvkALUVVNoVm<bits<6>
```tablegen
multiclass VROR_IV_V_X_I<string opcodestr, bits<6> funct6>
    : VALU_IV_V_X<opcodestr, funct6> {
  def I : RVInstIVI_VROR<funct6, (outs VR:$vd),
              (ins VR:$vs2, uimm6:$imm, VMaskOp:$vm),
              opcodestr # ".vi", "$vd, $vs2, $imm$vm">,
          SchedUnaryMC<"WriteVRotI", "ReadVRotV">;
}

// op vd, vs2, vs1
class ZvkALUVVNoVm<bits<6> funct6, RISCVVFormat opv, string opcodestr>
    : VALUVVNoVm<funct6, opv, opcodestr> {
  let Inst{6-0} = OPC_OP_VE.Value;
}
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 59-76: TableGen class ZvkALUVVNoVmTernary<bits<6> / TableGen 类 ZvkALUVVNoVmTernary<bits<6>
```tablegen
// op vd, vs2, vs1
class ZvkALUVVNoVmTernary<bits<6> funct6, RISCVVFormat opv, string opcodestr>
    : RVInstVV<funct6, opv, (outs VR:$vd_wb),
               (ins VR:$vd, VR:$vs2, VR:$vs1),
               opcodestr, "$vd, $vs2, $vs1"> {
  let Constraints = "$vd = $vd_wb";
  let vm = 1;
  let Inst{6-0} = OPC_OP_VE.Value;

  let VMConstraint = false;
}

// op vd, vs2, imm
class ZvkALUVINoVm<bits<6> funct6, string opcodestr, Operand optype>
    : VALUVINoVm<funct6, opcodestr, optype> {
  let Inst{6-0} = OPC_OP_VE.Value;
  let Inst{14-12} = OPMVV.Value;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 77-90: TableGen class ZvkALUVINoVmBinary<bits<6> / TableGen 类 ZvkALUVINoVmBinary<bits<6>
```tablegen

// op vd, vs2, imm where vd is also a source regardless of tail policy
class ZvkALUVINoVmBinary<bits<6> funct6, string opcodestr, Operand optype>
    : RVInstIVI<funct6, (outs VR:$vd_wb),
                (ins VR:$vd, VR:$vs2, optype:$imm),
                opcodestr, "$vd, $vs2, $imm"> {
  let Constraints = "$vd = $vd_wb";
  let vm = 1;
  let Inst{6-0} = OPC_OP_VE.Value;
  let Inst{14-12} = OPMVV.Value;

  let VMConstraint = false;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 91-111: Reusable TableGen multiclass ZvkALUVs2NoVmBinary<bits<6> / 可复用的 TableGen 多类 ZvkALUVs2NoVmBinary<bits<6>
```tablegen
// op vd, vs2 (use vs1 as instruction encoding) where vd is also a source
// regardless of tail policy
class ZvkALUVs2NoVmBinary<bits<6> funct6, bits<5> vs1, RISCVVFormat opv,
                        string opcodestr>
    : RVInstVUnary<funct6, vs1, opv, (outs VR:$vd_wb), (ins VR:$vd, VR:$vs2),
                   opcodestr, "$vd, $vs2"> {
  let Constraints = "$vd = $vd_wb";
  let vm = 1;
  let Inst{6-0} = OPC_OP_VE.Value;

  let VMConstraint = false;
}

multiclass VAES_MV_V_S<bits<6> funct6_vv, bits<6> funct6_vs, bits<5> vs1,
                         RISCVVFormat opv, string opcodestr> {
  def NAME # _VV : ZvkALUVs2NoVmBinary<funct6_vv, vs1, opv, opcodestr # ".vv">,
                   SchedBinaryMC<"WriteVAESMVV", "ReadVAESMVV", "ReadVAESMVV">;
  let VS1VS2Constraint = VS2Constraint in
  def NAME # _VS : ZvkALUVs2NoVmBinary<funct6_vs, vs1, opv, opcodestr # ".vs">,
                   SchedBinaryMC<"WriteVAESMVV", "ReadVAESMVV", "ReadVAESMVV">;
}
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 112-129: Bulk record instantiation VBREV_V / 批量记录实例化 VBREV_V
```tablegen
} // hasSideEffects = 0, mayLoad = 0, mayStore = 0

//===----------------------------------------------------------------------===//
// Instructions
//===----------------------------------------------------------------------===//

let ElementsDependOn = EltDepsNone in {

let Predicates = [HasStdExtZvbb] in {
  def  VBREV_V  : VALUVs2<0b010010, 0b01010, OPMVV, "vbrev.v">;
  def  VCLZ_V   : VALUVs2<0b010010, 0b01100, OPMVV, "vclz.v">;
  def  VCPOP_V  : VALUVs2<0b010010, 0b01110, OPMVV, "vcpop.v">;
  def  VCTZ_V   : VALUVs2<0b010010, 0b01101, OPMVV, "vctz.v">;
  let Constraints = "@earlyclobber $vd", VS1VS2Constraint = WidenV,
      DestEEW = EEWSEWx2 in
  defm VWSLL_V  : VSHT_IV_V_X_I<"vwsll", 0b110101>;
} // Predicates = [HasStdExtZvbb]
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 130-144: Bulk record instantiation VCLMUL_V / 批量记录实例化 VCLMUL_V
```tablegen
let Predicates = [HasStdExtZvbcOrZvbc32e] in {
  defm VCLMUL_V  : VCLMUL_MV_V_X<"vclmul", 0b001100>;
  defm VCLMULH_V : VCLMUL_MV_V_X<"vclmulh", 0b001101>;
} // Predicates = [HasStdExtZvbcOrZvbc32e]

let Predicates = [HasStdExtZvkb] in {
  defm VANDN_V  : VALU_IV_V_X<"vandn", 0b000001>;
  def  VBREV8_V : VALUVs2<0b010010, 0b01000, OPMVV, "vbrev8.v">;
  def  VREV8_V  : VALUVs2<0b010010, 0b01001, OPMVV, "vrev8.v">;
  defm VROL_V   : VALU_IV_V_X<"vrol", 0b010101>;
  defm VROR_V   : VROR_IV_V_X_I<"vror", 0b010100>;
} // Predicates = [HasStdExtZvkb]

} // ElementsDependOn = EltDepsNone
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 145-160: TableGen record VGHSH_VV / TableGen 记录 VGHSH_VV
```tablegen
let Predicates = [HasStdExtZvkg] in {
  def VGHSH_VV : ZvkALUVVNoVmTernary<0b101100, OPMVV, "vghsh.vv">,
                 SchedTernaryMC<"WriteVGHSHV", "ReadVGHSHV", "ReadVGHSHV",
                                "ReadVGHSHV">;
  def VGMUL_VV : ZvkALUVs2NoVmBinary<0b101000, 0b10001, OPMVV, "vgmul.vv">,
                 SchedBinaryMC<"WriteVGMULV", "ReadVGMULV", "ReadVGMULV">;
} // Predicates = [HasStdExtZvkg]

let Predicates = [HasStdExtZvkgs], VS1VS2Constraint = VS2Constraint in {
  def VGHSH_VS : ZvkALUVVNoVmTernary<0b100011, OPMVV, "vghsh.vs">,
                 SchedTernaryMC<"WriteVGHSHV", "ReadVGHSHV", "ReadVGHSHV",
                                "ReadVGHSHV">;
  def VGMUL_VS : ZvkALUVs2NoVmBinary<0b101001, 0b10001, OPMVV, "vgmul.vs">,
                 SchedBinaryMC<"WriteVGMULV", "ReadVGMULV", "ReadVGMULV">;
} // Predicates = [HasStdExtZvkgs]
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 161-186: Bulk record instantiation VSHA2CH_VV / 批量记录实例化 VSHA2CH_VV
```tablegen
let Predicates = [HasStdExtZvknha], VS1VS2Constraint = Sha2Constraint in {
  def VSHA2CH_VV : ZvkALUVVNoVmTernary<0b101110, OPMVV, "vsha2ch.vv">,
                   SchedTernaryMC<"WriteVSHA2CHV", "ReadVSHA2CHV", "ReadVSHA2CHV",
                                  "ReadVSHA2CHV">;
  def VSHA2CL_VV : ZvkALUVVNoVmTernary<0b101111, OPMVV, "vsha2cl.vv">,
                   SchedTernaryMC<"WriteVSHA2CLV", "ReadVSHA2CLV", "ReadVSHA2CLV",
                                  "ReadVSHA2CLV">;
  def VSHA2MS_VV : ZvkALUVVNoVmTernary<0b101101, OPMVV, "vsha2ms.vv">,
                   SchedTernaryMC<"WriteVSHA2MSV", "ReadVSHA2MSV", "ReadVSHA2MSV",
                                  "ReadVSHA2MSV">;
} // Predicates = [HasStdExtZvknha]

let Predicates = [HasStdExtZvkned] in {
  defm VAESDF     : VAES_MV_V_S<0b101000, 0b101001, 0b00001, OPMVV, "vaesdf">;
  defm VAESDM     : VAES_MV_V_S<0b101000, 0b101001, 0b00000, OPMVV, "vaesdm">;
  defm VAESEF     : VAES_MV_V_S<0b101000, 0b101001, 0b00011, OPMVV, "vaesef">;
  defm VAESEM     : VAES_MV_V_S<0b101000, 0b101001, 0b00010, OPMVV, "vaesem">;
  def  VAESKF1_VI : ZvkALUVINoVm<0b100010, "vaeskf1.vi", uimm5>,
                    SchedUnaryMC<"WriteVAESKF1V", "ReadVAESKF1V">;
  def  VAESKF2_VI : ZvkALUVINoVmBinary<0b101010, "vaeskf2.vi", uimm5>,
                    SchedBinaryMC<"WriteVAESKF2V", "ReadVAESKF2V", "ReadVAESKF2V">;
  let VS1VS2Constraint = VS2Constraint in
  def  VAESZ_VS   : ZvkALUVs2NoVmBinary<0b101001, 0b00111, OPMVV, "vaesz.vs">,
                    SchedBinaryMC<"WriteVAESZV", "ReadVAESZV", "ReadVAESZV">;
} // Predicates = [HasStdExtZvkned]
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 187-200: Bulk record instantiation VSM4K_VI / 批量记录实例化 VSM4K_VI
```tablegen
let Predicates = [HasStdExtZvksed] in {
  def  VSM4K_VI : ZvkALUVINoVm<0b100001, "vsm4k.vi", uimm5>,
                  SchedUnaryMC<"WriteVSM4KV", "ReadVSM4KV">;
  defm VSM4R    : VAES_MV_V_S<0b101000, 0b101001, 0b10000, OPMVV, "vsm4r">;
} // Predicates = [HasStdExtZvksed]

let Predicates = [HasStdExtZvksh], VS1VS2Constraint = VS2Constraint in {
  def VSM3C_VI  : ZvkALUVINoVmBinary<0b101011, "vsm3c.vi", uimm5>,
                  SchedBinaryMC<"WriteVSM3CV", "ReadVSM3CV", "ReadVSM3CV">;
  def VSM3ME_VV : ZvkALUVVNoVm<0b100000, OPMVV, "vsm3me.vv">,
                  SchedUnaryMC<"WriteVSM3MEV", "ReadVSM3MEV">;
} // Predicates = [HasStdExtZvksh]

//===----------------------------------------------------------------------===//
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 201-214: TableGen class ZvkI32IntegerVectors<string / TableGen 类 ZvkI32IntegerVectors<string
```tablegen
// Pseudo instructions
//===----------------------------------------------------------------------===//

defvar I32IntegerVectors = !filter(vti, AllIntegerVectors, !eq(vti.SEW, 32));

class ZvkI32IntegerVectors<string vd_lmul> {
  list<VTypeInfo> vs2_types = !cond(!eq(vd_lmul, "M8")  : !filter(vti, I32IntegerVectors, !le(vti.LMul.octuple, 32)),
                                    !eq(vd_lmul, "M4")  : !filter(vti, I32IntegerVectors, !le(vti.LMul.octuple, 32)),
                                    !eq(vd_lmul, "M2")  : !filter(vti, I32IntegerVectors, !le(vti.LMul.octuple, 16)),
                                    !eq(vd_lmul, "M1")  : !filter(vti, I32IntegerVectors, !le(vti.LMul.octuple, 8)),
                                    !eq(vd_lmul, "MF2")  : !filter(vti, I32IntegerVectors, !le(vti.LMul.octuple, 4)),
                                    !eq(vd_lmul, "MF4")  : !filter(vti, I32IntegerVectors, !le(vti.LMul.octuple, 2)),
                                    !eq(vd_lmul, "MF8")  : !filter(vti, I32IntegerVectors, !le(vti.LMul.octuple, 1)));
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 215-237: TableGen class ZvkMxSet<string / TableGen 类 ZvkMxSet<string
```tablegen

class ZvkMxSet<string vd_lmul> {
  list<LMULInfo> vs2_lmuls = !cond(!eq(vd_lmul, "M8")  : [V_MF8, V_MF4, V_MF2, V_M1, V_M2, V_M4],
                                   !eq(vd_lmul, "M4")  : [V_MF8, V_MF4, V_MF2, V_M1, V_M2, V_M4],
                                   !eq(vd_lmul, "M2")  : [V_MF8, V_MF4, V_MF2, V_M1, V_M2],
                                   !eq(vd_lmul, "M1")  : [V_MF8, V_MF4, V_MF2, V_M1],
                                   !eq(vd_lmul, "MF2") : [V_MF8, V_MF4, V_MF2],
                                   !eq(vd_lmul, "MF4") : [V_MF8, V_MF4],
                                   !eq(vd_lmul, "MF8") : [V_MF8]);
}

class VPseudoBinaryNoMask_Zvk<DAGOperand RetClass, VReg OpClass> :
      RISCVVPseudo<(outs RetClass:$rd_wb),
        (ins RetClass:$rd, OpClass:$rs2, AVL:$vl, sew:$sew, vec_policy:$policy), []> {
  let mayLoad = 0;
  let mayStore = 0;
  let hasSideEffects = 0;
  let Constraints = "$rd_wb = $rd";
  let HasVLOp = 1;
  let HasSEWOp = 1;
  let HasVecPolicyOp = 1;
  let BaseInstr = !cast<Instruction>(PseudoToVInst<NAME>.VInst);
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 238-253: TableGen class VPseudoTernaryNoMask_Zvk<VReg / TableGen 类 VPseudoTernaryNoMask_Zvk<VReg
```tablegen

class VPseudoTernaryNoMask_Zvk<VReg RetClass,
                               VReg Op1Class,
                               DAGOperand Op2Class> :
        RISCVVPseudo<(outs RetClass:$rd_wb),
               (ins RetClass:$rd, Op1Class:$rs2, Op2Class:$rs1,
                    AVL:$vl, sew:$sew, vec_policy:$policy), []> {
  let mayLoad = 0;
  let mayStore = 0;
  let hasSideEffects = 0;
  let Constraints = "$rd_wb = $rd";
  let HasVLOp = 1;
  let HasSEWOp = 1;
  let HasVecPolicyOp = 1;
  let BaseInstr = !cast<Instruction>(PseudoToVInst<NAME>.VInst);
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 254-273: Reusable TableGen multiclass VPseudoBinaryNoMaskPolicy_Zvk / 可复用的 TableGen 多类 VPseudoBinaryNoMaskPolicy_Zvk
```tablegen

multiclass VPseudoBinaryNoMaskPolicy_Zvk<VReg RetClass,
                                         VReg Op1Class,
                                         DAGOperand Op2Class,
                                         LMULInfo MInfo,
                                         string Constraint = ""> {
  let VLMul = MInfo.value in {
    def "_" # MInfo.MX : VPseudoBinaryNoMaskPolicy<RetClass, Op1Class, Op2Class,
                                           Constraint>;
  }
}

multiclass VPseudoTernaryNoMask_Zvk<VReg RetClass,
                                    VReg Op1Class,
                                    DAGOperand Op2Class,
                                    LMULInfo MInfo, int sew = 0> {
  let VLMul = MInfo.value, SEW = sew in {
    defvar suffix = !if(sew, "_" # MInfo.MX # "_E" # sew, "_" # MInfo.MX);
    def suffix : VPseudoTernaryNoMask_Zvk<RetClass, Op1Class, Op2Class>;
  }
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 274-287: Reusable TableGen multiclass VPseudoBinaryV_V_NoMask_Zvk / 可复用的 TableGen 多类 VPseudoBinaryV_V_NoMask_Zvk
```tablegen
}

multiclass VPseudoBinaryV_V_NoMask_Zvk<LMULInfo m> {
  let VLMul = m.value in {
    def "_VV_" # m.MX : VPseudoBinaryNoMask_Zvk<m.vrclass, m.vrclass>;
  }
}

multiclass VPseudoBinaryV_S_NoMask_Zvk<LMULInfo m> {
  let VLMul = m.value in
    foreach vs2_lmul = ZvkMxSet<m.MX>.vs2_lmuls in
      def "_VS_" # m.MX # "_" # vs2_lmul.MX : VPseudoBinaryNoMask_Zvk<m.vrclass, vs2_lmul.vrclass>;
}
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 288-303: Reusable TableGen multiclass VPseudoVGMUL / 可复用的 TableGen 多类 VPseudoVGMUL
```tablegen
multiclass VPseudoVGMUL {
  foreach m = MxListVF4 in {
    defvar mx = m.MX;
    defm "" : VPseudoBinaryV_V_NoMask_Zvk<m>,
              SchedBinary<"WriteVGMULV", "ReadVGMULV", "ReadVGMULV", mx>;
  }
}

multiclass VPseudoVAESMV {
  foreach m = MxListVF4 in {
    defvar mx = m.MX;
    defm "" : VPseudoBinaryV_V_NoMask_Zvk<m>,
              SchedBinary<"WriteVAESMVV", "ReadVAESMVV", "ReadVAESMVV", mx>;
    defm "" : VPseudoBinaryV_S_NoMask_Zvk<m>,
              SchedBinary<"WriteVAESMVV", "ReadVAESMVV", "ReadVAESMVV", mx>;
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 304-317: Reusable TableGen multiclass VPseudoVSM4R / 可复用的 TableGen 多类 VPseudoVSM4R
```tablegen
  }
}

multiclass VPseudoVSM4R {
  foreach m = MxListVF4 in {
    defvar mx = m.MX;
    defm "" : VPseudoBinaryV_V_NoMask_Zvk<m>,
              SchedBinary<"WriteVSM4RV", "ReadVSM4RV", "ReadVSM4RV", mx>;
    defm "" : VPseudoBinaryV_S_NoMask_Zvk<m>,
              SchedBinary<"WriteVSM4RV", "ReadVSM4RV", "ReadVSM4RV", mx>;

  }
}
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 318-333: Reusable TableGen multiclass VPseudoVGHSH / 可复用的 TableGen 多类 VPseudoVGHSH
```tablegen
multiclass VPseudoVGHSH {
  foreach m = MxListVF4 in {
    defvar mx = m.MX;
    defm _VV : VPseudoTernaryNoMask_Zvk<m.vrclass, m.vrclass, m.vrclass, m>,
               SchedTernary<"WriteVGHSHV", "ReadVGHSHV", "ReadVGHSHV",
                            "ReadVGHSHV", mx>;
  }
}

multiclass VPseudoVSHA2CH {
  foreach m = MxListVF4 in {
    defvar mx = m.MX;
    defm _VV : VPseudoTernaryNoMask_Zvk<m.vrclass, m.vrclass, m.vrclass, m>,
               SchedTernary<"WriteVSHA2CHV", "ReadVSHA2CHV", "ReadVSHA2CHV",
                            "ReadVSHA2CHV", mx>;
  }
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 334-351: Reusable TableGen multiclass VPseudoVSHA2CL / 可复用的 TableGen 多类 VPseudoVSHA2CL
```tablegen
}

multiclass VPseudoVSHA2CL {
  foreach m = MxListVF4 in {
    defvar mx = m.MX;
    defm _VV : VPseudoTernaryNoMask_Zvk<m.vrclass, m.vrclass, m.vrclass, m>,
               SchedTernary<"WriteVSHA2CLV", "ReadVSHA2CLV", "ReadVSHA2CLV",
                            "ReadVSHA2CLV", mx>;
  }
}

multiclass VPseudoVSHA2MS<int sew = 0> {
  foreach m = !if(!eq(sew, 64), MxListVF8, MxListVF4) in {
    defvar mx = m.MX;
    defm _VV : VPseudoTernaryNoMask_Zvk<m.vrclass, m.vrclass, m.vrclass, m, sew = sew>,
               SchedTernary<"WriteVSHA2MSV", "ReadVSHA2MSV", "ReadVSHA2MSV",
                            "ReadVSHA2MSV", mx, sew>;
  }
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 352-369: Reusable TableGen multiclass VPseudoVAESKF1 / 可复用的 TableGen 多类 VPseudoVAESKF1
```tablegen
}

multiclass VPseudoVAESKF1 {
  foreach m = MxListVF4 in {
    defvar mx = m.MX;
    defm _VI : VPseudoBinaryNoMaskPolicy_Zvk<m.vrclass, m.vrclass, uimm5, m>,
               SchedBinary<"WriteVAESKF1V", "ReadVAESKF1V", "ReadVAESKF1V", mx,
                           forcePassthruRead=true>;
  }
}

multiclass VPseudoVAESKF2 {
  foreach m = MxListVF4 in {
    defvar mx = m.MX;
    defm _VI : VPseudoTernaryNoMask_Zvk<m.vrclass, m.vrclass, uimm5, m>,
               SchedTernary<"WriteVAESKF2V", "ReadVAESKF2V", "ReadVAESKF2V",
                            "ReadVAESKF2V", mx>;
  }
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 370-386: Reusable TableGen multiclass VPseudoVAESZ / 可复用的 TableGen 多类 VPseudoVAESZ
```tablegen
}

multiclass VPseudoVAESZ {
  foreach m = MxListVF4 in {
    defvar mx = m.MX;
    defm "" : VPseudoBinaryV_S_NoMask_Zvk<m>,
              SchedBinary<"WriteVAESZV", "ReadVAESZV", "ReadVAESZV", mx>;
  }
}

multiclass VPseudoVSM3C {
  foreach m = MxListVF4 in {
    defvar mx = m.MX;
    defm _VI : VPseudoTernaryNoMask_Zvk<m.vrclass, m.vrclass, uimm5, m>,
               SchedTernary<"WriteVSM3CV", "ReadVSM3CV", "ReadVSM3CV",
                            "ReadVSM3CV", mx>;
  }
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 387-404: Reusable TableGen multiclass VPseudoVSM4K / 可复用的 TableGen 多类 VPseudoVSM4K
```tablegen
}

multiclass VPseudoVSM4K {
  foreach m = MxListVF4 in {
    defvar mx = m.MX;
    defm _VI : VPseudoBinaryNoMaskPolicy_Zvk<m.vrclass, m.vrclass, uimm5, m>,
               SchedBinary<"WriteVSM4KV", "ReadVSM4KV", "ReadVSM4KV", mx,
                           forcePassthruRead=true>;
  }
}

multiclass VPseudoVSM3ME {
  foreach m = MxListVF4 in {
    defvar mx = m.MX;
    defm _VV : VPseudoBinaryNoMaskPolicy_Zvk<m.vrclass, m.vrclass, m.vrclass, m>,
               SchedBinary<"WriteVSM3MEV", "ReadVSM3MEV", "ReadVSM3MEV", mx,
                           forcePassthruRead=true>;
  }
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 405-418: Reusable TableGen multiclass VPseudoVCLMUL_VV_VX / 可复用的 TableGen 多类 VPseudoVCLMUL_VV_VX
```tablegen
}

multiclass VPseudoVCLMUL_VV_VX {
  foreach m = MxList in {
    defvar mx = m.MX;
    defm "" : VPseudoBinaryV_VV<m>,
              SchedBinary<"WriteVCLMULV", "ReadVCLMULV", "ReadVCLMULV", mx,
                          forcePassthruRead=true>;
    defm "" : VPseudoBinaryV_VX<m>,
              SchedBinary<"WriteVCLMULX", "ReadVCLMULV", "ReadVCLMULX", mx,
                          forcePassthruRead=true>;
  }
}
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 419-433: Reusable TableGen multiclass VPseudoUnaryV_V / 可复用的 TableGen 多类 VPseudoUnaryV_V
```tablegen
multiclass VPseudoUnaryV_V<LMULInfo m> {
  let VLMul = m.value in {
    defvar suffix = "_V_" # m.MX;
    def suffix : VPseudoUnaryNoMask<m.vrclass, m.vrclass>;
    def suffix # "_MASK" : VPseudoUnaryMask<m.vrclass, m.vrclass>,
                                            RISCVMaskedPseudo<MaskIdx=2>;
  }
}

multiclass VPseudoVBREV {
  foreach m = MxList in {
    defvar mx = m.MX;
    defm "" : VPseudoUnaryV_V<m>,
              SchedUnary<"WriteVBREVV", "ReadVBREVV", mx, forcePassthruRead=true>;
  }
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 434-449: Reusable TableGen multiclass VPseudoVCLZ / 可复用的 TableGen 多类 VPseudoVCLZ
```tablegen
}

multiclass VPseudoVCLZ {
  foreach m = MxList in {
    defvar mx = m.MX;
    defm "" : VPseudoUnaryV_V<m>,
              SchedUnary<"WriteVCLZV", "ReadVCLZV", mx, forcePassthruRead=true>;
  }
}

multiclass VPseudoVCTZ {
  foreach m = MxList in {
    defvar mx = m.MX;
    defm "" : VPseudoUnaryV_V<m>,
              SchedUnary<"WriteVCTZV", "ReadVCTZV", mx, forcePassthruRead=true>;
  }
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 450-472: Reusable TableGen multiclass VPseudoVCPOP / 可复用的 TableGen 多类 VPseudoVCPOP
```tablegen
}

multiclass VPseudoVCPOP {
  foreach m = MxList in {
    defvar mx = m.MX;
    defm "" : VPseudoUnaryV_V<m>,
              SchedUnary<"WriteVCPOPV", "ReadVCPOPV", mx, forcePassthruRead=true>;
  }
}

multiclass VPseudoVWSLL {
  foreach m = MxListW in {
    defvar mx = m.MX;
    defm "" : VPseudoBinaryW_VV<m>,
              SchedBinary<"WriteVWSLLV", "ReadVWSLLV", "ReadVWSLLV", mx,
                          forcePassthruRead=true>;
    defm "" : VPseudoBinaryW_VX<m>,
              SchedBinary<"WriteVWSLLX", "ReadVWSLLV", "ReadVWSLLX", mx,
                          forcePassthruRead=true>;
    defm "" : VPseudoBinaryW_VI<uimm5, m>,
              SchedUnary<"WriteVWSLLI", "ReadVWSLLV", mx,
                         forcePassthruRead=true>;
  }
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 473-491: Reusable TableGen multiclass VPseudoVANDN / 可复用的 TableGen 多类 VPseudoVANDN
```tablegen
}

multiclass VPseudoVANDN {
 foreach m = MxList in {
    defm "" : VPseudoBinaryV_VV<m>,
              SchedBinary<"WriteVIALUV", "ReadVIALUV", "ReadVIALUV", m.MX,
                          forcePassthruRead=true>;
    defm "" : VPseudoBinaryV_VX<m>,
              SchedBinary<"WriteVIALUX", "ReadVIALUV", "ReadVIALUX", m.MX,
                          forcePassthruRead=true>;
  }
}

multiclass VPseudoVBREV8 {
  foreach m = MxList in {
    defvar mx = m.MX;
    defm "" : VPseudoUnaryV_V<m>,
              SchedUnary<"WriteVBREV8V", "ReadVBREV8V", mx, forcePassthruRead=true>;
  }
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 492-510: Reusable TableGen multiclass VPseudoVREV8 / 可复用的 TableGen 多类 VPseudoVREV8
```tablegen
}

multiclass VPseudoVREV8 {
  foreach m = MxList in {
    defvar mx = m.MX;
    defm "" : VPseudoUnaryV_V<m>,
              SchedUnary<"WriteVREV8V", "ReadVREV8V", mx, forcePassthruRead=true>;
  }
}

multiclass VPseudoVROT_VV_VX {
 foreach m = MxList in {
    defm "" : VPseudoBinaryV_VV<m>,
              SchedBinary<"WriteVRotV", "ReadVRotV", "ReadVRotV", m.MX,
                          forcePassthruRead=true>;
    defm "" : VPseudoBinaryV_VX<m>,
              SchedBinary<"WriteVRotX", "ReadVRotV", "ReadVRotX", m.MX,
                          forcePassthruRead=true>;
  }
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 511-529: Reusable TableGen multiclass VPseudoVROT_VV_VX_VI / 可复用的 TableGen 多类 VPseudoVROT_VV_VX_VI
```tablegen
}

multiclass VPseudoVROT_VV_VX_VI
   : VPseudoVROT_VV_VX {
  foreach m = MxList in {
    defm "" : VPseudoBinaryV_VI<uimm6, m>,
              SchedUnary<"WriteVRotI", "ReadVRotV", m.MX,
                         forcePassthruRead=true>;
  }
}

let Predicates = [HasStdExtZvbb] in {
  defm PseudoVBREV  : VPseudoVBREV;
  defm PseudoVCLZ   : VPseudoVCLZ;
  defm PseudoVCTZ   : VPseudoVCTZ;
  defm PseudoVCPOP  : VPseudoVCPOP;
  defm PseudoVWSLL : VPseudoVWSLL;
} // Predicates = [HasStdExtZvbb]
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 530-547: Bulk record instantiation PseudoVCLMUL / 批量记录实例化 PseudoVCLMUL
```tablegen
let Predicates = [HasStdExtZvbcOrZvbc32e] in {
  defm PseudoVCLMUL  : VPseudoVCLMUL_VV_VX;
  defm PseudoVCLMULH : VPseudoVCLMUL_VV_VX;
} // Predicates = [HasStdExtZvbc]

let Predicates = [HasStdExtZvkb] in {
  defm PseudoVANDN  : VPseudoVANDN;
  defm PseudoVBREV8 : VPseudoVBREV8;
  defm PseudoVREV8  : VPseudoVREV8;
  defm PseudoVROL   : VPseudoVROT_VV_VX;
  defm PseudoVROR   : VPseudoVROT_VV_VX_VI;
} // Predicates = [HasStdExtZvkb]

let Predicates = [HasStdExtZvkg] in {
  defm PseudoVGHSH : VPseudoVGHSH;
  defm PseudoVGMUL : VPseudoVGMUL;
} // Predicates = [HasStdExtZvkg]
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 548-565: Bulk record instantiation PseudoVAESDF / 批量记录实例化 PseudoVAESDF
```tablegen
let Predicates = [HasStdExtZvkned] in {
  defm PseudoVAESDF  : VPseudoVAESMV;
  defm PseudoVAESDM  : VPseudoVAESMV;
  defm PseudoVAESEF  : VPseudoVAESMV;
  defm PseudoVAESEM  : VPseudoVAESMV;
  defm PseudoVAESKF1 : VPseudoVAESKF1;
  defm PseudoVAESKF2 : VPseudoVAESKF2;
  defm PseudoVAESZ   : VPseudoVAESZ;
} // Predicates = [HasStdExtZvkned]

let Predicates = [HasStdExtZvknha] in {
  defm PseudoVSHA2CH : VPseudoVSHA2CH;
  defm PseudoVSHA2CL : VPseudoVSHA2CL;
  defm PseudoVSHA2MS : VPseudoVSHA2MS<sew=32>;
  let Predicates = [HasStdExtZvknhb] in
  defm PseudoVSHA2MS : VPseudoVSHA2MS<sew=64>;
} // Predicates = [HasStdExtZvknha]
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 566-579: Bulk record instantiation PseudoVSM4K / 批量记录实例化 PseudoVSM4K
```tablegen
let Predicates = [HasStdExtZvksed] in {
  defm PseudoVSM4K : VPseudoVSM4K;
  defm PseudoVSM4R : VPseudoVSM4R;
} // Predicates = [HasStdExtZvksed]

let Predicates = [HasStdExtZvksh] in {
  defm PseudoVSM3C  : VPseudoVSM3C;
  defm PseudoVSM3ME : VPseudoVSM3ME;
} // Predicates = [HasStdExtZvksh]

//===----------------------------------------------------------------------===//
// SDNode patterns
//===----------------------------------------------------------------------===//
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 580-593: Reusable TableGen multiclass VPatUnarySDNode_V / 可复用的 TableGen 多类 VPatUnarySDNode_V
```tablegen
multiclass VPatUnarySDNode_V<SDPatternOperator op, string instruction_name,
                             Predicate predicate = HasStdExtZvbb> {
  foreach vti = AllIntegerVectors in {
    let Predicates = !listconcat([predicate],
                                 GetVTypePredicates<vti>.Predicates) in {
      def : Pat<(vti.Vector (op (vti.Vector vti.RegClass:$rs1))),
                (!cast<Instruction>(instruction_name#"_V_"#vti.LMul.MX)
                   (vti.Vector (IMPLICIT_DEF)),
                   vti.RegClass:$rs1,
                   vti.AVL, vti.Log2SEW, TA_MA)>;
    }
  }
}
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 594-621: TableGen record riscv_splat_vector / TableGen 记录 riscv_splat_vector
```tablegen
// Helpers for detecting splats since we preprocess splat_vector to vmv.v.x
// This should match the logic in RISCVDAGToDAGISel::selectVSplat
def riscv_splat_vector : PatFrag<(ops node:$rs1),
                                 (riscv_vmv_v_x_vl undef, node:$rs1, srcvalue)>;
def allonessew8  : ImmLeaf<XLenVT, "return SignExtend64<8>(Imm) == -1LL;">;
def allonessew16 : ImmLeaf<XLenVT, "return SignExtend64<16>(Imm) == -1LL;">;
def allonessew32 : ImmLeaf<XLenVT, "return SignExtend64<32>(Imm) == -1LL;">;
def allonessew64 : ImmLeaf<XLenVT, "return Imm == -1LL;">;

foreach vti = AllIntegerVectors in {
  let Predicates = !listconcat([HasStdExtZvkb],
                               GetVTypePredicates<vti>.Predicates) in {
    def : Pat<(vti.Vector (and (xor vti.RegClass:$rs1,
                                    (riscv_splat_vector !cast<ImmLeaf>("allonessew"#vti.SEW))),
                               vti.RegClass:$rs2)),
              (!cast<Instruction>("PseudoVANDN_VV_"#vti.LMul.MX)
                 (vti.Vector (IMPLICIT_DEF)),
                 vti.RegClass:$rs2,
                 vti.RegClass:$rs1,
                 vti.AVL, vti.Log2SEW, TA_MA)>;
    def : Pat<(vti.Vector (and (riscv_splat_vector
                                 (not vti.ScalarRegClass:$rs1)),
                               vti.RegClass:$rs2)),
              (!cast<Instruction>("PseudoVANDN_VX_"#vti.LMul.MX)
                 (vti.Vector (IMPLICIT_DEF)),
                 vti.RegClass:$rs2,
                 vti.ScalarRegClass:$rs1,
                 vti.AVL, vti.Log2SEW, TA_MA)>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 622-637: Bulk record instantiation / 批量记录实例化
```tablegen
    def : Pat<(vti.Vector (and (riscv_splat_vector invLogicImm:$rs1),
                               vti.RegClass:$rs2)),
              (!cast<Instruction>("PseudoVANDN_VX_"#vti.LMul.MX)
                 (vti.Vector (IMPLICIT_DEF)),
                 vti.RegClass:$rs2,
                 invLogicImm:$rs1,
                 vti.AVL, vti.Log2SEW, TA_MA)>;
  }
}

defm : VPatUnarySDNode_V<bitreverse, "PseudoVBREV">;
defm : VPatUnarySDNode_V<bswap, "PseudoVREV8", HasStdExtZvkb>;
defm : VPatUnarySDNode_V<ctlz, "PseudoVCLZ">;
defm : VPatUnarySDNode_V<cttz, "PseudoVCTZ">;
defm : VPatUnarySDNode_V<ctpop, "PseudoVCPOP">;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 638-657: Bulk record instantiation InvRot8Imm / 批量记录实例化 InvRot8Imm
```tablegen
defm : VPatBinarySDNode_VV_VX<rotl, "PseudoVROL", ExtraPreds=[HasStdExtZvkb]>;

// Invert the immediate and mask it to SEW for readability.
def InvRot8Imm : SDNodeXForm<imm, [{
  return CurDAG->getTargetConstant(0x7 & (64 - N->getZExtValue()), SDLoc(N),
                                   N->getValueType(0));
}]>;
def InvRot16Imm : SDNodeXForm<imm, [{
  return CurDAG->getTargetConstant(0xf & (64 - N->getZExtValue()), SDLoc(N),
                                   N->getValueType(0));
}]>;
def InvRot32Imm : SDNodeXForm<imm, [{
  return CurDAG->getTargetConstant(0x1f & (64 - N->getZExtValue()), SDLoc(N),
                                   N->getValueType(0));
}]>;
def InvRot64Imm : SDNodeXForm<imm, [{
  return CurDAG->getTargetConstant(0x3f & (64 - N->getZExtValue()), SDLoc(N),
                                   N->getValueType(0));
}]>;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 658-671: TableGen record vti / TableGen 记录 vti
```tablegen
// Although there is no vrol.vi, an immediate rotate left can be achieved by
// negating the immediate in vror.vi
foreach vti = AllIntegerVectors in {
  let Predicates = !listconcat([HasStdExtZvkb],
                               GetVTypePredicates<vti>.Predicates) in {
    def : Pat<(vti.Vector (rotl vti.RegClass:$rs2,
                                (vti.Vector (SplatPat_uimm6 uimm6:$rs1)))),
              (!cast<Instruction>("PseudoVROR_VI_"#vti.LMul.MX)
                 (vti.Vector (IMPLICIT_DEF)),
                 vti.RegClass:$rs2,
                 (!cast<SDNodeXForm>("InvRot" # vti.SEW # "Imm") uimm6:$rs1),
                 vti.AVL, vti.Log2SEW, TA_MA)>;
  }
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 672-686: Bulk record instantiation vtiToWti / 批量记录实例化 vtiToWti
```tablegen
defm : VPatBinarySDNode_VV_VX_VI<rotr, "PseudoVROR", uimm6, ExtraPreds=[HasStdExtZvkb]>;

foreach vtiToWti = AllWidenableIntVectors in {
  defvar vti = vtiToWti.Vti;
  defvar wti = vtiToWti.Wti;
  let Predicates = !listconcat([HasStdExtZvbb],
                               GetVTypePredicates<vti>.Predicates,
                               GetVTypePredicates<wti>.Predicates) in {
    def : Pat<(shl (wti.Vector (zext_oneuse (vti.Vector vti.RegClass:$rs2))),
                   (wti.Vector (ext_oneuse (vti.Vector vti.RegClass:$rs1)))),
              (!cast<Instruction>("PseudoVWSLL_VV_"#vti.LMul.MX)
                 (wti.Vector (IMPLICIT_DEF)),
                 vti.RegClass:$rs2, vti.RegClass:$rs1,
                 vti.AVL, vti.Log2SEW, TA_MA)>;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 687-700: TableGen record definition / TableGen 记录定义
```tablegen
    def : Pat<(shl (wti.Vector (zext_oneuse (vti.Vector vti.RegClass:$rs2))),
                   (wti.Vector (Low8BitsSplatPat (XLenVT GPR:$rs1)))),
              (!cast<Instruction>("PseudoVWSLL_VX_"#vti.LMul.MX)
                 (wti.Vector (IMPLICIT_DEF)),
                 vti.RegClass:$rs2, GPR:$rs1,
                 vti.AVL, vti.Log2SEW, TA_MA)>;

    def : Pat<(shl (wti.Vector (zext_oneuse (vti.Vector vti.RegClass:$rs2))),
                   (wti.Vector (SplatPat_uimm5 uimm5:$rs1))),
              (!cast<Instruction>("PseudoVWSLL_VI_"#vti.LMul.MX)
                 (wti.Vector (IMPLICIT_DEF)),
                 vti.RegClass:$rs2, uimm5:$rs1,
                 vti.AVL, vti.Log2SEW, TA_MA)>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 701-716: TableGen record definition / TableGen 记录定义
```tablegen
    // Patterns where the LHS is an any_extend.
    // TODO: vv pattern when we have a test.
    def : Pat<(shl (wti.Vector (anyext_oneuse (vti.Vector vti.RegClass:$rs2))),
                   (wti.Vector (Low8BitsSplatPat (XLenVT GPR:$rs1)))),
              (!cast<Instruction>("PseudoVWSLL_VX_"#vti.LMul.MX)
                 (wti.Vector (IMPLICIT_DEF)),
                 vti.RegClass:$rs2, GPR:$rs1,
                 vti.AVL, vti.Log2SEW, TA_MA)>;

    def : Pat<(shl (wti.Vector (anyext_oneuse (vti.Vector vti.RegClass:$rs2))),
                   (wti.Vector (SplatPat_uimm5 uimm5:$rs1))),
              (!cast<Instruction>("PseudoVWSLL_VI_"#vti.LMul.MX)
                 (wti.Vector (IMPLICIT_DEF)),
                 vti.RegClass:$rs2, uimm5:$rs1,
                 vti.AVL, vti.Log2SEW, TA_MA)>;
  }
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 717-730: Bulk record instantiation / 批量记录实例化
```tablegen
}

defm : VPatBinarySDNode_VV_VX<clmul, "PseudoVCLMUL", I64IntegerVectors,
                              ExtraPreds=[HasStdExtZvbc]>;
defm : VPatBinarySDNode_VV_VX<clmulh, "PseudoVCLMULH", I64IntegerVectors,
                              ExtraPreds=[HasStdExtZvbc]>;

defvar NonI64IntegerVectors = !filter(vti, AllIntegerVectors, !le(vti.SEW, 32));
defm : VPatBinarySDNode_VV_VX<clmul, "PseudoVCLMUL", NonI64IntegerVectors,
                              ExtraPreds=[HasStdExtZvbc32e]>;
defm : VPatBinarySDNode_VV_VX<clmulh, "PseudoVCLMULH", NonI64IntegerVectors,
                              ExtraPreds=[HasStdExtZvbc32e]>;

//===----------------------------------------------------------------------===//
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 731-755: TableGen record vti / TableGen 记录 vti
```tablegen
// VL patterns
//===----------------------------------------------------------------------===//

foreach vti = AllIntegerVectors in {
  let Predicates = !listconcat([HasStdExtZvkb],
                               GetVTypePredicates<vti>.Predicates) in {
    def : Pat<(vti.Vector (riscv_and_vl (riscv_xor_vl
                                           (vti.Vector vti.RegClass:$rs1),
                                           (riscv_splat_vector !cast<ImmLeaf>("allonessew"#vti.SEW)),
                                           (vti.Vector vti.RegClass:$passthru),
                                           (vti.Mask VMV0:$vm),
                                           VLOpFrag),
                                        (vti.Vector vti.RegClass:$rs2),
                                        (vti.Vector vti.RegClass:$passthru),
                                        (vti.Mask VMV0:$vm),
                                        VLOpFrag)),
              (!cast<Instruction>("PseudoVANDN_VV_"#vti.LMul.MX#"_MASK")
                 vti.RegClass:$passthru,
                 vti.RegClass:$rs2,
                 vti.RegClass:$rs1,
                 (vti.Mask VMV0:$vm),
                 GPR:$vl,
                 vti.Log2SEW,
                 TAIL_AGNOSTIC)>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 756-770: TableGen record definition / TableGen 记录定义
```tablegen
    def : Pat<(vti.Vector (riscv_and_vl (riscv_splat_vector
                                           (not vti.ScalarRegClass:$rs1)),
                                        (vti.Vector vti.RegClass:$rs2),
                                        (vti.Vector vti.RegClass:$passthru),
                                        (vti.Mask VMV0:$vm),
                                        VLOpFrag)),
              (!cast<Instruction>("PseudoVANDN_VX_"#vti.LMul.MX#"_MASK")
                 vti.RegClass:$passthru,
                 vti.RegClass:$rs2,
                 vti.ScalarRegClass:$rs1,
                 (vti.Mask VMV0:$vm),
                 GPR:$vl,
                 vti.Log2SEW,
                 TAIL_AGNOSTIC)>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 771-784: TableGen record definition / TableGen 记录定义
```tablegen
    def : Pat<(vti.Vector (riscv_and_vl (riscv_splat_vector invLogicImm:$rs1),
                                        (vti.Vector vti.RegClass:$rs2),
                                        (vti.Vector vti.RegClass:$passthru),
                                        (vti.Mask VMV0:$vm),
                                        VLOpFrag)),
              (!cast<Instruction>("PseudoVANDN_VX_"#vti.LMul.MX#"_MASK")
                 vti.RegClass:$passthru,
                 vti.RegClass:$rs2,
                 invLogicImm:$rs1,
                 (vti.Mask VMV0:$vm),
                 GPR:$vl,
                 vti.Log2SEW,
                 TAIL_AGNOSTIC)>;
  }
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 785-808: Bulk record instantiation vti / 批量记录实例化 vti
```tablegen
}

defm : VPatUnaryVL_V<riscv_bitreverse_vl, "PseudoVBREV">;
defm : VPatUnaryVL_V<riscv_bswap_vl, "PseudoVREV8", HasStdExtZvkb>;
defm : VPatUnaryVL_V<riscv_ctlz_vl, "PseudoVCLZ">;
defm : VPatUnaryVL_V<riscv_cttz_vl, "PseudoVCTZ">;
defm : VPatUnaryVL_V<riscv_ctpop_vl, "PseudoVCPOP">;

defm : VPatBinaryVL_VV_VX<riscv_rotl_vl, "PseudoVROL", ExtraPreds=[HasStdExtZvkb]>;
// Although there is no vrol.vi, an immediate rotate left can be achieved by
// negating the immediate in vror.vi
foreach vti = AllIntegerVectors in {
  let Predicates = !listconcat([HasStdExtZvkb],
                               GetVTypePredicates<vti>.Predicates) in {
    def : Pat<(riscv_rotl_vl vti.RegClass:$rs2,
                             (vti.Vector (SplatPat_uimm6 uimm6:$rs1)),
                             (vti.Vector vti.RegClass:$passthru),
                             (vti.Mask VMV0:$vm), VLOpFrag),
              (!cast<Instruction>("PseudoVROR_VI_"#vti.LMul.MX#"_MASK")
                 vti.RegClass:$passthru,
                 vti.RegClass:$rs2,
                 (!cast<SDNodeXForm>("InvRot" # vti.SEW # "Imm") uimm6:$rs1),
                 (vti.Mask VMV0:$vm), GPR:$vl, vti.Log2SEW, TAIL_AGNOSTIC)>;
  }
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 809-826: Bulk record instantiation vtiToWti / 批量记录实例化 vtiToWti
```tablegen
}
defm : VPatBinaryVL_VV_VX_VI<riscv_rotr_vl, "PseudoVROR", uimm6, ExtraPreds=[HasStdExtZvkb]>;

foreach vtiToWti = AllWidenableIntVectors in {
  defvar vti = vtiToWti.Vti;
  defvar wti = vtiToWti.Wti;
  let Predicates = !listconcat([HasStdExtZvbb],
                               GetVTypePredicates<vti>.Predicates,
                               GetVTypePredicates<wti>.Predicates) in {
    def : Pat<(riscv_shl_vl
                 (wti.Vector (zext_oneuse (vti.Vector vti.RegClass:$rs2))),
                 (wti.Vector (ext_oneuse (vti.Vector vti.RegClass:$rs1))),
                 (wti.Vector wti.RegClass:$passthru),
                 (vti.Mask VMV0:$vm), VLOpFrag),
              (!cast<Instruction>("PseudoVWSLL_VV_"#vti.LMul.MX#"_MASK")
                 wti.RegClass:$passthru, vti.RegClass:$rs2, vti.RegClass:$rs1,
                 (vti.Mask VMV0:$vm), GPR:$vl, vti.Log2SEW, TAIL_AGNOSTIC)>;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 827-848: TableGen record definition / TableGen 记录定义
```tablegen
    def : Pat<(riscv_shl_vl
                 (wti.Vector (riscv_zext_vl_oneuse
                                (vti.Vector vti.RegClass:$rs2),
                                (vti.Mask VMV0:$vm), VLOpFrag)),
                 (wti.Vector (riscv_ext_vl_oneuse
                                (vti.Vector vti.RegClass:$rs1),
                                (vti.Mask VMV0:$vm), VLOpFrag)),
                 (wti.Vector wti.RegClass:$passthru),
                 (vti.Mask VMV0:$vm), VLOpFrag),
              (!cast<Instruction>("PseudoVWSLL_VV_"#vti.LMul.MX#"_MASK")
                 wti.RegClass:$passthru, vti.RegClass:$rs2, vti.RegClass:$rs1,
                 (vti.Mask VMV0:$vm), GPR:$vl, vti.Log2SEW, TAIL_AGNOSTIC)>;

    def : Pat<(riscv_shl_vl
                 (wti.Vector (zext_oneuse (vti.Vector vti.RegClass:$rs2))),
                 (wti.Vector (Low8BitsSplatPat (XLenVT GPR:$rs1))),
                 (wti.Vector wti.RegClass:$passthru),
                 (vti.Mask VMV0:$vm), VLOpFrag),
              (!cast<Instruction>("PseudoVWSLL_VX_"#vti.LMul.MX#"_MASK")
                 wti.RegClass:$passthru, vti.RegClass:$rs2, GPR:$rs1,
                 (vti.Mask VMV0:$vm), GPR:$vl, vti.Log2SEW, TAIL_AGNOSTIC)>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 849-868: TableGen record definition / TableGen 记录定义
```tablegen
    def : Pat<(riscv_shl_vl
                 (wti.Vector (riscv_zext_vl_oneuse
                                (vti.Vector vti.RegClass:$rs2),
                                (vti.Mask VMV0:$vm), VLOpFrag)),
                 (wti.Vector (Low8BitsSplatPat (XLenVT GPR:$rs1))),
                 (wti.Vector wti.RegClass:$passthru),
                 (vti.Mask VMV0:$vm), VLOpFrag),
              (!cast<Instruction>("PseudoVWSLL_VX_"#vti.LMul.MX#"_MASK")
                 wti.RegClass:$passthru, vti.RegClass:$rs2, GPR:$rs1,
                 (vti.Mask VMV0:$vm), GPR:$vl, vti.Log2SEW, TAIL_AGNOSTIC)>;

    def : Pat<(riscv_shl_vl
                 (wti.Vector (zext_oneuse (vti.Vector vti.RegClass:$rs2))),
                 (wti.Vector (SplatPat_uimm5 uimm5:$rs1)),
                 (wti.Vector wti.RegClass:$passthru),
                 (vti.Mask VMV0:$vm), VLOpFrag),
              (!cast<Instruction>("PseudoVWSLL_VI_"#vti.LMul.MX#"_MASK")
                 wti.RegClass:$passthru, vti.RegClass:$rs2, uimm5:$rs1,
                 (vti.Mask VMV0:$vm), GPR:$vl, vti.Log2SEW, TAIL_AGNOSTIC)>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 869-888: TableGen record definition / TableGen 记录定义
```tablegen
    def : Pat<(riscv_shl_vl
                 (wti.Vector (riscv_zext_vl_oneuse
                                (vti.Vector vti.RegClass:$rs2),
                                (vti.Mask VMV0:$vm), VLOpFrag)),
                 (wti.Vector (SplatPat_uimm5 uimm5:$rs1)),
                 (wti.Vector wti.RegClass:$passthru),
                 (vti.Mask VMV0:$vm), VLOpFrag),
              (!cast<Instruction>("PseudoVWSLL_VI_"#vti.LMul.MX#"_MASK")
                 wti.RegClass:$passthru, vti.RegClass:$rs2, uimm5:$rs1,
                 (vti.Mask VMV0:$vm), GPR:$vl, vti.Log2SEW, TAIL_AGNOSTIC)>;

    def : Pat<(riscv_vwsll_vl
                 (vti.Vector vti.RegClass:$rs2),
                 (vti.Vector vti.RegClass:$rs1),
                 (wti.Vector wti.RegClass:$passthru),
                 (vti.Mask VMV0:$vm), VLOpFrag),
              (!cast<Instruction>("PseudoVWSLL_VV_"#vti.LMul.MX#"_MASK")
                 wti.RegClass:$passthru, vti.RegClass:$rs2, vti.RegClass:$rs1,
                 (vti.Mask VMV0:$vm), GPR:$vl, vti.Log2SEW, TAIL_AGNOSTIC)>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 889-906: TableGen record definition / TableGen 记录定义
```tablegen
    def : Pat<(riscv_vwsll_vl
                 (vti.Vector vti.RegClass:$rs2),
                 (vti.Vector (Low8BitsSplatPat (XLenVT GPR:$rs1))),
                 (wti.Vector wti.RegClass:$passthru),
                 (vti.Mask VMV0:$vm), VLOpFrag),
              (!cast<Instruction>("PseudoVWSLL_VX_"#vti.LMul.MX#"_MASK")
                 wti.RegClass:$passthru, vti.RegClass:$rs2, GPR:$rs1,
                 (vti.Mask VMV0:$vm), GPR:$vl, vti.Log2SEW, TAIL_AGNOSTIC)>;

    def : Pat<(riscv_vwsll_vl
                 (vti.Vector vti.RegClass:$rs2),
                 (vti.Vector (SplatPat_uimm5 uimm5:$rs1)),
                 (wti.Vector wti.RegClass:$passthru),
                 (vti.Mask VMV0:$vm), VLOpFrag),
              (!cast<Instruction>("PseudoVWSLL_VI_"#vti.LMul.MX#"_MASK")
                 wti.RegClass:$passthru, vti.RegClass:$rs2, uimm5:$rs1,
                 (vti.Mask VMV0:$vm), GPR:$vl, vti.Log2SEW, TAIL_AGNOSTIC)>;
  }
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 907-930: TableGen class VPatUnaryNoMask_Zvk<string / TableGen 类 VPatUnaryNoMask_Zvk<string
```tablegen
}

//===----------------------------------------------------------------------===//
// Codegen patterns
//===----------------------------------------------------------------------===//

class VPatUnaryNoMask_Zvk<string intrinsic_name,
                          string inst,
                          string kind,
                          ValueType result_type,
                          ValueType op2_type,
                          int sew,
                          LMULInfo vlmul,
                          VReg result_reg_class,
                          VReg op2_reg_class> :
  Pat<(result_type (!cast<Intrinsic>(intrinsic_name)
                   (result_type result_reg_class:$rd),
                   (op2_type op2_reg_class:$rs2),
                   VLOpFrag, (XLenVT timm:$policy))),
                   (!cast<Instruction>(inst#"_"#kind#"_"#vlmul.MX)
                   (result_type result_reg_class:$rd),
                   (op2_type op2_reg_class:$rs2),
                   GPR:$vl, sew, (XLenVT timm:$policy))>;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 931-949: TableGen class VPatUnaryNoMask_VS_Zvk<string / TableGen 类 VPatUnaryNoMask_VS_Zvk<string
```tablegen
class VPatUnaryNoMask_VS_Zvk<string intrinsic_name,
                             string inst,
                             string kind,
                             ValueType result_type,
                             ValueType op2_type,
                             int sew,
                             LMULInfo vlmul,
                             LMULInfo vs2_lmul,
                             VReg result_reg_class,
                             VReg op2_reg_class> :
  Pat<(result_type (!cast<Intrinsic>(intrinsic_name)
                   (result_type result_reg_class:$rd),
                   (op2_type op2_reg_class:$rs2),
                   VLOpFrag, (XLenVT timm:$policy))),
                   (!cast<Instruction>(inst#"_"#kind#"_"#vlmul.MX#"_"#vs2_lmul.MX)
                   (result_type result_reg_class:$rd),
                   (op2_type op2_reg_class:$rs2),
                   GPR:$vl, sew, (XLenVT timm:$policy))>;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 950-971: Reusable TableGen multiclass VPatUnaryV_V_NoMask_Zvk / 可复用的 TableGen 多类 VPatUnaryV_V_NoMask_Zvk
```tablegen
multiclass VPatUnaryV_V_NoMask_Zvk<string intrinsic, string instruction,
                                   list<VTypeInfo> vtilist,
                                   list<Predicate> ExtraPreds> {
  foreach vti = vtilist in
    let Predicates = !listconcat(ExtraPreds, GetVTypePredicates<vti>.Predicates) in
    def : VPatUnaryNoMask_Zvk<intrinsic # "_vv", instruction, "VV",
                              vti.Vector, vti.Vector, vti.Log2SEW,
                              vti.LMul, vti.RegClass, vti.RegClass>;
}

multiclass VPatUnaryV_S_NoMaskVectorCrypto<string intrinsic, string instruction,
                                           list<VTypeInfo> vtilist,
                                           list<Predicate> ExtraPreds> {
  foreach vti = vtilist in
    foreach vti_vs2 = ZvkI32IntegerVectors<vti.LMul.MX>.vs2_types in
      let Predicates = !listconcat(ExtraPreds,
                                   GetVTypePredicates<vti>.Predicates,
                                   GetVTypePredicates<vti_vs2>.Predicates) in
      def : VPatUnaryNoMask_VS_Zvk<intrinsic # "_vs", instruction, "VS",
                            vti.Vector, vti_vs2.Vector, vti.Log2SEW,
                            vti.LMul, vti_vs2.LMul, vti.RegClass, vti_vs2.RegClass>;
}
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 972-992: Reusable TableGen multiclass VPatUnaryV_V_S_NoMask_Zvk / 可复用的 TableGen 多类 VPatUnaryV_V_S_NoMask_Zvk
```tablegen

multiclass VPatUnaryV_V_S_NoMask_Zvk<string intrinsic, string instruction,
                                     list<VTypeInfo> vtilist,
                                     list<Predicate> ExtraPreds> {
  defm : VPatUnaryV_V_NoMask_Zvk<intrinsic, instruction, vtilist, ExtraPreds>;
  defm : VPatUnaryV_S_NoMaskVectorCrypto<intrinsic, instruction, vtilist,
                                         ExtraPreds>;
}

multiclass VPatBinaryV_VV_NoMask<string intrinsic, string instruction,
                                 list<VTypeInfo> vtilist,
                                 list<Predicate> ExtraPreds,
                                 bit isSEWAware = false> {
  foreach vti = vtilist in
    let Predicates = !listconcat(ExtraPreds, GetVTypePredicates<vti>.Predicates) in
    def : VPatTernaryNoMaskWithPolicy<intrinsic, instruction, "VV",
                                      vti.Vector, vti.Vector, vti.Vector,
                                      vti.Log2SEW, vti.LMul, vti.RegClass,
                                      vti.RegClass, vti.RegClass,
                                      isSEWAware = isSEWAware>;
}
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 993-1015: Reusable TableGen multiclass VPatBinaryV_VI_NoMask / 可复用的 TableGen 多类 VPatBinaryV_VI_NoMask
```tablegen

multiclass VPatBinaryV_VI_NoMask<string intrinsic, string instruction,
                                 list<VTypeInfo> vtilist,
                                 list<Predicate> ExtraPreds,
                                 Operand imm_type = tuimm5> {
  foreach vti = vtilist in
    let Predicates = !listconcat(ExtraPreds, GetVTypePredicates<vti>.Predicates) in
    def : VPatTernaryNoMaskWithPolicy<intrinsic, instruction, "VI",
                                      vti.Vector, vti.Vector, XLenVT,
                                      vti.Log2SEW, vti.LMul, vti.RegClass,
                                      vti.RegClass, imm_type>;
}

multiclass VPatBinaryV_VI_NoMaskTU<string intrinsic, string instruction,
                                   list<VTypeInfo> vtilist,
                                   list<Predicate> ExtraPreds,
                                   Operand imm_type = tuimm5> {
  foreach vti = vtilist in
    let Predicates = !listconcat(ExtraPreds, GetVTypePredicates<vti>.Predicates) in
    def : VPatBinaryNoMaskTU<intrinsic, instruction # "_VI_" # vti.LMul.MX,
                             vti.Vector, vti.Vector, XLenVT, vti.Log2SEW,
                             vti.RegClass, vti.RegClass, imm_type>;
}
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 1016-1039: Reusable TableGen multiclass VPatBinaryV_VV_NoMaskTU / 可复用的 TableGen 多类 VPatBinaryV_VV_NoMaskTU
```tablegen

multiclass VPatBinaryV_VV_NoMaskTU<string intrinsic, string instruction,
                                   list<VTypeInfo> vtilist,
                                   list<Predicate> ExtraPreds> {
  foreach vti = vtilist in
    let Predicates = !listconcat(ExtraPreds, GetVTypePredicates<vti>.Predicates) in
    def : VPatBinaryNoMaskTU<intrinsic, instruction # "_VV_" # vti.LMul.MX,
                             vti.Vector, vti.Vector, vti.Vector, vti.Log2SEW,
                             vti.RegClass, vti.RegClass, vti.RegClass>;
}

multiclass VPatBinaryV_VX_VROTATE<string intrinsic, string instruction,
                                  list<VTypeInfo> vtilist, bit isSEWAware = 0> {
  foreach vti = vtilist in {
    defvar kind = "V"#vti.ScalarSuffix;
    let Predicates = GetVTypePredicates<vti>.Predicates in
    defm : VPatBinary<intrinsic,
                      !if(isSEWAware,
                          instruction#"_"#kind#"_"#vti.LMul.MX#"_E"#vti.SEW,
                          instruction#"_"#kind#"_"#vti.LMul.MX),
                      vti.Vector, vti.Vector, XLenVT, vti.Mask,
                      vti.Log2SEW, vti.RegClass,
                      vti.RegClass, vti.ScalarRegClass>;
  }
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 1040-1060: Reusable TableGen multiclass VPatBinaryV_VI_VROL / 可复用的 TableGen 多类 VPatBinaryV_VI_VROL
```tablegen
}

multiclass VPatBinaryV_VI_VROL<string intrinsic, string instruction,
                               list<VTypeInfo> vtilist, bit isSEWAware = 0,
                               list<Predicate> ExtraPreds = []> {
  foreach vti = vtilist in {
    defvar Intr = !cast<Intrinsic>(intrinsic);
    defvar Pseudo = !cast<Instruction>(
        !if(isSEWAware, instruction#"_VI_"#vti.LMul.MX#"_E"#vti.SEW,
                        instruction#"_VI_"#vti.LMul.MX));
    defvar InvRotImm = !cast<SDNodeXForm>("InvRot" # vti.SEW # "Imm");
    let Predicates = !listconcat(ExtraPreds, GetVTypePredicates<vti>.Predicates) in
    def : Pat<(vti.Vector (Intr (vti.Vector vti.RegClass:$passthru),
                          (vti.Vector vti.RegClass:$rs2),
                          (XLenVT uimm6:$rs1),
                          VLOpFrag)),
                          (Pseudo (vti.Vector vti.RegClass:$passthru),
                          (vti.Vector vti.RegClass:$rs2),
                          (InvRotImm uimm6:$rs1),
                          GPR:$vl, vti.Log2SEW, TU_MU)>;
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 1061-1076: TableGen record definition / TableGen 记录定义
```tablegen
    defvar IntrMask = !cast<Intrinsic>(intrinsic#"_mask");
    defvar PseudoMask = !cast<Instruction>(
        !if(isSEWAware, instruction#"_VI_"#vti.LMul.MX#"_E"#vti.SEW#"_MASK",
                        instruction#"_VI_"#vti.LMul.MX#"_MASK"));
    let Predicates = !listconcat(ExtraPreds, GetVTypePredicates<vti>.Predicates) in
    def : Pat<(vti.Vector (IntrMask (vti.Vector vti.RegClass:$passthru),
                          (vti.Vector vti.RegClass:$rs2),
                          (XLenVT uimm6:$rs1),
                          (vti.Mask VMV0:$vm),
                          VLOpFrag, (XLenVT timm:$policy))),
                          (PseudoMask (vti.Vector vti.RegClass:$passthru),
                          (vti.Vector vti.RegClass:$rs2),
                          (InvRotImm uimm6:$rs1),
                          (vti.Mask VMV0:$vm),
                          GPR:$vl, vti.Log2SEW, (XLenVT timm:$policy))>;
    }
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 1077-1104: Reusable TableGen multiclass VPatBinaryV_VV_VX_VROL / 可复用的 TableGen 多类 VPatBinaryV_VV_VX_VROL
```tablegen
}

multiclass VPatBinaryV_VV_VX_VROL<string intrinsic, string instruction,
                                  string instruction2, list<VTypeInfo> vtilist,
                                  list<Predicate> ExtraPreds>
    : VPatBinaryV_VV_VX<intrinsic, instruction, vtilist, ExtraPreds=ExtraPreds>,
      VPatBinaryV_VI_VROL<intrinsic, instruction2, vtilist, ExtraPreds=ExtraPreds>;

multiclass VPatBinaryW_VV_VX_VI_VWSLL<string intrinsic, string instruction,
                                      list<VTypeInfoToWide> vtilist,
                                      list<Predicate> ExtraPreds>
    : VPatBinaryW_VV<intrinsic, instruction, vtilist, ExtraPreds=ExtraPreds> {
  foreach VtiToWti = vtilist in {
    defvar Vti = VtiToWti.Vti;
    defvar Wti = VtiToWti.Wti;
    defvar kind = "V"#Vti.ScalarSuffix;
    let Predicates = !listconcat(ExtraPreds,
                                 GetVTypePredicates<Vti>.Predicates,
                                 GetVTypePredicates<Wti>.Predicates) in {
      defm : VPatBinary<intrinsic, instruction#"_"#kind#"_"#Vti.LMul.MX,
                        Wti.Vector, Vti.Vector, XLenVT, Vti.Mask,
                        Vti.Log2SEW, Wti.RegClass,
                        Vti.RegClass, Vti.ScalarRegClass>;
      defm : VPatBinary<intrinsic, instruction # "_VI_" # Vti.LMul.MX,
                        Wti.Vector, Vti.Vector, XLenVT, Vti.Mask,
                        Vti.Log2SEW, Wti.RegClass,
                        Vti.RegClass, uimm5>;
    }
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 1105-1122: Bulk record instantiation / 批量记录实例化
```tablegen
  }
}

defm : VPatUnaryV_V<"int_riscv_vbrev", "PseudoVBREV", AllIntegerVectors, ExtraPreds=[HasStdExtZvbb]>;
defm : VPatUnaryV_V<"int_riscv_vclz", "PseudoVCLZ", AllIntegerVectors, ExtraPreds=[HasStdExtZvbb]>;
defm : VPatUnaryV_V<"int_riscv_vctz", "PseudoVCTZ", AllIntegerVectors, ExtraPreds=[HasStdExtZvbb]>;
defm : VPatUnaryV_V<"int_riscv_vcpopv", "PseudoVCPOP", AllIntegerVectors, ExtraPreds=[HasStdExtZvbb]>;
defm : VPatBinaryW_VV_VX_VI_VWSLL<"int_riscv_vwsll", "PseudoVWSLL", AllWidenableIntVectors, [HasStdExtZvbb]>;

defm : VPatBinaryV_VV_VX<"int_riscv_vclmul", "PseudoVCLMUL", I64IntegerVectors, ExtraPreds=[HasStdExtZvbc]>;
defm : VPatBinaryV_VV_VX<"int_riscv_vclmulh", "PseudoVCLMULH", I64IntegerVectors, ExtraPreds=[HasStdExtZvbc]>;

defm : VPatBinaryV_VV_VX<"int_riscv_vandn", "PseudoVANDN", AllIntegerVectors, ExtraPreds=[HasStdExtZvkb]>;
defm : VPatUnaryV_V<"int_riscv_vbrev8", "PseudoVBREV8", AllIntegerVectors, ExtraPreds=[HasStdExtZvkb]>;
defm : VPatUnaryV_V<"int_riscv_vrev8", "PseudoVREV8", AllIntegerVectors, ExtraPreds=[HasStdExtZvkb]>;
defm : VPatBinaryV_VV_VX_VROL<"int_riscv_vrol", "PseudoVROL", "PseudoVROR", AllIntegerVectors, [HasStdExtZvkb]>;
defm : VPatBinaryV_VV_VX_VI<"int_riscv_vror", "PseudoVROR", AllIntegerVectors, uimm6, ExtraPreds=[HasStdExtZvkb]>;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 1123-1137: Bulk record instantiation / 批量记录实例化
```tablegen
defm : VPatBinaryV_VV_NoMask<"int_riscv_vghsh", "PseudoVGHSH", I32IntegerVectors, [HasStdExtZvkg]>;
defm : VPatUnaryV_V_NoMask_Zvk<"int_riscv_vgmul", "PseudoVGMUL", I32IntegerVectors, [HasStdExtZvkg]>;

defm : VPatUnaryV_V_S_NoMask_Zvk<"int_riscv_vaesdf", "PseudoVAESDF", I32IntegerVectors, [HasStdExtZvkned]>;
defm : VPatUnaryV_V_S_NoMask_Zvk<"int_riscv_vaesdm", "PseudoVAESDM", I32IntegerVectors, [HasStdExtZvkned]>;
defm : VPatUnaryV_V_S_NoMask_Zvk<"int_riscv_vaesef", "PseudoVAESEF", I32IntegerVectors, [HasStdExtZvkned]>;
defm : VPatUnaryV_V_S_NoMask_Zvk<"int_riscv_vaesem", "PseudoVAESEM", I32IntegerVectors, [HasStdExtZvkned]>;
defm : VPatBinaryV_VI_NoMaskTU<"int_riscv_vaeskf1", "PseudoVAESKF1", I32IntegerVectors, [HasStdExtZvkned]>;
defm : VPatBinaryV_VI_NoMask<"int_riscv_vaeskf2", "PseudoVAESKF2", I32IntegerVectors, [HasStdExtZvkned]>;
defm : VPatUnaryV_S_NoMaskVectorCrypto<"int_riscv_vaesz", "PseudoVAESZ", I32IntegerVectors, [HasStdExtZvkned] >;

defm : VPatBinaryV_VV_NoMask<"int_riscv_vsha2ch", "PseudoVSHA2CH", I32IntegerVectors, [HasStdExtZvknha]>;
defm : VPatBinaryV_VV_NoMask<"int_riscv_vsha2cl", "PseudoVSHA2CL", I32IntegerVectors, [HasStdExtZvknha]>;
defm : VPatBinaryV_VV_NoMask<"int_riscv_vsha2ms", "PseudoVSHA2MS", I32IntegerVectors, [HasStdExtZvknha], isSEWAware=true>;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 1138-1146: Bulk record instantiation / 批量记录实例化
```tablegen
defm : VPatBinaryV_VV_NoMask<"int_riscv_vsha2ch", "PseudoVSHA2CH", I64IntegerVectors, [HasStdExtZvknhb]>;
defm : VPatBinaryV_VV_NoMask<"int_riscv_vsha2cl", "PseudoVSHA2CL", I64IntegerVectors, [HasStdExtZvknhb]>;
defm : VPatBinaryV_VV_NoMask<"int_riscv_vsha2ms", "PseudoVSHA2MS", I64IntegerVectors, [HasStdExtZvknhb], isSEWAware=true>;

defm : VPatBinaryV_VI_NoMaskTU<"int_riscv_vsm4k", "PseudoVSM4K", I32IntegerVectors, ExtraPreds=[HasStdExtZvksed]>;
defm : VPatUnaryV_V_S_NoMask_Zvk<"int_riscv_vsm4r", "PseudoVSM4R", I32IntegerVectors, [HasStdExtZvksed]>;

defm : VPatBinaryV_VI_NoMask<"int_riscv_vsm3c", "PseudoVSM3C", I32IntegerVectors, [HasStdExtZvksh]>;
defm : VPatBinaryV_VV_NoMaskTU<"int_riscv_vsm3me", "PseudoVSM3ME", I32IntegerVectors, [HasStdExtZvksh]>;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

## Key Concepts / 关键概念
- **Instruction definitions** / **指令定义**
- **TableGen DSL** / **TableGen DSL**

## Dependencies / 依赖关系
- No explicit source includes in this file; dependencies are mostly implicit through surrounding backend integration. / 该文件没有显式源码级 include，依赖主要通过周边后端集成隐式体现。
