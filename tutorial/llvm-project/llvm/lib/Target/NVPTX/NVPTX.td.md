# NVPTX.td — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/NVPTX/NVPTX.td`
- Repository: `llvm-project`
- Purpose (EN): This is the top level entry point for the NVPTX target.
- 目的（中文）: 该文件使用 TableGen DSL 描述目标后端元数据，并驱动自动生成代码。
- Language: TableGen DSL (TableGen DSL noted below)
- Note: This file is written in TableGen DSL, so many records are declarative descriptions consumed by LLVM code generators.
- 说明：该文件使用 TableGen DSL 编写，因此许多记录都是供 LLVM 代码生成器消费的声明式描述。

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-60
```tablegen
 1: //===- NVPTX.td - Describe the NVPTX Target Machine -----------*- tblgen -*-==//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: // This is the top level entry point for the NVPTX target.
 9: //===----------------------------------------------------------------------===//
10:
11: //===----------------------------------------------------------------------===//
12: // Target-independent interfaces
13: //===----------------------------------------------------------------------===//
14:
15: include "llvm/Target/Target.td"
16:
17: include "NVPTXRegisterInfo.td"
18: include "NVPTXInstrInfo.td"
19:
20: //===----------------------------------------------------------------------===//
21: // Subtarget Features.
22: // - We use the SM version number instead of explicit feature table.
23: // - Need at least one feature to avoid generating zero sized array by
24: //   TableGen in NVPTXGenSubtarget.inc.
25: //===----------------------------------------------------------------------===//
26:
27: class FeatureSM<string sm, int value>:
28:    SubtargetFeature<"sm_"# sm, "FullSmVersion",
29:                     "" # value,
30:                     "Target SM " # sm>;
31:
32: class FeaturePTX<int version>:
33:    SubtargetFeature<"ptx"# version, "PTXVersion",
34:                     "" # version,
35:                     "Use PTX version " # version>;
36: // NVPTX Architecture Hierarchy and Ordering:
37: //
38: // GPU architectures: sm_2Y/sm_3Y/sm_5Y/sm_6Y/sm_7Y/sm_8Y/sm_9Y/sm_10Y/sm_12Y
39: // ('Y' represents version within the architecture)
40: // The architectures have name of form sm_XYz where 'X' represent the generation
41: // number, 'Y' represents the version within the architecture, and 'z' represents
42: // the optional feature suffix.
43: // If X1Y1 <= X2Y2, then GPU capabilities of sm_X1Y1 are included in sm_X2Y2.
44: // For example, take sm_90 (9 represents 'X', 0 represents 'Y', and no feature
45: // suffix) and sm_103 architectures (10 represents 'X', 3 represents 'Y', and no
46: // feature suffix). Since 90 <= 103, sm_90 is compatible with sm_103.
47: //
48: // The family-specific variants have 'f' feature suffix and they follow
49: // following order:
50: // sm_X{Y2}f > sm_X{Y1}f iff Y2 > Y1
51: // sm_XY{f} > sm_{XY}{}
52: //
53: // For example, take sm_100f (10 represents 'X', 0 represents 'Y', and 'f'
54: // represents 'z') and sm_103f (10 represents 'X', 3 represents 'Y', and 'f'
55: // represents 'z') architecture variants. Since Y1 < Y2, sm_100f is compatible with
56: // sm_103f. Similarly based on the second rule, sm_90 is compatible with sm_103f.
57: //
58: // Some counter examples, take sm_100f and sm_120f (12 represents 'X', 0
59: // represents 'Y', and 'f' represents 'z') architecture variants. Since both
60: // belongs to different family i.e. X1 != X2, sm_100f is not compatible with
```
- EN: This range uses TableGen DSL to describe records such as FeatureSM, FeaturePTX; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 FeatureSM、FeaturePTX 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 61-120
```tablegen
 61: // sm_120f.
 62: //
 63: // The architecture-specific variants have 'a' feature suffix and they follow
 64: // following order:
 65: // sm_XY{a} > sm_XY{f} > sm_{XY}{}
 66: //
 67: // For example, take sm_103a (10 represents 'X', 3 represents 'Y', and 'a'
 68: // represents 'z'), sm_103f, and sm_103 architecture variants. The sm_103 is
 69: // compatible with sm_103a and sm_103f, and sm_103f is compatible with sm_103a.
 70: //
 71: // Encoding := Arch * 10 + ArchSuffixOffset
 72: // Arch := X * 10 + Y
 73: // ArchSuffixOffset := 0 (base), 2 ('f'), or 3 ('a')
 74: //
 75: // For example, sm_103a is encoded as 1033 (103 * 10 + 3) and sm_103f is
 76: // encoded as 1032 (103 * 10 + 2).
 77: //
 78: // This encoding allows simple partial ordering of the architectures.
 79: //  + Compare Family and Arch by dividing FullSMVersion by 100 and 10
 80: //    respectively before the comparison.
 81: //  + Compare within the family by comparing FullSMVersion, given both belongs to
 82: //    the same family.
 83: //  + Detect 'a' variants by checking FullSMVersion & 1.
 84: class Proc<FeatureSM SM>
 85:  : Processor<SM.Name, NoItineraries, [SM]>;
 86:
 87: foreach sm = [20, 21, 30, 32, 35, 37, 50, 52, 53, 60,
 88:               61, 62, 70, 72, 75, 80, 86, 87, 88, 89,
 89:               90, 100, 101, 103, 110, 120, 121] in {
 90:   // Base SM version (e.g. FullSMVersion for sm_100 is 1000)
 91:   def SM#sm : FeatureSM<""#sm, !mul(sm, 10)>;
 92:   def : Proc<!cast<FeatureSM>("SM"#sm)>;
 93:
 94:   // Family-specific variants, compatible within same family (e.g. sm_100f = 1002)
 95:   if !ge(sm, 100) then {
 96:     def SM#sm#f : FeatureSM<""#sm#"f", !add(!mul(sm, 10), 2)>;
 97:     def : Proc<!cast<FeatureSM>("SM"#sm#"f")>;
 98:   }
 99:
100:   // Architecture-specific variants, incompatible across architectures (e.g. sm_100a = 1003)
101:   if !ge(sm, 90) then {
102:     def SM#sm#a : FeatureSM<""#sm#"a", !add(!mul(sm, 10), 3)>;
103:     def : Proc<!cast<FeatureSM>("SM"#sm#"a")>;
104:   }
105: }
106:
107: foreach version = [32, 40, 41, 42, 43, 50, 60, 61, 62, 63, 64, 65, 70, 71, 72,
108:                    73, 74, 75, 76, 77, 78, 80, 81, 82, 83, 84, 85, 86, 87, 88,
109:                    90, 91, 92] in
110:   def PTX#version : FeaturePTX<version>;
111:
112: def Is64Bit : Predicate<"Subtarget->getTargetTriple().getArch() == Triple::nvptx64">;
113: def NVPTX64 : HwMode<[Is64Bit]>;
114:
115: def nvptx_ptr_rc : RegClassByHwMode<
116:   [DefaultMode, NVPTX64],
117:   [B32, B64]>;
118:
119: defm : RemapAllTargetPseudoPointerOperands<nvptx_ptr_rc>;
120:
```
- EN: This range uses TableGen DSL to describe records such as Proc, SM#sm, SM#sm#f, SM#sm#a; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 Proc、SM#sm、SM#sm#f、SM#sm#a 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 121-131
```tablegen
121: def NVPTXInstrInfo : InstrInfo {
122: }
123:
124: def NVPTXAsmWriter : AsmWriter {
125:   int PassSubtarget = 1;
126: }
127:
128: def NVPTX : Target {
129:   let InstructionSet = NVPTXInstrInfo;
130:   let AssemblyWriters = [NVPTXAsmWriter];
131: }
```
- EN: This range uses TableGen DSL to describe records such as NVPTXInstrInfo, NVPTXAsmWriter, NVPTX; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 NVPTXInstrInfo、NVPTXAsmWriter、NVPTX 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

## Key Concepts / 关键概念

- EN: TableGen files use declarative records that LLVM expands into generated C++ tables and helper code.
  - CN: TableGen 文件使用声明式记录，LLVM 会将其展开为生成的 C++ 表和辅助代码。
- EN: NVPTX backend code reflects GPU execution concerns such as address spaces, intrinsics, and PTX-specific lowering.
  - CN: NVPTX 后端代码体现了 GPU 执行特性，例如地址空间、内建函数以及 PTX 特定降级。
- EN: Key symbols in this file include FeatureSM, FeaturePTX, Proc, SM#sm, SM#sm#f, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 FeatureSM, FeaturePTX, Proc, SM#sm, SM#sm#f，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `NVPTXRegisterInfo.td`
  - `NVPTXInstrInfo.td`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/Target/Target.td`
- TableGen pipeline / TableGen 流水线: records in this file are consumed by LLVM TableGen emitters to produce generated lookup tables and helper code.
- TableGen 流水线说明：该文件中的记录会被 LLVM 的 TableGen 生成器消费，进而产出查找表和辅助代码。
