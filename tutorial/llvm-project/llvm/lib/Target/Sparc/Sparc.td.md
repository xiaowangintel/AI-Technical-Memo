# Sparc.td — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/Sparc/Sparc.td`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Defines backend metadata in TableGen DSL that is later expanded into generated tables and matcher code.
  - **CN**: 使用 TableGen DSL 定义后端元数据，随后会展开为生成的表和匹配代码。
- **Note / 说明**:
  - **EN**: This file uses LLVM TableGen DSL, so many definitions are declarative records rather than executable C++ statements.
  - **CN**: 该文件使用 LLVM TableGen DSL，因此许多内容是声明式记录，而不是可执行的 C++ 语句。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```tablegen
//===-- Sparc.td - Describe the Sparc Target Machine -------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 8-14
```tablegen
//
//
//===----------------------------------------------------------------------===//

//===----------------------------------------------------------------------===//
// Target-independent interfaces which we are implementing
//===----------------------------------------------------------------------===//
```
- **EN**: Documents the next group of declarations or explains a target-specific rule.
- **CN**: 为接下来的声明分组提供说明，或解释目标相关规则。

### Lines 15-22
```tablegen

include "llvm/Target/Target.td"
include "llvm/TableGen/SearchableTable.td"

//===----------------------------------------------------------------------===//
// SPARC Subtarget features.
//

```
- **EN**: Imports shared TableGen building blocks such as `llvm/Target/Target.td`, `llvm/TableGen/SearchableTable.td` so this file can reuse operand, format, or helper definitions.
- **CN**: 导入共享的 TableGen 构件，例如 `llvm/Target/Target.td`, `llvm/TableGen/SearchableTable.td`，以便复用操作数、格式或辅助定义。

### Lines 23-36
```tablegen
// Some words about SPARC ISA versions and feature levels:
// - V7: The first public version, used in some early Sun 4 models. Lacks integer
//       multiply and divide in hardware, and doesn't have a formal memory model,
//       though in practice the few multiprocessor machines that do exist
//       are sequentially consistent.
// - V8: Adds hardware multiply and divide, and formally defines TSO and PSO
//       memory models. Most 32-bit chips currently used are of this version.
// - V9: Adds 64-bit capability and RMO memory model, aside from new instructions.
//       Note that unlike most other architectures, instructions that operate
//       on 64-bit registers (e.g casx) are still available even in 32-bit ILP32
//       mode - the G/O registers are always 64-bit regardless of runtime bitness.
//       To let compilers take advantage of this feature, Sun defined an extension
//       to the 32-bit ABI called V8+. It's still possible, though, to compile for
//       V9 ISA level while not emitting any of the 64-bit instructions, that is
```
- **EN**: Documents the next group of declarations or explains a target-specific rule.
- **CN**: 为接下来的声明分组提供说明，或解释目标相关规则。

### Lines 37-50
```tablegen
//       why -mv8plus is defined separately from the usual -mcpu flags.
//       (OTOH, the combination of `-mcpu=v8 -mv8plus` is effectively a no-op)
//
// As for memory models, the ordering from weakest to strongest is as follows:
// (weaker) RMO -> PSO -> TSO -> SC (stronger)
// Those are defined in such a way that code compiled for a weaker model will
// run fine on processors that only implements a stronger model.
// For simplcity, LLVM targets the weakest model available in a particular
// ISA version: RMO for v9, and PSO for v7/v8.
//
// As for triples, Linux/BSD and Solaris follows different conventions:
// Linux/BSD name | Solaris name | Meaning
// ---------------|--------------|-------------
// sparc-*-*      | sparc-*-*    | 32-bit SPARC
```
- **EN**: Documents the next group of declarations or explains a target-specific rule.
- **CN**: 为接下来的声明分组提供说明，或解释目标相关规则。

### Lines 51-58
```tablegen
// sparc64-*-*    | sparcv9-*-*  | 64-bit SPARC
// Rather confusingly, Solaris calls the 64-bit triple "sparcv9".
// LLVM accepts both sparc64- and sparcv9- forms for 64-bit triples.

def FeatureSoftMulDiv
  : SubtargetFeature<"soft-mul-div", "UseSoftMulDiv", "true",
                     "Use software emulation for integer multiply and divide">;

```
- **EN**: Declares TableGen records such as `FeatureSoftMulDiv` for the backend description.
- **CN**: 为后端描述声明了 `FeatureSoftMulDiv` 等 TableGen 记录。

### Lines 59-65
```tablegen
def FeatureNoFSMULD
  : SubtargetFeature<"no-fsmuld", "HasNoFSMULD", "true",
                     "Disable the fsmuld instruction.">;
def FeatureNoFMULS
  : SubtargetFeature<"no-fmuls", "HasNoFMULS", "true",
                     "Disable the fmuls instruction.">;

```
- **EN**: Declares TableGen records such as `FeatureNoFSMULD`, `FeatureNoFMULS` for the backend description.
- **CN**: 为后端描述声明了 `FeatureNoFSMULD`, `FeatureNoFMULS` 等 TableGen 记录。

### Lines 66-79
```tablegen
def FeatureV9
  : SubtargetFeature<"v9", "IsV9", "true",
                     "Enable SPARC-V9 instructions">;
def Feature64Bit : SubtargetFeature<"64bit", "Is64Bit", "true",
                                    "Enable 64-bit mode", [FeatureV9]>;

def FeatureV8Plus
  : SubtargetFeature<"v8plus", "IsV8Plus", "true",
                     "Enable V8+ mode, allowing use of 64-bit V9 instructions in 32-bit code">;
def FeatureV8Deprecated
  : SubtargetFeature<"deprecated-v8", "UseV8DeprecatedInsts", "true",
                     "Enable deprecated V8 instructions in V9 mode">;
def FeatureVIS
  : SubtargetFeature<"vis", "IsVIS", "true",
```
- **EN**: Declares TableGen records such as `FeatureV9`, `Feature64Bit`, `FeatureV8Plus`, `FeatureV8Deprecated`, ... for the backend description.
- **CN**: 为后端描述声明了 `FeatureV9`, `Feature64Bit`, `FeatureV8Plus`, `FeatureV8Deprecated`, ... 等 TableGen 记录。

### Lines 80-93
```tablegen
                     "Enable UltraSPARC Visual Instruction Set extensions",
                     [FeatureV9]>;
def FeatureVIS2
  : SubtargetFeature<"vis2", "IsVIS2", "true",
                     "Enable Visual Instruction Set extensions II",
                     [FeatureV9]>;
def FeatureVIS3
  : SubtargetFeature<"vis3", "IsVIS3", "true",
                     "Enable Visual Instruction Set extensions III",
                     [FeatureV9]>;
def FeatureUA2005
  : SubtargetFeature<"ua2005", "IsUA2005", "true",
                     "Enable UltraSPARC Architecture 2005 extensions",
                     [FeatureV9, FeatureVIS, FeatureVIS2]>;
```
- **EN**: Declares TableGen records such as `FeatureVIS2`, `FeatureVIS3`, `FeatureUA2005`; this block attaches SelectionDAG patterns.
- **CN**: 声明了 `FeatureVIS2`, `FeatureVIS3`, `FeatureUA2005` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式。

### Lines 94-107
```tablegen
def FeatureUA2007
  : SubtargetFeature<"ua2007", "IsUA2007", "true",
                     "Enable UltraSPARC Architecture 2007 extensions",
                     [FeatureV9, FeatureVIS, FeatureVIS2]>;
def FeatureOSA2011
  : SubtargetFeature<"osa2011", "IsOSA2011", "true",
                     "Enable Oracle SPARC Architecture 2011 extensions",
                     [FeatureV9, FeatureVIS, FeatureVIS2, FeatureVIS3]>;
def FeatureCrypto
  : SubtargetFeature<"crypto", "IsCrypto", "true",
                     "Enable cryptographic extensions",
                     [FeatureOSA2011]>;
def FeatureLeon
  : SubtargetFeature<"leon", "IsLeon", "true",
```
- **EN**: Declares TableGen records such as `FeatureUA2007`, `FeatureOSA2011`, `FeatureCrypto`, `FeatureLeon` for the backend description.
- **CN**: 为后端描述声明了 `FeatureUA2007`, `FeatureOSA2011`, `FeatureCrypto`, `FeatureLeon` 等 TableGen 记录。

### Lines 108-116
```tablegen
                     "Enable LEON extensions">;
def FeaturePWRPSR
  : SubtargetFeature<"leonpwrpsr", "HasPWRPSR", "true",
                     "Enable the PWRPSR instruction">;

def FeatureHardQuad
  : SubtargetFeature<"hard-quad-float", "HasHardQuad", "true",
                     "Enable quad-word floating point instructions">;

```
- **EN**: Declares TableGen records such as `FeaturePWRPSR`, `FeatureHardQuad` for the backend description.
- **CN**: 为后端描述声明了 `FeaturePWRPSR`, `FeatureHardQuad` 等 TableGen 记录。

### Lines 117-123
```tablegen
def UsePopc : SubtargetFeature<"popc", "UsePopc", "true",
                               "Use the popc (population count) instruction">;

def FeatureSoftFloat : SubtargetFeature<"soft-float", "UseSoftFloat", "true",
                              "Use software emulation for floating point">;

//===----------------------------------------------------------------------===//
```
- **EN**: Declares TableGen records such as `UsePopc`, `FeatureSoftFloat` for the backend description.
- **CN**: 为后端描述声明了 `UsePopc`, `FeatureSoftFloat` 等 TableGen 记录。

### Lines 124-132
```tablegen
// SPARC Subtarget tuning features.
//

def TuneSlowRDPC : SubtargetFeature<"slow-rdpc", "HasSlowRDPC", "true",
                                    "rd %pc, %XX is slow", [FeatureV9]>;

def TuneNoPredictor : SubtargetFeature<"no-predictor", "HasNoPredictor", "true",
                                    "Processor has no branch predictor, branches stall execution", []>;

```
- **EN**: Declares TableGen records such as `TuneSlowRDPC`, `TuneNoPredictor` for the backend description.
- **CN**: 为后端描述声明了 `TuneSlowRDPC`, `TuneNoPredictor` 等 TableGen 记录。

### Lines 133-146
```tablegen
//==== Features added predmoninantly for LEON subtarget support
include "LeonFeatures.td"

//==== Register allocation tweaks needed by some low-level software
foreach i = 1 ... 7  in
    def FeatureReserveG#i : SubtargetFeature<"reserve-g"#i, "ReserveRegister["#i#" + SP::G0]", "true",
                                             "Reserve G"#i#", making it unavailable as a GPR">;
foreach i = 0 ... 5 in
    def FeatureReserveO#i : SubtargetFeature<"reserve-o"#i, "ReserveRegister["#i#" + SP::O0]", "true",
                                             "Reserve O"#i#", making it unavailable as a GPR">;
foreach i = 0 ... 7 in
    def FeatureReserveL#i : SubtargetFeature<"reserve-l"#i, "ReserveRegister["#i#" + SP::L0]", "true",
                                             "Reserve L"#i#", making it unavailable as a GPR">;
foreach i = 0 ... 5 in
```
- **EN**: Imports shared TableGen building blocks such as `LeonFeatures.td` so this file can reuse operand, format, or helper definitions.
- **CN**: 导入共享的 TableGen 构件，例如 `LeonFeatures.td`，以便复用操作数、格式或辅助定义。

### Lines 147-153
```tablegen
    def FeatureReserveI#i : SubtargetFeature<"reserve-i"#i, "ReserveRegister["#i#" + SP::I0]", "true",
                                             "Reserve I"#i#", making it unavailable as a GPR">;

//===----------------------------------------------------------------------===//
// Register File, Calling Conv, Instruction Descriptions
//===----------------------------------------------------------------------===//

```
- **EN**: Declares TableGen records such as `FeatureReserveI` for the backend description.
- **CN**: 为后端描述声明了 `FeatureReserveI` 等 TableGen 记录。

### Lines 154-160
```tablegen
include "SparcASITags.td"
include "SparcPrefetchTags.td"
include "SparcRegisterInfo.td"
include "SparcCallingConv.td"
include "SparcSchedule.td"
include "SparcInstrInfo.td"

```
- **EN**: Imports shared TableGen building blocks such as `SparcASITags.td`, `SparcPrefetchTags.td`, `SparcRegisterInfo.td`, `SparcCallingConv.td` so this file can reuse operand, format, or helper definitions.
- **CN**: 导入共享的 TableGen 构件，例如 `SparcASITags.td`, `SparcPrefetchTags.td`, `SparcRegisterInfo.td`, `SparcCallingConv.td`，以便复用操作数、格式或辅助定义。

### Lines 161-169
```tablegen
defm : RemapAllTargetPseudoPointerOperands<sparc_ptr_rc>;

def SparcInstrInfo : InstrInfo;

def SparcAsmParser : AsmParser {
  let ShouldEmitMatchRegisterAltName = true;
  let AllowDuplicateRegisterNames = true;
}

```
- **EN**: Declares TableGen records such as `SparcInstrInfo`, `SparcAsmParser`; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `SparcInstrInfo`, `SparcAsmParser` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

### Lines 170-176
```tablegen
def SparcAsmParserVariant : AsmParserVariant {
  let RegisterPrefix = "%";
}

//===----------------------------------------------------------------------===//
// SPARC processors supported.
//===----------------------------------------------------------------------===//
```
- **EN**: Declares TableGen records such as `SparcAsmParserVariant`; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `SparcAsmParserVariant` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

### Lines 177-190
```tablegen

class Proc<string Name, list<SubtargetFeature> Features,
           list<SubtargetFeature> TuneFeatures = []>
 : Processor<Name, NoItineraries, Features, TuneFeatures>;

def : Proc<"generic",         []>;
def : Proc<"v7",              [FeatureSoftMulDiv, FeatureNoFSMULD]>;
def : Proc<"v8",              []>;
def : Proc<"supersparc",      []>;
def : Proc<"sparclite",       []>;
def : Proc<"f934",            []>;
def : Proc<"hypersparc",      []>;
def : Proc<"sparclite86x",    []>;
def : Proc<"sparclet",        []>;
```
- **EN**: Declares TableGen records such as `Proc` for the backend description.
- **CN**: 为后端描述声明了 `Proc` 等 TableGen 记录。

### Lines 191-204
```tablegen
def : Proc<"tsc701",          []>;
def : Proc<"myriad2",         [FeatureLeon, LeonCASA]>;
def : Proc<"myriad2.1",       [FeatureLeon, LeonCASA]>;
def : Proc<"myriad2.2",       [FeatureLeon, LeonCASA]>;
def : Proc<"myriad2.3",       [FeatureLeon, LeonCASA]>;
def : Proc<"ma2100",          [FeatureLeon, LeonCASA]>;
def : Proc<"ma2150",          [FeatureLeon, LeonCASA]>;
def : Proc<"ma2155",          [FeatureLeon, LeonCASA]>;
def : Proc<"ma2450",          [FeatureLeon, LeonCASA]>;
def : Proc<"ma2455",          [FeatureLeon, LeonCASA]>;
def : Proc<"ma2x5x",          [FeatureLeon, LeonCASA]>;
def : Proc<"ma2080",          [FeatureLeon, LeonCASA]>;
def : Proc<"ma2085",          [FeatureLeon, LeonCASA]>;
def : Proc<"ma2480",          [FeatureLeon, LeonCASA]>;
```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 205-218
```tablegen
def : Proc<"ma2485",          [FeatureLeon, LeonCASA]>;
def : Proc<"ma2x8x",          [FeatureLeon, LeonCASA]>;
def : Proc<"v9",              [FeatureV9]>;
def : Proc<"ultrasparc",      [FeatureV9, FeatureV8Deprecated, FeatureVIS],
                              [TuneSlowRDPC]>;
def : Proc<"ultrasparc3",     [FeatureV9, FeatureV8Deprecated, FeatureVIS,
                               FeatureVIS2],
                              [TuneSlowRDPC]>;
def : Proc<"niagara",         [FeatureV9, FeatureV8Deprecated, FeatureVIS,
                               FeatureVIS2, FeatureUA2005],
                              [TuneNoPredictor]>;
def : Proc<"niagara2",        [FeatureV9, FeatureV8Deprecated, UsePopc,
                               FeatureVIS, FeatureVIS2, FeatureUA2005],
                              [TuneNoPredictor]>;
```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 219-227
```tablegen
def : Proc<"niagara3",        [FeatureV9, FeatureV8Deprecated, UsePopc,
                               FeatureVIS, FeatureVIS2, FeatureVIS3,
                               FeatureUA2005, FeatureUA2007],
                              [TuneNoPredictor]>;
def : Proc<"niagara4",        [FeatureV9, FeatureV8Deprecated, UsePopc,
                               FeatureVIS, FeatureVIS2, FeatureVIS3,
                               FeatureUA2005, FeatureUA2007, FeatureOSA2011,
                               FeatureCrypto]>;

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 228-236
```tablegen
// LEON 2 FT generic
def : Processor<"leon2", LEON2Itineraries,
                [FeatureLeon]>;

// LEON 2 FT (AT697E)
// TO DO: Place-holder: Processor specific features will be added *very* soon here.
def : Processor<"at697e", LEON2Itineraries,
                [FeatureLeon, InsertNOPLoad]>;

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 237-246
```tablegen
// LEON 2 FT (AT697F)
// TO DO: Place-holder: Processor specific features will be added *very* soon here.
def : Processor<"at697f", LEON2Itineraries,
                [FeatureLeon, InsertNOPLoad]>;


// LEON 3 FT generic
def : Processor<"leon3", LEON3Itineraries,
                [FeatureLeon, UMACSMACSupport]>;

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 247-256
```tablegen
// LEON 3 FT (UT699). Provides features for the UT699 processor
// - covers all the erratum fixes for LEON3, but does not support the CASA instruction.
def : Processor<"ut699", LEON3Itineraries,
                [FeatureLeon, InsertNOPLoad, FeatureNoFSMULD, FeatureNoFMULS, FixAllFDIVSQRT]>;

// LEON3 FT (GR712RC). Provides features for the GR712RC processor.
// - covers all the erratum fixed for LEON3 and support for the CASA instruction.
def : Processor<"gr712rc", LEON3Itineraries,
                [FeatureLeon, LeonCASA]>;

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 257-266
```tablegen
// LEON 4 FT generic
def : Processor<"leon4", LEON4Itineraries,
                [FeatureLeon, UMACSMACSupport, LeonCASA]>;

// LEON 4 FT (GR740)
// TO DO: Place-holder: Processor specific features will be added *very* soon here.
def : Processor<"gr740", LEON4Itineraries,
                [FeatureLeon, UMACSMACSupport, LeonCASA, LeonCycleCounter,
                 FeaturePWRPSR]>;

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 267-276
```tablegen
//===----------------------------------------------------------------------===//
// Declare the target which we are implementing
//===----------------------------------------------------------------------===//

def SparcAsmWriter : AsmWriter {
  string AsmWriterClassName  = "InstPrinter";
  int PassSubtarget = 1;
  int Variant = 0;
}

```
- **EN**: Declares TableGen records such as `SparcAsmWriter` for the backend description.
- **CN**: 为后端描述声明了 `SparcAsmWriter` 等 TableGen 记录。

### Lines 277-284
```tablegen
def Sparc : Target {
  // Pull in Instruction Info:
  let InstructionSet = SparcInstrInfo;
  let AssemblyParsers  = [SparcAsmParser];
  let AssemblyParserVariants = [SparcAsmParserVariant];
  let AssemblyWriters = [SparcAsmWriter];
  let AllowRegisterRenaming = 1;
}
```
- **EN**: Declares TableGen records such as `Sparc`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments.
- **CN**: 声明了 `Sparc` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段。

## Key Concepts / 关键概念

- **TableGen DSL / TableGen 领域专用语言**:
  - **EN**: Declarative backend description expanded into generated tables
  - **CN**: 以声明式方式描述后端并生成表格代码
- **Generated backend tables / 生成的后端表**:
  - **EN**: The records here feed TableGen generators that emit `.inc` matcher/data tables
  - **CN**: 此处记录会喂给 TableGen 生成器，产出 `.inc` 匹配器/数据表

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `llvm/Target/Target.td`, `llvm/TableGen/SearchableTable.td`, `LeonFeatures.td`, `SparcASITags.td`, `SparcPrefetchTags.td`, `SparcRegisterInfo.td`, `SparcCallingConv.td`, `SparcSchedule.td`, `SparcInstrInfo.td`
- **LLVM subsystems / LLVM 子系统**: TableGen generators
