# AMDGPULaneMaskUtils.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/AMDGPULaneMaskUtils.h`
- **Repository**: llvm/llvm-project
- **Purpose**: This header declares the interfaces, data structures, and pass entry points for AMDGPULaneMaskUtils in the LLVM backend utilities. It defines the contracts consumed by other AMDGPU backend components. / 该头文件声明 LLVM 后端工具中 AMDGPULaneMaskUtils 的接口、数据结构与 Pass 入口。它定义了其他 AMDGPU 后端组件依赖的契约。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20: File banner, includes, and setup
```cpp
//===- AMDGPULaneMaskUtils.h - Exec/lane mask helper functions -*- C++ -*--===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_AMDGPU_UTILS_AMDGPULANEMASKUTILS_H
#define LLVM_LIB_TARGET_AMDGPU_UTILS_AMDGPULANEMASKUTILS_H

#include "GCNSubtarget.h"
#include "llvm/CodeGen/Register.h"

namespace llvm {

class GCNSubtarget;

namespace AMDGPU {

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation. Main symbols: `GCNSubtarget`.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。 主要符号：`GCNSubtarget`。

### Lines 21-44: Declares class LaneMaskConstants
```cpp
class LaneMaskConstants {
public:
  const Register ExecReg;
  const Register VccReg;
  const unsigned AndOpc;
  const unsigned AndTermOpc;
  const unsigned AndN2Opc;
  const unsigned AndN2SaveExecOpc;
  const unsigned AndN2TermOpc;
  const unsigned AndSaveExecOpc;
  const unsigned AndSaveExecTermOpc;
  const unsigned BfmOpc;
  const unsigned CMovOpc;
  const unsigned CSelectOpc;
  const unsigned MovOpc;
  const unsigned MovTermOpc;
  const unsigned OrOpc;
  const unsigned OrN2Opc;
  const unsigned OrTermOpc;
  const unsigned OrSaveExecOpc;
  const unsigned XorOpc;
  const unsigned XorTermOpc;
  const unsigned WQMOpc;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `LaneMaskConstants`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`LaneMaskConstants`。

### Lines 45-68: Defines LaneMaskConstants
```cpp
  constexpr LaneMaskConstants(bool IsWave32)
      : ExecReg(IsWave32 ? AMDGPU::EXEC_LO : AMDGPU::EXEC),
        VccReg(IsWave32 ? AMDGPU::VCC_LO : AMDGPU::VCC),
        AndOpc(IsWave32 ? AMDGPU::S_AND_B32 : AMDGPU::S_AND_B64),
        AndTermOpc(IsWave32 ? AMDGPU::S_AND_B32_term : AMDGPU::S_AND_B64_term),
        AndN2Opc(IsWave32 ? AMDGPU::S_ANDN2_B32 : AMDGPU::S_ANDN2_B64),
        AndN2SaveExecOpc(IsWave32 ? AMDGPU::S_ANDN2_SAVEEXEC_B32
                                  : AMDGPU::S_ANDN2_SAVEEXEC_B64),
        AndN2TermOpc(IsWave32 ? AMDGPU::S_ANDN2_B32_term
                              : AMDGPU::S_ANDN2_B64_term),
        AndSaveExecOpc(IsWave32 ? AMDGPU::S_AND_SAVEEXEC_B32
                                : AMDGPU::S_AND_SAVEEXEC_B64),
        AndSaveExecTermOpc(IsWave32 ? AMDGPU::S_AND_SAVEEXEC_B32_term
                                    : AMDGPU::S_AND_SAVEEXEC_B64_term),
        BfmOpc(IsWave32 ? AMDGPU::S_BFM_B32 : AMDGPU::S_BFM_B64),
        CMovOpc(IsWave32 ? AMDGPU::S_CMOV_B32 : AMDGPU::S_CMOV_B64),
        CSelectOpc(IsWave32 ? AMDGPU::S_CSELECT_B32 : AMDGPU::S_CSELECT_B64),
        MovOpc(IsWave32 ? AMDGPU::S_MOV_B32 : AMDGPU::S_MOV_B64),
        MovTermOpc(IsWave32 ? AMDGPU::S_MOV_B32_term : AMDGPU::S_MOV_B64_term),
        OrOpc(IsWave32 ? AMDGPU::S_OR_B32 : AMDGPU::S_OR_B64),
        OrN2Opc(IsWave32 ? AMDGPU::S_ORN2_B32 : AMDGPU::S_ORN2_B64),
        OrTermOpc(IsWave32 ? AMDGPU::S_OR_B32_term : AMDGPU::S_OR_B64_term),
        OrSaveExecOpc(IsWave32 ? AMDGPU::S_OR_SAVEEXEC_B32
                               : AMDGPU::S_OR_SAVEEXEC_B64),
```
**EN:** This section contains concrete logic for LaneMaskConstants. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 LaneMaskConstants 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 69-91: Preprocessor guards and macros
```cpp
        XorOpc(IsWave32 ? AMDGPU::S_XOR_B32 : AMDGPU::S_XOR_B64),
        XorTermOpc(IsWave32 ? AMDGPU::S_XOR_B32_term : AMDGPU::S_XOR_B64_term),
        WQMOpc(IsWave32 ? AMDGPU::S_WQM_B32 : AMDGPU::S_WQM_B64) {}

  static inline const LaneMaskConstants &get(const GCNSubtarget &ST);
};

static constexpr LaneMaskConstants LaneMaskConstants32 =
    LaneMaskConstants(/*IsWave32=*/true);
static constexpr LaneMaskConstants LaneMaskConstants64 =
    LaneMaskConstants(/*IsWave32=*/false);

inline const LaneMaskConstants &LaneMaskConstants::get(const GCNSubtarget &ST) {
  unsigned WavefrontSize = ST.getWavefrontSize();
  assert(WavefrontSize == 32 || WavefrontSize == 64);
  return WavefrontSize == 32 ? LaneMaskConstants32 : LaneMaskConstants64;
}

} // end namespace AMDGPU

} // end namespace llvm

#endif // LLVM_LIB_TARGET_AMDGPU_UTILS_AMDGPULANEMASKUTILS_H
```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled. Main symbols: `LaneMaskConstants::get`.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。 主要符号：`LaneMaskConstants::get`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ header
- **Primary symbols / 主要符号**: `GCNSubtarget`, `LaneMaskConstants`, `LaneMaskConstants::get`
- **Main themes / 核心主题**: register management / 寄存器管理; subtarget modeling / 子目标建模
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"GCNSubtarget.h"`
- `"llvm/CodeGen/Register.h"`
