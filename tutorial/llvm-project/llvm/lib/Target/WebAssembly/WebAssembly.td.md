# WebAssembly.td — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/WebAssembly/WebAssembly.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This is a target description file for the WebAssembly architecture, which is also known as "wasm". It is written in LLVM TableGen DSL and feeds generated backend tables.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/WebAssembly/WebAssembly.td`，主要负责 WebAssembly 后端的该后端的 TableGen 声明。 它使用 LLVM TableGen DSL 编写，用于描述声明式目标信息并驱动生成代码。
- **Language**: TableGen DSL / TableGen 领域特定语言

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```tablegen
//- WebAssembly.td - Describe the WebAssembly Target Machine --*- tablegen -*-//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-12

```tablegen
//===----------------------------------------------------------------------===//
///
/// \file
/// This is a target description file for the WebAssembly architecture,
/// which is also known as "wasm".
///
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "\file".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“\file”。

### Lines 13-14

```tablegen
//===----------------------------------------------------------------------===//
```
- **EN**: Continues the WebAssembly backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 WebAssembly 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 15-16

```tablegen
//===----------------------------------------------------------------------===//
// Target-independent interfaces which we are implementing
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Target-independent interfaces which we are implementing".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Target-independent interfaces which we are implementing”。

### Lines 17-20

```tablegen
//===----------------------------------------------------------------------===//

include "llvm/Target/Target.td"
```
- **EN**: Pulls in direct dependencies required by this TableGen declarations for the backend, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该该后端的 TableGen 声明所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 21-22

```tablegen
//===----------------------------------------------------------------------===//
// WebAssembly Subtarget features.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "WebAssembly Subtarget features.". Subtarget feature gating influences the behavior here.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“WebAssembly Subtarget features.”。 子目标特性裁剪会影响这里的行为。

### Lines 23-31

```tablegen
//===----------------------------------------------------------------------===//

def FeatureAtomics : SubtargetFeature<"atomics", "HasAtomics", "true",
                                      "Enable Atomics">;

def FeatureBulkMemory :
      SubtargetFeature<"bulk-memory", "HasBulkMemory", "true",
                       "Enable bulk memory operations">;
```
- **EN**: Defines TableGen subtarget feature flags for WebAssembly, allowing CPU or feature strings to toggle generated behavior. Subtarget feature gating influences the behavior here.
- **CN**: 这里为 WebAssembly 定义 TableGen 子目标特性标志，使 CPU 或 feature 字符串能够切换生成出来的行为。 子目标特性裁剪会影响这里的行为。

### Lines 32-39

```tablegen
def FeatureBulkMemoryOpt :
      SubtargetFeature<"bulk-memory-opt", "HasBulkMemoryOpt", "true",
                       "Enable bulk memory optimization operations">;

def FeatureCallIndirectOverlong :
      SubtargetFeature<"call-indirect-overlong", "HasCallIndirectOverlong", "true",
                       "Enable overlong encoding for call_indirect immediates">;
```
- **EN**: Defines TableGen subtarget feature flags for WebAssembly, allowing CPU or feature strings to toggle generated behavior. Subtarget feature gating influences the behavior here.
- **CN**: 这里为 WebAssembly 定义 TableGen 子目标特性标志，使 CPU 或 feature 字符串能够切换生成出来的行为。 子目标特性裁剪会影响这里的行为。

### Lines 40-47

```tablegen
def FeatureExceptionHandling :
      SubtargetFeature<"exception-handling", "HasExceptionHandling", "true",
                       "Enable Wasm exception handling">;

def FeatureExtendedConst :
      SubtargetFeature<"extended-const", "HasExtendedConst", "true",
                       "Enable extended const expressions">;
```
- **EN**: Defines TableGen subtarget feature flags for WebAssembly, allowing CPU or feature strings to toggle generated behavior. Subtarget feature gating influences the behavior here.
- **CN**: 这里为 WebAssembly 定义 TableGen 子目标特性标志，使 CPU 或 feature 字符串能够切换生成出来的行为。 子目标特性裁剪会影响这里的行为。

### Lines 48-57

```tablegen
def FeatureFP16 :
      SubtargetFeature<"fp16", "HasFP16", "true",
                       "Enable FP16 instructions">;

def FeatureGC : SubtargetFeature<"gc", "HasGC", "true", "Enable wasm gc">;

def FeatureMultiMemory :
      SubtargetFeature<"multimemory", "HasMultiMemory", "true",
                       "Enable multiple memories">;
```
- **EN**: Defines TableGen subtarget feature flags for WebAssembly, allowing CPU or feature strings to toggle generated behavior. Subtarget feature gating influences the behavior here.
- **CN**: 这里为 WebAssembly 定义 TableGen 子目标特性标志，使 CPU 或 feature 字符串能够切换生成出来的行为。 子目标特性裁剪会影响这里的行为。

### Lines 58-66

```tablegen
def FeatureMultivalue :
      SubtargetFeature<"multivalue",
                       "HasMultivalue", "true",
                       "Enable multivalue blocks, instructions, and functions">;

def FeatureMutableGlobals :
      SubtargetFeature<"mutable-globals", "HasMutableGlobals", "true",
                       "Enable mutable globals">;
```
- **EN**: Defines TableGen subtarget feature flags for WebAssembly, allowing CPU or feature strings to toggle generated behavior. Subtarget feature gating influences the behavior here.
- **CN**: 这里为 WebAssembly 定义 TableGen 子目标特性标志，使 CPU 或 feature 字符串能够切换生成出来的行为。 子目标特性裁剪会影响这里的行为。

### Lines 67-75

```tablegen
def FeatureNontrappingFPToInt :
      SubtargetFeature<"nontrapping-fptoint",
                       "HasNontrappingFPToInt", "true",
                       "Enable non-trapping float-to-int conversion operators">;

def FeatureReferenceTypes :
      SubtargetFeature<"reference-types", "HasReferenceTypes", "true",
                       "Enable reference types">;
```
- **EN**: Defines TableGen subtarget feature flags for WebAssembly, allowing CPU or feature strings to toggle generated behavior. Subtarget feature gating influences the behavior here.
- **CN**: 这里为 WebAssembly 定义 TableGen 子目标特性标志，使 CPU 或 feature 字符串能够切换生成出来的行为。 子目标特性裁剪会影响这里的行为。

### Lines 76-83

```tablegen
def FeatureRelaxedAtomics :
      SubtargetFeature<"relaxed-atomics", "HasRelaxedAtomics", "true",
                       "Enable relaxed-atomics proposal">;

def FeatureRelaxedSIMD :
      SubtargetFeature<"relaxed-simd", "SIMDLevel", "RelaxedSIMD",
                       "Enable relaxed-simd instructions">;
```
- **EN**: Defines TableGen subtarget feature flags for WebAssembly, allowing CPU or feature strings to toggle generated behavior. Subtarget feature gating influences the behavior here.
- **CN**: 这里为 WebAssembly 定义 TableGen 子目标特性标志，使 CPU 或 feature 字符串能够切换生成出来的行为。 子目标特性裁剪会影响这里的行为。

### Lines 84-90

```tablegen
def FeatureSignExt :
      SubtargetFeature<"sign-ext", "HasSignExt", "true",
                       "Enable sign extension operators">;

def FeatureSIMD128 : SubtargetFeature<"simd128", "SIMDLevel", "SIMD128",
                                      "Enable 128-bit SIMD">;
```
- **EN**: Defines TableGen subtarget feature flags for WebAssembly, allowing CPU or feature strings to toggle generated behavior. Subtarget feature gating influences the behavior here.
- **CN**: 这里为 WebAssembly 定义 TableGen 子目标特性标志，使 CPU 或 feature 字符串能够切换生成出来的行为。 子目标特性裁剪会影响这里的行为。

### Lines 91-98

```tablegen
def FeatureTailCall :
      SubtargetFeature<"tail-call", "HasTailCall", "true",
                       "Enable tail call instructions">;

def FeatureWideArithmetic :
      SubtargetFeature<"wide-arithmetic", "HasWideArithmetic", "true",
                       "Enable wide-arithmetic instructions">;
```
- **EN**: Defines TableGen subtarget feature flags for WebAssembly, allowing CPU or feature strings to toggle generated behavior. Subtarget feature gating influences the behavior here.
- **CN**: 这里为 WebAssembly 定义 TableGen 子目标特性标志，使 CPU 或 feature 字符串能够切换生成出来的行为。 子目标特性裁剪会影响这里的行为。

### Lines 99-100

```tablegen
//===----------------------------------------------------------------------===//
// Architectures.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Architectures.".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Architectures.”。

### Lines 101-102

```tablegen
//===----------------------------------------------------------------------===//
```
- **EN**: Continues the WebAssembly backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 WebAssembly 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 103-104

```tablegen
//===----------------------------------------------------------------------===//
// Register File Description
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Register File Description".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Register File Description”。

### Lines 105-109

```tablegen
//===----------------------------------------------------------------------===//

include "WebAssemblyRegisterInfo.td"
include "WebAssemblyRegisterBanks.td"
```
- **EN**: Pulls in direct dependencies required by this TableGen declarations for the backend, so later declarations can reuse LLVM infrastructure and target-specific helpers. Register assignment and register-class constraints matter here.
- **CN**: 这一段引入该该后端的 TableGen 声明所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 110-111

```tablegen
//===----------------------------------------------------------------------===//
// Instruction Descriptions
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Instruction Descriptions".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Instruction Descriptions”。

### Lines 112-121

```tablegen
//===----------------------------------------------------------------------===//

include "WebAssemblyInstrInfo.td"

def WASM64 : HwMode<[HasAddr64]>;

def wasm_ptr_rc : RegClassByHwMode<
  [DefaultMode, WASM64],
  [I32, I64]>;
```
- **EN**: Pulls in direct dependencies required by this TableGen declarations for the backend, so later declarations can reuse LLVM infrastructure and target-specific helpers. Notable symbols in this range include `WASM64`, `wasm_ptr_rc`.
- **CN**: 这一段引入该该后端的 TableGen 声明所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 该区间中较显眼的符号包括 `WASM64`, `wasm_ptr_rc`。

### Lines 122-125

```tablegen
defm : RemapAllTargetPseudoPointerOperands<wasm_ptr_rc>;

def WebAssemblyInstrInfo : InstrInfo;
```
- **EN**: Adds declarative TableGen records such as `WebAssemblyInstrInfo` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `WebAssemblyInstrInfo`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 126-127

```tablegen
//===----------------------------------------------------------------------===//
// WebAssembly Processors supported.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "WebAssembly Processors supported.".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“WebAssembly Processors supported.”。

### Lines 128-138

```tablegen
//===----------------------------------------------------------------------===//

// Minimal Viable Product.
def : ProcessorModel<"mvp", NoSchedModel, []>;

// Generic processor: latest stable version.
//
// This includes features that have achieved phase 4 of the standards process,
// and that are expected to work for most users in the current time, with
// consideration given to available support in relevant engines and tools, and
// the importance of the features.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Minimal Viable Product.". Scheduling or processor-model metadata is part of this section.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Minimal Viable Product.”。 这一段包含调度或处理器模型元数据。

### Lines 139-145

```tablegen
def : ProcessorModel<"generic", NoSchedModel,
                      [FeatureBulkMemory, FeatureBulkMemoryOpt,
                       FeatureCallIndirectOverlong, FeatureMultivalue,
                       FeatureMutableGlobals, FeatureNontrappingFPToInt,
                       FeatureReferenceTypes, FeatureSignExt]>;

// Lime1: <https://github.com/WebAssembly/tool-conventions/blob/main/Lime.md#lime1>
```
- **EN**: Lists processor models and binds the default feature bundles or scheduling data attached to each model. Subtarget feature gating influences the behavior here.
- **CN**: 这一段列出处理器模型，并把默认特性集合或调度数据绑定到相应模型上。 子目标特性裁剪会影响这里的行为。

### Lines 146-152

```tablegen
def : ProcessorModel<"lime1", NoSchedModel,
                      [FeatureBulkMemoryOpt, FeatureCallIndirectOverlong,
                       FeatureExtendedConst, FeatureMultivalue,
                       FeatureMutableGlobals, FeatureNontrappingFPToInt,
                       FeatureSignExt]>;

// Latest and greatest experimental version of WebAssembly. Bugs included!
```
- **EN**: Lists processor models and binds the default feature bundles or scheduling data attached to each model. Subtarget feature gating influences the behavior here.
- **CN**: 这一段列出处理器模型，并把默认特性集合或调度数据绑定到相应模型上。 子目标特性裁剪会影响这里的行为。

### Lines 153-161

```tablegen
def : ProcessorModel<"bleeding-edge", NoSchedModel,
                     [FeatureAtomics, FeatureBulkMemory, FeatureBulkMemoryOpt,
                      FeatureCallIndirectOverlong, FeatureExceptionHandling,
                      FeatureExtendedConst, FeatureFP16, FeatureGC,
                      FeatureMultiMemory, FeatureMultivalue, FeatureMutableGlobals,
                      FeatureNontrappingFPToInt, FeatureRelaxedSIMD,
                      FeatureReferenceTypes, FeatureSIMD128,
                      FeatureSignExt, FeatureTailCall]>;
```
- **EN**: Lists processor models and binds the default feature bundles or scheduling data attached to each model. Subtarget feature gating influences the behavior here.
- **CN**: 这一段列出处理器模型，并把默认特性集合或调度数据绑定到相应模型上。 子目标特性裁剪会影响这里的行为。

### Lines 162-163

```tablegen
//===----------------------------------------------------------------------===//
// Target Declaration
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Target Declaration".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Target Declaration”。

### Lines 164-170

```tablegen
//===----------------------------------------------------------------------===//

def WebAssemblyAsmParser : AsmParser {
  // The physical register names are not in the binary format or asm text
  let ShouldEmitMatchRegisterName = 0;
}
```
- **EN**: Adds declarative TableGen records such as `WebAssemblyAsmParser` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `WebAssemblyAsmParser`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 171-177

```tablegen
def WebAssemblyAsmWriter : AsmWriter {
  string AsmWriterClassName  = "InstPrinter";
  int PassSubtarget = 1;
  int Variant = 0;
  bit isMCAsmWriter = 1;
}
```
- **EN**: Adds declarative TableGen records such as `WebAssemblyAsmWriter` that LLVM later expands into generated tables or helper code. Subtarget feature gating influences the behavior here.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `WebAssemblyAsmWriter`，LLVM 随后会把它们展开成生成表或辅助代码。 子目标特性裁剪会影响这里的行为。

### Lines 178-182

```tablegen
def WebAssembly : Target {
  let InstructionSet = WebAssemblyInstrInfo;
  let AssemblyParsers  = [WebAssemblyAsmParser];
  let AssemblyWriters = [WebAssemblyAsmWriter];
}
```
- **EN**: Adds declarative TableGen records such as `WebAssembly` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `WebAssembly`，LLVM 随后会把它们展开成生成表或辅助代码。

## Key Concepts / 关键概念

- WebAssembly backend integration / WebAssembly 后端集成
- TableGen declarations for the backend / 该后端的 TableGen 声明
- LLVM TableGen DSL / LLVM TableGen 领域特定语言
- Register modeling / 寄存器建模
- Subtarget features / 子目标特性
- Assembly parsing / 汇编解析
- Object format integration / 目标文件格式集成
- Declarative TableGen records / 声明式 TableGen 记录

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `llvm/Target/Target.td`
- `WebAssemblyRegisterInfo.td`
- `WebAssemblyRegisterBanks.td`
- `WebAssemblyInstrInfo.td`

### Important Collaborators / 重要协作组件

- LLVM MC layer / LLVM MC 层
- Generated backend tables / 生成式后端表
- WebAssembly target-specific helpers / WebAssembly 目标专用辅助组件
