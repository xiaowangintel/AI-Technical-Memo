# PPCInstrFutureMMA.td — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/PowerPC/PPCInstrFutureMMA.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: PPCInstrFutureMMA.td - Future Instruction Set. It is written in LLVM TableGen DSL and feeds generated backend tables.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/PowerPC/PPCInstrFutureMMA.td`，主要负责 PowerPC 后端的该后端的 TableGen 声明。 它使用 LLVM TableGen DSL 编写，用于描述声明式目标信息并驱动生成代码。
- **Language**: TableGen DSL / TableGen 领域特定语言

## Line-by-Line Analysis / 逐行分析

### Lines 1-7

```tablegen
//===-- PPCInstrFutureMMA.td - Future Instruction Set ------*- tablegen -*-===//
//
//                     The LLVM Compiler Infrastructure
//
// This file is distributed under the University of Illinois Open Source
// License. See LICENSE.TXT for details.
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 8-12

```tablegen
//===----------------------------------------------------------------------===//
//
// This file describes the instructions introduced for the Future CPU for MMA.
// Please reference "PPCInstrVSX.td" for file structure.
//
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "This file describes the instructions introduced for the Future CPU for MMA.".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“This file describes the instructions introduced for the Future CPU for MMA.”。

### Lines 13-14

```tablegen
//===----------------------------------------------------------------------===//
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 15-21

```tablegen
//===----------------------------------------------------------------------===//
// PowerPC ISA Future specific type constraints.
//

def SDT_PPCInst512 : SDTypeProfile<1, 2, [
  SDTCisVT<0, v512i1>, SDTCisVT<1, v256i1>, SDTCisVT<2, v256i1>
]>;
```
- **EN**: Adds declarative TableGen records such as `SDT_PPCInst512` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SDT_PPCInst512`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 22-25

```tablegen
def SDT_PPCInst256 : SDTypeProfile<1, 2, [
  SDTCisVT<0, v256i1>, SDTCisVT<1, v256i1>, SDTCisVT<2, i32>
]>;
```
- **EN**: Adds declarative TableGen records such as `SDT_PPCInst256` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SDT_PPCInst256`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 26-33

```tablegen
//===----------------------------------------------------------------------===//
// ISA Future specific PPCISD nodes.
//

def PPCInst512   : SDNode<"PPCISD::INST512", SDT_PPCInst512, []>;
def PPCInst512Hi : SDNode<"PPCISD::INST512HI", SDT_PPCInst512, []>;
def PPCInst256   : SDNode<"PPCISD::INST256", SDT_PPCInst256, []>;
```
- **EN**: Adds declarative TableGen records such as `PPCInst512`, `PPCInst512Hi`, `PPCInst256` that LLVM later expands into generated tables or helper code. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `PPCInst512`, `PPCInst512Hi`, `PPCInst256`，LLVM 随后会把它们展开成生成表或辅助代码。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 34-43

```tablegen
//===----------------------------------------------------------------------===//

class XX3Form_AT3_XABp5_P1<bits<6> opcode, bits<8> xo, dag OOL, dag IOL,
                           string asmstr, list<dag> pattern>
    : I<opcode, OOL, IOL, asmstr, NoItinerary> {
  bits<3> AT;
  bits<5> XAp;
  bits<5> XBp;
  bits<1> P;
```
- **EN**: Declares a backend-facing type `XX3Form_AT3_XABp5_P1` and outlines the API or state that nearby code will rely on. Scheduling or processor-model metadata is part of this section.
- **CN**: 这里声明面向后端的类型 `XX3Form_AT3_XABp5_P1`，并勾勒出周边代码会依赖的接口或状态。 这一段包含调度或处理器模型元数据。

### Lines 44-50

```tablegen
  let Pattern = pattern;

  let Inst{6...8} = AT{2...0};
  let Inst{9...10} = 0;
  let Inst{11...14} = XAp{3...0};
  let Inst{15} = P;
  let Inst{16...19} = XBp{3...0};
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 51-57

```tablegen
  let Inst{20} = 0;
  let Inst{21...28} = xo;
  let Inst{29} = XAp{4};
  let Inst{30} = XBp{4};
  let Inst{31} = 0;
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 58-64

```tablegen
class XX2Form_AT3_XBp5_P2<bits<6> opcode, bits<9> xo, dag OOL, dag IOL,
                          string asmstr, list<dag> pattern>
    : I<opcode, OOL, IOL, asmstr, NoItinerary> {
  bits<3> AT;
  bits<5> XBp;
  bits<2> P;
```
- **EN**: Declares a backend-facing type `XX2Form_AT3_XBp5_P2` and outlines the API or state that nearby code will rely on. Scheduling or processor-model metadata is part of this section.
- **CN**: 这里声明面向后端的类型 `XX2Form_AT3_XBp5_P2`，并勾勒出周边代码会依赖的接口或状态。 这一段包含调度或处理器模型元数据。

### Lines 65-71

```tablegen
  let Pattern = pattern;

  let Inst{6...8} = AT{2...0};
  let Inst{9...14} = 0;
  let Inst{15} = P{0};
  let Inst{16...19} = XBp{3...0};
  let Inst{20} = P{1};
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 72-82

```tablegen
  let Inst{21...29} = xo;
  let Inst{30} = XBp{4};
  let Inst{31} = 0;
}

class XForm_ATB3<bits<6> opcode, bits<5> o, bits<10> xo, dag OOL, dag IOL,
                 string asmstr, list<dag> pattern>
    : I<opcode, OOL, IOL, asmstr, NoItinerary> {
  bits<3> AT;
  bits<3> AB;
```
- **EN**: Declares a backend-facing type `XForm_ATB3` and outlines the API or state that nearby code will rely on. Scheduling or processor-model metadata is part of this section.
- **CN**: 这里声明面向后端的类型 `XForm_ATB3`，并勾勒出周边代码会依赖的接口或状态。 这一段包含调度或处理器模型元数据。

### Lines 83-89

```tablegen
  let Pattern = pattern;

  let Inst{6...8} = AT{2...0};
  let Inst{9...10} = 0;
  let Inst{11...15} = o;
  let Inst{16...18} = AB{2...0};
  let Inst{19...20} = 0;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 90-100

```tablegen
  let Inst{21...30} = xo;
  let Inst{31} = 0;
}

class XX3Form_AT3_XAp5B6<bits<6> opcode, bits<8> xo, dag OOL, dag IOL,
                         string asmstr, InstrItinClass itin, list<dag> pattern>
    : I<opcode, OOL, IOL, asmstr, itin> {
  bits<3> AT;
  bits<5> XAp;
  bits<6> XB;
```
- **EN**: Declares a backend-facing type `XX3Form_AT3_XAp5B6` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XX3Form_AT3_XAp5B6`，并勾勒出周边代码会依赖的接口或状态。

### Lines 101-107

```tablegen
  let Pattern = pattern;

  let Inst{6...8} = AT;
  let Inst{9...10} = 0;
  let Inst{11...14} = XAp{3...0};
  let Inst{15} = 0;
  let Inst{16...20} = XB{4...0};
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 108-124

```tablegen
  let Inst{21...28} = xo;
  let Inst{29} = XAp{4};
  let Inst{30} = XB{5};
  let Inst{31} = 0;
}

class MMIRR_XX3Form_X8YP4_XAp5B6<bits<6> opcode, bits<8> xo, dag OOL, dag IOL,
                                 string asmstr, InstrItinClass itin,
                                 list<dag> pattern>
    : PI<1, opcode, OOL, IOL, asmstr, itin> {
  bits<3> AT;
  bits<5> XAp;
  bits<6> XB;
  bits<8> XMSK;
  bits<4> YMSK;
  bits<4> PMSK;
```
- **EN**: Declares a backend-facing type `MMIRR_XX3Form_X8YP4_XAp5B6` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `MMIRR_XX3Form_X8YP4_XAp5B6`，并勾勒出周边代码会依赖的接口或状态。

### Lines 125-131

```tablegen
  let Pattern = pattern;

  // The prefix.
  let Inst{6...7} = 3;
  let Inst{8...11} = 9;
  let Inst{12...15} = 0;
  let Inst{16...19} = PMSK;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 132-138

```tablegen
  let Inst{20...27} = XMSK;
  let Inst{28...31} = YMSK;

  // The instruction.
  let Inst{38...40} = AT;
  let Inst{41...42} = 0;
  let Inst{43...46} = XAp{3...0};
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 139-146

```tablegen
  let Inst{47} = 0;
  let Inst{48...52} = XB{4...0};
  let Inst{53...60} = xo;
  let Inst{61} = XAp{4};
  let Inst{62} = XB{5};
  let Inst{63} = 0;
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 147-157

```tablegen
class MMIRR_XX3Form_X8Y4P2_XAp5B6<bits<6> opcode, bits<8> xo, dag OOL, dag IOL,
                                  string asmstr, InstrItinClass itin,
                                  list<dag> pattern>
    : PI<1, opcode, OOL, IOL, asmstr, itin> {
  bits<3> AT;
  bits<5> XAp;
  bits<6> XB;
  bits<8> XMSK;
  bits<4> YMSK;
  bits<2> PMSK;
```
- **EN**: Declares a backend-facing type `MMIRR_XX3Form_X8Y4P2_XAp5B6` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `MMIRR_XX3Form_X8Y4P2_XAp5B6`，并勾勒出周边代码会依赖的接口或状态。

### Lines 158-164

```tablegen
  let Pattern = pattern;

  // The prefix.
  let Inst{6...7} = 3;
  let Inst{8...11} = 9;
  let Inst{12...15} = 0;
  let Inst{16...17} = PMSK;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 165-171

```tablegen
  let Inst{18...19} = 0;
  let Inst{20...27} = XMSK;
  let Inst{28...31} = YMSK;

  // The instruction.
  let Inst{38...40} = AT;
  let Inst{41...42} = 0;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 172-180

```tablegen
  let Inst{43...46} = XAp{3...0};
  let Inst{47} = 0;
  let Inst{48...52} = XB{4...0};
  let Inst{53...60} = xo;
  let Inst{61} = XAp{4};
  let Inst{62} = XB{5};
  let Inst{63} = 0;
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 181-187

```tablegen
multiclass DMR_UM_XOEO<bits<6> opcode, bits<8> xo, dag IOL, string asmbase,
                       string asmstr> {
  let Predicates = [MMA, IsISAFuture] in {
    def NAME
        : XX3Form_AT3_XAp5B6<opcode, !or(xo, 0x01), (outs dmr:$AT), IOL,
                             !strconcat(asmbase#" ", asmstr), IIC_VecFP, []>,
          RegConstraint<"@earlyclobber $AT">;
```
- **EN**: Adds declarative TableGen records such as `DMR_UM_XOEO` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `DMR_UM_XOEO`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 188-195

```tablegen
    def PP
        : XX3Form_AT3_XAp5B6<opcode, xo, (outs dmr:$AT),
                             !con((ins dmr:$ATi), IOL),
                             !strconcat(asmbase#"pp ", asmstr), IIC_VecFP, []>,
          RegConstraint<"$ATi = $AT">;
  }
}
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 196-206

```tablegen
multiclass DMR_UM_M448_XOEO<bits<6> opcode, bits<8> xo, dag IOL, string asmbase,
                            string asmstr> {
  defm NAME : DMR_UM_XOEO<opcode, xo, IOL, asmbase, asmstr>;
  let Predicates = [MMA, PrefixInstrs, IsISAFuture] in {
    def PM#NAME
        : MMIRR_XX3Form_X8YP4_XAp5B6<
              opcode, !or(xo, 0x01), (outs dmr:$AT),
              !con(IOL, (ins u8imm:$XMSK, u4imm:$YMSK, u4imm:$PMSK)),
              !strconcat("pm"#asmbase#" ", asmstr#", $XMSK, $YMSK, $PMSK"),
              IIC_VecFP, []>,
          RegConstraint<"@earlyclobber $AT">;
```
- **EN**: Adds declarative TableGen records such as `DMR_UM_M448_XOEO` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `DMR_UM_M448_XOEO`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 207-217

```tablegen
    def PM#NAME#PP
        : MMIRR_XX3Form_X8YP4_XAp5B6<
              opcode, xo, (outs dmr:$AT),
              !con((ins dmr:$ATi),
                   !con(IOL, (ins u8imm:$XMSK, u4imm:$YMSK, u4imm:$PMSK))),
              !strconcat("pm"#asmbase#"pp ", asmstr#", $XMSK, $YMSK, $PMSK"),
              IIC_VecFP, []>,
          RegConstraint<"$ATi = $AT">;
  }
}
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 218-224

```tablegen
multiclass DMR_BF16_UM_XOEO<bits<6> opcode, bits<8> xo, dag IOL, string asmbase,
                            string asmstr> {
  let Predicates = [MMA, IsISAFuture] in {
    def NAME
        : XX3Form_AT3_XAp5B6<opcode, !or(xo, 0x11), (outs dmr:$AT), IOL,
                             !strconcat(asmbase#" ", asmstr), IIC_VecFP, []>,
          RegConstraint<"@earlyclobber $AT">;
```
- **EN**: Adds declarative TableGen records such as `DMR_BF16_UM_XOEO` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `DMR_BF16_UM_XOEO`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 225-232

```tablegen
    def PP
        : XX3Form_AT3_XAp5B6<opcode, xo, (outs dmr:$AT),
                             !con((ins dmr:$ATi), IOL),
                             !strconcat(asmbase#"pp ", asmstr), IIC_VecFP, []>,
          RegConstraint<"$ATi = $AT">;
  }
}
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 233-243

```tablegen
multiclass DMR_BF16_UM_M284_XOEO<bits<6> opcode, bits<8> xo, dag IOL,
                                 string asmbase, string asmstr> {
  defm NAME : DMR_BF16_UM_XOEO<opcode, xo, IOL, asmbase, asmstr>;
  let Predicates = [MMA, PrefixInstrs, IsISAFuture] in {
    def PM#NAME
        : MMIRR_XX3Form_X8Y4P2_XAp5B6<
              opcode, !or(xo, 0x11), (outs dmr:$AT),
              !con(IOL, (ins u8imm:$XMSK, u4imm:$YMSK, u2imm:$PMSK)),
              !strconcat("pm"#asmbase#" ", asmstr#", $XMSK, $YMSK, $PMSK"),
              IIC_VecFP, []>,
          RegConstraint<"@earlyclobber $AT">;
```
- **EN**: Adds declarative TableGen records such as `DMR_BF16_UM_M284_XOEO` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `DMR_BF16_UM_M284_XOEO`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 244-254

```tablegen
    def PM#NAME#PP
        : MMIRR_XX3Form_X8Y4P2_XAp5B6<
              opcode, xo, (outs dmr:$AT),
              !con((ins dmr:$ATi),
                   !con(IOL, (ins u8imm:$XMSK, u4imm:$YMSK, u2imm:$PMSK))),
              !strconcat("pm"#asmbase#"pp ", asmstr#", $XMSK, $YMSK, $PMSK"),
              IIC_VecFP, []>,
          RegConstraint<"$ATi = $AT">;
  }
}
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 255-265

```tablegen
multiclass DMR_F16_UM_M284_XOEO<bits<6> opcode, bits<8> xo, dag IOL,
                                string asmbase, string asmstr> {
  defm NAME : DMR_UM_XOEO<opcode, xo, IOL, asmbase, asmstr>;
  let Predicates = [MMA, PrefixInstrs, IsISAFuture] in {
    def PM#NAME
        : MMIRR_XX3Form_X8Y4P2_XAp5B6<
              opcode, !or(xo, 0x01), (outs dmr:$AT),
              !con(IOL, (ins u8imm:$XMSK, u4imm:$YMSK, u2imm:$PMSK)),
              !strconcat("pm"#asmbase#" ", asmstr#", $XMSK, $YMSK, $PMSK"),
              IIC_VecFP, []>,
          RegConstraint<"@earlyclobber $AT">;
```
- **EN**: Adds declarative TableGen records such as `DMR_F16_UM_M284_XOEO` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `DMR_F16_UM_M284_XOEO`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 266-276

```tablegen
    def PM#NAME#PP
        : MMIRR_XX3Form_X8Y4P2_XAp5B6<
              opcode, xo, (outs dmr:$AT),
              !con((ins dmr:$ATi),
                   !con(IOL, (ins u8imm:$XMSK, u4imm:$YMSK, u2imm:$PMSK))),
              !strconcat("pm"#asmbase#"pp ", asmstr#", $XMSK, $YMSK, $PMSK"),
              IIC_VecFP, []>,
          RegConstraint<"$ATi = $AT">;
  }
}
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 277-285

```tablegen
multiclass DMR_NEG_UM_M284_XOXORf939a0<bits<6> opcode, bits<8> xo, dag IOL,
                                       string asmbase, string asmstr> {
  defm NAME : DMR_BF16_UM_M284_XOEO<opcode, xo, IOL, asmbase, asmstr>;
  let Predicates = [MMA, IsISAFuture] in {
    def PN
        : XX3Form_AT3_XAp5B6<opcode, !xor(xo, 0xF9), (outs dmr:$AT),
                             !con((ins dmr:$ATi), IOL),
                             !strconcat(asmbase#"pn ", asmstr), IIC_VecFP, []>,
          RegConstraint<"$ATi = $AT">;
```
- **EN**: Adds declarative TableGen records such as `DMR_NEG_UM_M284_XOXORf939a0` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `DMR_NEG_UM_M284_XOXORf939a0`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 286-296

```tablegen
    def NP
        : XX3Form_AT3_XAp5B6<opcode, !xor(xo, 0x39), (outs dmr:$AT),
                             !con((ins dmr:$ATi), IOL),
                             !strconcat(asmbase#"np ", asmstr), IIC_VecFP, []>,
          RegConstraint<"$ATi = $AT">;
    def NN
        : XX3Form_AT3_XAp5B6<opcode, !xor(xo, 0xA0), (outs dmr:$AT),
                             !con((ins dmr:$ATi), IOL),
                             !strconcat(asmbase#"nn ", asmstr), IIC_VecFP, []>,
          RegConstraint<"$ATi = $AT">;
  }
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 297-305

```tablegen
  let Predicates = [MMA, PrefixInstrs, IsISAFuture] in {
    def PM#NAME#PN
        : MMIRR_XX3Form_X8Y4P2_XAp5B6<
              opcode, !xor(xo, 0xF9), (outs dmr:$AT),
              !con((ins dmr:$ATi),
                   !con(IOL, (ins u8imm:$XMSK, u4imm:$YMSK, u2imm:$PMSK))),
              !strconcat("pm"#asmbase#"pn ", asmstr#", $XMSK, $YMSK, $PMSK"),
              IIC_VecFP, []>,
          RegConstraint<"$ATi = $AT">;
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend TableGen declarations for the backend. These declarations feed generated pattern-matching logic.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 TableGen 声明所需的辅助例程。 这些声明会进入生成式模式匹配逻辑。

### Lines 306-313

```tablegen
    def PM#NAME#NP
        : MMIRR_XX3Form_X8Y4P2_XAp5B6<
              opcode, !xor(xo, 0x39), (outs dmr:$AT),
              !con((ins dmr:$ATi),
                   !con(IOL, (ins u8imm:$XMSK, u4imm:$YMSK, u2imm:$PMSK))),
              !strconcat("pm"#asmbase#"np ", asmstr#", $XMSK, $YMSK, $PMSK"),
              IIC_VecFP, []>,
          RegConstraint<"$ATi = $AT">;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 314-324

```tablegen
    def PM#NAME#NN
        : MMIRR_XX3Form_X8Y4P2_XAp5B6<
              opcode, !xor(xo, 0xA0), (outs dmr:$AT),
              !con((ins dmr:$ATi),
                   !con(IOL, (ins u8imm:$XMSK, u4imm:$YMSK, u2imm:$PMSK))),
              !strconcat("pm"#asmbase#"nn ", asmstr#", $XMSK, $YMSK, $PMSK"),
              IIC_VecFP, []>,
          RegConstraint<"$ATi = $AT">;
  }
}
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 325-333

```tablegen
multiclass DMR_NEG_UM_M284_XOXORd11188<bits<6> opcode, bits<8> xo, dag IOL,
                                       string asmbase, string asmstr> {
  defm NAME : DMR_F16_UM_M284_XOEO<opcode, xo, IOL, asmbase, asmstr>;
  let Predicates = [MMA, IsISAFuture] in {
    def PN
        : XX3Form_AT3_XAp5B6<opcode, !xor(xo, 0xD1), (outs dmr:$AT),
                             !con((ins dmr:$ATi), IOL),
                             !strconcat(asmbase#"pn ", asmstr), IIC_VecFP, []>,
          RegConstraint<"$ATi = $AT">;
```
- **EN**: Adds declarative TableGen records such as `DMR_NEG_UM_M284_XOXORd11188` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `DMR_NEG_UM_M284_XOXORd11188`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 334-344

```tablegen
    def NP
        : XX3Form_AT3_XAp5B6<opcode, !xor(xo, 0x11), (outs dmr:$AT),
                             !con((ins dmr:$ATi), IOL),
                             !strconcat(asmbase#"np ", asmstr), IIC_VecFP, []>,
          RegConstraint<"$ATi = $AT">;
    def NN
        : XX3Form_AT3_XAp5B6<opcode, !xor(xo, 0x88), (outs dmr:$AT),
                             !con((ins dmr:$ATi), IOL),
                             !strconcat(asmbase#"nn ", asmstr), IIC_VecFP, []>,
          RegConstraint<"$ATi = $AT">;
  }
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 345-353

```tablegen
  let Predicates = [MMA, PrefixInstrs, IsISAFuture] in {
    def PM#NAME#PN
        : MMIRR_XX3Form_X8Y4P2_XAp5B6<
              opcode, !xor(xo, 0xD1), (outs dmr:$AT),
              !con((ins dmr:$ATi),
                   !con(IOL, (ins u8imm:$XMSK, u4imm:$YMSK, u2imm:$PMSK))),
              !strconcat("pm"#asmbase#"pn ", asmstr#", $XMSK, $YMSK, $PMSK"),
              IIC_VecFP, []>,
          RegConstraint<"$ATi = $AT">;
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend TableGen declarations for the backend. These declarations feed generated pattern-matching logic.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 TableGen 声明所需的辅助例程。 这些声明会进入生成式模式匹配逻辑。

### Lines 354-361

```tablegen
    def PM#NAME#NP
        : MMIRR_XX3Form_X8Y4P2_XAp5B6<
              opcode, !xor(xo, 0x11), (outs dmr:$AT),
              !con((ins dmr:$ATi),
                   !con(IOL, (ins u8imm:$XMSK, u4imm:$YMSK, u2imm:$PMSK))),
              !strconcat("pm"#asmbase#"np ", asmstr#", $XMSK, $YMSK, $PMSK"),
              IIC_VecFP, []>,
          RegConstraint<"$ATi = $AT">;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 362-372

```tablegen
    def PM#NAME#NN
        : MMIRR_XX3Form_X8Y4P2_XAp5B6<
              opcode, !xor(xo, 0x88), (outs dmr:$AT),
              !con((ins dmr:$ATi),
                   !con(IOL, (ins u8imm:$XMSK, u4imm:$YMSK, u2imm:$PMSK))),
              !strconcat("pm"#asmbase#"nn ", asmstr#", $XMSK, $YMSK, $PMSK"),
              IIC_VecFP, []>,
          RegConstraint<"$ATi = $AT">;
  }
}
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 373-379

```tablegen
class XForm_AT3_T1_AB3<bits<6> opcode, bits<5> o, bits<10> xo, dag OOL, dag IOL,
                       string asmstr, list<dag> pattern>
    : I<opcode, OOL, IOL, asmstr, NoItinerary> {
  bits<3> AT;
  bits<3> AB;
  bits<1> T;
```
- **EN**: Declares a backend-facing type `XForm_AT3_T1_AB3` and outlines the API or state that nearby code will rely on. Scheduling or processor-model metadata is part of this section.
- **CN**: 这里声明面向后端的类型 `XForm_AT3_T1_AB3`，并勾勒出周边代码会依赖的接口或状态。 这一段包含调度或处理器模型元数据。

### Lines 380-386

```tablegen
  let Pattern = pattern;

  let Inst{6...8} = AT{2...0};
  let Inst{9} = 0;
  let Inst{10} = T;
  let Inst{11...15} = o;
  let Inst{16...18} = AB{2...0};
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 387-397

```tablegen
  let Inst{19...20} = 0;
  let Inst{21...30} = xo;
  let Inst{31} = 0;
}

class XForm_ATp2_SR5<bits<6> opcode, bits<5> o, bits<10> xo, dag OOL, dag IOL,
                     string asmstr, list<dag> pattern>
    : I<opcode, OOL, IOL, asmstr, NoItinerary> {
  bits<2> ATp;
  bits<5> SR;
```
- **EN**: Declares a backend-facing type `XForm_ATp2_SR5` and outlines the API or state that nearby code will rely on. Scheduling or processor-model metadata is part of this section.
- **CN**: 这里声明面向后端的类型 `XForm_ATp2_SR5`，并勾勒出周边代码会依赖的接口或状态。 这一段包含调度或处理器模型元数据。

### Lines 398-404

```tablegen
  let Pattern = pattern;

  let Inst{6...7} = ATp{1...0};
  let Inst{8...10} = 0;
  let Inst{11...15} = o;
  let Inst{16...20} = SR{4...0};
  let Inst{21...30} = xo;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 405-416

```tablegen
  let Inst{31} = 0;
}

class XX2Form_AT3_XB6_ID2_E1_BL2<bits<6> opcode, bits<9> xo, dag OOL, dag IOL,
                                 string asmstr, list<dag> pattern>
    : I<opcode, OOL, IOL, asmstr, NoItinerary> {
  bits<3> AT;
  bits<6> XB;
  bits<2> ID;
  bits<1> E;
  bits<2> BL;
```
- **EN**: Declares a backend-facing type `XX2Form_AT3_XB6_ID2_E1_BL2` and outlines the API or state that nearby code will rely on. Scheduling or processor-model metadata is part of this section.
- **CN**: 这里声明面向后端的类型 `XX2Form_AT3_XB6_ID2_E1_BL2`，并勾勒出周边代码会依赖的接口或状态。 这一段包含调度或处理器模型元数据。

### Lines 417-423

```tablegen
  let Pattern = pattern;

  let Inst{6...8} = AT{2...0};
  let Inst{9...10} = 0;
  let Inst{11...12} = ID{1...0};
  let Inst{13} = E;
  let Inst{14...15} = BL{1...0};
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 424-434

```tablegen
  let Inst{16...20} = XB{4...0};
  let Inst{21...29} = xo;
  let Inst{30} = XB{5};
  let Inst{31} = 0;
}

//-------------------------- Instruction definitions -------------------------//
// Predicate combinations available:
// [MMA, IsISAFuture]
// [MMA, PrefixInstrs, IsISAFuture]
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "-------------------------- Instruction definitions -------------------------//". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“-------------------------- Instruction definitions -------------------------//”。 这些声明会进入生成式模式匹配逻辑。

### Lines 435-442

```tablegen
let Predicates = [MMA, IsISAFuture] in {
  def DMXXEXTFDMR512
      : XX3Form_AT3_XABp5_P1<60, 226, (outs vsrprc:$XAp, vsrprc:$XBp),
                             (ins wacc:$AT),
                             "dmxxextfdmr512 $XAp, $XBp, $AT, 0", []> {
    let P = 0;
  }
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 443-449

```tablegen
  def DMXXEXTFDMR512_HI
      : XX3Form_AT3_XABp5_P1<60, 226, (outs vsrprc:$XAp, vsrprc:$XBp),
                             (ins wacc_hi:$AT),
                             "dmxxextfdmr512 $XAp, $XBp, $AT, 1", []> {
    let P = 1;
  }
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 450-458

```tablegen
  def DMXXINSTDMR512
      : XX3Form_AT3_XABp5_P1<60, 234, (outs wacc:$AT),
                             (ins vsrprc:$XAp, vsrprc:$XBp),
                             "dmxxinstdmr512 $AT, $XAp, $XBp, 0",
                             [(set v512i1:$AT, (PPCInst512 v256i1:$XAp,
                                                v256i1:$XBp))]> {
    let P = 0;
  }
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 459-467

```tablegen
  def DMXXINSTDMR512_HI
      : XX3Form_AT3_XABp5_P1<60, 234, (outs wacc_hi:$AT),
                             (ins vsrprc:$XAp, vsrprc:$XBp),
                             "dmxxinstdmr512 $AT, $XAp, $XBp, 1",
                             [(set v512i1:$AT, (PPCInst512Hi v256i1:$XAp,
                                                v256i1:$XBp))]> {
    let P = 1;
  }
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 468-476

```tablegen
  def DMXXEXTFDMR256 : XX2Form_AT3_XBp5_P2<60, 484, (outs vsrprc:$XBp),
                                           (ins dmrrowp:$AT, u2imm:$P),
                                           "dmxxextfdmr256 $XBp, $AT, $P", []>;

  def DMXXINSTDMR256 : XX2Form_AT3_XBp5_P2<60, 485, (outs dmrrowp:$AT),
                                           (ins vsrprc:$XBp, u2imm:$P),
                                           "dmxxinstdmr256 $AT, $XBp, $P",
                                           [(set v256i1:$AT, (PPCInst256 v256i1:$XBp, u2imm_timm:$P))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 477-486

```tablegen
  def DMMR
      : XForm_ATB3<31, 6, 177, (outs dmr:$AT), (ins dmr:$AB), "dmmr $AT, $AB",
                   [(set v1024i1:$AT, (int_ppc_dmmr v1024i1:$AB))]>;

  def DMXOR : XForm_ATB3<31, 7, 177, (outs dmr:$AT), (ins dmr:$ATi, dmr:$AB),
                         "dmxor $AT, $AB",
                         [(set v1024i1:$AT, (int_ppc_dmxor v1024i1:$ATi,
                                                v1024i1:$AB))]>,
              RegConstraint<"$ATi = $AT">;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 487-497

```tablegen
  def DMSETDMRZ
      : XForm_AT3<31, 2, 177, (outs dmr:$AT), (ins), "dmsetdmrz $AT",
                  NoItinerary, [(set v1024i1:$AT, (int_ppc_dmsetdmrz))]>;

  // DMXVI8GERX4, DMXVI8GERX4PP, PMDMXVI8GERX4,  PMDMXVI8GERX4PP
  defm DMXVI8GERX4 : DMR_UM_M448_XOEO<59, 10, (ins vsrprc:$XAp, vsrc:$XB),
                                      "dmxvi8gerx4", "$AT, $XAp, $XB">;

  // DMXVBF16GERX2, DMXVBF16GERX2PP, DMXVBF16GERX2PN, dMXVBF16GERX2NP,
  // DMXVBF16GERX2NN PMDMXVBF16GERX2, PMDMXVBF16GERX2PP, PMDMXVBF16GERX2PN,
  // PMDMXVBF16GERX2NP, PMDMXVBF16GERX2NN
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "DMXVI8GERX4, DMXVI8GERX4PP, PMDMXVI8GERX4,  PMDMXVI8GERX4PP". Scheduling or processor-model metadata is part of this section.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“DMXVI8GERX4, DMXVI8GERX4PP, PMDMXVI8GERX4,  PMDMXVI8GERX4PP”。 这一段包含调度或处理器模型元数据。

### Lines 498-504

```tablegen
  defm DMXVBF16GERX2
      : DMR_NEG_UM_M284_XOXORf939a0<59, 74, (ins vsrprc:$XAp, vsrc:$XB),
                                    "dmxvbf16gerx2", "$AT, $XAp, $XB">;

  // DMXVF16GERX2, DMXVF16GERX2PP, DMXVF16GERX2PN, dMXVF16GERX2NP,
  // DMXVF16GERX2NN PMDMXVF16GERX2, PMDMXVF16GERX2PP, PMDMXVF16GERX2PN,
  // PMDMXVF16GERX2NP, PMDMXVF16GERX2NN
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "DMXVF16GERX2, DMXVF16GERX2PP, DMXVF16GERX2PN, dMXVF16GERX2NP,".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“DMXVF16GERX2, DMXVF16GERX2PP, DMXVF16GERX2PN, dMXVF16GERX2NP,”。

### Lines 505-516

```tablegen
  defm DMXVF16GERX2
      : DMR_NEG_UM_M284_XOXORd11188<59, 66, (ins vsrprc:$XAp, vsrc:$XB),
                                    "dmxvf16gerx2", "$AT, $XAp, $XB">;

  // DMF cryptography [support] Instructions
  def DMSHA2HASH
      : XForm_AT3_T1_AB3<
            31, 14, 177, (outs dmr:$AT), (ins dmr:$ATi, dmr:$AB, u1imm:$T),
            "dmsha2hash $AT, $AB, $T",
            [(set v1024i1:$AT, (int_ppc_dmsha2hash v1024i1:$ATi,
                                   v1024i1:$AB, u1imm_timm:$T))]>,
        RegConstraint<"$ATi = $AT">;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 517-523

```tablegen
  def DMSHA3HASH
      : XForm_ATp2_SR5<31, 15, 177, (outs dmrp:$ATp),
                       (ins dmrp:$ATpi, u5imm:$SR), "dmsha3hash $ATp, $SR",
                       [(set v2048i1:$ATp,
                           (int_ppc_dmsha3hash v2048i1:$ATpi,
                                                   u5imm_timm:$SR))]>,
        RegConstraint<"$ATpi = $ATp">;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 524-531

```tablegen
  def DMXXSHAPAD
      : XX2Form_AT3_XB6_ID2_E1_BL2<60, 421, (outs dmr:$AT),
                                   (ins dmr:$ATi, vsrc:$XB, u2imm:$ID, u1imm:$E,
                                       u2imm:$BL),
                                   "dmxxshapad $AT, $XB, $ID, $E, $BL", []>,
        RegConstraint<"$ATi = $AT">;

  // MMA+ accumulating/non-accumulating instructions.
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 532-539

```tablegen
  def DMXVI8GERX4SPP
      : XX3Form_AT3_XAp5B6<59, 98, (outs dmr:$AT),
                           (ins dmr:$ATi, vsrprc:$XAp, vsrc:$XB),
                           "dmxvi8gerx4spp $AT, $XAp, $XB", IIC_VecGeneral, []>,
        RegConstraint<"$ATi = $AT">;

} // End of [MMA, IsISAFuture]
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 540-555

```tablegen
let Predicates = [MMA, PrefixInstrs, IsISAFuture] in {
  def PMDMXVI8GERX4SPP
      : MMIRR_XX3Form_X8YP4_XAp5B6<
            59, 98, (outs dmr:$AT),
            (ins dmr:$ATi, vsrprc:$XAp, vsrc:$XB, u8imm:$XMSK, u4imm:$YMSK,
                u4imm:$PMSK),
            "pmdmxvi8gerx4spp $AT, $XAp, $XB, $XMSK, $YMSK, $PMSK",
            IIC_VecGeneral, []>,
        RegConstraint<"$ATi = $AT">;
}

//---------------------------- Anonymous Patterns ----------------------------//
// Predicate combinations available:
// [MMA, IsISAFuture]
// [MMA, PrefixInstrs, IsISAFuture]
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 556-562

```tablegen
let Predicates = [MMA, IsISAFuture] in {
  // MMA+ Intrinsics
  def : Pat<(v1024i1 (int_ppc_mma_dmxvi8gerx4 v256i1:$XAp, v16i8:$XB)),
            (DMXVI8GERX4 $XAp, RCCp.BToVSRC)>;
  def : Pat<(v1024i1 (int_ppc_mma_dmxvi8gerx4pp v1024i1:$ATi, v256i1:$XAp,
                v16i8:$XB)),
            (DMXVI8GERX4PP $ATi, $XAp, RCCp.BToVSRC)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 563-570

```tablegen
  def : Pat<(v1024i1 (int_ppc_mma_dmxvi8gerx4spp v1024i1:$ATi, v256i1:$XAp,
                v16i8:$XB)),
            (DMXVI8GERX4SPP $ATi, $XAp, RCCp.BToVSRC)>;
  def : Pat<(v1024i1 (int_ppc_mma_dmxvbf16gerx2 v256i1:$XAp, v16i8:$XB)),
            (DMXVBF16GERX2 $XAp, RCCp.BToVSRC)>;
  def : Pat<(v1024i1 (int_ppc_mma_dmxvbf16gerx2pp v1024i1:$ATi, v256i1:$XAp,
                v16i8:$XB)),
            (DMXVBF16GERX2PP $ATi, $XAp, RCCp.BToVSRC)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 571-579

```tablegen
  def : Pat<(v1024i1 (int_ppc_mma_dmxvbf16gerx2pn v1024i1:$ATi, v256i1:$XAp,
                v16i8:$XB)),
            (DMXVBF16GERX2PN $ATi, $XAp, RCCp.BToVSRC)>;
  def : Pat<(v1024i1 (int_ppc_mma_dmxvbf16gerx2np v1024i1:$ATi, v256i1:$XAp,
                v16i8:$XB)),
            (DMXVBF16GERX2NP $ATi, $XAp, RCCp.BToVSRC)>;
  def : Pat<(v1024i1 (int_ppc_mma_dmxvbf16gerx2nn v1024i1:$ATi, v256i1:$XAp,
                v16i8:$XB)),
            (DMXVBF16GERX2NN $ATi, $XAp, RCCp.BToVSRC)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 580-587

```tablegen
  def : Pat<(v1024i1 (int_ppc_mma_dmxvf16gerx2 v256i1:$XAp, v16i8:$XB)),
            (DMXVF16GERX2 $XAp, RCCp.BToVSRC)>;
  def : Pat<(v1024i1 (int_ppc_mma_dmxvf16gerx2pp v1024i1:$ATi, v256i1:$XAp,
                v16i8:$XB)),
            (DMXVF16GERX2PP $ATi, $XAp, RCCp.BToVSRC)>;
  def : Pat<(v1024i1 (int_ppc_mma_dmxvf16gerx2pn v1024i1:$ATi, v256i1:$XAp,
                v16i8:$XB)),
            (DMXVF16GERX2PN $ATi, $XAp, RCCp.BToVSRC)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 588-595

```tablegen
  def : Pat<(v1024i1 (int_ppc_mma_dmxvf16gerx2np v1024i1:$ATi, v256i1:$XAp,
                v16i8:$XB)),
            (DMXVF16GERX2NP $ATi, $XAp, RCCp.BToVSRC)>;
  def : Pat<(v1024i1 (int_ppc_mma_dmxvf16gerx2nn v1024i1:$ATi, v256i1:$XAp,
                v16i8:$XB)),
            (DMXVF16GERX2NN $ATi, $XAp, RCCp.BToVSRC)>;

  // Cryptography Intrinsic
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 596-606

```tablegen
  def : Pat<(v1024i1 (int_ppc_dmxxshapad v1024i1:$ATi, v16i8:$XB,
                u2imm_timm:$ID, u1imm_timm:$E, u2imm_timm:$BL)),
            (DMXXSHAPAD $ATi, RCCp.BToVSRC, $ID, $E, $BL)>;
}

let Predicates = [MMA, PrefixInstrs, IsISAFuture] in {
  def : Pat<(v1024i1 (int_ppc_mma_pmdmxvi8gerx4 v256i1:$XAp, v16i8:$XB,
                Msk8Imm:$XMSK, Msk4Imm:$YMSK, Msk4Imm:$PMSK)),
            (PMDMXVI8GERX4 $XAp, RCCp.BToVSRC, Msk8Imm:$XMSK, Msk4Imm:$YMSK,
                Msk4Imm:$PMSK)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 607-616

```tablegen
  def : Pat<(v1024i1 (int_ppc_mma_pmdmxvi8gerx4pp v1024i1:$ATi, v256i1:$XAp,
                v16i8:$XB, Msk8Imm:$XMSK, Msk4Imm:$YMSK, Msk4Imm:$PMSK)),
            (PMDMXVI8GERX4PP $ATi, $XAp, RCCp.BToVSRC, Msk8Imm:$XMSK,
                Msk4Imm:$YMSK, Msk4Imm:$PMSK)>;

  def : Pat<(v1024i1 (int_ppc_mma_pmdmxvi8gerx4spp v1024i1:$ATi, v256i1:$XAp,
                v16i8:$XB, Msk8Imm:$XMSK, Msk4Imm:$YMSK, Msk4Imm:$PMSK)),
            (PMDMXVI8GERX4SPP $ATi, $XAp, RCCp.BToVSRC, Msk8Imm:$XMSK,
                Msk4Imm:$YMSK, Msk4Imm:$PMSK)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 617-626

```tablegen
  def : Pat<(v1024i1 (int_ppc_mma_pmdmxvbf16gerx2 v256i1:$XAp, v16i8:$XB,
                Msk8Imm:$XMSK, Msk4Imm:$YMSK, Msk2Imm:$PMSK)),
            (PMDMXVBF16GERX2 $XAp, RCCp.BToVSRC, Msk8Imm:$XMSK, Msk4Imm:$YMSK,
                Msk2Imm:$PMSK)>;

  def : Pat<(v1024i1 (int_ppc_mma_pmdmxvbf16gerx2pp v1024i1:$ATi, v256i1:$XAp,
                v16i8:$XB, Msk8Imm:$XMSK, Msk4Imm:$YMSK, Msk2Imm:$PMSK)),
            (PMDMXVBF16GERX2PP $ATi, $XAp, RCCp.BToVSRC, Msk8Imm:$XMSK,
                Msk4Imm:$YMSK, Msk2Imm:$PMSK)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 627-636

```tablegen
  def : Pat<(v1024i1 (int_ppc_mma_pmdmxvbf16gerx2pn v1024i1:$ATi, v256i1:$XAp,
                v16i8:$XB, Msk8Imm:$XMSK, Msk4Imm:$YMSK, Msk2Imm:$PMSK)),
            (PMDMXVBF16GERX2PN $ATi, $XAp, RCCp.BToVSRC, Msk8Imm:$XMSK,
                Msk4Imm:$YMSK, Msk2Imm:$PMSK)>;

  def : Pat<(v1024i1 (int_ppc_mma_pmdmxvbf16gerx2np v1024i1:$ATi, v256i1:$XAp,
                v16i8:$XB, Msk8Imm:$XMSK, Msk4Imm:$YMSK, Msk2Imm:$PMSK)),
            (PMDMXVBF16GERX2NP $ATi, $XAp, RCCp.BToVSRC, Msk8Imm:$XMSK,
                Msk4Imm:$YMSK, Msk2Imm:$PMSK)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 637-646

```tablegen
  def : Pat<(v1024i1 (int_ppc_mma_pmdmxvbf16gerx2nn v1024i1:$ATi, v256i1:$XAp,
                v16i8:$XB, Msk8Imm:$XMSK, Msk4Imm:$YMSK, Msk2Imm:$PMSK)),
            (PMDMXVBF16GERX2NN $ATi, $XAp, RCCp.BToVSRC, Msk8Imm:$XMSK,
                Msk4Imm:$YMSK, Msk2Imm:$PMSK)>;

  def : Pat<(v1024i1 (int_ppc_mma_pmdmxvf16gerx2 v256i1:$XAp, v16i8:$XB,
                Msk8Imm:$XMSK, Msk4Imm:$YMSK, Msk2Imm:$PMSK)),
            (PMDMXVF16GERX2 $XAp, RCCp.BToVSRC, Msk8Imm:$XMSK, Msk4Imm:$YMSK,
                Msk2Imm:$PMSK)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 647-656

```tablegen
  def : Pat<(v1024i1 (int_ppc_mma_pmdmxvf16gerx2pp v1024i1:$ATi, v256i1:$XAp,
                v16i8:$XB, Msk8Imm:$XMSK, Msk4Imm:$YMSK, Msk2Imm:$PMSK)),
            (PMDMXVF16GERX2PP $ATi, $XAp, RCCp.BToVSRC, Msk8Imm:$XMSK,
                Msk4Imm:$YMSK, Msk2Imm:$PMSK)>;

  def : Pat<(v1024i1 (int_ppc_mma_pmdmxvf16gerx2pn v1024i1:$ATi, v256i1:$XAp,
                v16i8:$XB, Msk8Imm:$XMSK, Msk4Imm:$YMSK, Msk2Imm:$PMSK)),
            (PMDMXVF16GERX2PN $ATi, $XAp, RCCp.BToVSRC, Msk8Imm:$XMSK,
                Msk4Imm:$YMSK, Msk2Imm:$PMSK)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 657-669

```tablegen
  def : Pat<(v1024i1 (int_ppc_mma_pmdmxvf16gerx2np v1024i1:$ATi, v256i1:$XAp,
                v16i8:$XB, Msk8Imm:$XMSK, Msk4Imm:$YMSK, Msk2Imm:$PMSK)),
            (PMDMXVF16GERX2NP $ATi, $XAp, RCCp.BToVSRC, Msk8Imm:$XMSK,
                Msk4Imm:$YMSK, Msk2Imm:$PMSK)>;

  def : Pat<(v1024i1 (int_ppc_mma_pmdmxvf16gerx2nn v1024i1:$ATi, v256i1:$XAp,
                v16i8:$XB, Msk8Imm:$XMSK, Msk4Imm:$YMSK, Msk2Imm:$PMSK)),
            (PMDMXVF16GERX2NN $ATi, $XAp, RCCp.BToVSRC, Msk8Imm:$XMSK,
                Msk4Imm:$YMSK, Msk2Imm:$PMSK)>;
}

//---------------------------- Instruction aliases ---------------------------//
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 670-676

```tablegen
let Predicates = [MMA, IsISAFuture] in {
  def : InstAlias<"dmsha256hash $AT, $AB", (DMSHA2HASH dmr:$AT, dmr:$AB, 0)>;
  def : InstAlias<"dmsha512hash $AT, $AB", (DMSHA2HASH dmr:$AT, dmr:$AB, 1)>;
  def : InstAlias<"dmsha3dw $ATp", (DMSHA3HASH dmrp:$ATp, 0)>;
  def : InstAlias<"dmcryshash $ATp", (DMSHA3HASH dmrp:$ATp, 12)>;
  def : InstAlias<"dmxxsha3512pad $AT, $XB, $E", (DMXXSHAPAD dmr:$AT, vsrc:$XB,
                                                     0, u1imm:$E, 0)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 677-684

```tablegen
  def : InstAlias<"dmxxsha3384pad $AT, $XB, $E", (DMXXSHAPAD dmr:$AT, vsrc:$XB,
                                                     0, u1imm:$E, 1)>;
  def : InstAlias<"dmxxsha3256pad $AT, $XB, $E", (DMXXSHAPAD dmr:$AT, vsrc:$XB,
                                                     0, u1imm:$E, 2)>;
  def : InstAlias<"dmxxsha3224pad $AT, $XB, $E", (DMXXSHAPAD dmr:$AT, vsrc:$XB,
                                                     0, u1imm:$E, 3)>;
  def : InstAlias<"dmxxshake256pad $AT, $XB, $E", (DMXXSHAPAD dmr:$AT, vsrc:$XB,
                                                      1, u1imm:$E, 0)>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 685-691

```tablegen
  def : InstAlias<"dmxxshake128pad $AT, $XB, $E", (DMXXSHAPAD dmr:$AT, vsrc:$XB,
                                                      1, u1imm:$E, 1)>;
  def : InstAlias<"dmxxsha384512pad $AT, $XB", (DMXXSHAPAD dmr:$AT, vsrc:$XB, 2,
                                                   0, 0)>;
  def : InstAlias<"dmxxsha224256pad $AT, $XB", (DMXXSHAPAD dmr:$AT, vsrc:$XB, 3,
                                                   0, 0)>;
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

## Key Concepts / 关键概念

- PowerPC backend integration / PowerPC 后端集成
- TableGen declarations for the backend / 该后端的 TableGen 声明
- LLVM TableGen DSL / LLVM TableGen 领域特定语言
- Instruction selection or opcode handling / 指令选择或操作码处理
- Declarative TableGen records / 声明式 TableGen 记录

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- None explicitly included in the source text / 源码文本中没有显式包含项

### Important Collaborators / 重要协作组件

- Generated backend tables / 生成式后端表
- PowerPC target-specific helpers / PowerPC 目标专用辅助组件
