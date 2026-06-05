# lexer.h — Code Analysis / 代码分析

## Source / 来源
- File / 文件: `torch/csrc/profiler/unwind/lexer.h`
- Repository / 仓库: `pytorch`
- Purpose / 作用:
  - EN: Implements native stack unwinding and symbolization helpers for profiling.
  - CN: 实现用于 profiling 的原生栈展开与符号化辅助工具。

## Line-by-Line Analysis / 逐行分析

### Lines 1-31
```cpp
 1 | #pragma once
 2 | #include <cstdint>
 3 | #include <cstring>
 4 | #include <utility>
 5 | 
 6 | #include <torch/csrc/profiler/unwind/dwarf_enums.h>
 7 | #include <torch/csrc/profiler/unwind/unwind_error.h>
 8 | 
 9 | namespace torch::unwind {
10 | 
11 | template <bool checked>
12 | struct LexerImpl {
13 |   LexerImpl(void* data, void* base = nullptr, void* end = nullptr)
14 |       : next_((const char*)data),
15 |         base_((int64_t)base),
16 |         end_((const char*)end) {}
17 | 
18 |   template <typename T>
19 |   T read() {
20 |     T result;
21 |     auto end = next_ + sizeof(T);
22 |     UNWIND_CHECK(
23 |         !checked || end <= end_,
24 |         "read out of bounds {} >= {}",
25 |         (void*)end,
26 |         (void*)end_);
27 |     memcpy(&result, next_, sizeof(T));
28 |     next_ = end;
29 |     return result;
30 |   }
31 | 
```
- EN: Brings in project headers such as `<torch/csrc/profiler/unwind/dwarf_enums.h>`, `<torch/csrc/profiler/unwind/unwind_error.h>` and system or third-party headers such as `<cstdint>`, `<cstring>`, `<utility>` so this section can use their types, APIs, or macros. Uses preprocessor directives to guard compilation, platform branches, or exported declarations. Places the implementation in namespace scopes (`torch::unwind`) so ownership matches the PyTorch subsystem layout. Defines or extends data abstractions such as `LexerImpl` that structure the state handled by this file.
- CN: 这里引入了项目头文件，例如 `<torch/csrc/profiler/unwind/dwarf_enums.h>`、`<torch/csrc/profiler/unwind/unwind_error.h>`以及系统或第三方头文件，例如 `<cstdint>`、`<cstring>`、`<utility>`，使本段代码能够使用相关类型、API 或宏。 使用预处理指令来控制编译开关、平台分支或导出声明。 把实现放入命名空间作用域（`torch::unwind`），使其归属与 PyTorch 子系统布局保持一致。 定义或扩展了 `LexerImpl` 等数据抽象，用来组织本文件处理的状态。

### Lines 32-53
```cpp
32 |   // SLEB/ULEB code adapted from LLVM equivalents
33 |   int64_t readSLEB128() {
34 |     int64_t Value = 0;
35 |     unsigned Shift = 0;
36 |     uint8_t Byte = 0;
37 |     do {
38 |       Byte = read<uint8_t>();
39 |       uint64_t Slice = Byte & 0x7f;
40 |       if ((Shift >= 64 && Slice != (Value < 0 ? 0x7f : 0x00)) ||
41 |           (Shift == 63 && Slice != 0 && Slice != 0x7f)) {
42 |         throw UnwindError("sleb128 too big for int64");
43 |       }
44 |       Value |= int64_t(Slice << Shift);
45 |       Shift += 7;
46 |     } while (Byte >= 128);
47 |     // Sign extend negative numbers if needed.
48 |     if (Shift < 64 && (Byte & 0x40)) {
49 |       Value |= int64_t((-1ULL) << Shift);
50 |     }
51 |     return Value;
52 |   }
53 | 
```
- EN: Implements routines such as `readSLEB128`, `UnwindError` that expose the key API or control flow of this region. At the statement level, this block declares abstract hooks that derived backends must provide; returns computed state or forwards values to callers.
- CN: 实现了 `readSLEB128`、`UnwindError` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段声明了需要由派生后端提供的抽象钩子；返回计算状态或把值转交给调用方。

### Lines 54-77
```cpp
54 |   uint64_t readULEB128() {
55 |     uint64_t Value = 0;
56 |     unsigned Shift = 0;
57 |     uint8_t p = 0;
58 |     do {
59 |       p = read<uint8_t>();
60 |       uint64_t Slice = p & 0x7f;
61 |       if ((Shift >= 64 && Slice != 0) || Slice << Shift >> Shift != Slice) {
62 |         throw UnwindError("uleb128 too big for uint64");
63 |       }
64 |       Value += Slice << Shift;
65 |       Shift += 7;
66 |     } while (p >= 128);
67 |     return Value;
68 |   }
69 |   const char* readCString() {
70 |     auto result = next_;
71 |     if (!checked) {
72 |       next_ += strlen(next_) + 1;
73 |       return result;
74 |     }
75 |     while (next_ < end_) {
76 |       if (*next_++ == '\0') {
77 |         return result;
```
- EN: Implements routines such as `readULEB128`, `UnwindError`, `readCString` that expose the key API or control flow of this region. At the statement level, this block declares abstract hooks that derived backends must provide; returns computed state or forwards values to callers.
- CN: 实现了 `readULEB128`、`UnwindError`、`readCString` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段声明了需要由派生后端提供的抽象钩子；返回计算状态或把值转交给调用方。

### Lines 78-109
```cpp
 78 |       }
 79 |     }
 80 |     UNWIND_CHECK(
 81 |         false, "string is out of bounds {} >= {}", (void*)next_, (void*)end_);
 82 |   }
 83 |   int64_t readEncoded(uint8_t enc) {
 84 |     int64_t r = 0;
 85 |     switch (enc & (~DW_EH_PE_indirect & 0xF0)) {
 86 |       case DW_EH_PE_absptr:
 87 |         break;
 88 |       case DW_EH_PE_pcrel:
 89 |         r = (int64_t)next_;
 90 |         break;
 91 |       case DW_EH_PE_datarel:
 92 |         r = base_;
 93 |         break;
 94 |       default:
 95 |         throw UnwindError("unknown encoding");
 96 |     }
 97 |     return r + readEncodedValue(enc);
 98 |   }
 99 |   int64_t readEncodedOr(uint8_t enc, int64_t orelse) {
100 |     if (enc == DW_EH_PE_omit) {
101 |       return orelse;
102 |     }
103 |     return readEncoded(enc);
104 |   }
105 | 
106 |   int64_t read4or8Length() {
107 |     return readSectionLength().first;
108 |   }
109 | 
```
- EN: Implements routines such as `readEncoded`, `UnwindError`, `readEncodedOr`, `read4or8Length` that expose the key API or control flow of this region. At the statement level, this block declares abstract hooks that derived backends must provide; returns computed state or forwards values to callers.
- CN: 实现了 `readEncoded`、`UnwindError`、`readEncodedOr`、`read4or8Length` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段声明了需要由派生后端提供的抽象钩子；返回计算状态或把值转交给调用方。

### Lines 110-125
```cpp
110 |   std::pair<int64_t, bool> readSectionLength() {
111 |     int64_t length = read<uint32_t>();
112 |     if (length == 0xFFFFFFFF) {
113 |       return std::make_pair(read<int64_t>(), true);
114 |     }
115 |     return std::make_pair(length, false);
116 |   }
117 | 
118 |   void* loc() const {
119 |     return (void*)next_;
120 |   }
121 |   LexerImpl& skip(size_t bytes) {
122 |     next_ += bytes;
123 |     return *this;
124 |   }
125 | 
```
- EN: Implements routines such as `readSectionLength`, `loc`, `skip` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 实现了 `readSectionLength`、`loc`、`skip` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 126-154
```cpp
126 |   int64_t readEncodedValue(uint8_t enc) {
127 |     switch (enc & 0xF) {
128 |       case DW_EH_PE_udata2:
129 |         return read<uint16_t>();
130 |       case DW_EH_PE_sdata2:
131 |         return read<int16_t>();
132 |       case DW_EH_PE_udata4:
133 |         return read<uint32_t>();
134 |       case DW_EH_PE_sdata4:
135 |         return read<int32_t>();
136 |       case DW_EH_PE_udata8:
137 |         return read<uint64_t>();
138 |       case DW_EH_PE_sdata8:
139 |         return read<int64_t>();
140 |       case DW_EH_PE_uleb128:
141 |         return readULEB128();
142 |       case DW_EH_PE_sleb128:
143 |         return readSLEB128();
144 |       default:
145 |         throw UnwindError("not implemented");
146 |     }
147 |   }
148 | 
149 |  private:
150 |   const char* next_;
151 |   int64_t base_;
152 |   const char* end_;
153 | };
154 | 
```
- EN: Implements routines such as `readEncodedValue`, `readULEB128`, `readSLEB128`, `UnwindError` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; stores long-lived member state for later calls.
- CN: 实现了 `readEncodedValue`、`readULEB128`、`readSLEB128`、`UnwindError` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；保存供后续调用使用的长期成员状态。

### Lines 155-159
```cpp
155 | // using Lexer = LexerImpl<false>;
156 | using CheckedLexer = LexerImpl<true>;
157 | using Lexer = LexerImpl<false>;
158 | 
159 | } // namespace torch::unwind
```
- EN: Continues the file's main role: Implements native stack unwinding and symbolization helpers for profiling.
- CN: 继续承担本文件的主要职责：实现用于 profiling 的原生栈展开与符号化辅助工具。

## Key Concepts / 关键概念

- **Primary types / 主要类型**
  - EN: `LexerImpl`.
  - CN: `LexerImpl`。
- **Important routines / 重要例程**
  - EN: `read`, `readSLEB128`, `UnwindError`, `readULEB128`, `readCString`, `readEncoded`, `readEncodedOr`, `read4or8Length`.
  - CN: `read`、`readSLEB128`、`UnwindError`、`readULEB128`、`readCString`、`readEncoded`、`readEncodedOr`、`read4or8Length`。
- **Subsystem scope / 子系统作用域**
  - EN: The code lives under `torch::unwind`, indicating the owning PyTorch subsystem.
  - CN: 代码位于 `torch::unwind` 命名空间下，这说明了其所属的 PyTorch 子系统。

## Dependencies / 依赖关系

- Local includes / 本地头文件: `<torch/csrc/profiler/unwind/dwarf_enums.h>`, `<torch/csrc/profiler/unwind/unwind_error.h>`
- External includes / 外部头文件: `<cstdint>`, `<cstring>`, `<utility>`
- Relationship / 关系:
  - EN: The unit relies on neighboring PyTorch headers for subsystem-specific types, dispatcher hooks, and utility helpers. Standard-library or third-party headers provide generic facilities such as containers, threading, math, or backend integration points. Its exported or visible routines are then consumed by callers in the same namespace and by higher-level runtime paths elsewhere in PyTorch.
  - CN: 该编译单元依赖相邻的 PyTorch 头文件，以获得子系统类型、分发器钩子和辅助工具。 标准库或第三方头文件提供了通用能力，例如容器、线程、数学支持或后端集成入口。 随后，这里导出或可见的例程会被同一命名空间中的调用方，以及 PyTorch 其他位置的更高层运行时路径所使用。
