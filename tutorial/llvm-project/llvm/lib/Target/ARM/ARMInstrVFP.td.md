# ARMInstrVFP.td — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/ARMInstrVFP.td`
- Repository: `llvm-project`
- Purpose (EN): This file describes the ARM VFP instruction set.
- 用途 (CN): 使用 LLVM TableGen DSL 定义 ARM 后端中的 `ARMInstrVFP`，涵盖指令定义与目标操作码元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```tablegen
//===-- ARMInstrVFP.td - VFP support for ARM ---------------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file describes the ARM VFP instruction set.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 13-17
```tablegen
def SDT_CMPFP : SDTypeProfile<1, 2, [
  SDTCisVT<0, FlagsVT>, // out flags
  SDTCisFP<1>,          // lhs
  SDTCisSameAs<2, 1>    // rhs
]>;
```
- EN: Defines TableGen record `SDT_CMPFP` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SDT_CMPFP`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 19-22
```tablegen
def SDT_CMPFP0 : SDTypeProfile<1, 1, [
  SDTCisVT<0, FlagsVT>, // out flags
  SDTCisFP<1>           // operand
]>;
```
- EN: Defines TableGen record `SDT_CMPFP0` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SDT_CMPFP0`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 24-27
```tablegen
def SDT_VMOVDRR : SDTypeProfile<1, 2, [SDTCisVT<0, f64>, SDTCisVT<1, i32>,
                                       SDTCisSameAs<1, 2>]>;
def SDT_VMOVRRD : SDTypeProfile<2, 1, [SDTCisVT<0, i32>, SDTCisSameAs<0, 1>,
                                       SDTCisVT<2, f64>]>;
```
- EN: Defines TableGen record `SDT_VMOVDRR` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SDT_VMOVDRR`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 29-29
```tablegen
def SDT_VMOVSR : SDTypeProfile<1, 1, [SDTCisVT<0, f32>, SDTCisVT<1, i32>]>;
```
- EN: Defines TableGen record `SDT_VMOVSR` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SDT_VMOVSR`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 31-32
```tablegen
// ARM VFP compare instruction, sets FPSCR.
def arm_cmpfp   : SDNode<"ARMISD::CMPFP",    SDT_CMPFP>;
```
- EN: Defines TableGen record `arm_cmpfp` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `arm_cmpfp`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 34-35
```tablegen
// ARM VFP compare against zero instruction, sets FPSCR.
def arm_cmpfp0  : SDNode<"ARMISD::CMPFPw0",  SDT_CMPFP0>;
```
- EN: Defines TableGen record `arm_cmpfp0` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `arm_cmpfp0`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 37-38
```tablegen
// ARM VFP signalling compare instruction, sets FPSCR.
def arm_cmpfpe  : SDNode<"ARMISD::CMPFPE",   SDT_CMPFP>;
```
- EN: Defines TableGen record `arm_cmpfpe` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `arm_cmpfpe`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 40-42
```tablegen
// ARM VFP signalling compare against zero instruction, sets
// FPSCR.
def arm_cmpfpe0 : SDNode<"ARMISD::CMPFPEw0", SDT_CMPFP0>;
```
- EN: Defines TableGen record `arm_cmpfpe0` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `arm_cmpfpe0`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 44-50
```tablegen
// ARM fmstat instruction.
def arm_fmstat : SDNode<"ARMISD::FMSTAT",
  SDTypeProfile<1, 1, [
    SDTCisVT<0, FlagsVT>, // out flags
    SDTCisVT<1, FlagsVT>  // in flags
  ]>
>;
```
- EN: Defines TableGen record `arm_fmstat` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `arm_fmstat`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 52-53
```tablegen
// Two gprs to double.
def arm_fmdrr  : SDNode<"ARMISD::VMOVDRR", SDT_VMOVDRR>;
```
- EN: Defines TableGen record `arm_fmdrr` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `arm_fmdrr`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 55-56
```tablegen
// double to two gprs.
def arm_fmrrd  : SDNode<"ARMISD::VMOVRRD", SDT_VMOVRRD>;
```
- EN: Defines TableGen record `arm_fmrrd` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `arm_fmrrd`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 58-59
```tablegen
// move gpr to single, used for f32 literal constructed in a gpr
def arm_vmovsr  : SDNode<"ARMISD::VMOVSR", SDT_VMOVSR>;
```
- EN: Defines TableGen record `arm_vmovsr` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `arm_vmovsr`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 61-62
```tablegen
def SDT_VMOVhr : SDTypeProfile<1, 1, [SDTCisFP<0>, SDTCisVT<1, i32>] >;
def SDT_VMOVrh : SDTypeProfile<1, 1, [SDTCisVT<0, i32>, SDTCisFP<1>] >;
```
- EN: Defines TableGen record `SDT_VMOVhr` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SDT_VMOVhr`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 64-66
```tablegen
// Move H <-> R, clearing top 16 bits
def arm_vmovhr : SDNode<"ARMISD::VMOVhr", SDT_VMOVhr>;
def arm_vmovrh : SDNode<"ARMISD::VMOVrh", SDT_VMOVrh>;
```
- EN: Defines TableGen record `arm_vmovhr` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `arm_vmovhr`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 68-70
```tablegen
//===----------------------------------------------------------------------===//
// Operand Definitions.
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 72-76
```tablegen
// 8-bit floating-point immediate encodings.
def FPImmOperand : AsmOperandClass {
  let Name = "FPImm";
  let ParserMethod = "parseFPImm";
}
```
- EN: Defines TableGen record `FPImmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FPImmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 78-87
```tablegen
def vfp_f16imm : Operand<f16>,
                 PatLeaf<(f16 fpimm), [{
      return ARM_AM::getFP16Imm(N->getValueAPF()) != -1;
    }], SDNodeXForm<fpimm, [{
      uint32_t Enc = ARM_AM::getFP16Imm(N->getValueAPF());
      return CurDAG->getTargetConstant(Enc, SDLoc(N), MVT::i32);
    }]>> {
  let PrintMethod = "printFPImmOperand";
  let ParserMatchClass = FPImmOperand;
}
```
- EN: Defines TableGen record `vfp_f16imm` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `vfp_f16imm`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 89-92
```tablegen
def vfp_f32f16imm_xform : SDNodeXForm<fpimm, [{
      uint32_t Enc = ARM_AM::getFP32FP16Imm(N->getValueAPF());
      return CurDAG->getTargetConstant(Enc, SDLoc(N), MVT::i32);
    }]>;
```
- EN: Defines TableGen record `vfp_f32f16imm_xform` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `vfp_f32f16imm_xform`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 94-96
```tablegen
def vfp_f32f16imm : PatLeaf<(f32 fpimm), [{
      return ARM_AM::getFP32FP16Imm(N->getValueAPF()) != -1;
    }], vfp_f32f16imm_xform>;
```
- EN: Defines TableGen record `vfp_f32f16imm` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `vfp_f32f16imm`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 98-101
```tablegen
def vfp_f32imm_xform : SDNodeXForm<fpimm, [{
      uint32_t Enc = ARM_AM::getFP32Imm(N->getValueAPF());
      return CurDAG->getTargetConstant(Enc, SDLoc(N), MVT::i32);
    }]>;
```
- EN: Defines TableGen record `vfp_f32imm_xform` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `vfp_f32imm_xform`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 103-104
```tablegen
def gi_vfp_f32imm : GICustomOperandRenderer<"renderVFPF32Imm">,
                    GISDNodeXFormEquiv<vfp_f32imm_xform>;
```
- EN: Defines TableGen record `gi_vfp_f32imm` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `gi_vfp_f32imm`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 106-118
```tablegen
def vfp_f32imm : Operand<f32>,
                 PatLeaf<(f32 fpimm), [{
      return ARM_AM::getFP32Imm(N->getValueAPF()) != -1;
    }], vfp_f32imm_xform> {
  let PrintMethod = "printFPImmOperand";
  let ParserMatchClass = FPImmOperand;
  let GISelPredicateCode = [{
      const auto &MO = MI.getOperand(1);
      if (!MO.isFPImm())
        return false;
      return ARM_AM::getFP32Imm(MO.getFPImm()->getValueAPF()) != -1;
    }];
}
```
- EN: Defines TableGen record `vfp_f32imm` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `vfp_f32imm`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 120-123
```tablegen
def vfp_f64imm_xform : SDNodeXForm<fpimm, [{
      uint32_t Enc = ARM_AM::getFP64Imm(N->getValueAPF());
      return CurDAG->getTargetConstant(Enc, SDLoc(N), MVT::i32);
    }]>;
```
- EN: Defines TableGen record `vfp_f64imm_xform` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `vfp_f64imm_xform`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 125-126
```tablegen
def gi_vfp_f64imm : GICustomOperandRenderer<"renderVFPF64Imm">,
                    GISDNodeXFormEquiv<vfp_f64imm_xform>;
```
- EN: Defines TableGen record `gi_vfp_f64imm` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `gi_vfp_f64imm`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 128-140
```tablegen
def vfp_f64imm : Operand<f64>,
                 PatLeaf<(f64 fpimm), [{
      return ARM_AM::getFP64Imm(N->getValueAPF()) != -1;
    }], vfp_f64imm_xform> {
  let PrintMethod = "printFPImmOperand";
  let ParserMatchClass = FPImmOperand;
  let GISelPredicateCode = [{
      const auto &MO = MI.getOperand(1);
      if (!MO.isFPImm())
        return false;
      return ARM_AM::getFP64Imm(MO.getFPImm()->getValueAPF()) != -1;
    }];
}
```
- EN: Defines TableGen record `vfp_f64imm` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `vfp_f64imm`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 142-144
```tablegen
def alignedload16 : PatFrag<(ops node:$ptr), (load node:$ptr), [{
  return cast<LoadSDNode>(N)->getAlign() >= 2;
}]>;
```
- EN: Defines TableGen record `alignedload16` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `alignedload16`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 146-148
```tablegen
def alignedload32 : PatFrag<(ops node:$ptr), (load node:$ptr), [{
  return cast<LoadSDNode>(N)->getAlign() >= 4;
}]>;
```
- EN: Defines TableGen record `alignedload32` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `alignedload32`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 150-153
```tablegen
def alignedstore16 : PatFrag<(ops node:$val, node:$ptr),
                             (store node:$val, node:$ptr), [{
  return cast<StoreSDNode>(N)->getAlign() >= 2;
}]>;
```
- EN: Defines TableGen record `alignedstore16` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `alignedstore16`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 155-158
```tablegen
def alignedstore32 : PatFrag<(ops node:$val, node:$ptr),
                             (store node:$val, node:$ptr), [{
  return cast<StoreSDNode>(N)->getAlign() >= 4;
}]>;
```
- EN: Defines TableGen record `alignedstore32` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `alignedstore32`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 160-169
```tablegen
// The VCVT to/from fixed-point instructions encode the 'fbits' operand
// (the number of fixed bits) differently than it appears in the assembly
// source. It's encoded as "Size - fbits" where Size is the size of the
// fixed-point representation (32 or 16) and fbits is the value appearing
// in the assembly source, an integer in [0,16] or (0,32], depending on size.
def fbits32_asm_operand : AsmOperandClass { let Name = "FBits32"; }
def fbits32 : Operand<i32> {
  let PrintMethod = "printFBits32";
  let ParserMatchClass = fbits32_asm_operand;
}
```
- EN: Defines TableGen record `fbits32_asm_operand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `fbits32_asm_operand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 171-175
```tablegen
def fbits16_asm_operand : AsmOperandClass { let Name = "FBits16"; }
def fbits16 : Operand<i32> {
  let PrintMethod = "printFBits16";
  let ParserMatchClass = fbits16_asm_operand;
}
```
- EN: Defines TableGen record `fbits16_asm_operand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `fbits16_asm_operand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 177-179
```tablegen
//===----------------------------------------------------------------------===//
//  Load / store Instructions.
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 181-181
```tablegen
let canFoldAsLoad = 1, isReMaterializable = 1 in {
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 183-186
```tablegen
def VLDRD : ADI5<0b1101, 0b01, (outs DPR:$Dd), (ins addrmode5:$addr),
                 IIC_fpLoad64, "vldr", "\t$Dd, $addr",
                 [(set DPR:$Dd, (f64 (alignedload32 addrmode5:$addr)))]>,
            Requires<[HasFPRegs]>;
```
- EN: Defines TableGen record `VLDRD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLDRD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 188-195
```tablegen
def VLDRS : ASI5<0b1101, 0b01, (outs SPR:$Sd), (ins addrmode5:$addr),
                 IIC_fpLoad32, "vldr", "\t$Sd, $addr",
                 [(set SPR:$Sd, (alignedload32 addrmode5:$addr))]>,
            Requires<[HasFPRegs]> {
  // Some single precision VFP instructions may be executed on both NEON and VFP
  // pipelines.
  let D = VFPNeonDomain;
}
```
- EN: Defines TableGen record `VLDRS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLDRS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 197-201
```tablegen
let isUnpredicable = 1 in
def VLDRH : AHI5<0b1101, 0b01, (outs HPR:$Sd), (ins addrmode5fp16:$addr),
                 IIC_fpLoad16, "vldr", ".16\t$Sd, $addr",
                 [(set HPR:$Sd, (f16 (alignedload16 addrmode5fp16:$addr)))]>,
            Requires<[HasFPRegs16]>;
```
- EN: Defines TableGen record `VLDRH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLDRH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 203-203
```tablegen
} // End of 'let canFoldAsLoad = 1, isReMaterializable = 1 in'
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 205-216
```tablegen
def : Pat<(bf16 (alignedload16 addrmode5fp16:$addr)),
          (VLDRH addrmode5fp16:$addr)> {
  let Predicates = [HasFPRegs16];
}
def : Pat<(bf16 (alignedload16 addrmode3:$addr)),
          (COPY_TO_REGCLASS (LDRH addrmode3:$addr), HPR)> {
  let Predicates = [HasNoFPRegs16, IsARM];
}
def : Pat<(bf16 (alignedload16 t2addrmode_imm12:$addr)),
          (COPY_TO_REGCLASS (t2LDRHi12 t2addrmode_imm12:$addr), HPR)> {
  let Predicates = [HasNoFPRegs16, IsThumb];
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 218-221
```tablegen
def VSTRD : ADI5<0b1101, 0b00, (outs), (ins DPR:$Dd, addrmode5:$addr),
                 IIC_fpStore64, "vstr", "\t$Dd, $addr",
                 [(alignedstore32 (f64 DPR:$Dd), addrmode5:$addr)]>,
            Requires<[HasFPRegs]>;
```
- EN: Defines TableGen record `VSTRD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VSTRD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 223-230
```tablegen
def VSTRS : ASI5<0b1101, 0b00, (outs), (ins SPR:$Sd, addrmode5:$addr),
                 IIC_fpStore32, "vstr", "\t$Sd, $addr",
                 [(alignedstore32 SPR:$Sd, addrmode5:$addr)]>,
            Requires<[HasFPRegs]> {
  // Some single precision VFP instructions may be executed on both NEON and VFP
  // pipelines.
  let D = VFPNeonDomain;
}
```
- EN: Defines TableGen record `VSTRS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VSTRS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 232-236
```tablegen
let isUnpredicable = 1 in
def VSTRH : AHI5<0b1101, 0b00, (outs), (ins HPR:$Sd, addrmode5fp16:$addr),
                 IIC_fpStore16, "vstr", ".16\t$Sd, $addr",
                 [(alignedstore16 (f16 HPR:$Sd), addrmode5fp16:$addr)]>,
            Requires<[HasFPRegs16]>;
```
- EN: Defines TableGen record `VSTRH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VSTRH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 238-249
```tablegen
def : Pat<(alignedstore16 (bf16 HPR:$Sd), addrmode5fp16:$addr),
          (VSTRH (bf16 HPR:$Sd), addrmode5fp16:$addr)> {
  let Predicates = [HasFPRegs16];
}
def : Pat<(alignedstore16 (bf16 HPR:$Sd), addrmode3:$addr),
          (STRH (COPY_TO_REGCLASS $Sd, GPR), addrmode3:$addr)> {
  let Predicates = [HasNoFPRegs16, IsARM];
}
def : Pat<(alignedstore16 (bf16 HPR:$Sd), t2addrmode_imm12:$addr),
          (t2STRHi12 (COPY_TO_REGCLASS $Sd, GPR), t2addrmode_imm12:$addr)> {
  let Predicates = [HasNoFPRegs16, IsThumb];
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 251-253
```tablegen
//===----------------------------------------------------------------------===//
//  Load / store multiple Instructions.
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 255-272
```tablegen
multiclass vfp_ldst_mult<string asm, bit L_bit,
                         InstrItinClass itin, InstrItinClass itin_upd> {
  let Predicates = [HasFPRegs] in {
  // Double Precision
  def DIA :
    AXDI4<(outs), (ins GPR:$Rn, pred:$p, dpr_reglist:$regs, variable_ops),
          IndexModeNone, itin,
          !strconcat(asm, "ia${p}\t$Rn, $regs"), "", []> {
    let Inst{24-23} = 0b01;       // Increment After
    let Inst{21}    = 0;          // No writeback
    let Inst{20}    = L_bit;
  }
  def DIA_UPD :
    AXDI4<(outs GPR:$wb), (ins GPR:$Rn, pred:$p, dpr_reglist:$regs,
                               variable_ops),
          IndexModeUpd, itin_upd,
          !strconcat(asm, "ia${p}\t$Rn!, $regs"), "$Rn = $wb", []> {
    let Inst{24-23} = 0b01;       // Increment After
```
- EN: Declares TableGen `multiclass vfp_ldst_mult`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass vfp_ldst_mult`，它是一个可复用模板，可展开为多个相关记录。

### Lines 273-284
```tablegen
    let Inst{21}    = 1;          // Writeback
    let Inst{20}    = L_bit;
  }
  def DDB_UPD :
    AXDI4<(outs GPR:$wb), (ins GPR:$Rn, pred:$p, dpr_reglist:$regs,
                               variable_ops),
          IndexModeUpd, itin_upd,
          !strconcat(asm, "db${p}\t$Rn!, $regs"), "$Rn = $wb", []> {
    let Inst{24-23} = 0b10;       // Decrement Before
    let Inst{21}    = 1;          // Writeback
    let Inst{20}    = L_bit;
  }
```
- EN: Defines TableGen record `DDB_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `DDB_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 286-293
```tablegen
  // Single Precision
  def SIA :
    AXSI4<(outs), (ins GPR:$Rn, pred:$p, spr_reglist:$regs, variable_ops),
          IndexModeNone, itin,
          !strconcat(asm, "ia${p}\t$Rn, $regs"), "", []> {
    let Inst{24-23} = 0b01;       // Increment After
    let Inst{21}    = 0;          // No writeback
    let Inst{20}    = L_bit;
```
- EN: Defines TableGen record `SIA` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SIA`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 295-306
```tablegen
    // Some single precision VFP instructions may be executed on both NEON and
    // VFP pipelines.
    let D = VFPNeonDomain;
  }
  def SIA_UPD :
    AXSI4<(outs GPR:$wb), (ins GPR:$Rn, pred:$p, spr_reglist:$regs,
                               variable_ops),
          IndexModeUpd, itin_upd,
          !strconcat(asm, "ia${p}\t$Rn!, $regs"), "$Rn = $wb", []> {
    let Inst{24-23} = 0b01;       // Increment After
    let Inst{21}    = 1;          // Writeback
    let Inst{20}    = L_bit;
```
- EN: Defines TableGen record `SIA_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SIA_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 308-319
```tablegen
    // Some single precision VFP instructions may be executed on both NEON and
    // VFP pipelines.
    let D = VFPNeonDomain;
  }
  def SDB_UPD :
    AXSI4<(outs GPR:$wb), (ins GPR:$Rn, pred:$p, spr_reglist:$regs,
                               variable_ops),
          IndexModeUpd, itin_upd,
          !strconcat(asm, "db${p}\t$Rn!, $regs"), "$Rn = $wb", []> {
    let Inst{24-23} = 0b10;       // Decrement Before
    let Inst{21}    = 1;          // Writeback
    let Inst{20}    = L_bit;
```
- EN: Defines TableGen record `SDB_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SDB_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 321-326
```tablegen
    // Some single precision VFP instructions may be executed on both NEON and
    // VFP pipelines.
    let D = VFPNeonDomain;
  }
  }
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 328-328
```tablegen
let hasSideEffects = 0 in {
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 330-331
```tablegen
let mayLoad = 1, hasExtraDefRegAllocReq = 1 in
defm VLDM : vfp_ldst_mult<"vldm", 1, IIC_fpLoad_m, IIC_fpLoad_mu>;
```
- EN: Defines TableGen record `VLDM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLDM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 333-334
```tablegen
let mayStore = 1, hasExtraSrcRegAllocReq = 1 in
defm VSTM : vfp_ldst_mult<"vstm", 0, IIC_fpStore_m, IIC_fpStore_mu>;
```
- EN: Defines TableGen record `VSTM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VSTM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 336-336
```tablegen
} // hasSideEffects
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 338-339
```tablegen
def : MnemonicAlias<"vldm", "vldmia">;
def : MnemonicAlias<"vstm", "vstmia">;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 342-353
```tablegen
//===----------------------------------------------------------------------===//
//  Lazy load / store multiple Instructions
//
// VLLDM and VLSTM:
// 2 encoding options:
// T1 (bit 7 is 0):
// T1 takes an optional dpr_reglist, must be '{d0-d15}' (exactly)
// T1 require v8-M.Main, secure state, target with 16 D registers (or with no D registers - NOP)
// T2 (bit 7 is 1):
// T2 takes a mandatory dpr_reglist, must be '{d0-d31}' (exactly)
// T2 require v8.1-M.Main, secure state, target with 16/32 D registers (or with no D registers - NOP)
// (source: Arm v8-M ARM, DDI0553B.v ID16122022)
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 355-372
```tablegen
def VLLDM : AXSI4FR<"vlldm${p}\t$Rn, $regs", 0, 1>,
            Requires<[HasV8MMainline, Has8MSecExt]> {
    let Defs = [VPR, FPSCR, FPSCR_NZCV, D0, D1, D2, D3, D4, D5, D6, D7, D8, D9, D10, D11, D12, D13, D14, D15];
    let DecoderMethod = "DecodeLazyLoadStoreMul";
}
// T1: assembly does not contains the register list.
def : InstAlias<"vlldm${p}\t$Rn", (VLLDM GPRnopc:$Rn, pred:$p, 0)>,
                Requires<[HasV8MMainline, Has8MSecExt]>;
// T2: assembly must contains the register list.
// The register list has no effect on the encoding, it is for assembly/disassembly purposes only.
def VLLDM_T2 : AXSI4FR<"vlldm${p}\t$Rn, $regs", 1, 1>,
            Requires<[HasV8_1MMainline, Has8MSecExt]> {
    let Defs = [VPR, FPSCR, FPSCR_NZCV, D0,  D1,  D2,  D3,  D4,  D5,  D6,  D7,  D8,  D9,  D10, D11, D12, D13, D14, D15,
                                        D16, D17, D18, D19, D20, D21, D22, D23, D24, D25, D26, D27, D28, D29, D30, D31];
    let DecoderMethod = "DecodeLazyLoadStoreMul";
}
// T1: assembly contains the register list.
// The register list has no effect on the encoding, it is for assembly/disassembly purposes only.
```
- EN: Defines TableGen record `VLLDM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLLDM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 373-390
```tablegen
def VLSTM : AXSI4FR<"vlstm${p}\t$Rn, $regs", 0, 0>,
            Requires<[HasV8MMainline, Has8MSecExt]> {
    let Defs = [VPR, FPSCR, FPSCR_NZCV];
    let Uses = [VPR, FPSCR, FPSCR_NZCV, D0, D1, D2, D3, D4, D5, D6, D7, D8, D9, D10, D11, D12, D13, D14, D15];
    let DecoderMethod = "DecodeLazyLoadStoreMul";
}
// T1: assembly does not contain the register list.
def : InstAlias<"vlstm${p}\t$Rn", (VLSTM GPRnopc:$Rn, pred:$p, 0)>,
                Requires<[HasV8MMainline, Has8MSecExt]>;
// T2: assembly must contain the register list.
// The register list has no effect on the encoding, it is for assembly/disassembly purposes only.
def VLSTM_T2 : AXSI4FR<"vlstm${p}\t$Rn, $regs", 1, 0>,
            Requires<[HasV8_1MMainline, Has8MSecExt]> {
    let Defs = [VPR, FPSCR, FPSCR_NZCV];
    let Uses = [VPR, FPSCR, FPSCR_NZCV, D0,  D1,  D2,  D3,  D4,  D5,  D6,  D7,  D8,  D9,  D10, D11, D12, D13, D14, D15,
                                        D16, D17, D18, D19, D20, D21, D22, D23, D24, D25, D26, D27, D28, D29, D30, D31];
    let DecoderMethod = "DecodeLazyLoadStoreMul";
}
```
- EN: Defines TableGen record `VLSTM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLSTM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 392-407
```tablegen
def : InstAlias<"vpush${p} $r", (VSTMDDB_UPD SP, pred:$p, dpr_reglist:$r), 0>,
                Requires<[HasFPRegs]>;
def : InstAlias<"vpush${p} $r", (VSTMSDB_UPD SP, pred:$p, spr_reglist:$r), 0>,
                Requires<[HasFPRegs]>;
def : InstAlias<"vpop${p} $r",  (VLDMDIA_UPD SP, pred:$p, dpr_reglist:$r), 0>,
                Requires<[HasFPRegs]>;
def : InstAlias<"vpop${p} $r",  (VLDMSIA_UPD SP, pred:$p, spr_reglist:$r), 0>,
                Requires<[HasFPRegs]>;
defm : VFPDTAnyInstAlias<"vpush${p}", "$r",
                         (VSTMSDB_UPD SP, pred:$p, spr_reglist:$r)>;
defm : VFPDTAnyInstAlias<"vpush${p}", "$r",
                         (VSTMDDB_UPD SP, pred:$p, dpr_reglist:$r)>;
defm : VFPDTAnyInstAlias<"vpop${p}", "$r",
                         (VLDMSIA_UPD SP, pred:$p, spr_reglist:$r)>;
defm : VFPDTAnyInstAlias<"vpop${p}", "$r",
                         (VLDMDIA_UPD SP, pred:$p, dpr_reglist:$r)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 409-426
```tablegen
// FLDMX, FSTMX - Load and store multiple unknown precision registers for
// pre-armv6 cores.
// These instruction are deprecated so we don't want them to get selected.
// However, there is no UAL syntax for them, so we keep them around for
// (dis)assembly only.
multiclass vfp_ldstx_mult<string asm, bit L_bit> {
  let Predicates = [HasFPRegs], hasNoSchedulingInfo = 1 in {
  // Unknown precision
  def XIA :
    AXXI4<(outs), (ins GPR:$Rn, pred:$p, dpr_reglist:$regs, variable_ops),
          IndexModeNone, !strconcat(asm, "iax${p}\t$Rn, $regs"), "", []> {
    let Inst{24-23} = 0b01;       // Increment After
    let Inst{21}    = 0;          // No writeback
    let Inst{20}    = L_bit;
  }
  def XIA_UPD :
    AXXI4<(outs GPR:$wb), (ins GPR:$Rn, pred:$p, dpr_reglist:$regs, variable_ops),
          IndexModeUpd, !strconcat(asm, "iax${p}\t$Rn!, $regs"), "$Rn = $wb", []> {
```
- EN: Declares TableGen `multiclass vfp_ldstx_mult`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass vfp_ldstx_mult`，它是一个可复用模板，可展开为多个相关记录。

### Lines 427-439
```tablegen
    let Inst{24-23} = 0b01;         // Increment After
    let Inst{21}    = 1;            // Writeback
    let Inst{20}    = L_bit;
  }
  def XDB_UPD :
    AXXI4<(outs GPR:$wb), (ins GPR:$Rn, pred:$p, dpr_reglist:$regs, variable_ops),
          IndexModeUpd, !strconcat(asm, "dbx${p}\t$Rn!, $regs"), "$Rn = $wb", []> {
    let Inst{24-23} = 0b10;         // Decrement Before
    let Inst{21}    = 1;            // Writeback
    let Inst{20}    = L_bit;
  }
  }
}
```
- EN: Defines TableGen record `XDB_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `XDB_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 441-442
```tablegen
defm FLDM : vfp_ldstx_mult<"fldm", 1>;
defm FSTM : vfp_ldstx_mult<"fstm", 0>;
```
- EN: Defines TableGen record `FLDM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FLDM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 444-445
```tablegen
def : VFP2MnemonicAlias<"fldmeax", "fldmdbx">;
def : VFP2MnemonicAlias<"fldmfdx", "fldmiax">;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 447-448
```tablegen
def : VFP2MnemonicAlias<"fstmeax", "fstmiax">;
def : VFP2MnemonicAlias<"fstmfdx", "fstmdbx">;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 450-452
```tablegen
//===----------------------------------------------------------------------===//
// FP Binary Operations.
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 454-459
```tablegen
let TwoOperandAliasConstraint = "$Dn = $Dd", mayRaiseFPException = 1, Uses = [FPSCR_RM] in
def VADDD  : ADbI<0b11100, 0b11, 0, 0,
                  (outs DPR:$Dd), (ins DPR:$Dn, DPR:$Dm),
                  IIC_fpALU64, "vadd", ".f64\t$Dd, $Dn, $Dm",
                  [(set DPR:$Dd, (any_fadd DPR:$Dn, (f64 DPR:$Dm)))]>,
             Sched<[WriteFPALU64]>;
```
- EN: Defines TableGen record `VADDD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VADDD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 461-470
```tablegen
let TwoOperandAliasConstraint = "$Sn = $Sd", mayRaiseFPException = 1, Uses = [FPSCR_RM] in
def VADDS  : ASbIn<0b11100, 0b11, 0, 0,
                   (outs SPR:$Sd), (ins SPR:$Sn, SPR:$Sm),
                   IIC_fpALU32, "vadd", ".f32\t$Sd, $Sn, $Sm",
                   [(set SPR:$Sd, (any_fadd SPR:$Sn, SPR:$Sm))]>,
             Sched<[WriteFPALU32]> {
  // Some single precision VFP instructions may be executed on both NEON and
  // VFP pipelines on A8.
  let D = VFPNeonA8Domain;
}
```
- EN: Defines TableGen record `VADDS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VADDS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 472-477
```tablegen
let TwoOperandAliasConstraint = "$Sn = $Sd", mayRaiseFPException = 1, Uses = [FPSCR_RM] in
def VADDH  : AHbI<0b11100, 0b11, 0, 0,
                  (outs HPR:$Sd), (ins HPR:$Sn, HPR:$Sm),
                  IIC_fpALU16, "vadd", ".f16\t$Sd, $Sn, $Sm",
                  [(set (f16 HPR:$Sd), (any_fadd (f16 HPR:$Sn), (f16 HPR:$Sm)))]>,
             Sched<[WriteFPALU32]>;
```
- EN: Defines TableGen record `VADDH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VADDH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 479-484
```tablegen
let TwoOperandAliasConstraint = "$Dn = $Dd", mayRaiseFPException = 1, Uses = [FPSCR_RM] in
def VSUBD  : ADbI<0b11100, 0b11, 1, 0,
                  (outs DPR:$Dd), (ins DPR:$Dn, DPR:$Dm),
                  IIC_fpALU64, "vsub", ".f64\t$Dd, $Dn, $Dm",
                  [(set DPR:$Dd, (any_fsub DPR:$Dn, (f64 DPR:$Dm)))]>,
             Sched<[WriteFPALU64]>;
```
- EN: Defines TableGen record `VSUBD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VSUBD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 486-495
```tablegen
let TwoOperandAliasConstraint = "$Sn = $Sd", mayRaiseFPException = 1, Uses = [FPSCR_RM] in
def VSUBS  : ASbIn<0b11100, 0b11, 1, 0,
                   (outs SPR:$Sd), (ins SPR:$Sn, SPR:$Sm),
                   IIC_fpALU32, "vsub", ".f32\t$Sd, $Sn, $Sm",
                   [(set SPR:$Sd, (any_fsub SPR:$Sn, SPR:$Sm))]>,
             Sched<[WriteFPALU32]>{
  // Some single precision VFP instructions may be executed on both NEON and
  // VFP pipelines on A8.
  let D = VFPNeonA8Domain;
}
```
- EN: Defines TableGen record `VSUBS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VSUBS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 497-502
```tablegen
let TwoOperandAliasConstraint = "$Sn = $Sd", mayRaiseFPException = 1, Uses = [FPSCR_RM] in
def VSUBH  : AHbI<0b11100, 0b11, 1, 0,
                  (outs HPR:$Sd), (ins HPR:$Sn, HPR:$Sm),
                  IIC_fpALU16, "vsub", ".f16\t$Sd, $Sn, $Sm",
                  [(set (f16 HPR:$Sd), (any_fsub (f16 HPR:$Sn), (f16 HPR:$Sm)))]>,
            Sched<[WriteFPALU32]>;
```
- EN: Defines TableGen record `VSUBH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VSUBH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 504-509
```tablegen
let TwoOperandAliasConstraint = "$Dn = $Dd", mayRaiseFPException = 1, Uses = [FPSCR_RM] in
def VDIVD  : ADbI<0b11101, 0b00, 0, 0,
                  (outs DPR:$Dd), (ins DPR:$Dn, DPR:$Dm),
                  IIC_fpDIV64, "vdiv", ".f64\t$Dd, $Dn, $Dm",
                  [(set DPR:$Dd, (any_fdiv DPR:$Dn, (f64 DPR:$Dm)))]>,
             Sched<[WriteFPDIV64]>;
```
- EN: Defines TableGen record `VDIVD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VDIVD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 511-516
```tablegen
let TwoOperandAliasConstraint = "$Sn = $Sd", mayRaiseFPException = 1, Uses = [FPSCR_RM] in
def VDIVS  : ASbI<0b11101, 0b00, 0, 0,
                  (outs SPR:$Sd), (ins SPR:$Sn, SPR:$Sm),
                  IIC_fpDIV32, "vdiv", ".f32\t$Sd, $Sn, $Sm",
                  [(set SPR:$Sd, (any_fdiv SPR:$Sn, SPR:$Sm))]>,
             Sched<[WriteFPDIV32]>;
```
- EN: Defines TableGen record `VDIVS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VDIVS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 518-523
```tablegen
let TwoOperandAliasConstraint = "$Sn = $Sd", mayRaiseFPException = 1, Uses = [FPSCR_RM]  in
def VDIVH  : AHbI<0b11101, 0b00, 0, 0,
                  (outs HPR:$Sd), (ins HPR:$Sn, HPR:$Sm),
                  IIC_fpDIV16, "vdiv", ".f16\t$Sd, $Sn, $Sm",
                  [(set (f16 HPR:$Sd), (any_fdiv (f16 HPR:$Sn), (f16 HPR:$Sm)))]>,
             Sched<[WriteFPDIV32]>;
```
- EN: Defines TableGen record `VDIVH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VDIVH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 525-530
```tablegen
let TwoOperandAliasConstraint = "$Dn = $Dd", mayRaiseFPException = 1, Uses = [FPSCR_RM] in
def VMULD  : ADbI<0b11100, 0b10, 0, 0,
                  (outs DPR:$Dd), (ins DPR:$Dn, DPR:$Dm),
                  IIC_fpMUL64, "vmul", ".f64\t$Dd, $Dn, $Dm",
                  [(set DPR:$Dd, (any_fmul DPR:$Dn, (f64 DPR:$Dm)))]>,
             Sched<[WriteFPMUL64, ReadFPMUL, ReadFPMUL]>;
```
- EN: Defines TableGen record `VMULD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VMULD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 532-541
```tablegen
let TwoOperandAliasConstraint = "$Sn = $Sd", mayRaiseFPException = 1, Uses = [FPSCR_RM] in
def VMULS  : ASbIn<0b11100, 0b10, 0, 0,
                   (outs SPR:$Sd), (ins SPR:$Sn, SPR:$Sm),
                   IIC_fpMUL32, "vmul", ".f32\t$Sd, $Sn, $Sm",
                   [(set SPR:$Sd, (any_fmul SPR:$Sn, SPR:$Sm))]>,
            Sched<[WriteFPMUL32, ReadFPMUL, ReadFPMUL]> {
  // Some single precision VFP instructions may be executed on both NEON and
  // VFP pipelines on A8.
  let D = VFPNeonA8Domain;
}
```
- EN: Defines TableGen record `VMULS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VMULS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 543-548
```tablegen
let TwoOperandAliasConstraint = "$Sn = $Sd", mayRaiseFPException = 1, Uses = [FPSCR_RM] in
def VMULH  : AHbI<0b11100, 0b10, 0, 0,
                  (outs HPR:$Sd), (ins HPR:$Sn, HPR:$Sm),
                  IIC_fpMUL16, "vmul", ".f16\t$Sd, $Sn, $Sm",
                  [(set (f16 HPR:$Sd), (any_fmul (f16 HPR:$Sn), (f16 HPR:$Sm)))]>,
             Sched<[WriteFPMUL32, ReadFPMUL, ReadFPMUL]>;
```
- EN: Defines TableGen record `VMULH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VMULH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 550-555
```tablegen
let TwoOperandAliasConstraint = "$Dn = $Dd", mayRaiseFPException = 1, Uses = [FPSCR_RM] in
def VNMULD : ADbI<0b11100, 0b10, 1, 0,
                  (outs DPR:$Dd), (ins DPR:$Dn, DPR:$Dm),
                  IIC_fpMUL64, "vnmul", ".f64\t$Dd, $Dn, $Dm",
                  [(set DPR:$Dd, (fneg (any_fmul DPR:$Dn, (f64 DPR:$Dm))))]>,
             Sched<[WriteFPMUL64, ReadFPMUL, ReadFPMUL]>;
```
- EN: Defines TableGen record `VNMULD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VNMULD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 557-566
```tablegen
let TwoOperandAliasConstraint = "$Sn = $Sd", mayRaiseFPException = 1, Uses = [FPSCR_RM] in
def VNMULS : ASbI<0b11100, 0b10, 1, 0,
                  (outs SPR:$Sd), (ins SPR:$Sn, SPR:$Sm),
                  IIC_fpMUL32, "vnmul", ".f32\t$Sd, $Sn, $Sm",
                  [(set SPR:$Sd, (fneg (any_fmul SPR:$Sn, SPR:$Sm)))]>,
            Sched<[WriteFPMUL32, ReadFPMUL, ReadFPMUL]> {
  // Some single precision VFP instructions may be executed on both NEON and
  // VFP pipelines on A8.
  let D = VFPNeonA8Domain;
}
```
- EN: Defines TableGen record `VNMULS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VNMULS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 568-573
```tablegen
let TwoOperandAliasConstraint = "$Sn = $Sd", mayRaiseFPException = 1, Uses = [FPSCR_RM] in
def VNMULH : AHbI<0b11100, 0b10, 1, 0,
                  (outs HPR:$Sd), (ins HPR:$Sn, HPR:$Sm),
                  IIC_fpMUL16, "vnmul", ".f16\t$Sd, $Sn, $Sm",
                  [(set (f16 HPR:$Sd), (fneg (any_fmul (f16 HPR:$Sn), (f16 HPR:$Sm))))]>,
             Sched<[WriteFPMUL32, ReadFPMUL, ReadFPMUL]>;
```
- EN: Defines TableGen record `VNMULH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VNMULH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 575-583
```tablegen
multiclass vsel_inst<string op, bits<2> opc, int CC> {
  let DecoderNamespace = "VFPV8", PostEncoderMethod = "",
      Uses = [CPSR], AddedComplexity = 4, isUnpredicable = 1 in {
    def H : AHbInp<0b11100, opc, 0,
                   (outs HPR:$Sd), (ins HPR:$Sn, HPR:$Sm),
                   NoItinerary, !strconcat("vsel", op, ".f16\t$Sd, $Sn, $Sm"),
                   [(set (f16 HPR:$Sd),
                         (ARMcmov (f16 HPR:$Sm), (f16 HPR:$Sn), CC, CPSR))]>,
                   Requires<[HasFullFP16]>;
```
- EN: Declares TableGen `multiclass vsel_inst`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass vsel_inst`，它是一个可复用模板，可展开为多个相关记录。

### Lines 585-589
```tablegen
    def S : ASbInp<0b11100, opc, 0,
                   (outs SPR:$Sd), (ins SPR:$Sn, SPR:$Sm),
                   NoItinerary, !strconcat("vsel", op, ".f32\t$Sd, $Sn, $Sm"),
                   [(set SPR:$Sd, (ARMcmov SPR:$Sm, SPR:$Sn, CC, CPSR))]>,
                   Requires<[HasFPARMv8]>;
```
- EN: Defines TableGen record `S` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `S`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 591-598
```tablegen
    def D : ADbInp<0b11100, opc, 0,
                   (outs DPR:$Dd), (ins DPR:$Dn, DPR:$Dm),
                   NoItinerary, !strconcat("vsel", op, ".f64\t$Dd, $Dn, $Dm"),
                   [(set DPR:$Dd,
                         (ARMcmov (f64 DPR:$Dm), (f64 DPR:$Dn), CC, CPSR))]>,
                   Requires<[HasFPARMv8, HasDPVFP]>;
  }
}
```
- EN: Defines TableGen record `D` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `D`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 600-604
```tablegen
// The CC constants here match ARMCC::CondCodes.
defm VSELGT : vsel_inst<"gt", 0b11, 12>;
defm VSELGE : vsel_inst<"ge", 0b10, 10>;
defm VSELEQ : vsel_inst<"eq", 0b00, 0>;
defm VSELVS : vsel_inst<"vs", 0b01, 6>;
```
- EN: Defines TableGen record `VSELGT` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VSELGT`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 606-613
```tablegen
multiclass vmaxmin_inst<string op, bit opc, PatFrags SD> {
  let DecoderNamespace = "VFPV8", PostEncoderMethod = "",
      isUnpredicable = 1, mayRaiseFPException = 1 in {
    def H : AHbInp<0b11101, 0b00, opc,
                   (outs HPR:$Sd), (ins HPR:$Sn, HPR:$Sm),
                   NoItinerary, !strconcat(op, ".f16\t$Sd, $Sn, $Sm"),
                   [(set (f16 HPR:$Sd), (SD (f16 HPR:$Sn), (f16 HPR:$Sm)))]>,
                   Requires<[HasFullFP16]>;
```
- EN: Declares TableGen `multiclass vmaxmin_inst`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass vmaxmin_inst`，它是一个可复用模板，可展开为多个相关记录。

### Lines 615-619
```tablegen
    def S : ASbInp<0b11101, 0b00, opc,
                   (outs SPR:$Sd), (ins SPR:$Sn, SPR:$Sm),
                   NoItinerary, !strconcat(op, ".f32\t$Sd, $Sn, $Sm"),
                   [(set SPR:$Sd, (SD SPR:$Sn, SPR:$Sm))]>,
                   Requires<[HasFPARMv8]>;
```
- EN: Defines TableGen record `S` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `S`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 621-627
```tablegen
    def D : ADbInp<0b11101, 0b00, opc,
                   (outs DPR:$Dd), (ins DPR:$Dn, DPR:$Dm),
                   NoItinerary, !strconcat(op, ".f64\t$Dd, $Dn, $Dm"),
                   [(set DPR:$Dd, (f64 (SD (f64 DPR:$Dn), (f64 DPR:$Dm))))]>,
                   Requires<[HasFPARMv8, HasDPVFP]>;
  }
}
```
- EN: Defines TableGen record `D` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `D`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 629-630
```tablegen
defm VFP_VMAXNM : vmaxmin_inst<"vmaxnm", 0, any_fmaxnum>;
defm VFP_VMINNM : vmaxmin_inst<"vminnm", 1, any_fminnum>;
```
- EN: Defines TableGen record `VFP_VMAXNM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VFP_VMAXNM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 632-637
```tablegen
// Match reassociated forms only if not sign dependent rounding.
def : Pat<(fmul (fneg DPR:$a), (f64 DPR:$b)),
          (VNMULD DPR:$a, DPR:$b)>,
          Requires<[NoHonorSignDependentRounding,HasDPVFP]>;
def : Pat<(fmul (fneg SPR:$a), SPR:$b),
          (VNMULS SPR:$a, SPR:$b)>, Requires<[NoHonorSignDependentRounding]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 639-644
```tablegen
// These are encoded as unary instructions.
let Defs = [FPSCR_NZCV], mayRaiseFPException = 1, Uses = [FPSCR_RM] in {
def VCMPED : ADuI<0b11101, 0b11, 0b0100, 0b11, 0,
                  (outs), (ins DPR:$Dd, DPR:$Dm),
                  IIC_fpCMP64, "vcmpe", ".f64\t$Dd, $Dm", "",
                  [(set FPSCR_NZCV, (arm_cmpfpe DPR:$Dd, (f64 DPR:$Dm)))]>;
```
- EN: Defines TableGen record `VCMPED` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VCMPED`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 646-653
```tablegen
def VCMPES : ASuI<0b11101, 0b11, 0b0100, 0b11, 0,
                  (outs), (ins SPR:$Sd, SPR:$Sm),
                  IIC_fpCMP32, "vcmpe", ".f32\t$Sd, $Sm", "",
                  [(set FPSCR_NZCV, (arm_cmpfpe SPR:$Sd, SPR:$Sm))]> {
  // Some single precision VFP instructions may be executed on both NEON and
  // VFP pipelines on A8.
  let D = VFPNeonA8Domain;
}
```
- EN: Defines TableGen record `VCMPES` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VCMPES`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 655-658
```tablegen
def VCMPEH : AHuI<0b11101, 0b11, 0b0100, 0b11, 0,
                  (outs), (ins HPR:$Sd, HPR:$Sm),
                  IIC_fpCMP16, "vcmpe", ".f16\t$Sd, $Sm",
                  [(set FPSCR_NZCV, (arm_cmpfpe (f16 HPR:$Sd), (f16 HPR:$Sm)))]>;
```
- EN: Defines TableGen record `VCMPEH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VCMPEH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 660-663
```tablegen
def VCMPD  : ADuI<0b11101, 0b11, 0b0100, 0b01, 0,
                  (outs), (ins DPR:$Dd, DPR:$Dm),
                  IIC_fpCMP64, "vcmp", ".f64\t$Dd, $Dm", "",
                  [(set FPSCR_NZCV, (arm_cmpfp DPR:$Dd, (f64 DPR:$Dm)))]>;
```
- EN: Defines TableGen record `VCMPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VCMPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 665-672
```tablegen
def VCMPS  : ASuI<0b11101, 0b11, 0b0100, 0b01, 0,
                  (outs), (ins SPR:$Sd, SPR:$Sm),
                  IIC_fpCMP32, "vcmp", ".f32\t$Sd, $Sm", "",
                  [(set FPSCR_NZCV, (arm_cmpfp SPR:$Sd, SPR:$Sm))]> {
  // Some single precision VFP instructions may be executed on both NEON and
  // VFP pipelines on A8.
  let D = VFPNeonA8Domain;
}
```
- EN: Defines TableGen record `VCMPS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VCMPS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 674-678
```tablegen
def VCMPH  : AHuI<0b11101, 0b11, 0b0100, 0b01, 0,
                  (outs), (ins HPR:$Sd, HPR:$Sm),
                  IIC_fpCMP16, "vcmp", ".f16\t$Sd, $Sm",
                  [(set FPSCR_NZCV, (arm_cmpfp (f16 HPR:$Sd), (f16 HPR:$Sm)))]>;
} // Defs = [FPSCR_NZCV]
```
- EN: Defines TableGen record `VCMPH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VCMPH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 680-682
```tablegen
//===----------------------------------------------------------------------===//
// FP Unary Operations.
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 684-687
```tablegen
def VABSD  : ADuI<0b11101, 0b11, 0b0000, 0b11, 0,
                  (outs DPR:$Dd), (ins DPR:$Dm),
                  IIC_fpUNA64, "vabs", ".f64\t$Dd, $Dm", "",
                  [(set DPR:$Dd, (fabs (f64 DPR:$Dm)))]>;
```
- EN: Defines TableGen record `VABSD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VABSD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 689-696
```tablegen
def VABSS  : ASuIn<0b11101, 0b11, 0b0000, 0b11, 0,
                   (outs SPR:$Sd), (ins SPR:$Sm),
                   IIC_fpUNA32, "vabs", ".f32\t$Sd, $Sm",
                   [(set SPR:$Sd, (fabs SPR:$Sm))]> {
  // Some single precision VFP instructions may be executed on both NEON and
  // VFP pipelines on A8.
  let D = VFPNeonA8Domain;
}
```
- EN: Defines TableGen record `VABSS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VABSS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 698-701
```tablegen
def VABSH  : AHuI<0b11101, 0b11, 0b0000, 0b11, 0,
                   (outs HPR:$Sd), (ins HPR:$Sm),
                   IIC_fpUNA16, "vabs", ".f16\t$Sd, $Sm",
                   [(set (f16 HPR:$Sd), (fabs (f16 HPR:$Sm)))]>;
```
- EN: Defines TableGen record `VABSH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VABSH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 703-710
```tablegen
let Defs = [FPSCR_NZCV], mayRaiseFPException = 1, Uses = [FPSCR_RM] in {
def VCMPEZD : ADuI<0b11101, 0b11, 0b0101, 0b11, 0,
                   (outs), (ins DPR:$Dd),
                   IIC_fpCMP64, "vcmpe", ".f64\t$Dd, #0", "",
                   [(set FPSCR_NZCV, (arm_cmpfpe0 (f64 DPR:$Dd)))]> {
  let Inst{3-0} = 0b0000;
  let Inst{5}   = 0;
}
```
- EN: Defines TableGen record `VCMPEZD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VCMPEZD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 712-717
```tablegen
def VCMPEZS : ASuI<0b11101, 0b11, 0b0101, 0b11, 0,
                   (outs), (ins SPR:$Sd),
                   IIC_fpCMP32, "vcmpe", ".f32\t$Sd, #0", "",
                   [(set FPSCR_NZCV, (arm_cmpfpe0 SPR:$Sd))]> {
  let Inst{3-0} = 0b0000;
  let Inst{5}   = 0;
```
- EN: Defines TableGen record `VCMPEZS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VCMPEZS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 719-722
```tablegen
  // Some single precision VFP instructions may be executed on both NEON and
  // VFP pipelines on A8.
  let D = VFPNeonA8Domain;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 724-730
```tablegen
def VCMPEZH : AHuI<0b11101, 0b11, 0b0101, 0b11, 0,
                   (outs), (ins HPR:$Sd),
                   IIC_fpCMP16, "vcmpe", ".f16\t$Sd, #0",
                   [(set FPSCR_NZCV, (arm_cmpfpe0 (f16 HPR:$Sd)))]> {
  let Inst{3-0} = 0b0000;
  let Inst{5}   = 0;
}
```
- EN: Defines TableGen record `VCMPEZH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VCMPEZH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 732-738
```tablegen
def VCMPZD  : ADuI<0b11101, 0b11, 0b0101, 0b01, 0,
                   (outs), (ins DPR:$Dd),
                   IIC_fpCMP64, "vcmp", ".f64\t$Dd, #0", "",
                   [(set FPSCR_NZCV, (arm_cmpfp0 (f64 DPR:$Dd)))]> {
  let Inst{3-0} = 0b0000;
  let Inst{5}   = 0;
}
```
- EN: Defines TableGen record `VCMPZD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VCMPZD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 740-745
```tablegen
def VCMPZS  : ASuI<0b11101, 0b11, 0b0101, 0b01, 0,
                   (outs), (ins SPR:$Sd),
                   IIC_fpCMP32, "vcmp", ".f32\t$Sd, #0", "",
                   [(set FPSCR_NZCV, (arm_cmpfp0 SPR:$Sd))]> {
  let Inst{3-0} = 0b0000;
  let Inst{5}   = 0;
```
- EN: Defines TableGen record `VCMPZS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VCMPZS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 747-750
```tablegen
  // Some single precision VFP instructions may be executed on both NEON and
  // VFP pipelines on A8.
  let D = VFPNeonA8Domain;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 752-759
```tablegen
def VCMPZH  : AHuI<0b11101, 0b11, 0b0101, 0b01, 0,
                   (outs), (ins HPR:$Sd),
                   IIC_fpCMP16, "vcmp", ".f16\t$Sd, #0",
                   [(set FPSCR_NZCV, (arm_cmpfp0 (f16 HPR:$Sd)))]> {
  let Inst{3-0} = 0b0000;
  let Inst{5}   = 0;
}
} // Defs = [FPSCR_NZCV]
```
- EN: Defines TableGen record `VCMPZH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VCMPZH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 761-769
```tablegen
let mayRaiseFPException = 1, Uses = [FPSCR_RM] in
def VCVTDS  : ASuI<0b11101, 0b11, 0b0111, 0b11, 0,
                   (outs DPR:$Dd), (ins SPR:$Sm),
                   IIC_fpCVTDS, "vcvt", ".f64.f32\t$Dd, $Sm", "",
                   [(set DPR:$Dd, (any_fpextend SPR:$Sm))]>,
             Sched<[WriteFPCVT]> {
  // Instruction operands.
  bits<5> Dd;
  bits<5> Sm;
```
- EN: Defines TableGen record `VCVTDS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VCVTDS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 771-775
```tablegen
  // Encode instruction operands.
  let Inst{3-0}   = Sm{4-1};
  let Inst{5}     = Sm{0};
  let Inst{15-12} = Dd{3-0};
  let Inst{22}    = Dd{4};
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 777-779
```tablegen
  let Predicates = [HasVFP2, HasDPVFP];
  let hasSideEffects = 0;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 781-789
```tablegen
// Special case encoding: bits 11-8 is 0b1011.
let mayRaiseFPException = 1, Uses = [FPSCR_RM] in
def VCVTSD  : VFPAI<(outs SPR:$Sd), (ins DPR:$Dm), VFPUnaryFrm,
                    IIC_fpCVTSD, "vcvt", ".f32.f64\t$Sd, $Dm", "",
                    [(set SPR:$Sd, (any_fpround DPR:$Dm))]>,
              Sched<[WriteFPCVT]> {
  // Instruction operands.
  bits<5> Sd;
  bits<5> Dm;
```
- EN: Defines TableGen record `VCVTSD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VCVTSD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 791-795
```tablegen
  // Encode instruction operands.
  let Inst{3-0}   = Dm{3-0};
  let Inst{5}     = Dm{4};
  let Inst{15-12} = Sd{4-1};
  let Inst{22}    = Sd{0};
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 797-801
```tablegen
  let Inst{27-23} = 0b11101;
  let Inst{21-16} = 0b110111;
  let Inst{11-8}  = 0b1011;
  let Inst{7-6}   = 0b11;
  let Inst{4}     = 0;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 803-805
```tablegen
  let Predicates = [HasVFP2, HasDPVFP];
  let hasSideEffects = 0;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 807-813
```tablegen
// Between half, single and double-precision.
let hasSideEffects = 0, mayRaiseFPException = 1, Uses = [FPSCR_RM] in
def VCVTBHS: ASuI<0b11101, 0b11, 0b0010, 0b01, 0, (outs SPR:$Sd), (ins SPR:$Sm),
                 /* FIXME */ IIC_fpCVTSH, "vcvtb", ".f32.f16\t$Sd, $Sm", "",
                 [/* Intentionally left blank, see patterns below */]>,
                 Requires<[HasFP16]>,
             Sched<[WriteFPCVT]>;
```
- EN: Defines TableGen record `VCVTBHS:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VCVTBHS:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 815-818
```tablegen
def : FP16Pat<(f32 (any_fpextend (f16 HPR:$Sm))),
              (VCVTBHS (COPY_TO_REGCLASS (f16 HPR:$Sm), SPR))>;
def : FP16Pat<(any_f16_to_fp GPR:$a),
              (VCVTBHS (COPY_TO_REGCLASS GPR:$a, SPR))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 820-825
```tablegen
let hasSideEffects = 0, mayRaiseFPException = 1, Uses = [FPSCR_RM] in
def VCVTBSH: ASuI<0b11101, 0b11, 0b0011, 0b01, 0, (outs SPR:$Sd), (ins SPR:$Sda, SPR:$Sm),
                 /* FIXME */ IIC_fpCVTHS, "vcvtb", ".f16.f32\t$Sd, $Sm", "$Sd = $Sda",
                 [/* Intentionally left blank, see patterns below */]>,
                 Requires<[HasFP16]>,
             Sched<[WriteFPCVT]>;
```
- EN: Defines TableGen record `VCVTBSH:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VCVTBSH:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 827-840
```tablegen
def : FP16Pat<(f16 (any_fpround SPR:$Sm)),
              (COPY_TO_REGCLASS (VCVTBSH (IMPLICIT_DEF), SPR:$Sm), HPR)>;
def : FP16Pat<(any_fp_to_f16 SPR:$a),
              (i32 (COPY_TO_REGCLASS (VCVTBSH (IMPLICIT_DEF), SPR:$a), GPR))>;
def : FP16Pat<(insertelt (v8f16 MQPR:$src1), (f16 (any_fpround (f32 SPR:$src2))), imm_even:$lane),
              (v8f16 (INSERT_SUBREG (v8f16 MQPR:$src1),
                                    (VCVTBSH (EXTRACT_SUBREG (v8f16 MQPR:$src1), (SSubReg_f16_reg imm:$lane)),
                                             SPR:$src2),
                                    (SSubReg_f16_reg imm:$lane)))>;
def : FP16Pat<(insertelt (v4f16 DPR:$src1), (f16 (any_fpround (f32 SPR:$src2))), imm_even:$lane),
              (v4f16 (INSERT_SUBREG (v4f16 DPR:$src1),
                                    (VCVTBSH (EXTRACT_SUBREG (v4f16 DPR:$src1), (SSubReg_f16_reg imm:$lane)),
                                             SPR:$src2),
                                    (SSubReg_f16_reg imm:$lane)))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 842-847
```tablegen
let hasSideEffects = 0, mayRaiseFPException = 1, Uses = [FPSCR_RM] in
def VCVTTHS: ASuI<0b11101, 0b11, 0b0010, 0b11, 0, (outs SPR:$Sd), (ins SPR:$Sm),
                 /* FIXME */ IIC_fpCVTSH, "vcvtt", ".f32.f16\t$Sd, $Sm", "",
                 [/* Intentionally left blank, see patterns below */]>,
                 Requires<[HasFP16]>,
             Sched<[WriteFPCVT]>;
```
- EN: Defines TableGen record `VCVTTHS:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VCVTTHS:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 849-854
```tablegen
def : FP16Pat<(f32 (any_fpextend (extractelt (v8f16 MQPR:$src), imm_odd:$lane))),
              (VCVTTHS (EXTRACT_SUBREG MQPR:$src, (SSubReg_f16_reg imm_odd:$lane)))>;
def : FP16Pat<(f32 (any_fpextend (extractelt (v4f16 DPR:$src), imm_odd:$lane))),
              (VCVTTHS (EXTRACT_SUBREG
                (v2f32 (COPY_TO_REGCLASS (v4f16 DPR:$src), DPR_VFP2)),
                (SSubReg_f16_reg imm_odd:$lane)))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 856-861
```tablegen
let hasSideEffects = 0, mayRaiseFPException = 1, Uses = [FPSCR_RM] in
def VCVTTSH: ASuI<0b11101, 0b11, 0b0011, 0b11, 0, (outs SPR:$Sd), (ins SPR:$Sda, SPR:$Sm),
                 /* FIXME */ IIC_fpCVTHS, "vcvtt", ".f16.f32\t$Sd, $Sm", "$Sd = $Sda",
                 [/* Intentionally left blank, see patterns below */]>,
                 Requires<[HasFP16]>,
            Sched<[WriteFPCVT]>;
```
- EN: Defines TableGen record `VCVTTSH:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VCVTTSH:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 863-872
```tablegen
def : FP16Pat<(insertelt (v8f16 MQPR:$src1), (f16 (any_fpround (f32 SPR:$src2))), imm_odd:$lane),
              (v8f16 (INSERT_SUBREG (v8f16 MQPR:$src1),
                                    (VCVTTSH (EXTRACT_SUBREG (v8f16 MQPR:$src1), (SSubReg_f16_reg imm:$lane)),
                                             SPR:$src2),
                                    (SSubReg_f16_reg imm:$lane)))>;
def : FP16Pat<(insertelt (v4f16 DPR:$src1), (f16 (any_fpround (f32 SPR:$src2))), imm_odd:$lane),
              (v4f16 (INSERT_SUBREG (v4f16 DPR:$src1),
                                    (VCVTTSH (EXTRACT_SUBREG (v4f16 DPR:$src1), (SSubReg_f16_reg imm:$lane)),
                                             SPR:$src2),
                                    (SSubReg_f16_reg imm:$lane)))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 874-882
```tablegen
let mayRaiseFPException = 1, Uses = [FPSCR_RM] in 
def VCVTBHD : ADuI<0b11101, 0b11, 0b0010, 0b01, 0,
                   (outs DPR:$Dd), (ins SPR:$Sm),
                   NoItinerary, "vcvtb", ".f64.f16\t$Dd, $Sm", "",
                   [/* Intentionally left blank, see patterns below */]>,
                   Requires<[HasFPARMv8, HasDPVFP]>,
              Sched<[WriteFPCVT]> {
  // Instruction operands.
  bits<5> Sm;
```
- EN: Defines TableGen record `VCVTBHD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VCVTBHD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 884-886
```tablegen
  // Encode instruction operands.
  let Inst{3-0} = Sm{4-1};
  let Inst{5}   = Sm{0};
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 888-889
```tablegen
  let hasSideEffects = 0;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 891-896
```tablegen
def : FullFP16Pat<(f64 (any_fpextend (f16 HPR:$Sm))),
                  (VCVTBHD (COPY_TO_REGCLASS (f16 HPR:$Sm), SPR))>,
                  Requires<[HasFPARMv8, HasDPVFP]>;
def : FP16Pat<(f64 (any_f16_to_fp GPR:$a)),
              (VCVTBHD (COPY_TO_REGCLASS GPR:$a, SPR))>,
              Requires<[HasFPARMv8, HasDPVFP]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 898-906
```tablegen
let mayRaiseFPException = 1, Uses = [FPSCR_RM] in
def VCVTBDH : ADuI<0b11101, 0b11, 0b0011, 0b01, 0,
                   (outs SPR:$Sd), (ins SPR:$Sda, DPR:$Dm),
                   NoItinerary, "vcvtb", ".f16.f64\t$Sd, $Dm", "$Sd = $Sda",
                   [/* Intentionally left blank, see patterns below */]>,
                   Requires<[HasFPARMv8, HasDPVFP]> {
  // Instruction operands.
  bits<5> Sd;
  bits<5> Dm;
```
- EN: Defines TableGen record `VCVTBDH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VCVTBDH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 908-912
```tablegen
  // Encode instruction operands.
  let Inst{3-0}     = Dm{3-0};
  let Inst{5}       = Dm{4};
  let Inst{15-12}   = Sd{4-1};
  let Inst{22}      = Sd{0};
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 914-915
```tablegen
  let hasSideEffects = 0;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 917-922
```tablegen
def : FullFP16Pat<(f16 (any_fpround DPR:$Dm)),
                  (COPY_TO_REGCLASS (VCVTBDH (IMPLICIT_DEF), DPR:$Dm), HPR)>,
                  Requires<[HasFPARMv8, HasDPVFP]>;
def : FP16Pat<(any_fp_to_f16 (f64 DPR:$a)),
              (i32 (COPY_TO_REGCLASS (VCVTBDH (IMPLICIT_DEF), DPR:$a), GPR))>,
                   Requires<[HasFPARMv8, HasDPVFP]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 924-930
```tablegen
let mayRaiseFPException = 1, Uses = [FPSCR_RM] in
def VCVTTHD : ADuI<0b11101, 0b11, 0b0010, 0b11, 0,
                   (outs DPR:$Dd), (ins SPR:$Sm),
                   NoItinerary, "vcvtt", ".f64.f16\t$Dd, $Sm", "",
                   []>, Requires<[HasFPARMv8, HasDPVFP]> {
  // Instruction operands.
  bits<5> Sm;
```
- EN: Defines TableGen record `VCVTTHD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VCVTTHD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 932-934
```tablegen
  // Encode instruction operands.
  let Inst{3-0} = Sm{4-1};
  let Inst{5}   = Sm{0};
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 936-937
```tablegen
  let hasSideEffects = 0;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 939-946
```tablegen
let mayRaiseFPException = 1, Uses = [FPSCR_RM] in
def VCVTTDH : ADuI<0b11101, 0b11, 0b0011, 0b11, 0,
                   (outs SPR:$Sd), (ins SPR:$Sda, DPR:$Dm),
                   NoItinerary, "vcvtt", ".f16.f64\t$Sd, $Dm", "$Sd = $Sda",
                   []>, Requires<[HasFPARMv8, HasDPVFP]> {
  // Instruction operands.
  bits<5> Sd;
  bits<5> Dm;
```
- EN: Defines TableGen record `VCVTTDH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VCVTTDH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 948-952
```tablegen
  // Encode instruction operands.
  let Inst{15-12} = Sd{4-1};
  let Inst{22}    = Sd{0};
  let Inst{3-0}   = Dm{3-0};
  let Inst{5}     = Dm{4};
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 954-955
```tablegen
  let hasSideEffects = 0;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 957-967
```tablegen
multiclass vcvt_inst<string opc, bits<2> rm,
                     SDPatternOperator node = null_frag> {
  let PostEncoderMethod = "", DecoderNamespace = "VFPV8", hasSideEffects = 0, 
      mayRaiseFPException = 1 in {
    def SH : AHuInp<0b11101, 0b11, 0b1100, 0b11, 0,
                    (outs SPR:$Sd), (ins HPR:$Sm),
                    NoItinerary, !strconcat("vcvt", opc, ".s32.f16\t$Sd, $Sm"),
                    []>,
                    Requires<[HasFullFP16]> {
      let Inst{17-16} = rm;
    }
```
- EN: Declares TableGen `multiclass vcvt_inst`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass vcvt_inst`，它是一个可复用模板，可展开为多个相关记录。

### Lines 969-975
```tablegen
    def UH : AHuInp<0b11101, 0b11, 0b1100, 0b01, 0,
                    (outs SPR:$Sd), (ins HPR:$Sm),
                    NoItinerary, !strconcat("vcvt", opc, ".u32.f16\t$Sd, $Sm"),
                    []>,
                    Requires<[HasFullFP16]> {
      let Inst{17-16} = rm;
    }
```
- EN: Defines TableGen record `UH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `UH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 977-983
```tablegen
    def SS : ASuInp<0b11101, 0b11, 0b1100, 0b11, 0,
                    (outs SPR:$Sd), (ins SPR:$Sm),
                    NoItinerary, !strconcat("vcvt", opc, ".s32.f32\t$Sd, $Sm"),
                    []>,
                    Requires<[HasFPARMv8]> {
      let Inst{17-16} = rm;
    }
```
- EN: Defines TableGen record `SS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 985-991
```tablegen
    def US : ASuInp<0b11101, 0b11, 0b1100, 0b01, 0,
                    (outs SPR:$Sd), (ins SPR:$Sm),
                    NoItinerary, !strconcat("vcvt", opc, ".u32.f32\t$Sd, $Sm"),
                    []>,
                    Requires<[HasFPARMv8]> {
      let Inst{17-16} = rm;
    }
```
- EN: Defines TableGen record `US` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `US`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 993-998
```tablegen
    def SD : ASuInp<0b11101, 0b11, 0b1100, 0b11, 0,
                    (outs SPR:$Sd), (ins DPR:$Dm),
                    NoItinerary, !strconcat("vcvt", opc, ".s32.f64\t$Sd, $Dm"),
                    []>,
                    Requires<[HasFPARMv8, HasDPVFP]> {
      bits<5> Dm;
```
- EN: Defines TableGen record `SD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1000-1000
```tablegen
      let Inst{17-16} = rm;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1002-1006
```tablegen
      // Encode instruction operands.
      let Inst{3-0} = Dm{3-0};
      let Inst{5}   = Dm{4};
      let Inst{8} = 1;
    }
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1008-1013
```tablegen
    def UD : ASuInp<0b11101, 0b11, 0b1100, 0b01, 0,
                    (outs SPR:$Sd), (ins DPR:$Dm),
                    NoItinerary, !strconcat("vcvt", opc, ".u32.f64\t$Sd, $Dm"),
                    []>,
                    Requires<[HasFPARMv8, HasDPVFP]> {
      bits<5> Dm;
```
- EN: Defines TableGen record `UD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `UD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1015-1015
```tablegen
      let Inst{17-16} = rm;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1017-1022
```tablegen
      // Encode instruction operands
      let Inst{3-0}  = Dm{3-0};
      let Inst{5}    = Dm{4};
      let Inst{8} = 1;
    }
  }
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1024-1029
```tablegen
  let Predicates = [HasFPARMv8] in {
    let Predicates = [HasFullFP16] in {
    def : Pat<(i32 (any_fp_to_sint (node (f16 HPR:$a)))),
              (COPY_TO_REGCLASS
                (!cast<Instruction>(NAME#"SH") (f16 HPR:$a)),
                GPR)>;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1031-1048
```tablegen
    def : Pat<(i32 (any_fp_to_uint (node (f16 HPR:$a)))),
              (COPY_TO_REGCLASS
                (!cast<Instruction>(NAME#"UH") (f16 HPR:$a)),
                GPR)>;
    }
    def : Pat<(i32 (any_fp_to_sint (node SPR:$a))),
              (COPY_TO_REGCLASS
                (!cast<Instruction>(NAME#"SS") SPR:$a),
                GPR)>;
    def : Pat<(i32 (any_fp_to_uint (node SPR:$a))),
              (COPY_TO_REGCLASS
                (!cast<Instruction>(NAME#"US") SPR:$a),
                GPR)>;
  }
  let Predicates = [HasFPARMv8, HasDPVFP] in {
    def : Pat<(i32 (any_fp_to_sint (node (f64 DPR:$a)))),
              (COPY_TO_REGCLASS
                (!cast<Instruction>(NAME#"SD") DPR:$a),
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1049-1055
```tablegen
                GPR)>;
    def : Pat<(i32 (any_fp_to_uint (node (f64 DPR:$a)))),
              (COPY_TO_REGCLASS
                (!cast<Instruction>(NAME#"UD") DPR:$a),
                GPR)>;
  }
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1057-1060
```tablegen
defm VCVTA : vcvt_inst<"a", 0b00, any_fround>;
defm VCVTN : vcvt_inst<"n", 0b01>;
defm VCVTP : vcvt_inst<"p", 0b10, any_fceil>;
defm VCVTM : vcvt_inst<"m", 0b11, any_ffloor>;
```
- EN: Defines TableGen record `VCVTA` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VCVTA`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1062-1065
```tablegen
def VNEGD  : ADuI<0b11101, 0b11, 0b0001, 0b01, 0,
                  (outs DPR:$Dd), (ins DPR:$Dm),
                  IIC_fpUNA64, "vneg", ".f64\t$Dd, $Dm", "",
                  [(set DPR:$Dd, (fneg (f64 DPR:$Dm)))]>;
```
- EN: Defines TableGen record `VNEGD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VNEGD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1067-1074
```tablegen
def VNEGS  : ASuIn<0b11101, 0b11, 0b0001, 0b01, 0,
                   (outs SPR:$Sd), (ins SPR:$Sm),
                   IIC_fpUNA32, "vneg", ".f32\t$Sd, $Sm",
                   [(set SPR:$Sd, (fneg SPR:$Sm))]> {
  // Some single precision VFP instructions may be executed on both NEON and
  // VFP pipelines on A8.
  let D = VFPNeonA8Domain;
}
```
- EN: Defines TableGen record `VNEGS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VNEGS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1076-1079
```tablegen
def VNEGH  : AHuI<0b11101, 0b11, 0b0001, 0b01, 0,
                  (outs HPR:$Sd), (ins HPR:$Sm),
                  IIC_fpUNA16, "vneg", ".f16\t$Sd, $Sm",
                  [(set (f16 HPR:$Sd), (fneg (f16 HPR:$Sm)))]>;
```
- EN: Defines TableGen record `VNEGH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VNEGH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1081-1091
```tablegen
multiclass vrint_inst_zrx<string opc, bit op, bit op2, SDPatternOperator node,
                          list<Register> uses = [], bit fpexc = 0> {
  let Uses = uses, mayRaiseFPException = fpexc in {
  def H : AHuI<0b11101, 0b11, 0b0110, 0b11, 0,
               (outs HPR:$Sd), (ins HPR:$Sm),
               NoItinerary, !strconcat("vrint", opc), ".f16\t$Sd, $Sm",
               [(set (f16 HPR:$Sd), (node (f16 HPR:$Sm)))]>,
               Requires<[HasFullFP16]> {
    let Inst{7} = op2;
    let Inst{16} = op;
  }
```
- EN: Declares TableGen `multiclass vrint_inst_zrx`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass vrint_inst_zrx`，它是一个可复用模板，可展开为多个相关记录。

### Lines 1093-1109
```tablegen
  def S : ASuI<0b11101, 0b11, 0b0110, 0b11, 0,
               (outs SPR:$Sd), (ins SPR:$Sm),
               NoItinerary, !strconcat("vrint", opc), ".f32\t$Sd, $Sm", "",
               [(set (f32 SPR:$Sd), (node (f32 SPR:$Sm)))]>,
               Requires<[HasFPARMv8]> {
    let Inst{7} = op2;
    let Inst{16} = op;
  }
  def D : ADuI<0b11101, 0b11, 0b0110, 0b11, 0,
                (outs DPR:$Dd), (ins DPR:$Dm),
                NoItinerary, !strconcat("vrint", opc), ".f64\t$Dd, $Dm", "",
                [(set (f64 DPR:$Dd), (node (f64 DPR:$Dm)))]>,
                Requires<[HasFPARMv8, HasDPVFP]> {
    let Inst{7} = op2;
    let Inst{16} = op;
  }
  }
```
- EN: Defines TableGen record `S` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `S`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1111-1120
```tablegen
  def : InstAlias<!strconcat("vrint", opc, "$p.f16.f16\t$Sd, $Sm"),
                  (!cast<Instruction>(NAME#"H") SPR:$Sd, SPR:$Sm, pred:$p), 0>,
        Requires<[HasFullFP16]>;
  def : InstAlias<!strconcat("vrint", opc, "$p.f32.f32\t$Sd, $Sm"),
                  (!cast<Instruction>(NAME#"S") SPR:$Sd, SPR:$Sm, pred:$p), 0>,
        Requires<[HasFPARMv8]>;
  def : InstAlias<!strconcat("vrint", opc, "$p.f64.f64\t$Dd, $Dm"),
                  (!cast<Instruction>(NAME#"D") DPR:$Dd, DPR:$Dm, pred:$p), 0>,
        Requires<[HasFPARMv8,HasDPVFP]>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1122-1124
```tablegen
defm VRINTZ : vrint_inst_zrx<"z", 0, 1, any_ftrunc, [], 0>;
defm VRINTR : vrint_inst_zrx<"r", 0, 0, any_fnearbyint, [FPSCR_RM], 0>;
defm VRINTX : vrint_inst_zrx<"x", 1, 0, any_frint, [FPSCR_RM], 1>;
```
- EN: Defines TableGen record `VRINTZ` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VRINTZ`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1126-1143
```tablegen
multiclass vrint_inst_anpm<string opc, bits<2> rm,
                           SDPatternOperator node = null_frag> {
  let PostEncoderMethod = "", DecoderNamespace = "VFPV8",
      isUnpredicable = 1 in {
    def H : AHuInp<0b11101, 0b11, 0b1000, 0b01, 0,
                   (outs HPR:$Sd), (ins HPR:$Sm),
                   NoItinerary, !strconcat("vrint", opc, ".f16\t$Sd, $Sm"),
                   [(set (f16 HPR:$Sd), (node (f16 HPR:$Sm)))]>,
                   Requires<[HasFullFP16]> {
      let Inst{17-16} = rm;
    }
    def S : ASuInp<0b11101, 0b11, 0b1000, 0b01, 0,
                   (outs SPR:$Sd), (ins SPR:$Sm),
                   NoItinerary, !strconcat("vrint", opc, ".f32\t$Sd, $Sm"),
                   [(set (f32 SPR:$Sd), (node (f32 SPR:$Sm)))]>,
                   Requires<[HasFPARMv8]> {
      let Inst{17-16} = rm;
    }
```
- EN: Declares TableGen `multiclass vrint_inst_anpm`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass vrint_inst_anpm`，它是一个可复用模板，可展开为多个相关记录。

### Lines 1144-1151
```tablegen
    def D : ADuInp<0b11101, 0b11, 0b1000, 0b01, 0,
                   (outs DPR:$Dd), (ins DPR:$Dm),
                   NoItinerary, !strconcat("vrint", opc, ".f64\t$Dd, $Dm"),
                   [(set (f64 DPR:$Dd), (node (f64 DPR:$Dm)))]>,
                   Requires<[HasFPARMv8, HasDPVFP]> {
      let Inst{17-16} = rm;
    }
  }
```
- EN: Defines TableGen record `D` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `D`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1153-1162
```tablegen
  def : InstAlias<!strconcat("vrint", opc, ".f16.f16\t$Sd, $Sm"),
                  (!cast<Instruction>(NAME#"H") HPR:$Sd, HPR:$Sm), 0>,
        Requires<[HasFullFP16]>;
  def : InstAlias<!strconcat("vrint", opc, ".f32.f32\t$Sd, $Sm"),
                  (!cast<Instruction>(NAME#"S") SPR:$Sd, SPR:$Sm), 0>,
        Requires<[HasFPARMv8]>;
  def : InstAlias<!strconcat("vrint", opc, ".f64.f64\t$Dd, $Dm"),
                  (!cast<Instruction>(NAME#"D") DPR:$Dd, DPR:$Dm), 0>,
        Requires<[HasFPARMv8,HasDPVFP]>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1164-1167
```tablegen
defm VRINTA : vrint_inst_anpm<"a", 0b00, any_fround>;
defm VRINTN : vrint_inst_anpm<"n", 0b01, any_froundeven>;
defm VRINTP : vrint_inst_anpm<"p", 0b10, any_fceil>;
defm VRINTM : vrint_inst_anpm<"m", 0b11, any_ffloor>;
```
- EN: Defines TableGen record `VRINTA` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VRINTA`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1170-1175
```tablegen
let mayRaiseFPException = 1, Uses = [FPSCR_RM] in
def VSQRTD : ADuI<0b11101, 0b11, 0b0001, 0b11, 0,
                  (outs DPR:$Dd), (ins DPR:$Dm),
                  IIC_fpSQRT64, "vsqrt", ".f64\t$Dd, $Dm", "",
                  [(set DPR:$Dd, (any_fsqrt (f64 DPR:$Dm)))]>,
             Sched<[WriteFPSQRT64]>;
```
- EN: Defines TableGen record `VSQRTD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VSQRTD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1177-1182
```tablegen
let mayRaiseFPException = 1, Uses = [FPSCR_RM] in
def VSQRTS : ASuI<0b11101, 0b11, 0b0001, 0b11, 0,
                  (outs SPR:$Sd), (ins SPR:$Sm),
                  IIC_fpSQRT32, "vsqrt", ".f32\t$Sd, $Sm", "",
                  [(set SPR:$Sd, (any_fsqrt SPR:$Sm))]>,
             Sched<[WriteFPSQRT32]>;
```
- EN: Defines TableGen record `VSQRTS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VSQRTS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1184-1188
```tablegen
let mayRaiseFPException = 1, Uses = [FPSCR_RM] in
def VSQRTH : AHuI<0b11101, 0b11, 0b0001, 0b11, 0,
                  (outs HPR:$Sd), (ins HPR:$Sm),
                  IIC_fpSQRT16, "vsqrt", ".f16\t$Sd, $Sm",
                  [(set (f16 HPR:$Sd), (any_fsqrt (f16 HPR:$Sm)))]>;
```
- EN: Defines TableGen record `VSQRTH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VSQRTH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1190-1195
```tablegen
let hasSideEffects = 0 in {
let isMoveReg = 1 in {
def VMOVD  : ADuI<0b11101, 0b11, 0b0000, 0b01, 0,
                  (outs DPR:$Dd), (ins DPR:$Dm),
                  IIC_fpUNA64, "vmov", ".f64\t$Dd, $Dm", "", []>,
             Requires<[HasFPRegs64]>;
```
- EN: Defines TableGen record `VMOVD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VMOVD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1197-1201
```tablegen
def VMOVS  : ASuI<0b11101, 0b11, 0b0000, 0b01, 0,
                  (outs SPR:$Sd), (ins SPR:$Sm),
                  IIC_fpUNA32, "vmov", ".f32\t$Sd, $Sm", "", []>,
             Requires<[HasFPRegs]>;
} // isMoveReg
```
- EN: Defines TableGen record `VMOVS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VMOVS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1203-1207
```tablegen
let PostEncoderMethod = "", DecoderNamespace = "VFPV8", isUnpredicable = 1 in {
def VMOVH  : ASuInp<0b11101, 0b11, 0b0000, 0b01, 0,
                  (outs SPR:$Sd), (ins SPR:$Sm),
                  IIC_fpUNA16, "vmovx.f16\t$Sd, $Sm", []>,
             Requires<[HasFullFP16]>;
```
- EN: Defines TableGen record `VMOVH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VMOVH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1209-1214
```tablegen
def VINSH  : ASuInp<0b11101, 0b11, 0b0000, 0b11, 0,
                  (outs SPR:$Sd), (ins SPR:$Sda, SPR:$Sm),
                  IIC_fpUNA16, "vins.f16\t$Sd, $Sm", []>,
             Requires<[HasFullFP16]> {
  let Constraints = "$Sd = $Sda";
}
```
- EN: Defines TableGen record `VINSH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VINSH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1216-1217
```tablegen
} // PostEncoderMethod
} // hasSideEffects
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1219-1221
```tablegen
//===----------------------------------------------------------------------===//
// FP <-> GPR Copies.  Int <-> FP Conversions.
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 1223-1232
```tablegen
let isMoveReg = 1 in {
def VMOVRS : AVConv2I<0b11100001, 0b1010,
                      (outs GPR:$Rt), (ins SPR:$Sn),
                      IIC_fpMOVSI, "vmov", "\t$Rt, $Sn",
                      [(set GPR:$Rt, (bitconvert SPR:$Sn))]>,
             Requires<[HasFPRegs]>,
             Sched<[WriteFPMOV]> {
  // Instruction operands.
  bits<4> Rt;
  bits<5> Sn;
```
- EN: Defines TableGen record `VMOVRS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VMOVRS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1234-1237
```tablegen
  // Encode instruction operands.
  let Inst{19-16} = Sn{4-1};
  let Inst{7}     = Sn{0};
  let Inst{15-12} = Rt;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1239-1240
```tablegen
  let Inst{6-5}   = 0b00;
  let Inst{3-0}   = 0b0000;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1242-1245
```tablegen
  // Some single precision VFP instructions may be executed on both NEON and VFP
  // pipelines.
  let D = VFPNeonDomain;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1247-1256
```tablegen
// Bitcast i32 -> f32.  NEON prefers to use VMOVDRR.
def VMOVSR : AVConv4I<0b11100000, 0b1010,
                      (outs SPR:$Sn), (ins GPR:$Rt),
                      IIC_fpMOVIS, "vmov", "\t$Sn, $Rt",
                      [(set SPR:$Sn, (bitconvert GPR:$Rt))]>,
             Requires<[HasFPRegs, UseVMOVSR]>,
             Sched<[WriteFPMOV]> {
  // Instruction operands.
  bits<5> Sn;
  bits<4> Rt;
```
- EN: Defines TableGen record `VMOVSR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VMOVSR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1258-1261
```tablegen
  // Encode instruction operands.
  let Inst{19-16} = Sn{4-1};
  let Inst{7}     = Sn{0};
  let Inst{15-12} = Rt;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1263-1264
```tablegen
  let Inst{6-5}   = 0b00;
  let Inst{3-0}   = 0b0000;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1266-1271
```tablegen
  // Some single precision VFP instructions may be executed on both NEON and VFP
  // pipelines.
  let D = VFPNeonDomain;
}
} // isMoveReg
def : Pat<(arm_vmovsr GPR:$Rt), (VMOVSR GPR:$Rt)>, Requires<[HasFPRegs, UseVMOVSR]>;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1273-1283
```tablegen
let hasSideEffects = 0 in {
def VMOVRRD  : AVConv3I<0b11000101, 0b1011,
                        (outs GPR:$Rt, GPR:$Rt2), (ins DPR:$Dm),
                        IIC_fpMOVDI, "vmov", "\t$Rt, $Rt2, $Dm",
                 [(set GPR:$Rt, GPR:$Rt2, (arm_fmrrd DPR:$Dm))]>,
               Requires<[HasFPRegs]>,
               Sched<[WriteFPMOV]> {
  // Instruction operands.
  bits<5> Dm;
  bits<4> Rt;
  bits<4> Rt2;
```
- EN: Defines TableGen record `VMOVRRD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VMOVRRD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1285-1289
```tablegen
  // Encode instruction operands.
  let Inst{3-0}   = Dm{3-0};
  let Inst{5}     = Dm{4};
  let Inst{15-12} = Rt;
  let Inst{19-16} = Rt2;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1291-1291
```tablegen
  let Inst{7-6} = 0b00;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1293-1295
```tablegen
  // Some single precision VFP instructions may be executed on both NEON and VFP
  // pipelines.
  let D = VFPNeonDomain;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1297-1301
```tablegen
  // This instruction is equivalent to
  // $Rt = EXTRACT_SUBREG $Dm, ssub_0
  // $Rt2 = EXTRACT_SUBREG $Dm, ssub_1
  let isExtractSubreg = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1303-1311
```tablegen
def VMOVRRS  : AVConv3I<0b11000101, 0b1010,
                      (outs GPR:$Rt, GPR:$Rt2), (ins SPR:$src1, SPR:$src2),
                 IIC_fpMOVDI, "vmov", "\t$Rt, $Rt2, $src1, $src2",
                 [/* For disassembly only; pattern left blank */]>,
               Requires<[HasFPRegs]>,
               Sched<[WriteFPMOV]> {
  bits<5> src1;
  bits<4> Rt;
  bits<4> Rt2;
```
- EN: Defines TableGen record `VMOVRRS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VMOVRRS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1313-1317
```tablegen
  // Encode instruction operands.
  let Inst{3-0}   = src1{4-1};
  let Inst{5}     = src1{0};
  let Inst{15-12} = Rt;
  let Inst{19-16} = Rt2;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1319-1319
```tablegen
  let Inst{7-6} = 0b00;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1321-1326
```tablegen
  // Some single precision VFP instructions may be executed on both NEON and VFP
  // pipelines.
  let D = VFPNeonDomain;
  let DecoderMethod = "DecodeVMOVRRS";
}
} // hasSideEffects
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1328-1329
```tablegen
// FMDHR: GPR -> SPR
// FMDLR: GPR -> SPR
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 1331-1340
```tablegen
def VMOVDRR : AVConv5I<0b11000100, 0b1011,
                      (outs DPR:$Dm), (ins GPR:$Rt, GPR:$Rt2),
                      IIC_fpMOVID, "vmov", "\t$Dm, $Rt, $Rt2",
                      [(set DPR:$Dm, (arm_fmdrr GPR:$Rt, GPR:$Rt2))]>,
              Requires<[HasFPRegs]>,
              Sched<[WriteFPMOV]> {
  // Instruction operands.
  bits<5> Dm;
  bits<4> Rt;
  bits<4> Rt2;
```
- EN: Defines TableGen record `VMOVDRR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VMOVDRR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1342-1346
```tablegen
  // Encode instruction operands.
  let Inst{3-0}   = Dm{3-0};
  let Inst{5}     = Dm{4};
  let Inst{15-12} = Rt;
  let Inst{19-16} = Rt2;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1348-1348
```tablegen
  let Inst{7-6}   = 0b00;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1350-1352
```tablegen
  // Some single precision VFP instructions may be executed on both NEON and VFP
  // pipelines.
  let D = VFPNeonDomain;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1354-1357
```tablegen
  // This instruction is equivalent to
  // $Dm = REG_SEQUENCE $Rt, ssub_0, $Rt2, ssub_1
  let isRegSequence = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1359-1373
```tablegen
// Hoist an fabs or a fneg of a value coming from integer registers
// and do the fabs/fneg on the integer value. This is never a lose
// and could enable the conversion to float to be removed completely.
def : Pat<(fabs (arm_fmdrr GPR:$Rl, GPR:$Rh)),
          (VMOVDRR GPR:$Rl, (BFC GPR:$Rh, (i32 0x7FFFFFFF)))>,
      Requires<[IsARM, HasV6T2]>;
def : Pat<(fabs (arm_fmdrr GPR:$Rl, GPR:$Rh)),
          (VMOVDRR GPR:$Rl, (t2BFC GPR:$Rh, (i32 0x7FFFFFFF)))>,
      Requires<[IsThumb2, HasV6T2]>;
def : Pat<(fneg (arm_fmdrr GPR:$Rl, GPR:$Rh)),
          (VMOVDRR GPR:$Rl, (EORri GPR:$Rh, (i32 0x80000000)))>,
      Requires<[IsARM]>;
def : Pat<(fneg (arm_fmdrr GPR:$Rl, GPR:$Rh)),
          (VMOVDRR GPR:$Rl, (t2EORri GPR:$Rh, (i32 0x80000000)))>,
      Requires<[IsThumb2]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1375-1385
```tablegen
let hasSideEffects = 0 in
def VMOVSRR : AVConv5I<0b11000100, 0b1010,
                     (outs SPR:$dst1, SPR:$dst2), (ins GPR:$src1, GPR:$src2),
                IIC_fpMOVID, "vmov", "\t$dst1, $dst2, $src1, $src2",
                [/* For disassembly only; pattern left blank */]>,
              Requires<[HasFPRegs]>,
              Sched<[WriteFPMOV]> {
  // Instruction operands.
  bits<5> dst1;
  bits<4> src1;
  bits<4> src2;
```
- EN: Defines TableGen record `VMOVSRR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VMOVSRR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1387-1391
```tablegen
  // Encode instruction operands.
  let Inst{3-0}   = dst1{4-1};
  let Inst{5}     = dst1{0};
  let Inst{15-12} = src1;
  let Inst{19-16} = src2;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1393-1393
```tablegen
  let Inst{7-6} = 0b00;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1395-1397
```tablegen
  // Some single precision VFP instructions may be executed on both NEON and VFP
  // pipelines.
  let D = VFPNeonDomain;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1399-1400
```tablegen
  let DecoderMethod = "DecodeVMOVSRR";
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1402-1411
```tablegen
// Move H->R, clearing top 16 bits
def VMOVRH : AVConv2I<0b11100001, 0b1001,
                      (outs rGPR:$Rt), (ins HPR:$Sn),
                      IIC_fpMOVSI, "vmov", ".f16\t$Rt, $Sn",
                      []>,
             Requires<[HasFPRegs16]>,
             Sched<[WriteFPMOV]> {
  // Instruction operands.
  bits<4> Rt;
  bits<5> Sn;
```
- EN: Defines TableGen record `VMOVRH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VMOVRH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1413-1416
```tablegen
  // Encode instruction operands.
  let Inst{19-16} = Sn{4-1};
  let Inst{7}     = Sn{0};
  let Inst{15-12} = Rt;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1418-1419
```tablegen
  let Inst{6-5}   = 0b00;
  let Inst{3-0}   = 0b0000;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1421-1422
```tablegen
  let isUnpredicable = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1424-1433
```tablegen
// Move R->H, clearing top 16 bits
def VMOVHR : AVConv4I<0b11100000, 0b1001,
                      (outs HPR:$Sn), (ins rGPR:$Rt),
                      IIC_fpMOVIS, "vmov", ".f16\t$Sn, $Rt",
                      []>,
             Requires<[HasFPRegs16]>,
             Sched<[WriteFPMOV]> {
  // Instruction operands.
  bits<5> Sn;
  bits<4> Rt;
```
- EN: Defines TableGen record `VMOVHR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VMOVHR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1435-1438
```tablegen
  // Encode instruction operands.
  let Inst{19-16} = Sn{4-1};
  let Inst{7}     = Sn{0};
  let Inst{15-12} = Rt;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1440-1441
```tablegen
  let Inst{6-5}   = 0b00;
  let Inst{3-0}   = 0b0000;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1443-1444
```tablegen
  let isUnpredicable = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1446-1449
```tablegen
def : FPRegs16Pat<(arm_vmovrh (f16 HPR:$Sn)), (VMOVRH (f16 HPR:$Sn))>;
def : FPRegs16Pat<(arm_vmovrh (bf16 HPR:$Sn)), (VMOVRH (bf16 HPR:$Sn))>;
def : FPRegs16Pat<(f16 (arm_vmovhr rGPR:$Rt)), (VMOVHR rGPR:$Rt)>;
def : FPRegs16Pat<(bf16 (arm_vmovhr rGPR:$Rt)), (VMOVHR rGPR:$Rt)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1451-1456
```tablegen
// FMRDH: SPR -> GPR
// FMRDL: SPR -> GPR
// FMRRS: SPR -> GPR
// FMRX:  SPR system reg -> GPR
// FMSRR: GPR -> SPR
// FMXR:  GPR -> VFP system reg
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 1459-1459
```tablegen
// Int -> FP:
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 1461-1469
```tablegen
class AVConv1IDs_Encode<bits<5> opcod1, bits<2> opcod2, bits<4> opcod3,
                        bits<4> opcod4, dag oops, dag iops,
                        InstrItinClass itin, string opc, string asm,
                        list<dag> pattern>
  : AVConv1I<opcod1, opcod2, opcod3, opcod4, oops, iops, itin, opc, asm,
             pattern> {
  // Instruction operands.
  bits<5> Dd;
  bits<5> Sm;
```
- EN: Declares reusable TableGen class `AVConv1IDs_Encode` for `ARMInstrVFP`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrVFP` 声明可复用的 TableGen 类 `AVConv1IDs_Encode`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1471-1475
```tablegen
  // Encode instruction operands.
  let Inst{3-0}   = Sm{4-1};
  let Inst{5}     = Sm{0};
  let Inst{15-12} = Dd{3-0};
  let Inst{22}    = Dd{4};
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1477-1479
```tablegen
  let Predicates = [HasVFP2, HasDPVFP];
  let hasSideEffects = 0;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1481-1488
```tablegen
class AVConv1InSs_Encode<bits<5> opcod1, bits<2> opcod2, bits<4> opcod3,
                         bits<4> opcod4, dag oops, dag iops,InstrItinClass itin,
                         string opc, string asm, list<dag> pattern>
  : AVConv1In<opcod1, opcod2, opcod3, opcod4, oops, iops, itin, opc, asm,
              pattern> {
  // Instruction operands.
  bits<5> Sd;
  bits<5> Sm;
```
- EN: Declares reusable TableGen class `AVConv1InSs_Encode` for `ARMInstrVFP`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrVFP` 声明可复用的 TableGen 类 `AVConv1InSs_Encode`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1490-1494
```tablegen
  // Encode instruction operands.
  let Inst{3-0}   = Sm{4-1};
  let Inst{5}     = Sm{0};
  let Inst{15-12} = Sd{4-1};
  let Inst{22}    = Sd{0};
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1496-1497
```tablegen
  let hasSideEffects = 0;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1499-1507
```tablegen
class AVConv1IHs_Encode<bits<5> opcod1, bits<2> opcod2, bits<4> opcod3,
                        bits<4> opcod4, dag oops, dag iops,
                        InstrItinClass itin, string opc, string asm,
                        list<dag> pattern>
  : AVConv1I<opcod1, opcod2, opcod3, opcod4, oops, iops, itin, opc, asm,
             pattern> {
  // Instruction operands.
  bits<5> Sd;
  bits<5> Sm;
```
- EN: Declares reusable TableGen class `AVConv1IHs_Encode` for `ARMInstrVFP`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrVFP` 声明可复用的 TableGen 类 `AVConv1IHs_Encode`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1509-1513
```tablegen
  // Encode instruction operands.
  let Inst{3-0}   = Sm{4-1};
  let Inst{5}     = Sm{0};
  let Inst{15-12} = Sd{4-1};
  let Inst{22}    = Sd{0};
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1515-1517
```tablegen
  let Predicates = [HasFullFP16];
  let hasSideEffects = 0;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1519-1526
```tablegen
let mayRaiseFPException = 1 in 
def VSITOD : AVConv1IDs_Encode<0b11101, 0b11, 0b1000, 0b1011,
                               (outs DPR:$Dd), (ins SPR:$Sm),
                               IIC_fpCVTID, "vcvt", ".f64.s32\t$Dd, $Sm",
                               []>,
             Sched<[WriteFPCVT]> {
  let Inst{7} = 1; // s32
}
```
- EN: Defines TableGen record `VSITOD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VSITOD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1528-1530
```tablegen
let Predicates=[HasVFP2, HasDPVFP] in {
  def : VFPPat<(f64 (any_sint_to_fp GPR:$a)),
               (VSITOD (COPY_TO_REGCLASS GPR:$a, SPR))>;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1532-1534
```tablegen
  def : VFPPat<(f64 (any_sint_to_fp (i32 (alignedload32 addrmode5:$a)))),
               (VSITOD (VLDRS addrmode5:$a))>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1536-1542
```tablegen
let mayRaiseFPException = 1 in 
def VSITOS : AVConv1InSs_Encode<0b11101, 0b11, 0b1000, 0b1010,
                                (outs SPR:$Sd),(ins SPR:$Sm),
                                IIC_fpCVTIS, "vcvt", ".f32.s32\t$Sd, $Sm",
                                []>,
             Sched<[WriteFPCVT]> {
  let Inst{7} = 1; // s32
```
- EN: Defines TableGen record `VSITOS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VSITOS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1544-1547
```tablegen
  // Some single precision VFP instructions may be executed on both NEON and
  // VFP pipelines on A8.
  let D = VFPNeonA8Domain;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1549-1550
```tablegen
def : VFPNoNEONPat<(f32 (any_sint_to_fp GPR:$a)),
                   (VSITOS (COPY_TO_REGCLASS GPR:$a, SPR))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1552-1553
```tablegen
def : VFPNoNEONPat<(f32 (any_sint_to_fp (i32 (alignedload32 addrmode5:$a)))),
                   (VSITOS (VLDRS addrmode5:$a))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1555-1563
```tablegen
let mayRaiseFPException = 1 in 
def VSITOH : AVConv1IHs_Encode<0b11101, 0b11, 0b1000, 0b1001,
                               (outs HPR:$Sd), (ins SPR:$Sm),
                               IIC_fpCVTIH, "vcvt", ".f16.s32\t$Sd, $Sm",
                               []>,
             Sched<[WriteFPCVT]> {
  let Inst{7} = 1; // s32
  let isUnpredicable = 1;
}
```
- EN: Defines TableGen record `VSITOH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VSITOH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1565-1566
```tablegen
def : VFPNoNEONPat<(f16 (any_sint_to_fp GPR:$a)),
                   (VSITOH (COPY_TO_REGCLASS GPR:$a, SPR))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1568-1575
```tablegen
let mayRaiseFPException = 1 in 
def VUITOD : AVConv1IDs_Encode<0b11101, 0b11, 0b1000, 0b1011,
                               (outs DPR:$Dd), (ins SPR:$Sm),
                               IIC_fpCVTID, "vcvt", ".f64.u32\t$Dd, $Sm",
                               []>,
             Sched<[WriteFPCVT]> {
  let Inst{7} = 0; // u32
}
```
- EN: Defines TableGen record `VUITOD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VUITOD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1577-1579
```tablegen
let Predicates=[HasVFP2, HasDPVFP] in {
  def : VFPPat<(f64 (any_uint_to_fp GPR:$a)),
               (VUITOD (COPY_TO_REGCLASS GPR:$a, SPR))>;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1581-1583
```tablegen
  def : VFPPat<(f64 (any_uint_to_fp (i32 (alignedload32 addrmode5:$a)))),
               (VUITOD (VLDRS addrmode5:$a))>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1585-1591
```tablegen
let mayRaiseFPException = 1 in 
def VUITOS : AVConv1InSs_Encode<0b11101, 0b11, 0b1000, 0b1010,
                                (outs SPR:$Sd), (ins SPR:$Sm),
                                IIC_fpCVTIS, "vcvt", ".f32.u32\t$Sd, $Sm",
                                []>,
             Sched<[WriteFPCVT]> {
  let Inst{7} = 0; // u32
```
- EN: Defines TableGen record `VUITOS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VUITOS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1593-1596
```tablegen
  // Some single precision VFP instructions may be executed on both NEON and
  // VFP pipelines on A8.
  let D = VFPNeonA8Domain;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1598-1599
```tablegen
def : VFPNoNEONPat<(f32 (any_uint_to_fp GPR:$a)),
                   (VUITOS (COPY_TO_REGCLASS GPR:$a, SPR))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1601-1602
```tablegen
def : VFPNoNEONPat<(f32 (any_uint_to_fp (i32 (alignedload32 addrmode5:$a)))),
                   (VUITOS (VLDRS addrmode5:$a))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1604-1612
```tablegen
let mayRaiseFPException = 1 in 
def VUITOH : AVConv1IHs_Encode<0b11101, 0b11, 0b1000, 0b1001,
                                (outs HPR:$Sd), (ins SPR:$Sm),
                                IIC_fpCVTIH, "vcvt", ".f16.u32\t$Sd, $Sm",
                                []>,
             Sched<[WriteFPCVT]> {
  let Inst{7} = 0; // u32
  let isUnpredicable = 1;
}
```
- EN: Defines TableGen record `VUITOH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VUITOH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1614-1615
```tablegen
def : VFPNoNEONPat<(f16 (any_uint_to_fp GPR:$a)),
                   (VUITOH (COPY_TO_REGCLASS GPR:$a, SPR))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1617-1617
```tablegen
// FP -> Int:
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 1619-1627
```tablegen
class AVConv1IsD_Encode<bits<5> opcod1, bits<2> opcod2, bits<4> opcod3,
                        bits<4> opcod4, dag oops, dag iops,
                        InstrItinClass itin, string opc, string asm,
                        list<dag> pattern>
  : AVConv1I<opcod1, opcod2, opcod3, opcod4, oops, iops, itin, opc, asm,
             pattern> {
  // Instruction operands.
  bits<5> Sd;
  bits<5> Dm;
```
- EN: Declares reusable TableGen class `AVConv1IsD_Encode` for `ARMInstrVFP`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrVFP` 声明可复用的 TableGen 类 `AVConv1IsD_Encode`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1629-1633
```tablegen
  // Encode instruction operands.
  let Inst{3-0}   = Dm{3-0};
  let Inst{5}     = Dm{4};
  let Inst{15-12} = Sd{4-1};
  let Inst{22}    = Sd{0};
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1635-1637
```tablegen
  let Predicates = [HasVFP2, HasDPVFP];
  let hasSideEffects = 0;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1639-1647
```tablegen
class AVConv1InsS_Encode<bits<5> opcod1, bits<2> opcod2, bits<4> opcod3,
                         bits<4> opcod4, dag oops, dag iops,
                         InstrItinClass itin, string opc, string asm,
                         list<dag> pattern>
  : AVConv1In<opcod1, opcod2, opcod3, opcod4, oops, iops, itin, opc, asm,
              pattern> {
  // Instruction operands.
  bits<5> Sd;
  bits<5> Sm;
```
- EN: Declares reusable TableGen class `AVConv1InsS_Encode` for `ARMInstrVFP`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrVFP` 声明可复用的 TableGen 类 `AVConv1InsS_Encode`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1649-1653
```tablegen
  // Encode instruction operands.
  let Inst{3-0}   = Sm{4-1};
  let Inst{5}     = Sm{0};
  let Inst{15-12} = Sd{4-1};
  let Inst{22}    = Sd{0};
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1655-1656
```tablegen
  let hasSideEffects = 0;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1658-1666
```tablegen
class AVConv1IsH_Encode<bits<5> opcod1, bits<2> opcod2, bits<4> opcod3,
                         bits<4> opcod4, dag oops, dag iops,
                         InstrItinClass itin, string opc, string asm,
                         list<dag> pattern>
  : AVConv1I<opcod1, opcod2, opcod3, opcod4, oops, iops, itin, opc, asm,
              pattern> {
  // Instruction operands.
  bits<5> Sd;
  bits<5> Sm;
```
- EN: Declares reusable TableGen class `AVConv1IsH_Encode` for `ARMInstrVFP`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrVFP` 声明可复用的 TableGen 类 `AVConv1IsH_Encode`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1668-1672
```tablegen
  // Encode instruction operands.
  let Inst{3-0}   = Sm{4-1};
  let Inst{5}     = Sm{0};
  let Inst{15-12} = Sd{4-1};
  let Inst{22}    = Sd{0};
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1674-1676
```tablegen
  let Predicates = [HasFullFP16];
  let hasSideEffects = 0;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1678-1686
```tablegen
// Always set Z bit in the instruction, i.e. "round towards zero" variants.
let mayRaiseFPException = 1 in
def VTOSIZD : AVConv1IsD_Encode<0b11101, 0b11, 0b1101, 0b1011,
                                (outs SPR:$Sd), (ins DPR:$Dm),
                                IIC_fpCVTDI, "vcvt", ".s32.f64\t$Sd, $Dm",
                                []>,
              Sched<[WriteFPCVT]> {
  let Inst{7} = 1; // Z bit
}
```
- EN: Defines TableGen record `VTOSIZD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VTOSIZD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1688-1692
```tablegen
let Predicates=[HasVFP2, HasDPVFP] in {
  def : VFPPat<(i32 (any_fp_to_sint (f64 DPR:$a))),
               (COPY_TO_REGCLASS (VTOSIZD DPR:$a), GPR)>;
  def : VFPPat<(i32 (fp_to_sint_sat (f64 DPR:$a), i32)),
               (COPY_TO_REGCLASS (VTOSIZD DPR:$a), GPR)>;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1694-1698
```tablegen
  def : VFPPat<(alignedstore32 (i32 (any_fp_to_sint (f64 DPR:$a))), addrmode5:$ptr),
               (VSTRS (VTOSIZD DPR:$a), addrmode5:$ptr)>;
  def : VFPPat<(alignedstore32 (i32 (fp_to_sint_sat (f64 DPR:$a), i32)), addrmode5:$ptr),
               (VSTRS (VTOSIZD DPR:$a), addrmode5:$ptr)>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1700-1706
```tablegen
let mayRaiseFPException = 1 in
def VTOSIZS : AVConv1InsS_Encode<0b11101, 0b11, 0b1101, 0b1010,
                                 (outs SPR:$Sd), (ins SPR:$Sm),
                                 IIC_fpCVTSI, "vcvt", ".s32.f32\t$Sd, $Sm",
                                 []>,
              Sched<[WriteFPCVT]> {
  let Inst{7} = 1; // Z bit
```
- EN: Defines TableGen record `VTOSIZS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VTOSIZS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1708-1711
```tablegen
  // Some single precision VFP instructions may be executed on both NEON and
  // VFP pipelines on A8.
  let D = VFPNeonA8Domain;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1713-1716
```tablegen
def : VFPNoNEONPat<(i32 (any_fp_to_sint SPR:$a)),
                   (COPY_TO_REGCLASS (VTOSIZS SPR:$a), GPR)>;
def : VFPPat<(i32 (fp_to_sint_sat SPR:$a, i32)),
             (COPY_TO_REGCLASS (VTOSIZS SPR:$a), GPR)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1718-1723
```tablegen
def : VFPNoNEONPat<(alignedstore32 (i32 (any_fp_to_sint (f32 SPR:$a))),
                                   addrmode5:$ptr),
                   (VSTRS (VTOSIZS SPR:$a), addrmode5:$ptr)>;
def : VFPPat<(alignedstore32 (i32 (fp_to_sint_sat (f32 SPR:$a), i32)),
                                   addrmode5:$ptr),
             (VSTRS (VTOSIZS SPR:$a), addrmode5:$ptr)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1725-1733
```tablegen
let mayRaiseFPException = 1 in
def VTOSIZH : AVConv1IsH_Encode<0b11101, 0b11, 0b1101, 0b1001,
                                 (outs SPR:$Sd), (ins HPR:$Sm),
                                 IIC_fpCVTHI, "vcvt", ".s32.f16\t$Sd, $Sm",
                                 []>,
              Sched<[WriteFPCVT]> {
  let Inst{7} = 1; // Z bit
  let isUnpredicable = 1;
}
```
- EN: Defines TableGen record `VTOSIZH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VTOSIZH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1735-1738
```tablegen
def : VFPNoNEONPat<(i32 (any_fp_to_sint (f16 HPR:$a))),
                   (COPY_TO_REGCLASS (VTOSIZH (f16 HPR:$a)), GPR)>;
def : VFPPat<(i32 (fp_to_sint_sat (f16 HPR:$a), i32)),
             (COPY_TO_REGCLASS (VTOSIZH (f16 HPR:$a)), GPR)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1740-1747
```tablegen
let mayRaiseFPException = 1 in
def VTOUIZD : AVConv1IsD_Encode<0b11101, 0b11, 0b1100, 0b1011,
                               (outs SPR:$Sd), (ins DPR:$Dm),
                               IIC_fpCVTDI, "vcvt", ".u32.f64\t$Sd, $Dm",
                               []>,
              Sched<[WriteFPCVT]> {
  let Inst{7} = 1; // Z bit
}
```
- EN: Defines TableGen record `VTOUIZD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VTOUIZD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1749-1753
```tablegen
let Predicates=[HasVFP2, HasDPVFP] in {
  def : VFPPat<(i32 (any_fp_to_uint (f64 DPR:$a))),
               (COPY_TO_REGCLASS (VTOUIZD DPR:$a), GPR)>;
  def : VFPPat<(i32 (fp_to_uint_sat (f64 DPR:$a), i32)),
               (COPY_TO_REGCLASS (VTOUIZD DPR:$a), GPR)>;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1755-1759
```tablegen
  def : VFPPat<(alignedstore32 (i32 (any_fp_to_uint (f64 DPR:$a))), addrmode5:$ptr),
               (VSTRS (VTOUIZD DPR:$a), addrmode5:$ptr)>;
  def : VFPPat<(alignedstore32 (i32 (fp_to_uint_sat (f64 DPR:$a), i32)), addrmode5:$ptr),
               (VSTRS (VTOUIZD DPR:$a), addrmode5:$ptr)>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1761-1767
```tablegen
let mayRaiseFPException = 1 in
def VTOUIZS : AVConv1InsS_Encode<0b11101, 0b11, 0b1100, 0b1010,
                                 (outs SPR:$Sd), (ins SPR:$Sm),
                                 IIC_fpCVTSI, "vcvt", ".u32.f32\t$Sd, $Sm",
                                 []>,
              Sched<[WriteFPCVT]> {
  let Inst{7} = 1; // Z bit
```
- EN: Defines TableGen record `VTOUIZS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VTOUIZS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1769-1772
```tablegen
  // Some single precision VFP instructions may be executed on both NEON and
  // VFP pipelines on A8.
  let D = VFPNeonA8Domain;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1774-1777
```tablegen
def : VFPNoNEONPat<(i32 (any_fp_to_uint SPR:$a)),
                   (COPY_TO_REGCLASS (VTOUIZS SPR:$a), GPR)>;
def : VFPPat<(i32 (fp_to_uint_sat SPR:$a, i32)),
             (COPY_TO_REGCLASS (VTOUIZS SPR:$a), GPR)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1779-1784
```tablegen
def : VFPNoNEONPat<(alignedstore32 (i32 (any_fp_to_uint (f32 SPR:$a))),
                                   addrmode5:$ptr),
                  (VSTRS (VTOUIZS SPR:$a), addrmode5:$ptr)>;
def : VFPPat<(alignedstore32 (i32 (fp_to_uint_sat (f32 SPR:$a), i32)),
                                   addrmode5:$ptr),
             (VSTRS (VTOUIZS SPR:$a), addrmode5:$ptr)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1786-1794
```tablegen
let mayRaiseFPException = 1 in
def VTOUIZH : AVConv1IsH_Encode<0b11101, 0b11, 0b1100, 0b1001,
                                 (outs SPR:$Sd), (ins HPR:$Sm),
                                 IIC_fpCVTHI, "vcvt", ".u32.f16\t$Sd, $Sm",
                                 []>,
              Sched<[WriteFPCVT]> {
  let Inst{7} = 1; // Z bit
  let isUnpredicable = 1;
}
```
- EN: Defines TableGen record `VTOUIZH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VTOUIZH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1796-1799
```tablegen
def : VFPNoNEONPat<(i32 (any_fp_to_uint (f16 HPR:$a))),
                   (COPY_TO_REGCLASS (VTOUIZH (f16 HPR:$a)), GPR)>;
def : VFPPat<(i32 (fp_to_uint_sat (f16 HPR:$a), i32)),
             (COPY_TO_REGCLASS (VTOUIZH (f16 HPR:$a)), GPR)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1801-1809
```tablegen
// And the Z bit '0' variants, i.e. use the rounding mode specified by FPSCR.
let mayRaiseFPException = 1, Uses = [FPSCR_RM] in {
def VTOSIRD : AVConv1IsD_Encode<0b11101, 0b11, 0b1101, 0b1011,
                                (outs SPR:$Sd), (ins DPR:$Dm),
                                IIC_fpCVTDI, "vcvtr", ".s32.f64\t$Sd, $Dm",
                                [(set SPR:$Sd, (int_arm_vcvtr (f64 DPR:$Dm)))]>,
              Sched<[WriteFPCVT]> {
  let Inst{7} = 0; // Z bit
}
```
- EN: Defines TableGen record `VTOSIRD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VTOSIRD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1811-1817
```tablegen
def VTOSIRS : AVConv1InsS_Encode<0b11101, 0b11, 0b1101, 0b1010,
                                 (outs SPR:$Sd), (ins SPR:$Sm),
                                 IIC_fpCVTSI, "vcvtr", ".s32.f32\t$Sd, $Sm",
                                 [(set SPR:$Sd, (int_arm_vcvtr SPR:$Sm))]>,
              Sched<[WriteFPCVT]> {
  let Inst{7} = 0; // Z bit
}
```
- EN: Defines TableGen record `VTOSIRS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VTOSIRS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1819-1826
```tablegen
def VTOSIRH : AVConv1IsH_Encode<0b11101, 0b11, 0b1101, 0b1001,
                                 (outs SPR:$Sd), (ins SPR:$Sm),
                                 IIC_fpCVTHI, "vcvtr", ".s32.f16\t$Sd, $Sm",
                                 []>,
              Sched<[WriteFPCVT]> {
  let Inst{7} = 0; // Z bit
  let isUnpredicable = 1;
}
```
- EN: Defines TableGen record `VTOSIRH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VTOSIRH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1828-1834
```tablegen
def VTOUIRD : AVConv1IsD_Encode<0b11101, 0b11, 0b1100, 0b1011,
                                (outs SPR:$Sd), (ins DPR:$Dm),
                                IIC_fpCVTDI, "vcvtr", ".u32.f64\t$Sd, $Dm",
                                [(set SPR:$Sd, (int_arm_vcvtru(f64 DPR:$Dm)))]>,
              Sched<[WriteFPCVT]> {
  let Inst{7} = 0; // Z bit
}
```
- EN: Defines TableGen record `VTOUIRD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VTOUIRD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1836-1842
```tablegen
def VTOUIRS : AVConv1InsS_Encode<0b11101, 0b11, 0b1100, 0b1010,
                                 (outs SPR:$Sd), (ins SPR:$Sm),
                                 IIC_fpCVTSI, "vcvtr", ".u32.f32\t$Sd, $Sm",
                                 [(set SPR:$Sd, (int_arm_vcvtru SPR:$Sm))]>,
              Sched<[WriteFPCVT]> {
  let Inst{7} = 0; // Z bit
}
```
- EN: Defines TableGen record `VTOUIRS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VTOUIRS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1844-1852
```tablegen
def VTOUIRH : AVConv1IsH_Encode<0b11101, 0b11, 0b1100, 0b1001,
                                 (outs SPR:$Sd), (ins SPR:$Sm),
                                 IIC_fpCVTHI, "vcvtr", ".u32.f16\t$Sd, $Sm",
                                 []>,
              Sched<[WriteFPCVT]> {
  let Inst{7} = 0; // Z bit
  let isUnpredicable = 1;
}
} // mayRaiseFPException = 1, Uses = [FPSCR_RM]
```
- EN: Defines TableGen record `VTOUIRH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VTOUIRH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1854-1861
```tablegen
// v8.3-a Javascript Convert to Signed fixed-point
def VJCVT : AVConv1IsD_Encode<0b11101, 0b11, 0b1001, 0b1011,
                                (outs SPR:$Sd), (ins DPR:$Dm),
                                IIC_fpCVTDI, "vjcvt", ".s32.f64\t$Sd, $Dm",
                                []>,
            Requires<[HasFPARMv8, HasV8_3a]> {
  let Inst{7} = 1; // Z bit
}
```
- EN: Defines TableGen record `VJCVT` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VJCVT`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1863-1868
```tablegen
// Convert between floating-point and fixed-point
// Data type for fixed-point naming convention:
//   S16 (U=0, sx=0) -> SH
//   U16 (U=1, sx=0) -> UH
//   S32 (U=0, sx=1) -> SL
//   U32 (U=1, sx=1) -> UL
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 1870-1870
```tablegen
let Constraints = "$a = $dst", mayRaiseFPException = 1 in {
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1872-1872
```tablegen
// FP to Fixed-Point:
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 1874-1882
```tablegen
// Single Precision register
class AVConv1XInsS_Encode<bits<5> op1, bits<2> op2, bits<4> op3, bits<4> op4,
                          bit op5, dag oops, dag iops, InstrItinClass itin,
                          string opc, string asm, list<dag> pattern>
  : AVConv1XI<op1, op2, op3, op4, op5, oops, iops, itin, opc, asm, pattern> {
  bits<5> dst;
  // if dp_operation then UInt(D:Vd) else UInt(Vd:D);
  let Inst{22} = dst{0};
  let Inst{15-12} = dst{4-1};
```
- EN: Declares reusable TableGen class `AVConv1XInsS_Encode` for `ARMInstrVFP`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrVFP` 声明可复用的 TableGen 类 `AVConv1XInsS_Encode`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1884-1885
```tablegen
  let hasSideEffects = 0;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1887-1895
```tablegen
// Double Precision register
class AVConv1XInsD_Encode<bits<5> op1, bits<2> op2, bits<4> op3, bits<4> op4,
                          bit op5, dag oops, dag iops, InstrItinClass itin,
                          string opc, string asm, list<dag> pattern>
  : AVConv1XI<op1, op2, op3, op4, op5, oops, iops, itin, opc, asm, pattern> {
  bits<5> dst;
  // if dp_operation then UInt(D:Vd) else UInt(Vd:D);
  let Inst{22} = dst{4};
  let Inst{15-12} = dst{3-0};
```
- EN: Declares reusable TableGen class `AVConv1XInsD_Encode` for `ARMInstrVFP`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrVFP` 声明可复用的 TableGen 类 `AVConv1XInsD_Encode`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1897-1899
```tablegen
  let hasSideEffects = 0;
  let Predicates = [HasVFP2, HasDPVFP];
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1901-1901
```tablegen
let isUnpredicable = 1 in {
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1903-1907
```tablegen
def VTOSHH : AVConv1XInsS_Encode<0b11101, 0b11, 0b1110, 0b1001, 0,
                       (outs SPR:$dst), (ins SPR:$a, fbits16:$fbits),
                 IIC_fpCVTHI, "vcvt", ".s16.f16\t$dst, $a, $fbits", []>,
             Requires<[HasFullFP16]>,
             Sched<[WriteFPCVT]>;
```
- EN: Defines TableGen record `VTOSHH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VTOSHH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1909-1913
```tablegen
def VTOUHH : AVConv1XInsS_Encode<0b11101, 0b11, 0b1111, 0b1001, 0,
                       (outs SPR:$dst), (ins SPR:$a, fbits16:$fbits),
                 IIC_fpCVTHI, "vcvt", ".u16.f16\t$dst, $a, $fbits", []>,
             Requires<[HasFullFP16]>,
             Sched<[WriteFPCVT]>;
```
- EN: Defines TableGen record `VTOUHH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VTOUHH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1915-1919
```tablegen
def VTOSLH : AVConv1XInsS_Encode<0b11101, 0b11, 0b1110, 0b1001, 1,
                       (outs SPR:$dst), (ins SPR:$a, fbits32:$fbits),
                 IIC_fpCVTHI, "vcvt", ".s32.f16\t$dst, $a, $fbits", []>,
             Requires<[HasFullFP16]>,
             Sched<[WriteFPCVT]>;
```
- EN: Defines TableGen record `VTOSLH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VTOSLH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1921-1925
```tablegen
def VTOULH : AVConv1XInsS_Encode<0b11101, 0b11, 0b1111, 0b1001, 1,
                       (outs SPR:$dst), (ins SPR:$a, fbits32:$fbits),
                 IIC_fpCVTHI, "vcvt", ".u32.f16\t$dst, $a, $fbits", []>,
             Requires<[HasFullFP16]>,
             Sched<[WriteFPCVT]>;
```
- EN: Defines TableGen record `VTOULH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VTOULH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1927-1927
```tablegen
} // End of 'let isUnpredicable = 1 in'
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1929-1936
```tablegen
def VTOSHS : AVConv1XInsS_Encode<0b11101, 0b11, 0b1110, 0b1010, 0,
                       (outs SPR:$dst), (ins SPR:$a, fbits16:$fbits),
                 IIC_fpCVTSI, "vcvt", ".s16.f32\t$dst, $a, $fbits", []>,
             Sched<[WriteFPCVT]> {
  // Some single precision VFP instructions may be executed on both NEON and
  // VFP pipelines on A8.
  let D = VFPNeonA8Domain;
}
```
- EN: Defines TableGen record `VTOSHS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VTOSHS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1938-1945
```tablegen
def VTOUHS : AVConv1XInsS_Encode<0b11101, 0b11, 0b1111, 0b1010, 0,
                       (outs SPR:$dst), (ins SPR:$a, fbits16:$fbits),
                 IIC_fpCVTSI, "vcvt", ".u16.f32\t$dst, $a, $fbits", []>,
             Sched<[WriteFPCVT]> {
  // Some single precision VFP instructions may be executed on both NEON and
  // VFP pipelines on A8.
  let D = VFPNeonA8Domain;
}
```
- EN: Defines TableGen record `VTOUHS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VTOUHS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1947-1954
```tablegen
def VTOSLS : AVConv1XInsS_Encode<0b11101, 0b11, 0b1110, 0b1010, 1,
                       (outs SPR:$dst), (ins SPR:$a, fbits32:$fbits),
                 IIC_fpCVTSI, "vcvt", ".s32.f32\t$dst, $a, $fbits", []>,
             Sched<[WriteFPCVT]> {
  // Some single precision VFP instructions may be executed on both NEON and
  // VFP pipelines on A8.
  let D = VFPNeonA8Domain;
}
```
- EN: Defines TableGen record `VTOSLS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VTOSLS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1956-1963
```tablegen
def VTOULS : AVConv1XInsS_Encode<0b11101, 0b11, 0b1111, 0b1010, 1,
                       (outs SPR:$dst), (ins SPR:$a, fbits32:$fbits),
                 IIC_fpCVTSI, "vcvt", ".u32.f32\t$dst, $a, $fbits", []>,
             Sched<[WriteFPCVT]> {
  // Some single precision VFP instructions may be executed on both NEON and
  // VFP pipelines on A8.
  let D = VFPNeonA8Domain;
}
```
- EN: Defines TableGen record `VTOULS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VTOULS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1965-1968
```tablegen
def VTOSHD : AVConv1XInsD_Encode<0b11101, 0b11, 0b1110, 0b1011, 0,
                       (outs DPR:$dst), (ins DPR:$a, fbits16:$fbits),
                 IIC_fpCVTDI, "vcvt", ".s16.f64\t$dst, $a, $fbits", []>,
             Sched<[WriteFPCVT]>;
```
- EN: Defines TableGen record `VTOSHD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VTOSHD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1970-1973
```tablegen
def VTOUHD : AVConv1XInsD_Encode<0b11101, 0b11, 0b1111, 0b1011, 0,
                       (outs DPR:$dst), (ins DPR:$a, fbits16:$fbits),
                 IIC_fpCVTDI, "vcvt", ".u16.f64\t$dst, $a, $fbits", []>,
             Sched<[WriteFPCVT]>;
```
- EN: Defines TableGen record `VTOUHD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VTOUHD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1975-1978
```tablegen
def VTOSLD : AVConv1XInsD_Encode<0b11101, 0b11, 0b1110, 0b1011, 1,
                       (outs DPR:$dst), (ins DPR:$a, fbits32:$fbits),
                 IIC_fpCVTDI, "vcvt", ".s32.f64\t$dst, $a, $fbits", []>,
             Sched<[WriteFPCVT]>;
```
- EN: Defines TableGen record `VTOSLD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VTOSLD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1980-1983
```tablegen
def VTOULD : AVConv1XInsD_Encode<0b11101, 0b11, 0b1111, 0b1011, 1,
                       (outs DPR:$dst), (ins DPR:$a, fbits32:$fbits),
                 IIC_fpCVTDI, "vcvt", ".u32.f64\t$dst, $a, $fbits", []>,
             Sched<[WriteFPCVT]>;
```
- EN: Defines TableGen record `VTOULD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VTOULD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1985-1985
```tablegen
// Fixed-Point to FP:
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 1987-1987
```tablegen
let isUnpredicable = 1 in {
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1989-1993
```tablegen
def VSHTOH : AVConv1XInsS_Encode<0b11101, 0b11, 0b1010, 0b1001, 0,
                       (outs SPR:$dst), (ins SPR:$a, fbits16:$fbits),
                 IIC_fpCVTIH, "vcvt", ".f16.s16\t$dst, $a, $fbits", []>,
             Requires<[HasFullFP16]>,
             Sched<[WriteFPCVT]>;
```
- EN: Defines TableGen record `VSHTOH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VSHTOH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1995-1999
```tablegen
def VUHTOH : AVConv1XInsS_Encode<0b11101, 0b11, 0b1011, 0b1001, 0,
                       (outs SPR:$dst), (ins SPR:$a, fbits16:$fbits),
                 IIC_fpCVTIH, "vcvt", ".f16.u16\t$dst, $a, $fbits", []>,
             Requires<[HasFullFP16]>,
             Sched<[WriteFPCVT]>;
```
- EN: Defines TableGen record `VUHTOH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VUHTOH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2001-2005
```tablegen
def VSLTOH : AVConv1XInsS_Encode<0b11101, 0b11, 0b1010, 0b1001, 1,
                       (outs SPR:$dst), (ins SPR:$a, fbits32:$fbits),
                 IIC_fpCVTIH, "vcvt", ".f16.s32\t$dst, $a, $fbits", []>,
             Requires<[HasFullFP16]>,
             Sched<[WriteFPCVT]>;
```
- EN: Defines TableGen record `VSLTOH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VSLTOH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2007-2011
```tablegen
def VULTOH : AVConv1XInsS_Encode<0b11101, 0b11, 0b1011, 0b1001, 1,
                       (outs SPR:$dst), (ins SPR:$a, fbits32:$fbits),
                 IIC_fpCVTIH, "vcvt", ".f16.u32\t$dst, $a, $fbits", []>,
             Requires<[HasFullFP16]>,
             Sched<[WriteFPCVT]>;
```
- EN: Defines TableGen record `VULTOH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VULTOH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2013-2013
```tablegen
} // End of 'let isUnpredicable = 1 in'
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2015-2022
```tablegen
def VSHTOS : AVConv1XInsS_Encode<0b11101, 0b11, 0b1010, 0b1010, 0,
                       (outs SPR:$dst), (ins SPR:$a, fbits16:$fbits),
                 IIC_fpCVTIS, "vcvt", ".f32.s16\t$dst, $a, $fbits", []>,
             Sched<[WriteFPCVT]> {
  // Some single precision VFP instructions may be executed on both NEON and
  // VFP pipelines on A8.
  let D = VFPNeonA8Domain;
}
```
- EN: Defines TableGen record `VSHTOS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VSHTOS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2024-2031
```tablegen
def VUHTOS : AVConv1XInsS_Encode<0b11101, 0b11, 0b1011, 0b1010, 0,
                       (outs SPR:$dst), (ins SPR:$a, fbits16:$fbits),
                 IIC_fpCVTIS, "vcvt", ".f32.u16\t$dst, $a, $fbits", []>,
             Sched<[WriteFPCVT]> {
  // Some single precision VFP instructions may be executed on both NEON and
  // VFP pipelines on A8.
  let D = VFPNeonA8Domain;
}
```
- EN: Defines TableGen record `VUHTOS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VUHTOS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2033-2040
```tablegen
def VSLTOS : AVConv1XInsS_Encode<0b11101, 0b11, 0b1010, 0b1010, 1,
                       (outs SPR:$dst), (ins SPR:$a, fbits32:$fbits),
                 IIC_fpCVTIS, "vcvt", ".f32.s32\t$dst, $a, $fbits", []>,
             Sched<[WriteFPCVT]> {
  // Some single precision VFP instructions may be executed on both NEON and
  // VFP pipelines on A8.
  let D = VFPNeonA8Domain;
}
```
- EN: Defines TableGen record `VSLTOS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VSLTOS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2042-2049
```tablegen
def VULTOS : AVConv1XInsS_Encode<0b11101, 0b11, 0b1011, 0b1010, 1,
                       (outs SPR:$dst), (ins SPR:$a, fbits32:$fbits),
                 IIC_fpCVTIS, "vcvt", ".f32.u32\t$dst, $a, $fbits", []>,
             Sched<[WriteFPCVT]> {
  // Some single precision VFP instructions may be executed on both NEON and
  // VFP pipelines on A8.
  let D = VFPNeonA8Domain;
}
```
- EN: Defines TableGen record `VULTOS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VULTOS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2051-2054
```tablegen
def VSHTOD : AVConv1XInsD_Encode<0b11101, 0b11, 0b1010, 0b1011, 0,
                       (outs DPR:$dst), (ins DPR:$a, fbits16:$fbits),
                 IIC_fpCVTID, "vcvt", ".f64.s16\t$dst, $a, $fbits", []>,
             Sched<[WriteFPCVT]>;
```
- EN: Defines TableGen record `VSHTOD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VSHTOD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2056-2059
```tablegen
def VUHTOD : AVConv1XInsD_Encode<0b11101, 0b11, 0b1011, 0b1011, 0,
                       (outs DPR:$dst), (ins DPR:$a, fbits16:$fbits),
                 IIC_fpCVTID, "vcvt", ".f64.u16\t$dst, $a, $fbits", []>,
             Sched<[WriteFPCVT]>;
```
- EN: Defines TableGen record `VUHTOD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VUHTOD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2061-2064
```tablegen
def VSLTOD : AVConv1XInsD_Encode<0b11101, 0b11, 0b1010, 0b1011, 1,
                       (outs DPR:$dst), (ins DPR:$a, fbits32:$fbits),
                 IIC_fpCVTID, "vcvt", ".f64.s32\t$dst, $a, $fbits", []>,
             Sched<[WriteFPCVT]>;
```
- EN: Defines TableGen record `VSLTOD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VSLTOD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2066-2069
```tablegen
def VULTOD : AVConv1XInsD_Encode<0b11101, 0b11, 0b1011, 0b1011, 1,
                       (outs DPR:$dst), (ins DPR:$a, fbits32:$fbits),
                 IIC_fpCVTID, "vcvt", ".f64.u32\t$dst, $a, $fbits", []>,
             Sched<[WriteFPCVT]>;
```
- EN: Defines TableGen record `VULTOD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VULTOD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2071-2071
```tablegen
} // End of 'let Constraints = "$a = $dst", mayRaiseFPException = 1 in'
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2073-2083
```tablegen
// BFloat16  - Single precision, unary, predicated
let mayRaiseFPException = 1, Uses = [FPSCR_RM] in
class BF16_VCVT<string opc, bits<2> op7_6>
   : VFPAI<(outs SPR:$Sd), (ins SPR:$dst, SPR:$Sm),
           VFPUnaryFrm, NoItinerary,
           opc, ".bf16.f32\t$Sd, $Sm", "", []>,
      RegConstraint<"$dst = $Sd">,
      Requires<[HasBF16]>,
     Sched<[]> {
  bits<5> Sd;
  bits<5> Sm;
```
- EN: Declares reusable TableGen class `BF16_VCVT` for `ARMInstrVFP`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrVFP` 声明可复用的 TableGen 类 `BF16_VCVT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2085-2089
```tablegen
  // Encode instruction operands.
  let Inst{3-0}   = Sm{4-1};
  let Inst{5}     = Sm{0};
  let Inst{15-12} = Sd{4-1};
  let Inst{22}    = Sd{0};
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2091-2096
```tablegen
  let Inst{27-23} = 0b11101; // opcode1
  let Inst{21-20} = 0b11;    // opcode2
  let Inst{19-16} = 0b0011;  // opcode3
  let Inst{11-8}  = 0b1001;
  let Inst{7-6}   = op7_6;
  let Inst{4}     = 0;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2098-2100
```tablegen
  let DecoderNamespace = "VFPV8";
  let hasSideEffects = 0;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2102-2103
```tablegen
def BF16_VCVTB : BF16_VCVT<"vcvtb", 0b01>;
def BF16_VCVTT : BF16_VCVT<"vcvtt", 0b11>;
```
- EN: Defines TableGen record `BF16_VCVTB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BF16_VCVTB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2105-2107
```tablegen
//===----------------------------------------------------------------------===//
// FP Multiply-Accumulate Operations.
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 2109-2117
```tablegen
let mayRaiseFPException = 1, Uses = [FPSCR_RM] in
def VMLAD : ADbI<0b11100, 0b00, 0, 0,
                 (outs DPR:$Dd), (ins DPR:$Ddin, DPR:$Dn, DPR:$Dm),
                 IIC_fpMAC64, "vmla", ".f64\t$Dd, $Dn, $Dm",
                 [(set DPR:$Dd, (fadd_mlx (fmul_su DPR:$Dn, DPR:$Dm),
                                          (f64 DPR:$Ddin)))]>,
              RegConstraint<"$Ddin = $Dd">,
              Requires<[HasVFP2,HasDPVFP,UseFPVMLx]>,
              Sched<[WriteFPMAC64, ReadFPMAC, ReadFPMUL, ReadFPMUL]>;
```
- EN: Defines TableGen record `VMLAD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VMLAD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2119-2131
```tablegen
let mayRaiseFPException = 1, Uses = [FPSCR_RM] in
def VMLAS : ASbIn<0b11100, 0b00, 0, 0,
                  (outs SPR:$Sd), (ins SPR:$Sdin, SPR:$Sn, SPR:$Sm),
                  IIC_fpMAC32, "vmla", ".f32\t$Sd, $Sn, $Sm",
                  [(set SPR:$Sd, (fadd_mlx (fmul_su SPR:$Sn, SPR:$Sm),
                                           SPR:$Sdin))]>,
              RegConstraint<"$Sdin = $Sd">,
              Requires<[HasVFP2,DontUseNEONForFP,UseFPVMLx]>,
              Sched<[WriteFPMAC32, ReadFPMAC, ReadFPMUL, ReadFPMUL]> {
  // Some single precision VFP instructions may be executed on both NEON and
  // VFP pipelines on A8.
  let D = VFPNeonA8Domain;
}
```
- EN: Defines TableGen record `VMLAS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VMLAS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2133-2140
```tablegen
let mayRaiseFPException = 1, Uses = [FPSCR_RM] in
def VMLAH : AHbI<0b11100, 0b00, 0, 0,
                  (outs HPR:$Sd), (ins HPR:$Sdin, HPR:$Sn, HPR:$Sm),
                  IIC_fpMAC16, "vmla", ".f16\t$Sd, $Sn, $Sm",
                  [(set (f16 HPR:$Sd), (fadd_mlx (fmul_su (f16 HPR:$Sn), (f16 HPR:$Sm)),
                                           (f16 HPR:$Sdin)))]>,
              RegConstraint<"$Sdin = $Sd">,
              Requires<[HasFullFP16,UseFPVMLx]>;
```
- EN: Defines TableGen record `VMLAH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VMLAH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2142-2150
```tablegen
def : Pat<(fadd_mlx DPR:$dstin, (fmul_su DPR:$a, (f64 DPR:$b))),
          (VMLAD DPR:$dstin, DPR:$a, DPR:$b)>,
          Requires<[HasVFP2,HasDPVFP,UseFPVMLx]>;
def : Pat<(fadd_mlx SPR:$dstin, (fmul_su SPR:$a, SPR:$b)),
          (VMLAS SPR:$dstin, SPR:$a, SPR:$b)>,
          Requires<[HasVFP2,DontUseNEONForFP, UseFPVMLx]>;
def : Pat<(fadd_mlx HPR:$dstin, (fmul_su (f16 HPR:$a), HPR:$b)),
          (VMLAH HPR:$dstin, (f16 HPR:$a), HPR:$b)>,
          Requires<[HasFullFP16,DontUseNEONForFP, UseFPVMLx]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2153-2161
```tablegen
let mayRaiseFPException = 1, Uses = [FPSCR_RM] in
def VMLSD : ADbI<0b11100, 0b00, 1, 0,
                 (outs DPR:$Dd), (ins DPR:$Ddin, DPR:$Dn, DPR:$Dm),
                 IIC_fpMAC64, "vmls", ".f64\t$Dd, $Dn, $Dm",
                 [(set DPR:$Dd, (fadd_mlx (fneg (fmul_su DPR:$Dn,DPR:$Dm)),
                                          (f64 DPR:$Ddin)))]>,
              RegConstraint<"$Ddin = $Dd">,
              Requires<[HasVFP2,HasDPVFP,UseFPVMLx]>,
              Sched<[WriteFPMAC64, ReadFPMAC, ReadFPMUL, ReadFPMUL]>;
```
- EN: Defines TableGen record `VMLSD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VMLSD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2163-2175
```tablegen
let mayRaiseFPException = 1, Uses = [FPSCR_RM] in
def VMLSS : ASbIn<0b11100, 0b00, 1, 0,
                  (outs SPR:$Sd), (ins SPR:$Sdin, SPR:$Sn, SPR:$Sm),
                  IIC_fpMAC32, "vmls", ".f32\t$Sd, $Sn, $Sm",
                  [(set SPR:$Sd, (fadd_mlx (fneg (fmul_su SPR:$Sn, SPR:$Sm)),
                                           SPR:$Sdin))]>,
              RegConstraint<"$Sdin = $Sd">,
              Requires<[HasVFP2,DontUseNEONForFP,UseFPVMLx]>,
              Sched<[WriteFPMAC32, ReadFPMAC, ReadFPMUL, ReadFPMUL]> {
  // Some single precision VFP instructions may be executed on both NEON and
  // VFP pipelines on A8.
  let D = VFPNeonA8Domain;
}
```
- EN: Defines TableGen record `VMLSS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VMLSS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2177-2184
```tablegen
let mayRaiseFPException = 1, Uses = [FPSCR_RM] in
def VMLSH : AHbI<0b11100, 0b00, 1, 0,
                  (outs HPR:$Sd), (ins HPR:$Sdin, HPR:$Sn, HPR:$Sm),
                  IIC_fpMAC16, "vmls", ".f16\t$Sd, $Sn, $Sm",
                  [(set (f16 HPR:$Sd), (fadd_mlx (fneg (fmul_su (f16 HPR:$Sn), (f16 HPR:$Sm))),
                                           (f16 HPR:$Sdin)))]>,
              RegConstraint<"$Sdin = $Sd">,
              Requires<[HasFullFP16,UseFPVMLx]>;
```
- EN: Defines TableGen record `VMLSH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VMLSH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2186-2194
```tablegen
def : Pat<(fsub_mlx DPR:$dstin, (fmul_su DPR:$a, (f64 DPR:$b))),
          (VMLSD DPR:$dstin, DPR:$a, DPR:$b)>,
          Requires<[HasVFP2,HasDPVFP,UseFPVMLx]>;
def : Pat<(fsub_mlx SPR:$dstin, (fmul_su SPR:$a, SPR:$b)),
          (VMLSS SPR:$dstin, SPR:$a, SPR:$b)>,
          Requires<[HasVFP2,DontUseNEONForFP,UseFPVMLx]>;
def : Pat<(fsub_mlx HPR:$dstin, (fmul_su (f16 HPR:$a), HPR:$b)),
          (VMLSH HPR:$dstin, (f16 HPR:$a), HPR:$b)>,
          Requires<[HasFullFP16,DontUseNEONForFP,UseFPVMLx]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2196-2204
```tablegen
let mayRaiseFPException = 1, Uses = [FPSCR_RM] in
def VNMLAD : ADbI<0b11100, 0b01, 1, 0,
                  (outs DPR:$Dd), (ins DPR:$Ddin, DPR:$Dn, DPR:$Dm),
                  IIC_fpMAC64, "vnmla", ".f64\t$Dd, $Dn, $Dm",
                  [(set DPR:$Dd,(fsub_mlx (fneg (fmul_su DPR:$Dn,DPR:$Dm)),
                                          (f64 DPR:$Ddin)))]>,
                RegConstraint<"$Ddin = $Dd">,
                Requires<[HasVFP2,HasDPVFP,UseFPVMLx]>,
                Sched<[WriteFPMAC64, ReadFPMAC, ReadFPMUL, ReadFPMUL]>;
```
- EN: Defines TableGen record `VNMLAD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VNMLAD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2206-2218
```tablegen
let mayRaiseFPException = 1, Uses = [FPSCR_RM] in
def VNMLAS : ASbI<0b11100, 0b01, 1, 0,
                  (outs SPR:$Sd), (ins SPR:$Sdin, SPR:$Sn, SPR:$Sm),
                  IIC_fpMAC32, "vnmla", ".f32\t$Sd, $Sn, $Sm",
                  [(set SPR:$Sd, (fsub_mlx (fneg (fmul_su SPR:$Sn, SPR:$Sm)),
                                           SPR:$Sdin))]>,
                RegConstraint<"$Sdin = $Sd">,
                Requires<[HasVFP2,DontUseNEONForFP,UseFPVMLx]>,
                Sched<[WriteFPMAC32, ReadFPMAC, ReadFPMUL, ReadFPMUL]> {
  // Some single precision VFP instructions may be executed on both NEON and
  // VFP pipelines on A8.
  let D = VFPNeonA8Domain;
}
```
- EN: Defines TableGen record `VNMLAS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VNMLAS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2220-2227
```tablegen
let mayRaiseFPException = 1, Uses = [FPSCR_RM] in
def VNMLAH : AHbI<0b11100, 0b01, 1, 0,
                  (outs HPR:$Sd), (ins HPR:$Sdin, HPR:$Sn, HPR:$Sm),
                  IIC_fpMAC16, "vnmla", ".f16\t$Sd, $Sn, $Sm",
                  [(set (f16 HPR:$Sd), (fsub_mlx (fneg (fmul_su (f16 HPR:$Sn), (f16 HPR:$Sm))),
                                           (f16 HPR:$Sdin)))]>,
                RegConstraint<"$Sdin = $Sd">,
                Requires<[HasFullFP16,UseFPVMLx]>;
```
- EN: Defines TableGen record `VNMLAH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VNMLAH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2229-2238
```tablegen
// (-(a * b) - dst) -> -(dst + (a * b))
def : Pat<(fsub_mlx (fneg (fmul_su DPR:$a, (f64 DPR:$b))), DPR:$dstin),
          (VNMLAD DPR:$dstin, DPR:$a, DPR:$b)>,
          Requires<[HasVFP2,HasDPVFP,UseFPVMLx]>;
def : Pat<(fsub_mlx (fneg (fmul_su SPR:$a, SPR:$b)), SPR:$dstin),
          (VNMLAS SPR:$dstin, SPR:$a, SPR:$b)>,
          Requires<[HasVFP2,DontUseNEONForFP,UseFPVMLx]>;
def : Pat<(fsub_mlx (fneg (fmul_su (f16 HPR:$a), HPR:$b)), HPR:$dstin),
          (VNMLAH HPR:$dstin, (f16 HPR:$a), HPR:$b)>,
          Requires<[HasFullFP16,DontUseNEONForFP,UseFPVMLx]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2240-2249
```tablegen
// (-dst - (a * b)) -> -(dst + (a * b))
def : Pat<(fsub_mlx (fneg DPR:$dstin), (fmul_su DPR:$a, (f64 DPR:$b))),
          (VNMLAD DPR:$dstin, DPR:$a, DPR:$b)>,
          Requires<[HasVFP2,HasDPVFP,UseFPVMLx]>;
def : Pat<(fsub_mlx (fneg SPR:$dstin), (fmul_su SPR:$a, SPR:$b)),
          (VNMLAS SPR:$dstin, SPR:$a, SPR:$b)>,
          Requires<[HasVFP2,DontUseNEONForFP,UseFPVMLx]>;
def : Pat<(fsub_mlx (fneg HPR:$dstin), (fmul_su (f16 HPR:$a), HPR:$b)),
          (VNMLAH HPR:$dstin, (f16 HPR:$a), HPR:$b)>,
          Requires<[HasFullFP16,DontUseNEONForFP,UseFPVMLx]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2251-2259
```tablegen
let mayRaiseFPException = 1, Uses = [FPSCR_RM] in
def VNMLSD : ADbI<0b11100, 0b01, 0, 0,
                  (outs DPR:$Dd), (ins DPR:$Ddin, DPR:$Dn, DPR:$Dm),
                  IIC_fpMAC64, "vnmls", ".f64\t$Dd, $Dn, $Dm",
                  [(set DPR:$Dd, (fsub_mlx (fmul_su DPR:$Dn, DPR:$Dm),
                                           (f64 DPR:$Ddin)))]>,
               RegConstraint<"$Ddin = $Dd">,
               Requires<[HasVFP2,HasDPVFP,UseFPVMLx]>,
               Sched<[WriteFPMAC64, ReadFPMAC, ReadFPMUL, ReadFPMUL]>;
```
- EN: Defines TableGen record `VNMLSD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VNMLSD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2261-2272
```tablegen
let mayRaiseFPException = 1, Uses = [FPSCR_RM] in
def VNMLSS : ASbI<0b11100, 0b01, 0, 0,
                  (outs SPR:$Sd), (ins SPR:$Sdin, SPR:$Sn, SPR:$Sm),
                  IIC_fpMAC32, "vnmls", ".f32\t$Sd, $Sn, $Sm",
             [(set SPR:$Sd, (fsub_mlx (fmul_su SPR:$Sn, SPR:$Sm), SPR:$Sdin))]>,
                         RegConstraint<"$Sdin = $Sd">,
                Requires<[HasVFP2,DontUseNEONForFP,UseFPVMLx]>,
             Sched<[WriteFPMAC32, ReadFPMAC, ReadFPMUL, ReadFPMUL]> {
  // Some single precision VFP instructions may be executed on both NEON and
  // VFP pipelines on A8.
  let D = VFPNeonA8Domain;
}
```
- EN: Defines TableGen record `VNMLSS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VNMLSS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2274-2280
```tablegen
let mayRaiseFPException = 1, Uses = [FPSCR_RM] in
def VNMLSH : AHbI<0b11100, 0b01, 0, 0,
                  (outs HPR:$Sd), (ins HPR:$Sdin, HPR:$Sn, HPR:$Sm),
                  IIC_fpMAC16, "vnmls", ".f16\t$Sd, $Sn, $Sm",
             [(set (f16 HPR:$Sd), (fsub_mlx (fmul_su (f16 HPR:$Sn), (f16 HPR:$Sm)), (f16 HPR:$Sdin)))]>,
                         RegConstraint<"$Sdin = $Sd">,
                Requires<[HasFullFP16,UseFPVMLx]>;
```
- EN: Defines TableGen record `VNMLSH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VNMLSH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2282-2290
```tablegen
def : Pat<(fsub_mlx (fmul_su DPR:$a, (f64 DPR:$b)), DPR:$dstin),
          (VNMLSD DPR:$dstin, DPR:$a, DPR:$b)>,
          Requires<[HasVFP2,HasDPVFP,UseFPVMLx]>;
def : Pat<(fsub_mlx (fmul_su SPR:$a, SPR:$b), SPR:$dstin),
          (VNMLSS SPR:$dstin, SPR:$a, SPR:$b)>,
          Requires<[HasVFP2,DontUseNEONForFP,UseFPVMLx]>;
def : Pat<(fsub_mlx (fmul_su (f16 HPR:$a), HPR:$b), HPR:$dstin),
          (VNMLSH HPR:$dstin, (f16 HPR:$a), HPR:$b)>,
          Requires<[HasFullFP16,DontUseNEONForFP,UseFPVMLx]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2292-2303
```tablegen
//===----------------------------------------------------------------------===//
// Fused FP Multiply-Accumulate Operations.
//
let mayRaiseFPException = 1, Uses = [FPSCR_RM] in
def VFMAD : ADbI<0b11101, 0b10, 0, 0,
                 (outs DPR:$Dd), (ins DPR:$Ddin, DPR:$Dn, DPR:$Dm),
                 IIC_fpFMAC64, "vfma", ".f64\t$Dd, $Dn, $Dm",
                 [(set DPR:$Dd, (fadd_mlx (fmul_su DPR:$Dn, DPR:$Dm),
                                          (f64 DPR:$Ddin)))]>,
              RegConstraint<"$Ddin = $Dd">,
              Requires<[HasVFP4,HasDPVFP,UseFusedMAC]>,
            Sched<[WriteFPMAC64, ReadFPMAC, ReadFPMUL, ReadFPMUL]>;
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 2305-2316
```tablegen
let mayRaiseFPException = 1, Uses = [FPSCR_RM] in
def VFMAS : ASbIn<0b11101, 0b10, 0, 0,
                  (outs SPR:$Sd), (ins SPR:$Sdin, SPR:$Sn, SPR:$Sm),
                  IIC_fpFMAC32, "vfma", ".f32\t$Sd, $Sn, $Sm",
                  [(set SPR:$Sd, (fadd_mlx (fmul_su SPR:$Sn, SPR:$Sm),
                                           SPR:$Sdin))]>,
              RegConstraint<"$Sdin = $Sd">,
              Requires<[HasVFP4,DontUseNEONForFP,UseFusedMAC]>,
            Sched<[WriteFPMAC32, ReadFPMAC, ReadFPMUL, ReadFPMUL]> {
  // Some single precision VFP instructions may be executed on both NEON and
  // VFP pipelines.
}
```
- EN: Defines TableGen record `VFMAS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VFMAS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2318-2326
```tablegen
let mayRaiseFPException = 1, Uses = [FPSCR_RM] in
def VFMAH : AHbI<0b11101, 0b10, 0, 0,
                  (outs HPR:$Sd), (ins HPR:$Sdin, HPR:$Sn, HPR:$Sm),
                  IIC_fpFMAC16, "vfma", ".f16\t$Sd, $Sn, $Sm",
                  [(set (f16 HPR:$Sd), (fadd_mlx (fmul_su (f16 HPR:$Sn), (f16 HPR:$Sm)),
                                           (f16 HPR:$Sdin)))]>,
              RegConstraint<"$Sdin = $Sd">,
              Requires<[HasFullFP16,UseFusedMAC]>,
            Sched<[WriteFPMAC32, ReadFPMAC, ReadFPMUL, ReadFPMUL]>;
```
- EN: Defines TableGen record `VFMAH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VFMAH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2328-2336
```tablegen
def : Pat<(fadd_mlx DPR:$dstin, (fmul_su DPR:$a, (f64 DPR:$b))),
          (VFMAD DPR:$dstin, DPR:$a, DPR:$b)>,
          Requires<[HasVFP4,HasDPVFP,UseFusedMAC]>;
def : Pat<(fadd_mlx SPR:$dstin, (fmul_su SPR:$a, SPR:$b)),
          (VFMAS SPR:$dstin, SPR:$a, SPR:$b)>,
          Requires<[HasVFP4,DontUseNEONForFP,UseFusedMAC]>;
def : Pat<(fadd_mlx HPR:$dstin, (fmul_su (f16 HPR:$a), HPR:$b)),
          (VFMAH HPR:$dstin, (f16 HPR:$a), HPR:$b)>,
          Requires<[HasFullFP16,DontUseNEONForFP,UseFusedMAC]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2338-2348
```tablegen
// Match @llvm.fma.* intrinsics
// (fma x, y, z) -> (vfms z, x, y)
def : Pat<(f64 (any_fma DPR:$Dn, DPR:$Dm, DPR:$Ddin)),
          (VFMAD DPR:$Ddin, DPR:$Dn, DPR:$Dm)>,
      Requires<[HasVFP4,HasDPVFP]>;
def : Pat<(f32 (any_fma SPR:$Sn, SPR:$Sm, SPR:$Sdin)),
          (VFMAS SPR:$Sdin, SPR:$Sn, SPR:$Sm)>,
      Requires<[HasVFP4]>;
def : Pat<(f16 (any_fma HPR:$Sn, HPR:$Sm, (f16 HPR:$Sdin))),
          (VFMAH (f16 HPR:$Sdin), (f16 HPR:$Sn), (f16 HPR:$Sm))>,
      Requires<[HasFullFP16]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2350-2358
```tablegen
let mayRaiseFPException = 1, Uses = [FPSCR_RM] in
def VFMSD : ADbI<0b11101, 0b10, 1, 0,
                 (outs DPR:$Dd), (ins DPR:$Ddin, DPR:$Dn, DPR:$Dm),
                 IIC_fpFMAC64, "vfms", ".f64\t$Dd, $Dn, $Dm",
                 [(set DPR:$Dd, (fadd_mlx (fneg (fmul_su DPR:$Dn,DPR:$Dm)),
                                          (f64 DPR:$Ddin)))]>,
              RegConstraint<"$Ddin = $Dd">,
              Requires<[HasVFP4,HasDPVFP,UseFusedMAC]>,
              Sched<[WriteFPMAC64, ReadFPMAC, ReadFPMUL, ReadFPMUL]>;
```
- EN: Defines TableGen record `VFMSD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VFMSD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2360-2371
```tablegen
let mayRaiseFPException = 1, Uses = [FPSCR_RM] in
def VFMSS : ASbIn<0b11101, 0b10, 1, 0,
                  (outs SPR:$Sd), (ins SPR:$Sdin, SPR:$Sn, SPR:$Sm),
                  IIC_fpFMAC32, "vfms", ".f32\t$Sd, $Sn, $Sm",
                  [(set SPR:$Sd, (fadd_mlx (fneg (fmul_su SPR:$Sn, SPR:$Sm)),
                                           SPR:$Sdin))]>,
              RegConstraint<"$Sdin = $Sd">,
              Requires<[HasVFP4,DontUseNEONForFP,UseFusedMAC]>,
              Sched<[WriteFPMAC32, ReadFPMAC, ReadFPMUL, ReadFPMUL]> {
  // Some single precision VFP instructions may be executed on both NEON and
  // VFP pipelines.
}
```
- EN: Defines TableGen record `VFMSS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VFMSS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2373-2381
```tablegen
let mayRaiseFPException = 1, Uses = [FPSCR_RM] in
def VFMSH : AHbI<0b11101, 0b10, 1, 0,
                  (outs HPR:$Sd), (ins HPR:$Sdin, HPR:$Sn, HPR:$Sm),
                  IIC_fpFMAC16, "vfms", ".f16\t$Sd, $Sn, $Sm",
                  [(set (f16 HPR:$Sd), (fadd_mlx (fneg (fmul_su (f16 HPR:$Sn), (f16 HPR:$Sm))),
                                           (f16 HPR:$Sdin)))]>,
              RegConstraint<"$Sdin = $Sd">,
              Requires<[HasFullFP16,UseFusedMAC]>,
              Sched<[WriteFPMAC32, ReadFPMAC, ReadFPMUL, ReadFPMUL]>;
```
- EN: Defines TableGen record `VFMSH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VFMSH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2383-2391
```tablegen
def : Pat<(fsub_mlx DPR:$dstin, (fmul_su DPR:$a, (f64 DPR:$b))),
          (VFMSD DPR:$dstin, DPR:$a, DPR:$b)>,
          Requires<[HasVFP4,HasDPVFP,UseFusedMAC]>;
def : Pat<(fsub_mlx SPR:$dstin, (fmul_su SPR:$a, SPR:$b)),
          (VFMSS SPR:$dstin, SPR:$a, SPR:$b)>,
          Requires<[HasVFP4,DontUseNEONForFP,UseFusedMAC]>;
def : Pat<(fsub_mlx HPR:$dstin, (fmul_su (f16 HPR:$a), HPR:$b)),
          (VFMSH HPR:$dstin, (f16 HPR:$a), HPR:$b)>,
          Requires<[HasFullFP16,DontUseNEONForFP,UseFusedMAC]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2393-2403
```tablegen
// Match @llvm.fma.* intrinsics
// (fma (fneg x), y, z) -> (vfms z, x, y)
def : Pat<(f64 (any_fma (fneg DPR:$Dn), DPR:$Dm, DPR:$Ddin)),
          (VFMSD DPR:$Ddin, DPR:$Dn, DPR:$Dm)>,
      Requires<[HasVFP4,HasDPVFP]>;
def : Pat<(f32 (any_fma (fneg SPR:$Sn), SPR:$Sm, SPR:$Sdin)),
          (VFMSS SPR:$Sdin, SPR:$Sn, SPR:$Sm)>,
      Requires<[HasVFP4]>;
def : Pat<(f16 (any_fma (fneg (f16 HPR:$Sn)), (f16 HPR:$Sm), (f16 HPR:$Sdin))),
          (VFMSH (f16 HPR:$Sdin), (f16 HPR:$Sn), (f16 HPR:$Sm))>,
      Requires<[HasFullFP16]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2405-2413
```tablegen
let mayRaiseFPException = 1, Uses = [FPSCR_RM] in
def VFNMAD : ADbI<0b11101, 0b01, 1, 0,
                  (outs DPR:$Dd), (ins DPR:$Ddin, DPR:$Dn, DPR:$Dm),
                  IIC_fpFMAC64, "vfnma", ".f64\t$Dd, $Dn, $Dm",
                  [(set DPR:$Dd,(fsub_mlx (fneg (fmul_su DPR:$Dn,DPR:$Dm)),
                                          (f64 DPR:$Ddin)))]>,
                RegConstraint<"$Ddin = $Dd">,
                Requires<[HasVFP4,HasDPVFP,UseFusedMAC]>,
                Sched<[WriteFPMAC64, ReadFPMAC, ReadFPMUL, ReadFPMUL]>;
```
- EN: Defines TableGen record `VFNMAD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VFNMAD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2415-2426
```tablegen
let mayRaiseFPException = 1, Uses = [FPSCR_RM] in
def VFNMAS : ASbI<0b11101, 0b01, 1, 0,
                  (outs SPR:$Sd), (ins SPR:$Sdin, SPR:$Sn, SPR:$Sm),
                  IIC_fpFMAC32, "vfnma", ".f32\t$Sd, $Sn, $Sm",
                  [(set SPR:$Sd, (fsub_mlx (fneg (fmul_su SPR:$Sn, SPR:$Sm)),
                                           SPR:$Sdin))]>,
                RegConstraint<"$Sdin = $Sd">,
                Requires<[HasVFP4,DontUseNEONForFP,UseFusedMAC]>,
                Sched<[WriteFPMAC32, ReadFPMAC, ReadFPMUL, ReadFPMUL]> {
  // Some single precision VFP instructions may be executed on both NEON and
  // VFP pipelines.
}
```
- EN: Defines TableGen record `VFNMAS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VFNMAS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2428-2436
```tablegen
let mayRaiseFPException = 1, Uses = [FPSCR_RM] in
def VFNMAH : AHbI<0b11101, 0b01, 1, 0,
                  (outs HPR:$Sd), (ins HPR:$Sdin, HPR:$Sn, HPR:$Sm),
                  IIC_fpFMAC16, "vfnma", ".f16\t$Sd, $Sn, $Sm",
                  [(set (f16 HPR:$Sd), (fsub_mlx (fneg (fmul_su (f16 HPR:$Sn), (f16 HPR:$Sm))),
                                           (f16 HPR:$Sdin)))]>,
                RegConstraint<"$Sdin = $Sd">,
                Requires<[HasFullFP16,UseFusedMAC]>,
                Sched<[WriteFPMAC32, ReadFPMAC, ReadFPMUL, ReadFPMUL]>;
```
- EN: Defines TableGen record `VFNMAH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VFNMAH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2438-2443
```tablegen
def : Pat<(fsub_mlx (fneg (fmul_su DPR:$a, (f64 DPR:$b))), DPR:$dstin),
          (VFNMAD DPR:$dstin, DPR:$a, DPR:$b)>,
          Requires<[HasVFP4,HasDPVFP,UseFusedMAC]>;
def : Pat<(fsub_mlx (fneg (fmul_su SPR:$a, SPR:$b)), SPR:$dstin),
          (VFNMAS SPR:$dstin, SPR:$a, SPR:$b)>,
          Requires<[HasVFP4,DontUseNEONForFP,UseFusedMAC]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2445-2462
```tablegen
// Match @llvm.fma.* intrinsics
// (fneg (fma x, y, z)) -> (vfnma z, x, y)
def : Pat<(fneg (any_fma (f64 DPR:$Dn), (f64 DPR:$Dm), (f64 DPR:$Ddin))),
          (VFNMAD DPR:$Ddin, DPR:$Dn, DPR:$Dm)>,
      Requires<[HasVFP4,HasDPVFP]>;
def : Pat<(fneg (any_fma (f32 SPR:$Sn), (f32 SPR:$Sm), (f32 SPR:$Sdin))),
          (VFNMAS SPR:$Sdin, SPR:$Sn, SPR:$Sm)>,
      Requires<[HasVFP4]>;
def : Pat<(fneg (any_fma (f16 HPR:$Sn), (f16 HPR:$Sm), (f16 (f16 HPR:$Sdin)))),
          (VFNMAH (f16 HPR:$Sdin), (f16 HPR:$Sn), (f16 HPR:$Sm))>,
      Requires<[HasFullFP16]>;
// (fma (fneg x), y, (fneg z)) -> (vfnma z, x, y)
def : Pat<(f64 (any_fma (fneg DPR:$Dn), DPR:$Dm, (fneg DPR:$Ddin))),
          (VFNMAD DPR:$Ddin, DPR:$Dn, DPR:$Dm)>,
      Requires<[HasVFP4,HasDPVFP]>;
def : Pat<(f32 (any_fma (fneg SPR:$Sn), SPR:$Sm, (fneg SPR:$Sdin))),
          (VFNMAS SPR:$Sdin, SPR:$Sn, SPR:$Sm)>,
      Requires<[HasVFP4]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2463-2465
```tablegen
def : Pat<(f16 (any_fma (fneg (f16 HPR:$Sn)), (f16 HPR:$Sm), (fneg (f16 HPR:$Sdin)))),
          (VFNMAH (f16 HPR:$Sdin), (f16 HPR:$Sn), (f16 HPR:$Sm))>,
      Requires<[HasFullFP16]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2467-2475
```tablegen
let mayRaiseFPException = 1, Uses = [FPSCR_RM] in
def VFNMSD : ADbI<0b11101, 0b01, 0, 0,
                  (outs DPR:$Dd), (ins DPR:$Ddin, DPR:$Dn, DPR:$Dm),
                  IIC_fpFMAC64, "vfnms", ".f64\t$Dd, $Dn, $Dm",
                  [(set DPR:$Dd, (fsub_mlx (fmul_su DPR:$Dn, DPR:$Dm),
                                           (f64 DPR:$Ddin)))]>,
               RegConstraint<"$Ddin = $Dd">,
               Requires<[HasVFP4,HasDPVFP,UseFusedMAC]>,
               Sched<[WriteFPMAC64, ReadFPMAC, ReadFPMUL, ReadFPMUL]>;
```
- EN: Defines TableGen record `VFNMSD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VFNMSD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2477-2487
```tablegen
let mayRaiseFPException = 1, Uses = [FPSCR_RM] in
def VFNMSS : ASbI<0b11101, 0b01, 0, 0,
                  (outs SPR:$Sd), (ins SPR:$Sdin, SPR:$Sn, SPR:$Sm),
                  IIC_fpFMAC32, "vfnms", ".f32\t$Sd, $Sn, $Sm",
             [(set SPR:$Sd, (fsub_mlx (fmul_su SPR:$Sn, SPR:$Sm), SPR:$Sdin))]>,
                         RegConstraint<"$Sdin = $Sd">,
                  Requires<[HasVFP4,DontUseNEONForFP,UseFusedMAC]>,
                  Sched<[WriteFPMAC32, ReadFPMAC, ReadFPMUL, ReadFPMUL]> {
  // Some single precision VFP instructions may be executed on both NEON and
  // VFP pipelines.
}
```
- EN: Defines TableGen record `VFNMSS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VFNMSS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2489-2496
```tablegen
let mayRaiseFPException = 1, Uses = [FPSCR_RM] in
def VFNMSH : AHbI<0b11101, 0b01, 0, 0,
                  (outs HPR:$Sd), (ins HPR:$Sdin, HPR:$Sn, HPR:$Sm),
                  IIC_fpFMAC16, "vfnms", ".f16\t$Sd, $Sn, $Sm",
             [(set (f16 HPR:$Sd), (fsub_mlx (fmul_su (f16 HPR:$Sn), (f16 HPR:$Sm)), (f16 HPR:$Sdin)))]>,
                         RegConstraint<"$Sdin = $Sd">,
                  Requires<[HasFullFP16,UseFusedMAC]>,
                  Sched<[WriteFPMAC32, ReadFPMAC, ReadFPMUL, ReadFPMUL]>;
```
- EN: Defines TableGen record `VFNMSH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VFNMSH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2498-2503
```tablegen
def : Pat<(fsub_mlx (fmul_su DPR:$a, (f64 DPR:$b)), DPR:$dstin),
          (VFNMSD DPR:$dstin, DPR:$a, DPR:$b)>,
          Requires<[HasVFP4,HasDPVFP,UseFusedMAC]>;
def : Pat<(fsub_mlx (fmul_su SPR:$a, SPR:$b), SPR:$dstin),
          (VFNMSS SPR:$dstin, SPR:$a, SPR:$b)>,
          Requires<[HasVFP4,DontUseNEONForFP,UseFusedMAC]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2505-2505
```tablegen
// Match @llvm.fma.* intrinsics
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 2507-2524
```tablegen
// (fma x, y, (fneg z)) -> (vfnms z, x, y))
def : Pat<(f64 (any_fma DPR:$Dn, DPR:$Dm, (fneg DPR:$Ddin))),
          (VFNMSD DPR:$Ddin, DPR:$Dn, DPR:$Dm)>,
      Requires<[HasVFP4,HasDPVFP]>;
def : Pat<(f32 (any_fma SPR:$Sn, SPR:$Sm, (fneg SPR:$Sdin))),
          (VFNMSS SPR:$Sdin, SPR:$Sn, SPR:$Sm)>,
      Requires<[HasVFP4]>;
def : Pat<(f16 (any_fma (f16 HPR:$Sn), (f16 HPR:$Sm), (fneg (f16 HPR:$Sdin)))),
          (VFNMSH (f16 HPR:$Sdin), (f16 HPR:$Sn), (f16 HPR:$Sm))>,
      Requires<[HasFullFP16]>;
// (fneg (fma (fneg x), y, z)) -> (vfnms z, x, y)
def : Pat<(fneg (f64 (any_fma (fneg DPR:$Dn), DPR:$Dm, DPR:$Ddin))),
          (VFNMSD DPR:$Ddin, DPR:$Dn, DPR:$Dm)>,
      Requires<[HasVFP4,HasDPVFP]>;
def : Pat<(fneg (f32 (any_fma (fneg SPR:$Sn), SPR:$Sm, SPR:$Sdin))),
          (VFNMSS SPR:$Sdin, SPR:$Sn, SPR:$Sm)>,
      Requires<[HasVFP4]>;
def : Pat<(fneg (f16 (any_fma (fneg (f16 HPR:$Sn)), (f16 HPR:$Sm), (f16 HPR:$Sdin)))),
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2525-2526
```tablegen
          (VFNMSH (f16 HPR:$Sdin), (f16 HPR:$Sn), (f16 HPR:$Sm))>,
      Requires<[HasFullFP16]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2528-2530
```tablegen
//===----------------------------------------------------------------------===//
// FP Conditional moves.
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 2532-2535
```tablegen
let hasSideEffects = 0 in {
def VMOVDcc  : PseudoInst<(outs DPR:$Dd), (ins DPR:$Dn, DPR:$Dm, pred:$p),
                          IIC_fpUNA64, []>,
               RegConstraint<"$Dn = $Dd">, Requires<[HasFPRegs64]>;
```
- EN: Defines TableGen record `VMOVDcc` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VMOVDcc`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2537-2539
```tablegen
def VMOVScc  : PseudoInst<(outs SPR:$Sd), (ins SPR:$Sn, SPR:$Sm, pred:$p),
                          IIC_fpUNA32, []>,
               RegConstraint<"$Sn = $Sd">, Requires<[HasFPRegs]>;
```
- EN: Defines TableGen record `VMOVScc` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VMOVScc`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2541-2544
```tablegen
def VMOVHcc  : PseudoInst<(outs HPR:$Sd), (ins HPR:$Sn, HPR:$Sm, pred:$p),
                          IIC_fpUNA16, []>,
               RegConstraint<"$Sd = $Sn">, Requires<[HasFPRegs]>;
} // hasSideEffects
```
- EN: Defines TableGen record `VMOVHcc` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VMOVHcc`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2546-2548
```tablegen
// The following patterns have to be defined out-of-line because the number
// of instruction operands does not match the number of SDNode operands
// (`pred` counts as one operand).
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 2550-2552
```tablegen
def : Pat<(ARMcmov f64:$Dn, f64:$Dm, imm:$cc, CPSR),
          (VMOVDcc $Dn, $Dm, imm:$cc, CPSR)>,
      Requires<[HasFPRegs64]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2554-2556
```tablegen
def : Pat<(ARMcmov f32:$Sn, f32:$Sm, imm:$cc, CPSR),
          (VMOVScc $Sn, $Sm, imm:$cc, CPSR)>,
      Requires<[HasFPRegs]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2558-2560
```tablegen
def : Pat<(ARMcmov f16:$Sn, f16:$Sm, imm:$cc, CPSR),
          (VMOVHcc $Sn, $Sm, imm:$cc, CPSR)>,
      Requires<[HasFPRegs]>; // FIXME: Shouldn't this be HasFPRegs16?
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2562-2564
```tablegen
//===----------------------------------------------------------------------===//
// Move from VFP System Register to ARM core register.
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 2566-2568
```tablegen
class MovFromVFP<bits<4> opc19_16, dag oops, dag iops, string opc, string asm,
                 list<dag> pattern>:
  VFPAI<oops, iops, VFPMiscFrm, IIC_fpSTAT, opc, asm, "", pattern> {
```
- EN: Declares reusable TableGen class `MovFromVFP` for `ARMInstrVFP`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrVFP` 声明可复用的 TableGen 类 `MovFromVFP`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2570-2571
```tablegen
  // Instruction operand.
  bits<4> Rt;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2573-2582
```tablegen
  let Inst{27-20} = 0b11101111;
  let Inst{19-16} = opc19_16;
  let Inst{15-12} = Rt;
  let Inst{11-8}  = 0b1010;
  let Inst{7}     = 0;
  let Inst{6-5}   = 0b00;
  let Inst{4}     = 1;
  let Inst{3-0}   = 0b0000;
  let Unpredictable{7-5} = 0b111;
  let Unpredictable{3-0} = 0b1111;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2584-2586
```tablegen
  // Needed to avoid errors when a MachineInstrt::FrameSetup flag is set.
  let mayStore = 0;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2588-2595
```tablegen
let DecoderMethod = "DecodeForVMRSandVMSR" in {
 // APSR is the application level alias of CPSR. This FPSCR N, Z, C, V flags
 // to APSR.
 let Defs = [CPSR], Uses = [FPSCR_NZCV], Predicates = [HasFPRegs],
     Rt = 0b1111 /* apsr_nzcv */ in
 def FMSTAT : MovFromVFP<0b0001 /* fpscr */, (outs), (ins),
                         "vmrs", "\tAPSR_nzcv, fpscr",
                         [(set CPSR, (arm_fmstat FPSCR_NZCV))]>;
```
- EN: Defines TableGen record `FMSTAT` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FMSTAT`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2597-2601
```tablegen
 // Application level FPSCR -> GPR
 let hasSideEffects = 1, Uses = [FPSCR], Predicates = [HasFPRegs] in
 def VMRS :  MovFromVFP<0b0001 /* fpscr */, (outs GPRnopc:$Rt), (ins),
                        "vmrs", "\t$Rt, fpscr",
                        [(set GPRnopc:$Rt, (int_arm_get_fpscr))]>;
```
- EN: Defines TableGen record `VMRS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VMRS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2603-2620
```tablegen
 // System level FPEXC, FPSID -> GPR
 let Uses = [FPSCR] in {
   def VMRS_FPEXC : MovFromVFP<0b1000 /* fpexc */, (outs GPRnopc:$Rt), (ins),
                               "vmrs", "\t$Rt, fpexc", []>;
   def VMRS_FPSID : MovFromVFP<0b0000 /* fpsid */, (outs GPRnopc:$Rt), (ins),
                               "vmrs", "\t$Rt, fpsid", []>;
   def VMRS_MVFR0 : MovFromVFP<0b0111 /* mvfr0 */, (outs GPRnopc:$Rt), (ins),
                              "vmrs", "\t$Rt, mvfr0", []>;
   def VMRS_MVFR1 : MovFromVFP<0b0110 /* mvfr1 */, (outs GPRnopc:$Rt), (ins),
                               "vmrs", "\t$Rt, mvfr1", []>;
   let Predicates = [HasFPARMv8] in {
     def VMRS_MVFR2 : MovFromVFP<0b0101 /* mvfr2 */, (outs GPRnopc:$Rt), (ins),
                                 "vmrs", "\t$Rt, mvfr2", []>;
   }
   def VMRS_FPINST : MovFromVFP<0b1001 /* fpinst */, (outs GPRnopc:$Rt), (ins),
                                "vmrs", "\t$Rt, fpinst", []>;
   def VMRS_FPINST2 : MovFromVFP<0b1010 /* fpinst2 */, (outs GPRnopc:$Rt),
                                 (ins), "vmrs", "\t$Rt, fpinst2", []>;
```
- EN: Defines TableGen record `VMRS_FPEXC` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VMRS_FPEXC`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2621-2638
```tablegen
   let Predicates = [HasV8_1MMainline, HasFPRegs] in {
     // System level FPSCR_NZCVQC -> GPR
     def VMRS_FPSCR_NZCVQC
       : MovFromVFP<0b0010 /* fpscr_nzcvqc */,
                    (outs GPR:$Rt), (ins cl_FPSCR_NZCV:$fpscr_in),
                    "vmrs", "\t$Rt, fpscr_nzcvqc", []>;
   }
 }
 let Predicates = [HasV8_1MMainline, Has8MSecExt] in {
   // System level FPSCR -> GPR, with context saving for security extensions
   def VMRS_FPCXTNS : MovFromVFP<0b1110 /* fpcxtns */, (outs GPR:$Rt), (ins),
                                 "vmrs", "\t$Rt, fpcxtns", []>;
 }
 let Predicates = [HasV8_1MMainline, Has8MSecExt] in {
   // System level FPSCR -> GPR, with context saving for security extensions
   def VMRS_FPCXTS : MovFromVFP<0b1111 /* fpcxts */, (outs GPR:$Rt), (ins),
                                "vmrs", "\t$Rt, fpcxts", []>;
 }
```
- EN: Defines TableGen record `VMRS_FPSCR_NZCVQC` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VMRS_FPSCR_NZCVQC`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2640-2644
```tablegen
 let Predicates = [HasV8_1MMainline, HasMVEInt] in {
   // System level VPR/P0 -> GPR
   let Uses = [VPR] in
   def VMRS_VPR : MovFromVFP<0b1100 /* vpr */, (outs GPR:$Rt), (ins),
                             "vmrs", "\t$Rt, vpr", []>;
```
- EN: Defines TableGen record `VMRS_VPR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VMRS_VPR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2646-2649
```tablegen
   def VMRS_P0  : MovFromVFP<0b1101 /* p0 */, (outs GPR:$Rt), (ins VCCR:$cond),
                             "vmrs", "\t$Rt, p0", []>;
 }
}
```
- EN: Defines TableGen record `VMRS_P0` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VMRS_P0`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2651-2653
```tablegen
//===----------------------------------------------------------------------===//
// Move from ARM core register to VFP System Register.
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 2655-2657
```tablegen
class MovToVFP<bits<4> opc19_16, dag oops, dag iops, string opc, string asm,
               list<dag> pattern>:
  VFPAI<oops, iops, VFPMiscFrm, IIC_fpSTAT, opc, asm, "", pattern> {
```
- EN: Declares reusable TableGen class `MovToVFP` for `ARMInstrVFP`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrVFP` 声明可复用的 TableGen 类 `MovToVFP`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2659-2660
```tablegen
  // Instruction operand.
  bits<4> Rt;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2662-2673
```tablegen
  let Inst{27-20} = 0b11101110;
  let Inst{19-16} = opc19_16;
  let Inst{15-12} = Rt;
  let Inst{11-8}  = 0b1010;
  let Inst{7}     = 0;
  let Inst{6-5}   = 0b00;
  let Inst{4}     = 1;
  let Inst{3-0}   = 0b0000;
  let Predicates = [HasVFP2];
  let Unpredictable{7-5} = 0b111;
  let Unpredictable{3-0} = 0b1111;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2675-2692
```tablegen
let DecoderMethod = "DecodeForVMRSandVMSR" in {
 let Defs = [FPSCR] in {
   let Predicates = [HasFPRegs] in
   // Application level GPR -> FPSCR
   def VMSR : MovToVFP<0b0001 /* fpscr */, (outs), (ins GPRnopc:$Rt),
                       "vmsr", "\tfpscr, $Rt",
                       [(int_arm_set_fpscr GPRnopc:$Rt)]>;
   // System level GPR -> FPEXC
   def VMSR_FPEXC : MovToVFP<0b1000 /* fpexc */, (outs), (ins GPRnopc:$Rt),
                               "vmsr", "\tfpexc, $Rt", []>;
   // System level GPR -> FPSID
   def VMSR_FPSID : MovToVFP<0b0000 /* fpsid */, (outs), (ins GPRnopc:$Rt),
                             "vmsr", "\tfpsid, $Rt", []>;
   def VMSR_FPINST : MovToVFP<0b1001 /* fpinst */, (outs), (ins GPRnopc:$Rt),
                              "vmsr", "\tfpinst, $Rt", []>;
   def VMSR_FPINST2 : MovToVFP<0b1010 /* fpinst2 */, (outs), (ins GPRnopc:$Rt),
                               "vmsr", "\tfpinst2, $Rt", []>;
 }
```
- EN: Defines TableGen record `VMSR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VMSR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2693-2709
```tablegen
 let Predicates = [HasV8_1MMainline, Has8MSecExt] in {
   // System level GPR -> FPSCR with context saving for security extensions
   def VMSR_FPCXTNS : MovToVFP<0b1110 /* fpcxtns */, (outs), (ins GPR:$Rt),
                               "vmsr", "\tfpcxtns, $Rt", []>;
 }
 let Predicates = [HasV8_1MMainline, Has8MSecExt] in {
   // System level GPR -> FPSCR with context saving for security extensions
   def VMSR_FPCXTS : MovToVFP<0b1111 /* fpcxts */, (outs), (ins GPR:$Rt),
                              "vmsr", "\tfpcxts, $Rt", []>;
 }
 let Predicates = [HasV8_1MMainline, HasFPRegs] in {
   // System level GPR -> FPSCR_NZCVQC
   def VMSR_FPSCR_NZCVQC
     : MovToVFP<0b0010 /* fpscr_nzcvqc */,
                (outs cl_FPSCR_NZCV:$fpscr_out), (ins GPR:$Rt),
                "vmsr", "\tfpscr_nzcvqc, $Rt", []>;
 }
```
- EN: Defines TableGen record `VMSR_FPCXTNS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VMSR_FPCXTNS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2711-2715
```tablegen
 let Predicates = [HasV8_1MMainline, HasMVEInt] in {
   // System level GPR -> VPR/P0
   let Defs = [VPR] in
   def VMSR_VPR : MovToVFP<0b1100 /* vpr */, (outs), (ins GPR:$Rt),
                           "vmsr", "\tvpr, $Rt", []>;
```
- EN: Defines TableGen record `VMSR_VPR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VMSR_VPR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2717-2720
```tablegen
   def VMSR_P0  : MovToVFP<0b1101 /* p0 */, (outs VCCR:$cond), (ins GPR:$Rt),
                           "vmsr", "\tp0, $Rt", []>;
 }
}
```
- EN: Defines TableGen record `VMSR_P0` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VMSR_P0`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2722-2724
```tablegen
//===----------------------------------------------------------------------===//
// Misc.
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 2726-2734
```tablegen
// Materialize FP immediates. VFP3 only.
let isReMaterializable = 1 in {
def FCONSTD : VFPAI<(outs DPR:$Dd), (ins vfp_f64imm:$imm),
                    VFPMiscFrm, IIC_fpUNA64,
                    "vmov", ".f64\t$Dd, $imm", "",
                    [(set DPR:$Dd, vfp_f64imm:$imm)]>,
              Requires<[HasVFP3,HasDPVFP]> {
  bits<5> Dd;
  bits<8> imm;
```
- EN: Defines TableGen record `FCONSTD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FCONSTD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2736-2745
```tablegen
  let Inst{27-23} = 0b11101;
  let Inst{22}    = Dd{4};
  let Inst{21-20} = 0b11;
  let Inst{19-16} = imm{7-4};
  let Inst{15-12} = Dd{3-0};
  let Inst{11-9}  = 0b101;
  let Inst{8}     = 1;          // Double precision.
  let Inst{7-4}   = 0b0000;
  let Inst{3-0}   = imm{3-0};
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2747-2752
```tablegen
def FCONSTS : VFPAI<(outs SPR:$Sd), (ins vfp_f32imm:$imm),
                     VFPMiscFrm, IIC_fpUNA32,
                     "vmov", ".f32\t$Sd, $imm", "",
                     [(set SPR:$Sd, vfp_f32imm:$imm)]>, Requires<[HasVFP3]> {
  bits<5> Sd;
  bits<8> imm;
```
- EN: Defines TableGen record `FCONSTS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FCONSTS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2754-2763
```tablegen
  let Inst{27-23} = 0b11101;
  let Inst{22}    = Sd{0};
  let Inst{21-20} = 0b11;
  let Inst{19-16} = imm{7-4};
  let Inst{15-12} = Sd{4-1};
  let Inst{11-9}  = 0b101;
  let Inst{8}     = 0;          // Single precision.
  let Inst{7-4}   = 0b0000;
  let Inst{3-0}   = imm{3-0};
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2765-2771
```tablegen
def FCONSTH : VFPAI<(outs HPR:$Sd), (ins vfp_f16imm:$imm),
                     VFPMiscFrm, IIC_fpUNA16,
                     "vmov", ".f16\t$Sd, $imm", "",
                     [(set (f16 HPR:$Sd), vfp_f16imm:$imm)]>,
              Requires<[HasFullFP16]> {
  bits<5> Sd;
  bits<8> imm;
```
- EN: Defines TableGen record `FCONSTH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FCONSTH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2773-2780
```tablegen
  let Inst{27-23} = 0b11101;
  let Inst{22}    = Sd{0};
  let Inst{21-20} = 0b11;
  let Inst{19-16} = imm{7-4};
  let Inst{15-12} = Sd{4-1};
  let Inst{11-8}  = 0b1001;     // Half precision
  let Inst{7-4}   = 0b0000;
  let Inst{3-0}   = imm{3-0};
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2782-2784
```tablegen
  let isUnpredicable = 1;
}
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2786-2789
```tablegen
def : Pat<(f32 (vfp_f32f16imm:$imm)),
          (f32 (COPY_TO_REGCLASS (f16 (FCONSTH (vfp_f32f16imm_xform (f32 $imm)))), SPR))> {
  let Predicates = [HasFullFP16];
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2791-2796
```tablegen
// Floating-point environment management.
def : Pat<(get_fpenv), (VMRS)>;
def : Pat<(set_fpenv GPRnopc:$Rt), (VMSR GPRnopc:$Rt)>;
def : Pat<(reset_fpenv), (VMSR (MOVi 0))>, Requires<[IsARM]>;
def : Pat<(reset_fpenv), (VMSR (tMOVi8 0))>, Requires<[IsThumb]>;
def : Pat<(get_fpmode), (VMRS)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2798-2815
```tablegen
//===----------------------------------------------------------------------===//
// Assembler aliases.
//
// A few mnemonic aliases for pre-unifixed syntax. We don't guarantee to
// support them all, but supporting at least some of the basics is
// good to be friendly.
def : VFP2MnemonicAlias<"flds", "vldr">;
def : VFP2MnemonicAlias<"fldd", "vldr">;
def : VFP2MnemonicAlias<"fmrs", "vmov">;
def : VFP2MnemonicAlias<"fmsr", "vmov">;
def : VFP2MnemonicAlias<"fsqrts", "vsqrt">;
def : VFP2MnemonicAlias<"fsqrtd", "vsqrt">;
def : VFP2MnemonicAlias<"fadds", "vadd.f32">;
def : VFP2MnemonicAlias<"faddd", "vadd.f64">;
def : VFP2MnemonicAlias<"fmrdd", "vmov">;
def : VFP2MnemonicAlias<"fmrds", "vmov">;
def : VFP2MnemonicAlias<"fmrrd", "vmov">;
def : VFP2MnemonicAlias<"fmdrr", "vmov">;
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 2816-2833
```tablegen
def : VFP2MnemonicAlias<"fmuls", "vmul.f32">;
def : VFP2MnemonicAlias<"fmuld", "vmul.f64">;
def : VFP2MnemonicAlias<"fnegs", "vneg.f32">;
def : VFP2MnemonicAlias<"fnegd", "vneg.f64">;
def : VFP2MnemonicAlias<"ftosizd", "vcvt.s32.f64">;
def : VFP2MnemonicAlias<"ftosid", "vcvtr.s32.f64">;
def : VFP2MnemonicAlias<"ftosizs", "vcvt.s32.f32">;
def : VFP2MnemonicAlias<"ftosis", "vcvtr.s32.f32">;
def : VFP2MnemonicAlias<"ftouizd", "vcvt.u32.f64">;
def : VFP2MnemonicAlias<"ftouid", "vcvtr.u32.f64">;
def : VFP2MnemonicAlias<"ftouizs", "vcvt.u32.f32">;
def : VFP2MnemonicAlias<"ftouis", "vcvtr.u32.f32">;
def : VFP2MnemonicAlias<"fsitod", "vcvt.f64.s32">;
def : VFP2MnemonicAlias<"fsitos", "vcvt.f32.s32">;
def : VFP2MnemonicAlias<"fuitod", "vcvt.f64.u32">;
def : VFP2MnemonicAlias<"fuitos", "vcvt.f32.u32">;
def : VFP2MnemonicAlias<"fsts", "vstr">;
def : VFP2MnemonicAlias<"fstd", "vstr">;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2834-2843
```tablegen
def : VFP2MnemonicAlias<"fmacd", "vmla.f64">;
def : VFP2MnemonicAlias<"fmacs", "vmla.f32">;
def : VFP2MnemonicAlias<"fcpys", "vmov.f32">;
def : VFP2MnemonicAlias<"fcpyd", "vmov.f64">;
def : VFP2MnemonicAlias<"fcmps", "vcmp.f32">;
def : VFP2MnemonicAlias<"fcmpd", "vcmp.f64">;
def : VFP2MnemonicAlias<"fdivs", "vdiv.f32">;
def : VFP2MnemonicAlias<"fdivd", "vdiv.f64">;
def : VFP2MnemonicAlias<"fmrx", "vmrs">;
def : VFP2MnemonicAlias<"fmxr", "vmsr">;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2845-2847
```tablegen
// Be friendly and accept the old form of zero-compare
def : VFP2DPInstAlias<"fcmpzd${p} $val", (VCMPZD DPR:$val, pred:$p)>;
def : VFP2InstAlias<"fcmpzs${p} $val", (VCMPZS SPR:$val, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2850-2858
```tablegen
def : InstAlias<"fmstat${p}", (FMSTAT pred:$p), 0>, Requires<[HasFPRegs]>;
def : VFP2InstAlias<"fadds${p} $Sd, $Sn, $Sm",
                    (VADDS SPR:$Sd, SPR:$Sn, SPR:$Sm, pred:$p)>;
def : VFP2DPInstAlias<"faddd${p} $Dd, $Dn, $Dm",
                      (VADDD DPR:$Dd, DPR:$Dn, DPR:$Dm, pred:$p)>;
def : VFP2InstAlias<"fsubs${p} $Sd, $Sn, $Sm",
                    (VSUBS SPR:$Sd, SPR:$Sn, SPR:$Sm, pred:$p)>;
def : VFP2DPInstAlias<"fsubd${p} $Dd, $Dn, $Dm",
                      (VSUBD DPR:$Dd, DPR:$Dn, DPR:$Dm, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2860-2862
```tablegen
// No need for the size suffix on VSQRT. It's implied by the register classes.
def : VFP2InstAlias<"vsqrt${p} $Sd, $Sm", (VSQRTS SPR:$Sd, SPR:$Sm, pred:$p)>;
def : VFP2DPInstAlias<"vsqrt${p} $Dd, $Dm", (VSQRTD DPR:$Dd, DPR:$Dm, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2864-2872
```tablegen
// VLDR/VSTR accept an optional type suffix.
def : VFP2InstAlias<"vldr${p}.32 $Sd, $addr",
                    (VLDRS SPR:$Sd, addrmode5:$addr, pred:$p)>;
def : VFP2InstAlias<"vstr${p}.32 $Sd, $addr",
                    (VSTRS SPR:$Sd, addrmode5:$addr, pred:$p)>;
def : VFP2InstAlias<"vldr${p}.64 $Dd, $addr",
                    (VLDRD DPR:$Dd, addrmode5:$addr, pred:$p)>;
def : VFP2InstAlias<"vstr${p}.64 $Dd, $addr",
                    (VSTRD DPR:$Dd, addrmode5:$addr, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2874-2886
```tablegen
// VMOV can accept optional 32-bit or less data type suffix suffix.
def : VFP2InstAlias<"vmov${p}.8 $Rt, $Sn",
                    (VMOVRS GPR:$Rt, SPR:$Sn, pred:$p)>;
def : VFP2InstAlias<"vmov${p}.16 $Rt, $Sn",
                    (VMOVRS GPR:$Rt, SPR:$Sn, pred:$p)>;
def : VFP2InstAlias<"vmov${p}.32 $Rt, $Sn",
                    (VMOVRS GPR:$Rt, SPR:$Sn, pred:$p)>;
def : VFP2InstAlias<"vmov${p}.8 $Sn, $Rt",
                    (VMOVSR SPR:$Sn, GPR:$Rt, pred:$p)>;
def : VFP2InstAlias<"vmov${p}.16 $Sn, $Rt",
                    (VMOVSR SPR:$Sn, GPR:$Rt, pred:$p)>;
def : VFP2InstAlias<"vmov${p}.32 $Sn, $Rt",
                    (VMOVSR SPR:$Sn, GPR:$Rt, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2888-2891
```tablegen
def : VFP2InstAlias<"vmov${p}.f64 $Rt, $Rt2, $Dn",
                    (VMOVRRD GPR:$Rt, GPR:$Rt2, DPR:$Dn, pred:$p)>;
def : VFP2InstAlias<"vmov${p}.f64 $Dn, $Rt, $Rt2",
                    (VMOVDRR DPR:$Dn, GPR:$Rt, GPR:$Rt2, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2893-2896
```tablegen
// VMOVS doesn't need the .f32 to disambiguate from the NEON encoding the way
// VMOVD does.
def : VFP2InstAlias<"vmov${p} $Sd, $Sm",
                    (VMOVS SPR:$Sd, SPR:$Sm, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2898-2907
```tablegen
// FCONSTD/FCONSTS alias for vmov.f64/vmov.f32
// These aliases provide added functionality over vmov.f instructions by
// allowing users to write assembly containing encoded floating point constants
// (e.g. #0x70 vs #1.0).  Without these alises there is no way for the
// assembler to accept encoded fp constants (but the equivalent fp-literal is
// accepted directly by vmovf).
def : VFP3InstAlias<"fconstd${p} $Dd, $val",
                    (FCONSTD DPR:$Dd, vfp_f64imm:$val, pred:$p)>;
def : VFP3InstAlias<"fconsts${p} $Sd, $val",
                    (FCONSTS SPR:$Sd, vfp_f32imm:$val, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2909-2919
```tablegen
def VSCCLRMD : VFPXI<(outs), (ins pred:$p, fp_dreglist_with_vpr:$regs, variable_ops),
                      AddrModeNone, 4, IndexModeNone, VFPMiscFrm, NoItinerary,
                      "vscclrm{$p}\t$regs", "", []>, Sched<[]> {
  bits<13> regs;
  let Inst{31-23} = 0b111011001;
  let Inst{22} = regs{12};
  let Inst{21-16} = 0b011111;
  let Inst{15-12} = regs{11-8};
  let Inst{11-8} = 0b1011;
  let Inst{7-1} = regs{7-1};
  let Inst{0} = 0;
```
- EN: Defines TableGen record `VSCCLRMD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VSCCLRMD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2921-2921
```tablegen
  let DecoderMethod = "DecodeVSCCLRM";
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2923-2924
```tablegen
  list<Predicate> Predicates = [HasV8_1MMainline, Has8MSecExt];
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2926-2935
```tablegen
def VSCCLRMS : VFPXI<(outs), (ins pred:$p, fp_sreglist_with_vpr:$regs, variable_ops),
                      AddrModeNone, 4, IndexModeNone, VFPMiscFrm, NoItinerary,
                      "vscclrm{$p}\t$regs", "", []>, Sched<[]> {
  bits<13> regs;
  let Inst{31-23} = 0b111011001;
  let Inst{22} = regs{8};
  let Inst{21-16} = 0b011111;
  let Inst{15-12} = regs{12-9};
  let Inst{11-8} = 0b1010;
  let Inst{7-0} = regs{7-0};
```
- EN: Defines TableGen record `VSCCLRMS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VSCCLRMS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2937-2937
```tablegen
  let DecoderMethod = "DecodeVSCCLRM";
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2939-2940
```tablegen
  list<Predicate> Predicates = [HasV8_1MMainline, Has8MSecExt];
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2942-2944
```tablegen
//===----------------------------------------------------------------------===//
// Store VFP System Register to memory.
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 2946-2963
```tablegen
class vfp_vstrldr<bit opc, bit P, bit W, bits<4> SysReg, string sysreg,
                  dag oops, dag iops, IndexMode im, string Dest, string cstr>
    : VFPI<oops, iops, AddrModeT2_i7s4, 4, im, VFPLdStFrm, IIC_fpSTAT,
           !if(opc,"vldr","vstr"), !strconcat("\t", sysreg, ", ", Dest), cstr, []>,
      Sched<[]> {
  bits<12> addr;
  let Inst{27-25} = 0b110;
  let Inst{24} = P;
  let Inst{23} = addr{7};
  let Inst{22} = SysReg{3};
  let Inst{21} = W;
  let Inst{20} = opc;
  let Inst{19-16} = addr{11-8};
  let Inst{15-13} = SysReg{2-0};
  let Inst{12-7} = 0b011111;
  let Inst{6-0} = addr{6-0};
  list<Predicate> Predicates = [HasFPRegs, HasV8_1MMainline];
  let mayLoad = opc;
```
- EN: Declares reusable TableGen class `vfp_vstrldr` for `ARMInstrVFP`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrVFP` 声明可复用的 TableGen 类 `vfp_vstrldr`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2964-2966
```tablegen
  let mayStore = !if(opc, 0b0, 0b1);
  let hasSideEffects = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2968-2975
```tablegen
multiclass vfp_vstrldr_sysreg<bit opc, bits<4> SysReg, string sysreg,
                              dag oops=(outs), dag iops=(ins)> {
  def _off :
    vfp_vstrldr<opc, 1, 0, SysReg, sysreg,
                oops, !con(iops, (ins t2addrmode_imm7s4:$addr)),
                IndexModePost, "$addr", "" > {
    let DecoderMethod = "DecodeVSTRVLDR_SYSREG<false>";
  }
```
- EN: Declares TableGen `multiclass vfp_vstrldr_sysreg`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass vfp_vstrldr_sysreg`，它是一个可复用模板，可展开为多个相关记录。

### Lines 2977-2983
```tablegen
  def _pre :
    vfp_vstrldr<opc, 1, 1, SysReg, sysreg,
                !con(oops, (outs GPRnopc:$wb)),
                !con(iops, (ins t2addrmode_imm7s4_pre:$addr)),
                IndexModePre, "$addr!", "$addr.base = $wb"> {
    let DecoderMethod = "DecodeVSTRVLDR_SYSREG<true>";
  }
```
- EN: Defines TableGen record `_pre` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `_pre`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2985-2995
```tablegen
  def _post :
    vfp_vstrldr<opc, 0, 1, SysReg, sysreg,
                !con(oops, (outs GPRnopc:$wb)),
                !con(iops, (ins t2_addr_offset_none:$Rn,
                                t2am_imm7s4_offset:$addr)),
                IndexModePost, "$Rn$addr", "$Rn.base = $wb"> {
   bits<4> Rn;
   let Inst{19-16} = Rn{3-0};
   let DecoderMethod = "DecodeVSTRVLDR_SYSREG<true>";
 }
}
```
- EN: Defines TableGen record `_post` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `_post`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2997-2998
```tablegen
let Uses = [FPSCR] in {
  defm VSTR_FPSCR          : vfp_vstrldr_sysreg<0b0,0b0001, "fpscr">;
```
- EN: Defines TableGen record `VSTR_FPSCR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VSTR_FPSCR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3000-3004
```tablegen
  let Predicates = [HasV8_1MMainline, Has8MSecExt] in {
    defm VSTR_FPCXTNS      : vfp_vstrldr_sysreg<0b0,0b1110, "fpcxtns">;
    defm VSTR_FPCXTS       : vfp_vstrldr_sysreg<0b0,0b1111, "fpcxts">;
  }
}
```
- EN: Defines TableGen record `VSTR_FPCXTNS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VSTR_FPCXTNS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3006-3011
```tablegen
let Predicates = [HasV8_1MMainline, HasMVEInt] in {
  let Uses = [VPR] in {
    defm VSTR_VPR          : vfp_vstrldr_sysreg<0b0,0b1100, "vpr">;
  }
  defm VSTR_P0             : vfp_vstrldr_sysreg<0b0,0b1101, "p0",
                                                (outs), (ins VCCR:$P0)>;
```
- EN: Defines TableGen record `VSTR_VPR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VSTR_VPR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3013-3018
```tablegen
  let Defs = [VPR] in {
    defm VLDR_VPR          : vfp_vstrldr_sysreg<0b1,0b1100, "vpr">;
  }
  defm VLDR_P0             : vfp_vstrldr_sysreg<0b1,0b1101, "p0",
                                                (outs VCCR:$P0), (ins)>;
}
```
- EN: Defines TableGen record `VLDR_VPR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLDR_VPR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3020-3021
```tablegen
let Defs = [FPSCR] in {
  defm VLDR_FPSCR          : vfp_vstrldr_sysreg<0b1,0b0001, "fpscr">;
```
- EN: Defines TableGen record `VLDR_FPSCR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLDR_FPSCR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3023-3027
```tablegen
  let Predicates = [HasV8_1MMainline, Has8MSecExt] in {
    defm VLDR_FPCXTNS      : vfp_vstrldr_sysreg<0b1,0b1110, "fpcxtns">;
    defm VLDR_FPCXTS       : vfp_vstrldr_sysreg<0b1,0b1111, "fpcxts">;
  }
}
```
- EN: Defines TableGen record `VLDR_FPCXTNS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLDR_FPCXTNS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3029-3034
```tablegen
defm VSTR_FPSCR_NZCVQC   : vfp_vstrldr_sysreg<0b0,0b0010, "fpscr_nzcvqc",
                                              (outs), (ins cl_FPSCR_NZCV:$fpscr)>;
let canFoldAsLoad = 1, isReMaterializable = 1 in {
defm VLDR_FPSCR_NZCVQC   : vfp_vstrldr_sysreg<0b1,0b0010, "fpscr_nzcvqc",
                                              (outs cl_FPSCR_NZCV:$fpscr), (ins)>;
}
```
- EN: Defines TableGen record `VSTR_FPSCR_NZCVQC` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VSTR_FPSCR_NZCVQC`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

## Key Concepts / 关键概念

- EN: Primary role: instruction definitions and target opcode metadata.
  - CN: 核心职责：指令定义与目标操作码元数据。
- EN: LLVM TableGen DSL records drive generated enums, matchers, encoders, or scheduling data.
  - CN: LLVM TableGen DSL 记录会驱动生成枚举、匹配器、编码器或调度数据。
- EN: SelectionDAG patterns or node profiles connect IR-level intent to target instructions.
  - CN: SelectionDAG 模式或节点轮廓把 IR 层意图连接到目标指令。
- EN: Bitfield assignments describe exact instruction encodings and per-instruction flags.
  - CN: 位字段赋值用于描述精确的指令编码以及每条指令的标志位。
- EN: Predicates and feature gates control when records apply to a subtarget or ISA variant.
  - CN: 谓词与特性门控决定记录在何种子目标或 ISA 变体下生效。

## Dependencies / 依赖关系

- EN: TableGen includes: `(none)`.
  - CN: TableGen 包含项：`(none)`。
- EN: Generated backend tables depend on tblgen consuming these records and emitting C++/inc fragments.
  - CN: 生成后的后端表依赖 tblgen 消费这些记录并产出 C++/inc 片段。
