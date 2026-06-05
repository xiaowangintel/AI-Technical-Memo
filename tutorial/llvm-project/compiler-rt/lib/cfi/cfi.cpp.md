# cfi.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `compiler-rt/lib/cfi/cfi.cpp`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose / 目的**:
  - **EN**: This file implements the runtime support for the cross-DSO CFI.
  - **CN**: 实现 与 `cfi` 相关的控制流完整性运行时支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16 / 第 1-16 行
```cpp
 1 | //===-------- cfi.cpp -----------------------------------------------------===//
 2 | //
 3 | // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4 | // See https://llvm.org/LICENSE.txt for license information.
 5 | // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6 | //
 7 | //===----------------------------------------------------------------------===//
 8 | //
 9 | // This file implements the runtime support for the cross-DSO CFI.
10 | //
11 | //===----------------------------------------------------------------------===//
12 | 
13 | #include <assert.h>
14 | #include <elf.h>
15 | 
16 | #include "sanitizer_common/sanitizer_common.h"
```
- **Line 1 / 第 1 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 2 / 第 2 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 3 / 第 3 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 4 / 第 4 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 5 / 第 5 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 6 / 第 6 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 7 / 第 7 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 8 / 第 8 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 9 / 第 9 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 10 / 第 10 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 11 / 第 11 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 12 / 第 12 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 13 / 第 13 行**: EN: Includes `assert.h` so this file can use its declarations. CN: 包含 `assert.h`，以便当前文件使用其中的声明。
- **Line 14 / 第 14 行**: EN: Includes `elf.h` so this file can use its declarations. CN: 包含 `elf.h`，以便当前文件使用其中的声明。
- **Line 15 / 第 15 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 16 / 第 16 行**: EN: Includes `sanitizer_common/sanitizer_common.h` so this file can use its declarations. CN: 包含 `sanitizer_common/sanitizer_common.h`，以便当前文件使用其中的声明。

### Lines 17-32 / 第 17-32 行
```cpp
17 | #if SANITIZER_FREEBSD
18 | #include <sys/link_elf.h>
19 | #endif
20 | #include <link.h>
21 | #include <string.h>
22 | #include <stdlib.h>
23 | #include <sys/mman.h>
24 | 
25 | #if SANITIZER_LINUX
26 | typedef ElfW(Phdr) Elf_Phdr;
27 | typedef ElfW(Ehdr) Elf_Ehdr;
28 | typedef ElfW(Addr) Elf_Addr;
29 | typedef ElfW(Sym) Elf_Sym;
30 | typedef ElfW(Dyn) Elf_Dyn;
31 | #elif SANITIZER_FREEBSD
32 | #if SANITIZER_WORDSIZE == 64
```
- **Line 17 / 第 17 行**: EN: Starts a conditional-compilation guard. CN: 开始一个条件编译保护块。
- **Line 18 / 第 18 行**: EN: Includes `sys/link_elf.h` so this file can use its declarations. CN: 包含 `sys/link_elf.h`，以便当前文件使用其中的声明。
- **Line 19 / 第 19 行**: EN: Closes the conditional-compilation block. CN: 结束条件编译块。
- **Line 20 / 第 20 行**: EN: Includes `link.h` so this file can use its declarations. CN: 包含 `link.h`，以便当前文件使用其中的声明。
- **Line 21 / 第 21 行**: EN: Includes `string.h` so this file can use its declarations. CN: 包含 `string.h`，以便当前文件使用其中的声明。
- **Line 22 / 第 22 行**: EN: Includes `stdlib.h` so this file can use its declarations. CN: 包含 `stdlib.h`，以便当前文件使用其中的声明。
- **Line 23 / 第 23 行**: EN: Includes `sys/mman.h` so this file can use its declarations. CN: 包含 `sys/mman.h`，以便当前文件使用其中的声明。
- **Line 24 / 第 24 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 25 / 第 25 行**: EN: Starts a conditional-compilation guard. CN: 开始一个条件编译保护块。
- **Line 26 / 第 26 行**: EN: Defines a typedef alias for an existing type. CN: 为已有类型定义 typedef 别名。
- **Line 27 / 第 27 行**: EN: Defines a typedef alias for an existing type. CN: 为已有类型定义 typedef 别名。
- **Line 28 / 第 28 行**: EN: Defines a typedef alias for an existing type. CN: 为已有类型定义 typedef 别名。
- **Line 29 / 第 29 行**: EN: Defines a typedef alias for an existing type. CN: 为已有类型定义 typedef 别名。
- **Line 30 / 第 30 行**: EN: Defines a typedef alias for an existing type. CN: 为已有类型定义 typedef 别名。
- **Line 31 / 第 31 行**: EN: Selects an alternate conditional-compilation branch. CN: 选择条件编译的另一分支。
- **Line 32 / 第 32 行**: EN: Starts a conditional-compilation guard. CN: 开始一个条件编译保护块。

### Lines 33-48 / 第 33-48 行
```cpp
33 | #define ElfW64_Dyn Elf_Dyn
34 | #define ElfW64_Sym Elf_Sym
35 | #else
36 | #define ElfW32_Dyn Elf_Dyn
37 | #define ElfW32_Sym Elf_Sym
38 | #endif
39 | #endif
40 | 
41 | #include "interception/interception.h"
42 | #include "sanitizer_common/sanitizer_flag_parser.h"
43 | #include "ubsan/ubsan_init.h"
44 | #include "ubsan/ubsan_flags.h"
45 | 
46 | #ifdef CFI_ENABLE_DIAG
47 | #include "ubsan/ubsan_handlers.h"
48 | #endif
```
- **Line 33 / 第 33 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 34 / 第 34 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 35 / 第 35 行**: EN: Selects an alternate conditional-compilation branch. CN: 选择条件编译的另一分支。
- **Line 36 / 第 36 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 37 / 第 37 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 38 / 第 38 行**: EN: Closes the conditional-compilation block. CN: 结束条件编译块。
- **Line 39 / 第 39 行**: EN: Closes the conditional-compilation block. CN: 结束条件编译块。
- **Line 40 / 第 40 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 41 / 第 41 行**: EN: Includes `interception/interception.h` so this file can use its declarations. CN: 包含 `interception/interception.h`，以便当前文件使用其中的声明。
- **Line 42 / 第 42 行**: EN: Includes `sanitizer_common/sanitizer_flag_parser.h` so this file can use its declarations. CN: 包含 `sanitizer_common/sanitizer_flag_parser.h`，以便当前文件使用其中的声明。
- **Line 43 / 第 43 行**: EN: Includes `ubsan/ubsan_init.h` so this file can use its declarations. CN: 包含 `ubsan/ubsan_init.h`，以便当前文件使用其中的声明。
- **Line 44 / 第 44 行**: EN: Includes `ubsan/ubsan_flags.h` so this file can use its declarations. CN: 包含 `ubsan/ubsan_flags.h`，以便当前文件使用其中的声明。
- **Line 45 / 第 45 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 46 / 第 46 行**: EN: Starts a conditional-compilation guard. CN: 开始一个条件编译保护块。
- **Line 47 / 第 47 行**: EN: Includes `ubsan/ubsan_handlers.h` so this file can use its declarations. CN: 包含 `ubsan/ubsan_handlers.h`，以便当前文件使用其中的声明。
- **Line 48 / 第 48 行**: EN: Closes the conditional-compilation block. CN: 结束条件编译块。

### Lines 49-64 / 第 49-64 行
```cpp
49 | 
50 | using namespace __sanitizer;
51 | 
52 | namespace __cfi {
53 | 
54 | #if SANITIZER_LOONGARCH64
55 | #define kCfiShadowLimitsStorageSize 16384 // 16KiB on loongarch64 per page
56 | #else
57 | #define kCfiShadowLimitsStorageSize 4096 // 1 page
58 | #endif
59 | // Lets hope that the data segment is mapped with 4K pages.
60 | // The pointer to the cfi shadow region is stored at the start of this page.
61 | // The rest of the page is unused and re-mapped read-only.
62 | static union {
63 |   char space[kCfiShadowLimitsStorageSize];
64 |   struct {
```
- **Line 49 / 第 49 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 50 / 第 50 行**: EN: Adds a using declaration or alias for a type, symbol, or namespace. CN: 为类型、符号或命名空间添加 using 声明或别名。
- **Line 51 / 第 51 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 52 / 第 52 行**: EN: Opens namespace `__cfi` to scope related declarations. CN: 打开命名空间 `__cfi`，为相关声明建立作用域。
- **Line 53 / 第 53 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 54 / 第 54 行**: EN: Starts a conditional-compilation guard. CN: 开始一个条件编译保护块。
- **Line 55 / 第 55 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 56 / 第 56 行**: EN: Selects an alternate conditional-compilation branch. CN: 选择条件编译的另一分支。
- **Line 57 / 第 57 行**: EN: Defines a macro or header-guard symbol. CN: 定义宏或头文件保护符。
- **Line 58 / 第 58 行**: EN: Closes the conditional-compilation block. CN: 结束条件编译块。
- **Line 59 / 第 59 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 60 / 第 60 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 61 / 第 61 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 62 / 第 62 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 63 / 第 63 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 64 / 第 64 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 65-80 / 第 65-80 行
```cpp
65 |     uptr start;
66 |     uptr size;
67 |   } limits;
68 | } cfi_shadow_limits_storage
69 |     __attribute__((aligned(kCfiShadowLimitsStorageSize)));
70 | static constexpr uptr kShadowGranularity = 12;
71 | static constexpr uptr kShadowAlign = 1UL << kShadowGranularity; // 4096
72 | 
73 | static constexpr uint16_t kInvalidShadow = 0;
74 | static constexpr uint16_t kUncheckedShadow = 0xFFFFU;
75 | 
76 | // Get the start address of the CFI shadow region.
77 | uptr GetShadow() {
78 |   return cfi_shadow_limits_storage.limits.start;
79 | }
80 | 
```
- **Line 65 / 第 65 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 66 / 第 66 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 67 / 第 67 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 68 / 第 68 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 69 / 第 69 行**: EN: Declares function or method `__attribute__`. CN: 声明函数或方法 `__attribute__`。
- **Line 70 / 第 70 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 71 / 第 71 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 72 / 第 72 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 73 / 第 73 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 74 / 第 74 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 75 / 第 75 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 76 / 第 76 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 77 / 第 77 行**: EN: Starts the definition of function or method `GetShadow`. CN: 开始定义函数或方法 `GetShadow`。
- **Line 78 / 第 78 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 79 / 第 79 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 80 / 第 80 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。

### Lines 81-96 / 第 81-96 行
```cpp
81 | uptr GetShadowSize() {
82 |   return cfi_shadow_limits_storage.limits.size;
83 | }
84 | 
85 | // This will only work while the shadow is not allocated.
86 | void SetShadowSize(uptr size) {
87 |   cfi_shadow_limits_storage.limits.size = size;
88 | }
89 | 
90 | uptr MemToShadowOffset(uptr x) {
91 |   return (x >> kShadowGranularity) << 1;
92 | }
93 | 
94 | uint16_t *MemToShadow(uptr x, uptr shadow_base) {
95 |   return (uint16_t *)(shadow_base + MemToShadowOffset(x));
96 | }
```
- **Line 81 / 第 81 行**: EN: Starts the definition of function or method `GetShadowSize`. CN: 开始定义函数或方法 `GetShadowSize`。
- **Line 82 / 第 82 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 83 / 第 83 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 84 / 第 84 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 85 / 第 85 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 86 / 第 86 行**: EN: Starts the definition of function or method `SetShadowSize`. CN: 开始定义函数或方法 `SetShadowSize`。
- **Line 87 / 第 87 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 88 / 第 88 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 89 / 第 89 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 90 / 第 90 行**: EN: Starts the definition of function or method `MemToShadowOffset`. CN: 开始定义函数或方法 `MemToShadowOffset`。
- **Line 91 / 第 91 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 92 / 第 92 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 93 / 第 93 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 94 / 第 94 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 95 / 第 95 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 96 / 第 96 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。

### Lines 97-112 / 第 97-112 行
```cpp
 97 | 
 98 | typedef int (*CFICheckFn)(u64, void *, void *);
 99 | 
100 | // This class reads and decodes the shadow contents.
101 | class ShadowValue {
102 |   uptr addr;
103 |   uint16_t v;
104 |   explicit ShadowValue(uptr addr, uint16_t v) : addr(addr), v(v) {}
105 | 
106 | public:
107 |   bool is_invalid() const { return v == kInvalidShadow; }
108 | 
109 |   bool is_unchecked() const { return v == kUncheckedShadow; }
110 | 
111 |   CFICheckFn get_cfi_check() const {
112 |     assert(!is_invalid() && !is_unchecked());
```
- **Line 97 / 第 97 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 98 / 第 98 行**: EN: Defines a typedef alias for an existing type. CN: 为已有类型定义 typedef 别名。
- **Line 99 / 第 99 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 100 / 第 100 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 101 / 第 101 行**: EN: Begins the declaration of class `ShadowValue`. CN: 开始声明 class `ShadowValue`。
- **Line 102 / 第 102 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 103 / 第 103 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 104 / 第 104 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 105 / 第 105 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 106 / 第 106 行**: EN: Sets access control for the following members. CN: 设置后续成员的访问控制级别。
- **Line 107 / 第 107 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 108 / 第 108 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 109 / 第 109 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 110 / 第 110 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 111 / 第 111 行**: EN: Starts the definition of function or method `get_cfi_check`. CN: 开始定义函数或方法 `get_cfi_check`。
- **Line 112 / 第 112 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。

### Lines 113-128 / 第 113-128 行
```cpp
113 |     uptr aligned_addr = addr & ~(kShadowAlign - 1);
114 |     uptr p = aligned_addr - (((uptr)v - 1) << kShadowGranularity);
115 |     return reinterpret_cast<CFICheckFn>(p);
116 |   }
117 | 
118 |   // Load a shadow value for the given application memory address.
119 |   static const ShadowValue load(uptr addr) {
120 |     uptr shadow_base = GetShadow();
121 |     uptr shadow_offset = MemToShadowOffset(addr);
122 |     if (shadow_offset > GetShadowSize())
123 |       return ShadowValue(addr, kInvalidShadow);
124 |     else
125 |       return ShadowValue(
126 |           addr, *reinterpret_cast<uint16_t *>(shadow_base + shadow_offset));
127 |   }
128 | };
```
- **Line 113 / 第 113 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 114 / 第 114 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 115 / 第 115 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 116 / 第 116 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 117 / 第 117 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 118 / 第 118 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 119 / 第 119 行**: EN: Starts the definition of function or method `load`. CN: 开始定义函数或方法 `load`。
- **Line 120 / 第 120 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 121 / 第 121 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 122 / 第 122 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 123 / 第 123 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 124 / 第 124 行**: EN: Handles the fallback branch when earlier conditions fail. CN: 当前面条件不满足时处理回退分支。
- **Line 125 / 第 125 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 126 / 第 126 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 127 / 第 127 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 128 / 第 128 行**: EN: Terminates a type declaration or scoped block. CN: 结束一个类型声明或作用域块。

### Lines 129-144 / 第 129-144 行
```cpp
129 | 
130 | class ShadowBuilder {
131 |   uptr shadow_;
132 | 
133 | public:
134 |   // Allocate a new empty shadow (for the entire address space) on the side.
135 |   void Start();
136 |   // Mark the given address range as unchecked.
137 |   // This is used for uninstrumented libraries like libc.
138 |   // Any CFI check with a target in that range will pass.
139 |   void AddUnchecked(uptr begin, uptr end);
140 |   // Mark the given address range as belonging to a library with the given
141 |   // cfi_check function.
142 |   void Add(uptr begin, uptr end, uptr cfi_check);
143 |   // Finish shadow construction. Atomically switch the current active shadow
144 |   // region with the newly constructed one and deallocate the former.
```
- **Line 129 / 第 129 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 130 / 第 130 行**: EN: Begins the declaration of class `ShadowBuilder`. CN: 开始声明 class `ShadowBuilder`。
- **Line 131 / 第 131 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 132 / 第 132 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 133 / 第 133 行**: EN: Sets access control for the following members. CN: 设置后续成员的访问控制级别。
- **Line 134 / 第 134 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 135 / 第 135 行**: EN: Declares function or method `Start`. CN: 声明函数或方法 `Start`。
- **Line 136 / 第 136 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 137 / 第 137 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 138 / 第 138 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 139 / 第 139 行**: EN: Declares function or method `AddUnchecked`. CN: 声明函数或方法 `AddUnchecked`。
- **Line 140 / 第 140 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 141 / 第 141 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 142 / 第 142 行**: EN: Declares function or method `Add`. CN: 声明函数或方法 `Add`。
- **Line 143 / 第 143 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 144 / 第 144 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。

### Lines 145-160 / 第 145-160 行
```cpp
145 |   void Install();
146 | };
147 | 
148 | void ShadowBuilder::Start() {
149 |   shadow_ = (uptr)MmapNoReserveOrDie(GetShadowSize(), "CFI shadow");
150 |   VReport(1, "CFI: shadow at %zx .. %zx\n", shadow_, shadow_ + GetShadowSize());
151 | }
152 | 
153 | void ShadowBuilder::AddUnchecked(uptr begin, uptr end) {
154 |   uint16_t *shadow_begin = MemToShadow(begin, shadow_);
155 |   uint16_t *shadow_end = MemToShadow(end - 1, shadow_) + 1;
156 |   // memset takes a byte, so our unchecked shadow value requires both bytes to
157 |   // be the same. Make sure we're ok during compilation.
158 |   static_assert((kUncheckedShadow & 0xff) == ((kUncheckedShadow >> 8) & 0xff),
159 |                 "Both bytes of the 16-bit value must be the same!");
160 |   memset(shadow_begin, kUncheckedShadow & 0xff,
```
- **Line 145 / 第 145 行**: EN: Declares function or method `Install`. CN: 声明函数或方法 `Install`。
- **Line 146 / 第 146 行**: EN: Terminates a type declaration or scoped block. CN: 结束一个类型声明或作用域块。
- **Line 147 / 第 147 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 148 / 第 148 行**: EN: Starts the definition of function or method `ShadowBuilder::Start`. CN: 开始定义函数或方法 `ShadowBuilder::Start`。
- **Line 149 / 第 149 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 150 / 第 150 行**: EN: Declares function or method `VReport`. CN: 声明函数或方法 `VReport`。
- **Line 151 / 第 151 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 152 / 第 152 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 153 / 第 153 行**: EN: Starts the definition of function or method `ShadowBuilder::AddUnchecked`. CN: 开始定义函数或方法 `ShadowBuilder::AddUnchecked`。
- **Line 154 / 第 154 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 155 / 第 155 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 156 / 第 156 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 157 / 第 157 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 158 / 第 158 行**: EN: Checks a compile-time invariant before the file can build. CN: 在文件成功构建前检查一个编译期不变式。
- **Line 159 / 第 159 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 160 / 第 160 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。

### Lines 161-176 / 第 161-176 行
```cpp
161 |          (shadow_end - shadow_begin) * sizeof(*shadow_begin));
162 | }
163 | 
164 | void ShadowBuilder::Add(uptr begin, uptr end, uptr cfi_check) {
165 |   assert((cfi_check & (kShadowAlign - 1)) == 0);
166 | 
167 |   // Don't fill anything below cfi_check. We can not represent those addresses
168 |   // in the shadow, and must make sure at codegen to place all valid call
169 |   // targets above cfi_check.
170 |   begin = Max(begin, cfi_check);
171 |   uint16_t *s = MemToShadow(begin, shadow_);
172 |   uint16_t *s_end = MemToShadow(end - 1, shadow_) + 1;
173 |   uint16_t sv = ((begin - cfi_check) >> kShadowGranularity) + 1;
174 |   for (; s < s_end; s++, sv++)
175 |     *s = sv;
176 | }
```
- **Line 161 / 第 161 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 162 / 第 162 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 163 / 第 163 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 164 / 第 164 行**: EN: Starts the definition of function or method `ShadowBuilder::Add`. CN: 开始定义函数或方法 `ShadowBuilder::Add`。
- **Line 165 / 第 165 行**: EN: Declares function or method `assert`. CN: 声明函数或方法 `assert`。
- **Line 166 / 第 166 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 167 / 第 167 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 168 / 第 168 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 169 / 第 169 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 170 / 第 170 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 171 / 第 171 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 172 / 第 172 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 173 / 第 173 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 174 / 第 174 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 175 / 第 175 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 176 / 第 176 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。

### Lines 177-192 / 第 177-192 行
```cpp
177 | 
178 | #if SANITIZER_LINUX || SANITIZER_FREEBSD || SANITIZER_NETBSD
179 | void ShadowBuilder::Install() {
180 |   MprotectReadOnly(shadow_, GetShadowSize());
181 |   uptr main_shadow = GetShadow();
182 |   if (main_shadow) {
183 |     // Update.
184 | #if SANITIZER_LINUX
185 |     void *res = mremap((void *)shadow_, GetShadowSize(), GetShadowSize(),
186 |                        MREMAP_MAYMOVE | MREMAP_FIXED, (void *)main_shadow);
187 |     CHECK(res != MAP_FAILED);
188 | #elif SANITIZER_NETBSD
189 |     void *res = mremap((void *)shadow_, GetShadowSize(), (void *)main_shadow,
190 |                        GetShadowSize(), MAP_FIXED);
191 |     CHECK(res != MAP_FAILED);
192 | #else
```
- **Line 177 / 第 177 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 178 / 第 178 行**: EN: Starts a conditional-compilation guard. CN: 开始一个条件编译保护块。
- **Line 179 / 第 179 行**: EN: Starts the definition of function or method `ShadowBuilder::Install`. CN: 开始定义函数或方法 `ShadowBuilder::Install`。
- **Line 180 / 第 180 行**: EN: Declares function or method `MprotectReadOnly`. CN: 声明函数或方法 `MprotectReadOnly`。
- **Line 181 / 第 181 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 182 / 第 182 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 183 / 第 183 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 184 / 第 184 行**: EN: Starts a conditional-compilation guard. CN: 开始一个条件编译保护块。
- **Line 185 / 第 185 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 186 / 第 186 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 187 / 第 187 行**: EN: Invokes a macro that expands portability, ABI, or registration boilerplate. CN: 调用一个宏，以展开可移植性、ABI 或注册相关样板代码。
- **Line 188 / 第 188 行**: EN: Selects an alternate conditional-compilation branch. CN: 选择条件编译的另一分支。
- **Line 189 / 第 189 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 190 / 第 190 行**: EN: Declares function or method `GetShadowSize`. CN: 声明函数或方法 `GetShadowSize`。
- **Line 191 / 第 191 行**: EN: Invokes a macro that expands portability, ABI, or registration boilerplate. CN: 调用一个宏，以展开可移植性、ABI 或注册相关样板代码。
- **Line 192 / 第 192 行**: EN: Selects an alternate conditional-compilation branch. CN: 选择条件编译的另一分支。

### Lines 193-208 / 第 193-208 行
```cpp
193 |     void *res = MmapFixedOrDie(shadow_, GetShadowSize(), "cfi shadow");
194 |     CHECK(res != MAP_FAILED);
195 |     ::memcpy(&shadow_, &main_shadow, GetShadowSize());
196 | #endif
197 |   } else {
198 |     // Initial setup.
199 |     CHECK_EQ(kCfiShadowLimitsStorageSize, GetPageSizeCached());
200 |     CHECK_EQ(0, GetShadow());
201 |     cfi_shadow_limits_storage.limits.start = shadow_;
202 |     MprotectReadOnly((uptr)&cfi_shadow_limits_storage,
203 |                      sizeof(cfi_shadow_limits_storage));
204 |     CHECK_EQ(shadow_, GetShadow());
205 |   }
206 | }
207 | #else
208 | #error not implemented
```
- **Line 193 / 第 193 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 194 / 第 194 行**: EN: Invokes a macro that expands portability, ABI, or registration boilerplate. CN: 调用一个宏，以展开可移植性、ABI 或注册相关样板代码。
- **Line 195 / 第 195 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 196 / 第 196 行**: EN: Closes the conditional-compilation block. CN: 结束条件编译块。
- **Line 197 / 第 197 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 198 / 第 198 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 199 / 第 199 行**: EN: Invokes a macro that expands portability, ABI, or registration boilerplate. CN: 调用一个宏，以展开可移植性、ABI 或注册相关样板代码。
- **Line 200 / 第 200 行**: EN: Invokes a macro that expands portability, ABI, or registration boilerplate. CN: 调用一个宏，以展开可移植性、ABI 或注册相关样板代码。
- **Line 201 / 第 201 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 202 / 第 202 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 203 / 第 203 行**: EN: Declares function or method `sizeof`. CN: 声明函数或方法 `sizeof`。
- **Line 204 / 第 204 行**: EN: Invokes a macro that expands portability, ABI, or registration boilerplate. CN: 调用一个宏，以展开可移植性、ABI 或注册相关样板代码。
- **Line 205 / 第 205 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 206 / 第 206 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 207 / 第 207 行**: EN: Selects an alternate conditional-compilation branch. CN: 选择条件编译的另一分支。
- **Line 208 / 第 208 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 209-224 / 第 209-224 行
```cpp
209 | #endif
210 | 
211 | // This is a workaround for a glibc bug:
212 | // https://sourceware.org/bugzilla/show_bug.cgi?id=15199
213 | // Other platforms can, hopefully, just do
214 | //    dlopen(RTLD_NOLOAD | RTLD_LAZY)
215 | //    dlsym("__cfi_check").
216 | uptr find_cfi_check_in_dso(dl_phdr_info *info) {
217 |   const Elf_Dyn *dynamic = nullptr;
218 |   for (int i = 0; i < info->dlpi_phnum; ++i) {
219 |     if (info->dlpi_phdr[i].p_type == PT_DYNAMIC) {
220 |       dynamic =
221 |           (const Elf_Dyn *)(info->dlpi_addr + info->dlpi_phdr[i].p_vaddr);
222 |       break;
223 |     }
224 |   }
```
- **Line 209 / 第 209 行**: EN: Closes the conditional-compilation block. CN: 结束条件编译块。
- **Line 210 / 第 210 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 211 / 第 211 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 212 / 第 212 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 213 / 第 213 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 214 / 第 214 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 215 / 第 215 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 216 / 第 216 行**: EN: Starts the definition of function or method `find_cfi_check_in_dso`. CN: 开始定义函数或方法 `find_cfi_check_in_dso`。
- **Line 217 / 第 217 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 218 / 第 218 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 219 / 第 219 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 220 / 第 220 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 221 / 第 221 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 222 / 第 222 行**: EN: Exits the nearest loop or switch block. CN: 退出最近的循环或 switch 代码块。
- **Line 223 / 第 223 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 224 / 第 224 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。

### Lines 225-240 / 第 225-240 行
```cpp
225 |   if (!dynamic) return 0;
226 |   uptr strtab = 0, symtab = 0, strsz = 0;
227 |   for (const Elf_Dyn *p = dynamic; p->d_tag != PT_NULL; ++p) {
228 |     if (p->d_tag == DT_SYMTAB)
229 |       symtab = p->d_un.d_ptr;
230 |     else if (p->d_tag == DT_STRTAB)
231 |       strtab = p->d_un.d_ptr;
232 |     else if (p->d_tag == DT_STRSZ)
233 |       strsz = p->d_un.d_ptr;
234 |   }
235 | 
236 |   if (symtab > strtab) {
237 |     VReport(1, "Can not handle: symtab > strtab (%zx > %zx)\n", symtab, strtab);
238 |     return 0;
239 |   }
240 | 
```
- **Line 225 / 第 225 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 226 / 第 226 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 227 / 第 227 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 228 / 第 228 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 229 / 第 229 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 230 / 第 230 行**: EN: Adds another conditional branch for alternative logic. CN: 为替代逻辑添加另一个条件分支。
- **Line 231 / 第 231 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 232 / 第 232 行**: EN: Adds another conditional branch for alternative logic. CN: 为替代逻辑添加另一个条件分支。
- **Line 233 / 第 233 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 234 / 第 234 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 235 / 第 235 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 236 / 第 236 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 237 / 第 237 行**: EN: Declares function or method `VReport`. CN: 声明函数或方法 `VReport`。
- **Line 238 / 第 238 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 239 / 第 239 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 240 / 第 240 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。

### Lines 241-256 / 第 241-256 行
```cpp
241 |   // Verify that strtab and symtab are inside of the same LOAD segment.
242 |   // This excludes VDSO, which has (very high) bogus strtab and symtab pointers.
243 |   int phdr_idx;
244 |   for (phdr_idx = 0; phdr_idx < info->dlpi_phnum; phdr_idx++) {
245 |     const Elf_Phdr *phdr = &info->dlpi_phdr[phdr_idx];
246 |     if (phdr->p_type == PT_LOAD) {
247 |       uptr beg = info->dlpi_addr + phdr->p_vaddr;
248 |       uptr end = beg + phdr->p_memsz;
249 |       if (strtab >= beg && strtab + strsz < end && symtab >= beg &&
250 |           symtab < end)
251 |         break;
252 |     }
253 |   }
254 |   if (phdr_idx == info->dlpi_phnum) {
255 |     // Nope, either different segments or just bogus pointers.
256 |     // Can not handle this.
```
- **Line 241 / 第 241 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 242 / 第 242 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 243 / 第 243 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 244 / 第 244 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 245 / 第 245 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 246 / 第 246 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 247 / 第 247 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 248 / 第 248 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 249 / 第 249 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 250 / 第 250 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 251 / 第 251 行**: EN: Exits the nearest loop or switch block. CN: 退出最近的循环或 switch 代码块。
- **Line 252 / 第 252 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 253 / 第 253 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 254 / 第 254 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 255 / 第 255 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 256 / 第 256 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。

### Lines 257-272 / 第 257-272 行
```cpp
257 |     VReport(1, "Can not handle: symtab %zx, strtab %zx\n", symtab, strtab);
258 |     return 0;
259 |   }
260 | 
261 |   for (const Elf_Sym *p = (const Elf_Sym *)symtab; (Elf_Addr)p < strtab;
262 |        ++p) {
263 |     // There is no reliable way to find the end of the symbol table. In
264 |     // lld-produces files, there are other sections between symtab and strtab.
265 |     // Stop looking when the symbol name is not inside strtab.
266 |     if (p->st_name >= strsz) break;
267 |     char *name = (char*)(strtab + p->st_name);
268 |     if (strcmp(name, "__cfi_check") == 0) {
269 |       assert(p->st_info == ELF32_ST_INFO(STB_GLOBAL, STT_FUNC) ||
270 |              p->st_info == ELF32_ST_INFO(STB_WEAK, STT_FUNC));
271 |       uptr addr = info->dlpi_addr + p->st_value;
272 |       return addr;
```
- **Line 257 / 第 257 行**: EN: Declares function or method `VReport`. CN: 声明函数或方法 `VReport`。
- **Line 258 / 第 258 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 259 / 第 259 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 260 / 第 260 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 261 / 第 261 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 262 / 第 262 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 263 / 第 263 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 264 / 第 264 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 265 / 第 265 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 266 / 第 266 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 267 / 第 267 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 268 / 第 268 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 269 / 第 269 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 270 / 第 270 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 271 / 第 271 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 272 / 第 272 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。

### Lines 273-288 / 第 273-288 行
```cpp
273 |     }
274 |   }
275 |   return 0;
276 | }
277 | 
278 | int dl_iterate_phdr_cb(dl_phdr_info *info, size_t size, void *data) {
279 |   uptr cfi_check = find_cfi_check_in_dso(info);
280 |   if (cfi_check)
281 |     VReport(1, "Module '%s' __cfi_check %zx\n", info->dlpi_name, cfi_check);
282 | 
283 |   ShadowBuilder *b = reinterpret_cast<ShadowBuilder *>(data);
284 | 
285 |   for (int i = 0; i < info->dlpi_phnum; i++) {
286 |     const Elf_Phdr *phdr = &info->dlpi_phdr[i];
287 |     if (phdr->p_type == PT_LOAD) {
288 |       // Jump tables are in the executable segment.
```
- **Line 273 / 第 273 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 274 / 第 274 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 275 / 第 275 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 276 / 第 276 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 277 / 第 277 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 278 / 第 278 行**: EN: Starts the definition of function or method `dl_iterate_phdr_cb`. CN: 开始定义函数或方法 `dl_iterate_phdr_cb`。
- **Line 279 / 第 279 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 280 / 第 280 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 281 / 第 281 行**: EN: Declares function or method `VReport`. CN: 声明函数或方法 `VReport`。
- **Line 282 / 第 282 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 283 / 第 283 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 284 / 第 284 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 285 / 第 285 行**: EN: Starts a loop over a range, iterator, or index sequence. CN: 开始一个遍历区间、迭代器或索引序列的循环。
- **Line 286 / 第 286 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 287 / 第 287 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 288 / 第 288 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。

### Lines 289-304 / 第 289-304 行
```cpp
289 |       // VTables are in the non-executable one.
290 |       // Need to fill shadow for both.
291 |       // FIXME: reject writable if vtables are in the r/o segment. Depend on
292 |       // PT_RELRO?
293 |       uptr cur_beg = info->dlpi_addr + phdr->p_vaddr;
294 |       uptr cur_end = cur_beg + phdr->p_memsz;
295 |       if (cfi_check) {
296 |         VReport(1, "   %zx .. %zx\n", cur_beg, cur_end);
297 |         b->Add(cur_beg, cur_end, cfi_check);
298 |       } else {
299 |         b->AddUnchecked(cur_beg, cur_end);
300 |       }
301 |     }
302 |   }
303 |   return 0;
304 | }
```
- **Line 289 / 第 289 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 290 / 第 290 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 291 / 第 291 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 292 / 第 292 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 293 / 第 293 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 294 / 第 294 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 295 / 第 295 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 296 / 第 296 行**: EN: Declares function or method `VReport`. CN: 声明函数或方法 `VReport`。
- **Line 297 / 第 297 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 298 / 第 298 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 299 / 第 299 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 300 / 第 300 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 301 / 第 301 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 302 / 第 302 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 303 / 第 303 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 304 / 第 304 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。

### Lines 305-320 / 第 305-320 行
```cpp
305 | 
306 | // Init or update shadow for the current set of loaded libraries.
307 | void UpdateShadow() {
308 |   ShadowBuilder b;
309 |   b.Start();
310 |   dl_iterate_phdr(dl_iterate_phdr_cb, &b);
311 |   b.Install();
312 | }
313 | 
314 | void InitShadow() {
315 |   CHECK_EQ(0, GetShadow());
316 |   CHECK_EQ(0, GetShadowSize());
317 | 
318 |   uptr vma = GetMaxUserVirtualAddress();
319 |   // Shadow is 2 -> 2**kShadowGranularity.
320 |   SetShadowSize((vma >> (kShadowGranularity - 1)) + 1);
```
- **Line 305 / 第 305 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 306 / 第 306 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 307 / 第 307 行**: EN: Starts the definition of function or method `UpdateShadow`. CN: 开始定义函数或方法 `UpdateShadow`。
- **Line 308 / 第 308 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 309 / 第 309 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 310 / 第 310 行**: EN: Declares function or method `dl_iterate_phdr`. CN: 声明函数或方法 `dl_iterate_phdr`。
- **Line 311 / 第 311 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 312 / 第 312 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 313 / 第 313 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 314 / 第 314 行**: EN: Starts the definition of function or method `InitShadow`. CN: 开始定义函数或方法 `InitShadow`。
- **Line 315 / 第 315 行**: EN: Invokes a macro that expands portability, ABI, or registration boilerplate. CN: 调用一个宏，以展开可移植性、ABI 或注册相关样板代码。
- **Line 316 / 第 316 行**: EN: Invokes a macro that expands portability, ABI, or registration boilerplate. CN: 调用一个宏，以展开可移植性、ABI 或注册相关样板代码。
- **Line 317 / 第 317 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 318 / 第 318 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 319 / 第 319 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 320 / 第 320 行**: EN: Declares function or method `SetShadowSize`. CN: 声明函数或方法 `SetShadowSize`。

### Lines 321-336 / 第 321-336 行
```cpp
321 |   VReport(1, "CFI: VMA size %zx, shadow size %zx\n", vma, GetShadowSize());
322 | 
323 |   UpdateShadow();
324 | }
325 | 
326 | THREADLOCAL int in_loader;
327 | Mutex shadow_update_lock;
328 | 
329 | void EnterLoader() SANITIZER_NO_THREAD_SAFETY_ANALYSIS {
330 |   if (in_loader == 0) {
331 |     shadow_update_lock.Lock();
332 |   }
333 |   ++in_loader;
334 | }
335 | 
336 | void ExitLoader() SANITIZER_NO_THREAD_SAFETY_ANALYSIS {
```
- **Line 321 / 第 321 行**: EN: Declares function or method `VReport`. CN: 声明函数或方法 `VReport`。
- **Line 322 / 第 322 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 323 / 第 323 行**: EN: Declares function or method `UpdateShadow`. CN: 声明函数或方法 `UpdateShadow`。
- **Line 324 / 第 324 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 325 / 第 325 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 326 / 第 326 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 327 / 第 327 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 328 / 第 328 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 329 / 第 329 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 330 / 第 330 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 331 / 第 331 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 332 / 第 332 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 333 / 第 333 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 334 / 第 334 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 335 / 第 335 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 336 / 第 336 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 337-352 / 第 337-352 行
```cpp
337 |   CHECK(in_loader > 0);
338 |   --in_loader;
339 |   UpdateShadow();
340 |   if (in_loader == 0) {
341 |     shadow_update_lock.Unlock();
342 |   }
343 | }
344 | 
345 | ALWAYS_INLINE void CfiSlowPathCommon(u64 CallSiteTypeId, void *Ptr,
346 |                                      void *DiagData) {
347 |   uptr Addr = (uptr)Ptr;
348 |   VReport(3, "__cfi_slowpath: %llx, %p\n", CallSiteTypeId, Ptr);
349 |   ShadowValue sv = ShadowValue::load(Addr);
350 |   if (sv.is_invalid()) {
351 |     VReport(1, "CFI: invalid memory region for a check target: %p\n", Ptr);
352 | #ifdef CFI_ENABLE_DIAG
```
- **Line 337 / 第 337 行**: EN: Invokes a macro that expands portability, ABI, or registration boilerplate. CN: 调用一个宏，以展开可移植性、ABI 或注册相关样板代码。
- **Line 338 / 第 338 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 339 / 第 339 行**: EN: Declares function or method `UpdateShadow`. CN: 声明函数或方法 `UpdateShadow`。
- **Line 340 / 第 340 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 341 / 第 341 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 342 / 第 342 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 343 / 第 343 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 344 / 第 344 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 345 / 第 345 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 346 / 第 346 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 347 / 第 347 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 348 / 第 348 行**: EN: Declares function or method `VReport`. CN: 声明函数或方法 `VReport`。
- **Line 349 / 第 349 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 350 / 第 350 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 351 / 第 351 行**: EN: Declares function or method `VReport`. CN: 声明函数或方法 `VReport`。
- **Line 352 / 第 352 行**: EN: Starts a conditional-compilation guard. CN: 开始一个条件编译保护块。

### Lines 353-368 / 第 353-368 行
```cpp
353 |     if (DiagData) {
354 |       __ubsan_handle_cfi_check_fail(
355 |           reinterpret_cast<__ubsan::CFICheckFailData *>(DiagData), Addr, false);
356 |       return;
357 |     }
358 | #endif
359 |     Trap();
360 |   }
361 |   if (sv.is_unchecked()) {
362 |     VReport(2, "CFI: unchecked call (shadow=FFFF): %p\n", Ptr);
363 |     return;
364 |   }
365 |   CFICheckFn cfi_check = sv.get_cfi_check();
366 |   VReport(2, "__cfi_check at %p\n", (void *)cfi_check);
367 |   cfi_check(CallSiteTypeId, Ptr, DiagData);
368 | }
```
- **Line 353 / 第 353 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 354 / 第 354 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 355 / 第 355 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 356 / 第 356 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 357 / 第 357 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 358 / 第 358 行**: EN: Closes the conditional-compilation block. CN: 结束条件编译块。
- **Line 359 / 第 359 行**: EN: Declares function or method `Trap`. CN: 声明函数或方法 `Trap`。
- **Line 360 / 第 360 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 361 / 第 361 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 362 / 第 362 行**: EN: Declares function or method `VReport`. CN: 声明函数或方法 `VReport`。
- **Line 363 / 第 363 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 364 / 第 364 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 365 / 第 365 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 366 / 第 366 行**: EN: Declares function or method `VReport`. CN: 声明函数或方法 `VReport`。
- **Line 367 / 第 367 行**: EN: Declares function or method `cfi_check`. CN: 声明函数或方法 `cfi_check`。
- **Line 368 / 第 368 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。

### Lines 369-384 / 第 369-384 行
```cpp
369 | 
370 | void InitializeFlags() {
371 |   SetCommonFlagsDefaults();
372 | #ifdef CFI_ENABLE_DIAG
373 |   __ubsan::Flags *uf = __ubsan::flags();
374 |   uf->SetDefaults();
375 | #endif
376 | 
377 |   FlagParser cfi_parser;
378 |   RegisterCommonFlags(&cfi_parser);
379 |   cfi_parser.ParseStringFromEnv("CFI_OPTIONS");
380 | 
381 | #ifdef CFI_ENABLE_DIAG
382 |   FlagParser ubsan_parser;
383 |   __ubsan::RegisterUbsanFlags(&ubsan_parser, uf);
384 |   RegisterCommonFlags(&ubsan_parser);
```
- **Line 369 / 第 369 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 370 / 第 370 行**: EN: Starts the definition of function or method `InitializeFlags`. CN: 开始定义函数或方法 `InitializeFlags`。
- **Line 371 / 第 371 行**: EN: Declares function or method `SetCommonFlagsDefaults`. CN: 声明函数或方法 `SetCommonFlagsDefaults`。
- **Line 372 / 第 372 行**: EN: Starts a conditional-compilation guard. CN: 开始一个条件编译保护块。
- **Line 373 / 第 373 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 374 / 第 374 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 375 / 第 375 行**: EN: Closes the conditional-compilation block. CN: 结束条件编译块。
- **Line 376 / 第 376 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 377 / 第 377 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 378 / 第 378 行**: EN: Declares function or method `RegisterCommonFlags`. CN: 声明函数或方法 `RegisterCommonFlags`。
- **Line 379 / 第 379 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 380 / 第 380 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 381 / 第 381 行**: EN: Starts a conditional-compilation guard. CN: 开始一个条件编译保护块。
- **Line 382 / 第 382 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 383 / 第 383 行**: EN: Declares function or method `__ubsan::RegisterUbsanFlags`. CN: 声明函数或方法 `__ubsan::RegisterUbsanFlags`。
- **Line 384 / 第 384 行**: EN: Declares function or method `RegisterCommonFlags`. CN: 声明函数或方法 `RegisterCommonFlags`。

### Lines 385-400 / 第 385-400 行
```cpp
385 | 
386 |   const char *ubsan_default_options = __ubsan_default_options();
387 |   ubsan_parser.ParseString(ubsan_default_options);
388 |   ubsan_parser.ParseStringFromEnv("UBSAN_OPTIONS");
389 | #endif
390 | 
391 |   InitializeCommonFlags();
392 | 
393 |   if (Verbosity())
394 |     ReportUnrecognizedFlags();
395 | 
396 |   if (common_flags()->help) {
397 |     cfi_parser.PrintFlagDescriptions();
398 |   }
399 | }
400 | 
```
- **Line 385 / 第 385 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 386 / 第 386 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 387 / 第 387 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 388 / 第 388 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 389 / 第 389 行**: EN: Closes the conditional-compilation block. CN: 结束条件编译块。
- **Line 390 / 第 390 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 391 / 第 391 行**: EN: Declares function or method `InitializeCommonFlags`. CN: 声明函数或方法 `InitializeCommonFlags`。
- **Line 392 / 第 392 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 393 / 第 393 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 394 / 第 394 行**: EN: Declares function or method `ReportUnrecognizedFlags`. CN: 声明函数或方法 `ReportUnrecognizedFlags`。
- **Line 395 / 第 395 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 396 / 第 396 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 397 / 第 397 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 398 / 第 398 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 399 / 第 399 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 400 / 第 400 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。

### Lines 401-416 / 第 401-416 行
```cpp
401 | } // namespace __cfi
402 | 
403 | using namespace __cfi;
404 | 
405 | extern "C" SANITIZER_INTERFACE_ATTRIBUTE void
406 | __cfi_slowpath(u64 CallSiteTypeId, void *Ptr) {
407 |   CfiSlowPathCommon(CallSiteTypeId, Ptr, nullptr);
408 | }
409 | 
410 | #ifdef CFI_ENABLE_DIAG
411 | extern "C" SANITIZER_INTERFACE_ATTRIBUTE void
412 | __cfi_slowpath_diag(u64 CallSiteTypeId, void *Ptr, void *DiagData) {
413 |   CfiSlowPathCommon(CallSiteTypeId, Ptr, DiagData);
414 | }
415 | #endif
416 | 
```
- **Line 401 / 第 401 行**: EN: Closes a named namespace scope. CN: 关闭一个具名命名空间作用域。
- **Line 402 / 第 402 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 403 / 第 403 行**: EN: Adds a using declaration or alias for a type, symbol, or namespace. CN: 为类型、符号或命名空间添加 using 声明或别名。
- **Line 404 / 第 404 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 405 / 第 405 行**: EN: Marks an ABI-level compiler-rt symbol exported to generated code or other runtimes. CN: 标记导出给生成代码或其他运行时使用的 compiler-rt ABI 级符号。
- **Line 406 / 第 406 行**: EN: Starts the definition of function or method `__cfi_slowpath`. CN: 开始定义函数或方法 `__cfi_slowpath`。
- **Line 407 / 第 407 行**: EN: Declares function or method `CfiSlowPathCommon`. CN: 声明函数或方法 `CfiSlowPathCommon`。
- **Line 408 / 第 408 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 409 / 第 409 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 410 / 第 410 行**: EN: Starts a conditional-compilation guard. CN: 开始一个条件编译保护块。
- **Line 411 / 第 411 行**: EN: Marks an ABI-level compiler-rt symbol exported to generated code or other runtimes. CN: 标记导出给生成代码或其他运行时使用的 compiler-rt ABI 级符号。
- **Line 412 / 第 412 行**: EN: Starts the definition of function or method `__cfi_slowpath_diag`. CN: 开始定义函数或方法 `__cfi_slowpath_diag`。
- **Line 413 / 第 413 行**: EN: Declares function or method `CfiSlowPathCommon`. CN: 声明函数或方法 `CfiSlowPathCommon`。
- **Line 414 / 第 414 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 415 / 第 415 行**: EN: Closes the conditional-compilation block. CN: 结束条件编译块。
- **Line 416 / 第 416 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。

### Lines 417-432 / 第 417-432 行
```cpp
417 | static void EnsureInterceptorsInitialized();
418 | 
419 | // Setup shadow for dlopen()ed libraries.
420 | // The actual shadow setup happens after dlopen() returns, which means that
421 | // a library can not be a target of any CFI checks while its constructors are
422 | // running. It's unclear how to fix this without some extra help from libc.
423 | // In glibc, mmap inside dlopen is not interceptable.
424 | // Maybe a seccomp-bpf filter?
425 | // We could insert a high-priority constructor into the library, but that would
426 | // not help with the uninstrumented libraries.
427 | INTERCEPTOR(void*, dlopen, const char *filename, int flag) {
428 |   EnsureInterceptorsInitialized();
429 |   EnterLoader();
430 |   void *handle = REAL(dlopen)(filename, flag);
431 |   ExitLoader();
432 |   return handle;
```
- **Line 417 / 第 417 行**: EN: Declares function or method `EnsureInterceptorsInitialized`. CN: 声明函数或方法 `EnsureInterceptorsInitialized`。
- **Line 418 / 第 418 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 419 / 第 419 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 420 / 第 420 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 421 / 第 421 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 422 / 第 422 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 423 / 第 423 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 424 / 第 424 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 425 / 第 425 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 426 / 第 426 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 427 / 第 427 行**: EN: Invokes a macro that expands portability, ABI, or registration boilerplate. CN: 调用一个宏，以展开可移植性、ABI 或注册相关样板代码。
- **Line 428 / 第 428 行**: EN: Declares function or method `EnsureInterceptorsInitialized`. CN: 声明函数或方法 `EnsureInterceptorsInitialized`。
- **Line 429 / 第 429 行**: EN: Declares function or method `EnterLoader`. CN: 声明函数或方法 `EnterLoader`。
- **Line 430 / 第 430 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 431 / 第 431 行**: EN: Declares function or method `ExitLoader`. CN: 声明函数或方法 `ExitLoader`。
- **Line 432 / 第 432 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。

### Lines 433-448 / 第 433-448 行
```cpp
433 | }
434 | 
435 | INTERCEPTOR(int, dlclose, void *handle) {
436 |   EnsureInterceptorsInitialized();
437 |   EnterLoader();
438 |   int res = REAL(dlclose)(handle);
439 |   ExitLoader();
440 |   return res;
441 | }
442 | 
443 | static Mutex interceptor_init_lock;
444 | static bool interceptors_inited = false;
445 | 
446 | static void EnsureInterceptorsInitialized() {
447 |   Lock lock(&interceptor_init_lock);
448 |   if (interceptors_inited)
```
- **Line 433 / 第 433 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 434 / 第 434 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 435 / 第 435 行**: EN: Invokes a macro that expands portability, ABI, or registration boilerplate. CN: 调用一个宏，以展开可移植性、ABI 或注册相关样板代码。
- **Line 436 / 第 436 行**: EN: Declares function or method `EnsureInterceptorsInitialized`. CN: 声明函数或方法 `EnsureInterceptorsInitialized`。
- **Line 437 / 第 437 行**: EN: Declares function or method `EnterLoader`. CN: 声明函数或方法 `EnterLoader`。
- **Line 438 / 第 438 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 439 / 第 439 行**: EN: Declares function or method `ExitLoader`. CN: 声明函数或方法 `ExitLoader`。
- **Line 440 / 第 440 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 441 / 第 441 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 442 / 第 442 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 443 / 第 443 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 444 / 第 444 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 445 / 第 445 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 446 / 第 446 行**: EN: Starts the definition of function or method `EnsureInterceptorsInitialized`. CN: 开始定义函数或方法 `EnsureInterceptorsInitialized`。
- **Line 447 / 第 447 行**: EN: Declares function or method `lock`. CN: 声明函数或方法 `lock`。
- **Line 448 / 第 448 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。

### Lines 449-464 / 第 449-464 行
```cpp
449 |     return;
450 | 
451 |   INTERCEPT_FUNCTION(dlopen);
452 |   INTERCEPT_FUNCTION(dlclose);
453 | 
454 |   interceptors_inited = true;
455 | }
456 | 
457 | extern "C" SANITIZER_INTERFACE_ATTRIBUTE
458 | #if !SANITIZER_CAN_USE_PREINIT_ARRAY
459 | // On ELF platforms, the constructor is invoked using .preinit_array (see below)
460 | __attribute__((constructor(0)))
461 | #endif
462 | void __cfi_init() {
463 |   SanitizerToolName = "CFI";
464 |   InitializeFlags();
```
- **Line 449 / 第 449 行**: EN: Returns a value or transfers control to the caller. CN: 返回一个值，或将控制权交还给调用者。
- **Line 450 / 第 450 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 451 / 第 451 行**: EN: Invokes a macro that expands portability, ABI, or registration boilerplate. CN: 调用一个宏，以展开可移植性、ABI 或注册相关样板代码。
- **Line 452 / 第 452 行**: EN: Invokes a macro that expands portability, ABI, or registration boilerplate. CN: 调用一个宏，以展开可移植性、ABI 或注册相关样板代码。
- **Line 453 / 第 453 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 454 / 第 454 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 455 / 第 455 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 456 / 第 456 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 457 / 第 457 行**: EN: Marks an ABI-level compiler-rt symbol exported to generated code or other runtimes. CN: 标记导出给生成代码或其他运行时使用的 compiler-rt ABI 级符号。
- **Line 458 / 第 458 行**: EN: Starts a conditional-compilation guard. CN: 开始一个条件编译保护块。
- **Line 459 / 第 459 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 460 / 第 460 行**: EN: Starts the definition of function or method `__attribute__`. CN: 开始定义函数或方法 `__attribute__`。
- **Line 461 / 第 461 行**: EN: Closes the conditional-compilation block. CN: 结束条件编译块。
- **Line 462 / 第 462 行**: EN: Starts the definition of function or method `__cfi_init`. CN: 开始定义函数或方法 `__cfi_init`。
- **Line 463 / 第 463 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 464 / 第 464 行**: EN: Declares function or method `InitializeFlags`. CN: 声明函数或方法 `InitializeFlags`。

### Lines 465-480 / 第 465-480 行
```cpp
465 |   InitShadow();
466 | 
467 | #ifdef CFI_ENABLE_DIAG
468 |   __ubsan::InitAsPlugin();
469 | #endif
470 | }
471 | 
472 | #if SANITIZER_CAN_USE_PREINIT_ARRAY
473 | // On ELF platforms, run cfi initialization before any other constructors.
474 | // On other platforms we use the constructor attribute to arrange to run our
475 | // initialization early.
476 | extern "C" {
477 | __attribute__((section(".preinit_array"),
478 |                used)) void (*__cfi_preinit)(void) = __cfi_init;
479 | }
480 | #endif
```
- **Line 465 / 第 465 行**: EN: Declares function or method `InitShadow`. CN: 声明函数或方法 `InitShadow`。
- **Line 466 / 第 466 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 467 / 第 467 行**: EN: Starts a conditional-compilation guard. CN: 开始一个条件编译保护块。
- **Line 468 / 第 468 行**: EN: Declares function or method `__ubsan::InitAsPlugin`. CN: 声明函数或方法 `__ubsan::InitAsPlugin`。
- **Line 469 / 第 469 行**: EN: Closes the conditional-compilation block. CN: 结束条件编译块。
- **Line 470 / 第 470 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 471 / 第 471 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 472 / 第 472 行**: EN: Starts a conditional-compilation guard. CN: 开始一个条件编译保护块。
- **Line 473 / 第 473 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 474 / 第 474 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 475 / 第 475 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 476 / 第 476 行**: EN: Uses C linkage so the declaration keeps a stable ABI. CN: 使用 C 链接约定，以保持稳定 ABI。
- **Line 477 / 第 477 行**: EN: Continues a multi-line argument list, initializer, or aggregate entry. CN: 继续一个多行参数列表、初始化器或聚合项。
- **Line 478 / 第 478 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 479 / 第 479 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 480 / 第 480 行**: EN: Closes the conditional-compilation block. CN: 结束条件编译块。

## Key Concepts / 关键概念

- **EN**: control-flow integrity diagnostics
  - **CN**: 控制流完整性诊断
- **EN**: namespace scoping and organization
  - **CN**: 命名空间作用域与组织
- **EN**: shadow memory bookkeeping
  - **CN**: 影子内存簿记
- **EN**: interceptor-based runtime hooks
  - **CN**: 基于拦截器的运行时钩子

## Dependencies / 依赖关系

- `assert.h` — Local or subsystem header dependency / 本地或子系统头文件依赖
- `elf.h` — Local or subsystem header dependency / 本地或子系统头文件依赖
- `sanitizer_common/sanitizer_common.h` — compiler-rt runtime subsystem dependency / compiler-rt 运行时子系统依赖
- `sys/link_elf.h` — System or standard library dependency / 系统或标准库依赖
- `link.h` — Local or subsystem header dependency / 本地或子系统头文件依赖
- `string.h` — Local or subsystem header dependency / 本地或子系统头文件依赖
- `stdlib.h` — Local or subsystem header dependency / 本地或子系统头文件依赖
- `sys/mman.h` — System or standard library dependency / 系统或标准库依赖
- `interception/interception.h` — compiler-rt runtime subsystem dependency / compiler-rt 运行时子系统依赖
- `sanitizer_common/sanitizer_flag_parser.h` — compiler-rt runtime subsystem dependency / compiler-rt 运行时子系统依赖
- `ubsan/ubsan_init.h` — System or standard library dependency / 系统或标准库依赖
- `ubsan/ubsan_flags.h` — System or standard library dependency / 系统或标准库依赖
