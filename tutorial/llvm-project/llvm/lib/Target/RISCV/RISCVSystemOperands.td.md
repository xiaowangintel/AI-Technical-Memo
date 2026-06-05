# RISCVSystemOperands.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVSystemOperands.td`
- **Repository**: llvm/llvm-project
- **Purpose**: Defines TableGen records for named system operands such as CSRs and related RISC-V metadata. / 使用 TableGen 定义CSR 等具名系统操作数及相关 RISC-V 元数据。
- **Note**: This file uses the TableGen DSL to describe target metadata declaratively. / 该文件使用 TableGen DSL 以声明式方式描述目标元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: Commentary and design intent / 注释与设计意图
```tablegen
//===- RISCVSystemOperands.td ------------------------------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines the symbolic operands permitted for various kinds of
// RISC-V system instruction.
//
//===----------------------------------------------------------------------===//

include "llvm/TableGen/SearchableTable.td"
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 16-39: TableGen class SysReg<string / TableGen 类 SysReg<string
```tablegen
//===----------------------------------------------------------------------===//
// CSR (control and status register read/write) instruction options.
//===----------------------------------------------------------------------===//

class SysReg<string name, bits<12> op> {
  string Name = name;
  // Custom vendor CSRs have a "<vendor>." prefix. Convert these to "<vendor>_"
  // before passing it to the SysRegEncodings GenericEnum below.
  string EnumName = !subst(".", "_", name);
  bits<12> Encoding = op;
  // FIXME: add these additional fields when needed.
  // Privilege Access: Read and Write = 0, 1, 2; Read-Only = 3.
  // Privilege Mode: User = 0, System = 1 or Machine = 3.
  // bits<2> ReadWrite = op{11 - 10};
  // bits<2> XMode = op{9 - 8};
  // Check Extra field name and what bits 7-6 correspond to.
  // bits<2> Extra = op{7 - 6};
  // Register number without the privilege bits.
  // bits<6> Number = op{5 - 0};
  code FeaturesRequired = [{ {} }];
  bit isRV32Only = 0;
  bit isAltName = 0;
  bit isDeprecatedName = 0;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 40-53: TableGen record SysRegsList / TableGen 记录 SysRegsList
```tablegen

def SysRegsList : GenericTable {
  let FilterClass = "SysReg";
  // FIXME: add "ReadWrite", "Mode", "Extra", "Number" fields when needed.
  let Fields = [
    "Name", "Encoding", "FeaturesRequired",
    "isRV32Only", "isAltName", "isDeprecatedName"
  ];

  let PrimaryKey = [ "Encoding" ];
  let PrimaryKeyName = "lookupSysRegByEncoding";
  let PrimaryKeyReturnRange = true;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 54-68: TableGen record SysRegEncodings / TableGen 记录 SysRegEncodings
```tablegen
def SysRegEncodings : GenericEnum {
  let FilterClass = "SysReg";
  let NameField = "EnumName";
  let ValueField = "Encoding";
}

def lookupSysRegByName : SearchIndex {
  let Table = SysRegsList;
  let Key = [ "Name" ];
}

// The following CSR encodings match those given in Tables 2.2,
// 2.3, 2.4, 2.5 and 2.6 in the RISC-V Instruction Set Manual
// Volume II: Privileged Architecture.
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 69-87: TableGen record SysRegFFLAGS / TableGen 记录 SysRegFFLAGS
```tablegen
//===----------------------------------------------------------------------===//
// Unprivileged Floating-Point CSRs
//===----------------------------------------------------------------------===//

def SysRegFFLAGS : SysReg<"fflags", 0x001>;
def SysRegFRM    : SysReg<"frm", 0x002>;
def SysRegFCSR   : SysReg<"fcsr", 0x003>;

//===----------------------------------------------------------------------===//
// Unprivileged Vector CSRs
//===----------------------------------------------------------------------===//
def : SysReg<"vstart", 0x008>;
def : SysReg<"vxsat", 0x009>;
def SysRegVXRM : SysReg<"vxrm", 0x00A>;
def : SysReg<"vcsr", 0x00F>;
def SysRegVL : SysReg<"vl", 0xC20>;
def : SysReg<"vtype", 0xC21>;
def SysRegVLENB: SysReg<"vlenb", 0xC22>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 88-102: TableGen record SEED / TableGen 记录 SEED
```tablegen
//===----------------------------------------------------------------------===//
// Unprivileged Zicfiss extension CSR
//===----------------------------------------------------------------------===//

def : SysReg<"ssp", 0x011>;

//===-----------------------------------------------
// Unprivileged Entropy Source Extension CSR
//===-----------------------------------------------

def SEED : SysReg<"seed", 0x015>;

//===-----------------------------------------------
// Unprivileged Zcmt Extension CSR
//===-----------------------------------------------
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 103-117: TableGen record CYCLE / TableGen 记录 CYCLE
```tablegen

def : SysReg<"jvt", 0x017>;

//===----------------------------------------------------------------------===//
// Unprivileged Counter/Timers
//===----------------------------------------------------------------------===//

def CYCLE   : SysReg<"cycle", 0xC00>;
def TIME    : SysReg<"time", 0xC01>;
def INSTRET : SysReg<"instret", 0xC02>;

// hpmcounter3-hpmcounter31 at 0xC03-0xC1F.
foreach i = 3...31 in
  def : SysReg<"hpmcounter"#i, !add(0xC03, !sub(i, 3))>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 118-131: TableGen record CYCLEH / TableGen 记录 CYCLEH
```tablegen
let isRV32Only = 1 in {
def CYCLEH   : SysReg<"cycleh", 0xC80>;
def TIMEH    : SysReg<"timeh", 0xC81>;
def INSTRETH : SysReg<"instreth", 0xC82>;

// hpmcounter3h-hpmcounter31h at 0xC83-0xC9F.
foreach i = 3...31 in
  def : SysReg<"hpmcounter"#i#"h", !add(0xC83, !sub(i, 3))>;
}

//===----------------------------------------------------------------------===//
// Supervisor Trap Setup
//===----------------------------------------------------------------------===//
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 132-145: TableGen record definition / TableGen 记录定义
```tablegen
def : SysReg<"sstatus", 0x100>;
def : SysReg<"sie", 0x104>;
def : SysReg<"stvec", 0x105>;
def : SysReg<"scounteren", 0x106>;

//===----------------------------------------------------------------------===//
// Supervisor Configuration
//===----------------------------------------------------------------------===//

def : SysReg<"senvcfg", 0x10A>;

//===----------------------------------------------------------------------===//
// Supervisor Counter Setup
//===----------------------------------------------------------------------===//
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 146-161: TableGen record definition / TableGen 记录定义
```tablegen

def : SysReg<"scountinhibit", 0x120>;

//===----------------------------------------------------------------------===//
// Supervisor Trap Handling
//===----------------------------------------------------------------------===//

def : SysReg<"sscratch", 0x140>;
def : SysReg<"sepc", 0x141>;
def : SysReg<"scause", 0x142>;
def : SysReg<"stval", 0x143>;
let isDeprecatedName = 1 in
def : SysReg<"sbadaddr", 0x143>;
def : SysReg<"sip", 0x144>;
def : SysReg<"scountovf", 0xDA0>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 162-177: TableGen record definition / TableGen 记录定义
```tablegen
//===----------------------------------------------------------------------===//
// Supervisor Protection and Translation
//===----------------------------------------------------------------------===//

def : SysReg<"satp", 0x180>;
let isDeprecatedName = 1 in
def : SysReg<"sptbr", 0x180>;

//===----------------------------------------------------------------------===//
// Supervisor Timer Compare
//===----------------------------------------------------------------------===//

def : SysReg<"stimecmp", 0x14D>;
let isRV32Only = 1 in
def : SysReg<"stimecmph", 0x15D>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 178-192: TableGen record definition / TableGen 记录定义
```tablegen
//===----------------------------------------------------------------------===//
// Debug/Trace Registers
//===----------------------------------------------------------------------===//

def : SysReg<"scontext", 0x5A8>;

//===----------------------------------------------------------------------===//
// Supervisor Resource Management Configuration
//===----------------------------------------------------------------------===//

def : SysReg<"srmcfg", 0x181>;

//===----------------------------------------------------------------------===//
// Supervisor State Enable
//===----------------------------------------------------------------------===//
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 193-210: TableGen record i / TableGen 记录 i
```tablegen

foreach i = 0...3 in {
  def : SysReg<"sstateen"#i, !add(0x10C, i)>;
}

//===----------------------------------------------------------------------===//
// Hypervisor Trap Setup
//===----------------------------------------------------------------------===//

def : SysReg<"hstatus", 0x600>;
def : SysReg<"hedeleg", 0x602>;
def : SysReg<"hideleg", 0x603>;
def : SysReg<"hie", 0x604>;
def : SysReg<"hcounteren", 0x606>;
def : SysReg<"hgeie", 0x607>;
let isRV32Only = 1 in
def : SysReg<"hedelegh", 0x612>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 211-224: TableGen record definition / TableGen 记录定义
```tablegen
//===----------------------------------------------------------------------===//
// Hypervisor Trap Handling
//===----------------------------------------------------------------------===//

def : SysReg<"htval", 0x643>;
def : SysReg<"hip", 0x644>;
def : SysReg<"hvip", 0x645>;
def : SysReg<"htinst", 0x64A>;
def : SysReg<"hgeip", 0xE12>;

//===----------------------------------------------------------------------===//
// Hypervisor Configuration
//===----------------------------------------------------------------------===//
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 225-238: TableGen record definition / TableGen 记录定义
```tablegen
def : SysReg<"henvcfg", 0x60A>;
let isRV32Only = 1 in
def : SysReg<"henvcfgh", 0x61A>;

//===----------------------------------------------------------------------===//
// Hypervisor Protection and Translation
//===----------------------------------------------------------------------===//

def : SysReg<"hgatp", 0x680>;

//===----------------------------------------------------------------------===//
// Debug/Trace Registers
//===----------------------------------------------------------------------===//
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 239-252: TableGen record definition / TableGen 记录定义
```tablegen
def : SysReg<"hcontext", 0x6A8>;

//===----------------------------------------------------------------------===//
// Hypervisor Counter/Timer Virtualization Registers
//===----------------------------------------------------------------------===//

def : SysReg<"htimedelta", 0x605>;
let isRV32Only = 1 in
def : SysReg<"htimedeltah", 0x615>;

//===----------------------------------------------------------------------===//
// Hypervisor State Enable Registers
//===----------------------------------------------------------------------===//
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 253-272: TableGen record i / TableGen 记录 i
```tablegen
foreach i = 0...3 in {
  def : SysReg<"hstateen"#i, !add(0x60C, i)>;
  let isRV32Only = 1 in
  def : SysReg<"hstateen"#i#"h", !add(0x61C, i)>;
}

//===----------------------------------------------------------------------===//
// Virtual Supervisor Registers
//===----------------------------------------------------------------------===//

def : SysReg<"vsstatus", 0x200>;
def : SysReg<"vsie", 0x204>;
def : SysReg<"vstvec", 0x205>;
def : SysReg<"vsscratch", 0x240>;
def : SysReg<"vsepc", 0x241>;
def : SysReg<"vscause", 0x242>;
def : SysReg<"vstval", 0x243>;
def : SysReg<"vsip", 0x244>;
def : SysReg<"vsatp", 0x280>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 273-290: TableGen record definition / TableGen 记录定义
```tablegen
//===----------------------------------------------------------------------===//
// Virtual Supervisor Timer Compare
//===----------------------------------------------------------------------===//

def : SysReg<"vstimecmp", 0x24D>;
let isRV32Only = 1 in
def : SysReg<"vstimecmph", 0x25D>;

//===----------------------------------------------------------------------===//
// Machine Information Registers
//===----------------------------------------------------------------------===//

def : SysReg<"mvendorid", 0xF11>;
def : SysReg<"marchid", 0xF12>;
def : SysReg<"mimpid", 0xF13>;
def : SysReg<"mhartid", 0xF14>;
def : SysReg<"mconfigptr", 0xF15>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 291-306: TableGen record definition / TableGen 记录定义
```tablegen
//===----------------------------------------------------------------------===//
// Machine Trap Setup
//===----------------------------------------------------------------------===//

def : SysReg<"mstatus", 0x300>;
def : SysReg<"misa", 0x301>;
def : SysReg<"medeleg", 0x302>;
def : SysReg<"mideleg", 0x303>;
def : SysReg<"mie", 0x304>;
def : SysReg<"mtvec", 0x305>;
def : SysReg<"mcounteren", 0x306>;
let isRV32Only = 1 in {
def : SysReg<"mstatush", 0x310>;
def : SysReg<"medelegh", 0x312>;
} // isRV32Only
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 307-320: TableGen record definition / TableGen 记录定义
```tablegen
//===----------------------------------------------------------------------===//
// Machine Trap Handling
//===----------------------------------------------------------------------===//
def : SysReg<"mscratch", 0x340>;
def : SysReg<"mepc", 0x341>;
def : SysReg<"mcause", 0x342>;
def : SysReg<"mtval", 0x343>;
let isDeprecatedName = 1 in
def : SysReg<"mbadaddr", 0x343>;
def : SysReg<"mip", 0x344>;
def : SysReg<"mtinst", 0x34A>;
def : SysReg<"mtval2", 0x34B>;

//===----------------------------------------------------------------------===//
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 321-334: TableGen record definition / TableGen 记录定义
```tablegen
// Machine Configuration
//===----------------------------------------------------------------------===//

def : SysReg<"menvcfg", 0x30A>;
let isRV32Only = 1 in
def : SysReg<"menvcfgh", 0x31A>;
def : SysReg<"mseccfg", 0x747>;
let isRV32Only = 1 in
def : SysReg<"mseccfgh", 0x757>;

//===----------------------------------------------------------------------===//
// Machine Protection and Translation
//===----------------------------------------------------------------------===//
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 335-348: TableGen record i / TableGen 记录 i
```tablegen
// pmpcfg0-pmpcfg15 at 0x3A0-0x3AF. Odd-numbered registers are RV32-only.
foreach i = 0...15 in {
  let isRV32Only = !and(i, 1) in
  def : SysReg<"pmpcfg"#i, !add(0x3A0, i)>;
}

// pmpaddr0-pmpaddr63 at 0x3B0-0x3EF.
foreach i = 0...63 in
  def : SysReg<"pmpaddr"#i, !add(0x3B0, i)>;

//===----------------------------------------------------------------------===//
// Machine State Enable Registers
//===----------------------------------------------------------------------===//
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 349-362: TableGen record i / TableGen 记录 i
```tablegen
foreach i = 0...3 in {
  def : SysReg<"mstateen"#i, !add(0x30C, i)>;
  let isRV32Only = 1 in
  def : SysReg<"mstateen"#i#"h", !add(0x31C, i)>;
}

//===-----------------------------------------------
// Resumable Non-Maskable Interrupts(Smrnmi) CSRs
//===-----------------------------------------------
def : SysReg<"mnscratch", 0x740>;
def : SysReg<"mnepc", 0x741>;
def : SysReg<"mncause", 0x742>;
def : SysReg<"mnstatus", 0x744>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 363-377: TableGen record i / TableGen 记录 i
```tablegen
//===----------------------------------------------------------------------===//
// Machine Counter and Timers
//===----------------------------------------------------------------------===//

def : SysReg<"mcycle", 0xB00>;
def : SysReg<"minstret", 0xB02>;

// mhpmcounter3-mhpmcounter31 at 0xB03-0xB1F.
foreach i = 3...31 in
  def : SysReg<"mhpmcounter"#i, !add(0xB03, !sub(i, 3))>;

let isRV32Only = 1 in {
def: SysReg<"mcycleh", 0xB80>;
def: SysReg<"minstreth", 0xB82>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 378-392: TableGen record i / TableGen 记录 i
```tablegen
// mhpmcounter3h-mhpmcounter31h at 0xB83-0xB9F.
foreach i = 3...31 in
  def : SysReg<"mhpmcounter"#i#"h", !add(0xB83, !sub(i, 3))>;
}

//===----------------------------------------------------------------------===//
// Machine Counter Setup
//===----------------------------------------------------------------------===//

def : SysReg<"mcountinhibit", 0x320>;

// mhpmevent3-mhpmevent31 at 0x323-0x33F.
foreach i = 3...31 in
  def : SysReg<"mhpmevent"#i, !add(0x323, !sub(i, 3))>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 393-412: TableGen record i / TableGen 记录 i
```tablegen
// mhpmevent3h-mhpmevent31h at 0x723-0x73F
foreach i = 3...31 in {
  let isRV32Only = 1 in
  def : SysReg<"mhpmevent"#i#"h", !add(0x723, !sub(i, 3))>;
}

//===----------------------------------------------------------------------===//
// Debug/ Trace Registers (shared with Debug Mode)
//===----------------------------------------------------------------------===//

def : SysReg<"tselect", 0x7A0>;
def : SysReg<"tdata1", 0x7A1>;
let isAltName = 1 in {
def : SysReg<"mcontrol", 0x7A1>;
def : SysReg<"mcontrol6", 0x7A1>;
def : SysReg<"icount", 0x7A1>;
def : SysReg<"itrigger", 0x7A1>;
def : SysReg<"etrigger", 0x7A1>;
def : SysReg<"tmexttrigger", 0x7A1>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 413-426: TableGen record definition / TableGen 记录定义
```tablegen
def : SysReg<"tdata2", 0x7A2>;
def : SysReg<"tdata3", 0x7A3>;
let isAltName = 1 in {
def : SysReg<"textra32", 0x7A3>;
def : SysReg<"textra64", 0x7A3>;
}
def : SysReg<"tinfo", 0x7A4>;
def : SysReg<"tcontrol", 0x7A5>;
def : SysReg<"mcontext", 0x7A8>;
def : SysReg<"mscontext", 0x7AA>;

//===----------------------------------------------------------------------===//
// Debug Mode Registers
//===----------------------------------------------------------------------===//
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 427-440: TableGen record definition / TableGen 记录定义
```tablegen

def : SysReg<"dcsr", 0x7B0>;
def : SysReg<"dpc", 0x7B1>;

// "dscratch" is an alternative name for "dscratch0" which appeared in earlier
// drafts of the RISC-V debug spec
def : SysReg<"dscratch0", 0x7B2>;
let isAltName = 1 in
def : SysReg<"dscratch", 0x7B2>;
def : SysReg<"dscratch1", 0x7B3>;

//===-----------------------------------------------
// Advanced Interrupt Architecture
//===-----------------------------------------------
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 441-462: TableGen record i / TableGen 记录 i
```tablegen

// Machine-level CSRs
def : SysReg<"miselect", 0x350>;
def : SysReg<"mireg", 0x351>;
foreach i = 2...3 in {
  def : SysReg<"mireg"#i, !add(0x350, i)>;
}
foreach i = 4...6 in {
  def : SysReg<"mireg"#i, !add(0x351, i)>;
}
def : SysReg<"mtopei", 0x35C>;
def : SysReg<"mtopi", 0xFB0>;
def : SysReg<"mvien", 0x308>;
def : SysReg<"mvip", 0x309>;
let isRV32Only = 1 in {
def : SysReg<"midelegh", 0x313>;
def : SysReg<"mieh", 0x314>;
def : SysReg<"mvienh", 0x318>;
def : SysReg<"mviph", 0x319>;
def : SysReg<"miph", 0x354>;
} // isRV32Only
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 463-478: TableGen record i / TableGen 记录 i
```tablegen
// Supervisor-level CSRs
def : SysReg<"siselect", 0x150>;
def : SysReg<"sireg", 0x151>;
foreach i = 2...3 in {
  def : SysReg<"sireg"#i, !add(0x150, i)>;
}
foreach i = 4...6 in {
  def : SysReg<"sireg"#i, !add(0x151, i)>;
}
def : SysReg<"stopei", 0x15C>;
def : SysReg<"stopi", 0xDB0>;
let isRV32Only = 1 in {
def : SysReg<"sieh", 0x114>;
def : SysReg<"siph", 0x154>;
} // isRV32Only
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 479-503: TableGen record i / TableGen 记录 i
```tablegen
// Hypervisor and VS CSRs
def : SysReg<"hvien", 0x608>;
def : SysReg<"hvictl", 0x609>;
def : SysReg<"hviprio1", 0x646>;
def : SysReg<"hviprio2", 0x647>;
def : SysReg<"vsiselect", 0x250>;
def : SysReg<"vsireg", 0x251>;
foreach i = 2...3 in {
  def : SysReg<"vsireg"#i, !add(0x250, i)>;
}
foreach i = 4...6 in {
  def : SysReg<"vsireg"#i, !add(0x251, i)>;
}
def : SysReg<"vstopei", 0x25C>;
def : SysReg<"vstopi", 0xEB0>;
let isRV32Only = 1 in {
def : SysReg<"hidelegh", 0x613>;
def : SysReg<"hvienh", 0x618>;
def : SysReg<"hviph", 0x655>;
def : SysReg<"hviprio1h", 0x656>;
def : SysReg<"hviprio2h", 0x657>;
def : SysReg<"vsieh", 0x214>;
def : SysReg<"vsiph", 0x254>;
} // isRV32Only
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 504-522: TableGen record definition / TableGen 记录定义
```tablegen
//===-----------------------------------------------
// Control Transfer Records CSRs
//===-----------------------------------------------
def : SysReg<"sctrctl", 0x14e>;
def : SysReg<"sctrstatus", 0x14f>;
def : SysReg<"sctrdepth", 0x15f>;
def : SysReg<"vsctrctl", 0x24e>;
def : SysReg<"mctrctl", 0x34e>;

//===-----------------------------------------------
// Cycle and Instret Privilege Mode Filtering (Smcntrpmf)
//===-----------------------------------------------
def : SysReg<"mcyclecfg", 0x321>;
def : SysReg<"minstretcfg", 0x322>;
let isRV32Only = 1 in {
def : SysReg<"mcyclecfgh", 0x721>;
def : SysReg<"minstretcfgh", 0x722>;
} // isRV32Only
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 523-543: TableGen record definition / TableGen 记录定义
```tablegen
//===-----------------------------------------------
// Vendor CSRs
//===-----------------------------------------------

// XSfmclic
let FeaturesRequired = [{ {RISCV::FeatureVendorXSfmclic} }] in {
def : SysReg<"sf.mtvt", 0x307>;
def : SysReg<"sf.mnxti", 0x345>;
def : SysReg<"sf.mintstatus", 0x346>;
def : SysReg<"sf.mscratchcsw", 0x348>;
def : SysReg<"sf.mscratchcswl", 0x349>;
}

// XSfsclic
let FeaturesRequired = [{ {RISCV::FeatureVendorXSfsclic} }] in {
def : SysReg<"sf.stvt", 0x107>;
def : SysReg<"sf.snxti", 0x145>;
def : SysReg<"sf.sintstatus", 0x146>;
def : SysReg<"sf.sscratchcsw", 0x148>;
def : SysReg<"sf.sscratchcswl", 0x149>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 544-557: TableGen record i / TableGen 记录 i
```tablegen

// Xqciint
let FeaturesRequired = [{ {RISCV::FeatureVendorXqciint} }], isRV32Only = 1 in {
def : SysReg<"qc.mmcr", 0x7C0>;
def : SysReg<"qc.mntvec", 0x7C3>;
def : SysReg<"qc.mstktopaddr", 0x7C4>;
def : SysReg<"qc.mstkbottomaddr", 0x7C5>;
def : SysReg<"qc.mthreadptr", 0x7C8>;
def : SysReg<"qc.mcause", 0x7C9>;

foreach i = 0 - 7 in {
  def : SysReg<"qc.mclicip" # i, !add(0x7F0, i)>;
  def : SysReg<"qc.mclicie" # i, !add(0x7F8, i)>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 558-585: TableGen record i / TableGen 记录 i
```tablegen

foreach i = 0 - 31 in {
  def : SysReg<"qc.mclicilvl" # !if(!lt(i, 10), "0", "") # i, 
               !add(0xBC0, i)>;
}

foreach i = 0 - 3 in {
  def : SysReg<"qc.mwpstartaddr" # i, !add(0x7D0, i)>;
  def : SysReg<"qc.mwpendaddr" # i,   !add(0x7D4, i)>;
}
} // FeatureVendorXqciint, isRV32Only

// XAIFET
let FeaturesRequired = [{ {RISCV::FeatureVendorXAIFET} }] in {
  def : SysReg<"aif.cache_invalidate", 0x7d0>;
  def : SysReg<"aif.dcache_debug", 0xfc0>;
  def : SysReg<"aif.evict_sw", 0x7f9>;
  def : SysReg<"aif.evict_va", 0x89f>;
  def : SysReg<"aif.excl_mode", 0x7d3>;
  def : SysReg<"aif.fcc", 0x821>;
  def : SysReg<"aif.fccnb", 0xcc0>;
  def : SysReg<"aif.flb", 0x820>;
  def : SysReg<"aif.flush_sw", 0x7fb>;
  def : SysReg<"aif.flush_va", 0x8bf>;
  def : SysReg<"aif.gsc_progress", 0x840>;
  def : SysReg<"aif.hartid", 0xcd0>;
  def : SysReg<"aif.lock_sw", 0x7fd>;
  def : SysReg<"aif.lock_va", 0x8df>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 586-613: TableGen record definition / TableGen 记录定义
```tablegen
  def : SysReg<"aif.matp", 0x7c0>;
  def : SysReg<"aif.mbusaddr", 0x7d5>;
  def : SysReg<"aif.mcache_control", 0x7e0>;
  def : SysReg<"aif.menable_shadows", 0x7d2>;
  def : SysReg<"aif.minstmask", 0x7cd>;
  def : SysReg<"aif.minstmatch", 0x7ce>;
  def : SysReg<"aif.portctrl0", 0x9cc>;
  def : SysReg<"aif.portctrl1", 0x9cd>;
  def : SysReg<"aif.portctrl2", 0x9ce>;
  def : SysReg<"aif.portctrl3", 0x9cf>;
  def : SysReg<"aif.porthead0", 0xcc8>;
  def : SysReg<"aif.porthead1", 0xcc9>;
  def : SysReg<"aif.porthead2", 0xcca>;
  def : SysReg<"aif.porthead3", 0xccb>;
  def : SysReg<"aif.portheadnb0", 0xccc>;
  def : SysReg<"aif.portheadnb1", 0xccd>;
  def : SysReg<"aif.portheadnb2", 0xcce>;
  def : SysReg<"aif.portheadnb3", 0xccf>;
  def : SysReg<"aif.prefetch_va", 0x81f>;
  def : SysReg<"aif.stall", 0x822>;
  def : SysReg<"aif.tensor_conv_ctrl", 0x803>;
  def : SysReg<"aif.tensor_conv_size", 0x802>;
  def : SysReg<"aif.tensor_coop", 0x804>;
  def : SysReg<"aif.tensor_error", 0x808>;
  def : SysReg<"aif.tensor_fma", 0x801>;
  def : SysReg<"aif.tensor_load", 0x83f>;
  def : SysReg<"aif.tensor_load_l2", 0x85f>;
  def : SysReg<"aif.tensor_mask", 0x805>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 614-626: TableGen record definition / TableGen 记录定义
```tablegen
  def : SysReg<"aif.tensor_quant", 0x806>;
  def : SysReg<"aif.tensor_reduce", 0x800>;
  def : SysReg<"aif.tensor_store", 0x87f>;
  def : SysReg<"aif.tensor_wait", 0x830>;
  def : SysReg<"aif.tex_send", 0x807>;
  def : SysReg<"aif.ucache_control", 0x810>;
  def : SysReg<"aif.unlock_sw", 0x7ff>;
  def : SysReg<"aif.unlock_va", 0x8ff>;
  def : SysReg<"aif.validation0", 0x8d0>;
  def : SysReg<"aif.validation1", 0x8d1>;
  def : SysReg<"aif.validation2", 0x8d2>;
  def : SysReg<"aif.validation3", 0x8d3>;
} // FeatureVendorXAIFET
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

## Key Concepts / 关键概念
- **System operand metadata** / **系统操作数元数据**
- **TableGen DSL** / **TableGen DSL**

## Dependencies / 依赖关系
- `llvm/TableGen/SearchableTable.td` — Directly referenced by this file. / 该文件直接引用的依赖。
