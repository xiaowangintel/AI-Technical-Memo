# integer.h — Code Analysis / 代码分析

## Source / 来源

- File: `flang/include/flang/Evaluate/integer.h`
- Repository: `llvm-project` / `llvm-project`
- Purpose (EN): Emulates binary integers of an arbitrary (but fixed) bit size for use when the host C++ environment does not support that size or when the full suite of Fortran's integer intrinsic scalar functions are needed. The data model is typeless, so signed* and unsigned operations.
- Purpose (CN): 声明与 integer 相关的接口、数据结构或工具。

## Line-by-Line Analysis / 逐行分析

### Line 1

~~~~cpp
//===-- include/flang/Evaluate/integer.h ------------------------*- C++ -*-===//
~~~~
- EN: Provides a file banner or visual separator.
- CN: 提供文件横幅或视觉分隔线。

### Line 2

~~~~cpp
//
~~~~
- EN: Provides a file banner or visual separator.
- CN: 提供文件横幅或视觉分隔线。

### Line 3

~~~~cpp
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
~~~~
- EN: Records licensing and redistribution terms for this file.
- CN: 记录该文件的许可证与再分发条款。

### Line 4

~~~~cpp
// See https://llvm.org/LICENSE.txt for license information.
~~~~
- EN: Records licensing and redistribution terms for this file.
- CN: 记录该文件的许可证与再分发条款。

### Line 5

~~~~cpp
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
~~~~
- EN: Records licensing and redistribution terms for this file.
- CN: 记录该文件的许可证与再分发条款。

### Line 6

~~~~cpp
//
~~~~
- EN: Provides a file banner or visual separator.
- CN: 提供文件横幅或视觉分隔线。

### Line 7

~~~~cpp
//===----------------------------------------------------------------------===//
~~~~
- EN: Provides a file banner or visual separator.
- CN: 提供文件横幅或视觉分隔线。

### Line 8

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 9

~~~~cpp
#ifndef FORTRAN_EVALUATE_INTEGER_H_
~~~~
- EN: Controls conditional compilation with `#ifndef`.
- CN: 使用 `#ifndef` 控制条件编译。

### Line 10

~~~~cpp
#define FORTRAN_EVALUATE_INTEGER_H_
~~~~
- EN: Defines the preprocessor macro `FORTRAN_EVALUATE_INTEGER_H_`.
- CN: 定义预处理宏 `FORTRAN_EVALUATE_INTEGER_H_`。

### Line 11

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 12

~~~~cpp
// Emulates binary integers of an arbitrary (but fixed) bit size for use
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 13

~~~~cpp
// when the host C++ environment does not support that size or when the
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 14

~~~~cpp
// full suite of Fortran's integer intrinsic scalar functions are needed.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 15

~~~~cpp
// The data model is typeless, so signed* and unsigned operations
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 16

~~~~cpp
// are distinguished from each other with distinct member function interfaces.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 17

~~~~cpp
// (*"Signed" here means two's-complement, just to be clear.  Ones'-complement
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 18

~~~~cpp
// and signed-magnitude encodings appear to be extinct in 2018.)
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 19

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 20

~~~~cpp
#include "flang/Common/bit-population-count.h"
~~~~
- EN: Includes the internal header `flang/Common/bit-population-count.h` so this file can use its declarations.
- CN: 引入内部头文件 `flang/Common/bit-population-count.h`，以便使用其中的声明。

### Line 21

~~~~cpp
#include "flang/Common/leading-zero-bit-count.h"
~~~~
- EN: Includes the internal header `flang/Common/leading-zero-bit-count.h` so this file can use its declarations.
- CN: 引入内部头文件 `flang/Common/leading-zero-bit-count.h`，以便使用其中的声明。

### Line 22

~~~~cpp
#include "flang/Evaluate/common.h"
~~~~
- EN: Includes the internal header `flang/Evaluate/common.h` so this file can use its declarations.
- CN: 引入内部头文件 `flang/Evaluate/common.h`，以便使用其中的声明。

### Line 23

~~~~cpp
#include <cinttypes>
~~~~
- EN: Includes the external or standard header `<cinttypes>` for supporting facilities.
- CN: 引入外部或标准头文件 `<cinttypes>` 以获得所需支持功能。

### Line 24

~~~~cpp
#include <climits>
~~~~
- EN: Includes the external or standard header `<climits>` for supporting facilities.
- CN: 引入外部或标准头文件 `<climits>` 以获得所需支持功能。

### Line 25

~~~~cpp
#include <cstddef>
~~~~
- EN: Includes the external or standard header `<cstddef>` for supporting facilities.
- CN: 引入外部或标准头文件 `<cstddef>` 以获得所需支持功能。

### Line 26

~~~~cpp
#include <cstdint>
~~~~
- EN: Includes the external or standard header `<cstdint>` for supporting facilities.
- CN: 引入外部或标准头文件 `<cstdint>` 以获得所需支持功能。

### Line 27

~~~~cpp
#include <string>
~~~~
- EN: Includes the external or standard header `<string>` for supporting facilities.
- CN: 引入外部或标准头文件 `<string>` 以获得所需支持功能。

### Line 28

~~~~cpp
#include <type_traits>
~~~~
- EN: Includes the external or standard header `<type_traits>` for supporting facilities.
- CN: 引入外部或标准头文件 `<type_traits>` 以获得所需支持功能。

### Line 29

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 30

~~~~cpp
// Some environments, viz. glibc 2.17 and *BSD, allow the macro HUGE
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 31

~~~~cpp
// to leak out of <math.h>.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 32

~~~~cpp
#undef HUGE
~~~~
- EN: Undefines the preprocessor macro `HUGE` to avoid leaking it further.
- CN: 取消定义预处理宏 `HUGE`，避免其继续影响后续代码。

### Line 33

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 34

~~~~cpp
namespace Fortran::evaluate::value {
~~~~
- EN: Opens namespace scope `Fortran::evaluate::value` to group related symbols.
- CN: 打开命名空间作用域 `Fortran::evaluate::value`，用于组织相关符号。

### Line 35

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 36

~~~~cpp
// Computes decimal range in the sense of SELECTED_INT_KIND
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 37

~~~~cpp
static constexpr int DecimalRange(int bits) {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 38

~~~~cpp
  // This magic value is LOG10(2.)*1E12.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 39

~~~~cpp
  return static_cast<int>((bits * 301029995664) / 1000000000000);
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 40

~~~~cpp
}
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 41

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 42

~~~~cpp
// Implements an integer as an assembly of smaller host integer parts
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 43

~~~~cpp
// that constitute the digits of a large-radix fixed-point number.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 44

~~~~cpp
// For best performance, the type of these parts should be half of the
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 45

~~~~cpp
// size of the largest efficient integer supported by the host processor.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 46

~~~~cpp
// These parts are stored in either little- or big-endian order, which can
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 47

~~~~cpp
// match that of the host's endianness or not; but if the ordering matches
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 48

~~~~cpp
// that of the host, raw host data can be overlaid with a properly configured
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 49

~~~~cpp
// instance of this class and used in situ.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 50

~~~~cpp
// To facilitate exhaustive testing of what would otherwise be more rare
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 51

~~~~cpp
// edge cases, this class template may be configured to use other part
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 52

~~~~cpp
// types &/or partial fields in the parts.  The radix (i.e., the number
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 53

~~~~cpp
// of possible values in a part), however, must be a power of two; this
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 54

~~~~cpp
// template class is not generalized to enable, say, decimal arithmetic.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 55

~~~~cpp
// Member functions that correspond to Fortran intrinsic functions are
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 56

~~~~cpp
// named accordingly in ALL CAPS so that they can be referenced easily in
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 57

~~~~cpp
// the language standard.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 58

~~~~cpp
template <int BITS, bool IS_LITTLE_ENDIAN = isHostLittleEndian,
~~~~
- EN: Starts a template parameter list or template-related declaration.
- CN: 开始一个模板参数列表或模板相关声明。

### Line 59

~~~~cpp
    int PARTBITS = BITS <= 32 ? BITS
~~~~
- EN: Declares and initializes a local variable or data member.
- CN: 声明并初始化一个局部变量或数据成员。

### Line 60

~~~~cpp
        : BITS % 32 == 0      ? 32
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 61

~~~~cpp
        : BITS % 16 == 0      ? 16
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 62

~~~~cpp
                              : 8,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 63

~~~~cpp
    typename PART = HostUnsignedInt<PARTBITS>,
~~~~
- EN: Declares and initializes a local variable or data member.
- CN: 声明并初始化一个局部变量或数据成员。

### Line 64

~~~~cpp
    typename BIGPART = HostUnsignedInt<PARTBITS * 2>, int ALIGNMENT = BITS>
~~~~
- EN: Declares and initializes a local variable or data member.
- CN: 声明并初始化一个局部变量或数据成员。

### Line 65

~~~~cpp
class Integer {
~~~~
- EN: Begins the definition of class `Integer`.
- CN: 开始定义 class `Integer`。

### Line 66

~~~~cpp
public:
~~~~
- EN: Switches subsequent class members to `public` access.
- CN: 将后续类成员切换为 `public` 访问级别。

### Line 67

~~~~cpp
  static constexpr int bits{BITS};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 68

~~~~cpp
  static constexpr int partBits{PARTBITS};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 69

~~~~cpp
  using Part = PART;
~~~~
- EN: Creates the alias `Part` for a longer type or entity name.
- CN: 为较长的类型或实体名称创建别名 `Part`。

### Line 70

~~~~cpp
  using BigPart = BIGPART;
~~~~
- EN: Creates the alias `BigPart` for a longer type or entity name.
- CN: 为较长的类型或实体名称创建别名 `BigPart`。

### Line 71

~~~~cpp
  static_assert(std::is_integral_v<Part>);
~~~~
- EN: Performs a compile-time sanity check.
- CN: 执行编译期健全性检查。

### Line 72

~~~~cpp
  static_assert(std::is_unsigned_v<Part>);
~~~~
- EN: Performs a compile-time sanity check.
- CN: 执行编译期健全性检查。

### Line 73

~~~~cpp
  static_assert(std::is_integral_v<BigPart>);
~~~~
- EN: Performs a compile-time sanity check.
- CN: 执行编译期健全性检查。

### Line 74

~~~~cpp
  static_assert(std::is_unsigned_v<BigPart>);
~~~~
- EN: Performs a compile-time sanity check.
- CN: 执行编译期健全性检查。

### Line 75

~~~~cpp
  static_assert(CHAR_BIT * sizeof(BigPart) >= 2 * partBits);
~~~~
- EN: Performs a compile-time sanity check.
- CN: 执行编译期健全性检查。

### Line 76

~~~~cpp
  static constexpr bool littleEndian{IS_LITTLE_ENDIAN};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 77

~~~~cpp
  static constexpr int alignment{ALIGNMENT};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 78

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 79

~~~~cpp
private:
~~~~
- EN: Switches subsequent class members to `private` access.
- CN: 将后续类成员切换为 `private` 访问级别。

### Line 80

~~~~cpp
  static constexpr int maxPartBits{CHAR_BIT * sizeof(Part)};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 81

~~~~cpp
  static_assert(partBits > 0 && partBits <= maxPartBits);
~~~~
- EN: Performs a compile-time sanity check.
- CN: 执行编译期健全性检查。

### Line 82

~~~~cpp
  static constexpr int extraPartBits{maxPartBits - partBits};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 83

~~~~cpp
  static constexpr int parts{(bits + partBits - 1) / partBits};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 84

~~~~cpp
  static_assert(parts >= 1);
~~~~
- EN: Performs a compile-time sanity check.
- CN: 执行编译期健全性检查。

### Line 85

~~~~cpp
  static constexpr int extraTopPartBits{
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 86

~~~~cpp
      extraPartBits + (parts * partBits) - bits};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 87

~~~~cpp
  static constexpr int topPartBits{maxPartBits - extraTopPartBits};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 88

~~~~cpp
  static_assert(topPartBits > 0 && topPartBits <= partBits);
~~~~
- EN: Performs a compile-time sanity check.
- CN: 执行编译期健全性检查。

### Line 89

~~~~cpp
  static_assert((parts - 1) * partBits + topPartBits == bits);
~~~~
- EN: Performs a compile-time sanity check.
- CN: 执行编译期健全性检查。

### Line 90

~~~~cpp
  static constexpr Part partMask{static_cast<Part>(~0) >> extraPartBits};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 91

~~~~cpp
  static constexpr Part topPartMask{static_cast<Part>(~0) >> extraTopPartBits};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 92

~~~~cpp
  static constexpr int partsWithAlignment{
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 93

~~~~cpp
      (ALIGNMENT + partBits - 1) / partBits};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 94

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 95

~~~~cpp
public:
~~~~
- EN: Switches subsequent class members to `public` access.
- CN: 将后续类成员切换为 `public` 访问级别。

### Line 96

~~~~cpp
  // Some types used for member function results
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 97

~~~~cpp
  struct ValueWithOverflow {
~~~~
- EN: Begins the definition of struct `ValueWithOverflow`.
- CN: 开始定义 struct `ValueWithOverflow`。

### Line 98

~~~~cpp
    Integer value;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 99

~~~~cpp
    bool overflow;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 100

~~~~cpp
  };
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 101

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 102

~~~~cpp
  struct ValueWithCarry {
~~~~
- EN: Begins the definition of struct `ValueWithCarry`.
- CN: 开始定义 struct `ValueWithCarry`。

### Line 103

~~~~cpp
    Integer value;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 104

~~~~cpp
    bool carry;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 105

~~~~cpp
  };
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 106

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 107

~~~~cpp
  struct Product {
~~~~
- EN: Begins the definition of struct `Product`.
- CN: 开始定义 struct `Product`。

### Line 108

~~~~cpp
    bool SignedMultiplicationOverflowed() const {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 109

~~~~cpp
      return lower.IsNegative() ? (upper.POPCNT() != bits) : !upper.IsZero();
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 110

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 111

~~~~cpp
    Integer upper, lower;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 112

~~~~cpp
  };
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 113

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 114

~~~~cpp
  struct QuotientWithRemainder {
~~~~
- EN: Begins the definition of struct `QuotientWithRemainder`.
- CN: 开始定义 struct `QuotientWithRemainder`。

### Line 115

~~~~cpp
    Integer quotient, remainder;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 116

~~~~cpp
    bool divisionByZero, overflow;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 117

~~~~cpp
  };
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 118

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 119

~~~~cpp
  struct PowerWithErrors {
~~~~
- EN: Begins the definition of struct `PowerWithErrors`.
- CN: 开始定义 struct `PowerWithErrors`。

### Line 120

~~~~cpp
    Integer power;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 121

~~~~cpp
    bool divisionByZero{false}, overflow{false}, zeroToZero{false};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 122

~~~~cpp
  };
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 123

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 124

~~~~cpp
  // Constructors and value-generating static functions
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 125

~~~~cpp
  constexpr Integer() { Clear(); } // default constructor: zero
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 126

~~~~cpp
  constexpr Integer(const Integer &) = default;
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 127

~~~~cpp
  constexpr Integer(Integer &&) = default;
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 128

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 129

~~~~cpp
  // C++'s integral types can all be converted to Integer
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 130

~~~~cpp
  // with silent truncation.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 131

~~~~cpp
  template <typename INT, typename = std::enable_if_t<std::is_integral_v<INT>>>
~~~~
- EN: Starts a template parameter list or template-related declaration.
- CN: 开始一个模板参数列表或模板相关声明。

### Line 132

~~~~cpp
  constexpr Integer(INT n) {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 133

~~~~cpp
    constexpr int nBits = CHAR_BIT * sizeof n;
~~~~
- EN: Declares and initializes a local variable or data member.
- CN: 声明并初始化一个局部变量或数据成员。

### Line 134

~~~~cpp
    if constexpr (nBits < partBits) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 135

~~~~cpp
      if constexpr (std::is_unsigned_v<INT>) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 136

~~~~cpp
        // Zero-extend an unsigned smaller value.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 137

~~~~cpp
        SetLEPart(0, n);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 138

~~~~cpp
        for (int j{1}; j < parts; ++j) {
~~~~
- EN: Starts a loop that iterates over a sequence or range.
- CN: 开始一个遍历序列或范围的循环。

### Line 139

~~~~cpp
          SetLEPart(j, 0);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 140

~~~~cpp
        }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 141

~~~~cpp
      } else {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 142

~~~~cpp
        // n has a signed type smaller than the usable
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 143

~~~~cpp
        // bits in a Part.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 144

~~~~cpp
        // Avoid conversions that change both size and sign.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 145

~~~~cpp
        using SignedPart = std::make_signed_t<Part>;
~~~~
- EN: Creates the alias `SignedPart` for a longer type or entity name.
- CN: 为较长的类型或实体名称创建别名 `SignedPart`。

### Line 146

~~~~cpp
        Part p = static_cast<SignedPart>(n);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 147

~~~~cpp
        SetLEPart(0, p);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 148

~~~~cpp
        if constexpr (parts > 1) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 149

~~~~cpp
          Part signExtension = static_cast<SignedPart>(-(n < 0));
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 150

~~~~cpp
          for (int j{1}; j < parts; ++j) {
~~~~
- EN: Starts a loop that iterates over a sequence or range.
- CN: 开始一个遍历序列或范围的循环。

### Line 151

~~~~cpp
            SetLEPart(j, signExtension);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 152

~~~~cpp
          }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 153

~~~~cpp
        }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 154

~~~~cpp
      }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 155

~~~~cpp
    } else {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 156

~~~~cpp
      // n has some integral type no smaller than the usable
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 157

~~~~cpp
      // bits in a Part.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 158

~~~~cpp
      // Ensure that all shifts are smaller than a whole word.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 159

~~~~cpp
      if constexpr (std::is_unsigned_v<INT>) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 160

~~~~cpp
        for (int j{0}; j < parts; ++j) {
~~~~
- EN: Starts a loop that iterates over a sequence or range.
- CN: 开始一个遍历序列或范围的循环。

### Line 161

~~~~cpp
          SetLEPart(j, static_cast<Part>(n));
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 162

~~~~cpp
          if constexpr (nBits > partBits) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 163

~~~~cpp
            n >>= partBits;
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 164

~~~~cpp
          } else {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 165

~~~~cpp
            n = 0;
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 166

~~~~cpp
          }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 167

~~~~cpp
        }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 168

~~~~cpp
      } else {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 169

~~~~cpp
        // Avoid left shifts of negative signed values (that's an undefined
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 170

~~~~cpp
        // behavior in C++).
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 171

~~~~cpp
        auto signExtension{std::make_unsigned_t<INT>(n < 0)};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 172

~~~~cpp
        signExtension = ~signExtension + 1;
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 173

~~~~cpp
        static_assert(nBits >= partBits);
~~~~
- EN: Performs a compile-time sanity check.
- CN: 执行编译期健全性检查。

### Line 174

~~~~cpp
        if constexpr (nBits > partBits) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 175

~~~~cpp
          signExtension <<= nBits - partBits;
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 176

~~~~cpp
          for (int j{0}; j < parts; ++j) {
~~~~
- EN: Starts a loop that iterates over a sequence or range.
- CN: 开始一个遍历序列或范围的循环。

### Line 177

~~~~cpp
            SetLEPart(j, static_cast<Part>(n));
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 178

~~~~cpp
            n >>= partBits;
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 179

~~~~cpp
            n |= signExtension;
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 180

~~~~cpp
          }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 181

~~~~cpp
        } else {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 182

~~~~cpp
          SetLEPart(0, static_cast<Part>(n));
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 183

~~~~cpp
          for (int j{1}; j < parts; ++j) {
~~~~
- EN: Starts a loop that iterates over a sequence or range.
- CN: 开始一个遍历序列或范围的循环。

### Line 184

~~~~cpp
            SetLEPart(j, static_cast<Part>(signExtension));
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 185

~~~~cpp
          }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 186

~~~~cpp
        }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 187

~~~~cpp
      }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 188

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 189

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 190

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 191

~~~~cpp
  constexpr Integer &operator=(const Integer &) = default;
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 192

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 193

~~~~cpp
  constexpr bool operator<(const Integer &that) const {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 194

~~~~cpp
    return CompareSigned(that) == Ordering::Less;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 195

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 196

~~~~cpp
  constexpr bool operator<=(const Integer &that) const {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 197

~~~~cpp
    return CompareSigned(that) != Ordering::Greater;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 198

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 199

~~~~cpp
  constexpr bool operator==(const Integer &that) const {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 200

~~~~cpp
    return CompareSigned(that) == Ordering::Equal;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 201

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 202

~~~~cpp
  constexpr bool operator!=(const Integer &that) const {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 203

~~~~cpp
    return !(*this == that);
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 204

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 205

~~~~cpp
  constexpr bool operator>=(const Integer &that) const {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 206

~~~~cpp
    return CompareSigned(that) != Ordering::Less;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 207

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 208

~~~~cpp
  constexpr bool operator>(const Integer &that) const {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 209

~~~~cpp
    return CompareSigned(that) == Ordering::Greater;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 210

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 211

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 212

~~~~cpp
  // Left-justified mask (e.g., MASKL(1) has only its sign bit set)
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 213

~~~~cpp
  static constexpr Integer MASKL(int places) {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 214

~~~~cpp
    if (places <= 0) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 215

~~~~cpp
      return {};
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 216

~~~~cpp
    } else if (places >= bits) {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 217

~~~~cpp
      return MASKR(bits);
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 218

~~~~cpp
    } else {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 219

~~~~cpp
      return MASKR(bits - places).NOT();
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 220

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 221

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 222

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 223

~~~~cpp
  // Right-justified mask (e.g., MASKR(1) == 1, MASKR(2) == 3, &c.)
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 224

~~~~cpp
  static constexpr Integer MASKR(int places) {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 225

~~~~cpp
    Integer result{nullptr};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 226

~~~~cpp
    int j{0};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 227

~~~~cpp
    for (; j + 1 < parts && places >= partBits; ++j, places -= partBits) {
~~~~
- EN: Starts a loop that iterates over a sequence or range.
- CN: 开始一个遍历序列或范围的循环。

### Line 228

~~~~cpp
      result.LEPart(j) = partMask;
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 229

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 230

~~~~cpp
    if (places > 0) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 231

~~~~cpp
      if (j + 1 < parts) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 232

~~~~cpp
        result.LEPart(j++) = partMask >> (partBits - places);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 233

~~~~cpp
      } else if (j + 1 == parts) {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 234

~~~~cpp
        if (places >= topPartBits) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 235

~~~~cpp
          result.LEPart(j++) = topPartMask;
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 236

~~~~cpp
        } else {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 237

~~~~cpp
          result.LEPart(j++) = topPartMask >> (topPartBits - places);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 238

~~~~cpp
        }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 239

~~~~cpp
      }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 240

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 241

~~~~cpp
    for (; j < parts; ++j) {
~~~~
- EN: Starts a loop that iterates over a sequence or range.
- CN: 开始一个遍历序列或范围的循环。

### Line 242

~~~~cpp
      result.LEPart(j) = 0;
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 243

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 244

~~~~cpp
    return result;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 245

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 246

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 247

~~~~cpp
  static constexpr ValueWithOverflow Read(
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 248

~~~~cpp
      const char *&pp, std::uint64_t base = 10, bool isSigned = false) {
~~~~
- EN: Declares and initializes a local variable or data member.
- CN: 声明并初始化一个局部变量或数据成员。

### Line 249

~~~~cpp
    Integer result;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 250

~~~~cpp
    bool overflow{false};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 251

~~~~cpp
    const char *p{pp};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 252

~~~~cpp
    while (*p == ' ' || *p == '\t') {
~~~~
- EN: Starts a while-loop controlled by a runtime condition.
- CN: 开始一个由运行时条件控制的 while 循环。

### Line 253

~~~~cpp
      ++p;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 254

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 255

~~~~cpp
    bool negate{*p == '-'};
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 256

~~~~cpp
    if (negate || *p == '+') {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 257

~~~~cpp
      while (*++p == ' ' || *p == '\t') {
~~~~
- EN: Starts a while-loop controlled by a runtime condition.
- CN: 开始一个由运行时条件控制的 while 循环。

### Line 258

~~~~cpp
      }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 259

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 260

~~~~cpp
    Integer radix{base};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 261

~~~~cpp
    // This code makes assumptions about local contiguity in regions of the
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 262

~~~~cpp
    // character set and only works up to base 36.  These assumptions hold
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 263

~~~~cpp
    // for all current combinations of surviving character sets (ASCII, UTF-8,
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 264

~~~~cpp
    // EBCDIC) and the bases used in Fortran source and formatted I/O
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 265

~~~~cpp
    // (viz., 2, 8, 10, & 16).  But: management thought that a disclaimer
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 266

~~~~cpp
    // might be needed here to warn future users of this code about these
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 267

~~~~cpp
    // assumptions, so here you go, future programmer in some postapocalyptic
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 268

~~~~cpp
    // hellscape, and best of luck with the inexorable killer robots.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 269

~~~~cpp
    for (; std::uint64_t digit = *p; ++p) {
~~~~
- EN: Starts a loop that iterates over a sequence or range.
- CN: 开始一个遍历序列或范围的循环。

### Line 270

~~~~cpp
      if (digit >= '0' && digit <= '9' && digit < '0' + base) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 271

~~~~cpp
        digit -= '0';
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 272

~~~~cpp
      } else if (base > 10 && digit >= 'A' && digit < 'A' + base - 10) {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 273

~~~~cpp
        digit -= 'A' - 10;
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 274

~~~~cpp
      } else if (base > 10 && digit >= 'a' && digit < 'a' + base - 10) {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 275

~~~~cpp
        digit -= 'a' - 10;
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 276

~~~~cpp
      } else {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 277

~~~~cpp
        break;
~~~~
- EN: Terminates the nearest loop or switch branch.
- CN: 终止最近一层循环或 switch 分支。

### Line 278

~~~~cpp
      }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 279

~~~~cpp
      Product shifted{result.MultiplyUnsigned(radix)};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 280

~~~~cpp
      overflow |= !shifted.upper.IsZero();
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 281

~~~~cpp
      ValueWithCarry next{shifted.lower.AddUnsigned(Integer{digit})};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 282

~~~~cpp
      overflow |= next.carry;
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 283

~~~~cpp
      result = next.value;
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 284

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 285

~~~~cpp
    pp = p;
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 286

~~~~cpp
    if (negate) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 287

~~~~cpp
      result = result.Negate().value;
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 288

~~~~cpp
      overflow |= isSigned && !result.IsNegative() && !result.IsZero();
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 289

~~~~cpp
    } else {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 290

~~~~cpp
      overflow |= isSigned && result.IsNegative();
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 291

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 292

~~~~cpp
    return {result, overflow};
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 293

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 294

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 295

~~~~cpp
  template <typename FROM>
~~~~
- EN: Starts a template parameter list or template-related declaration.
- CN: 开始一个模板参数列表或模板相关声明。

### Line 296

~~~~cpp
  static constexpr ValueWithOverflow ConvertUnsigned(const FROM &that) {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 297

~~~~cpp
    std::uint64_t field{that.ToUInt64()};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 298

~~~~cpp
    ValueWithOverflow result{field, false};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 299

~~~~cpp
    if constexpr (bits < 64) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 300

~~~~cpp
      result.overflow = (field >> bits) != 0;
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 301

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 302

~~~~cpp
    for (int j{64}; j < that.bits && !result.overflow; j += 64) {
~~~~
- EN: Starts a loop that iterates over a sequence or range.
- CN: 开始一个遍历序列或范围的循环。

### Line 303

~~~~cpp
      field = that.SHIFTR(j).ToUInt64();
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 304

~~~~cpp
      if (bits <= j) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 305

~~~~cpp
        result.overflow = field != 0;
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 306

~~~~cpp
      } else {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 307

~~~~cpp
        result.value = result.value.IOR(Integer{field}.SHIFTL(j));
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 308

~~~~cpp
        if (bits < j + 64) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 309

~~~~cpp
          result.overflow = (field >> (bits - j)) != 0;
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 310

~~~~cpp
        }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 311

~~~~cpp
      }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 312

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 313

~~~~cpp
    return result;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 314

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 315

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 316

~~~~cpp
  template <typename FROM>
~~~~
- EN: Starts a template parameter list or template-related declaration.
- CN: 开始一个模板参数列表或模板相关声明。

### Line 317

~~~~cpp
  static constexpr ValueWithOverflow ConvertSigned(const FROM &that) {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 318

~~~~cpp
    ValueWithOverflow result{ConvertUnsigned(that)};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 319

~~~~cpp
    if constexpr (bits > FROM::bits) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 320

~~~~cpp
      if (that.IsNegative()) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 321

~~~~cpp
        result.value = result.value.IOR(MASKL(bits - FROM::bits));
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 322

~~~~cpp
      }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 323

~~~~cpp
      result.overflow = false;
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 324

~~~~cpp
    } else if constexpr (bits < FROM::bits) {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 325

~~~~cpp
      auto back{FROM::template ConvertSigned<Integer>(result.value)};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 326

~~~~cpp
      result.overflow = back.value.CompareUnsigned(that) != Ordering::Equal;
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 327

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 328

~~~~cpp
    return result;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 329

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 330

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 331

~~~~cpp
  std::string UnsignedDecimal() const {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 332

~~~~cpp
    if constexpr (bits < 4) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 333

~~~~cpp
      char digit = '0' + ToUInt64();
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 334

~~~~cpp
      return {digit};
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 335

~~~~cpp
    } else if (IsZero()) {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 336

~~~~cpp
      return {'0'};
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 337

~~~~cpp
    } else {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 338

~~~~cpp
      QuotientWithRemainder qr{DivideUnsigned(10)};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 339

~~~~cpp
      char digit = '0' + qr.remainder.ToUInt64();
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 340

~~~~cpp
      if (qr.quotient.IsZero()) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 341

~~~~cpp
        return {digit};
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 342

~~~~cpp
      } else {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 343

~~~~cpp
        return qr.quotient.UnsignedDecimal() + digit;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 344

~~~~cpp
      }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 345

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 346

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 347

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 348

~~~~cpp
  std::string SignedDecimal() const {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 349

~~~~cpp
    if (IsNegative()) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 350

~~~~cpp
      return std::string{'-'} + Negate().value.UnsignedDecimal();
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 351

~~~~cpp
    } else {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 352

~~~~cpp
      return UnsignedDecimal();
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 353

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 354

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 355

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 356

~~~~cpp
  // Omits a leading "0x".
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 357

~~~~cpp
  std::string Hexadecimal() const {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 358

~~~~cpp
    std::string result;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 359

~~~~cpp
    int digits{(bits + 3) >> 2};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 360

~~~~cpp
    for (int j{0}; j < digits; ++j) {
~~~~
- EN: Starts a loop that iterates over a sequence or range.
- CN: 开始一个遍历序列或范围的循环。

### Line 361

~~~~cpp
      int pos{(digits - 1 - j) * 4};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 362

~~~~cpp
      char nybble = IBITS(pos, 4).ToUInt64();
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 363

~~~~cpp
      if (nybble != 0 || !result.empty() || j + 1 == digits) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 364

~~~~cpp
        char digit = '0' + nybble;
~~~~
- EN: Declares and initializes a local variable or data member.
- CN: 声明并初始化一个局部变量或数据成员。

### Line 365

~~~~cpp
        if (digit > '9') {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 366

~~~~cpp
          digit += 'a' - ('9' + 1);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 367

~~~~cpp
        }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 368

~~~~cpp
        result += digit;
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 369

~~~~cpp
      }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 370

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 371

~~~~cpp
    return result;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 372

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 373

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 374

~~~~cpp
  static constexpr int DIGITS{bits - 1}; // don't count the sign bit
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 375

~~~~cpp
  static constexpr Integer HUGE() { return MASKR(bits - 1); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 376

~~~~cpp
  static constexpr Integer Least() { return MASKL(1); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 377

~~~~cpp
  static constexpr int RANGE{DecimalRange(bits - 1)};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 378

~~~~cpp
  static constexpr int UnsignedRANGE{DecimalRange(bits)};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 379

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 380

~~~~cpp
  constexpr bool IsZero() const {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 381

~~~~cpp
    for (int j{0}; j < parts; ++j) {
~~~~
- EN: Starts a loop that iterates over a sequence or range.
- CN: 开始一个遍历序列或范围的循环。

### Line 382

~~~~cpp
      if (part_[j] != 0) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 383

~~~~cpp
        return false;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 384

~~~~cpp
      }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 385

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 386

~~~~cpp
    return true;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 387

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 388

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 389

~~~~cpp
  constexpr bool IsNegative() const {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 390

~~~~cpp
    return (LEPart(parts - 1) >> (topPartBits - 1)) & 1;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 391

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 392

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 393

~~~~cpp
  constexpr Ordering CompareToZeroSigned() const {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 394

~~~~cpp
    if (IsNegative()) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 395

~~~~cpp
      return Ordering::Less;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 396

~~~~cpp
    } else if (IsZero()) {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 397

~~~~cpp
      return Ordering::Equal;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 398

~~~~cpp
    } else {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 399

~~~~cpp
      return Ordering::Greater;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 400

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 401

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 402

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 403

~~~~cpp
  // Count the number of contiguous most-significant bit positions
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 404

~~~~cpp
  // that are clear.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 405

~~~~cpp
  constexpr int LEADZ() const {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 406

~~~~cpp
    if (LEPart(parts - 1) != 0) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 407

~~~~cpp
      int lzbc{common::LeadingZeroBitCount(LEPart(parts - 1))};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 408

~~~~cpp
      return lzbc - extraTopPartBits;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 409

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 410

~~~~cpp
    int upperZeroes{topPartBits};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 411

~~~~cpp
    for (int j{1}; j < parts; ++j) {
~~~~
- EN: Starts a loop that iterates over a sequence or range.
- CN: 开始一个遍历序列或范围的循环。

### Line 412

~~~~cpp
      if (Part p{LEPart(parts - 1 - j)}) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 413

~~~~cpp
        int lzbc{common::LeadingZeroBitCount(p)};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 414

~~~~cpp
        return upperZeroes + lzbc - extraPartBits;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 415

~~~~cpp
      }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 416

~~~~cpp
      upperZeroes += partBits;
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 417

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 418

~~~~cpp
    return bits;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 419

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 420

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 421

~~~~cpp
  // Count the number of bit positions that are set.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 422

~~~~cpp
  constexpr int POPCNT() const {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 423

~~~~cpp
    int count{0};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 424

~~~~cpp
    for (int j{0}; j < parts; ++j) {
~~~~
- EN: Starts a loop that iterates over a sequence or range.
- CN: 开始一个遍历序列或范围的循环。

### Line 425

~~~~cpp
      count += common::BitPopulationCount(part_[j]);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 426

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 427

~~~~cpp
    return count;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 428

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 429

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 430

~~~~cpp
  // True when POPCNT is odd.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 431

~~~~cpp
  constexpr bool POPPAR() const { return POPCNT() & 1; }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 432

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 433

~~~~cpp
  constexpr int TRAILZ() const {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 434

~~~~cpp
    auto minus1{AddUnsigned(MASKR(bits))}; // { x-1, carry = x > 0 }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 435

~~~~cpp
    if (!minus1.carry) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 436

~~~~cpp
      return bits; // was zero
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 437

~~~~cpp
    } else {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 438

~~~~cpp
      // x ^ (x-1) has all bits set at and below original least-order set bit.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 439

~~~~cpp
      return IEOR(minus1.value).POPCNT() - 1;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 440

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 441

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 442

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 443

~~~~cpp
  constexpr bool BTEST(int pos) const {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 444

~~~~cpp
    if (pos < 0 || pos >= bits) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 445

~~~~cpp
      return false;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 446

~~~~cpp
    } else {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 447

~~~~cpp
      return (LEPart(pos / partBits) >> (pos % partBits)) & 1;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 448

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 449

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 450

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 451

~~~~cpp
  constexpr Ordering CompareUnsigned(const Integer &y) const {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 452

~~~~cpp
    for (int j{parts}; j-- > 0;) {
~~~~
- EN: Starts a loop that iterates over a sequence or range.
- CN: 开始一个遍历序列或范围的循环。

### Line 453

~~~~cpp
      if (LEPart(j) > y.LEPart(j)) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 454

~~~~cpp
        return Ordering::Greater;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 455

~~~~cpp
      }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 456

~~~~cpp
      if (LEPart(j) < y.LEPart(j)) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 457

~~~~cpp
        return Ordering::Less;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 458

~~~~cpp
      }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 459

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 460

~~~~cpp
    return Ordering::Equal;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 461

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 462

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 463

~~~~cpp
  constexpr bool BGE(const Integer &y) const {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 464

~~~~cpp
    return CompareUnsigned(y) != Ordering::Less;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 465

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 466

~~~~cpp
  constexpr bool BGT(const Integer &y) const {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 467

~~~~cpp
    return CompareUnsigned(y) == Ordering::Greater;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 468

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 469

~~~~cpp
  constexpr bool BLE(const Integer &y) const { return !BGT(y); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 470

~~~~cpp
  constexpr bool BLT(const Integer &y) const { return !BGE(y); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 471

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 472

~~~~cpp
  constexpr Ordering CompareSigned(const Integer &y) const {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 473

~~~~cpp
    bool isNegative{IsNegative()};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 474

~~~~cpp
    if (isNegative != y.IsNegative()) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 475

~~~~cpp
      return isNegative ? Ordering::Less : Ordering::Greater;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 476

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 477

~~~~cpp
    return CompareUnsigned(y);
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 478

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 479

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 480

~~~~cpp
  template <typename UINT = std::uint64_t> constexpr UINT ToUInt() const {
~~~~
- EN: Starts a template parameter list or template-related declaration.
- CN: 开始一个模板参数列表或模板相关声明。

### Line 481

~~~~cpp
    UINT n{LEPart(0)};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 482

~~~~cpp
    std::size_t filled{partBits};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 483

~~~~cpp
    constexpr std::size_t maxBits{CHAR_BIT * sizeof n};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 484

~~~~cpp
    for (int j{1}; filled < maxBits && j < parts; ++j, filled += partBits) {
~~~~
- EN: Starts a loop that iterates over a sequence or range.
- CN: 开始一个遍历序列或范围的循环。

### Line 485

~~~~cpp
      n |= UINT{LEPart(j)} << filled;
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 486

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 487

~~~~cpp
    return n;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 488

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 489

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 490

~~~~cpp
  template <typename SINT = std::int64_t, typename UINT = std::uint64_t>
~~~~
- EN: Starts a template parameter list or template-related declaration.
- CN: 开始一个模板参数列表或模板相关声明。

### Line 491

~~~~cpp
  constexpr SINT ToSInt() const {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 492

~~~~cpp
    SINT n = ToUInt<UINT>();
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 493

~~~~cpp
    constexpr std::size_t maxBits{CHAR_BIT * sizeof n};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 494

~~~~cpp
    if constexpr (bits < maxBits) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 495

~~~~cpp
      // Avoid left shifts of negative signed values (that's an undefined
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 496

~~~~cpp
      // behavior in C++).
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 497

~~~~cpp
      auto u{std::make_unsigned_t<SINT>(ToUInt())};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 498

~~~~cpp
      u = (u >> (bits - 1)) << (bits - 1); // Get the sign bit only.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 499

~~~~cpp
      u = ~u + 1; // Negate top bits if not 0.
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 500

~~~~cpp
      n |= static_cast<SINT>(u);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 501

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 502

~~~~cpp
    return n;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 503

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 504

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 505

~~~~cpp
  constexpr std::uint64_t ToUInt64() const { return ToUInt<std::uint64_t>(); }
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 506

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 507

~~~~cpp
  constexpr std::int64_t ToInt64() const {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 508

~~~~cpp
    return ToSInt<std::int64_t, std::uint64_t>();
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 509

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 510

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 511

~~~~cpp
  // Ones'-complement (i.e., C's ~)
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 512

~~~~cpp
  constexpr Integer NOT() const {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 513

~~~~cpp
    Integer result{nullptr};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 514

~~~~cpp
    for (int j{0}; j < parts; ++j) {
~~~~
- EN: Starts a loop that iterates over a sequence or range.
- CN: 开始一个遍历序列或范围的循环。

### Line 515

~~~~cpp
      result.SetLEPart(j, ~LEPart(j));
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 516

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 517

~~~~cpp
    return result;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 518

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 519

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 520

~~~~cpp
  // Two's-complement negation (-x = ~x + 1).
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 521

~~~~cpp
  // An overflow flag accompanies the result, and will be true when the
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 522

~~~~cpp
  // operand is the most negative signed number (MASKL(1)).
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 523

~~~~cpp
  constexpr ValueWithOverflow Negate() const {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 524

~~~~cpp
    Integer result{nullptr};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 525

~~~~cpp
    Part carry{1};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 526

~~~~cpp
    for (int j{0}; j + 1 < parts; ++j) {
~~~~
- EN: Starts a loop that iterates over a sequence or range.
- CN: 开始一个遍历序列或范围的循环。

### Line 527

~~~~cpp
      Part newCarry{LEPart(j) == 0 && carry};
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 528

~~~~cpp
      result.SetLEPart(j, ~LEPart(j) + carry);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 529

~~~~cpp
      carry = newCarry;
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 530

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 531

~~~~cpp
    Part top{LEPart(parts - 1)};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 532

~~~~cpp
    result.SetLEPart(parts - 1, ~top + carry);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 533

~~~~cpp
    bool overflow{top != 0 && result.LEPart(parts - 1) == top};
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 534

~~~~cpp
    return {result, overflow};
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 535

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 536

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 537

~~~~cpp
  constexpr ValueWithOverflow ABS() const {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 538

~~~~cpp
    if (IsNegative()) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 539

~~~~cpp
      return Negate();
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 540

~~~~cpp
    } else {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 541

~~~~cpp
      return {*this, false};
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 542

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 543

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 544

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 545

~~~~cpp
  // Shifts the operand left when the count is positive, right when negative.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 546

~~~~cpp
  // Vacated bit positions are filled with zeroes.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 547

~~~~cpp
  constexpr Integer ISHFT(int count) const {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 548

~~~~cpp
    if (count < 0) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 549

~~~~cpp
      return SHIFTR(-count);
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 550

~~~~cpp
    } else {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 551

~~~~cpp
      return SHIFTL(count);
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 552

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 553

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 554

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 555

~~~~cpp
  // Left shift with zero fill.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 556

~~~~cpp
  constexpr Integer SHIFTL(int count) const {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 557

~~~~cpp
    if (count <= 0) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 558

~~~~cpp
      return *this;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 559

~~~~cpp
    } else {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 560

~~~~cpp
      Integer result{nullptr};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 561

~~~~cpp
      int shiftParts{count / partBits};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 562

~~~~cpp
      int bitShift{count - partBits * shiftParts};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 563

~~~~cpp
      int j{parts - 1};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 564

~~~~cpp
      if (bitShift == 0) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 565

~~~~cpp
        for (; j >= shiftParts; --j) {
~~~~
- EN: Starts a loop that iterates over a sequence or range.
- CN: 开始一个遍历序列或范围的循环。

### Line 566

~~~~cpp
          result.SetLEPart(j, LEPart(j - shiftParts));
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 567

~~~~cpp
        }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 568

~~~~cpp
        for (; j >= 0; --j) {
~~~~
- EN: Starts a loop that iterates over a sequence or range.
- CN: 开始一个遍历序列或范围的循环。

### Line 569

~~~~cpp
          result.LEPart(j) = 0;
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 570

~~~~cpp
        }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 571

~~~~cpp
      } else {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 572

~~~~cpp
        for (; j > shiftParts; --j) {
~~~~
- EN: Starts a loop that iterates over a sequence or range.
- CN: 开始一个遍历序列或范围的循环。

### Line 573

~~~~cpp
          result.SetLEPart(j,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 574

~~~~cpp
              ((LEPart(j - shiftParts) << bitShift) |
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 575

~~~~cpp
                  (LEPart(j - shiftParts - 1) >> (partBits - bitShift))));
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 576

~~~~cpp
        }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 577

~~~~cpp
        if (j == shiftParts) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 578

~~~~cpp
          result.SetLEPart(j, LEPart(0) << bitShift);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 579

~~~~cpp
          --j;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 580

~~~~cpp
        }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 581

~~~~cpp
        for (; j >= 0; --j) {
~~~~
- EN: Starts a loop that iterates over a sequence or range.
- CN: 开始一个遍历序列或范围的循环。

### Line 582

~~~~cpp
          result.LEPart(j) = 0;
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 583

~~~~cpp
        }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 584

~~~~cpp
      }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 585

~~~~cpp
      return result;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 586

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 587

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 588

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 589

~~~~cpp
  // Circular shift of a field of least-significant bits.  The least-order
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 590

~~~~cpp
  // "size" bits are shifted circularly in place by "count" positions;
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 591

~~~~cpp
  // the shift is leftward if count is nonnegative, rightward otherwise.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 592

~~~~cpp
  // Higher-order bits are unchanged.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 593

~~~~cpp
  constexpr Integer ISHFTC(int count, int size = bits) const {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 594

~~~~cpp
    if (count == 0 || size <= 0) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 595

~~~~cpp
      return *this;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 596

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 597

~~~~cpp
    if (size > bits) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 598

~~~~cpp
      size = bits;
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 599

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 600

~~~~cpp
    count %= size;
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 601

~~~~cpp
    if (count == 0) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 602

~~~~cpp
      return *this;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 603

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 604

~~~~cpp
    int middleBits{size - count}, leastBits{count};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 605

~~~~cpp
    if (count < 0) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 606

~~~~cpp
      middleBits = -count;
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 607

~~~~cpp
      leastBits = size + count;
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 608

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 609

~~~~cpp
    if (size == bits) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 610

~~~~cpp
      return SHIFTL(leastBits).IOR(SHIFTR(middleBits));
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 611

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 612

~~~~cpp
    Integer unchanged{IAND(MASKL(bits - size))};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 613

~~~~cpp
    Integer middle{IAND(MASKR(middleBits)).SHIFTL(leastBits)};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 614

~~~~cpp
    Integer least{SHIFTR(middleBits).IAND(MASKR(leastBits))};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 615

~~~~cpp
    return unchanged.IOR(middle).IOR(least);
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 616

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 617

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 618

~~~~cpp
  // Double shifts, aka shifts with specific fill.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 619

~~~~cpp
  constexpr Integer SHIFTLWithFill(const Integer &fill, int count) const {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 620

~~~~cpp
    if (count <= 0) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 621

~~~~cpp
      return *this;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 622

~~~~cpp
    } else if (count >= 2 * bits) {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 623

~~~~cpp
      return {};
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 624

~~~~cpp
    } else if (count > bits) {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 625

~~~~cpp
      return fill.SHIFTL(count - bits);
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 626

~~~~cpp
    } else if (count == bits) {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 627

~~~~cpp
      return fill;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 628

~~~~cpp
    } else {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 629

~~~~cpp
      return SHIFTL(count).IOR(fill.SHIFTR(bits - count));
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 630

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 631

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 632

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 633

~~~~cpp
  constexpr Integer SHIFTRWithFill(const Integer &fill, int count) const {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 634

~~~~cpp
    if (count <= 0) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 635

~~~~cpp
      return *this;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 636

~~~~cpp
    } else if (count >= 2 * bits) {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 637

~~~~cpp
      return {};
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 638

~~~~cpp
    } else if (count > bits) {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 639

~~~~cpp
      return fill.SHIFTR(count - bits);
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 640

~~~~cpp
    } else if (count == bits) {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 641

~~~~cpp
      return fill;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 642

~~~~cpp
    } else {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 643

~~~~cpp
      return SHIFTR(count).IOR(fill.SHIFTL(bits - count));
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 644

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 645

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 646

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 647

~~~~cpp
  constexpr Integer DSHIFTL(const Integer &fill, int count) const {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 648

~~~~cpp
    // DSHIFTL(I,J) shifts I:J left; the second argument is the right fill.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 649

~~~~cpp
    return SHIFTLWithFill(fill, count);
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 650

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 651

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 652

~~~~cpp
  constexpr Integer DSHIFTR(const Integer &value, int count) const {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 653

~~~~cpp
    // DSHIFTR(I,J) shifts I:J right; the *first* argument is the left fill.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 654

~~~~cpp
    return value.SHIFTRWithFill(*this, count);
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 655

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 656

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 657

~~~~cpp
  // Vacated upper bits are filled with zeroes.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 658

~~~~cpp
  constexpr Integer SHIFTR(int count) const {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 659

~~~~cpp
    if (count <= 0) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 660

~~~~cpp
      return *this;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 661

~~~~cpp
    } else {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 662

~~~~cpp
      Integer result{nullptr};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 663

~~~~cpp
      int shiftParts{count / partBits};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 664

~~~~cpp
      int bitShift{count - partBits * shiftParts};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 665

~~~~cpp
      int j{0};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 666

~~~~cpp
      if (bitShift == 0) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 667

~~~~cpp
        for (; j + shiftParts < parts; ++j) {
~~~~
- EN: Starts a loop that iterates over a sequence or range.
- CN: 开始一个遍历序列或范围的循环。

### Line 668

~~~~cpp
          result.LEPart(j) = LEPart(j + shiftParts);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 669

~~~~cpp
        }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 670

~~~~cpp
        for (; j < parts; ++j) {
~~~~
- EN: Starts a loop that iterates over a sequence or range.
- CN: 开始一个遍历序列或范围的循环。

### Line 671

~~~~cpp
          result.LEPart(j) = 0;
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 672

~~~~cpp
        }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 673

~~~~cpp
      } else {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 674

~~~~cpp
        for (; j + shiftParts + 1 < parts; ++j) {
~~~~
- EN: Starts a loop that iterates over a sequence or range.
- CN: 开始一个遍历序列或范围的循环。

### Line 675

~~~~cpp
          result.SetLEPart(j,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 676

~~~~cpp
              (LEPart(j + shiftParts) >> bitShift) |
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 677

~~~~cpp
                  (LEPart(j + shiftParts + 1) << (partBits - bitShift)));
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 678

~~~~cpp
        }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 679

~~~~cpp
        if (j + shiftParts + 1 == parts) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 680

~~~~cpp
          result.LEPart(j++) = LEPart(parts - 1) >> bitShift;
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 681

~~~~cpp
        }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 682

~~~~cpp
        for (; j < parts; ++j) {
~~~~
- EN: Starts a loop that iterates over a sequence or range.
- CN: 开始一个遍历序列或范围的循环。

### Line 683

~~~~cpp
          result.LEPart(j) = 0;
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 684

~~~~cpp
        }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 685

~~~~cpp
      }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 686

~~~~cpp
      return result;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 687

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 688

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 689

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 690

~~~~cpp
  // Be advised, an arithmetic (sign-filling) right shift is not
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 691

~~~~cpp
  // the same as a division by a power of two in all cases.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 692

~~~~cpp
  constexpr Integer SHIFTA(int count) const {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 693

~~~~cpp
    if (count <= 0) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 694

~~~~cpp
      return *this;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 695

~~~~cpp
    } else if (IsNegative()) {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 696

~~~~cpp
      return SHIFTR(count).IOR(MASKL(count));
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 697

~~~~cpp
    } else {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 698

~~~~cpp
      return SHIFTR(count);
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 699

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 700

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 701

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 702

~~~~cpp
  // Clears a single bit.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 703

~~~~cpp
  constexpr Integer IBCLR(int pos) const {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 704

~~~~cpp
    if (pos < 0 || pos >= bits) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 705

~~~~cpp
      return *this;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 706

~~~~cpp
    } else {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 707

~~~~cpp
      Integer result{*this};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 708

~~~~cpp
      result.LEPart(pos / partBits) &= ~(Part{1} << (pos % partBits));
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 709

~~~~cpp
      return result;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 710

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 711

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 712

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 713

~~~~cpp
  // Sets a single bit.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 714

~~~~cpp
  constexpr Integer IBSET(int pos) const {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 715

~~~~cpp
    if (pos < 0 || pos >= bits) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 716

~~~~cpp
      return *this;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 717

~~~~cpp
    } else {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 718

~~~~cpp
      Integer result{*this};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 719

~~~~cpp
      result.LEPart(pos / partBits) |= Part{1} << (pos % partBits);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 720

~~~~cpp
      return result;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 721

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 722

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 723

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 724

~~~~cpp
  // Extracts a field.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 725

~~~~cpp
  constexpr Integer IBITS(int pos, int size) const {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 726

~~~~cpp
    return SHIFTR(pos).IAND(MASKR(size));
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 727

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 728

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 729

~~~~cpp
  constexpr Integer IAND(const Integer &y) const {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 730

~~~~cpp
    Integer result{nullptr};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 731

~~~~cpp
    for (int j{0}; j < parts; ++j) {
~~~~
- EN: Starts a loop that iterates over a sequence or range.
- CN: 开始一个遍历序列或范围的循环。

### Line 732

~~~~cpp
      result.LEPart(j) = LEPart(j) & y.LEPart(j);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 733

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 734

~~~~cpp
    return result;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 735

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 736

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 737

~~~~cpp
  constexpr Integer IOR(const Integer &y) const {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 738

~~~~cpp
    Integer result{nullptr};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 739

~~~~cpp
    for (int j{0}; j < parts; ++j) {
~~~~
- EN: Starts a loop that iterates over a sequence or range.
- CN: 开始一个遍历序列或范围的循环。

### Line 740

~~~~cpp
      result.LEPart(j) = LEPart(j) | y.LEPart(j);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 741

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 742

~~~~cpp
    return result;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 743

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 744

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 745

~~~~cpp
  constexpr Integer IEOR(const Integer &y) const {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 746

~~~~cpp
    Integer result{nullptr};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 747

~~~~cpp
    for (int j{0}; j < parts; ++j) {
~~~~
- EN: Starts a loop that iterates over a sequence or range.
- CN: 开始一个遍历序列或范围的循环。

### Line 748

~~~~cpp
      result.LEPart(j) = LEPart(j) ^ y.LEPart(j);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 749

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 750

~~~~cpp
    return result;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 751

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 752

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 753

~~~~cpp
  constexpr Integer MERGE_BITS(const Integer &y, const Integer &mask) const {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 754

~~~~cpp
    return IAND(mask).IOR(y.IAND(mask.NOT()));
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 755

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 756

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 757

~~~~cpp
  constexpr Integer MAX(const Integer &y) const {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 758

~~~~cpp
    if (CompareSigned(y) == Ordering::Less) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 759

~~~~cpp
      return y;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 760

~~~~cpp
    } else {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 761

~~~~cpp
      return *this;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 762

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 763

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 764

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 765

~~~~cpp
  constexpr Integer MIN(const Integer &y) const {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 766

~~~~cpp
    if (CompareSigned(y) == Ordering::Less) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 767

~~~~cpp
      return *this;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 768

~~~~cpp
    } else {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 769

~~~~cpp
      return y;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 770

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 771

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 772

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 773

~~~~cpp
  // Unsigned addition with carry.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 774

~~~~cpp
  constexpr ValueWithCarry AddUnsigned(
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 775

~~~~cpp
      const Integer &y, bool carryIn = false) const {
~~~~
- EN: Declares and initializes a local variable or data member.
- CN: 声明并初始化一个局部变量或数据成员。

### Line 776

~~~~cpp
    Integer sum{nullptr};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 777

~~~~cpp
    BigPart carry{carryIn};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 778

~~~~cpp
    for (int j{0}; j + 1 < parts; ++j) {
~~~~
- EN: Starts a loop that iterates over a sequence or range.
- CN: 开始一个遍历序列或范围的循环。

### Line 779

~~~~cpp
      carry += LEPart(j);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 780

~~~~cpp
      carry += y.LEPart(j);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 781

~~~~cpp
      sum.SetLEPart(j, carry);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 782

~~~~cpp
      carry >>= partBits;
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 783

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 784

~~~~cpp
    carry += LEPart(parts - 1);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 785

~~~~cpp
    carry += y.LEPart(parts - 1);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 786

~~~~cpp
    sum.SetLEPart(parts - 1, carry);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 787

~~~~cpp
    return {sum, carry > topPartMask};
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 788

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 789

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 790

~~~~cpp
  constexpr ValueWithOverflow AddSigned(const Integer &y) const {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 791

~~~~cpp
    bool isNegative{IsNegative()};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 792

~~~~cpp
    bool sameSign{isNegative == y.IsNegative()};
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 793

~~~~cpp
    ValueWithCarry sum{AddUnsigned(y)};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 794

~~~~cpp
    bool overflow{sameSign && sum.value.IsNegative() != isNegative};
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 795

~~~~cpp
    return {sum.value, overflow};
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 796

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 797

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 798

~~~~cpp
  constexpr ValueWithOverflow SubtractSigned(const Integer &y) const {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 799

~~~~cpp
    bool isNegative{IsNegative()};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 800

~~~~cpp
    bool sameSign{isNegative == y.IsNegative()};
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 801

~~~~cpp
    ValueWithCarry diff{AddUnsigned(y.Negate().value)};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 802

~~~~cpp
    bool overflow{!sameSign && diff.value.IsNegative() != isNegative};
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 803

~~~~cpp
    return {diff.value, overflow};
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 804

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 805

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 806

~~~~cpp
  // DIM(X,Y)=MAX(X-Y, 0)
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 807

~~~~cpp
  constexpr ValueWithOverflow DIM(const Integer &y) const {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 808

~~~~cpp
    if (CompareSigned(y) != Ordering::Greater) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 809

~~~~cpp
      return {};
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 810

~~~~cpp
    } else {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 811

~~~~cpp
      return SubtractSigned(y);
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 812

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 813

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 814

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 815

~~~~cpp
  constexpr ValueWithOverflow SIGN(bool toNegative) const {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 816

~~~~cpp
    if (toNegative == IsNegative()) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 817

~~~~cpp
      return {*this, false};
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 818

~~~~cpp
    } else if (toNegative) {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 819

~~~~cpp
      return Negate();
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 820

~~~~cpp
    } else {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 821

~~~~cpp
      return ABS();
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 822

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 823

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 824

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 825

~~~~cpp
  constexpr ValueWithOverflow SIGN(const Integer &sign) const {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 826

~~~~cpp
    return SIGN(sign.IsNegative());
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 827

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 828

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 829

~~~~cpp
  constexpr Product MultiplyUnsigned(const Integer &y) const {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 830

~~~~cpp
    Part product[2 * parts]{}; // little-endian full product
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 831

~~~~cpp
    for (int j{0}; j < parts; ++j) {
~~~~
- EN: Starts a loop that iterates over a sequence or range.
- CN: 开始一个遍历序列或范围的循环。

### Line 832

~~~~cpp
      if (Part xpart{LEPart(j)}) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 833

~~~~cpp
        for (int k{0}; k < parts; ++k) {
~~~~
- EN: Starts a loop that iterates over a sequence or range.
- CN: 开始一个遍历序列或范围的循环。

### Line 834

~~~~cpp
          if (Part ypart{y.LEPart(k)}) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 835

~~~~cpp
            BigPart xy{xpart};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 836

~~~~cpp
            xy *= ypart;
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 837

~~~~cpp
#if defined __GNUC__ && __GNUC__ < 8 || __GNUC__ >= 12
~~~~
- EN: Controls conditional compilation with `#if`.
- CN: 使用 `#if` 控制条件编译。

### Line 838

~~~~cpp
            // && to < (2 * parts) was added to avoid GCC build failure on
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 839

~~~~cpp
            // -Werror=array-bounds. This can be removed if -Werror is disabled.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 840

~~~~cpp
            for (int to{j + k}; xy != 0 && to < (2 * parts); ++to) {
~~~~
- EN: Starts a loop that iterates over a sequence or range.
- CN: 开始一个遍历序列或范围的循环。

### Line 841

~~~~cpp
#else
~~~~
- EN: Controls conditional compilation with `#else`.
- CN: 使用 `#else` 控制条件编译。

### Line 842

~~~~cpp
            for (int to{j + k}; xy != 0; ++to) {
~~~~
- EN: Starts a loop that iterates over a sequence or range.
- CN: 开始一个遍历序列或范围的循环。

### Line 843

~~~~cpp
#endif
~~~~
- EN: Controls conditional compilation with `#endif`.
- CN: 使用 `#endif` 控制条件编译。

### Line 844

~~~~cpp
              xy += product[to];
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 845

~~~~cpp
              product[to] = xy & partMask;
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 846

~~~~cpp
              xy >>= partBits;
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 847

~~~~cpp
            }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 848

~~~~cpp
          }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 849

~~~~cpp
        }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 850

~~~~cpp
      }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 851

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 852

~~~~cpp
    Integer upper{nullptr}, lower{nullptr};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 853

~~~~cpp
    for (int j{0}; j < parts; ++j) {
~~~~
- EN: Starts a loop that iterates over a sequence or range.
- CN: 开始一个遍历序列或范围的循环。

### Line 854

~~~~cpp
      lower.LEPart(j) = product[j];
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 855

~~~~cpp
      upper.LEPart(j) = product[j + parts];
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 856

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 857

~~~~cpp
    if constexpr (topPartBits < partBits) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 858

~~~~cpp
      upper = upper.SHIFTL(partBits - topPartBits);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 859

~~~~cpp
      upper.LEPart(0) |= lower.LEPart(parts - 1) >> topPartBits;
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 860

~~~~cpp
      lower.LEPart(parts - 1) &= topPartMask;
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 861

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 862

~~~~cpp
    return {upper, lower};
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 863

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 864

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 865

~~~~cpp
  constexpr Product MultiplySigned(const Integer &y) const {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 866

~~~~cpp
    bool yIsNegative{y.IsNegative()};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 867

~~~~cpp
    Integer absy{y};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 868

~~~~cpp
    if (yIsNegative) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 869

~~~~cpp
      absy = y.Negate().value;
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 870

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 871

~~~~cpp
    bool isNegative{IsNegative()};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 872

~~~~cpp
    Integer absx{*this};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 873

~~~~cpp
    if (isNegative) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 874

~~~~cpp
      absx = Negate().value;
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 875

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 876

~~~~cpp
    Product product{absx.MultiplyUnsigned(absy)};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 877

~~~~cpp
    if (isNegative != yIsNegative) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 878

~~~~cpp
      product.lower = product.lower.NOT();
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 879

~~~~cpp
      product.upper = product.upper.NOT();
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 880

~~~~cpp
      Integer one{1};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 881

~~~~cpp
      auto incremented{product.lower.AddUnsigned(one)};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 882

~~~~cpp
      product.lower = incremented.value;
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 883

~~~~cpp
      if (incremented.carry) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 884

~~~~cpp
        product.upper = product.upper.AddUnsigned(one).value;
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 885

~~~~cpp
      }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 886

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 887

~~~~cpp
    return product;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 888

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 889

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 890

~~~~cpp
  constexpr QuotientWithRemainder DivideUnsigned(const Integer &divisor) const {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 891

~~~~cpp
    if (divisor.IsZero()) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 892

~~~~cpp
      return {MASKR(bits), Integer{}, true, false}; // overflow to max value
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 893

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 894

~~~~cpp
    int bitsDone{LEADZ()};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 895

~~~~cpp
    Integer top{SHIFTL(bitsDone)};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 896

~~~~cpp
    Integer quotient, remainder;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 897

~~~~cpp
    for (; bitsDone < bits; ++bitsDone) {
~~~~
- EN: Starts a loop that iterates over a sequence or range.
- CN: 开始一个遍历序列或范围的循环。

### Line 898

~~~~cpp
      auto doubledTop{top.AddUnsigned(top)};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 899

~~~~cpp
      top = doubledTop.value;
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 900

~~~~cpp
      remainder = remainder.AddUnsigned(remainder, doubledTop.carry).value;
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 901

~~~~cpp
      bool nextBit{remainder.CompareUnsigned(divisor) != Ordering::Less};
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 902

~~~~cpp
      quotient = quotient.AddUnsigned(quotient, nextBit).value;
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 903

~~~~cpp
      if (nextBit) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 904

~~~~cpp
        remainder = remainder.SubtractSigned(divisor).value;
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 905

~~~~cpp
      }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 906

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 907

~~~~cpp
    return {quotient, remainder, false, false};
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 908

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 909

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 910

~~~~cpp
  // A nonzero remainder has the sign of the dividend, i.e., it computes
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 911

~~~~cpp
  // the MOD intrinsic (X-INT(X/Y)*Y), not MODULO (which is below).
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 912

~~~~cpp
  // 8/5 = 1r3;  -8/5 = -1r-3;  8/-5 = -1r3;  -8/-5 = 1r-3
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 913

~~~~cpp
  constexpr QuotientWithRemainder DivideSigned(Integer divisor) const {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 914

~~~~cpp
    bool dividendIsNegative{IsNegative()};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 915

~~~~cpp
    bool negateQuotient{dividendIsNegative};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 916

~~~~cpp
    Ordering divisorOrdering{divisor.CompareToZeroSigned()};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 917

~~~~cpp
    if (divisorOrdering == Ordering::Less) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 918

~~~~cpp
      negateQuotient = !negateQuotient;
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 919

~~~~cpp
      auto negated{divisor.Negate()};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 920

~~~~cpp
      if (negated.overflow) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 921

~~~~cpp
        // divisor was (and is) the most negative number
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 922

~~~~cpp
        if (CompareUnsigned(divisor) == Ordering::Equal) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 923

~~~~cpp
          return {MASKR(1), Integer{}, false, bits <= 1};
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 924

~~~~cpp
        } else {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 925

~~~~cpp
          return {Integer{}, *this, false, false};
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 926

~~~~cpp
        }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 927

~~~~cpp
      }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 928

~~~~cpp
      divisor = negated.value;
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 929

~~~~cpp
    } else if (divisorOrdering == Ordering::Equal) {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 930

~~~~cpp
      // division by zero
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 931

~~~~cpp
      if (dividendIsNegative) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 932

~~~~cpp
        return {MASKL(1), Integer{}, true, false};
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 933

~~~~cpp
      } else {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 934

~~~~cpp
        return {MASKR(bits - 1), Integer{}, true, false};
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 935

~~~~cpp
      }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 936

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 937

~~~~cpp
    Integer dividend{*this};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 938

~~~~cpp
    if (dividendIsNegative) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 939

~~~~cpp
      auto negated{Negate()};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 940

~~~~cpp
      if (negated.overflow) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 941

~~~~cpp
        // Dividend was (and remains) the most negative number.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 942

~~~~cpp
        // See whether the original divisor was -1 (if so, it's 1 now).
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 943

~~~~cpp
        if (divisorOrdering == Ordering::Less &&
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 944

~~~~cpp
            divisor.CompareUnsigned(Integer{1}) == Ordering::Equal) {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 945

~~~~cpp
          // most negative number / -1 is the sole overflow case
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 946

~~~~cpp
          return {*this, Integer{}, false, true};
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 947

~~~~cpp
        }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 948

~~~~cpp
      } else {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 949

~~~~cpp
        dividend = negated.value;
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 950

~~~~cpp
      }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 951

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 952

~~~~cpp
    // Overflow is not possible, and both the dividend and divisor
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 953

~~~~cpp
    // are now positive.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 954

~~~~cpp
    QuotientWithRemainder result{dividend.DivideUnsigned(divisor)};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 955

~~~~cpp
    if (negateQuotient) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 956

~~~~cpp
      result.quotient = result.quotient.Negate().value;
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 957

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 958

~~~~cpp
    if (dividendIsNegative) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 959

~~~~cpp
      result.remainder = result.remainder.Negate().value;
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 960

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 961

~~~~cpp
    return result;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 962

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 963

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 964

~~~~cpp
  // Result has the sign of the divisor argument.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 965

~~~~cpp
  // 8 mod 5 = 3;  -8 mod 5 = 2;  8 mod -5 = -2;  -8 mod -5 = -3
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 966

~~~~cpp
  constexpr ValueWithOverflow MODULO(const Integer &divisor) const {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 967

~~~~cpp
    bool negativeDivisor{divisor.IsNegative()};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 968

~~~~cpp
    bool distinctSigns{IsNegative() != negativeDivisor};
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 969

~~~~cpp
    QuotientWithRemainder divided{DivideSigned(divisor)};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 970

~~~~cpp
    if (distinctSigns && !divided.remainder.IsZero()) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 971

~~~~cpp
      return {divided.remainder.AddUnsigned(divisor).value, divided.overflow};
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 972

~~~~cpp
    } else {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 973

~~~~cpp
      return {divided.remainder, divided.overflow};
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 974

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 975

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 976

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 977

~~~~cpp
  constexpr PowerWithErrors Power(const Integer &exponent) const {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 978

~~~~cpp
    PowerWithErrors result{1, false, false, false};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 979

~~~~cpp
    if (exponent.IsZero()) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 980

~~~~cpp
      // x**0 -> 1, including the case 0**0, which is not defined specifically
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 981

~~~~cpp
      // in F'18 afaict; however, other Fortrans tested all produce 1, not 0,
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 982

~~~~cpp
      // apart from nagfor, which stops with an error at runtime.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 983

~~~~cpp
      // Ada, APL, C's pow(), Haskell, Julia, MATLAB, and R all produce 1 too.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 984

~~~~cpp
      // F'77 explicitly states that 0**0 is mathematically undefined and
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 985

~~~~cpp
      // therefore prohibited.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 986

~~~~cpp
      result.zeroToZero = IsZero();
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 987

~~~~cpp
    } else if (exponent.IsNegative()) {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 988

~~~~cpp
      if (IsZero()) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 989

~~~~cpp
        result.divisionByZero = true;
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 990

~~~~cpp
        result.power = MASKR(bits - 1);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 991

~~~~cpp
      } else if (CompareSigned(Integer{1}) == Ordering::Equal) {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 992

~~~~cpp
        result.power = *this; // 1**x -> 1
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 993

~~~~cpp
      } else if (CompareSigned(Integer{-1}) == Ordering::Equal) {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 994

~~~~cpp
        if (exponent.BTEST(0)) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 995

~~~~cpp
          result.power = *this; // (-1)**x -> -1 if x is odd
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 996

~~~~cpp
        }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 997

~~~~cpp
      } else {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 998

~~~~cpp
        result.power.Clear(); // j**k -> 0 if |j| > 1 and k < 0
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 999

~~~~cpp
      }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1000

~~~~cpp
    } else {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 1001

~~~~cpp
      Integer shifted{*this};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1002

~~~~cpp
      Integer pow{exponent};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1003

~~~~cpp
      int nbits{bits - pow.LEADZ()};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1004

~~~~cpp
      for (int j{0}; j < nbits; ++j) {
~~~~
- EN: Starts a loop that iterates over a sequence or range.
- CN: 开始一个遍历序列或范围的循环。

### Line 1005

~~~~cpp
        if (pow.BTEST(j)) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 1006

~~~~cpp
          Product product{result.power.MultiplySigned(shifted)};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1007

~~~~cpp
          result.power = product.lower;
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 1008

~~~~cpp
          result.overflow |= product.SignedMultiplicationOverflowed();
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 1009

~~~~cpp
        }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1010

~~~~cpp
        if (j + 1 < nbits) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 1011

~~~~cpp
          Product squared{shifted.MultiplySigned(shifted)};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1012

~~~~cpp
          result.overflow |= squared.SignedMultiplicationOverflowed();
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 1013

~~~~cpp
          shifted = squared.lower;
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 1014

~~~~cpp
        }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1015

~~~~cpp
      }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1016

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1017

~~~~cpp
    return result;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 1018

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1019

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1020

~~~~cpp
private:
~~~~
- EN: Switches subsequent class members to `private` access.
- CN: 将后续类成员切换为 `private` 访问级别。

### Line 1021

~~~~cpp
  // A private constructor, selected by the use of nullptr,
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1022

~~~~cpp
  // that is used by member functions when it would be a waste
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1023

~~~~cpp
  // of time to initialize parts_[].
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1024

~~~~cpp
  constexpr Integer(std::nullptr_t) {}
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1025

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1026

~~~~cpp
  // Accesses parts in little-endian order.
~~~~
- EN: Documents intent, behavior, or context for the nearby code.
- CN: 说明附近代码的意图、行为或上下文。

### Line 1027

~~~~cpp
  constexpr const Part &LEPart(int part) const {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 1028

~~~~cpp
    if constexpr (littleEndian) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 1029

~~~~cpp
      return part_[part];
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 1030

~~~~cpp
    } else {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 1031

~~~~cpp
      return part_[parts - 1 - part];
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 1032

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1033

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1034

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1035

~~~~cpp
  constexpr Part &LEPart(int part) {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 1036

~~~~cpp
    if constexpr (littleEndian) {
~~~~
- EN: Starts a conditional branch.
- CN: 开始一个条件分支。

### Line 1037

~~~~cpp
      return part_[part];
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 1038

~~~~cpp
    } else {
~~~~
- EN: Begins a new compound statement or declaration body.
- CN: 开始一个新的复合语句或声明体。

### Line 1039

~~~~cpp
      return part_[parts - 1 - part];
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 1040

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1041

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1042

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1043

~~~~cpp
  constexpr void SetLEPart(int part, Part x) {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 1044

~~~~cpp
    LEPart(part) = x & PartMask(part);
~~~~
- EN: Declares a function or method signature.
- CN: 声明一个函数或方法签名。

### Line 1045

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1046

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1047

~~~~cpp
  static constexpr Part PartMask(int part) {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 1048

~~~~cpp
    return part == parts - 1 ? topPartMask : partMask;
~~~~
- EN: Returns a value from the current function or method.
- CN: 从当前函数或方法返回一个值。

### Line 1049

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1050

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1051

~~~~cpp
  constexpr void Clear() {
~~~~
- EN: Begins a function, method, or callable definition.
- CN: 开始一个函数、方法或可调用对象的定义。

### Line 1052

~~~~cpp
    for (int j{0}; j < parts; ++j) {
~~~~
- EN: Starts a loop that iterates over a sequence or range.
- CN: 开始一个遍历序列或范围的循环。

### Line 1053

~~~~cpp
      part_[j] = 0;
~~~~
- EN: Computes and stores a value through assignment or initialization.
- CN: 通过赋值或初始化计算并保存一个值。

### Line 1054

~~~~cpp
    }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1055

~~~~cpp
  }
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1056

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1057

~~~~cpp
  Part part_[partsWithAlignment]{};
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1058

~~~~cpp
};
~~~~
- EN: Closes the current scope or declaration block.
- CN: 结束当前作用域或声明块。

### Line 1059

~~~~text
<blank line>
~~~~
- EN: Separates adjacent declarations or logical blocks.
- CN: 用于分隔相邻的声明或逻辑块。

### Line 1060

~~~~cpp
extern template class Integer<8>;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1061

~~~~cpp
extern template class Integer<16>;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1062

~~~~cpp
extern template class Integer<32>;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1063

~~~~cpp
extern template class Integer<64>;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1064

~~~~cpp
using X87IntegerContainer =
~~~~
- EN: Declares and initializes a local variable or data member.
- CN: 声明并初始化一个局部变量或数据成员。

### Line 1065

~~~~cpp
    Integer<80, isHostLittleEndian, 16, std::uint16_t, std::uint32_t, 128>;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1066

~~~~cpp
extern template class Integer<80, isHostLittleEndian, 16, std::uint16_t,
~~~~
- EN: Contributes to a larger multi-line declaration or expression.
- CN: 构成更大多行声明或表达式的一部分。

### Line 1067

~~~~cpp
    std::uint32_t, 128>;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1068

~~~~cpp
extern template class Integer<128>;
~~~~
- EN: Completes a declaration, definition, or expression statement.
- CN: 完成一个声明、定义或表达式语句。

### Line 1069

~~~~cpp
} // namespace Fortran::evaluate::value
~~~~
- EN: Closes namespace scope `Fortran::evaluate::value`.
- CN: 结束命名空间作用域 `Fortran::evaluate::value`。

### Line 1070

~~~~cpp
#endif // FORTRAN_EVALUATE_INTEGER_H_
~~~~
- EN: Controls conditional compilation with `#endif`.
- CN: 使用 `#endif` 控制条件编译。

## Key Concepts / 关键概念

- **Public interface surface / 公共接口层**: The file mainly exposes declarations meant to be consumed by other translation units. / 该文件主要暴露供其他编译单元使用的声明。
- **Templates and generic programming / 模板与泛型编程**: Templates let the file abstract behavior across types or compile-time parameters. / 模板让该文件能够在类型或编译期参数之间抽象行为。
- **Compile-time validation / 编译期校验**: Static assertions catch configuration mistakes before runtime. / 静态断言可在运行前捕获配置错误。
- **Namespace organization / 命名空间组织**: Namespaces keep related symbols grouped and reduce accidental collisions. / 命名空间用于组织相关符号并减少命名冲突。
- **Preprocessor coordination / 预处理器协作**: Preprocessor directives shape how the file is configured or expanded. / 预处理指令决定文件的配置方式或展开方式。

## Dependencies / 依赖关系

- **Internal project dependencies / 工程内依赖**:
  - `flang/Common/bit-population-count.h` — referenced directly from this file / 该文件直接引用
  - `flang/Common/leading-zero-bit-count.h` — referenced directly from this file / 该文件直接引用
  - `flang/Evaluate/common.h` — referenced directly from this file / 该文件直接引用
- **External or standard dependencies / 外部或标准依赖**:
  - `<cinttypes>` — supporting library header / 支撑性库头文件
  - `<climits>` — supporting library header / 支撑性库头文件
  - `<cstddef>` — supporting library header / 支撑性库头文件
  - `<cstdint>` — supporting library header / 支撑性库头文件
  - `<string>` — supporting library header / 支撑性库头文件
  - `<type_traits>` — supporting library header / 支撑性库头文件
