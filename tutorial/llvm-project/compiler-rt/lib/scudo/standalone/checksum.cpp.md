# checksum.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `compiler-rt/lib/scudo/standalone/checksum.cpp`
- **Repository**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose (EN)**: i386 and x86_64 specific code to detect CRC32 hardware support via CPUID. CRC32 requires the SSE 4.2 instruction set.
- **目的（中文）**: 该实现文件提供与 `checksum` 相关的运行时逻辑。

## Line-by-Line Analysis / 逐行分析
### Line 1
````cpp
//===-- checksum.cpp --------------------------------------------*- C++ -*-===//
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
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 9
````cpp
#include "checksum.h"
````
- **EN**: Includes the local dependency `checksum.h`.
- **CN**: 引入本地依赖 `checksum.h`。

### Line 10
````cpp
#include "atomic_helpers.h"
````
- **EN**: Includes the local dependency `atomic_helpers.h`.
- **CN**: 引入本地依赖 `atomic_helpers.h`。

### Line 11
````cpp
#include "chunk.h"
````
- **EN**: Includes the local dependency `chunk.h`.
- **CN**: 引入本地依赖 `chunk.h`。

### Line 12
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 13
````cpp
#if defined(__x86_64__) || defined(__i386__)
````
- **EN**: Starts a preprocessor condition: `#if defined(__x86_64__) || defined(__i386__)`.
- **CN**: 开始一个预处理条件：`#if defined(__x86_64__) || defined(__i386__)`。

### Line 14
````cpp
#include <cpuid.h>
````
- **EN**: Includes the system dependency `cpuid.h`.
- **CN**: 引入系统依赖 `cpuid.h`。

### Line 15
````cpp
#elif defined(__arm__) || defined(__aarch64__)
````
- **EN**: Checks an alternate preprocessor branch: `#elif defined(__arm__) || defined(__aarch64__)`.
- **CN**: 检查预处理器的备用分支：`#elif defined(__arm__) || defined(__aarch64__)`。

### Line 16
````cpp
#if SCUDO_FUCHSIA
````
- **EN**: Starts a preprocessor condition: `#if SCUDO_FUCHSIA`.
- **CN**: 开始一个预处理条件：`#if SCUDO_FUCHSIA`。

### Line 17
````cpp
#include <zircon/features.h>
````
- **EN**: Includes the system dependency `zircon/features.h`.
- **CN**: 引入系统依赖 `zircon/features.h`。

### Line 18
````cpp
#include <zircon/syscalls.h>
````
- **EN**: Includes the system dependency `zircon/syscalls.h`.
- **CN**: 引入系统依赖 `zircon/syscalls.h`。

### Line 19
````cpp
#else
````
- **EN**: Starts the fallback branch of the current preprocessor condition.
- **CN**: 开始当前预处理条件的后备分支。

### Line 20
````cpp
#include <sys/auxv.h>
````
- **EN**: Includes the system dependency `sys/auxv.h`.
- **CN**: 引入系统依赖 `sys/auxv.h`。

### Line 21
````cpp
#endif
````
- **EN**: Ends the current preprocessor condition.
- **CN**: 结束当前的预处理条件。

### Line 22
````cpp
#elif defined(__loongarch__)
````
- **EN**: Checks an alternate preprocessor branch: `#elif defined(__loongarch__)`.
- **CN**: 检查预处理器的备用分支：`#elif defined(__loongarch__)`。

### Line 23
````cpp
#include <sys/auxv.h>
````
- **EN**: Includes the system dependency `sys/auxv.h`.
- **CN**: 引入系统依赖 `sys/auxv.h`。

### Line 24
````cpp
#endif
````
- **EN**: Ends the current preprocessor condition.
- **CN**: 结束当前的预处理条件。

### Line 25
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 26
````cpp
namespace scudo {
````
- **EN**: Opens namespace `scudo`.
- **CN**: 打开命名空间 `scudo`。

### Line 27
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 28
````cpp
Checksum HashAlgorithm = {Checksum::BSD};
````
- **EN**: Assigns or initializes state with `Checksum HashAlgorithm = {Checksum::BSD};`.
- **CN**: 使用 `Checksum HashAlgorithm = {Checksum::BSD};` 进行赋值或初始化。

### Line 29
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 30
````cpp
#if defined(__x86_64__) || defined(__i386__)
````
- **EN**: Starts a preprocessor condition: `#if defined(__x86_64__) || defined(__i386__)`.
- **CN**: 开始一个预处理条件：`#if defined(__x86_64__) || defined(__i386__)`。

### Line 31
````cpp
// i386 and x86_64 specific code to detect CRC32 hardware support via CPUID.
````
- **EN**: Comment documenting `i386 and x86_64 specific code to detect CRC32 hardware support via CPUID.`.
- **CN**: 注释说明了 `i386 and x86_64 specific code to detect CRC32 hardware support via CPUID.`。

### Line 32
````cpp
// CRC32 requires the SSE 4.2 instruction set.
````
- **EN**: Comment documenting `CRC32 requires the SSE 4.2 instruction set.`.
- **CN**: 注释说明了 `CRC32 requires the SSE 4.2 instruction set.`。

### Line 33
````cpp
#ifndef bit_SSE4_2
````
- **EN**: Starts a preprocessor condition: `#ifndef bit_SSE4_2`.
- **CN**: 开始一个预处理条件：`#ifndef bit_SSE4_2`。

### Line 34
````cpp
#define bit_SSE4_2 bit_SSE42 // clang and gcc have different defines.
````
- **EN**: Defines a macro or compile-time constant: `#define bit_SSE4_2 bit_SSE42 // clang and gcc have different defines.`.
- **CN**: 定义宏或编译期常量：`#define bit_SSE4_2 bit_SSE42 // clang and gcc have different defines.`。

### Line 35
````cpp
#endif
````
- **EN**: Ends the current preprocessor condition.
- **CN**: 结束当前的预处理条件。

### Line 36
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 37
````cpp
#ifndef signature_HYGON_ebx // They are not defined in gcc.
````
- **EN**: Starts a preprocessor condition: `#ifndef signature_HYGON_ebx // They are not defined in gcc.`.
- **CN**: 开始一个预处理条件：`#ifndef signature_HYGON_ebx // They are not defined in gcc.`。

### Line 38
````cpp
// HYGON: "HygonGenuine".
````
- **EN**: Comment documenting `HYGON: "HygonGenuine".`.
- **CN**: 注释说明了 `HYGON: "HygonGenuine".`。

### Line 39
````cpp
#define signature_HYGON_ebx 0x6f677948
````
- **EN**: Defines a macro or compile-time constant: `#define signature_HYGON_ebx 0x6f677948`.
- **CN**: 定义宏或编译期常量：`#define signature_HYGON_ebx 0x6f677948`。

### Line 40
````cpp
#define signature_HYGON_edx 0x6e65476e
````
- **EN**: Defines a macro or compile-time constant: `#define signature_HYGON_edx 0x6e65476e`.
- **CN**: 定义宏或编译期常量：`#define signature_HYGON_edx 0x6e65476e`。

### Line 41
````cpp
#define signature_HYGON_ecx 0x656e6975
````
- **EN**: Defines a macro or compile-time constant: `#define signature_HYGON_ecx 0x656e6975`.
- **CN**: 定义宏或编译期常量：`#define signature_HYGON_ecx 0x656e6975`。

### Line 42
````cpp
#endif
````
- **EN**: Ends the current preprocessor condition.
- **CN**: 结束当前的预处理条件。

### Line 43
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 44
````cpp
bool hasHardwareCRC32() {
````
- **EN**: Begins a function or method definition: `bool hasHardwareCRC32() {`.
- **CN**: 开始一个函数或方法定义：`bool hasHardwareCRC32() {`。

### Line 45
````cpp
  u32 Eax, Ebx = 0, Ecx = 0, Edx = 0;
````
- **EN**: Assigns or initializes state with `u32 Eax, Ebx = 0, Ecx = 0, Edx = 0;`.
- **CN**: 使用 `u32 Eax, Ebx = 0, Ecx = 0, Edx = 0;` 进行赋值或初始化。

### Line 46
````cpp
  __get_cpuid(0, &Eax, &Ebx, &Ecx, &Edx);
````
- **EN**: Invokes a function-like statement: `__get_cpuid(0, &Eax, &Ebx, &Ecx, &Edx);`.
- **CN**: 调用一个类似函数的语句：`__get_cpuid(0, &Eax, &Ebx, &Ecx, &Edx);`。

### Line 47
````cpp
  const bool IsIntel = (Ebx == signature_INTEL_ebx) &&
````
- **EN**: Carries part of the local implementation logic: `const bool IsIntel = (Ebx == signature_INTEL_ebx) &&`.
- **CN**: 承载局部实现逻辑：`const bool IsIntel = (Ebx == signature_INTEL_ebx) &&`。

### Line 48
````cpp
                       (Edx == signature_INTEL_edx) &&
````
- **EN**: Carries part of the local implementation logic: `(Edx == signature_INTEL_edx) &&`.
- **CN**: 承载局部实现逻辑：`(Edx == signature_INTEL_edx) &&`。

### Line 49
````cpp
                       (Ecx == signature_INTEL_ecx);
````
- **EN**: Invokes a function-like statement: `(Ecx == signature_INTEL_ecx);`.
- **CN**: 调用一个类似函数的语句：`(Ecx == signature_INTEL_ecx);`。

### Line 50
````cpp
  const bool IsAMD = (Ebx == signature_AMD_ebx) && (Edx == signature_AMD_edx) &&
````
- **EN**: Carries part of the local implementation logic: `const bool IsAMD = (Ebx == signature_AMD_ebx) && (Edx == signature_AMD_edx) &&`.
- **CN**: 承载局部实现逻辑：`const bool IsAMD = (Ebx == signature_AMD_ebx) && (Edx == signature_AMD_edx) &&`。

### Line 51
````cpp
                     (Ecx == signature_AMD_ecx);
````
- **EN**: Invokes a function-like statement: `(Ecx == signature_AMD_ecx);`.
- **CN**: 调用一个类似函数的语句：`(Ecx == signature_AMD_ecx);`。

### Line 52
````cpp
  const bool IsHygon = (Ebx == signature_HYGON_ebx) &&
````
- **EN**: Carries part of the local implementation logic: `const bool IsHygon = (Ebx == signature_HYGON_ebx) &&`.
- **CN**: 承载局部实现逻辑：`const bool IsHygon = (Ebx == signature_HYGON_ebx) &&`。

### Line 53
````cpp
                       (Edx == signature_HYGON_edx) &&
````
- **EN**: Carries part of the local implementation logic: `(Edx == signature_HYGON_edx) &&`.
- **CN**: 承载局部实现逻辑：`(Edx == signature_HYGON_edx) &&`。

### Line 54
````cpp
                       (Ecx == signature_HYGON_ecx);
````
- **EN**: Invokes a function-like statement: `(Ecx == signature_HYGON_ecx);`.
- **CN**: 调用一个类似函数的语句：`(Ecx == signature_HYGON_ecx);`。

### Line 55
````cpp
  if (!IsIntel && !IsAMD && !IsHygon)
````
- **EN**: Evaluates the conditional branch `if (!IsIntel && !IsAMD && !IsHygon)`.
- **CN**: 计算条件分支 `if (!IsIntel && !IsAMD && !IsHygon)`。

### Line 56
````cpp
    return false;
````
- **EN**: Returns from the current function with `false;`.
- **CN**: 使用 `false;` 从当前函数返回。

### Line 57
````cpp
  __get_cpuid(1, &Eax, &Ebx, &Ecx, &Edx);
````
- **EN**: Invokes a function-like statement: `__get_cpuid(1, &Eax, &Ebx, &Ecx, &Edx);`.
- **CN**: 调用一个类似函数的语句：`__get_cpuid(1, &Eax, &Ebx, &Ecx, &Edx);`。

### Line 58
````cpp
  return !!(Ecx & bit_SSE4_2);
````
- **EN**: Returns from the current function with `!!(Ecx & bit_SSE4_2);`.
- **CN**: 使用 `!!(Ecx & bit_SSE4_2);` 从当前函数返回。

### Line 59
````cpp
}
````
- **EN**: Closes the current scope block.
- **CN**: 结束当前作用域块。

### Line 60
````cpp
#elif defined(__arm__) || defined(__aarch64__)
````
- **EN**: Checks an alternate preprocessor branch: `#elif defined(__arm__) || defined(__aarch64__)`.
- **CN**: 检查预处理器的备用分支：`#elif defined(__arm__) || defined(__aarch64__)`。

### Line 61
````cpp
#ifndef AT_HWCAP
````
- **EN**: Starts a preprocessor condition: `#ifndef AT_HWCAP`.
- **CN**: 开始一个预处理条件：`#ifndef AT_HWCAP`。

### Line 62
````cpp
#define AT_HWCAP 16
````
- **EN**: Defines a macro or compile-time constant: `#define AT_HWCAP 16`.
- **CN**: 定义宏或编译期常量：`#define AT_HWCAP 16`。

### Line 63
````cpp
#endif
````
- **EN**: Ends the current preprocessor condition.
- **CN**: 结束当前的预处理条件。

### Line 64
````cpp
#ifndef HWCAP_CRC32
````
- **EN**: Starts a preprocessor condition: `#ifndef HWCAP_CRC32`.
- **CN**: 开始一个预处理条件：`#ifndef HWCAP_CRC32`。

### Line 65
````cpp
#define HWCAP_CRC32 (1U << 7) // HWCAP_CRC32 is missing on older platforms.
````
- **EN**: Defines a macro or compile-time constant: `#define HWCAP_CRC32 (1U << 7) // HWCAP_CRC32 is missing on older platforms.`.
- **CN**: 定义宏或编译期常量：`#define HWCAP_CRC32 (1U << 7) // HWCAP_CRC32 is missing on older platforms.`。

### Line 66
````cpp
#endif
````
- **EN**: Ends the current preprocessor condition.
- **CN**: 结束当前的预处理条件。

### Line 67
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 68
````cpp
bool hasHardwareCRC32() {
````
- **EN**: Begins a function or method definition: `bool hasHardwareCRC32() {`.
- **CN**: 开始一个函数或方法定义：`bool hasHardwareCRC32() {`。

### Line 69
````cpp
#if SCUDO_FUCHSIA
````
- **EN**: Starts a preprocessor condition: `#if SCUDO_FUCHSIA`.
- **CN**: 开始一个预处理条件：`#if SCUDO_FUCHSIA`。

### Line 70
````cpp
  u32 HWCap;
````
- **EN**: Executes or declares `u32 HWCap;` within the current scope.
- **CN**: 在当前作用域中执行或声明 `u32 HWCap;`。

### Line 71
````cpp
  const zx_status_t Status =
````
- **EN**: Carries part of the local implementation logic: `const zx_status_t Status =`.
- **CN**: 承载局部实现逻辑：`const zx_status_t Status =`。

### Line 72
````cpp
      zx_system_get_features(ZX_FEATURE_KIND_CPU, &HWCap);
````
- **EN**: Invokes a function-like statement: `zx_system_get_features(ZX_FEATURE_KIND_CPU, &HWCap);`.
- **CN**: 调用一个类似函数的语句：`zx_system_get_features(ZX_FEATURE_KIND_CPU, &HWCap);`。

### Line 73
````cpp
  if (Status != ZX_OK)
````
- **EN**: Evaluates the conditional branch `if (Status != ZX_OK)`.
- **CN**: 计算条件分支 `if (Status != ZX_OK)`。

### Line 74
````cpp
    return false;
````
- **EN**: Returns from the current function with `false;`.
- **CN**: 使用 `false;` 从当前函数返回。

### Line 75
````cpp
  return !!(HWCap & ZX_ARM64_FEATURE_ISA_CRC32);
````
- **EN**: Returns from the current function with `!!(HWCap & ZX_ARM64_FEATURE_ISA_CRC32);`.
- **CN**: 使用 `!!(HWCap & ZX_ARM64_FEATURE_ISA_CRC32);` 从当前函数返回。

### Line 76
````cpp
#else
````
- **EN**: Starts the fallback branch of the current preprocessor condition.
- **CN**: 开始当前预处理条件的后备分支。

### Line 77
````cpp
  return !!(getauxval(AT_HWCAP) & HWCAP_CRC32);
````
- **EN**: Returns from the current function with `!!(getauxval(AT_HWCAP) & HWCAP_CRC32);`.
- **CN**: 使用 `!!(getauxval(AT_HWCAP) & HWCAP_CRC32);` 从当前函数返回。

### Line 78
````cpp
#endif // SCUDO_FUCHSIA
````
- **EN**: Ends the current preprocessor condition.
- **CN**: 结束当前的预处理条件。

### Line 79
````cpp
}
````
- **EN**: Closes the current scope block.
- **CN**: 结束当前作用域块。

### Line 80
````cpp
#elif defined(__loongarch__)
````
- **EN**: Checks an alternate preprocessor branch: `#elif defined(__loongarch__)`.
- **CN**: 检查预处理器的备用分支：`#elif defined(__loongarch__)`。

### Line 81
````cpp
// The definition is only pulled in by <sys/auxv.h> since glibc 2.38, so
````
- **EN**: Comment documenting `The definition is only pulled in by <sys/auxv.h> since glibc 2.38, so`.
- **CN**: 注释说明了 `The definition is only pulled in by <sys/auxv.h> since glibc 2.38, so`。

### Line 82
````cpp
// supply it if missing.
````
- **EN**: Comment documenting `supply it if missing.`.
- **CN**: 注释说明了 `supply it if missing.`。

### Line 83
````cpp
#ifndef HWCAP_LOONGARCH_CRC32
````
- **EN**: Starts a preprocessor condition: `#ifndef HWCAP_LOONGARCH_CRC32`.
- **CN**: 开始一个预处理条件：`#ifndef HWCAP_LOONGARCH_CRC32`。

### Line 84
````cpp
#define HWCAP_LOONGARCH_CRC32 (1 << 6)
````
- **EN**: Defines a macro or compile-time constant: `#define HWCAP_LOONGARCH_CRC32 (1 << 6)`.
- **CN**: 定义宏或编译期常量：`#define HWCAP_LOONGARCH_CRC32 (1 << 6)`。

### Line 85
````cpp
#endif
````
- **EN**: Ends the current preprocessor condition.
- **CN**: 结束当前的预处理条件。

### Line 86
````cpp
// Query HWCAP for platform capability, according to *Software Development and
````
- **EN**: Comment documenting `Query HWCAP for platform capability, according to *Software Development and`.
- **CN**: 注释说明了 `Query HWCAP for platform capability, according to *Software Development and`。

### Line 87
````cpp
// Build Convention for LoongArch Architectures* v0.1, Section 9.1.
````
- **EN**: Comment documenting `Build Convention for LoongArch Architectures* v0.1, Section 9.1.`.
- **CN**: 注释说明了 `Build Convention for LoongArch Architectures* v0.1, Section 9.1.`。

### Line 88
````cpp
//
````
- **EN**: Comment formatting line in a banner or block comment.
- **CN**: 注释格式行，用于组织文件头或注释块。

### Line 89
````cpp
// Link:
````
- **EN**: Comment documenting `Link:`.
- **CN**: 注释说明了 `Link:`。

### Line 90
````cpp
// https://github.com/loongson/la-softdev-convention/blob/v0.1/la-softdev-convention.adoc#kernel-development
````
- **EN**: Comment documenting `https://github.com/loongson/la-softdev-convention/blob/v0.1/la-softdev-convention.adoc#kernel-development`.
- **CN**: 注释说明了 `https://github.com/loongson/la-softdev-convention/blob/v0.1/la-softdev-convention.adoc#kernel-development`。

### Line 91
````cpp
bool hasHardwareCRC32() {
````
- **EN**: Begins a function or method definition: `bool hasHardwareCRC32() {`.
- **CN**: 开始一个函数或方法定义：`bool hasHardwareCRC32() {`。

### Line 92
````cpp
  return !!(getauxval(AT_HWCAP) & HWCAP_LOONGARCH_CRC32);
````
- **EN**: Returns from the current function with `!!(getauxval(AT_HWCAP) & HWCAP_LOONGARCH_CRC32);`.
- **CN**: 使用 `!!(getauxval(AT_HWCAP) & HWCAP_LOONGARCH_CRC32);` 从当前函数返回。

### Line 93
````cpp
}
````
- **EN**: Closes the current scope block.
- **CN**: 结束当前作用域块。

### Line 94
````cpp
#else
````
- **EN**: Starts the fallback branch of the current preprocessor condition.
- **CN**: 开始当前预处理条件的后备分支。

### Line 95
````cpp
// No hardware CRC32 implemented in Scudo for other architectures.
````
- **EN**: Comment documenting `No hardware CRC32 implemented in Scudo for other architectures.`.
- **CN**: 注释说明了 `No hardware CRC32 implemented in Scudo for other architectures.`。

### Line 96
````cpp
bool hasHardwareCRC32() { return false; }
````
- **EN**: Carries part of the local implementation logic: `bool hasHardwareCRC32() { return false; }`.
- **CN**: 承载局部实现逻辑：`bool hasHardwareCRC32() { return false; }`。

### Line 97
````cpp
#endif // defined(__x86_64__) || defined(__i386__)
````
- **EN**: Ends the current preprocessor condition.
- **CN**: 结束当前的预处理条件。

### Line 98
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 99
````cpp
} // namespace scudo
````
- **EN**: Closes namespace `scudo`.
- **CN**: 关闭命名空间 `scudo`。

## Key Concepts / 关键概念
- Preprocessor control / 预处理控制
- Namespaces / 命名空间
- Function logic / 函数逻辑
- Threading and synchronization / 线程与同步

## Dependencies / 依赖关系
- **Local headers / 本地头文件**: `checksum.h`, `atomic_helpers.h`, `chunk.h`
- **System headers / 系统头文件**: `cpuid.h`, `zircon/features.h`, `zircon/syscalls.h`, `sys/auxv.h`, `sys/auxv.h`
- **Compile-time conditions / 编译期条件**:
  - `#if defined(__x86_64__) || defined(__i386__)`
  - `#if SCUDO_FUCHSIA`
  - `#if defined(__x86_64__) || defined(__i386__)`
  - `#ifndef bit_SSE4_2`
  - `#ifndef signature_HYGON_ebx // They are not defined in gcc.`
  - `#ifndef AT_HWCAP`
  - `#ifndef HWCAP_CRC32`
  - `#if SCUDO_FUCHSIA`
  - `#ifndef HWCAP_LOONGARCH_CRC32`
