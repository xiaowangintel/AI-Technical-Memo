# fcmp.h — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `compiler-rt/lib/builtins/arm/fcmp.h`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose / 目的**:
  - **EN**: This code is the skeleton of a double-precision FP compare, with two details left out: which input value is in which register, and how to make the return value. It allows the main comparison logic to be shared between (for example) __lesf2 .
  - **CN**: 实现 compiler-rt 内建运行时例程 `fcmp`，用于补足目标平台或编译器生成代码所需的基础运算。

## Line-by-Line Analysis / 逐行分析

### Lines 1-10 / 第 1-10 行
```cpp
   1 | //===-- fcmp.h - shared code for single-precision FP comparison functions -===//
   2 | //
   3 | // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4 | // See https://llvm.org/LICENSE.txt for license information.
   5 | // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6 | //
   7 | //===----------------------------------------------------------------------===//
   8 | //
   9 | // This code is the skeleton of a double-precision FP compare, with two details
  10 | // left out: which input value is in which register, and how to make the return
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

### Lines 11-20 / 第 11-20 行
```cpp
  11 | // value. It allows the main comparison logic to be shared between (for
  12 | // example) __lesf2 and __gesf2, varying only those details.
  13 | //
  14 | //===----------------------------------------------------------------------===//
  15 | 
  16 | // How to use this header file:
  17 | //
  18 | // This header file is expected to be #included from inside a function
  19 | // definition in a .S file. The source file including this header should
  20 | // provide the following:
```
- **Line 11 / 第 11 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 12 / 第 12 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 13 / 第 13 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 14 / 第 14 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 15 / 第 15 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 16 / 第 16 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 17 / 第 17 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 18 / 第 18 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 19 / 第 19 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 20 / 第 20 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。

### Lines 21-30 / 第 21-30 行
```cpp
  21 | //
  22 | // op0 and op1: register aliases (via .req) for the registers containing the
  23 | // input operands.
  24 | //  - For most comparisons, op0 will correspond to r0 and op1 to r1.
  25 | //  - But a function with the reversed semantics of __aeabi_cfrcmple wil define
  26 | //    them the other way round.
  27 | //
  28 | // SetReturnRegister: an assembly macro that looks at the PSR flags and sets up
  29 | // an appropriate return value in r0, for the cases that do *not* involve NaN.
  30 | //  - On entry to this macro, the condition codes LO, EQ and HI indicate that
```
- **Line 21 / 第 21 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 22 / 第 22 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 23 / 第 23 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 24 / 第 24 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 25 / 第 25 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 26 / 第 26 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 27 / 第 27 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 28 / 第 28 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 29 / 第 29 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 30 / 第 30 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。

### Lines 31-40 / 第 31-40 行
```cpp
  31 | //    op0 < op1, op0 == op1 or op0 > op1 respectively.
  32 | //  - For functions that return a result in the flags, this macro can be empty,
  33 | //    because those are the correct flags to return anyway.
  34 | //  - Functions that return a boolean in r0 should set it up by checking the
  35 | //    flags.
  36 | //
  37 | // LOCAL_LABEL(NaN): a label defined within the compare function, after the
  38 | // #include of this header. Called when at least one input is a NaN, and sets
  39 | // up the appropriate return value for that case.
  40 | 
```
- **Line 31 / 第 31 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 32 / 第 32 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 33 / 第 33 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 34 / 第 34 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 35 / 第 35 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 36 / 第 36 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 37 / 第 37 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 38 / 第 38 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 39 / 第 39 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 40 / 第 40 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。

### Lines 41-50 / 第 41-50 行
```cpp
  41 | // --------------------------------------------------
  42 | // The actual entry point of the compare function.
  43 | //
  44 | // The basic plan is to start by ORing together the two inputs. This tells us
  45 | // two things:
  46 | //  - the top bit of the output tells us whether both inputs are positive, or
  47 | //    whether at least one is negative
  48 | //  - if the 8 exponent bits of the output are not all 1, then there are
  49 | //    definitely no NaNs, so a fast path can handle most non-NaN cases.
  50 | 
```
- **Line 41 / 第 41 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 42 / 第 42 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 43 / 第 43 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 44 / 第 44 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 45 / 第 45 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 46 / 第 46 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 47 / 第 47 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 48 / 第 48 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 49 / 第 49 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 50 / 第 50 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。

### Lines 51-60 / 第 51-60 行
```cpp
  51 | // clang-format off
  52 | 
  53 |   // First diverge control for the negative-numbers case.
  54 |   orrs    r12, op0, op1
  55 |   bmi     LOCAL_LABEL(negative)         // high bit set => at least one negative input
  56 | 
  57 |   // Here, both inputs are positive. Try adding 1<<23 to their bitwise OR in
  58 |   // r12. This will carry all the way into the top bit, setting the N flag, if
  59 |   // all 8 exponent bits were set.
  60 |   cmn     r12, #1 << 23
```
- **Line 51 / 第 51 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 52 / 第 52 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 53 / 第 53 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 54 / 第 54 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 55 / 第 55 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 56 / 第 56 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 57 / 第 57 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 58 / 第 58 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 59 / 第 59 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 60 / 第 60 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 61-70 / 第 61-70 行
```cpp
  61 |   bmi     LOCAL_LABEL(NaNInf_check_positive) // need to look harder for NaNs
  62 | 
  63 |   // The fastest fast path: both inputs positive and we could easily tell there
  64 |   // were no NaNs. So we just compare op0 and op1 as unsigned integers.
  65 |   cmp     op0, op1
  66 |   SetReturnRegister
  67 |   bx      lr
  68 | 
  69 | LOCAL_LABEL(NaNInf_check_positive):
  70 |   // Second tier for positive numbers. We come here if both inputs are
```
- **Line 61 / 第 61 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 62 / 第 62 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 63 / 第 63 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 64 / 第 64 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 65 / 第 65 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 66 / 第 66 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 67 / 第 67 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 68 / 第 68 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 69 / 第 69 行**: EN: Invokes a macro that expands portability, ABI, or registration boilerplate. CN: 调用一个宏，以展开可移植性、ABI 或注册相关样板代码。
- **Line 70 / 第 70 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。

### Lines 71-80 / 第 71-80 行
```cpp
  71 |   // positive, but our fast initial check didn't manage to rule out a NaN. But
  72 |   // it's not guaranteed that there _is_ a NaN, for two reasons:
  73 |   //
  74 |   //  1. An input with exponent 0xFF might be an infinity instead. Those behave
  75 |   //    normally under comparison.
  76 |   //
  77 |   //  2. There might not even _be_ an input with exponent 0xFF. All we know so
  78 |   //     far is that the two inputs ORed together had all the exponent bits
  79 |   //     set. So each of those bits is set in _at least one_ of the inputs, but
  80 |   //     not necessarily all in the _same_ input.
```
- **Line 71 / 第 71 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 72 / 第 72 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 73 / 第 73 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 74 / 第 74 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 75 / 第 75 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 76 / 第 76 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 77 / 第 77 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 78 / 第 78 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 79 / 第 79 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 80 / 第 80 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。

### Lines 81-90 / 第 81-90 行
```cpp
  81 |   //
  82 |   // Test each exponent individually for 0xFF, using the same CMN idiom as
  83 |   // above. If neither one carries into the sign bit then we have no NaNs _or_
  84 |   // infinities and can compare the registers and return again.
  85 |   cmn     op0, #1 << 23
  86 |   cmnpl   op1, #1 << 23
  87 |   bmi     LOCAL_LABEL(NaN_check_positive)
  88 | 
  89 |   // Second-tier return path, now we've ruled out anything difficult.
  90 |   cmp     op0, op1
```
- **Line 81 / 第 81 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 82 / 第 82 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 83 / 第 83 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 84 / 第 84 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 85 / 第 85 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 86 / 第 86 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 87 / 第 87 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 88 / 第 88 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 89 / 第 89 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 90 / 第 90 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 91-100 / 第 91-100 行
```cpp
  91 |   SetReturnRegister
  92 |   bx      lr
  93 | 
  94 | LOCAL_LABEL(NaN_check_positive):
  95 |   // Third tier for positive numbers. Here we know that at least one of the
  96 |   // inputs has exponent 0xFF. But they might still be infinities rather than
  97 |   // NaNs. So now we must check whether there's an actual NaN, by shifting each
  98 |   // input left to get rid of the sign bit, and seeing if the result is
  99 |   // _greater_ than 0xFF000000 (but not equal).
 100 |   //
```
- **Line 91 / 第 91 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 92 / 第 92 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 93 / 第 93 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 94 / 第 94 行**: EN: Invokes a macro that expands portability, ABI, or registration boilerplate. CN: 调用一个宏，以展开可移植性、ABI 或注册相关样板代码。
- **Line 95 / 第 95 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 96 / 第 96 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 97 / 第 97 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 98 / 第 98 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 99 / 第 99 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 100 / 第 100 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。

### Lines 101-110 / 第 101-110 行
```cpp
 101 |   // We could have skipped the second-tier check and done this more rigorous
 102 |   // test immediately. But that would cost an extra instruction in the case
 103 |   // where there are no infinities or NaNs, and we assume that that is so much
 104 |   // more common that it's worth optimizing for.
 105 |   mov     r12, #0xFF << 24
 106 |   cmp     r12, op0, LSL #1   // if LO, then r12 < (op0 << 1), so op0 is a NaN
 107 |   cmphs   r12, op1, LSL #1   // if not LO, then do the same check for op1
 108 |   blo     LOCAL_LABEL(NaN)           // now, if LO, there's definitely a NaN
 109 | 
 110 |   // Now we've finally ruled out NaNs! And we still know both inputs are
```
- **Line 101 / 第 101 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 102 / 第 102 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 103 / 第 103 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 104 / 第 104 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 105 / 第 105 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 106 / 第 106 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 107 / 第 107 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 108 / 第 108 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 109 / 第 109 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 110 / 第 110 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。

### Lines 111-120 / 第 111-120 行
```cpp
 111 |   // positive. So the third-tier return path can just compare the numbers
 112 |   // again.
 113 |   cmp     op0, op1
 114 |   SetReturnRegister
 115 |   bx      lr
 116 | 
 117 | LOCAL_LABEL(negative):
 118 |   // We come here if at least one operand is negative. We haven't checked for
 119 |   // NaNs at all yet (the sign check came first), so repeat the first-tier
 120 |   // check strategy of seeing if all exponent bits are set in r12.
```
- **Line 111 / 第 111 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 112 / 第 112 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 113 / 第 113 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 114 / 第 114 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 115 / 第 115 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 116 / 第 116 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 117 / 第 117 行**: EN: Invokes a macro that expands portability, ABI, or registration boilerplate. CN: 调用一个宏，以展开可移植性、ABI 或注册相关样板代码。
- **Line 118 / 第 118 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 119 / 第 119 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 120 / 第 120 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。

### Lines 121-130 / 第 121-130 行
```cpp
 121 |   //
 122 |   // On this path, the sign bit in r12 is set, so if adding 1 to the low
 123 |   // exponent bit carries all the way through into the sign bit, it will
 124 |   // _clear_ the sign bit rather than setting it. So we expect MI to be the
 125 |   // "definitely no NaNs" result, where it was PL on the positive branch.
 126 |   cmn     r12, #1 << 23
 127 |   bpl     LOCAL_LABEL(NaNInf_check_negative)
 128 | 
 129 |   // Now we have no NaNs, but at least one negative number. This gives us two
 130 |   // complications:
```
- **Line 121 / 第 121 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 122 / 第 122 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 123 / 第 123 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 124 / 第 124 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 125 / 第 125 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 126 / 第 126 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 127 / 第 127 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 128 / 第 128 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 129 / 第 129 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 130 / 第 130 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。

### Lines 131-140 / 第 131-140 行
```cpp
 131 |   //
 132 |   //  1. Floating-point numbers are sign/magnitude, not two's complement, so we
 133 |   //     have to consider separately the cases of "both negative" and "one of
 134 |   //     each sign".
 135 |   //
 136 |   //  2. -0 and +0 are required to compare equal.
 137 |   //
 138 |   // But problem #1 is not as hard as it sounds! If both operands are negative,
 139 |   // then we can get the result we want by comparing them as unsigned integers
 140 |   // the opposite way round, because the input with the smaller value (as an
```
- **Line 131 / 第 131 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 132 / 第 132 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 133 / 第 133 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 134 / 第 134 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 135 / 第 135 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 136 / 第 136 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 137 / 第 137 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 138 / 第 138 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 139 / 第 139 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 140 / 第 140 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。

### Lines 141-150 / 第 141-150 行
```cpp
 141 |   // integer) is the larger number in an FP ordering sense. And if one operand
 142 |   // is negative and the other is positive, the _same_ reversed comparison
 143 |   // works, because the positive number (with zero sign bit) will always
 144 |   // compare less than the negative one in an unsigned-integers sense.
 145 |   //
 146 |   // So we only have to worry about problem #2, signed zeroes. This only
 147 |   // affects the answer if _both_ operands are zero. And we can check that
 148 |   // easily, because it happens if and only if r12 = 0x80000000. (We know r12
 149 |   // has its sign bit set; if it has no other bits set, that's because both
 150 |   // inputs were either 0x80000000 or 0x00000000.)
```
- **Line 141 / 第 141 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 142 / 第 142 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 143 / 第 143 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 144 / 第 144 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 145 / 第 145 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 146 / 第 146 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 147 / 第 147 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 148 / 第 148 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 149 / 第 149 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 150 / 第 150 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。

### Lines 151-160 / 第 151-160 行
```cpp
 151 |   cmp     r12, #0x80000000        // EQ if both inputs are zero
 152 |   cmpne   op1, op0                // otherwise, compare them backwards
 153 |   SetReturnRegister
 154 |   bx      lr
 155 | 
 156 | LOCAL_LABEL(NaNInf_check_negative):
 157 |   // Second tier for negative numbers: we know the OR of the exponents is 0xFF,
 158 |   // but again, we might not have either _actual_ exponent 0xFF, and also, an
 159 |   // exponent 0xFF might be an infinity instead of a NaN.
 160 |   //
```
- **Line 151 / 第 151 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 152 / 第 152 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 153 / 第 153 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 154 / 第 154 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 155 / 第 155 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 156 / 第 156 行**: EN: Invokes a macro that expands portability, ABI, or registration boilerplate. CN: 调用一个宏，以展开可移植性、ABI 或注册相关样板代码。
- **Line 157 / 第 157 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 158 / 第 158 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 159 / 第 159 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 160 / 第 160 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。

### Lines 161-170 / 第 161-170 行
```cpp
 161 |   // On this path we've already branched twice (once for negative numbers and
 162 |   // once for the first-tier NaN check), so we'll just go straight to the
 163 |   // precise check for NaNs.
 164 |   mov     r12, #0xFF << 24
 165 |   cmp     r12, op0, LSL #1   // if LO, then r12 < (op0 << 1), so op0 is a NaN
 166 |   cmphs   r12, op1, LSL #1   // if not LO, then do the same check for op1
 167 |   blo     LOCAL_LABEL(NaN)
 168 | 
 169 |   // Now we've ruled out NaNs, so we can just compare the two input registers
 170 |   // and return. On this path we _don't_ need to check for the special case of
```
- **Line 161 / 第 161 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 162 / 第 162 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 163 / 第 163 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 164 / 第 164 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 165 / 第 165 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 166 / 第 166 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 167 / 第 167 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 168 / 第 168 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 169 / 第 169 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 170 / 第 170 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。

### Lines 171-176 / 第 171-176 行
```cpp
 171 |   // comparing two zeroes, because we only came here if the bitwise OR of the
 172 |   // exponent fields was 0xFF, which means the exponents can't both have been
 173 |   // zero! So we can _just_ do the reversed CMP and finish.
 174 |   cmp     op1, op0
 175 |   SetReturnRegister
 176 |   bx      lr
```
- **Line 171 / 第 171 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 172 / 第 172 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 173 / 第 173 行**: EN: Comment describing intent, behavior, or metadata. CN: 注释，用于说明意图、行为或元数据。
- **Line 174 / 第 174 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 175 / 第 175 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 176 / 第 176 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

## Key Concepts / 关键概念

- **EN**: compiler builtin lowering
  - **CN**: 编译器 builtin 降级实现
- **EN**: portable low-level arithmetic
  - **CN**: 可移植的底层算术实现

## Dependencies / 依赖关系

- No direct `#include` dependencies appear in this file. / 该文件中没有直接出现 `#include` 依赖。
