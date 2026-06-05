# ARMSystemRegister.td — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/ARMSystemRegister.td`
- Repository: `llvm-project`
- Purpose (EN): Defines `ARMSystemRegister` in LLVM TableGen DSL for the ARM backend, covering target descriptions, instruction records, and code-generation metadata.
- 用途 (CN): 使用 LLVM TableGen DSL 定义 ARM 后端中的 `ARMSystemRegister`，涵盖目标描述、指令记录以及代码生成元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```tablegen
//===-- ARMSystemRegister.td - ARM Register defs -------------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 9-9
```tablegen
include "llvm/TableGen/SearchableTable.td"
```
- EN: Pulls in other TableGen fragments so this file can reuse common records, predicates, and target-wide definitions.
- CN: 这里引入其他 TableGen 片段，以便复用公共记录、谓词以及整个目标范围内的定义。

### Lines 11-13
```tablegen
//===----------------------------------------------------------------------===//
//  Declarations that describe the ARM system-registers
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 15-26
```tablegen
// M-Class System Registers.
// 'Mask' bits create unique keys for searches.
//
class MClassSysReg<bits<1> UniqMask1,
                   bits<1> UniqMask2,
                   bits<1> UniqMask3,
                   bits<12> Enc12,
                   string name> {
  string Name;
  bits<13> M1Encoding12;
  bits<10> M2M3Encoding8;
  bits<12> Encoding;
```
- EN: Declares reusable TableGen class `MClassSysReg` for `ARMSystemRegister`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMSystemRegister` 声明可复用的 TableGen 类 `MClassSysReg`，通常用于抽象共享字段、谓词或编码结构。

### Lines 28-28
```tablegen
  let Name = name;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 30-32
```tablegen
  let M1Encoding12{12}    = UniqMask1;
  let M1Encoding12{11-00} = Enc12;
  let Encoding            = Enc12;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 34-38
```tablegen
  let M2M3Encoding8{9}    = UniqMask2;
  let M2M3Encoding8{8}    = UniqMask3;
  let M2M3Encoding8{7-0}  = Enc12{7-0};
  code Requires           = [{ {} }];
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 40-44
```tablegen
def MClassSysRegsList : GenericTable {
  let FilterClass = "MClassSysReg";
  let Fields = ["Name", "M1Encoding12", "M2M3Encoding8", "Encoding",
                "Requires"];
}
```
- EN: Defines TableGen record `MClassSysRegsList` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MClassSysRegsList`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 46-49
```tablegen
def lookupMClassSysRegByName : SearchIndex {
  let Table = MClassSysRegsList;
  let Key = ["Name"];
}
```
- EN: Defines TableGen record `lookupMClassSysRegByName` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `lookupMClassSysRegByName`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 51-54
```tablegen
def lookupMClassSysRegByM1Encoding12 : SearchIndex {
  let Table = MClassSysRegsList;
  let Key = ["M1Encoding12"];
}
```
- EN: Defines TableGen record `lookupMClassSysRegByM1Encoding12` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `lookupMClassSysRegByM1Encoding12`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 56-59
```tablegen
def lookupMClassSysRegByM2M3Encoding8 : SearchIndex {
  let Table = MClassSysRegsList;
  let Key = ["M2M3Encoding8"];
}
```
- EN: Defines TableGen record `lookupMClassSysRegByM2M3Encoding8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `lookupMClassSysRegByM2M3Encoding8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 61-72
```tablegen
// [|i|e|x]apsr_nzcvq has alias [|i|e|x]apsr.
//                 Mask1 Mask2 Mask3 Enc12, Name
let Requires = [{ {ARM::FeatureDSP} }] in {
def : MClassSysReg<0,    0,    0,    0x400, "apsr_g">;
def : MClassSysReg<0,    1,    1,    0xc00, "apsr_nzcvqg">;
def : MClassSysReg<0,    0,    0,    0x401, "iapsr_g">;
def : MClassSysReg<0,    1,    1,    0xc01, "iapsr_nzcvqg">;
def : MClassSysReg<0,    0,    0,    0x402, "eapsr_g">;
def : MClassSysReg<0,    1,    1,    0xc02, "eapsr_nzcvqg">;
def : MClassSysReg<0,    0,    0,    0x403, "xpsr_g">;
def : MClassSysReg<0,    1,    1,    0xc03, "xpsr_nzcvqg">;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 74-81
```tablegen
def : MClassSysReg<0,    0,    1,    0x800, "apsr">;
def : MClassSysReg<1,    1,    0,    0x800, "apsr_nzcvq">;
def : MClassSysReg<0,    0,    1,    0x801, "iapsr">;
def : MClassSysReg<1,    1,    0,    0x801, "iapsr_nzcvq">;
def : MClassSysReg<0,    0,    1,    0x802, "eapsr">;
def : MClassSysReg<1,    1,    0,    0x802, "eapsr_nzcvq">;
def : MClassSysReg<0,    0,    1,    0x803, "xpsr">;
def : MClassSysReg<1,    1,    0,    0x803, "xpsr_nzcvq">;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 83-87
```tablegen
def : MClassSysReg<0,    0,    1,    0x805, "ipsr">;
def : MClassSysReg<0,    0,    1,    0x806, "epsr">;
def : MClassSysReg<0,    0,    1,    0x807, "iepsr">;
def : MClassSysReg<0,    0,    1,    0x808, "msp">;
def : MClassSysReg<0,    0,    1,    0x809, "psp">;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 89-92
```tablegen
let Requires = [{ {ARM::HasV8MBaselineOps} }] in {
def : MClassSysReg<0,    0,    1,    0x80a, "msplim">;
def : MClassSysReg<0,    0,    1,    0x80b, "psplim">;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 94-94
```tablegen
def : MClassSysReg<0,    0,    1,    0x810, "primask">;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 96-100
```tablegen
let Requires = [{ {ARM::HasV7Ops} }] in {
def : MClassSysReg<0,    0,    1,    0x811, "basepri">;
def : MClassSysReg<0,    0,    1,    0x812, "basepri_max">;
def : MClassSysReg<0,    0,    1,    0x813, "faultmask">;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 102-102
```tablegen
def : MClassSysReg<0,    0,    1,    0x814, "control">;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 104-107
```tablegen
let Requires = [{ {ARM::Feature8MSecExt} }] in {
def : MClassSysReg<0,    0,    1,    0x888, "msp_ns">;
def : MClassSysReg<0,    0,    1,    0x889, "psp_ns">;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 109-112
```tablegen
let Requires = [{ {ARM::Feature8MSecExt, ARM::HasV8MBaselineOps} }] in {
def : MClassSysReg<0,    0,    1,    0x88a, "msplim_ns">;
def : MClassSysReg<0,    0,    1,    0x88b, "psplim_ns">;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 114-114
```tablegen
def : MClassSysReg<0,    0,    1,    0x890, "primask_ns">;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 116-119
```tablegen
let Requires = [{ {ARM::Feature8MSecExt, ARM::HasV7Ops} }] in {
def : MClassSysReg<0,    0,    1,    0x891, "basepri_ns">;
def : MClassSysReg<0,    0,    1,    0x893, "faultmask_ns">;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 121-124
```tablegen
let Requires = [{ {ARM::Feature8MSecExt} }] in {
def : MClassSysReg<0,    0,    1,    0x894, "control_ns">;
def : MClassSysReg<0,    0,    1,    0x898, "sp_ns">;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 126-143
```tablegen
let Requires = [{ {ARM::FeaturePACBTI} }] in {
def : MClassSysReg<0,    0,    1,    0x820, "pac_key_p_0">;
def : MClassSysReg<0,    0,    1,    0x821, "pac_key_p_1">;
def : MClassSysReg<0,    0,    1,    0x822, "pac_key_p_2">;
def : MClassSysReg<0,    0,    1,    0x823, "pac_key_p_3">;
def : MClassSysReg<0,    0,    1,    0x824, "pac_key_u_0">;
def : MClassSysReg<0,    0,    1,    0x825, "pac_key_u_1">;
def : MClassSysReg<0,    0,    1,    0x826, "pac_key_u_2">;
def : MClassSysReg<0,    0,    1,    0x827, "pac_key_u_3">;
def : MClassSysReg<0,    0,    1,    0x8a0, "pac_key_p_0_ns">;
def : MClassSysReg<0,    0,    1,    0x8a1, "pac_key_p_1_ns">;
def : MClassSysReg<0,    0,    1,    0x8a2, "pac_key_p_2_ns">;
def : MClassSysReg<0,    0,    1,    0x8a3, "pac_key_p_3_ns">;
def : MClassSysReg<0,    0,    1,    0x8a4, "pac_key_u_0_ns">;
def : MClassSysReg<0,    0,    1,    0x8a5, "pac_key_u_1_ns">;
def : MClassSysReg<0,    0,    1,    0x8a6, "pac_key_u_2_ns">;
def : MClassSysReg<0,    0,    1,    0x8a7, "pac_key_u_3_ns">;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 145-152
```tablegen
// Banked Registers
//
class BankedReg<string name,  bits<8> enc> {
  string Name;
  bits<8> Encoding;
  let Name = name;
  let Encoding = enc;
}
```
- EN: Declares reusable TableGen class `BankedReg` for `ARMSystemRegister`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMSystemRegister` 声明可复用的 TableGen 类 `BankedReg`，通常用于抽象共享字段、谓词或编码结构。

### Lines 154-156
```tablegen
def BankedRegsList : GenericTable {
  let FilterClass = "BankedReg";
  let Fields = ["Name", "Encoding"];
```
- EN: Defines TableGen record `BankedRegsList` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BankedRegsList`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 158-160
```tablegen
  let PrimaryKey = ["Encoding"];
  let PrimaryKeyName = "lookupBankedRegByEncoding";
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 162-165
```tablegen
def lookupBankedRegByName : SearchIndex {
  let Table = BankedRegsList;
  let Key = ["Name"];
}
```
- EN: Defines TableGen record `lookupBankedRegByName` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `lookupBankedRegByName`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 168-185
```tablegen
// The values here come from B9.2.3 of the ARM ARM, where bits 4-0 are SysM
// and bit 5 is R.
def : BankedReg<"r8_usr",   0x00>;
def : BankedReg<"r9_usr",   0x01>;
def : BankedReg<"r10_usr",  0x02>;
def : BankedReg<"r11_usr",  0x03>;
def : BankedReg<"r12_usr",  0x04>;
def : BankedReg<"sp_usr",   0x05>;
def : BankedReg<"lr_usr",   0x06>;
def : BankedReg<"r8_fiq",   0x08>;
def : BankedReg<"r9_fiq",   0x09>;
def : BankedReg<"r10_fiq",  0x0a>;
def : BankedReg<"r11_fiq",  0x0b>;
def : BankedReg<"r12_fiq",  0x0c>;
def : BankedReg<"sp_fiq",   0x0d>;
def : BankedReg<"lr_fiq",   0x0e>;
def : BankedReg<"lr_irq",   0x10>;
def : BankedReg<"sp_irq",   0x11>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 186-202
```tablegen
def : BankedReg<"lr_svc",   0x12>;
def : BankedReg<"sp_svc",   0x13>;
def : BankedReg<"lr_abt",   0x14>;
def : BankedReg<"sp_abt",   0x15>;
def : BankedReg<"lr_und",   0x16>;
def : BankedReg<"sp_und",   0x17>;
def : BankedReg<"lr_mon",   0x1c>;
def : BankedReg<"sp_mon",   0x1d>;
def : BankedReg<"elr_hyp",  0x1e>;
def : BankedReg<"sp_hyp",   0x1f>;
def : BankedReg<"spsr_fiq", 0x2e>;
def : BankedReg<"spsr_irq", 0x30>;
def : BankedReg<"spsr_svc", 0x32>;
def : BankedReg<"spsr_abt", 0x34>;
def : BankedReg<"spsr_und", 0x36>;
def : BankedReg<"spsr_mon", 0x3c>;
def : BankedReg<"spsr_hyp", 0x3e>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

## Key Concepts / 关键概念

- EN: Primary role: target descriptions, instruction records, and code-generation metadata.
  - CN: 核心职责：目标描述、指令记录以及代码生成元数据。
- EN: LLVM TableGen DSL records drive generated enums, matchers, encoders, or scheduling data.
  - CN: LLVM TableGen DSL 记录会驱动生成枚举、匹配器、编码器或调度数据。
- EN: Bitfield assignments describe exact instruction encodings and per-instruction flags.
  - CN: 位字段赋值用于描述精确的指令编码以及每条指令的标志位。
- EN: Predicates and feature gates control when records apply to a subtarget or ISA variant.
  - CN: 谓词与特性门控决定记录在何种子目标或 ISA 变体下生效。

## Dependencies / 依赖关系

- EN: TableGen includes: `llvm/TableGen/SearchableTable.td`.
  - CN: TableGen 包含项：`llvm/TableGen/SearchableTable.td`。
- EN: Generated backend tables depend on tblgen consuming these records and emitting C++/inc fragments.
  - CN: 生成后的后端表依赖 tblgen 消费这些记录并产出 C++/inc 片段。
