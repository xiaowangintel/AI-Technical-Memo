# PPC.td — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/PowerPC/PPC.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: PPC.td - Describe the PowerPC Target Machine. It is written in LLVM TableGen DSL and feeds generated backend tables.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/PowerPC/PPC.td`，主要负责 PowerPC 后端的该后端的 TableGen 声明。 它使用 LLVM TableGen DSL 编写，用于描述声明式目标信息并驱动生成代码。
- **Language**: TableGen DSL / TableGen 领域特定语言

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```tablegen
//===-- PPC.td - Describe the PowerPC Target Machine -------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-10

```tablegen
//===----------------------------------------------------------------------===//
//
// This is the top level entry point for the PowerPC target.
//
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "This is the top level entry point for the PowerPC target.".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“This is the top level entry point for the PowerPC target.”。

### Lines 11-16

```tablegen
//===----------------------------------------------------------------------===//

// Get the target-independent interfaces which we are implementing.
//
include "llvm/Target/Target.td"
```
- **EN**: Pulls in direct dependencies required by this TableGen declarations for the backend, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该该后端的 TableGen 声明所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 17-20

```tablegen
//===----------------------------------------------------------------------===//
// PowerPC Subtarget features.
//
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "PowerPC Subtarget features.". Subtarget feature gating influences the behavior here.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“PowerPC Subtarget features.”。 子目标特性裁剪会影响这里的行为。

### Lines 21-22

```tablegen
//===----------------------------------------------------------------------===//
// CPU Directives                                                             //
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "CPU Directives                                                             //".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“CPU Directives                                                             //”。

### Lines 23-29

```tablegen
//===----------------------------------------------------------------------===//

def Directive440 : SubtargetFeature<"", "CPUDirective", "PPC::DIR_440", "">;
def Directive601 : SubtargetFeature<"", "CPUDirective", "PPC::DIR_601", "">;
def Directive602 : SubtargetFeature<"", "CPUDirective", "PPC::DIR_602", "">;
def Directive603 : SubtargetFeature<"", "CPUDirective", "PPC::DIR_603", "">;
def Directive604 : SubtargetFeature<"", "CPUDirective", "PPC::DIR_603", "">;
```
- **EN**: Defines TableGen subtarget feature flags for PowerPC, allowing CPU or feature strings to toggle generated behavior. Subtarget feature gating influences the behavior here.
- **CN**: 这里为 PowerPC 定义 TableGen 子目标特性标志，使 CPU 或 feature 字符串能够切换生成出来的行为。 子目标特性裁剪会影响这里的行为。

### Lines 30-36

```tablegen
def Directive620 : SubtargetFeature<"", "CPUDirective", "PPC::DIR_603", "">;
def Directive7400: SubtargetFeature<"", "CPUDirective", "PPC::DIR_7400", "">;
def Directive750 : SubtargetFeature<"", "CPUDirective", "PPC::DIR_750", "">;
def Directive970 : SubtargetFeature<"", "CPUDirective", "PPC::DIR_970", "">;
def Directive32  : SubtargetFeature<"", "CPUDirective", "PPC::DIR_32", "">;
def Directive64  : SubtargetFeature<"", "CPUDirective", "PPC::DIR_64", "">;
def DirectiveA2  : SubtargetFeature<"", "CPUDirective", "PPC::DIR_A2", "">;
```
- **EN**: Defines TableGen subtarget feature flags for PowerPC, allowing CPU or feature strings to toggle generated behavior. Subtarget feature gating influences the behavior here.
- **CN**: 这里为 PowerPC 定义 TableGen 子目标特性标志，使 CPU 或 feature 字符串能够切换生成出来的行为。 子目标特性裁剪会影响这里的行为。

### Lines 37-43

```tablegen
def DirectiveE500   : SubtargetFeature<"", "CPUDirective",
                                       "PPC::DIR_E500", "">;
def DirectiveE500mc : SubtargetFeature<"", "CPUDirective",
                                       "PPC::DIR_E500mc", "">;
def DirectiveE5500  : SubtargetFeature<"", "CPUDirective",
                                       "PPC::DIR_E5500", "">;
def DirectivePwr3: SubtargetFeature<"", "CPUDirective", "PPC::DIR_PWR3", "">;
```
- **EN**: Defines TableGen subtarget feature flags for PowerPC, allowing CPU or feature strings to toggle generated behavior. Subtarget feature gating influences the behavior here.
- **CN**: 这里为 PowerPC 定义 TableGen 子目标特性标志，使 CPU 或 feature 字符串能够切换生成出来的行为。 子目标特性裁剪会影响这里的行为。

### Lines 44-50

```tablegen
def DirectivePwr4: SubtargetFeature<"", "CPUDirective", "PPC::DIR_PWR4", "">;
def DirectivePwr5: SubtargetFeature<"", "CPUDirective", "PPC::DIR_PWR5", "">;
def DirectivePwr5x
    : SubtargetFeature<"", "CPUDirective", "PPC::DIR_PWR5X", "">;
def DirectivePwr6: SubtargetFeature<"", "CPUDirective", "PPC::DIR_PWR6", "">;
def DirectivePwr6x
    : SubtargetFeature<"", "CPUDirective", "PPC::DIR_PWR6X", "">;
```
- **EN**: Defines TableGen subtarget feature flags for PowerPC, allowing CPU or feature strings to toggle generated behavior. Subtarget feature gating influences the behavior here.
- **CN**: 这里为 PowerPC 定义 TableGen 子目标特性标志，使 CPU 或 feature 字符串能够切换生成出来的行为。 子目标特性裁剪会影响这里的行为。

### Lines 51-60

```tablegen
def DirectivePwr7: SubtargetFeature<"", "CPUDirective", "PPC::DIR_PWR7", "">;
def DirectivePwr8: SubtargetFeature<"", "CPUDirective", "PPC::DIR_PWR8", "">;
def DirectivePwr9: SubtargetFeature<"", "CPUDirective", "PPC::DIR_PWR9", "">;
def DirectivePwr10: SubtargetFeature<"", "CPUDirective", "PPC::DIR_PWR10", "">;
def DirectivePwr11: SubtargetFeature<"", "CPUDirective", "PPC::DIR_PWR11", "">;
def DirectivePwrFuture
    : SubtargetFeature<"", "CPUDirective", "PPC::DIR_PWR_FUTURE", "">;

// Specifies that the selected CPU supports 64-bit instructions, regardless of
// whether we are in 32-bit or 64-bit mode.
```
- **EN**: Defines TableGen subtarget feature flags for PowerPC, allowing CPU or feature strings to toggle generated behavior. Subtarget feature gating influences the behavior here.
- **CN**: 这里为 PowerPC 定义 TableGen 子目标特性标志，使 CPU 或 feature 字符串能够切换生成出来的行为。 子目标特性裁剪会影响这里的行为。

### Lines 61-67

```tablegen
def Feature64BitSupport : SubtargetFeature<"64bit-support", "Has64BitSupport", "true",
                                        "Supports 64-bit instructions">;
// 64-bit is enabled.
def Feature64Bit     : SubtargetFeature<"64bit", "IsPPC64", "true",
                                        "Enable 64-bit mode",
                                        [Feature64BitSupport]>;
```
- **EN**: Defines TableGen subtarget feature flags for PowerPC, allowing CPU or feature strings to toggle generated behavior. Subtarget feature gating influences the behavior here.
- **CN**: 这里为 PowerPC 定义 TableGen 子目标特性标志，使 CPU 或 feature 字符串能够切换生成出来的行为。 子目标特性裁剪会影响这里的行为。

### Lines 68-76

```tablegen
def AIXOS: SubtargetFeature<"aix", "IsAIX", "true", "AIX OS">;
def FeatureModernAIXAs
    : SubtargetFeature<"modern-aix-as", "HasModernAIXAs", "true",
                       "AIX system assembler is modern enough to support new mnes">;
def FeatureHardFloat : SubtargetFeature<"hard-float", "HasHardFloat", "true",
                              "Enable floating-point instructions">;

// Specifies that we are in 64-bit mode or that we should use 64-bit registers
// in 32-bit mode when possible. Requires Feature64Bit to be enabled.
```
- **EN**: Defines TableGen subtarget feature flags for PowerPC, allowing CPU or feature strings to toggle generated behavior. Subtarget feature gating influences the behavior here.
- **CN**: 这里为 PowerPC 定义 TableGen 子目标特性标志，使 CPU 或 feature 字符串能够切换生成出来的行为。 子目标特性裁剪会影响这里的行为。

### Lines 77-83

```tablegen
def Feature64BitRegs : SubtargetFeature<"64bitregs","Use64BitRegs", "true",
                              "Enable 64-bit registers usage for ppc32 [beta]">;

// Specify if we should store and manipulate i1 values in the individual
// condition register bits.
def FeatureCRBits    : SubtargetFeature<"crbits", "UseCRBits", "true",
                              "Use condition-register bits individually">;
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Specify if we should store and manipulate i1 values in the individual". Subtarget feature gating influences the behavior here.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Specify if we should store and manipulate i1 values in the individual”。 子目标特性裁剪会影响这里的行为。

### Lines 84-92

```tablegen
def FeatureFPU       : SubtargetFeature<"fpu","HasFPU","true",
                                        "Enable classic FPU instructions",
                                        [FeatureHardFloat]>;
def FeatureAltivec   : SubtargetFeature<"altivec","HasAltivec", "true",
                                        "Enable Altivec instructions",
                                        [FeatureFPU]>;
def FeatureSPE       : SubtargetFeature<"spe","HasSPE", "true",
                                        "Enable SPE instructions",
                                        [FeatureHardFloat]>;
```
- **EN**: Defines TableGen subtarget feature flags for PowerPC, allowing CPU or feature strings to toggle generated behavior. Subtarget feature gating influences the behavior here.
- **CN**: 这里为 PowerPC 定义 TableGen 子目标特性标志，使 CPU 或 feature 字符串能够切换生成出来的行为。 子目标特性裁剪会影响这里的行为。

### Lines 93-100

```tablegen
def FeatureEFPU2 : SubtargetFeature<"efpu2", "HasEFPU2", "true",
                                        "Enable Embedded Floating-Point APU 2 instructions",
                                        [FeatureSPE]>;
def FeatureMFOCRF    : SubtargetFeature<"mfocrf","HasMFOCRF", "true",
                                        "Enable the MFOCRF instruction">;
def FeatureFSqrt     : SubtargetFeature<"fsqrt","HasFSQRT", "true",
                                        "Enable the fsqrt instruction",
                                        [FeatureFPU]>;
```
- **EN**: Defines TableGen subtarget feature flags for PowerPC, allowing CPU or feature strings to toggle generated behavior. Subtarget feature gating influences the behavior here.
- **CN**: 这里为 PowerPC 定义 TableGen 子目标特性标志，使 CPU 或 feature 字符串能够切换生成出来的行为。 子目标特性裁剪会影响这里的行为。

### Lines 101-109

```tablegen
def FeatureFCPSGN    : SubtargetFeature<"fcpsgn", "HasFCPSGN", "true",
                                        "Enable the fcpsgn instruction",
                                        [FeatureFPU]>;
def FeatureFRE       : SubtargetFeature<"fre", "HasFRE", "true",
                                        "Enable the fre instruction",
                                        [FeatureFPU]>;
def FeatureFRES      : SubtargetFeature<"fres", "HasFRES", "true",
                                        "Enable the fres instruction",
                                        [FeatureFPU]>;
```
- **EN**: Defines TableGen subtarget feature flags for PowerPC, allowing CPU or feature strings to toggle generated behavior. Subtarget feature gating influences the behavior here.
- **CN**: 这里为 PowerPC 定义 TableGen 子目标特性标志，使 CPU 或 feature 字符串能够切换生成出来的行为。 子目标特性裁剪会影响这里的行为。

### Lines 110-117

```tablegen
def FeatureFRSQRTE   : SubtargetFeature<"frsqrte", "HasFRSQRTE", "true",
                                        "Enable the frsqrte instruction",
                                        [FeatureFPU]>;
def FeatureFRSQRTES  : SubtargetFeature<"frsqrtes", "HasFRSQRTES", "true",
                                        "Enable the frsqrtes instruction",
                                        [FeatureFPU]>;
def FeatureRecipPrec : SubtargetFeature<"recipprec", "HasRecipPrec", "true",
                              "Assume higher precision reciprocal estimates">;
```
- **EN**: Defines TableGen subtarget feature flags for PowerPC, allowing CPU or feature strings to toggle generated behavior. Subtarget feature gating influences the behavior here.
- **CN**: 这里为 PowerPC 定义 TableGen 子目标特性标志，使 CPU 或 feature 字符串能够切换生成出来的行为。 子目标特性裁剪会影响这里的行为。

### Lines 118-126

```tablegen
def FeatureSTFIWX    : SubtargetFeature<"stfiwx","HasSTFIWX", "true",
                                        "Enable the stfiwx instruction",
                                        [FeatureFPU]>;
def FeatureLFIWAX    : SubtargetFeature<"lfiwax","HasLFIWAX", "true",
                                        "Enable the lfiwax instruction",
                                        [FeatureFPU]>;
def FeatureFPRND     : SubtargetFeature<"fprnd", "HasFPRND", "true",
                                        "Enable the fri[mnpz] instructions",
                                        [FeatureFPU]>;
```
- **EN**: Defines TableGen subtarget feature flags for PowerPC, allowing CPU or feature strings to toggle generated behavior. Subtarget feature gating influences the behavior here.
- **CN**: 这里为 PowerPC 定义 TableGen 子目标特性标志，使 CPU 或 feature 字符串能够切换生成出来的行为。 子目标特性裁剪会影响这里的行为。

### Lines 127-133

```tablegen
def FeatureFPCVT     : SubtargetFeature<"fpcvt", "HasFPCVT", "true",
  "Enable fc[ft]* (unsigned and single-precision) and lfiwzx instructions",
                                        [FeatureFPU]>;
def FeatureISEL      : SubtargetFeature<"isel","HasISEL", "true",
                                        "Enable the isel instruction">;
def FeatureBPERMD    : SubtargetFeature<"bpermd", "HasBPERMD", "true",
                                        "Enable the bpermd instruction">;
```
- **EN**: Defines TableGen subtarget feature flags for PowerPC, allowing CPU or feature strings to toggle generated behavior. Subtarget feature gating influences the behavior here.
- **CN**: 这里为 PowerPC 定义 TableGen 子目标特性标志，使 CPU 或 feature 字符串能够切换生成出来的行为。 子目标特性裁剪会影响这里的行为。

### Lines 134-141

```tablegen
def FeatureExtDiv    : SubtargetFeature<"extdiv", "HasExtDiv", "true",
                                        "Enable extended divide instructions">;
def FeatureLDBRX     : SubtargetFeature<"ldbrx","HasLDBRX", "true",
                                        "Enable the ldbrx instruction">;
def FeatureCMPB      : SubtargetFeature<"cmpb", "HasCMPB", "true",
                                        "Enable the cmpb instruction">;
def FeatureICBT      : SubtargetFeature<"icbt","HasICBT", "true",
                                        "Enable icbt instruction">;
```
- **EN**: Defines TableGen subtarget feature flags for PowerPC, allowing CPU or feature strings to toggle generated behavior. Subtarget feature gating influences the behavior here.
- **CN**: 这里为 PowerPC 定义 TableGen 子目标特性标志，使 CPU 或 feature 字符串能够切换生成出来的行为。 子目标特性裁剪会影响这里的行为。

### Lines 142-149

```tablegen
def FeatureBookE     : SubtargetFeature<"booke", "IsBookE", "true",
                                        "Enable Book E instructions",
                                        [FeatureICBT]>;
def FeatureMSYNC     : SubtargetFeature<"msync", "HasOnlyMSYNC", "true",
                              "Has only the msync instruction instead of sync",
                              [FeatureBookE]>;
def FeatureE500      : SubtargetFeature<"e500", "IsE500", "true",
                                        "Enable E500/E500mc instructions">;
```
- **EN**: Defines TableGen subtarget feature flags for PowerPC, allowing CPU or feature strings to toggle generated behavior. Subtarget feature gating influences the behavior here.
- **CN**: 这里为 PowerPC 定义 TableGen 子目标特性标志，使 CPU 或 feature 字符串能够切换生成出来的行为。 子目标特性裁剪会影响这里的行为。

### Lines 150-158

```tablegen
def FeatureSecurePlt : SubtargetFeature<"secure-plt","IsSecurePlt", "true",
                                        "Enable secure plt mode">;
def FeaturePPC4xx    : SubtargetFeature<"ppc4xx", "IsPPC4xx", "true",
                                        "Enable PPC 4xx instructions">;
def FeaturePPC6xx    : SubtargetFeature<"ppc6xx", "IsPPC6xx", "true",
                                        "Enable PPC 6xx instructions">;
def FeatureVSX       : SubtargetFeature<"vsx","HasVSX", "true",
                                        "Enable VSX instructions",
                                        [FeatureAltivec]>;
```
- **EN**: Defines TableGen subtarget feature flags for PowerPC, allowing CPU or feature strings to toggle generated behavior. Subtarget feature gating influences the behavior here.
- **CN**: 这里为 PowerPC 定义 TableGen 子目标特性标志，使 CPU 或 feature 字符串能够切换生成出来的行为。 子目标特性裁剪会影响这里的行为。

### Lines 159-167

```tablegen
def FeatureTwoConstNR :
  SubtargetFeature<"two-const-nr", "NeedsTwoConstNR", "true",
                   "Requires two constant Newton-Raphson computation">;
def FeatureP8Altivec : SubtargetFeature<"power8-altivec", "HasP8Altivec", "true",
                                        "Enable POWER8 Altivec instructions",
                                        [FeatureAltivec]>;
def FeatureP8Crypto : SubtargetFeature<"crypto", "HasP8Crypto", "true",
                                       "Enable POWER8 Crypto instructions",
                                       [FeatureP8Altivec]>;
```
- **EN**: Defines TableGen subtarget feature flags for PowerPC, allowing CPU or feature strings to toggle generated behavior. Subtarget feature gating influences the behavior here.
- **CN**: 这里为 PowerPC 定义 TableGen 子目标特性标志，使 CPU 或 feature 字符串能够切换生成出来的行为。 子目标特性裁剪会影响这里的行为。

### Lines 168-174

```tablegen
def FeatureP8Vector  : SubtargetFeature<"power8-vector", "HasP8Vector", "true",
                                        "Enable POWER8 vector instructions",
                                        [FeatureVSX, FeatureP8Altivec]>;
def FeatureDirectMove :
  SubtargetFeature<"direct-move", "HasDirectMove", "true",
                   "Enable Power8 direct move instructions",
                   [FeatureVSX]>;
```
- **EN**: Defines TableGen subtarget feature flags for PowerPC, allowing CPU or feature strings to toggle generated behavior. Subtarget feature gating influences the behavior here.
- **CN**: 这里为 PowerPC 定义 TableGen 子目标特性标志，使 CPU 或 feature 字符串能够切换生成出来的行为。 子目标特性裁剪会影响这里的行为。

### Lines 175-184

```tablegen
def FeaturePartwordAtomic : SubtargetFeature<"partword-atomics",
                                             "HasPartwordAtomics", "true",
                                             "Enable l[bh]arx and st[bh]cx.">;
def FeatureQuadwordAtomic : SubtargetFeature<"quadword-atomics",
                                             "HasQuadwordAtomics", "true",
                                             "Enable lqarx and stqcx.">;
def FeatureInvariantFunctionDescriptors :
  SubtargetFeature<"invariant-function-descriptors",
                   "HasInvariantFunctionDescriptors", "true",
                   "Assume function descriptors are invariant">;
```
- **EN**: Defines TableGen subtarget feature flags for PowerPC, allowing CPU or feature strings to toggle generated behavior. Subtarget feature gating influences the behavior here.
- **CN**: 这里为 PowerPC 定义 TableGen 子目标特性标志，使 CPU 或 feature 字符串能够切换生成出来的行为。 子目标特性裁剪会影响这里的行为。

### Lines 185-192

```tablegen
def FeatureLongCall : SubtargetFeature<"longcall", "UseLongCalls", "true",
                                       "Always use indirect calls">;
def FeatureHTM : SubtargetFeature<"htm", "HasHTM", "true",
                                  "Enable Hardware Transactional Memory instructions">;
def FeatureMFTB   : SubtargetFeature<"", "IsFeatureMFTB", "true",
                                        "Implement mftb using the mfspr instruction">;
def FeatureFusion : SubtargetFeature<"fusion", "HasFusion", "true",
                                     "Target supports instruction fusion">;
```
- **EN**: Defines TableGen subtarget feature flags for PowerPC, allowing CPU or feature strings to toggle generated behavior. Subtarget feature gating influences the behavior here.
- **CN**: 这里为 PowerPC 定义 TableGen 子目标特性标志，使 CPU 或 feature 字符串能够切换生成出来的行为。 子目标特性裁剪会影响这里的行为。

### Lines 193-200

```tablegen
def FeatureAddiLoadFusion : SubtargetFeature<"fuse-addi-load",
                                             "HasAddiLoadFusion", "true",
                                             "Power8 Addi-Load fusion",
                                             [FeatureFusion]>;
def FeatureAddisLoadFusion : SubtargetFeature<"fuse-addis-load",
                                              "HasAddisLoadFusion", "true",
                                              "Power8 Addis-Load fusion",
                                              [FeatureFusion]>;
```
- **EN**: Defines TableGen subtarget feature flags for PowerPC, allowing CPU or feature strings to toggle generated behavior. Subtarget feature gating influences the behavior here.
- **CN**: 这里为 PowerPC 定义 TableGen 子目标特性标志，使 CPU 或 feature 字符串能够切换生成出来的行为。 子目标特性裁剪会影响这里的行为。

### Lines 201-207

```tablegen
def FeatureStoreFusion : SubtargetFeature<"fuse-store", "HasStoreFusion", "true",
                                          "Target supports store clustering",
                                          [FeatureFusion]>;
def FeatureArithAddFusion :
  SubtargetFeature<"fuse-arith-add", "HasArithAddFusion", "true",
                   "Target supports Arithmetic Operations with Add fusion",
                   [FeatureFusion]>;
```
- **EN**: Defines TableGen subtarget feature flags for PowerPC, allowing CPU or feature strings to toggle generated behavior. Subtarget feature gating influences the behavior here.
- **CN**: 这里为 PowerPC 定义 TableGen 子目标特性标志，使 CPU 或 feature 字符串能够切换生成出来的行为。 子目标特性裁剪会影响这里的行为。

### Lines 208-215

```tablegen
def FeatureAddLogicalFusion :
  SubtargetFeature<"fuse-add-logical", "HasAddLogicalFusion", "true",
                   "Target supports Add with Logical Operations fusion",
                   [FeatureFusion]>;
def FeatureLogicalAddFusion :
  SubtargetFeature<"fuse-logical-add", "HasLogicalAddFusion", "true",
                   "Target supports Logical with Add Operations fusion",
                   [FeatureFusion]>;
```
- **EN**: Defines TableGen subtarget feature flags for PowerPC, allowing CPU or feature strings to toggle generated behavior. Subtarget feature gating influences the behavior here.
- **CN**: 这里为 PowerPC 定义 TableGen 子目标特性标志，使 CPU 或 feature 字符串能够切换生成出来的行为。 子目标特性裁剪会影响这里的行为。

### Lines 216-223

```tablegen
def FeatureLogicalFusion :
  SubtargetFeature<"fuse-logical", "HasLogicalFusion", "true",
                   "Target supports Logical Operations fusion",
                   [FeatureFusion]>;
def FeatureSha3Fusion :
  SubtargetFeature<"fuse-sha3", "HasSha3Fusion", "true",
                   "Target supports SHA3 assist fusion",
                   [FeatureFusion]>;
```
- **EN**: Defines TableGen subtarget feature flags for PowerPC, allowing CPU or feature strings to toggle generated behavior. Subtarget feature gating influences the behavior here.
- **CN**: 这里为 PowerPC 定义 TableGen 子目标特性标志，使 CPU 或 feature 字符串能够切换生成出来的行为。 子目标特性裁剪会影响这里的行为。

### Lines 224-231

```tablegen
def FeatureCompareFusion:
  SubtargetFeature<"fuse-cmp", "HasCompareFusion", "true",
                   "Target supports Comparison Operations fusion",
                   [FeatureFusion]>;
def FeatureWideImmFusion:
  SubtargetFeature<"fuse-wideimm", "HasWideImmFusion", "true",
                   "Target supports Wide-Immediate fusion",
                   [FeatureFusion]>;
```
- **EN**: Defines TableGen subtarget feature flags for PowerPC, allowing CPU or feature strings to toggle generated behavior. Subtarget feature gating influences the behavior here.
- **CN**: 这里为 PowerPC 定义 TableGen 子目标特性标志，使 CPU 或 feature 字符串能够切换生成出来的行为。 子目标特性裁剪会影响这里的行为。

### Lines 232-239

```tablegen
def FeatureZeroMoveFusion:
  SubtargetFeature<"fuse-zeromove", "HasZeroMoveFusion", "true",
                   "Target supports move to SPR with branch fusion",
                   [FeatureFusion]>;
def FeatureBack2BackFusion:
  SubtargetFeature<"fuse-back2back", "HasBack2BackFusion", "true",
                   "Target supports general back to back fusion",
                   [FeatureFusion]>;
```
- **EN**: Defines TableGen subtarget feature flags for PowerPC, allowing CPU or feature strings to toggle generated behavior. Subtarget feature gating influences the behavior here.
- **CN**: 这里为 PowerPC 定义 TableGen 子目标特性标志，使 CPU 或 feature 字符串能够切换生成出来的行为。 子目标特性裁剪会影响这里的行为。

### Lines 240-248

```tablegen
def FeatureUnalignedFloats :
  SubtargetFeature<"allow-unaligned-fp-access", "AllowsUnalignedFPAccess",
                   "true", "CPU does not trap on unaligned FP access">;
def FeaturePPCPreRASched:
  SubtargetFeature<"ppc-prera-sched", "UsePPCPreRASchedStrategy", "true",
                   "Use PowerPC pre-RA scheduling strategy">;
def FeaturePPCPostRASched:
  SubtargetFeature<"ppc-postra-sched", "UsePPCPostRASchedStrategy", "true",
                   "Use PowerPC post-RA scheduling strategy">;
```
- **EN**: Defines TableGen subtarget feature flags for PowerPC, allowing CPU or feature strings to toggle generated behavior. Subtarget feature gating influences the behavior here.
- **CN**: 这里为 PowerPC 定义 TableGen 子目标特性标志，使 CPU 或 feature 字符串能够切换生成出来的行为。 子目标特性裁剪会影响这里的行为。

### Lines 249-258

```tablegen
def FeatureFloat128 :
  SubtargetFeature<"float128", "HasFloat128", "true",
                   "Enable the __float128 data type for IEEE-754R Binary128.",
                   [FeatureVSX]>;
def FeaturePOPCNTD   : SubtargetFeature<"popcntd","HasPOPCNTD",
                                        "POPCNTD_Fast",
                                        "Enable the popcnt[dw] instructions">;
// Note that for the a2 processor models we should not use popcnt[dw] by
// default. These processors do support the instructions, but they're
// microcoded, and the software emulation is about twice as fast.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Note that for the a2 processor models we should not use popcnt[dw] by". Subtarget feature gating influences the behavior here.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Note that for the a2 processor models we should not use popcnt[dw] by”。 子目标特性裁剪会影响这里的行为。

### Lines 259-265

```tablegen
def FeatureSlowPOPCNTD : SubtargetFeature<"slow-popcntd","HasPOPCNTD",
                                          "POPCNTD_Slow",
                                          "Has slow popcnt[dw] instructions">;

def DeprecatedDST    : SubtargetFeature<"", "IsDeprecatedDST", "true",
  "Treat vector data stream cache control instructions as deprecated">;
```
- **EN**: Defines TableGen subtarget feature flags for PowerPC, allowing CPU or feature strings to toggle generated behavior. Subtarget feature gating influences the behavior here.
- **CN**: 这里为 PowerPC 定义 TableGen 子目标特性标志，使 CPU 或 feature 字符串能够切换生成出来的行为。 子目标特性裁剪会影响这里的行为。

### Lines 266-275

```tablegen
def FeatureISA2_06 : SubtargetFeature<"isa-v206-instructions", "IsISA2_06",
                                      "true",
                                      "Enable instructions in ISA 2.06.">;
def FeatureISA2_07 : SubtargetFeature<"isa-v207-instructions", "IsISA2_07",
                                      "true",
                                      "Enable instructions in ISA 2.07.">;
def FeatureISA3_0 : SubtargetFeature<"isa-v30-instructions", "IsISA3_0",
                                     "true",
                                     "Enable instructions in ISA 3.0.",
                                     [FeatureISA2_07]>;
```
- **EN**: Defines TableGen subtarget feature flags for PowerPC, allowing CPU or feature strings to toggle generated behavior. Subtarget feature gating influences the behavior here.
- **CN**: 这里为 PowerPC 定义 TableGen 子目标特性标志，使 CPU 或 feature 字符串能够切换生成出来的行为。 子目标特性裁剪会影响这里的行为。

### Lines 276-283

```tablegen
def FeatureISA3_1 : SubtargetFeature<"isa-v31-instructions", "IsISA3_1",
                                     "true",
                                     "Enable instructions in ISA 3.1.",
                                     [FeatureISA3_0]>;
def FeatureISAFuture : SubtargetFeature<"isa-future-instructions",
                                        "IsISAFuture", "true",
                                        "Enable instructions for Future ISA.",
                                        [FeatureISA3_1]>;
```
- **EN**: Defines TableGen subtarget feature flags for PowerPC, allowing CPU or feature strings to toggle generated behavior. Subtarget feature gating influences the behavior here.
- **CN**: 这里为 PowerPC 定义 TableGen 子目标特性标志，使 CPU 或 feature 字符串能够切换生成出来的行为。 子目标特性裁剪会影响这里的行为。

### Lines 284-290

```tablegen
def FeatureP9Altivec : SubtargetFeature<"power9-altivec", "HasP9Altivec", "true",
                                        "Enable POWER9 Altivec instructions",
                                        [FeatureISA3_0, FeatureP8Altivec]>;
def FeatureP9Vector  : SubtargetFeature<"power9-vector", "HasP9Vector", "true",
                                        "Enable POWER9 vector instructions",
                                        [FeatureISA3_0, FeatureP8Vector,
                                         FeatureP9Altivec]>;
```
- **EN**: Defines TableGen subtarget feature flags for PowerPC, allowing CPU or feature strings to toggle generated behavior. Subtarget feature gating influences the behavior here.
- **CN**: 这里为 PowerPC 定义 TableGen 子目标特性标志，使 CPU 或 feature 字符串能够切换生成出来的行为。 子目标特性裁剪会影响这里的行为。

### Lines 291-301

```tablegen
def FeatureP10Vector  : SubtargetFeature<"power10-vector", "HasP10Vector",
                                         "true",
                                         "Enable POWER10 vector instructions",
                                         [FeatureISA3_1, FeatureP9Vector]>;
def FeatureFutureVector : SubtargetFeature<"future-vector", "HasFutureVector",
                                           "true",
                                           "Enable Future vector instructions",
                                           [FeatureISAFuture, FeatureP10Vector]>;
// A separate feature for this even though it is equivalent to P9Vector
// because this is a feature of the implementation rather than the architecture
// and may go away with future CPU's.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "A separate feature for this even though it is equivalent to P9Vector". Subtarget feature gating influences the behavior here.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“A separate feature for this even though it is equivalent to P9Vector”。 子目标特性裁剪会影响这里的行为。

### Lines 302-309

```tablegen
def FeatureVectorsUseTwoUnits : SubtargetFeature<"vectors-use-two-units",
                                                 "VectorsUseTwoUnits",
                                                 "true",
                                                 "Vectors use two units">;
def FeaturePrefixInstrs : SubtargetFeature<"prefix-instrs", "HasPrefixInstrs",
                                           "true",
                                           "Enable prefixed instructions",
                                           [FeatureISA3_1]>;
```
- **EN**: Defines TableGen subtarget feature flags for PowerPC, allowing CPU or feature strings to toggle generated behavior. Subtarget feature gating influences the behavior here.
- **CN**: 这里为 PowerPC 定义 TableGen 子目标特性标志，使 CPU 或 feature 字符串能够切换生成出来的行为。 子目标特性裁剪会影响这里的行为。

### Lines 310-317

```tablegen
def FeaturePCRelativeMemops :
  SubtargetFeature<"pcrelative-memops", "HasPCRelativeMemops", "true",
                   "Enable PC relative Memory Ops",
                   [FeatureISA3_0, FeaturePrefixInstrs]>;
def FeaturePairedVectorMemops:
  SubtargetFeature<"paired-vector-memops", "PairedVectorMemops", "true",
                   "32Byte load and store instructions",
                   [FeatureISA3_0, FeatureVSX]>;
```
- **EN**: Defines TableGen subtarget feature flags for PowerPC, allowing CPU or feature strings to toggle generated behavior. Subtarget feature gating influences the behavior here.
- **CN**: 这里为 PowerPC 定义 TableGen 子目标特性标志，使 CPU 或 feature 字符串能够切换生成出来的行为。 子目标特性裁剪会影响这里的行为。

### Lines 318-325

```tablegen
def FeatureMMA : SubtargetFeature<"mma", "HasMMA", "true",
                                  "Enable MMA instructions",
                                  [FeatureP8Vector, FeatureP9Altivec,
                                   FeaturePairedVectorMemops]>;
def FeatureROPProtect :
  SubtargetFeature<"rop-protect", "HasROPProtect", "true",
                   "Add ROP protect">;
```
- **EN**: Defines TableGen subtarget feature flags for PowerPC, allowing CPU or feature strings to toggle generated behavior. Subtarget feature gating influences the behavior here.
- **CN**: 这里为 PowerPC 定义 TableGen 子目标特性标志，使 CPU 或 feature 字符串能够切换生成出来的行为。 子目标特性裁剪会影响这里的行为。

### Lines 326-335

```tablegen
def FeaturePrivileged :
  SubtargetFeature<"privileged", "HasPrivileged", "true",
                   "Add privileged instructions">;

// Specifies that local-exec TLS accesses in any function with this target
// attribute should use the optimized TOC-free sequence (where the offset is an
// immediate off of R13 for which the linker might add fix-up code if the
// immediate is too large).
// Clearly, this isn't really a feature of the subtarget, but is used as a
// convenient way to affect code generation for individual functions.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Specifies that local-exec TLS accesses in any function with this target". Subtarget feature gating influences the behavior here.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Specifies that local-exec TLS accesses in any function with this target”。 子目标特性裁剪会影响这里的行为。

### Lines 336-344

```tablegen
def FeatureAIXLocalExecTLS :
  SubtargetFeature<"aix-small-local-exec-tls", "HasAIXSmallLocalExecTLS", "true",
                   "Produce a TOC-free local-exec TLS sequence for this function "
                   "for 64-bit AIX">;

// Specifies that local-dynamic TLS accesses in any function with this target
// attribute should use the optimized sequence (where the offset is an immediate
// off the module-handle for which the linker might add fix-up code for if the
// immediate is too large).
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Specifies that local-dynamic TLS accesses in any function with this target". Subtarget feature gating influences the behavior here.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Specifies that local-dynamic TLS accesses in any function with this target”。 子目标特性裁剪会影响这里的行为。

### Lines 345-355

```tablegen
def FeatureAIXLocalDynamicTLS :
  SubtargetFeature<"aix-small-local-dynamic-tls", "HasAIXSmallLocalDynamicTLS",
                   "true", "Produce a faster local-dynamic TLS sequence for this "
                   "function for 64-bit AIX">;

def FeatureAIXSharedLibTLSModelOpt :
  SubtargetFeature<"aix-shared-lib-tls-model-opt",
                   "HasAIXShLibTLSModelOpt", "true",
                   "Tune TLS model at function level in shared library loaded "
                   "with the main program (for 64-bit AIX only)">;
```
- **EN**: Defines TableGen subtarget feature flags for PowerPC, allowing CPU or feature strings to toggle generated behavior. Subtarget feature gating influences the behavior here.
- **CN**: 这里为 PowerPC 定义 TableGen 子目标特性标志，使 CPU 或 feature 字符串能够切换生成出来的行为。 子目标特性裁剪会影响这里的行为。

### Lines 356-366

```tablegen
def FeaturePredictableSelectIsExpensive :
  SubtargetFeature<"predictable-select-expensive",
                   "PredictableSelectIsExpensive",
                   "true",
                   "Prefer likely predicted branches over selects">;

def FeatureFastMFLR : SubtargetFeature<"fast-MFLR", "HasFastMFLR", "true",
                                       "MFLR is a fast instruction">;

// When enabled indirect calls will place the address of the descriptor
// into r11 and do a direct branch to the ._ptrgl routine.
```
- **EN**: Defines TableGen subtarget feature flags for PowerPC, allowing CPU or feature strings to toggle generated behavior. Subtarget feature gating influences the behavior here.
- **CN**: 这里为 PowerPC 定义 TableGen 子目标特性标志，使 CPU 或 feature 字符串能够切换生成出来的行为。 子目标特性裁剪会影响这里的行为。

### Lines 367-370

```tablegen
def FeaturePointerGlueHelper :
 SubtargetFeature<"use-ptrgl-helper", "UsePointerGlueHelper", "true",
                  "Use ._ptrgl for indirect calls">;
```
- **EN**: Defines TableGen subtarget feature flags for PowerPC, allowing CPU or feature strings to toggle generated behavior. Subtarget feature gating influences the behavior here.
- **CN**: 这里为 PowerPC 定义 TableGen 子目标特性标志，使 CPU 或 feature 字符串能够切换生成出来的行为。 子目标特性裁剪会影响这里的行为。

### Lines 371-377

```tablegen
//===----------------------------------------------------------------------===//
// PowerPC Instruction Predicate Definitions.

def IsLittleEndian : Predicate<"Subtarget->isLittleEndian()">;
def IsBigEndian : Predicate<"!Subtarget->isLittleEndian()">;
def IsPPC32 : Predicate<"!Subtarget->isPPC64()">;
def IsPPC64 : Predicate<"Subtarget->isPPC64()">;
```
- **EN**: Adds declarative TableGen records such as `IsLittleEndian`, `IsBigEndian`, `IsPPC32` that LLVM later expands into generated tables or helper code. Subtarget feature gating influences the behavior here.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `IsLittleEndian`, `IsBigEndian`, `IsPPC32`，LLVM 随后会把它们展开成生成表或辅助代码。 子目标特性裁剪会影响这里的行为。

### Lines 378-384

```tablegen
def IsBookE  : Predicate<"Subtarget->isBookE()">;
def IsNotBookE  : Predicate<"!Subtarget->isBookE()">;
def HasOnlyMSYNC : Predicate<"Subtarget->hasOnlyMSYNC()">;
def HasSYNC   : Predicate<"!Subtarget->hasOnlyMSYNC()">;
def IsPPC4xx  : Predicate<"Subtarget->isPPC4xx()">;
def IsPPC6xx  : Predicate<"Subtarget->isPPC6xx()">;
def IsE500  : Predicate<"Subtarget->isE500()">;
```
- **EN**: Adds declarative TableGen records such as `IsBookE`, `IsNotBookE`, `HasOnlyMSYNC` that LLVM later expands into generated tables or helper code. Subtarget feature gating influences the behavior here.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `IsBookE`, `IsNotBookE`, `HasOnlyMSYNC`，LLVM 随后会把它们展开成生成表或辅助代码。 子目标特性裁剪会影响这里的行为。

### Lines 385-391

```tablegen
def HasSPE  : Predicate<"Subtarget->hasSPE()">;
def HasICBT : Predicate<"Subtarget->hasICBT()">;
def HasPartwordAtomics : Predicate<"Subtarget->hasPartwordAtomics()">;
def HasQuadwordAtomics : Predicate<"Subtarget->hasQuadwordAtomics()">;
def HasBPERMD : Predicate<"Subtarget->hasBPERMD()">;
def HasExtDiv : Predicate<"Subtarget->hasExtDiv()">;
def HasFPU : Predicate<"Subtarget->hasFPU()">;
```
- **EN**: Adds declarative TableGen records such as `HasSPE`, `HasICBT`, `HasPartwordAtomics` that LLVM later expands into generated tables or helper code. Subtarget feature gating influences the behavior here.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `HasSPE`, `HasICBT`, `HasPartwordAtomics`，LLVM 随后会把它们展开成生成表或辅助代码。 子目标特性裁剪会影响这里的行为。

### Lines 392-400

```tablegen
def HasHTM : Predicate<"Subtarget->hasHTM()">;
def HasDirectMove : Predicate<"Subtarget->hasDirectMove()">;
def HasP8Crypto : Predicate<"Subtarget->hasP8Crypto()">;
def PCRelativeMemops : Predicate<"Subtarget->hasPCRelativeMemops()">;
def PrefixInstrs : Predicate<"Subtarget->hasPrefixInstrs()">;
def PairedVectorMemops : Predicate<"Subtarget->pairedVectorMemops()">;
def MMA : Predicate<"Subtarget->hasMMA()">;

// Vector support predicates
```
- **EN**: Adds declarative TableGen records such as `HasHTM`, `HasDirectMove`, `HasP8Crypto` that LLVM later expands into generated tables or helper code. Subtarget feature gating influences the behavior here.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `HasHTM`, `HasDirectMove`, `HasP8Crypto`，LLVM 随后会把它们展开成生成表或辅助代码。 子目标特性裁剪会影响这里的行为。

### Lines 401-407

```tablegen
def HasVSX : Predicate<"Subtarget->hasVSX()">;
def NoP8Vector : Predicate<"!Subtarget->hasP8Vector()">;
def HasP8Vector : Predicate<"Subtarget->hasP8Vector()">;
def HasAltivec : Predicate<"Subtarget->hasAltivec()">;
def HasP8Altivec : Predicate<"Subtarget->hasP8Altivec()">;
def NoP9Vector : Predicate<"!Subtarget->hasP9Vector()">;
def HasP9Vector : Predicate<"Subtarget->hasP9Vector()">;
```
- **EN**: Adds declarative TableGen records such as `HasVSX`, `NoP8Vector`, `HasP8Vector` that LLVM later expands into generated tables or helper code. Subtarget feature gating influences the behavior here.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `HasVSX`, `NoP8Vector`, `HasP8Vector`，LLVM 随后会把它们展开成生成表或辅助代码。 子目标特性裁剪会影响这里的行为。

### Lines 408-415

```tablegen
def NoP9Altivec : Predicate<"!Subtarget->hasP9Altivec()">;
def HasP9Altivec : Predicate<"Subtarget->hasP9Altivec()">;
def HasOnlySwappingMemOps : Predicate<"!Subtarget->hasP9Vector()">;
def NoP10Vector : Predicate<"!Subtarget->hasP10Vector()">;
def HasP10Vector : Predicate<"Subtarget->hasP10Vector()">;
def HasFutureVector : Predicate<"Subtarget->hasFutureVector()">;

// Predicates used to differenciate between different ISAs.
```
- **EN**: Adds declarative TableGen records such as `NoP9Altivec`, `HasP9Altivec`, `HasOnlySwappingMemOps` that LLVM later expands into generated tables or helper code. Subtarget feature gating influences the behavior here.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `NoP9Altivec`, `HasP9Altivec`, `HasOnlySwappingMemOps`，LLVM 随后会把它们展开成生成表或辅助代码。 子目标特性裁剪会影响这里的行为。

### Lines 416-422

```tablegen
def IsISA2_06 : Predicate<"Subtarget->isISA2_06()">;
def IsISA2_07 : Predicate<"Subtarget->isISA2_07()">;
def IsISA3_0 : Predicate<"Subtarget->isISA3_0()">;
def IsNotISA3_0 : Predicate<"!Subtarget->isISA3_0()">;
def IsISA3_1 : Predicate<"Subtarget->isISA3_1()">;
def IsNotISA3_1 : Predicate<"!Subtarget->isISA3_1()">;
def IsISAFuture : Predicate<"Subtarget->isISAFuture()">;
```
- **EN**: Adds declarative TableGen records such as `IsISA2_06`, `IsISA2_07`, `IsISA3_0` that LLVM later expands into generated tables or helper code. Subtarget feature gating influences the behavior here.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `IsISA2_06`, `IsISA2_07`, `IsISA3_0`，LLVM 随后会把它们展开成生成表或辅助代码。 子目标特性裁剪会影响这里的行为。

### Lines 423-430

```tablegen
def IsNotISAFuture : Predicate<"!Subtarget->isISAFuture()">;

// AIX assembler may not be modern enough to support some extended mne.
def ModernAs: Predicate<"!Subtarget->isAIXABI() || Subtarget->HasModernAIXAs">,
                 AssemblerPredicate<(any_of (not AIXOS), FeatureModernAIXAs)>;
def IsAIX : Predicate<"Subtarget->isAIXABI()">;
def NotAIX : Predicate<"!Subtarget->isAIXABI()">;
```
- **EN**: Adds declarative TableGen records such as `IsNotISAFuture`, `ModernAs`, `IsAIX` that LLVM later expands into generated tables or helper code. Subtarget feature gating influences the behavior here.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `IsNotISAFuture`, `ModernAs`, `IsAIX`，LLVM 随后会把它们展开成生成表或辅助代码。 子目标特性裁剪会影响这里的行为。

### Lines 431-432

```tablegen
//===----------------------------------------------------------------------===//
// HwModes
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "HwModes".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“HwModes”。

### Lines 433-455

```tablegen
//===----------------------------------------------------------------------===//

defvar PPC32 = DefaultMode;
def PPC64 : HwMode<[IsPPC64]>;

// Since new processors generally contain a superset of features of those that
// came before them, the idea is to make implementations of new processors
// less error prone and easier to read.
// Namely:
//     list<SubtargetFeature> P8InheritableFeatures = ...
//     list<SubtargetFeature> FutureProcessorAddtionalFeatures =
//         [ features that Power8 does not support but inheritable ]
//     list<SubtargetFeature> FutureProcessorSpecificFeatures =
//         [ features that Power8 does not support and not inheritable ]
//     list<SubtargetFeature> FutureProcessorInheritableFeatures =
//         !listconcat(P8InheritableFeatures, FutureProcessorAddtionalFeatures)
//     list<SubtargetFeature> FutureProcessorFeatures =
//         !listconcat(FutureProcessorInheritableFeatures,
//                     FutureProcessorSpecificFeatures)

// Makes it explicit and obvious what is new in FutureProcessor vs. Power8 as
// well as providing a single point of definition if the feature set will be
// used elsewhere.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Since new processors generally contain a superset of features of those that". Subtarget feature gating influences the behavior here.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Since new processors generally contain a superset of features of those that”。 子目标特性裁剪会影响这里的行为。

### Lines 456-493

```tablegen
def ProcessorFeatures {
  // Power7
  list<SubtargetFeature> P7InheritableFeatures = [DirectivePwr7,
                                                  FeatureAltivec,
                                                  FeatureVSX,
                                                  FeatureMFOCRF,
                                                  FeatureFCPSGN,
                                                  FeatureFSqrt,
                                                  FeatureFRE,
                                                  FeatureFRES,
                                                  FeatureFRSQRTE,
                                                  FeatureFRSQRTES,
                                                  FeatureRecipPrec,
                                                  FeatureSTFIWX,
                                                  FeatureLFIWAX,
                                                  FeatureFPRND,
                                                  FeatureFPCVT,
                                                  FeatureISEL,
                                                  FeaturePOPCNTD,
                                                  FeatureCMPB,
                                                  FeatureLDBRX,
                                                  Feature64BitSupport,
                                                  /* Feature64BitRegs, */
                                                  FeatureBPERMD,
                                                  FeatureExtDiv,
                                                  FeatureMFTB,
                                                  DeprecatedDST,
                                                  FeatureTwoConstNR,
                                                  FeatureUnalignedFloats,
                                                  FeatureISA2_06];
  list<SubtargetFeature> P7SpecificFeatures = [];
  list<SubtargetFeature> P7Features =
    !listconcat(P7InheritableFeatures, P7SpecificFeatures);

  // Power8
  list<SubtargetFeature> P8AdditionalFeatures =
    [DirectivePwr8,
     FeatureP8Altivec,
```
- **EN**: Defines TableGen subtarget feature flags for PowerPC, allowing CPU or feature strings to toggle generated behavior. Subtarget feature gating influences the behavior here.
- **CN**: 这里为 PowerPC 定义 TableGen 子目标特性标志，使 CPU 或 feature 字符串能够切换生成出来的行为。 子目标特性裁剪会影响这里的行为。

### Lines 494-531

```tablegen
     FeatureP8Vector,
     FeatureP8Crypto,
     FeatureDirectMove,
     FeatureICBT,
     FeaturePartwordAtomic,
     FeatureQuadwordAtomic,
     FeaturePredictableSelectIsExpensive,
     FeatureISA2_07,
     FeatureCRBits
    ];

  list<SubtargetFeature> P8SpecificFeatures = [FeatureAddiLoadFusion,
                                               FeatureHTM,
                                               FeatureAddisLoadFusion];
  list<SubtargetFeature> P8InheritableFeatures =
    !listconcat(P7InheritableFeatures, P8AdditionalFeatures);
  list<SubtargetFeature> P8Features =
    !listconcat(P8InheritableFeatures, P8SpecificFeatures);

  // Power9
  list<SubtargetFeature> P9AdditionalFeatures =
    [DirectivePwr9,
     FeatureP9Altivec,
     FeatureP9Vector,
     FeaturePPCPreRASched,
     FeaturePPCPostRASched,
     FeatureISA3_0
    ];

  // Some features are unique to Power9 and there is no reason to assume
  // they will be part of any future CPUs. One example is the narrower
  // dispatch for vector operations than scalar ones. For the time being,
  // this list also includes scheduling-related features since we do not have
  // enough info to create custom scheduling strategies for future CPUs.
  list<SubtargetFeature> P9SpecificFeatures = [FeatureVectorsUseTwoUnits, FeatureHTM];
  list<SubtargetFeature> P9InheritableFeatures =
    !listconcat(P8InheritableFeatures, P9AdditionalFeatures);
  list<SubtargetFeature> P9Features =
```
- **EN**: Defines TableGen subtarget feature flags for PowerPC, allowing CPU or feature strings to toggle generated behavior. Subtarget feature gating influences the behavior here.
- **CN**: 这里为 PowerPC 定义 TableGen 子目标特性标志，使 CPU 或 feature 字符串能够切换生成出来的行为。 子目标特性裁剪会影响这里的行为。

### Lines 532-569

```tablegen
    !listconcat(P9InheritableFeatures, P9SpecificFeatures);

  // Power10
  // For P10 CPU we assume that all of the existing features from Power9
  // still exist with the exception of those we know are Power9 specific.
  list<SubtargetFeature> FusionFeatures = [
    FeatureStoreFusion, FeatureAddLogicalFusion, FeatureLogicalAddFusion,
    FeatureLogicalFusion, FeatureArithAddFusion, FeatureSha3Fusion,
  ];
  list<SubtargetFeature> P10AdditionalFeatures =
    !listconcat(FusionFeatures, [
       DirectivePwr10, FeatureISA3_1, FeaturePrefixInstrs,
       FeaturePCRelativeMemops, FeatureP10Vector, FeatureMMA,
       FeaturePairedVectorMemops, FeatureFastMFLR]);
  list<SubtargetFeature> P10SpecificFeatures = [];
  list<SubtargetFeature> P10InheritableFeatures =
    !listconcat(P9InheritableFeatures, P10AdditionalFeatures);
  list<SubtargetFeature> P10Features =
    !listconcat(P10InheritableFeatures, P10SpecificFeatures);

  // Power11
  // For P11 CPU we assume that all the existing features from Power10
  // still exist with the exception of those we know are Power10 specific.
  list<SubtargetFeature> P11AdditionalFeatures =
    [DirectivePwr11];
  list<SubtargetFeature> P11SpecificFeatures =
    [];
  list<SubtargetFeature> P11InheritableFeatures =
    !listconcat(P10InheritableFeatures, P11AdditionalFeatures);
  list<SubtargetFeature> P11Features =
    !listconcat(P11InheritableFeatures, P11SpecificFeatures);

  // Future
  // For future CPU we assume that all of the existing features from Power11
  // still exist with the exception of those we know are Power11 specific.
  list<SubtargetFeature> FutureAdditionalFeatures = [DirectivePwrFuture,
                                                     FeatureISAFuture,
                                                     FeatureFutureVector];
```
- **EN**: Defines TableGen subtarget feature flags for PowerPC, allowing CPU or feature strings to toggle generated behavior. Subtarget feature gating influences the behavior here.
- **CN**: 这里为 PowerPC 定义 TableGen 子目标特性标志，使 CPU 或 feature 字符串能够切换生成出来的行为。 子目标特性裁剪会影响这里的行为。

### Lines 570-582

```tablegen
  list<SubtargetFeature> FutureSpecificFeatures = [];
  list<SubtargetFeature> FutureInheritableFeatures =
    !listconcat(P11InheritableFeatures, FutureAdditionalFeatures);
  list<SubtargetFeature> FutureFeatures =
    !listconcat(FutureInheritableFeatures, FutureSpecificFeatures);
}

// Note: Future features to add when support is extended to more
// recent ISA levels:
//
// DFP          p6, p6x, p7        decimal floating-point instructions
// POPCNTB      p5 through p7      popcntb and related instructions
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Note: Future features to add when support is extended to more". Subtarget feature gating influences the behavior here.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Note: Future features to add when support is extended to more”。 子目标特性裁剪会影响这里的行为。

### Lines 583-584

```tablegen
//===----------------------------------------------------------------------===//
// Classes used for relation maps.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Classes used for relation maps.".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Classes used for relation maps.”。

### Lines 585-591

```tablegen
//===----------------------------------------------------------------------===//
// RecFormRel - Filter class used to relate non-record-form instructions with
// their record-form variants.
class RecFormRel;

// AltVSXFMARel - Filter class used to relate the primary addend-killing VSX
// FMA instruction forms with their corresponding factor-killing forms.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "RecFormRel - Filter class used to relate non-record-form instructions with". Notable symbols in this range include `RecFormRel`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“RecFormRel - Filter class used to relate non-record-form instructions with”。 该区间中较显眼的符号包括 `RecFormRel`。

### Lines 592-595

```tablegen
class AltVSXFMARel {
  bit IsVSXFMAAlt = 0;
}
```
- **EN**: Declares a backend-facing type `AltVSXFMARel` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `AltVSXFMARel`，并勾勒出周边代码会依赖的接口或状态。

### Lines 596-597

```tablegen
//===----------------------------------------------------------------------===//
// Relation Map Definitions.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Relation Map Definitions.".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Relation Map Definitions.”。

### Lines 598-605

```tablegen
//===----------------------------------------------------------------------===//

def getRecordFormOpcode : InstrMapping {
  let FilterClass = "RecFormRel";
  // Instructions with the same BaseName and Interpretation64Bit values
  // form a row.
  let RowFields = ["BaseName", "Interpretation64Bit"];
  // Instructions with the same RC value form a column.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Instructions with the same BaseName and Interpretation64Bit values". Notable symbols in this range include `getRecordFormOpcode`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Instructions with the same BaseName and Interpretation64Bit values”。 该区间中较显眼的符号包括 `getRecordFormOpcode`。

### Lines 606-612

```tablegen
  let ColFields = ["RC"];
  // The key column are the non-record-form instructions.
  let KeyCol = ["0"];
  // Value columns RC=1
  let ValueCols = [["1"]];
}
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "The key column are the non-record-form instructions.".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“The key column are the non-record-form instructions.”。

### Lines 613-620

```tablegen
def getNonRecordFormOpcode : InstrMapping {
  let FilterClass = "RecFormRel";
  // Instructions with the same BaseName and Interpretation64Bit values
  // form a row.
  let RowFields = ["BaseName", "Interpretation64Bit"];
  // Instructions with the same RC value form a column.
  let ColFields = ["RC"];
  // The key column are the record-form instructions.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Instructions with the same BaseName and Interpretation64Bit values". Notable symbols in this range include `getNonRecordFormOpcode`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Instructions with the same BaseName and Interpretation64Bit values”。 该区间中较显眼的符号包括 `getNonRecordFormOpcode`。

### Lines 621-628

```tablegen
  let KeyCol = ["1"];
  // Value columns are RC=0
  let ValueCols = [["0"]];
}

def getAltVSXFMAOpcode : InstrMapping {
  let FilterClass = "AltVSXFMARel";
  // Instructions with the same BaseName value form a row.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Value columns are RC=0". Notable symbols in this range include `getAltVSXFMAOpcode`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Value columns are RC=0”。 该区间中较显眼的符号包括 `getAltVSXFMAOpcode`。

### Lines 629-637

```tablegen
  let RowFields = ["BaseName"];
  // Instructions with the same IsVSXFMAAlt value form a column.
  let ColFields = ["IsVSXFMAAlt"];
  // The key column are the (default) addend-killing instructions.
  let KeyCol = ["0"];
  // Value columns IsVSXFMAAlt=1
  let ValueCols = [["1"]];
}
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Instructions with the same IsVSXFMAAlt value form a column.".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Instructions with the same IsVSXFMAAlt value form a column.”。

### Lines 638-644

```tablegen
def get64BitInstrFromSignedExt32BitInstr : InstrMapping {
  let FilterClass = "SExt32To64";
  // Instructions with the same opcode.
  let RowFields = ["Inst"];
  // Instructions with the same Interpretation64Bit value form a column.
  let ColFields = ["Interpretation64Bit"];
  // The key column are not the Interpretation64Bit-form instructions.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Instructions with the same opcode.". Notable symbols in this range include `get64BitInstrFromSignedExt32BitInstr`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Instructions with the same opcode.”。 该区间中较显眼的符号包括 `get64BitInstrFromSignedExt32BitInstr`。

### Lines 645-649

```tablegen
  let KeyCol = ["0"];
  // Value columns are the Interpretation64Bit-form instructions.
  let ValueCols = [["1"]];
}
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Value columns are the Interpretation64Bit-form instructions.".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Value columns are the Interpretation64Bit-form instructions.”。

### Lines 650-651

```tablegen
//===----------------------------------------------------------------------===//
// Register File Description
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Register File Description".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Register File Description”。

### Lines 652-657

```tablegen
//===----------------------------------------------------------------------===//

include "PPCRegisterInfo.td"
include "PPCSchedule.td"
include "GISel/PPCRegisterBanks.td"
```
- **EN**: Pulls in direct dependencies required by this TableGen declarations for the backend, so later declarations can reuse LLVM infrastructure and target-specific helpers. Register assignment and register-class constraints matter here.
- **CN**: 这一段引入该该后端的 TableGen 声明所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 658-667

```tablegen
//===----------------------------------------------------------------------===//
// PowerPC processors supported.
//

def : Processor<"generic", G3Itineraries, [Directive32, FeatureHardFloat,
                                           FeatureMFTB]>;
def : ProcessorModel<"440", PPC440Model, [Directive440, FeatureISEL,
                                          FeatureFRES, FeatureFRSQRTE,
                                          FeatureICBT, FeatureBookE,
                                          FeatureMSYNC, FeatureMFTB]>;
```
- **EN**: Lists processor models and binds the default feature bundles or scheduling data attached to each model. Subtarget feature gating influences the behavior here.
- **CN**: 这一段列出处理器模型，并把默认特性集合或调度数据绑定到相应模型上。 子目标特性裁剪会影响这里的行为。

### Lines 668-674

```tablegen
def : ProcessorModel<"450", PPC440Model, [Directive440, FeatureISEL,
                                          FeatureFRES, FeatureFRSQRTE,
                                          FeatureICBT, FeatureBookE,
                                          FeatureMSYNC, FeatureMFTB]>;
def : Processor<"601", G3Itineraries, [Directive601, FeatureFPU]>;
def : Processor<"602", G3Itineraries, [Directive602, FeatureFPU,
                                       FeatureMFTB]>;
```
- **EN**: Lists processor models and binds the default feature bundles or scheduling data attached to each model. Subtarget feature gating influences the behavior here.
- **CN**: 这一段列出处理器模型，并把默认特性集合或调度数据绑定到相应模型上。 子目标特性裁剪会影响这里的行为。

### Lines 675-683

```tablegen
def : Processor<"603", G3Itineraries, [Directive603,
                                       FeatureFRES, FeatureFRSQRTE,
                                       FeatureMFTB]>;
def : Processor<"603e", G3Itineraries, [Directive603,
                                        FeatureFRES, FeatureFRSQRTE,
                                        FeatureMFTB]>;
def : Processor<"603ev", G3Itineraries, [Directive603,
                                         FeatureFRES, FeatureFRSQRTE,
                                         FeatureMFTB]>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. Subtarget feature gating influences the behavior here.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 子目标特性裁剪会影响这里的行为。

### Lines 684-692

```tablegen
def : Processor<"604", G3Itineraries, [Directive604,
                                       FeatureFRES, FeatureFRSQRTE,
                                       FeatureMFTB]>;
def : Processor<"604e", G3Itineraries, [Directive604,
                                        FeatureFRES, FeatureFRSQRTE,
                                        FeatureMFTB]>;
def : Processor<"620", G3Itineraries, [Directive620,
                                       FeatureFRES, FeatureFRSQRTE,
                                       FeatureMFTB]>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. Subtarget feature gating influences the behavior here.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 子目标特性裁剪会影响这里的行为。

### Lines 693-701

```tablegen
def : Processor<"750", G4Itineraries, [Directive750,
                                       FeatureFRES, FeatureFRSQRTE,
                                       FeatureMFTB]>;
def : Processor<"g3", G3Itineraries, [Directive750,
                                      FeatureFRES, FeatureFRSQRTE,
                                      FeatureMFTB]>;
def : Processor<"7400", G4Itineraries, [Directive7400, FeatureAltivec,
                                        FeatureFRES, FeatureFRSQRTE,
                                        FeatureMFTB]>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. Subtarget feature gating influences the behavior here.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 子目标特性裁剪会影响这里的行为。

### Lines 702-711

```tablegen
def : Processor<"g4", G4Itineraries, [Directive7400, FeatureAltivec,
                                      FeatureFRES, FeatureFRSQRTE,
                                      FeatureMFTB]>;
def : Processor<"7450", G4PlusItineraries, [Directive7400, FeatureAltivec,
                                            FeatureFRES, FeatureFRSQRTE,
                                            FeatureMFTB]>;
def : Processor<"g4+", G4PlusItineraries, [Directive7400, FeatureAltivec,
                                           FeatureFRES, FeatureFRSQRTE,
                                           FeatureMFTB]>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. Subtarget feature gating influences the behavior here.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 子目标特性裁剪会影响这里的行为。

### Lines 712-723

```tablegen
def : ProcessorModel<"970", G5Model,
                  [Directive970, FeatureAltivec,
                   FeatureMFOCRF, FeatureFSqrt,
                   FeatureFRES, FeatureFRSQRTE, FeatureSTFIWX,
                   Feature64BitSupport /*, Feature64BitRegs */,
                   FeatureMFTB]>;
def : ProcessorModel<"g5", G5Model,
                  [Directive970, FeatureAltivec,
                   FeatureMFOCRF, FeatureFSqrt, FeatureSTFIWX,
                   FeatureFRES, FeatureFRSQRTE,
                   Feature64BitSupport /*, Feature64BitRegs */,
                   FeatureMFTB, DeprecatedDST]>;
```
- **EN**: Lists processor models and binds the default feature bundles or scheduling data attached to each model. Subtarget feature gating influences the behavior here.
- **CN**: 这一段列出处理器模型，并把默认特性集合或调度数据绑定到相应模型上。 子目标特性裁剪会影响这里的行为。

### Lines 724-731

```tablegen
def : ProcessorModel<"e500", PPCE500Model,
                  [DirectiveE500,
                   FeatureICBT, FeatureBookE,
                   FeatureISEL, FeatureMFTB, FeatureMSYNC, FeatureSPE]>;
def : ProcessorModel<"e500mc", PPCE500mcModel,
                  [DirectiveE500mc,
                   FeatureSTFIWX, FeatureICBT, FeatureBookE,
                   FeatureISEL, FeatureMFTB]>;
```
- **EN**: Lists processor models and binds the default feature bundles or scheduling data attached to each model. Subtarget feature gating influences the behavior here.
- **CN**: 这一段列出处理器模型，并把默认特性集合或调度数据绑定到相应模型上。 子目标特性裁剪会影响这里的行为。

### Lines 732-744

```tablegen
def : ProcessorModel<"e5500", PPCE5500Model,
                  [DirectiveE5500, FeatureMFOCRF, Feature64BitSupport,
                   FeatureSTFIWX, FeatureICBT, FeatureBookE,
                   FeatureISEL, FeatureMFTB]>;
def : ProcessorModel<"a2", PPCA2Model,
                  [DirectiveA2, FeatureICBT, FeatureBookE, FeatureMFOCRF,
                   FeatureFCPSGN, FeatureFSqrt, FeatureFRE, FeatureFRES,
                   FeatureFRSQRTE, FeatureFRSQRTES, FeatureRecipPrec,
                   FeatureSTFIWX, FeatureLFIWAX,
                   FeatureFPRND, FeatureFPCVT, FeatureISEL,
                   FeatureSlowPOPCNTD, FeatureCMPB, FeatureLDBRX,
                   Feature64BitSupport /*, Feature64BitRegs */, FeatureMFTB,
                   FeatureISA2_06]>;
```
- **EN**: Lists processor models and binds the default feature bundles or scheduling data attached to each model. Subtarget feature gating influences the behavior here.
- **CN**: 这一段列出处理器模型，并把默认特性集合或调度数据绑定到相应模型上。 子目标特性裁剪会影响这里的行为。

### Lines 745-752

```tablegen
def : ProcessorModel<"pwr3", G5Model,
                  [DirectivePwr3, FeatureAltivec,
                   FeatureFRES, FeatureFRSQRTE, FeatureMFOCRF,
                   FeatureSTFIWX, Feature64BitSupport]>;
def : ProcessorModel<"pwr4", G5Model,
                  [DirectivePwr4, FeatureAltivec, FeatureMFOCRF,
                   FeatureFSqrt, FeatureFRES, FeatureFRSQRTE,
                   FeatureSTFIWX, Feature64BitSupport, FeatureMFTB]>;
```
- **EN**: Lists processor models and binds the default feature bundles or scheduling data attached to each model. Subtarget feature gating influences the behavior here.
- **CN**: 这一段列出处理器模型，并把默认特性集合或调度数据绑定到相应模型上。 子目标特性裁剪会影响这里的行为。

### Lines 753-764

```tablegen
def : ProcessorModel<"pwr5", G5Model,
                  [DirectivePwr5, FeatureAltivec, FeatureMFOCRF,
                   FeatureFSqrt, FeatureFRE, FeatureFRES,
                   FeatureFRSQRTE, FeatureFRSQRTES,
                   FeatureSTFIWX, Feature64BitSupport,
                   FeatureMFTB, DeprecatedDST]>;
def : ProcessorModel<"pwr5x", G5Model,
                  [DirectivePwr5x, FeatureAltivec, FeatureMFOCRF,
                   FeatureFSqrt, FeatureFRE, FeatureFRES,
                   FeatureFRSQRTE, FeatureFRSQRTES,
                   FeatureSTFIWX, FeatureFPRND, Feature64BitSupport,
                   FeatureMFTB, DeprecatedDST]>;
```
- **EN**: Lists processor models and binds the default feature bundles or scheduling data attached to each model. Subtarget feature gating influences the behavior here.
- **CN**: 这一段列出处理器模型，并把默认特性集合或调度数据绑定到相应模型上。 子目标特性裁剪会影响这里的行为。

### Lines 765-771

```tablegen
def : ProcessorModel<"pwr6", G5Model,
                  [DirectivePwr6, FeatureAltivec,
                   FeatureMFOCRF, FeatureFCPSGN, FeatureFSqrt, FeatureFRE,
                   FeatureFRES, FeatureFRSQRTE, FeatureFRSQRTES,
                   FeatureRecipPrec, FeatureSTFIWX, FeatureLFIWAX, FeatureCMPB,
                   FeatureFPRND, Feature64BitSupport /*, Feature64BitRegs */,
                   FeatureMFTB, DeprecatedDST]>;
```
- **EN**: Lists processor models and binds the default feature bundles or scheduling data attached to each model. Subtarget feature gating influences the behavior here.
- **CN**: 这一段列出处理器模型，并把默认特性集合或调度数据绑定到相应模型上。 子目标特性裁剪会影响这里的行为。

### Lines 772-778

```tablegen
def : ProcessorModel<"pwr6x", G5Model,
                  [DirectivePwr5x, FeatureAltivec, FeatureMFOCRF,
                   FeatureFCPSGN, FeatureFSqrt, FeatureFRE, FeatureFRES,
                   FeatureFRSQRTE, FeatureFRSQRTES, FeatureRecipPrec,
                   FeatureSTFIWX, FeatureLFIWAX, FeatureCMPB,
                   FeatureFPRND, Feature64BitSupport,
                   FeatureMFTB, DeprecatedDST]>;
```
- **EN**: Lists processor models and binds the default feature bundles or scheduling data attached to each model. Subtarget feature gating influences the behavior here.
- **CN**: 这一段列出处理器模型，并把默认特性集合或调度数据绑定到相应模型上。 子目标特性裁剪会影响这里的行为。

### Lines 779-786

```tablegen
def : ProcessorModel<"pwr7", P7Model, ProcessorFeatures.P7Features>;
def : ProcessorModel<"pwr8", P8Model, ProcessorFeatures.P8Features>;
def : ProcessorModel<"pwr9", P9Model, ProcessorFeatures.P9Features>;
def : ProcessorModel<"pwr10", P10Model, ProcessorFeatures.P10Features>;
def : ProcessorModel<"pwr11", P10Model, ProcessorFeatures.P11Features>;
// No scheduler model for future CPU.
def : ProcessorModel<"future", NoSchedModel,
                  ProcessorFeatures.FutureFeatures>;
```
- **EN**: Lists processor models and binds the default feature bundles or scheduling data attached to each model. Subtarget feature gating influences the behavior here.
- **CN**: 这一段列出处理器模型，并把默认特性集合或调度数据绑定到相应模型上。 子目标特性裁剪会影响这里的行为。

### Lines 787-796

```tablegen
def : Processor<"ppc", G3Itineraries, [Directive32, FeatureHardFloat,
                                       FeatureMFTB]>;
def : Processor<"ppc32", G3Itineraries, [Directive32, FeatureHardFloat,
                                         FeatureMFTB]>;
def : ProcessorModel<"ppc64", G5Model,
                  [Directive64, FeatureAltivec,
                   FeatureMFOCRF, FeatureFSqrt, FeatureFRES,
                   FeatureFRSQRTE, FeatureSTFIWX,
                   Feature64BitSupport /*, Feature64BitRegs */,
                   FeatureMFTB]>;
```
- **EN**: Lists processor models and binds the default feature bundles or scheduling data attached to each model. Subtarget feature gating influences the behavior here.
- **CN**: 这一段列出处理器模型，并把默认特性集合或调度数据绑定到相应模型上。 子目标特性裁剪会影响这里的行为。

### Lines 797-798

```tablegen
def : ProcessorModel<"ppc64le", P8Model, ProcessorFeatures.P8Features>;
```
- **EN**: Lists processor models and binds the default feature bundles or scheduling data attached to each model. Subtarget feature gating influences the behavior here.
- **CN**: 这一段列出处理器模型，并把默认特性集合或调度数据绑定到相应模型上。 子目标特性裁剪会影响这里的行为。

### Lines 799-800

```tablegen
//===----------------------------------------------------------------------===//
// Calling Conventions
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Calling Conventions".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Calling Conventions”。

### Lines 801-808

```tablegen
//===----------------------------------------------------------------------===//

include "PPCCallingConv.td"

def PPCInstrInfo : InstrInfo {
  let isLittleEndianEncoding = 1;
}
```
- **EN**: Pulls in direct dependencies required by this TableGen declarations for the backend, so later declarations can reuse LLVM infrastructure and target-specific helpers. ABI and calling-convention details are important in this part of the code.
- **CN**: 这一段引入该该后端的 TableGen 声明所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 ABI 与调用约定细节是这一部分代码的重要约束。

### Lines 809-815

```tablegen
def PPCAsmWriter : AsmWriter {
  string AsmWriterClassName  = "InstPrinter";
  int PassSubtarget = 1;
  int Variant = 0;
  bit isMCAsmWriter = 1;
}
```
- **EN**: Adds declarative TableGen records such as `PPCAsmWriter` that LLVM later expands into generated tables or helper code. Subtarget feature gating influences the behavior here.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `PPCAsmWriter`，LLVM 随后会把它们展开成生成表或辅助代码。 子目标特性裁剪会影响这里的行为。

### Lines 816-830

```tablegen
def PPCAsmParser : AsmParser {
  let ShouldEmitMatchRegisterName = 1;
  let AllowDuplicateRegisterNames = 1;
}

def PPCAsmParserVariant : AsmParserVariant {
  int Variant = 0;

  // We do not use hard coded registers in asm strings.  However, some
  // InstAlias definitions use immediate literals.  Set RegisterPrefix
  // so that those are not misinterpreted as registers.
  string RegisterPrefix = "%";
  string BreakCharacters = ".";
}
```
- **EN**: Adds declarative TableGen records such as `PPCAsmParser`, `PPCAsmParserVariant` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `PPCAsmParser`, `PPCAsmParserVariant`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 831-837

```tablegen
defm : RemapAllTargetPseudoPointerOperands<ppc_ptr_rc>;

def PPC : Target {
  // Information about the instructions.
  let InstructionSet = PPCInstrInfo;

  let AssemblyWriters = [PPCAsmWriter];
```
- **EN**: Adds declarative TableGen records such as `PPC` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `PPC`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 838-842

```tablegen
  let AssemblyParsers = [PPCAsmParser];
  let AssemblyParserVariants = [PPCAsmParserVariant];
  let AllowRegisterRenaming = 1;
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 843-844

```tablegen
//===----------------------------------------------------------------------===//
// Pfm Counters
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Pfm Counters".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Pfm Counters”。

### Lines 845-847

```tablegen
//===----------------------------------------------------------------------===//

include "PPCPfmCounters.td"
```
- **EN**: Pulls in direct dependencies required by this TableGen declarations for the backend, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该该后端的 TableGen 声明所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

## Key Concepts / 关键概念

- PowerPC backend integration / PowerPC 后端集成
- TableGen declarations for the backend / 该后端的 TableGen 声明
- LLVM TableGen DSL / LLVM TableGen 领域特定语言
- Calling convention handling / 调用约定处理
- Register modeling / 寄存器建模
- Subtarget features / 子目标特性
- Assembly parsing / 汇编解析
- Instruction selection or opcode handling / 指令选择或操作码处理

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `llvm/Target/Target.td`
- `PPCRegisterInfo.td`
- `PPCSchedule.td`
- `GISel/PPCRegisterBanks.td`
- `PPCCallingConv.td`
- `PPCPfmCounters.td`

### Important Collaborators / 重要协作组件

- LLVM MC layer / LLVM MC 层
- Generated backend tables / 生成式后端表
- PowerPC target-specific helpers / PowerPC 目标专用辅助组件
