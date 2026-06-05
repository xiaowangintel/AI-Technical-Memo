# copy_xe_legacy_U16.hpp — Code Analysis / 代码分析

## Source / 来源
- **Path:** `include/cute/arch/copy_xe_legacy_U16.hpp`
- **EN:** Provides legacy Intel Xe copy builtins, SPIR-V wrappers, or typed front-ends.
- **CN:** 提供传统 Intel Xe 拷贝内建、SPIR-V 封装或类型化前端。

## Line-by-Line Analysis / 逐行分析

### Lines 1-31
```cpp
/***************************************************************************************************
 * Copyright (c) 2024 - 2025 Codeplay Software Ltd. All rights reserved.
 * SPDX-License-Identifier: BSD-3-Clause
 *
 * Redistribution and use in source and binary forms, with or without
 * modification, are permitted provided that the following conditions are met:
 *
 * 1. Redistributions of source code must retain the above copyright notice, this
 * list of conditions and the following disclaimer.
 *
 * 2. Redistributions in binary form must reproduce the above copyright notice,
 * this list of conditions and the following disclaimer in the documentation
 * and/or other materials provided with the distribution.
 *
 * 3. Neither the name of the copyright holder nor the names of its
 * contributors may be used to endorse or promote products derived from
 * this software without specific prior written permission.
 *
 * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
 * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
 * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
 * DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
 * FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
 * DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
 * SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
 * CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
 * OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
 * OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
 *
 **************************************************************************************************/
#pragma once
```
- **EN:** Carries the BSD-3-Clause license banner and enables one-time inclusion with `#pragma once`.
- **CN:** 给出 BSD-3-Clause 许可证声明，并通过 `#pragma once` 启用一次性包含保护。

### Lines 33-34
```cpp
#include <cute/util/sycl_vec.hpp>
#include "cute/config.hpp"
```
- **EN:** Imports `cute/util/sycl_vec.hpp` (related definitions from `cute/util/sycl_vec.hpp`); `cute/config.hpp` (core CuTe configuration macros and portability annotations).
- **CN:** 引入 `cute/util/sycl_vec.hpp`（来自 `cute/util/sycl_vec.hpp` 的相关定义）；`cute/config.hpp`（CuTe 核心配置宏与可移植性标注）。

### Lines 36-40
```cpp
namespace cute
{
struct XE_2D_U16x1x16_LD_N {
  using BlockShape = Shape<_1, _16>;
  using inst_dtype = uint16_t;
```
- **EN:** Enters or leaves namespace scope `cute` so related symbols stay grouped.
- **CN:** 进入或离开命名空间作用域 `cute`，以便把相关符号组织在一起。

### Lines 42-53
```cpp
  template <class T>
  CUTE_HOST_DEVICE static void copy(const void *baseoffset, int width,
                                    int height, int pitch, intel::coord_t coord,
                                    T *dst) {
#if defined(CUTE_ARCH_COPY_XE_ENABLED)
    static_assert(sizeof(T) == 2, "Expected T to have size 2");
    detail::XeSubgroup2DBlockLoad<2, 16, 1, 1>{}(baseoffset, width, height, pitch, coord, dst);
#else
    CUTE_INVALID_CONTROL_PATH("Trying to use block loads on non-Xe hardware");
#endif
  }
};
```
- **EN:** Defines or forwards `CUTE_INVALID_CONTROL_PATH` as part of this header's executable interface.
- **CN:** 定义或转发 `CUTE_INVALID_CONTROL_PATH`，作为该头文件可执行接口的一部分。

### Lines 55-69
```cpp
struct XE_2D_U16x2x16_LD_N {
  using BlockShape = Shape<_2, _16>;

  template <class T>
  CUTE_HOST_DEVICE static void copy(const void *baseoffset, int width,
                                    int height, int pitch, intel::coord_t coord,
                                    T *dst) {
#if defined(CUTE_ARCH_COPY_XE_ENABLED)
    static_assert(sizeof(T) == 2, "Expected T to have size 2");
    detail::XeSubgroup2DBlockLoad<2, 16, 2, 1>{}(baseoffset, width, height, pitch, coord, dst);
#else
    CUTE_INVALID_CONTROL_PATH("Trying to use block loads on non-Xe hardware");
#endif
  }
};
```
- **EN:** Defines `XE_2D_U16x2x16_LD_N` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XE_2D_U16x2x16_LD_N`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 71-85
```cpp
struct XE_2D_U16x4x16_LD_N {
  using BlockShape = Shape<_4, _16>;

  template <class T>
  CUTE_HOST_DEVICE static void copy(const void *baseoffset, int width,
                                    int height, int pitch, intel::coord_t coord,
                                    T *dst) {
#if defined(CUTE_ARCH_COPY_XE_ENABLED)
    static_assert(sizeof(T) == 2, "Expected T to have size 2");
    detail::XeSubgroup2DBlockLoad<2, 16, 4, 1>{}(baseoffset, width, height, pitch, coord, dst);
#else
    CUTE_INVALID_CONTROL_PATH("Trying to use block loads on non-Xe hardware");
#endif
  }
};
```
- **EN:** Defines `XE_2D_U16x4x16_LD_N` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XE_2D_U16x4x16_LD_N`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 87-114
```cpp
struct XE_2D_U16x8x16_LD_N {
  using BlockShape = Shape<_8, _16>;

  template <class T>
  CUTE_HOST_DEVICE static void copy(const void *baseoffset, int width,
                                    int height, int pitch, intel::coord_t coord,
                                    T *dst) {
#if defined(CUTE_ARCH_COPY_XE_ENABLED)
    static_assert(sizeof(T) == 2, "Expected T to have size 2");
    detail::XeSubgroup2DBlockLoad<2, 16, 8, 1>{}(baseoffset, width, height, pitch, coord, dst);
#else
    CUTE_INVALID_CONTROL_PATH("Trying to use block loads on non-Xe hardware");
#endif
  }

  struct PREFETCH {
    CUTE_HOST_DEVICE static void copy(const void *baseoffset, int width,
                                      int height, int pitch,
                                      intel::coord_t coord) {
#if defined(CUTE_ARCH_COPY_XE_ENABLED)
    detail::XeSubgroup2DBlockPrefetch<2, 16, 8, 1>{}(baseoffset, width, height, pitch, coord);
#else
      CUTE_INVALID_CONTROL_PATH(
          "Trying to use block prefetch on non-Xe hardware");
#endif
    }
  };
};
```
- **EN:** Defines `XE_2D_U16x8x16_LD_N` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XE_2D_U16x8x16_LD_N`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 116-143
```cpp
struct XE_2D_U16x16x16_LD_N {
  using BlockShape = Shape<_16, _16>;

  template <class T>
  CUTE_HOST_DEVICE static void copy(const void *baseoffset, int width,
                                    int height, int pitch, intel::coord_t coord,
                                    T *dst) {
#if defined(CUTE_ARCH_COPY_XE_ENABLED)
    static_assert(sizeof(T) == 2, "Expected T to have size 2");
    detail::XeSubgroup2DBlockLoad<2, 16, 16, 1>{}(baseoffset, width, height, pitch, coord, dst);
#else
    CUTE_INVALID_CONTROL_PATH("Trying to use block loads on non-Xe hardware");
#endif
  }

  struct PREFETCH {
    CUTE_HOST_DEVICE static void copy(const void *baseoffset, int width,
                                      int height, int pitch,
                                      intel::coord_t coord) {
#if defined(CUTE_ARCH_COPY_XE_ENABLED)
    detail::XeSubgroup2DBlockPrefetch<2, 16, 16, 1>{}(baseoffset, width, height, pitch, coord);
#else
      CUTE_INVALID_CONTROL_PATH(
          "Trying to use block prefetch on non-Xe hardware");
#endif
    }
  };
};
```
- **EN:** Defines `XE_2D_U16x16x16_LD_N` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XE_2D_U16x16x16_LD_N`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 145-172
```cpp
struct XE_2D_U16x32x16_LD_N {
  using BlockShape = Shape<_32, _16>;

  template <class T>
  CUTE_HOST_DEVICE static void copy(const void *baseoffset, int width,
                                    int height, int pitch, intel::coord_t coord,
                                    T *dst) {
#if defined(CUTE_ARCH_COPY_XE_ENABLED)
    static_assert(sizeof(T) == 2, "Expected T to have size 2");
    detail::XeSubgroup2DBlockLoad<2, 16, 32, 1>{}(baseoffset, width, height, pitch, coord, dst);
#else
    CUTE_INVALID_CONTROL_PATH("Trying to use block loads on non-Xe hardware");
#endif
  }

  struct PREFETCH {
    CUTE_HOST_DEVICE static void copy(const void *baseoffset, int width,
                                      int height, int pitch,
                                      intel::coord_t coord) {
#if defined(CUTE_ARCH_COPY_XE_ENABLED)
    detail::XeSubgroup2DBlockPrefetch<2, 16, 32, 1>{}(baseoffset, width, height, pitch, coord);
#else
      CUTE_INVALID_CONTROL_PATH(
          "Trying to use block prefetch on non-Xe hardware");
#endif
    }
  };
};
```
- **EN:** Defines `XE_2D_U16x32x16_LD_N` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XE_2D_U16x32x16_LD_N`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 174-202
```cpp
struct XE_2D_U16x1x32_LD_N {
  using BlockShape = Shape<_1, _32>;
  using inst_dtype = uint16_t;

  template <class T>
  CUTE_HOST_DEVICE static void copy(const void *baseoffset, int width,
                                    int height, int pitch, intel::coord_t coord,
                                    T *dst) {
#if defined(CUTE_ARCH_COPY_XE_ENABLED)
    static_assert(sizeof(T) == 2, "Expected T to have size 2");
    detail::XeSubgroup2DBlockLoad<2, 16, 1, 2>{}(baseoffset, width, height, pitch, coord, dst);
#else
    CUTE_INVALID_CONTROL_PATH("Trying to use block loads on non-Xe hardware");
#endif
  }

  struct PREFETCH {
    CUTE_HOST_DEVICE static void copy(const void *baseoffset, int width,
                                      int height, int pitch,
                                      intel::coord_t coord) {
#if defined(CUTE_ARCH_COPY_XE_ENABLED)
    detail::XeSubgroup2DBlockPrefetch<2, 16, 1, 2>{}(baseoffset, width, height, pitch, coord);
#else
      CUTE_INVALID_CONTROL_PATH(
          "Trying to use block prefetch on non-Xe hardware");
#endif
    }
  };
};
```
- **EN:** Defines `XE_2D_U16x1x32_LD_N` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XE_2D_U16x1x32_LD_N`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 204-232
```cpp
struct XE_2D_U16x2x32_LD_N {
  using BlockShape = Shape<_2, _32>;
  using inst_dtype = uint16_t;

  template <class T>
  CUTE_HOST_DEVICE static void copy(const void *baseoffset, int width,
                                    int height, int pitch, intel::coord_t coord,
                                    T *dst) {
#if defined(CUTE_ARCH_COPY_XE_ENABLED)
    static_assert(sizeof(T) == 2, "Expected T to have size 2");
    detail::XeSubgroup2DBlockLoad<2, 16, 2, 2>{}(baseoffset, width, height, pitch, coord, dst);
#else
    CUTE_INVALID_CONTROL_PATH("Trying to use block loads on non-Xe hardware");
#endif
  }

  struct PREFETCH {
    CUTE_HOST_DEVICE static void copy(const void *baseoffset, int width,
                                      int height, int pitch,
                                      intel::coord_t coord) {
#if defined(CUTE_ARCH_COPY_XE_ENABLED)
    detail::XeSubgroup2DBlockPrefetch<2, 16, 2, 2>{}(baseoffset, width, height, pitch, coord);
#else
      CUTE_INVALID_CONTROL_PATH(
          "Trying to use block prefetch on non-Xe hardware");
#endif
    }
  };
};
```
- **EN:** Defines `XE_2D_U16x2x32_LD_N` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XE_2D_U16x2x32_LD_N`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 234-262
```cpp
struct XE_2D_Packed_U16x16x32_LD_N {
  using BlockShape = Shape<_16, _32>;
  using inst_dtype = uint32_t;

  template <class T>
  CUTE_HOST_DEVICE static void copy(const void *baseoffset, int width,
                                    int height, int pitch, intel::coord_t coord,
                                    T *dst) {
#if defined(CUTE_ARCH_COPY_XE_ENABLED)
    static_assert(sizeof(T) == 2, "Expected T to have size 2");
    detail::XeSubgroup2DBlockLoad<4, 16, 16, 1>{}(baseoffset, width, height, pitch, coord, dst);
#else
    CUTE_INVALID_CONTROL_PATH("Trying to use block loads on non-Xe hardware");
#endif
  }

  struct PREFETCH {
    CUTE_HOST_DEVICE static void copy(const void *baseoffset, int width,
                                      int height, int pitch,
                                      intel::coord_t coord) {
#if defined(CUTE_ARCH_COPY_XE_ENABLED)
    detail::XeSubgroup2DBlockPrefetch<2, 16, 16, 1>{}(baseoffset, width, height, pitch, coord);
#else
      CUTE_INVALID_CONTROL_PATH(
          "Trying to use block prefetch on non-Xe hardware");
#endif
    }
  };
};
```
- **EN:** Defines `XE_2D_Packed_U16x16x32_LD_N` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XE_2D_Packed_U16x16x32_LD_N`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 264-292
```cpp
struct XE_2D_U16x4x32_LD_N {
  using BlockShape = Shape<_4, _32>;
  using inst_dtype = uint16_t;

  template <class T>
  CUTE_HOST_DEVICE static void copy(const void *baseoffset, int width,
                                    int height, int pitch, intel::coord_t coord,
                                    T *dst) {
#if defined(CUTE_ARCH_COPY_XE_ENABLED)
    static_assert(sizeof(T) == 2, "Expected T to have size 2");
    detail::XeSubgroup2DBlockLoad<2, 16, 4, 2>{}(baseoffset, width, height, pitch, coord, dst);
#else
    CUTE_INVALID_CONTROL_PATH("Trying to use block loads on non-Xe hardware");
#endif
  }

  struct PREFETCH {
    CUTE_HOST_DEVICE static void copy(const void *baseoffset, int width,
                                      int height, int pitch,
                                      intel::coord_t coord) {
#if defined(CUTE_ARCH_COPY_XE_ENABLED)
    detail::XeSubgroup2DBlockPrefetch<2, 16, 4, 2>{}(baseoffset, width, height, pitch, coord);
#else
      CUTE_INVALID_CONTROL_PATH(
          "Trying to use block prefetch on non-Xe hardware");
#endif
    }
  };
};
```
- **EN:** Defines `XE_2D_U16x4x32_LD_N` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XE_2D_U16x4x32_LD_N`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 294-322
```cpp
struct XE_2D_U16x8x32_LD_N {
  using BlockShape = Shape<_8, _32>;
  using inst_dtype = uint16_t;

  template <class T>
  CUTE_HOST_DEVICE static void copy(const void *baseoffset, int width,
                                    int height, int pitch, intel::coord_t coord,
                                    T *dst) {
#if defined(CUTE_ARCH_COPY_XE_ENABLED)
    static_assert(sizeof(T) == 2, "Expected T to have size 2");
    detail::XeSubgroup2DBlockLoad<2, 16, 8, 2>{}(baseoffset, width, height, pitch, coord, dst);
#else
    CUTE_INVALID_CONTROL_PATH("Trying to use block loads on non-Xe hardware");
#endif
  }

  struct PREFETCH {
    CUTE_HOST_DEVICE static void copy(const void *baseoffset, int width,
                                      int height, int pitch,
                                      intel::coord_t coord) {
#if defined(CUTE_ARCH_COPY_XE_ENABLED)
    detail::XeSubgroup2DBlockPrefetch<2, 16, 8, 2>{}(baseoffset, width, height, pitch, coord);
#else
      CUTE_INVALID_CONTROL_PATH(
          "Trying to use block prefetch on non-Xe hardware");
#endif
    }
  };
};
```
- **EN:** Defines `XE_2D_U16x8x32_LD_N` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XE_2D_U16x8x32_LD_N`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 324-352
```cpp
struct XE_2D_U16x16x32_LD_N {
  using BlockShape = Shape<_16, _32>;
  using inst_dtype = uint16_t;

  template <class T>
  CUTE_HOST_DEVICE static void copy(const void *baseoffset, int width,
                                    int height, int pitch, intel::coord_t coord,
                                    T *dst) {
#if defined(CUTE_ARCH_COPY_XE_ENABLED)
    static_assert(sizeof(T) == 2, "Expected T to have size 2");
    detail::XeSubgroup2DBlockLoad<2, 16, 16, 2>{}(baseoffset, width, height, pitch, coord, dst);
#else
    CUTE_INVALID_CONTROL_PATH("Trying to use block loads on non-Xe hardware");
#endif
  }

  struct PREFETCH {
    CUTE_HOST_DEVICE static void copy(const void *baseoffset, int width,
                                      int height, int pitch,
                                      intel::coord_t coord) {
#if defined(CUTE_ARCH_COPY_XE_ENABLED)
    detail::XeSubgroup2DBlockPrefetch<2, 16, 16, 2>{}(baseoffset, width, height, pitch, coord);
#else
      CUTE_INVALID_CONTROL_PATH(
          "Trying to use block prefetch on non-Xe hardware");
#endif
    }
  };
};
```
- **EN:** Defines `XE_2D_U16x16x32_LD_N` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XE_2D_U16x16x32_LD_N`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 354-382
```cpp
struct XE_2D_U16x32x32_LD_N {
  using BlockShape = Shape<_32, _32>;
  using inst_dtype = uint16_t;

  template <class T>
  CUTE_HOST_DEVICE static void copy(const void *baseoffset, int width,
                                    int height, int pitch, intel::coord_t coord,
                                    T *dst) {
#if defined(CUTE_ARCH_COPY_XE_ENABLED)
    static_assert(sizeof(T) == 2, "Expected T to have size 2");
    detail::XeSubgroup2DBlockLoad<2, 16, 32, 2>{}(baseoffset, width, height, pitch, coord, dst);
#else
    CUTE_INVALID_CONTROL_PATH("Trying to use block loads on non-Xe hardware");
#endif
  }

  struct PREFETCH {
    CUTE_HOST_DEVICE static void copy(const void *baseoffset, int width,
                                      int height, int pitch,
                                      intel::coord_t coord) {
#if defined(CUTE_ARCH_COPY_XE_ENABLED)
    detail::XeSubgroup2DBlockPrefetch<2, 16, 32, 2>{}(baseoffset, width, height, pitch, coord);
#else
      CUTE_INVALID_CONTROL_PATH(
          "Trying to use block prefetch on non-Xe hardware");
#endif
    }
  };
};
```
- **EN:** Defines `XE_2D_U16x32x32_LD_N` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XE_2D_U16x32x32_LD_N`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 384-411
```cpp
struct XE_2D_Packed_U16x32x32_LD_N {
  using BlockShape = Shape<_32, _32>;
  using inst_dtype = uint32_t;
  template <class T>
  CUTE_HOST_DEVICE static void copy(const void *baseoffset, int width,
                                    int height, int pitch, intel::coord_t coord,
                                    T *dst) {
#if defined(CUTE_ARCH_COPY_XE_ENABLED)
    static_assert(sizeof(T) == 2, "Expected T to have size 2");
    detail::XeSubgroup2DBlockLoad<4, 16, 32, 1>{}(baseoffset, width, height, pitch, coord, dst);
#else
    CUTE_INVALID_CONTROL_PATH("Trying to use block loads on non-Xe hardware");
#endif
  }

  struct PREFETCH {
    CUTE_HOST_DEVICE static void copy(const void *baseoffset, int width,
                                      int height, int pitch,
                                      intel::coord_t coord) {
#if defined(CUTE_ARCH_COPY_XE_ENABLED)
    detail::XeSubgroup2DBlockPrefetch<2, 16, 32, 2>{}(baseoffset, width, height, pitch, coord);
#else
      CUTE_INVALID_CONTROL_PATH(
          "Trying to use block prefetch on non-Xe hardware");
#endif
    }
  };
};
```
- **EN:** Defines `XE_2D_Packed_U16x32x32_LD_N` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XE_2D_Packed_U16x32x32_LD_N`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 413-440
```cpp
struct XE_2D_U16x16x16_LD_V {
  using BlockShape = Shape<_16, _16>;

  template <class T>
  CUTE_HOST_DEVICE static void copy(const void *baseoffset, int width,
                                    int height, int pitch, intel::coord_t coord,
                                    T *dst) {
#if defined(CUTE_ARCH_COPY_XE_ENABLED)
    static_assert(sizeof(T) == 2, "Expected T to have size 2");
    detail::XeSubgroup2DBlockLoadTransform<2, 16, 16, 1>{}(baseoffset, width, height, pitch, coord, dst);
#else
    CUTE_INVALID_CONTROL_PATH("Trying to use block loads on non-Xe hardware");
#endif
  }

  struct PREFETCH {
    CUTE_HOST_DEVICE static void copy(const void *baseoffset, int width,
                                      int height, int pitch,
                                      intel::coord_t coord) {
#if defined(CUTE_ARCH_COPY_XE_ENABLED)
    detail::XeSubgroup2DBlockPrefetch<2, 16, 16, 1>{}(baseoffset, width, height, pitch, coord);
#else
      CUTE_INVALID_CONTROL_PATH(
          "Trying to use block prefetch on non-Xe hardware");
#endif
    }
  };
};
```
- **EN:** Defines `XE_2D_U16x16x16_LD_V` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XE_2D_U16x16x16_LD_V`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 442-469
```cpp
struct XE_2D_U16x32x16_LD_V {
  using BlockShape = Shape<_32, _16>;

  template <class T>
  CUTE_HOST_DEVICE static void copy(const void *baseoffset, int width,
                                    int height, int pitch, intel::coord_t coord,
                                    T *dst) {
#if defined(CUTE_ARCH_COPY_XE_ENABLED)
    static_assert(sizeof(T) == 2, "Expected T to have size 2");
    detail::XeSubgroup2DBlockLoadTransform<2, 16, 32, 1>{}(baseoffset, width, height, pitch, coord, dst);
#else
    CUTE_INVALID_CONTROL_PATH("Trying to use block loads on non-Xe hardware");
#endif
  }

  struct PREFETCH {
    CUTE_HOST_DEVICE static void copy(const void *baseoffset, int width,
                                      int height, int pitch,
                                      intel::coord_t coord) {
#if defined(CUTE_ARCH_COPY_XE_ENABLED)
    detail::XeSubgroup2DBlockPrefetch<2, 16, 32, 1>{}(baseoffset, width, height, pitch, coord);
#else
      CUTE_INVALID_CONTROL_PATH(
          "Trying to use block prefetch on non-Xe hardware");
#endif
    }
  };
};
```
- **EN:** Defines `XE_2D_U16x32x16_LD_V` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XE_2D_U16x32x16_LD_V`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 471-498
```cpp
struct XE_2D_U16x16x32_LD_V {
  using BlockShape = Shape<_16, _32>;

  template <class T>
  CUTE_HOST_DEVICE static void copy(const void *baseoffset, int width,
                                    int height, int pitch, intel::coord_t coord,
                                    T *dst) {
#if defined(CUTE_ARCH_COPY_XE_ENABLED)
    static_assert(sizeof(T) == 2, "Expected T to have size 2");
    detail::XeSubgroup2DBlockLoadTransform<2, 16, 16, 2>{}(baseoffset, width, height, pitch, coord, dst);
#else
    CUTE_INVALID_CONTROL_PATH("Trying to use block loads on non-Xe hardware");
#endif
  }

  struct PREFETCH {
    CUTE_HOST_DEVICE static void copy(const void *baseoffset, int width,
                                      int height, int pitch,
                                      intel::coord_t coord) {
#if defined(CUTE_ARCH_COPY_XE_ENABLED)
    detail::XeSubgroup2DBlockPrefetch<2, 16, 16, 2>{}(baseoffset, width, height, pitch, coord);
#else
      CUTE_INVALID_CONTROL_PATH(
          "Trying to use block prefetch on non-Xe hardware");
#endif
    }
  };
};
```
- **EN:** Defines `XE_2D_U16x16x32_LD_V` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XE_2D_U16x16x32_LD_V`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 500-527
```cpp
struct XE_2D_U16x32x32_LD_V {
  using BlockShape = Shape<_32, _32>;

  template <class T>
  CUTE_HOST_DEVICE static void copy(const void *baseoffset, int width,
                                    int height, int pitch, intel::coord_t coord,
                                    T *dst) {
#if defined(CUTE_ARCH_COPY_XE_ENABLED)
    static_assert(sizeof(T) == 2, "Expected T to have size 2");
    detail::XeSubgroup2DBlockLoadTransform<2, 16, 32, 2>{}(baseoffset, width, height, pitch, coord, dst);
#else
    CUTE_INVALID_CONTROL_PATH("Trying to use block loads on non-Xe hardware");
#endif
  }

  struct PREFETCH {
    CUTE_HOST_DEVICE static void copy(const void *baseoffset, int width,
                                      int height, int pitch,
                                      intel::coord_t coord) {
#if defined(CUTE_ARCH_COPY_XE_ENABLED)
    detail::XeSubgroup2DBlockPrefetch<2, 16, 32, 2>{}(baseoffset, width, height, pitch, coord);
#else
      CUTE_INVALID_CONTROL_PATH(
          "Trying to use block prefetch on non-Xe hardware");
#endif
    }
  };
};
```
- **EN:** Defines `XE_2D_U16x32x32_LD_V` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XE_2D_U16x32x32_LD_V`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 529-546
```cpp
struct XE_2D_U16x16x8_LD_T {
  using BlockShape = Shape<_8, _16>;
  using inst_dtype = uint32_t;

  static constexpr bool is_transpose = true;

  template <class T>
  CUTE_HOST_DEVICE static void copy(const void *baseoffset, int width,
                                    int height, int pitch, intel::coord_t coord,
                                    T *dst) {
#if defined(CUTE_ARCH_COPY_XE_ENABLED)
    static_assert(sizeof(T) == 2, "Expected T to have size 4");
    detail::XeSubgroup2DBlockLoadTranspose<4, 4, 16, 1>{}(baseoffset, width, height, pitch, coord, dst);
#else
    CUTE_INVALID_CONTROL_PATH("Trying to use block loads on non-Xe hardware");
#endif
  }
};
```
- **EN:** Defines `XE_2D_U16x16x8_LD_T` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XE_2D_U16x16x8_LD_T`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 548-565
```cpp
struct XE_2D_U16x16x16_LD_T {
  using BlockShape = Shape<_16, _16>;
  using inst_dtype = uint32_t;

  static constexpr bool is_transpose = true;

  template <class T>
  CUTE_HOST_DEVICE static void copy(const void *baseoffset, int width,
                                    int height, int pitch, intel::coord_t coord,
                                    T *dst) {
#if defined(CUTE_ARCH_COPY_XE_ENABLED)
    static_assert(sizeof(T) == 2, "Expected T to have size 2");
    detail::XeSubgroup2DBlockLoadTranspose<4, 8, 16, 1>{}(baseoffset, width, height, pitch, coord, dst);
#else
    CUTE_INVALID_CONTROL_PATH("Trying to use block loads on non-Xe hardware");
#endif
  }
};
```
- **EN:** Defines `XE_2D_U16x16x16_LD_T` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XE_2D_U16x16x16_LD_T`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 567-581
```cpp
struct XE_2D_U16x1x16_ST_N {
  using BlockShape = Shape<_1, _16>;

  template <class T>
  CUTE_HOST_DEVICE static void copy(void *baseoffset, int width, int height,
                                    int pitch, intel::coord_t coord,
                                    const T *src) {
#if defined(CUTE_ARCH_COPY_XE_ENABLED)
    // static_assert(sizeof(T) == 2, "Expected T to have size 2");
    detail::XeSubgroup2DBlockStore<2, 16, 1, 1>{}(baseoffset, width, height, pitch, coord, src);
#else
    CUTE_INVALID_CONTROL_PATH("Trying to use block loads on non-Xe hardware");
#endif
  }
};
```
- **EN:** Defines `XE_2D_U16x1x16_ST_N` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XE_2D_U16x1x16_ST_N`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 583-597
```cpp
struct XE_2D_U16x2x16_ST_N {
  using BlockShape = Shape<_2, _16>;

  template <class T>
  CUTE_HOST_DEVICE static void copy(void *baseoffset, int width, int height,
                                    int pitch, intel::coord_t coord,
                                    const T *src) {
#if defined(CUTE_ARCH_COPY_XE_ENABLED)
    // static_assert(sizeof(T) == 2, "Expected T to have size 2");
    detail::XeSubgroup2DBlockStore<2, 16, 2, 1>{}(baseoffset, width, height, pitch, coord, src);
#else
    CUTE_INVALID_CONTROL_PATH("Trying to use block loads on non-Xe hardware");
#endif
  }
};
```
- **EN:** Defines `XE_2D_U16x2x16_ST_N` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XE_2D_U16x2x16_ST_N`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 599-613
```cpp
struct XE_2D_U16x4x16_ST_N {
  using BlockShape = Shape<_4, _16>;

  template <class T>
  CUTE_HOST_DEVICE static void copy(void *baseoffset, int width, int height,
                                    int pitch, intel::coord_t coord,
                                    const T *src) {
#if defined(CUTE_ARCH_COPY_XE_ENABLED)
    // static_assert(sizeof(T) == 2, "Expected T to have size 2");
    detail::XeSubgroup2DBlockStore<2, 16, 4, 1>{}(baseoffset, width, height, pitch, coord, src);
#else
    CUTE_INVALID_CONTROL_PATH("Trying to use block loads on non-Xe hardware");
#endif
  }
};
```
- **EN:** Defines `XE_2D_U16x4x16_ST_N` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XE_2D_U16x4x16_ST_N`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 615-630
```cpp
struct XE_2D_U16x8x16_ST_N {
  using BlockShape = Shape<_8, _16>;

  template <class T>
  CUTE_HOST_DEVICE static void copy(void *baseoffset, int width, int height,
                                    int pitch, intel::coord_t coord,
                                    const T *src) {
#if defined(CUTE_ARCH_COPY_XE_ENABLED)
    // static_assert(sizeof(T) == 2, "Expected T to have size 2");
    detail::XeSubgroup2DBlockStore<2, 16, 8, 1>{}(baseoffset, width, height, pitch, coord, src);
#else
    CUTE_INVALID_CONTROL_PATH("Trying to use block loads on non-Xe hardware");
#endif
  }
};
} // end namespace cute
```
- **EN:** Defines `XE_2D_U16x8x16_ST_N` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XE_2D_U16x8x16_ST_N`，把该头文件这一部分所需的类型、常量与行为组织在一起。

## Key Concepts / 关键概念

- **EN:** Copy traits/policies separate abstract data movement from the concrete instruction selected underneath.
  **CN:** Copy trait/策略把抽象数据搬运与底层实际选用的指令解耦。
- **EN:** Compile-time assertions encode hardware and type constraints directly in the API surface.
  **CN:** 编译期断言把硬件与类型约束直接编码到 API 表面。
- **EN:** Prefetch paths try to reduce latency by staging data or metadata early.
  **CN:** 预取路径尝试通过提前准备数据或元数据来降低延迟。
- **EN:** The Xe/SYCL path bridges CuTe abstractions to Intel GPU builtins or SPIR-V operations.
  **CN:** Xe/SYCL 路径把 CuTe 抽象桥接到 Intel GPU 内建函数或 SPIR-V 操作。

## Dependencies / 依赖关系

- **EN:** `cute/util/sycl_vec.hpp` supplies related definitions from `cute/util/sycl_vec.hpp`.
  **CN:** `cute/util/sycl_vec.hpp` 提供了来自 `cute/util/sycl_vec.hpp` 的相关定义。
- **EN:** `cute/config.hpp` supplies core CuTe configuration macros and portability annotations.
  **CN:** `cute/config.hpp` 提供了CuTe 核心配置宏与可移植性标注。
