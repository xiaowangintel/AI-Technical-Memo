# AArch64FMV.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/AArch64/AArch64FMV.td`
- **Repository**: llvm/llvm-project
- **Purpose**: This file covers Describe AArch64 FMV Features. It is written in the TableGen DSL. / 该文件实现 AArch64 后端中的AArch64 后端逻辑。 它使用 TableGen DSL 描述目标规则。
- **Note**: Written in the TableGen DSL / 使用 TableGen DSL 编写
## Line-by-Line Analysis / 逐行分析
### Lines 1-18: Documented TableGen section
```tablegen
//=------ AArch64FMV.td - Describe AArch64 FMV Features ------*- tablegen -*-=//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Function MultiVersioning (FMV) properties. FMV features are accepted by the
// attributes target_version and target_clones, and they correspond to a mapping
// from the FMV feature name to:
//  - A bit in the FMV ABI, as defined by the ACLE.
//  - The FMV priority, as defined by the ACLE.
//  - A list of backend features.
// 
// The list of backend features is not a set of dependencies; it is specific to
// LLVM and indicates how to do codegen when the FMV feature is present.
// 
```
**EN:** This comment block marks a new TableGen section and explains the intent of the records that follow.  
**CN:** 该注释块标记了新的 TableGen 章节，并说明后续记录的设计意图。
### Lines 19-34: Documented TableGen section
```tablegen
// Therefore FMVExtensions are separated from regular AArch64 Extensions, which
// encode dependencies between themselves and other SubtargetFeatures.
//===----------------------------------------------------------------------===//


// Something you can add to target_version or target_clones.
class FMVExtension<string name, string enumeration> {
    // Name, as spelled in target_version or target_clones. e.g. "memtag".
    string Name = name;

    // A C++ expression giving the number of the bit in the FMV ABI.
    // Currently this is given as a value from the enum "CPUFeatures".
    string FeatureBit = "FEAT_" # enumeration;

    // SubtargetFeature enabled for codegen when this FMV feature is present.
    string BackendFeature = name;
```
**EN:** This comment block marks a new TableGen section and explains the intent of the records that follow.  
**CN:** 该注释块标记了新的 TableGen 章节，并说明后续记录的设计意图。
### Lines 35-51: Documented TableGen section
```tablegen

    // A C++ expression giving the number of the priority bit.
    // Currently this is given as a value from the enum "FeatPriorities".
    string PriorityBit = "PRIOR_" # enumeration;
}

def : FMVExtension<"aes", "PMULL">;
def : FMVExtension<"bf16", "BF16">;
def : FMVExtension<"bti", "BTI">;
def : FMVExtension<"crc", "CRC">;
def : FMVExtension<"dit", "DIT">;
def : FMVExtension<"dotprod", "DOTPROD">;
let BackendFeature = "ccpp" in def : FMVExtension<"dpb", "DPB">;
let BackendFeature = "ccdp" in def : FMVExtension<"dpb2", "DPB2">;
def : FMVExtension<"f32mm", "SVE_F32MM">;
def : FMVExtension<"f64mm", "SVE_F64MM">;
def : FMVExtension<"fcma", "FCMA">;
```
**EN:** This comment block marks a new TableGen section and explains the intent of the records that follow.  
**CN:** 该注释块标记了新的 TableGen 章节，并说明后续记录的设计意图。
### Lines 52-68: TableGen definition
```tablegen
def : FMVExtension<"flagm", "FLAGM">;
let BackendFeature = "altnzcv" in def : FMVExtension<"flagm2", "FLAGM2">;
def : FMVExtension<"fp", "FP">;
def : FMVExtension<"fp16", "FP16">;
def : FMVExtension<"fp16fml", "FP16FML">;
let BackendFeature = "fptoint" in def : FMVExtension<"frintts", "FRINTTS">;
def : FMVExtension<"i8mm", "I8MM">;
def : FMVExtension<"jscvt", "JSCVT">;
def : FMVExtension<"lse", "LSE">;
def : FMVExtension<"memtag", "MEMTAG2">;
def : FMVExtension<"mops", "MOPS">;
def : FMVExtension<"rcpc", "RCPC">;
let BackendFeature = "rcpc-immo" in def : FMVExtension<"rcpc2", "RCPC2">;
def : FMVExtension<"rcpc3", "RCPC3">;
def : FMVExtension<"rdm", "RDM">;
def : FMVExtension<"rng", "RNG">;
def : FMVExtension<"sb", "SB">;
```
**EN:** This definition materializes this definition as a concrete backend record that TableGen can later turn into generated matchers, encoders, or metadata.  
**CN:** 该定义将 this definition 实例化为具体后端记录，后续 TableGen 可据此生成匹配器、编码器或元数据。
### Lines 69-85: TableGen definition
```tablegen
def : FMVExtension<"sha2", "SHA2">;
def : FMVExtension<"sha3", "SHA3">;
def : FMVExtension<"simd", "SIMD">;
def : FMVExtension<"sm4", "SM4">;
def : FMVExtension<"sme", "SME">;
def : FMVExtension<"sme-f64f64", "SME_F64">;
def : FMVExtension<"sme-i16i64", "SME_I64">;
def : FMVExtension<"sme2", "SME2">;
def : FMVExtension<"ssbs", "SSBS2">;
def : FMVExtension<"sve", "SVE">;
def : FMVExtension<"sve2", "SVE2">;
def : FMVExtension<"sve2-aes", "SVE_PMULL128">;
def : FMVExtension<"sve2-bitperm", "SVE_BITPERM">;
def : FMVExtension<"sve2-sha3", "SVE_SHA3">;
def : FMVExtension<"sve2-sm4", "SVE_SM4">;
def : FMVExtension<"wfxt", "WFXT">;
def : FMVExtension<"cssc", "CSSC">;
```
**EN:** This definition materializes this definition as a concrete backend record that TableGen can later turn into generated matchers, encoders, or metadata.  
**CN:** 该定义将 this definition 实例化为具体后端记录，后续 TableGen 可据此生成匹配器、编码器或元数据。
### Lines 86-96: TableGen definition
```tablegen

// Extensions which allow the user to override version priority.
// 8-bits allow 256-1 priority levels (excluding all zeros).
def : FMVExtension<"P0", "P0">;
def : FMVExtension<"P1", "P1">;
def : FMVExtension<"P2", "P2">;
def : FMVExtension<"P3", "P3">;
def : FMVExtension<"P4", "P4">;
def : FMVExtension<"P5", "P5">;
def : FMVExtension<"P6", "P6">;
def : FMVExtension<"P7", "P7">;
```
**EN:** This definition materializes this definition as a concrete backend record that TableGen can later turn into generated matchers, encoders, or metadata.  
**CN:** 该定义将 this definition 实例化为具体后端记录，后续 TableGen 可据此生成匹配器、编码器或元数据。
## Key Concepts / 关键概念
- **EN:** AArch64 target backend structure **CN:** AArch64 目标后端结构
- **EN:** Declarative backend description in TableGen DSL **CN:** 使用 TableGen DSL 的声明式后端描述
## Dependencies / 依赖关系
- **EN:** Consumed by LLVM TableGen to produce generated matchers, encoders, register info, or scheduling tables. **CN:** 该文件会被 LLVM TableGen 消费，用于生成匹配器、编码器、寄存器信息或调度表。
- **EN:** Closely connected with neighboring AArch64 backend components responsible for AArch64 backend logic. **CN:** 与周边负责AArch64 后端逻辑的 AArch64 后端组件紧密协作。
