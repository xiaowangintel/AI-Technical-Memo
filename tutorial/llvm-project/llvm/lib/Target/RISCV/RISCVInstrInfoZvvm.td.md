# RISCVInstrInfoZvvm.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVInstrInfoZvvm.td`
- **Repository**: llvm/llvm-project
- **Purpose**: Defines TableGen records for declarative instruction records, selection patterns, and extension-specific metadata for the RISC-V backend. / 使用 TableGen 定义RISC-V 后端的声明式指令记录、选择模式与扩展专用元数据。
- **Note**: This file uses the TableGen DSL to describe target metadata declaratively. / 该文件使用 TableGen DSL 以声明式方式描述目标元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: Commentary and design intent / 注释与设计意图
```tablegen
//===----------------------------------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file describes the RISC-V instructions from the standard 'Zvvm' family
// of Integrated Matrix extensions.
// These extensions are still experimental as they haven't been ratified yet.
//
//===----------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 14-26: TableGen record VScaleAsmOperand / TableGen 记录 VScaleAsmOperand
```tablegen

//===----------------------------------------------------------------------===//
// Instructions
//===----------------------------------------------------------------------===//

def VScaleAsmOperand : AsmOperandClass {
  let Name = "RVVScaleRegOpOperand";
  let RenderMethod = "addRegOperands";
  let PredicateMethod = "isV0Reg";
  let ParserMethod = "parseVScaleReg";
  let DiagnosticType = "InvalidVScaleRegister";
  let DiagnosticString = "operand must be v0.scale";
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 27-35: TableGen record VScaleOp / TableGen 记录 VScaleOp
```tablegen

// An always present v0.scale operand encoded with vm=0. Classes that use this
// must set the vm field in RVInstV* to 0.
def VScaleOp : RegisterOperand<VMV0> {
  let ParserMatchClass = VScaleAsmOperand;
  let PrintMethod = "printVScaleReg";
  let EncoderMethod = "getVMaskReg";
  let DecoderMethod = "decodeVMaskReg";
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 36-47: TableGen class VIMEMACVV<bits<6> / TableGen 类 VIMEMACVV<bits<6>
```tablegen

class VIMEMACVV<bits<6> funct6, string opcodestr>
    : RVInstVV<funct6, OPIVV, (outs VR:$vd_wb),
               (ins VR:$vd, VR:$vs1, VR:$vs2), opcodestr,
               "$vd, $vs1, $vs2"> {
  let mayLoad = 0;
  let mayStore = 0;
  let hasSideEffects = 0;
  let Constraints = "$vd = $vd_wb";
  let vm = 1;
  let VMConstraint = false;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 48-61: TableGen class VFMEMACVV<bits<6> / TableGen 类 VFMEMACVV<bits<6>
```tablegen

class VFMEMACVV<bits<6> funct6, string opcodestr>
    : RVInstVV<funct6, OPFVV, (outs VR:$vd_wb),
               (ins VR:$vd, VR:$vs1, VR:$vs2), opcodestr,
               "$vd, $vs1, $vs2"> {
  let mayLoad = 0;
  let mayStore = 0;
  let hasSideEffects = 0;
  let Constraints = "$vd = $vd_wb";
  let Uses = [FRM, VL, VTYPE];
  let mayRaiseFPException = true;
  let vm = 1;
  let VMConstraint = false;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 62-75: TableGen class VFMEMACScaleVV<bits<6> / TableGen 类 VFMEMACScaleVV<bits<6>
```tablegen

class VFMEMACScaleVV<bits<6> funct6, string opcodestr>
    : RVInstVV<funct6, OPFVV, (outs VR:$vd_wb),
               (ins VR:$vd, VR:$vs1, VR:$vs2, VScaleOp:$vm), opcodestr,
               "$vd, $vs1, $vs2$vm"> {
  let mayLoad = 0;
  let mayStore = 0;
  let hasSideEffects = 0;
  let Constraints = "$vd = $vd_wb";
  let Uses = [FRM, VL, VTYPE];
  let mayRaiseFPException = true;
  let vm = 0;
  let VMConstraint = false;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 76-89: TableGen class VIFMEMACScaleVV<bits<6> / TableGen 类 VIFMEMACScaleVV<bits<6>
```tablegen

class VIFMEMACScaleVV<bits<6> funct6, string opcodestr>
    : RVInstVV<funct6, OPIVV, (outs VR:$vd_wb),
               (ins VR:$vd, VR:$vs1, VR:$vs2, VScaleOp:$vm), opcodestr,
               "$vd, $vs1, $vs2$vm"> {
  let mayLoad = 0;
  let mayStore = 0;
  let hasSideEffects = 0;
  let Constraints = "$vd = $vd_wb";
  let Uses = [FRM, VL, VTYPE];
  let mayRaiseFPException = true;
  let vm = 0;
  let VMConstraint = false;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 90-97: TableGen record VMMACC_VV / TableGen 记录 VMMACC_VV
```tablegen

let Predicates = [HasStdExtZvvmm] in {
  def VMMACC_VV : VIMEMACVV<0b111000, "vmmacc.vv">;
  def VWMMACC_VV : VIMEMACVV<0b111001, "vwmmacc.vv">;
  def VQMMACC_VV : VIMEMACVV<0b111010, "vqmmacc.vv">;
  def V8WMMACC_VV : VIMEMACVV<0b111011, "v8wmmacc.vv">;
} // Predicates = [HasStdExtZvvmm]
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 98-113: TableGen record VFMMACC_VV / TableGen 记录 VFMMACC_VV
```tablegen
let Predicates = [HasStdExtZvvfmm] in {
  def VFMMACC_VV : VFMEMACVV<0b010100, "vfmmacc.vv">;
  def VFWMMACC_VV : VFMEMACVV<0b010101, "vfwmmacc.vv">;
  def VFQMMACC_VV : VFMEMACVV<0b010110, "vfqmmacc.vv">;
  def VF8WMMACC_VV : VFMEMACVV<0b010111, "vf8wmmacc.vv">;
  def VFWMMACC_VV_SCALE : VFMEMACScaleVV<0b010101, "vfwmmacc.vv">;
  def VFQMMACC_VV_SCALE : VFMEMACScaleVV<0b010110, "vfqmmacc.vv">;
  def VF8WMMACC_VV_SCALE : VFMEMACScaleVV<0b010111, "vf8wmmacc.vv">;
  // FIXME: The integer-input MX forms should be gated by the
  // Zvvxi*/Zvvxni* microscaling extensions once LLVM models those
  // individual extension names. They are temporarily enabled under
  // experimental-zvvfmm for MC bring-up.
  def VFWIMMACC_VV : VIFMEMACScaleVV<0b111001, "vfwimmacc.vv">;
  def VFQIMMACC_VV : VIFMEMACScaleVV<0b111010, "vfqimmacc.vv">;
  def VF8WIMMACC_VV : VIFMEMACScaleVV<0b111011, "vf8wimmacc.vv">;
} // Predicates = [HasStdExtZvvfmm]
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

## Key Concepts / 关键概念
- **Instruction definitions** / **指令定义**
- **TableGen DSL** / **TableGen DSL**

## Dependencies / 依赖关系
- No explicit source includes in this file; dependencies are mostly implicit through surrounding backend integration. / 该文件没有显式源码级 include，依赖主要通过周边后端集成隐式体现。
