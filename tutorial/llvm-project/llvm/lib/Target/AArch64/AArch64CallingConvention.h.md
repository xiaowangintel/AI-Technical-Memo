# AArch64CallingConvention.h — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/AArch64/AArch64CallingConvention.h`
- **Repository**: llvm/llvm-project
- **Purpose**: This file declares the entry points for AArch64 calling convention analysis. / 该文件实现 AArch64 后端中的调用约定分析。
## Line-by-Line Analysis / 逐行分析
### Lines 1-16: Documented code section
```cpp
//=== AArch64CallingConvention.h - AArch64 CC entry points ------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file declares the entry points for AArch64 calling convention analysis.
//
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_AARCH64_AARCH64CALLINGCONVENTION_H
#define LLVM_LIB_TARGET_AARCH64_AARCH64CALLINGCONVENTION_H

#include "llvm/CodeGen/CallingConvLower.h"
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
### Lines 17-34: Namespace llvm
```cpp

namespace llvm {
bool CC_AArch64_AAPCS(unsigned ValNo, MVT ValVT, MVT LocVT,
                      CCValAssign::LocInfo LocInfo, ISD::ArgFlagsTy ArgFlags,
                      Type *OrigTy, CCState &State);
bool CC_AArch64_Arm64EC_VarArg(unsigned ValNo, MVT ValVT, MVT LocVT,
                               CCValAssign::LocInfo LocInfo,
                               ISD::ArgFlagsTy ArgFlags, Type *OrigTy,
                               CCState &State);
bool CC_AArch64_Arm64EC_Thunk(unsigned ValNo, MVT ValVT, MVT LocVT,
                              CCValAssign::LocInfo LocInfo,
                              ISD::ArgFlagsTy ArgFlags, Type *OrigTy,
                              CCState &State);
bool CC_AArch64_Arm64EC_Thunk_Native(unsigned ValNo, MVT ValVT, MVT LocVT,
                                     CCValAssign::LocInfo LocInfo,
                                     ISD::ArgFlagsTy ArgFlags, Type *OrigTy,
                                     CCState &State);
bool CC_AArch64_DarwinPCS_VarArg(unsigned ValNo, MVT ValVT, MVT LocVT,
```
**EN:** This block scopes the following declarations to the intended namespace boundary.  
**CN:** 该代码块将后续声明限定在目标命名空间边界内。
### Lines 35-52: Core AArch64 backend logic
```cpp
                                 CCValAssign::LocInfo LocInfo,
                                 ISD::ArgFlagsTy ArgFlags, Type *OrigTy,
                                 CCState &State);
bool CC_AArch64_DarwinPCS(unsigned ValNo, MVT ValVT, MVT LocVT,
                          CCValAssign::LocInfo LocInfo,
                          ISD::ArgFlagsTy ArgFlags, Type *OrigTy,
                          CCState &State);
bool CC_AArch64_DarwinPCS_ILP32_VarArg(unsigned ValNo, MVT ValVT, MVT LocVT,
                                       CCValAssign::LocInfo LocInfo,
                                       ISD::ArgFlagsTy ArgFlags, Type *OrigTy,
                                       CCState &State);
bool CC_AArch64_Win64PCS(unsigned ValNo, MVT ValVT, MVT LocVT,
                         CCValAssign::LocInfo LocInfo, ISD::ArgFlagsTy ArgFlags,
                         Type *OrigTy, CCState &State);
bool CC_AArch64_Win64_VarArg(unsigned ValNo, MVT ValVT, MVT LocVT,
                             CCValAssign::LocInfo LocInfo,
                             ISD::ArgFlagsTy ArgFlags, Type *OrigTy,
                             CCState &State);
```
**EN:** This block continues the file's main calling-convention analysis logic and connects local helpers with the wider AArch64 backend.  
**CN:** 该代码块延续本文件的调用约定分析主线逻辑，并把本地辅助实现连接到更广泛的 AArch64 后端中。
### Lines 53-70: Core AArch64 backend logic
```cpp
bool CC_AArch64_Win64_CFGuard_Check(unsigned ValNo, MVT ValVT, MVT LocVT,
                                    CCValAssign::LocInfo LocInfo,
                                    ISD::ArgFlagsTy ArgFlags, Type *OrigTy,
                                    CCState &State);
bool CC_AArch64_Arm64EC_CFGuard_Check(unsigned ValNo, MVT ValVT, MVT LocVT,
                                      CCValAssign::LocInfo LocInfo,
                                      ISD::ArgFlagsTy ArgFlags, Type *OrigTy,
                                      CCState &State);
bool CC_AArch64_GHC(unsigned ValNo, MVT ValVT, MVT LocVT,
                    CCValAssign::LocInfo LocInfo, ISD::ArgFlagsTy ArgFlags,
                    Type *OrigTy, CCState &State);
bool CC_AArch64_Preserve_None(unsigned ValNo, MVT ValVT, MVT LocVT,
                              CCValAssign::LocInfo LocInfo,
                              ISD::ArgFlagsTy ArgFlags, Type *OrigTy,
                              CCState &State);
bool RetCC_AArch64_AAPCS(unsigned ValNo, MVT ValVT, MVT LocVT,
                         CCValAssign::LocInfo LocInfo, ISD::ArgFlagsTy ArgFlags,
                         Type *OrigTy, CCState &State);
```
**EN:** This block continues the file's main calling-convention analysis logic and connects local helpers with the wider AArch64 backend.  
**CN:** 该代码块延续本文件的调用约定分析主线逻辑，并把本地辅助实现连接到更广泛的 AArch64 后端中。
### Lines 71-81: Core AArch64 backend logic
```cpp
bool RetCC_AArch64_Arm64EC_Thunk(unsigned ValNo, MVT ValVT, MVT LocVT,
                                 CCValAssign::LocInfo LocInfo,
                                 ISD::ArgFlagsTy ArgFlags, Type *OrigTy,
                                 CCState &State);
bool RetCC_AArch64_Arm64EC_CFGuard_Check(unsigned ValNo, MVT ValVT, MVT LocVT,
                                         CCValAssign::LocInfo LocInfo,
                                         ISD::ArgFlagsTy ArgFlags, Type *OrigTy,
                                         CCState &State);
} // namespace llvm

#endif
```
**EN:** This block continues the file's main calling-convention analysis logic and connects local helpers with the wider AArch64 backend.  
**CN:** 该代码块延续本文件的调用约定分析主线逻辑，并把本地辅助实现连接到更广泛的 AArch64 后端中。
## Key Concepts / 关键概念
- **EN:** AArch64 target backend structure **CN:** AArch64 目标后端结构
- **EN:** ABI-specific argument and return assignment **CN:** 特定 ABI 的参数与返回值分配
## Dependencies / 依赖关系
- **EN:** Core LLVM interfaces: llvm/CodeGen/CallingConvLower.h **CN:** 核心 LLVM 接口：llvm/CodeGen/CallingConvLower.h
- **EN:** Closely connected with neighboring AArch64 backend components responsible for calling-convention analysis. **CN:** 与周边负责调用约定分析的 AArch64 后端组件紧密协作。
