# PPCInstrFuture.td — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/PowerPC/PPCInstrFuture.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: PPCInstrFuture.td - Future Instruction Set. It is written in LLVM TableGen DSL and feeds generated backend tables.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/PowerPC/PPCInstrFuture.td`，主要负责 PowerPC 后端的该后端的 TableGen 声明。 它使用 LLVM TableGen DSL 编写，用于描述声明式目标信息并驱动生成代码。
- **Language**: TableGen DSL / TableGen 领域特定语言

## Line-by-Line Analysis / 逐行分析

### Lines 1-7

```tablegen
//===-- PPCInstrFuture.td - Future Instruction Set --------*- tablegen -*-===//
//
//                     The LLVM Compiler Infrastructure
//
// This file is distributed under the University of Illinois Open Source
// License. See LICENSE.TXT for details.
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 8-11

```tablegen
//===----------------------------------------------------------------------===//
//
// This file describes the instructions introduced for the Future CPU.
//
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "This file describes the instructions introduced for the Future CPU.".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“This file describes the instructions introduced for the Future CPU.”。

### Lines 12-19

```tablegen
//===----------------------------------------------------------------------===//

class XForm_RS5<bits<6> opcode, bits<10> xo, dag OOL, dag IOL, string asmstr,
                list<dag> pattern> : I<opcode, OOL, IOL, asmstr, NoItinerary> {
  bits<5> RS;

  let Pattern = pattern;
```
- **EN**: Declares a backend-facing type `XForm_RS5` and outlines the API or state that nearby code will rely on. These declarations feed generated pattern-matching logic.
- **CN**: 这里声明面向后端的类型 `XForm_RS5`，并勾勒出周边代码会依赖的接口或状态。 这些声明会进入生成式模式匹配逻辑。

### Lines 20-33

```tablegen
  let Inst{6...10} = RS;
  let Inst{11...20} = 0;
  let Inst{21...30} = xo;
  let Inst{31} = 0;
}

class XOForm_RTAB5_L1<bits<6> opcode, bits<9> xo, dag OOL, dag IOL,
                      string asmstr, list<dag> pattern>
    : I<opcode, OOL, IOL, asmstr, NoItinerary> {
  bits<5> RT;
  bits<5> RA;
  bits<5> RB;
  bit L;
```
- **EN**: Declares a backend-facing type `XOForm_RTAB5_L1` and outlines the API or state that nearby code will rely on. Scheduling or processor-model metadata is part of this section.
- **CN**: 这里声明面向后端的类型 `XOForm_RTAB5_L1`，并勾勒出周边代码会依赖的接口或状态。 这一段包含调度或处理器模型元数据。

### Lines 34-40

```tablegen
  let Pattern = pattern;

  bit RC = 0; // set by isRecordForm

  let Inst{6...10} = RT;
  let Inst{11...15} = RA;
  let Inst{16...20} = RB;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 41-47

```tablegen
  let Inst{21} = L;
  let Inst{22...30} = xo;
  let Inst{31} = RC;
}

multiclass XOForm_RTAB5_L1r<bits<6> opcode, bits<9> xo, dag OOL, dag IOL,
                            string asmbase, string asmstr, list<dag> pattern> {
```
- **EN**: Adds declarative TableGen records such as `XOForm_RTAB5_L1r` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `XOForm_RTAB5_L1r`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 48-59

```tablegen
  let BaseName = asmbase in {
    def NAME : XOForm_RTAB5_L1<opcode, xo, OOL, IOL,
                               !strconcat(asmbase, !strconcat(" ", asmstr)),
                               pattern>,
               RecFormRel;
    let Defs = [CR0] in def _rec
        : XOForm_RTAB5_L1<opcode, xo, OOL, IOL,
                          !strconcat(asmbase, !strconcat(". ", asmstr)), []>,
        isRecordForm, RecFormRel;
  }
}
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend TableGen declarations for the backend.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 TableGen 声明所需的辅助例程。

### Lines 60-67

```tablegen
class VXForm_VRTB5_Base<bits<11> xo, dag OOL, dag IOL, string asmstr,
                        list<dag> pattern>
    : I<4, OOL, IOL, asmstr, NoItinerary> {
  bits<5> VRT;
  bits<5> VRB;

  let Pattern = pattern;
```
- **EN**: Declares a backend-facing type `VXForm_VRTB5_Base` and outlines the API or state that nearby code will rely on. These declarations feed generated pattern-matching logic.
- **CN**: 这里声明面向后端的类型 `VXForm_VRTB5_Base`，并勾勒出周边代码会依赖的接口或状态。 这些声明会进入生成式模式匹配逻辑。

### Lines 68-76

```tablegen
  let Inst{6...10} = VRT;
  let Inst{16...20} = VRB;
  let Inst{21...31} = xo;
}

class VXForm_VRTB5<bits<11> xo, bits<5> R, dag OOL, dag IOL, string asmstr,
                   list<dag> pattern>
    : VXForm_VRTB5_Base<xo, OOL, IOL, asmstr, pattern> {
```
- **EN**: Declares a backend-facing type `VXForm_VRTB5` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `VXForm_VRTB5`，并勾勒出周边代码会依赖的接口或状态。

### Lines 77-84

```tablegen
  let Inst{11...15} = R;
}

class VXForm_VRTB5_UIM2<bits<11> xo, bits<3> R, dag OOL, dag IOL, string asmstr,
                        list<dag> pattern>
    : VXForm_VRTB5_Base<xo, OOL, IOL, asmstr, pattern> {
  bits<2> UIM;
```
- **EN**: Declares a backend-facing type `VXForm_VRTB5_UIM2` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `VXForm_VRTB5_UIM2`，并勾勒出周边代码会依赖的接口或状态。

### Lines 85-93

```tablegen
  let Inst{11...13} = R;
  let Inst{14...15} = UIM;
}

class VXForm_VRTB5_UIM1<bits<11> xo, bits<4> R, dag OOL, dag IOL, string asmstr,
                        list<dag> pattern>
    : VXForm_VRTB5_Base<xo, OOL, IOL, asmstr, pattern> {
  bits<1> UIM;
```
- **EN**: Declares a backend-facing type `VXForm_VRTB5_UIM1` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `VXForm_VRTB5_UIM1`，并勾勒出周边代码会依赖的接口或状态。

### Lines 94-102

```tablegen
  let Inst{11...14} = R;
  let Inst{15} = UIM;
}

class VXForm_VRTB5_UIM3<bits<11> xo, bits<2> R, dag OOL, dag IOL, string asmstr,
                        list<dag> pattern>
    : VXForm_VRTB5_Base<xo, OOL, IOL, asmstr, pattern> {
  bits<3> UIM;
```
- **EN**: Declares a backend-facing type `VXForm_VRTB5_UIM3` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `VXForm_VRTB5_UIM3`，并勾勒出周边代码会依赖的接口或状态。

### Lines 103-111

```tablegen
  let Inst{11...12} = R;
  let Inst{13...15} = UIM;
}

class VXForm_VRTAB5<bits<11> xo, dag OOL, dag IOL, string asmstr,
                    list<dag> pattern>
    : VXForm_VRTB5_Base<xo, OOL, IOL, asmstr, pattern> {
  bits<5> VRA;
```
- **EN**: Declares a backend-facing type `VXForm_VRTAB5` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `VXForm_VRTAB5`，并勾勒出周边代码会依赖的接口或状态。

### Lines 112-122

```tablegen
  let Inst{11...15} = VRA;
}

class XX3Form_XTBp5_M2<bits<9> xo, dag OOL, dag IOL, string asmstr,
                       list<dag> pattern>
    : I<60, OOL, IOL, asmstr, NoItinerary> {

  bits<5> XTp;
  bits<5> XBp;
  bits<2> M;
```
- **EN**: Declares a backend-facing type `XX3Form_XTBp5_M2` and outlines the API or state that nearby code will rely on. Scheduling or processor-model metadata is part of this section.
- **CN**: 这里声明面向后端的类型 `XX3Form_XTBp5_M2`，并勾勒出周边代码会依赖的接口或状态。 这一段包含调度或处理器模型元数据。

### Lines 123-129

```tablegen
  let Pattern = pattern;

  let Inst{6...9} = XTp{3...0};
  let Inst {10} = XTp{4};
  let Inst{15} = M{0};
  let Inst{16...19} = XBp{3...0};
  let Inst{20} = M{1};
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 130-142

```tablegen
  let Inst{21...29} = xo;
  let Inst{30} = XBp{4};
}

class XX3Form_XTABp5_M2<bits<8> xo, dag OOL, dag IOL, string asmstr,
                        list<dag> pattern>
    : I<60, OOL, IOL, asmstr, NoItinerary> {

  bits<5> XTp;
  bits<5> XAp;
  bits<5> XBp;
  bits<2> M;
```
- **EN**: Declares a backend-facing type `XX3Form_XTABp5_M2` and outlines the API or state that nearby code will rely on. Scheduling or processor-model metadata is part of this section.
- **CN**: 这里声明面向后端的类型 `XX3Form_XTABp5_M2`，并勾勒出周边代码会依赖的接口或状态。 这一段包含调度或处理器模型元数据。

### Lines 143-149

```tablegen
  let Pattern = pattern;

  let Inst{6...9} = XTp{3...0};
  let Inst{10} = XTp{4};
  let Inst{11...14} = XAp{3...0};
  let Inst{15} = M{0};
  let Inst{16...19} = XBp{3...0};
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 150-164

```tablegen
  let Inst{20} = M{1};
  let Inst{21...28} = xo;
  let Inst{29} = XAp{4};
  let Inst{30} = XBp{4};
}

class XX3Form_XTAB6_P1<bits<5> xo, dag OOL, dag IOL, string asmstr,
                       list<dag> pattern>
    : I<60, OOL, IOL, asmstr, NoItinerary> {

  bits<6> XT;
  bits<6> XA;
  bits<6> XB;
  bits<1> P;
```
- **EN**: Declares a backend-facing type `XX3Form_XTAB6_P1` and outlines the API or state that nearby code will rely on. Scheduling or processor-model metadata is part of this section.
- **CN**: 这里声明面向后端的类型 `XX3Form_XTAB6_P1`，并勾勒出周边代码会依赖的接口或状态。 这一段包含调度或处理器模型元数据。

### Lines 165-171

```tablegen
  let Pattern = pattern;

  let Inst{6...10} = XT{4...0};
  let Inst{11...15} = XA{4...0};
  let Inst{16...20} = XB{4...0};
  let Inst{21...22} = 3;
  let Inst{23} = P;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 172-185

```tablegen
  let Inst{24...28} = xo;
  let Inst{29} = XA{5};
  let Inst{30} = XB{5};
  let Inst{31} = XT{5};
}

class XX3Form_XTAB6<bits<6> opcode, bits<8> xo, dag OOL, dag IOL, string asmstr,
                    list<dag> pattern>
    : I<opcode, OOL, IOL, asmstr, NoItinerary> {

  bits<6> XT;
  bits<6> XA;
  bits<6> XB;
```
- **EN**: Declares a backend-facing type `XX3Form_XTAB6` and outlines the API or state that nearby code will rely on. Scheduling or processor-model metadata is part of this section.
- **CN**: 这里声明面向后端的类型 `XX3Form_XTAB6`，并勾勒出周边代码会依赖的接口或状态。 这一段包含调度或处理器模型元数据。

### Lines 186-192

```tablegen
  let Pattern = pattern;

  let Inst{6...10} = XT{4...0};
  let Inst{11...15} = XA{4...0};
  let Inst{16...20} = XB{4...0};
  let Inst{21...28} = xo;
  let Inst{29} = XA{5};
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 193-203

```tablegen
  let Inst{30} = XB{5};
  let Inst{31} = XT{5};
}

class XForm_RBS5<bits<6> opCode, bits<10> xo, dag OOL, dag IOL, string asmstr,
                 InstrItinClass itin, list<dag> pattern>
    : I<opCode, OOL, IOL, asmstr, itin> {

  bits<5> RB;
  bits<5> RS;
```
- **EN**: Declares a backend-facing type `XForm_RBS5` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XForm_RBS5`，并勾勒出周边代码会依赖的接口或状态。

### Lines 204-212

```tablegen
  let Pattern = pattern;

  let Inst{6...10} = RS;
  let Inst{11...15} = 0;
  let Inst{16...20} = RB;
  let Inst{21...30} = xo;
  let Inst{31} = 0;
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 213-219

```tablegen
class XX3Form_XTAB6_S<bits<5> xo, dag OOL, dag IOL, string asmstr,
                       list<dag> pattern>
    : I<59, OOL, IOL, asmstr, NoItinerary> {
  bits<6> XT;
  bits<6> XA;
  bits<6> XB;
```
- **EN**: Declares a backend-facing type `XX3Form_XTAB6_S` and outlines the API or state that nearby code will rely on. Scheduling or processor-model metadata is part of this section.
- **CN**: 这里声明面向后端的类型 `XX3Form_XTAB6_S`，并勾勒出周边代码会依赖的接口或状态。 这一段包含调度或处理器模型元数据。

### Lines 220-226

```tablegen
  let Pattern = pattern;

  let Inst{6...10} = XT{4...0};
  let Inst{11...15} = XA{4...0};
  let Inst{16...20} = XB{4...0};
  let Inst{24...28} = xo;
  let Inst{29} = XA{5};
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 227-235

```tablegen
  let Inst{30} = XB{5};
  let Inst{31} = XT{5};
}

class XX3Form_XTAB6_S3<bits<5> xo, dag OOL, dag IOL, string asmstr,
                       list<dag> pattern>
    : XX3Form_XTAB6_S<xo, OOL, IOL, asmstr, pattern> {

  bits<3> S;
```
- **EN**: Declares a backend-facing type `XX3Form_XTAB6_S3` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XX3Form_XTAB6_S3`，并勾勒出周边代码会依赖的接口或状态。

### Lines 236-246

```tablegen
  let Inst{21...23} = S;
}

class XX3Form_XTAB6_3S1<bits<5> xo, dag OOL, dag IOL, string asmstr,
                       list<dag> pattern>
    : XX3Form_XTAB6_S<xo, OOL, IOL, asmstr, pattern> {

  bits<1> S0;
  bits<1> S1;
  bits<1> S2;
```
- **EN**: Declares a backend-facing type `XX3Form_XTAB6_3S1` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XX3Form_XTAB6_3S1`，并勾勒出周边代码会依赖的接口或状态。

### Lines 247-258

```tablegen
  let Inst{21} = S0;
  let Inst{22} = S1;
  let Inst{23} = S2;
}

class XX3Form_XTAB6_2S1<bits<5> xo, dag OOL, dag IOL, string asmstr,
                       list<dag> pattern>
    : XX3Form_XTAB6_S<xo, OOL, IOL, asmstr, pattern> {

  bits<1> S1;
  bits<1> S2;
```
- **EN**: Declares a backend-facing type `XX3Form_XTAB6_2S1` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XX3Form_XTAB6_2S1`，并勾勒出周边代码会依赖的接口或状态。

### Lines 259-272

```tablegen
  let Inst{21} = 0;
  let Inst{22} = S1;
  let Inst{23} = S2;
}

class XX3Form_XTAB6_P<bits<7> xo, dag OOL, dag IOL, string asmstr,
                      list<dag> pattern>
    : I<59, OOL, IOL, asmstr, NoItinerary> {

  bits<6> XT;
  bits<6> XA;
  bits<6> XB;
  bits<1> P;
```
- **EN**: Declares a backend-facing type `XX3Form_XTAB6_P` and outlines the API or state that nearby code will rely on. Scheduling or processor-model metadata is part of this section.
- **CN**: 这里声明面向后端的类型 `XX3Form_XTAB6_P`，并勾勒出周边代码会依赖的接口或状态。 这一段包含调度或处理器模型元数据。

### Lines 273-279

```tablegen
  let Pattern = pattern;

  let Inst{6...10} = XT{4...0};
  let Inst{11...15} = XA{4...0};
  let Inst{16...20} = XB{4...0};
  let Inst{21} = P;
  let Inst{22...28} = xo;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 280-286

```tablegen
  let Inst{29} = XA{5};
  let Inst{30} = XB{5};
  let Inst{31} = XT{5};
}

// Prefix instruction classes.
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 287-295

```tablegen
class 8RR_XX4Form_XTABC6_P<bits<6> opcode, dag OOL, dag IOL, string asmstr,
                           InstrItinClass itin, list<dag> pattern>
    : PI<1, opcode, OOL, IOL, asmstr, itin> {
  bits<6> XT;
  bits<6> XA;
  bits<6> XB;
  bits<6> XC;
  bits<1> P;
```
- **EN**: Declares a backend-facing type and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型，并勾勒出周边代码会依赖的接口或状态。

### Lines 296-302

```tablegen
  let Pattern = pattern;

  // The prefix.
  let Inst{6...7} = 1;
  let Inst{8...11} = 0;

  // The instruction.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "The prefix.". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“The prefix.”。 这些声明会进入生成式模式匹配逻辑。

### Lines 303-309

```tablegen
  let Inst{38...42} = XT{4...0};
  let Inst{43...47} = XA{4...0};
  let Inst{48...52} = XB{4...0};
  let Inst{53...57} = XC{4...0};
  let Inst{58} = 1;
  let Inst{59} = P;
  let Inst{60} = XC{5};
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 310-321

```tablegen
  let Inst{61} = XA{5};
  let Inst{62} = XB{5};
  let Inst{63} = XT{5};
}

class MLS_DForm_R_SI32_RTA5<bits<6> opcode, dag OOL, dag IOL, string asmstr,
                            InstrItinClass itin, list<dag> pattern>
    : PI<1, opcode, OOL, IOL, asmstr, itin> {
  bits<5> RT;
  bits<5> RA;
  bits<32> SI;
```
- **EN**: Declares a backend-facing type `MLS_DForm_R_SI32_RTA5` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `MLS_DForm_R_SI32_RTA5`，并勾勒出周边代码会依赖的接口或状态。

### Lines 322-330

```tablegen
  let Pattern = pattern;

  // The prefix.
  let Inst{6...7} = 2;
  let Inst{8} = 0;
  let Inst{11} = PCRel;
  let Inst{16...31} = SI{31...16};

  // The instruction.
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 331-338

```tablegen
  let Inst{38...42} = RT;
  let Inst{43...47} = RA;
  let Inst{48...63} = SI{15...0};
}

multiclass MLS_DForm_R_SI32_RTA5_p<bits<6> opcode, dag OOL, dag IOL,
                                   dag PCRel_IOL, string asmstr,
                                   InstrItinClass itin> {
```
- **EN**: Adds declarative TableGen records such as `MLS_DForm_R_SI32_RTA5_p` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `MLS_DForm_R_SI32_RTA5_p`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 339-352

```tablegen
  def NAME : MLS_DForm_R_SI32_RTA5<opcode, OOL, IOL, !strconcat(asmstr, ", 0"),
                                   itin, []>;
  def pc : MLS_DForm_R_SI32_RTA5<opcode, OOL, PCRel_IOL,
                                 !strconcat(asmstr, ", 1"), itin, []>,
           isPCRel;
}

//-------------------------- Instruction definitions -------------------------//
// Predicate combinations available:
// [IsISAFuture]
// [IsISAFuture, PrefixInstrs]
// [HasVSX, IsISAFuture]
// [HasVSX, PrefixInstrs, IsISAFuture]
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "-------------------------- Instruction definitions -------------------------//". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“-------------------------- Instruction definitions -------------------------//”。 这些声明会进入生成式模式匹配逻辑。

### Lines 353-360

```tablegen
let Predicates = [IsISAFuture] in {
  defm SUBFUS : XOForm_RTAB5_L1r<31, 72, (outs g8rc:$RT),
                                 (ins g8rc:$RA, g8rc:$RB, u1imm:$L), "subfus",
                                 "$RT, $L, $RA, $RB", []>;
  def TLBSYNCIO
      : XForm_RS5<31, 564, (outs), (ins g8rc:$RS), "tlbsyncio $RS", []>;
  def PTESYNCIO
      : XForm_RS5<31, 596, (outs), (ins g8rc:$RS), "ptesyncio $RS", []>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 361-367

```tablegen
  def TLBIEP : XForm_RSB5_UIMM2_2UIMM1<31, 50, (outs),
                                       (ins gprc:$RB, gprc:$RS, u2imm:$RIC,
                                           u1imm:$PRS, u1imm:$R),
                                       "tlbiep $RB, $RS, $RIC, $PRS, $R", []>;
  def TLBIEIO
      : XForm_RSB5_UIMM2<31, 18, (outs), (ins g8rc:$RB, g8rc:$RS, u2imm:$RIC),
                         "tlbieio $RB, $RS, $RIC", []>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 368-375

```tablegen
  def MTLPL : XForm_RBS5<31, 275, (outs), (ins gprc:$RB, gprc:$RS),
                         "mtlpl $RB, $RS", IIC_SprMTSPR, []>;
  let Interpretation64Bit = 1, isCodeGenOnly = 1 in {
    def TLBIEP8
        : XForm_RSB5_UIMM2_2UIMM1<31, 50, (outs),
                                  (ins g8rc:$RB, g8rc:$RS, u2imm:$RIC,
                                      u1imm:$PRS, u1imm:$R),
                                  "tlbiep $RB, $RS, $RIC, $PRS, $R", []>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 376-385

```tablegen
    def MTLPL8 : XForm_RBS5<31, 275, (outs), (ins g8rc:$RB, g8rc:$RS),
                            "mtlpl $RB, $RS", IIC_SprMTSPR, []>, isPPC64;
  }
}

let Predicates = [IsISAFuture, PrefixInstrs] in {
  defm PADDIS : MLS_DForm_R_SI32_RTA5_p<15, (outs gprc:$RT),
                                        (ins gprc_nor0:$RA, s32imm:$SI),
                                        (ins immZero:$RA, s32imm_pcrel:$SI),
                                        "paddis $RT, $RA, $SI", IIC_LdStLFD>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 386-392

```tablegen
  let Interpretation64Bit = 1, isCodeGenOnly = 1 in
    defm PADDIS8 : MLS_DForm_R_SI32_RTA5_p<15, (outs g8rc:$RT),
                                           (ins g8rc_nox0:$RA, s32imm64:$SI),
                                           (ins immZero:$RA, s32imm64_pcrel:$SI),
                                           "paddis $RT, $RA, $SI", IIC_LdStLFD>;
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 393-400

```tablegen
let Predicates = [HasFutureVector] in {
  let mayLoad = 1 in {
    def LXVRL : XX1Form_memOp<31, 525, (outs vsrc:$XT),
                              (ins (memr $RA):$addr, g8rc:$RB),
                              "lxvrl $XT, $addr, $RB", IIC_LdStLoad, []>;
    def LXVRLL : XX1Form_memOp<31, 557, (outs vsrc:$XT),
                               (ins (memr $RA):$addr, g8rc:$RB),
                               "lxvrll $XT, $addr, $RB", IIC_LdStLoad, []>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 401-412

```tablegen
    def LXVPRL : XForm_XTp5_RAB5<31, 589, (outs vsrprc:$XTp),
                                 (ins (memr $RA):$addr, g8rc:$RB),
                                 "lxvprl $XTp, $addr, $RB", IIC_LdStLFD, []>;
    def LXVPRLL : XForm_XTp5_RAB5<31, 621, (outs vsrprc:$XTp),
                                  (ins (memr $RA):$addr, g8rc:$RB),
                                  "lxvprll $XTp, $addr, $RB", IIC_LdStLFD, []>;
    def LXVPB32X
        : XForm_XTp5_RAB5<31, 877, (outs vsrprc:$XTp),
                          (ins (memr $RA):$addr, g8rc:$RB),
                          "lxvpb32x $XTp, $addr, $RB", IIC_LdStLFD, []>;
  }
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 413-419

```tablegen
  let mayStore = 1 in {
    def STXVRL : XX1Form_memOp<31, 653, (outs),
                               (ins vsrc:$XT, (memr $RA):$addr, g8rc:$RB),
                               "stxvrl $XT, $addr, $RB", IIC_LdStLoad, []>;
    def STXVRLL : XX1Form_memOp<31, 685, (outs),
                                (ins vsrc:$XT, (memr $RA):$addr, g8rc:$RB),
                                "stxvrll $XT, $addr, $RB", IIC_LdStLoad, []>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 420-426

```tablegen
    def STXVPRL : XForm_XTp5_RAB5<31, 717, (outs),
                                  (ins vsrprc:$XTp, (memr $RA):$addr, g8rc:$RB),
                                  "stxvprl $XTp, $addr, $RB", IIC_LdStLFD, []>;
    def STXVPRLL
        : XForm_XTp5_RAB5<31, 749, (outs),
                          (ins vsrprc:$XTp, (memr $RA):$addr, g8rc:$RB),
                          "stxvprll $XTp, $addr, $RB", IIC_LdStLFD, []>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 427-436

```tablegen
    def STXVPB32X
        : XForm_XTp5_RAB5<31, 1005, (outs),
                          (ins vsrprc:$XTp, (memr $RA):$addr, g8rc:$RB),
                          "stxvpb32x $XTp, $addr, $RB", IIC_LdStLFD, []>;
  }

  def VUPKHSNTOB : VXForm_VRTB5<387, 0, (outs vrrc:$VRT), (ins vrrc:$VRB),
                                "vupkhsntob $VRT, $VRB",
                                [(set v16i8:$VRT,
                                  (int_ppc_altivec_vupkhsntob v16i8:$VRB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 437-445

```tablegen
  def VUPKLSNTOB : VXForm_VRTB5<387, 1, (outs vrrc:$VRT), (ins vrrc:$VRB),
                                "vupklsntob $VRT, $VRB",
                                [(set v16i8:$VRT,
                                  (int_ppc_altivec_vupklsntob v16i8:$VRB))]>;
  def VUPKINT4TOBF16
      : VXForm_VRTB5_UIM2<387, 2, (outs vrrc:$VRT), (ins vrrc:$VRB, u2imm:$UIM),
                          "vupkint4tobf16 $VRT, $VRB, $UIM",
                          [(set v16i8:$VRT,
                            (int_ppc_altivec_vupkint4tobf16 v16i8:$VRB, u2imm_timm:$UIM))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 446-455

```tablegen
  def VUPKINT8TOBF16
      : VXForm_VRTB5_UIM1<387, 1, (outs vrrc:$VRT), (ins vrrc:$VRB, u1imm:$UIM),
                          "vupkint8tobf16 $VRT, $VRB, $UIM",
                          [(set v16i8:$VRT,
                            (int_ppc_altivec_vupkint8tobf16 v16i8:$VRB, u1imm_timm:$UIM))]>;
  def VUPKINT8TOFP32
      : VXForm_VRTB5_UIM2<387, 3, (outs vrrc:$VRT), (ins vrrc:$VRB, u2imm:$UIM),
                          "vupkint8tofp32 $VRT, $VRB, $UIM",
                          [(set v16i8:$VRT,
                            (int_ppc_altivec_vupkint8tofp32 v16i8:$VRB, u2imm_timm:$UIM))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 456-465

```tablegen
  def VUPKINT4TOFP32
      : VXForm_VRTB5_UIM3<387, 2, (outs vrrc:$VRT), (ins vrrc:$VRB, u3imm:$UIM),
                          "vupkint4tofp32 $VRT, $VRB, $UIM",
                          [(set v16i8:$VRT,
                            (int_ppc_altivec_vupkint4tofp32 v16i8:$VRB, u3imm_timm:$UIM))]>;

  def VUCMPRHN : VXForm_VRTAB5<3, (outs vrrc:$VRT), (ins vrrc:$VRA, vrrc:$VRB),
                               "vucmprhn $VRT, $VRA, $VRB",
                               [(set v16i8:$VRT,
                                 (int_ppc_altivec_vucmprhn v16i8:$VRA, v16i8:$VRB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 466-474

```tablegen
  def VUCMPRLN : VXForm_VRTAB5<67, (outs vrrc:$VRT), (ins vrrc:$VRA, vrrc:$VRB),
                               "vucmprln $VRT, $VRA, $VRB",
                               [(set v16i8:$VRT,
                                 (int_ppc_altivec_vucmprln v16i8:$VRA, v16i8:$VRB))]>;
  def VUCMPRHB
      : VXForm_VRTAB5<131, (outs vrrc:$VRT), (ins vrrc:$VRA, vrrc:$VRB),
                      "vucmprhb $VRT, $VRA, $VRB",
                      [(set v16i8:$VRT,
                        (int_ppc_altivec_vucmprhb v16i8:$VRA, v16i8:$VRB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 475-484

```tablegen
  def VUCMPRLB
      : VXForm_VRTAB5<195, (outs vrrc:$VRT), (ins vrrc:$VRA, vrrc:$VRB),
                      "vucmprlb $VRT, $VRA, $VRB",
                      [(set v16i8:$VRT,
                        (int_ppc_altivec_vucmprlb v16i8:$VRA, v16i8:$VRB))]>;
  def VUCMPRHH
      : VXForm_VRTAB5<259, (outs vrrc:$VRT), (ins vrrc:$VRA, vrrc:$VRB),
                      "vucmprhh $VRT, $VRA, $VRB",
                      [(set v16i8:$VRT,
                        (int_ppc_altivec_vucmprhh v16i8:$VRA, v16i8:$VRB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 485-497

```tablegen
  def VUCMPRLH
      : VXForm_VRTAB5<323, (outs vrrc:$VRT), (ins vrrc:$VRA, vrrc:$VRB),
                      "vucmprlh $VRT, $VRA, $VRB",
                      [(set v16i8:$VRT,
                        (int_ppc_altivec_vucmprlh v16i8:$VRA, v16i8:$VRB))]>;

  def XVRLW : XX3Form_XTAB6<60, 184, (outs vsrc:$XT), (ins vsrc:$XA, vsrc:$XB),
                            "xvrlw $XT, $XA, $XB",
                            [(set v4i32:$XT, (int_ppc_vsx_xvrlw v4i32:$XA,
                                                 v4i32:$XB))]>;
}

  // AES Acceleration Instructions
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 498-504

```tablegen
let Predicates = [HasFutureVector, PairedVectorMemops] in {
  def XXAESENCP
      : XX3Form_XTABp5_M2<194, (outs vsrprc:$XTp),
                          (ins vsrprc:$XAp, vsrprc:$XBp, u2imm:$M),
                          "xxaesencp $XTp, $XAp, $XBp, $M",
                          [(set v256i1:$XTp,
                                (int_ppc_aes_encrypt_paired v256i1:$XAp, v256i1:$XBp, u2imm_timm:$M))]>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 505-517

```tablegen
  def XXAESDECP
      : XX3Form_XTABp5_M2<202, (outs vsrprc:$XTp),
                          (ins vsrprc:$XAp, vsrprc:$XBp, u2imm:$M),
                          "xxaesdecp $XTp, $XAp, $XBp, $M",
                          [(set v256i1:$XTp,
                                (int_ppc_aes_decrypt_paired v256i1:$XAp, v256i1:$XBp, u2imm_timm:$M))]>;
  def XXAESGENLKP
      : XX3Form_XTBp5_M2<420, (outs vsrprc:$XTp), (ins vsrprc:$XBp, u2imm:$M),
                         "xxaesgenlkp $XTp, $XBp, $M",
                         [(set v256i1:$XTp,
                               (int_ppc_aes_genlastkey_paired v256i1:$XBp, u2imm_timm:$M))]>;
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 518-525

```tablegen
let Predicates = [HasFutureVector] in {
  def XXGFMUL128 : XX3Form_XTAB6_P1<26, (outs vsrc:$XT),
                                    (ins vsrc:$XA, vsrc:$XB, u1imm:$P),
                                    "xxgfmul128 $XT, $XA, $XB, $P", []>;

  // VSX Vector Integer Arithmetic Instructions
  def XVADDUWM : XX3Form_XTAB6<60, 131, (outs vsrc:$XT), (ins vsrc:$XA, vsrc:$XB),
                               "xvadduwm $XT, $XA, $XB", []>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 526-533

```tablegen
  def XVADDUHM : XX3Form_XTAB6<60, 139, (outs vsrc:$XT), (ins vsrc:$XA, vsrc:$XB),
                               "xvadduhm $XT, $XA, $XB", []>;
  def XVSUBUWM: XX3Form_XTAB6<60, 147, (outs vsrc:$XT), (ins vsrc:$XA, vsrc:$XB),
                              "xvsubuwm $XT, $XA, $XB", []>;
  def XVSUBUHM: XX3Form_XTAB6<60, 155, (outs vsrc:$XT), (ins vsrc:$XA, vsrc:$XB),
                              "xvsubuhm $XT, $XA, $XB", []>;
  def XVMULUWM: XX3Form_XTAB6<60, 163, (outs vsrc:$XT), (ins vsrc:$XA, vsrc:$XB),
                              "xvmuluwm $XT, $XA, $XB", []>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 534-541

```tablegen
  def XVMULUHM: XX3Form_XTAB6<60, 171, (outs vsrc:$XT), (ins vsrc:$XA, vsrc:$XB),
                              "xvmuluhm $XT, $XA, $XB", []>;
  def XVMULHSW: XX3Form_XTAB6<60, 179, (outs vsrc:$XT), (ins vsrc:$XA, vsrc:$XB),
                              "xvmulhsw $XT, $XA, $XB", []>;
  def XVMULHSH: XX3Form_XTAB6<60, 187, (outs vsrc:$XT), (ins vsrc:$XA, vsrc:$XB),
                              "xvmulhsh $XT, $XA, $XB", []>;
  def XVMULHUW: XX3Form_XTAB6<60, 114, (outs vsrc:$XT), (ins vsrc:$XA, vsrc:$XB),
                              "xvmulhuw $XT, $XA, $XB", []>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 542-548

```tablegen
  def XVMULHUH: XX3Form_XTAB6<60, 122, (outs vsrc:$XT), (ins vsrc:$XA, vsrc:$XB),
                              "xvmulhuh $XT, $XA, $XB", []>;

  // Elliptic Curve Cryptography Acceleration Instructions.
  def XXMULMUL
      : XX3Form_XTAB6_S3<1, (outs vsrc:$XT), (ins vsrc:$XA, vsrc:$XB, u3imm:$S),
                         "xxmulmul $XT, $XA, $XB, $S", []>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 549-557

```tablegen
  def XXMULMULHIADD
      : XX3Form_XTAB6_3S1<9, (outs vsrc:$XT),
                          (ins vsrc:$XA, vsrc:$XB, u1imm:$S0, u1imm:$S1,
                              u1imm:$S2),
                          "xxmulmulhiadd $XT, $XA, $XB, $S0, $S1, $S2", []>;
  def XXMULMULLOADD
      : XX3Form_XTAB6_2S1<17, (outs vsrc:$XT),
                          (ins vsrc:$XA, vsrc:$XB, u1imm:$S1, u1imm:$S2),
                          "xxmulmulloadd $XT, $XA, $XB, $S1, $S2", []>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 558-566

```tablegen
  def XXSSUMUDM
      : XX3Form_XTAB6_P<25, (outs vsrc:$XT), (ins vsrc:$XA, vsrc:$XB, u1imm:$P),
                        "xxssumudm $XT, $XA, $XB, $P", []>;
  def XXSSUMUDMC
      : XX3Form_XTAB6_P<57, (outs vsrc:$XT), (ins vsrc:$XA, vsrc:$XB, u1imm:$P),
                        "xxssumudmc $XT, $XA, $XB, $P", []>;
  def XSADDADDUQM
      : XX3Form_XTAB6<59, 96, (outs vsrc:$XT), (ins vsrc:$XA, vsrc:$XB),
                      "xsaddadduqm $XT, $XA, $XB", []>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 567-575

```tablegen
  def XSADDADDSUQM
      : XX3Form_XTAB6<59, 104, (outs vsrc:$XT), (ins vsrc:$XA, vsrc:$XB),
                      "xsaddaddsuqm $XT, $XA, $XB", []>;
  def XSADDSUBUQM
      : XX3Form_XTAB6<59, 112, (outs vsrc:$XT), (ins vsrc:$XA, vsrc:$XB),
                      "xsaddsubuqm $XT, $XA, $XB", []>;
  def XSADDSUBSUQM
      : XX3Form_XTAB6<59, 224, (outs vsrc:$XT), (ins vsrc:$XA, vsrc:$XB),
                      "xsaddsubsuqm $XT, $XA, $XB", []>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 576-584

```tablegen
  def XSMERGE2T1UQM
      : XX3Form_XTAB6<59, 232, (outs vsrc:$XT), (ins vsrc:$XA, vsrc:$XB),
                      "xsmerge2t1uqm $XT, $XA, $XB", []>;
  def XSMERGE2T2UQM
      : XX3Form_XTAB6<59, 240, (outs vsrc:$XT), (ins vsrc:$XA, vsrc:$XB),
                      "xsmerge2t2uqm $XT, $XA, $XB", []>;
  def XSMERGE2T3UQM
      : XX3Form_XTAB6<59, 89, (outs vsrc:$XT), (ins vsrc:$XA, vsrc:$XB),
                      "xsmerge2t3uqm $XT, $XA, $XB", []>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 585-593

```tablegen
  def XSMERGE3T1UQM
      : XX3Form_XTAB6<59, 121, (outs vsrc:$XT), (ins vsrc:$XA, vsrc:$XB),
                      "xsmerge3t1uqm $XT, $XA, $XB", []>;
  def XSREBASE2T1UQM
      : XX3Form_XTAB6<59, 145, (outs vsrc:$XT), (ins vsrc:$XA, vsrc:$XB),
                      "xsrebase2t1uqm $XT, $XA, $XB", []>;
  def XSREBASE2T2UQM
      : XX3Form_XTAB6<59, 177, (outs vsrc:$XT), (ins vsrc:$XA, vsrc:$XB),
                      "xsrebase2t2uqm $XT, $XA, $XB", []>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 594-602

```tablegen
  def XSREBASE2T3UQM
      : XX3Form_XTAB6<59, 209, (outs vsrc:$XT), (ins vsrc:$XA, vsrc:$XB),
                      "xsrebase2t3uqm $XT, $XA, $XB", []>;
  def XSREBASE2T4UQM
      : XX3Form_XTAB6<59, 217, (outs vsrc:$XT), (ins vsrc:$XA, vsrc:$XB),
                      "xsrebase2t4uqm $XT, $XA, $XB", []>;
  def XSREBASE3T1UQM
      : XX3Form_XTAB6<59, 241, (outs vsrc:$XT), (ins vsrc:$XA, vsrc:$XB),
                      "xsrebase3t1uqm $XT, $XA, $XB", []>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 603-610

```tablegen
  def XSREBASE3T2UQM
      : XX3Form_XTAB6<59, 249, (outs vsrc:$XT), (ins vsrc:$XA, vsrc:$XB),
                      "xsrebase3t2uqm $XT, $XA, $XB", []>;
  def XSREBASE3T3UQM
      : XX3Form_XTAB6<59, 195, (outs vsrc:$XT), (ins vsrc:$XA, vsrc:$XB),
                      "xsrebase3t3uqm $XT, $XA, $XB", []>;
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 611-621

```tablegen
let Predicates = [HasFutureVector, PrefixInstrs] in {
  def XXSSUMUDMCEXT
      : 8RR_XX4Form_XTABC6_P<
            34, (outs vsrc:$XT), (ins vsrc:$XA, vsrc:$XB, vsrc:$XC, u1imm:$P),
            "xxssumudmcext $XT, $XA, $XB, $XC, $P", IIC_VecGeneral, []>;
}

//---------------------------- Anonymous Patterns ----------------------------//
// Predicate combinations available:

// Load/Store VSX Vector with Right Length (Left-justified).
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "---------------------------- Anonymous Patterns ----------------------------//". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“---------------------------- Anonymous Patterns ----------------------------//”。 这些声明会进入生成式模式匹配逻辑。

### Lines 622-629

```tablegen
def : Pat<(v4i32 (int_ppc_vsx_lxvrl addr:$RA, i64:$RB)), (LXVRL $RA, $RB)>;
def : Pat<(v4i32 (int_ppc_vsx_lxvrll addr:$RA, i64:$RB)), (LXVRLL $RA, $RB)>;
def : Pat<(int_ppc_vsx_stxvrl v4i32:$XT, addr:$RA, i64:$RB), (STXVRL $XT, $RA,
                                                                 $RB)>;
def : Pat<(int_ppc_vsx_stxvrll v4i32:$XT, addr:$RA, i64:$RB), (STXVRLL $XT, $RA,
                                                                  $RB)>;

// Load/Store VSX Vector pair with Right Length (Left-justified).
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 630-636

```tablegen
def : Pat<(v256i1 (int_ppc_vsx_lxvprl addr:$RA, i64:$RB)), (LXVPRL $RA, $RB)>;
def : Pat<(v256i1 (int_ppc_vsx_lxvprll addr:$RA, i64:$RB)), (LXVPRLL $RA, $RB)>;
def : Pat<(int_ppc_vsx_stxvprl v256i1:$XTp, addr:$RA, i64:$RB), (STXVPRL $XTp,
                                                                    $RA, $RB)>;
def : Pat<(int_ppc_vsx_stxvprll v256i1:$XTp, addr:$RA, i64:$RB), (STXVPRLL $XTp,
                                                                     $RA, $RB)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 637-643

```tablegen
def: Pat<(v16i8 (int_ppc_galois_field_mult v16i8:$XA, v16i8:$XB, u1imm_timm:$IMM)),
         (COPY_TO_REGCLASS (XXGFMUL128 RCCp.AToVSRC, RCCp.BToVSRC, $IMM), VSRC)>;

// Regular load/store patterns for v256i1 (for ISA Future)
let Predicates = [HasFutureVector, PairedVectorMemops] in {
  def : Pat<(v256i1 (load iaddrX16:$src)), (LXVP iaddrX16:$src)>;
  def : Pat<(v256i1 (load PDForm:$src)), (PLXVP memri34:$src)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 644-650

```tablegen
  def : Pat<(v256i1 (load xoaddr:$src)), (LXVPX xoaddr:$src)>;
  def : Pat<(store v256i1:$XSp, iaddrX16:$dst), (STXVP $XSp, iaddrX16:$dst)>;
  def : Pat<(store v256i1:$XSp, PDForm:$dst), (PSTXVP $XSp, memri34:$dst)>;
  def : Pat<(store v256i1:$XSp, xoaddr:$dst), (STXVPX $XSp, xoaddr:$dst)>;
}

let Predicates = [HasFutureVector] in {
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 651-658

```tablegen
  def : Pat<(v4i32 (rotl v4i32:$vA, v4i32:$vB)), (v4i32 (XVRLW v4i32:$vA,
                                                     v4i32:$vB))>;
  def : Pat<(v4i32 (int_ppc_altivec_vrlw v4i32:$vA, v4i32:$vB)),
            (v4i32 (XVRLW v4i32:$vA, v4i32:$vB))>;

  // Elliptic Curve Cryptography Patterns
  def : Pat<(v16i8 (int_ppc_xxmulmul v16i8:$XA, v16i8:$XB, u3imm_timm:$S)),
            (v16i8 (COPY_TO_REGCLASS (XXMULMUL RCCp.AToVSRC, RCCp.BToVSRC, $S), VSRC))>;
```
- **EN**: Declares function entry points that other backend components call later. These declarations feed generated pattern-matching logic.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。 这些声明会进入生成式模式匹配逻辑。

### Lines 659-666

```tablegen
  def : Pat<(v16i8 (int_ppc_xxmulmulhiadd v16i8:$XA, v16i8:$XB, u1imm_timm:$S0, u1imm_timm:$S1, u1imm_timm:$S2)),
            (v16i8 (COPY_TO_REGCLASS (XXMULMULHIADD RCCp.AToVSRC, RCCp.BToVSRC, $S0, $S1, $S2), VSRC))>;
  def : Pat<(v16i8 (int_ppc_xxmulmulloadd v16i8:$XA, v16i8:$XB, u1imm_timm:$S1, u1imm_timm:$S2)),
            (v16i8 (COPY_TO_REGCLASS (XXMULMULLOADD RCCp.AToVSRC, RCCp.BToVSRC, $S1, $S2), VSRC))>;
  def : Pat<(v16i8 (int_ppc_xxssumudm v16i8:$XA, v16i8:$XB, u1imm_timm:$P)),
            (v16i8 (COPY_TO_REGCLASS (XXSSUMUDM RCCp.AToVSRC, RCCp.BToVSRC, $P), VSRC))>;
  def : Pat<(v16i8 (int_ppc_xxssumudmc v16i8:$XA, v16i8:$XB, u1imm_timm:$P)),
            (v16i8 (COPY_TO_REGCLASS (XXSSUMUDMC RCCp.AToVSRC, RCCp.BToVSRC, $P), VSRC))>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 667-674

```tablegen
  def : Pat<(v16i8 (int_ppc_xxssumudmcext v16i8:$XA, v16i8:$XB, v16i8:$XC, u1imm_timm:$P)),
            (v16i8 (COPY_TO_REGCLASS (XXSSUMUDMCEXT RCCp.AToVSRC, RCCp.BToVSRC, RCCp.CToVSRC, $P), VSRC))>;
  def : Pat<(v16i8 (int_ppc_xsaddadduqm v16i8:$XA, v16i8:$XB)),
            (v16i8 (COPY_TO_REGCLASS (XSADDADDUQM RCCp.AToVSRC, RCCp.BToVSRC), VSRC))>;
  def : Pat<(v16i8 (int_ppc_xsaddaddsuqm v16i8:$XA, v16i8:$XB)),
            (v16i8 (COPY_TO_REGCLASS (XSADDADDSUQM RCCp.AToVSRC, RCCp.BToVSRC), VSRC))>;
  def : Pat<(v16i8 (int_ppc_xsaddsubuqm v16i8:$XA, v16i8:$XB)),
            (v16i8 (COPY_TO_REGCLASS (XSADDSUBUQM RCCp.AToVSRC, RCCp.BToVSRC), VSRC))>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 675-682

```tablegen
  def : Pat<(v16i8 (int_ppc_xsaddsubsuqm v16i8:$XA, v16i8:$XB)),
            (v16i8 (COPY_TO_REGCLASS (XSADDSUBSUQM RCCp.AToVSRC, RCCp.BToVSRC), VSRC))>;
  def : Pat<(v16i8 (int_ppc_xsmerge2t1uqm v16i8:$XA, v16i8:$XB)),
            (v16i8 (COPY_TO_REGCLASS (XSMERGE2T1UQM RCCp.AToVSRC, RCCp.BToVSRC), VSRC))>;
  def : Pat<(v16i8 (int_ppc_xsmerge2t2uqm v16i8:$XA, v16i8:$XB)),
            (v16i8 (COPY_TO_REGCLASS (XSMERGE2T2UQM RCCp.AToVSRC, RCCp.BToVSRC), VSRC))>;
  def : Pat<(v16i8 (int_ppc_xsmerge2t3uqm v16i8:$XA, v16i8:$XB)),
            (v16i8 (COPY_TO_REGCLASS (XSMERGE2T3UQM RCCp.AToVSRC, RCCp.BToVSRC), VSRC))>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 683-690

```tablegen
  def : Pat<(v16i8 (int_ppc_xsmerge3t1uqm v16i8:$XA, v16i8:$XB)),
            (v16i8 (COPY_TO_REGCLASS (XSMERGE3T1UQM RCCp.AToVSRC, RCCp.BToVSRC), VSRC))>;
  def : Pat<(v16i8 (int_ppc_xsrebase2t1uqm v16i8:$XA, v16i8:$XB)),
            (v16i8 (COPY_TO_REGCLASS (XSREBASE2T1UQM RCCp.AToVSRC, RCCp.BToVSRC), VSRC))>;
  def : Pat<(v16i8 (int_ppc_xsrebase2t2uqm v16i8:$XA, v16i8:$XB)),
            (v16i8 (COPY_TO_REGCLASS (XSREBASE2T2UQM RCCp.AToVSRC, RCCp.BToVSRC), VSRC))>;
  def : Pat<(v16i8 (int_ppc_xsrebase2t3uqm v16i8:$XA, v16i8:$XB)),
            (v16i8 (COPY_TO_REGCLASS (XSREBASE2T3UQM RCCp.AToVSRC, RCCp.BToVSRC), VSRC))>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 691-702

```tablegen
  def : Pat<(v16i8 (int_ppc_xsrebase2t4uqm v16i8:$XA, v16i8:$XB)),
            (v16i8 (COPY_TO_REGCLASS (XSREBASE2T4UQM RCCp.AToVSRC, RCCp.BToVSRC), VSRC))>;
  def : Pat<(v16i8 (int_ppc_xsrebase3t1uqm v16i8:$XA, v16i8:$XB)),
            (v16i8 (COPY_TO_REGCLASS (XSREBASE3T1UQM RCCp.AToVSRC, RCCp.BToVSRC), VSRC))>;
  def : Pat<(v16i8 (int_ppc_xsrebase3t2uqm v16i8:$XA, v16i8:$XB)),
            (v16i8 (COPY_TO_REGCLASS (XSREBASE3T2UQM RCCp.AToVSRC, RCCp.BToVSRC), VSRC))>;
  def : Pat<(v16i8 (int_ppc_xsrebase3t3uqm v16i8:$XA, v16i8:$XB)),
            (v16i8 (COPY_TO_REGCLASS (XSREBASE3T3UQM RCCp.AToVSRC, RCCp.BToVSRC), VSRC))>;
}

// Post Quantum Cryptography Acceleration patterns.
// Use AddedComplexity to prefer these patterns over AltiVec patterns.
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 703-709

```tablegen
let Predicates = [HasFutureVector], AddedComplexity = 400 in {
  // Vector add
  def : Pat<(v4i32 (add v4i32:$XA, v4i32:$XB)), (v4i32 (XVADDUWM $XA, $XB))>;
  def : Pat<(v8i16 (add v8i16:$XA, v8i16:$XB)),
            (COPY_TO_REGCLASS (XVADDUHM RCCp.AToVSRC, RCCp.BToVSRC), VSRC)>;
  // Vector subtract
  // Don't have a VSX negate instruction so use VNEGW instead.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Vector add". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Vector add”。 这些声明会进入生成式模式匹配逻辑。

### Lines 710-718

```tablegen
  def : Pat<(v4i32 (sub (v4i32 immAllZerosV), v4i32:$vB)), (v4i32 (VNEGW $vB))>;
  def : Pat<(v4i32 (sub v4i32:$XA, v4i32:$XB)), (v4i32 (XVSUBUWM $XA, $XB))>;
  def : Pat<(v8i16 (sub v8i16:$XA, v8i16:$XB)),
            (COPY_TO_REGCLASS (XVSUBUHM RCCp.AToVSRC, RCCp.BToVSRC), VSRC)>;
  // Vector multiply
  def : Pat<(v4i32 (mul v4i32:$XA, v4i32:$XB)), (v4i32 (XVMULUWM $XA, $XB))>;
  def : Pat<(v8i16 (mul v8i16:$XA, v8i16:$XB)),
            (COPY_TO_REGCLASS (XVMULUHM RCCp.AToVSRC, RCCp.BToVSRC), VSRC)>;
  // Vector multiply high intrinsics
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 719-732

```tablegen
  def : Pat<(v4i32 (int_ppc_altivec_vmulhsw v4i32:$XA, v4i32:$XB)),
            (v4i32 (XVMULHSW $XA, $XB))>;
  def : Pat<(v4i32 (int_ppc_altivec_vmulhuw v4i32:$XA, v4i32:$XB)),
            (v4i32 (XVMULHUW $XA, $XB))>;
  def : Pat<(v8i16 (int_ppc_altivec_vmulhsh v8i16:$XA, v8i16:$XB)),
            (COPY_TO_REGCLASS (XVMULHSH RCCp.AToVSRC, RCCp.BToVSRC), VSRC)>;
  def : Pat<(v8i16 (int_ppc_altivec_vmulhuh v8i16:$XA, v8i16:$XB)),
            (COPY_TO_REGCLASS (XVMULHUH RCCp.AToVSRC, RCCp.BToVSRC), VSRC)>;
}

//---------------------------- Instruction aliases ---------------------------//
// Predicate combinations available:
// [HasVSX, IsISAFuture]
```
- **EN**: Declares function entry points that other backend components call later. These declarations feed generated pattern-matching logic.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。 这些声明会进入生成式模式匹配逻辑。

### Lines 733-740

```tablegen
def : InstAlias<"paddis $RT, $RA, $SI", (PADDIS gprc:$RT, gprc_nor0:$RA,
                                            s32imm:$SI)>;
def : InstAlias<"paddis $RT, $RA, $SI", (PADDIS8 g8rc:$RT, g8rc_nox0:$RA,
                                            s32imm64:$SI)>;
def : InstAlias<"plis $RT, $SI", (PADDIS gprc:$RT, ZERO, s32imm:$SI)>;
def : InstAlias<"plis $RT, $SI", (PADDIS8 g8rc:$RT, ZERO8, s32imm64:$SI)>;
// psubis: subtract immediate from register
// For assembly: psubis $RT, $RA, $SI -> paddis $RT, $RA, -$SI, 0
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "psubis: subtract immediate from register".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“psubis: subtract immediate from register”。

### Lines 741-748

```tablegen
def : InstAlias<"psubis $RT, $RA, $SI", (PADDIS gprc:$RT, gprc_nor0:$RA,
                                            neg_s32imm:$SI)>;
def : InstAlias<"psubis $RT, $RA, $SI", (PADDIS8 g8rc:$RT, g8rc_nox0:$RA,
                                            neg_s32imm64:$SI)>;

let Predicates = [HasFutureVector] in {
  def : InstAlias<"xxaes128encp $XTp, $XAp, $XBp",
                  (XXAESENCP vsrprc:$XTp, vsrprc:$XAp, vsrprc:$XBp, 0)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 749-756

```tablegen
  def : InstAlias<"xxaes192encp $XTp, $XAp, $XBp",
                  (XXAESENCP vsrprc:$XTp, vsrprc:$XAp, vsrprc:$XBp, 1)>;
  def : InstAlias<"xxaes256encp $XTp, $XAp, $XBp",
                  (XXAESENCP vsrprc:$XTp, vsrprc:$XAp, vsrprc:$XBp, 2)>;
  def : InstAlias<"xxaes128decp $XTp, $XAp, $XBp",
                  (XXAESDECP vsrprc:$XTp, vsrprc:$XAp, vsrprc:$XBp, 0)>;
  def : InstAlias<"xxaes192decp $XTp, $XAp, $XBp",
                  (XXAESDECP vsrprc:$XTp, vsrprc:$XAp, vsrprc:$XBp, 1)>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 757-764

```tablegen
  def : InstAlias<"xxaes256decp $XTp, $XAp, $XBp",
                  (XXAESDECP vsrprc:$XTp, vsrprc:$XAp, vsrprc:$XBp, 2)>;
  def : InstAlias<"xxaes128genlkp $XTp, $XBp", (XXAESGENLKP vsrprc:$XTp,
                                                   vsrprc:$XBp, 0)>;
  def : InstAlias<"xxaes192genlkp $XTp, $XBp", (XXAESGENLKP vsrprc:$XTp,
                                                   vsrprc:$XBp, 1)>;
  def : InstAlias<"xxaes256genlkp $XTp, $XBp", (XXAESGENLKP vsrprc:$XTp,
                                                   vsrprc:$XBp, 2)>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 765-769

```tablegen
  def : InstAlias<"xxgfmul128gcm $XT, $XA, $XB", (XXGFMUL128 vsrc:$XT, vsrc:$XA,
                                                     vsrc:$XB, 0)>;
  def : InstAlias<"xxgfmul128xts $XT, $XA, $XB", (XXGFMUL128 vsrc:$XT, vsrc:$XA,
                                                     vsrc:$XB, 1)>;
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

## Key Concepts / 关键概念

- PowerPC backend integration / PowerPC 后端集成
- TableGen declarations for the backend / 该后端的 TableGen 声明
- LLVM TableGen DSL / LLVM TableGen 领域特定语言
- Declarative TableGen records / 声明式 TableGen 记录
- Vector or SIMD behavior / 向量或 SIMD 行为

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- None explicitly included in the source text / 源码文本中没有显式包含项

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- Generated backend tables / 生成式后端表
- PowerPC target-specific helpers / PowerPC 目标专用辅助组件
