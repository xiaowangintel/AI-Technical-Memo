# wasm_simd128.c — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `cross-project-tests/intrinsic-header-tests/wasm_simd128.c`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose / 目的**:
  - **EN**: Implements cross-project checks that validate compiler intrinsic headers and their generated interfaces.
  - **CN**: 实现跨项目检查，用于验证编译器 intrinsic 头文件及其生成接口。

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

````c
// REQUIRES: webassembly-registered-target
// expected-no-diagnostics

// RUN: %clang %s -O2 -S -o - -target wasm32-unknown-unknown \
// RUN: -msimd128 -mrelaxed-simd -mfp16 -Wcast-qual -Werror | FileCheck %s

#include <wasm_simd128.h>

// CHECK-LABEL: test_v128_load:
// CHECK: v128.load 0:p2align=0{{$}}
v128_t test_v128_load(const void *mem) { return wasm_v128_load(mem); }

// CHECK-LABEL: test_v128_load8_splat:
// CHECK: v128.load8_splat 0{{$}}
v128_t test_v128_load8_splat(const void *mem) {
  return wasm_v128_load8_splat(mem);
}

// CHECK-LABEL: test_v128_load16_splat:
// CHECK: v128.load16_splat 0:p2align=0{{$}}
v128_t test_v128_load16_splat(const void *mem) {
  return wasm_v128_load16_splat(mem);
}

````
- **L1 EN**: Comment documents nearby intent or constraints: `REQUIRES: webassembly-registered-target`.
  **L1 CN**: 注释说明附近代码的意图或约束：`REQUIRES: webassembly-registered-target`。
- **L2 EN**: Comment documents nearby intent or constraints: `expected-no-diagnostics`.
  **L2 CN**: 注释说明附近代码的意图或约束：`expected-no-diagnostics`。
- **L3 EN**: Blank line separating nearby declarations or logic.
  **L3 CN**: 空行，用于分隔相邻声明或逻辑。
- **L4 EN**: Comment documents nearby intent or constraints: `RUN: %clang %s -O2 -S -o - -target wasm32-unknown-unknown \`.
  **L4 CN**: 注释说明附近代码的意图或约束：`RUN: %clang %s -O2 -S -o - -target wasm32-unknown-unknown \`。
- **L5 EN**: Comment documents nearby intent or constraints: `RUN: -msimd128 -mrelaxed-simd -mfp16 -Wcast-qual -Werror | FileCheck %s`.
  **L5 CN**: 注释说明附近代码的意图或约束：`RUN: -msimd128 -mrelaxed-simd -mfp16 -Wcast-qual -Werror | FileCheck %s`。
- **L6 EN**: Blank line separating nearby declarations or logic.
  **L6 CN**: 空行，用于分隔相邻声明或逻辑。
- **L7 EN**: Includes <wasm_simd128.h> to access C or C++ standard library facilities.
  **L7 CN**: 引入 <wasm_simd128.h> 以使用 C 或 C++ 标准库设施。
- **L8 EN**: Blank line separating nearby declarations or logic.
  **L8 CN**: 空行，用于分隔相邻声明或逻辑。
- **L9 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_v128_load:`.
  **L9 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_v128_load:`。
- **L10 EN**: Comment documents nearby intent or constraints: `CHECK: v128.load 0:p2align=0{{$}}`.
  **L10 CN**: 注释说明附近代码的意图或约束：`CHECK: v128.load 0:p2align=0{{$}}`。
- **L11 EN**: Starts a function or method definition for `test_v128_load`.
  **L11 CN**: 开始定义函数或方法 `test_v128_load`。
- **L12 EN**: Blank line separating nearby declarations or logic.
  **L12 CN**: 空行，用于分隔相邻声明或逻辑。
- **L13 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_v128_load8_splat:`.
  **L13 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_v128_load8_splat:`。
- **L14 EN**: Comment documents nearby intent or constraints: `CHECK: v128.load8_splat 0{{$}}`.
  **L14 CN**: 注释说明附近代码的意图或约束：`CHECK: v128.load8_splat 0{{$}}`。
- **L15 EN**: Starts a function or method definition for `test_v128_load8_splat`.
  **L15 CN**: 开始定义函数或方法 `test_v128_load8_splat`。
- **L16 EN**: Returns from the current function with `wasm_v128_load8_splat(mem)`.
  **L16 CN**: 以 `wasm_v128_load8_splat(mem)` 从当前函数返回。
- **L17 EN**: Closes the current lexical scope or compound statement.
  **L17 CN**: 结束当前词法作用域或复合语句块。
- **L18 EN**: Blank line separating nearby declarations or logic.
  **L18 CN**: 空行，用于分隔相邻声明或逻辑。
- **L19 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_v128_load16_splat:`.
  **L19 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_v128_load16_splat:`。
- **L20 EN**: Comment documents nearby intent or constraints: `CHECK: v128.load16_splat 0:p2align=0{{$}}`.
  **L20 CN**: 注释说明附近代码的意图或约束：`CHECK: v128.load16_splat 0:p2align=0{{$}}`。
- **L21 EN**: Starts a function or method definition for `test_v128_load16_splat`.
  **L21 CN**: 开始定义函数或方法 `test_v128_load16_splat`。
- **L22 EN**: Returns from the current function with `wasm_v128_load16_splat(mem)`.
  **L22 CN**: 以 `wasm_v128_load16_splat(mem)` 从当前函数返回。
- **L23 EN**: Closes the current lexical scope or compound statement.
  **L23 CN**: 结束当前词法作用域或复合语句块。
- **L24 EN**: Blank line separating nearby declarations or logic.
  **L24 CN**: 空行，用于分隔相邻声明或逻辑。

### Lines 25-48

````c
// CHECK-LABEL: test_v128_load32_splat:
// CHECK: v128.load32_splat 0:p2align=0{{$}}
v128_t test_v128_load32_splat(const void *mem) {
  return wasm_v128_load32_splat(mem);
}

// CHECK-LABEL: test_v128_load64_splat:
// CHECK: v128.load64_splat 0:p2align=0{{$}}
v128_t test_v128_load64_splat(const void *mem) {
  return wasm_v128_load64_splat(mem);
}

// CHECK-LABEL: test_i16x8_load8x8:
// CHECK: i16x8.load8x8_s 0:p2align=0{{$}}
v128_t test_i16x8_load8x8(const void *mem) { return wasm_i16x8_load8x8(mem); }

// CHECK-LABEL: test_u16x8_load8x8:
// CHECK: i16x8.load8x8_u 0:p2align=0{{$}}
v128_t test_u16x8_load8x8(const void *mem) { return wasm_u16x8_load8x8(mem); }

// CHECK-LABEL: test_i32x4_load16x4:
// CHECK: i32x4.load16x4_s 0:p2align=0{{$}}
v128_t test_i32x4_load16x4(const void *mem) { return wasm_i32x4_load16x4(mem); }

````
- **L25 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_v128_load32_splat:`.
  **L25 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_v128_load32_splat:`。
- **L26 EN**: Comment documents nearby intent or constraints: `CHECK: v128.load32_splat 0:p2align=0{{$}}`.
  **L26 CN**: 注释说明附近代码的意图或约束：`CHECK: v128.load32_splat 0:p2align=0{{$}}`。
- **L27 EN**: Starts a function or method definition for `test_v128_load32_splat`.
  **L27 CN**: 开始定义函数或方法 `test_v128_load32_splat`。
- **L28 EN**: Returns from the current function with `wasm_v128_load32_splat(mem)`.
  **L28 CN**: 以 `wasm_v128_load32_splat(mem)` 从当前函数返回。
- **L29 EN**: Closes the current lexical scope or compound statement.
  **L29 CN**: 结束当前词法作用域或复合语句块。
- **L30 EN**: Blank line separating nearby declarations or logic.
  **L30 CN**: 空行，用于分隔相邻声明或逻辑。
- **L31 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_v128_load64_splat:`.
  **L31 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_v128_load64_splat:`。
- **L32 EN**: Comment documents nearby intent or constraints: `CHECK: v128.load64_splat 0:p2align=0{{$}}`.
  **L32 CN**: 注释说明附近代码的意图或约束：`CHECK: v128.load64_splat 0:p2align=0{{$}}`。
- **L33 EN**: Starts a function or method definition for `test_v128_load64_splat`.
  **L33 CN**: 开始定义函数或方法 `test_v128_load64_splat`。
- **L34 EN**: Returns from the current function with `wasm_v128_load64_splat(mem)`.
  **L34 CN**: 以 `wasm_v128_load64_splat(mem)` 从当前函数返回。
- **L35 EN**: Closes the current lexical scope or compound statement.
  **L35 CN**: 结束当前词法作用域或复合语句块。
- **L36 EN**: Blank line separating nearby declarations or logic.
  **L36 CN**: 空行，用于分隔相邻声明或逻辑。
- **L37 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i16x8_load8x8:`.
  **L37 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i16x8_load8x8:`。
- **L38 EN**: Comment documents nearby intent or constraints: `CHECK: i16x8.load8x8_s 0:p2align=0{{$}}`.
  **L38 CN**: 注释说明附近代码的意图或约束：`CHECK: i16x8.load8x8_s 0:p2align=0{{$}}`。
- **L39 EN**: Starts a function or method definition for `test_i16x8_load8x8`.
  **L39 CN**: 开始定义函数或方法 `test_i16x8_load8x8`。
- **L40 EN**: Blank line separating nearby declarations or logic.
  **L40 CN**: 空行，用于分隔相邻声明或逻辑。
- **L41 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_u16x8_load8x8:`.
  **L41 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_u16x8_load8x8:`。
- **L42 EN**: Comment documents nearby intent or constraints: `CHECK: i16x8.load8x8_u 0:p2align=0{{$}}`.
  **L42 CN**: 注释说明附近代码的意图或约束：`CHECK: i16x8.load8x8_u 0:p2align=0{{$}}`。
- **L43 EN**: Starts a function or method definition for `test_u16x8_load8x8`.
  **L43 CN**: 开始定义函数或方法 `test_u16x8_load8x8`。
- **L44 EN**: Blank line separating nearby declarations or logic.
  **L44 CN**: 空行，用于分隔相邻声明或逻辑。
- **L45 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i32x4_load16x4:`.
  **L45 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i32x4_load16x4:`。
- **L46 EN**: Comment documents nearby intent or constraints: `CHECK: i32x4.load16x4_s 0:p2align=0{{$}}`.
  **L46 CN**: 注释说明附近代码的意图或约束：`CHECK: i32x4.load16x4_s 0:p2align=0{{$}}`。
- **L47 EN**: Starts a function or method definition for `test_i32x4_load16x4`.
  **L47 CN**: 开始定义函数或方法 `test_i32x4_load16x4`。
- **L48 EN**: Blank line separating nearby declarations or logic.
  **L48 CN**: 空行，用于分隔相邻声明或逻辑。

### Lines 49-72

````c
// CHECK-LABEL: test_u32x4_load16x4:
// CHECK: i32x4.load16x4_u 0:p2align=0{{$}}
v128_t test_u32x4_load16x4(const void *mem) { return wasm_u32x4_load16x4(mem); }

// CHECK-LABEL: test_i64x2_load32x2:
// CHECK: i64x2.load32x2_s 0:p2align=0{{$}}
v128_t test_i64x2_load32x2(const void *mem) { return wasm_i64x2_load32x2(mem); }

// CHECK-LABEL: test_u64x2_load32x2:
// CHECK: i64x2.load32x2_u 0:p2align=0{{$}}
v128_t test_u64x2_load32x2(const void *mem) { return wasm_u64x2_load32x2(mem); }

// CHECK-LABEL: test_v128_load32_zero:
// CHECK: v128.load32_zero 0:p2align=0{{$}}
v128_t test_v128_load32_zero(const void *mem) {
  return wasm_v128_load32_zero(mem);
}

// CHECK-LABEL: test_v128_load64_zero:
// CHECK: v128.load64_zero 0:p2align=0{{$}}
v128_t test_v128_load64_zero(const void *mem) {
  return wasm_v128_load64_zero(mem);
}

````
- **L49 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_u32x4_load16x4:`.
  **L49 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_u32x4_load16x4:`。
- **L50 EN**: Comment documents nearby intent or constraints: `CHECK: i32x4.load16x4_u 0:p2align=0{{$}}`.
  **L50 CN**: 注释说明附近代码的意图或约束：`CHECK: i32x4.load16x4_u 0:p2align=0{{$}}`。
- **L51 EN**: Starts a function or method definition for `test_u32x4_load16x4`.
  **L51 CN**: 开始定义函数或方法 `test_u32x4_load16x4`。
- **L52 EN**: Blank line separating nearby declarations or logic.
  **L52 CN**: 空行，用于分隔相邻声明或逻辑。
- **L53 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i64x2_load32x2:`.
  **L53 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i64x2_load32x2:`。
- **L54 EN**: Comment documents nearby intent or constraints: `CHECK: i64x2.load32x2_s 0:p2align=0{{$}}`.
  **L54 CN**: 注释说明附近代码的意图或约束：`CHECK: i64x2.load32x2_s 0:p2align=0{{$}}`。
- **L55 EN**: Starts a function or method definition for `test_i64x2_load32x2`.
  **L55 CN**: 开始定义函数或方法 `test_i64x2_load32x2`。
- **L56 EN**: Blank line separating nearby declarations or logic.
  **L56 CN**: 空行，用于分隔相邻声明或逻辑。
- **L57 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_u64x2_load32x2:`.
  **L57 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_u64x2_load32x2:`。
- **L58 EN**: Comment documents nearby intent or constraints: `CHECK: i64x2.load32x2_u 0:p2align=0{{$}}`.
  **L58 CN**: 注释说明附近代码的意图或约束：`CHECK: i64x2.load32x2_u 0:p2align=0{{$}}`。
- **L59 EN**: Starts a function or method definition for `test_u64x2_load32x2`.
  **L59 CN**: 开始定义函数或方法 `test_u64x2_load32x2`。
- **L60 EN**: Blank line separating nearby declarations or logic.
  **L60 CN**: 空行，用于分隔相邻声明或逻辑。
- **L61 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_v128_load32_zero:`.
  **L61 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_v128_load32_zero:`。
- **L62 EN**: Comment documents nearby intent or constraints: `CHECK: v128.load32_zero 0:p2align=0{{$}}`.
  **L62 CN**: 注释说明附近代码的意图或约束：`CHECK: v128.load32_zero 0:p2align=0{{$}}`。
- **L63 EN**: Starts a function or method definition for `test_v128_load32_zero`.
  **L63 CN**: 开始定义函数或方法 `test_v128_load32_zero`。
- **L64 EN**: Returns from the current function with `wasm_v128_load32_zero(mem)`.
  **L64 CN**: 以 `wasm_v128_load32_zero(mem)` 从当前函数返回。
- **L65 EN**: Closes the current lexical scope or compound statement.
  **L65 CN**: 结束当前词法作用域或复合语句块。
- **L66 EN**: Blank line separating nearby declarations or logic.
  **L66 CN**: 空行，用于分隔相邻声明或逻辑。
- **L67 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_v128_load64_zero:`.
  **L67 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_v128_load64_zero:`。
- **L68 EN**: Comment documents nearby intent or constraints: `CHECK: v128.load64_zero 0:p2align=0{{$}}`.
  **L68 CN**: 注释说明附近代码的意图或约束：`CHECK: v128.load64_zero 0:p2align=0{{$}}`。
- **L69 EN**: Starts a function or method definition for `test_v128_load64_zero`.
  **L69 CN**: 开始定义函数或方法 `test_v128_load64_zero`。
- **L70 EN**: Returns from the current function with `wasm_v128_load64_zero(mem)`.
  **L70 CN**: 以 `wasm_v128_load64_zero(mem)` 从当前函数返回。
- **L71 EN**: Closes the current lexical scope or compound statement.
  **L71 CN**: 结束当前词法作用域或复合语句块。
- **L72 EN**: Blank line separating nearby declarations or logic.
  **L72 CN**: 空行，用于分隔相邻声明或逻辑。

### Lines 73-96

````c
// CHECK-LABEL: test_v128_load8_lane:
// CHECK: v128.load8_lane 0, 15{{$}}
v128_t test_v128_load8_lane(uint8_t *ptr, v128_t vec) {
  return wasm_v128_load8_lane(ptr, vec, 15);
}

// CHECK-LABEL: test_v128_load16_lane:
// CHECK: v128.load16_lane 0:p2align=0, 7{{$}}
v128_t test_v128_load16_lane(uint16_t *ptr, v128_t vec) {
  return wasm_v128_load16_lane(ptr, vec, 7);
}

// CHECK-LABEL: test_v128_load32_lane:
// CHECK: v128.load32_lane 0:p2align=0, 3{{$}}
v128_t test_v128_load32_lane(uint32_t *ptr, v128_t vec) {
  return wasm_v128_load32_lane(ptr, vec, 3);
}

// CHECK-LABEL: test_v128_load64_lane:
// CHECK: v128.load64_lane 0:p2align=0, 1{{$}}
v128_t test_v128_load64_lane(uint64_t *ptr, v128_t vec) {
  return wasm_v128_load64_lane(ptr, vec, 1);
}

````
- **L73 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_v128_load8_lane:`.
  **L73 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_v128_load8_lane:`。
- **L74 EN**: Comment documents nearby intent or constraints: `CHECK: v128.load8_lane 0, 15{{$}}`.
  **L74 CN**: 注释说明附近代码的意图或约束：`CHECK: v128.load8_lane 0, 15{{$}}`。
- **L75 EN**: Starts a function or method definition for `test_v128_load8_lane`.
  **L75 CN**: 开始定义函数或方法 `test_v128_load8_lane`。
- **L76 EN**: Returns from the current function with `wasm_v128_load8_lane(ptr, vec, 15)`.
  **L76 CN**: 以 `wasm_v128_load8_lane(ptr, vec, 15)` 从当前函数返回。
- **L77 EN**: Closes the current lexical scope or compound statement.
  **L77 CN**: 结束当前词法作用域或复合语句块。
- **L78 EN**: Blank line separating nearby declarations or logic.
  **L78 CN**: 空行，用于分隔相邻声明或逻辑。
- **L79 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_v128_load16_lane:`.
  **L79 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_v128_load16_lane:`。
- **L80 EN**: Comment documents nearby intent or constraints: `CHECK: v128.load16_lane 0:p2align=0, 7{{$}}`.
  **L80 CN**: 注释说明附近代码的意图或约束：`CHECK: v128.load16_lane 0:p2align=0, 7{{$}}`。
- **L81 EN**: Starts a function or method definition for `test_v128_load16_lane`.
  **L81 CN**: 开始定义函数或方法 `test_v128_load16_lane`。
- **L82 EN**: Returns from the current function with `wasm_v128_load16_lane(ptr, vec, 7)`.
  **L82 CN**: 以 `wasm_v128_load16_lane(ptr, vec, 7)` 从当前函数返回。
- **L83 EN**: Closes the current lexical scope or compound statement.
  **L83 CN**: 结束当前词法作用域或复合语句块。
- **L84 EN**: Blank line separating nearby declarations or logic.
  **L84 CN**: 空行，用于分隔相邻声明或逻辑。
- **L85 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_v128_load32_lane:`.
  **L85 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_v128_load32_lane:`。
- **L86 EN**: Comment documents nearby intent or constraints: `CHECK: v128.load32_lane 0:p2align=0, 3{{$}}`.
  **L86 CN**: 注释说明附近代码的意图或约束：`CHECK: v128.load32_lane 0:p2align=0, 3{{$}}`。
- **L87 EN**: Starts a function or method definition for `test_v128_load32_lane`.
  **L87 CN**: 开始定义函数或方法 `test_v128_load32_lane`。
- **L88 EN**: Returns from the current function with `wasm_v128_load32_lane(ptr, vec, 3)`.
  **L88 CN**: 以 `wasm_v128_load32_lane(ptr, vec, 3)` 从当前函数返回。
- **L89 EN**: Closes the current lexical scope or compound statement.
  **L89 CN**: 结束当前词法作用域或复合语句块。
- **L90 EN**: Blank line separating nearby declarations or logic.
  **L90 CN**: 空行，用于分隔相邻声明或逻辑。
- **L91 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_v128_load64_lane:`.
  **L91 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_v128_load64_lane:`。
- **L92 EN**: Comment documents nearby intent or constraints: `CHECK: v128.load64_lane 0:p2align=0, 1{{$}}`.
  **L92 CN**: 注释说明附近代码的意图或约束：`CHECK: v128.load64_lane 0:p2align=0, 1{{$}}`。
- **L93 EN**: Starts a function or method definition for `test_v128_load64_lane`.
  **L93 CN**: 开始定义函数或方法 `test_v128_load64_lane`。
- **L94 EN**: Returns from the current function with `wasm_v128_load64_lane(ptr, vec, 1)`.
  **L94 CN**: 以 `wasm_v128_load64_lane(ptr, vec, 1)` 从当前函数返回。
- **L95 EN**: Closes the current lexical scope or compound statement.
  **L95 CN**: 结束当前词法作用域或复合语句块。
- **L96 EN**: Blank line separating nearby declarations or logic.
  **L96 CN**: 空行，用于分隔相邻声明或逻辑。

### Lines 97-120

````c
// CHECK-LABEL: test_v128_store:
// CHECK: v128.store 0:p2align=0{{$}}
void test_v128_store(void *mem, v128_t a) { return wasm_v128_store(mem, a); }

// CHECK-LABEL: test_v128_store8_lane:
// CHECK: v128.store8_lane 0, 15{{$}}
void test_v128_store8_lane(uint8_t *ptr, v128_t vec) {
  return wasm_v128_store8_lane(ptr, vec, 15);
}

// CHECK-LABEL: test_v128_store16_lane:
// CHECK: v128.store16_lane 0:p2align=0, 7{{$}}
void test_v128_store16_lane(uint16_t *ptr, v128_t vec) {
  return wasm_v128_store16_lane(ptr, vec, 7);
}

// CHECK-LABEL: test_v128_store32_lane:
// CHECK: v128.store32_lane 0:p2align=0, 3{{$}}
void test_v128_store32_lane(uint32_t *ptr, v128_t vec) {
  return wasm_v128_store32_lane(ptr, vec, 3);
}

// CHECK-LABEL: test_v128_store64_lane:
// CHECK: v128.store64_lane 0:p2align=0, 1{{$}}
````
- **L97 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_v128_store:`.
  **L97 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_v128_store:`。
- **L98 EN**: Comment documents nearby intent or constraints: `CHECK: v128.store 0:p2align=0{{$}}`.
  **L98 CN**: 注释说明附近代码的意图或约束：`CHECK: v128.store 0:p2align=0{{$}}`。
- **L99 EN**: Starts a function or method definition for `test_v128_store`.
  **L99 CN**: 开始定义函数或方法 `test_v128_store`。
- **L100 EN**: Blank line separating nearby declarations or logic.
  **L100 CN**: 空行，用于分隔相邻声明或逻辑。
- **L101 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_v128_store8_lane:`.
  **L101 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_v128_store8_lane:`。
- **L102 EN**: Comment documents nearby intent or constraints: `CHECK: v128.store8_lane 0, 15{{$}}`.
  **L102 CN**: 注释说明附近代码的意图或约束：`CHECK: v128.store8_lane 0, 15{{$}}`。
- **L103 EN**: Starts a function or method definition for `test_v128_store8_lane`.
  **L103 CN**: 开始定义函数或方法 `test_v128_store8_lane`。
- **L104 EN**: Returns from the current function with `wasm_v128_store8_lane(ptr, vec, 15)`.
  **L104 CN**: 以 `wasm_v128_store8_lane(ptr, vec, 15)` 从当前函数返回。
- **L105 EN**: Closes the current lexical scope or compound statement.
  **L105 CN**: 结束当前词法作用域或复合语句块。
- **L106 EN**: Blank line separating nearby declarations or logic.
  **L106 CN**: 空行，用于分隔相邻声明或逻辑。
- **L107 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_v128_store16_lane:`.
  **L107 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_v128_store16_lane:`。
- **L108 EN**: Comment documents nearby intent or constraints: `CHECK: v128.store16_lane 0:p2align=0, 7{{$}}`.
  **L108 CN**: 注释说明附近代码的意图或约束：`CHECK: v128.store16_lane 0:p2align=0, 7{{$}}`。
- **L109 EN**: Starts a function or method definition for `test_v128_store16_lane`.
  **L109 CN**: 开始定义函数或方法 `test_v128_store16_lane`。
- **L110 EN**: Returns from the current function with `wasm_v128_store16_lane(ptr, vec, 7)`.
  **L110 CN**: 以 `wasm_v128_store16_lane(ptr, vec, 7)` 从当前函数返回。
- **L111 EN**: Closes the current lexical scope or compound statement.
  **L111 CN**: 结束当前词法作用域或复合语句块。
- **L112 EN**: Blank line separating nearby declarations or logic.
  **L112 CN**: 空行，用于分隔相邻声明或逻辑。
- **L113 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_v128_store32_lane:`.
  **L113 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_v128_store32_lane:`。
- **L114 EN**: Comment documents nearby intent or constraints: `CHECK: v128.store32_lane 0:p2align=0, 3{{$}}`.
  **L114 CN**: 注释说明附近代码的意图或约束：`CHECK: v128.store32_lane 0:p2align=0, 3{{$}}`。
- **L115 EN**: Starts a function or method definition for `test_v128_store32_lane`.
  **L115 CN**: 开始定义函数或方法 `test_v128_store32_lane`。
- **L116 EN**: Returns from the current function with `wasm_v128_store32_lane(ptr, vec, 3)`.
  **L116 CN**: 以 `wasm_v128_store32_lane(ptr, vec, 3)` 从当前函数返回。
- **L117 EN**: Closes the current lexical scope or compound statement.
  **L117 CN**: 结束当前词法作用域或复合语句块。
- **L118 EN**: Blank line separating nearby declarations or logic.
  **L118 CN**: 空行，用于分隔相邻声明或逻辑。
- **L119 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_v128_store64_lane:`.
  **L119 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_v128_store64_lane:`。
- **L120 EN**: Comment documents nearby intent or constraints: `CHECK: v128.store64_lane 0:p2align=0, 1{{$}}`.
  **L120 CN**: 注释说明附近代码的意图或约束：`CHECK: v128.store64_lane 0:p2align=0, 1{{$}}`。

### Lines 121-144

````c
void test_v128_store64_lane(uint64_t *ptr, v128_t vec) {
  return wasm_v128_store64_lane(ptr, vec, 1);
}

// CHECK-LABEL: test_i8x16_make:
// CHECK:      local.get 0{{$}}
// CHECK-NEXT: i8x16.splat{{$}}
// CHECK-NEXT: local.get 1{{$}}
// CHECK-NEXT: i8x16.replace_lane 1{{$}}
// CHECK-NEXT: local.get 2{{$}}
// CHECK-NEXT: i8x16.replace_lane 2{{$}}
// CHECK-NEXT: local.get 3{{$}}
// CHECK-NEXT: i8x16.replace_lane 3{{$}}
// CHECK-NEXT: local.get 4{{$}}
// CHECK-NEXT: i8x16.replace_lane 4{{$}}
// CHECK-NEXT: local.get 5{{$}}
// CHECK-NEXT: i8x16.replace_lane 5{{$}}
// CHECK-NEXT: local.get 6{{$}}
// CHECK-NEXT: i8x16.replace_lane 6{{$}}
// CHECK-NEXT: local.get 7{{$}}
// CHECK-NEXT: i8x16.replace_lane 7{{$}}
// CHECK-NEXT: local.get 8{{$}}
// CHECK-NEXT: i8x16.replace_lane 8{{$}}
// CHECK-NEXT: local.get 9{{$}}
````
- **L121 EN**: Starts a function or method definition for `test_v128_store64_lane`.
  **L121 CN**: 开始定义函数或方法 `test_v128_store64_lane`。
- **L122 EN**: Returns from the current function with `wasm_v128_store64_lane(ptr, vec, 1)`.
  **L122 CN**: 以 `wasm_v128_store64_lane(ptr, vec, 1)` 从当前函数返回。
- **L123 EN**: Closes the current lexical scope or compound statement.
  **L123 CN**: 结束当前词法作用域或复合语句块。
- **L124 EN**: Blank line separating nearby declarations or logic.
  **L124 CN**: 空行，用于分隔相邻声明或逻辑。
- **L125 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i8x16_make:`.
  **L125 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i8x16_make:`。
- **L126 EN**: Comment documents nearby intent or constraints: `CHECK:      local.get 0{{$}}`.
  **L126 CN**: 注释说明附近代码的意图或约束：`CHECK:      local.get 0{{$}}`。
- **L127 EN**: Comment documents nearby intent or constraints: `CHECK-NEXT: i8x16.splat{{$}}`.
  **L127 CN**: 注释说明附近代码的意图或约束：`CHECK-NEXT: i8x16.splat{{$}}`。
- **L128 EN**: Comment documents nearby intent or constraints: `CHECK-NEXT: local.get 1{{$}}`.
  **L128 CN**: 注释说明附近代码的意图或约束：`CHECK-NEXT: local.get 1{{$}}`。
- **L129 EN**: Comment documents nearby intent or constraints: `CHECK-NEXT: i8x16.replace_lane 1{{$}}`.
  **L129 CN**: 注释说明附近代码的意图或约束：`CHECK-NEXT: i8x16.replace_lane 1{{$}}`。
- **L130 EN**: Comment documents nearby intent or constraints: `CHECK-NEXT: local.get 2{{$}}`.
  **L130 CN**: 注释说明附近代码的意图或约束：`CHECK-NEXT: local.get 2{{$}}`。
- **L131 EN**: Comment documents nearby intent or constraints: `CHECK-NEXT: i8x16.replace_lane 2{{$}}`.
  **L131 CN**: 注释说明附近代码的意图或约束：`CHECK-NEXT: i8x16.replace_lane 2{{$}}`。
- **L132 EN**: Comment documents nearby intent or constraints: `CHECK-NEXT: local.get 3{{$}}`.
  **L132 CN**: 注释说明附近代码的意图或约束：`CHECK-NEXT: local.get 3{{$}}`。
- **L133 EN**: Comment documents nearby intent or constraints: `CHECK-NEXT: i8x16.replace_lane 3{{$}}`.
  **L133 CN**: 注释说明附近代码的意图或约束：`CHECK-NEXT: i8x16.replace_lane 3{{$}}`。
- **L134 EN**: Comment documents nearby intent or constraints: `CHECK-NEXT: local.get 4{{$}}`.
  **L134 CN**: 注释说明附近代码的意图或约束：`CHECK-NEXT: local.get 4{{$}}`。
- **L135 EN**: Comment documents nearby intent or constraints: `CHECK-NEXT: i8x16.replace_lane 4{{$}}`.
  **L135 CN**: 注释说明附近代码的意图或约束：`CHECK-NEXT: i8x16.replace_lane 4{{$}}`。
- **L136 EN**: Comment documents nearby intent or constraints: `CHECK-NEXT: local.get 5{{$}}`.
  **L136 CN**: 注释说明附近代码的意图或约束：`CHECK-NEXT: local.get 5{{$}}`。
- **L137 EN**: Comment documents nearby intent or constraints: `CHECK-NEXT: i8x16.replace_lane 5{{$}}`.
  **L137 CN**: 注释说明附近代码的意图或约束：`CHECK-NEXT: i8x16.replace_lane 5{{$}}`。
- **L138 EN**: Comment documents nearby intent or constraints: `CHECK-NEXT: local.get 6{{$}}`.
  **L138 CN**: 注释说明附近代码的意图或约束：`CHECK-NEXT: local.get 6{{$}}`。
- **L139 EN**: Comment documents nearby intent or constraints: `CHECK-NEXT: i8x16.replace_lane 6{{$}}`.
  **L139 CN**: 注释说明附近代码的意图或约束：`CHECK-NEXT: i8x16.replace_lane 6{{$}}`。
- **L140 EN**: Comment documents nearby intent or constraints: `CHECK-NEXT: local.get 7{{$}}`.
  **L140 CN**: 注释说明附近代码的意图或约束：`CHECK-NEXT: local.get 7{{$}}`。
- **L141 EN**: Comment documents nearby intent or constraints: `CHECK-NEXT: i8x16.replace_lane 7{{$}}`.
  **L141 CN**: 注释说明附近代码的意图或约束：`CHECK-NEXT: i8x16.replace_lane 7{{$}}`。
- **L142 EN**: Comment documents nearby intent or constraints: `CHECK-NEXT: local.get 8{{$}}`.
  **L142 CN**: 注释说明附近代码的意图或约束：`CHECK-NEXT: local.get 8{{$}}`。
- **L143 EN**: Comment documents nearby intent or constraints: `CHECK-NEXT: i8x16.replace_lane 8{{$}}`.
  **L143 CN**: 注释说明附近代码的意图或约束：`CHECK-NEXT: i8x16.replace_lane 8{{$}}`。
- **L144 EN**: Comment documents nearby intent or constraints: `CHECK-NEXT: local.get 9{{$}}`.
  **L144 CN**: 注释说明附近代码的意图或约束：`CHECK-NEXT: local.get 9{{$}}`。

### Lines 145-168

````c
// CHECK-NEXT: i8x16.replace_lane 9{{$}}
// CHECK-NEXT: local.get 10{{$}}
// CHECK-NEXT: i8x16.replace_lane 10{{$}}
// CHECK-NEXT: local.get 11{{$}}
// CHECK-NEXT: i8x16.replace_lane 11{{$}}
// CHECK-NEXT: local.get 12{{$}}
// CHECK-NEXT: i8x16.replace_lane 12{{$}}
// CHECK-NEXT: local.get 13{{$}}
// CHECK-NEXT: i8x16.replace_lane 13{{$}}
// CHECK-NEXT: local.get 14{{$}}
// CHECK-NEXT: i8x16.replace_lane 14{{$}}
// CHECK-NEXT: local.get 15{{$}}
// CHECK-NEXT: i8x16.replace_lane 15{{$}}
v128_t test_i8x16_make(int8_t c0, int8_t c1, int8_t c2, int8_t c3, int8_t c4,
                       int8_t c5, int8_t c6, int8_t c7, int8_t c8, int8_t c9,
                       int8_t c10, int8_t c11, int8_t c12, int8_t c13,
                       int8_t c14, int8_t c15) {
  return wasm_i8x16_make(c0, c1, c2, c3, c4, c5, c6, c7, c8, c9, c10, c11, c12,
                         c13, c14, c15);
}

// CHECK-LABEL: test_i16x8_make:
// CHECK:      local.get 0{{$}}
// CHECK-NEXT: i16x8.splat{{$}}
````
- **L145 EN**: Comment documents nearby intent or constraints: `CHECK-NEXT: i8x16.replace_lane 9{{$}}`.
  **L145 CN**: 注释说明附近代码的意图或约束：`CHECK-NEXT: i8x16.replace_lane 9{{$}}`。
- **L146 EN**: Comment documents nearby intent or constraints: `CHECK-NEXT: local.get 10{{$}}`.
  **L146 CN**: 注释说明附近代码的意图或约束：`CHECK-NEXT: local.get 10{{$}}`。
- **L147 EN**: Comment documents nearby intent or constraints: `CHECK-NEXT: i8x16.replace_lane 10{{$}}`.
  **L147 CN**: 注释说明附近代码的意图或约束：`CHECK-NEXT: i8x16.replace_lane 10{{$}}`。
- **L148 EN**: Comment documents nearby intent or constraints: `CHECK-NEXT: local.get 11{{$}}`.
  **L148 CN**: 注释说明附近代码的意图或约束：`CHECK-NEXT: local.get 11{{$}}`。
- **L149 EN**: Comment documents nearby intent or constraints: `CHECK-NEXT: i8x16.replace_lane 11{{$}}`.
  **L149 CN**: 注释说明附近代码的意图或约束：`CHECK-NEXT: i8x16.replace_lane 11{{$}}`。
- **L150 EN**: Comment documents nearby intent or constraints: `CHECK-NEXT: local.get 12{{$}}`.
  **L150 CN**: 注释说明附近代码的意图或约束：`CHECK-NEXT: local.get 12{{$}}`。
- **L151 EN**: Comment documents nearby intent or constraints: `CHECK-NEXT: i8x16.replace_lane 12{{$}}`.
  **L151 CN**: 注释说明附近代码的意图或约束：`CHECK-NEXT: i8x16.replace_lane 12{{$}}`。
- **L152 EN**: Comment documents nearby intent or constraints: `CHECK-NEXT: local.get 13{{$}}`.
  **L152 CN**: 注释说明附近代码的意图或约束：`CHECK-NEXT: local.get 13{{$}}`。
- **L153 EN**: Comment documents nearby intent or constraints: `CHECK-NEXT: i8x16.replace_lane 13{{$}}`.
  **L153 CN**: 注释说明附近代码的意图或约束：`CHECK-NEXT: i8x16.replace_lane 13{{$}}`。
- **L154 EN**: Comment documents nearby intent or constraints: `CHECK-NEXT: local.get 14{{$}}`.
  **L154 CN**: 注释说明附近代码的意图或约束：`CHECK-NEXT: local.get 14{{$}}`。
- **L155 EN**: Comment documents nearby intent or constraints: `CHECK-NEXT: i8x16.replace_lane 14{{$}}`.
  **L155 CN**: 注释说明附近代码的意图或约束：`CHECK-NEXT: i8x16.replace_lane 14{{$}}`。
- **L156 EN**: Comment documents nearby intent or constraints: `CHECK-NEXT: local.get 15{{$}}`.
  **L156 CN**: 注释说明附近代码的意图或约束：`CHECK-NEXT: local.get 15{{$}}`。
- **L157 EN**: Comment documents nearby intent or constraints: `CHECK-NEXT: i8x16.replace_lane 15{{$}}`.
  **L157 CN**: 注释说明附近代码的意图或约束：`CHECK-NEXT: i8x16.replace_lane 15{{$}}`。
- **L158 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `v128_t test_i8x16_make(int8_t c0, int8_t c1, int8_t c2, int8_t c3, int8_t c4,`.
  **L158 CN**: 继续一个多行参数列表、初始化器或聚合项：`v128_t test_i8x16_make(int8_t c0, int8_t c1, int8_t c2, int8_t c3, int8_t c4,`。
- **L159 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `int8_t c5, int8_t c6, int8_t c7, int8_t c8, int8_t c9,`.
  **L159 CN**: 继续一个多行参数列表、初始化器或聚合项：`int8_t c5, int8_t c6, int8_t c7, int8_t c8, int8_t c9,`。
- **L160 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `int8_t c10, int8_t c11, int8_t c12, int8_t c13,`.
  **L160 CN**: 继续一个多行参数列表、初始化器或聚合项：`int8_t c10, int8_t c11, int8_t c12, int8_t c13,`。
- **L161 EN**: Continues the surrounding expression or declaration: `int8_t c14, int8_t c15) {`.
  **L161 CN**: 继续构造周围的表达式或声明：`int8_t c14, int8_t c15) {`。
- **L162 EN**: Returns from the current function with `wasm_i8x16_make(c0, c1, c2, c3, c4, c5, c6, c7, c8, c9, c10, c11, c12,`.
  **L162 CN**: 以 `wasm_i8x16_make(c0, c1, c2, c3, c4, c5, c6, c7, c8, c9, c10, c11, c12,` 从当前函数返回。
- **L163 EN**: Executes a standalone statement or declaration: `c13, c14, c15);`.
  **L163 CN**: 执行一条独立语句或声明：`c13, c14, c15);`。
- **L164 EN**: Closes the current lexical scope or compound statement.
  **L164 CN**: 结束当前词法作用域或复合语句块。
- **L165 EN**: Blank line separating nearby declarations or logic.
  **L165 CN**: 空行，用于分隔相邻声明或逻辑。
- **L166 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i16x8_make:`.
  **L166 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i16x8_make:`。
- **L167 EN**: Comment documents nearby intent or constraints: `CHECK:      local.get 0{{$}}`.
  **L167 CN**: 注释说明附近代码的意图或约束：`CHECK:      local.get 0{{$}}`。
- **L168 EN**: Comment documents nearby intent or constraints: `CHECK-NEXT: i16x8.splat{{$}}`.
  **L168 CN**: 注释说明附近代码的意图或约束：`CHECK-NEXT: i16x8.splat{{$}}`。

### Lines 169-192

````c
// CHECK-NEXT: local.get 1{{$}}
// CHECK-NEXT: i16x8.replace_lane 1{{$}}
// CHECK-NEXT: local.get 2{{$}}
// CHECK-NEXT: i16x8.replace_lane 2{{$}}
// CHECK-NEXT: local.get 3{{$}}
// CHECK-NEXT: i16x8.replace_lane 3{{$}}
// CHECK-NEXT: local.get 4{{$}}
// CHECK-NEXT: i16x8.replace_lane 4{{$}}
// CHECK-NEXT: local.get 5{{$}}
// CHECK-NEXT: i16x8.replace_lane 5{{$}}
// CHECK-NEXT: local.get 6{{$}}
// CHECK-NEXT: i16x8.replace_lane 6{{$}}
// CHECK-NEXT: local.get 7{{$}}
// CHECK-NEXT: i16x8.replace_lane 7{{$}}
v128_t test_i16x8_make(int16_t c0, int16_t c1, int16_t c2, int16_t c3,
                       int16_t c4, int16_t c5, int16_t c6, int16_t c7) {
  return wasm_i16x8_make(c0, c1, c2, c3, c4, c5, c6, c7);
}

// CHECK-LABEL: test_i32x4_make:
// CHECK:      local.get 0{{$}}
// CHECK-NEXT: i32x4.splat{{$}}
// CHECK-NEXT: local.get 1{{$}}
// CHECK-NEXT: i32x4.replace_lane 1{{$}}
````
- **L169 EN**: Comment documents nearby intent or constraints: `CHECK-NEXT: local.get 1{{$}}`.
  **L169 CN**: 注释说明附近代码的意图或约束：`CHECK-NEXT: local.get 1{{$}}`。
- **L170 EN**: Comment documents nearby intent or constraints: `CHECK-NEXT: i16x8.replace_lane 1{{$}}`.
  **L170 CN**: 注释说明附近代码的意图或约束：`CHECK-NEXT: i16x8.replace_lane 1{{$}}`。
- **L171 EN**: Comment documents nearby intent or constraints: `CHECK-NEXT: local.get 2{{$}}`.
  **L171 CN**: 注释说明附近代码的意图或约束：`CHECK-NEXT: local.get 2{{$}}`。
- **L172 EN**: Comment documents nearby intent or constraints: `CHECK-NEXT: i16x8.replace_lane 2{{$}}`.
  **L172 CN**: 注释说明附近代码的意图或约束：`CHECK-NEXT: i16x8.replace_lane 2{{$}}`。
- **L173 EN**: Comment documents nearby intent or constraints: `CHECK-NEXT: local.get 3{{$}}`.
  **L173 CN**: 注释说明附近代码的意图或约束：`CHECK-NEXT: local.get 3{{$}}`。
- **L174 EN**: Comment documents nearby intent or constraints: `CHECK-NEXT: i16x8.replace_lane 3{{$}}`.
  **L174 CN**: 注释说明附近代码的意图或约束：`CHECK-NEXT: i16x8.replace_lane 3{{$}}`。
- **L175 EN**: Comment documents nearby intent or constraints: `CHECK-NEXT: local.get 4{{$}}`.
  **L175 CN**: 注释说明附近代码的意图或约束：`CHECK-NEXT: local.get 4{{$}}`。
- **L176 EN**: Comment documents nearby intent or constraints: `CHECK-NEXT: i16x8.replace_lane 4{{$}}`.
  **L176 CN**: 注释说明附近代码的意图或约束：`CHECK-NEXT: i16x8.replace_lane 4{{$}}`。
- **L177 EN**: Comment documents nearby intent or constraints: `CHECK-NEXT: local.get 5{{$}}`.
  **L177 CN**: 注释说明附近代码的意图或约束：`CHECK-NEXT: local.get 5{{$}}`。
- **L178 EN**: Comment documents nearby intent or constraints: `CHECK-NEXT: i16x8.replace_lane 5{{$}}`.
  **L178 CN**: 注释说明附近代码的意图或约束：`CHECK-NEXT: i16x8.replace_lane 5{{$}}`。
- **L179 EN**: Comment documents nearby intent or constraints: `CHECK-NEXT: local.get 6{{$}}`.
  **L179 CN**: 注释说明附近代码的意图或约束：`CHECK-NEXT: local.get 6{{$}}`。
- **L180 EN**: Comment documents nearby intent or constraints: `CHECK-NEXT: i16x8.replace_lane 6{{$}}`.
  **L180 CN**: 注释说明附近代码的意图或约束：`CHECK-NEXT: i16x8.replace_lane 6{{$}}`。
- **L181 EN**: Comment documents nearby intent or constraints: `CHECK-NEXT: local.get 7{{$}}`.
  **L181 CN**: 注释说明附近代码的意图或约束：`CHECK-NEXT: local.get 7{{$}}`。
- **L182 EN**: Comment documents nearby intent or constraints: `CHECK-NEXT: i16x8.replace_lane 7{{$}}`.
  **L182 CN**: 注释说明附近代码的意图或约束：`CHECK-NEXT: i16x8.replace_lane 7{{$}}`。
- **L183 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `v128_t test_i16x8_make(int16_t c0, int16_t c1, int16_t c2, int16_t c3,`.
  **L183 CN**: 继续一个多行参数列表、初始化器或聚合项：`v128_t test_i16x8_make(int16_t c0, int16_t c1, int16_t c2, int16_t c3,`。
- **L184 EN**: Continues the surrounding expression or declaration: `int16_t c4, int16_t c5, int16_t c6, int16_t c7) {`.
  **L184 CN**: 继续构造周围的表达式或声明：`int16_t c4, int16_t c5, int16_t c6, int16_t c7) {`。
- **L185 EN**: Returns from the current function with `wasm_i16x8_make(c0, c1, c2, c3, c4, c5, c6, c7)`.
  **L185 CN**: 以 `wasm_i16x8_make(c0, c1, c2, c3, c4, c5, c6, c7)` 从当前函数返回。
- **L186 EN**: Closes the current lexical scope or compound statement.
  **L186 CN**: 结束当前词法作用域或复合语句块。
- **L187 EN**: Blank line separating nearby declarations or logic.
  **L187 CN**: 空行，用于分隔相邻声明或逻辑。
- **L188 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i32x4_make:`.
  **L188 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i32x4_make:`。
- **L189 EN**: Comment documents nearby intent or constraints: `CHECK:      local.get 0{{$}}`.
  **L189 CN**: 注释说明附近代码的意图或约束：`CHECK:      local.get 0{{$}}`。
- **L190 EN**: Comment documents nearby intent or constraints: `CHECK-NEXT: i32x4.splat{{$}}`.
  **L190 CN**: 注释说明附近代码的意图或约束：`CHECK-NEXT: i32x4.splat{{$}}`。
- **L191 EN**: Comment documents nearby intent or constraints: `CHECK-NEXT: local.get 1{{$}}`.
  **L191 CN**: 注释说明附近代码的意图或约束：`CHECK-NEXT: local.get 1{{$}}`。
- **L192 EN**: Comment documents nearby intent or constraints: `CHECK-NEXT: i32x4.replace_lane 1{{$}}`.
  **L192 CN**: 注释说明附近代码的意图或约束：`CHECK-NEXT: i32x4.replace_lane 1{{$}}`。

### Lines 193-216

````c
// CHECK-NEXT: local.get 2{{$}}
// CHECK-NEXT: i32x4.replace_lane 2{{$}}
// CHECK-NEXT: local.get 3{{$}}
// CHECK-NEXT: i32x4.replace_lane 3{{$}}
v128_t test_i32x4_make(int32_t c0, int32_t c1, int32_t c2, int32_t c3) {
  return wasm_i32x4_make(c0, c1, c2, c3);
}

// CHECK-LABEL: test_i64x2_make:
// CHECK:      local.get 0{{$}}
// CHECK-NEXT: i64x2.splat{{$}}
// CHECK-NEXT: local.get 1{{$}}
// CHECK-NEXT: i64x2.replace_lane 1{{$}}
v128_t test_i64x2_make(int64_t c0, int64_t c1) {
  return wasm_i64x2_make(c0, c1);
}

// CHECK-LABEL: test_f32x4_make:
// CHECK:      local.get 0{{$}}
// CHECK-NEXT: f32x4.splat{{$}}
// CHECK-NEXT: local.get 1{{$}}
// CHECK-NEXT: f32x4.replace_lane 1{{$}}
// CHECK-NEXT: local.get 2{{$}}
// CHECK-NEXT: f32x4.replace_lane 2{{$}}
````
- **L193 EN**: Comment documents nearby intent or constraints: `CHECK-NEXT: local.get 2{{$}}`.
  **L193 CN**: 注释说明附近代码的意图或约束：`CHECK-NEXT: local.get 2{{$}}`。
- **L194 EN**: Comment documents nearby intent or constraints: `CHECK-NEXT: i32x4.replace_lane 2{{$}}`.
  **L194 CN**: 注释说明附近代码的意图或约束：`CHECK-NEXT: i32x4.replace_lane 2{{$}}`。
- **L195 EN**: Comment documents nearby intent or constraints: `CHECK-NEXT: local.get 3{{$}}`.
  **L195 CN**: 注释说明附近代码的意图或约束：`CHECK-NEXT: local.get 3{{$}}`。
- **L196 EN**: Comment documents nearby intent or constraints: `CHECK-NEXT: i32x4.replace_lane 3{{$}}`.
  **L196 CN**: 注释说明附近代码的意图或约束：`CHECK-NEXT: i32x4.replace_lane 3{{$}}`。
- **L197 EN**: Starts a function or method definition for `test_i32x4_make`.
  **L197 CN**: 开始定义函数或方法 `test_i32x4_make`。
- **L198 EN**: Returns from the current function with `wasm_i32x4_make(c0, c1, c2, c3)`.
  **L198 CN**: 以 `wasm_i32x4_make(c0, c1, c2, c3)` 从当前函数返回。
- **L199 EN**: Closes the current lexical scope or compound statement.
  **L199 CN**: 结束当前词法作用域或复合语句块。
- **L200 EN**: Blank line separating nearby declarations or logic.
  **L200 CN**: 空行，用于分隔相邻声明或逻辑。
- **L201 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i64x2_make:`.
  **L201 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i64x2_make:`。
- **L202 EN**: Comment documents nearby intent or constraints: `CHECK:      local.get 0{{$}}`.
  **L202 CN**: 注释说明附近代码的意图或约束：`CHECK:      local.get 0{{$}}`。
- **L203 EN**: Comment documents nearby intent or constraints: `CHECK-NEXT: i64x2.splat{{$}}`.
  **L203 CN**: 注释说明附近代码的意图或约束：`CHECK-NEXT: i64x2.splat{{$}}`。
- **L204 EN**: Comment documents nearby intent or constraints: `CHECK-NEXT: local.get 1{{$}}`.
  **L204 CN**: 注释说明附近代码的意图或约束：`CHECK-NEXT: local.get 1{{$}}`。
- **L205 EN**: Comment documents nearby intent or constraints: `CHECK-NEXT: i64x2.replace_lane 1{{$}}`.
  **L205 CN**: 注释说明附近代码的意图或约束：`CHECK-NEXT: i64x2.replace_lane 1{{$}}`。
- **L206 EN**: Starts a function or method definition for `test_i64x2_make`.
  **L206 CN**: 开始定义函数或方法 `test_i64x2_make`。
- **L207 EN**: Returns from the current function with `wasm_i64x2_make(c0, c1)`.
  **L207 CN**: 以 `wasm_i64x2_make(c0, c1)` 从当前函数返回。
- **L208 EN**: Closes the current lexical scope or compound statement.
  **L208 CN**: 结束当前词法作用域或复合语句块。
- **L209 EN**: Blank line separating nearby declarations or logic.
  **L209 CN**: 空行，用于分隔相邻声明或逻辑。
- **L210 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f32x4_make:`.
  **L210 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f32x4_make:`。
- **L211 EN**: Comment documents nearby intent or constraints: `CHECK:      local.get 0{{$}}`.
  **L211 CN**: 注释说明附近代码的意图或约束：`CHECK:      local.get 0{{$}}`。
- **L212 EN**: Comment documents nearby intent or constraints: `CHECK-NEXT: f32x4.splat{{$}}`.
  **L212 CN**: 注释说明附近代码的意图或约束：`CHECK-NEXT: f32x4.splat{{$}}`。
- **L213 EN**: Comment documents nearby intent or constraints: `CHECK-NEXT: local.get 1{{$}}`.
  **L213 CN**: 注释说明附近代码的意图或约束：`CHECK-NEXT: local.get 1{{$}}`。
- **L214 EN**: Comment documents nearby intent or constraints: `CHECK-NEXT: f32x4.replace_lane 1{{$}}`.
  **L214 CN**: 注释说明附近代码的意图或约束：`CHECK-NEXT: f32x4.replace_lane 1{{$}}`。
- **L215 EN**: Comment documents nearby intent or constraints: `CHECK-NEXT: local.get 2{{$}}`.
  **L215 CN**: 注释说明附近代码的意图或约束：`CHECK-NEXT: local.get 2{{$}}`。
- **L216 EN**: Comment documents nearby intent or constraints: `CHECK-NEXT: f32x4.replace_lane 2{{$}}`.
  **L216 CN**: 注释说明附近代码的意图或约束：`CHECK-NEXT: f32x4.replace_lane 2{{$}}`。

### Lines 217-240

````c
// CHECK-NEXT: local.get 3{{$}}
// CHECK-NEXT: f32x4.replace_lane 3{{$}}
v128_t test_f32x4_make(float c0, float c1, float c2, float c3) {
  return wasm_f32x4_make(c0, c1, c2, c3);
}

// CHECK-LABEL: test_f64x2_make:
// CHECK:      local.get 0{{$}}
// CHECK-NEXT: f64x2.splat{{$}}
// CHECK-NEXT: local.get 1{{$}}
// CHECK-NEXT: f64x2.replace_lane 1{{$}}
v128_t test_f64x2_make(double c0, double c1) { return wasm_f64x2_make(c0, c1); }

// CHECK-LABEL: test_i8x16_const:
// CHECK: v128.const 50462976, 117835012, 185207048, 252579084{{$}}
v128_t test_i8x16_const() {
  return wasm_i8x16_const(0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15);
}

// CHECK-LABEL: test_i16x8_const:
// CHECK: v128.const 65536, 196610, 327684, 458758{{$}}
v128_t test_i16x8_const() { return wasm_i16x8_const(0, 1, 2, 3, 4, 5, 6, 7); }

// CHECK-LABEL: test_i32x4_const:
````
- **L217 EN**: Comment documents nearby intent or constraints: `CHECK-NEXT: local.get 3{{$}}`.
  **L217 CN**: 注释说明附近代码的意图或约束：`CHECK-NEXT: local.get 3{{$}}`。
- **L218 EN**: Comment documents nearby intent or constraints: `CHECK-NEXT: f32x4.replace_lane 3{{$}}`.
  **L218 CN**: 注释说明附近代码的意图或约束：`CHECK-NEXT: f32x4.replace_lane 3{{$}}`。
- **L219 EN**: Starts a function or method definition for `test_f32x4_make`.
  **L219 CN**: 开始定义函数或方法 `test_f32x4_make`。
- **L220 EN**: Returns from the current function with `wasm_f32x4_make(c0, c1, c2, c3)`.
  **L220 CN**: 以 `wasm_f32x4_make(c0, c1, c2, c3)` 从当前函数返回。
- **L221 EN**: Closes the current lexical scope or compound statement.
  **L221 CN**: 结束当前词法作用域或复合语句块。
- **L222 EN**: Blank line separating nearby declarations or logic.
  **L222 CN**: 空行，用于分隔相邻声明或逻辑。
- **L223 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f64x2_make:`.
  **L223 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f64x2_make:`。
- **L224 EN**: Comment documents nearby intent or constraints: `CHECK:      local.get 0{{$}}`.
  **L224 CN**: 注释说明附近代码的意图或约束：`CHECK:      local.get 0{{$}}`。
- **L225 EN**: Comment documents nearby intent or constraints: `CHECK-NEXT: f64x2.splat{{$}}`.
  **L225 CN**: 注释说明附近代码的意图或约束：`CHECK-NEXT: f64x2.splat{{$}}`。
- **L226 EN**: Comment documents nearby intent or constraints: `CHECK-NEXT: local.get 1{{$}}`.
  **L226 CN**: 注释说明附近代码的意图或约束：`CHECK-NEXT: local.get 1{{$}}`。
- **L227 EN**: Comment documents nearby intent or constraints: `CHECK-NEXT: f64x2.replace_lane 1{{$}}`.
  **L227 CN**: 注释说明附近代码的意图或约束：`CHECK-NEXT: f64x2.replace_lane 1{{$}}`。
- **L228 EN**: Starts a function or method definition for `test_f64x2_make`.
  **L228 CN**: 开始定义函数或方法 `test_f64x2_make`。
- **L229 EN**: Blank line separating nearby declarations or logic.
  **L229 CN**: 空行，用于分隔相邻声明或逻辑。
- **L230 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i8x16_const:`.
  **L230 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i8x16_const:`。
- **L231 EN**: Comment documents nearby intent or constraints: `CHECK: v128.const 50462976, 117835012, 185207048, 252579084{{$}}`.
  **L231 CN**: 注释说明附近代码的意图或约束：`CHECK: v128.const 50462976, 117835012, 185207048, 252579084{{$}}`。
- **L232 EN**: Starts a function or method definition for `test_i8x16_const`.
  **L232 CN**: 开始定义函数或方法 `test_i8x16_const`。
- **L233 EN**: Returns from the current function with `wasm_i8x16_const(0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15)`.
  **L233 CN**: 以 `wasm_i8x16_const(0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15)` 从当前函数返回。
- **L234 EN**: Closes the current lexical scope or compound statement.
  **L234 CN**: 结束当前词法作用域或复合语句块。
- **L235 EN**: Blank line separating nearby declarations or logic.
  **L235 CN**: 空行，用于分隔相邻声明或逻辑。
- **L236 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i16x8_const:`.
  **L236 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i16x8_const:`。
- **L237 EN**: Comment documents nearby intent or constraints: `CHECK: v128.const 65536, 196610, 327684, 458758{{$}}`.
  **L237 CN**: 注释说明附近代码的意图或约束：`CHECK: v128.const 65536, 196610, 327684, 458758{{$}}`。
- **L238 EN**: Starts a function or method definition for `test_i16x8_const`.
  **L238 CN**: 开始定义函数或方法 `test_i16x8_const`。
- **L239 EN**: Blank line separating nearby declarations or logic.
  **L239 CN**: 空行，用于分隔相邻声明或逻辑。
- **L240 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i32x4_const:`.
  **L240 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i32x4_const:`。

### Lines 241-264

````c
// CHECK: v128.const 0, 1, 2, 3{{$}}
v128_t test_i32x4_const() { return wasm_i32x4_const(0, 1, 2, 3); }

// CHECK-LABEL: test_i64x2_const:
// CHECK: v128.const 0, 0, 1, 0{{$}}
v128_t test_i64x2_const() { return wasm_i64x2_const(0, 1); }

// CHECK-LABEL: test_f32x4_const:
// CHECK: v128.const 0, 1065353216, 1073741824, 1077936128{{$}}
v128_t test_f32x4_const() { return wasm_f32x4_const(0, 1, 2, 3); }

// CHECK-LABEL: test_f64x2_const:
// CHECK: v128.const 0, 0, 0, 1072693248{{$}}
v128_t test_f64x2_const() { return wasm_f64x2_const(0, 1); }

// CHECK-LABEL: test_i8x16_splat:
// CHECK: i8x16.splat{{$}}
v128_t test_i8x16_splat(int8_t a) { return wasm_i8x16_splat(a); }

// CHECK-LABEL: test_i8x16_extract_lane:
// CHECK: i8x16.extract_lane_s 15{{$}}
int8_t test_i8x16_extract_lane(v128_t a) {
  return wasm_i8x16_extract_lane(a, 15);
}
````
- **L241 EN**: Comment documents nearby intent or constraints: `CHECK: v128.const 0, 1, 2, 3{{$}}`.
  **L241 CN**: 注释说明附近代码的意图或约束：`CHECK: v128.const 0, 1, 2, 3{{$}}`。
- **L242 EN**: Starts a function or method definition for `test_i32x4_const`.
  **L242 CN**: 开始定义函数或方法 `test_i32x4_const`。
- **L243 EN**: Blank line separating nearby declarations or logic.
  **L243 CN**: 空行，用于分隔相邻声明或逻辑。
- **L244 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i64x2_const:`.
  **L244 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i64x2_const:`。
- **L245 EN**: Comment documents nearby intent or constraints: `CHECK: v128.const 0, 0, 1, 0{{$}}`.
  **L245 CN**: 注释说明附近代码的意图或约束：`CHECK: v128.const 0, 0, 1, 0{{$}}`。
- **L246 EN**: Starts a function or method definition for `test_i64x2_const`.
  **L246 CN**: 开始定义函数或方法 `test_i64x2_const`。
- **L247 EN**: Blank line separating nearby declarations or logic.
  **L247 CN**: 空行，用于分隔相邻声明或逻辑。
- **L248 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f32x4_const:`.
  **L248 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f32x4_const:`。
- **L249 EN**: Comment documents nearby intent or constraints: `CHECK: v128.const 0, 1065353216, 1073741824, 1077936128{{$}}`.
  **L249 CN**: 注释说明附近代码的意图或约束：`CHECK: v128.const 0, 1065353216, 1073741824, 1077936128{{$}}`。
- **L250 EN**: Starts a function or method definition for `test_f32x4_const`.
  **L250 CN**: 开始定义函数或方法 `test_f32x4_const`。
- **L251 EN**: Blank line separating nearby declarations or logic.
  **L251 CN**: 空行，用于分隔相邻声明或逻辑。
- **L252 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f64x2_const:`.
  **L252 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f64x2_const:`。
- **L253 EN**: Comment documents nearby intent or constraints: `CHECK: v128.const 0, 0, 0, 1072693248{{$}}`.
  **L253 CN**: 注释说明附近代码的意图或约束：`CHECK: v128.const 0, 0, 0, 1072693248{{$}}`。
- **L254 EN**: Starts a function or method definition for `test_f64x2_const`.
  **L254 CN**: 开始定义函数或方法 `test_f64x2_const`。
- **L255 EN**: Blank line separating nearby declarations or logic.
  **L255 CN**: 空行，用于分隔相邻声明或逻辑。
- **L256 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i8x16_splat:`.
  **L256 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i8x16_splat:`。
- **L257 EN**: Comment documents nearby intent or constraints: `CHECK: i8x16.splat{{$}}`.
  **L257 CN**: 注释说明附近代码的意图或约束：`CHECK: i8x16.splat{{$}}`。
- **L258 EN**: Starts a function or method definition for `test_i8x16_splat`.
  **L258 CN**: 开始定义函数或方法 `test_i8x16_splat`。
- **L259 EN**: Blank line separating nearby declarations or logic.
  **L259 CN**: 空行，用于分隔相邻声明或逻辑。
- **L260 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i8x16_extract_lane:`.
  **L260 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i8x16_extract_lane:`。
- **L261 EN**: Comment documents nearby intent or constraints: `CHECK: i8x16.extract_lane_s 15{{$}}`.
  **L261 CN**: 注释说明附近代码的意图或约束：`CHECK: i8x16.extract_lane_s 15{{$}}`。
- **L262 EN**: Starts a function or method definition for `test_i8x16_extract_lane`.
  **L262 CN**: 开始定义函数或方法 `test_i8x16_extract_lane`。
- **L263 EN**: Returns from the current function with `wasm_i8x16_extract_lane(a, 15)`.
  **L263 CN**: 以 `wasm_i8x16_extract_lane(a, 15)` 从当前函数返回。
- **L264 EN**: Closes the current lexical scope or compound statement.
  **L264 CN**: 结束当前词法作用域或复合语句块。

### Lines 265-288

````c

// CHECK-LABEL: test_u8x16_extract_lane:
// CHECK: i8x16.extract_lane_u 15{{$}}
uint8_t test_u8x16_extract_lane(v128_t a) {
  return wasm_u8x16_extract_lane(a, 15);
}

// CHECK-LABEL: test_i8x16_replace_lane:
// CHECK: i8x16.replace_lane 15{{$}}
v128_t test_i8x16_replace_lane(v128_t a, int8_t b) {
  return wasm_i8x16_replace_lane(a, 15, b);
}

// CHECK-LABEL: test_i16x8_splat:
// CHECK: i16x8.splat{{$}}
v128_t test_i16x8_splat(int16_t a) { return wasm_i16x8_splat(a); }

// CHECK-LABEL: test_i16x8_extract_lane:
// CHECK: i16x8.extract_lane_s 7{{$}}
int16_t test_i16x8_extract_lane(v128_t a) {
  return wasm_i16x8_extract_lane(a, 7);
}

// CHECK-LABEL: test_u16x8_extract_lane:
````
- **L265 EN**: Blank line separating nearby declarations or logic.
  **L265 CN**: 空行，用于分隔相邻声明或逻辑。
- **L266 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_u8x16_extract_lane:`.
  **L266 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_u8x16_extract_lane:`。
- **L267 EN**: Comment documents nearby intent or constraints: `CHECK: i8x16.extract_lane_u 15{{$}}`.
  **L267 CN**: 注释说明附近代码的意图或约束：`CHECK: i8x16.extract_lane_u 15{{$}}`。
- **L268 EN**: Starts a function or method definition for `test_u8x16_extract_lane`.
  **L268 CN**: 开始定义函数或方法 `test_u8x16_extract_lane`。
- **L269 EN**: Returns from the current function with `wasm_u8x16_extract_lane(a, 15)`.
  **L269 CN**: 以 `wasm_u8x16_extract_lane(a, 15)` 从当前函数返回。
- **L270 EN**: Closes the current lexical scope or compound statement.
  **L270 CN**: 结束当前词法作用域或复合语句块。
- **L271 EN**: Blank line separating nearby declarations or logic.
  **L271 CN**: 空行，用于分隔相邻声明或逻辑。
- **L272 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i8x16_replace_lane:`.
  **L272 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i8x16_replace_lane:`。
- **L273 EN**: Comment documents nearby intent or constraints: `CHECK: i8x16.replace_lane 15{{$}}`.
  **L273 CN**: 注释说明附近代码的意图或约束：`CHECK: i8x16.replace_lane 15{{$}}`。
- **L274 EN**: Starts a function or method definition for `test_i8x16_replace_lane`.
  **L274 CN**: 开始定义函数或方法 `test_i8x16_replace_lane`。
- **L275 EN**: Returns from the current function with `wasm_i8x16_replace_lane(a, 15, b)`.
  **L275 CN**: 以 `wasm_i8x16_replace_lane(a, 15, b)` 从当前函数返回。
- **L276 EN**: Closes the current lexical scope or compound statement.
  **L276 CN**: 结束当前词法作用域或复合语句块。
- **L277 EN**: Blank line separating nearby declarations or logic.
  **L277 CN**: 空行，用于分隔相邻声明或逻辑。
- **L278 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i16x8_splat:`.
  **L278 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i16x8_splat:`。
- **L279 EN**: Comment documents nearby intent or constraints: `CHECK: i16x8.splat{{$}}`.
  **L279 CN**: 注释说明附近代码的意图或约束：`CHECK: i16x8.splat{{$}}`。
- **L280 EN**: Starts a function or method definition for `test_i16x8_splat`.
  **L280 CN**: 开始定义函数或方法 `test_i16x8_splat`。
- **L281 EN**: Blank line separating nearby declarations or logic.
  **L281 CN**: 空行，用于分隔相邻声明或逻辑。
- **L282 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i16x8_extract_lane:`.
  **L282 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i16x8_extract_lane:`。
- **L283 EN**: Comment documents nearby intent or constraints: `CHECK: i16x8.extract_lane_s 7{{$}}`.
  **L283 CN**: 注释说明附近代码的意图或约束：`CHECK: i16x8.extract_lane_s 7{{$}}`。
- **L284 EN**: Starts a function or method definition for `test_i16x8_extract_lane`.
  **L284 CN**: 开始定义函数或方法 `test_i16x8_extract_lane`。
- **L285 EN**: Returns from the current function with `wasm_i16x8_extract_lane(a, 7)`.
  **L285 CN**: 以 `wasm_i16x8_extract_lane(a, 7)` 从当前函数返回。
- **L286 EN**: Closes the current lexical scope or compound statement.
  **L286 CN**: 结束当前词法作用域或复合语句块。
- **L287 EN**: Blank line separating nearby declarations or logic.
  **L287 CN**: 空行，用于分隔相邻声明或逻辑。
- **L288 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_u16x8_extract_lane:`.
  **L288 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_u16x8_extract_lane:`。

### Lines 289-312

````c
// CHECK: i16x8.extract_lane_u 7{{$}}
uint16_t test_u16x8_extract_lane(v128_t a) {
  return wasm_u16x8_extract_lane(a, 7);
}

// CHECK-LABEL: test_i16x8_replace_lane:
// CHECK: i16x8.replace_lane 7{{$}}
v128_t test_i16x8_replace_lane(v128_t a, int16_t b) {
  return wasm_i16x8_replace_lane(a, 7, b);
}

// CHECK-LABEL: test_i32x4_splat:
// CHECK: i32x4.splat{{$}}
v128_t test_i32x4_splat(int32_t a) { return wasm_i32x4_splat(a); }

// CHECK-LABEL: test_i32x4_extract_lane:
// CHECK: i32x4.extract_lane 3{{$}}
int32_t test_i32x4_extract_lane(v128_t a) {
  return wasm_i32x4_extract_lane(a, 3);
}

// CHECK-LABEL: test_i32x4_replace_lane:
// CHECK: i32x4.replace_lane 3{{$}}
v128_t test_i32x4_replace_lane(v128_t a, int32_t b) {
````
- **L289 EN**: Comment documents nearby intent or constraints: `CHECK: i16x8.extract_lane_u 7{{$}}`.
  **L289 CN**: 注释说明附近代码的意图或约束：`CHECK: i16x8.extract_lane_u 7{{$}}`。
- **L290 EN**: Starts a function or method definition for `test_u16x8_extract_lane`.
  **L290 CN**: 开始定义函数或方法 `test_u16x8_extract_lane`。
- **L291 EN**: Returns from the current function with `wasm_u16x8_extract_lane(a, 7)`.
  **L291 CN**: 以 `wasm_u16x8_extract_lane(a, 7)` 从当前函数返回。
- **L292 EN**: Closes the current lexical scope or compound statement.
  **L292 CN**: 结束当前词法作用域或复合语句块。
- **L293 EN**: Blank line separating nearby declarations or logic.
  **L293 CN**: 空行，用于分隔相邻声明或逻辑。
- **L294 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i16x8_replace_lane:`.
  **L294 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i16x8_replace_lane:`。
- **L295 EN**: Comment documents nearby intent or constraints: `CHECK: i16x8.replace_lane 7{{$}}`.
  **L295 CN**: 注释说明附近代码的意图或约束：`CHECK: i16x8.replace_lane 7{{$}}`。
- **L296 EN**: Starts a function or method definition for `test_i16x8_replace_lane`.
  **L296 CN**: 开始定义函数或方法 `test_i16x8_replace_lane`。
- **L297 EN**: Returns from the current function with `wasm_i16x8_replace_lane(a, 7, b)`.
  **L297 CN**: 以 `wasm_i16x8_replace_lane(a, 7, b)` 从当前函数返回。
- **L298 EN**: Closes the current lexical scope or compound statement.
  **L298 CN**: 结束当前词法作用域或复合语句块。
- **L299 EN**: Blank line separating nearby declarations or logic.
  **L299 CN**: 空行，用于分隔相邻声明或逻辑。
- **L300 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i32x4_splat:`.
  **L300 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i32x4_splat:`。
- **L301 EN**: Comment documents nearby intent or constraints: `CHECK: i32x4.splat{{$}}`.
  **L301 CN**: 注释说明附近代码的意图或约束：`CHECK: i32x4.splat{{$}}`。
- **L302 EN**: Starts a function or method definition for `test_i32x4_splat`.
  **L302 CN**: 开始定义函数或方法 `test_i32x4_splat`。
- **L303 EN**: Blank line separating nearby declarations or logic.
  **L303 CN**: 空行，用于分隔相邻声明或逻辑。
- **L304 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i32x4_extract_lane:`.
  **L304 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i32x4_extract_lane:`。
- **L305 EN**: Comment documents nearby intent or constraints: `CHECK: i32x4.extract_lane 3{{$}}`.
  **L305 CN**: 注释说明附近代码的意图或约束：`CHECK: i32x4.extract_lane 3{{$}}`。
- **L306 EN**: Starts a function or method definition for `test_i32x4_extract_lane`.
  **L306 CN**: 开始定义函数或方法 `test_i32x4_extract_lane`。
- **L307 EN**: Returns from the current function with `wasm_i32x4_extract_lane(a, 3)`.
  **L307 CN**: 以 `wasm_i32x4_extract_lane(a, 3)` 从当前函数返回。
- **L308 EN**: Closes the current lexical scope or compound statement.
  **L308 CN**: 结束当前词法作用域或复合语句块。
- **L309 EN**: Blank line separating nearby declarations or logic.
  **L309 CN**: 空行，用于分隔相邻声明或逻辑。
- **L310 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i32x4_replace_lane:`.
  **L310 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i32x4_replace_lane:`。
- **L311 EN**: Comment documents nearby intent or constraints: `CHECK: i32x4.replace_lane 3{{$}}`.
  **L311 CN**: 注释说明附近代码的意图或约束：`CHECK: i32x4.replace_lane 3{{$}}`。
- **L312 EN**: Starts a function or method definition for `test_i32x4_replace_lane`.
  **L312 CN**: 开始定义函数或方法 `test_i32x4_replace_lane`。

### Lines 313-336

````c
  return wasm_i32x4_replace_lane(a, 3, b);
}

// CHECK-LABEL: test_i64x2_splat:
// CHECK: i64x2.splat{{$}}
v128_t test_i64x2_splat(int64_t a) { return wasm_i64x2_splat(a); }

// CHECK-LABEL: test_i64x2_extract_lane:
// CHECK: i64x2.extract_lane 1{{$}}
int64_t test_i64x2_extract_lane(v128_t a) {
  return wasm_i64x2_extract_lane(a, 1);
}

// CHECK-LABEL: test_i64x2_replace_lane:
// CHECK: i64x2.replace_lane 1{{$}}
v128_t test_i64x2_replace_lane(v128_t a, int64_t b) {
  return wasm_i64x2_replace_lane(a, 1, b);
}

// CHECK-LABEL: test_f32x4_splat:
// CHECK: f32x4.splat{{$}}
v128_t test_f32x4_splat(float a) { return wasm_f32x4_splat(a); }

// CHECK-LABEL: test_f32x4_extract_lane:
````
- **L313 EN**: Returns from the current function with `wasm_i32x4_replace_lane(a, 3, b)`.
  **L313 CN**: 以 `wasm_i32x4_replace_lane(a, 3, b)` 从当前函数返回。
- **L314 EN**: Closes the current lexical scope or compound statement.
  **L314 CN**: 结束当前词法作用域或复合语句块。
- **L315 EN**: Blank line separating nearby declarations or logic.
  **L315 CN**: 空行，用于分隔相邻声明或逻辑。
- **L316 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i64x2_splat:`.
  **L316 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i64x2_splat:`。
- **L317 EN**: Comment documents nearby intent or constraints: `CHECK: i64x2.splat{{$}}`.
  **L317 CN**: 注释说明附近代码的意图或约束：`CHECK: i64x2.splat{{$}}`。
- **L318 EN**: Starts a function or method definition for `test_i64x2_splat`.
  **L318 CN**: 开始定义函数或方法 `test_i64x2_splat`。
- **L319 EN**: Blank line separating nearby declarations or logic.
  **L319 CN**: 空行，用于分隔相邻声明或逻辑。
- **L320 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i64x2_extract_lane:`.
  **L320 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i64x2_extract_lane:`。
- **L321 EN**: Comment documents nearby intent or constraints: `CHECK: i64x2.extract_lane 1{{$}}`.
  **L321 CN**: 注释说明附近代码的意图或约束：`CHECK: i64x2.extract_lane 1{{$}}`。
- **L322 EN**: Starts a function or method definition for `test_i64x2_extract_lane`.
  **L322 CN**: 开始定义函数或方法 `test_i64x2_extract_lane`。
- **L323 EN**: Returns from the current function with `wasm_i64x2_extract_lane(a, 1)`.
  **L323 CN**: 以 `wasm_i64x2_extract_lane(a, 1)` 从当前函数返回。
- **L324 EN**: Closes the current lexical scope or compound statement.
  **L324 CN**: 结束当前词法作用域或复合语句块。
- **L325 EN**: Blank line separating nearby declarations or logic.
  **L325 CN**: 空行，用于分隔相邻声明或逻辑。
- **L326 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i64x2_replace_lane:`.
  **L326 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i64x2_replace_lane:`。
- **L327 EN**: Comment documents nearby intent or constraints: `CHECK: i64x2.replace_lane 1{{$}}`.
  **L327 CN**: 注释说明附近代码的意图或约束：`CHECK: i64x2.replace_lane 1{{$}}`。
- **L328 EN**: Starts a function or method definition for `test_i64x2_replace_lane`.
  **L328 CN**: 开始定义函数或方法 `test_i64x2_replace_lane`。
- **L329 EN**: Returns from the current function with `wasm_i64x2_replace_lane(a, 1, b)`.
  **L329 CN**: 以 `wasm_i64x2_replace_lane(a, 1, b)` 从当前函数返回。
- **L330 EN**: Closes the current lexical scope or compound statement.
  **L330 CN**: 结束当前词法作用域或复合语句块。
- **L331 EN**: Blank line separating nearby declarations or logic.
  **L331 CN**: 空行，用于分隔相邻声明或逻辑。
- **L332 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f32x4_splat:`.
  **L332 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f32x4_splat:`。
- **L333 EN**: Comment documents nearby intent or constraints: `CHECK: f32x4.splat{{$}}`.
  **L333 CN**: 注释说明附近代码的意图或约束：`CHECK: f32x4.splat{{$}}`。
- **L334 EN**: Starts a function or method definition for `test_f32x4_splat`.
  **L334 CN**: 开始定义函数或方法 `test_f32x4_splat`。
- **L335 EN**: Blank line separating nearby declarations or logic.
  **L335 CN**: 空行，用于分隔相邻声明或逻辑。
- **L336 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f32x4_extract_lane:`.
  **L336 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f32x4_extract_lane:`。

### Lines 337-360

````c
// CHECK: f32x4.extract_lane 3{{$}}
float test_f32x4_extract_lane(v128_t a) {
  return wasm_f32x4_extract_lane(a, 3);
}

// CHECK-LABEL: test_f32x4_replace_lane:
// CHECK: f32x4.replace_lane 3{{$}}
v128_t test_f32x4_replace_lane(v128_t a, float b) {
  return wasm_f32x4_replace_lane(a, 3, b);
}

// CHECK-LABEL: test_f64x2_splat:
// CHECK: f64x2.splat{{$}}
v128_t test_f64x2_splat(double a) { return wasm_f64x2_splat(a); }

// CHECK-LABEL: test_f64x2_extract_lane:
// CHECK: f64x2.extract_lane 1{{$}}
double test_f64x2_extract_lane(v128_t a) {
  return wasm_f64x2_extract_lane(a, 1);
}

// CHECK-LABEL: test_f64x2_replace_lane:
// CHECK: f64x2.replace_lane 1{{$}}
v128_t test_f64x2_replace_lane(v128_t a, double b) {
````
- **L337 EN**: Comment documents nearby intent or constraints: `CHECK: f32x4.extract_lane 3{{$}}`.
  **L337 CN**: 注释说明附近代码的意图或约束：`CHECK: f32x4.extract_lane 3{{$}}`。
- **L338 EN**: Starts a function or method definition for `test_f32x4_extract_lane`.
  **L338 CN**: 开始定义函数或方法 `test_f32x4_extract_lane`。
- **L339 EN**: Returns from the current function with `wasm_f32x4_extract_lane(a, 3)`.
  **L339 CN**: 以 `wasm_f32x4_extract_lane(a, 3)` 从当前函数返回。
- **L340 EN**: Closes the current lexical scope or compound statement.
  **L340 CN**: 结束当前词法作用域或复合语句块。
- **L341 EN**: Blank line separating nearby declarations or logic.
  **L341 CN**: 空行，用于分隔相邻声明或逻辑。
- **L342 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f32x4_replace_lane:`.
  **L342 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f32x4_replace_lane:`。
- **L343 EN**: Comment documents nearby intent or constraints: `CHECK: f32x4.replace_lane 3{{$}}`.
  **L343 CN**: 注释说明附近代码的意图或约束：`CHECK: f32x4.replace_lane 3{{$}}`。
- **L344 EN**: Starts a function or method definition for `test_f32x4_replace_lane`.
  **L344 CN**: 开始定义函数或方法 `test_f32x4_replace_lane`。
- **L345 EN**: Returns from the current function with `wasm_f32x4_replace_lane(a, 3, b)`.
  **L345 CN**: 以 `wasm_f32x4_replace_lane(a, 3, b)` 从当前函数返回。
- **L346 EN**: Closes the current lexical scope or compound statement.
  **L346 CN**: 结束当前词法作用域或复合语句块。
- **L347 EN**: Blank line separating nearby declarations or logic.
  **L347 CN**: 空行，用于分隔相邻声明或逻辑。
- **L348 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f64x2_splat:`.
  **L348 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f64x2_splat:`。
- **L349 EN**: Comment documents nearby intent or constraints: `CHECK: f64x2.splat{{$}}`.
  **L349 CN**: 注释说明附近代码的意图或约束：`CHECK: f64x2.splat{{$}}`。
- **L350 EN**: Starts a function or method definition for `test_f64x2_splat`.
  **L350 CN**: 开始定义函数或方法 `test_f64x2_splat`。
- **L351 EN**: Blank line separating nearby declarations or logic.
  **L351 CN**: 空行，用于分隔相邻声明或逻辑。
- **L352 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f64x2_extract_lane:`.
  **L352 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f64x2_extract_lane:`。
- **L353 EN**: Comment documents nearby intent or constraints: `CHECK: f64x2.extract_lane 1{{$}}`.
  **L353 CN**: 注释说明附近代码的意图或约束：`CHECK: f64x2.extract_lane 1{{$}}`。
- **L354 EN**: Starts a function or method definition for `test_f64x2_extract_lane`.
  **L354 CN**: 开始定义函数或方法 `test_f64x2_extract_lane`。
- **L355 EN**: Returns from the current function with `wasm_f64x2_extract_lane(a, 1)`.
  **L355 CN**: 以 `wasm_f64x2_extract_lane(a, 1)` 从当前函数返回。
- **L356 EN**: Closes the current lexical scope or compound statement.
  **L356 CN**: 结束当前词法作用域或复合语句块。
- **L357 EN**: Blank line separating nearby declarations or logic.
  **L357 CN**: 空行，用于分隔相邻声明或逻辑。
- **L358 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f64x2_replace_lane:`.
  **L358 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f64x2_replace_lane:`。
- **L359 EN**: Comment documents nearby intent or constraints: `CHECK: f64x2.replace_lane 1{{$}}`.
  **L359 CN**: 注释说明附近代码的意图或约束：`CHECK: f64x2.replace_lane 1{{$}}`。
- **L360 EN**: Starts a function or method definition for `test_f64x2_replace_lane`.
  **L360 CN**: 开始定义函数或方法 `test_f64x2_replace_lane`。

### Lines 361-384

````c
  return wasm_f64x2_replace_lane(a, 1, b);
}

// CHECK-LABEL: test_i8x16_eq:
// CHECK: i8x16.eq{{$}}
v128_t test_i8x16_eq(v128_t a, v128_t b) { return wasm_i8x16_eq(a, b); }

// CHECK-LABEL: test_i8x16_ne:
// CHECK: i8x16.ne{{$}}
v128_t test_i8x16_ne(v128_t a, v128_t b) { return wasm_i8x16_ne(a, b); }

// CHECK-LABEL: test_i8x16_lt:
// CHECK: i8x16.lt_s{{$}}
v128_t test_i8x16_lt(v128_t a, v128_t b) { return wasm_i8x16_lt(a, b); }

// CHECK-LABEL: test_u8x16_lt:
// CHECK: i8x16.lt_u{{$}}
v128_t test_u8x16_lt(v128_t a, v128_t b) { return wasm_u8x16_lt(a, b); }

// CHECK-LABEL: test_i8x16_gt:
// CHECK: i8x16.gt_s{{$}}
v128_t test_i8x16_gt(v128_t a, v128_t b) { return wasm_i8x16_gt(a, b); }

// CHECK-LABEL: test_u8x16_gt:
````
- **L361 EN**: Returns from the current function with `wasm_f64x2_replace_lane(a, 1, b)`.
  **L361 CN**: 以 `wasm_f64x2_replace_lane(a, 1, b)` 从当前函数返回。
- **L362 EN**: Closes the current lexical scope or compound statement.
  **L362 CN**: 结束当前词法作用域或复合语句块。
- **L363 EN**: Blank line separating nearby declarations or logic.
  **L363 CN**: 空行，用于分隔相邻声明或逻辑。
- **L364 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i8x16_eq:`.
  **L364 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i8x16_eq:`。
- **L365 EN**: Comment documents nearby intent or constraints: `CHECK: i8x16.eq{{$}}`.
  **L365 CN**: 注释说明附近代码的意图或约束：`CHECK: i8x16.eq{{$}}`。
- **L366 EN**: Starts a function or method definition for `test_i8x16_eq`.
  **L366 CN**: 开始定义函数或方法 `test_i8x16_eq`。
- **L367 EN**: Blank line separating nearby declarations or logic.
  **L367 CN**: 空行，用于分隔相邻声明或逻辑。
- **L368 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i8x16_ne:`.
  **L368 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i8x16_ne:`。
- **L369 EN**: Comment documents nearby intent or constraints: `CHECK: i8x16.ne{{$}}`.
  **L369 CN**: 注释说明附近代码的意图或约束：`CHECK: i8x16.ne{{$}}`。
- **L370 EN**: Starts a function or method definition for `test_i8x16_ne`.
  **L370 CN**: 开始定义函数或方法 `test_i8x16_ne`。
- **L371 EN**: Blank line separating nearby declarations or logic.
  **L371 CN**: 空行，用于分隔相邻声明或逻辑。
- **L372 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i8x16_lt:`.
  **L372 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i8x16_lt:`。
- **L373 EN**: Comment documents nearby intent or constraints: `CHECK: i8x16.lt_s{{$}}`.
  **L373 CN**: 注释说明附近代码的意图或约束：`CHECK: i8x16.lt_s{{$}}`。
- **L374 EN**: Starts a function or method definition for `test_i8x16_lt`.
  **L374 CN**: 开始定义函数或方法 `test_i8x16_lt`。
- **L375 EN**: Blank line separating nearby declarations or logic.
  **L375 CN**: 空行，用于分隔相邻声明或逻辑。
- **L376 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_u8x16_lt:`.
  **L376 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_u8x16_lt:`。
- **L377 EN**: Comment documents nearby intent or constraints: `CHECK: i8x16.lt_u{{$}}`.
  **L377 CN**: 注释说明附近代码的意图或约束：`CHECK: i8x16.lt_u{{$}}`。
- **L378 EN**: Starts a function or method definition for `test_u8x16_lt`.
  **L378 CN**: 开始定义函数或方法 `test_u8x16_lt`。
- **L379 EN**: Blank line separating nearby declarations or logic.
  **L379 CN**: 空行，用于分隔相邻声明或逻辑。
- **L380 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i8x16_gt:`.
  **L380 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i8x16_gt:`。
- **L381 EN**: Comment documents nearby intent or constraints: `CHECK: i8x16.gt_s{{$}}`.
  **L381 CN**: 注释说明附近代码的意图或约束：`CHECK: i8x16.gt_s{{$}}`。
- **L382 EN**: Starts a function or method definition for `test_i8x16_gt`.
  **L382 CN**: 开始定义函数或方法 `test_i8x16_gt`。
- **L383 EN**: Blank line separating nearby declarations or logic.
  **L383 CN**: 空行，用于分隔相邻声明或逻辑。
- **L384 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_u8x16_gt:`.
  **L384 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_u8x16_gt:`。

### Lines 385-408

````c
// CHECK: i8x16.gt_u{{$}}
v128_t test_u8x16_gt(v128_t a, v128_t b) { return wasm_u8x16_gt(a, b); }

// CHECK-LABEL: test_i8x16_le:
// CHECK: i8x16.le_s{{$}}
v128_t test_i8x16_le(v128_t a, v128_t b) { return wasm_i8x16_le(a, b); }

// CHECK-LABEL: test_u8x16_le:
// CHECK: i8x16.le_u{{$}}
v128_t test_u8x16_le(v128_t a, v128_t b) { return wasm_u8x16_le(a, b); }

// CHECK-LABEL: test_i8x16_ge:
// CHECK: i8x16.ge_s{{$}}
v128_t test_i8x16_ge(v128_t a, v128_t b) { return wasm_i8x16_ge(a, b); }

// CHECK-LABEL: test_u8x16_ge:
// CHECK: i8x16.ge_u{{$}}
v128_t test_u8x16_ge(v128_t a, v128_t b) { return wasm_u8x16_ge(a, b); }

// CHECK-LABEL: test_i16x8_eq:
// CHECK: i16x8.eq{{$}}
v128_t test_i16x8_eq(v128_t a, v128_t b) { return wasm_i16x8_eq(a, b); }

// CHECK-LABEL: test_i16x8_ne:
````
- **L385 EN**: Comment documents nearby intent or constraints: `CHECK: i8x16.gt_u{{$}}`.
  **L385 CN**: 注释说明附近代码的意图或约束：`CHECK: i8x16.gt_u{{$}}`。
- **L386 EN**: Starts a function or method definition for `test_u8x16_gt`.
  **L386 CN**: 开始定义函数或方法 `test_u8x16_gt`。
- **L387 EN**: Blank line separating nearby declarations or logic.
  **L387 CN**: 空行，用于分隔相邻声明或逻辑。
- **L388 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i8x16_le:`.
  **L388 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i8x16_le:`。
- **L389 EN**: Comment documents nearby intent or constraints: `CHECK: i8x16.le_s{{$}}`.
  **L389 CN**: 注释说明附近代码的意图或约束：`CHECK: i8x16.le_s{{$}}`。
- **L390 EN**: Starts a function or method definition for `test_i8x16_le`.
  **L390 CN**: 开始定义函数或方法 `test_i8x16_le`。
- **L391 EN**: Blank line separating nearby declarations or logic.
  **L391 CN**: 空行，用于分隔相邻声明或逻辑。
- **L392 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_u8x16_le:`.
  **L392 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_u8x16_le:`。
- **L393 EN**: Comment documents nearby intent or constraints: `CHECK: i8x16.le_u{{$}}`.
  **L393 CN**: 注释说明附近代码的意图或约束：`CHECK: i8x16.le_u{{$}}`。
- **L394 EN**: Starts a function or method definition for `test_u8x16_le`.
  **L394 CN**: 开始定义函数或方法 `test_u8x16_le`。
- **L395 EN**: Blank line separating nearby declarations or logic.
  **L395 CN**: 空行，用于分隔相邻声明或逻辑。
- **L396 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i8x16_ge:`.
  **L396 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i8x16_ge:`。
- **L397 EN**: Comment documents nearby intent or constraints: `CHECK: i8x16.ge_s{{$}}`.
  **L397 CN**: 注释说明附近代码的意图或约束：`CHECK: i8x16.ge_s{{$}}`。
- **L398 EN**: Starts a function or method definition for `test_i8x16_ge`.
  **L398 CN**: 开始定义函数或方法 `test_i8x16_ge`。
- **L399 EN**: Blank line separating nearby declarations or logic.
  **L399 CN**: 空行，用于分隔相邻声明或逻辑。
- **L400 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_u8x16_ge:`.
  **L400 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_u8x16_ge:`。
- **L401 EN**: Comment documents nearby intent or constraints: `CHECK: i8x16.ge_u{{$}}`.
  **L401 CN**: 注释说明附近代码的意图或约束：`CHECK: i8x16.ge_u{{$}}`。
- **L402 EN**: Starts a function or method definition for `test_u8x16_ge`.
  **L402 CN**: 开始定义函数或方法 `test_u8x16_ge`。
- **L403 EN**: Blank line separating nearby declarations or logic.
  **L403 CN**: 空行，用于分隔相邻声明或逻辑。
- **L404 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i16x8_eq:`.
  **L404 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i16x8_eq:`。
- **L405 EN**: Comment documents nearby intent or constraints: `CHECK: i16x8.eq{{$}}`.
  **L405 CN**: 注释说明附近代码的意图或约束：`CHECK: i16x8.eq{{$}}`。
- **L406 EN**: Starts a function or method definition for `test_i16x8_eq`.
  **L406 CN**: 开始定义函数或方法 `test_i16x8_eq`。
- **L407 EN**: Blank line separating nearby declarations or logic.
  **L407 CN**: 空行，用于分隔相邻声明或逻辑。
- **L408 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i16x8_ne:`.
  **L408 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i16x8_ne:`。

### Lines 409-432

````c
// CHECK: i16x8.ne{{$}}
v128_t test_i16x8_ne(v128_t a, v128_t b) { return wasm_i16x8_ne(a, b); }

// CHECK-LABEL: test_i16x8_lt:
// CHECK: i16x8.lt_s{{$}}
v128_t test_i16x8_lt(v128_t a, v128_t b) { return wasm_i16x8_lt(a, b); }

// CHECK-LABEL: test_u16x8_lt:
// CHECK: i16x8.lt_u{{$}}
v128_t test_u16x8_lt(v128_t a, v128_t b) { return wasm_u16x8_lt(a, b); }

// CHECK-LABEL: test_i16x8_gt:
// CHECK: i16x8.gt_s{{$}}
v128_t test_i16x8_gt(v128_t a, v128_t b) { return wasm_i16x8_gt(a, b); }

// CHECK-LABEL: test_u16x8_gt:
// CHECK: i16x8.gt_u{{$}}
v128_t test_u16x8_gt(v128_t a, v128_t b) { return wasm_u16x8_gt(a, b); }

// CHECK-LABEL: test_i16x8_le:
// CHECK: i16x8.le_s{{$}}
v128_t test_i16x8_le(v128_t a, v128_t b) { return wasm_i16x8_le(a, b); }

// CHECK-LABEL: test_u16x8_le:
````
- **L409 EN**: Comment documents nearby intent or constraints: `CHECK: i16x8.ne{{$}}`.
  **L409 CN**: 注释说明附近代码的意图或约束：`CHECK: i16x8.ne{{$}}`。
- **L410 EN**: Starts a function or method definition for `test_i16x8_ne`.
  **L410 CN**: 开始定义函数或方法 `test_i16x8_ne`。
- **L411 EN**: Blank line separating nearby declarations or logic.
  **L411 CN**: 空行，用于分隔相邻声明或逻辑。
- **L412 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i16x8_lt:`.
  **L412 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i16x8_lt:`。
- **L413 EN**: Comment documents nearby intent or constraints: `CHECK: i16x8.lt_s{{$}}`.
  **L413 CN**: 注释说明附近代码的意图或约束：`CHECK: i16x8.lt_s{{$}}`。
- **L414 EN**: Starts a function or method definition for `test_i16x8_lt`.
  **L414 CN**: 开始定义函数或方法 `test_i16x8_lt`。
- **L415 EN**: Blank line separating nearby declarations or logic.
  **L415 CN**: 空行，用于分隔相邻声明或逻辑。
- **L416 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_u16x8_lt:`.
  **L416 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_u16x8_lt:`。
- **L417 EN**: Comment documents nearby intent or constraints: `CHECK: i16x8.lt_u{{$}}`.
  **L417 CN**: 注释说明附近代码的意图或约束：`CHECK: i16x8.lt_u{{$}}`。
- **L418 EN**: Starts a function or method definition for `test_u16x8_lt`.
  **L418 CN**: 开始定义函数或方法 `test_u16x8_lt`。
- **L419 EN**: Blank line separating nearby declarations or logic.
  **L419 CN**: 空行，用于分隔相邻声明或逻辑。
- **L420 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i16x8_gt:`.
  **L420 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i16x8_gt:`。
- **L421 EN**: Comment documents nearby intent or constraints: `CHECK: i16x8.gt_s{{$}}`.
  **L421 CN**: 注释说明附近代码的意图或约束：`CHECK: i16x8.gt_s{{$}}`。
- **L422 EN**: Starts a function or method definition for `test_i16x8_gt`.
  **L422 CN**: 开始定义函数或方法 `test_i16x8_gt`。
- **L423 EN**: Blank line separating nearby declarations or logic.
  **L423 CN**: 空行，用于分隔相邻声明或逻辑。
- **L424 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_u16x8_gt:`.
  **L424 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_u16x8_gt:`。
- **L425 EN**: Comment documents nearby intent or constraints: `CHECK: i16x8.gt_u{{$}}`.
  **L425 CN**: 注释说明附近代码的意图或约束：`CHECK: i16x8.gt_u{{$}}`。
- **L426 EN**: Starts a function or method definition for `test_u16x8_gt`.
  **L426 CN**: 开始定义函数或方法 `test_u16x8_gt`。
- **L427 EN**: Blank line separating nearby declarations or logic.
  **L427 CN**: 空行，用于分隔相邻声明或逻辑。
- **L428 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i16x8_le:`.
  **L428 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i16x8_le:`。
- **L429 EN**: Comment documents nearby intent or constraints: `CHECK: i16x8.le_s{{$}}`.
  **L429 CN**: 注释说明附近代码的意图或约束：`CHECK: i16x8.le_s{{$}}`。
- **L430 EN**: Starts a function or method definition for `test_i16x8_le`.
  **L430 CN**: 开始定义函数或方法 `test_i16x8_le`。
- **L431 EN**: Blank line separating nearby declarations or logic.
  **L431 CN**: 空行，用于分隔相邻声明或逻辑。
- **L432 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_u16x8_le:`.
  **L432 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_u16x8_le:`。

### Lines 433-456

````c
// CHECK: i16x8.le_u{{$}}
v128_t test_u16x8_le(v128_t a, v128_t b) { return wasm_u16x8_le(a, b); }

// CHECK-LABEL: test_i16x8_ge:
// CHECK: i16x8.ge_s{{$}}
v128_t test_i16x8_ge(v128_t a, v128_t b) { return wasm_i16x8_ge(a, b); }

// CHECK-LABEL: test_u16x8_ge:
// CHECK: i16x8.ge_u{{$}}
v128_t test_u16x8_ge(v128_t a, v128_t b) { return wasm_u16x8_ge(a, b); }

// CHECK-LABEL: test_i32x4_eq:
// CHECK: i32x4.eq{{$}}
v128_t test_i32x4_eq(v128_t a, v128_t b) { return wasm_i32x4_eq(a, b); }

// CHECK-LABEL: test_i32x4_ne:
// CHECK: i32x4.ne{{$}}
v128_t test_i32x4_ne(v128_t a, v128_t b) { return wasm_i32x4_ne(a, b); }

// CHECK-LABEL: test_i32x4_lt:
// CHECK: i32x4.lt_s{{$}}
v128_t test_i32x4_lt(v128_t a, v128_t b) { return wasm_i32x4_lt(a, b); }

// CHECK-LABEL: test_u32x4_lt:
````
- **L433 EN**: Comment documents nearby intent or constraints: `CHECK: i16x8.le_u{{$}}`.
  **L433 CN**: 注释说明附近代码的意图或约束：`CHECK: i16x8.le_u{{$}}`。
- **L434 EN**: Starts a function or method definition for `test_u16x8_le`.
  **L434 CN**: 开始定义函数或方法 `test_u16x8_le`。
- **L435 EN**: Blank line separating nearby declarations or logic.
  **L435 CN**: 空行，用于分隔相邻声明或逻辑。
- **L436 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i16x8_ge:`.
  **L436 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i16x8_ge:`。
- **L437 EN**: Comment documents nearby intent or constraints: `CHECK: i16x8.ge_s{{$}}`.
  **L437 CN**: 注释说明附近代码的意图或约束：`CHECK: i16x8.ge_s{{$}}`。
- **L438 EN**: Starts a function or method definition for `test_i16x8_ge`.
  **L438 CN**: 开始定义函数或方法 `test_i16x8_ge`。
- **L439 EN**: Blank line separating nearby declarations or logic.
  **L439 CN**: 空行，用于分隔相邻声明或逻辑。
- **L440 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_u16x8_ge:`.
  **L440 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_u16x8_ge:`。
- **L441 EN**: Comment documents nearby intent or constraints: `CHECK: i16x8.ge_u{{$}}`.
  **L441 CN**: 注释说明附近代码的意图或约束：`CHECK: i16x8.ge_u{{$}}`。
- **L442 EN**: Starts a function or method definition for `test_u16x8_ge`.
  **L442 CN**: 开始定义函数或方法 `test_u16x8_ge`。
- **L443 EN**: Blank line separating nearby declarations or logic.
  **L443 CN**: 空行，用于分隔相邻声明或逻辑。
- **L444 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i32x4_eq:`.
  **L444 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i32x4_eq:`。
- **L445 EN**: Comment documents nearby intent or constraints: `CHECK: i32x4.eq{{$}}`.
  **L445 CN**: 注释说明附近代码的意图或约束：`CHECK: i32x4.eq{{$}}`。
- **L446 EN**: Starts a function or method definition for `test_i32x4_eq`.
  **L446 CN**: 开始定义函数或方法 `test_i32x4_eq`。
- **L447 EN**: Blank line separating nearby declarations or logic.
  **L447 CN**: 空行，用于分隔相邻声明或逻辑。
- **L448 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i32x4_ne:`.
  **L448 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i32x4_ne:`。
- **L449 EN**: Comment documents nearby intent or constraints: `CHECK: i32x4.ne{{$}}`.
  **L449 CN**: 注释说明附近代码的意图或约束：`CHECK: i32x4.ne{{$}}`。
- **L450 EN**: Starts a function or method definition for `test_i32x4_ne`.
  **L450 CN**: 开始定义函数或方法 `test_i32x4_ne`。
- **L451 EN**: Blank line separating nearby declarations or logic.
  **L451 CN**: 空行，用于分隔相邻声明或逻辑。
- **L452 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i32x4_lt:`.
  **L452 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i32x4_lt:`。
- **L453 EN**: Comment documents nearby intent or constraints: `CHECK: i32x4.lt_s{{$}}`.
  **L453 CN**: 注释说明附近代码的意图或约束：`CHECK: i32x4.lt_s{{$}}`。
- **L454 EN**: Starts a function or method definition for `test_i32x4_lt`.
  **L454 CN**: 开始定义函数或方法 `test_i32x4_lt`。
- **L455 EN**: Blank line separating nearby declarations or logic.
  **L455 CN**: 空行，用于分隔相邻声明或逻辑。
- **L456 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_u32x4_lt:`.
  **L456 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_u32x4_lt:`。

### Lines 457-480

````c
// CHECK: i32x4.lt_u{{$}}
v128_t test_u32x4_lt(v128_t a, v128_t b) { return wasm_u32x4_lt(a, b); }

// CHECK-LABEL: test_i32x4_gt:
// CHECK: i32x4.gt_s{{$}}
v128_t test_i32x4_gt(v128_t a, v128_t b) { return wasm_i32x4_gt(a, b); }

// CHECK-LABEL: test_u32x4_gt:
// CHECK: i32x4.gt_u{{$}}
v128_t test_u32x4_gt(v128_t a, v128_t b) { return wasm_u32x4_gt(a, b); }

// CHECK-LABEL: test_i32x4_le:
// CHECK: i32x4.le_s{{$}}
v128_t test_i32x4_le(v128_t a, v128_t b) { return wasm_i32x4_le(a, b); }

// CHECK-LABEL: test_u32x4_le:
// CHECK: i32x4.le_u{{$}}
v128_t test_u32x4_le(v128_t a, v128_t b) { return wasm_u32x4_le(a, b); }

// CHECK-LABEL: test_i32x4_ge:
// CHECK: i32x4.ge_s{{$}}
v128_t test_i32x4_ge(v128_t a, v128_t b) { return wasm_i32x4_ge(a, b); }

// CHECK-LABEL: test_u32x4_ge:
````
- **L457 EN**: Comment documents nearby intent or constraints: `CHECK: i32x4.lt_u{{$}}`.
  **L457 CN**: 注释说明附近代码的意图或约束：`CHECK: i32x4.lt_u{{$}}`。
- **L458 EN**: Starts a function or method definition for `test_u32x4_lt`.
  **L458 CN**: 开始定义函数或方法 `test_u32x4_lt`。
- **L459 EN**: Blank line separating nearby declarations or logic.
  **L459 CN**: 空行，用于分隔相邻声明或逻辑。
- **L460 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i32x4_gt:`.
  **L460 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i32x4_gt:`。
- **L461 EN**: Comment documents nearby intent or constraints: `CHECK: i32x4.gt_s{{$}}`.
  **L461 CN**: 注释说明附近代码的意图或约束：`CHECK: i32x4.gt_s{{$}}`。
- **L462 EN**: Starts a function or method definition for `test_i32x4_gt`.
  **L462 CN**: 开始定义函数或方法 `test_i32x4_gt`。
- **L463 EN**: Blank line separating nearby declarations or logic.
  **L463 CN**: 空行，用于分隔相邻声明或逻辑。
- **L464 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_u32x4_gt:`.
  **L464 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_u32x4_gt:`。
- **L465 EN**: Comment documents nearby intent or constraints: `CHECK: i32x4.gt_u{{$}}`.
  **L465 CN**: 注释说明附近代码的意图或约束：`CHECK: i32x4.gt_u{{$}}`。
- **L466 EN**: Starts a function or method definition for `test_u32x4_gt`.
  **L466 CN**: 开始定义函数或方法 `test_u32x4_gt`。
- **L467 EN**: Blank line separating nearby declarations or logic.
  **L467 CN**: 空行，用于分隔相邻声明或逻辑。
- **L468 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i32x4_le:`.
  **L468 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i32x4_le:`。
- **L469 EN**: Comment documents nearby intent or constraints: `CHECK: i32x4.le_s{{$}}`.
  **L469 CN**: 注释说明附近代码的意图或约束：`CHECK: i32x4.le_s{{$}}`。
- **L470 EN**: Starts a function or method definition for `test_i32x4_le`.
  **L470 CN**: 开始定义函数或方法 `test_i32x4_le`。
- **L471 EN**: Blank line separating nearby declarations or logic.
  **L471 CN**: 空行，用于分隔相邻声明或逻辑。
- **L472 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_u32x4_le:`.
  **L472 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_u32x4_le:`。
- **L473 EN**: Comment documents nearby intent or constraints: `CHECK: i32x4.le_u{{$}}`.
  **L473 CN**: 注释说明附近代码的意图或约束：`CHECK: i32x4.le_u{{$}}`。
- **L474 EN**: Starts a function or method definition for `test_u32x4_le`.
  **L474 CN**: 开始定义函数或方法 `test_u32x4_le`。
- **L475 EN**: Blank line separating nearby declarations or logic.
  **L475 CN**: 空行，用于分隔相邻声明或逻辑。
- **L476 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i32x4_ge:`.
  **L476 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i32x4_ge:`。
- **L477 EN**: Comment documents nearby intent or constraints: `CHECK: i32x4.ge_s{{$}}`.
  **L477 CN**: 注释说明附近代码的意图或约束：`CHECK: i32x4.ge_s{{$}}`。
- **L478 EN**: Starts a function or method definition for `test_i32x4_ge`.
  **L478 CN**: 开始定义函数或方法 `test_i32x4_ge`。
- **L479 EN**: Blank line separating nearby declarations or logic.
  **L479 CN**: 空行，用于分隔相邻声明或逻辑。
- **L480 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_u32x4_ge:`.
  **L480 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_u32x4_ge:`。

### Lines 481-504

````c
// CHECK: i32x4.ge_u{{$}}
v128_t test_u32x4_ge(v128_t a, v128_t b) { return wasm_u32x4_ge(a, b); }

// CHECK-LABEL: test_i64x2_eq:
// CHECK: i64x2.eq{{$}}
v128_t test_i64x2_eq(v128_t a, v128_t b) { return wasm_i64x2_eq(a, b); }

// CHECK-LABEL: test_i64x2_ne:
// CHECK: i64x2.ne{{$}}
v128_t test_i64x2_ne(v128_t a, v128_t b) { return wasm_i64x2_ne(a, b); }

// CHECK-LABEL: test_i64x2_lt:
// CHECK: i64x2.lt_s{{$}}
v128_t test_i64x2_lt(v128_t a, v128_t b) { return wasm_i64x2_lt(a, b); }

// CHECK-LABEL: test_i64x2_gt:
// CHECK: i64x2.gt_s{{$}}
v128_t test_i64x2_gt(v128_t a, v128_t b) { return wasm_i64x2_gt(a, b); }

// CHECK-LABEL: test_i64x2_le:
// CHECK: i64x2.le_s{{$}}
v128_t test_i64x2_le(v128_t a, v128_t b) { return wasm_i64x2_le(a, b); }

// CHECK-LABEL: test_i64x2_ge:
````
- **L481 EN**: Comment documents nearby intent or constraints: `CHECK: i32x4.ge_u{{$}}`.
  **L481 CN**: 注释说明附近代码的意图或约束：`CHECK: i32x4.ge_u{{$}}`。
- **L482 EN**: Starts a function or method definition for `test_u32x4_ge`.
  **L482 CN**: 开始定义函数或方法 `test_u32x4_ge`。
- **L483 EN**: Blank line separating nearby declarations or logic.
  **L483 CN**: 空行，用于分隔相邻声明或逻辑。
- **L484 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i64x2_eq:`.
  **L484 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i64x2_eq:`。
- **L485 EN**: Comment documents nearby intent or constraints: `CHECK: i64x2.eq{{$}}`.
  **L485 CN**: 注释说明附近代码的意图或约束：`CHECK: i64x2.eq{{$}}`。
- **L486 EN**: Starts a function or method definition for `test_i64x2_eq`.
  **L486 CN**: 开始定义函数或方法 `test_i64x2_eq`。
- **L487 EN**: Blank line separating nearby declarations or logic.
  **L487 CN**: 空行，用于分隔相邻声明或逻辑。
- **L488 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i64x2_ne:`.
  **L488 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i64x2_ne:`。
- **L489 EN**: Comment documents nearby intent or constraints: `CHECK: i64x2.ne{{$}}`.
  **L489 CN**: 注释说明附近代码的意图或约束：`CHECK: i64x2.ne{{$}}`。
- **L490 EN**: Starts a function or method definition for `test_i64x2_ne`.
  **L490 CN**: 开始定义函数或方法 `test_i64x2_ne`。
- **L491 EN**: Blank line separating nearby declarations or logic.
  **L491 CN**: 空行，用于分隔相邻声明或逻辑。
- **L492 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i64x2_lt:`.
  **L492 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i64x2_lt:`。
- **L493 EN**: Comment documents nearby intent or constraints: `CHECK: i64x2.lt_s{{$}}`.
  **L493 CN**: 注释说明附近代码的意图或约束：`CHECK: i64x2.lt_s{{$}}`。
- **L494 EN**: Starts a function or method definition for `test_i64x2_lt`.
  **L494 CN**: 开始定义函数或方法 `test_i64x2_lt`。
- **L495 EN**: Blank line separating nearby declarations or logic.
  **L495 CN**: 空行，用于分隔相邻声明或逻辑。
- **L496 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i64x2_gt:`.
  **L496 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i64x2_gt:`。
- **L497 EN**: Comment documents nearby intent or constraints: `CHECK: i64x2.gt_s{{$}}`.
  **L497 CN**: 注释说明附近代码的意图或约束：`CHECK: i64x2.gt_s{{$}}`。
- **L498 EN**: Starts a function or method definition for `test_i64x2_gt`.
  **L498 CN**: 开始定义函数或方法 `test_i64x2_gt`。
- **L499 EN**: Blank line separating nearby declarations or logic.
  **L499 CN**: 空行，用于分隔相邻声明或逻辑。
- **L500 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i64x2_le:`.
  **L500 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i64x2_le:`。
- **L501 EN**: Comment documents nearby intent or constraints: `CHECK: i64x2.le_s{{$}}`.
  **L501 CN**: 注释说明附近代码的意图或约束：`CHECK: i64x2.le_s{{$}}`。
- **L502 EN**: Starts a function or method definition for `test_i64x2_le`.
  **L502 CN**: 开始定义函数或方法 `test_i64x2_le`。
- **L503 EN**: Blank line separating nearby declarations or logic.
  **L503 CN**: 空行，用于分隔相邻声明或逻辑。
- **L504 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i64x2_ge:`.
  **L504 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i64x2_ge:`。

### Lines 505-528

````c
// CHECK: i64x2.ge_s{{$}}
v128_t test_i64x2_ge(v128_t a, v128_t b) { return wasm_i64x2_ge(a, b); }

// CHECK-LABEL: test_f32x4_eq:
// CHECK: f32x4.eq{{$}}
v128_t test_f32x4_eq(v128_t a, v128_t b) { return wasm_f32x4_eq(a, b); }

// CHECK-LABEL: test_f32x4_ne:
// CHECK: f32x4.ne{{$}}
v128_t test_f32x4_ne(v128_t a, v128_t b) { return wasm_f32x4_ne(a, b); }

// CHECK-LABEL: test_f32x4_lt:
// CHECK: f32x4.lt{{$}}
v128_t test_f32x4_lt(v128_t a, v128_t b) { return wasm_f32x4_lt(a, b); }

// CHECK-LABEL: test_f32x4_gt:
// CHECK: f32x4.gt{{$}}
v128_t test_f32x4_gt(v128_t a, v128_t b) { return wasm_f32x4_gt(a, b); }

// CHECK-LABEL: test_f32x4_le:
// CHECK: f32x4.le{{$}}
v128_t test_f32x4_le(v128_t a, v128_t b) { return wasm_f32x4_le(a, b); }

// CHECK-LABEL: test_f32x4_ge:
````
- **L505 EN**: Comment documents nearby intent or constraints: `CHECK: i64x2.ge_s{{$}}`.
  **L505 CN**: 注释说明附近代码的意图或约束：`CHECK: i64x2.ge_s{{$}}`。
- **L506 EN**: Starts a function or method definition for `test_i64x2_ge`.
  **L506 CN**: 开始定义函数或方法 `test_i64x2_ge`。
- **L507 EN**: Blank line separating nearby declarations or logic.
  **L507 CN**: 空行，用于分隔相邻声明或逻辑。
- **L508 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f32x4_eq:`.
  **L508 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f32x4_eq:`。
- **L509 EN**: Comment documents nearby intent or constraints: `CHECK: f32x4.eq{{$}}`.
  **L509 CN**: 注释说明附近代码的意图或约束：`CHECK: f32x4.eq{{$}}`。
- **L510 EN**: Starts a function or method definition for `test_f32x4_eq`.
  **L510 CN**: 开始定义函数或方法 `test_f32x4_eq`。
- **L511 EN**: Blank line separating nearby declarations or logic.
  **L511 CN**: 空行，用于分隔相邻声明或逻辑。
- **L512 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f32x4_ne:`.
  **L512 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f32x4_ne:`。
- **L513 EN**: Comment documents nearby intent or constraints: `CHECK: f32x4.ne{{$}}`.
  **L513 CN**: 注释说明附近代码的意图或约束：`CHECK: f32x4.ne{{$}}`。
- **L514 EN**: Starts a function or method definition for `test_f32x4_ne`.
  **L514 CN**: 开始定义函数或方法 `test_f32x4_ne`。
- **L515 EN**: Blank line separating nearby declarations or logic.
  **L515 CN**: 空行，用于分隔相邻声明或逻辑。
- **L516 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f32x4_lt:`.
  **L516 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f32x4_lt:`。
- **L517 EN**: Comment documents nearby intent or constraints: `CHECK: f32x4.lt{{$}}`.
  **L517 CN**: 注释说明附近代码的意图或约束：`CHECK: f32x4.lt{{$}}`。
- **L518 EN**: Starts a function or method definition for `test_f32x4_lt`.
  **L518 CN**: 开始定义函数或方法 `test_f32x4_lt`。
- **L519 EN**: Blank line separating nearby declarations or logic.
  **L519 CN**: 空行，用于分隔相邻声明或逻辑。
- **L520 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f32x4_gt:`.
  **L520 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f32x4_gt:`。
- **L521 EN**: Comment documents nearby intent or constraints: `CHECK: f32x4.gt{{$}}`.
  **L521 CN**: 注释说明附近代码的意图或约束：`CHECK: f32x4.gt{{$}}`。
- **L522 EN**: Starts a function or method definition for `test_f32x4_gt`.
  **L522 CN**: 开始定义函数或方法 `test_f32x4_gt`。
- **L523 EN**: Blank line separating nearby declarations or logic.
  **L523 CN**: 空行，用于分隔相邻声明或逻辑。
- **L524 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f32x4_le:`.
  **L524 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f32x4_le:`。
- **L525 EN**: Comment documents nearby intent or constraints: `CHECK: f32x4.le{{$}}`.
  **L525 CN**: 注释说明附近代码的意图或约束：`CHECK: f32x4.le{{$}}`。
- **L526 EN**: Starts a function or method definition for `test_f32x4_le`.
  **L526 CN**: 开始定义函数或方法 `test_f32x4_le`。
- **L527 EN**: Blank line separating nearby declarations or logic.
  **L527 CN**: 空行，用于分隔相邻声明或逻辑。
- **L528 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f32x4_ge:`.
  **L528 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f32x4_ge:`。

### Lines 529-552

````c
// CHECK: f32x4.ge{{$}}
v128_t test_f32x4_ge(v128_t a, v128_t b) { return wasm_f32x4_ge(a, b); }

// CHECK-LABEL: test_f64x2_eq:
// CHECK: f64x2.eq{{$}}
v128_t test_f64x2_eq(v128_t a, v128_t b) { return wasm_f64x2_eq(a, b); }

// CHECK-LABEL: test_f64x2_ne:
// CHECK: f64x2.ne{{$}}
v128_t test_f64x2_ne(v128_t a, v128_t b) { return wasm_f64x2_ne(a, b); }

// CHECK-LABEL: test_f64x2_lt:
// CHECK: f64x2.lt{{$}}
v128_t test_f64x2_lt(v128_t a, v128_t b) { return wasm_f64x2_lt(a, b); }

// CHECK-LABEL: test_f64x2_gt:
// CHECK: f64x2.gt{{$}}
v128_t test_f64x2_gt(v128_t a, v128_t b) { return wasm_f64x2_gt(a, b); }

// CHECK-LABEL: test_f64x2_le:
// CHECK: f64x2.le{{$}}
v128_t test_f64x2_le(v128_t a, v128_t b) { return wasm_f64x2_le(a, b); }

// CHECK-LABEL: test_f64x2_ge:
````
- **L529 EN**: Comment documents nearby intent or constraints: `CHECK: f32x4.ge{{$}}`.
  **L529 CN**: 注释说明附近代码的意图或约束：`CHECK: f32x4.ge{{$}}`。
- **L530 EN**: Starts a function or method definition for `test_f32x4_ge`.
  **L530 CN**: 开始定义函数或方法 `test_f32x4_ge`。
- **L531 EN**: Blank line separating nearby declarations or logic.
  **L531 CN**: 空行，用于分隔相邻声明或逻辑。
- **L532 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f64x2_eq:`.
  **L532 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f64x2_eq:`。
- **L533 EN**: Comment documents nearby intent or constraints: `CHECK: f64x2.eq{{$}}`.
  **L533 CN**: 注释说明附近代码的意图或约束：`CHECK: f64x2.eq{{$}}`。
- **L534 EN**: Starts a function or method definition for `test_f64x2_eq`.
  **L534 CN**: 开始定义函数或方法 `test_f64x2_eq`。
- **L535 EN**: Blank line separating nearby declarations or logic.
  **L535 CN**: 空行，用于分隔相邻声明或逻辑。
- **L536 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f64x2_ne:`.
  **L536 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f64x2_ne:`。
- **L537 EN**: Comment documents nearby intent or constraints: `CHECK: f64x2.ne{{$}}`.
  **L537 CN**: 注释说明附近代码的意图或约束：`CHECK: f64x2.ne{{$}}`。
- **L538 EN**: Starts a function or method definition for `test_f64x2_ne`.
  **L538 CN**: 开始定义函数或方法 `test_f64x2_ne`。
- **L539 EN**: Blank line separating nearby declarations or logic.
  **L539 CN**: 空行，用于分隔相邻声明或逻辑。
- **L540 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f64x2_lt:`.
  **L540 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f64x2_lt:`。
- **L541 EN**: Comment documents nearby intent or constraints: `CHECK: f64x2.lt{{$}}`.
  **L541 CN**: 注释说明附近代码的意图或约束：`CHECK: f64x2.lt{{$}}`。
- **L542 EN**: Starts a function or method definition for `test_f64x2_lt`.
  **L542 CN**: 开始定义函数或方法 `test_f64x2_lt`。
- **L543 EN**: Blank line separating nearby declarations or logic.
  **L543 CN**: 空行，用于分隔相邻声明或逻辑。
- **L544 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f64x2_gt:`.
  **L544 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f64x2_gt:`。
- **L545 EN**: Comment documents nearby intent or constraints: `CHECK: f64x2.gt{{$}}`.
  **L545 CN**: 注释说明附近代码的意图或约束：`CHECK: f64x2.gt{{$}}`。
- **L546 EN**: Starts a function or method definition for `test_f64x2_gt`.
  **L546 CN**: 开始定义函数或方法 `test_f64x2_gt`。
- **L547 EN**: Blank line separating nearby declarations or logic.
  **L547 CN**: 空行，用于分隔相邻声明或逻辑。
- **L548 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f64x2_le:`.
  **L548 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f64x2_le:`。
- **L549 EN**: Comment documents nearby intent or constraints: `CHECK: f64x2.le{{$}}`.
  **L549 CN**: 注释说明附近代码的意图或约束：`CHECK: f64x2.le{{$}}`。
- **L550 EN**: Starts a function or method definition for `test_f64x2_le`.
  **L550 CN**: 开始定义函数或方法 `test_f64x2_le`。
- **L551 EN**: Blank line separating nearby declarations or logic.
  **L551 CN**: 空行，用于分隔相邻声明或逻辑。
- **L552 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f64x2_ge:`.
  **L552 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f64x2_ge:`。

### Lines 553-576

````c
// CHECK: f64x2.ge{{$}}
v128_t test_f64x2_ge(v128_t a, v128_t b) { return wasm_f64x2_ge(a, b); }

// CHECK-LABEL: test_v128_not:
// CHECK: v128.not{{$}}
v128_t test_v128_not(v128_t a) { return wasm_v128_not(a); }

// CHECK-LABEL: test_v128_and:
// CHECK: v128.and{{$}}
v128_t test_v128_and(v128_t a, v128_t b) { return wasm_v128_and(a, b); }

// CHECK-LABEL: test_v128_or:
// CHECK: v128.or{{$}}
v128_t test_v128_or(v128_t a, v128_t b) { return wasm_v128_or(a, b); }

// CHECK-LABEL: test_v128_xor:
// CHECK: v128.xor{{$}}
v128_t test_v128_xor(v128_t a, v128_t b) { return wasm_v128_xor(a, b); }

// CHECK-LABEL: test_v128_andnot:
// CHECK: v128.andnot{{$}}
v128_t test_v128_andnot(v128_t a, v128_t b) { return wasm_v128_andnot(a, b); }

// CHECK-LABEL: test_v128_any_true:
````
- **L553 EN**: Comment documents nearby intent or constraints: `CHECK: f64x2.ge{{$}}`.
  **L553 CN**: 注释说明附近代码的意图或约束：`CHECK: f64x2.ge{{$}}`。
- **L554 EN**: Starts a function or method definition for `test_f64x2_ge`.
  **L554 CN**: 开始定义函数或方法 `test_f64x2_ge`。
- **L555 EN**: Blank line separating nearby declarations or logic.
  **L555 CN**: 空行，用于分隔相邻声明或逻辑。
- **L556 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_v128_not:`.
  **L556 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_v128_not:`。
- **L557 EN**: Comment documents nearby intent or constraints: `CHECK: v128.not{{$}}`.
  **L557 CN**: 注释说明附近代码的意图或约束：`CHECK: v128.not{{$}}`。
- **L558 EN**: Starts a function or method definition for `test_v128_not`.
  **L558 CN**: 开始定义函数或方法 `test_v128_not`。
- **L559 EN**: Blank line separating nearby declarations or logic.
  **L559 CN**: 空行，用于分隔相邻声明或逻辑。
- **L560 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_v128_and:`.
  **L560 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_v128_and:`。
- **L561 EN**: Comment documents nearby intent or constraints: `CHECK: v128.and{{$}}`.
  **L561 CN**: 注释说明附近代码的意图或约束：`CHECK: v128.and{{$}}`。
- **L562 EN**: Starts a function or method definition for `test_v128_and`.
  **L562 CN**: 开始定义函数或方法 `test_v128_and`。
- **L563 EN**: Blank line separating nearby declarations or logic.
  **L563 CN**: 空行，用于分隔相邻声明或逻辑。
- **L564 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_v128_or:`.
  **L564 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_v128_or:`。
- **L565 EN**: Comment documents nearby intent or constraints: `CHECK: v128.or{{$}}`.
  **L565 CN**: 注释说明附近代码的意图或约束：`CHECK: v128.or{{$}}`。
- **L566 EN**: Starts a function or method definition for `test_v128_or`.
  **L566 CN**: 开始定义函数或方法 `test_v128_or`。
- **L567 EN**: Blank line separating nearby declarations or logic.
  **L567 CN**: 空行，用于分隔相邻声明或逻辑。
- **L568 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_v128_xor:`.
  **L568 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_v128_xor:`。
- **L569 EN**: Comment documents nearby intent or constraints: `CHECK: v128.xor{{$}}`.
  **L569 CN**: 注释说明附近代码的意图或约束：`CHECK: v128.xor{{$}}`。
- **L570 EN**: Starts a function or method definition for `test_v128_xor`.
  **L570 CN**: 开始定义函数或方法 `test_v128_xor`。
- **L571 EN**: Blank line separating nearby declarations or logic.
  **L571 CN**: 空行，用于分隔相邻声明或逻辑。
- **L572 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_v128_andnot:`.
  **L572 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_v128_andnot:`。
- **L573 EN**: Comment documents nearby intent or constraints: `CHECK: v128.andnot{{$}}`.
  **L573 CN**: 注释说明附近代码的意图或约束：`CHECK: v128.andnot{{$}}`。
- **L574 EN**: Starts a function or method definition for `test_v128_andnot`.
  **L574 CN**: 开始定义函数或方法 `test_v128_andnot`。
- **L575 EN**: Blank line separating nearby declarations or logic.
  **L575 CN**: 空行，用于分隔相邻声明或逻辑。
- **L576 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_v128_any_true:`.
  **L576 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_v128_any_true:`。

### Lines 577-600

````c
// CHECK: v128.any_true{{$}}
bool test_v128_any_true(v128_t a) { return wasm_v128_any_true(a); }

// CHECK-LABEL: test_v128_bitselect:
// CHECK: v128.bitselect{{$}}
v128_t test_v128_bitselect(v128_t a, v128_t b, v128_t mask) {
  return wasm_v128_bitselect(a, b, mask);
}

// CHECK-LABEL: test_i8x16_abs:
// CHECK: i8x16.abs{{$}}
v128_t test_i8x16_abs(v128_t a) { return wasm_i8x16_abs(a); }

// CHECK-LABEL: test_i8x16_neg:
// CHECK: i8x16.neg{{$}}
v128_t test_i8x16_neg(v128_t a) { return wasm_i8x16_neg(a); }

// CHECK-LABEL: test_i8x16_all_true:
// CHECK: i8x16.all_true{{$}}
bool test_i8x16_all_true(v128_t a) { return wasm_i8x16_all_true(a); }

// CHECK-LABEL: test_i8x16_bitmask:
// CHECK: i8x16.bitmask{{$}}
int32_t test_i8x16_bitmask(v128_t a) { return wasm_i8x16_bitmask(a); }
````
- **L577 EN**: Comment documents nearby intent or constraints: `CHECK: v128.any_true{{$}}`.
  **L577 CN**: 注释说明附近代码的意图或约束：`CHECK: v128.any_true{{$}}`。
- **L578 EN**: Starts a function or method definition for `test_v128_any_true`.
  **L578 CN**: 开始定义函数或方法 `test_v128_any_true`。
- **L579 EN**: Blank line separating nearby declarations or logic.
  **L579 CN**: 空行，用于分隔相邻声明或逻辑。
- **L580 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_v128_bitselect:`.
  **L580 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_v128_bitselect:`。
- **L581 EN**: Comment documents nearby intent or constraints: `CHECK: v128.bitselect{{$}}`.
  **L581 CN**: 注释说明附近代码的意图或约束：`CHECK: v128.bitselect{{$}}`。
- **L582 EN**: Starts a function or method definition for `test_v128_bitselect`.
  **L582 CN**: 开始定义函数或方法 `test_v128_bitselect`。
- **L583 EN**: Returns from the current function with `wasm_v128_bitselect(a, b, mask)`.
  **L583 CN**: 以 `wasm_v128_bitselect(a, b, mask)` 从当前函数返回。
- **L584 EN**: Closes the current lexical scope or compound statement.
  **L584 CN**: 结束当前词法作用域或复合语句块。
- **L585 EN**: Blank line separating nearby declarations or logic.
  **L585 CN**: 空行，用于分隔相邻声明或逻辑。
- **L586 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i8x16_abs:`.
  **L586 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i8x16_abs:`。
- **L587 EN**: Comment documents nearby intent or constraints: `CHECK: i8x16.abs{{$}}`.
  **L587 CN**: 注释说明附近代码的意图或约束：`CHECK: i8x16.abs{{$}}`。
- **L588 EN**: Starts a function or method definition for `test_i8x16_abs`.
  **L588 CN**: 开始定义函数或方法 `test_i8x16_abs`。
- **L589 EN**: Blank line separating nearby declarations or logic.
  **L589 CN**: 空行，用于分隔相邻声明或逻辑。
- **L590 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i8x16_neg:`.
  **L590 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i8x16_neg:`。
- **L591 EN**: Comment documents nearby intent or constraints: `CHECK: i8x16.neg{{$}}`.
  **L591 CN**: 注释说明附近代码的意图或约束：`CHECK: i8x16.neg{{$}}`。
- **L592 EN**: Starts a function or method definition for `test_i8x16_neg`.
  **L592 CN**: 开始定义函数或方法 `test_i8x16_neg`。
- **L593 EN**: Blank line separating nearby declarations or logic.
  **L593 CN**: 空行，用于分隔相邻声明或逻辑。
- **L594 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i8x16_all_true:`.
  **L594 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i8x16_all_true:`。
- **L595 EN**: Comment documents nearby intent or constraints: `CHECK: i8x16.all_true{{$}}`.
  **L595 CN**: 注释说明附近代码的意图或约束：`CHECK: i8x16.all_true{{$}}`。
- **L596 EN**: Starts a function or method definition for `test_i8x16_all_true`.
  **L596 CN**: 开始定义函数或方法 `test_i8x16_all_true`。
- **L597 EN**: Blank line separating nearby declarations or logic.
  **L597 CN**: 空行，用于分隔相邻声明或逻辑。
- **L598 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i8x16_bitmask:`.
  **L598 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i8x16_bitmask:`。
- **L599 EN**: Comment documents nearby intent or constraints: `CHECK: i8x16.bitmask{{$}}`.
  **L599 CN**: 注释说明附近代码的意图或约束：`CHECK: i8x16.bitmask{{$}}`。
- **L600 EN**: Starts a function or method definition for `test_i8x16_bitmask`.
  **L600 CN**: 开始定义函数或方法 `test_i8x16_bitmask`。

### Lines 601-624

````c

// CHECK-LABEL: test_i8x16_popcnt:
// CHECK: i8x16.popcnt{{$}}
v128_t test_i8x16_popcnt(v128_t a) { return wasm_i8x16_popcnt(a); }

// CHECK-LABEL: test_i8x16_shl:
// CHECK: i8x16.shl{{$}}
v128_t test_i8x16_shl(v128_t a, int32_t b) { return wasm_i8x16_shl(a, b); }

// CHECK-LABEL: test_i8x16_shr:
// CHECK: i8x16.shr_s{{$}}
v128_t test_i8x16_shr(v128_t a, int32_t b) { return wasm_i8x16_shr(a, b); }

// CHECK-LABEL: test_u8x16_shr:
// CHECK: i8x16.shr_u{{$}}
v128_t test_u8x16_shr(v128_t a, int32_t b) { return wasm_u8x16_shr(a, b); }

// CHECK-LABEL: test_i8x16_add:
// CHECK: i8x16.add{{$}}
v128_t test_i8x16_add(v128_t a, v128_t b) { return wasm_i8x16_add(a, b); }

// CHECK-LABEL: test_i8x16_add_sat:
// CHECK: i8x16.add_sat_s{{$}}
v128_t test_i8x16_add_sat(v128_t a, v128_t b) {
````
- **L601 EN**: Blank line separating nearby declarations or logic.
  **L601 CN**: 空行，用于分隔相邻声明或逻辑。
- **L602 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i8x16_popcnt:`.
  **L602 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i8x16_popcnt:`。
- **L603 EN**: Comment documents nearby intent or constraints: `CHECK: i8x16.popcnt{{$}}`.
  **L603 CN**: 注释说明附近代码的意图或约束：`CHECK: i8x16.popcnt{{$}}`。
- **L604 EN**: Starts a function or method definition for `test_i8x16_popcnt`.
  **L604 CN**: 开始定义函数或方法 `test_i8x16_popcnt`。
- **L605 EN**: Blank line separating nearby declarations or logic.
  **L605 CN**: 空行，用于分隔相邻声明或逻辑。
- **L606 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i8x16_shl:`.
  **L606 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i8x16_shl:`。
- **L607 EN**: Comment documents nearby intent or constraints: `CHECK: i8x16.shl{{$}}`.
  **L607 CN**: 注释说明附近代码的意图或约束：`CHECK: i8x16.shl{{$}}`。
- **L608 EN**: Starts a function or method definition for `test_i8x16_shl`.
  **L608 CN**: 开始定义函数或方法 `test_i8x16_shl`。
- **L609 EN**: Blank line separating nearby declarations or logic.
  **L609 CN**: 空行，用于分隔相邻声明或逻辑。
- **L610 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i8x16_shr:`.
  **L610 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i8x16_shr:`。
- **L611 EN**: Comment documents nearby intent or constraints: `CHECK: i8x16.shr_s{{$}}`.
  **L611 CN**: 注释说明附近代码的意图或约束：`CHECK: i8x16.shr_s{{$}}`。
- **L612 EN**: Starts a function or method definition for `test_i8x16_shr`.
  **L612 CN**: 开始定义函数或方法 `test_i8x16_shr`。
- **L613 EN**: Blank line separating nearby declarations or logic.
  **L613 CN**: 空行，用于分隔相邻声明或逻辑。
- **L614 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_u8x16_shr:`.
  **L614 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_u8x16_shr:`。
- **L615 EN**: Comment documents nearby intent or constraints: `CHECK: i8x16.shr_u{{$}}`.
  **L615 CN**: 注释说明附近代码的意图或约束：`CHECK: i8x16.shr_u{{$}}`。
- **L616 EN**: Starts a function or method definition for `test_u8x16_shr`.
  **L616 CN**: 开始定义函数或方法 `test_u8x16_shr`。
- **L617 EN**: Blank line separating nearby declarations or logic.
  **L617 CN**: 空行，用于分隔相邻声明或逻辑。
- **L618 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i8x16_add:`.
  **L618 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i8x16_add:`。
- **L619 EN**: Comment documents nearby intent or constraints: `CHECK: i8x16.add{{$}}`.
  **L619 CN**: 注释说明附近代码的意图或约束：`CHECK: i8x16.add{{$}}`。
- **L620 EN**: Starts a function or method definition for `test_i8x16_add`.
  **L620 CN**: 开始定义函数或方法 `test_i8x16_add`。
- **L621 EN**: Blank line separating nearby declarations or logic.
  **L621 CN**: 空行，用于分隔相邻声明或逻辑。
- **L622 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i8x16_add_sat:`.
  **L622 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i8x16_add_sat:`。
- **L623 EN**: Comment documents nearby intent or constraints: `CHECK: i8x16.add_sat_s{{$}}`.
  **L623 CN**: 注释说明附近代码的意图或约束：`CHECK: i8x16.add_sat_s{{$}}`。
- **L624 EN**: Uses compile-time type-trait machinery to inspect or transform types.
  **L624 CN**: 使用编译期类型萃取机制来检查或变换类型。

### Lines 625-648

````c
  return wasm_i8x16_add_sat(a, b);
}

// CHECK-LABEL: test_u8x16_add_sat:
// CHECK: i8x16.add_sat_u{{$}}
v128_t test_u8x16_add_sat(v128_t a, v128_t b) {
  return wasm_u8x16_add_sat(a, b);
}

// CHECK-LABEL: test_i8x16_sub:
// CHECK: i8x16.sub{{$}}
v128_t test_i8x16_sub(v128_t a, v128_t b) { return wasm_i8x16_sub(a, b); }

// CHECK-LABEL: test_i8x16_sub_sat:
// CHECK: i8x16.sub_sat_s{{$}}
v128_t test_i8x16_sub_sat(v128_t a, v128_t b) {
  return wasm_i8x16_sub_sat(a, b);
}

// CHECK-LABEL: test_u8x16_sub_sat:
// CHECK: i8x16.sub_sat_u{{$}}
v128_t test_u8x16_sub_sat(v128_t a, v128_t b) {
  return wasm_u8x16_sub_sat(a, b);
}
````
- **L625 EN**: Returns from the current function with `wasm_i8x16_add_sat(a, b)`.
  **L625 CN**: 以 `wasm_i8x16_add_sat(a, b)` 从当前函数返回。
- **L626 EN**: Closes the current lexical scope or compound statement.
  **L626 CN**: 结束当前词法作用域或复合语句块。
- **L627 EN**: Blank line separating nearby declarations or logic.
  **L627 CN**: 空行，用于分隔相邻声明或逻辑。
- **L628 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_u8x16_add_sat:`.
  **L628 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_u8x16_add_sat:`。
- **L629 EN**: Comment documents nearby intent or constraints: `CHECK: i8x16.add_sat_u{{$}}`.
  **L629 CN**: 注释说明附近代码的意图或约束：`CHECK: i8x16.add_sat_u{{$}}`。
- **L630 EN**: Uses compile-time type-trait machinery to inspect or transform types.
  **L630 CN**: 使用编译期类型萃取机制来检查或变换类型。
- **L631 EN**: Returns from the current function with `wasm_u8x16_add_sat(a, b)`.
  **L631 CN**: 以 `wasm_u8x16_add_sat(a, b)` 从当前函数返回。
- **L632 EN**: Closes the current lexical scope or compound statement.
  **L632 CN**: 结束当前词法作用域或复合语句块。
- **L633 EN**: Blank line separating nearby declarations or logic.
  **L633 CN**: 空行，用于分隔相邻声明或逻辑。
- **L634 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i8x16_sub:`.
  **L634 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i8x16_sub:`。
- **L635 EN**: Comment documents nearby intent or constraints: `CHECK: i8x16.sub{{$}}`.
  **L635 CN**: 注释说明附近代码的意图或约束：`CHECK: i8x16.sub{{$}}`。
- **L636 EN**: Starts a function or method definition for `test_i8x16_sub`.
  **L636 CN**: 开始定义函数或方法 `test_i8x16_sub`。
- **L637 EN**: Blank line separating nearby declarations or logic.
  **L637 CN**: 空行，用于分隔相邻声明或逻辑。
- **L638 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i8x16_sub_sat:`.
  **L638 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i8x16_sub_sat:`。
- **L639 EN**: Comment documents nearby intent or constraints: `CHECK: i8x16.sub_sat_s{{$}}`.
  **L639 CN**: 注释说明附近代码的意图或约束：`CHECK: i8x16.sub_sat_s{{$}}`。
- **L640 EN**: Starts a function or method definition for `test_i8x16_sub_sat`.
  **L640 CN**: 开始定义函数或方法 `test_i8x16_sub_sat`。
- **L641 EN**: Returns from the current function with `wasm_i8x16_sub_sat(a, b)`.
  **L641 CN**: 以 `wasm_i8x16_sub_sat(a, b)` 从当前函数返回。
- **L642 EN**: Closes the current lexical scope or compound statement.
  **L642 CN**: 结束当前词法作用域或复合语句块。
- **L643 EN**: Blank line separating nearby declarations or logic.
  **L643 CN**: 空行，用于分隔相邻声明或逻辑。
- **L644 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_u8x16_sub_sat:`.
  **L644 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_u8x16_sub_sat:`。
- **L645 EN**: Comment documents nearby intent or constraints: `CHECK: i8x16.sub_sat_u{{$}}`.
  **L645 CN**: 注释说明附近代码的意图或约束：`CHECK: i8x16.sub_sat_u{{$}}`。
- **L646 EN**: Starts a function or method definition for `test_u8x16_sub_sat`.
  **L646 CN**: 开始定义函数或方法 `test_u8x16_sub_sat`。
- **L647 EN**: Returns from the current function with `wasm_u8x16_sub_sat(a, b)`.
  **L647 CN**: 以 `wasm_u8x16_sub_sat(a, b)` 从当前函数返回。
- **L648 EN**: Closes the current lexical scope or compound statement.
  **L648 CN**: 结束当前词法作用域或复合语句块。

### Lines 649-672

````c

// CHECK-LABEL: test_i8x16_min:
// CHECK: i8x16.min_s{{$}}
v128_t test_i8x16_min(v128_t a, v128_t b) { return wasm_i8x16_min(a, b); }

// CHECK-LABEL: test_u8x16_min:
// CHECK: i8x16.min_u{{$}}
v128_t test_u8x16_min(v128_t a, v128_t b) { return wasm_u8x16_min(a, b); }

// CHECK-LABEL: test_i8x16_max:
// CHECK: i8x16.max_s{{$}}
v128_t test_i8x16_max(v128_t a, v128_t b) { return wasm_i8x16_max(a, b); }

// CHECK-LABEL: test_u8x16_max:
// CHECK: i8x16.max_u{{$}}
v128_t test_u8x16_max(v128_t a, v128_t b) { return wasm_u8x16_max(a, b); }

// CHECK-LABEL: test_u8x16_avgr:
// CHECK: i8x16.avgr_u{{$}}
v128_t test_u8x16_avgr(v128_t a, v128_t b) { return wasm_u8x16_avgr(a, b); }

// CHECK-LABEL: test_i16x8_abs:
// CHECK: i16x8.abs{{$}}
v128_t test_i16x8_abs(v128_t a) { return wasm_i16x8_abs(a); }
````
- **L649 EN**: Blank line separating nearby declarations or logic.
  **L649 CN**: 空行，用于分隔相邻声明或逻辑。
- **L650 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i8x16_min:`.
  **L650 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i8x16_min:`。
- **L651 EN**: Comment documents nearby intent or constraints: `CHECK: i8x16.min_s{{$}}`.
  **L651 CN**: 注释说明附近代码的意图或约束：`CHECK: i8x16.min_s{{$}}`。
- **L652 EN**: Starts a function or method definition for `test_i8x16_min`.
  **L652 CN**: 开始定义函数或方法 `test_i8x16_min`。
- **L653 EN**: Blank line separating nearby declarations or logic.
  **L653 CN**: 空行，用于分隔相邻声明或逻辑。
- **L654 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_u8x16_min:`.
  **L654 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_u8x16_min:`。
- **L655 EN**: Comment documents nearby intent or constraints: `CHECK: i8x16.min_u{{$}}`.
  **L655 CN**: 注释说明附近代码的意图或约束：`CHECK: i8x16.min_u{{$}}`。
- **L656 EN**: Starts a function or method definition for `test_u8x16_min`.
  **L656 CN**: 开始定义函数或方法 `test_u8x16_min`。
- **L657 EN**: Blank line separating nearby declarations or logic.
  **L657 CN**: 空行，用于分隔相邻声明或逻辑。
- **L658 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i8x16_max:`.
  **L658 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i8x16_max:`。
- **L659 EN**: Comment documents nearby intent or constraints: `CHECK: i8x16.max_s{{$}}`.
  **L659 CN**: 注释说明附近代码的意图或约束：`CHECK: i8x16.max_s{{$}}`。
- **L660 EN**: Starts a function or method definition for `test_i8x16_max`.
  **L660 CN**: 开始定义函数或方法 `test_i8x16_max`。
- **L661 EN**: Blank line separating nearby declarations or logic.
  **L661 CN**: 空行，用于分隔相邻声明或逻辑。
- **L662 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_u8x16_max:`.
  **L662 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_u8x16_max:`。
- **L663 EN**: Comment documents nearby intent or constraints: `CHECK: i8x16.max_u{{$}}`.
  **L663 CN**: 注释说明附近代码的意图或约束：`CHECK: i8x16.max_u{{$}}`。
- **L664 EN**: Starts a function or method definition for `test_u8x16_max`.
  **L664 CN**: 开始定义函数或方法 `test_u8x16_max`。
- **L665 EN**: Blank line separating nearby declarations or logic.
  **L665 CN**: 空行，用于分隔相邻声明或逻辑。
- **L666 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_u8x16_avgr:`.
  **L666 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_u8x16_avgr:`。
- **L667 EN**: Comment documents nearby intent or constraints: `CHECK: i8x16.avgr_u{{$}}`.
  **L667 CN**: 注释说明附近代码的意图或约束：`CHECK: i8x16.avgr_u{{$}}`。
- **L668 EN**: Starts a function or method definition for `test_u8x16_avgr`.
  **L668 CN**: 开始定义函数或方法 `test_u8x16_avgr`。
- **L669 EN**: Blank line separating nearby declarations or logic.
  **L669 CN**: 空行，用于分隔相邻声明或逻辑。
- **L670 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i16x8_abs:`.
  **L670 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i16x8_abs:`。
- **L671 EN**: Comment documents nearby intent or constraints: `CHECK: i16x8.abs{{$}}`.
  **L671 CN**: 注释说明附近代码的意图或约束：`CHECK: i16x8.abs{{$}}`。
- **L672 EN**: Starts a function or method definition for `test_i16x8_abs`.
  **L672 CN**: 开始定义函数或方法 `test_i16x8_abs`。

### Lines 673-696

````c

// CHECK-LABEL: test_i16x8_neg:
// CHECK: i16x8.neg{{$}}
v128_t test_i16x8_neg(v128_t a) { return wasm_i16x8_neg(a); }

// CHECK-LABEL: test_i16x8_all_true:
// CHECK: i16x8.all_true{{$}}
bool test_i16x8_all_true(v128_t a) { return wasm_i16x8_all_true(a); }

// CHECK-LABEL: test_i16x8_bitmask:
// CHECK: i16x8.bitmask{{$}}
int32_t test_i16x8_bitmask(v128_t a) { return wasm_i16x8_bitmask(a); }

// CHECK-LABEL: test_i16x8_shl:
// CHECK: i16x8.shl{{$}}
v128_t test_i16x8_shl(v128_t a, int32_t b) { return wasm_i16x8_shl(a, b); }

// CHECK-LABEL: test_i16x8_shr:
// CHECK: i16x8.shr_s{{$}}
v128_t test_i16x8_shr(v128_t a, int32_t b) { return wasm_i16x8_shr(a, b); }

// CHECK-LABEL: test_u16x8_shr:
// CHECK: i16x8.shr_u{{$}}
v128_t test_u16x8_shr(v128_t a, int32_t b) { return wasm_u16x8_shr(a, b); }
````
- **L673 EN**: Blank line separating nearby declarations or logic.
  **L673 CN**: 空行，用于分隔相邻声明或逻辑。
- **L674 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i16x8_neg:`.
  **L674 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i16x8_neg:`。
- **L675 EN**: Comment documents nearby intent or constraints: `CHECK: i16x8.neg{{$}}`.
  **L675 CN**: 注释说明附近代码的意图或约束：`CHECK: i16x8.neg{{$}}`。
- **L676 EN**: Starts a function or method definition for `test_i16x8_neg`.
  **L676 CN**: 开始定义函数或方法 `test_i16x8_neg`。
- **L677 EN**: Blank line separating nearby declarations or logic.
  **L677 CN**: 空行，用于分隔相邻声明或逻辑。
- **L678 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i16x8_all_true:`.
  **L678 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i16x8_all_true:`。
- **L679 EN**: Comment documents nearby intent or constraints: `CHECK: i16x8.all_true{{$}}`.
  **L679 CN**: 注释说明附近代码的意图或约束：`CHECK: i16x8.all_true{{$}}`。
- **L680 EN**: Starts a function or method definition for `test_i16x8_all_true`.
  **L680 CN**: 开始定义函数或方法 `test_i16x8_all_true`。
- **L681 EN**: Blank line separating nearby declarations or logic.
  **L681 CN**: 空行，用于分隔相邻声明或逻辑。
- **L682 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i16x8_bitmask:`.
  **L682 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i16x8_bitmask:`。
- **L683 EN**: Comment documents nearby intent or constraints: `CHECK: i16x8.bitmask{{$}}`.
  **L683 CN**: 注释说明附近代码的意图或约束：`CHECK: i16x8.bitmask{{$}}`。
- **L684 EN**: Starts a function or method definition for `test_i16x8_bitmask`.
  **L684 CN**: 开始定义函数或方法 `test_i16x8_bitmask`。
- **L685 EN**: Blank line separating nearby declarations or logic.
  **L685 CN**: 空行，用于分隔相邻声明或逻辑。
- **L686 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i16x8_shl:`.
  **L686 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i16x8_shl:`。
- **L687 EN**: Comment documents nearby intent or constraints: `CHECK: i16x8.shl{{$}}`.
  **L687 CN**: 注释说明附近代码的意图或约束：`CHECK: i16x8.shl{{$}}`。
- **L688 EN**: Starts a function or method definition for `test_i16x8_shl`.
  **L688 CN**: 开始定义函数或方法 `test_i16x8_shl`。
- **L689 EN**: Blank line separating nearby declarations or logic.
  **L689 CN**: 空行，用于分隔相邻声明或逻辑。
- **L690 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i16x8_shr:`.
  **L690 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i16x8_shr:`。
- **L691 EN**: Comment documents nearby intent or constraints: `CHECK: i16x8.shr_s{{$}}`.
  **L691 CN**: 注释说明附近代码的意图或约束：`CHECK: i16x8.shr_s{{$}}`。
- **L692 EN**: Starts a function or method definition for `test_i16x8_shr`.
  **L692 CN**: 开始定义函数或方法 `test_i16x8_shr`。
- **L693 EN**: Blank line separating nearby declarations or logic.
  **L693 CN**: 空行，用于分隔相邻声明或逻辑。
- **L694 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_u16x8_shr:`.
  **L694 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_u16x8_shr:`。
- **L695 EN**: Comment documents nearby intent or constraints: `CHECK: i16x8.shr_u{{$}}`.
  **L695 CN**: 注释说明附近代码的意图或约束：`CHECK: i16x8.shr_u{{$}}`。
- **L696 EN**: Starts a function or method definition for `test_u16x8_shr`.
  **L696 CN**: 开始定义函数或方法 `test_u16x8_shr`。

### Lines 697-720

````c

// CHECK-LABEL: test_i16x8_add:
// CHECK: i16x8.add{{$}}
v128_t test_i16x8_add(v128_t a, v128_t b) { return wasm_i16x8_add(a, b); }

// CHECK-LABEL: test_i16x8_add_sat:
// CHECK: i16x8.add_sat_s{{$}}
v128_t test_i16x8_add_sat(v128_t a, v128_t b) {
  return wasm_i16x8_add_sat(a, b);
}

// CHECK-LABEL: test_u16x8_add_sat:
// CHECK: i16x8.add_sat_u{{$}}
v128_t test_u16x8_add_sat(v128_t a, v128_t b) {
  return wasm_u16x8_add_sat(a, b);
}

// CHECK-LABEL: test_i16x8_sub:
// CHECK: i16x8.sub{{$}}
v128_t test_i16x8_sub(v128_t a, v128_t b) { return wasm_i16x8_sub(a, b); }

// CHECK-LABEL: test_i16x8_sub_sat:
// CHECK: i16x8.sub_sat_s{{$}}
v128_t test_i16x8_sub_sat(v128_t a, v128_t b) {
````
- **L697 EN**: Blank line separating nearby declarations or logic.
  **L697 CN**: 空行，用于分隔相邻声明或逻辑。
- **L698 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i16x8_add:`.
  **L698 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i16x8_add:`。
- **L699 EN**: Comment documents nearby intent or constraints: `CHECK: i16x8.add{{$}}`.
  **L699 CN**: 注释说明附近代码的意图或约束：`CHECK: i16x8.add{{$}}`。
- **L700 EN**: Starts a function or method definition for `test_i16x8_add`.
  **L700 CN**: 开始定义函数或方法 `test_i16x8_add`。
- **L701 EN**: Blank line separating nearby declarations or logic.
  **L701 CN**: 空行，用于分隔相邻声明或逻辑。
- **L702 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i16x8_add_sat:`.
  **L702 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i16x8_add_sat:`。
- **L703 EN**: Comment documents nearby intent or constraints: `CHECK: i16x8.add_sat_s{{$}}`.
  **L703 CN**: 注释说明附近代码的意图或约束：`CHECK: i16x8.add_sat_s{{$}}`。
- **L704 EN**: Uses compile-time type-trait machinery to inspect or transform types.
  **L704 CN**: 使用编译期类型萃取机制来检查或变换类型。
- **L705 EN**: Returns from the current function with `wasm_i16x8_add_sat(a, b)`.
  **L705 CN**: 以 `wasm_i16x8_add_sat(a, b)` 从当前函数返回。
- **L706 EN**: Closes the current lexical scope or compound statement.
  **L706 CN**: 结束当前词法作用域或复合语句块。
- **L707 EN**: Blank line separating nearby declarations or logic.
  **L707 CN**: 空行，用于分隔相邻声明或逻辑。
- **L708 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_u16x8_add_sat:`.
  **L708 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_u16x8_add_sat:`。
- **L709 EN**: Comment documents nearby intent or constraints: `CHECK: i16x8.add_sat_u{{$}}`.
  **L709 CN**: 注释说明附近代码的意图或约束：`CHECK: i16x8.add_sat_u{{$}}`。
- **L710 EN**: Uses compile-time type-trait machinery to inspect or transform types.
  **L710 CN**: 使用编译期类型萃取机制来检查或变换类型。
- **L711 EN**: Returns from the current function with `wasm_u16x8_add_sat(a, b)`.
  **L711 CN**: 以 `wasm_u16x8_add_sat(a, b)` 从当前函数返回。
- **L712 EN**: Closes the current lexical scope or compound statement.
  **L712 CN**: 结束当前词法作用域或复合语句块。
- **L713 EN**: Blank line separating nearby declarations or logic.
  **L713 CN**: 空行，用于分隔相邻声明或逻辑。
- **L714 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i16x8_sub:`.
  **L714 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i16x8_sub:`。
- **L715 EN**: Comment documents nearby intent or constraints: `CHECK: i16x8.sub{{$}}`.
  **L715 CN**: 注释说明附近代码的意图或约束：`CHECK: i16x8.sub{{$}}`。
- **L716 EN**: Starts a function or method definition for `test_i16x8_sub`.
  **L716 CN**: 开始定义函数或方法 `test_i16x8_sub`。
- **L717 EN**: Blank line separating nearby declarations or logic.
  **L717 CN**: 空行，用于分隔相邻声明或逻辑。
- **L718 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i16x8_sub_sat:`.
  **L718 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i16x8_sub_sat:`。
- **L719 EN**: Comment documents nearby intent or constraints: `CHECK: i16x8.sub_sat_s{{$}}`.
  **L719 CN**: 注释说明附近代码的意图或约束：`CHECK: i16x8.sub_sat_s{{$}}`。
- **L720 EN**: Starts a function or method definition for `test_i16x8_sub_sat`.
  **L720 CN**: 开始定义函数或方法 `test_i16x8_sub_sat`。

### Lines 721-744

````c
  return wasm_i16x8_sub_sat(a, b);
}

// CHECK-LABEL: test_u16x8_sub_sat:
// CHECK: i16x8.sub_sat_u{{$}}
v128_t test_u16x8_sub_sat(v128_t a, v128_t b) {
  return wasm_u16x8_sub_sat(a, b);
}

// CHECK-LABEL: test_i16x8_mul:
// CHECK: i16x8.mul{{$}}
v128_t test_i16x8_mul(v128_t a, v128_t b) { return wasm_i16x8_mul(a, b); }

// CHECK-LABEL: test_i16x8_min:
// CHECK: i16x8.min_s{{$}}
v128_t test_i16x8_min(v128_t a, v128_t b) { return wasm_i16x8_min(a, b); }

// CHECK-LABEL: test_u16x8_min:
// CHECK: i16x8.min_u{{$}}
v128_t test_u16x8_min(v128_t a, v128_t b) { return wasm_u16x8_min(a, b); }

// CHECK-LABEL: test_i16x8_max:
// CHECK: i16x8.max_s{{$}}
v128_t test_i16x8_max(v128_t a, v128_t b) { return wasm_i16x8_max(a, b); }
````
- **L721 EN**: Returns from the current function with `wasm_i16x8_sub_sat(a, b)`.
  **L721 CN**: 以 `wasm_i16x8_sub_sat(a, b)` 从当前函数返回。
- **L722 EN**: Closes the current lexical scope or compound statement.
  **L722 CN**: 结束当前词法作用域或复合语句块。
- **L723 EN**: Blank line separating nearby declarations or logic.
  **L723 CN**: 空行，用于分隔相邻声明或逻辑。
- **L724 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_u16x8_sub_sat:`.
  **L724 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_u16x8_sub_sat:`。
- **L725 EN**: Comment documents nearby intent or constraints: `CHECK: i16x8.sub_sat_u{{$}}`.
  **L725 CN**: 注释说明附近代码的意图或约束：`CHECK: i16x8.sub_sat_u{{$}}`。
- **L726 EN**: Starts a function or method definition for `test_u16x8_sub_sat`.
  **L726 CN**: 开始定义函数或方法 `test_u16x8_sub_sat`。
- **L727 EN**: Returns from the current function with `wasm_u16x8_sub_sat(a, b)`.
  **L727 CN**: 以 `wasm_u16x8_sub_sat(a, b)` 从当前函数返回。
- **L728 EN**: Closes the current lexical scope or compound statement.
  **L728 CN**: 结束当前词法作用域或复合语句块。
- **L729 EN**: Blank line separating nearby declarations or logic.
  **L729 CN**: 空行，用于分隔相邻声明或逻辑。
- **L730 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i16x8_mul:`.
  **L730 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i16x8_mul:`。
- **L731 EN**: Comment documents nearby intent or constraints: `CHECK: i16x8.mul{{$}}`.
  **L731 CN**: 注释说明附近代码的意图或约束：`CHECK: i16x8.mul{{$}}`。
- **L732 EN**: Starts a function or method definition for `test_i16x8_mul`.
  **L732 CN**: 开始定义函数或方法 `test_i16x8_mul`。
- **L733 EN**: Blank line separating nearby declarations or logic.
  **L733 CN**: 空行，用于分隔相邻声明或逻辑。
- **L734 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i16x8_min:`.
  **L734 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i16x8_min:`。
- **L735 EN**: Comment documents nearby intent or constraints: `CHECK: i16x8.min_s{{$}}`.
  **L735 CN**: 注释说明附近代码的意图或约束：`CHECK: i16x8.min_s{{$}}`。
- **L736 EN**: Starts a function or method definition for `test_i16x8_min`.
  **L736 CN**: 开始定义函数或方法 `test_i16x8_min`。
- **L737 EN**: Blank line separating nearby declarations or logic.
  **L737 CN**: 空行，用于分隔相邻声明或逻辑。
- **L738 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_u16x8_min:`.
  **L738 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_u16x8_min:`。
- **L739 EN**: Comment documents nearby intent or constraints: `CHECK: i16x8.min_u{{$}}`.
  **L739 CN**: 注释说明附近代码的意图或约束：`CHECK: i16x8.min_u{{$}}`。
- **L740 EN**: Starts a function or method definition for `test_u16x8_min`.
  **L740 CN**: 开始定义函数或方法 `test_u16x8_min`。
- **L741 EN**: Blank line separating nearby declarations or logic.
  **L741 CN**: 空行，用于分隔相邻声明或逻辑。
- **L742 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i16x8_max:`.
  **L742 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i16x8_max:`。
- **L743 EN**: Comment documents nearby intent or constraints: `CHECK: i16x8.max_s{{$}}`.
  **L743 CN**: 注释说明附近代码的意图或约束：`CHECK: i16x8.max_s{{$}}`。
- **L744 EN**: Starts a function or method definition for `test_i16x8_max`.
  **L744 CN**: 开始定义函数或方法 `test_i16x8_max`。

### Lines 745-768

````c

// CHECK-LABEL: test_u16x8_max:
// CHECK: i16x8.max_u{{$}}
v128_t test_u16x8_max(v128_t a, v128_t b) { return wasm_u16x8_max(a, b); }

// CHECK-LABEL: test_u16x8_avgr:
// CHECK: i16x8.avgr_u{{$}}
v128_t test_u16x8_avgr(v128_t a, v128_t b) { return wasm_u16x8_avgr(a, b); }

// CHECK-LABEL: test_i32x4_abs:
// CHECK: i32x4.abs{{$}}
v128_t test_i32x4_abs(v128_t a) { return wasm_i32x4_abs(a); }

// CHECK-LABEL: test_i32x4_neg:
// CHECK: i32x4.neg{{$}}
v128_t test_i32x4_neg(v128_t a) { return wasm_i32x4_neg(a); }

// CHECK-LABEL: test_i32x4_all_true:
// CHECK: i32x4.all_true{{$}}
bool test_i32x4_all_true(v128_t a) { return wasm_i32x4_all_true(a); }

// CHECK-LABEL: test_i32x4_bitmask:
// CHECK: i32x4.bitmask{{$}}
int32_t test_i32x4_bitmask(v128_t a) { return wasm_i32x4_bitmask(a); }
````
- **L745 EN**: Blank line separating nearby declarations or logic.
  **L745 CN**: 空行，用于分隔相邻声明或逻辑。
- **L746 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_u16x8_max:`.
  **L746 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_u16x8_max:`。
- **L747 EN**: Comment documents nearby intent or constraints: `CHECK: i16x8.max_u{{$}}`.
  **L747 CN**: 注释说明附近代码的意图或约束：`CHECK: i16x8.max_u{{$}}`。
- **L748 EN**: Starts a function or method definition for `test_u16x8_max`.
  **L748 CN**: 开始定义函数或方法 `test_u16x8_max`。
- **L749 EN**: Blank line separating nearby declarations or logic.
  **L749 CN**: 空行，用于分隔相邻声明或逻辑。
- **L750 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_u16x8_avgr:`.
  **L750 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_u16x8_avgr:`。
- **L751 EN**: Comment documents nearby intent or constraints: `CHECK: i16x8.avgr_u{{$}}`.
  **L751 CN**: 注释说明附近代码的意图或约束：`CHECK: i16x8.avgr_u{{$}}`。
- **L752 EN**: Starts a function or method definition for `test_u16x8_avgr`.
  **L752 CN**: 开始定义函数或方法 `test_u16x8_avgr`。
- **L753 EN**: Blank line separating nearby declarations or logic.
  **L753 CN**: 空行，用于分隔相邻声明或逻辑。
- **L754 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i32x4_abs:`.
  **L754 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i32x4_abs:`。
- **L755 EN**: Comment documents nearby intent or constraints: `CHECK: i32x4.abs{{$}}`.
  **L755 CN**: 注释说明附近代码的意图或约束：`CHECK: i32x4.abs{{$}}`。
- **L756 EN**: Starts a function or method definition for `test_i32x4_abs`.
  **L756 CN**: 开始定义函数或方法 `test_i32x4_abs`。
- **L757 EN**: Blank line separating nearby declarations or logic.
  **L757 CN**: 空行，用于分隔相邻声明或逻辑。
- **L758 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i32x4_neg:`.
  **L758 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i32x4_neg:`。
- **L759 EN**: Comment documents nearby intent or constraints: `CHECK: i32x4.neg{{$}}`.
  **L759 CN**: 注释说明附近代码的意图或约束：`CHECK: i32x4.neg{{$}}`。
- **L760 EN**: Starts a function or method definition for `test_i32x4_neg`.
  **L760 CN**: 开始定义函数或方法 `test_i32x4_neg`。
- **L761 EN**: Blank line separating nearby declarations or logic.
  **L761 CN**: 空行，用于分隔相邻声明或逻辑。
- **L762 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i32x4_all_true:`.
  **L762 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i32x4_all_true:`。
- **L763 EN**: Comment documents nearby intent or constraints: `CHECK: i32x4.all_true{{$}}`.
  **L763 CN**: 注释说明附近代码的意图或约束：`CHECK: i32x4.all_true{{$}}`。
- **L764 EN**: Starts a function or method definition for `test_i32x4_all_true`.
  **L764 CN**: 开始定义函数或方法 `test_i32x4_all_true`。
- **L765 EN**: Blank line separating nearby declarations or logic.
  **L765 CN**: 空行，用于分隔相邻声明或逻辑。
- **L766 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i32x4_bitmask:`.
  **L766 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i32x4_bitmask:`。
- **L767 EN**: Comment documents nearby intent or constraints: `CHECK: i32x4.bitmask{{$}}`.
  **L767 CN**: 注释说明附近代码的意图或约束：`CHECK: i32x4.bitmask{{$}}`。
- **L768 EN**: Starts a function or method definition for `test_i32x4_bitmask`.
  **L768 CN**: 开始定义函数或方法 `test_i32x4_bitmask`。

### Lines 769-792

````c

// CHECK-LABEL: test_i32x4_shl:
// CHECK: i32x4.shl{{$}}
v128_t test_i32x4_shl(v128_t a, int32_t b) { return wasm_i32x4_shl(a, b); }

// CHECK-LABEL: test_i32x4_shr:
// CHECK: i32x4.shr_s{{$}}
v128_t test_i32x4_shr(v128_t a, int32_t b) { return wasm_i32x4_shr(a, b); }

// CHECK-LABEL: test_u32x4_shr:
// CHECK: i32x4.shr_u{{$}}
v128_t test_u32x4_shr(v128_t a, int32_t b) { return wasm_u32x4_shr(a, b); }

// CHECK-LABEL: test_i32x4_add:
// CHECK: i32x4.add{{$}}
v128_t test_i32x4_add(v128_t a, v128_t b) { return wasm_i32x4_add(a, b); }

// CHECK-LABEL: test_i32x4_sub:
// CHECK: i32x4.sub{{$}}
v128_t test_i32x4_sub(v128_t a, v128_t b) { return wasm_i32x4_sub(a, b); }

// CHECK-LABEL: test_i32x4_mul:
// CHECK: i32x4.mul{{$}}
v128_t test_i32x4_mul(v128_t a, v128_t b) { return wasm_i32x4_mul(a, b); }
````
- **L769 EN**: Blank line separating nearby declarations or logic.
  **L769 CN**: 空行，用于分隔相邻声明或逻辑。
- **L770 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i32x4_shl:`.
  **L770 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i32x4_shl:`。
- **L771 EN**: Comment documents nearby intent or constraints: `CHECK: i32x4.shl{{$}}`.
  **L771 CN**: 注释说明附近代码的意图或约束：`CHECK: i32x4.shl{{$}}`。
- **L772 EN**: Starts a function or method definition for `test_i32x4_shl`.
  **L772 CN**: 开始定义函数或方法 `test_i32x4_shl`。
- **L773 EN**: Blank line separating nearby declarations or logic.
  **L773 CN**: 空行，用于分隔相邻声明或逻辑。
- **L774 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i32x4_shr:`.
  **L774 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i32x4_shr:`。
- **L775 EN**: Comment documents nearby intent or constraints: `CHECK: i32x4.shr_s{{$}}`.
  **L775 CN**: 注释说明附近代码的意图或约束：`CHECK: i32x4.shr_s{{$}}`。
- **L776 EN**: Starts a function or method definition for `test_i32x4_shr`.
  **L776 CN**: 开始定义函数或方法 `test_i32x4_shr`。
- **L777 EN**: Blank line separating nearby declarations or logic.
  **L777 CN**: 空行，用于分隔相邻声明或逻辑。
- **L778 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_u32x4_shr:`.
  **L778 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_u32x4_shr:`。
- **L779 EN**: Comment documents nearby intent or constraints: `CHECK: i32x4.shr_u{{$}}`.
  **L779 CN**: 注释说明附近代码的意图或约束：`CHECK: i32x4.shr_u{{$}}`。
- **L780 EN**: Starts a function or method definition for `test_u32x4_shr`.
  **L780 CN**: 开始定义函数或方法 `test_u32x4_shr`。
- **L781 EN**: Blank line separating nearby declarations or logic.
  **L781 CN**: 空行，用于分隔相邻声明或逻辑。
- **L782 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i32x4_add:`.
  **L782 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i32x4_add:`。
- **L783 EN**: Comment documents nearby intent or constraints: `CHECK: i32x4.add{{$}}`.
  **L783 CN**: 注释说明附近代码的意图或约束：`CHECK: i32x4.add{{$}}`。
- **L784 EN**: Starts a function or method definition for `test_i32x4_add`.
  **L784 CN**: 开始定义函数或方法 `test_i32x4_add`。
- **L785 EN**: Blank line separating nearby declarations or logic.
  **L785 CN**: 空行，用于分隔相邻声明或逻辑。
- **L786 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i32x4_sub:`.
  **L786 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i32x4_sub:`。
- **L787 EN**: Comment documents nearby intent or constraints: `CHECK: i32x4.sub{{$}}`.
  **L787 CN**: 注释说明附近代码的意图或约束：`CHECK: i32x4.sub{{$}}`。
- **L788 EN**: Starts a function or method definition for `test_i32x4_sub`.
  **L788 CN**: 开始定义函数或方法 `test_i32x4_sub`。
- **L789 EN**: Blank line separating nearby declarations or logic.
  **L789 CN**: 空行，用于分隔相邻声明或逻辑。
- **L790 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i32x4_mul:`.
  **L790 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i32x4_mul:`。
- **L791 EN**: Comment documents nearby intent or constraints: `CHECK: i32x4.mul{{$}}`.
  **L791 CN**: 注释说明附近代码的意图或约束：`CHECK: i32x4.mul{{$}}`。
- **L792 EN**: Starts a function or method definition for `test_i32x4_mul`.
  **L792 CN**: 开始定义函数或方法 `test_i32x4_mul`。

### Lines 793-816

````c

// CHECK-LABEL: test_i32x4_min:
// CHECK: i32x4.min_s{{$}}
v128_t test_i32x4_min(v128_t a, v128_t b) { return wasm_i32x4_min(a, b); }

// CHECK-LABEL: test_u32x4_min:
// CHECK: i32x4.min_u{{$}}
v128_t test_u32x4_min(v128_t a, v128_t b) { return wasm_u32x4_min(a, b); }

// CHECK-LABEL: test_i32x4_max:
// CHECK: i32x4.max_s{{$}}
v128_t test_i32x4_max(v128_t a, v128_t b) { return wasm_i32x4_max(a, b); }

// CHECK-LABEL: test_u32x4_max:
// CHECK: i32x4.max_u{{$}}
v128_t test_u32x4_max(v128_t a, v128_t b) { return wasm_u32x4_max(a, b); }

// CHECK-LABEL: test_i32x4_dot_i16x8:
// CHECK: i32x4.dot_i16x8_s{{$}}
v128_t test_i32x4_dot_i16x8(v128_t a, v128_t b) {
  return wasm_i32x4_dot_i16x8(a, b);
}

// CHECK-LABEL: test_i64x2_abs:
````
- **L793 EN**: Blank line separating nearby declarations or logic.
  **L793 CN**: 空行，用于分隔相邻声明或逻辑。
- **L794 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i32x4_min:`.
  **L794 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i32x4_min:`。
- **L795 EN**: Comment documents nearby intent or constraints: `CHECK: i32x4.min_s{{$}}`.
  **L795 CN**: 注释说明附近代码的意图或约束：`CHECK: i32x4.min_s{{$}}`。
- **L796 EN**: Starts a function or method definition for `test_i32x4_min`.
  **L796 CN**: 开始定义函数或方法 `test_i32x4_min`。
- **L797 EN**: Blank line separating nearby declarations or logic.
  **L797 CN**: 空行，用于分隔相邻声明或逻辑。
- **L798 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_u32x4_min:`.
  **L798 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_u32x4_min:`。
- **L799 EN**: Comment documents nearby intent or constraints: `CHECK: i32x4.min_u{{$}}`.
  **L799 CN**: 注释说明附近代码的意图或约束：`CHECK: i32x4.min_u{{$}}`。
- **L800 EN**: Starts a function or method definition for `test_u32x4_min`.
  **L800 CN**: 开始定义函数或方法 `test_u32x4_min`。
- **L801 EN**: Blank line separating nearby declarations or logic.
  **L801 CN**: 空行，用于分隔相邻声明或逻辑。
- **L802 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i32x4_max:`.
  **L802 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i32x4_max:`。
- **L803 EN**: Comment documents nearby intent or constraints: `CHECK: i32x4.max_s{{$}}`.
  **L803 CN**: 注释说明附近代码的意图或约束：`CHECK: i32x4.max_s{{$}}`。
- **L804 EN**: Starts a function or method definition for `test_i32x4_max`.
  **L804 CN**: 开始定义函数或方法 `test_i32x4_max`。
- **L805 EN**: Blank line separating nearby declarations or logic.
  **L805 CN**: 空行，用于分隔相邻声明或逻辑。
- **L806 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_u32x4_max:`.
  **L806 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_u32x4_max:`。
- **L807 EN**: Comment documents nearby intent or constraints: `CHECK: i32x4.max_u{{$}}`.
  **L807 CN**: 注释说明附近代码的意图或约束：`CHECK: i32x4.max_u{{$}}`。
- **L808 EN**: Starts a function or method definition for `test_u32x4_max`.
  **L808 CN**: 开始定义函数或方法 `test_u32x4_max`。
- **L809 EN**: Blank line separating nearby declarations or logic.
  **L809 CN**: 空行，用于分隔相邻声明或逻辑。
- **L810 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i32x4_dot_i16x8:`.
  **L810 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i32x4_dot_i16x8:`。
- **L811 EN**: Comment documents nearby intent or constraints: `CHECK: i32x4.dot_i16x8_s{{$}}`.
  **L811 CN**: 注释说明附近代码的意图或约束：`CHECK: i32x4.dot_i16x8_s{{$}}`。
- **L812 EN**: Starts a function or method definition for `test_i32x4_dot_i16x8`.
  **L812 CN**: 开始定义函数或方法 `test_i32x4_dot_i16x8`。
- **L813 EN**: Returns from the current function with `wasm_i32x4_dot_i16x8(a, b)`.
  **L813 CN**: 以 `wasm_i32x4_dot_i16x8(a, b)` 从当前函数返回。
- **L814 EN**: Closes the current lexical scope or compound statement.
  **L814 CN**: 结束当前词法作用域或复合语句块。
- **L815 EN**: Blank line separating nearby declarations or logic.
  **L815 CN**: 空行，用于分隔相邻声明或逻辑。
- **L816 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i64x2_abs:`.
  **L816 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i64x2_abs:`。

### Lines 817-840

````c
// CHECK: i64x2.abs{{$}}
v128_t test_i64x2_abs(v128_t a) { return wasm_i64x2_abs(a); }

// CHECK-LABEL: test_i64x2_neg:
// CHECK: i64x2.neg{{$}}
v128_t test_i64x2_neg(v128_t a) { return wasm_i64x2_neg(a); }

// CHECK-LABEL: test_i64x2_all_true:
// CHECK: i64x2.all_true{{$}}
bool test_i64x2_all_true(v128_t a) { return wasm_i64x2_all_true(a); }

// CHECK-LABEL: test_i64x2_bitmask:
// CHECK: i64x2.bitmask{{$}}
int32_t test_i64x2_bitmask(v128_t a) { return wasm_i64x2_bitmask(a); }

// CHECK-LABEL: test_i64x2_shl:
// CHECK: i64x2.shl{{$}}
v128_t test_i64x2_shl(v128_t a, int32_t b) { return wasm_i64x2_shl(a, b); }

// CHECK-LABEL: test_i64x2_shr:
// CHECK: i64x2.shr_s{{$}}
v128_t test_i64x2_shr(v128_t a, int32_t b) { return wasm_i64x2_shr(a, b); }

// CHECK-LABEL: test_u64x2_shr:
````
- **L817 EN**: Comment documents nearby intent or constraints: `CHECK: i64x2.abs{{$}}`.
  **L817 CN**: 注释说明附近代码的意图或约束：`CHECK: i64x2.abs{{$}}`。
- **L818 EN**: Starts a function or method definition for `test_i64x2_abs`.
  **L818 CN**: 开始定义函数或方法 `test_i64x2_abs`。
- **L819 EN**: Blank line separating nearby declarations or logic.
  **L819 CN**: 空行，用于分隔相邻声明或逻辑。
- **L820 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i64x2_neg:`.
  **L820 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i64x2_neg:`。
- **L821 EN**: Comment documents nearby intent or constraints: `CHECK: i64x2.neg{{$}}`.
  **L821 CN**: 注释说明附近代码的意图或约束：`CHECK: i64x2.neg{{$}}`。
- **L822 EN**: Starts a function or method definition for `test_i64x2_neg`.
  **L822 CN**: 开始定义函数或方法 `test_i64x2_neg`。
- **L823 EN**: Blank line separating nearby declarations or logic.
  **L823 CN**: 空行，用于分隔相邻声明或逻辑。
- **L824 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i64x2_all_true:`.
  **L824 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i64x2_all_true:`。
- **L825 EN**: Comment documents nearby intent or constraints: `CHECK: i64x2.all_true{{$}}`.
  **L825 CN**: 注释说明附近代码的意图或约束：`CHECK: i64x2.all_true{{$}}`。
- **L826 EN**: Starts a function or method definition for `test_i64x2_all_true`.
  **L826 CN**: 开始定义函数或方法 `test_i64x2_all_true`。
- **L827 EN**: Blank line separating nearby declarations or logic.
  **L827 CN**: 空行，用于分隔相邻声明或逻辑。
- **L828 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i64x2_bitmask:`.
  **L828 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i64x2_bitmask:`。
- **L829 EN**: Comment documents nearby intent or constraints: `CHECK: i64x2.bitmask{{$}}`.
  **L829 CN**: 注释说明附近代码的意图或约束：`CHECK: i64x2.bitmask{{$}}`。
- **L830 EN**: Starts a function or method definition for `test_i64x2_bitmask`.
  **L830 CN**: 开始定义函数或方法 `test_i64x2_bitmask`。
- **L831 EN**: Blank line separating nearby declarations or logic.
  **L831 CN**: 空行，用于分隔相邻声明或逻辑。
- **L832 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i64x2_shl:`.
  **L832 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i64x2_shl:`。
- **L833 EN**: Comment documents nearby intent or constraints: `CHECK: i64x2.shl{{$}}`.
  **L833 CN**: 注释说明附近代码的意图或约束：`CHECK: i64x2.shl{{$}}`。
- **L834 EN**: Starts a function or method definition for `test_i64x2_shl`.
  **L834 CN**: 开始定义函数或方法 `test_i64x2_shl`。
- **L835 EN**: Blank line separating nearby declarations or logic.
  **L835 CN**: 空行，用于分隔相邻声明或逻辑。
- **L836 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i64x2_shr:`.
  **L836 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i64x2_shr:`。
- **L837 EN**: Comment documents nearby intent or constraints: `CHECK: i64x2.shr_s{{$}}`.
  **L837 CN**: 注释说明附近代码的意图或约束：`CHECK: i64x2.shr_s{{$}}`。
- **L838 EN**: Starts a function or method definition for `test_i64x2_shr`.
  **L838 CN**: 开始定义函数或方法 `test_i64x2_shr`。
- **L839 EN**: Blank line separating nearby declarations or logic.
  **L839 CN**: 空行，用于分隔相邻声明或逻辑。
- **L840 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_u64x2_shr:`.
  **L840 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_u64x2_shr:`。

### Lines 841-864

````c
// CHECK: i64x2.shr_u{{$}}
v128_t test_u64x2_shr(v128_t a, int32_t b) { return wasm_u64x2_shr(a, b); }

// CHECK-LABEL: test_i64x2_add:
// CHECK: i64x2.add{{$}}
v128_t test_i64x2_add(v128_t a, v128_t b) { return wasm_i64x2_add(a, b); }

// CHECK-LABEL: test_i64x2_sub:
// CHECK: i64x2.sub{{$}}
v128_t test_i64x2_sub(v128_t a, v128_t b) { return wasm_i64x2_sub(a, b); }

// CHECK-LABEL: test_i64x2_mul:
// CHECK: i64x2.mul{{$}}
v128_t test_i64x2_mul(v128_t a, v128_t b) { return wasm_i64x2_mul(a, b); }

// CHECK-LABEL: test_f32x4_abs:
// CHECK: f32x4.abs{{$}}
v128_t test_f32x4_abs(v128_t a) { return wasm_f32x4_abs(a); }

// CHECK-LABEL: test_f32x4_neg:
// CHECK: f32x4.neg{{$}}
v128_t test_f32x4_neg(v128_t a) { return wasm_f32x4_neg(a); }

// CHECK-LABEL: test_f32x4_sqrt:
````
- **L841 EN**: Comment documents nearby intent or constraints: `CHECK: i64x2.shr_u{{$}}`.
  **L841 CN**: 注释说明附近代码的意图或约束：`CHECK: i64x2.shr_u{{$}}`。
- **L842 EN**: Starts a function or method definition for `test_u64x2_shr`.
  **L842 CN**: 开始定义函数或方法 `test_u64x2_shr`。
- **L843 EN**: Blank line separating nearby declarations or logic.
  **L843 CN**: 空行，用于分隔相邻声明或逻辑。
- **L844 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i64x2_add:`.
  **L844 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i64x2_add:`。
- **L845 EN**: Comment documents nearby intent or constraints: `CHECK: i64x2.add{{$}}`.
  **L845 CN**: 注释说明附近代码的意图或约束：`CHECK: i64x2.add{{$}}`。
- **L846 EN**: Starts a function or method definition for `test_i64x2_add`.
  **L846 CN**: 开始定义函数或方法 `test_i64x2_add`。
- **L847 EN**: Blank line separating nearby declarations or logic.
  **L847 CN**: 空行，用于分隔相邻声明或逻辑。
- **L848 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i64x2_sub:`.
  **L848 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i64x2_sub:`。
- **L849 EN**: Comment documents nearby intent or constraints: `CHECK: i64x2.sub{{$}}`.
  **L849 CN**: 注释说明附近代码的意图或约束：`CHECK: i64x2.sub{{$}}`。
- **L850 EN**: Starts a function or method definition for `test_i64x2_sub`.
  **L850 CN**: 开始定义函数或方法 `test_i64x2_sub`。
- **L851 EN**: Blank line separating nearby declarations or logic.
  **L851 CN**: 空行，用于分隔相邻声明或逻辑。
- **L852 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i64x2_mul:`.
  **L852 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i64x2_mul:`。
- **L853 EN**: Comment documents nearby intent or constraints: `CHECK: i64x2.mul{{$}}`.
  **L853 CN**: 注释说明附近代码的意图或约束：`CHECK: i64x2.mul{{$}}`。
- **L854 EN**: Starts a function or method definition for `test_i64x2_mul`.
  **L854 CN**: 开始定义函数或方法 `test_i64x2_mul`。
- **L855 EN**: Blank line separating nearby declarations or logic.
  **L855 CN**: 空行，用于分隔相邻声明或逻辑。
- **L856 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f32x4_abs:`.
  **L856 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f32x4_abs:`。
- **L857 EN**: Comment documents nearby intent or constraints: `CHECK: f32x4.abs{{$}}`.
  **L857 CN**: 注释说明附近代码的意图或约束：`CHECK: f32x4.abs{{$}}`。
- **L858 EN**: Starts a function or method definition for `test_f32x4_abs`.
  **L858 CN**: 开始定义函数或方法 `test_f32x4_abs`。
- **L859 EN**: Blank line separating nearby declarations or logic.
  **L859 CN**: 空行，用于分隔相邻声明或逻辑。
- **L860 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f32x4_neg:`.
  **L860 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f32x4_neg:`。
- **L861 EN**: Comment documents nearby intent or constraints: `CHECK: f32x4.neg{{$}}`.
  **L861 CN**: 注释说明附近代码的意图或约束：`CHECK: f32x4.neg{{$}}`。
- **L862 EN**: Starts a function or method definition for `test_f32x4_neg`.
  **L862 CN**: 开始定义函数或方法 `test_f32x4_neg`。
- **L863 EN**: Blank line separating nearby declarations or logic.
  **L863 CN**: 空行，用于分隔相邻声明或逻辑。
- **L864 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f32x4_sqrt:`.
  **L864 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f32x4_sqrt:`。

### Lines 865-888

````c
// CHECK: f32x4.sqrt{{$}}
v128_t test_f32x4_sqrt(v128_t a) { return wasm_f32x4_sqrt(a); }

// CHECK-LABEL: test_f32x4_ceil:
// CHECK: f32x4.ceil{{$}}
v128_t test_f32x4_ceil(v128_t a) { return wasm_f32x4_ceil(a); }

// CHECK-LABEL: test_f32x4_floor:
// CHECK: f32x4.floor{{$}}
v128_t test_f32x4_floor(v128_t a) { return wasm_f32x4_floor(a); }

// CHECK-LABEL: test_f32x4_trunc:
// CHECK: f32x4.trunc{{$}}
v128_t test_f32x4_trunc(v128_t a) { return wasm_f32x4_trunc(a); }

// CHECK-LABEL: test_f32x4_nearest:
// CHECK: f32x4.nearest{{$}}
v128_t test_f32x4_nearest(v128_t a) { return wasm_f32x4_nearest(a); }

// CHECK-LABEL: test_f32x4_add:
// CHECK: f32x4.add{{$}}
v128_t test_f32x4_add(v128_t a, v128_t b) { return wasm_f32x4_add(a, b); }

// CHECK-LABEL: test_f32x4_sub:
````
- **L865 EN**: Comment documents nearby intent or constraints: `CHECK: f32x4.sqrt{{$}}`.
  **L865 CN**: 注释说明附近代码的意图或约束：`CHECK: f32x4.sqrt{{$}}`。
- **L866 EN**: Starts a function or method definition for `test_f32x4_sqrt`.
  **L866 CN**: 开始定义函数或方法 `test_f32x4_sqrt`。
- **L867 EN**: Blank line separating nearby declarations or logic.
  **L867 CN**: 空行，用于分隔相邻声明或逻辑。
- **L868 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f32x4_ceil:`.
  **L868 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f32x4_ceil:`。
- **L869 EN**: Comment documents nearby intent or constraints: `CHECK: f32x4.ceil{{$}}`.
  **L869 CN**: 注释说明附近代码的意图或约束：`CHECK: f32x4.ceil{{$}}`。
- **L870 EN**: Starts a function or method definition for `test_f32x4_ceil`.
  **L870 CN**: 开始定义函数或方法 `test_f32x4_ceil`。
- **L871 EN**: Blank line separating nearby declarations or logic.
  **L871 CN**: 空行，用于分隔相邻声明或逻辑。
- **L872 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f32x4_floor:`.
  **L872 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f32x4_floor:`。
- **L873 EN**: Comment documents nearby intent or constraints: `CHECK: f32x4.floor{{$}}`.
  **L873 CN**: 注释说明附近代码的意图或约束：`CHECK: f32x4.floor{{$}}`。
- **L874 EN**: Starts a function or method definition for `test_f32x4_floor`.
  **L874 CN**: 开始定义函数或方法 `test_f32x4_floor`。
- **L875 EN**: Blank line separating nearby declarations or logic.
  **L875 CN**: 空行，用于分隔相邻声明或逻辑。
- **L876 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f32x4_trunc:`.
  **L876 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f32x4_trunc:`。
- **L877 EN**: Comment documents nearby intent or constraints: `CHECK: f32x4.trunc{{$}}`.
  **L877 CN**: 注释说明附近代码的意图或约束：`CHECK: f32x4.trunc{{$}}`。
- **L878 EN**: Starts a function or method definition for `test_f32x4_trunc`.
  **L878 CN**: 开始定义函数或方法 `test_f32x4_trunc`。
- **L879 EN**: Blank line separating nearby declarations or logic.
  **L879 CN**: 空行，用于分隔相邻声明或逻辑。
- **L880 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f32x4_nearest:`.
  **L880 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f32x4_nearest:`。
- **L881 EN**: Comment documents nearby intent or constraints: `CHECK: f32x4.nearest{{$}}`.
  **L881 CN**: 注释说明附近代码的意图或约束：`CHECK: f32x4.nearest{{$}}`。
- **L882 EN**: Starts a function or method definition for `test_f32x4_nearest`.
  **L882 CN**: 开始定义函数或方法 `test_f32x4_nearest`。
- **L883 EN**: Blank line separating nearby declarations or logic.
  **L883 CN**: 空行，用于分隔相邻声明或逻辑。
- **L884 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f32x4_add:`.
  **L884 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f32x4_add:`。
- **L885 EN**: Comment documents nearby intent or constraints: `CHECK: f32x4.add{{$}}`.
  **L885 CN**: 注释说明附近代码的意图或约束：`CHECK: f32x4.add{{$}}`。
- **L886 EN**: Starts a function or method definition for `test_f32x4_add`.
  **L886 CN**: 开始定义函数或方法 `test_f32x4_add`。
- **L887 EN**: Blank line separating nearby declarations or logic.
  **L887 CN**: 空行，用于分隔相邻声明或逻辑。
- **L888 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f32x4_sub:`.
  **L888 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f32x4_sub:`。

### Lines 889-912

````c
// CHECK: f32x4.sub{{$}}
v128_t test_f32x4_sub(v128_t a, v128_t b) { return wasm_f32x4_sub(a, b); }

// CHECK-LABEL: test_f32x4_mul:
// CHECK: f32x4.mul{{$}}
v128_t test_f32x4_mul(v128_t a, v128_t b) { return wasm_f32x4_mul(a, b); }

// CHECK-LABEL: test_f32x4_div:
// CHECK: f32x4.div{{$}}
v128_t test_f32x4_div(v128_t a, v128_t b) { return wasm_f32x4_div(a, b); }

// CHECK-LABEL: test_f32x4_min:
// CHECK: f32x4.min{{$}}
v128_t test_f32x4_min(v128_t a, v128_t b) { return wasm_f32x4_min(a, b); }

// CHECK-LABEL: test_f32x4_max:
// CHECK: f32x4.max{{$}}
v128_t test_f32x4_max(v128_t a, v128_t b) { return wasm_f32x4_max(a, b); }

// CHECK-LABEL: test_f32x4_pmin:
// CHECK: f32x4.pmin{{$}}
v128_t test_f32x4_pmin(v128_t a, v128_t b) { return wasm_f32x4_pmin(a, b); }

// CHECK-LABEL: test_f32x4_pmax:
````
- **L889 EN**: Comment documents nearby intent or constraints: `CHECK: f32x4.sub{{$}}`.
  **L889 CN**: 注释说明附近代码的意图或约束：`CHECK: f32x4.sub{{$}}`。
- **L890 EN**: Starts a function or method definition for `test_f32x4_sub`.
  **L890 CN**: 开始定义函数或方法 `test_f32x4_sub`。
- **L891 EN**: Blank line separating nearby declarations or logic.
  **L891 CN**: 空行，用于分隔相邻声明或逻辑。
- **L892 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f32x4_mul:`.
  **L892 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f32x4_mul:`。
- **L893 EN**: Comment documents nearby intent or constraints: `CHECK: f32x4.mul{{$}}`.
  **L893 CN**: 注释说明附近代码的意图或约束：`CHECK: f32x4.mul{{$}}`。
- **L894 EN**: Starts a function or method definition for `test_f32x4_mul`.
  **L894 CN**: 开始定义函数或方法 `test_f32x4_mul`。
- **L895 EN**: Blank line separating nearby declarations or logic.
  **L895 CN**: 空行，用于分隔相邻声明或逻辑。
- **L896 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f32x4_div:`.
  **L896 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f32x4_div:`。
- **L897 EN**: Comment documents nearby intent or constraints: `CHECK: f32x4.div{{$}}`.
  **L897 CN**: 注释说明附近代码的意图或约束：`CHECK: f32x4.div{{$}}`。
- **L898 EN**: Starts a function or method definition for `test_f32x4_div`.
  **L898 CN**: 开始定义函数或方法 `test_f32x4_div`。
- **L899 EN**: Blank line separating nearby declarations or logic.
  **L899 CN**: 空行，用于分隔相邻声明或逻辑。
- **L900 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f32x4_min:`.
  **L900 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f32x4_min:`。
- **L901 EN**: Comment documents nearby intent or constraints: `CHECK: f32x4.min{{$}}`.
  **L901 CN**: 注释说明附近代码的意图或约束：`CHECK: f32x4.min{{$}}`。
- **L902 EN**: Starts a function or method definition for `test_f32x4_min`.
  **L902 CN**: 开始定义函数或方法 `test_f32x4_min`。
- **L903 EN**: Blank line separating nearby declarations or logic.
  **L903 CN**: 空行，用于分隔相邻声明或逻辑。
- **L904 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f32x4_max:`.
  **L904 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f32x4_max:`。
- **L905 EN**: Comment documents nearby intent or constraints: `CHECK: f32x4.max{{$}}`.
  **L905 CN**: 注释说明附近代码的意图或约束：`CHECK: f32x4.max{{$}}`。
- **L906 EN**: Starts a function or method definition for `test_f32x4_max`.
  **L906 CN**: 开始定义函数或方法 `test_f32x4_max`。
- **L907 EN**: Blank line separating nearby declarations or logic.
  **L907 CN**: 空行，用于分隔相邻声明或逻辑。
- **L908 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f32x4_pmin:`.
  **L908 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f32x4_pmin:`。
- **L909 EN**: Comment documents nearby intent or constraints: `CHECK: f32x4.pmin{{$}}`.
  **L909 CN**: 注释说明附近代码的意图或约束：`CHECK: f32x4.pmin{{$}}`。
- **L910 EN**: Starts a function or method definition for `test_f32x4_pmin`.
  **L910 CN**: 开始定义函数或方法 `test_f32x4_pmin`。
- **L911 EN**: Blank line separating nearby declarations or logic.
  **L911 CN**: 空行，用于分隔相邻声明或逻辑。
- **L912 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f32x4_pmax:`.
  **L912 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f32x4_pmax:`。

### Lines 913-936

````c
// CHECK: f32x4.pmax{{$}}
v128_t test_f32x4_pmax(v128_t a, v128_t b) { return wasm_f32x4_pmax(a, b); }

// CHECK-LABEL: test_f64x2_abs:
// CHECK: f64x2.abs{{$}}
v128_t test_f64x2_abs(v128_t a) { return wasm_f64x2_abs(a); }

// CHECK-LABEL: test_f64x2_neg:
// CHECK: f64x2.neg{{$}}
v128_t test_f64x2_neg(v128_t a) { return wasm_f64x2_neg(a); }

// CHECK-LABEL: test_f64x2_sqrt:
// CHECK: f64x2.sqrt{{$}}
v128_t test_f64x2_sqrt(v128_t a) { return wasm_f64x2_sqrt(a); }

// CHECK-LABEL: test_f64x2_ceil:
// CHECK: f64x2.ceil{{$}}
v128_t test_f64x2_ceil(v128_t a) { return wasm_f64x2_ceil(a); }

// CHECK-LABEL: test_f64x2_floor:
// CHECK: f64x2.floor{{$}}
v128_t test_f64x2_floor(v128_t a) { return wasm_f64x2_floor(a); }

// CHECK-LABEL: test_f64x2_trunc:
````
- **L913 EN**: Comment documents nearby intent or constraints: `CHECK: f32x4.pmax{{$}}`.
  **L913 CN**: 注释说明附近代码的意图或约束：`CHECK: f32x4.pmax{{$}}`。
- **L914 EN**: Starts a function or method definition for `test_f32x4_pmax`.
  **L914 CN**: 开始定义函数或方法 `test_f32x4_pmax`。
- **L915 EN**: Blank line separating nearby declarations or logic.
  **L915 CN**: 空行，用于分隔相邻声明或逻辑。
- **L916 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f64x2_abs:`.
  **L916 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f64x2_abs:`。
- **L917 EN**: Comment documents nearby intent or constraints: `CHECK: f64x2.abs{{$}}`.
  **L917 CN**: 注释说明附近代码的意图或约束：`CHECK: f64x2.abs{{$}}`。
- **L918 EN**: Starts a function or method definition for `test_f64x2_abs`.
  **L918 CN**: 开始定义函数或方法 `test_f64x2_abs`。
- **L919 EN**: Blank line separating nearby declarations or logic.
  **L919 CN**: 空行，用于分隔相邻声明或逻辑。
- **L920 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f64x2_neg:`.
  **L920 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f64x2_neg:`。
- **L921 EN**: Comment documents nearby intent or constraints: `CHECK: f64x2.neg{{$}}`.
  **L921 CN**: 注释说明附近代码的意图或约束：`CHECK: f64x2.neg{{$}}`。
- **L922 EN**: Starts a function or method definition for `test_f64x2_neg`.
  **L922 CN**: 开始定义函数或方法 `test_f64x2_neg`。
- **L923 EN**: Blank line separating nearby declarations or logic.
  **L923 CN**: 空行，用于分隔相邻声明或逻辑。
- **L924 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f64x2_sqrt:`.
  **L924 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f64x2_sqrt:`。
- **L925 EN**: Comment documents nearby intent or constraints: `CHECK: f64x2.sqrt{{$}}`.
  **L925 CN**: 注释说明附近代码的意图或约束：`CHECK: f64x2.sqrt{{$}}`。
- **L926 EN**: Starts a function or method definition for `test_f64x2_sqrt`.
  **L926 CN**: 开始定义函数或方法 `test_f64x2_sqrt`。
- **L927 EN**: Blank line separating nearby declarations or logic.
  **L927 CN**: 空行，用于分隔相邻声明或逻辑。
- **L928 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f64x2_ceil:`.
  **L928 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f64x2_ceil:`。
- **L929 EN**: Comment documents nearby intent or constraints: `CHECK: f64x2.ceil{{$}}`.
  **L929 CN**: 注释说明附近代码的意图或约束：`CHECK: f64x2.ceil{{$}}`。
- **L930 EN**: Starts a function or method definition for `test_f64x2_ceil`.
  **L930 CN**: 开始定义函数或方法 `test_f64x2_ceil`。
- **L931 EN**: Blank line separating nearby declarations or logic.
  **L931 CN**: 空行，用于分隔相邻声明或逻辑。
- **L932 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f64x2_floor:`.
  **L932 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f64x2_floor:`。
- **L933 EN**: Comment documents nearby intent or constraints: `CHECK: f64x2.floor{{$}}`.
  **L933 CN**: 注释说明附近代码的意图或约束：`CHECK: f64x2.floor{{$}}`。
- **L934 EN**: Starts a function or method definition for `test_f64x2_floor`.
  **L934 CN**: 开始定义函数或方法 `test_f64x2_floor`。
- **L935 EN**: Blank line separating nearby declarations or logic.
  **L935 CN**: 空行，用于分隔相邻声明或逻辑。
- **L936 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f64x2_trunc:`.
  **L936 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f64x2_trunc:`。

### Lines 937-960

````c
// CHECK: f64x2.trunc{{$}}
v128_t test_f64x2_trunc(v128_t a) { return wasm_f64x2_trunc(a); }

// CHECK-LABEL: test_f64x2_nearest:
// CHECK: f64x2.nearest{{$}}
v128_t test_f64x2_nearest(v128_t a) { return wasm_f64x2_nearest(a); }

// CHECK-LABEL: test_f64x2_add:
// CHECK: f64x2.add{{$}}
v128_t test_f64x2_add(v128_t a, v128_t b) { return wasm_f64x2_add(a, b); }

// CHECK-LABEL: test_f64x2_sub:
// CHECK: f64x2.sub{{$}}
v128_t test_f64x2_sub(v128_t a, v128_t b) { return wasm_f64x2_sub(a, b); }

// CHECK-LABEL: test_f64x2_mul:
// CHECK: f64x2.mul{{$}}
v128_t test_f64x2_mul(v128_t a, v128_t b) { return wasm_f64x2_mul(a, b); }

// CHECK-LABEL: test_f64x2_div:
// CHECK: f64x2.div{{$}}
v128_t test_f64x2_div(v128_t a, v128_t b) { return wasm_f64x2_div(a, b); }

// CHECK-LABEL: test_f64x2_min:
````
- **L937 EN**: Comment documents nearby intent or constraints: `CHECK: f64x2.trunc{{$}}`.
  **L937 CN**: 注释说明附近代码的意图或约束：`CHECK: f64x2.trunc{{$}}`。
- **L938 EN**: Starts a function or method definition for `test_f64x2_trunc`.
  **L938 CN**: 开始定义函数或方法 `test_f64x2_trunc`。
- **L939 EN**: Blank line separating nearby declarations or logic.
  **L939 CN**: 空行，用于分隔相邻声明或逻辑。
- **L940 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f64x2_nearest:`.
  **L940 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f64x2_nearest:`。
- **L941 EN**: Comment documents nearby intent or constraints: `CHECK: f64x2.nearest{{$}}`.
  **L941 CN**: 注释说明附近代码的意图或约束：`CHECK: f64x2.nearest{{$}}`。
- **L942 EN**: Starts a function or method definition for `test_f64x2_nearest`.
  **L942 CN**: 开始定义函数或方法 `test_f64x2_nearest`。
- **L943 EN**: Blank line separating nearby declarations or logic.
  **L943 CN**: 空行，用于分隔相邻声明或逻辑。
- **L944 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f64x2_add:`.
  **L944 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f64x2_add:`。
- **L945 EN**: Comment documents nearby intent or constraints: `CHECK: f64x2.add{{$}}`.
  **L945 CN**: 注释说明附近代码的意图或约束：`CHECK: f64x2.add{{$}}`。
- **L946 EN**: Starts a function or method definition for `test_f64x2_add`.
  **L946 CN**: 开始定义函数或方法 `test_f64x2_add`。
- **L947 EN**: Blank line separating nearby declarations or logic.
  **L947 CN**: 空行，用于分隔相邻声明或逻辑。
- **L948 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f64x2_sub:`.
  **L948 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f64x2_sub:`。
- **L949 EN**: Comment documents nearby intent or constraints: `CHECK: f64x2.sub{{$}}`.
  **L949 CN**: 注释说明附近代码的意图或约束：`CHECK: f64x2.sub{{$}}`。
- **L950 EN**: Starts a function or method definition for `test_f64x2_sub`.
  **L950 CN**: 开始定义函数或方法 `test_f64x2_sub`。
- **L951 EN**: Blank line separating nearby declarations or logic.
  **L951 CN**: 空行，用于分隔相邻声明或逻辑。
- **L952 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f64x2_mul:`.
  **L952 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f64x2_mul:`。
- **L953 EN**: Comment documents nearby intent or constraints: `CHECK: f64x2.mul{{$}}`.
  **L953 CN**: 注释说明附近代码的意图或约束：`CHECK: f64x2.mul{{$}}`。
- **L954 EN**: Starts a function or method definition for `test_f64x2_mul`.
  **L954 CN**: 开始定义函数或方法 `test_f64x2_mul`。
- **L955 EN**: Blank line separating nearby declarations or logic.
  **L955 CN**: 空行，用于分隔相邻声明或逻辑。
- **L956 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f64x2_div:`.
  **L956 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f64x2_div:`。
- **L957 EN**: Comment documents nearby intent or constraints: `CHECK: f64x2.div{{$}}`.
  **L957 CN**: 注释说明附近代码的意图或约束：`CHECK: f64x2.div{{$}}`。
- **L958 EN**: Starts a function or method definition for `test_f64x2_div`.
  **L958 CN**: 开始定义函数或方法 `test_f64x2_div`。
- **L959 EN**: Blank line separating nearby declarations or logic.
  **L959 CN**: 空行，用于分隔相邻声明或逻辑。
- **L960 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f64x2_min:`.
  **L960 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f64x2_min:`。

### Lines 961-984

````c
// CHECK: f64x2.min{{$}}
v128_t test_f64x2_min(v128_t a, v128_t b) { return wasm_f64x2_min(a, b); }

// CHECK-LABEL: test_f64x2_max:
// CHECK: f64x2.max{{$}}
v128_t test_f64x2_max(v128_t a, v128_t b) { return wasm_f64x2_max(a, b); }

// CHECK-LABEL: test_f64x2_pmin:
// CHECK: f64x2.pmin{{$}}
v128_t test_f64x2_pmin(v128_t a, v128_t b) { return wasm_f64x2_pmin(a, b); }

// CHECK-LABEL: test_f64x2_pmax:
// CHECK: f64x2.pmax{{$}}
v128_t test_f64x2_pmax(v128_t a, v128_t b) { return wasm_f64x2_pmax(a, b); }

// CHECK-LABEL: test_i32x4_trunc_sat_f32x4:
// CHECK: i32x4.trunc_sat_f32x4_s{{$}}
v128_t test_i32x4_trunc_sat_f32x4(v128_t a) {
  return wasm_i32x4_trunc_sat_f32x4(a);
}

// CHECK-LABEL: test_u32x4_trunc_sat_f32x4:
// CHECK: i32x4.trunc_sat_f32x4_u{{$}}
v128_t test_u32x4_trunc_sat_f32x4(v128_t a) {
````
- **L961 EN**: Comment documents nearby intent or constraints: `CHECK: f64x2.min{{$}}`.
  **L961 CN**: 注释说明附近代码的意图或约束：`CHECK: f64x2.min{{$}}`。
- **L962 EN**: Starts a function or method definition for `test_f64x2_min`.
  **L962 CN**: 开始定义函数或方法 `test_f64x2_min`。
- **L963 EN**: Blank line separating nearby declarations or logic.
  **L963 CN**: 空行，用于分隔相邻声明或逻辑。
- **L964 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f64x2_max:`.
  **L964 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f64x2_max:`。
- **L965 EN**: Comment documents nearby intent or constraints: `CHECK: f64x2.max{{$}}`.
  **L965 CN**: 注释说明附近代码的意图或约束：`CHECK: f64x2.max{{$}}`。
- **L966 EN**: Starts a function or method definition for `test_f64x2_max`.
  **L966 CN**: 开始定义函数或方法 `test_f64x2_max`。
- **L967 EN**: Blank line separating nearby declarations or logic.
  **L967 CN**: 空行，用于分隔相邻声明或逻辑。
- **L968 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f64x2_pmin:`.
  **L968 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f64x2_pmin:`。
- **L969 EN**: Comment documents nearby intent or constraints: `CHECK: f64x2.pmin{{$}}`.
  **L969 CN**: 注释说明附近代码的意图或约束：`CHECK: f64x2.pmin{{$}}`。
- **L970 EN**: Starts a function or method definition for `test_f64x2_pmin`.
  **L970 CN**: 开始定义函数或方法 `test_f64x2_pmin`。
- **L971 EN**: Blank line separating nearby declarations or logic.
  **L971 CN**: 空行，用于分隔相邻声明或逻辑。
- **L972 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f64x2_pmax:`.
  **L972 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f64x2_pmax:`。
- **L973 EN**: Comment documents nearby intent or constraints: `CHECK: f64x2.pmax{{$}}`.
  **L973 CN**: 注释说明附近代码的意图或约束：`CHECK: f64x2.pmax{{$}}`。
- **L974 EN**: Starts a function or method definition for `test_f64x2_pmax`.
  **L974 CN**: 开始定义函数或方法 `test_f64x2_pmax`。
- **L975 EN**: Blank line separating nearby declarations or logic.
  **L975 CN**: 空行，用于分隔相邻声明或逻辑。
- **L976 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i32x4_trunc_sat_f32x4:`.
  **L976 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i32x4_trunc_sat_f32x4:`。
- **L977 EN**: Comment documents nearby intent or constraints: `CHECK: i32x4.trunc_sat_f32x4_s{{$}}`.
  **L977 CN**: 注释说明附近代码的意图或约束：`CHECK: i32x4.trunc_sat_f32x4_s{{$}}`。
- **L978 EN**: Starts a function or method definition for `test_i32x4_trunc_sat_f32x4`.
  **L978 CN**: 开始定义函数或方法 `test_i32x4_trunc_sat_f32x4`。
- **L979 EN**: Returns from the current function with `wasm_i32x4_trunc_sat_f32x4(a)`.
  **L979 CN**: 以 `wasm_i32x4_trunc_sat_f32x4(a)` 从当前函数返回。
- **L980 EN**: Closes the current lexical scope or compound statement.
  **L980 CN**: 结束当前词法作用域或复合语句块。
- **L981 EN**: Blank line separating nearby declarations or logic.
  **L981 CN**: 空行，用于分隔相邻声明或逻辑。
- **L982 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_u32x4_trunc_sat_f32x4:`.
  **L982 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_u32x4_trunc_sat_f32x4:`。
- **L983 EN**: Comment documents nearby intent or constraints: `CHECK: i32x4.trunc_sat_f32x4_u{{$}}`.
  **L983 CN**: 注释说明附近代码的意图或约束：`CHECK: i32x4.trunc_sat_f32x4_u{{$}}`。
- **L984 EN**: Starts a function or method definition for `test_u32x4_trunc_sat_f32x4`.
  **L984 CN**: 开始定义函数或方法 `test_u32x4_trunc_sat_f32x4`。

### Lines 985-1008

````c
  return wasm_u32x4_trunc_sat_f32x4(a);
}

// CHECK-LABEL: test_f32x4_convert_i32x4:
// CHECK: f32x4.convert_i32x4_s{{$}}
v128_t test_f32x4_convert_i32x4(v128_t a) {
  return wasm_f32x4_convert_i32x4(a);
}

// CHECK-LABEL: test_f32x4_convert_u32x4:
// CHECK: f32x4.convert_i32x4_u{{$}}
v128_t test_f32x4_convert_u32x4(v128_t a) {
  return wasm_f32x4_convert_u32x4(a);
}

// CHECK-LABEL: test_f64x2_convert_low_i32x4:
// CHECK: f64x2.convert_low_i32x4_s{{$}}
v128_t test_f64x2_convert_low_i32x4(v128_t a) {
  return wasm_f64x2_convert_low_i32x4(a);
}

// CHECK-LABEL: test_f64x2_convert_low_u32x4:
// CHECK: f64x2.convert_low_i32x4_u{{$}}
v128_t test_f64x2_convert_low_u32x4(v128_t a) {
````
- **L985 EN**: Returns from the current function with `wasm_u32x4_trunc_sat_f32x4(a)`.
  **L985 CN**: 以 `wasm_u32x4_trunc_sat_f32x4(a)` 从当前函数返回。
- **L986 EN**: Closes the current lexical scope or compound statement.
  **L986 CN**: 结束当前词法作用域或复合语句块。
- **L987 EN**: Blank line separating nearby declarations or logic.
  **L987 CN**: 空行，用于分隔相邻声明或逻辑。
- **L988 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f32x4_convert_i32x4:`.
  **L988 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f32x4_convert_i32x4:`。
- **L989 EN**: Comment documents nearby intent or constraints: `CHECK: f32x4.convert_i32x4_s{{$}}`.
  **L989 CN**: 注释说明附近代码的意图或约束：`CHECK: f32x4.convert_i32x4_s{{$}}`。
- **L990 EN**: Starts a function or method definition for `test_f32x4_convert_i32x4`.
  **L990 CN**: 开始定义函数或方法 `test_f32x4_convert_i32x4`。
- **L991 EN**: Returns from the current function with `wasm_f32x4_convert_i32x4(a)`.
  **L991 CN**: 以 `wasm_f32x4_convert_i32x4(a)` 从当前函数返回。
- **L992 EN**: Closes the current lexical scope or compound statement.
  **L992 CN**: 结束当前词法作用域或复合语句块。
- **L993 EN**: Blank line separating nearby declarations or logic.
  **L993 CN**: 空行，用于分隔相邻声明或逻辑。
- **L994 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f32x4_convert_u32x4:`.
  **L994 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f32x4_convert_u32x4:`。
- **L995 EN**: Comment documents nearby intent or constraints: `CHECK: f32x4.convert_i32x4_u{{$}}`.
  **L995 CN**: 注释说明附近代码的意图或约束：`CHECK: f32x4.convert_i32x4_u{{$}}`。
- **L996 EN**: Starts a function or method definition for `test_f32x4_convert_u32x4`.
  **L996 CN**: 开始定义函数或方法 `test_f32x4_convert_u32x4`。
- **L997 EN**: Returns from the current function with `wasm_f32x4_convert_u32x4(a)`.
  **L997 CN**: 以 `wasm_f32x4_convert_u32x4(a)` 从当前函数返回。
- **L998 EN**: Closes the current lexical scope or compound statement.
  **L998 CN**: 结束当前词法作用域或复合语句块。
- **L999 EN**: Blank line separating nearby declarations or logic.
  **L999 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1000 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f64x2_convert_low_i32x4:`.
  **L1000 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f64x2_convert_low_i32x4:`。
- **L1001 EN**: Comment documents nearby intent or constraints: `CHECK: f64x2.convert_low_i32x4_s{{$}}`.
  **L1001 CN**: 注释说明附近代码的意图或约束：`CHECK: f64x2.convert_low_i32x4_s{{$}}`。
- **L1002 EN**: Starts a function or method definition for `test_f64x2_convert_low_i32x4`.
  **L1002 CN**: 开始定义函数或方法 `test_f64x2_convert_low_i32x4`。
- **L1003 EN**: Returns from the current function with `wasm_f64x2_convert_low_i32x4(a)`.
  **L1003 CN**: 以 `wasm_f64x2_convert_low_i32x4(a)` 从当前函数返回。
- **L1004 EN**: Closes the current lexical scope or compound statement.
  **L1004 CN**: 结束当前词法作用域或复合语句块。
- **L1005 EN**: Blank line separating nearby declarations or logic.
  **L1005 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1006 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f64x2_convert_low_u32x4:`.
  **L1006 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f64x2_convert_low_u32x4:`。
- **L1007 EN**: Comment documents nearby intent or constraints: `CHECK: f64x2.convert_low_i32x4_u{{$}}`.
  **L1007 CN**: 注释说明附近代码的意图或约束：`CHECK: f64x2.convert_low_i32x4_u{{$}}`。
- **L1008 EN**: Starts a function or method definition for `test_f64x2_convert_low_u32x4`.
  **L1008 CN**: 开始定义函数或方法 `test_f64x2_convert_low_u32x4`。

### Lines 1009-1032

````c
  return wasm_f64x2_convert_low_u32x4(a);
}

// CHECK-LABEL: test_i32x4_trunc_sat_f64x2_zero:
// CHECK: i32x4.trunc_sat_f64x2_s_zero{{$}}
v128_t test_i32x4_trunc_sat_f64x2_zero(v128_t a) {
  return wasm_i32x4_trunc_sat_f64x2_zero(a);
}

// CHECK-LABEL: test_u32x4_trunc_sat_f64x2_zero:
// CHECK: i32x4.trunc_sat_f64x2_u_zero{{$}}
v128_t test_u32x4_trunc_sat_f64x2_zero(v128_t a) {
  return wasm_u32x4_trunc_sat_f64x2_zero(a);
}

// CHECK-LABEL: test_f32x4_demote_f64x2_zero:
// CHECK: f32x4.demote_f64x2_zero{{$}}
v128_t test_f32x4_demote_f64x2_zero(v128_t a) {
  return wasm_f32x4_demote_f64x2_zero(a);
}

// CHECK-LABEL: test_f64x2_promote_low_f32x4:
// CHECK: f64x2.promote_low_f32x4{{$}}
v128_t test_f64x2_promote_low_f32x4(v128_t a) {
````
- **L1009 EN**: Returns from the current function with `wasm_f64x2_convert_low_u32x4(a)`.
  **L1009 CN**: 以 `wasm_f64x2_convert_low_u32x4(a)` 从当前函数返回。
- **L1010 EN**: Closes the current lexical scope or compound statement.
  **L1010 CN**: 结束当前词法作用域或复合语句块。
- **L1011 EN**: Blank line separating nearby declarations or logic.
  **L1011 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1012 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i32x4_trunc_sat_f64x2_zero:`.
  **L1012 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i32x4_trunc_sat_f64x2_zero:`。
- **L1013 EN**: Comment documents nearby intent or constraints: `CHECK: i32x4.trunc_sat_f64x2_s_zero{{$}}`.
  **L1013 CN**: 注释说明附近代码的意图或约束：`CHECK: i32x4.trunc_sat_f64x2_s_zero{{$}}`。
- **L1014 EN**: Starts a function or method definition for `test_i32x4_trunc_sat_f64x2_zero`.
  **L1014 CN**: 开始定义函数或方法 `test_i32x4_trunc_sat_f64x2_zero`。
- **L1015 EN**: Returns from the current function with `wasm_i32x4_trunc_sat_f64x2_zero(a)`.
  **L1015 CN**: 以 `wasm_i32x4_trunc_sat_f64x2_zero(a)` 从当前函数返回。
- **L1016 EN**: Closes the current lexical scope or compound statement.
  **L1016 CN**: 结束当前词法作用域或复合语句块。
- **L1017 EN**: Blank line separating nearby declarations or logic.
  **L1017 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1018 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_u32x4_trunc_sat_f64x2_zero:`.
  **L1018 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_u32x4_trunc_sat_f64x2_zero:`。
- **L1019 EN**: Comment documents nearby intent or constraints: `CHECK: i32x4.trunc_sat_f64x2_u_zero{{$}}`.
  **L1019 CN**: 注释说明附近代码的意图或约束：`CHECK: i32x4.trunc_sat_f64x2_u_zero{{$}}`。
- **L1020 EN**: Starts a function or method definition for `test_u32x4_trunc_sat_f64x2_zero`.
  **L1020 CN**: 开始定义函数或方法 `test_u32x4_trunc_sat_f64x2_zero`。
- **L1021 EN**: Returns from the current function with `wasm_u32x4_trunc_sat_f64x2_zero(a)`.
  **L1021 CN**: 以 `wasm_u32x4_trunc_sat_f64x2_zero(a)` 从当前函数返回。
- **L1022 EN**: Closes the current lexical scope or compound statement.
  **L1022 CN**: 结束当前词法作用域或复合语句块。
- **L1023 EN**: Blank line separating nearby declarations or logic.
  **L1023 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1024 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f32x4_demote_f64x2_zero:`.
  **L1024 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f32x4_demote_f64x2_zero:`。
- **L1025 EN**: Comment documents nearby intent or constraints: `CHECK: f32x4.demote_f64x2_zero{{$}}`.
  **L1025 CN**: 注释说明附近代码的意图或约束：`CHECK: f32x4.demote_f64x2_zero{{$}}`。
- **L1026 EN**: Starts a function or method definition for `test_f32x4_demote_f64x2_zero`.
  **L1026 CN**: 开始定义函数或方法 `test_f32x4_demote_f64x2_zero`。
- **L1027 EN**: Returns from the current function with `wasm_f32x4_demote_f64x2_zero(a)`.
  **L1027 CN**: 以 `wasm_f32x4_demote_f64x2_zero(a)` 从当前函数返回。
- **L1028 EN**: Closes the current lexical scope or compound statement.
  **L1028 CN**: 结束当前词法作用域或复合语句块。
- **L1029 EN**: Blank line separating nearby declarations or logic.
  **L1029 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1030 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f64x2_promote_low_f32x4:`.
  **L1030 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f64x2_promote_low_f32x4:`。
- **L1031 EN**: Comment documents nearby intent or constraints: `CHECK: f64x2.promote_low_f32x4{{$}}`.
  **L1031 CN**: 注释说明附近代码的意图或约束：`CHECK: f64x2.promote_low_f32x4{{$}}`。
- **L1032 EN**: Starts a function or method definition for `test_f64x2_promote_low_f32x4`.
  **L1032 CN**: 开始定义函数或方法 `test_f64x2_promote_low_f32x4`。

### Lines 1033-1056

````c
  return wasm_f64x2_promote_low_f32x4(a);
}

// CHECK-LABEL: test_f32x4_promote_low_f16x8:
// CHECK: f32x4.promote_low_f16x8{{$}}
v128_t test_f32x4_promote_low_f16x8(v128_t a) {
  return wasm_f32x4_promote_low_f16x8(a);
}

// CHECK-LABEL: test_i8x16_shuffle:
// CHECK: i8x16.shuffle 15, 14, 13, 12, 11, 10, 9, 8, 7, 6, 5, 4, 3, 2, 1,
// 0{{$}}
v128_t test_i8x16_shuffle(v128_t a, v128_t b) {
  return wasm_i8x16_shuffle(a, b, 15, 14, 13, 12, 11, 10, 9, 8, 7, 6, 5, 4, 3,
                            2, 1, 0);
}

// CHECK-LABEL: test_i16x8_shuffle:
// CHECK: i8x16.shuffle 14, 15, 12, 13, 10, 11, 8, 9, 6, 7, 4, 5, 2, 3, 0,
// 1{{$}}
v128_t test_i16x8_shuffle(v128_t a, v128_t b) {
  return wasm_i16x8_shuffle(a, b, 7, 6, 5, 4, 3, 2, 1, 0);
}

````
- **L1033 EN**: Returns from the current function with `wasm_f64x2_promote_low_f32x4(a)`.
  **L1033 CN**: 以 `wasm_f64x2_promote_low_f32x4(a)` 从当前函数返回。
- **L1034 EN**: Closes the current lexical scope or compound statement.
  **L1034 CN**: 结束当前词法作用域或复合语句块。
- **L1035 EN**: Blank line separating nearby declarations or logic.
  **L1035 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1036 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f32x4_promote_low_f16x8:`.
  **L1036 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f32x4_promote_low_f16x8:`。
- **L1037 EN**: Comment documents nearby intent or constraints: `CHECK: f32x4.promote_low_f16x8{{$}}`.
  **L1037 CN**: 注释说明附近代码的意图或约束：`CHECK: f32x4.promote_low_f16x8{{$}}`。
- **L1038 EN**: Starts a function or method definition for `test_f32x4_promote_low_f16x8`.
  **L1038 CN**: 开始定义函数或方法 `test_f32x4_promote_low_f16x8`。
- **L1039 EN**: Returns from the current function with `wasm_f32x4_promote_low_f16x8(a)`.
  **L1039 CN**: 以 `wasm_f32x4_promote_low_f16x8(a)` 从当前函数返回。
- **L1040 EN**: Closes the current lexical scope or compound statement.
  **L1040 CN**: 结束当前词法作用域或复合语句块。
- **L1041 EN**: Blank line separating nearby declarations or logic.
  **L1041 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1042 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i8x16_shuffle:`.
  **L1042 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i8x16_shuffle:`。
- **L1043 EN**: Comment documents nearby intent or constraints: `CHECK: i8x16.shuffle 15, 14, 13, 12, 11, 10, 9, 8, 7, 6, 5, 4, 3, 2, 1,`.
  **L1043 CN**: 注释说明附近代码的意图或约束：`CHECK: i8x16.shuffle 15, 14, 13, 12, 11, 10, 9, 8, 7, 6, 5, 4, 3, 2, 1,`。
- **L1044 EN**: Comment documents nearby intent or constraints: `0{{$}}`.
  **L1044 CN**: 注释说明附近代码的意图或约束：`0{{$}}`。
- **L1045 EN**: Starts a function or method definition for `test_i8x16_shuffle`.
  **L1045 CN**: 开始定义函数或方法 `test_i8x16_shuffle`。
- **L1046 EN**: Returns from the current function with `wasm_i8x16_shuffle(a, b, 15, 14, 13, 12, 11, 10, 9, 8, 7, 6, 5, 4, 3,`.
  **L1046 CN**: 以 `wasm_i8x16_shuffle(a, b, 15, 14, 13, 12, 11, 10, 9, 8, 7, 6, 5, 4, 3,` 从当前函数返回。
- **L1047 EN**: Executes a standalone statement or declaration: `2, 1, 0);`.
  **L1047 CN**: 执行一条独立语句或声明：`2, 1, 0);`。
- **L1048 EN**: Closes the current lexical scope or compound statement.
  **L1048 CN**: 结束当前词法作用域或复合语句块。
- **L1049 EN**: Blank line separating nearby declarations or logic.
  **L1049 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1050 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i16x8_shuffle:`.
  **L1050 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i16x8_shuffle:`。
- **L1051 EN**: Comment documents nearby intent or constraints: `CHECK: i8x16.shuffle 14, 15, 12, 13, 10, 11, 8, 9, 6, 7, 4, 5, 2, 3, 0,`.
  **L1051 CN**: 注释说明附近代码的意图或约束：`CHECK: i8x16.shuffle 14, 15, 12, 13, 10, 11, 8, 9, 6, 7, 4, 5, 2, 3, 0,`。
- **L1052 EN**: Comment documents nearby intent or constraints: `1{{$}}`.
  **L1052 CN**: 注释说明附近代码的意图或约束：`1{{$}}`。
- **L1053 EN**: Starts a function or method definition for `test_i16x8_shuffle`.
  **L1053 CN**: 开始定义函数或方法 `test_i16x8_shuffle`。
- **L1054 EN**: Returns from the current function with `wasm_i16x8_shuffle(a, b, 7, 6, 5, 4, 3, 2, 1, 0)`.
  **L1054 CN**: 以 `wasm_i16x8_shuffle(a, b, 7, 6, 5, 4, 3, 2, 1, 0)` 从当前函数返回。
- **L1055 EN**: Closes the current lexical scope or compound statement.
  **L1055 CN**: 结束当前词法作用域或复合语句块。
- **L1056 EN**: Blank line separating nearby declarations or logic.
  **L1056 CN**: 空行，用于分隔相邻声明或逻辑。

### Lines 1057-1080

````c
// CHECK-LABEL: test_i32x4_shuffle:
// CHECK: i8x16.shuffle 12, 13, 14, 15, 8, 9, 10, 11, 4, 5, 6, 7, 0, 1, 2,
// 3{{$}}
v128_t test_i32x4_shuffle(v128_t a, v128_t b) {
  return wasm_i32x4_shuffle(a, b, 3, 2, 1, 0);
}

// CHECK-LABEL: test_i64x2_shuffle:
// CHECK: i8x16.shuffle 8, 9, 10, 11, 12, 13, 14, 15, 0, 1, 2, 3, 4, 5, 6,
// 7{{$}}
v128_t test_i64x2_shuffle(v128_t a, v128_t b) {
  return wasm_i64x2_shuffle(a, b, 1, 0);
}

// CHECK-LABEL: test_i8x16_swizzle:
// CHECK: i8x16.swizzle{{$}}
v128_t test_i8x16_swizzle(v128_t a, v128_t b) {
  return wasm_i8x16_swizzle(a, b);
}

// CHECK-LABEL: test_i8x16_narrow_i16x8:
// CHECK: i8x16.narrow_i16x8_s{{$}}
v128_t test_i8x16_narrow_i16x8(v128_t a, v128_t b) {
  return wasm_i8x16_narrow_i16x8(a, b);
````
- **L1057 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i32x4_shuffle:`.
  **L1057 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i32x4_shuffle:`。
- **L1058 EN**: Comment documents nearby intent or constraints: `CHECK: i8x16.shuffle 12, 13, 14, 15, 8, 9, 10, 11, 4, 5, 6, 7, 0, 1, 2,`.
  **L1058 CN**: 注释说明附近代码的意图或约束：`CHECK: i8x16.shuffle 12, 13, 14, 15, 8, 9, 10, 11, 4, 5, 6, 7, 0, 1, 2,`。
- **L1059 EN**: Comment documents nearby intent or constraints: `3{{$}}`.
  **L1059 CN**: 注释说明附近代码的意图或约束：`3{{$}}`。
- **L1060 EN**: Starts a function or method definition for `test_i32x4_shuffle`.
  **L1060 CN**: 开始定义函数或方法 `test_i32x4_shuffle`。
- **L1061 EN**: Returns from the current function with `wasm_i32x4_shuffle(a, b, 3, 2, 1, 0)`.
  **L1061 CN**: 以 `wasm_i32x4_shuffle(a, b, 3, 2, 1, 0)` 从当前函数返回。
- **L1062 EN**: Closes the current lexical scope or compound statement.
  **L1062 CN**: 结束当前词法作用域或复合语句块。
- **L1063 EN**: Blank line separating nearby declarations or logic.
  **L1063 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1064 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i64x2_shuffle:`.
  **L1064 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i64x2_shuffle:`。
- **L1065 EN**: Comment documents nearby intent or constraints: `CHECK: i8x16.shuffle 8, 9, 10, 11, 12, 13, 14, 15, 0, 1, 2, 3, 4, 5, 6,`.
  **L1065 CN**: 注释说明附近代码的意图或约束：`CHECK: i8x16.shuffle 8, 9, 10, 11, 12, 13, 14, 15, 0, 1, 2, 3, 4, 5, 6,`。
- **L1066 EN**: Comment documents nearby intent or constraints: `7{{$}}`.
  **L1066 CN**: 注释说明附近代码的意图或约束：`7{{$}}`。
- **L1067 EN**: Starts a function or method definition for `test_i64x2_shuffle`.
  **L1067 CN**: 开始定义函数或方法 `test_i64x2_shuffle`。
- **L1068 EN**: Returns from the current function with `wasm_i64x2_shuffle(a, b, 1, 0)`.
  **L1068 CN**: 以 `wasm_i64x2_shuffle(a, b, 1, 0)` 从当前函数返回。
- **L1069 EN**: Closes the current lexical scope or compound statement.
  **L1069 CN**: 结束当前词法作用域或复合语句块。
- **L1070 EN**: Blank line separating nearby declarations or logic.
  **L1070 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1071 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i8x16_swizzle:`.
  **L1071 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i8x16_swizzle:`。
- **L1072 EN**: Comment documents nearby intent or constraints: `CHECK: i8x16.swizzle{{$}}`.
  **L1072 CN**: 注释说明附近代码的意图或约束：`CHECK: i8x16.swizzle{{$}}`。
- **L1073 EN**: Starts a function or method definition for `test_i8x16_swizzle`.
  **L1073 CN**: 开始定义函数或方法 `test_i8x16_swizzle`。
- **L1074 EN**: Returns from the current function with `wasm_i8x16_swizzle(a, b)`.
  **L1074 CN**: 以 `wasm_i8x16_swizzle(a, b)` 从当前函数返回。
- **L1075 EN**: Closes the current lexical scope or compound statement.
  **L1075 CN**: 结束当前词法作用域或复合语句块。
- **L1076 EN**: Blank line separating nearby declarations or logic.
  **L1076 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1077 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i8x16_narrow_i16x8:`.
  **L1077 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i8x16_narrow_i16x8:`。
- **L1078 EN**: Comment documents nearby intent or constraints: `CHECK: i8x16.narrow_i16x8_s{{$}}`.
  **L1078 CN**: 注释说明附近代码的意图或约束：`CHECK: i8x16.narrow_i16x8_s{{$}}`。
- **L1079 EN**: Starts a function or method definition for `test_i8x16_narrow_i16x8`.
  **L1079 CN**: 开始定义函数或方法 `test_i8x16_narrow_i16x8`。
- **L1080 EN**: Returns from the current function with `wasm_i8x16_narrow_i16x8(a, b)`.
  **L1080 CN**: 以 `wasm_i8x16_narrow_i16x8(a, b)` 从当前函数返回。

### Lines 1081-1104

````c
}

// CHECK-LABEL: test_u8x16_narrow_i16x8:
// CHECK: i8x16.narrow_i16x8_u{{$}}
v128_t test_u8x16_narrow_i16x8(v128_t a, v128_t b) {
  return wasm_u8x16_narrow_i16x8(a, b);
}

// CHECK-LABEL: test_i16x8_narrow_i32x4:
// CHECK: i16x8.narrow_i32x4_s{{$}}
v128_t test_i16x8_narrow_i32x4(v128_t a, v128_t b) {
  return wasm_i16x8_narrow_i32x4(a, b);
}

// CHECK-LABEL: test_u16x8_narrow_i32x4:
// CHECK: i16x8.narrow_i32x4_u{{$}}
v128_t test_u16x8_narrow_i32x4(v128_t a, v128_t b) {
  return wasm_u16x8_narrow_i32x4(a, b);
}

// CHECK-LABEL: test_i16x8_extend_low_i8x16:
// CHECK: i16x8.extend_low_i8x16_s{{$}}
v128_t test_i16x8_extend_low_i8x16(v128_t a) {
  return wasm_i16x8_extend_low_i8x16(a);
````
- **L1081 EN**: Closes the current lexical scope or compound statement.
  **L1081 CN**: 结束当前词法作用域或复合语句块。
- **L1082 EN**: Blank line separating nearby declarations or logic.
  **L1082 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1083 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_u8x16_narrow_i16x8:`.
  **L1083 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_u8x16_narrow_i16x8:`。
- **L1084 EN**: Comment documents nearby intent or constraints: `CHECK: i8x16.narrow_i16x8_u{{$}}`.
  **L1084 CN**: 注释说明附近代码的意图或约束：`CHECK: i8x16.narrow_i16x8_u{{$}}`。
- **L1085 EN**: Starts a function or method definition for `test_u8x16_narrow_i16x8`.
  **L1085 CN**: 开始定义函数或方法 `test_u8x16_narrow_i16x8`。
- **L1086 EN**: Returns from the current function with `wasm_u8x16_narrow_i16x8(a, b)`.
  **L1086 CN**: 以 `wasm_u8x16_narrow_i16x8(a, b)` 从当前函数返回。
- **L1087 EN**: Closes the current lexical scope or compound statement.
  **L1087 CN**: 结束当前词法作用域或复合语句块。
- **L1088 EN**: Blank line separating nearby declarations or logic.
  **L1088 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1089 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i16x8_narrow_i32x4:`.
  **L1089 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i16x8_narrow_i32x4:`。
- **L1090 EN**: Comment documents nearby intent or constraints: `CHECK: i16x8.narrow_i32x4_s{{$}}`.
  **L1090 CN**: 注释说明附近代码的意图或约束：`CHECK: i16x8.narrow_i32x4_s{{$}}`。
- **L1091 EN**: Starts a function or method definition for `test_i16x8_narrow_i32x4`.
  **L1091 CN**: 开始定义函数或方法 `test_i16x8_narrow_i32x4`。
- **L1092 EN**: Returns from the current function with `wasm_i16x8_narrow_i32x4(a, b)`.
  **L1092 CN**: 以 `wasm_i16x8_narrow_i32x4(a, b)` 从当前函数返回。
- **L1093 EN**: Closes the current lexical scope or compound statement.
  **L1093 CN**: 结束当前词法作用域或复合语句块。
- **L1094 EN**: Blank line separating nearby declarations or logic.
  **L1094 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1095 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_u16x8_narrow_i32x4:`.
  **L1095 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_u16x8_narrow_i32x4:`。
- **L1096 EN**: Comment documents nearby intent or constraints: `CHECK: i16x8.narrow_i32x4_u{{$}}`.
  **L1096 CN**: 注释说明附近代码的意图或约束：`CHECK: i16x8.narrow_i32x4_u{{$}}`。
- **L1097 EN**: Starts a function or method definition for `test_u16x8_narrow_i32x4`.
  **L1097 CN**: 开始定义函数或方法 `test_u16x8_narrow_i32x4`。
- **L1098 EN**: Returns from the current function with `wasm_u16x8_narrow_i32x4(a, b)`.
  **L1098 CN**: 以 `wasm_u16x8_narrow_i32x4(a, b)` 从当前函数返回。
- **L1099 EN**: Closes the current lexical scope or compound statement.
  **L1099 CN**: 结束当前词法作用域或复合语句块。
- **L1100 EN**: Blank line separating nearby declarations or logic.
  **L1100 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1101 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i16x8_extend_low_i8x16:`.
  **L1101 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i16x8_extend_low_i8x16:`。
- **L1102 EN**: Comment documents nearby intent or constraints: `CHECK: i16x8.extend_low_i8x16_s{{$}}`.
  **L1102 CN**: 注释说明附近代码的意图或约束：`CHECK: i16x8.extend_low_i8x16_s{{$}}`。
- **L1103 EN**: Starts a function or method definition for `test_i16x8_extend_low_i8x16`.
  **L1103 CN**: 开始定义函数或方法 `test_i16x8_extend_low_i8x16`。
- **L1104 EN**: Returns from the current function with `wasm_i16x8_extend_low_i8x16(a)`.
  **L1104 CN**: 以 `wasm_i16x8_extend_low_i8x16(a)` 从当前函数返回。

### Lines 1105-1128

````c
}

// CHECK-LABEL: test_i16x8_extend_high_i8x16:
// CHECK: i16x8.extend_high_i8x16_s{{$}}
v128_t test_i16x8_extend_high_i8x16(v128_t a) {
  return wasm_i16x8_extend_high_i8x16(a);
}

// CHECK-LABEL: test_u16x8_extend_low_u8x16:
// CHECK: i16x8.extend_low_i8x16_u{{$}}
v128_t test_u16x8_extend_low_u8x16(v128_t a) {
  return wasm_u16x8_extend_low_u8x16(a);
}

// CHECK-LABEL: test_u16x8_extend_high_u8x16:
// CHECK: i16x8.extend_high_i8x16_u{{$}}
v128_t test_u16x8_extend_high_u8x16(v128_t a) {
  return wasm_u16x8_extend_high_u8x16(a);
}

// CHECK-LABEL: test_i32x4_extend_low_i16x8:
// CHECK: i32x4.extend_low_i16x8_s{{$}}
v128_t test_i32x4_extend_low_i16x8(v128_t a) {
  return wasm_i32x4_extend_low_i16x8(a);
````
- **L1105 EN**: Closes the current lexical scope or compound statement.
  **L1105 CN**: 结束当前词法作用域或复合语句块。
- **L1106 EN**: Blank line separating nearby declarations or logic.
  **L1106 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1107 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i16x8_extend_high_i8x16:`.
  **L1107 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i16x8_extend_high_i8x16:`。
- **L1108 EN**: Comment documents nearby intent or constraints: `CHECK: i16x8.extend_high_i8x16_s{{$}}`.
  **L1108 CN**: 注释说明附近代码的意图或约束：`CHECK: i16x8.extend_high_i8x16_s{{$}}`。
- **L1109 EN**: Starts a function or method definition for `test_i16x8_extend_high_i8x16`.
  **L1109 CN**: 开始定义函数或方法 `test_i16x8_extend_high_i8x16`。
- **L1110 EN**: Returns from the current function with `wasm_i16x8_extend_high_i8x16(a)`.
  **L1110 CN**: 以 `wasm_i16x8_extend_high_i8x16(a)` 从当前函数返回。
- **L1111 EN**: Closes the current lexical scope or compound statement.
  **L1111 CN**: 结束当前词法作用域或复合语句块。
- **L1112 EN**: Blank line separating nearby declarations or logic.
  **L1112 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1113 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_u16x8_extend_low_u8x16:`.
  **L1113 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_u16x8_extend_low_u8x16:`。
- **L1114 EN**: Comment documents nearby intent or constraints: `CHECK: i16x8.extend_low_i8x16_u{{$}}`.
  **L1114 CN**: 注释说明附近代码的意图或约束：`CHECK: i16x8.extend_low_i8x16_u{{$}}`。
- **L1115 EN**: Starts a function or method definition for `test_u16x8_extend_low_u8x16`.
  **L1115 CN**: 开始定义函数或方法 `test_u16x8_extend_low_u8x16`。
- **L1116 EN**: Returns from the current function with `wasm_u16x8_extend_low_u8x16(a)`.
  **L1116 CN**: 以 `wasm_u16x8_extend_low_u8x16(a)` 从当前函数返回。
- **L1117 EN**: Closes the current lexical scope or compound statement.
  **L1117 CN**: 结束当前词法作用域或复合语句块。
- **L1118 EN**: Blank line separating nearby declarations or logic.
  **L1118 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1119 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_u16x8_extend_high_u8x16:`.
  **L1119 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_u16x8_extend_high_u8x16:`。
- **L1120 EN**: Comment documents nearby intent or constraints: `CHECK: i16x8.extend_high_i8x16_u{{$}}`.
  **L1120 CN**: 注释说明附近代码的意图或约束：`CHECK: i16x8.extend_high_i8x16_u{{$}}`。
- **L1121 EN**: Starts a function or method definition for `test_u16x8_extend_high_u8x16`.
  **L1121 CN**: 开始定义函数或方法 `test_u16x8_extend_high_u8x16`。
- **L1122 EN**: Returns from the current function with `wasm_u16x8_extend_high_u8x16(a)`.
  **L1122 CN**: 以 `wasm_u16x8_extend_high_u8x16(a)` 从当前函数返回。
- **L1123 EN**: Closes the current lexical scope or compound statement.
  **L1123 CN**: 结束当前词法作用域或复合语句块。
- **L1124 EN**: Blank line separating nearby declarations or logic.
  **L1124 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1125 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i32x4_extend_low_i16x8:`.
  **L1125 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i32x4_extend_low_i16x8:`。
- **L1126 EN**: Comment documents nearby intent or constraints: `CHECK: i32x4.extend_low_i16x8_s{{$}}`.
  **L1126 CN**: 注释说明附近代码的意图或约束：`CHECK: i32x4.extend_low_i16x8_s{{$}}`。
- **L1127 EN**: Starts a function or method definition for `test_i32x4_extend_low_i16x8`.
  **L1127 CN**: 开始定义函数或方法 `test_i32x4_extend_low_i16x8`。
- **L1128 EN**: Returns from the current function with `wasm_i32x4_extend_low_i16x8(a)`.
  **L1128 CN**: 以 `wasm_i32x4_extend_low_i16x8(a)` 从当前函数返回。

### Lines 1129-1152

````c
}

// CHECK-LABEL: test_i32x4_extend_high_i16x8:
// CHECK: i32x4.extend_high_i16x8_s{{$}}
v128_t test_i32x4_extend_high_i16x8(v128_t a) {
  return wasm_i32x4_extend_high_i16x8(a);
}

// CHECK-LABEL: test_u32x4_extend_low_u16x8:
// CHECK: i32x4.extend_low_i16x8_u{{$}}
v128_t test_u32x4_extend_low_u16x8(v128_t a) {
  return wasm_u32x4_extend_low_u16x8(a);
}

// CHECK-LABEL: test_u32x4_extend_high_u16x8:
// CHECK: i32x4.extend_high_i16x8_u{{$}}
v128_t test_u32x4_extend_high_u16x8(v128_t a) {
  return wasm_u32x4_extend_high_u16x8(a);
}

// CHECK-LABEL: test_i64x2_extend_low_i32x4:
// CHECK: i64x2.extend_low_i32x4_s{{$}}
v128_t test_i64x2_extend_low_i32x4(v128_t a) {
  return wasm_i64x2_extend_low_i32x4(a);
````
- **L1129 EN**: Closes the current lexical scope or compound statement.
  **L1129 CN**: 结束当前词法作用域或复合语句块。
- **L1130 EN**: Blank line separating nearby declarations or logic.
  **L1130 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1131 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i32x4_extend_high_i16x8:`.
  **L1131 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i32x4_extend_high_i16x8:`。
- **L1132 EN**: Comment documents nearby intent or constraints: `CHECK: i32x4.extend_high_i16x8_s{{$}}`.
  **L1132 CN**: 注释说明附近代码的意图或约束：`CHECK: i32x4.extend_high_i16x8_s{{$}}`。
- **L1133 EN**: Starts a function or method definition for `test_i32x4_extend_high_i16x8`.
  **L1133 CN**: 开始定义函数或方法 `test_i32x4_extend_high_i16x8`。
- **L1134 EN**: Returns from the current function with `wasm_i32x4_extend_high_i16x8(a)`.
  **L1134 CN**: 以 `wasm_i32x4_extend_high_i16x8(a)` 从当前函数返回。
- **L1135 EN**: Closes the current lexical scope or compound statement.
  **L1135 CN**: 结束当前词法作用域或复合语句块。
- **L1136 EN**: Blank line separating nearby declarations or logic.
  **L1136 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1137 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_u32x4_extend_low_u16x8:`.
  **L1137 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_u32x4_extend_low_u16x8:`。
- **L1138 EN**: Comment documents nearby intent or constraints: `CHECK: i32x4.extend_low_i16x8_u{{$}}`.
  **L1138 CN**: 注释说明附近代码的意图或约束：`CHECK: i32x4.extend_low_i16x8_u{{$}}`。
- **L1139 EN**: Starts a function or method definition for `test_u32x4_extend_low_u16x8`.
  **L1139 CN**: 开始定义函数或方法 `test_u32x4_extend_low_u16x8`。
- **L1140 EN**: Returns from the current function with `wasm_u32x4_extend_low_u16x8(a)`.
  **L1140 CN**: 以 `wasm_u32x4_extend_low_u16x8(a)` 从当前函数返回。
- **L1141 EN**: Closes the current lexical scope or compound statement.
  **L1141 CN**: 结束当前词法作用域或复合语句块。
- **L1142 EN**: Blank line separating nearby declarations or logic.
  **L1142 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1143 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_u32x4_extend_high_u16x8:`.
  **L1143 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_u32x4_extend_high_u16x8:`。
- **L1144 EN**: Comment documents nearby intent or constraints: `CHECK: i32x4.extend_high_i16x8_u{{$}}`.
  **L1144 CN**: 注释说明附近代码的意图或约束：`CHECK: i32x4.extend_high_i16x8_u{{$}}`。
- **L1145 EN**: Starts a function or method definition for `test_u32x4_extend_high_u16x8`.
  **L1145 CN**: 开始定义函数或方法 `test_u32x4_extend_high_u16x8`。
- **L1146 EN**: Returns from the current function with `wasm_u32x4_extend_high_u16x8(a)`.
  **L1146 CN**: 以 `wasm_u32x4_extend_high_u16x8(a)` 从当前函数返回。
- **L1147 EN**: Closes the current lexical scope or compound statement.
  **L1147 CN**: 结束当前词法作用域或复合语句块。
- **L1148 EN**: Blank line separating nearby declarations or logic.
  **L1148 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1149 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i64x2_extend_low_i32x4:`.
  **L1149 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i64x2_extend_low_i32x4:`。
- **L1150 EN**: Comment documents nearby intent or constraints: `CHECK: i64x2.extend_low_i32x4_s{{$}}`.
  **L1150 CN**: 注释说明附近代码的意图或约束：`CHECK: i64x2.extend_low_i32x4_s{{$}}`。
- **L1151 EN**: Starts a function or method definition for `test_i64x2_extend_low_i32x4`.
  **L1151 CN**: 开始定义函数或方法 `test_i64x2_extend_low_i32x4`。
- **L1152 EN**: Returns from the current function with `wasm_i64x2_extend_low_i32x4(a)`.
  **L1152 CN**: 以 `wasm_i64x2_extend_low_i32x4(a)` 从当前函数返回。

### Lines 1153-1176

````c
}

// CHECK-LABEL: test_i64x2_extend_high_i32x4:
// CHECK: i64x2.extend_high_i32x4_s{{$}}
v128_t test_i64x2_extend_high_i32x4(v128_t a) {
  return wasm_i64x2_extend_high_i32x4(a);
}

// CHECK-LABEL: test_u64x2_extend_low_u32x4:
// CHECK: i64x2.extend_low_i32x4_u{{$}}
v128_t test_u64x2_extend_low_u32x4(v128_t a) {
  return wasm_u64x2_extend_low_u32x4(a);
}

// CHECK-LABEL: test_u64x2_extend_high_u32x4:
// CHECK: i64x2.extend_high_i32x4_u{{$}}
v128_t test_u64x2_extend_high_u32x4(v128_t a) {
  return wasm_u64x2_extend_high_u32x4(a);
}

// CHECK-LABEL: test_i16x8_extadd_pairwise_i8x16:
// CHECK: i16x8.extadd_pairwise_i8x16_s{{$}}
v128_t test_i16x8_extadd_pairwise_i8x16(v128_t a) {
  return wasm_i16x8_extadd_pairwise_i8x16(a);
````
- **L1153 EN**: Closes the current lexical scope or compound statement.
  **L1153 CN**: 结束当前词法作用域或复合语句块。
- **L1154 EN**: Blank line separating nearby declarations or logic.
  **L1154 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1155 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i64x2_extend_high_i32x4:`.
  **L1155 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i64x2_extend_high_i32x4:`。
- **L1156 EN**: Comment documents nearby intent or constraints: `CHECK: i64x2.extend_high_i32x4_s{{$}}`.
  **L1156 CN**: 注释说明附近代码的意图或约束：`CHECK: i64x2.extend_high_i32x4_s{{$}}`。
- **L1157 EN**: Starts a function or method definition for `test_i64x2_extend_high_i32x4`.
  **L1157 CN**: 开始定义函数或方法 `test_i64x2_extend_high_i32x4`。
- **L1158 EN**: Returns from the current function with `wasm_i64x2_extend_high_i32x4(a)`.
  **L1158 CN**: 以 `wasm_i64x2_extend_high_i32x4(a)` 从当前函数返回。
- **L1159 EN**: Closes the current lexical scope or compound statement.
  **L1159 CN**: 结束当前词法作用域或复合语句块。
- **L1160 EN**: Blank line separating nearby declarations or logic.
  **L1160 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1161 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_u64x2_extend_low_u32x4:`.
  **L1161 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_u64x2_extend_low_u32x4:`。
- **L1162 EN**: Comment documents nearby intent or constraints: `CHECK: i64x2.extend_low_i32x4_u{{$}}`.
  **L1162 CN**: 注释说明附近代码的意图或约束：`CHECK: i64x2.extend_low_i32x4_u{{$}}`。
- **L1163 EN**: Starts a function or method definition for `test_u64x2_extend_low_u32x4`.
  **L1163 CN**: 开始定义函数或方法 `test_u64x2_extend_low_u32x4`。
- **L1164 EN**: Returns from the current function with `wasm_u64x2_extend_low_u32x4(a)`.
  **L1164 CN**: 以 `wasm_u64x2_extend_low_u32x4(a)` 从当前函数返回。
- **L1165 EN**: Closes the current lexical scope or compound statement.
  **L1165 CN**: 结束当前词法作用域或复合语句块。
- **L1166 EN**: Blank line separating nearby declarations or logic.
  **L1166 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1167 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_u64x2_extend_high_u32x4:`.
  **L1167 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_u64x2_extend_high_u32x4:`。
- **L1168 EN**: Comment documents nearby intent or constraints: `CHECK: i64x2.extend_high_i32x4_u{{$}}`.
  **L1168 CN**: 注释说明附近代码的意图或约束：`CHECK: i64x2.extend_high_i32x4_u{{$}}`。
- **L1169 EN**: Starts a function or method definition for `test_u64x2_extend_high_u32x4`.
  **L1169 CN**: 开始定义函数或方法 `test_u64x2_extend_high_u32x4`。
- **L1170 EN**: Returns from the current function with `wasm_u64x2_extend_high_u32x4(a)`.
  **L1170 CN**: 以 `wasm_u64x2_extend_high_u32x4(a)` 从当前函数返回。
- **L1171 EN**: Closes the current lexical scope or compound statement.
  **L1171 CN**: 结束当前词法作用域或复合语句块。
- **L1172 EN**: Blank line separating nearby declarations or logic.
  **L1172 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1173 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i16x8_extadd_pairwise_i8x16:`.
  **L1173 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i16x8_extadd_pairwise_i8x16:`。
- **L1174 EN**: Comment documents nearby intent or constraints: `CHECK: i16x8.extadd_pairwise_i8x16_s{{$}}`.
  **L1174 CN**: 注释说明附近代码的意图或约束：`CHECK: i16x8.extadd_pairwise_i8x16_s{{$}}`。
- **L1175 EN**: Uses compile-time type-trait machinery to inspect or transform types.
  **L1175 CN**: 使用编译期类型萃取机制来检查或变换类型。
- **L1176 EN**: Returns from the current function with `wasm_i16x8_extadd_pairwise_i8x16(a)`.
  **L1176 CN**: 以 `wasm_i16x8_extadd_pairwise_i8x16(a)` 从当前函数返回。

### Lines 1177-1200

````c
}

// CHECK-LABEL: test_u16x8_extadd_pairwise_u8x16:
// CHECK: i16x8.extadd_pairwise_i8x16_u{{$}}
v128_t test_u16x8_extadd_pairwise_u8x16(v128_t a) {
  return wasm_u16x8_extadd_pairwise_u8x16(a);
}

// CHECK-LABEL: test_i32x4_extadd_pairwise_i16x8:
// CHECK: i32x4.extadd_pairwise_i16x8_s{{$}}
v128_t test_i32x4_extadd_pairwise_i16x8(v128_t a) {
  return wasm_i32x4_extadd_pairwise_i16x8(a);
}

// CHECK-LABEL: test_u32x4_extadd_pairwise_u16x8:
// CHECK: i32x4.extadd_pairwise_i16x8_u{{$}}
v128_t test_u32x4_extadd_pairwise_u16x8(v128_t a) {
  return wasm_u32x4_extadd_pairwise_u16x8(a);
}

// CHECK-LABEL: test_i16x8_extmul_low_i8x16:
// CHECK: i16x8.extmul_low_i8x16_s{{$}}
v128_t test_i16x8_extmul_low_i8x16(v128_t a, v128_t b) {
  return wasm_i16x8_extmul_low_i8x16(a, b);
````
- **L1177 EN**: Closes the current lexical scope or compound statement.
  **L1177 CN**: 结束当前词法作用域或复合语句块。
- **L1178 EN**: Blank line separating nearby declarations or logic.
  **L1178 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1179 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_u16x8_extadd_pairwise_u8x16:`.
  **L1179 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_u16x8_extadd_pairwise_u8x16:`。
- **L1180 EN**: Comment documents nearby intent or constraints: `CHECK: i16x8.extadd_pairwise_i8x16_u{{$}}`.
  **L1180 CN**: 注释说明附近代码的意图或约束：`CHECK: i16x8.extadd_pairwise_i8x16_u{{$}}`。
- **L1181 EN**: Uses compile-time type-trait machinery to inspect or transform types.
  **L1181 CN**: 使用编译期类型萃取机制来检查或变换类型。
- **L1182 EN**: Returns from the current function with `wasm_u16x8_extadd_pairwise_u8x16(a)`.
  **L1182 CN**: 以 `wasm_u16x8_extadd_pairwise_u8x16(a)` 从当前函数返回。
- **L1183 EN**: Closes the current lexical scope or compound statement.
  **L1183 CN**: 结束当前词法作用域或复合语句块。
- **L1184 EN**: Blank line separating nearby declarations or logic.
  **L1184 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1185 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i32x4_extadd_pairwise_i16x8:`.
  **L1185 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i32x4_extadd_pairwise_i16x8:`。
- **L1186 EN**: Comment documents nearby intent or constraints: `CHECK: i32x4.extadd_pairwise_i16x8_s{{$}}`.
  **L1186 CN**: 注释说明附近代码的意图或约束：`CHECK: i32x4.extadd_pairwise_i16x8_s{{$}}`。
- **L1187 EN**: Uses compile-time type-trait machinery to inspect or transform types.
  **L1187 CN**: 使用编译期类型萃取机制来检查或变换类型。
- **L1188 EN**: Returns from the current function with `wasm_i32x4_extadd_pairwise_i16x8(a)`.
  **L1188 CN**: 以 `wasm_i32x4_extadd_pairwise_i16x8(a)` 从当前函数返回。
- **L1189 EN**: Closes the current lexical scope or compound statement.
  **L1189 CN**: 结束当前词法作用域或复合语句块。
- **L1190 EN**: Blank line separating nearby declarations or logic.
  **L1190 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1191 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_u32x4_extadd_pairwise_u16x8:`.
  **L1191 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_u32x4_extadd_pairwise_u16x8:`。
- **L1192 EN**: Comment documents nearby intent or constraints: `CHECK: i32x4.extadd_pairwise_i16x8_u{{$}}`.
  **L1192 CN**: 注释说明附近代码的意图或约束：`CHECK: i32x4.extadd_pairwise_i16x8_u{{$}}`。
- **L1193 EN**: Uses compile-time type-trait machinery to inspect or transform types.
  **L1193 CN**: 使用编译期类型萃取机制来检查或变换类型。
- **L1194 EN**: Returns from the current function with `wasm_u32x4_extadd_pairwise_u16x8(a)`.
  **L1194 CN**: 以 `wasm_u32x4_extadd_pairwise_u16x8(a)` 从当前函数返回。
- **L1195 EN**: Closes the current lexical scope or compound statement.
  **L1195 CN**: 结束当前词法作用域或复合语句块。
- **L1196 EN**: Blank line separating nearby declarations or logic.
  **L1196 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1197 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i16x8_extmul_low_i8x16:`.
  **L1197 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i16x8_extmul_low_i8x16:`。
- **L1198 EN**: Comment documents nearby intent or constraints: `CHECK: i16x8.extmul_low_i8x16_s{{$}}`.
  **L1198 CN**: 注释说明附近代码的意图或约束：`CHECK: i16x8.extmul_low_i8x16_s{{$}}`。
- **L1199 EN**: Starts a function or method definition for `test_i16x8_extmul_low_i8x16`.
  **L1199 CN**: 开始定义函数或方法 `test_i16x8_extmul_low_i8x16`。
- **L1200 EN**: Returns from the current function with `wasm_i16x8_extmul_low_i8x16(a, b)`.
  **L1200 CN**: 以 `wasm_i16x8_extmul_low_i8x16(a, b)` 从当前函数返回。

### Lines 1201-1224

````c
}

// CHECK-LABEL: test_i16x8_extmul_high_i8x16:
// CHECK: i16x8.extmul_high_i8x16_s{{$}}
v128_t test_i16x8_extmul_high_i8x16(v128_t a, v128_t b) {
  return wasm_i16x8_extmul_high_i8x16(a, b);
}

// CHECK-LABEL: test_u16x8_extmul_low_u8x16:
// CHECK: i16x8.extmul_low_i8x16_u{{$}}
v128_t test_u16x8_extmul_low_u8x16(v128_t a, v128_t b) {
  return wasm_u16x8_extmul_low_u8x16(a, b);
}

// CHECK-LABEL: test_u16x8_extmul_high_u8x16:
// CHECK: i16x8.extmul_high_i8x16_u{{$}}
v128_t test_u16x8_extmul_high_u8x16(v128_t a, v128_t b) {
  return wasm_u16x8_extmul_high_u8x16(a, b);
}

// CHECK-LABEL: test_i32x4_extmul_low_i16x8:
// CHECK: i32x4.extmul_low_i16x8_s{{$}}
v128_t test_i32x4_extmul_low_i16x8(v128_t a, v128_t b) {
  return wasm_i32x4_extmul_low_i16x8(a, b);
````
- **L1201 EN**: Closes the current lexical scope or compound statement.
  **L1201 CN**: 结束当前词法作用域或复合语句块。
- **L1202 EN**: Blank line separating nearby declarations or logic.
  **L1202 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1203 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i16x8_extmul_high_i8x16:`.
  **L1203 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i16x8_extmul_high_i8x16:`。
- **L1204 EN**: Comment documents nearby intent or constraints: `CHECK: i16x8.extmul_high_i8x16_s{{$}}`.
  **L1204 CN**: 注释说明附近代码的意图或约束：`CHECK: i16x8.extmul_high_i8x16_s{{$}}`。
- **L1205 EN**: Starts a function or method definition for `test_i16x8_extmul_high_i8x16`.
  **L1205 CN**: 开始定义函数或方法 `test_i16x8_extmul_high_i8x16`。
- **L1206 EN**: Returns from the current function with `wasm_i16x8_extmul_high_i8x16(a, b)`.
  **L1206 CN**: 以 `wasm_i16x8_extmul_high_i8x16(a, b)` 从当前函数返回。
- **L1207 EN**: Closes the current lexical scope or compound statement.
  **L1207 CN**: 结束当前词法作用域或复合语句块。
- **L1208 EN**: Blank line separating nearby declarations or logic.
  **L1208 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1209 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_u16x8_extmul_low_u8x16:`.
  **L1209 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_u16x8_extmul_low_u8x16:`。
- **L1210 EN**: Comment documents nearby intent or constraints: `CHECK: i16x8.extmul_low_i8x16_u{{$}}`.
  **L1210 CN**: 注释说明附近代码的意图或约束：`CHECK: i16x8.extmul_low_i8x16_u{{$}}`。
- **L1211 EN**: Starts a function or method definition for `test_u16x8_extmul_low_u8x16`.
  **L1211 CN**: 开始定义函数或方法 `test_u16x8_extmul_low_u8x16`。
- **L1212 EN**: Returns from the current function with `wasm_u16x8_extmul_low_u8x16(a, b)`.
  **L1212 CN**: 以 `wasm_u16x8_extmul_low_u8x16(a, b)` 从当前函数返回。
- **L1213 EN**: Closes the current lexical scope or compound statement.
  **L1213 CN**: 结束当前词法作用域或复合语句块。
- **L1214 EN**: Blank line separating nearby declarations or logic.
  **L1214 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1215 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_u16x8_extmul_high_u8x16:`.
  **L1215 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_u16x8_extmul_high_u8x16:`。
- **L1216 EN**: Comment documents nearby intent or constraints: `CHECK: i16x8.extmul_high_i8x16_u{{$}}`.
  **L1216 CN**: 注释说明附近代码的意图或约束：`CHECK: i16x8.extmul_high_i8x16_u{{$}}`。
- **L1217 EN**: Starts a function or method definition for `test_u16x8_extmul_high_u8x16`.
  **L1217 CN**: 开始定义函数或方法 `test_u16x8_extmul_high_u8x16`。
- **L1218 EN**: Returns from the current function with `wasm_u16x8_extmul_high_u8x16(a, b)`.
  **L1218 CN**: 以 `wasm_u16x8_extmul_high_u8x16(a, b)` 从当前函数返回。
- **L1219 EN**: Closes the current lexical scope or compound statement.
  **L1219 CN**: 结束当前词法作用域或复合语句块。
- **L1220 EN**: Blank line separating nearby declarations or logic.
  **L1220 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1221 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i32x4_extmul_low_i16x8:`.
  **L1221 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i32x4_extmul_low_i16x8:`。
- **L1222 EN**: Comment documents nearby intent or constraints: `CHECK: i32x4.extmul_low_i16x8_s{{$}}`.
  **L1222 CN**: 注释说明附近代码的意图或约束：`CHECK: i32x4.extmul_low_i16x8_s{{$}}`。
- **L1223 EN**: Starts a function or method definition for `test_i32x4_extmul_low_i16x8`.
  **L1223 CN**: 开始定义函数或方法 `test_i32x4_extmul_low_i16x8`。
- **L1224 EN**: Returns from the current function with `wasm_i32x4_extmul_low_i16x8(a, b)`.
  **L1224 CN**: 以 `wasm_i32x4_extmul_low_i16x8(a, b)` 从当前函数返回。

### Lines 1225-1248

````c
}

// CHECK-LABEL: test_i32x4_extmul_high_i16x8:
// CHECK: i32x4.extmul_high_i16x8_s{{$}}
v128_t test_i32x4_extmul_high_i16x8(v128_t a, v128_t b) {
  return wasm_i32x4_extmul_high_i16x8(a, b);
}

// CHECK-LABEL: test_u32x4_extmul_low_u16x8:
// CHECK: i32x4.extmul_low_i16x8_u{{$}}
v128_t test_u32x4_extmul_low_u16x8(v128_t a, v128_t b) {
  return wasm_u32x4_extmul_low_u16x8(a, b);
}

// CHECK-LABEL: test_u32x4_extmul_high_u16x8:
// CHECK: i32x4.extmul_high_i16x8_u{{$}}
v128_t test_u32x4_extmul_high_u16x8(v128_t a, v128_t b) {
  return wasm_u32x4_extmul_high_u16x8(a, b);
}

// CHECK-LABEL: test_i64x2_extmul_low_i32x4:
// CHECK: i64x2.extmul_low_i32x4_s{{$}}
v128_t test_i64x2_extmul_low_i32x4(v128_t a, v128_t b) {
  return wasm_i64x2_extmul_low_i32x4(a, b);
````
- **L1225 EN**: Closes the current lexical scope or compound statement.
  **L1225 CN**: 结束当前词法作用域或复合语句块。
- **L1226 EN**: Blank line separating nearby declarations or logic.
  **L1226 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1227 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i32x4_extmul_high_i16x8:`.
  **L1227 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i32x4_extmul_high_i16x8:`。
- **L1228 EN**: Comment documents nearby intent or constraints: `CHECK: i32x4.extmul_high_i16x8_s{{$}}`.
  **L1228 CN**: 注释说明附近代码的意图或约束：`CHECK: i32x4.extmul_high_i16x8_s{{$}}`。
- **L1229 EN**: Starts a function or method definition for `test_i32x4_extmul_high_i16x8`.
  **L1229 CN**: 开始定义函数或方法 `test_i32x4_extmul_high_i16x8`。
- **L1230 EN**: Returns from the current function with `wasm_i32x4_extmul_high_i16x8(a, b)`.
  **L1230 CN**: 以 `wasm_i32x4_extmul_high_i16x8(a, b)` 从当前函数返回。
- **L1231 EN**: Closes the current lexical scope or compound statement.
  **L1231 CN**: 结束当前词法作用域或复合语句块。
- **L1232 EN**: Blank line separating nearby declarations or logic.
  **L1232 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1233 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_u32x4_extmul_low_u16x8:`.
  **L1233 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_u32x4_extmul_low_u16x8:`。
- **L1234 EN**: Comment documents nearby intent or constraints: `CHECK: i32x4.extmul_low_i16x8_u{{$}}`.
  **L1234 CN**: 注释说明附近代码的意图或约束：`CHECK: i32x4.extmul_low_i16x8_u{{$}}`。
- **L1235 EN**: Starts a function or method definition for `test_u32x4_extmul_low_u16x8`.
  **L1235 CN**: 开始定义函数或方法 `test_u32x4_extmul_low_u16x8`。
- **L1236 EN**: Returns from the current function with `wasm_u32x4_extmul_low_u16x8(a, b)`.
  **L1236 CN**: 以 `wasm_u32x4_extmul_low_u16x8(a, b)` 从当前函数返回。
- **L1237 EN**: Closes the current lexical scope or compound statement.
  **L1237 CN**: 结束当前词法作用域或复合语句块。
- **L1238 EN**: Blank line separating nearby declarations or logic.
  **L1238 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1239 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_u32x4_extmul_high_u16x8:`.
  **L1239 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_u32x4_extmul_high_u16x8:`。
- **L1240 EN**: Comment documents nearby intent or constraints: `CHECK: i32x4.extmul_high_i16x8_u{{$}}`.
  **L1240 CN**: 注释说明附近代码的意图或约束：`CHECK: i32x4.extmul_high_i16x8_u{{$}}`。
- **L1241 EN**: Starts a function or method definition for `test_u32x4_extmul_high_u16x8`.
  **L1241 CN**: 开始定义函数或方法 `test_u32x4_extmul_high_u16x8`。
- **L1242 EN**: Returns from the current function with `wasm_u32x4_extmul_high_u16x8(a, b)`.
  **L1242 CN**: 以 `wasm_u32x4_extmul_high_u16x8(a, b)` 从当前函数返回。
- **L1243 EN**: Closes the current lexical scope or compound statement.
  **L1243 CN**: 结束当前词法作用域或复合语句块。
- **L1244 EN**: Blank line separating nearby declarations or logic.
  **L1244 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1245 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i64x2_extmul_low_i32x4:`.
  **L1245 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i64x2_extmul_low_i32x4:`。
- **L1246 EN**: Comment documents nearby intent or constraints: `CHECK: i64x2.extmul_low_i32x4_s{{$}}`.
  **L1246 CN**: 注释说明附近代码的意图或约束：`CHECK: i64x2.extmul_low_i32x4_s{{$}}`。
- **L1247 EN**: Starts a function or method definition for `test_i64x2_extmul_low_i32x4`.
  **L1247 CN**: 开始定义函数或方法 `test_i64x2_extmul_low_i32x4`。
- **L1248 EN**: Returns from the current function with `wasm_i64x2_extmul_low_i32x4(a, b)`.
  **L1248 CN**: 以 `wasm_i64x2_extmul_low_i32x4(a, b)` 从当前函数返回。

### Lines 1249-1272

````c
}

// CHECK-LABEL: test_i64x2_extmul_high_i32x4:
// CHECK: i64x2.extmul_high_i32x4_s{{$}}
v128_t test_i64x2_extmul_high_i32x4(v128_t a, v128_t b) {
  return wasm_i64x2_extmul_high_i32x4(a, b);
}

// CHECK-LABEL: test_u64x2_extmul_low_u32x4:
// CHECK: i64x2.extmul_low_i32x4_u{{$}}
v128_t test_u64x2_extmul_low_u32x4(v128_t a, v128_t b) {
  return wasm_u64x2_extmul_low_u32x4(a, b);
}

// CHECK-LABEL: test_u64x2_extmul_high_u32x4:
// CHECK: i64x2.extmul_high_i32x4_u{{$}}
v128_t test_u64x2_extmul_high_u32x4(v128_t a, v128_t b) {
  return wasm_u64x2_extmul_high_u32x4(a, b);
}

// CHECK-LABEL: test_i16x8_q15mulr_sat:
// CHECK: i16x8.q15mulr_sat_s{{$}}
v128_t test_i16x8_q15mulr_sat(v128_t a, v128_t b) {
  return wasm_i16x8_q15mulr_sat(a, b);
````
- **L1249 EN**: Closes the current lexical scope or compound statement.
  **L1249 CN**: 结束当前词法作用域或复合语句块。
- **L1250 EN**: Blank line separating nearby declarations or logic.
  **L1250 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1251 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i64x2_extmul_high_i32x4:`.
  **L1251 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i64x2_extmul_high_i32x4:`。
- **L1252 EN**: Comment documents nearby intent or constraints: `CHECK: i64x2.extmul_high_i32x4_s{{$}}`.
  **L1252 CN**: 注释说明附近代码的意图或约束：`CHECK: i64x2.extmul_high_i32x4_s{{$}}`。
- **L1253 EN**: Starts a function or method definition for `test_i64x2_extmul_high_i32x4`.
  **L1253 CN**: 开始定义函数或方法 `test_i64x2_extmul_high_i32x4`。
- **L1254 EN**: Returns from the current function with `wasm_i64x2_extmul_high_i32x4(a, b)`.
  **L1254 CN**: 以 `wasm_i64x2_extmul_high_i32x4(a, b)` 从当前函数返回。
- **L1255 EN**: Closes the current lexical scope or compound statement.
  **L1255 CN**: 结束当前词法作用域或复合语句块。
- **L1256 EN**: Blank line separating nearby declarations or logic.
  **L1256 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1257 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_u64x2_extmul_low_u32x4:`.
  **L1257 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_u64x2_extmul_low_u32x4:`。
- **L1258 EN**: Comment documents nearby intent or constraints: `CHECK: i64x2.extmul_low_i32x4_u{{$}}`.
  **L1258 CN**: 注释说明附近代码的意图或约束：`CHECK: i64x2.extmul_low_i32x4_u{{$}}`。
- **L1259 EN**: Starts a function or method definition for `test_u64x2_extmul_low_u32x4`.
  **L1259 CN**: 开始定义函数或方法 `test_u64x2_extmul_low_u32x4`。
- **L1260 EN**: Returns from the current function with `wasm_u64x2_extmul_low_u32x4(a, b)`.
  **L1260 CN**: 以 `wasm_u64x2_extmul_low_u32x4(a, b)` 从当前函数返回。
- **L1261 EN**: Closes the current lexical scope or compound statement.
  **L1261 CN**: 结束当前词法作用域或复合语句块。
- **L1262 EN**: Blank line separating nearby declarations or logic.
  **L1262 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1263 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_u64x2_extmul_high_u32x4:`.
  **L1263 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_u64x2_extmul_high_u32x4:`。
- **L1264 EN**: Comment documents nearby intent or constraints: `CHECK: i64x2.extmul_high_i32x4_u{{$}}`.
  **L1264 CN**: 注释说明附近代码的意图或约束：`CHECK: i64x2.extmul_high_i32x4_u{{$}}`。
- **L1265 EN**: Starts a function or method definition for `test_u64x2_extmul_high_u32x4`.
  **L1265 CN**: 开始定义函数或方法 `test_u64x2_extmul_high_u32x4`。
- **L1266 EN**: Returns from the current function with `wasm_u64x2_extmul_high_u32x4(a, b)`.
  **L1266 CN**: 以 `wasm_u64x2_extmul_high_u32x4(a, b)` 从当前函数返回。
- **L1267 EN**: Closes the current lexical scope or compound statement.
  **L1267 CN**: 结束当前词法作用域或复合语句块。
- **L1268 EN**: Blank line separating nearby declarations or logic.
  **L1268 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1269 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i16x8_q15mulr_sat:`.
  **L1269 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i16x8_q15mulr_sat:`。
- **L1270 EN**: Comment documents nearby intent or constraints: `CHECK: i16x8.q15mulr_sat_s{{$}}`.
  **L1270 CN**: 注释说明附近代码的意图或约束：`CHECK: i16x8.q15mulr_sat_s{{$}}`。
- **L1271 EN**: Starts a function or method definition for `test_i16x8_q15mulr_sat`.
  **L1271 CN**: 开始定义函数或方法 `test_i16x8_q15mulr_sat`。
- **L1272 EN**: Returns from the current function with `wasm_i16x8_q15mulr_sat(a, b)`.
  **L1272 CN**: 以 `wasm_i16x8_q15mulr_sat(a, b)` 从当前函数返回。

### Lines 1273-1296

````c
}

// CHECK-LABEL: test_f32x4_relaxed_madd:
// CHECK: f32x4.relaxed_madd{{$}}
v128_t test_f32x4_relaxed_madd(v128_t a, v128_t b, v128_t c) {
  return wasm_f32x4_relaxed_madd(a, b, c);
}

// CHECK-LABEL: test_f32x4_relaxed_nmadd:
// CHECK: f32x4.relaxed_nmadd{{$}}
v128_t test_f32x4_relaxed_nmadd(v128_t a, v128_t b, v128_t c) {
  return wasm_f32x4_relaxed_nmadd(a, b, c);
}

// CHECK-LABEL: test_f64x2_relaxed_madd:
// CHECK: f64x2.relaxed_madd{{$}}
v128_t test_f64x2_relaxed_madd(v128_t a, v128_t b, v128_t c) {
  return wasm_f64x2_relaxed_madd(a, b, c);
}

// CHECK-LABEL: test_f64x2_relaxed_nmadd:
// CHECK: f64x2.relaxed_nmadd{{$}}
v128_t test_f64x2_relaxed_nmadd(v128_t a, v128_t b, v128_t c) {
  return wasm_f64x2_relaxed_nmadd(a, b, c);
````
- **L1273 EN**: Closes the current lexical scope or compound statement.
  **L1273 CN**: 结束当前词法作用域或复合语句块。
- **L1274 EN**: Blank line separating nearby declarations or logic.
  **L1274 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1275 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f32x4_relaxed_madd:`.
  **L1275 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f32x4_relaxed_madd:`。
- **L1276 EN**: Comment documents nearby intent or constraints: `CHECK: f32x4.relaxed_madd{{$}}`.
  **L1276 CN**: 注释说明附近代码的意图或约束：`CHECK: f32x4.relaxed_madd{{$}}`。
- **L1277 EN**: Starts a function or method definition for `test_f32x4_relaxed_madd`.
  **L1277 CN**: 开始定义函数或方法 `test_f32x4_relaxed_madd`。
- **L1278 EN**: Returns from the current function with `wasm_f32x4_relaxed_madd(a, b, c)`.
  **L1278 CN**: 以 `wasm_f32x4_relaxed_madd(a, b, c)` 从当前函数返回。
- **L1279 EN**: Closes the current lexical scope or compound statement.
  **L1279 CN**: 结束当前词法作用域或复合语句块。
- **L1280 EN**: Blank line separating nearby declarations or logic.
  **L1280 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1281 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f32x4_relaxed_nmadd:`.
  **L1281 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f32x4_relaxed_nmadd:`。
- **L1282 EN**: Comment documents nearby intent or constraints: `CHECK: f32x4.relaxed_nmadd{{$}}`.
  **L1282 CN**: 注释说明附近代码的意图或约束：`CHECK: f32x4.relaxed_nmadd{{$}}`。
- **L1283 EN**: Starts a function or method definition for `test_f32x4_relaxed_nmadd`.
  **L1283 CN**: 开始定义函数或方法 `test_f32x4_relaxed_nmadd`。
- **L1284 EN**: Returns from the current function with `wasm_f32x4_relaxed_nmadd(a, b, c)`.
  **L1284 CN**: 以 `wasm_f32x4_relaxed_nmadd(a, b, c)` 从当前函数返回。
- **L1285 EN**: Closes the current lexical scope or compound statement.
  **L1285 CN**: 结束当前词法作用域或复合语句块。
- **L1286 EN**: Blank line separating nearby declarations or logic.
  **L1286 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1287 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f64x2_relaxed_madd:`.
  **L1287 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f64x2_relaxed_madd:`。
- **L1288 EN**: Comment documents nearby intent or constraints: `CHECK: f64x2.relaxed_madd{{$}}`.
  **L1288 CN**: 注释说明附近代码的意图或约束：`CHECK: f64x2.relaxed_madd{{$}}`。
- **L1289 EN**: Starts a function or method definition for `test_f64x2_relaxed_madd`.
  **L1289 CN**: 开始定义函数或方法 `test_f64x2_relaxed_madd`。
- **L1290 EN**: Returns from the current function with `wasm_f64x2_relaxed_madd(a, b, c)`.
  **L1290 CN**: 以 `wasm_f64x2_relaxed_madd(a, b, c)` 从当前函数返回。
- **L1291 EN**: Closes the current lexical scope or compound statement.
  **L1291 CN**: 结束当前词法作用域或复合语句块。
- **L1292 EN**: Blank line separating nearby declarations or logic.
  **L1292 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1293 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f64x2_relaxed_nmadd:`.
  **L1293 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f64x2_relaxed_nmadd:`。
- **L1294 EN**: Comment documents nearby intent or constraints: `CHECK: f64x2.relaxed_nmadd{{$}}`.
  **L1294 CN**: 注释说明附近代码的意图或约束：`CHECK: f64x2.relaxed_nmadd{{$}}`。
- **L1295 EN**: Starts a function or method definition for `test_f64x2_relaxed_nmadd`.
  **L1295 CN**: 开始定义函数或方法 `test_f64x2_relaxed_nmadd`。
- **L1296 EN**: Returns from the current function with `wasm_f64x2_relaxed_nmadd(a, b, c)`.
  **L1296 CN**: 以 `wasm_f64x2_relaxed_nmadd(a, b, c)` 从当前函数返回。

### Lines 1297-1320

````c
}

// CHECK-LABEL: test_i8x16_relaxed_laneselect:
// CHECK: i8x16.relaxed_laneselect{{$}}
v128_t test_i8x16_relaxed_laneselect(v128_t a, v128_t b, v128_t m) {
  return wasm_i8x16_relaxed_laneselect(a, b, m);
}

// CHECK-LABEL: test_i16x8_relaxed_laneselect:
// CHECK: i16x8.relaxed_laneselect{{$}}
v128_t test_i16x8_relaxed_laneselect(v128_t a, v128_t b, v128_t m) {
  return wasm_i16x8_relaxed_laneselect(a, b, m);
}

// CHECK-LABEL: test_i32x4_relaxed_laneselect:
// CHECK: i32x4.relaxed_laneselect{{$}}
v128_t test_i32x4_relaxed_laneselect(v128_t a, v128_t b, v128_t m) {
  return wasm_i32x4_relaxed_laneselect(a, b, m);
}

// CHECK-LABEL: test_i64x2_relaxed_laneselect:
// CHECK: i64x2.relaxed_laneselect{{$}}
v128_t test_i64x2_relaxed_laneselect(v128_t a, v128_t b, v128_t m) {
  return wasm_i64x2_relaxed_laneselect(a, b, m);
````
- **L1297 EN**: Closes the current lexical scope or compound statement.
  **L1297 CN**: 结束当前词法作用域或复合语句块。
- **L1298 EN**: Blank line separating nearby declarations or logic.
  **L1298 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1299 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i8x16_relaxed_laneselect:`.
  **L1299 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i8x16_relaxed_laneselect:`。
- **L1300 EN**: Comment documents nearby intent or constraints: `CHECK: i8x16.relaxed_laneselect{{$}}`.
  **L1300 CN**: 注释说明附近代码的意图或约束：`CHECK: i8x16.relaxed_laneselect{{$}}`。
- **L1301 EN**: Starts a function or method definition for `test_i8x16_relaxed_laneselect`.
  **L1301 CN**: 开始定义函数或方法 `test_i8x16_relaxed_laneselect`。
- **L1302 EN**: Returns from the current function with `wasm_i8x16_relaxed_laneselect(a, b, m)`.
  **L1302 CN**: 以 `wasm_i8x16_relaxed_laneselect(a, b, m)` 从当前函数返回。
- **L1303 EN**: Closes the current lexical scope or compound statement.
  **L1303 CN**: 结束当前词法作用域或复合语句块。
- **L1304 EN**: Blank line separating nearby declarations or logic.
  **L1304 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1305 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i16x8_relaxed_laneselect:`.
  **L1305 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i16x8_relaxed_laneselect:`。
- **L1306 EN**: Comment documents nearby intent or constraints: `CHECK: i16x8.relaxed_laneselect{{$}}`.
  **L1306 CN**: 注释说明附近代码的意图或约束：`CHECK: i16x8.relaxed_laneselect{{$}}`。
- **L1307 EN**: Starts a function or method definition for `test_i16x8_relaxed_laneselect`.
  **L1307 CN**: 开始定义函数或方法 `test_i16x8_relaxed_laneselect`。
- **L1308 EN**: Returns from the current function with `wasm_i16x8_relaxed_laneselect(a, b, m)`.
  **L1308 CN**: 以 `wasm_i16x8_relaxed_laneselect(a, b, m)` 从当前函数返回。
- **L1309 EN**: Closes the current lexical scope or compound statement.
  **L1309 CN**: 结束当前词法作用域或复合语句块。
- **L1310 EN**: Blank line separating nearby declarations or logic.
  **L1310 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1311 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i32x4_relaxed_laneselect:`.
  **L1311 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i32x4_relaxed_laneselect:`。
- **L1312 EN**: Comment documents nearby intent or constraints: `CHECK: i32x4.relaxed_laneselect{{$}}`.
  **L1312 CN**: 注释说明附近代码的意图或约束：`CHECK: i32x4.relaxed_laneselect{{$}}`。
- **L1313 EN**: Starts a function or method definition for `test_i32x4_relaxed_laneselect`.
  **L1313 CN**: 开始定义函数或方法 `test_i32x4_relaxed_laneselect`。
- **L1314 EN**: Returns from the current function with `wasm_i32x4_relaxed_laneselect(a, b, m)`.
  **L1314 CN**: 以 `wasm_i32x4_relaxed_laneselect(a, b, m)` 从当前函数返回。
- **L1315 EN**: Closes the current lexical scope or compound statement.
  **L1315 CN**: 结束当前词法作用域或复合语句块。
- **L1316 EN**: Blank line separating nearby declarations or logic.
  **L1316 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1317 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i64x2_relaxed_laneselect:`.
  **L1317 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i64x2_relaxed_laneselect:`。
- **L1318 EN**: Comment documents nearby intent or constraints: `CHECK: i64x2.relaxed_laneselect{{$}}`.
  **L1318 CN**: 注释说明附近代码的意图或约束：`CHECK: i64x2.relaxed_laneselect{{$}}`。
- **L1319 EN**: Starts a function or method definition for `test_i64x2_relaxed_laneselect`.
  **L1319 CN**: 开始定义函数或方法 `test_i64x2_relaxed_laneselect`。
- **L1320 EN**: Returns from the current function with `wasm_i64x2_relaxed_laneselect(a, b, m)`.
  **L1320 CN**: 以 `wasm_i64x2_relaxed_laneselect(a, b, m)` 从当前函数返回。

### Lines 1321-1344

````c
}

// CHECK-LABEL: test_i8x16_relaxed_swizzle:
// CHECK: i8x16.relaxed_swizzle{{$}}
v128_t test_i8x16_relaxed_swizzle(v128_t a, v128_t s) {
  return wasm_i8x16_relaxed_swizzle(a, s);
}

// CHECK-LABEL: test_f32x4_relaxed_min:
// CHECK: f32x4.relaxed_min{{$}}
v128_t test_f32x4_relaxed_min(v128_t a, v128_t b) {
  return wasm_f32x4_relaxed_min(a, b);
}

// CHECK-LABEL: test_f32x4_relaxed_max:
// CHECK: f32x4.relaxed_max{{$}}
v128_t test_f32x4_relaxed_max(v128_t a, v128_t b) {
  return wasm_f32x4_relaxed_max(a, b);
}

// CHECK-LABEL: test_f64x2_relaxed_min:
// CHECK: f64x2.relaxed_min{{$}}
v128_t test_f64x2_relaxed_min(v128_t a, v128_t b) {
  return wasm_f64x2_relaxed_min(a, b);
````
- **L1321 EN**: Closes the current lexical scope or compound statement.
  **L1321 CN**: 结束当前词法作用域或复合语句块。
- **L1322 EN**: Blank line separating nearby declarations or logic.
  **L1322 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1323 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i8x16_relaxed_swizzle:`.
  **L1323 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i8x16_relaxed_swizzle:`。
- **L1324 EN**: Comment documents nearby intent or constraints: `CHECK: i8x16.relaxed_swizzle{{$}}`.
  **L1324 CN**: 注释说明附近代码的意图或约束：`CHECK: i8x16.relaxed_swizzle{{$}}`。
- **L1325 EN**: Starts a function or method definition for `test_i8x16_relaxed_swizzle`.
  **L1325 CN**: 开始定义函数或方法 `test_i8x16_relaxed_swizzle`。
- **L1326 EN**: Returns from the current function with `wasm_i8x16_relaxed_swizzle(a, s)`.
  **L1326 CN**: 以 `wasm_i8x16_relaxed_swizzle(a, s)` 从当前函数返回。
- **L1327 EN**: Closes the current lexical scope or compound statement.
  **L1327 CN**: 结束当前词法作用域或复合语句块。
- **L1328 EN**: Blank line separating nearby declarations or logic.
  **L1328 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1329 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f32x4_relaxed_min:`.
  **L1329 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f32x4_relaxed_min:`。
- **L1330 EN**: Comment documents nearby intent or constraints: `CHECK: f32x4.relaxed_min{{$}}`.
  **L1330 CN**: 注释说明附近代码的意图或约束：`CHECK: f32x4.relaxed_min{{$}}`。
- **L1331 EN**: Starts a function or method definition for `test_f32x4_relaxed_min`.
  **L1331 CN**: 开始定义函数或方法 `test_f32x4_relaxed_min`。
- **L1332 EN**: Returns from the current function with `wasm_f32x4_relaxed_min(a, b)`.
  **L1332 CN**: 以 `wasm_f32x4_relaxed_min(a, b)` 从当前函数返回。
- **L1333 EN**: Closes the current lexical scope or compound statement.
  **L1333 CN**: 结束当前词法作用域或复合语句块。
- **L1334 EN**: Blank line separating nearby declarations or logic.
  **L1334 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1335 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f32x4_relaxed_max:`.
  **L1335 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f32x4_relaxed_max:`。
- **L1336 EN**: Comment documents nearby intent or constraints: `CHECK: f32x4.relaxed_max{{$}}`.
  **L1336 CN**: 注释说明附近代码的意图或约束：`CHECK: f32x4.relaxed_max{{$}}`。
- **L1337 EN**: Starts a function or method definition for `test_f32x4_relaxed_max`.
  **L1337 CN**: 开始定义函数或方法 `test_f32x4_relaxed_max`。
- **L1338 EN**: Returns from the current function with `wasm_f32x4_relaxed_max(a, b)`.
  **L1338 CN**: 以 `wasm_f32x4_relaxed_max(a, b)` 从当前函数返回。
- **L1339 EN**: Closes the current lexical scope or compound statement.
  **L1339 CN**: 结束当前词法作用域或复合语句块。
- **L1340 EN**: Blank line separating nearby declarations or logic.
  **L1340 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1341 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f64x2_relaxed_min:`.
  **L1341 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f64x2_relaxed_min:`。
- **L1342 EN**: Comment documents nearby intent or constraints: `CHECK: f64x2.relaxed_min{{$}}`.
  **L1342 CN**: 注释说明附近代码的意图或约束：`CHECK: f64x2.relaxed_min{{$}}`。
- **L1343 EN**: Starts a function or method definition for `test_f64x2_relaxed_min`.
  **L1343 CN**: 开始定义函数或方法 `test_f64x2_relaxed_min`。
- **L1344 EN**: Returns from the current function with `wasm_f64x2_relaxed_min(a, b)`.
  **L1344 CN**: 以 `wasm_f64x2_relaxed_min(a, b)` 从当前函数返回。

### Lines 1345-1368

````c
}

// CHECK-LABEL: test_f64x2_relaxed_max:
// CHECK: f64x2.relaxed_max
v128_t test_f64x2_relaxed_max(v128_t a, v128_t b) {
  return wasm_f64x2_relaxed_max(a, b);
}

// CHECK-LABEL: test_i32x4_relaxed_trunc_f32x4:
// CHECK: i32x4.relaxed_trunc_f32x4_s{{$}}
v128_t test_i32x4_relaxed_trunc_f32x4(v128_t a) {
  return wasm_i32x4_relaxed_trunc_f32x4(a);
}

// CHECK-LABEL: test_u32x4_relaxed_trunc_f32x4:
// CHECK: i32x4.relaxed_trunc_f32x4_u{{$}}
v128_t test_u32x4_relaxed_trunc_f32x4(v128_t a) {
  return wasm_u32x4_relaxed_trunc_f32x4(a);
}

// CHECK-LABEL: test_i32x4_relaxed_trunc_f64x2_zero:
// CHECK: i32x4.relaxed_trunc_f64x2_s_zero{{$}}
v128_t test_i32x4_relaxed_trunc_f64x2_zero(v128_t a) {
  return wasm_i32x4_relaxed_trunc_f64x2_zero(a);
````
- **L1345 EN**: Closes the current lexical scope or compound statement.
  **L1345 CN**: 结束当前词法作用域或复合语句块。
- **L1346 EN**: Blank line separating nearby declarations or logic.
  **L1346 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1347 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f64x2_relaxed_max:`.
  **L1347 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f64x2_relaxed_max:`。
- **L1348 EN**: Comment documents nearby intent or constraints: `CHECK: f64x2.relaxed_max`.
  **L1348 CN**: 注释说明附近代码的意图或约束：`CHECK: f64x2.relaxed_max`。
- **L1349 EN**: Starts a function or method definition for `test_f64x2_relaxed_max`.
  **L1349 CN**: 开始定义函数或方法 `test_f64x2_relaxed_max`。
- **L1350 EN**: Returns from the current function with `wasm_f64x2_relaxed_max(a, b)`.
  **L1350 CN**: 以 `wasm_f64x2_relaxed_max(a, b)` 从当前函数返回。
- **L1351 EN**: Closes the current lexical scope or compound statement.
  **L1351 CN**: 结束当前词法作用域或复合语句块。
- **L1352 EN**: Blank line separating nearby declarations or logic.
  **L1352 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1353 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i32x4_relaxed_trunc_f32x4:`.
  **L1353 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i32x4_relaxed_trunc_f32x4:`。
- **L1354 EN**: Comment documents nearby intent or constraints: `CHECK: i32x4.relaxed_trunc_f32x4_s{{$}}`.
  **L1354 CN**: 注释说明附近代码的意图或约束：`CHECK: i32x4.relaxed_trunc_f32x4_s{{$}}`。
- **L1355 EN**: Starts a function or method definition for `test_i32x4_relaxed_trunc_f32x4`.
  **L1355 CN**: 开始定义函数或方法 `test_i32x4_relaxed_trunc_f32x4`。
- **L1356 EN**: Returns from the current function with `wasm_i32x4_relaxed_trunc_f32x4(a)`.
  **L1356 CN**: 以 `wasm_i32x4_relaxed_trunc_f32x4(a)` 从当前函数返回。
- **L1357 EN**: Closes the current lexical scope or compound statement.
  **L1357 CN**: 结束当前词法作用域或复合语句块。
- **L1358 EN**: Blank line separating nearby declarations or logic.
  **L1358 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1359 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_u32x4_relaxed_trunc_f32x4:`.
  **L1359 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_u32x4_relaxed_trunc_f32x4:`。
- **L1360 EN**: Comment documents nearby intent or constraints: `CHECK: i32x4.relaxed_trunc_f32x4_u{{$}}`.
  **L1360 CN**: 注释说明附近代码的意图或约束：`CHECK: i32x4.relaxed_trunc_f32x4_u{{$}}`。
- **L1361 EN**: Starts a function or method definition for `test_u32x4_relaxed_trunc_f32x4`.
  **L1361 CN**: 开始定义函数或方法 `test_u32x4_relaxed_trunc_f32x4`。
- **L1362 EN**: Returns from the current function with `wasm_u32x4_relaxed_trunc_f32x4(a)`.
  **L1362 CN**: 以 `wasm_u32x4_relaxed_trunc_f32x4(a)` 从当前函数返回。
- **L1363 EN**: Closes the current lexical scope or compound statement.
  **L1363 CN**: 结束当前词法作用域或复合语句块。
- **L1364 EN**: Blank line separating nearby declarations or logic.
  **L1364 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1365 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i32x4_relaxed_trunc_f64x2_zero:`.
  **L1365 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i32x4_relaxed_trunc_f64x2_zero:`。
- **L1366 EN**: Comment documents nearby intent or constraints: `CHECK: i32x4.relaxed_trunc_f64x2_s_zero{{$}}`.
  **L1366 CN**: 注释说明附近代码的意图或约束：`CHECK: i32x4.relaxed_trunc_f64x2_s_zero{{$}}`。
- **L1367 EN**: Starts a function or method definition for `test_i32x4_relaxed_trunc_f64x2_zero`.
  **L1367 CN**: 开始定义函数或方法 `test_i32x4_relaxed_trunc_f64x2_zero`。
- **L1368 EN**: Returns from the current function with `wasm_i32x4_relaxed_trunc_f64x2_zero(a)`.
  **L1368 CN**: 以 `wasm_i32x4_relaxed_trunc_f64x2_zero(a)` 从当前函数返回。

### Lines 1369-1392

````c
}

// CHECK-LABEL: test_u32x4_relaxed_trunc_f64x2_zero:
// CHECK: i32x4.relaxed_trunc_f64x2_u_zero{{$}}
v128_t test_u32x4_relaxed_trunc_f64x2_zero(v128_t a) {
  return wasm_u32x4_relaxed_trunc_f64x2_zero(a);
}

// CHECK-LABEL: test_i16x8_relaxed_q15mulr:
// CHECK: i16x8.relaxed_q15mulr_s{{$}}
v128_t test_i16x8_relaxed_q15mulr(v128_t a, v128_t b) {
  return wasm_i16x8_relaxed_q15mulr(a, b);
}

// CHECK-LABEL: test_i16x8_relaxed_dot_i8x16_i7x16:
// CHECK: i16x8.relaxed_dot_i8x16_i7x16_s{{$}}
v128_t test_i16x8_relaxed_dot_i8x16_i7x16(v128_t a, v128_t b) {
  return wasm_i16x8_relaxed_dot_i8x16_i7x16(a, b);
}

// CHECK-LABEL: test_i32x4_relaxed_dot_i8x16_i7x16_add:
// CHECK: i32x4.relaxed_dot_i8x16_i7x16_add_s{{$}}
v128_t test_i32x4_relaxed_dot_i8x16_i7x16_add(v128_t a, v128_t b, v128_t c) {
  return wasm_i32x4_relaxed_dot_i8x16_i7x16_add(a, b, c);
````
- **L1369 EN**: Closes the current lexical scope or compound statement.
  **L1369 CN**: 结束当前词法作用域或复合语句块。
- **L1370 EN**: Blank line separating nearby declarations or logic.
  **L1370 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1371 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_u32x4_relaxed_trunc_f64x2_zero:`.
  **L1371 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_u32x4_relaxed_trunc_f64x2_zero:`。
- **L1372 EN**: Comment documents nearby intent or constraints: `CHECK: i32x4.relaxed_trunc_f64x2_u_zero{{$}}`.
  **L1372 CN**: 注释说明附近代码的意图或约束：`CHECK: i32x4.relaxed_trunc_f64x2_u_zero{{$}}`。
- **L1373 EN**: Starts a function or method definition for `test_u32x4_relaxed_trunc_f64x2_zero`.
  **L1373 CN**: 开始定义函数或方法 `test_u32x4_relaxed_trunc_f64x2_zero`。
- **L1374 EN**: Returns from the current function with `wasm_u32x4_relaxed_trunc_f64x2_zero(a)`.
  **L1374 CN**: 以 `wasm_u32x4_relaxed_trunc_f64x2_zero(a)` 从当前函数返回。
- **L1375 EN**: Closes the current lexical scope or compound statement.
  **L1375 CN**: 结束当前词法作用域或复合语句块。
- **L1376 EN**: Blank line separating nearby declarations or logic.
  **L1376 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1377 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i16x8_relaxed_q15mulr:`.
  **L1377 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i16x8_relaxed_q15mulr:`。
- **L1378 EN**: Comment documents nearby intent or constraints: `CHECK: i16x8.relaxed_q15mulr_s{{$}}`.
  **L1378 CN**: 注释说明附近代码的意图或约束：`CHECK: i16x8.relaxed_q15mulr_s{{$}}`。
- **L1379 EN**: Starts a function or method definition for `test_i16x8_relaxed_q15mulr`.
  **L1379 CN**: 开始定义函数或方法 `test_i16x8_relaxed_q15mulr`。
- **L1380 EN**: Returns from the current function with `wasm_i16x8_relaxed_q15mulr(a, b)`.
  **L1380 CN**: 以 `wasm_i16x8_relaxed_q15mulr(a, b)` 从当前函数返回。
- **L1381 EN**: Closes the current lexical scope or compound statement.
  **L1381 CN**: 结束当前词法作用域或复合语句块。
- **L1382 EN**: Blank line separating nearby declarations or logic.
  **L1382 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1383 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i16x8_relaxed_dot_i8x16_i7x16:`.
  **L1383 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i16x8_relaxed_dot_i8x16_i7x16:`。
- **L1384 EN**: Comment documents nearby intent or constraints: `CHECK: i16x8.relaxed_dot_i8x16_i7x16_s{{$}}`.
  **L1384 CN**: 注释说明附近代码的意图或约束：`CHECK: i16x8.relaxed_dot_i8x16_i7x16_s{{$}}`。
- **L1385 EN**: Starts a function or method definition for `test_i16x8_relaxed_dot_i8x16_i7x16`.
  **L1385 CN**: 开始定义函数或方法 `test_i16x8_relaxed_dot_i8x16_i7x16`。
- **L1386 EN**: Returns from the current function with `wasm_i16x8_relaxed_dot_i8x16_i7x16(a, b)`.
  **L1386 CN**: 以 `wasm_i16x8_relaxed_dot_i8x16_i7x16(a, b)` 从当前函数返回。
- **L1387 EN**: Closes the current lexical scope or compound statement.
  **L1387 CN**: 结束当前词法作用域或复合语句块。
- **L1388 EN**: Blank line separating nearby declarations or logic.
  **L1388 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1389 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i32x4_relaxed_dot_i8x16_i7x16_add:`.
  **L1389 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i32x4_relaxed_dot_i8x16_i7x16_add:`。
- **L1390 EN**: Comment documents nearby intent or constraints: `CHECK: i32x4.relaxed_dot_i8x16_i7x16_add_s{{$}}`.
  **L1390 CN**: 注释说明附近代码的意图或约束：`CHECK: i32x4.relaxed_dot_i8x16_i7x16_add_s{{$}}`。
- **L1391 EN**: Starts a function or method definition for `test_i32x4_relaxed_dot_i8x16_i7x16_add`.
  **L1391 CN**: 开始定义函数或方法 `test_i32x4_relaxed_dot_i8x16_i7x16_add`。
- **L1392 EN**: Returns from the current function with `wasm_i32x4_relaxed_dot_i8x16_i7x16_add(a, b, c)`.
  **L1392 CN**: 以 `wasm_i32x4_relaxed_dot_i8x16_i7x16_add(a, b, c)` 从当前函数返回。

### Lines 1393-1416

````c
}

// CHECK-LABEL: test_f16x8_splat:
// CHECK: f16x8.splat{{$}}
v128_t test_f16x8_splat(float a) { return wasm_f16x8_splat(a); }

// CHECK-LABEL: test_f16x8_extract_lane:
// CHECK: f16x8.extract_lane 7{{$}}
int16_t test_f16x8_extract_lane(v128_t a) {
  return wasm_f16x8_extract_lane(a, 7);
}

// CHECK-LABEL: test_f16x8_replace_lane:
// CHECK: f16x8.replace_lane 7{{$}}
v128_t test_f16x8_replace_lane(v128_t a, float b) {
  return wasm_f16x8_replace_lane(a, 7, b);
}

// CHECK-LABEL: test_f16x8_abs:
// CHECK: f16x8.abs{{$}}
v128_t test_f16x8_abs(v128_t a) { return wasm_f16x8_abs(a); }

// CHECK-LABEL: test_f16x8_neg:
// CHECK: f16x8.neg{{$}}
````
- **L1393 EN**: Closes the current lexical scope or compound statement.
  **L1393 CN**: 结束当前词法作用域或复合语句块。
- **L1394 EN**: Blank line separating nearby declarations or logic.
  **L1394 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1395 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f16x8_splat:`.
  **L1395 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f16x8_splat:`。
- **L1396 EN**: Comment documents nearby intent or constraints: `CHECK: f16x8.splat{{$}}`.
  **L1396 CN**: 注释说明附近代码的意图或约束：`CHECK: f16x8.splat{{$}}`。
- **L1397 EN**: Starts a function or method definition for `test_f16x8_splat`.
  **L1397 CN**: 开始定义函数或方法 `test_f16x8_splat`。
- **L1398 EN**: Blank line separating nearby declarations or logic.
  **L1398 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1399 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f16x8_extract_lane:`.
  **L1399 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f16x8_extract_lane:`。
- **L1400 EN**: Comment documents nearby intent or constraints: `CHECK: f16x8.extract_lane 7{{$}}`.
  **L1400 CN**: 注释说明附近代码的意图或约束：`CHECK: f16x8.extract_lane 7{{$}}`。
- **L1401 EN**: Starts a function or method definition for `test_f16x8_extract_lane`.
  **L1401 CN**: 开始定义函数或方法 `test_f16x8_extract_lane`。
- **L1402 EN**: Returns from the current function with `wasm_f16x8_extract_lane(a, 7)`.
  **L1402 CN**: 以 `wasm_f16x8_extract_lane(a, 7)` 从当前函数返回。
- **L1403 EN**: Closes the current lexical scope or compound statement.
  **L1403 CN**: 结束当前词法作用域或复合语句块。
- **L1404 EN**: Blank line separating nearby declarations or logic.
  **L1404 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1405 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f16x8_replace_lane:`.
  **L1405 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f16x8_replace_lane:`。
- **L1406 EN**: Comment documents nearby intent or constraints: `CHECK: f16x8.replace_lane 7{{$}}`.
  **L1406 CN**: 注释说明附近代码的意图或约束：`CHECK: f16x8.replace_lane 7{{$}}`。
- **L1407 EN**: Starts a function or method definition for `test_f16x8_replace_lane`.
  **L1407 CN**: 开始定义函数或方法 `test_f16x8_replace_lane`。
- **L1408 EN**: Returns from the current function with `wasm_f16x8_replace_lane(a, 7, b)`.
  **L1408 CN**: 以 `wasm_f16x8_replace_lane(a, 7, b)` 从当前函数返回。
- **L1409 EN**: Closes the current lexical scope or compound statement.
  **L1409 CN**: 结束当前词法作用域或复合语句块。
- **L1410 EN**: Blank line separating nearby declarations or logic.
  **L1410 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1411 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f16x8_abs:`.
  **L1411 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f16x8_abs:`。
- **L1412 EN**: Comment documents nearby intent or constraints: `CHECK: f16x8.abs{{$}}`.
  **L1412 CN**: 注释说明附近代码的意图或约束：`CHECK: f16x8.abs{{$}}`。
- **L1413 EN**: Starts a function or method definition for `test_f16x8_abs`.
  **L1413 CN**: 开始定义函数或方法 `test_f16x8_abs`。
- **L1414 EN**: Blank line separating nearby declarations or logic.
  **L1414 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1415 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f16x8_neg:`.
  **L1415 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f16x8_neg:`。
- **L1416 EN**: Comment documents nearby intent or constraints: `CHECK: f16x8.neg{{$}}`.
  **L1416 CN**: 注释说明附近代码的意图或约束：`CHECK: f16x8.neg{{$}}`。

### Lines 1417-1440

````c
v128_t test_f16x8_neg(v128_t a) { return wasm_f16x8_neg(a); }

// CHECK-LABEL: test_f16x8_sqrt:
// CHECK: f16x8.sqrt{{$}}
v128_t test_f16x8_sqrt(v128_t a) { return wasm_f16x8_sqrt(a); }

// CHECK-LABEL: test_f16x8_ceil:
// CHECK: f16x8.ceil{{$}}
v128_t test_f16x8_ceil(v128_t a) { return wasm_f16x8_ceil(a); }

// CHECK-LABEL: test_f16x8_floor:
// CHECK: f16x8.floor{{$}}
v128_t test_f16x8_floor(v128_t a) { return wasm_f16x8_floor(a); }

// CHECK-LABEL: test_f16x8_trunc:
// CHECK: f16x8.trunc{{$}}
v128_t test_f16x8_trunc(v128_t a) { return wasm_f16x8_trunc(a); }

// CHECK-LABEL: test_f16x8_nearest:
// CHECK: f16x8.nearest{{$}}
v128_t test_f16x8_nearest(v128_t a) { return wasm_f16x8_nearest(a); }

// CHECK-LABEL: test_f16x8_add:
// CHECK: f16x8.add{{$}}
````
- **L1417 EN**: Starts a function or method definition for `test_f16x8_neg`.
  **L1417 CN**: 开始定义函数或方法 `test_f16x8_neg`。
- **L1418 EN**: Blank line separating nearby declarations or logic.
  **L1418 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1419 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f16x8_sqrt:`.
  **L1419 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f16x8_sqrt:`。
- **L1420 EN**: Comment documents nearby intent or constraints: `CHECK: f16x8.sqrt{{$}}`.
  **L1420 CN**: 注释说明附近代码的意图或约束：`CHECK: f16x8.sqrt{{$}}`。
- **L1421 EN**: Starts a function or method definition for `test_f16x8_sqrt`.
  **L1421 CN**: 开始定义函数或方法 `test_f16x8_sqrt`。
- **L1422 EN**: Blank line separating nearby declarations or logic.
  **L1422 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1423 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f16x8_ceil:`.
  **L1423 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f16x8_ceil:`。
- **L1424 EN**: Comment documents nearby intent or constraints: `CHECK: f16x8.ceil{{$}}`.
  **L1424 CN**: 注释说明附近代码的意图或约束：`CHECK: f16x8.ceil{{$}}`。
- **L1425 EN**: Starts a function or method definition for `test_f16x8_ceil`.
  **L1425 CN**: 开始定义函数或方法 `test_f16x8_ceil`。
- **L1426 EN**: Blank line separating nearby declarations or logic.
  **L1426 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1427 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f16x8_floor:`.
  **L1427 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f16x8_floor:`。
- **L1428 EN**: Comment documents nearby intent or constraints: `CHECK: f16x8.floor{{$}}`.
  **L1428 CN**: 注释说明附近代码的意图或约束：`CHECK: f16x8.floor{{$}}`。
- **L1429 EN**: Starts a function or method definition for `test_f16x8_floor`.
  **L1429 CN**: 开始定义函数或方法 `test_f16x8_floor`。
- **L1430 EN**: Blank line separating nearby declarations or logic.
  **L1430 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1431 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f16x8_trunc:`.
  **L1431 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f16x8_trunc:`。
- **L1432 EN**: Comment documents nearby intent or constraints: `CHECK: f16x8.trunc{{$}}`.
  **L1432 CN**: 注释说明附近代码的意图或约束：`CHECK: f16x8.trunc{{$}}`。
- **L1433 EN**: Starts a function or method definition for `test_f16x8_trunc`.
  **L1433 CN**: 开始定义函数或方法 `test_f16x8_trunc`。
- **L1434 EN**: Blank line separating nearby declarations or logic.
  **L1434 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1435 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f16x8_nearest:`.
  **L1435 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f16x8_nearest:`。
- **L1436 EN**: Comment documents nearby intent or constraints: `CHECK: f16x8.nearest{{$}}`.
  **L1436 CN**: 注释说明附近代码的意图或约束：`CHECK: f16x8.nearest{{$}}`。
- **L1437 EN**: Starts a function or method definition for `test_f16x8_nearest`.
  **L1437 CN**: 开始定义函数或方法 `test_f16x8_nearest`。
- **L1438 EN**: Blank line separating nearby declarations or logic.
  **L1438 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1439 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f16x8_add:`.
  **L1439 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f16x8_add:`。
- **L1440 EN**: Comment documents nearby intent or constraints: `CHECK: f16x8.add{{$}}`.
  **L1440 CN**: 注释说明附近代码的意图或约束：`CHECK: f16x8.add{{$}}`。

### Lines 1441-1464

````c
v128_t test_f16x8_add(v128_t a, v128_t b) { return wasm_f16x8_add(a, b); }

// CHECK-LABEL: test_f16x8_sub:
// CHECK: f16x8.sub{{$}}
v128_t test_f16x8_sub(v128_t a, v128_t b) { return wasm_f16x8_sub(a, b); }

// CHECK-LABEL: test_f16x8_mul:
// CHECK: f16x8.mul{{$}}
v128_t test_f16x8_mul(v128_t a, v128_t b) { return wasm_f16x8_mul(a, b); }

// CHECK-LABEL: test_f16x8_div:
// CHECK: f16x8.div{{$}}
v128_t test_f16x8_div(v128_t a, v128_t b) { return wasm_f16x8_div(a, b); }

// CHECK-LABEL: test_f16x8_min:
// CHECK: f16x8.min{{$}}
v128_t test_f16x8_min(v128_t a, v128_t b) { return wasm_f16x8_min(a, b); }

// CHECK-LABEL: test_f16x8_max:
// CHECK: f16x8.max{{$}}
v128_t test_f16x8_max(v128_t a, v128_t b) { return wasm_f16x8_max(a, b); }

// CHECK-LABEL: test_f16x8_pmin:
// CHECK: f16x8.pmin{{$}}
````
- **L1441 EN**: Starts a function or method definition for `test_f16x8_add`.
  **L1441 CN**: 开始定义函数或方法 `test_f16x8_add`。
- **L1442 EN**: Blank line separating nearby declarations or logic.
  **L1442 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1443 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f16x8_sub:`.
  **L1443 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f16x8_sub:`。
- **L1444 EN**: Comment documents nearby intent or constraints: `CHECK: f16x8.sub{{$}}`.
  **L1444 CN**: 注释说明附近代码的意图或约束：`CHECK: f16x8.sub{{$}}`。
- **L1445 EN**: Starts a function or method definition for `test_f16x8_sub`.
  **L1445 CN**: 开始定义函数或方法 `test_f16x8_sub`。
- **L1446 EN**: Blank line separating nearby declarations or logic.
  **L1446 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1447 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f16x8_mul:`.
  **L1447 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f16x8_mul:`。
- **L1448 EN**: Comment documents nearby intent or constraints: `CHECK: f16x8.mul{{$}}`.
  **L1448 CN**: 注释说明附近代码的意图或约束：`CHECK: f16x8.mul{{$}}`。
- **L1449 EN**: Starts a function or method definition for `test_f16x8_mul`.
  **L1449 CN**: 开始定义函数或方法 `test_f16x8_mul`。
- **L1450 EN**: Blank line separating nearby declarations or logic.
  **L1450 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1451 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f16x8_div:`.
  **L1451 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f16x8_div:`。
- **L1452 EN**: Comment documents nearby intent or constraints: `CHECK: f16x8.div{{$}}`.
  **L1452 CN**: 注释说明附近代码的意图或约束：`CHECK: f16x8.div{{$}}`。
- **L1453 EN**: Starts a function or method definition for `test_f16x8_div`.
  **L1453 CN**: 开始定义函数或方法 `test_f16x8_div`。
- **L1454 EN**: Blank line separating nearby declarations or logic.
  **L1454 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1455 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f16x8_min:`.
  **L1455 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f16x8_min:`。
- **L1456 EN**: Comment documents nearby intent or constraints: `CHECK: f16x8.min{{$}}`.
  **L1456 CN**: 注释说明附近代码的意图或约束：`CHECK: f16x8.min{{$}}`。
- **L1457 EN**: Starts a function or method definition for `test_f16x8_min`.
  **L1457 CN**: 开始定义函数或方法 `test_f16x8_min`。
- **L1458 EN**: Blank line separating nearby declarations or logic.
  **L1458 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1459 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f16x8_max:`.
  **L1459 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f16x8_max:`。
- **L1460 EN**: Comment documents nearby intent or constraints: `CHECK: f16x8.max{{$}}`.
  **L1460 CN**: 注释说明附近代码的意图或约束：`CHECK: f16x8.max{{$}}`。
- **L1461 EN**: Starts a function or method definition for `test_f16x8_max`.
  **L1461 CN**: 开始定义函数或方法 `test_f16x8_max`。
- **L1462 EN**: Blank line separating nearby declarations or logic.
  **L1462 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1463 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f16x8_pmin:`.
  **L1463 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f16x8_pmin:`。
- **L1464 EN**: Comment documents nearby intent or constraints: `CHECK: f16x8.pmin{{$}}`.
  **L1464 CN**: 注释说明附近代码的意图或约束：`CHECK: f16x8.pmin{{$}}`。

### Lines 1465-1488

````c
v128_t test_f16x8_pmin(v128_t a, v128_t b) { return wasm_f16x8_pmin(a, b); }

// CHECK-LABEL: test_f16x8_pmax:
// CHECK: f16x8.pmax{{$}}
v128_t test_f16x8_pmax(v128_t a, v128_t b) { return wasm_f16x8_pmax(a, b); }

// CHECK-LABEL: test_f16x8_eq:
// CHECK: f16x8.eq{{$}}
v128_t test_f16x8_eq(v128_t a, v128_t b) { return wasm_f16x8_eq(a, b); }

// CHECK-LABEL: test_f16x8_ne:
// CHECK: f16x8.ne{{$}}
v128_t test_f16x8_ne(v128_t a, v128_t b) { return wasm_f16x8_ne(a, b); }

// CHECK-LABEL: test_f16x8_lt:
// CHECK: f16x8.lt{{$}}
v128_t test_f16x8_lt(v128_t a, v128_t b) { return wasm_f16x8_lt(a, b); }

// CHECK-LABEL: test_f16x8_gt:
// CHECK: f16x8.gt{{$}}
v128_t test_f16x8_gt(v128_t a, v128_t b) { return wasm_f16x8_gt(a, b); }

// CHECK-LABEL: test_f16x8_le:
// CHECK: f16x8.le{{$}}
````
- **L1465 EN**: Starts a function or method definition for `test_f16x8_pmin`.
  **L1465 CN**: 开始定义函数或方法 `test_f16x8_pmin`。
- **L1466 EN**: Blank line separating nearby declarations or logic.
  **L1466 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1467 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f16x8_pmax:`.
  **L1467 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f16x8_pmax:`。
- **L1468 EN**: Comment documents nearby intent or constraints: `CHECK: f16x8.pmax{{$}}`.
  **L1468 CN**: 注释说明附近代码的意图或约束：`CHECK: f16x8.pmax{{$}}`。
- **L1469 EN**: Starts a function or method definition for `test_f16x8_pmax`.
  **L1469 CN**: 开始定义函数或方法 `test_f16x8_pmax`。
- **L1470 EN**: Blank line separating nearby declarations or logic.
  **L1470 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1471 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f16x8_eq:`.
  **L1471 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f16x8_eq:`。
- **L1472 EN**: Comment documents nearby intent or constraints: `CHECK: f16x8.eq{{$}}`.
  **L1472 CN**: 注释说明附近代码的意图或约束：`CHECK: f16x8.eq{{$}}`。
- **L1473 EN**: Starts a function or method definition for `test_f16x8_eq`.
  **L1473 CN**: 开始定义函数或方法 `test_f16x8_eq`。
- **L1474 EN**: Blank line separating nearby declarations or logic.
  **L1474 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1475 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f16x8_ne:`.
  **L1475 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f16x8_ne:`。
- **L1476 EN**: Comment documents nearby intent or constraints: `CHECK: f16x8.ne{{$}}`.
  **L1476 CN**: 注释说明附近代码的意图或约束：`CHECK: f16x8.ne{{$}}`。
- **L1477 EN**: Starts a function or method definition for `test_f16x8_ne`.
  **L1477 CN**: 开始定义函数或方法 `test_f16x8_ne`。
- **L1478 EN**: Blank line separating nearby declarations or logic.
  **L1478 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1479 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f16x8_lt:`.
  **L1479 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f16x8_lt:`。
- **L1480 EN**: Comment documents nearby intent or constraints: `CHECK: f16x8.lt{{$}}`.
  **L1480 CN**: 注释说明附近代码的意图或约束：`CHECK: f16x8.lt{{$}}`。
- **L1481 EN**: Starts a function or method definition for `test_f16x8_lt`.
  **L1481 CN**: 开始定义函数或方法 `test_f16x8_lt`。
- **L1482 EN**: Blank line separating nearby declarations or logic.
  **L1482 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1483 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f16x8_gt:`.
  **L1483 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f16x8_gt:`。
- **L1484 EN**: Comment documents nearby intent or constraints: `CHECK: f16x8.gt{{$}}`.
  **L1484 CN**: 注释说明附近代码的意图或约束：`CHECK: f16x8.gt{{$}}`。
- **L1485 EN**: Starts a function or method definition for `test_f16x8_gt`.
  **L1485 CN**: 开始定义函数或方法 `test_f16x8_gt`。
- **L1486 EN**: Blank line separating nearby declarations or logic.
  **L1486 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1487 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f16x8_le:`.
  **L1487 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f16x8_le:`。
- **L1488 EN**: Comment documents nearby intent or constraints: `CHECK: f16x8.le{{$}}`.
  **L1488 CN**: 注释说明附近代码的意图或约束：`CHECK: f16x8.le{{$}}`。

### Lines 1489-1512

````c
v128_t test_f16x8_le(v128_t a, v128_t b) { return wasm_f16x8_le(a, b); }

// CHECK-LABEL: test_f16x8_ge:
// CHECK: f16x8.ge{{$}}
v128_t test_f16x8_ge(v128_t a, v128_t b) { return wasm_f16x8_ge(a, b); }

// CHECK-LABEL: test_i16x8_trunc_sat_f16x8:
// CHECK: i16x8.trunc_sat_f16x8_s{{$}}
v128_t test_i16x8_trunc_sat_f16x8(v128_t a) {
  return wasm_i16x8_trunc_sat_f16x8(a);
}

// CHECK-LABEL: test_u16x8_trunc_sat_f16x8:
// CHECK: i16x8.trunc_sat_f16x8_u{{$}}
v128_t test_u16x8_trunc_sat_f16x8(v128_t a) {
  return wasm_u16x8_trunc_sat_f16x8(a);
}

// CHECK-LABEL: test_f16x8_convert_i16x8:
// CHECK: f16x8.convert_i16x8_s{{$}}
v128_t test_f16x8_convert_i16x8(v128_t a) {
  return wasm_f16x8_convert_i16x8(a);
}

````
- **L1489 EN**: Starts a function or method definition for `test_f16x8_le`.
  **L1489 CN**: 开始定义函数或方法 `test_f16x8_le`。
- **L1490 EN**: Blank line separating nearby declarations or logic.
  **L1490 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1491 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f16x8_ge:`.
  **L1491 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f16x8_ge:`。
- **L1492 EN**: Comment documents nearby intent or constraints: `CHECK: f16x8.ge{{$}}`.
  **L1492 CN**: 注释说明附近代码的意图或约束：`CHECK: f16x8.ge{{$}}`。
- **L1493 EN**: Starts a function or method definition for `test_f16x8_ge`.
  **L1493 CN**: 开始定义函数或方法 `test_f16x8_ge`。
- **L1494 EN**: Blank line separating nearby declarations or logic.
  **L1494 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1495 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_i16x8_trunc_sat_f16x8:`.
  **L1495 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_i16x8_trunc_sat_f16x8:`。
- **L1496 EN**: Comment documents nearby intent or constraints: `CHECK: i16x8.trunc_sat_f16x8_s{{$}}`.
  **L1496 CN**: 注释说明附近代码的意图或约束：`CHECK: i16x8.trunc_sat_f16x8_s{{$}}`。
- **L1497 EN**: Starts a function or method definition for `test_i16x8_trunc_sat_f16x8`.
  **L1497 CN**: 开始定义函数或方法 `test_i16x8_trunc_sat_f16x8`。
- **L1498 EN**: Returns from the current function with `wasm_i16x8_trunc_sat_f16x8(a)`.
  **L1498 CN**: 以 `wasm_i16x8_trunc_sat_f16x8(a)` 从当前函数返回。
- **L1499 EN**: Closes the current lexical scope or compound statement.
  **L1499 CN**: 结束当前词法作用域或复合语句块。
- **L1500 EN**: Blank line separating nearby declarations or logic.
  **L1500 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1501 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_u16x8_trunc_sat_f16x8:`.
  **L1501 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_u16x8_trunc_sat_f16x8:`。
- **L1502 EN**: Comment documents nearby intent or constraints: `CHECK: i16x8.trunc_sat_f16x8_u{{$}}`.
  **L1502 CN**: 注释说明附近代码的意图或约束：`CHECK: i16x8.trunc_sat_f16x8_u{{$}}`。
- **L1503 EN**: Starts a function or method definition for `test_u16x8_trunc_sat_f16x8`.
  **L1503 CN**: 开始定义函数或方法 `test_u16x8_trunc_sat_f16x8`。
- **L1504 EN**: Returns from the current function with `wasm_u16x8_trunc_sat_f16x8(a)`.
  **L1504 CN**: 以 `wasm_u16x8_trunc_sat_f16x8(a)` 从当前函数返回。
- **L1505 EN**: Closes the current lexical scope or compound statement.
  **L1505 CN**: 结束当前词法作用域或复合语句块。
- **L1506 EN**: Blank line separating nearby declarations or logic.
  **L1506 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1507 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f16x8_convert_i16x8:`.
  **L1507 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f16x8_convert_i16x8:`。
- **L1508 EN**: Comment documents nearby intent or constraints: `CHECK: f16x8.convert_i16x8_s{{$}}`.
  **L1508 CN**: 注释说明附近代码的意图或约束：`CHECK: f16x8.convert_i16x8_s{{$}}`。
- **L1509 EN**: Starts a function or method definition for `test_f16x8_convert_i16x8`.
  **L1509 CN**: 开始定义函数或方法 `test_f16x8_convert_i16x8`。
- **L1510 EN**: Returns from the current function with `wasm_f16x8_convert_i16x8(a)`.
  **L1510 CN**: 以 `wasm_f16x8_convert_i16x8(a)` 从当前函数返回。
- **L1511 EN**: Closes the current lexical scope or compound statement.
  **L1511 CN**: 结束当前词法作用域或复合语句块。
- **L1512 EN**: Blank line separating nearby declarations or logic.
  **L1512 CN**: 空行，用于分隔相邻声明或逻辑。

### Lines 1513-1529

````c
// CHECK-LABEL: test_f16x8_convert_u16x8:
// CHECK: f16x8.convert_i16x8_u{{$}}
v128_t test_f16x8_convert_u16x8(v128_t a) {
  return wasm_f16x8_convert_u16x8(a);
}

// CHECK-LABEL: test_f16x8_relaxed_madd:
// CHECK: f16x8.madd{{$}}
v128_t test_f16x8_relaxed_madd(v128_t a, v128_t b, v128_t c) {
  return wasm_f16x8_relaxed_madd(a, b, c);
}

// CHECK-LABEL: test_f16x8_relaxed_nmadd:
// CHECK: f16x8.nmadd{{$}}
v128_t test_f16x8_relaxed_nmadd(v128_t a, v128_t b, v128_t c) {
  return wasm_f16x8_relaxed_nmadd(a, b, c);
}
````
- **L1513 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f16x8_convert_u16x8:`.
  **L1513 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f16x8_convert_u16x8:`。
- **L1514 EN**: Comment documents nearby intent or constraints: `CHECK: f16x8.convert_i16x8_u{{$}}`.
  **L1514 CN**: 注释说明附近代码的意图或约束：`CHECK: f16x8.convert_i16x8_u{{$}}`。
- **L1515 EN**: Starts a function or method definition for `test_f16x8_convert_u16x8`.
  **L1515 CN**: 开始定义函数或方法 `test_f16x8_convert_u16x8`。
- **L1516 EN**: Returns from the current function with `wasm_f16x8_convert_u16x8(a)`.
  **L1516 CN**: 以 `wasm_f16x8_convert_u16x8(a)` 从当前函数返回。
- **L1517 EN**: Closes the current lexical scope or compound statement.
  **L1517 CN**: 结束当前词法作用域或复合语句块。
- **L1518 EN**: Blank line separating nearby declarations or logic.
  **L1518 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1519 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f16x8_relaxed_madd:`.
  **L1519 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f16x8_relaxed_madd:`。
- **L1520 EN**: Comment documents nearby intent or constraints: `CHECK: f16x8.madd{{$}}`.
  **L1520 CN**: 注释说明附近代码的意图或约束：`CHECK: f16x8.madd{{$}}`。
- **L1521 EN**: Starts a function or method definition for `test_f16x8_relaxed_madd`.
  **L1521 CN**: 开始定义函数或方法 `test_f16x8_relaxed_madd`。
- **L1522 EN**: Returns from the current function with `wasm_f16x8_relaxed_madd(a, b, c)`.
  **L1522 CN**: 以 `wasm_f16x8_relaxed_madd(a, b, c)` 从当前函数返回。
- **L1523 EN**: Closes the current lexical scope or compound statement.
  **L1523 CN**: 结束当前词法作用域或复合语句块。
- **L1524 EN**: Blank line separating nearby declarations or logic.
  **L1524 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1525 EN**: Comment documents nearby intent or constraints: `CHECK-LABEL: test_f16x8_relaxed_nmadd:`.
  **L1525 CN**: 注释说明附近代码的意图或约束：`CHECK-LABEL: test_f16x8_relaxed_nmadd:`。
- **L1526 EN**: Comment documents nearby intent or constraints: `CHECK: f16x8.nmadd{{$}}`.
  **L1526 CN**: 注释说明附近代码的意图或约束：`CHECK: f16x8.nmadd{{$}}`。
- **L1527 EN**: Starts a function or method definition for `test_f16x8_relaxed_nmadd`.
  **L1527 CN**: 开始定义函数或方法 `test_f16x8_relaxed_nmadd`。
- **L1528 EN**: Returns from the current function with `wasm_f16x8_relaxed_nmadd(a, b, c)`.
  **L1528 CN**: 以 `wasm_f16x8_relaxed_nmadd(a, b, c)` 从当前函数返回。
- **L1529 EN**: Closes the current lexical scope or compound statement.
  **L1529 CN**: 结束当前词法作用域或复合语句块。

## Key Concepts / 关键概念

- **Cross-project integration testing / 跨项目集成测试**:
  - **EN**: Validates behavior that emerges only when multiple LLVM-family components cooperate.
  - **CN**: 验证多个 LLVM 家族组件协同工作时才会显现的行为。
- **Translation-unit implementation / 编译单元实现**:
  - **EN**: Contains concrete runtime logic rather than only declarations.
  - **CN**: 包含具体的运行时实现逻辑，而不仅仅是声明。
- **Dependency surface / 依赖表面**:
  - **EN**: Relies on neighboring headers and runtime interfaces to assemble the complete feature.
  - **CN**: 依赖相邻头文件与运行时接口来组装完整功能。

## Dependencies / 依赖关系

- **External or standard includes / 外部或标准包含**: `wasm_simd128.h`
- **Dependency categories / 依赖类别**: C or C++ standard library facilities / C 或 C++ 标准库设施 (1)

- **EN**: `wasm_simd128.h` provides C or C++ standard library facilities.
  - **CN**: `wasm_simd128.h` 提供 C 或 C++ 标准库设施。
