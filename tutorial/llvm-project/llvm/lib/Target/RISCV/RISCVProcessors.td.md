# RISCVProcessors.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVProcessors.td`
- **Repository**: llvm/llvm-project
- **Purpose**: Defines TableGen records for feature bits, processor models, and architectural profile metadata for the RISC-V backend. / 使用 TableGen 定义RISC-V 后端的特性位、处理器模型与架构配置档元数据。
- **Note**: This file uses the TableGen DSL to describe target metadata declaratively. / 该文件使用 TableGen DSL 以声明式方式描述目标元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17: Commentary and design intent / 注释与设计意图
```tablegen
//===-- RISCVProcessors.td - RISC-V Processors -------------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

//===----------------------------------------------------------------------===//
// RISC-V processors supported.
//===----------------------------------------------------------------------===//

// Predefined scheduling direction.
defvar TopDown       = [{ MISched::TopDown }];
defvar BottomUp      = [{ MISched::BottomUp }];
defvar Bidirectional = [{ MISched::Bidirectional }];
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 18-32: TableGen class RISCVTuneInfo / TableGen 类 RISCVTuneInfo
```tablegen
class RISCVTuneInfo {
  bits<8> PrefFunctionAlignment = 1;
  bits<8> PrefLoopAlignment = 1;

  // Information needed by LoopDataPrefetch.
  bits<16> CacheLineSize = 0;
  bits<16> PrefetchDistance = 0;
  bits<16> MinPrefetchStride = 1;
  bits<32> MaxPrefetchIterationsAhead = -1;

  bits<32> MinimumJumpTableEntries = 5;

  // Tail duplication threshold at -O3.
  bits<32> TailDupAggressiveThreshold = 6;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 33-48: Definitions and supporting logic / 定义与支撑逻辑
```tablegen
  bits<32> MaxStoresPerMemsetOptSize = 4;
  bits<32> MaxStoresPerMemset = 8;

  bits<32> MaxGluedStoresPerMemcpy = 0;
  bits<32> MaxStoresPerMemcpyOptSize = 4;
  bits<32> MaxStoresPerMemcpy = 8;

  bits<32> MaxStoresPerMemmoveOptSize = 4;
  bits<32> MaxStoresPerMemmove = 8;

  bits<32> MaxLoadsPerMemcmpOptSize = 4;
  bits<32> MaxLoadsPerMemcmp = 8;

  // The direction of PostRA scheduling.
  code PostRASchedDirection = TopDown;
}
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 49-62: TableGen record RISCVTuneInfoTable / TableGen 记录 RISCVTuneInfoTable
```tablegen

def RISCVTuneInfoTable : GenericTable {
  let FilterClass = "RISCVTuneInfo";
  let CppTypeName = "RISCVTuneInfo";
  let Fields = ["Name", "PrefFunctionAlignment", "PrefLoopAlignment",
                "CacheLineSize", "PrefetchDistance", "MinPrefetchStride",
                "MaxPrefetchIterationsAhead", "MinimumJumpTableEntries",
                "TailDupAggressiveThreshold", "MaxStoresPerMemsetOptSize",
                "MaxStoresPerMemset", "MaxGluedStoresPerMemcpy",
                "MaxStoresPerMemcpyOptSize", "MaxStoresPerMemcpy",
                "MaxStoresPerMemmoveOptSize", "MaxStoresPerMemmove",
                "MaxLoadsPerMemcmpOptSize", "MaxLoadsPerMemcmp",
                "PostRASchedDirection"];
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 63-81: TableGen class GenericTuneInfo: / TableGen 类 GenericTuneInfo:
```tablegen

def getRISCVTuneInfo : SearchIndex {
  let Table = RISCVTuneInfoTable;
  let Key = ["Name"];
}

class GenericTuneInfo: RISCVTuneInfo;

class RISCVProcessorModel<string n,
                          SchedMachineModel m,
                          list<SubtargetFeature> f,
                          list<SubtargetFeature> tunef = [],
                          string default_march = "">
    :  ProcessorModel<n, m, f, tunef> {
  string DefaultMarch = default_march;
  int MVendorID = 0;
  int MArchID = 0;
  int MImpID = 0;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 82-109: TableGen class RISCVTuneProcessorModel<string / TableGen 类 RISCVTuneProcessorModel<string
```tablegen
  list<RISCVTuneFeature> ConfigurableTuneFeatures = [];
}

class RISCVTuneProcessorModel<string n, SchedMachineModel m,
                              list<SubtargetFeature> tunef = [],
                              list<SubtargetFeature> f = []>
    : ProcessorModel<n, m, f, tunef> {
  list<RISCVTuneFeature> ConfigurableTuneFeatures = [];
}

defvar GenericTuneFeatures = [TuneOptimizedNF2SegmentLoadStore,
                              TuneEnableSelectOptimize];

def GENERIC_RV32 : RISCVProcessorModel<"generic-rv32",
                                       NoSchedModel,
                                       [Feature32Bit,
                                        FeatureStdExtI],
                                       GenericTuneFeatures>,
                   GenericTuneInfo;
def GENERIC_RV64 : RISCVProcessorModel<"generic-rv64",
                                       NoSchedModel,
                                       [Feature64Bit,
                                        FeatureStdExtI],
                                       GenericTuneFeatures>,
                   GenericTuneInfo;
// Support generic for compatibility with other targets. The triple will be used
// to change to the appropriate rv32/rv64 version.
def GENERIC : RISCVTuneProcessorModel<"generic", NoSchedModel>, GenericTuneInfo;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 110-132: TableGen record GENERIC_OOO / TableGen 记录 GENERIC_OOO
```tablegen
def GENERIC_OOO : RISCVTuneProcessorModel<"generic-ooo", GenericOOOModel>,
                  GenericTuneInfo;

// clang-format off
def MIPS_P8700 : RISCVProcessorModel<"mips-p8700",
                                     MIPSP8700Model,
                                     [Feature64Bit,
                                      FeatureStdExtI,
                                      FeatureStdExtM,
                                      FeatureStdExtA,
                                      FeatureStdExtF,
                                      FeatureStdExtD,
                                      FeatureStdExtC,
                                      FeatureStdExtZba,
                                      FeatureStdExtZbb,
                                      FeatureStdExtZifencei,
                                      FeatureStdExtZicsr,
                                      FeatureVendorXMIPSCMov,
                                      FeatureVendorXMIPSLSP,
                                      FeatureVendorXMIPSCBOP,
                                      FeatureVendorXMIPSEXECTL],
                                     [TuneMIPSP8700]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 133-147: TableGen record ROCKET_RV32 / TableGen 记录 ROCKET_RV32
```tablegen
def ROCKET_RV32 : RISCVProcessorModel<"rocket-rv32",
                                      RocketModel,
                                      [Feature32Bit,
                                       FeatureStdExtI,
                                       FeatureStdExtZifencei,
                                       FeatureStdExtZicsr]>;
def ROCKET_RV64 : RISCVProcessorModel<"rocket-rv64",
                                      RocketModel,
                                      [Feature64Bit,
                                       FeatureStdExtI,
                                       FeatureStdExtZifencei,
                                       FeatureStdExtZicsr]>;
def ROCKET : RISCVTuneProcessorModel<"rocket",
                                     RocketModel>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 148-162: TableGen record SIFIVE_7 / TableGen 记录 SIFIVE_7
```tablegen
defvar SiFive7TuneFeatures = [TuneSiFive7, TuneNoDefaultUnroll,
                              TuneShortForwardBranchIALU,
                              TunePostRAScheduler];
def SIFIVE_7 : RISCVTuneProcessorModel<"sifive-7-series",
                                       SiFive7Model, SiFive7TuneFeatures>;

def SIFIVE_E20 : RISCVProcessorModel<"sifive-e20",
                                     RocketModel,
                                     [Feature32Bit,
                                      FeatureStdExtI,
                                      FeatureStdExtZicsr,
                                      FeatureStdExtZifencei,
                                      FeatureStdExtM,
                                      FeatureStdExtC]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 163-182: TableGen record SIFIVE_E21 / TableGen 记录 SIFIVE_E21
```tablegen
def SIFIVE_E21 : RISCVProcessorModel<"sifive-e21",
                                     RocketModel,
                                     [Feature32Bit,
                                      FeatureStdExtI,
                                      FeatureStdExtZicsr,
                                      FeatureStdExtZifencei,
                                      FeatureStdExtM,
                                      FeatureStdExtA,
                                      FeatureStdExtC]>;

def SIFIVE_E24 : RISCVProcessorModel<"sifive-e24",
                                     RocketModel,
                                     [Feature32Bit,
                                      FeatureStdExtI,
                                      FeatureStdExtZifencei,
                                      FeatureStdExtM,
                                      FeatureStdExtA,
                                      FeatureStdExtF,
                                      FeatureStdExtC]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 183-202: TableGen record SIFIVE_E31 / TableGen 记录 SIFIVE_E31
```tablegen
def SIFIVE_E31 : RISCVProcessorModel<"sifive-e31",
                                     RocketModel,
                                     [Feature32Bit,
                                      FeatureStdExtI,
                                      FeatureStdExtZifencei,
                                      FeatureStdExtZicsr,
                                      FeatureStdExtM,
                                      FeatureStdExtA,
                                      FeatureStdExtC]>;

def SIFIVE_E34 : RISCVProcessorModel<"sifive-e34",
                                     RocketModel,
                                     [Feature32Bit,
                                      FeatureStdExtI,
                                      FeatureStdExtZifencei,
                                      FeatureStdExtM,
                                      FeatureStdExtA,
                                      FeatureStdExtF,
                                      FeatureStdExtC]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 203-223: TableGen record SIFIVE_E76 / TableGen 记录 SIFIVE_E76
```tablegen
def SIFIVE_E76 : RISCVProcessorModel<"sifive-e76",
                                     SiFive7Model,
                                     [Feature32Bit,
                                      FeatureStdExtI,
                                      FeatureStdExtZifencei,
                                      FeatureStdExtM,
                                      FeatureStdExtA,
                                      FeatureStdExtF,
                                      FeatureStdExtC],
                                     SiFive7TuneFeatures>;

def SIFIVE_S21 : RISCVProcessorModel<"sifive-s21",
                                     RocketModel,
                                     [Feature64Bit,
                                      FeatureStdExtI,
                                      FeatureStdExtZicsr,
                                      FeatureStdExtZifencei,
                                      FeatureStdExtM,
                                      FeatureStdExtA,
                                      FeatureStdExtC]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 224-244: TableGen record SIFIVE_S51 / TableGen 记录 SIFIVE_S51
```tablegen
def SIFIVE_S51 : RISCVProcessorModel<"sifive-s51",
                                     RocketModel,
                                     [Feature64Bit,
                                      FeatureStdExtI,
                                      FeatureStdExtZicsr,
                                      FeatureStdExtZifencei,
                                      FeatureStdExtM,
                                      FeatureStdExtA,
                                      FeatureStdExtC]>;

def SIFIVE_S54 : RISCVProcessorModel<"sifive-s54",
                                      RocketModel,
                                      [Feature64Bit,
                                       FeatureStdExtI,
                                       FeatureStdExtZifencei,
                                       FeatureStdExtM,
                                       FeatureStdExtA,
                                       FeatureStdExtF,
                                       FeatureStdExtD,
                                       FeatureStdExtC]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 245-268: TableGen record SIFIVE_S76 / TableGen 记录 SIFIVE_S76
```tablegen
def SIFIVE_S76 : RISCVProcessorModel<"sifive-s76",
                                     SiFive7Model,
                                     [Feature64Bit,
                                      FeatureStdExtI,
                                      FeatureStdExtZifencei,
                                      FeatureStdExtM,
                                      FeatureStdExtA,
                                      FeatureStdExtF,
                                      FeatureStdExtD,
                                      FeatureStdExtC,
                                      FeatureStdExtZihintpause],
                                     SiFive7TuneFeatures>;

def SIFIVE_U54 : RISCVProcessorModel<"sifive-u54",
                                     RocketModel,
                                     [Feature64Bit,
                                      FeatureStdExtI,
                                      FeatureStdExtZifencei,
                                      FeatureStdExtM,
                                      FeatureStdExtA,
                                      FeatureStdExtF,
                                      FeatureStdExtD,
                                      FeatureStdExtC]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 269-296: TableGen record SIFIVE_U74 / TableGen 记录 SIFIVE_U74
```tablegen
def SIFIVE_U74 : RISCVProcessorModel<"sifive-u74",
                                     SiFive7Model,
                                     [Feature64Bit,
                                      FeatureStdExtI,
                                      FeatureStdExtZifencei,
                                      FeatureStdExtM,
                                      FeatureStdExtA,
                                      FeatureStdExtF,
                                      FeatureStdExtD,
                                      FeatureStdExtC],
                                     SiFive7TuneFeatures>;

defvar SiFiveIntelligenceTuneFeatures = !listconcat(SiFive7TuneFeatures,
                                                    [TuneDLenFactor2,
                                                     TuneOptimizedZeroStrideLoad,
                                                     TuneOptimizedNF2SegmentLoadStore,
                                                     TuneVLDependentLatency]);
def SIFIVE_X280 : RISCVProcessorModel<"sifive-x280", SiFive7Model,
                                      [Feature64Bit,
                                       FeatureStdExtI,
                                       FeatureStdExtZifencei,
                                       FeatureStdExtM,
                                       FeatureStdExtA,
                                       FeatureStdExtF,
                                       FeatureStdExtD,
                                       FeatureStdExtC,
                                       FeatureStdExtV,
                                       FeatureStdExtZvl512b,
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 297-324: TableGen record SIFIVE_X390 / TableGen 记录 SIFIVE_X390
```tablegen
                                       FeatureStdExtZfh,
                                       FeatureStdExtZvfh,
                                       FeatureStdExtZba,
                                       FeatureStdExtZbb],
                                      SiFiveIntelligenceTuneFeatures> {
  let ConfigurableTuneFeatures = [TuneHasSingleElementVecFP64];
}

def SIFIVE_X390 : RISCVProcessorModel<"sifive-x390",
                                      SiFiveX390Model,
                                      [Feature64Bit,
                                       FeatureStdExtI,
                                       FeatureStdExtM,
                                       FeatureStdExtA,
                                       FeatureStdExtF,
                                       FeatureStdExtD,
                                       FeatureStdExtC,
                                       FeatureStdExtB,
                                       FeatureStdExtV,
                                       FeatureStdExtZic64b,
                                       FeatureStdExtZicbom,
                                       FeatureStdExtZicbop,
                                       FeatureStdExtZicboz,
                                       FeatureStdExtZiccamoa,
                                       FeatureStdExtZiccif,
                                       FeatureStdExtZiccrse,
                                       FeatureStdExtZicfilp,
                                       FeatureStdExtZicfiss,
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 325-351: TableGen field customization and iteration / TableGen 字段定制与迭代
```tablegen
                                       FeatureStdExtZicntr,
                                       FeatureStdExtZicond,
                                       FeatureStdExtZifencei,
                                       FeatureStdExtZihintntl,
                                       FeatureStdExtZihintpause,
                                       FeatureStdExtZihpm,
                                       FeatureStdExtZimop,
                                       FeatureStdExtZa64rs,
                                       FeatureStdExtZawrs,
                                       FeatureStdExtZfa,
                                       FeatureStdExtZfh,
                                       FeatureStdExtZcb,
                                       FeatureStdExtZcmop,
                                       FeatureStdExtZkr,
                                       FeatureStdExtZkt,
                                       FeatureStdExtZvbb,
                                       FeatureStdExtZvfbfmin,
                                       FeatureStdExtZvfbfwma,
                                       FeatureStdExtZvfh,
                                       FeatureStdExtZvkt,
                                       FeatureStdExtZvl1024b,
                                       FeatureVendorXSiFivecdiscarddlone,
                                       FeatureVendorXSiFivecflushdlone],
                                      !listconcat(SiFiveIntelligenceTuneFeatures,
                                                  [TuneHasSingleElementVecFP64])> {
  let ConfigurableTuneFeatures = [TuneHasSingleElementVecFP64];
}
```
**EN:** This block adjusts inherited fields or iterates over parameter sets to shape many related records consistently.

**CN:** 该区段调整继承字段或遍历参数集合，以一致地塑造多条相关记录。

### Lines 352-379: TableGen record SIFIVE_X160 / TableGen 记录 SIFIVE_X160
```tablegen

def SIFIVE_X160 : RISCVProcessorModel<"sifive-x160",
                                      SiFiveX100Model,
                                      [Feature32Bit,
                                       FeatureStdExtI,
                                       FeatureStdExtM,
                                       FeatureStdExtA,
                                       FeatureStdExtF,
                                       FeatureStdExtC,
                                       FeatureStdExtB,
                                       FeatureStdExtZicbom,
                                       FeatureStdExtZicbop,
                                       FeatureStdExtZicboz,
                                       FeatureStdExtZicond,
                                       FeatureStdExtZifencei,
                                       FeatureStdExtZihintntl,
                                       FeatureStdExtZihintpause,
                                       FeatureStdExtZihpm,
                                       FeatureStdExtZimop,
                                       FeatureStdExtZawrs,
                                       FeatureStdExtZfa,
                                       FeatureStdExtZfbfmin,
                                       FeatureStdExtZfh,
                                       FeatureStdExtZcb,
                                       FeatureStdExtZce,
                                       FeatureStdExtZcf,
                                       FeatureStdExtZcmop,
                                       FeatureStdExtZcmp,
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 380-395: Definitions and supporting logic / 定义与支撑逻辑
```tablegen
                                       FeatureStdExtZcmt,
                                       FeatureStdExtZkt,
                                       FeatureStdExtZvbb,
                                       FeatureStdExtZve32f,
                                       FeatureStdExtZve32x,
                                       FeatureStdExtZvfbfmin,
                                       FeatureStdExtZvfbfwma,
                                       FeatureStdExtZvfh,
                                       FeatureStdExtZvkt,
                                       FeatureStdExtZvl128b,
                                       FeatureVendorXSfcease,
                                       FeatureStdExtZicfilp,
                                       FeatureStdExtZvdot4a8i,
                                       FeatureStdExtZvfbfa],
                                       SiFiveIntelligenceTuneFeatures>;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 396-423: TableGen record SIFIVE_X180 / TableGen 记录 SIFIVE_X180
```tablegen
def SIFIVE_X180 : RISCVProcessorModel<"sifive-x180",
                                      SiFiveX100Model,
                                      [Feature64Bit,
                                       FeatureStdExtI,
                                       FeatureStdExtM,
                                       FeatureStdExtA,
                                       FeatureStdExtF,
                                       FeatureStdExtD,
                                       FeatureStdExtC,
                                       FeatureStdExtB,
                                       FeatureStdExtV,
                                       FeatureStdExtZic64b,
                                       FeatureStdExtZicbom,
                                       FeatureStdExtZicbop,
                                       FeatureStdExtZicboz,
                                       FeatureStdExtZiccamoa,
                                       FeatureStdExtZiccif,
                                       FeatureStdExtZiccrse,
                                       FeatureStdExtZicond,
                                       FeatureStdExtZifencei,
                                       FeatureStdExtZihintntl,
                                       FeatureStdExtZihintpause,
                                       FeatureStdExtZihpm,
                                       FeatureStdExtZimop,
                                       FeatureStdExtZa64rs,
                                       FeatureStdExtZawrs,
                                       FeatureStdExtZfa,
                                       FeatureStdExtZfbfmin,
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 424-447: TableGen field customization and iteration / TableGen 字段定制与迭代
```tablegen
                                       FeatureStdExtZfh,
                                       FeatureStdExtZcb,
                                       FeatureStdExtZcd,
                                       FeatureStdExtZcmop,
                                       FeatureStdExtZkt,
                                       FeatureStdExtZvbb,
                                       FeatureStdExtZvfbfmin,
                                       FeatureStdExtZvfbfwma,
                                       FeatureStdExtZvfh,
                                       FeatureStdExtZvkb,
                                       FeatureStdExtZvkt,
                                       FeatureStdExtZvl128b,
                                       FeatureVendorXSfcease,
                                       FeatureVendorXSfvfbfexp16e,
                                       FeatureVendorXSfvfexp16e,
                                       FeatureVendorXSfvfexp32e,
                                       FeatureVendorXSfvfexpa,
                                       FeatureVendorXSiFivecflushdlone,
                                       FeatureStdExtZicfilp,
                                       FeatureStdExtZvdot4a8i,
                                       FeatureStdExtZvfbfa],
                                       SiFiveIntelligenceTuneFeatures> {
  let ConfigurableTuneFeatures = [TuneHasSingleElementVecFP64];
}
```
**EN:** This block adjusts inherited fields or iterates over parameter sets to shape many related records consistently.

**CN:** 该区段调整继承字段或遍历参数集合，以一致地塑造多条相关记录。

### Lines 448-475: TableGen record SIFIVE_P450 / TableGen 记录 SIFIVE_P450
```tablegen

defvar SiFiveP400TuneFeatures = [TuneNoDefaultUnroll,
                                 TuneConditionalCompressedMoveFusion,
                                 TuneLUIADDIFusion,
                                 TuneAUIPCADDIFusion,
                                 TunePostRAScheduler];

def SIFIVE_P450 : RISCVProcessorModel<"sifive-p450", SiFiveP400Model,
                                      [Feature64Bit,
                                       FeatureStdExtI,
                                       FeatureStdExtM,
                                       FeatureStdExtA,
                                       FeatureStdExtF,
                                       FeatureStdExtD,
                                       FeatureStdExtC,
                                       FeatureStdExtZicsr,
                                       FeatureStdExtZiccif,
                                       FeatureStdExtZiccrse,
                                       FeatureStdExtZiccamoa,
                                       FeatureStdExtZicclsm,
                                       FeatureStdExtZa64rs,
                                       FeatureStdExtZihpm,
                                       FeatureStdExtZihintpause,
                                       FeatureStdExtB,
                                       FeatureStdExtZic64b,
                                       FeatureStdExtZicbom,
                                       FeatureStdExtZicbop,
                                       FeatureStdExtZicboz,
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 476-503: TableGen record SIFIVE_P470 / TableGen 记录 SIFIVE_P470
```tablegen
                                       FeatureStdExtZfhmin,
                                       FeatureStdExtZkt,
                                       FeatureStdExtZifencei,
                                       FeatureStdExtZihintntl,
                                       FeatureUnalignedScalarMem,
                                       FeatureUnalignedVectorMem],
                                      SiFiveP400TuneFeatures>;

def SIFIVE_P470 : RISCVProcessorModel<"sifive-p470", SiFiveP400Model,
                                      [Feature64Bit,
                                       FeatureStdExtI,
                                       FeatureStdExtM,
                                       FeatureStdExtA,
                                       FeatureStdExtF,
                                       FeatureStdExtD,
                                       FeatureStdExtC,
                                       FeatureStdExtZicsr,
                                       FeatureStdExtZiccif,
                                       FeatureStdExtZiccrse,
                                       FeatureStdExtZiccamoa,
                                       FeatureStdExtZicclsm,
                                       FeatureStdExtZa64rs,
                                       FeatureStdExtZihpm,
                                       FeatureStdExtZihintpause,
                                       FeatureStdExtB,
                                       FeatureStdExtZic64b,
                                       FeatureStdExtZicbom,
                                       FeatureStdExtZicbop,
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 504-523: Definitions and supporting logic / 定义与支撑逻辑
```tablegen
                                       FeatureStdExtZicboz,
                                       FeatureStdExtZfhmin,
                                       FeatureStdExtZkt,
                                       FeatureStdExtV,
                                       FeatureStdExtZifencei,
                                       FeatureStdExtZihintntl,
                                       FeatureStdExtZvl128b,
                                       FeatureStdExtZvbb,
                                       FeatureStdExtZvknc,
                                       FeatureStdExtZvkng,
                                       FeatureStdExtZvksc,
                                       FeatureStdExtZvksg,
                                       FeatureVendorXSiFivecdiscarddlone,
                                       FeatureVendorXSiFivecflushdlone,
                                       FeatureUnalignedScalarMem,
                                       FeatureUnalignedVectorMem],
                                      !listconcat(SiFiveP400TuneFeatures,
                                                  [TuneNoSinkSplatOperands,
                                                   TuneVXRMPipelineFlush])>;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 524-546: TableGen record SIFIVE_P550 / TableGen 记录 SIFIVE_P550
```tablegen
defvar SiFiveP500TuneFeatures = [TuneNoDefaultUnroll,
                                 TuneEnableSelectOptimize,
                                 TuneConditionalCompressedMoveFusion,
                                 TuneLUIADDIFusion,
                                 TuneAUIPCADDIFusion,
                                 TunePostRAScheduler];

def SIFIVE_P550 : RISCVProcessorModel<"sifive-p550", SiFiveP500Model,
                                      [Feature64Bit,
                                       FeatureStdExtI,
                                       FeatureStdExtZifencei,
                                       FeatureStdExtM,
                                       FeatureStdExtA,
                                       FeatureStdExtF,
                                       FeatureStdExtD,
                                       FeatureStdExtC,
                                       FeatureStdExtZba,
                                       FeatureStdExtZbb],
                                      SiFiveP500TuneFeatures> {
  let MVendorID = 0x489;
  let MArchID = 0x8000000000000008;
  let MImpID = 0x6220425;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 547-574: TableGen record SIFIVE_P670 / TableGen 记录 SIFIVE_P670
```tablegen

def SIFIVE_P670 : RISCVProcessorModel<"sifive-p670", SiFiveP600Model,
                                      [Feature64Bit,
                                       FeatureStdExtI,
                                       FeatureStdExtM,
                                       FeatureStdExtA,
                                       FeatureStdExtF,
                                       FeatureStdExtD,
                                       FeatureStdExtC,
                                       FeatureStdExtZicsr,
                                       FeatureStdExtZiccif,
                                       FeatureStdExtZiccrse,
                                       FeatureStdExtZiccamoa,
                                       FeatureStdExtZicclsm,
                                       FeatureStdExtZa64rs,
                                       FeatureStdExtZihpm,
                                       FeatureStdExtZihintpause,
                                       FeatureStdExtB,
                                       FeatureStdExtZic64b,
                                       FeatureStdExtZicbom,
                                       FeatureStdExtZicbop,
                                       FeatureStdExtZicboz,
                                       FeatureStdExtZfhmin,
                                       FeatureStdExtZkt,
                                       FeatureStdExtV,
                                       FeatureStdExtZifencei,
                                       FeatureStdExtZihintntl,
                                       FeatureStdExtZvl128b,
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 575-589: Definitions and supporting logic / 定义与支撑逻辑
```tablegen
                                       FeatureStdExtZvbb,
                                       FeatureStdExtZvknc,
                                       FeatureStdExtZvkng,
                                       FeatureStdExtZvksc,
                                       FeatureStdExtZvksg,
                                       FeatureUnalignedScalarMem,
                                       FeatureUnalignedVectorMem],
                                      [TuneNoDefaultUnroll,
                                       TuneConditionalCompressedMoveFusion,
                                       TuneLUIADDIFusion,
                                       TuneAUIPCADDIFusion,
                                       TuneNoSinkSplatOperands,
                                       TuneVXRMPipelineFlush,
                                       TunePostRAScheduler]>;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 590-613: TableGen record SIFIVE_P870 / TableGen 记录 SIFIVE_P870
```tablegen
def SIFIVE_P870 : RISCVProcessorModel<"sifive-p870", SiFiveP800Model,
                                      !listconcat(RVA23U64Features,
                                      [FeatureStdExtZama16b,
                                       FeatureStdExtZfh,
                                       FeatureStdExtZifencei,
                                       FeatureStdExtZkr,
                                       FeatureStdExtZvfbfmin,
                                       FeatureStdExtZvfbfwma,
                                       FeatureStdExtZvfh,
                                       FeatureStdExtZvknc,
                                       FeatureStdExtZvkng,
                                       FeatureStdExtZvksc,
                                       FeatureStdExtZvksg,
                                       FeatureStdExtZvl128b,
                                       FeatureUnalignedScalarMem,
                                       FeatureUnalignedVectorMem]),
                                      [TuneNoDefaultUnroll,
                                       TuneConditionalCompressedMoveFusion,
                                       TuneLUIADDIFusion,
                                       TuneAUIPCADDIFusion,
                                       TuneNoSinkSplatOperands,
                                       TuneVXRMPipelineFlush,
                                       TunePostRAScheduler]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 614-632: TableGen record SYNTACORE_SCR1_BASE / TableGen 记录 SYNTACORE_SCR1_BASE
```tablegen
def SYNTACORE_SCR1_BASE : RISCVProcessorModel<"syntacore-scr1-base",
                                              SyntacoreSCR1Model,
                                              [Feature32Bit,
                                               FeatureStdExtI,
                                               FeatureStdExtZicsr,
                                               FeatureStdExtZifencei,
                                               FeatureStdExtC],
                                              [TuneNoDefaultUnroll]>;

def SYNTACORE_SCR1_MAX : RISCVProcessorModel<"syntacore-scr1-max",
                                             SyntacoreSCR1Model,
                                             [Feature32Bit,
                                              FeatureStdExtI,
                                              FeatureStdExtZicsr,
                                              FeatureStdExtZifencei,
                                              FeatureStdExtM,
                                              FeatureStdExtC],
                                             [TuneNoDefaultUnroll]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 633-653: TableGen record SYNTACORE_SCR3_RV32 / TableGen 记录 SYNTACORE_SCR3_RV32
```tablegen
def SYNTACORE_SCR3_RV32 : RISCVProcessorModel<"syntacore-scr3-rv32",
                                              SyntacoreSCR3RV32Model,
                                              [Feature32Bit,
                                               FeatureStdExtI,
                                               FeatureStdExtZicsr,
                                               FeatureStdExtZifencei,
                                               FeatureStdExtM,
                                               FeatureStdExtC],
                                              [TuneNoDefaultUnroll, TunePostRAScheduler]>;

def SYNTACORE_SCR3_RV64 : RISCVProcessorModel<"syntacore-scr3-rv64",
                                              SyntacoreSCR3RV64Model,
                                              [Feature64Bit,
                                               FeatureStdExtI,
                                               FeatureStdExtZicsr,
                                               FeatureStdExtZifencei,
                                               FeatureStdExtM,
                                               FeatureStdExtA,
                                               FeatureStdExtC],
                                              [TuneNoDefaultUnroll, TunePostRAScheduler]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 654-678: TableGen record SYNTACORE_SCR4_RV32 / TableGen 记录 SYNTACORE_SCR4_RV32
```tablegen
def SYNTACORE_SCR4_RV32 : RISCVProcessorModel<"syntacore-scr4-rv32",
                                              SyntacoreSCR4RV32Model,
                                              [Feature32Bit,
                                               FeatureStdExtI,
                                               FeatureStdExtZicsr,
                                               FeatureStdExtZifencei,
                                               FeatureStdExtM,
                                               FeatureStdExtF,
                                               FeatureStdExtD,
                                               FeatureStdExtC],
                                              [TuneNoDefaultUnroll, TunePostRAScheduler]>;

def SYNTACORE_SCR4_RV64 : RISCVProcessorModel<"syntacore-scr4-rv64",
                                              SyntacoreSCR4RV64Model,
                                              [Feature64Bit,
                                               FeatureStdExtI,
                                               FeatureStdExtZicsr,
                                               FeatureStdExtZifencei,
                                               FeatureStdExtM,
                                               FeatureStdExtA,
                                               FeatureStdExtF,
                                               FeatureStdExtD,
                                               FeatureStdExtC],
                                              [TuneNoDefaultUnroll, TunePostRAScheduler]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 679-704: TableGen record SYNTACORE_SCR5_RV32 / TableGen 记录 SYNTACORE_SCR5_RV32
```tablegen
def SYNTACORE_SCR5_RV32 : RISCVProcessorModel<"syntacore-scr5-rv32",
                                              SyntacoreSCR5RV32Model,
                                              [Feature32Bit,
                                               FeatureStdExtI,
                                               FeatureStdExtZicsr,
                                               FeatureStdExtZifencei,
                                               FeatureStdExtM,
                                               FeatureStdExtA,
                                               FeatureStdExtF,
                                               FeatureStdExtD,
                                               FeatureStdExtC],
                                              [TuneNoDefaultUnroll, TunePostRAScheduler]>;

def SYNTACORE_SCR5_RV64 : RISCVProcessorModel<"syntacore-scr5-rv64",
                                              SyntacoreSCR5RV64Model,
                                              [Feature64Bit,
                                               FeatureStdExtI,
                                               FeatureStdExtZicsr,
                                               FeatureStdExtZifencei,
                                               FeatureStdExtM,
                                               FeatureStdExtA,
                                               FeatureStdExtF,
                                               FeatureStdExtD,
                                               FeatureStdExtC],
                                              [TuneNoDefaultUnroll, TunePostRAScheduler]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 705-723: TableGen record SYNTACORE_SCR7 / TableGen 记录 SYNTACORE_SCR7
```tablegen
def SYNTACORE_SCR7 : RISCVProcessorModel<"syntacore-scr7",
                                              SyntacoreSCR7Model,
                                              [Feature64Bit,
                                               FeatureStdExtI,
                                               FeatureStdExtZicsr,
                                               FeatureStdExtZifencei,
                                               FeatureStdExtM,
                                               FeatureStdExtA,
                                               FeatureStdExtF,
                                               FeatureStdExtD,
                                               FeatureStdExtC,
                                               FeatureStdExtV,
                                               FeatureStdExtZba,
                                               FeatureStdExtZbb,
                                               FeatureStdExtZbc,
                                               FeatureStdExtZbs,
                                               FeatureStdExtZkn],
                                              [TuneNoDefaultUnroll, TunePostRAScheduler]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 724-751: TableGen record TENSTORRENT_ASCALON_X / TableGen 记录 TENSTORRENT_ASCALON_X
```tablegen
def TENSTORRENT_ASCALON_X : RISCVProcessorModel<"tt-ascalon-x",
                                                TTAscalonXModel,
                                                !listconcat(RVA23S64Features,
                                                [FeatureStdExtSmaia,
                                                 FeatureStdExtSmmpm,
                                                 FeatureStdExtSmnpm,
                                                 FeatureStdExtSmrnmi,
                                                 FeatureStdExtSmstateen,
                                                 FeatureStdExtSsaia,
                                                 FeatureStdExtSsstrict,
                                                 FeatureStdExtZfbfmin,
                                                 FeatureStdExtZfh,
                                                 FeatureStdExtZkr,
                                                 FeatureStdExtZvbc,
                                                 FeatureStdExtZvfbfmin,
                                                 FeatureStdExtZvfbfwma,
                                                 FeatureStdExtZvfh,
                                                 FeatureStdExtZvkng,
                                                 FeatureStdExtZvl256b,
                                                 FeatureUnalignedScalarMem,
                                                 FeatureUnalignedVectorMem]),
                                                [TuneNoDefaultUnroll,
                                                 TuneNLogNVRGather,
                                                 TuneOptimizedNF2SegmentLoadStore,
                                                 TuneOptimizedNF3SegmentLoadStore,
                                                 TuneOptimizedNF4SegmentLoadStore,
                                                 TuneOptimizedNF5SegmentLoadStore,
                                                 TuneOptimizedNF6SegmentLoadStore,
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 752-779: TableGen record VENTANA_VEYRON_V1 / TableGen 记录 VENTANA_VEYRON_V1
```tablegen
                                                 TuneOptimizedNF7SegmentLoadStore,
                                                 TuneOptimizedNF8SegmentLoadStore,
                                                 TuneOptimizedZeroStrideLoad,
                                                 TunePostRAScheduler]>;

def VENTANA_VEYRON_V1 : RISCVProcessorModel<"veyron-v1",
                                            NoSchedModel,
                                            [Feature64Bit,
                                             FeatureStdExtI,
                                             FeatureStdExtZifencei,
                                             FeatureStdExtZicsr,
                                             FeatureStdExtZicntr,
                                             FeatureStdExtZihpm,
                                             FeatureStdExtZihintpause,
                                             FeatureStdExtM,
                                             FeatureStdExtA,
                                             FeatureStdExtF,
                                             FeatureStdExtD,
                                             FeatureStdExtC,
                                             FeatureStdExtZba,
                                             FeatureStdExtZbb,
                                             FeatureStdExtZbc,
                                             FeatureStdExtZbs,
                                             FeatureStdExtZicbom,
                                             FeatureStdExtZicbop,
                                             FeatureStdExtZicboz,
                                             FeatureVendorXVentanaCondOps],
                                             [TuneVentanaVeyron,
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 780-794: TableGen field customization and iteration / TableGen 字段定制与迭代
```tablegen
                                              TuneDisableMISchedLoadClustering,
                                              TuneDisablePostMISchedLoadClustering,
                                              TuneDisablePostMISchedStoreClustering,
                                              TuneLUIADDIFusion,
                                              TuneAUIPCADDIFusion,
                                              TuneZExtHFusion,
                                              TuneZExtWFusion,
                                              TuneShiftedZExtWFusion,
                                              TuneADDLoadFusion,
                                              TuneAUIPCLoadFusion,
                                              TuneLUILoadFusion]> {
  let MVendorID = 0x61f;
  let MArchID = 0x8000000000010000;
  let MImpID = 0x111;
}
```
**EN:** This block adjusts inherited fields or iterates over parameter sets to shape many related records consistently.

**CN:** 该区段调整继承字段或遍历参数集合，以一致地塑造多条相关记录。

### Lines 795-821: TableGen record XIANGSHAN_NANHU / TableGen 记录 XIANGSHAN_NANHU
```tablegen

def XIANGSHAN_NANHU : RISCVProcessorModel<"xiangshan-nanhu",
                                          XiangShanNanHuModel,
                                          [Feature64Bit,
                                           FeatureStdExtI,
                                           FeatureStdExtZicsr,
                                           FeatureStdExtZifencei,
                                           FeatureStdExtM,
                                           FeatureStdExtA,
                                           FeatureStdExtF,
                                           FeatureStdExtD,
                                           FeatureStdExtC,
                                           FeatureStdExtZba,
                                           FeatureStdExtZbb,
                                           FeatureStdExtZbc,
                                           FeatureStdExtZbs,
                                           FeatureStdExtZkn,
                                           FeatureStdExtZksed,
                                           FeatureStdExtZksh,
                                           FeatureStdExtSvinval,
                                           FeatureStdExtZicbom,
                                           FeatureStdExtZicboz],
                                           [TuneNoDefaultUnroll,
                                            TuneZExtHFusion,
                                            TuneZExtWFusion,
                                            TuneShiftedZExtWFusion]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 822-848: TableGen record XIANGSHAN_KUNMINGHU / TableGen 记录 XIANGSHAN_KUNMINGHU
```tablegen
def XIANGSHAN_KUNMINGHU : RISCVProcessorModel<"xiangshan-kunminghu",
                                              NoSchedModel,
                                              !listconcat(RVA23S64Features,
                                              [FeatureStdExtZacas,
                                               FeatureStdExtZbc,
                                               FeatureStdExtZfh,
                                               FeatureStdExtZkn,
                                               FeatureStdExtZks,
                                               FeatureStdExtZvfh,
                                               FeatureStdExtSmaia,
                                               FeatureStdExtSmcsrind,
                                               FeatureStdExtSmdbltrp,
                                               FeatureStdExtSmmpm,
                                               FeatureStdExtSmnpm,
                                               FeatureStdExtSmrnmi,
                                               FeatureStdExtSmstateen,
                                               FeatureStdExtSsaia,
                                               FeatureStdExtSscsrind,
                                               FeatureStdExtSsdbltrp,
                                               FeatureStdExtSspm,
                                               FeatureStdExtSsstrict,
                                               FeatureStdExtZvl128b]),
                                              [TuneNoDefaultUnroll,
                                               TuneZExtHFusion,
                                               TuneZExtWFusion,
                                               TuneShiftedZExtWFusion]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 849-876: TableGen record SPACEMIT_A100 / TableGen 记录 SPACEMIT_A100
```tablegen
def SPACEMIT_A100 : RISCVProcessorModel<"spacemit-a100",
                                        SpacemitX60Model,
                                        !listconcat(
                                        !listremove(RVA23S64Features,
                                        [FeatureStdExtSha]),
                                        [FeatureStdExtSmepmp,
                                         FeatureStdExtSmnpm,
                                         FeatureStdExtSmstateen,
                                         FeatureStdExtSspm,
                                         FeatureStdExtZbc,
                                         FeatureStdExtZbkc,
                                         FeatureStdExtZfbfmin,
                                         FeatureStdExtZfh,
                                         FeatureStdExtZvbc,
                                         FeatureStdExtZvfbfwma,
                                         FeatureStdExtZvfh,
                                         FeatureStdExtZvkng,
                                         FeatureStdExtZvknha,
                                         FeatureStdExtZvksc,
                                         FeatureStdExtZvksg,
                                         FeatureStdExtZvl1024b,
                                         FeatureUnalignedScalarMem]),
                                        [TuneDLenFactor2,
                                         TuneOptimizedNF2SegmentLoadStore,
                                         TuneOptimizedNF3SegmentLoadStore,
                                         TuneOptimizedNF4SegmentLoadStore,
                                         TuneVXRMPipelineFlush]> {
  let MVendorID = 0x710;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 877-904: TableGen record SPACEMIT_X60 / TableGen 记录 SPACEMIT_X60
```tablegen
  let MArchID = 0x8000000041000002;
  let MImpID =  0x10000000d5686200;
}

def SPACEMIT_X60 : RISCVProcessorModel<"spacemit-x60",
                                       SpacemitX60Model,
                                       !listconcat(RVA22S64Features,
                                       [FeatureStdExtV,
                                        FeatureStdExtSscofpmf,
                                        FeatureStdExtSstc,
                                        FeatureStdExtSvnapot,
                                        FeatureStdExtZbc,
                                        FeatureStdExtZbkc,
                                        FeatureStdExtZfh,
                                        FeatureStdExtZicond,
                                        FeatureStdExtZvfh,
                                        FeatureStdExtZvkt,
                                        FeatureStdExtZvl256b,
                                        FeatureVendorXSMTVDot,
                                        FeatureUnalignedScalarMem]),
                                       [TuneDLenFactor2,
                                        TuneOptimizedNF2SegmentLoadStore,
                                        TuneOptimizedNF3SegmentLoadStore,
                                        TuneOptimizedNF4SegmentLoadStore,
                                        TuneEnableSelectOptimize,
                                        TuneVXRMPipelineFlush,
                                        TunePreferAscendingLoadStore]> {
  let MVendorID = 0x710;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 905-932: TableGen record SPACEMIT_X100 / TableGen 记录 SPACEMIT_X100
```tablegen
  let MArchID = 0x8000000058000001;
  let MImpID = 0x1000000049772200;
}

def SPACEMIT_X100 : RISCVProcessorModel<"spacemit-x100",
                                        SpacemitX100Model,
                                        !listconcat(RVA23S64Features,
                                        [FeatureStdExtSmepmp,
                                         FeatureStdExtSmnpm,
                                         FeatureStdExtSmstateen,
                                         FeatureStdExtSspm,
                                         FeatureStdExtZbc,
                                         FeatureStdExtZbkc,
                                         FeatureStdExtZfbfmin,
                                         FeatureStdExtZfh,
                                         FeatureStdExtZvbc,
                                         FeatureStdExtZvfbfwma,
                                         FeatureStdExtZvfh,
                                         FeatureStdExtZvkng,
                                         FeatureStdExtZvknha,
                                         FeatureStdExtZvksc,
                                         FeatureStdExtZvksg,
                                         FeatureStdExtZvl256b,
                                         FeatureVendorXSMTVDot,
                                         FeatureUnalignedScalarMem]),
                                        [TuneDLenFactor2,
                                         TuneOptimizedNF2SegmentLoadStore,
                                         TuneOptimizedNF3SegmentLoadStore,
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 933-960: TableGen record RP2350_HAZARD3 / TableGen 记录 RP2350_HAZARD3
```tablegen
                                         TuneOptimizedNF4SegmentLoadStore,
                                         TuneVXRMPipelineFlush,
                                         TuneFusionLogicRegReg,
                                         TuneFusionLogicRegImm,
                                         TuneFusionLogicImmReg,
                                         TuneFusionMulAdd,
                                         TuneFusionAddMem,
                                         TuneFusionShiftBitExtract]> {
  let MVendorID = 0x710;
  let MArchID = 0x8000000058000002;
  let MImpID = 0x33d8a600;
}

def RP2350_HAZARD3 : RISCVProcessorModel<"rp2350-hazard3",
                                         NoSchedModel,
                                         [Feature32Bit,
                                          FeatureStdExtI,
                                          FeatureStdExtM,
                                          FeatureStdExtA,
                                          FeatureStdExtC,
                                          FeatureStdExtZicsr,
                                          FeatureStdExtZifencei,
                                          FeatureStdExtZba,
                                          FeatureStdExtZbb,
                                          FeatureStdExtZbs,
                                          FeatureStdExtZbkb,
                                          FeatureStdExtZcb,
                                          FeatureStdExtZcmp]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 961-974: TableGen record ANDES_A25 / TableGen 记录 ANDES_A25
```tablegen

def ANDES_A25 : RISCVProcessorModel<"andes-a25",
                                    NoSchedModel,
                                    [Feature32Bit,
                                     FeatureStdExtI,
                                     FeatureStdExtZicsr,
                                     FeatureStdExtZifencei,
                                     FeatureStdExtM,
                                     FeatureStdExtA,
                                     FeatureStdExtF,
                                     FeatureStdExtD,
                                     FeatureStdExtC,
                                     FeatureVendorXAndesPerf]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 975-992: TableGen record ANDES_AX25 / TableGen 记录 ANDES_AX25
```tablegen
def ANDES_AX25 : RISCVProcessorModel<"andes-ax25",
                                     NoSchedModel,
                                     [Feature64Bit,
                                      FeatureStdExtI,
                                      FeatureStdExtZicsr,
                                      FeatureStdExtZifencei,
                                      FeatureStdExtM,
                                      FeatureStdExtA,
                                      FeatureStdExtF,
                                      FeatureStdExtD,
                                      FeatureStdExtC,
                                      FeatureVendorXAndesPerf]>;

defvar Andes45TuneFeatures = [TuneAndes45,
                              TuneNoDefaultUnroll,
                              TuneShortForwardBranchIALU,
                              TunePostRAScheduler];
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 993-1009: TableGen record ANDES_45 / TableGen 记录 ANDES_45
```tablegen
def ANDES_45 : RISCVTuneProcessorModel<"andes-45-series",
                                       Andes45Model, Andes45TuneFeatures>;

def ANDES_N45 : RISCVProcessorModel<"andes-n45",
                                    Andes45Model,
                                    [Feature32Bit,
                                     FeatureStdExtI,
                                     FeatureStdExtZicsr,
                                     FeatureStdExtZifencei,
                                     FeatureStdExtM,
                                     FeatureStdExtA,
                                     FeatureStdExtF,
                                     FeatureStdExtD,
                                     FeatureStdExtC,
                                     FeatureVendorXAndesPerf],
                                    Andes45TuneFeatures>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 1010-1023: TableGen record ANDES_NX45 / TableGen 记录 ANDES_NX45
```tablegen
def ANDES_NX45 : RISCVProcessorModel<"andes-nx45",
                                     Andes45Model,
                                     [Feature64Bit,
                                      FeatureStdExtI,
                                      FeatureStdExtZicsr,
                                      FeatureStdExtZifencei,
                                      FeatureStdExtM,
                                      FeatureStdExtA,
                                      FeatureStdExtF,
                                      FeatureStdExtD,
                                      FeatureStdExtC,
                                      FeatureVendorXAndesPerf],
                                     Andes45TuneFeatures>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 1024-1037: TableGen record ANDES_A45 / TableGen 记录 ANDES_A45
```tablegen
def ANDES_A45 : RISCVProcessorModel<"andes-a45",
                                    Andes45Model,
                                    [Feature32Bit,
                                     FeatureStdExtI,
                                     FeatureStdExtZicsr,
                                     FeatureStdExtZifencei,
                                     FeatureStdExtM,
                                     FeatureStdExtA,
                                     FeatureStdExtF,
                                     FeatureStdExtD,
                                     FeatureStdExtC,
                                     FeatureVendorXAndesPerf],
                                    Andes45TuneFeatures>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 1038-1051: TableGen record ANDES_AX45 / TableGen 记录 ANDES_AX45
```tablegen
def ANDES_AX45 : RISCVProcessorModel<"andes-ax45",
                                     Andes45Model,
                                     [Feature64Bit,
                                      FeatureStdExtI,
                                      FeatureStdExtZicsr,
                                      FeatureStdExtZifencei,
                                      FeatureStdExtM,
                                      FeatureStdExtA,
                                      FeatureStdExtF,
                                      FeatureStdExtD,
                                      FeatureStdExtC,
                                      FeatureVendorXAndesPerf],
                                     Andes45TuneFeatures>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 1052-1066: TableGen record ANDES_AX45MPV / TableGen 记录 ANDES_AX45MPV
```tablegen
def ANDES_AX45MPV : RISCVProcessorModel<"andes-ax45mpv",
                                        Andes45Model,
                                        [Feature64Bit,
                                         FeatureStdExtI,
                                         FeatureStdExtZicsr,
                                         FeatureStdExtZifencei,
                                         FeatureStdExtM,
                                         FeatureStdExtA,
                                         FeatureStdExtF,
                                         FeatureStdExtD,
                                         FeatureStdExtC,
                                         FeatureStdExtV,
                                         FeatureVendorXAndesPerf],
                                        Andes45TuneFeatures>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 1067-1082: TableGen record ESPERANTO_SOC1 / TableGen 记录 ESPERANTO_SOC1
```tablegen
def ESPERANTO_SOC1 : RISCVProcessorModel<"et-soc1",
                                         NoSchedModel,
                                         [Feature64Bit,
                                          FeatureStdExtM,
                                          FeatureStdExtF,
                                          FeatureStdExtC,
                                          FeatureVendorXAIFET]>;

def AINEKKO_ERBIUM : RISCVProcessorModel<"an-erbium",
                                         NoSchedModel,
                                         [Feature64Bit,
                                          FeatureStdExtM,
                                          FeatureStdExtF,
                                          FeatureStdExtC,
                                          FeatureVendorXAIFET]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 1083-1104: TableGen record XUANTIE_C910V2 / TableGen 记录 XUANTIE_C910V2
```tablegen
def XUANTIE_C910V2 : RISCVProcessorModel<"xt-c910v2",
                                         GenericOOOModel,
                                         !listconcat(RVB23S64Features,
                                         [FeatureStdExtZfbfmin,
                                          FeatureStdExtZfh,
                                          FeatureStdExtZca,
                                          FeatureStdExtZcd,
                                          FeatureStdExtZbc,
                                          FeatureVendorXTHeadBa,
                                          FeatureVendorXTHeadBb,
                                          FeatureVendorXTHeadBs,
                                          FeatureVendorXTHeadCmo,
                                          FeatureVendorXTHeadCondMov,
                                          FeatureVendorXTHeadFMemIdx,
                                          FeatureVendorXTHeadMac,
                                          FeatureVendorXTHeadMemIdx,
                                          FeatureVendorXTHeadMemPair,
                                          FeatureVendorXTHeadSync,
                                          FeatureUnalignedScalarMem]),
                                         [TuneVXRMPipelineFlush,
                                          TunePostRAScheduler]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 1105-1132: TableGen record XUANTIE_C920V2 / TableGen 记录 XUANTIE_C920V2
```tablegen
def XUANTIE_C920V2 : RISCVProcessorModel<"xt-c920v2",
                                         GenericOOOModel,
                                         !listconcat(
                                           !listremove(RVB23S64Features, [FeatureStdExtZicclsm]),
                                           [FeatureStdExtV,
                                            FeatureStdExtZfbfmin,
                                            FeatureStdExtZfh,
                                            FeatureStdExtZca,
                                            FeatureStdExtZcd,
                                            FeatureStdExtZbc,
                                            FeatureStdExtZvfbfmin,
                                            FeatureStdExtZvfbfwma,
                                            FeatureStdExtZvfh,
                                            FeatureVendorXTHeadBa,
                                            FeatureVendorXTHeadBb,
                                            FeatureVendorXTHeadBs,
                                            FeatureVendorXTHeadCmo,
                                            FeatureVendorXTHeadCondMov,
                                            FeatureVendorXTHeadFMemIdx,
                                            FeatureVendorXTHeadMac,
                                            FeatureVendorXTHeadMemIdx,
                                            FeatureVendorXTHeadMemPair,
                                            FeatureVendorXTHeadSync,
                                            FeatureVendorXTHeadVdot,
                                            FeatureUnalignedScalarMem]),
                                         [TuneVXRMPipelineFlush,
                                          TunePostRAScheduler]>;
// clang-format on
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

## Key Concepts / 关键概念
- **Processor models** / **处理器模型**
- **TableGen DSL** / **TableGen DSL**

## Dependencies / 依赖关系
- No explicit source includes in this file; dependencies are mostly implicit through surrounding backend integration. / 该文件没有显式源码级 include，依赖主要通过周边后端集成隐式体现。
