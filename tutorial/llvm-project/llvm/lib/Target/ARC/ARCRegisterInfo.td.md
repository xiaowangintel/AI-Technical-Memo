# ARCRegisterInfo.td — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/ARC/ARCRegisterInfo.td`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): Defines target registers, register classes, and register relations in TableGen.
- 目的（中文）: 使用 TableGen 定义目标寄存器、寄存器类以及寄存器之间的关系。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18

```tablegen
   1: //===- ARCRegisterInfo.td - ARC Register defs --------------*- tablegen -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
   9: //===----------------------------------------------------------------------===//
  10: //  Declarations that describe the ARC register file
  11: //===----------------------------------------------------------------------===//
  12: 
  13: class ARCReg<string n, list<string> altNames> : Register<n, altNames> {
  14:   field bits<6> HwEncoding;
  15:   let Namespace = "ARC";
  16: }
  17: 
  18: // Registers are identified with 6-bit ID numbers.
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. This range defines declarative TableGen records such as ARCReg, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这一段定义了 ARCReg 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 19-36

```tablegen
  19: // Core - 32-bit core registers
  20: class Core<int num, string n, list<string>altNames=[]> : ARCReg<n, altNames> {
  21:   let HWEncoding = num;
  22: }
  23: 
  24: // Auxilary register
  25: class Aux<int num, string n, list<string> altNames=[]> : ARCReg<n, altNames> {
  26:   let HWEncoding = num;
  27: }
  28: 
  29: // Integer registers
  30: foreach i = 0 - 3 in
  31:   def R#i : Core<i, "%r"#i>, DwarfRegNum<[i]>;
  32: 
  33: let CostPerUse=[1] in {
  34:   foreach i = 4 - 11 in
  35:     def R#i : Core<i, "%r"#i>, DwarfRegNum<[i]>;
  36: }
```

- EN: This range defines declarative TableGen records such as Core, Aux, R, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 Core, Aux, R 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 37-54

```tablegen
  37: 
  38: foreach i = 12 - 15 in
  39:   def R#i : Core<i, "%r"#i>, DwarfRegNum<[i]>;
  40: 
  41: let CostPerUse=[1] in {
  42: 
  43:   foreach i = 16 - 25 in
  44:     def R#i : Core<i, "%r"#i>, DwarfRegNum<[i]>;
  45: 
  46:   def GP : Core<26, "%gp",["%r26"]>, DwarfRegNum<[26]>;
  47:   def FP : Core<27, "%fp", ["%r27"]>, DwarfRegNum<[27]>;
  48:   def SP : Core<28, "%sp", ["%r28"]>, DwarfRegNum<[28]>;
  49:   def ILINK : Core<29, "%ilink">, DwarfRegNum<[29]>;
  50:   def R30 : Core<30, "%r30">, DwarfRegNum<[30]>;
  51:   def BLINK : Core<31, "%blink">, DwarfRegNum<[31]>;
  52: 
  53:   // Define extended core registers R32..R63
  54:   foreach i = 32 - 63 in
```

- EN: This range defines declarative TableGen records such as R, R, GP, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 R, R, GP 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 55-72

```tablegen
  55:     def R#i : Core<i, "%r"#i>, DwarfRegNum<[i]>;
  56: }
  57: 
  58: // Auxilary registers
  59: let CostPerUse=[1] in {
  60:   def STATUS32 : Aux<10, "status32">; // No DwarfRegNum defined in the ARC ABI
  61: }
  62: 
  63: def GPR32: RegisterClass<"ARC", [i32], 32,
  64:   (add (sequence "R%u", 0, 25), GP, FP, SP, ILINK, R30, BLINK, (sequence "R%u", 32, 63))> {
  65:   let AltOrders=[(add (sequence "R%u", 0, 25), GP, FP, SP, ILINK, R30, BLINK)];
  66:   let AltOrderSelect = [{
  67:       // When referenced in a C++ code block like this
  68:       // 0 is all Core32 regs
  69:       // 1 is AltOrders[0]
  70:       // 2 is AltOrders[1] and so on
  71:       return 1;
  72:     }];
```

- EN: This range defines declarative TableGen records such as R, STATUS32, GPR32, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 R, STATUS32, GPR32 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 73-79

```tablegen
  73: }
  74: 
  75: def SREG : RegisterClass<"ARC", [i32], 1, (add STATUS32)>;
  76: 
  77: def GPR_S : RegisterClass<"ARC", [i32], 8,
  78:   (add R0, R1, R2, R3, R12, R13, R14, R15)>;
  79: 
```

- EN: This range defines declarative TableGen records such as SREG, GPR_S, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 SREG, GPR_S 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

## Key Concepts / 关键概念

- TableGen DSL / TableGen DSL
- Register definitions / 寄存器定义
- Register classes and aliases / 寄存器类与别名
- Register classes / 寄存器类

## Dependencies / 依赖关系

- LLVM subsystems / LLVM 子系统: TableGen
- Local companions / 本地配套文件: `ARCRegisterInfo.h`, `ARCRegisterInfo.cpp`
