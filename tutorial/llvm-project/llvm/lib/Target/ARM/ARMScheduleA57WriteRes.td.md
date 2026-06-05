# ARMScheduleA57WriteRes.td — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/ARMScheduleA57WriteRes.td`
- Repository: `llvm-project`
- Purpose (EN): Defines `ARMScheduleA57WriteRes` in LLVM TableGen DSL for the ARM backend, covering target descriptions, instruction records, and code-generation metadata.
- 用途 (CN): 使用 LLVM TableGen DSL 定义 ARM 后端中的 `ARMScheduleA57WriteRes`，涵盖目标描述、指令记录以及代码生成元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18
```tablegen
//=- ARMScheduleA57WriteRes.td - ARM Cortex-A57 Write Res ---*- tablegen -*-=//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Contains all of the Cortex-A57 specific SchedWriteRes types. The approach
// below is to define a generic SchedWriteRes for every combination of
// latency and microOps. The naming conventions is to use a prefix, one field
// for latency, and one or more microOp count/type designators.
//   Prefix: A57Write
//   Latency: #cyc
//   MicroOp Count/Types: #(B|I|M|L|S|X|W|V)
//
// e.g. A57Write_6cyc_1I_6S_4V means the total latency is 6 and there are
//      11 micro-ops to be issued as follows: one to I pipe, six to S pipes and
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 19-21
```tablegen
//      four to V pipes.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 23-24
```tablegen
//===----------------------------------------------------------------------===//
// Define Generic 1 micro-op types
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 26-43
```tablegen
def A57Write_5cyc_1M  : SchedWriteRes<[A57UnitM]> { let Latency = 5;  }
def A57Write_5cyc_1V  : SchedWriteRes<[A57UnitV]> { let Latency = 5;  }
def A57Write_5cyc_1W  : SchedWriteRes<[A57UnitW]> { let Latency = 5;  }
def A57Write_10cyc_1V : SchedWriteRes<[A57UnitV]> { let Latency = 10; }
def A57Write_17cyc_1W : SchedWriteRes<[A57UnitW]> { let Latency = 17;
                                                    let ReleaseAtCycles = [17]; }
def A57Write_18cyc_1X : SchedWriteRes<[A57UnitX]> { let Latency = 18;
                                                    let ReleaseAtCycles = [18]; }
def A57Write_19cyc_1M : SchedWriteRes<[A57UnitM]> { let Latency = 19;
                                                    let ReleaseAtCycles = [19]; }
def A57Write_20cyc_1M : SchedWriteRes<[A57UnitM]> { let Latency = 20;
                                                    let ReleaseAtCycles = [20]; }
def A57Write_1cyc_1B  : SchedWriteRes<[A57UnitB]> { let Latency = 1;  }
def A57Write_1cyc_1I  : SchedWriteRes<[A57UnitI]> { let Latency = 1;
                                                    let ReleaseAtCycles = [1]; }
def A57Write_2cyc_1I  : SchedWriteRes<[A57UnitI]> { let Latency = 2;
                                                    let ReleaseAtCycles = [1]; }
def A57Write_3cyc_1I  : SchedWriteRes<[A57UnitI]> { let Latency = 3;  }
```
- EN: Defines TableGen record `A57Write_5cyc_1M` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57Write_5cyc_1M`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 44-58
```tablegen
def A57Write_1cyc_1S  : SchedWriteRes<[A57UnitS]> { let Latency = 1;  }
def A57Write_2cyc_1S  : SchedWriteRes<[A57UnitS]> { let Latency = 2;  }
def A57Write_3cyc_1S  : SchedWriteRes<[A57UnitS]> { let Latency = 3;  }
def A57Write_2cyc_1M  : SchedWriteRes<[A57UnitM]> { let Latency = 2;
                                                    let ReleaseAtCycles = [1]; }
def A57Write_32cyc_1W : SchedWriteRes<[A57UnitW]> { let Latency = 32;
                                                    let ReleaseAtCycles = [32]; }
def A57Write_32cyc_1X : SchedWriteRes<[A57UnitX]> { let Latency = 32;
                                                    let ReleaseAtCycles = [32]; }
def A57Write_35cyc_1M : SchedWriteRes<[A57UnitM]> { let Latency = 35;
                                                    let ReleaseAtCycles = [35]; }
def A57Write_3cyc_1M  : SchedWriteRes<[A57UnitM]> { let Latency = 3;  }
def A57Write_3cyc_1V  : SchedWriteRes<[A57UnitV]> { let Latency = 3;  }
def A57Write_3cyc_1W  : SchedWriteRes<[A57UnitW]> { let Latency = 3;  }
def A57Write_3cyc_1X  : SchedWriteRes<[A57UnitX]> { let Latency = 3;  }
```
- EN: Defines TableGen record `A57Write_1cyc_1S` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57Write_1cyc_1S`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 60-65
```tablegen
// A57Write_3cyc_1L - A57Write_20cyc_1L
foreach Lat = 3-20 in {
  def A57Write_#Lat#cyc_1L : SchedWriteRes<[A57UnitL]> {
    let Latency = Lat;
  }
}
```
- EN: Defines TableGen record `A57Write_` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57Write_`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 67-72
```tablegen
// A57Write_4cyc_1S - A57Write_16cyc_1S
foreach Lat = 4-16 in {
  def A57Write_#Lat#cyc_1S : SchedWriteRes<[A57UnitS]> {
    let Latency = Lat;
  }
}
```
- EN: Defines TableGen record `A57Write_` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57Write_`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 74-83
```tablegen
def A57Write_4cyc_1M  : SchedWriteRes<[A57UnitM]> { let Latency = 4;  }
def A57Write_4cyc_1X  : SchedWriteRes<[A57UnitX]> { let Latency = 4;  }
def A57Write_4cyc_1W  : SchedWriteRes<[A57UnitW]> { let Latency = 4;  }
def A57Write_5cyc_1X  : SchedWriteRes<[A57UnitX]> { let Latency = 5;  }
def A57Write_6cyc_1X  : SchedWriteRes<[A57UnitX]> { let Latency = 6;  }
def A57Write_6cyc_1W  : SchedWriteRes<[A57UnitW]> { let Latency = 6;  }
def A57Write_8cyc_1V  : SchedWriteRes<[A57UnitV]> { let Latency = 8;  }
def A57Write_9cyc_1V  : SchedWriteRes<[A57UnitV]> { let Latency = 9;  }
def A57Write_6cyc_1M  : SchedWriteRes<[A57UnitM]> { let Latency = 6;  }
def A57Write_6cyc_1V  : SchedWriteRes<[A57UnitV]> { let Latency = 6;  }
```
- EN: Defines TableGen record `A57Write_4cyc_1M` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57Write_4cyc_1M`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 86-87
```tablegen
//===----------------------------------------------------------------------===//
// Define Generic 2 micro-op types
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 89-106
```tablegen
def A57Write_64cyc_2X    : SchedWriteRes<[A57UnitX, A57UnitX]> {
  let Latency     = 64;
  let NumMicroOps = 2;
  let ReleaseAtCycles = [32, 32];
}
def A57Write_6cyc_1I_1L  : SchedWriteRes<[A57UnitI,
                                          A57UnitL]> {
  let Latency     = 6;
  let NumMicroOps = 2;
}
def A57Write_6cyc_1V_1X  : SchedWriteRes<[A57UnitV,
                                          A57UnitX]> {
  let Latency     = 6;
  let NumMicroOps = 2;
}
def A57Write_7cyc_1V_1X  : SchedWriteRes<[A57UnitV,
                                          A57UnitX]> {
  let Latency     = 7;
```
- EN: Defines TableGen record `A57Write_64cyc_2X` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57Write_64cyc_2X`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 107-124
```tablegen
  let NumMicroOps = 2;
}
def A57Write_8cyc_1L_1V  : SchedWriteRes<[A57UnitL,
                                          A57UnitV]> {
  let Latency     = 8;
  let NumMicroOps = 2;
}
def A57Write_9cyc_1L_1V  : SchedWriteRes<[A57UnitL,
                                          A57UnitV]> {
  let Latency     = 9;
  let NumMicroOps = 2;
}
def A57Write_9cyc_2V     : SchedWriteRes<[A57UnitV, A57UnitV]> {
  let Latency     = 9;
  let NumMicroOps = 2;
}
def A57Write_8cyc_2X     : SchedWriteRes<[A57UnitX, A57UnitX]> {
  let Latency     = 8;
```
- EN: Defines TableGen record `A57Write_8cyc_1L_1V` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57Write_8cyc_1L_1V`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 125-142
```tablegen
  let NumMicroOps = 2;
}
def A57Write_6cyc_2L     : SchedWriteRes<[A57UnitL, A57UnitL]> {
  let Latency     = 6;
  let NumMicroOps = 2;
}
def A57Write_6cyc_2V     : SchedWriteRes<[A57UnitV, A57UnitV]> {
  let Latency     = 6;
  let NumMicroOps = 2;
}
def A57Write_6cyc_2W     : SchedWriteRes<[A57UnitW, A57UnitW]> {
  let Latency     = 6;
  let NumMicroOps = 2;
}
def A57Write_5cyc_1I_1L  : SchedWriteRes<[A57UnitI,
                                          A57UnitL]> {
  let Latency     = 5;
  let NumMicroOps = 2;
```
- EN: Defines TableGen record `A57Write_6cyc_2L` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57Write_6cyc_2L`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 143-160
```tablegen
}
def A57Write_5cyc_1I_1M  : SchedWriteRes<[A57UnitI,
                                          A57UnitM]> {
  let Latency     = 5;
  let NumMicroOps = 2;
}
def A57Write_5cyc_2V     : SchedWriteRes<[A57UnitV, A57UnitV]> {
  let Latency     = 5;
  let NumMicroOps = 2;
}
def A57Write_5cyc_2X     : SchedWriteRes<[A57UnitX, A57UnitX]> {
  let Latency     = 5;
  let NumMicroOps = 2;
}
def A57Write_10cyc_1L_1V : SchedWriteRes<[A57UnitL,
                                          A57UnitV]> {
  let Latency     = 10;
  let NumMicroOps = 2;
```
- EN: Defines TableGen record `A57Write_5cyc_1I_1M` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57Write_5cyc_1I_1M`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 161-178
```tablegen
}
def A57Write_10cyc_2V    : SchedWriteRes<[A57UnitV, A57UnitV]> {
  let Latency     = 10;
  let NumMicroOps = 2;
}
def A57Write_1cyc_1B_1I  : SchedWriteRes<[A57UnitB,
                                          A57UnitI]> {
  let Latency     = 1;
  let NumMicroOps = 2;
}
def A57Write_1cyc_1I_1S  : SchedWriteRes<[A57UnitI,
                                          A57UnitS]> {
  let Latency     = 1;
  let NumMicroOps = 2;
}
def A57Write_1cyc_1S_1I  : SchedWriteRes<[A57UnitS,
                                          A57UnitI]> {
  let Latency     = 1;
```
- EN: Defines TableGen record `A57Write_10cyc_2V` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57Write_10cyc_2V`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 179-196
```tablegen
  let NumMicroOps = 2;
}
def A57Write_2cyc_1S_1I  : SchedWriteRes<[A57UnitS,
                                          A57UnitI]> {
  let Latency     = 2;
  let NumMicroOps = 2;
}
def A57Write_3cyc_1S_1I  : SchedWriteRes<[A57UnitS,
                                          A57UnitI]> {
  let Latency     = 3;
  let NumMicroOps = 2;
}
def A57Write_1cyc_1S_1M  : SchedWriteRes<[A57UnitS,
                                          A57UnitM]> {
  let Latency     = 1;
  let NumMicroOps = 2;
}
def A57Write_2cyc_1B_1I  : SchedWriteRes<[A57UnitB,
```
- EN: Defines TableGen record `A57Write_2cyc_1S_1I` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57Write_2cyc_1S_1I`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 197-214
```tablegen
                                          A57UnitI]> {
  let Latency     = 2;
  let NumMicroOps = 2;
}
def A57Write_3cyc_1B_1I  : SchedWriteRes<[A57UnitB,
                                          A57UnitI]> {
  let Latency     = 3;
  let NumMicroOps = 2;
}
def A57Write_6cyc_1B_1L  : SchedWriteRes<[A57UnitB,
                                          A57UnitI]> {
  let Latency     = 6;
  let NumMicroOps = 2;
}
def A57Write_2cyc_1I_1M  : SchedWriteRes<[A57UnitI,
                                          A57UnitM]> {
  let Latency     = 2;
  let NumMicroOps = 2;
```
- EN: Defines TableGen record `A57Write_3cyc_1B_1I` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57Write_3cyc_1B_1I`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 215-232
```tablegen
}
def A57Write_2cyc_2S     : SchedWriteRes<[A57UnitS, A57UnitS]> {
  let Latency     = 2;
  let NumMicroOps = 2;
}
def A57Write_2cyc_2V     : SchedWriteRes<[A57UnitV, A57UnitV]> {
  let Latency     = 2;
  let NumMicroOps = 2;
}
def A57Write_36cyc_2X    : SchedWriteRes<[A57UnitX, A57UnitX]> {
  let Latency     = 36;
  let NumMicroOps = 2;
  let ReleaseAtCycles = [18, 18];
}
def A57Write_3cyc_1I_1M  : SchedWriteRes<[A57UnitI,
                                          A57UnitM]> {
  let Latency     = 3;
  let NumMicroOps = 2;
```
- EN: Defines TableGen record `A57Write_2cyc_2S` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57Write_2cyc_2S`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 233-238
```tablegen
}
def A57Write_4cyc_1I_1M  : SchedWriteRes<[A57UnitI,
                                          A57UnitM]> {
  let Latency     = 4;
  let NumMicroOps = 2;
}
```
- EN: Defines TableGen record `A57Write_4cyc_1I_1M` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57Write_4cyc_1I_1M`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 240-245
```tablegen
// A57Write_3cyc_1L_1I - A57Write_20cyc_1L_1I
foreach Lat = 3-20 in {
  def A57Write_#Lat#cyc_1L_1I : SchedWriteRes<[A57UnitL, A57UnitI]> {
    let Latency = Lat; let NumMicroOps = 2;
  }
}
```
- EN: Defines TableGen record `A57Write_` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57Write_`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 247-264
```tablegen
def A57Write_3cyc_1I_1S  : SchedWriteRes<[A57UnitI,
                                          A57UnitS]> {
  let Latency     = 3;
  let NumMicroOps = 2;
}
def A57Write_3cyc_1S_1V  : SchedWriteRes<[A57UnitS,
                                          A57UnitV]> {
  let Latency     = 3;
  let NumMicroOps = 2;
}
def A57Write_4cyc_1S_1V  : SchedWriteRes<[A57UnitS,
                                          A57UnitV]> {
  let Latency     = 4;
  let NumMicroOps = 2;
}
def A57Write_3cyc_2V     : SchedWriteRes<[A57UnitV, A57UnitV]> {
  let Latency     = 3;
  let NumMicroOps = 2;
```
- EN: Defines TableGen record `A57Write_3cyc_1I_1S` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57Write_3cyc_1I_1S`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 265-265
```tablegen
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 267-272
```tablegen
// A57Write_4cyc_1S_1I - A57Write_16cyc_1S_1I
foreach Lat = 4-16 in {
  def A57Write_#Lat#cyc_1S_1I : SchedWriteRes<[A57UnitS, A57UnitI]> {
    let Latency = Lat; let NumMicroOps = 2;
  }
}
```
- EN: Defines TableGen record `A57Write_` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57Write_`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 274-277
```tablegen
def A57Write_4cyc_2X     : SchedWriteRes<[A57UnitX, A57UnitX]> {
  let Latency     = 4;
  let NumMicroOps = 2;
}
```
- EN: Defines TableGen record `A57Write_4cyc_2X` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57Write_4cyc_2X`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 280-281
```tablegen
//===----------------------------------------------------------------------===//
// Define Generic 3 micro-op types
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 283-300
```tablegen
def A57Write_10cyc_3V       : SchedWriteRes<[A57UnitV, A57UnitV, A57UnitV]> {
  let Latency     = 10;
  let NumMicroOps = 3;
}
def A57Write_2cyc_1I_2S     : SchedWriteRes<[A57UnitI,
                                             A57UnitS, A57UnitS]> {
  let Latency     = 2;
  let NumMicroOps = 3;
}
def A57Write_3cyc_1I_1S_1V  : SchedWriteRes<[A57UnitI,
                                             A57UnitS,
                                             A57UnitV]> {
  let Latency     = 3;
  let NumMicroOps = 3;
}
def A57Write_3cyc_1S_1V_1I  : SchedWriteRes<[A57UnitS,
                                             A57UnitV,
                                             A57UnitI]> {
```
- EN: Defines TableGen record `A57Write_10cyc_3V` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57Write_10cyc_3V`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 301-318
```tablegen
  let Latency     = 3;
  let NumMicroOps = 3;
}
def A57Write_4cyc_1S_1V_1I  : SchedWriteRes<[A57UnitS,
                                             A57UnitV,
                                             A57UnitI]> {
  let Latency     = 4;
  let NumMicroOps = 3;
}
def A57Write_4cyc_1I_1L_1M  : SchedWriteRes<[A57UnitI, A57UnitL, A57UnitM]> {
  let Latency     = 4;
  let NumMicroOps = 3;
}
def A57Write_8cyc_1L_1V_1I  : SchedWriteRes<[A57UnitL,
                                             A57UnitV,
                                             A57UnitI]> {
  let Latency     = 8;
  let NumMicroOps = 3;
```
- EN: Defines TableGen record `A57Write_4cyc_1S_1V_1I` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57Write_4cyc_1S_1V_1I`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 319-325
```tablegen
}
def A57Write_9cyc_1L_1V_1I  : SchedWriteRes<[A57UnitL,
                                             A57UnitV,
                                             A57UnitI]> {
  let Latency     = 9;
  let NumMicroOps = 3;
}
```
- EN: Defines TableGen record `A57Write_9cyc_1L_1V_1I` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `A57Write_9cyc_1L_1V_1I`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

## Key Concepts / 关键概念

- EN: Primary role: target descriptions, instruction records, and code-generation metadata.
  - CN: 核心职责：目标描述、指令记录以及代码生成元数据。
- EN: LLVM TableGen DSL records drive generated enums, matchers, encoders, or scheduling data.
  - CN: LLVM TableGen DSL 记录会驱动生成枚举、匹配器、编码器或调度数据。

## Dependencies / 依赖关系

- EN: TableGen includes: `(none)`.
  - CN: TableGen 包含项：`(none)`。
- EN: Generated backend tables depend on tblgen consuming these records and emitting C++/inc fragments.
  - CN: 生成后的后端表依赖 tblgen 消费这些记录并产出 C++/inc 片段。
