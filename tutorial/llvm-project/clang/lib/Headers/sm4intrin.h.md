# sm4intrin.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/Headers/sm4intrin.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: SM4 intrinsics.
- **Purpose (CN)**: 提供 SM4 intrinsic 接口。
- **Line Count / 行数**: 269

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

````c
/*===--------------- sm4intrin.h - SM4 intrinsics -----------------===
 *
 * Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 * See https://llvm.org/LICENSE.txt for license information.
 * SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 *
 *===-----------------------------------------------------------------------===
 */

#ifndef __IMMINTRIN_H
#error "Never use <sm4intrin.h> directly; include <immintrin.h> instead."
#endif // __IMMINTRIN_H

#ifndef __SM4INTRIN_H
#define __SM4INTRIN_H

/// This intrinsic performs four rounds of SM4 key expansion. The intrinsic
///    operates on independent 128-bit lanes. The calculated results are
///    stored in \a dst.
/// \headerfile <immintrin.h>
````
- **L1 EN**: Banner comment marking a file or section boundary.
  **L1 CN**: 横幅注释，用于标记文件或章节边界。
- **L2 EN**: Separator comment used for visual grouping.
  **L2 CN**: 用于视觉分组的分隔注释。
- **L3 EN**: Comment explains nearby logic, constraints, or intent: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`.
  **L3 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L4 EN**: Comment explains nearby logic, constraints, or intent: `See https://llvm.org/LICENSE.txt for license information.`.
  **L4 CN**: 注释解释附近代码的逻辑、约束或设计意图：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5 EN**: Comment explains nearby logic, constraints, or intent: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`.
  **L5 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6 EN**: Separator comment used for visual grouping.
  **L6 CN**: 用于视觉分组的分隔注释。
- **L7 EN**: Banner comment marking a file or section boundary.
  **L7 CN**: 横幅注释，用于标记文件或章节边界。
- **L8 EN**: Separator comment used for visual grouping.
  **L8 CN**: 用于视觉分组的分隔注释。
- **L9 EN**: Blank line separating nearby declarations or logic blocks.
  **L9 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L10 EN**: Starts a preprocessor conditional block: `#ifndef __IMMINTRIN_H`.
  **L10 CN**: 开始一个预处理条件块：`#ifndef __IMMINTRIN_H`。
- **L11 EN**: Emits a compilation error for an unsupported configuration: `#error "Never use <sm4intrin.h> directly; include <immintrin.h> instead."`.
  **L11 CN**: 为不受支持的配置触发编译错误：`#error "Never use <sm4intrin.h> directly; include <immintrin.h> instead."`。
- **L12 EN**: Closes the current preprocessor conditional block.
  **L12 CN**: 结束当前预处理条件块。
- **L13 EN**: Blank line separating nearby declarations or logic blocks.
  **L13 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L14 EN**: Starts a preprocessor conditional block: `#ifndef __SM4INTRIN_H`.
  **L14 CN**: 开始一个预处理条件块：`#ifndef __SM4INTRIN_H`。
- **L15 EN**: Defines macro `__SM4INTRIN_H` for conditional compilation, shorthand, or API generation.
  **L15 CN**: 定义宏 `__SM4INTRIN_H`，用于条件编译、简写或 API 生成。
- **L16 EN**: Blank line separating nearby declarations or logic blocks.
  **L16 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L17 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic performs four rounds of SM4 key expansion. The intrinsic`.
  **L17 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic performs four rounds of SM4 key expansion. The intrinsic`。
- **L18 EN**: Comment explains nearby logic, constraints, or intent: `operates on independent 128-bit lanes. The calculated results are`.
  **L18 CN**: 注释解释附近代码的逻辑、约束或设计意图：`operates on independent 128-bit lanes. The calculated results are`。
- **L19 EN**: Comment explains nearby logic, constraints, or intent: `stored in a dst.`.
  **L19 CN**: 注释解释附近代码的逻辑、约束或设计意图：`stored in a dst.`。
- **L20 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L20 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。

### Lines 21-40

````c
///
/// \code
/// __m128i _mm_sm4key4_epi32(__m128i __A, __m128i __B)
/// \endcode
///
/// This intrinsic corresponds to the \c VSM4KEY4 instruction.
///
/// \param __A
///    A 128-bit vector of [4 x int].
/// \param __B
///    A 128-bit vector of [4 x int].
/// \returns
///    A 128-bit vector of [4 x int].
///
/// \code{.operation}
/// DEFINE ROL32(dword, n) {
/// 	count := n % 32
/// 	dest := (dword << count) | (dword >> (32-count))
/// 	RETURN dest
/// }
````
- **L21 EN**: Separator comment used for visual grouping.
  **L21 CN**: 用于视觉分组的分隔注释。
- **L22 EN**: Comment explains nearby logic, constraints, or intent: `code`.
  **L22 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code`。
- **L23 EN**: Comment explains nearby logic, constraints, or intent: `__m128i _mm_sm4key4_epi32(__m128i __A, __m128i __B)`.
  **L23 CN**: 注释解释附近代码的逻辑、约束或设计意图：`__m128i _mm_sm4key4_epi32(__m128i __A, __m128i __B)`。
- **L24 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L24 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L25 EN**: Separator comment used for visual grouping.
  **L25 CN**: 用于视觉分组的分隔注释。
- **L26 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VSM4KEY4 instruction.`.
  **L26 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VSM4KEY4 instruction.`。
- **L27 EN**: Separator comment used for visual grouping.
  **L27 CN**: 用于视觉分组的分隔注释。
- **L28 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L28 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L29 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x int].`.
  **L29 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x int].`。
- **L30 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L30 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L31 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x int].`.
  **L31 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x int].`。
- **L32 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L32 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L33 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x int].`.
  **L33 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x int].`。
- **L34 EN**: Separator comment used for visual grouping.
  **L34 CN**: 用于视觉分组的分隔注释。
- **L35 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L35 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L36 EN**: Comment explains nearby logic, constraints, or intent: `DEFINE ROL32(dword, n) {`.
  **L36 CN**: 注释解释附近代码的逻辑、约束或设计意图：`DEFINE ROL32(dword, n) {`。
- **L37 EN**: Comment explains nearby logic, constraints, or intent: `count : n % 32`.
  **L37 CN**: 注释解释附近代码的逻辑、约束或设计意图：`count : n % 32`。
- **L38 EN**: Comment explains nearby logic, constraints, or intent: `dest : (dword << count) | (dword >> (32-count))`.
  **L38 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dest : (dword << count) | (dword >> (32-count))`。
- **L39 EN**: Comment explains nearby logic, constraints, or intent: `RETURN dest`.
  **L39 CN**: 注释解释附近代码的逻辑、约束或设计意图：`RETURN dest`。
- **L40 EN**: Comment explains nearby logic, constraints, or intent: `}`.
  **L40 CN**: 注释解释附近代码的逻辑、约束或设计意图：`}`。

### Lines 41-60

````c
/// DEFINE SBOX_BYTE(dword, i) {
/// 	RETURN sbox[dword.byte[i]]
/// }
/// DEFINE lower_t(dword) {
/// 	tmp.byte[0] := SBOX_BYTE(dword, 0)
/// 	tmp.byte[1] := SBOX_BYTE(dword, 1)
/// 	tmp.byte[2] := SBOX_BYTE(dword, 2)
/// 	tmp.byte[3] := SBOX_BYTE(dword, 3)
/// 	RETURN tmp
/// }
/// DEFINE L_KEY(dword) {
/// 	RETURN dword ^ ROL32(dword, 13) ^ ROL32(dword, 23)
/// }
/// DEFINE T_KEY(dword) {
/// 	RETURN L_KEY(lower_t(dword))
/// }
/// DEFINE F_KEY(X0, X1, X2, X3, round_key) {
/// 	RETURN X0 ^ T_KEY(X1 ^ X2 ^ X3 ^ round_key)
/// }
/// FOR i:= 0 to 0
````
- **L41 EN**: Comment explains nearby logic, constraints, or intent: `DEFINE SBOX_BYTE(dword, i) {`.
  **L41 CN**: 注释解释附近代码的逻辑、约束或设计意图：`DEFINE SBOX_BYTE(dword, i) {`。
- **L42 EN**: Comment explains nearby logic, constraints, or intent: `RETURN sbox[dword.byte[i]]`.
  **L42 CN**: 注释解释附近代码的逻辑、约束或设计意图：`RETURN sbox[dword.byte[i]]`。
- **L43 EN**: Comment explains nearby logic, constraints, or intent: `}`.
  **L43 CN**: 注释解释附近代码的逻辑、约束或设计意图：`}`。
- **L44 EN**: Comment explains nearby logic, constraints, or intent: `DEFINE lower_t(dword) {`.
  **L44 CN**: 注释解释附近代码的逻辑、约束或设计意图：`DEFINE lower_t(dword) {`。
- **L45 EN**: Comment explains nearby logic, constraints, or intent: `tmp.byte[0] : SBOX_BYTE(dword, 0)`.
  **L45 CN**: 注释解释附近代码的逻辑、约束或设计意图：`tmp.byte[0] : SBOX_BYTE(dword, 0)`。
- **L46 EN**: Comment explains nearby logic, constraints, or intent: `tmp.byte[1] : SBOX_BYTE(dword, 1)`.
  **L46 CN**: 注释解释附近代码的逻辑、约束或设计意图：`tmp.byte[1] : SBOX_BYTE(dword, 1)`。
- **L47 EN**: Comment explains nearby logic, constraints, or intent: `tmp.byte[2] : SBOX_BYTE(dword, 2)`.
  **L47 CN**: 注释解释附近代码的逻辑、约束或设计意图：`tmp.byte[2] : SBOX_BYTE(dword, 2)`。
- **L48 EN**: Comment explains nearby logic, constraints, or intent: `tmp.byte[3] : SBOX_BYTE(dword, 3)`.
  **L48 CN**: 注释解释附近代码的逻辑、约束或设计意图：`tmp.byte[3] : SBOX_BYTE(dword, 3)`。
- **L49 EN**: Comment explains nearby logic, constraints, or intent: `RETURN tmp`.
  **L49 CN**: 注释解释附近代码的逻辑、约束或设计意图：`RETURN tmp`。
- **L50 EN**: Comment explains nearby logic, constraints, or intent: `}`.
  **L50 CN**: 注释解释附近代码的逻辑、约束或设计意图：`}`。
- **L51 EN**: Comment explains nearby logic, constraints, or intent: `DEFINE L_KEY(dword) {`.
  **L51 CN**: 注释解释附近代码的逻辑、约束或设计意图：`DEFINE L_KEY(dword) {`。
- **L52 EN**: Comment explains nearby logic, constraints, or intent: `RETURN dword ^ ROL32(dword, 13) ^ ROL32(dword, 23)`.
  **L52 CN**: 注释解释附近代码的逻辑、约束或设计意图：`RETURN dword ^ ROL32(dword, 13) ^ ROL32(dword, 23)`。
- **L53 EN**: Comment explains nearby logic, constraints, or intent: `}`.
  **L53 CN**: 注释解释附近代码的逻辑、约束或设计意图：`}`。
- **L54 EN**: Comment explains nearby logic, constraints, or intent: `DEFINE T_KEY(dword) {`.
  **L54 CN**: 注释解释附近代码的逻辑、约束或设计意图：`DEFINE T_KEY(dword) {`。
- **L55 EN**: Comment explains nearby logic, constraints, or intent: `RETURN L_KEY(lower_t(dword))`.
  **L55 CN**: 注释解释附近代码的逻辑、约束或设计意图：`RETURN L_KEY(lower_t(dword))`。
- **L56 EN**: Comment explains nearby logic, constraints, or intent: `}`.
  **L56 CN**: 注释解释附近代码的逻辑、约束或设计意图：`}`。
- **L57 EN**: Comment explains nearby logic, constraints, or intent: `DEFINE F_KEY(X0, X1, X2, X3, round_key) {`.
  **L57 CN**: 注释解释附近代码的逻辑、约束或设计意图：`DEFINE F_KEY(X0, X1, X2, X3, round_key) {`。
- **L58 EN**: Comment explains nearby logic, constraints, or intent: `RETURN X0 ^ T_KEY(X1 ^ X2 ^ X3 ^ round_key)`.
  **L58 CN**: 注释解释附近代码的逻辑、约束或设计意图：`RETURN X0 ^ T_KEY(X1 ^ X2 ^ X3 ^ round_key)`。
- **L59 EN**: Comment explains nearby logic, constraints, or intent: `}`.
  **L59 CN**: 注释解释附近代码的逻辑、约束或设计意图：`}`。
- **L60 EN**: Comment explains nearby logic, constraints, or intent: `FOR i: 0 to 0`.
  **L60 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i: 0 to 0`。

### Lines 61-80

````c
/// 	P[0] := __B.xmm[i].dword[0]
/// 	P[1] := __B.xmm[i].dword[1]
/// 	P[2] := __B.xmm[i].dword[2]
/// 	P[3] := __B.xmm[i].dword[3]
/// 	C[0] := F_KEY(P[0], P[1], P[2], P[3], __A.xmm[i].dword[0])
/// 	C[1] := F_KEY(P[1], P[2], P[3], C[0], __A.xmm[i].dword[1])
/// 	C[2] := F_KEY(P[2], P[3], C[0], C[1], __A.xmm[i].dword[2])
/// 	C[3] := F_KEY(P[3], C[0], C[1], C[2], __A.xmm[i].dword[3])
/// 	DEST.xmm[i].dword[0] := C[0]
/// 	DEST.xmm[i].dword[1] := C[1]
/// 	DEST.xmm[i].dword[2] := C[2]
/// 	DEST.xmm[i].dword[3] := C[3]
/// ENDFOR
/// DEST[MAX:128] := 0
/// \endcode
#define _mm_sm4key4_epi32(A, B)                                                \
  (__m128i) __builtin_ia32_vsm4key4128((__v4su)A, (__v4su)B)

/// This intrinsic performs four rounds of SM4 key expansion. The intrinsic
///    operates on independent 128-bit lanes. The calculated results are
````
- **L61 EN**: Comment explains nearby logic, constraints, or intent: `P[0] : __B.xmm[i].dword[0]`.
  **L61 CN**: 注释解释附近代码的逻辑、约束或设计意图：`P[0] : __B.xmm[i].dword[0]`。
- **L62 EN**: Comment explains nearby logic, constraints, or intent: `P[1] : __B.xmm[i].dword[1]`.
  **L62 CN**: 注释解释附近代码的逻辑、约束或设计意图：`P[1] : __B.xmm[i].dword[1]`。
- **L63 EN**: Comment explains nearby logic, constraints, or intent: `P[2] : __B.xmm[i].dword[2]`.
  **L63 CN**: 注释解释附近代码的逻辑、约束或设计意图：`P[2] : __B.xmm[i].dword[2]`。
- **L64 EN**: Comment explains nearby logic, constraints, or intent: `P[3] : __B.xmm[i].dword[3]`.
  **L64 CN**: 注释解释附近代码的逻辑、约束或设计意图：`P[3] : __B.xmm[i].dword[3]`。
- **L65 EN**: Comment explains nearby logic, constraints, or intent: `C[0] : F_KEY(P[0], P[1], P[2], P[3], __A.xmm[i].dword[0])`.
  **L65 CN**: 注释解释附近代码的逻辑、约束或设计意图：`C[0] : F_KEY(P[0], P[1], P[2], P[3], __A.xmm[i].dword[0])`。
- **L66 EN**: Comment explains nearby logic, constraints, or intent: `C[1] : F_KEY(P[1], P[2], P[3], C[0], __A.xmm[i].dword[1])`.
  **L66 CN**: 注释解释附近代码的逻辑、约束或设计意图：`C[1] : F_KEY(P[1], P[2], P[3], C[0], __A.xmm[i].dword[1])`。
- **L67 EN**: Comment explains nearby logic, constraints, or intent: `C[2] : F_KEY(P[2], P[3], C[0], C[1], __A.xmm[i].dword[2])`.
  **L67 CN**: 注释解释附近代码的逻辑、约束或设计意图：`C[2] : F_KEY(P[2], P[3], C[0], C[1], __A.xmm[i].dword[2])`。
- **L68 EN**: Comment explains nearby logic, constraints, or intent: `C[3] : F_KEY(P[3], C[0], C[1], C[2], __A.xmm[i].dword[3])`.
  **L68 CN**: 注释解释附近代码的逻辑、约束或设计意图：`C[3] : F_KEY(P[3], C[0], C[1], C[2], __A.xmm[i].dword[3])`。
- **L69 EN**: Comment explains nearby logic, constraints, or intent: `DEST.xmm[i].dword[0] : C[0]`.
  **L69 CN**: 注释解释附近代码的逻辑、约束或设计意图：`DEST.xmm[i].dword[0] : C[0]`。
- **L70 EN**: Comment explains nearby logic, constraints, or intent: `DEST.xmm[i].dword[1] : C[1]`.
  **L70 CN**: 注释解释附近代码的逻辑、约束或设计意图：`DEST.xmm[i].dword[1] : C[1]`。
- **L71 EN**: Comment explains nearby logic, constraints, or intent: `DEST.xmm[i].dword[2] : C[2]`.
  **L71 CN**: 注释解释附近代码的逻辑、约束或设计意图：`DEST.xmm[i].dword[2] : C[2]`。
- **L72 EN**: Comment explains nearby logic, constraints, or intent: `DEST.xmm[i].dword[3] : C[3]`.
  **L72 CN**: 注释解释附近代码的逻辑、约束或设计意图：`DEST.xmm[i].dword[3] : C[3]`。
- **L73 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L73 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L74 EN**: Comment explains nearby logic, constraints, or intent: `DEST[MAX:128] : 0`.
  **L74 CN**: 注释解释附近代码的逻辑、约束或设计意图：`DEST[MAX:128] : 0`。
- **L75 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L75 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L76 EN**: Defines macro `_mm_sm4key4_epi32(A, B)` for conditional compilation, shorthand, or API generation.
  **L76 CN**: 定义宏 `_mm_sm4key4_epi32(A, B)`，用于条件编译、简写或 API 生成。
- **L77 EN**: Continues logic associated with callable symbol `__builtin_ia32_vsm4key4128`.
  **L77 CN**: 继续与可调用符号 `__builtin_ia32_vsm4key4128` 相关的逻辑。
- **L78 EN**: Blank line separating nearby declarations or logic blocks.
  **L78 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L79 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic performs four rounds of SM4 key expansion. The intrinsic`.
  **L79 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic performs four rounds of SM4 key expansion. The intrinsic`。
- **L80 EN**: Comment explains nearby logic, constraints, or intent: `operates on independent 128-bit lanes. The calculated results are`.
  **L80 CN**: 注释解释附近代码的逻辑、约束或设计意图：`operates on independent 128-bit lanes. The calculated results are`。

### Lines 81-100

````c
///    stored in \a dst.
/// \headerfile <immintrin.h>
///
/// \code
/// __m256i _mm256_sm4key4_epi32(__m256i __A, __m256i __B)
/// \endcode
///
/// This intrinsic corresponds to the \c VSM4KEY4 instruction.
///
/// \param __A
///    A 256-bit vector of [8 x int].
/// \param __B
///    A 256-bit vector of [8 x int].
/// \returns
///    A 256-bit vector of [8 x int].
///
/// \code{.operation}
/// DEFINE ROL32(dword, n) {
/// 	count := n % 32
/// 	dest := (dword << count) | (dword >> (32-count))
````
- **L81 EN**: Comment explains nearby logic, constraints, or intent: `stored in a dst.`.
  **L81 CN**: 注释解释附近代码的逻辑、约束或设计意图：`stored in a dst.`。
- **L82 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L82 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L83 EN**: Separator comment used for visual grouping.
  **L83 CN**: 用于视觉分组的分隔注释。
- **L84 EN**: Comment explains nearby logic, constraints, or intent: `code`.
  **L84 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code`。
- **L85 EN**: Comment explains nearby logic, constraints, or intent: `__m256i _mm256_sm4key4_epi32(__m256i __A, __m256i __B)`.
  **L85 CN**: 注释解释附近代码的逻辑、约束或设计意图：`__m256i _mm256_sm4key4_epi32(__m256i __A, __m256i __B)`。
- **L86 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L86 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L87 EN**: Separator comment used for visual grouping.
  **L87 CN**: 用于视觉分组的分隔注释。
- **L88 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VSM4KEY4 instruction.`.
  **L88 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VSM4KEY4 instruction.`。
- **L89 EN**: Separator comment used for visual grouping.
  **L89 CN**: 用于视觉分组的分隔注释。
- **L90 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L90 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L91 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [8 x int].`.
  **L91 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [8 x int].`。
- **L92 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L92 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L93 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [8 x int].`.
  **L93 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [8 x int].`。
- **L94 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L94 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L95 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [8 x int].`.
  **L95 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [8 x int].`。
- **L96 EN**: Separator comment used for visual grouping.
  **L96 CN**: 用于视觉分组的分隔注释。
- **L97 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L97 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L98 EN**: Comment explains nearby logic, constraints, or intent: `DEFINE ROL32(dword, n) {`.
  **L98 CN**: 注释解释附近代码的逻辑、约束或设计意图：`DEFINE ROL32(dword, n) {`。
- **L99 EN**: Comment explains nearby logic, constraints, or intent: `count : n % 32`.
  **L99 CN**: 注释解释附近代码的逻辑、约束或设计意图：`count : n % 32`。
- **L100 EN**: Comment explains nearby logic, constraints, or intent: `dest : (dword << count) | (dword >> (32-count))`.
  **L100 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dest : (dword << count) | (dword >> (32-count))`。

### Lines 101-120

````c
/// 	RETURN dest
/// }
/// DEFINE SBOX_BYTE(dword, i) {
/// 	RETURN sbox[dword.byte[i]]
/// }
/// DEFINE lower_t(dword) {
/// 	tmp.byte[0] := SBOX_BYTE(dword, 0)
/// 	tmp.byte[1] := SBOX_BYTE(dword, 1)
/// 	tmp.byte[2] := SBOX_BYTE(dword, 2)
/// 	tmp.byte[3] := SBOX_BYTE(dword, 3)
/// 	RETURN tmp
/// }
/// DEFINE L_KEY(dword) {
/// 	RETURN dword ^ ROL32(dword, 13) ^ ROL32(dword, 23)
/// }
/// DEFINE T_KEY(dword) {
/// 	RETURN L_KEY(lower_t(dword))
/// }
/// DEFINE F_KEY(X0, X1, X2, X3, round_key) {
/// 	RETURN X0 ^ T_KEY(X1 ^ X2 ^ X3 ^ round_key)
````
- **L101 EN**: Comment explains nearby logic, constraints, or intent: `RETURN dest`.
  **L101 CN**: 注释解释附近代码的逻辑、约束或设计意图：`RETURN dest`。
- **L102 EN**: Comment explains nearby logic, constraints, or intent: `}`.
  **L102 CN**: 注释解释附近代码的逻辑、约束或设计意图：`}`。
- **L103 EN**: Comment explains nearby logic, constraints, or intent: `DEFINE SBOX_BYTE(dword, i) {`.
  **L103 CN**: 注释解释附近代码的逻辑、约束或设计意图：`DEFINE SBOX_BYTE(dword, i) {`。
- **L104 EN**: Comment explains nearby logic, constraints, or intent: `RETURN sbox[dword.byte[i]]`.
  **L104 CN**: 注释解释附近代码的逻辑、约束或设计意图：`RETURN sbox[dword.byte[i]]`。
- **L105 EN**: Comment explains nearby logic, constraints, or intent: `}`.
  **L105 CN**: 注释解释附近代码的逻辑、约束或设计意图：`}`。
- **L106 EN**: Comment explains nearby logic, constraints, or intent: `DEFINE lower_t(dword) {`.
  **L106 CN**: 注释解释附近代码的逻辑、约束或设计意图：`DEFINE lower_t(dword) {`。
- **L107 EN**: Comment explains nearby logic, constraints, or intent: `tmp.byte[0] : SBOX_BYTE(dword, 0)`.
  **L107 CN**: 注释解释附近代码的逻辑、约束或设计意图：`tmp.byte[0] : SBOX_BYTE(dword, 0)`。
- **L108 EN**: Comment explains nearby logic, constraints, or intent: `tmp.byte[1] : SBOX_BYTE(dword, 1)`.
  **L108 CN**: 注释解释附近代码的逻辑、约束或设计意图：`tmp.byte[1] : SBOX_BYTE(dword, 1)`。
- **L109 EN**: Comment explains nearby logic, constraints, or intent: `tmp.byte[2] : SBOX_BYTE(dword, 2)`.
  **L109 CN**: 注释解释附近代码的逻辑、约束或设计意图：`tmp.byte[2] : SBOX_BYTE(dword, 2)`。
- **L110 EN**: Comment explains nearby logic, constraints, or intent: `tmp.byte[3] : SBOX_BYTE(dword, 3)`.
  **L110 CN**: 注释解释附近代码的逻辑、约束或设计意图：`tmp.byte[3] : SBOX_BYTE(dword, 3)`。
- **L111 EN**: Comment explains nearby logic, constraints, or intent: `RETURN tmp`.
  **L111 CN**: 注释解释附近代码的逻辑、约束或设计意图：`RETURN tmp`。
- **L112 EN**: Comment explains nearby logic, constraints, or intent: `}`.
  **L112 CN**: 注释解释附近代码的逻辑、约束或设计意图：`}`。
- **L113 EN**: Comment explains nearby logic, constraints, or intent: `DEFINE L_KEY(dword) {`.
  **L113 CN**: 注释解释附近代码的逻辑、约束或设计意图：`DEFINE L_KEY(dword) {`。
- **L114 EN**: Comment explains nearby logic, constraints, or intent: `RETURN dword ^ ROL32(dword, 13) ^ ROL32(dword, 23)`.
  **L114 CN**: 注释解释附近代码的逻辑、约束或设计意图：`RETURN dword ^ ROL32(dword, 13) ^ ROL32(dword, 23)`。
- **L115 EN**: Comment explains nearby logic, constraints, or intent: `}`.
  **L115 CN**: 注释解释附近代码的逻辑、约束或设计意图：`}`。
- **L116 EN**: Comment explains nearby logic, constraints, or intent: `DEFINE T_KEY(dword) {`.
  **L116 CN**: 注释解释附近代码的逻辑、约束或设计意图：`DEFINE T_KEY(dword) {`。
- **L117 EN**: Comment explains nearby logic, constraints, or intent: `RETURN L_KEY(lower_t(dword))`.
  **L117 CN**: 注释解释附近代码的逻辑、约束或设计意图：`RETURN L_KEY(lower_t(dword))`。
- **L118 EN**: Comment explains nearby logic, constraints, or intent: `}`.
  **L118 CN**: 注释解释附近代码的逻辑、约束或设计意图：`}`。
- **L119 EN**: Comment explains nearby logic, constraints, or intent: `DEFINE F_KEY(X0, X1, X2, X3, round_key) {`.
  **L119 CN**: 注释解释附近代码的逻辑、约束或设计意图：`DEFINE F_KEY(X0, X1, X2, X3, round_key) {`。
- **L120 EN**: Comment explains nearby logic, constraints, or intent: `RETURN X0 ^ T_KEY(X1 ^ X2 ^ X3 ^ round_key)`.
  **L120 CN**: 注释解释附近代码的逻辑、约束或设计意图：`RETURN X0 ^ T_KEY(X1 ^ X2 ^ X3 ^ round_key)`。

### Lines 121-140

````c
/// }
/// FOR i:= 0 to 1
/// 	P[0] := __B.xmm[i].dword[0]
/// 	P[1] := __B.xmm[i].dword[1]
/// 	P[2] := __B.xmm[i].dword[2]
/// 	P[3] := __B.xmm[i].dword[3]
/// 	C[0] := F_KEY(P[0], P[1], P[2], P[3], __A.xmm[i].dword[0])
/// 	C[1] := F_KEY(P[1], P[2], P[3], C[0], __A.xmm[i].dword[1])
/// 	C[2] := F_KEY(P[2], P[3], C[0], C[1], __A.xmm[i].dword[2])
/// 	C[3] := F_KEY(P[3], C[0], C[1], C[2], __A.xmm[i].dword[3])
/// 	DEST.xmm[i].dword[0] := C[0]
/// 	DEST.xmm[i].dword[1] := C[1]
/// 	DEST.xmm[i].dword[2] := C[2]
/// 	DEST.xmm[i].dword[3] := C[3]
/// ENDFOR
/// DEST[MAX:256] := 0
/// \endcode
#define _mm256_sm4key4_epi32(A, B)                                             \
  (__m256i) __builtin_ia32_vsm4key4256((__v8su)A, (__v8su)B)

````
- **L121 EN**: Comment explains nearby logic, constraints, or intent: `}`.
  **L121 CN**: 注释解释附近代码的逻辑、约束或设计意图：`}`。
- **L122 EN**: Comment explains nearby logic, constraints, or intent: `FOR i: 0 to 1`.
  **L122 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i: 0 to 1`。
- **L123 EN**: Comment explains nearby logic, constraints, or intent: `P[0] : __B.xmm[i].dword[0]`.
  **L123 CN**: 注释解释附近代码的逻辑、约束或设计意图：`P[0] : __B.xmm[i].dword[0]`。
- **L124 EN**: Comment explains nearby logic, constraints, or intent: `P[1] : __B.xmm[i].dword[1]`.
  **L124 CN**: 注释解释附近代码的逻辑、约束或设计意图：`P[1] : __B.xmm[i].dword[1]`。
- **L125 EN**: Comment explains nearby logic, constraints, or intent: `P[2] : __B.xmm[i].dword[2]`.
  **L125 CN**: 注释解释附近代码的逻辑、约束或设计意图：`P[2] : __B.xmm[i].dword[2]`。
- **L126 EN**: Comment explains nearby logic, constraints, or intent: `P[3] : __B.xmm[i].dword[3]`.
  **L126 CN**: 注释解释附近代码的逻辑、约束或设计意图：`P[3] : __B.xmm[i].dword[3]`。
- **L127 EN**: Comment explains nearby logic, constraints, or intent: `C[0] : F_KEY(P[0], P[1], P[2], P[3], __A.xmm[i].dword[0])`.
  **L127 CN**: 注释解释附近代码的逻辑、约束或设计意图：`C[0] : F_KEY(P[0], P[1], P[2], P[3], __A.xmm[i].dword[0])`。
- **L128 EN**: Comment explains nearby logic, constraints, or intent: `C[1] : F_KEY(P[1], P[2], P[3], C[0], __A.xmm[i].dword[1])`.
  **L128 CN**: 注释解释附近代码的逻辑、约束或设计意图：`C[1] : F_KEY(P[1], P[2], P[3], C[0], __A.xmm[i].dword[1])`。
- **L129 EN**: Comment explains nearby logic, constraints, or intent: `C[2] : F_KEY(P[2], P[3], C[0], C[1], __A.xmm[i].dword[2])`.
  **L129 CN**: 注释解释附近代码的逻辑、约束或设计意图：`C[2] : F_KEY(P[2], P[3], C[0], C[1], __A.xmm[i].dword[2])`。
- **L130 EN**: Comment explains nearby logic, constraints, or intent: `C[3] : F_KEY(P[3], C[0], C[1], C[2], __A.xmm[i].dword[3])`.
  **L130 CN**: 注释解释附近代码的逻辑、约束或设计意图：`C[3] : F_KEY(P[3], C[0], C[1], C[2], __A.xmm[i].dword[3])`。
- **L131 EN**: Comment explains nearby logic, constraints, or intent: `DEST.xmm[i].dword[0] : C[0]`.
  **L131 CN**: 注释解释附近代码的逻辑、约束或设计意图：`DEST.xmm[i].dword[0] : C[0]`。
- **L132 EN**: Comment explains nearby logic, constraints, or intent: `DEST.xmm[i].dword[1] : C[1]`.
  **L132 CN**: 注释解释附近代码的逻辑、约束或设计意图：`DEST.xmm[i].dword[1] : C[1]`。
- **L133 EN**: Comment explains nearby logic, constraints, or intent: `DEST.xmm[i].dword[2] : C[2]`.
  **L133 CN**: 注释解释附近代码的逻辑、约束或设计意图：`DEST.xmm[i].dword[2] : C[2]`。
- **L134 EN**: Comment explains nearby logic, constraints, or intent: `DEST.xmm[i].dword[3] : C[3]`.
  **L134 CN**: 注释解释附近代码的逻辑、约束或设计意图：`DEST.xmm[i].dword[3] : C[3]`。
- **L135 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L135 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L136 EN**: Comment explains nearby logic, constraints, or intent: `DEST[MAX:256] : 0`.
  **L136 CN**: 注释解释附近代码的逻辑、约束或设计意图：`DEST[MAX:256] : 0`。
- **L137 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L137 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L138 EN**: Defines macro `_mm256_sm4key4_epi32(A, B)` for conditional compilation, shorthand, or API generation.
  **L138 CN**: 定义宏 `_mm256_sm4key4_epi32(A, B)`，用于条件编译、简写或 API 生成。
- **L139 EN**: Continues logic associated with callable symbol `__builtin_ia32_vsm4key4256`.
  **L139 CN**: 继续与可调用符号 `__builtin_ia32_vsm4key4256` 相关的逻辑。
- **L140 EN**: Blank line separating nearby declarations or logic blocks.
  **L140 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 141-160

````c
/// This intrinisc performs four rounds of SM4 encryption. The intrinisc
///    operates on independent 128-bit lanes. The calculated results are
///    stored in \a dst.
/// \headerfile <immintrin.h>
///
/// \code
/// __m128i _mm_sm4rnds4_epi32(__m128i __A, __m128i __B)
/// \endcode
///
/// This intrinsic corresponds to the \c VSM4RNDS4 instruction.
///
/// \param __A
///    A 128-bit vector of [4 x int].
/// \param __B
///    A 128-bit vector of [4 x int].
/// \returns
///    A 128-bit vector of [4 x int].
///
/// \code{.operation}
/// DEFINE ROL32(dword, n) {
````
- **L141 EN**: Comment explains nearby logic, constraints, or intent: `This intrinisc performs four rounds of SM4 encryption. The intrinisc`.
  **L141 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinisc performs four rounds of SM4 encryption. The intrinisc`。
- **L142 EN**: Comment explains nearby logic, constraints, or intent: `operates on independent 128-bit lanes. The calculated results are`.
  **L142 CN**: 注释解释附近代码的逻辑、约束或设计意图：`operates on independent 128-bit lanes. The calculated results are`。
- **L143 EN**: Comment explains nearby logic, constraints, or intent: `stored in a dst.`.
  **L143 CN**: 注释解释附近代码的逻辑、约束或设计意图：`stored in a dst.`。
- **L144 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L144 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L145 EN**: Separator comment used for visual grouping.
  **L145 CN**: 用于视觉分组的分隔注释。
- **L146 EN**: Comment explains nearby logic, constraints, or intent: `code`.
  **L146 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code`。
- **L147 EN**: Comment explains nearby logic, constraints, or intent: `__m128i _mm_sm4rnds4_epi32(__m128i __A, __m128i __B)`.
  **L147 CN**: 注释解释附近代码的逻辑、约束或设计意图：`__m128i _mm_sm4rnds4_epi32(__m128i __A, __m128i __B)`。
- **L148 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L148 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L149 EN**: Separator comment used for visual grouping.
  **L149 CN**: 用于视觉分组的分隔注释。
- **L150 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VSM4RNDS4 instruction.`.
  **L150 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VSM4RNDS4 instruction.`。
- **L151 EN**: Separator comment used for visual grouping.
  **L151 CN**: 用于视觉分组的分隔注释。
- **L152 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L152 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L153 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x int].`.
  **L153 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x int].`。
- **L154 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L154 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L155 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x int].`.
  **L155 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x int].`。
- **L156 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L156 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。
- **L157 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [4 x int].`.
  **L157 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [4 x int].`。
- **L158 EN**: Separator comment used for visual grouping.
  **L158 CN**: 用于视觉分组的分隔注释。
- **L159 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L159 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L160 EN**: Comment explains nearby logic, constraints, or intent: `DEFINE ROL32(dword, n) {`.
  **L160 CN**: 注释解释附近代码的逻辑、约束或设计意图：`DEFINE ROL32(dword, n) {`。

### Lines 161-180

````c
/// 	count := n % 32
/// 	dest := (dword << count) | (dword >> (32-count))
/// 	RETURN dest
/// }
/// DEFINE lower_t(dword) {
/// 	tmp.byte[0] := SBOX_BYTE(dword, 0)
/// 	tmp.byte[1] := SBOX_BYTE(dword, 1)
/// 	tmp.byte[2] := SBOX_BYTE(dword, 2)
/// 	tmp.byte[3] := SBOX_BYTE(dword, 3)
/// 	RETURN tmp
/// }
/// DEFINE L_RND(dword) {
/// 	tmp := dword
/// 	tmp := tmp ^ ROL32(dword, 2)
/// 	tmp := tmp ^ ROL32(dword, 10)
/// 	tmp := tmp ^ ROL32(dword, 18)
/// 	tmp := tmp ^ ROL32(dword, 24)
///   RETURN tmp
/// }
/// DEFINE T_RND(dword) {
````
- **L161 EN**: Comment explains nearby logic, constraints, or intent: `count : n % 32`.
  **L161 CN**: 注释解释附近代码的逻辑、约束或设计意图：`count : n % 32`。
- **L162 EN**: Comment explains nearby logic, constraints, or intent: `dest : (dword << count) | (dword >> (32-count))`.
  **L162 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dest : (dword << count) | (dword >> (32-count))`。
- **L163 EN**: Comment explains nearby logic, constraints, or intent: `RETURN dest`.
  **L163 CN**: 注释解释附近代码的逻辑、约束或设计意图：`RETURN dest`。
- **L164 EN**: Comment explains nearby logic, constraints, or intent: `}`.
  **L164 CN**: 注释解释附近代码的逻辑、约束或设计意图：`}`。
- **L165 EN**: Comment explains nearby logic, constraints, or intent: `DEFINE lower_t(dword) {`.
  **L165 CN**: 注释解释附近代码的逻辑、约束或设计意图：`DEFINE lower_t(dword) {`。
- **L166 EN**: Comment explains nearby logic, constraints, or intent: `tmp.byte[0] : SBOX_BYTE(dword, 0)`.
  **L166 CN**: 注释解释附近代码的逻辑、约束或设计意图：`tmp.byte[0] : SBOX_BYTE(dword, 0)`。
- **L167 EN**: Comment explains nearby logic, constraints, or intent: `tmp.byte[1] : SBOX_BYTE(dword, 1)`.
  **L167 CN**: 注释解释附近代码的逻辑、约束或设计意图：`tmp.byte[1] : SBOX_BYTE(dword, 1)`。
- **L168 EN**: Comment explains nearby logic, constraints, or intent: `tmp.byte[2] : SBOX_BYTE(dword, 2)`.
  **L168 CN**: 注释解释附近代码的逻辑、约束或设计意图：`tmp.byte[2] : SBOX_BYTE(dword, 2)`。
- **L169 EN**: Comment explains nearby logic, constraints, or intent: `tmp.byte[3] : SBOX_BYTE(dword, 3)`.
  **L169 CN**: 注释解释附近代码的逻辑、约束或设计意图：`tmp.byte[3] : SBOX_BYTE(dword, 3)`。
- **L170 EN**: Comment explains nearby logic, constraints, or intent: `RETURN tmp`.
  **L170 CN**: 注释解释附近代码的逻辑、约束或设计意图：`RETURN tmp`。
- **L171 EN**: Comment explains nearby logic, constraints, or intent: `}`.
  **L171 CN**: 注释解释附近代码的逻辑、约束或设计意图：`}`。
- **L172 EN**: Comment explains nearby logic, constraints, or intent: `DEFINE L_RND(dword) {`.
  **L172 CN**: 注释解释附近代码的逻辑、约束或设计意图：`DEFINE L_RND(dword) {`。
- **L173 EN**: Comment explains nearby logic, constraints, or intent: `tmp : dword`.
  **L173 CN**: 注释解释附近代码的逻辑、约束或设计意图：`tmp : dword`。
- **L174 EN**: Comment explains nearby logic, constraints, or intent: `tmp : tmp ^ ROL32(dword, 2)`.
  **L174 CN**: 注释解释附近代码的逻辑、约束或设计意图：`tmp : tmp ^ ROL32(dword, 2)`。
- **L175 EN**: Comment explains nearby logic, constraints, or intent: `tmp : tmp ^ ROL32(dword, 10)`.
  **L175 CN**: 注释解释附近代码的逻辑、约束或设计意图：`tmp : tmp ^ ROL32(dword, 10)`。
- **L176 EN**: Comment explains nearby logic, constraints, or intent: `tmp : tmp ^ ROL32(dword, 18)`.
  **L176 CN**: 注释解释附近代码的逻辑、约束或设计意图：`tmp : tmp ^ ROL32(dword, 18)`。
- **L177 EN**: Comment explains nearby logic, constraints, or intent: `tmp : tmp ^ ROL32(dword, 24)`.
  **L177 CN**: 注释解释附近代码的逻辑、约束或设计意图：`tmp : tmp ^ ROL32(dword, 24)`。
- **L178 EN**: Comment explains nearby logic, constraints, or intent: `RETURN tmp`.
  **L178 CN**: 注释解释附近代码的逻辑、约束或设计意图：`RETURN tmp`。
- **L179 EN**: Comment explains nearby logic, constraints, or intent: `}`.
  **L179 CN**: 注释解释附近代码的逻辑、约束或设计意图：`}`。
- **L180 EN**: Comment explains nearby logic, constraints, or intent: `DEFINE T_RND(dword) {`.
  **L180 CN**: 注释解释附近代码的逻辑、约束或设计意图：`DEFINE T_RND(dword) {`。

### Lines 181-200

````c
/// 	RETURN L_RND(lower_t(dword))
/// }
/// DEFINE F_RND(X0, X1, X2, X3, round_key) {
/// 	RETURN X0 ^ T_RND(X1 ^ X2 ^ X3 ^ round_key)
/// }
/// FOR i:= 0 to 0
/// 	P[0] := __B.xmm[i].dword[0]
/// 	P[1] := __B.xmm[i].dword[1]
/// 	P[2] := __B.xmm[i].dword[2]
/// 	P[3] := __B.xmm[i].dword[3]
/// 	C[0] := F_RND(P[0], P[1], P[2], P[3], __A.xmm[i].dword[0])
/// 	C[1] := F_RND(P[1], P[2], P[3], C[0], __A.xmm[i].dword[1])
/// 	C[2] := F_RND(P[2], P[3], C[0], C[1], __A.xmm[i].dword[2])
/// 	C[3] := F_RND(P[3], C[0], C[1], C[2], __A.xmm[i].dword[3])
/// 	DEST.xmm[i].dword[0] := C[0]
/// 	DEST.xmm[i].dword[1] := C[1]
/// 	DEST.xmm[i].dword[2] := C[2]
/// 	DEST.xmm[i].dword[3] := C[3]
/// ENDFOR
/// DEST[MAX:128] := 0
````
- **L181 EN**: Comment explains nearby logic, constraints, or intent: `RETURN L_RND(lower_t(dword))`.
  **L181 CN**: 注释解释附近代码的逻辑、约束或设计意图：`RETURN L_RND(lower_t(dword))`。
- **L182 EN**: Comment explains nearby logic, constraints, or intent: `}`.
  **L182 CN**: 注释解释附近代码的逻辑、约束或设计意图：`}`。
- **L183 EN**: Comment explains nearby logic, constraints, or intent: `DEFINE F_RND(X0, X1, X2, X3, round_key) {`.
  **L183 CN**: 注释解释附近代码的逻辑、约束或设计意图：`DEFINE F_RND(X0, X1, X2, X3, round_key) {`。
- **L184 EN**: Comment explains nearby logic, constraints, or intent: `RETURN X0 ^ T_RND(X1 ^ X2 ^ X3 ^ round_key)`.
  **L184 CN**: 注释解释附近代码的逻辑、约束或设计意图：`RETURN X0 ^ T_RND(X1 ^ X2 ^ X3 ^ round_key)`。
- **L185 EN**: Comment explains nearby logic, constraints, or intent: `}`.
  **L185 CN**: 注释解释附近代码的逻辑、约束或设计意图：`}`。
- **L186 EN**: Comment explains nearby logic, constraints, or intent: `FOR i: 0 to 0`.
  **L186 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i: 0 to 0`。
- **L187 EN**: Comment explains nearby logic, constraints, or intent: `P[0] : __B.xmm[i].dword[0]`.
  **L187 CN**: 注释解释附近代码的逻辑、约束或设计意图：`P[0] : __B.xmm[i].dword[0]`。
- **L188 EN**: Comment explains nearby logic, constraints, or intent: `P[1] : __B.xmm[i].dword[1]`.
  **L188 CN**: 注释解释附近代码的逻辑、约束或设计意图：`P[1] : __B.xmm[i].dword[1]`。
- **L189 EN**: Comment explains nearby logic, constraints, or intent: `P[2] : __B.xmm[i].dword[2]`.
  **L189 CN**: 注释解释附近代码的逻辑、约束或设计意图：`P[2] : __B.xmm[i].dword[2]`。
- **L190 EN**: Comment explains nearby logic, constraints, or intent: `P[3] : __B.xmm[i].dword[3]`.
  **L190 CN**: 注释解释附近代码的逻辑、约束或设计意图：`P[3] : __B.xmm[i].dword[3]`。
- **L191 EN**: Comment explains nearby logic, constraints, or intent: `C[0] : F_RND(P[0], P[1], P[2], P[3], __A.xmm[i].dword[0])`.
  **L191 CN**: 注释解释附近代码的逻辑、约束或设计意图：`C[0] : F_RND(P[0], P[1], P[2], P[3], __A.xmm[i].dword[0])`。
- **L192 EN**: Comment explains nearby logic, constraints, or intent: `C[1] : F_RND(P[1], P[2], P[3], C[0], __A.xmm[i].dword[1])`.
  **L192 CN**: 注释解释附近代码的逻辑、约束或设计意图：`C[1] : F_RND(P[1], P[2], P[3], C[0], __A.xmm[i].dword[1])`。
- **L193 EN**: Comment explains nearby logic, constraints, or intent: `C[2] : F_RND(P[2], P[3], C[0], C[1], __A.xmm[i].dword[2])`.
  **L193 CN**: 注释解释附近代码的逻辑、约束或设计意图：`C[2] : F_RND(P[2], P[3], C[0], C[1], __A.xmm[i].dword[2])`。
- **L194 EN**: Comment explains nearby logic, constraints, or intent: `C[3] : F_RND(P[3], C[0], C[1], C[2], __A.xmm[i].dword[3])`.
  **L194 CN**: 注释解释附近代码的逻辑、约束或设计意图：`C[3] : F_RND(P[3], C[0], C[1], C[2], __A.xmm[i].dword[3])`。
- **L195 EN**: Comment explains nearby logic, constraints, or intent: `DEST.xmm[i].dword[0] : C[0]`.
  **L195 CN**: 注释解释附近代码的逻辑、约束或设计意图：`DEST.xmm[i].dword[0] : C[0]`。
- **L196 EN**: Comment explains nearby logic, constraints, or intent: `DEST.xmm[i].dword[1] : C[1]`.
  **L196 CN**: 注释解释附近代码的逻辑、约束或设计意图：`DEST.xmm[i].dword[1] : C[1]`。
- **L197 EN**: Comment explains nearby logic, constraints, or intent: `DEST.xmm[i].dword[2] : C[2]`.
  **L197 CN**: 注释解释附近代码的逻辑、约束或设计意图：`DEST.xmm[i].dword[2] : C[2]`。
- **L198 EN**: Comment explains nearby logic, constraints, or intent: `DEST.xmm[i].dword[3] : C[3]`.
  **L198 CN**: 注释解释附近代码的逻辑、约束或设计意图：`DEST.xmm[i].dword[3] : C[3]`。
- **L199 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L199 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L200 EN**: Comment explains nearby logic, constraints, or intent: `DEST[MAX:128] : 0`.
  **L200 CN**: 注释解释附近代码的逻辑、约束或设计意图：`DEST[MAX:128] : 0`。

### Lines 201-220

````c
/// \endcode
#define _mm_sm4rnds4_epi32(A, B)                                               \
  (__m128i) __builtin_ia32_vsm4rnds4128((__v4su)A, (__v4su)B)

/// This intrinisc performs four rounds of SM4 encryption. The intrinisc
///    operates on independent 128-bit lanes. The calculated results are
///    stored in \a dst.
/// \headerfile <immintrin.h>
///
/// \code
/// __m256i _mm256_sm4rnds4_epi32(__m256i __A, __m256i __B)
/// \endcode
///
/// This intrinsic corresponds to the \c VSM4RNDS4 instruction.
///
/// \param __A
///    A 256-bit vector of [8 x int].
/// \param __B
///    A 256-bit vector of [8 x int].
/// \returns
````
- **L201 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L201 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L202 EN**: Defines macro `_mm_sm4rnds4_epi32(A, B)` for conditional compilation, shorthand, or API generation.
  **L202 CN**: 定义宏 `_mm_sm4rnds4_epi32(A, B)`，用于条件编译、简写或 API 生成。
- **L203 EN**: Continues logic associated with callable symbol `__builtin_ia32_vsm4rnds4128`.
  **L203 CN**: 继续与可调用符号 `__builtin_ia32_vsm4rnds4128` 相关的逻辑。
- **L204 EN**: Blank line separating nearby declarations or logic blocks.
  **L204 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L205 EN**: Comment explains nearby logic, constraints, or intent: `This intrinisc performs four rounds of SM4 encryption. The intrinisc`.
  **L205 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinisc performs four rounds of SM4 encryption. The intrinisc`。
- **L206 EN**: Comment explains nearby logic, constraints, or intent: `operates on independent 128-bit lanes. The calculated results are`.
  **L206 CN**: 注释解释附近代码的逻辑、约束或设计意图：`operates on independent 128-bit lanes. The calculated results are`。
- **L207 EN**: Comment explains nearby logic, constraints, or intent: `stored in a dst.`.
  **L207 CN**: 注释解释附近代码的逻辑、约束或设计意图：`stored in a dst.`。
- **L208 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <immintrin.h>`.
  **L208 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <immintrin.h>`。
- **L209 EN**: Separator comment used for visual grouping.
  **L209 CN**: 用于视觉分组的分隔注释。
- **L210 EN**: Comment explains nearby logic, constraints, or intent: `code`.
  **L210 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code`。
- **L211 EN**: Comment explains nearby logic, constraints, or intent: `__m256i _mm256_sm4rnds4_epi32(__m256i __A, __m256i __B)`.
  **L211 CN**: 注释解释附近代码的逻辑、约束或设计意图：`__m256i _mm256_sm4rnds4_epi32(__m256i __A, __m256i __B)`。
- **L212 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L212 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L213 EN**: Separator comment used for visual grouping.
  **L213 CN**: 用于视觉分组的分隔注释。
- **L214 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic corresponds to the c VSM4RNDS4 instruction.`.
  **L214 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic corresponds to the c VSM4RNDS4 instruction.`。
- **L215 EN**: Separator comment used for visual grouping.
  **L215 CN**: 用于视觉分组的分隔注释。
- **L216 EN**: Comment explains nearby logic, constraints, or intent: `param __A`.
  **L216 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __A`。
- **L217 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [8 x int].`.
  **L217 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [8 x int].`。
- **L218 EN**: Comment explains nearby logic, constraints, or intent: `param __B`.
  **L218 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __B`。
- **L219 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [8 x int].`.
  **L219 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [8 x int].`。
- **L220 EN**: Comment explains nearby logic, constraints, or intent: `returns`.
  **L220 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns`。

### Lines 221-240

````c
///    A 256-bit vector of [8 x int].
///
/// \code{.operation}
/// DEFINE ROL32(dword, n) {
/// 	count := n % 32
/// 	dest := (dword << count) | (dword >> (32-count))
/// 	RETURN dest
/// }
/// DEFINE lower_t(dword) {
/// 	tmp.byte[0] := SBOX_BYTE(dword, 0)
/// 	tmp.byte[1] := SBOX_BYTE(dword, 1)
/// 	tmp.byte[2] := SBOX_BYTE(dword, 2)
/// 	tmp.byte[3] := SBOX_BYTE(dword, 3)
/// 	RETURN tmp
/// }
/// DEFINE L_RND(dword) {
/// 	tmp := dword
/// 	tmp := tmp ^ ROL32(dword, 2)
/// 	tmp := tmp ^ ROL32(dword, 10)
/// 	tmp := tmp ^ ROL32(dword, 18)
````
- **L221 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [8 x int].`.
  **L221 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [8 x int].`。
- **L222 EN**: Separator comment used for visual grouping.
  **L222 CN**: 用于视觉分组的分隔注释。
- **L223 EN**: Comment explains nearby logic, constraints, or intent: `code{.operation}`.
  **L223 CN**: 注释解释附近代码的逻辑、约束或设计意图：`code{.operation}`。
- **L224 EN**: Comment explains nearby logic, constraints, or intent: `DEFINE ROL32(dword, n) {`.
  **L224 CN**: 注释解释附近代码的逻辑、约束或设计意图：`DEFINE ROL32(dword, n) {`。
- **L225 EN**: Comment explains nearby logic, constraints, or intent: `count : n % 32`.
  **L225 CN**: 注释解释附近代码的逻辑、约束或设计意图：`count : n % 32`。
- **L226 EN**: Comment explains nearby logic, constraints, or intent: `dest : (dword << count) | (dword >> (32-count))`.
  **L226 CN**: 注释解释附近代码的逻辑、约束或设计意图：`dest : (dword << count) | (dword >> (32-count))`。
- **L227 EN**: Comment explains nearby logic, constraints, or intent: `RETURN dest`.
  **L227 CN**: 注释解释附近代码的逻辑、约束或设计意图：`RETURN dest`。
- **L228 EN**: Comment explains nearby logic, constraints, or intent: `}`.
  **L228 CN**: 注释解释附近代码的逻辑、约束或设计意图：`}`。
- **L229 EN**: Comment explains nearby logic, constraints, or intent: `DEFINE lower_t(dword) {`.
  **L229 CN**: 注释解释附近代码的逻辑、约束或设计意图：`DEFINE lower_t(dword) {`。
- **L230 EN**: Comment explains nearby logic, constraints, or intent: `tmp.byte[0] : SBOX_BYTE(dword, 0)`.
  **L230 CN**: 注释解释附近代码的逻辑、约束或设计意图：`tmp.byte[0] : SBOX_BYTE(dword, 0)`。
- **L231 EN**: Comment explains nearby logic, constraints, or intent: `tmp.byte[1] : SBOX_BYTE(dword, 1)`.
  **L231 CN**: 注释解释附近代码的逻辑、约束或设计意图：`tmp.byte[1] : SBOX_BYTE(dword, 1)`。
- **L232 EN**: Comment explains nearby logic, constraints, or intent: `tmp.byte[2] : SBOX_BYTE(dword, 2)`.
  **L232 CN**: 注释解释附近代码的逻辑、约束或设计意图：`tmp.byte[2] : SBOX_BYTE(dword, 2)`。
- **L233 EN**: Comment explains nearby logic, constraints, or intent: `tmp.byte[3] : SBOX_BYTE(dword, 3)`.
  **L233 CN**: 注释解释附近代码的逻辑、约束或设计意图：`tmp.byte[3] : SBOX_BYTE(dword, 3)`。
- **L234 EN**: Comment explains nearby logic, constraints, or intent: `RETURN tmp`.
  **L234 CN**: 注释解释附近代码的逻辑、约束或设计意图：`RETURN tmp`。
- **L235 EN**: Comment explains nearby logic, constraints, or intent: `}`.
  **L235 CN**: 注释解释附近代码的逻辑、约束或设计意图：`}`。
- **L236 EN**: Comment explains nearby logic, constraints, or intent: `DEFINE L_RND(dword) {`.
  **L236 CN**: 注释解释附近代码的逻辑、约束或设计意图：`DEFINE L_RND(dword) {`。
- **L237 EN**: Comment explains nearby logic, constraints, or intent: `tmp : dword`.
  **L237 CN**: 注释解释附近代码的逻辑、约束或设计意图：`tmp : dword`。
- **L238 EN**: Comment explains nearby logic, constraints, or intent: `tmp : tmp ^ ROL32(dword, 2)`.
  **L238 CN**: 注释解释附近代码的逻辑、约束或设计意图：`tmp : tmp ^ ROL32(dword, 2)`。
- **L239 EN**: Comment explains nearby logic, constraints, or intent: `tmp : tmp ^ ROL32(dword, 10)`.
  **L239 CN**: 注释解释附近代码的逻辑、约束或设计意图：`tmp : tmp ^ ROL32(dword, 10)`。
- **L240 EN**: Comment explains nearby logic, constraints, or intent: `tmp : tmp ^ ROL32(dword, 18)`.
  **L240 CN**: 注释解释附近代码的逻辑、约束或设计意图：`tmp : tmp ^ ROL32(dword, 18)`。

### Lines 241-260

````c
/// 	tmp := tmp ^ ROL32(dword, 24)
///   RETURN tmp
/// }
/// DEFINE T_RND(dword) {
/// 	RETURN L_RND(lower_t(dword))
/// }
/// DEFINE F_RND(X0, X1, X2, X3, round_key) {
/// 	RETURN X0 ^ T_RND(X1 ^ X2 ^ X3 ^ round_key)
/// }
/// FOR i:= 0 to 0
/// 	P[0] := __B.xmm[i].dword[0]
/// 	P[1] := __B.xmm[i].dword[1]
/// 	P[2] := __B.xmm[i].dword[2]
/// 	P[3] := __B.xmm[i].dword[3]
/// 	C[0] := F_RND(P[0], P[1], P[2], P[3], __A.xmm[i].dword[0])
/// 	C[1] := F_RND(P[1], P[2], P[3], C[0], __A.xmm[i].dword[1])
/// 	C[2] := F_RND(P[2], P[3], C[0], C[1], __A.xmm[i].dword[2])
/// 	C[3] := F_RND(P[3], C[0], C[1], C[2], __A.xmm[i].dword[3])
/// 	DEST.xmm[i].dword[0] := C[0]
/// 	DEST.xmm[i].dword[1] := C[1]
````
- **L241 EN**: Comment explains nearby logic, constraints, or intent: `tmp : tmp ^ ROL32(dword, 24)`.
  **L241 CN**: 注释解释附近代码的逻辑、约束或设计意图：`tmp : tmp ^ ROL32(dword, 24)`。
- **L242 EN**: Comment explains nearby logic, constraints, or intent: `RETURN tmp`.
  **L242 CN**: 注释解释附近代码的逻辑、约束或设计意图：`RETURN tmp`。
- **L243 EN**: Comment explains nearby logic, constraints, or intent: `}`.
  **L243 CN**: 注释解释附近代码的逻辑、约束或设计意图：`}`。
- **L244 EN**: Comment explains nearby logic, constraints, or intent: `DEFINE T_RND(dword) {`.
  **L244 CN**: 注释解释附近代码的逻辑、约束或设计意图：`DEFINE T_RND(dword) {`。
- **L245 EN**: Comment explains nearby logic, constraints, or intent: `RETURN L_RND(lower_t(dword))`.
  **L245 CN**: 注释解释附近代码的逻辑、约束或设计意图：`RETURN L_RND(lower_t(dword))`。
- **L246 EN**: Comment explains nearby logic, constraints, or intent: `}`.
  **L246 CN**: 注释解释附近代码的逻辑、约束或设计意图：`}`。
- **L247 EN**: Comment explains nearby logic, constraints, or intent: `DEFINE F_RND(X0, X1, X2, X3, round_key) {`.
  **L247 CN**: 注释解释附近代码的逻辑、约束或设计意图：`DEFINE F_RND(X0, X1, X2, X3, round_key) {`。
- **L248 EN**: Comment explains nearby logic, constraints, or intent: `RETURN X0 ^ T_RND(X1 ^ X2 ^ X3 ^ round_key)`.
  **L248 CN**: 注释解释附近代码的逻辑、约束或设计意图：`RETURN X0 ^ T_RND(X1 ^ X2 ^ X3 ^ round_key)`。
- **L249 EN**: Comment explains nearby logic, constraints, or intent: `}`.
  **L249 CN**: 注释解释附近代码的逻辑、约束或设计意图：`}`。
- **L250 EN**: Comment explains nearby logic, constraints, or intent: `FOR i: 0 to 0`.
  **L250 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FOR i: 0 to 0`。
- **L251 EN**: Comment explains nearby logic, constraints, or intent: `P[0] : __B.xmm[i].dword[0]`.
  **L251 CN**: 注释解释附近代码的逻辑、约束或设计意图：`P[0] : __B.xmm[i].dword[0]`。
- **L252 EN**: Comment explains nearby logic, constraints, or intent: `P[1] : __B.xmm[i].dword[1]`.
  **L252 CN**: 注释解释附近代码的逻辑、约束或设计意图：`P[1] : __B.xmm[i].dword[1]`。
- **L253 EN**: Comment explains nearby logic, constraints, or intent: `P[2] : __B.xmm[i].dword[2]`.
  **L253 CN**: 注释解释附近代码的逻辑、约束或设计意图：`P[2] : __B.xmm[i].dword[2]`。
- **L254 EN**: Comment explains nearby logic, constraints, or intent: `P[3] : __B.xmm[i].dword[3]`.
  **L254 CN**: 注释解释附近代码的逻辑、约束或设计意图：`P[3] : __B.xmm[i].dword[3]`。
- **L255 EN**: Comment explains nearby logic, constraints, or intent: `C[0] : F_RND(P[0], P[1], P[2], P[3], __A.xmm[i].dword[0])`.
  **L255 CN**: 注释解释附近代码的逻辑、约束或设计意图：`C[0] : F_RND(P[0], P[1], P[2], P[3], __A.xmm[i].dword[0])`。
- **L256 EN**: Comment explains nearby logic, constraints, or intent: `C[1] : F_RND(P[1], P[2], P[3], C[0], __A.xmm[i].dword[1])`.
  **L256 CN**: 注释解释附近代码的逻辑、约束或设计意图：`C[1] : F_RND(P[1], P[2], P[3], C[0], __A.xmm[i].dword[1])`。
- **L257 EN**: Comment explains nearby logic, constraints, or intent: `C[2] : F_RND(P[2], P[3], C[0], C[1], __A.xmm[i].dword[2])`.
  **L257 CN**: 注释解释附近代码的逻辑、约束或设计意图：`C[2] : F_RND(P[2], P[3], C[0], C[1], __A.xmm[i].dword[2])`。
- **L258 EN**: Comment explains nearby logic, constraints, or intent: `C[3] : F_RND(P[3], C[0], C[1], C[2], __A.xmm[i].dword[3])`.
  **L258 CN**: 注释解释附近代码的逻辑、约束或设计意图：`C[3] : F_RND(P[3], C[0], C[1], C[2], __A.xmm[i].dword[3])`。
- **L259 EN**: Comment explains nearby logic, constraints, or intent: `DEST.xmm[i].dword[0] : C[0]`.
  **L259 CN**: 注释解释附近代码的逻辑、约束或设计意图：`DEST.xmm[i].dword[0] : C[0]`。
- **L260 EN**: Comment explains nearby logic, constraints, or intent: `DEST.xmm[i].dword[1] : C[1]`.
  **L260 CN**: 注释解释附近代码的逻辑、约束或设计意图：`DEST.xmm[i].dword[1] : C[1]`。

### Lines 261-269

````c
/// 	DEST.xmm[i].dword[2] := C[2]
/// 	DEST.xmm[i].dword[3] := C[3]
/// ENDFOR
/// DEST[MAX:256] := 0
/// \endcode
#define _mm256_sm4rnds4_epi32(A, B)                                            \
  (__m256i) __builtin_ia32_vsm4rnds4256((__v8su)A, (__v8su)B)

#endif // __SM4INTRIN_H
````
- **L261 EN**: Comment explains nearby logic, constraints, or intent: `DEST.xmm[i].dword[2] : C[2]`.
  **L261 CN**: 注释解释附近代码的逻辑、约束或设计意图：`DEST.xmm[i].dword[2] : C[2]`。
- **L262 EN**: Comment explains nearby logic, constraints, or intent: `DEST.xmm[i].dword[3] : C[3]`.
  **L262 CN**: 注释解释附近代码的逻辑、约束或设计意图：`DEST.xmm[i].dword[3] : C[3]`。
- **L263 EN**: Comment explains nearby logic, constraints, or intent: `ENDFOR`.
  **L263 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ENDFOR`。
- **L264 EN**: Comment explains nearby logic, constraints, or intent: `DEST[MAX:256] : 0`.
  **L264 CN**: 注释解释附近代码的逻辑、约束或设计意图：`DEST[MAX:256] : 0`。
- **L265 EN**: Comment explains nearby logic, constraints, or intent: `endcode`.
  **L265 CN**: 注释解释附近代码的逻辑、约束或设计意图：`endcode`。
- **L266 EN**: Defines macro `_mm256_sm4rnds4_epi32(A, B)` for conditional compilation, shorthand, or API generation.
  **L266 CN**: 定义宏 `_mm256_sm4rnds4_epi32(A, B)`，用于条件编译、简写或 API 生成。
- **L267 EN**: Continues logic associated with callable symbol `__builtin_ia32_vsm4rnds4256`.
  **L267 CN**: 继续与可调用符号 `__builtin_ia32_vsm4rnds4256` 相关的逻辑。
- **L268 EN**: Blank line separating nearby declarations or logic blocks.
  **L268 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L269 EN**: Closes the current preprocessor conditional block.
  **L269 CN**: 结束当前预处理条件块。

## Key Concepts / 关键概念

- **Clang public headers / Clang 公共头文件**
- **Compiler builtin forwarding / 编译器 builtin 转发**
- **Macro-based API construction / 基于宏的 API 构造**
- **Header inclusion control / 头文件包含控制**

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: None. / 无。
- **Conditional macros / 条件宏**: `__IMMINTRIN_H`, `__SM4INTRIN_H`
- **External builtins / 外部 builtin**: `__builtin_ia32_vsm4key4128`, `__builtin_ia32_vsm4key4256`, `__builtin_ia32_vsm4rnds4128`, `__builtin_ia32_vsm4rnds4256`
