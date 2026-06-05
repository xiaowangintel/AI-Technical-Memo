# RISCVInstrFormatsSpacemitV.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVInstrFormatsSpacemitV.td`
- **Repository**: llvm/llvm-project
- **Purpose**: Defines TableGen records for instruction format classes, bit layouts, and operand encodings for RISC-V. / 使用 TableGen 定义RISC-V 的指令格式类、位布局与操作数编码。
- **Note**: This file uses the TableGen DSL to describe target metadata declaratively. / 该文件使用 TableGen DSL 以声明式方式描述目标元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-9: Commentary and design intent / 注释与设计意图
```tablegen
//===----------------------------------------------------------------------===//
//
//  This file describes the RISC-V SPACEMIT RVV extension instruction formats.
//
//===----------------------------------------------------------------------===//

//===----------------------------------------------------------------------===//
// RISC-V SPACEMIT RVV class templates
//===----------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 10-17: TableGen class SMTVDotOpcode<bits<7> / TableGen 类 SMTVDotOpcode<bits<7>
```tablegen
//===----------------------------------------------------------------------===//
// Operand definitions.
//===----------------------------------------------------------------------===//

class SMTVDotOpcode<bits<7> val> { bits<7> Value = val; }
def OPMMA : SMTVDotOpcode<0b1110001>;
def OPMMA_SLIDE : SMTVDotOpcode<0b1110011>;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 18-26: Commentary and design intent / 注释与设计意图
```tablegen
//===----------------------------------------------------------------------===//
// Vector Dot-Product Sign Encoding
// Defines the signed/unsigned mixing modes for vector dot-product operations.
// Encoding format: [1:0] bits
//   00: UU (Unsigned x Unsigned)
//   01: US (Unsigned x Signed)
//   10: SU (Signed x Unsigned)
//   11: SS (Signed x Signed)
//===----------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 27-36: TableGen class VTypeEncode<bits<2> / TableGen 类 VTypeEncode<bits<2>
```tablegen

class VTypeEncode<bits<2> encoding, string name> {
  bits<2> Encoding = encoding;
  string Name = name;
}
defvar VTypeEncodes = [VTypeEncode<0b00, "u">,
                       VTypeEncode<0b01, "us">,
                       VTypeEncode<0b10, "su">,
                       VTypeEncode<0b11, "">];
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 37-47: Commentary and design intent / 注释与设计意图
```tablegen
//===----------------------------------------------------------------------===//
// Vector Dot-Product Sliding Window Modes
// Encoding format: [1:0] bits
//   00: Slide1 (1-element sliding stride)
//   01: Slide2 (2-element sliding stride)
//   10: Slide3 (3-element sliding stride)
//   11: Reserved
//
// Used in sliding-window dot-product operations:
//   vd = vs1 • vs2.slide{1|2|3}  // • = dot product
//===----------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 48-56: TableGen class SlideEncode<bits<2> / TableGen 类 SlideEncode<bits<2>
```tablegen
class SlideEncode<bits<2> encoding, string name> {
  bits<2> Encoding = encoding;
  string Name = name;
}

defvar SlideEncodes = [SlideEncode<0b00, "1">,
                       SlideEncode<0b01, "2">,
                       SlideEncode<0b10, "3">];
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 57-69: TableGen class SMTVMADot<bits<2> / TableGen 类 SMTVMADot<bits<2>
```tablegen
//===----------------------------------------------------------------------===//
// Instruction formats
//===----------------------------------------------------------------------===//

let hasSideEffects = 0, mayLoad = 0, mayStore = 0 in {
  // Base vector dot product (no slide) format.
  class SMTVMADot<bits<2> sign, string opcodestr, bit EarlyClobber = 0>
      : RVInst<(outs VRM2:$vd_wb), (ins VRM2:$vd, VR:$vs1, VR:$vs2),
               opcodestr, "$vd, $vs1, $vs2", [], InstFormatR> {
    bits<5> vd;
    bits<5> vs1;
    bits<5> vs2;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 70-80: TableGen field customization and iteration / TableGen 字段定制与迭代
```tablegen
    let Inst{31-25} = OPMMA.Value;
    let Inst{24-20} = vs2;
    let Inst{19-15} = vs1;
    let Inst{14} = 0b0;
    let Inst{13-12} = sign;
    let Inst{11-7} = vd;
    let Inst{6-0} = OPC_CUSTOM_1.Value;
    let Constraints =
        !if(EarlyClobber, "@earlyclobber $vd_wb, $vd = $vd_wb", "$vd = $vd_wb");
    let UseNamedOperandTable = true;
  }
```
**EN:** This block adjusts inherited fields or iterates over parameter sets to shape many related records consistently.

**CN:** 该区段调整继承字段或遍历参数集合，以一致地塑造多条相关记录。

### Lines 81-90: TableGen class SMTVMADotSlide<bits<2> / TableGen 类 SMTVMADotSlide<bits<2>
```tablegen

  // Sliding-window vector dot product format.
  class SMTVMADotSlide<bits<2> funct2, bits<2> sign, string opcodestr,
                       bit EarlyClobber = 0>
      : RVInst<(outs VRM2:$vd_wb), (ins VRM2:$vd, VRM2:$vs1, VR:$vs2),
               opcodestr, "$vd, $vs1, $vs2", [], InstFormatR> {
    bits<5> vd;
    bits<5> vs1;
    bits<5> vs2;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 91-101: TableGen field customization and iteration / TableGen 字段定制与迭代
```tablegen
    let Inst{31-25} = OPMMA_SLIDE.Value;
    let Inst{24-20} = vs2;
    let Inst{19-16} = vs1{4-1};
    let Inst{15-14} = funct2;
    let Inst{13-12} = sign;
    let Inst{11-7} = vd;
    let Inst{6-0} = OPC_CUSTOM_1.Value;
    let Constraints =
        !if(EarlyClobber, "@earlyclobber $vd_wb, $vd = $vd_wb", "$vd = $vd_wb");
    let UseNamedOperandTable = true;
  }
```
**EN:** This block adjusts inherited fields or iterates over parameter sets to shape many related records consistently.

**CN:** 该区段调整继承字段或遍历参数集合，以一致地塑造多条相关记录。

### Lines 102-103: Definitions and supporting logic / 定义与支撑逻辑
```tablegen
}
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

## Key Concepts / 关键概念
- **Instruction encodings** / **指令编码**
- **TableGen DSL** / **TableGen DSL**

## Dependencies / 依赖关系
- No explicit source includes in this file; dependencies are mostly implicit through surrounding backend integration. / 该文件没有显式源码级 include，依赖主要通过周边后端集成隐式体现。
