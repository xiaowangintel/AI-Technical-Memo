# TargetLibraryInfo.td — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `llvm/include/llvm/Analysis/TargetLibraryInfo.td`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**: This TableGen file describes File that describes library functions records consumed by LLVM's analysis interfaces and cached program facts layer. / 该 TableGen 文件在 LLVM 的分析接口与缓存的程序事实层中描述 TargetLibraryInfo 相关记录，供生成阶段消费。

## Line-by-Line Analysis / 逐行分析

### Lines 1-28

```tablegen
//===-- TargetLibraryInfo.td - File that describes library functions ------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

include "llvm/Analysis/TargetLibraryInfoImpl.td"

/// void *operator new(unsigned int);
def msvc_new_int : TargetLibCall<"??2@YAPAXI@Z", Ptr, [Int]>;

/// void *operator new(unsigned int, const std::nothrow_t&);
def msvc_new_int_nothrow
    : TargetLibCall<"??2@YAPAXIABUnothrow_t@std@@@Z", Ptr, [Int, Ptr]>;

/// void *operator new(unsigned long long);
def msvc_new_longlong : TargetLibCall<"??2@YAPEAX_K@Z", Ptr, [LLong]>;

/// void *operator new(unsigned long long, const std::nothrow_t&);
def msvc_new_longlong_nothrow
    : TargetLibCall<"??2@YAPEAX_KAEBUnothrow_t@std@@@Z", Ptr, [LLong, Ptr]>;

/// void operator delete(void*);
def msvc_delete_ptr32 : TargetLibCall<"??3@YAXPAX@Z", Void, [Ptr]>;

/// void operator delete(void*, const std::nothrow_t&);
```

- **L1**: Banner comment that marks a major file section boundary. / 横幅注释，用于标记文件中的主要分节边界。
- **L2**: Separator comment used to visually divide nearby sections. / 分隔性注释，用于在视觉上划分附近的小节。
- **L3**: Comment documents the nearby API, invariant, or algorithmic intent: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`. / 这行注释说明了附近 API、不变量或算法意图：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L4**: Comment documents the nearby API, invariant, or algorithmic intent: `See https://llvm.org/LICENSE.txt for license information.`. / 这行注释说明了附近 API、不变量或算法意图：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5**: Comment documents the nearby API, invariant, or algorithmic intent: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`. / 这行注释说明了附近 API、不变量或算法意图：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6**: Separator comment used to visually divide nearby sections. / 分隔性注释，用于在视觉上划分附近的小节。
- **L7**: Banner comment that marks a major file section boundary. / 横幅注释，用于标记文件中的主要分节边界。
- **L8**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L9**: Includes TableGen description `llvm/Analysis/TargetLibraryInfoImpl.td` so this file can reuse its record classes and helper definitions. / 引入 TableGen 描述 `llvm/Analysis/TargetLibraryInfoImpl.td`，从而复用其中的记录类和辅助定义。
- **L10**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L11**: Comment documents the nearby API, invariant, or algorithmic intent: `void *operator new(unsigned int);`. / 这行注释说明了附近 API、不变量或算法意图：`void *operator new(unsigned int);`。
- **L12**: Defines TableGen record `msvc_new_int` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `msvc_new_int`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L13**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L14**: Comment documents the nearby API, invariant, or algorithmic intent: `void *operator new(unsigned int, const std::nothrow_t&);`. / 这行注释说明了附近 API、不变量或算法意图：`void *operator new(unsigned int, const std::nothrow_t&);`。
- **L15**: Defines TableGen record `msvc_new_int_nothrow`, adding one entry to the generated description database. / 定义 TableGen 记录 `msvc_new_int_nothrow`，向生成用描述数据库中加入一个条目。
- **L16**: Provides a declaration or standalone statement that contributes to the surrounding API or implementation. / 给出一条声明或独立语句，为周围的 API 或实现提供支撑。
- **L17**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L18**: Comment documents the nearby API, invariant, or algorithmic intent: `void *operator new(unsigned long long);`. / 这行注释说明了附近 API、不变量或算法意图：`void *operator new(unsigned long long);`。
- **L19**: Defines TableGen record `msvc_new_longlong` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `msvc_new_longlong`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L20**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L21**: Comment documents the nearby API, invariant, or algorithmic intent: `void *operator new(unsigned long long, const std::nothrow_t&);`. / 这行注释说明了附近 API、不变量或算法意图：`void *operator new(unsigned long long, const std::nothrow_t&);`。
- **L22**: Defines TableGen record `msvc_new_longlong_nothrow`, adding one entry to the generated description database. / 定义 TableGen 记录 `msvc_new_longlong_nothrow`，向生成用描述数据库中加入一个条目。
- **L23**: Provides a declaration or standalone statement that contributes to the surrounding API or implementation. / 给出一条声明或独立语句，为周围的 API 或实现提供支撑。
- **L24**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L25**: Comment documents the nearby API, invariant, or algorithmic intent: `void operator delete(void*);`. / 这行注释说明了附近 API、不变量或算法意图：`void operator delete(void*);`。
- **L26**: Defines TableGen record `msvc_delete_ptr32` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `msvc_delete_ptr32`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L27**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L28**: Comment documents the nearby API, invariant, or algorithmic intent: `void operator delete(void*, const std::nothrow_t&);`. / 这行注释说明了附近 API、不变量或算法意图：`void operator delete(void*, const std::nothrow_t&);`。

### Lines 29-56

```tablegen
def msvc_delete_ptr32_nothrow
    : TargetLibCall<"??3@YAXPAXABUnothrow_t@std@@@Z", Void, [Ptr, Ptr]>;

/// void operator delete(void*, unsigned int);
def msvc_delete_ptr32_int : TargetLibCall<"??3@YAXPAXI@Z", Void, [Ptr, Int]>;

/// void operator delete(void*);
def msvc_delete_ptr64 : TargetLibCall<"??3@YAXPEAX@Z", Void, [Ptr]>;

/// void operator delete(void*, const std::nothrow_t&);
def msvc_delete_ptr64_nothrow
    : TargetLibCall<"??3@YAXPEAXAEBUnothrow_t@std@@@Z", Void, [Ptr, Ptr]>;

/// void operator delete(void*, unsigned long long);
def msvc_delete_ptr64_longlong
    : TargetLibCall<"??3@YAXPEAX_K@Z", Void, [Ptr, LLong]>;

/// void *operator new[](unsigned int);
def msvc_new_array_int : TargetLibCall<"??_U@YAPAXI@Z", Ptr, [Int]>;

/// void *operator new](unsigned int, [const std::nothrow_t&);
def msvc_new_array_int_nothrow
    : TargetLibCall<"??_U@YAPAXIABUnothrow_t@std@@@Z", Ptr, [Int, Ptr]>;

/// void *operator new[](unsigned long long);
def msvc_new_array_longlong : TargetLibCall<"??_U@YAPEAX_K@Z", Ptr, [LLong]>;

/// void *operator new](unsigned long long, [const std::nothrow_t&);
```

- **L29**: Defines TableGen record `msvc_delete_ptr32_nothrow`, adding one entry to the generated description database. / 定义 TableGen 记录 `msvc_delete_ptr32_nothrow`，向生成用描述数据库中加入一个条目。
- **L30**: Provides a declaration or standalone statement that contributes to the surrounding API or implementation. / 给出一条声明或独立语句，为周围的 API 或实现提供支撑。
- **L31**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L32**: Comment documents the nearby API, invariant, or algorithmic intent: `void operator delete(void*, unsigned int);`. / 这行注释说明了附近 API、不变量或算法意图：`void operator delete(void*, unsigned int);`。
- **L33**: Defines TableGen record `msvc_delete_ptr32_int` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `msvc_delete_ptr32_int`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L34**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L35**: Comment documents the nearby API, invariant, or algorithmic intent: `void operator delete(void*);`. / 这行注释说明了附近 API、不变量或算法意图：`void operator delete(void*);`。
- **L36**: Defines TableGen record `msvc_delete_ptr64` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `msvc_delete_ptr64`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L37**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L38**: Comment documents the nearby API, invariant, or algorithmic intent: `void operator delete(void*, const std::nothrow_t&);`. / 这行注释说明了附近 API、不变量或算法意图：`void operator delete(void*, const std::nothrow_t&);`。
- **L39**: Defines TableGen record `msvc_delete_ptr64_nothrow`, adding one entry to the generated description database. / 定义 TableGen 记录 `msvc_delete_ptr64_nothrow`，向生成用描述数据库中加入一个条目。
- **L40**: Provides a declaration or standalone statement that contributes to the surrounding API or implementation. / 给出一条声明或独立语句，为周围的 API 或实现提供支撑。
- **L41**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L42**: Comment documents the nearby API, invariant, or algorithmic intent: `void operator delete(void*, unsigned long long);`. / 这行注释说明了附近 API、不变量或算法意图：`void operator delete(void*, unsigned long long);`。
- **L43**: Defines TableGen record `msvc_delete_ptr64_longlong`, adding one entry to the generated description database. / 定义 TableGen 记录 `msvc_delete_ptr64_longlong`，向生成用描述数据库中加入一个条目。
- **L44**: Provides a declaration or standalone statement that contributes to the surrounding API or implementation. / 给出一条声明或独立语句，为周围的 API 或实现提供支撑。
- **L45**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L46**: Comment documents the nearby API, invariant, or algorithmic intent: `void *operator new[](unsigned int);`. / 这行注释说明了附近 API、不变量或算法意图：`void *operator new[](unsigned int);`。
- **L47**: Defines TableGen record `msvc_new_array_int` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `msvc_new_array_int`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L48**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L49**: Comment documents the nearby API, invariant, or algorithmic intent: `void *operator new](unsigned int, [const std::nothrow_t&);`. / 这行注释说明了附近 API、不变量或算法意图：`void *operator new](unsigned int, [const std::nothrow_t&);`。
- **L50**: Defines TableGen record `msvc_new_array_int_nothrow`, adding one entry to the generated description database. / 定义 TableGen 记录 `msvc_new_array_int_nothrow`，向生成用描述数据库中加入一个条目。
- **L51**: Provides a declaration or standalone statement that contributes to the surrounding API or implementation. / 给出一条声明或独立语句，为周围的 API 或实现提供支撑。
- **L52**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L53**: Comment documents the nearby API, invariant, or algorithmic intent: `void *operator new[](unsigned long long);`. / 这行注释说明了附近 API、不变量或算法意图：`void *operator new[](unsigned long long);`。
- **L54**: Defines TableGen record `msvc_new_array_longlong` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `msvc_new_array_longlong`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L55**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L56**: Comment documents the nearby API, invariant, or algorithmic intent: `void *operator new](unsigned long long, [const std::nothrow_t&);`. / 这行注释说明了附近 API、不变量或算法意图：`void *operator new](unsigned long long, [const std::nothrow_t&);`。

### Lines 57-84

```tablegen
def msvc_new_array_longlong_nothrow
    : TargetLibCall<"??_U@YAPEAX_KAEBUnothrow_t@std@@@Z", Ptr, [LLong, Ptr]>;

/// void operator delete[](void*);
def msvc_delete_array_ptr32 : TargetLibCall<"??_V@YAXPAX@Z", Void, [Ptr]>;

/// void operator delete](void*, [const std::nothrow_t&);
def msvc_delete_array_ptr32_nothrow
    : TargetLibCall<"??_V@YAXPAXABUnothrow_t@std@@@Z", Void, [Ptr, Ptr]>;

/// void operator delete](void*, [unsigned int);
def msvc_delete_array_ptr32_int
    : TargetLibCall<"??_V@YAXPAXI@Z", Void, [Ptr, Int]>;

/// void operator delete[](void*);
def msvc_delete_array_ptr64 : TargetLibCall<"??_V@YAXPEAX@Z", Void, [Ptr]>;

/// void operator delete](void*, [const std::nothrow_t&);
def msvc_delete_array_ptr64_nothrow
    : TargetLibCall<"??_V@YAXPEAXAEBUnothrow_t@std@@@Z", Void, [Ptr, Ptr]>;

/// void operator delete](void*, [unsigned long long);
def msvc_delete_array_ptr64_longlong
    : TargetLibCall<"??_V@YAXPEAX_K@Z", Void, [Ptr, LLong]>;

/// int _IO_getc(_IO_FILE * __fp);
def under_IO_getc : TargetLibCall<"_IO_getc", Int, [Ptr]>;

```

- **L57**: Defines TableGen record `msvc_new_array_longlong_nothrow`, adding one entry to the generated description database. / 定义 TableGen 记录 `msvc_new_array_longlong_nothrow`，向生成用描述数据库中加入一个条目。
- **L58**: Provides a declaration or standalone statement that contributes to the surrounding API or implementation. / 给出一条声明或独立语句，为周围的 API 或实现提供支撑。
- **L59**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L60**: Comment documents the nearby API, invariant, or algorithmic intent: `void operator delete[](void*);`. / 这行注释说明了附近 API、不变量或算法意图：`void operator delete[](void*);`。
- **L61**: Defines TableGen record `msvc_delete_array_ptr32` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `msvc_delete_array_ptr32`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L62**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L63**: Comment documents the nearby API, invariant, or algorithmic intent: `void operator delete](void*, [const std::nothrow_t&);`. / 这行注释说明了附近 API、不变量或算法意图：`void operator delete](void*, [const std::nothrow_t&);`。
- **L64**: Defines TableGen record `msvc_delete_array_ptr32_nothrow`, adding one entry to the generated description database. / 定义 TableGen 记录 `msvc_delete_array_ptr32_nothrow`，向生成用描述数据库中加入一个条目。
- **L65**: Provides a declaration or standalone statement that contributes to the surrounding API or implementation. / 给出一条声明或独立语句，为周围的 API 或实现提供支撑。
- **L66**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L67**: Comment documents the nearby API, invariant, or algorithmic intent: `void operator delete](void*, [unsigned int);`. / 这行注释说明了附近 API、不变量或算法意图：`void operator delete](void*, [unsigned int);`。
- **L68**: Defines TableGen record `msvc_delete_array_ptr32_int`, adding one entry to the generated description database. / 定义 TableGen 记录 `msvc_delete_array_ptr32_int`，向生成用描述数据库中加入一个条目。
- **L69**: Provides a declaration or standalone statement that contributes to the surrounding API or implementation. / 给出一条声明或独立语句，为周围的 API 或实现提供支撑。
- **L70**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L71**: Comment documents the nearby API, invariant, or algorithmic intent: `void operator delete[](void*);`. / 这行注释说明了附近 API、不变量或算法意图：`void operator delete[](void*);`。
- **L72**: Defines TableGen record `msvc_delete_array_ptr64` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `msvc_delete_array_ptr64`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L73**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L74**: Comment documents the nearby API, invariant, or algorithmic intent: `void operator delete](void*, [const std::nothrow_t&);`. / 这行注释说明了附近 API、不变量或算法意图：`void operator delete](void*, [const std::nothrow_t&);`。
- **L75**: Defines TableGen record `msvc_delete_array_ptr64_nothrow`, adding one entry to the generated description database. / 定义 TableGen 记录 `msvc_delete_array_ptr64_nothrow`，向生成用描述数据库中加入一个条目。
- **L76**: Provides a declaration or standalone statement that contributes to the surrounding API or implementation. / 给出一条声明或独立语句，为周围的 API 或实现提供支撑。
- **L77**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L78**: Comment documents the nearby API, invariant, or algorithmic intent: `void operator delete](void*, [unsigned long long);`. / 这行注释说明了附近 API、不变量或算法意图：`void operator delete](void*, [unsigned long long);`。
- **L79**: Defines TableGen record `msvc_delete_array_ptr64_longlong`, adding one entry to the generated description database. / 定义 TableGen 记录 `msvc_delete_array_ptr64_longlong`，向生成用描述数据库中加入一个条目。
- **L80**: Provides a declaration or standalone statement that contributes to the surrounding API or implementation. / 给出一条声明或独立语句，为周围的 API 或实现提供支撑。
- **L81**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L82**: Comment documents the nearby API, invariant, or algorithmic intent: `int _IO_getc(_IO_FILE * __fp);`. / 这行注释说明了附近 API、不变量或算法意图：`int _IO_getc(_IO_FILE * __fp);`。
- **L83**: Defines TableGen record `under_IO_getc` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `under_IO_getc`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L84**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。

### Lines 85-112

```tablegen
/// int _IO_putc(int __c, _IO_FILE * __fp);
def under_IO_putc : TargetLibCall<"_IO_putc", Int, [Int, Ptr]>;

/// void operator delete[](void*);
def ZdaPv : TargetLibCall<"_ZdaPv", Void, [Ptr]>;

/// void operator delete](void*, [const std::nothrow_t&);
def ZdaPvRKSt9nothrow_t
    : TargetLibCall<"_ZdaPvRKSt9nothrow_t", Void, [Ptr, Ptr]>;

/// void operator delete](void*, [std::align_val_t);
def ZdaPvSt11align_val_t
    : TargetLibCall<"_ZdaPvSt11align_val_t", Void, [Ptr, IntPlus]>;

/// void operator delete](void*, [std::align_val_t, const std::nothrow_t&)
def ZdaPvSt11align_val_tRKSt9nothrow_t
    : TargetLibCall<"_ZdaPvSt11align_val_tRKSt9nothrow_t",
                    Void, [Ptr, IntPlus, Ptr]>;

/// void operator delete](void*, [unsigned int);
def ZdaPvj : TargetLibCall<"_ZdaPvj", Void, [Ptr, Int]>;

/// void operator delete](void*, [unsigned int, std::align_val_t);
def ZdaPvjSt11align_val_t
    : TargetLibCall<"_ZdaPvjSt11align_val_t", Void, [Ptr, Int, Int]>;

/// void operator delete](void*, [unsigned long);
def ZdaPvm : TargetLibCall<"_ZdaPvm", Void, [Ptr, Long]>;
```

- **L85**: Comment documents the nearby API, invariant, or algorithmic intent: `int _IO_putc(int __c, _IO_FILE * __fp);`. / 这行注释说明了附近 API、不变量或算法意图：`int _IO_putc(int __c, _IO_FILE * __fp);`。
- **L86**: Defines TableGen record `under_IO_putc` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `under_IO_putc`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L87**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L88**: Comment documents the nearby API, invariant, or algorithmic intent: `void operator delete[](void*);`. / 这行注释说明了附近 API、不变量或算法意图：`void operator delete[](void*);`。
- **L89**: Defines TableGen record `ZdaPv` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `ZdaPv`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L90**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L91**: Comment documents the nearby API, invariant, or algorithmic intent: `void operator delete](void*, [const std::nothrow_t&);`. / 这行注释说明了附近 API、不变量或算法意图：`void operator delete](void*, [const std::nothrow_t&);`。
- **L92**: Defines TableGen record `ZdaPvRKSt9nothrow_t`, adding one entry to the generated description database. / 定义 TableGen 记录 `ZdaPvRKSt9nothrow_t`，向生成用描述数据库中加入一个条目。
- **L93**: Provides a declaration or standalone statement that contributes to the surrounding API or implementation. / 给出一条声明或独立语句，为周围的 API 或实现提供支撑。
- **L94**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L95**: Comment documents the nearby API, invariant, or algorithmic intent: `void operator delete](void*, [std::align_val_t);`. / 这行注释说明了附近 API、不变量或算法意图：`void operator delete](void*, [std::align_val_t);`。
- **L96**: Defines TableGen record `ZdaPvSt11align_val_t`, adding one entry to the generated description database. / 定义 TableGen 记录 `ZdaPvSt11align_val_t`，向生成用描述数据库中加入一个条目。
- **L97**: Provides a declaration or standalone statement that contributes to the surrounding API or implementation. / 给出一条声明或独立语句，为周围的 API 或实现提供支撑。
- **L98**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L99**: Comment documents the nearby API, invariant, or algorithmic intent: `void operator delete](void*, [std::align_val_t, const std::nothrow_t&)`. / 这行注释说明了附近 API、不变量或算法意图：`void operator delete](void*, [std::align_val_t, const std::nothrow_t&)`。
- **L100**: Defines TableGen record `ZdaPvSt11align_val_tRKSt9nothrow_t`, adding one entry to the generated description database. / 定义 TableGen 记录 `ZdaPvSt11align_val_tRKSt9nothrow_t`，向生成用描述数据库中加入一个条目。
- **L101**: Continues the surrounding declaration or implementation logic. / 继续展开周围的声明或实现逻辑。
- **L102**: Provides a declaration or standalone statement that contributes to the surrounding API or implementation. / 给出一条声明或独立语句，为周围的 API 或实现提供支撑。
- **L103**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L104**: Comment documents the nearby API, invariant, or algorithmic intent: `void operator delete](void*, [unsigned int);`. / 这行注释说明了附近 API、不变量或算法意图：`void operator delete](void*, [unsigned int);`。
- **L105**: Defines TableGen record `ZdaPvj` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `ZdaPvj`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L106**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L107**: Comment documents the nearby API, invariant, or algorithmic intent: `void operator delete](void*, [unsigned int, std::align_val_t);`. / 这行注释说明了附近 API、不变量或算法意图：`void operator delete](void*, [unsigned int, std::align_val_t);`。
- **L108**: Defines TableGen record `ZdaPvjSt11align_val_t`, adding one entry to the generated description database. / 定义 TableGen 记录 `ZdaPvjSt11align_val_t`，向生成用描述数据库中加入一个条目。
- **L109**: Provides a declaration or standalone statement that contributes to the surrounding API or implementation. / 给出一条声明或独立语句，为周围的 API 或实现提供支撑。
- **L110**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L111**: Comment documents the nearby API, invariant, or algorithmic intent: `void operator delete](void*, [unsigned long);`. / 这行注释说明了附近 API、不变量或算法意图：`void operator delete](void*, [unsigned long);`。
- **L112**: Defines TableGen record `ZdaPvm` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `ZdaPvm`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。

### Lines 113-140

```tablegen

/// void operator delete](void*, [unsigned long, std::align_val_t);
def ZdaPvmSt11align_val_t
    : TargetLibCall<"_ZdaPvmSt11align_val_t", Void, [Ptr, Long, Long]>;

/// void operator delete(void*);
def ZdlPv : TargetLibCall<"_ZdlPv", Void, [Ptr]>;

/// void operator delete(void*, const std::nothrow_t&);
def ZdlPvRKSt9nothrow_t
    : TargetLibCall<"_ZdlPvRKSt9nothrow_t", Void, [Ptr, Ptr]>;

/// void operator delete(void*, std::align_val_t)
def ZdlPvSt11align_val_t
    : TargetLibCall<"_ZdlPvSt11align_val_t", Void, [Ptr, IntPlus]>;

/// void operator delete(void*, std::align_val_t, const std::nothrow_t&)
def ZdlPvSt11align_val_tRKSt9nothrow_t
    : TargetLibCall<"_ZdlPvSt11align_val_tRKSt9nothrow_t",
                    Void, [Ptr, IntPlus, Ptr]>;

/// void operator delete(void*, unsigned int);
def ZdlPvj : TargetLibCall<"_ZdlPvj", Void, [Ptr, Int]>;

/// void operator delete(void*, unsigned int, std::align_val_t)
def ZdlPvjSt11align_val_t
    : TargetLibCall<"_ZdlPvjSt11align_val_t", Void, [Ptr, Int, Int]>;

```

- **L113**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L114**: Comment documents the nearby API, invariant, or algorithmic intent: `void operator delete](void*, [unsigned long, std::align_val_t);`. / 这行注释说明了附近 API、不变量或算法意图：`void operator delete](void*, [unsigned long, std::align_val_t);`。
- **L115**: Defines TableGen record `ZdaPvmSt11align_val_t`, adding one entry to the generated description database. / 定义 TableGen 记录 `ZdaPvmSt11align_val_t`，向生成用描述数据库中加入一个条目。
- **L116**: Provides a declaration or standalone statement that contributes to the surrounding API or implementation. / 给出一条声明或独立语句，为周围的 API 或实现提供支撑。
- **L117**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L118**: Comment documents the nearby API, invariant, or algorithmic intent: `void operator delete(void*);`. / 这行注释说明了附近 API、不变量或算法意图：`void operator delete(void*);`。
- **L119**: Defines TableGen record `ZdlPv` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `ZdlPv`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L120**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L121**: Comment documents the nearby API, invariant, or algorithmic intent: `void operator delete(void*, const std::nothrow_t&);`. / 这行注释说明了附近 API、不变量或算法意图：`void operator delete(void*, const std::nothrow_t&);`。
- **L122**: Defines TableGen record `ZdlPvRKSt9nothrow_t`, adding one entry to the generated description database. / 定义 TableGen 记录 `ZdlPvRKSt9nothrow_t`，向生成用描述数据库中加入一个条目。
- **L123**: Provides a declaration or standalone statement that contributes to the surrounding API or implementation. / 给出一条声明或独立语句，为周围的 API 或实现提供支撑。
- **L124**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L125**: Comment documents the nearby API, invariant, or algorithmic intent: `void operator delete(void*, std::align_val_t)`. / 这行注释说明了附近 API、不变量或算法意图：`void operator delete(void*, std::align_val_t)`。
- **L126**: Defines TableGen record `ZdlPvSt11align_val_t`, adding one entry to the generated description database. / 定义 TableGen 记录 `ZdlPvSt11align_val_t`，向生成用描述数据库中加入一个条目。
- **L127**: Provides a declaration or standalone statement that contributes to the surrounding API or implementation. / 给出一条声明或独立语句，为周围的 API 或实现提供支撑。
- **L128**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L129**: Comment documents the nearby API, invariant, or algorithmic intent: `void operator delete(void*, std::align_val_t, const std::nothrow_t&)`. / 这行注释说明了附近 API、不变量或算法意图：`void operator delete(void*, std::align_val_t, const std::nothrow_t&)`。
- **L130**: Defines TableGen record `ZdlPvSt11align_val_tRKSt9nothrow_t`, adding one entry to the generated description database. / 定义 TableGen 记录 `ZdlPvSt11align_val_tRKSt9nothrow_t`，向生成用描述数据库中加入一个条目。
- **L131**: Continues the surrounding declaration or implementation logic. / 继续展开周围的声明或实现逻辑。
- **L132**: Provides a declaration or standalone statement that contributes to the surrounding API or implementation. / 给出一条声明或独立语句，为周围的 API 或实现提供支撑。
- **L133**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L134**: Comment documents the nearby API, invariant, or algorithmic intent: `void operator delete(void*, unsigned int);`. / 这行注释说明了附近 API、不变量或算法意图：`void operator delete(void*, unsigned int);`。
- **L135**: Defines TableGen record `ZdlPvj` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `ZdlPvj`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L136**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L137**: Comment documents the nearby API, invariant, or algorithmic intent: `void operator delete(void*, unsigned int, std::align_val_t)`. / 这行注释说明了附近 API、不变量或算法意图：`void operator delete(void*, unsigned int, std::align_val_t)`。
- **L138**: Defines TableGen record `ZdlPvjSt11align_val_t`, adding one entry to the generated description database. / 定义 TableGen 记录 `ZdlPvjSt11align_val_t`，向生成用描述数据库中加入一个条目。
- **L139**: Provides a declaration or standalone statement that contributes to the surrounding API or implementation. / 给出一条声明或独立语句，为周围的 API 或实现提供支撑。
- **L140**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。

### Lines 141-168

```tablegen
/// void operator delete(void*, unsigned long);
def ZdlPvm : TargetLibCall<"_ZdlPvm", Void, [Ptr, Long]>;

/// void operator delete(void*, unsigned long, std::align_val_t)
def ZdlPvmSt11align_val_t
    : TargetLibCall<"_ZdlPvmSt11align_val_t", Void, [Ptr, Long, Long]>;

/// void *operator new[](unsigned int);
def Znaj : TargetLibCall<"_Znaj", Ptr, [Int]>;

/// void *operator new](unsigned int, [const std::nothrow_t&);
def ZnajRKSt9nothrow_t : TargetLibCall<"_ZnajRKSt9nothrow_t", Ptr, [Int, Ptr]>;

/// void *operator new](unsigned int, [std::align_val_t)
def ZnajSt11align_val_t
    : TargetLibCall<"_ZnajSt11align_val_t", Ptr, [Int, Int]>;

/// void *operator new](unsigned int, [std::align_val_t, const std::nothrow_t&)
def ZnajSt11align_val_tRKSt9nothrow_t
    : TargetLibCall<"_ZnajSt11align_val_tRKSt9nothrow_t", Ptr, [Int, Int, Ptr]>;

/// void *operator new[](unsigned long);
def Znam : TargetLibCall<"_Znam", Ptr, [Long]>;

/// void *operator new](unsigned long, [__hot_cold_t)
/// Currently this and other operator new interfaces that take a __hot_cold_t
/// hint are supported by the open source version of tcmalloc, see:
/// https://github.com/google/tcmalloc/blob/master/tcmalloc/new_extension.h
```

- **L141**: Comment documents the nearby API, invariant, or algorithmic intent: `void operator delete(void*, unsigned long);`. / 这行注释说明了附近 API、不变量或算法意图：`void operator delete(void*, unsigned long);`。
- **L142**: Defines TableGen record `ZdlPvm` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `ZdlPvm`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L143**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L144**: Comment documents the nearby API, invariant, or algorithmic intent: `void operator delete(void*, unsigned long, std::align_val_t)`. / 这行注释说明了附近 API、不变量或算法意图：`void operator delete(void*, unsigned long, std::align_val_t)`。
- **L145**: Defines TableGen record `ZdlPvmSt11align_val_t`, adding one entry to the generated description database. / 定义 TableGen 记录 `ZdlPvmSt11align_val_t`，向生成用描述数据库中加入一个条目。
- **L146**: Provides a declaration or standalone statement that contributes to the surrounding API or implementation. / 给出一条声明或独立语句，为周围的 API 或实现提供支撑。
- **L147**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L148**: Comment documents the nearby API, invariant, or algorithmic intent: `void *operator new[](unsigned int);`. / 这行注释说明了附近 API、不变量或算法意图：`void *operator new[](unsigned int);`。
- **L149**: Defines TableGen record `Znaj` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `Znaj`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L150**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L151**: Comment documents the nearby API, invariant, or algorithmic intent: `void *operator new](unsigned int, [const std::nothrow_t&);`. / 这行注释说明了附近 API、不变量或算法意图：`void *operator new](unsigned int, [const std::nothrow_t&);`。
- **L152**: Defines TableGen record `ZnajRKSt9nothrow_t` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `ZnajRKSt9nothrow_t`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L153**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L154**: Comment documents the nearby API, invariant, or algorithmic intent: `void *operator new](unsigned int, [std::align_val_t)`. / 这行注释说明了附近 API、不变量或算法意图：`void *operator new](unsigned int, [std::align_val_t)`。
- **L155**: Defines TableGen record `ZnajSt11align_val_t`, adding one entry to the generated description database. / 定义 TableGen 记录 `ZnajSt11align_val_t`，向生成用描述数据库中加入一个条目。
- **L156**: Provides a declaration or standalone statement that contributes to the surrounding API or implementation. / 给出一条声明或独立语句，为周围的 API 或实现提供支撑。
- **L157**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L158**: Comment documents the nearby API, invariant, or algorithmic intent: `void *operator new](unsigned int, [std::align_val_t, const std::nothrow_t&)`. / 这行注释说明了附近 API、不变量或算法意图：`void *operator new](unsigned int, [std::align_val_t, const std::nothrow_t&)`。
- **L159**: Defines TableGen record `ZnajSt11align_val_tRKSt9nothrow_t`, adding one entry to the generated description database. / 定义 TableGen 记录 `ZnajSt11align_val_tRKSt9nothrow_t`，向生成用描述数据库中加入一个条目。
- **L160**: Provides a declaration or standalone statement that contributes to the surrounding API or implementation. / 给出一条声明或独立语句，为周围的 API 或实现提供支撑。
- **L161**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L162**: Comment documents the nearby API, invariant, or algorithmic intent: `void *operator new[](unsigned long);`. / 这行注释说明了附近 API、不变量或算法意图：`void *operator new[](unsigned long);`。
- **L163**: Defines TableGen record `Znam` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `Znam`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L164**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L165**: Comment documents the nearby API, invariant, or algorithmic intent: `void *operator new](unsigned long, [__hot_cold_t)`. / 这行注释说明了附近 API、不变量或算法意图：`void *operator new](unsigned long, [__hot_cold_t)`。
- **L166**: Comment documents the nearby API, invariant, or algorithmic intent: `Currently this and other operator new interfaces that take a __hot_cold_t`. / 这行注释说明了附近 API、不变量或算法意图：`Currently this and other operator new interfaces that take a __hot_cold_t`。
- **L167**: Comment documents the nearby API, invariant, or algorithmic intent: `hint are supported by the open source version of tcmalloc, see:`. / 这行注释说明了附近 API、不变量或算法意图：`hint are supported by the open source version of tcmalloc, see:`。
- **L168**: Comment documents the nearby API, invariant, or algorithmic intent: `https://github.com/google/tcmalloc/blob/master/tcmalloc/new_extension.h`. / 这行注释说明了附近 API、不变量或算法意图：`https://github.com/google/tcmalloc/blob/master/tcmalloc/new_extension.h`。

### Lines 169-196

```tablegen
/// and for the definition of the __hot_cold_t parameter see:
/// https://github.com/google/tcmalloc/blob/master/tcmalloc/malloc_extension.h
def Znam12__hot_cold_t
    : TargetLibCall<"_Znam12__hot_cold_t", Ptr, [Long, Bool]>;

/// void *operator new](unsigned long, [const std::nothrow_t&);
def ZnamRKSt9nothrow_t : TargetLibCall<"_ZnamRKSt9nothrow_t", Ptr, [Long, Ptr]>;

/// void *operator new](unsigned long, [const std::nothrow_t&, __hot_cold_t)
def ZnamRKSt9nothrow_t12__hot_cold_t
    : TargetLibCall<"_ZnamRKSt9nothrow_t12__hot_cold_t",
                    Ptr, [Long, Ptr, Bool]>;

/// void *operator new](unsigned long, [std::align_val_t)
def ZnamSt11align_val_t
    : TargetLibCall<"_ZnamSt11align_val_t", Ptr, [Long, Long]>;

/// void *operator new](unsigned long, [std::align_val_t, __hot_cold_t)
def ZnamSt11align_val_t12__hot_cold_t
    : TargetLibCall<"_ZnamSt11align_val_t12__hot_cold_t",
                    Ptr, [Long, Long, Bool]>;

/// void *operator new](unsigned long, [std::align_val_t, const std::nothrow_t&)
def ZnamSt11align_val_tRKSt9nothrow_t
    : TargetLibCall<"_ZnamSt11align_val_tRKSt9nothrow_t",
                    Ptr, [Long, Long, Ptr]>;

/// void *operator new](unsigned long, [std::align_val_t, const std::nothrow_t&,
```

- **L169**: Comment documents the nearby API, invariant, or algorithmic intent: `and for the definition of the __hot_cold_t parameter see:`. / 这行注释说明了附近 API、不变量或算法意图：`and for the definition of the __hot_cold_t parameter see:`。
- **L170**: Comment documents the nearby API, invariant, or algorithmic intent: `https://github.com/google/tcmalloc/blob/master/tcmalloc/malloc_extension.h`. / 这行注释说明了附近 API、不变量或算法意图：`https://github.com/google/tcmalloc/blob/master/tcmalloc/malloc_extension.h`。
- **L171**: Defines TableGen record `Znam12__hot_cold_t`, adding one entry to the generated description database. / 定义 TableGen 记录 `Znam12__hot_cold_t`，向生成用描述数据库中加入一个条目。
- **L172**: Provides a declaration or standalone statement that contributes to the surrounding API or implementation. / 给出一条声明或独立语句，为周围的 API 或实现提供支撑。
- **L173**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L174**: Comment documents the nearby API, invariant, or algorithmic intent: `void *operator new](unsigned long, [const std::nothrow_t&);`. / 这行注释说明了附近 API、不变量或算法意图：`void *operator new](unsigned long, [const std::nothrow_t&);`。
- **L175**: Defines TableGen record `ZnamRKSt9nothrow_t` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `ZnamRKSt9nothrow_t`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L176**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L177**: Comment documents the nearby API, invariant, or algorithmic intent: `void *operator new](unsigned long, [const std::nothrow_t&, __hot_cold_t)`. / 这行注释说明了附近 API、不变量或算法意图：`void *operator new](unsigned long, [const std::nothrow_t&, __hot_cold_t)`。
- **L178**: Defines TableGen record `ZnamRKSt9nothrow_t12__hot_cold_t`, adding one entry to the generated description database. / 定义 TableGen 记录 `ZnamRKSt9nothrow_t12__hot_cold_t`，向生成用描述数据库中加入一个条目。
- **L179**: Continues the surrounding declaration or implementation logic. / 继续展开周围的声明或实现逻辑。
- **L180**: Provides a declaration or standalone statement that contributes to the surrounding API or implementation. / 给出一条声明或独立语句，为周围的 API 或实现提供支撑。
- **L181**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L182**: Comment documents the nearby API, invariant, or algorithmic intent: `void *operator new](unsigned long, [std::align_val_t)`. / 这行注释说明了附近 API、不变量或算法意图：`void *operator new](unsigned long, [std::align_val_t)`。
- **L183**: Defines TableGen record `ZnamSt11align_val_t`, adding one entry to the generated description database. / 定义 TableGen 记录 `ZnamSt11align_val_t`，向生成用描述数据库中加入一个条目。
- **L184**: Provides a declaration or standalone statement that contributes to the surrounding API or implementation. / 给出一条声明或独立语句，为周围的 API 或实现提供支撑。
- **L185**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L186**: Comment documents the nearby API, invariant, or algorithmic intent: `void *operator new](unsigned long, [std::align_val_t, __hot_cold_t)`. / 这行注释说明了附近 API、不变量或算法意图：`void *operator new](unsigned long, [std::align_val_t, __hot_cold_t)`。
- **L187**: Defines TableGen record `ZnamSt11align_val_t12__hot_cold_t`, adding one entry to the generated description database. / 定义 TableGen 记录 `ZnamSt11align_val_t12__hot_cold_t`，向生成用描述数据库中加入一个条目。
- **L188**: Continues the surrounding declaration or implementation logic. / 继续展开周围的声明或实现逻辑。
- **L189**: Provides a declaration or standalone statement that contributes to the surrounding API or implementation. / 给出一条声明或独立语句，为周围的 API 或实现提供支撑。
- **L190**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L191**: Comment documents the nearby API, invariant, or algorithmic intent: `void *operator new](unsigned long, [std::align_val_t, const std::nothrow_t&)`. / 这行注释说明了附近 API、不变量或算法意图：`void *operator new](unsigned long, [std::align_val_t, const std::nothrow_t&)`。
- **L192**: Defines TableGen record `ZnamSt11align_val_tRKSt9nothrow_t`, adding one entry to the generated description database. / 定义 TableGen 记录 `ZnamSt11align_val_tRKSt9nothrow_t`，向生成用描述数据库中加入一个条目。
- **L193**: Continues the surrounding declaration or implementation logic. / 继续展开周围的声明或实现逻辑。
- **L194**: Provides a declaration or standalone statement that contributes to the surrounding API or implementation. / 给出一条声明或独立语句，为周围的 API 或实现提供支撑。
- **L195**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L196**: Comment documents the nearby API, invariant, or algorithmic intent: `void *operator new](unsigned long, [std::align_val_t, const std::nothrow_t&,`. / 这行注释说明了附近 API、不变量或算法意图：`void *operator new](unsigned long, [std::align_val_t, const std::nothrow_t&,`。

### Lines 197-224

```tablegen
/// __hot_cold_t)
def ZnamSt11align_val_tRKSt9nothrow_t12__hot_cold_t
    : TargetLibCall<"_ZnamSt11align_val_tRKSt9nothrow_t12__hot_cold_t",
                    Ptr, [Long, Long, Ptr, Bool]>;

/// void *operator new(unsigned int);
def Znwj : TargetLibCall<"_Znwj", Ptr, [Int]>;

/// void *operator new(unsigned int, const std::nothrow_t&);
def ZnwjRKSt9nothrow_t : TargetLibCall<"_ZnwjRKSt9nothrow_t", Ptr, [Int, Ptr]>;

/// void *operator new(unsigned int, std::align_val_t)
def ZnwjSt11align_val_t
    : TargetLibCall<"_ZnwjSt11align_val_t", Ptr, [Int, Int]>;

/// void *operator new(unsigned int, std::align_val_t, const std::nothrow_t&)
def ZnwjSt11align_val_tRKSt9nothrow_t
    : TargetLibCall<"_ZnwjSt11align_val_tRKSt9nothrow_t", Ptr, [Int, Int, Ptr]>;

/// void *operator new(unsigned long);
def Znwm : TargetLibCall<"_Znwm", Ptr, [Long]>;

/// void *operator new(unsigned long, __hot_cold_t)
def Znwm12__hot_cold_t
    : TargetLibCall<"_Znwm12__hot_cold_t", Ptr, [Long, Bool]>;

/// void *operator new(unsigned long, const std::nothrow_t&);
def ZnwmRKSt9nothrow_t : TargetLibCall<"_ZnwmRKSt9nothrow_t", Ptr, [Long, Ptr]>;
```

- **L197**: Comment documents the nearby API, invariant, or algorithmic intent: `__hot_cold_t)`. / 这行注释说明了附近 API、不变量或算法意图：`__hot_cold_t)`。
- **L198**: Defines TableGen record `ZnamSt11align_val_tRKSt9nothrow_t12__hot_cold_t`, adding one entry to the generated description database. / 定义 TableGen 记录 `ZnamSt11align_val_tRKSt9nothrow_t12__hot_cold_t`，向生成用描述数据库中加入一个条目。
- **L199**: Continues the surrounding declaration or implementation logic. / 继续展开周围的声明或实现逻辑。
- **L200**: Provides a declaration or standalone statement that contributes to the surrounding API or implementation. / 给出一条声明或独立语句，为周围的 API 或实现提供支撑。
- **L201**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L202**: Comment documents the nearby API, invariant, or algorithmic intent: `void *operator new(unsigned int);`. / 这行注释说明了附近 API、不变量或算法意图：`void *operator new(unsigned int);`。
- **L203**: Defines TableGen record `Znwj` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `Znwj`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L204**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L205**: Comment documents the nearby API, invariant, or algorithmic intent: `void *operator new(unsigned int, const std::nothrow_t&);`. / 这行注释说明了附近 API、不变量或算法意图：`void *operator new(unsigned int, const std::nothrow_t&);`。
- **L206**: Defines TableGen record `ZnwjRKSt9nothrow_t` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `ZnwjRKSt9nothrow_t`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L207**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L208**: Comment documents the nearby API, invariant, or algorithmic intent: `void *operator new(unsigned int, std::align_val_t)`. / 这行注释说明了附近 API、不变量或算法意图：`void *operator new(unsigned int, std::align_val_t)`。
- **L209**: Defines TableGen record `ZnwjSt11align_val_t`, adding one entry to the generated description database. / 定义 TableGen 记录 `ZnwjSt11align_val_t`，向生成用描述数据库中加入一个条目。
- **L210**: Provides a declaration or standalone statement that contributes to the surrounding API or implementation. / 给出一条声明或独立语句，为周围的 API 或实现提供支撑。
- **L211**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L212**: Comment documents the nearby API, invariant, or algorithmic intent: `void *operator new(unsigned int, std::align_val_t, const std::nothrow_t&)`. / 这行注释说明了附近 API、不变量或算法意图：`void *operator new(unsigned int, std::align_val_t, const std::nothrow_t&)`。
- **L213**: Defines TableGen record `ZnwjSt11align_val_tRKSt9nothrow_t`, adding one entry to the generated description database. / 定义 TableGen 记录 `ZnwjSt11align_val_tRKSt9nothrow_t`，向生成用描述数据库中加入一个条目。
- **L214**: Provides a declaration or standalone statement that contributes to the surrounding API or implementation. / 给出一条声明或独立语句，为周围的 API 或实现提供支撑。
- **L215**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L216**: Comment documents the nearby API, invariant, or algorithmic intent: `void *operator new(unsigned long);`. / 这行注释说明了附近 API、不变量或算法意图：`void *operator new(unsigned long);`。
- **L217**: Defines TableGen record `Znwm` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `Znwm`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L218**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L219**: Comment documents the nearby API, invariant, or algorithmic intent: `void *operator new(unsigned long, __hot_cold_t)`. / 这行注释说明了附近 API、不变量或算法意图：`void *operator new(unsigned long, __hot_cold_t)`。
- **L220**: Defines TableGen record `Znwm12__hot_cold_t`, adding one entry to the generated description database. / 定义 TableGen 记录 `Znwm12__hot_cold_t`，向生成用描述数据库中加入一个条目。
- **L221**: Provides a declaration or standalone statement that contributes to the surrounding API or implementation. / 给出一条声明或独立语句，为周围的 API 或实现提供支撑。
- **L222**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L223**: Comment documents the nearby API, invariant, or algorithmic intent: `void *operator new(unsigned long, const std::nothrow_t&);`. / 这行注释说明了附近 API、不变量或算法意图：`void *operator new(unsigned long, const std::nothrow_t&);`。
- **L224**: Defines TableGen record `ZnwmRKSt9nothrow_t` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `ZnwmRKSt9nothrow_t`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。

### Lines 225-252

```tablegen

/// void *operator new(unsigned long, const std::nothrow_t&, __hot_cold_t)
def ZnwmRKSt9nothrow_t12__hot_cold_t
    : TargetLibCall<"_ZnwmRKSt9nothrow_t12__hot_cold_t",
                    Ptr, [Long, Ptr, Bool]>;

/// void *operator new(unsigned long, std::align_val_t)
def ZnwmSt11align_val_t
    : TargetLibCall<"_ZnwmSt11align_val_t", Ptr, [Long, Long]>;

/// void *operator new(unsigned long, std::align_val_t, __hot_cold_t)
def ZnwmSt11align_val_t12__hot_cold_t
    : TargetLibCall<"_ZnwmSt11align_val_t12__hot_cold_t",
                    Ptr, [Long, Long, Bool]>;

/// void *operator new(unsigned long, std::align_val_t, const std::nothrow_t&)
def ZnwmSt11align_val_tRKSt9nothrow_t
    : TargetLibCall<"_ZnwmSt11align_val_tRKSt9nothrow_t",
                    Ptr, [Long, Long, Ptr]>;

/// void *operator new(unsigned long, std::align_val_t, const std::nothrow_t&,
/// __hot_cold_t)
def ZnwmSt11align_val_tRKSt9nothrow_t12__hot_cold_t
    : TargetLibCall<"_ZnwmSt11align_val_tRKSt9nothrow_t12__hot_cold_t",
                    Ptr, [Long, Long, Ptr, Bool]>;

/// The following are variants of operator new which return the actual size
/// reserved by the allocator proposed in P0901R5 (Size feedback in operator
```

- **L225**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L226**: Comment documents the nearby API, invariant, or algorithmic intent: `void *operator new(unsigned long, const std::nothrow_t&, __hot_cold_t)`. / 这行注释说明了附近 API、不变量或算法意图：`void *operator new(unsigned long, const std::nothrow_t&, __hot_cold_t)`。
- **L227**: Defines TableGen record `ZnwmRKSt9nothrow_t12__hot_cold_t`, adding one entry to the generated description database. / 定义 TableGen 记录 `ZnwmRKSt9nothrow_t12__hot_cold_t`，向生成用描述数据库中加入一个条目。
- **L228**: Continues the surrounding declaration or implementation logic. / 继续展开周围的声明或实现逻辑。
- **L229**: Provides a declaration or standalone statement that contributes to the surrounding API or implementation. / 给出一条声明或独立语句，为周围的 API 或实现提供支撑。
- **L230**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L231**: Comment documents the nearby API, invariant, or algorithmic intent: `void *operator new(unsigned long, std::align_val_t)`. / 这行注释说明了附近 API、不变量或算法意图：`void *operator new(unsigned long, std::align_val_t)`。
- **L232**: Defines TableGen record `ZnwmSt11align_val_t`, adding one entry to the generated description database. / 定义 TableGen 记录 `ZnwmSt11align_val_t`，向生成用描述数据库中加入一个条目。
- **L233**: Provides a declaration or standalone statement that contributes to the surrounding API or implementation. / 给出一条声明或独立语句，为周围的 API 或实现提供支撑。
- **L234**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L235**: Comment documents the nearby API, invariant, or algorithmic intent: `void *operator new(unsigned long, std::align_val_t, __hot_cold_t)`. / 这行注释说明了附近 API、不变量或算法意图：`void *operator new(unsigned long, std::align_val_t, __hot_cold_t)`。
- **L236**: Defines TableGen record `ZnwmSt11align_val_t12__hot_cold_t`, adding one entry to the generated description database. / 定义 TableGen 记录 `ZnwmSt11align_val_t12__hot_cold_t`，向生成用描述数据库中加入一个条目。
- **L237**: Continues the surrounding declaration or implementation logic. / 继续展开周围的声明或实现逻辑。
- **L238**: Provides a declaration or standalone statement that contributes to the surrounding API or implementation. / 给出一条声明或独立语句，为周围的 API 或实现提供支撑。
- **L239**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L240**: Comment documents the nearby API, invariant, or algorithmic intent: `void *operator new(unsigned long, std::align_val_t, const std::nothrow_t&)`. / 这行注释说明了附近 API、不变量或算法意图：`void *operator new(unsigned long, std::align_val_t, const std::nothrow_t&)`。
- **L241**: Defines TableGen record `ZnwmSt11align_val_tRKSt9nothrow_t`, adding one entry to the generated description database. / 定义 TableGen 记录 `ZnwmSt11align_val_tRKSt9nothrow_t`，向生成用描述数据库中加入一个条目。
- **L242**: Continues the surrounding declaration or implementation logic. / 继续展开周围的声明或实现逻辑。
- **L243**: Provides a declaration or standalone statement that contributes to the surrounding API or implementation. / 给出一条声明或独立语句，为周围的 API 或实现提供支撑。
- **L244**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L245**: Comment documents the nearby API, invariant, or algorithmic intent: `void *operator new(unsigned long, std::align_val_t, const std::nothrow_t&,`. / 这行注释说明了附近 API、不变量或算法意图：`void *operator new(unsigned long, std::align_val_t, const std::nothrow_t&,`。
- **L246**: Comment documents the nearby API, invariant, or algorithmic intent: `__hot_cold_t)`. / 这行注释说明了附近 API、不变量或算法意图：`__hot_cold_t)`。
- **L247**: Defines TableGen record `ZnwmSt11align_val_tRKSt9nothrow_t12__hot_cold_t`, adding one entry to the generated description database. / 定义 TableGen 记录 `ZnwmSt11align_val_tRKSt9nothrow_t12__hot_cold_t`，向生成用描述数据库中加入一个条目。
- **L248**: Continues the surrounding declaration or implementation logic. / 继续展开周围的声明或实现逻辑。
- **L249**: Provides a declaration or standalone statement that contributes to the surrounding API or implementation. / 给出一条声明或独立语句，为周围的 API 或实现提供支撑。
- **L250**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L251**: Comment documents the nearby API, invariant, or algorithmic intent: `The following are variants of operator new which return the actual size`. / 这行注释说明了附近 API、不变量或算法意图：`The following are variants of operator new which return the actual size`。
- **L252**: Comment documents the nearby API, invariant, or algorithmic intent: `reserved by the allocator proposed in P0901R5 (Size feedback in operator`. / 这行注释说明了附近 API、不变量或算法意图：`reserved by the allocator proposed in P0901R5 (Size feedback in operator`。

### Lines 253-280

```tablegen
/// new). https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p0901r5.html
/// They are implemented by tcmalloc, see source at
/// https://github.com/google/tcmalloc/blob/master/tcmalloc/malloc_extension.h

/// __sized_ptr_t __size_returning_new(size_t size)
def size_returning_new
    : TargetLibCall<"__size_returning_new", ? /* Checked manually. */>;

/// __sized_ptr_t __size_returning_new_hot_cold(size_t, __hot_cold_t)
def size_returning_new_hot_cold
    : TargetLibCall<"__size_returning_new_hot_cold", ? /* Checked manually. */>;

/// __sized_ptr_t __size_returning_new_aligned(size_t, std::align_val_t)
def size_returning_new_aligned
    : TargetLibCall<"__size_returning_new_aligned", ? /* Checked manually. */>;

/// __sized_ptr_t __size_returning_new_aligned(size_t, std::align_val_t,
/// __hot_cold_t)
def size_returning_new_aligned_hot_cold
    : TargetLibCall<"__size_returning_new_aligned_hot_cold",
                    ? /* Checked manually. */>;

/// double __acos_finite(double x);
def acos_finite : TargetLibCall<"__acos_finite", Dbl, [Dbl]>;

/// float __acosf_finite(float x);
def acosf_finite : TargetLibCall<"__acosf_finite", Flt, [Flt]>;

```

- **L253**: Comment documents the nearby API, invariant, or algorithmic intent: `new). https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p0901r5.html`. / 这行注释说明了附近 API、不变量或算法意图：`new). https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p0901r5.html`。
- **L254**: Comment documents the nearby API, invariant, or algorithmic intent: `They are implemented by tcmalloc, see source at`. / 这行注释说明了附近 API、不变量或算法意图：`They are implemented by tcmalloc, see source at`。
- **L255**: Comment documents the nearby API, invariant, or algorithmic intent: `https://github.com/google/tcmalloc/blob/master/tcmalloc/malloc_extension.h`. / 这行注释说明了附近 API、不变量或算法意图：`https://github.com/google/tcmalloc/blob/master/tcmalloc/malloc_extension.h`。
- **L256**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L257**: Comment documents the nearby API, invariant, or algorithmic intent: `__sized_ptr_t __size_returning_new(size_t size)`. / 这行注释说明了附近 API、不变量或算法意图：`__sized_ptr_t __size_returning_new(size_t size)`。
- **L258**: Defines TableGen record `size_returning_new`, adding one entry to the generated description database. / 定义 TableGen 记录 `size_returning_new`，向生成用描述数据库中加入一个条目。
- **L259**: Provides a declaration or standalone statement that contributes to the surrounding API or implementation. / 给出一条声明或独立语句，为周围的 API 或实现提供支撑。
- **L260**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L261**: Comment documents the nearby API, invariant, or algorithmic intent: `__sized_ptr_t __size_returning_new_hot_cold(size_t, __hot_cold_t)`. / 这行注释说明了附近 API、不变量或算法意图：`__sized_ptr_t __size_returning_new_hot_cold(size_t, __hot_cold_t)`。
- **L262**: Defines TableGen record `size_returning_new_hot_cold`, adding one entry to the generated description database. / 定义 TableGen 记录 `size_returning_new_hot_cold`，向生成用描述数据库中加入一个条目。
- **L263**: Provides a declaration or standalone statement that contributes to the surrounding API or implementation. / 给出一条声明或独立语句，为周围的 API 或实现提供支撑。
- **L264**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L265**: Comment documents the nearby API, invariant, or algorithmic intent: `__sized_ptr_t __size_returning_new_aligned(size_t, std::align_val_t)`. / 这行注释说明了附近 API、不变量或算法意图：`__sized_ptr_t __size_returning_new_aligned(size_t, std::align_val_t)`。
- **L266**: Defines TableGen record `size_returning_new_aligned`, adding one entry to the generated description database. / 定义 TableGen 记录 `size_returning_new_aligned`，向生成用描述数据库中加入一个条目。
- **L267**: Provides a declaration or standalone statement that contributes to the surrounding API or implementation. / 给出一条声明或独立语句，为周围的 API 或实现提供支撑。
- **L268**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L269**: Comment documents the nearby API, invariant, or algorithmic intent: `__sized_ptr_t __size_returning_new_aligned(size_t, std::align_val_t,`. / 这行注释说明了附近 API、不变量或算法意图：`__sized_ptr_t __size_returning_new_aligned(size_t, std::align_val_t,`。
- **L270**: Comment documents the nearby API, invariant, or algorithmic intent: `__hot_cold_t)`. / 这行注释说明了附近 API、不变量或算法意图：`__hot_cold_t)`。
- **L271**: Defines TableGen record `size_returning_new_aligned_hot_cold`, adding one entry to the generated description database. / 定义 TableGen 记录 `size_returning_new_aligned_hot_cold`，向生成用描述数据库中加入一个条目。
- **L272**: Continues the surrounding declaration or implementation logic. / 继续展开周围的声明或实现逻辑。
- **L273**: Provides a declaration or standalone statement that contributes to the surrounding API or implementation. / 给出一条声明或独立语句，为周围的 API 或实现提供支撑。
- **L274**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L275**: Comment documents the nearby API, invariant, or algorithmic intent: `double __acos_finite(double x);`. / 这行注释说明了附近 API、不变量或算法意图：`double __acos_finite(double x);`。
- **L276**: Defines TableGen record `acos_finite` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `acos_finite`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L277**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L278**: Comment documents the nearby API, invariant, or algorithmic intent: `float __acosf_finite(float x);`. / 这行注释说明了附近 API、不变量或算法意图：`float __acosf_finite(float x);`。
- **L279**: Defines TableGen record `acosf_finite` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `acosf_finite`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L280**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。

### Lines 281-308

```tablegen
/// double __acosh_finite(double x);
def acosh_finite : TargetLibCall<"__acosh_finite", Dbl, [Dbl]>;

/// float __acoshf_finite(float x);
def acoshf_finite : TargetLibCall<"__acoshf_finite", Flt, [Flt]>;

/// long double __acoshl_finite(long double x);
def acoshl_finite : TargetLibCall<"__acoshl_finite", LDbl, [LDbl]>;

/// long double __acosl_finite(long double x);
def acosl_finite : TargetLibCall<"__acosl_finite", LDbl, [LDbl]>;

/// double __asin_finite(double x);
def asin_finite : TargetLibCall<"__asin_finite", Dbl, [Dbl]>;

/// float __asinf_finite(float x);
def asinf_finite : TargetLibCall<"__asinf_finite", Flt, [Flt]>;

/// long double __asinl_finite(long double x);
def asinl_finite : TargetLibCall<"__asinl_finite", LDbl, [LDbl]>;

/// double atan2_finite(double y, double x);
def atan2_finite : TargetLibCall<"__atan2_finite", Dbl, [Dbl, Dbl]>;

/// float atan2f_finite(float y, float x);
def atan2f_finite : TargetLibCall<"__atan2f_finite", Flt, [Flt, Flt]>;

/// long double atan2l_finite(long double y, long double x);
```

- **L281**: Comment documents the nearby API, invariant, or algorithmic intent: `double __acosh_finite(double x);`. / 这行注释说明了附近 API、不变量或算法意图：`double __acosh_finite(double x);`。
- **L282**: Defines TableGen record `acosh_finite` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `acosh_finite`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L283**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L284**: Comment documents the nearby API, invariant, or algorithmic intent: `float __acoshf_finite(float x);`. / 这行注释说明了附近 API、不变量或算法意图：`float __acoshf_finite(float x);`。
- **L285**: Defines TableGen record `acoshf_finite` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `acoshf_finite`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L286**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L287**: Comment documents the nearby API, invariant, or algorithmic intent: `long double __acoshl_finite(long double x);`. / 这行注释说明了附近 API、不变量或算法意图：`long double __acoshl_finite(long double x);`。
- **L288**: Defines TableGen record `acoshl_finite` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `acoshl_finite`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L289**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L290**: Comment documents the nearby API, invariant, or algorithmic intent: `long double __acosl_finite(long double x);`. / 这行注释说明了附近 API、不变量或算法意图：`long double __acosl_finite(long double x);`。
- **L291**: Defines TableGen record `acosl_finite` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `acosl_finite`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L292**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L293**: Comment documents the nearby API, invariant, or algorithmic intent: `double __asin_finite(double x);`. / 这行注释说明了附近 API、不变量或算法意图：`double __asin_finite(double x);`。
- **L294**: Defines TableGen record `asin_finite` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `asin_finite`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L295**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L296**: Comment documents the nearby API, invariant, or algorithmic intent: `float __asinf_finite(float x);`. / 这行注释说明了附近 API、不变量或算法意图：`float __asinf_finite(float x);`。
- **L297**: Defines TableGen record `asinf_finite` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `asinf_finite`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L298**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L299**: Comment documents the nearby API, invariant, or algorithmic intent: `long double __asinl_finite(long double x);`. / 这行注释说明了附近 API、不变量或算法意图：`long double __asinl_finite(long double x);`。
- **L300**: Defines TableGen record `asinl_finite` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `asinl_finite`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L301**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L302**: Comment documents the nearby API, invariant, or algorithmic intent: `double atan2_finite(double y, double x);`. / 这行注释说明了附近 API、不变量或算法意图：`double atan2_finite(double y, double x);`。
- **L303**: Defines TableGen record `atan2_finite` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `atan2_finite`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L304**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L305**: Comment documents the nearby API, invariant, or algorithmic intent: `float atan2f_finite(float y, float x);`. / 这行注释说明了附近 API、不变量或算法意图：`float atan2f_finite(float y, float x);`。
- **L306**: Defines TableGen record `atan2f_finite` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `atan2f_finite`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L307**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L308**: Comment documents the nearby API, invariant, or algorithmic intent: `long double atan2l_finite(long double y, long double x);`. / 这行注释说明了附近 API、不变量或算法意图：`long double atan2l_finite(long double y, long double x);`。

### Lines 309-336

```tablegen
def atan2l_finite : TargetLibCall<"__atan2l_finite", LDbl, [LDbl, LDbl]>;

/// double __atanh_finite(double x);
def atanh_finite : TargetLibCall<"__atanh_finite", Dbl, [Dbl]>;

/// float __atanhf_finite(float x);
def atanhf_finite : TargetLibCall<"__atanhf_finite", Flt, [Flt]>;

/// long double __atanhl_finite(long double x);
def atanhl_finite : TargetLibCall<"__atanhl_finite", LDbl, [LDbl]>;

/// void __atomic_load(size_t size, void *mptr, void *vptr, int smodel);
def atomic_load : TargetLibCall<"__atomic_load", Void, [SizeT, Ptr, Ptr, Int]>;

/// void __atomic_store(size_t size, void *mptr, void *vptr, int smodel);
def atomic_store
    : TargetLibCall<"__atomic_store", Void, [SizeT, Ptr, Ptr, Int]>;

/// double __cosh_finite(double x);
def cosh_finite : TargetLibCall<"__cosh_finite", Dbl, [Dbl]>;

/// float __coshf_finite(float x);
def coshf_finite : TargetLibCall<"__coshf_finite", Flt, [Flt]>;

/// long double __coshl_finite(long double x);
def coshl_finite : TargetLibCall<"__coshl_finite", LDbl, [LDbl]>;

/// double __cospi(double x);
```

- **L309**: Defines TableGen record `atan2l_finite` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `atan2l_finite`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L310**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L311**: Comment documents the nearby API, invariant, or algorithmic intent: `double __atanh_finite(double x);`. / 这行注释说明了附近 API、不变量或算法意图：`double __atanh_finite(double x);`。
- **L312**: Defines TableGen record `atanh_finite` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `atanh_finite`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L313**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L314**: Comment documents the nearby API, invariant, or algorithmic intent: `float __atanhf_finite(float x);`. / 这行注释说明了附近 API、不变量或算法意图：`float __atanhf_finite(float x);`。
- **L315**: Defines TableGen record `atanhf_finite` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `atanhf_finite`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L316**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L317**: Comment documents the nearby API, invariant, or algorithmic intent: `long double __atanhl_finite(long double x);`. / 这行注释说明了附近 API、不变量或算法意图：`long double __atanhl_finite(long double x);`。
- **L318**: Defines TableGen record `atanhl_finite` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `atanhl_finite`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L319**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L320**: Comment documents the nearby API, invariant, or algorithmic intent: `void __atomic_load(size_t size, void *mptr, void *vptr, int smodel);`. / 这行注释说明了附近 API、不变量或算法意图：`void __atomic_load(size_t size, void *mptr, void *vptr, int smodel);`。
- **L321**: Defines TableGen record `atomic_load` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `atomic_load`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L322**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L323**: Comment documents the nearby API, invariant, or algorithmic intent: `void __atomic_store(size_t size, void *mptr, void *vptr, int smodel);`. / 这行注释说明了附近 API、不变量或算法意图：`void __atomic_store(size_t size, void *mptr, void *vptr, int smodel);`。
- **L324**: Defines TableGen record `atomic_store`, adding one entry to the generated description database. / 定义 TableGen 记录 `atomic_store`，向生成用描述数据库中加入一个条目。
- **L325**: Provides a declaration or standalone statement that contributes to the surrounding API or implementation. / 给出一条声明或独立语句，为周围的 API 或实现提供支撑。
- **L326**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L327**: Comment documents the nearby API, invariant, or algorithmic intent: `double __cosh_finite(double x);`. / 这行注释说明了附近 API、不变量或算法意图：`double __cosh_finite(double x);`。
- **L328**: Defines TableGen record `cosh_finite` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `cosh_finite`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L329**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L330**: Comment documents the nearby API, invariant, or algorithmic intent: `float __coshf_finite(float x);`. / 这行注释说明了附近 API、不变量或算法意图：`float __coshf_finite(float x);`。
- **L331**: Defines TableGen record `coshf_finite` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `coshf_finite`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L332**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L333**: Comment documents the nearby API, invariant, or algorithmic intent: `long double __coshl_finite(long double x);`. / 这行注释说明了附近 API、不变量或算法意图：`long double __coshl_finite(long double x);`。
- **L334**: Defines TableGen record `coshl_finite` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `coshl_finite`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L335**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L336**: Comment documents the nearby API, invariant, or algorithmic intent: `double __cospi(double x);`. / 这行注释说明了附近 API、不变量或算法意图：`double __cospi(double x);`。

### Lines 337-364

```tablegen
def cospi : TargetLibCall<"__cospi", Dbl, [Dbl]>;

/// float __cospif(float x);
def cospif : TargetLibCall<"__cospif", Flt, [Flt]>;

/// int __cxa_atexit(void (*f)(void *), void *p, void *d);
def cxa_atexit : TargetLibCall<"__cxa_atexit", Int, [Ptr, Ptr, Ptr]>;

/// int atexit(void (*f)(void));
def atexit : TargetLibCall<"atexit", Int, [Ptr]>;

/// void abort(void)
def abort : TargetLibCall<"abort", Void, []>;

/// void exit(int)
def exit : TargetLibCall<"exit", Void, [Int]>;

/// void _Exit(int)
def Exit : TargetLibCall<"_Exit", Void, [Int]>;

/// void std::terminate();
def terminate : TargetLibCall<"_ZSt9terminatev", Void, []>;

/// void __cxa_throw(void *, void *, void (*)(void *));
def cxa_throw : TargetLibCall<"__cxa_throw", Void, [Ptr, Ptr, Ptr]>;

/// void __cxa_guard_abort(guard_t *guard);
/// guard_t is int64_t in Itanium ABI or int32_t on ARM eabi.
```

- **L337**: Defines TableGen record `cospi` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `cospi`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L338**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L339**: Comment documents the nearby API, invariant, or algorithmic intent: `float __cospif(float x);`. / 这行注释说明了附近 API、不变量或算法意图：`float __cospif(float x);`。
- **L340**: Defines TableGen record `cospif` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `cospif`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L341**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L342**: Comment documents the nearby API, invariant, or algorithmic intent: `int __cxa_atexit(void (*f)(void *), void *p, void *d);`. / 这行注释说明了附近 API、不变量或算法意图：`int __cxa_atexit(void (*f)(void *), void *p, void *d);`。
- **L343**: Defines TableGen record `cxa_atexit` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `cxa_atexit`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L344**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L345**: Comment documents the nearby API, invariant, or algorithmic intent: `int atexit(void (*f)(void));`. / 这行注释说明了附近 API、不变量或算法意图：`int atexit(void (*f)(void));`。
- **L346**: Defines TableGen record `atexit` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `atexit`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L347**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L348**: Comment documents the nearby API, invariant, or algorithmic intent: `void abort(void)`. / 这行注释说明了附近 API、不变量或算法意图：`void abort(void)`。
- **L349**: Defines TableGen record `abort` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `abort`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L350**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L351**: Comment documents the nearby API, invariant, or algorithmic intent: `void exit(int)`. / 这行注释说明了附近 API、不变量或算法意图：`void exit(int)`。
- **L352**: Defines TableGen record `exit` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `exit`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L353**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L354**: Comment documents the nearby API, invariant, or algorithmic intent: `void _Exit(int)`. / 这行注释说明了附近 API、不变量或算法意图：`void _Exit(int)`。
- **L355**: Defines TableGen record `Exit` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `Exit`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L356**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L357**: Comment documents the nearby API, invariant, or algorithmic intent: `void std::terminate();`. / 这行注释说明了附近 API、不变量或算法意图：`void std::terminate();`。
- **L358**: Defines TableGen record `terminate` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `terminate`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L359**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L360**: Comment documents the nearby API, invariant, or algorithmic intent: `void __cxa_throw(void *, void *, void (*)(void *));`. / 这行注释说明了附近 API、不变量或算法意图：`void __cxa_throw(void *, void *, void (*)(void *));`。
- **L361**: Defines TableGen record `cxa_throw` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `cxa_throw`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L362**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L363**: Comment documents the nearby API, invariant, or algorithmic intent: `void __cxa_guard_abort(guard_t *guard);`. / 这行注释说明了附近 API、不变量或算法意图：`void __cxa_guard_abort(guard_t *guard);`。
- **L364**: Comment documents the nearby API, invariant, or algorithmic intent: `guard_t is int64_t in Itanium ABI or int32_t on ARM eabi.`. / 这行注释说明了附近 API、不变量或算法意图：`guard_t is int64_t in Itanium ABI or int32_t on ARM eabi.`。

### Lines 365-392

```tablegen
def cxa_guard_abort : TargetLibCall<"__cxa_guard_abort", Void, [Ptr]>;

/// int __cxa_guard_acquire(guard_t *guard);
def cxa_guard_acquire : TargetLibCall<"__cxa_guard_acquire", Int, [Ptr]>;

/// void __cxa_guard_release(guard_t *guard);
def cxa_guard_release : TargetLibCall<"__cxa_guard_release", Void, [Ptr]>;

/// double __exp10_finite(double x);
def exp10_finite : TargetLibCall<"__exp10_finite", Dbl, [Dbl]>;

/// float __exp10f_finite(float x);
def exp10f_finite : TargetLibCall<"__exp10f_finite", Flt, [Flt]>;

/// long double __exp10l_finite(long double x);
def exp10l_finite : TargetLibCall<"__exp10l_finite", LDbl, [LDbl]>;

/// double __exp2_finite(double x);
def exp2_finite : TargetLibCall<"__exp2_finite", Dbl, [Dbl]>;

/// float __exp2f_finite(float x);
def exp2f_finite : TargetLibCall<"__exp2f_finite", Flt, [Flt]>;

/// long double __exp2l_finite(long double x);
def exp2l_finite : TargetLibCall<"__exp2l_finite", LDbl, [LDbl]>;

/// double __exp_finite(double x);
def exp_finite : TargetLibCall<"__exp_finite", Dbl, [Dbl]>;
```

- **L365**: Defines TableGen record `cxa_guard_abort` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `cxa_guard_abort`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L366**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L367**: Comment documents the nearby API, invariant, or algorithmic intent: `int __cxa_guard_acquire(guard_t *guard);`. / 这行注释说明了附近 API、不变量或算法意图：`int __cxa_guard_acquire(guard_t *guard);`。
- **L368**: Defines TableGen record `cxa_guard_acquire` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `cxa_guard_acquire`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L369**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L370**: Comment documents the nearby API, invariant, or algorithmic intent: `void __cxa_guard_release(guard_t *guard);`. / 这行注释说明了附近 API、不变量或算法意图：`void __cxa_guard_release(guard_t *guard);`。
- **L371**: Defines TableGen record `cxa_guard_release` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `cxa_guard_release`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L372**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L373**: Comment documents the nearby API, invariant, or algorithmic intent: `double __exp10_finite(double x);`. / 这行注释说明了附近 API、不变量或算法意图：`double __exp10_finite(double x);`。
- **L374**: Defines TableGen record `exp10_finite` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `exp10_finite`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L375**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L376**: Comment documents the nearby API, invariant, or algorithmic intent: `float __exp10f_finite(float x);`. / 这行注释说明了附近 API、不变量或算法意图：`float __exp10f_finite(float x);`。
- **L377**: Defines TableGen record `exp10f_finite` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `exp10f_finite`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L378**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L379**: Comment documents the nearby API, invariant, or algorithmic intent: `long double __exp10l_finite(long double x);`. / 这行注释说明了附近 API、不变量或算法意图：`long double __exp10l_finite(long double x);`。
- **L380**: Defines TableGen record `exp10l_finite` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `exp10l_finite`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L381**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L382**: Comment documents the nearby API, invariant, or algorithmic intent: `double __exp2_finite(double x);`. / 这行注释说明了附近 API、不变量或算法意图：`double __exp2_finite(double x);`。
- **L383**: Defines TableGen record `exp2_finite` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `exp2_finite`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L384**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L385**: Comment documents the nearby API, invariant, or algorithmic intent: `float __exp2f_finite(float x);`. / 这行注释说明了附近 API、不变量或算法意图：`float __exp2f_finite(float x);`。
- **L386**: Defines TableGen record `exp2f_finite` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `exp2f_finite`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L387**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L388**: Comment documents the nearby API, invariant, or algorithmic intent: `long double __exp2l_finite(long double x);`. / 这行注释说明了附近 API、不变量或算法意图：`long double __exp2l_finite(long double x);`。
- **L389**: Defines TableGen record `exp2l_finite` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `exp2l_finite`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L390**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L391**: Comment documents the nearby API, invariant, or algorithmic intent: `double __exp_finite(double x);`. / 这行注释说明了附近 API、不变量或算法意图：`double __exp_finite(double x);`。
- **L392**: Defines TableGen record `exp_finite` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `exp_finite`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。

### Lines 393-420

```tablegen

/// float __expf_finite(float x);
def expf_finite : TargetLibCall<"__expf_finite", Flt, [Flt]>;

/// long double __expl_finite(long double x);
def expl_finite : TargetLibCall<"__expl_finite", LDbl, [LDbl]>;

/// int __isoc99_scanf (const char *format, ...)
def dunder_isoc99_scanf : TargetLibCall<"__isoc99_scanf", Int, [Ptr, Ellip]>;

/// int __isoc99_sscanf(const char *s, const char *format, ...)
def dunder_isoc99_sscanf
    : TargetLibCall<"__isoc99_sscanf", Int, [Ptr, Ptr, Ellip]>;

/// double __log10_finite(double x);
def log10_finite : TargetLibCall<"__log10_finite", Dbl, [Dbl]>;

/// float __log10f_finite(float x);
def log10f_finite : TargetLibCall<"__log10f_finite", Flt, [Flt]>;

/// long double __log10l_finite(long double x);
def log10l_finite : TargetLibCall<"__log10l_finite", LDbl, [LDbl]>;

/// double __log2_finite(double x);
def log2_finite : TargetLibCall<"__log2_finite", Dbl, [Dbl]>;

/// float __log2f_finite(float x);
def log2f_finite : TargetLibCall<"__log2f_finite", Flt, [Flt]>;
```

- **L393**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L394**: Comment documents the nearby API, invariant, or algorithmic intent: `float __expf_finite(float x);`. / 这行注释说明了附近 API、不变量或算法意图：`float __expf_finite(float x);`。
- **L395**: Defines TableGen record `expf_finite` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `expf_finite`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L396**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L397**: Comment documents the nearby API, invariant, or algorithmic intent: `long double __expl_finite(long double x);`. / 这行注释说明了附近 API、不变量或算法意图：`long double __expl_finite(long double x);`。
- **L398**: Defines TableGen record `expl_finite` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `expl_finite`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L399**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L400**: Comment documents the nearby API, invariant, or algorithmic intent: `int __isoc99_scanf (const char *format, ...)`. / 这行注释说明了附近 API、不变量或算法意图：`int __isoc99_scanf (const char *format, ...)`。
- **L401**: Defines TableGen record `dunder_isoc99_scanf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `dunder_isoc99_scanf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L402**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L403**: Comment documents the nearby API, invariant, or algorithmic intent: `int __isoc99_sscanf(const char *s, const char *format, ...)`. / 这行注释说明了附近 API、不变量或算法意图：`int __isoc99_sscanf(const char *s, const char *format, ...)`。
- **L404**: Defines TableGen record `dunder_isoc99_sscanf`, adding one entry to the generated description database. / 定义 TableGen 记录 `dunder_isoc99_sscanf`，向生成用描述数据库中加入一个条目。
- **L405**: Provides a declaration or standalone statement that contributes to the surrounding API or implementation. / 给出一条声明或独立语句，为周围的 API 或实现提供支撑。
- **L406**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L407**: Comment documents the nearby API, invariant, or algorithmic intent: `double __log10_finite(double x);`. / 这行注释说明了附近 API、不变量或算法意图：`double __log10_finite(double x);`。
- **L408**: Defines TableGen record `log10_finite` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `log10_finite`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L409**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L410**: Comment documents the nearby API, invariant, or algorithmic intent: `float __log10f_finite(float x);`. / 这行注释说明了附近 API、不变量或算法意图：`float __log10f_finite(float x);`。
- **L411**: Defines TableGen record `log10f_finite` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `log10f_finite`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L412**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L413**: Comment documents the nearby API, invariant, or algorithmic intent: `long double __log10l_finite(long double x);`. / 这行注释说明了附近 API、不变量或算法意图：`long double __log10l_finite(long double x);`。
- **L414**: Defines TableGen record `log10l_finite` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `log10l_finite`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L415**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L416**: Comment documents the nearby API, invariant, or algorithmic intent: `double __log2_finite(double x);`. / 这行注释说明了附近 API、不变量或算法意图：`double __log2_finite(double x);`。
- **L417**: Defines TableGen record `log2_finite` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `log2_finite`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L418**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L419**: Comment documents the nearby API, invariant, or algorithmic intent: `float __log2f_finite(float x);`. / 这行注释说明了附近 API、不变量或算法意图：`float __log2f_finite(float x);`。
- **L420**: Defines TableGen record `log2f_finite` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `log2f_finite`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。

### Lines 421-448

```tablegen

/// long double __log2l_finite(long double x);
def log2l_finite : TargetLibCall<"__log2l_finite", LDbl, [LDbl]>;

/// double __log_finite(double x);
def log_finite : TargetLibCall<"__log_finite", Dbl, [Dbl]>;

/// float __logf_finite(float x);
def logf_finite : TargetLibCall<"__logf_finite", Flt, [Flt]>;

/// long double __logl_finite(long double x);
def logl_finite : TargetLibCall<"__logl_finite", LDbl, [LDbl]>;

/// void *__memccpy_chk(void *dst, const void *src, int c, size_t n,
/// size_t dstsize)
def memccpy_chk
    : TargetLibCall<"__memccpy_chk", Ptr, [Ptr, Ptr, Int, SizeT, SizeT]>;

/// void *__memcpy_chk(void *s1, const void *s2, size_t n, size_t s1size);
def memcpy_chk : TargetLibCall<"__memcpy_chk", Ptr, [Ptr, Ptr, SizeT, SizeT]>;

/// void *__memmove_chk(void *s1, const void *s2, size_t n, size_t s1size);
def memmove_chk : TargetLibCall<"__memmove_chk", Ptr, [Ptr, Ptr, SizeT, SizeT]>;

/// void *__mempcpy_chk(void *s1, const void *s2, size_t n, size_t s1size);
def mempcpy_chk : TargetLibCall<"__mempcpy_chk", Ptr, [Ptr, Ptr, SizeT, SizeT]>;

/// void *__memset_chk(void *s, int v, size_t n, size_t s1size);
```

- **L421**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L422**: Comment documents the nearby API, invariant, or algorithmic intent: `long double __log2l_finite(long double x);`. / 这行注释说明了附近 API、不变量或算法意图：`long double __log2l_finite(long double x);`。
- **L423**: Defines TableGen record `log2l_finite` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `log2l_finite`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L424**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L425**: Comment documents the nearby API, invariant, or algorithmic intent: `double __log_finite(double x);`. / 这行注释说明了附近 API、不变量或算法意图：`double __log_finite(double x);`。
- **L426**: Defines TableGen record `log_finite` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `log_finite`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L427**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L428**: Comment documents the nearby API, invariant, or algorithmic intent: `float __logf_finite(float x);`. / 这行注释说明了附近 API、不变量或算法意图：`float __logf_finite(float x);`。
- **L429**: Defines TableGen record `logf_finite` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `logf_finite`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L430**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L431**: Comment documents the nearby API, invariant, or algorithmic intent: `long double __logl_finite(long double x);`. / 这行注释说明了附近 API、不变量或算法意图：`long double __logl_finite(long double x);`。
- **L432**: Defines TableGen record `logl_finite` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `logl_finite`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L433**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L434**: Comment documents the nearby API, invariant, or algorithmic intent: `void *__memccpy_chk(void *dst, const void *src, int c, size_t n,`. / 这行注释说明了附近 API、不变量或算法意图：`void *__memccpy_chk(void *dst, const void *src, int c, size_t n,`。
- **L435**: Comment documents the nearby API, invariant, or algorithmic intent: `size_t dstsize)`. / 这行注释说明了附近 API、不变量或算法意图：`size_t dstsize)`。
- **L436**: Defines TableGen record `memccpy_chk`, adding one entry to the generated description database. / 定义 TableGen 记录 `memccpy_chk`，向生成用描述数据库中加入一个条目。
- **L437**: Provides a declaration or standalone statement that contributes to the surrounding API or implementation. / 给出一条声明或独立语句，为周围的 API 或实现提供支撑。
- **L438**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L439**: Comment documents the nearby API, invariant, or algorithmic intent: `void *__memcpy_chk(void *s1, const void *s2, size_t n, size_t s1size);`. / 这行注释说明了附近 API、不变量或算法意图：`void *__memcpy_chk(void *s1, const void *s2, size_t n, size_t s1size);`。
- **L440**: Defines TableGen record `memcpy_chk` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `memcpy_chk`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L441**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L442**: Comment documents the nearby API, invariant, or algorithmic intent: `void *__memmove_chk(void *s1, const void *s2, size_t n, size_t s1size);`. / 这行注释说明了附近 API、不变量或算法意图：`void *__memmove_chk(void *s1, const void *s2, size_t n, size_t s1size);`。
- **L443**: Defines TableGen record `memmove_chk` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `memmove_chk`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L444**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L445**: Comment documents the nearby API, invariant, or algorithmic intent: `void *__mempcpy_chk(void *s1, const void *s2, size_t n, size_t s1size);`. / 这行注释说明了附近 API、不变量或算法意图：`void *__mempcpy_chk(void *s1, const void *s2, size_t n, size_t s1size);`。
- **L446**: Defines TableGen record `mempcpy_chk` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `mempcpy_chk`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L447**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L448**: Comment documents the nearby API, invariant, or algorithmic intent: `void *__memset_chk(void *s, int v, size_t n, size_t s1size);`. / 这行注释说明了附近 API、不变量或算法意图：`void *__memset_chk(void *s, int v, size_t n, size_t s1size);`。

### Lines 449-476

```tablegen
def memset_chk : TargetLibCall<"__memset_chk", Ptr, [Ptr, Int, SizeT, SizeT]>;

// int __nvvm_reflect(const char *)
def nvvm_reflect : TargetLibCall<"__nvvm_reflect", Int, [Ptr]>;

/// double __pow_finite(double x, double y);
def pow_finite : TargetLibCall<"__pow_finite", Dbl, [Dbl, Dbl]>;

/// float _powf_finite(float x, float y);
def powf_finite : TargetLibCall<"__powf_finite", Flt, [Flt, Flt]>;

/// long double __powl_finite(long double x, long double y);
def powl_finite : TargetLibCall<"__powl_finite", LDbl, [LDbl, LDbl]>;

/// double __sincospi_stret(double x);
def sincospi_stret
    : TargetLibCall<"__sincospi_stret", ? /* Checked manually. */>;

/// float __sincospif_stret(float x);
def sincospif_stret
    : TargetLibCall<"__sincospif_stret", ? /* Checked manually. */>;

/// double __sinh_finite(double x);
def sinh_finite : TargetLibCall<"__sinh_finite", Dbl, [Dbl]>;

/// float _sinhf_finite(float x);
def sinhf_finite : TargetLibCall<"__sinhf_finite", Flt, [Flt]>;

```

- **L449**: Defines TableGen record `memset_chk` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `memset_chk`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L450**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L451**: Comment documents the nearby API, invariant, or algorithmic intent: `int __nvvm_reflect(const char *)`. / 这行注释说明了附近 API、不变量或算法意图：`int __nvvm_reflect(const char *)`。
- **L452**: Defines TableGen record `nvvm_reflect` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `nvvm_reflect`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L453**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L454**: Comment documents the nearby API, invariant, or algorithmic intent: `double __pow_finite(double x, double y);`. / 这行注释说明了附近 API、不变量或算法意图：`double __pow_finite(double x, double y);`。
- **L455**: Defines TableGen record `pow_finite` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `pow_finite`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L456**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L457**: Comment documents the nearby API, invariant, or algorithmic intent: `float _powf_finite(float x, float y);`. / 这行注释说明了附近 API、不变量或算法意图：`float _powf_finite(float x, float y);`。
- **L458**: Defines TableGen record `powf_finite` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `powf_finite`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L459**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L460**: Comment documents the nearby API, invariant, or algorithmic intent: `long double __powl_finite(long double x, long double y);`. / 这行注释说明了附近 API、不变量或算法意图：`long double __powl_finite(long double x, long double y);`。
- **L461**: Defines TableGen record `powl_finite` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `powl_finite`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L462**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L463**: Comment documents the nearby API, invariant, or algorithmic intent: `double __sincospi_stret(double x);`. / 这行注释说明了附近 API、不变量或算法意图：`double __sincospi_stret(double x);`。
- **L464**: Defines TableGen record `sincospi_stret`, adding one entry to the generated description database. / 定义 TableGen 记录 `sincospi_stret`，向生成用描述数据库中加入一个条目。
- **L465**: Provides a declaration or standalone statement that contributes to the surrounding API or implementation. / 给出一条声明或独立语句，为周围的 API 或实现提供支撑。
- **L466**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L467**: Comment documents the nearby API, invariant, or algorithmic intent: `float __sincospif_stret(float x);`. / 这行注释说明了附近 API、不变量或算法意图：`float __sincospif_stret(float x);`。
- **L468**: Defines TableGen record `sincospif_stret`, adding one entry to the generated description database. / 定义 TableGen 记录 `sincospif_stret`，向生成用描述数据库中加入一个条目。
- **L469**: Provides a declaration or standalone statement that contributes to the surrounding API or implementation. / 给出一条声明或独立语句，为周围的 API 或实现提供支撑。
- **L470**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L471**: Comment documents the nearby API, invariant, or algorithmic intent: `double __sinh_finite(double x);`. / 这行注释说明了附近 API、不变量或算法意图：`double __sinh_finite(double x);`。
- **L472**: Defines TableGen record `sinh_finite` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `sinh_finite`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L473**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L474**: Comment documents the nearby API, invariant, or algorithmic intent: `float _sinhf_finite(float x);`. / 这行注释说明了附近 API、不变量或算法意图：`float _sinhf_finite(float x);`。
- **L475**: Defines TableGen record `sinhf_finite` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `sinhf_finite`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L476**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。

### Lines 477-504

```tablegen
/// long double __sinhl_finite(long double x);
def sinhl_finite : TargetLibCall<"__sinhl_finite", LDbl, [LDbl]>;

/// double __sinpi(double x);
def sinpi : TargetLibCall<"__sinpi", Dbl, [Dbl]>;

/// float __sinpif(float x);
def sinpif : TargetLibCall<"__sinpif", Flt, [Flt]>;

/// int __small_fprintf(FILE *stream, const char *format, ...);
def small_fprintf : TargetLibCall<"__small_fprintf", Int, [Ptr, Ptr, Ellip]>;

/// int __small_printf(const char *format, ...);
def small_printf : TargetLibCall<"__small_printf", Int, [Ptr, Ellip]>;

/// int __small_sprintf(char *str, const char *format, ...);
def small_sprintf : TargetLibCall<"__small_sprintf", Int, [Ptr, Ptr, Ellip]>;

/// int __snprintf_chk(char *s, size_t n, int flags, size_t slen,
/// const char *format, ...);
def snprintf_chk : TargetLibCall<"__snprintf_chk",
                                 Int, [Ptr, SizeT, Int, SizeT, Ptr, Ellip]>;

/// int __sprintf_chk(char *str, int flags, size_t str_len,
/// const char *format, ...);
def sprintf_chk
    : TargetLibCall<"__sprintf_chk", Int, [Ptr, Int, SizeT, Ptr, Ellip]>;

```

- **L477**: Comment documents the nearby API, invariant, or algorithmic intent: `long double __sinhl_finite(long double x);`. / 这行注释说明了附近 API、不变量或算法意图：`long double __sinhl_finite(long double x);`。
- **L478**: Defines TableGen record `sinhl_finite` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `sinhl_finite`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L479**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L480**: Comment documents the nearby API, invariant, or algorithmic intent: `double __sinpi(double x);`. / 这行注释说明了附近 API、不变量或算法意图：`double __sinpi(double x);`。
- **L481**: Defines TableGen record `sinpi` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `sinpi`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L482**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L483**: Comment documents the nearby API, invariant, or algorithmic intent: `float __sinpif(float x);`. / 这行注释说明了附近 API、不变量或算法意图：`float __sinpif(float x);`。
- **L484**: Defines TableGen record `sinpif` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `sinpif`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L485**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L486**: Comment documents the nearby API, invariant, or algorithmic intent: `int __small_fprintf(FILE *stream, const char *format, ...);`. / 这行注释说明了附近 API、不变量或算法意图：`int __small_fprintf(FILE *stream, const char *format, ...);`。
- **L487**: Defines TableGen record `small_fprintf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `small_fprintf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L488**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L489**: Comment documents the nearby API, invariant, or algorithmic intent: `int __small_printf(const char *format, ...);`. / 这行注释说明了附近 API、不变量或算法意图：`int __small_printf(const char *format, ...);`。
- **L490**: Defines TableGen record `small_printf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `small_printf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L491**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L492**: Comment documents the nearby API, invariant, or algorithmic intent: `int __small_sprintf(char *str, const char *format, ...);`. / 这行注释说明了附近 API、不变量或算法意图：`int __small_sprintf(char *str, const char *format, ...);`。
- **L493**: Defines TableGen record `small_sprintf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `small_sprintf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L494**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L495**: Comment documents the nearby API, invariant, or algorithmic intent: `int __snprintf_chk(char *s, size_t n, int flags, size_t slen,`. / 这行注释说明了附近 API、不变量或算法意图：`int __snprintf_chk(char *s, size_t n, int flags, size_t slen,`。
- **L496**: Comment documents the nearby API, invariant, or algorithmic intent: `const char *format, ...);`. / 这行注释说明了附近 API、不变量或算法意图：`const char *format, ...);`。
- **L497**: Defines TableGen record `snprintf_chk` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `snprintf_chk`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L498**: Provides a declaration or standalone statement that contributes to the surrounding API or implementation. / 给出一条声明或独立语句，为周围的 API 或实现提供支撑。
- **L499**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L500**: Comment documents the nearby API, invariant, or algorithmic intent: `int __sprintf_chk(char *str, int flags, size_t str_len,`. / 这行注释说明了附近 API、不变量或算法意图：`int __sprintf_chk(char *str, int flags, size_t str_len,`。
- **L501**: Comment documents the nearby API, invariant, or algorithmic intent: `const char *format, ...);`. / 这行注释说明了附近 API、不变量或算法意图：`const char *format, ...);`。
- **L502**: Defines TableGen record `sprintf_chk`, adding one entry to the generated description database. / 定义 TableGen 记录 `sprintf_chk`，向生成用描述数据库中加入一个条目。
- **L503**: Provides a declaration or standalone statement that contributes to the surrounding API or implementation. / 给出一条声明或独立语句，为周围的 API 或实现提供支撑。
- **L504**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。

### Lines 505-532

```tablegen
/// double __sqrt_finite(double x);
def sqrt_finite : TargetLibCall<"__sqrt_finite", Dbl, [Dbl]>;

/// float __sqrt_finite(float x);
def sqrtf_finite : TargetLibCall<"__sqrtf_finite", Flt, [Flt]>;

/// long double __sqrt_finite(long double x);
def sqrtl_finite : TargetLibCall<"__sqrtl_finite", LDbl, [LDbl]>;

/// char *__stpcpy_chk(char *s1, const char *s2, size_t s1size);
def stpcpy_chk : TargetLibCall<"__stpcpy_chk", Ptr, [Ptr, Ptr, SizeT]>;

/// char *__stpncpy_chk(char *s1, const char *s2, size_t n, size_t s1size);
def stpncpy_chk : TargetLibCall<"__stpncpy_chk", Ptr, [Ptr, Ptr, SizeT, SizeT]>;

/// char *__strcat_chk(char *s1, const char *s2, size_t s1size);
def strcat_chk : TargetLibCall<"__strcat_chk", Ptr, [Ptr, Ptr, SizeT]>;

/// char *__strcpy_chk(char *s1, const char *s2, size_t s1size);
def strcpy_chk : TargetLibCall<"__strcpy_chk", Ptr, [Ptr, Ptr, SizeT]>;

/// char * __strdup(const char *s);
def dunder_strdup : TargetLibCall<"__strdup", Ptr, [Ptr]>;

/// size_t __strlcat_chk(char *dst, const char *src, size_t size,
/// size_t dstsize);
def strlcat_chk
    : TargetLibCall<"__strlcat_chk", SizeT, [Ptr, Ptr, SizeT, SizeT]>;
```

- **L505**: Comment documents the nearby API, invariant, or algorithmic intent: `double __sqrt_finite(double x);`. / 这行注释说明了附近 API、不变量或算法意图：`double __sqrt_finite(double x);`。
- **L506**: Defines TableGen record `sqrt_finite` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `sqrt_finite`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L507**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L508**: Comment documents the nearby API, invariant, or algorithmic intent: `float __sqrt_finite(float x);`. / 这行注释说明了附近 API、不变量或算法意图：`float __sqrt_finite(float x);`。
- **L509**: Defines TableGen record `sqrtf_finite` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `sqrtf_finite`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L510**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L511**: Comment documents the nearby API, invariant, or algorithmic intent: `long double __sqrt_finite(long double x);`. / 这行注释说明了附近 API、不变量或算法意图：`long double __sqrt_finite(long double x);`。
- **L512**: Defines TableGen record `sqrtl_finite` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `sqrtl_finite`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L513**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L514**: Comment documents the nearby API, invariant, or algorithmic intent: `char *__stpcpy_chk(char *s1, const char *s2, size_t s1size);`. / 这行注释说明了附近 API、不变量或算法意图：`char *__stpcpy_chk(char *s1, const char *s2, size_t s1size);`。
- **L515**: Defines TableGen record `stpcpy_chk` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `stpcpy_chk`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L516**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L517**: Comment documents the nearby API, invariant, or algorithmic intent: `char *__stpncpy_chk(char *s1, const char *s2, size_t n, size_t s1size);`. / 这行注释说明了附近 API、不变量或算法意图：`char *__stpncpy_chk(char *s1, const char *s2, size_t n, size_t s1size);`。
- **L518**: Defines TableGen record `stpncpy_chk` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `stpncpy_chk`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L519**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L520**: Comment documents the nearby API, invariant, or algorithmic intent: `char *__strcat_chk(char *s1, const char *s2, size_t s1size);`. / 这行注释说明了附近 API、不变量或算法意图：`char *__strcat_chk(char *s1, const char *s2, size_t s1size);`。
- **L521**: Defines TableGen record `strcat_chk` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `strcat_chk`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L522**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L523**: Comment documents the nearby API, invariant, or algorithmic intent: `char *__strcpy_chk(char *s1, const char *s2, size_t s1size);`. / 这行注释说明了附近 API、不变量或算法意图：`char *__strcpy_chk(char *s1, const char *s2, size_t s1size);`。
- **L524**: Defines TableGen record `strcpy_chk` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `strcpy_chk`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L525**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L526**: Comment documents the nearby API, invariant, or algorithmic intent: `char * __strdup(const char *s);`. / 这行注释说明了附近 API、不变量或算法意图：`char * __strdup(const char *s);`。
- **L527**: Defines TableGen record `dunder_strdup` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `dunder_strdup`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L528**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L529**: Comment documents the nearby API, invariant, or algorithmic intent: `size_t __strlcat_chk(char *dst, const char *src, size_t size,`. / 这行注释说明了附近 API、不变量或算法意图：`size_t __strlcat_chk(char *dst, const char *src, size_t size,`。
- **L530**: Comment documents the nearby API, invariant, or algorithmic intent: `size_t dstsize);`. / 这行注释说明了附近 API、不变量或算法意图：`size_t dstsize);`。
- **L531**: Defines TableGen record `strlcat_chk`, adding one entry to the generated description database. / 定义 TableGen 记录 `strlcat_chk`，向生成用描述数据库中加入一个条目。
- **L532**: Provides a declaration or standalone statement that contributes to the surrounding API or implementation. / 给出一条声明或独立语句，为周围的 API 或实现提供支撑。

### Lines 533-560

```tablegen

/// size_t __strlcpy_chk(char *dst, const char *src, size_t size,
/// size_t dstsize);
def strlcpy_chk
    : TargetLibCall<"__strlcpy_chk", SizeT, [Ptr, Ptr, SizeT, SizeT]>;

/// size_t __strlen_chk(const char *s1, size_t s1size);
def strlen_chk : TargetLibCall<"__strlen_chk", SizeT, [Ptr, SizeT]>;

/// char *strncat_chk(char *s1, const char *s2, size_t n, size_t s1size);
def strncat_chk : TargetLibCall<"__strncat_chk", Ptr, [Ptr, Ptr, SizeT, SizeT]>;

/// char *__strncpy_chk(char *s1, const char *s2, size_t n, size_t s1size);
def strncpy_chk : TargetLibCall<"__strncpy_chk", Ptr, [Ptr, Ptr, SizeT, SizeT]>;

/// char *__strndup(const char *s, size_t n);
def dunder_strndup : TargetLibCall<"__strndup", Ptr, [Ptr, SizeT]>;

/// char * __strtok_r(char *s, const char *delim, char **save_ptr);
def dunder_strtok_r : TargetLibCall<"__strtok_r", Ptr, [Ptr, Ptr, Ptr]>;

/// int __vsnprintf_chk(char *s, size_t n, int flags, size_t slen,
/// const char *format, va_list ap);
def vsnprintf_chk
    : TargetLibCall<"__vsnprintf_chk", Int, [Ptr, SizeT, Int, SizeT, Ptr, Ptr]>;

/// int __vsprintf_chk(char *s, int flags, size_t slen, const char *format,
/// va_list ap);
```

- **L533**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L534**: Comment documents the nearby API, invariant, or algorithmic intent: `size_t __strlcpy_chk(char *dst, const char *src, size_t size,`. / 这行注释说明了附近 API、不变量或算法意图：`size_t __strlcpy_chk(char *dst, const char *src, size_t size,`。
- **L535**: Comment documents the nearby API, invariant, or algorithmic intent: `size_t dstsize);`. / 这行注释说明了附近 API、不变量或算法意图：`size_t dstsize);`。
- **L536**: Defines TableGen record `strlcpy_chk`, adding one entry to the generated description database. / 定义 TableGen 记录 `strlcpy_chk`，向生成用描述数据库中加入一个条目。
- **L537**: Provides a declaration or standalone statement that contributes to the surrounding API or implementation. / 给出一条声明或独立语句，为周围的 API 或实现提供支撑。
- **L538**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L539**: Comment documents the nearby API, invariant, or algorithmic intent: `size_t __strlen_chk(const char *s1, size_t s1size);`. / 这行注释说明了附近 API、不变量或算法意图：`size_t __strlen_chk(const char *s1, size_t s1size);`。
- **L540**: Defines TableGen record `strlen_chk` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `strlen_chk`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L541**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L542**: Comment documents the nearby API, invariant, or algorithmic intent: `char *strncat_chk(char *s1, const char *s2, size_t n, size_t s1size);`. / 这行注释说明了附近 API、不变量或算法意图：`char *strncat_chk(char *s1, const char *s2, size_t n, size_t s1size);`。
- **L543**: Defines TableGen record `strncat_chk` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `strncat_chk`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L544**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L545**: Comment documents the nearby API, invariant, or algorithmic intent: `char *__strncpy_chk(char *s1, const char *s2, size_t n, size_t s1size);`. / 这行注释说明了附近 API、不变量或算法意图：`char *__strncpy_chk(char *s1, const char *s2, size_t n, size_t s1size);`。
- **L546**: Defines TableGen record `strncpy_chk` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `strncpy_chk`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L547**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L548**: Comment documents the nearby API, invariant, or algorithmic intent: `char *__strndup(const char *s, size_t n);`. / 这行注释说明了附近 API、不变量或算法意图：`char *__strndup(const char *s, size_t n);`。
- **L549**: Defines TableGen record `dunder_strndup` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `dunder_strndup`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L550**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L551**: Comment documents the nearby API, invariant, or algorithmic intent: `char * __strtok_r(char *s, const char *delim, char **save_ptr);`. / 这行注释说明了附近 API、不变量或算法意图：`char * __strtok_r(char *s, const char *delim, char **save_ptr);`。
- **L552**: Defines TableGen record `dunder_strtok_r` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `dunder_strtok_r`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L553**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L554**: Comment documents the nearby API, invariant, or algorithmic intent: `int __vsnprintf_chk(char *s, size_t n, int flags, size_t slen,`. / 这行注释说明了附近 API、不变量或算法意图：`int __vsnprintf_chk(char *s, size_t n, int flags, size_t slen,`。
- **L555**: Comment documents the nearby API, invariant, or algorithmic intent: `const char *format, va_list ap);`. / 这行注释说明了附近 API、不变量或算法意图：`const char *format, va_list ap);`。
- **L556**: Defines TableGen record `vsnprintf_chk`, adding one entry to the generated description database. / 定义 TableGen 记录 `vsnprintf_chk`，向生成用描述数据库中加入一个条目。
- **L557**: Provides a declaration or standalone statement that contributes to the surrounding API or implementation. / 给出一条声明或独立语句，为周围的 API 或实现提供支撑。
- **L558**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L559**: Comment documents the nearby API, invariant, or algorithmic intent: `int __vsprintf_chk(char *s, int flags, size_t slen, const char *format,`. / 这行注释说明了附近 API、不变量或算法意图：`int __vsprintf_chk(char *s, int flags, size_t slen, const char *format,`。
- **L560**: Comment documents the nearby API, invariant, or algorithmic intent: `va_list ap);`. / 这行注释说明了附近 API、不变量或算法意图：`va_list ap);`。

### Lines 561-588

```tablegen
def vsprintf_chk
    : TargetLibCall<"__vsprintf_chk", Int, [Ptr, Int, SizeT, Ptr, Ptr]>;

/// int abs(int j);
def abs : TargetLibCall<"abs", Int, [Int]>;

/// int access(const char *path, int amode);
def access : TargetLibCall<"access", Int, [Ptr, Int]>;

/// double acos(double x);
def acos : TargetLibCall<"acos", Dbl, [Dbl]>;

/// float acosf(float x);
def acosf : TargetLibCall<"acosf", Flt, [Flt]>;

/// double acosh(double x);
def acosh : TargetLibCall<"acosh", Dbl, [Dbl]>;

/// float acoshf(float x);
def acoshf : TargetLibCall<"acoshf", Flt, [Flt]>;

/// long double acoshl(long double x);
def acoshl : TargetLibCall<"acoshl", LDbl, [LDbl]>;

/// long double acosl(long double x);
def acosl : TargetLibCall<"acosl", LDbl, [LDbl]>;

/// void *aligned_alloc(size_t alignment, size_t size);
```

- **L561**: Defines TableGen record `vsprintf_chk`, adding one entry to the generated description database. / 定义 TableGen 记录 `vsprintf_chk`，向生成用描述数据库中加入一个条目。
- **L562**: Provides a declaration or standalone statement that contributes to the surrounding API or implementation. / 给出一条声明或独立语句，为周围的 API 或实现提供支撑。
- **L563**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L564**: Comment documents the nearby API, invariant, or algorithmic intent: `int abs(int j);`. / 这行注释说明了附近 API、不变量或算法意图：`int abs(int j);`。
- **L565**: Defines TableGen record `abs` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `abs`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L566**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L567**: Comment documents the nearby API, invariant, or algorithmic intent: `int access(const char *path, int amode);`. / 这行注释说明了附近 API、不变量或算法意图：`int access(const char *path, int amode);`。
- **L568**: Defines TableGen record `access` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `access`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L569**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L570**: Comment documents the nearby API, invariant, or algorithmic intent: `double acos(double x);`. / 这行注释说明了附近 API、不变量或算法意图：`double acos(double x);`。
- **L571**: Defines TableGen record `acos` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `acos`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L572**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L573**: Comment documents the nearby API, invariant, or algorithmic intent: `float acosf(float x);`. / 这行注释说明了附近 API、不变量或算法意图：`float acosf(float x);`。
- **L574**: Defines TableGen record `acosf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `acosf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L575**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L576**: Comment documents the nearby API, invariant, or algorithmic intent: `double acosh(double x);`. / 这行注释说明了附近 API、不变量或算法意图：`double acosh(double x);`。
- **L577**: Defines TableGen record `acosh` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `acosh`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L578**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L579**: Comment documents the nearby API, invariant, or algorithmic intent: `float acoshf(float x);`. / 这行注释说明了附近 API、不变量或算法意图：`float acoshf(float x);`。
- **L580**: Defines TableGen record `acoshf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `acoshf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L581**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L582**: Comment documents the nearby API, invariant, or algorithmic intent: `long double acoshl(long double x);`. / 这行注释说明了附近 API、不变量或算法意图：`long double acoshl(long double x);`。
- **L583**: Defines TableGen record `acoshl` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `acoshl`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L584**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L585**: Comment documents the nearby API, invariant, or algorithmic intent: `long double acosl(long double x);`. / 这行注释说明了附近 API、不变量或算法意图：`long double acosl(long double x);`。
- **L586**: Defines TableGen record `acosl` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `acosl`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L587**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L588**: Comment documents the nearby API, invariant, or algorithmic intent: `void *aligned_alloc(size_t alignment, size_t size);`. / 这行注释说明了附近 API、不变量或算法意图：`void *aligned_alloc(size_t alignment, size_t size);`。

### Lines 589-616

```tablegen
def aligned_alloc : TargetLibCall<"aligned_alloc", Ptr, [SizeT, SizeT]>;

/// double asin(double x);
def asin : TargetLibCall<"asin", Dbl, [Dbl]>;

/// float asinf(float x);
def asinf : TargetLibCall<"asinf", Flt, [Flt]>;

/// double asinh(double x);
def asinh : TargetLibCall<"asinh", Dbl, [Dbl]>;

/// float asinhf(float x);
def asinhf : TargetLibCall<"asinhf", Flt, [Flt]>;

/// long double asinhl(long double x);
def asinhl : TargetLibCall<"asinhl", LDbl, [LDbl]>;

/// long double asinl(long double x);
def asinl : TargetLibCall<"asinl", LDbl, [LDbl]>;

/// double atan(double x);
def atan : TargetLibCall<"atan", Dbl, [Dbl]>;

/// double atan2(double y, double x);
def atan2 : TargetLibCall<"atan2", Dbl, [Dbl, Dbl]>;

/// float atan2f(float y, float x);
def atan2f : TargetLibCall<"atan2f", Flt, [Flt, Flt]>;
```

- **L589**: Defines TableGen record `aligned_alloc` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `aligned_alloc`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L590**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L591**: Comment documents the nearby API, invariant, or algorithmic intent: `double asin(double x);`. / 这行注释说明了附近 API、不变量或算法意图：`double asin(double x);`。
- **L592**: Defines TableGen record `asin` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `asin`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L593**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L594**: Comment documents the nearby API, invariant, or algorithmic intent: `float asinf(float x);`. / 这行注释说明了附近 API、不变量或算法意图：`float asinf(float x);`。
- **L595**: Defines TableGen record `asinf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `asinf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L596**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L597**: Comment documents the nearby API, invariant, or algorithmic intent: `double asinh(double x);`. / 这行注释说明了附近 API、不变量或算法意图：`double asinh(double x);`。
- **L598**: Defines TableGen record `asinh` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `asinh`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L599**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L600**: Comment documents the nearby API, invariant, or algorithmic intent: `float asinhf(float x);`. / 这行注释说明了附近 API、不变量或算法意图：`float asinhf(float x);`。
- **L601**: Defines TableGen record `asinhf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `asinhf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L602**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L603**: Comment documents the nearby API, invariant, or algorithmic intent: `long double asinhl(long double x);`. / 这行注释说明了附近 API、不变量或算法意图：`long double asinhl(long double x);`。
- **L604**: Defines TableGen record `asinhl` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `asinhl`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L605**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L606**: Comment documents the nearby API, invariant, or algorithmic intent: `long double asinl(long double x);`. / 这行注释说明了附近 API、不变量或算法意图：`long double asinl(long double x);`。
- **L607**: Defines TableGen record `asinl` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `asinl`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L608**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L609**: Comment documents the nearby API, invariant, or algorithmic intent: `double atan(double x);`. / 这行注释说明了附近 API、不变量或算法意图：`double atan(double x);`。
- **L610**: Defines TableGen record `atan` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `atan`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L611**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L612**: Comment documents the nearby API, invariant, or algorithmic intent: `double atan2(double y, double x);`. / 这行注释说明了附近 API、不变量或算法意图：`double atan2(double y, double x);`。
- **L613**: Defines TableGen record `atan2` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `atan2`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L614**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L615**: Comment documents the nearby API, invariant, or algorithmic intent: `float atan2f(float y, float x);`. / 这行注释说明了附近 API、不变量或算法意图：`float atan2f(float y, float x);`。
- **L616**: Defines TableGen record `atan2f` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `atan2f`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。

### Lines 617-644

```tablegen

/// long double atan2l(long double y, long double x);
def atan2l : TargetLibCall<"atan2l", LDbl, [LDbl, LDbl]>;

/// float atanf(float x);
def atanf : TargetLibCall<"atanf", Flt, [Flt]>;

/// double atanh(double x);
def atanh : TargetLibCall<"atanh", Dbl, [Dbl]>;

/// float atanhf(float x);
def atanhf : TargetLibCall<"atanhf", Flt, [Flt]>;

/// long double atanhl(long double x);
def atanhl : TargetLibCall<"atanhl", LDbl, [LDbl]>;

/// long double atanl(long double x);
def atanl : TargetLibCall<"atanl", LDbl, [LDbl]>;

/// double atof(const char *str);
def atof : TargetLibCall<"atof", Dbl, [Ptr]>;

/// int atoi(const char *str);
def atoi : TargetLibCall<"atoi", Int, [Ptr]>;

/// long atol(const char *str);
def atol : TargetLibCall<"atol", Long, [Ptr]>;

```

- **L617**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L618**: Comment documents the nearby API, invariant, or algorithmic intent: `long double atan2l(long double y, long double x);`. / 这行注释说明了附近 API、不变量或算法意图：`long double atan2l(long double y, long double x);`。
- **L619**: Defines TableGen record `atan2l` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `atan2l`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L620**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L621**: Comment documents the nearby API, invariant, or algorithmic intent: `float atanf(float x);`. / 这行注释说明了附近 API、不变量或算法意图：`float atanf(float x);`。
- **L622**: Defines TableGen record `atanf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `atanf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L623**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L624**: Comment documents the nearby API, invariant, or algorithmic intent: `double atanh(double x);`. / 这行注释说明了附近 API、不变量或算法意图：`double atanh(double x);`。
- **L625**: Defines TableGen record `atanh` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `atanh`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L626**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L627**: Comment documents the nearby API, invariant, or algorithmic intent: `float atanhf(float x);`. / 这行注释说明了附近 API、不变量或算法意图：`float atanhf(float x);`。
- **L628**: Defines TableGen record `atanhf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `atanhf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L629**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L630**: Comment documents the nearby API, invariant, or algorithmic intent: `long double atanhl(long double x);`. / 这行注释说明了附近 API、不变量或算法意图：`long double atanhl(long double x);`。
- **L631**: Defines TableGen record `atanhl` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `atanhl`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L632**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L633**: Comment documents the nearby API, invariant, or algorithmic intent: `long double atanl(long double x);`. / 这行注释说明了附近 API、不变量或算法意图：`long double atanl(long double x);`。
- **L634**: Defines TableGen record `atanl` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `atanl`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L635**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L636**: Comment documents the nearby API, invariant, or algorithmic intent: `double atof(const char *str);`. / 这行注释说明了附近 API、不变量或算法意图：`double atof(const char *str);`。
- **L637**: Defines TableGen record `atof` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `atof`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L638**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L639**: Comment documents the nearby API, invariant, or algorithmic intent: `int atoi(const char *str);`. / 这行注释说明了附近 API、不变量或算法意图：`int atoi(const char *str);`。
- **L640**: Defines TableGen record `atoi` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `atoi`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L641**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L642**: Comment documents the nearby API, invariant, or algorithmic intent: `long atol(const char *str);`. / 这行注释说明了附近 API、不变量或算法意图：`long atol(const char *str);`。
- **L643**: Defines TableGen record `atol` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `atol`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L644**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。

### Lines 645-672

```tablegen
/// long long atoll(const char *nptr);
def atoll : TargetLibCall<"atoll", LLong, [Ptr]>;

/// int bcmp(const void *s1, const void *s2, size_t n);
def bcmp : TargetLibCall<"bcmp", Int, [Ptr, Ptr, SizeT]>;

/// void bcopy(const void *s1, void *s2, size_t n);
def bcopy : TargetLibCall<"bcopy", Void, [Ptr, Ptr, SizeT]>;

/// void bzero(void *s, size_t n);
def bzero : TargetLibCall<"bzero", Void, [Ptr, SizeT]>;

/// double cabs(double complex z)
def cabs : TargetLibCall<"cabs", ? /* Checked manually. */>;

/// float cabs(float complex z)
def cabsf : TargetLibCall<"cabsf", ? /* Checked manually. */>;

/// long double cabs(long double complex z)
def cabsl : TargetLibCall<"cabsl", ? /* Checked manually. */>;

/// void *calloc(size_t count, size_t size);
def calloc : TargetLibCall<"calloc", Ptr, [SizeT, SizeT]>;

/// double cbrt(double x);
def cbrt : TargetLibCall<"cbrt", Dbl, [Dbl]>;

/// float cbrtf(float x);
```

- **L645**: Comment documents the nearby API, invariant, or algorithmic intent: `long long atoll(const char *nptr);`. / 这行注释说明了附近 API、不变量或算法意图：`long long atoll(const char *nptr);`。
- **L646**: Defines TableGen record `atoll` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `atoll`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L647**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L648**: Comment documents the nearby API, invariant, or algorithmic intent: `int bcmp(const void *s1, const void *s2, size_t n);`. / 这行注释说明了附近 API、不变量或算法意图：`int bcmp(const void *s1, const void *s2, size_t n);`。
- **L649**: Defines TableGen record `bcmp` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `bcmp`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L650**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L651**: Comment documents the nearby API, invariant, or algorithmic intent: `void bcopy(const void *s1, void *s2, size_t n);`. / 这行注释说明了附近 API、不变量或算法意图：`void bcopy(const void *s1, void *s2, size_t n);`。
- **L652**: Defines TableGen record `bcopy` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `bcopy`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L653**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L654**: Comment documents the nearby API, invariant, or algorithmic intent: `void bzero(void *s, size_t n);`. / 这行注释说明了附近 API、不变量或算法意图：`void bzero(void *s, size_t n);`。
- **L655**: Defines TableGen record `bzero` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `bzero`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L656**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L657**: Comment documents the nearby API, invariant, or algorithmic intent: `double cabs(double complex z)`. / 这行注释说明了附近 API、不变量或算法意图：`double cabs(double complex z)`。
- **L658**: Defines TableGen record `cabs` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `cabs`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L659**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L660**: Comment documents the nearby API, invariant, or algorithmic intent: `float cabs(float complex z)`. / 这行注释说明了附近 API、不变量或算法意图：`float cabs(float complex z)`。
- **L661**: Defines TableGen record `cabsf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `cabsf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L662**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L663**: Comment documents the nearby API, invariant, or algorithmic intent: `long double cabs(long double complex z)`. / 这行注释说明了附近 API、不变量或算法意图：`long double cabs(long double complex z)`。
- **L664**: Defines TableGen record `cabsl` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `cabsl`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L665**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L666**: Comment documents the nearby API, invariant, or algorithmic intent: `void *calloc(size_t count, size_t size);`. / 这行注释说明了附近 API、不变量或算法意图：`void *calloc(size_t count, size_t size);`。
- **L667**: Defines TableGen record `calloc` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `calloc`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L668**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L669**: Comment documents the nearby API, invariant, or algorithmic intent: `double cbrt(double x);`. / 这行注释说明了附近 API、不变量或算法意图：`double cbrt(double x);`。
- **L670**: Defines TableGen record `cbrt` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `cbrt`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L671**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L672**: Comment documents the nearby API, invariant, or algorithmic intent: `float cbrtf(float x);`. / 这行注释说明了附近 API、不变量或算法意图：`float cbrtf(float x);`。

### Lines 673-700

```tablegen
def cbrtf : TargetLibCall<"cbrtf", Flt, [Flt]>;

/// long double cbrtl(long double x);
def cbrtl : TargetLibCall<"cbrtl", LDbl, [LDbl]>;

/// double ceil(double x);
def ceil : TargetLibCall<"ceil", Dbl, [Dbl]>;

/// float ceilf(float x);
def ceilf : TargetLibCall<"ceilf", Flt, [Flt]>;

/// long double ceill(long double x);
def ceill : TargetLibCall<"ceill", LDbl, [LDbl]>;

/// int chmod(const char *path, mode_t mode);
def chmod : TargetLibCall<"chmod", Int, [Ptr, IntX]>;

/// int chown(const char *path, uid_t owner, gid_t group);
def chown : TargetLibCall<"chown", Int, [Ptr, IntX, IntX]>;

/// void clearerr(FILE *stream);
def clearerr : TargetLibCall<"clearerr", Void, [Ptr]>;

/// int closedir(DIR *dirp);
def closedir : TargetLibCall<"closedir", Int, [Ptr]>;

/// double copysign(double x, double y);
def copysign : TargetLibCall<"copysign", Dbl, [Dbl, Dbl]>;
```

- **L673**: Defines TableGen record `cbrtf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `cbrtf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L674**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L675**: Comment documents the nearby API, invariant, or algorithmic intent: `long double cbrtl(long double x);`. / 这行注释说明了附近 API、不变量或算法意图：`long double cbrtl(long double x);`。
- **L676**: Defines TableGen record `cbrtl` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `cbrtl`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L677**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L678**: Comment documents the nearby API, invariant, or algorithmic intent: `double ceil(double x);`. / 这行注释说明了附近 API、不变量或算法意图：`double ceil(double x);`。
- **L679**: Defines TableGen record `ceil` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `ceil`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L680**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L681**: Comment documents the nearby API, invariant, or algorithmic intent: `float ceilf(float x);`. / 这行注释说明了附近 API、不变量或算法意图：`float ceilf(float x);`。
- **L682**: Defines TableGen record `ceilf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `ceilf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L683**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L684**: Comment documents the nearby API, invariant, or algorithmic intent: `long double ceill(long double x);`. / 这行注释说明了附近 API、不变量或算法意图：`long double ceill(long double x);`。
- **L685**: Defines TableGen record `ceill` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `ceill`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L686**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L687**: Comment documents the nearby API, invariant, or algorithmic intent: `int chmod(const char *path, mode_t mode);`. / 这行注释说明了附近 API、不变量或算法意图：`int chmod(const char *path, mode_t mode);`。
- **L688**: Defines TableGen record `chmod` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `chmod`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L689**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L690**: Comment documents the nearby API, invariant, or algorithmic intent: `int chown(const char *path, uid_t owner, gid_t group);`. / 这行注释说明了附近 API、不变量或算法意图：`int chown(const char *path, uid_t owner, gid_t group);`。
- **L691**: Defines TableGen record `chown` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `chown`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L692**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L693**: Comment documents the nearby API, invariant, or algorithmic intent: `void clearerr(FILE *stream);`. / 这行注释说明了附近 API、不变量或算法意图：`void clearerr(FILE *stream);`。
- **L694**: Defines TableGen record `clearerr` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `clearerr`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L695**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L696**: Comment documents the nearby API, invariant, or algorithmic intent: `int closedir(DIR *dirp);`. / 这行注释说明了附近 API、不变量或算法意图：`int closedir(DIR *dirp);`。
- **L697**: Defines TableGen record `closedir` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `closedir`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L698**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L699**: Comment documents the nearby API, invariant, or algorithmic intent: `double copysign(double x, double y);`. / 这行注释说明了附近 API、不变量或算法意图：`double copysign(double x, double y);`。
- **L700**: Defines TableGen record `copysign` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `copysign`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。

### Lines 701-728

```tablegen

/// float copysignf(float x, float y);
def copysignf : TargetLibCall<"copysignf", Flt, [Flt, Flt]>;

/// long double copysignl(long double x, long double y);
def copysignl : TargetLibCall<"copysignl", LDbl, [LDbl, LDbl]>;

/// double cos(double x);
def cos : TargetLibCall<"cos", Dbl, [Dbl]>;

/// float cosf(float x);
def cosf : TargetLibCall<"cosf", Flt, [Flt]>;

/// double cosh(double x);
def cosh : TargetLibCall<"cosh", Dbl, [Dbl]>;

/// float coshf(float x);
def coshf : TargetLibCall<"coshf", Flt, [Flt]>;

/// long double coshl(long double x);
def coshl : TargetLibCall<"coshl", LDbl, [LDbl]>;

/// long double cosl(long double x);
def cosl : TargetLibCall<"cosl", LDbl, [LDbl]>;

/// char *ctermid(char *s);
def ctermid : TargetLibCall<"ctermid", Ptr, [Ptr]>;

```

- **L701**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L702**: Comment documents the nearby API, invariant, or algorithmic intent: `float copysignf(float x, float y);`. / 这行注释说明了附近 API、不变量或算法意图：`float copysignf(float x, float y);`。
- **L703**: Defines TableGen record `copysignf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `copysignf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L704**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L705**: Comment documents the nearby API, invariant, or algorithmic intent: `long double copysignl(long double x, long double y);`. / 这行注释说明了附近 API、不变量或算法意图：`long double copysignl(long double x, long double y);`。
- **L706**: Defines TableGen record `copysignl` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `copysignl`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L707**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L708**: Comment documents the nearby API, invariant, or algorithmic intent: `double cos(double x);`. / 这行注释说明了附近 API、不变量或算法意图：`double cos(double x);`。
- **L709**: Defines TableGen record `cos` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `cos`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L710**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L711**: Comment documents the nearby API, invariant, or algorithmic intent: `float cosf(float x);`. / 这行注释说明了附近 API、不变量或算法意图：`float cosf(float x);`。
- **L712**: Defines TableGen record `cosf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `cosf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L713**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L714**: Comment documents the nearby API, invariant, or algorithmic intent: `double cosh(double x);`. / 这行注释说明了附近 API、不变量或算法意图：`double cosh(double x);`。
- **L715**: Defines TableGen record `cosh` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `cosh`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L716**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L717**: Comment documents the nearby API, invariant, or algorithmic intent: `float coshf(float x);`. / 这行注释说明了附近 API、不变量或算法意图：`float coshf(float x);`。
- **L718**: Defines TableGen record `coshf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `coshf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L719**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L720**: Comment documents the nearby API, invariant, or algorithmic intent: `long double coshl(long double x);`. / 这行注释说明了附近 API、不变量或算法意图：`long double coshl(long double x);`。
- **L721**: Defines TableGen record `coshl` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `coshl`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L722**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L723**: Comment documents the nearby API, invariant, or algorithmic intent: `long double cosl(long double x);`. / 这行注释说明了附近 API、不变量或算法意图：`long double cosl(long double x);`。
- **L724**: Defines TableGen record `cosl` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `cosl`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L725**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L726**: Comment documents the nearby API, invariant, or algorithmic intent: `char *ctermid(char *s);`. / 这行注释说明了附近 API、不变量或算法意图：`char *ctermid(char *s);`。
- **L727**: Defines TableGen record `ctermid` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `ctermid`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L728**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。

### Lines 729-756

```tablegen
/// double erf(double x);
def erf : TargetLibCall<"erf", Dbl, [Dbl]>;

/// float erff(float x);
def erff : TargetLibCall<"erff", Flt, [Flt]>;

/// long double erfl(long double x);
def erfl : TargetLibCall<"erfl", LDbl, [LDbl]>;

/// double tgamma(double x);
def tgamma : TargetLibCall<"tgamma", Dbl, [Dbl]>;

/// float tgammaf(float x);
def tgammaf : TargetLibCall<"tgammaf", Flt, [Flt]>;

/// long double tgammal(long double x);
def tgammal : TargetLibCall<"tgammal", LDbl, [LDbl]>;

/// int execl(const char *path, const char *arg, ...);
def execl : TargetLibCall<"execl", Int, [Ptr, Ptr, Ellip]>;

/// int execle(const char *file, const char *arg, ..., char * const envp[]);
def execle : TargetLibCall<"execle", Int, [Ptr, Ptr, Ellip]>;

/// int execlp(const char *file, const char *arg, ...);
def execlp : TargetLibCall<"execlp", Int, [Ptr, Ptr, Ellip]>;

/// int execv(const char *path, char *const argv[]);
```

- **L729**: Comment documents the nearby API, invariant, or algorithmic intent: `double erf(double x);`. / 这行注释说明了附近 API、不变量或算法意图：`double erf(double x);`。
- **L730**: Defines TableGen record `erf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `erf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L731**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L732**: Comment documents the nearby API, invariant, or algorithmic intent: `float erff(float x);`. / 这行注释说明了附近 API、不变量或算法意图：`float erff(float x);`。
- **L733**: Defines TableGen record `erff` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `erff`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L734**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L735**: Comment documents the nearby API, invariant, or algorithmic intent: `long double erfl(long double x);`. / 这行注释说明了附近 API、不变量或算法意图：`long double erfl(long double x);`。
- **L736**: Defines TableGen record `erfl` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `erfl`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L737**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L738**: Comment documents the nearby API, invariant, or algorithmic intent: `double tgamma(double x);`. / 这行注释说明了附近 API、不变量或算法意图：`double tgamma(double x);`。
- **L739**: Defines TableGen record `tgamma` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `tgamma`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L740**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L741**: Comment documents the nearby API, invariant, or algorithmic intent: `float tgammaf(float x);`. / 这行注释说明了附近 API、不变量或算法意图：`float tgammaf(float x);`。
- **L742**: Defines TableGen record `tgammaf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `tgammaf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L743**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L744**: Comment documents the nearby API, invariant, or algorithmic intent: `long double tgammal(long double x);`. / 这行注释说明了附近 API、不变量或算法意图：`long double tgammal(long double x);`。
- **L745**: Defines TableGen record `tgammal` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `tgammal`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L746**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L747**: Comment documents the nearby API, invariant, or algorithmic intent: `int execl(const char *path, const char *arg, ...);`. / 这行注释说明了附近 API、不变量或算法意图：`int execl(const char *path, const char *arg, ...);`。
- **L748**: Defines TableGen record `execl` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `execl`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L749**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L750**: Comment documents the nearby API, invariant, or algorithmic intent: `int execle(const char *file, const char *arg, ..., char * const envp[]);`. / 这行注释说明了附近 API、不变量或算法意图：`int execle(const char *file, const char *arg, ..., char * const envp[]);`。
- **L751**: Defines TableGen record `execle` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `execle`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L752**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L753**: Comment documents the nearby API, invariant, or algorithmic intent: `int execlp(const char *file, const char *arg, ...);`. / 这行注释说明了附近 API、不变量或算法意图：`int execlp(const char *file, const char *arg, ...);`。
- **L754**: Defines TableGen record `execlp` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `execlp`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L755**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L756**: Comment documents the nearby API, invariant, or algorithmic intent: `int execv(const char *path, char *const argv[]);`. / 这行注释说明了附近 API、不变量或算法意图：`int execv(const char *path, char *const argv[]);`。

### Lines 757-784

```tablegen
def execv : TargetLibCall<"execv", Int, [Ptr, Ptr]>;

/// int execvP(const char *file, const char *search_path, char *const argv[]);
def execvP : TargetLibCall<"execvP", Int, [Ptr, Ptr, Ptr]>;

/// int execve(const char *filename, char *const argv], [char *const envp[]);
def execve : TargetLibCall<"execve", Int, [Ptr, Ptr, Ptr]>;

/// int execvp(const char *file, char *const argv[]);
def execvp : TargetLibCall<"execvp", Int, [Ptr, Ptr]>;

/// int execvpe(const char *file, char *const argv], [char *const envp[]);
def execvpe : TargetLibCall<"execvpe", Int, [Ptr, Ptr, Ptr]>;

/// double exp(double x);
def exp : TargetLibCall<"exp", Dbl, [Dbl]>;

/// double exp10(double x);
def exp10 : TargetLibCall<"exp10", Dbl, [Dbl]>;

/// float exp10f(float x);
def exp10f : TargetLibCall<"exp10f", Flt, [Flt]>;

/// long double exp10l(long double x);
def exp10l : TargetLibCall<"exp10l", LDbl, [LDbl]>;

/// double exp2(double x);
def exp2 : TargetLibCall<"exp2", Dbl, [Dbl]>;
```

- **L757**: Defines TableGen record `execv` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `execv`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L758**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L759**: Comment documents the nearby API, invariant, or algorithmic intent: `int execvP(const char *file, const char *search_path, char *const argv[]);`. / 这行注释说明了附近 API、不变量或算法意图：`int execvP(const char *file, const char *search_path, char *const argv[]);`。
- **L760**: Defines TableGen record `execvP` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `execvP`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L761**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L762**: Comment documents the nearby API, invariant, or algorithmic intent: `int execve(const char *filename, char *const argv], [char *const envp[]);`. / 这行注释说明了附近 API、不变量或算法意图：`int execve(const char *filename, char *const argv], [char *const envp[]);`。
- **L763**: Defines TableGen record `execve` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `execve`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L764**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L765**: Comment documents the nearby API, invariant, or algorithmic intent: `int execvp(const char *file, char *const argv[]);`. / 这行注释说明了附近 API、不变量或算法意图：`int execvp(const char *file, char *const argv[]);`。
- **L766**: Defines TableGen record `execvp` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `execvp`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L767**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L768**: Comment documents the nearby API, invariant, or algorithmic intent: `int execvpe(const char *file, char *const argv], [char *const envp[]);`. / 这行注释说明了附近 API、不变量或算法意图：`int execvpe(const char *file, char *const argv], [char *const envp[]);`。
- **L769**: Defines TableGen record `execvpe` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `execvpe`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L770**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L771**: Comment documents the nearby API, invariant, or algorithmic intent: `double exp(double x);`. / 这行注释说明了附近 API、不变量或算法意图：`double exp(double x);`。
- **L772**: Defines TableGen record `exp` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `exp`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L773**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L774**: Comment documents the nearby API, invariant, or algorithmic intent: `double exp10(double x);`. / 这行注释说明了附近 API、不变量或算法意图：`double exp10(double x);`。
- **L775**: Defines TableGen record `exp10` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `exp10`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L776**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L777**: Comment documents the nearby API, invariant, or algorithmic intent: `float exp10f(float x);`. / 这行注释说明了附近 API、不变量或算法意图：`float exp10f(float x);`。
- **L778**: Defines TableGen record `exp10f` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `exp10f`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L779**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L780**: Comment documents the nearby API, invariant, or algorithmic intent: `long double exp10l(long double x);`. / 这行注释说明了附近 API、不变量或算法意图：`long double exp10l(long double x);`。
- **L781**: Defines TableGen record `exp10l` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `exp10l`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L782**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L783**: Comment documents the nearby API, invariant, or algorithmic intent: `double exp2(double x);`. / 这行注释说明了附近 API、不变量或算法意图：`double exp2(double x);`。
- **L784**: Defines TableGen record `exp2` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `exp2`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。

### Lines 785-812

```tablegen

/// float exp2f(float x);
def exp2f : TargetLibCall<"exp2f", Flt, [Flt]>;

/// long double exp2l(long double x);
def exp2l : TargetLibCall<"exp2l", LDbl, [LDbl]>;

/// float expf(float x);
def expf : TargetLibCall<"expf", Flt, [Flt]>;

/// long double expl(long double x);
def expl : TargetLibCall<"expl", LDbl, [LDbl]>;

/// double expm1(double x);
def expm1 : TargetLibCall<"expm1", Dbl, [Dbl]>;

/// float expm1f(float x);
def expm1f : TargetLibCall<"expm1f", Flt, [Flt]>;

/// long double expm1l(long double x);
def expm1l : TargetLibCall<"expm1l", LDbl, [LDbl]>;

/// double fabs(double x);
def fabs : TargetLibCall<"fabs", Dbl, [Dbl]>;

/// float fabsf(float x);
def fabsf : TargetLibCall<"fabsf", Flt, [Flt]>;

```

- **L785**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L786**: Comment documents the nearby API, invariant, or algorithmic intent: `float exp2f(float x);`. / 这行注释说明了附近 API、不变量或算法意图：`float exp2f(float x);`。
- **L787**: Defines TableGen record `exp2f` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `exp2f`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L788**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L789**: Comment documents the nearby API, invariant, or algorithmic intent: `long double exp2l(long double x);`. / 这行注释说明了附近 API、不变量或算法意图：`long double exp2l(long double x);`。
- **L790**: Defines TableGen record `exp2l` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `exp2l`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L791**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L792**: Comment documents the nearby API, invariant, or algorithmic intent: `float expf(float x);`. / 这行注释说明了附近 API、不变量或算法意图：`float expf(float x);`。
- **L793**: Defines TableGen record `expf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `expf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L794**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L795**: Comment documents the nearby API, invariant, or algorithmic intent: `long double expl(long double x);`. / 这行注释说明了附近 API、不变量或算法意图：`long double expl(long double x);`。
- **L796**: Defines TableGen record `expl` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `expl`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L797**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L798**: Comment documents the nearby API, invariant, or algorithmic intent: `double expm1(double x);`. / 这行注释说明了附近 API、不变量或算法意图：`double expm1(double x);`。
- **L799**: Defines TableGen record `expm1` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `expm1`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L800**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L801**: Comment documents the nearby API, invariant, or algorithmic intent: `float expm1f(float x);`. / 这行注释说明了附近 API、不变量或算法意图：`float expm1f(float x);`。
- **L802**: Defines TableGen record `expm1f` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `expm1f`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L803**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L804**: Comment documents the nearby API, invariant, or algorithmic intent: `long double expm1l(long double x);`. / 这行注释说明了附近 API、不变量或算法意图：`long double expm1l(long double x);`。
- **L805**: Defines TableGen record `expm1l` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `expm1l`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L806**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L807**: Comment documents the nearby API, invariant, or algorithmic intent: `double fabs(double x);`. / 这行注释说明了附近 API、不变量或算法意图：`double fabs(double x);`。
- **L808**: Defines TableGen record `fabs` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `fabs`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L809**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L810**: Comment documents the nearby API, invariant, or algorithmic intent: `float fabsf(float x);`. / 这行注释说明了附近 API、不变量或算法意图：`float fabsf(float x);`。
- **L811**: Defines TableGen record `fabsf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `fabsf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L812**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。

### Lines 813-840

```tablegen
/// long double fabsl(long double x);
def fabsl : TargetLibCall<"fabsl", LDbl, [LDbl]>;

/// int fclose(FILE *stream);
def fclose : TargetLibCall<"fclose", Int, [Ptr]>;

/// FILE *fdopen(int fildes, const char *mode);
def fdopen : TargetLibCall<"fdopen", Ptr, [Int, Ptr]>;

/// int feof(FILE *stream);
def feof : TargetLibCall<"feof", Int, [Ptr]>;

/// int ferror(FILE *stream);
def ferror : TargetLibCall<"ferror", Int, [Ptr]>;

/// int fflush(FILE *stream);
def fflush : TargetLibCall<"fflush", Int, [Ptr]>;

/// int ffs(int i);
def ffs : TargetLibCall<"ffs", Int, [Int]>;

/// int ffsl(long int i);
def ffsl : TargetLibCall<"ffsl", Int, [Long]>;

/// int ffsll(long long int i);
def ffsll : TargetLibCall<"ffsll", Int, [LLong]>;

/// int fgetc(FILE *stream);
```

- **L813**: Comment documents the nearby API, invariant, or algorithmic intent: `long double fabsl(long double x);`. / 这行注释说明了附近 API、不变量或算法意图：`long double fabsl(long double x);`。
- **L814**: Defines TableGen record `fabsl` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `fabsl`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L815**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L816**: Comment documents the nearby API, invariant, or algorithmic intent: `int fclose(FILE *stream);`. / 这行注释说明了附近 API、不变量或算法意图：`int fclose(FILE *stream);`。
- **L817**: Defines TableGen record `fclose` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `fclose`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L818**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L819**: Comment documents the nearby API, invariant, or algorithmic intent: `FILE *fdopen(int fildes, const char *mode);`. / 这行注释说明了附近 API、不变量或算法意图：`FILE *fdopen(int fildes, const char *mode);`。
- **L820**: Defines TableGen record `fdopen` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `fdopen`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L821**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L822**: Comment documents the nearby API, invariant, or algorithmic intent: `int feof(FILE *stream);`. / 这行注释说明了附近 API、不变量或算法意图：`int feof(FILE *stream);`。
- **L823**: Defines TableGen record `feof` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `feof`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L824**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L825**: Comment documents the nearby API, invariant, or algorithmic intent: `int ferror(FILE *stream);`. / 这行注释说明了附近 API、不变量或算法意图：`int ferror(FILE *stream);`。
- **L826**: Defines TableGen record `ferror` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `ferror`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L827**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L828**: Comment documents the nearby API, invariant, or algorithmic intent: `int fflush(FILE *stream);`. / 这行注释说明了附近 API、不变量或算法意图：`int fflush(FILE *stream);`。
- **L829**: Defines TableGen record `fflush` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `fflush`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L830**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L831**: Comment documents the nearby API, invariant, or algorithmic intent: `int ffs(int i);`. / 这行注释说明了附近 API、不变量或算法意图：`int ffs(int i);`。
- **L832**: Defines TableGen record `ffs` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `ffs`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L833**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L834**: Comment documents the nearby API, invariant, or algorithmic intent: `int ffsl(long int i);`. / 这行注释说明了附近 API、不变量或算法意图：`int ffsl(long int i);`。
- **L835**: Defines TableGen record `ffsl` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `ffsl`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L836**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L837**: Comment documents the nearby API, invariant, or algorithmic intent: `int ffsll(long long int i);`. / 这行注释说明了附近 API、不变量或算法意图：`int ffsll(long long int i);`。
- **L838**: Defines TableGen record `ffsll` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `ffsll`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L839**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L840**: Comment documents the nearby API, invariant, or algorithmic intent: `int fgetc(FILE *stream);`. / 这行注释说明了附近 API、不变量或算法意图：`int fgetc(FILE *stream);`。

### Lines 841-868

```tablegen
def fgetc : TargetLibCall<"fgetc", Int, [Ptr]>;

/// int fgetc_unlocked(FILE *stream);
def fgetc_unlocked : TargetLibCall<"fgetc_unlocked", Int, [Ptr]>;

/// int fgetpos(FILE *stream, fpos_t *pos);
def fgetpos : TargetLibCall<"fgetpos", Int, [Ptr, Ptr]>;

/// char *fgets(char *s, int n, FILE *stream);
def fgets : TargetLibCall<"fgets", Ptr, [Ptr, Int, Ptr]>;

/// char *fgets_unlocked(char *s, int n, FILE *stream);
def fgets_unlocked : TargetLibCall<"fgets_unlocked", Ptr, [Ptr, Int, Ptr]>;

/// int fileno(FILE *stream);
def fileno : TargetLibCall<"fileno", Int, [Ptr]>;

/// int fiprintf(FILE *stream, const char *format, ...);
def fiprintf : TargetLibCall<"fiprintf", Int, [Ptr, Ptr, Ellip]>;

/// void flockfile(FILE *file);
def flockfile : TargetLibCall<"flockfile", Void, [Ptr]>;

/// double floor(double x);
def floor : TargetLibCall<"floor", Dbl, [Dbl]>;

/// float floorf(float x);
def floorf : TargetLibCall<"floorf", Flt, [Flt]>;
```

- **L841**: Defines TableGen record `fgetc` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `fgetc`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L842**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L843**: Comment documents the nearby API, invariant, or algorithmic intent: `int fgetc_unlocked(FILE *stream);`. / 这行注释说明了附近 API、不变量或算法意图：`int fgetc_unlocked(FILE *stream);`。
- **L844**: Defines TableGen record `fgetc_unlocked` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `fgetc_unlocked`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L845**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L846**: Comment documents the nearby API, invariant, or algorithmic intent: `int fgetpos(FILE *stream, fpos_t *pos);`. / 这行注释说明了附近 API、不变量或算法意图：`int fgetpos(FILE *stream, fpos_t *pos);`。
- **L847**: Defines TableGen record `fgetpos` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `fgetpos`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L848**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L849**: Comment documents the nearby API, invariant, or algorithmic intent: `char *fgets(char *s, int n, FILE *stream);`. / 这行注释说明了附近 API、不变量或算法意图：`char *fgets(char *s, int n, FILE *stream);`。
- **L850**: Defines TableGen record `fgets` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `fgets`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L851**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L852**: Comment documents the nearby API, invariant, or algorithmic intent: `char *fgets_unlocked(char *s, int n, FILE *stream);`. / 这行注释说明了附近 API、不变量或算法意图：`char *fgets_unlocked(char *s, int n, FILE *stream);`。
- **L853**: Defines TableGen record `fgets_unlocked` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `fgets_unlocked`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L854**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L855**: Comment documents the nearby API, invariant, or algorithmic intent: `int fileno(FILE *stream);`. / 这行注释说明了附近 API、不变量或算法意图：`int fileno(FILE *stream);`。
- **L856**: Defines TableGen record `fileno` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `fileno`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L857**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L858**: Comment documents the nearby API, invariant, or algorithmic intent: `int fiprintf(FILE *stream, const char *format, ...);`. / 这行注释说明了附近 API、不变量或算法意图：`int fiprintf(FILE *stream, const char *format, ...);`。
- **L859**: Defines TableGen record `fiprintf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `fiprintf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L860**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L861**: Comment documents the nearby API, invariant, or algorithmic intent: `void flockfile(FILE *file);`. / 这行注释说明了附近 API、不变量或算法意图：`void flockfile(FILE *file);`。
- **L862**: Defines TableGen record `flockfile` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `flockfile`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L863**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L864**: Comment documents the nearby API, invariant, or algorithmic intent: `double floor(double x);`. / 这行注释说明了附近 API、不变量或算法意图：`double floor(double x);`。
- **L865**: Defines TableGen record `floor` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `floor`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L866**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L867**: Comment documents the nearby API, invariant, or algorithmic intent: `float floorf(float x);`. / 这行注释说明了附近 API、不变量或算法意图：`float floorf(float x);`。
- **L868**: Defines TableGen record `floorf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `floorf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。

### Lines 869-896

```tablegen

/// long double floorl(long double x);
def floorl : TargetLibCall<"floorl", LDbl, [LDbl]>;

/// int fls(int i);
def fls : TargetLibCall<"fls", Int, [Int]>;

/// int flsl(long int i);
def flsl : TargetLibCall<"flsl", Int, [Long]>;

/// int flsll(long long int i);
def flsll : TargetLibCall<"flsll", Int, [LLong]>;

// Calls to fmax and fmin library functions expand to the llvm.maxnnum and
// llvm.minnum intrinsics with the correct parameter types for the arguments
// (all types must match).
/// double fmax(double x, double y);
def fmax : TargetLibCall<"fmax", Floating, [Same, Same]>;

/// float fmaxf(float x, float y);
def fmaxf : TargetLibCall<"fmaxf", Floating, [Same, Same]>;

/// long double fmaxl(long double x, long double y);
def fmaxl : TargetLibCall<"fmaxl", Floating, [Same, Same]>;

/// double fmin(double x, double y);
def fmin : TargetLibCall<"fmin", Floating, [Same, Same]>;

```

- **L869**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L870**: Comment documents the nearby API, invariant, or algorithmic intent: `long double floorl(long double x);`. / 这行注释说明了附近 API、不变量或算法意图：`long double floorl(long double x);`。
- **L871**: Defines TableGen record `floorl` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `floorl`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L872**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L873**: Comment documents the nearby API, invariant, or algorithmic intent: `int fls(int i);`. / 这行注释说明了附近 API、不变量或算法意图：`int fls(int i);`。
- **L874**: Defines TableGen record `fls` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `fls`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L875**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L876**: Comment documents the nearby API, invariant, or algorithmic intent: `int flsl(long int i);`. / 这行注释说明了附近 API、不变量或算法意图：`int flsl(long int i);`。
- **L877**: Defines TableGen record `flsl` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `flsl`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L878**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L879**: Comment documents the nearby API, invariant, or algorithmic intent: `int flsll(long long int i);`. / 这行注释说明了附近 API、不变量或算法意图：`int flsll(long long int i);`。
- **L880**: Defines TableGen record `flsll` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `flsll`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L881**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L882**: Comment documents the nearby API, invariant, or algorithmic intent: `Calls to fmax and fmin library functions expand to the llvm.maxnnum and`. / 这行注释说明了附近 API、不变量或算法意图：`Calls to fmax and fmin library functions expand to the llvm.maxnnum and`。
- **L883**: Comment documents the nearby API, invariant, or algorithmic intent: `llvm.minnum intrinsics with the correct parameter types for the arguments`. / 这行注释说明了附近 API、不变量或算法意图：`llvm.minnum intrinsics with the correct parameter types for the arguments`。
- **L884**: Comment documents the nearby API, invariant, or algorithmic intent: `(all types must match).`. / 这行注释说明了附近 API、不变量或算法意图：`(all types must match).`。
- **L885**: Comment documents the nearby API, invariant, or algorithmic intent: `double fmax(double x, double y);`. / 这行注释说明了附近 API、不变量或算法意图：`double fmax(double x, double y);`。
- **L886**: Defines TableGen record `fmax` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `fmax`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L887**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L888**: Comment documents the nearby API, invariant, or algorithmic intent: `float fmaxf(float x, float y);`. / 这行注释说明了附近 API、不变量或算法意图：`float fmaxf(float x, float y);`。
- **L889**: Defines TableGen record `fmaxf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `fmaxf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L890**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L891**: Comment documents the nearby API, invariant, or algorithmic intent: `long double fmaxl(long double x, long double y);`. / 这行注释说明了附近 API、不变量或算法意图：`long double fmaxl(long double x, long double y);`。
- **L892**: Defines TableGen record `fmaxl` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `fmaxl`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L893**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L894**: Comment documents the nearby API, invariant, or algorithmic intent: `double fmin(double x, double y);`. / 这行注释说明了附近 API、不变量或算法意图：`double fmin(double x, double y);`。
- **L895**: Defines TableGen record `fmin` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `fmin`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L896**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。

### Lines 897-924

```tablegen
/// float fminf(float x, float y);
def fminf : TargetLibCall<"fminf", Floating, [Same, Same]>;

/// long double fminl(long double x, long double y);
def fminl : TargetLibCall<"fminl", Floating, [Same, Same]>;

// Calls to fmaximum_num and fminimum_num library functions expand to the
// llvm.maximumnum and llvm.minimumnum intrinsics with the correct parameter
// types for the arguments (all types must match).
/// double fmaximum_num(double x, double y);
def fmaximum_num : TargetLibCall<"fmaximum_num", Floating, [Same, Same]>;

/// float fmaximum_numf(float x, float y);
def fmaximum_numf : TargetLibCall<"fmaximum_numf", Floating, [Same, Same]>;

/// long double fmaximum_numl(long double x, long double y);
def fmaximum_numl : TargetLibCall<"fmaximum_numl", Floating, [Same, Same]>;

/// double fminimum_num(double x, double y);
def fminimum_num : TargetLibCall<"fminimum_num", Floating, [Same, Same]>;

/// float fminimum_numf(float x, float y);
def fminimum_numf : TargetLibCall<"fminimum_numf", Floating, [Same, Same]>;

/// long double fminimum_numl(long double x, long double y);
def fminimum_numl : TargetLibCall<"fminimum_numl", Floating, [Same, Same]>;

/// double fmod(double x, double y);
```

- **L897**: Comment documents the nearby API, invariant, or algorithmic intent: `float fminf(float x, float y);`. / 这行注释说明了附近 API、不变量或算法意图：`float fminf(float x, float y);`。
- **L898**: Defines TableGen record `fminf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `fminf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L899**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L900**: Comment documents the nearby API, invariant, or algorithmic intent: `long double fminl(long double x, long double y);`. / 这行注释说明了附近 API、不变量或算法意图：`long double fminl(long double x, long double y);`。
- **L901**: Defines TableGen record `fminl` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `fminl`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L902**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L903**: Comment documents the nearby API, invariant, or algorithmic intent: `Calls to fmaximum_num and fminimum_num library functions expand to the`. / 这行注释说明了附近 API、不变量或算法意图：`Calls to fmaximum_num and fminimum_num library functions expand to the`。
- **L904**: Comment documents the nearby API, invariant, or algorithmic intent: `llvm.maximumnum and llvm.minimumnum intrinsics with the correct parameter`. / 这行注释说明了附近 API、不变量或算法意图：`llvm.maximumnum and llvm.minimumnum intrinsics with the correct parameter`。
- **L905**: Comment documents the nearby API, invariant, or algorithmic intent: `types for the arguments (all types must match).`. / 这行注释说明了附近 API、不变量或算法意图：`types for the arguments (all types must match).`。
- **L906**: Comment documents the nearby API, invariant, or algorithmic intent: `double fmaximum_num(double x, double y);`. / 这行注释说明了附近 API、不变量或算法意图：`double fmaximum_num(double x, double y);`。
- **L907**: Defines TableGen record `fmaximum_num` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `fmaximum_num`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L908**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L909**: Comment documents the nearby API, invariant, or algorithmic intent: `float fmaximum_numf(float x, float y);`. / 这行注释说明了附近 API、不变量或算法意图：`float fmaximum_numf(float x, float y);`。
- **L910**: Defines TableGen record `fmaximum_numf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `fmaximum_numf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L911**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L912**: Comment documents the nearby API, invariant, or algorithmic intent: `long double fmaximum_numl(long double x, long double y);`. / 这行注释说明了附近 API、不变量或算法意图：`long double fmaximum_numl(long double x, long double y);`。
- **L913**: Defines TableGen record `fmaximum_numl` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `fmaximum_numl`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L914**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L915**: Comment documents the nearby API, invariant, or algorithmic intent: `double fminimum_num(double x, double y);`. / 这行注释说明了附近 API、不变量或算法意图：`double fminimum_num(double x, double y);`。
- **L916**: Defines TableGen record `fminimum_num` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `fminimum_num`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L917**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L918**: Comment documents the nearby API, invariant, or algorithmic intent: `float fminimum_numf(float x, float y);`. / 这行注释说明了附近 API、不变量或算法意图：`float fminimum_numf(float x, float y);`。
- **L919**: Defines TableGen record `fminimum_numf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `fminimum_numf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L920**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L921**: Comment documents the nearby API, invariant, or algorithmic intent: `long double fminimum_numl(long double x, long double y);`. / 这行注释说明了附近 API、不变量或算法意图：`long double fminimum_numl(long double x, long double y);`。
- **L922**: Defines TableGen record `fminimum_numl` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `fminimum_numl`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L923**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L924**: Comment documents the nearby API, invariant, or algorithmic intent: `double fmod(double x, double y);`. / 这行注释说明了附近 API、不变量或算法意图：`double fmod(double x, double y);`。

### Lines 925-952

```tablegen
def fmod : TargetLibCall<"fmod", Dbl, [Dbl, Dbl]>;

/// float fmodf(float x, float y);
def fmodf : TargetLibCall<"fmodf", Flt, [Flt, Flt]>;

/// long double fmodl(long double x, long double y);
def fmodl : TargetLibCall<"fmodl", LDbl, [LDbl, LDbl]>;

/// FILE *fopen(const char *filename, const char *mode);
def fopen : TargetLibCall<"fopen", Ptr, [Ptr, Ptr]>;

/// FILE *fopen64(const char *filename, const char *opentype)
def fopen64 : TargetLibCall<"fopen64", Ptr, [Ptr, Ptr]>;

/// int fork();
def fork : TargetLibCall<"fork", Int, []>;

/// int fprintf(FILE *stream, const char *format, ...);
def fprintf : TargetLibCall<"fprintf", Int, [Ptr, Ptr, Ellip]>;

/// int fputc(int c, FILE *stream);
def fputc : TargetLibCall<"fputc", Int, [Int, Ptr]>;

/// int fputc_unlocked(int c, FILE *stream);
def fputc_unlocked : TargetLibCall<"fputc_unlocked", Int, [Int, Ptr]>;

/// int fputs(const char *s, FILE *stream);
def fputs : TargetLibCall<"fputs", Int, [Ptr, Ptr]>;
```

- **L925**: Defines TableGen record `fmod` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `fmod`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L926**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L927**: Comment documents the nearby API, invariant, or algorithmic intent: `float fmodf(float x, float y);`. / 这行注释说明了附近 API、不变量或算法意图：`float fmodf(float x, float y);`。
- **L928**: Defines TableGen record `fmodf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `fmodf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L929**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L930**: Comment documents the nearby API, invariant, or algorithmic intent: `long double fmodl(long double x, long double y);`. / 这行注释说明了附近 API、不变量或算法意图：`long double fmodl(long double x, long double y);`。
- **L931**: Defines TableGen record `fmodl` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `fmodl`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L932**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L933**: Comment documents the nearby API, invariant, or algorithmic intent: `FILE *fopen(const char *filename, const char *mode);`. / 这行注释说明了附近 API、不变量或算法意图：`FILE *fopen(const char *filename, const char *mode);`。
- **L934**: Defines TableGen record `fopen` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `fopen`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L935**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L936**: Comment documents the nearby API, invariant, or algorithmic intent: `FILE *fopen64(const char *filename, const char *opentype)`. / 这行注释说明了附近 API、不变量或算法意图：`FILE *fopen64(const char *filename, const char *opentype)`。
- **L937**: Defines TableGen record `fopen64` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `fopen64`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L938**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L939**: Comment documents the nearby API, invariant, or algorithmic intent: `int fork();`. / 这行注释说明了附近 API、不变量或算法意图：`int fork();`。
- **L940**: Defines TableGen record `fork` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `fork`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L941**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L942**: Comment documents the nearby API, invariant, or algorithmic intent: `int fprintf(FILE *stream, const char *format, ...);`. / 这行注释说明了附近 API、不变量或算法意图：`int fprintf(FILE *stream, const char *format, ...);`。
- **L943**: Defines TableGen record `fprintf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `fprintf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L944**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L945**: Comment documents the nearby API, invariant, or algorithmic intent: `int fputc(int c, FILE *stream);`. / 这行注释说明了附近 API、不变量或算法意图：`int fputc(int c, FILE *stream);`。
- **L946**: Defines TableGen record `fputc` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `fputc`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L947**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L948**: Comment documents the nearby API, invariant, or algorithmic intent: `int fputc_unlocked(int c, FILE *stream);`. / 这行注释说明了附近 API、不变量或算法意图：`int fputc_unlocked(int c, FILE *stream);`。
- **L949**: Defines TableGen record `fputc_unlocked` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `fputc_unlocked`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L950**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L951**: Comment documents the nearby API, invariant, or algorithmic intent: `int fputs(const char *s, FILE *stream);`. / 这行注释说明了附近 API、不变量或算法意图：`int fputs(const char *s, FILE *stream);`。
- **L952**: Defines TableGen record `fputs` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `fputs`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。

### Lines 953-980

```tablegen

/// int fputs_unlocked(const char *s, FILE *stream);
def fputs_unlocked : TargetLibCall<"fputs_unlocked", Int, [Ptr, Ptr]>;

/// size_t fread(void *ptr, size_t size, size_t nitems, FILE *stream);
def fread : TargetLibCall<"fread", SizeT, [Ptr, SizeT, SizeT, Ptr]>;

/// size_t fread_unlocked(void *ptr, size_t size, size_t nitems, FILE *stream);
def fread_unlocked
    : TargetLibCall<"fread_unlocked", SizeT, [Ptr, SizeT, SizeT, Ptr]>;

/// void free(void *ptr);
def free : TargetLibCall<"free", Void, [Ptr]>;

/// double frexp(double num, int *exp);
def frexp : TargetLibCall<"frexp", Dbl, [Dbl, Ptr]>;

/// float frexpf(float num, int *exp);
def frexpf : TargetLibCall<"frexpf", Flt, [Flt, Ptr]>;

/// long double frexpl(long double num, int *exp);
def frexpl : TargetLibCall<"frexpl", LDbl, [LDbl, Ptr]>;

/// int fscanf(FILE *stream, const char *format, ... );
def fscanf : TargetLibCall<"fscanf", Int, [Ptr, Ptr, Ellip]>;

/// int fseek(FILE *stream, long offset, int whence);
def fseek : TargetLibCall<"fseek", Int, [Ptr, Long, Int]>;
```

- **L953**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L954**: Comment documents the nearby API, invariant, or algorithmic intent: `int fputs_unlocked(const char *s, FILE *stream);`. / 这行注释说明了附近 API、不变量或算法意图：`int fputs_unlocked(const char *s, FILE *stream);`。
- **L955**: Defines TableGen record `fputs_unlocked` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `fputs_unlocked`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L956**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L957**: Comment documents the nearby API, invariant, or algorithmic intent: `size_t fread(void *ptr, size_t size, size_t nitems, FILE *stream);`. / 这行注释说明了附近 API、不变量或算法意图：`size_t fread(void *ptr, size_t size, size_t nitems, FILE *stream);`。
- **L958**: Defines TableGen record `fread` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `fread`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L959**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L960**: Comment documents the nearby API, invariant, or algorithmic intent: `size_t fread_unlocked(void *ptr, size_t size, size_t nitems, FILE *stream);`. / 这行注释说明了附近 API、不变量或算法意图：`size_t fread_unlocked(void *ptr, size_t size, size_t nitems, FILE *stream);`。
- **L961**: Defines TableGen record `fread_unlocked`, adding one entry to the generated description database. / 定义 TableGen 记录 `fread_unlocked`，向生成用描述数据库中加入一个条目。
- **L962**: Provides a declaration or standalone statement that contributes to the surrounding API or implementation. / 给出一条声明或独立语句，为周围的 API 或实现提供支撑。
- **L963**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L964**: Comment documents the nearby API, invariant, or algorithmic intent: `void free(void *ptr);`. / 这行注释说明了附近 API、不变量或算法意图：`void free(void *ptr);`。
- **L965**: Defines TableGen record `free` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `free`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L966**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L967**: Comment documents the nearby API, invariant, or algorithmic intent: `double frexp(double num, int *exp);`. / 这行注释说明了附近 API、不变量或算法意图：`double frexp(double num, int *exp);`。
- **L968**: Defines TableGen record `frexp` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `frexp`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L969**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L970**: Comment documents the nearby API, invariant, or algorithmic intent: `float frexpf(float num, int *exp);`. / 这行注释说明了附近 API、不变量或算法意图：`float frexpf(float num, int *exp);`。
- **L971**: Defines TableGen record `frexpf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `frexpf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L972**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L973**: Comment documents the nearby API, invariant, or algorithmic intent: `long double frexpl(long double num, int *exp);`. / 这行注释说明了附近 API、不变量或算法意图：`long double frexpl(long double num, int *exp);`。
- **L974**: Defines TableGen record `frexpl` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `frexpl`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L975**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L976**: Comment documents the nearby API, invariant, or algorithmic intent: `int fscanf(FILE *stream, const char *format, ... );`. / 这行注释说明了附近 API、不变量或算法意图：`int fscanf(FILE *stream, const char *format, ... );`。
- **L977**: Defines TableGen record `fscanf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `fscanf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L978**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L979**: Comment documents the nearby API, invariant, or algorithmic intent: `int fseek(FILE *stream, long offset, int whence);`. / 这行注释说明了附近 API、不变量或算法意图：`int fseek(FILE *stream, long offset, int whence);`。
- **L980**: Defines TableGen record `fseek` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `fseek`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。

### Lines 981-1008

```tablegen

/// int fseeko(FILE *stream, off_t offset, int whence);
def fseeko : TargetLibCall<"fseeko", Int, [Ptr, IntX, Int]>;

/// int fseeko64(FILE *stream, off64_t offset, int whence)
def fseeko64 : TargetLibCall<"fseeko64", Int, [Ptr, Int64, Int]>;

/// int fsetpos(FILE *stream, const fpos_t *pos);
def fsetpos : TargetLibCall<"fsetpos", Int, [Ptr, Ptr]>;

/// int fstat(int fildes, struct stat *buf);
def fstat : TargetLibCall<"fstat", Int, [Int, Ptr]>;

/// int fstat64(int filedes, struct stat64 *buf)
def fstat64 : TargetLibCall<"fstat64", Int, [Int, Ptr]>;

/// int fstatvfs(int fildes, struct statvfs *buf);
def fstatvfs : TargetLibCall<"fstatvfs", Int, [Int, Ptr]>;

/// int fstatvfs64(int fildes, struct statvfs64 *buf);
def fstatvfs64 : TargetLibCall<"fstatvfs64", Int, [Int, Ptr]>;

/// long ftell(FILE *stream);
def ftell : TargetLibCall<"ftell", Long, [Ptr]>;

/// off_t ftello(FILE *stream);
def ftello : TargetLibCall<"ftello", IntPlus, [Ptr]>;

```

- **L981**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L982**: Comment documents the nearby API, invariant, or algorithmic intent: `int fseeko(FILE *stream, off_t offset, int whence);`. / 这行注释说明了附近 API、不变量或算法意图：`int fseeko(FILE *stream, off_t offset, int whence);`。
- **L983**: Defines TableGen record `fseeko` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `fseeko`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L984**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L985**: Comment documents the nearby API, invariant, or algorithmic intent: `int fseeko64(FILE *stream, off64_t offset, int whence)`. / 这行注释说明了附近 API、不变量或算法意图：`int fseeko64(FILE *stream, off64_t offset, int whence)`。
- **L986**: Defines TableGen record `fseeko64` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `fseeko64`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L987**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L988**: Comment documents the nearby API, invariant, or algorithmic intent: `int fsetpos(FILE *stream, const fpos_t *pos);`. / 这行注释说明了附近 API、不变量或算法意图：`int fsetpos(FILE *stream, const fpos_t *pos);`。
- **L989**: Defines TableGen record `fsetpos` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `fsetpos`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L990**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L991**: Comment documents the nearby API, invariant, or algorithmic intent: `int fstat(int fildes, struct stat *buf);`. / 这行注释说明了附近 API、不变量或算法意图：`int fstat(int fildes, struct stat *buf);`。
- **L992**: Defines TableGen record `fstat` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `fstat`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L993**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L994**: Comment documents the nearby API, invariant, or algorithmic intent: `int fstat64(int filedes, struct stat64 *buf)`. / 这行注释说明了附近 API、不变量或算法意图：`int fstat64(int filedes, struct stat64 *buf)`。
- **L995**: Defines TableGen record `fstat64` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `fstat64`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L996**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L997**: Comment documents the nearby API, invariant, or algorithmic intent: `int fstatvfs(int fildes, struct statvfs *buf);`. / 这行注释说明了附近 API、不变量或算法意图：`int fstatvfs(int fildes, struct statvfs *buf);`。
- **L998**: Defines TableGen record `fstatvfs` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `fstatvfs`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L999**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1000**: Comment documents the nearby API, invariant, or algorithmic intent: `int fstatvfs64(int fildes, struct statvfs64 *buf);`. / 这行注释说明了附近 API、不变量或算法意图：`int fstatvfs64(int fildes, struct statvfs64 *buf);`。
- **L1001**: Defines TableGen record `fstatvfs64` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `fstatvfs64`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1002**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1003**: Comment documents the nearby API, invariant, or algorithmic intent: `long ftell(FILE *stream);`. / 这行注释说明了附近 API、不变量或算法意图：`long ftell(FILE *stream);`。
- **L1004**: Defines TableGen record `ftell` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `ftell`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1005**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1006**: Comment documents the nearby API, invariant, or algorithmic intent: `off_t ftello(FILE *stream);`. / 这行注释说明了附近 API、不变量或算法意图：`off_t ftello(FILE *stream);`。
- **L1007**: Defines TableGen record `ftello` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `ftello`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1008**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。

### Lines 1009-1036

```tablegen
/// off64_t ftello64(FILE *stream)
def ftello64 : TargetLibCall<"ftello64", Int64, [Ptr]>;

/// int ftrylockfile(FILE *file);
def ftrylockfile : TargetLibCall<"ftrylockfile", Int, [Ptr]>;

/// void funlockfile(FILE *file);
def funlockfile : TargetLibCall<"funlockfile", Void, [Ptr]>;

/// size_t fwrite(const void *ptr, size_t size, size_t nitems, FILE *stream);
def fwrite : TargetLibCall<"fwrite", SizeT, [Ptr, SizeT, SizeT, Ptr]>;

/// size_t fwrite_unlocked(const void *ptr, size_t size, size_t nitems,
/// FILE *stream);
def fwrite_unlocked
    : TargetLibCall<"fwrite_unlocked", SizeT, [Ptr, SizeT, SizeT, Ptr]>;

/// int getc(FILE *stream);
def getc : TargetLibCall<"getc", Int, [Ptr]>;

/// int getc_unlocked(FILE *stream);
def getc_unlocked : TargetLibCall<"getc_unlocked", Int, [Ptr]>;

/// int getchar(void);
def getchar : TargetLibCall<"getchar", Int, []>;

/// int getchar_unlocked(void);
def getchar_unlocked : TargetLibCall<"getchar_unlocked", Int, []>;
```

- **L1009**: Comment documents the nearby API, invariant, or algorithmic intent: `off64_t ftello64(FILE *stream)`. / 这行注释说明了附近 API、不变量或算法意图：`off64_t ftello64(FILE *stream)`。
- **L1010**: Defines TableGen record `ftello64` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `ftello64`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1011**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1012**: Comment documents the nearby API, invariant, or algorithmic intent: `int ftrylockfile(FILE *file);`. / 这行注释说明了附近 API、不变量或算法意图：`int ftrylockfile(FILE *file);`。
- **L1013**: Defines TableGen record `ftrylockfile` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `ftrylockfile`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1014**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1015**: Comment documents the nearby API, invariant, or algorithmic intent: `void funlockfile(FILE *file);`. / 这行注释说明了附近 API、不变量或算法意图：`void funlockfile(FILE *file);`。
- **L1016**: Defines TableGen record `funlockfile` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `funlockfile`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1017**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1018**: Comment documents the nearby API, invariant, or algorithmic intent: `size_t fwrite(const void *ptr, size_t size, size_t nitems, FILE *stream);`. / 这行注释说明了附近 API、不变量或算法意图：`size_t fwrite(const void *ptr, size_t size, size_t nitems, FILE *stream);`。
- **L1019**: Defines TableGen record `fwrite` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `fwrite`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1020**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1021**: Comment documents the nearby API, invariant, or algorithmic intent: `size_t fwrite_unlocked(const void *ptr, size_t size, size_t nitems,`. / 这行注释说明了附近 API、不变量或算法意图：`size_t fwrite_unlocked(const void *ptr, size_t size, size_t nitems,`。
- **L1022**: Comment documents the nearby API, invariant, or algorithmic intent: `FILE *stream);`. / 这行注释说明了附近 API、不变量或算法意图：`FILE *stream);`。
- **L1023**: Defines TableGen record `fwrite_unlocked`, adding one entry to the generated description database. / 定义 TableGen 记录 `fwrite_unlocked`，向生成用描述数据库中加入一个条目。
- **L1024**: Provides a declaration or standalone statement that contributes to the surrounding API or implementation. / 给出一条声明或独立语句，为周围的 API 或实现提供支撑。
- **L1025**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1026**: Comment documents the nearby API, invariant, or algorithmic intent: `int getc(FILE *stream);`. / 这行注释说明了附近 API、不变量或算法意图：`int getc(FILE *stream);`。
- **L1027**: Defines TableGen record `getc` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `getc`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1028**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1029**: Comment documents the nearby API, invariant, or algorithmic intent: `int getc_unlocked(FILE *stream);`. / 这行注释说明了附近 API、不变量或算法意图：`int getc_unlocked(FILE *stream);`。
- **L1030**: Defines TableGen record `getc_unlocked` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `getc_unlocked`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1031**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1032**: Comment documents the nearby API, invariant, or algorithmic intent: `int getchar(void);`. / 这行注释说明了附近 API、不变量或算法意图：`int getchar(void);`。
- **L1033**: Defines TableGen record `getchar` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `getchar`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1034**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1035**: Comment documents the nearby API, invariant, or algorithmic intent: `int getchar_unlocked(void);`. / 这行注释说明了附近 API、不变量或算法意图：`int getchar_unlocked(void);`。
- **L1036**: Defines TableGen record `getchar_unlocked` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `getchar_unlocked`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。

### Lines 1037-1064

```tablegen

/// char *getenv(const char *name);
def getenv : TargetLibCall<"getenv", Ptr, [Ptr]>;

/// int getitimer(int which, struct itimerval *value);
def getitimer : TargetLibCall<"getitimer", Int, [Int, Ptr]>;

/// int getlogin_r(char *name, size_t namesize);
def getlogin_r : TargetLibCall<"getlogin_r", Int, [Ptr, SizeT]>;

/// struct passwd *getpwnam(const char *name);
def getpwnam : TargetLibCall<"getpwnam", Ptr, [Ptr]>;

/// char *gets(char *s);
def gets : TargetLibCall<"gets", Ptr, [Ptr]>;

/// int gettimeofday(struct timeval *tp, void *tzp);
def gettimeofday : TargetLibCall<"gettimeofday", Int, [Ptr, Ptr]>;

/// uint32_t htonl(uint32_t hostlong);
def htonl : TargetLibCall<"htonl", Int32, [Int32]>;

/// uint16_t htons(uint16_t hostshort);
def htons : TargetLibCall<"htons", Int16, [Int16]>;

/// double hypot(double x, double y);
def hypot : TargetLibCall<"hypot", Dbl, [Dbl, Dbl]>;

```

- **L1037**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1038**: Comment documents the nearby API, invariant, or algorithmic intent: `char *getenv(const char *name);`. / 这行注释说明了附近 API、不变量或算法意图：`char *getenv(const char *name);`。
- **L1039**: Defines TableGen record `getenv` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `getenv`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1040**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1041**: Comment documents the nearby API, invariant, or algorithmic intent: `int getitimer(int which, struct itimerval *value);`. / 这行注释说明了附近 API、不变量或算法意图：`int getitimer(int which, struct itimerval *value);`。
- **L1042**: Defines TableGen record `getitimer` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `getitimer`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1043**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1044**: Comment documents the nearby API, invariant, or algorithmic intent: `int getlogin_r(char *name, size_t namesize);`. / 这行注释说明了附近 API、不变量或算法意图：`int getlogin_r(char *name, size_t namesize);`。
- **L1045**: Defines TableGen record `getlogin_r` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `getlogin_r`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1046**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1047**: Comment documents the nearby API, invariant, or algorithmic intent: `struct passwd *getpwnam(const char *name);`. / 这行注释说明了附近 API、不变量或算法意图：`struct passwd *getpwnam(const char *name);`。
- **L1048**: Defines TableGen record `getpwnam` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `getpwnam`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1049**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1050**: Comment documents the nearby API, invariant, or algorithmic intent: `char *gets(char *s);`. / 这行注释说明了附近 API、不变量或算法意图：`char *gets(char *s);`。
- **L1051**: Defines TableGen record `gets` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `gets`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1052**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1053**: Comment documents the nearby API, invariant, or algorithmic intent: `int gettimeofday(struct timeval *tp, void *tzp);`. / 这行注释说明了附近 API、不变量或算法意图：`int gettimeofday(struct timeval *tp, void *tzp);`。
- **L1054**: Defines TableGen record `gettimeofday` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `gettimeofday`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1055**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1056**: Comment documents the nearby API, invariant, or algorithmic intent: `uint32_t htonl(uint32_t hostlong);`. / 这行注释说明了附近 API、不变量或算法意图：`uint32_t htonl(uint32_t hostlong);`。
- **L1057**: Defines TableGen record `htonl` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `htonl`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1058**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1059**: Comment documents the nearby API, invariant, or algorithmic intent: `uint16_t htons(uint16_t hostshort);`. / 这行注释说明了附近 API、不变量或算法意图：`uint16_t htons(uint16_t hostshort);`。
- **L1060**: Defines TableGen record `htons` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `htons`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1061**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1062**: Comment documents the nearby API, invariant, or algorithmic intent: `double hypot(double x, double y);`. / 这行注释说明了附近 API、不变量或算法意图：`double hypot(double x, double y);`。
- **L1063**: Defines TableGen record `hypot` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `hypot`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1064**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。

### Lines 1065-1092

```tablegen
/// float hypotf(float x, float y);
def hypotf : TargetLibCall<"hypotf", Flt, [Flt, Flt]>;

/// long double hypotl(long double x, long double y);
def hypotl : TargetLibCall<"hypotl", LDbl, [LDbl, LDbl]>;

/// int iprintf(const char *format, ...);
def iprintf : TargetLibCall<"iprintf", Int, [Ptr, Ellip]>;

/// int isascii(int c);
def isascii : TargetLibCall<"isascii", Int, [Int]>;

/// int isdigit(int c);
def isdigit : TargetLibCall<"isdigit", Int, [Int]>;

/// long int labs(long int j);
def labs : TargetLibCall<"labs", Long, [Same]>;

/// int lchown(const char *path, uid_t owner, gid_t group);
def lchown : TargetLibCall<"lchown", Int, [Ptr, IntX, IntX]>;

/// double ldexp(double x, int n);
def ldexp : TargetLibCall<"ldexp", Dbl, [Dbl, Int]>;

/// float ldexpf(float x, int n);
def ldexpf : TargetLibCall<"ldexpf", Flt, [Flt, Int]>;

/// long double ldexpl(long double x, int n);
```

- **L1065**: Comment documents the nearby API, invariant, or algorithmic intent: `float hypotf(float x, float y);`. / 这行注释说明了附近 API、不变量或算法意图：`float hypotf(float x, float y);`。
- **L1066**: Defines TableGen record `hypotf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `hypotf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1067**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1068**: Comment documents the nearby API, invariant, or algorithmic intent: `long double hypotl(long double x, long double y);`. / 这行注释说明了附近 API、不变量或算法意图：`long double hypotl(long double x, long double y);`。
- **L1069**: Defines TableGen record `hypotl` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `hypotl`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1070**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1071**: Comment documents the nearby API, invariant, or algorithmic intent: `int iprintf(const char *format, ...);`. / 这行注释说明了附近 API、不变量或算法意图：`int iprintf(const char *format, ...);`。
- **L1072**: Defines TableGen record `iprintf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `iprintf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1073**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1074**: Comment documents the nearby API, invariant, or algorithmic intent: `int isascii(int c);`. / 这行注释说明了附近 API、不变量或算法意图：`int isascii(int c);`。
- **L1075**: Defines TableGen record `isascii` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `isascii`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1076**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1077**: Comment documents the nearby API, invariant, or algorithmic intent: `int isdigit(int c);`. / 这行注释说明了附近 API、不变量或算法意图：`int isdigit(int c);`。
- **L1078**: Defines TableGen record `isdigit` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `isdigit`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1079**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1080**: Comment documents the nearby API, invariant, or algorithmic intent: `long int labs(long int j);`. / 这行注释说明了附近 API、不变量或算法意图：`long int labs(long int j);`。
- **L1081**: Defines TableGen record `labs` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `labs`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1082**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1083**: Comment documents the nearby API, invariant, or algorithmic intent: `int lchown(const char *path, uid_t owner, gid_t group);`. / 这行注释说明了附近 API、不变量或算法意图：`int lchown(const char *path, uid_t owner, gid_t group);`。
- **L1084**: Defines TableGen record `lchown` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `lchown`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1085**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1086**: Comment documents the nearby API, invariant, or algorithmic intent: `double ldexp(double x, int n);`. / 这行注释说明了附近 API、不变量或算法意图：`double ldexp(double x, int n);`。
- **L1087**: Defines TableGen record `ldexp` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `ldexp`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1088**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1089**: Comment documents the nearby API, invariant, or algorithmic intent: `float ldexpf(float x, int n);`. / 这行注释说明了附近 API、不变量或算法意图：`float ldexpf(float x, int n);`。
- **L1090**: Defines TableGen record `ldexpf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `ldexpf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1091**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1092**: Comment documents the nearby API, invariant, or algorithmic intent: `long double ldexpl(long double x, int n);`. / 这行注释说明了附近 API、不变量或算法意图：`long double ldexpl(long double x, int n);`。

### Lines 1093-1120

```tablegen
def ldexpl : TargetLibCall<"ldexpl", LDbl, [LDbl, Int]>;

/// long long int llabs(long long int j);
def llabs : TargetLibCall<"llabs", LLong, [LLong]>;

/// double log(double x);
def log : TargetLibCall<"log", Dbl, [Dbl]>;

/// double log10(double x);
def log10 : TargetLibCall<"log10", Dbl, [Dbl]>;

/// float log10f(float x);
def log10f : TargetLibCall<"log10f", Flt, [Flt]>;

/// long double log10l(long double x);
def log10l : TargetLibCall<"log10l", LDbl, [LDbl]>;

/// double log1p(double x);
def log1p : TargetLibCall<"log1p", Dbl, [Dbl]>;

/// float log1pf(float x);
def log1pf : TargetLibCall<"log1pf", Flt, [Flt]>;

/// long double log1pl(long double x);
def log1pl : TargetLibCall<"log1pl", LDbl, [LDbl]>;

/// double log2(double x);
def log2 : TargetLibCall<"log2", Dbl, [Dbl]>;
```

- **L1093**: Defines TableGen record `ldexpl` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `ldexpl`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1094**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1095**: Comment documents the nearby API, invariant, or algorithmic intent: `long long int llabs(long long int j);`. / 这行注释说明了附近 API、不变量或算法意图：`long long int llabs(long long int j);`。
- **L1096**: Defines TableGen record `llabs` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `llabs`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1097**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1098**: Comment documents the nearby API, invariant, or algorithmic intent: `double log(double x);`. / 这行注释说明了附近 API、不变量或算法意图：`double log(double x);`。
- **L1099**: Defines TableGen record `log` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `log`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1100**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1101**: Comment documents the nearby API, invariant, or algorithmic intent: `double log10(double x);`. / 这行注释说明了附近 API、不变量或算法意图：`double log10(double x);`。
- **L1102**: Defines TableGen record `log10` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `log10`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1103**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1104**: Comment documents the nearby API, invariant, or algorithmic intent: `float log10f(float x);`. / 这行注释说明了附近 API、不变量或算法意图：`float log10f(float x);`。
- **L1105**: Defines TableGen record `log10f` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `log10f`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1106**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1107**: Comment documents the nearby API, invariant, or algorithmic intent: `long double log10l(long double x);`. / 这行注释说明了附近 API、不变量或算法意图：`long double log10l(long double x);`。
- **L1108**: Defines TableGen record `log10l` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `log10l`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1109**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1110**: Comment documents the nearby API, invariant, or algorithmic intent: `double log1p(double x);`. / 这行注释说明了附近 API、不变量或算法意图：`double log1p(double x);`。
- **L1111**: Defines TableGen record `log1p` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `log1p`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1112**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1113**: Comment documents the nearby API, invariant, or algorithmic intent: `float log1pf(float x);`. / 这行注释说明了附近 API、不变量或算法意图：`float log1pf(float x);`。
- **L1114**: Defines TableGen record `log1pf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `log1pf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1115**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1116**: Comment documents the nearby API, invariant, or algorithmic intent: `long double log1pl(long double x);`. / 这行注释说明了附近 API、不变量或算法意图：`long double log1pl(long double x);`。
- **L1117**: Defines TableGen record `log1pl` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `log1pl`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1118**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1119**: Comment documents the nearby API, invariant, or algorithmic intent: `double log2(double x);`. / 这行注释说明了附近 API、不变量或算法意图：`double log2(double x);`。
- **L1120**: Defines TableGen record `log2` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `log2`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。

### Lines 1121-1148

```tablegen

/// float log2f(float x);
def log2f : TargetLibCall<"log2f", Flt, [Flt]>;

/// double long double log2l(long double x);
def log2l : TargetLibCall<"log2l", LDbl, [LDbl]>;

/// int ilogb(double x);
def ilogb : TargetLibCall<"ilogb", Int, [Dbl]>;

/// int ilogbf(float x);
def ilogbf : TargetLibCall<"ilogbf", Int, [Flt]>;

/// int ilogbl(long double x);
def ilogbl : TargetLibCall<"ilogbl", Int, [LDbl]>;

/// double logb(double x);
def logb : TargetLibCall<"logb", Dbl, [Dbl]>;

/// float logbf(float x);
def logbf : TargetLibCall<"logbf", Flt, [Flt]>;

/// long double logbl(long double x);
def logbl : TargetLibCall<"logbl", LDbl, [LDbl]>;

/// float logf(float x);
def logf : TargetLibCall<"logf", Flt, [Flt]>;

```

- **L1121**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1122**: Comment documents the nearby API, invariant, or algorithmic intent: `float log2f(float x);`. / 这行注释说明了附近 API、不变量或算法意图：`float log2f(float x);`。
- **L1123**: Defines TableGen record `log2f` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `log2f`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1124**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1125**: Comment documents the nearby API, invariant, or algorithmic intent: `double long double log2l(long double x);`. / 这行注释说明了附近 API、不变量或算法意图：`double long double log2l(long double x);`。
- **L1126**: Defines TableGen record `log2l` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `log2l`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1127**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1128**: Comment documents the nearby API, invariant, or algorithmic intent: `int ilogb(double x);`. / 这行注释说明了附近 API、不变量或算法意图：`int ilogb(double x);`。
- **L1129**: Defines TableGen record `ilogb` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `ilogb`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1130**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1131**: Comment documents the nearby API, invariant, or algorithmic intent: `int ilogbf(float x);`. / 这行注释说明了附近 API、不变量或算法意图：`int ilogbf(float x);`。
- **L1132**: Defines TableGen record `ilogbf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `ilogbf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1133**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1134**: Comment documents the nearby API, invariant, or algorithmic intent: `int ilogbl(long double x);`. / 这行注释说明了附近 API、不变量或算法意图：`int ilogbl(long double x);`。
- **L1135**: Defines TableGen record `ilogbl` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `ilogbl`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1136**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1137**: Comment documents the nearby API, invariant, or algorithmic intent: `double logb(double x);`. / 这行注释说明了附近 API、不变量或算法意图：`double logb(double x);`。
- **L1138**: Defines TableGen record `logb` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `logb`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1139**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1140**: Comment documents the nearby API, invariant, or algorithmic intent: `float logbf(float x);`. / 这行注释说明了附近 API、不变量或算法意图：`float logbf(float x);`。
- **L1141**: Defines TableGen record `logbf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `logbf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1142**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1143**: Comment documents the nearby API, invariant, or algorithmic intent: `long double logbl(long double x);`. / 这行注释说明了附近 API、不变量或算法意图：`long double logbl(long double x);`。
- **L1144**: Defines TableGen record `logbl` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `logbl`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1145**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1146**: Comment documents the nearby API, invariant, or algorithmic intent: `float logf(float x);`. / 这行注释说明了附近 API、不变量或算法意图：`float logf(float x);`。
- **L1147**: Defines TableGen record `logf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `logf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1148**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。

### Lines 1149-1176

```tablegen
/// long double logl(long double x);
def logl : TargetLibCall<"logl", LDbl, [LDbl]>;

/// int lstat(const char *path, struct stat *buf);
def lstat : TargetLibCall<"lstat", Int, [Ptr, Ptr]>;

/// int lstat64(const char *path, struct stat64 *buf);
def lstat64 : TargetLibCall<"lstat64", Int, [Ptr, Ptr]>;

/// void *malloc(size_t size);
def malloc : TargetLibCall<"malloc", Ptr, [SizeT]>;

/// void *memalign(size_t boundary, size_t size);
def memalign : TargetLibCall<"memalign", Ptr, [SizeT, SizeT]>;

/// void *memccpy(void *s1, const void *s2, int c, size_t n);
def memccpy : TargetLibCall<"memccpy", Ptr, [Ptr, Ptr, Int, SizeT]>;

/// void *memchr(const void *s, int c, size_t n);
def memchr : TargetLibCall<"memchr", Ptr, [Ptr, Int, SizeT]>;

/// int memcmp(const void *s1, const void *s2, size_t n);
def memcmp : TargetLibCall<"memcmp", Int, [Ptr, Ptr, SizeT]>;

/// void *memcpy(void *s1, const void *s2, size_t n);
def memcpy : TargetLibCall<"memcpy", Ptr, [Ptr, Ptr, SizeT]>;

/// void *memmove(void *s1, const void *s2, size_t n);
```

- **L1149**: Comment documents the nearby API, invariant, or algorithmic intent: `long double logl(long double x);`. / 这行注释说明了附近 API、不变量或算法意图：`long double logl(long double x);`。
- **L1150**: Defines TableGen record `logl` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `logl`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1151**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1152**: Comment documents the nearby API, invariant, or algorithmic intent: `int lstat(const char *path, struct stat *buf);`. / 这行注释说明了附近 API、不变量或算法意图：`int lstat(const char *path, struct stat *buf);`。
- **L1153**: Defines TableGen record `lstat` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `lstat`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1154**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1155**: Comment documents the nearby API, invariant, or algorithmic intent: `int lstat64(const char *path, struct stat64 *buf);`. / 这行注释说明了附近 API、不变量或算法意图：`int lstat64(const char *path, struct stat64 *buf);`。
- **L1156**: Defines TableGen record `lstat64` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `lstat64`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1157**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1158**: Comment documents the nearby API, invariant, or algorithmic intent: `void *malloc(size_t size);`. / 这行注释说明了附近 API、不变量或算法意图：`void *malloc(size_t size);`。
- **L1159**: Defines TableGen record `malloc` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `malloc`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1160**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1161**: Comment documents the nearby API, invariant, or algorithmic intent: `void *memalign(size_t boundary, size_t size);`. / 这行注释说明了附近 API、不变量或算法意图：`void *memalign(size_t boundary, size_t size);`。
- **L1162**: Defines TableGen record `memalign` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `memalign`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1163**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1164**: Comment documents the nearby API, invariant, or algorithmic intent: `void *memccpy(void *s1, const void *s2, int c, size_t n);`. / 这行注释说明了附近 API、不变量或算法意图：`void *memccpy(void *s1, const void *s2, int c, size_t n);`。
- **L1165**: Defines TableGen record `memccpy` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `memccpy`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1166**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1167**: Comment documents the nearby API, invariant, or algorithmic intent: `void *memchr(const void *s, int c, size_t n);`. / 这行注释说明了附近 API、不变量或算法意图：`void *memchr(const void *s, int c, size_t n);`。
- **L1168**: Defines TableGen record `memchr` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `memchr`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1169**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1170**: Comment documents the nearby API, invariant, or algorithmic intent: `int memcmp(const void *s1, const void *s2, size_t n);`. / 这行注释说明了附近 API、不变量或算法意图：`int memcmp(const void *s1, const void *s2, size_t n);`。
- **L1171**: Defines TableGen record `memcmp` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `memcmp`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1172**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1173**: Comment documents the nearby API, invariant, or algorithmic intent: `void *memcpy(void *s1, const void *s2, size_t n);`. / 这行注释说明了附近 API、不变量或算法意图：`void *memcpy(void *s1, const void *s2, size_t n);`。
- **L1174**: Defines TableGen record `memcpy` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `memcpy`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1175**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1176**: Comment documents the nearby API, invariant, or algorithmic intent: `void *memmove(void *s1, const void *s2, size_t n);`. / 这行注释说明了附近 API、不变量或算法意图：`void *memmove(void *s1, const void *s2, size_t n);`。

### Lines 1177-1204

```tablegen
def memmove : TargetLibCall<"memmove", Ptr, [Ptr, Ptr, SizeT]>;

/// void *mempcpy(void *s1, const void *s2, size_t n);
def mempcpy : TargetLibCall<"mempcpy", Ptr, [Ptr, Ptr, SizeT]>;

/// void *memrchr(const void *s, int c, size_t n);
def memrchr : TargetLibCall<"memrchr", Ptr, [Ptr, Int, SizeT]>;

/// void *memset(void *b, int c, size_t len);
def memset : TargetLibCall<"memset", Ptr, [Ptr, Int, SizeT]>;

/// void memset_pattern16(void *b, const void *pattern16, size_t len);
def memset_pattern16
    : TargetLibCall<"memset_pattern16", Void, [Ptr, Ptr, SizeT]>;

/// void memset_pattern4(void *b, const void *pattern4, size_t len);
def memset_pattern4 : TargetLibCall<"memset_pattern4", Void, [Ptr, Ptr, SizeT]>;

/// void memset_pattern8(void *b, const void *pattern8, size_t len);
def memset_pattern8 : TargetLibCall<"memset_pattern8", Void, [Ptr, Ptr, SizeT]>;

/// int mkdir(const char *path, mode_t mode);
def mkdir : TargetLibCall<"mkdir", Int, [Ptr, IntX]>;

/// time_t mktime(struct tm *timeptr);
def mktime : TargetLibCall<"mktime", IntPlus, [Ptr]>;

/// double modf(double x, double *iptr);
```

- **L1177**: Defines TableGen record `memmove` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `memmove`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1178**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1179**: Comment documents the nearby API, invariant, or algorithmic intent: `void *mempcpy(void *s1, const void *s2, size_t n);`. / 这行注释说明了附近 API、不变量或算法意图：`void *mempcpy(void *s1, const void *s2, size_t n);`。
- **L1180**: Defines TableGen record `mempcpy` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `mempcpy`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1181**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1182**: Comment documents the nearby API, invariant, or algorithmic intent: `void *memrchr(const void *s, int c, size_t n);`. / 这行注释说明了附近 API、不变量或算法意图：`void *memrchr(const void *s, int c, size_t n);`。
- **L1183**: Defines TableGen record `memrchr` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `memrchr`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1184**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1185**: Comment documents the nearby API, invariant, or algorithmic intent: `void *memset(void *b, int c, size_t len);`. / 这行注释说明了附近 API、不变量或算法意图：`void *memset(void *b, int c, size_t len);`。
- **L1186**: Defines TableGen record `memset` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `memset`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1187**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1188**: Comment documents the nearby API, invariant, or algorithmic intent: `void memset_pattern16(void *b, const void *pattern16, size_t len);`. / 这行注释说明了附近 API、不变量或算法意图：`void memset_pattern16(void *b, const void *pattern16, size_t len);`。
- **L1189**: Defines TableGen record `memset_pattern16`, adding one entry to the generated description database. / 定义 TableGen 记录 `memset_pattern16`，向生成用描述数据库中加入一个条目。
- **L1190**: Provides a declaration or standalone statement that contributes to the surrounding API or implementation. / 给出一条声明或独立语句，为周围的 API 或实现提供支撑。
- **L1191**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1192**: Comment documents the nearby API, invariant, or algorithmic intent: `void memset_pattern4(void *b, const void *pattern4, size_t len);`. / 这行注释说明了附近 API、不变量或算法意图：`void memset_pattern4(void *b, const void *pattern4, size_t len);`。
- **L1193**: Defines TableGen record `memset_pattern4` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `memset_pattern4`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1194**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1195**: Comment documents the nearby API, invariant, or algorithmic intent: `void memset_pattern8(void *b, const void *pattern8, size_t len);`. / 这行注释说明了附近 API、不变量或算法意图：`void memset_pattern8(void *b, const void *pattern8, size_t len);`。
- **L1196**: Defines TableGen record `memset_pattern8` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `memset_pattern8`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1197**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1198**: Comment documents the nearby API, invariant, or algorithmic intent: `int mkdir(const char *path, mode_t mode);`. / 这行注释说明了附近 API、不变量或算法意图：`int mkdir(const char *path, mode_t mode);`。
- **L1199**: Defines TableGen record `mkdir` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `mkdir`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1200**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1201**: Comment documents the nearby API, invariant, or algorithmic intent: `time_t mktime(struct tm *timeptr);`. / 这行注释说明了附近 API、不变量或算法意图：`time_t mktime(struct tm *timeptr);`。
- **L1202**: Defines TableGen record `mktime` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `mktime`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1203**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1204**: Comment documents the nearby API, invariant, or algorithmic intent: `double modf(double x, double *iptr);`. / 这行注释说明了附近 API、不变量或算法意图：`double modf(double x, double *iptr);`。

### Lines 1205-1232

```tablegen
def modf : TargetLibCall<"modf", Dbl, [Dbl, Ptr]>;

/// float modff(float, float *iptr);
def modff : TargetLibCall<"modff", Flt, [Flt, Ptr]>;

/// long double modfl(long double value, long double *iptr);
def modfl : TargetLibCall<"modfl", LDbl, [LDbl, Ptr]>;

/// double nan(const char *arg);
def nan : TargetLibCall<"nan", Dbl, [Ptr]>;

/// float nanf(const char *arg);
def nanf : TargetLibCall<"nanf", Flt, [Ptr]>;

/// long double nanl(const char *arg);
def nanl : TargetLibCall<"nanl", LDbl, [Ptr]>;

/// double nearbyint(double x);
def nearbyint : TargetLibCall<"nearbyint", Dbl, [Dbl]>;

/// float nearbyintf(float x);
def nearbyintf : TargetLibCall<"nearbyintf", Flt, [Flt]>;

/// long double nearbyintl(long double x);
def nearbyintl : TargetLibCall<"nearbyintl", LDbl, [LDbl]>;

/// double nextafter(double x, double y);
def nextafter : TargetLibCall< "nextafter", Dbl, [Dbl, Dbl]>;
```

- **L1205**: Defines TableGen record `modf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `modf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1206**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1207**: Comment documents the nearby API, invariant, or algorithmic intent: `float modff(float, float *iptr);`. / 这行注释说明了附近 API、不变量或算法意图：`float modff(float, float *iptr);`。
- **L1208**: Defines TableGen record `modff` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `modff`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1209**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1210**: Comment documents the nearby API, invariant, or algorithmic intent: `long double modfl(long double value, long double *iptr);`. / 这行注释说明了附近 API、不变量或算法意图：`long double modfl(long double value, long double *iptr);`。
- **L1211**: Defines TableGen record `modfl` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `modfl`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1212**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1213**: Comment documents the nearby API, invariant, or algorithmic intent: `double nan(const char *arg);`. / 这行注释说明了附近 API、不变量或算法意图：`double nan(const char *arg);`。
- **L1214**: Defines TableGen record `nan` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `nan`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1215**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1216**: Comment documents the nearby API, invariant, or algorithmic intent: `float nanf(const char *arg);`. / 这行注释说明了附近 API、不变量或算法意图：`float nanf(const char *arg);`。
- **L1217**: Defines TableGen record `nanf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `nanf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1218**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1219**: Comment documents the nearby API, invariant, or algorithmic intent: `long double nanl(const char *arg);`. / 这行注释说明了附近 API、不变量或算法意图：`long double nanl(const char *arg);`。
- **L1220**: Defines TableGen record `nanl` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `nanl`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1221**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1222**: Comment documents the nearby API, invariant, or algorithmic intent: `double nearbyint(double x);`. / 这行注释说明了附近 API、不变量或算法意图：`double nearbyint(double x);`。
- **L1223**: Defines TableGen record `nearbyint` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `nearbyint`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1224**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1225**: Comment documents the nearby API, invariant, or algorithmic intent: `float nearbyintf(float x);`. / 这行注释说明了附近 API、不变量或算法意图：`float nearbyintf(float x);`。
- **L1226**: Defines TableGen record `nearbyintf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `nearbyintf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1227**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1228**: Comment documents the nearby API, invariant, or algorithmic intent: `long double nearbyintl(long double x);`. / 这行注释说明了附近 API、不变量或算法意图：`long double nearbyintl(long double x);`。
- **L1229**: Defines TableGen record `nearbyintl` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `nearbyintl`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1230**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1231**: Comment documents the nearby API, invariant, or algorithmic intent: `double nextafter(double x, double y);`. / 这行注释说明了附近 API、不变量或算法意图：`double nextafter(double x, double y);`。
- **L1232**: Defines TableGen record `nextafter` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `nextafter`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。

### Lines 1233-1260

```tablegen

/// float nextafterf(float x, float y);
def nextafterf : TargetLibCall< "nextafterf", Flt, [Flt, Flt]>;

/// long double nextafterl(long double x, long double y);
def nextafterl : TargetLibCall< "nextafterl", LDbl, [LDbl, LDbl]>;

/// double nexttoward(double x, long double y);
def nexttoward : TargetLibCall< "nexttoward", Dbl, [Dbl, LDbl]>;

/// float nexttowardf(float x, long double y);
def nexttowardf : TargetLibCall< "nexttowardf", Flt, [Flt, LDbl]>;

/// long double nexttowardl(long double x, long double y);
def nexttowardl : TargetLibCall< "nexttowardl", LDbl, [LDbl, LDbl]>;

/// uint32_t ntohl(uint32_t netlong);
def ntohl : TargetLibCall<"ntohl", Int32, [Int32]>;

/// uint16_t ntohs(uint16_t netshort);
def ntohs : TargetLibCall<"ntohs", Int16, [Int16]>;

/// int open(const char *path, int oflag, ... );
def open : TargetLibCall<"open", Int, [Ptr, Int, Ellip]>;

/// int open64(const char *filename, int flags, [mode_t mode])
def open64 : TargetLibCall<"open64", Int, [Ptr, Int, Ellip]>;

```

- **L1233**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1234**: Comment documents the nearby API, invariant, or algorithmic intent: `float nextafterf(float x, float y);`. / 这行注释说明了附近 API、不变量或算法意图：`float nextafterf(float x, float y);`。
- **L1235**: Defines TableGen record `nextafterf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `nextafterf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1236**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1237**: Comment documents the nearby API, invariant, or algorithmic intent: `long double nextafterl(long double x, long double y);`. / 这行注释说明了附近 API、不变量或算法意图：`long double nextafterl(long double x, long double y);`。
- **L1238**: Defines TableGen record `nextafterl` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `nextafterl`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1239**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1240**: Comment documents the nearby API, invariant, or algorithmic intent: `double nexttoward(double x, long double y);`. / 这行注释说明了附近 API、不变量或算法意图：`double nexttoward(double x, long double y);`。
- **L1241**: Defines TableGen record `nexttoward` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `nexttoward`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1242**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1243**: Comment documents the nearby API, invariant, or algorithmic intent: `float nexttowardf(float x, long double y);`. / 这行注释说明了附近 API、不变量或算法意图：`float nexttowardf(float x, long double y);`。
- **L1244**: Defines TableGen record `nexttowardf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `nexttowardf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1245**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1246**: Comment documents the nearby API, invariant, or algorithmic intent: `long double nexttowardl(long double x, long double y);`. / 这行注释说明了附近 API、不变量或算法意图：`long double nexttowardl(long double x, long double y);`。
- **L1247**: Defines TableGen record `nexttowardl` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `nexttowardl`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1248**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1249**: Comment documents the nearby API, invariant, or algorithmic intent: `uint32_t ntohl(uint32_t netlong);`. / 这行注释说明了附近 API、不变量或算法意图：`uint32_t ntohl(uint32_t netlong);`。
- **L1250**: Defines TableGen record `ntohl` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `ntohl`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1251**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1252**: Comment documents the nearby API, invariant, or algorithmic intent: `uint16_t ntohs(uint16_t netshort);`. / 这行注释说明了附近 API、不变量或算法意图：`uint16_t ntohs(uint16_t netshort);`。
- **L1253**: Defines TableGen record `ntohs` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `ntohs`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1254**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1255**: Comment documents the nearby API, invariant, or algorithmic intent: `int open(const char *path, int oflag, ... );`. / 这行注释说明了附近 API、不变量或算法意图：`int open(const char *path, int oflag, ... );`。
- **L1256**: Defines TableGen record `open` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `open`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1257**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1258**: Comment documents the nearby API, invariant, or algorithmic intent: `int open64(const char *filename, int flags, [mode_t mode])`. / 这行注释说明了附近 API、不变量或算法意图：`int open64(const char *filename, int flags, [mode_t mode])`。
- **L1259**: Defines TableGen record `open64` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `open64`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1260**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。

### Lines 1261-1288

```tablegen
/// DIR *opendir(const char *dirname);
def opendir : TargetLibCall<"opendir", Ptr, [Ptr]>;

/// int pclose(FILE *stream);
def pclose : TargetLibCall<"pclose", Int, [Ptr]>;

/// void perror(const char *s);
def perror : TargetLibCall<"perror", Void, [Ptr]>;

/// FILE *popen(const char *command, const char *mode);
def popen : TargetLibCall<"popen", Ptr, [Ptr, Ptr]>;

/// int posix_memalign(void **memptr, size_t alignment, size_t size);
def posix_memalign : TargetLibCall<"posix_memalign", Int, [Ptr, SizeT, SizeT]>;

/// double pow(double x, double y);
def pow : TargetLibCall<"pow", Dbl, [Dbl, Dbl]>;

/// float powf(float x, float y);
def powf : TargetLibCall<"powf", Flt, [Flt, Flt]>;

/// long double powl(long double x, long double y);
def powl : TargetLibCall<"powl", LDbl, [LDbl, LDbl]>;

/// ssize_t pread(int fildes, void *buf, size_t nbyte, off_t offset);
def pread : TargetLibCall<"pread", SSizeT, [Int, Ptr, SizeT, IntPlus]>;

/// int printf(const char *format, ...);
```

- **L1261**: Comment documents the nearby API, invariant, or algorithmic intent: `DIR *opendir(const char *dirname);`. / 这行注释说明了附近 API、不变量或算法意图：`DIR *opendir(const char *dirname);`。
- **L1262**: Defines TableGen record `opendir` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `opendir`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1263**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1264**: Comment documents the nearby API, invariant, or algorithmic intent: `int pclose(FILE *stream);`. / 这行注释说明了附近 API、不变量或算法意图：`int pclose(FILE *stream);`。
- **L1265**: Defines TableGen record `pclose` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `pclose`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1266**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1267**: Comment documents the nearby API, invariant, or algorithmic intent: `void perror(const char *s);`. / 这行注释说明了附近 API、不变量或算法意图：`void perror(const char *s);`。
- **L1268**: Defines TableGen record `perror` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `perror`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1269**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1270**: Comment documents the nearby API, invariant, or algorithmic intent: `FILE *popen(const char *command, const char *mode);`. / 这行注释说明了附近 API、不变量或算法意图：`FILE *popen(const char *command, const char *mode);`。
- **L1271**: Defines TableGen record `popen` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `popen`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1272**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1273**: Comment documents the nearby API, invariant, or algorithmic intent: `int posix_memalign(void **memptr, size_t alignment, size_t size);`. / 这行注释说明了附近 API、不变量或算法意图：`int posix_memalign(void **memptr, size_t alignment, size_t size);`。
- **L1274**: Defines TableGen record `posix_memalign` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `posix_memalign`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1275**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1276**: Comment documents the nearby API, invariant, or algorithmic intent: `double pow(double x, double y);`. / 这行注释说明了附近 API、不变量或算法意图：`double pow(double x, double y);`。
- **L1277**: Defines TableGen record `pow` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `pow`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1278**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1279**: Comment documents the nearby API, invariant, or algorithmic intent: `float powf(float x, float y);`. / 这行注释说明了附近 API、不变量或算法意图：`float powf(float x, float y);`。
- **L1280**: Defines TableGen record `powf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `powf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1281**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1282**: Comment documents the nearby API, invariant, or algorithmic intent: `long double powl(long double x, long double y);`. / 这行注释说明了附近 API、不变量或算法意图：`long double powl(long double x, long double y);`。
- **L1283**: Defines TableGen record `powl` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `powl`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1284**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1285**: Comment documents the nearby API, invariant, or algorithmic intent: `ssize_t pread(int fildes, void *buf, size_t nbyte, off_t offset);`. / 这行注释说明了附近 API、不变量或算法意图：`ssize_t pread(int fildes, void *buf, size_t nbyte, off_t offset);`。
- **L1286**: Defines TableGen record `pread` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `pread`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1287**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1288**: Comment documents the nearby API, invariant, or algorithmic intent: `int printf(const char *format, ...);`. / 这行注释说明了附近 API、不变量或算法意图：`int printf(const char *format, ...);`。

### Lines 1289-1316

```tablegen
def printf : TargetLibCall<"printf", Int, [Ptr, Ellip]>;

/// int putc(int c, FILE *stream);
def putc : TargetLibCall<"putc", Int, [Int, Ptr]>;

/// int putc_unlocked(int c, FILE *stream);
def putc_unlocked : TargetLibCall<"putc_unlocked", Int, [Int, Ptr]>;

/// int putchar(int c);
def putchar : TargetLibCall<"putchar", Int, [Int]>;

/// int putchar_unlocked(int c);
def putchar_unlocked : TargetLibCall<"putchar_unlocked", Int, [Int]>;

/// int puts(const char *s);
def puts : TargetLibCall<"puts", Int, [Ptr]>;

/// void *pvalloc(size_t size);
def pvalloc : TargetLibCall<"pvalloc", Ptr, [SizeT]>;

/// ssize_t pwrite(int fildes, const void *buf, size_t nbyte, off_t offset);
def pwrite : TargetLibCall<"pwrite", SSizeT, [Int, Ptr, SizeT, IntPlus]>;

/// void qsort(void *base, size_t nel, size_t width,
///            int (*compar)(const void *, const void *));
def qsort : TargetLibCall<"qsort", Void, [Ptr, SizeT, SizeT, Ptr]>;

/// ssize_t read(int fildes, void *buf, size_t nbyte);
```

- **L1289**: Defines TableGen record `printf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `printf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1290**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1291**: Comment documents the nearby API, invariant, or algorithmic intent: `int putc(int c, FILE *stream);`. / 这行注释说明了附近 API、不变量或算法意图：`int putc(int c, FILE *stream);`。
- **L1292**: Defines TableGen record `putc` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `putc`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1293**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1294**: Comment documents the nearby API, invariant, or algorithmic intent: `int putc_unlocked(int c, FILE *stream);`. / 这行注释说明了附近 API、不变量或算法意图：`int putc_unlocked(int c, FILE *stream);`。
- **L1295**: Defines TableGen record `putc_unlocked` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `putc_unlocked`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1296**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1297**: Comment documents the nearby API, invariant, or algorithmic intent: `int putchar(int c);`. / 这行注释说明了附近 API、不变量或算法意图：`int putchar(int c);`。
- **L1298**: Defines TableGen record `putchar` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `putchar`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1299**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1300**: Comment documents the nearby API, invariant, or algorithmic intent: `int putchar_unlocked(int c);`. / 这行注释说明了附近 API、不变量或算法意图：`int putchar_unlocked(int c);`。
- **L1301**: Defines TableGen record `putchar_unlocked` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `putchar_unlocked`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1302**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1303**: Comment documents the nearby API, invariant, or algorithmic intent: `int puts(const char *s);`. / 这行注释说明了附近 API、不变量或算法意图：`int puts(const char *s);`。
- **L1304**: Defines TableGen record `puts` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `puts`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1305**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1306**: Comment documents the nearby API, invariant, or algorithmic intent: `void *pvalloc(size_t size);`. / 这行注释说明了附近 API、不变量或算法意图：`void *pvalloc(size_t size);`。
- **L1307**: Defines TableGen record `pvalloc` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `pvalloc`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1308**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1309**: Comment documents the nearby API, invariant, or algorithmic intent: `ssize_t pwrite(int fildes, const void *buf, size_t nbyte, off_t offset);`. / 这行注释说明了附近 API、不变量或算法意图：`ssize_t pwrite(int fildes, const void *buf, size_t nbyte, off_t offset);`。
- **L1310**: Defines TableGen record `pwrite` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `pwrite`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1311**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1312**: Comment documents the nearby API, invariant, or algorithmic intent: `void qsort(void *base, size_t nel, size_t width,`. / 这行注释说明了附近 API、不变量或算法意图：`void qsort(void *base, size_t nel, size_t width,`。
- **L1313**: Comment documents the nearby API, invariant, or algorithmic intent: `int (*compar)(const void *, const void *));`. / 这行注释说明了附近 API、不变量或算法意图：`int (*compar)(const void *, const void *));`。
- **L1314**: Defines TableGen record `qsort` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `qsort`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1315**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1316**: Comment documents the nearby API, invariant, or algorithmic intent: `ssize_t read(int fildes, void *buf, size_t nbyte);`. / 这行注释说明了附近 API、不变量或算法意图：`ssize_t read(int fildes, void *buf, size_t nbyte);`。

### Lines 1317-1344

```tablegen
def read : TargetLibCall<"read", SSizeT, [Int, Ptr, SizeT]>;

/// ssize_t readlink(const char *path, char *buf, size_t bufsize);
def readlink : TargetLibCall<"readlink", SSizeT, [Ptr, Ptr, SizeT]>;

/// void *realloc(void *ptr, size_t size);
def realloc : TargetLibCall<"realloc", Ptr, [Ptr, SizeT]>;

/// void *reallocf(void *ptr, size_t size);
def reallocf : TargetLibCall<"reallocf", Ptr, [Ptr, SizeT]>;

/// void *reallocarray(void *ptr, size_t nmemb, size_t size);
def reallocarray : TargetLibCall<"reallocarray", Ptr, [Ptr, SizeT, SizeT]>;

/// char *realpath(const char *file_name, char *resolved_name);
def realpath : TargetLibCall<"realpath", Ptr, [Ptr, Ptr]>;

/// double remainder(double x, double y);
def remainder : TargetLibCall<"remainder", Dbl, [Dbl, Dbl]>;

/// float remainderf(float x, float y);
def remainderf : TargetLibCall<"remainderf", Flt, [Flt, Flt]>;

/// long double remainderl(long double x, long double y);
def remainderl : TargetLibCall<"remainderl", LDbl, [LDbl, LDbl]>;

/// double remquo(double x, double y, int *quo);
def remquo : TargetLibCall<"remquo", Dbl, [Dbl, Dbl, Ptr]>;
```

- **L1317**: Defines TableGen record `read` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `read`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1318**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1319**: Comment documents the nearby API, invariant, or algorithmic intent: `ssize_t readlink(const char *path, char *buf, size_t bufsize);`. / 这行注释说明了附近 API、不变量或算法意图：`ssize_t readlink(const char *path, char *buf, size_t bufsize);`。
- **L1320**: Defines TableGen record `readlink` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `readlink`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1321**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1322**: Comment documents the nearby API, invariant, or algorithmic intent: `void *realloc(void *ptr, size_t size);`. / 这行注释说明了附近 API、不变量或算法意图：`void *realloc(void *ptr, size_t size);`。
- **L1323**: Defines TableGen record `realloc` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `realloc`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1324**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1325**: Comment documents the nearby API, invariant, or algorithmic intent: `void *reallocf(void *ptr, size_t size);`. / 这行注释说明了附近 API、不变量或算法意图：`void *reallocf(void *ptr, size_t size);`。
- **L1326**: Defines TableGen record `reallocf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `reallocf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1327**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1328**: Comment documents the nearby API, invariant, or algorithmic intent: `void *reallocarray(void *ptr, size_t nmemb, size_t size);`. / 这行注释说明了附近 API、不变量或算法意图：`void *reallocarray(void *ptr, size_t nmemb, size_t size);`。
- **L1329**: Defines TableGen record `reallocarray` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `reallocarray`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1330**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1331**: Comment documents the nearby API, invariant, or algorithmic intent: `char *realpath(const char *file_name, char *resolved_name);`. / 这行注释说明了附近 API、不变量或算法意图：`char *realpath(const char *file_name, char *resolved_name);`。
- **L1332**: Defines TableGen record `realpath` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `realpath`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1333**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1334**: Comment documents the nearby API, invariant, or algorithmic intent: `double remainder(double x, double y);`. / 这行注释说明了附近 API、不变量或算法意图：`double remainder(double x, double y);`。
- **L1335**: Defines TableGen record `remainder` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `remainder`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1336**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1337**: Comment documents the nearby API, invariant, or algorithmic intent: `float remainderf(float x, float y);`. / 这行注释说明了附近 API、不变量或算法意图：`float remainderf(float x, float y);`。
- **L1338**: Defines TableGen record `remainderf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `remainderf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1339**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1340**: Comment documents the nearby API, invariant, or algorithmic intent: `long double remainderl(long double x, long double y);`. / 这行注释说明了附近 API、不变量或算法意图：`long double remainderl(long double x, long double y);`。
- **L1341**: Defines TableGen record `remainderl` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `remainderl`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1342**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1343**: Comment documents the nearby API, invariant, or algorithmic intent: `double remquo(double x, double y, int *quo);`. / 这行注释说明了附近 API、不变量或算法意图：`double remquo(double x, double y, int *quo);`。
- **L1344**: Defines TableGen record `remquo` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `remquo`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。

### Lines 1345-1372

```tablegen

/// float remquof(float x, float y, int *quo);
def remquof : TargetLibCall<"remquof", Flt, [Flt, Flt, Ptr]>;

/// long double remquol(long double x, long double y, int *quo);
def remquol : TargetLibCall<"remquol", LDbl, [LDbl, LDbl, Ptr]>;

/// double fdim(double x, double y);
def fdim : TargetLibCall<"fdim", Dbl, [Dbl, Dbl]>;

/// float fdimf(float x, float y);
def fdimf : TargetLibCall<"fdimf", Flt, [Flt, Flt]>;

/// long double fdiml(long double x, long double y);
def fdiml : TargetLibCall<"fdiml", LDbl, [LDbl, LDbl]>;

/// int remove(const char *path);
def remove : TargetLibCall<"remove", Int, [Ptr]>;

/// int rename(const char *old, const char *new);
def rename : TargetLibCall<"rename", Int, [Ptr, Ptr]>;

/// void rewind(FILE *stream);
def rewind : TargetLibCall<"rewind", Void, [Ptr]>;

/// double rint(double x);
def rint : TargetLibCall<"rint", Dbl, [Dbl]>;

```

- **L1345**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1346**: Comment documents the nearby API, invariant, or algorithmic intent: `float remquof(float x, float y, int *quo);`. / 这行注释说明了附近 API、不变量或算法意图：`float remquof(float x, float y, int *quo);`。
- **L1347**: Defines TableGen record `remquof` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `remquof`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1348**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1349**: Comment documents the nearby API, invariant, or algorithmic intent: `long double remquol(long double x, long double y, int *quo);`. / 这行注释说明了附近 API、不变量或算法意图：`long double remquol(long double x, long double y, int *quo);`。
- **L1350**: Defines TableGen record `remquol` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `remquol`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1351**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1352**: Comment documents the nearby API, invariant, or algorithmic intent: `double fdim(double x, double y);`. / 这行注释说明了附近 API、不变量或算法意图：`double fdim(double x, double y);`。
- **L1353**: Defines TableGen record `fdim` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `fdim`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1354**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1355**: Comment documents the nearby API, invariant, or algorithmic intent: `float fdimf(float x, float y);`. / 这行注释说明了附近 API、不变量或算法意图：`float fdimf(float x, float y);`。
- **L1356**: Defines TableGen record `fdimf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `fdimf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1357**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1358**: Comment documents the nearby API, invariant, or algorithmic intent: `long double fdiml(long double x, long double y);`. / 这行注释说明了附近 API、不变量或算法意图：`long double fdiml(long double x, long double y);`。
- **L1359**: Defines TableGen record `fdiml` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `fdiml`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1360**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1361**: Comment documents the nearby API, invariant, or algorithmic intent: `int remove(const char *path);`. / 这行注释说明了附近 API、不变量或算法意图：`int remove(const char *path);`。
- **L1362**: Defines TableGen record `remove` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `remove`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1363**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1364**: Comment documents the nearby API, invariant, or algorithmic intent: `int rename(const char *old, const char *new);`. / 这行注释说明了附近 API、不变量或算法意图：`int rename(const char *old, const char *new);`。
- **L1365**: Defines TableGen record `rename` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `rename`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1366**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1367**: Comment documents the nearby API, invariant, or algorithmic intent: `void rewind(FILE *stream);`. / 这行注释说明了附近 API、不变量或算法意图：`void rewind(FILE *stream);`。
- **L1368**: Defines TableGen record `rewind` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `rewind`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1369**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1370**: Comment documents the nearby API, invariant, or algorithmic intent: `double rint(double x);`. / 这行注释说明了附近 API、不变量或算法意图：`double rint(double x);`。
- **L1371**: Defines TableGen record `rint` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `rint`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1372**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。

### Lines 1373-1400

```tablegen
/// float rintf(float x);
def rintf : TargetLibCall<"rintf", Flt, [Flt]>;

/// long double rintl(long double x);
def rintl : TargetLibCall<"rintl", LDbl, [LDbl]>;

/// int rmdir(const char *path);
def rmdir : TargetLibCall<"rmdir", Int, [Ptr]>;

/// double round(double x);
def round : TargetLibCall<"round", Dbl, [Dbl]>;

/// double roundeven(double x);
def roundeven : TargetLibCall<"roundeven", Dbl, [Dbl]>;

/// float roundevenf(float x);
def roundevenf : TargetLibCall<"roundevenf", Flt, [Flt]>;

/// long double roundevenl(long double x);
def roundevenl : TargetLibCall<"roundevenl", LDbl, [LDbl]>;

/// float roundf(float x);
def roundf : TargetLibCall<"roundf", Flt, [Flt]>;

/// long double roundl(long double x);
def roundl : TargetLibCall<"roundl", LDbl, [LDbl]>;

/// double scalbln(double arg, long exp);
```

- **L1373**: Comment documents the nearby API, invariant, or algorithmic intent: `float rintf(float x);`. / 这行注释说明了附近 API、不变量或算法意图：`float rintf(float x);`。
- **L1374**: Defines TableGen record `rintf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `rintf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1375**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1376**: Comment documents the nearby API, invariant, or algorithmic intent: `long double rintl(long double x);`. / 这行注释说明了附近 API、不变量或算法意图：`long double rintl(long double x);`。
- **L1377**: Defines TableGen record `rintl` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `rintl`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1378**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1379**: Comment documents the nearby API, invariant, or algorithmic intent: `int rmdir(const char *path);`. / 这行注释说明了附近 API、不变量或算法意图：`int rmdir(const char *path);`。
- **L1380**: Defines TableGen record `rmdir` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `rmdir`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1381**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1382**: Comment documents the nearby API, invariant, or algorithmic intent: `double round(double x);`. / 这行注释说明了附近 API、不变量或算法意图：`double round(double x);`。
- **L1383**: Defines TableGen record `round` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `round`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1384**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1385**: Comment documents the nearby API, invariant, or algorithmic intent: `double roundeven(double x);`. / 这行注释说明了附近 API、不变量或算法意图：`double roundeven(double x);`。
- **L1386**: Defines TableGen record `roundeven` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `roundeven`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1387**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1388**: Comment documents the nearby API, invariant, or algorithmic intent: `float roundevenf(float x);`. / 这行注释说明了附近 API、不变量或算法意图：`float roundevenf(float x);`。
- **L1389**: Defines TableGen record `roundevenf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `roundevenf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1390**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1391**: Comment documents the nearby API, invariant, or algorithmic intent: `long double roundevenl(long double x);`. / 这行注释说明了附近 API、不变量或算法意图：`long double roundevenl(long double x);`。
- **L1392**: Defines TableGen record `roundevenl` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `roundevenl`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1393**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1394**: Comment documents the nearby API, invariant, or algorithmic intent: `float roundf(float x);`. / 这行注释说明了附近 API、不变量或算法意图：`float roundf(float x);`。
- **L1395**: Defines TableGen record `roundf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `roundf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1396**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1397**: Comment documents the nearby API, invariant, or algorithmic intent: `long double roundl(long double x);`. / 这行注释说明了附近 API、不变量或算法意图：`long double roundl(long double x);`。
- **L1398**: Defines TableGen record `roundl` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `roundl`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1399**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1400**: Comment documents the nearby API, invariant, or algorithmic intent: `double scalbln(double arg, long exp);`. / 这行注释说明了附近 API、不变量或算法意图：`double scalbln(double arg, long exp);`。

### Lines 1401-1428

```tablegen
def scalbln : TargetLibCall<"scalbln", Dbl, [Dbl, Long]>;

/// float scalblnf(float arg, long exp);
def scalblnf : TargetLibCall<"scalblnf", Flt, [Flt, Long]>;

/// long double scalblnl(long double arg, long exp);
def scalblnl : TargetLibCall<"scalblnl", LDbl, [LDbl, Long]>;

/// double scalbn(double arg, int exp);
def scalbn : TargetLibCall<"scalbn", Dbl, [Dbl, Int]>;

/// float scalbnf(float arg, int exp);
def scalbnf : TargetLibCall<"scalbnf", Flt, [Flt, Int]>;

/// long double scalbnl(long double arg, int exp);
def scalbnl : TargetLibCall<"scalbnl", LDbl, [LDbl, Int]>;

/// int scanf(const char *restrict format, ... );
def scanf : TargetLibCall<"scanf", Int, [Ptr, Ellip]>;

/// void setbuf(FILE *stream, char *buf);
def setbuf : TargetLibCall<"setbuf", Void, [Ptr, Ptr]>;

/// int setitimer(int which, const struct itimerval *value,
/// struct itimerval *ovalue);
def setitimer : TargetLibCall<"setitimer", Int, [Int, Ptr, Ptr]>;

/// int setvbuf(FILE *stream, char *buf, int type, size_t size);
```

- **L1401**: Defines TableGen record `scalbln` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `scalbln`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1402**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1403**: Comment documents the nearby API, invariant, or algorithmic intent: `float scalblnf(float arg, long exp);`. / 这行注释说明了附近 API、不变量或算法意图：`float scalblnf(float arg, long exp);`。
- **L1404**: Defines TableGen record `scalblnf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `scalblnf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1405**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1406**: Comment documents the nearby API, invariant, or algorithmic intent: `long double scalblnl(long double arg, long exp);`. / 这行注释说明了附近 API、不变量或算法意图：`long double scalblnl(long double arg, long exp);`。
- **L1407**: Defines TableGen record `scalblnl` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `scalblnl`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1408**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1409**: Comment documents the nearby API, invariant, or algorithmic intent: `double scalbn(double arg, int exp);`. / 这行注释说明了附近 API、不变量或算法意图：`double scalbn(double arg, int exp);`。
- **L1410**: Defines TableGen record `scalbn` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `scalbn`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1411**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1412**: Comment documents the nearby API, invariant, or algorithmic intent: `float scalbnf(float arg, int exp);`. / 这行注释说明了附近 API、不变量或算法意图：`float scalbnf(float arg, int exp);`。
- **L1413**: Defines TableGen record `scalbnf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `scalbnf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1414**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1415**: Comment documents the nearby API, invariant, or algorithmic intent: `long double scalbnl(long double arg, int exp);`. / 这行注释说明了附近 API、不变量或算法意图：`long double scalbnl(long double arg, int exp);`。
- **L1416**: Defines TableGen record `scalbnl` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `scalbnl`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1417**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1418**: Comment documents the nearby API, invariant, or algorithmic intent: `int scanf(const char *restrict format, ... );`. / 这行注释说明了附近 API、不变量或算法意图：`int scanf(const char *restrict format, ... );`。
- **L1419**: Defines TableGen record `scanf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `scanf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1420**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1421**: Comment documents the nearby API, invariant, or algorithmic intent: `void setbuf(FILE *stream, char *buf);`. / 这行注释说明了附近 API、不变量或算法意图：`void setbuf(FILE *stream, char *buf);`。
- **L1422**: Defines TableGen record `setbuf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `setbuf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1423**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1424**: Comment documents the nearby API, invariant, or algorithmic intent: `int setitimer(int which, const struct itimerval *value,`. / 这行注释说明了附近 API、不变量或算法意图：`int setitimer(int which, const struct itimerval *value,`。
- **L1425**: Comment documents the nearby API, invariant, or algorithmic intent: `struct itimerval *ovalue);`. / 这行注释说明了附近 API、不变量或算法意图：`struct itimerval *ovalue);`。
- **L1426**: Defines TableGen record `setitimer` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `setitimer`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1427**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1428**: Comment documents the nearby API, invariant, or algorithmic intent: `int setvbuf(FILE *stream, char *buf, int type, size_t size);`. / 这行注释说明了附近 API、不变量或算法意图：`int setvbuf(FILE *stream, char *buf, int type, size_t size);`。

### Lines 1429-1456

```tablegen
def setvbuf : TargetLibCall<"setvbuf", Int, [Ptr, Ptr, Int, SizeT]>;

/// double sin(double x);
def sin : TargetLibCall<"sin", Dbl, [Dbl]>;

/// float sinf(float x);
def sinf : TargetLibCall<"sinf", Flt, [Flt]>;

/// double sinh(double x);
def sinh : TargetLibCall<"sinh", Dbl, [Dbl]>;

/// float sinhf(float x);
def sinhf : TargetLibCall<"sinhf", Flt, [Flt]>;

/// long double sinhl(long double x);
def sinhl : TargetLibCall<"sinhl", LDbl, [LDbl]>;

/// long double sinl(long double x);
def sinl : TargetLibCall<"sinl", LDbl, [LDbl]>;

/// void sincos(double x, double *sin_out, double *cos_out);
def sincos : TargetLibCall<"sincos", Void, [Dbl, Ptr, Ptr]>;

/// void sincosf(float x, float *sin_out, float *cos_out);
def sincosf : TargetLibCall<"sincosf", Void, [Flt, Ptr, Ptr]>;

/// void sincosl(long double x, long double *sin_out, long double *cos_out);
def sincosl : TargetLibCall<"sincosl", Void, [LDbl, Ptr, Ptr]>;
```

- **L1429**: Defines TableGen record `setvbuf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `setvbuf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1430**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1431**: Comment documents the nearby API, invariant, or algorithmic intent: `double sin(double x);`. / 这行注释说明了附近 API、不变量或算法意图：`double sin(double x);`。
- **L1432**: Defines TableGen record `sin` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `sin`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1433**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1434**: Comment documents the nearby API, invariant, or algorithmic intent: `float sinf(float x);`. / 这行注释说明了附近 API、不变量或算法意图：`float sinf(float x);`。
- **L1435**: Defines TableGen record `sinf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `sinf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1436**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1437**: Comment documents the nearby API, invariant, or algorithmic intent: `double sinh(double x);`. / 这行注释说明了附近 API、不变量或算法意图：`double sinh(double x);`。
- **L1438**: Defines TableGen record `sinh` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `sinh`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1439**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1440**: Comment documents the nearby API, invariant, or algorithmic intent: `float sinhf(float x);`. / 这行注释说明了附近 API、不变量或算法意图：`float sinhf(float x);`。
- **L1441**: Defines TableGen record `sinhf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `sinhf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1442**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1443**: Comment documents the nearby API, invariant, or algorithmic intent: `long double sinhl(long double x);`. / 这行注释说明了附近 API、不变量或算法意图：`long double sinhl(long double x);`。
- **L1444**: Defines TableGen record `sinhl` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `sinhl`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1445**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1446**: Comment documents the nearby API, invariant, or algorithmic intent: `long double sinl(long double x);`. / 这行注释说明了附近 API、不变量或算法意图：`long double sinl(long double x);`。
- **L1447**: Defines TableGen record `sinl` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `sinl`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1448**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1449**: Comment documents the nearby API, invariant, or algorithmic intent: `void sincos(double x, double *sin_out, double *cos_out);`. / 这行注释说明了附近 API、不变量或算法意图：`void sincos(double x, double *sin_out, double *cos_out);`。
- **L1450**: Defines TableGen record `sincos` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `sincos`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1451**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1452**: Comment documents the nearby API, invariant, or algorithmic intent: `void sincosf(float x, float *sin_out, float *cos_out);`. / 这行注释说明了附近 API、不变量或算法意图：`void sincosf(float x, float *sin_out, float *cos_out);`。
- **L1453**: Defines TableGen record `sincosf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `sincosf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1454**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1455**: Comment documents the nearby API, invariant, or algorithmic intent: `void sincosl(long double x, long double *sin_out, long double *cos_out);`. / 这行注释说明了附近 API、不变量或算法意图：`void sincosl(long double x, long double *sin_out, long double *cos_out);`。
- **L1456**: Defines TableGen record `sincosl` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `sincosl`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。

### Lines 1457-1484

```tablegen

/// int siprintf(char *str, const char *format, ...);
def siprintf : TargetLibCall<"siprintf", Int, [Ptr, Ptr, Ellip]>;

/// int snprintf(char *s, size_t n, const char *format, ...);
def snprintf : TargetLibCall<"snprintf", Int, [Ptr, SizeT, Ptr, Ellip]>;

/// int sprintf(char *str, const char *format, ...);
def sprintf : TargetLibCall<"sprintf", Int, [Ptr, Ptr, Ellip]>;

/// double sqrt(double x);
def sqrt : TargetLibCall<"sqrt", Dbl, [Dbl]>;

/// float sqrtf(float x);
def sqrtf : TargetLibCall<"sqrtf", Flt, [Flt]>;

/// long double sqrtl(long double x);
def sqrtl : TargetLibCall<"sqrtl", LDbl, [LDbl]>;

/// int sscanf(const char *s, const char *format, ... );
def sscanf : TargetLibCall<"sscanf", Int, [Ptr, Ptr, Ellip]>;

/// int stat(const char *path, struct stat *buf);
def stat : TargetLibCall<"stat", Int, [Ptr, Ptr]>;

/// int stat64(const char *path, struct stat64 *buf);
def stat64 : TargetLibCall<"stat64", Int, [Ptr, Ptr]>;

```

- **L1457**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1458**: Comment documents the nearby API, invariant, or algorithmic intent: `int siprintf(char *str, const char *format, ...);`. / 这行注释说明了附近 API、不变量或算法意图：`int siprintf(char *str, const char *format, ...);`。
- **L1459**: Defines TableGen record `siprintf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `siprintf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1460**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1461**: Comment documents the nearby API, invariant, or algorithmic intent: `int snprintf(char *s, size_t n, const char *format, ...);`. / 这行注释说明了附近 API、不变量或算法意图：`int snprintf(char *s, size_t n, const char *format, ...);`。
- **L1462**: Defines TableGen record `snprintf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `snprintf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1463**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1464**: Comment documents the nearby API, invariant, or algorithmic intent: `int sprintf(char *str, const char *format, ...);`. / 这行注释说明了附近 API、不变量或算法意图：`int sprintf(char *str, const char *format, ...);`。
- **L1465**: Defines TableGen record `sprintf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `sprintf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1466**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1467**: Comment documents the nearby API, invariant, or algorithmic intent: `double sqrt(double x);`. / 这行注释说明了附近 API、不变量或算法意图：`double sqrt(double x);`。
- **L1468**: Defines TableGen record `sqrt` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `sqrt`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1469**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1470**: Comment documents the nearby API, invariant, or algorithmic intent: `float sqrtf(float x);`. / 这行注释说明了附近 API、不变量或算法意图：`float sqrtf(float x);`。
- **L1471**: Defines TableGen record `sqrtf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `sqrtf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1472**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1473**: Comment documents the nearby API, invariant, or algorithmic intent: `long double sqrtl(long double x);`. / 这行注释说明了附近 API、不变量或算法意图：`long double sqrtl(long double x);`。
- **L1474**: Defines TableGen record `sqrtl` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `sqrtl`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1475**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1476**: Comment documents the nearby API, invariant, or algorithmic intent: `int sscanf(const char *s, const char *format, ... );`. / 这行注释说明了附近 API、不变量或算法意图：`int sscanf(const char *s, const char *format, ... );`。
- **L1477**: Defines TableGen record `sscanf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `sscanf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1478**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1479**: Comment documents the nearby API, invariant, or algorithmic intent: `int stat(const char *path, struct stat *buf);`. / 这行注释说明了附近 API、不变量或算法意图：`int stat(const char *path, struct stat *buf);`。
- **L1480**: Defines TableGen record `stat` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `stat`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1481**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1482**: Comment documents the nearby API, invariant, or algorithmic intent: `int stat64(const char *path, struct stat64 *buf);`. / 这行注释说明了附近 API、不变量或算法意图：`int stat64(const char *path, struct stat64 *buf);`。
- **L1483**: Defines TableGen record `stat64` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `stat64`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1484**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。

### Lines 1485-1512

```tablegen
/// int statvfs(const char *path, struct statvfs *buf);
def statvfs : TargetLibCall<"statvfs", Int, [Ptr, Ptr]>;

/// int statvfs64(const char *path, struct statvfs64 *buf)
def statvfs64 : TargetLibCall<"statvfs64", Int, [Ptr, Ptr]>;

/// char *stpcpy(char *s1, const char *s2);
def stpcpy : TargetLibCall<"stpcpy", Ptr, [Ptr, Ptr]>;

/// char *stpncpy(char *s1, const char *s2, size_t n);
def stpncpy : TargetLibCall<"stpncpy", Ptr, [Ptr, Ptr, SizeT]>;

/// int strcasecmp(const char *s1, const char *s2);
def strcasecmp : TargetLibCall<"strcasecmp", Int, [Ptr, Ptr]>;

/// char *strcat(char *s1, const char *s2);
def strcat : TargetLibCall<"strcat", Ptr, [Ptr, Ptr]>;

/// char *strchr(const char *s, int c);
def strchr : TargetLibCall<"strchr", Ptr, [Ptr, Int]>;

/// int strcmp(const char *s1, const char *s2);
def strcmp : TargetLibCall<"strcmp", Int, [Ptr, Ptr]>;

/// int strcoll(const char *s1, const char *s2);
def strcoll : TargetLibCall<"strcoll", Int, [Ptr, Ptr]>;

/// char *strcpy(char *s1, const char *s2);
```

- **L1485**: Comment documents the nearby API, invariant, or algorithmic intent: `int statvfs(const char *path, struct statvfs *buf);`. / 这行注释说明了附近 API、不变量或算法意图：`int statvfs(const char *path, struct statvfs *buf);`。
- **L1486**: Defines TableGen record `statvfs` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `statvfs`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1487**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1488**: Comment documents the nearby API, invariant, or algorithmic intent: `int statvfs64(const char *path, struct statvfs64 *buf)`. / 这行注释说明了附近 API、不变量或算法意图：`int statvfs64(const char *path, struct statvfs64 *buf)`。
- **L1489**: Defines TableGen record `statvfs64` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `statvfs64`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1490**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1491**: Comment documents the nearby API, invariant, or algorithmic intent: `char *stpcpy(char *s1, const char *s2);`. / 这行注释说明了附近 API、不变量或算法意图：`char *stpcpy(char *s1, const char *s2);`。
- **L1492**: Defines TableGen record `stpcpy` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `stpcpy`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1493**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1494**: Comment documents the nearby API, invariant, or algorithmic intent: `char *stpncpy(char *s1, const char *s2, size_t n);`. / 这行注释说明了附近 API、不变量或算法意图：`char *stpncpy(char *s1, const char *s2, size_t n);`。
- **L1495**: Defines TableGen record `stpncpy` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `stpncpy`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1496**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1497**: Comment documents the nearby API, invariant, or algorithmic intent: `int strcasecmp(const char *s1, const char *s2);`. / 这行注释说明了附近 API、不变量或算法意图：`int strcasecmp(const char *s1, const char *s2);`。
- **L1498**: Defines TableGen record `strcasecmp` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `strcasecmp`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1499**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1500**: Comment documents the nearby API, invariant, or algorithmic intent: `char *strcat(char *s1, const char *s2);`. / 这行注释说明了附近 API、不变量或算法意图：`char *strcat(char *s1, const char *s2);`。
- **L1501**: Defines TableGen record `strcat` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `strcat`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1502**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1503**: Comment documents the nearby API, invariant, or algorithmic intent: `char *strchr(const char *s, int c);`. / 这行注释说明了附近 API、不变量或算法意图：`char *strchr(const char *s, int c);`。
- **L1504**: Defines TableGen record `strchr` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `strchr`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1505**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1506**: Comment documents the nearby API, invariant, or algorithmic intent: `int strcmp(const char *s1, const char *s2);`. / 这行注释说明了附近 API、不变量或算法意图：`int strcmp(const char *s1, const char *s2);`。
- **L1507**: Defines TableGen record `strcmp` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `strcmp`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1508**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1509**: Comment documents the nearby API, invariant, or algorithmic intent: `int strcoll(const char *s1, const char *s2);`. / 这行注释说明了附近 API、不变量或算法意图：`int strcoll(const char *s1, const char *s2);`。
- **L1510**: Defines TableGen record `strcoll` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `strcoll`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1511**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1512**: Comment documents the nearby API, invariant, or algorithmic intent: `char *strcpy(char *s1, const char *s2);`. / 这行注释说明了附近 API、不变量或算法意图：`char *strcpy(char *s1, const char *s2);`。

### Lines 1513-1540

```tablegen
def strcpy : TargetLibCall<"strcpy", Ptr, [Ptr, Ptr]>;

/// size_t strcspn(const char *s1, const char *s2);
def strcspn : TargetLibCall<"strcspn", SizeT, [Ptr, Ptr]>;

/// char *strdup(const char *s1);
def strdup : TargetLibCall<"strdup", Ptr, [Ptr]>;

/// size_t strlcat(char *dst, const char *src, size_t size);
def strlcat : TargetLibCall<"strlcat", SizeT, [Ptr, Ptr, SizeT]>;

/// size_t strlcpy(char *dst, const char *src, size_t size);
def strlcpy : TargetLibCall<"strlcpy", SizeT, [Ptr, Ptr, SizeT]>;

/// size_t strlen(const char *s);
def strlen : TargetLibCall<"strlen", SizeT, [Ptr]>;

/// int strncasecmp(const char *s1, const char *s2, size_t n);
def strncasecmp : TargetLibCall<"strncasecmp", Int, [Ptr, Ptr, SizeT]>;

/// char *strncat(char *s1, const char *s2, size_t n);
def strncat : TargetLibCall<"strncat", Ptr, [Ptr, Ptr, SizeT]>;

/// int strncmp(const char *s1, const char *s2, size_t n);
def strncmp : TargetLibCall<"strncmp", Int, [Ptr, Ptr, SizeT]>;

/// char *strncpy(char *s1, const char *s2, size_t n);
def strncpy : TargetLibCall<"strncpy", Ptr, [Ptr, Ptr, SizeT]>;
```

- **L1513**: Defines TableGen record `strcpy` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `strcpy`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1514**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1515**: Comment documents the nearby API, invariant, or algorithmic intent: `size_t strcspn(const char *s1, const char *s2);`. / 这行注释说明了附近 API、不变量或算法意图：`size_t strcspn(const char *s1, const char *s2);`。
- **L1516**: Defines TableGen record `strcspn` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `strcspn`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1517**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1518**: Comment documents the nearby API, invariant, or algorithmic intent: `char *strdup(const char *s1);`. / 这行注释说明了附近 API、不变量或算法意图：`char *strdup(const char *s1);`。
- **L1519**: Defines TableGen record `strdup` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `strdup`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1520**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1521**: Comment documents the nearby API, invariant, or algorithmic intent: `size_t strlcat(char *dst, const char *src, size_t size);`. / 这行注释说明了附近 API、不变量或算法意图：`size_t strlcat(char *dst, const char *src, size_t size);`。
- **L1522**: Defines TableGen record `strlcat` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `strlcat`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1523**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1524**: Comment documents the nearby API, invariant, or algorithmic intent: `size_t strlcpy(char *dst, const char *src, size_t size);`. / 这行注释说明了附近 API、不变量或算法意图：`size_t strlcpy(char *dst, const char *src, size_t size);`。
- **L1525**: Defines TableGen record `strlcpy` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `strlcpy`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1526**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1527**: Comment documents the nearby API, invariant, or algorithmic intent: `size_t strlen(const char *s);`. / 这行注释说明了附近 API、不变量或算法意图：`size_t strlen(const char *s);`。
- **L1528**: Defines TableGen record `strlen` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `strlen`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1529**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1530**: Comment documents the nearby API, invariant, or algorithmic intent: `int strncasecmp(const char *s1, const char *s2, size_t n);`. / 这行注释说明了附近 API、不变量或算法意图：`int strncasecmp(const char *s1, const char *s2, size_t n);`。
- **L1531**: Defines TableGen record `strncasecmp` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `strncasecmp`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1532**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1533**: Comment documents the nearby API, invariant, or algorithmic intent: `char *strncat(char *s1, const char *s2, size_t n);`. / 这行注释说明了附近 API、不变量或算法意图：`char *strncat(char *s1, const char *s2, size_t n);`。
- **L1534**: Defines TableGen record `strncat` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `strncat`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1535**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1536**: Comment documents the nearby API, invariant, or algorithmic intent: `int strncmp(const char *s1, const char *s2, size_t n);`. / 这行注释说明了附近 API、不变量或算法意图：`int strncmp(const char *s1, const char *s2, size_t n);`。
- **L1537**: Defines TableGen record `strncmp` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `strncmp`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1538**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1539**: Comment documents the nearby API, invariant, or algorithmic intent: `char *strncpy(char *s1, const char *s2, size_t n);`. / 这行注释说明了附近 API、不变量或算法意图：`char *strncpy(char *s1, const char *s2, size_t n);`。
- **L1540**: Defines TableGen record `strncpy` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `strncpy`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。

### Lines 1541-1568

```tablegen

/// char *strndup(const char *s1, size_t n);
def strndup : TargetLibCall<"strndup", Ptr, [Ptr, SizeT]>;

/// size_t strnlen(const char *s, size_t maxlen);
def strnlen : TargetLibCall<"strnlen", SizeT, [Ptr, SizeT]>;

/// char *strpbrk(const char *s1, const char *s2);
def strpbrk : TargetLibCall<"strpbrk", Ptr, [Ptr, Ptr]>;

/// char *strrchr(const char *s, int c);
def strrchr : TargetLibCall<"strrchr", Ptr, [Ptr, Int]>;

/// size_t strspn(const char *s1, const char *s2);
def strspn : TargetLibCall<"strspn", SizeT, [Ptr, Ptr]>;

/// char *strstr(const char *s1, const char *s2);
def strstr : TargetLibCall<"strstr", Ptr, [Ptr, Ptr]>;

/// double strtod(const char *nptr, char **endptr);
def strtod : TargetLibCall<"strtod", Dbl, [Ptr, Ptr]>;

/// float strtof(const char *nptr, char **endptr);
def strtof : TargetLibCall<"strtof", Flt, [Ptr, Ptr]>;

/// char *strtok(char *s1, const char *s2);
def strtok : TargetLibCall<"strtok", Ptr, [Ptr, Ptr]>;

```

- **L1541**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1542**: Comment documents the nearby API, invariant, or algorithmic intent: `char *strndup(const char *s1, size_t n);`. / 这行注释说明了附近 API、不变量或算法意图：`char *strndup(const char *s1, size_t n);`。
- **L1543**: Defines TableGen record `strndup` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `strndup`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1544**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1545**: Comment documents the nearby API, invariant, or algorithmic intent: `size_t strnlen(const char *s, size_t maxlen);`. / 这行注释说明了附近 API、不变量或算法意图：`size_t strnlen(const char *s, size_t maxlen);`。
- **L1546**: Defines TableGen record `strnlen` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `strnlen`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1547**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1548**: Comment documents the nearby API, invariant, or algorithmic intent: `char *strpbrk(const char *s1, const char *s2);`. / 这行注释说明了附近 API、不变量或算法意图：`char *strpbrk(const char *s1, const char *s2);`。
- **L1549**: Defines TableGen record `strpbrk` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `strpbrk`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1550**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1551**: Comment documents the nearby API, invariant, or algorithmic intent: `char *strrchr(const char *s, int c);`. / 这行注释说明了附近 API、不变量或算法意图：`char *strrchr(const char *s, int c);`。
- **L1552**: Defines TableGen record `strrchr` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `strrchr`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1553**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1554**: Comment documents the nearby API, invariant, or algorithmic intent: `size_t strspn(const char *s1, const char *s2);`. / 这行注释说明了附近 API、不变量或算法意图：`size_t strspn(const char *s1, const char *s2);`。
- **L1555**: Defines TableGen record `strspn` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `strspn`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1556**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1557**: Comment documents the nearby API, invariant, or algorithmic intent: `char *strstr(const char *s1, const char *s2);`. / 这行注释说明了附近 API、不变量或算法意图：`char *strstr(const char *s1, const char *s2);`。
- **L1558**: Defines TableGen record `strstr` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `strstr`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1559**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1560**: Comment documents the nearby API, invariant, or algorithmic intent: `double strtod(const char *nptr, char **endptr);`. / 这行注释说明了附近 API、不变量或算法意图：`double strtod(const char *nptr, char **endptr);`。
- **L1561**: Defines TableGen record `strtod` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `strtod`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1562**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1563**: Comment documents the nearby API, invariant, or algorithmic intent: `float strtof(const char *nptr, char **endptr);`. / 这行注释说明了附近 API、不变量或算法意图：`float strtof(const char *nptr, char **endptr);`。
- **L1564**: Defines TableGen record `strtof` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `strtof`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1565**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1566**: Comment documents the nearby API, invariant, or algorithmic intent: `char *strtok(char *s1, const char *s2);`. / 这行注释说明了附近 API、不变量或算法意图：`char *strtok(char *s1, const char *s2);`。
- **L1567**: Defines TableGen record `strtok` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `strtok`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1568**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。

### Lines 1569-1596

```tablegen
/// char *strtok_r(char *s, const char *sep, char **lasts);
def strtok_r : TargetLibCall<"strtok_r", Ptr, [Ptr, Ptr, Ptr]>;

/// long int strtol(const char *nptr, char **endptr, int base);
def strtol : TargetLibCall<"strtol", Long, [Ptr, Ptr, Int]>;

/// long double strtold(const char *nptr, char **endptr);
def strtold : TargetLibCall<"strtold", LDbl, [Ptr, Ptr]>;

/// long long int strtoll(const char *nptr, char **endptr, int base);
def strtoll : TargetLibCall<"strtoll", LLong, [Ptr, Ptr, Int]>;

/// unsigned long int strtoul(const char *nptr, char **endptr, int base);
def strtoul : TargetLibCall<"strtoul", Long, [Ptr, Ptr, Int]>;

/// unsigned long long int strtoull(const char *nptr, char **endptr, int base);
def strtoull : TargetLibCall<"strtoull", LLong, [Ptr, Ptr, Int]>;

/// size_t strxfrm(char *s1, const char *s2, size_t n);
def strxfrm : TargetLibCall<"strxfrm", SizeT, [Ptr, Ptr, SizeT]>;

/// int system(const char *command);
def system : TargetLibCall<"system", Int, [Ptr]>;

/// double tan(double x);
def tan : TargetLibCall<"tan", Dbl, [Dbl]>;

/// float tanf(float x);
```

- **L1569**: Comment documents the nearby API, invariant, or algorithmic intent: `char *strtok_r(char *s, const char *sep, char **lasts);`. / 这行注释说明了附近 API、不变量或算法意图：`char *strtok_r(char *s, const char *sep, char **lasts);`。
- **L1570**: Defines TableGen record `strtok_r` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `strtok_r`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1571**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1572**: Comment documents the nearby API, invariant, or algorithmic intent: `long int strtol(const char *nptr, char **endptr, int base);`. / 这行注释说明了附近 API、不变量或算法意图：`long int strtol(const char *nptr, char **endptr, int base);`。
- **L1573**: Defines TableGen record `strtol` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `strtol`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1574**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1575**: Comment documents the nearby API, invariant, or algorithmic intent: `long double strtold(const char *nptr, char **endptr);`. / 这行注释说明了附近 API、不变量或算法意图：`long double strtold(const char *nptr, char **endptr);`。
- **L1576**: Defines TableGen record `strtold` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `strtold`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1577**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1578**: Comment documents the nearby API, invariant, or algorithmic intent: `long long int strtoll(const char *nptr, char **endptr, int base);`. / 这行注释说明了附近 API、不变量或算法意图：`long long int strtoll(const char *nptr, char **endptr, int base);`。
- **L1579**: Defines TableGen record `strtoll` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `strtoll`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1580**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1581**: Comment documents the nearby API, invariant, or algorithmic intent: `unsigned long int strtoul(const char *nptr, char **endptr, int base);`. / 这行注释说明了附近 API、不变量或算法意图：`unsigned long int strtoul(const char *nptr, char **endptr, int base);`。
- **L1582**: Defines TableGen record `strtoul` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `strtoul`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1583**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1584**: Comment documents the nearby API, invariant, or algorithmic intent: `unsigned long long int strtoull(const char *nptr, char **endptr, int base);`. / 这行注释说明了附近 API、不变量或算法意图：`unsigned long long int strtoull(const char *nptr, char **endptr, int base);`。
- **L1585**: Defines TableGen record `strtoull` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `strtoull`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1586**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1587**: Comment documents the nearby API, invariant, or algorithmic intent: `size_t strxfrm(char *s1, const char *s2, size_t n);`. / 这行注释说明了附近 API、不变量或算法意图：`size_t strxfrm(char *s1, const char *s2, size_t n);`。
- **L1588**: Defines TableGen record `strxfrm` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `strxfrm`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1589**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1590**: Comment documents the nearby API, invariant, or algorithmic intent: `int system(const char *command);`. / 这行注释说明了附近 API、不变量或算法意图：`int system(const char *command);`。
- **L1591**: Defines TableGen record `system` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `system`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1592**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1593**: Comment documents the nearby API, invariant, or algorithmic intent: `double tan(double x);`. / 这行注释说明了附近 API、不变量或算法意图：`double tan(double x);`。
- **L1594**: Defines TableGen record `tan` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `tan`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1595**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1596**: Comment documents the nearby API, invariant, or algorithmic intent: `float tanf(float x);`. / 这行注释说明了附近 API、不变量或算法意图：`float tanf(float x);`。

### Lines 1597-1624

```tablegen
def tanf : TargetLibCall<"tanf", Flt, [Flt]>;

/// double tanh(double x);
def tanh : TargetLibCall<"tanh", Dbl, [Dbl]>;

/// float tanhf(float x);
def tanhf : TargetLibCall<"tanhf", Flt, [Flt]>;

/// long double tanhl(long double x);
def tanhl : TargetLibCall<"tanhl", LDbl, [LDbl]>;

/// long double tanl(long double x);
def tanl : TargetLibCall<"tanl", LDbl, [LDbl]>;

/// clock_t times(struct tms *buffer);
def times : TargetLibCall<"times", IntPlus, [Ptr]>;

/// FILE *tmpfile(void);
def tmpfile : TargetLibCall<"tmpfile", Ptr, []>;

/// FILE *tmpfile64(void)
def tmpfile64 : TargetLibCall<"tmpfile64", Ptr, []>;

/// int toascii(int c);
def toascii : TargetLibCall<"toascii", Int, [Int]>;

/// double trunc(double x);
def trunc : TargetLibCall<"trunc", Dbl, [Dbl]>;
```

- **L1597**: Defines TableGen record `tanf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `tanf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1598**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1599**: Comment documents the nearby API, invariant, or algorithmic intent: `double tanh(double x);`. / 这行注释说明了附近 API、不变量或算法意图：`double tanh(double x);`。
- **L1600**: Defines TableGen record `tanh` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `tanh`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1601**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1602**: Comment documents the nearby API, invariant, or algorithmic intent: `float tanhf(float x);`. / 这行注释说明了附近 API、不变量或算法意图：`float tanhf(float x);`。
- **L1603**: Defines TableGen record `tanhf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `tanhf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1604**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1605**: Comment documents the nearby API, invariant, or algorithmic intent: `long double tanhl(long double x);`. / 这行注释说明了附近 API、不变量或算法意图：`long double tanhl(long double x);`。
- **L1606**: Defines TableGen record `tanhl` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `tanhl`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1607**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1608**: Comment documents the nearby API, invariant, or algorithmic intent: `long double tanl(long double x);`. / 这行注释说明了附近 API、不变量或算法意图：`long double tanl(long double x);`。
- **L1609**: Defines TableGen record `tanl` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `tanl`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1610**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1611**: Comment documents the nearby API, invariant, or algorithmic intent: `clock_t times(struct tms *buffer);`. / 这行注释说明了附近 API、不变量或算法意图：`clock_t times(struct tms *buffer);`。
- **L1612**: Defines TableGen record `times` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `times`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1613**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1614**: Comment documents the nearby API, invariant, or algorithmic intent: `FILE *tmpfile(void);`. / 这行注释说明了附近 API、不变量或算法意图：`FILE *tmpfile(void);`。
- **L1615**: Defines TableGen record `tmpfile` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `tmpfile`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1616**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1617**: Comment documents the nearby API, invariant, or algorithmic intent: `FILE *tmpfile64(void)`. / 这行注释说明了附近 API、不变量或算法意图：`FILE *tmpfile64(void)`。
- **L1618**: Defines TableGen record `tmpfile64` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `tmpfile64`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1619**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1620**: Comment documents the nearby API, invariant, or algorithmic intent: `int toascii(int c);`. / 这行注释说明了附近 API、不变量或算法意图：`int toascii(int c);`。
- **L1621**: Defines TableGen record `toascii` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `toascii`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1622**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1623**: Comment documents the nearby API, invariant, or algorithmic intent: `double trunc(double x);`. / 这行注释说明了附近 API、不变量或算法意图：`double trunc(double x);`。
- **L1624**: Defines TableGen record `trunc` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `trunc`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。

### Lines 1625-1652

```tablegen

/// float truncf(float x);
def truncf : TargetLibCall<"truncf", Flt, [Flt]>;

/// long double truncl(long double x);
def truncl : TargetLibCall<"truncl", LDbl, [LDbl]>;

/// int uname(struct utsname *name);
def uname : TargetLibCall<"uname", Int, [Ptr]>;

/// int ungetc(int c, FILE *stream);
def ungetc : TargetLibCall<"ungetc", Int, [Int, Ptr]>;

/// int unlink(const char *path);
def unlink : TargetLibCall<"unlink", Int, [Ptr]>;

/// int unsetenv(const char *name);
def unsetenv : TargetLibCall<"unsetenv", Int, [Ptr]>;

/// int utime(const char *path, const struct utimbuf *times);
def utime : TargetLibCall<"utime", Int, [Ptr, Ptr]>;

/// int utimes(const char *path, const struct timeval times[2]);
def utimes : TargetLibCall<"utimes", Int, [Ptr, Ptr]>;

/// void *valloc(size_t size);
def valloc : TargetLibCall<"valloc", Ptr, [SizeT]>;

```

- **L1625**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1626**: Comment documents the nearby API, invariant, or algorithmic intent: `float truncf(float x);`. / 这行注释说明了附近 API、不变量或算法意图：`float truncf(float x);`。
- **L1627**: Defines TableGen record `truncf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `truncf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1628**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1629**: Comment documents the nearby API, invariant, or algorithmic intent: `long double truncl(long double x);`. / 这行注释说明了附近 API、不变量或算法意图：`long double truncl(long double x);`。
- **L1630**: Defines TableGen record `truncl` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `truncl`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1631**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1632**: Comment documents the nearby API, invariant, or algorithmic intent: `int uname(struct utsname *name);`. / 这行注释说明了附近 API、不变量或算法意图：`int uname(struct utsname *name);`。
- **L1633**: Defines TableGen record `uname` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `uname`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1634**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1635**: Comment documents the nearby API, invariant, or algorithmic intent: `int ungetc(int c, FILE *stream);`. / 这行注释说明了附近 API、不变量或算法意图：`int ungetc(int c, FILE *stream);`。
- **L1636**: Defines TableGen record `ungetc` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `ungetc`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1637**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1638**: Comment documents the nearby API, invariant, or algorithmic intent: `int unlink(const char *path);`. / 这行注释说明了附近 API、不变量或算法意图：`int unlink(const char *path);`。
- **L1639**: Defines TableGen record `unlink` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `unlink`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1640**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1641**: Comment documents the nearby API, invariant, or algorithmic intent: `int unsetenv(const char *name);`. / 这行注释说明了附近 API、不变量或算法意图：`int unsetenv(const char *name);`。
- **L1642**: Defines TableGen record `unsetenv` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `unsetenv`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1643**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1644**: Comment documents the nearby API, invariant, or algorithmic intent: `int utime(const char *path, const struct utimbuf *times);`. / 这行注释说明了附近 API、不变量或算法意图：`int utime(const char *path, const struct utimbuf *times);`。
- **L1645**: Defines TableGen record `utime` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `utime`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1646**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1647**: Comment documents the nearby API, invariant, or algorithmic intent: `int utimes(const char *path, const struct timeval times[2]);`. / 这行注释说明了附近 API、不变量或算法意图：`int utimes(const char *path, const struct timeval times[2]);`。
- **L1648**: Defines TableGen record `utimes` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `utimes`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1649**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1650**: Comment documents the nearby API, invariant, or algorithmic intent: `void *valloc(size_t size);`. / 这行注释说明了附近 API、不变量或算法意图：`void *valloc(size_t size);`。
- **L1651**: Defines TableGen record `valloc` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `valloc`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1652**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。

### Lines 1653-1680

```tablegen
/// void *vec_calloc(size_t count, size_t size);
def vec_calloc : TargetLibCall<"vec_calloc", Ptr, [SizeT, SizeT]>;

/// void vec_free(void *ptr);
def vec_free : TargetLibCall<"vec_free", Void, [Ptr]>;

/// void *vec_malloc(size_t size);
def vec_malloc : TargetLibCall<"vec_malloc", Ptr, [SizeT]>;

/// void *vec_realloc(void *ptr, size_t size);
def vec_realloc : TargetLibCall<"vec_realloc", Ptr, [Ptr, SizeT]>;

/// int vfprintf(FILE *stream, const char *format, va_list ap);
def vfprintf : TargetLibCall<"vfprintf", Int, [Ptr, Ptr, Ptr]>;

/// int vfscanf(FILE *stream, const char *format, va_list arg);
def vfscanf : TargetLibCall<"vfscanf", Int, [Ptr, Ptr, Ptr]>;

/// int vprintf(const char *restrict format, va_list ap);
def vprintf : TargetLibCall<"vprintf", Int, [Ptr, Ptr]>;

/// int vscanf(const char *format, va_list arg);
def vscanf : TargetLibCall<"vscanf", Int, [Ptr, Ptr]>;

/// int vsnprintf(char *s, size_t n, const char *format, va_list ap);
def vsnprintf : TargetLibCall<"vsnprintf", Int, [Ptr, SizeT, Ptr, Ptr]>;

/// int vsprintf(char *s, const char *format, va_list ap);
```

- **L1653**: Comment documents the nearby API, invariant, or algorithmic intent: `void *vec_calloc(size_t count, size_t size);`. / 这行注释说明了附近 API、不变量或算法意图：`void *vec_calloc(size_t count, size_t size);`。
- **L1654**: Defines TableGen record `vec_calloc` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `vec_calloc`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1655**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1656**: Comment documents the nearby API, invariant, or algorithmic intent: `void vec_free(void *ptr);`. / 这行注释说明了附近 API、不变量或算法意图：`void vec_free(void *ptr);`。
- **L1657**: Defines TableGen record `vec_free` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `vec_free`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1658**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1659**: Comment documents the nearby API, invariant, or algorithmic intent: `void *vec_malloc(size_t size);`. / 这行注释说明了附近 API、不变量或算法意图：`void *vec_malloc(size_t size);`。
- **L1660**: Defines TableGen record `vec_malloc` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `vec_malloc`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1661**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1662**: Comment documents the nearby API, invariant, or algorithmic intent: `void *vec_realloc(void *ptr, size_t size);`. / 这行注释说明了附近 API、不变量或算法意图：`void *vec_realloc(void *ptr, size_t size);`。
- **L1663**: Defines TableGen record `vec_realloc` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `vec_realloc`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1664**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1665**: Comment documents the nearby API, invariant, or algorithmic intent: `int vfprintf(FILE *stream, const char *format, va_list ap);`. / 这行注释说明了附近 API、不变量或算法意图：`int vfprintf(FILE *stream, const char *format, va_list ap);`。
- **L1666**: Defines TableGen record `vfprintf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `vfprintf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1667**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1668**: Comment documents the nearby API, invariant, or algorithmic intent: `int vfscanf(FILE *stream, const char *format, va_list arg);`. / 这行注释说明了附近 API、不变量或算法意图：`int vfscanf(FILE *stream, const char *format, va_list arg);`。
- **L1669**: Defines TableGen record `vfscanf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `vfscanf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1670**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1671**: Comment documents the nearby API, invariant, or algorithmic intent: `int vprintf(const char *restrict format, va_list ap);`. / 这行注释说明了附近 API、不变量或算法意图：`int vprintf(const char *restrict format, va_list ap);`。
- **L1672**: Defines TableGen record `vprintf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `vprintf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1673**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1674**: Comment documents the nearby API, invariant, or algorithmic intent: `int vscanf(const char *format, va_list arg);`. / 这行注释说明了附近 API、不变量或算法意图：`int vscanf(const char *format, va_list arg);`。
- **L1675**: Defines TableGen record `vscanf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `vscanf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1676**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1677**: Comment documents the nearby API, invariant, or algorithmic intent: `int vsnprintf(char *s, size_t n, const char *format, va_list ap);`. / 这行注释说明了附近 API、不变量或算法意图：`int vsnprintf(char *s, size_t n, const char *format, va_list ap);`。
- **L1678**: Defines TableGen record `vsnprintf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `vsnprintf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1679**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1680**: Comment documents the nearby API, invariant, or algorithmic intent: `int vsprintf(char *s, const char *format, va_list ap);`. / 这行注释说明了附近 API、不变量或算法意图：`int vsprintf(char *s, const char *format, va_list ap);`。

### Lines 1681-1690

```tablegen
def vsprintf : TargetLibCall<"vsprintf", Int, [Ptr, Ptr, Ptr]>;

/// int vsscanf(const char *s, const char *format, va_list arg);
def vsscanf : TargetLibCall<"vsscanf", Int, [Ptr, Ptr, Ptr]>;

/// size_t wcslen (const wchar_t* wcs);
def wcslen : TargetLibCall<"wcslen", SizeT, [Ptr]>;

/// ssize_t write(int fildes, const void *buf, size_t nbyte);
def write : TargetLibCall<"write", SSizeT, [Int, Ptr, SizeT]>;
```

- **L1681**: Defines TableGen record `vsprintf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `vsprintf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1682**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1683**: Comment documents the nearby API, invariant, or algorithmic intent: `int vsscanf(const char *s, const char *format, va_list arg);`. / 这行注释说明了附近 API、不变量或算法意图：`int vsscanf(const char *s, const char *format, va_list arg);`。
- **L1684**: Defines TableGen record `vsscanf` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `vsscanf`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1685**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1686**: Comment documents the nearby API, invariant, or algorithmic intent: `size_t wcslen (const wchar_t* wcs);`. / 这行注释说明了附近 API、不变量或算法意图：`size_t wcslen (const wchar_t* wcs);`。
- **L1687**: Defines TableGen record `wcslen` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `wcslen`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。
- **L1688**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1689**: Comment documents the nearby API, invariant, or algorithmic intent: `ssize_t write(int fildes, const void *buf, size_t nbyte);`. / 这行注释说明了附近 API、不变量或算法意图：`ssize_t write(int fildes, const void *buf, size_t nbyte);`。
- **L1690**: Defines TableGen record `write` as a `TargetLibCall` record, adding one entry to the generated description database. / 定义 TableGen 记录 `write`，并将其建模为 `TargetLibCall` 记录，向生成用描述数据库中加入一个条目。

## Key Concepts / 关键概念

- EN: Layer: `Analysis` belongs to LLVM's analysis interfaces and cached program facts subsystem.
  - CN: 层次：`Analysis` 属于 LLVM 的分析接口与缓存的程序事实子系统。
- EN: Primary entities: `msvc_new_int, msvc_new_int_nothrow, msvc_new_longlong, msvc_new_longlong_nothrow, msvc_delete_ptr32, msvc_delete_ptr32_nothrow, msvc_delete_ptr32_int, msvc_delete_ptr64` are the main named types, records, or call points surfaced in this file.
  - CN: 核心实体：`msvc_new_int, msvc_new_int_nothrow, msvc_new_longlong, msvc_new_longlong_nothrow, msvc_delete_ptr32, msvc_delete_ptr32_nothrow, msvc_delete_ptr32_int, msvc_delete_ptr64` 是该文件中出现的主要命名类型、记录或调用入口。
- EN: TableGen role: it contributes declarative records that LLVM later lowers into generated C++ data structures or lookup tables.
  - CN: TableGen 角色：它提供声明式记录，LLVM 后续会将其下沉为生成的 C++ 数据结构或查找表。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/TargetLibraryInfoImpl.td` provide cached facts, legality checks, or cost models referenced by this file.
  - CN: 分析头文件：`llvm/Analysis/TargetLibraryInfoImpl.td` 提供了本文件引用的缓存事实、合法性检查或代价模型。
