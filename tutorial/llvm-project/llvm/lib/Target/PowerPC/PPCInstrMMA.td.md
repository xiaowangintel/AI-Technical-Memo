# PPCInstrMMA.td — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/PowerPC/PPCInstrMMA.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file provides TableGen declarations for the backend for the PowerPC backend.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/PowerPC/PPCInstrMMA.td`，主要负责 PowerPC 后端的该后端的 TableGen 声明。 它使用 LLVM TableGen DSL 编写，用于描述声明式目标信息并驱动生成代码。
- **Language**: TableGen DSL / TableGen 领域特定语言

## Line-by-Line Analysis / 逐行分析

### Lines 1-8

```tablegen

// Multiclass definitions for MMA accumulator instructions.
// ----------------------------------------------------------------------------

// Defines 2 unmasked instructions where the xo field for acc/non-acc version
// is even/odd.
multiclass ACC_UM_XOEO<bits<6> opcode, bits<8> xo, dag IOL, string asmbase,
                       string asmstr> {
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file. Notable symbols in this range include `ACC_UM_XOEO`.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。 该区间中较显眼的符号包括 `ACC_UM_XOEO`。

### Lines 9-18

```tablegen
  let Predicates = [MMA, IsNotISAFuture] in {
  def NAME :
    XX3Form_AT3_XAB6<opcode, !or(xo, 0x01), (outs acc:$AT), IOL,
                     !strconcat(asmbase#" ", asmstr), IIC_VecFP, []>,
    RegConstraint<"@earlyclobber $AT">;
  def PP :
    XX3Form_AT3_XAB6<opcode, xo, (outs acc:$AT), !con((ins acc:$ATi), IOL),
                     !strconcat(asmbase#"pp ", asmstr), IIC_VecFP, []>,
    RegConstraint<"$ATi = $AT">;
  }
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend TableGen declarations for the backend. These declarations feed generated pattern-matching logic.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 TableGen 声明所需的辅助例程。 这些声明会进入生成式模式匹配逻辑。

### Lines 19-32

```tablegen
  let Predicates = [MMA, IsISAFuture], isCodeGenOnly = 1 in {
  def NAME#W :
    XX3Form_AT3_XAB6<opcode, !or(xo, 0x01), (outs wacc:$AT), IOL,
                     !strconcat(asmbase#" ", asmstr), IIC_VecFP, []>,
    RegConstraint<"@earlyclobber $AT">;
  def WPP :
    XX3Form_AT3_XAB6<opcode, xo, (outs wacc:$AT), !con((ins wacc:$ATi), IOL),
                     !strconcat(asmbase#"pp ", asmstr), IIC_VecFP, []>,
    RegConstraint<"$ATi = $AT">;
  }
}

// Defines 4 instructions, masked/unmasked with masks 8, 4, 4 bits.
// The XO field for acc/non-acc version is even/odd.
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend TableGen declarations for the backend. These declarations feed generated pattern-matching logic.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 TableGen 声明所需的辅助例程。 这些声明会进入生成式模式匹配逻辑。

### Lines 33-43

```tablegen
multiclass ACC_UM_M844_XOEO<bits<6> opcode, bits<8> xo, dag IOL, string asmbase,
                            string asmstr> {
  defm NAME : ACC_UM_XOEO<opcode, xo, IOL, asmbase, asmstr>;
  let Predicates = [MMA, PrefixInstrs, IsNotISAFuture] in {
  def PM#NAME :
    MMIRR_XX3Form_XY4P8_XAB6<
      opcode, !or(xo, 0x01), (outs acc:$AT),
      !con(IOL, (ins u4imm:$XMSK, u4imm:$YMSK, u8imm:$PMSK)),
      !strconcat("pm"#asmbase#" ", asmstr#", $XMSK, $YMSK, $PMSK"),
      IIC_VecFP, []>,
    RegConstraint<"@earlyclobber $AT">;
```
- **EN**: Adds declarative TableGen records such as `ACC_UM_M844_XOEO` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ACC_UM_M844_XOEO`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 44-52

```tablegen
  def PM#NAME#PP :
    MMIRR_XX3Form_XY4P8_XAB6<
      opcode, xo, (outs acc:$AT),
      !con((ins acc:$ATi),
           !con(IOL, (ins u4imm:$XMSK, u4imm:$YMSK, u8imm:$PMSK))),
      !strconcat("pm"#asmbase#"pp ", asmstr#", $XMSK, $YMSK, $PMSK"),
      IIC_VecFP, []>,
    RegConstraint<"$ATi = $AT">;
  }
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 53-60

```tablegen
  let Predicates = [MMA, PrefixInstrs, IsISAFuture], isCodeGenOnly = 1 in {
  def PM#NAME#W :
    MMIRR_XX3Form_XY4P8_XAB6<
      opcode, !or(xo, 0x01), (outs wacc:$AT),
      !con(IOL, (ins u4imm:$XMSK, u4imm:$YMSK, u8imm:$PMSK)),
      !strconcat("pm"#asmbase#" ", asmstr#", $XMSK, $YMSK, $PMSK"),
      IIC_VecFP, []>,
    RegConstraint<"@earlyclobber $AT">;
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend TableGen declarations for the backend. These declarations feed generated pattern-matching logic.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 TableGen 声明所需的辅助例程。 这些声明会进入生成式模式匹配逻辑。

### Lines 61-73

```tablegen
  def PM#NAME#WPP :
    MMIRR_XX3Form_XY4P8_XAB6<
      opcode, xo, (outs wacc:$AT),
      !con((ins wacc:$ATi),
           !con(IOL, (ins u4imm:$XMSK, u4imm:$YMSK, u8imm:$PMSK))),
      !strconcat("pm"#asmbase#"pp ", asmstr#", $XMSK, $YMSK, $PMSK"),
      IIC_VecFP, []>,
    RegConstraint<"$ATi = $AT">;
  }
}

// Defines 4 instructions, masked/unmasked with masks 4, 4, 4 bits.
// The XO field for acc/non-acc version is even/odd.
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 74-84

```tablegen
multiclass ACC_UM_M444_XOEO<bits<6> opcode, bits<8> xo, dag IOL, string asmbase,
                            string asmstr> {
  defm NAME : ACC_UM_XOEO<opcode, xo, IOL, asmbase, asmstr>;
  let Predicates = [MMA, PrefixInstrs, IsNotISAFuture] in {
  def PM#NAME :
    MMIRR_XX3Form_XYP4_XAB6<
      opcode, !or(xo, 0x01), (outs acc:$AT),
      !con(IOL, (ins u4imm:$XMSK, u4imm:$YMSK, u4imm:$PMSK)),
      !strconcat("pm"#asmbase#" ", asmstr#", $XMSK, $YMSK, $PMSK"),
      IIC_VecFP, []>,
    RegConstraint<"@earlyclobber $AT">;
```
- **EN**: Adds declarative TableGen records such as `ACC_UM_M444_XOEO` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ACC_UM_M444_XOEO`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 85-93

```tablegen
  def PM#NAME#PP :
    MMIRR_XX3Form_XYP4_XAB6<
      opcode, xo, (outs acc:$AT),
      !con((ins acc:$ATi),
           !con(IOL, (ins u4imm:$XMSK, u4imm:$YMSK, u4imm:$PMSK))),
      !strconcat("pm"#asmbase#"pp ", asmstr#", $XMSK, $YMSK, $PMSK"),
      IIC_VecFP, []>,
    RegConstraint<"$ATi = $AT">;
  }
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 94-101

```tablegen
  let Predicates = [MMA, PrefixInstrs, IsISAFuture], isCodeGenOnly = 1 in {
  def PM#NAME#W :
    MMIRR_XX3Form_XYP4_XAB6<
      opcode, !or(xo, 0x01), (outs wacc:$AT),
      !con(IOL, (ins u4imm:$XMSK, u4imm:$YMSK, u4imm:$PMSK)),
      !strconcat("pm"#asmbase#" ", asmstr#", $XMSK, $YMSK, $PMSK"),
      IIC_VecFP, []>,
    RegConstraint<"@earlyclobber $AT">;
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend TableGen declarations for the backend. These declarations feed generated pattern-matching logic.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 TableGen 声明所需的辅助例程。 这些声明会进入生成式模式匹配逻辑。

### Lines 102-114

```tablegen
  def PM#NAME#WPP :
    MMIRR_XX3Form_XYP4_XAB6<
      opcode, xo, (outs wacc:$AT),
      !con((ins wacc:$ATi),
           !con(IOL, (ins u4imm:$XMSK, u4imm:$YMSK, u4imm:$PMSK))),
      !strconcat("pm"#asmbase#"pp ", asmstr#", $XMSK, $YMSK, $PMSK"),
      IIC_VecFP, []>,
    RegConstraint<"$ATi = $AT">;
  }
}

// Defines 4 instructions, masked/unmasked with masks 2, 4, 4 bits.
// The XO field for acc/non-acc version is even/odd.
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 115-125

```tablegen
multiclass ACC_UM_M244_XOEO<bits<6> opcode, bits<8> xo, dag IOL, string asmbase,
                            string asmstr> {
  defm NAME : ACC_UM_XOEO<opcode, xo, IOL, asmbase, asmstr>;
  let Predicates = [MMA, PrefixInstrs, IsNotISAFuture] in {
  def PM#NAME :
    MMIRR_XX3Form_XY4P2_XAB6<
      opcode, !or(xo, 0x01), (outs acc:$AT),
      !con(IOL, (ins u4imm:$XMSK, u4imm:$YMSK, u2imm:$PMSK)),
      !strconcat("pm"#asmbase#" ", asmstr#", $XMSK, $YMSK, $PMSK"),
      IIC_VecFP, []>,
    RegConstraint<"@earlyclobber $AT">;
```
- **EN**: Adds declarative TableGen records such as `ACC_UM_M244_XOEO` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ACC_UM_M244_XOEO`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 126-133

```tablegen
  def PM#NAME#PP :
    MMIRR_XX3Form_XY4P2_XAB6<
      opcode, xo, (outs acc:$AT),
      !con((ins acc:$ATi), !con(IOL, (ins u4imm:$XMSK, u4imm:$YMSK, u2imm:$PMSK))),
      !strconcat("pm"#asmbase#"pp ", asmstr#", $XMSK, $YMSK, $PMSK"),
      IIC_VecFP, []>,
    RegConstraint<"$ATi = $AT">;
  }
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 134-141

```tablegen
  let Predicates = [MMA, PrefixInstrs, IsISAFuture], isCodeGenOnly = 1 in {
  def PM#NAME#W :
    MMIRR_XX3Form_XY4P2_XAB6<
      opcode, !or(xo, 0x01), (outs wacc:$AT),
      !con(IOL, (ins u4imm:$XMSK, u4imm:$YMSK, u2imm:$PMSK)),
      !strconcat("pm"#asmbase#" ", asmstr#", $XMSK, $YMSK, $PMSK"),
      IIC_VecFP, []>,
    RegConstraint<"@earlyclobber $AT">;
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend TableGen declarations for the backend. These declarations feed generated pattern-matching logic.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 TableGen 声明所需的辅助例程。 这些声明会进入生成式模式匹配逻辑。

### Lines 142-153

```tablegen
  def PM#NAME#WPP :
    MMIRR_XX3Form_XY4P2_XAB6<
      opcode, xo, (outs wacc:$AT),
      !con((ins wacc:$ATi), !con(IOL, (ins u4imm:$XMSK, u4imm:$YMSK, u2imm:$PMSK))),
      !strconcat("pm"#asmbase#"pp ", asmstr#", $XMSK, $YMSK, $PMSK"),
      IIC_VecFP, []>,
    RegConstraint<"$ATi = $AT">;
  }
}

// Defines 4 instructions, masked/unmasked with masks 2, 4, 4 bits.
// Upper nibble of XO field for acc/non-acc version is 0x4/0x6.
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 154-160

```tablegen
multiclass ACC_UM_M244_XO46<bits<6> opcode, bits<8> xo, dag IOL, string asmbase,
                            string asmstr> {
  let Predicates = [MMA, IsNotISAFuture] in {
  def NAME :
    XX3Form_AT3_XAB6<opcode, xo, (outs acc:$AT), IOL,
                     !strconcat(asmbase#" ", asmstr), IIC_VecFP, []>,
    RegConstraint<"@earlyclobber $AT">;
```
- **EN**: Adds declarative TableGen records such as `ACC_UM_M244_XO46` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ACC_UM_M244_XO46`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 161-167

```tablegen
  def PP :
    XX3Form_AT3_XAB6<
      opcode, !or(xo, 0x20), (outs acc:$AT), !con((ins acc:$ATi), IOL),
      !strconcat(asmbase#"pp ", asmstr), IIC_VecFP, []>,
    RegConstraint<"$ATi = $AT">;
  }
  let Predicates = [MMA, PrefixInstrs, IsNotISAFuture] in {
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend TableGen declarations for the backend. These declarations feed generated pattern-matching logic.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 TableGen 声明所需的辅助例程。 这些声明会进入生成式模式匹配逻辑。

### Lines 168-174

```tablegen
  def PM#NAME :
    MMIRR_XX3Form_XY4P2_XAB6<
      opcode, xo, (outs acc:$AT),
      !con(IOL, (ins u4imm:$XMSK, u4imm:$YMSK, u2imm:$PMSK)),
      !strconcat("pm"#asmbase#" ", asmstr#", $XMSK, $YMSK, $PMSK"),
      IIC_VecFP, []>,
    RegConstraint<"@earlyclobber $AT">;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 175-183

```tablegen
  def PM#NAME#PP :
    MMIRR_XX3Form_XY4P2_XAB6<
      opcode, !or(xo, 0x20), (outs acc:$AT),
      !con((ins acc:$ATi),
           !con(IOL, (ins u4imm:$XMSK, u4imm:$YMSK, u2imm:$PMSK))),
      !strconcat("pm"#asmbase#"pp ", asmstr#", $XMSK, $YMSK, $PMSK"),
      IIC_VecFP, []>,
    RegConstraint<"$ATi = $AT">;
  }
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 184-194

```tablegen
  let Predicates = [MMA, IsISAFuture], isCodeGenOnly = 1 in {
  def NAME#W :
    XX3Form_AT3_XAB6<opcode, xo, (outs wacc:$AT), IOL,
                     !strconcat(asmbase#" ", asmstr), IIC_VecFP, []>,
    RegConstraint<"@earlyclobber $AT">;
  def WPP :
    XX3Form_AT3_XAB6<
      opcode, !or(xo, 0x20), (outs wacc:$AT), !con((ins wacc:$ATi), IOL),
      !strconcat(asmbase#"pp ", asmstr), IIC_VecFP, []>,
    RegConstraint<"$ATi = $AT">;
  }
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend TableGen declarations for the backend. These declarations feed generated pattern-matching logic.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 TableGen 声明所需的辅助例程。 这些声明会进入生成式模式匹配逻辑。

### Lines 195-202

```tablegen
  let Predicates = [MMA, PrefixInstrs, IsISAFuture], isCodeGenOnly = 1 in {
  def PM#NAME#W :
    MMIRR_XX3Form_XY4P2_XAB6<
      opcode, xo, (outs wacc:$AT),
      !con(IOL, (ins u4imm:$XMSK, u4imm:$YMSK, u2imm:$PMSK)),
      !strconcat("pm"#asmbase#" ", asmstr#", $XMSK, $YMSK, $PMSK"),
      IIC_VecFP, []>,
    RegConstraint<"@earlyclobber $AT">;
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend TableGen declarations for the backend. These declarations feed generated pattern-matching logic.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 TableGen 声明所需的辅助例程。 这些声明会进入生成式模式匹配逻辑。

### Lines 203-215

```tablegen
  def PM#NAME#WPP :
    MMIRR_XX3Form_XY4P2_XAB6<
      opcode, !or(xo, 0x20), (outs wacc:$AT),
      !con((ins wacc:$ATi),
           !con(IOL, (ins u4imm:$XMSK, u4imm:$YMSK, u2imm:$PMSK))),
      !strconcat("pm"#asmbase#"pp ", asmstr#", $XMSK, $YMSK, $PMSK"),
      IIC_VecFP, []>,
    RegConstraint<"$ATi = $AT">;
  }
}

// Defines 10 instructions, operand negating, unmasked, masked with 2, 4, 4
// bits. Upper nibble are masked with 0x8, 0x4, 0xC for negating operands.
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 216-223

```tablegen
multiclass ACC_NEG_UM_M244_XOM84C<bits<6> opcode, bits<8> xo, dag IOL,
                                  string asmbase, string asmstr> {
  defm NAME : ACC_UM_M244_XOEO<opcode, xo, IOL, asmbase, asmstr>;
  let Predicates = [MMA, IsNotISAFuture] in {
  def PN : XX3Form_AT3_XAB6<
             opcode, !or(xo, 0x80), (outs acc:$AT), !con((ins acc:$ATi), IOL),
             !strconcat(asmbase#"pn ", asmstr), IIC_VecFP, []>,
           RegConstraint<"$ATi = $AT">;
```
- **EN**: Adds declarative TableGen records such as `ACC_NEG_UM_M244_XOM84C` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ACC_NEG_UM_M244_XOM84C`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 224-232

```tablegen
  def NP : XX3Form_AT3_XAB6<
             opcode, !or(xo, 0x40), (outs acc:$AT), !con((ins acc:$ATi), IOL),
             !strconcat(asmbase#"np ", asmstr), IIC_VecFP, []>,
           RegConstraint<"$ATi = $AT">;
  def NN : XX3Form_AT3_XAB6<
             opcode, !or(xo, 0xC0), (outs acc:$AT), !con((ins acc:$ATi), IOL),
             !strconcat(asmbase#"nn ", asmstr), IIC_VecFP, []>,
           RegConstraint<"$ATi = $AT">;
  }
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 233-241

```tablegen
  let Predicates = [MMA, IsISAFuture], isCodeGenOnly = 1 in {
  def WPN : XX3Form_AT3_XAB6<
              opcode, !or(xo, 0x80), (outs wacc:$AT), !con((ins wacc:$ATi), IOL),
              !strconcat(asmbase#"pn ", asmstr), IIC_VecFP, []>,
           RegConstraint<"$ATi = $AT">;
  def WNP : XX3Form_AT3_XAB6<
              opcode, !or(xo, 0x40), (outs wacc:$AT), !con((ins wacc:$ATi), IOL),
              !strconcat(asmbase#"np ", asmstr), IIC_VecFP, []>,
           RegConstraint<"$ATi = $AT">;
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend TableGen declarations for the backend. These declarations feed generated pattern-matching logic.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 TableGen 声明所需的辅助例程。 这些声明会进入生成式模式匹配逻辑。

### Lines 242-254

```tablegen
  def WNN : XX3Form_AT3_XAB6<
              opcode, !or(xo, 0xC0), (outs wacc:$AT), !con((ins wacc:$ATi), IOL),
              !strconcat(asmbase#"nn ", asmstr), IIC_VecFP, []>,
           RegConstraint<"$ATi = $AT">;
  }
  let Predicates = [MMA, PrefixInstrs, IsNotISAFuture] in {
  def PM#NAME#PN :
    MMIRR_XX3Form_XY4P2_XAB6<
      opcode, !or(xo, 0x80), (outs acc:$AT),
      !con((ins acc:$ATi), !con(IOL, (ins u4imm:$XMSK, u4imm:$YMSK, u2imm:$PMSK))),
      !strconcat("pm"#asmbase#"pn ", asmstr#", $XMSK, $YMSK, $PMSK"),
      IIC_VecFP, []>,
    RegConstraint<"$ATi = $AT">;
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend TableGen declarations for the backend. These declarations feed generated pattern-matching logic.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 TableGen 声明所需的辅助例程。 这些声明会进入生成式模式匹配逻辑。

### Lines 255-261

```tablegen
  def PM#NAME#NP :
    MMIRR_XX3Form_XY4P2_XAB6<
      opcode, !or(xo, 0x40), (outs acc:$AT),
      !con((ins acc:$ATi), !con(IOL, (ins u4imm:$XMSK, u4imm:$YMSK, u2imm:$PMSK))),
      !strconcat("pm"#asmbase#"np ", asmstr#", $XMSK, $YMSK, $PMSK"),
      IIC_VecFP, []>,
    RegConstraint<"$ATi = $AT">;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 262-269

```tablegen
  def PM#NAME#NN :
    MMIRR_XX3Form_XY4P2_XAB6<
      opcode, !or(xo, 0xC0), (outs acc:$AT),
      !con((ins acc:$ATi), !con(IOL, (ins u4imm:$XMSK, u4imm:$YMSK, u2imm:$PMSK))),
      !strconcat("pm"#asmbase#"nn ", asmstr#", $XMSK, $YMSK, $PMSK"),
      IIC_VecFP, []>,
    RegConstraint<"$ATi = $AT">;
  }
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 270-277

```tablegen
  let Predicates = [MMA, PrefixInstrs, IsISAFuture], isCodeGenOnly = 1 in {
  def PM#NAME#WPN :
    MMIRR_XX3Form_XY4P2_XAB6<
      opcode, !or(xo, 0x80), (outs wacc:$AT),
      !con((ins wacc:$ATi), !con(IOL, (ins u4imm:$XMSK, u4imm:$YMSK, u2imm:$PMSK))),
      !strconcat("pm"#asmbase#"pn ", asmstr#", $XMSK, $YMSK, $PMSK"),
      IIC_VecFP, []>,
    RegConstraint<"$ATi = $AT">;
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend TableGen declarations for the backend. These declarations feed generated pattern-matching logic.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 TableGen 声明所需的辅助例程。 这些声明会进入生成式模式匹配逻辑。

### Lines 278-284

```tablegen
  def PM#NAME#WNP :
    MMIRR_XX3Form_XY4P2_XAB6<
      opcode, !or(xo, 0x40), (outs wacc:$AT),
      !con((ins wacc:$ATi), !con(IOL, (ins u4imm:$XMSK, u4imm:$YMSK, u2imm:$PMSK))),
      !strconcat("pm"#asmbase#"np ", asmstr#", $XMSK, $YMSK, $PMSK"),
      IIC_VecFP, []>,
    RegConstraint<"$ATi = $AT">;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 285-296

```tablegen
  def PM#NAME#WNN :
    MMIRR_XX3Form_XY4P2_XAB6<
      opcode, !or(xo, 0xC0), (outs wacc:$AT),
      !con((ins wacc:$ATi), !con(IOL, (ins u4imm:$XMSK, u4imm:$YMSK, u2imm:$PMSK))),
      !strconcat("pm"#asmbase#"nn ", asmstr#", $XMSK, $YMSK, $PMSK"),
      IIC_VecFP, []>,
    RegConstraint<"$ATi = $AT">;
  }
}

// Defines 5 instructions, unmasked, operand negating.
// Upper nibble are masked with 0x8, 0x4, 0xC for negating operands.
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 297-304

```tablegen
multiclass ACC_NEG_UM_XOM84C<bits<6> opcode, bits<8> xo, dag IOL,
                             string asmbase, string asmstr> {
  defm NAME : ACC_UM_XOEO<opcode, xo, IOL, asmbase, asmstr>;
  let Predicates = [MMA, IsNotISAFuture] in {
  def PN : XX3Form_AT3_XAB6<opcode, !or(xo, 0x80), (outs acc:$AT),
                            !con((ins acc:$ATi), IOL),
                            !strconcat(asmbase#"pn ", asmstr), IIC_VecFP, []>,
           RegConstraint<"$ATi = $AT">;
```
- **EN**: Adds declarative TableGen records such as `ACC_NEG_UM_XOM84C` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ACC_NEG_UM_XOM84C`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 305-313

```tablegen
  def NP : XX3Form_AT3_XAB6<opcode, !or(xo, 0x40), (outs acc:$AT),
                            !con((ins acc:$ATi), IOL),
                            !strconcat(asmbase#"np ", asmstr), IIC_VecFP, []>,
           RegConstraint<"$ATi = $AT">;
  def NN : XX3Form_AT3_XAB6<opcode, !or(xo, 0xC0), (outs acc:$AT),
                            !con((ins acc:$ATi), IOL),
                            !strconcat(asmbase#"nn ", asmstr), IIC_VecFP, []>,
           RegConstraint<"$ATi = $AT">;
  }
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 314-322

```tablegen
  let Predicates = [MMA, IsISAFuture], isCodeGenOnly = 1 in {
  def WPN : XX3Form_AT3_XAB6<opcode, !or(xo, 0x80), (outs wacc:$AT),
                            !con((ins wacc:$ATi), IOL),
                            !strconcat(asmbase#"pn ", asmstr), IIC_VecFP, []>,
           RegConstraint<"$ATi = $AT">;
  def WNP : XX3Form_AT3_XAB6<opcode, !or(xo, 0x40), (outs wacc:$AT),
                            !con((ins wacc:$ATi), IOL),
                            !strconcat(asmbase#"np ", asmstr), IIC_VecFP, []>,
           RegConstraint<"$ATi = $AT">;
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend TableGen declarations for the backend. These declarations feed generated pattern-matching logic.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 TableGen 声明所需的辅助例程。 这些声明会进入生成式模式匹配逻辑。

### Lines 323-331

```tablegen
  def WNN : XX3Form_AT3_XAB6<opcode, !or(xo, 0xC0), (outs wacc:$AT),
                            !con((ins wacc:$ATi), IOL),
                            !strconcat(asmbase#"nn ", asmstr), IIC_VecFP, []>,
           RegConstraint<"$ATi = $AT">;
  }
}

// Defines 10 instructions, operand negating, unmasked, masked with 4, 4 bits.
// Upper nibble are masked with 0x8, 0x4, 0xC for negating operands.
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 332-342

```tablegen
multiclass ACC_NEG_UM_M44_XOM84C<bits<6> opcode, bits<8> xo, dag IOL,
                                 string asmbase, string asmstr> {
  defm NAME : ACC_NEG_UM_XOM84C<opcode, xo, IOL, asmbase, asmstr>;
  let Predicates = [MMA, PrefixInstrs, IsNotISAFuture] in {
  def PM#NAME :
    MMIRR_XX3Form_XY4_XAB6<
      opcode, !or(xo, 0x01), (outs acc:$AT),
      !con(IOL, (ins u4imm:$XMSK, u4imm:$YMSK)),
      !strconcat("pm"#asmbase#" ", asmstr#", $XMSK, $YMSK"),
      IIC_VecFP, []>,
    RegConstraint<"@earlyclobber $AT">;
```
- **EN**: Adds declarative TableGen records such as `ACC_NEG_UM_M44_XOM84C` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ACC_NEG_UM_M44_XOM84C`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 343-349

```tablegen
  def PM#NAME#PP :
    MMIRR_XX3Form_XY4_XAB6<
      opcode, xo, (outs acc:$AT),
      !con((ins acc:$ATi), !con(IOL, (ins u4imm:$XMSK, u4imm:$YMSK))),
      !strconcat("pm"#asmbase#"pp ", asmstr#", $XMSK, $YMSK"),
      IIC_VecFP, []>,
    RegConstraint<"$ATi = $AT">;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 350-356

```tablegen
  def PM#NAME#PN :
    MMIRR_XX3Form_XY4_XAB6<
      opcode, !or(xo, 0x80), (outs acc:$AT),
      !con((ins acc:$ATi), !con(IOL, (ins u4imm:$XMSK, u4imm:$YMSK))),
      !strconcat("pm"#asmbase#"pn ", asmstr#", $XMSK, $YMSK"),
      IIC_VecFP, []>,
    RegConstraint<"$ATi = $AT">;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 357-363

```tablegen
  def PM#NAME#NP :
    MMIRR_XX3Form_XY4_XAB6<
      opcode, !or(xo, 0x40), (outs acc:$AT),
      !con((ins acc:$ATi), !con(IOL, (ins u4imm:$XMSK, u4imm:$YMSK))),
      !strconcat("pm"#asmbase#"np ", asmstr#", $XMSK, $YMSK"),
      IIC_VecFP, []>,
    RegConstraint<"$ATi = $AT">;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 364-371

```tablegen
  def PM#NAME#NN :
    MMIRR_XX3Form_XY4_XAB6<
      opcode, !or(xo, 0xC0), (outs acc:$AT),
      !con((ins acc:$ATi), !con(IOL, (ins u4imm:$XMSK, u4imm:$YMSK))),
      !strconcat("pm"#asmbase#"nn ", asmstr#", $XMSK, $YMSK"),
      IIC_VecFP, []>,
    RegConstraint<"$ATi = $AT">;
  }
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 372-379

```tablegen
  let Predicates = [MMA, PrefixInstrs, IsISAFuture], isCodeGenOnly = 1 in {
  def PM#NAME#W :
    MMIRR_XX3Form_XY4_XAB6<
      opcode, !or(xo, 0x01), (outs wacc:$AT),
      !con(IOL, (ins u4imm:$XMSK, u4imm:$YMSK)),
      !strconcat("pm"#asmbase#" ", asmstr#", $XMSK, $YMSK"),
      IIC_VecFP, []>,
    RegConstraint<"@earlyclobber $AT">;
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend TableGen declarations for the backend. These declarations feed generated pattern-matching logic.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 TableGen 声明所需的辅助例程。 这些声明会进入生成式模式匹配逻辑。

### Lines 380-386

```tablegen
  def PM#NAME#WPP :
    MMIRR_XX3Form_XY4_XAB6<
      opcode, xo, (outs wacc:$AT),
      !con((ins wacc:$ATi), !con(IOL, (ins u4imm:$XMSK, u4imm:$YMSK))),
      !strconcat("pm"#asmbase#"pp ", asmstr#", $XMSK, $YMSK"),
      IIC_VecFP, []>,
    RegConstraint<"$ATi = $AT">;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 387-393

```tablegen
  def PM#NAME#WPN :
    MMIRR_XX3Form_XY4_XAB6<
      opcode, !or(xo, 0x80), (outs wacc:$AT),
      !con((ins wacc:$ATi), !con(IOL, (ins u4imm:$XMSK, u4imm:$YMSK))),
      !strconcat("pm"#asmbase#"pn ", asmstr#", $XMSK, $YMSK"),
      IIC_VecFP, []>,
    RegConstraint<"$ATi = $AT">;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 394-400

```tablegen
  def PM#NAME#WNP :
    MMIRR_XX3Form_XY4_XAB6<
      opcode, !or(xo, 0x40), (outs wacc:$AT),
      !con((ins wacc:$ATi), !con(IOL, (ins u4imm:$XMSK, u4imm:$YMSK))),
      !strconcat("pm"#asmbase#"np ", asmstr#", $XMSK, $YMSK"),
      IIC_VecFP, []>,
    RegConstraint<"$ATi = $AT">;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 401-412

```tablegen
  def PM#NAME#WNN :
    MMIRR_XX3Form_XY4_XAB6<
      opcode, !or(xo, 0xC0), (outs wacc:$AT),
      !con((ins wacc:$ATi), !con(IOL, (ins u4imm:$XMSK, u4imm:$YMSK))),
      !strconcat("pm"#asmbase#"nn ", asmstr#", $XMSK, $YMSK"),
      IIC_VecFP, []>,
    RegConstraint<"$ATi = $AT">;
  }
}

// Defines 10 instructions, operand negating, unmasked, masked with 4, 2 bits.
// Upper nibble are masked with 0x8, 0x4, 0xC for negating operands.
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 413-423

```tablegen
multiclass ACC_NEG_UM_M42_XOM84C<bits<6> opcode, bits<8> xo, dag IOL,
                                 string asmbase, string asmstr> {
  defm NAME : ACC_NEG_UM_XOM84C<opcode, xo, IOL, asmbase, asmstr>;
  let Predicates = [MMA, PrefixInstrs, IsNotISAFuture] in {
  def PM#NAME :
    MMIRR_XX3Form_X4Y2_XAB6<
      opcode, !or(xo, 0x01), (outs acc:$AT),
      !con(IOL, (ins u4imm:$XMSK, u2imm:$YMSK)),
      !strconcat("pm"#asmbase#" ", asmstr#", $XMSK, $YMSK"),
      IIC_VecFP, []>,
    RegConstraint<"@earlyclobber $AT">;
```
- **EN**: Adds declarative TableGen records such as `ACC_NEG_UM_M42_XOM84C` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ACC_NEG_UM_M42_XOM84C`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 424-430

```tablegen
  def PM#NAME#PP :
    MMIRR_XX3Form_X4Y2_XAB6<
      opcode, xo, (outs acc:$AT),
      !con((ins acc:$ATi), !con(IOL, (ins u4imm:$XMSK, u2imm:$YMSK))),
      !strconcat("pm"#asmbase#"pp ", asmstr#", $XMSK, $YMSK"),
      IIC_VecFP, []>,
    RegConstraint<"$ATi = $AT">;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 431-437

```tablegen
  def PM#NAME#PN :
    MMIRR_XX3Form_X4Y2_XAB6<
      opcode, !or(xo, 0x80), (outs acc:$AT),
      !con((ins acc:$ATi), !con(IOL, (ins u4imm:$XMSK, u2imm:$YMSK))),
      !strconcat("pm"#asmbase#"pn ", asmstr#", $XMSK, $YMSK"),
      IIC_VecFP, []>,
    RegConstraint<"$ATi = $AT">;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 438-444

```tablegen
  def PM#NAME#NP :
    MMIRR_XX3Form_X4Y2_XAB6<
      opcode, !or(xo, 0x40), (outs acc:$AT),
      !con((ins acc:$ATi), !con(IOL, (ins u4imm:$XMSK, u2imm:$YMSK))),
      !strconcat("pm"#asmbase#"np ", asmstr#", $XMSK, $YMSK"),
      IIC_VecFP, []>,
    RegConstraint<"$ATi = $AT">;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 445-452

```tablegen
  def PM#NAME#NN :
    MMIRR_XX3Form_X4Y2_XAB6<
      opcode, !or(xo, 0xC0), (outs acc:$AT),
      !con((ins acc:$ATi), !con(IOL, (ins u4imm:$XMSK, u2imm:$YMSK))),
      !strconcat("pm"#asmbase#"nn ", asmstr#", $XMSK, $YMSK"),
      IIC_VecFP, []>,
    RegConstraint<"$ATi = $AT">;
  }
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 453-460

```tablegen
  let Predicates = [MMA, PrefixInstrs, IsISAFuture], isCodeGenOnly = 1 in {
  def PM#NAME#W :
    MMIRR_XX3Form_X4Y2_XAB6<
      opcode, !or(xo, 0x01), (outs wacc:$AT),
      !con(IOL, (ins u4imm:$XMSK, u2imm:$YMSK)),
      !strconcat("pm"#asmbase#" ", asmstr#", $XMSK, $YMSK"),
      IIC_VecFP, []>,
    RegConstraint<"@earlyclobber $AT">;
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend TableGen declarations for the backend. These declarations feed generated pattern-matching logic.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 TableGen 声明所需的辅助例程。 这些声明会进入生成式模式匹配逻辑。

### Lines 461-467

```tablegen
  def PM#NAME#WPP :
    MMIRR_XX3Form_X4Y2_XAB6<
      opcode, xo, (outs wacc:$AT),
      !con((ins wacc:$ATi), !con(IOL, (ins u4imm:$XMSK, u2imm:$YMSK))),
      !strconcat("pm"#asmbase#"pp ", asmstr#", $XMSK, $YMSK"),
      IIC_VecFP, []>,
    RegConstraint<"$ATi = $AT">;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 468-474

```tablegen
  def PM#NAME#WPN :
    MMIRR_XX3Form_X4Y2_XAB6<
      opcode, !or(xo, 0x80), (outs wacc:$AT),
      !con((ins wacc:$ATi), !con(IOL, (ins u4imm:$XMSK, u2imm:$YMSK))),
      !strconcat("pm"#asmbase#"pn ", asmstr#", $XMSK, $YMSK"),
      IIC_VecFP, []>,
    RegConstraint<"$ATi = $AT">;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 475-481

```tablegen
  def PM#NAME#WNP :
    MMIRR_XX3Form_X4Y2_XAB6<
      opcode, !or(xo, 0x40), (outs wacc:$AT),
      !con((ins wacc:$ATi), !con(IOL, (ins u4imm:$XMSK, u2imm:$YMSK))),
      !strconcat("pm"#asmbase#"np ", asmstr#", $XMSK, $YMSK"),
      IIC_VecFP, []>,
    RegConstraint<"$ATi = $AT">;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 482-494

```tablegen
  def PM#NAME#WNN :
    MMIRR_XX3Form_X4Y2_XAB6<
      opcode, !or(xo, 0xC0), (outs wacc:$AT),
      !con((ins wacc:$ATi), !con(IOL, (ins u4imm:$XMSK, u2imm:$YMSK))),
      !strconcat("pm"#asmbase#"nn ", asmstr#", $XMSK, $YMSK"),
      IIC_VecFP, []>,
    RegConstraint<"$ATi = $AT">;
  }
}

// End of class definitions.
//-----------------------------------------------------------------------------
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 495-505

```tablegen
let Predicates = [MMA, IsNotISAFuture] in {
  def XXMFACC :
    XForm_AT3<31, 0, 177, (outs acc:$ATo), (ins acc:$AT), "xxmfacc $AT",
              IIC_VecGeneral,
              [(set v512i1:$ATo, (int_ppc_mma_xxmfacc v512i1:$AT))]>,
              RegConstraint<"$ATo = $AT">;
  def XXMTACC :
    XForm_AT3<31, 1, 177, (outs acc:$AT), (ins acc:$ATi), "xxmtacc $AT",
              IIC_VecGeneral,
              [(set v512i1:$AT, (int_ppc_mma_xxmtacc v512i1:$ATi))]>,
              RegConstraint<"$ATi = $AT">;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 506-513

```tablegen
  def KILL_PAIR : PPCPostRAExpPseudo<(outs vsrprc:$XTp), (ins vsrprc:$XSp),
                                      "#KILL_PAIR", []>,
                                      RegConstraint<"$XTp = $XSp">;
  def BUILD_UACC : PPCPostRAExpPseudo<(outs acc:$AT), (ins uacc:$AS),
                                      "#BUILD_UACC $AT, $AS", []>;
  // We define XXSETACCZ as rematerializable to undo CSE of that intrinsic in
  // the backend. We avoid CSE here because it generates a copy of the acc
  // register and this copy is more expensive than calling the intrinsic again.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "We define XXSETACCZ as rematerializable to undo CSE of that intrinsic in".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“We define XXSETACCZ as rematerializable to undo CSE of that intrinsic in”。

### Lines 514-522

```tablegen
  let isAsCheapAsAMove = 1, isReMaterializable = 1 in {
    def XXSETACCZ :
      XForm_AT3<31, 3, 177, (outs acc:$AT), (ins), "xxsetaccz $AT", IIC_VecGeneral,
                [(set v512i1:$AT, (int_ppc_mma_xxsetaccz))]>;
  }
  def XVI8GER4SPP :
    XX3Form_AT3_XAB6<59, 99, (outs acc:$AT), (ins acc:$ATi, vsrc:$XA, vsrc:$XB),
                     "xvi8ger4spp $AT, $XA, $XB", IIC_VecGeneral, []>,
    RegConstraint<"$ATi = $AT">;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 523-529

```tablegen
  let mayStore = 1 in {
    def SPILL_ACC: PPCEmitTimePseudo<(outs), (ins acc:$AT, memrix16:$dst),
                                     "#SPILL_ACC", []>;
    def SPILL_UACC: PPCEmitTimePseudo<(outs), (ins uacc:$AT, memrix16:$dst),
                                     "#SPILL_UACC", []>;
  }
  let mayLoad = 1, hasSideEffects = 0 in {
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 530-536

```tablegen
    def RESTORE_ACC: PPCEmitTimePseudo<(outs acc:$AT), (ins memrix16:$src),
                                       "#RESTORE_ACC", []>;
    def RESTORE_UACC: PPCEmitTimePseudo<(outs uacc:$AT), (ins memrix16:$src),
                                       "#RESTORE_UACC", []>;
  }
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 537-543

```tablegen
let Predicates = [MMA, IsISAFuture], isCodeGenOnly = 1 in {
  // For Future and up XXMFACCW and XXMTACCW will not have patterns.
  // On Future CPU the wacc registers no longer overlap with the vsr registers
  // and so register allocation would have to know to match 4 vsr registers
  // with one wacc register.
  // On top of that Future CPU has a more convenient way to move between vsrs
  // and wacc registers using xxextfdmr512 and xxinstdmr512.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "For Future and up XXMFACCW and XXMTACCW will not have patterns.". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“For Future and up XXMFACCW and XXMTACCW will not have patterns.”。 这些声明会进入生成式模式匹配逻辑。

### Lines 544-552

```tablegen
  def XXMFACCW :
    XForm_AT3<31, 0, 177, (outs wacc:$ATo), (ins wacc:$AT), "xxmfacc $AT",
              IIC_VecGeneral, []>,
              RegConstraint<"$ATo = $AT">;
  def XXMTACCW :
    XForm_AT3<31, 1, 177, (outs wacc:$AT), (ins wacc:$ATi), "xxmtacc $AT",
              IIC_VecGeneral, []>,
              RegConstraint<"$ATi = $AT">;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 553-564

```tablegen
  let isAsCheapAsAMove = 1, isReMaterializable = 1 in {
    def DMXXSETACCZ :
      XForm_AT3<31, 3, 177, (outs wacc:$AT), (ins), "dmxxsetaccz $AT",
                IIC_VecGeneral, [(set v512i1:$AT, (int_ppc_mma_xxsetaccz))]>;
  }

  def XVI8GER4WSPP :
    XX3Form_AT3_XAB6<59, 99, (outs wacc:$AT),
                     (ins wacc:$ATi, vsrc:$XA, vsrc:$XB),
                     "xvi8ger4spp $AT, $XA, $XB", IIC_VecGeneral, []>,
                     RegConstraint<"$ATi = $AT">;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 565-572

```tablegen
  let mayStore = 1 in {
    def SPILL_WACC: PPCEmitTimePseudo<(outs), (ins wacc:$AT, memrix16:$dst),
                                      "#SPILL_WACC", []>;
    def SPILL_DMRP: PPCEmitTimePseudo<(outs), (ins dmrp:$AT, memrix16:$dst),
                                      "#SPILL_DMRP", []>;
    def SPILL_DMR: PPCEmitTimePseudo<(outs), (ins dmr:$AT, memrix16:$dst),
                                      "#SPILL_DMR", []>;
  }
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 573-582

```tablegen
  let mayLoad = 1, hasSideEffects = 0 in {
    def RESTORE_WACC: PPCEmitTimePseudo<(outs wacc:$AT), (ins memrix16:$src),
                                        "#RESTORE_WACC", []>;
    def RESTORE_DMRP: PPCEmitTimePseudo<(outs dmrp:$AT), (ins memrix16:$src),
                                        "#RESTORE_DMRP", []>;
    def RESTORE_DMR: PPCEmitTimePseudo<(outs dmr:$AT), (ins memrix16:$src),
                                        "#RESTORE_DMR", []>;
  }
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 583-589

```tablegen
let Predicates = [MMA, IsISAFuture] in {
  def : InstAlias<"dmxxmmfacc $AT ", (XXMFACC acc:$AT)>;
  def : InstAlias<"dmxxmmtacc $AT ", (XXMTACC acc:$AT)>;
  def : InstAlias<"dmxxsetaccz $AT ", (XXSETACCZ acc:$AT)>;
}

let Predicates = [MMA, PrefixInstrs, IsNotISAFuture] in {
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 590-598

```tablegen
  def PMXVI8GER4SPP :
    MMIRR_XX3Form_XYP4_XAB6<59, 99, (outs acc:$AT),
                            (ins acc:$ATi, vsrc:$XA,vsrc:$XB, u4imm:$XMSK,
                             u4imm:$YMSK, u4imm:$PMSK),
                            "pmxvi8ger4spp $AT, $XA, $XB, $XMSK, $YMSK, $PMSK",
                            IIC_VecGeneral, []>,
    RegConstraint<"$ATi = $AT">;
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 599-613

```tablegen
let Predicates = [MMA, PrefixInstrs, IsISAFuture], isCodeGenOnly = 1 in {
  def PMXVI8GER4WSPP :
    MMIRR_XX3Form_XYP4_XAB6<59, 99, (outs wacc:$AT),
                            (ins wacc:$ATi, vsrc:$XA,vsrc:$XB, u4imm:$XMSK,
                             u4imm:$YMSK, u4imm:$PMSK),
                            "pmxvi8ger4spp $AT, $XA, $XB, $XMSK, $YMSK, $PMSK",
                            IIC_VecGeneral, []>,
    RegConstraint<"$ATi = $AT">;
}

// MMA accumulating/non-accumulating instructions.
//------------------------------------------------------------------------------

// XVBF16GER2, XVBF16GER2PP, XVBF16GER2PN, XVBF16GER2NP, XVBF16GER2NN
// PMXVBF16GER2, PMXVBF16GER2PP, PMXVBF16GER2PN, PMXVBF16GER2NP, PMXVBF16GER2NN
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 614-621

```tablegen
defm XVBF16GER2 : ACC_NEG_UM_M244_XOM84C<59, 50, (ins vsrc:$XA, vsrc:$XB),
                                         "xvbf16ger2", "$AT, $XA, $XB">;

// XVI4GER8, XVI4GER8PP, PMXVI4GER8,  PMXVI4GER8PP
defm XVI4GER8 : ACC_UM_M844_XOEO<59, 34, (ins vsrc:$XA, vsrc:$XB),
                                 "xvi4ger8", "$AT, $XA, $XB">;

// XVI8GER4, XVI8GER4PP, PMXVI8GER4, PMXVI8GER4PP
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "XVI4GER8, XVI4GER8PP, PMXVI4GER8,  PMXVI4GER8PP". Notable symbols in this range include `XVBF16GER2`, `XVI4GER8`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“XVI4GER8, XVI4GER8PP, PMXVI4GER8,  PMXVI4GER8PP”。 该区间中较显眼的符号包括 `XVBF16GER2`, `XVI4GER8`。

### Lines 622-629

```tablegen
defm XVI8GER4 : ACC_UM_M444_XOEO<59, 2, (ins vsrc:$XA, vsrc:$XB),
                                 "xvi8ger4", "$AT, $XA, $XB">;

// XVI16GER2, XVI16GER2PP, PMXVI16GER2, PMXVI16GER2PP
defm XVI16GER2 : ACC_UM_M244_XO46<59, 75, (ins vsrc:$XA, vsrc:$XB),
                                  "xvi16ger2", "$AT, $XA, $XB">;

// XVI16GER2S, XVI16GER2SPP, PMXVI16GER2S, PMXVI16GER2SPP
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "XVI16GER2, XVI16GER2PP, PMXVI16GER2, PMXVI16GER2PP". Notable symbols in this range include `XVI8GER4`, `XVI16GER2`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“XVI16GER2, XVI16GER2PP, PMXVI16GER2, PMXVI16GER2PP”。 该区间中较显眼的符号包括 `XVI8GER4`, `XVI16GER2`。

### Lines 630-639

```tablegen
defm XVI16GER2S : ACC_UM_M244_XOEO<59, 42, (ins vsrc:$XA, vsrc:$XB),
                                   "xvi16ger2s", "$AT, $XA, $XB">;

// XVF16GER2, XVF16GER2PP, XVF16GER2PN, XVF16GER2NP, XVF16GER2NN
// PMXVF16GER2, PMXVF16GER2PP, PMXVF16GER2PN, PMXVF16GER2NP, PMXVF16GER2NN
defm XVF16GER2 : ACC_NEG_UM_M244_XOM84C<59, 18, (ins vsrc:$XA, vsrc:$XB),
                                        "xvf16ger2", "$AT, $XA, $XB">;

// XVF32GER, XVF32GERPP, XVF32GERPN, XVF32GERNP, XVF32GERPP
// PMXVF32GER, PMXVF32GERPP, PMXVF32GERPN, PMXVF32GERNP, PMXVF32GERPP
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "XVF16GER2, XVF16GER2PP, XVF16GER2PN, XVF16GER2NP, XVF16GER2NN". Notable symbols in this range include `XVI16GER2S`, `XVF16GER2`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“XVF16GER2, XVF16GER2PP, XVF16GER2PN, XVF16GER2NP, XVF16GER2NN”。 该区间中较显眼的符号包括 `XVI16GER2S`, `XVF16GER2`。

### Lines 640-649

```tablegen
defm XVF32GER : ACC_NEG_UM_M44_XOM84C<59, 26, (ins vsrc:$XA, vsrc:$XB),
                                      "xvf32ger", "$AT, $XA, $XB">;

// XVF64GER, XVF64GERPP, XVF64GERPN, XVF64GERNP, XVF64GERNN
// PMXVF64GER, PMXVF64GERPP, PMXVF64GERPN, PMXVF64GERNP, PMXVF64GERNN
defm XVF64GER : ACC_NEG_UM_M42_XOM84C<59, 58, (ins vsrpevenrc:$XA, vsrc:$XB),
                                      "xvf64ger", "$AT, $XA, $XB">;
//------------------------------------------------------------------------------

// MMA Intrinsics
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "XVF64GER, XVF64GERPP, XVF64GERPN, XVF64GERNP, XVF64GERNN". Notable symbols in this range include `XVF32GER`, `XVF64GER`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“XVF64GER, XVF64GERPP, XVF64GERPN, XVF64GERNP, XVF64GERNN”。 该区间中较显眼的符号包括 `XVF32GER`, `XVF64GER`。

### Lines 650-657

```tablegen
let Predicates = [MMA, IsNotISAFuture] in {
  def : Pat<(v512i1 (int_ppc_mma_xvi4ger8 v16i8:$XA, v16i8:$XB)),
            (XVI4GER8 RCCp.AToVSRC, RCCp.BToVSRC)>;
  def : Pat<(v512i1 (int_ppc_mma_xvi4ger8pp v512i1:$ATi, v16i8:$XA, v16i8:$XB)),
            (XVI4GER8PP $ATi, RCCp.AToVSRC, RCCp.BToVSRC)>;

  def : Pat<(v512i1 (int_ppc_mma_xvi8ger4 v16i8:$XA, v16i8:$XB)),
            (XVI8GER4 RCCp.AToVSRC, RCCp.BToVSRC)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 658-666

```tablegen
  def : Pat<(v512i1 (int_ppc_mma_xvi8ger4pp v512i1:$ATi, v16i8:$XA, v16i8:$XB)),
            (XVI8GER4PP $ATi, RCCp.AToVSRC, RCCp.BToVSRC)>;

  def : Pat<(v512i1 (int_ppc_mma_xvi16ger2s v16i8:$XA, v16i8:$XB)),
            (XVI16GER2S RCCp.AToVSRC, RCCp.BToVSRC)>;
  def : Pat<(v512i1 (int_ppc_mma_xvi16ger2spp v512i1:$ATi, v16i8:$XA, v16i8:$XB)),
            (XVI16GER2SPP $ATi, RCCp.AToVSRC, RCCp.BToVSRC)>;
}
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 667-674

```tablegen
let Predicates = [MMA, IsISAFuture] in {
  def : Pat<(v512i1 (int_ppc_mma_xvi4ger8 v16i8:$XA, v16i8:$XB)),
            (XVI4GER8W RCCp.AToVSRC, RCCp.BToVSRC)>;
  def : Pat<(v512i1 (int_ppc_mma_xvi4ger8pp v512i1:$ATi, v16i8:$XA, v16i8:$XB)),
            (XVI4GER8WPP $ATi, RCCp.AToVSRC, RCCp.BToVSRC)>;

  def : Pat<(v512i1 (int_ppc_mma_xvi8ger4 v16i8:$XA, v16i8:$XB)),
            (XVI8GER4W RCCp.AToVSRC, RCCp.BToVSRC)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 675-683

```tablegen
  def : Pat<(v512i1 (int_ppc_mma_xvi8ger4pp v512i1:$ATi, v16i8:$XA, v16i8:$XB)),
            (XVI8GER4WPP $ATi, RCCp.AToVSRC, RCCp.BToVSRC)>;

  def : Pat<(v512i1 (int_ppc_mma_xvi16ger2s v16i8:$XA, v16i8:$XB)),
            (XVI16GER2SW RCCp.AToVSRC, RCCp.BToVSRC)>;
  def : Pat<(v512i1 (int_ppc_mma_xvi16ger2spp v512i1:$ATi, v16i8:$XA, v16i8:$XB)),
            (XVI16GER2SWPP $ATi, RCCp.AToVSRC, RCCp.BToVSRC)>;
}
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 684-690

```tablegen
let Predicates = [MMA, IsNotISAFuture] in {
  def : Pat<(v512i1 (int_ppc_mma_xvf16ger2 v16i8:$XA, v16i8:$XB)),
            (XVF16GER2 RCCp.AToVSRC, RCCp.BToVSRC)>;
  def : Pat<(v512i1 (int_ppc_mma_xvf16ger2pp v512i1:$ATi, v16i8:$XA, v16i8:$XB)),
            (XVF16GER2PP $ATi, RCCp.AToVSRC, RCCp.BToVSRC)>;
  def : Pat<(v512i1 (int_ppc_mma_xvf16ger2pn v512i1:$ATi, v16i8:$XA, v16i8:$XB)),
            (XVF16GER2PN $ATi, RCCp.AToVSRC, RCCp.BToVSRC)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 691-697

```tablegen
  def : Pat<(v512i1 (int_ppc_mma_xvf16ger2np v512i1:$ATi, v16i8:$XA, v16i8:$XB)),
            (XVF16GER2NP $ATi, RCCp.AToVSRC, RCCp.BToVSRC)>;
  def : Pat<(v512i1 (int_ppc_mma_xvf16ger2nn v512i1:$ATi, v16i8:$XA, v16i8:$XB)),
            (XVF16GER2NN $ATi, RCCp.AToVSRC, RCCp.BToVSRC)>;
}

let Predicates = [MMA, IsISAFuture] in {
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 698-705

```tablegen
  def : Pat<(v512i1 (int_ppc_mma_xvf16ger2 v16i8:$XA, v16i8:$XB)),
            (XVF16GER2W RCCp.AToVSRC, RCCp.BToVSRC)>;
  def : Pat<(v512i1 (int_ppc_mma_xvf16ger2pp v512i1:$ATi, v16i8:$XA, v16i8:$XB)),
            (XVF16GER2WPP $ATi, RCCp.AToVSRC, RCCp.BToVSRC)>;
  def : Pat<(v512i1 (int_ppc_mma_xvf16ger2pn v512i1:$ATi, v16i8:$XA, v16i8:$XB)),
            (XVF16GER2WPN $ATi, RCCp.AToVSRC, RCCp.BToVSRC)>;
  def : Pat<(v512i1 (int_ppc_mma_xvf16ger2np v512i1:$ATi, v16i8:$XA, v16i8:$XB)),
            (XVF16GER2WNP $ATi, RCCp.AToVSRC, RCCp.BToVSRC)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 706-712

```tablegen
  def : Pat<(v512i1 (int_ppc_mma_xvf16ger2nn v512i1:$ATi, v16i8:$XA, v16i8:$XB)),
            (XVF16GER2WNN $ATi, RCCp.AToVSRC, RCCp.BToVSRC)>;
}

let Predicates = [MMA, IsNotISAFuture] in {
  def : Pat<(v512i1 (int_ppc_mma_xvf32ger v16i8:$XA, v16i8:$XB)),
            (XVF32GER RCCp.AToVSRC, RCCp.BToVSRC)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 713-720

```tablegen
  def : Pat<(v512i1 (int_ppc_mma_xvf32gerpp v512i1:$ATi, v16i8:$XA, v16i8:$XB)),
            (XVF32GERPP $ATi, RCCp.AToVSRC, RCCp.BToVSRC)>;
  def : Pat<(v512i1 (int_ppc_mma_xvf32gerpn v512i1:$ATi, v16i8:$XA, v16i8:$XB)),
            (XVF32GERPN $ATi, RCCp.AToVSRC, RCCp.BToVSRC)>;
  def : Pat<(v512i1 (int_ppc_mma_xvf32gernp v512i1:$ATi, v16i8:$XA, v16i8:$XB)),
            (XVF32GERNP $ATi, RCCp.AToVSRC, RCCp.BToVSRC)>;
  def : Pat<(v512i1 (int_ppc_mma_xvf32gernn v512i1:$ATi, v16i8:$XA, v16i8:$XB)),
            (XVF32GERNN $ATi, RCCp.AToVSRC, RCCp.BToVSRC)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 721-728

```tablegen
  def : Pat<(v512i1 (int_ppc_mma_xvf64ger v256i1:$XA, v16i8:$XB)),
            (XVF64GER $XA, RCCp.BToVSRC)>;
  def : Pat<(v512i1 (int_ppc_mma_xvf64gerpp v512i1:$ATi, v256i1:$XA, v16i8:$XB)),
            (XVF64GERPP $ATi, $XA, RCCp.BToVSRC)>;
  def : Pat<(v512i1 (int_ppc_mma_xvf64gerpn v512i1:$ATi, v256i1:$XA, v16i8:$XB)),
            (XVF64GERPN $ATi, $XA, RCCp.BToVSRC)>;
  def : Pat<(v512i1 (int_ppc_mma_xvf64gernp v512i1:$ATi, v256i1:$XA, v16i8:$XB)),
            (XVF64GERNP $ATi, $XA, RCCp.BToVSRC)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 729-735

```tablegen
  def : Pat<(v512i1 (int_ppc_mma_xvf64gernn v512i1:$ATi, v256i1:$XA, v16i8:$XB)),
            (XVF64GERNN $ATi, $XA, RCCp.BToVSRC)>;

  def : Pat<(v512i1 (int_ppc_mma_xvbf16ger2 v16i8:$XA, v16i8:$XB)),
            (XVBF16GER2 RCCp.AToVSRC, RCCp.BToVSRC)>;
  def : Pat<(v512i1 (int_ppc_mma_xvbf16ger2pp v512i1:$ATi, v16i8:$XA, v16i8:$XB)),
            (XVBF16GER2PP $ATi, RCCp.AToVSRC, RCCp.BToVSRC)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 736-743

```tablegen
  def : Pat<(v512i1 (int_ppc_mma_xvbf16ger2pn v512i1:$ATi, v16i8:$XA, v16i8:$XB)),
            (XVBF16GER2PN $ATi, RCCp.AToVSRC, RCCp.BToVSRC)>;
  def : Pat<(v512i1 (int_ppc_mma_xvbf16ger2np v512i1:$ATi, v16i8:$XA, v16i8:$XB)),
            (XVBF16GER2NP $ATi, RCCp.AToVSRC, RCCp.BToVSRC)>;
  def : Pat<(v512i1 (int_ppc_mma_xvbf16ger2nn v512i1:$ATi, v16i8:$XA, v16i8:$XB)),
            (XVBF16GER2NN $ATi, RCCp.AToVSRC, RCCp.BToVSRC)>;
  def : Pat<(v512i1 (int_ppc_mma_xvi16ger2 v16i8:$XA, v16i8:$XB)),
            (XVI16GER2 RCCp.AToVSRC, RCCp.BToVSRC)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 744-750

```tablegen
  def : Pat<(v512i1 (int_ppc_mma_xvi16ger2pp v512i1:$ATi, v16i8:$XA, v16i8:$XB)),
            (XVI16GER2PP $ATi, RCCp.AToVSRC, RCCp.BToVSRC)>;
  def : Pat<(v512i1 (int_ppc_mma_xvi8ger4spp v512i1:$ATi, v16i8:$XA, v16i8:$XB)),
            (XVI8GER4SPP $ATi, RCCp.AToVSRC, RCCp.BToVSRC)>;
}

let Predicates = [MMA, IsISAFuture] in {
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 751-758

```tablegen
  def : Pat<(v512i1 (int_ppc_mma_xvf32ger v16i8:$XA, v16i8:$XB)),
            (XVF32GERW RCCp.AToVSRC, RCCp.BToVSRC)>;
  def : Pat<(v512i1 (int_ppc_mma_xvf32gerpp v512i1:$ATi, v16i8:$XA, v16i8:$XB)),
            (XVF32GERWPP $ATi, RCCp.AToVSRC, RCCp.BToVSRC)>;
  def : Pat<(v512i1 (int_ppc_mma_xvf32gerpn v512i1:$ATi, v16i8:$XA, v16i8:$XB)),
            (XVF32GERWPN $ATi, RCCp.AToVSRC, RCCp.BToVSRC)>;
  def : Pat<(v512i1 (int_ppc_mma_xvf32gernp v512i1:$ATi, v16i8:$XA, v16i8:$XB)),
            (XVF32GERWNP $ATi, RCCp.AToVSRC, RCCp.BToVSRC)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 759-766

```tablegen
  def : Pat<(v512i1 (int_ppc_mma_xvf32gernn v512i1:$ATi, v16i8:$XA, v16i8:$XB)),
            (XVF32GERWNN $ATi, RCCp.AToVSRC, RCCp.BToVSRC)>;
  def : Pat<(v512i1 (int_ppc_mma_xvf64ger v256i1:$XA, v16i8:$XB)),
            (XVF64GERW $XA, RCCp.BToVSRC)>;
  def : Pat<(v512i1 (int_ppc_mma_xvf64gerpp v512i1:$ATi, v256i1:$XA, v16i8:$XB)),
            (XVF64GERWPP $ATi, $XA, RCCp.BToVSRC)>;
  def : Pat<(v512i1 (int_ppc_mma_xvf64gerpn v512i1:$ATi, v256i1:$XA, v16i8:$XB)),
            (XVF64GERWPN $ATi, $XA, RCCp.BToVSRC)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 767-773

```tablegen
  def : Pat<(v512i1 (int_ppc_mma_xvf64gernp v512i1:$ATi, v256i1:$XA, v16i8:$XB)),
            (XVF64GERWNP $ATi, $XA, RCCp.BToVSRC)>;
  def : Pat<(v512i1 (int_ppc_mma_xvf64gernn v512i1:$ATi, v256i1:$XA, v16i8:$XB)),
            (XVF64GERWNN $ATi, $XA, RCCp.BToVSRC)>;

  def : Pat<(v512i1 (int_ppc_mma_xvbf16ger2 v16i8:$XA, v16i8:$XB)),
            (XVBF16GER2W RCCp.AToVSRC, RCCp.BToVSRC)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 774-781

```tablegen
  def : Pat<(v512i1 (int_ppc_mma_xvbf16ger2pp v512i1:$ATi, v16i8:$XA, v16i8:$XB)),
            (XVBF16GER2WPP $ATi, RCCp.AToVSRC, RCCp.BToVSRC)>;
  def : Pat<(v512i1 (int_ppc_mma_xvbf16ger2pn v512i1:$ATi, v16i8:$XA, v16i8:$XB)),
            (XVBF16GER2WPN $ATi, RCCp.AToVSRC, RCCp.BToVSRC)>;
  def : Pat<(v512i1 (int_ppc_mma_xvbf16ger2np v512i1:$ATi, v16i8:$XA, v16i8:$XB)),
            (XVBF16GER2WNP $ATi, RCCp.AToVSRC, RCCp.BToVSRC)>;
  def : Pat<(v512i1 (int_ppc_mma_xvbf16ger2nn v512i1:$ATi, v16i8:$XA, v16i8:$XB)),
            (XVBF16GER2WNN $ATi, RCCp.AToVSRC, RCCp.BToVSRC)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 782-790

```tablegen
  def : Pat<(v512i1 (int_ppc_mma_xvi16ger2 v16i8:$XA, v16i8:$XB)),
            (XVI16GER2W RCCp.AToVSRC, RCCp.BToVSRC)>;
  def : Pat<(v512i1 (int_ppc_mma_xvi16ger2pp v512i1:$ATi, v16i8:$XA, v16i8:$XB)),
            (XVI16GER2WPP $ATi, RCCp.AToVSRC, RCCp.BToVSRC)>;
  def : Pat<(v512i1 (int_ppc_mma_xvi8ger4spp v512i1:$ATi, v16i8:$XA, v16i8:$XB)),
            (XVI8GER4WSPP $ATi, RCCp.AToVSRC, RCCp.BToVSRC)>;
}
// MMA Intrinsics
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 791-801

```tablegen
let Predicates = [MMA, PrefixInstrs, IsNotISAFuture] in {
  def : Pat<(v512i1 (int_ppc_mma_pmxvi4ger8 v16i8:$XA, v16i8:$XB, Msk4Imm:$XMSK,
                                            Msk4Imm:$YMSK, Msk8Imm:$PMSK)),
            (PMXVI4GER8 RCCp.AToVSRC, RCCp.BToVSRC, Msk4Imm:$XMSK,
                        Msk4Imm:$YMSK, Msk8Imm:$PMSK)>;
  def : Pat<(v512i1 (int_ppc_mma_pmxvi4ger8pp v512i1:$ATi, v16i8:$XA, v16i8:$XB,
                                              Msk4Imm:$XMSK, Msk4Imm:$YMSK,
                                              Msk8Imm:$PMSK)),
            (PMXVI4GER8PP $ATi, RCCp.AToVSRC, RCCp.BToVSRC, Msk4Imm:$XMSK,
                          Msk4Imm:$YMSK, Msk8Imm:$PMSK)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 802-811

```tablegen
  def : Pat<(v512i1 (int_ppc_mma_pmxvi8ger4 v16i8:$XA, v16i8:$XB, Msk4Imm:$XMSK,
                                            Msk4Imm:$YMSK, Msk4Imm:$PMSK)),
            (PMXVI8GER4 RCCp.AToVSRC, RCCp.BToVSRC, Msk4Imm:$XMSK,
                        Msk4Imm:$YMSK, Msk4Imm:$PMSK)>;
  def : Pat<(v512i1 (int_ppc_mma_pmxvi8ger4pp v512i1:$ATi, v16i8:$XA, v16i8:$XB,
                                              Msk4Imm:$XMSK, Msk4Imm:$YMSK,
                                              Msk4Imm:$PMSK)),
            (PMXVI8GER4PP $ATi, RCCp.AToVSRC, RCCp.BToVSRC, Msk4Imm:$XMSK,
                          Msk4Imm:$YMSK, Msk4Imm:$PMSK)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 812-820

```tablegen
  def : Pat<(v512i1 (int_ppc_mma_pmxvi16ger2s v16i8:$XA, v16i8:$XB, Msk4Imm:$XMSK,
                                              Msk4Imm:$YMSK, Msk2Imm:$PMSK)),
            (PMXVI16GER2S RCCp.AToVSRC, RCCp.BToVSRC, Msk4Imm:$XMSK,
                          Msk4Imm:$YMSK, Msk2Imm:$PMSK)>;
  def : Pat<(v512i1 (int_ppc_mma_pmxvi16ger2spp v512i1:$ATi, v16i8:$XA, v16i8:$XB,
                                                Msk4Imm:$XMSK, Msk4Imm:$YMSK,
                                                Msk2Imm:$PMSK)),
            (PMXVI16GER2SPP $ATi, RCCp.AToVSRC, RCCp.BToVSRC, Msk4Imm:$XMSK,
                            Msk4Imm:$YMSK, Msk2Imm:$PMSK)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 821-829

```tablegen
  def : Pat<(v512i1 (int_ppc_mma_pmxvf16ger2 v16i8:$XA, v16i8:$XB, Msk4Imm:$XMSK,
                                             Msk4Imm:$YMSK, Msk2Imm:$PMSK)),
            (PMXVF16GER2 RCCp.AToVSRC, RCCp.BToVSRC, Msk4Imm:$XMSK,
                         Msk4Imm:$YMSK, Msk2Imm:$PMSK)>;
  def : Pat<(v512i1 (int_ppc_mma_pmxvf16ger2pp v512i1:$ATi, v16i8:$XA, v16i8:$XB,
                                               Msk4Imm:$XMSK, Msk4Imm:$YMSK,
                                               Msk2Imm:$PMSK)),
            (PMXVF16GER2PP $ATi, RCCp.AToVSRC, RCCp.BToVSRC, Msk4Imm:$XMSK,
                           Msk4Imm:$YMSK, Msk2Imm:$PMSK)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 830-839

```tablegen
  def : Pat<(v512i1 (int_ppc_mma_pmxvf16ger2pn v512i1:$ATi, v16i8:$XA, v16i8:$XB,
                                               Msk4Imm:$XMSK, Msk4Imm:$YMSK,
                                               Msk2Imm:$PMSK)),
            (PMXVF16GER2PN $ATi, RCCp.AToVSRC, RCCp.BToVSRC, Msk4Imm:$XMSK,
                           Msk4Imm:$YMSK, Msk2Imm:$PMSK)>;
  def : Pat<(v512i1 (int_ppc_mma_pmxvf16ger2np v512i1:$ATi, v16i8:$XA, v16i8:$XB,
                                               Msk4Imm:$XMSK, Msk4Imm:$YMSK,
                                               Msk2Imm:$PMSK)),
            (PMXVF16GER2NP $ATi, RCCp.AToVSRC, RCCp.BToVSRC, Msk4Imm:$XMSK,
                           Msk4Imm:$YMSK, Msk2Imm:$PMSK)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 840-849

```tablegen
  def : Pat<(v512i1 (int_ppc_mma_pmxvf16ger2nn v512i1:$ATi, v16i8:$XA, v16i8:$XB,
                                               Msk4Imm:$XMSK, Msk4Imm:$YMSK,
                                               Msk2Imm:$PMSK)),
            (PMXVF16GER2NN $ATi, RCCp.AToVSRC, RCCp.BToVSRC, Msk4Imm:$XMSK,
                           Msk4Imm:$YMSK, Msk2Imm:$PMSK)>;

  def : Pat<(v512i1 (int_ppc_mma_pmxvf32ger v16i8:$XA, v16i8:$XB, Msk4Imm:$XMSK,
                                            Msk4Imm:$YMSK)),
            (PMXVF32GER RCCp.AToVSRC, RCCp.BToVSRC, Msk4Imm:$XMSK,
                        Msk4Imm:$YMSK)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 850-857

```tablegen
  def : Pat<(v512i1 (int_ppc_mma_pmxvf32gerpp v512i1:$ATi, v16i8:$XA, v16i8:$XB,
                                              Msk4Imm:$XMSK, Msk4Imm:$YMSK)),
            (PMXVF32GERPP $ATi, RCCp.AToVSRC, RCCp.BToVSRC, Msk4Imm:$XMSK,
                          Msk4Imm:$YMSK)>;
  def : Pat<(v512i1 (int_ppc_mma_pmxvf32gerpn v512i1:$ATi, v16i8:$XA, v16i8:$XB,
                                              Msk4Imm:$XMSK, Msk4Imm:$YMSK)),
            (PMXVF32GERPN $ATi, RCCp.AToVSRC, RCCp.BToVSRC, Msk4Imm:$XMSK,
                          Msk4Imm:$YMSK)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 858-866

```tablegen
  def : Pat<(v512i1 (int_ppc_mma_pmxvf32gernp v512i1:$ATi, v16i8:$XA, v16i8:$XB,
                                              Msk4Imm:$XMSK, Msk4Imm:$YMSK)),
            (PMXVF32GERNP $ATi, RCCp.AToVSRC, RCCp.BToVSRC, Msk4Imm:$XMSK,
                          Msk4Imm:$YMSK)>;
  def : Pat<(v512i1 (int_ppc_mma_pmxvf32gernn v512i1:$ATi, v16i8:$XA, v16i8:$XB,
                                              Msk4Imm:$XMSK, Msk4Imm:$YMSK)),
            (PMXVF32GERNN $ATi, RCCp.AToVSRC, RCCp.BToVSRC, Msk4Imm:$XMSK,
                          Msk4Imm:$YMSK)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 867-873

```tablegen
  def : Pat<(v512i1 (int_ppc_mma_pmxvf64ger v256i1:$XA, v16i8:$XB, Msk4Imm:$XMSK,
                                            Msk2Imm:$YMSK)),
            (PMXVF64GER $XA, RCCp.BToVSRC, Msk4Imm:$XMSK, Msk2Imm:$YMSK)>;
  def : Pat<(v512i1 (int_ppc_mma_pmxvf64gerpp v512i1:$ATi, v256i1:$XA, v16i8:$XB,
                                              Msk4Imm:$XMSK, Msk2Imm:$YMSK)),
            (PMXVF64GERPP $ATi, $XA, RCCp.BToVSRC, Msk4Imm:$XMSK,
                          Msk2Imm:$YMSK)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 874-881

```tablegen
  def : Pat<(v512i1 (int_ppc_mma_pmxvf64gerpn v512i1:$ATi, v256i1:$XA, v16i8:$XB,
                                              Msk4Imm:$XMSK, Msk2Imm:$YMSK)),
            (PMXVF64GERPN $ATi, $XA, RCCp.BToVSRC, Msk4Imm:$XMSK,
                          Msk2Imm:$YMSK)>;
  def : Pat<(v512i1 (int_ppc_mma_pmxvf64gernp v512i1:$ATi, v256i1:$XA, v16i8:$XB,
                                              Msk4Imm:$XMSK, Msk2Imm:$YMSK)),
            (PMXVF64GERNP $ATi, $XA, RCCp.BToVSRC, Msk4Imm:$XMSK,
                          Msk2Imm:$YMSK)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 882-890

```tablegen
  def : Pat<(v512i1 (int_ppc_mma_pmxvf64gernn v512i1:$ATi, v256i1:$XA, v16i8:$XB,
                                              Msk4Imm:$XMSK, Msk2Imm:$YMSK)),
            (PMXVF64GERNN $ATi, $XA, RCCp.BToVSRC, Msk4Imm:$XMSK,
                          Msk2Imm:$YMSK)>;

  def : Pat<(v512i1 (int_ppc_mma_pmxvbf16ger2 v16i8:$XA, v16i8:$XB, Msk4Imm:$XMSK,
                                              Msk4Imm:$YMSK, Msk2Imm:$PMSK)),
            (PMXVBF16GER2 RCCp.AToVSRC, RCCp.BToVSRC, Msk4Imm:$XMSK,
                          Msk4Imm:$YMSK, Msk2Imm:$PMSK)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 891-900

```tablegen
  def : Pat<(v512i1 (int_ppc_mma_pmxvbf16ger2pp v512i1:$ATi, v16i8:$XA, v16i8:$XB,
                                                Msk4Imm:$XMSK, Msk4Imm:$YMSK,
                                                Msk2Imm:$PMSK)),
            (PMXVBF16GER2PP $ATi, RCCp.AToVSRC, RCCp.BToVSRC, Msk4Imm:$XMSK,
                            Msk4Imm:$YMSK, Msk2Imm:$PMSK)>;
  def : Pat<(v512i1 (int_ppc_mma_pmxvbf16ger2pn v512i1:$ATi, v16i8:$XA, v16i8:$XB,
                                                Msk4Imm:$XMSK, Msk4Imm:$YMSK,
                                                Msk2Imm:$PMSK)),
            (PMXVBF16GER2PN $ATi, RCCp.AToVSRC, RCCp.BToVSRC, Msk4Imm:$XMSK,
                            Msk4Imm:$YMSK, Msk2Imm:$PMSK)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 901-910

```tablegen
  def : Pat<(v512i1 (int_ppc_mma_pmxvbf16ger2np v512i1:$ATi, v16i8:$XA, v16i8:$XB,
                                                Msk4Imm:$XMSK, Msk4Imm:$YMSK,
                                                Msk2Imm:$PMSK)),
            (PMXVBF16GER2NP $ATi, RCCp.AToVSRC, RCCp.BToVSRC, Msk4Imm:$XMSK,
                            Msk4Imm:$YMSK, Msk2Imm:$PMSK)>;
  def : Pat<(v512i1 (int_ppc_mma_pmxvbf16ger2nn v512i1:$ATi, v16i8:$XA, v16i8:$XB,
                                                Msk4Imm:$XMSK, Msk4Imm:$YMSK,
                                                Msk2Imm:$PMSK)),
            (PMXVBF16GER2NN $ATi, RCCp.AToVSRC, RCCp.BToVSRC, Msk4Imm:$XMSK,
                            Msk4Imm:$YMSK, Msk2Imm:$PMSK)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 911-919

```tablegen
  def : Pat<(v512i1 (int_ppc_mma_pmxvi16ger2 v16i8:$XA, v16i8:$XB, Msk4Imm:$XMSK,
                                             Msk4Imm:$YMSK, Msk2Imm:$PMSK)),
            (PMXVI16GER2 RCCp.AToVSRC, RCCp.BToVSRC, Msk4Imm:$XMSK,
                         Msk4Imm:$YMSK, Msk2Imm:$PMSK)>;
  def : Pat<(v512i1 (int_ppc_mma_pmxvi8ger4spp v512i1:$ATi, v16i8:$XA, v16i8:$XB,
                                               Msk4Imm:$XMSK, Msk4Imm:$YMSK,
                                               Msk2Imm:$PMSK)),
            (PMXVI8GER4SPP $ATi, RCCp.AToVSRC, RCCp.BToVSRC, Msk4Imm:$XMSK,
                           Msk4Imm:$YMSK, Msk2Imm:$PMSK)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 920-926

```tablegen
  def : Pat<(v512i1 (int_ppc_mma_pmxvi16ger2pp v512i1:$ATi, v16i8:$XA, v16i8:$XB,
                                               Msk4Imm:$XMSK, Msk4Imm:$YMSK,
                                               Msk2Imm:$PMSK)),
            (PMXVI16GER2PP $ATi, RCCp.AToVSRC, RCCp.BToVSRC, Msk4Imm:$XMSK,
                           Msk4Imm:$YMSK, Msk2Imm:$PMSK)>;
}
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 927-937

```tablegen
let Predicates = [MMA, PrefixInstrs, IsISAFuture] in {
  def : Pat<(v512i1 (int_ppc_mma_pmxvi4ger8 v16i8:$XA, v16i8:$XB, Msk4Imm:$XMSK,
                                            Msk4Imm:$YMSK, Msk8Imm:$PMSK)),
            (PMXVI4GER8W RCCp.AToVSRC, RCCp.BToVSRC, Msk4Imm:$XMSK,
                        Msk4Imm:$YMSK, Msk8Imm:$PMSK)>;
  def : Pat<(v512i1 (int_ppc_mma_pmxvi4ger8pp v512i1:$ATi, v16i8:$XA, v16i8:$XB,
                                              Msk4Imm:$XMSK, Msk4Imm:$YMSK,
                                              Msk8Imm:$PMSK)),
            (PMXVI4GER8WPP $ATi, RCCp.AToVSRC, RCCp.BToVSRC, Msk4Imm:$XMSK,
                          Msk4Imm:$YMSK, Msk8Imm:$PMSK)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 938-947

```tablegen
  def : Pat<(v512i1 (int_ppc_mma_pmxvi8ger4 v16i8:$XA, v16i8:$XB, Msk4Imm:$XMSK,
                                            Msk4Imm:$YMSK, Msk4Imm:$PMSK)),
            (PMXVI8GER4W RCCp.AToVSRC, RCCp.BToVSRC, Msk4Imm:$XMSK,
                        Msk4Imm:$YMSK, Msk4Imm:$PMSK)>;
  def : Pat<(v512i1 (int_ppc_mma_pmxvi8ger4pp v512i1:$ATi, v16i8:$XA, v16i8:$XB,
                                              Msk4Imm:$XMSK, Msk4Imm:$YMSK,
                                              Msk4Imm:$PMSK)),
            (PMXVI8GER4WPP $ATi, RCCp.AToVSRC, RCCp.BToVSRC, Msk4Imm:$XMSK,
                          Msk4Imm:$YMSK, Msk4Imm:$PMSK)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 948-956

```tablegen
  def : Pat<(v512i1 (int_ppc_mma_pmxvi16ger2s v16i8:$XA, v16i8:$XB, Msk4Imm:$XMSK,
                                              Msk4Imm:$YMSK, Msk2Imm:$PMSK)),
            (PMXVI16GER2SW RCCp.AToVSRC, RCCp.BToVSRC, Msk4Imm:$XMSK,
                          Msk4Imm:$YMSK, Msk2Imm:$PMSK)>;
  def : Pat<(v512i1 (int_ppc_mma_pmxvi16ger2spp v512i1:$ATi, v16i8:$XA, v16i8:$XB,
                                                Msk4Imm:$XMSK, Msk4Imm:$YMSK,
                                                Msk2Imm:$PMSK)),
            (PMXVI16GER2SWPP $ATi, RCCp.AToVSRC, RCCp.BToVSRC, Msk4Imm:$XMSK,
                            Msk4Imm:$YMSK, Msk2Imm:$PMSK)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 957-965

```tablegen
  def : Pat<(v512i1 (int_ppc_mma_pmxvf16ger2 v16i8:$XA, v16i8:$XB, Msk4Imm:$XMSK,
                                             Msk4Imm:$YMSK, Msk2Imm:$PMSK)),
            (PMXVF16GER2W RCCp.AToVSRC, RCCp.BToVSRC, Msk4Imm:$XMSK,
                         Msk4Imm:$YMSK, Msk2Imm:$PMSK)>;
  def : Pat<(v512i1 (int_ppc_mma_pmxvf16ger2pp v512i1:$ATi, v16i8:$XA, v16i8:$XB,
                                               Msk4Imm:$XMSK, Msk4Imm:$YMSK,
                                               Msk2Imm:$PMSK)),
            (PMXVF16GER2WPP $ATi, RCCp.AToVSRC, RCCp.BToVSRC, Msk4Imm:$XMSK,
                           Msk4Imm:$YMSK, Msk2Imm:$PMSK)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 966-975

```tablegen
  def : Pat<(v512i1 (int_ppc_mma_pmxvf16ger2pn v512i1:$ATi, v16i8:$XA, v16i8:$XB,
                                               Msk4Imm:$XMSK, Msk4Imm:$YMSK,
                                               Msk2Imm:$PMSK)),
            (PMXVF16GER2WPN $ATi, RCCp.AToVSRC, RCCp.BToVSRC, Msk4Imm:$XMSK,
                           Msk4Imm:$YMSK, Msk2Imm:$PMSK)>;
  def : Pat<(v512i1 (int_ppc_mma_pmxvf16ger2np v512i1:$ATi, v16i8:$XA, v16i8:$XB,
                                               Msk4Imm:$XMSK, Msk4Imm:$YMSK,
                                               Msk2Imm:$PMSK)),
            (PMXVF16GER2WNP $ATi, RCCp.AToVSRC, RCCp.BToVSRC, Msk4Imm:$XMSK,
                           Msk4Imm:$YMSK, Msk2Imm:$PMSK)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 976-985

```tablegen
  def : Pat<(v512i1 (int_ppc_mma_pmxvf16ger2nn v512i1:$ATi, v16i8:$XA, v16i8:$XB,
                                               Msk4Imm:$XMSK, Msk4Imm:$YMSK,
                                               Msk2Imm:$PMSK)),
            (PMXVF16GER2WNN $ATi, RCCp.AToVSRC, RCCp.BToVSRC, Msk4Imm:$XMSK,
                           Msk4Imm:$YMSK, Msk2Imm:$PMSK)>;

  def : Pat<(v512i1 (int_ppc_mma_pmxvf32ger v16i8:$XA, v16i8:$XB, Msk4Imm:$XMSK,
                                            Msk4Imm:$YMSK)),
            (PMXVF32GERW RCCp.AToVSRC, RCCp.BToVSRC, Msk4Imm:$XMSK,
                        Msk4Imm:$YMSK)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 986-993

```tablegen
  def : Pat<(v512i1 (int_ppc_mma_pmxvf32gerpp v512i1:$ATi, v16i8:$XA, v16i8:$XB,
                                              Msk4Imm:$XMSK, Msk4Imm:$YMSK)),
            (PMXVF32GERWPP $ATi, RCCp.AToVSRC, RCCp.BToVSRC, Msk4Imm:$XMSK,
                          Msk4Imm:$YMSK)>;
  def : Pat<(v512i1 (int_ppc_mma_pmxvf32gerpn v512i1:$ATi, v16i8:$XA, v16i8:$XB,
                                              Msk4Imm:$XMSK, Msk4Imm:$YMSK)),
            (PMXVF32GERWPN $ATi, RCCp.AToVSRC, RCCp.BToVSRC, Msk4Imm:$XMSK,
                          Msk4Imm:$YMSK)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 994-1002

```tablegen
  def : Pat<(v512i1 (int_ppc_mma_pmxvf32gernp v512i1:$ATi, v16i8:$XA, v16i8:$XB,
                                              Msk4Imm:$XMSK, Msk4Imm:$YMSK)),
            (PMXVF32GERWNP $ATi, RCCp.AToVSRC, RCCp.BToVSRC, Msk4Imm:$XMSK,
                          Msk4Imm:$YMSK)>;
  def : Pat<(v512i1 (int_ppc_mma_pmxvf32gernn v512i1:$ATi, v16i8:$XA, v16i8:$XB,
                                              Msk4Imm:$XMSK, Msk4Imm:$YMSK)),
            (PMXVF32GERWNN $ATi, RCCp.AToVSRC, RCCp.BToVSRC, Msk4Imm:$XMSK,
                          Msk4Imm:$YMSK)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 1003-1009

```tablegen
  def : Pat<(v512i1 (int_ppc_mma_pmxvf64ger v256i1:$XA, v16i8:$XB, Msk4Imm:$XMSK,
                                            Msk2Imm:$YMSK)),
            (PMXVF64GERW $XA, RCCp.BToVSRC, Msk4Imm:$XMSK, Msk2Imm:$YMSK)>;
  def : Pat<(v512i1 (int_ppc_mma_pmxvf64gerpp v512i1:$ATi, v256i1:$XA, v16i8:$XB,
                                              Msk4Imm:$XMSK, Msk2Imm:$YMSK)),
            (PMXVF64GERWPP $ATi, $XA, RCCp.BToVSRC, Msk4Imm:$XMSK,
                          Msk2Imm:$YMSK)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 1010-1017

```tablegen
  def : Pat<(v512i1 (int_ppc_mma_pmxvf64gerpn v512i1:$ATi, v256i1:$XA, v16i8:$XB,
                                              Msk4Imm:$XMSK, Msk2Imm:$YMSK)),
            (PMXVF64GERWPN $ATi, $XA, RCCp.BToVSRC, Msk4Imm:$XMSK,
                          Msk2Imm:$YMSK)>;
  def : Pat<(v512i1 (int_ppc_mma_pmxvf64gernp v512i1:$ATi, v256i1:$XA, v16i8:$XB,
                                              Msk4Imm:$XMSK, Msk2Imm:$YMSK)),
            (PMXVF64GERWNP $ATi, $XA, RCCp.BToVSRC, Msk4Imm:$XMSK,
                          Msk2Imm:$YMSK)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 1018-1026

```tablegen
  def : Pat<(v512i1 (int_ppc_mma_pmxvf64gernn v512i1:$ATi, v256i1:$XA, v16i8:$XB,
                                              Msk4Imm:$XMSK, Msk2Imm:$YMSK)),
            (PMXVF64GERWNN $ATi, $XA, RCCp.BToVSRC, Msk4Imm:$XMSK,
                          Msk2Imm:$YMSK)>;

  def : Pat<(v512i1 (int_ppc_mma_pmxvbf16ger2 v16i8:$XA, v16i8:$XB, Msk4Imm:$XMSK,
                                              Msk4Imm:$YMSK, Msk2Imm:$PMSK)),
            (PMXVBF16GER2W RCCp.AToVSRC, RCCp.BToVSRC, Msk4Imm:$XMSK,
                          Msk4Imm:$YMSK, Msk2Imm:$PMSK)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 1027-1036

```tablegen
  def : Pat<(v512i1 (int_ppc_mma_pmxvbf16ger2pp v512i1:$ATi, v16i8:$XA, v16i8:$XB,
                                                Msk4Imm:$XMSK, Msk4Imm:$YMSK,
                                                Msk2Imm:$PMSK)),
            (PMXVBF16GER2WPP $ATi, RCCp.AToVSRC, RCCp.BToVSRC, Msk4Imm:$XMSK,
                            Msk4Imm:$YMSK, Msk2Imm:$PMSK)>;
  def : Pat<(v512i1 (int_ppc_mma_pmxvbf16ger2pn v512i1:$ATi, v16i8:$XA, v16i8:$XB,
                                                Msk4Imm:$XMSK, Msk4Imm:$YMSK,
                                                Msk2Imm:$PMSK)),
            (PMXVBF16GER2WPN $ATi, RCCp.AToVSRC, RCCp.BToVSRC, Msk4Imm:$XMSK,
                            Msk4Imm:$YMSK, Msk2Imm:$PMSK)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 1037-1046

```tablegen
  def : Pat<(v512i1 (int_ppc_mma_pmxvbf16ger2np v512i1:$ATi, v16i8:$XA, v16i8:$XB,
                                                Msk4Imm:$XMSK, Msk4Imm:$YMSK,
                                                Msk2Imm:$PMSK)),
            (PMXVBF16GER2WNP $ATi, RCCp.AToVSRC, RCCp.BToVSRC, Msk4Imm:$XMSK,
                            Msk4Imm:$YMSK, Msk2Imm:$PMSK)>;
  def : Pat<(v512i1 (int_ppc_mma_pmxvbf16ger2nn v512i1:$ATi, v16i8:$XA, v16i8:$XB,
                                                Msk4Imm:$XMSK, Msk4Imm:$YMSK,
                                                Msk2Imm:$PMSK)),
            (PMXVBF16GER2WNN $ATi, RCCp.AToVSRC, RCCp.BToVSRC, Msk4Imm:$XMSK,
                            Msk4Imm:$YMSK, Msk2Imm:$PMSK)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 1047-1055

```tablegen
  def : Pat<(v512i1 (int_ppc_mma_pmxvi16ger2 v16i8:$XA, v16i8:$XB, Msk4Imm:$XMSK,
                                             Msk4Imm:$YMSK, Msk2Imm:$PMSK)),
            (PMXVI16GER2W RCCp.AToVSRC, RCCp.BToVSRC, Msk4Imm:$XMSK,
                         Msk4Imm:$YMSK, Msk2Imm:$PMSK)>;
  def : Pat<(v512i1 (int_ppc_mma_pmxvi8ger4spp v512i1:$ATi, v16i8:$XA, v16i8:$XB,
                                               Msk4Imm:$XMSK, Msk4Imm:$YMSK,
                                               Msk2Imm:$PMSK)),
            (PMXVI8GER4WSPP $ATi, RCCp.AToVSRC, RCCp.BToVSRC, Msk4Imm:$XMSK,
                           Msk4Imm:$YMSK, Msk2Imm:$PMSK)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 1056-1062

```tablegen
  def : Pat<(v512i1 (int_ppc_mma_pmxvi16ger2pp v512i1:$ATi, v16i8:$XA, v16i8:$XB,
                                               Msk4Imm:$XMSK, Msk4Imm:$YMSK,
                                               Msk2Imm:$PMSK)),
            (PMXVI16GER2WPP $ATi, RCCp.AToVSRC, RCCp.BToVSRC, Msk4Imm:$XMSK,
                           Msk4Imm:$YMSK, Msk2Imm:$PMSK)>;
}
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 1063-1077

```tablegen
def ConcatsMMA {
   dag VecsToVecPair0 =
          (v256i1 (INSERT_SUBREG
                    (INSERT_SUBREG (IMPLICIT_DEF), $vs0, sub_vsx1),
                    $vs1, sub_vsx0));
   dag VecsToVecPair1 =
          (v256i1 (INSERT_SUBREG
                    (INSERT_SUBREG (IMPLICIT_DEF), $vs2, sub_vsx1),
                    $vs3, sub_vsx0));
  dag VecsToVecQuad = (BUILD_UACC
          (v512i1 (REG_SEQUENCE UACCRC,
                                (KILL_PAIR VecsToVecPair0), sub_pair0,
                                (KILL_PAIR VecsToVecPair1), sub_pair1)));
}
```
- **EN**: Adds declarative TableGen records such as `ConcatsMMA` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ConcatsMMA`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1078-1086

```tablegen
def Extracts {
  dag Pair0 = (v256i1 (EXTRACT_SUBREG $v, sub_pair0));
  dag Pair1 = (v256i1 (EXTRACT_SUBREG $v, sub_pair1));
  dag Vec0 = (v4i32 (EXTRACT_SUBREG Pair0, sub_vsx0));
  dag Vec1 = (v4i32 (EXTRACT_SUBREG Pair0, sub_vsx1));
  dag Vec2 = (v4i32 (EXTRACT_SUBREG Pair1, sub_vsx0));
  dag Vec3 = (v4i32 (EXTRACT_SUBREG Pair1, sub_vsx1));
}
```
- **EN**: Adds declarative TableGen records such as `Extracts` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `Extracts`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1087-1093

```tablegen
let Predicates = [MMA, IsNotISAFuture] in {
  def : Pat<(v512i1 (PPCAccBuild v4i32:$vs1, v4i32:$vs0, v4i32:$vs3, v4i32:$vs2)),
            (XXMTACC ConcatsMMA.VecsToVecQuad)>;
  def : Pat<(v512i1 (int_ppc_mma_assemble_acc v16i8:$vs1, v16i8:$vs0,
                                              v16i8:$vs3, v16i8:$vs2)),
            (XXMTACC ConcatsMMA.VecsToVecQuad)>;
  def : Pat<(v512i1 (PPCxxmfacc v512i1:$AS)), (XXMFACC acc:$AS)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 1094-1103

```tablegen
  def : Pat<(v4i32 (PPCAccExtractVsx acc:$v, 0)),
            Extracts.Vec0>;
  def : Pat<(v4i32 (PPCAccExtractVsx acc:$v, 1)),
            Extracts.Vec1>;
  def : Pat<(v4i32 (PPCAccExtractVsx acc:$v, 2)),
            Extracts.Vec2>;
  def : Pat<(v4i32 (PPCAccExtractVsx acc:$v, 3)),
            Extracts.Vec3>;
}
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 1104-1111

```tablegen
let Predicates = [MMA, IsISAFuture] in {
  def : Pat<(v512i1 (PPCAccBuild v4i32:$vs1, v4i32:$vs0, v4i32:$vs3, v4i32:$vs2)),
            (DMXXINSTDMR512 ConcatsMMA.VecsToVecPair0, ConcatsMMA.VecsToVecPair1)>;
  def : Pat<(v512i1 (int_ppc_mma_assemble_acc v16i8:$vs1, v16i8:$vs0,
                                              v16i8:$vs3, v16i8:$vs2)),
            (DMXXINSTDMR512 ConcatsMMA.VecsToVecPair0, ConcatsMMA.VecsToVecPair1)>;
  def : Pat<(v512i1 immAllZerosV), (DMXXSETACCZ)>;
}
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

## Key Concepts / 关键概念

- PowerPC backend integration / PowerPC 后端集成
- TableGen declarations for the backend / 该后端的 TableGen 声明
- LLVM TableGen DSL / LLVM TableGen 领域特定语言
- Declarative TableGen records / 声明式 TableGen 记录

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- None explicitly included in the source text / 源码文本中没有显式包含项

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- Generated backend tables / 生成式后端表
- PowerPC target-specific helpers / PowerPC 目标专用辅助组件
