# kernel.hpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `include/cute/util/compat/kernel.hpp`
- Purpose (EN): Implements the compatibility layer that maps CUDA-like concepts onto SYCL/back-end abstractions for atomics, kernels, launch, memory, math, and device queries.
- 作用 (CN): 实现兼容层，把类 CUDA 概念映射到 SYCL/后端抽象，覆盖原子操作、内核、启动、内存、数学和设备查询。

## Line-by-Line Analysis / 逐行分析

### Lines 1-21

```text
    1 | /***************************************************************************
    2 |  *
    3 |  *  Copyright (C) Codeplay Software Ltd.
    4 |  *  Copyright (C) 2025 Intel Corporation, All rights reserved.
    5 |  *
    6 |  *  Part of the LLVM Project, under the Apache License v2.0 with LLVM
    7 |  *  Exceptions. See https://llvm.org/LICENSE.txt for license information.
    8 |  *  SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
    9 |  *
   10 |  *  Unless required by applicable law or agreed to in writing, software
   11 |  *  distributed under the License is distributed on an "AS IS" BASIS,
   12 |  *  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   13 |  *  See the License for the specific language governing permissions and
   14 |  *  limitations under the License.
   15 |  *
   16 |  *  SYCL compatibility extension
   17 |  *
   18 |  *  kernel.hpp
   19 |  *
   20 |  *  Description:
   21 |  *    kernel functionality for the SYCL compatibility extension.
```
**EN:** Provides the license notice, copyright ownership, and redistribution terms for this header.
**CN:** 给出该头文件的许可证声明、版权归属以及再分发条款。

### Lines 22-34

```text
   22 |  **************************************************************************/
   24 | // The original source was under the license below:
   25 | //==---- kernel.hpp -------------------------------*- C++ -*----------------==//
   26 | //
   27 | // Copyright (C) Intel Corporation
   28 | // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   29 | // See https://llvm.org/LICENSE.txt for license information.
   30 | //
   31 | //===----------------------------------------------------------------------===//
   33 | #pragma once
   34 | #pragma GCC system_header
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 36-57

```text
   36 | #ifdef _WIN32
   37 | #include <unordered_set>
   38 | #include <windows.h>
   39 | #else
   40 | #include <dlfcn.h>
   41 | #endif
   43 | #if defined(__has_include) && __has_include(<filesystem>)
   44 | #include <filesystem>
   45 | #elif defined(__has_include) && __has_include(<experimental/filesystem>)
   46 | #include <experimental/filesystem>
   47 | #else
   48 | #error "SYCLomatic runtime requires C++ filesystem support"
   49 | #endif
   51 | #include <fstream>
   52 | #include <random>
   54 | #include <sycl/image.hpp>
   55 | #include <sycl/info/info_desc.hpp>
   56 | #include <sycl/nd_range.hpp>
   57 | #include <sycl/queue.hpp>
```
**EN:** Sets up the header dependencies for this file by importing `unordered_set`, `windows.h`, `dlfcn.h`, `filesystem`, `experimental/filesystem`, and 6 more include(s).
**CN:** 通过引入 `unordered_set`, `windows.h`, `dlfcn.h`, `filesystem`, `experimental/filesystem`，以及另外 6 个头文件 为该文件建立头文件依赖。

### Lines 59-59

```text
   59 | namespace compat {
```
**EN:** Opens or closes namespace scope so the following declarations remain grouped under the intended CuTe or backend namespace hierarchy.
**CN:** 打开或关闭命名空间作用域，使后续声明保持在预期的 CuTe 或后端命名空间层级中。

### Lines 61-62

```text
   61 | typedef void (*kernel_functor)(sycl::queue &, const sycl::nd_range<3> &,
   62 |                                unsigned int, void **, void **);
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 64-66

```text
   64 | struct kernel_function_info {
   65 |   int max_work_group_size = 0;
   66 | };
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 68-74

```text
   68 | static inline void get_kernel_function_info(kernel_function_info *kernel_info,
   69 |                                             const void *function) {
   70 |   kernel_info->max_work_group_size =
   71 |       detail::dev_mgr::instance()
   72 |           .current_device()
   73 |           .get_info<sycl::info::device::max_work_group_size>();
   74 | }
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 76-84

```text
   76 | static inline kernel_function_info
   77 | get_kernel_function_info(const void *function) {
   78 |   kernel_function_info kernel_info;
   79 |   kernel_info.max_work_group_size =
   80 |       detail::dev_mgr::instance()
   81 |           .current_device()
   82 |           .get_info<sycl::info::device::max_work_group_size>();
   83 |   return kernel_info;
   84 | }
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 86-86

```text
   86 | namespace detail {
```
**EN:** Opens or closes namespace scope so the following declarations remain grouped under the intended CuTe or backend namespace hierarchy.
**CN:** 打开或关闭命名空间作用域，使后续声明保持在预期的 CuTe 或后端命名空间层级中。

### Lines 88-92

```text
   88 | #if defined(__has_include) && __has_include(<filesystem>)
   89 | namespace fs = std::filesystem;
   90 | #else
   91 | namespace fs = std::experimental::filesystem;
   92 | #endif
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。

### Lines 94-99

```text
   94 | /// Write data to temporary file and return absolute path to temporary file.
   95 | /// Temporary file is created in a temporary directory both of which have random
   96 | /// names with only the user having access permissions.  Only one temporary file
   97 | /// will be created in the temporary directory.
   98 | static inline fs::path write_data_to_file(char const *const data, size_t size) {
   99 |   std::error_code ec;
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数。

### Lines 101-103

```text
  101 |   if (sizeof(size_t) >= sizeof(std::streamsize) &&
  102 |       size > (std::numeric_limits<std::streamsize>::max)())
  103 |     throw std::runtime_error("[Compat] data file too large");
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。

### Lines 105-108

```text
  105 |   // random number generator
  106 |   std::random_device dev;
  107 |   std::mt19937 prng(dev());
  108 |   std::uniform_int_distribution<uint64_t> rand(0);
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 110-113

```text
  110 |   // find temporary directory
  111 |   auto tmp_dir = fs::temp_directory_path(ec);
  112 |   if (ec)
  113 |     throw std::runtime_error("[Compat] could not find temporary directory");
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。

### Lines 115-119

```text
  115 |   // create private directory
  116 |   std::stringstream directory;
  117 |   fs::path directory_path;
  118 |   constexpr int max_attempts = 5;
  119 |   int i;
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还大量依赖编译期特化。

### Lines 121-129

```text
  121 |   for (i = 0; i < max_attempts; i++) {
  122 |     directory << std::hex << rand(prng);
  123 |     directory_path = tmp_dir / directory.str();
  124 |     if (fs::create_directory(directory_path)) {
  125 |       break;
  126 |     }
  127 |   }
  128 |   if (i == max_attempts)
  129 |     throw std::runtime_error("[Compat] could not create directory");
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。

### Lines 131-135

```text
  131 |   // only allow owner permissions to private directory
  132 |   fs::permissions(directory_path, fs::perms::owner_all, ec);
  133 |   if (ec)
  134 |     throw std::runtime_error(
  135 |         "[Compat] could not set directory permissions");
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。

### Lines 137-144

```text
  137 |   // random filename in private directory
  138 |   std::stringstream filename;
  139 |   filename << std::hex << rand(prng);
  140 | #ifdef _WIN32
  141 |   auto filepath = directory_path / (filename.str() + ".dll");
  142 | #else
  143 |   auto filepath = directory_path / filename.str();
  144 | #endif
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。

### Lines 146-165

```text
  146 |   // write data to temporary file
  147 |   auto outfile = std::ofstream(filepath, std::ios::out | std::ios::binary);
  148 |   if (outfile) {
  149 |     // only allow program to write file
  150 |     fs::permissions(filepath, fs::perms::owner_write, ec);
  151 |     if (ec)
  152 |       throw std::runtime_error("[Compat] could not set permissions");
  153 | 
  154 |     outfile.write(data, size);
  155 |     if (!outfile.good())
  156 |       throw std::runtime_error("[Compat] could not write data");
  157 |     outfile.close();
  158 | 
  159 |     // only allow program to read/execute file
  160 |     fs::permissions(filepath, fs::perms::owner_read | fs::perms::owner_exec,
  161 |                     ec);
  162 |     if (ec)
  163 |       throw std::runtime_error("[Compat] could not set permissions");
  164 |   } else
  165 |     throw std::runtime_error("[Compat] could not write data");
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。

### Lines 167-185

```text
  167 |   // check temporary file contents
  168 |   auto infile = std::ifstream(filepath, std::ios::in | std::ios::binary);
  169 |   if (infile) {
  170 |     bool mismatch = false;
  171 |     size_t cnt = 0;
  172 | 
  173 |     while (1) {
  174 |       char c;
  175 |       infile.get(c);
  176 |       if (infile.eof())
  177 |         break;
  178 |       if (c != data[cnt++])
  179 |         mismatch = true;
  180 |     }
  181 |     if (cnt != size || mismatch)
  182 |       throw std::runtime_error(
  183 |           "[Compat] file contents not written correctly");
  184 |   } else
  185 |     throw std::runtime_error("[Compat] could not validate file");
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。

### Lines 187-188

```text
  187 |   if (!filepath.is_absolute())
  188 |     throw std::runtime_error("[Compat] temporary filepath is not absolute");
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。

### Lines 190-191

```text
  190 |   return filepath;
  191 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数。

### Lines 193-194

```text
  193 | static inline uint16_t extract16(unsigned char const *const ptr) {
  194 |   uint16_t ret = 0;
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also adapts pointer/iterator behavior for CuTe memory models.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还为 CuTe 内存模型适配指针/迭代器行为。

### Lines 196-197

```text
  196 |   ret |= static_cast<uint16_t>(ptr[0]) << 0;
  197 |   ret |= static_cast<uint16_t>(ptr[1]) << 8;
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also adapts pointer/iterator behavior for CuTe memory models.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还为 CuTe 内存模型适配指针/迭代器行为。

### Lines 199-200

```text
  199 |   return (ret);
  200 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数。

### Lines 202-203

```text
  202 | static inline uint32_t extract32(unsigned char const *const ptr) {
  203 |   uint32_t ret = 0;
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also adapts pointer/iterator behavior for CuTe memory models.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还为 CuTe 内存模型适配指针/迭代器行为。

### Lines 205-208

```text
  205 |   ret |= static_cast<uint32_t>(ptr[0]) << 0;
  206 |   ret |= static_cast<uint32_t>(ptr[1]) << 8;
  207 |   ret |= static_cast<uint32_t>(ptr[2]) << 16;
  208 |   ret |= static_cast<uint32_t>(ptr[3]) << 24;
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also adapts pointer/iterator behavior for CuTe memory models.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还为 CuTe 内存模型适配指针/迭代器行为。

### Lines 210-211

```text
  210 |   return (ret);
  211 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数。

### Lines 213-214

```text
  213 | static inline uint64_t extract64(unsigned char const *const ptr) {
  214 |   uint64_t ret = 0;
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also adapts pointer/iterator behavior for CuTe memory models.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还为 CuTe 内存模型适配指针/迭代器行为。

### Lines 216-223

```text
  216 |   ret |= static_cast<uint64_t>(ptr[0]) << 0;
  217 |   ret |= static_cast<uint64_t>(ptr[1]) << 8;
  218 |   ret |= static_cast<uint64_t>(ptr[2]) << 16;
  219 |   ret |= static_cast<uint64_t>(ptr[3]) << 24;
  220 |   ret |= static_cast<uint64_t>(ptr[4]) << 32;
  221 |   ret |= static_cast<uint64_t>(ptr[5]) << 40;
  222 |   ret |= static_cast<uint64_t>(ptr[6]) << 48;
  223 |   ret |= static_cast<uint64_t>(ptr[7]) << 56;
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also adapts pointer/iterator behavior for CuTe memory models.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还为 CuTe 内存模型适配指针/迭代器行为。

### Lines 225-226

```text
  225 |   return (ret);
  226 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数。

### Lines 228-229

```text
  228 | static inline uint64_t get_lib_size(char const *const blob) {
  229 | #ifdef _WIN32
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。

### Lines 230-237

```text
  230 |   ///////////////////////////////////////////////////////////////////////
  231 |   // Analyze DOS stub
  232 |   unsigned char const *const ublob =
  233 |       reinterpret_cast<unsigned char const *const>(blob);
  234 |   if (ublob[0] != 0x4d || ublob[1] != 0x5a) {
  235 |     throw std::runtime_error("[Compat] blob is not a Windows DLL.");
  236 |   }
  237 |   uint32_t pe_header_offset = extract32(ublob + 0x3c);
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。

### Lines 239-241

```text
  239 |   ///////////////////////////////////////////////////////////////////////
  240 |   // Ananlyze PE-header
  241 |   unsigned char const *const pe_header = ublob + pe_header_offset;
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。

### Lines 243-248

```text
  243 |   // signature
  244 |   uint32_t pe_signature = extract32(pe_header + 0);
  245 |   if (pe_signature != 0x00004550) {
  246 |     throw std::runtime_error(
  247 |         "[Compat] PE-header signature is not 0x00004550");
  248 |   }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。

### Lines 250-254

```text
  250 |   // machine
  251 |   uint16_t machine = extract16(pe_header + 4);
  252 |   if (machine != 0x8664) {
  253 |     throw std::runtime_error("[Compat] only DLLs for x64 supported");
  254 |   }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。

### Lines 256-257

```text
  256 |   // number of sections
  257 |   uint16_t number_of_sections = extract16(pe_header + 6);
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。

### Lines 259-260

```text
  259 |   // sizeof optional header
  260 |   uint16_t sizeof_optional_header = extract16(pe_header + 20);
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。

### Lines 262-266

```text
  262 |   // magic
  263 |   uint16_t magic = extract16(pe_header + 24);
  264 |   if (magic != 0x10b && magic != 0x20b) {
  265 |     throw std::runtime_error("[Compat] MAGIC is not 0x010b or 0x020b");
  266 |   }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。

### Lines 268-270

```text
  268 |   ///////////////////////////////////////////////////////////////////////
  269 |   // Analyze tail of optional header
  270 |   constexpr int coff_header_size = 24;
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还大量依赖编译期特化。

### Lines 272-276

```text
  272 |   unsigned char const *const tail_of_optional_header =
  273 |       pe_header + coff_header_size + sizeof_optional_header;
  274 |   if (extract64(tail_of_optional_header - 8) != 0) {
  275 |     throw std::runtime_error("Optional header not zero-padded");
  276 |   }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。

### Lines 278-282

```text
  278 |   ///////////////////////////////////////////////////////////////////////
  279 |   // Analyze last section header
  280 |   constexpr int section_header_size = 40;
  281 |   unsigned char const *const last_section_header =
  282 |       tail_of_optional_header + section_header_size * (number_of_sections - 1);
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also leans heavily on compile-time specialization.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还大量依赖编译期特化。

### Lines 284-285

```text
  284 |   uint32_t sizeof_raw_data = extract32(last_section_header + 16);
  285 |   uint32_t pointer_to_raw_data = extract32(last_section_header + 20);
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also adapts pointer/iterator behavior for CuTe memory models.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还为 CuTe 内存模型适配指针/迭代器行为。

### Lines 287-290

```text
  287 |   return sizeof_raw_data + pointer_to_raw_data;
  288 | #else
  289 |   if (blob[0] != 0x7F || blob[1] != 'E' || blob[2] != 'L' || blob[3] != 'F')
  290 |     throw std::runtime_error("[Compat] blob is not in ELF format");
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and adapts pointer/iterator behavior for CuTe memory models.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并为 CuTe 内存模型适配指针/迭代器行为。

### Lines 292-293

```text
  292 |   if (blob[4] != 0x02)
  293 |     throw std::runtime_error("[Compat] only 64-bit headers are supported");
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。

### Lines 295-297

```text
  295 |   if (blob[5] != 0x01)
  296 |     throw std::runtime_error(
  297 |         "[Compat] only little-endian headers are supported");
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。

### Lines 299-303

```text
  299 |   unsigned char const *const ublob =
  300 |       reinterpret_cast<unsigned char const *const>(blob);
  301 |   uint64_t e_shoff = extract64(ublob + 0x28);
  302 |   uint16_t e_shentsize = extract16(ublob + 0x3A);
  303 |   uint16_t e_shnum = extract16(ublob + 0x3C);
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。

### Lines 305-307

```text
  305 |   return e_shoff + (e_shentsize * e_shnum);
  306 | #endif
  307 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数。

### Lines 309-334

```text
  309 | #ifdef _WIN32
  310 | class path_lib_record {
  311 | public:
  312 |   void operator=(const path_lib_record &) = delete;
  313 |   ~path_lib_record() {
  314 |     for (auto entry : lib_to_path) {
  315 |       FreeLibrary(static_cast<HMODULE>(entry.first));
  316 |       fs::permissions(entry.second, fs::perms::owner_all);
  317 |       fs::remove_all(entry.second.remove_filename());
  318 |     }
  319 |   }
  320 |   static void record_lib_path(fs::path path, void *library) {
  321 |     lib_to_path[library] = path;
  322 |   }
  323 |   static void remove_lib(void *library) {
  324 |     auto path = lib_to_path[library];
  325 |     std::error_code ec;
  326 | 
  327 |     FreeLibrary(static_cast<HMODULE>(library));
  328 |     fs::permissions(path, fs::perms::owner_all);
  329 |     if (fs::remove_all(path.remove_filename(), ec) != 2 || ec)
  330 |       // one directory and one temporary file should have been deleted
  331 |       throw std::runtime_error("[Compat] directory delete failed");
  332 | 
  333 |     lib_to_path.erase(library);
  334 |   }
```
**EN:** Defines `path_lib_record` as a reusable type-level building block in this header.
**CN:** 将 `path_lib_record` 定义为本头文件中的可复用类型级构件。

### Lines 336-341

```text
  336 | private:
  337 |   static inline std::unordered_map<void *, fs::path> lib_to_path;
  338 | };
  339 | #endif
  341 | } // namespace detail
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。

### Lines 343-346

```text
  343 | class kernel_library {
  344 | public:
  345 |   constexpr kernel_library() : ptr{nullptr} {}
  346 |   constexpr kernel_library(void *ptr) : ptr{ptr} {}
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 348-348

```text
  348 |   operator void *() const { return ptr; }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and adapts pointer/iterator behavior for CuTe memory models.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并为 CuTe 内存模型适配指针/迭代器行为。

### Lines 350-355

```text
  350 | private:
  351 |   void *ptr;
  352 | #ifdef _WIN32
  353 |   static inline detail::path_lib_record single_instance_to_trigger_destructor;
  354 | #endif
  355 | };
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also adapts pointer/iterator behavior for CuTe memory models.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还为 CuTe 内存模型适配指针/迭代器行为。

### Lines 357-357

```text
  357 | namespace detail {
```
**EN:** Opens or closes namespace scope so the following declarations remain grouped under the intended CuTe or backend namespace hierarchy.
**CN:** 打开或关闭命名空间作用域，使后续声明保持在预期的 CuTe 或后端命名空间层级中。

### Lines 359-368

```text
  359 | static inline kernel_library load_dl_from_data(char const *const data,
  360 |                                                size_t size) {
  361 |   fs::path filename = write_data_to_file(data, size);
  362 | #ifdef _WIN32
  363 |   void *so = LoadLibraryW(filename.wstring().c_str());
  364 | #else
  365 |   void *so = dlopen(filename.c_str(), RTLD_LAZY);
  366 | #endif
  367 |   if (so == nullptr)
  368 |     throw std::runtime_error("[Compat] failed to load kernel library");
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 370-373

```text
  370 | #ifdef _WIN32
  371 |   detail::path_lib_record::record_lib_path(filename, so);
  372 | #else
  373 |   std::error_code ec;
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。

### Lines 375-379

```text
  375 |   // Windows DLL cannot be deleted while in use
  376 |   if (fs::remove_all(filename.remove_filename(), ec) != 2 || ec)
  377 |     // one directory and one temporary file should have been deleted
  378 |     throw std::runtime_error("[Compat] directory delete failed");
  379 | #endif
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。

### Lines 381-384

```text
  381 |   return so;
  382 | }
  384 | } // namespace detail
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数。

### Lines 386-390

```text
  386 | /// Load kernel library and return a handle to use the library.
  387 | /// \param [in] name The name of the library.
  388 | static inline kernel_library load_kernel_library(const std::string &name) {
  389 |   std::ifstream ifs;
  390 |   ifs.open(name, std::ios::in | std::ios::binary);
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 392-393

```text
  392 |   std::stringstream buffer;
  393 |   buffer << ifs.rdbuf();
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。

### Lines 395-397

```text
  395 |   const std::string buffer_string = buffer.str();
  396 |   return detail::load_dl_from_data(buffer_string.c_str(), buffer_string.size());
  397 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数。

### Lines 399-403

```text
  399 | /// Load kernel library whose image is alreay in memory and return a handle to
  400 | /// use the library.
  401 | /// \param [in] image A pointer to the image in memory.
  402 | static inline kernel_library load_kernel_library_mem(char const *const image) {
  403 |   const size_t size = detail::get_lib_size(image);
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 405-406

```text
  405 |   return detail::load_dl_from_data(image, size);
  406 | }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数。

### Lines 408-416

```text
  408 | /// Unload kernel library.
  409 | /// \param [in,out] library Handle to the library to be closed.
  410 | static inline void unload_kernel_library(const kernel_library &library) {
  411 | #ifdef _WIN32
  412 |   detail::path_lib_record::remove_lib(library);
  413 | #else
  414 |   dlclose(library);
  415 | #endif
  416 | }
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 418-421

```text
  418 | class kernel_function {
  419 | public:
  420 |   constexpr kernel_function() : ptr{nullptr} {}
  421 |   constexpr kernel_function(kernel_functor ptr) : ptr{ptr} {}
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 423-423

```text
  423 |   operator void *() const { return ((void *)ptr); }
```
**EN:** Defines a supporting block used by the surrounding CuTe abstractions. It also acts as a factory/helper for constructing derived objects and adapts pointer/iterator behavior for CuTe memory models.
**CN:** 定义周围 CuTe 抽象所使用的辅助代码块。 它还充当构造派生对象的工厂/辅助函数并为 CuTe 内存模型适配指针/迭代器行为。

### Lines 425-428

```text
  425 |   void operator()(sycl::queue &q, const sycl::nd_range<3> &range,
  426 |                   unsigned int local_mem_size, void **args, void **extra) {
  427 |     ptr(q, range, local_mem_size, args, extra);
  428 |   }
```
**EN:** Implements `operator`, a helper routine used by the surrounding CuTe abstractions. It also implements coordinate mapping or slicing behavior and bridges to SYCL execution or group abstractions.
**CN:** 实现 `operator`，这是周围 CuTe 抽象所使用的辅助例程。 它还实现坐标映射或切片行为并桥接到 SYCL 执行模型或 group 抽象。

### Lines 430-432

```text
  430 | private:
  431 |   kernel_functor ptr;
  432 | };
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 434-450

```text
  434 | /// Find kernel function in a kernel library and return its address.
  435 | /// \param [in] library Handle to the kernel library.
  436 | /// \param [in] name Name of the kernel function.
  437 | static inline kernel_function get_kernel_function(kernel_library &library,
  438 |                                                   const std::string &name) {
  439 | #ifdef _WIN32
  440 |   kernel_functor fn = reinterpret_cast<kernel_functor>(
  441 |       GetProcAddress(static_cast<HMODULE>(static_cast<void *>(library)),
  442 |                      (name + std::string("_wrapper")).c_str()));
  443 | #else
  444 |   kernel_functor fn = reinterpret_cast<kernel_functor>(
  445 |       dlsym(library, (name + std::string("_wrapper")).c_str()));
  446 | #endif
  447 |   if (fn == nullptr)
  448 |     throw std::runtime_error("[Compat] failed to get function");
  449 |   return fn;
  450 | }
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

### Lines 452-471

```text
  452 | /// Invoke a kernel function.
  453 | /// \param [in] function kernel function.
  454 | /// \param [in] queue SYCL queue used to execute kernel
  455 | /// \param [in] group_range SYCL group range
  456 | /// \param [in] local_range SYCL local range
  457 | /// \param [in] local_mem_size The size of local memory required by the kernel
  458 | ///             function.
  459 | /// \param [in] kernel_params Array of pointers to kernel arguments.
  460 | /// \param [in] extra Extra arguments.
  461 | static inline void invoke_kernel_function(kernel_function &function,
  462 |                                           sycl::queue &queue,
  463 |                                           sycl::range<3> group_range,
  464 |                                           sycl::range<3> local_range,
  465 |                                           unsigned int local_mem_size,
  466 |                                           void **kernel_params, void **extra) {
  467 |   function(queue, sycl::nd_range<3>(group_range * local_range, local_range),
  468 |            local_mem_size, kernel_params, extra);
  469 | }
  471 | } // namespace compat
```
**EN:** Defines a backend compatibility helper that adapts common kernel/runtime concepts to the CuTe portability layer.
**CN:** 定义后端兼容辅助工具，把常见的内核/运行时概念适配到 CuTe 可移植层。

## Key Concepts / 关键概念

- Compile-time numeric metaprogramming / 编译期数值元编程
- Static containers and tuple-like storage / 静态容器与类元组存储
- Backend portability and SYCL-style compatibility / 后端可移植性与 SYCL 风格兼容层
- SYCL interoperability / SYCL 互操作
- Intel Xe-specific behavior / Intel Xe 特定行为

## Dependencies / 依赖关系

- Direct includes / 直接包含:
  - `unordered_set`
  - `windows.h`
  - `dlfcn.h`
  - `filesystem`
  - `experimental/filesystem`
  - `fstream`
  - `random`
  - `sycl/image.hpp`
  - `sycl/info/info_desc.hpp`
  - `sycl/nd_range.hpp`
  - `sycl/queue.hpp`
- Primary symbols / 主要符号: `kernel_function_info`, `path_lib_record`, `kernel_library`, `kernel_function`, `get_kernel_function_info`, `record_lib_path`, `remove_lib`, `unload_kernel_library`
- Dependency role / 依赖角色: Sits between CuTe algorithms and backend APIs, normalizing device, kernel, launch, memory, and math behavior across supported targets. / 位于 CuTe 算法与后端 API 之间，统一不同目标上的设备、内核、启动、内存和数学行为。
