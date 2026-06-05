# ARMProcessors.td — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/ARMProcessors.td`
- Repository: `llvm-project`
- Purpose (EN): Defines `ARMProcessors` in LLVM TableGen DSL for the ARM backend, covering target descriptions, instruction records, and code-generation metadata.
- 用途 (CN): 使用 LLVM TableGen DSL 定义 ARM 后端中的 `ARMProcessors`，涵盖目标描述、指令记录以及代码生成元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```tablegen
class ProcNoItin<string Name, list<SubtargetFeature> Features>
  : Processor<Name, NoItineraries, Features>;
```
- EN: Declares reusable TableGen class `ProcNoItin` for `ARMProcessors`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMProcessors` 声明可复用的 TableGen 类 `ProcNoItin`，通常用于抽象共享字段、谓词或编码结构。

### Lines 4-6
```tablegen
//===----------------------------------------------------------------------===//
// ARM Processor subtarget features.
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 8-25
```tablegen
def ProcA5      : SubtargetFeature<"a5", "ARMProcFamily", "CortexA5",
                                   "Cortex-A5 ARM processors", []>;
def ProcA7      : SubtargetFeature<"a7", "ARMProcFamily", "CortexA7",
                                   "Cortex-A7 ARM processors", []>;
def ProcA8      : SubtargetFeature<"a8", "ARMProcFamily", "CortexA8",
                                   "Cortex-A8 ARM processors", []>;
def ProcA9      : SubtargetFeature<"a9", "ARMProcFamily", "CortexA9",
                                   "Cortex-A9 ARM processors", []>;
def ProcA12     : SubtargetFeature<"a12", "ARMProcFamily", "CortexA12",
                                   "Cortex-A12 ARM processors", []>;
def ProcA15     : SubtargetFeature<"a15", "ARMProcFamily", "CortexA15",
                                   "Cortex-A15 ARM processors", []>;
def ProcA17     : SubtargetFeature<"a17", "ARMProcFamily", "CortexA17",
                                   "Cortex-A17 ARM processors", []>;
def ProcA32     : SubtargetFeature<"a32", "ARMProcFamily", "CortexA32",
                                   "Cortex-A32 ARM processors", []>;
def ProcA35     : SubtargetFeature<"a35", "ARMProcFamily", "CortexA35",
                                   "Cortex-A35 ARM processors", []>;
```
- EN: Defines TableGen record `ProcA5` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ProcA5`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 26-43
```tablegen
def ProcA53     : SubtargetFeature<"a53", "ARMProcFamily", "CortexA53",
                                   "Cortex-A53 ARM processors", []>;
def ProcA55     : SubtargetFeature<"a55", "ARMProcFamily", "CortexA55",
                                   "Cortex-A55 ARM processors", []>;
def ProcA57     : SubtargetFeature<"a57", "ARMProcFamily", "CortexA57",
                                   "Cortex-A57 ARM processors", []>;
def ProcA72     : SubtargetFeature<"a72", "ARMProcFamily", "CortexA72",
                                   "Cortex-A72 ARM processors", []>;
def ProcA73     : SubtargetFeature<"a73", "ARMProcFamily", "CortexA73",
                                   "Cortex-A73 ARM processors", []>;
def ProcA75     : SubtargetFeature<"a75", "ARMProcFamily", "CortexA75",
                                   "Cortex-A75 ARM processors", []>;
def ProcA76     : SubtargetFeature<"a76", "ARMProcFamily", "CortexA76",
                                   "Cortex-A76 ARM processors", []>;
def ProcA77     : SubtargetFeature<"a77", "ARMProcFamily", "CortexA77",
                                   "Cortex-A77 ARM processors", []>;
def ProcA78     : SubtargetFeature<"cortex-a78", "ARMProcFamily", "CortexA78",
                                   "Cortex-A78 ARM processors", []>;
```
- EN: Defines TableGen record `ProcA53` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ProcA53`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 44-55
```tablegen
def ProcA78AE   : SubtargetFeature<"cortex-a78ae", "ARMProcFamily", "CortexA78AE",
                                   "Cortex-A78AE ARM processors", []>;
def ProcA78C    : SubtargetFeature<"a78c", "ARMProcFamily", "CortexA78C",
                                   "Cortex-A78C ARM processors", []>;
def ProcA510    : SubtargetFeature<"cortex-a510", "ARMProcFamily",
                                   "CortexA510", "Cortex-A510 ARM processors", []>;
def ProcA710    : SubtargetFeature<"cortex-a710", "ARMProcFamily",
                                   "CortexA710", "Cortex-A710 ARM processors", []>;
def ProcX1      : SubtargetFeature<"cortex-x1", "ARMProcFamily", "CortexX1",
                                   "Cortex-X1 ARM processors", []>;
def ProcX1C     : SubtargetFeature<"cortex-x1c", "ARMProcFamily", "CortexX1C",
                                   "Cortex-X1C ARM processors", []>;
```
- EN: Defines TableGen record `ProcA78AE` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ProcA78AE`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 57-58
```tablegen
def ProcV1      : SubtargetFeature<"neoverse-v1", "ARMProcFamily",
                                   "NeoverseV1", "Neoverse-V1 ARM processors", []>;
```
- EN: Defines TableGen record `ProcV1` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ProcV1`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 60-65
```tablegen
def ProcKrait   : SubtargetFeature<"krait", "ARMProcFamily", "Krait",
                                   "Qualcomm Krait processors", []>;
def ProcKryo    : SubtargetFeature<"kryo", "ARMProcFamily", "Kryo",
                                   "Qualcomm Kryo processors", []>;
def ProcSwift   : SubtargetFeature<"swift", "ARMProcFamily", "Swift",
                                   "Swift ARM processors", []>;
```
- EN: Defines TableGen record `ProcKrait` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ProcKrait`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 67-84
```tablegen
def ProcExynos  : SubtargetFeature<"exynos", "ARMProcFamily", "Exynos",
                                   "Samsung Exynos processors",
                                   [FeatureZCZeroing,
                                    FeatureUseWideStrideVFP,
                                    FeatureSplatVFPToNeon,
                                    FeatureSlowVGETLNi32,
                                    FeatureSlowVDUP32,
                                    FeatureSlowFPBrcc,
                                    FeatureProfUnpredicate,
                                    FeatureHWDivThumb,
                                    FeatureHWDivARM,
                                    FeatureHasSlowFPVMLx,
                                    FeatureHasSlowFPVFMx,
                                    FeatureHasRetAddrStack,
                                    FeatureFuseLiterals,
                                    FeatureFuseAES,
                                    FeatureExpandMLx,
                                    FeatureCrypto,
```
- EN: Defines TableGen record `ProcExynos` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ProcExynos`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 85-85
```tablegen
                                    FeatureCRC]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 87-96
```tablegen
def ProcR4      : SubtargetFeature<"r4", "ARMProcFamily", "CortexR4",
                                   "Cortex-R4 ARM processors", []>;
def ProcR5      : SubtargetFeature<"r5", "ARMProcFamily", "CortexR5",
                                   "Cortex-R5 ARM processors", []>;
def ProcR7      : SubtargetFeature<"r7", "ARMProcFamily", "CortexR7",
                                   "Cortex-R7 ARM processors", []>;
def ProcR52     : SubtargetFeature<"r52", "ARMProcFamily", "CortexR52",
                                   "Cortex-R52 ARM processors", []>;
def ProcR52plus  : SubtargetFeature<"r52plus", "ARMProcFamily", "CortexR52plus",
                                   "Cortex-R52plus ARM processors", []>;
```
- EN: Defines TableGen record `ProcR4` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ProcR4`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 98-105
```tablegen
def ProcM3      : SubtargetFeature<"m3", "ARMProcFamily", "CortexM3",
                                   "Cortex-M3 ARM processors", []>;
def ProcM55      : SubtargetFeature<"m55", "ARMProcFamily", "CortexM55",
                                   "Cortex-M55 ARM processors", []>;
def ProcM7      : SubtargetFeature<"m7", "ARMProcFamily", "CortexM7",
                                   "Cortex-M7 ARM processors", []>;
def ProcM85      : SubtargetFeature<"m85", "ARMProcFamily", "CortexM85",
                                   "Cortex-M85 ARM processors", []>;
```
- EN: Defines TableGen record `ProcM3` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ProcM3`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 107-111
```tablegen
//===----------------------------------------------------------------------===//
// ARM processors
//
// Dummy CPU, used to target architectures
def : ProcessorModel<"generic",     CortexA8Model,      []>;
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 113-114
```tablegen
// FIXME: Several processors below are not using their own scheduler
// model, but one of similar/previous processor. These should be fixed.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 116-121
```tablegen
def : ProcNoItin<"arm8",                                [ARMv4]>;
def : ProcNoItin<"arm810",                              [ARMv4]>;
def : ProcNoItin<"strongarm",                           [ARMv4]>;
def : ProcNoItin<"strongarm110",                        [ARMv4]>;
def : ProcNoItin<"strongarm1100",                       [ARMv4]>;
def : ProcNoItin<"strongarm1110",                       [ARMv4]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 123-133
```tablegen
def : ProcNoItin<"arm7tdmi",                            [ARMv4t]>;
def : ProcNoItin<"arm7tdmi-s",                          [ARMv4t]>;
def : ProcNoItin<"arm710t",                             [ARMv4t]>;
def : ProcNoItin<"arm720t",                             [ARMv4t]>;
def : ProcNoItin<"arm9",                                [ARMv4t]>;
def : ProcNoItin<"arm9tdmi",                            [ARMv4t]>;
def : ProcNoItin<"arm920",                              [ARMv4t]>;
def : ProcNoItin<"arm920t",                             [ARMv4t]>;
def : ProcNoItin<"arm922t",                             [ARMv4t]>;
def : ProcNoItin<"arm940t",                             [ARMv4t]>;
def : ProcNoItin<"ep9312",                              [ARMv4t]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 135-136
```tablegen
def : ProcNoItin<"arm10tdmi",                           [ARMv5t]>;
def : ProcNoItin<"arm1020t",                            [ARMv5t]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 138-147
```tablegen
def : ProcNoItin<"arm9e",                               [ARMv5te]>;
def : ProcNoItin<"arm926ej-s",                          [ARMv5te]>;
def : ProcNoItin<"arm946e-s",                           [ARMv5te]>;
def : ProcNoItin<"arm966e-s",                           [ARMv5te]>;
def : ProcNoItin<"arm968e-s",                           [ARMv5te]>;
def : ProcNoItin<"arm10e",                              [ARMv5te]>;
def : ProcNoItin<"arm1020e",                            [ARMv5te]>;
def : ProcNoItin<"arm1022e",                            [ARMv5te]>;
def : ProcNoItin<"xscale",                              [ARMv5te]>;
def : ProcNoItin<"iwmmxt",                              [ARMv5te]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 149-152
```tablegen
def : Processor<"arm1136j-s",       ARMV6Itineraries,   [ARMv6]>;
def : Processor<"arm1136jf-s",      ARMV6Itineraries,   [ARMv6,
                                                         FeatureVFP2,
                                                         FeatureHasSlowFPVMLx]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 154-161
```tablegen
def : Processor<"cortex-m0",        ARMV6Itineraries,   [ARMv6m,
                                                         FeatureHasNoBranchPredictor]>;
def : Processor<"cortex-m0plus",    ARMV6Itineraries,   [ARMv6m,
                                                         FeatureHasNoBranchPredictor]>;
def : Processor<"cortex-m1",        ARMV6Itineraries,   [ARMv6m,
                                                         FeatureHasNoBranchPredictor]>;
def : Processor<"sc000",            ARMV6Itineraries,   [ARMv6m,
                                                         FeatureHasNoBranchPredictor]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 163-166
```tablegen
def : Processor<"arm1176jz-s",      ARMV6Itineraries,   [ARMv6kz]>;
def : Processor<"arm1176jzf-s",     ARMV6Itineraries,   [ARMv6kz,
                                                         FeatureVFP2,
                                                         FeatureHasSlowFPVMLx]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 168-171
```tablegen
def : Processor<"mpcorenovfp",      ARMV6Itineraries,   [ARMv6k]>;
def : Processor<"mpcore",           ARMV6Itineraries,   [ARMv6k,
                                                         FeatureVFP2,
                                                         FeatureHasSlowFPVMLx]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 173-176
```tablegen
def : Processor<"arm1156t2-s",      ARMV6Itineraries,   [ARMv6t2]>;
def : Processor<"arm1156t2f-s",     ARMV6Itineraries,   [ARMv6t2,
                                                         FeatureVFP2,
                                                         FeatureHasSlowFPVMLx]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 178-186
```tablegen
def : ProcessorModel<"cortex-a5",   CortexA8Model,      [ARMv7a, ProcA5,
                                                         FeatureHasRetAddrStack,
                                                         FeatureTrustZone,
                                                         FeatureSlowFPBrcc,
                                                         FeatureHasSlowFPVMLx,
                                                         FeatureHasSlowFPVFMx,
                                                         FeatureVMLxForwarding,
                                                         FeatureMP,
                                                         FeatureVFP4]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 188-198
```tablegen
def : ProcessorModel<"cortex-a7",   CortexA8Model,      [ARMv7a, ProcA7,
                                                         FeatureHasRetAddrStack,
                                                         FeatureTrustZone,
                                                         FeatureSlowFPBrcc,
                                                         FeatureHasVMLxHazards,
                                                         FeatureHasSlowFPVMLx,
                                                         FeatureHasSlowFPVFMx,
                                                         FeatureVMLxForwarding,
                                                         FeatureMP,
                                                         FeatureVFP4,
                                                         FeatureVirtualization]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 200-208
```tablegen
def : ProcessorModel<"cortex-a8",   CortexA8Model,      [ARMv7a, ProcA8,
                                                         FeatureHasRetAddrStack,
                                                         FeatureNonpipelinedVFP,
                                                         FeatureTrustZone,
                                                         FeatureSlowFPBrcc,
                                                         FeatureHasVMLxHazards,
                                                         FeatureHasSlowFPVMLx,
                                                         FeatureHasSlowFPVFMx,
                                                         FeatureVMLxForwarding]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 210-222
```tablegen
def : ProcessorModel<"cortex-a9",   CortexA9Model,      [ARMv7a, ProcA9,
                                                         FeatureHasRetAddrStack,
                                                         FeatureTrustZone,
                                                         FeatureHasVMLxHazards,
                                                         FeatureVMLxForwarding,
                                                         FeatureFP16,
                                                         FeatureAvoidPartialCPSR,
                                                         FeatureExpandMLx,
                                                         FeaturePreferVMOVSR,
                                                         FeatureMuxedUnits,
                                                         FeatureNEONForFPMovs,
                                                         FeatureCheckVLDnAlign,
                                                         FeatureMP]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 224-231
```tablegen
def : ProcessorModel<"cortex-a12",  CortexA9Model,      [ARMv7a, ProcA12,
                                                         FeatureHasRetAddrStack,
                                                         FeatureTrustZone,
                                                         FeatureVMLxForwarding,
                                                         FeatureVFP4,
                                                         FeatureAvoidPartialCPSR,
                                                         FeatureVirtualization,
                                                         FeatureMP]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 233-243
```tablegen
def : ProcessorModel<"cortex-a15",  CortexA9Model,      [ARMv7a, ProcA15,
                                                         FeatureDontWidenVMOVS,
                                                         FeatureSplatVFPToNeon,
                                                         FeatureHasRetAddrStack,
                                                         FeatureMuxedUnits,
                                                         FeatureTrustZone,
                                                         FeatureVFP4,
                                                         FeatureMP,
                                                         FeatureCheckVLDnAlign,
                                                         FeatureAvoidPartialCPSR,
                                                         FeatureVirtualization]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 245-252
```tablegen
def : ProcessorModel<"cortex-a17",  CortexA9Model,      [ARMv7a, ProcA17,
                                                         FeatureHasRetAddrStack,
                                                         FeatureTrustZone,
                                                         FeatureMP,
                                                         FeatureVMLxForwarding,
                                                         FeatureVFP4,
                                                         FeatureAvoidPartialCPSR,
                                                         FeatureVirtualization]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 254-264
```tablegen
// FIXME: krait has currently the same features as A9 plus VFP4 and  HWDiv
def : ProcessorModel<"krait",       CortexA9Model,      [ARMv7a, ProcKrait,
                                                         FeatureHasRetAddrStack,
                                                         FeatureMuxedUnits,
                                                         FeatureCheckVLDnAlign,
                                                         FeatureVMLxForwarding,
                                                         FeatureFP16,
                                                         FeatureAvoidPartialCPSR,
                                                         FeatureVFP4,
                                                         FeatureHWDivThumb,
                                                         FeatureHWDivARM]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 266-283
```tablegen
def : ProcessorModel<"swift",       SwiftModel,         [ARMv7a, ProcSwift,
                                                         FeatureHasRetAddrStack,
                                                         FeatureNEONForFP,
                                                         FeatureVFP4,
                                                         FeatureUseWideStrideVFP,
                                                         FeatureMP,
                                                         FeatureHWDivThumb,
                                                         FeatureHWDivARM,
                                                         FeatureAvoidPartialCPSR,
                                                         FeatureAvoidMOVsShOp,
                                                         FeatureHasSlowFPVMLx,
                                                         FeatureHasSlowFPVFMx,
                                                         FeatureHasVMLxHazards,
                                                         FeatureProfUnpredicate,
                                                         FeaturePrefISHSTBarrier,
                                                         FeatureSlowOddRegister,
                                                         FeatureSlowLoadDSubreg,
                                                         FeatureSlowVGETLNi32,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 284-286
```tablegen
                                                         FeatureSlowVDUP32,
                                                         FeatureUseMISched,
                                                         FeatureNoPostRASched]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 288-290
```tablegen
def : ProcessorModel<"cortex-r4",   CortexA8Model,      [ARMv7r, ProcR4,
                                                         FeatureHasRetAddrStack,
                                                         FeatureAvoidPartialCPSR]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 292-298
```tablegen
def : ProcessorModel<"cortex-r4f",  CortexA8Model,      [ARMv7r, ProcR4,
                                                         FeatureHasRetAddrStack,
                                                         FeatureSlowFPBrcc,
                                                         FeatureHasSlowFPVMLx,
                                                         FeatureHasSlowFPVFMx,
                                                         FeatureVFP3_D16,
                                                         FeatureAvoidPartialCPSR]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 300-307
```tablegen
def : ProcessorModel<"cortex-r5",   CortexA8Model,      [ARMv7r, ProcR5,
                                                         FeatureHasRetAddrStack,
                                                         FeatureVFP3_D16,
                                                         FeatureSlowFPBrcc,
                                                         FeatureHWDivARM,
                                                         FeatureHasSlowFPVMLx,
                                                         FeatureHasSlowFPVFMx,
                                                         FeatureAvoidPartialCPSR]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 309-318
```tablegen
def : ProcessorModel<"cortex-r7",   CortexA8Model,      [ARMv7r, ProcR7,
                                                         FeatureHasRetAddrStack,
                                                         FeatureVFP3_D16,
                                                         FeatureFP16,
                                                         FeatureMP,
                                                         FeatureSlowFPBrcc,
                                                         FeatureHWDivARM,
                                                         FeatureHasSlowFPVMLx,
                                                         FeatureHasSlowFPVFMx,
                                                         FeatureAvoidPartialCPSR]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 320-329
```tablegen
def : ProcessorModel<"cortex-r8",   CortexA8Model,      [ARMv7r,
                                                         FeatureHasRetAddrStack,
                                                         FeatureVFP3_D16,
                                                         FeatureFP16,
                                                         FeatureMP,
                                                         FeatureSlowFPBrcc,
                                                         FeatureHWDivARM,
                                                         FeatureHasSlowFPVMLx,
                                                         FeatureHasSlowFPVFMx,
                                                         FeatureAvoidPartialCPSR]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 331-335
```tablegen
def : ProcessorModel<"cortex-m3",   CortexM4Model,      [ARMv7m,
                                                         ProcM3,
                                                         FeaturePreferBranchAlign32,
                                                         FeatureUseMISched,
                                                         FeatureHasNoBranchPredictor]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 337-340
```tablegen
def : ProcessorModel<"sc300",       CortexM4Model,      [ARMv7m,
                                                         ProcM3,
                                                         FeatureUseMISched,
                                                         FeatureHasNoBranchPredictor]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 342-348
```tablegen
def : ProcessorModel<"cortex-m4", CortexM4Model,        [ARMv7em,
                                                         FeatureVFP4_D16_SP,
                                                         FeaturePreferBranchAlign32,
                                                         FeatureHasSlowFPVMLx,
                                                         FeatureHasSlowFPVFMx,
                                                         FeatureUseMISched,
                                                         FeatureHasNoBranchPredictor]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 350-355
```tablegen
def : ProcessorModel<"cortex-m7", CortexM7Model,        [ARMv7em,
                                                         ProcM7,
                                                         FeatureFPARMv8_D16,
                                                         FeaturePreferBranchAlign64,
                                                         FeatureUseMIPipeliner,
                                                         FeatureUseMISched]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 357-359
```tablegen
def : ProcNoItin<"cortex-m23",                          [ARMv8mBaseline,
                                                         FeatureNoMovt,
                                                         FeatureHasNoBranchPredictor]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 361-370
```tablegen
def : ProcessorModel<"cortex-m33", CortexM4Model,       [ARMv8mMainline,
                                                         FeatureDSP,
                                                         FeatureFPARMv8_D16_SP,
                                                         FeaturePreferBranchAlign32,
                                                         FeatureHasSlowFPVMLx,
                                                         FeatureHasSlowFPVFMx,
                                                         FeatureUseMISched,
                                                         FeatureHasNoBranchPredictor,
                                                         FeatureAvoidMULS,
                                                         FeatureFixCMSE_CVE_2021_35465]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 372-381
```tablegen
def : ProcessorModel<"star-mc1", CortexM4Model,         [ARMv8mMainline,
                                                         FeatureDSP,
                                                         FeatureFPARMv8_D16_SP,
                                                         FeaturePreferBranchAlign32,
                                                         FeatureHasSlowFPVMLx,
                                                         FeatureHasSlowFPVFMx,
                                                         FeatureUseMISched,
                                                         FeatureHasNoBranchPredictor,
                                                         FeatureAvoidMULS,
                                                         FeatureFixCMSE_CVE_2021_35465]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 383-391
```tablegen
def : ProcessorModel<"cortex-m35p", CortexM4Model,      [ARMv8mMainline,
                                                         FeatureDSP,
                                                         FeatureFPARMv8_D16_SP,
                                                         FeaturePreferBranchAlign32,
                                                         FeatureHasSlowFPVMLx,
                                                         FeatureHasSlowFPVFMx,
                                                         FeatureUseMISched,
                                                         FeatureHasNoBranchPredictor,
                                                         FeatureFixCMSE_CVE_2021_35465]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 393-402
```tablegen
def : ProcessorModel<"cortex-m55", CortexM55Model,      [ARMv81mMainline,
                                                         ProcM55,
                                                         FeatureDSP,
                                                         FeatureFPARMv8_D16,
                                                         FeatureUseMISched,
                                                         FeatureHasNoBranchPredictor,
                                                         FeaturePreferBranchAlign32,
                                                         FeatureHasSlowFPVMLx,
                                                         HasMVEFloatOps,
                                                         FeatureFixCMSE_CVE_2021_35465]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 404-411
```tablegen
def : ProcessorModel<"cortex-m85", CortexM85Model,      [ARMv81mMainline,
                                                         ProcM85,
                                                         FeatureDSP,
                                                         FeatureFPARMv8_D16,
                                                         FeaturePACBTI,
                                                         FeaturePreferBranchAlign64,
                                                         FeatureUseMISched,
                                                         HasMVEFloatOps]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 413-422
```tablegen
def : ProcessorModel<"cortex-m52", CortexM55Model,      [ARMv81mMainline,
                                                         FeatureDSP,
                                                         FeatureFPARMv8_D16,
                                                         FeatureHasNoBranchPredictor,
                                                         FeaturePACBTI,
                                                         FeatureUseMISched,
                                                         FeaturePreferBranchAlign32,
                                                         FeatureHasSlowFPVMLx,
                                                         FeatureMVEVectorCostFactor1,
                                                         HasMVEFloatOps]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 424-433
```tablegen
def : ProcessorModel<"star-mc3", CortexM55Model,      [ARMv81mMainline,
                                                         FeatureDSP,
                                                         FeatureFPARMv8_D16,
                                                         FeatureHasNoBranchPredictor,
                                                         FeaturePACBTI,
                                                         FeatureUseMISched,
                                                         FeaturePreferBranchAlign32,
                                                         FeatureHasSlowFPVMLx,
                                                         FeatureMVEVectorCostFactor1,
                                                         HasMVEFloatOps]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 435-439
```tablegen
def : ProcNoItin<"cortex-a32",                           [ARMv8a,
                                                         FeatureHWDivThumb,
                                                         FeatureHWDivARM,
                                                         FeatureCrypto,
                                                         FeatureCRC]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 441-445
```tablegen
def : ProcNoItin<"cortex-a35",                          [ARMv8a, ProcA35,
                                                         FeatureHWDivThumb,
                                                         FeatureHWDivARM,
                                                         FeatureCrypto,
                                                         FeatureCRC]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 447-452
```tablegen
def : ProcNoItin<"cortex-a53",                          [ARMv8a, ProcA53,
                                                         FeatureHWDivThumb,
                                                         FeatureHWDivARM,
                                                         FeatureCrypto,
                                                         FeatureCRC,
                                                         FeatureFPAO]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 454-457
```tablegen
def : ProcNoItin<"cortex-a55",                          [ARMv82a, ProcA55,
                                                         FeatureHWDivThumb,
                                                         FeatureHWDivARM,
                                                         FeatureDotProd]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 459-467
```tablegen
def : ProcessorModel<"cortex-a57",  CortexA57Model,     [ARMv8a, ProcA57,
                                                         FeatureHWDivThumb,
                                                         FeatureHWDivARM,
                                                         FeatureCrypto,
                                                         FeatureCRC,
                                                         FeatureFPAO,
                                                         FeatureAvoidPartialCPSR,
                                                         FeatureCheapPredicableCPSR,
                                                         FeatureFixCortexA57AES1742098]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 469-474
```tablegen
def : ProcessorModel<"cortex-a72",  CortexA57Model,     [ARMv8a, ProcA72,
                                                         FeatureHWDivThumb,
                                                         FeatureHWDivARM,
                                                         FeatureCrypto,
                                                         FeatureCRC,
                                                         FeatureFixCortexA57AES1742098]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 476-480
```tablegen
def : ProcNoItin<"cortex-a73",                          [ARMv8a, ProcA73,
                                                         FeatureHWDivThumb,
                                                         FeatureHWDivARM,
                                                         FeatureCrypto,
                                                         FeatureCRC]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 482-485
```tablegen
def : ProcNoItin<"cortex-a75",                          [ARMv82a, ProcA75,
                                                         FeatureHWDivThumb,
                                                         FeatureHWDivARM,
                                                         FeatureDotProd]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 487-493
```tablegen
def : ProcNoItin<"cortex-a76",                          [ARMv82a, ProcA76,
                                                         FeatureHWDivThumb,
                                                         FeatureHWDivARM,
                                                         FeatureCrypto,
                                                         FeatureCRC,
                                                         FeatureFullFP16,
                                                         FeatureDotProd]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 495-501
```tablegen
def : ProcNoItin<"cortex-a76ae",                        [ARMv82a, ProcA76,
                                                         FeatureHWDivThumb,
                                                         FeatureHWDivARM,
                                                         FeatureCrypto,
                                                         FeatureCRC,
                                                         FeatureFullFP16,
                                                         FeatureDotProd]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 503-509
```tablegen
def : ProcNoItin<"cortex-a77",                          [ARMv82a, ProcA77,
                                                         FeatureHWDivThumb,
                                                         FeatureHWDivARM,
                                                         FeatureCrypto,
                                                         FeatureCRC,
                                                         FeatureFullFP16,
                                                         FeatureDotProd]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 511-517
```tablegen
def : ProcNoItin<"cortex-a78",                          [ARMv82a, ProcA78,
                                                         FeatureHWDivThumb,
                                                         FeatureHWDivARM,
                                                         FeatureCrypto,
                                                         FeatureCRC,
                                                         FeatureFullFP16,
                                                         FeatureDotProd]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 519-525
```tablegen
def : ProcNoItin<"cortex-a78ae",                        [ARMv82a, ProcA78AE,
                                                         FeatureHWDivThumb,
                                                         FeatureHWDivARM,
                                                         FeatureCrypto,
                                                         FeatureCRC,
                                                         FeatureFullFP16,
                                                         FeatureDotProd]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 527-533
```tablegen
def : ProcNoItin<"cortex-a78c",                         [ARMv82a, ProcA78C,
                                                         FeatureHWDivThumb,
                                                         FeatureHWDivARM,
                                                         FeatureCrypto,
                                                         FeatureCRC,
                                                         FeatureDotProd,
                                                         FeatureFullFP16]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 535-541
```tablegen
def : ProcNoItin<"cortex-a510",                         [ARMv9a, ProcA710,
                                                         FeatureHWDivThumb,
                                                         FeatureHWDivARM,
                                                         FeatureFP16FML,
                                                         FeatureBF16,
                                                         FeatureMatMulInt8,
                                                         FeatureSB]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 543-549
```tablegen
def : ProcNoItin<"cortex-a710",                         [ARMv9a, ProcA710,
                                                         FeatureHWDivThumb,
                                                         FeatureHWDivARM,
                                                         FeatureFP16FML,
                                                         FeatureBF16,
                                                         FeatureMatMulInt8,
                                                         FeatureSB]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 551-557
```tablegen
def : ProcNoItin<"cortex-x1",                           [ARMv82a, ProcX1,
                                                         FeatureHWDivThumb,
                                                         FeatureHWDivARM,
                                                         FeatureCrypto,
                                                         FeatureCRC,
                                                         FeatureFullFP16,
                                                         FeatureDotProd]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 559-565
```tablegen
def : ProcNoItin<"cortex-x1c",                          [ARMv82a, ProcX1C,
                                                         FeatureHWDivThumb,
                                                         FeatureHWDivARM,
                                                         FeatureCrypto,
                                                         FeatureCRC,
                                                         FeatureFullFP16,
                                                         FeatureDotProd]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 567-574
```tablegen
def : ProcNoItin<"neoverse-v1",                         [ARMv84a,
                                                         FeatureHWDivThumb,
                                                         FeatureHWDivARM,
                                                         FeatureCrypto,
                                                         FeatureCRC,
                                                         FeatureFullFP16,
                                                         FeatureBF16,
                                                         FeatureMatMulInt8]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 576-581
```tablegen
def : ProcNoItin<"neoverse-n1",                         [ARMv82a,
                                                         FeatureHWDivThumb,
                                                         FeatureHWDivARM,
                                                         FeatureCrypto,
                                                         FeatureCRC,
                                                         FeatureDotProd]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 583-586
```tablegen
def : ProcNoItin<"neoverse-n2",                         [ARMv9a,
                                                         FeatureBF16,
                                                         FeatureFP16FML,
                                                         FeatureMatMulInt8]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 588-602
```tablegen
def : ProcessorModel<"cyclone",     SwiftModel,         [ARMv8a, ProcSwift,
                                                         FeatureHasRetAddrStack,
                                                         FeatureNEONForFP,
                                                         FeatureVFP4,
                                                         FeatureMP,
                                                         FeatureHWDivThumb,
                                                         FeatureHWDivARM,
                                                         FeatureAvoidPartialCPSR,
                                                         FeatureAvoidMOVsShOp,
                                                         FeatureHasSlowFPVMLx,
                                                         FeatureHasSlowFPVFMx,
                                                         FeatureCrypto,
                                                         FeatureUseMISched,
                                                         FeatureZCZeroing,
                                                         FeatureNoPostRASched]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 604-610
```tablegen
def : ProcNoItin<"exynos-m3",                           [ARMv8a, ProcExynos]>;
def : ProcNoItin<"exynos-m4",                           [ARMv82a, ProcExynos,
                                                         FeatureFullFP16,
                                                         FeatureDotProd]>;
def : ProcNoItin<"exynos-m5",                           [ARMv82a, ProcExynos,
                                                         FeatureFullFP16,
                                                         FeatureDotProd]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 612-616
```tablegen
def : ProcNoItin<"kryo",                                [ARMv8a, ProcKryo,
                                                         FeatureHWDivThumb,
                                                         FeatureHWDivARM,
                                                         FeatureCrypto,
                                                         FeatureCRC]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 618-622
```tablegen
def : ProcessorModel<"cortex-r52", CortexR52Model,      [ARMv8r, ProcR52,
                                                         FeatureFPARMv8,
                                                         FeatureNEON,
                                                         FeatureUseMISched,
                                                         FeatureFPAO]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 624-628
```tablegen
def : ProcessorModel<"cortex-r52plus", CortexR52Model,  [ARMv8r, ProcR52plus,
                                                         FeatureFPARMv8,
                                                         FeatureNEON,
                                                         FeatureUseMISched,
                                                         FeatureFPAO]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

## Key Concepts / 关键概念

- EN: Primary role: target descriptions, instruction records, and code-generation metadata.
  - CN: 核心职责：目标描述、指令记录以及代码生成元数据。
- EN: LLVM TableGen DSL records drive generated enums, matchers, encoders, or scheduling data.
  - CN: LLVM TableGen DSL 记录会驱动生成枚举、匹配器、编码器或调度数据。
- EN: Predicates and feature gates control when records apply to a subtarget or ISA variant.
  - CN: 谓词与特性门控决定记录在何种子目标或 ISA 变体下生效。

## Dependencies / 依赖关系

- EN: TableGen includes: `(none)`.
  - CN: TableGen 包含项：`(none)`。
- EN: Generated backend tables depend on tblgen consuming these records and emitting C++/inc fragments.
  - CN: 生成后的后端表依赖 tblgen 消费这些记录并产出 C++/inc 片段。
