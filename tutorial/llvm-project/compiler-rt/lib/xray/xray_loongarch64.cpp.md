# xray_loongarch64.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `compiler-rt/lib/xray/xray_loongarch64.cpp`
- **Repository**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose (EN)**: This file is a part of XRay, a dynamic runtime instrumentation system.
- **目的（中文）**: 该实现文件提供与 `XRay loongarch64` 相关的运行时逻辑。

## Line-by-Line Analysis / 逐行分析
### Line 1
````cpp
//===-------- xray_loongarch64.cpp ------------------------------*- C++ -*-===//
````
- **EN**: Banner comment delimiting a file header or major section.
- **CN**: 横幅注释，用于分隔文件头或主要章节。

### Line 2
````cpp
//
````
- **EN**: Comment formatting line in a banner or block comment.
- **CN**: 注释格式行，用于组织文件头或注释块。

### Line 3
````cpp
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
````
- **EN**: Comment documenting `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`.
- **CN**: 注释说明了 `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。

### Line 4
````cpp
// See https://llvm.org/LICENSE.txt for license information.
````
- **EN**: Comment documenting `See https://llvm.org/LICENSE.txt for license information.`.
- **CN**: 注释说明了 `See https://llvm.org/LICENSE.txt for license information.`。

### Line 5
````cpp
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
````
- **EN**: Comment documenting `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`.
- **CN**: 注释说明了 `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。

### Line 6
````cpp
//
````
- **EN**: Comment formatting line in a banner or block comment.
- **CN**: 注释格式行，用于组织文件头或注释块。

### Line 7
````cpp
//===----------------------------------------------------------------------===//
````
- **EN**: Banner comment delimiting a file header or major section.
- **CN**: 横幅注释，用于分隔文件头或主要章节。

### Line 8
````cpp
//
````
- **EN**: Comment formatting line in a banner or block comment.
- **CN**: 注释格式行，用于组织文件头或注释块。

### Line 9
````cpp
// This file is a part of XRay, a dynamic runtime instrumentation system.
````
- **EN**: Comment documenting `This file is a part of XRay, a dynamic runtime instrumentation system.`.
- **CN**: 注释说明了 `This file is a part of XRay, a dynamic runtime instrumentation system.`。

### Line 10
````cpp
//
````
- **EN**: Comment formatting line in a banner or block comment.
- **CN**: 注释格式行，用于组织文件头或注释块。

### Line 11
````cpp
// Implementation of loongarch-specific routines.
````
- **EN**: Comment documenting `Implementation of loongarch-specific routines.`.
- **CN**: 注释说明了 `Implementation of loongarch-specific routines.`。

### Line 12
````cpp
//
````
- **EN**: Comment formatting line in a banner or block comment.
- **CN**: 注释格式行，用于组织文件头或注释块。

### Line 13
````cpp
//===----------------------------------------------------------------------===//
````
- **EN**: Banner comment delimiting a file header or major section.
- **CN**: 横幅注释，用于分隔文件头或主要章节。

### Line 14
````cpp
#include "sanitizer_common/sanitizer_common.h"
````
- **EN**: Includes the local dependency `sanitizer_common/sanitizer_common.h`.
- **CN**: 引入本地依赖 `sanitizer_common/sanitizer_common.h`。

### Line 15
````cpp
#include "xray_defs.h"
````
- **EN**: Includes the local dependency `xray_defs.h`.
- **CN**: 引入本地依赖 `xray_defs.h`。

### Line 16
````cpp
#include "xray_interface_internal.h"
````
- **EN**: Includes the local dependency `xray_interface_internal.h`.
- **CN**: 引入本地依赖 `xray_interface_internal.h`。

### Line 17
````cpp
#include <atomic>
````
- **EN**: Includes the system dependency `atomic`.
- **CN**: 引入系统依赖 `atomic`。

### Line 18
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 19
````cpp
namespace __xray {
````
- **EN**: Opens namespace `__xray`.
- **CN**: 打开命名空间 `__xray`。

### Line 20
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 21
````cpp
enum RegNum : uint32_t {
````
- **EN**: Declares the enum `RegNum`.
- **CN**: 声明 enum `RegNum`。

### Line 22
````cpp
  RN_RA = 1,
````
- **EN**: Carries part of the local implementation logic: `RN_RA = 1,`.
- **CN**: 承载局部实现逻辑：`RN_RA = 1,`。

### Line 23
````cpp
  RN_SP = 3,
````
- **EN**: Carries part of the local implementation logic: `RN_SP = 3,`.
- **CN**: 承载局部实现逻辑：`RN_SP = 3,`。

### Line 24
````cpp
  RN_T0 = 12,
````
- **EN**: Carries part of the local implementation logic: `RN_T0 = 12,`.
- **CN**: 承载局部实现逻辑：`RN_T0 = 12,`。

### Line 25
````cpp
  RN_T1 = 13,
````
- **EN**: Carries part of the local implementation logic: `RN_T1 = 13,`.
- **CN**: 承载局部实现逻辑：`RN_T1 = 13,`。

### Line 26
````cpp
};
````
- **EN**: Closes a type definition or aggregate block.
- **CN**: 结束一个类型定义或聚合块。

### Line 27
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 28
````cpp
// Encode instructions in the 2RIx format, where the primary formats here
````
- **EN**: Comment documenting `Encode instructions in the 2RIx format, where the primary formats here`.
- **CN**: 注释说明了 `Encode instructions in the 2RIx format, where the primary formats here`。

### Line 29
````cpp
// are 2RI12-type and 2RI16-type.
````
- **EN**: Comment documenting `are 2RI12-type and 2RI16-type.`.
- **CN**: 注释说明了 `are 2RI12-type and 2RI16-type.`。

### Line 30
````cpp
static inline uint32_t
````
- **EN**: Carries part of the local implementation logic: `static inline uint32_t`.
- **CN**: 承载局部实现逻辑：`static inline uint32_t`。

### Line 31
````cpp
encodeInstruction2RIx(uint32_t Opcode, uint32_t Rd, uint32_t Rj,
````
- **EN**: Carries part of the local implementation logic: `encodeInstruction2RIx(uint32_t Opcode, uint32_t Rd, uint32_t Rj,`.
- **CN**: 承载局部实现逻辑：`encodeInstruction2RIx(uint32_t Opcode, uint32_t Rd, uint32_t Rj,`。

### Line 32
````cpp
                      uint32_t Imm) XRAY_NEVER_INSTRUMENT {
````
- **EN**: Carries part of the local implementation logic: `uint32_t Imm) XRAY_NEVER_INSTRUMENT {`.
- **CN**: 承载局部实现逻辑：`uint32_t Imm) XRAY_NEVER_INSTRUMENT {`。

### Line 33
````cpp
  return Opcode | (Imm << 10) | (Rj << 5) | Rd;
````
- **EN**: Returns from the current function with `Opcode | (Imm << 10) | (Rj << 5) | Rd;`.
- **CN**: 使用 `Opcode | (Imm << 10) | (Rj << 5) | Rd;` 从当前函数返回。

### Line 34
````cpp
}
````
- **EN**: Closes the current scope block.
- **CN**: 结束当前作用域块。

### Line 35
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 36
````cpp
// Encode instructions in 1RI20 format, e.g. lu12i.w/lu32i.d.
````
- **EN**: Comment documenting `Encode instructions in 1RI20 format, e.g. lu12i.w/lu32i.d.`.
- **CN**: 注释说明了 `Encode instructions in 1RI20 format, e.g. lu12i.w/lu32i.d.`。

### Line 37
````cpp
static inline uint32_t
````
- **EN**: Carries part of the local implementation logic: `static inline uint32_t`.
- **CN**: 承载局部实现逻辑：`static inline uint32_t`。

### Line 38
````cpp
encodeInstruction1RI20(uint32_t Opcode, uint32_t Rd,
````
- **EN**: Carries part of the local implementation logic: `encodeInstruction1RI20(uint32_t Opcode, uint32_t Rd,`.
- **CN**: 承载局部实现逻辑：`encodeInstruction1RI20(uint32_t Opcode, uint32_t Rd,`。

### Line 39
````cpp
                       uint32_t Imm) XRAY_NEVER_INSTRUMENT {
````
- **EN**: Carries part of the local implementation logic: `uint32_t Imm) XRAY_NEVER_INSTRUMENT {`.
- **CN**: 承载局部实现逻辑：`uint32_t Imm) XRAY_NEVER_INSTRUMENT {`。

### Line 40
````cpp
  return Opcode | (Imm << 5) | Rd;
````
- **EN**: Returns from the current function with `Opcode | (Imm << 5) | Rd;`.
- **CN**: 使用 `Opcode | (Imm << 5) | Rd;` 从当前函数返回。

### Line 41
````cpp
}
````
- **EN**: Closes the current scope block.
- **CN**: 结束当前作用域块。

### Line 42
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 43
````cpp
static inline bool patchSled(const bool Enable, const uint32_t FuncId,
````
- **EN**: Carries part of the local implementation logic: `static inline bool patchSled(const bool Enable, const uint32_t FuncId,`.
- **CN**: 承载局部实现逻辑：`static inline bool patchSled(const bool Enable, const uint32_t FuncId,`。

### Line 44
````cpp
                             const XRaySledEntry &Sled,
````
- **EN**: Carries part of the local implementation logic: `const XRaySledEntry &Sled,`.
- **CN**: 承载局部实现逻辑：`const XRaySledEntry &Sled,`。

### Line 45
````cpp
                             void (*TracingHook)()) XRAY_NEVER_INSTRUMENT {
````
- **EN**: Carries part of the local implementation logic: `void (*TracingHook)()) XRAY_NEVER_INSTRUMENT {`.
- **CN**: 承载局部实现逻辑：`void (*TracingHook)()) XRAY_NEVER_INSTRUMENT {`。

### Line 46
````cpp
  // When |Enable| == true,
````
- **EN**: Comment documenting `When |Enable| == true,`.
- **CN**: 注释说明了 `When |Enable| == true,`。

### Line 47
````cpp
  // We replace the following compile-time stub (sled):
````
- **EN**: Comment documenting `We replace the following compile-time stub (sled):`.
- **CN**: 注释说明了 `We replace the following compile-time stub (sled):`。

### Line 48
````cpp
  //
````
- **EN**: Comment formatting line in a banner or block comment.
- **CN**: 注释格式行，用于组织文件头或注释块。

### Line 49
````cpp
  // .Lxray_sled_beginN:
````
- **EN**: Comment documenting `.Lxray_sled_beginN:`.
- **CN**: 注释说明了 `.Lxray_sled_beginN:`。

### Line 50
````cpp
  //	B .Lxray_sled_endN
````
- **EN**: Comment documenting `B .Lxray_sled_endN`.
- **CN**: 注释说明了 `B .Lxray_sled_endN`。

### Line 51
````cpp
  //	11 NOPs (44 bytes)
````
- **EN**: Comment documenting `11 NOPs (44 bytes)`.
- **CN**: 注释说明了 `11 NOPs (44 bytes)`。

### Line 52
````cpp
  // .Lxray_sled_endN:
````
- **EN**: Comment documenting `.Lxray_sled_endN:`.
- **CN**: 注释说明了 `.Lxray_sled_endN:`。

### Line 53
````cpp
  //
````
- **EN**: Comment formatting line in a banner or block comment.
- **CN**: 注释格式行，用于组织文件头或注释块。

### Line 54
````cpp
  // With the following runtime patch:
````
- **EN**: Comment documenting `With the following runtime patch:`.
- **CN**: 注释说明了 `With the following runtime patch:`。

### Line 55
````cpp
  //
````
- **EN**: Comment formatting line in a banner or block comment.
- **CN**: 注释格式行，用于组织文件头或注释块。

### Line 56
````cpp
  // xray_sled_n:
````
- **EN**: Comment documenting `xray_sled_n:`.
- **CN**: 注释说明了 `xray_sled_n:`。

### Line 57
````cpp
  //   addi.d  sp, sp, -16                       ; create the stack frame
````
- **EN**: Comment documenting `addi.d  sp, sp, -16                       ; create the stack frame`.
- **CN**: 注释说明了 `addi.d  sp, sp, -16                       ; create the stack frame`。

### Line 58
````cpp
  //   st.d    ra, sp, 8                         ; save the return address
````
- **EN**: Comment documenting `st.d    ra, sp, 8                         ; save the return address`.
- **CN**: 注释说明了 `st.d    ra, sp, 8                         ; save the return address`。

### Line 59
````cpp
  //   lu12i.w t0, %abs_hi20(__xray_FunctionEntry/Exit)
````
- **EN**: Comment documenting `lu12i.w t0, %abs_hi20(__xray_FunctionEntry/Exit)`.
- **CN**: 注释说明了 `lu12i.w t0, %abs_hi20(__xray_FunctionEntry/Exit)`。

### Line 60
````cpp
  //   ori     t0, t0, %abs_lo12(__xray_FunctionEntry/Exit)
````
- **EN**: Comment documenting `ori     t0, t0, %abs_lo12(__xray_FunctionEntry/Exit)`.
- **CN**: 注释说明了 `ori     t0, t0, %abs_lo12(__xray_FunctionEntry/Exit)`。

### Line 61
````cpp
  //   lu32i.d t0, %abs64_lo20(__xray_FunctionEntry/Exit)
````
- **EN**: Comment documenting `lu32i.d t0, %abs64_lo20(__xray_FunctionEntry/Exit)`.
- **CN**: 注释说明了 `lu32i.d t0, %abs64_lo20(__xray_FunctionEntry/Exit)`。

### Line 62
````cpp
  //   lu52i.d t0, t0, %abs64_hi12(__xray_FunctionEntry/Exit)
````
- **EN**: Comment documenting `lu52i.d t0, t0, %abs64_hi12(__xray_FunctionEntry/Exit)`.
- **CN**: 注释说明了 `lu52i.d t0, t0, %abs64_hi12(__xray_FunctionEntry/Exit)`。

### Line 63
````cpp
  //   lu12i.w t1, %abs_hi20(function_id)
````
- **EN**: Comment documenting `lu12i.w t1, %abs_hi20(function_id)`.
- **CN**: 注释说明了 `lu12i.w t1, %abs_hi20(function_id)`。

### Line 64
````cpp
  //   ori     t1, t1, %abs_lo12(function_id)    ; pass the function id
````
- **EN**: Comment documenting `ori     t1, t1, %abs_lo12(function_id)    ; pass the function id`.
- **CN**: 注释说明了 `ori     t1, t1, %abs_lo12(function_id)    ; pass the function id`。

### Line 65
````cpp
  //   jirl    ra, t0, 0                         ; call the tracing hook
````
- **EN**: Comment documenting `jirl    ra, t0, 0                         ; call the tracing hook`.
- **CN**: 注释说明了 `jirl    ra, t0, 0                         ; call the tracing hook`。

### Line 66
````cpp
  //   ld.d    ra, sp, 8                         ; restore the return address
````
- **EN**: Comment documenting `ld.d    ra, sp, 8                         ; restore the return address`.
- **CN**: 注释说明了 `ld.d    ra, sp, 8                         ; restore the return address`。

### Line 67
````cpp
  //   addi.d  sp, sp, 16                        ; de-allocate the stack frame
````
- **EN**: Comment documenting `addi.d  sp, sp, 16                        ; de-allocate the stack frame`.
- **CN**: 注释说明了 `addi.d  sp, sp, 16                        ; de-allocate the stack frame`。

### Line 68
````cpp
  //
````
- **EN**: Comment formatting line in a banner or block comment.
- **CN**: 注释格式行，用于组织文件头或注释块。

### Line 69
````cpp
  // Replacement of the first 4-byte instruction should be the last and atomic
````
- **EN**: Comment documenting `Replacement of the first 4-byte instruction should be the last and atomic`.
- **CN**: 注释说明了 `Replacement of the first 4-byte instruction should be the last and atomic`。

### Line 70
````cpp
  // operation, so that the user code which reaches the sled concurrently
````
- **EN**: Comment documenting `operation, so that the user code which reaches the sled concurrently`.
- **CN**: 注释说明了 `operation, so that the user code which reaches the sled concurrently`。

### Line 71
````cpp
  // either jumps over the whole sled, or executes the whole sled when the
````
- **EN**: Comment documenting `either jumps over the whole sled, or executes the whole sled when the`.
- **CN**: 注释说明了 `either jumps over the whole sled, or executes the whole sled when the`。

### Line 72
````cpp
  // latter is ready.
````
- **EN**: Comment documenting `latter is ready.`.
- **CN**: 注释说明了 `latter is ready.`。

### Line 73
````cpp
  //
````
- **EN**: Comment formatting line in a banner or block comment.
- **CN**: 注释格式行，用于组织文件头或注释块。

### Line 74
````cpp
  // When |Enable|==false, we set the first instruction in the sled back to
````
- **EN**: Comment documenting `When |Enable|==false, we set the first instruction in the sled back to`.
- **CN**: 注释说明了 `When |Enable|==false, we set the first instruction in the sled back to`。

### Line 75
````cpp
  //   B #48
````
- **EN**: Comment documenting `B #48`.
- **CN**: 注释说明了 `B #48`。

### Line 76
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 77
````cpp
  uint32_t *Address = reinterpret_cast<uint32_t *>(Sled.address());
````
- **EN**: Declares an interface element or prototype: `uint32_t *Address = reinterpret_cast<uint32_t *>(Sled.address());`.
- **CN**: 声明一个接口元素或原型：`uint32_t *Address = reinterpret_cast<uint32_t *>(Sled.address());`。

### Line 78
````cpp
  if (Enable) {
````
- **EN**: Evaluates the conditional branch `if (Enable) {`.
- **CN**: 计算条件分支 `if (Enable) {`。

### Line 79
````cpp
    uint32_t LoTracingHookAddr = reinterpret_cast<int64_t>(TracingHook) & 0xfff;
````
- **EN**: Declares an interface element or prototype: `uint32_t LoTracingHookAddr = reinterpret_cast<int64_t>(TracingHook) & 0xfff;`.
- **CN**: 声明一个接口元素或原型：`uint32_t LoTracingHookAddr = reinterpret_cast<int64_t>(TracingHook) & 0xfff;`。

### Line 80
````cpp
    uint32_t HiTracingHookAddr =
````
- **EN**: Carries part of the local implementation logic: `uint32_t HiTracingHookAddr =`.
- **CN**: 承载局部实现逻辑：`uint32_t HiTracingHookAddr =`。

### Line 81
````cpp
        (reinterpret_cast<int64_t>(TracingHook) >> 12) & 0xfffff;
````
- **EN**: Invokes a function-like statement: `(reinterpret_cast<int64_t>(TracingHook) >> 12) & 0xfffff;`.
- **CN**: 调用一个类似函数的语句：`(reinterpret_cast<int64_t>(TracingHook) >> 12) & 0xfffff;`。

### Line 82
````cpp
    uint32_t HigherTracingHookAddr =
````
- **EN**: Carries part of the local implementation logic: `uint32_t HigherTracingHookAddr =`.
- **CN**: 承载局部实现逻辑：`uint32_t HigherTracingHookAddr =`。

### Line 83
````cpp
        (reinterpret_cast<int64_t>(TracingHook) >> 32) & 0xfffff;
````
- **EN**: Invokes a function-like statement: `(reinterpret_cast<int64_t>(TracingHook) >> 32) & 0xfffff;`.
- **CN**: 调用一个类似函数的语句：`(reinterpret_cast<int64_t>(TracingHook) >> 32) & 0xfffff;`。

### Line 84
````cpp
    uint32_t HighestTracingHookAddr =
````
- **EN**: Carries part of the local implementation logic: `uint32_t HighestTracingHookAddr =`.
- **CN**: 承载局部实现逻辑：`uint32_t HighestTracingHookAddr =`。

### Line 85
````cpp
        (reinterpret_cast<int64_t>(TracingHook) >> 52) & 0xfff;
````
- **EN**: Invokes a function-like statement: `(reinterpret_cast<int64_t>(TracingHook) >> 52) & 0xfff;`.
- **CN**: 调用一个类似函数的语句：`(reinterpret_cast<int64_t>(TracingHook) >> 52) & 0xfff;`。

### Line 86
````cpp
    uint32_t LoFunctionID = FuncId & 0xfff;
````
- **EN**: Assigns or initializes state with `uint32_t LoFunctionID = FuncId & 0xfff;`.
- **CN**: 使用 `uint32_t LoFunctionID = FuncId & 0xfff;` 进行赋值或初始化。

### Line 87
````cpp
    uint32_t HiFunctionID = (FuncId >> 12) & 0xfffff;
````
- **EN**: Declares an interface element or prototype: `uint32_t HiFunctionID = (FuncId >> 12) & 0xfffff;`.
- **CN**: 声明一个接口元素或原型：`uint32_t HiFunctionID = (FuncId >> 12) & 0xfffff;`。

### Line 88
````cpp
    Address[1] = encodeInstruction2RIx(0x29c00000, RegNum::RN_RA, RegNum::RN_SP,
````
- **EN**: Carries part of the local implementation logic: `Address[1] = encodeInstruction2RIx(0x29c00000, RegNum::RN_RA, RegNum::RN_SP,`.
- **CN**: 承载局部实现逻辑：`Address[1] = encodeInstruction2RIx(0x29c00000, RegNum::RN_RA, RegNum::RN_SP,`。

### Line 89
````cpp
                                       0x8); // st.d ra, sp, 8
````
- **EN**: Carries part of the local implementation logic: `0x8); // st.d ra, sp, 8`.
- **CN**: 承载局部实现逻辑：`0x8); // st.d ra, sp, 8`。

### Line 90
````cpp
    Address[2] = encodeInstruction1RI20(
````
- **EN**: Carries part of the local implementation logic: `Address[2] = encodeInstruction1RI20(`.
- **CN**: 承载局部实现逻辑：`Address[2] = encodeInstruction1RI20(`。

### Line 91
````cpp
        0x14000000, RegNum::RN_T0,
````
- **EN**: Carries part of the local implementation logic: `0x14000000, RegNum::RN_T0,`.
- **CN**: 承载局部实现逻辑：`0x14000000, RegNum::RN_T0,`。

### Line 92
````cpp
        HiTracingHookAddr); // lu12i.w t0, HiTracingHookAddr
````
- **EN**: Carries part of the local implementation logic: `HiTracingHookAddr); // lu12i.w t0, HiTracingHookAddr`.
- **CN**: 承载局部实现逻辑：`HiTracingHookAddr); // lu12i.w t0, HiTracingHookAddr`。

### Line 93
````cpp
    Address[3] = encodeInstruction2RIx(
````
- **EN**: Carries part of the local implementation logic: `Address[3] = encodeInstruction2RIx(`.
- **CN**: 承载局部实现逻辑：`Address[3] = encodeInstruction2RIx(`。

### Line 94
````cpp
        0x03800000, RegNum::RN_T0, RegNum::RN_T0,
````
- **EN**: Carries part of the local implementation logic: `0x03800000, RegNum::RN_T0, RegNum::RN_T0,`.
- **CN**: 承载局部实现逻辑：`0x03800000, RegNum::RN_T0, RegNum::RN_T0,`。

### Line 95
````cpp
        LoTracingHookAddr); // ori t0, t0, LoTracingHookAddr
````
- **EN**: Carries part of the local implementation logic: `LoTracingHookAddr); // ori t0, t0, LoTracingHookAddr`.
- **CN**: 承载局部实现逻辑：`LoTracingHookAddr); // ori t0, t0, LoTracingHookAddr`。

### Line 96
````cpp
    Address[4] = encodeInstruction1RI20(
````
- **EN**: Carries part of the local implementation logic: `Address[4] = encodeInstruction1RI20(`.
- **CN**: 承载局部实现逻辑：`Address[4] = encodeInstruction1RI20(`。

### Line 97
````cpp
        0x16000000, RegNum::RN_T0,
````
- **EN**: Carries part of the local implementation logic: `0x16000000, RegNum::RN_T0,`.
- **CN**: 承载局部实现逻辑：`0x16000000, RegNum::RN_T0,`。

### Line 98
````cpp
        HigherTracingHookAddr); // lu32i.d t0, HigherTracingHookAddr
````
- **EN**: Carries part of the local implementation logic: `HigherTracingHookAddr); // lu32i.d t0, HigherTracingHookAddr`.
- **CN**: 承载局部实现逻辑：`HigherTracingHookAddr); // lu32i.d t0, HigherTracingHookAddr`。

### Line 99
````cpp
    Address[5] = encodeInstruction2RIx(
````
- **EN**: Carries part of the local implementation logic: `Address[5] = encodeInstruction2RIx(`.
- **CN**: 承载局部实现逻辑：`Address[5] = encodeInstruction2RIx(`。

### Line 100
````cpp
        0x03000000, RegNum::RN_T0, RegNum::RN_T0,
````
- **EN**: Carries part of the local implementation logic: `0x03000000, RegNum::RN_T0, RegNum::RN_T0,`.
- **CN**: 承载局部实现逻辑：`0x03000000, RegNum::RN_T0, RegNum::RN_T0,`。

### Line 101
````cpp
        HighestTracingHookAddr); // lu52i.d t0, t0, HighestTracingHookAddr
````
- **EN**: Carries part of the local implementation logic: `HighestTracingHookAddr); // lu52i.d t0, t0, HighestTracingHookAddr`.
- **CN**: 承载局部实现逻辑：`HighestTracingHookAddr); // lu52i.d t0, t0, HighestTracingHookAddr`。

### Line 102
````cpp
    Address[6] =
````
- **EN**: Carries part of the local implementation logic: `Address[6] =`.
- **CN**: 承载局部实现逻辑：`Address[6] =`。

### Line 103
````cpp
        encodeInstruction1RI20(0x14000000, RegNum::RN_T1,
````
- **EN**: Carries part of the local implementation logic: `encodeInstruction1RI20(0x14000000, RegNum::RN_T1,`.
- **CN**: 承载局部实现逻辑：`encodeInstruction1RI20(0x14000000, RegNum::RN_T1,`。

### Line 104
````cpp
                               HiFunctionID); // lu12i.w t1, HiFunctionID
````
- **EN**: Carries part of the local implementation logic: `HiFunctionID); // lu12i.w t1, HiFunctionID`.
- **CN**: 承载局部实现逻辑：`HiFunctionID); // lu12i.w t1, HiFunctionID`。

### Line 105
````cpp
    Address[7] =
````
- **EN**: Carries part of the local implementation logic: `Address[7] =`.
- **CN**: 承载局部实现逻辑：`Address[7] =`。

### Line 106
````cpp
        encodeInstruction2RIx(0x03800000, RegNum::RN_T1, RegNum::RN_T1,
````
- **EN**: Carries part of the local implementation logic: `encodeInstruction2RIx(0x03800000, RegNum::RN_T1, RegNum::RN_T1,`.
- **CN**: 承载局部实现逻辑：`encodeInstruction2RIx(0x03800000, RegNum::RN_T1, RegNum::RN_T1,`。

### Line 107
````cpp
                              LoFunctionID); // ori t1, t1, LoFunctionID
````
- **EN**: Carries part of the local implementation logic: `LoFunctionID); // ori t1, t1, LoFunctionID`.
- **CN**: 承载局部实现逻辑：`LoFunctionID); // ori t1, t1, LoFunctionID`。

### Line 108
````cpp
    Address[8] = encodeInstruction2RIx(0x4c000000, RegNum::RN_RA, RegNum::RN_T0,
````
- **EN**: Carries part of the local implementation logic: `Address[8] = encodeInstruction2RIx(0x4c000000, RegNum::RN_RA, RegNum::RN_T0,`.
- **CN**: 承载局部实现逻辑：`Address[8] = encodeInstruction2RIx(0x4c000000, RegNum::RN_RA, RegNum::RN_T0,`。

### Line 109
````cpp
                                       0); // jirl ra, t0, 0
````
- **EN**: Carries part of the local implementation logic: `0); // jirl ra, t0, 0`.
- **CN**: 承载局部实现逻辑：`0); // jirl ra, t0, 0`。

### Line 110
````cpp
    Address[9] = encodeInstruction2RIx(0x28c00000, RegNum::RN_RA, RegNum::RN_SP,
````
- **EN**: Carries part of the local implementation logic: `Address[9] = encodeInstruction2RIx(0x28c00000, RegNum::RN_RA, RegNum::RN_SP,`.
- **CN**: 承载局部实现逻辑：`Address[9] = encodeInstruction2RIx(0x28c00000, RegNum::RN_RA, RegNum::RN_SP,`。

### Line 111
````cpp
                                       0x8); // ld.d ra, sp, 8
````
- **EN**: Carries part of the local implementation logic: `0x8); // ld.d ra, sp, 8`.
- **CN**: 承载局部实现逻辑：`0x8); // ld.d ra, sp, 8`。

### Line 112
````cpp
    Address[10] = encodeInstruction2RIx(
````
- **EN**: Carries part of the local implementation logic: `Address[10] = encodeInstruction2RIx(`.
- **CN**: 承载局部实现逻辑：`Address[10] = encodeInstruction2RIx(`。

### Line 113
````cpp
        0x02c00000, RegNum::RN_SP, RegNum::RN_SP, 0x10); // addi.d sp, sp, 16
````
- **EN**: Carries part of the local implementation logic: `0x02c00000, RegNum::RN_SP, RegNum::RN_SP, 0x10); // addi.d sp, sp, 16`.
- **CN**: 承载局部实现逻辑：`0x02c00000, RegNum::RN_SP, RegNum::RN_SP, 0x10); // addi.d sp, sp, 16`。

### Line 114
````cpp
    uint32_t CreateStackSpace = encodeInstruction2RIx(
````
- **EN**: Carries part of the local implementation logic: `uint32_t CreateStackSpace = encodeInstruction2RIx(`.
- **CN**: 承载局部实现逻辑：`uint32_t CreateStackSpace = encodeInstruction2RIx(`。

### Line 115
````cpp
        0x02c00000, RegNum::RN_SP, RegNum::RN_SP, 0xff0); // addi.d sp, sp, -16
````
- **EN**: Carries part of the local implementation logic: `0x02c00000, RegNum::RN_SP, RegNum::RN_SP, 0xff0); // addi.d sp, sp, -16`.
- **CN**: 承载局部实现逻辑：`0x02c00000, RegNum::RN_SP, RegNum::RN_SP, 0xff0); // addi.d sp, sp, -16`。

### Line 116
````cpp
    std::atomic_store_explicit(
````
- **EN**: Carries part of the local implementation logic: `std::atomic_store_explicit(`.
- **CN**: 承载局部实现逻辑：`std::atomic_store_explicit(`。

### Line 117
````cpp
        reinterpret_cast<std::atomic<uint32_t> *>(Address), CreateStackSpace,
````
- **EN**: Carries part of the local implementation logic: `reinterpret_cast<std::atomic<uint32_t> *>(Address), CreateStackSpace,`.
- **CN**: 承载局部实现逻辑：`reinterpret_cast<std::atomic<uint32_t> *>(Address), CreateStackSpace,`。

### Line 118
````cpp
        std::memory_order_release);
````
- **EN**: Executes or declares `std::memory_order_release);` within the current scope.
- **CN**: 在当前作用域中执行或声明 `std::memory_order_release);`。

### Line 119
````cpp
  } else {
````
- **EN**: Carries part of the local implementation logic: `} else {`.
- **CN**: 承载局部实现逻辑：`} else {`。

### Line 120
````cpp
    std::atomic_store_explicit(
````
- **EN**: Carries part of the local implementation logic: `std::atomic_store_explicit(`.
- **CN**: 承载局部实现逻辑：`std::atomic_store_explicit(`。

### Line 121
````cpp
        reinterpret_cast<std::atomic<uint32_t> *>(Address),
````
- **EN**: Carries part of the local implementation logic: `reinterpret_cast<std::atomic<uint32_t> *>(Address),`.
- **CN**: 承载局部实现逻辑：`reinterpret_cast<std::atomic<uint32_t> *>(Address),`。

### Line 122
````cpp
        uint32_t(0x50003000), std::memory_order_release); // b #48
````
- **EN**: Carries part of the local implementation logic: `uint32_t(0x50003000), std::memory_order_release); // b #48`.
- **CN**: 承载局部实现逻辑：`uint32_t(0x50003000), std::memory_order_release); // b #48`。

### Line 123
````cpp
  }
````
- **EN**: Closes the current scope block.
- **CN**: 结束当前作用域块。

### Line 124
````cpp
  return true;
````
- **EN**: Returns from the current function with `true;`.
- **CN**: 使用 `true;` 从当前函数返回。

### Line 125
````cpp
}
````
- **EN**: Closes the current scope block.
- **CN**: 结束当前作用域块。

### Line 126
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 127
````cpp
bool patchFunctionEntry(const bool Enable, const uint32_t FuncId,
````
- **EN**: Carries part of the local implementation logic: `bool patchFunctionEntry(const bool Enable, const uint32_t FuncId,`.
- **CN**: 承载局部实现逻辑：`bool patchFunctionEntry(const bool Enable, const uint32_t FuncId,`。

### Line 128
````cpp
                        const XRaySledEntry &Sled,
````
- **EN**: Carries part of the local implementation logic: `const XRaySledEntry &Sled,`.
- **CN**: 承载局部实现逻辑：`const XRaySledEntry &Sled,`。

### Line 129
````cpp
                        const XRayTrampolines &Trampolines,
````
- **EN**: Carries part of the local implementation logic: `const XRayTrampolines &Trampolines,`.
- **CN**: 承载局部实现逻辑：`const XRayTrampolines &Trampolines,`。

### Line 130
````cpp
                        bool LogArgs) XRAY_NEVER_INSTRUMENT {
````
- **EN**: Carries part of the local implementation logic: `bool LogArgs) XRAY_NEVER_INSTRUMENT {`.
- **CN**: 承载局部实现逻辑：`bool LogArgs) XRAY_NEVER_INSTRUMENT {`。

### Line 131
````cpp
  auto Trampoline =
````
- **EN**: Carries part of the local implementation logic: `auto Trampoline =`.
- **CN**: 承载局部实现逻辑：`auto Trampoline =`。

### Line 132
````cpp
      LogArgs ? Trampolines.LogArgsTrampoline : Trampolines.EntryTrampoline;
````
- **EN**: Executes or declares `LogArgs ? Trampolines.LogArgsTrampoline : Trampolines.EntryTrampoline;` within the current scope.
- **CN**: 在当前作用域中执行或声明 `LogArgs ? Trampolines.LogArgsTrampoline : Trampolines.EntryTrampoline;`。

### Line 133
````cpp
  return patchSled(Enable, FuncId, Sled, Trampoline);
````
- **EN**: Returns from the current function with `patchSled(Enable, FuncId, Sled, Trampoline);`.
- **CN**: 使用 `patchSled(Enable, FuncId, Sled, Trampoline);` 从当前函数返回。

### Line 134
````cpp
}
````
- **EN**: Closes the current scope block.
- **CN**: 结束当前作用域块。

### Line 135
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 136
````cpp
bool patchFunctionExit(
````
- **EN**: Carries part of the local implementation logic: `bool patchFunctionExit(`.
- **CN**: 承载局部实现逻辑：`bool patchFunctionExit(`。

### Line 137
````cpp
    const bool Enable, const uint32_t FuncId, const XRaySledEntry &Sled,
````
- **EN**: Carries part of the local implementation logic: `const bool Enable, const uint32_t FuncId, const XRaySledEntry &Sled,`.
- **CN**: 承载局部实现逻辑：`const bool Enable, const uint32_t FuncId, const XRaySledEntry &Sled,`。

### Line 138
````cpp
    const XRayTrampolines &Trampolines) XRAY_NEVER_INSTRUMENT {
````
- **EN**: Carries part of the local implementation logic: `const XRayTrampolines &Trampolines) XRAY_NEVER_INSTRUMENT {`.
- **CN**: 承载局部实现逻辑：`const XRayTrampolines &Trampolines) XRAY_NEVER_INSTRUMENT {`。

### Line 139
````cpp
  return patchSled(Enable, FuncId, Sled, Trampolines.ExitTrampoline);
````
- **EN**: Returns from the current function with `patchSled(Enable, FuncId, Sled, Trampolines.ExitTrampoline);`.
- **CN**: 使用 `patchSled(Enable, FuncId, Sled, Trampolines.ExitTrampoline);` 从当前函数返回。

### Line 140
````cpp
}
````
- **EN**: Closes the current scope block.
- **CN**: 结束当前作用域块。

### Line 141
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 142
````cpp
bool patchFunctionTailExit(
````
- **EN**: Carries part of the local implementation logic: `bool patchFunctionTailExit(`.
- **CN**: 承载局部实现逻辑：`bool patchFunctionTailExit(`。

### Line 143
````cpp
    const bool Enable, const uint32_t FuncId, const XRaySledEntry &Sled,
````
- **EN**: Carries part of the local implementation logic: `const bool Enable, const uint32_t FuncId, const XRaySledEntry &Sled,`.
- **CN**: 承载局部实现逻辑：`const bool Enable, const uint32_t FuncId, const XRaySledEntry &Sled,`。

### Line 144
````cpp
    const XRayTrampolines &Trampolines) XRAY_NEVER_INSTRUMENT {
````
- **EN**: Carries part of the local implementation logic: `const XRayTrampolines &Trampolines) XRAY_NEVER_INSTRUMENT {`.
- **CN**: 承载局部实现逻辑：`const XRayTrampolines &Trampolines) XRAY_NEVER_INSTRUMENT {`。

### Line 145
````cpp
  // TODO: In the future we'd need to distinguish between non-tail exits and
````
- **EN**: Comment recording follow-up work: `TODO: In the future we'd need to distinguish between non-tail exits and`.
- **CN**: 注释记录后续待办事项：`TODO: In the future we'd need to distinguish between non-tail exits and`。

### Line 146
````cpp
  // tail exits for better information preservation.
````
- **EN**: Comment documenting `tail exits for better information preservation.`.
- **CN**: 注释说明了 `tail exits for better information preservation.`。

### Line 147
````cpp
  return patchSled(Enable, FuncId, Sled, Trampolines.ExitTrampoline);
````
- **EN**: Returns from the current function with `patchSled(Enable, FuncId, Sled, Trampolines.ExitTrampoline);`.
- **CN**: 使用 `patchSled(Enable, FuncId, Sled, Trampolines.ExitTrampoline);` 从当前函数返回。

### Line 148
````cpp
}
````
- **EN**: Closes the current scope block.
- **CN**: 结束当前作用域块。

### Line 149
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 150
````cpp
bool patchCustomEvent(const bool Enable, const uint32_t FuncId,
````
- **EN**: Carries part of the local implementation logic: `bool patchCustomEvent(const bool Enable, const uint32_t FuncId,`.
- **CN**: 承载局部实现逻辑：`bool patchCustomEvent(const bool Enable, const uint32_t FuncId,`。

### Line 151
````cpp
                      const XRaySledEntry &Sled) XRAY_NEVER_INSTRUMENT {
````
- **EN**: Carries part of the local implementation logic: `const XRaySledEntry &Sled) XRAY_NEVER_INSTRUMENT {`.
- **CN**: 承载局部实现逻辑：`const XRaySledEntry &Sled) XRAY_NEVER_INSTRUMENT {`。

### Line 152
````cpp
  // FIXME: Implement in loongarch?
````
- **EN**: Comment recording follow-up work: `FIXME: Implement in loongarch?`.
- **CN**: 注释记录后续待办事项：`FIXME: Implement in loongarch?`。

### Line 153
````cpp
  return false;
````
- **EN**: Returns from the current function with `false;`.
- **CN**: 使用 `false;` 从当前函数返回。

### Line 154
````cpp
}
````
- **EN**: Closes the current scope block.
- **CN**: 结束当前作用域块。

### Line 155
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 156
````cpp
bool patchTypedEvent(const bool Enable, const uint32_t FuncId,
````
- **EN**: Carries part of the local implementation logic: `bool patchTypedEvent(const bool Enable, const uint32_t FuncId,`.
- **CN**: 承载局部实现逻辑：`bool patchTypedEvent(const bool Enable, const uint32_t FuncId,`。

### Line 157
````cpp
                     const XRaySledEntry &Sled) XRAY_NEVER_INSTRUMENT {
````
- **EN**: Carries part of the local implementation logic: `const XRaySledEntry &Sled) XRAY_NEVER_INSTRUMENT {`.
- **CN**: 承载局部实现逻辑：`const XRaySledEntry &Sled) XRAY_NEVER_INSTRUMENT {`。

### Line 158
````cpp
  // FIXME: Implement in loongarch?
````
- **EN**: Comment recording follow-up work: `FIXME: Implement in loongarch?`.
- **CN**: 注释记录后续待办事项：`FIXME: Implement in loongarch?`。

### Line 159
````cpp
  return false;
````
- **EN**: Returns from the current function with `false;`.
- **CN**: 使用 `false;` 从当前函数返回。

### Line 160
````cpp
}
````
- **EN**: Closes the current scope block.
- **CN**: 结束当前作用域块。

### Line 161
````cpp
} // namespace __xray
````
- **EN**: Closes namespace `__xray`.
- **CN**: 关闭命名空间 `__xray`。

### Line 162
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 163
````cpp
extern "C" void __xray_ArgLoggerEntry() XRAY_NEVER_INSTRUMENT {
````
- **EN**: Declares C linkage for the following interface: `extern "C" void __xray_ArgLoggerEntry() XRAY_NEVER_INSTRUMENT {`.
- **CN**: 为后续接口声明 C 语言链接：`extern "C" void __xray_ArgLoggerEntry() XRAY_NEVER_INSTRUMENT {`。

### Line 164
````cpp
  // TODO: This will have to be implemented in the trampoline assembly file.
````
- **EN**: Comment recording follow-up work: `TODO: This will have to be implemented in the trampoline assembly file.`.
- **CN**: 注释记录后续待办事项：`TODO: This will have to be implemented in the trampoline assembly file.`。

### Line 165
````cpp
}
````
- **EN**: Closes the current scope block.
- **CN**: 结束当前作用域块。

### Line 166
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 167
````cpp
extern "C" void __xray_FunctionTailExit() XRAY_NEVER_INSTRUMENT {
````
- **EN**: Declares C linkage for the following interface: `extern "C" void __xray_FunctionTailExit() XRAY_NEVER_INSTRUMENT {`.
- **CN**: 为后续接口声明 C 语言链接：`extern "C" void __xray_FunctionTailExit() XRAY_NEVER_INSTRUMENT {`。

### Line 168
````cpp
  // FIXME: this will have to be implemented in the trampoline assembly file
````
- **EN**: Comment recording follow-up work: `FIXME: this will have to be implemented in the trampoline assembly file`.
- **CN**: 注释记录后续待办事项：`FIXME: this will have to be implemented in the trampoline assembly file`。

### Line 169
````cpp
}
````
- **EN**: Closes the current scope block.
- **CN**: 结束当前作用域块。

## Key Concepts / 关键概念
- Namespaces / 命名空间
- Types and interfaces / 类型与接口
- Function logic / 函数逻辑
- Memory management / 内存管理
- Threading and synchronization / 线程与同步

## Dependencies / 依赖关系
- **Local headers / 本地头文件**: `sanitizer_common/sanitizer_common.h`, `xray_defs.h`, `xray_interface_internal.h`
- **System headers / 系统头文件**: `atomic`
